# Module 21: Distributed Systems Failures & Cascades

## Learning Objectives

By the end of this module, you will be able to:
- Deconstruct the multi-step mechanics of a **Distributed Cascading Failure**: **Local Failure $\to$ Dependency Failure $\to$ Retry $\to$ Load Increase $\to$ Resource Exhaustion $\to$ Cascading Collapse**.
- Break every link in the cascade chain using **Rate Limiting, Load Shedding, Circuit Breakers, and Bulkheads**.
- Reason rigorously about partial distributed failures: **Network Partitions, Clock Skew, NTP Drift, Duplicate Messages, and Out-of-Order Delivery**.
- Design distributed systems using **Lamport Timestamps, Vector Clocks, and Idempotency Keys**.

---

## Lessons in This Module

| File | Topic | Focus |
| :--- | :--- | :--- |
| [01-anatomy-of-a-cascading-failure.md](01-anatomy-of-a-cascading-failure.md) | Anatomy of a Cascading Failure | The 6-link cascade chain, thundering herds, breaking every link |
| [02-network-partitions-clock-skew-and-ordering.md](02-network-partitions-clock-skew-and-ordering.md) | Partitions, Clock Skew & Message Ordering | CAP theorem, NTP skew (TrueTime), idempotency, deduplication |
