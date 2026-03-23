# IT Operations Knowledge Base — Master Index

> **Version:** 1.0 | **Last Updated:** 2024 | **Maintained by:** IT Operations

---

## 📁 Document Library

### 🔧 Troubleshooting
| File | Description |
|------|-------------|
| [General Troubleshooting Methodology](troubleshooting/01-general-troubleshooting-methodology.md) | OSI model approach, ticket triage, escalation matrix |
| [Windows Workstation Troubleshooting](troubleshooting/02-windows-workstation-troubleshooting.md) | Boot issues, BSOD, performance, drivers |
| [macOS Troubleshooting](troubleshooting/03-macos-troubleshooting.md) | Kernel panics, permission issues, MDM, profiles |
| [Network Connectivity Troubleshooting](troubleshooting/04-network-connectivity-troubleshooting.md) | DNS, DHCP, TCP/IP, VPN, Wi-Fi |
| [Application Troubleshooting](troubleshooting/05-application-troubleshooting.md) | Crashes, slow apps, Office 365, licensing |
| [Printer & Peripheral Troubleshooting](troubleshooting/06-printer-peripheral-troubleshooting.md) | Drivers, spooler, USB, display issues |
| [Email & Collaboration Troubleshooting](troubleshooting/07-email-collaboration-troubleshooting.md) | Exchange, Outlook, Teams, calendar |
| [VPN & Remote Access Troubleshooting](troubleshooting/08-vpn-remote-access-troubleshooting.md) | Split tunnel, certificates, MFA failures |
| [Server Troubleshooting](troubleshooting/09-server-troubleshooting.md) | Disk, memory, CPU, event logs, services |
| [Cloud Services Troubleshooting](troubleshooting/10-cloud-services-troubleshooting.md) | Azure, AWS, M365, SSO failures |

### 🌐 Networking
| File | Description |
|------|-------------|
| [Network Fundamentals Reference](networking/01-network-fundamentals-reference.md) | OSI model, protocols, subnetting |
| [IP Addressing & Subnetting Guide](networking/02-ip-addressing-subnetting.md) | CIDR, VLSM, supernetting, IPv6 |
| [DNS Administration Guide](networking/03-dns-administration.md) | Record types, zones, troubleshooting, DNSSEC |
| [DHCP Administration Guide](networking/04-dhcp-administration.md) | Scopes, reservations, relay agents, failover |
| [Firewall Administration](networking/05-firewall-administration.md) | Rules, NAT, zones, logging, review process |
| [VPN Configuration & Management](networking/06-vpn-configuration-management.md) | IPSec, SSL, split tunnel, site-to-site |
| [Switching & VLANs](networking/07-switching-vlans.md) | STP, trunking, access ports, QoS |
| [Wireless Network Administration](networking/08-wireless-network-administration.md) | 802.11, WPA3, controller management, rogue AP |
| [Network Monitoring & Performance](networking/09-network-monitoring-performance.md) | SNMP, NetFlow, baselines, alerting |
| [Network Diagrams & Documentation Standards](networking/10-network-documentation-standards.md) | Conventions, tools, diagram types |

### 🏢 Active Directory
| File | Description |
|------|-------------|
| [AD Administration Fundamentals](active-directory/01-ad-administration-fundamentals.md) | Objects, schema, sites, replication |
| [User Account Management](active-directory/02-user-account-management.md) | Provisioning, deprovisioning, attribute management |
| [Group Policy Administration](active-directory/03-group-policy-administration.md) | GPO creation, linking, troubleshooting, RSOP |
| [AD Security & Hardening](active-directory/04-ad-security-hardening.md) | Tiered admin model, PAWs, PIM, attack paths |
| [AD Replication & Sites](active-directory/05-ad-replication-sites.md) | Site links, KCC, ISTG, replication monitoring |
| [LDAP & Directory Services](active-directory/06-ldap-directory-services.md) | Queries, attributes, service accounts, binds |
| [AD Disaster Recovery](active-directory/07-ad-disaster-recovery.md) | Authoritative restore, forest recovery, USN rollback |
| [Azure AD / Entra ID Integration](active-directory/08-azure-ad-entra-integration.md) | Sync, hybrid join, Conditional Access, SSPR |

### 👥 Roles & Responsibilities
| File | Description |
|------|-------------|
| [IT Director Role & Responsibilities](roles/01-it-director.md) | Strategy, budget, governance, vendor management |
| [IT Manager Role & Responsibilities](roles/02-it-manager.md) | Team management, SLA ownership, project oversight |
| [Systems Administrator (IC)](roles/03-sysadmin-ic.md) | Day-to-day operations, patching, monitoring |
| [Network Engineer (IC)](roles/04-network-engineer-ic.md) | Design, implementation, maintenance |
| [Help Desk / Desktop Support Technician](roles/05-helpdesk-technician.md) | L1/L2 support, ticketing, user onboarding |
| [Security Engineer (IC)](roles/06-security-engineer-ic.md) | SIEM, vulnerability management, incident response |
| [Cloud / DevOps Engineer (IC)](roles/07-cloud-devops-engineer-ic.md) | IaC, CI/CD, cost optimization, reliability |
| [IT Operations Lead / Team Lead](roles/08-it-ops-lead.md) | Escalation point, mentoring, process improvement |

