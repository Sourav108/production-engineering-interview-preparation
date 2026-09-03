# 02. Credential Compromise, Blast Radius Containment & Incident Response

## 1. Problem
An engineer accidentally commits an AWS IAM Admin Access Key or production database superuser password to a public GitHub repository. Within 4 minutes, automated threat crawlers discover the key and begin spinning up unauthorized crypto-mining instances and accessing customer databases.

## 2. Production Context
A security incident is a high-severity production outage. Response must be rapid, decisive, and follow a strict containment sequence.

## 3. Mental Model: The Security Incident Containment Protocol
$$\mathbf{Revoke\ Credential} \longrightarrow \mathbf{Isolate\ Compromised\ Nodes} \longrightarrow \mathbf{Audit\ CloudTrail\ / \ Database\ Logs} \longrightarrow \mathbf{Assess\ Data\ Exfiltration} \longrightarrow \mathbf{Restore}$$

---

## 4. Immediate Step-by-Step Response Protocol

1. **Step 1: Immediate Invalidation (0–2 mins)**
   - Deactivate / delete the leaked IAM Access Key via AWS CLI:
     `aws iam update-access-key --access-key-id AKIA... --status Inactive`
   - Rotate database password and terminate all active sessions connected with the compromised user:
     `SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE usename = 'compromised_user';`
2. **Step 2: Threat Quarantine (2–10 mins)**
   - Attach a deny-all IAM inline policy to the compromised role/user.
   - Quarantine any EC2 instance or pod created by the attacker using network security groups.
3. **Step 3: Forensic Audit Investigation**
   - Query AWS CloudTrail / SIEM logs for all API calls made by the leaked access key within the last 48 hours:
     `aws cloudtrail lookup-events --lookup-attributes AttributeKey=AccessKeyId,AttributeValue=AKIA...`
4. **Step 4: Legal & Notification Assessment**
   - Determine if PII, payment tokens, or proprietary data were accessed or exfiltrated for regulatory breach reporting (GDPR / HIPAA).

---

## 5. Interview Questions & Model Answers

**Q1: Walk me through your immediate operational actions upon discovering that a production AWS IAM admin key was leaked publicly.**
**Answer**:
1. **Deactivate immediately**: I immediately deactivate the access key via AWS CLI/console to block further API calls.
2. **Attach Deny-All Boundary**: Attach an explicit `Deny *` inline policy to the associated IAM user/role.
3. **Revoke Active Sessions**: Invalidate any temporary STS session tokens issued by the key (`aws iam put-user-policy --policy-name RevokeOlderSessions`).
4. **Audit CloudTrail**: Inspect CloudTrail logs for all events initiated by that `AccessKeyId` to identify every resource modified, created, or read.
5. **Terminate Unauthorized Resources**: Terminate any unauthorized instances, lambda functions, or security group rules created by the attacker.
6. **Rotate Credentials & Conduct Postmortem**: Generate new credentials, update CI/CD pipelines, and conduct a blameless postmortem to implement automated secret scanning pre-commit hooks (e.g. `gitleaks` / `trufflehog`).
