# IT Vendor Management Guide

## Overview
Comprehensive guide for IT vendor management covering selection, contracts, SLA management, risk assessment, performance monitoring, and relationship lifecycle. Designed for IT directors, managers, and procurement teams.

---

## 1. Vendor Management Framework

### Vendor Categorization
```
TIER 1 — STRATEGIC PARTNERS
  Criteria: Critical to business operations, high spend, long-term
  Examples: Cloud providers (AWS/Azure), ERP vendor, MSSP
  Management: Executive sponsor, QBRs, strategic roadmap sessions
  Review frequency: Monthly + Quarterly Business Reviews

TIER 2 — PREFERRED VENDORS
  Criteria: Important, moderate spend, recurring relationship
  Examples: Network hardware, security tools, major SaaS
  Management: IT Manager or Director ownership
  Review frequency: Quarterly

TIER 3 — APPROVED VENDORS
  Criteria: Lower spend, transactional, replaceable
  Examples: Commodity hardware, niche software
  Management: Procurement-led
  Review frequency: Annual

TIER 4 — SPOT/ONE-TIME
  Criteria: Single purchase, low risk, low spend
  Examples: One-time consultant, minor equipment
  Management: Standard procurement only
  Review frequency: None
```

### Vendor Register Fields
| Field | Description |
|-------|-------------|
| Vendor ID | Unique identifier |
| Company Name | Legal entity name |
| Tier | 1-4 classification |
| Category | Cloud, Hardware, Software, Services, etc. |
| Primary Contact | Vendor account manager |
| Internal Owner | IT team member responsible |
| Contract Expiry | Date |
| Annual Spend | $amount |
| Data Shared | What data types (PII, PHI, financial, etc.) |
| Access Level | None, Read, Admin, etc. |
| Risk Rating | Low, Medium, High, Critical |
| Compliance Status | Current, Pending, Non-compliant |

---

## 2. Vendor Selection Process

### Requirements Definition
```
VENDOR REQUIREMENTS DOCUMENT
Project: [Name]
Date: [Date]

FUNCTIONAL REQUIREMENTS (Must-Have):
F1. [Feature/capability required]
F2. [Integration requirements]
F3. [Performance requirements]

NON-FUNCTIONAL REQUIREMENTS:
N1. Uptime SLA: 99.9% minimum
N2. Data residency: US region only
N3. Encryption: AES-256 at rest, TLS 1.2+ in transit
N4. Support: 24/7 for critical issues

SECURITY REQUIREMENTS:
S1. SOC 2 Type II certification (current)
S2. ISO 27001 certification (preferred)
S3. Penetration test within last 12 months
S4. MFA for all admin access
S5. Background checks for personnel with data access
S6. Incident notification within 72 hours

COMPLIANCE REQUIREMENTS:
C1. GDPR compliant data processing
C2. HIPAA BAA available (if applicable)
C3. PCI DSS Level 1 (if applicable)
```

### RFP Evaluation Scorecard
```
EVALUATION SCORECARD — [Vendor Name]

CATEGORY          | WEIGHT | SCORE (1-5) | WEIGHTED
Technical Fit     | 30%    | [score]     | [calc]
Security/Compliance| 25%   | [score]     | [calc]
Total Cost (5yr)  | 20%    | [score]     | [calc]
Support & SLAs    | 10%    | [score]     | [calc]
Implementation    | 10%    | [score]     | [calc]
References        |  5%    | [score]     | [calc]
TOTAL             | 100%   |             | [total]

Scoring guide: 5=Excellent 4=Good 3=Satisfactory 2=Below expectations 1=Unacceptable
```

### Proof of Concept (PoC) Criteria
```
PoC Scope: [What you're testing]
Duration: [2-4 weeks typical]
Success Criteria:
  - [Performance metric, e.g., "Process 10,000 records in < 60 seconds"]
  - [Integration requirement tested]
  - [Security requirement validated]
  - [User acceptance threshold, e.g., "> 80% positive from pilot users"]
Participants: [5-10 representative users]
Decision: Go/No-Go based on criteria above
```

---

## 3. Contract Management

