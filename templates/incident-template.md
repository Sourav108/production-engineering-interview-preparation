# Incident Scenario Template

## 1. Symptoms
[User-facing symptoms, alert fires, or error rate spikes.]

## 2. Impact
[Quantified blast radius: affected users, failed requests, revenue or latency impact.]

## 3. Initial Signals
[Key dashboards, alert notifications, and initial telemetry.]

## 4. Timeline
- **T0 (00:00)**: Event triggered.
- **T1 (+05m)**: Alert fired.
- **T2 (+10m)**: On-call acknowledged, investigation began.
- **T3 (+25m)**: Mitigation applied.
- **T4 (+35m)**: Recovery verified.

## 5. Hypotheses
1. [Hypothesis A] - Tested & Rejected
2. [Hypothesis B] - Confirmed

## 6. Commands & Diagnostic Queries
```bash
# Example diagnostic command
kubectl top pods -n production
```

## 7. Metrics
[Specific PromQL or metric timeseries observations.]

## 8. Logs
[Relevant structured log snippets demonstrating the failure.]

## 9. Traces
[Span details and trace IDs illustrating bottleneck or error.]

## 10. Root Cause
[One-sentence precise technical root cause.]

## 11. Mitigation
[Immediate actions taken to stop customer impact.]

## 12. Recovery
[Step-by-step restoration of normal system state.]

## 13. Verification
[Evidence confirming full recovery.]

## 14. Permanent Fix
[Code, infrastructure, or configuration fix to eradicate the bug.]

## 15. Prevention
[Guardrails, architectural changes, or monitors to prevent recurrence.]

## 16. Interview Discussion
[How to discuss this incident in a production engineering interview.]
