# macOS Troubleshooting Guide

## Overview
Comprehensive troubleshooting reference for macOS environments in enterprise settings. Covers common issues, diagnostic tools, MDM integration, and resolution procedures.

---

## 1. macOS Diagnostic Tools

### Built-In Tools
| Tool | Location | Use Case |
|------|----------|----------|
| Console.app | /Applications/Utilities | System/app logs |
| Activity Monitor | /Applications/Utilities | CPU, memory, disk, network usage |
| Disk Utility | /Applications/Utilities | Disk repair, SMART status |
| System Information | Apple Menu > About This Mac | Hardware overview |
| Network Utility | Deprecated (use Terminal) | Network diagnostics |
| Terminal | /Applications/Utilities | CLI diagnostics |

### Key Terminal Commands
```bash
# System info
system_profiler SPHardwareDataType
sw_vers                           # macOS version
uname -a                          # Kernel info

# Process management
top -o cpu                        # CPU usage sorted
ps aux | grep <process>          # Find specific process
kill -9 <PID>                    # Force kill process

# Disk
diskutil list                    # List all disks
diskutil verifyVolume /          # Verify boot volume
df -h                            # Disk free space
du -sh /Users/*                  # User folder sizes

# Network
ifconfig                         # Interface info
networksetup -listallnetworkservices
networksetup -getinfo "Wi-Fi"
scutil --dns                     # DNS configuration
scutil --proxy                   # Proxy settings
ping -c 4 google.com
traceroute google.com
nslookup google.com
netstat -an | grep LISTEN        # Open ports

# Logs
log show --last 1h --predicate 'eventMessage contains "error"'
log stream --level debug         # Live log stream
syslog -k Sender kernel          # Kernel messages

# Memory
vm_stat                          # Virtual memory stats
sysctl hw.memsize                # Total RAM

# Startup
launchctl list                   # Running launch agents/daemons
```

---

## 2. Boot Issues

### Mac Won't Turn On
1. Check power connection / battery
2. Hold power button 10 seconds (force restart)
3. Reset SMC (System Management Controller)
4. Reset NVRAM/PRAM

### SMC Reset
**Intel Macs (non-removable battery):**
1. Shut down
2. Hold: Shift + Control + Option + Power for 10 seconds
3. Release all, press Power

**Apple Silicon (M1/M2/M3):**
- No SMC reset needed; restart handles it

### NVRAM/PRAM Reset (Intel only)
1. Shut down
2. Press Power, immediately hold: Option + Command + P + R
3. Hold 20 seconds or until startup sound plays twice

### Apple Silicon Recovery Mode
1. Shut down completely
2. Press and hold Power until "Loading startup options" appears
3. Select Options > Continue

### Safe Mode
- **Intel:** Hold Shift during boot
- **Apple Silicon:** Hold Shift at startup options screen, select volume, continue in Safe Mode

### Common Boot Error Codes
| Symbol | Meaning | Action |
|--------|---------|--------|
| ⊘ (circle-slash) | Incompatible startup disk | Boot recovery, reinstall macOS |
| 🔒 (lock) | FileVault locked | Enter password |
| ? (folder) | No bootable volume found | Recovery mode, disk repair |
| 🔧 (wrench) | Self-repair in progress | Wait |
| Progress bar stops | Kernel panic / bad driver | Safe mode, remove extensions |

---

## 3. Performance Issues

### High CPU Usage
```bash
# Find CPU hogs
top -o cpu -n 20
ps -Ao pid,pcpu,comm | sort -k2 -rn | head -20

# Spotlight reindexing (common culprit: mds/mdworker)
sudo mdutil -a -i off           # Disable indexing
sudo mdutil -a -i on            # Re-enable
sudo mdutil -E /                # Force reindex
```

**Common High-CPU Processes:**
- `mds` / `mdworker` — Spotlight indexing (temporary, let complete)
- `kernel_task` — Thermal management (check cooling/temps)
- `WindowServer` — Display compositor (restart display server or reboot)
- `cloudd` / `bird` — iCloud sync (check iCloud storage/connectivity)

### Memory Pressure
```bash
vm_stat | grep -E "Pages (free|active|inactive|wired)"
# High swap usage = insufficient RAM
sysctl vm.swapusage
```

**Diagnose with Activity Monitor:**
- Memory tab > Memory Pressure graph (green=good, yellow=caution, red=critical)
- Check for memory leaks (process growing over time)

### Slow Startup
```bash
# List login items
osascript -e 'tell application "System Events" to get the name of every login item'

# Check launch agents
ls ~/Library/LaunchAgents/
ls /Library/LaunchAgents/
ls /Library/LaunchDaemons/

# Disable a launch daemon
launchctl unload -w /Library/LaunchAgents/<plist>
```

---

## 4. Network Issues

