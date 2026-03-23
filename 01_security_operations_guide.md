# IT Security Operations Guide

## Overview
Comprehensive security operations reference covering endpoint security, identity security, network security, incident response, vulnerability management, and compliance. Applicable to organizations of all sizes.

---

## Table of Contents
1. [Security Framework Overview](#framework)
2. [Identity & Access Security](#identity)
3. [Endpoint Security](#endpoint)
4. [Network Security](#network)
5. [Vulnerability Management](#vuln-mgmt)
6. [Security Incident Response](#ir)
7. [Log Management & SIEM](#siem)
8. [Email Security](#email)
9. [Data Protection & DLP](#dlp)
10. [Security Baselines & Hardening](#hardening)
11. [Security Awareness & Training](#awareness)
12. [Compliance Frameworks Reference](#compliance)

---

## 1. Security Framework Overview <a name="framework"></a>

### NIST Cybersecurity Framework (CSF)

**IDENTIFY:** Know your assets, risks, and responsibilities
- Asset inventory (hardware, software, data)
- Risk assessment
- Governance policies
- Supply chain risk management

**PROTECT:** Implement safeguards
- Access control (MFA, RBAC, least privilege)
- Awareness training
- Data security
- Endpoint protection
- Secure configuration

**DETECT:** Monitor for events
- SIEM / log aggregation
- Endpoint detection (EDR)
- Network detection (IDS/IPS, NDR)
- Anomaly detection

**RESPOND:** Act on detected events
- Incident response plan
- Communications plan
- Analysis and containment
- Improvements

**RECOVER:** Restore normal operations
- Recovery planning
- Lessons learned
- Communications

### Security Controls Priority Order (SANS CIS Top 18)
```
ESSENTIAL (Start Here):
  1. Inventory and Control of Enterprise Assets
  2. Inventory and Control of Software Assets
  3. Data Protection
  4. Secure Configuration of Enterprise Assets
  5. Account Management
  6. Access Control Management

HIGH VALUE:
  7. Continuous Vulnerability Management
  8. Audit Log Management
  9. Email and Web Browser Protections
  10. Malware Defenses
  11. Data Recovery
  12. Network Infrastructure Management
  13. Network Monitoring and Defense
```

---

## 2. Identity & Access Security <a name="identity"></a>

### Multi-Factor Authentication (MFA)

**Deployment priority:**
```
1. All admin accounts (immediate — no exceptions)
2. All remote access (VPN, RDP, SSH)
3. All email / collaboration (Microsoft 365, Google Workspace)
4. All SaaS applications
5. All internal applications
```

**MFA Method Strength (strongest to weakest):**
```
Strongest:
  Hardware security keys (FIDO2/WebAuthn) — phishing-resistant
  
Strong:
  Authenticator apps (TOTP: Microsoft/Google Authenticator)
  
Acceptable:
  Push notifications (Microsoft Authenticator push)
  
Weaker (avoid if possible):
  SMS/Voice — SIM swap attacks possible
  
Never:
  Security questions alone
  No MFA
```

### Privileged Access Management (PAM)

```powershell
# Protected Users Security Group — applies strict Kerberos settings
# Prevents: NTLM auth, credential caching, DES/RC4 encryption, unconstrained delegation
# Use for all Tier 0/1 admin accounts

Add-ADGroupMember -Identity "Protected Users" -Members "admin.jsmith","admin.tjones"

# Verify Protected Users settings apply
Get-ADGroupMember "Protected Users" | Select-Object SamAccountName

# Just-In-Time access via PIM (Azure AD Privileged Identity Management)
# - Admin role not permanently assigned
# - Request activation with justification
# - Activation valid for 1–8 hours
# - Audited and alertable
```

### Conditional Access Policies (Azure AD)

**Essential policies:**
```
Policy 1: Require MFA for all users
  Users: All
  Conditions: Any location
  Grant: Require MFA

Policy 2: Block legacy authentication
  Users: All
  Conditions: Client apps = Exchange ActiveSync, Other clients
  Grant: Block

Policy 3: Require compliant device for sensitive apps
  Users: All
  Cloud apps: Exchange, SharePoint, Salesforce
  Conditions: Any device
  Grant: Require compliant device OR MFA

Policy 4: Block access from high-risk countries
  Users: All  
  Conditions: Locations = Named locations (exclude your countries)
  Grant: Block

Policy 5: Require MFA for admin roles
  Users: All admin roles
  Conditions: Any location
  Grant: Require MFA + Require compliant device
```

---

## 3. Endpoint Security <a name="endpoint"></a>

### Endpoint Security Stack

```
Layer 1: Prevention
  - Microsoft Defender Antivirus / Endpoint (or third-party EDR)
  - Application whitelisting (AppLocker, WDAC)
  - Patch management (within 30 days for critical, 14 for emergency)
  - Host-based firewall
  - USB/removable media control

Layer 2: Detection
  - EDR agent with behavioral analysis
  - Endpoint logging to SIEM
  - File integrity monitoring (critical systems)

Layer 3: Response
  - EDR isolation capability (quarantine infected endpoint)
  - Remote wipe capability (for mobile/laptop)
  - Forensic imaging capability
```

### Microsoft Defender for Endpoint Key Configurations

```powershell
# Check Defender status
Get-MpComputerStatus | Select-Object AMEngineVersion, AMProductVersion, AMServiceEnabled, 
  AntispywareEnabled, AntivirusEnabled, RealTimeProtectionEnabled, 
  TamperProtectionSource, IoavProtectionEnabled

# Update signatures
Update-MpSignature

# Run a quick scan
Start-MpScan -ScanType QuickScan

# Check threat history
Get-MpThreatDetection | Sort-Object InitialDetectionTime -Descending | 
  Select-Object -First 10 | Select-Object ThreatName, InitialDetectionTime, ActionSuccess

# Enable cloud-delivered protection
Set-MpPreference -MAPSReporting Advanced -SubmitSamplesConsent SendAllSamples

# Enable Attack Surface Reduction rules
Set-MpPreference -AttackSurfaceReductionRules_Ids @(
  "BE9BA2D9-53EA-4CDC-84E5-9B1EEEE46550",  # Block executable content from email
  "D4F940AB-401B-4EFC-AADC-AD5F3C50688A",  # Block Office apps from creating child processes
  "3B576869-A4EC-4529-8536-B80A7769E899",  # Block Office apps from creating executable content
  "75668C1F-73B5-4CF0-BB93-3ECF5CB7CC84",  # Block Office from injecting into processes
  "D3E037E1-3EB8-44C8-A917-57927947596D",  # Block JavaScript/VBScript from launching executables
  "5BEB7EFE-FD9A-4556-801D-275E5FFC04CC",  # Block potentially obfuscated scripts
  "92E97FA1-2EDF-4476-BDD6-9DD0B4DDDC7B"   # Block Win32 API calls from Office macros
) -AttackSurfaceReductionRules_Actions @(1,1,1,1,1,1,1)  # 1 = Block
```

### Patch Management Process

```
Patch Categories:
  Critical (CVSS 9.0+): Patch within 7 days, emergency 24 hours
  High (CVSS 7.0–8.9): Patch within 14 days
  Medium (CVSS 4.0–6.9): Patch within 30 days
  Low (<4.0): Next maintenance cycle

Patch Deployment Rings:
  Ring 0 (Pilot): IT team computers (5–10 devices) — immediate
  Ring 1 (Early): Volunteers + IT-adjacent teams (10–15%) — +1 week
  Ring 2 (Broad): General population (70%) — +2 weeks
  Ring 3 (Deferred): Special systems (servers, kiosks, etc.) — +4 weeks

Emergency Patch Protocol:
  1. Assess impact scope
  2. Get approval (manager + CISO or delegate)
  3. Deploy to Ring 0 with 2-hour observation
  4. If stable, broad deploy within 24–48 hours
  5. Document all exceptions
```

---

## 4. Network Security <a name="network"></a>

### Network Segmentation

```
Best practice segmentation:
  Internet → [Firewall] → DMZ → [Firewall] → Internal

  Internal zones:
    Production: ERP, databases, critical apps
    Users: Employee workstations
    Servers: Internal servers
    Management: Network management, out-of-band
    IoT/OT: Printers, cameras, HVAC, industrial
    Guest: Internet-only, no internal access
    Development: Dev/test systems (isolated from prod)

  Key rules:
    - Never allow IoT/Guest to reach internal zones
    - Require explicit allow rules (default deny)
    - Log all cross-zone traffic
    - No lateral movement from user VLAN to server VLAN without proxy/control
```

### Firewall Rule Audit

```bash
# Regular firewall rule review questions:
# 1. Are there ANY or 0.0.0.0/0 source rules? (Red flag — review every one)
# 2. Are there rules for decommissioned systems? (Remove them)
# 3. Are rules documented with business justification? (Each should have a ticket)
# 4. Are there rules for legacy protocols (Telnet, FTP, HTTP for admin)? (Remove/restrict)
# 5. Are rules as specific as possible? (Avoid broad port ranges if not needed)

# Review firewall rules for "any" sources (Windows Firewall)
Get-NetFirewallRule | Where-Object {$_.Enabled -eq "True"} | 
  Get-NetFirewallAddressFilter | Where-Object {$_.RemoteAddress -eq "Any"} |
  ForEach-Object { Get-NetFirewallRule -AssociatedNetFirewallAddressFilter $_ | 
    Select-Object DisplayName, Direction, Action }
```

### DNS Security

```
DNSSEC: Enable for zones you control
DNS over HTTPS (DoH): Consider for client security
DNS Filtering: Block malicious domains at DNS level (Cisco Umbrella, Cloudflare Gateway, etc.)

DNS monitoring for:
  - Large numbers of NXDOMAIN responses (possible malware C2 beaconing)
  - Unusually long subdomain queries (DNS tunneling)
  - Queries to known malicious domains (threat intel feeds)
  - Internal systems querying external DNS (bypass of internal DNS)
```

---

## 5. Vulnerability Management <a name="vuln-mgmt"></a>

### Vulnerability Management Program

```
Scan Frequency:
  External attack surface: Weekly
  Internal infrastructure: Monthly
  Web applications: Quarterly + after major changes
  
Tools: Nessus, Rapid7 InsightVM, Qualys, Microsoft Defender Vulnerability Management

Workflow:
  1. Scan → 2. Ingest results → 3. Prioritize (CVSS + exploitability + asset criticality)
  4. Assign to owners → 5. Track remediation → 6. Verify → 7. Report
```

### Vulnerability Prioritization (CVSS + Context)
```
Prioritize ABOVE their CVSS score:
  - Public exploit available (Metasploit, ExploitDB)
  - Part of CISA KEV (Known Exploited Vulnerabilities catalog)
  - Exposed to internet
  - On Tier 0/1 critical systems
  - No available workaround

Deprioritize (document the reasoning):
  - Air-gapped systems with no external connectivity
  - Compensating controls in place
  - Vendor end-of-life with documented exception
  - False positive (document and verify)
```

### Vulnerability Remediation Commands

```powershell
# Windows: Check for missing patches
Get-WUList   # Requires PSWindowsUpdate module
Get-HotFix | Sort-Object InstalledOn -Descending | Select-Object -First 20

# Check specific CVE by KB article
Get-HotFix -Id KB5005033  # Example KB

# Linux: Check for updates
apt list --upgradable 2>/dev/null | head -20
yum check-update
dnf check-update

# Apply security updates only (Ubuntu/Debian)
apt-get upgrade --with-new-pkgs $(apt-get --dry-run upgrade | grep -i security | awk '{print $2}')
```

---

## 6. Security Incident Response <a name="ir"></a>

### Incident Response Plan Phases

**Phase 1: Preparation**
```
- IR team defined with roles: Incident Commander, Technical Lead, Communications Lead
- Contact list (CISO, legal, HR, PR, key executives)
- Playbooks for common scenarios (ransomware, phishing, data breach, insider threat)
- Tools ready: forensic workstation, SIEM access, EDR console, communication channel
- Tabletop exercises quarterly
```

**Phase 2: Identification**
```
Sources of detection:
  - SIEM alert
  - EDR alert (malware, suspicious behavior)
  - IDS/IPS alert
  - User report
  - External notification (FBI, vendor, third party)
  - Threat intel

Key questions:
  - What systems are affected?
  - What data may be at risk?
  - Is the incident ongoing?
  - What is the attacker's likely objective?
```

**Phase 3: Containment**

*Short-term containment:*
```powershell
# Isolate infected endpoint (Microsoft Defender for Endpoint)
# Via Defender portal or:
Invoke-MpIsolation -Action Isolate  # Run from Defender console

# Disable user account
Disable-ADAccount -Identity compromised.user

# Block IP at firewall
New-NetFirewallRule -DisplayName "BLOCK - Malicious IP" -Direction Outbound -RemoteAddress "203.0.113.1" -Action Block

# Revoke Azure AD refresh tokens (all sessions)
Revoke-AzureADUserAllRefreshToken -ObjectId (Get-AzureADUser -ObjectId "user@domain.com").ObjectId
```

**Phase 4: Eradication**
```
Remove malware:
  - Run EDR full scan
  - Manual artifact removal if needed
  - Reimage if trust cannot be re-established

Remove persistence mechanisms:
  - Check scheduled tasks, services, registry run keys, startup items
  - Review AD for new accounts, modified group memberships
  - Check for new admin accounts or elevated permissions
  - Review email rules for forwarding/deletion rules
```

**Phase 5: Recovery**
```
- Restore from known-clean backup
- Verify backup integrity BEFORE restoring
- Restore in isolated environment to test
- Monitor restored systems closely for 2–4 weeks
- Change all potentially compromised passwords
- Rotate service account credentials
- Enable enhanced logging on affected systems
```

**Phase 6: Lessons Learned**
```
Post-Incident Review (within 2 weeks):
  - Timeline of events
  - Root cause analysis
  - What detection was missed and why?
  - What slowed down response?
  - What went well?
  - Action items to prevent recurrence (with owners and due dates)
```

### Ransomware Response Playbook
```
IMMEDIATE (0–60 minutes):
  1. DO NOT PAY — get leadership/legal/insurance involved before any payment decision
  2. Disconnect affected systems from network (pull cable or disable NIC — do NOT shut down if possible)
  3. Notify CISO, IT Director, Legal immediately
  4. Activate incident response team
  5. Preserve logs and forensic artifacts
  6. Document scope: What's encrypted? What systems? What's the ransom note?

SHORT-TERM (1–24 hours):
  1. Identify patient zero (which system was first infected)
  2. Identify attack vector (phishing? exposed RDP? vendor compromise?)
  3. Check if backups are intact and unaffected
  4. Assess regulatory notification requirements (GDPR, HIPAA, state laws)
  5. Notify cyber insurance carrier
  6. Consider law enforcement notification (FBI IC3)

RECOVERY DECISION:
  Option A: Restore from clean backups (preferred)
  Option B: Decrypt with decryption tool (check nomoreransom.org for free tools)
  Option C: Ransom payment (last resort, legal/insurance/leadership decision)
```

---

## 7. Log Management & SIEM <a name="siem"></a>

### Critical Logs to Collect
```
Windows Domain Controllers:
  - Security log: 4624/4625 (logon success/fail), 4720 (user created), 4728/4732/4756 (group changes),
    4740 (lockout), 4776 (credential validation), 5136 (AD object modified)

Windows Servers/Endpoints:
  - Security log: 4624, 4625, 4648, 4688 (process creation), 4698/4702 (scheduled task)
  - Sysmon (if deployed): Process creation, network connections, registry changes

Network Devices:
  - Firewall: Allow/deny logs, especially for cross-zone traffic
  - VPN: Authentication events, connection duration, data transferred
  - DNS: All queries (if volume permits), or at minimum NXDOMAIN

Applications:
  - Web proxies: All outbound web traffic
  - Email gateway: Delivery, rejection, quarantine events
  - Authentication systems: All auth events (success + failure)

Cloud:
  - Azure AD Sign-in logs + Audit logs
  - AWS CloudTrail
  - O365 Unified Audit Log
```

### Key SIEM Detection Rules

```
Rule: Multiple failed logins followed by success
  Condition: >5 failed logins in 10 min from same source, then success
  Severity: HIGH
  Possible: Brute force / credential stuffing

Rule: Login from impossible travel
  Condition: User logs in from Country A, then Country B within < 2 hours
  Severity: HIGH
  Possible: Compromised credentials, VPN usage

Rule: New local admin account created
  Condition: Event 4720 + 4732 (added to Administrators group) for local accounts
  Severity: HIGH
  Possible: Persistence mechanism after compromise

Rule: Suspicious process execution
  Condition: PowerShell.exe spawned by Word.exe or Excel.exe
  Severity: CRITICAL
  Possible: Macro-based malware

Rule: Large data transfer to external IP
  Condition: Single internal host sending >500MB to external IP in 1 hour
  Severity: HIGH
  Possible: Data exfiltration

Rule: Disabled AV / tamper protection
  Condition: Event ID 5001 (Defender disabled)
  Severity: CRITICAL
  Possible: Attacker disabling defenses
```

---

## 8. Email Security <a name="email"></a>

### Email Authentication Records (DNS)

```
SPF (Sender Policy Framework):
  TXT record at @:
  "v=spf1 include:spf.protection.outlook.com include:sendgrid.net -all"
  # -all = hard fail (reject) for unauthorized senders

DKIM (DomainKeys Identified Mail):
  CNAME or TXT record at selector._domainkey:
  # Configured through your email provider's admin portal

DMARC (Domain-based Message Authentication):
  TXT record at _dmarc:
  "v=DMARC1; p=reject; rua=mailto:dmarc-reports@domain.com; ruf=mailto:dmarc-forensic@domain.com; pct=100"
  
  # Policy progression:
  # p=none → p=quarantine → p=reject
  # Start with none, monitor reports, then tighten

BIMI (Brand Indicators for Message Identification):
  Requires DMARC enforcement (reject/quarantine) + verified logo
```

### Anti-Phishing Controls

```
Technical controls:
  □ Enable External Email Warning banner (Outlook/Exchange)
  □ Configure Safe Links (detonate URLs at click time)
  □ Configure Safe Attachments (sandbox email attachments)
  □ Block auto-forwarding to external domains
  □ Block legacy auth protocols
  □ Implement spoof intelligence / anti-impersonation
  □ Enable zero-hour auto purge (ZAP) for post-delivery removal

User controls:
  □ Phishing simulation training (quarterly at minimum)
  □ Easy reporting mechanism (Report Phishing button)
  □ Clear escalation path when employees receive suspicious email
```

---

## 9. Data Protection & DLP <a name="dlp"></a>

### Data Classification Framework
```
CONFIDENTIAL:
  - Passwords, credentials, private keys
  - PII (SSN, DOB, financial account numbers)
  - Health information (PHI)
  - Trade secrets, M&A information
  - Payment card data (PCI DSS scope)
  Controls: Encryption at rest and in transit, strict access control, DLP policies

INTERNAL:
  - Business strategies, non-public financials
  - Employee records, HR data
  - Customer data (non-regulated)
  Controls: Access control, standard encryption

PUBLIC:
  - Press releases, marketing materials
  - Public website content
  Controls: Integrity protection, no special confidentiality needed
```

### Microsoft Purview DLP Key Policies
```
Essential DLP policies to implement:
  1. Block sending US SSN to external email
  2. Block sending credit card numbers to external email  
  3. Alert on bulk export of customer data to USB
  4. Block sharing sensitive files via SharePoint to anonymous links
  5. Alert on large amounts of data downloaded to unmanaged device
```

---

## 10. Security Baselines & Hardening <a name="hardening"></a>

### Windows Server Hardening Checklist
```
AUTHENTICATION:
  □ Rename default Administrator account
  □ Disable Guest account
  □ Enable account lockout policy
  □ Require complex passwords (min 14 chars)
  □ Enable MFA for administrative access

NETWORK:
  □ Disable unused network protocols (IPv6 if not used, NetBIOS if not needed)
  □ Enable Windows Firewall (all profiles)
  □ Restrict RDP to management subnet only
  □ Disable SMBv1 (critical!)
  □ Enable SMB signing

SERVICES:
  □ Disable unnecessary services (Print Spooler if not printing, Telnet, FTP server, etc.)
  □ Run services as least-privileged accounts (not SYSTEM if avoidable)
  □ Enable Windows Defender
  □ Enable audit policies (logon, account management, privilege use, object access)

PATCHING:
  □ Current OS patches applied
  □ All software current
  □ No EOL software running

REMOTE MANAGEMENT:
  □ Disable Telnet, FTP
  □ Use WinRM over HTTPS
  □ Restrict PowerShell remoting to admin subnet
  □ Enable PowerShell Script Block Logging and Module Logging
```

### Critical Security Commands

```powershell
# Disable SMBv1 (URGENT — WannaCry/NotPetya attack vector)
Set-SmbServerConfiguration -EnableSMB1Protocol $false -Force
Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol

# Enable SMB signing
Set-SmbServerConfiguration -RequireSecuritySignature $true -EnableSecuritySignature $true -Force

# Disable legacy TLS (1.0, 1.1)
# Disable TLS 1.0
New-Item -Path "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.0\Server" -Force
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.0\Server" -Name "Enabled" -Value 0

# Enable PowerShell logging
$registryPath = "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging"
New-Item -Path $registryPath -Force
Set-ItemProperty -Path $registryPath -Name "EnableScriptBlockLogging" -Value 1

# Check for open shares (data exposure risk)
Get-SmbShare | Where-Object {$_.Name -notmatch "ADMIN\$|C\$|IPC\$"} | 
  Select-Object Name, Path, Description
Get-SmbShareAccess -Name "SharedFolder"
```

---

## 11. Security Awareness & Training <a name="awareness"></a>

### Annual Security Training Program

```
Monthly: Short security tips (newsletter, Teams/Slack message)
Quarterly: Phishing simulations (vary templates: invoice, IT helpdesk, CEO fraud)
Quarterly: Security news briefing (most relevant recent attacks)
Annually: Mandatory security awareness training (all staff)
On-hire: Security orientation as part of IT onboarding

Metrics to track:
  - Phishing click rate (target: <5%)
  - Training completion rate (target: 100% by deadline)
  - Simulated phishing report rate (target: >50% report vs. click)
  - Security incident self-reports (increase = positive awareness indicator)
```

---

## 12. Compliance Frameworks Reference <a name="compliance"></a>

### Framework Quick Reference

| Framework | Applies To | Key Controls |
|---|---|---|
| SOC 2 Type II | SaaS vendors, tech companies | Availability, security, confidentiality |
| ISO 27001 | Any organization | Full ISMS, 114 controls |
| PCI DSS | Card payment processing | 12 requirements, quarterly scans |
| HIPAA | US healthcare + business associates | PHI protection, breach notification |
| GDPR | EU personal data processing | Data rights, DPO, 72hr breach notification |
| CCPA | California consumer data | Consumer rights, opt-out |
| NIST 800-53 | US federal agencies + contractors | 900+ controls in 20 families |
| NIST CSF | Any organization | Identify/Protect/Detect/Respond/Recover |
| CIS Controls | Any organization | 18 prioritized controls |

### GDPR Key Requirements for IT
```
Technical requirements:
  □ Encryption of personal data at rest and in transit
  □ Pseudonymization where possible
  □ Access logs for systems processing personal data
  □ Data retention automation (purge per schedule)
  □ Secure deletion capability (right to erasure)
  □ Breach detection and 72-hour notification capability
  □ Privacy by design in new system development
  □ Data Processing Impact Assessments (DPIA) for high-risk processing
```
