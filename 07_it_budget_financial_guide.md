# IT Budget & Financial Management Guide

## Table of Contents
1. [IT Finance Fundamentals](#fundamentals)
2. [Budget Planning Process](#planning)
3. [CapEx vs. OpEx](#capex-opex)
4. [IT Cost Categories](#categories)
5. [Building the IT Budget](#building)
6. [Budget Presentation & Defense](#presentation)
7. [Cost Allocation & Chargeback](#chargeback)
8. [Cloud FinOps](#finops)
9. [Cost Reduction Strategies](#reduction)
10. [Financial Metrics & Reporting](#metrics)
11. [Business Case Templates](#business-case)

---

## 1. IT Finance Fundamentals <a name="fundamentals"></a>

### Key Financial Concepts for IT Leaders
```
CapEx (Capital Expenditure)
- One-time purchases of long-lived assets
- Depreciated over useful life (typically 3-5 years)
- Examples: Servers, network hardware, datacenter equipment
- Impact: Spread over multiple years on P&L
- CFO prefers: Can be financed; doesn't hit expense budget fully in year 1

OpEx (Operating Expenditure)
- Ongoing costs to run the business
- Expensed fully in the period incurred
- Examples: SaaS subscriptions, support contracts, cloud, salaries
- Impact: Full cost in current year P&L
- CFO prefers: Predictable, tied to business activity

Depreciation
- Annual expense for CapEx assets
- Straight-line most common: Asset value / Useful life years
- Example: $120,000 server / 5 years = $24,000/year depreciation

EBITDA impact
- IT costs reduce EBITDA (operating profit)
- CFOs track IT as % of Revenue (benchmark: 3-7% for mid-market)
- Cloud shift: CapEx → OpEx increases operating expense, reduces EBITDA margin

ROI (Return on Investment)
- ROI = (Benefit - Cost) / Cost x 100%
- Example: $200k project saves $80k/year
  Year 1: (80k - 200k) / 200k = -60% (investment year)
  Year 3: ($240k total savings - $200k) / $200k = 20% ROI
  Payback period: 200k / 80k/year = 2.5 years

NPV (Net Present Value)
- Accounts for time value of money
- Positive NPV = project creates value
- Required for projects >$250k at most companies
```

### IT Spend Benchmarks
| Metric | Small (<$50M rev) | Mid ($50-500M) | Large ($500M+) |
|--------|-------------------|----------------|----------------|
| IT as % of Revenue | 5-8% | 3-6% | 2-4% |
| IT staff per 100 employees | 2-4 | 1.5-3 | 1-2 |
| Helpdesk cost per ticket | $20-50 | $15-35 | $10-25 |
| Cost per managed device | $800-1500/yr | $600-1200/yr | $400-900/yr |
| Cloud as % of IT budget | 20-40% | 30-50% | 35-55% |

---

## 2. Budget Planning Process <a name="planning"></a>

### Annual Budget Calendar
```
Month 1-2 (July-August): Strategic planning
- IT leadership reviews 3-year roadmap
- Business unit input on upcoming projects
- Vendor contract renewals identified
- Technology refresh cycles assessed

Month 3 (September): Bottom-up build
- Department managers submit requests
- IT managers prepare detailed budgets
- New projects scoped with preliminary estimates
- Headcount plans developed

Month 4 (October): Consolidation and review
- IT finance consolidates all submissions
- Gap analysis vs. prior year
- Prioritization of new investments
- Initial challenge sessions with managers

Month 5 (November): Executive review
- IT Director/CIO reviews consolidated budget
- Challenge session with CFO/finance
- Trade-off decisions (what gets funded, what doesn't)
- Resubmissions after cuts

Month 6 (December): Final approval
- Board/executive approval
- Communication of approved budget
- Project pipeline confirmed
- Year 1 spending plans locked

January: New fiscal year begins
- Budget loaded into financial system
- Monthly actuals tracking begins
```

### Zero-Based vs. Incremental Budgeting
```
Incremental (most common):
- Start with prior year actuals
- Add/subtract based on known changes
- Quick, but perpetuates inefficiencies
- CFO may apply a flat cut ("take out 5%")

Zero-Based Budgeting (ZBB):
- Every dollar must be justified from scratch
- Identifies waste, but time-intensive
- Often used in IT for specific categories or during cost restructuring
- Useful for: Application portfolio rationalization

Hybrid approach (recommended):
- Run/grow activities: Incremental (baseline + trend factors)
- Transform activities: Zero-based (business case required)
- New technologies: Zero-based (full ROI analysis)
```

---

## 3. CapEx vs. OpEx <a name="capex-opex"></a>

### Traditional vs. Cloud-Era IT Spend
```
Traditional IT (CapEx-heavy):
- Buy servers, storage, networking hardware
- Capitalize at purchase
- Depreciate over 3-5 years
- Predictable, long-term commitment
- Large upfront cash requirement

Modern IT (OpEx-heavy):
- Cloud: Pay-as-you-go (OpEx)
- SaaS: Monthly/annual subscriptions (OpEx)
- IaaS/PaaS: Consumption-based (OpEx)
- Lower upfront cost, more flexibility
- Can grow/shrink based on business needs
- Risk: Costs can spike unexpectedly

CFO conversation: Cloud shift impact
"Moving to cloud will increase our operating expense by $X this year,
but eliminates the need for $Y in capital expenditure over 3 years.
Net TCO savings: $Z. The higher OpEx will reduce EBITDA margin by X%,
but improves cash flow flexibility."
```

### CapEx Decision Framework
```
When to CapEx vs. OpEx:
CapEx makes sense when:
- You need the asset for 5+ years with no expected change
- Purchase price significantly lower than lease/subscription over useful life
- Regulatory/compliance requires data on-premise
- Tax benefits of depreciation are valuable

OpEx/Cloud makes sense when:
- Flexibility needed (scale up/down)
- Rapid technology change expected
- Avoiding large upfront capital is strategic priority
- OpEx budget is less constrained than CapEx budget

Lease vs. Buy analysis:
Equipment cost: $100,000
Useful life: 5 years
Lease cost: $2,000/month = $120,000 over 5 years
Buy cost: $100,000 + $10,000 maintenance = $110,000 over 5 years
+ opportunity cost of capital
+ flexibility of end-of-lease refresh

General rule: If lease premium <15% over purchase, lease for flexibility.
```

---

## 4. IT Cost Categories <a name="categories"></a>

### IT Cost Structure
```
LABOR (typically 30-50% of IT budget)
- Internal IT staff salaries + benefits (loaded cost = salary x 1.25-1.35)
- Contractors and consultants
- Managed service provider fees

INFRASTRUCTURE (15-25%)
- Hardware: Servers, storage, networking, end-user devices
- Data center: Co-lo rent, power, cooling
- Telecommunications: WAN circuits, internet, SD-WAN
- Cloud IaaS/PaaS: Azure, AWS, GCP compute/storage/network

SOFTWARE (20-35%)
- Enterprise licenses: ERP, HRIS, CRM, M365
- Security tools: EDR, SIEM, vulnerability management
- Development tools, databases, middleware
- SaaS applications (increasingly the dominant category)

SUPPORT & MAINTENANCE (5-15%)
- Hardware maintenance contracts
- Software maintenance and support
- Vendor-managed services

PROJECTS (variable, typically 10-20%)
- Implementation professional services
- New project capital expenditure
- Migration costs

TRAINING & CERTIFICATION (1-3%)
- Staff training courses
- Certification exam fees
- Conference attendance
```

### Run/Grow/Transform Model
```
Run (keeping lights on — typically 60-70% of IT budget):
- Helpdesk operations
- Infrastructure maintenance
- License renewals
- Patching and security monitoring
- Standard hardware refresh

Grow (enabling business growth — typically 20-30%):
- New business systems for expansion
- Capacity increases for growth
- Process automation
- Enabling new business capabilities

Transform (changing the game — typically 10-20%):
- Digital transformation projects
- Major platform replacements
- Emerging technology pilots
- New business model enablement

Budget allocation health check:
- If Run > 75%: IT is too reactive, needs investment in modernization
- If Transform > 25%: May be under-investing in stable operations
- Target: Shift Run spending down 2-3% per year by automating and optimizing
```

---

## 5. Building the IT Budget <a name="building"></a>

### Budget Build Process
```
Step 1: Start with confirmed obligations (non-negotiables)
- Headcount costs (salaries + benefits already committed)
- Multi-year contracts (years 2, 3 of existing deals)
- Hardware under warranty/maintenance contract
- Depreciation on existing CapEx
Total: This is your fixed baseline

Step 2: Add known increases
- Salary increases (merit + market: typically 3-5%)
- Vendor price escalations (average IT vendor: 3-8% per year)
- Growth-based increases (headcount, cloud usage, new offices)
- License count increases (new employees)
Total: Baseline + known increases = Continuation budget

Step 3: Add renewals and replacements
- Contracts renewing (may change in cost)
- Hardware due for refresh (from asset database)
- Software end-of-life requiring replacement
Total: Steady-state run budget

Step 4: Add growth and new investments
- New projects approved in strategic planning
- Headcount additions
- Technology upgrades
- Each requires business case or justification
Total: Full budget request

Step 5: Prioritize and scenario-plan
- Rank all new investments by ROI/strategic importance
- Build three scenarios:
  * Base: Continuation + highest-priority new investments
  * Stretch: Base + moderate new investments
  * Constraint: Continuation budget only (what do we cut or defer?)
```

### Budget Template by Category
```
IT BUDGET WORKSHEET — FY[Year]

LABOR
Position          | HC | Annual Salary | Benefits (28%) | Total Loaded
IT Director       | 1  | $175,000      | $49,000        | $224,000
IT Manager        | 2  | $120,000      | $33,600        | $307,200
Sr. Systems Admin | 3  | $90,000       | $25,200        | $345,600
Systems Admin     | 4  | $70,000       | $19,600        | $358,400
Help Desk         | 6  | $50,000       | $14,000        | $384,000
[Subtotal Labor]                                        | $1,619,200

CONTRACTORS/MSP
Helpdesk overflow MSP        | $120,000
Security monitoring MSSP     | $180,000
[Subtotal Contractors]       | $300,000

INFRASTRUCTURE
Cloud - Azure                | $240,000
Cloud - AWS                  | $85,000
Co-location facility         | $180,000
Internet/WAN circuits        | $96,000
Hardware refresh (laptops)   | $150,000
[Subtotal Infrastructure]    | $751,000

SOFTWARE
Microsoft 365 (EA)           | $420,000
ServiceNow                   | $180,000
Security tools               | $240,000
Business applications        | $320,000
[Subtotal Software]          | $1,160,000

PROJECTS
Network refresh project      | $450,000 (CapEx)
Cloud migration Phase 2      | $280,000 (OpEx services)
[Subtotal Projects]          | $730,000

TOTAL IT BUDGET              | $4,560,200
vs. Revenue (assume $80M)    | 5.7%
vs. Prior Year ($4.2M)       | +8.6%
```

---

## 6. Budget Presentation & Defense <a name="presentation"></a>

### CFO Communication Framework
```
Structure your budget presentation in this order:

1. BUSINESS CONTEXT (not IT context)
   - What is the business doing next year?
   - What are the top 3 business priorities?
   - How does IT support each priority?

2. LAST YEAR PERFORMANCE
   - Did we deliver what we promised?
   - Actual vs. budget (explain material variances)
   - Key accomplishments (business outcomes, not IT outputs)

3. WHAT WE'RE COMMITTING TO
   - Top 3-5 IT outcomes for next year (measurable)
   - How each ties to business strategy

4. WHAT IT COSTS
   - Budget summary (high level, not line-by-line)
   - Year-over-year change with key drivers explained
   - Benchmark comparison (are we expensive or cheap vs. industry?)

5. TRADE-OFFS
   - Show what doesn't get funded if budget is cut
   - Let the CFO make the trade-off decision (don't bury it)

6. RISK PROFILE
   - What risks are we managing with this budget?
   - What risks increase if budget is reduced?

Golden rule: Talk about business outcomes, not technology features.
Wrong: "We need $450k for SD-WAN to replace MPLS"
Right: "We need $450k for network modernization that reduces circuit costs by $180k/year while improving reliability for our 8 remote offices"
```

### Handling Budget Cuts
```
When told to cut 10-15%:

Step 1: Don't just cut across the board (shows lack of priorities)

Step 2: Identify cut options by impact:
Option A — Low impact cuts (do these first):
  - Defer non-critical projects 6 months (-$X)
  - Renegotiate vendor contracts (-$X)
  - Reduce contractor hours (-$X)
  - Eliminate low-use SaaS tools (-$X)
  Subtotal: -$X (% of budget)

Option B — Medium impact cuts (present risk):
  - Defer hardware refresh (increases failure risk by X%)
  - Reduce training budget (retention/skill risk)
  - Reduce security tool coverage (name specific risk)
  Subtotal: -$X (% of budget)

Option C — High impact cuts (escalate):
  - Reduce helpdesk staffing (SLA impact: tickets go from 4hr to 8hr response)
  - Cancel strategic project (describe business impact)
  - Reduce cloud capacity (performance impact)
  Subtotal: -$X (% of budget)

Step 3: Present options to CFO/business
"We can achieve the 12% reduction by doing A+B, but there are risks.
Option C would achieve the target but here are the business impacts.
What would you like us to prioritize?"

Key: Never just cut and say nothing. Always surface the risk of each cut.
```

---

## 7. Cost Allocation & Chargeback <a name="chargeback"></a>

### Models for IT Cost Allocation
```
Model 1: Single Pool (simplest)
- IT costs pooled and split equally by headcount or revenue
- Easy to administer
- Not tied to actual consumption
- Risk: Large departments subsidize small, no incentive to optimize

Model 2: Cost Centers
- IT allocated to cost centers by fixed percentages
- Percentages set annually (negotiated or usage-based survey)
- More accurate than single pool
- Still not real-time consumption

Model 3: Chargeback (true showback)
- IT bills each department for actual usage
- Most accurate, creates demand management
- Highest administrative overhead
- Risk: Department managers find workarounds, Shadow IT increases

Model 4: Showback (recommended for most)
- Show departments what they would be charged
- No actual money transfer
- Drives awareness without billing complexity
- Good stepping stone to full chargeback

Allocation metrics by category:
- Helpdesk: Per incident/request
- End-user compute: Per device
- Email/M365: Per user
- Shared infrastructure: CPU/storage consumption
- Applications: Per named user or per business unit
```

### Implementing Showback
```
Phase 1: Tag everything in financial systems
- Projects → assign project codes
- Cloud resources → tag with department, project
- License purchases → allocate by department

Phase 2: Build monthly reports
- Show each business unit their IT consumption
- Trend over time
- Comparison vs. peer departments (normalized by headcount)

Phase 3: Present monthly to business leaders
- Review in business unit reviews
- Discuss optimization opportunities
- Build shared accountability for IT costs

Sample showback report:
Department: Sales
Month: October 2025

Service           | Units | Unit Cost | Monthly Total
Managed Devices   | 42    | $85       | $3,570
M365 Licenses     | 42    | $36       | $1,512
Salesforce        | 38    | $150      | $5,700
Helpdesk Tickets  | 47    | $28       | $1,316
Corporate Apps    | 42    | $45       | $1,890

Total IT Cost:     $13,988
Per Headcount:     $333/employee
vs. Company Avg:   $310/employee (+7.3%)
```

---

## 8. Cloud FinOps <a name="finops"></a>

### FinOps Practice Maturity
```
Crawl (Month 1-3):
- Enable billing dashboards (Azure Cost Management, AWS Cost Explorer)
- Create basic tagging policy
- Identify top 5 cost drivers
- Set up budget alerts
- Assign cloud cost ownership to IT

Walk (Month 4-9):
- Tagging compliance >80%
- Right-size top 20 oversized VMs
- Purchase Reserved Instances for stable workloads
- Auto-shutdown dev/test environments
- Monthly cost review cadence established
- Showback to departments

Run (Month 10+):
- Reserved Instance coverage >70%
- Spot/Preemptible Instances for appropriate workloads
- Savings Plans optimized
- Architecture optimization (eliminate data transfer costs)
- Engineering teams own their cloud costs (FinOps embedded in teams)
- Quarterly optimization reviews
```

### Reserved Instance Strategy
```
Reserved Instances (RI) / Savings Plans — Key concepts:

When to buy RIs:
- Workload has been running stably for 3+ months
- Usage is predictable (doesn't disappear on weekends)
- Minimum 70% utilization of RI commitment

RI terms:
- 1-year: ~30-40% discount vs. on-demand
- 3-year: ~50-65% discount vs. on-demand (recommend only if very stable)

RI payment options:
- All Upfront: Maximum discount (5-10% more than partial)
- Partial Upfront: Balanced (most common)
- No Upfront: Smaller discount, maximum flexibility

Azure approach: Reserved VM Instances + Savings Plans (compute)
AWS approach: Reserved Instances + Compute Savings Plans (more flexible)

RI management process:
1. Monthly: Review RI utilization (target >85%)
2. Monthly: Identify on-demand spend for new RI candidates
3. Quarterly: Purchase new RIs for stable workloads
4. Quarterly: Sell underutilized RIs (AWS Marketplace, Azure canceled)
5. Annually: Review all expiring RIs for renewal decision
```

---

## 9. Cost Reduction Strategies <a name="reduction"></a>

### Cost Reduction Playbook
```
QUICK WINS (30-60 days, minimal risk)
1. SaaS application audit
   - Export list of all SaaS subscriptions from expense system
   - Identify duplicate functionality (how many project management tools?)
   - Find unused licenses (last login >90 days)
   - Target: 15-25% SaaS spend reduction

2. Cloud rightsizing
   - Pull rightsizing recommendations from cloud console
   - Filter for >30% CPU headroom recommendations
   - Resize down one tier after validating with app owner
   - Target: 15-25% compute cost reduction

3. Vendor contract renegotiation
   - Identify contracts renewing in next 6 months
   - Get competitive quotes before renewal discussions
   - Use multi-year commitment for better pricing
   - Target: 5-15% on renewed contracts

MEDIUM-TERM (3-6 months)
4. Reserved Instances / Savings Plans
   - Identify steady-state cloud workloads
   - Purchase appropriate commitment level
   - Target: 30-40% on covered workloads

5. Hardware refresh delay (with risk acceptance)
   - Extend lifecycle of equipment 1 year beyond standard
   - Requires executive risk acceptance
   - Increase maintenance/warranty accordingly
   - Target: Defer $X in refresh spending 12 months

6. Insource/outsource analysis
   - Compare outsourced functions vs. insourcing cost
   - Managed services often cheaper for routine functions
   - Internal expertise better for strategic/complex work

STRATEGIC (6-18 months)
7. Application rationalization
   - Identify all applications in portfolio
   - Categorize: Strategic / Tactical / Legacy / Retire
   - Retire 20-30% of applications (typical finding)
   - Target: Eliminate $X in licensing and maintenance costs

8. Data center exit / cloud migration
   - Co-lo and on-prem infrastructure → cloud
   - Eliminate hardware refresh CapEx
   - Reduce data center facility costs
   - Target: 30-40% infrastructure cost reduction (varies)

9. Process automation
   - Automate repetitive IT tasks (user onboarding, patching)
   - Reduce helpdesk volume through self-service
   - Free up staff for higher-value work
   - Target: 20% reduction in helpdesk tickets
```

---

## 10. Financial Metrics & Reporting <a name="metrics"></a>

### Monthly IT Financial Report
```
MONTHLY IT FINANCIAL SCORECARD

Period: [Month Year]

BUDGET PERFORMANCE
Category        | Budget  | Actual  | Variance | YTD Budget | YTD Actual | YTD Var
Labor           | $180k   | $182k   | -$2k     | $1,440k    | $1,438k    | +$2k
Infrastructure  | $62k    | $58k    | +$4k     | $496k      | $489k      | +$7k
Software        | $97k    | $101k   | -$4k     | $776k      | $784k      | -$8k
Projects        | $45k    | $38k    | +$7k     | $360k      | $340k      | +$20k
TOTAL           | $384k   | $379k   | +$5k     | $3,072k    | $3,051k    | +$21k

YTD: $21k under budget (0.7% favorable) — ON TRACK

KEY VARIANCES
+ Infrastructure $7k favorable: Delayed server purchase (Q4 still planned)
- Software $8k unfavorable: M365 seats above forecast due to 12 new hires in Oct

CLOUD SPEND
Azure this month:    $38,500 (budget: $40,000)
AWS this month:      $7,200  (budget: $7,000)
Top 5 cost drivers:  [List by service]
RI coverage:         72% (target: 75%)

UPCOMING COMMITMENTS
Nov: Cisco SmartNet renewal — $45,000
Dec: ServiceNow renewal — $180,000 (under negotiation, target <$165,000)

FORECAST TO YEAR END
Full-year budget:    $4,608,000
Forecast spend:      $4,580,000
Projected variance:  +$28,000 favorable (0.6%)
```

### IT Financial KPIs
| KPI | Formula | Target | Benchmark |
|-----|---------|--------|-----------|
| IT as % of Revenue | IT Budget / Annual Revenue | 3-6% | Industry dependent |
| IT Cost per Employee | Total IT Budget / Employee Count | $5k-15k | Varies by industry |
| Cloud % of IT Budget | Cloud Spend / Total IT Budget | 30-50% | Growing each year |
| Run vs. Transform % | Run Cost / Total IT Budget | <70% Run | Depends on maturity |
| IT Staff Ratio | IT Staff / Total Employee Count | 1:50-1:100 | Varies by industry |
| Cost per Helpdesk Ticket | Helpdesk Cost / Ticket Volume | $20-40 | Lower with self-service |
| Budget Accuracy | (Actual - Budget) / Budget | ±5% | <3% excellent |

---

## 11. Business Case Templates <a name="business-case"></a>

### IT Investment Business Case
```
BUSINESS CASE: [Project/Investment Name]

EXECUTIVE SUMMARY
[3-5 sentences: Problem, proposed solution, key financial summary, recommendation]

PROBLEM STATEMENT
Current state: [Describe current situation with data/metrics]
Business impact: [How does this affect the business? Quantify if possible]
Risk of inaction: [What happens if we don't invest?]

PROPOSED SOLUTION
Description: [What are we proposing to buy/build/do?]
Alternatives considered:
  Option A: [Do nothing] — Cost/Risk/Impact
  Option B: [Minimal investment] — Cost/Risk/Impact
  Option C: [Proposed solution] — Cost/Risk/Impact (recommended)
  Option D: [Premium option] — Cost/Risk/Impact

FINANCIAL ANALYSIS

Costs (3-year):
Year 1: $[amount] (Implementation: $X + Licensing: $Y + Labor: $Z)
Year 2: $[amount] (Ongoing: $X + Labor: $Y)
Year 3: $[amount] (Ongoing: $X + Labor: $Y)
3-Year Total Cost: $[amount]

Benefits (3-year):
Hard benefits (quantified):
  - Labor savings from automation: $[amount]/year
  - License consolidation savings: $[amount]/year
  - Infrastructure cost reduction: $[amount]/year
  Total hard benefits (3-year): $[amount]

Soft benefits (not quantified):
  - Improved employee experience
  - Reduced security risk
  - Enhanced scalability

Financial Summary:
  Total 3-year cost:         $[amount]
  Total 3-year benefit:      $[amount]
  Net 3-year value:          $[amount]
  ROI:                       [%]
  Payback period:            [months]
  NPV (at 8% discount rate): $[amount]

RISK ANALYSIS
Implementation risks: [key risks and mitigations]
Technology risks: [key risks and mitigations]
Risk of inaction: [what risk does this investment mitigate?]

RECOMMENDATION
Approve investment of $[amount] in FY[Year].
Expected payback: [X] months.
Funding source: [CapEx/OpEx / existing budget / new request]

APPROVALS REQUIRED
IT Director: By [Date]
CFO: By [Date]
CEO/Board: (if >$[threshold])
```

---

*Last Updated: 2025 | IT Budget & Financial Management Guide v2.0*
