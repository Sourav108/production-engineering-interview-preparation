# 02. Controlled Failure Injection & Automated Abort Rules

## 1. Problem
An uncontrolled chaos experiment in production causes real customer checkouts to fail, breaching company SLAs because nobody configured an automated abort condition to roll back the test.

## 2. Production Context
Chaos testing in production must operate within strict **Blast Radius Bounds** and include an **Automated Dead-Man Switch (Emergency Abort)**.

## 3. Chaos Mesh Failure Injection Specification (Network Latency)

```yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: NetworkChaos
metadata:
  name: payment-delay-experiment
  namespace: production
spec:
  action: delay
  mode: fixed
  value: '1' # Target exactly 1 pod only (Bounded Blast Radius!)
  selector:
    namespaces:
      - production
    labelSelectors:
      app: payment-service
  delay:
    latency: '500ms'
    jitter: '50ms'
    correlation: '50'
  duration: '5m' # Automatically aborts and cleans up after 5 minutes!
```

---

## 4. The 3 Automated Abort Guardrails

Every automated chaos controller must continuously evaluate:
1. **Error Rate Abort**: If global HTTP 5xx error rate $> 0.5\% \implies$ **Instant Abort & Cleanup**.
2. **SLO Error Budget Burn Abort**: If 1-hour error budget burn rate $> 6.0 \implies$ **Instant Abort**.
3. **Manual Big Red Button**: A human engineer must be able to cancel the experiment with a single keystroke (`kubectl delete networkchaos payment-delay-experiment`).

---

## 5. Interview Questions & Model Answers

**Q1: What safety guardrails must be in place before executing a chaos experiment in a production environment?**
**Answer**:
1. **Baseline Steady-State Definition**: Clear metrics defining normal health (e.g. p99 latency $< 200\text{ms}$ and 5xx error rate $< 0.01\%$).
2. **Bounded Blast Radius**: Restrict the fault to a single canary pod, a specific non-critical tenant, or $\le 1\%$ of traffic.
3. **Automated Abort Trigger**: An automated watchdog that terminates the experiment and cleans up network/kernel rules if error rates exceed a safety threshold (e.g. $> 0.5\%$).
4. **Time-Bounded Execution**: Hard TTLs on the fault injection (e.g. max 5 minutes).
5. **On-Call Alignment**: Schedule during peak team staffing (never on Friday evening or during major sales).