### 🔐 Security
| File | Description |
|------|-------------|
| [Security Incident Response Playbook](security/01-incident-response-playbook.md) | PICERL framework, playbooks, runbooks |
| [Identity & Access Management](security/02-identity-access-management.md) | IAM lifecycle, RBAC, PAM, access reviews |
| [Patch Management Policy & Procedure](security/03-patch-management.md) | Cadence, testing, rollback, reporting |
| [Vulnerability Management Program](security/04-vulnerability-management.md) | Scanning, CVSS, remediation SLAs, exceptions |
| [Security Awareness & Phishing](security/05-security-awareness.md) | Training programs, simulation, metrics |
| [Endpoint Security](security/06-endpoint-security.md) | EDR, AV, DLP, device encryption |
| [MFA & Authentication Standards](security/07-mfa-authentication-standards.md) | Methods, enrollment, exception process |

### 🏗️ Infrastructure
| File | Description |
|------|-------------|
| [Server Build & Hardening Standards](infrastructure/01-server-build-hardening.md) | CIS benchmarks, baseline config, documentation |
| [Storage & Backup Administration](infrastructure/02-storage-backup-administration.md) | SAN, NAS, backup jobs, RTO/RPO, restore testing |
| [Virtualization Administration](infrastructure/03-virtualization-administration.md) | VMware/Hyper-V, vMotion, snapshots, capacity |
| [Monitoring & Alerting](infrastructure/04-monitoring-alerting.md) | Tools, thresholds, on-call, escalation |
| [Certificate Management](infrastructure/05-certificate-management.md) | PKI, renewal, inventory, expiry alerting |
| [Asset Management & CMDB](infrastructure/06-asset-management-cmdb.md) | Lifecycle, discovery, configuration items |

### ☁️ Cloud Operations
| File | Description |
|------|-------------|
| [Cloud Governance Framework](cloud/01-cloud-governance-framework.md) | Landing zones, tagging, cost controls, policies |
| [Azure Operations Guide](cloud/02-azure-operations-guide.md) | Resource groups, RBAC, Cost Management, policies |
| [AWS Operations Guide](cloud/03-aws-operations-guide.md) | IAM, Organizations, GuardDuty, Config, budgets |
| [Microsoft 365 Administration](cloud/04-m365-administration.md) | Tenant config, licensing, compliance, Teams admin |

### 🎫 Help Desk Operations
| File | Description |
|------|-------------|
| [Ticketing System Guide & SLAs](helpdesk/01-ticketing-slas.md) | Priority matrix, SLA definitions, escalation paths |
| [New Employee Onboarding IT Checklist](helpdesk/02-new-employee-onboarding.md) | Account creation, hardware provisioning, access |
| [Employee Offboarding IT Checklist](helpdesk/03-employee-offboarding.md) | Account disablement, data retention, hardware return |
| [Common User Requests Runbook](helpdesk/04-common-user-requests-runbook.md) | Password resets, software installs, access requests |

### 🤝 Vendor & Procurement
| File | Description |
|------|-------------|
| [Vendor Management Guide](vendor/01-vendor-management.md) | Evaluation, contracts, SLAs, renewals, offboarding |
| [Software License Management](vendor/02-software-license-management.md) | Inventory, compliance, true-up, SAM tools |

### 🔄 Change Management
| File | Description |
|------|-------------|
| [Change Management Process](change-management/01-change-management-process.md) | RFC, CAB, standard vs emergency, rollback |
| [Maintenance Window Standards](change-management/02-maintenance-window-standards.md) | Scheduling, communication, approvals |
| [IT Runbook Template](change-management/03-runbook-template.md) | Standard format for all operational procedures |

---

## 📌 Quick Reference

### Priority / Severity Matrix
| Priority | Response | Resolution | Example |
|----------|----------|------------|---------|
| P1 – Critical | 15 min | 4 hrs | Total outage, security breach |
| P2 – High | 1 hr | 8 hrs | Partial outage, exec impacted |
| P3 – Medium | 4 hrs | 2 days | Single user, non-critical service |
| P4 – Low | 1 day | 5 days | How-to questions, minor requests |

### Escalation Path
```
L1 Help Desk → L2 Systems/Network Admin → L3 Senior Engineer → IT Manager → IT Director
```

### Key Contacts Template
| Role | Name | Phone | Email | On-call |
|------|------|-------|-------|---------|
| IT Director | | | | |
| IT Manager | | | | |
| NOC / On-call | | | | Pagerduty |
| Security Team | | | | SIEM alert |
| Vendor Support | | | | Per contract |
