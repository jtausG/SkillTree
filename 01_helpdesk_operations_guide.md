# Help Desk & Service Desk Operations Guide

## Overview
Complete operations guide for IT Help Desk / Service Desk teams covering ticketing workflows, common resolutions, user communication, SLA management, escalation procedures, and team quality standards.

---

## Table of Contents
1. [Help Desk Role & Philosophy](#role)
2. [Ticket Lifecycle Management](#tickets)
3. [Common Issue Resolution Index](#resolutions)
4. [User Communication Standards](#communication)
5. [Escalation Procedures](#escalation)
6. [Remote Support Best Practices](#remote)
7. [Knowledge Base Management](#kb)
8. [Quality Assurance](#qa)
9. [Help Desk Metrics & Reporting](#metrics)
10. [Shift Handoff Procedures](#handoff)

---

## 1. Help Desk Role & Philosophy <a name="role"></a>

### Mission Statement
The Help Desk is the face of IT to the business. Every interaction is an opportunity to earn trust, demonstrate IT's value, and enable employees to do their best work.

### The 3 Cs of Help Desk Excellence
```
COMPETENCE: Know your stuff, and know when to escalate
COMMUNICATION: Proactive, clear, and human — never let users wonder about their ticket
COMMITMENT: Follow through on what you say you'll do
```

### First Call Resolution (FCR) Philosophy
```
Goal: Resolve 75%+ of tickets without escalation

To improve FCR:
  - Document every unique issue in the KB
  - Review escalated tickets weekly — could they have been solved at L1?
  - Cross-train on common L2 issues
  - Use remote desktop tools proactively
  - Don't escalate before trying obvious fixes
  - Ask: "What would a senior tech do here?"
```

---

## 2. Ticket Lifecycle Management <a name="tickets"></a>

### Ticket Intake Checklist

Every ticket must have these fields captured:
```
□ Reporter name and contact info (phone/email)
□ Affected user (if different from reporter)
□ Affected device/system (hostname if possible)
□ Problem description (in user's words)
□ Error message (exact text)
□ Steps to reproduce
□ When did it start?
□ Has it worked before?
□ How many users affected?
□ Business impact
□ Priority set correctly
□ Category/subcategory assigned
□ Assigned to correct team/person
```

### Priority Determination Guide

| Priority | Definition | Example | Response | Resolution |
|---|---|---|---|---|
| P1 - Critical | Complete business stoppage or security incident | Email server down, ransomware, e-commerce outage | 15 min | 1 hour |
| P2 - High | Major impact, workaround available | Core app degraded, exec cannot work, VPN down for team | 30 min | 4 hours |
| P3 - Medium | Individual impact, workaround exists | Single user can't access application, printer broken | 2 hours | 24 hours |
| P4 - Low | Minor issue, not time-sensitive | Cosmetic bug, non-urgent request, training needed | 8 hours | 72 hours |
| Service Request | New user setup, access request, equipment order | Onboarding new hire, password reset | 4 hours | Per SLA |

### Ticket States
```
New → Assigned → In Progress → Pending (waiting on user/vendor/part) → Resolved → Closed

Rules:
  - "Pending" requires a note explaining what you're waiting for and expected resolution date
  - Resolved tickets auto-close after 3 business days if user doesn't reopen
  - Never close a ticket without documenting the resolution
  - Reopen a closed ticket rather than creating a new one for the same issue
```

---

## 3. Common Issue Resolution Index <a name="resolutions"></a>

### Password & Account Issues

**Password Reset:**
```
VERIFY IDENTITY FIRST:
  □ Employee ID + manager name (phone)
  □ Secondary email confirmation (if available)
  □ Video call verification (for exec or sensitive accounts)
  Never reset over email if identity unverified — social engineering risk!

Steps:
  1. Verify user identity per above
  2. Reset via AD → Active Directory Users and Computers
     OR: Set-ADAccountPassword -Identity jsmith -Reset -NewPassword (Read-Host -AsSecureString)
  3. Check "User must change password at next logon"
  4. Unlock if locked: Unlock-ADAccount -Identity jsmith
  5. Advise user to change password immediately
  6. If MFA is used, verify MFA still works after reset
```

**Account Lockout:**
```
1. Verify identity
2. Check lockout source:
   Event ID 4740 on PDC Emulator (most accurate)
   Get-WinEvent -ComputerName <PDC> -FilterHashtable @{LogName='Security';Id=4740} | 
     Where-Object {$_.Message -match "jsmith"} | Select-Object TimeCreated, Message

3. Common lockout sources:
   - Cached credentials on an old device/phone
   - Mapped drives with old credentials
   - Scheduled tasks running as the user
   - Browser saved passwords (Chrome/Edge)
   - Outlook mobile with old password

4. Resolution:
   - Unlock account: Unlock-ADAccount -Identity jsmith
   - Instruct user to update password everywhere it's saved
   - If keeps locking: install LockoutStatus tool, identify source computer
```

**MFA Issues:**
```
Authenticator app issues:
  1. Time sync: Ensure phone time is set to "automatic"
  2. Re-sync: In app → cog icon → refresh/sync
  3. If app missing: Delete and re-add account in authenticator
  4. If phone lost: Admin resets MFA from Azure AD portal →
     User → Authentication methods → Delete MFA methods
     User re-enrolls at next login

Hardware token (FIDO2/YubiKey):
  1. Check if registered: Azure AD → User → Authentication methods
  2. Re-register if needed
  3. For lost tokens: Remove from AD, provide temporary access method
```

### Connectivity Issues

**No Network / "Limited Connectivity":**
```
Step 1: Physical layer
  - Confirm cable is plugged in / Wi-Fi is on
  - Check link light on NIC
  - Try different cable or port

Step 2: IP configuration
  ipconfig /all  → Is IP in correct range? Or 169.254.x.x (APIPA — DHCP failed)?
  
  If APIPA:
    ipconfig /release
    ipconfig /renew
    If still APIPA: Check switch port VLAN, DHCP server/relay

Step 3: Connectivity tests
  ping 127.0.0.1    → Loopback (TCP/IP stack OK)
  ping <gateway>    → LAN OK
  ping 8.8.8.8      → Internet OK (bypass DNS)
  ping google.com   → DNS + Internet OK

Step 4: DNS
  ipconfig /flushdns
  nslookup google.com
  nslookup google.com 8.8.8.8  (test alternate DNS)

Step 5: Reset network stack (if all else fails)
  netsh int ip reset
  netsh winsock reset
  Reboot required
```

**VPN Issues:**
```
Cannot connect:
  □ Check internet connectivity first (VPN needs internet)
  □ Verify credentials (try same on web portal)
  □ Check MFA is working
  □ Try different VPN protocol (SSL vs IPsec)
  □ Disable local firewall temporarily (test)
  □ Check VPN client version (outdated?)
  □ Reinstall VPN client

Connected but cannot reach resources:
  □ Check split tunnel configuration — is the resource in split tunnel exclusion?
  □ DNS while on VPN: nslookup internalserver
  □ Try IP address directly (if DNS works, DNS is fine)
  □ Check time sync (Kerberos requires <5 min skew)
```

### Email Issues

**Outlook Not Opening / Crashes:**
```
1. Safe mode: outlook.exe /safe
2. Disable add-ins: File → Options → Add-Ins → Manage COM Add-Ins → Disable all → Test
3. Repair profile: outlook.exe /resetnavpane
4. Create new Outlook profile: Control Panel → Mail → Show Profiles → Add
5. Repair Office: Settings → Apps → Microsoft 365 → Modify → Quick Repair
6. Online repair (if Quick Repair fails): As above → Online Repair
```

**Cannot Send/Receive:**
```
1. Check connectivity to mail server:
   Test-NetConnection -ComputerName outlook.office365.com -Port 993
   Test-NetConnection -ComputerName smtp.office365.com -Port 587

2. Check account settings: File → Account Settings → Repair

3. Check if issue is all email or just one contact
   - All email: Server/connectivity issue
   - One contact: Check spam filters, recipient server rejecting

4. Review Send/Receive error:
   Send/Receive → Show Progress → Details → Errors
```

### Printer Issues

**Printer Not Printing:**
```
1. Check physical:
   - Power on, paper loaded, no error lights
   - Check printer display for error messages

2. Check print queue:
   - Open Devices and Printers → right-click printer → See what's printing
   - Cancel stuck jobs
   - Restart print spooler:
     Stop-Service Spooler -Force
     Remove-Item "$env:SystemRoot\System32\spool\PRINTERS\*" -Force -ErrorAction SilentlyContinue
     Start-Service Spooler

3. Check driver:
   - Remove and re-add printer
   - Download latest driver from manufacturer

4. Network printer:
   - Ping the printer IP
   - Check if printer webpage loads (most have web interface)
   - Verify port configuration: Devices and Printers → Printer Properties → Ports
```

### Slow Computer

**Standard Performance Checklist:**
```
1. Check disk space: Must have >15% free (C: drive)
   - Clean temp files: cleanmgr.exe
   - Windows cleanup: Settings → System → Storage → Temp files

2. Check processes (Task Manager):
   - High CPU: Which process? Antivirus scan? Windows Update?
   - High RAM: Too many apps open? Memory leak?
   - High Disk: 100% disk usage → check for Windows Update or antivirus scan in progress

3. Startup items:
   Task Manager → Startup tab → Disable unnecessary items

4. Hardware:
   - Verify device is not exceeding thermal limits (fans working?)
   - SSD vs HDD: HDDs dramatically slower
   - RAM: Check if at minimum spec (8GB minimum, 16GB recommended)
   - Run Windows Memory Diagnostic: mdsched.exe

5. Malware scan:
   Windows Security → Virus & threat protection → Quick scan

6. System maintenance:
   - SFC /scannow (corrupted system files)
   - DISM /Online /Cleanup-Image /RestoreHealth (if SFC fails)
```

---

## 4. User Communication Standards <a name="communication"></a>

### Phone Etiquette
```
Answer: "IT Help Desk, this is [Name], how can I help you?"
Never say:
  - "I don't know" → "That's a great question, let me find out for you"
  - "That's not my problem" → "Let me connect you with the right team"
  - "You need to restart it" → "Let's try a restart together — I'll stay on the line"
  - "Did you read the email we sent?" → Assume they didn't; help anyway

When you need to put someone on hold:
  - Ask permission: "May I put you on hold for about 2 minutes while I look into this?"
  - Never hold longer than 2 minutes without checking back in
  - "Thank you for holding — I'm still researching this. Can I call you back within 30 minutes?"
```

### Ticket Update Communication Templates

**Acknowledgment (within SLA response time):**
```
Hi [Name],

Thank you for reaching out. I've received your ticket regarding [brief description] 
and I'm looking into it now.

I'll have an update for you by [specific time/date].

Ticket #: [number]

[Your name]
IT Help Desk
```

**Update while working:**
```
Hi [Name],

Quick update on ticket #[number]: I've [what you've done] and I'm currently 
[what you're doing next].

Current status: [In Progress / Waiting on vendor / Waiting on part]
Next update: [by specific time]

Please let me know if the business impact changes or you need to discuss.

[Your name]
```

**Resolution:**
```
Hi [Name],

I'm pleased to let you know that [brief description of issue] has been resolved.

What was done: [Clear non-technical summary of the fix]

Please test and confirm everything is working for you. If you experience 
any further issues, please don't hesitate to reach out or reply to this email.

Thank you for your patience!

[Your name]
IT Help Desk
Ext: [extension]
```

---

## 5. Escalation Procedures <a name="escalation"></a>

### When to Escalate

```
Escalate immediately:
  ✓ Any P1 or suspected P1 — don't wait, escalate immediately
  ✓ Suspected security incident (malware, data breach, account compromise)
  ✓ Issue affecting C-suite or VIP users
  ✓ System or service with widespread impact (10+ users)

Escalate after 15–30 minutes of L1 investigation:
  ✓ Issue requires access you don't have
  ✓ Issue requires domain admin or server admin knowledge
  ✓ Problem not resolved after following known runbooks
  ✓ Issue seems related to infrastructure (server, network, database)

Escalate to manager:
  ✓ Hostile or unreasonable user
  ✓ User threatening legal action
  ✓ Sensitive or HR-related requests
  ✓ P1/P2 that may need business communications
```

### Escalation Handoff Template
```
Escalating to: [L2 / Tier 2 / Infrastructure Team / Manager]

TICKET: #[number]
PRIORITY: [P1/P2/P3]
USER: [Name, title, phone]
SYSTEM: [Computer name, OS]

PROBLEM: [Clear description]
ERROR MESSAGE: [Exact text]
STARTED: [When]
IMPACT: [How many users / what's broken]

STEPS TAKEN:
1. [What you tried]
2. [What you tried]
3. [Outcome of each]

HYPOTHESIS: [Your best guess at root cause]

USER AVAILABILITY: [Best way to contact user, their schedule]
BUSINESS URGENCY: [Any deadlines or events depending on this?]
```

---

## 6. Remote Support Best Practices <a name="remote"></a>

### Remote Session Guidelines
```
Always:
  □ Announce when you're connecting ("I'm connecting to your computer now")
  □ Narrate what you're doing: "I'm going to open Task Manager to check processes"
  □ Ask before making changes: "I'd like to restart the print spooler — is that OK?"
  □ Disconnect when done: "I'm done — you have control of your computer back"

Never:
  □ Leave a remote session unattended with control
  □ Browse unrelated to the issue
  □ Access personal files or email
  □ Disable security software and leave it disabled
```

### Remote Diagnostic Commands (Quick Reference)
```powershell
# Check computer name and basic info
hostname; (Get-CimInstance Win32_OperatingSystem).Caption

# Check current user
whoami; query user

# Check IP configuration
ipconfig /all | Select-String -Pattern "IPv4|Default Gateway|DNS"

# Quick disk check
Get-PSDrive -PSProvider FileSystem | Select-Object Name, @{N='FreeGB';E={[Math]::Round($_.Free/1GB,1)}}, @{N='TotalGB';E={[Math]::Round(($_.Free+$_.Used)/1GB,1)}}

# Check top CPU processes
Get-Process | Sort-Object CPU -Descending | Select-Object -First 5 Name, Id, CPU

# Last 5 errors in event log
Get-EventLog -LogName System -EntryType Error -Newest 5 | Select-Object TimeGenerated, Source, Message

# Check if computer is domain-joined
(Get-WmiObject Win32_ComputerSystem).PartOfDomain; (Get-WmiObject Win32_ComputerSystem).Domain
```

---

## 7. Knowledge Base Management <a name="kb"></a>

### Writing Good KB Articles

**Title format:** Issue symptom + error code or context
```
BAD: "Outlook issue"
GOOD: "Outlook 365 crashes on startup — Error 0xc0000142"

BAD: "Can't print"  
GOOD: "HP LaserJet 400 not printing after Windows Update KB5005033"
```

**Article Quality Checklist:**
```
□ Title is searchable (includes error code, product name, symptom)
□ Symptoms section describes what the user experiences (not IT language)
□ Resolution steps are numbered and unambiguous
□ Screenshots included for complex UI steps
□ Resolution verified — someone other than the author tested it
□ Last reviewed date is within 6 months
□ Related articles linked
□ Tags applied (OS, product, category)
```

### KB Maintenance
```
Review triggers:
  - Software update or major OS change
  - Three tickets in 30 days marked "KB article was wrong"
  - Article not accessed in 12 months (consider archiving)

Quarterly KB audit:
  - Review top 20 most-accessed articles for accuracy
  - Review lowest-rated articles (fix or remove)
  - Identify gaps: tickets with no associated KB
```

---

## 8. Quality Assurance <a name="qa"></a>

### Ticket Quality Review

**Monthly random ticket review (10% sample):**

Score each ticket 1–5 on:
```
Category: Response Time
  5 = Responded within SLA
  3 = Slight SLA breach (<15 min over)
  1 = Major SLA breach (>1 hour over)

Category: Diagnosis Quality
  5 = Root cause identified, documented
  3 = Issue resolved without root cause documented
  1 = Issue closed without clear resolution

Category: Communication
  5 = User proactively updated, clear language, polite
  3 = Updates only when user asked
  1 = No communication until resolution

Category: Resolution Notes
  5 = Full root cause + resolution steps documented
  3 = Basic resolution noted
  1 = "Fixed" or blank
```

### User Satisfaction Surveys

**Post-resolution survey (auto-sent, 3 questions max):**
```
1. Was your issue resolved?  [Yes / No / Partially]
2. How would you rate your IT support experience? [1–5 stars]
3. Any additional comments? [Optional text]
```

**Response targets:**
- Survey completion rate: >30%
- Satisfaction score: >4.0/5.0
- Resolution confirmed: >85%

---

## 9. Help Desk Metrics & Reporting <a name="metrics"></a>

### Core KPIs

| Metric | Formula | Target |
|---|---|---|
| First Call Resolution (FCR) | Resolved at L1 / Total tickets | ≥75% |
| Mean Time to Respond (MTTR-R) | Sum of response times / ticket count | P1: <15min, P3: <2hrs |
| Mean Time to Resolve (MTTR) | Sum of resolution times / ticket count | P1: <1hr, P3: <24hrs |
| SLA Compliance | Tickets resolved within SLA / Total | ≥95% |
| Customer Satisfaction (CSAT) | Average satisfaction score | ≥4.0/5.0 |
| Ticket Volume per Agent | Total tickets / FTE | Benchmark: 8–12/day |
| Reopen Rate | Reopened tickets / Closed tickets | <5% |
| Backlog Trend | Open tickets week-over-week | Flat or declining |

### Weekly Metrics Report Template
```
HELP DESK WEEKLY REPORT — Week of [Date]

VOLUME:
  Total tickets received: [X] (vs. [X] last week, [X] last year same week)
  Tickets closed: [X]
  Open backlog: [X]

SLA PERFORMANCE:
  P1/P2 response SLA: [X]%
  Overall resolution SLA: [X]%
  Average resolution time: [X hours]

QUALITY:
  First Call Resolution: [X]%
  Customer Satisfaction: [X]/5.0
  Reopen rate: [X]%

TOP ISSUE CATEGORIES (this week):
  1. [Category] — [X] tickets ([X]%)
  2. [Category] — [X] tickets
  3. [Category] — [X] tickets

NOTABLE ITEMS:
  - [Any P1/P2 incidents, major issues, notable achievements]

TRENDS / CONCERNS:
  - [Any concerning trends requiring attention]

ACTION ITEMS:
  - [Item — Owner — Due date]
```

---

## 10. Shift Handoff Procedures <a name="handoff"></a>

### Shift Handoff Checklist

**Outgoing tech completes before end of shift:**
```
□ All open P1/P2 tickets have current status notes
□ Any tickets pending on the outgoing tech's action → reassigned or noted
□ Verbal or written briefing prepared for incoming tech
□ Any ongoing incidents documented and owner updated
□ On-call contact info confirmed for overnight/weekend
□ Escalation contacts reviewed (changes to who's on call?)
```

**Handoff Briefing Template:**
```
SHIFT HANDOFF — [Date] [Time]

OPEN P1/P2 INCIDENTS:
  [Ticket #] [System] [Issue] [Current status] [Who's working it] [Next action]

NOTABLE OPEN TICKETS:
  [Ticket #] [Brief issue] [Status] [Anything incoming needs to know]

THINGS TO WATCH:
  [Any known issues that might escalate]
  [Scheduled maintenance happening]
  [Ongoing vendor issues]

CHANGES MADE THIS SHIFT:
  [Any infrastructure changes completed]

CONTACTS:
  On-call escalation tonight: [Name] [Phone]
  Vendor tickets open: [Ticket # / description]
```
