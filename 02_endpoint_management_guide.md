# Endpoint Management & Deployment Guide

## Overview
Comprehensive guide for managing Windows endpoints in enterprise environments. Covers imaging, MDM/Intune deployment, configuration management, patch management, and endpoint lifecycle.

---

## 1. Endpoint Deployment Strategy

### Deployment Methods Comparison
| Method | Best For | Pros | Cons |
|--------|---------|------|------|
| Windows Autopilot | Modern, cloud-joined | Zero-touch, no imaging | Requires internet, AAD/Intune |
| SCCM/MECM OSD | On-prem, complex requirements | Full control, offline capable | Complex infrastructure, slow |
| MDT (lite-touch) | Smaller environments | Free, flexible | Manual steps required |
| Provisioning Packages | Bulk kiosk/shared devices | Fast, no server required | Limited customization |
| Fresh start/Reset | Refresh existing devices | Simple, built-in | Longer than reimaging |

### Modern Deployment Decision Tree
```
New device arriving?
├── YES: Does it support Autopilot?
│   ├── YES: Use Windows Autopilot (cloud-native)
│   └── NO: Use SCCM/MDT OSD
└── NO (reprovisioning existing):
    ├── Need full OS reinstall? → SCCM OSD or Reset + Autopilot
    └── Just re-enroll MDM? → Unenroll + re-enroll Autopilot
```

---

## 2. Windows Autopilot

### Prerequisites
- Windows 10/11 devices
- Azure AD Premium P1 or P2
- Intune license
- Hardware hash registered in Autopilot
- Device connected to internet during OOBE

### Hardware Hash Registration

**From running Windows device:**
```powershell
# Method 1: Use Get-WindowsAutoPilotInfo script
Install-Script -Name Get-WindowsAutoPilotInfo -Force
Get-WindowsAutoPilotInfo -OutputFile AutoPilotHWID.csv

# Method 2: Manual hash extraction
$session = New-CimSession
$devDetail = Get-CimInstance -CimSession $session -Namespace root/cimv2/mdm/dmmap `
    -ClassName MDM_DevDetail_Ext01 -Filter "InstanceID='Ext' AND ParentID='./DevDetail'"
$hash = $devDetail.DeviceHardwareData

[PSCustomObject]@{
    "Device Serial Number" = (Get-CimInstance -ClassName Win32_BIOS).SerialNumber
    "Windows Product ID"   = ""
    "Hardware Hash"        = $hash
} | Export-Csv "autopilot_hash.csv" -NoTypeInformation
```

**Upload to Intune:**
1. Intune > Windows enrollment > Devices
2. Import CSV file
3. Wait for sync (up to 30 min)
4. Assign profile to device or group

### Autopilot Profiles
**User-Driven (most common):**
- User signs in with corporate credentials during OOBE
- Device joins Azure AD
- Intune policies deploy automatically
- User gets personalized desktop

**Self-Deploying (kiosk/shared):**
- No user interaction during setup
- Device-based authentication (TPM required)
- Good for kiosks, meeting room PCs

**Pre-Provisioned (White Glove):**
- IT pre-stages apps/configs
- User just completes final personalization
- Fastest end-user experience

### Autopilot Troubleshooting
| Error | Meaning | Resolution |
|-------|---------|-----------|
| 801c03ed | Device not registered in Autopilot | Register hardware hash |
| 801c0003 | User not licensed for Intune | Assign Intune license |
| 80070774 | MDM enrollment failed | Check MDM scope settings |
| 0x800705b4 | Timeout waiting for policies | Check Intune connectivity |
| 80180014 | Not authorized to enroll | Check enrollment restrictions |

**Diagnostics:**
```powershell
# Run on device during/after OOBE issues
# Export Autopilot diagnostic logs
MdmDiagnosticsTool.exe -area Autopilot -cab C:\AutopilotDiag.cab

