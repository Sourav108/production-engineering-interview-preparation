# Production Incident Response Checklist

A step-by-step checklist for on-call engineers and Incident Commanders navigating active production outages.

---

## 🚨 Incident Triage Protocol

### Phase 1: Detection & Triage (First 5 Minutes)
- [ ] Acknowledge PagerDuty page within SLA (e.g. 5 mins).
- [ ] Verify severity level (Sev-1: Full outage, Sev-2: Degraded major feature, Sev-3: Minor issue).
- [ ] Establish Incident Command: Designate Incident Commander (IC) and Scribe.
- [ ] Open dedicated incident Slack channel and war room call.

### Phase 2: Scope & Signals (Minutes 5–15)
- [ ] **Scope Impact**: What percentage of users or regions are failing? (5xx rate, p99 latency).
- [ ] **Inspect Signals**: Check service RED metrics, database metrics, and load balancer health.
- [ ] **Check Recent Changes**: Were there deployments, config pushes, or network updates in the last 60 minutes?

### Phase 3: Mitigation (Focus on Stopping User Pain)
- [ ] **Rollback**: If triggered after a deployment, immediately initiate automated rollback.
- [ ] **Traffic Shedding**: Shed low-priority background traffic or enable rate limiters.
- [ ] **Failover**: Reroute traffic away from degraded availability zone or region.
- [ ] **Circuit Breaking**: Trip circuit breaker on degraded third-party dependency.
- [ ] **DO NOT**: Perform blind restarts without capturing diagnostics (thread/heap dumps).

### Phase 4: Recovery & Communication
- [ ] Verify telemetry returns to baseline (error rate $< 0.1\%$, p99 latency normal).
- [ ] Send external customer status page update.
- [ ] Declare incident mitigated; close war room call.

### Phase 5: Learning & Postmortem (Within 48 Hours)
- [ ] Schedule blameless postmortem review.
- [ ] Populate timeline and root cause analysis in [templates/postmortem-template.md](templates/postmortem-template.md).
- [ ] Assign action items with owners and due dates.
