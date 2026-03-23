# Microsoft 365 Administration Guide

## Overview
Comprehensive M365 administration reference covering tenant management, Exchange Online, SharePoint, Teams, licensing, security, and compliance. Designed for IT administrators managing M365 environments.

---

## 1. Tenant Administration

### Admin Center Navigation
| Portal | URL | Purpose |
|--------|-----|---------|
| Microsoft 365 Admin Center | admin.microsoft.com | Users, licenses, billing |
| Exchange Admin Center | admin.exchange.microsoft.com | Email, mailboxes, transport |
| SharePoint Admin Center | <tenant>-admin.sharepoint.com | Sites, storage, policies |
| Teams Admin Center | admin.teams.microsoft.com | Teams policies, meetings |
| Entra ID (Azure AD) | entra.microsoft.com | Identity, MFA, Conditional Access |
| Intune | intune.microsoft.com | Device management |
| Defender | security.microsoft.com | Security, threat protection |
| Purview | compliance.microsoft.com | Compliance, DLP, retention |
| Service Health | admin.microsoft.com/adminportal/home#/servicehealth | Incident status |

### PowerShell Modules
```powershell
# Install required modules
Install-Module -Name ExchangeOnlineManagement -Force
Install-Module -Name Microsoft.Graph -Force
Install-Module -Name MicrosoftTeams -Force
Install-Module -Name Microsoft.Online.SharePoint.PowerShell -Force

# Connect
Connect-ExchangeOnline -UserPrincipalName admin@domain.com
Connect-MgGraph -Scopes "User.ReadWrite.All","Group.ReadWrite.All"
Connect-MicrosoftTeams
Connect-SPOService -Url https://tenant-admin.sharepoint.com

# Disconnect
Disconnect-ExchangeOnline -Confirm:$false
Disconnect-MgGraph
```

---

## 2. User Management

### User Lifecycle

**Create New User:**
```powershell
# Create user via Graph
$params = @{
    displayName = "Jane Smith"
    givenName = "Jane"
    surname = "Smith"
    userPrincipalName = "jsmith@domain.com"
    mailNickname = "jsmith"
    passwordProfile = @{
        forceChangePasswordNextSignIn = $true
        password = "TempP@ssw0rd!"
    }
    accountEnabled = $true
    usageLocation = "US"
}
New-MgUser -BodyParameter $params
```

**Bulk Create from CSV:**
```powershell
$users = Import-Csv "new_users.csv"
foreach ($user in $users) {
    New-MgUser -DisplayName $user.DisplayName `
               -UserPrincipalName $user.UPN `
               -MailNickname $user.Alias `
               -AccountEnabled $true `
               -UsageLocation "US" `
               -PasswordProfile @{
                   forceChangePasswordNextSignIn = $true
                   password = $user.TempPassword
               }
}
```

**Disable/Enable User:**
```powershell
# Disable
Update-MgUser -UserId "jsmith@domain.com" -AccountEnabled $false

# Enable
Update-MgUser -UserId "jsmith@domain.com" -AccountEnabled $true
```

**Offboarding Checklist (PowerShell):**
```powershell
$userUPN = "leavinguser@domain.com"

# 1. Block sign-in
Update-MgUser -UserId $userUPN -AccountEnabled $false

# 2. Revoke sessions
Revoke-MgUserSignInSession -UserId $userUPN

# 3. Remove from groups
$user = Get-MgUser -UserId $userUPN
$groups = Get-MgUserMemberOf -UserId $user.Id
foreach ($group in $groups) {
    Remove-MgGroupMemberByRef -GroupId $group.Id -DirectoryObjectId $user.Id
}

# 4. Convert to shared mailbox (Exchange)
Set-Mailbox $userUPN -Type Shared

# 5. Remove license (after converting mailbox)
# Done via admin center or Graph API
```

---

## 3. Licensing

### License Management

**View Licenses:**
```powershell
# Available licenses in tenant
Get-MgSubscribedSku | Select SkuPartNumber, ConsumedUnits, @{N="Available";E={$_.PrepaidUnits.Enabled - $_.ConsumedUnits}}

