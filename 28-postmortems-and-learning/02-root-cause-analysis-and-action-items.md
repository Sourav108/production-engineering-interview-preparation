# 02. Root Cause Analysis, 5-Whys, and Action Item Governance

## 1. Problem
Postmortems produce vague, un-owned action items (e.g. "We will improve testing" or "Write better documentation"), which are promptly forgotten, leading to repeat outages within 90 days.

## 2. Production Context
A postmortem is only as effective as the engineering action items it produces and tracks to completion.

## 3. Mental Model: 5-Whys Deep-Dive Analysis

```
 Outage: Payment API failed for 42 minutes.
 ├── 1. Why? The database connection pool ran out of available connections.
 ├── 2. Why? Queries to the billing table were taking 4,500ms each instead of 5ms.
 ├── 3. Why? The database was executing a sequential table scan instead of using an index.
 ├── 4. Why? A recent schema migration dropped the index on user_id.
 └── 5. Why? (Root Cause): The CI/CD schema linter did not validate query execution plans
      against production table statistics before allowing migration scripts to merge.
```

---

## 4. The SRE Action Item Governance Matrix

Every action item generated in a postmortem must be categorized and assigned:

| Category | Purpose | Example | Target SLA |
| :--- | :--- | :--- | :--- |
| **Prevent** | Eliminates the root cause permanently | Add CI query plan lint check to block unindexed foreign key queries | **30 Days** |
| **Mitigate** | Reduces the duration/impact if it happens again | Add circuit breaker to timeout queries after 500ms | **14 Days** |
| **Detect** | Reduces time to detect (MTTD) | Add Prometheus multi-window burn rate alert on DB connection wait time | **7 Days** |

### Mandatory Action Item Tracking Invariants:
- **Every action item MUST have a single assigned engineer owner**.
- **Every action item MUST have a hard calendar due date**.
- **Open postmortem action items are reviewed weekly in SRE engineering standups**.

---

## 5. Interview Questions & Model Answers

**Q1: How do you ensure that postmortem action items are actually completed rather than forgotten?**
**Answer**:
1. **Direct Ticket Creation**: All action items are created as high-priority Jira tickets during the postmortem review meeting itself (never left as notes in a doc).
2. **Explicit Ownership**: Each ticket is assigned to a specific engineer (not a generic team alias) with a firm due date (7, 14, or 30 days based on tier).
3. **Engineering SLA & Weekly Review**: Postmortem action items carry a strict engineering SLA and are reviewed weekly in cross-team engineering leadership meetings. If an action item is overdue, new feature pull requests from that service are blocked until the reliability item is resolved.
