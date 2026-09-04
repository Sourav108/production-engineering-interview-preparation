# Incident Scenario: VPC Peering MTU Blackhole Causing Intermittent TCP Drops

**Category**: Packet Loss | **Severity**: Sev-1 | **Target Level**: Staff / Senior Production Engineer

---

## 1. Symptoms
High customer error rates observed on public endpoints; HTTP 5xx errors spiked to 8.4% and p99 latency exceeded 4,000ms.

## 2. Impact
Approximately 14,000 active checkout users impacted; customer transactions stalled for 24 minutes; revenue degradation estimated at ,000.

## 3. Initial Signals
- PagerDuty Sev-1 Alert fired:  on checkout service.
- Grafana RED dashboard showing p99 latency spike and elevated error rate.

## 4. Timeline (UTC)
- **14:00**: Event triggered under load surge.
- **14:02**: Prometheus multi-window burn rate alert fired.
- **14:05**: On-call engineer acknowledged page and established Incident Command.
- **14:14**: Diagnostic investigation isolated root cause (packet loss).
- **14:20**: Targeted operational mitigation applied.
- **14:26**: Telemetry verified normal; incident declared mitigated.

## 5. Hypotheses
1. *Hypothesis A (Network Outage)*: Tested via ping/traceroute $	o$ Rejected (Network latency normal).
2. *Hypothesis B (Packet Loss)*: Tested via telemetry queries $	o$ **CONFIRMED**.

## 6. Commands & Diagnostic Queries


## 7. Metrics
- : Spiked from 0.01% to 8.4%.
- : Rose from 45ms to 4,200ms.

## 8. Logs


## 9. Traces
Distributed trace spans in Jaeger show a 4,100ms bottleneck span on the affected resource hop.

## 10. Root Cause
Resource contention in packet loss crossed non-linear saturation thresholds under peak concurrency, queueing in-flight worker threads.

## 11. Mitigation
Applied immediate operational remediation (rate limiting, circuit breaker trip, or traffic failover) to shed load and stabilize throughput.

## 12. Recovery
Scaled cluster capacity and cleared in-flight backlogs; verified error rate dropped to $< 0.01\%$.

## 13. Verification
Confirmed p99 latency returned to baseline (35ms) across all three Availability Zones.

## 14. Permanent Fix
Refactored connection pooling, added automated guardrails, and updated Kubernetes resource limits.

## 15. Prevention
Added predictive linear threshold alerting in Prometheus and incorporated this failure mode into weekly Chaos GameDays.

## 16. Interview Discussion
How to discuss this incident in a Staff Production Engineering interview: focus on rapid hypothesis elimination without blind restarts, evidence collection, and permanent architectural guardrails.
