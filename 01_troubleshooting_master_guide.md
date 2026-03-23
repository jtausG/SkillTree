# IT Troubleshooting Master Guide

## Overview
This guide provides a systematic approach to diagnosing and resolving IT issues across all infrastructure layers. It is intended for IT professionals at all levels — from help desk technicians to senior engineers.

---

## Table of Contents
1. [The Troubleshooting Methodology](#methodology)
2. [Information Gathering Checklist](#information-gathering)
3. [Layered Diagnostic Approach](#layered-diagnostic)
4. [Common Error Categories](#error-categories)
5. [Escalation Decision Tree](#escalation)
6. [Documentation Standards](#documentation)
7. [Tools Reference](#tools-reference)

---

## 1. The Troubleshooting Methodology <a name="methodology"></a>

### The OSI-Aligned Approach
Always work bottom-up (physical → application) unless symptoms clearly point to a specific layer.

```
Step 1: Define the problem clearly
Step 2: Gather information (symptoms, affected users, timeline)
Step 3: Identify possible causes
Step 4: Create a hypothesis
Step 5: Test the hypothesis with the least disruptive method
Step 6: Implement a fix
Step 7: Verify the fix works
Step 8: Document the resolution
Step 9: Perform root cause analysis
Step 10: Implement preventive measures
```

### The "5 Whys" Technique
Recursively ask "why" to find root cause:
- Problem: User cannot access shared drive
- Why 1: Network share is unreachable
- Why 2: DNS not resolving the file server FQDN
- Why 3: DNS server not responding
- Why 4: DNS service crashed on DC01
- Why 5: Disk full on DC01 caused service crash → **Root cause: Disk space management failure**

---

## 2. Information Gathering Checklist <a name="information-gathering"></a>

### Initial Questions to Ask the User
- [ ] What exactly are you trying to do?
- [ ] What error message do you see (exact text or screenshot)?
- [ ] When did this start? Was anything changed before it happened?
- [ ] Is anyone else affected or just you?
- [ ] Does it happen all the time or intermittently?
- [ ] What have you already tried?
- [ ] What is the business impact / urgency?

### System Information to Collect
```bash
# Windows
systeminfo
ipconfig /all
hostname
whoami /all
net user %username% /domain

# Linux/Mac
uname -a
ifconfig / ip addr
hostname
id
cat /etc/os-release
```

### Event Logs to Check
- **Windows**: Event Viewer → System, Application, Security
- **Linux**: `/var/log/syslog`, `/var/log/auth.log`, `journalctl -xe`
- **Application Logs**: Check application-specific log paths

---

## 3. Layered Diagnostic Approach <a name="layered-diagnostic"></a>

### Layer 1: Physical
- Is the device powered on?
- Are cables connected / lights active on NIC?
- Check for hardware failures: disk errors, RAM issues, CPU thermal throttling

### Layer 2: Data Link
```bash
# Check NIC driver / MAC address
ipconfig /all          # Windows
ip link show           # Linux

# Check for duplicate MACs or ARP conflicts
arp -a
```

### Layer 3: Network
```bash
# Connectivity checks
ping 127.0.0.1        # Loopback (TCP/IP stack)
ping <default_gateway> # Local network
ping 8.8.8.8          # Internet (bypasses DNS)
ping google.com       # DNS + Internet

# Route table
route print            # Windows
ip route               # Linux
tracert / traceroute   # Path tracing
```

### Layer 4: Transport
```bash
# Check for port connectivity
Test-NetConnection -ComputerName server01 -Port 443   # PowerShell
telnet server01 443                                    # Legacy
nc -zv server01 443                                    # Linux

# View open connections
netstat -ano           # Windows
ss -tulpn              # Linux
```

### Layer 5-7: Application
```bash
# DNS Resolution
nslookup server01
nslookup server01 8.8.8.8    # Bypass local DNS
Resolve-DnsName server01     # PowerShell

# HTTP checks
curl -v https://internal.site.com
Invoke-WebRequest -Uri https://internal.site.com -UseBasicParsing

# Application-specific logs
Get-EventLog -LogName Application -Newest 50
```

---

## 4. Common Error Categories <a name="error-categories"></a>

### Connectivity Issues
| Symptom | Likely Cause | First Check |
|---|---|---|
| Can ping IP, not hostname | DNS resolution failure | `nslookup hostname` |
| Can ping gateway, not internet | Firewall / routing issue | `tracert 8.8.8.8` |
| Cannot ping gateway | NIC, VLAN, or switch issue | Check IP config, VLAN |
| Intermittent drops | Duplex mismatch, bad cable | NIC stats, cable test |
| Slow network | Bandwidth saturation, packet loss | `ping -t`, NIC stats |

### Authentication Issues
| Symptom | Likely Cause | First Check |
|---|---|---|
| "Access Denied" | Permissions, group membership | `whoami /groups` |
| "Account locked out" | Failed login attempts | AD Lockout tool |
| Cannot log in via RDP | NLA, group policy, firewall | Check RDP port 3389 |
| Kerberos errors | Time sync, DNS, SPN issues | Check time sync |
| NTLM fallback | DNS or SPN misconfiguration | Check SPN records |

### Performance Issues
| Symptom | Likely Cause | First Check |
|---|---|---|
| High CPU | Runaway process, malware | Task Manager / top |
| High RAM | Memory leak, insufficient RAM | Resource Monitor |
| Slow disk | Fragmentation, HDD failing | SMART data, disk I/O |
| Application hang | Deadlock, resource exhaustion | Process dump analysis |

---

## 5. Escalation Decision Tree <a name="escalation"></a>

```
Is the issue resolved within 15 minutes?
├─ YES → Document and close
└─ NO →
   Is it a P1/P2 (business critical)?
   ├─ YES → Escalate immediately + notify manager
   └─ NO →
      Does it require elevated access you don't have?
      ├─ YES → Escalate to Tier 2/3 with full notes
      └─ NO →
         Have you tried all known fixes?
         ├─ NO → Continue troubleshooting
         └─ YES → Escalate with documentation
```

### Escalation Information to Include
- Ticket number and priority
- Exact error messages (screenshots preferred)
- Steps already taken
- Time of first occurrence
- Number of affected users
- Business impact statement
- Any recent changes

---

## 6. Documentation Standards <a name="documentation"></a>

### Ticket Documentation Template
```
PROBLEM STATEMENT:
[Clear, concise description of the issue]

AFFECTED SYSTEMS/USERS:
[Specific systems, user count, departments]

TIMELINE:
[When first reported, when it started, any recent changes]

INVESTIGATION STEPS:
1. [What you checked and what you found]
2. [Commands run, output observed]
3. [Hypotheses tested]

ROOT CAUSE:
[What actually caused the issue]

RESOLUTION:
[Exact steps taken to resolve]

VERIFICATION:
[How you confirmed the fix worked]

PREVENTIVE ACTIONS:
[What should be done to prevent recurrence]
```

---

## 7. Tools Reference <a name="tools-reference"></a>

### Windows Built-in Tools
| Tool | Use Case |
|---|---|
| `eventvwr.msc` | Event logs |
| `perfmon` | Performance monitoring |
| `resmon` | Resource Monitor |
| `msconfig` | Startup configuration |
| `msinfo32` | System information |
| `devmgmt.msc` | Device Manager |
| `services.msc` | Services management |
| `netsh` | Network configuration |
| `Get-NetAdapter` (PS) | NIC details |
| `Test-NetConnection` (PS) | Network testing |

### Linux Tools
| Tool | Use Case |
|---|---|
| `top` / `htop` | Process monitoring |
| `iostat` | Disk I/O stats |
| `vmstat` | VM performance |
| `netstat` / `ss` | Network connections |
| `tcpdump` | Packet capture |
| `strace` | System call tracing |
| `lsof` | Open files / ports |
| `journalctl` | System logs |
| `dmesg` | Kernel messages |

### Third-Party Tools
| Tool | Use Case |
|---|---|
| Wireshark | Deep packet inspection |
| SolarWinds | Network monitoring |
| Sysinternals Suite | Advanced Windows diagnostics |
| Nagios/Zabbix | Infrastructure monitoring |
| Splunk | Log aggregation and analysis |
| nmap | Network scanning |
