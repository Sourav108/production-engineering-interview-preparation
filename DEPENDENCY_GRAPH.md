# Production Engineering Learning Dependency Graph

This document defines the strict pedagogical progression from system foundations to advanced distributed resilience and Staff-level production architecture.

---

## 🗺️ Master Prerequisite Dependency Graph

```mermaid
flowchart TD
    %% Main Linear Spine
    M01[01: Foundations & Reliability Dimensions] --> M02[02: Linux & Systems Behavior]
    M02 --> M03[03: Processes, CPU & Memory Leaks]
    M03 --> M04[04: Networking & Request Path]
    M04 --> M05[05: Service Reliability & Failure Domains]
    M05 --> M06[06: Observability Fundamentals]
    M06 --> M07[07: Logs, Metrics & Traces]
    M07 --> M08[08: Monitoring & Actionable Alerting]
    M08 --> M09[09: SLIs, SLOs & Error Budgets]
    M09 --> M10[10: On-Call & Incident Management]
    M10 --> M11[11: Production Debugging]
    M11 --> M12[12: Latency, Queueing & Little's Law]
    M12 --> M13[13: Capacity Planning & Headroom]
    M13 --> M14[14: Load Testing & Benchmarking]
    M14 --> M15[15: Dependency Management]
    M15 --> M16[16: Timeouts, Retries & Breakers]
    M16 --> M17[17: Rate Limiting & Backpressure]
    M17 --> M18[18: Fault Isolation & Degradation]
    M18 --> M19[19: Caching & Stampedes]
    M19 --> M20[20: Database Reliability]
    M20 --> M21[21: Distributed Systems Failures]
    M21 --> M22[22: Release Engineering & Canaries]
    M22 --> M23[23: Change Management & Rollback]
    M23 --> M24[24: High Availability & DR]
    M24 --> M25[25: Production Security]
    M25 --> M26[26: Operational Automation & Toil]
    M26 --> M27[27: Reliability Testing & Chaos]
    M27 --> M28[28: Postmortems & Learning]
    M28 --> M29[29: 500 Interview Questions]
    M29 --> M30[30: System Design & Projects]

    %% Parallel Cross-Cutting Tracks
    subgraph Parallel Track: Observability
        M06 -.-> M07
        M07 -.-> M08
        M08 -.-> M09
    end

    subgraph Parallel Track: Incident & Operations
        M10 -.-> M11
        M11 -.-> M28
    end

    subgraph Parallel Track: Resilience
        M16 -.-> M17
        M17 -.-> M18
        M18 -.-> M27
    end
```

---

## 📌 Module Prerequisite Guide

| Module | Hard Prerequisites | Unlocks |
| :--- | :--- | :--- |
| **01–05: Systems Foundations** | Basic Linux/programming knowledge | Observability, Networking, Debugging |
| **06–10: Observability & Incidents** | Modules 01–05 | Production Debugging, SLO Error Budgets |
| **11–14: Performance & Capacity** | Modules 02, 03, 07 | Load Testing, Capacity Forecasting |
| **15–21: Resilience & Distributed Systems** | Modules 04, 05, 12 | Release Safety, Chaos Engineering |
| **22–28: Operations & Security** | Modules 10, 16, 21 | DR Failover, Automated Self-Healing |
| **29–30: Interview Mastery & Projects** | Modules 01–28 | Senior & Staff Hiring Loops |
