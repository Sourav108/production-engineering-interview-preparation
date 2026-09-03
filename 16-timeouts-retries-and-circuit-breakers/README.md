# Module 16: Timeouts, Retries & Circuit Breakers

## Learning Objectives

By the end of this module, you will be able to:
- Explain mathematically how naive client retries transform minor transient glitches into catastrophic **Retry Storms** that permanently prevent downstream systems from recovering.
- Implement **Exponential Backoff with Full Jitter** and enforce **Retry Budgets** (e.g. max 10% retries).
- Master the **Circuit Breaker State Machine** (`Closed $\to$ Open $\to$ Half-Open`) using Resilience4j and Envoy proxy configurations.
- Configure client-side and server-side socket, connection, and request deadlines.

---

## Lessons in This Module

| File | Topic | Focus |
| :--- | :--- | :--- |
| [01-anatomy-of-a-retry-storm.md](01-anatomy-of-a-retry-storm.md) | Anatomy of a Retry Storm | How retries multiply load ($3\times - 10\times$), thundering herds, recovery failure |
| [02-exponential-backoff-full-jitter-and-retry-budgets.md](02-exponential-backoff-full-jitter-and-retry-budgets.md) | Backoff, Jitter & Retry Budgets | Full jitter formula, token-bucket retry budgets, safe retry rules |
| [03-circuit-breaker-state-machine-mechanics.md](03-circuit-breaker-state-machine-mechanics.md) | Circuit Breaker State Machine | Closed/Open/Half-Open transitions, sliding window metrics, Resilience4j |
