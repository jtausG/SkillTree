# PowerShell Automation Guide for IT Operations

## Overview
Comprehensive PowerShell scripting reference for IT administrators. Covers core concepts, automation patterns, common IT tasks, error handling, and production-grade scripting standards.

---

## 1. PowerShell Fundamentals

### Version & Setup
```powershell
# Check version
$PSVersionTable.PSVersion

# Install PowerShell 7 (cross-platform, recommended)
# Windows: winget install Microsoft.PowerShell
# Linux: sudo apt install powershell
# macOS: brew install powershell

# Set execution policy (required for scripts)
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

# Install essential modules
Install-Module -Name PSReadLine -Force          # Better console experience
Install-Module -Name Pester -Force              # Testing framework
Install-Module -Name ImportExcel -Force         # Excel export without Excel
```

### Core Syntax
```powershell
# Variables
$name = "John"
$count = 42
$isEnabled = $true
$nothing = $null

# Arrays
$servers = @("srv-01", "srv-02", "srv-03")
$servers += "srv-04"                            # Add element
$servers[0]                                     # First element
$servers.Count                                  # Length
$servers | Where-Object {$_ -like "srv-0*"}     # Filter

# Hashtables
$user = @{
    Name = "Jane Smith"
    Email = "jsmith@domain.com"
    Department = "IT"
}
$user.Name
$user["Email"]
$user.Keys
$user.Values

# String operations
"Hello, $name!"                                 # Interpolation
"Path: {0}\{1}" -f $env:TEMP, "file.txt"       # Format
"server01".ToUpper()
"  trim me  ".Trim()
"a,b,c".Split(",")
"Hello World" -replace "World", "PowerShell"
"admin@domain.com" -match "@domain\.com"        # Regex
```

### Flow Control
```powershell
# If/ElseIf/Else
if ($count -gt 10) {
    Write-Host "Large"
} elseif ($count -gt 5) {
    Write-Host "Medium"
} else {
    Write-Host "Small"
}

# Switch
switch ($status) {
    "Running"  { Write-Host "Service is running" }
    "Stopped"  { Write-Host "Service is stopped"; Start-Service $svcName }
    "Starting" { Write-Host "Service is starting, please wait" }
    default    { Write-Host "Unknown status: $status" }
}

# ForEach-Object (pipeline)
$servers | ForEach-Object {
    Write-Host "Processing: $_"
    Test-Connection -ComputerName $_ -Count 1 -Quiet
}

# foreach loop
foreach ($server in $servers) {
    $ping = Test-Connection -ComputerName $server -Count 1 -Quiet
    Write-Host "$server : $(if ($ping) {'Online'} else {'Offline'})"
}

# While loop
$attempt = 0
while ($attempt -lt 3) {
    $attempt++
    Write-Host "Attempt $attempt"
    Start-Sleep -Seconds 2
}

# do..while
do {
    $input = Read-Host "Enter 'yes' to continue"
} while ($input -ne "yes")
```

---

## 2. Functions & Modules

