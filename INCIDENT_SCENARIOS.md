# Master Incident Scenarios Index

A catalog linking 50+ real-world production incident scenarios across host, networking, application, database, and distributed systems failure domains.

---

## 🚨 Incident Scenario Categories

1. **Host & Resource Incidents**: CPU saturation, memory leaks, Linux OOM kills, socket/FD exhaustion.
2. **Network & Request Path Outages**: DNS resolution failure, TLS cert expiry, proxy 504 timeouts, packet loss.
3. **Application & Concurrency Failures**: Thread pool starvation, deadlock cascades, synchronous HTTP calls inside DB transactions.
4. **Database & Storage Incidents**: Connection pool exhaustion, slow query table scans, lock queuing, replication lag.
5. **Distributed Systems & Dependency Cascades**: Retry storms, cache stampedes, split-brain, network partitions.

*Detailed incident write-ups with full 16-field postmortem structures are documented in [`incidents/`](incidents/) and [Module 28](28-postmortems-and-learning/).*
