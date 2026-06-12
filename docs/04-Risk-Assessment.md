# Document 4 — Risk Assessment (NIST SP 800-30)

> **RMF Step 4**
>
> **Why this matters:** Knowing your controls exist is not enough — you need to identify what threats could still succeed. The Risk Assessment answers the question: *"What could go wrong, how likely is it, and how bad would it be?"* This drives prioritization of security investments.

---

## Executive Summary

This Risk Assessment was conducted for PatientPortal v1.0 following the **NIST SP 800-30 Revision 1** methodology. Five threat scenarios were identified and analyzed based on the system's architecture, data sensitivity, and known threat intelligence for healthcare systems.

Each threat is rated by **likelihood** and **impact** to produce an overall risk level. The results identify **two HIGH risks** requiring immediate attention, **two MODERATE risks** requiring near-term remediation... and overall, all five scenarios reach a HIGH risk rating before mitigation, with **MODERATE residual risk** remaining after existing controls are applied.

---

## What Is a Risk Assessment?

A risk assessment is a structured way of asking:

> *"What could go wrong with this system, and how bad would it be?"*

For each threat, we ask two questions — **How likely is this to happen?** and **If it does happen, how much damage will it cause?** Multiplying these two factors gives us the **risk level**, which tells us where to focus our security efforts first.

---

## Scope

This assessment covers all assets within the PatientPortal v1.0 authorization boundary. Threat analysis considered:

- **External threat actors** (hackers, nation-states, ransomware groups)
- **Internal threats** (malicious or negligent insiders)
- **Environmental threats** (cloud service disruptions)

The assessment period covers the **12-month operational cycle** following initial deployment.

## Assets Reviewed

All 10 system assets (A-01 through A-10) were considered. High-priority assets for this analysis include:

- Patient Database (A-03)
- Authentication Service (A-04)
- Application Server (A-02)
- Backup Storage (A-08)
- EHR Integration API (A-10)

---

## Risk Rating Methodology

| Likelihood ↓ / Impact → | Low | Moderate | High |
|---|---|---|---|
| **High** | Moderate | High | High |
| **Moderate** | Low | Moderate | High |
| **Low** | Low | Low | Moderate |

---

## Threat Scenarios & Risk Analysis

### T-01: Phishing Attack Leading to Credential Theft

| Field | Details |
|---|---|
| **Threat Description** | A malicious actor sends a convincing fake email to a clinical staff member, tricking them into entering their PatientPortal credentials on a fake login page. The attacker then uses those credentials to log in and access patient records. |
| **Affected Assets** | Authentication Service (A-04), Application Server (A-02) |
| **Likelihood** | High |
| **Impact** | High |
| **Overall Risk Level** | 🔴 High |
| **Existing Controls** | MFA (IA-2), Security Awareness Training (AT-2), Email Filtering |
| **Residual Risk After Controls** | 🟡 Moderate |
| **Analyst Rationale** | Healthcare workers receive phishing emails constantly. Even with MFA, sophisticated phishing (e.g., real-time MFA relay attacks) can succeed. The impact is severe because PHI of thousands of patients is at risk. |

---

### T-02: Ransomware Attack Encrypting Patient Database

| Field | Details |
|---|---|
| **Threat Description** | Ransomware malware infiltrates the system (often via phishing or unpatched software) and encrypts the patient database, making all health records inaccessible until a ransom is paid. |
| **Affected Assets** | Patient Database (A-03), Backup Storage (A-08) |
| **Likelihood** | High |
| **Impact** | High |
| **Overall Risk Level** | 🔴 High |
| **Existing Controls** | Endpoint Protection, Network Segmentation, Daily Backups (CP-9) |
| **Residual Risk After Controls** | 🟡 Moderate |
| **Analyst Rationale** | Ransomware attacks on healthcare are at an all-time high — they accounted for 54% of healthcare cyberattacks in 2023. The backup controls reduce impact, but a successful attack still disrupts care and requires PHI breach notification. |

---

### T-03: Unauthorized Insider Access to Patient Records

| Field | Details |
|---|---|
| **Threat Description** | A hospital employee (e.g., a curious staff member or a disgruntled employee) accesses patient records they have no clinical need to view, potentially leaking or misusing private health information. |
| **Affected Assets** | Patient Database (A-03), Application Server (A-02) |
| **Likelihood** | Moderate |
| **Impact** | High |
| **Overall Risk Level** | 🔴 High |
| **Existing Controls** | Role-Based Access Control (AC-6), Audit Logging (AU-2), Audit Review (AU-6) |
| **Residual Risk After Controls** | 🟡 Moderate |
| **Analyst Rationale** | Insider threats are a leading cause of healthcare data breaches. While RBAC limits access, users may still access records within their permission level inappropriately. Automated monitoring reduces but does not eliminate this risk. |

---

