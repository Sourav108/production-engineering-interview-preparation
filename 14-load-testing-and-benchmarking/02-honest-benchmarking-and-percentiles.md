# 02. Honest Benchmarking, Warmup Cycles, and Coordinated Omission

## 1. Problem
Load testing tools report a "p99 latency of 15ms" during a 5-second server freeze, completely hiding the outage due to a subtle measurement error known as **Coordinated Omission**.

## 2. Production Context
Benchmarking results that deceive engineering teams lead to disastrous production rollouts. Senior/Staff engineers must understand the mechanics of measurement bias.

## 3. Mental Model: Gil Tene's Coordinated Omission Problem

Consider a load testing tool configured to send 100 requests per second (1 request every 10ms):
1. **Server Freezes** for **10,000ms (10 seconds)** due to a Full GC pause or database lock.
2. **Naive Load Generator**: Sends request #1 $\to$ Blocks waiting for response for 10 seconds.
3. **The Trap**: During those 10 seconds, the load generator **failed to send the 1,000 scheduled requests that *should* have been sent**!
4. **The Measurement Error**: The tool records only **1 single slow request (10s)** and calculates p99 as fast, completely omitting the 999 requests that would have waited 9.9s, 9.8s, 9.7s...

$$\mathbf{Coordinated\ Omission\ shrinks\ 1,000\ catastrophic\ latency\ samples\ into\ 1\ sample!}$$

---

## 4. The 4 Rules of Honest Benchmarking

1. **Use Open-Model Load Generators**: Use tools that decouple request scheduling from response arrival (e.g. `k6` with arrival-rate executors, `Vegeta`, or `Wrk2` with coordinated omission correction).
2. **Mandatory Warmup Phases**: Discard the first 3–5 minutes of benchmark data to allow JIT compilers (HotSpot C2), connection pools, and database buffer caches to reach steady state.
3. **Never Test from a Saturated Client**: Verify that the load generator machine itself has $< 60\%$ CPU utilization and zero dropped network packets during the test.
4. **Percentile Rigor**: Always report **p50, p90, p95, p99, and p99.9** along with error rates. Never report averages alone!

---

## 5. Interview Questions & Model Answers

**Q1: What is Coordinated Omission in load testing, and how does it distort benchmark results?**
**Answer**: Coordinated Omission occurs in closed-loop load generators where the client waits for a response before issuing the next scheduled request. When the target server pauses (e.g. during a 5-second GC pause), the client stalls and stops generating new requests. Consequently, hundreds of requests that should have arrived and queued during that window are never generated or measured. When the server resumes, the tool records only the single delayed request rather than recording all the queued, delayed requests, artificially deflating reported p99 and p99.9 latency percentiles by orders of magnitude. We prevent this by using open-model load generators (like `k6` constant-arrival-rate executors or `Vegeta`) that issue requests on a strict schedule regardless of server response timing.

**Q2: Why must benchmark results discard the initial JVM warmup period?**
**Answer**: On JVM startup, code executes in interpreted mode or via the Tier 1 C1 compiler. As methods become "hot", the HotSpot C2 compiler applies aggressive optimizations (method inlining, loop unrolling, escape analysis, devirtualization). Furthermore, database buffer pools and OS Page Caches are cold upon initial start. Measuring latency during this phase records cold cache misses and compilation pauses rather than steady-state production behavior. A valid benchmark always includes a dedicated 3–5 minute warmup stage before recording metrics.
