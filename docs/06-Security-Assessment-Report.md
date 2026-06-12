# Document 6 — Security Assessment Report (SAR)

> **RMF Step 6**
>
> **Why this matters:** The SAR is the final independent verification that the system's security controls actually work. It is the document an Authorizing Official reads to decide whether to grant an **Authority to Operate (ATO)**. One page of the SAR can make or break a system's approval.

---

## Executive Summary

This Security Assessment Report (SAR) documents the results of the security control assessment conducted for PatientPortal v1.0 between **May 15–30, 2026**. The assessment evaluated 12 security controls from the NIST SP 800-53 High baseline as documented in the [System Security Plan](03-system-security-plan.md). **Six controls were tested in depth** for this report.

| Result | Count |
|---|---|
| ✅ Pass | 4 |
| ◑ Partial | 1 |
| ✘ Fail | 1 |

**Two high-priority findings require remediation before Authority to Operate (ATO) can be granted.**

---

## Assessment Scope

The assessment covered all 10 system assets within the PatientPortal v1.0 authorization boundary. Six controls from the SSP were selected for in-depth testing, representing the highest-risk areas identified in the [Risk Assessment](04-risk-assessment.md). Assessment methods included documentation review, technical testing, staff interviews, and configuration inspection.

## Assessment Methodology

| Method | Controls Tested | Description |
|---|---|---|
| Documentation Review | All 6 | Reviewed SSP, policies, procedures, and configuration records against control requirements |
| Technical Testing | IA-2, SC-8, SC-28, SI-2 | Hands-on verification: tested MFA enforcement, encryption protocols, backup encryption, and vulnerability scan results |
| Configuration Inspection | CM-6, AC-2 | Reviewed AWS Config rules, Okta policies, and IAM role configurations |
| Staff Interviews | AC-2, IR-4, AU-6 | Interviewed system administrator, security analyst, and two clinical staff members |
| Log Review | AU-2, AU-6 | Sampled 30 days of CloudWatch logs to verify completeness and alert response times |

---

## Control Assessment Findings

### SAR-F-01 — IA-2 — Identification and Authentication
**Result: ✅ PASS**

| Field | Assessment Details |
|---|---|
| **Finding Summary** | Multi-Factor Authentication is fully enforced for all user accounts. Testing confirmed that MFA cannot be bypassed for any account type including administrative, clinical, and patient-facing accounts. |
| **Evidence Reviewed** | Tested 10 user accounts across all roles. MFA was required without exception. Okta audit logs confirmed 100% MFA enrollment. Attempted bypass using password-only authentication was blocked. |
| **Detailed Analysis** | MFA enforcement is configured at the Okta tenant level, preventing any administrator from inadvertently disabling it for individual users. Hardware YubiKey tokens are in use for all privileged accounts. SMS OTP is used for patient accounts. Session timeout of 15 minutes is enforced and verified. HIPAA §164.312(d) authentication requirement is met. |
| **Gaps Identified** | None identified. |
| **Recommendation** | Continue current implementation. Consider upgrading patient accounts from SMS OTP to TOTP app-based MFA to reduce SIM-swapping risk in future cycle. |

---

### SAR-F-02 — AU-2 / AU-6 — Event Logging and Audit Review
**Result: ✅ PASS**

| Field | Assessment Details |
|---|---|
| **Finding Summary** | Audit logging captures all required event types. Automated monitoring is active and alert response times meet defined SLAs. Log integrity protection is in place. |
| **Evidence Reviewed** | Reviewed 30 days of CloudWatch logs. Sampled 50 login events, 20 record access events, and 5 privilege escalation events — all were captured with required fields. GuardDuty generated 3 alerts during the review period; all were acknowledged within 12 minutes. |
| **Detailed Analysis** | Logs capture all 6 required fields: user ID, event type, date/time, success/fail, origination, and affected resource. Logs are stored in write-once S3 bucket preventing tampering. Log retention of 3 years meets HIPAA requirements. Automated alerting for anomalies is functioning correctly. Average alert response time of 8 minutes is well within the 15-minute SLA. |
| **Gaps Identified** | Minor gap: Two log entries during a server maintenance window had incomplete user ID fields (system process rather than user account). This is a known edge case and does not represent a compliance failure. |
| **Recommendation** | Document the maintenance window logging exception in the SSP. Configure system process events to use a dedicated service account identifier for traceability. |

---

### SAR-F-03 — SC-8 / SC-28 — Encryption in Transit and at Rest
**Result: ✅ PASS**