### Writing Functions
```powershell
function Get-ServerStatus {
    <#
    .SYNOPSIS
        Tests connectivity and service status on remote servers.
    .DESCRIPTION
        Pings servers and checks if specified services are running.
    .PARAMETER ComputerName
        One or more server names or IP addresses.
    .PARAMETER Service
        Service name to check. Default is 'wuauserv'.
    .EXAMPLE
        Get-ServerStatus -ComputerName "srv-01","srv-02"
    .EXAMPLE
        Get-ServerStatus -ComputerName (Get-Content servers.txt) -Service "Spooler"
    #>
    [CmdletBinding()]
    param (
        [Parameter(Mandatory = $true, ValueFromPipeline = $true)]
        [string[]]$ComputerName,

        [Parameter(Mandatory = $false)]
        [string]$Service = "wuauserv"
    )

    begin {
        $results = [System.Collections.Generic.List[PSCustomObject]]::new()
    }

    process {
        foreach ($computer in $ComputerName) {
            $online = Test-Connection -ComputerName $computer -Count 1 -Quiet -ErrorAction SilentlyContinue
            $serviceStatus = "N/A"

            if ($online) {
                try {
                    $svc = Get-Service -ComputerName $computer -Name $Service -ErrorAction Stop
                    $serviceStatus = $svc.Status
                } catch {
                    $serviceStatus = "Error: $($_.Exception.Message)"
                }
            }

            $results.Add([PSCustomObject]@{
                ComputerName  = $computer
                Online        = $online
                ServiceName   = $Service
                ServiceStatus = $serviceStatus
                Timestamp     = Get-Date
            })
        }
    }

    end {
        return $results
    }
}

# Usage
$servers = @("srv-01","srv-02","srv-03")
$status = Get-ServerStatus -ComputerName $servers -Service "Spooler"
$status | Format-Table -AutoSize
$status | Export-Csv "server_status.csv" -NoTypeInformation
```

### Module Structure
```
MyITModule/
├── MyITModule.psd1      # Module manifest
├── MyITModule.psm1      # Root module (dot-sources functions)
├── Public/
│   ├── Get-ServerStatus.ps1
│   ├── Set-UserPassword.ps1
│   └── Get-DiskReport.ps1
└── Private/
    ├── Write-Log.ps1
    └── Test-IsAdmin.ps1
```

**Module Manifest (.psd1):**
```powershell
@{
    ModuleVersion = '1.0.0'
    GUID = 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'
    Author = 'IT Operations'
    Description = 'IT Operations automation module'
    PowerShellVersion = '5.1'
    FunctionsToExport = @('Get-ServerStatus','Set-UserPassword','Get-DiskReport')
    RequiredModules = @()
}
```

---

## 3. Error Handling

### Try/Catch/Finally
```powershell
function Invoke-SafeCommand {
    param([string]$ComputerName)

    try {
        $result = Invoke-Command -ComputerName $ComputerName -ScriptBlock {
            Get-Process
        } -ErrorAction Stop

        return $result
    }
    catch [System.Management.Automation.Remoting.PSRemotingTransportException] {
        Write-Warning "Cannot connect to $ComputerName - network or WinRM issue"
        Write-Log -Message "Connection failed: $ComputerName" -Level Warning
        return $null
    }
    catch [System.UnauthorizedAccessException] {
        Write-Warning "Access denied to $ComputerName"
        return $null
    }
    catch {
        Write-Error "Unexpected error on $ComputerName : $($_.Exception.Message)"
        Write-Log -Message "Error: $($_.Exception.GetType().FullName): $($_.Exception.Message)" -Level Error
        return $null
    }
    finally {
        # Always runs, even if exception
        Write-Verbose "Completed processing $ComputerName"
    }
}
```

### Logging Function
```powershell
function Write-Log {
    [CmdletBinding()]
    param (
        [Parameter(Mandatory=$true)]
        [string]$Message,

        [ValidateSet("Info","Warning","Error","Debug")]
        [string]$Level = "Info",

        [string]$LogPath = "C:\Logs\IT_Operations.log"
    )

    $timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
    $caller = (Get-PSCallStack)[1].Command
    $logEntry = "[$timestamp] [$Level] [$caller] $Message"

    # Ensure log directory exists
    $logDir = Split-Path $LogPath -Parent
    if (-not (Test-Path $logDir)) {
        New-Item -ItemType Directory -Path $logDir -Force | Out-Null
    }

    Add-Content -Path $LogPath -Value $logEntry

    switch ($Level) {
        "Error"   { Write-Error $Message }
        "Warning" { Write-Warning $Message }
        "Debug"   { Write-Debug $Message }
        default   { Write-Verbose $Message }
    }
}
```

---

## 4. Active Directory Automation

