# Mobile Device Troubleshooting Guide (iOS & Android)

## Overview
Enterprise mobile device troubleshooting for iOS (iPhone/iPad) and Android devices managed via MDM. Covers enrollment, email, VPN, app issues, and hardware problems.

---

## 1. iOS Troubleshooting

### Device Won't Turn On
1. Charge device for 30 minutes
2. Force restart:
   - **iPhone 8+/iPad (no Home button):** Volume Up > Volume Down > Hold Side button
   - **iPhone 7:** Hold Volume Down + Side button 10s
   - **iPhone 6s/SE (1st gen):** Hold Home + Side button 10s

### Recovery Mode (iOS)
1. Force restart sequence but **keep holding Side button** after Apple logo
2. Connect to Mac/PC with Finder/iTunes
3. Choose Restore or Update

### DFU Mode (deepest restore)
1. Connect to computer
2. iPhone 8+: Volume Up > Volume Down > Hold Side 10s > release Side, keep holding Volume Down 5s
3. Screen stays black — Finder/iTunes detects DFU device

---

### iOS Network Issues

**Wi-Fi:**
```
Settings > Wi-Fi > (i) next to network > Forget Network > Reconnect
Settings > General > Transfer or Reset iPhone > Reset > Reset Network Settings
```

**Cellular:**
- Check carrier settings update: Settings > General > About (prompt appears if available)
- Toggle Airplane Mode on/off
- Remove/reinsert SIM card
- Check for carrier outage

**VPN Not Connecting:**
1. Check VPN profile still installed: Settings > General > VPN & Device Management
2. Verify credentials not expired
3. Test on Wi-Fi vs cellular
4. Re-enroll VPN profile from MDM

---

### iOS Email (Exchange/O365)

**Mail Not Syncing:**
1. Settings > Mail > Accounts > <Account> > Toggle Mail off/on
2. Delete and re-add account
3. Check Exchange Autodiscover: `https://autodiscover.<domain.com>/autodiscover/autodiscover.xml`
4. Verify Modern Auth / MFA not blocking

**ActiveSync Blocked:**
- Check Conditional Access policy in Azure AD
- Device may need to be compliant (enrolled in Intune/MDM)
- Check device compliance status in Intune portal

---

### MDM Enrollment (iOS / Intune / Jamf)

**Enrollment Steps (Company Portal):**
1. Install Company Portal from App Store
2. Sign in with corporate account
3. Follow enrollment wizard
4. Install management profile when prompted
5. Trust management certificate

**Enrollment Fails:**
| Error | Cause | Fix |
|-------|-------|-----|
| "Device limit reached" | Too many enrolled devices | Admin removes old device |
| "Not authorized" | License not assigned | Assign Intune license in M365 admin |
| "Profile installation failed" | Network/cert issue | Connect to corporate Wi-Fi, retry |
| "Management profile not trusted" | Trust not completed | Settings > General > VPN & Device Management > Trust |

**Check Enrollment Status:**
```
Settings > General > VPN & Device Management
```

**Force MDM Sync:**
- Intune Company Portal app > Devices > Select device > Check Status / Sync
- Jamf Now: Open app > Sync

---

### iOS App Issues

