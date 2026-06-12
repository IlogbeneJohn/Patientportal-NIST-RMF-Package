# Document 3 — System Security Plan (SSP)

> **RMF Step 3**
>
> **Why this matters:** The SSP is the master security document for the system. It describes EVERY security control and explains exactly how the organization implements it. Auditors, assessors, and authorizing officials rely entirely on the SSP to decide if the system is safe to operate.

---

## Executive Summary

This System Security Plan (SSP) documents the security controls implemented for PatientPortal v1.0 in accordance with **NIST SP 800-53 Revision 5, High baseline**, and the **HIPAA Security Rule**.

It describes **12 selected controls** across key security domains including access control, identification and authentication, audit logging, incident response, and configuration management. Each control description explains **what** the control is, **why** it matters, and **exactly how** PatientPortal v1.0 implements it — in plain language accessible to technical and non-technical stakeholders.

---

## Scope

This SSP covers all system components within the PatientPortal v1.0 authorization boundary as defined in [Document 1 — System Description](01-system-description.md). It applies to all personnel who develop, operate, maintain, or use the system.

The selected controls represent a subset of the NIST SP 800-53 High baseline, prioritized based on the **FIPS 199 High categorization** and **HIPAA Security Rule** requirements.

## Assets Reviewed

All 10 assets documented in the System Description (A-01 through A-10) are within scope for this SSP. Controls are applied system-wide unless otherwise noted.

---

## Selected Security Controls & Implementation

### AC-2 — Account Management
**Family:** Access Control | **Status:** ● Implemented

| | |
|---|---|
| **What is this control?** | This control requires the organization to formally manage all user accounts — who gets created, who gets modified, and who gets deleted. |
| **Why does it matter?** | Unauthorized or forgotten accounts are one of the most common ways attackers gain access to systems. If a nurse leaves the hospital but their account remains active, a hacker could use it to access 15,000 patient records. |
| **How PatientPortal implements it** | CareFirst uses Okta as the Identity Provider for PatientPortal. HR triggers automated account provisioning and deprovisioning workflows. Accounts are reviewed quarterly by the system owner. All administrative accounts require dual approval before creation. Dormant accounts (no login in 60 days) are automatically disabled. |

---

### AC-6 — Least Privilege
**Family:** Access Control | **Status:** ● Implemented

| | |
|---|---|
| **What is this control?** | Users are given only the minimum access permissions they need to do their specific job — nothing more. |
| **Why does it matter?** | If a help desk worker can access and edit clinical records, that is a much bigger security risk than if they can only see names and contact numbers. Least privilege limits the damage any one account can cause. |
| **How PatientPortal implements it** | PatientPortal uses Role-Based Access Control (RBAC) with six defined roles (see [Document 1](01-system-description.md)). Patients can only see their own records. Clinical staff can only access assigned patient panels. Administrators cannot modify clinical records. Access requests require manager and security team approval. |

---

### IA-2 — Identification and Authentication
**Family:** Identification & Authentication | **Status:** ● Implemented

| | |
|---|---|
| **What is this control?** | Every user must prove who they are before accessing the system, and each person must have a unique identity. |
| **Why does it matter?** | If two people share a login, you cannot tell who made a change to a patient's record. Unique IDs ensure accountability and are required by HIPAA. |
| **How PatientPortal implements it** | All users receive unique usernames in Okta. **Multi-Factor Authentication (MFA)** is mandatory for all access. Patients use email + SMS OTP. Clinical staff use hardware security keys (YubiKey). Shared accounts are strictly prohibited. Session timeouts occur after 15 minutes of inactivity. |

---

### IA-5 — Authenticator Management
**Family:** Identification & Authentication | **Status:** ● Implemented

| | |
|---|---|
| **What is this control?** | Governs how passwords, tokens, and other login credentials are created, protected, and reset. |
| **Why does it matter?** | Weak or reused passwords are the most common entry point in data breaches. Strong password policies significantly reduce this risk. |
| **How PatientPortal implements it** | PatientPortal enforces a minimum **14-character password** with complexity requirements via Okta policy. Passwords expire every 90 days. Previous 12 passwords cannot be reused. Passwords are stored as **bcrypt hashes** (never plaintext). Password reset requires verified email + SMS confirmation. Temporary credentials expire in 24 hours. |

