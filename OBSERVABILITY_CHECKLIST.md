# Production Observability Checklist

A comprehensive checklist to verify that a service is fully observable before deploying to production.

---

## 📋 Pre-Production Observability Gates

### 1. Metrics (RED & USE Methods)
- [ ] **Rate**: Inbound request throughput (QPS/RPS) tagged by route, HTTP method, and response status.
- [ ] **Errors**: Count and percentage of HTTP 5xx and 4xx responses.
- [ ] **Duration**: Latency recorded as explicit histogram buckets with accurate p50, p90, p95, p99, and p99.9 percentile tracking.
- [ ] **Utilization**: Host CPU, JVM heap/non-heap memory, disk I/O, network bandwidth.
- [ ] **Saturation**: Thread pool queue depth, HikariCP connection wait count, Linux PSI (Pressure Stall Information).
- [ ] **Errors (System)**: GC pause duration, OOM kills, network socket packet drops.

### 2. Structured Logging
- [ ] Logs emitted in structured JSON format to stdout/stderr.
- [ ] Standard fields present: `timestamp`, `level`, `service`, `environment`, `trace_id`, `span_id`, `user_id`.
- [ ] Sensitive data (passwords, credit cards, PII) masked or sanitized before emission.
- [ ] High-cardinality unbounded keys prevented in structured tags.

### 3. Distributed Tracing (OpenTelemetry)
- [ ] Trace context propagated across all HTTP, gRPC, and messaging queue headers (`W3C TraceContext`).
- [ ] Database queries, cache calls, and outbound external HTTP requests wrapped in spans.
- [ ] Span errors recorded with exception message and stack trace.

### 4. Dashboards & Alerts
- [ ] Service overview dashboard available with RED metrics at a glance.
- [ ] Every alert linked to an actionable production runbook.
