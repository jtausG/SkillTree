# Azure Administration Guide

## Overview
Enterprise Azure administration covering resource management, networking, identity, compute, storage, monitoring, and cost management. Designed for Azure administrators and cloud engineers.

---

## 1. Azure Fundamentals

### Management Hierarchy
```
Tenant (Azure AD)
└── Management Groups
    └── Subscriptions
        └── Resource Groups
            └── Resources
```

**Best Practice Structure:**
```
Root Management Group
├── Platform MG
│   ├── Connectivity Sub (hub networking)
│   ├── Identity Sub (AD DS, AAD Connect)
│   └── Management Sub (monitoring, security)
├── Landing Zones MG
│   ├── Corp MG
│   │   ├── Production Sub
│   │   └── Non-Production Sub
│   └── Online MG
└── Sandbox MG
    └── Dev/Test Sub
```

### Azure CLI & PowerShell Setup
```bash
# Install Azure CLI
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash   # Ubuntu/Debian
brew install azure-cli                                     # macOS

# Login
az login
az account list --output table
az account set --subscription "<subscription-name>"

# Current context
az account show
```

```powershell
# Install Az module
Install-Module -Name Az -AllowClobber -Force

# Connect
Connect-AzAccount
Get-AzSubscription
Set-AzContext -SubscriptionName "<subscription-name>"
```

---

## 2. Identity & Access Management

### Azure AD (Entra ID) Admin Tasks

**User Management:**
```powershell
# Create user
$params = @{
    displayName = "John Doe"
    userPrincipalName = "jdoe@tenant.onmicrosoft.com"
    accountEnabled = $true
    passwordProfile = @{
        forceChangePasswordNextSignIn = $true
        password = "TempP@ss123!"
    }
    usageLocation = "US"
}
New-MgUser -BodyParameter $params

# Guest user invitation
New-MgInvitation -InvitedUserEmailAddress "partner@external.com" `
    -InviteRedirectUrl "https://myapps.microsoft.com" `
    -SendInvitationMessage $true
```

**Groups:**
```powershell
# Create security group
New-MgGroup -DisplayName "IT Admins" `
    -MailNickname "ITAdmins" `
    -SecurityEnabled $true `
    -MailEnabled $false

# Dynamic group (auto-membership by attribute)
New-MgGroup -DisplayName "All FTE Users" `
    -MailNickname "AllFTE" `
    -SecurityEnabled $true `
    -MailEnabled $false `
    -GroupTypes @("DynamicMembership") `
    -MembershipRule "(user.userType -eq 'Member') and (user.jobTitle -ne null)" `
    -MembershipRuleProcessingState On
```

### Role-Based Access Control (RBAC)

**Built-in Roles (Common):**
| Role | Scope | Permissions |
|------|-------|-------------|
| Owner | Sub/RG/Resource | Full access + assign roles |
| Contributor | Sub/RG/Resource | Manage resources, no RBAC |
| Reader | Sub/RG/Resource | View only |
| User Access Administrator | Sub/RG/Resource | Manage access only |
| Virtual Machine Contributor | VM | Manage VMs |
| Storage Account Contributor | Storage | Manage storage |
| Network Contributor | Network | Manage networking |
| Key Vault Administrator | Key Vault | Manage secrets/certs/keys |

**Assign RBAC:**
```bash
# Assign role
az role assignment create \
    --assignee "user@domain.com" \
    --role "Contributor" \
    --scope "/subscriptions/<sub-id>/resourceGroups/<rg-name>"

# List assignments
az role assignment list --resource-group <rg-name> --output table

# Remove assignment
az role assignment delete \
    --assignee "user@domain.com" \
    --role "Contributor" \
    --scope "/subscriptions/<sub-id>/resourceGroups/<rg-name>"