# User's assigned licenses
(Get-MgUser -UserId "user@domain.com" -Property AssignedLicenses).AssignedLicenses
```

**Assign License:**
```powershell
# Get SKU ID
$sku = Get-MgSubscribedSku | Where {$_.SkuPartNumber -eq "SPE_E3"}

# Assign
Set-MgUserLicense -UserId "user@domain.com" `
    -AddLicenses @{SkuId = $sku.SkuId} `
    -RemoveLicenses @()
```

**Remove License:**
```powershell
$license = (Get-MgUser -UserId "user@domain.com" -Property AssignedLicenses).AssignedLicenses[0]
Set-MgUserLicense -UserId "user@domain.com" `
    -AddLicenses @() `
    -RemoveLicenses @($license.SkuId)
```

**Common SKU Names:**
| SKU Part Number | Product |
|-----------------|---------|
| SPE_E3 | Microsoft 365 E3 |
| SPE_E5 | Microsoft 365 E5 |
| ENTERPRISEPREMIUM | Office 365 E5 |
| ENTERPRISEPACK | Office 365 E3 |
| DESKLESSPACK | Office 365 F3 |
| EMS | Enterprise Mobility + Security E3 |
| EMSPREMIUM | Enterprise Mobility + Security E5 |
| POWER_BI_PRO | Power BI Pro |
| PROJECTPREMIUM | Project Plan 5 |

### Group-Based Licensing
Best practice: assign licenses via Azure AD groups, not directly to users.
1. Entra ID > Groups > Create group
2. Assign license to group: Licenses blade
3. Add users to group → license auto-assigned
4. Remove from group → license auto-removed

---

## 4. Exchange Online

### Mailbox Management

**Mailbox Types:**
| Type | Use Case |
|------|---------|
| User Mailbox | Individual employee |
| Shared Mailbox | Team/department inbox (no license needed <50GB) |
| Room Mailbox | Conference room resource |
| Equipment Mailbox | Resource (projector, car, etc.) |
| Distribution Group | Email list (no mailbox storage) |
| Microsoft 365 Group | Modern group with mailbox + Teams + SharePoint |

**Create Shared Mailbox:**
```powershell
New-Mailbox -Shared -Name "IT Help Desk" `
    -DisplayName "IT Help Desk" `
    -Alias "helpdesk" `
    -PrimarySmtpAddress "helpdesk@domain.com"

# Grant access
Add-MailboxPermission -Identity "helpdesk@domain.com" `
    -User "jsmith@domain.com" `
    -AccessRights FullAccess `
    -InheritanceType All

# Grant Send As
Add-RecipientPermission -Identity "helpdesk@domain.com" `
    -Trustee "jsmith@domain.com" `
    -AccessRights SendAs
```

**Mailbox Size and Stats:**
```powershell
# Single mailbox
Get-MailboxStatistics -Identity "user@domain.com" | 
    Select DisplayName, TotalItemSize, ItemCount

# All mailboxes over 10GB
Get-Mailbox -ResultSize Unlimited | 
    Get-MailboxStatistics | 
    Where {$_.TotalItemSize -gt "10GB"} | 
    Select DisplayName, TotalItemSize | 
    Sort TotalItemSize -Descending
```

**Mailbox Permissions:**
```powershell
# View permissions
Get-MailboxPermission -Identity "user@domain.com"
Get-RecipientPermission -Identity "user@domain.com"

# Remove permission
Remove-MailboxPermission -Identity "user@domain.com" `
    -User "delegate@domain.com" `
    -AccessRights FullAccess
```

### Email Flow

**Mail Flow Rules (Transport Rules):**
```powershell
# Create disclaimer rule
New-TransportRule -Name "Email Disclaimer" `
    -ApplyHtmlDisclaimerText "<p style='font-size:8pt'>Confidential...</p>" `
    -ApplyHtmlDisclaimerFallbackAction Wrap `
    -ApplyHtmlDisclaimerLocation Append

# Block external email with specific word
New-TransportRule -Name "Block PII in Subject" `
    -SubjectContainsWords "SSN","Social Security" `
    -SentToScope NotInOrganization `
    -RejectMessageReasonText "Message blocked by policy"
