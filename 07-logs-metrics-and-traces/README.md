# Module 07: Logs, Metrics & Distributed Traces

## Learning Objectives

By the end of this module, you will be able to:
- Implement production-grade **Structured JSON Logging** with MDC context propagation and PII data sanitization.
- Master metric data types: **Counters, Gauges, Summaries, and Histograms (including Native Histograms)** in Prometheus and Micrometer.
- Explain mathematically why **Average Latency is a Dangerous Lie** and compute true histogram-based percentiles ($p50, p95, p99, p99.9$).
- Architect distributed trace context propagation using the **W3C TraceContext** and OpenTelemetry standard.

---

## Lessons in This Module

| File | Topic | Focus |
| :--- | :--- | :--- |
| [01-structured-logging-and-correlation.md](01-structured-logging-and-correlation.md) | Structured Logging & Correlation IDs | JSON formatting, MDC context propagation, sensitive data masking |
| [02-metrics-types-and-percentiles.md](02-metrics-types-and-percentiles.md) | Metric Types & Tail Percentiles | Counters, Gauges, Histograms, why averages hide outages, PromQL |
| [03-opentelemetry-and-distributed-tracing.md](03-opentelemetry-and-distributed-tracing.md) | OpenTelemetry & Distributed Tracing | W3C TraceContext headers, span lifecycles, tail-based sampling |