### T-04: Unpatched Vulnerability Exploited in Web Application

| Field | Details |
|---|---|
| **Threat Description** | An attacker discovers a publicly known software vulnerability in the PatientPortal application or its underlying infrastructure that has not yet been patched, and exploits it to gain unauthorized access. |
| **Affected Assets** | Web Server (A-01), Application Server (A-02) |
| **Likelihood** | Moderate |
| **Impact** | High |
| **Overall Risk Level** | 🔴 High |
| **Existing Controls** | AWS Inspector scanning, Patch Management (SI-2 — partially implemented) |
| **Residual Risk After Controls** | 🟡 Moderate |
| **Analyst Rationale** | SI-2 is only partially implemented — patching of moderate/low severity findings is behind schedule. This creates a window of exposure. The HIGH impact reflects that a successful exploit could compromise the entire application. |

---

### T-05: Cloud Misconfiguration Exposing Sensitive Data

| Field | Details |
|---|---|
| **Threat Description** | An administrator accidentally misconfigures an AWS S3 bucket, making encrypted patient backups or logs publicly accessible on the internet. This type of error has caused some of the largest healthcare data breaches in history. |
| **Affected Assets** | Backup Storage (A-08), Audit Logging (A-05) |
| **Likelihood** | Moderate |
| **Impact** | High |
| **Overall Risk Level** | 🔴 High |
| **Existing Controls** | AWS Config, CM-6 Configuration Management (partially implemented), AWS Security Hub |
| **Residual Risk After Controls** | 🟡 Moderate |
| **Analyst Rationale** | Cloud misconfiguration is the #1 cause of cloud data breaches. CM-6 is partially implemented, meaning configuration drift detection is not fully mature. A single misconfigured bucket setting could expose all backup data. |

---

## Risk Register Summary

| ID | Threat | Likelihood | Impact | Risk Level | Residual Risk | Priority |
|---|---|---|---|---|---|---|
| T-01 | Phishing Attack Leading to Credential Theft | High | High | 🔴 High | 🟡 Moderate | P1 |
| T-02 | Ransomware Attack Encrypting Patient Database | High | High | 🔴 High | 🟡 Moderate | P2 |
| T-03 | Unauthorized Insider Access to Patient Records | Moderate | High | 🔴 High | 🟡 Moderate | P3 |
| T-04 | Unpatched Vulnerability Exploited in Web Application | Moderate | High | 🔴 High | 🟡 Moderate | P4 |
| T-05 | Cloud Misconfiguration Exposing Sensitive Data | Moderate | High | 🔴 High | 🟡 Moderate | P5 |

---

## Gap Table — Security Control Weaknesses Contributing to Risk

| Gap ID | Control Gap | Related Threat(s) | Risk Contributed | Recommended Action |
|---|---|---|---|---|
| G-01 | SI-2 patch management not fully enforced for moderate/low vulnerabilities | T-04 | High | Complete patch management process — apply all open patches within defined SLAs. Track in POA&M. |
| G-02 | CM-6 configuration management lacks automated remediation (detect-only) | T-05 | High | Implement AWS Config auto-remediation rules for critical misconfigurations. |
| G-03 | Security awareness training not yet role-specific for clinical staff | T-01 | Moderate | Develop role-specific phishing simulation and training for clinical staff. |
| G-04 | No dedicated Privileged Access Management (PAM) tool for admin accounts | T-03 | Moderate | Implement CyberArk or AWS SSM Session Manager for privileged session recording. |
| G-05 | Backup restoration testing conducted only quarterly (not monthly) | T-02 | Low–Moderate | Increase backup restoration testing to monthly and document results. |

---

## Improvement Plans & Next Steps

1. Immediately remediate all Critical (CVSS 9.0+) and High (CVSS 7.0+) vulnerabilities flagged by AWS Inspector within defined SLA windows. *(Addresses G-01, T-04)*
2. Implement AWS Config Auto-Remediation Rules for the top 5 misconfiguration scenarios (public S3 bucket, disabled encryption, open security groups). *(Addresses G-02, T-05)*
3. Commission role-specific phishing simulation campaigns for all clinical staff within 60 days. *(Addresses G-03, T-01)*
4. Evaluate and implement a Privileged Access Management (PAM) solution for all administrative accounts. *(Addresses G-04, T-03)*
5. Increase backup restoration testing to monthly and integrate into the security operations calendar. *(Addresses G-05, T-02)*

---

## Approval

| Role | Signature | Date |
|---|---|---|
| Assessor Name | _______________________ | _______________ |
| Information System Owner | _______________________ | _______________ |
| Authorizing Official | _______________________ | _______________ |

---

**Previous:** [← Document 3 — System Security Plan (SSP)](03-system-security-plan.md)
**Next:** [Document 5 — Plan of Action & Milestones (POA&M) →](05-poam.md)
