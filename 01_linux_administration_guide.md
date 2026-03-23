# Linux Administration Guide
**Version:** 2.0 | **Audience:** L2–L4 SysAdmins | **Last Updated:** 2025

---

## Table of Contents
1. [Linux Fundamentals](#1-linux-fundamentals)
2. [File System & Storage](#2-file-system--storage)
3. [User & Group Management](#3-user--group-management)
4. [Process & Service Management](#4-process--service-management)
5. [Networking on Linux](#5-networking-on-linux)
6. [Package Management](#6-package-management)
7. [Security Hardening](#7-security-hardening)
8. [Shell Scripting & Automation](#8-shell-scripting--automation)
9. [Performance Monitoring & Tuning](#9-performance-monitoring--tuning)
10. [Logging & Auditing](#10-logging--auditing)
11. [Backup & Recovery](#11-backup--recovery)
12. [Troubleshooting Playbook](#12-troubleshooting-playbook)

---

## 1. Linux Fundamentals

### Distributions Overview

| Distro Family | Examples | Package Manager | Use Case |
|---|---|---|---|
| RHEL/CentOS | RHEL 9, Rocky 9, AlmaLinux | dnf/yum | Enterprise servers |
| Debian/Ubuntu | Ubuntu 22.04 LTS, Debian 12 | apt | General purpose, cloud |
| SUSE | SLES 15, openSUSE | zypper | SAP, enterprise |
| Arch | Arch, Manjaro | pacman | Rolling release, desktop |

### Filesystem Hierarchy Standard (FHS)

```
/               Root of the filesystem
├── bin/        Essential user binaries
├── boot/       Boot loader files, kernel
├── dev/        Device files
├── etc/        System configuration files
├── home/       User home directories
├── lib/        Essential shared libraries
├── media/      Mount points for removable media
├── mnt/        Temporary mount points
├── opt/        Optional/third-party software
├── proc/       Virtual filesystem for process info
├── root/       Root user's home directory
├── run/        Runtime variable data (tmpfs)
├── sbin/       System binaries (root use)
├── srv/        Service data
├── sys/        Virtual filesystem for kernel objects
├── tmp/        Temporary files (cleared on reboot)
├── usr/        User utilities and applications
│   ├── bin/    User commands
│   ├── lib/    Libraries
│   ├── local/  Locally compiled software
│   └── share/  Architecture-independent data
└── var/        Variable data (logs, spool, etc.)
    ├── log/
    ├── spool/
    └── www/
```

### Essential Commands Quick Reference

```bash
# Navigation
pwd                     # Print working directory
ls -lah                 # List with details, hidden, human-readable
cd -                    # Return to previous directory
find / -name "*.conf" -type f 2>/dev/null
locate filename         # Faster search using mlocate db

# File Operations
cp -rp source/ dest/    # Copy recursive, preserve attributes
mv file newname         # Move/rename
rm -rf directory/       # Remove recursive (CAUTION)
ln -s /path/to/target linkname   # Symbolic link
rsync -avz --progress source/ dest/  # Sync with progress

# File Content
cat, less, more, head -n 20, tail -f /var/log/syslog
grep -rn "pattern" /etc/
grep -E "error|warn" /var/log/messages
awk '{print $1, $4}' file.log
sed -i 's/old/new/g' file.conf

# Permissions
chmod 755 file          # rwxr-xr-x
chmod u+x,g-w file      # Symbolic mode
chown user:group file
chown -R www-data:www-data /var/www/
umask 022               # Default permission mask
```

### Run Levels & Targets (systemd)

```bash
# systemd targets (replace old runlevels)
systemctl get-default           # Show current target
systemctl set-default multi-user.target   # Set default (no GUI)
systemctl isolate rescue.target  # Switch target now

# Target mapping
# runlevel 0 = poweroff.target
# runlevel 1 = rescue.target
# runlevel 3 = multi-user.target
# runlevel 5 = graphical.target
# runlevel 6 = reboot.target
```

---

## 2. File System & Storage

### Partition Management

```bash
# List block devices
lsblk -o NAME,SIZE,FSTYPE,MOUNTPOINT,UUID
fdisk -l
parted -l

# Create partitions (fdisk)
fdisk /dev/sdb
# Commands: n (new), p (primary), w (write), q (quit), d (delete), p (print)

# GPT partitioning (preferred for large disks)
gdisk /dev/sdb
parted /dev/sdb mklabel gpt
parted /dev/sdb mkpart primary ext4 0% 100%

# Format
mkfs.ext4 /dev/sdb1
mkfs.xfs /dev/sdb1
mkfs.vfat /dev/sdb1       # FAT32

# Check and repair
fsck -y /dev/sdb1         # Auto-repair (unmounted only!)
e2fsck -f /dev/sdb1
xfs_repair /dev/sdb1
```

### LVM (Logical Volume Manager)

```bash
# LVM hierarchy: Physical Volume (PV) → Volume Group (VG) → Logical Volume (LV)

# Create PV
pvcreate /dev/sdb /dev/sdc
pvdisplay; pvs

# Create VG
vgcreate vg_data /dev/sdb /dev/sdc
vgdisplay; vgs

# Create LV
lvcreate -L 50G -n lv_data vg_data
lvcreate -l 100%FREE -n lv_data vg_data
lvdisplay; lvs

# Format and mount
mkfs.ext4 /dev/vg_data/lv_data
mount /dev/vg_data/lv_data /data

# Extend LV (online for ext4/xfs)
lvextend -L +20G /dev/vg_data/lv_data
resize2fs /dev/vg_data/lv_data     # ext4
xfs_growfs /data                    # xfs

# Add disk to VG
pvcreate /dev/sdd
vgextend vg_data /dev/sdd

# Snapshot
lvcreate -s -L 10G -n lv_data_snap /dev/vg_data/lv_data
```

### /etc/fstab

```
# <device>               <mount>     <type>  <options>              <dump> <pass>
UUID=abc123              /           ext4    defaults,errors=remount-ro  0  1
UUID=def456              /data       xfs     defaults,noatime        0      2
//server/share           /mnt/share  cifs    credentials=/etc/.cifs,uid=1000  0  0
server:/export           /mnt/nfs    nfs     defaults,_netdev        0      0
tmpfs                    /tmp        tmpfs   defaults,size=2G        0      0
```

```bash
# Test fstab without reboot
mount -a        # Mount all in fstab
systemctl daemon-reload
```

### RAID (mdadm)

```bash
# RAID 1 mirror
mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc
echo "DEVICE /dev/sdb /dev/sdc" >> /etc/mdadm/mdadm.conf
mdadm --detail --scan >> /etc/mdadm/mdadm.conf

# Monitor
mdadm --detail /dev/md0
cat /proc/mdstat

# Add spare
mdadm /dev/md0 --add /dev/sdd

# Fail and remove
mdadm /dev/md0 --fail /dev/sdb
mdadm /dev/md0 --remove /dev/sdb
```

---

## 3. User & Group Management

### User Administration

```bash
# Create user
useradd -m -s /bin/bash -G sudo,docker -c "John Doe" jdoe
useradd -r -s /sbin/nologin serviceaccount   # System/service account

# Set password
passwd jdoe
echo "jdoe:TempPass123!" | chpasswd

# Modify user
usermod -aG groupname jdoe    # Add to group
usermod -s /bin/bash jdoe     # Change shell
usermod -L jdoe               # Lock account
usermod -U jdoe               # Unlock account
usermod -e 2025-12-31 jdoe   # Set expiry

# Delete user
userdel -r jdoe               # -r removes home directory

# User info
id jdoe
finger jdoe
getent passwd jdoe
last jdoe                     # Login history
lastlog -u jdoe
```

### /etc/passwd and /etc/shadow

```
# /etc/passwd format:
# username:x:UID:GID:comment:home:shell
jdoe:x:1001:1001:John Doe:/home/jdoe:/bin/bash

# /etc/shadow format:
# username:hashed_pw:lastchange:min:max:warn:inactive:expire
jdoe:$6$hash....:18000:0:90:7:30:

# Password aging
chage -l jdoe                 # Show aging info
chage -M 90 -W 14 -I 30 jdoe # Max 90 days, warn 14, inactive after 30
chage -d 0 jdoe               # Force change on next login
```

### Group Management

```bash
# Create/modify/delete groups
groupadd developers
groupmod -n devs developers    # Rename
groupdel devs

# Manage members
gpasswd -a jdoe developers     # Add
gpasswd -d jdoe developers     # Remove
gpasswd -M user1,user2 developers  # Set members

# View
getent group developers
groups jdoe
```

### sudo Configuration

```bash
# Edit sudoers safely
visudo

# /etc/sudoers entries:
jdoe    ALL=(ALL:ALL) ALL                    # Full sudo
%admins ALL=(ALL) ALL                        # Group sudo
jdoe    ALL=(ALL) NOPASSWD: /sbin/reboot     # No password for specific command
jdoe    ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart nginx

# Include drop-in files
# /etc/sudoers.d/devops
%devops ALL=(ALL) NOPASSWD: /usr/bin/docker, /usr/bin/kubectl
```

### PAM (Pluggable Authentication Modules)

```bash
# Key PAM files
/etc/pam.d/sshd
/etc/pam.d/sudo
/etc/pam.d/common-auth      # Debian/Ubuntu
/etc/pam.d/system-auth      # RHEL

# Password complexity (pwquality)
# /etc/security/pwquality.conf
minlen = 14
dcredit = -1     # At least 1 digit
ucredit = -1     # At least 1 uppercase
lcredit = -1     # At least 1 lowercase
ocredit = -1     # At least 1 special
maxrepeat = 3
```

---

## 4. Process & Service Management

### systemd Service Management

```bash
# Service control
systemctl start nginx
systemctl stop nginx
systemctl restart nginx
systemctl reload nginx          # Reload config without stopping
systemctl enable nginx          # Start at boot
systemctl disable nginx
systemctl mask nginx            # Prevent starting entirely
systemctl status nginx -l       # Detailed status with logs

# View all units
systemctl list-units --type=service --state=running
systemctl list-unit-files --type=service

# Analyze boot time
systemd-analyze blame
systemd-analyze critical-chain
```

### Creating Custom systemd Units

```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My Application
Documentation=https://docs.myapp.com
After=network.target postgresql.service
Requires=postgresql.service

[Service]
Type=simple
User=myapp
Group=myapp
WorkingDirectory=/opt/myapp
Environment=NODE_ENV=production
EnvironmentFile=/etc/myapp/env
ExecStartPre=/opt/myapp/prestart.sh
ExecStart=/opt/myapp/bin/server
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
RestartSec=10
StandardOutput=journal
StandardError=journal
SyslogIdentifier=myapp
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

```bash
systemctl daemon-reload
systemctl enable --now myapp
```

### Process Management

```bash
# Process viewing
ps aux                        # All processes
ps aux --sort=-%cpu | head    # Top CPU
ps aux --sort=-%mem | head    # Top memory
pstree -p                     # Tree view with PIDs

# top / htop alternatives
top -b -n 1                   # Batch mode, one iteration
htop                          # Interactive (install separately)

# Kill processes
kill PID
kill -9 PID                   # SIGKILL (force)
kill -15 PID                  # SIGTERM (graceful)
killall nginx                 # Kill by name
pkill -f "python script.py"   # Kill by pattern

# Background jobs
command &                     # Run in background
jobs                          # List background jobs
fg %1                         # Bring job 1 to foreground
bg %1                         # Resume job 1 in background
nohup command &               # Immune to hangup
screen / tmux                 # Terminal multiplexers

# Process priority
nice -n 10 command            # Start with priority (higher = lower priority)
renice -n 5 -p PID           # Change running process priority
```

---

## 5. Networking on Linux

### Network Interface Configuration

```bash
# ip command (modern)
ip addr show
ip addr add 192.168.1.100/24 dev eth0
ip addr del 192.168.1.100/24 dev eth0
ip link set eth0 up/down
ip route show
ip route add default via 192.168.1.1
ip route add 10.0.0.0/8 via 192.168.1.254

# Persistent config - Ubuntu (Netplan)
# /etc/netplan/00-network.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: false
      addresses: [192.168.1.100/24]
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]

# Apply Netplan
netplan apply

# Persistent config - RHEL (nmcli)
nmcli con add type ethernet con-name eth0 ifname eth0
nmcli con mod eth0 ipv4.addresses 192.168.1.100/24
nmcli con mod eth0 ipv4.gateway 192.168.1.1
nmcli con mod eth0 ipv4.dns "8.8.8.8 8.8.4.4"
nmcli con mod eth0 ipv4.method manual
nmcli con up eth0
```

### Firewall (firewalld & iptables & nftables)

```bash
# firewalld (RHEL/CentOS)
firewall-cmd --state
firewall-cmd --list-all
firewall-cmd --add-service=https --permanent
firewall-cmd --add-port=8080/tcp --permanent
firewall-cmd --remove-service=http --permanent
firewall-cmd --reload
firewall-cmd --zone=public --list-all

# UFW (Ubuntu)
ufw status verbose
ufw allow 22/tcp
ufw allow from 192.168.1.0/24 to any port 443
ufw deny 23/tcp
ufw enable
ufw reload

# iptables (universal)
iptables -L -n -v --line-numbers
iptables -A INPUT -p tcp --dport 443 -j ACCEPT
iptables -A INPUT -s 10.0.0.0/8 -j ACCEPT
iptables -I INPUT 1 -i lo -j ACCEPT
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -P INPUT DROP      # Default deny
iptables-save > /etc/iptables/rules.v4
iptables-restore < /etc/iptables/rules.v4
```

### Network Diagnostics

```bash
# Connectivity
ping -c 4 8.8.8.8
ping6 ::1
traceroute -n 8.8.8.8
mtr --report 8.8.8.8        # Combined ping+traceroute

# DNS
dig google.com
dig @8.8.8.8 google.com A
dig -x 8.8.8.8              # Reverse lookup
nslookup google.com
host google.com

# Port and socket
ss -tulnp                   # Modern netstat
ss -tnp state established
netstat -tulnp              # Legacy
nmap -sV -p 22,80,443 target

# Packet capture
tcpdump -i eth0 port 80 -w capture.pcap
tcpdump -i any 'host 10.0.0.1 and tcp port 443'
tcpdump -r capture.pcap
```

---

## 6. Package Management

### apt (Debian/Ubuntu)

```bash
apt update                          # Update package lists
apt upgrade                         # Upgrade all packages
apt full-upgrade                    # Upgrade with dependency changes
apt install package1 package2
apt remove package                  # Remove, keep config
apt purge package                   # Remove including config
apt autoremove                      # Remove orphaned packages
apt search keyword
apt show packagename
apt list --installed
apt-cache policy packagename        # Show versions and repos

# Sources
cat /etc/apt/sources.list
ls /etc/apt/sources.list.d/
add-apt-repository ppa:name/ppa
```

### dnf/yum (RHEL/CentOS)

```bash
dnf update
dnf install httpd
dnf remove httpd
dnf search apache
dnf info httpd
dnf list installed
dnf history                         # Transaction history
dnf history undo last               # Undo last transaction
dnf provides /usr/bin/python3       # Find which package provides file
dnf repolist
dnf config-manager --add-repo URL
dnf module list
dnf module install nodejs:18/default
```

### RPM & DPKG Direct

```bash
# RPM
rpm -ivh package.rpm               # Install
rpm -Uvh package.rpm               # Upgrade
rpm -e packagename                 # Remove
rpm -qa                            # Query all installed
rpm -qi packagename                # Package info
rpm -ql packagename                # List files in package
rpm -qf /usr/bin/python3           # Which package owns file
rpm --checksig package.rpm        # Verify signature

# DPKG
dpkg -i package.deb
dpkg -r packagename
dpkg -l                            # List all
dpkg -L packagename                # List files
dpkg -S /usr/bin/python3          # Which package owns file
dpkg --configure -a               # Fix broken packages
```

---

## 7. Security Hardening

### SSH Hardening

```bash
# /etc/ssh/sshd_config best practices
Port 2222                           # Non-standard port
PermitRootLogin no
PasswordAuthentication no          # Key-only auth
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
MaxAuthTries 3
LoginGraceTime 30
ClientAliveInterval 300
ClientAliveCountMax 2
AllowUsers jdoe admin
AllowGroups sshusers
X11Forwarding no
PermitEmptyPasswords no
UseDNS no
Banner /etc/ssh/banner.txt
```

```bash
# SSH key management
ssh-keygen -t ed25519 -C "user@host" -f ~/.ssh/id_ed25519
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@server
ssh-keyscan -H server >> ~/.ssh/known_hosts

# Disable weak algorithms
# /etc/ssh/sshd_config
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com
KexAlgorithms curve25519-sha256,diffie-hellman-group16-sha512

systemctl reload sshd
```

### SELinux (RHEL)

```bash
# Status
getenforce                          # Enforcing/Permissive/Disabled
sestatus

# Set mode
setenforce 1                        # Enforcing (temporary)
setenforce 0                        # Permissive (temporary)
# Permanent: /etc/selinux/config -> SELINUX=enforcing

# Troubleshooting
ausearch -m avc -ts recent          # Recent denials
sealert -a /var/log/audit/audit.log
audit2why < /var/log/audit/audit.log

# Fix context
ls -Z /var/www/html/
chcon -t httpd_sys_content_t /var/www/html/file
restorecon -Rv /var/www/html/       # Restore default context

# Boolean management
getsebool -a | grep httpd
setsebool -P httpd_can_network_connect 1
```

### AppArmor (Ubuntu/Debian)

```bash
# Status
aa-status
apparmor_status

# Modes
aa-enforce /etc/apparmor.d/usr.sbin.nginx
aa-complain /etc/apparmor.d/usr.sbin.nginx  # Log only, don't block

# Reload profile
apparmor_parser -r /etc/apparmor.d/usr.sbin.nginx
```

### System Hardening Checklist

```bash
# Disable unused services
systemctl disable bluetooth avahi-daemon cups
systemctl mask bluetooth

# Kernel hardening (/etc/sysctl.conf or /etc/sysctl.d/99-security.conf)
net.ipv4.ip_forward = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.log_martians = 1
net.ipv4.icmp_echo_ignore_broadcasts = 1
kernel.randomize_va_space = 2
fs.suid_dumpable = 0
kernel.dmesg_restrict = 1

sysctl -p   # Apply

# Restrict cron
echo "root" > /etc/cron.allow
chmod 600 /etc/cron.allow
chmod 600 /etc/cron.deny

# Audit SUID/SGID files
find / -perm /4000 -o -perm /2000 -type f 2>/dev/null
find / -nouser -o -nogroup 2>/dev/null | head
```

---

## 8. Shell Scripting & Automation

### Bash Script Best Practices

```bash
#!/usr/bin/env bash
# Script: backup_configs.sh
# Purpose: Back up system config files
# Author: IT Operations
# Date: 2025

set -euo pipefail                   # Exit on error, unbound var, pipe fail
IFS=$'\n\t'                         # Safer word splitting

# --- Constants ---
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly LOG_FILE="/var/log/backup_configs.log"
readonly BACKUP_DIR="/backup/configs/$(date +%Y%m%d)"
readonly TIMESTAMP=$(date '+%Y-%m-%d %H:%M:%S')

# --- Functions ---
log() {
    echo "[${TIMESTAMP}] $1" | tee -a "$LOG_FILE"
}

error() {
    log "ERROR: $1" >&2
    exit 1
}

check_root() {
    [[ $EUID -eq 0 ]] || error "Must run as root"
}

backup_directory() {
    local src="$1"
    local desc="$2"
    if [[ -d "$src" ]]; then
        cp -rp "$src" "$BACKUP_DIR/" && log "Backed up $desc ($src)"
    else
        log "WARNING: $src not found, skipping $desc"
    fi
}

cleanup() {
    log "Cleaning up..."
    # Remove backups older than 30 days
    find /backup/configs -maxdepth 1 -type d -mtime +30 -exec rm -rf {} +
}

# --- Main ---
main() {
    check_root
    mkdir -p "$BACKUP_DIR"
    log "=== Backup started ==="

    backup_directory "/etc/nginx"      "nginx config"
    backup_directory "/etc/ssh"        "ssh config"
    backup_directory "/etc/pam.d"      "PAM config"
    backup_directory "/etc/sudoers.d"  "sudoers"

    cleanup
    log "=== Backup complete: $BACKUP_DIR ==="
}

trap 'error "Script interrupted on line $LINENO"' ERR
main "$@"
```

### Useful Script Patterns

```bash
# Argument parsing
while [[ $# -gt 0 ]]; do
    case "$1" in
        -h|--help) show_help; exit 0 ;;
        -v|--verbose) VERBOSE=true ;;
        -n|--name) NAME="$2"; shift ;;
        *) error "Unknown option: $1" ;;
    esac
    shift
done

# Input validation
[[ -z "${NAME:-}" ]] && error "Name required"
[[ "$NAME" =~ ^[a-zA-Z0-9_-]+$ ]] || error "Invalid name format"

# Retry logic
retry() {
    local n=0
    local max=3
    until [[ $n -ge $max ]]; do
        "$@" && return 0
        n=$((n+1))
        echo "Attempt $n/$max failed, retrying in 5s..."
        sleep 5
    done
    return 1
}
retry curl -sSf https://api.endpoint/health

# Locking (prevent concurrent runs)
LOCKFILE="/tmp/$(basename $0).lock"
exec 9>"$LOCKFILE"
flock -n 9 || error "Script already running"

# Temporary files
TMPFILE=$(mktemp)
trap "rm -f $TMPFILE" EXIT
```

---

## 9. Performance Monitoring & Tuning

### Key Performance Metrics

```bash
# CPU
top -b -n1 | head -5
vmstat 1 5                  # CPU, memory, swap, IO stats
mpstat -P ALL 1 3           # Per-core stats
sar -u 1 10                 # CPU utilization history

# Memory
free -h
cat /proc/meminfo
vmstat -s
slabtop                     # Kernel slab cache

# Disk I/O
iostat -xz 1 5              # Extended disk stats
iotop -o                    # Per-process I/O
dstat --disk-util           # Real-time disk utilization
df -hT                      # Filesystem usage
du -sh /var/* | sort -rh | head -10

# Network
sar -n DEV 1 5              # Network stats
nethogs eth0                # Per-process network usage
iftop -i eth0               # Bandwidth usage
ss -s                       # Socket statistics summary

# System load
uptime
w
cat /proc/loadavg
```

### Performance Tuning

```bash
# CPU governor (performance mode)
cpupower frequency-set -g performance
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor

# I/O scheduler
cat /sys/block/sda/queue/scheduler
echo "mq-deadline" > /sys/block/sda/queue/scheduler   # For SSDs
echo "bfq" > /sys/block/sda/queue/scheduler            # For HDDs

# TCP performance tuning
# /etc/sysctl.d/99-network-performance.conf
net.core.rmem_max = 134217728
net.core.wmem_max = 134217728
net.ipv4.tcp_rmem = 4096 87380 67108864
net.ipv4.tcp_wmem = 4096 65536 67108864
net.core.netdev_max_backlog = 5000
net.ipv4.tcp_max_syn_backlog = 8096

# Transparent Huge Pages
echo never > /sys/kernel/mm/transparent_hugepage/enabled   # For databases

# File descriptor limits
ulimit -n 65536             # Per-session
# /etc/security/limits.conf
*    soft    nofile    65536
*    hard    nofile    65536
```

---

## 10. Logging & Auditing

### journald & rsyslog

```bash
# journalctl
journalctl -u nginx                 # Service logs
journalctl -f                       # Follow all logs
journalctl -p err -b                # Errors since last boot
journalctl --since "2025-01-01" --until "2025-01-02"
journalctl -n 50 --no-pager
journalctl --disk-usage
journalctl --vacuum-size=1G         # Trim journal

# /etc/systemd/journald.conf
[Journal]
Storage=persistent
SystemMaxUse=2G
SystemKeepFree=1G
MaxRetentionSec=30day

# rsyslog
# /etc/rsyslog.d/50-defaults.conf
*.info;mail.none;authpriv.none;cron.none  /var/log/messages
authpriv.*                                /var/log/secure
mail.*                                    /var/log/maillog

# Forward to remote syslog
*.* @192.168.1.100:514      # UDP
*.* @@192.168.1.100:514     # TCP
```

### auditd (Linux Audit Framework)

```bash
# Install and start
apt install auditd / dnf install audit
systemctl enable --now auditd

# /etc/audit/rules.d/audit.rules
-D                                          # Delete all rules first
-b 8192                                     # Buffer size
-f 1                                        # Failure mode

# Monitor file access
-w /etc/passwd -p wa -k identity
-w /etc/shadow -p wa -k identity
-w /etc/sudoers -p wa -k sudoers
-w /etc/ssh/sshd_config -p wa -k sshd

# Monitor privileged commands
-a always,exit -F path=/usr/bin/sudo -F perm=x -F auid>=1000 -k privileged
-a always,exit -F path=/usr/bin/su -F perm=x -F auid>=1000 -k privileged

# Reload rules
augenrules --load

# Query audit log
ausearch -k identity -ts today
ausearch -ua jdoe -ts recent
aureport --auth                     # Authentication report
aureport --failed                   # Failed events
```

---

## 11. Backup & Recovery

### rsync Backup Strategies

```bash
# Full backup
rsync -avz --delete /source/ /destination/

# Incremental backup with hard links (time-machine style)
rsync -avz --link-dest=/backup/latest /source/ /backup/$(date +%Y%m%d)/
rm -f /backup/latest
ln -s /backup/$(date +%Y%m%d) /backup/latest

# Remote backup over SSH
rsync -avz -e "ssh -p 2222 -i /root/.ssh/backup_key" /etc/ backup@server:/backup/etc/

# Exclude patterns
rsync -avz --exclude='*.log' --exclude='tmp/' /source/ /dest/
rsync -avz --exclude-from='/etc/rsync-exclude.txt' /source/ /dest/
```

### System Backup with tar

```bash
# Full system backup (exclude non-essential)
tar -czpf /backup/system-$(date +%Y%m%d).tar.gz \
    --exclude=/proc \
    --exclude=/sys \
    --exclude=/dev \
    --exclude=/run \
    --exclude=/tmp \
    --exclude=/mnt \
    --exclude=/media \
    --exclude=/backup \
    /

# Incremental backup
find / -newer /backup/last_backup_timestamp -type f > /tmp/changed_files.txt
tar -czf /backup/incremental-$(date +%Y%m%d).tar.gz -T /tmp/changed_files.txt
touch /backup/last_backup_timestamp

# Restore
tar -xzpf backup.tar.gz -C /restore/point/
tar -xzf backup.tar.gz specific/file/path
```

### Disaster Recovery Testing

```bash
# Bare metal restore procedure
# 1. Boot from live media
# 2. Partition and format target disk
# 3. Mount target
mount /dev/sda1 /mnt/restore

# 4. Restore data
rsync -avz /backup/system/ /mnt/restore/

# 5. Restore bootloader
mount --bind /dev /mnt/restore/dev
mount --bind /proc /mnt/restore/proc
mount --bind /sys /mnt/restore/sys
chroot /mnt/restore
grub-install /dev/sda
update-grub
exit

# 6. Update /etc/fstab with new UUIDs if needed
blkid
vi /mnt/restore/etc/fstab
```

---

## 12. Troubleshooting Playbook

### Boot Issues

```bash
# Kernel boot messages
dmesg | grep -E "error|fail|warn" | head -30
dmesg -T | tail -50                 # With timestamps
journalctl -b -1 -p err            # Previous boot errors

# Check last shutdown reason
last -x | grep -E "shutdown|runlevel" | head
journalctl --list-boots
journalctl -b -1                    # Logs from last boot

# Recovery mode / single user
# At GRUB: press 'e', find linux line, add 'single' or 'init=/bin/bash'
# Or append: systemd.unit=rescue.target
```

### Disk Full Issues

```bash
# Find large files
du -sh /* 2>/dev/null | sort -rh | head -20
du -sh /var/* | sort -rh | head
find / -size +100M -type f 2>/dev/null | sort -k5 -rh

# Clean up common culprits
journalctl --vacuum-size=500M
apt clean / dnf clean all
find /tmp -mtime +7 -delete
find /var/log -name "*.gz" -mtime +30 -delete
truncate -s 0 /var/log/large.log  # Zero out large log (if in use)
```

### High CPU / Memory

```bash
# Find CPU hog
ps aux --sort=-%cpu | head -10
top (then press 'P' to sort by CPU, 'M' for memory)

# Strace hanging process
strace -p PID -e trace=all 2>&1 | head -50

# Memory leak investigation
cat /proc/PID/status | grep VmRSS
cat /proc/PID/smaps | awk '/^Rss/{rss+=$2} END{print rss/1024 "MB"}'

# OOM killer events
dmesg | grep -i "oom"
grep -i "oom" /var/log/kern.log
```

### Network Issues

```bash
# No connectivity debug flow
ip link show                   # 1. Interface up?
ip addr show eth0              # 2. IP assigned?
ping 127.0.0.1                 # 3. Loopback works?
ping <gateway>                 # 4. Default gateway reachable?
ping 8.8.8.8                   # 5. Internet reachable?
ping google.com                # 6. DNS works?
cat /etc/resolv.conf           # 7. DNS servers configured?
dig @8.8.8.8 google.com       # 8. External DNS works?
```

### Service Won't Start

```bash
# Detailed diagnosis
systemctl status service -l
journalctl -u service -n 50 --no-pager
journalctl -u service -f        # Follow in real-time

# Check config
nginx -t
apache2ctl configtest
sshd -t

# Dependency issues
systemctl list-dependencies service
systemd-analyze verify /etc/systemd/system/myapp.service

# Port conflict
ss -tulnp | grep :80
fuser 80/tcp
```

---

*This guide covers day-to-day Linux administration. For container operations, see the container management guide. For configuration management at scale, see the Ansible guide.*
