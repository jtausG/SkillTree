# IT Onboarding & Offboarding Procedures

## Overview
Detailed procedures for IT onboarding of new employees and offboarding of departing employees. Covers all user types including standard employees, contractors, executives, and privileged users.

---

## 1. Onboarding Overview

### Onboarding Stakeholders
| Role | Responsibility |
|------|---------------|
| HR | Initiates IT request, provides start date, role, location |
| Hiring Manager | Approves system access, provides department-specific needs |
| IT Helpdesk | Standard provisioning tasks |
| IT Security | Privileged access, security exceptions |
| IT Operations | Device preparation, network access |
| Facilities | Badge access (if separate from IT) |

### Onboarding Request Lead Times
| User Type | IT Request Lead Time | Notes |
|-----------|--------------------|----|
| Standard Employee | 5 business days | Most provisioning automated |
| Executive/VP+ | 7 business days | Special device config, dedicated support |
| IT/Admin Staff | 7 business days | Additional access review required |
| Contractor (off-site) | 3 business days | Typically limited access |
| Contractor (on-site) | 5 business days | May need device |
| Intern | 5 business days | Restricted access profile |

---

## 2. Standard Employee Onboarding

### Phase 1: Pre-Arrival (T-5 to T-1 business days)

**IT Actions upon receiving HR request:**

**Identity Provisioning:**
```
1. Create Active Directory account
   - Username format: first initial + last name (jsmith)
   - UPN: jsmith@company.com
   - Display name: Jane Smith
   - OU: OU=<Department>,OU=Users,DC=corp,DC=com
   - Password: Generated temporary (force change at login)
   - Account enabled: YES
   - Manager: Set per HR request

2. Create M365 account (or sync via AAD Connect)
   - Assign license: M365 E3 (or per role)
   - Mailbox auto-created upon license assignment

3. Add to security groups:
   - All-Employees
   - <Department>-Staff
   - VPN-Users (if applicable)
   - Any application-specific groups

4. Add to distribution lists:
   - All-Staff@company.com
   - <Department>@company.com

5. Configure MFA:
   - Create account with temporary password
   - User self-registers MFA on Day 1
   - Conditional Access enforces MFA registration

6. Set up shared mailbox access (if needed):
   - Add to department shared mailbox

7. Configure file share / SharePoint access:
   - Department SharePoint site: Member
   - Shared drive: per department standard

8. Provision applications:
   - HRIS: auto-provisioned via SCIM
   - Expense system: manual request to Finance
   - Other apps: per role
```

**Device Preparation:**
```
1. Assign hardware from inventory OR order new (check lead time)
2. Device imaging/provisioning:
   - Windows Autopilot: Register hardware hash if new
   - Standard image via SCCM if existing device
   - Configure Intune profile assignment (user group)
3. Apply asset tag, update CMDB
4. Pre-install department-specific software
5. Confirm device is fully patched
6. Package device for delivery (remote) or stage at desk (office)
```

**Physical Access:**
```
1. Request badge access from Facilities (if IT owns this):
   - Building access
   - Server room: only if IT role requires
   - Floor access matching department
2. Parking access if applicable
```

### Phase 2: Day 1 Setup

**Welcome Package / Email to User:**
```
Subject: Your IT Access — Welcome to [Company]!

Hi [Name],

Welcome! Below is everything you need to get started.

COMPUTER LOGIN:
Username: jsmith@company.com
Temporary Password: [TempPass123!]
→ You'll be prompted to change your password on first login.

EMAIL: jsmith@company.com
→ Accessible via Outlook desktop app or https://outlook.office.com

MULTI-FACTOR AUTHENTICATION (MFA):
→ After logging in, you'll be prompted to set up MFA. Please do this before closing the browser. Use the Microsoft Authenticator app on your mobile phone.

VPN ACCESS: [If applicable]
→ Install the Cisco AnyConnect / GlobalProtect / etc. client
→ Server: vpn.company.com | Use your company credentials + MFA

HELPFUL LINKS:
• IT Help Desk: helpdesk@company.com | Ext. 1234
• Self-service portal: https://helpdesk.company.com
• Password reset: https://aka.ms/sspr
• IT Knowledge Base: https://kb.company.com

Your IT support contact for onboarding: [Name], [phone/email]

Questions? Don't hesitate to reach out.
IT Operations Team
```

