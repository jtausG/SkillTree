# VoIP & Telephony Operations Guide

## Table of Contents
1. [VoIP Fundamentals](#voip-fundamentals)
2. [SIP Protocol Deep Dive](#sip-protocol)
3. [PBX Systems](#pbx-systems)
4. [Microsoft Teams Phone](#teams-phone)
5. [Cisco Unified Communications](#cisco-uc)
6. [Network Requirements for VoIP](#network-requirements)
7. [Call Quality Management](#call-quality)
8. [Troubleshooting VoIP Issues](#troubleshooting)
9. [Security Considerations](#security)
10. [DR and Redundancy](#dr-redundancy)

---

## 1. VoIP Fundamentals

### How VoIP Works
Voice over IP converts analog audio signals into digital data packets transmitted over IP networks.

**Signal Flow:**
```
Microphone → ADC → Codec (compress) → RTP Packets → Network → 
Jitter Buffer → Codec (decompress) → DAC → Speaker
```

### Key Protocols

| Protocol | Port | Purpose |
|----------|------|---------|
| SIP | 5060 (TCP/UDP), 5061 (TLS) | Session initiation/signaling |
| RTP | 16384–32767 (UDP) | Media transport |
| RTCP | RTP port +1 | Quality statistics |
| SRTP | Same as RTP | Encrypted media |
| H.323 | 1720 TCP | Legacy signaling |
| MGCP | 2427/2727 UDP | Gateway control |
| SCCP (Skinny) | 2000/2443 TCP | Cisco proprietary |

### Codecs Comparison

| Codec | Bandwidth | Quality | Use Case |
|-------|-----------|---------|----------|
| G.711 | 64 kbps | Excellent | LAN/internal calls |
| G.729 | 8 kbps | Good | WAN/low bandwidth |
| G.722 | 64 kbps | HD (wideband) | High-quality internal |
| G.726 | 16–40 kbps | Good | Variable bandwidth |
| Opus | 6–510 kbps | Excellent | Teams/WebRTC |
| iLBC | 15.2/13.3 kbps | Good | Packet loss resilient |

**Bandwidth calculation (G.711):**
```
Payload: 160 bytes (20ms)
Headers: 40 bytes (IP+UDP+RTP)
Total per packet: 200 bytes × 50 pps = 80 kbps per call
```

---

## 2. SIP Protocol Deep Dive

### SIP Message Types

**Requests:**
- `INVITE` — Initiate a session
- `ACK` — Acknowledge INVITE response
- `BYE` — Terminate a session
- `CANCEL` — Cancel pending request
- `REGISTER` — Register UA with registrar
- `OPTIONS` — Query capabilities
- `SUBSCRIBE` — Subscribe to event notifications
- `NOTIFY` — Send event notification
- `REFER` — Transfer a call
- `MESSAGE` — Instant messaging

**Response Codes:**
```
1xx — Provisional (100 Trying, 180 Ringing, 183 Session Progress)
2xx — Success (200 OK, 202 Accepted)
3xx — Redirection (301 Moved Permanently, 302 Moved Temporarily)
4xx — Client Error (400 Bad Request, 401 Unauthorized, 403 Forbidden, 
                     404 Not Found, 408 Timeout, 486 Busy Here, 487 Request Terminated)
5xx — Server Error (500 Internal, 503 Unavailable)
6xx — Global Failure (600 Busy Everywhere, 603 Decline)
```

### Basic Call Flow (SIP Trapezoid)
```
    UA-A (Alice)                           UA-B (Bob)
        |                                       |
        |---INVITE SDP offer------------------->|
        |<--100 Trying--------------------------|
        |<--180 Ringing-------------------------|
        |<--200 OK SDP answer-------------------|
        |---ACK---------------------------------|
        |<======= RTP Media =================>  |
        |---BYE---------------------------------|
        |<--200 OK------------------------------|
```

### SIP Trunk Configuration Checklist
```
□ SIP trunk authentication (IP-based vs. username/password)
□ Codec negotiation order configured
□ DTMF method: RFC 2833 / SIP INFO / In-band
□ SIP OPTIONS keepalives enabled
□ T.38 fax passthrough configured (if needed)
□ Outbound caller ID / P-Asserted-Identity header
□ Inbound DID routing rules
□ Emergency (E911) routing configured
□ Failover trunk configured
□ Max concurrent calls limit set
```

---

## 3. PBX Systems

### On-Premises PBX Options

**Cisco Call Manager (CUCM):**
- Enterprise-grade, highly scalable
- Supports SCCP and SIP endpoints
- Integrates with Unity Connection (voicemail), UCCX (contact center)
- High availability: Publisher + Subscribers cluster

**3CX:**
- Software-based, Windows or Linux
- Hybrid cloud capable
- Built-in WebRTC client
- SIP trunk agnostic

**FreePBX/Asterisk:**
- Open source
- Highly customizable
- Community support
- Good for small-medium businesses

**Avaya:**
- Legacy enterprise
- Aura platform
- Strong contact center capabilities

### Key PBX Concepts

**Dial Plan Components:**
```
Extension ranges:     1xx, 2xxx, 3xxx
Outside access code:  9 (traditional) or just dial
Long distance:        1 + NPA + NXX + XXXX
International:        011 + country code + number
Emergency:            911 (direct, no delay)
```

**Ring Groups vs. Hunt Groups vs. Queues:**
- **Ring Group** — All phones ring simultaneously; first to answer gets call
- **Hunt Group** — Sequential ringing (linear or round-robin)
- **Call Queue** — Caller waits with hold music; agent availability tracked

**IVR/Auto-Attendant Best Practices:**
```
- Keep menu options to 4 or fewer
- State option BEFORE key ("For Sales, press 1" not "Press 1 for Sales")
- Always offer 0 to reach operator
- Record professional prompts (avoid TTS for main greetings)
- Test all paths including invalid input and timeout
- Log call volumes per option for regular optimization
```

---

## 4. Microsoft Teams Phone

### Architecture Overview
```
PSTN Options:
├── Calling Plan (Microsoft-hosted PSTN)
├── Direct Routing (bring your own SBC/trunk)
├── Operator Connect (carrier-managed SBC)
└── Teams Phone Mobile (carrier SIM integration)
```

### Direct Routing Setup

**Requirements:**
- Teams Phone System license (included in E5, add-on for E3)
- Session Border Controller (SBC) certified by Microsoft
- Public IP + FQDN for SBC
- Validated TLS certificate on SBC

**PowerShell Configuration:**
```powershell
# Connect to Teams
Connect-MicrosoftTeams

# Create SBC
New-CsOnlinePSTNGateway `
  -Fqdn "sbc.contoso.com" `
  -SipSignalingPort 5067 `
  -ForwardCallHistory $true `
  -ForwardPAI $true `
  -SendSipOptions $true `
  -MaxConcurrentSessions 100 `
  -Enabled $true

# Create Voice Route
New-CsOnlineVoiceRoute `
  -Identity "US-Route" `
  -NumberPattern "^\+1(\d{10})$" `
  -OnlinePstnGatewayList "sbc.contoso.com" `
  -Priority 1

# Create PSTN Usage
Set-CsOnlinePstnUsage -Usage @{Add="US"}

# Create Voice Routing Policy
New-CsOnlineVoiceRoutingPolicy `
  -Identity "US-VRP" `
  -OnlinePstnUsages "US"

# Assign to user
Grant-CsOnlineVoiceRoutingPolicy `
  -Identity "user@contoso.com" `
  -PolicyName "US-VRP"

# Enable user for Enterprise Voice
Set-CsPhoneNumberAssignment `
  -Identity "user@contoso.com" `
  -PhoneNumber "+12065551234" `
  -PhoneNumberType DirectRouting
```

### Teams Phone Dial Plan
```powershell
# Normalization rules convert local numbers to E.164
New-CsVoiceNormalizationRule `
  -Parent "US-DialPlan" `
  -Name "10-digit-local" `
  -Pattern "^(\d{10})$" `
  -Translation "+1$1" `
  -InternalExtension $false

New-CsVoiceNormalizationRule `
  -Parent "US-DialPlan" `
  -Name "5-digit-extension" `
  -Pattern "^(1\d{4})$" `
  -Translation "+1425555$1"
```

### Common Teams Phone Issues

| Symptom | Likely Cause | Resolution |
|---------|-------------|------------|
| Can't make PSTN calls | No voice routing policy | Assign VRP to user |
| One-way audio | NAT/firewall blocking RTP | Open UDP 3478-3481, 50000-50059 |
| Call drops at 30 sec | SIP OPTIONS timeout | Configure SBC keepalives |
| Poor call quality | Insufficient bandwidth | Enable QoS, prioritize Teams traffic |
| No dial tone from device | SBC certificate expired | Renew and re-upload cert |

---

## 5. Cisco Unified Communications

### CUCM Administration

**Device Registration States:**
- **Registered** — Active, ready to receive calls
- **Unregistered** — Was registered, now unavailable
- **Unknown** — Never registered or no status
- **Rejected** — Auth failure

**Common CUCM Phone Issues:**
```
"Registering" stuck:
  → Check IP connectivity to CUCM
  → Verify DHCP option 150 (TFTP server IP)
  → Check TFTP service running on CUCM
  → Verify correct device pool/CUCM group

"Updating Configuration":
  → TFTP file mismatch
  → Check SEP{MAC}.cnf.xml file on TFTP
  → Restart TFTP service if needed

CTI Route Points not working:
  → Verify application user has CTI permissions
  → Check device is associated with app user
```

**Useful CUCM CLI Commands:**
```bash
# Check service status
utils service list

# Restart service
utils service restart Cisco CallManager

# Check DB replication
utils dbreplication status

# Network test
utils network ping <ip>

# Collect logs
file get activelog cm/trace/ccm/sdl/*.txt
```

---

## 6. Network Requirements for VoIP

### QoS Configuration

**DSCP Marking:**
```
Voice (RTP):     EF (46) / DSCP 101110
Signaling (SIP): CS3 (24) / DSCP 011000
Video:           AF41 (34) / DSCP 100010
Data:            Default (0) / Best effort
```

**Cisco Switch QoS for VoIP:**
```
! Auto QoS on access port
interface GigabitEthernet1/0/1
 switchport mode access
 switchport access vlan 10
 switchport voice vlan 20
 mls qos trust cos
 auto qos voip cisco-phone
 spanning-tree portfast

! Manual DSCP marking
class-map match-all VOICE-RTP
 match ip dscp ef

policy-map VOIP-QOS
 class VOICE-RTP
  priority percent 30
 class class-default
  fair-queue

interface GigabitEthernet0/0
 service-policy output VOIP-QOS
```

### Firewall Rules for VoIP
```
# SIP Signaling
TCP/UDP 5060 — SIP (unencrypted)
TCP/UDP 5061 — SIP-TLS
TCP 2000    — SCCP (Cisco)

# Media
UDP 10000-20000 — RTP (adjust per vendor)
UDP 16384-32767 — RTP (Cisco default)
UDP 3478-3481   — STUN/TURN (Teams)
UDP 50000-50059 — Teams media

# Management
TCP 443        — CUCM Admin, Teams
TCP 8443       — CUCM Admin (alt)
```

### VLAN Segmentation
```
Voice VLAN design:
- Separate voice VLAN from data VLAN
- Voice VLAN: 172.16.20.0/24 (example)
- Data VLAN: 192.168.10.0/24 (example)
- Enable CDP/LLDP-MED on access switches
  (phones auto-detect voice VLAN)
- DHCP option 150 on voice VLAN scope
- ACL: Voice VLAN → CUCM/SBC only, restrict lateral movement
```

---

## 7. Call Quality Management

### MOS Score Reference

| MOS Score | Quality | User Perception |
|-----------|---------|----------------|
| 4.3–5.0 | Excellent | Very satisfied |
| 4.0–4.3 | Good | Satisfied |
| 3.6–4.0 | Fair | Slightly annoyed |
| 3.1–3.6 | Poor | Annoyed |
| 2.6–3.1 | Bad | Very annoyed |
| < 2.6 | Terrible | Unacceptable |

### Quality Metrics

**Latency (One-way):**
- < 150 ms: Excellent
- 150–300 ms: Acceptable
- > 300 ms: Unacceptable

**Jitter:**
- < 30 ms: Good
- 30–50 ms: Marginal
- > 50 ms: Poor

**Packet Loss:**
- < 1%: Good
- 1–5%: Marginal
- > 5%: Unacceptable

### Diagnosing Call Quality

```powershell
# Test network path quality (Windows)
pathping -n -q 100 <sbc-ip>

# Continuous ping with timestamp
ping -t <ip> | foreach { "$(Get-Date -Format HH:mm:ss) $_" }
```

```bash
# Linux: test UDP packet loss to RTP port range
hping3 -2 -p 5060 -c 1000 <sip-server>

# Capture RTP stream
tcpdump -i eth0 udp portrange 10000-20000 -w voip_capture.pcap

# Analyze with tshark
tshark -r voip_capture.pcap -q -z rtp,streams
```

**Common Quality Issues and Causes:**

| Issue | Symptom | Root Cause |
|-------|---------|------------|
| Echo | Hear yourself | Long delay + sidetone; acoustic echo |
| Choppy/robotic | Intermittent cuts | Jitter/packet loss |
| One-way audio | Can't hear remote | NAT traversal failure |
| Delay | Noticeable lag | High latency, wrong codec |
| Static/noise | Background noise | Codec mismatch, bad codec negotiation |
| Call drops | Disconnects mid-call | SIP re-INVITE failure, keepalive timeout |

---

## 8. Troubleshooting VoIP Issues

### Systematic Approach

```
Step 1: Gather info
  - Affected users (one person, site, all)?
  - Inbound, outbound, or internal calls?
  - Error message / tone / silence?
  - When did it start?
  - Any recent changes?

Step 2: Reproduce the issue
  - Test call with known-good endpoint
  - Test with softphone (eliminates hardware)
  - Test from different network location

Step 3: Check infrastructure
  - SBC/PBX service status
  - SIP trunk registration status
  - Network connectivity PBX ↔ carrier

Step 4: Capture and analyze
  - SIP/RTP packet capture
  - PBX call logs
  - Network QoS stats
```

### SIP Capture Analysis

```bash
# Capture SIP traffic
tcpdump -i eth0 -w sip_capture.pcap port 5060 or port 5061

# View SIP messages in Wireshark
# Filter: sip
# Follow SIP call: Telephony > VoIP Calls > select call > Flow Sequence

# tshark SIP analysis
tshark -r capture.pcap -Y sip -T fields \
  -e frame.time -e ip.src -e ip.dst \
  -e sip.Method -e sip.Status-Code -e sip.r-uri
```

### Registration Failures

```
401 Unauthorized:
  → Wrong password / authentication credentials
  → Realm mismatch between phone and PBX

403 Forbidden:
  → IP not whitelisted on SIP trunk
  → Account disabled or suspended

404 Not Found:
  → Wrong SIP domain / proxy address
  → Extension doesn't exist on PBX

503 Service Unavailable:
  → PBX overloaded or down
  → SIP trunk carrier issue
  → Check carrier status page

No response / timeout:
  → Firewall blocking SIP port
  → Wrong SIP server IP/FQDN
  → DNS resolution failure
```

---

## 9. Security Considerations

### VoIP Threats

| Threat | Description | Mitigation |
|--------|-------------|-----------|
| Toll Fraud | Unauthorized calls billed to you | Limit international calling; alerts on high spend |
| Eavesdropping | Capturing unencrypted calls | Use SRTP + TLS (SIPS) |
| DoS/DDoS | Flood SIP server | Rate limiting, SBC with DoS protection |
| SIP Scanning | Brute-force extension discovery | SBC IP whitelisting, fail2ban |
| Vishing | Voice phishing attacks | Caller ID validation, user training |
| SPIT | Spam over Internet Telephony | Call filtering, CAPTCHA for unknown callers |

### Hardening Checklist
```
□ Enable SRTP for media encryption
□ Enable TLS/SIPS for signaling
□ SBC in DMZ, not directly on internet
□ Whitelist carrier SIP IPs; block all others
□ Disable international calling for users who don't need it
□ Set concurrent call limits per trunk and user
□ Enable failed registration alerting
□ Disable unused codecs and features
□ Change default admin passwords on PBX/SBC
□ Apply vendor security patches promptly
□ Monitor call detail records for anomalies
□ Geo-block high-risk countries if not needed
□ Enable call recording for compliance (where legal)
```

---

## 10. DR and Redundancy

### High Availability Options

**Survivable Remote Site Telephony (SRST):**
```
- Cisco router acts as fallback call manager
- Phones register to local router if WAN fails
- Configured on ISR/ASR router:

telephony-service
 max-ephones 50
 max-dn 100
 ip source-address 10.1.1.1 port 2000
 auto assign 1 to 100
 system message SRST Mode
```

**SBC Redundancy:**
```
Active/Standby SBC pair:
- Shared virtual IP (VIP) using VRRP/HSRP
- SIP trunk registered to VIP
- Failover < 2 seconds

Active/Active SBC:
- DNS round-robin or SRV records
- Load balanced across both SBCs
- Session continuity on failover
```

### DR Checklist for Telephony
```
□ Document all DIDs and routing rules
□ Configure PSTN failover (mobile backup numbers)
□ Test failover monthly (after-hours)
□ Maintain SIP trunk capacity headroom (>20%)
□ Emergency contact tree documented and current
□ E911 routing tested from all locations
□ Voicemail DR: backup configuration exported
□ IVR prompts backed up and version-controlled
□ Carrier support contacts and account numbers documented
□ RTO/RPO defined for telephony platform
```

---

## Appendix: VoIP Quick Reference

### Common Ports Summary
```
SIP:          5060 UDP/TCP, 5061 TLS
RTP:          16384-32767 UDP (dynamic)
SCCP:         2000 TCP, 2443 TLS
H.323:        1720 TCP
MGCP:         2427 UDP
Teams Media:  3478-3481 UDP, 50000-50059 UDP
Zoom:         8801-8802 UDP
WebRTC:       10000-60000 UDP (browser)
```

### E.164 Number Formatting
```
Format:  +[country code][subscriber number]
US:      +1 (NPA)(NXX)(XXXX)  → +12065551234
UK:      +44 (area)(number)   → +441614960000
DE:      +49 (ortsvorwahl)(number) → +4930123456
```
