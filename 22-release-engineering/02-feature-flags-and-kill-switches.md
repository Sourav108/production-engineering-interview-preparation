# 02. Feature Flags and Emergency Kill Switches

## 1. Problem
When a newly launched feature causes high database load, rolling back the entire binary deployment takes 15–30 minutes through CI/CD pipelines, prolonging the outage.

## 2. Production Context
**Feature Flags (Toggles)** decouple the physical deployment of code from the activation/exposure of business logic.

## 3. Mental Model: The Emergency Kill Switch

```java
public OrderResponse processOrder(OrderRequest req) {
    // Kill Switch: Evaluated dynamically from LaunchDarkly / Redis in <1ms
    if (featureFlags.isEnabled("use-new-recommendation-engine", req.getUserId())) {
        try {
            return newEngine.recommend(req);
        } catch (Exception e) {
            log.warn("New engine failed, falling back to legacy", e);
            return legacyEngine.recommend(req);
        }
    }
    return legacyEngine.recommend(req);
}
```

---

## 4. The 4 Feature Flag Archetypes

| Flag Category | Longevity | Dynamism | Purpose |
| :--- | :--- | :--- | :--- |
| **Release Flag** | Short (1–2 weeks) | Dynamic | Gradual rollout of new feature; removed once at 100% |
| **Kill Switch / Circuit Flag** | Permanent | High Speed | Instantly disable resource-heavy features during incidents |
| **Experiment Flag (A/B)** | Medium (1–4 weeks) | Dynamic | Multivariate statistical conversion testing |
| **Permission / Entitlement** | Permanent | Per-User | Gating features based on customer subscription tier |

---

## 5. Interview Questions & Model Answers

**Q1: How do Feature Flags and Kill Switches reduce Mean Time to Mitigate (MTTM) during an incident?**
**Answer**: In traditional architectures without feature flags, mitigating a buggy new feature requires initiating a full CI/CD deployment rollback, rebuilding images, running tests, and executing a rolling pod restart—taking 15 to 45 minutes. With a centralized feature flag system (such as LaunchDarkly, Unleash, or Redis-backed config), an on-call engineer can flip an emergency **Kill Switch** toggle via an administrative dashboard or CLI. The updated flag state propagates to all running microservices via WebSocket/SSE streams in **$< 200\text{ms}$**, instantly disabling the problematic code path and eliminating user impact in seconds without a redeploy.
