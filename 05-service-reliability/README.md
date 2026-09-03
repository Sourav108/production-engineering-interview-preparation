# Module 05: Service Reliability & Safe Failure Design

## Learning Objectives

By the end of this module, you will be able to:
- Design services that **fail safely** under partial degradation following the sequence: **Failure $\to$ Contain $\to$ Degrade $\to$ Recover**.
- Segment systems into independent **Failure Domains** (processes, hosts, racks, Availability Zones, Regions) to minimize blast radius.
- Implement rock-solid **Liveness, Readiness, and Startup Probes** in Kubernetes without inducing cascading restarts.
- Configure deterministic **Graceful Shutdown Hooks** to drain in-flight requests with zero client drops during deployments.

---

## Lessons in This Module

| File | Topic | Focus |
| :--- | :--- | :--- |
| [01-failure-domains-and-redundancy.md](01-failure-domains-and-redundancy.md) | Failure Domains & Redundancy | Multi-AZ isolation, cell-based architectures, blast radius containment |
| [02-health-checks-liveness-vs-readiness.md](02-health-checks-liveness-vs-readiness.md) | Liveness vs. Readiness Probes | Health check traps, dependency checking anti-patterns, probe tuning |
| [03-graceful-shutdown-and-lifecycle.md](03-graceful-shutdown-and-lifecycle.md) | Graceful Shutdown & Draining | SIGTERM vs SIGKILL, preStop hooks, draining connection pools |
