# Network Security & Firewall Management Guide

## Table of Contents
1. [Network Security Architecture](#architecture)
2. [Firewall Management](#firewall)
3. [Network Segmentation & VLANs](#segmentation)
4. [Zero Trust Networking](#zerotrust)
5. [VPN & Remote Access](#vpn)
6. [DNS Security](#dns)
7. [Network Access Control (NAC)](#nac)
8. [Intrusion Detection & Prevention](#ids-ips)
9. [DDoS Protection](#ddos)
10. [Network Monitoring & Logging](#monitoring)
11. [Wireless Security](#wireless)
12. [Network Security Hardening](#hardening)
13. [Incident Response for Network Events](#incident)

---

## 1. Network Security Architecture <a name="architecture"></a>

### Defense-in-Depth Network Model
```
INTERNET
    │
    ▼
┌─────────────────────────────────────────┐
│  EDGE / PERIMETER                        │
│  - DDoS scrubbing (Cloudflare/Akamai)    │
│  - Border firewall / ASA / FortiGate     │
│  - BGP routing, upstream ISP             │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│  DMZ (Demilitarized Zone)                │
│  - Web application firewalls (WAF)       │
│  - Reverse proxies (nginx, F5)           │
│  - Public-facing servers                 │
│  - SMTP relay / email gateway            │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│  INTERNAL NETWORK FIREWALL               │
│  - Next-gen firewall (NGFW)              │
│  - East-West traffic inspection          │
│  - Application-layer filtering           │
└──────────────┬──────────────────────────┘
               │
    ┌──────────┴──────────────┐
    ▼                         ▼
┌──────────┐            ┌──────────┐
│  CORP    │            │  SERVER  │
│  VLAN    │            │  VLAN    │
│  10.1.x  │            │  10.10.x │
└──────────┘            └──────────┘
    │                         │
    ▼                         ▼
┌──────────┐            ┌──────────┐
│  GUEST   │            │  DATA    │
│  VLAN    │            │  VLAN    │
│  10.99.x │            │  10.20.x │
└──────────┘            └──────────┘
```

### Security Zones Definition
| Zone | Trust Level | Contents | Inter-zone Policy |
|------|-------------|----------|-------------------|
| Internet | Untrusted (0) | External, unknown | Block all inbound by default |
| DMZ | Low (1) | Public-facing servers, proxies | Limited to specific services only |
| Corporate | Medium (2) | End-user workstations, printers | Restricted to necessary services |
| Server | High (3) | Application/web servers | Explicit allow, deny all else |
| Data | Very High (4) | Databases, file servers | Minimal access, logged |
| Management | Critical (5) | Network devices, hypervisors, AD | Dedicated segment, jump servers only |
| OT/IoT | Isolated | Industrial, building systems | Air-gapped or strict unidirectional |

---

## 2. Firewall Management <a name="firewall"></a>

### Firewall Rule Best Practices
```
Fundamental principles:
1. Default deny: Block everything unless explicitly allowed
2. Least privilege: Only open ports and protocols required
3. Specificity: Rules as specific as possible (not "allow all")
4. Order: Most specific rules first (firewalls process top-down)
5. Documentation: Every rule must have a comment with business justification
6. Review: Quarterly review of all rules, annual cleanup

Rule components:
Source IP/Range | Destination IP/Range | Port/Protocol | Action | Log | Comment

Good rule example:
10.1.0.0/24 → 10.10.50.10 : TCP/443 : ALLOW : YES : "HR portal access from corporate VLAN"

Bad rule examples:
ANY → ANY : ANY : ALLOW  (never acceptable)
ANY → 10.10.0.0/16 : ANY : ALLOW  (too permissive)
10.0.0.0/8 → ANY : TCP/80,443 : ALLOW  (too broad source)
```

### Firewall Ruleset Audit
```
Monthly: Review last 30 days logs
- Rules with 0 hits in 30 days (candidate for removal)
- Rules with excessive hits (investigate if expected)
- Denied traffic patterns (port scans, brute force)

Quarterly: Full ruleset review
- Check each rule for: Owner, business justification, expiry date
- Remove expired temporary rules (tag with expiry date on creation)
- Consolidate overlapping rules
- Verify source/destination still exist (decommissioned servers)

Annual: Complete ruleset cleanup
- Zero-hit rules for 12 months: Remove or document why keep
- Rules without business owner: Reach out or disable/remove
- Shadow rules (rules made redundant by broader rule above)
- Duplicate rules

Firewall rule request template:
- Requestor name and date
- Business justification (not just "we need it")
- Source IP/range
- Destination IP/range
- Ports and protocols
- Direction (inbound/outbound/both)
- Temporary (expiry date) or permanent
- Change management ticket number
```

### Palo Alto Specific Administration
```bash
# Palo Alto CLI useful commands

# View security policy hits
show running security-policy

# Check interface status
show interface all

# View routing table
show routing route

# Monitor traffic logs
show log traffic direction equal forward

# Commit pending changes
commit

# Rollback last commit
debug device-server clear-fib-cache
# OR: in Panorama, use revision history

# Export configuration
scp export configuration from running-config.xml
to <username>@<ip>:<path>

# Check sessions
show session all filter source 10.1.1.100

# High availability status
show high-availability all

# Threat logs
show log threat
```

### Cisco ASA/FTD Administration
```bash
# Cisco ASA key commands

# View access lists
show access-list

# Check connection table
show conn

# View NAT table
show xlate

# Interface status
show interface ip brief

# View routing
show route

# Packet tracer (simulate traffic)
packet-tracer input INSIDE tcp 10.1.1.100 12345 8.8.8.8 443

# Capture traffic
capture CAP interface INSIDE match tcp host 10.1.1.100 any
show capture CAP

# HA status
show failover

# Syslog configuration
logging enable
logging host INSIDE 10.10.1.20
logging trap informational
```

### FortiGate Administration
```bash
# FortiGate CLI commands

# View firewall policies
get firewall policy

# Check interface status
get system interface

# View session table
get system session list

# Monitor real-time traffic
diagnose sniffer packet any 'host 10.1.1.100' 4

# Check routing
get router info routing-table all

# High availability status
get system ha status

# View IPS signature updates
get system fortiguard-service status

# Export config backup
execute backup config ftp <ip> <dir> <user> <pass>
```

---

## 3. Network Segmentation & VLANs <a name="segmentation"></a>

### VLAN Design Standards
```
Recommended VLAN structure:

VLAN 1:   (Default — never use for production traffic)
VLAN 10:  Corporate Users
VLAN 20:  Server Network
VLAN 30:  Database / Data Tier
VLAN 40:  VoIP
VLAN 50:  Printers / Peripherals
VLAN 60:  Wireless Corporate (802.1x authenticated)
VLAN 70:  IoT / Building Systems
VLAN 80:  Security Cameras / Physical Security
VLAN 90:  Point of Sale (PCI scope — isolated)
VLAN 99:  Guest / Quarantine
VLAN 100: Management (OOB — out-of-band management)
VLAN 200: DMZ

IP Scheme (RFC 1918):
10.10.x.x  — Servers (10.10.VLAN.0/24)
10.20.x.x  — User VLANs by site/floor
10.99.x.x  — Guest networks
172.16.x.x — Infrastructure / management

Avoid: 192.168.0.x and 192.168.1.x (used by home routers — VPN conflicts)
```

### Micro-Segmentation
```
Traditional segmentation (per VLAN):
- All servers in VLAN 20 can talk to each other freely
- Only segmented at the VLAN boundary

Micro-segmentation:
- Each application tier in its own segment
- East-west traffic filtered between segments
- Even within a VLAN, traffic is filtered

Implementation options:
1. Hardware firewalls between segments (legacy, expensive)
2. NSX-T or VMware micro-segmentation (software-defined)
3. Zero Trust Network Access (ZTNA) products
4. Host-based firewalls (Windows Firewall, iptables) as last resort

Micro-segmentation use cases:
- PCI-DSS: Isolate cardholder data environment
- Healthcare: Isolate clinical systems from corporate
- Multi-tenant: Isolate customer environments from each other
- Crown jewels: Extra isolation for IP, financial data, R&D
```

### Inter-VLAN Routing Policies
```
Policy: Inter-VLAN traffic denied by default, only specifically allowed flows permitted.

Permitted flows:
FROM            → TO              PORTS           REASON
Corp Users      → Server VLAN     TCP/80,443,8080 Web app access
Corp Users      → Server VLAN     TCP/445         File share (SMB)
Corp Users      → DNS Servers     UDP/53          DNS resolution
Corp Users      → Proxy Server    TCP/3128,8080   Internet access
Server VLAN     → Database VLAN   TCP/1433,3306   App-to-DB
Management VLAN → All VLANs       TCP/22,3389     Admin access
VoIP VLAN       → External        UDP/5060,5061   SIP trunks
Guest VLAN      → Internet ONLY   TCP/80,443      No internal access

Blocked flows:
Guest VLAN      → Corporate: ALL BLOCKED
IoT VLAN        → Corporate: ALL BLOCKED
Printer VLAN    → Internet: BLOCKED (printers should not initiate internet)
Server VLAN     → Internet directly: ROUTE THROUGH PROXY
```

---

## 4. Zero Trust Networking <a name="zerotrust"></a>

### Zero Trust Principles
```
Core tenets:
1. Never trust, always verify — No implicit trust based on network location
2. Least privilege access — Minimum access required for each task
3. Assume breach — Design as if attackers are already inside
4. Verify explicitly — Use all available data points (identity, device, location, behavior)
5. Inspect and log all traffic — East-west, north-south, encrypted

Zero Trust vs. Castle-and-Moat:
Castle-and-moat: Hard perimeter, soft interior
- Once inside the network, relatively free movement
- VPN gives "on-prem" trust
- Fails when perimeter is breached

Zero Trust: No perimeter
- Identity is the new perimeter
- Every access request verified regardless of location
- Continuous validation (not just at login)
```

### Zero Trust Implementation Roadmap
```
Phase 1: Identity (Months 1-3)
- Enforce MFA for all users
- Implement Conditional Access policies
- Privileged Identity Management (PIM/PAM)
- Single Sign-On for all applications

Phase 2: Devices (Months 4-6)
- Enroll all devices in MDM
- Device compliance policies
- Block non-compliant device access
- Certificate-based device authentication

Phase 3: Network (Months 7-12)
- Micro-segmentation of critical systems
- Replace VPN with ZTNA (Zscaler, Cloudflare Access, etc.)
- Inspect all traffic (including TLS)
- Software-defined perimeter

Phase 4: Applications (Months 12-18)
- Application-level access control
- No network-level access — only app-level
- API security gateway
- CASB for SaaS visibility

Phase 5: Data (Months 18-24)
- Data classification
- DLP enforcement
- Information barriers
- Rights management (IRM/AIP)
```

### ZTNA vs. Traditional VPN
| Feature | Traditional VPN | ZTNA |
|---------|----------------|------|
| Trust model | Network-level (once in, trusted) | Per-application, per-session |
| Access granted to | Entire network segment | Specific application only |
| Verification | At connection time only | Continuous |
| Device posture | Limited | Enforced per-session |
| User experience | Full tunnel (slow, heavy) | App-specific (faster) |
| Split tunneling | Optional | Default (only app traffic) |
| Lateral movement | Possible | Prevented by design |
| Examples | Cisco AnyConnect, FortiClient | Zscaler ZPA, Cloudflare Access, Palo Alto Prisma |

---

## 5. VPN & Remote Access <a name="vpn"></a>

### VPN Types Reference
```
Site-to-Site VPN:
- Connects two networks permanently
- Tunnel between two VPN gateways
- Protocols: IPsec (IKEv2), GRE, VXLAN
- Use case: Branch office to HQ, multi-cloud connectivity

Client VPN (Remote Access):
- Individual user connects to corporate network
- Full tunnel: ALL traffic through VPN
- Split tunnel: Only corporate traffic through VPN (recommended)
- Protocols: IKEv2, SSTP, OpenVPN, SSL/TLS
- Use case: Remote employees

Always On VPN (Windows):
- Automatically connects at device boot (device tunnel)
- Automatically connects at user login (user tunnel)
- Replaces DirectAccess
- Requires: Windows 10+ clients, Windows Server 2016+ RAS

Zero Trust VPN (ZTNA):
- No network-level access
- Application-specific tunnels
- Identity + device + context verification per session
```

### VPN Hardening
```
Cipher suite (minimum acceptable):
IKE Phase 1: AES-256, SHA-256, DH Group 14+
IKE Phase 2: AES-256, SHA-256, PFS (Perfect Forward Secrecy)

Deprecated (must not use):
- DES, 3DES encryption
- MD5, SHA-1 hashing
- DH Group 1, 2, 5 (Diffie-Hellman)

Authentication:
- Certificate-based (most secure)
- MFA required for all user VPN (SMS OTP minimum, TOTP preferred)
- No password-only authentication

VPN gateway hardening:
- Disable weak cipher suites
- Disable SSL 2.0/3.0, TLS 1.0/1.1 (TLS 1.2 minimum, TLS 1.3 preferred)
- IP allowlisting for site-to-site (only peer gateway IP allowed)
- Geo-blocking for user VPN (if all users in specific countries)
- Rate limiting on authentication attempts (5 failures = 15 min lockout)

Split tunnel configuration (recommended):
- Route corporate IP ranges through VPN
- Route all internet traffic directly from client
- Benefits: Better performance, less bandwidth at VPN concentrator
- Risk: Corporate-bound malware may use direct internet for C2
- Mitigate: Force DNS through VPN, deploy EDR for visibility
```

---

## 6. DNS Security <a name="dns"></a>

### DNS Security Fundamentals
```
DNS threats:
1. DNS hijacking: Attacker redirects DNS responses
2. DNS cache poisoning: Corrupt resolver cache with false records
3. DNS tunneling: Use DNS as covert C2 channel
4. DGA (Domain Generation Algorithms): Malware generates random domains
5. Fast flux: Rapidly changing IP records to evade detection
6. NXDOMAIN attacks: Flood with queries for non-existent domains

Defenses:
1. DNSSEC: Cryptographic signing of DNS records (prevents poisoning)
2. DNS over HTTPS (DoH) / DNS over TLS (DoT): Encrypts DNS queries
3. Recursive resolver filtering: Block known malicious domains
4. DNS logging: Log all queries for threat hunting
5. Split DNS: Internal resolution for internal names, external for public
```

### DNS Filtering (Protective DNS)
```
Recommended DNS filtering services:
- Cisco Umbrella (enterprise)
- Cloudflare Gateway (enterprise/SMB)
- Palo Alto DNS Security
- Quad9 (free, privacy-focused)
- NextDNS (SMB/consumer)

Filtering categories to block:
- Malware and C2 servers
- Phishing pages
- Newly registered domains (<30 days old) — high risk
- Parking pages and ad networks
- Gambling, adult (adjust per policy)

Implementation:
- Point all corporate DNS resolvers to filtering service
- Enforce via DHCP (default DNS for all clients)
- Redirect all DNS (UDP/TCP 53) to your resolver (prevent bypass)
- Block DoH to external providers (can bypass filtering)
- Monitor for DNS-over-HTTPS abuse

DNS logging best practice:
- Log all DNS queries (source IP, query, response, timestamp)
- Retain 30-90 days for threat hunting
- Alert on:
  * Queries to newly registered domains
  * High-frequency queries to same domain (DGA behavior)
  * DNS queries with unusually large payloads (tunneling)
  * Queries for internal domains going externally (data exfil)
```

### Internal DNS Hardening
```
Windows DNS Server hardening:
1. Restrict zone transfers (only to authorized secondary DNS)
   dnscmd /config /SecureResponses 1

2. Disable recursion on authoritative servers
   dnscmd /config /NoRecursion 1

3. Enable DNS audit logging
   Audit: DNS Server Audit log (Windows Event Log 6001-6010)

4. Use dedicated DNS servers (not on AD domain controllers if possible)

5. Enable DNSSEC signing for internal zones

6. Restrict DNS queries to internal networks only (public DNS should not expose internal zones)

Protect against DNS amplification (if running open resolver):
- Restrict recursive queries to internal clients only
- Rate limiting on responses (DNS RRL - Response Rate Limiting)
- BCP38 ingress filtering to prevent IP spoofing
```

---

## 7. Network Access Control (NAC) <a name="nac"></a>

### NAC Architecture
```
NAC workflow:
1. Device connects to network (wired or wireless)
2. NAC agent or agentless scan evaluates device posture:
   - Is it domain-joined?
   - Is it MDM-enrolled and compliant?
   - Does it have required security software?
   - Is OS patched to minimum level?
3. Based on posture:
   - Compliant: Full network access (corporate VLAN)
   - Remediation needed: Quarantine VLAN (access to patch servers only)
   - Unknown/BYOD: Limited guest access or deny
   - Non-compliant: Block or redirect to remediation portal

NAC products:
- Cisco ISE (enterprise standard)
- Aruba ClearPass
- Forescout
- Microsoft NPS + CA (basic 802.1x)
```

### 802.1x Authentication
```
802.1x components:
- Supplicant: Client device requesting access
- Authenticator: Network switch/AP
- Authentication Server: RADIUS (usually NPS or ISE)

Authentication methods:
- EAP-TLS: Certificate-based (most secure, requires PKI)
- PEAP-MSCHAPv2: Username/password (AD credentials)
- EAP-TTLS: Username/password with TLS tunnel

Switch configuration (Cisco):
interface FastEthernet0/1
  dot1x port-control auto
  authentication port-control auto
  authentication host-mode multi-auth
  authentication order dot1x mab
  authentication priority dot1x mab
  mab
  spanning-tree portfast

# MAB (MAC Authentication Bypass) as fallback for devices without 802.1x (printers, etc.)

RADIUS server setup (Windows NPS):
1. Add network policy server role
2. Register in Active Directory
3. Create RADIUS clients (switches/APs)
4. Create network policy:
   - Conditions: Windows groups (Domain Computers)
   - VLAN assignment via RADIUS attributes
5. Configure connection request policy
```

---

## 8. Intrusion Detection & Prevention <a name="ids-ips"></a>

### IDS vs. IPS
```
IDS (Intrusion Detection System):
- Passive — monitors and alerts
- Does NOT block traffic
- Lower false positive risk
- Forensic and logging value
- Good for: High-traffic links where blocking could disrupt business

IPS (Intrusion Prevention System):
- Active — monitors, alerts, AND blocks
- Can prevent attacks in real-time
- False positives can disrupt legitimate traffic
- Good for: Critical network segments, internet edge

Both deployed via:
- Network tap or SPAN port (inline or out-of-band)
- Next-gen firewalls (integrated IPS — most common today)
- Cloud-based (AWS GuardDuty, Azure Defender for Network)
```

### Snort/Suricata Rules
```
# Suricata rule syntax
# action  proto  src_ip/port   direction  dst_ip/port  (options)

# Detect Mimikatz LSASS dump attempt
alert tcp $HOME_NET any -> $HOME_NET any (msg:"MIMIKATZ LSASS Access"; content:"sekurlsa"; nocase; classtype:credential-theft; sid:1000001; rev:1;)

# Detect DNS tunneling (unusually long DNS queries)
alert dns any any -> any 53 (msg:"Possible DNS Tunneling - Long Query"; dns.query; content:!"."; isdataat:50; classtype:policy-violation; sid:1000002; rev:1;)

# Alert on port scan (10 ports in 1 second)
event_filter gen_id 1, sig_id 1000003, type threshold, track by_src, count 10, seconds 1

# Block known malicious IP
drop ip [1.2.3.4,5.6.7.8] any -> $HOME_NET any (msg:"Known Malicious IP"; classtype:trojan-activity; sid:1000010; rev:1;)
```

### Signature Tuning
```
IPS tuning process (prevent alert fatigue):

Step 1: Deploy in Detection Only mode (1-2 weeks)
Step 2: Review alerts — categorize:
  - True positive: Real threat, should alert/block
  - False positive: Legitimate traffic incorrectly flagged
  - Informational: Worth knowing but not blocking

Step 3: Tune false positives
  - Create exceptions for known-good traffic
  - Adjust signature thresholds
  - Whitelist trusted IP ranges for noisy signatures

Step 4: Enable prevention mode for high-confidence signatures
  - Critical vulnerabilities (CVSSv3 9.0+)
  - Known exploit frameworks (Metasploit, Cobalt Strike)
  - Web application attacks (SQLi, XSS, command injection)

Step 5: Ongoing review
  - Weekly: Review new alerts, tune as needed
  - Monthly: Review false positive rate (target <5%)
  - Quarterly: Update signature packs, review coverage
```

---

## 9. DDoS Protection <a name="ddos"></a>

### DDoS Attack Types
| Type | Layer | Method | Example |
|------|-------|--------|---------|
| Volumetric | 3-4 | Flood bandwidth | UDP flood, ICMP flood, DNS amplification |
| Protocol | 3-4 | Exhaust state tables | SYN flood, Ping of Death |
| Application | 7 | Exhaust server resources | HTTP flood, Slowloris, SSL flood |
| DNS | 7 | Exhaust DNS infrastructure | DNS query flood, NXDOMAIN flood |

### DDoS Mitigation Architecture
```
Layer 1: Upstream scrubbing (cloud-based)
- Cloudflare, Akamai, AWS Shield, Azure DDoS Protection
- Traffic directed to scrubbing center
- Clean traffic forwarded to origin
- Handles volumetric attacks (Tbps scale)
- Cost: $500-5000/month (Cloudflare Business/Enterprise)

Layer 2: ISP-level filtering
- Most ISPs offer basic DDoS filtering
- Blackhole routing for attacked IPs (traffic dropped upstream)
- Limitation: Kills both attack and legitimate traffic

Layer 3: On-premise mitigation
- Rate limiting on firewalls
- SYN cookies to handle SYN floods
- Connection rate limiting
- IP reputation blocking
- Effective for smaller attacks (<10 Gbps)

Always-on vs. On-demand scrubbing:
- Always-on: All traffic routes through scrubbing center (higher cost, always protected)
- On-demand: Only diverted during attack (lower cost, detection lag)
```

---

## 10. Network Monitoring & Logging <a name="monitoring"></a>

### Network Flow Analysis
```
NetFlow / sFlow / IPFIX:
- Collects metadata about network conversations (not payload)
- Source/destination IP, port, protocol, byte count, duration
- Stored in flow collector (Elastic, Splunk, ntopng)
- No packet capture overhead — can monitor all traffic

NetFlow configuration (Cisco):
interface GigabitEthernet0/1
  ip flow ingress
  ip flow egress

ip flow-export destination 10.10.1.50 9996
ip flow-export version 9
ip flow-cache timeout active 1
ip flow-cache timeout inactive 15

# For entire router
ip flow-export source Loopback0

Flow analysis use cases:
- Top talkers (who is generating most traffic?)
- Bandwidth by application (what's using the pipe?)
- Anomaly detection (sudden spike = ransomware / exfiltration?)
- Network baselining (what is normal?)
- Incident investigation (who talked to C2 server on port 4444?)
```

### Network Logging Standards
```
What to log:
- Firewall: All denied traffic, all allowed traffic to critical servers
- VPN: All connections (user, source IP, duration, bytes)
- DNS: All queries (source, query, response)
- DHCP: All leases (MAC, IP, hostname, timestamp)
- Authentication: All 802.1x attempts (success and failure)
- Switches: AAA events, port status changes, spanning tree events

Log retention:
- Firewall logs: 90 days minimum (1 year for compliance)
- DNS logs: 30 days minimum
- VPN logs: 1 year
- Authentication logs: 1 year

Centralized logging architecture:
Devices → Syslog → SIEM (Splunk/Sentinel/QRadar)
                  → Log archive (cold storage, compliance)

Syslog server setup (Linux/rsyslog):
# /etc/rsyslog.d/network.conf
$ModLoad imudp
$UDPServerRun 514
$ModLoad imtcp
$InputTCPServerRun 514

template(name="NetworkLogs" type="string" string="/var/log/network/%HOSTNAME%/%$YEAR%/%$MONTH%/%$DAY%/syslog.log")
*.* ?NetworkLogs
```

---

## 11. Wireless Security <a name="wireless"></a>

### Enterprise Wireless Security Standards
```
Authentication:
- WPA3-Enterprise: Current gold standard (802.1x with WPA3)
- WPA2-Enterprise: Acceptable (802.1x, AES-CCMP)
- WPA2-Personal (PSK): Only for guest/IoT (shared key = shared risk)
- Never: WEP, WPA (TKIP) — completely broken

SSID design:
CORP-WIRELESS   → WPA3-Enterprise, 802.1x, corporate VLAN
VOIP-WIRELESS   → WPA2-Enterprise, dedicated VoIP VLAN
IOT-WIRELESS    → WPA2-PSK (complex), isolated IoT VLAN
GUEST           → WPA2-PSK (rotated monthly), internet-only

Wireless security features to enable:
- PMF (Protected Management Frames) — prevents deauth attacks
- OWE (Opportunistic Wireless Encryption) — encrypts open networks
- 802.11w — management frame protection
- SSID isolation — clients can't see each other on same SSID
- Band steering — encourage clients to 5GHz / 6GHz
- Fast BSS Transition (802.11r) — faster roaming for VoIP

Rogue AP detection:
- Enable WIDS/WIPS (Wireless Intrusion Detection/Prevention)
- All APs scan for unauthorized SSIDs
- Alert on: SSIDs with similar name to corporate, unrecognized BSSIDs
- Physical walkthrough with spectrum analyzer quarterly
```

### Wireless Hardening Checklist
```
Access Point Hardening:
[ ] Change default admin passwords (use complex + store in PAM)
[ ] Disable Telnet, HTTP management (use SSH + HTTPS only)
[ ] Enable management VLAN access restriction (only from management network)
[ ] Disable unused radios (if site only needs 5GHz, disable 2.4GHz)
[ ] Enable AP certificate for controller communication
[ ] Keep firmware updated (subscribe to vendor security advisories)
[ ] Disable WPS (Wi-Fi Protected Setup) — easily brute-forced
[ ] Disable SSID broadcast for sensitive internal SSIDs (minimal security benefit but reduces noise)
[ ] Set inactivity timeout for management sessions (10 minutes)
[ ] Log all management authentication events

Controller Security:
[ ] Multi-factor authentication for controller access
[ ] Role-based access (read-only for NOC, admin for network team)
[ ] Configuration backup daily
[ ] Audit log for all configuration changes
```

---

## 12. Network Security Hardening <a name="hardening"></a>

### Switch Hardening (Cisco)
```
! Port security
interface range FastEthernet0/1 - 24
  switchport mode access
  switchport nonegotiate
  switchport access vlan 10
  spanning-tree portfast
  spanning-tree bpduguard enable  ! Protect against rogue switches
  storm-control broadcast level 20 ! Limit broadcast floods
  ip dhcp snooping limit rate 15  ! Protect against DHCP exhaustion
  ip arp inspection limit rate 100 ! Protect against ARP spoofing
  shutdown                         ! Disable all unused ports

! Disable unused services
no cdp run                         ! Disable CDP globally (or per interface)
no lldp run
no service tcp-small-servers
no service udp-small-servers
no ip http server
no ip http secure-server

! DHCP snooping (prevents rogue DHCP servers)
ip dhcp snooping
ip dhcp snooping vlan 10,20,30
no ip dhcp snooping information option

! Dynamic ARP inspection
ip arp inspection vlan 10,20,30

! Management access
line vty 0 4
  transport input ssh
  exec-timeout 10 0
  access-class MGMT-ACCESS in
  logging synchronous

ip access-list standard MGMT-ACCESS
  permit 10.100.0.0 0.0.0.255      ! Management VLAN only
  deny   any log

! Enable login security
login block-for 300 attempts 5 within 60
```

### Router Hardening
```
! Disable unnecessary services
no service finger
no service pad
no service tcp-small-servers
no service udp-small-servers
no ip source-route           ! Prevent IP source routing attacks
no ip proxy-arp
no ip directed-broadcast     ! Prevent smurf attacks

! uRPF (Unicast Reverse Path Forwarding) — anti-spoofing
interface GigabitEthernet0/0
  ip verify unicast source reachable-via rx

! BCP38 ingress filtering on edge
ip access-list extended BOGON-FILTER
  deny ip 10.0.0.0 0.255.255.255 any     ! RFC 1918
  deny ip 172.16.0.0 0.15.255.255 any    ! RFC 1918
  deny ip 192.168.0.0 0.0.255.255 any    ! RFC 1918
  deny ip 127.0.0.0 0.255.255.255 any    ! Loopback
  deny ip 0.0.0.0 0.255.255.255 any      ! This network
  deny ip 224.0.0.0 15.255.255.255 any   ! Multicast
  deny ip 240.0.0.0 15.255.255.255 any   ! Reserved
  permit ip any any

interface GigabitEthernet0/0    ! WAN interface
  ip access-group BOGON-FILTER in
```

---

## 13. Incident Response for Network Events <a name="incident"></a>

### Network Incident Playbooks

#### Suspected Network Intrusion
```
DETECTION:
- IDS/IPS alert on known attack pattern
- Unusual traffic volume (NetFlow anomaly)
- Security alert from endpoint (beacon to known C2)
- User reports network slowness

TRIAGE (0-15 minutes):
1. Identify affected host(s) from logs
2. Identify C2 or attack IP from alerts
3. Assess scope: Is this isolated or widespread?
4. Escalate to security team / SOC

CONTAINMENT (15-60 minutes):
1. Block C2 IP at firewall immediately
   fw block 1.2.3.4
2. Isolate affected host(s) at switch level (VLAN change or port shutdown)
3. Block any malicious domain at DNS
4. Preserve logs before any system changes

INVESTIGATION (1-4 hours):
1. Collect packet captures around time of incident
2. Pull all firewall logs for affected IPs
3. Review DNS logs for malicious domain queries
4. Pull NetFlow data to identify scope of communication
5. Coordinate with endpoint team for host forensics

ERADICATION:
1. Remove compromised hosts from production
2. Verify all C2 indicators are blocked
3. Scan environment for other infected hosts

RECOVERY:
1. Rebuild compromised hosts from clean image
2. Monitor re-deployed hosts closely
3. Verify no reinfection
```

#### DDoS Attack Response
```
DETECTION:
- ISP alerts high traffic volume
- Internet-facing services unreachable
- Firewall CPU at 100%
- Unusual packet flood in NetFlow

IMMEDIATE (0-10 minutes):
1. Confirm DDoS vs. legitimate traffic spike
2. Identify attack type (volumetric, protocol, application layer)
3. Activate DDoS mitigation service (Cloudflare, ISP scrubbing)
4. Notify management

MITIGATION:
Volumetric attack:
- ISP blackhole routing (sacrifices IP, stops attack upstream)
- Cloudflare/scrubbing center activation (keeps service running)
- Null routing on border router: ip route [attacked IP] null0

Protocol attack:
- Enable SYN cookies on servers
- Block ICMP floods
- Firewall rate limiting

Application layer:
- WAF rules to block attack patterns
- Challenge pages (CAPTCHA) for all visitors
- Rate limiting by IP

COMMUNICATION:
- Internal: Notify IT leadership, NOC
- External: If customers impacted, prepare status page update
- ISP: Engage DDoS mitigation support line

POST-INCIDENT:
- Document attack characteristics (duration, volume, type, source IPs)
- Review mitigation effectiveness
- Adjust protection for similar future attacks
```

---

*Last Updated: 2025 | Network Security & Firewall Management Guide v2.0*