```

**Custom RBAC Role:**
```json
{
    "Name": "VM Start/Stop Operator",
    "Description": "Can start and stop virtual machines",
    "Actions": [
        "Microsoft.Compute/virtualMachines/start/action",
        "Microsoft.Compute/virtualMachines/deallocate/action",
        "Microsoft.Compute/virtualMachines/restart/action",
        "Microsoft.Compute/virtualMachines/read"
    ],
    "NotActions": [],
    "DataActions": [],
    "NotDataActions": [],
    "AssignableScopes": ["/subscriptions/<sub-id>"]
}
```
```bash
az role definition create --role-definition vm-operator-role.json
```

### Privileged Identity Management (PIM)
- Provides just-in-time access to Azure/Entra ID roles
- Requires Azure AD Premium P2

**PIM Workflow:**
1. User requests role activation
2. Provides justification and duration
3. Approval required (if configured)
4. Role active for defined time window
5. Auto-deactivated at expiry
6. Audit log entry created

---

## 3. Networking

### Virtual Network (VNet) Design

**Address Space Planning:**
```
10.0.0.0/8 — Total enterprise range
├── 10.1.0.0/16 — East US Hub VNet
│   ├── 10.1.0.0/24 — GatewaySubnet
│   ├── 10.1.1.0/24 — AzureFirewallSubnet
│   ├── 10.1.2.0/24 — AzureBastionSubnet
│   └── 10.1.3.0/24 — ManagementSubnet
├── 10.2.0.0/16 — East US Production Spoke
│   ├── 10.2.0.0/24 — WebTier
│   ├── 10.2.1.0/24 — AppTier
│   └── 10.2.2.0/24 — DataTier
└── 10.3.0.0/16 — Non-Production
```

**Create VNet:**
```bash
az network vnet create \
    --resource-group rg-networking \
    --name vnet-prod-eastus \
    --address-prefix 10.2.0.0/16 \
    --location eastus

az network vnet subnet create \
    --resource-group rg-networking \
    --vnet-name vnet-prod-eastus \
    --name snet-web \
    --address-prefix 10.2.0.0/24
```

**VNet Peering:**
```bash
# Peer hub to spoke
az network vnet peering create \
    --resource-group rg-networking \
    --name hub-to-spoke-prod \
    --vnet-name vnet-hub-eastus \
    --remote-vnet vnet-prod-eastus \
    --allow-forwarded-traffic true \
    --allow-gateway-transit true

# Peer spoke to hub
az network vnet peering create \
    --resource-group rg-prod \
    --name spoke-prod-to-hub \
    --vnet-name vnet-prod-eastus \
    --remote-vnet vnet-hub-eastus \
    --use-remote-gateways true
```

### Network Security Groups (NSG)
```bash
# Create NSG
az network nsg create \
    --resource-group rg-networking \
    --name nsg-web-tier

# Add inbound rule - allow HTTPS from internet
az network nsg rule create \
    --resource-group rg-networking \
    --nsg-name nsg-web-tier \
    --name Allow-HTTPS-Inbound \
    --priority 100 \
    --protocol Tcp \
    --destination-port-ranges 443 \
    --source-address-prefixes Internet \
    --access Allow \
    --direction Inbound

# Associate NSG to subnet
az network vnet subnet update \
    --resource-group rg-networking \
    --vnet-name vnet-prod-eastus \
    --name snet-web \
    --network-security-group nsg-web-tier
```

### Azure Firewall
```bash
# Create Azure Firewall (requires AzureFirewallSubnet /26 minimum)
az network firewall create \
    --resource-group rg-networking \
    --name fw-hub-eastus \
    --location eastus \
    --tier Standard

# Create firewall policy
az network firewall policy create \
    --resource-group rg-networking \
    --name fwpol-hub-eastus

# Add application rule
az network firewall policy rule-collection-group create \
    --resource-group rg-networking \
    --policy-name fwpol-hub-eastus \
    --name AppRuleCollectionGroup \
    --priority 300
```

### VPN Gateway
```bash
# Create VPN Gateway (20-45 min deployment)
az network vnet-gateway create \
    --resource-group rg-networking \
    --name vpngw-hub-eastus \
    --vnet vnet-hub-eastus \
    --gateway-type Vpn \
    --vpn-type RouteBased \
    --sku VpnGw2 \
    --no-wait

# Create local network gateway (represents on-premises)
az network local-gateway create \
    --resource-group rg-networking \
    --name lng-onprem \
    --gateway-ip-address <your-public-ip> \
    --local-address-prefixes 192.168.0.0/16

# Create connection
az network vpn-connection create \
    --resource-group rg-networking \
    --name conn-onprem \
    --vnet-gateway1 vpngw-hub-eastus \
    --local-gateway2 lng-onprem \
    --shared-key "YourPreSharedKey123!"
```

---

## 4. Compute (Virtual Machines)

### VM Deployment

**Create VM:**
```bash
az vm create \
    --resource-group rg-prod \
    --name vm-web-01 \
    --image Win2022Datacenter \
    --size Standard_D2s_v3 \
    --vnet-name vnet-prod-eastus \
    --subnet snet-web \
    --admin-username azureadmin \
    --admin-password "SecureP@ssword123!" \
    --nsg nsg-web-tier \
    --public-ip-address "" \
    --no-wait
