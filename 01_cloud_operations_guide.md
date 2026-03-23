# Cloud Operations Guide — Azure & AWS
**Version:** 2.0 | **Audience:** Cloud Engineers, SysAdmins | **Last Updated:** 2025

---

## Table of Contents
1. [Cloud Strategy & Governance](#1-cloud-strategy--governance)
2. [Azure Operations](#2-azure-operations)
3. [AWS Operations](#3-aws-operations)
4. [Identity & Access Management](#4-identity--access-management)
5. [Networking in the Cloud](#5-networking-in-the-cloud)
6. [Compute Management](#6-compute-management)
7. [Storage Management](#7-storage-management)
8. [Cost Management & Optimization](#8-cost-management--optimization)
9. [Monitoring & Observability](#9-monitoring--observability)
10. [Security & Compliance](#10-security--compliance)
11. [Disaster Recovery & Business Continuity](#11-disaster-recovery--business-continuity)
12. [Automation & Infrastructure as Code](#12-automation--infrastructure-as-code)

---

## 1. Cloud Strategy & Governance

### Cloud Operating Models

| Model | Description | Best For |
|---|---|---|
| Cloud-first | All new workloads in cloud | Greenfield organizations |
| Hybrid | Mix of on-prem + cloud | Legacy + modern workloads |
| Multi-cloud | Azure + AWS + GCP | Avoid vendor lock-in |
| Cloud-native | Microservices, containers, serverless | Digital-native orgs |

### Well-Architected Pillars

**AWS Well-Architected / Azure Well-Architected:**
1. **Operational Excellence** — Run and monitor systems, continually improve
2. **Security** — Protect data, systems, and assets
3. **Reliability** — Recover from failures, meet availability targets
4. **Performance Efficiency** — Use computing resources efficiently
5. **Cost Optimization** — Avoid unnecessary costs
6. **Sustainability** — Minimize environmental impact

### Cloud Governance Framework

```
Landing Zone Structure:
├── Management Groups
│   ├── Platform
│   │   ├── Identity Subscription
│   │   ├── Connectivity Subscription
│   │   └── Management Subscription
│   └── Workloads
│       ├── Production
│       ├── Non-Production
│       └── Sandbox
└── Policies
    ├── Allowed Regions
    ├── Required Tags (Environment, Owner, CostCenter)
    ├── Deny Public IPs on VMs
    └── Require Encryption at Rest
```

### Tagging Strategy

```
Required Tags:
- Environment: prod | staging | dev | sandbox
- Application: name-of-app
- Owner: team-email@company.com
- CostCenter: 12345
- DataClassification: public | internal | confidential | restricted
- Backup: daily | weekly | none
```

---

## 2. Azure Operations

### Azure CLI Essentials

```bash
# Install
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# Authentication
az login
az login --use-device-code
az login --service-principal -u APP_ID -p PASSWORD --tenant TENANT_ID

# Context/Subscription
az account list --output table
az account set --subscription "Subscription Name"
az account show

# Resource Groups
az group create --name rg-prod-eastus --location eastus
az group list --output table
az group delete --name rg-old --yes --no-wait

# Resources
az resource list --resource-group rg-prod-eastus --output table
az resource show --ids /subscriptions/.../resourceGroups/rg/providers/.../resource
```

### Azure Virtual Machines

```bash
# Create VM
az vm create \
  --resource-group rg-prod-eastus \
  --name vm-web-01 \
  --image Ubuntu2204 \
  --size Standard_D2s_v3 \
  --admin-username azureadmin \
  --ssh-key-values ~/.ssh/id_ed25519.pub \
  --vnet-name vnet-prod \
  --subnet snet-web \
  --public-ip-address "" \
  --nsg "" \
  --os-disk-size-gb 64 \
  --storage-sku Premium_LRS \
  --tags Environment=prod Application=web Owner=ops@company.com

# VM operations
az vm start/stop/deallocate/restart --name vm-web-01 --resource-group rg-prod-eastus
az vm list --resource-group rg-prod-eastus --show-details --output table

# Resize VM
az vm resize --resource-group rg-prod-eastus --name vm-web-01 --size Standard_D4s_v3

# Disk management
az disk create --name datadisk01 --resource-group rg-prod-eastus --size-gb 128 --sku Premium_LRS
az vm disk attach --vm-name vm-web-01 --resource-group rg-prod-eastus --name datadisk01

# Capture VM image
az vm deallocate --resource-group rg-prod-eastus --name vm-web-01
az vm generalize --resource-group rg-prod-eastus --name vm-web-01
az image create --resource-group rg-images --name img-web-2025 --source vm-web-01 --resource-group rg-prod-eastus
```

### Azure Networking

```bash
# Virtual Network
az network vnet create \
  --resource-group rg-prod-eastus \
  --name vnet-prod-eastus \
  --address-prefix 10.0.0.0/16 \
  --location eastus

# Subnets
az network vnet subnet create \
  --resource-group rg-prod-eastus \
  --vnet-name vnet-prod-eastus \
  --name snet-web \
  --address-prefix 10.0.1.0/24

# NSG
az network nsg create --resource-group rg-prod-eastus --name nsg-web
az network nsg rule create \
  --resource-group rg-prod-eastus \
  --nsg-name nsg-web \
  --name allow-https \
  --priority 100 \
  --direction Inbound \
  --protocol Tcp \
  --destination-port-ranges 443 \
  --access Allow

# VNet Peering
az network vnet peering create \
  --name peer-prod-to-hub \
  --resource-group rg-prod-eastus \
  --vnet-name vnet-prod-eastus \
  --remote-vnet vnet-hub-eastus \
  --allow-vnet-access \
  --allow-forwarded-traffic

# Application Gateway
az network application-gateway create \
  --name agw-prod \
  --resource-group rg-prod-eastus \
  --sku WAF_v2 \
  --capacity 2 \
  --vnet-name vnet-prod-eastus \
  --subnet snet-agw \
  --public-ip-address pip-agw-prod
```

### Azure Storage

```bash
# Storage Account
az storage account create \
  --name stprodeastus001 \
  --resource-group rg-prod-eastus \
  --location eastus \
  --sku Standard_GRS \
  --kind StorageV2 \
  --access-tier Hot \
  --https-only true \
  --min-tls-version TLS1_2 \
  --allow-blob-public-access false

# Blob operations
az storage container create --name backups --account-name stprodeastus001
az storage blob upload --file /backup/data.tar.gz --container-name backups --name data.tar.gz --account-name stprodeastus001
az storage blob list --container-name backups --account-name stprodeastus001 --output table

# SAS token
az storage blob generate-sas \
  --account-name stprodeastus001 \
  --container-name backups \
  --name data.tar.gz \
  --permissions r \
  --expiry 2025-12-31T23:59:59Z \
  --https-only
```

### Azure Key Vault

```bash
# Create
az keyvault create \
  --name kv-prod-eastus-001 \
  --resource-group rg-prod-eastus \
  --location eastus \
  --enable-purge-protection \
  --enable-soft-delete \
  --retention-days 90

# Secrets
az keyvault secret set --vault-name kv-prod-eastus-001 --name "db-password" --value "SecurePass123!"
az keyvault secret show --vault-name kv-prod-eastus-001 --name "db-password" --query value -o tsv
az keyvault secret list --vault-name kv-prod-eastus-001 --output table

# Access policies (legacy)
az keyvault set-policy --name kv-prod-eastus-001 --upn user@domain.com --secret-permissions get list

# RBAC (preferred)
az role assignment create \
  --role "Key Vault Secrets User" \
  --assignee user@domain.com \
  --scope /subscriptions/.../resourceGroups/rg-prod/providers/Microsoft.KeyVault/vaults/kv-prod
```

---

## 3. AWS Operations

### AWS CLI Essentials

```bash
# Install
pip install awscli --break-system-packages
# or: curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

# Configure
aws configure                    # Interactive setup
aws configure --profile prod     # Named profile
# Stored in ~/.aws/credentials and ~/.aws/config

# Switch profile
export AWS_PROFILE=prod
aws sts get-caller-identity      # Verify current identity

# Basic queries
aws ec2 describe-instances --output table
aws ec2 describe-instances --filters "Name=tag:Environment,Values=prod" --query 'Reservations[*].Instances[*].[InstanceId,InstanceType,State.Name,Tags[?Key==`Name`].Value|[0]]' --output table
```

### AWS EC2 Management

```bash
# Launch instance
aws ec2 run-instances \
  --image-id ami-0abcdef1234567890 \
  --instance-type t3.medium \
  --key-name mykey \
  --security-group-ids sg-12345678 \
  --subnet-id subnet-12345678 \
  --iam-instance-profile Name=EC2InstanceRole \
  --block-device-mappings '[{"DeviceName":"/dev/xvda","Ebs":{"VolumeSize":50,"VolumeType":"gp3","Encrypted":true,"DeleteOnTermination":true}}]' \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=web-01},{Key=Environment,Value=prod}]' \
  --user-data file://userdata.sh

# Instance operations
aws ec2 start-instances --instance-ids i-1234567890abcdef0
aws ec2 stop-instances --instance-ids i-1234567890abcdef0
aws ec2 terminate-instances --instance-ids i-1234567890abcdef0
aws ec2 reboot-instances --instance-ids i-1234567890abcdef0

# AMI creation
aws ec2 create-image \
  --instance-id i-1234567890abcdef0 \
  --name "web-server-$(date +%Y%m%d)" \
  --no-reboot \
  --tag-specifications 'ResourceType=image,Tags=[{Key=Name,Value=web-server-backup}]'

# SSM Session (no SSH needed)
aws ssm start-session --target i-1234567890abcdef0
```

### AWS VPC Networking

```bash
# Create VPC
aws ec2 create-vpc --cidr-block 10.0.0.0/16 --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=vpc-prod}]'

# Subnets (multi-AZ)
aws ec2 create-subnet --vpc-id vpc-12345 --cidr-block 10.0.1.0/24 --availability-zone us-east-1a
aws ec2 create-subnet --vpc-id vpc-12345 --cidr-block 10.0.2.0/24 --availability-zone us-east-1b

# Internet Gateway
aws ec2 create-internet-gateway
aws ec2 attach-internet-gateway --internet-gateway-id igw-12345 --vpc-id vpc-12345

# Route Table
aws ec2 create-route-table --vpc-id vpc-12345
aws ec2 create-route --route-table-id rtb-12345 --destination-cidr-block 0.0.0.0/0 --gateway-id igw-12345
aws ec2 associate-route-table --route-table-id rtb-12345 --subnet-id subnet-12345

# Security Group
aws ec2 create-security-group --group-name sg-web --description "Web servers" --vpc-id vpc-12345
aws ec2 authorize-security-group-ingress --group-id sg-12345 --protocol tcp --port 443 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-id sg-12345 --protocol tcp --port 22 --source-group sg-bastion
```

### AWS S3

```bash
# Bucket management
aws s3 mb s3://company-backups-prod-2025
aws s3 rb s3://old-bucket --force

# Sync and copy
aws s3 sync /local/path s3://company-backups-prod-2025/backups/
aws s3 sync s3://source-bucket s3://dest-bucket
aws s3 cp file.tar.gz s3://company-backups-prod-2025/
aws s3 cp s3://bucket/file.tar.gz /local/path/

# Object operations
aws s3 ls s3://company-backups-prod-2025/ --recursive --human-readable
aws s3 rm s3://bucket/path --recursive

# Bucket policy
aws s3api put-bucket-policy --bucket company-backups-prod-2025 --policy file://bucket-policy.json

# Lifecycle rules
aws s3api put-bucket-lifecycle-configuration \
  --bucket company-backups-prod-2025 \
  --lifecycle-configuration file://lifecycle.json

# Presigned URL
aws s3 presign s3://bucket/file.tar.gz --expires-in 3600
```

### AWS IAM

```bash
# User management
aws iam create-user --user-name jdoe
aws iam create-login-profile --user-name jdoe --password TempPass123! --password-reset-required
aws iam create-access-key --user-name jdoe
aws iam attach-user-policy --user-name jdoe --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess

# Groups
aws iam create-group --group-name Developers
aws iam add-user-to-group --group-name Developers --user-name jdoe
aws iam attach-group-policy --group-name Developers --policy-arn arn:aws:iam::aws:policy/PowerUserAccess

# Roles
aws iam create-role --role-name EC2S3Access --assume-role-policy-document file://trust-policy.json
aws iam attach-role-policy --role-name EC2S3Access --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# List and audit
aws iam list-users --output table
aws iam get-account-summary
aws iam generate-credential-report
aws iam get-credential-report --query Content --output text | base64 -d
```

---

## 4. Identity & Access Management

### Azure AD / Entra ID

```bash
# Using Microsoft Graph PowerShell
Connect-MgGraph -Scopes "User.ReadWrite.All","Group.ReadWrite.All","Directory.ReadWrite.All"

# Users
New-MgUser -DisplayName "John Doe" -UserPrincipalName "jdoe@company.com" -PasswordProfile @{Password="TempPass123!"; ForceChangePasswordNextSignIn=$true} -AccountEnabled

Get-MgUser -Filter "UserType eq 'Member'" | Select DisplayName, UPN, AccountEnabled

# Groups
New-MgGroup -DisplayName "IT Operations" -MailNickname "it-operations" -SecurityEnabled -MailEnabled:$false
Add-MgGroupMember -GroupId "group-object-id" -DirectoryObjectId "user-object-id"

# App Registrations
New-MgApplication -DisplayName "MyApp" -SignInAudience AzureADMyOrg
```

### Conditional Access Policies (Azure)

```
Key Policies to Implement:
1. Require MFA for all users
   - Users: All
   - Cloud apps: All
   - Conditions: Any location
   - Grant: Require MFA

2. Block legacy authentication
   - Users: All
   - Cloud apps: All
   - Conditions: Client apps = legacy protocols
   - Grant: Block

3. Require compliant device
   - Users: All
   - Cloud apps: Microsoft 365
   - Grant: Require device compliance OR hybrid Azure AD join

4. Privileged access - require FIDO2
   - Users: Global Admins, Privileged Role Admins
   - Cloud apps: All
   - Grant: Require FIDO2 security key

5. High-risk sign-in → MFA + password change
   - Users: All
   - Risk level: High
   - Grant: Require MFA + require password change
```

---

## 5. Networking in the Cloud

### Azure Hub-Spoke Topology

```
Hub VNet (10.0.0.0/16)
├── GatewaySubnet (10.0.0.0/27) - VPN/ExpressRoute Gateway
├── AzureFirewallSubnet (10.0.1.0/26) - Azure Firewall
├── AzureBastionSubnet (10.0.2.0/27) - Azure Bastion
└── ManagementSubnet (10.0.3.0/24) - Jump boxes, monitoring

Spoke VNets (peered to Hub)
├── vnet-prod-app (10.1.0.0/16)
│   ├── snet-web (10.1.1.0/24)
│   ├── snet-app (10.1.2.0/24)
│   └── snet-data (10.1.3.0/24)
└── vnet-nonprod (10.2.0.0/16)
    ├── snet-staging (10.2.1.0/24)
    └── snet-dev (10.2.2.0/24)
```

### AWS VPC Design

```
VPC: 10.0.0.0/16
├── Public Subnets (one per AZ)
│   ├── us-east-1a: 10.0.0.0/24 - Load balancers, NAT GW
│   ├── us-east-1b: 10.0.1.0/24
│   └── us-east-1c: 10.0.2.0/24
├── Private App Subnets (one per AZ)
│   ├── us-east-1a: 10.0.10.0/24 - App servers
│   ├── us-east-1b: 10.0.11.0/24
│   └── us-east-1c: 10.0.12.0/24
└── Private Data Subnets (one per AZ)
    ├── us-east-1a: 10.0.20.0/24 - Databases
    ├── us-east-1b: 10.0.21.0/24
    └── us-east-1c: 10.0.22.0/24
```

---

## 6. Compute Management

### VM Sizing Guidelines

| Workload | Azure | AWS |
|---|---|---|
| Dev/Test | B2s (2 vCPU, 4GB) | t3.medium |
| Web/App server | D4s_v3 (4 vCPU, 16GB) | m5.xlarge |
| Database | E8s_v3 (8 vCPU, 64GB) | r5.2xlarge |
| High CPU | F8s_v2 (8 vCPU, 16GB) | c5.2xlarge |
| GPU | NC6s_v3 | p3.2xlarge |

### Auto-Scaling

```bash
# Azure VMSS
az vmss create \
  --resource-group rg-prod \
  --name vmss-web \
  --image Ubuntu2204 \
  --instance-count 2 \
  --vm-sku Standard_D2s_v3 \
  --upgrade-policy-mode automatic

az monitor autoscale create \
  --resource-group rg-prod \
  --resource vmss-web \
  --resource-type Microsoft.Compute/virtualMachineScaleSets \
  --name autoscale-web \
  --min-count 2 --max-count 10 --count 2

az monitor autoscale rule create \
  --resource-group rg-prod \
  --autoscale-name autoscale-web \
  --scale out 2 \
  --condition "Percentage CPU > 75 avg 5m"

# AWS Auto Scaling Group
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name asg-web \
  --launch-template LaunchTemplateName=lt-web,Version='$Latest' \
  --min-size 2 \
  --max-size 10 \
  --desired-capacity 2 \
  --vpc-zone-identifier "subnet-1,subnet-2,subnet-3" \
  --target-group-arns arn:aws:elasticloadbalancing:...

aws autoscaling put-scaling-policy \
  --auto-scaling-group-name asg-web \
  --policy-name target-tracking \
  --policy-type TargetTrackingScaling \
  --target-tracking-configuration file://target-tracking.json
```

---

## 7. Storage Management

### Azure Storage Tiers & Types

| Type | Use Case | Redundancy Options |
|---|---|---|
| Blob (Hot) | Frequently accessed data | LRS, ZRS, GRS, GZRS |
| Blob (Cool) | Infrequently accessed, 30+ days | LRS, ZRS, GRS |
| Blob (Cold) | Rarely accessed, 90+ days | LRS, ZRS, GRS |
| Blob (Archive) | Rarely accessed, 180+ days | LRS, GRS |
| Azure Files | SMB/NFS file shares | LRS, ZRS, GRS |
| Azure Disks | VM block storage | LRS, ZRS |
| Azure NetApp | High performance NFS | LRS |

### AWS Storage Types

| Service | Type | Use Case |
|---|---|---|
| S3 Standard | Object | General purpose |
| S3-IA | Object | Infrequent access |
| S3 Glacier | Object | Archive |
| EBS gp3 | Block | Boot volumes, low-latency |
| EBS io2 | Block | High IOPS databases |
| EFS | File (NFS) | Shared file systems |
| FSx for Windows | File (SMB) | Windows workloads |

---

## 8. Cost Management & Optimization

### Azure Cost Management

```bash
# Export costs via CLI
az consumption usage list --start-date 2025-01-01 --end-date 2025-01-31 --output table

# Set budget alert
az consumption budget create \
  --budget-name monthly-budget-prod \
  --amount 5000 \
  --category Cost \
  --time-grain Monthly \
  --start-date 2025-01-01 \
  --end-date 2025-12-31 \
  --resource-group rg-prod \
  --notifications '[{"enabled":true,"operator":"GreaterThan","threshold":80,"contactEmails":["ops@company.com"]}]'
```

### Cost Optimization Strategies

```
Reserved Instances / Savings Plans:
- Azure Reserved VMs: 1 or 3 year commitment, 40-72% savings
- AWS Reserved Instances: 1 or 3 year, up to 75% savings
- AWS Savings Plans: Flexible usage, up to 66% savings

Right-Sizing:
- Review CPU/memory utilization (target: 40-70% average)
- Azure: Advisor recommendations
- AWS: Compute Optimizer
- Downsize overprovisioned instances
- Upgrade undersized instances (reliability risk)

Scheduling:
- Dev/Test: shut down nights and weekends (save ~65%)
- Azure: Auto-shutdown on VMs
- AWS: Lambda + EventBridge to stop/start

Storage Optimization:
- Enable lifecycle policies to move data to cheaper tiers
- Delete unattached disks
- Remove old snapshots (>90 days)
- S3 Intelligent-Tiering for uncertain access patterns

Network:
- Use VPN vs. ExpressRoute/Direct Connect based on bandwidth needs
- Minimize data transfer between regions
- Use CDN for static content delivery
```

### Cost Dashboard KPIs

| Metric | Target | Alert Threshold |
|---|---|---|
| Monthly spend vs. budget | < 100% | > 80% |
| Cost per unit (per user, per app) | Baseline | +20% MoM |
| Reserved instance coverage | > 70% | < 50% |
| Idle resource count | 0 | > 5 resources |
| Untagged resources | 0% | > 5% |

---

## 9. Monitoring & Observability

### Azure Monitor Stack

```bash
# Log Analytics workspace
az monitor log-analytics workspace create \
  --resource-group rg-management \
  --workspace-name law-prod-eastus \
  --location eastus \
  --retention-time 90

# Enable diagnostics on VM
az monitor diagnostic-settings create \
  --name vm-diagnostics \
  --resource /subscriptions/.../virtualMachines/vm-web-01 \
  --workspace /subscriptions/.../workspaces/law-prod-eastus \
  --metrics '[{"category":"AllMetrics","enabled":true}]' \
  --logs '[{"category":"Administrative","enabled":true}]'

# Alert rule
az monitor metrics alert create \
  --name "High CPU Alert" \
  --resource-group rg-prod \
  --scopes /subscriptions/.../virtualMachines/vm-web-01 \
  --condition "avg Percentage CPU > 90" \
  --window-size 5m \
  --evaluation-frequency 1m \
  --action /subscriptions/.../actionGroups/ag-ops-email \
  --severity 2
```

### AWS CloudWatch

```bash
# Create alarm
aws cloudwatch put-metric-alarm \
  --alarm-name "HighCPU-web-01" \
  --alarm-description "CPU > 90% for 5 minutes" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 90 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=InstanceId,Value=i-1234567890abcdef0 \
  --evaluation-periods 1 \
  --alarm-actions arn:aws:sns:us-east-1:123456789:ops-alerts

# Query CloudWatch Logs Insights
aws logs start-query \
  --log-group-name /aws/ec2/web-servers \
  --start-time $(date -d '1 hour ago' +%s) \
  --end-time $(date +%s) \
  --query-string 'fields @timestamp, @message | filter @message like /ERROR/ | sort @timestamp desc | limit 50'
```

### Key Metrics to Monitor

```
Infrastructure:
- CPU utilization (alert >85%)
- Memory usage (alert >90%)
- Disk I/O (alert if saturated >95%)
- Network in/out (alert if near bandwidth limit)
- Disk capacity (alert >80%)

Application:
- Response time (p50, p95, p99)
- Error rate (4xx, 5xx)
- Request throughput (RPS)
- Queue depth / backlog

Database:
- Query latency
- Connection pool usage
- Replication lag
- Index hit ratio

Security:
- Failed login attempts (alert >10/min)
- IAM policy changes
- Security group changes
- Unusual data egress
```

---

## 10. Security & Compliance

### Cloud Security Baseline

```
Azure Security Baseline:
✅ Enable Microsoft Defender for Cloud (all plans)
✅ Enable Azure Security Center recommendations
✅ Enable Azure AD Identity Protection
✅ Enable Microsoft Sentinel (SIEM)
✅ Enable Defender for Servers P2 on all VMs
✅ All storage encrypted with CMK
✅ Key Vault for all secrets
✅ Private Endpoints for PaaS services
✅ Azure Policy for compliance enforcement
✅ Just-In-Time VM access enabled
✅ Adaptive application controls

AWS Security Baseline:
✅ Enable AWS GuardDuty (all regions)
✅ Enable AWS Security Hub
✅ Enable AWS Config rules
✅ Enable CloudTrail (all regions, S3, management + data events)
✅ Enable VPC Flow Logs
✅ Enable AWS Macie for S3
✅ AWS Inspector for EC2 vulnerability scanning
✅ All EBS volumes encrypted
✅ S3 Block Public Access (account level)
✅ IMDSv2 enforced on all EC2
✅ Root account MFA enabled
✅ No root account access keys
```

### AWS Security Audit

```bash
# Check for public S3 buckets
aws s3api list-buckets --query 'Buckets[*].Name' --output text | tr '\t' '\n' | while read bucket; do
    policy=$(aws s3api get-bucket-policy-status --bucket $bucket 2>/dev/null)
    if echo $policy | grep -q '"IsPublic": true'; then
        echo "PUBLIC BUCKET: $bucket"
    fi
done

# Find EC2 with public IPs
aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running" \
  --query 'Reservations[*].Instances[?PublicIpAddress!=null].[InstanceId,PublicIpAddress,Tags[?Key==`Name`].Value|[0]]' \
  --output table

# IAM users with console + access keys
aws iam list-users --query 'Users[*].UserName' --output text | tr '\t' '\n' | while read user; do
    keys=$(aws iam list-access-keys --user-name $user --query 'AccessKeyMetadata[?Status==`Active`].AccessKeyId' --output text)
    profile=$(aws iam get-login-profile --user-name $user 2>/dev/null)
    if [[ -n "$keys" && -n "$profile" ]]; then
        echo "USER WITH BOTH: $user (Keys: $keys)"
    fi
done
```

---

## 11. Disaster Recovery & Business Continuity

### RPO/RTO Targets by Tier

| Tier | Description | RTO | RPO |
|---|---|---|---|
| Tier 0 | Mission critical (payments, auth) | < 15 min | < 5 min |
| Tier 1 | Business critical (core apps) | < 1 hour | < 1 hour |
| Tier 2 | Important (reporting, analytics) | < 4 hours | < 4 hours |
| Tier 3 | Standard (dev, test) | < 24 hours | < 24 hours |

### Azure Site Recovery

```bash
# Enable replication for Azure VM
az site-recovery protected-item create \
  --resource-group rg-asr \
  --vault-name rsv-asr-prod \
  --fabric-name azure-eastus \
  --protection-container default-container \
  --name vm-web-01-replication \
  --policy defaultPolicyName \
  --vm-id /subscriptions/.../virtualMachines/vm-web-01

# Test failover
az site-recovery replication-protected-item planned-failover \
  --resource-group rg-asr \
  --vault-name rsv-asr-prod \
  --fabric-name azure-westus \
  --protection-container default \
  --name vm-web-01-replication
```

### Multi-Region DR Architecture

```
Primary Region (East US):                Secondary Region (West US):
├── App VMs (Active)                     ├── App VMs (Standby/Replicated)
├── Azure SQL (Primary)       ──────►    ├── Azure SQL (Geo-Replica, readable)
├── Storage (GRS Primary)     ──────►    ├── Storage (GRS Secondary)
├── Traffic Manager / Front Door         └── Traffic Manager endpoint (priority 2)
│   └── Priority 1 endpoint
└── Recovery Services Vault  ──────►    Azure Site Recovery replication
```

---

## 12. Automation & Infrastructure as Code

### Terraform for Azure

```hcl
# main.tf
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
  backend "azurerm" {
    resource_group_name  = "rg-terraform-state"
    storage_account_name = "sttfstateeastus001"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
  }
}

provider "azurerm" {
  features {}
  subscription_id = var.subscription_id
}

resource "azurerm_resource_group" "main" {
  name     = "rg-${var.environment}-${var.location}"
  location = var.location
  tags     = local.common_tags
}

resource "azurerm_virtual_network" "main" {
  name                = "vnet-${var.environment}-${var.location}"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  address_space       = [var.vnet_cidr]
  tags                = local.common_tags
}

locals {
  common_tags = {
    Environment = var.environment
    ManagedBy   = "Terraform"
    Owner       = "ops@company.com"
  }
}
```

```bash
# Terraform workflow
terraform init          # Initialize
terraform fmt           # Format code
terraform validate      # Validate syntax
terraform plan -out=tfplan.out    # Preview changes
terraform apply tfplan.out         # Apply changes
terraform destroy -target=azurerm_resource_group.old  # Targeted destroy
terraform state list    # List state
terraform import azurerm_resource_group.main /subscriptions/.../resourceGroups/rg-existing
```

### Terraform for AWS

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  backend "s3" {
    bucket         = "company-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-lock"
  }
}

provider "aws" {
  region  = var.aws_region
  profile = var.aws_profile
  default_tags {
    tags = {
      Environment = var.environment
      ManagedBy   = "Terraform"
    }
  }
}

module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"
  name    = "vpc-${var.environment}"
  cidr    = var.vpc_cidr
  azs     = var.availability_zones
  private_subnets = var.private_subnet_cidrs
  public_subnets  = var.public_subnet_cidrs
  enable_nat_gateway = true
  single_nat_gateway = var.environment != "prod"
}
```

### GitHub Actions for Cloud Deployments

```yaml
# .github/workflows/deploy-infra.yml
name: Deploy Infrastructure

on:
  push:
    branches: [main]
    paths: ['terraform/**']
  pull_request:
    branches: [main]
    paths: ['terraform/**']

jobs:
  terraform:
    runs-on: ubuntu-latest
    environment: production
    permissions:
      contents: read
      id-token: write  # For OIDC auth

    steps:
      - uses: actions/checkout@v4

      - name: Azure Login (OIDC)
        uses: azure/login@v1
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

      - name: Terraform Setup
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: 1.7.0

      - name: Terraform Init
        run: terraform init
        working-directory: terraform/

      - name: Terraform Plan
        run: terraform plan -out=tfplan.out
        working-directory: terraform/

      - name: Terraform Apply
        if: github.ref == 'refs/heads/main'
        run: terraform apply tfplan.out
        working-directory: terraform/
```

---

*For container-specific cloud operations (AKS, EKS), see the Kubernetes Operations Guide. For CI/CD pipelines, see the DevOps Pipelines Guide.*
