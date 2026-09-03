# Module 03: Processes, Memory and CPU

## Learning Objectives

By the end of this module, you will be able to:
- Diagnose the four classic host resource incidents from first principles: **CPU Saturation, Memory Leaks & OOM Kills, Thread Pool Starvation, and File Descriptor/Socket Exhaustion**.
- Explain Linux process lifecycles, thread states, and the cost of kernel context switching.
- Analyze the Linux kernel **Out-of-Memory (OOM) Killer** algorithm (`oom_score_adj`, `oom_score`) and prevent container terminations.
- Capture diagnostic artifacts (thread dumps, heap dumps, CPU flamegraphs) **before** taking any mitigating restart action.

---

## Lessons in This Module

| File | Topic | Focus |
| :--- | :--- | :--- |
| [01-process-states-and-threads.md](01-process-states-and-threads.md) | Process States & Thread Pools | Task states (R, S, D, Z, T), context switching, thread pool sizing |
| [02-memory-leaks-and-oom-killer.md](02-memory-leaks-and-oom-killer.md) | Memory Leaks & Linux OOM Killer | Virtual vs Resident memory, JVM native leaks, OOM killer mechanics |
| [03-cpu-saturation-and-profiling.md](03-cpu-saturation-and-profiling.md) | CPU Saturation & Profiling | High CPU user vs system, hot loops, thread profiling & flamegraphs |
| [04-fd-and-socket-exhaustion.md](04-fd-and-socket-exhaustion.md) | FD & Socket Exhaustion | File descriptors, ephemeral port exhaustion, TIME_WAIT, `ulimit` |
