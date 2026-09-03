# 03. Automated Database Failover, Consensus & Split-Brain Prevention

## 1. Problem
A network partition severs communication between a primary database and its replica. Both nodes assume the other is dead and accept writes concurrently (**Split-Brain**), resulting in irreversible data divergence and corruption.

## 2. Production Context
Automated failover must guarantee that at any single instant in time, **exactly one primary node exists** (Single Leader Invariant).

## 3. Mental Model: Quorum & Fencing (STONITH)

```mermaid
flowchart TD
    subgraph Quorum Consensus [Etcd / Raft 3-Node Cluster]
        E1[Etcd Node 1] <--> E2[Etcd Node 2]
        E2 <--> E3[Etcd Node 3]
    end

    subgraph Database Nodes [Patroni Managed]
        DB_PRI[Old Primary: Network Partitioned!]
        DB_REP[Replica: Promoted by Quorum]
    end

    E1 -.->|Heartbeat Lost| DB_PRI
    E1 -->|Elects New Leader (2/3 Quorum)| DB_REP
    DB_PRI -->|Fenced / Killed via STONITH| DEAD[Power Off / SIGKILL]
    DB_REP -->|Promoted to Read-Write Primary| APP[Application Writes Routed Here]
```

---

## 4. The 3 Invariants of Safe Database Failover

1. **Strict Majority Quorum ($\frac{N}{2} + 1$)**: Leader election requires agreement from an odd number of consensus nodes (e.g. 3 or 5 nodes in etcd/Consul).
2. **Node Fencing (STONITH - "Shoot The Other Node In The Head")**: The old primary must be physically fenced (power-off, AWS API terminate instance, or network port blocked) *before* the replica is promoted.
3. **Fencing Tokens / Epoch Numbers**: Downstream storage verifies a monotonically increasing epoch number (e.g. Raft term) on every write; writes from older epoch primaries are rejected immediately.

---

## 5. Interview Questions & Model Answers

**Q1: What is Split-Brain in distributed database clusters, and how is it prevented?**
**Answer**: Split-Brain occurs when a network partition divides a database cluster into disconnected segments, and each segment mistakenly elects a leader that accepts writes independently. When the partition heals, the two divergent datasets cannot be reconciled without manual conflict resolution and data loss. To prevent split-brain:
1. **Majority Quorum**: Only the partition that can communicate with a strict majority ($\ge \frac{N+1}{2}$) of consensus nodes is permitted to elect a leader or accept writes.
2. **Node Fencing (STONITH)**: Automatically terminate or power off the old primary before promoting a replica.
3. **Monotonic Epoch Numbers**: Every write is tagged with the current election epoch; any lingering old primary attempting to write to storage with a stale epoch is rejected.
