# Data Flow Diagram — Patient Login & Record Retrieval

This diagram illustrates how patient data moves through PatientPortal v1.0 during a standard login and medical record retrieval session, as described in [Document 1 — System Description](../docs/01-system-description.md).

---

## Sequence Diagram

```mermaid
sequenceDiagram
    autonumber
    participant Patient as Patient (Web Browser)
    participant Web as PatientPortal Web Server (Apache 2.4)
    participant App as Application Server (Node.js)
    participant Okta as Okta SSO
    participant DB as Patient Database (PostgreSQL on AWS RDS)

    Patient->>Web: Login Request / Patient Record Request (HTTPS / TLS 1.3)
    Web->>App: Forward Request
    App->>Okta: Authentication Request
    Okta-->>App: Authentication Response
    App->>DB: Query Patient Records
    Note over DB: AES-256 Encryption at Rest
    DB-->>App: Encrypted Patient Data
    App-->>Web: Return Results
    Web-->>Patient: HTTPS / TLS 1.3 Response
```

---

## Component Flow Diagram

```mermaid
flowchart TB
    A[Patient<br/>Web Browser<br/><i>HTTPS / TLS 1.3</i>] -->|"Login Request /<br/>Patient Record Request"| B[PatientPortal<br/>Web Server]
    B -->|Forward Request| C[Application Server]
    C -->|Authentication Request| D[Okta Single Sign-On]
    D -->|Authentication Response| C
    C -->|Query Patient Records| E[("Patient Database<br/>PostgreSQL on AWS RDS<br/>AES-256 Encryption at Rest")]
    E -->|Encrypted Patient Data| C
    C -->|Return Results| B
    B -->|HTTPS / TLS 1.3| A

    style E fill:#f9d6d6,stroke:#c0392b
    style D fill:#d6e8f9,stroke:#2980b9
```

---

**Related diagrams:**
- [Audit Logging Flow](audit-logging-flow.md)
- [Prescription Refill Flow](prescription-refill-flow.md)

**Back to:** [README](../README.md) | [Document 1 — System Description](../docs/01-system-description.md)
