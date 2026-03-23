# IT Compliance & Audit Guide

## Overview
Comprehensive guide to IT compliance frameworks, audit preparation, evidence collection, and control implementation. Covers SOC 2, ISO 27001, NIST CSF, PCI DSS, HIPAA, GDPR, and CIS Controls.

---

## 1. Compliance Frameworks Overview

### Framework Selection Guide
| Framework | Who Needs It | Focus |
|-----------|-------------|-------|
| SOC 2 Type II | SaaS/cloud service providers | Trust service criteria |
| ISO 27001 | Global orgs seeking certification | ISMS (Info Security Mgmt System) |
| NIST CSF | US government, critical infrastructure | Cybersecurity risk framework |
| PCI DSS | Any org handling payment cards | Cardholder data protection |
| HIPAA | Healthcare, business associates | Protected health information |
| GDPR | EU data subjects' data | Privacy and data rights |
| CIS Controls | Any organization | Prescriptive security controls |
| SOX | Publicly traded US companies | Financial reporting controls |

### Framework Relationships
```
NIST CSF
  ↕ (maps to)
ISO 27001 ←→ SOC 2
  ↕
CIS Controls v8 (implementation guidance)
```

---

## 2. SOC 2 Type II

### Trust Service Categories
| Category | Description |
|----------|-------------|
| **Security** | Required — Protection against unauthorized access |
| Availability | System availability per SLA |
| Processing Integrity | Complete, valid, accurate processing |
| Confidentiality | Confidential information protection |
| Privacy | Personal information handling |

### Common Criteria (CC) — Security Category

**CC1: Control Environment**
- Demonstrate management commitment to security
- Org chart, security policies approved by leadership
- Board/committee oversight of security

**CC2: Communication**
- Internal communication of security responsibilities
- External communication to customers (privacy notice)

**CC3: Risk Assessment**
- Annual risk assessment documented
- Risk register with owners, mitigations, residual risk

**CC4: Monitoring**
- Continuous monitoring program
- Vulnerability scanning results
- SOC monitoring alerts

**CC5: Control Activities**
- Access control policies
- Change management procedures
- Incident response plan

**CC6: Logical Access Controls**
- User access provisioning/deprovisioning
- MFA enforcement
- Privileged access management
- Quarterly access reviews

**CC7: System Operations**
- Anomaly detection and response
- Backup and recovery procedures
- Incident management

**CC8: Change Management**
- Software development lifecycle
- Change approval process
- Testing before production

**CC9: Risk Mitigation**
- Vendor risk management
- Business continuity planning

### SOC 2 Evidence Collection Guide

**Access Control Evidence:**
```
CC6.1 — User Provisioning
- Screenshots of onboarding process
- Ticket samples (5-10 with approval documented)
- AD/Entra ID user export (sanitized)
- Offboarding process samples

CC6.2 — Authentication
- MFA policy documented
- MFA enrollment report (% covered)
- Password policy configuration screenshots
- Conditional Access policy screenshots

CC6.3 — Access Reviews
- Quarterly access review completion records
- Remediation evidence (revocations done)
- Review sign-off by business owner

CC6.6 — Remote Access
- VPN usage logs
- MFA for VPN documented
- Certificate-based auth config
```

**Change Management Evidence:**
```
CC8.1 — Change Process
- Change request tickets (sample 10)
- Approvals documented in tickets
- Test results before promotion
- Change Advisory Board meeting minutes
- Rollback procedures
```

**Availability Evidence:**
```
A1.1 — Availability Commitments
- SLA documentation
- Uptime monitoring reports (99.9% target)
- Planned maintenance notifications

A1.2 — Capacity Management
- Capacity monitoring reports
- Performance metrics trending
- Cloud auto-scaling configurations
```

---

## 3. ISO 27001

### ISMS Scope Document
```
INFORMATION SECURITY MANAGEMENT SYSTEM SCOPE
Organization: [Name]
Version: 1.0 | Date: [Date]

IN SCOPE:
- Systems: [List production systems, networks, endpoints]
- Locations: [Offices, data centers, cloud regions]
- Departments: [IT, Engineering, Finance, HR]
- Services: [List services delivered to customers]

OUT OF SCOPE:
- [Explicitly excluded systems/locations]

INTERFACES AND DEPENDENCIES:
- Third-party vendors: [List with data flows]
- Cloud providers: [AWS, Azure, GCP etc.]

STATEMENT: This ISMS applies to all information assets
supporting [core business process] within [organization name].
```

### Annex A Controls (ISO 27001:2022)
**Organizational Controls (Clause 5):**
- A.5.1 Information security policies
- A.5.2 Roles and responsibilities
- A.5.3 Segregation of duties
- A.5.8 Security in project management
- A.5.15 Access control
- A.5.24 Incident management

