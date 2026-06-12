# Document 5 — Plan of Action & Milestones (POA&M)

> **RMF Step 5**
>
> **Why this matters:** The POA&M is the GRC analyst's daily working document. Every security gap that cannot be fixed immediately gets logged here with a deadline, a responsible person, and a status. Auditors treat an up-to-date POA&M as proof that an organization takes security seriously and is actively managing its risks.

---

## Executive Summary

This Plan of Action and Milestones (POA&M) documents **three security control gaps** identified during the System Security Plan review and Risk Assessment for PatientPortal v1.0. Each finding includes a detailed description of the gap, the associated risk, the responsible party, the remediation plan, milestone dates, and current status.

The POA&M is a **living document**, updated monthly by the GRC Analyst.

---

## Scope

This POA&M covers three findings selected from the [Risk Assessment](04-risk-assessment.md) and [SSP review](03-system-security-plan.md):

- Incomplete patch management (**SI-2**)
- Configuration management detect-only posture (**CM-6**)
- Lack of Privileged Access Management (**AC-6 / AC-17**)

Additional findings may be added following the [Security Assessment Report](06-security-assessment-report.md).

## Assets Reviewed

Affected assets: Web Server (A-01), Application Server (A-02), Patient Database (A-03), Admin Workstations (A-06). These assets were identified in the Risk Assessment as having control gaps that elevate residual risk.

---

## POA&M Findings Detail

### POA-001 — SI-2 — Flaw Remediation

| Field | Details |
|---|---|
| **Control Gap Description** | AWS Inspector scans identify 23 open vulnerabilities across the application server and web server. 18 are rated Moderate (CVSS 4.0–6.9) and 5 are rated Low (CVSS 0.0–3.9). All Critical and High findings are patched per SLA. However, Moderate and Low findings have not been addressed within defined timeframes — the oldest unpatched Moderate vulnerability is **47 days old** against a **30-day SLA**. |
| **Business Impact** | Unpatched moderate vulnerabilities in a healthcare system can be chained by attackers to achieve privilege escalation or lateral movement. While individually moderate, a chain of such vulnerabilities could compromise the entire system. |
| **Risk Level** | 🔴 High |
| **Responsible Party** | Lead DevOps Engineer — Michael Adeyemi |
| **Remediation Cost Estimate** | Estimated 40 engineer hours ($4,800 at burdened rate) |
| **Current Status** | 🟡 In Progress |
| **Analyst Notes** | The delay was caused by a freeze on production deployments during the hospital's Q1 fiscal year-end period. The freeze was lifted **May 15, 2026**. Patching is now actively in progress. |

#### Remediation Milestones

| Target Date | Action Required |
|---|---|
| June 30, 2026 | Patch all 18 Moderate vulnerabilities in staging environment. Validate no service disruption. |
| July 15, 2026 | Deploy all patched Moderate vulnerabilities to production. Document completion. |
| July 31, 2026 | Patch all 5 Low vulnerabilities in monthly patching window. |
| August 1, 2026 | Implement automated patch compliance reporting in AWS Security Hub. Report to ISSO weekly. |

---

### POA-002 — CM-6 — Configuration Settings (Auto-Remediation Gap)

| Field | Details |
|---|---|
| **Control Gap Description** | AWS Config is fully deployed and detecting configuration drift across all AWS resources. However, **auto-remediation is not enabled**. When a misconfiguration is detected (e.g., an S3 bucket becoming public, a security group opening port 22 to the world), the system generates an alert but does NOT automatically revert the change. Manual remediation currently takes **2–8 hours** after an alert fires, creating a window of exposure. |
| **Business Impact** | A 2–8 hour window where a misconfigured resource is publicly exposed is sufficient for automated scanners operated by threat actors to discover and exploit the exposure. For a healthcare system, this could result in a HIPAA-reportable breach within the exposure window. |
| **Risk Level** | 🔴 High |
| **Responsible Party** | Cloud Security Engineer — Amaka Okafor |
| **Remediation Cost Estimate** | Estimated 60 engineer hours + $200/month AWS Lambda costs |
| **Current Status** | 🟡 In Progress |
| **Analyst Notes** | Auto-remediation was intentionally disabled at launch to prevent accidental disruption of production configurations. The team now has sufficient operational maturity to enable it safely with change management controls. |