```

**VM Sizing Guide:**
| Series | Use Case | Example Sizes |
|--------|---------|---------------|
| B-series | Dev/test, low CPU burstable | B2s, B4ms |
| D-series | General purpose | D2s_v3, D4s_v3 |
| E-series | Memory optimized | E4s_v3, E8s_v3 |
| F-series | Compute optimized | F4s_v2, F8s_v2 |
| L-series | Storage optimized | L8s_v3, L16s_v3 |
| M-series | Memory intensive (SAP) | M32ts, M64ms |
| N-series | GPU (AI/rendering) | NC6s_v3, NV6 |

**Common VM Operations:**
```bash
# Start/Stop/Restart
az vm start --resource-group rg-prod --name vm-web-01
az vm stop --resource-group rg-prod --name vm-web-01
az vm deallocate --resource-group rg-prod --name vm-web-01  # Stops billing
az vm restart --resource-group rg-prod --name vm-web-01

# Resize VM
az vm resize --resource-group rg-prod --name vm-web-01 --size Standard_D4s_v3

# List VMs with status
az vm list --resource-group rg-prod --show-details --output table
```

**Run Commands on VM:**
```bash
# Run PowerShell script
az vm run-command invoke \
    --resource-group rg-prod \
    --name vm-web-01 \
    --command-id RunPowerShellScript \
    --scripts "Get-Process | Sort CPU -Descending | Select -First 10"

# Run bash on Linux VM
az vm run-command invoke \
    --resource-group rg-prod \
    --name vm-linux-01 \
    --command-id RunShellScript \
    --scripts "df -h && free -m"
```

### VM Extensions
```bash
# Install custom script extension
az vm extension set \
    --resource-group rg-prod \
    --vm-name vm-web-01 \
    --name CustomScriptExtension \
    --publisher Microsoft.Compute \
    --settings '{"fileUris":["https://storage.blob.core.windows.net/scripts/setup.ps1"],"commandToExecute":"powershell -ExecutionPolicy Unrestricted -File setup.ps1"}'

# Install Azure Monitor Agent
az vm extension set \
    --resource-group rg-prod \
    --vm-name vm-web-01 \
    --name AzureMonitorWindowsAgent \
    --publisher Microsoft.Azure.Monitor
```

### Azure Bastion (Secure RDP/SSH)
```bash
# Deploy Bastion (requires AzureBastionSubnet /26+)
az network bastion create \
    --resource-group rg-networking \
    --name bastion-hub-eastus \
    --vnet-name vnet-hub-eastus \
    --public-ip-address pip-bastion

# Connect via Bastion (portal or CLI)
az network bastion rdp --name bastion-hub-eastus \
    --resource-group rg-networking \
    --target-resource-id /subscriptions/<sub>/resourceGroups/rg-prod/providers/Microsoft.Compute/virtualMachines/vm-web-01
```

---

## 5. Storage

### Storage Account Management
```bash
# Create storage account
az storage account create \
    --resource-group rg-storage \
    --name stprodeastus001 \
    --location eastus \
    --sku Standard_GRS \
    --kind StorageV2 \
    --https-only true \
    --min-tls-version TLS1_2 \
    --allow-blob-public-access false

# Create blob container
az storage container create \
    --account-name stprodeastus001 \
    --name backups \
    --auth-mode login
```

**Storage Tiers:**
| Tier | Use Case | Access Pattern | Cost |
|------|---------|----------------|------|
| Premium | Low-latency, high IOPS | Frequent | High |
| Hot | Frequently accessed | Frequent | Medium |
| Cool | Infrequently accessed | ~30 days | Low storage, higher access |
| Archive | Rarely accessed | ~180 days | Lowest storage, high access |

**Lifecycle Management Policy:**
```json
{
    "rules": [
        {
            "name": "MoveToArchive",
            "type": "Lifecycle",
            "definition": {
                "actions": {
                    "baseBlob": {
                        "tierToCool": { "daysAfterModificationGreaterThan": 30 },
                        "tierToArchive": { "daysAfterModificationGreaterThan": 90 },
                        "delete": { "daysAfterModificationGreaterThan": 365 }
                    }
                },
                "filters": {
                    "blobTypes": ["blockBlob"],
                    "prefixMatch": ["backups/"]
                }
            }
        }
    ]
}
```

### Azure Files (SMB File Shares)
```bash
# Create file share
az storage share create \
    --account-name stprodeastus001 \
    --name dept-share \
    --quota 100

