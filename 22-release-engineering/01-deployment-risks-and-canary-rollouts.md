# 01. Canary Deployments & Automated Canary Analysis (ACA)

## 1. Problem
Deploying a subtle software regression (e.g. a null pointer exception triggered only by users on Safari iOS) to 100% of servers causes an immediate company-wide outage affecting millions of users.

## 2. Production Context
Releases must be treated as **Controlled Scientific Experiments**: a new version is introduced to a small, isolated fraction of live traffic while automated statistical algorithms compare error rates and latency against an identical baseline version.

## 3. Mental Model: The Staged Rollout Pipeline
$$\mathbf{Canary\ Cadence:} \quad \mathbf{1\%\ Traffic\ (10m)} \longrightarrow \mathbf{5\%\ (20m)} \longrightarrow \mathbf{25\%\ (30m)} \longrightarrow \mathbf{100\%\ (Complete)}$$

```mermaid
flowchart TD
    INGRESS[Smart Ingress Router: 100k RPS]
    
    subgraph Baseline Fleet [99% Traffic: v2.3.0]
        BASE_PODS[99 Pods: Measure Baseline Error Rate]
    end
    
    subgraph Canary Fleet [1% Traffic: v2.4.0]
        CANARY_POD[1 Pod: Measure Canary Error Rate]
    end
    
    INGRESS -->|99%| Baseline Fleet
    INGRESS -->|1%| Canary Fleet
    
    BASE_PODS -.-> ACA{Automated Canary Analysis}
    CANARY_POD -.-> ACA
    
    ACA -->|Canary 5xx > Baseline + 0.5%| ABORT[INSTANT AUTOMATED ROLLBACK (<30s)!]
    ACA -->|Canary Metrics Healthy| PROMOTE[Advance to Next Stage: 5%]
```

---

## 4. Kubernetes Argo Rollouts Canary Specification (YAML)

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: payment-service
spec:
  replicas: 50
  strategy:
    canary:
      analysis:
        templates:
          - templateName: error-rate-check
        args:
          - name: service-name
            value: payment-service
      steps:
        - setWeight: 2       # 2% Traffic to Canary
        - pause: { duration: 10m } # Observe for 10 minutes
        - setWeight: 10      # 10% Traffic
        - pause: { duration: 20m }
        - setWeight: 50      # 50% Traffic
        - pause: { duration: 15m }
```

---

## 5. Interview Questions & Model Answers

**Q1: How does Automated Canary Analysis (ACA) prevent bad code releases from causing large outages?**
**Answer**: ACA routes a tiny percentage of production traffic (e.g. 1% to 2%) to a newly deployed canary instance running alongside the stable baseline fleet. An automated controller (like Argo Rollouts or Kayenta) continuously queries Prometheus, comparing the canary's RED metrics (5xx error percentage, p95/p99 latency, and CPU/memory trends) against the baseline. If the canary's error rate deviates statistically from the baseline (e.g. error rate $> \text{baseline} + 0.5\%$), the controller **aborts the deployment and rolls back traffic in $<30$ seconds**, restricting customer impact to $<1\%$ of users.