```

**Message Trace:**
```powershell
# Search last 48 hours
Get-MessageTrace -SenderAddress "sender@external.com" `
    -RecipientAddress "user@domain.com" `
    -StartDate (Get-Date).AddDays(-2) `
    -EndDate (Get-Date)

# Get message trace details
Get-MessageTraceDetail -MessageTraceId <ID> -RecipientAddress "user@domain.com"
```

**Spam/Junk Configuration:**
```powershell
# View anti-spam policy
Get-HostedContentFilterPolicy | Select Name, BulkThreshold, SpamAction

# Whitelist sender
Set-HostedContentFilterPolicy -Identity Default `
    -AllowedSenders @{Add="trusted@vendor.com"}

# Whitelist domain
Set-HostedContentFilterPolicy -Identity Default `
    -AllowedSenderDomains @{Add="trusteddomain.com"}
```

### Email Authentication
```powershell
# Check DKIM
Get-DkimSigningConfig | Select Domain, Enabled, Status

# Enable DKIM for domain
Enable-DkimSigningConfig -Identity domain.com

# Rotate DKIM keys
Rotate-DkimSigningConfig -KeySize 2048 -Identity domain.com
```

**SPF Record (DNS TXT):**
```
v=spf1 include:spf.protection.outlook.com -all
```

**DMARC Record (DNS TXT for _dmarc.domain.com):**
```
v=DMARC1; p=reject; rua=mailto:dmarc@domain.com; ruf=mailto:dmarc@domain.com; pct=100
```

---

## 5. SharePoint Online

### Site Management

**Create Site:**
```powershell
# Team site
New-SPOSite -Url "https://tenant.sharepoint.com/sites/ProjectX" `
    -Title "Project X" `
    -Owner "admin@domain.com" `
    -StorageQuota 1024 `
    -Template "STS#3"

# Communication site
New-SPOSite -Url "https://tenant.sharepoint.com/sites/Intranet" `
    -Title "Intranet" `
    -Owner "admin@domain.com" `
    -Template "SITEPAGEPUBLISHING#0"
```

**Site Permissions:**
```powershell
# Get site admins
Get-SPOSiteGroup -Site "https://tenant.sharepoint.com/sites/ProjectX"

# Add site admin
Set-SPOUser -Site "https://tenant.sharepoint.com/sites/ProjectX" `
    -LoginName "user@domain.com" `
    -IsSiteCollectionAdmin $true

# Set external sharing
Set-SPOSite -Identity "https://tenant.sharepoint.com/sites/ProjectX" `
    -SharingCapability ExternalUserAndGuestSharing
```

**Storage Management:**
```powershell
# Check storage usage
Get-SPOSite -Limit All | Select Url, StorageUsageCurrent, StorageQuota | 
    Sort StorageUsageCurrent -Descending

# Increase quota
Set-SPOSite -Identity "https://tenant.sharepoint.com/sites/ProjectX" `
    -StorageQuota 5120
```

### External Sharing Policy
```powershell
# Tenant-wide setting
Set-SPOTenant -SharingCapability ExternalUserAndGuestSharing

# Allowed domains only
Set-SPOTenant -SharingCapability ExternalUserAndGuestSharing `
    -SharingAllowedDomainList "partner.com,vendor.org" `
    -SharingDomainRestrictionMode AllowList
```

---

## 6. Microsoft Teams Administration

### Teams Policies

**Meeting Policies:**
```powershell
# View policies
Get-CsTeamsMeetingPolicy | Select Identity, AllowCloudRecording, AllowTranscription

# Create custom policy
New-CsTeamsMeetingPolicy -Identity "Restricted Meetings" `
    -AllowCloudRecording $false `
    -AllowExternalParticipantGiveRequestControl $false `
    -AllowAnonymousUsersToJoinMeeting $false

# Assign to user
Grant-CsTeamsMeetingPolicy -PolicyName "Restricted Meetings" -Identity "user@domain.com"
```

**Messaging Policies:**
```powershell
New-CsTeamsMessagingPolicy -Identity "Standard Users" `
    -AllowGiphy $false `
    -AllowMemes $false `
    -AllowUserEditMessage $true `
    -AllowUserDeleteMessage $false

Grant-CsTeamsMessagingPolicy -PolicyName "Standard Users" -Identity "user@domain.com"
```