### User Management Scripts
```powershell
# Bulk create AD users from CSV
# CSV columns: FirstName,LastName,Department,Title,Manager,OU

$users = Import-Csv "new_hires.csv"
$defaultPassword = ConvertTo-SecureString "TempP@ssw0rd!" -AsPlainText -Force

foreach ($user in $users) {
    $samAccount = ($user.FirstName[0] + $user.LastName).ToLower()
    $upn = "$samAccount@domain.com"
    $displayName = "$($user.FirstName) $($user.LastName)"

    # Check if exists
    if (Get-ADUser -Filter {SamAccountName -eq $samAccount} -ErrorAction SilentlyContinue) {
        Write-Warning "User $samAccount already exists. Skipping."
        continue
    }

    try {
        New-ADUser -SamAccountName $samAccount `
                   -UserPrincipalName $upn `
                   -GivenName $user.FirstName `
                   -Surname $user.LastName `
                   -DisplayName $displayName `
                   -Department $user.Department `
                   -Title $user.Title `
                   -Manager $user.Manager `
                   -Path $user.OU `
                   -AccountPassword $defaultPassword `
                   -ChangePasswordAtLogon $true `
                   -Enabled $true

        Write-Log -Message "Created user: $upn" -Level Info
    }
    catch {
        Write-Log -Message "Failed to create $upn : $($_.Exception.Message)" -Level Error
    }
}
```

```powershell
# Find inactive users (no login in 90 days)
$cutoff = (Get-Date).AddDays(-90)
$inactiveUsers = Get-ADUser -Filter {
    Enabled -eq $true -and
    LastLogonDate -lt $cutoff -and
    LastLogonDate -ne $null
} -Properties LastLogonDate, Department, Manager |
Select SamAccountName, DisplayName, LastLogonDate, Department, Manager |
Sort LastLogonDate