# Mount on Windows
# Net Use Z: \\stprodeastus001.file.core.windows.net\dept-share /user:AZURE\stprodeastus001 <storageKey>

# Mount on Linux
# mount -t cifs //stprodeastus001.file.core.windows.net/dept-share /mnt/dept-share -o vers=3.0,username=stprodeastus001,password=<key>,serverino
```

---

## 6. Monitoring & Alerts

### Azure Monitor

**Log Analytics Workspace:**
```bash
az monitor log-analytics workspace create \
    --resource-group rg-monitoring \
    --workspace-name law-prod-eastus \
    --location eastus \
    --sku PerGB2018 \
    --retention-time 90
```

**KQL Queries (Common):**
```kql
-- Top errors from VMs
Event
| where EventLevelName == "Error"
| summarize count() by Computer, EventLog, EventID
| sort by count_ desc
| take 20

-- CPU > 80% (via Metrics)
Perf
| where ObjectName == "Processor"
    and CounterName == "% Processor Time"
    and InstanceName == "_Total"
| where CounterValue > 80
| summarize AvgCPU = avg(CounterValue) by Computer, bin(TimeGenerated, 5m)
| sort by AvgCPU desc

-- Failed RDP logins
SecurityEvent
| where EventID == 4625
| where LogonType == 10  // Remote Interactive
| summarize FailedAttempts = count() by TargetAccount, IpAddress
| sort by FailedAttempts desc

-- Disk space below 10%
Perf
| where ObjectName == "LogicalDisk"
    and CounterName == "% Free Space"
    and InstanceName != "_Total"
    and InstanceName != "HarddiskVolume"
| where CounterValue < 10
| summarize FreePercent = avg(CounterValue) by Computer, InstanceName
```

**Create Alerts:**
```bash
# CPU alert
az monitor metrics alert create \
    --resource-group rg-monitoring \
    --name "High CPU - vm-web-01" \
    --scopes /subscriptions/<sub>/resourceGroups/rg-prod/providers/Microsoft.Compute/virtualMachines/vm-web-01 \
    --condition "avg Percentage CPU > 80" \
    --window-size 5m \
    --evaluation-frequency 1m \
    --action /subscriptions/<sub>/resourceGroups/rg-monitoring/providers/microsoft.insights/actionGroups/ag-ops-team \
    --severity 2
```

### Azure Security Center / Defender for Cloud
```bash
# View security score
az security secure-score-controls list --output table

# View recommendations
az security assessment list --output table

# View active alerts
az security alert list --output table
```

---

## 7. Cost Management

### Cost Analysis
```bash
# Current month spend
az consumption usage list \
    --start-date $(date -d "$(date +%Y-%m-01)" +%Y-%m-%d) \
    --end-date $(date +%Y-%m-%d) \
    --output table

# By resource group
az costmanagement query \
    --type ActualCost \
    --timeframe MonthToDate \
    --dataset-granularity Daily \
    --dataset-grouping type=Dimension name=ResourceGroup
```

### Budgets & Alerts
```bash
az consumption budget create \
    --budget-name "Monthly-Production-Budget" \
    --amount 10000 \
    --category Cost \
    --time-grain Monthly \
    --start-date "2025-01-01" \
    --end-date "2025-12-31" \
    --resource-group rg-prod
```

### Cost Optimization Checklist
- [ ] Right-size VMs (Azure Advisor recommendations)
- [ ] Reserved Instances for stable workloads (1 or 3 year = 40-72% savings)
- [ ] Azure Hybrid Benefit for Windows/SQL Server
- [ ] Spot VMs for non-critical batch workloads (up to 90% savings)
- [ ] Auto-shutdown dev/test VMs outside business hours
- [ ] Storage lifecycle management (tiering to cool/archive)
- [ ] Delete unattached managed disks and NICs
- [ ] Remove unused public IP addresses
- [ ] Consolidate underused storage accounts
- [ ] Review and terminate idle ExpressRoute/VPN circuits

---

## 8. Backup & Recovery

### Azure Backup

**Backup VM:**
```bash
# Create Recovery Services Vault
az backup vault create \
    --resource-group rg-backup \
    --name rsv-prod-eastus \
    --location eastus

