# 01. Evidence-Driven Production Debugging: The 12-Step Protocol

## 1. Problem
When a complex distributed outage occurs, engineers often succumb to "guessing games"—randomly editing configuration parameters, bouncing servers, or clearing caches—which muddies logs, destroys forensic traces, and prolongs downtime.

## 2. Production Context
Production debugging is an empirical science. Every action taken in production must be guided by testable hypotheses and verifiable telemetry evidence.

## 3. Mental Model: The 12-Step Production Debugging Sequence
$$\begin{aligned}
\mathbf{1.\ Symptom} &\implies \text{Define exact user pain (e.g. 504 Gateway Timeout on /checkout)} \\
\mathbf{2.\ Scope} &\implies \text{Determine blast radius (Is it all users, one region, or one tenant?)} \\
\mathbf{3.\ Metrics} &\implies \text{Inspect RED metrics (Rate, Errors, Duration) across services} \\
\mathbf{4.\ Logs} &\implies \text{Query structured error logs for specific exception signatures} \\
\mathbf{5.\ Traces} &\implies \text{Inspect slowest spans to pinpoint the exact lagging hop} \\
\mathbf{6.\ Recent\ Changes} &\implies \text{Audit deployments, config pushes, and flag toggles in last 2 hours} \\
\mathbf{7.\ Dependencies} &\implies \text{Check database IOPS, cache hit ratios, and 3rd-party vendor health} \\
\mathbf{8.\ Hypothesis} &\implies \text{Formulate 2-3 specific, testable differential root causes} \\
\mathbf{9.\ Test} &\implies \text{Validate or reject hypotheses via non-destructive telemetry queries} \\
\mathbf{10.\ Root\ Cause} &\implies \text{Confirm precise mechanism (e.g. unindexed foreign key lock)} \\
\mathbf{11.\ Mitigation} &\implies \text{Apply targeted operational remediation (e.g. rate limit / rollback)} \\
\mathbf{12.\ Verification} &\implies \text{Prove telemetry and error budget recovery on production dashboards}
\end{aligned}$$

---

## 4. The Anti-Pattern: The Blind Restart

```
                               The Blind Restart Anti-Pattern
 ──────────────────────────────────────────────────────────────────────────────────────────
 1. Latency spikes on Service A → On-call engineer restarts container pods.
 2. In-memory thread dumps and heap forensics are permanently destroyed!
 3. Restarted pods connect to DB simultaneously → DB connection pool saturates.
 4. Outage worsens, and the underlying root cause remains unidentified.
```

**Rule**: Always capture diagnostic artifacts (`jcmd Thread.print`, `jcmd GC.heap_dump`, `netstat -s`, `free -m`) before initiating any container restart!

---

## 5. Interview Questions & Model Answers

**Q1: Walk me through your step-by-step diagnostic workflow when p99 latency spikes on a critical API.**
**Answer**: I follow a strict evidence-driven protocol:
1. **Define the Symptom and Scope**: Check whether the latency spike affects all routes or a specific endpoint, and whether it is localized to a single Availability Zone or customer segment.
2. **Decompose via Tracing**: Pull the slowest trace spans in OpenTelemetry/Jaeger to determine whether time is spent in network transit, application CPU processing, cache lookup, or database query execution.
3. **Correlate with Recent Changes**: Audit CI/CD deployment history and feature flag updates within the last 60 minutes.
4. **Inspect Resource Saturation**: Check host CPU, memory pressure stall information (PSI), thread pool queue depth, and database connection pool saturation.
5. **Formulate and Test Hypotheses**: If traces show database queries are slow, check `pg_stat_activity` for lock contention rather than guessing.
6. **Mitigate and Verify**: Apply targeted mitigation (e.g. circuit breaking or rolling back) and verify tail latency returns below the SLO threshold.
