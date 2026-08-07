---
title: DA-01 Spec — Dues Detailed Analytics
date: '2026-05-01'
tags:
  - rentok
  - prd
  - spec
  - homescreen
  - financial-insights
  - detailed-analytics
  - dues
aliases:
  - DA-01 Spec
  - Dues Detailed Analytics Spec
  - DA-01 Clean Spec
status: SUPERSEDED — historical reference only
version: Clean spec (May 1 vintage)
superseded_by: DA-01 Dues Detailed Analytics (V3.3) + DA-01 Build Sheet (V2)
figma: KgBQXiT7r7oGrcqZHCWxyU
section: Dues
---
> [!WARNING] SUPERSEDED — May 1 vintage
> This doc predates the V3.3 PRD and V2 Build Sheet (May 8). Kept for historical reference only.
> **Active canonical "why" doc:** [[DA-01 Dues Detailed Analytics]] (V3.3)
> **Active engineering Build Sheet:** [[DA-01 Build Sheet]] (V2, 9-column ticket-ready)
> **Field reference:** [[_Ground Truth Field Map]]
> Do NOT use this doc for current implementation. PM may archive to `_archive/` folder at their discretion.

> [!INFO] Source of Truth (May 1 vintage)
> Local spec: `/Users/eazypg/RentOk Manager Homescreen/spec/DA-01-spec.md`
> **Companion docs:** [[DA-01 Engineering]] · [[DA-01 Codebase Feasibility]] · [[DA-01 V2 Roadmap]]
> **Original v3.1 (untouched):** [[DA-01 Dues Detailed Analytics]]
> **Design:** Figma `KgBQXiT7r7oGrcqZHCWxyU` → Financials → Dues container (`7837:99662`)
> **Supersedes:** Lark "Dues (Live) · Financial Detailed Insights v1.1"

# DA-01 — Dues Detailed Analytics

> **Status:** V1 — locked. **Version:** Clean spec (supersedes [[DA-01 Dues Detailed Analytics|v3.1]] in style; semantics identical). **Last updated:** 2026-05-01.

---

## At a Glance

**The screen answers:** *"Kitna paisa atak raha hai, kidhar atak raha hai, aur aaj kisko call karna hai?"* — How much money is stuck, where is it stuck, and who do I call today?

**The central mental model:** Total Dues is always live. The date filter doesn't change the total — it reslices the urgency bar's buckets and updates the period-activity sections (Performance, Added By). Everything else is live always.

### Personas served

| Persona | Their question | Where served |
|---------|----------------|--------------|
| **Rajesh** — single-property owner-admin | "Who do I call today?" | Primary persona. Hero, urgency, action card, top defaulters built around his morning workflow. |
| **Priya** — multi-property owner | "Which property needs attention?" | Property Breakdown section ranks them; multi-property worklist groups by property. |
| **Amit** — new manager, sparse data | "What should I do first?" | Setting-Up State (§9) handles the first 30 days. |
| **Meena** — collections coordinator | "What can I act on?" | Permissions hide surfaces she lacks access to (EC-10). |

Indirect: **Accountant** (FY filter + Excel pivot for exact past-timestamp balances). **Owner-investor** (V2.2 WhatsApp digest only — not served by V1).

### Six rules

1. **Live by default — and live always.** Total Dues, Population, Property Breakdown, Action Card, By Category, Defaulters, and the Aging chart show current outstanding regardless of date filter. The filter only reslices Urgency Bar buckets and updates Performance + Added By.
2. **The numbers must match.** A count shown on this screen must equal the count in the drill-down worklist exactly.
3. **Red means action now.** A thick red urgency bar segment communicates urgency before any number is read.
4. **Old tenant dues are not forgotten dues.** Moved-out tenants stay visible with deposit context.
5. **Deep analytics are on demand, not on load.** Advanced Insights accordions are collapsed by default.
6. **Time filter is global, exceptions are explicit.** All drill-downs inherit the date filter unless explicitly listed as live.

### Reconciliation summary

Every aggregate sums to the same Total Dues. Drill-down counts match parent counts exactly. Cross-screen, the homescreen Dues tile equals the detailed hero. **Formulas:** see [[DA-01 Engineering]].

---

## A Morning with Rajesh

It's the 5th of the month. Rajesh manages a 40-bed PG in Bangalore. He opens the app with his chai.

The Dues screen opens. He sees ₹16L — bigger than last month. Red chip: ▲8% vs last month. Not good.

