# 02. Metric Types, Histograms, and Why Averages Hide Outages

## 1. Problem
An on-call engineer looks at a dashboard showing an **Average Latency of 45ms** and reports that the system is completely healthy, while $1\%$ of high-value checkout customers are experiencing **30-second timeouts** and abandoning their purchases.

## 2. Production Context
Averages (Arithmetic Mean) mathematically dilute extreme outliers. In distributed microservices where a single user request calls 50 downstream services in parallel, the probability of hitting a tail outlier approaches $100\%$ ($1 - (1 - 0.01)^{50} \approx 39.5\%$).

## 3. Mental Model: Why Average Latency is a Dangerous Lie

Consider 100 requests:
- 99 requests take **10ms**.
- 1 request hangs for **10,000ms** (10 seconds).
$$\mathbf{Average\ Latency} = \frac{(99 \times 10) + 10000}{100} = \frac{10990}{100} = \mathbf{109.9\text{ms}}$$
*An average of 109ms looks completely acceptable, completely masking the fact that a user suffered a catastrophic 10-second freeze!*

$$\mathbf{p99\ Latency} = \mathbf{10,000\text{ms}}\ \ (\text{Exposes the real user pain!})$$

---

## 4. The 4 Prometheus Metric Types

| Type | Semantics | Behavior | Example Use Case |
| :--- | :--- | :--- | :--- |
| **Counter** | Monotonically increasing number (can only increment or reset to 0) | Rate queries via `rate()` | `http_requests_total`, `packet_errors_total` |
| **Gauge** | Value that can arbitrarily go up or down | Instantaneous point-in-time value | `jvm_memory_used_bytes`, `active_threads`, `cpu_temp` |
| **Histogram** | Samples observations into configurable cumulative buckets | Enables aggregation across nodes via `histogram_quantile()` | `http_request_duration_seconds_bucket` |
| **Summary** | Computes sliding percentiles on client side | Cannot be aggregated across multiple server instances | Client-side standalone latency metrics |

---

## 5. Calculating Tail Latency with PromQL

```promql
# Calculate p99 latency across all pods in production
histogram_quantile(0.99, sum(rate(http_request_duration_seconds_bucket{environment="production"}[5m])) by (le))

# Calculate error rate percentage
(sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m]))) * 100
```

---

## 6. Interview Questions & Model Answers

**Q1: Why can you NOT average p99 latency metrics across multiple servers?**
**Answer**: Percentiles are non-linear, non-additive order statistics. If Server A has a p99 of 100ms and Server B has a p99 of 500ms, the mathematical average $\frac{100 + 500}{2} = 300\text{ms}$ is statistically meaningless because it ignores the distribution and sample counts of requests served by each node. To compute a valid global p99 across a cluster, each node must emit **cumulative histogram buckets** (counts of requests $\le 10\text{ms}, \le 50\text{ms}, \le 200\text{ms}$, etc.). Prometheus then sums the individual bucket counts across all nodes and interpolates the true global 99th percentile using `histogram_quantile()`.

**Q2: What is the difference between `rate()` and `irate()` in PromQL?**
**Answer**: `rate()` calculates the per-second average rate of increase of a counter over a specified time window (e.g. `[5m]`), smoothing out transient spikes and making it ideal for stable alerting and SLO evaluation. `irate()` calculates the instant per-second rate of increase using only the **last two data points** within the range window, capturing rapid, high-frequency spikes and making it ideal for real-time interactive troubleshooting on dashboards.
