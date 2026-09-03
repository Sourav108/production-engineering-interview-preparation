# 03. OpenTelemetry and Distributed Tracing Architecture

## 1. Problem
In a microservices mesh where a single client click flows through 15 services, logs and metrics show *that* an error or latency spike occurred, but cannot reveal which specific RPC hop, database query, or thread lock caused the delay.

## 2. Production Context
**Distributed Tracing** creates a causal DAG (Directed Acyclic Graph) of operations across network and process boundaries, capturing exact start/end timestamps and error context per hop.

## 3. Mental Model
$$\begin{aligned}
\mathbf{Trace} &\implies \text{The end-to-end journey of a single request through the entire distributed system} \\
\mathbf{Span} &\implies \text{A single contiguous block of work executed by a specific service/component (with start/end time)} \\
\mathbf{Context\ Propagation} &\implies \text{Injecting and extracting trace metadata across network wire protocols (W3C TraceContext)}
\end{aligned}$$

## 4. System Diagram: The W3C TraceContext Wire Standard
```mermaid
sequenceDiagram
    autonumber
    participant Client
    participant Gateway as API Gateway
    participant OrderSvc as Order Service
    participant PaySvc as Payment Service
    participant DB as PostgreSQL

    Client->>Gateway: HTTP POST /v1/orders
    Note over Gateway: Gateway generates TraceId: 4bf92f3577b3... & Root SpanId: 00f067aa...
    Gateway->>OrderSvc: Forward with Header: traceparent: 00-4bf92f3577b3-00f067aa-01
    Note over OrderSvc: Extracts context; creates Child Span (Parent: 00f067aa)
    OrderSvc->>PaySvc: HTTP POST /charges with traceparent header
    Note over PaySvc: Creates Child Span
    PaySvc->>DB: Execute SQL UPDATE balance (Span: db.query)
    DB-->>PaySvc: SQL OK (4ms)
    PaySvc-->>OrderSvc: 200 OK
    OrderSvc-->>Gateway: 200 OK
    Gateway-->>Client: 200 OK
```

---

## 5. The W3C `traceparent` Header Specification
$$\texttt{traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01}$$
- `00`: Protocol version (current standard: `00`).
- `4bf92f3577b34da6a3ce929d0e0e4736`: **16-byte Trace ID** (globally unique across all microservices).
- `00f067aa0ba902b7`: **8-byte Parent Span ID**.
- `01`: **Trace Flags** (`01` = Sampled; `00` = Not Sampled).

---

## 6. Sampling Strategies: Head vs. Tail Sampling

| Dimension | Head-Based Sampling | Tail-Based Sampling (Collector / Proxy) |
| :--- | :--- | :--- |
| **Where Decided** | Edge Gateway / Client at request start | OpenTelemetry Collector after request finishes |
| **Decision Rule** | Fixed probabilistic percentage (e.g. 1% of all requests) | **Sample 100% of errors (5xx) + 100% of slow requests (>1s)** + 0.1% normal |
| **Storage Cost** | Predictable, bounded | Optimized for high signal-to-noise ratio |
| **Risk** | May drop rare critical production error traces | Requires OpenTelemetry Collector memory buffering |

---

## 7. Interview Questions & Model Answers

**Q1: How does distributed context propagation work across asynchronous message queues (e.g. Kafka or RabbitMQ)?**
**Answer**: In asynchronous message queues, context cannot be propagated via HTTP headers. Instead, OpenTelemetry injects the `traceparent` and `baggage` metadata into the **message record headers** (e.g. Kafka `RecordHeaders` / RabbitMQ message properties). When the consumer worker reads the message from the topic, the OpenTelemetry SDK extracts the trace context from the message headers, establishes the message producer span as the parent (or as a `Link` reference), and continues the distributed trace DAG seamlessly across the asynchronous boundary.

**Q2: What is the difference between a Trace Span Child relationship and a Span Link?**
**Answer**: A **Child Span** represents a direct synchronous or causal hierarchy where a parent operation invoked and waited for (or directly initiated) a sub-task. A **Span Link** connects a span to one or more unrelated traces without implying direct parenthood—crucial for batch processing operations (e.g. a worker thread pulling 100 distinct order messages from a queue and processing them in a single batch transaction).
