# Production Reliability Anti-Patterns Catalog

A catalog of hazardous design choices, operational habits, and architectural traps that cause production outages.

---

## ⚠️ The 10 Most Dangerous Production Anti-Patterns

| # | Anti-Pattern Name | Description & Mechanism | Disastrous Production Consequence |
| :-: | :--- | :--- | :--- |
| **01** | **Immediate Infinite Retries** | Retrying failed network calls in a tight loop without backoff or jitter | **Retry Storm**: Downstream service goes down; millions of retries prevent it from ever rebooting |
| **02** | **Blind Restart on High CPU** | Restarting servers as first mitigation without capturing heap/thread dumps | **Outage Recurrence**: Root cause survives; newly started instances instantly saturate and crash |
| **03** | **Unbounded In-Memory Queues** | Producers pushing items to unlimited JVM memory queues | **OOM Kill**: Memory grows monotonically until Linux OOM-killer terminates the process |
| **04** | **Averaged Latency Dashboards** | Monitoring `AVG(response_time)` instead of p95/p99/p99.9 percentiles | **Hidden Tail Pain**: 1% of users experience 30-second timeouts while average looks healthy (50ms) |
| **05** | **Shared Global Thread Pool** | Single thread pool handling both fast internal cache reads and slow external API calls | **Thread Starvation**: One slow external vendor locks all threads, crashing the entire service |
| **06** | **Missing DB Lock Timeout** | Running schema migrations without `SET lock_timeout = '3s';` | **Lock Queue Outage**: DDL queues behind long query; all incoming web queries queue behind DDL |
| **07** | **Liveness Probe Checking Dependencies** | Pod liveness probe returning 500 when downstream database or cache is slow | **Cascading Pod Restarts**: Kubernetes restarts all pods simultaneously, amplifying DB overload |
| **08** | **Synchronous External RPCs in DB Transactions** | Calling third-party HTTP payment gateway inside `@Transactional` | **Connection Exhaustion**: Database connection held open for 5 seconds waiting for HTTP reply |
| **09** | **Silent Dangerous Fallbacks** | Returning empty list `[]` on recommendation failure that downstream interprets as "no products" | **Silent Data Corruption**: Customers see empty catalog and checkout empty carts |
| **10** | **Alerting on Symptoms Without Actionability** | Paging engineers on "CPU > 75%" when no customer latency or error SLA is breached | **Alert Fatigue**: Engineers ignore alerts, missing real customer-impacting outages |
