# 02. Exponential Backoff, Full Jitter, and Retry Budgets

## 1. Problem
Standard exponential backoff without randomness causes **Synchronized Thundering Herds**: if 5,000 clients fail at the exact same second, backing off by $2^1 = 2\text{s}$ means all 5,000 clients retry together in a massive synchronized spike at second 2, second 4, and second 8.

## 2. Production Context
Adding **Full Jitter** breaks synchronization by spreading retries uniformly across the entire backoff interval.

## 3. Mental Model: The AWS Full Jitter Formula
$$\mathbf{Sleep\ Duration} = \mathbf{random}\left(0,\ \min\left(\text{MaxBackoff},\ \text{BaseSleep} \times 2^{\text{attempt}}\right)\right)$$

### Visual Comparison: No Jitter vs. Full Jitter
```
 No Jitter:    ───► [5,000 Retries at 2.0s] ───► [5,000 Retries at 4.0s] (Destructive Spikes!)
 Full Jitter:  ───► Smooth uniform distribution of retries between 0.0s and 2.0s (No Spike!)
```

---

## 4. Retry Budgets (Token Bucket Enforcement)

To prevent cascading overload, clients enforce a **Retry Budget**:
- **Rule**: Retries are only permitted if they constitute $< 10\%$ of total request volume.
- **Mechanism**: Successful requests deposit 1 credit into a token bucket (max capacity 100); each retry consumes 10 credits. If the bucket is empty, retries are **immediately dropped without network dispatch**.

---

## 5. Interview Questions & Model Answers

**Q1: Why is Full Jitter mathematically superior to Equal Jitter or Decorrelated Jitter?**
**Answer**: AWS Architecture benchmarks proved that **Full Jitter** ($\text{random}(0, \text{backoff})$) provides the lowest total client wait time while delivering the most uniform load distribution on the downstream service. While Equal Jitter keeps a fixed half-interval, Full Jitter allows some retries to execute almost immediately if capacity opens up, while dispersing the remainder smoothly across the entire backoff window, eliminating any discrete wave spikes.

**Q2: What is a Retry Budget, and how does it prevent cascading retry storms?**
**Answer**: A Retry Budget is a client-side rate-limiting mechanism that caps the total volume of retries as a fixed percentage (typically 10%) of overall outbound requests. In steady state where failure rate is $<1\%$, retries execute normally. However, during a widespread downstream outage where $80\%$ of calls fail, the client exhausts its retry budget within seconds. Subsequent failures fail-fast immediately rather than hammering the failing service, allowing downstream systems to recover.
