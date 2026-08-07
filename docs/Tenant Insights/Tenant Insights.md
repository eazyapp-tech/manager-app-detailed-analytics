> [!WARNING] Superseded in part — 2026-08-07
> This document (v1.1, **3 Feb 2026**) is still the best statement of *why* this screen exists, and its Snapshot-vs-Flow model, its invariants and its glossary were all carried forward. **[[DA-09 Tenants — Handoff Sheet]] is what gets built**, and it overrides this document on the seven points below. Everything not listed here still stands.
>
> **1. Voluntary vs eviction notices cannot be told apart.** This document builds an invariant, two urgency scales and two chart types on that split. The system records **who raised the notice** — the tenant, or the property — not why. A manager recording "she is leaving on the 30th" is indistinguishable from a manager removing somebody, and the reason exists only in free-typed remarks. The handoff uses *raised by the tenant* / *raised by the property*, which is true and just as useful. **This also answers this document's own blocking item #2.**
>
> **2. Active includes people under notice.** This document sets Total Tenants = Active + Under Notice as two separate groups. The handoff keeps one headcount, with under notice as a sub-slice inside it — matching what the system does, and matching the Inventory screen, which counts a tenant under notice as still occupying their bed. Read "Total Tenants" here as "Active Tenants" there.
>
> **3. The time filter is the suite's, not this one's.** This Month (default) · Last Month · Current FY · Custom · All Time · Coming up. Today, Yesterday, Next 7 and Next 30 are dropped so the control matches every other tab.
>
> **4. The screen does not send anything.** Threshold callouts stay — they are this document's best idea. The button opens the filtered list, where messaging a group already works.
>
> **5. Money is two figures, not a category.** Rent at risk, and dues owed by people leaving. Both confirmed buildable: rent is set on **97.3%** of tenants under notice. This document's fallback plan for missing rent data is no longer needed.
>
> **6. Three of the eight blocking items are answered.** #1 move-out dates exist on 97% of departed tenants · #2 see above · #4 agreement coverage is **21%** of active tenants.
>
> **7. Compliance percentages need a coverage line.** This document's invariants assume every present tenant has exactly one state for agreement, police verification and eKYC. In practice most have none recorded, so a bare percentage reads as failure when it is absence. Each shows what it is based on.

**

# PART 0: EXECUTIVE SUMMARY 

## What We're Building

Tenant Insights is an early warning system for tenant health in the RentOk Manager App.

It answers the five questions operators ask daily: "Kitne ja rahe hain?" (exits), "Kiska renewal pending hai?" (retention), "Compliance kaisa hai?" (legal risk), "Bookings convert ho rahe hain?" (acquisition), and "Portfolio ka haal kaisa hai?" (composition).

Unlike Tenant List (individual records) or Financial Insights (₹ transactions), this screen shows patterns, trends, and actionable alerts — surfacing problems BEFORE they cause damage.

---

## The Problem

### Meet Rajesh (Primary Persona)

Rajesh manages 3 PG properties in Koramangala, Bangalore — 500 beds total, mix of working professionals and students. He has 4 staff members across properties. The owner, Mr. Sharma, calls every Monday asking "tenant health kaisa hai?"

### Also Meet Sunita (Secondary Persona — Tier-2/Seasonal)

Sunita runs a 200-bed girls' hostel near a coaching institute in Kota. Her business is seasonal — 90% occupancy during the coaching season (July-May), 40% in summer. Her tenants are students aged 16-20, and their parents make all decisions. Police verification compliance is strictly monitored in her area.

Why two personas: Rajesh represents metro multi-property operators (Bangalore, Gurgaon, Pune). Sunita represents tier-2 single-property operators with seasonal patterns and stricter compliance environments. Both need the same core visibility but with different urgency drivers.

---

### What Happened Last Month

#### Rajesh's Month (Metro, Multi-Property)

Week 1: Surprise Exits

Rajesh discovers 3 rooms are empty. Tenants moved out over the weekend. He didn't know their notice period ended — they'd given notice 28 days ago, he forgot to track.

Cost: 3 beds × ₹12,000 rent × 0.5 month vacancy = ₹18,000 lost

Week 2: Renewal Blindness

4 tenants give sudden one-day notice. "Agreement khatam ho gaya, naya nahi kiya, toh hum ja rahe hain." Rajesh didn't know their agreements had expired 2 months ago — no one initiated renewal.

Cost: 4 beds × ₹10,000 × 1 month vacancy = ₹40,000 lost + acquisition cost

Week 3: Police Raid

Routine police verification drive. They check PV records. 23 tenants (out of 180) don't have PV completed — some joined 3 months ago. Warning notice issued. Next time: fine + license risk.

Cost: Warning notice (no fine this time) + 2 days of Rajesh's time = ₹5,000 opportunity cost + future risk exposure

Week 4: Owner Call

Mr. Sharma calls: "Retention kaisa hai? Pichle mahine se better hai ya worse? Kitne log renew kar rahe hain?" Rajesh: "Sir, main check karke batata hoon..." He spends 3 hours pulling data. Still can't answer the retention question — he doesn't track it.

Cost: Owner trust eroded, Rajesh looks incompetent

Week 4: Month-End Discovery

Reviewing move-outs, Rajesh finds 6 tenants left with unpaid dues totaling ₹48,000. He only discovered this AFTER they vacated. Recovery is now nearly impossible.

Cost: ₹48,000 likely written off

#### Sunita's Month (Tier-2, Seasonal, Compliance-Heavy)

The PV Deadline Crisis

Sunita's area has a strict 14-day PV requirement. 8 new students joined after semester started. Day 12: she realizes none have PV completed — Aadhaar copies weren't collected properly during rushed check-ins.

Day 15: Deadline missed for all 8. Local police are known to fine ₹10,000 per tenant for violations.

Cost: ₹80,000 fine exposure for 8 tenants

The Seasonal Churn

May arrives. Coaching year ends. 60 students give notice in the same week. Sunita has no way to see which ones have pending dues, which have deposit refunds due, which agreements need closure.

Cost: Chaos. 3 students leave without settling ₹35,000 total dues. Deposit refund disputes with 5 families.

---

### The Monthly Toll (Rajesh's 500-Bed Operation)