**Guest Access:**
```powershell
# Check guest access
Get-CsTeamsClientConfiguration | Select AllowGuestUser

# Enable/disable
Set-CsTeamsClientConfiguration -AllowGuestUser $true
```

### Teams Management

**Create a Team:**
```powershell
$team = New-Team -DisplayName "Project X" `
    -Description "Project X team" `
    -Visibility Private `
    -Owner "projectlead@domain.com"

# Add members
Add-TeamUser -GroupId $team.GroupId -User "member1@domain.com" -Role Member
Add-TeamUser -GroupId $team.GroupId -User "member2@domain.com" -Role Owner
```

**Teams Governance:**
- Set expiration policy for inactive teams
- Enable Teams templates for consistent provisioning
- Configure naming policies (prefix/suffix)
- Block specific words in team names

```powershell
# Naming policy
$policy = New-AzureADDirectorySetting
# Configure via Azure AD portal for naming policy
```

---

## 7. Multi-Factor Authentication

### MFA Management (Per-User vs Conditional Access)

**Per-User MFA (Legacy — avoid for new deployments):**
```powershell
# View MFA status
Get-MsolUser -All | Select UserPrincipalName, @{N="MFA";E={
    if ($_.StrongAuthenticationRequirements) {"Enabled"} else {"Disabled"}
}}

# Enable per-user MFA
$auth = New-Object -TypeName Microsoft.Online.Administration.StrongAuthenticationRequirement
$auth.RelyingParty = "*"
$auth.State = "Enabled"
Set-MsolUser -UserPrincipalName "user@domain.com" -StrongAuthenticationRequirements $auth
```

**Conditional Access MFA (Recommended):**
1. Entra ID > Security > Conditional Access
2. New Policy > Name: "Require MFA - All Users"
3. Users: All users (exclude break-glass accounts)
4. Cloud apps: All cloud apps
5. Grant: Require multi-factor authentication
6. Enable policy

**MFA Registration Report:**
```powershell
# Requires Microsoft.Graph
Connect-MgGraph -Scopes "Reports.Read.All"
Get-MgReportAuthenticationMethodUserRegistrationDetail | 
    Select UserPrincipalName, IsMfaRegistered, IsPasswordlessCapable |
    Export-Csv "mfa_report.csv" -NoTypeInformation
```

**Reset User MFA:**
1. Entra ID > Users > Select user > Authentication methods
2. Require re-register MFA: Click "Require re-register multifactor authentication"
3. Or delete specific methods listed

---

## 8. Security & Compliance

### Microsoft Defender for O365

**Safe Links:**
```powershell
# View Safe Links policy
Get-SafeLinksPolicy | Select Name, IsEnabled, DoNotAllowClickThrough

# Create policy
New-SafeLinksPolicy -Name "Corporate Safe Links" `
    -IsEnabled $true `
    -ScanUrls $true `
    -EnableForInternalSenders $true `
    -DoNotAllowClickThrough $true `
    -DeliverMessageAfterScan $true
```

**Safe Attachments:**
```powershell
# View policy
Get-SafeAttachmentPolicy

# Create policy
New-SafeAttachmentPolicy -Name "Corporate Safe Attachments" `
    -Enable $true `
    -Action DynamicDelivery `
    -Redirect $false
```

**Anti-Phishing:**
```powershell
Get-AntiPhishPolicy | Select Name, Enabled, ImpersonationProtectionState

New-AntiPhishPolicy -Name "Corporate Anti-Phish" `
    -EnableMailboxIntelligence $true `
    -EnableMailboxIntelligenceProtection $true `
    -EnableOrganizationDomainsProtection $true `
    -EnableTargetedUserProtection $true
```

### Audit Logs

**Search Audit Log:**
```powershell
Connect-ExchangeOnline

