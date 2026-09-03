# Module 17: Rate Limiting & Backpressure

## Learning Objectives

By the end of this module, you will be able to:
- Implement and compare core rate-limiting algorithms: **Token Bucket, Leaky Bucket, Fixed Window, Sliding Window Log, and Sliding Window Counter**.
- Architect distributed rate limiters using **Redis Token Buckets with Lua Scripts** to ensure atomicity.
- Implement **Adaptive Load Shedding** and **Admission Control** using Little's Law and gradient concurrency limits (Netflix Concurrency Limits pattern).
- Apply reactive backpressure across async streaming pipelines to prevent memory saturation.

---

## Lessons in This Module

| File | Topic | Focus |
| :--- | :--- | :--- |
| [01-token-bucket-and-leaky-bucket-algorithms.md](01-token-bucket-and-leaky-bucket-algorithms.md) | Token Bucket vs. Leaky Bucket | Algorithm trade-offs, burst handling, Redis Lua implementation |
| [02-adaptive-load-shedding-and-admission-control.md](02-adaptive-load-shedding-and-admission-control.md) | Adaptive Load Shedding & Admission Control | Shedding non-critical load, Netflix gradient concurrency limits |
