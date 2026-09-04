# 03. Production System Design: Multi-Channel Notification Platform

## 1. System Requirements & Functional Scope
- **Functional**: Ingest, template, rate-limit, and deliver transactional and marketing notifications across **Push (APNs/FCM), Email (SendGrid), SMS (Twilio), and WebSockets**.
- **Throughput Target**: Ingest up to $100,000\text{ Notifications/sec}$; deliver critical transactional OTPs in $\le 2\text{ seconds}$.
- **Resilience Invariants**: **Zero Message Loss (Durability)**, Backpressure protection against external vendor rate limits, Dead-Letter Queues (DLQs) for poison pills.

---

## 2. Architecture & Queue Partitioning Diagram

```mermaid
flowchart TD
    API[Notification Ingestion API] --> KAFKA[Apache Kafka: Partitioned Topics]
    
    subgraph Priority Partitioning
        KAFKA --> TOPIC_CRIT[Topic: High-Priority OTP / 2FA (16 Partitions)]
        KAFKA --> TOPIC_MKT[Topic: Low-Priority Marketing (64 Partitions)]
    end
    
    TOPIC_CRIT --> WORKER_CRIT[Dedicated OTP Consumer Fleet: Instant Delivery]
    TOPIC_MKT --> WORKER_MKT[Throttled Marketing Consumers]
    
    WORKER_CRIT --> TWILIO[Twilio SMS Gateway: Leaky Bucket Rate Limiter]
    WORKER_MKT --> SENDGRID[SendGrid Email: Bulk Dispatch]
    
    WORKER_CRIT -.->|3 Retries Failed| DLQ[(Dead Letter Queue: S3 / PostgreSQL Archive)]
```

---

## 3. Backpressure & Rate-Limiting External Providers
- External SMS/Email providers enforce strict downstream rate limits (e.g. Twilio: 100 SMS/sec per phone number).
- **Leaky Bucket Consumer Throttling**: Kafka consumer workers pull messages using reactive backpressure; if downstream rate limiters encounter HTTP 429, consumer pauses Kafka partition offset polling via `KafkaConsumer.pause()` for 5 seconds without crashing worker memory.

---

## 4. Poison Pill Containment via Dead Letter Queues (DLQs)
- If a malformed notification payload (e.g. invalid unicode character or broken template) throws an unrecoverable exception:
  - Worker retries with exponential backoff up to **3 times**.
  - Upon 3rd failure, the message is routed to the **DLQ Topic**, committing the Kafka offset and allowing the partition stream to continue unblocked.

---

## 5. Trade-offs & Production Defense
- **Delivery Speed vs Infrastructure Cost**: Partitioning Kafka topics by priority guarantees that a 10-million-user promotional email campaign cannot delay a single user's 2-factor authentication SMS code.
