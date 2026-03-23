# IT Project Management Guide

## Overview
Comprehensive guide for IT project managers and team leads covering project lifecycle, methodologies, stakeholder management, risk, budgeting, and delivery. Applicable to infrastructure, application, and digital transformation projects.

---

## 1. Project Management Fundamentals

### The IT Project Lifecycle
```
Initiation → Planning → Execution → Monitoring & Control → Closing
     ↑                                       |
     └───────────────────────────────────────┘
                (change requests)
```

### Initiation Phase

**Project Charter Components:**
1. Project purpose and justification
2. High-level scope
3. High-level timeline (milestones only)
4. Estimated budget range
5. Identified stakeholders
6. Project sponsor
7. Project manager assignment
8. Success criteria

**Project Charter Template:**
```
PROJECT CHARTER
================
Project Name: [Name]
Project ID: [PRJ-YYYY-NNN]
Date: [Date]
Version: 1.0

BUSINESS CASE
What problem are we solving?
What is the cost/risk of NOT doing this?
What value does success deliver?

OBJECTIVES (SMART)
1. [Specific, Measurable, Achievable, Relevant, Time-bound]
2. ...

SCOPE
In Scope:
- [Deliverable 1]
- [Deliverable 2]

Out of Scope:
- [Explicitly excluded item 1]

CONSTRAINTS
- Budget: $[amount]
- Timeline: [start] - [end]
- Resources: [team size / named resources]

ASSUMPTIONS
- [assumption 1]
- [assumption 2]

RISKS (Initial)
- [Risk 1]: [Probability] / [Impact]

STAKEHOLDERS
Sponsor: [Name, Title]
PM: [Name]
Key Stakeholders: [list]

APPROVAL
Sponsor: _____________ Date: _______
PM: __________________ Date: _______
```

---

## 2. Project Planning

### Work Breakdown Structure (WBS)

**Example: Server Migration Project WBS**
```
1.0 Server Migration Project
├── 1.1 Project Management
│   ├── 1.1.1 Project planning
│   ├── 1.1.2 Status reporting
│   └── 1.1.3 Change management
├── 1.2 Assessment & Design
│   ├── 1.2.1 Current state inventory
│   ├── 1.2.2 Dependency mapping
│   ├── 1.2.3 Target architecture design
│   └── 1.2.4 Design review/approval
├── 1.3 Environment Preparation
│   ├── 1.3.1 Procure hardware/licenses
│   ├── 1.3.2 Network configuration
│   ├── 1.3.3 Storage provisioning
│   └── 1.3.4 OS deployment
├── 1.4 Migration Execution
│   ├── 1.4.1 Test migrations
│   ├── 1.4.2 Wave 1 migration (non-critical)
│   ├── 1.4.3 Wave 2 migration (standard)
│   └── 1.4.4 Wave 3 migration (critical)
├── 1.5 Testing & Validation
│   ├── 1.5.1 UAT per application
│   ├── 1.5.2 Performance testing
│   └── 1.5.3 DR testing
└── 1.6 Decommission & Close
    ├── 1.6.1 Old server decommission
    ├── 1.6.2 Documentation
    └── 1.6.3 Project close-out
```

### Estimating Techniques

**Three-Point Estimation (PERT):**
```
Expected = (Optimistic + 4×Most Likely + Pessimistic) / 6
Standard Deviation = (Pessimistic - Optimistic) / 6

Example:
Task: Migrate database server
Optimistic: 4 hours
Most Likely: 8 hours
Pessimistic: 20 hours

Expected = (4 + 32 + 20) / 6 = 9.3 hours
Std Dev = (20 - 4) / 6 = 2.7 hours
95% confidence range: 9.3 ± (2 × 2.7) = 3.9 to 14.7 hours
```

**Story Points (Agile):**
- Use Fibonacci sequence: 1, 2, 3, 5, 8, 13, 21
- Relative sizing, not time-based
- Team velocity measured in sprint 1-3, then forecasting improves
- Compare to reference story: "Installing a standard laptop = 3 points"

