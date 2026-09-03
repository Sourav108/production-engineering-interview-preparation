# Production Engineering Version Matrix

This repository tracks verified software versions, kernel assumptions, telemetry standards, and testing tooling to ensure technical accuracy and currency.

---

## 🛠️ Baseline Software Versions & Tooling

| Component | Standard / Baseline Version | Verification Source | Currency Note |
| :--- | :--- | :--- | :--- |
| **Linux Kernel** | `Linux 6.6+ LTS` (cgroups v2, PSI) | `kernel.org` | Modern cgroup v2 & Pressure Stall Information assumed |
| **OpenTelemetry** | `OTel SDK v1.35+` / Spec v1.30+ | `opentelemetry.io` | OTLP HTTP/gRPC protocol as standard tracing reference |
| **Prometheus** | `v2.53+` / `v3.0` | `prometheus.io` | Native histograms, PromQL multi-window burn rate alerts |
| **Grafana** | `v11.x+` | `grafana.com` | Dashboard panels, alerting rules, trace-to-metrics navigation |
| **PostgreSQL** | `18.6` / `16.x` | `postgresql.org` | System views (`pg_stat_activity`, `pg_stat_statements`), pool sizing |
| **Java / JVM** | `OpenJDK 21 LTS` (ZGC, G1GC) | `openjdk.org` | Garbage-First (G1) and Generational ZGC production behavior |
| **Spring Boot** | `v3.3.x` | `spring.io` | Micrometer, Actuator `/actuator/health`, HikariCP defaults |
| **Load Testing** | `k6 v0.52+` / `Vegeta v12+` | `k6.io` | Scriptable percentile load and spike testing |
| **Chaos Testing** | `Chaos Mesh` / `LitmusChaos` | `chaos-mesh.org` | Kubernetes-native pod kill, network delay, packet drop |

---

## 🏷️ Fact Attribution Tags Standard

Every version-sensitive or empirical claim in the repository uses one of the following tags:
- `[Doc: <source>, checked <date>]`: Verified against official documentation on the specified date.
- `[Inference]`: Derived logically from system architecture principles.
- `[Approximation — verify before relying on this]`: Illustrative baseline or order-of-magnitude figure.
