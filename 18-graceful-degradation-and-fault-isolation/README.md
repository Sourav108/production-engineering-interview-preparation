# Module 18: Graceful Degradation & Fault Isolation

## Learning Objectives

By the end of this module, you will be able to:
- Architect **Bulkhead Isolation** across thread pools, connection pools, and compute clusters to prevent noisy-neighbor and slow-dependency contagion.
- Implement **Graceful Degradation** patterns: static mock fallbacks, stale cache reads, and dynamic feature shedding.
- Identify and prevent the **Dangerous Fallback Anti-Pattern** (silently returning corrupt or misleading default state).
- Maintain partial availability ($99.99\%$ core availability) during multi-service dependency outages.

---

## Lessons in This Module

| File | Topic | Focus |
| :--- | :--- | :--- |
| [01-bulkhead-isolation-architecture.md](01-bulkhead-isolation-architecture.md) | Bulkhead Isolation Architecture | Physical vs thread pool bulkheads, noisy-neighbor defense |
| [02-graceful-fallbacks-and-dangerous-fallback-traps.md](02-graceful-fallbacks-and-dangerous-fallback-traps.md) | Fallbacks & Dangerous Fallback Traps | Stale cache fallbacks, silent data corruption, dangerous empty-list returns |
