# 01. Failure Domains, Redundancy, and Blast Radius Containment

## 1. Problem
A localized hardware fault (e.g. a failing network switch in one data center rack or a single corrupted database partition) propagates across the entire distributed system, causing a global, company-wide outage.

## 2. Production Context
At scale, components *will* fail. Production engineers design systems using failure domain segmentation and cell-based architectures so that no single failure can take down more than a bounded fraction of traffic (e.g. $\le 5\%$).

## 3. Mental Model
$$\text{Failure Domain Hierarchy:}$$
$$\text{Thread} \subset \text{Process} \subset \text{Host VM} \subset \text{Rack} \subset \text{Availability Zone (AZ)} \subset \text{Region}$$
- **Blast Radius**: The maximum percentage of users, requests, or revenue impacted when a specific failure domain collapses.
- **Cell-Based Architecture**: Partitioning the entire user base into isolated, self-contained "cells" (e.g. 20 cells, each serving 5% of users). If Cell 4 experiences a catastrophic database corruption, $95\%$ of global users experience zero degradation.

## 4. System Diagram
```mermaid
flowchart TD
    subgraph Cell-Based Architecture [Bounded Blast Radius]
        ROUTER[Smart Gateway Router]
        
        subgraph Cell 1 [5% Users]
            C1_APP[App Pods] --> C1_DB[(Isolated DB)]
        end
        
        subgraph Cell 2 [5% Users]
            C2_APP[App Pods] --> C2_DB[(Isolated DB)]
        end
        
        subgraph Cell N [5% Users]
            CN_APP[App Pods] --> CN_DB[(Isolated DB)]
        end
        
        ROUTER -->|User A..F| Cell 1
        ROUTER -->|User G..M| Cell 2
        ROUTER -->|User N..Z| Cell N
    end
```

## 5. Signals
- **Localized Outage**: High error rates isolated strictly to Availability Zone `us-east-1a` while `us-east-1b` and `us-east-1c` remain 100% healthy.
- **Cascading Contagion**: Error rate jumping from $5\%$ in AZ-a to $100\%$ globally within 60 seconds (indicating lack of failure domain isolation).

## 6. Failure Modes
- **Shared Global Dependency**: All cells relying on a single centralized configuration database; when the config DB locks up, all cells collapse simultaneously.
- **Corrupted Poison Pill**: A malformed request routed sequentially to all nodes, crashing each pod upon deserialization (Poison Pill Cascade).

## 7. Detection
- Slice Prometheus metrics by dimensions: `zone`, `region`, `cell_id`, and `node_name`.
- Check if errors correlate strictly with a specific availability zone or node pool.

## 8. Diagnosis
1. Inspect error rate grouped by AZ:
   `sum(rate(http_requests_total{status=~"5.."}[2m])) by (zone)`
2. If one zone is $100\%$ failing while others are healthy: Network switch, power loss, or EBS degradation in that specific AZ.

## 9. Mitigation
- **Zone Eviction**: Update load balancer target group to immediately disable traffic routing to the degraded Availability Zone.
- **Cell Isolation**: Quarantine the affected cell and route incoming traffic to maintenance fallback.

## 10. Recovery
- Repair the failed node or wait for cloud provider AZ recovery before slowly re-enabling traffic weight ($10\% \to 50\% \to 100\%$).

## 11. Verification
- Verify global error rate drops back to baseline ($< 0.05\%$) immediately after draining the failed AZ.

## 12. Prevention
- Enforce **Topology Spread Constraints** in Kubernetes to distribute pod replicas evenly across availability zones:
  `topologySpreadConstraints: [{ maxSkew: 1, topologyKey: topology.kubernetes.io/zone, whenUnsatisfiable: DoNotSchedule }]`

## 13. Automation
- Automated AZ evacuation controllers that detect elevated error rates in a single zone and automatically remove DNS/LB weights.

## 14. Performance
- Multi-AZ deployments incur minor cross-AZ network latency ($0.5\text{ms} - 1.5\text{ms}$) compared to single-AZ co-location.

## 15. Reliability
- $N+1$ or $N+2$ cluster capacity guarantees that remaining healthy AZs can absorb $100\%$ of traffic without CPU saturation when 1 AZ fails.

## 16. Trade-offs
- **Cross-AZ Traffic Costs vs Availability**: Multi-AZ architectures incur data transfer charges for inter-zone replication, but eliminate single-datacenter catastrophic downtime.

## 17. Production Example
At fictional messaging platform *ChatWave*, an AWS Availability Zone experienced a major EBS storage degradation. Because *ChatWave* implemented Kubernetes Topology Spread Constraints and cell-based partitioning, 2 out of 3 AZs continued operating normally. An automated zone-eviction health check removed the degraded AZ from the Route53 latency routing pool within 45 seconds, resulting in a minor $1.2\%$ blip in error rate that self-healed before breaching the monthly 99.95% SLO.

## 18. Interview Questions
- **Q1**: *What is a Cell-Based Architecture, and how does it prevent global cascading outages?*
- **Q2**: *How do Kubernetes Topology Spread Constraints work across Availability Zones?*
- **Q3**: *Why is an active-active multi-AZ architecture preferred over active-passive for stateless web services?*

## 19. Strong Interview Answer
> *"A failure domain defines the geographical or logical boundary within which a fault is contained. In modern production systems, we design multi-layered failure domains: at the host level via container cgroups, at the data center level across at least 3 Availability Zones, and at the architectural level using Cell-Based Architectures. In a cell-based system, complete stacks (app, cache, database) are replicated into isolated units serving a deterministic hash of the user base. This mathematically bounds the blast radius of any catastrophic failure—such as a schema migration bug or database lock contention—to a single cell (e.g. 5% of users), preventing global outages."*

## 20. Hands-on Exercise
1. **Setup**: Inspect a Kubernetes deployment YAML spec.
2. **Configure Spread**: Add a `topologySpreadConstraints` block targeting `topology.kubernetes.io/zone`.
3. **Verify**: Deploy 6 replicas on a multi-node cluster and verify pods are scheduled evenly (2 pods per zone) using `kubectl get pods -o wide`.
