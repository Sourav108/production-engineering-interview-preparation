# 05. Production System Design: Multi-Region Active-Active Backend

## 1. System Requirements & Functional Scope
- **Functional**: Globally distributed banking and identity backend deployed simultaneously across 3 cloud regions (**US-East, EU-West, AP-South**).
- **Resilience Invariant**: **$\mathbf{RPO = 0}$ (Zero Data Loss) & $\mathbf{RTO < 1\text{ second}}$ (Instant Failover)** if an entire cloud region is destroyed by natural disaster or fiber severance.

---

## 2. Global Architecture: Distributed Consensus with CockroachDB / Spanner

```mermaid
flowchart TD
    subgraph Region 1: US-East [Live Traffic]
        US_USER[US Users] --> US_DNS[Anycast Edge]
        US_DNS --> US_APP[US Microservices]
        US_APP <--> CRDB_US[(CockroachDB Region 1: Raft Range Leaseholder)]
    end

    subgraph Region 2: EU-West [Live Traffic]
        EU_USER[EU Users] --> EU_DNS[Anycast Edge]
        EU_DNS --> EU_APP[EU Microservices]
        EU_APP <--> CRDB_EU[(CockroachDB Region 2: Raft Follower)]
    end

    subgraph Region 3: AP-South [Live Traffic]
        AP_USER[Asia Users] --> AP_DNS[Anycast Edge]
        AP_DNS --> AP_APP[Asia Microservices]
        AP_APP <--> CRDB_AP[(CockroachDB Region 3: Raft Follower)]
    end

    CRDB_US <===>|Raft Distributed Consensus: Quorum (2/3)| CRDB_EU
    CRDB_EU <===>|Raft Distributed Consensus: Quorum (2/3)| CRDB_AP
    CRDB_US <===>|Raft Distributed Consensus: Quorum (2/3)| CRDB_AP
```

---

## 3. Data Locality & Latency Optimization
- **Table Partitioning by Region (Row-Level Geo-Partitioning)**:
  - European users' rows have their Raft Leaseholder pinned to `EU-West` nodes $\implies$ Local write latency is **$\approx 3\text{ms}$** (satisfies GDPR data residency laws).
  - US users' rows pinned to `US-East` $\implies$ Local write latency is **$\approx 3\text{ms}$**.
- **Global Read Consistency**: Non-localized global reference tables (e.g. currency exchange rates) use bounded staleness (`AS OF SYSTEM TIME`) for sub-millisecond local reads across all three continents.

---

## 4. Regional Failover Protocol (Catastrophic Outage Simulation)
1. **US-East Drops Offline** (Complete datacenter blackout).
2. **Raft Quorum Maintenance**: EU-West and AP-South still form a strict **2 out of 3 majority quorum**.
3. **Automated Range Lease Re-election**: Within $900\text{ms}$, CockroachDB re-elects leaseholders in EU-West.
4. **Anycast BGP Traffic Rerouting**: Cloudflare Anycast automatically shifts US traffic to EU-West in $< 15\text{ seconds}$.
5. **RPO & RTO Realized**: **RPO = 0** (No committed transactions lost) and **RTO = 0.9s** (Database remained fully online with zero manual DBA intervention).

---

## 5. Architectural Trade-offs & Production Defense
- **Infrastructure Cost vs Reliability**: Running multi-region distributed SQL with inter-region consensus generates continuous data transfer charges and requires at least 9 database nodes (3 per region). This is defended for mission-critical banking, identity, and payment systems where even 5 minutes of regional downtime exceeds millions in SLA violation penalties.
