# Infrastructure Management Guide

## Overview
Reference for managing enterprise server infrastructure — covering virtualization, storage, backup, monitoring, capacity planning, and physical/cloud hybrid environments.

---

## Table of Contents
1. [Server Infrastructure Fundamentals](#servers)
2. [Virtualization (VMware & Hyper-V)](#virtualization)
3. [Storage Management](#storage)
4. [Backup & Recovery Operations](#backup)
5. [Monitoring & Alerting](#monitoring)
6. [Capacity Planning](#capacity)
7. [Infrastructure as Code (IaC)](#iac)
8. [Cloud Infrastructure (Azure & AWS)](#cloud)
9. [Patch & Configuration Management](#patching)
10. [Physical Infrastructure (Datacenter/Server Room)](#physical)

---

## 1. Server Infrastructure Fundamentals <a name="servers"></a>

### Server Role Classification

```
Tier 0 — Mission Critical (RTO: <1hr, RPO: <1hr)
  - Domain Controllers
  - Primary authentication systems (ADFS, Entra Connect)
  - Core network services (DNS, DHCP)
  - PKI / Certificate Authority

Tier 1 — Business Critical (RTO: <4hr, RPO: <4hr)
  - ERP / CRM systems
  - Email servers
  - File servers
  - Primary databases (prod)
  - Core business applications

Tier 2 — Important (RTO: <24hr, RPO: <8hr)
  - Secondary applications
  - Reporting / BI servers
  - Dev/Test environments
  - Secondary file servers

Tier 3 — Non-Critical (RTO: 72hr, RPO: 24hr)
  - Archival systems
  - Non-production systems
  - Training environments
```

### Server Naming Convention
```
Format: [SITE][TYPE][OS][FUNCTION][NUMBER]
Examples:
  NYDC01    → New York, Domain Controller, #01
  SFSRV001  → San Francisco, Server, #001
  LON-WSQL-01 → London, Windows SQL Server, #01
  AZU-LWEB-01  → Azure, Linux Web Server, #01

Type codes:
  DC = Domain Controller
  SRV = General Server
  SQL = SQL Server
  WEB = Web Server
  APP = Application Server
  FILE = File Server
  MON = Monitoring
  JUMP = Jump Server / Bastion
  
OS codes:
  W = Windows
  L = Linux
  (omit for mixed or not relevant)
```

---

## 2. Virtualization (VMware & Hyper-V) <a name="virtualization"></a>

### VMware vSphere Management

```powershell
# PowerCLI — Connect to vCenter
Connect-VIServer -Server vcenter.domain.com -Credential (Get-Credential)

# List all VMs with resource usage
Get-VM | Select-Object Name, PowerState, NumCPU, MemoryGB, 
  @{N='UsedSpaceGB';E={[Math]::Round($_.UsedSpaceGB,1)}}, VMHost | 
  Sort-Object VMHost | Format-Table

# Find powered-off VMs (potential cost savings)
Get-VM | Where-Object {$_.PowerState -eq "PoweredOff"} | 
  Select-Object Name, Notes, @{N='LastModified';E={(Get-VIEvent -Entity $_ -MaxSamples 1).CreatedTime}}

# Find VMs with snapshots (performance risk)
Get-VM | Get-Snapshot | Select-Object VM, Name, Created, SizeGB |
  Sort-Object SizeGB -Descending

# Remove all snapshots for a VM (CAUTION)
Get-VM "ServerName" | Get-Snapshot | Remove-Snapshot -Confirm:$true

# VM resource allocation
Get-VM "ServerName" | Get-VMResourceConfiguration | 
  Select-Object VM, CpuReservationMhz, MemReservationMB, DiskLimit

# Find overcommitted hosts
Get-VMHost | Select-Object Name, 
  @{N='CPUUsage%';E={[Math]::Round(($_.CpuUsageMhz/$_.CpuTotalMhz)*100,1)}},
  @{N='MemUsage%';E={[Math]::Round(($_.MemoryUsageGB/$_.MemoryTotalGB)*100,1)}} | 
  Where-Object {$_.'CPUUsage%' -gt 80 -or $_.'MemUsage%' -gt 85}
```

### VMware Best Practices
```
CPU:
  - vCPU:pCPU ratio should not exceed 4:1 for most workloads
  - Avoid oversizing VMs (unused vCPUs waste scheduler resources)
  - Use CPU reservations only for VMs with strict performance needs

Memory:
  - Memory overcommit (balloon/swap) impacts performance — monitor closely
  - Never over-provision memory beyond physical host capacity for Tier 0/1 VMs
  - Use memory reservations for databases and critical systems

Networking:
  - Separate vSwitches for VM traffic, management, vMotion, storage
  - Use LACP/port channels for uplinks
  - vMotion on dedicated VLAN

Storage:
  - Use VMware datastores (VMFS or vSAN) — avoid RDM unless required
  - Storage vMotion to balance I/O
  - Snapshot size monitoring — remove old snapshots
  - Use thin provisioning with monitoring for overcommit
```

### Hyper-V Management

```powershell
# List all VMs
Get-VM | Select-Object Name, State, CPUUsage, MemoryMB, VMHost | Format-Table

# Get VM configuration
Get-VM "ServerName" | Get-VMProcessor
Get-VM "ServerName" | Get-VMMemory
Get-VM "ServerName" | Get-VMHardDiskDrive

# Check replication status (Hyper-V Replica)
Get-VMReplication | Select-Object VMName, State, Health, LastReplicationTime | Format-Table

# Live migration
Move-VM -Name "ServerName" -DestinationHost "HV02" -IncludeStorage -DestinationStoragePath "C:\VMs"

# Checkpoint (snapshot) management
Get-VMCheckpoint -VMName "ServerName"
Remove-VMCheckpoint -VMName "ServerName" -Name "Pre-Patch"

# Export VM
Export-VM -Name "ServerName" -Path "\\BackupShare\VMExports\"

# Enable dynamic memory
Set-VMMemory "ServerName" -DynamicMemoryEnabled $true -MinimumBytes 1GB -MaximumBytes 8GB -StartupBytes 4GB
```

---

## 3. Storage Management <a name="storage"></a>

### Storage Architecture Types

```
DAS (Direct Attached Storage):
  Use case: Single server, low cost, no sharing
  
NAS (Network Attached Storage):
  Protocol: SMB (Windows), NFS (Linux)
  Use case: File sharing, home directories, backups
  
SAN (Storage Area Network):
  Protocol: Fibre Channel, iSCSI
  Use case: High-performance block storage, databases, VMs

Object Storage:
  Protocol: S3-compatible API, Azure Blob
  Use case: Unstructured data, backups, archives, cloud-native
```

### Windows Storage Management

```powershell
# View physical disks
Get-PhysicalDisk | Select-Object FriendlyName, Size, MediaType, OperationalStatus, HealthStatus

# View storage pools
Get-StoragePool | Select-Object FriendlyName, AllocatedSize, Size, OperationalStatus, HealthStatus

# Create storage pool and virtual disk
$disks = Get-PhysicalDisk -CanPool $true
New-StoragePool -FriendlyName "DataPool" -StorageSubsystemFriendlyName "Windows Storage*" -PhysicalDisks $disks

New-VirtualDisk -StoragePoolFriendlyName "DataPool" -FriendlyName "DataVDisk" `
  -Size 2TB -ResiliencySettingName Mirror -ProvisioningType Fixed

# Initialize and format
Initialize-Disk -FriendlyName "DataVDisk"
New-Partition -DiskFriendlyName "DataVDisk" -UseMaximumSize -AssignDriveLetter
Format-Volume -DriveLetter D -FileSystem NTFS -NewFileSystemLabel "Data" -AllocationUnitSize 65536  # 64K for SQL/VMs

# Check disk health (SMART)
Get-PhysicalDisk | Where-Object {$_.HealthStatus -ne "Healthy"} | Format-List *

# Find large files eating disk space
Get-ChildItem D:\ -Recurse -ErrorAction SilentlyContinue | 
  Sort-Object Length -Descending | 
  Select-Object -First 20 FullName, @{N='SizeMB';E={[Math]::Round($_.Length/1MB,1)}}
```

### DFS (Distributed File System)

```powershell
# DFS Namespaces
New-DfsnRoot -Path "\\domain.com\Files" -TargetPath "\\FileServer01\Files" -Type DomainV2

# Add DFS folder target
New-DfsnFolderTarget -Path "\\domain.com\Files\HR" -TargetPath "\\FileServer01\HR"

# Add second target for redundancy
Add-DfsnFolderTarget -Path "\\domain.com\Files\HR" -TargetPath "\\FileServer02\HR"

# DFS Replication
New-DfsReplicationGroup -GroupName "HR-Files-Replication"
Add-DfsrMember -GroupName "HR-Files-Replication" -ComputerName "FileServer01"
Add-DfsrMember -GroupName "HR-Files-Replication" -ComputerName "FileServer02"
New-DfsReplicatedFolder -GroupName "HR-Files-Replication" -FolderName "HR" -DfsnPath "\\domain.com\Files\HR"
Add-DfsrConnection -GroupName "HR-Files-Replication" -SourceComputerName "FileServer01" -DestinationComputerName "FileServer02"

# Check DFS replication health
Get-DfsrState -ComputerName FileServer01 | Select-Object ComputerName, ReplicationGroupName, State

# Check backlog
Get-DfsrBacklog -SourceComputerName FileServer01 -DestinationComputerName FileServer02 -GroupName "HR-Files-Replication" -FolderName "HR" | Measure-Object
```

---

## 4. Backup & Recovery Operations <a name="backup"></a>

### Backup Strategy — 3-2-1-1-0 Rule
```
3 copies of data
2 different storage media types
1 copy offsite (or cloud)
1 copy offline / air-gapped (immutable)
0 unverified backups (test restores regularly)
```

### Veeam Backup Operations

```powershell
# Connect to Veeam server
Add-PSSnapin -Name VeeamPSSnapIn

# List backup jobs
Get-VBRJob | Select-Object Name, JobType, LastRun, LastResult, IsScheduleEnabled

# List restore points
Get-VBRRestorePoint | Sort-Object CreationTime -Descending | 
  Select-Object Name, CreationTime, @{N='SizeGB';E={[Math]::Round($_.ApproxSize/1GB,1)}} | 
  Select-Object -First 20

# Start a backup job
Start-VBRJob -Job (Get-VBRJob -Name "Daily Backup")

# Check backup repository space
Get-VBRBackupRepository | Select-Object Name, Path, @{N='FreeGB';E={[Math]::Round($_.FreeSpace/1GB,1)}}, @{N='TotalGB';E={[Math]::Round($_.Capacity/1GB,1)}}

# Verify last backup sessions
Get-VBRSession | Sort-Object CreationTime -Descending | 
  Select-Object -First 20 |
  Select-Object JobName, CreationTime, Result, @{N='DurationMin';E={($_.EndTime-$_.CreationTime).TotalMinutes}} |
  Format-Table
```

### Windows Server Backup

```powershell
# Check backup history
Get-WBJob -Previous 10 | Select-Object StartTime, JobType, JobState, HResult

# Check last backup time
Get-WBSummary | Select-Object LastSuccessfulBackupTime, LastBackupTarget

# Start manual backup
Start-WBBackup -Policy (Get-WBPolicy)

# List available restore points
Get-WBBackupTarget | Get-WBBackupSet | Sort-Object BackupTime -Descending | Select-Object -First 5
```

### Test Restore Procedures
```
Monthly restore tests (minimum):
  [ ] Restore a random file from file server backup
  [ ] Restore a VM to isolated test environment
  [ ] Verify all services start correctly
  [ ] Confirm data integrity

Quarterly full DR test:
  [ ] Restore primary server to secondary site or cloud
  [ ] Confirm services accessible from test network
  [ ] Measure actual RTO vs. target RTO
  [ ] Document results and gaps

Document every restore test:
  Date | System | Backup Age Used | Result | RTO Achieved | Issues Found
```

---

## 5. Monitoring & Alerting <a name="monitoring"></a>

### What to Monitor

**Infrastructure:**
```
Servers:
  - CPU utilization (alert >85% sustained 5 min)
  - Memory utilization (alert >90%)
  - Disk utilization (alert >85%, critical >95%)
  - Disk I/O queue length (alert >2 sustained)
  - Network interface errors/discards
  - Service availability (check every 60 seconds)

Storage:
  - RAID/storage pool health
  - Disk SMART status
  - NAS/SAN free space

Networking:
  - Interface utilization (alert >80%)
  - Packet loss (alert >1%)
  - Latency to key resources
  - BGP/routing table changes

Security:
  - Failed authentication spikes
  - AV/EDR alerts
  - Firewall denied traffic spikes
  - Certificate expiration (alert 60 days, critical 14 days)
```

### PowerShell Monitoring Scripts

```powershell
# Check all servers for disk space below threshold
$servers = Get-ADComputer -Filter {OperatingSystem -like "*Server*"} | Select-Object -ExpandProperty Name
$threshold = 85  # percent

foreach ($server in $servers) {
  if (Test-Connection $server -Count 1 -Quiet) {
    try {
      $disks = Get-CimInstance -ComputerName $server -ClassName Win32_LogicalDisk -Filter "DriveType=3" |
        Select-Object DeviceID, 
          @{N='TotalGB';E={[Math]::Round($_.Size/1GB,1)}},
          @{N='FreeGB';E={[Math]::Round($_.FreeSpace/1GB,1)}},
          @{N='UsedPct';E={[Math]::Round(100-($_.FreeSpace/$_.Size*100),1)}}
      
      $disks | Where-Object {$_.UsedPct -gt $threshold} | ForEach-Object {
        Write-Warning "DISK ALERT: $server $($_.DeviceID) — $($_.UsedPct)% used ($($_.FreeGB) GB free)"
      }
    }
    catch { Write-Warning "Cannot query $server : $($_.Exception.Message)" }
  }
}

# Check certificate expiration
$servers = @("web01.domain.com", "portal.domain.com", "vpn.domain.com")
$daysWarning = 60

foreach ($server in $servers) {
  try {
    $cert = [System.Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}
    $socket = New-Object System.Net.Sockets.TcpClient($server, 443)
    $stream = New-Object System.Net.Security.SslStream($socket.GetStream())
    $stream.AuthenticateAsClient($server)
    $cert = $stream.RemoteCertificate
    $expiry = [DateTime]::Parse($cert.GetExpirationDateString())
    $daysLeft = ($expiry - (Get-Date)).Days
    
    if ($daysLeft -lt $daysWarning) {
      Write-Warning "CERT EXPIRY: $server — expires $expiry ($daysLeft days remaining)"
    }
    $stream.Close(); $socket.Close()
  }
  catch { Write-Warning "Cannot check cert for $server" }
}
```

---

## 6. Capacity Planning <a name="capacity"></a>

### Capacity Planning Process

```
Monthly:
  1. Collect: CPU/RAM/disk trends for previous month (from monitoring)
  2. Project: Extrapolate 6-month trend
  3. Compare: Against available capacity and warning thresholds
  4. Plan: If projected to breach threshold within 6 months, plan expansion

Key thresholds for action:
  CPU: Average >70% → investigate; Average >80% sustained → add capacity
  RAM: >85% → investigate; >90% → urgent action
  Disk: >80% → plan expansion; >90% → urgent
  
Annual:
  - Full infrastructure review for next year budget planning
  - Business growth projections from sales/ops team
  - Hardware refresh planning (end-of-support dates)
```

---

## 7. Infrastructure as Code (IaC) <a name="iac"></a>

### Terraform Basics

```hcl
# Azure VM deployment example
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

provider "azurerm" {
  features {}
}

resource "azurerm_resource_group" "main" {
  name     = "rg-servers-prod"
  location = "East US"
}

resource "azurerm_virtual_machine" "appserver" {
  name                = "vm-appserver-01"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  vm_size             = "Standard_D2s_v3"
  
  # ... (NIC, OS disk, image reference, etc.)
}
```

### Ansible Basics (Configuration Management)

```yaml
# playbook: configure-windows-baseline.yml
---
- name: Configure Windows Server Baseline
  hosts: windows_servers
  tasks:
    - name: Ensure Windows Firewall is enabled
      win_firewall:
        state: enabled
        profiles:
          - Domain
          - Private
          - Public

    - name: Disable SMBv1
      win_powershell:
        script: |
          Set-SmbServerConfiguration -EnableSMB1Protocol $false -Force

    - name: Enable audit logging
      win_audit_policy_system:
        subcategory: Logon
        audit_type: success,failure

    - name: Ensure Windows Update service is running
      win_service:
        name: wuauserv
        state: started
        start_mode: auto

    - name: Set NTP server
      win_command: "w32tm /config /manualpeerlist:time.windows.com /syncfromflags:MANUAL /reliable:YES /update"
```

---

## 8. Cloud Infrastructure (Azure & AWS) <a name="cloud"></a>

### Azure CLI Common Commands

```bash
# Login
az login

# List subscriptions
az account list --output table

# List VMs
az vm list --output table
az vm list --query "[?powerState=='VM running']" --show-details

# Start/stop VM
az vm start --resource-group rg-prod --name vm-appserver-01
az vm deallocate --resource-group rg-prod --name vm-appserver-01  # Stopped + deallocated = no compute charges

# VM sizes/pricing
az vm list-sizes --location eastus --output table

# Create VM from CLI
az vm create \
  --resource-group rg-prod \
  --name vm-webserver-01 \
  --image Win2022Datacenter \
  --size Standard_D2s_v3 \
  --admin-username azureadmin \
  --vnet-name vnet-prod \
  --subnet subnet-servers \
  --public-ip-address ""   # No public IP (best practice)

# View resource costs (requires billing reader role)
az consumption usage list --start-date 2024-01-01 --end-date 2024-01-31 --output table
```

### AWS CLI Common Commands

```bash
# Configure
aws configure

# EC2 instances
aws ec2 describe-instances --query "Reservations[].Instances[].[InstanceId,State.Name,InstanceType,Tags[?Key=='Name'].Value|[0]]" --output table

# Start/stop EC2
aws ec2 start-instances --instance-ids i-1234567890abcdef0
aws ec2 stop-instances --instance-ids i-1234567890abcdef0

# S3 operations
aws s3 ls s3://my-bucket/
aws s3 cp local-file.txt s3://my-bucket/remote-file.txt
aws s3 sync /local/folder s3://my-bucket/folder --delete

# CloudWatch logs
aws logs describe-log-groups
aws logs get-log-events --log-group-name /aws/lambda/my-function --log-stream-name latest
```

---

## 9. Patch & Configuration Management <a name="patching"></a>

### WSUS Management

```powershell
# Check WSUS server sync status
(Get-WsusServer).GetSubscription().GetSynchronizationHistory() | 
  Sort-Object StartTime -Descending | Select-Object -First 5 | 
  Select-Object StartTime, Result, SyncType

# Find unapproved updates
Get-WsusUpdate -Approval Unapproved -Status Needed | 
  Where-Object {$_.Update.MsrcSeverity -in "Critical","Important"} |
  Select-Object @{N='Title';E={$_.Update.Title}}, @{N='Severity';E={$_.Update.MsrcSeverity}}, 
    @{N='Released';E={$_.Update.CreationDate}} | Sort-Object Released -Descending

# Get client compliance
(Get-WsusServer).GetComputerTargetGroups() | ForEach-Object {
  $group = $_
  $computers = $group.GetComputerTargets()
  [PSCustomObject]@{
    Group = $group.Name
    TotalComputers = $computers.Count
    UpToDate = ($computers | Where-Object {$_.GetUpdateInstallationSummary().UpToDateCount -eq $computers.Count}).Count
  }
}
```

---

## 10. Physical Infrastructure <a name="physical"></a>

### Data Center / Server Room Checklist

**Environmental Monitoring:**
```
□ Temperature: 18–27°C (64–80°F) — alert at 28°C, critical at 32°C
□ Humidity: 40–60% relative humidity
□ UPS runtime: Sufficient for graceful shutdown + diesel generator start (typically 10–20 min)
□ Generator test: Monthly under load
□ PDU monitoring: Per-outlet power monitoring on high-density racks
□ Hot/cold aisle containment: Cold air in front, hot exhaust in back/top
□ CRAC/CRAH units: Redundant cooling
□ Water leak detection: Under raised floor and near CRAC units
□ Smoke/fire detection: Early warning smoke detection (VESDA)
□ Access control: Badge-only access, camera coverage, visitor log
```

**Rack Management:**
```
□ Cable management: Front and rear cable managers
□ Labeling: Every cable labeled at both ends
□ Power redundancy: Dual PSU servers on separate PDUs (PDU-A/PDU-B)
□ Rack capacity: Max 80% of rated load per rack
□ Weight limits: Adhere to floor load ratings (typically 1000–1500 kg/m²)
□ Inventory: Rack diagram current and accurate
□ Spare parts: Critical spare drives, PSUs, NIC cards on-hand
```

### Server Hardware Maintenance

```powershell
# Check server hardware health (iDRAC/iLO via IPMI)
# For Dell iDRAC:
racadm -r idrac-address -u root -p password getsysinfo

# Windows hardware health check
Get-WmiObject -Class Win32_PNPEntity | Where-Object {$_.ConfigManagerErrorCode -ne 0} | 
  Select-Object Name, ConfigManagerErrorCode, Description

# Check system event log for hardware errors
Get-WinEvent -LogName System | Where-Object {$_.ProviderName -like "*disk*" -or $_.ProviderName -like "*hardware*"} | 
  Where-Object {$_.Level -le 2} | Select-Object -First 20 | Format-List TimeCreated, ProviderName, Message

# Memory test schedule
# Schedule at next restart
mdsched.exe  # Windows Memory Diagnostic
```
