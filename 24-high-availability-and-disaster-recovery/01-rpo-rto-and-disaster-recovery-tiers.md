# 01. RPO, RTO, and Disaster Recovery Architecture Tiers

## 1. Problem
During a regional cloud outage (e.g. AWS US-East-1 datacenter power failure), an organization realizes its backups were never tested for restore compatibility, turning a planned 15-minute recovery into a 3-day data loss catastrophe.

## 2. Production Context
Disaster Recovery (DR) plans that exist only as untested Word documents fail in production. Recovery must be continuously proven via automated drills.

## 3. Mental Model: RPO vs. RTO

```
 ◄────────────────────── Past ──────────────────────┼────────────────────── Future ──────────────────────►
                                              Disaster Occurs
 
 ◄─────── [RPO: Maximum Data Loss] ─────────────────► ◄─────── [RTO: Maximum Downtime] ────────────────►
 Last Committed Backup / Replica Sync                Service Fully Restored & Serving Traffic
```

- **Recovery Point Objective (RPO)**: The maximum acceptable age of data that can be permanently lost when a failure occurs (measured in minutes/seconds of data loss).
- **Recovery Time Objective (RTO)**: The maximum acceptable duration of service downtime before normal operations are restored (measured in minutes/hours of outage).

---

## 4. The 4 Disaster Recovery Architectural Tiers

| DR Strategy | Description | Typical RPO | Typical RTO | Infrastructure Cost |
| :--- | :--- | :--- | :--- | :--- |
| **1. Backup & Restore** | Daily snapshots stored in secondary cloud region | $\approx 24\text{ hours}$ | $12 - 24\text{ hours}$ | $\mathbf{\$}$ (Lowest) |
| **2. Pilot Light** | Core DB replicated continuously; compute scaled to 0 | $< 1\text{ minute}$ | $15 - 30\text{ minutes}$ | $\mathbf{\$\$}$ |
| **3. Warm Standby** | Scaled-down minimal cluster running in secondary region | $< 5\text{ seconds}$ | $< 5\text{ minutes}$ | $\mathbf{\$\$\$}$ |
| **4. Multi-Region Active-Active**| Full live capacity in $2+$ regions serving traffic concurrently | $\mathbf{\approx 0\text{ seconds}}$ | $\mathbf{\approx 0\text{ seconds}}$ (Seamless) | $\mathbf{\$\$\$\$}$ (Highest) |

---

## 5. The Golden Rule of Backups: An Untested Backup Does Not Exist
$$\mathbf{Automated\ Daily\ Restore\ Verification\ Job:}$$
$$\text{Restore S3 Snapshot} \longrightarrow \text{Spin Up Ephemeral RDS Instance} \longrightarrow \text{Execute SQL Validation Checks} \longrightarrow \text{Tear Down}$$

---

## 6. Interview Questions & Model Answers

**Q1: What is the difference between RPO and RTO?**
**Answer**: **RPO (Recovery Point Objective)** defines the maximum tolerable volume of data loss measured in time: it answers *"How much data can the business afford to lose from the last backup/replication point?"* (e.g. an RPO of 5 minutes means we can lose at most 5 minutes of recent transactions). **RTO (Recovery Time Objective)** defines the maximum tolerable duration of system downtime: it answers *"How long can the business afford to wait before the system is back online and operational?"* (e.g. an RTO of 30 minutes means the failover and DNS switch must complete within 30 minutes).