**App Won't Install:**
1. Check available storage (Settings > General > iPhone Storage)
2. Sign out/in Apple ID (Settings > <name> > Sign Out)
3. Check MDM app assignment
4. Reset App Store: Press and hold App Store icon > Remove from Home Screen (don't delete), re-add

**App Crashes:**
1. Force close: Swipe up from bottom, swipe app away
2. Offload and reinstall: Settings > General > iPhone Storage > <App> > Offload App
3. Check iOS version compatibility
4. Review crash logs: Settings > Privacy & Security > Analytics & Improvements > Analytics Data

**Volume Purchase Program (VPP) Apps:**
- Apps disappear if VPP license revoked
- Check MDM console for license availability
- Re-push app from MDM

---

## 2. Android Troubleshooting

### Device Won't Boot
- Hold Power + Volume Down for 10s (most devices)
- Varies by manufacturer — check device-specific key combo
- Boot into Recovery: Power + Volume Up (most devices)

### Android Recovery Mode
- Wipe cache partition (fixes many boot/performance issues without data loss)
- Factory reset (last resort)

---

### Android Network Issues

**Wi-Fi:**
```
Settings > Network & Internet > Wi-Fi > Long press network > Forget
Settings > System > Reset options > Reset Wi-Fi, mobile & Bluetooth
```

**Check Proxy Settings:**
```
Settings > Wi-Fi > Long press network > Modify > Advanced options > Proxy
```

**DNS Issues:**
```
Settings > Network & Internet > Private DNS
Set to: dns.google or your corporate DNS
```

---

### Android MDM Enrollment (Intune / Android Enterprise)

**Work Profile Enrollment:**
1. Install Intune Company Portal
2. Sign in with corporate account
3. Set up work profile when prompted
4. Complete compliance requirements

**Fully Managed (COBO) Enrollment:**
- Requires factory reset
- QR code or NFC enrollment during setup wizard
- Zero-touch enrollment for bulk deployment

**Enrollment Errors:**
| Error | Fix |
|-------|-----|
| "Device not supported" | Check Android version requirements |
| "Play Protect certification required" | Uncertified device — use certified hardware |
| "Work profile setup failed" | Clear Company Portal data, retry |
| Compliance not met | Check password policy, encryption, OS version |

---

### Android Email

**Outlook for Android Issues:**
1. Clear Outlook cache: Settings > Apps > Outlook > Storage > Clear Cache
2. Remove account and re-add
3. Check Conditional Access / device compliance
4. Verify Modern Authentication enabled in tenant

**Gmail (G Suite/Workspace):**
1. Settings > Accounts > Google > Account sync > Toggle Mail
2. Remove and re-add account
3. Check app permissions (Contacts, Calendar)

---

### Android App Issues

**App Not Installing from MDM:**
1. Check Google Play managed account is correct
2. Ensure device is in correct assignment group
3. Check Play Store connectivity
4. Review Android Enterprise binding in MDM console

**Work Apps Not Appearing:**
- Work profile badge should appear on managed apps
- Check work profile is running: pull down notifications > Work profile tile
- If work profile paused, tap to resume

---

## 3. General Mobile Device Issues

### Battery Drain
**iOS:**
```
Settings > Battery > Battery Health & Charging
Settings > Battery > (review per-app usage)
Settings > Privacy & Security > Location Services (limit background)
```

**Android:**
```
Settings > Battery > Battery usage
Settings > Apps > [App] > Battery > Restrict background activity
```

**Common culprits:** Push email (reduce fetch frequency), Location services, Background app refresh, Screen brightness, Sync intervals

---

### Storage Full

**iOS:**
```
Settings > General > iPhone Storage
# Review recommendations: Offload Unused Apps
# Delete large apps, photos, messages
```

**Android:**
```
Settings > Storage
Files by Google app > Clean
```

**MDM Storage Reports:**
- Intune: Devices > All devices > Hardware > Free storage space
- Jamf: Device inventory > Storage

---

### Passcode/Biometric Issues

**iOS Passcode Forgotten:**
1. If MDM enrolled: Admin can send remote lock/wipe
2. Recovery mode restore (erases device)
3. If iCloud Activation Lock: requires Apple ID

**Android PIN Forgotten:**
1. After failed attempts: option to unlock with Google account
2. MDM admin can send remote lock with new PIN
3. Factory reset via Recovery mode

---

## 4. MDM Console — Common Admin Tasks

### Intune (Microsoft Endpoint Manager)

**Remote Actions:**
- Remote lock
- Reset passcode (iOS)
- Retire (remove corporate data, unenroll)
- Wipe (factory reset)
- Sync device
- Collect diagnostics

**Check Compliance:**
```
Devices > All devices > [Device] > Device compliance
Devices > Monitor > Device compliance
```

**Push Policies:**
```
Devices > Configuration profiles > Assign
Apps > All apps > Assignments
```

### Jamf Pro (iOS)

**Remote Commands:**
```
Computers/Devices > [Device] > Management > Send Remote Command
- Lock Device
- Erase Device
- Clear Passcode
- Send Blank Push (wake device for MDM sync)
```

**Scope Issues:**
1. Check device is in correct static/smart group
2. Verify policy/profile scope includes the device
3. Check policy frequency (once vs ongoing)
4. Flush policy logs: Management > Flush Policy Logs

---

## 5. Activation Lock

### iOS Activation Lock
- Enabled automatically with iCloud/Apple ID
- Blocks device use after erase without Apple ID
- **MDM Bypass (ABM enrolled devices):**
  - Jamf: Devices > [Device] > Management > Remove Activation Lock
  - Intune: requires Activation Lock bypass code
- Requires device was added to ABM before enrollment

### Android Factory Reset Protection
- Similar to iOS Activation Lock
- Google account required after factory reset
- **MDM bypass:** Zero-touch or Managed Google Play account

---

## 6. Mobile Security

### Lost or Stolen Device Procedure
1. Admin receives report from user
2. **Immediately:** Send Remote Lock via MDM
3. Verify last location (if location services enabled)
4. **Within 1 hour:** Decision — wait for recovery or Remote Wipe
5. Document incident, notify security team
6. Revoke OAuth tokens (Azure AD > User > Revoke sign-in sessions)
7. File police report if required by policy

### Conditional Access (Azure AD / Intune)
Policies to enforce:
- Require device compliance
- Require approved client apps (Outlook, Teams)
- Require MFA on mobile
- Block legacy authentication protocols
- Restrict access from unknown/unmanaged devices

### App Protection Policies (MAM)
Without full enrollment:
- Require PIN for work apps
- Block copy/paste between work and personal apps
- Block screenshots in work apps
- Remote wipe work data only (leave personal intact)
- Encrypt work data at rest

---

## 7. Escalation Checklist

Before escalating mobile issues:
- [ ] Device model and OS version documented
- [ ] MDM enrollment status confirmed
- [ ] Error message/screenshot captured
- [ ] Network type tested (Wi-Fi vs cellular)
- [ ] Basic reboot performed
- [ ] Company Portal / MDM app version noted
- [ ] User's M365 license and group membership verified
- [ ] Compliance status reviewed in MDM console
- [ ] Recent policy or app changes reviewed

---

*Last Updated: 2025 | IT Operations Documentation Library*