### Resource Planning
```
Resource Plan Template:
Role          | Person         | % Alloc | Sprint 1 | Sprint 2 | Sprint 3
Infrastructure Lead | J. Smith | 75%     | Design   | Build    | Test
Network Eng   | A. Jones       | 50%     | Design   | Build    | Validate
Security      | T. Kim         | 25%     | Review   | Review   | Sign-off
PM            | L. Davis       | 50%     | Plan     | Track    | Report
```

---

## 3. Methodologies

### ITIL-Aligned IT Projects
For infrastructure and service changes:
- Align project changes with Change Management process
- Submit RFCs at key milestones
- Emergency change process for critical patches
- Project close triggers service transition activities

### Agile for IT Projects

**Sprint Structure (2 weeks):**
```
Day 1:    Sprint Planning (2-4 hours)
         - Review backlog, select stories
         - Break into tasks, estimate hours
         - Define sprint goal

Days 2-9: Development/Execution
         - Daily standup (15 min)
           • What did I do yesterday?
           • What will I do today?
           • Any blockers?

Day 10:   Sprint Review (1-2 hours)
         - Demo completed work
         - Stakeholder feedback
         - Update backlog

Day 10:   Sprint Retrospective (1 hour)
         - What went well?
         - What could improve?
         - Action items for next sprint
```

**Backlog Grooming:**
- Happens mid-sprint for next sprint
- PM/BA clarifies requirements
- Team estimates stories
- Priority set by Product Owner/stakeholder

**Definition of Done (DoD):**
- [ ] Code/configuration reviewed by peer
- [ ] Testing completed (functional + regression)
- [ ] Documentation updated
- [ ] Security review completed (if applicable)
- [ ] Change request approved and implemented
- [ ] Acceptance criteria met
- [ ] Demo-able to stakeholder

### Waterfall for IT Projects
Use when:
- Requirements are fixed and well-understood
- Regulatory compliance required
- Hardware procurement involved
- Large, complex system integrations

**Phase Gates:**
Each phase requires formal approval before proceeding:
- Initiation Gate: Charter approved, funding secured
- Design Gate: Architecture approved, security reviewed
- Build Gate: Infrastructure ready, code in UAT
- Test Gate: Testing complete, UAT sign-off
- Deploy Gate: Change window approved, rollback plan ready
- Close Gate: Post-implementation review, lessons learned

---

## 4. Risk Management

### Risk Register

**Risk Matrix:**
```
         IMPACT
         Low  | Med  | High | Critical
      ----+------+------+---------
H    High |  M  |  H   |  C   |  C
I    -----+------+------+------+------
G    Med  |  L  |  M   |  H   |  C
H    -----+------+------+------+------
T    Low  |  L  |  L   |  M   |  H
     -----+------+------+------+------

C = Critical (Unacceptable - must mitigate)
H = High (Requires mitigation plan)
M = Medium (Mitigation recommended)
L = Low (Monitor)
```

**Risk Register Template:**

| ID | Risk Description | Probability | Impact | Score | Owner | Mitigation | Contingency | Status |
|----|-----------------|-------------|--------|-------|-------|------------|-------------|--------|
| R01 | Vendor delivers hardware late | Medium | High | H | PM | Order 8 weeks early, dual vendor | Use cloud temporary environment | Open |
| R02 | Security vulnerabilities found in pen test | Low | Critical | C | CISO | Security review in design phase | Delay launch, emergency patching | Open |
| R03 | Key team member leaves | Low | High | H | PM | Cross-train, document | Engage contractor | Open |

**Risk Response Strategies:**
- **Avoid:** Change plan to eliminate risk
- **Transfer:** Insurance, contracts, outsource
- **Mitigate:** Reduce probability or impact
- **Accept:** Monitor, accept residual risk (for low risks)

---

## 5. Stakeholder Management

### Stakeholder Analysis Matrix

