# DNS & DHCP Enterprise Administration Guide

## Table of Contents
1. [DNS Architecture](#dns-architecture)
2. [DNS Zone Management](#dns-zones)
3. [DNS Records Reference](#dns-records)
4. [DNS Security (DNSSEC)](#dnssec)
5. [DNS Troubleshooting](#dns-troubleshooting)
6. [DHCP Server Setup & Management](#dhcp-setup)
7. [DHCP Scopes & Options](#dhcp-scopes)
8. [DHCP High Availability](#dhcp-ha)
9. [DHCP Troubleshooting](#dhcp-troubleshooting)
10. [PowerShell Automation](#dns-dhcp-powershell)
11. [Split DNS / Split-Brain DNS](#split-dns)
12. [DNS Integration with AD](#dns-ad-integration)

---

## 1. DNS Architecture {#dns-architecture}

### DNS Resolution Flow
```
Client Query: "www.contoso.com"
  │
  ▼
1. Client DNS cache check
  │ (miss)
  ▼
2. Local DNS Resolver (AD-integrated DC)
  │ (authoritative for contoso.com — answers directly)
  │ (external — forwards to)
  ▼
3. Forwarder (e.g., 8.8.8.8 or internal recursive resolver)
  │
  ▼
4. Root hints → .com TLD → contoso.com authoritative NS
```

### AD-Integrated DNS (Best Practice)
- DNS zones stored in Active Directory, not flat files
- Replicated via AD replication (more efficient than zone transfers)
- Dynamic updates secured by Kerberos
- Fault-tolerant — all DCs with DNS role are authoritative

### DNS Zone Types
| Type | Storage | Replication | Use Case |
|------|---------|-------------|----------|
| Primary | Zone file or AD | Manual transfer or AD | Main writable zone |
| Secondary | Read-only zone file | Zone transfer from primary | Redundancy, remote sites |
| Stub | NS/SOA/A records only | Zone transfer | Delegation pointer |
| Forwarder | None (forwards queries) | N/A | External resolution |
| AD-Integrated | AD database | AD replication | Best practice for internal |

### DNS Replication Scope (AD-Integrated)
| Scope | Replicated To | Use Case |
|-------|--------------|----------|
| Domain DNS Zone | All DCs in domain | Default for internal zones |
| Forest DNS Zone | All DCs in forest | _msdcs zone, multi-domain |
| Domain | All DCs in domain | Legacy |
| Custom application partition | Specified DCs | Granular control |

---

## 2. DNS Zone Management {#dns-zones}

### Creating Zones (PowerShell)
```powershell
# Create primary AD-integrated zone
Add-DnsServerPrimaryZone -Name "corp.contoso.com" `
    -ReplicationScope Forest `
    -DynamicUpdate Secure

# Create reverse lookup zone for 192.168.10.0/24
Add-DnsServerPrimaryZone -NetworkID "192.168.10.0/24" `
    -ReplicationScope Domain `
    -DynamicUpdate Secure

# Create stub zone
Add-DnsServerStubZone -Name "subsidiary.com" `
    -MasterServers 10.5.0.10, 10.5.0.11 `
    -ReplicationScope Domain

# Create conditional forwarder
Add-DnsServerConditionalForwarderZone -Name "partner.com" `
    -MasterServers 203.0.113.10 `
    -ReplicationScope Forest

# Configure forwarders (for external resolution)
Set-DnsServerForwarder -IPAddress 8.8.8.8, 8.8.4.4 -UseRootHint $true
```

### Zone Transfer Configuration
```powershell
# Allow zone transfer to specific servers only
Set-DnsServerPrimaryZone -Name "corp.contoso.com" `
    -SecureSecondaries TransferToSecureServers `
    -SecondaryServers 10.1.0.11, 10.1.0.12

# Notify secondary servers of changes
Set-DnsServerPrimaryZone -Name "corp.contoso.com" `
    -Notify NotifyServers `
    -NotifyServers 10.1.0.11, 10.1.0.12
```

---

## 3. DNS Records Reference {#dns-records}

### Record Types Cheat Sheet
| Type | Purpose | Example |
|------|---------|---------|
| **A** | Hostname → IPv4 | www → 10.1.1.10 |
| **AAAA** | Hostname → IPv6 | www → 2001:db8::1 |
| **CNAME** | Alias → canonical name | mail → exchange01 |
| **MX** | Mail server | @ → mail.contoso.com (priority 10) |
| **PTR** | IPv4 → hostname (reverse) | 10.10.1.192 → server01.corp.contoso.com |
| **NS** | Authoritative name servers | contoso.com → ns1.contoso.com |
| **SOA** | Zone authority info | Serial, refresh, retry, expire, TTL |
| **SRV** | Service location | _ldap._tcp → dc01.corp.contoso.com:389 |
| **TXT** | Text (SPF, DKIM, verification) | @ → "v=spf1 include:spf.protection.outlook.com -all" |
| **DNAME** | Delegate subdomain | legacy.contoso.com → contoso.com |

### Managing DNS Records (PowerShell)
```powershell
# Add A record
Add-DnsServerResourceRecordA -ZoneName "corp.contoso.com" `
    -Name "webserver01" -IPv4Address "10.1.1.50" -TimeToLive 01:00:00

# Add CNAME
Add-DnsServerResourceRecordCName -ZoneName "corp.contoso.com" `
    -Name "www" -HostNameAlias "webserver01.corp.contoso.com"

# Add MX record
Add-DnsServerResourceRecordMX -ZoneName "contoso.com" `
    -Name "@" -MailExchange "mail.contoso.com" -Preference 10

# Add TXT record (SPF)
Add-DnsServerResourceRecord -ZoneName "contoso.com" -Txt `
    -Name "@" -DescriptiveText "v=spf1 include:spf.protection.outlook.com -all"

# Add PTR record
Add-DnsServerResourceRecordPtr -ZoneName "1.168.192.in-addr.arpa" `
    -Name "50" -PtrDomainName "webserver01.corp.contoso.com"

# Add SRV record
Add-DnsServerResourceRecord -ZoneName "corp.contoso.com" -Srv `
    -Name "_http._tcp" -DomainName "webserver01.corp.contoso.com" `
    -Priority 10 -Weight 20 -Port 80

# Get all records in zone
Get-DnsServerResourceRecord -ZoneName "corp.contoso.com" | Sort-Object RecordType, HostName

# Remove record
Remove-DnsServerResourceRecord -ZoneName "corp.contoso.com" `
    -RRType A -Name "oldserver" -RecordData "10.1.1.99" -Force

# Update existing record (change IP)
$old = Get-DnsServerResourceRecord -ZoneName "corp.contoso.com" -Name "webserver01" -RRType A
$new = $old.Clone()
$new.RecordData.IPv4Address = [System.Net.IPAddress]"10.1.1.51"
Set-DnsServerResourceRecord -ZoneName "corp.contoso.com" -OldInputObject $old -NewInputObject $new
```

### Critical AD DNS SRV Records
```
# These must exist for AD to function:
_ldap._tcp.corp.contoso.com          → DCs (389)
_ldap._tcp.dc._msdcs.corp.contoso.com → DCs
_kerberos._tcp.corp.contoso.com      → DCs (88)
_gc._tcp.corp.contoso.com            → GC servers (3268)
_kpasswd._tcp.corp.contoso.com       → PDC (464)
_ldap._tcp.pdc._msdcs.corp.contoso.com → PDC emulator

# Verify SRV records exist
nslookup -type=srv _ldap._tcp.corp.contoso.com
Resolve-DnsName -Name "_ldap._tcp.corp.contoso.com" -Type SRV
```

---

## 4. DNS Security (DNSSEC) {#dnssec}

### DNSSEC Overview
DNSSEC adds cryptographic signatures to DNS records to prevent cache poisoning and spoofing.

```powershell
# Sign a zone with DNSSEC
$params = @{
    ZoneName            = "corp.contoso.com"
    KeyMasterServer     = "DC01"
    NSec3OptOut         = $false
    NSec3Iterations     = 50
    NSec3SaltLength     = 8
}
Invoke-DnsServerZoneSign @params

# Check DNSSEC status
Get-DnsServerZone -Name "corp.contoso.com" | Select-Object ZoneName, IsSigned

# View DNSSEC keys
Get-DnsServerDnsSecZoneSetting -ZoneName "corp.contoso.com"

# Unsign zone
Invoke-DnsServerZoneUnsign -ZoneName "corp.contoso.com" -Force
```

### DNS Security Best Practices
```
✓ Use AD-integrated zones with Secure Only dynamic updates
✓ Disable DNS recursion on authoritative servers
✓ Restrict zone transfers to specific secondary DNS servers
✓ Enable DNS audit logging (event ID 541-542)
✓ Monitor for DNS cache poisoning (unusual NXDOMAIN rates)
✓ Block DNS over non-standard ports at firewall
✓ Use DNS over HTTPS (DoH) or DNS over TLS for clients
✓ Enable DNS logging for security monitoring
```

### Enable DNS Audit Logging
```powershell
# Enable diagnostic logging
Set-DnsServerDiagnostics -All $true

# Or selectively
Set-DnsServerDiagnostics `
    -Queries $true `
    -Answers $true `
    -SendPackets $true `
    -TcpInformation $true `
    -EventLogLevel 7 `
    -LogFilePath "C:\DNS_Logs\dns_debug.log" `
    -MaxMBFileSize 500
```

---

## 5. DNS Troubleshooting {#dns-troubleshooting}

### Diagnostic Commands
```cmd
# Basic resolution test
nslookup server01.corp.contoso.com
nslookup server01.corp.contoso.com 10.1.0.10    # Query specific DNS server

# Reverse lookup
nslookup 10.1.1.50

# SRV records
nslookup -type=srv _ldap._tcp.corp.contoso.com

# SOA record
nslookup -type=soa corp.contoso.com

# All records
nslookup -type=any corp.contoso.com
```

```powershell
# PowerShell DNS tests
Resolve-DnsName -Name "server01.corp.contoso.com"
Resolve-DnsName -Name "server01.corp.contoso.com" -Server "10.1.0.10" -Type A
Resolve-DnsName -Name "_ldap._tcp.corp.contoso.com" -Type SRV

# Clear DNS cache (client)
Clear-DnsClientCache
ipconfig /flushdns

# Clear DNS server cache
Clear-DnsServerCache -Force

# Test DNS server connectivity
Test-NetConnection -ComputerName "10.1.0.10" -Port 53
```

### Common DNS Issues
| Problem | Symptoms | Resolution |
|---------|----------|------------|
| Stale DNS records | Clients resolve to old IP | Enable scavenging, manually remove stale records |
| Dynamic DNS not updating | Hostname doesn't resolve | Check DHCP-DNS integration, re-register: `ipconfig /registerdns` |
| Split DNS misconfigured | Internal vs external mismatch | Verify internal zone has all needed records |
| Zone not replicating | Different DCs return different results | Check `repadmin /showrepl`, verify DNS replication scope |
| NXDOMAIN for AD resources | Logon failures | Verify DC locator SRV records exist |
| Slow DNS resolution | Long pause before resolution | Check forwarder availability, disable root hints fallback |

### DNS Scavenging (Stale Record Cleanup)
```powershell
# Enable scavenging on DNS server
Set-DnsServerScavenging -ScavengingState $true -ScavengingInterval 7.00:00:00

# Configure scavenging on zone
Set-DnsServerZoneAging -ZoneName "corp.contoso.com" `
    -Aging $true `
    -NoRefreshInterval 7.00:00:00 `
    -RefreshInterval 7.00:00:00

# Manually trigger scavenging
Start-DnsServerScavenging -Force

# View records that would be scavenged
Get-DnsServerResourceRecord -ZoneName "corp.contoso.com" |
    Where-Object {$_.TimeStamp -ne $null -and $_.TimeStamp -lt (Get-Date).AddDays(-14)} |
    Select-Object HostName, RecordType, TimeStamp
```

---

## 6. DHCP Server Setup & Management {#dhcp-setup}

### Install DHCP Server Role
```powershell
# Install role
Install-WindowsFeature -Name DHCP -IncludeManagementTools

# Authorize DHCP server in AD (required for AD environments)
Add-DhcpServerInDC -DnsName "DHCPSERVER01.corp.contoso.com" -IPAddress 10.1.0.5

# Verify authorization
Get-DhcpServerInDC

# Complete post-install configuration
Set-ItemProperty -Path registry::HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\ServerManager\Roles\12 `
    -Name ConfigurationState -Value 2
```

### DHCP Server Settings
```powershell
# Configure DNS dynamic update credentials
$cred = Get-Credential
Set-DhcpServerDnsCredential -Credential $cred

# Set DNS dynamic update behavior
Set-DhcpServerv4DnsSetting `
    -DynamicUpdates Always `
    -DeleteDnsRROnLeaseExpiry $true `
    -UpdateDnsRRForOlderClients $true `
    -DisableDnsPtrRRUpdate $false `
    -NameProtection $true

# Enable DHCP audit logging
Set-DhcpServerAuditLog -Enable $true -Path "C:\Windows\System32\DHCP"
```

---

## 7. DHCP Scopes & Options {#dhcp-scopes}

### Create IPv4 Scope
```powershell
# Create scope
Add-DhcpServerv4Scope `
    -Name "Corporate LAN - VLAN 10" `
    -StartRange 10.1.10.100 `
    -EndRange 10.1.10.250 `
    -SubnetMask 255.255.255.0 `
    -State Active `
    -LeaseDuration 8.00:00:00 `
    -Description "VLAN 10 - Corporate Workstations"

# Add exclusions (for static IPs within the range)
Add-DhcpServerv4ExclusionRange `
    -ScopeId 10.1.10.0 `
    -StartRange 10.1.10.100 `
    -EndRange 10.1.10.120

# Set scope options
Set-DhcpServerv4OptionValue -ScopeId 10.1.10.0 `
    -DnsServer 10.1.0.10, 10.1.0.11 `
    -DnsDomain "corp.contoso.com" `
    -Router 10.1.10.1

# Option code reference:
# 003 = Router (Default Gateway)
# 006 = DNS Servers
# 015 = DNS Domain Name
# 044 = WINS Servers
# 066 = TFTP Server (PXE Boot)
# 067 = Bootfile Name (PXE)
# 119 = DNS Search Domains

# Add custom option (e.g., PXE boot)
Set-DhcpServerv4OptionValue -ScopeId 10.1.10.0 -OptionId 66 -Value "10.1.0.20"
Set-DhcpServerv4OptionValue -ScopeId 10.1.10.0 -OptionId 67 -Value "boot\x64\wdsnbp.com"
```

### DHCP Reservations
```powershell
# Add reservation by MAC address
Add-DhcpServerv4Reservation `
    -ScopeId 10.1.10.0 `
    -IPAddress 10.1.10.50 `
    -ClientId "AA-BB-CC-DD-EE-FF" `
    -Name "Printer-Finance-01" `
    -Description "Finance dept HP LaserJet"

# Bulk import reservations from CSV
# CSV format: IPAddress,MAC,Name,Description
Import-Csv "C:\DHCP\reservations.csv" | ForEach-Object {
    Add-DhcpServerv4Reservation `
        -ScopeId ($_.IPAddress -replace '\.\d+$', '.0') `
        -IPAddress $_.IPAddress `
        -ClientId $_.MAC `
        -Name $_.Name `
        -Description $_.Description
}

# Export all leases and reservations
Get-DhcpServerv4Lease -ScopeId 10.1.10.0 | 
    Select-Object IPAddress, ClientId, HostName, LeaseExpiryTime, AddressState |
    Export-Csv "C:\DHCP\leases_export.csv" -NoTypeInformation
```

### DHCP Policies (Conditional Options)
```powershell
# Policy: Different settings for MAC vendor (e.g., VoIP phones)
Add-DhcpServerv4Policy `
    -Name "VoIP Phones" `
    -ScopeId 10.1.10.0 `
    -Condition OR `
    -MacAddress EQ,"00-1A-E8-*"     # Cisco IP phone MAC prefix

Set-DhcpServerv4OptionValue `
    -ScopeId 10.1.10.0 `
    -PolicyName "VoIP Phones" `
    -OptionId 3 -Value 10.1.10.254  # Different gateway for VoIP
```

### Server-Level Options (Apply to All Scopes)
```powershell
# Set server-wide DNS settings
Set-DhcpServerv4OptionValue `
    -DnsServer 10.1.0.10, 10.1.0.11 `
    -DnsDomain "corp.contoso.com"

# These are overridden by scope-level options
```

---

## 8. DHCP High Availability {#dhcp-ha}

### DHCP Failover (Windows Server 2012+)
```powershell
# Add failover partnership (Hot Standby mode — one active, one standby)
Add-DhcpServerv4Failover `
    -Name "DHCP-Failover-Corp" `
    -PartnerServer "DHCPSERVER02.corp.contoso.com" `
    -ScopeId 10.1.10.0, 10.1.20.0 `
    -Mode HotStandby `
    -AutoStateTransition $true `
    -MaxClientLeadTime 2:00:00 `
    -SharedSecret "P@ssw0rd123!"

# Load Balance mode (both active — better for large deployments)
Add-DhcpServerv4Failover `
    -Name "DHCP-Failover-Corp" `
    -PartnerServer "DHCPSERVER02.corp.contoso.com" `
    -ScopeId 10.1.10.0 `
    -Mode LoadBalance `
    -LoadBalancePercent 50 `
    -SharedSecret "P@ssw0rd123!"

# Replicate scope to partner
Invoke-DhcpServerv4FailoverReplication -Name "DHCP-Failover-Corp" -Force

# Check failover status
Get-DhcpServerv4Failover

# Remove failover (to re-configure)
Remove-DhcpServerv4Failover -Name "DHCP-Failover-Corp" -Force
```

### DHCP Database Backup & Restore
```powershell
# Backup DHCP database
Backup-DhcpServer -Path "C:\DHCPBackup"

# Restore DHCP database
Restore-DhcpServer -Path "C:\DHCPBackup" -Force

# Export specific scope (for migration)
Export-DhcpServer -Leases -File "C:\DHCP\full_export.xml" -Force

# Import on new server
Import-DhcpServer -File "C:\DHCP\full_export.xml" -BackupPath "C:\DHCP\backup" -ScopeId 10.1.10.0 -Force

# Automatically backs up every 60 minutes to:
# C:\Windows\System32\DHCP\Backup
```

---

## 9. DHCP Troubleshooting {#dhcp-troubleshooting}

### Common Issues
| Issue | Cause | Resolution |
|-------|-------|------------|
| Client gets 169.254.x.x APIPA | No DHCP response | Check DHCP server auth, scope active, IP in range |
| Scope exhausted | Too many leases | Add exclusions, shorten lease time, expand scope |
| DNS not updating | DNSUpdateProxy missing perms | Add DHCP service account to DnsUpdateProxy group |
| Wrong options given | Wrong scope selected | Verify client VLAN/subnet matches scope |
| Duplicate IPs | Static IP in DHCP range | Add exclusion or reservation |
| Client gets wrong gateway | Scope option conflict | Verify scope-level vs server-level options |

### DHCP Diagnostic Commands
```cmd
# Release and renew on client
ipconfig /release
ipconfig /renew

# Show current DHCP lease
ipconfig /all | findstr -i "DHCP\|IP\|Gateway\|DNS"

# Check DHCP server event log
eventvwr /c: Microsoft-Windows-Dhcp-Server/Operational
```

```powershell
# Check DHCP lease for specific host
Get-DhcpServerv4Lease -ScopeId 10.1.10.0 | Where-Object HostName -like "*LAPTOP01*"

# Find who has a specific IP
Get-DhcpServerv4Lease -ScopeId 10.1.10.0 | Where-Object IPAddress -eq "10.1.10.150"

# Show scope utilization
Get-DhcpServerv4ScopeStatistics | Select-Object ScopeId, Free, InUse, Reserved, Pending, PercentageInUse

# Find scopes over 80% utilization
Get-DhcpServerv4ScopeStatistics | Where-Object PercentageInUse -gt 80

# Check for DHCP conflicts
Get-DhcpServerv4Lease -ScopeId 10.1.10.0 | Where-Object AddressState -eq "ActiveReservation"
```

---

## 10. PowerShell Automation {#dns-dhcp-powershell}

### Weekly DHCP Utilization Report
```powershell
$report = Get-DhcpServerv4ScopeStatistics | ForEach-Object {
    $scope = Get-DhcpServerv4Scope -ScopeId $_.ScopeId
    [PSCustomObject]@{
        ScopeName       = $scope.Name
        ScopeId         = $_.ScopeId
        TotalAddresses  = $_.Total
        InUse           = $_.InUse
        Available       = $_.Free
        PercentUsed     = [math]::Round($_.PercentageInUse, 1)
        Status          = if ($_.PercentageInUse -gt 90) {"CRITICAL"} 
                          elseif ($_.PercentageInUse -gt 75) {"WARNING"} 
                          else {"OK"}
    }
}

$report | Sort-Object PercentUsed -Descending | Format-Table -AutoSize
$report | Export-Csv "C:\Reports\DHCP_Utilization_$(Get-Date -f yyyyMMdd).csv" -NoTypeInformation

# Email alert for critical scopes
$critical = $report | Where-Object Status -eq "CRITICAL"
if ($critical) {
    Send-MailMessage -To "itops@corp.contoso.com" -From "monitoring@corp.contoso.com" `
        -Subject "DHCP CRITICAL: Scope Exhaustion Imminent" `
        -Body ($critical | Out-String) -SmtpServer "smtp.corp.contoso.com"
}
```

### DNS Health Check Script
```powershell
function Test-DnsHealth {
    param([string[]]$DnsServers, [string]$Domain)
    
    foreach ($server in $DnsServers) {
        Write-Host "`nTesting DNS Server: $server" -ForegroundColor Cyan
        
        # Test forward lookup
        try {
            $result = Resolve-DnsName -Name "dc01.$Domain" -Server $server -ErrorAction Stop
            Write-Host "  Forward Lookup: OK ($($result.IPAddress))" -ForegroundColor Green
        } catch {
            Write-Host "  Forward Lookup: FAILED" -ForegroundColor Red
        }
        
        # Test AD SRV records
        try {
            $srv = Resolve-DnsName -Name "_ldap._tcp.$Domain" -Type SRV -Server $server -ErrorAction Stop
            Write-Host "  AD SRV Records: OK ($($srv.Count) records)" -ForegroundColor Green
        } catch {
            Write-Host "  AD SRV Records: FAILED" -ForegroundColor Red
        }
        
        # Test external resolution (via forwarder)
        try {
            $ext = Resolve-DnsName -Name "www.google.com" -Server $server -ErrorAction Stop
            Write-Host "  External Resolution: OK" -ForegroundColor Green
        } catch {
            Write-Host "  External Resolution: FAILED (forwarder issue?)" -ForegroundColor Red
        }
        
        # Check response time
        $start = Get-Date
        Resolve-DnsName -Name $Domain -Server $server -ErrorAction SilentlyContinue | Out-Null
        $elapsed = ((Get-Date) - $start).TotalMilliseconds
        $color = if ($elapsed -gt 100) {"Yellow"} else {"Green"}
        Write-Host "  Response Time: $([math]::Round($elapsed,1))ms" -ForegroundColor $color
    }
}

Test-DnsHealth -DnsServers "10.1.0.10","10.1.0.11" -Domain "corp.contoso.com"
```

---

## 11. Split DNS / Split-Brain DNS {#split-dns}

### Use Case
Internal users resolve `mail.contoso.com` → internal IP (10.1.5.10)
External users resolve `mail.contoso.com` → public IP (203.0.113.50)

### Configuration
```powershell
# Create internal zone (same name as public zone)
Add-DnsServerPrimaryZone -Name "contoso.com" `
    -ReplicationScope Domain `
    -DynamicUpdate NonsecureAndSecure

# Add internal record
Add-DnsServerResourceRecordA -ZoneName "contoso.com" `
    -Name "mail" -IPv4Address "10.1.5.10"

# External DNS (at registrar or DMZ DNS) has:
# mail.contoso.com → 203.0.113.50

# Result: Internal DNS answers with internal IP, external DNS with public IP
```

### DNS Policy (Windows Server 2016+) — More Granular
```powershell
# Create client subnets
Add-DnsServerClientSubnet -Name "InternalSubnets" -IPv4Subnet "10.0.0.0/8","172.16.0.0/12"
Add-DnsServerClientSubnet -Name "ExternalSubnets" -IPv4Subnet "0.0.0.0/0"

# Create zone scopes
Add-DnsServerZoneScope -ZoneName "contoso.com" -Name "InternalScope"
Add-DnsServerZoneScope -ZoneName "contoso.com" -Name "ExternalScope"

# Add records to each scope
Add-DnsServerResourceRecord -ZoneName "contoso.com" -ZoneScope "InternalScope" `
    -A -Name "mail" -IPv4Address "10.1.5.10"
Add-DnsServerResourceRecord -ZoneName "contoso.com" -ZoneScope "ExternalScope" `
    -A -Name "mail" -IPv4Address "203.0.113.50"

# Create resolution policies
Add-DnsServerQueryResolutionPolicy -Name "InternalPolicy" `
    -Action ALLOW -ClientSubnet "EQ,InternalSubnets" `
    -ZoneScope "InternalScope,1" -ZoneName "contoso.com" -ProcessingOrder 1

Add-DnsServerQueryResolutionPolicy -Name "ExternalPolicy" `
    -Action ALLOW -ClientSubnet "EQ,ExternalSubnets" `
    -ZoneScope "ExternalScope,1" -ZoneName "contoso.com" -ProcessingOrder 2
```

---

## 12. DNS Integration with AD {#dns-ad-integration}

### Key Considerations
- DNS must be healthy for AD to function — authentication, replication, and GPO all depend on DNS
- Always use AD-integrated zones
- DCs should point to themselves as primary DNS, another DC as secondary — NEVER external DNS as primary on a DC

### DC DNS Configuration Best Practice
```
DC01 DNS settings:
  Primary:   127.0.0.1 (itself) — for fast resolution
  Secondary: 10.1.0.11 (DC02)

DC02 DNS settings:
  Primary:   10.1.0.10 (DC01)
  Secondary: 127.0.0.1 (itself)

Workstations:
  Primary:   10.1.0.10 (DC01)
  Secondary: 10.1.0.11 (DC02)
  Never use: 8.8.8.8 as primary (will break AD authentication)
```

### Verify AD DNS Health
```powershell
# Run AD DNS diagnostics
dcdiag /test:DNS /v /c /e /s:DC01

# Check for DNS errors in AD
repadmin /showrepl * /errorsonly

# Verify all DCs registered in DNS
Get-ADDomainController -Filter * | ForEach-Object {
    $dc = $_.HostName
    try {
        $r = Resolve-DnsName $dc -ErrorAction Stop
        Write-Host "$dc → $($r.IPAddress)" -ForegroundColor Green
    } catch {
        Write-Host "$dc → RESOLUTION FAILED" -ForegroundColor Red
    }
}

# Check netlogon DNS registrations
nltest /dsgetdc:corp.contoso.com
nltest /dnsgetdc:corp.contoso.com
```

---

*Last Updated: 2025 | Applies to: Windows Server 2019/2022 DNS/DHCP Roles*
