# Sibling Repositories & External References

This repository focuses strictly on **production reliability, incident management, observability, performance engineering, capacity planning, and operational architecture**. It links to dedicated sibling repositories for language syntax, framework internals, and database querying rather than duplicating that content.

---

## 🔗 Sibling Repositories Boundary Matrix

| Sibling Domain | Sibling Repository | This Repository Owns Instead |
| :--- | :--- | :--- |
| **Java & JVM Internals** | [`java-interview-preparation`](../java-interview-preparation/) | JVM production behavior, GC pause incidents, thread starvation, native memory leaks, CPU profiling |
| **Spring Boot Framework** | [`spring-boot-interview-preparation`](../spring-boot-interview-preparation/) | Spring production operations, HikariCP pool exhaustion, health probes, graceful shutdown, release incidents |
| **SQL & Relational DBs** | [`sql-interview-preparation`](../sql-interview-preparation/) | Database production incidents, connection exhaustion, lock queues, replication lag, failover, backup failures |
| **Distributed System Design** | [`system-design-interview`](../system-design-interview/) | Production operations, failure modes, blast radius containment, backpressure, high availability, disaster recovery |
| **Low-Level Design (LLD)** | [`LLD`](../LLD/) / [`awesome-low-level-design`](../awesome-low-level-design/) | Resilience patterns implementation: circuit breakers, token bucket rate limiters, bulkhead thread pools |

---

## 📚 Industry Standards & Documentation References
- **OpenTelemetry Standard**: [opentelemetry.io](https://opentelemetry.io/docs/)
- **Prometheus Metric Types & Alerting**: [prometheus.io/docs](https://prometheus.io/docs/)
- **Linux Kernel Documentation (`/proc`, PSI, cgroups v2)**: [kernel.org/doc](https://www.kernel.org/doc/html/latest/)
- **Google SRE Books (Site Reliability Engineering, The Site Reliability Workbook)**: [sre.google/books](https://sre.google/books/)