# Event logs
Get-WinEvent -LogName "Microsoft-Windows-AAD/Operational" | Select -First 50
Get-WinEvent -LogName "Microsoft-Windows-DeviceManagement-Enterprise-Diagnostics-Provider/Admin"
```

---

## 3. Microsoft Intune (Endpoint Manager)

### Enrollment Types

**Windows:**
- Automatic enrollment via Azure AD join (OOBE or Settings)
- Autopilot (preferred)
- Enrollment manager (shared/kiosk)
- Bulk enrollment (provisioning package)

**macOS:**
- Apple Business Manager + Automated Device Enrollment
- Company Portal app enrollment

**iOS/Android:**
- Company Portal app
- Android Enterprise (Work Profile or Fully Managed)

### Configuration Profiles

**Common Profile Types:**
| Profile Type | Examples |
|-------------|---------|
| Device restrictions | Block cameras, app store, USB |
| Endpoint protection | Defender settings, Firewall |
| Wi-Fi | Corporate SSID auto-connect |
| VPN | Always On VPN, per-app VPN |
| Certificate | SCEP, PKCS, Trusted root |
| Custom (OMA-URI) | Advanced settings not in UI |
| Administrative templates | Group Policy-equivalent |
| Settings catalog | Comprehensive OS settings |

**Create Profile via PowerShell/Graph:**
```powershell
Connect-MgGraph -Scopes "DeviceManagementConfiguration.ReadWrite.All"

# Create device restriction profile
$profile = @{
    "@odata.type" = "#microsoft.graph.windows10GeneralConfiguration"
    displayName = "Corporate - Device Restrictions"
    description = "Standard device restriction policy"
    cameraBlocked = $false
    cortanaBlocked = $false
    defenderMonitorFileActivity = "monitorAllFiles"
    storageBlockRemovableStorage = $true
    usbBlocked = $false
    passwordRequired = $true
    passwordMinimumLength = 8
    passwordRequiredType = "alphanumeric"
    passwordMinutesOfInactivityBeforeScreenTimeout = 15
}

Invoke-MgGraphRequest -Method POST `
    -Uri "https://graph.microsoft.com/v1.0/deviceManagement/deviceConfigurations" `
    -Body ($profile | ConvertTo-Json) `
    -ContentType "application/json"
```

### Compliance Policies

**Windows Compliance Policy:**
```json
{
    "@odata.type": "#microsoft.graph.windows10CompliancePolicy",
    "displayName": "Windows - Corporate Compliance",
    "bitLockerEnabled": true,
    "secureBootEnabled": true,
    "codeIntegrityEnabled": true,
    "storageRequireEncryption": true,
    "passwordRequired": true,
    "passwordMinimumLength": 8,
    "passwordRequiredType": "alphanumeric",
    "passwordMinutesOfInactivityBeforeLock": 15,
    "osMinimumVersion": "10.0.19044",
    "defenderEnabled": true,
    "defenderVersion": "",
    "signatureOutOfDate": false,
    "rtpEnabled": true,
    "antivirusRequired": true,
    "antiSpywareRequired": true,
    "deviceThreatProtectionEnabled": true,
    "deviceThreatProtectionRequiredSecurityLevel": "medium"
}
```

**Non-Compliance Actions:**
1. Send email to user (immediate)
2. Send push notification (immediate)
3. Mark device non-compliant (after 1 day grace)
4. Retire device (after 30 days — use carefully)

### App Deployment

**Win32 App Deployment:**
1. Package app with IntuneWinAppUtil.exe
```cmd
IntuneWinAppUtil.exe -c "C:\AppSource\7zip" -s "7z2301-x64.exe" -o "C:\Output"
```
2. Upload .intunewin file to Intune
3. Configure install/uninstall commands
4. Set detection rules
5. Assign to groups

