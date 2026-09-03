# 02. The Telemetry Spectrum and the High-Cardinality Trap

## 1. Problem
Engineers add arbitrary labels (like `user_id`, `email`, or `order_id`) to Prometheus metrics, causing the time-series database (TSDB) to experience a **Cardinality Explosion**, consuming 100GB of RAM and crashing the entire monitoring infrastructure during an active incident.

## 2. Production Context
Telemetry consists of five complementary pillars, each optimized for distinct trade-offs in cost, latency, retention, and analytical granularity.

## 3. Mental Model: The 5 Telemetry Pillars

| Signal Pillar | What It Is | Strengths | Weaknesses | Ideal Use Case |
| :--- | :--- | :--- | :--- | :--- |
| **Metrics** | Numeric time-series values aggregated over fixed time intervals | Extreme query speed, cheap storage, ideal for alerting | Low dimensionality; cannot store raw IDs | High-level alerting (RED/USE), error rates |
| **Structured Logs** | Timestamped JSON events containing rich contextual payloads | High detail, searchable text, auditability | Expensive storage, slow querying across petabytes | Deep forensic debugging of individual errors |
| **Distributed Traces** | Graph of timed spans tracing a single request across services | Pinpoints cross-service bottlenecks and network latency | Expensive data volume without sampling | Isolating tail latency across microservices |
| **Continuous Profiles** | Periodic statistical sampling of CPU and heap memory allocations | Identifies exact lines of code and hot loops | Requires profiler agent overhead | Algorithmic optimization, memory leak tracing |
| **Audit Events** | Immutable records of critical business/system state changes | High compliance and audit durability | Not designed for high-frequency queries | Security auditing, schema migration history |

---

## 4. The High-Cardinality Explosion Formula

$$\mathbf{Total\ Time\text{-}Series} = \prod_{i=1}^{N} \text{Cardinality of Label } i$$

### Example:
- `http_requests_total`
- Labels:
  - `method`: 5 values (`GET`, `POST`, `PUT`, `DELETE`, `PATCH`)
  - `status`: 10 values (`200`, `201`, `400`, `401`, `403`, `404`, `500`, `502`, `503`, `504`)
  - `route`: 20 values (`/api/v1/users`, `/api/v1/orders`, etc.)
  - $\mathbf{\text{Total Series}} = 5 \times 10 \times 20 = \mathbf{1,000\ series}$ (Healthy!)

**The Fatal Mistake**:
- Adding `user_id` ($1,000,000$ active users) as a label:
  $$\mathbf{\text{Total Series}} = 5 \times 10 \times 20 \times 1,000,000 = \mathbf{1,000,000,000\ series} \implies \text{TSDB Crashes (OOM)!}$$

---

## 5. The Golden Rule of Cardinality
- **Metrics**: Strictly **Bounded Cardinality** (HTTP method, HTTP status class, service name, region, cluster).
- **Logs & Traces**: Unbounded Cardinality allowed (`user_id`, `trace_id`, `order_id`, `request_id`).

---

## 6. Interview Questions & Model Answers

**Q1: What is high cardinality in metrics, and why does it cause Prometheus or TSDB crashes?**
**Answer**: Cardinality in time-series monitoring refers to the number of unique combinations of metric label key-value pairs. Each unique combination generates a distinct, independent time series that the TSDB must index in memory and store on disk. When high-cardinality values (such as UUIDs, user IDs, or email addresses) are added as metric labels, the Cartesian product of label combinations explodes into millions or billions of active time series. This exhausts TSDB memory buffers, saturates the inverted index, causes heavy GC thrashing, and eventually crashes the monitoring server with an Out-of-Memory error.

**Q2: When should an engineer use a Metric vs a Structured Log vs a Distributed Trace?**
**Answer**: Use **Metrics** for real-time alerting, dashboarding, and tracking aggregate trends (e.g. 5xx error rate or p99 latency) because they are compact, cheap to store, and fast to evaluate in alert rules. Use **Distributed Traces** to understand request flow across distributed service boundaries and isolate which microservice or database hop caused a tail latency spike. Use **Structured Logs** for post-hoc forensic debugging of specific edge cases, inspecting detailed contextual payloads (error stack traces, input arguments, and tenant IDs) for an isolated transaction.