**Day 1 Checklist — IT Verification:**
```
ONBOARDING VERIFICATION CHECKLIST
Employee: [Name] | Start Date: [Date] | Ticket: [#]

IDENTITY:
[ ] AD account created and enabled
[ ] UPN matches email address
[ ] Manager attribute set
[ ] Department/title attributes populated
[ ] M365 license assigned, mailbox provisioned
[ ] MFA registration completed
[ ] Password changed from temporary

DEVICE:
[ ] Device assigned and documented in CMDB
[ ] Device boots and logs in successfully
[ ] Autopilot enrollment completed (if applicable)
[ ] Intune compliance: COMPLIANT
[ ] BitLocker enabled and recovery key stored
[ ] Software deployed: Office 365, VPN client, AV

ACCESS:
[ ] Email accessible (send test)
[ ] Teams/Slack access working
[ ] File share / SharePoint accessible
[ ] VPN connection tested (if remote)
[ ] Required applications accessible

DOCUMENTATION:
[ ] Ticket updated with all accounts created
[ ] Asset tag documented and linked to user in CMDB
[ ] IT onboarding checklist signed (if required)
```

---

## 3. Elevated/Privileged User Onboarding

### IT Staff Onboarding (Additional Steps)
```
ADDITIONAL PROVISIONING — IT STAFF:

Admin Accounts:
[ ] Create separate admin account (adm-jsmith@company.com)
    - Used ONLY for administrative tasks
    - No email, no browsing from admin account
    - PAM solution enrollment (CyberArk, BeyondTrust, etc.)

Server/Infrastructure Access:
[ ] RDP access to jump server
[ ] Requested server OU admin rights (minimum necessary)
[ ] SQL access (if DBA role)
[ ] Network device access (if network role)

Security Tools:
[ ] SIEM console access
[ ] Vulnerability scanner access
[ ] MDM admin console access
[ ] Firewall console access

Cloud:
[ ] Azure/AWS console access
    - Entra ID admin role (minimum necessary)
    - Tagged in IAM for access reviews

Documentation:
[ ] Review and acknowledge: Acceptable Use Policy
[ ] Review and acknowledge: Privileged Access Policy
[ ] Complete: Privileged Access Training (if required)
[ ] Emergency access procedure briefing
```

### Executive Onboarding
```
EXECUTIVE/VIP ONBOARDING:

White-Glove Service:
[ ] Dedicated IT contact assigned
[ ] Executive receives direct calendar invite for setup session
[ ] Device pre-configured and personally delivered
[ ] In-person (or video call) orientation — DO NOT just hand off

Executive-Specific:
[ ] Executive mobile device enrolled
[ ] Personal assistant given delegate access (if requested)
[ ] Executive's assistant set up with Send As / Full Access
[ ] External email signature configured
[ ] Profile photo uploaded to M365
[ ] Executive in VIP group (ensures expedited support SLA)

Security:
[ ] MFA confirmed working on all devices
[ ] Phishing-resistant MFA recommended (FIDO2/passkey)
[ ] Brief security awareness conversation
[ ] Emergency IT contact card provided
```

---

## 4. Offboarding Overview

### Offboarding Priority — Time-Sensitive
Offboarding is **CRITICAL** — delays create security risk.

| Trigger | Target Action Time |
|---------|-------------------|
| Voluntary resignation (advance notice) | Access revoked at stated last day EOD |
| Involuntary termination | Access revoked immediately (before or during HR meeting) |
| Leave of absence | Access suspended, device may be collected |
| Contract end | Access revoked at contract end date |
| Termination for cause | Immediate revocation, escalated procedure |

---

## 5. Standard Employee Offboarding

### Phase 1: Pre-Departure Preparation

**When IT receives HR notification:**
```
1. Create offboarding ticket, link to HR ticket
2. Note last day, manager, user type
3. Schedule all tasks to execute on/before last day
4. Notify: Helpdesk, Security (for sensitive roles), Facilities
```

### Phase 2: Last Day Execution