$inactiveUsers | Export-Csv "inactive_users_$(Get-Date -Format yyyyMMdd).csv" -NoTypeInformation
Write-Host "Found $($inactiveUsers.Count) inactive users"
```

```powershell
# Disable and move inactive users
function Disable-InactiveUsers {
    param(
        [int]$InactiveDays = 90,
        [string]$DisabledOU = "OU=Disabled,DC=domain,DC=com",
        [switch]$WhatIf
    )

    $cutoff = (Get-Date).AddDays(-$InactiveDays)
    $users = Get-ADUser -Filter {
        Enabled -eq $true -and LastLogonDate -lt $cutoff
    } -Properties LastLogonDate

    foreach ($user in $users) {
        $action = if ($WhatIf) {"[WHATIF]"} else {""}

        if (-not $WhatIf) {
            Disable-ADAccount -Identity $user.SamAccountName
            Move-ADObject -Identity $user.DistinguishedName -TargetPath $DisabledOU
            Set-ADUser -Identity $user.SamAccountName `
                -Description "Disabled $(Get-Date -Format 'yyyy-MM-dd') - Inactive $InactiveDays days"
        }

        Write-Log -Message "$action Disabled inactive user: $($user.SamAccountName) (Last login: $($user.LastLogonDate))"
    }

    Write-Host "$($users.Count) users processed"
}

# Test run first
Disable-InactiveUsers -InactiveDays 90 -WhatIf
# Then execute
# Disable-InactiveUsers -InactiveDays 90
```

### Password & Security Audits
```powershell
# Find accounts with password never expires
Get-ADUser -Filter {PasswordNeverExpires -eq $true -and Enabled -eq $true} `
    -Properties PasswordNeverExpires, PasswordLastSet, Department |
Select SamAccountName, DisplayName, PasswordLastSet, Department |
Export-Csv "password_never_expires.csv" -NoTypeInformation

# Find accounts that haven't changed password in 180 days
$cutoff = (Get-Date).AddDays(-180)
Get-ADUser -Filter {Enabled -eq $true} -Properties PasswordLastSet |
Where {$_.PasswordLastSet -lt $cutoff -or $_.PasswordLastSet -eq $null} |
Select SamAccountName, DisplayName, PasswordLastSet |
Sort PasswordLastSet |
Export-Csv "stale_passwords.csv" -NoTypeInformation

# Domain admin audit
Get-ADGroupMember -Identity "Domain Admins" -Recursive |
Where {$_.objectClass -eq "user"} |
Get-ADUser -Properties LastLogonDate, PasswordLastSet, Enabled |
Select SamAccountName, DisplayName, Enabled, LastLogonDate, PasswordLastSet |
Export-Csv "domain_admins_audit.csv" -NoTypeInformation
```

---

## 5. System & Server Administration

### Disk Space Monitoring
```powershell
function Get-DiskSpaceReport {
    param(
        [string[]]$ComputerName = @($env:COMPUTERNAME),
        [int]$WarningThreshold = 20,
        [int]$CriticalThreshold = 10
    )

    $results = foreach ($computer in $ComputerName) {
        try {
            $disks = Get-WmiObject -Class Win32_LogicalDisk `
                -ComputerName $computer `
                -Filter "DriveType=3" `
                -ErrorAction Stop

            foreach ($disk in $disks) {
                $freePercent = [math]::Round(($disk.FreeSpace / $disk.Size) * 100, 1)
                $status = if ($freePercent -lt $CriticalThreshold) {"CRITICAL"}
                          elseif ($freePercent -lt $WarningThreshold) {"WARNING"}
                          else {"OK"}

                [PSCustomObject]@{
                    Computer    = $computer
                    Drive       = $disk.DeviceID
                    TotalGB     = [math]::Round($disk.Size / 1GB, 1)
                    FreeGB      = [math]::Round($disk.FreeSpace / 1GB, 1)
                    FreePercent = $freePercent
                    Status      = $status
                }
            }
        }
        catch {
            [PSCustomObject]@{
                Computer    = $computer
                Drive       = "ERROR"
                TotalGB     = 0
                FreeGB      = 0
                FreePercent = 0
                Status      = "UNREACHABLE: $($_.Exception.Message)"
            }
        }
    }

    $results | Sort Status, FreePercent
}

# Get all servers from AD
$servers = Get-ADComputer -Filter {OperatingSystem -like "*Server*"} |
    Select -ExpandProperty Name

$report = Get-DiskSpaceReport -ComputerName $servers
$report | Where {$_.Status -ne "OK"} | Format-Table -AutoSize
$report | Export-Csv "disk_report_$(Get-Date -Format yyyyMMdd).csv" -NoTypeInformation
```

### Service Monitoring & Auto-Restart
```powershell
function Watch-CriticalServices {
    param(
        [Parameter(Mandatory=$true)]
        [hashtable]$ServiceMap,  # @{"Server01"=@("wuauserv","Spooler")}
        [switch]$AutoRestart
    )

    foreach ($computer in $ServiceMap.Keys) {
        foreach ($svcName in $ServiceMap[$computer]) {
            try {
                $svc = Get-Service -ComputerName $computer -Name $svcName -ErrorAction Stop

                if ($svc.Status -ne "Running") {
                    Write-Warning "$computer : $svcName is $($svc.Status)"
                    Write-Log -Message "$computer : $svcName stopped" -Level Warning

                    if ($AutoRestart) {
                        Start-Service -InputObject $svc
                        Start-Sleep -Seconds 5
                        $svc.Refresh()

                        if ($svc.Status -eq "Running") {
                            Write-Log -Message "Restarted $svcName on $computer" -Level Info
                        } else {
                            Write-Log -Message "Failed to restart $svcName on $computer" -Level Error
                            # Send-AlertEmail here
                        }
                    }
                }
            }
            catch {
                Write-Log -Message "Cannot check $svcName on $computer : $($_.Exception.Message)" -Level Error
            }
        }
    }
}

$criticalServices = @{
    "srv-app-01" = @("w3svc","WAS","MSSQLSERVER")
    "srv-app-02" = @("w3svc","WAS")
    "srv-db-01"  = @("MSSQLSERVER","SQLAgent","SQLSERVERAGENT")
}

Watch-CriticalServices -ServiceMap $criticalServices -AutoRestart
```

### Windows Update Management
```powershell
# Check pending updates
function Get-PendingUpdates {
    param([string[]]$ComputerName = @($env:COMPUTERNAME))

    foreach ($computer in $ComputerName) {
        try {
            $updates = Invoke-Command -ComputerName $computer -ScriptBlock {
                $session = New-Object -ComObject Microsoft.Update.Session
                $searcher = $session.CreateUpdateSearcher()
                $result = $searcher.Search("IsInstalled=0 and Type='Software'")
                $result.Updates | Select Title, @{N="SizeMB";E={[math]::Round($_.MaxDownloadSize/1MB,1)}}
            }

            [PSCustomObject]@{
                Computer      = $computer
                PendingCount  = $updates.Count
                Updates       = $updates
            }
        }
        catch {
            [PSCustomObject]@{
                Computer     = $computer
                PendingCount = -1
                Updates      = "Error: $($_.Exception.Message)"
            }
        }
    }
}
```

---

## 6. Reporting & Email

### HTML Report Generation
```powershell
function New-HTMLReport {
    param(
        [string]$Title,
        [System.Collections.Generic.List[PSCustomObject]]$Data,
        [string]$OutputPath
    )

    $css = @"
    <style>
        body { font-family: Segoe UI, sans-serif; font-size: 12px; }
        h1 { color: #0078D4; }
        table { border-collapse: collapse; width: 100%; }
        th { background-color: #0078D4; color: white; padding: 8px; text-align: left; }
        td { padding: 6px 8px; border-bottom: 1px solid #ddd; }
        tr:nth-child(even) { background-color: #f2f2f2; }
        .critical { background-color: #FFB6C1; }
        .warning { background-color: #FFD700; }
        .ok { background-color: #90EE90; }
    </style>
"@

    $html = @"
    <!DOCTYPE html>
    <html>
    <head><title>$Title</title>$css</head>
    <body>
    <h1>$Title</h1>
    <p>Generated: $(Get-Date -Format 'yyyy-MM-dd HH:mm:ss')</p>
    $($Data | ConvertTo-Html -Fragment)
    </body>
    </html>
"@

    $html | Out-File $OutputPath -Encoding UTF8
    Write-Host "Report saved: $OutputPath"
}
```

### Send Email Alerts
```powershell
function Send-AlertEmail {
    param(
        [string]$To,
        [string]$Subject,
        [string]$Body,
        [string]$SmtpServer = "smtp.domain.com",
        [string]$From = "monitoring@domain.com",
        [string[]]$Attachments = @()
    )

    $params = @{
        To         = $To
        From       = $From
        Subject    = $Subject
        Body       = $Body
        BodyAsHtml = $true
        SmtpServer = $SmtpServer
        Port       = 587
        UseSsl     = $true
    }

    if ($Attachments.Count -gt 0) {
        $params.Attachments = $Attachments
    }

    try {
        Send-MailMessage @params
        Write-Log -Message "Email sent to $To : $Subject"
    }
    catch {
        Write-Log -Message "Email send failed: $($_.Exception.Message)" -Level Error
    }
}
```

---

## 7. Scheduled Task Automation

### Create Scheduled Task
```powershell
# Create scheduled task for a PS script
$action = New-ScheduledTaskAction `
    -Execute "PowerShell.exe" `
    -Argument "-NonInteractive -WindowStyle Hidden -ExecutionPolicy Bypass -File C:\Scripts\DiskReport.ps1"

$trigger = New-ScheduledTaskTrigger `
    -Daily `
    -At "6:00 AM"

$principal = New-ScheduledTaskPrincipal `
    -UserId "DOMAIN\svc-monitoring" `
    -LogonType Password `
    -RunLevel Highest

$settings = New-ScheduledTaskSettingsSet `
    -ExecutionTimeLimit (New-TimeSpan -Hours 1) `
    -RestartCount 3 `
    -RestartInterval (New-TimeSpan -Minutes 5) `
    -MultipleInstances IgnoreNew

Register-ScheduledTask `
    -TaskName "IT - Daily Disk Report" `
    -TaskPath "\IT Operations\" `
    -Action $action `
    -Trigger $trigger `
    -Principal $principal `
    -Settings $settings `
    -Description "Generates and emails daily disk space report"
```

---

## 8. Security Scripts

### Local Admin Audit
```powershell
function Get-LocalAdminReport {
    param([string[]]$ComputerName)

    foreach ($computer in $ComputerName) {
        try {
            $admins = Invoke-Command -ComputerName $computer -ScriptBlock {
                $group = [ADSI]"WinNT://$env:COMPUTERNAME/Administrators,group"
                $group.Members() | ForEach-Object {
                    $path = $_.GetType().InvokeMember("AdsPath","GetProperty",$null,$_,$null)
                    $name = $_.GetType().InvokeMember("Name","GetProperty",$null,$_,$null)
                    [PSCustomObject]@{
                        Name = $name
                        Path = $path
                    }
                }
            }
            foreach ($admin in $admins) {
                [PSCustomObject]@{
                    Computer  = $computer
                    AdminName = $admin.Name
                    AdminPath = $admin.Path
                    Timestamp = Get-Date
                }
            }
        }
        catch {
            [PSCustomObject]@{
                Computer  = $computer
                AdminName = "ERROR"
                AdminPath = $_.Exception.Message
                Timestamp = Get-Date
            }
        }
    }
}
```

### Certificate Expiry Check
```powershell
function Get-ExpiringCertificates {
    param(
        [string[]]$ComputerName = @($env:COMPUTERNAME),
        [int]$DaysWarning = 30
    )

    $expiringCerts = foreach ($computer in $ComputerName) {
        $certs = Invoke-Command -ComputerName $computer -ScriptBlock {
            param($days)
            $stores = @("LocalMachine\My","LocalMachine\WebHosting")
            foreach ($storePath in $stores) {
                $parts = $storePath.Split("\")
                $store = New-Object System.Security.Cryptography.X509Certificates.X509Store($parts[1], $parts[0])
                $store.Open("ReadOnly")
                $store.Certificates | Where {
                    $_.NotAfter -lt (Get-Date).AddDays($days)
                } | Select Subject, Thumbprint, NotAfter,
                    @{N="DaysRemaining";E={[int]($_.NotAfter - (Get-Date)).TotalDays}},
                    @{N="Store";E={$storePath}}
                $store.Close()
            }
        } -ArgumentList $DaysWarning

        foreach ($cert in $certs) {
            $cert | Add-Member -NotePropertyName Computer -NotePropertyValue $computer -PassThru
        }
    }

    $expiringCerts | Sort DaysRemaining | Format-Table Computer, Subject, NotAfter, DaysRemaining -AutoSize
}
```

---

## 9. Script Templates

### Production Script Template
```powershell
#Requires -Version 5.1
#Requires -Modules ActiveDirectory

<#
.SYNOPSIS
    Brief description of what the script does.
.DESCRIPTION
    Detailed description.
.PARAMETER InputFile
    Path to input CSV file.
.PARAMETER LogPath
    Where to write logs. Default: C:\Logs\<ScriptName>.log
.PARAMETER WhatIf
    Show what would happen without making changes.
.EXAMPLE
    .\MyScript.ps1 -InputFile "C:\input.csv"
.NOTES
    Author: IT Operations
    Version: 1.0.0
    Created: 2025-01-01
    Modified: 2025-01-01
    Requires: AD module, run as administrator
#>

[CmdletBinding(SupportsShouldProcess=$true)]
param (
    [Parameter(Mandatory=$true)]
    [ValidateScript({Test-Path $_})]
    [string]$InputFile,

    [Parameter(Mandatory=$false)]
    [string]$LogPath = "C:\Logs\$([System.IO.Path]::GetFileNameWithoutExtension($MyInvocation.MyCommand.Name)).log",

    [Parameter(Mandatory=$false)]
    [switch]$WhatIf
)

Set-StrictMode -Version Latest
$ErrorActionPreference = "Stop"

# ---- Functions ----
function Write-Log {
    param([string]$Message, [string]$Level = "Info")
    $entry = "[$(Get-Date -Format 'yyyy-MM-dd HH:mm:ss')] [$Level] $Message"
    Add-Content -Path $LogPath -Value $entry
    switch ($Level) {
        "Error"   { Write-Error $Message }
        "Warning" { Write-Warning $Message }
        default   { Write-Verbose $Message }
    }
}

# ---- Prerequisites ----
if (-not (Test-Path (Split-Path $LogPath -Parent))) {
    New-Item -ItemType Directory -Path (Split-Path $LogPath -Parent) -Force | Out-Null
}

# ---- Main ----
Write-Log -Message "=== Script started by $env:USERNAME on $env:COMPUTERNAME ==="

try {
    $data = Import-Csv $InputFile
    Write-Log -Message "Loaded $($data.Count) records from $InputFile"

    foreach ($item in $data) {
        if ($PSCmdlet.ShouldProcess($item.Name, "Process item")) {
            # Do work here
            Write-Log -Message "Processed: $($item.Name)"
        }
    }

    Write-Log -Message "=== Script completed successfully ==="
}
catch {
    Write-Log -Message "Fatal error: $($_.Exception.Message)" -Level Error
    Write-Log -Message "Stack trace: $($_.ScriptStackTrace)" -Level Error
    exit 1
}
```

---

## 10. Useful One-Liners

```powershell
# Find all computers in AD, ping test, export results
Get-ADComputer -Filter * | Select -Expand Name | ForEach {
    [PSCustomObject]@{Name=$_; Online=Test-Connection $_ -Count 1 -Quiet 2>$null}
} | Export-Csv "ping_results.csv" -NoTypeInformation

# Get top 10 largest files on C:
Get-ChildItem C:\ -Recurse -ErrorAction SilentlyContinue |
    Sort Length -Descending | Select -First 10 Name, DirectoryName,
    @{N="SizeMB";E={[math]::Round($_.Length/1MB,2)}}

# List all installed software
Get-ItemProperty HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*,
    HKLM:\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\* |
    Select DisplayName, DisplayVersion, Publisher, InstallDate |
    Where DisplayName | Sort DisplayName

# Check which process is using a port
Get-NetTCPConnection -LocalPort 443 -State Listen |
    Select LocalAddress, LocalPort, OwningProcess |
    ForEach {$_ | Add-Member -NotePropertyName ProcessName -NotePropertyValue (Get-Process -Id $_.OwningProcess).Name -PassThru}

# Find domain computers with BitLocker not enabled
Get-ADComputer -Filter {OperatingSystem -like "*Windows*"} |
    ForEach {
        $bde = Get-BitLockerVolume -MountPoint C: -ErrorAction SilentlyContinue
        if (-not $bde -or $bde.ProtectionStatus -ne "On") {
            $_.Name
        }
    }

# Export all AD group memberships
Get-ADGroup -Filter * | ForEach {
    $grp = $_
    Get-ADGroupMember -Identity $grp | ForEach {
        [PSCustomObject]@{Group=$grp.Name; Member=$_.SamAccountName; Type=$_.objectClass}
    }
} | Export-Csv "all_group_memberships.csv" -NoTypeInformation

# Restart service on multiple servers if stopped
"srv-01","srv-02","srv-03" | ForEach {
    $svc = Get-Service -ComputerName $_ -Name "Spooler"
    if ($svc.Status -ne "Running") { $svc | Start-Service; Write-Host "Started Spooler on $_" }
}
```

---

*Last Updated: 2025 | IT Operations Documentation Library*
