# Capacity Planning & Headroom Guide

A structured engineering methodology for forecasting resource demand, identifying saturation points, and calculating production headroom without guesswork.

---

## 📈 The Capacity Planning Workflow

$$\mathbf{Traffic\ Forecast} \implies \mathbf{Resource\ Profiling} \implies \mathbf{Saturation\ Discovery} \implies \mathbf{Headroom\ Calculation}$$

---

## 📐 Core Formulas & Principles

### 1. Knee-of-the-Curve Saturation
As resource utilization exceeds 70–80%, queueing delays grow exponentially according to Kingman's formula ($V \times U \times T$).

### 2. Sizing Headroom
$$\text{Headroom Ratio} = \frac{\text{Maximum Proven Load (RPS)}}{\text{Current Peak Traffic (RPS)}}$$
- **Target**: Maintain at least **$1.5\times - 2.0\times$ headroom** for tier-1 user-facing services.
- **$N+1$ / $N+2$ Redundancy**: Ensure the cluster can survive the loss of an entire Availability Zone or host rack during peak traffic without exceeding $75\%$ CPU utilization.

### 3. Database Connection Pool Sizing
$$\text{Max Pool Connections} = (\text{CPU Cores} \times 2) + \text{Effective Spindle Count}$$
*Over-allocating database connection pools increases context switching and lock contention, degrading query throughput.*
