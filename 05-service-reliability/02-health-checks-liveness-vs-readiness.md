# 02. Liveness, Readiness, and Startup Probes

## 1. Problem
Misconfigured health checks are one of the most common causes of cascading outages: when a downstream database slows down, naive liveness probes fail across all app pods simultaneously, causing Kubernetes to kill and restart every pod in a thundering herd death spiral.

## 2. Production Context
Kubernetes provides three distinct probe mechanisms. Production engineers must understand the exact semantics, failure actions, and isolation boundaries of each probe.

## 3. Mental Model

| Probe Type | The Production Question It Answers | Action Taken on Failure | Typical Failure Trigger |
| :--- | :--- | :--- | :--- |
| **`startupProbe`** | *Has the slow application finished initial boot/warmup?* | Pauses liveness checks; kills pod if `failureThreshold` exceeded | JVM classloading, cache pre-warming |
| **`readinessProbe`**| *Is the pod currently ready to accept user network traffic?* | **Removes pod IP from Load Balancer Service Endpoints** (Does NOT restart!) | Thread pool saturated, warmup in progress |
| **`livenessProbe`** | *Is the process in an unrecoverable deadlock / hung state?* | **KILLS the container and restarts it** (`SIGTERM → SIGKILL`) | Fatal JVM thread deadlock, infinite loop |

---

## 4. The Critical Anti-Pattern: Checking External Dependencies in Liveness Probes

```
                                The Liveness Death Spiral
 ──────────────────────────────────────────────────────────────────────────────────────────
 1. Database experiences brief CPU spike → Query latency increases from 5ms to 5,000ms.
 2. Pod Liveness Probe executes: SELECT 1 FROM database → Query times out after 1,000ms.
 3. Liveness probe fails on all 50 application pods simultaneously.
 4. Kubernetes Kills all 50 pods at the exact same second!
 5. 50 new pods start up together → Execute DB connection handshakes & schema validations.
 6. Database CPU hits 100% under connection avalanche → Outage becomes permanent!
```

**The Invariant Rule**:
- **Liveness Probes must ONLY test local process health** (e.g. is the event loop responsive, is there a thread deadlock?). **NEVER ping databases, Redis, or third-party APIs in a liveness probe!**
- **Readiness Probes test whether the pod can serve traffic right now**. If an essential local cache is empty, fail readiness (shed traffic) until warmed up.

---

## 5. Production Kubernetes Probe Configuration

```yaml
spec:
  containers:
    - name: api-service
      image: api-service:v2.4.0
      
      # 1. Startup Probe: Allows up to 60s for slow JVM boot before liveness starts
      startupProbe:
        httpGet:
          path: /actuator/health/liveness
          port: 8080
        initialDelaySeconds: 5
        periodSeconds: 5
        failureThreshold: 12 # 12 * 5s = 60s max startup window

      # 2. Liveness Probe: Checks ONLY local deadlock / event loop health
      livenessProbe:
        httpGet:
          path: /actuator/health/liveness
          port: 8080
        periodSeconds: 10
        timeoutSeconds: 2
        failureThreshold: 3

      # 3. Readiness Probe: Checks connection pool and local readiness
      readinessProbe:
        httpGet:
          path: /actuator/health/readiness
          port: 8080
        periodSeconds: 5
        timeoutSeconds: 2
        failureThreshold: 2
```

---

## 6. Interview Questions & Model Answers

**Q1: Why should you never check an external database in a Kubernetes liveness probe?**
**Answer**: A liveness probe failure causes Kubernetes to kill and restart the container. If the downstream database is overloaded or experiencing a network partition, the liveness check will fail across all application pods simultaneously. Kubernetes will terminate the entire fleet at once. When the restarted pods boot up concurrently, their initialization handshakes and connection pool creations will hammer the already struggling database in a thundering herd, transforming a minor database slow-down into a total, unrecoverable cascading outage. Database health should only influence readiness (or be handled via circuit breakers), while liveness must strictly test local process responsiveness.

**Q2: What is the purpose of a `startupProbe` compared to `initialDelaySeconds`?**
**Answer**: `initialDelaySeconds` hardcodes a fixed delay before liveness probes begin, which wastes time on fast startups and fails if a container occasionally takes longer to initialize under cluster load. A `startupProbe` polls the container during boot: once it succeeds, liveness checks take over immediately. If it takes longer (e.g. warming a local cache), the `startupProbe` extends the grace period without triggering premature liveness kills.
