# Active Directory Administration Guide

## Overview
Complete reference for Active Directory (AD) administration in enterprise environments. Covers user and group management, OU design, Group Policy, DNS integration, replication, trusts, and security hardening.

---

## Table of Contents
1. [AD Architecture & Design](#architecture)
2. [User Account Management](#users)
3. [Group Management](#groups)
4. [Organizational Units (OUs)](#ous)
5. [Group Policy Objects (GPOs)](#gpo)
6. [Service Accounts & Managed Accounts](#service-accounts)
7. [AD Trusts](#trusts)
8. [Replication Management](#replication)
9. [AD Security & Hardening](#security)
10. [PowerShell AD Automation](#powershell)
11. [AD Troubleshooting](#troubleshooting)
12. [AD Backup & Recovery](#backup)

---

## 1. AD Architecture & Design <a name="architecture"></a>

### Core Concepts
| Term | Description |
|---|---|
| Forest | Top-level container; separate forests = separate security boundaries |
| Domain | Administrative boundary within forest; DNS namespace |
| Tree | Group of domains sharing contiguous namespace |
| OU | Organizational Unit — container for users/computers/groups; GPO target |
| Site | Physical network location; controls replication and authentication traffic |
| Domain Controller | Server hosting AD DS role; authoritative for its domain |
| FSMO Roles | 5 special roles (PDC, RID, Infrastructure, Schema, Domain Naming Master) |
| Global Catalog | Domain controller with data for all domains in forest |

### FSMO Role Reference
| Role | Scope | Purpose |
|---|---|---|
| PDC Emulator | Domain | Time sync, password changes, GPO updates, legacy compatibility |
| RID Master | Domain | Allocates RID pools to DCs for SID creation |
| Infrastructure Master | Domain | Updates cross-domain group memberships |
| Schema Master | Forest | Controls AD schema changes |
| Domain Naming Master | Forest | Adds/removes domains from forest |

```powershell
# Check FSMO role holders
netdom query fsmo

# Or with PowerShell
Get-ADDomain | Select-Object PDCEmulator, RIDMaster, InfrastructureMaster
Get-ADForest | Select-Object SchemaMaster, DomainNamingMaster
```

### Functional Levels
- **Domain Functional Level**: Controls features available in the domain
- **Forest Functional Level**: Controls forest-wide features
- Higher = more features; lowest DC OS version determines max level

```powershell
# Check functional levels
Get-ADDomain | Select-Object DomainMode
Get-ADForest | Select-Object ForestMode

# Raise DFL (all DCs must be at target OS or higher)
Set-ADDomainMode -Identity "domain.com" -DomainMode Windows2016Domain
```

---

## 2. User Account Management <a name="users"></a>

### Creating Users

```powershell
# Create a single user
New-ADUser -Name "John Smith" `
  -GivenName "John" `
  -Surname "Smith" `
  -SamAccountName "jsmith" `
  -UserPrincipalName "jsmith@domain.com" `
  -DisplayName "John Smith" `
  -Department "IT Operations" `
  -Title "Systems Administrator" `
  -Office "HQ" `
  -EmailAddress "jsmith@domain.com" `
  -Path "OU=IT,OU=Users,DC=domain,DC=com" `
  -AccountPassword (ConvertTo-SecureString "P@ssw0rd!" -AsPlainText -Force) `
  -ChangePasswordAtLogon $true `
  -Enabled $true

# Bulk user creation from CSV
Import-Csv "C:\users.csv" | ForEach-Object {
  New-ADUser -Name "$($_.FirstName) $($_.LastName)" `
    -GivenName $_.FirstName `
    -Surname $_.LastName `
    -SamAccountName $_.Username `
    -UserPrincipalName "$($_.Username)@domain.com" `
    -Path "OU=Users,DC=domain,DC=com" `
    -AccountPassword (ConvertTo-SecureString $_.Password -AsPlainText -Force) `
    -ChangePasswordAtLogon $true `
    -Enabled $true
  Write-Host "Created: $($_.Username)"
}
```

### Modifying & Disabling Users

```powershell
# Modify user attributes
Set-ADUser -Identity jsmith -Title "Senior Systems Administrator" -Department "IT Security"

# Disable account (preferred over deletion for offboarding)
Disable-ADAccount -Identity jsmith

# Move to disabled OU
Move-ADObject -Identity (Get-ADUser jsmith).DistinguishedName -TargetPath "OU=Disabled,DC=domain,DC=com"

# Reset password
Set-ADAccountPassword -Identity jsmith -Reset -NewPassword (ConvertTo-SecureString "NewP@ss!" -AsPlainText -Force)
Set-ADUser -Identity jsmith -ChangePasswordAtLogon $true

# Unlock account
Unlock-ADAccount -Identity jsmith

# Full offboarding script
$user = "jsmith"
Disable-ADAccount -Identity $user
Get-ADUser $user -Properties MemberOf | Select-Object -ExpandProperty MemberOf | ForEach-Object {
  Remove-ADGroupMember -Identity $_ -Members $user -Confirm:$false
}
Move-ADObject -Identity (Get-ADUser $user).DistinguishedName -TargetPath "OU=Disabled,DC=domain,DC=com"
Set-ADUser -Identity $user -Description "Disabled: $(Get-Date -Format yyyy-MM-dd) - Offboarded"
Write-Host "User $user offboarded successfully"
```

### Finding & Auditing Users

```powershell
# Find inactive users (no login in 90 days)
$90DaysAgo = (Get-Date).AddDays(-90)
Get-ADUser -Filter {LastLogonDate -lt $90DaysAgo -and Enabled -eq $true} `
  -Properties LastLogonDate, Department | 
  Select-Object SamAccountName, DisplayName, LastLogonDate, Department |
  Sort-Object LastLogonDate

# Find users with password never expires
Get-ADUser -Filter {PasswordNeverExpires -eq $true -and Enabled -eq $true} `
  -Properties PasswordNeverExpires | 
  Select-Object SamAccountName, DisplayName, PasswordNeverExpires

# Find locked-out accounts
Search-ADAccount -LockedOut | Select-Object SamAccountName, Name, LockedOut, LastLogonDate

# Find disabled accounts still in groups
Get-ADUser -Filter {Enabled -eq $false} -Properties MemberOf | 
  Where-Object {$_.MemberOf.Count -gt 0} |
  Select-Object SamAccountName, @{N='Groups';E={($_.MemberOf | Get-ADGroup).Name -join "; "}}

# Find accounts expiring in next 7 days
Search-ADAccount -AccountExpiring -TimeSpan "7" | Select-Object SamAccountName, AccountExpirationDate
```

---

## 3. Group Management <a name="groups"></a>

### Group Types & Scopes
| Scope | Members | Use Case |
|---|---|---|
| Domain Local | Any domain/forest members | Assign permissions to local resources |
| Global | Same domain only | Organize users by role/department |
| Universal | Any domain in forest | Multi-domain resource access |

**Best Practice — AGDLP Model:**
```
Accounts → Global Groups → Domain Local Groups → Permissions

Example:
- Users "jsmith", "tjones" → Global group "GG_IT_Admins"
- "GG_IT_Admins" → Domain Local group "DL_FileServer01_Full"
- "DL_FileServer01_Full" → Full Control permission on FileServer01
```

### Group Operations

```powershell
# Create groups
New-ADGroup -Name "GG_IT_Admins" -GroupScope Global -GroupCategory Security -Path "OU=Groups,DC=domain,DC=com"
New-ADGroup -Name "DL_FileServer01_Full" -GroupScope DomainLocal -GroupCategory Security -Path "OU=Groups,DC=domain,DC=com"

# Add members
Add-ADGroupMember -Identity "GG_IT_Admins" -Members "jsmith","tjones"
Add-ADGroupMember -Identity "DL_FileServer01_Full" -Members "GG_IT_Admins"

# View members
Get-ADGroupMember -Identity "GG_IT_Admins" -Recursive | Select-Object SamAccountName, Name, objectClass

# View user's groups
Get-ADUser -Identity jsmith -Properties MemberOf | 
  Select-Object -ExpandProperty MemberOf | 
  ForEach-Object { (Get-ADGroup $_).Name } | Sort-Object

# Remove from group
Remove-ADGroupMember -Identity "GG_IT_Admins" -Members "jsmith" -Confirm:$false

# Find empty groups
Get-ADGroup -Filter * | Where-Object {(Get-ADGroupMember $_).Count -eq 0} | Select-Object Name, GroupScope
```

---

## 4. Organizational Units (OUs) <a name="ous"></a>

### OU Design Principles
1. Design OUs for **administration, not politics** — OUs are for GPO/delegation, not org chart mirroring
2. Depth of 4–5 levels maximum for complexity management
3. Separate OUs for computers and users
4. Create a "Staging" OU for new devices awaiting configuration

### Recommended OU Structure

```
domain.com
├── _Admin (protected from deletion)
│   ├── Service Accounts
│   ├── Admin Accounts
│   └── Groups
│       ├── Role Groups
│       └── Resource Groups
├── Computers
│   ├── Workstations
│   │   ├── Finance
│   │   ├── HR
│   │   └── IT
│   ├── Servers
│   │   ├── File Servers
│   │   ├── App Servers
│   │   └── Infrastructure
│   └── Staging
├── Users
│   ├── Finance
│   ├── HR
│   ├── IT
│   └── Contractors
└── Disabled
    ├── Users
    └── Computers
```

### OU Management

```powershell
# Create OU
New-ADOrganizationalUnit -Name "IT" -Path "OU=Users,DC=domain,DC=com" -ProtectedFromAccidentalDeletion $true

# Move objects to OU
Get-ADComputer -Filter {Name -like "WS*"} | Move-ADObject -TargetPath "OU=Workstations,OU=Computers,DC=domain,DC=com"

# Delegate control on an OU (example: Help Desk can reset passwords)
# Use ADSI Edit or Delegation of Control Wizard, or:
$ACL = Get-Acl "AD:\OU=Users,DC=domain,DC=com"
# (Full delegation scripting requires dsacls or AD module advanced use)

# List objects in OU
Get-ADUser -SearchBase "OU=IT,OU=Users,DC=domain,DC=com" -Filter * | Select-Object Name, SamAccountName

# Find computers not in expected OU
Get-ADComputer -Filter * | Where-Object {$_.DistinguishedName -notlike "*OU=Workstations*" -and $_.DistinguishedName -notlike "*OU=Servers*"} | Select-Object Name, DistinguishedName
```

---

## 5. Group Policy Objects (GPOs) <a name="gpo"></a>

### GPO Fundamentals
- GPOs are processed in this order: **Local → Site → Domain → OU**
- Later-applied GPOs win (OU GPOs override Domain GPOs by default)
- Use **Enforced** (No Override) to prevent child OUs from overriding
- Use **Block Inheritance** on an OU to ignore parent GPOs (except Enforced)
- Use **WMI Filters** to target specific OS versions or hardware
- Use **Security Filtering** to target specific groups

### Essential GPO Settings

**Password Policy (Domain-level only):**
```
Computer Configuration → Windows Settings → Security Settings → Account Policies → Password Policy

Minimum password length: 14+
Password complexity: Enabled
Maximum password age: 60-90 days
Minimum password age: 1 day
Enforce password history: 24
Reversible encryption: Disabled
```

**Account Lockout Policy:**
```
Computer Configuration → Windows Settings → Security Settings → Account Policies → Account Lockout

Lockout threshold: 5-10 attempts
Lockout duration: 15-30 minutes
Reset counter after: 15-30 minutes
```

**Fine-Grained Password Policies (PSOs)** — for admin accounts:
```powershell
# Create PSO for admin accounts (stricter policy)
New-ADFineGrainedPasswordPolicy -Name "PSO_Admins" `
  -Precedence 10 `
  -MinPasswordLength 16 `
  -PasswordHistoryCount 24 `
  -MaxPasswordAge "60.00:00:00" `
  -LockoutThreshold 3 `
  -LockoutDuration "01:00:00" `
  -LockoutObservationWindow "00:30:00" `
  -ComplexityEnabled $true

# Apply to admin group
Add-ADFineGrainedPasswordPolicySubject -Identity "PSO_Admins" -Subjects "GG_Domain_Admins"
```

### GPO Management Commands

```powershell
# Get all GPOs
Get-GPO -All | Select-Object DisplayName, GpoStatus, CreationTime, ModificationTime | Sort-Object ModificationTime -Descending

# Create a GPO
New-GPO -Name "Workstation Baseline Security"

# Link GPO to OU
New-GPLink -Name "Workstation Baseline Security" -Target "OU=Workstations,OU=Computers,DC=domain,DC=com"

# Get GPO report
Get-GPOReport -Name "Workstation Baseline Security" -ReportType HTML -Path "C:\Reports\GPO_Report.html"

# Backup all GPOs
Backup-GPO -All -Path "C:\GPO_Backups\$(Get-Date -Format yyyyMMdd)"

# Restore a GPO
Restore-GPO -Name "Workstation Baseline Security" -Path "C:\GPO_Backups\20240101"

# Force GPO refresh on remote computer
Invoke-GPUpdate -Computer "WS001" -Force -RandomDelayInMinutes 0

# Check GPO application on computer
gpresult /h C:\gpresult.html /f
Get-GPResultantSetOfPolicy -ReportType Html -Path C:\rsop.html
```

---

## 6. Service Accounts & Managed Accounts <a name="service-accounts"></a>

### Group Managed Service Accounts (gMSA)
gMSAs are the preferred service account type — password managed automatically by AD.

```powershell
# Prerequisites: KDS Root Key (one-time setup)
Add-KdsRootKey -EffectiveImmediately   # For test environments
# Add-KdsRootKey -EffectiveTime ((Get-Date).AddHours(-10))  # For prod (10hr wait)

# Create gMSA
New-ADServiceAccount -Name "svc-webapp01" `
  -DNSHostName "svc-webapp01.domain.com" `
  -PrincipalsAllowedToRetrieveManagedPassword "WEB-SERVER-01$" `
  -ServicePrincipalNames "HTTP/webapp01.domain.com","HTTP/webapp01"

# Install gMSA on target server (run on the server)
Install-ADServiceAccount -Identity "svc-webapp01"

# Verify
Test-ADServiceAccount -Identity "svc-webapp01"

# Configure service to use gMSA
# In Services: account name = DOMAIN\svc-webapp01$, leave password blank
```

### Traditional Service Account Best Practices
- Create in dedicated "Service Accounts" OU
- Apply Fine-Grained Password Policy (long, complex, no expiry)
- Use "Logon as a service" right only — not interactive logon
- Apply just enough permissions — never Domain Admins
- Set "User cannot change password"
- Document every service account and its purpose

---

## 7. AD Trusts <a name="trusts"></a>

### Trust Types
| Type | Direction | Transitive | Use Case |
|---|---|---|---|
| Parent-Child | Bidirectional | Yes | Automatic within domain tree |
| Tree-Root | Bidirectional | Yes | Automatic within forest |
| Forest | Bidirectional/One-way | Yes | Cross-forest access |
| External | One-way | No | Access to specific external domain |
| Shortcut | Bidirectional | Yes | Speed up cross-domain auth |
| Realm | One-way/Bidirectional | Optional | AD to non-Windows Kerberos |

```powershell
# View existing trusts
Get-ADTrust -Filter * | Select-Object Name, TrustType, TrustDirection, TrustAttributes

# Create forest trust (requires Enterprise Admin)
netdom trust forest2.com /domain:forest1.com /twoway /add /passwordt:<password>

# Verify trust
netdom trust forest2.com /domain:forest1.com /verify
```

---

## 8. Replication Management <a name="replication"></a>

### AD Sites & Replication

```powershell
# Check replication status
repadmin /showrepl
repadmin /replsummary
repadmin /showrepl * /csv > C:\repl_report.csv

# Check for replication errors
repadmin /showrepl * /errorsonly

# Force replication to all DCs
repadmin /syncall /AdeP

# Force replication from specific DC
repadmin /replicate DC02 DC01 "DC=domain,DC=com"

# Check DC health
dcdiag /test:replications
dcdiag /test:netlogons
dcdiag /test:sysvol
dcdiag /v /c /e > C:\dcdiag.txt
```

### Site Topology

```powershell
# View sites
Get-ADReplicationSite -Filter *

# View site links
Get-ADReplicationSiteLink -Filter * | Select-Object Name, Cost, ReplicationFrequencyInMinutes, SitesIncluded

# Create a site
New-ADReplicationSite -Name "Site-B-Chicago"

# Create site link
New-ADReplicationSiteLink -Name "HQ-Chicago" -SitesIncluded "Default-First-Site-Name","Site-B-Chicago" -Cost 100 -ReplicationFrequencyInMinutes 180

# Move subnet to site
New-ADReplicationSubnet -Name "10.2.0.0/16" -Site "Site-B-Chicago"
```

---

## 9. AD Security & Hardening <a name="security"></a>

### Privileged Access Management
```
Tier 0: Domain Controllers, AD, PKI, ADFS — most privileged
Tier 1: Servers (application, database, web)
Tier 2: Workstations, end-user devices

Rules:
- Tier 0 admins ONLY log into Tier 0 systems
- Tier 1 admins ONLY log into Tier 1 and below (never Tier 0)
- Tier 2 admins manage workstations only
- Admin accounts ≠ daily-use accounts
- Use PAWs (Privileged Access Workstations) for Tier 0/1 admin
```

### Key Security Settings to Audit

```powershell
# Find users with admin privileges
Get-ADGroupMember "Domain Admins" -Recursive | Select-Object SamAccountName, Name
Get-ADGroupMember "Enterprise Admins" -Recursive | Select-Object SamAccountName, Name
Get-ADGroupMember "Schema Admins" -Recursive | Select-Object SamAccountName, Name

# Find accounts that can perform DCSync (dangerous!)
# Users with "Replicating Directory Changes All" permission
# Use ADACLscanner tool for this

# Find Kerberoastable accounts (SPNs set on user accounts)
Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName |
  Select-Object SamAccountName, ServicePrincipalName

# Find ASREProastable accounts (no pre-auth required)
Get-ADUser -Filter {DoesNotRequirePreAuth -eq $true} -Properties DoesNotRequirePreAuth |
  Select-Object SamAccountName, DoesNotRequirePreAuth

# Ensure DCs only have required members in local admins
# Audit via GPO Restricted Groups

# Check for stale admin accounts
Get-ADUser -Filter {AdminCount -eq 1} -Properties AdminCount, LastLogonDate, Enabled |
  Where-Object {$_.LastLogonDate -lt (Get-Date).AddDays(-90) -or $_.Enabled -eq $false} |
  Select-Object SamAccountName, Enabled, LastLogonDate
```

### Recommended Security GPO Settings
```
Enable: Credential Guard
Enable: Protected Users group for privileged accounts
Enable: Audit logon events, account management, privilege use
Disable: NTLM v1 (network security: LAN Manager authentication level = NTLMv2 only)
Enable: SMB Signing (required)
Disable: Guest account
Restrict: Local admin accounts (LAPS for workstations)
Enable: Windows Defender for all clients
Configure: AppLocker or WDAC (application whitelisting)
```

---

## 10. PowerShell AD Automation <a name="powershell"></a>

### Import AD Module
```powershell
Import-Module ActiveDirectory

# If on non-DC machine, install RSAT first
Add-WindowsCapability -Online -Name Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0
```

### Bulk Operations Scripts

```powershell
# Report: All users, their groups, and last logon
Get-ADUser -Filter * -Properties MemberOf, LastLogonDate, Department, Title, Enabled |
  Select-Object @{N='Username';E={$_.SamAccountName}},
                DisplayName,
                Department,
                Title,
                Enabled,
                LastLogonDate,
                @{N='Groups';E={($_.MemberOf | ForEach-Object {(Get-ADGroup $_).Name}) -join "; "}} |
  Export-Csv "C:\Reports\UserGroupReport_$(Get-Date -Format yyyyMMdd).csv" -NoTypeInformation

# Disable all accounts in a specific OU
Get-ADUser -SearchBase "OU=Contractors,OU=Users,DC=domain,DC=com" -Filter {Enabled -eq $true} | 
  ForEach-Object {
    Disable-ADAccount -Identity $_
    Write-Host "Disabled: $($_.SamAccountName)"
  }

# Find and log all computers not seen in 60+ days
$60Days = (Get-Date).AddDays(-60)
Get-ADComputer -Filter {LastLogonDate -lt $60Days -and Enabled -eq $true} `
  -Properties LastLogonDate, OperatingSystem |
  Select-Object Name, LastLogonDate, OperatingSystem |
  Export-Csv "C:\Reports\StaleComputers.csv" -NoTypeInformation
```

---

## 11. AD Troubleshooting <a name="troubleshooting"></a>

### Common Issues & Fixes

```powershell
# Authentication failures - check lockout source
# Get DC where lockout originated
Get-ADDomainController -Filter * | ForEach-Object {
  Get-WinEvent -ComputerName $_.Name -FilterHashtable @{
    LogName='Security'; Id=4740
  } -ErrorAction SilentlyContinue |
  Select-Object TimeCreated, @{N='LockedUser';E={$_.Properties[0].Value}}, @{N='CallerPC';E={$_.Properties[1].Value}}
} | Sort-Object TimeCreated -Descending | Select-Object -First 20

# Kerberos issues - check time sync
w32tm /query /status
w32tm /resync /force
net time /domain

# Fix Netlogon issues
nltest /dsgetdc:domain.com
nltest /dbflag:0x2080ffff   # Enable debug logging
# Logs: C:\Windows\debug\netlogon.log

# Test DC connectivity from client
nltest /sc_verify:domain.com
Test-ComputerSecureChannel -Verbose

# Re-join domain if secure channel broken
Reset-ComputerMachinePassword -Server DC01 -Credential (Get-Credential)
# Or:
netdom resetpwd /server:DC01 /userd:DOMAIN\Admin /passwordd:Password
```

---

## 12. AD Backup & Recovery <a name="backup"></a>

### Active Directory Backup

```powershell
# Windows Server Backup (built-in)
# Install feature
Install-WindowsFeature Windows-Server-Backup

# Backup system state (includes AD DS, SYSVOL, registry)
wbadmin start systemstatebackup -backuptarget:E:

# Backup to network share
wbadmin start systemstatebackup -backuptarget:\\backupserver\ADBackups

# Verify backup
wbadmin get versions
```

### SYSVOL Recovery
```powershell
# Check SYSVOL replication status
Get-DfsrState -ComputerName DC01
dfsrdiag pollad

# If SYSVOL not sharing/replicating, authoritative restore:
# 1. Stop DFSR service on all DCs
# 2. On the authoritative DC:
Set-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Services\DFSR\Parameters\SysVols\Seeding SysVols\<domain>" -Name "SYSVOL Replication Enabled" -Value 1
# 3. Restart DFSR
# 4. Restart DFSR on other DCs
```

### Tombstone & Object Recovery
```powershell
# Recover deleted AD objects (within tombstone lifetime — default 180 days)
Get-ADObject -Filter {Deleted -eq $true} -IncludeDeletedObjects -Properties * | 
  Where-Object {$_.Name -like "*jsmith*"} |
  Select-Object Name, DistinguishedName, WhenChanged

# Restore deleted object
Restore-ADObject -Identity "<Object-GUID>"

# With Active Directory Recycle Bin enabled (requires 2008 R2+ FFL)
Get-ADObject -Filter {Deleted -eq $true -and Name -eq "jsmith"} -IncludeDeletedObjects | Restore-ADObject
```
