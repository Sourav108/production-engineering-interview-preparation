# Module 22: Release Engineering & Canary Deployments

## Learning Objectives

By the end of this module, you will be able to:
- Transform risky "all-at-once" production deployments into **Safe Controlled Scientific Experiments**.
- Architect automated **Canary Deployment Pipelines** ($1\% \to 5\% \to 25\% \to 100\%$) with real-time metric analysis and automated abort thresholds.
- Implement **Feature Flags & Emergency Kill Switches** to decouple code deployment from feature exposure.
- Enforce pre-deployment and post-deployment validation gates.

---

## Lessons in This Module

| File | Topic | Focus |
| :--- | :--- | :--- |
| [01-deployment-risks-and-canary-rollouts.md](01-deployment-risks-and-canary-rollouts.md) | Canary Deployments & Automated Analysis | Staged rollouts, baseline vs canary metrics, Argo Rollouts |
| [02-feature-flags-and-kill-switches.md](02-feature-flags-and-kill-switches.md) | Feature Flags & Emergency Kill Switches | Decoupling release from deploy, circuit-breaker flags, technical debt |
