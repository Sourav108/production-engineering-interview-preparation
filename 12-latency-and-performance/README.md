# Module 12: Latency & Performance Engineering

## Learning Objectives

By the end of this module, you will be able to:
- Deconstruct performance bottlenecks using the **Latency Decomposition Model**: $\text{Latency} = \text{Network} + \text{Queue} + \text{Application} + \text{Dependency} + \text{Database}$.
- Apply **Little's Law** ($L = \lambda W$) and **Kingman's Formula** to calculate queueing saturation and concurrency requirements.
- Mitigate **Tail Latency Amplification** in fan-out distributed architectures using hedged requests and tied requests.

---

## Lessons in This Module

| File | Topic | Focus |
| :--- | :--- | :--- |
| [01-latency-decomposition-model.md](01-latency-decomposition-model.md) | Latency Decomposition Model | Network, Queue, App, Dependency, and Database latency components |
| [02-queueing-theory-and-littles-law.md](02-queueing-theory-and-littles-law.md) | Queueing Theory & Little's Law | $L = \lambda W$, Kingman's formula, utilization vs queue wait times |
| [03-tail-latency-amplification.md](03-tail-latency-amplification.md) | Tail Latency Amplification | Fan-out explosion math, Hedged Requests, speculative execution |
