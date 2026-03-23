# Network Connectivity Troubleshooting

> **Category:** Troubleshooting | **Audience:** L1/L2/L3 Support | **Scope:** LAN, WAN, Wi-Fi, VPN

---

## Table of Contents
1. [Quick Triage Flowchart](#1-quick-triage-flowchart)
2. [Physical Layer (Layer 1)](#2-physical-layer-layer-1)
3. [IP Addressing Issues (Layer 3)](#3-ip-addressing-issues-layer-3)
4. [DNS Troubleshooting](#4-dns-troubleshooting)
5. [DHCP Troubleshooting](#5-dhcp-troubleshooting)
6. [Routing & Default Gateway](#6-routing--default-gateway)
7. [Firewall & ACL Issues](#7-firewall--acl-issues)
8. [Wi-Fi Troubleshooting](#8-wi-fi-troubleshooting)
9. [VPN Troubleshooting](#9-vpn-troubleshooting)
10. [Site-Wide Outages](#10-site-wide-outages)
11. [Performance & Latency Issues](#11-performance--latency-issues)
12. [Packet Capture with Wireshark](#12-packet-capture-with-wireshark)

---

## 1. Quick Triage Flowchart

```
User reports "no internet" / "can't connect to X"
                    │
          ┌─────────▼──────────┐
          │ Can you ping your  │
          │ default gateway?   │
          └─────────┬──────────┘
          NO        │        YES
    ┌─────▼────┐    │    ┌────▼──────────────────┐
    │Physical/ │    │    │Can you ping 8.8.8.8?  │
    │DHCP issue│    │    └────┬──────────────────┘
    └──────────┘    │    NO   │      YES
                    │  ┌──────▼─┐  ┌──────▼────────────┐
                    │  │Routing │  │Does DNS resolve?  │
                    │  │/WAN    │  └────┬──────────────┘
                    │  │issue   │  NO   │     YES
                    │  └────────┘ ┌────▼─┐ ┌──────▼──────────┐
                    │             │ DNS  │ │Port/firewall or │
                    │             │issue │ │application issue│
                    │             └──────┘ └─────────────────┘
```

---

## 2. Physical Layer (Layer 1)

### Wired Connection Checks
- [ ] **Cable seated fully** — click heard when inserting RJ45
- [ ] **Link lights on switch and NIC** — link (green/amber), activity (blinking)
- [ ] **Try a different cable** — cable tester preferred
- [ ] **Try a different switch port** — port could be bad or in wrong VLAN
- [ ] **Check NIC status** in Device Manager — any error codes?
- [ ] **Check duplex/speed mismatch** — auto-negotiate should match on both ends

```powershell
# Windows: Check NIC link status
Get-NetAdapter | Select-Object Name, Status, LinkSpeed, MediaType, MacAddress

# Check NIC statistics (errors = bad cable or duplex mismatch)
Get-NetAdapterStatistics | Select-Object Name, ReceivedErrors, OutboundErrors, ReceivedDiscardedPackets

# Linux
ip link show
ethtool eth0 | grep -E "Speed|Duplex|Link"
```

### Switch Port Investigation
```
# Cisco IOS
show interfaces GigabitEthernet0/1
show interfaces GigabitEthernet0/1 counters errors  # CRC errors = bad cable
show mac address-table interface GigabitEthernet0/1
show spanning-tree interface GigabitEthernet0/1
```

---

## 3. IP Addressing Issues (Layer 3)

### Diagnosing IP Problems
```powershell
# Windows
ipconfig /all
# Key fields:
# - IPv4 Address      (169.254.x.x = APIPA = DHCP failure)
# - Subnet Mask       (wrong mask = can't reach other hosts)
# - Default Gateway   (missing = no routing beyond local subnet)
# - DNS Servers       (missing or wrong = name resolution failure)
# - DHCP Enabled      (No = static IP, check if intentional)
# - Lease Obtained/Expires (expired = DHCP problem)

# Linux / macOS
ip addr show
ip route show
cat /etc/resolv.conf
```

### APIPA Address (169.254.x.x)
**Cause:** DHCP server unreachable  
**Fix:**
1. Check physical connection (cable/switch)
2. Check DHCP server is running
3. Check DHCP relay agent if DHCP is in different subnet
4. `ipconfig /release` then `ipconfig /renew`

### Duplicate IP Address
```powershell
# Symptoms: frequent disconnections, arping errors
# On Windows, Event Viewer shows Event ID 4199 in System log
Get-WinEvent -FilterHashtable @{LogName='System'; Id=4199} -MaxEvents 5

# Find the MAC of the conflicting IP
arp -a | findstr <IP>

# On Linux
arping -I eth0 <IP>  # Shows duplicate if another host responds
```

### Static IP Misconfiguration
```powershell
# Verify IP is correct and not conflicting
ping <static IP>  # From another host

# Set IP via PowerShell
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 10.10.1.50 -PrefixLength 24 -DefaultGateway 10.10.1.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 10.10.1.10,10.10.1.11

# Or GUI: ncpa.cpl → right-click adapter → Properties → IPv4
```

---

## 4. DNS Troubleshooting

### Quick DNS Tests
```powershell
# Test DNS resolution
nslookup google.com                  # Uses default DNS server
nslookup google.com 8.8.8.8          # Use specific DNS server
nslookup google.com 10.10.1.10       # Use internal DNS

# PowerShell
Resolve-DnsName google.com
Resolve-DnsName google.com -Server 10.10.1.10
Resolve-DnsName google.com -Type MX   # Query MX records

# Linux / macOS
dig google.com
dig @10.10.1.10 google.com
dig google.com MX
host google.com
```

### Common DNS Issues

#### Can ping IP but not hostname
**Cause:** DNS resolution failure  
```powershell
# Check which DNS server is being used
Get-DnsClientServerAddress

# Test the DNS server
nslookup hostname 10.10.1.10  # Replace with your DNS server IP

# If DNS returns NXDOMAIN for internal name:
# - Check if record exists in DNS
# - Check if client is using correct DNS server (internal, not 8.8.8.8)
# - Check DNS suffix search list
ipconfig /all  # Look for "DNS Suffix Search List"
```

#### DNS returns wrong IP (DNS poisoning or stale cache)
```powershell
# Flush DNS cache
ipconfig /flushdns

# Linux
sudo systemd-resolve --flush-caches
# or
sudo /etc/init.d/nscd restart

# Clear DNS on Windows Server
Clear-DnsServerCache

# View DNS client cache
ipconfig /displaydns
Get-DnsClientCache
```

#### Internal DNS not resolving external names (DNS forwarders)
```powershell
# Check forwarders on DNS server
Get-DnsServerForwarder

# If missing, add Google/Cloudflare as conditional forwarder or general forwarder
Add-DnsServerForwarder -IPAddress 8.8.8.8, 1.1.1.1

# Check root hints
Get-DnsServerRootHint
```

#### PTR Record Issues (Reverse DNS)
```powershell
# Test reverse lookup
nslookup 10.10.1.50
Resolve-DnsName 10.10.1.50 -Type PTR

# Check reverse lookup zone exists
Get-DnsServerZone | Where-Object IsReverseLookupZone -eq $true
```

---

## 5. DHCP Troubleshooting

```powershell
# Release and renew IP
ipconfig /release
ipconfig /renew

# Check DHCP server logs (Windows DHCP Server)
# C:\Windows\System32\dhcp\DhcpSrvLog-Mon.log (etc. by day)

# PowerShell on DHCP server
Get-DhcpServerv4Scope
Get-DhcpServerv4Lease -ScopeId 10.10.1.0   # List all leases
Get-DhcpServerv4Statistics                   # Pool utilization
Get-DhcpServerv4FreeIPAddress -ScopeId 10.10.1.0 -NumAddress 5  # Check free IPs

# Find lease by MAC
Get-DhcpServerv4Lease -ScopeId 10.10.1.0 | Where-Object ClientId -eq "aa-bb-cc-dd-ee-ff"

# Check scope is not exhausted
Get-DhcpServerv4ScopeStatistics -ScopeId 10.10.1.0
# If PercentageInUse near 100%: expand scope or reduce lease time
```

### DHCP Relay Agent Issues
```
# Symptoms: Clients in remote subnet getting APIPA or wrong scope
# Check: ip helper-address on router interface facing client subnet
# Cisco: show ip helper-address
# Verify UDP port 67/68 not blocked between client subnet and DHCP server
```

---

## 6. Routing & Default Gateway

```powershell
# View routing table
route print        # Windows
ip route show      # Linux
netstat -rn        # Both

# Test gateway reachability
ping 10.10.1.1     # Default gateway IP
arp -a | findstr 10.10.1.1  # Should show gateway MAC

# Trace route to destination
tracert 8.8.8.8    # Windows
traceroute 8.8.8.8  # Linux

# Interpreting traceroute:
# * * *  = No response (may be ICMP blocked — not always a problem)
# High RTT at a hop = latency at that device
# Traceroute stops = routing black hole at that hop

# Add static route (temporary)
route add 192.168.100.0 mask 255.255.255.0 10.10.1.1
# Permanent:
route -p add 192.168.100.0 mask 255.255.255.0 10.10.1.1

# Linux
ip route add 192.168.100.0/24 via 10.10.1.1
```

---

## 7. Firewall & ACL Issues

### Diagnosing Firewall Blocks
```powershell
# Test specific port connectivity
Test-NetConnection -ComputerName 10.10.1.20 -Port 443
Test-NetConnection -ComputerName 10.10.1.20 -Port 3389

# Windows
telnet 10.10.1.20 443   # telnet must be installed: dism /online /Enable-Feature /FeatureName:TelnetClient

# Linux
nc -zv 10.10.1.20 443
curl -v telnet://10.10.1.20:443

# Port scan (from trusted machine to target)
nmap -p 22,80,443,3389 10.10.1.20

# Check Windows Firewall rules
Get-NetFirewallRule | Where-Object Enabled -eq True | Where-Object Direction -eq Inbound | 
  Select-Object DisplayName, Action | Sort-Object Action
```

### Windows Firewall Troubleshooting
```powershell
# Temporarily disable Windows Firewall to test (do in isolated environment)
Set-NetFirewallProfile -All -Enabled False
# Re-enable:
Set-NetFirewallProfile -All -Enabled True

# Check if specific rule is blocking
Get-NetFirewallRule -DisplayName "*Remote Desktop*"
Enable-NetFirewallRule -DisplayName "Remote Desktop - User Mode (TCP-In)"

# Windows Firewall logs
# Enable logging: wf.msc → Windows Firewall Properties → Logging
# Log file: %systemroot%\system32\LogFiles\Firewall\pfirewall.log
```

### Network Firewall / ACL Investigation
```
# Work with your network team to:
1. Check firewall policy between source and destination zones
2. Review ACLs on intervening routers
3. Check NAT translations if crossing NAT boundaries
4. Review proxy settings if web traffic

# Cisco firewall (ASA)
show conn | include <src-ip>
show access-list
packet-tracer input <interface> tcp <src-ip> 12345 <dst-ip> 443

# Palo Alto
> test security-policy-match source <ip> destination <ip> destination-port 443 protocol 6
```

---

## 8. Wi-Fi Troubleshooting

### Client-Side Checks
```powershell
# Windows Wi-Fi diagnostics
netsh wlan show interfaces
netsh wlan show networks
netsh wlan show profiles

# Signal strength and connected AP
netsh wlan show interfaces | Select-String "Signal|BSSID|SSID|Channel"

# Forget and reconnect
netsh wlan delete profile name="<SSID>"

# Disable/enable Wi-Fi adapter
netsh interface set interface "Wi-Fi" admin=disabled
netsh interface set interface "Wi-Fi" admin=enabled

# Windows Wi-Fi event log
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-WLAN-AutoConfig/Operational'} -MaxEvents 30
```

### Common Wi-Fi Issues

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| Connected but no internet | DHCP failure or gateway issue | Check DHCP, reboot AP |
| Can't see SSID | SSID hidden or on 5GHz only | Check band, unhide SSID for testing |
| Authentication fails | Wrong password, certificate issue | Verify PSK, check 802.1X certs |
| Intermittent drops | RF interference or AP overloaded | Survey channels, check client count |
| Slow speeds | Channel congestion, low signal | Move closer, change channel |
| Roaming issues | Sticky client, no 802.11r | Enable fast BSS transition |

### Controller-Side Investigation (Cisco, Meraki, Aruba, Ubiquiti)
```
For cloud controllers:
- Check client event log in dashboard
- Review AP association table
- Check RADIUS authentication logs (for 802.1X)
- Review RF health/channel utilization
- Check for rogue APs

For Meraki:
Network → Wireless → Air Marshal (rogue APs)
Network → Wireless → Clients → filter by SSID
```

---

## 9. VPN Troubleshooting

### Common VPN Error Categories
| Error Type | Possible Cause |
|-----------|---------------|
| Can't connect at all | Firewall blocking VPN ports, DNS failure, server down |
| Authentication failure | Wrong credentials, expired certificate, MFA issue |
| Connected but no access | Split tunnel config, DNS not pushing, routing issue |
| Connected then drops | MTU issue, session timeout, DPD, ISP instability |
| Slow VPN performance | MTU/fragmentation, server load, routing inefficiency |

### IPSec/IKE VPN
```powershell
# Check VPN ports are open (IKEv2)
Test-NetConnection -ComputerName vpn.company.com -Port 500    # IKE
Test-NetConnection -ComputerName vpn.company.com -Port 4500   # NAT-T
Test-NetConnection -ComputerName vpn.company.com -Port 443    # SSTP fallback

# Windows built-in VPN diagnostics
rasdial <VPN Name>  # Try connecting via CLI for better error output
Get-VpnConnection
```

### SSL VPN (Cisco AnyConnect, GlobalProtect, etc.)
```powershell
# Check DNS resolves VPN gateway
nslookup vpn.company.com

# Test HTTPS to gateway
Test-NetConnection vpn.company.com -Port 443

# Cisco AnyConnect logs
# C:\ProgramData\Cisco\Cisco AnyConnect Secure Mobility Client\Temp\Logs\

# GlobalProtect logs
# C:\Program Files\Palo Alto Networks\GlobalProtect\PanGPS.log
# C:\Users\<user>\AppData\Roaming\Palo Alto Networks\GlobalProtect\

# Collect diagnostics
# AnyConnect: Help → Diagnostics
# GlobalProtect: Troubleshooting → Collect Logs
```

### MTU Issues on VPN
```powershell
# Test MTU with ping (Windows)
ping -l 1400 -f 8.8.8.8    # If fails, reduce size
ping -l 1300 -f 8.8.8.8    # Binary search for max MTU

# Recommended VPN MTU:
# Ethernet MTU 1500 - IPSec overhead (~60 bytes) = ~1440 for inner packets
# Set interface MTU on VPN adapter to 1400 for safety

# Linux
ip link set dev tun0 mtu 1400
```

---

## 10. Site-Wide Outages

### Initial Assessment
```
1. Verify scope: Is it one office? All offices? One VLAN? All VLANs?
2. Check ISP status page / NOC
3. Check core switch/router status
4. Check internet edge device (firewall/router)
5. Check if internal-to-internal traffic is also affected
   - If YES: likely core switch or routing issue
   - If NO: likely WAN/ISP or edge device issue
```

### WAN Failure Checklist
```
□ Check ISP circuit status (call NOC, check portal)
□ Check physical WAN link (SFP, fiber, copper)
□ Check interface on edge router: show interfaces <WAN_int>
□ Check BGP/static route to ISP: show ip route
□ Test with different endpoint (ping ISP gateway directly)
□ Check for DDoS or traffic spike in ISP dashboard
□ Activate failover circuit if available
```

### Core Network Failure Checklist
```
□ Console into core switch/router (out-of-band access)
□ Check CPU/memory: show processes cpu | show memory
□ Check for spanning tree topology change: show spanning-tree detail | inc topology
□ Check for broadcast storm: show interfaces | inc input rate
□ Check for routing protocol issues: show ip ospf neighbor / show ip bgp summary
□ Review syslog for recent errors
□ Check for looped cable (watch for rapidly flashing MAC table)
```

---

## 11. Performance & Latency Issues

### Baseline First
```powershell
# Establish baseline during normal operations
# Compare against during degradation

# Continuous ping with timestamp
ping -t 8.8.8.8 | ForEach-Object {"$(Get-Date -Format HH:mm:ss) $_"}

# Pathping (combines ping and traceroute with packet loss stats)
pathping -n 8.8.8.8

# iPerf bandwidth test (requires iPerf server at destination)
iperf3 -c 10.10.1.20 -t 30        # TCP test
iperf3 -c 10.10.1.20 -u -b 100M   # UDP test at 100Mbps

# MTR (My Traceroute) — Linux
mtr --report 8.8.8.8
```

### Identifying Congestion
```powershell
# Check interface utilization on router/switch
# show interfaces GigabitEthernet0/0   (look for input/output rate vs bandwidth)

# Check QoS policy
# show policy-map interface

# Check for buffer drops
# show interfaces | include drops|errors

# Monitor from Windows
# Resource Monitor → Network tab → shows per-process bandwidth usage
```

---

## 12. Packet Capture with Wireshark

### Quick Capture Filters
```wireshark
# Specific host
host 10.10.1.50

# Specific port
port 443
port 53   # DNS

# Between two hosts
host 10.10.1.50 and host 10.10.1.10

# HTTP/HTTPS
tcp port 80 or tcp port 443

# ICMP only
icmp
```

### Display Filters (After Capture)
```wireshark
# DNS queries and responses
dns

# TCP retransmissions (indicates packet loss)
tcp.analysis.retransmission

# TCP connection resets
tcp.flags.reset == 1

# Specific conversation
ip.addr == 10.10.1.50

# HTTP errors
http.response.code >= 400

# TLS handshake issues
ssl.alert_message or tls.alert_message

# Large packet sizes (MTU issues)
frame.len > 1500
```

### Capturing on Remote Windows Host
```powershell
# Built-in capture (Windows 7+)
netsh trace start capture=yes IPv4.Address=10.10.1.50 tracefile=C:\Temp\capture.etl maxsize=500
netsh trace stop

# Convert to pcap for Wireshark analysis
# Use Microsoft Message Analyzer or etl2pcapng tool
```

---

*See also: [DNS Administration Guide](../networking/03-dns-administration.md) | [VPN Configuration & Management](../networking/06-vpn-configuration-management.md) | [Firewall Administration](../networking/05-firewall-administration.md)*
