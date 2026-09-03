# Module 11: Production Debugging Methodology

## Learning Objectives

By the end of this module, you will be able to:
- Execute a disciplined, evidence-driven production debugging protocol following the exact 12-step sequence: **Symptom $\to$ Scope $\to$ Metrics $\to$ Logs $\to$ Traces $\to$ Recent Changes $\to$ Dependencies $\to$ Hypothesis $\to$ Test $\to$ Root Cause $\to$ Mitigation $\to$ Verification**.
- Formulate and rigorously eliminate differential diagnostic hypotheses using telemetry without executing blind trial-and-error restarts.
- Isolate non-deterministic, intermittent production regressions under high concurrency.

---

## Lessons in This Module

| File | Topic | Focus |
| :--- | :--- | :--- |
| [01-evidence-driven-debugging-methodology.md](01-evidence-driven-debugging-methodology.md) | Evidence-Driven Debugging Methodology | The 12-step triage protocol, avoiding the restart anti-pattern |
| [02-hypothesis-elimination-and-differential-diagnosis.md](02-hypothesis-elimination-and-differential-diagnosis.md) | Hypothesis Elimination & Differential Diagnosis | Systematic hypothesis testing, correlating metrics with logs |