|   |   |   |   |   |
|---|---|---|---|---|
|Blind Spot|What Happens|Frequency|Unit Cost|Monthly Cost|
|Surprise exits (no notice tracking)|Tenants leave, rooms sit empty|3-5 tenants|₹10K × 0.5 month vacancy|₹15,000-25,000|
|Renewal blindness (no pipeline)|Tenants churn that could've renewed|4-6 tenants|₹10K × 1 month vacancy|₹40,000-60,000|
|Compliance gaps (no deadline tracking)|Warning notices, future fine risk|15-25 tenants|Risk exposure|₹10,000-25,000 amortized|
|Defaulter exits (no flag before move-out)|Tenants leave with unpaid dues|5-8 tenants|₹6-8K avg dues|₹30,000-64,000|
|Total Monthly Exposure||||₹95,000-1,74,000|

Annualized for 500-bed operation: ₹12-20L in preventable losses

Note: Compliance cost is amortized monthly risk exposure. Actual fine when it hits could be ₹5-25K per tenant.

---

### The Root Cause

Rajesh and Sunita aren't lazy or incompetent. They're blind.

|   |   |   |
|---|---|---|
|What They Need to Know|Where That Data Lives|Why They Can't See It|
|Who's under notice right now?|Tenant List (filter by status)|No aggregate view, no urgency buckets|
|Whose notice period ends this week?|Calculated from notice date + period|No deadline tracking surfaced|
|Whose agreement expires this month?|Agreement records|Not surfaced, no proactive alert|
|Who's been here <90 days? (churn risk)|Calculated from move-in dates|No one calculates this|
|Who's missing PV? How many days are overdue?|Compliance records|No deadline tracking by tenant|
|What's our renewal rate? Churn rate?|Would need manual cohort analysis|Doesn't exist as a metric|
|How many defaulters are under notice?|Cross-reference dues + status|No overlay view|
|Is this month better or worse than last?|Manual comparison|No trends tracked|

The problem isn't time spent collecting data. The problem is: by the time they learn about an issue, it's already cost money.

---

## The Solution

### Tenant Insights: From Blind to Proactive

One screen that surfaces what matters BEFORE it becomes a problem.

### The Daily Check (60 seconds)

|   |   |   |
|---|---|---|
|What Rajesh Does|What He Sees|Time|
|Opens Tenant Insights|Overview loads with current property, MTD default|2 sec|
|Scans callouts in Overview section|🔴 "8 PV overdue >14 days"|5 sec|
||🟠 "12 agreements expire this week — 9 no renewal initiated"||
|Notes exit risk|Notices: 15 (₹1.8L monthly rent at risk) • Confirmed exits: 8 this month|5 sec|
|Checks retention health|Churn: 6.2% ↓ • Renewal rate: 72% ↑ • 90-day retention: 84%|5 sec|
|Sees compliance status|PV: 92% complete • Agreements: 88% valid • eKYC: 95% done|5 sec|
|Decides where to act|Taps "12 agreements expiring" → filtered tenant list → starts renewal calls|10 sec|

Total: 30-60 seconds to know where to focus. Action before damage.

### The Owner Call (Instant Answers)

|   |   |
|---|---|
|Mr. Sharma Asks|Rajesh Answers (from screen)|
|"Kitne tenant hain?"|"412 active, 15 under notice, 23 confirmed move-outs this month"|
|"Retention kaisa hai?"|"90-day retention is 84%, up from 79% last month. Monthly churn is 6.2%."|
|"Renewals ho rahe hain?"|"72% renewal rate this month. 23 due, 12 in progress, 8 completed."|
|"Compliance theek hai?"|"92% PV complete. 8 overdue beyond 14 days, I'm following up today."|
|"Pichle mahine se better?"|"Move-ins up 15%, churn down 1.2 points, renewal rate up 8 points."|

Total: Instant. No "I'll check and call back."

### The Weekly Review (5 minutes)

|   |   |   |
|---|---|---|
|Section|What Rajesh Reviews|Action Taken|
|Notices & Exits|15 under notice → 6 in 0-7 days bucket (urgent) → 4 have pending dues|Assigns staff to collect dues before exit|
|Renewal Pipeline|23 due → 9 not initiated → 5 are long-tenure high-value|Calls top 5 personally to retain|
|Compliance|8 PV overdue → 3 are >30 days (critical)|Escalates to staff, sends tenant reminders|
|Eviction Pipeline|4 active evictions → 2 in 0-7 days bucket (immediate action needed)|Coordinates with lawyer for urgent cases|
|Acquisition|Booking conversion 68% → cancellation rate 22%|Investigates cancellation reasons|

### The Monthly Report (2 minutes)

|   |   |   |   |
|---|---|---|---|
|Metric|This Month|Last Month|Trend|
|Total Tenants|412|405|↑ +7|
|Move-ins|34|29|↑ +17%|
|Move-outs|27|31|↓ -13%|
|Net Change|+7|-2|↑ Improved|
|Churn Rate|6.2%|7.4%|↓ Better|
|Renewal Rate|72%|64%|↑ +8pp|
|90-Day Retention|84%|79%|↑ +5pp|
|PV Compliance|92%|87%|↑ +5pp|

Screenshot or drill-down → Share with owner via WhatsApp. Done.

---

## Scope Boundary

### This Screen Covers

|   |   |   |
|---|---|---|
|Area|What's Included|Why It Matters|
|Portfolio Snapshot|Total tenants, active, under notice, move-ins, move-outs with trends|"Kitne hain, kitne aa rahe, kitne ja rahe"|
|Exit & Notice Tracking|Notices by type (voluntary/eviction), by days-based urgency bucket, monthly rent at risk, defaulter overlay|"Kaun ja raha hai, kab tak, kitna risk hai"|
|Renewal & Retention|Agreements expiring (by days bucket), renewal pipeline (due → in progress → completed), renewal rate, churn rate, 90-day retention, early termination rate|"Kaun ruk raha hai, kaun churn ho sakta hai"|
|Compliance Status|PV status with days-since-move-in tracking, Agreement status (valid/expired/none), eKYC status, with urgency callouts for overdue items|"Legal risk kahan hai"|
|Acquisition Funnel|Bookings → Approved → Cancelled → Converted (moved-in), conversion rate, cancellation rate|"Pipeline kaisa hai"|
|Composition|Tenure distribution, tenant type, demographics (gender, age, food preference, institute, office)|"Kaun hai humare tenants"|
|Revenue Context (Limited)|₹ monthly rent at risk (notices), ₹ dues unpaid (defaulters exiting) — as context indicators only|"₹ impact kya hai"|
|Actionable Callouts|Threshold-based alerts in Overview section (e.g., "8 PV overdue >14 days → View & Send Reminders")|"Kahan action lena hai"|
|Trends & Comparison|MoM deltas, trend arrows, period-over-period comparison|"Better ho raha hai ya worse"|

### This Screen Does NOT Cover