### Wi-Fi Troubleshooting
```bash
# Check Wi-Fi status
networksetup -getairportpower en0
networksetup -setairportpower en0 on

# List available networks
/System/Library/PrivateFrameworks/Apple80211.framework/Versions/Current/Resources/airport -s

# Check current connection
/System/Library/PrivateFrameworks/Apple80211.framework/Versions/Current/Resources/airport -I

# Renew DHCP
ipconfig set en0 DHCP
sudo ipconfig set en0 DHCP

# Flush DNS cache
sudo dscacheutil -flushcache
sudo killall -HUP mDNSResponder

# Reset network settings
sudo rm /Library/Preferences/SystemConfiguration/NetworkInterfaces.plist
sudo rm /Library/Preferences/SystemConfiguration/preferences.plist
# Restart after
```

### VPN Issues
```bash
# Check VPN status
scutil --nc list                 # List VPN configurations
scutil --nc status "VPN Name"   # Check specific VPN

# Logs
log show --predicate 'subsystem == "com.apple.networkextension"' --last 30m
```

### Proxy Configuration
```bash
networksetup -getwebproxy Wi-Fi
networksetup -setwebproxy Wi-Fi <server> <port>
networksetup -setwebproxystate Wi-Fi off
```

---

## 5. User Account Issues

### Password Reset (Local Account)
**As Admin:**
```bash
# List users
dscl . list /Users | grep -v '^_'

# Reset password
sudo dscl . -passwd /Users/<username> <newpassword>

# Unlock account
sudo pwpolicy -u <username> -clearaccountpolicies
```

**Recovery Mode:**
1. Boot Recovery Mode
2. Utilities > Terminal
3. `resetpassword` (GUI tool)

### Account Locked Out (MDM/Policy)
```bash
# Check password policy
pwpolicy -u <username> -getaccountpolicies

# Check failed attempts
dscl . -read /Users/<username> accountPolicyData
```

### Keychain Issues
```bash
# Reset keychain (will lose saved passwords)
security delete-keychain ~/Library/Keychains/login.keychain-db
# Log out and log back in to recreate

# Lock keychain
security lock-keychain ~/Library/Keychains/login.keychain-db

# Find item in keychain
security find-generic-password -l "<name>"
```

### Profile/Preference Corruption
```bash
# Move prefs to test (as user)
mv ~/Library/Preferences/com.apple.<app>.plist ~/Desktop/
# Log out / log in to recreate

# Reset Dock
defaults delete com.apple.dock; killall Dock

# Reset Finder
defaults delete com.apple.finder; killall Finder
```

---

## 6. Application Issues

### App Won't Launch
1. Force quit (Cmd+Option+Escape)
2. Check Console.app for crash reports
3. Delete app preferences: `~/Library/Preferences/com.<vendor>.<app>.plist`
4. Delete app support: `~/Library/Application Support/<AppName>/`
5. Reinstall application

### Crash Reports
```bash
# View crash logs
ls ~/Library/Logs/DiagnosticReports/
ls /Library/Logs/DiagnosticReports/

# Recent crashes
log show --predicate 'eventMessage contains "crashed"' --last 1h
```

### Gatekeeper / Security Blocking App
```bash
# Check quarantine
xattr -l /Applications/<App>.app

# Remove quarantine flag (use carefully)
xattr -d com.apple.quarantine /Applications/<App>.app

# Allow app from unidentified developer (one-time)
# System Preferences > Security & Privacy > "Open Anyway"

# Check code signing
codesign -vv /Applications/<App>.app
spctl --assess -vv /Applications/<App>.app
```

---

## 7. Storage Issues

### Disk Full
```bash
# Find large files
find / -size +500M -type f 2>/dev/null
du -sh /Users/*/Downloads /Users/*/Desktop

# Clean system caches
sudo rm -rf /Library/Caches/*
rm -rf ~/Library/Caches/*

# Remove old iOS/iPadOS backups
ls ~/Library/Application\ Support/MobileSync/Backup/

# Check Time Machine local snapshots
tmutil listlocalsnapshots /
# Delete specific snapshot
tmutil deletelocalsnapshots <YYYY-MM-DD-HHMMSS>
```

### Disk Repair
```bash
# Verify disk
diskutil verifyDisk disk0

# Repair disk (from Recovery Mode for boot volume)
diskutil repairDisk disk0
diskutil repairVolume /

# First Aid in Disk Utility (GUI)
# Or from Recovery: Disk Utility > Select volume > First Aid
```

### FileVault Issues
```bash
# Check FileVault status
fdesetup status

# List FileVault users
sudo fdesetup list

# Add recovery key
sudo fdesetup changerecovery -personal

# Decrypt (disable FileVault)
sudo fdesetup disable
```

---

## 8. Printer Issues

### Add/Remove Printers
```bash
# List printers
lpstat -p -d

# Delete printer
lpadmin -x <printer-name>

# Reset print system
# System Preferences > Printers & Scanners > Right-click list > Reset printing system
```

### Clear Print Queue
```bash
cancel -a                        # Cancel all jobs
lprm -                           # Remove all jobs
sudo launchctl stop org.cups.cupsd
sudo launchctl start org.cups.cupsd
```

---

## 9. MDM (Mobile Device Management) Troubleshooting

