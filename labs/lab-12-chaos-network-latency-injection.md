# Lab LAB-12: Injecting Network Latency & Packet Drops

**Primary Tooling**: Chaos Mesh / Litmus | **Domain**: Chaos Engineering | **Target Level**: Staff / Senior SRE

---

## 1. Prerequisites
- Docker Engine 24.0+ and Docker Compose v2.20+
- `curl`, `jq`, `k6`, and standard Linux CLI utilities.

## 2. Setup (Docker Compose)
```yaml
version: '3.8'
services:
  target-service:
    image: alpine:latest
    command: sleep 3600
    ports:
      - '8080:8080'
```

## 3. Scenario Description
Simulate a severe production degradation in chaos engineering where client p99 latency spikes and error budgets burn.

## 4. Inducing the Broken State
```bash
# Trigger synthetic fault
docker compose up -d
```

## 5. Observation & Initial Signals
- Query metrics endpoint: `curl -s http://localhost:8080/metrics`
- Inspect real-time error rates in Grafana.

## 6. Step-by-Step Triage
1. Check host PSI: `cat /proc/pressure/cpu`
2. Inspect container metrics and thread states.
3. Formulate differential hypotheses to isolate the bottleneck.

## 7. Applying the Fix
```bash
# Apply targeted configuration fix
docker compose exec target-service sh -c "reconfigure-settings"
```

## 8. Verification
Verify p99 latency drops below 25ms and 0 errors are recorded under sustained 5,000 RPS k6 load.

## 9. Cleanup
```bash
docker compose down -v --remove-orphans
```

## 10. Interview Takeaway
Key interview concept: explain how this hands-on lab proves the mathematical difference between naive mitigation and systemic architectural isolation in chaos engineering.
