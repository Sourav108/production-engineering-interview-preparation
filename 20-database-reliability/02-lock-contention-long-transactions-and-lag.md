# 02. Database Lock Contention, Long Transactions & Replication Lag

## 1. Problem
A developer executes an `ALTER TABLE` migration during peak hours. The DDL statement acquires an `AccessExclusiveLock`, queuing behind a 10-second analytics query. Within seconds, hundreds of incoming customer web requests queue behind the DDL statement, freezing the entire application.

## 2. Production Context
Lock contention and long-running transactions are the #1 cause of sudden database outages in relational systems.

## 3. Mental Model: The Lock Queue Inversion Disaster

```
                                 The DDL Lock Queue Trap
 ──────────────────────────────────────────────────────────────────────────────────────────
 1. Long Query (PID 100): SELECT * FROM orders (Takes 30s, holds AccessShareLock).
 2. Migration (PID 101): ALTER TABLE orders ADD COLUMN status (Demands AccessExclusiveLock).
    → BLOCKED waiting for PID 100!
 3. Incoming Web Reads (PIDs 102..500): SELECT * FROM orders WHERE id = 123.
    → BLOCKED waiting behind PID 101! (PostgreSQL prioritizes exclusive lock requests).
    → OUTAGE: All 500 web threads freeze waiting for a 30s query to finish!
```

---

## 4. The Lock Timeout Protection Invariant

**Rule**: Never execute any DDL schema migration or maintenance command without an explicit **Lock Timeout**:

```sql
-- Safe Zero-Downtime Migration Pattern
SET lock_timeout = '2s';
SET statement_timeout = '10s';
ALTER TABLE orders ADD COLUMN status VARCHAR(20);
```
*(If the lock cannot be acquired within 2 seconds, the DDL command fails immediately without queuing, protecting customer traffic from freezing).*

---

## 5. Triage Queries: Finding Blocked Queries in PostgreSQL

```sql
-- Find which query is blocking which other queries
SELECT
    blocked_locks.pid     AS blocked_pid,
    blocked_activity.usename  AS blocked_user,
    blocking_locks.pid    AS blocking_pid,
    blocking_activity.usename AS blocking_user,
    blocked_activity.query    AS blocked_statement,
    blocking_activity.query   AS blocking_statement
FROM  pg_catalog.pg_locks         blocked_locks
JOIN pg_catalog.pg_stat_activity blocked_activity ON blocked_activity.pid = blocked_locks.pid
JOIN pg_catalog.pg_locks         blocking_locks 
    ON blocking_locks.locktype = blocked_locks.locktype
    AND blocking_locks.database IS NOT DISTINCT FROM blocked_locks.database
    AND blocking_locks.relation IS NOT DISTINCT FROM blocked_locks.relation
    AND blocking_locks.page IS NOT DISTINCT FROM blocked_locks.page
    AND blocking_locks.tuple IS NOT DISTINCT FROM blocked_locks.tuple
    AND blocking_locks.virtualxid IS NOT DISTINCT FROM blocked_locks.virtualxid
    AND blocking_locks.transactionid IS NOT DISTINCT FROM blocked_locks.transactionid
    AND blocking_locks.classid IS NOT DISTINCT FROM blocked_locks.classid
    AND blocking_locks.objid IS NOT DISTINCT FROM blocked_locks.objid
    AND blocking_locks.objsubid IS NOT DISTINCT FROM blocked_locks.objsubid
    AND blocking_locks.pid != blocked_locks.pid
JOIN pg_catalog.pg_stat_activity blocking_activity ON blocking_activity.pid = blocking_locks.pid
WHERE NOT blocked_locks.granted;
```

---

## 6. Interview Questions & Model Answers

**Q1: How does a long-running SELECT transaction cause database table bloat in PostgreSQL MVCC?**
**Answer**: In PostgreSQL Multi-Version Concurrency Control (MVCC), when a row is updated or deleted, the old tuple remains on disk as a "dead tuple" until the `VACUUM` process reclaims it. However, `VACUUM` can only purge dead tuples with an `xmin` older than the **oldest active transaction in the entire database**. If a long-running transaction (e.g. an uncommitted `idle in transaction` connection or 2-hour reporting query) is running, `VACUUM` is blocked from removing *any* dead tuples generated after that transaction began, causing table and index disk bloat to explode into hundreds of gigabytes and degrading query performance.

**Q2: How do you solve Read-Your-Own-Writes inconsistency when querying asynchronous read replicas?**
**Answer**: Asynchronous replication introduces **Replication Lag** ($\text{WAL bytes}$ pending apply). If a user updates their profile and immediately refreshes the page, routing the read to a lagged replica will display the old profile data. To solve this:
1. **Sticky Session / Pinning to Primary**: Route all reads for a user to the Primary database for 5–10 seconds following any write operation.
2. **Replication Offset Tracking (Causal Consistency)**: Return the write's Log Sequence Number (LSN) to the client; the client only queries read replicas whose `pg_last_wal_replay_lsn()` has advanced past that LSN.