He scans the urgency bar. The red segment is thick — ₹6L already overdue. He taps it.

A list opens: 32 bills, oldest first. He sees Karan Mehta — ₹12,000 overdue, 18 days, no payments since 12 March. He taps Call.

Done in 90 seconds. He didn't open a spreadsheet, didn't ask his accountant. He saw the problem, found the person, made the call.

That is the experience this screen must deliver.

---

## Scope

This screen is for **money owed TO the property — unpaid bills only**.

| Included | Excluded |
|----------|----------|
| Rent invoices · Security deposit invoices · Food/Mess · Electricity · Maintenance · Late fees as line items within invoices | Collections (separate tab) · Expenses · Deposits Held · Refunds · Discounts · Late Fee dedicated analytics (Phase 2) |

---

## How Rajesh Gets Here

`Homescreen → Overview → "View Detailed Analytics" → Financial Insights → Dues tab (selected by default)`

---

## What Rajesh Sees

### 1. Header

**"Dues (Live)"** with an info icon. The label doesn't change with the date filter — Total Dues is always live.

### 2. Total Dues + MoM Chip

Large number (e.g., ₹16L) with subtitle "(120 Invoices)". Multi-property: "(120 Invoices · across 3 properties)".

Above the number, a comparison chip with **inverted semantics**:
- ▼12% vs last month (green) — dues decreased, good
- ▲8% vs last month (red) — dues increased, action needed

The chip compares today's outstanding to the same day of the previous month. Independent of the date filter. Hidden when no comparison data exists (first month of operation).

### 3. Urgency Bar

Stacked horizontal bar. **Reslices based on date filter — total stays the same.**

**Today mode** (default — filter is "Today" only): 4 buckets vs today.

| Segment | Color | Covers |
|---------|-------|--------|
| Already Due | Red | Bills past due date |
| Due Today | Amber | Bills due today |
| Due this week | Blue | Bills due Tue–Sun of this calendar week |
| Due Later | Green | Bills due beyond this week |

Calendar week is Mon–Sun, NOT rolling 7 days. PG operators plan in weekly cycles.

**Range mode** (any other period selected): 3 buckets vs the selected period.

| Segment | Covers |
|---------|--------|
| Carried Forward | Currently outstanding bills with due date < period start |
| Due in Period | Currently outstanding bills with due date in period |
| Due After Period | Currently outstanding bills with due date > period end |

This is **not** historical reconstruction. The hero stays at the same total; the bar just reslices to show "of the ₹16L stuck right now, ₹4L is old debt, ₹10L came from this period, ₹2L is future-dated."

For the accountant on "This Fiscal Year": Carried Forward = pre-FY old debt, Due in Period = FY's contribution, Due After Period = post-FY-dated bills. FY analysis without leaving the dashboard.

Bar widths use amounts, not counts. Min segment width 4px so small amounts stay visible. Tapping a segment or its label opens the worklist.

### 4. Population Breakdown

Always live. Three rows:

- **Active Tenants** — currently checked-in tenants
- **Booking** — confirmed future move-ins
- **Old Tenants** — moved out but still owe

Old Tenants row shows deposit context inline:
- "₹8,000 deposit · Covered ✓" — relax, deposit covers it
- "₹3,000 deposit · ₹2,000 shortfall" — manual recovery needed
- "No deposit held" — full exposure, follow up immediately

This context only appears on Old Tenants. It's the #1 silent leakage source in PG ops.

### 5. Property Breakdown (multi-property only)

Visible when the account has multiple properties AND the user is viewing more than one. List of properties: name + outstanding amount + share % + mini urgency bar. Sorted biggest problem first. Tap → worklist scoped to that property.

For Priya: *Sunshine PG ₹8L (50%) · Green Hostel ₹5L (31%) · Blue Residence ₹3L (19%).* Always live.

### 6. Action Card

A dark, high-contrast card: `32 Rent & Bills Overdue · View All >`. Always live. Shown only when there are overdue bills. Tap "View All" → overdue worklist sorted most-overdue-first. Same target as the Already Due segment in today mode.

The card is dark because urgency deserves visual weight.

### 7. Advanced Insights

A section divider separates this from the always-on view above. Four accordions, all collapsed by default.

**7A. Performance** — *"Am I billing correctly and collecting efficiently?"* Period-sensitive (respects date filter).

