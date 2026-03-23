# Disaster Recovery & Business Continuity Planning
**Version:** 2.0 | **Audience:** IT Directors, Managers, Infrastructure Teams | **Last Updated:** 2025

---

## Table of Contents
1. [DR/BCP Foundations](#1-drbcp-foundations)
2. [Business Impact Analysis (BIA)](#2-business-impact-analysis-bia)
3. [Risk Assessment](#3-risk-assessment)
4. [Recovery Strategy Design](#4-recovery-strategy-design)
5. [DR Plan Documentation](#5-dr-plan-documentation)
6. [Infrastructure Recovery Runbooks](#6-infrastructure-recovery-runbooks)
7. [Testing & Exercises](#7-testing--exercises)
8. [Communication Plans](#8-communication-plans)
9. [Vendor & Third-Party DR](#9-vendor--third-party-dr)
10. [DR Program Governance](#10-dr-program-governance)

---

## 1. DR/BCP Foundations

### Key Terms Defined

| Term | Definition |
|---|---|
| **RTO** (Recovery Time Objective) | Maximum acceptable downtime before business impact becomes unacceptable |
| **RPO** (Recovery Point Objective) | Maximum acceptable data loss measured in time |
| **MTTR** (Mean Time to Recover) | Average time to restore a system after failure |
| **MTBF** (Mean Time Between Failures) | Average uptime between failures |
| **RLO** (Recovery Level Objective) | Minimum service level required at recovery point |
| **WRT** (Work Recovery Time) | Time to verify system integrity and restore normal operations |
| **MTD** (Maximum Tolerable Downtime) | RTO + WRT; absolute maximum before catastrophic business impact |

### BCP vs. DR Scope

```
Business Continuity Plan (BCP)
└── Broader: Covers ALL disruptions — IT, people, facilities, supply chain
    ├── Disaster Recovery Plan (DRP)
    │   └── IT-focused: How to restore technology systems
    ├── Crisis Management Plan
    │   └── Leadership response, communications, decision-making
    ├── Business Resumption Plan
    │   └── Manual processes while IT systems are down
    └── Occupant Emergency Plan
        └── Life safety, evacuation, facility response
```

### Standards & Frameworks

| Standard | Focus |
|---|---|
| ISO 22301 | Business Continuity Management Systems |
| NIST SP 800-34 | IT Contingency Planning |
| DRII BCLE | BCP professional certification framework |
| ITIL | Service continuity within IT service management |
| SOC 2 (CC9) | Business continuity as part of trust services criteria |

---

## 2. Business Impact Analysis (BIA)

### BIA Process

```
Step 1: Identify Business Processes
  → Interview department heads
  → Document all critical functions
  → Map dependencies

Step 2: Assess Impact of Disruption
  → Financial impact per hour
  → Operational impact (customers, employees)
  → Regulatory/compliance impact
  → Reputational impact

Step 3: Determine RTO/RPO/MTD
  → Based on impact assessment
  → Approved by business owners

Step 4: Identify IT Dependencies
  → Applications supporting each process
  → Infrastructure supporting each application
  → Data dependencies
```

### BIA Template

| Business Process | Dept Owner | Critical Period | Financial Impact/hr | RTO | RPO | MTD | Supporting Apps |
|---|---|---|---|---|---|---|---|
| Payment Processing | Finance | Always | $50,000 | 15 min | 5 min | 2 hrs | ERP, Payment Gateway, DB |
| Customer Service Portal | CX | Business hours | $10,000 | 1 hr | 30 min | 4 hrs | CRM, Web App, Auth |
| Order Management | Operations | Always | $25,000 | 30 min | 15 min | 2 hrs | OMS, Inventory DB |
| Employee HR Portal | HR | Payroll periods critical | $500 | 8 hrs | 4 hrs | 24 hrs | HRIS, SSO |
| Internal Email | All | Always | $5,000 | 4 hrs | 1 hr | 8 hrs | Exchange/M365 |
| Analytics/Reporting | Finance | Month-end | $1,000 | 48 hrs | 24 hrs | 5 days | Data Warehouse |

### System Criticality Tiers

```
Tier 1 — Mission Critical (RTO < 1hr, RPO < 15min)
  Examples: Payment systems, authentication, core APIs
  Strategy: Active-active, hot standby, synchronous replication
  DR: Automated failover, no manual steps required

Tier 2 — Business Critical (RTO < 4hr, RPO < 1hr)
  Examples: Core applications, CRM, ERP
  Strategy: Hot/warm standby, asynchronous replication
  DR: Orchestrated failover, minimal manual steps (<30min)

Tier 3 — Important (RTO < 24hr, RPO < 4hr)
  Examples: Internal tools, secondary apps
  Strategy: Warm standby, daily backups
  DR: Manual recovery process with runbook

Tier 4 — Standard (RTO < 72hr, RPO < 24hr)
  Examples: Dev/test, analytics, reporting
  Strategy: Cold standby or rebuild from backup
  DR: Restore from backup, rebuild if necessary
```

---

## 3. Risk Assessment

### Risk Register Template

| Risk ID | Threat | Likelihood (1-5) | Impact (1-5) | Risk Score | Current Controls | Residual Risk | Owner |
|---|---|---|---|---|---|---|---|
| R-001 | Ransomware attack | 4 | 5 | 20 | EDR, backups, segmentation | 12 | CISO |
| R-002 | Primary datacenter fire | 1 | 5 | 5 | DR site, fire suppression | 4 | Infra Lead |
| R-003 | ISP outage | 3 | 4 | 12 | Dual ISP, SD-WAN | 6 | NetOps |
| R-004 | Cloud provider outage | 2 | 4 | 8 | Multi-region deploy | 5 | Cloud Ops |
| R-005 | Key staff unavailable | 3 | 3 | 9 | Cross-training, runbooks | 6 | IT Manager |
| R-006 | Database corruption | 2 | 5 | 10 | Backups, replication, testing | 6 | DBA |
| R-007 | Flood (datacenter) | 1 | 5 | 5 | Elevated floor, DR site | 3 | Facilities |
| R-008 | DDoS attack | 3 | 3 | 9 | DDoS mitigation service | 5 | SecOps |

### Risk Scoring Matrix

```
Impact Scale:
5 = Catastrophic  (>$1M loss, regulatory action, company existential)
4 = Critical      ($100K-$1M, major customer impact, SLA breach)
3 = Significant   ($10K-$100K, moderate customer impact)
2 = Moderate      ($1K-$10K, minor customer impact)
1 = Minor         (<$1K, internal impact only)

Likelihood Scale:
5 = Almost certain (>90% probability this year)
4 = Likely         (50-90%)
3 = Possible       (10-50%)
2 = Unlikely       (1-10%)
1 = Rare           (<1%)

Score = Likelihood × Impact
20-25 = Critical: Immediate action required
12-19 = High: Action required within 30 days
6-11  = Medium: Action within 90 days
1-5   = Low: Monitor and accept
```

---

## 4. Recovery Strategy Design

### Recovery Strategies by Tier

```
Active-Active (Tier 1 only)
  ├── Traffic split across multiple sites simultaneously
  ├── Synchronous data replication (zero data loss)
  ├── Failover is transparent — no RTO
  ├── Most expensive option
  └── Examples: AWS multi-region with Route53 latency routing

Hot Standby (Tier 1-2)
  ├── Secondary site fully provisioned and receiving data
  ├── Near-synchronous replication (RPO: seconds to minutes)
  ├── Failover: Automated or near-automated (minutes)
  └── Examples: Azure Site Recovery, RDS Multi-AZ

Warm Standby (Tier 2-3)
  ├── Infrastructure pre-deployed but scaled down
  ├── Asynchronous replication (RPO: minutes to hours)
  ├── Failover: Scale up + route traffic (30 min - 4 hours)
  └── Examples: Scaled-down EC2 fleet, read replica promotion

Cold Standby / Pilot Light (Tier 3-4)
  ├── Minimal infrastructure (just core services)
  ├── Backup data shipped to secondary location
  ├── Failover: Provision + restore from backup (hours to days)
  └── Examples: AMIs + terraform apply, tape/object restore

Rebuild Strategy (Tier 4 only)
  ├── No secondary infrastructure
  ├── Rebuild from IaC + restore from backup
  ├── Failover: Build + restore (days)
  └── Acceptable for non-critical systems only
```

### Data Replication Options

| Technology | Type | RPO | Latency Impact | Cost |
|---|---|---|---|---|
| Synchronous DB replication | Block/Row | Near-zero | High | High |
| Async DB replication | Row | Minutes | Low | Medium |
| Storage replication (SAN-to-SAN) | Block | Near-zero | Moderate | High |
| Azure Site Recovery | Hypervisor | 30 seconds | Low | Medium |
| AWS Storage Gateway | Block/File | Minutes | Low | Low |
| Backup to object storage | File | Hours | None | Low |

---

## 5. DR Plan Documentation

### DR Plan Structure

```
Document: IT Disaster Recovery Plan
├── 1. Purpose, Scope, Objectives
├── 2. Roles and Responsibilities
│   ├── DR Coordinator
│   ├── Technical Recovery Teams
│   ├── Business Liaisons
│   └── Executive Sponsor
├── 3. Activation Criteria & Decision Authority
├── 4. Notification & Escalation Procedures
├── 5. System Recovery Procedures (by tier)
│   ├── Tier 1: Mission Critical
│   ├── Tier 2: Business Critical
│   ├── Tier 3: Important
│   └── Tier 4: Standard
├── 6. Application Recovery Runbooks (linked)
├── 7. Infrastructure Recovery Runbooks (linked)
├── 8. Data Recovery Procedures
├── 9. Communication Templates
├── 10. Return to Normal Operations
├── 11. Post-Incident Review
└── Appendices
    ├── A: Contact Directory
    ├── B: System Inventory
    ├── C: Vendor Contacts
    └── D: Network Diagrams
```

### DR Activation Decision Framework

```
INCIDENT OCCURS
     │
     ▼
Is production affected?
├── NO → Standard incident management
└── YES ▼
     Is recovery likely within 2 hours via normal means?
     ├── YES → Work the incident, re-evaluate every 30 min
     └── NO ▼
          Notify DR Coordinator + IT Director
               │
               ▼
          Assess: Is DR declaration warranted?
          Criteria:
          - Estimated downtime > RTO for Tier 1/2 systems
          - Data loss > RPO threshold
          - Multiple critical systems affected
          - Primary site inaccessible
               │
               ▼
     ┌─────────────────────────────────┐
     │  DECLARE DR EVENT               │
     │  - Activate DR team             │
     │  - Notify executive sponsor     │
     │  - Begin DR procedures          │
     │  - Open stakeholder bridge call │
     └─────────────────────────────────┘
```

---

## 6. Infrastructure Recovery Runbooks

### Runbook: Active Directory Recovery

```
RUNBOOK: AD-001 — Active Directory Forest Recovery
Priority: Tier 1
Estimated Recovery Time: 2-4 hours
Prerequisites: AD backup on Veeam, DR site DC pre-staged

Step 1: Assess Impact (15 min)
  [ ] Identify which DCs are affected
  [ ] Determine if forest-wide or domain-specific
  [ ] Test: Can users authenticate? Can DNS resolve?
  [ ] Decision: Restore DC vs. rebuild vs. seize FSMO roles

Step 2: Isolate Failed DCs (10 min)
  [ ] Remove failed DCs from DNS if still resolving
  [ ] Do NOT replicate from a known-bad DC (check replication health)
  [ ] Command: repadmin /showrepl * /csv > repl-status.csv

Step 3: Promote DR Site DC as Operations Master (if needed)
  [ ] Seize all 5 FSMO roles on surviving/DR DC:
      ntdsutil
      roles
      connections
      connect to server dc-dr-01.domain.com
      quit
      seize schema master
      seize domain naming master
      seize PDC
      seize RID master
      seize infrastructure master
      quit
      quit
  [ ] Verify: netdom query fsmo

Step 4: Restore from Backup (if full forest recovery)
  [ ] Boot first DC in DS Restore Mode (F8 → DSRM)
  [ ] Restore from Veeam/Windows Server Backup
  [ ] Perform authoritative restore if needed:
      ntdsutil
      authoritative restore
      restore subtree "DC=domain,DC=com"
  [ ] Reboot normally

Step 5: Verify and Test (30 min)
  [ ] Replication: repadmin /replsummary
  [ ] DNS: dcdiag /test:dns /v
  [ ] SYSVOL: net share (SYSVOL and NETLOGON should appear)
  [ ] Auth test: Test user login from client
  [ ] FSMO: netdom query fsmo

Step 6: Clean Up
  [ ] Remove metadata for failed DCs:
      ntdsutil > metadata cleanup > remove selected server DC-DEAD-01
  [ ] Update DNS if DCs had static IPs that changed
  [ ] Notify security team (auth system changes)
```

### Runbook: Database Recovery

```
RUNBOOK: DB-001 — SQL Server Recovery
Priority: Tier 1
Estimated Recovery Time: 30 min (failover) / 4 hrs (full restore)

Option A: Failover to AlwaysOn Secondary (Preferred)
Step 1: [ ] Confirm primary is unhealthy via SSMS or:
             SELECT * FROM sys.availability_groups
             SELECT * FROM sys.dm_hadr_availability_replica_states
Step 2: [ ] Force manual failover:
             ALTER AVAILABILITY GROUP [AG_Prod]
             FORCE_FAILOVER_ALLOW_DATA_LOSS;  -- Only if not synchronous
             -- Or preferred:
             ALTER AVAILABILITY GROUP [AG_Prod] FAILOVER;
Step 3: [ ] Update connection strings (or listener auto-handles)
Step 4: [ ] Verify application connectivity
Step 5: [ ] Check data loss (if any) via LSN comparison

Option B: Point-in-Time Restore from Backup
Step 1: [ ] Identify backup chain: FULL + DIFF + LOG backups
Step 2: [ ] Calculate target restore point (RPO)
Step 3: [ ] Restore in NORECOVERY mode:
             RESTORE DATABASE [Prod] FROM DISK = 'full_backup.bak'
             WITH NORECOVERY, STATS=10
             RESTORE DATABASE [Prod] FROM DISK = 'diff_backup.bak'
             WITH NORECOVERY
             RESTORE LOG [Prod] FROM DISK = 'log_backup.trn'
             WITH STOPAT = '2025-01-15 14:30:00', RECOVERY
Step 4: [ ] Verify database integrity:
             DBCC CHECKDB ([Prod]) WITH NO_INFOMSGS
Step 5: [ ] Update application connection strings
Step 6: [ ] Test application connectivity and data integrity
Step 7: [ ] Notify DBA and application owners of restore point
```

---

## 7. Testing & Exercises

### DR Test Types

| Test Type | Description | Frequency | Disruption |
|---|---|---|---|
| Document Review | Verify plan accuracy, contacts, procedures | Quarterly | None |
| Tabletop Exercise | Walkthrough scenario with team, no systems | Semi-annual | None |
| Walkthrough Test | Team rehearses roles, verify procedures | Annually | None |
| Simulation Test | Full scenario simulation, limited system access | Annually | Low |
| Parallel Test | Activate DR systems while production runs | Annually | Low |
| Full Failover Test | Complete failover to DR, production offline | 1-3 years | High |

### Tabletop Exercise Guide

```
Exercise: Ransomware Attack Scenario
Duration: 3 hours
Participants: IT Director, Security, Infrastructure, App Teams, Legal, Comms

Scenario Injection Timeline:
T+0:00  — Security analyst detects unusual encryption activity on file servers
T+0:30  — Confirmed ransomware on 3 file servers. How do you respond?
T+1:00  — Ransomware has spread to backup server. Backups may be compromised.
T+1:30  — CFO is asking when systems will be back. How do you communicate?
T+2:00  — Attackers contact company demanding ransom. Legal implications?
T+2:30  — Day 2: Can you restore from offsite backups? What's the sequence?
T+3:00  — Debrief: What worked? What gaps did we find?

Discussion Questions:
1. Who makes the DR declaration? What criteria?
2. What systems are isolated first? How?
3. How do you verify backup integrity before restoring?
4. What do you tell employees who can't work?
5. What regulatory notifications are required and when?
6. Do we rebuild from scratch or restore infected systems?
```

### DR Test Report Template

```
DR TEST REPORT
Test Date: [Date]
Test Type: [Parallel Test / Full Failover / Tabletop]
Test Lead: [Name]
Participants: [List]

Systems Tested:
- [App/System 1] — Target RTO: 1hr | Actual: 47 min | PASS
- [App/System 2] — Target RTO: 4hr | Actual: 5.5 hr | FAIL

Objectives Met:
✅ All Tier 1 systems recovered within RTO
✅ Data loss within RPO
❌ System X exceeded RTO by 90 minutes (runbook gap)
❌ DR contact list had 3 outdated phone numbers

Findings:
1. [Critical] Database failover runbook step 4 is ambiguous — caused 45 min delay
2. [High] DR backup credentials in Key Vault expired
3. [Medium] DR documentation not updated after November migration

Action Items:
| # | Finding | Owner | Due Date | Priority |
|---|---------|-------|----------|----------|
| 1 | Update DB runbook step 4 with exact commands | DBA | 2025-02-15 | Critical |
| 2 | Rotate DR Key Vault credentials | SecOps | 2025-01-31 | High |
| 3 | Update DR documentation | IT Manager | 2025-03-01 | Medium |

Next Test Scheduled: [Date]
```

---

## 8. Communication Plans

### Stakeholder Notification Matrix

| Audience | Method | Timing | Owner | Template |
|---|---|---|---|---|
| Executive Team | Phone + Email | Within 30 min | IT Director | EXEC-001 |
| IT Staff | Teams/Slack channel | Within 15 min | IT Manager | STAFF-001 |
| All Employees | Company-wide email | Within 1 hr | Corp Comms | EMP-001 |
| Key Customers | Account Manager call | Within 2 hrs | Sales/CX | CUST-001 |
| Board of Directors | IT Director → CEO → Board | If MTD exceeded | CTO/CEO | BOARD-001 |
| Regulators | Legal team | Per regulatory SLA | Legal | REG-001 |
| Press | "No comment" until approved | Only if public impact | PR | PRESS-001 |

### Communication Templates

```
TEMPLATE: EMP-001 — Employee Notification

Subject: IT System Disruption — [Date/Time]

Dear Team,

We are currently experiencing a disruption affecting [SYSTEM/SERVICE NAME].

Current Status: [Systems are unavailable / degraded / being restored]
Impact: [Who is affected and what they cannot do]
Estimated Resolution: [Time estimate or "under investigation"]

What you should do:
- [Specific action 1: e.g., Use manual order forms, contact details below]
- [Specific action 2: e.g., Do not attempt to log in repeatedly]

We will provide updates every [30/60] minutes at [location/channel].
Next update: [specific time]

For urgent needs: Contact [Name] at [phone/email]

Thank you for your patience.
[IT Operations Team]
```

```
TEMPLATE: EXEC-001 — Executive Notification

PRIORITY: P1 IT INCIDENT — IMMEDIATE ATTENTION REQUIRED
Date/Time: [Timestamp]

SITUATION: [2 sentences — what happened, what is affected]

IMPACT:
- Business: [Revenue/customer/operational impact]
- Duration: [Estimated downtime vs. RTO target]
- Data: [Any data loss?]

CURRENT ACTIONS:
- [Team] is [action] — ETA: [time]
- [Team] is [action] — ETA: [time]

DECISION NEEDED (if any):
- [Approve DR spend of $X?]
- [Notify regulator by deadline?]

NEXT BRIEFING: [Time]
BRIDGE LINE: [Number / Link]
INCIDENT COMMANDER: [Name, Phone]
```

---

## 9. Vendor & Third-Party DR

### Vendor DR Questionnaire

```
Questions to ask critical vendors:
1. What is your published SLA for this service?
2. What is your RTO and RPO in a major outage?
3. Do you have active-active or active-passive DC setup?
4. What geographic regions are your DCs in?
5. Do you have a tested DR plan? When was it last tested?
6. What is your notification SLA for incidents?
7. Do you have a status page? URL?
8. What escalation path exists for P1 outages?
9. What contractual remedies exist for SLA breaches?
10. Can you provide your SOC 2 Type II or ISO 22301 certificate?
```

### Vendor Risk Tiers

```
Tier 1 — Critical Vendors (outage stops business):
  - Microsoft 365, Salesforce, AWS/Azure
  Actions:
  - Include in BIA
  - Monitor vendor status pages
  - Test manual fallback procedures
  - Review DR capability in contract
  - Maintain offline export of critical data

Tier 2 — Important Vendors (outage severely impacts productivity):
  - Slack, Zoom, Okta, ServiceNow
  Actions:
  - Document manual workarounds
  - Alternative tool identified
  - Include in tabletop exercises

Tier 3 — Standard Vendors (outage causes inconvenience):
  - Monitoring tools, developer tools
  Actions:
  - Note in vendor register
  - No specific DR actions required
```

---

## 10. DR Program Governance

### DR Program KPIs

| Metric | Target | Measurement |
|---|---|---|
| % of Tier 1/2 systems with tested DR | 100% | Annual |
| DR plan review currency | < 12 months old | Quarterly check |
| DR test completion rate | 100% scheduled tests | Quarterly |
| RTO achievement rate in tests | > 90% | Per test |
| Action item closure rate | > 85% in 30 days | Monthly |
| Staff trained on DR procedures | 100% of DR team | Annual |

### Annual DR Calendar

```
Q1 (Jan-Mar):
  - Annual DR plan review and update
  - Update contact lists
  - Review and update system inventory
  - Tabletop exercise: Infrastructure failure scenario

Q2 (Apr-Jun):
  - Tier 1 systems parallel test
  - Update risk register
  - Vendor DR review (top 5 vendors)
  - DR awareness training for new staff

Q3 (Jul-Sep):
  - Tabletop exercise: Cybersecurity scenario
  - Backup restoration test (random systems)
  - Review and update runbooks
  - BIA review with business owners

Q4 (Oct-Dec):
  - Annual DR test (full scope)
  - DR program effectiveness report to leadership
  - Budget planning for next year improvements
  - Update DR plan with year's changes
```

### Continuous Improvement

```
Post-Test/Incident Review Process:
1. Debrief within 48 hours of test/incident
2. Document all findings in test report
3. Categorize findings: Critical / High / Medium / Low
4. Assign owners and due dates
5. Track action items in project tracker
6. Review closure at next DR committee meeting
7. Update plan/runbooks with lessons learned
8. Report trends to IT Director quarterly
```

---

*This plan should be reviewed and updated at minimum annually, and after any significant infrastructure change, major incident, or organizational change.*
