# IT Standard Operating Procedures (SOPs)

## Overview
Collection of standard operating procedures for common IT operations tasks. Each SOP is designed to be followed step-by-step, ensuring consistent and reliable execution across team members.

---

## SOP Index

1. [SOP-001: New User Onboarding](#sop-001)
2. [SOP-002: Employee Offboarding](#sop-002)
3. [SOP-003: Server Patching Procedure](#sop-003)
4. [SOP-004: Backup Verification](#sop-004)
5. [SOP-005: Emergency Change Management](#sop-005)
6. [SOP-006: Workstation Deployment](#sop-006)
7. [SOP-007: Certificate Renewal](#sop-007)
8. [SOP-008: Access Request Fulfillment](#sop-008)
9. [SOP-009: Firewall Rule Request](#sop-009)
10. [SOP-010: Vendor Remote Access Management](#sop-010)

---

## SOP-001: New User Onboarding <a name="sop-001"></a>

**Version:** 2.1  
**Owner:** IT Help Desk Lead  
**Last Reviewed:** 2025-01-15  
**Trigger:** HR submits onboarding request form ≥5 business days before start date

### Prerequisites
- Completed onboarding request from HR (includes: start date, name, job title, department, manager, work location, remote/office, equipment needed)
- Manager approval for system access levels
- Equipment available in inventory

### Procedure

**Step 1: Create Active Directory Account**
```powershell
$firstName = "<FirstName>"
$lastName = "<LastName>"
$username = "$($firstName.Substring(0,1).ToLower())$($lastName.ToLower())"  # e.g., jsmith
$dept = "<Department>"
$title = "<Job Title>"
$manager = "<Manager SAMAccountName>"
$ou = "<OU path based on department>"

New-ADUser `
  -GivenName $firstName `
  -Surname $lastName `
  -Name "$firstName $lastName" `
  -SamAccountName $username `
  -UserPrincipalName "$username@domain.com" `
  -DisplayName "$firstName $lastName" `
  -Department $dept `
  -Title $title `
  -Manager $manager `
  -Path $ou `
  -AccountPassword (ConvertTo-SecureString "Temp@$(Get-Random -Maximum 9999)" -AsPlainText -Force) `
  -ChangePasswordAtLogon $true `
  -Enabled $true
```
✅ Verify: Account visible in ADUC, in correct OU, enabled

**Step 2: Assign AD Group Memberships**
```
Based on department, add to:
□ DL_<Department>_All (department distribution list)
□ GG_<Department>_<Role> (role-based security group)
□ GG_All_Users (all-users group if applicable)
□ Any specific application groups per manager request
```

**Step 3: Create Microsoft 365 Account**
```
Microsoft 365 Admin Center → Users → Add user
- Assign license: Microsoft 365 Business Standard/E3 (per company plan)
- Enable: Exchange Online, Teams, SharePoint, OneDrive
- Set primary email: firstname.lastname@domain.com
- Aliases if needed
```
✅ Verify: Email accessible, Teams license assigned

**Step 4: Configure MFA**
```
Entra ID Admin Center → Users → [user] → Authentication methods → Add method
Add: Microsoft Authenticator
- OR: Set policy to require MFA setup at first login (preferred)
```

**Step 5: Provision Hardware**
```
□ Retrieve assigned device from inventory
□ Verify device is current on OS patches
□ Verify antivirus/EDR agent is active
□ Verify device is domain-joined (or Azure AD joined for cloud-only)
□ Test login with new account credentials
□ Confirm all required applications are installed/available
□ Label device with asset tag
□ Record serial number and asset tag in CMDB
□ Prepare accessories (power adapter, bag if policy)
```

**Step 6: Provision Additional Access**
```
Per manager-approved access request:
□ VPN access (add to VPN group)
□ File share access (add to appropriate DL groups)
□ Business applications (Salesforce, ERP, etc.) — create accounts
□ Building/badge access — submit request to facilities
□ Phone/extension assignment
```

**Step 7: Welcome Communication**

Send to user and their manager 1 business day before start:
```
Subject: Your IT Setup — Welcome to [Company Name]!

Hi [Name],

Welcome! Your IT setup is complete. Here's everything you need for Day 1:

USERNAME: firstname.lastname@domain.com
TEMPORARY PASSWORD: [password] — you'll be prompted to change this at first login
COMPUTER: [Model] — [serial or asset tag]
LOCATION: [Where to pick it up / it will be at your desk]

On your first login:
  1. You'll be asked to set a new password (must be 14+ characters)
  2. You'll need to set up Microsoft Authenticator for two-factor authentication
     (Search "Microsoft Authenticator" in your phone's app store and follow the prompts)

IT Help Desk: ext. 1234 | helpdesk@domain.com | Teams: @ITHelpDesk

Looking forward to working with you!
[Your name]
IT Help Desk
```

**Step 8: Documentation**
```
□ CMDB updated: Device assigned to user
□ Ticket closed with list of all access provisioned
□ Note any pending items (badge access, specific app accounts)
□ Notify manager that setup is complete
```

---

## SOP-002: Employee Offboarding <a name="sop-002"></a>

**Version:** 2.0  
**Owner:** IT Help Desk Lead  
**Last Reviewed:** 2025-01-15  
**Trigger:** HR submits offboarding request (last day: _____)

### ⚠️ Confidentiality Note
Offboarding is sensitive. Do not discuss details with other employees. Coordinate only with HR and the departing employee's direct manager.

### Day-of Procedure (Execute at close of business on last day)

**Step 1: Disable All Authentication**
```powershell
$username = "<samaccountname>"

# Disable AD account
Disable-ADAccount -Identity $username

# Reset password (prevent re-enabling with known password)
Set-ADAccountPassword -Identity $username -Reset -NewPassword (
  ConvertTo-SecureString ([System.Web.Security.Membership]::GeneratePassword(20,4)) -AsPlainText -Force
)

# Record time disabled in ticket
```
✅ Verify: Cannot log into domain

**Step 2: Revoke Cloud Access**
```
Entra ID Admin Center → Users → [user]:
□ Sign out of all sessions: Revoke sign-in sessions
□ Disable account in Azure AD (sync from on-prem AD, or disable directly in cloud-only)
□ Revoke all refresh tokens: Revoke-AzureADUserAllRefreshToken -ObjectId [ObjectId]
□ Remove from all MFA methods (prevents reactivation attacks)
```

**Step 3: Email Handling**
```
Per manager/HR instruction:
Option A: Auto-reply + delegate to manager
  - Set out-of-office message: "This employee has left. Please contact [manager] at [email]."
  - Grant manager access to mailbox for 90 days
  
Option B: Forward to manager
  New-InboxRule -Mailbox $username -ForwardTo "manager@domain.com"

Note: Check for legal/compliance hold requirements before any deletion
Per retention policy: Mailbox preserved for [X days/years]
```

**Step 4: Remove From Groups / Mailing Lists**
```powershell
# Remove from all groups (except for audit record preservation)
Get-ADUser -Identity $username -Properties MemberOf | 
  Select-Object -ExpandProperty MemberOf | ForEach-Object {
    Remove-ADGroupMember -Identity $_ -Members $username -Confirm:$false
    Write-Host "Removed from: $_"
  }
```

**Step 5: Revoke Application Access**
```
□ VPN: Remove from VPN access group
□ Salesforce / CRM: Deactivate user
□ ERP system: Deactivate user
□ All other SaaS tools: Per application list in offboarding form
□ Building access: Notify facilities (if separate system)
□ Physical keys/access cards: Confirm collected by HR
```

**Step 6: Move AD Account to Disabled OU**
```powershell
Move-ADObject `
  -Identity (Get-ADUser $username).DistinguishedName `
  -TargetPath "OU=Disabled,DC=domain,DC=com"

Set-ADUser -Identity $username `
  -Description "DISABLED: $(Get-Date -Format 'yyyy-MM-dd') | Reason: Employee Departure | Ticket: #XXXXX"
```

**Step 7: Equipment Return**
```
□ Laptop returned to IT (or shipped via provided label for remote employees)
□ Power adapter, docking station, peripherals collected
□ Update CMDB: Device status = "Available" or "In Repair"
□ Wipe device: Bitlocker key change or factory reset
□ Re-image before redeployment
```

**30-Day Follow-up:**
```
□ Archive mailbox per data retention policy
□ Remove Office 365 license (save cost — confirm with HR/manager)
□ Final audit: Any remaining active accounts in SaaS tools?
□ Deprovision any service accounts created for this user
```

---

## SOP-003: Server Patching Procedure <a name="sop-003"></a>

**Version:** 1.5  
**Owner:** Infrastructure Team  
**Last Reviewed:** 2025-02-01

### Patch Rings

| Ring | Timing | Systems |
|---|---|---|
| Ring 0 — Pilot | Tuesday after Patch Tuesday | IT team workstations |
| Ring 1 — Early | Week 2 after Patch Tuesday | 10% of workstations |
| Ring 2 — Production | Week 3 after Patch Tuesday | All workstations |
| Ring 3 — Servers | Week 4 / maintenance window | All servers (Tier 2 first, then Tier 1) |
| Ring 4 — Critical | Separate window, extra testing | DCs, Tier 0 systems |

### Server Patching Checklist

**Pre-Patching (1 week before):**
```
□ Review patch list — identify any known problematic patches (check vendor KB)
□ Notify application owners of patching window
□ Confirm backup is current (within 24 hours)
□ Snapshot VM (for VMs — snapshot before, delete after successful patch)
□ Confirm rollback procedure
□ Schedule maintenance window in monitoring (silence alerts)
□ Verify sufficient disk space (Windows Update needs ~5GB free)
```

**Patching Execution:**
```powershell
# Review pending updates
Get-WindowsUpdate -MicrosoftUpdate | Select-Object Title, MsrcSeverity, Size | Format-Table

# Install all updates (schedule reboot)
Install-WindowsUpdate -MicrosoftUpdate -AcceptAll -AutoReboot | Select-Object Title, Result

# Or install without reboot (schedule separately)
Install-WindowsUpdate -MicrosoftUpdate -AcceptAll -IgnoreReboot

# Schedule reboot
shutdown /r /t 0 /c "Scheduled maintenance reboot for security patches"

# Or using Task Scheduler for off-hours
$trigger = New-ScheduledTaskTrigger -Once -At "2:00AM"
$action = New-ScheduledTaskAction -Execute "shutdown.exe" -Argument "/r /t 0"
Register-ScheduledTask -TaskName "MaintenanceReboot" -Trigger $trigger -Action $action -RunLevel Highest
```

**Post-Patching Verification:**
```
□ Server rebooted successfully
□ All critical services are running
□ Application tested (ping application owner for confirmation)
□ Check Event Viewer for post-reboot errors
□ Verify patch installation:
  Get-HotFix | Sort-Object InstalledOn -Descending | Select-Object -First 10
□ Delete VM snapshot (only after successful verification — within 48 hours)
□ Remove monitoring silence
□ Document patches applied in ticket
```

---

## SOP-004: Backup Verification <a name="sop-004"></a>

**Version:** 1.2  
**Owner:** Infrastructure Team  
**Frequency:** Monthly (for Tier 1), Quarterly (for full DR test)

### Monthly Backup Verification

```
□ Review backup job history for previous 30 days:
  - Any failed jobs?
  - Any jobs with warnings?
  - All systems covered?

□ Verify backup storage:
  - Sufficient free space (>30% of total capacity)
  - No repository errors

□ Perform test restore:
  - Select a random file from file server (last 7 days)
  - Restore to alternate location
  - Verify file integrity and accessibility
  - Document: System, backup date used, restore time, success/fail

□ Verify offsite/cloud backup replication:
  - Check replication status
  - Verify last successful replication timestamp

□ Check backup agent status on all servers:
  - All agents checking in?
  - Any agents with errors?
```

### Quarterly DR Test

```
□ Identify test scope (which system to restore)
□ Reserve isolated test environment
□ Restore server from backup to isolated environment
□ Boot and verify:
  - OS boots without errors
  - Services start successfully
  - Data appears intact (spot check key files)
  - Application functional (basic smoke test)
□ Measure actual RTO vs. target RTO
□ Measure data loss (actual RPO vs. target RPO)
□ Document results including:
  - Start time
  - Completion time
  - Issues encountered
  - Pass/fail against RTO/RPO targets
□ Destroy test environment
□ Report results to IT Manager
□ Log in the DR test log
```

---

## SOP-005: Emergency Change Management <a name="sop-005"></a>

**Version:** 1.3  
**Owner:** IT Manager  
**Use:** Unplanned changes needed outside normal CAB process

### Definition
Emergency Change: A change required to restore service, prevent imminent outage, or address a security incident that cannot wait for the next CAB cycle.

### Emergency Change Process

```
Step 1: Assess (5 minutes)
  - Is this truly an emergency? (Would waiting until next maintenance window cause harm?)
  - What's the risk of making the change vs. not making it?
  - Is there a tested rollback plan?

Step 2: Get Approval (immediate)
  Required approvers (get minimum 2):
  □ IT Manager (or designated on-call manager)
  □ System/application owner
  Optional but recommended:
  □ CISO (for security-related emergency changes)
  □ Business owner (if high business risk)

  Document: Who approved, when, via what method (verbal → follow up with email/ticket)

Step 3: Implement
  □ Execute change with another IT staff member present/monitoring if possible
  □ Document every command/action as you go (real-time notes in ticket)
  □ Monitor impact immediately after each step

Step 4: Verify
  □ Confirm issue is resolved
  □ Test affected services from end-user perspective
  □ Monitor for 30 minutes before declaring success

Step 5: Document (within 24 hours)
  □ Full RFC document completed retroactively
  □ All approvals documented
  □ Timeline and actions documented
  □ Impact documented
  □ Lessons learned noted
  □ Presented at next CAB meeting for review
```

---

## SOP-006: Workstation Deployment <a name="sop-006"></a>

**Version:** 2.0  
**Owner:** Help Desk Team  
**Use:** Deploying a new or replacement workstation

### Deployment Process

**Option A: Automated (MDT/Intune Autopilot — preferred)**
```
1. Add device serial number to Autopilot in Intune
2. Power on device and connect to network
3. Device automatically:
   - Enrolls in Intune
   - Joins Azure AD / Hybrid Azure AD join
   - Installs all required applications
   - Applies security baselines
4. Technician verifies completion (~45–90 min)
5. Hand off to user with instructions
```

**Option B: Manual Deployment**
```
Step 1: Image the device
□ Boot from WDS/MDT server or USB imaging drive
□ Apply standard image (Windows 11 Enterprise, latest build)
□ Verify image applied successfully

Step 2: Initial Configuration
□ Set computer name per naming convention: [SITE][Dept][Type][Number]
  Rename-Computer -NewName "NYC-IT-WS001" -Restart
□ Join domain:
  Add-Computer -DomainName "domain.com" -Credential (Get-Credential) -Restart

Step 3: Software Installation
□ Required: Microsoft 365 Apps, Chrome/Edge, VPN client, EDR agent
□ Department-specific: Per deployment checklist for role
□ Via SCCM/Intune push or manual install

Step 4: Windows Update
□ Run Windows Update fully — reboot as needed
□ Minimum: All critical and important updates installed

Step 5: Security Configuration
□ Verify EDR agent active and reporting
□ Verify Bitlocker enabled and key escrowed
□ Verify device appears in Intune/SCCM

Step 6: User Account Setup
□ Log in as new user (verify AD account works)
□ Verify MFA working
□ Verify email, Teams, SharePoint accessible

Step 7: Handoff
□ Update CMDB: Assign to user
□ Apply asset tag label
□ Provide user with quick-start guide
□ Document deployment in ticket
```

---

## SOP-007: SSL/TLS Certificate Renewal <a name="sop-007"></a>

**Version:** 1.1  
**Owner:** Infrastructure Team  
**Trigger:** Certificate expiry alert (60-day warning)

### Certificate Renewal Process

**Step 1: Identify the Certificate**
```powershell
# Check certificate details
$cert = Get-ChildItem Cert:\LocalMachine\My | Where-Object {$_.Subject -like "*server.domain.com*"}
$cert | Select-Object Subject, NotAfter, Issuer, Thumbprint

# Check all certificates expiring in 60 days
Get-ChildItem Cert:\LocalMachine\My | 
  Where-Object {$_.NotAfter -lt (Get-Date).AddDays(60)} |
  Select-Object Subject, NotAfter, Issuer | Sort-Object NotAfter
```

**Step 2: Generate CSR (if external CA)**
```powershell
# Create CSR via IIS Manager or certreq
$inf = @"
[Version]
Signature="`$Windows NT`$"
[NewRequest]
Subject = "CN=server.domain.com,OU=IT,O=Company,C=US"
KeySpec = 1
KeyLength = 2048
Exportable = TRUE
MachineKeySet = TRUE
SMIME = False
PrivateKeyArchive = FALSE
UserProtected = FALSE
UseExistingKeySet = FALSE
ProviderName = "Microsoft RSA SChannel Cryptographic Provider"
ProviderType = 12
RequestType = CMC
KeyUsage = 0xa0
[EnhancedKeyUsageExtension]
OID=1.3.6.1.5.5.7.3.1 ; Server Authentication
[Extensions]
2.5.29.17 = "{text}dns=server.domain.com&dns=www.server.domain.com"
"@

$inf | Out-File -FilePath C:\csr.inf -Encoding ASCII
certreq -new C:\csr.inf C:\csr.req
```

**Step 3: Submit to CA and Install**
```
External CA: Submit CSR via CA portal, download certificate
Internal CA: Auto-enroll via GPO or manual via certsrv

Install certificate:
certreq -accept <certificate.cer>

Or via IIS:
IIS Manager → [Server] → Server Certificates → Complete Certificate Request
```

**Step 4: Bind Certificate**
```powershell
# IIS binding update
Import-Module WebAdministration
$cert = Get-ChildItem Cert:\LocalMachine\My | Where-Object {$_.Subject -like "*server.domain.com*"} | Select-Object -First 1

# Remove old binding
Remove-WebBinding -Name "Default Web Site" -BindingInformation "*:443:"

# Add new binding with new certificate
New-WebBinding -Name "Default Web Site" -Protocol "https" -Port 443 -IPAddress "*"
$binding = Get-WebBinding -Name "Default Web Site" -Protocol "https"
$binding.AddSslCertificate($cert.Thumbprint, "My")
```

**Step 5: Verify**
```
□ Test URL in browser — no certificate warning
□ Verify expiry date: New cert shows correct expiry
□ Check certificate chain is complete (no missing intermediate)
□ Test from external network or use ssllabs.com (for public certs)
□ Remove old/expired certificate from store
□ Update certificate inventory spreadsheet
```

---

## SOP-008: Access Request Fulfillment <a name="sop-008"></a>

**Version:** 1.4  
**Owner:** IT Help Desk  
**Trigger:** Access request ticket submitted with manager approval

### Access Request Verification
```
Before provisioning ANY access:
□ Request submitted by the user or their manager (not a third party)
□ Manager approval documented in ticket (email, approval system, or verbal confirmed with manager)
□ Request is business-justified (job role requires this access)
□ Access is available to their role per data classification policy
```

### Common Access Types

**File Share Access:**
```powershell
# Add user to appropriate AD security group for the share
Add-ADGroupMember -Identity "DL_ShareName_ReadWrite" -Members "jsmith"

# Verify
(Get-ADGroup "DL_ShareName_ReadWrite" | Get-ADGroupMember).SamAccountName
```

**Application Access:**
```
Per-application process:
  Salesforce: Setup → Manage Users → New User or assign license
  ERP: Application admin console → User management
  SharePoint: Site Settings → People and Groups
  VPN: Add to "GG_VPN_Users" AD group
```

---

## SOP-009: Firewall Rule Request <a name="sop-009"></a>

**Version:** 1.0  
**Owner:** Network/Security Team

### Firewall Rule Request Requirements
Every firewall rule change requires:
```
□ Source IP/subnet (be specific — no "any" unless justified)
□ Destination IP/subnet (be specific)
□ Protocol (TCP/UDP/ICMP)
□ Port(s) — be specific, not port ranges unless required
□ Direction (inbound/outbound/both)
□ Business justification (what application/service requires this?)
□ Owner/requester (who is accountable for this rule?)
□ Expiry date (if temporary) or permanent business justification
□ Manager approval
□ Security review for high-risk rules (internet-facing, privileged ports)
```

### Rule Implementation
```
1. Review request for completeness and security concerns
2. Test in non-production if available
3. Implement with documentation comment in rule:
   "Ticket#12345 - 2024-03-15 - JSMITH - Allow HR app server to AD LDAP"
4. Verify functionality with requester
5. Document in firewall rule change log
6. Add to next firewall audit cycle
```

---

## SOP-010: Vendor Remote Access Management <a name="sop-010"></a>

**Version:** 1.2  
**Owner:** IT Security / Help Desk

### Vendor Remote Access Principles
```
1. Least privilege: Vendor gets access only to systems they support
2. Time-limited: Access granted for specific time window only
3. Monitored: All sessions logged and recorded if possible
4. Named accounts: No shared credentials for vendors
5. Approved: Business owner approval required before granting
```

### Granting Vendor Access
```
Step 1: Verify request
□ Ticket from authorized internal requester
□ Business owner approval documented
□ Vendor has signed NDA and data processing agreement
□ Access start and end time specified

Step 2: Provision access
□ Create named vendor account: vendor_companyname_technician
□ Set account expiry date = end of access window
□ Restrict to specific systems only (GPO, firewall rules)
□ Provide credentials via secure method (not email — use encrypted message or phone)

Step 3: Monitor session
□ Enable session recording if capability available (CyberArk, BeyondTrust, Jump)
□ Internal IT staff present or monitoring during sensitive access
□ Vendor informed: "This session is being recorded"

Step 4: Access revocation
□ At expiry: Account auto-disables (verify this happens)
□ Manual: Disable immediately upon vendor confirmation of completion
□ Review logs for any unexpected activity after session
□ Document in vendor access log
```
