# 02. Dependency Latency Budgets & Blast Radius Containment

## 1. Problem
An upstream orchestrator has a 500ms client SLA. It calls 4 downstream microservices in serial. When downstream Service C slows down from 20ms to 400ms, the orchestrator breaches its SLA and cascades timeouts to the client.

## 2. Production Context
In distributed microservices, latency budgets must be explicitly partitioned across the downstream call graph.

## 3. Mental Model: The Distributed Latency Budget Tree

$$\mathbf{SLA}_{\text{Client}} = 500\text{ms}$$
$$\text{Reserved Orchestrator Overhead (Serialization + Routing)} = 50\text{ms}$$
$$\mathbf{Total\ Available\ Downstream\ Budget} = 500 - 50 = \mathbf{450\text{ms}}$$

```
                       Distributed Latency Budget Tree
 ──────────────────────────────────────────────────────────────────────────────────────────
 Service A (User Auth)          ──► Budget: 50ms  (Hard Timeout: 50ms)
 Service B (Inventory Check)   ──► Budget: 100ms (Hard Timeout: 100ms)
 Service C (Pricing Engine)     ──► Budget: 150ms (Hard Timeout: 150ms)
 Service D (Tax Calculation)    ──► Budget: 100ms (Hard Timeout: 100ms)
 ──────────────────────────────────────────────────────────────────────────────────────────
 Sum of Budgets: 50 + 100 + 150 + 100 = 400ms (< 450ms Available Budget! Guaranteed Safe!)
```

---

## 4. Deadline Propagation (gRPC / HTTP `X-Request-Deadline`)
Instead of static per-hop timeouts, pass an absolute **Deadline Timestamp** down the call chain:

$$\text{Deadline} = \text{Current Time} + 500\text{ms}$$
- Hop 1 consumes 100ms $\implies$ Remaining budget passed to Hop 2: **400ms**.
- Hop 2 consumes 250ms $\implies$ Remaining budget passed to Hop 3: **150ms**.
- If Hop 3 receives a request where `Remaining Budget <= 0ms`, it **immediately cancels and drops the request** rather than wasting CPU on a response the client has already abandoned!

---

## 5. Interview Questions & Model Answers

**Q1: What is Deadline Propagation, and how does it prevent wasted backend computation in distributed systems?**
**Answer**: Deadline Propagation (implemented natively in gRPC via Context deadlines and in HTTP via `Request-Timeout` / `X-Request-Deadline` headers) transmits the total remaining time-to-live for a request across every downstream service in the call tree. If an upstream service or user cancels the request or if earlier hops consume the entire latency budget, downstream services can inspect the deadline before initiating expensive operations (like complex database queries). If the deadline has already expired, the downstream service immediately short-circuits and aborts execution, saving compute, memory, and database IOPS from processing orphaned requests.