# Search for specific activity
Search-UnifiedAuditLog -StartDate (Get-Date).AddDays(-7) `
    -EndDate (Get-Date) `
    -Operations "FileDownloaded","FileAccessed" `
    -ResultSize 1000 |
    Select CreationDate, UserIds, Operations, AuditData

# Admin activity
Search-UnifiedAuditLog -StartDate (Get-Date).AddDays(-30) `
    -EndDate (Get-Date) `
    -RecordType ExchangeAdmin `
    -ResultSize 5000 |
    Export-Csv "exchange_admin_audit.csv" -NoTypeInformation
```

### Data Loss Prevention (DLP)

**Create DLP Policy:**
1. Purview > Data loss prevention > Policies > Create policy
2. Choose template (Financial, HIPAA, GDPR, etc.) or custom
3. Define locations (Exchange, SharePoint, Teams, Endpoints)
4. Configure rules:
   - Content contains: sensitive info types
   - Actions: block, notify, audit
5. Policy mode: Simulate first, then Enforce

**Common Sensitive Info Types:**
- Credit Card Numbers
- Social Security Numbers
- Bank Account Numbers
- Passport Numbers
- Health Insurance Claims
- Drug Enforcement Agency (DEA) Number

---

## 9. Email Migration

### Cutover Migration (Small <150 mailboxes)
1. Configure Outlook Anywhere on Exchange on-premises
2. Create migration batch in EAC
3. Synchronize mailboxes
4. Update MX records
5. Complete migration, delete batch

### Hybrid Migration (Large/Complex)
1. Deploy Azure AD Connect
2. Run Hybrid Configuration Wizard
3. Install hybrid connectors
4. Migrate mailboxes in waves using New-MigrationBatch

### PST Import
```powershell
# Network upload method
# 1. Upload PSTs to Azure Blob Storage
# 2. Create mapping CSV
# 3. Create import job in Purview

# CSV mapping format:
# Workload,FilePath,Name,Mailbox,IsArchive,TargetRootFolder,ContentCodePage,SPFileContainer,SPManifestContainer,SPSiteUrl
```

---

## 10. Reporting & Monitoring

### Usage Reports
```powershell
# Email activity
Get-MgReportEmailActivityUserDetail -Period D30 -OutFile "email_activity.csv"

# OneDrive usage
Get-MgReportOneDriveUsageAccountDetail -Period D30 -OutFile "onedrive_usage.csv"

# Teams activity
Get-MgReportTeamsUserActivityUserDetail -Period D30 -OutFile "teams_activity.csv"

# Inactive users (no activity 30 days)
Get-MgReportM365AppUserDetail -Period D30 | 
    ConvertFrom-Csv | 
    Where {$_."Last Activity Date" -eq ""} |
    Select "User Principal Name","Display Name","Last Activity Date"
```

### Service Health Monitoring
```powershell
# Current incidents
Get-MgServiceAnnouncementIssue -Filter "status ne 'resolved'" |
    Select StartDateTime, Title, Status, Service

# Set up health alerts in Admin Center:
# Health > Service health > Customize notifications
```

---

## 11. Best Practices & Governance

### Tenant Hardening Checklist
- [ ] Enable Security Defaults or Conditional Access
- [ ] MFA required for all users and admins
- [ ] Dedicated admin accounts (no licenses or mailboxes)
- [ ] Break-glass accounts documented and tested
- [ ] DMARC/DKIM/SPF configured and enforced
- [ ] Safe Attachments and Safe Links enabled
- [ ] Audit logging enabled (minimum 90 days, 1 year recommended)
- [ ] Legacy authentication blocked
- [ ] External sharing restricted by policy
- [ ] Privileged Identity Management (PIM) for admin roles
- [ ] Regular access reviews scheduled
- [ ] DLP policies covering major data types

### Role Assignments (Least Privilege)
| Role | Use Case |
|------|---------|
| Global Administrator | Break-glass only |
| Exchange Administrator | Email administration |
| SharePoint Administrator | SharePoint/OneDrive |
| Teams Administrator | Teams policy management |
| User Administrator | User/license management |
| Security Administrator | Defender, security policies |
| Compliance Administrator | Purview, DLP |
| Helpdesk Administrator | Password reset, license assignment |
| Reports Reader | Reporting access only |

---

*Last Updated: 2025 | IT Operations Documentation Library*
