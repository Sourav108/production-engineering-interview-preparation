# Module 04: Networking & the Request Path

## Learning Objectives

By the end of this module, you will be able to:
- Trace a production HTTP/gRPC request end-to-end across every hop: **DNS $\to$ Edge CDN $\to$ TCP Handshake $\to$ TLS Termination $\to$ Load Balancer / Proxy $\to$ Gateway $\to$ Application $\to$ Cache $\to$ Database**.
- Diagnose low-level networking failures: DNS TTL caching issues, TCP SYN packet drops, connection backlog overflows, and TLS handshake timeouts.
- Configure and tune edge and reverse-proxy timeouts (Connect Timeout, Idle Timeout, Socket Read Timeout) to prevent client-side hanging.

---

## Lessons in This Module

| File | Topic | Focus |
| :--- | :--- | :--- |
| [01-end-to-end-request-path.md](01-end-to-end-request-path.md) | The End-to-End Request Path | Hop-by-hop latency decomposition, proxy layers, socket queues |
| [02-dns-tcp-and-tls-failure-modes.md](02-dns-tcp-and-tls-failure-modes.md) | DNS, TCP & TLS Failures | DNS negative caching, TCP 3-way handshake, SYN floods, TLS cert expiry |
| [03-load-balancers-and-timeouts.md](03-load-balancers-and-timeouts.md) | Load Balancers & Timeouts | L4 vs L7 balancing, connection pooling, timeout cascading, HTTP 504 triage |
