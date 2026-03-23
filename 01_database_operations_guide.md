# Database Operations Guide for IT Administrators

## Table of Contents
1. [Database Fundamentals for IT Ops](#fundamentals)
2. [SQL Server Administration](#sql-server)
3. [MySQL / MariaDB Administration](#mysql)
4. [PostgreSQL Administration](#postgresql)
5. [Database Backup & Recovery](#backup-recovery)
6. [Performance Monitoring](#performance)
7. [Security & Access Control](#security)
8. [High Availability & Replication](#ha-replication)
9. [Common Troubleshooting](#troubleshooting)
10. [Maintenance Procedures](#maintenance)

---

## 1. Database Fundamentals for IT Ops

### Why IT Ops Needs Database Knowledge
- Applications depend on databases — DB issues manifest as app outages
- Storage, network, and OS problems impact DB performance
- Backup validation requires understanding DB state
- Security audits include database access reviews
- Capacity planning requires DB growth projections

### Key Concepts

**ACID Properties:**
- **Atomicity** — Transaction is all-or-nothing
- **Consistency** — Database remains in valid state
- **Isolation** — Concurrent transactions don't interfere
- **Durability** — Committed transactions survive failures

**Database Types:**
```
Relational (SQL):     SQL Server, MySQL, PostgreSQL, Oracle
Document:             MongoDB, CouchDB
Key-Value:            Redis, DynamoDB
Column-Family:        Cassandra, HBase
Graph:                Neo4j, Amazon Neptune
Time-Series:          InfluxDB, TimescaleDB
```

**Common Database Files (SQL Server):**
```
.mdf  — Primary data file
.ndf  — Secondary data file
.ldf  — Transaction log file
.bak  — Backup file
.trn  — Transaction log backup
```

---

## 2. SQL Server Administration

### SQL Server Services

| Service | Name | Purpose |
|---------|------|---------|
| Database Engine | MSSQLSERVER | Core DB service |
| SQL Agent | SQLSERVERAGENT | Scheduled jobs |
| SSRS | ReportServer | Reporting |
| SSAS | MSSQLServerOLAPService | Analysis |
| SSIS | MsDtsServer150 | Integration |

```powershell
# Check SQL Server services
Get-Service -Name "MSSQL*","SQL*" | Select Name, Status, StartType

# Start/Stop SQL Server
Start-Service MSSQLSERVER
Stop-Service MSSQLSERVER -Force

# Check SQL Server version
Invoke-Sqlcmd -Query "SELECT @@VERSION" -ServerInstance "localhost"
```

### Essential T-SQL for IT Admins

```sql
-- Check database sizes
SELECT 
    name AS DatabaseName,
    ROUND(SUM(size * 8.0 / 1024), 2) AS SizeMB,
    ROUND(SUM(FILEPROPERTY(name, 'SpaceUsed') * 8.0 / 1024), 2) AS UsedMB
FROM sys.master_files
GROUP BY name
ORDER BY SizeMB DESC;

-- Check active connections
SELECT 
    DB_NAME(dbid) AS DBName,
    COUNT(*) AS Connections,
    loginame
FROM sysprocesses
WHERE dbid > 0
GROUP BY dbid, loginame
ORDER BY Connections DESC;

-- Kill a blocking session
KILL 57;  -- Replace 57 with SPID

-- Check blocking
SELECT 
    blocking_session_id,
    session_id,
    wait_type,
    wait_time,
    wait_resource,
    DB_NAME(database_id) AS DatabaseName
FROM sys.dm_exec_requests
WHERE blocking_session_id > 0;

-- Check disk usage by database
SELECT 
    DB_NAME() AS DbName,
    name AS FileName,
    physical_name,
    ROUND(size * 8.0 / 1024, 2) AS SizeMB,
    ROUND(FILEPROPERTY(name,'SpaceUsed') * 8.0 / 1024, 2) AS UsedMB,
    ROUND((size - FILEPROPERTY(name,'SpaceUsed')) * 8.0 / 1024, 2) AS FreeMB
FROM sys.database_files;

-- Check SQL Agent jobs and last run status
SELECT 
    j.name AS JobName,
    jh.run_status,
    msdb.dbo.agent_datetime(jh.run_date, jh.run_time) AS LastRun,
    CASE jh.run_status
        WHEN 0 THEN 'Failed'
        WHEN 1 THEN 'Succeeded'
        WHEN 2 THEN 'Retry'
        WHEN 3 THEN 'Canceled'
    END AS Status
FROM msdb.dbo.sysjobs j
JOIN msdb.dbo.sysjobhistory jh ON j.job_id = jh.job_id
WHERE jh.step_id = 0
ORDER BY LastRun DESC;

-- Check backup history
SELECT 
    database_name,
    backup_type = CASE type WHEN 'D' THEN 'Full' WHEN 'L' THEN 'Log' WHEN 'I' THEN 'Diff' END,
    backup_start_date,
    backup_finish_date,
    ROUND(backup_size / 1048576.0, 2) AS SizeMB,
    physical_device_name
FROM msdb.dbo.backupset bs
JOIN msdb.dbo.backupmediafamily bm ON bs.media_set_id = bm.media_set_id
ORDER BY backup_start_date DESC;
```

### SQL Server Backup Commands

```sql
-- Full backup
BACKUP DATABASE [MyDB] 
TO DISK = 'D:\Backups\MyDB_Full.bak'
WITH COMPRESSION, CHECKSUM, STATS = 10;

-- Differential backup
BACKUP DATABASE [MyDB]
TO DISK = 'D:\Backups\MyDB_Diff.bak'
WITH DIFFERENTIAL, COMPRESSION, CHECKSUM;

-- Transaction log backup
BACKUP LOG [MyDB]
TO DISK = 'D:\Backups\MyDB_Log.trn'
WITH COMPRESSION, CHECKSUM;

-- Restore (with NORECOVERY for more log restores)
RESTORE DATABASE [MyDB_Test]
FROM DISK = 'D:\Backups\MyDB_Full.bak'
WITH MOVE 'MyDB' TO 'D:\Data\MyDB_Test.mdf',
     MOVE 'MyDB_log' TO 'D:\Logs\MyDB_Test.ldf',
     NORECOVERY, STATS = 10;

-- Apply differential
RESTORE DATABASE [MyDB_Test]
FROM DISK = 'D:\Backups\MyDB_Diff.bak'
WITH NORECOVERY;

-- Bring online
RESTORE DATABASE [MyDB_Test] WITH RECOVERY;

-- Verify backup without restoring
RESTORE VERIFYONLY FROM DISK = 'D:\Backups\MyDB_Full.bak';
```

### SQL Server Performance Queries

```sql
-- Top 10 CPU-intensive queries
SELECT TOP 10
    qs.total_worker_time / qs.execution_count AS AvgCPU,
    qs.execution_count,
    qs.total_worker_time,
    SUBSTRING(qt.TEXT, (qs.statement_start_offset/2)+1,
        ((CASE qs.statement_end_offset
            WHEN -1 THEN DATALENGTH(qt.TEXT)
            ELSE qs.statement_end_offset END 
        - qs.statement_start_offset)/2)+1) AS QueryText
FROM sys.dm_exec_query_stats qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) qt
ORDER BY AvgCPU DESC;

-- Check missing indexes
SELECT TOP 20
    migs.avg_total_user_cost * migs.avg_user_impact * (migs.user_seeks + migs.user_scans) AS ImprovementMeasure,
    mid.statement AS TableName,
    mid.equality_columns,
    mid.inequality_columns,
    mid.included_columns
FROM sys.dm_db_missing_index_groups mig
JOIN sys.dm_db_missing_index_group_stats migs ON mig.index_group_handle = migs.group_handle
JOIN sys.dm_db_missing_index_details mid ON mig.index_handle = mid.index_handle
ORDER BY ImprovementMeasure DESC;

-- Check index fragmentation
SELECT 
    OBJECT_NAME(ind.OBJECT_ID) AS TableName,
    ind.name AS IndexName,
    indexstats.index_type_desc,
    indexstats.avg_fragmentation_in_percent,
    indexstats.page_count
FROM sys.dm_db_index_physical_stats(DB_ID(), NULL, NULL, NULL, NULL) indexstats
JOIN sys.indexes ind ON ind.object_id = indexstats.object_id
    AND ind.index_id = indexstats.index_id
WHERE indexstats.avg_fragmentation_in_percent > 10
ORDER BY avg_fragmentation_in_percent DESC;
```

---

## 3. MySQL / MariaDB Administration

### Essential MySQL Commands

```sql
-- Show all databases
SHOW DATABASES;

-- Database size
SELECT 
    table_schema AS 'Database',
    ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS 'Size (MB)'
FROM information_schema.tables
GROUP BY table_schema
ORDER BY SUM(data_length + index_length) DESC;

-- Active connections
SHOW PROCESSLIST;
SHOW FULL PROCESSLIST;

-- Kill a query
KILL QUERY 1234;  -- Kill just the query
KILL 1234;        -- Kill the connection

-- Check replication status
SHOW MASTER STATUS\G
SHOW SLAVE STATUS\G

-- Check slow queries
SHOW VARIABLES LIKE 'slow_query%';
SHOW VARIABLES LIKE 'long_query_time';

-- Check InnoDB status
SHOW ENGINE INNODB STATUS\G

-- User management
CREATE USER 'appuser'@'192.168.1.%' IDENTIFIED BY 'StrongPassword123!';
GRANT SELECT, INSERT, UPDATE, DELETE ON myapp.* TO 'appuser'@'192.168.1.%';
FLUSH PRIVILEGES;

-- Show grants
SHOW GRANTS FOR 'appuser'@'192.168.1.%';

-- Remove user
DROP USER 'appuser'@'192.168.1.%';
```

### MySQL Backup (mysqldump)

```bash
# Full database backup
mysqldump -u root -p \
  --single-transaction \
  --routines \
  --triggers \
  --all-databases \
  --master-data=2 \
  | gzip > /backups/mysql_full_$(date +%Y%m%d).sql.gz

# Single database backup
mysqldump -u root -p \
  --single-transaction \
  myapp > /backups/myapp_$(date +%Y%m%d).sql

# Restore from backup
mysql -u root -p myapp < /backups/myapp_20240101.sql

# Point-in-time recovery using binary logs
mysqlbinlog --start-datetime="2024-01-01 00:00:00" \
            --stop-datetime="2024-01-01 12:00:00" \
            /var/lib/mysql/mysql-bin.000001 | mysql -u root -p
```

### MySQL my.cnf Key Settings

```ini
[mysqld]
# InnoDB Buffer Pool (70-80% of available RAM)
innodb_buffer_pool_size = 8G

# Log file size
innodb_log_file_size = 512M

# Max connections
max_connections = 500

# Query cache (disabled in MySQL 8+)
# query_cache_size = 0

# Slow query log
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log
long_query_time = 2

# Binary logging for replication/PITR
log_bin = /var/lib/mysql/mysql-bin
expire_logs_days = 7
binlog_format = ROW

# Character set
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
```

---

## 4. PostgreSQL Administration

### Essential psql Commands

```sql
-- List databases
\l

-- Connect to database
\c myapp

-- List tables
\dt

-- Describe table
\d tablename

-- List users/roles
\du

-- Check connections
SELECT pid, usename, application_name, client_addr, state, query
FROM pg_stat_activity
WHERE state != 'idle';

-- Kill a query
SELECT pg_terminate_backend(pid) FROM pg_stat_activity 
WHERE pid = 12345;

-- Kill all connections to a database (for maintenance)
SELECT pg_terminate_backend(pid) FROM pg_stat_activity 
WHERE datname = 'myapp' AND pid <> pg_backend_pid();

-- Database sizes
SELECT datname, pg_size_pretty(pg_database_size(datname)) 
FROM pg_database ORDER BY pg_database_size(datname) DESC;

-- Table sizes
SELECT tablename, pg_size_pretty(pg_total_relation_size(tablename::regclass))
FROM pg_tables WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(tablename::regclass) DESC;

-- Check for bloat / dead tuples
SELECT relname, n_dead_tup, n_live_tup, 
       ROUND(n_dead_tup * 100.0 / NULLIF(n_live_tup + n_dead_tup, 0), 2) AS bloat_pct
FROM pg_stat_user_tables
ORDER BY n_dead_tup DESC;

-- Create user
CREATE ROLE appuser WITH LOGIN PASSWORD 'StrongPass123!';
GRANT CONNECT ON DATABASE myapp TO appuser;
GRANT USAGE ON SCHEMA public TO appuser;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO appuser;
```

### PostgreSQL Backup

```bash
# pg_dump — single database logical backup
pg_dump -U postgres -Fc myapp > /backups/myapp_$(date +%Y%m%d).dump

# Restore
pg_restore -U postgres -d myapp /backups/myapp_20240101.dump

# pg_dumpall — all databases including globals
pg_dumpall -U postgres > /backups/pg_all_$(date +%Y%m%d).sql

# Continuous archiving (WAL) — enable in postgresql.conf
# archive_mode = on
# archive_command = 'cp %p /mnt/wal_archive/%f'
```

---

## 5. Database Backup & Recovery

### Backup Strategy

**3-2-1-1-0 Rule Applied to Databases:**
```
3 copies of data
2 different media types (local disk + tape or cloud)
1 offsite copy
1 offline/immutable copy
0 untested backups — always verify restores
```

**Backup Types:**
```
Full:           Complete copy of entire database
                Frequency: Weekly minimum
                Restore: Single file needed
                
Differential:   Changes since last full backup
                Frequency: Daily
                Restore: Full + most recent differential

Transaction Log: All transactions since last log backup
                 Frequency: Every 15–60 minutes
                 Restore: Full + diff + all logs = point-in-time
```

### Recovery Time Planning

```
Recovery Point Objective (RPO) = max acceptable data loss
Recovery Time Objective (RTO) = max acceptable downtime

RPO 1 hour  → Transaction log backups every 60 min
RPO 15 min  → Log backups every 15 min
RPO ~0      → Log shipping or Always On synchronous replica

RTO 4 hours → Single server restore from backup
RTO 1 hour  → Warm standby (log shipping)
RTO < 5 min → Hot standby (Always On / streaming replication)
```

### Backup Validation Script

```powershell
# SQL Server - Test restore to verify backup integrity
$BackupFile = "D:\Backups\MyDB_Full.bak"
$TestDB = "MyDB_RestoreTest"

Invoke-Sqlcmd -Query @"
RESTORE DATABASE [$TestDB]
FROM DISK = '$BackupFile'
WITH MOVE 'MyDB' TO 'D:\TestData\MyDB_Test.mdf',
     MOVE 'MyDB_log' TO 'D:\TestData\MyDB_Test.ldf',
     REPLACE, RECOVERY, STATS = 10;
"@

# Verify row count
$Count = Invoke-Sqlcmd -Query "SELECT COUNT(*) FROM [$TestDB].dbo.ImportantTable"
Write-Output "Row count: $($Count.Column1)"

# Drop test DB
Invoke-Sqlcmd -Query "DROP DATABASE [$TestDB]"
```

---

## 6. Performance Monitoring

### Key Metrics to Monitor

| Metric | Warning | Critical | Tool |
|--------|---------|---------|------|
| CPU % | > 80% sustained | > 95% | OS monitor |
| RAM/Buffer Pool | > 90% used | > 98% | DB internals |
| Disk I/O latency | > 10ms reads | > 50ms | Storage monitor |
| Disk space | < 20% free | < 10% free | OS monitor |
| Active connections | > 80% max | > 95% max | DB monitor |
| Replication lag | > 30 sec | > 5 min | Replication status |
| Long-running queries | > 1 min | > 5 min | Query monitor |
| Deadlocks | Any | > 5/hour | DB error log |

### SQL Server: Set Up Monitoring Alerts

```sql
-- Enable SQL Agent alerts for critical errors
EXEC msdb.dbo.sp_add_alert
    @name = 'Severity 17 Error',
    @severity = 17,
    @enabled = 1,
    @delay_between_responses = 60,
    @notification_message = 'SQL Server Severity 17 error occurred';

-- Notify operator
EXEC msdb.dbo.sp_add_notification
    @alert_name = 'Severity 17 Error',
    @operator_name = 'DBA Team',
    @notification_method = 1; -- Email
```

---

## 7. Security & Access Control

### Principle of Least Privilege

```sql
-- SQL Server: Application-specific login
-- Never use sa for applications
CREATE LOGIN AppServiceAccount WITH PASSWORD = 'ComplexPass123!';
USE MyApp;
CREATE USER AppServiceAccount FOR LOGIN AppServiceAccount;
GRANT SELECT, INSERT, UPDATE, DELETE ON SCHEMA::dbo TO AppServiceAccount;
-- DO NOT grant: db_owner, sysadmin, ALTER, DROP

-- Audit logins
SELECT name, type_desc, is_disabled, create_date, modify_date
FROM sys.server_principals
WHERE type IN ('S','U') -- SQL and Windows logins
ORDER BY create_date DESC;

-- Find SQL logins with blank passwords
SELECT name FROM sys.sql_logins WHERE PWDCOMPARE('', password_hash) = 1;

-- Check sysadmin members (should be minimal)
SELECT p.name, p.type_desc
FROM sys.server_role_members rm
JOIN sys.server_principals r ON rm.role_principal_id = r.principal_id
JOIN sys.server_principals p ON rm.member_principal_id = p.principal_id
WHERE r.name = 'sysadmin';
```

### Database Encryption

```sql
-- Transparent Data Encryption (TDE) — SQL Server
-- Step 1: Master key
USE master;
CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'MasterKeyPass!';

-- Step 2: Certificate
CREATE CERTIFICATE TDE_Cert WITH SUBJECT = 'TDE Certificate';

-- Step 3: Backup certificate (CRITICAL - do this immediately)
BACKUP CERTIFICATE TDE_Cert TO FILE = 'D:\Certs\TDE_Cert.cer'
WITH PRIVATE KEY (
    FILE = 'D:\Certs\TDE_Cert.pvk',
    ENCRYPTION BY PASSWORD = 'CertBackupPass!'
);

-- Step 4: Enable TDE on database
USE MyApp;
CREATE DATABASE ENCRYPTION KEY
    WITH ALGORITHM = AES_256
    ENCRYPTION BY SERVER CERTIFICATE TDE_Cert;
ALTER DATABASE MyApp SET ENCRYPTION ON;
```

---

## 8. High Availability & Replication

### SQL Server Always On Availability Groups

**Architecture:**
```
Primary Replica ──── Synchronous Commit ────► Secondary (HA)
Primary Replica ──── Asynchronous Commit ───► Secondary (DR, read scale)

Listener: Virtual IP that clients connect to
Automatic failover: Synchronous replicas only
```

**Key Commands:**
```sql
-- Check AG health
SELECT 
    ag.name AS AGName,
    ar.replica_server_name,
    ars.role_desc,
    ars.synchronization_health_desc,
    ars.connected_state_desc,
    ars.operational_state_desc
FROM sys.availability_groups ag
JOIN sys.availability_replicas ar ON ag.group_id = ar.group_id
JOIN sys.dm_hadr_availability_replica_states ars ON ar.replica_id = ars.replica_id;

-- Check DB synchronization state
SELECT 
    ag.name,
    d.name AS DatabaseName,
    drs.synchronization_state_desc,
    drs.synchronization_health_desc,
    drs.log_send_queue_size,
    drs.redo_queue_size
FROM sys.availability_groups ag
JOIN sys.availability_databases_cluster adc ON ag.group_id = adc.group_id
JOIN sys.databases d ON adc.database_name = d.name
JOIN sys.dm_hadr_database_replica_states drs ON d.database_id = drs.database_id;

-- Manual failover (planned, no data loss)
ALTER AVAILABILITY GROUP [AGName] FAILOVER;

-- Forced failover (unplanned, potential data loss)
ALTER AVAILABILITY GROUP [AGName] FORCE_FAILOVER_ALLOW_DATA_LOSS;
```

### MySQL Replication

```sql
-- On Primary: Check binary log position
SHOW MASTER STATUS;

-- On Replica: Configure replication
CHANGE MASTER TO
    MASTER_HOST = '192.168.1.10',
    MASTER_USER = 'replicator',
    MASTER_PASSWORD = 'ReplPass123!',
    MASTER_LOG_FILE = 'mysql-bin.000001',
    MASTER_LOG_POS = 154;

START SLAVE;
SHOW SLAVE STATUS\G

-- Check replication lag
-- Key fields in SHOW SLAVE STATUS:
-- Seconds_Behind_Master: replication lag in seconds
-- Slave_IO_Running: Yes/No — binary log download
-- Slave_SQL_Running: Yes/No — applying events
```

---

## 9. Common Troubleshooting

### Database Won't Start

```
SQL Server:
1. Check Windows Event Log (Application + System)
2. Check SQL Server error log: C:\Program Files\Microsoft SQL Server\
   MSSQL15.MSSQLSERVER\MSSQL\Log\ERRORLOG
3. Common causes:
   - Disk full (data/log files can't grow)
   - master database corrupted
   - SQL Server service account locked out
   - Port 1433 conflict

MySQL:
1. Check /var/log/mysql/error.log
2. Common causes:
   - InnoDB crash recovery needed
   - My.cnf syntax error (test: mysqld --validate-config)
   - Port 3306 already in use
   - Disk full (check /var/lib/mysql)
```

### Application Can't Connect to Database

```
Troubleshooting checklist:
1. Can you ping the DB server from the app server?
2. Is the DB service running? (net start / systemctl)
3. Is the correct port open? (telnet dbserver 1433)
4. Does the connection string have correct server/instance name?
5. Is the login account enabled and not locked?
6. Does the account have permissions to the database?
7. Is the firewall (host + network) allowing the connection?
8. Check DB server max connections not exceeded
9. Check app pool credentials match DB login

SQL Server connection string examples:
Server=myServer;Database=myDB;User Id=myUser;Password=myPass;
Server=myServer\SQLEXPRESS;Database=myDB;Trusted_Connection=True;
Server=myServer,1433;Database=myDB;User Id=myUser;Password=myPass;
```

### Database Disk Space Full

```sql
-- Immediate: Find largest tables
SELECT TOP 20
    OBJECT_SCHEMA_NAME(t.OBJECT_ID) AS SchemaName,
    t.Name AS TableName,
    p.rows AS RowCounts,
    ROUND((SUM(a.total_pages) * 8) / 1024.0, 2) AS TotalMB
FROM sys.tables t
JOIN sys.indexes i ON t.OBJECT_ID = i.object_id
JOIN sys.partitions p ON i.object_id = p.OBJECT_ID AND i.index_id = p.index_id
JOIN sys.allocation_units a ON p.partition_id = a.container_id
GROUP BY t.Name, t.OBJECT_ID, p.Rows
ORDER BY TotalMB DESC;

-- Shrink log file (emergency only - not recommended regularly)
USE MyDB;
DBCC SHRINKFILE (MyDB_log, 100); -- Shrink to 100MB

-- Better: Set log to simple recovery temporarily
ALTER DATABASE MyDB SET RECOVERY SIMPLE;
DBCC SHRINKFILE (MyDB_log, 100);
ALTER DATABASE MyDB SET RECOVERY FULL;
-- Then take a full backup to restart log chain
```

---

## 10. Maintenance Procedures

### Weekly Maintenance Checklist

```sql
-- SQL Server maintenance job template

-- 1. Update statistics
EXEC sp_updatestats;

-- 2. Rebuild/reorganize indexes
DECLARE @TableName NVARCHAR(256)
DECLARE @IndexName NVARCHAR(256)
DECLARE @Fragmentation FLOAT

DECLARE IndexCursor CURSOR FOR
SELECT 
    OBJECT_NAME(i.object_id),
    i.name,
    ips.avg_fragmentation_in_percent
FROM sys.dm_db_index_physical_stats(DB_ID(), NULL, NULL, NULL, 'LIMITED') ips
JOIN sys.indexes i ON ips.object_id = i.object_id AND ips.index_id = i.index_id
WHERE ips.avg_fragmentation_in_percent > 5 AND ips.page_count > 1000

OPEN IndexCursor
FETCH NEXT FROM IndexCursor INTO @TableName, @IndexName, @Fragmentation

WHILE @@FETCH_STATUS = 0
BEGIN
    IF @Fragmentation > 30
        EXEC('ALTER INDEX [' + @IndexName + '] ON [' + @TableName + '] REBUILD')
    ELSE
        EXEC('ALTER INDEX [' + @IndexName + '] ON [' + @TableName + '] REORGANIZE')
    
    FETCH NEXT FROM IndexCursor INTO @TableName, @IndexName, @Fragmentation
END
CLOSE IndexCursor
DEALLOCATE IndexCursor

-- 3. Check database integrity
DBCC CHECKDB WITH NO_INFOMSGS, ALL_ERRORMSGS;
```

### Database Health Report Script (PowerShell)

```powershell
$ServerInstance = "localhost"
$Report = @()

# Get all databases
$DBs = Invoke-Sqlcmd -ServerInstance $ServerInstance `
    -Query "SELECT name, state_desc, recovery_model_desc FROM sys.databases WHERE state_desc = 'ONLINE'"

foreach ($DB in $DBs) {
    # Last backup
    $LastBackup = Invoke-Sqlcmd -ServerInstance $ServerInstance -Query @"
        SELECT TOP 1 backup_finish_date, type
        FROM msdb.dbo.backupset
        WHERE database_name = '$($DB.name)'
        ORDER BY backup_finish_date DESC
"@

    $DaysSinceBackup = if ($LastBackup) {
        (Get-Date) - $LastBackup.backup_finish_date | Select-Object -ExpandProperty TotalDays
    } else { 9999 }

    $Report += [PSCustomObject]@{
        Database = $DB.name
        State = $DB.state_desc
        RecoveryModel = $DB.recovery_model_desc
        LastBackup = if ($LastBackup) { $LastBackup.backup_finish_date } else { "NEVER" }
        DaysSinceBackup = [math]::Round($DaysSinceBackup, 1)
        Status = if ($DaysSinceBackup -gt 1) { "WARNING" } else { "OK" }
    }
}

$Report | Format-Table -AutoSize
$Report | Where-Object Status -eq "WARNING" | ForEach-Object {
    Write-Warning "Database $($_.Database) last backed up $($_.DaysSinceBackup) days ago!"
}
```