# Enable backup for VM
az backup protection enable-for-vm \
    --resource-group rg-backup \
    --vault-name rsv-prod-eastus \
    --vm vm-web-01 \
    --policy-name DefaultPolicy

# Trigger on-demand backup
az backup protection backup-now \
    --resource-group rg-backup \
    --vault-name rsv-prod-eastus \
    --container-name vm-web-01 \
    --item-name vm-web-01 \
    --backup-management-type AzureIaasVM \
    --retain-until 2025-12-31
```

**Restore VM:**
```bash
# Get recovery points
az backup recoverypoint list \
    --resource-group rg-backup \
    --vault-name rsv-prod-eastus \
    --container-name vm-web-01 \
    --item-name vm-web-01 \
    --backup-management-type AzureIaasVM \
    --output table

# Restore to new VM
az backup restore restore-azurevm \
    --vault-name rsv-prod-eastus \
    --resource-group rg-backup \
    --rp-name <recovery-point-name> \
    --container-name vm-web-01 \
    --item-name vm-web-01 \
    --restore-to-staging-storage-account strestoretmp \
    --target-resource-group rg-restored \
    --new-vm-name vm-web-01-restored
```

---

## 9. Governance & Policy

### Azure Policy
```bash
# List built-in policies
az policy definition list --query "[?policyType=='BuiltIn'].{Name:displayName, ID:name}" --output table

# Assign policy - require tags
az policy assignment create \
    --name "require-environment-tag" \
    --policy "/providers/Microsoft.Authorization/policyDefinitions/96670d01-0a4d-4649-9c89-2d3abc0a5025" \
    --scope /subscriptions/<sub-id> \
    --params '{"tagName":{"value":"Environment"}}'
```

**Common Built-in Policies:**
- Require tag on resources
- Allowed locations
- Not allowed resource types
- Require HTTPS on storage
- Audit VMs without managed disks
- Deploy Log Analytics agent

### Resource Locks
```bash
# Apply ReadOnly lock (prevents modifications)
az lock create \
    --resource-group rg-prod \
    --name "Production-ReadOnly-Lock" \
    --lock-type ReadOnly

# Apply CanNotDelete lock
az lock create \
    --resource-group rg-networking \
    --name "Network-CanNotDelete-Lock" \
    --lock-type CanNotDelete

# List locks
az lock list --resource-group rg-prod --output table
```

### Tagging Strategy
```
Environment: Production | NonProduction | Development | Sandbox
Owner: team-name or email
CostCenter: department code
Application: application name
Criticality: Mission-Critical | High | Medium | Low
DataClassification: Public | Internal | Confidential | Restricted
```

```bash
# Tag resource group
az group update \
    --name rg-prod \
    --tags Environment=Production Owner=it-ops CostCenter=1234

# Tag all untagged VMs in subscription
az vm list --query "[?tags.Environment==null].{Name:name, RG:resourceGroup}" -o tsv | \
while IFS=$'\t' read -r name rg; do
    az vm update --name "$name" --resource-group "$rg" --set tags.Environment=Production
done
```

---

## 10. Automation

### Azure Automation Account
```bash
# Create Automation Account
az automation account create \
    --resource-group rg-automation \
    --name aa-prod-eastus \
    --location eastus \
    --sku Basic

# Import runbook
az automation runbook create \
    --resource-group rg-automation \
    --automation-account-name aa-prod-eastus \
    --name "StartVMs-Runbook" \
    --type PowerShell

# Schedule runbook
az automation schedule create \
    --resource-group rg-automation \
    --automation-account-name aa-prod-eastus \
    --name "Daily-7AM" \
    --frequency Day \
    --interval 1 \
    --start-time "2025-01-01T07:00:00"
```

### Infrastructure as Code (Bicep/ARM)

**Bicep Example — Storage Account:**
```bicep
param location string = resourceGroup().location
param storageAccountName string

resource storageAccount 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: storageAccountName
  location: location
  sku: {
    name: 'Standard_GRS'
  }
  kind: 'StorageV2'
  properties: {
    supportsHttpsTrafficOnly: true
    minimumTlsVersion: 'TLS1_2'
    allowBlobPublicAccess: false
  }
}

output storageAccountId string = storageAccount.id
```

```bash
# Deploy Bicep
az deployment group create \
    --resource-group rg-storage \
    --template-file storage.bicep \
    --parameters storageAccountName=stprodeastus001
```

---

*Last Updated: 2025 | IT Operations Documentation Library*
