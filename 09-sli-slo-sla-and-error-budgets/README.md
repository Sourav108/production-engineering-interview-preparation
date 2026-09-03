# Module 09: SLIs, SLOs, SLAs & Error Budget Governance

## Learning Objectives

By the end of this module, you will be able to:
- Formulate precise, user-centric **Service Level Indicators (SLIs)** across Availability, Latency, Freshness, and Correctness.
- Derive mathematically grounded **Service Level Objectives (SLOs)** and contractual **Service Level Agreements (SLAs)** based on historical user pain thresholds.
- Enforce strict **Error Budget Policies** to govern release gating and balance development velocity with system reliability.
- Configure Google SRE **Multi-Window Multi-Burn-Rate Alerting Rules** in Prometheus.

---

## Lessons in This Module

| File | Topic | Focus |
| :--- | :--- | :--- |
| [01-defining-meaningful-slis.md](01-defining-meaningful-slis.md) | Defining Meaningful SLIs | User journeys, good vs valid events, availability/latency/freshness SLIs |
| [02-deriving-slos-and-error-budget-policies.md](02-deriving-slos-and-error-budget-policies.md) | Deriving SLOs & Error Budget Policies | Calculating allowed downtime minutes, feature freeze rules, SLA buffers |
| [03-multi-window-multi-burn-rate-alerting.md](03-multi-window-multi-burn-rate-alerting.md) | Multi-Window Multi-Burn-Rate Alerting | 14.4x, 6x, 1x burn rate math, short vs long window PromQL alert rules |