**Common Detection Methods:**
```powershell
# Registry detection
HKLM:\Software\7-Zip
Value: Path
Type: String
Detection method: Key exists

# File detection
%ProgramFiles%\7-Zip\7z.exe
Detection: File exists

# MSI product code detection
{23170F69-40C1-2702-2301-000001000000}  # 7-Zip example
```

**Microsoft Store (WinGet) Apps:**
- Intune > Apps > All apps > Add > Microsoft Store app (new)
- Search WinGet package ID
- Assign to groups

**Office 365 Apps Deployment:**
1. Apps > All apps > Add > Microsoft 365 Apps (Windows 10)
2. Configure suite: Select apps (Word, Excel, Outlook, Teams, etc.)
3. Update channel: Current, Monthly Enterprise, Semi-Annual
4. Assign to All Devices or All Users group

---

## 4. SCCM / MECM (On-Premises)

### OS Deployment (OSD) Task Sequence

**Task Sequence Steps (Typical):**
```
1. Restart in WinPE
2. Partition Disk (UEFI or BIOS)
   - System (500MB EFI)
   - Windows (Remaining)
   - Recovery (500MB)
3. Apply OS Image
4. Apply Windows Settings
5. Apply Network Settings
6. Setup Windows and ConfigMgr
7. Install Applications
   - Office 365
   - 7-Zip
   - PDF Reader
   - AV Client
8. Run PowerShell - Configure settings
9. Install Updates
10. Join Domain (if applicable)
11. Run Powershell - Rename computer
12. Restart Computer
```

**Driver Management:**
- Import drivers per hardware model
- Create driver packages per model
- Use "Auto Apply Drivers" or model-specific packages

**PXE Boot Checklist:**
- [ ] DHCP options 66 (TFTP server IP) and 67 (boot file) configured
- [ ] WDS role installed on DP
- [ ] DP configured for PXE
- [ ] Firewall allows TFTP (UDP 69) and 4011
- [ ] Boundary groups include deployment point

### Software Update Point (SUP)

**WSUS/SUP Configuration:**
```powershell
# Sync categories (typical for enterprise)
# Products: Windows 10, Windows 11, Windows Server 2019/2022
#           Microsoft 365 Apps, SQL Server, .NET Framework
# Classifications: Critical Updates, Security Updates, Definition Updates
#                 Feature Packs, Service Packs, Update Rollups

# Check SUP sync status
Get-CMSoftwareUpdateSyncStatus

# Force sync
Invoke-CMSoftwareUpdateSummarizationSchedule
```

---

## 5. Patch Management

### Patch Strategy (Ring-Based)
```
Ring 0 - Pilot (5% of org): Day 1 of release
    → IT staff, tech-savvy volunteers
Ring 1 - Early Adopters (15%): Day 7
    → Power users by department
Ring 2 - General (60%): Day 21
    → Standard users
Ring 3 - Late Adopters (15%): Day 35
    → Critical systems, exceptions
Ring 4 - Deferred (5%): Day 60+
    → Locked-down, production-critical
```

### Intune Update Rings (Windows)
```json
{
    "displayName": "Ring 2 - General",
    "description": "Standard user workstations - 21 day deferral",
    "qualityUpdatesDeferralPeriodInDays": 21,
    "featureUpdatesDeferralPeriodInDays": 90,
    "businessReadyUpdatesOnly": "all",
    "automaticUpdateMode": "autoInstallAndRebootAtScheduledTime",
    "scheduledInstallDay": "saturday",
    "scheduledInstallTime": 3,
    "updateNotificationLevel": "defaultNotifications",
    "allowWindows11Upgrade": false
}
```

### Patch Tuesday Workflow
```
Patch Tuesday (2nd Tuesday of month):
T+0:  Microsoft releases patches
T+1:  Review patch content, known issues, CVE severity
T+3:  Deploy to Ring 0 (Pilot)
T+7:  Validate Ring 0, check for issues
T+10: Deploy to Ring 1 (Early Adopters)
T+14: Validate Ring 1
T+17: Deploy to Ring 2 (General)
T+21: Deploy to Ring 3 (Late Adopters)
T+35: Deploy to Ring 4 (Deferred)

Out-of-Band (Critical security patches):
T+0:  Evaluate severity (is it actively exploited?)
T+0:  If CVSS 9.0+ or actively exploited → Emergency ring
T+1:  Deploy to all rings with accelerated schedule
```

