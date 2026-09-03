# 02. Graceful Fallbacks & The Dangerous Fallback Anti-Pattern

## 1. Problem
When a user profile service fails, a naive fallback returns a default empty user object with `role = null`. A downstream authorization service interprets `null` as `admin` (or checks `if (user != null)`), introducing a catastrophic security vulnerability or silent data corruption.

## 2. Production Context
Fallbacks are designed to maintain degraded user experiences during partial outages. However, **a poorly designed fallback is more dangerous than an explicit HTTP 500 error**, because it silently propagates false assumptions through the distributed system.

## 3. Mental Model: Safe vs. Dangerous Fallback Patterns

| Scenario | ❌ Dangerous Fallback (Anti-Pattern) | ✅ Safe Fallback (Production Pattern) |
| :--- | :--- | :--- |
| **Pricing / Discount Service Fails** | Return `price = 0.00` or `discount = 100%` (Financial Disaster!) | Return original catalog list price (zero discount) or fail-fast with explicit error |
| **Fraud Detection Service Fails** | Return `isFraud = false` (Allows fraudulent transfers during outages!) | Queue transaction for asynchronous manual review or fail-fast |
| **User Recommendations Fail** | Return empty list `[]` that causes UI to break or crash | Return static cached "Top 10 Global Best Sellers" list |
| **User Address Service Fails** | Return default warehouse address (Ships goods to wrong location!) | Prompt user: "Please re-enter delivery address manually" |

---

## 4. The 3 Invariant Rules of Safe Fallbacks
1. **Never Fallback on Security, Authentication, or Financial Auditing**: Fail-fast immediately.
2. **Always Mark Fallback Payloads as Degraded**: Include metadata headers or response flags:
   `"is_degraded": true, "fallback_applied": "cached_stale_data"`
3. **Emit Metrics on Fallback Invocations**: Track `resilience4j_circuitbreaker_fallback_calls_total` to ensure fallbacks do not mask silently failing dependencies.

---

## 5. Interview Questions & Model Answers

**Q1: When is returning an explicit HTTP 500 error better than returning a fallback response?**
**Answer**: An explicit HTTP 500 error is superior whenever returning a default or degraded response compromises **Correctness, Financial Integrity, or Security**. For example, in payment authorizations, pricing calculations, or fraud validation, returning a fallback that assumes zero cost or passes fraud checks introduces catastrophic business losses. In those critical paths, failing fast with an unambiguous error ensures data integrity, triggers automated retries at safe boundaries, and alerts operators immediately. Fallbacks should be reserved strictly for non-critical user-experience enhancements (like recommendation feeds, reviews, or avatar rendering).