### Contract Types
| Type | Use Case | Key Considerations |
|------|---------|-------------------|
| MSA + SOW | Ongoing services | Master terms + project-specific SOW |
| SaaS Subscription | Cloud software | Auto-renewal terms, data ownership |
| Licensing Agreement | Software licenses | Per user/device/core, upgrade rights |
| Professional Services | One-time project | Milestone-based payment, IP ownership |
| Managed Services | Ongoing outsourced | SLAs, transition provisions |
| Hardware Purchase | Equipment | Warranty, maintenance, end-of-life |

### Critical Contract Clauses

**SLA Requirements:**
```
SERVICE LEVEL AGREEMENT MINIMUMS:
Uptime: 99.9% (8.7 hrs/year downtime)
Critical Response: 1 hour (P1 incidents)
High Response: 4 hours (P2 incidents)
Remedies: Service credits per breach (5-25% of monthly fee)
Measurement: Real-time dashboard, monthly report
Exclusions: Clearly defined (maintenance windows, force majeure)
```

**Data Provisions:**
```
DATA HANDLING CLAUSES:
- Data ownership: Customer retains all ownership
- Data portability: Machine-readable export within 30 days on request
- Data deletion: Certified deletion within 30 days of termination
- Data location: Specify permitted regions/countries
- Subprocessors: List of approved, notification of changes
- Audit rights: Right to audit or receive third-party audit report
- Breach notification: Vendor notifies within 24/48/72 hours
```

**Termination Provisions:**
```
TERMINATION:
- Convenience: 30/60/90 day written notice
- Cause: Material breach, immediate with cure period
- Insolvency: Immediate upon filing
- Post-termination: Data export period, transition assistance
- Transition: N months of continued service during replacement
```

**Liability and Indemnification:**
```
LIABILITY:
- Cap: 12 months of fees paid (minimum)
- Carve-outs: IP infringement, data breach, gross negligence (uncapped)
- Indemnification: Vendor indemnifies for their IP infringement
- Insurance: Require cyber liability ($5M+), E&O, general liability
```

### Contract Renewal Process
```
T-180 days: Flag contract for renewal review
  - Evaluate performance vs SLA
  - Assess vendor risk standing
  - Review pricing vs market

T-120 days: Renewal decision meeting
  - Keep as-is, renegotiate, or replace
  - If replace: Begin RFP process

T-90 days: Negotiate (if renewing)
  - Price benchmarking
  - SLA improvements
  - Terms modernization

T-30 days: Execute renewal or issue termination notice

T-0 days: Contract expires or renews
```

---

## 4. Vendor Risk Management

### Third-Party Risk Assessment

**Risk Factors:**
| Factor | Assessment Areas |
|--------|----------------|
| Data sensitivity | PII, PHI, financial, IP exposure |
| Access level | Network access, admin access, read-only |
| Criticality | Business impact if vendor fails |
| Financial stability | Revenue, ownership changes, funding |
| Geographic risk | Country of operation, data residency |
| Security posture | Certifications, breach history |
| Concentration risk | Over-reliance on single vendor |

**Risk Questionnaire — Key Areas:**
```
SECURITY QUESTIONNAIRE (Key Questions)

DATA HANDLING:
1. What security standards/certifications do you hold? (SOC 2, ISO 27001, etc.)
2. Attach your most recent SOC 2 Type II report.
3. How is customer data encrypted at rest and in transit?
4. What regions is customer data stored in?
5. Who has access to customer data? How is access logged and reviewed?

ACCESS CONTROLS:
6. Is MFA required for all employees accessing production systems?
7. How do you manage privileged access? (PAM solution?)
8. How quickly do you terminate access upon employee departure?

INCIDENT RESPONSE:
9. Do you have a documented incident response plan?
10. What is your process for notifying customers of a security incident?
11. Have you experienced a data breach in the last 3 years? If yes, describe.
12. What is your SLA for breach notification?

BUSINESS CONTINUITY:
13. What is your RTO/RPO for production systems?
14. When was your last DR test and what were the results?

SUBPROCESSORS:
15. Do you use subprocessors/subcontractors with access to customer data?
16. How do you manage security of your subprocessors?

COMPLIANCE:
17. Do you have a data processing agreement (DPA) available?
18. How do you handle data subject access requests?
```

