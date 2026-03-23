# Group Policy Deep Dive — Enterprise Administration Guide

## Table of Contents
1. [GPO Architecture & Processing](#gpo-architecture)
2. [GPMC Administration](#gpmc-administration)
3. [Computer vs. User Policies](#computer-vs-user)
4. [Security Baseline GPOs](#security-baselines)
5. [Software Deployment via GPO](#software-deployment)
6. [Scripts & Logon Automation](#scripts-logon)
7. [Loopback Processing](#loopback-processing)
8. [WMI Filters](#wmi-filters)
9. [GPO Troubleshooting](#gpo-troubleshooting)
10. [PowerShell Group Policy Automation](#powershell-gpo)
11. [GPO Migration & Backup](#gpo-migration)
12. [Common Enterprise GPO Templates](#enterprise-templates)

---

## 1. GPO Architecture & Processing {#gpo-architecture}

### Processing Order (LSDOU)
```
1. Local Policy          — Applied first, lowest precedence
2. Site GPOs             — Linked to AD site
3. Domain GPOs           — Linked to domain root
4. OU GPOs               — Applied closest to object = highest precedence
   └── Child OU GPOs     — Override parent OU GPOs
```

### Key Processing Rules
| Rule | Description |
|------|-------------|
| **Last Write Wins** | Within same link level, highest link order wins |
| **Block Inheritance** | Prevents parent GPOs from flowing down (can be overridden by Enforced) |
| **Enforce (No Override)** | Forces GPO to apply regardless of Block Inheritance |
| **Loopback Processing** | Applies user GPOs based on computer location, not user OU |
| **Security Filtering** | Applies GPO only to specific users/computers/groups |
| **WMI Filtering** | Applies GPO only when WMI query returns TRUE |

### GPO Components
```
GPO = GPC (Group Policy Container) + GPT (Group Policy Template)
├── GPC: Stored in AD — cn=Policies,cn=System,DC=domain,DC=com
└── GPT: Stored in SYSVOL — \\domain\SYSVOL\domain\Policies\{GUID}\
    ├── Machine\         — Computer settings
    │   ├── Scripts\
    │   └── Microsoft\Windows NT\SecEdit\
    └── User\            — User settings
        └── Scripts\
```

### Refresh Intervals
| Target | Default Interval | Notes |
|--------|-----------------|-------|
| Computers | 90 min ± 30 min offset | Background refresh |
| Domain Controllers | 5 minutes | More frequent |
| Security settings | 16 hours (even if no change) | Always refreshes |
| Forced refresh | `gpupdate /force` | Immediate |

---

## 2. GPMC Administration {#gpmc-administration}

### GPMC Installation
```powershell
# Windows Server
Install-WindowsFeature -Name GPMC

# Windows 10/11 (RSAT)
Add-WindowsCapability -Online -Name Rsat.GroupPolicy.Management.Tools~~~~0.0.1.0
```

### Creating and Linking GPOs
```powershell
# Create new GPO
New-GPO -Name "Workstation Security Baseline" -Domain "corp.contoso.com" -Comment "CIS Level 1 Baseline"

# Link to OU
New-GPLink -Name "Workstation Security Baseline" -Target "OU=Workstations,DC=corp,DC=contoso,DC=com" -LinkEnabled Yes -Order 1

# Link with Enforce
New-GPLink -Name "Domain Security Policy" -Target "DC=corp,DC=contoso,DC=com" -Enforced Yes

# Disable GPO link without removing
Set-GPLink -Name "Workstation Security Baseline" -Target "OU=Workstations,DC=corp,DC=contoso,DC=com" -LinkEnabled No
```

### GPO Settings Management
```powershell
# Set a registry-based policy
Set-GPRegistryValue -Name "Workstation Security Baseline" `
  -Key "HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System" `
  -ValueName "EnableLUA" -Type DWord -Value 1

# Remove a registry value from GPO
Remove-GPRegistryValue -Name "Workstation Security Baseline" `
  -Key "HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System" `
  -ValueName "EnableLUA"

# Get all registry values in a GPO
Get-GPRegistryValue -Name "Workstation Security Baseline" `
  -Key "HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System"
```

### GPO Reporting
```powershell
# HTML report for single GPO
Get-GPOReport -Name "Workstation Security Baseline" -ReportType HTML -Path "C:\GPOReports\WSBaseline.html"

# XML report (for scripting)
Get-GPOReport -Name "Workstation Security Baseline" -ReportType XML -Path "C:\GPOReports\WSBaseline.xml"

# Report all GPOs in domain
Get-GPO -All | ForEach-Object {
    Get-GPOReport -Guid $_.Id -ReportType HTML -Path "C:\GPOReports\$($_.DisplayName).html"
}
```

---

## 3. Computer vs. User Policies {#computer-vs-user}

### Scope Differences
| Aspect | Computer Policy | User Policy |
|--------|----------------|-------------|
| Applied to | Computer object in AD | User object in AD |
| When applied | Computer startup, background refresh | User logon, background refresh |
| HIVE | HKEY_LOCAL_MACHINE | HKEY_CURRENT_USER |
| OU determines target | Computer OU | User OU (unless loopback) |
| Examples | Firewall, BitLocker, WSUS | Desktop wallpaper, IE settings, drive maps |

### Best Practice: Separate GPOs
```
Corp.Contoso.com
├── Domain GPO: "Default Domain Policy" (password policy only)
├── OU: Workstations
│   ├── GPO: "Workstation Computer Baseline" (Computer Configuration only)
│   └── GPO: "Workstation User Settings" (User Configuration only)
└── OU: Servers
    └── GPO: "Server Computer Baseline" (Computer Configuration only)
```

**Always create separate GPOs for computer and user settings** — easier to troubleshoot and manage.

### Disabling Unused Configuration
```powershell
# Disable User Configuration in a computer-only GPO (speeds up processing)
$gpo = Get-GPO -Name "Workstation Computer Baseline"
$gpo.GpoStatus = "UserSettingsDisabled"

# Disable Computer Configuration in a user-only GPO
$gpo = Get-GPO -Name "Workstation User Settings"
$gpo.GpoStatus = "ComputerSettingsDisabled"

# All values: AllSettingsEnabled, UserSettingsDisabled, ComputerSettingsDisabled, AllSettingsDisabled
```

---

## 4. Security Baseline GPOs {#security-baselines}

### Microsoft Security Compliance Toolkit
Download: https://www.microsoft.com/en-us/download/details.aspx?id=55319

```powershell
# Import Microsoft security baseline using Policy Analyzer / LGPO.exe
# Extract SCT, then:
.\Baseline-LocalInstall.ps1 -Win11NonDomainJoined    # Local
# For domain: Import the included GPO backups via GPMC
```

### CIS Benchmark Implementation — Key Settings

#### Account Policies (Domain Level Only)
```
Computer Config > Windows Settings > Security Settings > Account Policies

Password Policy:
  Enforce password history: 24
  Maximum password age: 60 days
  Minimum password age: 1 day
  Minimum password length: 14 characters
  Complexity requirements: Enabled
  Reversible encryption: Disabled

Account Lockout:
  Lockout duration: 15 minutes
  Lockout threshold: 5 attempts
  Observation window: 15 minutes
```

#### Local Security Options
```
Computer Config > Windows Settings > Security Settings > Local Policies > Security Options

Interactive logon: Do not display last user name: Enabled
Interactive logon: Machine inactivity limit: 900 seconds
Network access: Do not allow anonymous enumeration of SAM accounts: Enabled
Network access: Restrict anonymous access to Named Pipes and Shares: Enabled
Network security: LAN Manager authentication level: Send NTLMv2 only, refuse LM & NTLM
Network security: LDAP client signing requirements: Negotiate signing
User Account Control: Admin Approval Mode for Built-in Administrator: Enabled
User Account Control: Behavior of elevation prompt for administrators: Prompt for credentials
User Account Control: Run all administrators in Admin Approval Mode: Enabled
```

#### Audit Policy (Advanced)
```
Computer Config > Windows Settings > Security Settings > Advanced Audit Policy

Account Logon:
  Credential Validation: Success, Failure
  Kerberos Authentication: Success, Failure

Account Management:
  Computer Account Management: Success, Failure
  Security Group Management: Success, Failure
  User Account Management: Success, Failure

Logon/Logoff:
  Logon: Success, Failure
  Logoff: Success
  Account Lockout: Failure
  Special Logon: Success

Object Access:
  File System: Failure
  Registry: Failure

Policy Change:
  Audit Policy Change: Success, Failure
  Authentication Policy Change: Success

Privilege Use:
  Sensitive Privilege Use: Success, Failure

System:
  Security System Extension: Success, Failure
  System Integrity: Success, Failure
```

#### Windows Firewall via GPO
```
Computer Config > Windows Settings > Security Settings > Windows Defender Firewall

Domain Profile:
  Firewall state: On
  Inbound connections: Block
  Outbound connections: Allow
  Log dropped packets: Yes (4096 KB)

Private/Public Profiles: Same settings
```

---

## 5. Software Deployment via GPO {#software-deployment}

### MSI Deployment
```
Computer Config > Software Settings > Software Installation
  → New Package → assign/publish/advanced

Requirements:
- Package must be MSI (or MST transform)
- Must be on UNC share accessible to computer account: \\fileserver\Software\App\app.msi
- Use Assigned (computers/users) or Published (users only, via Add/Remove Programs)
```

### Software Deployment UNC Share Permissions
```
Share Permissions: Domain Computers — Read
NTFS Permissions:  Domain Computers — Read & Execute
                   Domain Admins — Full Control
```

### Startup/Shutdown Scripts vs. Logon/Logoff
| Script Type | Runs As | Timing |
|-------------|---------|--------|
| Startup | Local System | Before logon |
| Shutdown | Local System | After logoff |
| Logon | Logged-in user | At user logon |
| Logoff | Logged-in user | At user logoff |

```
Computer Config > Windows Settings > Scripts > Startup/Shutdown
User Config > Windows Settings > Scripts > Logon/Logoff
```

---

## 6. Scripts & Logon Automation {#scripts-logon}

### Drive Mapping via GPO Preferences
```
User Config > Preferences > Windows Settings > Drive Maps
  Action: Update (idempotent — won't fail if already mapped)
  Location: \\fileserver\shares\%username%
  Label: Home Drive
  Drive: H:
  Reconnect: Yes
  
Item-level targeting:
  - Security Group member: "CN=Sales,OU=Groups,DC=corp,DC=contoso,DC=com"
  - OS version
  - Site name
```

### Printer Deployment via GPO Preferences
```
Computer Config > Preferences > Control Panel Settings > Printers
  Action: Update
  Share path: \\printserver\HP-Color-3F
  Set as default: Yes (for specific group via ILT)
```

### Environment Variables
```
User Config > Preferences > Windows Settings > Environment
  Action: Update
  Name: DEPT_SHARE
  Value: \\fileserver\dept\finance
```

### PowerShell Logon Script Example
```powershell
# \\corp\SYSVOL\corp.contoso.com\scripts\UserLogon.ps1
# Deployed via: User Config > Windows Settings > Scripts > Logon (PowerShell tab)

# Map drives based on group membership
$user = [System.Security.Principal.WindowsIdentity]::GetCurrent().Name
$adUser = Get-ADUser -Identity ($user.Split('\')[1]) -Properties MemberOf

if ($adUser.MemberOf -match "CN=Finance") {
    New-PSDrive -Name "F" -PSProvider FileSystem -Root "\\fileserver\Finance" -Persist
}

# Set wallpaper
$RegPath = "HKCU:\Control Panel\Desktop"
Set-ItemProperty -Path $RegPath -Name Wallpaper -Value "\\fileserver\wallpaper\corp_wallpaper.jpg"
RUNDLL32.EXE user32.dll, UpdatePerUserSystemParameters

# Map default printer
(New-Object -ComObject WScript.Network).SetDefaultPrinter("\\printserver\Reception-HP")
```

---

## 7. Loopback Processing {#loopback-processing}

### What It Does
Normally, User Configuration GPOs are applied based on the **user's OU**. Loopback applies User Configuration GPOs based on the **computer's OU** instead.

### Use Cases
- Kiosk machines — apply locked-down user settings regardless of who logs in
- Lab computers — restrict applications for all users
- Terminal Servers / RDS — apply session-specific user settings
- Conference room computers

### Modes
| Mode | Behavior |
|------|---------|
| **Replace** | Only use User Config GPOs from the computer's OU. Ignore user's own OU GPOs |
| **Merge** | Apply user's OU GPOs AND computer OU User Config GPOs. Computer's GPOs win on conflict |

```
Computer Config > Administrative Templates > System > Group Policy
  → Configure user Group Policy loopback processing mode: Enabled
  → Mode: Replace (for kiosks) or Merge (for RDS)
```

---

## 8. WMI Filters {#wmi-filters}

### Common WMI Filter Queries

```
# Windows 10 only
SELECT * FROM Win32_OperatingSystem WHERE Version LIKE "10.0%" AND ProductType = "1"

# Windows 11 only
SELECT * FROM Win32_OperatingSystem WHERE Version LIKE "10.0.2%" AND ProductType = "1"

# 64-bit OS only
SELECT * FROM Win32_Processor WHERE AddressWidth = "64"

# Laptops only (uses battery)
SELECT * FROM Win32_Battery

# Desktops only
SELECT * FROM Win32_SystemEnclosure WHERE ChassisTypes = {3} OR ChassisTypes = {4} OR ChassisTypes = {6} OR ChassisTypes = {7}

# Specific model
SELECT * FROM Win32_ComputerSystem WHERE Model LIKE "%ThinkPad%"

# RAM >= 8GB
SELECT * FROM Win32_PhysicalMemoryArray WHERE MaxCapacity >= 8388608

# Domain member
SELECT * FROM Win32_ComputerSystem WHERE DomainRole >= 1
```

### Creating WMI Filters via PowerShell
```powershell
# Create WMI filter for Windows 10/11
$msWMICreationDate = (Get-Date).ToUniversalTime().ToString("yyyyMMddHHmmss.ffffff-000")
$wmifilter = [ADSI]"LDAP://CN=WMI Filters,CN=System,$((Get-ADDomain).DistinguishedName)"

$newFilter = $wmifilter.Create("msWMI-Som","CN=Windows10Filter")
$newFilter.Put("msWMI-Name","Windows 10 Workstations")
$newFilter.Put("msWMI-Parm1","Targets Windows 10 only")
$newFilter.Put("msWMI-Parm2",'1;3;10;30;WQL;root\CIMv2;SELECT * FROM Win32_OperatingSystem WHERE Version LIKE "10.0%" AND ProductType = "1";')
$newFilter.Put("msWMI-Author","admin@corp.contoso.com")
$newFilter.Put("msWMI-CreationDate",$msWMICreationDate)
$newFilter.Put("msWMI-ChangeDate",$msWMICreationDate)
$newFilter.SetInfo()
```

---

## 9. GPO Troubleshooting {#gpo-troubleshooting}

### Troubleshooting Workflow
```
1. gpresult /r             — Quick summary of applied GPOs
2. gpresult /h C:\gp.html  — Full HTML report with details
3. rsop.msc                — GUI Resultant Set of Policy
4. Event Viewer > Applications and Services Logs > Microsoft > Windows > Group Policy
5. Trace logging (advanced)
```

### gpresult Commands
```cmd
# Current user and computer
gpresult /r

# Verbose (shows all applied and filtered settings)
gpresult /v

# HTML report
gpresult /h C:\Reports\gpresult.html /f

# For a specific user on a remote computer
gpresult /s COMPUTERNAME /u DOMAIN\username /r

# Computer scope only
gpresult /r /scope computer

# User scope only
gpresult /r /scope user
```

### Common GPO Issues & Fixes

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| GPO not applying to computer | Computer not in correct OU | Move computer account to right OU |
| GPO shows in GPMC but not gpresult | Security filtering issue | Check "Apply Group Policy" permission on GPO |
| GPO applies to wrong users | Security filtering too broad | Add specific group to security filter, remove Authenticated Users |
| Settings not sticking | Preference vs Policy difference | Switch to Policy where available |
| New GPO not applying | Replication not complete | Wait 15 min or force replication: `repadmin /syncall` |
| GPO applies but setting doesn't work | Conflicting local policy | Use `gpedit.msc` to check local overrides |
| Slow logon | Too many GPOs or large GPOs | Consolidate GPOs, check SYSVOL replication |
| WMI filter blocking unexpectedly | WMI query incorrect | Test query manually: `wmic /node:PCNAME` |

### Testing WMI Filter on Target Computer
```cmd
# Test WMI query from command line (run on target machine)
wmic /namespace:\\root\cimv2 path Win32_OperatingSystem where "Version like '10.0%' and ProductType='1'" get Caption

# PowerShell
Get-WmiObject -Query "SELECT * FROM Win32_OperatingSystem WHERE Version LIKE '10.0%' AND ProductType = '1'"
```

### GPO Replication Verification
```powershell
# Check SYSVOL replication status
repadmin /showrepl

# Check if SYSVOL is healthy on all DCs
Get-ADDomainController -Filter * | ForEach-Object {
    $dc = $_.HostName
    $sysvol = "\\$dc\SYSVOL"
    $accessible = Test-Path $sysvol
    [PSCustomObject]@{
        DC = $dc
        SYSVOL_Accessible = $accessible
    }
}

# Force GP replication
Invoke-GPUpdate -Computer "WORKSTATION01" -Force -RandomDelayInMinutes 0
```

### GPO Event Log Analysis
```powershell
# Get recent Group Policy events (errors and warnings)
Get-WinEvent -LogName "Microsoft-Windows-GroupPolicy/Operational" |
    Where-Object {$_.LevelDisplayName -in @("Error","Warning")} |
    Select-Object TimeCreated, Id, Message |
    Format-List

# Most common event IDs
# 4016 - Starting Computer/User GP processing
# 4017 - Downloading computer settings
# 5312 - List of applicable GPOs
# 7016 - CSE completed without errors
# 7017 - CSE failed with errors
# 8004 - GPO not applied (filtered)
```

---

## 10. PowerShell Group Policy Automation {#powershell-gpo}

### Bulk GPO Inventory
```powershell
# Full GPO inventory report
$gpos = Get-GPO -All
$report = foreach ($gpo in $gpos) {
    $links = (Get-GPOReport -Guid $gpo.Id -ReportType XML) -as [xml]
    $linkedTo = $links.GPO.LinksTo | ForEach-Object { $_.SOMPath }
    
    [PSCustomObject]@{
        Name           = $gpo.DisplayName
        GUID           = $gpo.Id
        Status         = $gpo.GpoStatus
        CreationTime   = $gpo.CreationTime
        ModifiedTime   = $gpo.ModificationTime
        ComputerEnabled = ($gpo.GpoStatus -ne "ComputerSettingsDisabled" -and $gpo.GpoStatus -ne "AllSettingsDisabled")
        UserEnabled    = ($gpo.GpoStatus -ne "UserSettingsDisabled" -and $gpo.GpoStatus -ne "AllSettingsDisabled")
        LinkedTo       = ($linkedTo -join "; ")
        WMIFilter      = $gpo.WmiFilter.Name
    }
}
$report | Export-Csv "C:\Reports\GPO_Inventory.csv" -NoTypeInformation
Write-Host "Exported $($report.Count) GPOs" -ForegroundColor Green
```

### Find Unlinked GPOs
```powershell
Get-GPO -All | ForEach-Object {
    $xml = [xml](Get-GPOReport -Guid $_.Id -ReportType XML)
    if (-not $xml.GPO.LinksTo) {
        [PSCustomObject]@{
            Name     = $_.DisplayName
            GUID     = $_.Id
            Modified = $_.ModificationTime
        }
    }
} | Format-Table -AutoSize
```

### Clone GPO Settings
```powershell
# Backup source GPO
Backup-GPO -Name "Workstation Security Baseline" -Path "C:\GPOBackups"

# Restore as new GPO name
$backup = Get-ChildItem "C:\GPOBackups" | Sort-Object LastWriteTime -Descending | Select-Object -First 1
Restore-GPO -Name "Workstation Security Baseline v2" -Path "C:\GPOBackups" -BackupId $backup.Name
```

### Audit GPO Permissions
```powershell
# Find GPOs where Authenticated Users does NOT have Apply permission
Get-GPO -All | ForEach-Object {
    $gpo = $_
    $acl = Get-GPPermission -Guid $gpo.Id -All
    $hasApply = $acl | Where-Object {
        $_.Trustee.Name -eq "Authenticated Users" -and $_.Permission -eq "GpoApply"
    }
    if (-not $hasApply) {
        Write-Warning "GPO '$($gpo.DisplayName)' — no Authenticated Users Apply permission"
    }
}
```

---

## 11. GPO Migration & Backup {#gpo-migration}

### Backup All GPOs
```powershell
$backupPath = "C:\GPOBackups\$(Get-Date -Format 'yyyy-MM-dd')"
New-Item -ItemType Directory -Path $backupPath -Force

Get-GPO -All | ForEach-Object {
    $folderName = $_.DisplayName -replace '[\\/:*?"<>|]', '_'
    $gpoPath = Join-Path $backupPath $folderName
    Backup-GPO -Guid $_.Id -Path $backupPath
    Write-Host "Backed up: $($_.DisplayName)" -ForegroundColor Green
}

Write-Host "All GPOs backed up to $backupPath" -ForegroundColor Cyan
```

### Restore GPO from Backup
```powershell
# List available backups
Get-GPOBackup -Path "C:\GPOBackups\2024-11-01" | Select-Object DisplayName, BackupId, Timestamp

# Restore specific GPO (overwrites existing)
Restore-GPO -Name "Workstation Security Baseline" -Path "C:\GPOBackups\2024-11-01"

# Restore to NEW GPO name (won't overwrite)
Import-GPO -BackupGpoName "Workstation Security Baseline" `
           -Path "C:\GPOBackups\2024-11-01" `
           -TargetName "Workstation Security Baseline RESTORED" `
           -CreateIfNeeded
```

### Migration Between Domains
```powershell
# Use Migration Table for UNC paths and security principals
# Create migration table in GPMC: Tools > Open Migration Table Editor

# Import using migration table
Import-GPO -BackupGpoName "Workstation Security Baseline" `
           -Path "C:\GPOBackups" `
           -TargetName "Workstation Security Baseline" `
           -MigrationTable "C:\GPOMigration\migration.migtable" `
           -CreateIfNeeded
```

---

## 12. Common Enterprise GPO Templates {#enterprise-templates}

### Recommended GPO Architecture
```
Domain Root:
  └── "Default Domain Policy"      — Password policy, account lockout ONLY
  └── "Domain-wide Security"       — Audit policy, Kerberos settings

OU: Computers
  ├── OU: Workstations
  │     └── "Workstation Computer Baseline"  — OS hardening, firewall, WSUS
  │     └── "Workstation User Standard"      — Desktop settings, drive maps
  ├── OU: Laptops
  │     └── "Laptop Computer Baseline"       — + BitLocker, VPN
  ├── OU: Servers
  │     └── "Server Computer Baseline"       — Server hardening
  └── OU: Kiosks
        └── "Kiosk Computer Policy"          — Locked-down + loopback replace

OU: Users
  ├── "All Users Baseline"          — Common user settings
  ├── OU: IT Staff
  │     └── "IT Staff Settings"     — Admin tools, elevated settings
  └── OU: Executives
        └── "Executive Settings"    — Minimal restrictions
```

### Quick Reference: Key Registry Path Mappings
| GPO Setting Area | Registry Location |
|-----------------|-------------------|
| Password/lockout | AD attribute (not registry) |
| User Account Control | HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System |
| Windows Update | HKLM\Software\Policies\Microsoft\Windows\WindowsUpdate |
| Windows Firewall | HKLM\Software\Policies\Microsoft\WindowsFirewall |
| IE/Edge settings | HKCU\Software\Policies\Microsoft\Internet Explorer |
| Remote Desktop | HKLM\System\CurrentControlSet\Control\Terminal Server |
| AppLocker | HKLM\Software\Policies\Microsoft\Windows\SrpV2 |
| Screensaver | HKCU\Software\Policies\Microsoft\Windows\Control Panel\Desktop |

---

*Last Updated: 2025 | Applies to: Windows Server 2019/2022, Windows 10/11, GPMC 10.0+*
