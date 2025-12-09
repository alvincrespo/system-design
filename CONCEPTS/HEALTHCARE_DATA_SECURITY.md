# Healthcare Data Security

> **Key Takeaway:** *"In healthcare systems, design with defense in depth: network segmentation, field-level encryption for PHI, RBAC with principle of least privilege, comprehensive audit logging, and automated breach detection. Any healthcare system needs to assume breach and have detection/response procedures, not just prevention."*

---

## Table of Contents

1. [HIPAA Breach Notification Requirements](#1-hipaa-breach-notification-requirements)
2. [Minimum Necessary Standard](#2-minimum-necessary-standard)
3. [De-identification vs. Anonymization](#3-de-identification-vs-anonymization)
4. [Audit Logging Requirements](#4-audit-logging-requirements)
5. [Role-Based Access Control (RBAC) in Healthcare](#5-role-based-access-control-rbac-in-healthcare)
6. [Defense in Depth: Healthcare Security Architecture](#6-defense-in-depth-healthcare-security-architecture)
7. [Quick Reference: Key Numbers & Timelines](#quick-reference-key-numbers--timelines)
8. [References](#references)

---

## 1. HIPAA Breach Notification Requirements

### Definition of a Breach

Under HIPAA, a breach is defined as an impermissible use or disclosure of Protected Health Information (PHI) that compromises the security or privacy of the PHI. An impermissible use or disclosure is **presumed to be a breach** unless the covered entity can demonstrate a **low probability** that the PHI has been compromised.

### Four-Factor Risk Assessment

To determine if there's a low probability of compromise, perform this assessment:

1. **Nature and Extent:** What types of identifiers were involved? What is the likelihood of re-identification?
2. **Unauthorized Recipient:** Who was the unauthorized person who used or received the PHI?
3. **Actual Acquisition:** Was the PHI actually acquired or viewed?
4. **Mitigation:** To what extent has the risk been mitigated?

### Notification Requirements

| Breach Size | Who to Notify | Timeline |
|-------------|---------------|----------|
| **500+ individuals** | Affected individuals, HHS Secretary, prominent media outlets | Within 60 days of discovery |
| **Under 500 individuals** | Affected individuals, HHS Secretary | Individuals: within 60 days; HHS: within 60 days of calendar year end |
| **Business Associate breach** | Must notify Covered Entity | Within 60 days of discovery |

> *System Design Tip: Build automated breach detection and notification systems. The 60-day clock starts at discovery, so rapid detection is critical.*

### Notification Content Requirements

Individual breach notifications must include:

- Description of what happened, including dates of breach and discovery
- Types of unsecured PHI involved (name, SSN, DOB, diagnosis codes, etc.)
- Steps individuals should take to protect themselves
- What the entity is doing to investigate and prevent future breaches
- Contact procedures (toll-free number, email, website, or postal address)

### "Unsecured" PHI Exception

Breach notification is only required for **unsecured** PHI. PHI is considered secure if it has been rendered unusable, unreadable, or indecipherable through:

- **Encryption:** Using NIST-approved algorithms (AES-128, AES-256)
- **Destruction:** Paper shredding or electronic media destruction per NIST guidelines

---

## 2. Minimum Necessary Standard

### Core Principle

The minimum necessary standard requires covered entities to make **reasonable efforts** to limit PHI access, use, and disclosure to the **minimum necessary** to accomplish the intended purpose. This is derived from traditional confidentiality practices in healthcare.

### When It Applies

The standard applies to:

- Internal uses of PHI within the organization
- Routine and non-routine disclosures to third parties
- Requests for PHI from other covered entities
- Healthcare operations, payment processing, and administrative functions

### Exceptions (Does NOT Apply)

1. **Treatment purposes:** Healthcare providers sharing PHI for patient treatment
2. **Patient access:** Disclosures to the individual who is the subject of the PHI
3. **Patient authorization:** Uses/disclosures made pursuant to patient's written authorization
4. **HIPAA compliance:** Required for Administrative Simplification Rules compliance
5. **HHS enforcement:** Disclosures to HHS for enforcement purposes
6. **Required by law:** Other legal mandates requiring disclosure

### Implementation Requirements

**For Internal Uses:**
- Identify persons/classes who need PHI access for their job duties
- Define categories of PHI each role needs
- Establish conditions appropriate to such access
- If entire medical record is necessary, document explicit justification

**For Routine Disclosures:**
- Develop standard protocols limiting PHI to minimum necessary
- Individual review of each disclosure is NOT required

**For Non-Routine Disclosures:**
- Develop criteria for case-by-case review
- Each request must be reviewed individually against criteria

> *System Design Implication: Implement role-based access control (RBAC) that maps directly to job functions. Use attribute-based policies to enforce minimum necessary at the field level—e.g., billing staff sees diagnosis codes but not clinical notes.*

---

## 3. De-identification vs. Anonymization

### Why De-identify PHI?

De-identified health information is **not subject to HIPAA Privacy Rule restrictions**. This enables secondary uses like medical research, policy assessments, comparative effectiveness studies, and data analytics without individual authorizations.

### Two HIPAA-Compliant Methods

#### Method 1: Safe Harbor ([§164.514(b)(2)](https://www.ecfr.gov/current/title-45/part-164/section-164.514#p-164.514(b)(2)))

Remove **all 18 specified identifiers** from the data set:

| Direct Identifiers | Other Identifiers |
|-------------------|-------------------|
| 1. Names | 10. Account numbers |
| 2. Geographic subdivisions < state* | 11. Certificate/license numbers |
| 3. Dates (except year)** | 12. Vehicle identifiers/serial numbers |
| 4. Phone numbers | 13. Device identifiers/serial numbers |
| 5. Fax numbers | 14. Web URLs |
| 6. Email addresses | 15. IP addresses |
| 7. Social Security numbers | 16. Biometric identifiers |
| 8. Medical record numbers | 17. Full-face photos |
| 9. Health plan beneficiary numbers | 18. Any other unique identifier*** |

**Notes:**
- *ZIP codes: First 3 digits allowed if geographic unit has >20,000 people. 17 specific 3-digit ZIPs must be changed to 000.
- **Dates: Year can be retained except for ages >89 (aggregate to 90+). Birth dates, admission/discharge dates, death dates affected.
- ***Note: Original 1999 list is outdated. Also remove: social media handles, Medicare Beneficiary Numbers, LGBTQ+ status if identifying.

**"Actual Knowledge" Requirement:** The covered entity must have no actual knowledge that the remaining information could be used to identify an individual.

#### Method 2: Expert Determination ([§164.514(b)(1)](https://www.ecfr.gov/current/title-45/part-164/section-164.514#p-164.514(b)(1)))

A qualified statistical expert determines that the risk of re-identification is "very small."

**Expert Requirements:**
- Knowledge of statistical/scientific principles for de-identification
- Must document methods and results justifying the determination
- Must consider risk from combining with "reasonably available" external data

**Common Techniques:**
- **Suppression:** Remove values or entire records that are too unique
- **Generalization:** Replace specific values with ranges (age 45 → 40-50)
- **Perturbation:** Add random noise while preserving statistical properties
- **k-Anonymity:** Ensure each record is indistinguishable from at least k-1 others

### De-identification vs. Anonymization Comparison

| Aspect | De-identification | Anonymization |
|--------|------------------|---------------|
| **Definition** | Remove identifiers per HIPAA standards | Irreversibly remove all identifying info |
| **Re-identification** | Possible with code/key (per [§164.514(c)](https://www.ecfr.gov/current/title-45/part-164/section-164.514#p-164.514(c))) | Should be impossible |
| **Data Utility** | Higher—can link back for research | Lower—no way to link to original |
| **HIPAA Status** | No longer PHI if properly done | No longer PHI |

---

## 4. Audit Logging Requirements

### HIPAA Security Rule Foundation

The HIPAA Security Rule ([45 CFR §164.312(b)](https://www.ecfr.gov/current/title-45/part-164/section-164.312#p-164.312(b))) requires covered entities to implement **audit controls** that record and examine activity in information systems containing or using ePHI.

### Three Types of Audit Trails

#### 1. Application Audit Trails

Monitor and log user activities within applications:
- Opening and closing ePHI data files
- Creating, reading, editing, and deleting ePHI records
- Printing or exporting PHI
- Query execution and results

#### 2. System-Level Audit Trails

Capture infrastructure-level events:
- Successful and unsuccessful log-on attempts
- Log-on ID/username and timestamps
- Devices used to log on
- Applications accessed (successful and unsuccessful)
- System configuration changes

#### 3. User Audit Trails

Record events initiated by specific users:
- All commands directly initiated by the user
- Identification and authentication events
- Access to ePHI files and resources
- Permission changes and role assignments

### Required Log Elements

| Element | Description |
|---------|-------------|
| **User Identity** | Who performed the action (user ID, role) |
| **Timestamp** | When the action occurred (synchronized time) |
| **Action Performed** | What was done (read, write, delete, export) |
| **Resource Accessed** | Which data/system was affected (patient ID, record type) |
| **Source/Destination** | IP address, device identifier, location |
| **Outcome** | Success or failure of the action |

### Retention & Review Requirements

- **Retention:** Minimum 6 years (per HIPAA documentation requirements)
- **Storage:** Uncompressed for 6-12 months, then archive in compressed format
- **Integrity:** Use tamper-proof methods (WORM storage, cryptographic hashing)
- **Review:** Regular review procedures (daily, weekly, or monthly based on risk assessment)
- **Alerts:** Automated alerts for suspicious activities (multiple failed logins, bulk data exports)

> *System Design Tip: Implement centralized logging with SIEM integration. Develop dashboards to monitor the usage and performance of critical projects. Consider real-time anomaly detection for unusual access patterns.*

---

## 5. Role-Based Access Control (RBAC) in Healthcare

### Core Principles

RBAC in healthcare implements the **Principle of Least Privilege**: users should only have access to the information necessary to perform their job functions—no more, no less.

### RBAC Components

1. **Roles:** Job functions (Physician, Nurse, Billing Clerk, Administrator)
2. **Permissions:** Actions allowed (Read, Write, Delete, Export)
3. **Resources:** Data objects (Patient Records, Lab Results, Billing Data)
4. **Users:** Individual employees assigned to roles

### Healthcare Role Matrix Example

| Role | Demographics | Clinical Notes | Lab Results | Billing |
|------|--------------|----------------|-------------|---------|
| Physician | R/W | R/W | R/W | R |
| Nurse | R/W | R/W | R | — |
| Billing Clerk | R | — | — | R/W |
| Lab Tech | R | — | R/W | — |
| Receptionist | R/W | — | — | — |

*Legend: R/W = Read/Write, R = Read Only, — = No Access*

### Beyond Basic RBAC: Contextual Access

Healthcare often requires **Attribute-Based Access Control (ABAC)** layered on top of RBAC to handle context-sensitive scenarios:

- **Break-the-glass:** Emergency override with mandatory logging and justification
- **Care team filtering:** Access only to patients on user's care team
- **Time-based:** Access limited to shift hours or appointment windows
- **Location-based:** Access restricted by physical location or network
- **Consent-based:** Patient can restrict access to certain providers

### Implementation Best Practices

1. **Role Definition:** Work with HR and department heads to map job functions to data needs
2. **Separation of Duties:** No single role should have excessive privileges (e.g., create user + approve access)
3. **Regular Review:** Quarterly access reviews, immediate revocation on termination
4. **Audit Trail:** Log all role assignments, changes, and access attempts
5. **MFA Enforcement:** Multi-factor authentication for all PHI access

---

## 6. Defense in Depth: Healthcare Security Architecture

A comprehensive healthcare security architecture implements multiple overlapping security controls:

### Security Layers

#### 1. Network Segmentation

- Isolate clinical systems from administrative networks
- Separate medical devices (IoMT) into dedicated VLANs
- Implement firewalls between segments with strict ACLs
- Use micro-segmentation for high-value assets

#### 2. Encryption

- **At Rest:** AES-256 for databases, file systems, and backups
- **In Transit:** TLS 1.2+ for all communications (never SSL)
- **Field-Level:** Encrypt sensitive fields (SSN, diagnosis codes) separately
- **Key Management:** HSM for key storage, regular rotation

#### 3. Access Controls

- RBAC with principle of least privilege
- Multi-factor authentication for PHI access
- Just-in-time access for elevated privileges
- Session timeout and automatic logoff

#### 4. Monitoring & Detection

- SIEM for centralized log aggregation and correlation
- IDS/IPS for network threat detection
- Endpoint detection and response (EDR)
- User behavior analytics (UBA) for anomaly detection

#### 5. Incident Response

**7-Step Process:**

1. **Preparation:** IRT team, playbooks, tools, training
2. **Identification:** Detect and triage incidents
3. **Containment:** Short-term (isolate) and long-term (patch)
4. **Eradication:** Root cause analysis and threat removal
5. **Recovery:** System restoration with continuous monitoring
6. **Post-Incident Review:** Lessons learned and plan updates
7. **Continuous Improvement:** Testing, automation, refinement

---

## Quick Reference: Key Numbers & Timelines

| Requirement | Value |
|-------------|-------|
| Breach notification deadline | **60 days** from discovery |
| Media notification threshold | **500+** individuals affected |
| Audit log retention | **6 years** minimum |
| Substitute notice display time | **90 days** on website |
| Safe Harbor identifiers to remove | **18** specified types |
| ZIP code population threshold | **20,000** people (for 3-digit ZIPs) |
| Age generalization threshold | **89+** years (aggregate to 90+) |
| Restricted 3-digit ZIP codes | **17** codes must be changed to 000 |
| HIPAA penalty range | **$100 - $50,000** per violation |
| TLS minimum version | **TLS 1.2** (never SSL) |
| Encryption standard for PHI | **AES-128 or AES-256** |

---

## References

[Guidance Regarding Methods for De-identification of Protected Health Information in Accordance with the Health Insurance Portability and Accountability Act (HIPAA) Privacy Rule](https://www.hhs.gov/hipaa/for-professionals/special-topics/de-identification/index.html)

[De-identification of Protected Health Information: How to Anonymize PHI](https://www.hipaajournal.com/de-identification-protected-health-information/)

[45 CFR 164.504 - Organizational Requirements](https://www.ecfr.gov/current/title-45/subtitle-A/subchapter-C/part-164/subpart-E/section-164.504)

[45 CFR Part 164 Subpart D - Notification in the Case of Breach of Unsecured Protected Health Information](https://www.ecfr.gov/current/title-45/subtitle-A/subchapter-C/part-164/subpart-D)

[Encryption Policy Best Practices | TLS vs SSL](https://www.strongdm.com/blog/encryption-policy)

[What is Healthcare Data Security? Challenges & Best Practices](https://www.strongdm.com/blog/healthcare-data-security)

[HHS - Breach Notification Rule](https://www.hhs.gov/hipaa/for-professionals/breach-notification/index.html)

[HHS - Minimum Necessary Requirement](https://www.hhs.gov/hipaa/for-professionals/privacy/guidance/minimum-necessary-requirement/index.html)

[Incident Response Recommendations and Considerations for Cybersecurity Risk Management: A CSF 2.0 Community Profile](https://csrc.nist.gov/pubs/sp/800/61/r3/final)