|   |   |   |
|---|---|---|
|Area|Where It Lives|Boundary|
|Financial transactions (collections, expenses, deposits, invoices)|Financial Insights|We show ₹ risk indicators, not ₹ transactions|
|Room/bed-level occupancy|Occupancy Insights|Asset view, not tenant view|
|Individual tenant record management (edit, update)|Tenant Detail screen|We aggregate, don't manage individuals|
|Lead management (pre-booking stage)|Lead Insights|Sales funnel before booking|
|Bulk tenant operations (bulk reminders, bulk updates)|Tenant List screen|Action screen, not insights|
|Actual PV submission, agreement creation/signing|Compliance Module|We show status, don't manage process|
|Data export to PDF/Excel|TBD (see Open Items)|Ownership unclear|

### Scope Clarification: Revenue Context

|   |   |
|---|---|
|We Show (Risk Indicators)|We Don't Show (Transactions)|
|"Notices = ₹1.8L monthly rent at risk" (sum of rent for tenants under notice)|Actual rent collection records|
|"Defaulters exiting = ₹48K dues unpaid" (sum of outstanding for move-outs with dues)|Dues breakdown, payment history|

What We Don't Show: "Revenue secured from renewals" — this would require rent amount per tenant which may live in Financial Service. Deferred to Phase 2 if needed.

Rationale: Operators think in ₹. "15 notices" means little. "15 notices = ₹1.8L at risk" drives action. But transaction management (collection, settlement) belongs in Financial Insights.

---

## Success Metrics

### Primary Outcomes (Business Impact)

|   |   |   |   |
|---|---|---|---|
|Metric|Current State|Target (6 months)|How to Measure|
|Unaction exits|8-10/month exit with no engagement during notice period|<2/month|Exits where no recorded activity (call, renewal initiation, settlement attempt) during notice period. Requires activity logging.|
|Renewal conversion rate|~55% (estimated)|>75%|Renewals completed ÷ Agreements expired in period|
|Monthly churn rate|Unknown (not tracked)|<8%, tracked monthly|Move-outs ÷ Avg active tenants in period|
|90-day retention|Unknown (not tracked)|>85%, tracked monthly|Tenants staying >90 days ÷ Move-ins from 90+ days ago|
|Compliance gaps at audit|15-20 tenants with issues|<5 tenants|PV overdue + Agreement expired/missing, discovered by police or external audit|
|Defaulters flagged before exit|30-40% identified before move-out|>90% identified|Tenants with outstanding dues whose status was visible (flagged in Tenant Insights) before their move-out date|

### Secondary Outcomes (Operational Efficiency)

|   |   |   |   |
|---|---|---|---|
|Metric|Current State|Target|How to Measure|
|Time to answer owner questions|"Let me check and call back" (hours)|Instant (<60 sec)|Owner feedback, support tickets|
|Screen adoption|—|80% of managers visit weekly|Product analytics|
|Action-to-insight ratio|—|60% of sessions include drill-down or action|Product analytics|

### North Star

Operators know their tenant's health at a glance and act on problems before they cost money.

Measured by: Reduction in unactioned exits + Improvement in renewal rate + Zero compliance gaps at audit

---

## Module Dependencies & Impact

### This Module Depends On (Upstream)

|   |   |   |   |
|---|---|---|---|
|Module|What We Need|If It Changes|Severity|
|Tenant Service|Tenant records, status, move-in/out dates, notice dates, demographics|All metrics break|Critical|
|Booking Service|Booking records, status, conversion dates, cancellation reasons|Acquisition funnel breaks|Critical|
|Agreement Service|Agreement records, start/end dates, renewal status|Renewal pipeline breaks|Critical|
|Compliance Service|PV status, PV submission date, eKYC status|Compliance section breaks|Critical|
|Notice/Eviction Service|Notice records, notice type (voluntary/eviction), notice date, notice period|Exit tracking breaks|Critical|
|Financial Service|Outstanding dues per tenant (for defaulter flags, ₹ at risk calculation), monthly rent per tenant|Defaulter overlays break, ₹ context breaks|High|
|Property Service|Property details, bed inventory|Property filter, context breaks|High|
|Staff/Team Service|Staff assignments (for eviction ownership attribution)|Attribution breaks|Medium|

### This Module Impacts (Downstream)

|   |   |   |
|---|---|---|
|Module|What It Uses From Us|If We Change|
|Homescreen (Section 9: Lifecycle)|May pull same tenant KPIs (Active, Under Notice, Churn)|Numbers must match exactly|
|Financial Insights|Cross-links for "tenants with dues under notice"|Navigation, filter inheritance|
|Notifications Service|May trigger alerts based on thresholds (e.g., "PV overdue")|Alert logic needs sync|
|Reports Module|May pull same metrics for scheduled/exported reports|Calculation consistency required|

### Shared Components

|   |   |   |
|---|---|---|
|Component|Also Used By|Change Coordination|
|Time Filter Component|All Insights screens|Design system team|
|Property Filter Component|All Insights screens, Homescreen|Design system team|
|Drill-down List Component|Financial Insights, Occupancy Insights|List patterns team|
|Trend Indicator (↑↓ with color)|Homescreen, all Insights|Design system team|
|Action Callout Component|Financial Insights (Dues reminders)|Notifications team|
|Compliance Status Indicators|Tenant Detail screen|Compliance team|

---

## Open Items

### Blocking (Must Resolve Before Dev)

|   |   |   |   |   |
|---|---|---|---|---|
|#|Item|Question|Owner|Status|
|1|Historical move-out dates|Do we have accurate move-out dates for all past tenants? Needed for churn calculation baseline.|Data|Open|
|2|Notice type field|Is voluntary vs eviction notice type captured in current schema?|Backend|Open|
|3|Notice date + period|Do we store notice start date and notice period (days) to calculate notice end date?|Backend|Open|
|4|Agreement coverage baseline|What % of current active tenants have agreement records? Need baseline before launch.|Ops|Open|
|5|PV deadline configuration|Different states have different PV deadlines (14/21/30 days). Is this configurable per property?|Product|Open|
|6|Monthly rent per tenant|Where does monthly rent amount live? Tenant Service or Financial Service? Needed for "₹ at risk" calculation.|Backend|Open|
|7|Performance at scale|Can we calculate all metrics real-time for 500+ bed properties without latency?|Backend|Open|
|8|Activity logging for "unactioned exits"|Success metric requires knowing if any engagement happened during notice period. Do we log calls, renewal attempts?|Product|Open|

### Non-Blocking (Resolve During or After Dev)