**People Controls (Clause 6):**
- A.6.1 Screening (background checks)
- A.6.2 Terms of employment
- A.6.3 Security awareness training
- A.6.5 Termination procedures

**Physical Controls (Clause 7):**
- A.7.1 Physical security perimeters
- A.7.2 Physical entry
- A.7.8 Clear desk/screen policy
- A.7.14 Secure disposal

**Technological Controls (Clause 8):**
- A.8.1 User endpoint devices
- A.8.2 Privileged access rights
- A.8.5 Secure authentication
- A.8.7 Protection against malware
- A.8.8 Vulnerability management
- A.8.9 Configuration management
- A.8.15 Logging
- A.8.20 Network security
- A.8.24 Cryptography
- A.8.28 Secure coding

### Risk Assessment Process (ISO 27005)
```
1. ASSET INVENTORY
   - Information assets (databases, files, systems)
   - Software assets
   - Hardware assets
   - Service assets (cloud, outsourced)
   - People assets

2. THREAT IDENTIFICATION
   - Natural disasters (flood, fire)
   - Technical failures (hardware, software)
   - Human factors (error, malicious insider)
   - External attacks (hacking, malware)

3. VULNERABILITY ASSESSMENT
   - Vulnerability scans
   - Penetration testing
   - Configuration review
   - Policy gap analysis

4. RISK CALCULATION
   Risk = Likelihood × Impact (1-5 scale each)
   Risk Score: 1-5 = Low, 6-12 = Medium, 13-19 = High, 20-25 = Critical

5. RISK TREATMENT OPTIONS
   - Treat (implement controls)
   - Transfer (insurance, contracts)
   - Avoid (stop activity)
   - Accept (document residual risk)

6. STATEMENT OF APPLICABILITY (SoA)
   For each Annex A control: Applicable? | Justification | Implementation status
```

---

## 4. NIST Cybersecurity Framework (CSF 2.0)

### CSF Core Functions
```
GOVERN — Establish governance structures
   ↓
IDENTIFY — Understand assets and risks
   ↓
PROTECT — Implement safeguards
   ↓
DETECT — Identify cybersecurity events
   ↓
RESPOND — Contain and eradicate
   ↓
RECOVER — Restore capabilities
```

### Implementation Tiers
| Tier | Name | Description |
|------|------|-------------|
| 1 | Partial | Ad hoc, reactive, limited risk awareness |
| 2 | Risk Informed | Approved policies, not org-wide |
| 3 | Repeatable | Consistent enterprise-wide practices |
| 4 | Adaptive | Continuously improving, threat-informed |

### CSF Profile Template
```
Current Profile vs Target Profile Assessment:

Function | Category | Current (Tier) | Target (Tier) | Gap | Priority
IDENTIFY | Asset Management | 2 | 3 | Medium | High
IDENTIFY | Risk Assessment | 1 | 3 | High | Critical
PROTECT | Access Control | 3 | 4 | Low | Medium
PROTECT | Security Awareness | 2 | 3 | Medium | High
DETECT | Continuous Monitoring | 1 | 3 | High | Critical
RESPOND | Incident Response | 2 | 3 | Medium | High
RECOVER | Recovery Planning | 1 | 3 | High | High
```

---

## 5. PCI DSS (v4.0)

### 12 PCI DSS Requirements

| # | Requirement |
|---|-------------|
| 1 | Install and maintain network security controls |
| 2 | Apply secure configurations |
| 3 | Protect stored account data |
| 4 | Protect cardholder data in transit (TLS) |
| 5 | Protect from malicious software |
| 6 | Develop secure systems and software |
| 7 | Restrict access by business need |
| 8 | Identify users and authenticate access |
| 9 | Restrict physical access to cardholder data |
| 10 | Log and monitor all access to system components |
| 11 | Test security of systems regularly |
| 12 | Support security with org policies and programs |

### Cardholder Data Environment (CDE) Scoping

**In-Scope Systems:**
- Systems that store, process, or transmit PAN
- Systems that can connect to CDE
- Systems providing security services to CDE

**Scope Reduction Strategies:**
- Tokenization (replace PAN with token)
- Point-to-point encryption (P2PE) — removes POS terminals from scope
- Using validated payment processors
- Network segmentation (firewall between CDE and rest of network)

### Key PCI Technical Controls
```
Req 8 — Authentication Requirements:
- Unique user IDs for all users
- MFA for all non-console admin access
- MFA for all remote access
- Passwords: minimum 12 characters (as of v4.0)
- Account lockout after 10 failed attempts
- Session idle timeout: 15 minutes
- No shared/generic accounts

Req 10 — Logging Requirements:
- Log all individual user access to cardholder data
- Log all actions by root/admin
- Log all access to audit trails
- Log invalid login attempts
- Log use of identification/authentication
- Retain logs: 12 months (3 months online/accessible)
- Daily log review

Req 11 — Testing Requirements:
- Quarterly internal vulnerability scans
- Quarterly external vulnerability scans (ASV)
- Annual penetration test
- Penetration test after significant changes
- Weekly file integrity monitoring review
```

