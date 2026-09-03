# 03. Multi-Window Multi-Burn-Rate Alerting Architecture

## 1. Problem
Alerting on a single static threshold over a single time window creates an impossible dilemma: a short window (e.g. 5 minutes) produces false alarms on transient spikes, while a long window (e.g. 24 hours) takes hours to alert during a catastrophic $100\%$ outage.

## 2. Production Context
Google SRE developed **Multi-Window Multi-Burn-Rate Alerting** to mathematically solve this trade-off: alert based on the **Burn Rate** of the error budget evaluated across concurrent short and long lookback windows.

## 3. Mental Model
$$\mathbf{Burn\ Rate\ (BR)} = \frac{\text{Observed Error Rate}}{\text{Allowed Error Rate for SLO}}$$
- $\mathbf{BR = 1.0}$: The service will consume exactly $100\%$ of its error budget over the 30-day window.
- $\mathbf{BR = 14.4}$: The service will consume **100% of its monthly error budget in 2 days** ($2\%$ of budget consumed in 1 hour). $\implies$ **PAGE ON-CALL (Sev-1)**!
- $\mathbf{BR = 6.0}$: The service will consume **100% of its budget in 5 days** ($5\%$ consumed in 6 hours). $\implies$ **PAGE ON-CALL (Sev-2)**!

---

## 4. Multi-Window Alerting Truth Table (Google SRE Standard)

| Severity | Burn Rate | % Budget Consumed | Long Window | Short Window (1/12th) | Notification Channel |
| :-: | :-: | :-: | :-: | :-: | :--- |
| **Sev-1 (Critical)** | **$14.4\times$** | $2\%$ | **1 hour** | **5 minutes** | PagerDuty (Phone/SMS) |
| **Sev-2 (Major)** | **$6.0\times$** | $5\%$ | **6 hours** | **30 minutes** | PagerDuty (Push) |
| **Sev-3 (Warning)** | **$1.0\times$** | $10\%$ | **3 days** | **6 hours** | Slack Channel |

$$\mathbf{Rule}: \text{Alert triggers ONLY when BOTH the Long Window AND the Short Window exceed the Burn Rate threshold!}$$
*(This eliminates false pages from transient 30-second spikes while catching persistent fast burns in minutes).*

---

## 5. Production PromQL Multi-Window Alert Rule Example

```yaml
groups:
  - name: slo_burn_rate_alerts
    rules:
      # Sev-1: 14.4x Burn Rate Alert (Consumes 2% budget in 1 hour)
      - alert: CheckoutErrorBudgetFastBurn
        expr: |
          (
            job:http_request_error_rate:ratio_rate1h{job="checkout"} > (14.4 * (1 - 0.999))
            and
            job:http_request_error_rate:ratio_rate5m{job="checkout"} > (14.4 * (1 - 0.999))
          )
        labels:
          severity: critical
        annotations:
          summary: "Checkout SLO Error Budget burning at 14.4x rate"
          description: "Service has consumed 2% of its monthly error budget in the last hour."
          runbook_url: "https://wiki.example.com/runbooks/checkout-fast-burn"
```

---

## 6. Interview Questions & Model Answers

**Q1: Why are two lookback windows (a long window and a short window) required in SRE burn rate alerting?**
**Answer**: A single long window (e.g. 1 hour) exhibits **Alert Reset Delay**: if an error spike lasts for 10 minutes and then completely stops, a 1-hour average will continue to evaluate above the threshold for the next 50 minutes, keeping an alert firing long after the incident has self-healed. Requiring that a short window (e.g. 5 minutes) also exceeds the threshold ensures that the moment the incident resolves, the short window drops below the threshold and immediately resolves the alert, eliminating false lingering pages.

**Q2: How do you calculate the burn rate threshold for a 99.9% SLO to detect a 2% budget burn in 1 hour?**
**Answer**: For a 30-day rolling window ($30 \times 24 = 720\text{ hours}$), 1 hour represents $\frac{1}{720}$ of the total time. If we wish to detect an event that consumes $2\%$ ($0.02$) of the budget in 1 hour, the required burn rate is:
$$\text{Burn Rate} = \frac{0.02}{1 / 720} = 0.02 \times 720 = \mathbf{14.4}$$
Therefore, alerting when the error rate exceeds $14.4 \times (1 - 0.999) = 14.4 \times 0.001 = 0.0144$ ($1.44\%$ error rate) over a 1-hour window precisely catches a 2% budget burn.
