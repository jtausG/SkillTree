# ITIL & IT Service Management (ITSM) Guide
**Version:** 2.0 | **Audience:** IT Managers, Service Desk Leaders, ITSM Practitioners | **Last Updated:** 2025

---

## Table of Contents
1. [ITIL 4 Framework Overview](#1-itil-4-framework-overview)
2. [Incident Management](#2-incident-management)
3. [Problem Management](#3-problem-management)
4. [Change Enablement](#4-change-enablement)
5. [Service Request Management](#5-service-request-management)
6. [Configuration Management (CMDB)](#6-configuration-management-cmdb)
7. [Knowledge Management](#7-knowledge-management)
8. [SLA Management](#8-sla-management)
9. [Continual Improvement](#9-continual-improvement)
10. [ITSM Tooling Guide](#10-itsm-tooling-guide)
11. [ITSM Metrics & Reporting](#11-itsm-metrics--reporting)

---

## 1. ITIL 4 Framework Overview

### The Four Dimensions

```
         ┌────────────────────────────────────────┐
         │           VALUE CHAIN                  │
         │    Plan → Improve → Engage →           │
         │    Design/Transition → Obtain/Build →  │
         │    Deliver/Support                     │
         └────────────────────────────────────────┘
                      Surrounded by:

1. Organizations & People
   - Roles, responsibilities, culture
   - Skills and competencies

2. Information & Technology
   - Data, knowledge, tools
   - Technologies enabling services

3. Partners & Suppliers
   - Third-party relationships
   - Contracts and agreements

4. Value Streams & Processes
   - How work flows through the organization
   - Processes and procedures
```

### Key ITIL 4 Guiding Principles

| Principle | Application |
|---|---|
| Focus on value | Every activity should contribute to value for customers/org |
| Start where you are | Assess current state before changing |
| Progress iteratively with feedback | Small improvements, measure each step |
| Collaborate and promote visibility | Break silos, share information |
| Think and work holistically | No process operates in isolation |
| Keep it simple and practical | Eliminate waste and complexity |
| Optimize and automate | Manual → optimized → automated |

### ITIL 4 Service Value System

```
Opportunity/Demand
      │
      ▼
┌─────────────────────────────────────────────────────┐
│                SERVICE VALUE SYSTEM                 │
│  ┌─────────────┐  ┌──────────────┐  ┌───────────┐  │
│  │  Guiding    │  │   Service    │  │Governance │  │
│  │ Principles  │  │ Value Chain  │  │           │  │
│  └─────────────┘  └──────────────┘  └───────────┘  │
│  ┌─────────────┐  ┌──────────────┐                  │
│  │  Practices  │  │  Continual   │                  │
│  │  (34 total) │  │ Improvement  │                  │
│  └─────────────┘  └──────────────┘                  │
└─────────────────────────────────────────────────────┘
      │
      ▼
    Value
```

---

## 2. Incident Management

### Incident Classification

```
Priority Matrix:
                  HIGH IMPACT    LOW IMPACT
HIGH URGENCY      P1 (Critical)  P2 (High)
LOW URGENCY       P2 (High)      P3 (Medium)/P4 (Low)

Impact Definition:
- High: Multiple users/departments affected, business-critical service
- Medium: Single team affected or single user in critical role
- Low: Individual user, non-critical function

Urgency Definition:
- High: Business operations halted or severely degraded
- Low: Workaround available, business can continue
```

### Priority SLAs

| Priority | Label | Response SLA | Update SLA | Resolution SLA | Example |
|---|---|---|---|---|---|
| P1 | Critical | 15 min | Every 30 min | 2 hours | Prod server down, no access for all users |
| P2 | High | 30 min | Every 2 hrs | 8 hours | Core app degraded, partial functionality |
| P3 | Medium | 4 hrs | Daily | 3 business days | Single user issue, workaround available |
| P4 | Low | 1 business day | Weekly | 10 business days | Minor inconvenience, cosmetic issue |

### Incident Lifecycle

```
NEW → ASSIGNED → IN PROGRESS → PENDING → RESOLVED → CLOSED

State Definitions:
NEW         — Ticket created, not yet acknowledged
ASSIGNED    — Assigned to resolver group/individual
IN PROGRESS — Actively being worked
PENDING     — Waiting for: user info / vendor / change window / 3rd party
ON HOLD     — Intentionally paused (planned maintenance, user unavailable)
RESOLVED    — Fix applied, verification needed
CLOSED      — User confirmed resolution or auto-closed after N days
```

### P1 Incident Response Procedure

```
MINUTE 0-15: DETECTION & TRIAGE
  [ ] Alert fires or ticket created
  [ ] On-call engineer acknowledges within 5 minutes
  [ ] Confirm impact: How many users? Which systems?
  [ ] Create P1 ticket with "[P1]" in subject
  [ ] Notify: IT Manager, Service Desk lead, relevant resolver groups
  [ ] Open bridge call: Teams/Zoom bridge

MINUTE 15-30: MOBILIZATION
  [ ] Bridge call assembled: Incident Commander, Technical Leads, Comms
  [ ] Incident Commander role claimed
  [ ] Initial stakeholder notification sent (EMP-001 template)
  [ ] Assign roles: Technical, Comms, Scribe
  [ ] Identify working theory for root cause

MINUTE 30-120: INVESTIGATION & RESOLUTION
  [ ] Scribe logs all actions, timestamps, and findings in ticket
  [ ] Technical team works on resolution
  [ ] Status updates every 30 minutes to stakeholders
  [ ] Consider escalation if no progress in 60 minutes
  [ ] Apply fix or workaround

RESOLUTION:
  [ ] Verify fix in production
  [ ] Confirm with business stakeholders
  [ ] Send all-clear communication
  [ ] Close bridge call
  [ ] Update ticket with resolution details

POST-INCIDENT (within 5 business days):
  [ ] Post-incident review (PIR) meeting scheduled
  [ ] PIR report written (see template below)
  [ ] Action items tracked to completion
```

### P1 Post-Incident Review Template

```
POST-INCIDENT REVIEW (PIR)
Incident ID: INC-XXXXX
Incident Date: [Date/Time]
PIR Date: [Date, within 5 business days]
Participants: [List]
Author: [Name]

INCIDENT SUMMARY
Duration: [HH:MM]
Systems Affected: [List]
Users Impacted: ~[N] users / [all/dept/subset]
Business Impact: [Description of impact]
SLA Met: Yes / No (Target: 2hr, Actual: Xhr)

TIMELINE OF EVENTS
[Time] — [Event]
[Time] — [Event]
[Time] — [Resolution applied]
[Time] — [Service restored]

ROOT CAUSE ANALYSIS
Primary Cause: [Single sentence describing the root cause]
Contributing Factors:
1. [Factor 1]
2. [Factor 2]

5 WHYS:
Why 1: [Why did the incident occur?]
Why 2: [Why did that happen?]
Why 3: [Why did that happen?]
Why 4: [Why did that happen?]
Why 5: [Root cause]

WHAT WENT WELL
1. [Detection was fast — alert fired within 2 minutes]
2. [Bridge call assembled quickly]

WHAT COULD BE IMPROVED
1. [Runbook was outdated — took 45 min to find correct procedure]
2. [Escalation criteria unclear]

ACTION ITEMS
| # | Action | Owner | Due Date | Priority |
|---|--------|-------|----------|----------|
| 1 | Update runbook for XYZ system | [Name] | [Date] | High |
| 2 | Add monitoring for [gap] | [Name] | [Date] | High |
| 3 | Conduct team training on bridge call protocol | [Name] | [Date] | Med |
```

---

## 3. Problem Management

### Problem vs. Incident

```
Incident: A disruption to service — restore ASAP
Problem: The underlying cause of incidents — find root cause and fix permanently

Example:
  Incident: Server ran out of disk space → cleared logs → service restored
  Problem:  Why does the log volume grow unchecked? Fix: log rotation policy

Problem Types:
  Reactive:  Raised after recurring incidents or major incident
  Proactive: Raised from trend analysis before incidents occur
```

### Problem Management Process

```
1. IDENTIFY
   - Recurring incidents (same CI, 3+ times in 30 days)
   - Major incidents (P1/P2) always generate a problem
   - Trend analysis flags
   - Risk assessment finds potential issues

2. LOG & CATEGORIZE
   - Create Problem record linked to incidents
   - Assign problem owner
   - Set priority using impact/urgency
   - Document known symptoms

3. INVESTIGATE
   - Root cause analysis (RCA)
   - Techniques: 5 Whys, Fishbone, Fault Tree
   - Reproduce in test environment if possible
   - Document workarounds (Known Error)

4. KNOWN ERROR
   - When root cause known but permanent fix not yet applied
   - Document in Known Error Database (KEDB)
   - Link to KB article for service desk
   - This helps resolve future incidents faster

5. RESOLVE
   - Permanent fix identified
   - RFC (Request for Change) raised if change needed
   - Change implemented
   - Verify incidents no longer recur

6. CLOSE
   - Confirm fix effective
   - Update CMDB
   - Close linked incidents
   - Update KB article
```

### RCA Techniques

```
FISHBONE (Ishikawa) Diagram — Cause and Effect

       Equipment    People        Process
          |            |             |
          └────────────┴─────────────┴──────► EFFECT (Problem)
          ┌────────────┬─────────────┬──────►
          |            |             |
       Materials   Environment    Methods

Example for "Database performance degraded":
- Equipment: Insufficient RAM, disk I/O bottleneck
- People: DBA changed query plan
- Process: No change freeze for month-end
- Materials: Statistics not updated
- Environment: Temp spike caused thermal throttling
- Methods: Missing indexes on new table

FAULT TREE ANALYSIS (FTA)
Top Event: Service Unavailable
├── OR Gate
│   ├── Network failure
│   │   ├── ISP outage
│   │   └── Switch failure
│   ├── Application failure
│   │   ├── Memory exhaustion
│   │   └── Code bug
│   └── Infrastructure failure
│       ├── Server hardware
│       └── Storage unavailable
```

---

## 4. Change Enablement

### Change Types

| Type | Definition | Approval | Lead Time |
|---|---|---|---|
| Standard | Pre-approved, low risk, routine | Pre-approved | 0 |
| Normal Low | Low risk, assessed individually | Manager | 5 days |
| Normal High | Higher risk, significant impact | CAB | 10 days |
| Emergency | Urgent, unplanned, in response to incident | ECAB | Same day |

### Change Advisory Board (CAB)

```
CAB Composition:
- IT Manager (Chair)
- Change Manager
- Infrastructure Lead
- Security Representative
- Application Owners (rotating)
- Business Stakeholders (for significant changes)

CAB Meeting: Weekly (Wednesdays, 30 min)
Emergency CAB (ECAB): Ad hoc, can be virtual, quorum = 3 members

CAB Reviews:
1. Proposed changes for next week
2. Changes in progress
3. Post-implementation reviews
4. Failed changes from last week
```

### Change Request Template

```
CHANGE REQUEST: CHG-XXXXX
Title: [Descriptive title]
Date Submitted: [Date]
Requested By: [Name]
Assigned To: [Name]
Target Implementation: [Date/Time]
Change Type: Normal Low / Normal High / Emergency

CHANGE DESCRIPTION
Summary: [What is being changed?]
Systems Affected: [List of CIs]
Business Justification: [Why is this needed?]

RISK ASSESSMENT
Risk Level: Low / Medium / High
Risk Description: [What could go wrong?]
Impact if Fails: [Business impact]
Probability of Failure: Low / Medium / High

IMPLEMENTATION PLAN
Duration: [Estimated downtime / change window]
Window: [Start time - End time]
Implementation Steps:
1. [Step 1]
2. [Step 2]
3. [Step N]

ROLLBACK PLAN
Rollback Trigger: [When to rollback]
Rollback Steps:
1. [Step 1]
2. [Step 2]
Rollback Time Estimate: [HH:MM]

TEST PLAN
1. [Test 1: Expected result]
2. [Test 2: Expected result]

APPROVAL
□ Change Manager: _____________ Date: _____
□ Technical Reviewer: _________ Date: _____
□ CAB Chair: _________________ Date: _____
```

### Change Freeze Periods

```
Standard Change Freeze (announce 4 weeks prior):
- Major holidays (Christmas, Thanksgiving week)
- Quarter/year-end financial close (typically 2 weeks)
- Major product launches
- Annual contract renewal periods

Emergency changes ALWAYS require:
1. ECAB approval (minimum 3 approvers)
2. Management sign-off
3. Rollback plan reviewed before implementation
4. PIR within 48 hours
```

---

## 5. Service Request Management

### Request Catalog Design

```
Common Service Requests by Category:

ACCESS MANAGEMENT
  - New user account creation
  - Application access request
  - VPN access request
  - Elevated/admin access request (with justification)
  - Guest/contractor account

HARDWARE
  - Laptop/desktop request
  - Mobile device request
  - Hardware replacement
  - Peripheral (monitor, keyboard, etc.)
  - Lab/lab equipment reservation

SOFTWARE
  - Software installation request
  - License request
  - Software upgrade
  - Software removal

INFRASTRUCTURE
  - New server/VM provisioning
  - DNS record creation/modification
  - Firewall rule request
  - SSL certificate request
  - Storage allocation

COLLABORATION
  - Distribution group creation
  - Shared mailbox creation
  - Teams channel creation
  - SharePoint site provisioning
```

### Request Fulfillment SLAs

| Request Type | SLA | Auto-Approve? |
|---|---|---|
| Password reset | 15 min (self-service preferred) | Yes |
| Account unlock | 30 min | Yes (with MFA) |
| Standard software install | 4 hours | Yes (whitelist) |
| New user onboarding | 1 business day | No (manager approval) |
| Application access | 4 hours | Yes (pre-approved) |
| Admin access request | 2 business days | No (CISO/IT Dir) |
| New VM provisioning | 2 business days | No (CAB) |
| Firewall rule | 5 business days | No (Security) |

---

## 6. Configuration Management (CMDB)

### CMDB Architecture

```
Configuration Item (CI) Classes:
├── Hardware
│   ├── Server (physical)
│   ├── Workstation
│   ├── Laptop
│   ├── Network Device (switch, router, firewall)
│   ├── Storage Device
│   └── Printer/Peripheral
├── Virtual Infrastructure
│   ├── Virtual Machine
│   ├── Container
│   └── Cloud Resource
├── Software
│   ├── Application
│   ├── Database
│   ├── Operating System
│   └── Software License
├── Services
│   ├── Business Service (customer-facing)
│   └── Technical Service (infrastructure)
└── Network
    ├── IP Address
    ├── VLAN
    └── DNS Record
```

### CI Attributes Standard

```
Mandatory CI Attributes (all CIs):
- CI ID (auto-generated)
- CI Name
- CI Class
- Status: In Use / In Storage / Retired / Maintenance
- Environment: Production / Staging / Development / DR
- Owner (team)
- Business Owner (for apps/services)
- Created Date
- Last Modified Date

Hardware-Specific:
- Make/Model
- Serial Number
- Asset Tag
- Purchase Date
- Warranty Expiry
- Location (datacenter, rack, row, room)

Application-Specific:
- Version
- Installation Path
- Vendor
- License Type
- Dependent CIs (relationships)
- SLA Tier

Relationships (critical for impact analysis):
- "Runs On" — App → Server
- "Depends On" — App → Database
- "Connected To" — Server → Switch
- "Part Of" — VM → Physical Host
- "Supports" — Service → Business Function
```

### CMDB Hygiene

```
Governance Rules:
1. No CI may be deployed to production without being in CMDB
2. CMDB updated as part of change process (not after)
3. Monthly automated reconciliation with discovery tools
4. Retired CIs marked "Retired" — never deleted
5. Orphaned CIs (no owner) reviewed quarterly

Reconciliation Process (Monthly):
  1. Run discovery scan (SCCM, Lansweeper, Nessus)
  2. Compare to CMDB
  3. Flag: In discovery but NOT in CMDB → add with owner
  4. Flag: In CMDB but NOT in discovery → investigate/retire
  5. Flag: Attribute mismatch (hostname, IP) → update CMDB
  6. Report: CMDB accuracy score = CIs correct / total CIs × 100
  7. Target: >95% accuracy
```

---

## 7. Knowledge Management

### Knowledge Article Types

| Type | Purpose | Audience | Review Frequency |
|---|---|---|---|
| How-To | Step-by-step instructions | End users | Annual |
| Troubleshooting Guide | Diagnose and fix specific issues | Technicians | After each related incident |
| Known Error | Workaround for known issue | Service Desk | When issue resolved |
| FAQ | Common questions | End users | Semi-annual |
| Runbook | Detailed operational procedure | IT Staff | After each related change |
| Policy | IT rules and requirements | All staff | Annual or when changed |

### Knowledge Article Template

```
KNOWLEDGE ARTICLE
ID: KB-XXXXX
Title: [Clear, searchable title]
Category: [Category] / [Subcategory]
Applies To: [Systems/apps/users]
Last Reviewed: [Date]
Reviewed By: [Name]
Article Type: How-To / Troubleshooting / Known Error

SUMMARY
[1-2 sentence description of what this article covers]

SYMPTOMS / WHEN TO USE THIS ARTICLE
- [Symptom 1 — exact error message if applicable]
- [Symptom 2]

CAUSE (for troubleshooting articles)
[Root cause of the issue]

SOLUTION / PROCEDURE
Prerequisites:
- [What access/tools are needed?]

Steps:
1. [Step 1 — clear, specific action]
   Expected result: [What should happen]

2. [Step 2]
   Expected result: [What should happen]

3. [Step N]

Verification:
[How to confirm the issue is resolved]

RELATED ARTICLES
- [KB-XXXXX: Related article title]

FEEDBACK
Was this article helpful? [Yes/No/Partially]
If no, what was missing? [Free text]
```

---

## 8. SLA Management

### SLA Framework

```
SLA Hierarchy:
OLA (Operational Level Agreement) — Internal team commitments
  ↓
UC  (Underpinning Contract) — External vendor commitments
  ↓
SLA (Service Level Agreement) — IT commits to business/customer

Example:
Business requires: Web app available 99.9% monthly (SLA)
IT Operations commits: Infrastructure team patches during approved windows (OLA)
AWS commits: EC2 99.99% uptime (UC)
```

### Availability Calculation

```
Availability % = (Agreed Service Time - Downtime) / Agreed Service Time × 100

Agreed Service Time (AST): Excludes planned maintenance windows
Downtime: Unplanned outages only

Example:
AST: 24×7 = 730 hours/month
Unplanned downtime: 45 minutes
Availability = (730 - 0.75) / 730 × 100 = 99.90%

Availability Targets:
99.999% ("Five 9s") = 5.26 min/year downtime
99.99%               = 52.6 min/year
99.9%                = 8.7 hours/year
99.5%                = 43.8 hours/year
99.0%                = 87.6 hours/year
```

### SLA Review Process

```
Monthly SLA Report Contents:
1. Overall SLA compliance rate
2. Incident metrics:
   - Total incidents by priority
   - SLA met/breached by priority
   - MTTR by priority
   - Top 5 repeat offenders
3. Change metrics:
   - Total changes
   - Failed changes
   - Emergency changes
4. Request metrics:
   - Total requests
   - SLA met/breached
   - Backlog age
5. Customer satisfaction score (CSAT)
6. Top 5 improvement actions
```

---

## 9. Continual Improvement

### Continual Improvement Register

| ID | Initiative | Objective | Baseline | Target | Current | Status | Owner |
|---|---|---|---|---|---|---|---|
| CI-001 | Implement self-service portal | Reduce SR volume | 80% manual | 50% manual | 65% manual | In Progress | Help Desk Mgr |
| CI-002 | Improve P1 MTTR | Faster resolution | 4 hrs avg | 2 hrs avg | 2.5 hrs avg | In Progress | IT Manager |
| CI-003 | CMDB accuracy | Better impact analysis | 78% accurate | 95% accurate | 91% accurate | In Progress | ITSM Lead |
| CI-004 | Knowledge article coverage | Reduce repeat contacts | 45% KB use | 65% KB use | 52% KB use | Planning | Help Desk Mgr |

### CSI (Continual Service Improvement) Approach

```
1. What is the vision?
   → Business goal / IT strategy alignment

2. Where are we now?
   → Baseline measurement (current state)

3. Where do we want to be?
   → Target/goal definition (SMART)

4. How do we get there?
   → Improvement plan / roadmap

5. Take action
   → Implement improvements

6. Did we get there?
   → Measure improvement

7. How do we keep the momentum going?
   → Embed in operations, raise next improvement
```

---

## 10. ITSM Tooling Guide

### Tool Comparison

| Feature | ServiceNow | Jira Service Mgmt | Freshservice | Zendesk |
|---|---|---|---|---|
| Incident Mgmt | ✅ Excellent | ✅ Good | ✅ Good | ✅ Good |
| Problem Mgmt | ✅ Excellent | ✅ Good | ✅ Good | ⚠️ Limited |
| Change Mgmt | ✅ Excellent | ✅ Good | ✅ Good | ❌ Poor |
| CMDB | ✅ Excellent | ⚠️ Basic | ⚠️ Basic | ❌ None |
| SLA Mgmt | ✅ Excellent | ✅ Good | ✅ Good | ✅ Good |
| KB Mgmt | ✅ Excellent | ✅ Good | ✅ Good | ✅ Good |
| Self-Service Portal | ✅ Excellent | ✅ Good | ✅ Good | ✅ Good |
| Cost | $$$$$ | $$$ | $$ | $$$ |
| Best For | Enterprise | Dev+IT integration | SMB/Mid-market | Customer support |

### ServiceNow Key Configuration

```javascript
// Business Rule: Auto-assign based on CI
// Table: incident
// When: before insert and update

(function executeRule(current, previous) {
    var ci = current.cmdb_ci;
    if (ci) {
        var grGR = new GlideRecord('cmdb_ci');
        grGR.get(ci);
        var supportGroup = grGR.support_group;
        if (supportGroup) {
            current.assignment_group = supportGroup;
        }
    }
})(current, previous);
```

---

## 11. ITSM Metrics & Reporting

### Key ITSM KPIs

```
Service Desk Metrics:
├── First Contact Resolution (FCR) — Target: >70%
├── Average Speed to Answer (ASA) — Target: <30 sec
├── Abandoned Call Rate — Target: <5%
├── Customer Satisfaction (CSAT) — Target: >4.0/5.0
├── Ticket Backlog — Target: <3× daily volume
└── Cost per Ticket — Benchmark: $15-$50

Incident Metrics:
├── P1 MTTR — Target: <2 hours
├── P2 MTTR — Target: <8 hours
├── SLA Compliance Rate — Target: >95%
├── Repeat Incidents (same CI, 30 days) — Target: <5%
└── Major Incident Frequency — Target: decreasing trend

Change Metrics:
├── Change Success Rate — Target: >95%
├── Emergency Change Rate — Target: <10% of all changes
├── Unauthorized Change Rate — Target: 0%
└── Change Lead Time — Track by type

Problem Metrics:
├── Problems Opened/Closed — Target: closed > opened
├── Known Errors with Workaround — Target: 100% have workaround
├── Age of Open Problems — Target: no problems >90 days
└── Problem-to-Incident Reduction — Target: >20% YoY reduction
```

### Monthly ITSM Dashboard Template

```
═══════════════════════════════════════
ITSM MONTHLY REPORT — [Month Year]
═══════════════════════════════════════

SERVICE DESK PERFORMANCE
  Total Tickets:              850
  Incidents:                  620  (73%)
  Service Requests:           230  (27%)
  FCR Rate:                   74%  ✅ Target: >70%
  CSAT Score:                 4.2  ✅ Target: >4.0
  Tickets per Agent/Day:       18

INCIDENT MANAGEMENT
  P1 Incidents:                 2  (MTTR: 1h 45m ✅)
  P2 Incidents:                 8  (MTTR: 5h 20m ✅)
  P3 Incidents:               180  (MTTR: 1.5 days ✅)
  Overall SLA Compliance:     96%  ✅ Target: >95%
  Repeat Incidents:             12  ⚠️ Watch trend

CHANGE MANAGEMENT
  Total Changes:               42
  Standard:                    28  (67%)
  Normal:                      11  (26%)
  Emergency:                    3  (7%) ✅ Target: <10%
  Failed Changes:               1  (2%) ✅ Target: <5%

TOP 5 RECURRING ISSUES
  1. VPN disconnects (23 incidents) → Problem #P-0045
  2. Outlook slowness (18 incidents) → Known error KE-0012
  3. Printer offline (15 incidents) → Being addressed by CI-005
  4. MFA issues (12 incidents) → KB-00234 published
  5. SharePoint access errors (8 incidents) → Investigating

IMPROVEMENT HIGHLIGHTS
  ✅ Self-service password reset deployed → 15% ticket reduction
  ✅ Updated 12 KB articles (stale articles review)
  ⚠️ CMDB accuracy at 89% (target 95%) — reconciliation in progress

NEXT MONTH FOCUS
  1. Complete CMDB reconciliation
  2. Implement VPN stability fix (Problem P-0045)
  3. Launch IT satisfaction survey
═══════════════════════════════════════
```

---

*For day-to-day helpdesk operations, see the Helpdesk Operations Guide. For change implementation procedures, see the Standard Operating Procedures guide.*