---

## 6. BitLocker Encryption

### Deploy BitLocker via Intune
1. Create Endpoint Protection profile
2. Configure BitLocker settings:
   - OS drives: Require encryption, XTS-AES 256
   - Recovery key: Store in Azure AD
   - Pre-boot PIN: Require for high-security devices
3. Assign to device groups

### BitLocker Recovery Key Management
```powershell
# Find recovery key in Azure AD / Entra ID
Connect-MgGraph -Scopes "BitlockerKey.ReadBasic.All"

# Get keys for specific device
$deviceId = (Get-MgDevice -Filter "displayName eq 'DESKTOP-ABC123'").Id
Get-MgInformationProtectionBitlockerRecoveryKey -Filter "deviceId eq '$deviceId'" |
    Select Id, VolumeType, CreatedDateTime, DeviceId

# Get specific key value (requires BitlockerKey.Read.All)
Get-MgInformationProtectionBitlockerRecoveryKey -BitlockerRecoveryKeyId <keyId> -Property key
```

```powershell
# On-premises: Get key from AD
Get-ADObject -Filter {objectClass -eq 'msFVE-RecoveryInformation'} `
    -SearchBase "CN=COMPUTER-NAME,OU=Workstations,DC=domain,DC=com" `
    -Properties msFVE-RecoveryPassword |
    Select Name, msFVE-RecoveryPassword
```

**BitLocker Status Checks:**
```powershell
# Local check
manage-bde -status C:

# Remote check via PowerShell
Invoke-Command -ComputerName "ws-001" -ScriptBlock {
    Get-BitLockerVolume | Select MountPoint, EncryptionMethod, VolumeStatus, ProtectionStatus, EncryptionPercentage
}
```

---

## 7. Endpoint Security Baseline

### CIS Benchmark Implementation (Windows)
Key settings via Intune Settings Catalog or GPO:

**Account Policies:**
- Password minimum length: 14
- Password complexity: Enabled
- Password history: 24
- Max password age: 60 days
- Account lockout threshold: 5 attempts
- Lockout duration: 15 minutes

**Local Policies:**
- Audit logon events: Success and Failure
- Audit account management: Success and Failure
- Audit policy change: Success
- Network access: Do not allow anonymous SID/Name translations
- Interactive logon: Don't display last username

**Windows Defender Antivirus:**
- Real-time protection: Enabled
- Behavior monitoring: Enabled
- IOAV protection: Enabled
- Network protection: Enabled
- PUA protection: Block
- Signature update interval: 4 hours
- Cloud-delivered protection: Enabled
- Automatic sample submission: Enabled

**Attack Surface Reduction (ASR) Rules:**
```powershell
# Enable via Intune or Defender CSP
# Block Office apps from creating child processes
# Block credential stealing from LSASS
# Block execution of potentially obfuscated scripts
# Block Win32 API calls from Office macros
# Block executable content from email/webmail
# Use advanced protection against ransomware

# Check ASR rule status
Get-MpPreference | Select AttackSurfaceReductionRules_Ids, AttackSurfaceReductionRules_Actions
```

---

## 8. Endpoint Lifecycle Management

### Hardware Refresh Lifecycle
| Phase | Action | Timeline |
|-------|--------|---------|
| Procurement | Order, configure, register Autopilot | 4-6 weeks before need |
| Deployment | Ship or stage, user receives | Day of onboarding |
| Active Use | Monitor, patch, support | 3-5 years |
| Refresh Assessment | Compare to refresh triggers | Year 3+ |
| Refresh | New device provisioned, old retired | Device age 4-5 years |
| Retirement | Wipe, asset disposal, ITAD | Within 30 days of replacement |

