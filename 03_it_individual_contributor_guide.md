# IT Individual Contributor (IC) — Career & Technical Excellence Guide

## Overview
Reference guide for IT professionals at the IC level — covering technical skills development, career progression, professional practices, documentation, automation, and working effectively within an IT organization.

---

## Table of Contents
1. [IC Career Levels & Expectations](#levels)
2. [Technical Skills by Domain](#skills)
3. [Professional Practices](#practices)
4. [Effective Documentation](#documentation)
5. [Automation & Scripting Fundamentals](#automation)
6. [Problem-Solving Frameworks](#problem-solving)
7. [Communication Skills for ICs](#communication)
8. [Certifications Roadmap](#certifications)
9. [On-Call & After-Hours Best Practices](#on-call)
10. [Building Your Technical Reputation](#reputation)

---

## 1. IC Career Levels & Expectations <a name="levels"></a>

### IT Career Ladder

| Level | Title Examples | Years of Experience | Key Differentiator |
|---|---|---|---|
| L1 | Help Desk Technician, Desktop Support | 0–2 | Follows procedures, resolves common issues |
| L2 | Systems Administrator, IT Support Specialist | 2–5 | Owns domains, solves complex issues |
| L3 | Senior Systems Administrator, Senior Engineer | 5–10 | Deep expertise, mentors others, leads projects |
| L4 | Principal/Staff Engineer, IT Architect | 10–15 | Designs systems, cross-team influence |
| L5 | Distinguished Engineer | 15+ | Industry-level expertise, strategic direction |

### What Separates L1 from L3

```
L1 → L2: 
  - Goes beyond "restart and see" — actually diagnoses root cause
  - Can document a runbook others can follow
  - Proactively learns; doesn't wait to be trained
  - Handles escalations without hand-holding

L2 → L3:
  - Thinks in systems, not just tasks
  - Considers security and scalability in every design
  - Automates repetitive work
  - Mentors junior team members
  - Leads projects from start to finish
  - Communicates to non-technical stakeholders
  
L3 → L4:
  - Shapes the technology strategy for their domain
  - Builds platforms and standards others use
  - Influences hiring decisions
  - Recognized expertise outside their immediate team
  - Reduces complexity, not just adds features
```

---

## 2. Technical Skills by Domain <a name="skills"></a>

### Windows Systems Administration

**Core skills every Windows admin must have:**
```
Infrastructure:
  □ Active Directory: Users, groups, OUs, GPO
  □ DNS: Zones, records, troubleshooting
  □ DHCP: Scopes, options, reservations
  □ File Services: NTFS permissions, shares, DFS
  □ Certificate Services: PKI basics, cert deployment
  □ Remote Desktop Services / Virtual Desktop

Automation:
  □ PowerShell: Scripts, pipelines, error handling, modules
  □ Task Scheduler: Automated maintenance tasks
  □ Group Policy Preferences: Software deployment, shortcuts
  
Monitoring:
  □ Windows Event Log analysis
  □ Performance Monitor / Resource Monitor
  □ Windows Admin Center
  □ SCOM or alternative monitoring
```

**PowerShell proficiency levels:**
```
Beginner: Run existing scripts, basic cmdlets, Get-Help
Intermediate: Write scripts, loops, conditionals, pipeline, error handling
Advanced: Functions, modules, classes, REST API calls, DSC
Expert: Build tools used by others, CI/CD integration, testing frameworks
```

### Linux Systems Administration

**Core skills:**
```
Shell & Commands:
  □ Bash scripting (variables, loops, functions, cron)
  □ File operations, permissions, ownership
  □ Process management (systemd, jobs, signals)
  □ Network tools (ip, ss, netstat, tcpdump, curl)
  □ Package management (apt, yum/dnf, rpm)
  □ Log analysis (journalctl, grep, awk, sed)

Services:
  □ SSH hardening and key management
  □ Apache/Nginx web server basics
  □ Samba for Windows file sharing
  □ NFS for Linux file sharing
  □ Firewall (iptables, firewalld, ufw)

Monitoring:
  □ top, htop, iotop, nload
  □ Prometheus + Grafana basics
  □ ELK or similar log platform
```

### Cloud Fundamentals (AWS/Azure)

**Azure (common in enterprise):**
```
Core Services to Know:
  □ Azure Active Directory (Entra ID)
  □ Azure Virtual Machines
  □ Azure Virtual Network (VNet, subnets, NSGs)
  □ Azure Storage (Blob, File, Queue)
  □ Azure Key Vault
  □ Azure Monitor + Log Analytics
  □ Azure Backup
  □ Microsoft 365 integration

IAM Basics:
  □ Roles (RBAC) vs. Policies
  □ Managed Identities
  □ Conditional Access
  □ Privileged Identity Management (PIM)
```

**AWS:**
```
Core Services to Know:
  □ EC2 (compute)
  □ S3 (storage)
  □ VPC (networking)
  □ IAM (identity)
  □ RDS (databases)
  □ CloudWatch (monitoring)
  □ Route 53 (DNS)
  □ Systems Manager
```

---

## 3. Professional Practices <a name="practices"></a>

### The "Leave It Better Than You Found It" Rule
Every time you touch a system:
- Fix the immediate problem AND note technical debt you observed
- If you have time, fix or document the tech debt
- Update documentation if it was wrong or outdated
- Add a monitoring check if one didn't exist

### Version Control for Everything
```bash
# Even for scripts and configs — use Git
git init
git add .
git commit -m "Initial commit: AD user provisioning script"

# Branch for changes
git checkout -b feature/add-email-notification
# Make changes
git add modified_script.ps1
git commit -m "Add email notification on user creation"
git checkout main
git merge feature/add-email-notification

# Always commit meaningful messages
# BAD: git commit -m "fix"
# GOOD: git commit -m "Fix: Handle case where user already exists in AD"
```

### Testing Your Changes
```
Before implementing ANY change:
  1. Test in non-production first (dev/staging/lab)
  2. Have a rollback plan before starting
  3. Verify your change worked (don't just assume)
  4. Watch for 15–30 minutes after for unexpected effects

Change implementation checklist:
  □ Backup/snapshot taken
  □ Rollback steps documented
  □ Maintenance window scheduled/communicated
  □ Monitoring active
  □ Colleague aware (not solo on critical changes)
  □ Testing completed in non-prod
```

### Security Mindset for ICs
```
Every day, ask:
  - Am I following least-privilege?
  - Would I be comfortable if this configuration was audited?
  - Am I storing credentials securely? (No plaintext passwords!)
  - Did I log what I changed and why?
  - Is this change auditable?

Never:
  - Store passwords in scripts as plaintext
  - Use shared admin accounts
  - Disable security controls "temporarily" (it's never temporary)
  - Grant broader access than needed "for now"
  - Work on production from a personal device
```

---

## 4. Effective Documentation <a name="documentation"></a>

### Documentation Types

**Runbooks (Operational Procedures)**
```
What: Step-by-step procedure for a specific task
When: Any repeatable task performed more than once
Format:
  - Title: Clear action (e.g., "Add User to Active Directory")
  - Purpose: What this achieves
  - Prerequisites: Access, tools, knowledge required
  - Steps: Numbered, specific, verifiable
  - Expected output: What success looks like
  - Troubleshooting: Common errors and fixes
  - Related: Links to related runbooks
```

**Architecture Documentation**
```
What: How systems are designed and why
When: New systems, significant changes
Format:
  - System overview and purpose
  - Architecture diagram (draw.io, Visio, Lucidchart)
  - Component list (servers, services, dependencies)
  - Data flows
  - Security controls
  - Disaster recovery approach
  - Known limitations / tech debt
  - Maintenance notes
```

**Runbook Template:**
```markdown
# Runbook: [Task Name]

**Last Updated:** YYYY-MM-DD  
**Author:** [Name]  
**Reviewed By:** [Name]  

## Purpose
[One sentence: what does this runbook accomplish?]

## When to Use
[Under what circumstances is this procedure used?]

## Prerequisites
- Access: [Required permissions/credentials]
- Tools: [Required software/access]
- Knowledge: [What you should know before attempting this]

## Steps

### Step 1: [Action]
```command or action```
Expected output: [What you should see]

### Step 2: [Action]
...

## Verification
[How do you confirm the task was completed successfully?]

## Rollback
[Steps to undo this task if needed]

## Troubleshooting
| Error | Cause | Fix |
|-------|-------|-----|
|       |       |     |

## Related Runbooks
- [Link to related runbook]
```

### Knowledge Base Article Template
```markdown
# [Article Title — Searchable, Problem-Statement Format]
e.g., "User Cannot Access Network Share: Access Denied Error"

## Symptoms
- [Exact error message]
- [User experience: what they observe]
- [Affected scope: all users or specific users]

## Cause
[Explanation of why this happens]

## Resolution

### Option 1: [Most common fix]
[Steps]

### Option 2: [Alternative fix]
[Steps]

## Prevention
[How to avoid this issue in future]

## Related Issues
- [Link to related KB article]
```

---

## 5. Automation & Scripting Fundamentals <a name="automation"></a>

### PowerShell Best Practices

```powershell
#Requires -Modules ActiveDirectory
<#
.SYNOPSIS
    Brief one-line description

.DESCRIPTION
    Detailed description of what the script does

.PARAMETER Username
    The AD username to process

.EXAMPLE
    .\New-UserAccount.ps1 -Username jsmith -Department IT

.NOTES
    Author: [Name]
    Date: [Date]
    Version: 1.0
    Change Log:
      1.0 - Initial version
#>

[CmdletBinding(SupportsShouldProcess)]
param(
    [Parameter(Mandatory)]
    [string]$Username,
    
    [Parameter(Mandatory)]
    [string]$Department
)

# --- Always use strict mode and error handling ---
Set-StrictMode -Version Latest
$ErrorActionPreference = 'Stop'

# --- Logging function ---
function Write-Log {
    param([string]$Message, [string]$Level = "INFO")
    $timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
    "$timestamp [$Level] $Message" | Tee-Object -FilePath "$PSScriptRoot\script.log" -Append
}

# --- Main logic ---
try {
    Write-Log "Starting user creation for $Username"
    
    if (-not (Get-ADUser -Filter {SamAccountName -eq $Username} -ErrorAction SilentlyContinue)) {
        if ($PSCmdlet.ShouldProcess($Username, "Create AD user")) {
            # Create user...
            Write-Log "Successfully created user $Username"
        }
    } else {
        Write-Log "User $Username already exists" -Level "WARN"
    }
}
catch {
    Write-Log "ERROR: $($_.Exception.Message)" -Level "ERROR"
    throw
}
```

### Bash Scripting Best Practices

```bash
#!/usr/bin/env bash
# Script: backup_configs.sh
# Purpose: Backup configuration files for key services
# Author: [Name]
# Date: [Date]

# --- Safety settings ---
set -euo pipefail   # Exit on error, unset vars, pipe failures
IFS=$'\n\t'         # Safer word splitting

# --- Configuration ---
BACKUP_DIR="/backups/configs"
DATE=$(date +%Y%m%d_%H%M%S)
LOG_FILE="/var/log/backup_configs.log"

# --- Logging ---
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"
}

# --- Error handling ---
trap 'log "ERROR: Script failed at line $LINENO"' ERR

# --- Functions ---
backup_file() {
    local source="$1"
    local dest="$BACKUP_DIR/$(basename $source).$DATE"
    
    if [[ -f "$source" ]]; then
        cp "$source" "$dest"
        log "Backed up: $source → $dest"
    else
        log "WARNING: File not found: $source"
    fi
}

# --- Main ---
log "Starting config backup"
mkdir -p "$BACKUP_DIR"

backup_file "/etc/nginx/nginx.conf"
backup_file "/etc/ssh/sshd_config"
backup_file "/etc/hosts"

log "Backup complete"
```

### Automating Common Tasks

```powershell
# --- Example: Daily stale user report emailed to manager ---

$staleUsers = Get-ADUser -Filter {
    LastLogonDate -lt (Get-Date).AddDays(-90) -and 
    Enabled -eq $true
} -Properties LastLogonDate, Department, Manager |
Select-Object SamAccountName, DisplayName, Department, LastLogonDate

$htmlTable = $staleUsers | ConvertTo-Html -As Table -Fragment

$emailParams = @{
    To         = "itmanager@company.com"
    From       = "it-alerts@company.com"
    SmtpServer = "smtp.company.com"
    Subject    = "Daily Report: Stale User Accounts - $(Get-Date -Format 'yyyy-MM-dd')"
    Body       = "<html><body><h2>Stale Users (>90 days no login)</h2>$htmlTable</body></html>"
    BodyAsHtml = $true
}

Send-MailMessage @emailParams
```

---

## 6. Problem-Solving Frameworks <a name="problem-solving"></a>

### The OODA Loop (Observe, Orient, Decide, Act)
```
OBSERVE: Gather facts (logs, metrics, user reports)
ORIENT: Context (what changed? what's the pattern?)
DECIDE: Choose the most likely hypothesis to test
ACT: Implement and observe results

Repeat until resolved.
```

### Rubber Duck Debugging
When stuck: explain the problem out loud (to a duck, colleague, or in writing). 
The act of articulating the problem often reveals the solution.

### Divide and Conquer
```
When a complex system isn't working:
1. Find the midpoint of the data/request flow
2. Test — is it working at the midpoint?
3. If YES: problem is in the second half → test midpoint of second half
4. If NO: problem is in the first half → test midpoint of first half
5. Repeat until you isolate the failure point
```

---

## 7. Communication Skills for ICs <a name="communication"></a>

### Writing Effective Tickets & Incident Notes

**BAD ticket note:**
```
"Looked at the server. Seems fine now. Closed."
```

**GOOD ticket note:**
```
"Investigated CPU spike on APPSERVER01 (Event ID 4625 in Application log).
Found IIS application pool 'FinanceApp' consuming 98% CPU since 14:30.
Recycled the application pool at 15:45. CPU returned to normal (12%).
Root cause: Memory leak in FinanceApp v2.3.1 — notified application team 
(ticket #45231). Monitoring for 2 hours before closing.
Resolution confirmed by user at 16:10."
```

### Giving Technical Updates to Non-Technical Stakeholders

**Framework: So What / Now What**
```
Instead of: "The RAID controller on the NAS failed and we're degraded."
Say:
  "We have a storage hardware issue [WHAT].
   This means file access for the Finance team may be slower than normal [SO WHAT].
   We're replacing the failed component tonight — expected to be fully resolved 
   by 8am tomorrow [NOW WHAT]."
```

### Asking for Help Effectively
```
When asking for help (from a colleague, manager, or Stack Overflow), include:
  1. What you're trying to accomplish
  2. What you've already tried
  3. What you observed (exact errors, not "it doesn't work")
  4. What your current hypothesis is

Example: "I'm trying to get users to authenticate to the new SharePoint site via SSO.
I've configured the SAML app in Azure AD, but users get a 403 after redirect.
I've verified the claim rules and the SP metadata is correct.
I think it might be a role assignment issue — the app registration in Azure may 
be missing the required API permissions. Can you review my SAML trace?"
```

---

## 8. Certifications Roadmap <a name="certifications"></a>

### Certification Paths by Role

**Help Desk / General IT:**
```
Entry:
  CompTIA A+ → CompTIA Network+ → CompTIA Security+
  
Next steps (specialize):
  Microsoft: MS-900 (M365 Fundamentals) → MD-102 (Endpoint Administrator)
  Azure: AZ-900 (Fundamentals) → AZ-104 (Administrator)
```

**Systems Administrator:**
```
Microsoft Path:
  AZ-104: Azure Administrator Associate
  MS-102: M365 Administrator Expert
  AZ-305: Azure Solutions Architect Expert (advanced)

Linux Path:
  CompTIA Linux+ → LPIC-1 → LPIC-2 → RHCSA → RHCE

VMware:
  VCP-DCV (vSphere) → VCAP-DCV
```

**Network Engineer:**
```
Cisco Path:
  CCNA → CCNP Enterprise → CCIE Enterprise
  
Palo Alto:
  PCNSA → PCNSE

Cloud Networking:
  AWS Advanced Networking / Azure Network Engineer Associate
```

**Security:**
```
Foundational:
  CompTIA Security+ → CompTIA CySA+ or eJPT

Intermediate:
  CEH → CompTIA CASP+ → OSCP (offensive)
  CISSP (management/architecture track)

Specialized:
  GIAC (GSEC, GCIH, GPEN, etc.) — highly respected
  AWS Security Specialty / Azure Security Engineer
```

**Cloud / DevOps:**
```
Cloud:
  AWS: SAA-C03 (Solutions Architect) → SAP-C02 (Professional) → Specialty
  Azure: AZ-104 → AZ-305 → specialty (AZ-500, DP-203, etc.)
  GCP: Associate Cloud Engineer → Professional Cloud Architect

DevOps:
  HashiCorp Terraform Associate → Professional
  Kubernetes: CKA (Administrator) → CKAD (Developer) → CKS (Security)
  Docker: Docker Certified Associate
```

---

## 9. On-Call & After-Hours Best Practices <a name="on-call"></a>

### On-Call Mindset
```
On-call is a responsibility, not a punishment. Approach it professionally:
  - Be ready to respond within the agreed SLA (typically 15–30 min)
  - Have your tools accessible (VPN, laptop, credentials)
  - Know who to escalate to and how to reach them
  - Don't guess on production changes at 2am — escalate if unsure
  - Document everything you do, even at 3am
```

### On-Call Response Protocol
```
1. Acknowledge the alert within SLA
2. Gather facts before touching anything
3. Identify severity — is this truly P1 or lower?
4. If P1: Wake up your manager (they expect it and prefer it to finding out in the morning)
5. Implement only well-understood fixes at night
6. Uncertain? Implement a workaround/mitigation, escalate to next shift for root cause
7. Document every command you ran
8. Write the incident ticket before you go back to sleep
```

### Handling On-Call Fatigue
- Track on-call burden per person — distribute fairly
- Post-on-call day off (where policy allows) after a busy rotation
- Reduce alert noise — every alert should be actionable
- Conduct on-call retrospectives: "Why were we paged 15 times last week?"
- Automate common fixes to reduce manual intervention

---

## 10. Building Your Technical Reputation <a name="reputation"></a>

### Be the Person Who Documents Things
Most engineers resist documentation. The ones who document become invaluable:
- Write the runbook after you solve something for the first time
- Update documentation when you find it wrong
- Build a personal knowledge base (Obsidian, Notion, Confluence)

### Build a Lab Environment
```
Why: Practice without breaking production
Options:
  - Physical: Old hardware, home lab servers
  - Virtual: VMware Workstation, Hyper-V, VirtualBox
  - Cloud: AWS/Azure free tier (minimal cost for basic labs)
  - Proxmox: Free, powerful hypervisor for home labs

What to run:
  - Windows Server with AD/DNS/DHCP
  - Linux server (Ubuntu Server or RHEL-based)
  - pfSense/OPNsense for networking practice
  - Kubernetes cluster (k3s for lightweight)
  - Monitoring stack (Grafana, Prometheus)
```

### Contribute Beyond Your Job Description
- Volunteer to document a process no one else wants to
- Mentor junior team members
- Present a technical topic in a lunch-and-learn
- Propose a process improvement (with data)
- Write an internal blog post about something you solved

### Stay Current
- Follow relevant subreddits: r/sysadmin, r/networking, r/netsec
- Attend local user groups or virtual meetups (VMUG, CISSP Chapter, etc.)
- Read: The Practical Sysadmin, Ars Technica, Krebs on Security, AWS/Azure blogs
- Set up a personal lab and break things intentionally
- Contribute to open-source tools you use
