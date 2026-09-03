# Module 02: Linux Internals & System Behavior

## Learning Objectives

By the end of this module, you will be able to:
- Rapidly diagnose a heavily stressed Linux production host using the standard CLI triage toolchain.
- Explain the physical mechanics of Linux **CPU scheduling, Load Average, Virtual Memory, Page Cache, File Descriptors, and Network Sockets**.
- Query the `/proc` pseudo-filesystem and interpret **Pressure Stall Information (PSI)** to identify exact resource starvation bottlenecks.
- Trace system calls and kernel-user space transitions using `strace` and `perf`.

---

## Lessons in This Module

| File | Topic | Focus |
| :--- | :--- | :--- |
| [01-reading-a-stressed-linux-system.md](01-reading-a-stressed-linux-system.md) | Reading a Stressed Linux System | Load average, run queue vs disk wait, CPU user vs system vs iowait |
| [02-proc-filesystem-and-pressure-stall-info.md](02-proc-filesystem-and-pressure-stall-info.md) | `/proc` & Pressure Stall Info (PSI) | `/proc/cpuinfo`, `/proc/meminfo`, `/proc/pressure/*`, cgroups v2 |
| [03-linux-cli-triage-toolchain.md](03-linux-cli-triage-toolchain.md) | The SRE Linux CLI Toolchain | `top`, `ps`, `free`, `vmstat`, `iostat`, `ss`, `lsof`, `sar`, `strace`, `journalctl` |