| Metric                    | What it means                                                          |
| ------------------------- | ---------------------------------------------------------------------- |
| Invoices Due ₹22L         | Total amount of bills with due date in selected period (paid + unpaid) |
| Current Dues ₹16L         | Of those bills, how much is still unpaid now                           |
| Collection Efficiency 27% | (Invoices Due − Current Dues) ÷ Invoices Due                           |
|                           |                                                                        |

Efficiency: Green >70% · Amber 40–70% · Red <40%.

Below: callout "₹6L collected against bills due Apr 1 → Apr 30." All metrics use `due_date` (when bill was due), not `created_at` (when bill was raised).

**7B. By Category** — *"Is this a rent problem or utilities problem?"* Always live. Ranked list: Rent · Electricity · Food · Security Deposit · Maintenance · Others. Sorted by outstanding amount. Tap → worklist filtered to that category.

**7C. By Defaulters** — *"Which tenants are becoming a write-off risk?"* Always live (aging is always counted from today).

Aging breakdown: 1–7d (95% recoverable) · 8–15d (85%) · 16–30d (60%) · 31–60d (30%) · 60+d (<15%). Each shows tenant count + outstanding. Tap → worklist filtered to that aging window.

Top Defaulters: top 5 tenants by outstanding. Each row shows name, room, outstanding, days overdue, last payment date. Tap a tenant → Tenant Dues Detail. *(Repeat badge — flags chronic defaulters overdue in 3+ of last 6 months — deferred to V2.0 P0; see [[DA-01 V2 Roadmap]]. V1 ships without the badge; layout reserves space for it.)*

**7D. Added By** — *"Who on my team is creating bills?"* Period-sensitive. Ranked list of bill creators (team members + RentOk system row for auto-generated). Tap → worklist filtered to that creator AND due in selected period.

### 8. Defaulter Analysis Chart

Always visible below all accordions. Always live. Stacked bar chart, four bars (0–7d, 8–15d, 16–30d, 31+d). Blue = collected, light = still outstanding. Health scan at a glance: tall left bars = healthy, tall right bars = problem.

Figma example: 84/52/88/88 — the 16–30d and 31+d buckets matching 0–7d is a warning sign.

---

## 9. Setting-Up State (for new or sparse accounts)

**The persona:** Amit signed up last week. 5 of 30 beds have tenants. 3 bills exist (all auto-generated). He opens this screen — hero says ₹15K, urgency bar mostly empty, Performance accordion shows weird zeros. **The screen FEELS broken to him.** Without intervention, he closes it and assumes analytics don't work.

### When this state applies

ANY of:
- Account age < 30 days
- Total invoices < 10
- < 30% of beds have an active tenant
- < 30 days of billing history

### What changes

**Persistent banner** above the date filter:

> "Setting up — your analytics will get richer as you add tenants and bills. **5 of 30 beds have tenants · 3 bills created so far** · [Quick setup checklist →]"

Dismissable per session. Reappears next session until criteria are met.

**Per-component behavior:**

| Component | Behavior |
|-----------|----------|
| Hero | Real number (could be ₹0) |
| MoM chip | Hidden — no comparison baseline |
| Urgency bar | Real data; empty segments blank, not "loading" |
| Population, Action Card, By Category, Defaulters, Aging | Real data, even sparse |
| Property Breakdown | Hidden if only 1 property |
| Performance | Shows with overlay: "Need 30 days of bills to show meaningful efficiency. You have X days." |
| Added By | If everything is auto-generated, banner: "Most bills are auto-generated. [Add manual bills →]" |

### What we deliberately do NOT do

- Show fake/sample data anywhere
- Hide entire sections
- Force a guided tour
- Block drill-downs (Rule 2 still applies)

The "Quick setup checklist" link routes outside DA-01 to the property setup flow. We reference it; we don't duplicate it. Banner self-dismisses permanently when all setup criteria are met AND 30 days have passed AND ≥10 invoices exist.

---

## Drilling Down

Every tap leads to either the **Dues Worklist** (filtered list) or **Tenant Dues Detail** (single tenant).

### Drill-down matrix

