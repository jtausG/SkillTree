# IT Vendor Management & Contract Guide

## Table of Contents
1. [Vendor Strategy](#strategy)
2. [Vendor Selection & RFP](#selection)
3. [Contract Fundamentals](#contracts)
4. [SLA Management](#sla)
5. [Vendor Performance Management](#performance)
6. [Software Licensing](#licensing)
7. [Hardware Procurement](#hardware)
8. [Cloud Provider Management](#cloud)
9. [Vendor Risk Management](#risk)
10. [Offboarding Vendors](#offboarding)
11. [Templates](#templates)

---

## 1. Vendor Strategy <a name="strategy"></a>

### Vendor Tiering Model
```
Tier 1 — Strategic Partners (5-10 vendors)
- Mission-critical systems, deep integration
- Executive relationships, quarterly business reviews (QBR)
- Joint roadmap planning
- Examples: Microsoft, ServiceNow, VMware, SIEM vendor

Tier 2 — Preferred Vendors (15-25 vendors)
- Important but not mission-critical
- Regular account reviews (semi-annual)
- Preferred pricing agreements in place
- Examples: Dell/HP hardware, ISP, backup vendor

Tier 3 — Commodity/Spot Vendors (remaining)
- Transactional relationships
- No special relationship management
- Shop competitively each purchase
- Examples: Office supplies, cabling vendor, low-cost SaaS

Benefits of tiering:
- Concentrate relationship investment where it matters
- Negotiate better terms with strategic vendors
- Reduce proliferation of vendors (consolidation savings)
```

### Vendor Consolidation
```
Signs you have too many vendors:
- Multiple vendors for same category (5 cloud tools doing similar things)
- Duplicate licensing fees
- Increased attack surface (more vendors = more risk)
- Management overhead too high

Consolidation process:
1. Audit all vendors and spend (accounts payable data)
2. Categorize by function
3. Identify overlaps and redundancies
4. Evaluate consolidation options per category
5. Build business case (cost savings vs. migration risk)
6. Execute consolidation over 12-18 months
7. Measure and report savings

Target: 20-30% vendor reduction is typical for mature consolidation effort
```

### Total Cost of Ownership (TCO)
```
For any major vendor purchase, calculate TCO (3-5 years):

Direct costs:
- License/subscription fees
- Hardware costs
- Professional services / implementation
- Training and enablement
- Support and maintenance contracts

Indirect costs:
- Internal IT labor to manage
- Integration development and maintenance
- End-user productivity impact
- Security and compliance overhead
- Migration / exit costs at end of contract

TCO comparison example (on-prem vs. SaaS):
                    On-Prem    SaaS
Year 1 license:     $80,000    $30,000
Year 1 hardware:    $40,000    $0
Year 1 PS:          $20,000    $5,000
Annual maintenance: $15,000    $30,000 (included in SaaS)
IT labor (annual):  $25,000    $10,000
3-year TCO:         $295,000   $140,000
Winner: SaaS (53% savings, but validate assumptions carefully)
```

---

## 2. Vendor Selection & RFP <a name="selection"></a>

### RFI vs. RFP vs. RFQ
| Document | Purpose | When to Use |
|----------|---------|------------|
| RFI (Request for Information) | Market research, early discovery | Don't know what solutions exist |
| RFP (Request for Proposal) | Competitive selection with requirements | Know requirements, selecting vendor |
| RFQ (Request for Quote) | Pricing only for defined specs | Know exactly what you want, need pricing |
| SOW (Statement of Work) | Define project-specific work | After selecting vendor, defining project |

### RFP Document Structure
```
SECTION 1: EXECUTIVE SUMMARY
- Organization background
- Project overview and business problem
- What you're looking for
- Timeline

SECTION 2: SCOPE OF REQUIREMENTS
2.1 Functional Requirements
    - Must-have (numbered: FR-001, FR-002...)
    - Nice-to-have (numbered: FR-O-001...)
2.2 Technical Requirements
    - Infrastructure requirements
    - Integration requirements
    - Performance/scale requirements
2.3 Security Requirements
    - Compliance certifications required (SOC 2, ISO 27001)
    - Data residency requirements
    - Authentication requirements (SSO, MFA)
2.4 Support Requirements
    - Response time SLAs
    - Support hours and channels

SECTION 3: VENDOR QUALIFICATION CRITERIA
- Years in business
- Customer references (similar size/industry)
- Financial stability (audited financials)
- Certifications

SECTION 4: RESPONSE FORMAT
- Executive summary (2 pages max)
- Company overview and financials
- Technical approach per requirement
- Implementation plan and timeline
- Pricing (separate sealed envelope or separate section)
- References (3 minimum, similar size)

SECTION 5: EVALUATION CRITERIA AND WEIGHTINGS
Clearly state how you will score responses:
- Technical fit: 35%
- Implementation approach: 20%
- Company stability: 15%
- Total cost of ownership: 20%
- Support model: 10%

SECTION 6: PROCESS AND TIMELINE
- RFP issue date
- Q&A deadline (questions submitted)
- Q&A response date
- Proposal due date
- Evaluation period
- Finalist notification
- Decision date

SECTION 7: TERMS AND CONDITIONS
- NDA requirement before receiving RFP
- IP ownership of responses
- Reservation of right to reject all proposals
- Confidentiality of pricing
```

### Vendor Evaluation Scorecard
```
Vendor Evaluation Matrix

Criteria                | Weight | Vendor A | Vendor B | Vendor C
Functional Requirements | 35%    |          |          |
  - Req FR-001          |        | 4/5      | 5/5      | 3/5
  - Req FR-002          |        | 5/5      | 4/5      | 4/5
  - Req FR-003          |        | 3/5      | 5/5      | 5/5
  Subtotal              |        | 4.0      | 4.67     | 4.0

Implementation Approach | 20%    |          |          |
  - Timeline realism    |        | 4/5      | 3/5      | 5/5
  - Methodology         |        | 5/5      | 4/5      | 3/5
  Subtotal              |        | 4.5      | 3.5      | 4.0

Company Stability       | 15%    |          |          |
  - Financial health    |        | 5/5      | 3/5      | 4/5
  - Customer references |        | 4/5      | 4/5      | 3/5
  Subtotal              |        | 4.5      | 3.5      | 3.5

Total Cost (3-year TCO) | 20%    |          |          |
  - $240k               |        | 4/5      |          |
  - $195k               |        |          | 5/5      |
  - $280k               |        |          |          | 3/5
  Subtotal              |        | 4.0      | 5.0      | 3.0

Support Model           | 10%    |          |          |
  - SLA commitments     |        | 4/5      | 4/5      | 5/5
  - Support hours       |        | 5/5      | 3/5      | 5/5
  Subtotal              |        | 4.5      | 3.5      | 5.0

WEIGHTED TOTAL                  | 4.18     | 4.28     | 3.78

RECOMMENDATION: Vendor B (best technical fit + lowest TCO despite lower support score)
```

### Reference Check Questions
```
Required: Call at least 3 references for finalists.
Ask for references from similar-size organizations in similar industries.

Questions to ask:
1. How long have you been using this vendor's product?
2. What was the implementation experience like? Any issues?
3. How does the support team respond to problems?
4. What has surprised you (positively or negatively) since going live?
5. What does the product do well? Where does it fall short?
6. Did the product deliver the promised ROI?
7. Is there anything you wish you had known before selecting this vendor?
8. Would you select them again today?
9. Are you planning to renew your contract?
10. What one thing would you change about the relationship?

Red flags in references:
- Reference can't say anything specific (coached response)
- Reference mentions problems they "resolved" but won't elaborate
- Reference was the internal champion who is now employed at the vendor
```

---

## 3. Contract Fundamentals <a name="contracts"></a>

### Key Contract Clauses to Negotiate
```
1. PAYMENT TERMS
   - Net 30 or Net 45 (never pay on delivery for services)
   - Milestone-based payments for professional services
   - Annual invoicing upfront (often 5-10% discount available)
   - Early payment discounts

2. PRICE PROTECTION
   - Price caps for renewal (max 3-5% annual increase)
   - Multi-year pricing locks
   - Most Favored Nation (MFN) clause (same price as best customer)

3. SLA COMMITMENTS
   - Uptime guarantees (99.9% = ~8.7 hrs downtime/year)
   - Response times for different severity levels
   - Credit/remedy structure for SLA breaches
   - Measurement methodology (how is uptime calculated?)

4. DATA PROVISIONS
   - Data ownership (your data is yours, always)
   - Data portability (extract your data at any time)
   - Data deletion (complete deletion within 30 days of termination)
   - Data residency (where is data stored? Can it leave the country?)

5. SECURITY & COMPLIANCE
   - Security standards vendor must maintain (SOC 2 Type II, ISO 27001)
   - Right to audit clause
   - Breach notification timeline (usually 72 hours)
   - Sub-processor notification requirements

6. INTELLECTUAL PROPERTY
   - Your IP remains yours (including data, configurations, custom code)
   - Vendor cannot use your data to train AI without consent
   - Work for hire provisions for custom development

7. TERMINATION
   - Termination for cause (material breach + cure period)
   - Termination for convenience (30-90 days notice)
   - Termination assistance (vendor helps with migration for 90 days)
   - Data return upon termination

8. LIABILITY
   - Cap on vendor liability (negotiate to 12-24 months of fees, not 1 month)
   - Indemnification for IP infringement
   - Carve-outs for gross negligence and willful misconduct

9. CHANGE MANAGEMENT
   - Vendor must notify you of material product changes (90 days)
   - Pricing changes with advance notice
   - API changes with deprecation notice (12 months minimum)

10. ASSIGNMENT
    - Vendor cannot assign contract without your consent (M&A protection)
    - Your right to assign if your company is acquired
```

### Software License Types
| License Type | Description | Watch Out For |
|-------------|-------------|---------------|
| Per user (named) | Specific individuals licensed | Count carefully; avoid sharing |
| Per user (concurrent) | Number of simultaneous users | Requires license server, audit risk |
| Per device | Licensed to specific hardware | Device refresh requires new license |
| Per CPU/core | Based on server CPU count | Virtualization can increase count |
| Site license | All users at a location | Define "location" carefully |
| Enterprise agreement | All users org-wide | Often best TCO, requires scale |
| SaaS subscription | Monthly/annual per user | Price increases at renewal |
| Perpetual | One-time fee, permanent use | May require separate maintenance |

### Contract Negotiation Tips
```
Preparation:
- Know your BATNA (Best Alternative To Negotiated Agreement)
- Research market rates (analyst reports, peer benchmarks)
- Know your timeline (urgency weakens your position)
- Identify all stakeholders in the deal on vendor side

Tactics:
- Never accept the first offer
- Negotiate on value, not just price (add services, extend term)
- Bundle purchases for better pricing
- Use competitive pressure (even if you prefer this vendor)
- End-of-quarter leverage (vendor reps have quotas)
- Multi-year commitments for better rates (25-40% discount possible)

What vendors will negotiate:
- Price (always try — worst they can say is no)
- Payment terms
- Contract length
- Implementation services
- Training credits
- SLA commitments
- Auto-renewal terms (eliminate or reduce notice window)

What's harder to negotiate:
- Fundamental product architecture
- Core security practices
- Terms imposed by their own upstream providers
```

---

## 4. SLA Management <a name="sla"></a>

### SLA Reference Guide
```
Uptime SLAs and what they mean:
99.0%   = 87.6 hours downtime/year    (7.3 hrs/month)
99.5%   = 43.8 hours downtime/year    (3.65 hrs/month)
99.9%   = 8.76 hours downtime/year    (43.8 min/month)
99.95%  = 4.38 hours downtime/year    (21.9 min/month)
99.99%  = 52.6 minutes downtime/year  (4.38 min/month)
99.999% = 5.26 minutes downtime/year  (26.3 sec/month)

"Uptime" definitions vary — always clarify:
- Platform reachable (most generous to vendor)
- Core features functional
- Full feature parity (strictest)
- Excluding scheduled maintenance windows
```

### SLA Credit Structure
```
Negotiate service credits that reflect actual business impact:

Uptime Achieved  | Credit
99.0% - 99.9%   | 10% monthly fee
95.0% - 99.0%   | 25% monthly fee
90.0% - 95.0%   | 50% monthly fee
< 90.0%         | 100% monthly fee (right to terminate)

Important credit terms to negotiate:
- Credits applied automatically (not requiring you to request)
- Credits applied to invoice, not as cash
- Credits do not waive right to terminate for cause
- Annual cap on credits (vendor likes cap, you want none)

When credits aren't enough — negotiate:
- Right to terminate without penalty if SLA missed 3 months in rolling 12
- Right to source backup solution at vendor's expense
- Executive escalation procedure for repeat misses
```

### SLA Monitoring
```
Monthly SLA review process:
1. Pull uptime/availability data from vendor portal
2. Cross-reference with your own monitoring (don't rely on vendor data alone)
3. Calculate credit owed (if any)
4. Document support ticket response times
5. Review open support tickets (aging analysis)
6. Prepare SLA performance summary for management

SLA scorecard metrics:
- System availability: Target vs. Actual
- Incident response time by severity: Target vs. Actual
- Incident resolution time by severity: Target vs. Actual
- Tickets opened/closed/aging
- Change success rate
- Service credit owed this period
```

---

## 5. Vendor Performance Management <a name="performance"></a>

### Quarterly Business Review (QBR) Agenda
```
QBR Agenda (Strategic Vendors — 2 hours)

1. Executive Welcome and Context (10 min)
   - Your company's strategic context, IT priorities

2. Relationship Health Check (15 min)
   - Overall relationship rating (1-5)
   - Key wins and successes this quarter
   - Areas of concern from both sides

3. SLA and Performance Review (20 min)
   - Uptime/availability (actual vs. SLA)
   - Support ticket metrics
   - Escalations and resolutions
   - Credits earned and applied

4. Product Roadmap Update (20 min)
   - What's coming in next 6-12 months
   - How does roadmap align with your needs?
   - Your top feature requests — status update

5. Roadblocks and Issues (15 min)
   - Open escalations
   - Contract issues
   - Pricing concerns

6. Joint Success Plan (20 min)
   - Your 12-month initiatives where this vendor is relevant
   - Resource and support needs
   - Training and adoption plans

7. Next Steps and Action Items (10 min)
   - Action items with owners and dates
   - Next QBR date
```

### Vendor Scorecard
```
Vendor: [Name] | Period: Q[X] [Year] | Reviewer: [Name]

PERFORMANCE DIMENSIONS          | Weight | Score (1-5) | Weighted Score
Service Availability/Reliability|  25%   |             |
Support Responsiveness          |  20%   |             |
Product/Service Quality         |  20%   |             |
Relationship/Proactiveness      |  15%   |             |
Contract Compliance             |  10%   |             |
Innovation/Roadmap              |  10%   |             |

TOTAL WEIGHTED SCORE: ___/5.0

Score Interpretation:
4.5 - 5.0: Outstanding — consider deeper strategic partnership
3.5 - 4.4: Good — continue with planned renewals
2.5 - 3.4: Needs Improvement — develop improvement plan with vendor
1.5 - 2.4: Poor — consider contract remedies or replacement
< 1.5:     Critical — initiate exit planning

NOTES:
[Specific achievements and concerns with examples]

ACTION ITEMS:
[Improvement actions with owner and dates]
```

---

## 6. Software Licensing <a name="licensing"></a>

### License Management Program
```
Software Asset Management (SAM) fundamentals:

1. DISCOVERY
   - Deploy SAM tool (e.g., Flexera, Snow, ServiceNow SAM)
   - Discover all installed software automatically
   - Include cloud SaaS subscriptions (often forgotten)
   - Discover virtual environments (VMs, containers)

2. ENTITLEMENT TRACKING
   - Import all license entitlements from contracts
   - Map discovered installs to entitlements
   - Calculate position: licensed vs. installed

3. OPTIMIZATION
   - Remove unused licenses (Shadow IT cleanup)
   - Right-size over-licensed products
   - Identify consolidation opportunities
   - Harvest unused licenses before purchasing more

4. COMPLIANCE
   - Identify unlicensed software (audit risk)
   - Track license compliance by vendor
   - Prepare for vendor audits proactively
```

### Vendor Audit Response
```
When you receive an audit notice:
1. Don't panic — vendors use audits as sales tactics
2. Respond promptly (acknowledge receipt, request timeline)
3. Engage legal and finance immediately
4. Pull your license entitlement documentation
5. Run internal SAM tool report immediately
6. Request audit scope limitations (specific products, date range)
7. Request audit in phases (not everything at once)
8. Never submit data directly — have legal review first

Negotiating audit findings:
- Challenge calculation methodology
- Present mitigating factors (products in POC, decommissioned servers)
- Negotiate a contract expansion vs. back-payment
- Request a compliance period to remediate
- Consider this an opportunity to negotiate a favorable EA

Common audit traps:
- Virtualization: Per-core licenses on VMware hosts
- Secondary use rights: Using virtualized software on portable devices
- Dev/Test: Using production licenses in dev without rights
- Bundles: Using components of a bundle beyond licensed use
```

### Microsoft Enterprise Agreement Management
```
EA Key Terms:
- True-Up: Annual reconciliation of actual users vs. enrolled
- Step-Up: Upgrade from lower to higher SKU (E3 → E5)
- Enrollment: 3-year commitment period
- Platform: Choice of Server & Cloud, Applications, or combined
- Qualified User: Employee who accesses services

EA Management best practices:
- Track user count monthly (not just at true-up)
- Review product usage vs. entitlement quarterly
- Use M365 usage reports to right-size
- Leverage "cloud optimization" credits when applicable
- Negotiate EA in Q4 (Microsoft fiscal year end = June 30)

M365 License right-sizing:
- F1: Frontline workers with limited access ($2.25/user/month)
- E1: Basic productivity, no desktop apps ($10/user/month)
- E3: Full productivity suite ($36/user/month)
- E5: E3 + advanced security + compliance ($57/user/month)

Common E5 upsell mistake: Buying E5 for everyone when only 10% need advanced security features. 
Right approach: E3 base + E5 Security add-on for security team only.
Savings: ~$21/user/month for 90% of users
```

---

## 7. Hardware Procurement <a name="hardware"></a>

### Procurement Process
```
Standard hardware procurement workflow:

1. Need identification
   - User/manager submits request with business justification
   - IT reviews vs. standards (approved hardware list)

2. Specification selection
   - Match standard spec to role:
     * Knowledge worker: Standard laptop (i5/16GB/256GB SSD)
     * Power user: Premium laptop (i7/32GB/512GB SSD)
     * Developer: Premium + external GPU option
     * Exec: Premium + lightweight option
     * Call center: Desktop or thin client
   - Non-standard requests require IT director approval

3. Procurement
   - Use preferred vendors (volume pricing)
   - Generate PO through procurement system
   - Confirm delivery timeline
   - Asset tag immediately upon receipt

4. Configuration
   - Autopilot/ADE registration before deployment
   - Standard image / OEM OS + Intune enrollment
   - QA test before shipping to user

5. Delivery
   - Asset tracked in CMDB
   - User acknowledges receipt
   - Old device collected (if refresh)

Lead times (plan accordingly):
- Laptops: 1-4 weeks (standard), 8-16 weeks (custom/shortage periods)
- Desktops: 1-3 weeks
- Servers: 8-24 weeks (critical: order early!)
- Networking equipment: 4-52 weeks (Cisco, Arista currently long lead)
```

### Approved Hardware Standards
```
Laptop Standards:
Tier        | Example Models         | Target Users
Standard    | Dell Latitude 5xxx     | General workforce
Premium     | Dell Latitude 7xxx/XPS | Managers, power users
Executive   | Dell XPS 13/MacBook Air| Executives, frequent travelers
Developer   | MacBook Pro/ThinkPad X1| Developers, data scientists

Desktop Standards:
Standard Desktop  | Dell OptiPlex Small Form Factor
Developer Desktop | Dell Precision Tower
All-in-One       | Dell OptiPlex AIO (where desk space limited)
Thin Client      | Dell Wyse (call centers, VDI environments)

Monitor Standards:
- Single: 24" 1080p (minimum standard)
- Dual: 27" 1440p (recommended for knowledge workers)
- Ultrawide: 34"+ (developer/financial analyst workflows)
```

### Hardware Refresh Planning
```
Refresh cycle policy:
Device Type    | Years | Trigger Criteria
Laptop         | 4     | Age OR: failure rate >2 repairs/year
Desktop        | 5     | Age OR: <8GB RAM, <256GB SSD, can't run Win 11
Server (phys)  | 5-7   | Age OR: end of vendor support
Network switch | 7-10  | Age OR: end of software support
Mobile device  | 3     | Age OR: OS no longer supported by MDM

Refresh budget planning:
Annual replacement budget = Total device count / Average useful life
Example: 500 laptops / 4 year life = 125 laptops/year
At $1,200/laptop average = $150,000/year refresh budget

Refresh wave planning:
- Identify devices by age from CMDB
- Prioritize oldest devices first
- Batch by department for efficiency
- Time with major OS upgrades or office moves
- Create 3-year rolling refresh plan
```

---

## 8. Cloud Provider Management <a name="cloud"></a>

### Cloud Cost Management
```
FinOps fundamentals for IT:

VISIBILITY (Month 1)
- Enable cost management dashboards (Azure Cost Management, AWS Cost Explorer)
- Tag all resources: Environment, Department, Project, Owner
- Establish showback/chargeback model
- Identify top 10 cost drivers

OPTIMIZATION (Month 2-3)
- Right-size oversized VMs (common: 20-30% reduction available)
- Purchase Reserved Instances for stable workloads (40-60% discount vs. on-demand)
- Delete unused resources (orphaned disks, unused IPs, test environments)
- Enable auto-shutdown for dev/test environments (nights/weekends)
- Review data transfer costs (often a surprise)

GOVERNANCE (Ongoing)
- Budget alerts at 80% and 100% of monthly allocation
- Require tags for all new resources (policy enforcement)
- Monthly cloud cost review with department heads
- Quarterly optimization review with cloud architect

Cloud cost optimization targets:
Year 1: Establish visibility and governance
Year 2: 20-30% cost reduction through right-sizing and Reserved Instances
Year 3: Advanced optimization (Spot instances, architecture refactoring)
```

### AWS Enterprise Support
```
AWS Support tiers:
- Basic: Free, documentation, community forums
- Developer: $29/month min, business hours support
- Business: $100/month min or 10% of usage, 24/7, < 1hr critical response
- Enterprise On-Ramp: $5,500/month, TAM pool, <30 min critical
- Enterprise: $15,000/month, dedicated TAM, <15 min critical

Enterprise Support negotiation:
- TAM assignment is negotiable (request specific TAM)
- Enterprise Discount Program (EDP) for committed spend:
  * $1M/year+ commitment = 5-15% discount
  * $3M/year+ = 15-25% discount
  * Multi-year commitments for higher discounts
- Negotiate training credits, migration assistance
```

### Azure Enterprise Agreement
```
Azure EA structure:
- Enrollment: 3-year commitment with Azure Prepayment
- Departments: Billing segmentation
- Subscriptions: Deployment containers
- Budgets: Cost controls per subscription

Negotiation points:
- Monetary commitment discount (higher commit = bigger discount)
- Azure Hybrid Benefit (use on-prem Windows/SQL licenses in Azure)
- Dev/Test pricing (significant discounts for non-prod workloads)
- Reserved VM Instances (1-year or 3-year at 40-60% discount)
```

---

## 9. Vendor Risk Management <a name="risk"></a>

### Vendor Risk Assessment
```
Annual assessment for all Tier 1 vendors, biennial for Tier 2:

FINANCIAL RISK
- Request audited financial statements or use Dun & Bradstreet
- Signs of distress: layoffs, executive departures, missed earnings
- Private equity ownership: May mean aggressive cost-cutting
- Mitigation: Escrow for critical IP, data portability rights

OPERATIONAL RISK
- Single points of failure in their operations
- Disaster recovery capabilities (ask for BCDR documentation)
- Geographic concentration of staff/infrastructure
- Mitigation: Redundancy SLAs, right to source backup at their cost

SECURITY RISK
- SOC 2 Type II (most important for cloud/SaaS)
- ISO 27001 certification
- Penetration test results (ask for executive summary)
- Incident history (public breaches, CVE responses)
- Subprocessor chain (your data passing through how many vendors?)
- Mitigation: Contract security requirements, right to audit, breach notification

COMPLIANCE RISK
- GDPR/CCPA: Does vendor handle personal data? DPA required.
- HIPAA: BAA required for covered entities
- PCI-DSS: QSA attestation for payment-related vendors
- Export controls (ITAR/EAR for certain tech vendors)
- Mitigation: Contractual representations, annual self-attestation

CONCENTRATION RISK
- Vendor provides >30% of critical infrastructure
- No viable alternatives exist (switching cost too high)
- Mitigation: Maintain competitive tension, multi-vendor strategy
```

### Third-Party Risk Management (TPRM) Program
```
TPRM workflow for new vendors:

1. Initial screening (before any access)
   - Business justification
   - Data classification (what data will they access?)
   - SOC 2 / security questionnaire review

2. Risk scoring
   - Criticality (what happens if they go down?)
   - Data sensitivity (PII, confidential, public)
   - Access level (read-only, admin, physical)
   - Score = Criticality x Data x Access (1-5 scale each)

3. Risk mitigation
   - High risk (>8): Full security review, legal review, DPA/BAA
   - Medium risk (4-8): Security questionnaire, DPA if data
   - Low risk (<4): Standard vendor agreement, self-attestation

4. Ongoing monitoring
   - Annual re-assessment for active vendors
   - Trigger re-assessment on: breach news, ownership change, scope expansion
   - Continuous monitoring via BitSight, SecurityScorecard, or similar

Vendor security questionnaire key areas:
- Data encryption (in transit and at rest)
- Access control and MFA
- Vulnerability management and patching SLA
- Incident response plan and notification procedures
- Employee background checks
- Subprocessors list
- Penetration test frequency and findings remediation
```

---

## 10. Offboarding Vendors <a name="offboarding"></a>

### Vendor Exit Checklist
```
Decision to exit (18 months before contract end):
- Build business case for replacement or renewal
- Identify replacement vendor
- Begin procurement process (RFP, selection)

12 months before exit:
- Notify vendor per contractual notice requirements
- Begin data export and migration planning
- Request data portability documentation from vendor
- Identify integration dependencies

6 months before exit:
- Execute migration project
- Parallel run period where applicable
- Training on new system
- Validate data completeness in new system

At contract termination:
- Confirm data deletion per SLA (get written confirmation)
- Revoke all access (API keys, SSO, admin accounts)
- Cancel all sub-accounts and integrations
- Terminate ACH/credit card authorizations
- Update asset records and CMDB
- Archive all contracts and documentation

Post-termination:
- Invoice reconciliation (final invoices can have errors)
- Security deactivation (remove vendor from authorized systems list)
- Update vendor list and TPRM database
- Document lessons learned
```

---

## 11. Templates <a name="templates"></a>

### Vendor Management Dashboard Metrics
```
Monthly Report — Vendor Management

SPEND SUMMARY
Total IT vendor spend this month: $[amount]
vs. Budget: $[+/-] [%]
vs. Same month prior year: $[+/-] [%]
Top 5 vendors by spend: [list]

CONTRACT RENEWALS (Next 90 days)
Vendor     | Contract End | Annual Value | Decision
[Vendor A] | [Date]       | $[amount]    | Renew / RFP / Exit
[Vendor B] | [Date]       | $[amount]    | Negotiating

SLA PERFORMANCE
Vendor     | SLA Metric | Target | Actual | Credit Earned
[Vendor A] | Uptime     | 99.9%  | 99.95% | $0
[Vendor B] | Response   | 4 hrs  | 6 hrs  | $[amount]

RISK ALERTS
- [Vendor C]: Press reports of financial difficulties — monitor closely
- [Vendor D]: SOC 2 report expired — requesting renewal

ACTION ITEMS
- [Action 1] — Owner: [Name] — Due: [Date]
- [Action 2] — Owner: [Name] — Due: [Date]
```

### Vendor Contract Summary Card
```
VENDOR CONTRACT SUMMARY

Vendor:           [Name]
Category:         [e.g., Cloud Infrastructure, Security, SaaS]
Tier:             1 / 2 / 3
Contract Start:   [Date]
Contract End:     [Date]
Auto-Renew:       Yes / No — Notice period: [X] days
Annual Value:     $[amount]
3-Year TCO:       $[amount]

KEY CONTACTS
Account Manager:     [Name] | [Email] | [Phone]
Support Portal:      [URL] | [Account ID]
Executive Contact:   [Name] | [Email]
IT Owner:            [Name]
Finance Owner:       [Name]

KEY CONTRACT TERMS
Uptime SLA:          [%]
Credit structure:    [details]
Data residency:      [Country/Region]
Notice to terminate: [days]
Price cap at renewal:[%]

SLA PERFORMANCE TREND
Q1: [Green/Yellow/Red]
Q2: [Green/Yellow/Red]
Q3: [Green/Yellow/Red]
Q4: [Green/Yellow/Red]

RENEWAL NOTES
[Key negotiation points, risks, decisions for next renewal]
```

---

*Last Updated: 2025 | IT Vendor Management Guide v2.0*
