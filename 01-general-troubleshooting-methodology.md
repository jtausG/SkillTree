# General Troubleshooting Methodology

> **Category:** Troubleshooting | **Audience:** All IT Staff | **Level:** Foundational

---

## Table of Contents
1. [The Troubleshooting Mindset](#1-the-troubleshooting-mindset)
2. [Structured Troubleshooting Framework](#2-structured-troubleshooting-framework)
3. [OSI Model as a Diagnostic Tool](#3-osi-model-as-a-diagnostic-tool)
4. [Information Gathering Techniques](#4-information-gathering-techniques)
5. [Hypothesis & Testing Methodology](#5-hypothesis--testing-methodology)
6. [Escalation Decision Framework](#6-escalation-decision-framework)
7. [Documentation Standards](#7-documentation-standards)
8. [Common Cognitive Traps](#8-common-cognitive-traps)
9. [Post-Resolution Steps](#9-post-resolution-steps)

---

## 1. The Troubleshooting Mindset

Effective troubleshooting is a disciplined, scientific process — not guessing. The goal is to **identify the root cause**, not just restore service (though service restoration is an urgent priority).

### Core Principles
- **Reproduce before you fix.** Understand the exact conditions that trigger the issue.
- **Change one variable at a time.** Multiple simultaneous changes invalidate your diagnostic data.
- **Document everything as you go.** Notes taken during the incident are exponentially more valuable than memory.
- **Don't assume — verify.** "I think it's the firewall" must become "I have confirmed X packets are being dropped at Y rule."
- **Consider what changed.** The vast majority of issues are caused by a recent change. Ask first.

---

## 2. Structured Troubleshooting Framework

### The DETECT → ISOLATE → FIX → VERIFY → DOCUMENT cycle

```
1. DETECT       Identify that a problem exists and understand its scope
2. DEFINE       Clearly state the problem (what works vs. what doesn't)
3. GATHER       Collect logs, symptoms, affected systems, timeline
4. HYPOTHESIZE  Form a ranked list of possible causes
5. TEST         Test the most likely hypothesis with minimum disruption
6. ISOLATE      Narrow down to the root cause
7. FIX          Apply the solution
8. VERIFY       Confirm the fix resolves the issue without introducing new problems
9. DOCUMENT     Record root cause, fix, and prevention steps
10. PREVENT     Implement monitoring or process changes to prevent recurrence
```

---

## 3. OSI Model as a Diagnostic Tool

Use the OSI model to guide network-related troubleshooting. Work from Layer 1 up (physical → application) for most issues, or top-down when application-specific issues are suspected.

| Layer | Name | Components to Check | Tools |
|-------|------|---------------------|-------|
| 7 | Application | App errors, APIs, authentication | Browser DevTools, app logs, curl |
| 6 | Presentation | SSL/TLS certificates, encoding | `openssl s_client`, Wireshark |
| 5 | Session | Sessions, NetBIOS, RPC, NFS | `netstat`, `ss`, session logs |
| 4 | Transport | TCP/UDP ports, firewall rules | `netstat`, `telnet`, `nmap`, `tracert` |
| 3 | Network | IP addressing, routing, ICMP | `ping`, `traceroute`, `route print` |
| 2 | Data Link | MAC addresses, ARP, VLANs, switches | `arp -a`, switch port logs |
| 1 | Physical | Cables, NICs, SFPs, power, LEDs | Physical inspection, cable tester |

### Bottom-Up Checklist
- [ ] Is the cable connected? Are LEDs lit?
- [ ] Does `ipconfig`/`ip addr` show a valid IP?
- [ ] Can you ping the default gateway?
- [ ] Can you ping a public IP (8.8.8.8)?
- [ ] Does DNS resolve (`nslookup google.com`)?
- [ ] Can you reach the service port (`telnet host 443`)?
- [ ] Does the application respond?

---

## 4. Information Gathering Techniques

### Initial Questions (Ask Every Time)
1. **What exactly is the problem?** (Get specific — not "email is broken," but "Outlook shows error 0x80040115 when sending to external addresses")
2. **When did it start?** (Exact time, or "it was working at X, broken by Y")
3. **What changed recently?** (Updates, config changes, new software, network changes)
4. **Who / what is affected?** (One user? One office? Everyone? Specific service?)
5. **Is it consistent or intermittent?** (Always fails, or fails 30% of the time?)
6. **What has already been tried?** (Prevent duplicating effort; some "fixes" create new problems)
7. **What is the business impact?** (Drives urgency)

### Evidence Collection Checklist
- [ ] Screenshot of error messages (exact text — copy/paste preferred over paraphrasing)
- [ ] Event Viewer logs (System, Application, Security) — export relevant entries
- [ ] Application logs (`%AppData%`, `/var/log/`, application-specific paths)
- [ ] Network captures (Wireshark `.pcapng`) if network-related
- [ ] System performance counters (CPU, RAM, disk I/O at time of failure)
- [ ] DNS query logs
- [ ] Firewall/proxy logs
- [ ] DHCP lease table
- [ ] Recent change records

### Key Windows Log Locations
```
Event Viewer:   eventvwr.msc → Windows Logs
System Logs:    C:\Windows\System32\winevt\Logs\
App Logs:       %AppData%\Local\Temp\ or application-specific
MSI Logs:       C:\Windows\Temp\*.log
IIS Logs:       C:\inetpub\logs\LogFiles\
DHCP Logs:      C:\Windows\System32\dhcp\
DNS Logs:       C:\Windows\System32\dns\
```

### Key Linux Log Locations
```
Syslog:         /var/log/syslog or /var/log/messages
Auth:           /var/log/auth.log or /var/log/secure
Kernel:         /var/log/kern.log | dmesg
App-specific:   /var/log/<application>/
Journal:        journalctl -xe
```

---

## 5. Hypothesis & Testing Methodology

### Ranking Hypotheses
After gathering information, rank potential causes by:
1. **Probability** — What causes this type of issue most often?
2. **Recency** — Was there a recent change that aligns with the symptom onset?
3. **Scope alignment** — Does the scope of impact (one user vs. all users) narrow the cause?
4. **Evidence** — What does collected data point to?

### Testing Safely
- **Test in a non-production environment first** when possible
- **Create a rollback plan** before making any change
- **Test during low-impact windows** for production changes
- **Verify only one change at a time** — concurrent changes make it impossible to know what fixed it

### Divide and Conquer
For complex issues, use binary elimination:
- Split the system in half (e.g., if the problem occurs in Office apps, test one Office app)
- Eliminate half the hypothesis space with each test
- Example: "Is this Windows or the app?" → "Is this a permissions issue or a config issue?" → etc.

---

## 6. Escalation Decision Framework

### Escalate When:
- You have exhausted your available diagnostic steps without resolution
- The issue requires access or permissions you don't have
- Resolution requires a change outside your authorization level
- Business impact is high and resolution will exceed your SLA
- The issue involves security or data breach indicators
- Specialized expertise is needed (storage, network core, AD, security)

### Escalation Information Package (always include)
```markdown
## Escalation Summary — [Ticket #] — [Date/Time]

**Problem Statement:**
Clear 1-2 sentence description of what is failing.

**Impact:**
X users/systems affected. Business impact: [describe]

**Timeline:**
- [Time]: Issue first reported
- [Time]: Initial investigation started
- [Time]: Specific steps taken and findings

**Symptoms:**
- Exact error messages (copy/paste)
- Affected systems/users
- Systems NOT affected (helps isolate)

**Steps Already Taken:**
1. Action → Result
2. Action → Result

**Current Hypothesis:**
Most likely cause based on evidence: [describe]
Why other causes were ruled out: [describe]

**Logs/Evidence Attached:**
- event_log_export.evtx
- network_capture.pcapng

**Urgency:** P1 / P2 / P3 — SLA expires at [time]
**Escalating to:** [Name/Team]
**Requesting:** [Specific help needed]
```

### Escalation Matrix
| Issue Type | L1 Handles | Escalate to L2 | Escalate to L3 | Escalate to Vendor |
|-----------|-----------|---------------|----------------|-------------------|
| Password reset | ✅ | | | |
| Workstation rebuild | ✅ | | | |
| Network connectivity (single user) | ✅ | | | |
| Network connectivity (site-wide) | | ✅ | | |
| Server down | | ✅ | | |
| AD replication failure | | | ✅ | |
| Firewall/core network | | | ✅ | |
| Hardware failure (under warranty) | | ✅ | | ✅ |
| SaaS platform outage | ✅ (status page) | | | ✅ |
| Security incident | | | ✅ Security team | |

---

## 7. Documentation Standards

### During the Incident
- Keep a running timestamped log of every action taken and its result
- Note commands run and exact output (copy/paste > paraphrase)
- Record who you spoke to and what they said
- Save all collected logs with timestamps in the ticket

### Ticket Notes Format
```
[HH:MM] Action taken: {what you did}
[HH:MM] Result: {exact output or observation}
[HH:MM] Interpretation: {what this tells you}
[HH:MM] Next step: {what you'll try next}
```

### Root Cause Analysis (RCA) Template
```markdown
**Incident:** [Title]
**Date/Time:** 
**Duration:**
**Severity:** P1/P2/P3
**Affected Systems:**

**Timeline of Events:**
| Time | Event |
|------|-------|

**Root Cause:**
Technical description of what failed and why.

**Contributing Factors:**
- Factor 1
- Factor 2

**Resolution:**
What was done to restore service.

**Prevention Actions:**
| Action | Owner | Due Date | Status |
|--------|-------|----------|--------|

**Lessons Learned:**
What would we do differently?
```

---

## 8. Common Cognitive Traps

### Confirmation Bias
Tendency to only look for evidence that confirms your initial hypothesis. Counter: Actively try to **disprove** your hypothesis.

### Einstellung Effect
Applying a previous solution to a similar-looking (but different) problem. Counter: Fully define the new problem independently before reaching for old solutions.

### Tunnel Vision
Fixating on one area while missing the actual cause in an unrelated area. Counter: Step back, review scope, ask "what am I not looking at?"

### Solutionitis
Applying fixes before fully diagnosing. Counter: Require a written hypothesis before making any change.

### Over-confidence in Vendor Statements
"Vendor says their system is fine." Counter: Independently verify with your own logs and data.

---

## 9. Post-Resolution Steps

### Immediately After Resolution
- [ ] Update ticket with root cause and resolution steps
- [ ] Notify affected users/stakeholders of resolution
- [ ] Verify the fix is stable (monitor for 30–60 minutes for critical issues)
- [ ] Remove any temporary workarounds and replace with permanent fixes
- [ ] Update monitoring/alerting if a gap was identified
- [ ] Check if the same issue could affect other systems

### Within 24 Hours
- [ ] Write a formal RCA for P1/P2 incidents
- [ ] Update runbook/knowledge base if the fix wasn't already documented
- [ ] Create a change request if a permanent configuration change was made
- [ ] Schedule follow-up review if the root cause isn't 100% confirmed

### Long-Term
- [ ] Review recurrence metrics in ticketing system
- [ ] Consider proactive measures (patching, monitoring, process change)
- [ ] Share learnings with the team in a post-mortem or standup

---

## Quick Reference Command Card

### Windows
```powershell
# Network diagnostics
ipconfig /all
ipconfig /flushdns
nslookup <hostname> <dns-server>
ping -n 10 <host>
tracert <host>
pathping <host>
netstat -ano
Test-NetConnection -ComputerName <host> -Port 443

# System info
systeminfo
Get-EventLog -LogName System -Newest 50 | Where-Object {$_.EntryType -eq "Error"}
Get-Process | Sort-Object CPU -Descending | Select-Object -First 10
```

### Linux / macOS
```bash
# Network diagnostics
ip addr show
cat /etc/resolv.conf
dig @8.8.8.8 hostname
ping -c 10 host
traceroute host
ss -tulnp
curl -v https://host:443

# System info
top / htop
df -h
dmesg | tail -50
journalctl -xe --since "1 hour ago"
```

---

*See also: [Network Connectivity Troubleshooting](04-network-connectivity-troubleshooting.md) | [Server Troubleshooting](09-server-troubleshooting.md)*