---

## 6. HIPAA

### HIPAA Rule Overview
| Rule | Focus |
|------|-------|
| Privacy Rule | PHI use and disclosure |
| Security Rule | Electronic PHI (ePHI) safeguards |
| Breach Notification | Notification requirements |
| Omnibus Rule | Business associate requirements |

### Security Rule: Required vs. Addressable
| Safeguard | Type | Examples |
|-----------|------|---------|
| Access control | Required | Unique user IDs, emergency access |
| Audit controls | Required | Hardware, software, activity tracking |
| Integrity controls | Addressable | Authentication mechanisms |
| Transmission security | Required | Encryption in transit |
| Facility access | Addressable | Physical access controls |
| Workstation security | Required | Physical safeguards for workstations |
| Contingency plan | Required | Backup, disaster recovery |
| BAA agreements | Required | All business associates |

### Business Associate Agreement (BAA) Requirements
Must include:
- Description of permitted/required uses of PHI
- BA cannot use PHI for own purposes
- Safeguards to prevent unauthorized use
- Reporting of breaches/unauthorized disclosures
- Sub-business associate requirements
- PHI return/destruction at contract termination

### Breach Assessment — 4-Factor Test
A breach is presumed unless ALL 4 factors are low risk:
1. Nature and extent of PHI involved
2. Who used/accessed the PHI
3. Whether PHI was actually acquired/viewed
4. Extent to which risk has been mitigated

**Breach Notification Timeline:**
- Individuals: Within 60 days of discovery
- HHS: Within 60 days (large breaches), annually for small
- Media: Within 60 days if 500+ in a state/jurisdiction

---

## 7. GDPR

### Key Principles (Article 5)
1. **Lawfulness, Fairness, Transparency** — Legal basis for processing
2. **Purpose Limitation** — Collected for specific, explicit purposes
3. **Data Minimisation** — Only what's necessary
4. **Accuracy** — Kept up to date
5. **Storage Limitation** — Not kept longer than necessary
6. **Integrity and Confidentiality** — Appropriate security
7. **Accountability** — Controller responsible for compliance

### Lawful Bases for Processing
- Consent (freely given, specific, informed, unambiguous)
- Contract performance
- Legal obligation
- Vital interests
- Public task
- Legitimate interests (requires balancing test)

### Data Subject Rights (IT Implementation)
| Right | IT Implementation |
|-------|------------------|
| Access (DSAR) | Ability to extract all data for a person across systems |
| Rectification | Update records in all systems |
| Erasure (Right to be Forgotten) | Delete across all systems, logs, backups |
| Restriction | Ability to pause processing without deletion |
| Data Portability | Machine-readable export (JSON/CSV) |
| Object | Ability to stop certain processing types |

### GDPR Technical Measures (Article 32)
```
Pseudonymisation and/or encryption of personal data
Ability to ensure ongoing confidentiality, integrity, availability
Ability to restore access to personal data after incident
Process for regularly testing and evaluating effectiveness

Practical controls:
- Encryption at rest (AES-256) and in transit (TLS 1.2+)
- Access logging and monitoring
- Data retention schedules and auto-deletion
- Regular penetration testing
- Privacy by design in new systems
- Data Protection Impact Assessment (DPIA) for high-risk processing
```

### Breach Notification (GDPR)
- **72 hours** from becoming aware: Notify supervisory authority (if risk to individuals)
- **Without undue delay:** Notify affected individuals (if high risk)
- No need to notify authority if: negligible risk, encrypted data lost with key retained

---

## 8. CIS Controls v8

### Implementation Groups
| IG | Organization Size | Cybersecurity Staff | Focus |
|----|------------------|---------------------|-------|
| IG1 | Small | Limited/none | Basic cyber hygiene (56 safeguards) |
| IG2 | Medium | Some dedicated | IG1 + additional 74 safeguards |
| IG3 | Large | Dedicated security team | All 153 safeguards |

### Top 18 Controls
| # | Control | Priority |
|---|---------|---------|
| 1 | Inventory and Control of Enterprise Assets | IG1 |
| 2 | Inventory and Control of Software Assets | IG1 |
| 3 | Data Protection | IG1 |
| 4 | Secure Configuration of Enterprise Assets | IG1 |
| 5 | Account Management | IG1 |
| 6 | Access Control Management | IG1 |
| 7 | Continuous Vulnerability Management | IG1 |
| 8 | Audit Log Management | IG1 |
| 9 | Email and Web Browser Protections | IG1 |
| 10 | Malware Defenses | IG1 |
| 11 | Data Recovery | IG1 |
| 12 | Network Infrastructure Management | IG1 |
| 13 | Network Monitoring and Defense | IG2 |
| 14 | Security Awareness and Skills Training | IG1 |
| 15 | Service Provider Management | IG2 |
| 16 | Application Software Security | IG2 |
| 17 | Incident Response Management | IG2 |
| 18 | Penetration Testing | IG2 |

