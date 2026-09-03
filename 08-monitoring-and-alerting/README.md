# Module 08: Monitoring, Alerting & On-Call Fatigue Elimination

## Learning Objectives

By the end of this module, you will be able to:
- Architect an alerting philosophy centered on **actionability, precision, and zero on-call fatigue**.
- Distinguish **Symptom-Based Alerting** (alerting on user-visible pain) from **Cause-Based Alerting** (noisy internal symptoms).
- Refactor noisy, brittle threshold alerts into robust, multi-window SLO burn rate alerts.
- Configure escalation policies, deduplication, and suppression rules in Alertmanager / PagerDuty.

---

## Lessons in This Module

| File | Topic | Focus |
| :--- | :--- | :--- |
| [01-designing-actionable-alerts.md](01-designing-actionable-alerts.md) | Designing Actionable Alerts | The 4 golden signals (Latency, Traffic, Errors, Saturation), alert actionability |
| [02-symptom-vs-cause-based-alerting.md](02-symptom-vs-cause-based-alerting.md) | Symptom vs. Cause-Based Alerting | Paging on user pain, suppressing transient blips, alert routing |
| [03-bad-alert-to-improved-alert-refactoring.md](03-bad-alert-to-improved-alert-refactoring.md) | Bad Alert $\to$ Improved Alert Refactoring | 5 concrete production refactoring case studies |
