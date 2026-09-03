# 03. Bad Alert $\to$ Improved Alert: 5 Concrete Refactoring Case Studies

## 1. Problem
Legacy alerting configurations are filled with brittle threshold alerts that cause false pages or fail to trigger during real outages.

---

## 🛠️ The 5 Production Alert Refactoring Exercises

### Case 1: The CPU Spike Alert
- ❌ **Bad Alert**: `100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[1m])) * 100) > 85`
  - *Why it's bad*: High CPU during batch jobs is normal; triggers pages at 2 AM when user latency is completely unaffected.
- ✅ **Improved Alert**: Multi-window SLO burn rate on p99 latency:
  ```promql
  histogram_quantile(0.99, sum(rate(http_request_duration_seconds_bucket[5m])) by (le)) > 1.5
  and
  histogram_quantile(0.99, sum(rate(http_request_duration_seconds_bucket[30m])) by (le)) > 1.2
  ```

---

### Case 2: The Memory Utilization Alert
- ❌ **Bad Alert**: `node_memory_MemFree_bytes / node_memory_MemTotal_bytes < 0.10`
  - *Why it's bad*: Linux naturally uses free RAM for Page Cache; `MemFree` is almost always $<10\%$.
- ✅ **Improved Alert**: Monitor Pressure Stall Information (PSI) or Kubernetes container OOM score:
  ```promql
  rate(node_pressure_memory_waiting_seconds_total[5m]) > 0.20
  ```

---

### Case 3: The Single-Instance Down Alert
- ❌ **Bad Alert**: `up{job="api"} == 0` (Immediate Page)
  - *Why it's bad*: During rolling deployments or node spot terminations, individual pods restart normally; creates 20 pages per deploy.
- ✅ **Improved Alert**: Alert on **Cluster Quorum / Target Group Health**:
  ```promql
  (sum(up{job="api"}) / count(up{job="api"})) < 0.70
  ```
  *(Pages only if $>30\%$ of fleet is simultaneously down).*

---

### Case 4: The Static Error Count Alert
- ❌ **Bad Alert**: `sum(rate(http_requests_total{status="500"}[1m])) > 10`
  - *Why it's bad*: 10 errors at 100,000 QPS is a $0.01\%$ error rate (healthy); 10 errors at 10 QPS is a $100\%$ outage (catastrophic).
- ✅ **Improved Alert**: **Error Percentage Ratio**:
  ```promql
  (sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m]))) > 0.01
  ```

---

### Case 5: The Disk Space Threshold Alert
- ❌ **Bad Alert**: `node_filesystem_free_bytes / node_filesystem_size_bytes < 0.15`
  - *Why it's bad*: On a 20TB disk, $15\%$ is 3TB of free space (weeks of headroom), waking engineers unnecessarily.
- ✅ **Improved Alert**: **Predictive Linear Disk Exhaustion**:
  ```promql
  predict_linear(node_filesystem_free_bytes[4h], 86400 * 2) < 0
  ```
  *(Pages only if disk is projected to fill to $100\%$ within the next 48 hours based on the 4-hour fill rate).*

---

## 2. Interview Questions & Model Answers

**Q1: How does `predict_linear()` in PromQL improve disk space alerting?**
**Answer**: Static percentage alerts (e.g. `Disk < 10%`) fail at both extremes: on small disks they trigger too late, and on multi-terabyte volumes they page engineers when gigabytes of headroom remain. `predict_linear()` uses linear regression over a moving historical window (e.g. 4 hours) to calculate the derivative (fill rate) and project the exact time until the disk reaches zero bytes. This ensures that engineers are only notified when a disk is genuinely on a trajectory to fill up within a specific operational window (e.g. 24 or 48 hours), eliminating false alarms for stable, high-fill partitions.
