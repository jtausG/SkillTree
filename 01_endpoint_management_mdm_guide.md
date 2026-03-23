# Endpoint Management & MDM Guide

## Table of Contents
1. [Endpoint Strategy Overview](#strategy)
2. [Microsoft Intune / Endpoint Manager](#intune)
3. [Windows Autopilot](#autopilot)
4. [Configuration Profiles & Policies](#policies)
5. [Application Management](#apps)
6. [Compliance & Conditional Access](#compliance)
7. [Patch Management](#patching)
8. [Endpoint Detection & Response](#edr)
9. [Mobile Device Management](#mdm)
10. [Mac Management (Jamf / Intune)](#mac)
11. [Endpoint Inventory & Reporting](#inventory)
12. [Troubleshooting Endpoints](#troubleshooting)

---

## 1. Endpoint Strategy Overview <a name="strategy"></a>

### Modern Endpoint Management Principles
- **Cloud-first**: Manage all devices through cloud-based MDM, minimize on-prem dependencies
- **Zero Trust**: Never trust, always verify — no implicit trust based on network location
- **Least Privilege**: Users and devices get only what they need
- **Automation**: Provisioning, patching, and remediation automated end-to-end
- **Visibility**: Full inventory and telemetry on every managed device

### Management Tiers
| Device Type | Primary Tool | Enrollment | Join Type |
|-------------|-------------|------------|-----------|
| Windows Corporate | Intune + SCCM co-mgmt | Autopilot / manual | Azure AD Joined |
| Windows BYOD | Intune MAM/MDM | User-initiated | Azure AD Registered |
| macOS Corporate | Jamf Pro or Intune | ADE (automated) | Azure AD Registered |
| iOS/iPadOS | Intune | ADE or user-initiated | Azure AD Registered |
| Android Corporate | Intune | AE (Android Enterprise) | Work Profile or COPE |
| Linux | Intune (preview) / Ansible | Manual | — |

### Decision Framework: MDM vs Co-Management vs Hybrid AD
```
New Device?
    └─ Yes → Cloud-only (Azure AD Join + Intune)
    └─ No (existing) →
        └─ On-prem AD required? →
            └─ Yes → Hybrid AD Join + Intune co-management
            └─ No → Migrate to Azure AD Join over time
```

---

## 2. Microsoft Intune / Endpoint Manager <a name="intune"></a>

### Admin Portal Navigation
- **URL**: https://intune.microsoft.com
- **Devices** → Enrolled devices, enrollment restrictions, compliance policies
- **Apps** → App deployments, protection policies, app configuration
- **Endpoint Security** → Antivirus, disk encryption, firewall, EDR
- **Reports** → Device compliance, app inventory, update status

### Enrollment Methods

#### Windows Enrollment
```
Method 1: Windows Autopilot (preferred)
- Pre-register device hash with Autopilot
- User powers on → Signs in with corporate credentials
- Device auto-configures, no IT hands-on required

Method 2: Bulk Enrollment (provisioning package)
- Create package with Windows Configuration Designer
- Apply via USB or network share
- Good for shared/kiosk devices

Method 3: Manual MDM Enrollment
- Settings → Accounts → Access work or school → Connect
- User enters UPN → Device enrolls

Method 4: Group Policy MDM enrollment
- GPO: Computer Configuration → Administrative Templates → Windows Components → MDM
- Auto-enrolls domain-joined devices into Intune
```

#### Enrollment Restrictions
```json
{
  "deviceTypeRestrictions": {
    "allowPersonallyOwnedWindows": false,
    "allowPersonallyOwnedIOS": true,
    "allowPersonallyOwnedAndroid": true,
    "allowPersonallyOwnedMac": false
  },
  "deviceLimitRestrictions": {
    "maxDevicesPerUser": 5
  }
}
```

### Key Intune Concepts
| Concept | Description |
|---------|-------------|
| Device Group | Azure AD group used for targeting policies/apps |
| Configuration Profile | Settings applied to devices (Wi-Fi, VPN, restrictions) |
| Compliance Policy | Rules defining "compliant" status |
| App Protection Policy | Data protection rules (MAM) without full MDM enrollment |
| Remediation Scripts | PowerShell scripts to detect and fix issues |
| Filters | Dynamic targeting based on device properties |

---

## 3. Windows Autopilot <a name="autopilot"></a>

### Autopilot Prerequisites
- Azure AD Premium P1 (minimum)
- Intune license for all users
- Device hash registered in Autopilot
- Autopilot deployment profile created and assigned

### Registering Device Hashes
```powershell
# Method 1: Run on device to collect hardware hash
Install-Script -Name Get-WindowsAutopilotInfo
Get-WindowsAutopilotInfo -OutputFile C:\Temp\AutopilotHWID.csv

# Method 2: Collect during OOBE (Out of Box Experience)
# Press Ctrl+Shift+F3 during first setup to enter Audit Mode
# Run: Get-WindowsAutopilotInfo -Online (uploads directly to Intune)

# Method 3: Bulk import from CSV
# Intune → Devices → Enrollment → Devices → Import
# CSV columns: Device Serial Number, Windows Product ID, Hardware Hash

# Method 4: Via Microsoft CSP/distributor
# Have vendor register hashes at purchase
```

### Autopilot Deployment Profiles

#### User-Driven (Standard)
```
Profile Settings:
- Deployment mode: User-driven
- Join to Azure AD as: Azure AD joined
- EULA: Hide
- Privacy settings: Hide
- Hide change account options: Show
- User account type: Standard User (enforce with Conditional Access)
- Apply device name template: IT-%RAND:5% (e.g., IT-A3F2B)
```

#### Self-Deploying (Kiosk/Shared)
```
Profile Settings:
- Deployment mode: Self-deploying
- Join to Azure AD as: Azure AD joined
- Enrollment Status Page: Enabled, block device until provisioned
- No user interaction required
- Ideal for: shared workstations, conference room PCs, kiosks
```

### Enrollment Status Page (ESP) Configuration
```
Recommended ESP Settings:
- Show app and profile installation progress: Yes
- Show error when installation takes longer than: 60 minutes
- Allow users to collect logs: Yes (for troubleshooting)
- Block device use if required apps fail: Yes
- Allow users to reset device if error: No (require IT)

Track these apps in ESP:
- Company Portal
- Microsoft 365 Apps
- VPN client
- Security baseline config
```

### Autopilot Troubleshooting
```powershell
# Check Autopilot registration status
Get-AutopilotDevice -serial "YOUR_SERIAL" | Select-Object *

# View ESP logs during enrollment
# C:\Windows\ServiceProfiles\LocalService\AppData\Local\Temp\SentinelAgent\
# Or: Event Viewer → Applications and Services → Microsoft → Windows → Provisioning-Diagnostics-Provider

# Common errors and fixes:
# 0x800705b4 - Timeout: Increase ESP timeout or reduce required apps
# 0x80180018 - Enrollment blocked: Check enrollment restrictions
# 0x801c0003 - Not licensed: Verify user has Intune license
# 0xcaa2000c - Azure AD error: Verify DNS, connectivity to Azure
```

---

## 4. Configuration Profiles & Policies <a name="policies"></a>

### Windows Security Baseline
```
Microsoft recommends applying the Microsoft Security Baseline, which includes:

Identity:
- Block Microsoft accounts for local sign-in
- Require Ctrl+Alt+Del
- Hide usernames on sign-in screen

Browser:
- Force SmartScreen
- Block password manager (use enterprise solution)
- Disable autofill for forms

Windows Defender:
- Enable real-time protection
- Enable PUA (potentially unwanted app) protection
- Enable tamper protection

BitLocker:
- Require BitLocker on OS drive
- TPM + PIN startup
- Save recovery key to Azure AD

Account Lockout:
- Lockout threshold: 10 failed attempts
- Observation window: 10 minutes
- Lockout duration: 10 minutes
```

### Custom Configuration Profiles

#### Wi-Fi Profile (Enterprise)
```xml
<!-- Intune Wi-Fi Profile for WPA2-Enterprise -->
<WLANProfile>
  <name>CorpWiFi</name>
  <SSIDConfig>
    <SSID><name>CORP-WIFI</name></SSID>
    <nonBroadcast>false</nonBroadcast>
  </SSIDConfig>
  <connectionType>ESS</connectionType>
  <MSM>
    <security>
      <authEncryption>
        <authentication>WPA2</authentication>
        <encryption>AES</encryption>
        <useOneX>true</useOneX>
      </authEncryption>
      <OneX>
        <EAPConfig>
          <!-- PEAP-MSCHAPv2 or EAP-TLS for certificate auth -->
        </EAPConfig>
      </OneX>
    </security>
  </MSM>
</WLANProfile>
```

#### VPN Profile (Always On)
```
Profile Type: VPN
VPN Connection Name: Corporate VPN
Connection Type: IKEv2 (or SSTP)
Server: vpn.contoso.com

Always On VPN:
  - Device tunnel: Connects before user logon
  - User tunnel: Connects at user logon
  - Traffic rules: Include split tunneling rules
  - Authentication: Machine/user certificates from Intune SCEP

Split Tunneling:
  Include routes: 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16
  Exclude: All internet traffic (reduce bandwidth consumption)
```

#### PowerShell Remediation Scripts
```powershell
# Detection script - runs hourly
# Returns exit code 0 (compliant) or 1 (not compliant)

$screenTimeout = (powercfg /query SCHEME_CURRENT SUB_VIDEO VIDEOIDLE 2>$null)
$acTimeout = ($screenTimeout | Select-String "Current AC Power Setting Index: (.+)").Matches.Groups[1].Value
$acTimeoutSec = [Convert]::ToInt32($acTimeout, 16)

if ($acTimeoutSec -gt 900 -or $acTimeoutSec -eq 0) {  # 15 min = 900 sec
    Write-Output "Screen timeout non-compliant: $acTimeoutSec seconds"
    exit 1
}
exit 0

---

# Remediation script - runs if detection returns exit 1
powercfg /change monitor-timeout-ac 15   # 15 minutes on AC
powercfg /change monitor-timeout-dc 5    # 5 minutes on battery
Write-Output "Screen timeout remediated"
exit 0
```

---

## 5. Application Management <a name="apps"></a>

### App Deployment Types
| Type | Use Case | Packaging Format |
|------|----------|-----------------|
| Win32 App | Complex MSI/EXE with dependencies | .intunewin (wrapped) |
| Microsoft Store | UWP apps | Direct from store |
| Line of Business | Simple MSI/MSIX | Upload .msi/.msix |
| Web App | Browser shortcuts | URL + icon |
| Microsoft 365 | M365 suite | Built-in deployment wizard |

### Win32 App Packaging
```powershell
# Step 1: Prepare source folder
# C:\AppSource\
# ├── install.ps1 (or setup.exe)
# └── [any other required files]

# Step 2: Create .intunewin file
# Download: IntuneWinAppUtil.exe from Microsoft
.\IntuneWinAppUtil.exe `
    -c "C:\AppSource" `
    -s "install.ps1" `
    -o "C:\IntunePackages"

# Step 3: Upload to Intune
# Apps → Add → App type: Windows app (Win32)

# Install command example
powershell.exe -ExecutionPolicy Bypass -File install.ps1

# Uninstall command example
powershell.exe -ExecutionPolicy Bypass -File uninstall.ps1

# Detection rules - use Registry or File:
# Registry: HKLM\SOFTWARE\Contoso\AppName, Value "Version" = "2.1.0"
# File: C:\Program Files\App\app.exe (version 2.1.0+)

# Return codes:
# 0 = Success
# 1707 = Success
# 3010 = Reboot required
# 1641 = Reboot initiated
```

### App Protection Policies (MAM)
```
MAM Without Enrollment (BYOD):
- Protect corporate data in M365 apps without managing the whole device
- Settings:
  - Require PIN to open corporate apps
  - Block copy/paste to personal apps
  - Block screenshots (Android)
  - Require managed browser for web links
  - Wipe corporate data on unenrollment

MAM With Enrollment (MDM):
- Full device management + data protection
- Recommended for corporate-owned devices
```

### Required vs Available Apps
```
Required (mandatory):
- Pushed automatically, cannot uninstall
- Use for: Security tools, VPN client, Company Portal, DLP agent
- Target: Device groups (applies regardless of user)

Available (optional):
- Shows in Company Portal for user to install
- Use for: Adobe, specialized tools, optional productivity apps
- Target: User groups

Uninstall:
- Removes app if user leaves targeted group
- Use for: Offboarding scenarios
```

---

## 6. Compliance & Conditional Access <a name="compliance"></a>

### Compliance Policy Settings
```
Windows Compliance Policy:
┌─ Device Health
│   ├── Require BitLocker: Yes
│   ├── Require Secure Boot: Yes
│   └── Require code integrity: Yes
├─ Device Properties
│   ├── Minimum OS version: 10.0.19041 (20H1)
│   └── Maximum OS version: (leave blank unless needed)
├─ Configuration Manager Compliance
│   └── Require compliance from SCCM (if co-managed): Yes
├─ System Security
│   ├── Firewall: Required
│   ├── Antivirus: Required
│   ├── Antispyware: Required
│   └── Microsoft Defender Antimalware: Required
└─ Microsoft Defender for Endpoint
    └── Require device to be at or under machine risk score: Medium
```

### Conditional Access Policies

#### Require MFA for All Apps
```
Name: Require MFA - All Cloud Apps
Assignments:
  Users: All users (exclude break-glass accounts)
  Cloud apps: All cloud apps
  Conditions: Any location

Access controls:
  Grant: Require MFA
  Require all selected controls: Yes
```

#### Block Non-Compliant Devices
```
Name: Block Non-Compliant Devices
Assignments:
  Users: All users
  Cloud apps: All cloud apps
  Conditions:
    Device platforms: Windows, macOS, iOS, Android
    Filter for devices: device.isCompliant -ne True

Access controls:
  Block access
```

#### Require Compliant Device for Sensitive Apps
```
Name: Require Compliant Device - Finance Apps
Assignments:
  Users: Finance-Users group
  Cloud apps: Dynamics 365, SAP, Finance portal
  
Access controls:
  Grant: Require device to be marked as compliant
```

### Grace Period & Compliance Notifications
```
Compliance notification timeline:
Day 0: Device becomes non-compliant → Marked non-compliant
Day 1: Email notification to user
Day 3: Push notification + email
Day 7: Final warning email with escalation
Day 14: Block access via Conditional Access
Day 30: Retire/wipe device (optional)

Email template (non-compliance):
Subject: Action Required: Your device is not compliant
Body: Your device [DEVICE_NAME] does not meet IT security requirements.
      Please open the Company Portal app and follow the remediation steps.
      Access to corporate resources will be blocked in [DAYS] days.
```

---

## 7. Patch Management <a name="patching"></a>

### Windows Update for Business (WUfB) Rings
```
Ring 0 - Pilot (IT staff): 0 days deferral
Ring 1 - Early Adopters: 7 days deferral
Ring 2 - General: 14-21 days deferral
Ring 3 - Critical/Executive: 28 days deferral
Ring 4 - Servers: Manual approval, 45+ days

Feature Updates (new Windows versions):
- Defer 90-180 days until validated
- Test in Ring 0 and 1 first
- Block using safeguard holds if known issues

Quality Updates (monthly patches):
- Defer 7 days for Ring 0, scale up
- Deadline: Force install after 7-14 days
- Active hours: 8AM-6PM (prevent restart during work)
```

### Patch Compliance Monitoring
```powershell
# Pull Windows Update compliance from Intune (PowerShell + Graph API)
$token = Get-GraphAccessToken -TenantId $tenantId -ClientId $clientId -ClientSecret $secret

$headers = @{ Authorization = "Bearer $token" }

# Get all devices missing patches
$url = "https://graph.microsoft.com/beta/deviceManagement/managedDevices?`$filter=operatingSystem eq 'Windows'&`$select=deviceName,osVersion,lastSyncDateTime,complianceState,userDisplayName"
$devices = Invoke-RestMethod -Uri $url -Headers $headers

# Flag devices not synced in 7 days
$stale = $devices.value | Where-Object {
    [datetime]$_.lastSyncDateTime -lt (Get-Date).AddDays(-7)
}
```

### WSUS / SCCM Patching (On-Prem)
```
WSUS Approval Workflow:
1. Microsoft releases patches (Patch Tuesday + out-of-band)
2. WSUS synchronizes (daily sync recommended)
3. IT reviews and approves to pilot group
4. Monitor 48-72 hours for issues
5. Approve to general collection
6. Run compliance reports

SCCM Deployment Collections:
- Collection: Pilot-Workstations (IT staff machines)
- Collection: Wave1-Workstations (20% sample)
- Collection: General-Workstations (remaining)
- Collection: Servers-Dev → Servers-QA → Servers-Prod
- Collection: Excluded-Devices (critical exceptions)

Deployment schedule:
- Pilot: Day 1 (Patch Tuesday + 1)
- Wave 1: Day 7
- General: Day 14
- Servers: Day 21 (maintenance window required)
```

---

## 8. Endpoint Detection & Response <a name="edr"></a>

### Microsoft Defender for Endpoint (MDE)

#### Onboarding
```powershell
# Check MDE onboarding status
Get-MpComputerStatus | Select-Object AMRunningMode, OnboardingState

# Onboarding states:
# 0 = Not onboarded
# 1 = Onboarded

# Via Intune: Endpoint Security → Endpoint Detection and Response
# → Onboard via MDM policy (preferred)

# Manual onboarding:
# Download onboarding package from security.microsoft.com
# Run: WindowsDefenderATPOnboardingScript.cmd
```

#### Attack Surface Reduction (ASR) Rules
```powershell
# ASR rules deployment via Intune or PowerShell
# Modes: Disabled (0), Block (1), Audit (2), Warn (3)

$ASRRules = @{
    # Block Office apps from creating child processes
    "D4F940AB-401B-4EFC-AADC-AD5F3C50688A" = "Block"
    # Block credential stealing from LSASS
    "9E6C4E1F-7D60-472F-BA1A-A39EF669E4B0" = "Block"
    # Block process creations from PSExec/WMI
    "D1E49AAC-8F56-4280-B9BA-993A6D77406C" = "Block"
    # Block untrusted/unsigned processes from USB
    "B2B3F03D-6A65-4F7B-A9C7-1C7EF74A9BA4" = "Block"
    # Block Office macros from making Win32 API calls
    "92E97FA1-2EDF-4476-BDD6-9DD0B4DDDC7B" = "Block"
}

# Start in Audit mode, review 30 days, then switch to Block
```

#### Threat & Vulnerability Management
```
TVM Dashboard: security.microsoft.com → Vulnerability management

Key metrics to track:
- Exposure Score (target: < 30)
- Microsoft Secure Score for Devices
- Top vulnerable software
- Top recommendations by exposure impact

Weekly TVM review process:
1. Review critical/high CVEs (CVSS 7.0+)
2. Prioritize by exposure impact
3. Assign to patching team
4. Track remediation in ServiceNow
5. Verify remediation in TVM
```

---

## 9. Mobile Device Management <a name="mdm"></a>

### iOS/iPadOS Management

#### Enrollment via ADE (Apple Device Enrollment)
```
Prerequisites:
1. Apple Business Manager (ABM) or Apple School Manager (ASM) account
2. MDM server token from ABM synced with Intune
3. Devices purchased from Apple or Apple Authorized Reseller with ABM enrollment

Setup in Intune:
1. Devices → iOS/iPadOS → Enrollment → Enrollment program tokens
2. Upload ABM public key to ABM → Create MDM server
3. Download MDM server token → Upload to Intune
4. Sync devices from ABM to Intune
5. Assign enrollment profile to device group

Enrollment Profile settings:
- Department name: IT (shown during setup)
- Supervised: Yes (required for Kiosk, MDM removal block)
- User affinity: Enroll with user affinity (for personal-use devices)
               : Enroll without user affinity (shared/kiosk)
- Allow pairing: No (prevent local sync without IT approval)
- Locked enrollment: Yes (prevent MDM removal)
```

#### iOS Configuration Profile Key Settings
```
Restrictions:
- Require Face ID/Touch ID: Yes
- Allow App Store: No (for managed devices)
- Allow iCloud backup: No (corporate data stays on premises)
- Allow Siri: Yes (or No for high-security environments)
- Force encrypted backups: Yes

Per-App VPN:
- Associates specific apps with VPN connection
- Only those app's traffic routes through VPN

Email Profile:
- Exchange server: mail.contoso.com
- Account name: Corporate Email
- Authentication: Modern Auth / OAuth
- S/MIME: Enabled for legal/executive users
```

### Android Enterprise
```
Work Profile (BYOD):
- Creates separate work container
- Corporate apps in work profile, personal apps separate
- IT can wipe only work profile on offboarding
- MDM cannot see personal apps

Fully Managed (COBO - Corporate Owned):
- Complete IT control
- No personal use (or allowed as secondary container)
- Best for dedicated devices

Enrollment via QR Code or Zero-Touch:
1. Factory reset device
2. Scan IT-generated QR code during setup
3. Device auto-enrolls with policy
```

---

## 10. Mac Management (Jamf / Intune) <a name="mac"></a>

### Jamf Pro Essentials
```
Key Jamf Concepts:
- Smart Groups: Dynamic groups based on device criteria
- Policies: Software deployment, scripts, settings
- Configuration Profiles: MDM payloads (same as Intune profiles)
- Pre-Stage Enrollments: Zero-touch for ADE devices
- Extension Attributes: Custom inventory data

Common Jamf Policies:
- Software deployment: Deploy .pkg files
- Printer installation: Add network printers
- Script execution: Run shell scripts at enrollment/event
- Self Service: Employee-facing app catalog

Jamf Smart Group examples:
- "macOS < 14.x": computer.osVersion.startsWith("13.") OR startsWith("12.")
- "Encrypted Drives": fileVaultStatus != "FileVault Enabled"
- "Missing Agents": applicationTitle != "CrowdStrike Falcon.app"
```

### macOS via Microsoft Intune
```
Enrollment methods:
1. ADE (automated, preferred for corporate)
2. Device enrollment (user-initiated)
3. User-approved MDM enrollment

Key configuration profiles for macOS:
- Password policy (8+ chars, complexity)
- FileVault encryption (escrowing key to Intune)
- Gatekeeper (allow only App Store + identified developers)
- Firewall (enabled, stealth mode)
- Screen saver password (15 min)
- Certificate deployment (corp CA, Wi-Fi cert)

Shell scripts via Intune:
- Run as: Root (for system changes) or logged-in user
- Run script as managed pkg: No (run directly)
- Frequency: Once, every login, or every 15 min (check interval)

Example: Install Homebrew without popup
#!/bin/bash
NONINTERACTIVE=1 /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

---

## 11. Endpoint Inventory & Reporting <a name="inventory"></a>

### Intune Inventory Reports
```powershell
# Export all Intune-managed device inventory via Graph API
function Get-IntuneDeviceInventory {
    param([string]$AccessToken)

    $headers = @{ Authorization = "Bearer $AccessToken" }
    $select = "deviceName,serialNumber,manufacturer,model,operatingSystem,osVersion,enrolledDateTime,lastSyncDateTime,complianceState,managementState,userDisplayName,userPrincipalName,azureADDeviceId,id"

    $url = "https://graph.microsoft.com/v1.0/deviceManagement/managedDevices?`$select=$select"
    $all = @()

    do {
        $response = Invoke-RestMethod -Uri $url -Headers $headers
        $all += $response.value
        $url = $response.'@odata.nextLink'
    } while ($url)

    return $all
}

# Generate asset report
$devices = Get-IntuneDeviceInventory -AccessToken $token
$devices | Select-Object deviceName, serialNumber, manufacturer, model,
    operatingSystem, osVersion, lastSyncDateTime, complianceState, userDisplayName |
    Export-Csv "C:\Reports\DeviceInventory_$(Get-Date -Format yyyyMMdd).csv" -NoTypeInformation
```

### Hardware Asset Tracking
```
Asset Database fields (minimum):
- Asset Tag (IT-assigned label)
- Serial Number
- Model / Manufacturer
- Purchase Date
- Warranty Expiry
- Assigned User
- Location (site/floor/desk)
- OS Version
- Encryption Status
- Last Seen Date
- Lifecycle Status (Active/Spare/EOL/Disposed)

Refresh cycle policy:
- Laptops: 4 years
- Desktops: 5 years
- Servers: 5-7 years (based on contract)
- Mobile devices: 3 years

Disposition process:
1. IT wipes device (factory reset + wipe report)
2. Remove from MDM
3. Update asset database (Disposed)
4. Transfer to approved recycler (data destruction certificate)
```

---

## 12. Troubleshooting Endpoints <a name="troubleshooting"></a>

### Intune Sync Issues
```powershell
# Force Intune sync from device
# Option 1: Company Portal → Sync
# Option 2: Settings → Accounts → Access work or school → Info → Sync
# Option 3: PowerShell (as admin)
$EnrollmentPath = "HKLM:\SOFTWARE\Microsoft\Enrollments"
Get-ChildItem $EnrollmentPath | ForEach-Object {
    $enrollmentId = $_.PSChildName
    $obj = [wmiclass]"\root\cimv2\mdm\dmmap:MDM_EnterpriseModernAppManagement_AppManagement01"
    $obj.UpdateScanMethod() 2>$null
}

# Check MDM enrollment status
dsregcmd /status

# Look for:
# AzureAdJoined: YES
# DomainJoined: NO (or YES for hybrid)
# MDMUrl: https://enrollment.manage.microsoft.com/...
# IsDeviceProtectedByPolicy: YES

# View Intune logs
# Event Viewer → Applications and Services → Microsoft → Windows → DeviceManagement-Enterprise-Diagnostics-Provider
# Or: C:\Windows\Temp\MDMDiagnostics\

# Collect MDM diagnostic report
MdmDiagnosticsTool.exe -area DeviceEnrollment;DeviceProvisioning;TPM -cab C:\Temp\mdmdiag.cab
```

### Common Enrollment Errors
| Error Code | Meaning | Fix |
|------------|---------|-----|
| 0x80180018 | MDM enrollment blocked | Check enrollment restrictions in Intune |
| 0x80180014 | Invalid enrollment limit | Increase per-user device limit |
| 0x801c0003 | Not authorized | Check user license, AAD group membership |
| 0xcaa2000c | Network error | Verify connectivity to login.microsoftonline.com |
| 0x80070774 | Certificate error | Check SCEP/PKCS profile, CA connectivity |
| 0x8007064c | Already enrolled | Remove stale enrollment, re-enroll |

### Compliance Policy Troubleshooting
```
User reports: "I can't access email — device not compliant"

Diagnostic steps:
1. Open Company Portal → Check compliance status + reason
2. Intune Portal → Devices → [Device] → Device compliance → per-policy status
3. Review which specific setting failed
4. Check if setting was intentional (BitLocker off = IT exception needed)
5. If false positive: Re-sync device, wait 15 min
6. If persistent: Check if compliance policy is targeting correct group
7. For BitLocker: Run "manage-bde -status" to verify encryption state
8. For Defender: Run "Get-MpComputerStatus" to verify AV status

Common false positives:
- Firewall: 3rd party firewall not recognized → Use Defender Firewall or configure exclusion
- Antivirus: 3rd party AV → Use Intune exclusion setting or switch to Defender
- OS version: WUfB update pending reboot → Remind user to restart
```

### Remote Actions in Intune
```
Available remote actions (all logged with requester):
- Sync: Force policy/app check-in
- Restart: Remote reboot
- Remote lock: Lock device immediately
- Reset password: (mobile only)
- Wipe: Full factory reset (use carefully - irreversible)
- Retire: Remove corporate data, unenroll (BYOD-safe)
- Fresh Start: Reinstall Windows, keep user data option
- Collect diagnostics: Pull logs to Intune
- Run remediation: Trigger specific remediation script

Wipe vs Retire:
- Wipe: Removes ALL data. Use for lost/stolen corporate devices.
- Retire: Removes only corporate data/apps. Use for BYOD or voluntary offboarding.
```

---

*Last Updated: 2025 | Endpoint Management Guide v2.0*