### Jamf Pro Common Issues
```bash
# Check MDM enrollment
profiles status -type enrollment

# List installed profiles
profiles list -all

# Jamf check-in
sudo jamf recon                  # Update inventory
sudo jamf policy                 # Run all policies
sudo jamf manage                 # Re-enable management

# Jamf logs
cat /var/log/jamf.log
tail -f /var/log/jamf.log

# Remove MDM enrollment (if authorized)
sudo profiles remove -all
```

### MDM Profile Issues
```bash
# List configuration profiles
profiles show -all

# Remove specific profile
sudo profiles -R -p <ProfileID>

# Check for pending MDM commands
log show --predicate 'subsystem == "com.apple.mdmclient"' --last 1h
```

### Enrollment Troubleshooting
1. Verify device is in ABM/ASM (Apple Business/School Manager)
2. Check MDM server URL is accessible
3. Confirm correct enrollment profile installed
4. Verify system clock is accurate
5. Check network connectivity to MDM server

---

## 10. Software Update Issues

### macOS Update Fails
```bash
# Check available updates
softwareupdate -l

# Install specific update
sudo softwareupdate -i "<update name>"

# Install all updates
sudo softwareupdate -ia

# Clear update cache
sudo rm -rf /Library/Updates/*
sudo softwareupdate --clear-catalog

# Check update logs
log show --predicate 'subsystem == "com.apple.SoftwareUpdate"' --last 2h
```

### MDM-Pushed Update Not Installing
1. Check MDM deferral settings
2. Verify device is not in DND mode
3. Ensure sufficient disk space (15 GB+ free)
4. Check battery level (20%+ or plugged in)
5. Review Jamf/MDM logs for error codes

---

## 11. Directory Services / AD Binding

### Active Directory Binding
```bash
# Check AD binding status
dsconfigad -show

# Bind to Active Directory
sudo dsconfigad -add <domain.com> -username <admin> -password <pass> \
  -ou "OU=Macs,DC=domain,DC=com" -mobile enable -mobileconfirm disable

# Unbind from Active Directory
sudo dsconfigad -remove -username <admin> -password <pass>

# Force AD sync
dscacheutil -flushcache
sudo killall -HUP DirectoryService

# Check AD user info
id <username>
dscl /Active\ Directory/DOMAIN/All\ Domains -read /Users/<username>
```

### Directory Service Issues
```bash
# Restart Directory Services
sudo launchctl stop com.apple.opendirectoryd
sudo launchctl start com.apple.opendirectoryd

# Check directory service status
odutil show statistics

# Test authentication
dscl /Active\ Directory/<DOMAIN>/All\ Domains -authonly <username> <password>
```

---

## 12. Security & Privacy

### Full Disk Access Issues (Applications)
- System Preferences > Security & Privacy > Privacy > Full Disk Access
- Add applications that need disk access (MDM tools, backup agents)
- Common apps needing FDA: Jamf, backup software, AV solutions

### Transparency, Consent, and Control (TCC)
```bash
# Reset TCC database (requires SIP disabled - not recommended in prod)
# Better: grant via MDM privacy preferences payload

# Check TCC approvals for app
sqlite3 "/Users/<user>/Library/Application Support/com.apple.TCC/TCC.db" \
  "SELECT * FROM access WHERE client='<bundle-id>'"
```

### System Integrity Protection (SIP)
```bash
# Check SIP status
csrutil status

# Never disable SIP on managed/production machines
# If disabled for testing, re-enable:
# Boot Recovery Mode > Utilities > Terminal > csrutil enable
```

---

## 13. Common Error Messages

| Error | Cause | Resolution |
|-------|-------|-----------|
| "App is damaged and can't be opened" | Quarantine/cert issue | Remove quarantine flag or re-download |
| "You don't have permission" | File permissions | `chmod`/`chown` or ACL issue |
| "No network connection" when connected | DNS/proxy issue | Flush DNS, check proxy settings |
| "Can't connect to App Store" | Date/time or cert issue | Sync NTP, check network |
| "This copy of macOS is too old" | Version check failure | Update macOS |
| Spinning beach ball | App hung/resource starvation | Force quit, check memory |
| Kernel panic | Hardware/driver/memory issue | Check Console, run Diagnostics |

---

## 14. Apple Diagnostics

### Run Hardware Diagnostics
1. Shut down Mac
2. Power on, immediately hold **D**
3. For internet diagnostics: hold **Option+D**
4. Follow on-screen instructions

### Reference Codes
| Code Prefix | Component |
|-------------|-----------|
| ADP | Bluetooth |
| NDT | Wireless |
| NDD | Wi-Fi |
| PPT | Power |
| HDD | Storage |
| MLB | Logic Board |
| MEM | Memory (RAM) |
| GPU | Graphics |

---

## 15. Escalation Checklist

Before escalating macOS issues:
- [ ] macOS version documented
- [ ] Hardware model and serial number captured
- [ ] Recent changes (updates, new software) noted
- [ ] Console logs exported from time of issue
- [ ] Reproduced in new user account (rules out profile corruption)
- [ ] Tested in Safe Mode
- [ ] Apple Diagnostics run (hardware issues)
- [ ] MDM enrollment status confirmed
- [ ] Steps taken documented

---

*Last Updated: 2025 | IT Operations Documentation Library*