**Power/Interest Grid:**
```
HIGH  |  Keep Satisfied    |  Manage Closely  |
POWER |  (minimal effort)  |  (high effort)   |
      |---------------------|------------------|
LOW   |  Monitor            |  Keep Informed   |
POWER |  (minimal effort)  |  (regular comms) |
      |---------------------|------------------|
             LOW INTEREST        HIGH INTEREST
```

**RACI Matrix:**
| Task | PM | Dev Lead | Infra | Security | Exec Sponsor |
|------|----|----------|-------|----------|-------------|
| Architecture design | A | R | C | C | I |
| Security review | I | I | I | R | A |
| Change approval | R | I | I | C | A |
| User acceptance | A | R | C | I | I |
| Go-live decision | I | I | I | I | A/R |

R=Responsible, A=Accountable, C=Consulted, I=Informed

### Communication Plan

| Stakeholder | Info Needed | Frequency | Method | Owner |
|-------------|-------------|-----------|--------|-------|
| Executive Sponsor | Status, risks, budget | Bi-weekly | Executive summary email | PM |
| Steering Committee | Milestone status, decisions needed | Monthly | Steering deck | PM |
| Project Team | Sprint goals, blockers | Daily | Standup | PM |
| End Users | Change impacts, training | As needed | Email, all-hands | Change Mgr |
| IT Operations | Deployment details, runbooks | Weekly | Meeting | PM |
| Vendors | Deliverables, timelines | Weekly | Email/call | PM |

---

## 6. IT Project Budgeting

### Budget Components
```
CAPEX (Capital Expenditure):
  Hardware:           $___
  Software licenses:  $___
  Consulting:         $___
  Internal labor:     $___
  CAPEX Subtotal:     $___

OPEX (Operational Expenditure - ongoing):
  Annual SaaS/cloud:  $___
  Support contracts:  $___
  Maintenance:        $___
  Training:           $___
  OPEX Annual:        $___

CONTINGENCY RESERVE (typically 10-20%):
  Reserve:            $___

MANAGEMENT RESERVE (unknown unknowns, 5-10%):
  Reserve:            $___

TOTAL PROJECT BUDGET: $___
```

### Tracking Budget vs. Actuals
```
Budget Tracking Report — Month of [Date]

Category          | Budget  | Spent  | Committed | Remaining | % Spent
------------------|---------|--------|-----------|-----------|--------
Hardware          | $50,000 | $47,500| $0        | $2,500    | 95%
Software          | $20,000 | $18,000| $2,000    | $0        | 100%
Consulting        | $30,000 | $12,000| $8,000    | $10,000   | 67%
Internal Labor    | $25,000 | $20,000| $3,000    | $2,000    | 92%
Contingency       | $12,500 | $0     | $0        | $12,500   | 0%
---------         |---------|--------|-----------|-----------|
TOTAL             |$137,500 | $97,500| $13,000   | $27,000   | 81%

Budget Status: ON TRACK ✓
EAC (Estimate at Completion): $135,000
Variance: -$2,500 (under budget)
```

### Earned Value Management (EVM)
```
PV  = Planned Value (budgeted work scheduled by now)
EV  = Earned Value (budgeted cost of work actually done)
AC  = Actual Cost (what we actually spent)

SV  = EV - PV       (Schedule Variance; negative = behind)
CV  = EV - AC       (Cost Variance; negative = over budget)
SPI = EV / PV       (Schedule Performance Index; <1 = behind)
CPI = EV / AC       (Cost Performance Index; <1 = over budget)
EAC = BAC / CPI     (Estimate at Completion)
VAC = BAC - EAC     (Variance at Completion)
```

---

## 7. Change Management (Organizational)

### ADKAR Model
| Element | Description | IT Context |
|---------|-------------|-----------|
| **A**wareness | Why the change is needed | Communicate business problem being solved |
| **D**esire | Willingness to support | Show "what's in it for me" |
| **K**nowledge | How to change | Training, documentation, job aids |
| **A**bility | Demonstrate new skills | Hands-on practice, helpdesk support |
| **R**einforcement | Sustain the change | Follow-up, remove old way |