**Identity Revocation (execute in this order):**
```powershell
$userUPN = "jsmith@company.com"
$samAccount = "jsmith"

# 1. Block M365 / Azure AD sign-in (FIRST priority)
Update-MgUser -UserId $userUPN -AccountEnabled $false
Write-Host "Step 1: Azure AD sign-in blocked"

# 2. Revoke all active sessions (M365, OAuth tokens)
Revoke-MgUserSignInSession -UserId $userUPN
Write-Host "Step 2: Sessions revoked"

# 3. Disable Active Directory account
Disable-ADAccount -Identity $samAccount
Write-Host "Step 3: AD account disabled"

# 4. Move to disabled OU
$disabledOU = "OU=Disabled,OU=Users,DC=corp,DC=com"
Get-ADUser -Identity $samAccount | Move-ADObject -TargetPath $disabledOU
Write-Host "Step 4: Account moved to Disabled OU"

# 5. Remove from all security/distribution groups
$user = Get-ADUser -Identity $samAccount
$groups = Get-ADPrincipalGroupMembership -Identity $samAccount |
    Where {$_.Name -ne "Domain Users"}
foreach ($group in $groups) {
    Remove-ADGroupMember -Identity $group -Members $samAccount -Confirm:$false
    Write-Host "Removed from group: $($group.Name)"
}

# 6. Set description with termination date
Set-ADUser -Identity $samAccount -Description "TERMINATED: $(Get-Date -Format 'yyyy-MM-dd') - Manager: $manager"

# 7. Hide from Global Address List
Set-ADUser -Identity $samAccount -Add @{msExchHideFromAddressLists=$true}
```

**Email Handling:**
```powershell
# Option A: Forward email to manager for N days
Set-Mailbox $userUPN -ForwardingSmtpAddress "manager@company.com" -DeliverToMailboxAndForward $false

# Option B: Auto-reply informing senders
Set-MailboxAutoReplyConfiguration -Identity $userUPN `
    -AutoReplyState Enabled `
    -ExternalMessage "This person is no longer with [Company]. Please contact [dept email]." `
    -InternalMessage "This person is no longer with [Company]. Please contact [manager]."

# Option C: Convert to shared mailbox (preserves email, no license needed)
Set-Mailbox $userUPN -Type Shared

# Revoke license AFTER converting to shared mailbox
# (Shared mailbox < 50 GB doesn't need a license)
```

**Application Access Revocation:**
```
FOR EACH APPLICATION IN USER'S PROFILE:
[ ] HRIS: Auto-provisioning should auto-revoke
[ ] Salesforce: Deactivate user
[ ] GitHub/Azure DevOps: Remove from org
[ ] AWS/Azure: Remove IAM user / Entra ID role assignments
[ ] VPN: Revoke certificate / remove from VPN group
[ ] Password manager: Remove from company vault
[ ] Monitoring tools: Remove account
[ ] Vendor portals: Notify vendors of account removal
[ ] Physical tokens/hardware: Collect YubiKey, RSA token etc.
```

**Device Recovery:**
```
Remote employee:
[ ] Intune: Remote wipe device (Windows/Mac/Mobile)
[ ] OR: Send prepaid return shipping label
[ ] Update CMDB: Device status = "In Transit" → "Received" → "Wiped" → "Available"

On-site employee:
[ ] Collect device on last day
[ ] Collect all peripherals (charger, docking station, etc.)
[ ] Update CMDB immediately
[ ] Wipe and reimage device within 3 business days

Mobile device:
[ ] Intune: Remote wipe (if company-owned) OR Selective wipe (BYOD)
[ ] iCloud Activation Lock removal if company-owned Apple device
```

**Physical Access:**
```
[ ] Deactivate badge (Facilities or IT depending on org)
[ ] Collect physical keys
[ ] Collect access cards
[ ] Remove parking access
[ ] Update visitors/reception of departure (VIP or sensitive roles)
```

### Phase 3: Post-Departure (30-Day Tasks)
```
[ ] Confirm all access is revoked (run access audit for former user)
[ ] Delete or archive home drive files (per retention policy)
[ ] Remove personal data (per privacy policy / GDPR)
[ ] Confirm email forwarding disabled at 30 days (or per policy)
[ ] Delete M365 account (or retain per eDiscovery hold)
    - Note: Deleted M365 accounts retained 30 days in Recycle Bin
[ ] Release software licenses back to pool
[ ] Release phone number for reuse
[ ] Confirm CMDB updated: user → device unlinked
[ ] Close offboarding ticket with completion notes
```

---

## 6. Involuntary Termination Protocol

### Pre-Termination Preparation
Work with HR to prepare revocation BEFORE the HR meeting:
```
48-72 hours before termination meeting (confidential):
[ ] Pre-stage all revocation steps (test in staging if available)
[ ] Identify all system access for the user
[ ] Brief Security team if user has privileged access
[ ] Prepare device recovery plan (desk location, remote ship, etc.)
[ ] Prepare manager account access (manager may need access to files)
[ ] Identify any critical projects/work that needs knowledge transfer
```

### Day-Of Execution
```
T=0 (HR meeting begins):
  IT must execute steps in REAL TIME as HR is notifying the employee