---

### AU-2 — Event Logging
**Family:** Audit and Accountability | **Status:** ● Implemented

| | |
|---|---|
| **What is this control?** | The system records a log of security-relevant events — who logged in, what they accessed, and what actions they took. |
| **Why does it matter?** | Logs are the security camera footage of your IT system. Without them, you cannot investigate a breach, detect unusual behavior, or prove compliance. |
| **How PatientPortal implements it** | AWS CloudWatch captures all authentication events, record access, privilege escalations, and system errors. Logs include: timestamp, user ID, IP address, action performed, data accessed, and outcome (success/fail). Logs are stored in a **write-once S3 bucket** separate from the main system. Log retention is **3 years** per HIPAA requirements. |

---

### AU-6 — Audit Record Review
**Family:** Audit and Accountability | **Status:** ● Implemented

| | |
|---|---|
| **What is this control?** | Security staff must actively review the logs — not just collect them — to identify suspicious or unusual activity. |
| **Why does it matter?** | Collecting logs is useless if nobody reads them. Automated monitoring plus human review catches attacks that automated systems might miss. |
| **How PatientPortal implements it** | AWS GuardDuty provides automated threat detection and anomaly alerting. The Security Analyst reviews daily security dashboards. Weekly automated reports flag: failed login attempts exceeding 5 per hour, off-hours access to PHI, bulk data downloads exceeding 100 records, and access from unexpected geographic locations. Critical alerts trigger PagerDuty notifications to the on-call security team within **15 minutes**. |

---

### CM-6 — Configuration Settings
**Family:** Configuration Management | **Status:** ◑ Partially Implemented

| | |
|---|---|
| **What is this control?** | All systems must be configured securely, using industry-recognized security benchmarks rather than manufacturer default settings. |
| **Why does it matter?** | Default settings are publicly known and often insecure. For example, many servers ship with default admin passwords of "admin" — a hacker's first guess. |
| **How PatientPortal implements it** | All servers are hardened to **CIS Benchmarks Level 2** before deployment. AWS Config continuously monitors for configuration drift. Any deviation from the approved baseline triggers an automated alert. System administrators must document and obtain approval before making any configuration change. Docker container images are scanned for vulnerabilities before deployment using Amazon ECR scanning. |