| Field | Assessment Details |
|---|---|
| **Finding Summary** | All data in transit is encrypted using TLS 1.3. All PHI at rest is encrypted using AES-256. Encryption key management is functional and meets NIST SP 800-57 requirements. |
| **Evidence Reviewed** | SSL Labs scan of the portal returned **A+** grade. Confirmed TLS 1.2 minimum, TLS 1.3 preferred. RC4, DES, and 3DES cipher suites are disabled. RDS encryption verified via AWS console. S3 SSE-KMS confirmed active on all backup buckets. KMS key rotation policy verified (90-day rotation). |
| **Detailed Analysis** | The portal achieves an A+ rating on SSL Labs with HSTS enabled, forward secrecy on all cipher suites, and OCSP stapling. Database encryption covers all data including backups and snapshots. Key rotation is automated and tested. One historical S3 bucket (legacy test bucket from pre-production) was found with SSE-S3 instead of SSE-KMS — this bucket contains no live data and was scheduled for deletion. |
| **Gaps Identified** | Legacy test S3 bucket using SSE-S3 instead of SSE-KMS. Bucket contains no production data but should be deleted or migrated. |
| **Recommendation** | Delete the legacy test S3 bucket (`s3://patientportal-test-2024-legacy`) within 30 days. Minor finding — does not require remediation before ATO. |

---

### SAR-F-04 — AC-2 — Account Management
**Result: ✅ PASS**

| Field | Assessment Details |
|---|---|
| **Finding Summary** | User accounts are properly managed through the Okta provisioning workflow. Account reviews are conducted quarterly. Orphaned accounts are not present. |
| **Evidence Reviewed** | Reviewed Okta admin console. Confirmed 0 accounts with no login activity in 90+ days that remain active (automated disablement policy is working). Reviewed 5 employee termination records against Okta account status — all 5 were disabled on the date of termination or within 1 business day. |
| **Detailed Analysis** | The HR system triggers automated Okta deprovisioning workflows via SCIM provisioning. All 5 reviewed terminations were processed correctly. Quarterly account review was last completed March 31, 2026 — all findings from that review were remediated. Admin account creation requires dual approval — verified via Okta workflow logs for the past 90 days (4 admin accounts reviewed, all had dual approval). |
| **Gaps Identified** | None identified. |
| **Recommendation** | Continue current implementation. Consider implementing a privileged account review quarterly in addition to the full account review to increase scrutiny on high-risk accounts. |

---

### SAR-F-05 — CM-6 — Configuration Settings
**Result: ◑ PARTIAL PASS**

