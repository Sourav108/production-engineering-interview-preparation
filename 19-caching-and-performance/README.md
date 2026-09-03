# Module 19: Caching & Performance in Production

## Learning Objectives

By the end of this module, you will be able to:
- Architect reliable caching patterns (**Cache-Aside, Write-Through, Write-Behind, Refresh-Ahead**) with explicit consistency guarantees.
- Mitigate the three catastrophic cache failure modes: **Cache Stampede (Dogpiling), Cache Penetration, and Cache Avalanche**.
- Implement **Probabilistic Early Expiration (XFetch algorithm)** and mutex locking to prevent database meltdown during cache key expiry.
- Design cache failure fallback strategies ensuring the database is never exposed to 100% uncached load.

---

## Lessons in This Module

| File | Topic | Focus |
| :--- | :--- | :--- |
| [01-cache-aside-and-invalidation.md](01-cache-aside-and-invalidation.md) | Cache-Aside & Invalidation Strategies | Cache-Aside vs Write-Through, Dual-Write race conditions, TTL tuning |
| [02-cache-stampede-and-dogpiling-mitigation.md](02-cache-stampede-and-dogpiling-mitigation.md) | Cache Stampedes & Dogpiling Mitigation | Stampede mechanics, Mutex locks, XFetch probabilistic early recomputation |