MINUTE 0: Block Azure AD / M365 sign-in (most critical — cloud access)
MINUTE 1: Revoke all sessions (Revoke-MgUserSignInSession)
MINUTE 2: Disable AD account
MINUTE 5: Revoke VPN access
MINUTE 10: Change any shared/service account passwords the user knew
MINUTE 30: Remove from all groups and applications

Post-meeting:
[ ] Escort to collect personal items (do NOT leave unescorted)
[ ] Collect badge, keys, access cards, company device
[ ] Confirm device is retrieved or remotely wiped
[ ] Brief IT security team on completion
[ ] Log all actions with timestamps
```

---

## 7. Contractor / Vendor Access Management

### Contractor Onboarding
```
CONTRACTOR ACCESS PROVISIONING:

Principle: Minimum Access Required, Time-Limited

Identity:
[ ] Create contractor-specific account (prefix: c-jsmith@company.com or separate directory)
[ ] Set account expiry = contract end date
[ ] Assign only contractor-tier licenses (no M365 E3 if not needed)
[ ] No access to corporate email distribution lists

Access:
[ ] Only systems required for the engagement
[ ] Time-limited access (expire with contract)
[ ] No access to HR, Finance, or other sensitive systems unless specifically required
[ ] VPN: Restricted split-tunnel limiting to project systems

Remote Access:
[ ] Vendor portal (preferred for external vendor support)
[ ] Require MFA for all remote access
[ ] Session recording for privileged vendor access (PAM tool)
[ ] Restrict access windows (business hours only if possible)

Documentation:
[ ] NDA / confidentiality agreement signed
[ ] Security awareness acknowledgment
[ ] Access request approved by project sponsor and IT Security
[ ] Contractor registered in vendor management system
```

### Contractor Offboarding
```
Contractor End of Engagement:
[ ] Same-day revocation of all access
[ ] Account disabled and marked with end date
[ ] Any company-provided device collected
[ ] Return any credentials, hardware tokens
[ ] Confirm no data retained (signed data destruction certificate)
[ ] Close vendor access in all external portals
[ ] Archive project work to company systems
[ ] Final invoice/PO cleared
```

---

## 8. Access Review Process

### Quarterly Access Review
```
QUARTERLY ACCESS REVIEW PROCEDURE

Schedule: Q1 (Jan), Q2 (Apr), Q3 (Jul), Q4 (Oct)

STEP 1: Generate access report
  - Export all user-to-system access assignments
  - Identify joiners/movers/leavers since last review
  - Flag accounts inactive > 60 days

STEP 2: Distribute to managers/application owners
  - Send "certify or revoke" review request
  - Include: User, system, access level, last login date
  - Deadline: 10 business days

STEP 3: Manager review actions
  - CERTIFY: User still needs access, level is appropriate
  - MODIFY: Access level should change
  - REVOKE: User no longer needs access

STEP 4: Execute changes
  - IT implements all revoke/modify decisions within 5 business days
  - Document changes in tickets

STEP 5: Non-response escalation
  - If manager doesn't respond: Escalate to their manager
  - If no response by deadline: Default to REVOKE (for SOC2 compliance)

STEP 6: Report and archive
  - Access review completion report (% certified, % revoked)
  - Archive for audit evidence
  - Report non-compliance to Security/Compliance team
```

---

## 9. Metrics & SLA

### Onboarding/Offboarding SLAs
| Metric | Target | Measurement |
|--------|--------|-------------|
| Account creation (standard) | Day before start | % on time |
| Account creation (rush/same day) | 4 business hours | % on time |
| Involuntary termination revocation | < 30 minutes | % within SLA |
| Voluntary termination revocation | Same day EOD | % within SLA |
| Device provisioning | T-1 (ready Day 1) | % on time |
| Device recovery (involuntary) | Same day | % same day |
| Contractor access expiry | Auto-expiry = contract date | % expired on time |
| Quarterly access review completion | 100% by deadline | % completion |

### Monthly Reporting
```
IDENTITY LIFECYCLE METRICS — [Month]

Joiners: [N]
  Avg provisioning time: [N] hours
  On-time (pre-start): [N]%
  Issues during onboarding: [N]

Leavers: [N]
  Avg revocation time: [N] minutes (involuntary)
  Same-day revocation rate: [N]%
  Devices recovered: [N]/[N]

Movers (role changes): [N]
  Access updated within 5 days: [N]%

Orphaned accounts (no login 90 days): [N]
  Disabled this month: [N]

Access reviews:
  Quarterly review completion: [N]%
  Accesses revoked: [N]
```

---

*Last Updated: 2025 | IT Operations Documentation Library*