| Source | List filter | List header |
|--------|-------------|-------------|
| Already Due (today mode) | `due_date < today` | "Already Due · 32 Bills · ₹6L" |
| Due Today (today mode) | `due_date = today` | "Due Today · 18 Bills · ₹4L" |
| Due this week (today mode) | `due_date in [Mon..Sun] of this week` | "Due This Week · 28 Bills · ₹3.8L" |
| Due Later (today mode) | `due_date > end of this week` | "Due Later · 42 Bills · ₹2.2L" |
| Carried Forward (range mode) | `due_date < period_start` | "[Period] · Carried Forward · 12 Bills · ₹4L" |
| Due in Period (range mode) | `due_date in [period_start, period_end]` | "[Period] · Due in Period · 60 Bills · ₹10L" |
| Due After Period (range mode) | `due_date > period_end` | "[Period] · Due After Period · 18 Bills · ₹2L" |
| Active Tenants row | `tenant.status = 1` | "Active Tenants · 85 Tenants · ₹4L" |
| Booking row | `tenant.status = 2` | "Bookings · 12 Tenants · ₹2L" |
| Old Tenants row | `tenant.status = 0` | "Old Tenants · 8 Tenants · ₹0.5L" |
| Property Breakdown row | `property = X` | "[Property Name] · 45 Bills · ₹8L" |
| Action Card "View All" | `due_date < today` | "Past Due · 32 Bills · ₹6L" |
| Category row | `due_type = X` | "Rent · 24 Bills · ₹8L" |
| Aging bucket | computed `today − due_date` in window | "1–7 Days Overdue · 84 Tenants · ₹3L" |
| Top Defaulter row | (opens Tenant Dues Detail, not worklist) | — |
| "View All Defaulters" | (no extra filter) | "All Defaulters · 120 Tenants · ₹16L" |
| Performance: Invoices Due | `due_date in [start, end]` | "[Period] Invoices Due · 80 Bills · ₹22L" |
| Performance: Current Dues | `due_date in [start, end]` AND outstanding > 0 | "[Period] Current Dues · 60 Bills · ₹16L" |
| Added By: team member | `created_by = X` AND `due_date in [start, end]` | "[Period] · Bills by Riya · 14 Bills · ₹2.1L" |

Default action surfaced varies by source — Send Reminder for active/overdue contexts, Adjust from Deposit for Old Tenants, View Bill Detail for future-dated.

### Time-context propagation rules

1. **Default:** drill-down inherits the global date filter as `start_date`/`end_date` on `due_date`.
2. **Always-live elements** (Population, Property Breakdown, Action Card, By Category, Aging buckets, Top Defaulters, View All Defaulters): drill-down sends NO date filter — list shows current state regardless of dashboard period.
3. **Mode-aware urgency buckets:** drill-down sends a custom `due_date` range derived from period boundaries, not the global filter directly.
4. **Period-sensitive accordions** (Performance, Added By): drill-down inherits the global filter on `due_date`.

If the operator changes a filter on the worklist itself, that change is local to the worklist and does NOT propagate back to the dashboard.

### Worklist behavior

- **Header confirms context.** Single-period: `[Period] · [Cohort] · X Bills · ₹Y`. Multi-property aggregated: `All Properties · [Cohort] · X Bills · ₹Y`.
- **Filter chips** at top show what's pre-applied (e.g., `[Due Date: Apr 1 – Apr 30 ✕] [Category: Rent ✕]`). Tap ✕ to remove.
- **Available filters:** Urgency, Population, Category, Property (multi-property only), Aging window, Amount range, Tenant search, Payment status (Phase 2).
- **Per-row:** Tenant name, room, outstanding (prominent), due date (or "X days overdue"), invoice type badge. Multi-property mode: also shows property name. *(Repeat badge — V2.0 deferred.)*
- **Per-row actions:** Send Reminder, Record Payment, Call Tenant, View Bill Detail, Adjust from Deposit (Old Tenants only).
- **Bulk actions:** Send Bulk Reminder (multi-select), Export to Excel.
- **Multi-property:** Rows grouped by property (section headers like `Sunshine PG · 12 Bills · ₹3.5L`). Property filter chip narrows. Single-property tap from Property Breakdown bypasses grouping.
- **Refresh** on: load, pull-to-refresh, after any in-list mutation, after back from Tenant Detail.
- **Back-nav:** Tenant Detail → back returns to same worklist with same filters and scroll. Worklist → back returns to dashboard with same filter and accordion state.
- **Deep links:** `rentok://dues?filter=overdue&property=PG123` — pushes/WhatsApp can route here directly.
- **Pagination:** 20/page, infinite scroll.

### Tenant Dues Detail

Reached from Top Defaulters tap or worklist row tap.

Header: tenant name, room (or "was in [room]" for old tenants), total outstanding, deposit held, days since last payment.

Four sections: Open Bills (oldest due first) · Payment History (last 10) · Reminder History (last 5) · Late Fee History.

Actions: Call · Send Reminder · Record Payment · View Full Profile · Adjust from Deposit (moved-out + deposit > 0).

