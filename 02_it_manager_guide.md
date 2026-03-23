# IT Manager — Operations & Team Leadership Guide

## Overview
A practical reference for IT Managers covering day-to-day operations management, team leadership, performance management, incident management, change management, vendor coordination, and stakeholder communication.

---

## Table of Contents
1. [The IT Manager Role](#role)
2. [Day-to-Day Operations Management](#operations)
3. [Team Leadership & Performance](#team)
4. [Incident Management](#incident)
5. [Change & Release Management](#change)
6. [ITSM & Ticket Management](#itsm)
7. [Capacity Planning](#capacity)
8. [Stakeholder Communication](#communication)
9. [Onboarding & Offboarding Processes](#onboarding)
10. [Hiring & Interviewing](#hiring)
11. [Meetings That Actually Work](#meetings)
12. [Personal Effectiveness](#effectiveness)

---

## 1. The IT Manager Role <a name="role"></a>

### Manager vs. Individual Contributor
```
As a manager, your job is to multiply the output of your team, not do the work yourself.

Key shifts:
  IC: Depth of technical skill → Manager: Breadth + process + people
  IC: Personal productivity → Manager: Team throughput
  IC: Solve problems yourself → Manager: Enable others to solve problems
  IC: Technical accuracy → Manager: Prioritization and trade-offs
  IC: Short-term focus → Manager: Near + medium-term planning
```

### Core Accountabilities
- Ensure IT services meet SLAs and business needs
- Hire, develop, and retain IT talent
- Manage escalated incidents to resolution
- Oversee change management process for your team
- Own the team's portion of IT budget
- Report operational status to IT Director
- Drive process improvement initiatives
- Build relationships with business unit counterparts

### Manager's Weekly Rhythm

```
Monday:
  - Review weekend/overnight alerts and incidents
  - Team standup (or async if remote)
  - Review week's upcoming changes (CAB prep)
  - Review open tickets by priority

Tuesday–Thursday:
  - 1-on-1s with direct reports
  - Project check-ins
  - Stakeholder meetings
  - Handle escalations

Friday:
  - Weekly metrics review
  - Write up week's summary (incidents, changes, notable items)
  - Prepare for Monday
  - Team retrospective (bi-weekly)
```

---

## 2. Day-to-Day Operations Management <a name="operations"></a>

### Morning Check Routine (15 minutes)
```
□ Review monitoring dashboards (infrastructure, app, security)
□ Check overnight alert emails/tickets
□ Review queue depth and SLA status
□ Note any open P1/P2 tickets and assign/escalate
□ Check major upcoming changes for the day
□ Brief team at standup on priorities
```

### Operations Dashboard Metrics to Watch
| Metric | Review Frequency | Alert Threshold |
|---|---|---|
| Open P1 tickets | Real-time | Any open > 30 min |
| Open P2 tickets | Hourly | >2 open |
| Ticket queue depth | Daily | >20% above baseline |
| SLA breach risk | Daily | Any ticket >80% of SLA window |
| Server disk/CPU/RAM | Daily | >85% sustained |
| Failed login attempts | Daily | Spikes |
| Backup job failures | Daily | Any failure |
| Patch compliance | Weekly | <95% |

### Escalation Handling

```
P1 — Critical (business-critical system down, data breach)
  ✓ Notify your manager immediately
  ✓ Assemble incident response team
  ✓ Communicate to affected business leaders (15-min intervals)
  ✓ Dedicated bridge call open while active
  ✓ Document in real-time (incident commander takes notes)
  Target resolution: 1 hour

P2 — Major (significant degradation, workaround available)
  ✓ Assign senior engineer
  ✓ Notify affected business teams
  ✓ Hourly updates to stakeholders
  Target resolution: 4 hours

P3 — Moderate (non-critical issue, workaround available)
  ✓ Assign in queue
  ✓ Standard SLA applies
  Target resolution: 24 hours

P4 — Low (minor issue, low impact)
  ✓ Queue and assign per capacity
  Target resolution: 72 hours
```

---

## 3. Team Leadership & Performance <a name="team"></a>

### 1-on-1 Meeting Framework
Frequency: Weekly for new/struggling employees; bi-weekly for experienced staff

```
Agenda (30 minutes):
  5 min: How are you doing? (personal check-in, not project status)
  10 min: Their agenda — what do they want to discuss?
  10 min: Your agenda — feedback, coaching, priorities
  5 min: Action items and next steps

DO:
  - Let them lead the agenda
  - Ask about blockers, frustrations, career goals
  - Give specific, timely feedback
  - Follow up on previous action items

DON'T:
  - Use it as a status meeting (you have standups for that)
  - Cancel frequently (signals you don't value them)
  - Avoid difficult conversations (this is the place for them)
```

### Performance Management

**Setting Expectations:**
```
SMART Goals per team member:
  Specific: Clear definition of success
  Measurable: Quantifiable outcome
  Achievable: Realistic given capacity
  Relevant: Aligned to team/business objectives
  Time-bound: Clear deadline

Example:
  "Reduce mean time to resolve P3 tickets from 48 hours to 24 hours 
   by implementing a new triage process by June 30."
```

**Performance Issue Handling:**
```
Step 1: Verbal conversation (document in your notes)
  - Describe the specific behavior/gap (not the person)
  - State the expected standard
  - Ask for their perspective
  - Agree on action plan

Step 2: If not improved, written Performance Improvement Plan (PIP)
  - Clear goals with measurable outcomes
  - 30/60/90 day timeline
  - HR involvement required
  - Regular check-ins documented

Step 3: If still not improved
  - Follow HR process for separation
  - Never a surprise if you've done steps 1–2 properly
```

### Career Development Conversations
```
Annual questions to ask your team:
  - Where do you want to be in 2–3 years?
  - What skills do you want to build?
  - What's frustrating or not working for you?
  - What opportunities are you interested in?
  - What can I do differently to support you?

Build Individual Development Plans (IDPs):
  - 1 stretch assignment per year
  - Training budget usage plan
  - Mentorship (internal or external)
  - Certification path if applicable
```

---

## 4. Incident Management <a name="incident"></a>

### Incident Response Process

```
1. DETECTION
   Source: Monitoring alert, user report, automated trigger
   Action: Acknowledge alert within SLA, create incident ticket

2. TRIAGE
   Assess: Impact, urgency, affected users/systems
   Assign: Priority level (P1–P4)
   Assign: Resolver (based on domain, availability)

3. DIAGNOSIS
   Identify: What's broken, what's not
   Gather: Logs, metrics, user reports
   Hypothesize: Most likely root cause(s)

4. RESOLUTION
   Implement: Fix or workaround
   Test: Confirm issue resolved for affected users
   Monitor: Watch for recurrence

5. COMMUNICATION
   Notify: Stakeholders per communication plan
   Update: Ticket with resolution details

6. POST-INCIDENT REVIEW (P1/P2)
   Timeline: Within 48–72 hours of resolution
   Content: Timeline, root cause, contributing factors,
            action items (with owners and due dates)
   Format: Blameless — focus on systems, not people
```

### Incident Communication Templates

**Initial Notification (P1):**
```
SUBJECT: [INCIDENT] P1 - <System> Outage - <Time>

We are currently investigating an issue with <System> affecting <scope>.

Impact: <What users cannot do>
Status: Investigating
Next Update: <Time, e.g., 30 minutes>

Incident Manager: <Name>
Bridge: <Call link/number>
```

**Resolution Notification:**
```
SUBJECT: [RESOLVED] P1 - <System> Outage - Resolved at <Time>

The incident affecting <System> has been resolved.

Resolution: <Brief description of fix>
Duration: <Start time> to <End time>
Affected Users: <Count/scope>

A Post-Incident Review will be conducted on <Date>.
```

### Post-Incident Review Template
```
INCIDENT POST-MORTEM
Date: 
Severity: P1 / P2
Duration:
Systems Affected:
Users Impacted:

TIMELINE
[HH:MM] - Event or action
[HH:MM] - Event or action
[HH:MM] - Resolution

ROOT CAUSE
[Single root cause or contributing factors]

CONTRIBUTING FACTORS
1.
2.

WHAT WENT WELL
-
-

WHAT COULD BE IMPROVED
-
-

ACTION ITEMS
| Action | Owner | Due Date | Status |
|--------|-------|----------|--------|
|        |       |          |        |
```

---

## 5. Change & Release Management <a name="change"></a>

### Change Management Essentials

**The 5 Questions for Every Change:**
1. What are you changing and why?
2. What is the risk if it goes wrong?
3. What is the rollback plan?
4. How will you know it was successful?
5. Who needs to be notified?

**Maintenance Window Policy:**
```
Standard Maintenance Windows:
  - Production: Tuesday/Wednesday/Thursday 10pm–2am (avoid Mon/Fri)
  - Non-Production: Any time with approval

Emergency Changes:
  - Require: IT Manager + Business Owner approval
  - Complete full RFC within 24 hours post-implementation
  - Review at next CAB

Change Freeze Periods:
  - End of fiscal quarters (last 2 weeks)
  - Holiday periods
  - Major business events (product launches, trade shows)
```

---

## 6. ITSM & Ticket Management <a name="itsm"></a>

### Queue Management Principles
```
Daily Queue Review:
  1. Prioritize P1/P2 first (immediate attention)
  2. Review tickets approaching SLA breach
  3. Check tickets with no activity in 24+ hours
  4. Balance workload across team
  5. Identify tickets needing escalation or cross-team input
```

### Ticket Quality Standards
```
Every ticket should have:
  □ Clear problem statement (not just "doesn't work")
  □ Steps to reproduce
  □ Impact/affected users
  □ Time of occurrence
  □ What has already been tried
  □ Assigned owner
  □ Priority set correctly

Resolution notes should include:
  □ Root cause
  □ Steps taken to resolve
  □ Verification method
  □ Preventive recommendations
```

### SLA Targets by Priority
| Priority | Response Time | Update Frequency | Resolution Target |
|---|---|---|---|
| P1 - Critical | 15 min | Every 30 min | 1 hour |
| P2 - High | 30 min | Every 2 hours | 4 hours |
| P3 - Medium | 2 hours | Every 8 hours | 24 hours |
| P4 - Low | 8 hours | As needed | 72 hours |

---

## 7. Capacity Planning <a name="capacity"></a>

### Team Capacity Model
```
Available hours per FTE per week:
  Total: 40 hours
  Less: Meetings, admin, email: ~8 hours
  Less: Unplanned/reactive work: ~8 hours (varies)
  Available for planned work: ~24 hours

For a team of 5 engineers:
  Total planned capacity: 120 hours/week
  Rule of thumb: Don't exceed 80% planned utilization (buffer for reactive)
  Max planned load: 96 hours/week
```

### Staffing Level Indicators
```
Understaffed signals:
  - Team regularly working overtime
  - Ticket backlog growing week-over-week
  - Project timelines slipping due to reactive work
  - Staff expressing burnout
  - Quality of work declining

Overstaffed signals:
  - Engineers frequently asking for work
  - Low ticket volumes sustained
  - Projects completing well ahead of schedule
  - Difficulty justifying headcount to leadership
```

---

## 8. Stakeholder Communication <a name="communication"></a>

### Communication Cadence
```
Daily: Standup with team (10 min)
Weekly: Status email to IT Director
Weekly: Check-in with key business partners
Monthly: Metrics report to management
Quarterly: Business review with major stakeholders
Ad-hoc: Incident updates, major change notifications
```

### Writing Effective Status Reports
```
FORMAT: RAG status + bullet points
  
This Week:
  ✅ GREEN: [Item] - [Brief status]
  ⚠️ YELLOW: [Item] - [Issue and mitigation]
  🔴 RED: [Item] - [Problem, impact, action being taken]

Next Week:
  - [Planned milestone or focus area]

Issues Requiring Attention:
  - [Any blockers needing Director action or decision]

KPIs:
  - Uptime: 99.95%
  - Open P1/P2: 0
  - Ticket queue: 47 open (normal)
  - SLA compliance: 94%
```

---

## 9. Onboarding & Offboarding <a name="onboarding"></a>

### New Employee IT Onboarding Checklist

```
PRE-ARRIVAL (submitted by HR/Manager 5+ business days before):
  □ Request hardware (laptop, peripherals, phone if needed)
  □ Create AD account
  □ Set up email
  □ Assign required licenses (Microsoft 365, etc.)
  □ Add to required security groups
  □ Set up MFA
  □ Provision business applications
  □ Create ITSM account
  □ Add to distribution lists and Teams channels
  □ Prepare welcome email with credentials

DAY 1:
  □ Device ready and tested before employee arrives
  □ IT orientation (30 min walk-through of systems)
  □ Confirm all access is working
  □ Provide IT contact info and how to get help
  □ Set up password change on first login

WEEK 1 FOLLOW-UP:
  □ Check in on any access issues
  □ Confirm all required apps are accessible
  □ Gather feedback on IT setup experience
```

### Employee Offboarding Checklist

```
TRIGGER: HR submits offboarding request (last day: ____)

24 HOURS BEFORE LAST DAY:
  □ Brief manager: "Accounts will be disabled at end of business on [date]"

LAST DAY (end of business):
  □ Disable AD account
  □ Reset password
  □ Revoke MFA tokens
  □ Disable VPN access
  □ Suspend/reassign email (set auto-reply, forward to manager)
  □ Revoke remote access tools
  □ Remove from distribution lists (or retain if needed)

WITHIN 48 HOURS:
  □ Retrieve and document equipment return
  □ Remove from all SaaS applications
  □ Reassign or deprovision licenses
  □ Document in offboarding log

30 DAYS POST-DEPARTURE:
  □ Archive mailbox (per retention policy)
  □ Review for any remaining active accounts
  □ Wipe and redeploy returned equipment
```

---

## 10. Hiring & Interviewing <a name="hiring"></a>

### Interview Process for IT Roles

```
Stage 1: Phone Screen (30 min, IT Manager)
  - Verify experience vs. JD
  - Gauge communication skills
  - Discuss compensation expectations
  - Sell the opportunity

Stage 2: Technical Assessment
  - Option A: Live troubleshooting exercise
  - Option B: Take-home scenario (return within 48 hrs)
  - Evaluate: Diagnostic approach, not just correct answer

Stage 3: Panel Interview (60–90 min)
  - IT Manager + 2 team members
  - Behavioral questions (STAR method)
  - Technical deep-dive
  - Culture/team fit assessment

Stage 4: Director/Final Interview
  - Strategic alignment
  - Career trajectory
  - Final questions
```

### Technical Interview Questions by Level

**L1 Help Desk:**
- "Walk me through how you would troubleshoot a user who can't connect to Wi-Fi."
- "What's the difference between DNS and DHCP?"
- "A user calls and says their computer is slow. What do you do first?"

**L2 Systems Administrator:**
- "Explain what happens when you type a URL into a browser and press Enter."
- "How would you diagnose a server with intermittent high CPU usage?"
- "Describe your experience with Group Policy — what's the most complex GPO you've configured?"
- "A user's account is locked out every morning. How do you investigate?"

**Senior/L3 Engineer:**
- "Describe how you would design a new AD site for a branch office."
- "How would you approach migrating 500 on-prem mailboxes to Exchange Online?"
- "Walk me through your approach to a P1 incident at 2am — a core application is down."
- "How do you balance security hardening with operational usability?"

---

## 11. Meetings That Actually Work <a name="meetings"></a>

### Meeting Hygiene Rules
```
Before scheduling:
  - Can this be an email?
  - Who MUST be there vs. nice to have?
  - Is 30 minutes enough instead of 60?

Every meeting needs:
  - Clear objective: "By end of this meeting, we will have decided/resolved..."
  - Agenda sent 24+ hours in advance
  - Owner for each agenda item
  - Time limit enforced

After every meeting:
  - Action items documented (WHO does WHAT by WHEN)
  - Notes distributed within 24 hours
  - Decisions recorded (not just actions)
```

### Effective Standup (10 minutes max)
```
Each person answers:
  1. What did I complete yesterday?
  2. What am I doing today?
  3. What's blocking me?

Rules:
  - Stand up (keeps it short)
  - Start on time, end on time
  - Detailed discussion → "take it offline" with relevant people
  - Manager: listen for blockers to remove, not to report status
```

---

## 12. Personal Effectiveness <a name="effectiveness"></a>

### Managing Up Effectively
- Give your Director no surprises (tell them bad news before they hear it elsewhere)
- Bring solutions, not just problems (present 2–3 options when escalating)
- Know their priorities — align your work to what they care about
- Proactively communicate status — don't make them ask
- Ask for feedback explicitly — "What could I be doing better?"

### Manager Pitfalls to Avoid
```
Technical trap: Spending too much time doing IC work instead of enabling your team
Conflict avoidance: Delaying difficult performance conversations
Over-promising: Committing to deadlines without team input
Under-delegating: Not trusting your team with meaningful work
Isolation: Not managing relationships with peers and business partners
Reactive-only mode: No time for proactive improvement or planning
```

### Reading List for IT Managers
- *The Phoenix Project* — Gene Kim (IT as a business partner)
- *Team of Teams* — Stanley McChrystal (adaptable organizations)
- *Radical Candor* — Kim Scott (feedback and management)
- *The Manager's Path* — Camille Fournier (technical management)
- *An Elegant Puzzle* — Will Larson (engineering management)
- *Site Reliability Engineering* — Google (operating at scale)