> ⚠️ **Gap identified:** Detection works, but **auto-remediation is not enabled**. See [POA-002](05-poam.md#poa-002--cm-6--configuration-settings-auto-remediation-gap) and [SAR-F-05](06-security-assessment-report.md#sar-f-05--cm-6--configuration-settings--partial-pass).

---

### IR-4 — Incident Handling
**Family:** Incident Response | **Status:** ● Implemented

| | |
|---|---|
| **What is this control?** | Defines how the organization detects, responds to, and recovers from cybersecurity incidents. |
| **Why does it matter?** | Every organization will eventually face a security incident. The difference between a minor disruption and a catastrophic breach is how quickly and effectively the team responds. |
| **How PatientPortal implements it** | CareFirst maintains a documented **Incident Response Plan (IRP v2.1)**. The plan defines 5 incident severity levels. For PHI breaches, the plan follows the **HIPAA Breach Notification Rule** requirements (notify HHS within 60 days, notify affected patients within 60 days). An Incident Response Team (IRT) is on-call 24/7. Tabletop exercises are conducted semi-annually. Post-incident lessons-learned reports are mandatory. |

---

### SC-8 — Transmission Confidentiality and Integrity
**Family:** System & Communications Protection | **Status:** ● Implemented

| | |
|---|---|
| **What is this control?** | Data transmitted across networks must be encrypted so it cannot be intercepted and read. |
| **Why does it matter?** | Without encryption, health data sent between a patient's browser and the server is visible to anyone monitoring the network — like sending a postcard instead of a sealed letter. |
| **How PatientPortal implements it** | All communications use **TLS 1.3** (minimum TLS 1.2). HTTP is automatically redirected to HTTPS. SSL certificates are managed via AWS Certificate Manager with auto-renewal. API communications use **mutual TLS (mTLS)**. Internal service-to-service communication within AWS is encrypted using VPC endpoints. Weak cipher suites (RC4, DES, 3DES) are explicitly disabled in web server configuration. |

---

### SC-28 — Protection of Information at Rest
**Family:** System & Communications Protection | **Status:** ● Implemented

| | |
|---|---|
| **What is this control?** | Data stored on servers, databases, and backups must be encrypted so it cannot be read if storage media is stolen. |
| **Why does it matter?** | If a hard drive from the database server is stolen and data is not encrypted, the thief can read every patient's records. Encryption ensures stolen data is unreadable without the encryption key. |
| **How PatientPortal implements it** | The PostgreSQL database is encrypted using **AWS RDS encryption (AES-256)**. S3 backup buckets use server-side encryption (SSE-KMS). All encryption keys are managed in AWS KMS with **quarterly rotation**. Admin workstations use BitLocker full-disk encryption. Encryption is validated monthly via automated compliance checks in AWS Security Hub. |

---

### SI-2 — Flaw Remediation (Patch Management)
**Family:** System and Information Integrity | **Status:** ◑ Partially Implemented

| | |
|---|---|
| **What is this control?** | Security vulnerabilities in software must be identified and patched (fixed) within a defined timeframe. |
| **Why does it matter?** | Most successful cyberattacks exploit known vulnerabilities that have existing patches. Organizations that delay patching are leaving known doors unlocked. |
| **How PatientPortal implements it** | AWS Inspector scans all EC2 instances and container images weekly for vulnerabilities. **Critical patches (CVSS 9.0+)** must be applied within **24 hours**. **High patches (CVSS 7.0–8.9)** must be applied within **72 hours**. Moderate and low patches are applied in the monthly patching window. All patches are tested in the staging environment before production deployment. Patch compliance reports are reviewed weekly by the security team. |

> ⚠️ **Gap identified:** Moderate/Low severity patches are overdue. See [POA-001](05-poam.md#poa-001--si-2--flaw-remediation) and [SAR-F-06](06-security-assessment-report.md#sar-f-06--si-2--flaw-remediation-patch-management--fail).

---

### CP-9 — System Backup
**Family:** Contingency Planning | **Status:** ● Implemented

| | |
|---|---|
| **What is this control?** | Regular backups of system data must be created, tested, and stored securely so the system can be restored after a failure or attack. |
| **Why does it matter?** | Ransomware attacks encrypt an organization's data and demand payment. Organizations with tested, offline backups can restore their data without paying. Healthcare data loss can directly harm patients. |
| **How PatientPortal implements it** | Daily automated backups of the PostgreSQL database and application files are performed at **2:00 AM EST**. Backups are stored in AWS S3 (encrypted, versioned, cross-region replicated to us-west-2). Backup restoration is tested **quarterly**. Recovery Point Objective (**RPO**) is **24 hours**. Recovery Time Objective (**RTO**) is **4 hours**. Backup access is restricted to the backup service account only. |

---

## Control Implementation Summary

| Control ID | Control Name | Status | Family |
|---|---|---|---|
| AC-2 | Account Management | ✅ Pass | Access Control |
| AC-6 | Least Privilege | ✅ Pass | Access Control |
| IA-2 | Identification and Authentication | ✅ Pass | Identification & Authentication |
| IA-5 | Authenticator Management | ✅ Pass | Identification & Authentication |
| AU-2 | Event Logging | ✅ Pass | Audit and Accountability |
| AU-6 | Audit Record Review | ✅ Pass | Audit and Accountability |
| CM-6 | Configuration Settings | ⚠️ Partial | Configuration Management |
| IR-4 | Incident Handling | ✅ Pass | Incident Response |
| SC-8 | Transmission Confidentiality and Integrity | ✅ Pass | System & Communications Protection |
| SC-28 | Protection of Information at Rest | ✅ Pass | System & Communications Protection |
| SI-2 | Flaw Remediation (Patch Management) | ⚠️ Partial | System and Information Integrity |
| CP-9 | System Backup | ✅ Pass | Contingency Planning |

---

**Previous:** [← Document 2 — FIPS 199 Categorization Memo](02-fips-199-categorization.md)
**Next:** [Document 4 — Risk Assessment (SP 800-30) →](04-risk-assessment.md)
