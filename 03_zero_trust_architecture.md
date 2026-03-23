# Zero Trust Security Architecture — Enterprise Implementation Guide

## Table of Contents
1. [Zero Trust Principles](#zt-principles)
2. [Zero Trust Maturity Model](#zt-maturity)
3. [Identity Pillar](#identity-pillar)
4. [Device Pillar](#device-pillar)
5. [Network Pillar](#network-pillar)
6. [Application Pillar](#app-pillar)
7. [Data Pillar](#data-pillar)
8. [Visibility & Analytics](#visibility)
9. [Automation & Orchestration](#automation)
10. [Microsoft Zero Trust Implementation](#ms-zt)
11. [Zero Trust Assessment](#zt-assessment)
12. [Roadmap & Quick Wins](#zt-roadmap)

---

## 1. Zero Trust Principles {#zt-principles}

### Core Tenets
> **"Never trust, always verify. Assume breach. Verify explicitly. Use least-privilege access."**

| Old Model (Castle & Moat) | Zero Trust |
|--------------------------|------------|
| Trust everything inside the perimeter | Trust nothing by default — inside or outside |
| VPN = trusted access | Continuous verification of every request |
| Single sign-on at network boundary | Identity-aware access for every resource |
| Flat internal network | Micro-segmented, least-privilege network |
| Reactive security | Assume breach, proactive detection |

### NIST SP 800-207 Zero Trust Tenets
1. All data sources and services are treated as resources
2. All communication is secured regardless of network location
3. Access to individual enterprise resources is granted per session
4. Access determined by dynamic policy (identity + device + behavior)
5. All owned and non-owned devices are monitored for security posture
6. Authentication and authorization are dynamic and strictly enforced
7. Collect as much information as possible to improve security posture

---

## 2. Zero Trust Maturity Model {#zt-maturity}

### CISA Zero Trust Maturity Model (5 Pillars, 4 Stages)

```
Stage 1: Traditional     → Siloed, manual, static
Stage 2: Initial         → Attribute-aware, partial automation
Stage 3: Advanced        → Integrated pillars, automated enforcement
Stage 4: Optimal         → Fully dynamic, fully automated, AI-driven
```

### Maturity Assessment Grid
| Pillar | Traditional | Initial | Advanced | Optimal |
|--------|-------------|---------|----------|---------|
| **Identity** | Static roles, no MFA | MFA on some apps | Risk-based MFA everywhere | Continuous adaptive auth |
| **Device** | No compliance check | Basic MDM | Compliance gates access | Real-time health signal |
| **Network** | Flat LAN, VPN | Some segmentation | Microsegmentation | All east-west inspected |
| **Application** | Perimeter-based access | SSO added | ZTNA for all apps | App-aware policies |
| **Data** | No classification | Manual labels | Auto-classification | ML-based, encrypted everywhere |

---

## 3. Identity Pillar {#identity-pillar}

### Identity Is the New Perimeter

```
Every access request must answer:
  WHO is requesting? (Identity)
  WHAT are they trying to access? (Resource)
  FROM WHERE? (Device, location, network)
  HOW? (Application, method)
  IS THIS NORMAL? (Behavior analytics)
```

### Conditional Access Policies (Microsoft Entra ID)

#### Policy 1: Require MFA for All Cloud Apps
```json
{
  "displayName": "ZT-001: Require MFA for All Users",
  "state": "enabled",
  "conditions": {
    "users": { "includeUsers": ["All"] },
    "applications": { "includeApplications": ["All"] },
    "locations": { "excludeLocations": ["AllTrusted"] }
  },
  "grantControls": {
    "operator": "OR",
    "builtInControls": ["mfa"]
  }
}
```

#### Policy 2: Block Legacy Authentication
```json
{
  "displayName": "ZT-002: Block Legacy Authentication",
  "state": "enabled",
  "conditions": {
    "clientAppTypes": ["exchangeActiveSync", "other"],
    "users": { "includeUsers": ["All"] }
  },
  "grantControls": {
    "operator": "OR",
    "builtInControls": ["block"]
  }
}
```

#### Policy 3: Require Compliant Device
```json
{
  "displayName": "ZT-003: Require Compliant or Hybrid AD Joined Device",
  "conditions": {
    "users": { "includeUsers": ["All"] },
    "applications": { "includeApplications": ["All"] }
  },
  "grantControls": {
    "operator": "OR",
    "builtInControls": ["compliantDevice", "domainJoinedDevice"]
  }
}
```

#### Policy 4: High-Risk Sign-In Remediation
```json
{
  "displayName": "ZT-004: Block High Risk Sign-Ins",
  "conditions": {
    "signInRiskLevels": ["high"],
    "users": { "includeUsers": ["All"] }
  },
  "grantControls": {
    "operator": "OR",
    "builtInControls": ["block"]
  }
}
```

### Privileged Identity Management (PIM)
```
Zero Trust Rule: No permanent privileged access.
All admin roles = JIT (Just-In-Time), time-limited, approval-required.

PIM Flow:
  User requests elevation → Justification provided → Approval (manager or auto)
  → MFA required → Role active for N hours → Auto-expires → Audit logged

PIM Configuration Recommendations:
  Global Admin:           Require approval, 1-hour activation, MFA
  Privileged Role Admin:  Require approval, 2-hour activation, MFA
  Security Admin:         Self-approval, 4-hour activation, MFA
  Exchange Admin:         Self-approval, 8-hour activation, MFA
  Helpdesk Admin:         Self-approval, 8-hour activation, MFA
```

### Identity Governance
```powershell
# Access reviews — who still needs what access?
# Quarterly review for privileged roles
# Semi-annual review for sensitive groups
# Annual review for all external users

# PowerShell: Find users with no MFA registered
$users = Get-MgUser -All
foreach ($user in $users) {
    $methods = Get-MgUserAuthenticationMethod -UserId $user.Id
    if ($methods.Count -le 1) {   # Only password
        Write-Output "$($user.UserPrincipalName) — No MFA registered"
    }
}
```

---

## 4. Device Pillar {#device-pillar}

### Device Trust Framework
```
Level 0 — Unknown device       → Block or guest network only
Level 1 — Registered           → Basic access (read-only, web apps)
Level 2 — Enrolled (MDM)       → Standard access
Level 3 — Compliant            → Full corporate access
Level 4 — Privileged/Secured   → Admin tools, sensitive data
```

### Intune Compliance Policy (Windows)
```
Minimum OS version: 10.0.19041 (Win10 20H1)
Maximum OS version: Latest
BitLocker required: Yes
Secure Boot required: Yes
Code Integrity required: Yes
Firewall required: Yes
Antivirus required: Yes (Defender)
Real-time protection: Required
Microsoft Defender ATP: Required — Machine Risk Score ≤ Medium
Password required: Yes (6+ chars, complexity)
Inactivity lock: 15 minutes
```

### Microsoft Defender for Endpoint Risk-Based Access
```
Risk Level → Access Granted:
  Clear  → Full access
  Low    → Full access (monitor)
  Medium → Limited access (block sensitive data)
  High   → Block all corporate access
  
Integration: Intune compliance policy checks MDE risk score
→ Non-compliant = Conditional Access blocks access
→ No manual intervention required
```

### Device Attestation (Windows Hello for Business)
```
WHfB replaces passwords with:
- TPM-backed key pair (device-bound)
- PIN or biometric (user-bound)
- Kerberos Cloud Trust or Key Trust

Result: Phishing-resistant authentication — no password to steal
Requirement: TPM 2.0, Windows 10 1709+
```

---

## 5. Network Pillar {#network-pillar}

### Microsegmentation Strategy
```
Old:    Internet → Firewall → [Flat internal network — trust everything]

New (ZT):
  Internet
    ↓
  Perimeter Firewall (L7 inspection)
    ↓
  ┌─────────────────────────────────────┐
  │  DMZ Segment                        │
  │  (Web servers, load balancers)      │
  └─────────────────────────────────────┘
    ↓ (application-aware firewall)
  ┌─────────────────────────────────────┐
  │  App Tier Segment (VLAN 100)        │
  │  Only accepts traffic from DMZ      │
  └─────────────────────────────────────┘
    ↓ (restrict to DB ports only)
  ┌─────────────────────────────────────┐
  │  Data Tier Segment (VLAN 200)       │
  │  Only app tier can query DB         │
  └─────────────────────────────────────┘
  
  User Segments (VLAN per department)
  Each VLAN: Can't talk to other VLANs without explicit firewall allow rule
```

### Zero Trust Network Access (ZTNA) vs. VPN
| Aspect | Traditional VPN | ZTNA |
|--------|----------------|------|
| Access granted | Full network | Per-application only |
| Trust model | Trusted once connected | Verified per request |
| Lateral movement | Possible | Blocked by design |
| User experience | Poor (slow, always-on) | Better (app-specific) |
| Examples | Cisco AnyConnect | Zscaler Private Access, Microsoft Entra App Proxy |

### Software-Defined Perimeter Implementation
```
ZTNA Components:
1. Identity Provider (Entra ID, Okta)
2. Device posture check (Intune, CrowdStrike)  
3. ZTNA Broker/Gateway (Zscaler, Cloudflare Access)
4. Application connector (lightweight agent in app network)

Flow:
  User → ZTNA Client → Identity verification + Device check
       → Policy engine evaluates → Access granted to SPECIFIC app
       → Connection proxied through broker (app never internet-exposed)
```

### DNS-Based Security (DNS Filtering)
```
Block categories:
  - Malware/C2 domains
  - Phishing sites
  - Adult content
  - Gambling/P2P
  - Known botnets

Tools: Cisco Umbrella, Cloudflare Gateway, Windows DNS Policy
  
Implementation via Intune (Windows):
  Deploy DNS over HTTPS profile → Force all DNS through filtered resolver
```

---

## 6. Application Pillar {#app-pillar}

### Application Access Tiers
```
Tier 1 — Public SaaS (M365, Salesforce)
  Control via: Conditional Access, CASB, session policies

Tier 2 — Internal Web Apps
  Control via: Entra Application Proxy (no VPN required)

Tier 3 — Legacy/Internal Apps
  Control via: ZTNA broker or privileged access workstation (PAW)

Tier 4 — Administrative interfaces (AD, vCenter, firewalls)
  Control via: Privileged Access Workstations + Just-in-Time access only
```

### Cloud App Security / CASB Policies
```
Policy: Block download from unmanaged devices
  Trigger: User accesses SharePoint from non-compliant device
  Action: Allow read-only via browser session, block download

Policy: Detect shadow IT
  Method: Analyze proxy/firewall logs for unsanctioned apps
  Action: Alert on new apps > 50 users, block if high-risk category

Policy: Data exfiltration detection
  Trigger: >500MB upload to personal cloud storage in 1 hour
  Action: Block + alert security team + require justification

Policy: Impossible travel alert
  Trigger: Login from US then login from Europe within 30 minutes
  Action: Block session, require re-authentication + MFA
```

### Session Controls (Reverse Proxy)
```
Microsoft Defender for Cloud Apps Session Policy:
  When: Access sensitive SharePoint site
  From: Compliant managed device
  Action: Allow full access

  When: Access sensitive SharePoint site  
  From: Unmanaged/personal device
  Action: Allow view only (block download, copy, print)
  
  Protect session in real-time — no need to block access entirely
```

---

## 7. Data Pillar {#data-pillar}

### Data Classification Framework
```
Level 1 — Public
  Examples: Marketing material, public website
  Controls: None required

Level 2 — Internal
  Examples: Internal wikis, non-sensitive email
  Controls: Require authentication

Level 3 — Confidential
  Examples: HR records, financial data, client data
  Controls: MFA + compliant device + encryption + audit logging

Level 4 — Restricted / Highly Confidential
  Examples: M&A data, PII, credentials, board materials
  Controls: PAW required + PIM elevation + DLP + Rights Management
```

### Microsoft Purview (MIP) Sensitivity Labels
```
Labels applied via:
  Auto-labeling policies (ML-based content matching)
  User application in Office apps
  Default labels for containers (Teams, SharePoint, Groups)

Encryption settings per label:
  Confidential: Encrypt, restrict to org users, no forwarding
  Restricted: Encrypt, only specific group can decrypt

DLP Policy Examples:
  Rule: Credit card numbers detected in email → Block send + notify user
  Rule: Confidential label + external recipient → Block + manager approval
  Rule: Bulk download of labeled files (>100) → Block + alert
```

### Data Loss Prevention (DLP) Policy Priorities
```
Priority 1: Block exfiltration of restricted data
  - No upload to personal cloud storage
  - No email to external with PII/PCI data unencrypted
  
Priority 2: Encrypt data in transit and at rest
  - All endpoints: BitLocker (at rest)
  - All data in M365: Customer-managed keys (advanced)
  - All database: Transparent Data Encryption
  
Priority 3: Monitor and audit data access
  - Unified audit log: All file access, sharing, download events
  - Alert on unusual access patterns
  - Retain logs minimum 12 months
```

---

## 8. Visibility & Analytics {#visibility}

### Security Information & Event Management (SIEM)
```
Microsoft Sentinel Data Sources for Zero Trust:
  ✓ Entra ID Sign-in Logs (identity signals)
  ✓ Entra ID Audit Logs (changes)
  ✓ Conditional Access logs
  ✓ Intune Device Compliance logs
  ✓ Defender for Endpoint alerts
  ✓ Defender for Cloud Apps activity
  ✓ Network firewall logs
  ✓ DNS query logs
  ✓ Azure Activity logs
```

### Key Sentinel Analytics Rules (KQL)
```kql
// Detect impossible travel
SigninLogs
| where ResultType == 0
| extend City = tostring(LocationDetails.city)
| extend Country = tostring(LocationDetails.countryOrRegion)
| summarize Logins = count(), Countries = make_set(Country) by UserPrincipalName, bin(TimeGenerated, 1h)
| where array_length(Countries) > 1
| project UserPrincipalName, TimeGenerated, Countries

// Detect MFA fatigue attack (many MFA prompts)
SigninLogs
| where AuthenticationRequirement == "multiFactorAuthentication"
| where ResultType in ("50074", "500121")   // MFA denied
| summarize FailedMFA = count() by UserPrincipalName, bin(TimeGenerated, 1h)
| where FailedMFA > 10
| project UserPrincipalName, TimeGenerated, FailedMFA

// Detect admin role elevation outside business hours
AuditLogs
| where OperationName == "Add member to role"
| extend Target = tostring(TargetResources[0].userPrincipalName)
| extend Role = tostring(TargetResources[0].displayName)
| extend Hour = hourofday(TimeGenerated)
| where Hour < 7 or Hour > 19
| project TimeGenerated, InitiatedBy, Target, Role
```

### User & Entity Behavior Analytics (UEBA)
```
Baseline normal behavior:
  - Typical login times and locations
  - Normal file access patterns
  - Typical email volume and recipients
  - Normal application usage

Alert on deviations:
  - First-time access to sensitive resource
  - Access from new country
  - 10x increase in data downloaded
  - Lateral movement (accessing many systems quickly)
  - Accessing resources after hours
```

---

## 9. Automation & Orchestration {#automation}

### Automated Incident Response
```
Scenario: Compromised account detected (Entra ID Protection high risk)
  
  Automated response (SOAR playbook):
  1. Immediately revoke all active sessions (Revoke-MgUserSignInSession)
  2. Disable the user account
  3. Force MFA re-registration on next login
  4. Alert security team in Teams/email
  5. Create incident ticket in ServiceNow
  6. Add to watchlist for 30 days
  7. Notify manager
  
  No human required for steps 1-6 — completed within 60 seconds of detection
```

### Logic App / Power Automate Security Playbooks
```json
// Simplified playbook trigger: Sentinel Incident with "HighRisk" entity
{
  "trigger": "When_Microsoft_Sentinel_incident_created",
  "condition": "severity == 'High' AND entities contains 'Account'",
  "actions": [
    "Disable_Entra_User_Account",
    "Revoke_User_Sessions",
    "Post_Teams_Alert_to_SecOps",
    "Create_ServiceNow_Incident",
    "Send_Email_to_Manager"
  ]
}
```

---

## 10. Microsoft Zero Trust Implementation {#ms-zt}

### Recommended Implementation Order
```
Phase 1 — Identity (Months 1-3)
  ✓ Entra ID P2 (MFA + CA + PIM + ID Protection)
  ✓ Enable MFA for all users
  ✓ Block legacy authentication
  ✓ Deploy SSPR (Self-Service Password Reset)
  ✓ Configure PIM for all privileged roles

Phase 2 — Device (Months 2-4)
  ✓ Enroll all devices in Intune
  ✓ Configure compliance policies
  ✓ Enable Windows Autopilot
  ✓ Deploy Defender for Endpoint
  ✓ Conditional Access: Require compliant device

Phase 3 — Application (Months 3-6)
  ✓ Deploy Defender for Cloud Apps (CASB)
  ✓ Configure App Proxy for internal apps
  ✓ Implement session controls
  ✓ Discover and govern shadow IT

Phase 4 — Data (Months 4-8)
  ✓ Deploy Purview sensitivity labels
  ✓ Configure auto-labeling policies
  ✓ Implement DLP policies
  ✓ Encrypt sensitive data at rest

Phase 5 — Network (Months 6-12)
  ✓ Deploy ZTNA (replace VPN for most use cases)
  ✓ Implement microsegmentation
  ✓ Enable DNS filtering
  ✓ Inspect east-west traffic

Phase 6 — Visibility (Ongoing)
  ✓ Deploy Microsoft Sentinel
  ✓ Enable UEBA
  ✓ Automate incident response
  ✓ Continuous posture improvement
```

### Microsoft Secure Score
```
Target thresholds:
  Year 1: 40%+ (baseline hardening complete)
  Year 2: 60%+ (ZT Phase 1-3 complete)
  Year 3: 75%+ (ZT Phase 4-6 complete)

Top score improvements by effort:
  Easy wins:      MFA, block legacy auth, disable unused admin accounts
  Medium effort:  Intune compliance, Defender deployment, sensitivity labels
  High effort:    ZTNA, microsegmentation, custom detection rules
```

---

## 11. Zero Trust Assessment {#zt-assessment}

### Self-Assessment Checklist

#### Identity
- [ ] MFA enforced for all users (not just admins)
- [ ] Legacy authentication blocked
- [ ] Privileged access managed via PIM (no permanent admin roles)
- [ ] Risky sign-ins automatically blocked/remediated
- [ ] Access reviews conducted quarterly
- [ ] Service accounts use managed identities where possible
- [ ] Break-glass accounts secured and monitored

#### Device
- [ ] All devices enrolled in MDM
- [ ] Compliance policies enforced
- [ ] Non-compliant devices blocked via Conditional Access
- [ ] EDR deployed on all endpoints (Defender or equivalent)
- [ ] Patch compliance > 95% within 30 days of release
- [ ] BitLocker/FileVault enabled on all laptops

#### Network
- [ ] Network segmented (no flat LAN)
- [ ] East-west traffic inspected between segments
- [ ] VPN replaced or supplemented with ZTNA for remote access
- [ ] DNS filtering enabled
- [ ] Outbound internet access proxied and filtered

#### Application
- [ ] All apps behind SSO
- [ ] Conditional Access applied to all apps
- [ ] Shadow IT discovered and governed
- [ ] Session controls for unmanaged devices
- [ ] No direct internet exposure of admin interfaces

#### Data
- [ ] Data classification implemented
- [ ] Sensitivity labels applied to sensitive data
- [ ] DLP policies active for email, SharePoint, endpoint
- [ ] Encryption at rest for all sensitive data stores
- [ ] Data access audit logs retained 12+ months

---

## 12. Roadmap & Quick Wins {#zt-roadmap}

### 30-Day Quick Wins (Immediate Impact)
```
Week 1:
  ✓ Enable MFA for all global admins (< 1 hour)
  ✓ Create Conditional Access policy: Block legacy authentication (< 30 min)
  ✓ Enable Entra ID Identity Protection (risk policies)
  ✓ Review and disable unused admin accounts

Week 2:
  ✓ Enroll remaining unmanaged devices in Intune
  ✓ Create baseline compliance policy
  ✓ Enable Microsoft Defender for Endpoint (if not already)
  ✓ Enable unified audit logging in M365

Week 3:
  ✓ Configure Conditional Access: Require MFA for all users
  ✓ Block sign-ins from high-risk countries (if applicable)
  ✓ Enable SSPR to reduce helpdesk password resets
  ✓ Audit privileged role assignments — remove unnecessary

Week 4:
  ✓ Deploy Purview sensitivity labels (minimum 3: Internal, Confidential, Restricted)
  ✓ Create DLP policy for credit card / SSN detection
  ✓ Review external sharing settings in SharePoint/Teams
  ✓ Enable Defender for Cloud Apps discovery
```

### Success Metrics
| Metric | Baseline | 6-Month Target | 12-Month Target |
|--------|---------|----------------|-----------------|
| MFA adoption | 0% | 95% | 100% |
| Compliant devices | 0% | 80% | 95% |
| Legacy auth blocked | No | Yes | Yes |
| Privileged perm accounts | 100% | 25% | 0% |
| Mean time to detect | Unknown | < 7 days | < 24 hours |
| Mean time to respond | Manual | < 4 hours | < 1 hour (auto) |
| Secure Score | Baseline | +20 points | +40 points |

---

*Last Updated: 2025 | Framework: NIST SP 800-207, CISA ZT Maturity Model, Microsoft ZT Architecture*
