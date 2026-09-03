# Module 24: High Availability & Disaster Recovery (HA/DR)

## Learning Objectives

By the end of this module, you will be able to:
- Define and calculate **Recovery Point Objective (RPO)** and **Recovery Time Objective (RTO)** across Tier-1 vs Tier-3 business systems.
- Architect and compare **Multi-Region Active-Active vs. Active-Passive (Warm Standby / Pilot Light)** deployments.
- Master asynchronous cross-region database replication, conflict resolution (CRDTs), and split-brain fencing.
- Design and execute **Continuous Disaster Recovery Drills (GameDays)** to mathematically prove recoverability.

---

## Lessons in This Module

| File | Topic | Focus |
| :--- | :--- | :--- |
| [01-rpo-rto-and-disaster-recovery-tiers.md](01-rpo-rto-and-disaster-recovery-tiers.md) | RPO, RTO & Disaster Recovery Tiers | RPO (data loss) vs RTO (downtime), backup verification drills |
| [02-multi-region-active-active-vs-active-passive.md](02-multi-region-active-active-vs-active-passive.md) | Multi-Region Active-Active vs. Active-Passive | DNS Anycast routing, cross-region replication lag, write routing |
