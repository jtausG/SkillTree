# IT Governance, Risk & Compliance (GRC) Guide

## Table of Contents
1. [GRC Fundamentals](#grc-fundamentals)
2. [IT Risk Management](#risk-management)
3. [Compliance Frameworks Overview](#compliance-frameworks)
4. [SOC 2 Preparation](#soc2)
5. [ISO 27001 Implementation](#iso27001)
6. [NIST Cybersecurity Framework](#nist-csf)
7. [Policy Management](#policy-management)
8. [Audit Preparation](#audit-preparation)
9. [Vendor Risk Management](#vendor-risk)
10. [GRC Tools & Metrics](#grc-tools)

---

## 1. GRC Fundamentals

### What is GRC?
- **Governance** — How decisions are made; who is accountable; organizational structure and oversight
- **Risk** — Identifying, assessing, and mitigating threats to organizational objectives
- **Compliance** — Adhering to laws, regulations, standards, and internal policies

### Why GRC Matters for IT
- Regulatory fines can exceed millions of dollars
- Cyber insurance requires demonstrated controls
- Customer contracts increasingly require compliance attestations
- Board and executive accountability for information security
- Third-party audits are standard in enterprise deals

### GRC Organizational Roles

| Role | Responsibility |
|------|---------------|
| CISO | Overall security strategy and risk ownership |
| IT Director | IT governance, budget, vendor oversight |
| GRC Manager/Analyst | Framework implementation, audit coordination |
| Risk Officer | Enterprise risk register, risk reporting |
| Compliance Officer | Regulatory tracking, audit management |
| Data Privacy Officer (DPO) | GDPR/privacy compliance |

---

## 2. IT Risk Management

### Risk Assessment Methodology

**Risk Formula:**
```
Risk = Likelihood × Impact

Likelihood levels:
  1 = Rare (< 5% probability in 12 months)
  2 = Unlikely (5–25%)
  3 = Possible (25–50%)
  4 = Likely (50–75%)
  5 = Almost Certain (> 75%)

Impact levels:
  1 = Negligible (minimal financial/operational impact)
  2 = Minor (< $10K loss, short outage)
  3 = Moderate ($10K–$100K, hours of downtime)
  4 = Major ($100K–$1M, days of downtime, regulatory notice)
  5 = Catastrophic (> $1M, business-threatening, legal action)

Risk Score (1–25):
  1–6:   Low — Accept or monitor
  7–12:  Medium — Mitigate or transfer
  13–19: High — Priority mitigation required
  20–25: Critical — Immediate action, escalate to leadership
```

### Risk Register Template

```
Risk ID: RISK-2024-042
Title: Ransomware attack on file servers
Category: Cybersecurity
Asset(s): File servers, NAS storage
Threat: Ransomware encryption of business data
Vulnerability: Incomplete endpoint detection; slow patch cycle
Current Controls: Antivirus, daily backup, email filtering
Inherent Risk: Likelihood 4 × Impact 5 = 20 (Critical)
Control Effectiveness: Moderate
Residual Risk: Likelihood 2 × Impact 5 = 10 (Medium)
Treatment: MITIGATE
Treatment Plan: Deploy EDR, implement immutable backup, network segmentation
Owner: CISO
Due Date: 2024-Q2
Status: In Progress
Last Reviewed: 2024-01-15
```

### Risk Treatment Options

```
Accept:   Risk is within appetite; cost to mitigate > risk value
Mitigate: Implement controls to reduce likelihood or impact
Transfer: Cyber insurance, contractual liability shift
Avoid:    Stop doing the activity that creates the risk

Risk Appetite Statement examples:
- "We accept low risks with annual probability < $10K impact"
- "We do not accept any risk of unauthorized access to PII"
- "We transfer all risks exceeding $1M through cyber insurance"
```

### Key Risk Indicators (KRIs)

```
Security KRIs:
□ % of systems with critical patches applied within 30 days
□ Mean Time to Detect (MTTD) security incidents
□ % of users with MFA enabled
□ Number of phishing simulation failures per month
□ Number of critical vulnerabilities open > 30 days
□ % of privileged accounts reviewed in last 90 days

Operational KRIs:
□ System availability % vs. SLA target
□ Number of unplanned outages per month
□ Backup success rate %
□ % of changes that cause incidents
□ DR test completion and results
□ % of IT assets in CMDB vs. actual count
```

---

## 3. Compliance Frameworks Overview

### Framework Comparison

| Framework | Focus | Audience | Certification? |
|-----------|-------|---------|---------------|
| SOC 2 | Security/availability/privacy controls | Cloud/SaaS vendors | Yes (CPA audit) |
| ISO 27001 | Information security management system | Any organization | Yes (accredited body) |
| NIST CSF | Cybersecurity risk management | US organizations | No (framework) |
| CIS Controls | Specific security controls | Technical teams | No (framework) |
| PCI DSS | Payment card data security | Card processors | Yes (QSA audit) |
| HIPAA | Healthcare data privacy/security | US healthcare | No (self-attestation) |
| GDPR | Personal data protection | EU personal data processors | No (regulatory) |
| FedRAMP | Cloud for US federal government | Cloud providers to fed agencies | Yes |
| CMMC | Defense contractor security | DoD contractors | Yes |

### Mapping Across Frameworks

```
Control domain: Access Control

NIST CSF:   PR.AC — Protect / Access Control
ISO 27001:  A.9 — Access Control domain
CIS v8:     Control 5 — Account Management, Control 6 — Access Control Management
SOC 2:      CC6 — Logical and Physical Access
PCI DSS:    Req. 7 — Restrict Access, Req. 8 — Identify and Authenticate

Benefit: Implementing strong access controls satisfies requirements across all frameworks simultaneously
```

---

## 4. SOC 2 Preparation

### SOC 2 Trust Services Criteria

```
Security (CC): Required for all SOC 2 reports
  - CC1: Control environment (tone at top, HR controls)
  - CC2: Communication and information
  - CC3: Risk assessment
  - CC4: Monitoring activities
  - CC5: Control activities (policies, procedures)
  - CC6: Logical and physical access controls
  - CC7: System operations
  - CC8: Change management
  - CC9: Risk mitigation

Availability (A): System available per commitments (optional)
Confidentiality (C): Confidential information protected (optional)
Processing Integrity (PI): Processing complete and accurate (optional)
Privacy (P): Personal information collected/used per policy (optional)
```

### SOC 2 Readiness Assessment

```
Phase 1: Gap Assessment (4–8 weeks)
  □ Identify system(s) in scope
  □ Map current controls to TSC criteria
  □ Document control gaps
  □ Prioritize gap remediation

Phase 2: Remediation (3–6 months)
  □ Implement missing controls
  □ Document policies and procedures
  □ Deploy required technical controls
  □ Begin collecting evidence

Phase 3: Observation Period (Type 2: minimum 6 months)
  □ Operate controls consistently
  □ Collect evidence continuously
  □ Internal audit / pre-assessment
  □ Remediate findings

Phase 4: Audit (4–8 weeks)
  □ Auditor walkthroughs
  □ Evidence review
  □ Management responses to exceptions
  □ Report issuance
```

### Evidence Collection Checklist (SOC 2)

```
Access Management:
□ User access review records (quarterly)
□ Privileged access review records
□ Terminated user account removal logs
□ New user provisioning approval records
□ MFA enforcement evidence

Change Management:
□ Change tickets with approvals
□ Change advisory board meeting minutes
□ Production deployment approvals
□ Rollback procedure documentation

Vulnerability Management:
□ Scan reports (monthly minimum)
□ Remediation tracking records
□ Patch management reports

Incident Response:
□ IR procedure documentation
□ Incident logs and response actions
□ Post-incident review records

Availability:
□ System uptime metrics
□ Maintenance window notifications
□ Capacity monitoring reports

Vendor Management:
□ Vendor security assessments
□ Vendor contracts with security requirements
□ Critical vendor SOC 2 reports

HR Controls:
□ Background check policy and samples
□ Security awareness training completion records
□ Acceptable use policy acknowledgments
```

---

## 5. ISO 27001 Implementation

### Implementation Phases

```
Phase 1: Context & Scope
  - Define scope of the ISMS
  - Identify interested parties (regulators, customers, employees)
  - Understand legal and regulatory requirements
  - Document organizational context

Phase 2: Leadership & Planning
  - Obtain top management commitment
  - Define information security policy
  - Assign roles (ISMS owner, risk owner)
  - Define risk assessment methodology
  - Conduct risk assessment and treatment

Phase 3: Support
  - Determine required resources
  - Security awareness program
  - Document control of documented information
  - Internal/external communications plan

Phase 4: Operation
  - Implement risk treatment plan
  - Implement Annex A controls (as applicable)
  - Conduct supplier/vendor security assessments
  - Manage security incidents

Phase 5: Performance Evaluation
  - Monitor and measure ISMS performance
  - Internal audit program (annual minimum)
  - Management review (annual minimum)

Phase 6: Improvement
  - Corrective action for nonconformities
  - Continual improvement process
  - External certification audit (Stage 1 + Stage 2)
```

### ISO 27001 Annex A Key Domains

```
A.5  — Information Security Policies
A.6  — Organization of Information Security
A.7  — Human Resource Security
A.8  — Asset Management
A.9  — Access Control
A.10 — Cryptography
A.11 — Physical and Environmental Security
A.12 — Operations Security
A.13 — Communications Security
A.14 — System Acquisition, Development and Maintenance
A.15 — Supplier Relationships
A.16 — Information Security Incident Management
A.17 — Business Continuity Management
A.18 — Compliance
```

---

## 6. NIST Cybersecurity Framework

### CSF 2.0 Core Functions

```
GOVERN (new in CSF 2.0):
  GV.OC — Organizational Context
  GV.RM — Risk Management Strategy
  GV.RR — Roles, Responsibilities, Authorities
  GV.PO — Policies, Processes, Procedures
  GV.OV — Oversight
  GV.SC — Cybersecurity Supply Chain Risk Management

IDENTIFY:
  ID.AM — Asset Management
  ID.RA — Risk Assessment
  ID.IM — Improvement

PROTECT:
  PR.AA — Identity Management and Access Control
  PR.AT — Awareness and Training
  PR.DS — Data Security
  PR.PS — Platform Security
  PR.IR — Technology Infrastructure Resilience

DETECT:
  DE.AE — Adverse Event Analysis
  DE.CM — Continuous Monitoring

RESPOND:
  RS.MA — Incident Management
  RS.AN — Incident Analysis
  RS.CO — Incident Response Reporting and Communication
  RS.MI — Incident Mitigation

RECOVER:
  RC.RP — Incident Recovery Plan
  RC.CO — Incident Recovery Communication
```

### NIST CSF Maturity Tiers

```
Tier 1 — Partial:
  - Risk management not formalized
  - Limited awareness of cybersecurity risk
  - Ad-hoc processes, reactive approach

Tier 2 — Risk Informed:
  - Risk management approved but not org-wide
  - Some awareness of risk
  - Processes and procedures in place but not consistently applied

Tier 3 — Repeatable:
  - Formally approved risk management policies
  - Organization-wide consistent practices
  - Risk-informed decisions regularly made

Tier 4 — Adaptive:
  - Continuous improvement based on lessons learned
  - Proactive risk management
  - Cybersecurity deeply integrated in organizational culture

Goal: Progress from current tier toward Tier 3–4
```

---

## 7. Policy Management

### Policy Hierarchy

```
Level 1: Policy
  High-level statement of management intent
  "All employees must use MFA to access corporate systems"
  Owner: CISO / IT Director
  Review: Annual

Level 2: Standard
  Specific mandatory requirements
  "MFA must use TOTP or hardware key; SMS OTP not permitted"
  Owner: Security Manager
  Review: Annual

Level 3: Procedure
  Step-by-step how to comply with the standard
  "How to enroll in Microsoft Authenticator"
  Owner: IT Operations
  Review: When process changes

Level 4: Guideline
  Recommended best practices (non-mandatory)
  "Best practices for securing your home office network"
  Owner: IT Security
  Review: Annual
```

### Essential IT Policies

```
Foundational Policies (required for most frameworks):
□ Information Security Policy (umbrella policy)
□ Acceptable Use Policy (AUP)
□ Access Control Policy
□ Password / Authentication Policy
□ Data Classification Policy
□ Data Retention and Disposal Policy
□ Incident Response Policy
□ Change Management Policy
□ Business Continuity / Disaster Recovery Policy
□ Vendor / Third Party Security Policy
□ Remote Work / Telework Policy
□ BYOD Policy
□ Mobile Device Policy
□ Patch Management Policy
□ Vulnerability Management Policy
□ Physical Security Policy
□ Encryption Policy
□ Logging and Monitoring Policy
□ Privacy Policy (if handling personal data)
□ Software Development Security Policy (if developing software)
```

### Policy Lifecycle

```
Draft → Review (technical) → Review (legal) → 
Management Approval → Communication → 
Training → Implementation → 
Annual Review → Update or Retire
```

---

## 8. Audit Preparation

### Audit Types

| Type | Who conducts | Purpose |
|------|-------------|---------|
| Internal audit | Internal team or internal audit function | Self-assessment, gap identification |
| External audit | Independent third party | Certification, customer assurance |
| Regulatory exam | Regulator (FTC, HHS, SEC) | Enforcement, compliance verification |
| Penetration test | Security firm | Technical vulnerability assessment |
| Tabletop exercise | Facilitated simulation | Test IR and continuity plans |

### Pre-Audit Preparation

```
90 days before:
□ Review scope — confirm what systems/processes are in scope
□ Review prior year findings — confirm remediation complete
□ Inventory all policies — ensure current, approved, distributed
□ Review access lists — identify any anomalies
□ Verify logging is functioning and retention is correct

30 days before:
□ Compile evidence package (access reviews, change logs, training records)
□ Brief team members who will be interviewed
□ Assign evidence request coordinator
□ Pre-audit internal walkthrough
□ Remediate any obvious gaps

Day of audit:
□ Have all evidence pre-organized in shared drive or GRC tool
□ Brief technical staff on professional interview conduct:
  - Answer only what is asked
  - "I don't know" is better than speculation
  - Escalate to manager if unsure
□ Single point of contact for all auditor requests
□ Log all auditor requests and responses
```

### Responding to Audit Findings

```
Finding structure:
- Observation: What the auditor found
- Criteria: What was expected (policy, framework requirement)
- Risk: Why it matters
- Recommendation: What should be done

Your response:
1. Management response: Agree / Disagree with finding
2. Remediation plan: Specific actions to address finding
3. Owner: Who is responsible
4. Target date: When it will be fixed
5. Compensating controls: Interim mitigations while remediation is in progress

Response writing tips:
- Be specific — "We will implement X by Y date, assigned to Z"
- Avoid defensiveness — auditors are helping you
- Don't agree to unrealistic timelines
- For findings you disagree with: explain clearly with evidence
```

---

## 9. Vendor Risk Management

### Vendor Classification

```
Tier 1 — Critical / High Risk:
  Definition: Stores/processes confidential data; critical for operations
  Examples: Cloud infrastructure, HRIS, ERP, payroll
  Requirements: Full security questionnaire, SOC 2 report, contract security requirements, annual review

Tier 2 — Moderate Risk:
  Definition: Limited data access or non-critical services
  Examples: Marketing tools, project management, conferencing
  Requirements: Abbreviated questionnaire, review vendor security page, annual review

Tier 3 — Low Risk:
  Definition: No data access, non-critical
  Examples: Office supplies, facilities vendors
  Requirements: Standard contract terms, initial registration only
```

### Vendor Security Assessment

```
Standard questionnaire topics:
□ Company overview and certifications (SOC 2, ISO 27001)
□ Data handling — what data stored, where, how classified
□ Access controls — how access to your data is controlled
□ Encryption — in transit and at rest
□ Vulnerability management — patching cadence
□ Incident response — notification SLA, past incidents
□ Sub-processors — who else has access to your data
□ Business continuity — RTO/RPO, last DR test
□ Employee screening — background checks
□ Termination — data deletion/return procedures
□ Audit rights — can you audit them?

Red flags:
⚠ No SOC 2 or equivalent certification
⚠ Won't answer security questions
⚠ Cannot name where data is stored (country/region)
⚠ No incident history but no IR plan either
⚠ No data deletion/return process at termination
⚠ Sub-processors without controls
```

### Contract Security Requirements

```
Standard security clauses to include:
□ Data Processing Agreement (DPA) — GDPR requirement
□ Confidentiality provisions
□ Security incident notification SLA (typically 72 hours or less)
□ Right to audit (or accept SOC 2 as equivalent)
□ Data retention and deletion obligations
□ Subprocessor approval requirements
□ Security control requirements (encryption, access control, MFA)
□ Background check requirements for staff with data access
□ Liability and indemnification for data breach
□ Insurance requirements (cyber liability)
□ Business continuity requirements
□ Termination data return/destruction
```

---

## 10. GRC Tools & Metrics

### GRC Platform Categories

| Category | Examples | Use Case |
|----------|---------|---------|
| Risk Management | RSA Archer, MetricStream, LogicGate | Enterprise risk register, workflow |
| Compliance Management | Vanta, Drata, Tugboat Logic | Automated SOC 2 / ISO evidence |
| Policy Management | PolicyTech, ConvergePoint | Policy lifecycle management |
| Audit Management | AuditBoard, TeamMate+ | Audit planning and findings |
| Vendor Risk | OneTrust, ProcessUnity, BitSight | Vendor questionnaires and scoring |
| GRC All-in-One | ServiceNow GRC, ServiceNow IRM | Enterprise integrated GRC |

### Lightweight Approach (Small IT Teams)

```
Risk Register: Excel / Google Sheets (works fine to start)
Policy Management: SharePoint / Confluence with approval workflow
Evidence Collection: SharePoint document library, organized by control
Audit Tracking: Project tracker (Asana, Jira, Monday)
Vendor Risk: Excel tracker with annual review reminders

Automate when you have > 50 controls or > 2 audits/year
```

### GRC Dashboard Metrics

```
Executive-level KPIs:
□ Overall risk score (trend up/down)
□ % of critical risks with mitigation plans
□ Compliance status by framework
□ Open audit findings by severity and age
□ Policy attestation completion rate
□ Security awareness training completion rate
□ Vendor risk assessment completion rate

Operational metrics:
□ Controls tested vs. total in scope
□ Evidence collection completion %
□ Findings remediated on time vs. overdue
□ Risk register review cadence (last reviewed)
□ Policy review overdue count
□ Vendor assessments overdue
```

### GRC Maturity Roadmap

```
Year 1: Foundation
  - Establish risk register and assessment methodology
  - Develop essential policy set
  - Map controls to primary compliance framework
  - Initiate vendor risk program
  - Assign GRC ownership

Year 2: Formalization
  - Achieve first compliance certification (SOC 2 Type 1 or ISO 27001)
  - Implement access review processes
  - Establish audit program (internal)
  - Integrate GRC into change management
  - Board-level security reporting

Year 3: Optimization
  - SOC 2 Type 2 or ISO surveillance audit
  - Automate evidence collection
  - Implement GRC tooling
  - Risk-based security investment decisions
  - Third-party penetration testing annually
  - Mature vendor risk program with tiering
```