### Risk Rating Calculation
```
VENDOR RISK MATRIX

Data Sensitivity Score:
- None/public: 1
- Internal only: 2
- PII/business confidential: 3
- PHI/financial/PCI: 4
- Regulated or highly sensitive: 5

Access Level Score:
- No access to our systems: 1
- Read-only access: 2
- Limited write access: 3
- Full application admin: 4
- Network/infrastructure admin: 5

Criticality Score:
- No business impact if unavailable: 1
- Minor impact: 2
- Moderate impact: 3
- Major impact: 4
- Critical/business-stopping: 5

Security Posture Score:
- SOC 2 Type II + ISO 27001 + recent pentest: 1 (low risk)
- SOC 2 Type II only: 2
- Self-attestation, no certification: 3
- No documentation available: 4
- Known incidents or issues: 5

INHERENT RISK = (Data + Access + Criticality) / 3

RESIDUAL RISK = Inherent Risk × (Security Posture / 3)
Risk = 1.0-1.5: Low | 1.6-2.5: Medium | 2.6-3.5: High | 3.6+: Critical
```

---

## 5. SLA Management

### SLA Tracking Dashboard Template
```
VENDOR SLA REPORT — [Vendor Name] — [Month/Year]

SERVICE: [Cloud hosting / Support / etc.]

AVAILABILITY:
Target:  99.9%
Actual:  [N]%
Status:  [GREEN | YELLOW | RED]
Incidents: [List any downtime events, duration]

SUPPORT RESPONSE:
P1 Response Target: 1 hour
P1 Response Actual: [avg] hours | Compliance: [N]%
P2 Response Target: 4 hours
P2 Response Actual: [avg] hours | Compliance: [N]%

SERVICE CREDITS DUE:
Month: $[amount] (if any)
YTD: $[amount]

OPEN ISSUES:
[List any unresolved issues with age]

TREND:
[3-month trend: Improving / Stable / Declining]
```

### Credit Escalation Process
1. Document downtime with start/end timestamps (screenshots of monitoring)
2. Reference SLA clause and calculate credit due
3. Submit credit request via vendor portal or email
4. Follow up if not acknowledged within 5 business days
5. Escalate to account manager / legal if unresponsive
6. Apply credits to invoice at next billing cycle

---

## 6. Quarterly Business Review (QBR)

### QBR Agenda Template
```
QUARTERLY BUSINESS REVIEW
Vendor: [Name] | Q[N] [Year]
Duration: 60-90 minutes

ATTENDEES:
Our side: IT Director, IT Manager, Finance (optional)
Vendor: Account Manager, Technical Account Manager, Executive Sponsor

AGENDA:
1. Executive Summary (5 min)
   - Relationship health: GREEN/YELLOW/RED
   - Key wins this quarter

2. Performance Review (15 min)
   - SLA scorecard vs. targets
   - Incidents and root cause analysis
   - Support ticket trends

3. Roadmap & Innovation (15 min)
   - Product roadmap relevant to us
   - New features/releases since last QBR
   - Our feature requests — status

4. Upcoming Work (10 min)
   - Projects this quarter
   - Resources needed from vendor
   - Training/certification

5. Issues & Escalations (10 min)
   - Open issues review
   - Action items from last QBR

6. Commercial Review (10 min)
   - Spend review
   - Contract milestones
   - Renewal / expansion discussion

7. Action Items (5 min)
   - Assign owners and due dates

POST-QBR: Distribute meeting notes + action items within 48 hours
```

---

## 7. Vendor Offboarding

