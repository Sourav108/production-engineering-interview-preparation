# 03. Circuit Breaker State Machine & Mechanics

## 1. Problem
When a downstream microservice experiences a hard outage or takes 10 seconds to respond, upstream callers continue dispatching thousands of requests, exhausting their own thread pools and crashing the entire platform.

## 2. Production Context
A **Circuit Breaker** acts as an automatic safety switch: it monitors failure rates and, when errors exceed a threshold, trips to **fail-fast immediately** without making network calls, protecting both upstream callers and downstream dependencies.

## 3. Mental Model: The Circuit Breaker State Machine

```mermaid
stateDiagram-v2
    [*] --> Closed
    
    Closed --> Open : Failure Rate >= 50% (over sliding window of 100 requests)
    note right of Closed : Normal operation: all requests pass through.
    
    Open --> HalfOpen : Wait Duration in Open State Expired (e.g. after 30s)
    note right of Open : Fail-Fast: Calls fail immediately without network dispatch!
    
    HalfOpen --> Closed : Trial Requests (e.g. 10 calls) Succeed (>= 90% Success)
    HalfOpen --> Open : Any Trial Request Fails
    note right of HalfOpen : Probe state: allows 10 requests to test downstream health.
```

---

## 4. Production Resilience4j Configuration (YAML)

```yaml
resilience4j:
  circuitbreaker:
    instances:
      paymentService:
        slidingWindowType: COUNT_BASED
        slidingWindowSize: 100               # Evaluate last 100 requests
        minimumNumberOfCalls: 20             # Minimum samples before evaluating threshold
        failureRateThreshold: 50.0           # Trip to OPEN if >= 50% requests fail
        slowCallRateThreshold: 50.0          # Trip to OPEN if >= 50% requests take > 1000ms
        slowCallDurationThreshold: 1000ms    # What constitutes a slow call
        waitDurationInOpenState: 30s         # Stay in OPEN for 30s before trying HALF_OPEN
        permittedNumberOfCallsInHalfOpenState: 10 # Allow 10 trial probes in HALF_OPEN
        automaticTransitionFromOpenToHalfOpenEnabled: true
```

---

## 5. Interview Questions & Model Answers

**Q1: How does a Circuit Breaker protect an upstream calling service from thread exhaustion?**
**Answer**: When a downstream dependency hangs or slows down, upstream worker threads block on network socket read timeouts (e.g. 5,000ms per thread). Under heavy traffic, all available upstream worker threads become trapped waiting for the slow service, exhausting the upstream thread pool and preventing the caller from serving any other traffic. When a Circuit Breaker trips to `OPEN`, it short-circuits subsequent calls in $< 0.1\text{ms}$ by immediately throwing a `CallNotPermittedException` (or invoking a fallback), releasing worker threads instantly and preserving upstream availability.

**Q2: What is the difference between a Count-Based sliding window and a Time-Based sliding window in circuit breakers?**
**Answer**: A **Count-Based** sliding window records the outcome of the last $N$ requests (e.g. last 100 calls) in a ring buffer, making it responsive regardless of request rate. A **Time-Based** sliding window evaluates the failure rate over the last $N$ seconds (e.g. last 60 seconds), which is ideal for high-throughput, continuous traffic streams where failure rate per unit of time provides a more accurate reflection of real-time service degradation.
