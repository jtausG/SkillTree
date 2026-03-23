# Network Device Configuration Guide

## Overview
Practical configuration reference for enterprise network devices including Cisco switches and routers, Palo Alto/Fortinet firewalls, and network access control. Includes CLI commands, best practices, and troubleshooting.

---

## 1. Cisco Switch Configuration

### Initial Setup
```
! Connect via console (9600 baud, 8N1)
! Enter privileged mode
enable
configure terminal

! Set hostname
hostname SW-CORE-01

! Set domain name (required for SSH)
ip domain-name corp.domain.com

! Create local admin user
username admin privilege 15 secret <strongpassword>

! Generate RSA key for SSH
crypto key generate rsa modulus 2048

! Enable SSH v2 only, disable telnet
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
line vty 0 15
  transport input ssh
  login local
  exec-timeout 15 0

! Set console timeout
line con 0
  exec-timeout 15 0
  logging synchronous

! Enable secret (enable password - encrypted)
enable secret <strongpassword>

! Encrypt all plaintext passwords
service password-encryption

! Disable unused services
no ip http server
no ip http secure-server
no cdp run          ! Or keep CDP for Cisco Discovery Protocol
no lldp run

! Banner
banner motd ^
UNAUTHORIZED ACCESS TO THIS DEVICE IS PROHIBITED.
All activity is monitored and logged.
^
```

### VLAN Configuration
```
! Create VLANs
vlan 10
  name Users
vlan 20
  name Servers
vlan 30
  name VoIP
vlan 40
  name Management
vlan 99
  name Native

! Configure access port (end device)
interface GigabitEthernet1/0/1
  description WORKSTATION-DESK-101
  switchport mode access
  switchport access vlan 10
  spanning-tree portfast
  spanning-tree bpduguard enable
  no shutdown

! Configure trunk port (uplink/inter-switch)
interface GigabitEthernet1/0/48
  description UPLINK-TO-DISTRIBUTION
  switchport mode trunk
  switchport trunk native vlan 99
  switchport trunk allowed vlan 10,20,30,40
  no shutdown

! Configure voice VLAN (desk phone + PC on same port)
interface GigabitEthernet1/0/5
  description VOIP-DESK-102
  switchport mode access
  switchport access vlan 10
  switchport voice vlan 30
  spanning-tree portfast
  no shutdown
```

### Layer 3 (Routing) — Distribution/Core Switch
```
! Enable routing
ip routing

! Create SVI (VLAN interface) for inter-VLAN routing
interface Vlan10
  description Users_Gateway
  ip address 10.10.10.1 255.255.255.0
  no shutdown

interface Vlan20
  description Servers_Gateway
  ip address 10.20.20.1 255.255.255.0
  no shutdown

interface Vlan40
  description Management
  ip address 10.40.40.1 255.255.255.0
  no shutdown

! Default route (to firewall)
ip route 0.0.0.0 0.0.0.0 10.100.100.1

! OSPF configuration
router ospf 1
  router-id 10.40.40.1
  network 10.10.0.0 0.0.255.255 area 0
  network 10.20.0.0 0.0.255.255 area 0
  passive-interface default
  no passive-interface GigabitEthernet1/1/1  ! Uplinks only
```

### Port Security
```
! Limit MAC addresses per port (user ports)
interface GigabitEthernet1/0/1
  switchport port-security maximum 2
  switchport port-security violation restrict
  switchport port-security aging time 10
  switchport port-security

! 802.1X (NAC - requires RADIUS/AAA server)
aaa new-model
aaa authentication dot1x default group radius
aaa authorization network default group radius
dot1x system-auth-control

interface GigabitEthernet1/0/1
  authentication port-control auto
  dot1x pae authenticator

! RADIUS server config
radius server CORPRADIUS
  address ipv4 10.40.40.50 auth-port 1812 acct-port 1813
  key <radiussecret>
```

### Spanning Tree
```
! Enable Rapid PVST+ (default on Cisco IOS)
spanning-tree mode rapid-pvst

! Set core as root bridge for all VLANs
spanning-tree vlan 1-4094 priority 4096

! Set distribution as secondary root
spanning-tree vlan 1-4094 priority 8192

! PortFast and BPDU Guard on ALL access ports (global)
spanning-tree portfast default
spanning-tree portfast bpduguard default

! BPDU Filter (use carefully - only where absolutely needed)
! spanning-tree portfast bpdufilter default
```

### EtherChannel (LAG/LACP)
```
! Create LACP port channel (uplink redundancy/bandwidth)
interface range GigabitEthernet1/0/47-48
  channel-group 1 mode active
  description PORT-CHANNEL-TO-CORE

interface Port-channel1
  description LACP-UPLINK-TO-CORE
  switchport mode trunk
  switchport trunk native vlan 99
  switchport trunk allowed vlan 10,20,30,40
  no shutdown
```