**Refresh Triggers:**
- Hardware failure rate increasing
- Device age > 4 years
- Can no longer run current OS
- Security compliance failures
- User productivity severely impacted

### Retirement & Data Destruction
```powershell
# Intune: Retire device (removes corporate data, unenrolls)
# Intune > Devices > Select device > Retire

# Intune: Wipe device (factory reset)
# Intune > Devices > Select device > Wipe

# SCCM: Wipe certification
# SCCM console > Assets > Devices > Right-click > Remote Device Actions > Wipe

# BitLocker: Ensure drive is encrypted before physical disposal
# Physical destruction: NSA/CSS EPL approved methods for classified
# NIST 800-88: Clear (software wipe) or Purge (cryptographic erase)
```

**ITAD Checklist:**
- [ ] Remove from Active Directory (AD Users & Computers)
- [ ] Remove from Azure AD / Intune
- [ ] Remove from SCCM/Autopilot inventory
- [ ] Remove DNS/DHCP reservations
- [ ] Update asset management system
- [ ] BitLocker wipe or physical destruction documentation
- [ ] Certificate of data destruction obtained from ITAD vendor
- [ ] Update software license inventory

---

## 9. Asset Management

### Key Data to Track
| Field | Description |
|-------|-------------|
| Asset Tag | Physical label ID |
| Serial Number | Manufacturer serial |
| Model | Make and model |
| Purchase Date | For warranty tracking |
| Warranty Expiry | Service contract end |
| Assigned User | Primary user |
| Location | Building/floor/room |
| OS Version | Current OS |
| Last Seen | Last inventory date |
| Compliance Status | Intune compliance state |

### Automated Inventory via Intune
```powershell
Connect-MgGraph -Scopes "DeviceManagementManagedDevices.Read.All"

$devices = Get-MgDeviceManagementManagedDevice -All -Property *

$inventory = $devices | Select `
    DeviceName,
    SerialNumber,
    Manufacturer,
    Model,
    OperatingSystem,
    OsVersion,
    UserDisplayName,
    UserPrincipalName,
    ComplianceState,
    ManagementState,
    LastSyncDateTime,
    @{N="StorageGB";E={[math]::Round($_.TotalStorageSpaceInBytes/1GB,0)}},
    @{N="FreeStorageGB";E={[math]::Round($_.FreeStorageSpaceInBytes/1GB,0)}}

$inventory | Export-Csv "endpoint_inventory_$(Get-Date -Format yyyyMMdd).csv" -NoTypeInformation
Write-Host "Exported $($inventory.Count) devices"
```

---

## 10. Troubleshooting Intune Enrollment

### Enrollment Failure Matrix
| Error Code | Message | Fix |
|-----------|---------|-----|
| 0x80180014 | Not authorized | Check enrollment restrictions, device limit |
| 0x80180026 | Device is already enrolled | Unenroll first via Settings |
| 0x80180022 | OS not supported | Check device compliance/version |
| 0x803C003C | Device type not supported | Policy restricts device type |
| 80090016 | TPM issue | Clear TPM in BIOS |

### Diagnostic Commands
```cmd
:: Collect MDM logs
MdmDiagnosticsTool.exe -out C:\MDMLogs

:: Check enrollment status
dsregcmd /status

:: Force MDM sync
deviceenroller.exe /o /c /s

:: Check Intune Management Extension
Get-WinEvent -LogName "Microsoft-Windows-DeviceManagement-Enterprise-Diagnostics-Provider/Admin" |
    Select -First 50
```

```powershell
# Intune Management Extension log
Get-Content "C:\ProgramData\Microsoft\IntuneManagementExtension\Logs\IntuneManagementExtension.log" |
    Select-String "error|fail|exception" -CaseSensitive:$false
```

---

*Last Updated: 2025 | IT Operations Documentation Library*
