# Module 15: Dependency Management & Failure Propagation

## Learning Objectives

By the end of this module, you will be able to:
- Map and categorize service dependencies into **Critical vs. Non-Critical (Optional)** tiers.
- Formulate the fundamental production resilience question for every dependency: *"What happens to our user experience when this dependency gets slow or completely dies?"*
- Establish **Dependency Latency & Availability Budgets** to contain failure propagation.
- Implement hard isolation boundaries to prevent third-party vendor slowdowns from freezing core business transactions.

---

## Lessons in This Module

| File | Topic | Focus |
| :--- | :--- | :--- |
| [01-critical-vs-optional-dependencies.md](01-critical-vs-optional-dependencies.md) | Critical vs. Optional Dependencies | Hard vs soft dependencies, graceful fallback paths, failure maps |
| [02-dependency-budgets-and-blast-radius.md](02-dependency-budgets-and-blast-radius.md) | Dependency Budgets & Blast Radius | Latency budgeting across the call tree, transitive failure isolation |
