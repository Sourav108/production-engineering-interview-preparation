# Production Engineering Hands-On Labs Catalog

A catalog of reproducible, local failure injection experiments and performance benchmarking labs.

---

## 🧪 Laboratory Categories

| Lab Category | Core Failure Mode / Focus | Prerequisites |
| :--- | :--- | :--- |
| **Linux Systems Labs** | CPU thrashing, memory leaks, Linux OOM kills | Docker, `stress-ng` |
| **Networking Labs** | Packet drops, DNS latency, proxy timeouts | Docker, `tc` (Traffic Control) |
| **Application Labs** | Thread pool starvation, slow dependency cascades | Java 21, Spring Boot 3.3 |
| **Database Labs** | Connection pool exhaustion, lock contention | PostgreSQL 18.6 / 16.x |
| **Observability Labs** | Building RED metrics, OpenTelemetry distributed tracing | Prometheus, Grafana, Jaeger |
| **Reliability Labs** | Circuit breaker tripping, token bucket rate limiting | Resilience4j, Redis |

*Refer to [templates/lab-template.md](templates/lab-template.md) for the standard lab structure.*