### Change Resistance Tactics
- Involve affected users early (not just inform)
- Identify champions in each department
- Address concerns directly, don't dismiss
- Provide training before go-live, not after
- Ensure leadership visibly supports the change
- Quick wins early in the project

---

## 8. IT Project Governance

### Project Status Report Template
```
PROJECT STATUS REPORT
Project: [Name]          Period: [Date Range]
PM: [Name]               Report #: [N]

OVERALL STATUS: [GREEN | YELLOW | RED]

SCHEDULE STATUS: [GREEN/YELLOW/RED]
  Current milestone: [Milestone name]
  On track for: [Completion date]
  Variance: [+/- N days]

BUDGET STATUS: [GREEN/YELLOW/RED]
  Budget: $[amount]
  Spent to date: $[amount] ([N]%)
  Projected final: $[amount]

RISKS:
  New this period: [List]
  Top active risks: [List top 3]

ISSUES:
  New this period: [List]
  Open issues: [List with owners]

DECISIONS NEEDED:
  1. [Decision needed, from whom, by when]

ACCOMPLISHMENTS (this period):
  ✓ [Completed item 1]
  ✓ [Completed item 2]

PLANNED NEXT PERIOD:
  → [Upcoming task 1]
  → [Upcoming task 2]
```

### Steering Committee Deck Structure
1. Executive summary (one slide, RAG status)
2. Milestone timeline (Gantt overview)
3. Budget summary
4. Key risks and issues (top 3-5 only)
5. Decisions required
6. Next period focus

---

## 9. Vendor Management in Projects

### RFP / RFI Process
```
1. Define requirements (functional + non-functional + security)
2. Issue RFI to market (broad, no commitment)
3. Shortlist vendors (3-5)
4. Issue RFP (detailed, with evaluation criteria)
5. Vendor demos / PoC (optional)
6. Score and select vendor
7. Contract negotiation
8. Contract execution
```

**RFP Evaluation Criteria (Example):**
| Criterion | Weight |
|-----------|--------|
| Technical fit to requirements | 30% |
| Security and compliance | 25% |
| Total Cost of Ownership (5yr) | 20% |
| Implementation support | 10% |
| Vendor stability and roadmap | 10% |
| References and track record | 5% |

### Statement of Work (SOW) Essentials
- Scope of work (detailed deliverables)
- Acceptance criteria per deliverable
- Timeline and milestones
- Payment schedule (milestone-based, not time-based)
- IP ownership
- Confidentiality requirements
- Change order process
- Warranty/support period post-delivery
- Termination clauses

---

## 10. Project Close-Out

### Closure Checklist
- [ ] All deliverables accepted by stakeholders
- [ ] All open issues resolved or formally accepted
- [ ] Final budget reconciliation complete
- [ ] All contracts closed out
- [ ] Documentation complete and stored
- [ ] Knowledge transfer to operations complete
- [ ] Lessons learned session conducted
- [ ] Team appreciation/recognition done
- [ ] Project formally closed in ITSM system
- [ ] Post-implementation review scheduled (30 days)

### Lessons Learned Template
```
LESSONS LEARNED REPORT
Project: [Name]   Date: [Date]   PM: [Name]

WHAT WENT WELL:
1. [Example: Autopilot deployment reduced deployment time by 60%]
2. ...

WHAT COULD IMPROVE:
1. [Example: Vendor selection process started too late, caused 3-week delay]
2. ...

ROOT CAUSES:
[For each improvement area: why did it happen?]

RECOMMENDATIONS FOR FUTURE PROJECTS:
1. [Actionable recommendation]
2. ...

METRICS:
Schedule: Delivered [on time | N days late | N days early]
Budget: [Under | Over | On] budget by $[N] ([N]%)
Quality: [N] defects post-launch vs [N] target
Satisfaction: User satisfaction score: [N]/10
```

---

*Last Updated: 2025 | IT Operations Documentation Library*