---

## 9. Audit Preparation

### Internal Audit Planning
```
AUDIT PLAN TEMPLATE
Audit: [Control area, e.g., Access Control]
Period: [Date range]
Lead Auditor: [Name]
Target Completion: [Date]

OBJECTIVES:
- Verify [control 1] is designed and operating effectively
- Confirm [control 2] operates as documented
- Identify gaps or deficiencies

SCOPE:
- Systems in scope: [list]
- Departments: [list]
- Time period: [dates]

TEST PROCEDURES:
1. Review policy and procedure documentation
2. Select sample of [N] transactions/events
3. Inspect evidence for each sample
4. Interview process owners
5. Technical testing (where applicable)

DOCUMENTATION REQUIRED:
- [List all evidence requests]
```

### Evidence Management
```
Evidence Organization Structure:
/Audit_2025_SOC2/
├── CC6_Access_Controls/
│   ├── CC6.1_User_Provisioning/
│   │   ├── 01_Policy_Access_Control_v2.3.pdf
│   │   ├── 02_Onboarding_Tickets_Sample.xlsx
│   │   └── 03_AD_User_Export_20250101.csv
│   └── CC6.3_Access_Reviews/
│       ├── Q1_2025_Access_Review_Results.xlsx
│       └── Q1_2025_Review_Signoff.pdf
└── CC8_Change_Management/
    ├── 01_Change_Management_Policy.pdf
    └── 02_Change_Ticket_Samples.pdf
```

### Audit Finding Severity Ratings
| Rating | Description | Required Response Time |
|--------|-------------|----------------------|
| Critical | Immediate risk of significant harm | 30 days |
| High | Significant control weakness | 60 days |
| Medium | Moderate control deficiency | 90 days |
| Low | Minor finding or improvement opportunity | 180 days |
| Informational | Best practice recommendation | Plan for next cycle |

### Finding Response Template
```
AUDIT FINDING RESPONSE

Finding ID: [ID]
Finding Title: [Short description]
Severity: [Critical/High/Medium/Low]
Finding: [Auditor's description of the issue]

ROOT CAUSE:
[Why did this control fail or gap exist?]

MANAGEMENT RESPONSE:
[Accept/Dispute the finding — if dispute, provide rationale]

REMEDIATION PLAN:
Action 1: [What will be done]
Owner: [Name]
Target Date: [Date]

Action 2: [If applicable]

EVIDENCE OF COMPLETION:
[How will remediation be evidenced to auditor?]

STATUS: [Open | In Progress | Complete | Risk Accepted]
```

---

## 10. Compliance Program Management

### Annual Compliance Calendar
```
JANUARY:    Annual risk assessment kickoff
FEBRUARY:   Security awareness training renewal (all staff)
MARCH:      Q1 access reviews, vendor risk reviews
APRIL:      Internal audit planning
MAY:        Penetration testing (annual)
JUNE:       Q2 access reviews, policy review cycle
JULY:       Mid-year compliance status report
AUGUST:     External audit preparation
SEPTEMBER:  Q3 access reviews, SOC 2 / ISO readiness
OCTOBER:    External audit / certification audit
NOVEMBER:   Vulnerability management report
DECEMBER:   Q4 access reviews, year-end policy sign-off
            Business continuity test
            Disaster recovery test
```

### Compliance Metrics Dashboard
```
COMPLIANCE METRICS — [Month] [Year]

ACCESS CONTROL
  MFA Coverage: [N]% (target: 100%)
  Access Review Completion: [N]% (target: 100%)
  Orphaned Accounts (> 30 days): [N] (target: 0)
  Privileged Account Count: [N] (target: < [N])

VULNERABILITY MANAGEMENT
  Critical Vulns (unpatched > 15 days): [N] (target: 0)
  High Vulns (unpatched > 30 days): [N] (target: 0)
  Scan Coverage: [N]% (target: 100%)

SECURITY AWARENESS
  Training Completion Rate: [N]% (target: 95%)
  Phishing Simulation Click Rate: [N]% (target: < 5%)

INCIDENT MANAGEMENT
  MTTR (Critical): [N] hours (target: < 4)
  Incidents this month: [N]
  Open incidents > SLA: [N] (target: 0)

CHANGE MANAGEMENT
  Emergency Changes: [N] (target: < 5%)
  Failed Changes: [N] (target: < 5%)
  Unauthorized Changes: [N] (target: 0)
```

---

*Last Updated: 2025 | IT Operations Documentation Library*
