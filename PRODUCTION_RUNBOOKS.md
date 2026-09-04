# Master Production Runbooks

The centralized operational runbook index for on-call engineers. Every runbook follows the strict 10-point standard: **Symptoms $\to$ Immediate Safety Checks $\to$ Evidence Collection $\to$ Diagnosis $\to$ Mitigation $\to$ Recovery $\to$ Verification $\to$ Escalation $\to$ Permanent Fix $\to$ Prevention**.

---

## 📑 Index of Production Runbooks

1. [High CPU Saturation Runbook](#1-high-cpu-saturation-runbook)
2. [High JVM Memory / OOM Kill Runbook](#2-high-jvm-memory--oom-kill-runbook)
3. [Database Connection Pool Exhaustion Runbook](#3-database-connection-pool-exhaustion-runbook)
4. [High API Tail Latency Runbook](#4-high-api-tail-latency-runbook)
5. [High 5xx Error Rate Runbook](#5-high-5xx-error-rate-runbook)
6. [Downstream Dependency Timeout Runbook](#6-downstream-dependency-timeout-runbook)
7. [Kafka Queue Consumer Backlog Runbook](#7-kafka-queue-consumer-backlog-runbook)
8. [Redis Cache Outage / Eviction Runbook](#8-redis-cache-outage--eviction-runbook)
9. [DNS Resolution Failure Runbook](#9-dns-resolution-failure-runbook)
10. [TLS Certificate Expiration Runbook](#10-tls-certificate-expiration-runbook)
11. [Disk Space & Inode Exhaustion Runbook](#11-disk-space--inode-exhaustion-runbook)
12. [Kubernetes Node NotReady Failure Runbook](#12-kubernetes-node-notready-failure-runbook)
13. [Deployment Regression & Rollback Runbook](#13-deployment-regression--rollback-runbook)
14. [Cloud Region Outage & Failover Runbook](#14-cloud-region-outage--failover-runbook)
15. [Database Replication Lag Runbook](#15-database-replication-lag-runbook)

---

## 1. High CPU Saturation Runbook
- **Symptoms**: Host or pod CPU utilization exceeds $90\%$; thread queue depth increasing; p99 latency spiking.
- **Immediate Safety Checks**: Verify whether database CPU is also high. Do NOT restart pods immediately without capturing a thread dump.
- **Evidence Collection**:
  ```bash
  # 1. Identify high-CPU threads
  top -H -p <pid>
  # 2. Capture Java thread dump
  jcmd <pid> Thread.print > /tmp/threaddump_$(date +%s).txt
  ```
- **Diagnosis**: Convert high-CPU native thread ID to hex (`printf "0x%x\n" <tid>`) and locate `nid=0x...` in thread dump to pinpoint hot loop or ReDoS regex.
- **Mitigation**:
  1. Scale deployment replicas horizontally: `kubectl scale deployment/api --replicas=40`.
  2. Enable adaptive rate limiting at API gateway to shed 20% of traffic.
- **Recovery**: Drain traffic from saturated pods; verify CPU stabilizes $< 65\%$.
- **Verification**: Confirm `container_cpu_usage_seconds_total` returns to normal baseline.
- **Escalation**: Escalate to Core Platform Team if CPU saturation is caused by kernel CFS throttling.
- **Permanent Fix**: Optimize algorithmic complexity or refactor regex to linear execution.
- **Prevention**: Add CPU profiling benchmark in load-testing CI gate.

---

## 2. High JVM Memory / OOM Kill Runbook
- **Symptoms**: Kubernetes pod restarts with `Exit Code 137` (OOMKilled); memory slope monotonically climbing.
- **Immediate Safety Checks**: Verify JVM heap dump flag is enabled (`-XX:+HeapDumpOnOutOfMemoryError`).
- **Evidence Collection**:
  ```bash
  dmesg -T | grep -i "killed process"
  jcmd <pid> VM.native_memory detail.diff
  ```
- **Diagnosis**: Inspect `.hprof` heap dump using Eclipse Memory Analyzer (MAT) to identify leaking GC Root.
- **Mitigation**:
  1. Temporarily increase container memory limit: `kubectl set resources deployment/api --limits=memory=8Gi`.
  2. Restart leaking pods in rolling batches ($10\%$ at a time).
- **Recovery**: Deploy memory leak patch.
- **Verification**: Monitor memory retention over 24 hours to confirm stable plateau.
- **Escalation**: Escalate to Application Engineering Owner.
- **Permanent Fix**: Wrap unclosed Netty ByteBuffers in `try-with-resources`.
- **Prevention**: Enforce 4-hour soak tests in pre-production staging.

---

## 3. Database Connection Pool Exhaustion Runbook
- **Symptoms**: Applications log `ConnectionTimeoutException: Connection is not available, request timed out after 30000ms`.
- **Immediate Safety Checks**: Do NOT increase application connection pool size without checking DB server CPU.
- **Evidence Collection**:
  ```sql
  SELECT pid, now() - xact_start AS duration, state, query
  FROM pg_stat_activity WHERE state != 'idle' ORDER BY duration DESC;
  ```
- **Diagnosis**: Identify blocking queries holding table locks or uncommitted `idle in transaction` sessions.
- **Mitigation**:
  1. Terminate long-running blocking query: `SELECT pg_terminate_backend(<pid>);`.
  2. Restart application pods with leaked connections.
- **Recovery**: Connection wait time drops to $0\text{ms}$; throughput recovers.
- **Verification**: Confirm `HikariPool-1 - Active connections` drops below $80\%$ pool capacity.
- **Escalation**: Escalate to Lead DBA on-call.
- **Permanent Fix**: Enforce `SET lock_timeout = '3s'` and `SET statement_timeout = '15s'` globally.
- **Prevention**: Size connection pools strictly using $(\text{cores} \times 2) + \text{spindles}$ formula.

---

## 4. Downstream Dependency Timeout Runbook
- **Symptoms**: Outbound HTTP calls returning HTTP 504 Gateway Timeout; thread pools filling up.
- **Immediate Safety Checks**: Identify which specific downstream service or third-party vendor is lagging.
- **Evidence Collection**: Inspect OpenTelemetry distributed trace spans for slowest outbound RPC hops.
- **Diagnosis**: Vendor status page confirms third-party API outage or degraded latency.
- **Mitigation**:
  1. Trip Circuit Breaker manually to `OPEN` state to fail-fast immediately in $< 0.1\text{ms}$.
  2. Enable Graceful Fallback: Serve cached stale data or default mock payload.
- **Recovery**: When downstream recovers, transition Circuit Breaker to `HALF_OPEN` with 10 trial requests.
- **Verification**: Confirm upstream p99 latency drops below 200ms.
- **Escalation**: Escalate to Vendor Relationship Lead.
- **Permanent Fix**: Implement bounded asynchronous execution with deadline propagation.
- **Prevention**: Require all downstream calls to specify explicit connect and read timeouts $\le 1000\text{ms}$.

---

## 5. Deployment Regression & Rollback Runbook
- **Symptoms**: Error rate increases immediately following a code deployment or canary rollout.
- **Immediate Safety Checks**: Check if database migrations were applied in this release.
- **Evidence Collection**: Check `git log` and CI/CD release commit ID.
- **Diagnosis**: Error logs show new exception introduced in release v2.4.0.
- **Mitigation**:
  ```bash
  # Execute automated rollback
  kubectl rollout undo deployment/api-service
  ```
- **Recovery**: Previous stable version v2.3.0 pods reach Ready state.
- **Verification**: Confirm HTTP 5xx error rate drops below $0.05\%$.
- **Escalation**: Notify Release Manager and Product Engineering Lead.
- **Permanent Fix**: Reproduce regression in staging, patch bug, and add automated regression test.
- **Prevention**: Enforce Automated Canary Analysis with automatic abort triggers on error rate deviation.