|   |   |   |   |   |
|---|---|---|---|---|
|#|Item|Description|Owner|Target|
|9|Export functionality|Does Tenant Insights have its own export (PDF/Excel), or do users go to Reports Module? Need to decide ownership.|Product|Before launch|
|10|Homescreen sync|Ensure Homescreen Section 9 (Lifecycle) pulls from same data source as Tenant Insights. Numbers must match.|Product|Before launch|
|11|Seasonality adjustment|For operators like Sunita, trends should account for seasonal patterns (e.g., July influx, May exodus). Phase 2?|Product|Phase 2|
|12|Year-over-year comparison|Toggle to compare vs same period last year.|Product|Phase 2|
|13|Revenue secured from renewals|"₹ secured" metric requires rent × renewed tenants. Deferred until rent data integration confirmed.|Product|Phase 2|
|14|Cohort analysis|"Tenants who joined in Jan — where are they now?"|Product|Phase 2|
|15|Property comparison|"Which property has best retention?" Needs dedicated comparison view.|Product|Phase 2|

---

## Document History

|   |   |   |   |
|---|---|---|---|
|Version|Date|Author|Changes|
|0.1|Jan 27, 2026|—|Initial FRs drafted|
|0.2|Jan 27, 2026|—|Added missing metrics (churn, retention, renewal rate), split funnels|
|0.3|Jan 27, 2026|—|Consolidated with workflows, edge cases, addendum merged|
|1.0|Feb 3, 2026|—|Full PRD format, Part 0 rewritten with operator perspective|
|1.1|Feb 3, 2026|—|Part 0 corrected: fixed math reconciliation, clarified personas, resolved scope conflicts, added blocking open items|

---

## Part 0 Verification Checklist

|   |   |   |
|---|---|---|
|Check|Status|Notes|
|Real persona with name, role, scale, context|✅|Rajesh (primary), Sunita (secondary with specific scenario)|
|Specific pain scenarios with ₹ impact|✅|5 Rajesh incidents + 2 Sunita incidents, all with ₹|
|Monthly toll math reconciles|✅|₹95K-1.74L/month, annualized ₹12-20L|
|Root cause identified (blind spots, not time)|✅|Table of what they need vs why they can't see it|
|Solution as early warning system|✅|Daily/Weekly/Monthly workflows with specific actions|
|Scope boundary clear|✅|In/Out tables, ₹ context vs ₹ transactions explicit|
|Revenue context limited and defined|✅|Only "at risk" shown, "secured" deferred to Phase 2|
|Success metrics tied to business outcomes|✅|Unactioned exits, renewal rate, churn, compliance|
|Success metrics measurable|✅|Each has measurement method noted|
|Dependencies documented|✅|Upstream with severity, downstream with impact|
|Open items include actual blockers|✅|8 blocking items with owners|
|Export ownership addressed|✅|In Open Items #9|
|Seasonality addressed|✅|In Open Items #11|
|Callout location clarified|✅|"in Overview section" stated|
|Time comparisons use supported filters|✅|Changed "last quarter" to "last month"|
|Eviction language consistent|✅|Days-based buckets throughout|
|Both personas used meaningfully|✅|Sunita has specific PV deadline + seasonal scenarios|
|No orphaned features|✅|Everything traces to problem or deferred|
|No conflicting statements|✅|Scope, revenue, metrics aligned|
|No undefined calculations|✅|All formulas explicit or in Open Items|

---

## Reconciliation: Part 0 vs FRs

|   |   |   |
|---|---|---|
|Part 0 Claim|Where It Must Appear in FRs|Status|
|Total Tenants = Active + Under Notice|Section 1: Overview|✅ In FR 1.1|
|Notices split by voluntary/eviction|Section 1 or 6: Exit tracking|✅ In FR 1.5, 6.x|
|₹ at risk = sum of monthly rent for notice tenants|Section 1 or 3|✅ In FR 1.5.1|
|Defaulter overlay on notices|Section 1 or 3|✅ In FR 1.5.2|
|Renewal rate %|Section 5: Renewal|✅ In FR 5.8|
|Churn rate %|Section 3: Move-in/Move-out|✅ In FR 3.6|
|90-day retention %|Section 2 or 3|✅ In FR 2.13|
|PV with days-since-move-in tracking|Section 7: Compliance|✅ In FR 7.x|
|Agreement status (valid/expired/none)|Section 7: Compliance|✅ In FR 7.7.1|
|Callouts with thresholds|Section 9: Action Callouts|✅ In FR 9.x|
|Time filter: Today, Yesterday, MTD, Last Month, Custom Date, Custom Range|Global Filters|✅ In FR G.2|
|MoM comparison logic|Global Filters|✅ In FR G.3|
|Drill-down preserves filters|Global Filters|✅ In FR G.4|

---

  
  

# PART 1: FOUNDATIONS

Module: Tenant Detail Insights Version: 1.1 Date: February 3, 2026

---

## 1.1 Mental Model

### How Operators Should Think About This Screen

Tenant Insights is your tenant health dashboard — a diagnostic panel, not a transaction screen.

Think of it like the dashboard of a car:

- You glance at it to know if something needs attention (warning lights)
    
- You see overall health (speed, fuel, temperature)
    
- You don't repair the engine from the dashboard — you go to the mechanic (Tenant List, Compliance Module)
    

|   |   |
|---|---|
|Dashboard Analogy|Tenant Insights Equivalent|
|Warning lights|Action Callouts ("8 PV overdue", "12 renewals expiring")|
|Speedometer|Churn rate, Net change, Conversion rates|
|Fuel gauge|Compliance %, Renewal rate|
|Odometer|Total tenants, Avg tenure|
|Trip computer|Period comparisons, Trends|

### The Two Types of Data

Every metric on this screen falls into one of two categories:

|   |   |   |   |
|---|---|---|---|
|Type|What It Answers|Time Behavior|Examples|
|Snapshot|"How many RIGHT NOW?"|Always shows current live state, regardless of time filter|Total Tenants, Active, Under Notice, Compliance %, Eviction Pipeline, Demographics|
|Flow|"How many DURING PERIOD?"|Changes based on time filter selected|Move-ins, Move-outs, Bookings created, Renewals completed, Churn Rate|

Why this matters: When Rajesh selects "Last Month" filter:

- Snapshot metrics still show TODAY's counts (current tenants, current compliance)
    
- Flow metrics show LAST MONTH's activity (move-ins, move-outs, bookings created)
    

This is intentional. Operators need current state visibility even when reviewing historical performance.

  
  
  
  
  

### The Lifecycle Lens

Tenant Insights is organized around the tenant lifecycle:

ACQUISITION          ACTIVE               RETENTION            EXIT

─────────────────────────────────────────────────────────────────────

