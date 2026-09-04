# Production Engineering Hands-On Labs Catalog

A collection of **15 self-contained, reproducible production engineering lab scenarios** equipped with Docker Compose configurations, fault injection scripts, observability dashboards, and step-by-step triage procedures.

---

## 🧪 Index of Hands-On Labs

| Lab ID | Lab Title | Primary Toolchain | Domain Focus |
| :--- | :--- | :--- | :--- |
| [Lab 01](labs/lab-01-linux-host-saturation.md) | Linux Host CPU Saturation & PSI Triage | `stress-ng`, `/proc/pressure/cpu` | Linux Kernel & PSI |
| [Lab 02](labs/lab-02-jvm-memory-leak-oom.md) | JVM Memory Leak & Heap Dump Forensics | Java 21, Eclipse MAT, `jcmd` | Memory Leaks & OOM |
| [Lab 03](labs/lab-03-db-connection-pool-exhaustion.md) | HikariCP Connection Pool Exhaustion & PgBouncer | PostgreSQL 16, HikariCP, PgBouncer | Connection Pooling |
| [Lab 04](labs/lab-04-opentelemetry-distributed-tracing.md) | OpenTelemetry Tracing & Tail Latency Analysis | OTel Collector, Jaeger | Distributed Tracing |
| [Lab 05](labs/lab-05-multi-window-burn-rate-alerting.md) | Prometheus Multi-Window Multi-Burn Rate Alerting | Prometheus, Alertmanager | SLO Burn-Rate Alerting |
| [Lab 06](labs/lab-06-latency-decomposition-analysis.md) | Decomposing Latency Bottlenecks with k6 & Grafana | k6, Spring Boot, Grafana | Latency Decomposition |
| [Lab 07](labs/lab-07-cascading-retry-storm.md) | Simulating & Breaking a Cascading Retry Storm | Go Microservices, WireMock | Retry Storms & Backoff |
| [Lab 08](labs/lab-08-circuit-breaker-resilience4j.md) | Resilience4j Circuit Breaker & Fallback Pipeline | Resilience4j, Spring Cloud | Circuit Breakers |
| [Lab 09](labs/lab-09-distributed-rate-limiting-redis.md) | Distributed Rate Limiting with Atomic Redis Lua | Redis 7, NGINX | Rate Limiting |
| [Lab 10](labs/lab-10-cache-stampede-xfetch.md) | Cache Stampede Mitigation via XFetch Algorithm | Redis 7, PostgreSQL 16 | Cache Reliability |
| [Lab 11](labs/lab-11-postgresql-lock-contention.md) | Diagnosing & Resolving PostgreSQL Lock Queues | PostgreSQL 16, `pg_locks` | DB Lock Contention |
| [Lab 12](labs/lab-12-chaos-network-latency-injection.md) | Injecting Network Latency & Packet Drops | Chaos Mesh, Litmus | Chaos Engineering |
| [Lab 13](labs/lab-13-load-testing-coordinated-omission.md) | Eliminating Coordinated Omission in k6 Benchmarks | k6, HdrHistogram | Load Testing |
| [Lab 14](labs/lab-14-automated-canary-argo-rollouts.md) | Automated Canary Rollouts with Metric Analysis | Kubernetes, Argo Rollouts | Canary Deployments |
| [Lab 15](labs/lab-15-production-gameday-simulation.md) | Running a Production GameDay Simulation Drill | GameDay Simulator CLI | SRE GameDays |
