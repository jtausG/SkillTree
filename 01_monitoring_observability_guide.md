# IT Infrastructure Monitoring & Observability Guide

## Table of Contents
1. [Monitoring Strategy & Architecture](#monitoring-strategy)
2. [Windows Server Monitoring](#windows-monitoring)
3. [Network Monitoring](#network-monitoring)
4. [Application Performance Monitoring](#apm)
5. [Log Management](#log-management)
6. [Alerting Strategy](#alerting)
7. [Dashboards & Reporting](#dashboards)
8. [Azure Monitor & Log Analytics](#azure-monitor)
9. [SNMP & Infrastructure Tools](#snmp)
10. [Synthetic Monitoring](#synthetic)
11. [On-Call & Incident Response Integration](#oncall)
12. [PowerShell Monitoring Scripts](#ps-monitoring)

---

## 1. Monitoring Strategy & Architecture {#monitoring-strategy}

### The Four Golden Signals (Google SRE)
```
1. LATENCY    — Time to serve a request (distinguish success vs. error latency)
2. TRAFFIC    — Demand on your system (requests/sec, concurrent users)
3. ERRORS     — Rate of failed requests (explicit 5xx, implicit wrong data)
4. SATURATION — How full your system is (CPU%, disk%, queue depth)
```

### Observability Pillars
```
METRICS    — Numeric measurements over time (CPU%, response time ms, request count)
LOGS       — Timestamped records of events (application logs, audit logs, system events)
TRACES     — Request paths through distributed systems (spans, latency per service)
```

### Monitoring Tiers
| Tier | What to Monitor | Tools |
|------|----------------|-------|
| Infrastructure | CPU, RAM, disk, network interfaces | Zabbix, PRTG, Nagios, Azure Monitor |
| Network | Bandwidth, latency, packet loss, BGP | PRTG, SolarWinds, Grafana + Telegraf |
| Application | Response time, error rates, throughput | AppDynamics, New Relic, Azure App Insights |
| Security | Failed logins, firewall hits, anomalies | SIEM, Defender, Sentinel |
| Business | Transactions, SLA compliance, user experience | Custom dashboards, Dynatrace |

### Monitoring Architecture (On-Premises + Cloud)
```
                  ┌─────────────────────────────┐
                  │     CENTRALIZED PLATFORM     │
                  │  Grafana / Azure Monitor /   │
                  │  Prometheus + Alertmanager   │
                  └───────────┬─────────────────┘
                              │
        ┌─────────────────────┼──────────────────────┐
        │                     │                      │
┌───────▼──────┐   ┌──────────▼───────┐   ┌─────────▼────────┐
│  Servers     │   │  Network Devices │   │  Cloud (Azure)   │
│  (WMI/SNMP)  │   │  (SNMP/NetFlow)  │   │  (Diagnostic     │
│  Windows MMA │   │  Telegraf/Input  │   │   Settings)      │
└──────────────┘   └──────────────────┘   └──────────────────┘
```

---

## 2. Windows Server Monitoring {#windows-monitoring}

### Critical Metrics to Monitor
| Metric | Warning | Critical | Collection Method |
|--------|---------|----------|-------------------|
| CPU Usage (sustained) | > 80% | > 95% | WMI, Perfmon |
| Available Memory | < 20% | < 10% | WMI |
| Disk Space | < 20% | < 10% | WMI |
| Disk Queue Length | > 2 | > 5 | Perfmon |
| Network Errors | > 0.1% | > 1% | WMI |
| Windows Service (critical) | Stopped | — | WMI |
| Event Log Errors | Count > 10/hr | Count > 50/hr | Event Log |
| RDP/WinRM connectivity | Timeout | Unreachable | TCP check |

### Performance Monitor (Perfmon) Key Counters
```
Processor:
  \Processor(_Total)\% Processor Time              → CPU utilization
  \Processor(_Total)\% Interrupt Time               → Hardware interrupt load
  \System\Processor Queue Length                    → CPU bottleneck (> 2 per core = issue)

Memory:
  \Memory\Available MBytes                          → Free physical RAM
  \Memory\Pages/sec                                 → Paging activity (> 20 = issue)
  \Memory\% Committed Bytes In Use                  → Virtual memory pressure

Disk:
  \PhysicalDisk(_Total)\% Disk Time                 → Overall disk busy time
  \PhysicalDisk(_Total)\Avg. Disk Queue Length      → Disk queue (> 2 = issue)
  \PhysicalDisk(_Total)\Avg. Disk sec/Read          → Read latency (> 20ms = concern)
  \PhysicalDisk(_Total)\Avg. Disk sec/Write         → Write latency

Network:
  \Network Interface(*)\Bytes Total/sec             → Bandwidth usage
  \Network Interface(*)\Packets Received Errors     → Receive errors
  \Network Interface(*)\Output Queue Length         → Transmit congestion (> 2 = issue)

IIS:
  \Web Service(_Total)\Current Connections          → Active connections
  \Web Service(_Total)\Requests/sec                 → Request throughput
  \ASP.NET\Requests Queued                          → App pool queue depth
```

### Scheduled Performance Data Collector
```cmd
# Create a data collector set for baseline
logman create counter "ServerBaseline" -f csv -si 60 -v mmddhhmm -max 100 -cnf 24:00:00 ^
  -c "\Processor(_Total)\% Processor Time" ^
     "\Memory\Available MBytes" ^
     "\PhysicalDisk(_Total)\Avg. Disk Queue Length" ^
     "\Network Interface(*)\Bytes Total/sec" ^
  -o "C:\PerfLogs\ServerBaseline"

logman start "ServerBaseline"
logman stop "ServerBaseline"
```

### Windows Event Log Monitoring

#### Critical Event IDs to Alert On
```
Security Log:
  4625  — Failed logon (alert on > 5 in 5 min from same source)
  4648  — Explicit credential use (pass-the-hash indicator)
  4719  — System audit policy changed
  4720  — User account created
  4726  — User account deleted
  4728  — Member added to security-enabled global group
  4732  — Member added to security-enabled local group
  4740  — User account locked out
  4756  — Member added to universal security group
  4776  — DC attempted to validate credentials (failed = lockout source)

System Log:
  6008  — Unexpected shutdown
  7034  — Service crashed
  7036  — Service stopped
  7045  — New service installed (malware indicator)
  1074  — System shutdown initiated

Application Log:
  1000  — Application crash
  1001  — Application hang/crash (Windows Error Reporting)

Directory Service (DC only):
  2887  — LDAP signing not enforced (security risk)
  2886  — LDAP channel binding not enforced
```

```powershell
# Monitor for critical events in real-time
Register-EngineEvent -SourceIdentifier "PowerShell.Exiting" -Forward
$query = @"
<QueryList>
  <Query Id="0">
    <Select Path="Security">
      *[System[(EventID=4740 or EventID=4625 or EventID=7045) and 
      TimeCreated[timediff(@SystemTime) &lt;= 300000]]]
    </Select>
  </Query>
</QueryList>
"@

$events = Get-WinEvent -FilterXml $query
$events | Select-Object TimeCreated, Id, Message | Format-List
```

---

## 3. Network Monitoring {#network-monitoring}

### Interface & Bandwidth Monitoring

#### SNMP Polling Setup (Zabbix/PRTG)
```
Required on network devices:
  snmp-server community "MonitoringRO" RO
  snmp-server host 10.1.0.20 version 2c MonitoringRO    # Monitoring server IP
  
Key SNMP OIDs:
  ifDescr     .1.3.6.1.2.1.2.2.1.2     — Interface name
  ifOperStatus .1.3.6.1.2.1.2.2.1.8    — Interface up/down (1=up, 2=down)
  ifInOctets  .1.3.6.1.2.1.2.2.1.10    — Incoming bytes (counter)
  ifOutOctets .1.3.6.1.2.1.2.2.1.16    — Outgoing bytes (counter)
  ifInErrors  .1.3.6.1.2.1.2.2.1.14    — Receive errors
  ifOutErrors .1.3.6.1.2.1.2.2.1.20    — Transmit errors
  sysUpTime   .1.3.6.1.2.1.1.3.0       — Device uptime
```

#### NetFlow / sFlow Analysis
```
NetFlow collection setup (Cisco):
  ip flow-export version 9
  ip flow-export destination 10.1.0.20 2055   # Collector IP and port
  
  interface GigabitEthernet0/1
    ip flow ingress
    ip flow egress

Collectors: ntopng, Elastic + Kibana, SolarWinds NTA
Use cases:
  - Top talkers (who is using the most bandwidth?)
  - Application breakdown (what protocols/apps?)
  - Suspicious connections (unusual ports, destinations)
  - Bandwidth trending and capacity planning
```

### Ping & Latency Monitoring
```powershell
# Continuous ping monitoring with alerting
function Start-PingMonitor {
    param(
        [string[]]$Targets,
        [int]$Interval = 60,
        [int]$ThresholdMs = 50,
        [string]$LogFile = "C:\Monitoring\ping_log.csv"
    )
    
    while ($true) {
        foreach ($target in $Targets) {
            $ping = Test-Connection -ComputerName $target -Count 4 -ErrorAction SilentlyContinue
            if ($ping) {
                $avg = ($ping | Measure-Object ResponseTime -Average).Average
                $status = if ($avg -gt $ThresholdMs) {"SLOW"} else {"OK"}
            } else {
                $avg = -1
                $status = "DOWN"
                # Alert here
                Write-Warning "ALERT: $target is DOWN at $(Get-Date)"
            }
            
            [PSCustomObject]@{
                Timestamp   = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
                Target      = $target
                AvgLatencyMs = [math]::Round($avg,1)
                Status      = $status
            } | Export-Csv $LogFile -Append -NoTypeInformation
        }
        Start-Sleep -Seconds $Interval
    }
}

Start-PingMonitor -Targets "10.1.0.1","8.8.8.8","fileserver01" -ThresholdMs 50
```

### Network Topology Discovery
```powershell
# Discover active hosts on subnet
function Get-SubnetHosts {
    param([string]$Subnet = "10.1.1")
    
    $jobs = 1..254 | ForEach-Object {
        $ip = "$Subnet.$_"
        Start-Job -ScriptBlock {
            param($addr)
            if (Test-Connection -ComputerName $addr -Count 1 -Quiet) {
                $hostname = try { [System.Net.Dns]::GetHostEntry($addr).HostName } catch { "Unknown" }
                [PSCustomObject]@{ IP = $addr; Hostname = $hostname; Status = "Online" }
            }
        } -ArgumentList $ip
    }
    
    $jobs | Wait-Job | Receive-Job | Where-Object {$_} | Sort-Object {[version]$_.IP}
    $jobs | Remove-Job
}

Get-SubnetHosts -Subnet "10.1.1"
```

---

## 4. Application Performance Monitoring {#apm}

### IIS Web Application Monitoring
```powershell
# Get IIS application pool status
Import-Module WebAdministration
Get-WebConfiguration system.applicationHost/applicationPools/add | 
    Select-Object name, @{n="State";e={
        (Get-WebConfigurationProperty "system.applicationHost/applicationPools/add[@name='$($_.name)']" -name state).Value
    }}

# Recycle failed app pools automatically
Get-WebConfiguration system.applicationHost/applicationPools/add | ForEach-Object {
    $poolName = $_.name
    $state = (Get-WebConfigurationProperty "system.applicationHost/applicationPools/add[@name='$poolName']" -name state).Value
    if ($state -ne "Started") {
        Write-Warning "App Pool '$poolName' is $state — attempting restart"
        Start-WebAppPool -Name $poolName
    }
}
```

### SQL Server Monitoring Queries
```sql
-- Currently running queries and duration
SELECT 
    r.session_id,
    r.status,
    r.blocking_session_id,
    r.wait_type,
    r.wait_time / 1000 AS wait_seconds,
    r.cpu_time / 1000 AS cpu_seconds,
    r.total_elapsed_time / 1000 AS elapsed_seconds,
    DB_NAME(r.database_id) AS database_name,
    SUBSTRING(st.text, (r.statement_start_offset/2)+1, 
        ((CASE r.statement_end_offset WHEN -1 THEN DATALENGTH(st.text)
          ELSE r.statement_end_offset END - r.statement_start_offset)/2)+1) AS query_text
FROM sys.dm_exec_requests r
CROSS APPLY sys.dm_exec_sql_text(r.sql_handle) st
WHERE r.session_id > 50;

-- Database sizes
SELECT 
    name,
    size * 8 / 1024 AS size_mb,
    FILEPROPERTY(name, 'SpaceUsed') * 8 / 1024 AS used_mb
FROM sys.master_files
WHERE type = 0   -- data files only
ORDER BY size_mb DESC;

-- Index fragmentation
SELECT 
    OBJECT_NAME(ips.object_id) AS table_name,
    i.name AS index_name,
    ips.avg_fragmentation_in_percent,
    ips.page_count
FROM sys.dm_db_index_physical_stats(DB_ID(), NULL, NULL, NULL, 'LIMITED') ips
JOIN sys.indexes i ON ips.object_id = i.object_id AND ips.index_id = i.index_id
WHERE ips.avg_fragmentation_in_percent > 30 AND ips.page_count > 1000
ORDER BY ips.avg_fragmentation_in_percent DESC;
```

---

## 5. Log Management {#log-management}

### Centralized Log Collection Architecture
```
Sources → Collectors → Aggregator → Storage → Analysis

Sources:
  Windows: WEF (Windows Event Forwarding) → WEC (Collector)
  Linux: rsyslog/syslog-ng → Logstash
  Network: Syslog → Logstash/Graylog
  Apps: Custom agents → Kafka/Fluentd

Stack Options:
  ELK Stack: Elasticsearch + Logstash + Kibana
  Graylog:   Open source, good for on-prem
  Splunk:    Enterprise, expensive but powerful
  Azure:     Log Analytics Workspace
```

### Windows Event Forwarding (WEF) Setup
```powershell
# On Windows Event Collector server:
wecutil qc /q    # Configure WEC service

# Create subscription (pull all security events from domain)
# Save as: C:\WEF\SecurityEvents.xml then:
wecutil cs SecurityEvents.xml

# Subscription XML example:
<Subscription xmlns="http://schemas.microsoft.com/2006/03/windows/events/subscription">
  <SubscriptionId>SecurityEvents</SubscriptionId>
  <SubscriptionType>SourceInitiated</SubscriptionType>
  <Description>Security Events from all domain computers</Description>
  <Enabled>true</Enabled>
  <Uri>http://schemas.microsoft.com/wbem/wsman/1/windows/EventLog</Uri>
  <ConfigurationMode>MinLatency</ConfigurationMode>
  <Query><![CDATA[
    <QueryList>
      <Query Id="0">
        <Select Path="Security">*[System[EventID=4624 or EventID=4625 or EventID=4740]]</Select>
      </Query>
    </QueryList>
  ]]></Query>
  <AllowedSourceDomainComputers>O:NSG:NSD:(A;;GA;;;DC)</AllowedSourceDomainComputers>
</Subscription>
```

### Log Retention Policy
| Log Type | Retention | Reason |
|----------|-----------|--------|
| Security events | 12 months minimum | Incident investigation, compliance |
| Authentication logs | 12 months | SOC 2, ISO 27001 |
| Application logs | 90 days | Debugging |
| Network flow | 90 days | Security investigation |
| DNS queries | 30 days | Security/forensics |
| System logs | 30 days | Troubleshooting |
| Firewall logs | 90 days | Security, compliance |
| Email logs | 12 months | HR/legal investigations |

---

## 6. Alerting Strategy {#alerting}

### Alert Hierarchy
```
P1 — CRITICAL  (PagerDuty/phone call, 24/7 on-call)
  Examples: Production system down, security breach, data loss
  Response time: Immediate (< 5 min acknowledgment)
  
P2 — HIGH      (PagerDuty/Teams, business hours + on-call)
  Examples: Service degraded, replication failed, disk > 90%
  Response time: < 30 minutes
  
P3 — MEDIUM    (Teams/email)
  Examples: Backup failed, certificate expiring < 30 days, disk > 80%
  Response time: < 4 hours (business hours)
  
P4 — LOW       (Email/ticket)
  Examples: Informational, trends, non-urgent warnings
  Response time: Next business day
```

### Alert Fatigue Prevention
```
Anti-patterns (avoid):
  ✗ Alert on every threshold breach without time aggregation
  ✗ Same alert fires 50 times before someone acknowledges
  ✗ Informational alerts go to same channel as critical alerts
  ✗ Alerts with no runbook/context
  ✗ Alerting on metrics that don't indicate real problems

Best practices:
  ✓ Use sustained thresholds (> 80% for 5 consecutive minutes)
  ✓ Implement alert grouping and deduplication
  ✓ Every alert has a runbook link
  ✓ Separate alert channels by severity
  ✓ Weekly review of alert noise — tune or remove noisy alerts
  ✓ Auto-resolve alerts when condition clears
  ✓ Track alert frequency — high-frequency = automation candidate
```

### PagerDuty / On-Call Rotation Setup
```
Escalation Policy:
  Level 1: Primary on-call engineer (5 min to acknowledge)
  Level 2: Secondary on-call engineer (15 min if Level 1 unresponsive)
  Level 3: Engineering manager (30 min if Level 2 unresponsive)
  
Rotation: Weekly rotation, minimum 2 engineers per rotation
Schedule: Cover weekdays 24/7 + weekends for P1/P2 only
Compensation: On-call pay per policy, time off in lieu for incidents
```

---

## 7. Dashboards & Reporting {#dashboards}

### Operations Dashboard Checklist
```
Executive Dashboard (C-Suite):
  ✓ Overall system health (green/yellow/red)
  ✓ SLA compliance (% uptime last 30 days)
  ✓ Open incidents by severity
  ✓ MTTR trend
  ✓ Planned vs unplanned downtime

Operations Dashboard (IT Ops):
  ✓ Server CPU/memory/disk heatmap
  ✓ Network bandwidth utilization
  ✓ Active alerts by severity
  ✓ Service availability (last 24h)
  ✓ Backup job status
  ✓ Patch compliance %

Security Dashboard (Security Team):
  ✓ Failed login attempts (last hour/24h)
  ✓ Locked accounts
  ✓ New devices detected
  ✓ Security incidents open/closed
  ✓ Vulnerability scan results
  ✓ MFA coverage %
```

### Grafana Dashboard — Prometheus Metrics
```yaml
# prometheus.yml scrape config
scrape_configs:
  - job_name: 'windows_servers'
    static_configs:
      - targets: ['server01:9182', 'server02:9182']   # windows_exporter
    
  - job_name: 'network_devices'
    static_configs:
      - targets: ['10.1.0.1:161']    # SNMP exporter
    metrics_path: /snmp
    params:
      module: [if_mib]
      
  - job_name: 'blackbox'   # Ping/HTTP checks
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
          - https://intranet.corp.contoso.com
          - https://mail.corp.contoso.com
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - target_label: __address__
        replacement: blackbox_exporter:9115
```

---

## 8. Azure Monitor & Log Analytics {#azure-monitor}

### Key KQL Queries for Operations

```kql
// VM CPU over 80% for last 24 hours
Perf
| where TimeGenerated > ago(24h)
| where ObjectName == "Processor" and CounterName == "% Processor Time"
| where InstanceName == "_Total"
| summarize AvgCPU = avg(CounterValue), MaxCPU = max(CounterValue) by Computer, bin(TimeGenerated, 5m)
| where MaxCPU > 80
| order by MaxCPU desc

// Disk space trending
Perf
| where TimeGenerated > ago(7d)
| where ObjectName == "LogicalDisk" and CounterName == "% Free Space"
| where InstanceName !in ("_Total", "HarddiskVolume1")
| summarize AvgFree = avg(CounterValue) by Computer, InstanceName, bin(TimeGenerated, 1h)
| where AvgFree < 20
| order by AvgFree asc

// Windows service stopped
Event
| where TimeGenerated > ago(1h)
| where EventLog == "System" and EventID == 7036
| where RenderedDescription contains "stopped"
| project TimeGenerated, Computer, RenderedDescription

// Failed logons from multiple sources
SecurityEvent
| where TimeGenerated > ago(1h)
| where EventID == 4625
| summarize FailCount = count() by Account, IpAddress, bin(TimeGenerated, 5m)
| where FailCount > 5
| order by FailCount desc

// Certificate expiry (if logged to LA)
AzureDiagnostics
| where ResourceType == "APPLICATIONGATEWAYS"
| where Message contains "certificate"
| project TimeGenerated, Resource, Message
```

### Azure Monitor Alerts (Bicep)
```bicep
resource cpuAlert 'Microsoft.Insights/metricAlerts@2018-03-01' = {
  name: 'High-CPU-Alert'
  location: 'global'
  properties: {
    description: 'CPU > 80% for 5 minutes'
    severity: 2
    enabled: true
    scopes: [virtualMachineId]
    evaluationFrequency: 'PT1M'
    windowSize: 'PT5M'
    criteria: {
      'odata.type': 'Microsoft.Azure.Monitor.SingleResourceMultipleMetricCriteria'
      allOf: [{
        name: 'HighCPU'
        metricName: 'Percentage CPU'
        operator: 'GreaterThan'
        threshold: 80
        timeAggregation: 'Average'
      }]
    }
    actions: [{
      actionGroupId: actionGroupId
    }]
  }
}
```

---

## 9. SNMP & Infrastructure Tools {#snmp}

### SNMP v3 Configuration (Secure)
```
# Cisco device — SNMPv3 with auth and encryption
snmp-server group MonitorGroup v3 priv
snmp-server user MonitorUser MonitorGroup v3 auth sha AuthPass123! priv aes 128 PrivPass456!
snmp-server host 10.1.0.20 version 3 priv MonitorUser

# Test SNMPv3 from monitoring server
snmpget -v3 -u MonitorUser -l authPriv -a SHA -A AuthPass123! -x AES -X PrivPass456! 10.1.0.1 sysUpTime.0
```

### Key Monitoring Tools Comparison
| Tool | Type | Best For | License |
|------|------|---------|---------|
| **Zabbix** | Agent/Agentless | Mixed on-prem environments | Free/Open Source |
| **PRTG** | Agent/SNMP/WMI | Windows-heavy shops | Commercial |
| **Nagios** | Plugin-based | Unix/network | Open Source |
| **Grafana + Prometheus** | Metrics stack | Modern DevOps | Open Source |
| **Datadog** | SaaS | Cloud-first, APM | Commercial |
| **Azure Monitor** | SaaS | Azure-heavy environments | Azure pricing |
| **SolarWinds NPM** | Network focus | Network operations | Commercial |

---

## 10. Synthetic Monitoring {#synthetic}

### HTTP Endpoint Checks (PowerShell)
```powershell
function Test-WebEndpoint {
    param(
        [hashtable[]]$Endpoints,
        [int]$TimeoutSec = 10,
        [string]$ReportPath = "C:\Monitoring\web_checks.csv"
    )
    
    $results = foreach ($ep in $Endpoints) {
        $start = Get-Date
        try {
            $response = Invoke-WebRequest -Uri $ep.URL -TimeoutSec $TimeoutSec `
                -MaximumRedirection 5 -ErrorAction Stop
            $elapsed = ((Get-Date) - $start).TotalMilliseconds
            $status = if ($ep.ExpectedCode -and $response.StatusCode -ne $ep.ExpectedCode) {
                "WRONG_CODE"
            } elseif ($ep.ExpectedContent -and $response.Content -notmatch $ep.ExpectedContent) {
                "CONTENT_MISMATCH"
            } else { "OK" }
            
            [PSCustomObject]@{
                Timestamp    = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
                URL          = $ep.URL
                StatusCode   = $response.StatusCode
                ResponseMs   = [math]::Round($elapsed, 0)
                Status       = $status
                Error        = ""
            }
        } catch {
            [PSCustomObject]@{
                Timestamp    = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
                URL          = $ep.URL
                StatusCode   = 0
                ResponseMs   = -1
                Status       = "DOWN"
                Error        = $_.Exception.Message
            }
        }
    }
    
    $results | Export-Csv $ReportPath -Append -NoTypeInformation
    $results | Format-Table -AutoSize
    
    # Alert on failures
    $failures = $results | Where-Object Status -ne "OK"
    foreach ($fail in $failures) {
        Write-Warning "ALERT: $($fail.URL) — $($fail.Status) — $($fail.Error)"
    }
}

$endpoints = @(
    @{URL = "https://intranet.corp.contoso.com"; ExpectedCode = 200; ExpectedContent = "Welcome"}
    @{URL = "https://helpdesk.corp.contoso.com/health"; ExpectedCode = 200}
    @{URL = "https://mail.corp.contoso.com"; ExpectedCode = 200}
)

Test-WebEndpoint -Endpoints $endpoints
```

---

## 11. On-Call & Incident Response Integration {#oncall}

### Runbook Template for Each Alert
```markdown
## Alert: [ALERT NAME]

**Severity:** P1/P2/P3
**Team:** Infrastructure / Security / Application
**PagerDuty Policy:** [Policy name]

### Description
Brief description of what this alert means.

### Impact
What is affected if this condition continues.

### Initial Diagnosis
1. Check [specific thing] in [specific tool]
2. Run: `command to verify`
3. Check event logs for: [specific events]

### Resolution Steps
**If cause is X:**
  1. Do this
  2. Then this
  
**If cause is Y:**
  1. Do this
  
### Escalation
- If not resolved in 30 min → escalate to [Team/Person]
- If data loss risk → immediately involve [Manager]

### Post-Resolution
- Update ticket with root cause
- Schedule post-mortem if P1
```

---

## 12. PowerShell Monitoring Scripts {#ps-monitoring}

### Comprehensive Server Health Check
```powershell
function Get-ServerHealthReport {
    param([string[]]$Servers)
    
    $report = foreach ($server in $Servers) {
        if (-not (Test-Connection -ComputerName $server -Count 1 -Quiet)) {
            [PSCustomObject]@{ Server = $server; Status = "UNREACHABLE"; CPU = "N/A"; RAM = "N/A"; Disk = "N/A" }
            continue
        }
        
        try {
            $os = Get-WmiObject Win32_OperatingSystem -ComputerName $server -ErrorAction Stop
            $cpu = Get-WmiObject Win32_Processor -ComputerName $server | 
                   Measure-Object LoadPercentage -Average | Select-Object -ExpandProperty Average
            $disks = Get-WmiObject Win32_LogicalDisk -ComputerName $server -Filter "DriveType=3" |
                     Where-Object {($_.FreeSpace/$_.Size) -lt 0.20}
            
            $ramUsed = [math]::Round(($os.TotalVisibleMemorySize - $os.FreePhysicalMemory) / $os.TotalVisibleMemorySize * 100, 1)
            $diskAlert = if ($disks) { ($disks | ForEach-Object {"$($_.DeviceID) $([math]::Round($_.FreeSpace/$_.Size*100,1))% free"}) -join "; " } else { "OK" }
            
            [PSCustomObject]@{
                Server     = $server
                Status     = "Online"
                CPU_Pct    = "$cpu%"
                RAM_Used   = "$ramUsed%"
                Disk_Alert = $diskAlert
                Uptime_Days = [math]::Round($os.ConvertToDateTime($os.LastBootUpTime) | 
                              ForEach-Object {((Get-Date) - $_).TotalDays}, 1)
            }
        } catch {
            [PSCustomObject]@{ Server = $server; Status = "WMI_ERROR: $($_.Exception.Message)" }
        }
    }
    
    $report | Format-Table -AutoSize
    
    # Highlight issues
    Write-Host "`n=== ISSUES ===" -ForegroundColor Yellow
    $report | Where-Object {$_.CPU_Pct -gt 80 -or $_.RAM_Used -gt 90 -or $_.Disk_Alert -ne "OK" -or $_.Status -ne "Online"} |
        Format-Table -AutoSize
}

Get-ServerHealthReport -Servers (Get-ADComputer -Filter {OperatingSystem -like "*Server*"} | Select-Object -ExpandProperty Name)
```

### Certificate Expiry Monitoring
```powershell
function Get-ExpiringCertificates {
    param(
        [string[]]$Servers,
        [int]$DaysWarning = 30,
        [int]$DaysCritical = 14
    )
    
    foreach ($server in $Servers) {
        $certs = Invoke-Command -ComputerName $server -ScriptBlock {
            Get-ChildItem Cert:\LocalMachine\My | 
                Where-Object {$_.NotAfter -gt (Get-Date) -and $_.NotAfter -lt (Get-Date).AddDays($using:DaysWarning)} |
                Select-Object Subject, Thumbprint, NotAfter, 
                    @{n="DaysLeft";e={[math]::Round(($_.NotAfter - (Get-Date)).TotalDays,0)}}
        }
        
        foreach ($cert in $certs) {
            $severity = if ($cert.DaysLeft -le $DaysCritical) {"CRITICAL"} else {"WARNING"}
            [PSCustomObject]@{
                Server   = $server
                Subject  = $cert.Subject
                Expires  = $cert.NotAfter.ToString("yyyy-MM-dd")
                DaysLeft = $cert.DaysLeft
                Severity = $severity
            }
        }
    }
}

Get-ExpiringCertificates -Servers "webserver01","exchangeserver","rdgateway" | 
    Sort-Object DaysLeft | Format-Table -AutoSize
```

---

*Last Updated: 2025 | Tools: Windows Server 2019/2022, Azure Monitor, Prometheus/Grafana, Zabbix*