### Useful Cisco Switch Show Commands
```
show version                    ! IOS version, uptime
show running-config             ! Current config
show interfaces status          ! All interface summary
show interfaces Gi1/0/1         ! Detailed interface stats
show vlan brief                 ! VLAN table
show mac address-table          ! MAC address table
show ip arp                     ! ARP table
show spanning-tree              ! STP topology
show etherchannel summary       ! Port channel status
show ip route                   ! Routing table
show ip ospf neighbor           ! OSPF neighbors
show cdp neighbors detail       ! CDP neighbor devices
show port-security              ! Port security summary
show logging                    ! System log
show inventory                  ! Hardware inventory
show environment                ! Power/temperature/fans
debug spanning-tree events      ! STP debug (use carefully)
```

---

## 2. Cisco Router Configuration

### Basic Setup
```
hostname RTR-EDGE-01
ip domain-name corp.domain.com
crypto key generate rsa modulus 2048
ip ssh version 2
username admin privilege 15 secret <password>
enable secret <password>
service password-encryption
no ip http server
no ip http secure-server

! Interface configuration (WAN)
interface GigabitEthernet0/0
  description WAN-ISP1
  ip address dhcp               ! Or static
  no shutdown

! Interface configuration (LAN)
interface GigabitEthernet0/1
  description LAN-TO-CORE-SWITCH
  ip address 10.100.100.1 255.255.255.252
  no shutdown
```

### DHCP Server
```
! Exclude static IPs (gateways, servers, printers)
ip dhcp excluded-address 10.10.10.1 10.10.10.20

! Create DHCP pool
ip dhcp pool USERS_VLAN10
  network 10.10.10.0 255.255.255.0
  default-router 10.10.10.1
  dns-server 10.40.40.10 10.40.40.11
  domain-name corp.domain.com
  lease 1 0 0                   ! 1 day

! DHCP relay agent (on SVI when DHCP server is elsewhere)
interface Vlan10
  ip helper-address 10.40.40.50
```

### NAT/PAT (Internet access)
```
! Define inside/outside interfaces
interface GigabitEthernet0/0
  ip nat outside

interface GigabitEthernet0/1
  ip nat inside

! PAT (overload) - typical internet sharing
access-list 10 permit 10.0.0.0 0.255.255.255
ip nat inside source list 10 interface GigabitEthernet0/0 overload

! Static NAT (publish internal server)
ip nat inside source static 10.20.20.10 <public-ip>
```

### BGP (Internet Routing)
```
! eBGP with ISP
router bgp 65001
  bgp router-id 203.0.113.1
  neighbor 203.0.113.254 remote-as 64512    ! ISP ASN
  neighbor 203.0.113.254 description ISP-PEER
  !
  address-family ipv4
    network 203.0.113.0 mask 255.255.255.0  ! Your public range
    neighbor 203.0.113.254 activate
    neighbor 203.0.113.254 prefix-list FILTER-IN in
    neighbor 203.0.113.254 prefix-list FILTER-OUT out

! Prefix list to protect default route
ip prefix-list FILTER-IN permit 0.0.0.0/0   ! Accept default route only
ip prefix-list FILTER-OUT permit 203.0.113.0/24  ! Advertise only our prefix
```

---

## 3. Palo Alto Firewall Configuration

### Initial Setup (CLI)
```bash
# Configure management interface
set deviceconfig system ip-address 10.40.40.1
set deviceconfig system netmask 255.255.255.0
set deviceconfig system default-gateway 10.40.40.254
set deviceconfig system dns-setting servers primary 8.8.8.8

# Set admin password
set mgt-config users admin password

# Commit
commit

# Access GUI: https://10.40.40.1
# Default: admin/admin (change immediately)
```

### Security Zones
```
# Create security zones (via GUI or CLI)
set zone trust network layer3 ethernet1/1
set zone untrust network layer3 ethernet1/2
set zone dmz network layer3 ethernet1/3
set zone mgmt network layer3 loopback.1
```

### Security Policies

**Palo Alto policy format:**
```
Source Zone | Source Address | Destination Zone | Destination Address | Application | Service | Action
```

