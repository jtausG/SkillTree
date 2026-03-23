# IT Project Management Guide

## Table of Contents
1. [IT PM Fundamentals](#fundamentals)
2. [Project Lifecycle](#lifecycle)
3. [Scoping & Charter](#scoping)
4. [Stakeholder Management](#stakeholders)
5. [Planning & Scheduling](#planning)
6. [Risk Management](#risk)
7. [Change Management (Organizational)](#change)
8. [Agile for IT Projects](#agile)
9. [Infrastructure Project Specifics](#infrastructure)
10. [Vendor & Procurement Management](#vendor)
11. [Project Communication](#communication)
12. [Project Closure & Lessons Learned](#closure)
13. [Templates & Tools](#templates)

---

## 1. IT PM Fundamentals <a name="fundamentals"></a>

### The Iron Triangle
```
        Scope
       /     \
      /       \
   Time ——— Cost

Constraints:
- Fixed scope + Fixed time = Variable cost
- Fixed scope + Fixed cost = Variable time
- Fixed time + Fixed cost = Variable scope

IT PM reality: All three are usually "fixed" by stakeholders.
Your job: Manage expectations and escalate trade-off decisions early.
```

### Project vs. Operation vs. Program
| Type | Definition | IT Example |
|------|-----------|------------|
| Project | Temporary, unique, defined end | Migrate to Azure, deploy new ERP |
| Operation | Ongoing, repetitive | Running helpdesk, maintaining servers |
| Program | Group of related projects | Digital transformation initiative |
| Portfolio | Collection of programs/projects | IT strategic portfolio |

### PMBOK Knowledge Areas (Applied to IT)
1. **Integration** — Project charter, change control
2. **Scope** — Requirements, WBS, scope control
3. **Schedule** — Gantt chart, critical path, milestones
4. **Cost** — Budget, EVM, forecasting
5. **Quality** — Acceptance criteria, testing, QA
6. **Resource** — Team assignments, RACI, vendor management
7. **Communications** — Status reports, meetings, escalation
8. **Risk** — Risk register, mitigation, contingency
9. **Procurement** — RFP, SOW, contracts
10. **Stakeholder** — Stakeholder analysis, engagement

---

## 2. Project Lifecycle <a name="lifecycle"></a>

### Phase 1: Initiation
```
Activities:
- Business case creation / sponsor approval
- Project charter sign-off
- Initial stakeholder identification
- Preliminary scope and budget estimate (+/- 50% accuracy)
- Go/no-go decision

Deliverables:
- Business case document
- Signed project charter
- Stakeholder register (initial)
- Preliminary budget estimate

Exit criteria:
- Executive sponsor identified and committed
- Funding approved or in budget cycle
- PM assigned
```

### Phase 2: Planning
```
Activities:
- Requirements gathering (functional + technical + security)
- WBS creation
- Detailed schedule with milestones
- Resource plan and assignments
- Risk register
- Communication plan
- Procurement plan (if vendors involved)
- Testing strategy

Deliverables:
- Project Management Plan
- Work Breakdown Structure (WBS)
- Project schedule (Gantt)
- RACI matrix
- Risk register
- Communication plan
- Technical design document

Exit criteria:
- All deliverables approved by sponsor
- Team resources confirmed and available
- Vendor contracts signed (if applicable)
- Kickoff meeting scheduled
```

### Phase 3: Execution
```
Activities:
- Project kickoff meeting
- Daily/weekly execution
- Regular status meetings
- Issue and risk tracking
- Change request management
- Stakeholder updates
- Vendor management

Key monitoring:
- % complete vs. schedule (are we on track?)
- Budget burn rate (are we within budget?)
- Open issues and risks (any blockers?)
- Scope creep (are we doing things not in scope?)

Cadence:
- Daily: Team standup (15 min) during active phases
- Weekly: Status report + team meeting
- Bi-weekly: Steering committee update
- Monthly: Executive sponsor briefing
```

### Phase 4: Testing & UAT
```
Testing types:
- Unit testing: Individual component works correctly
- Integration testing: Components work together
- System testing: Full system performs as expected
- UAT: Business users validate
- Performance testing: System handles expected load
- Security testing: Pen test, vulnerability scan
- DR test: Failover works as documented

UAT process:
1. Create UAT test scripts (based on requirements)
2. Identify UAT testers (business users, not IT)
3. Set up UAT environment (prod-like)
4. Execute UAT (typically 2-4 weeks)
5. Log and triage defects
6. Regression test after fixes
7. UAT sign-off from business owner

Defect severity:
- Critical: Blocks UAT, must fix before go-live
- High: Major functionality broken, must fix before go-live
- Medium: Workaround exists, fix in first patch
- Low: Cosmetic, add to backlog
```

### Phase 5: Deployment
```
Deployment checklist:
- Technical go/no-go criteria met
- Business go/no-go criteria met
- Rollback plan documented and tested
- Hypercare support plan in place
- Communication sent to all affected users
- Change request approved (CAB)
- Maintenance window confirmed
- On-call team identified
- Monitoring alerts configured
- Training completed

Deployment risk-based windows:
- Low risk: Business hours with IT on standby
- Medium risk: Off-hours (Friday evening, weekend)
- High risk: Long maintenance window + business blackout period
```

### Phase 6: Closure
```
Activities:
- Verify all deliverables accepted
- Obtain formal project closure sign-off
- Document lessons learned
- Release resources back to resource pool
- Archive project documentation
- Transition to operations (runbook, training, support plan)
- Celebrate team success

Key deliverables:
- Formal acceptance document (signed)
- Lessons learned report
- Transition document (to operations team)
- Final project report

Hypercare period:
- Standard: 30 days post go-live with heightened support
- Complex: 60-90 days (ERP, major infrastructure)
```

---

## 3. Scoping & Charter <a name="scoping"></a>

### Project Charter Template
```
PROJECT CHARTER

Project Name: [Name]
Project ID: [ID]
Date: [Date]

PROJECT OVERVIEW
Problem Statement: [What problem are we solving?]
Proposed Solution: [High-level description]
Business Justification: [Why now? ROI summary]

OBJECTIVES (SMART)
1. [Specific, Measurable, Achievable, Relevant, Time-bound]

SUCCESS CRITERIA
- [Measurable outcome 1]
- [Measurable outcome 2]

IN SCOPE
- [Deliverable 1]
- [System/location 1]

OUT OF SCOPE
- [Explicitly excluded items]
- [Future phase items]

ASSUMPTIONS
- [e.g., "Vendor will provide implementation support"]

CONSTRAINTS
- Budget: $[amount]
- Timeline: Complete by [date]

RISKS (Initial)
- [Risk 1]: [Likelihood/Impact]

STAKEHOLDERS
Role             | Name     | Responsibility
Executive Sponsor| [Name]   | Final decisions, funding
Project Manager  | [Name]   | Day-to-day management
Business Owner   | [Name]   | Requirements, UAT sign-off

BUDGET ESTIMATE
Labor (internal): $[amount]
Vendor/Licensing: $[amount]
Hardware:         $[amount]
Training:         $[amount]
Contingency (15%):$[amount]
TOTAL:            $[amount]

APPROVALS
Executive Sponsor: _________________ Date: _______
IT Director:       _________________ Date: _______
Business Owner:    _________________ Date: _______
```

### Scope Creep Management
```
Prevention:
1. Detailed scope with explicit "out of scope" list
2. Formal change request process communicated at kickoff
3. Requirements freeze 2 weeks before go-live
4. Requirements traceability matrix (RTM)

Change Request Process:
1. Requestor submits Change Request form
2. PM evaluates impact on scope/schedule/cost
3. PM presents options to sponsor (accept/defer/reject)
4. Sponsor approves or denies
5. PM updates project plan if approved

Change Request Form fields:
- Requestor name/date
- Description of requested change
- Business justification
- Impact analysis (schedule +X days, cost +$Y)
- Recommendation
- Sponsor approval/rejection
```

---

## 4. Stakeholder Management <a name="stakeholders"></a>

### Power/Interest Grid
```
                 High Power
                     |
    Manage Closely   |  Keep Satisfied
    (high interest)  |  (low interest)
—————————————————————+————————————————
    Keep Informed    |  Monitor
    (high interest)  |  (low interest)
                     |
                  Low Power

Plot each stakeholder and manage accordingly.
```

### Stakeholder Register
| ID | Name | Role | Power | Interest | Attitude | Engagement Strategy | Frequency |
|----|------|------|-------|----------|----------|---------------------|-----------|
| 1 | CEO | Exec Sponsor | High | Low | Champion | Monthly exec briefing | Monthly |
| 2 | IT Director | IT Lead | High | High | Champion | Weekly status, daily involvement | Weekly |
| 3 | Dept Manager | Business Owner | Medium | High | Resistant | Weekly 1:1, involve in UAT | Weekly |
| 4 | End Users | Users | Low | Medium | Neutral | Newsletters, training | As needed |

### Managing Difficult Stakeholders
```
The Disengaged Executive:
- Lead with BLUF (Bottom Line Up Front)
- Use dashboards, not paragraphs
- Frame in business terms (cost, risk, revenue)

The Resistant Business Owner:
- Understand their resistance (fear of change? past failures?)
- Include them early in requirements
- Show quick wins and address concerns directly

The Scope Creeper:
- Reference the charter at every meeting
- "Great idea — let's add that to Phase 2"
- Formal change request process makes cost/time visible

The Absent Technical Lead:
- Escalate to their manager (project risk)
- Document all technical decisions with source
- Identify backup resource
```

---

## 5. Planning & Scheduling <a name="planning"></a>

### Work Breakdown Structure (WBS)
```
WBS Example: Email Migration to M365

1.0 Email Migration Project
  1.1 Project Management
    1.1.1 Project planning
    1.1.2 Status reporting
  1.2 Discovery & Assessment
    1.2.1 Inventory current environment
    1.2.2 Analyze mailbox sizes
    1.2.3 Identify shared mailboxes and distribution lists
  1.3 Design & Configuration
    1.3.1 Azure AD Connect configuration
    1.3.2 Exchange hybrid setup
    1.3.3 Mail flow rules design
  1.4 Pilot Migration (25 users)
    1.4.1 Migrate pilot mailboxes
    1.4.2 Validate email functionality
    1.4.3 Pilot user feedback
  1.5 Full Migration (waves)
    1.5.1 Wave 1 migration (100 users)
    1.5.2 Wave 2 migration (200 users)
  1.6 Cutover
    1.6.1 MX record cutover
    1.6.2 Decommission on-prem Exchange
  1.7 Training & Documentation
  1.8 Project Closure

Rule: Each lowest-level item should be completable in 8-80 hours.
```

### Estimation Techniques
```
Analogous: Base estimate on similar past project
- Fast, low accuracy (+/-50%)
- Good for initial business cases

Parametric: Use metrics (e.g., $500 per mailbox migrated)
- Moderate accuracy (+/-25%)
- Requires historical data

Bottom-up: Estimate each WBS element, roll up
- Most accurate (+/-10%)
- Time-intensive

Three-point (PERT):
E = (Optimistic + 4 x Most Likely + Pessimistic) / 6

Example: Configure server
- Optimistic: 2 hours, Most likely: 4 hours, Pessimistic: 8 hours
- PERT estimate: (2 + 16 + 8) / 6 = 4.3 hours

Contingency reserve:
- Low risk project: 10% of total estimate
- Medium risk: 15-20%
- High risk / new technology: 20-30%
```

### Critical Path Method
```
Steps:
1. List all tasks with durations and dependencies
2. Calculate Early Start (ES) and Early Finish (EF) forward pass
3. Calculate Late Start (LS) and Late Finish (LF) backward pass
4. Float = LS - ES
5. Critical path = tasks with 0 float

Key insight: Any delay to a critical path task delays the project.

Schedule compression:
- Crashing: Add resources to critical tasks (cost increases)
- Fast-tracking: Overlap sequential tasks (risk increases)
- Scope reduction: Remove lowest-priority requirements
```

---

## 6. Risk Management <a name="risk"></a>

### Risk Register Template
```
ID  | Description                    | Probability | Impact  | Score | Response Strategy        | Owner      | Status
R01 | Key SME leaves mid-project      | Medium (3)  | High (4)| 12    | Mitigate: document/cross-train backup | IT Mgr | Open
R02 | Vendor delivery delay           | Medium (3)  | High (4)| 12    | Mitigate: contract penalties, weekly check-in | PM | Open
R03 | Legacy system incompatibility   | Low (2)     | High (5)| 10    | Mitigate: PoC before commit; contingency API | Tech Lead | Open
R04 | Budget overrun from scope creep | Medium (3)  | Med (3) | 9     | Avoid: strict change control; 15% contingency | PM | Open

Score = Probability (1-5) x Impact (1-5)
Critical threshold: Score >= 12 = immediate attention required
```

### Risk Response Strategies
| Strategy | When to Use | Example |
|----------|------------|---------|
| Avoid | Eliminate the risk | Drop risky feature from scope |
| Mitigate | Reduce probability or impact | Add testing, hire backup resource |
| Transfer | Shift risk to third party | Insurance, vendor SLA with penalties |
| Accept | Too small or unavoidable | Document and monitor |
| Escalate | Outside PM authority | Inform sponsor, get decision |

### Issue Management
```
Risk vs. Issue:
- Risk: Something that MIGHT happen (future)
- Issue: Something that HAS happened (present, needs action now)

Issue Log fields:
- Issue ID and date raised
- Description and impact
- Priority (Critical/High/Medium/Low)
- Owner (who is resolving)
- Action plan and target resolution date
- Status

Escalation triggers:
- Critical issue unresolved after 48 hours
- Issue impacts project baseline (scope/schedule/budget)
- Requires decision outside PM authority
```

---

## 7. Change Management (Organizational) <a name="change"></a>

### ADKAR Model
```
A — Awareness: Why the change is needed
D — Desire: Personal motivation to support the change
K — Knowledge: How to change (training, documentation)
A — Ability: Skills and practice to implement
R — Reinforcement: Sustain the change, prevent reverting

Applying to a system migration:
A: Town halls, "why are we doing this?" FAQs
D: Show personal benefits (faster, easier), early adopter program
K: Training sessions, video tutorials, live Q&A
A: Practice environment, 30-day support hotline
R: Share success metrics, reward early adopters
```

### Resistance Management
```
"This system works fine" — Show metrics proving the problem
"I don't have time to learn" — Make training easy and flexible
"IT never delivers on time" — Communicate progress; celebrate milestones
"This isn't what we asked for" — Involve users in requirements and UAT

Rule: Don't argue with resistance. Diagnose and address the root cause.
```

---

## 8. Agile for IT Projects <a name="agile"></a>

### Scrum for IT Infrastructure
```
Roles:
- Product Owner: Business stakeholder defining priorities
- Scrum Master: Facilitates ceremonies, removes blockers
- Development Team: IT engineers doing the work

Artifacts:
- Product Backlog: All work to be done, prioritized
- Sprint Backlog: Work committed to current sprint
- Increment: Working output delivered each sprint

Ceremonies:
- Sprint Planning (2h): Select backlog items, create tasks
- Daily Standup (15min): Yesterday/Today/Blockers only
- Sprint Review (1h): Demo to stakeholders
- Sprint Retrospective (1h): What worked? What to improve?

Sprint length: 2 weeks standard

User Story format:
"As a [user type], I want [functionality] so that [business value]"

Definition of Done (DoD):
- Configuration reviewed by peer
- Tested in staging environment
- Documentation updated
- Security reviewed (if applicable)
- Product Owner acceptance
```

### Kanban for IT Operations
```
Board: Backlog → To Do → In Progress → Review → Done

WIP limits:
- To Do: 10 items max
- In Progress: 3 per person max
- Review: 5 items max

Metrics:
- Cycle time: Work starts → Work finishes
- Lead time: Request received → Delivered
- Throughput: Items completed per week

Kanban vs Scrum:
- Kanban: Continuous flow (helpdesk, operations, maintenance)
- Scrum: Time-boxed projects with defined scope
```

---

## 9. Infrastructure Project Specifics <a name="infrastructure"></a>

### Data Center Migration Checklist
```
Phase 1: Discovery (4-8 weeks)
- Complete server/application inventory
- Map all dependencies (app-to-DB, app-to-app)
- Identify all network connections
- Document storage requirements
- Interview application owners
- Classify applications by criticality
- Establish migration waves

Phase 2: Planning (4 weeks)
- Define migration strategy per app (Lift-Shift vs. Refactor)
- Create cutover runbook per application
- Plan IP scheme for target environment
- Order hardware (8-16 week lead times)

Phase 3: Build Target Environment
- Rack and stack hardware / provision cloud resources
- Configure networking, DNS, DHCP
- Install and configure OS
- Configure monitoring and backup

Phase 4: Migration Waves
- Migrate pilot application (low criticality, full test)
- Validate: Functionality, performance, security
- Execute subsequent waves per plan
- Decommission source servers after validation

Phase 5: Cutover
- Final migration of remaining systems
- DNS TTL reduced 48 hours before cutover
- Communication sent to business
- Hypercare activated
```

### Network Upgrade Project
```
Key phases:
1. Network assessment (current state)
2. Design (topology, VLAN plan, addressing scheme)
3. Hardware procurement (8-26 week lead times for enterprise gear)
4. Lab/staging configuration
5. Site-by-site rollout (least risk first)
6. Validation and acceptance testing per site
7. Documentation update

Always have:
- Out-of-band management (console server, IPMI)
- Rollback plan per site (previous config backed up)
- Maintenance windows per site
- Network team on-call during cutovers
```

---

## 10. Vendor & Procurement Management <a name="vendor"></a>

### RFP Process Timeline
```
Week 1-2: Define requirements (business, technical, security, commercial)
Week 3:   Create RFP document
Week 4:   Distribute to shortlisted vendors (3-5)
Week 5-6: Vendor Q&A period
Week 7:   Responses due
Week 8-9: Evaluate, score, request demos
Week 10:  Finalist selection + reference checks
Week 11-12: Negotiation + contract execution

Evaluation scorecard (100 points):
- Technical fit:              35 points
- Implementation approach:    20 points
- Company stability/references:15 points
- Total cost of ownership:    20 points
- Support model/SLA:          10 points
```

### SOW Key Clauses
```
Must-have sections:
1. Scope of work (detailed deliverables list)
2. Assumptions and exclusions
3. Acceptance criteria
4. Project timeline with milestones
5. Roles and responsibilities
6. Pricing and payment schedule (tied to milestone acceptance)
7. Change order process
8. Escalation process
9. SLAs during project and hypercare
10. IP ownership
11. Termination clauses
12. Penalty clauses for missed milestones

Payment structure best practice:
- 25% contract execution
- 25% design complete
- 25% UAT sign-off
- 25% go-live + 30-day hypercare
- Never pay 100% upfront
```

---

## 11. Project Communication <a name="communication"></a>

### Status Report Template (Weekly)
```
PROJECT STATUS REPORT
Project: [Name] | Date: [Date] | PM: [Name]
Overall: GREEN | Schedule: GREEN | Budget: YELLOW | Scope: GREEN

EXECUTIVE SUMMARY
[2-3 sentences: Where we are, what happened, what's coming next]

THIS WEEK
- Completed item 1
- Completed item 2

NEXT WEEK
- Planned item 1 — Owner: [Name]
- Planned item 2 — Owner: [Name]

MILESTONES
Milestone             | Target     | Forecast   | Status
Design Complete       | 2025-01-15 | 2025-01-15 | Done
Build Complete        | 2025-02-28 | 2025-03-07 | AT RISK (+7 days, hardware delay)
Go-Live               | 2025-04-01 | 2025-04-08 | AT RISK

BUDGET
Approved:      $450,000
Spent to Date: $187,000 (42%)
Forecasted:    $465,000 (+$15,000 hardware price increase)
Variance:      -3.3% ESCALATED

TOP RISKS/ISSUES
ISSUE: Hardware delivery delayed 2 weeks
  Impact: Go-live may slip to April 8
  Action: Sourcing alternate vendor

DECISIONS NEEDED
- Should we proceed with alternate hardware vendor at +$15k?
  Needed by: [Date] | Decision maker: [Sponsor]
```

### Escalation Framework
```
Level 1 — PM resolves: Minor issues within PM authority
Level 2 — IT Manager: Cross-team blockers, small budget variance (<5%)
Level 3 — IT Director: Schedule slip >2 weeks, budget >5%, vendor issues
Level 4 — Exec Sponsor: Schedule slip >1 month, budget >10%, scope material change

Escalation email format:
Subject: [ESCALATION] [Project] — [Issue summary]
- Situation: What happened
- Impact: Effect on project (cost/schedule/scope)
- Options: 2-3 options with pros/cons
- Recommendation: What PM recommends
- Decision needed by: [Date]
```

---

## 12. Project Closure & Lessons Learned <a name="closure"></a>

### Lessons Learned Workshop (90 min)
```
1. Ground rules (10 min)
   - Focus on process, not people
   - All feedback is valid

2. What went well? (20 min)
   - Round-robin contributions
   - Document practices to repeat

3. What could have gone better? (30 min)
   - Focus on root cause (ask "Why?" 3-5 times)
   - Examples: underestimated scope, poor user comms, vendor delays

4. What would we do differently? (20 min)
   - Turn each problem into an action recommendation
   - Assign owner and target date

5. Wrap-up (10 min)
   - Share report with stakeholders
   - Feed improvements into PM process
```

### Transition to Operations
```
Transition document must include:
- System architecture and topology diagram
- Runbook (day-to-day operational procedures)
- Monitoring and alerting setup
- Backup and recovery procedures
- Vendor contacts and support contracts
- License information and renewal dates
- Known issues and workarounds
- Escalation paths for support
- Training materials for support team

Acceptance criteria:
- Support team trained and demonstrates competency
- Runbook reviewed and approved
- Monitoring live and tested
- Backup tested (restore verified)
- Helpdesk KB articles published
- Formal sign-off from IT operations lead
```

---

## 13. Templates & Tools <a name="templates"></a>

### RACI Matrix
```
Activity                  | PM | Tech Lead | Dev Team | Security | Business Owner | Exec Sponsor
Requirements gathering     | A  | R         | C        | C        | R              | I
Technical design           | I  | A         | R        | C        | C              | I
Build / Configure          | A  | R         | R        | C        | I              | I
UAT                        | A  | C         | C        | I        | R              | I
Go / No-Go decision        | R  | C         | I        | C        | C              | A
Production deployment      | A  | R         | R        | C        | I              | I
Project closure sign-off   | A  | I         | I        | I        | C              | R

R = Responsible | A = Accountable | C = Consulted | I = Informed
```

### PM Tool Selection Guide
| Tool | Best For | Cost |
|------|----------|------|
| Microsoft Project | Complex schedules, resource management | Paid |
| Azure DevOps | Infra/software sprints, Kanban | Free/Paid |
| Jira | Agile IT projects, issue tracking | Paid |
| Asana | Team task management | Paid |
| M365 Planner | Simple projects, M365 integrated | Included in M365 |
| Trello | Small teams, Kanban boards | Free/Paid |

### Meeting Best Practices
```
Every meeting should have:
- Agenda sent 24 hours in advance
- Defined start and end time (honor both)
- Documented decisions and action items (Owner + Due Date)
- Notes distributed within 24 hours

Bad meeting warning signs:
- No agenda
- Decisions revisited at next meeting
- Action items with no owner or date
- Consistently runs over time
- Same people doing all the talking
```

---

*Last Updated: 2025 | IT Project Management Guide v2.0*
