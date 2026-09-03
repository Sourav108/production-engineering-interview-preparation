# Module 25: Production Security & Secret Operations

## Learning Objectives

By the end of this module, you will be able to:
- Treat security as an **Operational Engineering Discipline** across runtime infrastructure.
- Implement zero-trust **Secrets Management (HashiCorp Vault / AWS Secrets Manager)** with short-lived dynamic credentials.
- Automate **TLS / mTLS Certificate Lifecycle & Zero-Downtime Rotation** using `cert-manager` and SPIFFE/SPIRE.
- Execute rapid containment and forensic auditing during **Production Credential-Compromise Incidents**.

---

## Lessons in This Module

| File | Topic | Focus |
| :--- | :--- | :--- |
| [01-secrets-management-and-cert-rotation.md](01-secrets-management-and-cert-rotation.md) | Secrets Management & Cert Rotation | Dynamic ephemeral credentials, Vault agent sidecars, automated cert renewals |
| [02-credential-compromise-and-incident-response.md](02-credential-compromise-and-incident-response.md) | Credential Compromise & Security Response | Blast radius containment, revoking AWS IAM/DB keys, forensic audit logging |
