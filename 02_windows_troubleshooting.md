# Windows Troubleshooting Guide

## Overview
Detailed runbook for diagnosing and resolving common Windows OS, application, and user issues in enterprise environments.

---

## Table of Contents
1. [Boot & Startup Issues](#boot)
2. [Blue Screen of Death (BSOD)](#bsod)
3. [Performance Issues](#performance)
4. [User Profile Issues](#user-profile)
5. [Application Issues](#application)
6. [Windows Update Issues](#updates)
7. [Printing Issues](#printing)
8. [Remote Desktop Issues](#rdp)
9. [File & Permission Issues](#files)
10. [Registry Troubleshooting](#registry)
11. [PowerShell Diagnostic Scripts](#powershell)

---

## 1. Boot & Startup Issues <a name="boot"></a>

### Symptoms
- Machine won't boot / stuck at logo
- Slow boot times (>2 minutes)
- Boot loop
- "Repairing disk" loop

### Diagnostic Steps

```powershell
# Check last successful boot
Get-EventLog -LogName System -InstanceId 6005 -Newest 5

# Check for boot errors
Get-EventLog -LogName System -EntryType Error -Newest 20

# View startup programs
Get-CimInstance Win32_StartupCommand | Select-Object Name, Command, Location

# Check BCD (Boot Configuration Data)
bcdedit /enum all
```

### Common Fixes

**Stuck at boot:**
```
1. Boot to Windows Recovery (F8 or recovery media)
2. Startup Repair → Automatic repair
3. If fails: bootrec /fixmbr → bootrec /fixboot → bootrec /rebuildbcd
```

**Slow boot:**
```powershell
# Disable unnecessary startup items
Get-CimInstance Win32_StartupCommand

# Check boot performance logs
Get-WinEvent -ProviderName Microsoft-Windows-Diagnostics-Performance | Where-Object {$_.Id -eq 100} | Select-Object -First 10 | Format-List

# Enable Fast Startup (if not on SSD or causing issues)
powercfg /hibernate on
```

**Boot loop after update:**
```
1. Boot to Safe Mode (F8 → Safe Mode)
2. Uninstall recent Windows Update:
   Settings → Update & Security → View Update History → Uninstall Updates
   
# Via DISM/PowerShell
Get-WindowsPackage -Online | Where-Object PackageState -eq Installed | Sort-Object InstallTime -Descending | Select-Object -First 5
Remove-WindowsPackage -Online -PackageName <PackageName>
```

---

## 2. Blue Screen of Death (BSOD) <a name="bsod"></a>

### Reading Stop Codes
| Stop Code | Common Cause |
|---|---|
| `IRQL_NOT_LESS_OR_EQUAL` | Driver issue, RAM |
| `SYSTEM_SERVICE_EXCEPTION` | Driver/software conflict |
| `PAGE_FAULT_IN_NONPAGED_AREA` | RAM or driver bug |
| `CRITICAL_PROCESS_DIED` | Corrupt system files |
| `BAD_POOL_HEADER` | Memory corruption |
| `NTFS_FILE_SYSTEM` | Disk error |
| `DPC_WATCHDOG_VIOLATION` | Driver timeout |

### Analyzing Minidump Files

```powershell
# Minidumps location
ls C:\Windows\Minidump\

# Configure dump type (requires reboot)
# 1 = Small memory dump
# 2 = Kernel memory dump  
# 3 = Complete memory dump
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\CrashControl" -Name "CrashDumpEnabled" -Value 2

# View crashes with WinDBG or use WinEvent
Get-WinEvent -LogName System | Where-Object {$_.Id -eq 41} | Select-Object -First 10
```

### BSOD Resolution Workflow

```
1. Note exact stop code and module name
2. Locate minidump: C:\Windows\Minidump\
3. Open in WinDbg: !analyze -v
4. Identify offending driver/module
5. Update or roll back the driver
6. Run: sfc /scannow
7. Run: DISM /Online /Cleanup-Image /RestoreHealth
8. Run: mdsched.exe (Windows Memory Diagnostic)
9. Check disk: chkdsk C: /f /r /x
```

---

## 3. Performance Issues <a name="performance"></a>

### CPU High Usage

```powershell
# Find top CPU-consuming processes
Get-Process | Sort-Object CPU -Descending | Select-Object -First 10 | Format-Table Name, Id, CPU, WorkingSet

# Find processes with high CPU per second (live)
while($true) { 
  Get-Process | Sort-Object CPU -Descending | Select-Object -First 5
  Start-Sleep 2
  Clear-Host 
}

# Check for scheduled tasks running
Get-ScheduledTask | Where-Object State -eq Running

# Check WMI activity (common CPU hog)
Get-Process wmiprvse | Select-Object Id, CPU, WorkingSet
```

### RAM High Usage

```powershell
# Memory summary
Get-CimInstance Win32_OperatingSystem | Select-Object TotalVisibleMemorySize, FreePhysicalMemory

# Top RAM consumers
Get-Process | Sort-Object WorkingSet -Descending | Select-Object -First 15 Name, Id, @{N='RAM(MB)';E={[Math]::Round($_.WorkingSet/1MB,1)}}

# Check for memory leaks (watch a process over time)
$proc = Get-Process -Name "someapp"
while($true) {
  $proc.Refresh()
  Write-Host "$(Get-Date): $($proc.WorkingSet/1MB) MB"
  Start-Sleep 30
}
```

### Disk I/O Issues

```powershell
# Check disk health via SMART
Get-PhysicalDisk | Select-Object FriendlyName, OperationalStatus, HealthStatus

# Run chkdsk (schedule for next boot)
chkdsk C: /scan

# Check disk queue length via perfmon counter
(Get-Counter '\PhysicalDisk(*)\Current Disk Queue Length').CounterSamples | Select-Object InstanceName, CookedValue

# Find large files
Get-ChildItem C:\ -Recurse -ErrorAction SilentlyContinue | Sort-Object Length -Descending | Select-Object -First 20 FullName, @{N='Size(MB)';E={[Math]::Round($_.Length/1MB,1)}}
```

---

## 4. User Profile Issues <a name="user-profile"></a>

### Temporary Profile
Symptoms: Desktop/settings reset on each login, profile path ends in `.bak` or `TEMP`

```powershell
# Check profile list
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList\*" | 
  Select-Object PSChildName, ProfileImagePath, State

# State values:
# 0 = Normal
# 1 = Temporary
# 4 = Mandatory

# Fix: Remove corrupt profile entry
# 1. Back up user data
# 2. Remove profile from registry (be careful!)
Remove-Item "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList\<SID>" -Recurse

# 3. Delete C:\Users\<username> (requires admin / different account logged in)
# 4. User logs in fresh - new profile created
```

### Profile Corruption

```
Resolution Steps:
1. Log in as admin (not the affected user)
2. Copy C:\Users\<baduser> to a backup location
3. Rename C:\Users\<baduser> to C:\Users\<baduser>.OLD
4. Remove registry key for that SID under ProfileList
5. Have user log back in (new profile created)
6. Manually copy documents, desktop, favorites from .OLD folder
```

### Roaming Profile Issues

```powershell
# Check roaming profile path
net user <username> /domain | Select-String "Profile"

# Force local profile (GPO preferred)
Set-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" -Name "DefaultUserName" -Value ""

# Check profile sync events
Get-WinEvent -LogName "Microsoft-Windows-User Profiles Service/Operational" | Select-Object -First 20
```

---

## 5. Application Issues <a name="application"></a>

### Application Crashes

```powershell
# Check application crash events
Get-EventLog -LogName Application -Source "Application Error" -Newest 20 | 
  Select-Object TimeGenerated, Message | Format-List

# Check Windows Error Reporting
Get-ChildItem "$env:LOCALAPPDATA\CrashDumps" -ErrorAction SilentlyContinue

# Check .NET errors
Get-EventLog -LogName Application -Source ".NET Runtime" -Newest 10
```

### Application Won't Start

```
Checklist:
[ ] Check Event Viewer → Application log for errors
[ ] Verify the application service is running (if applicable)
[ ] Check for missing DLLs (Dependency Walker or Process Monitor)
[ ] Verify .NET/Visual C++ Redistributable versions
[ ] Run as Administrator (UAC issue?)
[ ] Compatibility mode (right-click → Properties → Compatibility)
[ ] Clear application cache
[ ] Repair or reinstall the application
[ ] Check antivirus exclusions
```

### Office-Specific Issues

```powershell
# Repair Office
# Quick Repair (no internet needed)
# Online Repair (requires internet, more thorough)
# Via Settings → Apps → Microsoft Office → Modify

# Reset Office activation
cscript "C:\Program Files\Microsoft Office\Office16\OSPP.VBS" /dstatus
cscript "C:\Program Files\Microsoft Office\Office16\OSPP.VBS" /rearm

# Clear Office credential cache
cmdkey /list | Select-String "MicrosoftOffice"
cmdkey /delete:MicrosoftOffice16...

# Fix Outlook profile
outlook.exe /resetnavpane
outlook.exe /profiles
```

---

## 6. Windows Update Issues <a name="updates"></a>

### Common Error Codes
| Code | Meaning |
|---|---|
| 0x80070005 | Access denied |
| 0x80070057 | Invalid parameter |
| 0x8007000D | Invalid data |
| 0x80073712 | Component store corrupted |
| 0x800705B4 | Timeout |
| 0x80240034 | WU_E_DOWNLOAD_FAILED |
| 0x80242006 | WU_E_UH_INVALIDMETADATA |

### Windows Update Troubleshooter

```powershell
# Reset Windows Update components
net stop wuauserv
net stop cryptSvc
net stop bits
net stop msiserver

Rename-Item C:\Windows\SoftwareDistribution SoftwareDistribution.old -ErrorAction SilentlyContinue
Rename-Item C:\Windows\System32\catroot2 catroot2.old -ErrorAction SilentlyContinue

net start wuauserv
net start cryptSvc
net start bits
net start msiserver

# Force update check
wuauclt /detectnow
(New-Object -ComObject Microsoft.Update.AutoUpdate).DetectNow()

# DISM repair (common fix for update issues)
DISM /Online /Cleanup-Image /CheckHealth
DISM /Online /Cleanup-Image /ScanHealth
DISM /Online /Cleanup-Image /RestoreHealth

# SFC after DISM
sfc /scannow
```

---

## 7. Printing Issues <a name="printing"></a>

### Print Spooler Reset

```powershell
# Clear print queue and restart spooler
Stop-Service -Name Spooler -Force
Remove-Item "$env:SystemRoot\System32\spool\PRINTERS\*" -Recurse -Force -ErrorAction SilentlyContinue
Start-Service -Name Spooler

# List installed printers
Get-Printer | Select-Object Name, DriverName, PortName, PrinterStatus

# Check spooler errors
Get-EventLog -LogName System -Source "Print Spooler" -Newest 20
```

### Driver Issues

```powershell
# List printer drivers
Get-PrinterDriver | Select-Object Name, InfPath

# Remove a printer driver
Remove-PrinterDriver -Name "HP LaserJet 400 PCL6"

# Add printer via command line
Add-Printer -Name "MyPrinter" -DriverName "Microsoft Print To PDF" -PortName "PORTPROMPT:"
```

---

## 8. Remote Desktop Issues <a name="rdp"></a>

### RDP Connection Failures

```powershell
# Check RDP service status
Get-Service TermService

# Check if RDP is enabled
(Get-ItemProperty "HKLM:\System\CurrentControlSet\Control\Terminal Server").fDenyTSConnections
# 0 = Enabled, 1 = Disabled

# Enable RDP via registry
Set-ItemProperty -Path "HKLM:\System\CurrentControlSet\Control\Terminal Server" -Name fDenyTSConnections -Value 0

# Check firewall for RDP
Get-NetFirewallRule -DisplayName "*Remote Desktop*" | Select-Object DisplayName, Enabled, Direction, Action

# Enable RDP in firewall
Enable-NetFirewallRule -DisplayGroup "Remote Desktop"

# Check listening port
netstat -ano | Select-String ":3389"

# Test RDP connectivity from another machine
Test-NetConnection -ComputerName server01 -Port 3389
```

### NLA (Network Level Authentication) Issues

```powershell
# Check NLA requirement
(Get-WmiObject -Class Win32_TerminalServiceSetting -Namespace root\cimv2\TerminalServices).UserAuthenticationRequired
# 0 = Disabled, 1 = Enabled (NLA required)

# Disable NLA (less secure, use for troubleshooting only)
$TSSettings = Get-WmiObject -Class Win32_TerminalServiceSetting -Namespace root\cimv2\TerminalServices
$TSSettings.SetUserAuthenticationRequired(0)
```

---

## 9. File & Permission Issues <a name="files"></a>

### Access Denied Errors

```powershell
# Check current permissions
(Get-Acl "C:\folder\file.txt").Access | Format-Table IdentityReference, FileSystemRights, AccessControlType

# Take ownership
takeown /f "C:\folder" /r /d y

# Reset permissions (use carefully)
icacls "C:\folder" /reset /t /c /q

# Add specific permission
icacls "C:\folder" /grant "DOMAIN\username:(OI)(CI)M"

# Check who has access to a share
Get-SmbShareAccess -Name "ShareName"
```

### File in Use / Cannot Delete

```powershell
# Find which process has a file open (Sysinternals Handle)
handle.exe "filename.txt"

# Using PowerShell (limited)
$file = "C:\path\to\file.txt"
$processes = Get-Process | Where-Object { $_.Modules.FileName -like "*$file*" }

# Force delete (after identifying and closing the process)
# Option 1: Close the locking process
Stop-Process -Id <PID>

# Option 2: Schedule deletion at next boot
# Use Sysinternals MoveFile tool
```

---

## 10. Registry Troubleshooting <a name="registry"></a>

### Best Practices
> ⚠️ **Always back up the registry before making changes!**

```powershell
# Export a registry key
reg export "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" C:\backup\run_backup.reg

# Import a registry key
reg import C:\backup\run_backup.reg

# Query a value
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion" | Select-Object ProductName, ReleaseId, CurrentBuildNumber
```

### Common Registry Fixes

```powershell
# Fix broken file associations
# Reset .exe association
cmd /c assoc .exe=exefile
cmd /c ftype exefile="%1" %*

# Fix UAC issues
Set-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name "EnableLUA" -Value 1

# Fix network issues - reset TCP/IP
netsh int ip reset
netsh winsock reset
```

---

## 11. PowerShell Diagnostic Scripts <a name="powershell"></a>

### System Health Check Script

```powershell
# Comprehensive system health snapshot
$report = @{}

# OS Info
$os = Get-CimInstance Win32_OperatingSystem
$report["OS"] = "$($os.Caption) Build $($os.BuildNumber)"
$report["Uptime"] = (Get-Date) - $os.LastBootUpTime

# CPU
$cpu = Get-CimInstance Win32_Processor
$report["CPU_Load"] = "$($cpu.LoadPercentage)%"

# Memory
$mem = Get-CimInstance Win32_OperatingSystem
$report["RAM_Free_GB"] = [Math]::Round($mem.FreePhysicalMemory/1MB, 2)
$report["RAM_Total_GB"] = [Math]::Round($mem.TotalVisibleMemorySize/1MB, 2)

# Disk
Get-PSDrive -PSProvider FileSystem | ForEach-Object {
  $report["Disk_$($_.Name)_Free_GB"] = [Math]::Round($_.Free/1GB, 2)
}

# Top processes
$report["Top_CPU_Process"] = (Get-Process | Sort-Object CPU -Descending | Select-Object -First 1).ProcessName

# Network
$adapters = Get-NetAdapter | Where-Object Status -eq "Up"
$report["Active_NICs"] = $adapters.Name -join ", "

# Output
$report | Format-Table -AutoSize

# Recent errors
Write-Host "`n=== Recent System Errors ===" -ForegroundColor Red
Get-EventLog -LogName System -EntryType Error -Newest 5 | 
  Select-Object TimeGenerated, Source, Message | Format-List
```
