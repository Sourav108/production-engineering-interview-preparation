# 03. Graceful Shutdown, Draining, and Container Lifecycle

## 1. Problem
During rolling deployments, pods are terminated while still processing client requests, causing HTTP 502/504 errors, broken database transactions, and dropped connections for active users.

## 2. Production Context
In Kubernetes, terminating a pod involves two asynchronous, concurrent workflows:
1. **Network Endpoint Deregistration**: The Kubernetes control plane removes the Pod IP from the Service `Endpoints` / `EndpointSlice` object and updates load balancers (kube-proxy / Ingress Envoy).
2. **Container Process Termination**: The kubelet sends a `SIGTERM` signal to the container process.

Because endpoint propagation across distributed load balancers takes several seconds, if the container shuts down immediately upon receiving `SIGTERM`, it will reject in-flight requests that are still being routed to it!

## 3. Mental Model
$$\text{Graceful Shutdown Sequence:}$$
$$\mathbf{preStop\ Hook\ Sleep} \to \mathbf{SIGTERM} \to \mathbf{Stop\ Accepting\ New\ Conns} \to \mathbf{Drain\ In\text{-}Flight\ Work} \to \mathbf{Close\ DB\ Pools} \to \mathbf{Exit\ 0}$$

## 4. System Diagram
```mermaid
sequenceDiagram
    autonumber
    participant Kube as Kubernetes Control Plane
    participant Ingress as Ingress / Load Balancer
    participant Pod as Application Pod (Tomcat / JVM)

    Kube->>Ingress: 1. Remove Pod IP from Endpoints (Takes ~3-5s to propagate)
    Kube->>Pod: 2. Execute preStop Hook (sleep 5s)
    Note over Pod: Pod continues serving in-flight & late arriving requests!
    Ingress-->>Pod: 3. Last in-flight request routed
    Note over Kube: 4. preStop finishes; Kube sends SIGTERM
    Pod->>Pod: 5. Stop accepting new sockets; drain active threads (e.g. 15s)
    Pod->>Pod: 6. Flush logs, close HikariCP DB pool
    Pod-->>Kube: 7. Process Exits cleanly (Exit 0)
```

---

## 5. Production Kubernetes Pod Configuration

```yaml
spec:
  terminationGracePeriodSeconds: 45 # Maximum time allowed before SIGKILL (9)
  containers:
    - name: backend-api
      image: backend-api:v1.2.0
      lifecycle:
        preStop:
          exec:
            command: ["/bin/sh", "-c", "sleep 5"] # Allows load balancer endpoints to drain
```

### Spring Boot / Application Graceful Shutdown Configuration
```yaml
server:
  shutdown: graceful # Pauses new connections, allows in-flight requests to complete

spring:
  lifecycle:
    timeout-per-shutdown-phase: 20s # Max drain timeout for worker thread pools
```

---

## 6. Interview Questions & Model Answers

**Q1: Why is a `preStop: exec: command: ["sleep", "5"]` hook necessary in Kubernetes even if the application implements graceful shutdown?**
**Answer**: When Kubernetes terminates a pod, removing the pod's IP from the `EndpointSlice` and updating all iptables rules / Ingress proxy routing tables across the cluster is an **asynchronous distributed operation** that takes 2 to 5 seconds. If the application handles `SIGTERM` by immediately closing its listening socket, there is a multi-second race condition where ingress proxies are still routing traffic to the pod, resulting in connection resets (`RST`) and HTTP 502 errors. The `preStop` sleep delays `SIGTERM`, ensuring the pod continues accepting and serving traffic until all proxies have completed endpoint deregistration.

**Q2: What is the difference between `SIGTERM` and `SIGKILL`?**
**Answer**: `SIGTERM (Signal 15)` is a graceful termination request that can be caught, handled, and deferred by the application, allowing it to complete in-flight requests, flush log buffers, and close database connections. `SIGKILL (Signal 9)` is handled directly by the kernel and **cannot be caught or ignored**; the kernel abruptly destroys the process's address space and file descriptors, which can leave external state in an inconsistent condition if transactions were mid-flight. Kubernetes sends `SIGTERM`, waits for `terminationGracePeriodSeconds`, and only sends `SIGKILL` if the process fails to terminate within that window.
