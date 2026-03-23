# Storage Administration & Backup Deep Dive

## Table of Contents
1. [Storage Technologies](#storage-technologies)
2. [SAN Administration](#san-administration)
3. [NAS Administration](#nas-administration)
4. [Windows Storage Spaces & ReFS](#windows-storage)
5. [Linux Storage Management](#linux-storage)
6. [Enterprise Backup Architecture](#backup-architecture)
7. [Veeam Backup & Replication](#veeam)
8. [Backup Testing & Validation](#backup-testing)
9. [Storage Performance Tuning](#performance)
10. [Capacity Planning](#capacity-planning)

---

## 1. Storage Technologies

### Storage Type Comparison

| Type | Protocol | Latency | Throughput | Best For |
|------|----------|---------|-----------|---------|
| Local NVMe | PCIe | < 100 μs | 7+ GB/s | DB, VMs, latency-critical |
| Local SAS SSD | SAS | 200–500 μs | 2–4 GB/s | General servers |
| Local SATA SSD | SATA | 500 μs–1 ms | 500 MB/s | General purpose |
| SAN (FC) | Fibre Channel | 200–500 μs | 16–32 Gbps | Shared block storage |
| SAN (iSCSI) | TCP/IP | 500 μs–2 ms | 10–25 Gbps | Cost-effective SAN |
| NAS (NFS/SMB) | Ethernet | 1–5 ms | 10–40 Gbps | File sharing |
| Object (S3) | HTTP | 50–100 ms | Unlimited scale | Backup, archival, cloud |
| HDD | Various | 5–20 ms | 200 MB/s | Backup, archival |

### RAID Levels

| RAID | Min Disks | Fault Tolerance | Space Efficiency | Notes |
|------|-----------|----------------|-----------------|-------|
| 0 | 2 | None | 100% | Stripe only, no tolerance |
| 1 | 2 | 1 disk | 50% | Mirror, simple |
| 5 | 3 | 1 disk | (n-1)/n | Parity, common |
| 6 | 4 | 2 disks | (n-2)/n | Dual parity, safe |
| 10 | 4 | 1 per mirror | 50% | Stripe of mirrors, fast |
| 50 | 6 | 1 per group | Varies | RAID 5 stripes |
| 60 | 8 | 2 per group | Varies | RAID 6 stripes |

**Rule of thumb:**
- RAID 10: Best performance + protection (VMs, databases)
- RAID 6: Best capacity efficiency (NAS, file storage)
- RAID 5: Adequate for read-heavy, small arrays
- Never run critical data without redundancy (no RAID 0 for production)

### Storage Protocols

**Block Storage Protocols:**
```
Fibre Channel (FC):
  - Dedicated storage network (SAN fabric)
  - Uses HBAs (Host Bus Adapters), FC switches
  - Zoning: Single-initiator / Single-target (best practice)
  - Lower latency, higher reliability than iSCSI
  - Cost: High (specialized hardware)

iSCSI:
  - Block storage over standard TCP/IP Ethernet
  - Lower cost than FC
  - Use dedicated VLAN for iSCSI traffic
  - Jumbo frames (MTU 9000) for performance
  - CHAP authentication for security

NVMe over Fabrics (NVMe-oF):
  - Ultra-low latency block storage
  - Runs over FC, Ethernet (RoCE), or InfiniBand
  - Emerging in high-performance environments
```

**File Storage Protocols:**
```
SMB/CIFS:  Windows file sharing (SMB 1.0 disabled, use SMB 3.x)
NFS:       Unix/Linux file sharing (NFSv4 preferred)
AFP:       Legacy Apple file sharing (deprecated)

SMB3 features:
  - SMB Signing and Encryption
  - SMB Multichannel (multiple NICs = more throughput)
  - SMB Direct (RDMA over InfiniBand/RoCE)
  - Transparent failover (with Windows File Server cluster)
```

---

## 2. SAN Administration

### Fibre Channel Zoning

```
Zoning types:
- Hard zoning (WWN-based): Enforced by FC switch; most secure
- Soft zoning (port-based): Legacy, less secure

Best practices:
- Single-initiator / single-target zoning (most restrictive)
- Never put competing initiators in same zone
- Name zones descriptively: Host01_HBA1_ARRAY_PORT1

Cisco MDS (Brocade similar):
! Show all zones
show zone

! Create a zone
zone name HOST01-ARRAY-LUN1 vsan 100
  member pwwn 21:00:00:24:ff:ab:cd:ef  ! Host HBA
  member pwwn 50:06:01:60:88:03:xx:xx  ! Array port

! Add zone to zoneset and activate
zoneset name PRODUCTION vsan 100
  member HOST01-ARRAY-LUN1

zoneset activate name PRODUCTION vsan 100
```

### LUN Provisioning

```
Key sizing considerations:
- Provision what's needed now + 20-30% growth headroom
- Align LUN size to OS partition alignment (512-byte or 4K sectors)
- For VMware: LUN per datastore; size per expected VM count
  Rule: 2–5 VMs per LUN (balance performance and management)

VMware datastore LUN sizing:
  VMs per datastore: 4–8 recommended
  LUN size: VM average size × count + 20% headroom
  Example: 4 VMs × 100 GB average + 20% = 480 GB → 500 GB LUN
```

### iSCSI Configuration (Windows Initiator)

```powershell
# Enable iSCSI service
Set-Service -Name MSiSCSI -StartupType Automatic
Start-Service MSiSCSI

# Discover iSCSI target
iscsicli AddTargetPortal 192.168.100.10 3260

# Login to target
iscsicli LoginTarget iqn.2024-01.com.vendor:storage1 T * * * * * * * * * * * 0 0 0 0 0

# List connected sessions
iscsicli SessionList

# PowerShell method
$Portal = New-IscsiTargetPortal -TargetPortalAddress "192.168.100.10"
Get-IscsiTarget | Connect-IscsiTarget -IsPersistent $true

# After connecting, initialize disk in Disk Management or:
Get-Disk | Where-Object PartitionStyle -eq RAW | Initialize-Disk -PartitionStyle GPT
```

### Storage Array Performance Metrics

| Metric | Good | Warning | Critical |
|--------|------|---------|---------|
| Read Latency | < 1 ms | 1–5 ms | > 5 ms |
| Write Latency | < 2 ms | 2–10 ms | > 10 ms |
| Cache Hit Rate | > 90% | 70–90% | < 70% |
| Disk Utilization | < 60% | 60–80% | > 80% |
| Queue Depth | < 4 | 4–16 | > 16 |

---

## 3. NAS Administration

### Windows File Server

```powershell
# Install File Server role
Install-WindowsFeature -Name FS-FileServer -IncludeManagementTools

# Create share with specific permissions
$SharePath = "D:\Shares\Marketing"
New-Item -ItemType Directory -Path $SharePath
New-SmbShare -Name "Marketing" -Path $SharePath `
    -FullAccess "CONTOSO\Marketing_Admins" `
    -ChangeAccess "CONTOSO\Marketing_Users" `
    -ReadAccess "CONTOSO\Marketing_ReadOnly" `
    -FolderEnumerationMode AccessBased  # Hide folders user can't access

# Set NTFS permissions (separate from share permissions)
$Acl = Get-Acl $SharePath
$Rule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "CONTOSO\Marketing_Users", "Modify", "ContainerInherit,ObjectInherit", 
    "None", "Allow"
)
$Acl.AddAccessRule($Rule)
Set-Acl $SharePath $Acl

# Enable access-based enumeration (hides folders users can't access)
Set-SmbShare -Name "Marketing" -FolderEnumerationMode AccessBased

# Enable SMB encryption for sensitive shares
Set-SmbShare -Name "Finance" -EncryptData $true
```

### DFS (Distributed File System)

```powershell
# Install DFS features
Install-WindowsFeature -Name FS-DFS-Namespace, FS-DFS-Replication -IncludeManagementTools

# Create DFS namespace (domain-based)
New-DfsnRoot -Path "\\contoso.com\Files" -Type DomainV2 -TargetPath "\\FS01\Files"

# Add DFS folder
New-DfsnFolder -Path "\\contoso.com\Files\Marketing" `
    -TargetPath "\\FS01\Marketing" -EnableTargetFailback $true

# Add second target (failover/load balance)
New-DfsnFolderTarget -Path "\\contoso.com\Files\Marketing" `
    -TargetPath "\\FS02\Marketing"

# DFS Replication group (keeps targets in sync)
New-DfsReplicationGroup -GroupName "Marketing-Replication"
Add-DfsrMember -GroupName "Marketing-Replication" -ComputerName FS01, FS02
New-DfsReplicatedFolder -GroupName "Marketing-Replication" -FolderName "Marketing" `
    -DfsnPath "\\contoso.com\Files\Marketing"
```

### File Server Resource Manager (FSRM)

```powershell
# Install FSRM
Install-WindowsFeature -Name FS-Resource-Manager -IncludeManagementTools

# Create storage quota (5 GB per user folder)
$QuotaTemplate = New-FsrmQuotaTemplate -Name "5GB User Quota" `
    -Size 5GB -SoftLimit $false

# Apply quota to user home folders
New-FsrmQuota -Path "D:\HomeDirectories" -Template "5GB User Quota" -AutoQuota

# Create file screen (block executables from file shares)
$BlockedTypes = @("*.exe","*.com","*.bat","*.vbs","*.ps1","*.msi")
New-FsrmFileGroup -Name "Executable Files" -IncludePattern $BlockedTypes
New-FsrmFileScreen -Path "D:\Shares" -Template "Block Executable Files"

# Email notification when quota reaches 85%
New-FsrmAction Email -MailTo "[Admin Email]" `
    -Subject "Quota Alert: [Quota Path]" `
    -Body "User [Source Io Owner] has used [Quota Used Percent]% of their quota"
```

---

## 4. Windows Storage Spaces & ReFS

### Storage Spaces Direct (S2D)

```powershell
# Validate cluster readiness
Test-Cluster -Node S2D-Node1, S2D-Node2, S2D-Node3, S2D-Node4 `
    -Include "Storage Spaces Direct","Inventory","Network","System Configuration"

# Enable S2D after cluster creation
Enable-ClusterStorageSpacesDirect `
    -PoolFriendlyName "S2D Pool" `
    -CacheState Enabled `
    -CacheMode ReadWrite

# Create tiered virtual disk
New-Volume -FriendlyName "VMs" `
    -StorageTierFriendlyNames "Performance","Capacity" `
    -StorageTierSizes 500GB, 2TB `
    -FileSystem CSVFS_ReFS `
    -ResiliencySettingName Mirror

# Monitor S2D health
Get-StoragePool -FriendlyName "S2D Pool" | Get-PhysicalDisk | 
    Select-Object FriendlyName, HealthStatus, OperationalStatus, Size, MediaType

Get-VirtualDisk | Select-Object FriendlyName, HealthStatus, OperationalStatus, Size
```

### ReFS vs NTFS

| Feature | NTFS | ReFS |
|---------|------|------|
| Max volume size | 256 TB | 35 PB |
| Integrity streams | No | Yes |
| Corruption auto-repair | No | Yes (with S2D) |
| Block cloning | No | Yes |
| Sparse VDL | No | Yes (fast VM provisioning) |
| Deduplication | Yes | Limited |
| Compression | Yes | No |
| Quotas | Yes | No |
| Encryption (BitLocker) | Yes | Yes |

**Use ReFS for:** Hyper-V VM storage, backup targets, large file workloads  
**Use NTFS for:** Boot volumes, user data shares, anything needing quotas/compression

---

## 5. Linux Storage Management

### Disk and Partition Management

```bash
# List all disks and partitions
lsblk -f
fdisk -l

# Partition a new disk
parted /dev/sdb
(parted) mklabel gpt
(parted) mkpart primary 0% 100%
(parted) quit

# Format partition
mkfs.xfs /dev/sdb1        # XFS: recommended for large files
mkfs.ext4 /dev/sdb1       # ext4: traditional Linux filesystem

# Mount and add to fstab
mkdir /mnt/data
mount /dev/sdb1 /mnt/data

# Get UUID for fstab (use UUID, not /dev/sdX — device names can change)
blkid /dev/sdb1
# Add to /etc/fstab:
# UUID=xxxx-xxxx /mnt/data xfs defaults 0 2
```

### LVM (Logical Volume Manager)

```bash
# Initialize physical volume
pvcreate /dev/sdb /dev/sdc

# Create volume group
vgcreate vg_data /dev/sdb /dev/sdc

# Create logical volume
lvcreate -L 500G -n lv_database vg_data

# Format and mount
mkfs.xfs /dev/vg_data/lv_database
mount /dev/vg_data/lv_database /data

# Extend logical volume online (XFS supports online grow)
# First add disk if needed
pvcreate /dev/sdd
vgextend vg_data /dev/sdd

# Extend LV and filesystem
lvextend -L +200G /dev/vg_data/lv_database
xfs_growfs /data   # XFS: online grow
# resize2fs /data  # ext4: online grow

# Show LVM status
pvs   # Physical volumes
vgs   # Volume groups
lvs   # Logical volumes
```

### Linux NFS Server

```bash
# Install NFS server
apt install nfs-kernel-server   # Debian/Ubuntu
yum install nfs-utils           # RHEL/CentOS

# Configure exports
cat >> /etc/exports << 'EOF'
/data/shared    192.168.1.0/24(rw,sync,no_subtree_check,no_root_squash)
/data/readonly  *(ro,sync,no_subtree_check)
EOF

# Apply exports
exportfs -ra
systemctl enable --now nfs-server

# Client mount
mount -t nfs -o nfsvers=4,rsize=1048576,wsize=1048576 \
    server1:/data/shared /mnt/nfs

# fstab entry
echo "server1:/data/shared /mnt/nfs nfs nfsvers=4,rsize=1048576,wsize=1048576,_netdev 0 0" >> /etc/fstab
```

### Disk Health Monitoring

```bash
# SMART status check
smartctl -a /dev/sda
smartctl -H /dev/sda   # Quick health check

# Key SMART attributes to watch:
# ID 5   Reallocated Sectors Count  (should be 0)
# ID 10  Spin Retry Count           (should be 0)
# ID 187 Reported Uncorrectable Err (should be 0)
# ID 188 Command Timeout            (any increase is bad)
# ID 197 Current Pending Sector Ct  (should be 0)
# ID 198 Offline Uncorrectable Sect (should be 0)

# Disk I/O statistics
iostat -x 1 10     # Extended stats, 10 samples, 1 second interval
iotop -o           # Live I/O by process

# Check filesystem usage and inode usage
df -h              # Disk space
df -i              # Inode usage (can run out before disk space)

# Find large files
find /data -type f -size +1G -exec ls -lh {} \; 2>/dev/null | sort -k5 -hr | head -20

# Find directories consuming most space
du -h --max-depth=2 /data | sort -hr | head -20
```

---

## 6. Enterprise Backup Architecture

### Backup Architecture Components

```
Source Data (Production)
        ↓
Backup Agent / API (installed on servers or via VSS/VMware snapshot)
        ↓
Backup Server (Veeam, Commvault, Rubrik, etc.)
        ↓
Primary Backup Repository (fast disk, 14–30 day retention)
        ↓ (Copy job)
Secondary Repository (different location, 30–90 days)
        ↓ (Tape/Archive job)
Offsite / Archive (tape, cloud, > 90 days)
```

### Retention Policy Design

```
Production databases:
  Full:           Weekly (Sunday night)
  Differential:   Daily (Mon–Sat)
  Transaction log: Every 15 minutes
  Retention:      30 days locally, 1 year archive

File servers:
  Full:           Monthly
  Incremental:    Daily
  Retention:      30 days locally, 7 years archive (legal hold)

VMs (non-critical):
  Full:           Weekly
  Incremental:    Daily
  Retention:      14 days

VMs (critical):
  Full:           Weekly
  Incremental:    Every 4 hours
  Retention:      30 days, 1 year archive

Compliance-driven retention:
  HIPAA:          6 years
  SOX:            7 years
  PCI DSS:        1 year (logs), varies for data
  GDPR:           Minimum necessary (not a backup mandate, a deletion requirement)
```

### Backup Repository Sizing

```
Formula for full + incremental:
  Repo size = Full size × (1 + daily_change% × days_in_cycle)
  
Example (VMs, 20% daily change, 14-day retention):
  Total VM size: 10 TB
  Daily change: 20% = 2 TB/day
  Incremental size: 2 TB × 14 days = 28 TB
  Full: 10 TB
  With compression (2:1): (10 + 28) / 2 = 19 TB
  With dedup (3:1): 19 / 3 ≈ 6.3 TB
  Safety margin (20%): ~8 TB

Note: Real dedup/compression ratios vary widely by data type
  - VMs with OS: 3:1 to 5:1 common
  - Already-compressed data (video, zip): near 1:1
  Always measure actual ratio in your environment
```

---

## 7. Veeam Backup & Replication

### Veeam Architecture

```
Components:
Veeam Backup Server    — Management server, job scheduler, catalog
Veeam Proxy           — Data mover; reads VM data from hypervisor
Veeam Repository      — Storage target for backups
Veeam WAN Accelerator — Dedup cache for WAN transfers
Veeam Mount Server    — Mounts backup for file-level recovery
```

### Veeam PowerShell Essentials

```powershell
# Connect to Veeam server
Add-PSSnapin VeeamPSSnapIn -ErrorAction SilentlyContinue
Connect-VBRServer -Server "veeam01" -Credential (Get-Credential)

# Get all backup jobs and last result
Get-VBRJob | Select-Object Name, JobType, IsScheduleEnabled,
    @{N="LastResult";E={$_.GetLastResult()}},
    @{N="LastRun";E={$_.FindLastSession().EndTime}} |
    Format-Table

# Get failed jobs in last 24 hours
$Since = (Get-Date).AddHours(-24)
Get-VBRBackupSession | 
    Where-Object { $_.EndTime -gt $Since -and $_.Result -eq "Failed" } |
    Select-Object JobName, CreationTime, EndTime, Result, JobResult |
    Format-Table

# Start a backup job
$Job = Get-VBRJob -Name "Daily-Prod-VMs"
Start-VBRJob -Job $Job

# File-level restore
$RestorePoint = Get-VBRRestorePoint -Name "Server01" | 
    Sort-Object CreationTime -Descending | Select-Object -First 1
Start-VBRWindowsFileRestore -RestorePoint $RestorePoint

# Instant VM recovery
$RestorePoint = Get-VBRRestorePoint -Name "WebServer01" |
    Sort-Object CreationTime -Descending | Select-Object -First 1
Start-VBRInstantRecovery -RestorePoint $RestorePoint `
    -Server (Get-VBRServer -Name "ESXi-Host-01") `
    -Datastore (Find-VBRViDatastore -Server (Get-VBRServer -Name "ESXi-Host-01") -Name "Datastore01")

# Check repository capacity
Get-VBRBackupRepository | 
    Select-Object Name, 
        @{N="CapacityGB";E={[math]::Round($_.GetContainer().CachedTotalSpace.InGigabytes,1)}},
        @{N="FreeGB";E={[math]::Round($_.GetContainer().CachedFreeSpace.InGigabytes,1)}},
        @{N="UsedPct";E={[math]::Round((1-($_.GetContainer().CachedFreeSpace.InGigabytes/$_.GetContainer().CachedTotalSpace.InGigabytes))*100,1)}} |
    Format-Table
```

### Veeam Scale-Out Backup Repository

```
SOBR extends repository capacity across multiple extents:
- Performance tier: Fast disk (NVMe/SSD) for recent backups
- Capacity tier: Object storage (S3/Azure Blob) for older backups
- Archive tier: Cold storage (S3 Glacier) for long-term retention

Data movement policy:
  Operational restore window: 14 days (stay in performance tier)
  Move to capacity tier after: 14 days
  Move to archive tier after: 180 days
```

---

## 8. Backup Testing & Validation

### Recovery Testing Plan

```
Test Type 1: Backup Job Verification
  Frequency: Every backup job
  Method: Veeam SureBackup / vendor equivalent (automated VM boot test)
  Pass criteria: VM boots successfully, heartbeat detected

Test Type 2: File Recovery Test
  Frequency: Monthly
  Method: Restore random file from random server, verify integrity
  Pass criteria: File restores, hash matches original

Test Type 3: Application Recovery Test
  Frequency: Quarterly
  Method: Restore application VM to isolated network, verify function
  Application examples: SQL Server (run queries), Active Directory (login test), Exchange (send/receive test)
  Pass criteria: Application functional within RTO

Test Type 4: Full DR Test
  Frequency: Annual minimum
  Method: Fail over entire environment to DR site
  Pass criteria: All critical applications functional within RTO target
  Duration: Plan 1–2 full business days
```

### Backup Reporting

```powershell
# Generate daily backup health report
$Yesterday = (Get-Date).AddDays(-1).Date
$Sessions = Get-VBRBackupSession | Where-Object { $_.CreationTime -gt $Yesterday }

$Report = $Sessions | Group-Object Result | ForEach-Object {
    [PSCustomObject]@{
        Status = $_.Name
        Count = $_.Count
        Jobs = ($_.Group.JobName -join ", ")
    }
}

# Send email report
$Body = $Report | ConvertTo-Html -Property Status, Count, Jobs `
    -Head "<style>table{border-collapse:collapse}td,th{border:1px solid black;padding:5px}</style>"

Send-MailMessage -From "backups@contoso.com" -To "it-team@contoso.com" `
    -Subject "Daily Backup Report - $(Get-Date -Format 'yyyy-MM-dd')" `
    -BodyAsHtml -Body ($Body | Out-String) `
    -SmtpServer "smtp.contoso.com"
```

---

## 9. Storage Performance Tuning

### Windows Storage Optimization

```powershell
# Check disk performance
Get-PhysicalDisk | Get-StorageReliabilityCounter |
    Select-Object PSComputerName, ReadErrorsTotal, WriteErrorsTotal, Temperature

# Enable write-back cache (for non-battery-backed arrays — use with caution)
Set-PhysicalDisk -UniqueId "xxxx" -MediaType HDD

# Optimize drives (defrag HDD, TRIM SSD)
Optimize-Volume -DriveLetter D -Defrag -Verbose    # HDD
Optimize-Volume -DriveLetter D -ReTrim -Verbose    # SSD

# Check and set storage QoS
New-StorageQosPolicy -Name "Gold" -MaximumIops 10000 -MinimumIops 2000
New-StorageQosPolicy -Name "Silver" -MaximumIops 5000 -MinimumIops 500
```

### VMware Storage Performance

```powershell
# Check datastore latency via PowerCLI
Get-Datastore | Get-Stat -Stat "datastore.totalReadLatency.average",
    "datastore.totalWriteLatency.average" -Realtime |
    Select-Object Entity, MetricId, Value |
    Format-Table

# Set storage I/O control (SIOC) on datastores
Get-Datastore "Production-DS1" | Get-View | ForEach-Object {
    $_.ConfigureDatastoreIORM_Task(
        (New-Object VMware.Vim.StorageIORMConfigSpec -Property @{
            Enabled = $true
            CongestionThresholdMode = "automatic"
        })
    )
}
```

---

## 10. Capacity Planning

### Storage Growth Modeling

```
Data collection period: Minimum 3 months (12 months ideal)

Metrics to track:
- Total allocated capacity
- Total used capacity  
- Monthly growth (GB and %)
- Growth by storage tier
- Application-specific growth (DB logs, backup, user data)

Projection formula:
  Current: 10 TB used
  Monthly growth rate: 5%
  Month N projection: 10 TB × (1 + 0.05)^N

Months to 80% capacity (procurement trigger):
  Array capacity: 20 TB
  80% threshold: 16 TB
  Current used: 10 TB
  Remaining headroom: 6 TB
  At 5%/month: ~10 months to threshold
  
Procurement lead time: 8–12 weeks for enterprise storage
```

### Storage Capacity Report (PowerShell)

```powershell
# Collect storage metrics from multiple servers
$Servers = "FS01","FS02","SQL01","SQL02"

$Report = foreach ($Server in $Servers) {
    $Disks = Get-WmiObject -Class Win32_LogicalDisk -ComputerName $Server `
        -Filter "DriveType=3"
    
    foreach ($Disk in $Disks) {
        [PSCustomObject]@{
            Server = $Server
            Drive = $Disk.DeviceID
            SizeGB = [math]::Round($Disk.Size / 1GB, 1)
            FreeGB = [math]::Round($Disk.FreeSpace / 1GB, 1)
            UsedGB = [math]::Round(($Disk.Size - $Disk.FreeSpace) / 1GB, 1)
            UsedPct = [math]::Round(($Disk.Size - $Disk.FreeSpace) / $Disk.Size * 100, 1)
            Status = if (($Disk.FreeSpace / $Disk.Size) -lt 0.1) { "CRITICAL" }
                     elseif (($Disk.FreeSpace / $Disk.Size) -lt 0.2) { "WARNING" }
                     else { "OK" }
        }
    }
}

$Report | Sort-Object UsedPct -Descending | Format-Table
$Report | Export-Csv "Storage_Report_$(Get-Date -Format yyyyMMdd).csv" -NoTypeInformation

# Alert on critical drives
$Report | Where-Object Status -eq "CRITICAL" | ForEach-Object {
    Write-Warning "$($_.Server) $($_.Drive): Only $($_.FreeGB)GB free ($($_.UsedPct)% used)"
}
```