Booking → Move-in → Active Tenant → Renewal/Notice → Move-out/Eviction

         │              │                │                │

    Conversion      Compliance       Renewal Rate      Churn Rate

    Rate            Tracking         90-Day Retention   Revenue at Risk

  

Each section of the screen maps to a lifecycle stage. Operators can focus on the stage that needs attention.

### Drill-Down Philosophy: Insight → List → Action

1. Insight: See aggregated metric (e.g., "15 under notice")
    
2. Drill-down: Tap to see filtered list (15 tenants, sorted by urgency)
    
3. Action: Take action on individual (call, renew, settle dues)
    

The count on the Insights screen MUST match the count in the drill-down list. If Insights shows "15 under notice", the list must show exactly 15 records. (See INV-17.)

---

## 1.2 Core Concepts & Glossary

### 1.2.1 Key Population Concept: "Total Tenants"

Throughout this screen, the base population is Total Tenants = tenants physically present in the property.

|   |   |   |
|---|---|---|
|Population Term|Definition|When Used|
|Total Tenants|Active + Under Notice|Base population for most metrics, compliance denominators|
|Active Tenants|Tenants currently staying, no exit notice|Subset of Total Tenants|
|Under Notice|Tenants under notice, still physically present|Subset of Total Tenants|

Why this matters: A tenant who gave notice yesterday is still physically in the bed, still needs PV compliance, and still counts toward occupancy. They leave the population only when they physically move out.

---

### 1.2.2 Tenant Statuses

|   |   |   |   |
|---|---|---|---|
|Status|Definition|In "Total Tenants"?|In Property?|
|Active|Tenant currently staying, no exit notice given|Yes|Yes|
|Under Notice|Tenant has given notice (voluntary) OR received eviction notice, still physically present|Yes|Yes|
|Evicted|Tenant has vacated (eviction completed)|No|No|

---

### 1.2.3 Notice Types

|   |   |   |   |
|---|---|---|---|
|Type|Definition|Initiated By|Typical Duration|
|Voluntary Notice|Tenant decides to leave|Tenant|15–30 days (per agreement)|
|Eviction Notice|Property decides tenant must leave|Property/Manager|Varies (legal process, no fixed end date)|

Under Notice = Voluntary Notices + Eviction Notices

Time Direction Difference:

- Voluntary Notice: has a known exit date → tracked as days remaining (countdown to exit)
    
- Eviction: has no fixed end date → tracked as days since issued (duration of process)
    

This is intentional. Voluntary notices have a deadline to act before (retention attempt, replacement booking). Evictions have no deadline — you track how long it's taking.

---

### 1.2.4 Booking Statuses

|   |   |   |   |
|---|---|---|---|
|Status|Definition|Transitions From|Transitions To|
|Pending|Booking created, awaiting approval|—|Approved, Cancelled|
|Approved|Booking confirmed, awaiting move-in|Pending|Converted, Cancelled|
|Cancelled|Booking cancelled (by tenant or property)|Pending, Approved|— (terminal)|
|Converted|Booking completed, tenant moved in|Approved|— (becomes Active Tenant)|

Rules:

- Total Bookings (in period) = Created in period, across all statuses
    
- Converted can only come from Approved (not direct from Pending)
    
- Once Converted, booking is closed — tenant record takes over
    

---

### 1.2.5 Agreement Statuses

|   |   |   |   |
|---|---|---|---|
|Data Status|Definition|Criteria|UI Display|
|Valid|Agreement is current|End date ≥ Today|Shows as "Valid"|
|Valid (Expiring)|Agreement current but ending soon|End date ≥ Today AND End date within 60 days|Shows as "Expiring" with days remaining|
|Expired|Agreement ended, tenant still present|End date < Today AND Tenant in Total Tenants|Shows as "Expired" with days overdue|
|None|No agreement record exists|No agreement linked to tenant|Shows as "No Agreement"|

Key Clarification: "Expiring" is a display state within Valid, not a separate data status. For invariant purposes: Valid (including Expiring) + Expired + None = Total Tenants.

Why this matters for INV-9: The system stores agreements with end dates. "Expiring" is a UI highlight for agreements ending within 60 days — the underlying status is still "Valid."

---

### 1.2.6 Compliance Items

Legal Compliance (Regulatory Risk):

|   |   |   |   |
|---|---|---|---|
|Item|Definition|Status Values|Deadline Rule|
|Police Verification (PV)|Mandatory verification with police|Not Started, Submitted, Verified|Due within X days of move-in (configurable per property: 14/21/30 days)|
|Agreement|Rent agreement signed|Valid, Expired, None|Expected at move-in, renewed before expiry|
|eKYC|Identity verification via Aadhaar/documents|Complete, Incomplete|No fixed deadline, expected at move-in|

Operational Compliance (Business Practice):

|   |   |   |   |
|---|---|---|---|
|Item|Definition|Status Values|Deadline Rule|
|Autopay|Automatic rent deduction setup|Enabled, Not Enabled|Optional, no deadline|

Why the split: Police will check PV and Agreement. They won't check Autopay. Legal compliance items carry fine/license risk. Operational items are business best practices. Both appear in the Compliance section of the screen but are labeled distinctly.

PV Overdue Definition: Tenant where (Today − Move-in Date) > PV Deadline (property-configured) AND PV Status = Not Started

PV Status Logic:

|   |   |   |
|---|---|---|
|PV Status|Condition|Overdue?|
|Verified|PV completed and verified by police|No|
|Submitted|PV documents submitted, awaiting verification|No|
|Not Started|No PV action taken|Yes, if past deadline|
|Not Started|No PV action taken|No, if within deadline|

---

### 1.2.7 Renewal Concepts

|   |   |
|---|---|
|Term|Definition|
|Renewal Due|Agreement ending within next 60 days, renewal not yet completed. This is a FORWARD-LOOKING pipeline.|
|Renewal In Progress|Renewal initiated (discussion started, new agreement drafted) but not signed|
|Renewal Completed|New agreement signed, extending tenancy|
|Renewal Rate|(Renewals Completed in Period ÷ Agreements Expired in Period) × 100|

Critical Distinction:

- Renewal Due = FUTURE-looking. Agreements that WILL expire within 60 days. Pipeline for action.
    
- Renewal Rate = PAST-looking. Agreements that DID expire in period, what % got renewed. Performance metric.
    

These are different populations with different time orientations.

---

### 1.2.8 Eviction Concepts

|   |   |
|---|---|
|Term|Definition|
|Active Eviction|Eviction notice issued, tenant has NOT yet vacated (still physically present)|
|Completed Eviction|Tenant has physically vacated following eviction process|
|Eviction Pipeline|All Active Evictions, bucketed by days since notice issued|

