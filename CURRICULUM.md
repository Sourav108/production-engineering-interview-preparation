# Production Engineering & SRE Curriculum

A structured, implementation-first syllabus for mastering production reliability, distributed systems operation, observability, incident management, and high-scale architecture.

---

## 🧭 Curriculum Architecture & Progression

```mermaid
flowchart TD
    subgraph Layer 1: Systems & Host Foundations [Weeks 1-2]
        M01[01: PE Foundations] --> M02[02: Linux & Systems]
        M02 --> M03[03: CPU & Memory]
        M03 --> M04[04: Network & Request Path]
        M04 --> M05[05: Service Reliability]
    end

    subgraph Layer 2: Observability & Incident Response [Weeks 2-3]
        M05 --> M06[06: Observability]
        M06 --> M07[07: Logs, Metrics & Traces]
        M07 --> M08[08: Monitoring & Alerting]
        M08 --> M09[09: SLI, SLO & Error Budgets]
        M09 --> M10[10: Incident Response]
    end

    subgraph Layer 3: Performance, Debugging & Capacity [Weeks 3-4]
        M10 --> M11[11: Production Debugging]
        M11 --> M12[12: Latency & Performance]
        M12 --> M13[13: Capacity Planning]
        M13 --> M14[14: Load Testing]
    end

    subgraph Layer 4: Resilience & Distributed Failures [Weeks 4-5]
        M14 --> M15[15: Dependencies]
        M15 --> M16[16: Timeouts & Retries]
        M16 --> M17[17: Rate Limiting]
        M17 --> M18[18: Fault Isolation]
        M18 --> M19[19: Caching]
        M19 --> M20[20: Database Reliability]
        M20 --> M21[21: Distributed Failures]
    end

    subgraph Layer 5: Operations, Release, Security & Automation [Weeks 5-6]
        M21 --> M22[22: Release Engineering]
        M22 --> M23[23: Rollback & Migrations]
        M23 --> M24[24: HA & Disaster Recovery]
        M24 --> M25[25: Security]
        M25 --> M26[26: Automation & Toil]
        M26 --> M27[27: Chaos Testing]
        M27 --> M28[28: Postmortems]
    end

    subgraph Layer 6: Interview Mastery & Projects [Week 6]
        M28 --> M29[29: Interview Question Bank]
        M29 --> M30[30: System Design & Projects]
    end
```

---

## 📋 Detailed Module Breakdown

### Track 1: Host & System Foundations (Modules 01–05)
- **Module 01: Production Engineering Foundations**: Definitions of reliability, availability, durability, operability, scalability; PE vs SRE vs Platform Engineering.
- **Module 02: Linux and System Behavior**: Diagnosing CPU, memory, `/proc`, PSI, disk I/O, file descriptors, sockets with standard Linux tools (`top`, `vmstat`, `iostat`, `ss`, `lsof`, `sar`, `strace`).
- **Module 03: Processes, Memory and CPU**: Process lifecycles, thread pools, CPU saturation, context switching, OOM-killer mechanics, GC pauses, memory leaks.
- **Module 04: Networking and Request Path**: Tracing requests across DNS, TCP handshakes, TLS termination, HTTP protocols, Load Balancers, Proxies, and connection timeouts.
- **Module 05: Service Reliability**: Designing for failure, liveness vs readiness probes, graceful shutdown hooks, failure domains, blast radius containment.

### Track 2: Observability & Incident Response (Modules 06–10)
- **Module 06: Observability Fundamentals**: The telemetry spectrum, high cardinality data, debugging unknown-unknowns.
- **Module 07: Logs, Metrics and Traces**: Structured logging, OpenTelemetry distributed tracing, metric counters/gauges/histograms, tail latency percentiles (p95/p99 vs misleading averages).
- **Module 08: Monitoring and Alerting**: Designing actionable alerts, symptom-based alerting, multi-window multi-burn-rate alerts, eliminating alert fatigue.
- **Module 09: SLI, SLO, SLA and Error Budgets**: Deriving SLIs from customer journeys, setting data-driven SLOs, error budget burn policies and release gating.
- **Module 10: On-Call and Incident Response**: The SIGNAL framework, Incident Commander role, severity classification, communication cadences, timeline discipline.

### Track 3: Performance, Debugging & Capacity (Modules 11–14)
- **Module 11: Production Debugging**: Repeatable hypothesis-driven debugging methodology, isolating culprits without random restarts.
- **Module 12: Latency and Performance**: Latency decomposition model ($\text{Network} + \text{Queue} + \text{App} + \text{Dep} + \text{DB}$), Little's Law, queueing saturation.
- **Module 13: Capacity Planning**: Resource demand forecasting, identifying knee-of-the-curve saturation points, headroom calculation.
- **Module 14: Load Testing and Benchmarking**: Stress, soak, spike, and breakpoint testing using k6; honest percentile reporting.

### Track 4: Dependency Resilience & Distributed Systems (Modules 15–21)
- **Module 15: Dependency Management**: Critical vs non-critical dependencies, blast radius isolation, cascading failure containment.
- **Module 16: Timeouts, Retries and Circuit Breakers**: Exponential backoff with full jitter, retry budgets, circuit breaker state machines.
- **Module 17: Rate Limiting and Backpressure**: Token bucket, leaky bucket, concurrency limits, adaptive load shedding.
- **Module 18: Graceful Degradation and Fault Isolation**: Fallback strategies, bulkheads, dependency shedding, avoiding dangerous fallbacks.
- **Module 19: Caching and Performance**: Cache-aside patterns, cache stampede prevention, TTL tuning, cache failure resilience.
- **Module 20: Database Reliability**: DB connection pool exhaustion, slow query triage, lock contention, replication lag, failover management.
- **Module 21: Distributed Systems Failures**: Network partitions, clock skew, split-brain, message ordering, distributed cascading failures.

### Track 5: Release, Operations, Security & Chaos (Modules 22–28)
- **Module 22: Release Engineering**: Canary deployments, blue-green releases, feature flags, deployment risk mitigation.
- **Module 23: Change Management and Rollback**: Automated rollback triggers, backward-compatible database schema migrations.
- **Module 24: High Availability and Disaster Recovery**: Multi-zone/multi-region architectures, RPO, RTO, active-active vs active-passive failover.
- **Module 25: Production Security**: Secret management, certificate rotation, least privilege, credential compromise response.
- **Module 26: Operational Automation and Toil**: Quantifying and eliminating toil, self-healing systems, safe remediation automation.
- **Module 27: Reliability Testing and Chaos**: Chaos engineering, fault injection experiments, blast radius guardrails and abort conditions.
- **Module 28: Postmortems and Learning**: Blameless postmortem culture, root cause analysis, actionable prevention tracking.

### Track 6: Interview Mastery, System Design & Projects (Modules 29–30)
- **Module 29: Production Engineering Interview Questions**: 500 categorized SDE2/Senior/Staff interview questions.
- **Module 30: Production System Design and Projects**: 5 full production system designs and 8 standalone production projects.
