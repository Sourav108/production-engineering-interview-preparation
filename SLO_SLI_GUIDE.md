# Service Level Objectives (SLOs), SLIs & Error Budgets Guide

A practical guide to connecting user pain to engineering decisions using Service Level Indicators (SLIs), Objectives (SLOs), Agreements (SLAs), and Error Budget policies.

---

## 🎯 The Hierarchy of Reliability Targets

$$\mathbf{SLI} \implies \mathbf{SLO} \implies \mathbf{Error\ Budget} \implies \mathbf{SLA}$$

- **Service Level Indicator (SLI)**: A quantifiable metric measuring service performance (e.g. $\frac{\text{Good Requests}}{\text{Total Valid Requests}} \times 100\%$).
- **Service Level Objective (SLO)**: The target reliability percentage agreed upon internally by engineering and product (e.g. $99.9\%$ over a 30-day rolling window).
- **Error Budget**: The allowable unreliability ($100\% - \text{SLO}$). For a $99.9\%$ SLO, the error budget is $0.1\%$ ($43.2$ minutes of downtime per 30 days).
- **Service Level Agreement (SLA)**: The contractual commitment to customers with financial penalties if breached (typically looser than the internal SLO, e.g. $99.5\%$).

---

## 📊 Standard SLI Calculation Patterns

### 1. Availability SLI
$$\text{Availability SLI} = \frac{\sum \text{HTTP status } \in \{200..499\}}{\sum \text{Total HTTP requests}} \times 100\%$$

### 2. Latency SLI
$$\text{Latency SLI} = \frac{\sum \text{Requests completed in } \le 200\text{ms}}{\sum \text{Total valid HTTP requests}} \times 100\%$$

---

## 🔥 Multi-Window Multi-Burn-Rate Alerting

Instead of alerting when the error budget is fully exhausted, alert based on how rapidly the budget is burning:

| Burn Rate | % Budget Consumed | Time to 100% Budget Exhaustion | Severity / Action |
| :-: | :-: | :-: | :--- |
| **$14.4\times$** | $2\%$ in 1 hour | **2 days** | **Page On-Call (Sev-1)** |
| **$6\times$** | $5\%$ in 6 hours | **5 days** | **Page On-Call (Sev-2)** |
| **$1\times$** | $10\%$ in 3 days | **30 days** | **Ticket / Slack Notification** |
