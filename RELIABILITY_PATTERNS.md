# Production Reliability Patterns Reference

A curated catalog of production-proven architectural and operational patterns for building resilient distributed systems.

---

## 🛡️ Core Reliability Patterns Catalog

| # | Pattern Name | Problem Solved | Key Mechanism |
| :-: | :--- | :--- | :--- |
| **01** | **Circuit Breaker** | Slow/failing dependency causing thread exhaustion | Transitions `Closed → Open → Half-Open` based on error rate |
| **02** | **Exponential Backoff + Full Jitter** | Retry storms overwhelming recovering downstream | Sleep duration $t = \text{random}(0, \min(M, B \times 2^{\text{attempt}}))$ |
| **03** | **Token Bucket Rate Limiting** | Traffic spikes saturating service capacity | Requests consume tokens refilled at steady rate |
| **04** | **Adaptive Load Shedding** | Latency death spirals when CPU/queues saturate | Rejects lower-priority traffic when p99 latency breaches threshold |
| **05** | **Bulkhead Isolation** | One slow dependency exhausting global thread pool | Dedicated thread/connection pools per downstream dependency |
| **06** | **Cache Stampede Prevention (Probabilistic Early Expiration)** | Millions of concurrent requests hitting DB on cache key expiry | Recompute key before TTL expires: $-\beta \times \log(\text{rand}()) \times \Delta > \text{TTL}$ |
| **07** | **Graceful Degradation / Fallback** | Hard outage on non-critical service feature | Return cached/stale data or default mock payload |
| **08** | **Canary Deployment** | Bad release impacting 100% of production traffic | Route 1% $\to$ 5% $\to$ 25% $\to$ 100% traffic with automated rollback |
| **09** | **Idempotent Consumer** | Duplicate message delivery in distributed queues | Unique idempotency key checked in DB before processing |
| **10** | **Dead Letter Queue (DLQ)** | Poison pill messages crashing consumer threads | Route failed messages to separate queue after $N$ retry attempts |
