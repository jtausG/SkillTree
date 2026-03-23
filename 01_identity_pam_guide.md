# Identity & Privileged Access Management (PAM) Guide

## Table of Contents
1. [Identity Fundamentals](#identity-fundamentals)
2. [Identity Lifecycle Management](#identity-lifecycle)
3. [Multi-Factor Authentication (MFA)](#mfa)
4. [Single Sign-On (SSO) & Federation](#sso-federation)
5. [Privileged Access Management](#pam)
6. [Just-in-Time Access](#jit-access)
7. [Identity Governance](#governance)
8. [Conditional Access & Zero Trust Identity](#conditional-access)
9. [Service Account Management](#service-accounts)
10. [Identity Threat Detection](#threat-detection)

---

## 1. Identity Fundamentals

### The Identity Perimeter
Modern security treats identity as the new perimeter. With cloud and remote work, network location no longer guarantees trust. Every access request must be verified regardless of where it originates.

**Core Identity Principles:**
- **Verify explicitly** — Authenticate and authorize based on all available signals
- **Least privilege** — Grant minimum access needed, for minimum time
- **Assume breach** — Operate as though attackers are already inside

### Identity Pillars

```
Authentication (AuthN):  Who are you? (Username + password + MFA)
Authorization (AuthZ):   What can you do? (RBAC, ABAC, permissions)
Accounting:             What did you do? (Audit logs)
```

### Identity Providers

| Product | Type | Best For |
|---------|------|---------|
| Azure AD / Entra ID | Cloud IdP | Microsoft 365, cloud-first orgs |
| Active Directory | On-prem IdP | Legacy on-prem environments |
| Okta | Cloud IdP | Multi-cloud, app federation |
| Ping Identity | Enterprise IdP | Complex federation scenarios |
| JumpCloud | Cloud directory | SMB, non-Microsoft shops |
| Auth0 | Developer IdP | Application identity |

---

## 2. Identity Lifecycle Management

### Joiner-Mover-Leaver Process

**Joiner (New Employee):**
```
HR System Creates Record
        ↓
Identity Provisioning (AD/Azure AD account created)
        ↓
Group Membership (based on department/role/location)
        ↓
License Assignment (M365, Salesforce, Slack, etc.)
        ↓
Device Provisioning (Autopilot / Intune enrollment)
        ↓
Access Verification (manager confirms access is correct)
        ↓
Welcome Communication (credentials, MFA setup instructions)
```

**Mover (Role Change):**
```
HR System Updates Record
        ↓
Identity Attributes Updated (department, title, manager, location)
        ↓
Old Access Removed (previous role groups)
        ↓
New Access Granted (new role groups/licenses)
        ↓
Manager Attestation (new manager confirms access)
        ↓
Audit Log Entry
```

**Leaver (Termination):**
```
HR Triggers Termination
        ↓
Account Disabled (T=0, same day)
        ↓
Active Sessions Revoked (Azure AD sign-in tokens revoked)
        ↓
MFA Methods Cleared
        ↓
Licenses Reclaimed (within 24 hours)
        ↓
Email Delegated (to manager, per policy)
        ↓
Data Preservation (mailbox/OneDrive litigation hold)
        ↓
Account Deleted (after retention period, typically 30–90 days)
```

### Automated Provisioning with Azure AD

```powershell
# Configure SCIM provisioning for an app
# Done in Azure AD portal: Enterprise Apps > [App] > Provisioning

# Using MS Graph API to check provisioning status
Connect-MgGraph -Scopes "Application.ReadWrite.All","Directory.ReadWrite.All"

# Get provisioning logs
Get-MgAuditLogProvisioning -Filter "activityDateTime ge 2024-01-01" |
    Select-Object ActivityDateTime, ActivityDisplayName, Status, 
                  TargetResources | Format-Table

# Check user's group memberships
$User = Get-MgUser -Filter "userPrincipalName eq 'john.doe@contoso.com'" 
Get-MgUserMemberOf -UserId $User.Id | 
    Select-Object @{N="GroupName";E={$_.AdditionalProperties.displayName}}
```

---

## 3. Multi-Factor Authentication (MFA)

### MFA Methods Comparison

| Method | Security Level | Phishing Resistant | User Experience |
|--------|---------------|-------------------|----------------|
| SMS OTP | Low | No | Poor (SIM swap risk) |
| Voice Call OTP | Low | No | Poor |
| TOTP App (Authenticator) | Medium | No | Good |
| Push Notification | Medium | Partially | Very Good |
| FIDO2 / Passkey | High | Yes | Excellent |
| Hardware Token (YubiKey) | Very High | Yes | Good |
| Certificate-based | Very High | Yes | Transparent |

**Recommendation Priority:**
```
1st choice: FIDO2 passkeys / Windows Hello for Business
2nd choice: Microsoft/Google Authenticator (TOTP or Push)
3rd choice: Hardware TOTP token
Avoid: SMS/Voice OTP (acceptable as fallback only)
```

### Azure AD MFA Configuration

```powershell
# Check MFA registration status
Connect-MgGraph -Scopes "UserAuthenticationMethod.Read.All"

$Users = Get-MgUser -All -Select "id,userPrincipalName,displayName"
$MFAReport = foreach ($User in $Users) {
    $Methods = Get-MgUserAuthenticationMethod -UserId $User.Id
    [PSCustomObject]@{
        User = $User.UserPrincipalName
        MFAMethods = ($Methods | Select-Object -ExpandProperty AdditionalProperties | 
                      ForEach-Object { $_['@odata.type'] }) -join "; "
        MFAEnabled = $Methods.Count -gt 1
    }
}

$MFAReport | Where-Object { -not $_.MFAEnabled } | 
    Export-Csv "MFA_Not_Registered.csv" -NoTypeInformation

# Disable legacy authentication (blocks Basic Auth — required for MFA enforcement)
# Done via Conditional Access Policy:
# - Cloud apps: All apps
# - Conditions > Client apps: Exchange ActiveSync + Other clients
# - Grant: Block
```

### FIDO2 / Passkey Deployment

```
Prerequisites:
□ Azure AD joined or Hybrid Azure AD joined devices
□ Windows 10 1903+ or Windows 11
□ FIDO2-compatible security key or platform authenticator
□ Enable FIDO2 in Azure AD Authentication Methods policy

Deployment Steps:
1. Azure AD > Authentication Methods > Policies
2. Enable FIDO2 Security Key method
3. Configure: Key restrictions (if using specific key vendors)
4. Target initial pilot group
5. User self-registration at aka.ms/mysecurityinfo
6. Phased rollout by department
```

---

## 4. Single Sign-On (SSO) & Federation

### SSO Protocols

**SAML 2.0 (Security Assertion Markup Language):**
```
Flow:
1. User accesses Service Provider (SP) app
2. SP redirects to Identity Provider (IdP) with AuthnRequest
3. IdP authenticates user
4. IdP sends signed SAML Assertion back to SP
5. SP validates assertion, creates session

Used by: Salesforce, Workday, ServiceNow (enterprise apps)
```

**OAuth 2.0 + OpenID Connect (OIDC):**
```
Flow:
1. App redirects to IdP with OAuth request
2. User authenticates
3. IdP returns authorization code
4. App exchanges code for ID token + access token
5. App validates token, creates session

Used by: Modern apps, APIs, mobile apps
```

**WS-Federation:**
```
Legacy Microsoft protocol
Used by: SharePoint, older Microsoft apps
Being replaced by SAML/OIDC
```

### Azure AD Enterprise App SSO Setup

```powershell
# Get enterprise apps with SSO configured
Connect-MgGraph -Scopes "Application.Read.All"

Get-MgServicePrincipal -All -Filter "tags/any(t:t eq 'WindowsAzureActiveDirectoryIntegratedApp')" |
    Select-Object DisplayName, AppId, SignInAudience |
    Format-Table

# Check app assignments
$AppId = "your-app-id"
$SP = Get-MgServicePrincipal -Filter "appId eq '$AppId'"
Get-MgServicePrincipalAppRoleAssignment -ServicePrincipalId $SP.Id |
    ForEach-Object {
        [PSCustomObject]@{
            PrincipalType = $_.PrincipalType
            PrincipalId = $_.PrincipalId
            AppRole = $_.AppRoleId
        }
    }
```

### SAML Troubleshooting

```
Common SAML Errors:

"AADSTS50105: The signed-in user is not assigned to a role for the application"
→ User not assigned to the Enterprise App
→ Fix: Add user/group assignment in Enterprise App > Users and Groups

"AADSTS75011: Authentication method does not match"
→ App requested specific auth method not available to user
→ Fix: Check Conditional Access policies and app sign-in settings

"Invalid certificate" / "Signature validation failed"
→ Certificate expired or wrong cert uploaded
→ Fix: Rotate SAML signing certificate, update in SP

"SAML response not valid"
→ Clock skew > 5 minutes between IdP and SP
→ Fix: Sync time on all servers (NTP)

Debugging SAML: Use browser extension "SAML-tracer" to capture assertions
```

---

## 5. Privileged Access Management

### Tiered Access Model (Microsoft PAW Model)

```
Tier 0 — Identity/Security Infrastructure:
  Assets:   Domain Controllers, Azure AD, PKI, PAM systems
  Accounts: Separate Tier 0 admin accounts
  Access:   Only from Tier 0 PAWs (hardened admin workstations)
  
Tier 1 — Server/Application Infrastructure:
  Assets:   Member servers, applications, cloud services
  Accounts: Separate Tier 1 admin accounts
  Access:   Only from Tier 1 PAWs or jump servers
  
Tier 2 — Workstation/Device:
  Assets:   End-user workstations, devices
  Accounts: Tier 2 admin accounts (local admin)
  Access:   Standard IT workstations

RULE: Accounts from lower tiers CANNOT administer higher tiers
      Tier 2 account cannot log into Tier 1 server
      Tier 1 account cannot log into Tier 0 systems
```

### PAM Solutions

**Microsoft PIM (Privileged Identity Management):**
```powershell
# Activate eligible role (user self-service)
# Done in Azure portal: PIM > My roles > Activate

# PowerShell: Assign eligible role
Connect-MgGraph -Scopes "RoleEligibilitySchedule.ReadWrite.Directory"

$User = Get-MgUser -UserId "admin@contoso.com"
$Role = Get-MgRoleManagementDirectoryRoleDefinition -Filter "displayName eq 'Global Administrator'"

New-MgRoleManagementDirectoryRoleEligibilityScheduleRequest -BodyParameter @{
    Action = "adminAssign"
    Justification = "Need break-glass admin access"
    RoleDefinitionId = $Role.Id
    DirectoryScopeId = "/"
    PrincipalId = $User.Id
    ScheduleInfo = @{
        StartDateTime = (Get-Date).ToString("o")
        Expiration = @{
            Type = "noExpiration"
        }
    }
}

# Get active PIM activations
Get-MgRoleManagementDirectoryRoleAssignmentScheduleInstance -All |
    Where-Object { $_.AssignmentType -eq "Activated" } |
    Select-Object PrincipalId, RoleDefinitionId, StartDateTime, EndDateTime
```

**CyberArk / Beyond Trust / Delinea:**
```
Core capabilities:
- Password vault: Rotate and check out privileged credentials
- Session recording: Record all privileged sessions
- Just-in-time access: Grant access only when needed
- Credential injection: Apps never see passwords
- Threat analytics: Detect anomalous privileged behavior

Key workflows:
1. Admin requests access to server
2. PAM system authenticates and authorizes request
3. PAM checks out credential from vault
4. Session launched through PAM proxy (recorded)
5. Session ends → credential auto-rotated
6. Full session recording stored for audit
```

### Break-Glass Accounts

```
Break-glass (emergency access) accounts:
- Purpose: Access Azure AD if MFA/PIM unavailable
- Should have: Global Admin role permanently assigned
- NOT protected by: Conditional Access, MFA (by design)
- Protected by: Extremely strong password, hardware FIDO2 key
- Quantity: Minimum 2 accounts (geographic/vendor redundancy)
- Storage: Password sealed in envelope in physical safe
- Testing: Test quarterly (verify can sign in), rotate credential
- Monitoring: Alert on every sign-in (should never happen normally)

Naming: BreakGlass-01@contoso.com (obviously named for audit clarity)
```

---

## 6. Just-in-Time Access

### JIT Principles

```
Traditional model: Standing access (always available)
JIT model: Access granted only when needed, expires automatically

Benefits:
- Reduces attack surface dramatically
- Limits damage from credential compromise
- Creates natural audit trail (every activation logged)
- Forces justification for privileged actions

JIT workflow:
Request → Approval (manager or automatic) → 
Time-limited access → Auto-expiry → Audit log
```

### Azure AD PIM JIT Configuration

```
Policy settings per role:
- Activation duration: 1–8 hours (recommended)
- Require approval: Yes for Tier 0, optional for Tier 1
- Require justification: Yes
- Require MFA on activation: Yes
- Require ticket number: Optional (integrates with ITSM)
- Notification on activation: Yes

Eligible assignment expiration:
- Tier 0 roles: 6 months (re-certify regularly)
- Tier 1 roles: 12 months
- Read-only admin: 12 months
```

---

## 7. Identity Governance

### Access Reviews

```
What to review:
- Privileged role memberships (monthly)
- Application assignments (quarterly)
- Guest/external user accounts (quarterly)
- All user group memberships (annually)
- Service account permissions (annually)

Review process:
1. System generates list of access to review
2. Sends to reviewer (manager or resource owner)
3. Reviewer approves or denies each assignment
4. Denied access automatically removed
5. Results logged for audit/compliance

Azure AD Access Reviews:
Portal: Azure AD > Identity Governance > Access Reviews
```

### Entitlement Management

```powershell
# Azure AD Entitlement Management (Access Packages)
# Access packages bundle multiple resource accesses together

# Example: "Marketing Team Member" access package contains:
# - SharePoint Marketing Site access
# - Marketing Teams channel membership  
# - Marketing Salesforce profile
# - Adobe Creative Cloud license

# Users request package; approvers approve once; all access granted
# Package expires → all access removed automatically

# PowerShell to check access packages
Connect-MgGraph -Scopes "EntitlementManagement.ReadWrite.All"

Get-MgEntitlementManagementAccessPackage -All |
    Select-Object DisplayName, Description, IsHidden, CreatedDateTime
```

### Access Certification / Attestation

```
Certification campaign types:
- Manager certification: Manager reviews their team's access
- Owner certification: Resource owner reviews who has access
- User certification: User reviews their own access (self-service)
- Role certification: Who holds privileged roles?

Frequency by risk level:
Critical/Privileged: Quarterly
Normal business: Annually  
Read-only: Every 2 years
```

---

## 8. Conditional Access & Zero Trust Identity

### Conditional Access Framework

```
Conditions (IF):                    Controls (THEN):
- User/Group                   →    Require MFA
- Application                  →    Require compliant device
- Location (IP / country)      →    Require Hybrid AD join
- Device platform              →    Block access
- Sign-in risk (AI-scored)     →    Require password change
- User risk                    →    Limit session (no download)
- Client app type              →    Require approved app
```

### Recommended CA Policy Set

```
Policy 1: Require MFA for All Users
  - Users: All (exclude break-glass)
  - Apps: All
  - Grant: Require MFA

Policy 2: Block Legacy Authentication
  - Users: All
  - Conditions: Client apps = Legacy auth clients
  - Grant: Block

Policy 3: Require Compliant Device for Corporate Apps
  - Users: All
  - Apps: SharePoint, Exchange
  - Conditions: Device platforms = All
  - Grant: Require compliant device OR Hybrid AD join

Policy 4: Block Risky Sign-ins
  - Users: All
  - Conditions: Sign-in risk = High
  - Grant: Block (or require MFA + password reset)

Policy 5: High-Risk Users Must Reset Password
  - Users: All  
  - Conditions: User risk = High
  - Grant: Require MFA + password change

Policy 6: Admin MFA Every Time (No persistent sessions)
  - Users: All Global/Privileged Admins
  - Apps: All
  - Session: Sign-in frequency = Every time
  - Grant: Require MFA + compliant device

Policy 7: Block Unapproved Countries
  - Users: All (exclude break-glass)
  - Conditions: Locations = exclude approved countries
  - Grant: Block
```

### PowerShell: Conditional Access Audit

```powershell
Connect-MgGraph -Scopes "Policy.Read.All"

# List all CA policies
Get-MgIdentityConditionalAccessPolicy -All |
    Select-Object DisplayName, State, CreatedDateTime |
    Sort-Object State, DisplayName |
    Format-Table

# Export CA policies for documentation
$Policies = Get-MgIdentityConditionalAccessPolicy -All
$Policies | ConvertTo-Json -Depth 10 | 
    Out-File "CA_Policies_Backup_$(Get-Date -Format yyyyMMdd).json"

# Find policies not in "enabled" state
$Policies | Where-Object { $_.State -ne "enabled" } |
    Select-Object DisplayName, State |
    Format-Table
```

---

## 9. Service Account Management

### Service Account Best Practices

```
Naming convention:
  svc-[application]-[environment]@contoso.com
  Examples: svc-sqlbackup-prod, svc-sccm-corp, svc-monitoring-all

Requirements per service account:
□ Business owner documented
□ Password rotation schedule defined (or use gMSA)
□ Access documented (what it needs and why)
□ Annual review assigned
□ No interactive login allowed (logon as service only)
□ No MFA exclusions unless absolutely required with compensating controls
□ Dedicated account per application (no sharing)
```

### Group Managed Service Accounts (gMSA)

```powershell
# gMSAs: Passwords managed automatically by AD, 30-day rotation
# No manual password management needed

# Prerequisites: KDS root key exists
Add-KdsRootKey -EffectiveImmediately  # For lab only
Add-KdsRootKey -EffectiveTime ((Get-Date).AddHours(-10))  # Production

# Create gMSA
New-ADServiceAccount `
    -Name "gMSA-SQLBackup" `
    -DNSHostName "gMSA-SQLBackup.contoso.com" `
    -PrincipalsAllowedToRetrieveManagedPassword "SQLServer01$","SQLServer02$" `
    -Description "SQL Backup service account" `
    -Enabled $true

# Install gMSA on the server that needs it
Install-ADServiceAccount -Identity "gMSA-SQLBackup"
Test-ADServiceAccount -Identity "gMSA-SQLBackup"

# Configure service to use gMSA (in Services MMC or PowerShell)
$Service = Get-WmiObject Win32_Service -Filter "Name='SQLBackupSvc'"
$Service.Change($null,$null,$null,$null,$null,$null,"contoso\gMSA-SQLBackup$",$null)
```

### Service Account Audit

```powershell
# Find service accounts not using gMSA
Get-ADUser -Filter { ServicePrincipalName -like "*" } -Properties * |
    Where-Object { $_.ObjectClass -eq "user" } |
    Select-Object Name, SamAccountName, PasswordLastSet, PasswordNeverExpires |
    Format-Table

# Find accounts that haven't been used in 90+ days
$CutoffDate = (Get-Date).AddDays(-90)
Get-ADUser -Filter { 
    Enabled -eq $true -and LastLogonDate -lt $CutoffDate 
} -Properties LastLogonDate, PasswordLastSet |
    Where-Object { $_.SamAccountName -like "svc-*" } |
    Select-Object SamAccountName, LastLogonDate, PasswordLastSet |
    Export-Csv "Stale_ServiceAccounts.csv"
```

---

## 10. Identity Threat Detection

### Common Identity Attacks

| Attack | Description | Detection Signal |
|--------|-------------|-----------------|
| Password spray | Low-and-slow brute force across many accounts | Many failed logins from single IP, different accounts |
| Credential stuffing | Using leaked credential lists | Failed logins from Tor/proxies, unusual geography |
| MFA fatigue | Spamming push notifications | Many MFA denials, odd hours |
| Pass-the-Hash | Using NTLM hash without password | Lateral movement using hash, no Kerberos |
| Kerberoasting | Cracking service account tickets offline | SPN enumeration, large TGS ticket requests |
| Golden Ticket | Forged Kerberos ticket using KRBTGT hash | Tickets with unusual lifetime, invalid options |
| Token theft | Stealing OAuth tokens (cloud) | Sign-in from impossible travel after token issuance |

### Azure AD Identity Protection

```powershell
# Check risky users
Connect-MgGraph -Scopes "IdentityRiskyUser.Read.All","IdentityRiskEvent.Read.All"

Get-MgRiskyUser -All -Filter "riskState eq 'atRisk'" |
    Select-Object UserPrincipalName, RiskLevel, RiskState, RiskLastUpdatedDateTime |
    Format-Table

# Get risk detections
Get-MgRiskDetection -All -Filter "riskState eq 'atRisk'" |
    Select-Object UserPrincipalName, DetectionTimingType, RiskEventType, 
                  RiskLevel, IpAddress, Location |
    Sort-Object DetectedDateTime -Descending |
    Select-Object -First 20 |
    Format-Table

# Dismiss risk for a user (after investigation)
$User = Get-MgRiskyUser -Filter "userPrincipalName eq 'user@contoso.com'"
Invoke-MgDismissRiskyUser -UserIds @($User.Id)

# Confirm compromise (force password reset + revoke tokens)
Invoke-MgConfirmRiskyUserCompromised -UserIds @($User.Id)
```

### Identity Monitoring Alerts to Configure

```
Critical (immediate response):
□ Break-glass account sign-in
□ Global Admin role activation (especially outside business hours)
□ MFA method registration outside onboarding window
□ Bulk user deletion or disablement (>10 accounts in 5 minutes)
□ New Conditional Access policy created or modified
□ KRBTGT password reset
□ Domain trust creation

High (response within 1 hour):
□ Impossible travel sign-in
□ Sign-in from anonymous IP / Tor
□ High-risk sign-in detected by Identity Protection
□ User risk elevated to High
□ New Global/Security admin added
□ Service account password change

Medium (daily review):
□ Failed MFA attempts > threshold
□ New user with privileged role assignment
□ Guest user invitation spike
□ App permission grants (OAuth consent)
```
