# Document 2 — FIPS 199 Security Categorization Memo

> **RMF Step 2**
>
> **Why this matters:** Before selecting security controls, you must determine how critical the system is. FIPS 199 is the federal standard for doing exactly that — rating a system's sensitivity across three dimensions.

---

## Executive Summary

This memorandum documents the formal security categorization of **PatientPortal v1.0** in accordance with **FIPS Publication 199** (*Standards for Security Categorization of Federal Information and Information Systems*) and **NIST SP 800-60**.

The system has been categorized as **HIGH overall**, driven by the **High confidentiality** and **High integrity** requirements of the Protected Health Information (PHI) it stores and processes.

---

## What Is FIPS 199?

FIPS 199 is a U.S. government standard that asks one key question:

> *"If this system is hacked, how bad would it be?"*

It evaluates three properties — **Confidentiality** (keeping data private), **Integrity** (keeping data accurate), and **Availability** (keeping the system running). Each is rated **Low**, **Moderate**, or **High**. The **overall system rating is whatever the highest individual rating is.**

---

## Scope

This categorization applies to all information types processed, stored, or transmitted by PatientPortal v1.0, including:

- Protected Health Information (PHI)
- Personally Identifiable Information (PII)
- Clinical records
- User authentication data
- System audit logs

---

## Security Impact Analysis

### 🔒 Confidentiality — What If Patient Data Is Exposed?

**Impact Rating: HIGH**

Confidentiality means keeping data private and only accessible to authorized people. A breach of confidentiality in PatientPortal v1.0 would mean unauthorized parties gaining access to patient diagnoses, medications, mental health history, and Social Security Numbers.

- Patients could face discrimination (e.g., insurance companies denying coverage based on exposed diagnoses).
- Victims of identity theft using exposed SSNs could suffer severe financial harm.
- CareFirst Health Systems would face **HIPAA penalties of up to $1.9 million per violation category per year**.
- **Rating is HIGH** because the potential harm to individuals and the organization is severe and potentially irreversible.

### 🛡️ Integrity — What If Patient Data Is Changed Without Authorization?

**Impact Rating: HIGH**

Integrity means ensuring data is accurate and has not been tampered with. In a healthcare system, a breach of integrity is potentially **life-threatening**.

- An attacker who changes a patient's medication dosage in the system could cause a fatal overdose or medical emergency.
- Altered allergy records could result in a patient receiving a life-threatening medication.
- Tampered lab results could lead to incorrect diagnoses and wrong treatment.
- **Rating is HIGH** because unauthorized modification of health data can directly endanger human life.

### ⏱️ Availability — What If the System Goes Offline?

**Impact Rating: MODERATE**

Availability means the system is accessible when users need it. While downtime is disruptive, PatientPortal is a supplementary portal — critical care decisions can be made through the primary EHR system (Epic) even if the portal is unavailable.

- Patients cannot access their records or book appointments during an outage.
- Staff may experience delays in prescription refill processing.
- A prolonged outage (more than 72 hours) could significantly disrupt care coordination.
- **Rating is MODERATE** because the system supports — but is not solely responsible for — care delivery.

---

## FIPS 199 Categorization Summary Table

| Security Objective | Potential Impact | Rating | Rationale |
|---|---|---|---|
| **Confidentiality** | Unauthorized disclosure of PHI/PII causing patient harm, legal penalties, reputational damage | **High** | Patient SSNs, diagnoses, mental health records — highly sensitive by law (HIPAA) |
| **Integrity** | Unauthorized modification of health records potentially causing patient injury or death | **High** | Incorrect medication/dosage data could directly threaten patient safety |
| **Availability** | Temporary service disruption causing delays in non-emergency care coordination | **Moderate** | Backup processes exist; portal is supplementary to the primary EHR system |

---

## Final System Categorization

```
SC(PatientPortal v1.0) = { Confidentiality: HIGH, Integrity: HIGH, Availability: MODERATE }

Overall System Impact Level = HIGH
```

> The **HIGH** categorization means PatientPortal v1.0 must implement the **NIST SP 800-53 HIGH control baseline** — the most rigorous set of security controls available. This drives all subsequent security planning decisions.

---

## Regulatory Implications

| Regulation / Standard | Requirement Triggered | Impact on PatientPortal |
|---|---|---|
| **HIPAA Security Rule** | PHI protection, access controls, audit controls | Mandatory compliance — all ePHI must be encrypted, access-controlled, and audited |
| **FISMA** (if federally funded) | Annual security assessments, ATO required | Requires full RMF process and Authority to Operate before go-live |
| **NIST SP 800-53 Rev 5 High** | ~500+ security controls applicable | HIGH baseline controls apply across all 20 control families |
| **NIST SP 800-66** | HIPAA-NIST implementation guidance | Bridges HIPAA requirements to NIST control implementations |

---

**Previous:** [← Document 1 — System Description](01-system-description.md)
**Next:** [Document 3 — System Security Plan (SSP) →](03-system-security-plan.md)