| Field | Assessment Details |
|---|---|
| **Finding Summary** | Configuration management monitoring is active and detecting drift. However, auto-remediation is not enabled, creating a 2–8 hour exposure window when misconfigurations are detected. Additionally, 3 out of 47 AWS Config rules are in a non-compliant state. |
| **Evidence Reviewed** | AWS Config dashboard reviewed. 44 of 47 rules are compliant. 3 rules are non-compliant: (1) One EC2 instance missing required tag, (2) One security group with port 3389 (RDP) open to 0.0.0.0/0, (3) One IAM user without MFA enabled (service account used for legacy integration). No auto-remediation is configured for any rule. |
| **Detailed Analysis** | The open RDP port (finding 2) is the most serious finding. RDP (Remote Desktop Protocol) exposed to the internet is a primary target for brute-force attacks and ransomware deployment. The service account without MFA (finding 3) presents insider threat and credential-theft risk. The missing EC2 tag (finding 1) is administrative and low risk. The auto-remediation gap (documented in [POA-002](05-poam.md#poa-002--cm-6--configuration-settings-auto-remediation-gap)) means these findings were detected but not automatically resolved. |
| **Gaps Identified** | **G-CM-01:** Open RDP port on security group `sg-0abc1234` exposes production subnet to internet. **HIGH risk.** <br/> **G-CM-02:** Service account `iam-legacy-svc-01` has no MFA and console access enabled. **MODERATE risk.** <br/> **G-CM-03:** Auto-remediation disabled — detected misconfigurations require manual intervention. **HIGH risk (process gap).** |
| **Recommendation** | **IMMEDIATE:** Close the RDP port in security group `sg-0abc1234` within 24 hours. HIGH priority. **Within 7 days:** Disable console access for `iam-legacy-svc-01` and convert to access-key-only API account. POA-002 auto-remediation remains on track for August 15, 2026. |

---

### SAR-F-06 — SI-2 — Flaw Remediation (Patch Management)
**Result: ✘ FAIL**

| Field | Assessment Details |
|---|---|
| **Finding Summary** | Patch management processes for Critical and High severity vulnerabilities are functioning correctly. However, Moderate and Low severity patches are significantly overdue, with the oldest open finding at **47 days** past its **30-day SLA**. This represents a formal control failure. |
| **Evidence Reviewed** | AWS Inspector scan reviewed (scan date: May 28, 2026). 23 open findings identified: 0 Critical, 0 High, 18 Moderate, 5 Low. Oldest Moderate finding: 47 days open (SLA: 30 days). CVSS scores of open findings range from 4.1 to 6.8. No findings are in the web application layer — all findings are in underlying OS packages. |
| **Detailed Analysis** | The 18 overdue Moderate findings span 3 servers: web server (7 findings), application server (8 findings), and a legacy integration server (3 findings). The most significant finding is `CVE-2024-XXXX` (CVSS 6.8) — an OpenSSL vulnerability on the application server that could allow a remote attacker to cause a denial of service. While not directly exploitable for data theft, it could be combined with other vulnerabilities. Remediation is actively in progress per [POA-001](05-poam.md#poa-001--si-2--flaw-remediation) but has not yet met the SLA. |
| **Gaps Identified** | **G-SI-01:** 18 Moderate vulnerabilities overdue past 30-day SLA. Oldest is 47 days. <br/> **G-SI-02:** No automated escalation when patches exceed SLA — manual process relies on weekly reviews which are insufficient. <br/> **G-SI-03:** Legacy integration server has no defined owner for patch approval — causing delay. |
| **Recommendation** | **REQUIRED BEFORE ATO:** Develop and execute a 30-day patch remediation sprint to close all overdue findings. Assign explicit patch ownership to the legacy integration server. Implement automated SLA breach alerts in AWS Security Hub. Control must achieve Implemented status before ATO is granted. Interim ATO with conditions is acceptable if a firm remediation date is provided. |

---

## Control Assessment Summary Table

| Finding ID | Control | Assessment Result | Key Finding | ATO Impact |
|---|---|---|---|---|
| SAR-F-01 | IA-2 Authentication | ✅ Pass | MFA fully enforced across all accounts | No issue |
| SAR-F-02 | AU-2/AU-6 Logging | ✅ Pass | Logging complete; alert response avg 8 min | No issue |
| SAR-F-03 | SC-8/SC-28 Encryption | ✅ Pass | A+ TLS; AES-256 at rest; minor legacy bucket | Minor — cleanup only |
| SAR-F-04 | AC-2 Account Mgmt | ✅ Pass | Zero orphaned accounts; timely deprovisioning | No issue |
| SAR-F-05 | CM-6 Configuration | ◑ Partial | Open RDP port & no MFA on service account | Remediate within 7 days |
| SAR-F-06 | SI-2 Patch Mgmt | ✘ Fail | 18 Moderate CVEs past 30-day SLA | Required before ATO — or Conditional ATO |

---

## Gap Table — SAR Findings Requiring Action

| Gap ID | Description | Severity | Control | Remediation Required By |
|---|---|---|---|---|
| G-CM-01 | Open RDP port to internet on production security group | 🔴 High | CM-6 | Within 24 hours of SAR issuance |
| G-CM-02 | Service account with console access and no MFA enabled | 🟡 Moderate | CM-6 | Within 7 days |
| G-SI-01 | 18 Moderate CVEs past 30-day SLA; oldest at 47 days | 🔴 High | SI-2 | Before ATO grant (or Conditional ATO with 30-day sprint) |
| G-SI-02 | No automated SLA breach alerts for patch management | 🟡 Moderate | SI-2 | Within 30 days |
| G-AU-01 | Incomplete user ID in 2 maintenance window log entries | ⚪ Low | AU-2 | Within 60 days |
| G-SC-01 | Legacy test S3 bucket using SSE-S3 instead of SSE-KMS | ⚪ Low | SC-28 | Within 30 days (deletion) |

---

## ATO Recommendation

> ### ASSESSOR RECOMMENDATION: CONDITIONAL AUTHORITY TO OPERATE (ATO)

PatientPortal v1.0 demonstrates a generally strong security posture with **4 of 6 tested controls passing assessment**. Two findings — an open RDP internet exposure (CM-6) and overdue patch management (SI-2) — require formal remediation.

The assessor recommends a **Conditional ATO** with the following conditions:

1. Close the open RDP port within **24 hours** of ATO issuance.
2. Disable legacy service account console access within **7 days**.
3. Complete patch sprint ([POA-001](05-poam.md#poa-001--si-2--flaw-remediation)) within **30 days**, with weekly status reports to the ISSO.

**Full ATO (without conditions)** may be granted once all three conditions are verified closed by the ISSO.

---

## Improvement Plans & Next Steps

1. **IMMEDIATE (within 24 hours):** Close open RDP port on security group `sg-0abc1234`. Verify via AWS Config and provide screenshot evidence to ISSO.
2. **SHORT-TERM (within 7 days):** Disable console access for `iam-legacy-svc-01`. Convert to API-key-only access. Document change in the SSP.
3. **30-DAY SPRINT:** Execute patch remediation plan per [POA-001](05-poam.md#poa-001--si-2--flaw-remediation). Apply all 18 Moderate CVEs in staging then production. Report weekly to ISSO.
4. **PROCESS IMPROVEMENT:** Implement automated SLA breach alerts in AWS Security Hub for patch management. Assign dedicated patch owner for the legacy integration server.
5. **ONGOING:** Add all new SAR findings to the POA&M within 5 business days of SAR issuance. Schedule follow-up assessment in 90 days to verify closure of all conditional ATO items.

---

## Approvals

| Role | Signature | Date |
|---|---|---|
| Security Control Assessor (SCA) | ____________________ | ____________________ |
| Information System Owner | ____________________ | ____________________ |
| Authorizing Official (AO) | ____________________ | ____________________ |

---

**Previous:** [← Document 5 — Plan of Action & Milestones (POA&M)](05-poam.md)
**Back to:** [README — Package Overview](../README.md)