### Vendor Termination Checklist
```
VENDOR TERMINATION CHECKLIST
Vendor: [Name] | Termination Date: [Date]

PRE-TERMINATION:
[ ] Termination notice delivered per contract terms
[ ] Replacement vendor/solution identified and ready
[ ] Data export initiated (request export in vendor format)
[ ] Data integrity verified in export
[ ] Transition plan documented

DURING TRANSITION:
[ ] Parallel run period (if applicable)
[ ] Knowledge transfer from vendor completed
[ ] Runbooks and documentation received
[ ] Integration cutover tested
[ ] Users/staff notified and trained on replacement

AT TERMINATION:
[ ] All system access revoked (SSO, service accounts, API keys)
[ ] Credentials changed where shared
[ ] Network firewall rules updated
[ ] VPN certificates/access removed
[ ] Vendor accounts in our systems closed

DATA CLEANUP:
[ ] Confirm vendor has deleted our data (get written certification)
[ ] Delete vendor data from our systems
[ ] Update data inventory/ROPA

POST-TERMINATION:
[ ] Final invoice settled
[ ] Service credits claimed
[ ] Vendor removed from approved vendor list
[ ] Lessons learned documented
[ ] Insurance certificates returned
[ ] Contract archived
```

---

## 8. Software License Management

### License Types
| Type | Description | Compliance Risk |
|------|-------------|----------------|
| Per User (Named) | Specific user only | High — track joiners/leavers |
| Per Device | Specific device | Medium — track device lifecycle |
| Concurrent/Floating | N simultaneous users | Medium — monitor peak usage |
| Subscription | Per user per month | Low — auto-managed |
| Enterprise Agreement | Site-wide, all users | Low — negotiated coverage |
| Per Core/CPU | Server processor-based | High — track VM configuration |
| OEM | Tied to specific hardware | Medium — track hardware changes |

### License Compliance Audit
```powershell
# Discover installed software from AD computers
$computers = Get-ADComputer -Filter {OperatingSystem -like "*Windows*"} | Select -Expand Name
$results = @()

foreach ($computer in $computers) {
    $software = Invoke-Command -ComputerName $computer -ScriptBlock {
        Get-ItemProperty HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*,
            HKLM:\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\* |
            Select DisplayName, DisplayVersion, Publisher
    } -ErrorAction SilentlyContinue

    foreach ($app in $software | Where DisplayName) {
        $results += [PSCustomObject]@{
            Computer = $computer
            Application = $app.DisplayName
            Version = $app.DisplayVersion
            Publisher = $app.Publisher
        }
    }
}

# Export and compare against license inventory
$results | Export-Csv "software_audit.csv" -NoTypeInformation

# Pivot — count by application
$results | Group-Object Application | 
    Select Name, Count | 
    Sort Count -Descending |
    Export-Csv "app_summary.csv" -NoTypeInformation
```

### Software Asset Management (SAM) Tools
| Tool | Best For |
|------|---------|
| Microsoft SCCM/MECM | Windows-heavy environments |
| Snow Software | Large, mixed environments |
| Flexera | Complex licensing, SaaS + on-prem |
| ManageEngine AssetExplorer | Mid-market |
| Intune + Defender for Endpoint | Cloud-first |

---

## 9. Cloud Provider Management

### Multi-Cloud Governance
```
CLOUD GOVERNANCE FRAMEWORK

ACCOUNT STRUCTURE:
- Management Account (billing, policies)
- Security Account (centralized security tooling)
- Log Archive Account (all logs, immutable)
- Production Accounts (by workload)
- Non-Production Accounts

CLOUD CENTER OF EXCELLENCE (CCoE):
- Develop cloud policies and guardrails
- Cloud architecture review board
- Cost optimization working group
- Security review process

SPENDING CONTROLS:
- Budget alerts at 80%, 100%, 120%
- Commitment (Reserved/Savings Plans) for baseline
- Tagging enforcement for cost allocation
- Monthly FinOps review

SECURITY GUARDRAILS (enforced via SCP/Policy):
- No public S3 buckets
- MFA required for all console access
- CloudTrail/Activity Logs must be enabled
- Approved regions only
- No root account usage
```

### Cloud Invoice Reconciliation
```
Monthly Cloud Cost Review Checklist:
[ ] Review spend vs. budget (by account, by service)
[ ] Identify top 5 cost drivers and validate
[ ] Check Reserved Instance/Savings Plan utilization
[ ] Review unused resources (idle VMs, unattached storage)
[ ] Validate tags for cost allocation accuracy
[ ] Dispute any unexpected charges (submit support case)
[ ] Update cost forecast for next month
[ ] Report to finance/leadership
```

---

*Last Updated: 2025 | IT Operations Documentation Library*