**Example policies (CLI):**
```bash
# Allow internal users to internet
set rulebase security rules "Users-to-Internet" from trust
set rulebase security rules "Users-to-Internet" to untrust
set rulebase security rules "Users-to-Internet" source any
set rulebase security rules "Users-to-Internet" destination any
set rulebase security rules "Users-to-Internet" application ["web-browsing","ssl","dns"]
set rulebase security rules "Users-to-Internet" service application-default
set rulebase security rules "Users-to-Internet" action allow
set rulebase security rules "Users-to-Internet" profile-setting profiles url-filtering strict-urlfiltering
set rulebase security rules "Users-to-Internet" profile-setting profiles virus default
set rulebase security rules "Users-to-Internet" profile-setting profiles spyware strict

# Allow DNS to internal servers
set rulebase security rules "DNS-Internal" from trust
set rulebase security rules "DNS-Internal" to trust
set rulebase security rules "DNS-Internal" source any
set rulebase security rules "DNS-Internal" destination ["10.40.40.10","10.40.40.11"]
set rulebase security rules "DNS-Internal" application dns
set rulebase security rules "DNS-Internal" service application-default
set rulebase security rules "DNS-Internal" action allow

# Block everything else (explicit deny with logging)
set rulebase security rules "Deny-All" from any
set rulebase security rules "Deny-All" to any
set rulebase security rules "Deny-All" source any
set rulebase security rules "Deny-All" destination any
set rulebase security rules "Deny-All" application any
set rulebase security rules "Deny-All" service any
set rulebase security rules "Deny-All" action deny
set rulebase security rules "Deny-All" log-setting default
```

### NAT Rules
```bash
# Source NAT (outbound internet)
set rulebase nat rules "Outbound-NAT" source-translation dynamic-ip-and-port interface-address interface ethernet1/2
set rulebase nat rules "Outbound-NAT" from trust
set rulebase nat rules "Outbound-NAT" to untrust
set rulebase nat rules "Outbound-NAT" source any
set rulebase nat rules "Outbound-NAT" destination any
set rulebase nat rules "Outbound-NAT" service any

# Destination NAT (inbound - publish DMZ server)
set rulebase nat rules "Inbound-WebServer" destination-translation translated-address 10.30.30.10
set rulebase nat rules "Inbound-WebServer" from untrust
set rulebase nat rules "Inbound-WebServer" to untrust
set rulebase nat rules "Inbound-WebServer" destination <public-ip>
set rulebase nat rules "Inbound-WebServer" service service-https
```

### PAN-OS Useful Commands
```bash
# Interface status
show interface all
show interface ethernet1/1

# Routing table
show routing route

# ARP table
show arp all

# Session table
show session all
show session id <id>

# Traffic logs (recent denies)
show log traffic direction equal backward
show log traffic dst-address equal 10.20.20.100

# Threat logs
show log threat

# HA status (high availability)
show high-availability all

# Commit status
show jobs all

# Software version
show system info

# Test connectivity
ping host 8.8.8.8 source ethernet1/1
test security-policy-match from trust to untrust source 10.10.10.100 destination 8.8.8.8 protocol 6 destination-port 443
```

---

## 4. FortiGate Firewall Configuration

### Initial CLI Setup
```bash
config system admin
    edit admin
        set password <newpassword>
    next
end

config system interface
    edit "mgmt"
        set ip 10.40.40.2 255.255.255.0
        set allowaccess https ssh ping
    next
end

config router static
    edit 1
        set gateway 10.40.40.254
        set device "mgmt"
    next
end
```

### Firewall Policies
```bash
config firewall policy
    edit 1
        set name "LAN-to-WAN"
        set srcintf "internal"
        set dstintf "wan1"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "ALL"
        set nat enable
        set utm-status enable
        set av-profile "default"
        set webfilter-profile "default"
        set ssl-ssh-profile "certificate-inspection"
        set logtraffic all
    next
    edit 2
        set name "Deny-All"
        set srcintf "any"
        set dstintf "any"
        set srcaddr "all"
        set dstaddr "all"
        set action deny
        set schedule "always"
        set service "ALL"
        set logtraffic all
    next
end
```

### Useful FortiGate Commands
```bash
get system status              # Version and status
get system interface           # Interface summary
diagnose ip route list         # Routing table
diagnose sys session list      # Session table
diagnose debug flow filter addr <ip>
diagnose debug flow show console enable
diagnose debug enable
diagnose debug flow trace start 10
# Stop debug:
diagnose debug flow trace stop
diagnose debug disable

# Packet capture
diagnose sniffer packet any "host 10.10.10.100 and port 443" 4
# Stop: Ctrl+C
```

---

## 5. Network Troubleshooting Playbook

### Layer-by-Layer Approach

**Layer 1 (Physical):**
```
- Check link lights (green = connected, amber = activity)
- Inspect cable connections
- Test with known good cable
- Check port speed/duplex settings
show interfaces Gi1/0/1 | include duplex|speed|error

# Common issue: duplex mismatch
interface Gi1/0/1
  duplex full
  speed 1000
```

