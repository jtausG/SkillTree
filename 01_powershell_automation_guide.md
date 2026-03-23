# PowerShell Automation Guide for IT Operations

## Table of Contents
1. [PowerShell Fundamentals](#fundamentals)
2. [Execution Policy & Security](#security)
3. [Variables, Types & Operators](#variables)
4. [Control Flow](#control-flow)
5. [Functions & Modules](#functions)
6. [Error Handling](#error-handling)
7. [File System Operations](#filesystem)
8. [Active Directory Automation](#ad-automation)
9. [Network Automation](#network-automation)
10. [User & Group Management](#user-management)
11. [System Administration](#system-admin)
12. [Reporting & Logging](#reporting)
13. [Scheduled Tasks & Automation](#scheduling)
14. [REST API Integration](#api)
15. [Best Practices & Style Guide](#best-practices)

---

## 1. PowerShell Fundamentals <a name="fundamentals"></a>

### Version Check & Requirements
```powershell
# Check PowerShell version
$PSVersionTable.PSVersion

# Check if running as Administrator
([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)

# Self-elevate script
if (-not ([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)) {
    Start-Process PowerShell -Verb RunAs -ArgumentList "-File `"$PSCommandPath`""
    Exit
}
```

### Essential Concepts
| Concept | Description | Example |
|---------|-------------|---------|
| Cmdlets | Verb-Noun commands | `Get-Process`, `Set-Item` |
| Pipeline | Chain commands with \| | `Get-Service \| Where-Object {$_.Status -eq 'Running'}` |
| Objects | Everything is an object | `(Get-Date).DayOfWeek` |
| Providers | Virtual drives | `HKLM:`, `Cert:`, `AD:` |
| Remoting | Execute on remote hosts | `Invoke-Command -ComputerName` |

### Help System
```powershell
# Get help for any cmdlet
Get-Help Get-ADUser -Full
Get-Help Get-ADUser -Examples
Get-Help Get-ADUser -Online

# Update help files
Update-Help -Force

# Find commands
Get-Command -Verb Get -Noun *AD*
Get-Command -Module ActiveDirectory
```

---

## 2. Execution Policy & Security <a name="security"></a>

### Execution Policies
```powershell
# Check current policy
Get-ExecutionPolicy -List

# Set policy (requires admin)
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
Set-ExecutionPolicy -ExecutionPolicy AllSigned -Scope LocalMachine

# Bypass for single script (use cautiously)
PowerShell.exe -ExecutionPolicy Bypass -File "C:\Scripts\myscript.ps1"
```

### Script Signing
```powershell
# Get code signing certificate
$cert = Get-ChildItem Cert:\CurrentUser\My -CodeSigningCert

# Sign a script
Set-AuthenticodeSignature -FilePath "C:\Scripts\myscript.ps1" -Certificate $cert

# Verify signature
Get-AuthenticodeSignature -FilePath "C:\Scripts\myscript.ps1"
```

### Constrained Language Mode
```powershell
# Check language mode
$ExecutionContext.SessionState.LanguageMode

# Full, Constrained, RestrictedLanguage, NoLanguage
```

---

## 3. Variables, Types & Operators <a name="variables"></a>

### Variable Declaration
```powershell
# Basic variables
$name = "John Doe"
$age = 30
$isAdmin = $true
$nothing = $null

# Type-constrained variables
[string]$username = "jdoe"
[int]$port = 443
[datetime]$created = Get-Date
[array]$servers = @("srv01", "srv02", "srv03")
[hashtable]$config = @{
    Server   = "dc01.contoso.com"
    Port     = 389
    SSL      = $true
}

# Automatic variables
$_ # Current pipeline object
$PSScriptRoot # Directory of running script
$MyInvocation # Script invocation info
$args # Arguments passed to script
$env:COMPUTERNAME # Environment variables
```

### String Operations
```powershell
# String formatting
$server = "SRV01"
$message = "Connecting to $server on port 443"
$formatted = "Server: {0,-20} Status: {1}" -f $server, "Online"

# Here-string (multi-line)
$body = @"
Dear $name,

Your account has been created.
Username: $username
"@

# String methods
$str = "  Hello World  "
$str.Trim()
$str.ToUpper()
$str.Replace("World", "PowerShell")
$str.Split(" ")
"hello" -match "^h"    # Regex match
"hello" -like "h*"     # Wildcard match
```

### Arrays & Collections
```powershell
# Array operations
$arr = @(1, 2, 3, 4, 5)
$arr += 6
$arr[0]         # First element
$arr[-1]        # Last element
$arr[1..3]      # Slice
$arr.Count

# ArrayList (better performance for additions)
$list = [System.Collections.ArrayList]@()
$list.Add("item1")
$list.Remove("item1")

# Generic List
$typed = [System.Collections.Generic.List[string]]@()
$typed.Add("server01")

# Hashtable operations
$ht = @{ Name = "John"; Age = 30 }
$ht["Email"] = "john@company.com"
$ht.Keys
$ht.Values
$ht.ContainsKey("Name")

# Ordered hashtable
$ordered = [ordered]@{ First = 1; Second = 2; Third = 3 }
```

---

## 4. Control Flow <a name="control-flow"></a>

### Conditionals
```powershell
# If/ElseIf/Else
if ($age -ge 18 -and $isAdmin) {
    Write-Host "Adult admin"
} elseif ($age -ge 18) {
    Write-Host "Adult non-admin"
} else {
    Write-Host "Minor"
}

# Switch statement
switch ($errorCode) {
    0    { "Success" }
    1    { "Access Denied" }
    5    { "Error" }
    { $_ -gt 100 } { "High error code: $_" }
    default { "Unknown: $_" }
}

# Ternary-style (PS 7+)
$result = $condition ? "Yes" : "No"

# Null coalescing (PS 7+)
$value = $possiblyNull ?? "default"
```

### Loops
```powershell
# ForEach-Object (pipeline)
$servers | ForEach-Object {
    Test-Connection -ComputerName $_ -Count 1 -Quiet
}

# foreach loop
foreach ($server in $servers) {
    Write-Host "Processing: $server"
}

# For loop
for ($i = 0; $i -lt 10; $i++) {
    Write-Host "Iteration: $i"
}

# While / Do-While
$count = 0
while ($count -lt 5) {
    $count++
}

do {
    $input = Read-Host "Enter value (or 'quit')"
} while ($input -ne "quit")

# Break and Continue
foreach ($item in $collection) {
    if ($item -eq "skip") { continue }
    if ($item -eq "stop") { break }
    Process-Item $item
}
```

### Pipeline & Filtering
```powershell
# Where-Object
Get-Process | Where-Object { $_.CPU -gt 100 -and $_.Name -ne "Idle" }
Get-Service | Where-Object Status -eq 'Running'

# Select-Object
Get-ADUser -Filter * | Select-Object Name, SamAccountName, LastLogonDate -First 10

# Sort-Object
Get-Process | Sort-Object CPU -Descending | Select-Object -First 20

# Group-Object
Get-EventLog -LogName System -Newest 1000 | Group-Object EventID | Sort-Object Count -Descending

# Measure-Object
Get-ChildItem C:\Logs -Recurse | Measure-Object -Property Length -Sum -Average -Maximum
```

---

## 5. Functions & Modules <a name="functions"></a>

### Advanced Functions
```powershell
function Invoke-ServerHealthCheck {
    [CmdletBinding(SupportsShouldProcess)]
    param(
        [Parameter(Mandatory, ValueFromPipeline, ValueFromPipelineByPropertyName)]
        [string[]]$ComputerName,

        [Parameter()]
        [ValidateRange(1, 65535)]
        [int]$TimeoutSeconds = 30,

        [Parameter()]
        [ValidateSet('Basic', 'Full', 'Network')]
        [string]$CheckType = 'Basic',

        [Parameter()]
        [switch]$PassThru
    )

    begin {
        $results = [System.Collections.ArrayList]@()
        Write-Verbose "Starting health checks with type: $CheckType"
    }

    process {
        foreach ($computer in $ComputerName) {
            Write-Verbose "Checking: $computer"

            $result = [PSCustomObject]@{
                ComputerName = $computer
                Timestamp    = Get-Date
                Online       = $false
                PingMs       = $null
                DiskFree     = $null
                CPUPercent   = $null
                MemFreeGB    = $null
                Errors       = @()
            }

            try {
                # Ping test
                $ping = Test-Connection -ComputerName $computer -Count 2 -ErrorAction Stop
                $result.Online = $true
                $result.PingMs = ($ping | Measure-Object -Property ResponseTime -Average).Average

                if ($CheckType -in 'Full', 'Basic') {
                    # Disk check
                    $disk = Get-WmiObject -Class Win32_LogicalDisk -ComputerName $computer -Filter "DeviceID='C:'" -ErrorAction Stop
                    $result.DiskFree = [math]::Round($disk.FreeSpace / 1GB, 2)
                }

                if ($CheckType -eq 'Full') {
                    # CPU/Memory
                    $cpu = Get-WmiObject -Class Win32_Processor -ComputerName $computer -ErrorAction Stop
                    $result.CPUPercent = $cpu.LoadPercentage

                    $os = Get-WmiObject -Class Win32_OperatingSystem -ComputerName $computer -ErrorAction Stop
                    $result.MemFreeGB = [math]::Round($os.FreePhysicalMemory / 1MB, 2)
                }
            }
            catch {
                $result.Errors += $_.Exception.Message
                Write-Warning "Error checking $computer`: $_"
            }

            [void]$results.Add($result)

            if ($PassThru) { Write-Output $result }
        }
    }

    end {
        if (-not $PassThru) { Write-Output $results }
        Write-Verbose "Completed health checks for $($results.Count) computers"
    }
}
```

### Module Creation
```powershell
# Module structure
# MyITModule/
# ├── MyITModule.psd1  (manifest)
# ├── MyITModule.psm1  (root module)
# ├── Public/          (exported functions)
# │   ├── Get-ServerHealth.ps1
# │   └── Set-UserPassword.ps1
# └── Private/         (internal functions)
#     └── Write-Log.ps1

# Root module (MyITModule.psm1)
$Public  = Get-ChildItem "$PSScriptRoot\Public\*.ps1"
$Private = Get-ChildItem "$PSScriptRoot\Private\*.ps1"

foreach ($file in @($Private + $Public)) {
    . $file.FullName
}

Export-ModuleMember -Function $Public.BaseName

# Create manifest
New-ModuleManifest -Path ".\MyITModule.psd1" `
    -RootModule "MyITModule.psm1" `
    -ModuleVersion "1.0.0" `
    -Author "IT Operations" `
    -Description "IT Operations Automation Module" `
    -FunctionsToExport @('Get-ServerHealth', 'Set-UserPassword') `
    -RequiredModules @('ActiveDirectory')
```

---

## 6. Error Handling <a name="error-handling"></a>

### Try/Catch/Finally
```powershell
function Connect-DatabaseSafely {
    param([string]$Server, [string]$Database)

    try {
        $connection = New-Object System.Data.SqlClient.SqlConnection
        $connection.ConnectionString = "Server=$Server;Database=$Database;Integrated Security=True"
        $connection.Open()
        Write-Verbose "Connected to $Server\$Database"
        return $connection
    }
    catch [System.Data.SqlClient.SqlException] {
        Write-Error "SQL Error connecting to $Server`: $($_.Exception.Message)"
        throw
    }
    catch [System.Net.Sockets.SocketException] {
        Write-Error "Network error reaching $Server`: $($_.Exception.Message)"
        throw
    }
    catch {
        Write-Error "Unexpected error: $($_.Exception.GetType().Name) - $($_.Exception.Message)"
        throw
    }
    finally {
        # Always runs - cleanup if needed
        Write-Verbose "Connection attempt completed"
    }
}

# ErrorAction preferences
Get-Service -Name "NonExistentService" -ErrorAction SilentlyContinue
Get-Service -Name "NonExistentService" -ErrorAction Stop
$ErrorActionPreference = 'Stop'  # Script-wide default

# $Error automatic variable
$Error[0]  # Most recent error
$Error.Clear()

# Trap (older method, avoid in new code)
trap [System.IO.IOException] {
    Write-Warning "IO Error: $_"
    continue
}
```

### Validation & Defensive Coding
```powershell
# Parameter validation attributes
param(
    [ValidateNotNullOrEmpty()]
    [string]$Username,

    [ValidatePattern('^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$')]
    [string]$Email,

    [ValidateScript({
        if (Test-Path $_) { $true }
        else { throw "Path '$_' does not exist" }
    })]
    [string]$LogPath,

    [ValidateRange(1, 65535)]
    [int]$Port
)
```

---

## 7. File System Operations <a name="filesystem"></a>

### File & Directory Management
```powershell
# Create/Read/Write files
New-Item -Path "C:\Logs" -ItemType Directory -Force
Set-Content -Path "C:\Logs\output.txt" -Value "Initial content"
Add-Content -Path "C:\Logs\output.txt" -Value "Appended line"
$content = Get-Content -Path "C:\Logs\output.txt"
$content = Get-Content -Path "C:\Logs\output.txt" -Raw  # As single string

# Copy/Move/Delete
Copy-Item -Path "C:\Source\*" -Destination "C:\Dest" -Recurse -Force
Move-Item -Path "C:\Old" -Destination "C:\New"
Remove-Item -Path "C:\Temp\*" -Recurse -Force -WhatIf

# File info
$file = Get-Item "C:\Scripts\deploy.ps1"
$file.Length       # Size in bytes
$file.LastWriteTime
$file.Extension

# Search files
Get-ChildItem -Path "C:\Logs" -Filter "*.log" -Recurse |
    Where-Object { $_.LastWriteTime -lt (Get-Date).AddDays(-30) }

# Find files containing text
Get-ChildItem -Path "C:\" -Recurse -Filter "*.ps1" |
    Select-String -Pattern "password" -CaseSensitive |
    Select-Object Path, LineNumber, Line
```

### CSV & JSON Operations
```powershell
# CSV Import/Export
$users = Import-Csv -Path "C:\users.csv"
$users | Where-Object { $_.Department -eq "IT" } |
    Export-Csv -Path "C:\it_users.csv" -NoTypeInformation

# JSON Import/Export
$config = Get-Content "config.json" | ConvertFrom-Json
$config.ServerList += "srv04"
$config | ConvertTo-Json -Depth 5 | Set-Content "config.json"

# XML Operations
[xml]$xml = Get-Content "config.xml"
$xml.Configuration.Database.Server
$xml.Configuration.Database.Server = "newsrv"
$xml.Save("config.xml")
```

### Log File Management
```powershell
function Write-Log {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory)]
        [string]$Message,

        [ValidateSet('INFO', 'WARN', 'ERROR', 'DEBUG')]
        [string]$Level = 'INFO',

        [string]$LogFile = "$PSScriptRoot\Logs\script_$(Get-Date -Format 'yyyyMMdd').log"
    )

    $timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
    $logEntry = "[$timestamp] [$Level] $Message"

    # Ensure log directory exists
    $logDir = Split-Path $LogFile
    if (-not (Test-Path $logDir)) {
        New-Item -Path $logDir -ItemType Directory -Force | Out-Null
    }

    Add-Content -Path $LogFile -Value $logEntry

    switch ($Level) {
        'ERROR' { Write-Error $Message }
        'WARN'  { Write-Warning $Message }
        'DEBUG' { Write-Verbose $Message }
        default { Write-Verbose $Message }
    }
}

# Rotate logs older than 30 days
function Remove-OldLogs {
    param(
        [string]$LogPath = "C:\Logs",
        [int]$DaysToKeep = 30
    )

    $cutoff = (Get-Date).AddDays(-$DaysToKeep)
    Get-ChildItem -Path $LogPath -Filter "*.log" |
        Where-Object { $_.LastWriteTime -lt $cutoff } |
        Remove-Item -Force
}
```

---

## 8. Active Directory Automation <a name="ad-automation"></a>

### Bulk User Operations
```powershell
# Create users from CSV
# CSV format: FirstName,LastName,Department,Manager,Title
Import-Csv "C:\NewUsers.csv" | ForEach-Object {
    $params = @{
        Name              = "$($_.FirstName) $($_.LastName)"
        GivenName         = $_.FirstName
        Surname           = $_.LastName
        SamAccountName    = "$($_.FirstName.Substring(0,1))$($_.LastName)".ToLower()
        UserPrincipalName = "$($_.FirstName.Substring(0,1))$($_.LastName)@contoso.com".ToLower()
        Department        = $_.Department
        Title             = $_.Title
        Manager           = (Get-ADUser -Filter "Name -eq '$($_.Manager)'").DistinguishedName
        Path              = "OU=$($_.Department),OU=Users,DC=contoso,DC=com"
        AccountPassword   = (ConvertTo-SecureString "Welcome@123!" -AsPlainText -Force)
        ChangePasswordAtLogon = $true
        Enabled           = $true
    }

    try {
        New-ADUser @params
        Write-Log "Created user: $($params.SamAccountName)"
    }
    catch {
        Write-Log "Failed to create $($params.SamAccountName): $_" -Level ERROR
    }
}

# Disable inactive users (90+ days)
$cutoff = (Get-Date).AddDays(-90)
Get-ADUser -Filter {LastLogonDate -lt $cutoff -and Enabled -eq $true} `
    -Properties LastLogonDate |
    Where-Object { $_.DistinguishedName -notmatch "OU=ServiceAccounts" } |
    ForEach-Object {
        Disable-ADAccount -Identity $_
        Move-ADObject -Identity $_.DistinguishedName -TargetPath "OU=Disabled,DC=contoso,DC=com"
        Write-Log "Disabled inactive user: $($_.SamAccountName) (Last logon: $($_.LastLogonDate))"
    }

# Export all AD users with key attributes
Get-ADUser -Filter * -Properties * |
    Select-Object @{N='Username';E={$_.SamAccountName}},
                  @{N='Display Name';E={$_.DisplayName}},
                  @{N='Email';E={$_.EmailAddress}},
                  @{N='Department';E={$_.Department}},
                  @{N='Title';E={$_.Title}},
                  @{N='Enabled';E={$_.Enabled}},
                  @{N='Last Logon';E={$_.LastLogonDate}},
                  @{N='Password Expires';E={$_.PasswordExpiry}},
                  @{N='Created';E={$_.Created}} |
    Export-Csv "C:\Reports\AD_Users_$(Get-Date -Format yyyyMMdd).csv" -NoTypeInformation
```

### Group Management
```powershell
# Add multiple users to group from CSV
Import-Csv "C:\GroupMembers.csv" | ForEach-Object {
    try {
        Add-ADGroupMember -Identity $_.GroupName -Members $_.Username
    }
    catch {
        Write-Warning "Could not add $($_.Username) to $($_.GroupName): $_"
    }
}

# Audit group membership changes (event 4728/4729/4732/4733)
Get-WinEvent -ComputerName "DC01" -FilterHashtable @{
    LogName   = 'Security'
    Id        = 4728, 4729, 4732, 4733, 4756, 4757
    StartTime = (Get-Date).AddDays(-7)
} | Select-Object TimeCreated,
    @{N='Action';E={$_.Message.Split("`n")[0]}},
    @{N='GroupName';E={($_.Message | Select-String 'Group Name:\s+(.+)').Matches.Groups[1].Value}},
    @{N='MemberName';E={($_.Message | Select-String 'Member Name:\s+(.+)').Matches.Groups[1].Value}} |
    Export-Csv "C:\Reports\GroupChanges.csv" -NoTypeInformation

# Find empty groups
Get-ADGroup -Filter * -Properties Members |
    Where-Object { $_.Members.Count -eq 0 } |
    Select-Object Name, GroupCategory, GroupScope, DistinguishedName |
    Export-Csv "C:\Reports\EmptyGroups.csv" -NoTypeInformation

# Find groups with no owner
Get-ADGroup -Filter * -Properties ManagedBy |
    Where-Object { $null -eq $_.ManagedBy } |
    Select-Object Name, DistinguishedName
```

### Password & Account Management
```powershell
# Find expiring passwords (next 14 days)
$maxPasswordAge = (Get-ADDefaultDomainPasswordPolicy).MaxPasswordAge.Days
$expiringSoon = Get-ADUser -Filter {Enabled -eq $true -and PasswordNeverExpires -eq $false} `
    -Properties PasswordLastSet, EmailAddress, DisplayName |
    Select-Object DisplayName, SamAccountName, EmailAddress,
        @{N='DaysUntilExpiry';E={
            $expiry = $_.PasswordLastSet.AddDays($maxPasswordAge)
            [math]::Round(($expiry - (Get-Date)).TotalDays)
        }} |
    Where-Object { $_.DaysUntilExpiry -le 14 -and $_.DaysUntilExpiry -gt 0 } |
    Sort-Object DaysUntilExpiry

# Send expiry notification emails
foreach ($user in $expiringSoon) {
    Send-MailMessage -To $user.EmailAddress `
        -From "noreply@contoso.com" `
        -SmtpServer "smtp.contoso.com" `
        -Subject "Password Expiring in $($user.DaysUntilExpiry) days" `
        -Body "Your password expires in $($user.DaysUntilExpiry) days. Please change it at https://aka.ms/sspr"
}

# Reset password with force change
function Reset-ADUserPassword {
    param(
        [Parameter(Mandatory)]
        [string]$Username,
        [string]$TempPassword = "Welcome@$(Get-Date -Format 'MMMyyyy')!"
    )

    $securePass = ConvertTo-SecureString $TempPassword -AsPlainText -Force
    Set-ADAccountPassword -Identity $Username -NewPassword $securePass -Reset
    Set-ADUser -Identity $Username -ChangePasswordAtLogon $true
    Unlock-ADAccount -Identity $Username
    Write-Log "Password reset for: $Username"
}
```

---

## 9. Network Automation <a name="network-automation"></a>

### Network Scanning & Testing
```powershell
# Parallel ping sweep
function Invoke-PingSweep {
    param(
        [string]$Network = "192.168.1",
        [int]$Start = 1,
        [int]$End = 254,
        [int]$ThrottleLimit = 50
    )

    $Start..$End | ForEach-Object -Parallel {
        $ip = "$using:Network.$_"
        $result = Test-Connection -ComputerName $ip -Count 1 -TimeoutSeconds 1 -Quiet 2>$null
        if ($result) {
            [PSCustomObject]@{
                IP       = $ip
                Online   = $true
                Hostname = try { [System.Net.Dns]::GetHostEntry($ip).HostName } catch { "Unknown" }
            }
        }
    } -ThrottleLimit $ThrottleLimit | Sort-Object IP
}

# Port scanner
function Test-PortRange {
    param(
        [string]$ComputerName,
        [int[]]$Ports = @(22, 23, 80, 135, 389, 443, 445, 3389, 5985, 8080)
    )

    foreach ($port in $Ports) {
        $tcp = New-Object System.Net.Sockets.TcpClient
        $result = $tcp.BeginConnect($ComputerName, $port, $null, $null)
        $wait = $result.AsyncWaitHandle.WaitOne(100, $false)

        [PSCustomObject]@{
            Host  = $ComputerName
            Port  = $port
            State = if ($wait -and -not $tcp.Client.Connected) { "Closed" }
                    elseif ($wait) { "Open" }
                    else { "Filtered" }
        }

        $tcp.Close()
    }
}

# DNS lookup tools
function Resolve-AllDNS {
    param([string]$Hostname)

    [PSCustomObject]@{
        Hostname = $Hostname
        A        = (Resolve-DnsName $Hostname -Type A    -ErrorAction SilentlyContinue).IPAddress
        AAAA     = (Resolve-DnsName $Hostname -Type AAAA -ErrorAction SilentlyContinue).IPAddress
        MX       = (Resolve-DnsName $Hostname -Type MX   -ErrorAction SilentlyContinue).NameExchange
        TXT      = (Resolve-DnsName $Hostname -Type TXT  -ErrorAction SilentlyContinue).Strings
        CNAME    = (Resolve-DnsName $Hostname -Type CNAME -ErrorAction SilentlyContinue).NameHost
        SOA      = (Resolve-DnsName $Hostname -Type SOA  -ErrorAction SilentlyContinue).PrimaryServer
    }
}
```

### Firewall Management
```powershell
# List all enabled firewall rules
Get-NetFirewallRule -Enabled True |
    Select-Object DisplayName, Direction, Action, Profile |
    Sort-Object Direction

# Add firewall rule
New-NetFirewallRule -DisplayName "Allow HTTPS Inbound" `
    -Direction Inbound `
    -Protocol TCP `
    -LocalPort 443 `
    -Action Allow `
    -Profile Domain

# Find rules allowing all traffic (security audit)
Get-NetFirewallRule | Where-Object {
    $_.Action -eq 'Allow' -and $_.Enabled -eq 'True'
} | Get-NetFirewallPortFilter | Where-Object {
    $_.LocalPort -eq 'Any' -and $_.RemotePort -eq 'Any'
}
```

---

## 10. User & Group Management <a name="user-management"></a>

### Local Account Management
```powershell
# List local users
Get-LocalUser | Select-Object Name, Enabled, LastLogon, PasswordRequired

# Create local admin
$password = ConvertTo-SecureString "P@ssw0rd!" -AsPlainText -Force
New-LocalUser -Name "LocalAdmin" -Password $password -FullName "Local Administrator"
Add-LocalGroupMember -Group "Administrators" -Member "LocalAdmin"

# Audit local admins on remote computers
$computers = Get-ADComputer -Filter * | Select-Object -ExpandProperty Name
foreach ($comp in $computers) {
    try {
        $admins = Invoke-Command -ComputerName $comp -ScriptBlock {
            Get-LocalGroupMember -Group "Administrators"
        } -ErrorAction Stop

        foreach ($admin in $admins) {
            [PSCustomObject]@{
                Computer = $comp
                Admin    = $admin.Name
                Type     = $admin.PrincipalSource
            }
        }
    }
    catch { Write-Warning "Could not query $comp`: $_" }
}
```

---

## 11. System Administration <a name="system-admin"></a>

### Service Management
```powershell
# Monitor critical services
$criticalServices = @("W32Time", "DNS", "Netlogon", "ADWS", "WinRM")

$criticalServices | ForEach-Object {
    $svc = Get-Service -Name $_ -ErrorAction SilentlyContinue
    [PSCustomObject]@{
        Service = $_
        Status  = if ($svc) { $svc.Status } else { "Not Found" }
        StartType = if ($svc) { $svc.StartType } else { "N/A" }
    }
} | Format-Table -AutoSize

# Restart service with logging
function Restart-ServiceSafely {
    param([string]$ServiceName, [string]$ComputerName = $env:COMPUTERNAME)

    $svc = Get-Service -Name $ServiceName -ComputerName $ComputerName -ErrorAction Stop
    Write-Log "Stopping $ServiceName on $ComputerName (was: $($svc.Status))"
    Stop-Service -InputObject $svc -Force
    Start-Sleep -Seconds 5
    Start-Service -InputObject $svc
    $svc.Refresh()
    Write-Log "$ServiceName on $ComputerName is now: $($svc.Status)"
}
```

### Disk & Performance
```powershell
# Disk space report for all servers
function Get-DiskSpaceReport {
    param([string[]]$ComputerName)

    $ComputerName | ForEach-Object {
        Get-WmiObject -Class Win32_LogicalDisk -ComputerName $_ -Filter "DriveType=3" |
            Select-Object @{N='Server';E={$_.SystemName}},
                          @{N='Drive';E={$_.DeviceID}},
                          @{N='SizeGB';E={[math]::Round($_.Size/1GB,1)}},
                          @{N='FreeGB';E={[math]::Round($_.FreeSpace/1GB,1)}},
                          @{N='UsedPct';E={[math]::Round(100-($_.FreeSpace/$_.Size*100),1)}},
                          @{N='Status';E={
                              $pct = 100-($_.FreeSpace/$_.Size*100)
                              if ($pct -gt 90) {'CRITICAL'} elseif ($pct -gt 80) {'WARNING'} else {'OK'}
                          }}
    }
}

# Top processes by CPU
Get-Process | Sort-Object CPU -Descending |
    Select-Object -First 20 Name, Id, CPU,
        @{N='MemMB';E={[math]::Round($_.WorkingSet/1MB,1)}},
        @{N='User';E={(Get-Process -Id $_.Id -IncludeUserName -ErrorAction SilentlyContinue).UserName}} |
    Format-Table -AutoSize

# Windows Event Log - last 24 hours errors
Get-WinEvent -FilterHashtable @{
    LogName   = 'System', 'Application'
    Level     = 1, 2  # Critical, Error
    StartTime = (Get-Date).AddHours(-24)
} -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, LevelDisplayName, ProviderName, Message |
    Export-Csv "C:\Reports\Errors_$(Get-Date -Format yyyyMMdd).csv" -NoTypeInformation
```

### Remote Management
```powershell
# Invoke-Command patterns
# Run on multiple computers
Invoke-Command -ComputerName "SRV01","SRV02","SRV03" -ScriptBlock {
    Get-Service -Name "Spooler" | Select-Object Name, Status
}

# Pass variables to remote session
$serviceName = "W3SVC"
Invoke-Command -ComputerName "WebSrv01" -ScriptBlock {
    param($svc)
    Restart-Service -Name $svc -Force
} -ArgumentList $serviceName

# Persistent session
$session = New-PSSession -ComputerName "DC01" -Credential (Get-Credential)
Invoke-Command -Session $session -ScriptBlock { hostname }
Copy-Item -Path "C:\Script.ps1" -Destination "C:\Scripts\" -ToSession $session
Remove-PSSession $session

# WinRM configuration
Enable-PSRemoting -Force
Set-Item WSMan:\localhost\Client\TrustedHosts -Value "*.contoso.com" -Force
Test-WSMan -ComputerName "SRV01"
```

---

## 12. Reporting & Logging <a name="reporting"></a>

### HTML Report Generation
```powershell
function New-HTMLReport {
    param(
        [string]$Title,
        [hashtable[]]$Sections,
        [string]$OutputPath
    )

    $css = @"
    <style>
        body { font-family: Segoe UI, sans-serif; margin: 20px; }
        h1 { color: #0078d4; }
        h2 { color: #106ebe; border-bottom: 2px solid #0078d4; }
        table { border-collapse: collapse; width: 100%; margin-bottom: 20px; }
        th { background: #0078d4; color: white; padding: 8px; text-align: left; }
        td { padding: 6px 8px; border-bottom: 1px solid #ddd; }
        tr:nth-child(even) { background: #f5f5f5; }
        .ok { color: green; font-weight: bold; }
        .warning { color: orange; font-weight: bold; }
        .critical { color: red; font-weight: bold; }
        .timestamp { color: #666; font-size: 12px; }
    </style>
"@

    $html = "<html><head><title>$Title</title>$css</head><body>"
    $html += "<h1>$Title</h1>"
    $html += "<p class='timestamp'>Generated: $(Get-Date -Format 'yyyy-MM-dd HH:mm:ss') | By: $env:USERNAME</p>"

    foreach ($section in $Sections) {
        $html += "<h2>$($section.Title)</h2>"
        $html += $section.Data | ConvertTo-Html -Fragment
    }

    $html += "</body></html>"
    $html | Set-Content $OutputPath
    Write-Host "Report saved: $OutputPath"
}

# Example usage
$diskData = Get-DiskSpaceReport -ComputerName @("SRV01","SRV02")
New-HTMLReport -Title "Daily Server Report" `
    -Sections @(@{Title="Disk Space"; Data=$diskData}) `
    -OutputPath "C:\Reports\daily_$(Get-Date -Format yyyyMMdd).html"
```

### Email Reporting
```powershell
function Send-ITReport {
    param(
        [string]$Subject,
        [string]$Body,
        [string[]]$To,
        [string[]]$Attachments
    )

    $mailParams = @{
        From       = "it-reports@contoso.com"
        To         = $To
        Subject    = "[$((Get-Date).ToString('yyyy-MM-dd'))] $Subject"
        Body       = $Body
        BodyAsHtml = $true
        SmtpServer = "smtp.contoso.com"
    }

    if ($Attachments) {
        $mailParams['Attachments'] = $Attachments
    }

    Send-MailMessage @mailParams
}
```

---

## 13. Scheduled Tasks & Automation <a name="scheduling"></a>

### Task Scheduler
```powershell
# Create scheduled task
$action  = New-ScheduledTaskAction -Execute "PowerShell.exe" `
    -Argument "-NonInteractive -ExecutionPolicy Bypass -File C:\Scripts\DailyReport.ps1"

$trigger = New-ScheduledTaskTrigger -Daily -At "06:00AM"

$settings = New-ScheduledTaskSettingsSet `
    -RunOnlyIfNetworkAvailable `
    -StartWhenAvailable `
    -ExecutionTimeLimit (New-TimeSpan -Hours 1)

$principal = New-ScheduledTaskPrincipal `
    -UserId "CONTOSO\svc-scripts" `
    -LogonType ServiceAccount `
    -RunLevel Highest

Register-ScheduledTask `
    -TaskName "IT Daily Report" `
    -TaskPath "\IT Operations\" `
    -Action $action `
    -Trigger $trigger `
    -Settings $settings `
    -Principal $principal

# Run task immediately
Start-ScheduledTask -TaskName "IT Daily Report" -TaskPath "\IT Operations\"

# Get task history
Get-ScheduledTaskInfo -TaskName "IT Daily Report" | Select-Object LastRunTime, LastTaskResult, NextRunTime
```

### Windows Task Automation Patterns
```powershell
# Monitor and auto-restart services
while ($true) {
    $services = @("W3SVC", "MSSQLSERVER", "WinRM")

    foreach ($svcName in $services) {
        $svc = Get-Service -Name $svcName -ErrorAction SilentlyContinue
        if ($svc -and $svc.Status -ne 'Running') {
            Write-Log "Service $svcName is $($svc.Status) - attempting restart" -Level WARN
            Start-Service -Name $svcName
            Write-Log "Service $svcName restarted" -Level INFO

            # Alert
            Send-ITReport -Subject "Service Restarted: $svcName" `
                -Body "Service $svcName on $env:COMPUTERNAME was found stopped and restarted at $(Get-Date)" `
                -To "it-alerts@contoso.com"
        }
    }

    Start-Sleep -Seconds 60
}
```

---

## 14. REST API Integration <a name="api"></a>

### ServiceNow Integration
```powershell
function New-ServiceNowTicket {
    param(
        [string]$ShortDescription,
        [string]$Description,
        [ValidateSet('1','2','3','4')]
        [string]$Urgency = '3',
        [string]$CallerEmail
    )

    $credentials = [Convert]::ToBase64String(
        [Text.Encoding]::ASCII.GetBytes("$($env:SNOW_USER):$($env:SNOW_PASS)")
    )

    $body = @{
        short_description = $ShortDescription
        description       = $Description
        urgency           = $Urgency
        caller_id         = $CallerEmail
    } | ConvertTo-Json

    $params = @{
        Uri     = "https://contoso.service-now.com/api/now/table/incident"
        Method  = "POST"
        Headers = @{
            Authorization  = "Basic $credentials"
            "Content-Type" = "application/json"
            Accept         = "application/json"
        }
        Body = $body
    }

    $response = Invoke-RestMethod @params
    return $response.result.number
}

# Microsoft Graph API
function Get-GraphAccessToken {
    param([string]$TenantId, [string]$ClientId, [string]$ClientSecret)

    $body = @{
        grant_type    = "client_credentials"
        client_id     = $ClientId
        client_secret = $ClientSecret
        scope         = "https://graph.microsoft.com/.default"
    }

    $token = Invoke-RestMethod -Uri "https://login.microsoftonline.com/$TenantId/oauth2/v2.0/token" `
        -Method POST -Body $body

    return $token.access_token
}

function Get-AllM365Users {
    param([string]$AccessToken)

    $headers = @{ Authorization = "Bearer $AccessToken" }
    $users = @()
    $url = "https://graph.microsoft.com/v1.0/users?`$select=displayName,userPrincipalName,department,jobTitle,accountEnabled,lastSignInDateTime"

    do {
        $response = Invoke-RestMethod -Uri $url -Headers $headers
        $users += $response.value
        $url = $response.'@odata.nextLink'
    } while ($url)

    return $users
}
```

---

## 15. Best Practices & Style Guide <a name="best-practices"></a>

### Coding Standards
```powershell
# ✅ DO: Use approved verbs
function Get-ServerInfo { }
function Set-Configuration { }
function Invoke-Deployment { }
function Test-Connectivity { }

# ❌ DON'T: Use unapproved verbs
function Fetch-Data { }       # Use Get-
function Check-Status { }     # Use Test-
function Run-Script { }       # Use Invoke-

# ✅ DO: Use full cmdlet names in scripts
Get-ChildItem  # Not: gci, dir, ls
Set-Location   # Not: cd
Write-Output   # Not: echo

# ✅ DO: Comment your code
# Check if server is reachable before attempting WMI query
if (-not (Test-Connection -ComputerName $server -Count 1 -Quiet)) {
    Write-Warning "Cannot reach $server - skipping"
    return
}

# ✅ DO: Use -WhatIf for destructive operations
Remove-Item @removeParams -WhatIf  # Test first
Remove-Item @removeParams          # Then execute

# ✅ DO: Handle credentials securely
# Store encrypted password
$password | ConvertTo-SecureString -AsPlainText -Force |
    ConvertFrom-SecureString |
    Set-Content "C:\Secure\cred.txt"

# Retrieve credential
$securePass = Get-Content "C:\Secure\cred.txt" | ConvertTo-SecureString
$credential = New-Object System.Management.Automation.PSCredential("domain\user", $securePass)

# ❌ NEVER: Hardcode credentials
$password = "P@ssword123"  # NEVER DO THIS
```

### Performance Tips
```powershell
# Use -Filter instead of Where-Object for AD/WMI (server-side filtering)
# SLOW:
Get-ADUser -Filter * | Where-Object { $_.Department -eq "IT" }
# FAST:
Get-ADUser -Filter { Department -eq "IT" }

# Use ArrayList or Generic List over arrays for large collections
# SLOW: $arr += $item (creates new array each time)
# FAST: $list.Add($item)

# Suppress output with [void] instead of Out-Null (faster)
[void]$list.Add($item)   # Faster than $list.Add($item) | Out-Null

# Use -AsJob for parallel execution
$job = Get-Process -ComputerName "SRV01" -AsJob
# ... do other work ...
Receive-Job $job

# ForEach-Object -Parallel (PS 7+) for CPU-bound tasks
$servers | ForEach-Object -Parallel {
    Test-Connection -ComputerName $_ -Count 1
} -ThrottleLimit 20
```

### Security Best Practices
| Practice | Implementation |
|----------|---------------|
| No hardcoded creds | Use SecretStore, Key Vault, or encrypted files |
| Least privilege | Run scripts as service account with minimum rights |
| Script signing | Sign all production scripts with code signing cert |
| Logging | Log all significant actions with user, time, action |
| Input validation | Validate all parameters with ValidateScript/Pattern |
| WhatIf support | Add `SupportsShouldProcess` to all state-changing functions |
| Version control | Store scripts in Git with PR review process |
| Error handling | Always use try/catch, never silently swallow errors |

---

*Last Updated: 2025 | IT Operations PowerShell Guide v3.0*
