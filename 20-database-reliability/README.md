# Module 20: Database Reliability & Production Operations

## Learning Objectives

By the end of this module, you will be able to:
- Treat relational and NoSQL databases as **stateful production dependencies** with bounded connection and I/O capacities.
- Mathematically size **HikariCP / PgBouncer Connection Pools** using the PostgreSQL spindle/core formula to prevent connection thrashing.
- Diagnose and terminate **Database Lock Queues and Long-Running Transactions** via `pg_stat_activity` and `pg_locks`.
- Manage **Replication Lag, Read-Your-Own-Writes Inconsistencies, and Automated Failover / Split-Brain Scenarios**.

*(Note: For SQL querying semantics, indexing internals, and EXPLAIN plans, refer to [`sql-interview-preparation`](../sql-interview-preparation/) linked in [`REFERENCES.md`](../REFERENCES.md)).*

---

## Lessons in This Module

| File | Topic | Focus |
| :--- | :--- | :--- |
| [01-connection-pool-sizing-and-exhaustion.md](01-connection-pool-sizing-and-exhaustion.md) | Connection Pool Sizing & Exhaustion | HikariCP pool formula, connection leak triage, PgBouncer proxying |
| [02-lock-contention-long-transactions-and-lag.md](02-lock-contention-long-transactions-and-lag.md) | Locks, Long Transactions & Replication Lag | `pg_stat_activity`, `lock_timeout`, replica lag, MVCC vacuum stalls |
| [03-failover-mechanics-and-split-brain.md](03-failover-mechanics-and-split-brain.md) | Failover Mechanics & Split-Brain | Automated promotion (Patroni/Raft), fencing / STONITH, WAL replication |