**Layer 2 (Data Link):**
```
# Check if device MAC is in table
show mac address-table | include <partial-mac>

# Check if VLAN is active
show vlan brief | include active

# Check port in correct VLAN
show interfaces Gi1/0/1 switchport

# STP issues - port stuck in non-forwarding
show spanning-tree vlan 10
show spanning-tree detail | include forwarding|blocking|listening
```

**Layer 3 (Network):**
```
# Can you ping default gateway from device?
# Can switch ping the device?
ping 10.10.10.100 source Vlan10

# Check ARP (device must be in ARP table)
show ip arp 10.10.10.100

# Routing table — is route present?
show ip route 8.8.8.8
traceroute 8.8.8.8 source Vlan10
```

**Layer 7 (Application):**
```
# DNS resolution
nslookup google.com
# On Cisco router:
debug ip dns

# Firewall policy match test (Palo Alto)
test security-policy-match from trust to untrust source 10.10.10.100 destination 8.8.8.8

# Check session on firewall
show session all filter source 10.10.10.100
```

### Common Scenarios

**"Can't reach internet" from user PC:**
```
1. ipconfig /all — Is IP/gateway/DNS correct?
2. ping gateway — Layer 3 to switch?
3. ping 8.8.8.8 — Gateway to internet?
4. nslookup google.com — DNS working?
5. Check firewall logs for denies
6. Check proxy settings if applicable
```

**"Two sites can't communicate" (VPN):**
```
1. Verify VPN tunnel is up (show crypto isakmp sa / ipsec sa)
2. Check interesting traffic (ACL matching both sides)
3. Verify routing — route to remote subnet via tunnel interface
4. Check firewall rules allow inter-site traffic
5. Ping from each side — which side fails?
6. Check NAT — are private addresses being NATted before VPN?
```

---

## 6. IP Address Management (IPAM)

### Subnet Documentation Template
```
Subnet: 10.10.10.0/24
VLAN: 10
Name: Corporate Users - Site A
Gateway: 10.10.10.1
DHCP Range: 10.10.10.50 - 10.10.10.200
DNS: 10.40.40.10, 10.40.40.11
Static Reserve: 10.10.10.1 - 10.10.10.49
Purpose: End user workstations
Location: Building A
Notes: Proxy required for internet
```

### Subnetting Quick Reference
| CIDR | Mask | Hosts | Use Case |
|------|------|-------|---------|
| /30 | 255.255.255.252 | 2 | Point-to-point links |
| /29 | 255.255.255.248 | 6 | Small LANs |
| /28 | 255.255.255.240 | 14 | Very small LAN |
| /27 | 255.255.255.224 | 30 | Small VLAN |
| /26 | 255.255.255.192 | 62 | Azure Bastion minimum |
| /25 | 255.255.255.128 | 126 | Small office |
| /24 | 255.255.255.0 | 254 | Standard VLAN |
| /23 | 255.255.254.0 | 510 | Medium VLAN |
| /22 | 255.255.252.0 | 1022 | Large VLAN |
| /16 | 255.255.0.0 | 65,534 | Site range |

---

## 7. Wireless Network Configuration

### Cisco WLC (Wireless LAN Controller)

**SSID Configuration (GUI-based, CLI reference):**
```
# Create WLAN
config wlan create 10 "Corporate-WiFi" "Corporate-WiFi"
config wlan security wpa akm dot1x enable 10   ! 802.1X auth
config wlan security wpa wpa2 enable 10
config wlan security wpa wpa2 ciphers aes enable 10
config wlan interface 10 vlan10-interface
config wlan enable 10

# Guest SSID (isolated)
config wlan create 20 "Guest-WiFi" "Guest-WiFi"
config wlan security web-auth enable 20
config wlan interface 20 guest-interface
config wlan enable 20
```

**Key Wireless Standards:**
| Standard | Band | Max Speed | Range | Notes |
|----------|------|-----------|-------|-------|
| 802.11n (Wi-Fi 4) | 2.4/5 GHz | 600 Mbps | Good | Legacy, still common |
| 802.11ac (Wi-Fi 5) | 5 GHz | 3.5 Gbps | Medium | Current standard |
| 802.11ax (Wi-Fi 6) | 2.4/5/6 GHz | 9.6 Gbps | Best | Modern deployments |
| 802.11ax (Wi-Fi 6E) | 6 GHz only | 9.6 Gbps | Short | New spectrum |

**Wireless Troubleshooting:**
```
# Check AP association
show ap summary

# Check client association
show client summary
show client detail <mac-address>

# AP radio stats
show ap dot11 5ghz summary

# Rogue AP detection
show rogue ap summary

# RADIUS authentication failure
show client detail <mac> | include RADIUS
```

---

*Last Updated: 2025 | IT Operations Documentation Library*
