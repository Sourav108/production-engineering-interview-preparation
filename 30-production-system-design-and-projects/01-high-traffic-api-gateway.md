# 01. Production System Design: High-Traffic Global API Gateway

## 1. System Requirements & Functional Scope
- **Functional**: Terminate client TLS, authenticate JWT tokens, route requests dynamically, apply rate limiting per tenant/IP, and aggregate telemetry.
- **Availability Target**: $99.99\%$ Availability ($< 4.32\text{ minutes}$ downtime/month).
- **Latency SLO**: $p99 \le 15\text{ms}$ gateway proxy overhead.

---

## 2. Traffic & Load Projections
- **Peak Traffic**: $250,000\text{ Requests Per Second (RPS)}$.
- **Average Request Payload**: $2\text{KB}$; **Average Response Payload**: $8\text{KB}$.
- **Network Bandwidth Demand**:
  $$\text{Ingress Bandwidth} = 250,000 \times 2\text{KB} = 500\text{ MB/s} = \mathbf{4.0\text{ Gbps}}$$
  $$\text{Egress Bandwidth} = 250,000 \times 8\text{KB} = 2,000\text{ MB/s} = \mathbf{16.0\text{ Gbps}}$$

---

## 3. Latency Budget Breakdown
- **Edge Anycast TLS Termination**: $\le 1\text{ms}$ (Local Point of Presence).
- **JWT Signature Verification (Ed25519 in memory)**: $\le 0.2\text{ms}$.
- **Redis Lua Token-Bucket Rate Limit Check**: $\le 1.5\text{ms}$.
- **Envoy Reverse Proxy Routing Overhead**: $\le 0.8\text{ms}$.
- **Total Gateway Overhead**: $\mathbf{\approx 3.5\text{ms}}$ (Well within the 15ms budget).

---

## 4. Capacity & Hardware Sizing
- **CPU Demand**: Envoy handles $\approx 3,000\text{ RPS per CPU Core}$.
  $$\text{Cores Required} = \frac{250,000}{3,000 \times 0.60\text{ (Safe Utilization)}} = \mathbf{138\ Cores}$$
- **Kubernetes Deployment**: 36 pods (each with 4 CPU cores, 4GB RAM) spread across 3 Availability Zones (12 pods per AZ) with $N+1$ redundancy.
- **Redis Cluster for Rate Limiting**: 6-node Redis cluster (3 Masters, 3 Replicas) with NVMe memory storage.

---

## 5. Architecture & System Diagram

```mermaid
flowchart TD
    CLIENT[Global Mobile / Web Clients] --> ANYCAST[Anycast BGP Edge Routing]
    ANYCAST --> NLB[AWS Network Load Balancer - Layer 4]
    NLB --> ENVOY[Envoy Gateway Pods - Layer 7: Envoy Filter Chain]
    
    subgraph Envoy Filter Chain Pipeline
        ENVOY --> AUTH[1. JWT Auth Filter (Local In-Memory Cache)]
        AUTH --> RATELIMIT[2. Rate Limiting Filter (Redis Cluster)]
        RATELIMIT --> SHED[3. Adaptive Load Shedding Filter]
        SHED --> TRACE[4. OpenTelemetry W3C Tracing Injector]
    end
    
    RATELIMIT <--> REDIS[(Redis Cluster: Atomic Token Bucket)]
    TRACE --> UPSTREAM[Internal Backend Microservice Mesh]
```

---

## 6. Dependency Resilience & Failure Isolation
- **Redis Rate-Limiter Outage**: If the Redis cluster becomes unavailable, the rate-limiting filter **fails open** (allows traffic) rather than failing closed, preserving core API availability.
- **Upstream Service Slowdown**: Envoy routes traffic using active outlier detection: if an upstream instance returns 5 consecutive HTTP 503s, Envoy automatically ejects it from the load balancer pool for 30 seconds.

---

## 7. Observability & Telemetry
- **Prometheus RED Metrics**: `envoy_http_downstream_rq_total` (Rate & Errors), `envoy_http_downstream_rq_time` (Histograms for p95/p99 latency).
- **Distributed Tracing**: Generates W3C `traceparent` headers for $100\%$ of error requests and $1\%$ of normal requests.
- **Alert Rules**: Multi-window burn rate alert on $14.4\times$ error budget burn (Pages on-call within 2 minutes).

---

## 8. Failure Modes & Mitigations
- **DDoS Layer 7 Flood**: Cloudflare / AWS Shield drops malicious IP ranges at edge Anycast layer before reaching NLBs.
- **Ingress TLS Expiration**: `cert-manager` automatically renews Let's Encrypt certificates 30 days prior to expiry via DNS-01 challenges.

---

## 9. Security & Compliance
- mTLS (Mutual TLS) enforced between Gateway and internal backend microservices via SPIRE/Istio service mesh.
- Rate limits per API key enforced to prevent credential brute-forcing.

---

## 10. Disaster Recovery & Failover
- Global Anycast DNS routing (Cloudflare / Route53 Latency-Based Routing) automatically reroutes traffic to backup region within 15 seconds if an entire region suffers a datacenter outage.

---

## 11. Architectural Trade-offs
- **Fail-Open vs Fail-Closed Rate Limiter**: We chose **Fail-Open** when Redis fails to prioritize Availability over strict Quota Enforcement.
- **In-Memory JWT vs Centralized Session DB**: We chose **Stateless JWTs with local public key caching** to achieve sub-millisecond authentication at the cost of requiring token revocation lists for immediate blacklisting.
