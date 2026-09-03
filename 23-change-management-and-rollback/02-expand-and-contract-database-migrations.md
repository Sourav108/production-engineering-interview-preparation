# 02. Expand & Contract (Parallel Run) Database Migrations

## 1. Problem
A developer renames a database column from `phone_number` to `phone_e164`. The application crashes immediately upon deployment, and rolling back the application code *also* crashes because the old code expects `phone_number`, which no longer exists!

## 2. Production Context
Application code and database schemas cannot be deployed in a single atomic instantaneous lockstep. All schema modifications must be executed through the **Expand and Contract (Parallel Run) Pattern** across multiple deployment phases.

## 3. Mental Model: The 4-Phase Expand/Contract Sequence

```mermaid
flowchart TD
    subgraph Phase 1: Expand [Backward-Compatible Schema Change]
        P1[1. Add new column phone_e164 nullable alongside old column phone_number]
    end

    subgraph Phase 2: Dual-Writing [Deploy App v2.0]
        P2[2. App writes to BOTH columns; reads from phone_number with fallback to phone_e164]
    end

    subgraph Phase 3: Backfill [Async Data Migration]
        P3[3. Run background batch worker to copy legacy phone_number data to phone_e164]
    end

    subgraph Phase 4: Contract [Deploy App v3.0 & Drop Legacy]
        P4[4. App reads/writes ONLY phone_e164. Drop column phone_number safely!]
    end

    Phase 1 --> Phase 2 --> Phase 3 --> Phase 4
```

---

## 4. Why This Guarantees Safe Rollback
- If App v2.0 experiences a critical bug during Phase 2, rolling back to App v1.0 is **100% safe** because the legacy `phone_number` column was never dropped and continues receiving writes.

---

## 5. Interview Questions & Model Answers

**Q1: How do you rename a column on a multi-terabyte database table in production with zero downtime?**
**Answer**: Renaming a column directly (`ALTER TABLE users RENAME COLUMN a TO b`) breaks running application instances and prevents rollbacks. I execute the **Expand and Contract** pattern across four separate deployments:
1. **Expand**: Add the new column `b` as nullable alongside `a`.
2. **Dual-Write**: Deploy application code that writes to both `a` and `b`, but continues reading from `a`.
3. **Backfill**: Run a throttled background script to migrate existing historic rows from `a` to `b` in small batches (e.g. 1,000 rows at a time) to avoid locking.
4. **Switch Read**: Deploy code that reads and writes exclusively from `b`.
5. **Contract**: Once verified and old code is retired, drop the old column `a`.
