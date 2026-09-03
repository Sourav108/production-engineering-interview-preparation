# Module 27: Reliability Testing & Chaos Engineering

## Learning Objectives

By the end of this module, you will be able to:
- Design and execute **Safe, Controlled Chaos Engineering Experiments** following the standard scientific method: **Steady State $\to$ Hypothesis $\to$ Blast Radius Guardrails $\to$ Failure Injection $\to$ Automated Abort $\to$ Verification**.
- Inject realistic distributed faults: **Pod Kills, Network Latency / Packet Drops, Database Primary Failovers, CPU Stress, and Disk I/O Saturation** using Chaos Mesh / Litmus.
- Lead high-impact **Production GameDays (Wheel of Misfortune)** to train engineering teams.
- Establish strict **Abort Conditions** to ensure chaos tests never breach customer SLOs.

---

## Lessons in This Module

| File | Topic | Focus |
| :--- | :--- | :--- |
| [01-chaos-engineering-principles-and-game-days.md](01-chaos-engineering-principles-and-game-days.md) | Chaos Engineering Principles & GameDays | Scientific method in chaos, steady-state metrics, leading GameDays |
| [02-controlled-failure-injection-and-abort-rules.md](02-controlled-failure-injection-and-abort-rules.md) | Controlled Failure Injection & Abort Rules | Network delay injection, pod kill experiments, automated emergency abort triggers |
