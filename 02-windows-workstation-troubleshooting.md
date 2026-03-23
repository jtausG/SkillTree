# Windows Workstation Troubleshooting

> **Category:** Troubleshooting | **Audience:** L1/L2 Support | **Platform:** Windows 10/11

---

## Table of Contents
1. [Boot & Startup Issues](#1-boot--startup-issues)
2. [Blue Screen of Death (BSOD)](#2-blue-screen-of-death-bsod)
3. [Performance Issues](#3-performance-issues)
4. [Driver & Hardware Problems](#4-driver--hardware-problems)
5. [Windows Update Issues](#5-windows-update-issues)
6. [Profile & User Account Issues](#6-profile--user-account-issues)
7. [Disk & Storage Issues](#7-disk--storage-issues)
8. [System File Corruption](#8-system-file-corruption)
9. [Domain Join & Authentication Issues](#9-domain-join--authentication-issues)
10. [Remote Desktop Issues](#10-remote-desktop-issues)
11. [Key Diagnostic Commands](#11-key-diagnostic-commands)

---

## 1. Boot & Startup Issues

### Symptoms
- Machine won't POST
- Windows logo freezes / loops
- "No bootable device found"
- Recovery environment boots instead of Windows
- Black screen after login

### Diagnostic Steps

#### Won't POST (No BIOS/UEFI screen)
1. Check power cable and power button connection
2. Reseat RAM modules (one stick at a time if multiple)
3. Remove all peripherals (USB, external drives)
4. Try a different power outlet / power strip
5. Check for POST beep codes (consult motherboard manual)
6. Test with known-good power supply

#### Windows Logo Freezes / Boot Loop
```powershell
# Boot into Recovery Environment (WinRE)
# Method 1: Power off 3 times during boot (forces WinRE)
# Method 2: Shift + Restart from login screen

# In WinRE Command Prompt:
# Check and repair boot record
bootrec /fixmbr
bootrec /fixboot
bootrec /scanos
bootrec /rebuildbcd

# Check disk health
chkdsk C: /f /r /x

# If BitLocker enabled — get recovery key from AD or Azure AD before running chkdsk
manage-bde -status C:
```

#### "No Bootable Device"
1. Verify boot order in BIOS/UEFI (HDD/SSD should be first)
2. Check if drive is detected in BIOS/UEFI storage menu
3. Test SATA/NVMe cable (if desktop)
4. Boot from USB installer → Troubleshoot → Startup Repair
5. If drive not detected: hardware failure — replace drive

#### Black Screen After Login
```powershell
# Boot into Safe Mode first
# In WinRE: Troubleshoot → Advanced Options → Startup Settings → Enable Safe Mode

# Check Explorer process
taskmgr  # In Safe Mode — is explorer.exe running?

# Restart Explorer
taskkill /f /im explorer.exe
explorer.exe

# Check for corrupt user profile (see Section 6)

# Disable startup items
msconfig → Startup tab → Disable all
# Or via Task Manager → Startup tab

# Check for GPU driver issues — roll back or uninstall display adapter
devmgmt.msc → Display adapters → Roll back or uninstall

# Check shell value in registry
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v Shell
# Should be: explorer.exe
# If corrupt: reg add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v Shell /t REG_SZ /d explorer.exe /f
```

---

## 2. Blue Screen of Death (BSOD)

### Reading a BSOD
Modern BSODs show:
- **Stop code** (e.g., `SYSTEM_SERVICE_EXCEPTION`) — Most important field
- **What failed** (e.g., `ntfs.sys`, `nvlddmkm.sys`) — Often points to driver
- **QR code** — Links to Microsoft support

### Common Stop Codes & Fixes

| Stop Code | Common Cause | Fix |
|-----------|-------------|-----|
| `DRIVER_IRQL_NOT_LESS_OR_EQUAL` | Faulty driver | Roll back/update driver, especially NIC or GPU |
| `SYSTEM_SERVICE_EXCEPTION` | Driver or software corruption | Run SFC /scannow, update drivers |
| `CRITICAL_PROCESS_DIED` | Core Windows process failed | SFC, DISM, check for malware |
| `MEMORY_MANAGEMENT` | RAM failure or driver | Run MemTest86, update chipset drivers |
| `PAGE_FAULT_IN_NONPAGED_AREA` | RAM or driver bug | MemTest86, roll back recent driver |
| `NTFS_FILE_SYSTEM` | Disk corruption | chkdsk /f /r |
| `KERNEL_SECURITY_CHECK_FAILURE` | Driver incompatibility | Safe Mode → roll back driver |
| `WHEA_UNCORRECTABLE_ERROR` | Hardware failure (CPU/RAM/MB) | Check temps, MemTest, hardware replacement |
| `DPC_WATCHDOG_VIOLATION` | SSD driver or firmware | Update SSD firmware/drivers |
| `BAD_SYSTEM_CONFIG_INFO` | Registry corruption | System Restore, registry repair |

### Analyzing Minidumps
```powershell
# Minidump location
C:\Windows\Minidump\*.dmp

# Use WinDbg (Windows Debugging Tools) to analyze
# Or upload to https://www.osronline.com/page.cfm?name=analyzeNT (OSR Online)

# Quick analysis with PowerShell (requires debugging symbols)
# Better approach: Use Windows Debugger (WinDbg Preview from Microsoft Store)
# Open .dmp file → !analyze -v

# Enable complete memory dumps for thorough analysis
# System Properties → Advanced → Startup and Recovery → Write debugging information
# Set to "Complete memory dump"
```

### BSOD Recurring Prevention
```powershell
# Check Windows Event Log for critical errors before BSOD
Get-WinEvent -FilterHashtable @{LogName='System'; Level=1,2; StartTime=(Get-Date).AddDays(-7)} | 
  Select-Object TimeCreated, Id, Message | Format-List

# Verify driver signatures
sigverif

# Check for problematic Windows updates
Get-HotFix | Sort-Object InstalledOn -Descending | Select-Object -First 10

# Driver verifier (advanced — only in test environment)
verifier /standard /all  # Enable
verifier /reset           # Disable after debugging
```

---

## 3. Performance Issues

### Slow Startup
```powershell
# Check startup impact in Task Manager → Startup
# Disable non-essential items

# View boot time
(Get-EventLog System | Where-Object EventID -eq 6005 | Select-Object -First 1).TimeGenerated

# Check Superfetch/SysMain service
Get-Service SysMain | Select-Object Status

# Defrag (HDD only — never SSD)
Optimize-Volume -DriveLetter C -Analyze -Verbose

# Review startup event logs
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Diagnostics-Performance/Operational'; Id=100} |
  Select-Object -First 5 | Format-List
```

### High CPU Usage
```powershell
# Find top CPU consumers
Get-Process | Sort-Object CPU -Descending | Select-Object -First 15 Name, CPU, Id

# Identify a specific process
Get-Process -Id <PID> | Select-Object *

# Check for WMI issues (common culprit)
Get-Process WmiPrvSE | Select-Object CPU, WorkingSet

# Check scheduled tasks running
Get-ScheduledTask | Where-Object State -eq Running

# Common offenders:
# - MsMpEng.exe (Windows Defender) — check exclusions, update definitions
# - svchost.exe — use Resource Monitor to identify which service
# - SearchIndexer — index rebuild in progress
# - TiWorker.exe — Windows Update in progress
```

### High Memory Usage
```powershell
# Check memory usage
Get-Process | Sort-Object WorkingSet64 -Descending | Select-Object -First 15 Name, @{N='RAM(MB)';E={[Math]::Round($_.WorkingSet64/1MB,1)}}

# Check for memory leaks (watch process over time)
while ($true) { Get-Process -Name <ProcessName> | Select-Object Name,@{N='RAM(MB)';E={[Math]::Round($_.WorkingSet64/1MB,1)}}; Start-Sleep 60 }

# Check virtual memory / pagefile
$cs = Get-WmiObject -Class Win32_ComputerSystem
"Physical RAM: $([Math]::Round($cs.TotalPhysicalMemory/1GB,2)) GB"
Get-WmiObject Win32_PageFileUsage | Select-Object Name, CurrentUsage, PeakUsage

# Adjust pagefile if needed
# System Properties → Advanced → Performance → Virtual Memory
```

### Slow Application Response
```powershell
# Check disk I/O bottleneck
# Use Resource Monitor → Disk tab → watch for high queue length (>1-2 = bottleneck)
resmon

# Check disk health
Get-PhysicalDisk | Select-Object FriendlyName, MediaType, OperationalStatus, HealthStatus

# SMART data for HDD/SSD
# Install CrystalDiskInfo or use wmic
wmic diskdrive get Model,Status,Size
```

---

## 4. Driver & Hardware Problems

### Device Manager Issues
```
Symbols in Device Manager:
❌ Red X         = Disabled device
⚠️ Yellow !     = Driver issue / unknown device
? Blue ?        = No driver (unknown device)
↓ Down arrow   = Disabled
```

```powershell
# List devices with issues
Get-WmiObject Win32_PnPEntity | Where-Object ConfigManagerErrorCode -ne 0 |
  Select-Object Name, ConfigManagerErrorCode, DeviceID

# Error codes:
# Code 1  — Device not configured correctly
# Code 10 — Device cannot start
# Code 28 — Drivers not installed
# Code 43 — Windows stopped device (hardware fault or driver error)
# Code 52 — Code signing issue (use sigverif)
```

### Driver Update / Rollback
```powershell
# Get all installed drivers
Get-WmiObject Win32_PnPSignedDriver | Select-Object DeviceName, DriverVersion, DriverDate | Sort-Object DriverDate -Descending

# Roll back a driver
# Device Manager → Right-click device → Properties → Driver tab → Roll Back Driver

# Force reinstall driver
# Device Manager → Right-click device → Uninstall device → ☑ Delete driver software → Reboot
# Or: pnputil /delete-driver oem##.inf /uninstall

# Export driver list
driverquery /v /fo csv > C:\Temp\drivers.csv
```

### USB Device Issues
```powershell
# Disable USB selective suspend
powercfg /SETACVALUEINDEX SCHEME_CURRENT 2a737441-1930-4402-8d77-b2bebba308a3 48e6b7a6-50f5-4782-a5d4-53bb8f07e226 0
powercfg /SETACTIVE SCHEME_CURRENT

# Reset USB controllers (Device Manager → Universal Serial Bus controllers → Uninstall all → Reboot)

# Check USB event log
Get-WinEvent -FilterHashtable @{LogName='System'; ProviderName='usb*'} -MaxEvents 20
```

---

## 5. Windows Update Issues

### Common Error Codes
| Error Code | Meaning | Fix |
|-----------|---------|-----|
| 0x80070005 | Access denied | Run Windows Update as admin, check permissions |
| 0x80072EFE | Connection error | Check proxy, firewall, WSUS settings |
| 0x800705B4 | Timeout | Retry, check WSUS connectivity |
| 0x80073701 | Missing assembly | DISM /RestoreHealth |
| 0x8007000D | Data invalid | Reset Windows Update components |
| 0xC1900101 | Driver incompatibility | Update/remove incompatible drivers |
| 0x80240034 | WU agent issue | Reset update components |

### Windows Update Troubleshooting
```powershell
# Run Windows Update Troubleshooter
msdt.exe /id WindowsUpdateDiagnostic

# Reset Windows Update components
net stop bits
net stop wuauserv
net stop appidsvc
net stop cryptsvc
Rename-Item C:\Windows\SoftwareDistribution SoftwareDistribution.old
Rename-Item C:\Windows\System32\catroot2 catroot2.old
net start bits
net start wuauserv
net start appidsvc
net start cryptsvc

# Check for component store corruption
DISM /Online /Cleanup-Image /CheckHealth
DISM /Online /Cleanup-Image /ScanHealth
DISM /Online /Cleanup-Image /RestoreHealth

# Force update scan
wuauclt.exe /detectnow
# Windows 10/11:
UsoClient StartScan

# Clear Windows Update cache
# Stop services → delete contents of C:\Windows\SoftwareDistribution\Download → Start services
```

### WSUS-Specific Issues
```powershell
# Check WSUS server URL
reg query "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate"

# Force report to WSUS
wuauclt /reportnow

# Re-register with WSUS
wuauclt /resetauthorization /detectnow

# Check WUAHandler log (SCCM/ConfigMgr environments)
# C:\Windows\CCM\Logs\WUAHandler.log
```

---

## 6. Profile & User Account Issues

### Temporary Profile / Corrupt Profile
**Symptoms:** User logs in to a "Temporary Profile" message, desktop settings lost, documents may be missing

```powershell
# Check event log for profile errors
Get-WinEvent -FilterHashtable @{LogName='Application'; ProviderName='Microsoft-Windows-User Profiles Service'} -MaxEvents 20

# Identify profile path
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList" /s | findstr "ProfileImagePath"

# Backup old profile data
# Source: C:\Users\<username>.000 or .BAK
# Copy Desktop, Documents, Favorites, AppData to new profile

# Fix: Delete corrupt profile entry in registry
# HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList
# Find the SID with .bak extension — delete the .bak entry, rename remaining entry to remove .bak
```

### Cannot Log In
```powershell
# Account locked out?
Search-ADAccount -LockedOut | Select-Object Name, SamAccountName, LockedOut

# Check from domain controller
Get-ADUser -Identity <username> -Properties LockedOut, BadLogonCount, LastBadPasswordAttempt, PasswordExpired

# Unlock account
Unlock-ADAccount -Identity <username>

# Password expired?
Get-ADUser -Identity <username> -Properties PasswordExpired, PasswordLastSet

# Local account issues (non-domain)
net user <username>
net user <username> * /domain  # Force password change
```

---

## 7. Disk & Storage Issues

```powershell
# Check disk health
Get-PhysicalDisk | Select-Object FriendlyName, MediaType, OperationalStatus, HealthStatus, Size

# Check volume health
Get-Volume | Select-Object DriveLetter, FileSystem, HealthStatus, SizeRemaining, Size

# Run CHKDSK
chkdsk C: /f /r /b     # /f=fix errors, /r=recover bad sectors, /b=re-evaluate bad clusters (for SSDs)
# Note: Will schedule for next reboot on system volume

# Disk cleanup
cleanmgr /sageset:65535
cleanmgr /sagerun:65535

# Check for large files
Get-ChildItem C:\ -Recurse -File -ErrorAction SilentlyContinue |
  Sort-Object Length -Descending |
  Select-Object -First 20 FullName, @{N='Size(MB)';E={[Math]::Round($_.Length/1MB,1)}}

# WinSxS cleanup (Windows component store)
DISM /Online /Cleanup-Image /StartComponentCleanup /ResetBase
```

---

## 8. System File Corruption

```powershell
# Step 1: SFC scan (System File Checker)
sfc /scannow
# Logs: C:\Windows\Logs\CBS\CBS.log

# Step 2: If SFC finds unfixable errors, run DISM first then SFC again
DISM /Online /Cleanup-Image /RestoreHealth
sfc /scannow

# Step 3: If DISM fails (can't reach Windows Update)
# Mount Windows ISO and point DISM to it
DISM /Online /Cleanup-Image /RestoreHealth /Source:D:\Sources\install.wim /LimitAccess

# Verify SFC results
findstr /c:"[SR]" %windir%\Logs\CBS\CBS.log | tail -50

# Repair Windows 11 image offline (from WinRE)
DISM /Image:C:\ /Cleanup-Image /RestoreHealth /Source:D:\Sources\install.wim
```

---

## 9. Domain Join & Authentication Issues

```powershell
# Test domain connectivity
nltest /sc_verify:<domain.com>
nltest /dsgetdc:<domain.com>

# Check secure channel
Test-ComputerSecureChannel -Verbose

# Reset secure channel (requires local admin)
Test-ComputerSecureChannel -Repair -Credential (Get-Credential domain\adminaccount)
# Or: netdom resetpwd /server:<DC> /userd:<domain\admin> /passwordd:*

# Check Kerberos tickets
klist
klist purge  # Clear cached tickets — forces re-authentication

# Verify DNS resolves domain controllers
nslookup -type=SRV _ldap._tcp.dc._msdcs.<domain.com>

# Check machine account in AD
Get-ADComputer -Identity <ComputerName> -Properties LastLogonDate, Enabled

# Rejoin domain (last resort — will lose local profiles)
Remove-Computer -WorkgroupName WORKGROUP -Force -Restart
# Then: Add-Computer -DomainName <domain.com> -Credential (Get-Credential) -Restart
```

---

## 10. Remote Desktop Issues

```powershell
# Verify RDP is enabled
Get-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server' -Name fDenyTSConnections
# 0 = RDP enabled, 1 = RDP disabled

# Enable RDP remotely (requires WMI access)
Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server' -Name fDenyTSConnections -Value 0

# Check firewall rule
Get-NetFirewallRule -Name "RemoteDesktop*" | Select-Object Name, Enabled, Direction

# Enable firewall rule
Enable-NetFirewallRule -Name "RemoteDesktop-UserMode-In-TCP"

# Check RDP port (default 3389)
netstat -an | findstr 3389
Test-NetConnection -ComputerName <host> -Port 3389

# Check RDP services
Get-Service TermService | Select-Object Name, Status, StartType

# View active RDP sessions
query session /server:<hostname>

# Disconnect session
logoff <SessionID> /server:<hostname>

# Certificate issues
# RDP certificate stored in: HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp
```

---

## 11. Key Diagnostic Commands

### System Information
```powershell
systeminfo                          # Full system info
Get-ComputerInfo                    # PowerShell equivalent
winver                              # Windows version
msinfo32                            # Comprehensive system info GUI
```

### Event Logs
```powershell
# Last 50 errors in System log
Get-EventLog -LogName System -EntryType Error -Newest 50

# Specific event ID
Get-WinEvent -FilterHashtable @{LogName='System'; Id=41}  # Kernel power failure

# Export event log
wevtutil epl System C:\Temp\System.evtx
```

### Network
```powershell
ipconfig /all
netstat -ano
Get-NetAdapter | Select-Object Name, Status, LinkSpeed, MacAddress
Test-NetConnection google.com -Port 443
Resolve-DnsName google.com
```

### Disk
```powershell
Get-Disk
Get-Partition
Get-Volume
diskpart  # Interactive disk management
```

### Process & Services
```powershell
Get-Process | Sort-Object CPU -Descending
Get-Service | Where-Object Status -eq Stopped | Where-Object StartType -eq Automatic
sc query <ServiceName>
```

---

*See also: [General Troubleshooting Methodology](01-general-troubleshooting-methodology.md) | [Network Connectivity Troubleshooting](04-network-connectivity-troubleshooting.md)*
