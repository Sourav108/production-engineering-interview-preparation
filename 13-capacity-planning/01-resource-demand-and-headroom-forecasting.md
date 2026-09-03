# 01. Resource Demand, Sizing, and Headroom Forecasting

## 1. Problem
Organizations either **over-provision** infrastructure by $500\%$ (wasting millions in cloud spend) or **under-provision** capacity, resulting in catastrophic crashes during Black Friday traffic surges.

## 2. Production Context
Capacity planning translates high-level business forecasts (e.g. "We expect 50,000 active checkout users per second during the flash sale") into concrete compute, memory, database, and network resource requirements.

## 3. Mental Model: The Capacity Equation
$$\mathbf{Total\ Cores\ Required} = \frac{\mathbf{Peak\ RPS} \times \mathbf{CPU\ Seconds\ per\ Request}}{\mathbf{Target\ Utilization\ Limit\ (e.g.\ 0.65)}} \times \mathbf{Redundancy\ Multiplier\ (N+1)}$$

### Step-by-Step Capacity Calculation Example:
1. **Business Metric**: $20,000\text{ RPS}$ peak checkout load.
2. **Benchmark Profile**: Each request requires $15\text{ms}$ of CPU time ($0.015\text{ CPU-seconds}$).
3. **Raw CPU Demand**: $20,000 \times 0.015 = \mathbf{300\ CPU\ Cores}$.
4. **Target Safe Utilization Limit**: $65\%$ ($0.65$) to avoid Kingman queueing knee.
   $$\text{Base Cores} = \frac{300}{0.65} \approx \mathbf{462\ Cores}$$
5. **Multi-AZ $N+1$ Headroom (3 Availability Zones)**:
   - Cluster must survive loss of 1 complete AZ ($33.3\%$ capacity loss).
   - $\text{Total Cores} = 462 \times 1.5 = \mathbf{693\ Cores}$ (or $\approx 87$ 8-core instances deployed as 29 instances per AZ).

---

## 4. Multi-AZ Redundancy Sizing Rules

```
                      AZ Failure Resiliency (3-AZ Deployment)
 ──────────────────────────────────────────────────────────────────────────────────────────
 Normal Operation (3 AZs Active)     ──► Each AZ operates at ~43% CPU utilization.
 1 AZ Fails (Network / Power Cut)    ──► Remaining 2 AZs absorb load at ~65% CPU utilization.
                                         (Zero queueing explosion, zero latency regression!)
```

---

## 5. Interview Questions & Model Answers

**Q1: How do you mathematically size a Kubernetes cluster for N+1 Availability Zone redundancy?**
**Answer**: For a 3-AZ deployment with $N+1$ redundancy, the cluster must be capable of sustaining 100% of peak traffic at safe CPU utilization ($\le 65\%-70\%$) even if an entire AZ drops offline. Since the remaining 2 AZs will carry all traffic, each surviving AZ must have $\frac{100\%}{2} = 50\%$ of the required peak compute capacity. Therefore, under normal 3-AZ operations, each AZ runs at only $\frac{100\%}{3} \approx 33.3\%$ total share, operating at $\approx 43\%$ CPU utilization. If one AZ collapses, the remaining 2 AZs naturally absorb the traffic without breaching the 65% knee-of-the-curve saturation threshold.
