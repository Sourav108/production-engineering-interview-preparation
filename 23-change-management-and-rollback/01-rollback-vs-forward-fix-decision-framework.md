# 01. Rollback vs. Fix-Forward Decision Framework

## 1. Problem
During a deployment outage, engineers spend 45 minutes attempting to write, test, and deploy a quick "one-line bug fix" (fix-forward) under high stress, which introduces a second bug and doubles the duration of the outage.

## 2. Production Context
Fixing forward under incident pressure is prone to cognitive error and skipped QA validations.

## 3. Mental Model: The 10-Minute Decision Invariant

```
                        The Rollback Decision Matrix
 ──────────────────────────────────────────────────────────────────────────────────────────
 1. Is the root cause a recently deployed code change?
    ├── YES ──► Can the change be rolled back in < 5 minutes?
    │            ├── YES ──► IMMEDIATELY ROLL BACK! (Mitigate first!).
    │            └── NO (e.g. Non-reversible DB migration) ──► Apply emergency fix-forward.
    └── NO  ──► Apply operational mitigation (Rate limit / Failover / Scale).
```

---

## 4. Comparison: Deployment Rollback Strategies

| Strategy | How Rollback Works | Rollback Speed | Resource Cost |
| :--- | :--- | :--- | :--- |
| **Canary Rollback** | Terminate canary pod ($2\%$ traffic); route $100\%$ back to baseline | **$< 30$ seconds** | Low ($+2\%$ compute) |
| **Blue-Green Rollback** | Flip load balancer routing router back to the idle "Blue" environment | **$< 5$ seconds** | High ($+100\%$ duplicated fleet) |
| **Rolling Update Rollback** | Issue `kubectl rollout undo deployment/<name>` | **2–5 minutes** | Low (Replaces pods incrementally) |

---

## 5. Interview Questions & Model Answers

**Q1: Under what specific production conditions is a Rollback NOT possible, forcing a Fix-Forward approach?**
**Answer**: A rollback is impossible or dangerous when:
1. **Destructive Database Schema Changes**: A migration dropped or altered columns/data types in a way that older application code cannot parse.
2. **Data Format Incompatibility**: The new version wrote hundreds of thousands of stateful messages/records in a new schema format that the older binary cannot deserialize.
3. **Irreversible Third-Party API Migrations**: External webhooks or OAuth tokens were permanently migrated to new authentication endpoints. In these rare scenarios, an emergency **Fix-Forward** or Feature Flag kill-switch must be executed.