#### Remediation Milestones

| Target Date | Action Required |
|---|---|
| June 20, 2026 | Identify and document the top 10 critical misconfiguration scenarios for auto-remediation (e.g., public S3, open ports, disabled encryption). |
| July 10, 2026 | Develop and test auto-remediation Lambda functions for top 5 scenarios in the dev environment. |
| July 25, 2026 | Deploy auto-remediation for top 5 scenarios to production. Monitor for false positives. |
| August 15, 2026 | Expand auto-remediation to cover all 10 identified scenarios. Close POA&M item pending 30-day stability monitoring. |

---

### POA-003 — AC-6 / AC-17 — Privileged Access Management Gap

| Field | Details |
|---|---|
| **Control Gap Description** | Administrative access to PatientPortal's production environment (including SSH access to servers and direct database access) is managed via shared SSH keys and individual IAM credentials with console access. There is no Privileged Access Management (PAM) tool recording admin sessions, no just-in-time (JIT) access provisioning for elevated privileges, and no session recording for administrative activities. This means that if an admin account is compromised, the attacker has persistent, unmonitored elevated access. |
| **Business Impact** | Without session recording and JIT access, there is no forensic evidence of admin actions in a breach investigation. Persistent privileged access is also a primary target for threat actors seeking to maximize damage. HIPAA requires demonstrating that PHI access is logged and auditable — a gap in admin session logging creates compliance exposure. |
| **Risk Level** | 🟡 Moderate |
| **Responsible Party** | ISSO — Dr. Chioma Ezeh |
| **Remediation Cost Estimate** | AWS SSM: $0 (included in AWS costs). CyberArk evaluation: 80 engineer hours ($9,600) |
| **Current Status** | ⚪ Open |
| **Analyst Notes** | This finding was originally identified as Low priority but was elevated to Moderate following T-03 (insider threat) analysis in the Risk Assessment. ISSO has been briefed and has committed resources beginning July 1. |

#### Remediation Milestones

| Target Date | Action Required |
|---|---|
| July 1, 2026 | Complete market evaluation of PAM solutions (CyberArk, HashiCorp Vault, AWS SSM Session Manager). Recommendation to CISO. |
| August 1, 2026 | Implement AWS SSM Session Manager as interim PAM solution (no licensing cost). Enable session logging to S3. |
| September 1, 2026 | Rotate all existing SSH keys and migrate all admin access to SSM. Disable direct SSH access to production. |
| October 1, 2026 | Implement JIT access provisioning for database admin access. Document and close POA&M item. |

---

## POA&M Summary Dashboard

| Finding ID | Control | Risk | Responsible Party | Target Close Date | Status |
|---|---|---|---|---|---|
| POA-001 | SI-2 Patch Management | 🔴 High | Michael Adeyemi | August 1, 2026 | 🟡 In Progress |
| POA-002 | CM-6 Auto-Remediation | 🔴 High | Amaka Okafor | August 15, 2026 | 🟡 In Progress |
| POA-003 | AC-6/AC-17 PAM Gap | 🟡 Moderate | Dr. Chioma Ezeh | October 1, 2026 | ⚪ Open |

---

## Gap Table

| Gap | Current State | Target State | Gap Description |
|---|---|---|---|
| Patch Management | Moderate/Low patches unaddressed (47 days overdue) | All patches addressed within SLA | 18 moderate and 5 low CVEs open beyond their remediation window |
| Config Auto-Remediation | Alert-only, manual remediation 2–8 hrs | Automated remediation within 5 minutes | No auto-remediation for critical AWS misconfigurations |
| Privileged Access | Persistent SSH keys, no session recording | JIT access, full session logging via SSM | No PAM tool; admin sessions not logged for forensics |

---

## Approvals

| Role | Signature | Date |
|---|---|---|
| System Owner | ____________________ | ____________________ |
| Information System Security Officer (ISSO) | ____________________ | ____________________ |
| Authorizing Official (AO) | ____________________ | ____________________ |

---

**Previous:** [← Document 4 — Risk Assessment (SP 800-30)](04-risk-assessment.md)
**Next:** [Document 6 — Security Assessment Report (SAR) →](06-security-assessment-report.md)
