# 02. Incident Command Roles, War Rooms, and Communication Cadence

## 1. Problem
During major incidents, executive leaders and stakeholders join the war room call demanding immediate updates, interrupting engineers actively working on mitigation and causing chaotic crosstalk.

## 2. Production Context
Modeled after the FEMA Incident Command System (ICS), technology companies use a clear separation of roles during Sev-1 incidents to shield investigating engineers from distractions.

## 3. Mental Model: The 4 Core Incident Command Roles

```
                                Incident Command Hierarchy
                                            │
                               ┌────────────┴────────────┐
                               ▼                         ▼
                      Incident Commander (IC)     Scribe / Historian
                      Leads strategy, assigns     Logs timestamps, actions,
                      tasks, enforces cadence.    and hypothesis outcomes.
                               │
               ┌───────────────┴───────────────┐
               ▼                               ▼
       Operations Lead                 Communications Lead
       Executes technical triage       Drafts external status page
       and mitigation commands.        and executive Slack updates.
```

---

## 4. The 15-Minute Structured Communication Cadence

Every 15 minutes during a Sev-1 incident, the Communications Lead posts a standard formatted status update to the internal incident channel:

### Production Status Update Template:
```
[INCIDENT STATUS UPDATE #3] - 14:45 UTC
- Severity: Sev-1 (Critical)
- Incident Commander: @alex (Staff SRE)
- Current Impact: ~8% of EU checkout requests returning HTTP 504.
- Current Working Hypothesis: Cache stampede on payment-gateway token cache after DB replica restart.
- Actions Taken in Last 15m: Applied token-bucket rate limiting at ingress (shedding 10% load).
- Next Actions Planned (Next 15m): Pre-warming Redis cache keys via emergency script; scaling pod replicas to 80.
- Next Update: 15:00 UTC (or immediately on status change).
```

---

## 5. Interview Questions & Model Answers

**Q1: Why should the Incident Commander (IC) NOT execute terminal commands or write code during an active Sev-1 incident?**
**Answer**: The Incident Commander's role is to maintain the high-level strategic overview of the incident: tracking hypotheses, preventing multiple engineers from executing conflicting production changes, ensuring communication cadences are met, and deciding when to pivot mitigation strategies. If the IC begins executing terminal commands or deep-diving code, they develop "tunnel vision," losing situational awareness of the broader system state, failing to coordinate the team, and slowing down overall resolution.

**Q2: How do you handle an executive who joins an active incident war room asking for immediate ETAs?**
**Answer**: As Incident Commander, I politely intervene and state: *"We are currently executing mitigation steps to restore availability. Our Communications Lead is posting official updates every 15 minutes in the #incident-sev1 channel. To allow the engineering team to focus on restoring service without distraction, please direct all questions to the comms channel."* This protects the engineers while keeping stakeholders informed.
