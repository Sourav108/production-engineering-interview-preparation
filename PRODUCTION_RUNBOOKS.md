# Master Production Runbooks Index

A centralized operational index linking standard production runbooks across critical failure domains.

---

## 📖 Production Runbook Catalog

| Runbook Domain | Trigger Symptom | Primary Mitigation Action | Detailed Guide |
| :--- | :--- | :--- | :--- |
| **High Host CPU Saturation** | CPU $> 90\%$ on app pods | Capture thread dump, scale replicas horizontally, inspect hot loops | [Module 03](03-processes-memory-and-cpu/) |
| **JVM Memory Leak & OOM** | Heap $> 90\%$, OOM kills | Capture heap dump (`jcmd`), restart pod, analyze memory leak | [Module 03](03-processes-memory-and-cpu/) |
| **DB Connection Pool Exhaustion** | HikariCP pool timeout errors | Inspect blocking PIDs in `pg_stat_activity`, terminate leaked txns | [Module 20](20-database-reliability/) |
| **High Latency Tail Spike** | p99 latency $> 1000\text{ms}$ | Inspect distributed trace spans, trip breaker on slow external API | [Module 12](12-latency-and-performance/) |
| **Dependency Timeout & Cascades**| 504 Gateway Timeouts | Enable fallback responses, shed non-critical traffic | [Module 16](16-timeouts-retries-and-circuit-breakers/) |
| **DNS Resolution Outage** | `UnknownHostException` | Verify CoreDNS pods, switch to fallback IP or local cache | [Module 04](04-networking-and-request-path/) |
| **TLS Certificate Expiration** | TLS handshake failures | Trigger automated Let's Encrypt / Vault cert renewal | [Module 25](25-production-security/) |
| **Disk Space Exhaustion** | Disk $> 90\%$ full | Truncate stale logs, vacuum dead tuples, rotate WAL segments | [Module 02](02-linux-and-system-behavior/) |
