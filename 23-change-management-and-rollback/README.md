# Module 23: Change Management & Safe Rollbacks

## Learning Objectives

By the end of this module, you will be able to:
- Apply the **Rollback vs. Fix-Forward Decision Framework** during active production outages.
- Architect non-destructive, backward-compatible database schema migrations using the **Expand / Contract (Parallel Run) Pattern**.
- Compare deployment rollback mechanics across **Rolling Update, Blue-Green, and Canary** strategies.
- Enforce strict database change safety to guarantee that application code rollbacks never crash due to missing database columns.

---

## Lessons in This Module

| File | Topic | Focus |
| :--- | :--- | :--- |
| [01-rollback-vs-forward-fix-decision-framework.md](01-rollback-vs-forward-fix-decision-framework.md) | Rollback vs. Fix-Forward Decision Framework | The 10-minute rule, automated rollback criteria, blast radius control |
| [02-expand-and-contract-database-migrations.md](02-expand-and-contract-database-migrations.md) | Expand & Contract Database Migrations | Safe schema refactoring, dual-writing, backward compatibility |