---

## Time Filter Behavior

The filter at the top of Financial Insights is **global**. It flows into every drill-down by default. Rule 6 codifies this. Below is the complete exception list:

| Component | Date filter behavior |
|-----------|----------------------|
| Hero (Total Dues) | Always live |
| MoM chip | Always compares right-now to same-day-last-month |
| Urgency bar | Reslices: today mode = 4 buckets, range mode = 3 |
| Population | Always live |
| Property Breakdown | Always live |
| Action Card | Always live |
| By Category | Always live |
| By Defaulters / Aging | Always live |
| Top Defaulters | Always live |
| Defaulter Analysis Chart | Always live |
| Performance | Respects filter |
| Added By | Respects filter |

When Rajesh picks "Last Month": hero, urgency total, population, property breakdown, action card, categories, defaulters, aging chart all stay current. Urgency bar reslices to 3 range buckets. Performance and Added By show March activity. Banner: *"Live values reflect right now. Period values show March activity."*

For exact past-timestamp balances (rare, formal accounting), accountants pivot Invoice + Payment exports in Excel/Tally. Frozen-period semantics for monthly book-closing arrives in V2.1 (see [[DA-01 V2 Roadmap]] P1-#16).

---

## Edge Cases

| ID | Scenario | Behavior |
|----|----------|----------|
| EC-01 | Zero dues | Hero ₹0. Urgency bar hidden. *"All clear! No pending dues across your property."* |
| EC-02 | No invoices ever | Hero ₹0. *"No bills created yet. Start by adding rent dues."* with "Add Dues" button (if permitted). |
| EC-03 | Invoice partly paid | ₹4K shown of ₹10K original. Counts as 1 invoice. Worklist row: "₹4,000 outstanding (of ₹10,000)." |
| EC-04 | Bill due today | Goes to "Due Today" bucket. Disappears from Dues after payment + refresh. |
| EC-05 | Tenant moved out with unpaid bills | Goes to Old Tenants. Deposit context inline. "(Moved out)" badge in worklist. Adjust from Deposit available. |
| EC-06 | Multi-property view (Priya) | Aggregate across selected properties. Property Breakdown section appears. Worklist groups by property. |
| EC-07 | Single-property tap from Property Breakdown | Worklist scopes to one property, bypasses grouping. Filter chip `[Property: Sunshine PG ✕]`. |
| EC-08 | Large property (500+ unpaid invoices) | Worklist paginates. Top Defaulters still top 5. K/L/Cr formatting. |
| EC-09 | Stale data between screens | Detail always fetches fresh on load. Pull-to-refresh always available. |
| EC-10 | Meena (collections coordinator role) | Section visible. Actions (Record Payment, Adjust from Deposit) hidden if she lacks permission — not grayed out. |
| EC-11 | Cancelled / voided invoices | Never shown. |
| EC-12 | One tenant, multiple overdue invoices | Vikram Joshi: ₹10K rent + ₹2K electricity + ₹3K food. Worklist: 3 rows. Top Defaulters: 1 row at ₹15K. Aging: each invoice ages independently. *(Tenant name distinct from operator persona Amit.)* |
| EC-13 | Range-mode period with zero data | Urgency bar shows "No bills due in this period" placeholder. Live elements still show today's state. |
| EC-14 | Period crosses fiscal year boundary | Buckets work normally. Carried Forward includes ALL pre-period bills regardless of FY. |

---

## Microcopy

### Section header
**"Dues (Live)"** — does not change with date filter.

### Hero tooltip
> "Total pending bills across selected properties, as of right now. Updates as payments are received or new bills are created."

### MoM chip tooltip
> "Compared to same day last month. ▼ means dues decreased (good). ▲ means dues increased (action needed)."

### Empty states

| When | Message | CTA |
|------|---------|-----|
| No invoices ever | "No bills created yet. Start by adding rent dues for your tenants." | "Add Dues" (if permitted) |
| All bills paid | "All clear! No pending dues across your property." | — |
| Worklist filter returns nothing | "No bills match your current filters." | "Clear Filters" |
| Range mode, no bills due | "No bills were due in this period." | — |

### Error states

| When | Message | Recovery |
|------|---------|----------|
| Network failure | "Couldn't load dues data. Check your connection and try again." | "Retry" |
| Accordion fails | "Couldn't load this section." | "Retry" on that accordion |
| Slow load on large dataset | "This is taking longer than usual for large properties." | Auto-retry once, then retry button |
| Drill-down count mismatch | "Numbers couldn't be loaded correctly. Tap to refresh." | "Retry" |

### Banners

- Range mode: *"Live values reflect right now. Period values show [period] activity."*
- Setting-up: *"Setting up — your analytics will get richer as you add tenants and bills. X of Y beds have tenants · Z bills created so far · [Quick setup checklist →]"*

### Data freshness
If data > 5 minutes old: *"Last updated X minutes ago"* (subtle, below hero).

### Section-specific tooltips

- **Already Due:** "Bills past their due date. Need immediate attention."
- **Due Today:** "Bills due today. Send reminders or follow up now."
- **Due this week:** "Bills due Tuesday through Sunday of this calendar week."
- **Due Later:** "Bills due beyond this week."
- **Carried Forward:** "Currently unpaid bills with due dates before [period]. Old debt."
- **Due in Period:** "Currently unpaid bills due in [period]."
- **Due After Period:** "Currently unpaid bills due after [period]. Future-dated."
- **Aging:** "How long bills have been overdue. Focus on the 8–30 day range — still recoverable, but getting urgent."
- **Repeat badge** *(V2.0 — deferred)*: "This tenant has been overdue in 3 or more of the last 6 months."
- **Others category:** "Tap to expand and see breakdown of other categories like Security Deposit, Maintenance, etc."
- **RentOk row:** "Bills auto-generated by RentOk system (recurring rent, scheduled charges, etc.)"

---

## Decisions Log

Where this spec differs from Lark v1.1 / Figma:

| Lark / Figma | This spec | Why |
|--------------|-----------|-----|
| 4 today buckets: "Past Due / Due This Week / Rest of Mo. / Future Due" | "Already Due / Due Today / Due this week / Due Later" | "Due Today" as own bucket is more actionable. |
| "Due This Week" boundary unclear | Calendar week (Mon–Sun) | Operators plan in weekly cycles. |
| Range mode buckets implied not specified | 3 buckets: Carried Forward / Due in Period / Due After Period | Code already has this in `getFinancialsV2`. |
| Performance metric "Invoiced Created" | "Invoices Due" | Screen semantic is `due_date`, not `created_at`. |
| Late Fee Section as Phase 1 | Phase 2 | Figma (newer than Lark) doesn't include it. |
| Property Breakdown as edge case | First-class section (multi-property only) | Multi-property is a real persona (Priya). |
| Defaulter Analysis chart inside accordion | Always visible below | Health scan at a glance. |
| "Send Bulk Reminder" as Phase 1 | Phase 2 | Needs deeper notification integration. |
| Drill-down spec was a flat single table | Single matrix + 4 propagation rules | Mode-aware, no ambiguity. |
| "Governance" / "accountability" framing | "By team member" / "Who on my team is creating bills" | Operator-first voice. |
| Only Rajesh as primary persona | All four personas auditable | Setting-Up state adds Amit; EC-06/07 add Priya; EC-10 adds Meena. |

---

## Phase 2 — Deferred

| Feature | Why deferred |
|---------|--------------|
| Late Fee Section | Figma doesn't include it. Will validate first whether managers track late fees as a separate workstream. |
| Category × Defaulter cross-view | Cross-cutting two analytical dimensions. Phase 1 serves them independently. |
| Partial Payment filter on worklist | Adds query complexity. Phase 1 shows all invoices with any outstanding. |
| Bulk Reminder scheduling | Needs notification system integration. |
| Collection timing analytics (On-Time/Early/Late) | Requires new tracking. |
| Inline property comparison | V2.0 P0 — see [[DA-01 V2 Roadmap]]. |
| Repeat defaulter detection (3 of 6 months) | Needs subquery/materialized flag. V2.0 P0. |
| Snapshot Reports (balance-as-of-date) | Excel pivot of exports already serves accountants. V2.1 if needed. |

---

## Companion Docs

- [[DA-01 Engineering]] — reconciliation invariants with formulas, list API filter requirements, data sources, architecture notes, Figma component node IDs.
- [[DA-01 Codebase Feasibility]] — point-in-time codebase audit: what exists, what's net-new, what was killed, staleness flags vs Lark.
- [[DA-01 V2 Roadmap]] — V2 roadmap (Q3 2026 → Q1 2027): trends, forecasts, predictive layer, scheduled distribution.
- [[DA-01 Dues Detailed Analytics]] — original v3.1 spec (untouched, semantically identical; this is the cleaner reorganization).
