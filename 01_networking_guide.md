# Networking Guide — Fundamentals, Design & Troubleshooting

## Overview
Comprehensive networking reference for IT Operations covering TCP/IP fundamentals, enterprise network design, VLAN management, routing, switching, wireless, and firewall administration.

---

## Table of Contents
1. [TCP/IP Fundamentals](#tcpip)
2. [Subnetting Reference](#subnetting)
3. [VLAN Design & Management](#vlans)
4. [Routing Protocols](#routing)
5. [Switching & STP](#switching)
6. [Wireless Networking](#wireless)
7. [Firewall & ACLs](#firewall)
8. [DNS Operations](#dns)
9. [DHCP Operations](#dhcp)
10. [Network Troubleshooting Playbook](#troubleshooting)
11. [Bandwidth & QoS](#qos)
12. [VPN Configuration](#vpn)

---

## 1. TCP/IP Fundamentals <a name="tcpip"></a>

### OSI Model Reference
| Layer | Name | Protocols | Devices |
|---|---|---|---|
| 7 | Application | HTTP, HTTPS, DNS, SMTP, FTP, SSH | - |
| 6 | Presentation | TLS/SSL, JPEG, MPEG | - |
| 5 | Session | NetBIOS, RPC, PPTP | - |
| 4 | Transport | TCP, UDP | Load Balancers |
| 3 | Network | IP, ICMP, OSPF, BGP | Routers |
| 2 | Data Link | Ethernet, 802.11, PPP | Switches, APs |
| 1 | Physical | Ethernet cables, Fiber, Radio | Hubs, Repeaters |

### Key Port Numbers
| Port | Protocol | Service |
|---|---|---|
| 20/21 | TCP | FTP |
| 22 | TCP | SSH |
| 23 | TCP | Telnet (insecure) |
| 25 | TCP | SMTP |
| 53 | TCP/UDP | DNS |
| 67/68 | UDP | DHCP |
| 80 | TCP | HTTP |
| 88 | TCP/UDP | Kerberos |
| 110 | TCP | POP3 |
| 135 | TCP | RPC Endpoint Mapper |
| 137-139 | TCP/UDP | NetBIOS |
| 143 | TCP | IMAP |
| 389 | TCP/UDP | LDAP |
| 443 | TCP | HTTPS |
| 445 | TCP | SMB |
| 465/587 | TCP | SMTP (TLS) |
| 636 | TCP | LDAPS |
| 993 | TCP | IMAPS |
| 995 | TCP | POP3S |
| 1433 | TCP | SQL Server |
| 1521 | TCP | Oracle DB |
| 3306 | TCP | MySQL |
| 3389 | TCP | RDP |
| 5985/5986 | TCP | WinRM |
| 8080/8443 | TCP | HTTP/HTTPS Alt |

---

## 2. Subnetting Reference <a name="subnetting"></a>

### CIDR Quick Reference
| CIDR | Subnet Mask | Usable Hosts | Network Size |
|---|---|---|---|
| /30 | 255.255.255.252 | 2 | Point-to-point links |
| /29 | 255.255.255.248 | 6 | Small segments |
| /28 | 255.255.255.240 | 14 | Small office |
| /27 | 255.255.255.224 | 30 | Small VLAN |
| /26 | 255.255.255.192 | 62 | Medium VLAN |
| /25 | 255.255.255.128 | 126 | Medium subnet |
| /24 | 255.255.255.0 | 254 | Standard LAN |
| /23 | 255.255.254.0 | 510 | Large subnet |
| /22 | 255.255.252.0 | 1022 | Campus subnet |
| /21 | 255.255.248.0 | 2046 | Large campus |
| /20 | 255.255.240.0 | 4094 | Enterprise |
| /16 | 255.255.0.0 | 65534 | Large enterprise |

### Private IP Ranges (RFC 1918)
```
10.0.0.0    /8   → 10.0.0.0 – 10.255.255.255
172.16.0.0  /12  → 172.16.0.0 – 172.31.255.255
192.168.0.0 /16  → 192.168.0.0 – 192.168.255.255
```

### Typical Enterprise Subnet Design
```
10.0.0.0/8 → Entire enterprise
  10.1.0.0/16 → Site A
    10.1.1.0/24  → VLAN 10 - Users (254 hosts)
    10.1.2.0/24  → VLAN 20 - Servers
    10.1.3.0/24  → VLAN 30 - Printers
    10.1.4.0/24  → VLAN 40 - VoIP
    10.1.5.0/24  → VLAN 50 - Management
    10.1.6.0/24  → VLAN 60 - Wireless
    10.1.100.0/30 → WAN Link to HQ
  10.2.0.0/16 → Site B
    (similar structure)
```

---

## 3. VLAN Design & Management <a name="vlans"></a>

### VLAN Best Practices
- Separate user, server, management, and IoT traffic
- Use VLAN 1 for management only (or avoid it — vendors differ)
- Trunk ports carry multiple VLANs; access ports carry one
- Always use 802.1Q tagging on trunks
- Document VLAN-to-subnet mappings in your IPAM

### Cisco VLAN Configuration
```cisco
! Create VLANs
vlan 10
 name USERS
vlan 20
 name SERVERS
vlan 30
 name MANAGEMENT

! Configure access port (end device)
interface GigabitEthernet0/1
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
 no shutdown

! Configure trunk port (to another switch or router)
interface GigabitEthernet0/24
 switchport mode trunk
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 10,20,30
 no shutdown

! Configure SVI (Layer 3 inter-VLAN routing)
interface vlan 10
 ip address 10.1.1.1 255.255.255.0
 ip helper-address 10.1.5.10   ! DHCP relay
 no shutdown
```

### Verifying VLANs
```cisco
show vlan brief
show interfaces trunk
show interfaces GigabitEthernet0/1 switchport
show spanning-tree vlan 10
```

---

## 4. Routing Protocols <a name="routing"></a>

### Static Routing
```cisco
! Default route
ip route 0.0.0.0 0.0.0.0 <next-hop-IP>

! Static route
ip route 192.168.2.0 255.255.255.0 10.0.0.2

! Floating static (backup with higher admin distance)
ip route 192.168.2.0 255.255.255.0 10.0.0.3 254
```

### OSPF Basics
```cisco
router ospf 1
 router-id 1.1.1.1
 network 10.1.0.0 0.0.255.255 area 0
 passive-interface GigabitEthernet0/1   ! Don't send OSPF on user ports
 default-information originate           ! Advertise default route

! Verify
show ip ospf neighbor
show ip ospf database
show ip route ospf
```

### BGP Overview
```cisco
router bgp 65001
 bgp router-id 1.1.1.1
 neighbor 203.0.113.1 remote-as 65002
 network 198.51.100.0 mask 255.255.255.0

! Verify
show bgp summary
show ip bgp
show ip bgp neighbors
```

---

## 5. Switching & STP <a name="switching"></a>

### Spanning Tree Protocol
STP prevents Layer 2 loops by blocking redundant paths.

```cisco
! Enable RSTP (faster convergence than classic STP)
spanning-tree mode rapid-pvst

! Set a switch as root for VLAN 10
spanning-tree vlan 10 root primary

! Set secondary root
spanning-tree vlan 10 root secondary

! Portfast (for end-device ports only!)
interface GigabitEthernet0/1
 spanning-tree portfast

! BPDU Guard (drop BPDU from unexpected ports)
interface GigabitEthernet0/1
 spanning-tree bpduguard enable

! Verify STP
show spanning-tree vlan 10
show spanning-tree interface GigabitEthernet0/1 detail
```

### EtherChannel / LACP (Link Aggregation)
```cisco
! LACP (active/active preferred)
interface range GigabitEthernet0/1 - 2
 channel-protocol lacp
 channel-group 1 mode active

interface Port-channel1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30

! Verify
show etherchannel summary
show etherchannel 1 detail
```

---

## 6. Wireless Networking <a name="wireless"></a>

### 802.11 Standards Reference
| Standard | Max Speed | Band | Notes |
|---|---|---|---|
| 802.11a | 54 Mbps | 5 GHz | Legacy |
| 802.11b | 11 Mbps | 2.4 GHz | Legacy |
| 802.11g | 54 Mbps | 2.4 GHz | Legacy |
| 802.11n (Wi-Fi 4) | 600 Mbps | 2.4/5 GHz | Widespread |
| 802.11ac (Wi-Fi 5) | 3.5 Gbps | 5 GHz | Common enterprise |
| 802.11ax (Wi-Fi 6) | 9.6 Gbps | 2.4/5/6 GHz | Current enterprise |
| 802.11be (Wi-Fi 7) | 46 Gbps | 2.4/5/6 GHz | Emerging |

### Enterprise WLAN Best Practices
- Use WPA3-Enterprise (802.1X) for corporate SSIDs
- Separate guest SSID with Internet-only VLAN
- Deploy a wireless controller (Cisco WLC, Aruba, Meraki) for centralized management
- Enable 802.11r (fast roaming) for voice/video
- Perform RF site surveys before and after deployment
- Set non-overlapping channels: 2.4GHz (1, 6, 11); 5GHz (many options)
- Enable band steering to push capable clients to 5GHz/6GHz

### Wireless Troubleshooting
```bash
# Windows - Check wireless adapter and signal
netsh wlan show interfaces
netsh wlan show networks mode=bssid
netsh wlan show profiles
netsh wlan show profile name="SSID" key=clear

# Check connection quality
netsh wlan show interface | Select-String "Signal"

# Linux
iwconfig
iwlist wlan0 scan
nmcli dev wifi list
```

---

## 7. Firewall & ACLs <a name="firewall"></a>

### Firewall Rule Design Principles
1. **Default Deny** — block all, allow by exception
2. **Principle of Least Privilege** — only open what is required
3. **Document every rule** — include ticket number and date
4. **Review regularly** — audit rules quarterly
5. **Log denied traffic** — critical for security monitoring

### Cisco ACL Examples
```cisco
! Standard ACL (source IP only)
ip access-list standard MGMT_ACCESS
 permit 10.1.5.0 0.0.0.255
 deny any log

! Extended ACL (source, destination, protocol, port)
ip access-list extended ALLOW_WEB
 permit tcp 10.1.1.0 0.0.0.255 any eq 80
 permit tcp 10.1.1.0 0.0.0.255 any eq 443
 deny ip any any log

! Apply to interface
interface GigabitEthernet0/0
 ip access-group ALLOW_WEB in

! Verify
show ip access-lists ALLOW_WEB
show interfaces GigabitEthernet0/0 | include access list
```

### Windows Firewall Management
```powershell
# View rules
Get-NetFirewallRule | Where-Object Enabled -eq True | Select-Object DisplayName, Direction, Action | Format-Table

# Add a rule
New-NetFirewallRule -DisplayName "Allow RDP" -Direction Inbound -Protocol TCP -LocalPort 3389 -Action Allow

# Block outbound to specific IP
New-NetFirewallRule -DisplayName "Block Bad Host" -Direction Outbound -RemoteAddress 192.168.99.1 -Action Block

# Remove a rule
Remove-NetFirewallRule -DisplayName "Allow RDP"

# Export all rules
Get-NetFirewallRule | Export-Csv C:\firewall_rules.csv
```

---

## 8. DNS Operations <a name="dns"></a>

### DNS Record Types
| Record | Purpose | Example |
|---|---|---|
| A | IPv4 address | `server01 → 10.1.2.10` |
| AAAA | IPv6 address | `server01 → ::1` |
| CNAME | Alias | `www → server01.domain.com` |
| MX | Mail server | `mail.domain.com, priority 10` |
| PTR | Reverse lookup | `10.1.2.10 → server01.domain.com` |
| NS | Name server | `domain.com → ns1.domain.com` |
| SOA | Zone authority | Start of Authority record |
| SRV | Service location | `_ldap._tcp.domain.com` |
| TXT | Text info | SPF, DKIM, DMARC records |

### DNS Troubleshooting Commands
```powershell
# Basic lookup
nslookup server01.domain.com
nslookup server01.domain.com 10.1.5.10   # Use specific DNS server

# PowerShell DNS queries
Resolve-DnsName server01.domain.com -Type A
Resolve-DnsName domain.com -Type MX
Resolve-DnsName 10.1.2.10 -Type PTR          # Reverse lookup

# Clear DNS cache
ipconfig /flushdns
Clear-DnsClientCache

# Check local DNS cache
Get-DnsClientCache | Select-Object Entry, Data, TimeToLive

# Test DNS resolution time
Measure-Command { Resolve-DnsName server01.domain.com }
```

### Windows DNS Server Management
```powershell
# Add A record
Add-DnsServerResourceRecordA -ZoneName "domain.com" -Name "newserver" -IPv4Address "10.1.2.50"

# Add CNAME
Add-DnsServerResourceRecordCName -ZoneName "domain.com" -Name "www" -HostNameAlias "webserver.domain.com."

# Add PTR record
Add-DnsServerResourceRecordPtr -ZoneName "2.1.10.in-addr.arpa" -Name "50" -PtrDomainName "newserver.domain.com."

# Remove a record
Remove-DnsServerResourceRecord -ZoneName "domain.com" -Name "oldserver" -RRType A -Force

# View zone records
Get-DnsServerResourceRecord -ZoneName "domain.com" | Sort-Object HostName | Format-Table

# Check DNS replication
repadmin /showrepl
```

---

## 9. DHCP Operations <a name="dhcp"></a>

### DHCP Scope Configuration
```powershell
# Add DHCP scope
Add-DhcpServerv4Scope -Name "VLAN10-Users" -StartRange 10.1.1.10 -EndRange 10.1.1.250 -SubnetMask 255.255.255.0 -State Active

# Set scope options
Set-DhcpServerv4OptionValue -ScopeId 10.1.1.0 -DnsDomain "domain.com" -DnsServer 10.1.5.10,10.1.5.11 -Router 10.1.1.1

# Add reservation
Add-DhcpServerv4Reservation -ScopeId 10.1.1.0 -IPAddress 10.1.1.20 -ClientId "AA-BB-CC-DD-EE-FF" -Description "Printer-01"

# View leases
Get-DhcpServerv4Lease -ScopeId 10.1.1.0 | Sort-Object LeaseExpiryTime | Format-Table

# Check for duplicate IPs
Get-DhcpServerv4Lease -ScopeId 10.1.1.0 | Group-Object IPAddress | Where-Object Count -gt 1
```

---

## 10. Network Troubleshooting Playbook <a name="troubleshooting"></a>

### Complete Connectivity Troubleshooting

```
SYMPTOM: User cannot reach a specific server

STEP 1: Gather info
  - What server? What service? (web, file share, RDP)
  - Can they reach other servers?
  - When did it start?
  - Any recent changes?

STEP 2: Client-side checks
  ipconfig /all              → Verify IP, gateway, DNS
  ping 127.0.0.1             → TCP/IP stack
  ping <gateway>             → Local LAN
  ping <server IP>           → Skip DNS
  ping <server FQDN>         → Include DNS
  Test-NetConnection -ComputerName server -Port 443  → Service port

STEP 3: DNS check
  nslookup <server>          → Does it resolve?
  nslookup <server> 8.8.8.8  → Test alternate DNS
  ipconfig /flushdns          → Clear cache and retry

STEP 4: Routing check
  tracert <server>           → Where does it fail?
  route print                → Check routing table

STEP 5: Server-side checks
  ping <server> from another machine
  Check server NIC, services, firewall
  netstat -ano | findstr <port>   → Is service listening?

STEP 6: Network device checks
  Check switch port status and VLAN
  Check firewall rules
  Check ACLs on router
  Check WAN link status if cross-site
```

### Network Performance Testing
```bash
# iPerf3 bandwidth test
# Server side
iperf3 -s

# Client side
iperf3 -c <server-ip> -t 30    # 30 second test
iperf3 -c <server-ip> -u -b 100M  # UDP test at 100 Mbps

# Packet loss test with ping
ping -t <server>               # Continuous ping, check for drops
ping <server> -n 1000          # 1000 pings, count losses

# Path MTU discovery
ping <server> -f -l 1472       # Test for 1500-byte MTU
```

---

## 11. Bandwidth & QoS <a name="qos"></a>

### QoS DSCP Markings
| Traffic Type | DSCP Value | Remarks |
|---|---|---|
| Voice (RTP) | EF (46) | Highest priority |
| Video conferencing | AF41 (34) | High priority |
| Call signaling | CS3 (24) | Important control |
| Business critical apps | AF21 (18) | Medium priority |
| General data | DF (0) | Best effort |
| Scavenger/Bulk | CS1 (8) | Low priority |

### Cisco QoS Policy
```cisco
! Class map - match traffic
class-map match-any VOICE
 match dscp ef
 match ip dscp 46

class-map match-any VIDEO
 match dscp af41

! Policy map - apply queuing
policy-map WAN-QOS
 class VOICE
  priority percent 20      ! Strict priority, 20% bandwidth
 class VIDEO
  bandwidth percent 30     ! Guaranteed bandwidth
 class class-default
  fair-queue               ! WFQ for rest

! Apply to interface
interface Serial0/0
 service-policy output WAN-QOS
```

---

## 12. VPN Configuration <a name="vpn"></a>

### SSL VPN vs IPsec VPN
| Feature | SSL VPN | IPsec VPN |
|---|---|---|
| Protocol | TLS (TCP 443) | ESP (IP 50), IKE (UDP 500/4500) |
| Client setup | Browser or thin client | Full client required |
| Firewall traversal | Excellent (uses 443) | Sometimes blocked |
| Split tunneling | Easy | Possible but complex |
| Performance | Good | Excellent |
| Use case | Remote users | Site-to-site, power users |

### Windows Always On VPN (Modern Remote Access)

```powershell
# Deploy AOVPN Device Tunnel (example profile)
$VpnProfileXml = @"
<VPNProfile>
  <AlwaysOn>true</AlwaysOn>
  <RememberCredentials>true</RememberCredentials>
  <NativeProfile>
    <Servers>vpn.company.com</Servers>
    <NativeProtocolType>IKEv2</NativeProtocolType>
    <Authentication>
      <MachineMethod>Certificate</MachineMethod>
    </Authentication>
    <RoutingPolicyType>SplitTunnel</RoutingPolicyType>
  </NativeProfile>
</VPNProfile>
"@

# Apply profile
Add-VpnConnection -Name "CorpVPN-Device" -ServerAddress "vpn.company.com" -TunnelType IKEv2 -AuthenticationMethod MachineCertificate -SplitTunneling -AllUserConnection

# View VPN connections
Get-VpnConnection -AllUserConnection
```
