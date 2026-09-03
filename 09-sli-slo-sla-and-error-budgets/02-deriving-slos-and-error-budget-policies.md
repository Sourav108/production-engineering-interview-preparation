# 02. Deriving SLOs, Error Budgets, and Governance Policies

## 1. Problem
Product and engineering leaders debate arbitrary availability targets (e.g. "We need 100% uptime!"), leading to paralyzed development velocity or unrealistic architectural over-engineering.

## 2. Production Context
$100\%$ availability is the wrong target for almost everything: the user's mobile cellular carrier or home ISP typically achieves only $99.0\% - 99.5\%$ availability. Any reliability engineering beyond what the user's network can perceive is wasted capital.

## 3. Mental Model
$$\mathbf{Error\ Budget} = 100\% - \mathbf{SLO}$$
The Error Budget is a quantifiable allowance of permissible unreliability over a rolling time window (e.g. 30 days) to be spent on **innovation, risk-taking, canary deployments, and infrastructure upgrades**.

---

## 4. The 9s of Availability Downtime Reference Table (30-Day Window)

| SLO Target ("9s") | Monthly Allowed Downtime | Weekly Allowed Downtime | Daily Allowed Downtime |
| :--- | :--- | :--- | :--- |
| **$99\%$ ("Two 9s")** | **7 hours, 18 minutes** | 1 hour, 40 minutes | 14.4 minutes |
| **$99.9\%$ ("Three 9s")** | **43.2 minutes** | 10.1 minutes | 1.44 minutes |
| **$99.95\%$** | **21.6 minutes** | 5.0 minutes | 43.2 seconds |
| **$99.99\%$ ("Four 9s")** | **4.32 minutes** | 1.01 minutes | 8.64 seconds |
| **$99.999\%$ ("Five 9s")** | **25.9 seconds** | 6.0 seconds | 0.86 seconds |

---

## 5. The Error Budget Policy Contract

When an error budget is burned, automated governance rules take effect:

```
                            Error Budget Policy Lifecycle
 ──────────────────────────────────────────────────────────────────────────────────────────
 Budget Remaining > 50%  ──► Standard Operations: Normal feature releases, risk-taking.
 Budget Remaining < 20%  ──► Yellow State: Non-critical canary deploys paused; PRR required.
 Budget Exhausted (0%)   ──► RED STATE: HARD FREEZE on all feature deployments.
                             100% of engineering bandwidth redirected to reliability,
                             performance fixes, architectural hardening, and bug fixes.
```

---

## 6. The SLA vs. SLO Buffer Rule
$$\mathbf{SLA\ Target} < \mathbf{SLO\ Target}$$
- **Internal SLO**: $99.9\%$ (Target for engineering on-call and release gating).
- **External SLA**: $99.5\%$ (Contractual commitment with financial penalties).
- **The Buffer ($0.4\%$)**: Gives engineering room to detect and mitigate incidents before incurring financial customer refunds.

---

## 7. Interview Questions & Model Answers

**Q1: Why is aiming for 100% availability an anti-pattern in modern software engineering?**
**Answer**: Aiming for 100% availability is counterproductive for three core reasons:
1. **Exponential Cost**: Moving from $99.9\%$ to $99.999\%$ increases infrastructure and operational costs by orders of magnitude (multi-region active-active consensus, continuous disaster recovery drills) with diminishing returns.
2. **Zero Feature Velocity**: 100% uptime requires never changing code or releasing new features, destroying business competitiveness.
3. **ISP Bottleneck**: End-user client devices and mobile networks rarely exceed $99.0\% - 99.5\%$ reliability. An extra nine of server availability is undetectable to a user whose cell connection drops periodically.

**Q2: What happens when an engineering team exhausts its 30-day error budget?**
**Answer**: When an error budget is exhausted ($0\%$ remaining), the **Error Budget Policy** is invoked. Feature deployments are temporarily frozen, and engineering capacity is redirected exclusively to reliability work: fixing recurring bugs, optimizing database queries, adding automated fallbacks, and improving observability. Deployments resume once reliability stabilizes and the rolling error budget recovers.
