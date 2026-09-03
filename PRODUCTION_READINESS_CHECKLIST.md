# Production Readiness Review (PRR) Checklist

A rigorous operational gate to ensure services meet resilience, observability, security, and scalability standards before release.

---

## 🚦 PRR Evaluation Pillars

### 1. Architecture & Reliability
- [ ] Deployed across at least 3 Availability Zones (Multi-AZ).
- [ ] Liveness probe checks process health only; Readiness probe checks local dependency availability.
- [ ] Graceful shutdown configured with adequate termination grace period (e.g. 30s) to drain in-flight requests.
- [ ] Outbound dependencies have explicit connection, socket read, and total timeout limits.
- [ ] Retries use exponential backoff with full jitter and bounded retry counts.
- [ ] Critical features isolated from non-critical features via bulkheads or circuit breakers.

### 2. Scalability & Capacity
- [ ] Load tested to at least $2\times$ expected peak production traffic.
- [ ] Autoscaling (HPA) configured based on CPU, memory, or custom request queue metrics.
- [ ] Database connection pools sized to prevent database saturation ($(\text{cores} \times 2) + \text{spindle\_count}$).

### 3. Release & Change Safety
- [ ] Automated Canary deployment pipeline configured with automatic rollback on error spike.
- [ ] Feature flags in place for major architectural changes.
- [ ] Database schema migrations tested for backward compatibility (Expand / Contract pattern).

### 4. Operations & On-Call
- [ ] Production Runbooks authored and linked in alert notifications.
- [ ] Escalation path and on-call rotation configured in PagerDuty / Opsgenie.
- [ ] SLIs and SLOs defined and error budget alert rules active.