---

### 1.2.9 Tenure Buckets

|   |   |   |
|---|---|---|
|Bucket|Definition|Risk Implication|
|< 30 days|New tenant|Highest churn risk (settling in)|
|1–3 months|Early stage|High churn risk (still evaluating)|
|3–6 months|Established|Medium risk|
|6–12 months|Stable|Lower risk|
|> 12 months|Long-term|Lowest churn risk|

Tenure Calculation: Today − Move-in Date (in days, converted to months for bucketing) Population: Active Tenants only (Under Notice excluded — they've already decided to leave)

---

### 1.2.10 Urgency Buckets

Notice Urgency — Days REMAINING Until Exit (Countdown):

|   |   |   |
|---|---|---|
|Bucket|Definition|Action|
|0–7 days|Exit imminent|Critical — settle dues, arrange replacement|
|8–15 days|Exit soon|High — retention attempt or replacement|
|16–30 days|Standard notice|Medium — initiate retention conversation|
|30+ days|Early notice|Low — plan ahead|

Applies to Voluntary Notices with known exit date.

Eviction Progress — Days SINCE Notice Issued (Count-Up):

|   |   |   |
|---|---|---|
|Bucket|Definition|Action|
|0–7 days|Just started|Initiate legal process|
|8–15 days|Early stage|Follow up on legal steps|
|16–30 days|Mid process|Escalate if no progress|
|31–45 days|Extended|Review legal status|
|45+ days|Prolonged|Escalate to management/lawyer|

Applies to Eviction Notices. No fixed end date — tracking duration.

---

### 1.2.11 Financial Context Terms

|   |   |   |   |
|---|---|---|---|
|Term|Definition|Source|Dependency|
|Monthly Rent|Tenant's monthly rent amount|Financial Service|Open Item #6 — data source TBD|
|Outstanding Dues|Total unpaid amount for tenant|Financial Service|Available|
|Defaulter|Tenant with Outstanding Dues > 0|Calculated from Financial Service|See note below|
|Revenue at Risk|Σ(Monthly Rent) for all tenants in Under Notice status|Calculated|Depends on Monthly Rent availability|
|Dues at Risk|Σ(Outstanding Dues) for tenants in Under Notice AND Defaulter = true|Calculated|Available|

Defaulter Threshold Note: Currently defined as Outstanding Dues > 0. If operators need a minimum threshold (e.g., >₹500 to exclude rounding), this becomes a configurable setting documented in Part 4. For v1, >0 is the definition.

Revenue at Risk Fallback: If Monthly Rent per tenant is unavailable (Open Item #6 unresolved), Revenue at Risk shows count only: "15 tenants under notice" without ₹ value. ₹ value added once data source confirmed.

---

### 1.2.12 Time Filter Options

|   |   |
|---|---|
|Filter|Definition|
|Today|Current calendar day (IST midnight to midnight)|
|Yesterday|Previous calendar day|
|Tomorrow|Next calendar day|
|MTD|Month-to-Date: 1st of current month through today (default filter)|
|Last Month|Previous full calendar month|
|Any Month|Month picker — select any past or future month|
|Next 7 Days|Today + next 6 calendar days|
|Next 30 Days|Today + next 29 calendar days|
|Custom Date|Any single date selected by user|
|Custom Range|Any date range (start date to end date)|

---

### 1.2.13 Comparison Terms

|   |   |
|---|---|
|Term|Definition|
|MoM|Month-over-Month comparison against previous equivalent period|
|Delta|Absolute or percentage change from comparison period|
|Trend|Direction indicator: ↑ improving, ↓ declining, → stable|
|pp|Percentage points — absolute change in % metrics (72% → 78% = +6pp)|

---

## 1.3 System Invariants

Invariants are rules that MUST always be true. Violation = system bug.

### Population Invariants

|   |   |   |   |
|---|---|---|---|
|#|Invariant|Formula|Notes|
|INV-1|Total Tenants composition|Total Tenants = Active + Under Notice|Only physically present tenants|
|INV-2|Notice type composition|Under Notice = Voluntary Notice + Eviction Notice|Every notice has exactly one type|
|INV-3|Move-out composition|Move-outs (period) = Voluntary Exits + Completed Evictions|Every departure has exactly one type|
|INV-4|Net change formula|Net Change = Move-ins − Move-outs|Must hold for any period|

### Distribution Invariants

|   |   |   |   |   |
|---|---|---|---|---|
|#|Invariant|Formula|Population|Notes|
|INV-5|Tenure distribution|Σ(Tenure Buckets) = Active Tenants|Active only|Under Notice excluded (already decided to leave)|
|INV-6|Tenant type distribution|Σ(Tenant Types) = Total Tenants|Total Tenants|Every tenant has exactly one type|
|INV-7|Gender distribution|Male + Female + Other + Unspecified = Total Tenants|Total Tenants|Includes unspecified as category|
|INV-8|Eviction pipeline|Σ(Eviction Progress Buckets) = Total Active Evictions|Active Evictions|Every active eviction in exactly one bucket|

### Compliance Invariants

|   |   |   |   |   |
|---|---|---|---|---|
|#|Invariant|Formula|Population|Notes|
|INV-9|Agreement status|Valid (incl. Expiring) + Expired + None = Total Tenants|Total Tenants|Every present tenant has exactly one agreement state|
|INV-10|PV status|(Verified + Submitted + Not Started) = Total Tenants|Total Tenants|Every present tenant has exactly one PV state|
|INV-11|eKYC status|Complete + Incomplete = Total Tenants|Total Tenants|Every present tenant has exactly one eKYC state|

### Funnel Invariants

|   |   |   |   |
|---|---|---|---|
|#|Invariant|Formula|Notes|
|INV-12|Booking funnel|Total Bookings (period) = Pending + Approved + Cancelled + Converted|All bookings in exactly one current status|
|INV-13|Conversion path|Converted can only come from Approved|Cannot skip approval step|
|INV-14|Cancellation constraint|Cancelled + Converted ≤ Total|Can't cancel or convert more than total|

### Renewal Invariants

|   |   |   |   |
|---|---|---|---|
|#|Invariant|Formula|Notes|
|INV-15|Renewal pipeline|Renewal Due = 0–30 day bucket + 31–60 day bucket|Due pipeline covers next 60 days|
|INV-16|Renewal rate bounds|0% ≤ Renewal Rate|No upper cap — can exceed 100% if backdated renewals count in period|

### Consistency Invariants

|   |   |   |   |
|---|---|---|---|
|#|Invariant|Formula|Notes|
|INV-17|Drill-down count match|Metric count on screen = Record count in drill-down list|What you tap = what you see|
|INV-18|Filter preservation|Drill-down inherits: property filter + time filter + segment filter|No dropped context|
|INV-19|Occupancy consistency|Occupancy % on this screen = Occupancy % on Homescreen|Same formula, same data source|

---

## 1.4 Key Formulas

### 1.4.1 Population Formulas

|   |   |   |
|---|---|---|
|Metric|Formula|Population Base|
|Total Tenants|Active + Under Notice|—|
|Occupancy Rate|(Total Tenants ÷ Total Sellable Beds) × 100|Matches Homescreen definition|

Occupancy Note: Total Sellable Beds = Configured Beds − Disabled Beds. Under Notice tenants are included because they physically occupy the bed until move-out. This matches Homescreen FR 2.6.

---

### 1.4.2 Retention Metrics

|   |   |   |
|---|---|---|
|Metric|Formula|Example|
|Churn Rate|(Move-outs in Period ÷ Avg Total Tenants in Period) × 100|27 move-outs ÷ avg 410 = 6.6%|
|90-Day Retention Rate|(Tenants present who moved in 90+ days ago ÷ Total move-ins from 90+ days ago) × 100|29 still present ÷ 35 moved in = 82.9%|
|Renewal Rate|(Renewals Completed in Period ÷ Agreements Expired in Period) × 100|18 renewed ÷ 25 expired = 72%|
|Early Termination Rate|(Move-outs with tenure < 90 days ÷ Total Move-outs in Period) × 100|8 early exits ÷ 27 total = 29.6%|

Churn Rate Detail:

- Avg Total Tenants = (Total Tenants at Period Start + Total Tenants at Period End) ÷ 2
    
- Move-outs = Voluntary Exits + Completed Evictions
    
- Lower is better
    

Churn Rate Worked Example:

- Jan 1: 408 Total Tenants (390 Active + 18 Under Notice)
    
- Jan 31: 412 Total Tenants (396 Active + 16 Under Notice)
    
- Move-outs in January: 27 (22 voluntary + 5 evictions)
    
- Avg Total Tenants = (408 + 412) ÷ 2 = 410
    
- Churn Rate = (27 ÷ 410) × 100 = 6.6%
    

90-Day Retention Detail:

- "Tenants present" = Status is Active OR Under Notice (haven't left yet)
    
- Moved Out or Evicted = not retained
    
- Under Notice counts as "present" because notice can be withdrawn and they haven't actually left
    

90-Day Retention Worked Example:

- Move-ins from 90+ days ago (October): 35 tenants moved in
    
- Of those 35 today: 26 Active, 3 Under Notice, 4 Moved Out, 2 Evicted
    
- Present = 26 + 3 = 29
    
- 90-Day Retention = (29 ÷ 35) × 100 = 82.9%
    

---

### 1.4.3 Acquisition Metrics

|   |   |   |
|---|---|---|
|Metric|Formula|Target|
|Booking Conversion Rate|(Converted ÷ Total Bookings in Period) × 100|—|
|Approval Rate|(Approved ÷ Total Bookings in Period) × 100|—|
|Move-in Conversion Rate|(Converted ÷ Approved in Period) × 100|>80% (Red <70%)|
|Cancellation Rate|(Cancelled ÷ Total Bookings in Period) × 100|<10% (Red >20%)|

---

### 1.4.4 Compliance Metrics

|   |   |   |   |
|---|---|---|---|
|Metric|Formula|Population|Target|
|PV Compliance Rate|(Verified + Submitted) ÷ Total Tenants × 100|All present tenants|≥95%|
|PV Overdue Count|Count where (Today − Move-in Date) > PV Deadline AND PV Status = Not Started|Total Tenants|0|
|Agreement Coverage|Valid Agreements (incl. Expiring) ÷ Total Tenants × 100|All present tenants|≥95%|
|Agreement Gap Count|Expired + None|Total Tenants|0|
|eKYC Rate|Complete ÷ Total Tenants × 100|All present tenants|≥95%|

PV Overdue Worked Example:

- Tenant moved in: Jan 10  
      
    
- Today: Jan 28  
      
    
- Days since move-in: 18 days  
      
    
- PV Deadline (property setting): 14 days  
      
    
- PV Status: Not Started  
      
    
- Result: Overdue (18 > 14 and status is Not Started)  
      
    
- If PV Status were "Submitted": Not Overdue (documents submitted, awaiting police)  
      
    

---

### 1.4.5 Revenue Context Metrics

|   |   |   |
|---|---|---|
|Metric|Formula|Dependency|
|Revenue at Risk|Σ(Monthly Rent) for all tenants where Status = Under Notice|Requires Monthly Rent per tenant (Open Item #6)|
|Dues at Risk|Σ(Outstanding Dues) for tenants where Status = Under Notice AND Outstanding Dues > 0|Available from Financial Service|
|Defaulter Count (Exiting)|Count where Status = Under Notice AND Outstanding Dues > 0|Available|

Revenue at Risk Fallback: If Monthly Rent is unavailable per tenant, display: "15 under notice" (count only). Add ₹ value once data source is confirmed per Open Item #6.

---

### 1.4.6 Net Change & Avg Tenure

|   |   |   |
|---|---|---|
|Metric|Formula|Notes|
|Net Change|Move-ins (period) − Move-outs (period)|Positive = portfolio growing|
|Avg Tenure|Σ(Today − Move-in Date for each Active Tenant) ÷ Active Tenant Count|In months. Active only (Under Notice excluded — their tenure is ending). Displayed in Overview or Composition (mapped in Part 3).|

---

## 1.5 Time Filter Behavior

### 1.5.1 Metric Type × Time Filter Matrix

|   |   |   |
|---|---|---|
|Metric Type|Time Filter Selected|Behavior|
|Snapshot|Any filter|Always shows CURRENT live state|
|Flow|Today|Today's activity|
|Flow|Yesterday|Yesterday's activity|
|Flow|MTD (default)|1st of month through today|
|Flow|Last Month|Previous full month|
|Flow|Any Month (past)|Full month's activity|
|Flow|Any Month (current)|Behaves as MTD|
|Flow|Any Month (future)|Scheduled/approved events only, labeled "Forecast"|
|Flow|Custom Date (past)|That date's activity|
|Flow|Custom Range|Range activity|
|Flow|Tomorrow / Next 7 / Next 30|Scheduled/approved only, labeled "Scheduled"|

---

### 1.5.2 Comparison Period Logic

|   |   |   |
|---|---|---|
|Selected Filter|Compares Against|Example|
|Today|Yesterday|Feb 3 vs Feb 2|
|Yesterday|Day before yesterday|Feb 2 vs Feb 1|
|MTD|Same # of days in previous month|Feb 1–3 vs Jan 1–3|
|Last Month|Month before|January vs December|
|Any Month (past)|Month before selected|September vs August|
|Any Month (current)|Same as MTD comparison|—|
|Custom Date|Same date previous month|Jan 15 vs Dec 15|
|Custom Range|Same duration immediately prior|Jan 10–20 (11 days) vs Dec 30–Jan 9 (11 days)|
|Tomorrow / Future filters|Today or N/A|No comparison shown|

---

### 1.5.3 Forward-Looking Filter Rules

When Tomorrow, Next 7 Days, Next 30 Days, or future Any Month selected:

- Show only approved/confirmed future events
    
- Pending events excluded
    
- Label counts as "Scheduled" or "Forecast"
    
- Action callouts HIDDEN (can't act on future events that haven't happened)
    
- Snapshot metrics still show current live state
    
- MoM comparison not shown
    

---

### 1.5.4 Historical Filter Rules

When filter is fully in past (Last Month, past Any Month, past Custom Date/Range):

- Action callouts HIDDEN (can't send reminders for past period)
    
- "Send Reminders" buttons hidden
    
- Pipeline cards (Renewal Due, Eviction Pipeline) always show CURRENT state with note: "Current as of today"
    
- Reason: Renewal Due is a forward-looking pipeline. Showing "who was due last month" is misleading — they may have already renewed or left.
    

---

## 1.6 Threshold Definitions

### 1.6.1 Color-Coded Status Thresholds

|   |   |   |   |
|---|---|---|---|
|Metric|Green (Healthy)|Orange (Warning)|Red (Critical)|
|Occupancy|≥85%|70–84%|<70%|
|Renewal Rate|≥75%|50–74%|<50%|
|90-Day Retention|≥85%|70–84%|<70%|
|Churn Rate|<6%|6–10%|>10%|
|PV Compliance|≥95%|80–94%|<80%|
|Agreement Coverage|≥95%|80–94%|<80%|
|eKYC Rate|≥95%|80–94%|<80%|
|Cancellation Rate|<10%|10–20%|>20%|
|Move-in Conversion|≥80%|70–79%|<70%|

---

### 1.6.2 Callout Trigger Conditions

|   |   |   |
|---|---|---|
|Condition|Callout Text|Severity|
|Any PV overdue >14 days|"X PV overdue (>14 days)"|Red|
|Any PV overdue >30 days|"X PV critical (>30 days)"|Red, escalated|
|Agreements expiring ≤7 days, renewal not initiated|"X agreements expire this week — Y no renewal started"|Orange|
|Active evictions in 0–7 day bucket|"X evictions need immediate action"|Red|
|Defaulters under notice (any)|"X tenants leaving with ₹Y dues"|Orange|
|Renewal rate <50% for current period|"Renewal rate low — X% this month"|Orange|

Callout Location: Callouts appear in the Overview section at the top of the screen, immediately visible on page load. Maximum 3 shown; "View all" if more.

---

## 1.7 Data Freshness Rules

|   |   |   |
|---|---|---|
|Data Category|Refresh Behavior|Stale Tolerance|
|Snapshot metrics|Real-time on screen open + pull-to-refresh|None — must be current|
|Flow metrics|Cached with 5-minute TTL|Show "Last updated X mins ago" if cache age > 1 min|
|Drill-down lists|Real-time on tap|None — must be current|
|Trends/Charts|Cached with 15-minute TTL|Acceptable — historical data|
|Compliance statuses|Real-time on screen open|None — legal risk data|

---

## Part 1 Verification

### Internal Consistency Checks

|   |   |
|---|---|
|Check|Status|
|Mental model explains snapshot vs flow|✅|
|Mental model explains lifecycle lens|✅|
|Total Tenants population defined clearly|✅|
|Tenant statuses complete and non-overlapping|✅|
|Notice types with time direction difference|✅|
|Booking statuses with transitions|✅|
|Agreement "Expiring" clarified as subset of Valid|✅|
|Compliance items split: Legal vs Operational|✅|
|PV Overdue logic with status consideration|✅|
|Renewal Due vs Renewal Rate distinction|✅|
|Active Eviction defined|✅|
|Tenure buckets — population clarified (Active only)|✅|
|Defaulter threshold documented with Part 4 flag|✅|
|Revenue at Risk fallback documented|✅|
|All 10 time filter options listed (incl. Any Month)|✅|
|Forward-looking filter rules|✅|
|Historical filter rules|✅|
|Churn denominator uses Total Tenants|✅|
|Occupancy includes Under Notice|✅|
|Compliance denominators use Total Tenants|✅|
|90-Day Retention includes Under Notice as "present"|✅|
|All formulas have worked examples|✅|
|INV-9 consistent with Expiring definition|✅|
|INV-19 added for Homescreen consistency|✅|
|Avg Tenure mapped (noted for Part 3)|✅|
|Callout location specified|✅|
|All 19 invariants non-conflicting|✅|
|No undefined terms from Part 0|✅|
|No open loops|✅|

### Part 0 ↔ Part 1 Reconciliation

|   |   |   |
|---|---|---|
|Part 0 Claim|Part 1 Coverage|Status|
|Total Tenants = Active + Under Notice|INV-1, Section 1.2.1|✅|
|Notices split by voluntary/eviction|INV-2, Section 1.2.3|✅|
|Revenue at Risk = Σ rent for notice tenants|Formula 1.4.5 with fallback|✅|
|Churn Rate defined|Formula 1.4.2 with worked example|✅|
|90-Day Retention defined|Formula 1.4.2 with worked example|✅|
|Renewal Rate defined|Formula 1.4.2|✅|
|PV overdue with configurable deadline|Section 1.2.6, Formula 1.4.4|✅|
|₹ context vs ₹ transactions|Section 1.2.11, explicit boundary|✅|
|Snapshot vs Flow metrics|Section 1.1, Matrix 1.5.1|✅|
|MoM comparison|Section 1.5.2|✅|
|Drill-down count consistency|INV-17|✅|
|Occupancy includes Under Notice|Formula 1.4.1, matches Homescreen|✅|
|Success metric: "unactioned exits"|Requires activity logging (Part 0 Open Item #8)|✅ Tracked|
|Success metric: "defaulters flagged before exit"|Defaulter overlay on notices (Section 1.2.11)|✅|
|Sunita's PV deadline = 14 days configurable|Section 1.2.6, PV Deadline configurable per property|✅|

---

END OF PART 1: FOUNDATIONS v1.1

  
**