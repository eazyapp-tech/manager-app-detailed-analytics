---
title: DA-01 — Dues Detailed Analytics
date: '2026-05-01'
tags:
  - rentok
  - prd
  - homescreen
  - financial-insights
  - detailed-analytics
  - dues
aliases:
  - DA-01
  - Dues Detailed Analytics
  - Financial Insights - Dues
status: Final — V1 locked (revised post structured operator audit 2026-05-07)
version: '3.4'
figma: KgBQXiT7r7oGrcqZHCWxyU
section: Dues
---
> [!INFO] Source of Truth
> Local spec: `/Users/eazypg/RentOk Manager Homescreen/spec/DA-01-dues-detailed-analytics.md`
> V2 Roadmap companion: [[DA-01 V2 Roadmap]]
> **Engineers:** see [[DA-01 Build Sheet]] (V2, 9-column ticket-ready format) for implementation. This doc is the canonical "why."
> **Supporting refs:** [[_Ground Truth Field Map]] (entity fields, formulas, endpoints, permissions) · [[_Build Sheet Generation Spec]] (column conventions)
> Supersedes Lark PRD: "Dues (Live) · Financial Detailed Insights v1.1" (older, partially out of date)
> Design reference: Figma `KgBQXiT7r7oGrcqZHCWxyU` → Financials → Dues container (`7837:99662`)

**Series (Financial Detailed Analytics):** [[DA-01 Dues Detailed Analytics]] · [[DA-02 Collections Detailed Analytics]] · [[DA-03 Refunds Detailed Analytics]] · [[DA-04 Expenses Detailed Analytics]] · [[DA-05 Discounts Detailed Analytics]] · [[DA-06 Liabilities Detailed Analytics]] · [[DA-07 Cash Flow Detailed Analytics]]

> **v3.2 frontmatter fix:** Earlier v3.1 listed the wrong DA suite (Tenants / Complaints / Leads / Bookings — that's an entirely different homescreen-tile family, not the Financial Detailed Analytics suite). Corrected to canonical 7-screen Financial suite.

**Personas served:** Rajesh (primary, single-property owner-admin) · Priya (multi-property owner) · Amit (new manager, sparse data) · Meena (collections coordinator). Indirectly: Accountant (FY filter + Excel pivot), Owner-investor (V2.2 digest only).

---

# Dues — Detailed Analytics

## The One Thing This Screen Does

It answers: **"Kitna paisa atka, kidhar atka, kisko call karna hai?"**
("How much money is stuck, where is it stuck, and who do I call today?")

Every component on this screen exists to answer one of those three questions. If it doesn't, it shouldn't be there.

---

## A Morning with Rajesh

It's the 5th of the month. Rajesh manages two PGs in Pune (40 beds combined). He opens the app with his chai.

The Dues screen opens. He sees **₹16L** — bigger than last month. Red chip: ▲8% vs last month. Not good.

He scans the urgency bar. The red segment is thick — ₹6L already overdue. He taps it.

A list opens: 32 bills, oldest first. He sees Karan Mehta — ₹12,000 overdue, 18 days. He taps Call. *(In Phase 2, a Repeat badge will flag chronic defaulters — this tenant qualifies but the badge ships in Phase 2.)*

Done in 90 seconds. He didn't need to open a spreadsheet. He didn't need to ask his accountant. He saw the problem, found the person, made the call.

That is the experience this screen must deliver.

---

## A Morning with Priya (multi-property, May 5)

Priya manages 3 PGs in Mumbai. She opens DA-01. Hero ₹52L outstanding (▲6% vs ₹49L last month — amber, slight rise).

Property Breakdown: Property 2 — ₹28L (54% of total · with mini urgency bar showing 60% in 31+ days bucket — RED). Property 1 — ₹15L. Property 3 — ₹9L. Property 2 is bleeding old debt.

She drills into Property 2 → Defaulter Analysis Chart → 31–60 day bucket has 18 tenants, 60+ has 12. She taps Top Defaulters → 5 tenants account for ₹14L (50%). She decides: 3 escalation calls today, 2 deposit-adjustment letters this week.

Done in 4 minutes. Without DA-01, she'd have asked her three managers for individual property reports.

---

## A Morning with the CA via WhatsApp screenshot

Priya forwards her DA-01 hero to her CA. The CA opens the screenshot.

Hero shows ₹52L · "Total Outstanding (Accounts Receivable)" GAAP subtitle · "(across 84 bills · 3 properties)" · ▲6% MoM chip with prior absolute "vs ₹49L Apr."

CA understands: this is gross AR (unpaid bills). She replies: *"Aging-wise breakdown, plus deposit coverage on Old Tenants — send Excel."* Priya goes to overflow menu (⋮) → Email Dues Report → done.

Without the GAAP subtitle and prior-absolute, the CA would have asked: "Is this outstanding or invoiced?" and "Compared to what?"

---

## Scope

**This section is for money owed TO the property — unpaid bills only.**

| Included | Excluded (separate sections) |
|----------|------------------------------|
| Rent invoices | Collections (money received) |
| Security deposit invoices | Expenses (money spent) |
| Food / Mess charges | Deposits held (liability) |
| Electricity / Utility charges | Refunds |
| Maintenance and other charges | Discounts |
| Late fees (as part of invoices) | Late Fee dedicated analytics (Phase 2) |

---

## Personas Served

This screen is designed for four operator personas. Each has a different mental model and a different question this screen must answer.

| Persona | Role / Scale | Their question | How V1 serves them |
|---------|-------------|----------------|---------------------|
| **Rajesh** | Owner-admin, single 50-bed PG | "Who do I call today and how much should I push for?" | Primary persona. Hero, urgency, action card, top defaulters all built around his morning workflow. |
| **Priya** | Multi-property owner, 3+ properties across cities | "Which of my properties needs my attention?" | Property Breakdown section (§5) ranks properties; multi-property worklist groups by property. V2.0 adds side-by-side metric comparison. |
| **Amit** | New manager, just signed up, sparse data | "What should I do first to make this useful?" | Setting-Up State (§9) — banner + per-accordion overlays guide him through the first 30 days. |
| **Meena** | Collections coordinator, role-restricted | "What can I act on, and what's hidden from me?" | Permission gating (EC-10) — surfaces hide cleanly when she lacks access. Same dashboard view as Rajesh, scoped by what she can do. |

**Two additional personas served indirectly:**
- **Accountant** (book-closer, monthly/FY-end) — served via "This Fiscal Year" filter + Excel pivot of Invoice/Payment exports for exact balance reconstruction. V2 adds frozen-period semantics.
- **Owner-investor** (Priya's senior, weekly check-in, often not on the app) — NOT served by V1. Served exclusively in V2.2 via WhatsApp owner digest.

The vignette in this doc uses Rajesh because he's the primary. But every section is auditable against all four personas.

---

## The Six Rules

These are the rules that define this screen. Engineering should treat them as constraints, not suggestions.

1. **Live by default — and live always.** Total Dues, Population, Property Breakdown, Action Card, By Category, Defaulters, and the Aging chart show current outstanding regardless of date filter. The date filter only changes the period-activity sections (Performance, Added By) and reslices the Urgency Bar's buckets.
2. **The numbers must match.** The count on this screen (e.g., "120 Invoices") must exactly match the count in the drill-down worklist when the operator taps through. If it doesn't, it's a bug.
3. **Red means action now.** The urgency bar is designed so that a large red segment immediately communicates urgency, even if the operator hasn't read the numbers yet.
4. **Old tenant dues are not forgotten dues.** Tenants who've moved out still appear — with deposit context — so Rajesh knows whether the shortfall is recoverable.
5. **Deep analytics are on demand, not on load.** The Advanced Insights accordions are collapsed by default. The 60-second promise only works if the opening view is clean and fast.
6. **The time filter is global. Live elements are explicitly listed; everything else inherits.** The date filter at the top of Financial Insights flows into every drill-down by default. Sections that ignore it are enumerated in the Time Filter Behavior section — there is no other exception.

---

## How Rajesh Gets Here

`Homescreen → Overview section → "View Detailed Analytics" CTA → Financial Insights screen → Dues (Live) tab (selected by default)`

The Dues tab is the first tab selected when Financial Insights opens. Rajesh doesn't need to hunt for it.

---

## What Rajesh Sees

### 1. The Header

**"Dues (Live)"** — with a small ⓘ icon.

The "(Live)" label is intentional. It tells Rajesh the hero number is what's outstanding right now, not a period summary. It does not change when he picks a different period — Total Dues is always live.

Tapping ⓘ shows:
> "Total pending bills across selected properties, as of right now. Updates as payments are received or new bills are created."

---

### 2. Total Dues + Month-over-Month Chip

**The question this answers:** "How much total money is stuck?"

Rajesh sees a large number — ₹16L — with a smaller count below: *(120 Invoices).*

The hero is **always live**. Picking a different period does not change this number. What changes is how the urgency bar slices the same total — see Section 3.

Above the number, a comparison chip. This chip uses **inverted semantics** — a number going DOWN is good news for dues (less money stuck):

| Chip | Color | What it means |
|------|-------|---------------|
| ▼12% vs ₹14.8L last month | Green | Dues decreased — collection is working |
| ▲8% vs ₹14.8L last month | Red | Dues increased — more money is piling up |

> **v3.2 — show prior absolute alongside %:** Earlier v3.1 showed only the % ("▲8% vs last month"). Operators reason in rupees, not percentages. Match DA-04/06/07 discipline by surfacing both: "▲8% vs ₹14.8L last month."

The chip compares today's total outstanding to the same day of the previous month. It is independent of the date filter — it always compares right-now to last-month-same-day.

Tooltip on chip:
> "Compared to same day last month. ▼ means dues decreased (good). ▲ means dues increased (action needed)."

**When the chip is hidden — calibrated v3.2:**
- **First month of operation** (account < 30 days): show "Not enough prior data" instead of hiding silently
- **Prior period total = ₹0** (would cause ÷0): show "Not enough prior data"
- **Same-day-last-month is in Setting-Up state** (sparse data): show "Not enough prior data"

> **v3.2 ÷0 + partial-month discipline:** Earlier v3.1 silently hid the chip in these cases, leaving operator confused why no comparison appeared. Match DA-04/07 explicit "Not enough prior data" messaging.

**Multi-property:** When Priya is viewing all her properties together, the count reads: *(120 Invoices · across 3 properties).*

---

### 3. The Urgency Bar

**The question this answers:** "How urgent is the problem?"

A horizontal stacked bar — color-coded from critical to comfortable. Rajesh can read the health of his receivables at a glance before reading a single number.

**The bar reslices based on the date filter, but the total stays the same.** Same ₹16L, sliced two different ways depending on whether the operator is looking at "today" or a chosen period.

#### Today Mode (default) — 4 buckets relative to today

| Segment | Color | What it covers | Tap goes to |
|---------|-------|----------------|-------------|
| Already Due | Red | Bills whose due date has passed | Worklist: overdue bills, oldest first |
| Due Today | Amber | Bills due today | Worklist: today's bills |
| Due this week | Blue | Bills due Tuesday through Sunday of this calendar week | Worklist: this week's bills, due date first |
| Due Later | Green | Everything beyond this week | Worklist: future bills |

**"Due this week" is anchored to the calendar week (Monday–Sunday), not a rolling 7-day window.** PG operators plan collection in weekly cycles. "Clear these before the weekend" is a natural frame. A rolling window that shifts every day is harder to act on.

#### Range Mode — 3 buckets relative to the selected period

When Rajesh picks "This Month," "Last Month," "This FY," or any custom range, the bar reslices:

| Segment | What it covers | Tap goes to |
|---------|----------------|-------------|
| Carried Forward | Currently outstanding bills with due date BEFORE the period started | Worklist: due date < period start |
| Due in Period | Currently outstanding bills with due date INSIDE the period | Worklist: due date in [period start, period end] |
| Due After Period | Currently outstanding bills with due date AFTER the period ended | Worklist: due date > period end |

**This is not historical reconstruction.** It is the current outstanding total, decomposed by *when each unpaid bill was due relative to the selected period.* The hero stays at ₹16L. The bar tells you "of that ₹16L, ₹4L is old debt from before March, ₹10L came from March's bills, and ₹2L is future-dated."

**For an accountant picking "This Financial Year":**
- Carried Forward = pre-FY old debt still on books
- Due in Period = the FY's contribution to current outstanding
- Due After Period = post-FY-dated bills already created

That is the FY analysis question, answered without leaving the dashboard.

#### Bar mechanics (both modes)

Below the bar, a 2×2 (today mode) or 1×3 (range mode) grid shows each segment's label and amount. Tapping either the bar segment or the label row opens the worklist.

**Bar proportions:** Based on amounts, not invoice counts. A minimum bar segment width of 4px ensures even small amounts are visible.

**Tooltips per segment:**
- Already Due: "Bills past their due date. Need immediate attention."
- Due Today: "Bills due today. Send reminders or follow up now."
- Due this week: "Bills due Tuesday through Sunday of this calendar week."
- Due Later: "Bills due beyond this week."
- Carried Forward: "Currently unpaid bills with due dates before [period]. Old debt."
- Due in Period: "Currently unpaid bills due in [period]."
- Due After Period: "Currently unpaid bills due after [period]. Future-dated."

---

### 4. Population Breakdown

**The question this answers:** "Who owes this money — and do I have any leverage?"

Always live. Three tappable rows:

| Row | Who | Tap goes to | Tooltip |
|-----|-----|-------------|---------|
| Active Tenants | Currently checked-in tenants | Worklist: active tenants only | "Dues from currently checked-in tenants." |
| Booking | Confirmed bookings not yet checked in | Worklist: bookings only | "Dues from confirmed bookings not yet checked-in. Clear these before move-in date." |
| Old Tenants | Tenants who've moved out but still owe | Worklist: moved-out tenants | "Dues from moved-out tenants. Consider adjusting from deposit or initiating recovery." |

**Old Tenants row shows deposit context inline:**

- "₹8,000 deposit held · Covered by deposit ✓" — Rajesh can relax, the deposit covers it
- "₹3,000 deposit held · ₹2,000 shortfall" — he needs to recover ₹2,000 manually
- "No deposit held" — full exposure, needs immediate follow-up

This context is only shown on the Old Tenants row. Old tenant dues are the #1 source of silent leakage in PG operations — Rajesh needs to know immediately whether each one is recoverable.

---

### 5. Property Breakdown (Multi-Property Only)

**Visibility:** Shown only when the account has multiple properties AND the user is viewing more than one property.

**The question this answers:** "Which property is dragging the average down?"

A list of properties, each row showing:
- Property name
- Outstanding amount
- Share of total (percentage)
- Mini urgency bar (today/range mode same logic)

Sorted by outstanding amount, biggest problem at the top. Tap → worklist filtered to that single property.

For Priya managing 3 properties: *Sunshine PG ₹8L (50%) · Green Hostel ₹5L (31%) · Blue Residence ₹3L (19%).* She can see in one glance that Sunshine is the problem, tap it, and the worklist scopes to Sunshine.

Always live (ignores date filter), same as Population.

---

### 6. The Action Card

**The question this answers:** "What should I do first, right now?"

A dark, high-contrast card:
`32 Rent & Bills Overdue     View All >`

Always live (ignores date filter). Shown only when there are overdue bills. Tapping "View All" opens the worklist sorted by most overdue first — same as tapping the "Already Due" segment on the urgency bar in today mode.

**Hidden when:** Zero overdue bills, or property is in setup mode (no invoices created yet).

The card is dark because urgency deserves visual weight. It should make Rajesh feel mild urgency — not panic, but enough to act.

---

### 7. Advanced Insights

A section divider separates the always-on view (1–6 above) from the on-demand analysis. Four accordions, all collapsed by default. Rajesh expands what he needs.

**Why collapsed by default:** The 60-second promise only works if the first scroll shows the essentials. Power users who want category breakdowns or defaulter lists know where to look. First-time users shouldn't be overwhelmed.

---

#### 7A. Performance — "Am I billing correctly and collecting efficiently?"

**This accordion respects the date filter.** All three metrics are scoped to bills with due date inside the selected period.

| What Rajesh sees | What it means |
|-----------------|---------------|
| **Invoices Due** ₹22L | Total amount of bills with due date in the selected period (paid + unpaid) |
| **Current Dues** ₹16L | Of those bills, how much is still unpaid as of now |
| **Collection Efficiency** 27% | (Invoices Due − Current Dues) ÷ Invoices Due — share of period's bills already collected. **÷0 guard (v3.2):** if `Invoices Due = ₹0` (Setting-Up state, or quiet month), show "—" or "Not enough billed yet" instead of computing ÷0 or showing 0%. |

Collection Efficiency thresholds:
- **Green (>70%):** Collecting well
- **Amber (40–70%):** Room for improvement
- **Red (<40%):** Needs immediate attention

Below the metrics, a callout: "₹6L collected against bills due Apr 1 → Apr 30."

**Tooltips:**
- *Invoices Due:* "Total amount of bills due in the selected period. Includes both paid and unpaid bills."
- *Current Dues:* "Outstanding amount from bills due in this period."
- *Collection Efficiency:* "Percentage of period's billed amount that has been collected. 100% means every bill due in this period is fully paid."

**Note:** This is `due_date IN period`, not `created_at IN period`. The semantic across the entire screen is "when was the bill due," not "when was it raised." This keeps drill-downs consistent and matches existing list API behavior.

---

#### 7B. By Category — "Is this a rent problem or a utilities problem?"

**Always live** (ignores date filter). A ranked list of due categories. Each row: category icon + name, outstanding amount, and total billed amount. Tapping any row opens the worklist filtered to that category.

| Category | What's included |
|----------|----------------|
| Rent | Rent invoices |
| Electricity | Utility charges |
| Food | Mess charges |
| Security Deposit | Deposit invoices |
| Maintenance | Maintenance charges |
| Others | Everything else |

Sorted by outstanding amount — biggest problem at the top. If more than 5 categories have outstanding dues, the 6th onwards collapse into "Others" with a tap-to-expand option.

**Why this matters:** If rent dues are low but electricity dues are high, the problem is billing accuracy, not collection discipline. Different actions, different owner.

Tooltip on Others: *"Tap to expand and see breakdown of other categories like Security Deposit, Maintenance, etc."*

---

#### 7C. By Defaulters — "Which tenants are becoming a write-off risk?"

**Always live** (aging is always counted from today, regardless of date filter).

Two sub-components:

**Aging Breakdown** — how long bills have been overdue:

| Bucket | How long overdue | Collection probability |
|--------|-----------------|----------------------|
| 1–7 days | Just lapsed | ~95% recoverable — needs a nudge |
| 8–15 days | Getting stale | ~85% — needs a call |
| 16–30 days | Concerning | ~60% — needs escalation |
| 31–60 days | Risky | ~30% — consider deposit adjustment |
| 60+ days | Critical | <15% — legal or write-off territory |

Each bucket shows: count of tenants + total outstanding. Tapping opens the worklist filtered to that aging window.

Tooltip: *"How long bills have been overdue. Focus on the 8–30 day range — still recoverable, but getting urgent."*

**Top Defaulters List** — the 5 tenants who owe the most:

Each row shows:
- Tenant name (Repeat badge deferred to Phase 2 — see §7C)
- Room number
- Total outstanding
- Days overdue (oldest bill)
- "Last paid X days ago" (warning color if >30 days; "No payments yet" if never)

A **Repeat badge** (Phase 2) will flag tenants who've been overdue in 3 or more of the last 6 months — chronic defaulters vs one-time delays.

> **v3.2 deferral resolved:** Earlier v3.1 had Repeat badge specced in §7C as a V1 feature, but the Engineering Filter table marked it `**PHASE 2**` and Phase 2 list also listed it. Three contradicting locations. v3.2 resolves: **Repeat badge is Phase 2** because (a) the 6-month lookback requires materialized state that doesn't exist, and (b) Setting-Up state (account < 30 days) has no clean fallback for "3 of last 6 months." Strip badge from worklist row + Top Defaulter row in v3.2; reintroduce in Phase 2 with explicit maturity gate: `account_age >= 6 months OR show "—" instead of badge`.

Tooltip on Repeat badge (Phase 2): *"This tenant has been overdue in 3 or more of the last 6 months."*

Tapping a tenant opens their individual Dues Detail screen.

"View All Defaulters" below the list → full worklist sorted by outstanding amount.

---

#### 7D. Added By — "Who on my team is creating bills?"

**Respects the date filter.** Shows bill creators for bills due in the selected period.

A list of bill creators ranked by total amount of bills due in the period:
- Team members (avatar + name + total + invoice count)
- **RentOk** (system row) — auto-generated invoices like recurring rent and scheduled charges

Useful for Rajesh to spot if a team member is creating bills inconsistently, or if most billing is automated vs manual.

Tap any row → worklist filtered to bills created by that person AND due in the selected period.

Tooltip on RentOk row: *"Bills auto-generated by RentOk system (recurring rent, scheduled charges, etc.)"*

---

### 8. Defaulter Analysis Chart (Always Visible)

Below all four accordions, a stacked bar chart is always shown — no tap required to see it. Always live.

**Five bars** matching the §7C Aging Breakdown schema (1-7 / 8-15 / 16-30 / 31-60 / 60+). Each bar shows the number of tenants in that window. **Blue = tenants who had at least one payment recorded in that aging window** (i.e., partial recovery happened); **light grey = still outstanding** (no payment received yet).

This gives Rajesh an at-a-glance aging health scan even if he doesn't open any accordion. A healthy property has tall bars on the left (short aging) and short bars on the right. A problem property has the reverse.

> **v3.2 schema unification:** Earlier v3.1 had Defaulter Analysis Chart at 4 buckets (0–7 / 8–15 / 16–30 / 31+) while §7C used 5 buckets (1–7 / 8–15 / 16–30 / 31–60 / 60+). Two queries against the same data must agree. v3.2 canonicalizes on §7C's 5 buckets — recovery-probability calibration depends on the 31–60 vs 60+ split for "deposit adjustment" vs "legal/write-off" decisions.

> **Blue bar definition (v3.2):** Earlier v3.1 said "tenants collected from in that window" — ambiguous. v3.2 explicit: "tenants who had at least one payment recorded in this aging window." A tenant in the 31–60 bucket with a partial payment 45 days post-due-date renders blue; one with no payment yet renders grey.

---

## 9. The Setting-Up State (For New or Sparse Accounts)

**Persona:** Amit just signed up last week. Of his 30 beds, 5 have tenants in the system. 3 bills exist, all auto-generated by RentOk. He opens Detailed Analytics. Hero says ₹15K. Urgency bar mostly empty. Performance accordion shows weird zeros. **The screen doesn't look broken — but it FEELS broken to him.**

If we don't address this in V1, Amit closes the screen, doesn't open it again, and assumes the analytics don't work. **First-session quality is V1 critical.**

### When this state applies

The screen enters Setting-Up state when ANY of these is true:
- Account age < 30 days
- Total invoices created in account < 10
- Less than 30% of configured beds have an active tenant
- Less than 30 days of billing history available

The frontend computes this once on dashboard load.

### What changes

**Persistent banner above the date filter:**

> "Setting up — your analytics will get richer as you add tenants and bills. **5 of 30 beds have tenants · 3 bills created so far** · [Quick setup checklist →]"

The banner is dismissable per session but reappears next session until the criteria are met.

**Per-component behavior in Setting-Up state:**

| Component | Behavior |
|-----------|---------|
| Hero Total Dues | Shows real number (could be ₹0 or small) — no override |
| MoM chip | Hidden — no comparison baseline yet |
| Urgency Bar | Shows real data; hidden segments are blank, not "loading" |
| Population Breakdown | Shows real data — even 5 tenants is real |
| Property Breakdown | Hidden if account has only 1 property in setup |
| Action Card | Shown if any overdue exists; hidden otherwise |
| Performance accordion | Shows with overlay: "Need 30 days of bills to show meaningful collection efficiency. You have X days of history." |
| By Category | Shows real categories present; "Others" collapses immediately |
| By Defaulters / Aging | Shows real data — 1 overdue tenant is still real |
| Top Defaulters | Shows up to 5; doesn't say "Top 5" if there are only 2 |
| Added By | Shows real creators; if everything is RentOk auto-generated, that's a flag — banner reads "Most bills are auto-generated. [Add manual bills →]" |
| Defaulter Analysis Chart | Shows real data; sparse buckets render as small but visible |

### What we deliberately do NOT do

- Show fake / sample data anywhere. Operators must trust real numbers.
- Hide entire sections. Better to show real data with context than to hide and confuse.
- Force a "guided tour." This is an analytics screen, not an onboarding flow. The banner suffices.
- Block drill-downs. Even with 1 overdue bill, the worklist works. Honor Rule 2 (numbers must match).

### Quick setup checklist (linked from banner)

The "[Quick setup checklist →]" link routes outside this screen — to the property setup flow, owned by a different team. DA-01 references it; doesn't duplicate it.

The checklist (referenced) covers:
- Add tenants
- Configure rent schedules
- Connect payment gateway
- Connect WhatsApp

When all 4 are done AND 30 days have passed AND ≥ 10 invoices exist, the banner self-dismisses permanently.

### Engineering invariant for Setting-Up state

Even in Setting-Up state, all reconciliation invariants must hold. The 32 in "32 Bills Overdue" still equals the worklist count. Setting-up doesn't relax correctness — it adds context.

---

## Drilling Down

Every tappable number on this screen leads to either the **Dues Worklist** (a filtered list of invoices) or the **Tenant Dues Detail** (a single tenant's complete picture).

This section is structured in three parts:
- **Part A:** the drill-down matrix — which dashboard element maps to which list filter
- **Part B:** time context propagation rules
- **Part C:** worklist behavior (filters, headers, multi-property, deep links)

### Part A — Drill-down Matrix

Each row specifies: source element on dashboard → list API parameters → list header text → default sort → primary action surfaced on each row.

#### Today-mode urgency drill-downs (date filter = today only)

| Source | API params | List header | Default sort | Primary action |
|--------|-----------|-------------|--------------|----------------|
| Already Due segment | `due_date < today` | "Already Due · 32 Bills · ₹6L" | Most overdue first | Send Reminder |
| Due Today segment | `due_date = today` | "Due Today · 18 Bills · ₹4L" | Amount DESC | Send Reminder |
| Due this week segment | `due_date IN [Mon, Sun]` of current calendar week, excluding today | "Due This Week · 28 Bills · ₹3.8L" | Due date ASC | Send Reminder |
| Due Later segment | `due_date > end of this calendar week` | "Due Later · 42 Bills · ₹2.2L" | Due date ASC | View Bill Detail |

#### Range-mode urgency drill-downs (date filter = any past or custom range)

| Source | API params | List header | Default sort | Primary action |
|--------|-----------|-------------|--------------|----------------|
| Carried Forward segment | `due_date < range_start` | "[Period] · Carried Forward · 12 Bills · ₹4L" | Most overdue first | Send Reminder |
| Due in Period segment | `due_date IN [range_start, range_end]` | "[Period] · Due in Period · 60 Bills · ₹10L" | Due date ASC | Send Reminder |
| Due After Period segment | `due_date > range_end` | "[Period] · Due After Period · 18 Bills · ₹2L" | Due date ASC | View Bill Detail |

#### Always-live drill-downs (ignore date filter, see Part B)

| Source | API params | List header | Default sort | Primary action |
|--------|-----------|-------------|--------------|----------------|
| Active Tenants row | `tenant.status = 1` | "Active Tenants · 85 Tenants · ₹4L" | Amount DESC | Send Reminder |
| Booking row | `tenant.status = 2` | "Bookings · 12 Tenants · ₹2L" | Amount DESC | Send Reminder |
| Old Tenants row | `tenant.status = 0` | "Old Tenants · 8 Tenants · ₹0.5L" | Days since checkout DESC | Adjust from Deposit |
| Property Breakdown row | `property = X` (single property scope) | "[Property Name] · 45 Bills · ₹8L" | Most overdue first | Send Reminder |
| Action Card "View All" | `due_date < today` | "Past Due · 32 Bills · ₹6L" | Most overdue first | Send Reminder |
| Category row | `due_type = X` | "Rent · 24 Bills · ₹8L" | Amount DESC | Send Reminder |
| Aging bucket | computed `today − due_date` in window | "1–7 Days Overdue · 84 Tenants · ₹3L" | Most overdue first | Send Reminder |
| Top Defaulter row | tap → Tenant Dues Detail (not worklist) | — | — | Call Tenant |
| "View All Defaulters" | no extra filter | "All Defaulters · 120 Tenants · ₹16L" | Outstanding DESC | Send Reminder |

#### Period-sensitive accordion drill-downs (inherit date filter)

| Source | API params | List header | Default sort | Primary action |
|--------|-----------|-------------|--------------|----------------|
| Performance: Invoices Due | `due_date IN [start, end]` | "[Period] Invoices Due · 80 Bills · ₹22L" | Due date ASC | View Bill Detail |
| Performance: Current Dues | `due_date IN [start, end]` AND `outstanding > 0` | "[Period] Current Dues · 60 Bills · ₹16L" | Most overdue first | Send Reminder |
| Performance: Collection callout | Routes to Collections tab, not Dues worklist | — | — | — |
| Added By: team member | `created_by = X` AND `due_date IN [start, end]` | "[Period] · Bills by Riya · 14 Bills · ₹2.1L" | Due date DESC | View Bill Detail |
| Added By: RentOk row | `created_by = system` AND `due_date IN [start, end]` | "[Period] · Auto-Generated Bills · 250 Bills · ₹18L" | Due date DESC | View Bill Detail |

---

### Part B — Time Context Propagation Rules

**The default rule:** every drill-down inherits the global date filter as `start_date` / `end_date` on the list API, applied to `due_date`.

**Three explicit exception classes** — these are the ONLY exceptions:

**Class 1 — Always-live elements:** drill-down sends NO `start_date` / `end_date`. The list shows current outstanding regardless of what date is selected on the dashboard. These are: Active Tenants, Booking, Old Tenants, Property Breakdown rows, Action Card, By Category rows, Aging buckets, Top Defaulters, "View All Defaulters."

**Class 2 — Mode-aware urgency buckets:** the urgency bar uses the date filter to define its own segments. Drill-downs from urgency segments send a custom `due_date` range (today / week / range_start / range_end-derived) — not the global filter directly.

**Class 3 — Period-sensitive accordions:** Performance and Added By drill-downs inherit the global filter as `start_date` / `end_date` on `due_date`. No need for `created_at` semantics — `due_date` is the operating field across the screen.

If the operator changes a filter on the worklist itself, that change is local to the worklist. It does NOT propagate back to the dashboard.

---

### Part C — Worklist Behavior

**List header always confirms context.** When arriving via drill-down:
- Single-period contexts: "[Period if any] · [Cohort] · X Bills · ₹Y"
- Single-property scope: prefix property name
- Multi-property aggregated: "All Properties · [Cohort] · X Bills · ₹Y"

**Filter chip display.** When arriving from a drill-down, the worklist shows removable filter chips at the top so Rajesh knows why this list is filtered: e.g., `[Due Date: Apr 1 – Apr 30 ✕] [Category: Rent ✕]`. Tapping ✕ removes that filter; the list re-fetches.

**Available filters on the worklist:**
- Urgency bucket
- Population (Active / Booking / Old)
- Category (Rent / Electricity / Food / Security Deposit / Maintenance / Others)
- **Property** (multi-property accounts only)
- Aging window
- Amount range (custom min–max)
- Search by tenant name or phone
- Payment status: Unpaid / Partially Paid (Phase 2)

**Per-invoice row:** Tenant name, room number, outstanding amount (prominent), due date ("18 days overdue" if past), invoice type badge. **In multi-property mode, also shows property name.** *(Repeat badge — Phase 2 — see §7C.)*

**Per-invoice actions:** Send Reminder (WhatsApp/SMS), Record Payment, Call Tenant, View Bill Detail, Adjust from Deposit (Old Tenants only). The primary action surfaced varies by drill-down origin — see the matrix above.

**Bulk actions:** Send Bulk Reminder (select multiple), Export to Excel.

**Multi-property worklist behavior.** When viewing aggregated across multiple properties:
- Rows are grouped by property — each property is a section header (`Sunshine PG · 12 Bills · ₹3.5L`) followed by its rows
- Property filter chip allows narrowing to a single property
- Sort applies within each property group; group order matches sort criterion
- Single-property tap from Property Breakdown bypasses grouping

**The count must match** (Rule 2). The "32" in "32 Rent & Bills Overdue" must equal the row count in the worklist when Rajesh taps "View All." Tested every release.

**Refresh.** Worklist refetches on initial load, pull-to-refresh, after any in-list mutation, and after back-navigation from Tenant Detail.

**Back-navigation.** Worklist → Tenant Detail → back returns to the same worklist with same filters and same scroll position. Worklist → back returns to dashboard with original date filter and accordion state preserved.

**Deep links.** WhatsApp reminders and push notifications can deep-link directly into the worklist with pre-applied filters: `rentok://dues?filter=overdue&property=PG123`. Bypasses the dashboard.

**Pagination.** 20 rows per page, infinite scroll. (Note: current API hardcodes limit=5000; Phase 1 ships with proper pagination.)

---

## Tenant Dues Detail

Reached by tapping a tenant in the Top Defaulters list or in the worklist.

**Header:** Tenant name, room (or "was in [room]" for old tenants), total outstanding, deposit held, days since last payment.

**Four sections:**

| Section | What's shown | Order |
|---------|-------------|-------|
| Open Bills | All unpaid invoices for this tenant | Oldest due date first |
| Payment History | Last 10 payments | Most recent first |
| Reminder History | Last 5 reminders sent | Most recent first |
| Late Fee History | Late fees added and paid | Most recent first |

**Actions:** Call Tenant · Send Reminder · Record Payment · View Full Profile · Adjust from Deposit *(only for moved-out tenants with deposit > 0)*

---

## Drill-Down Behavior

> **Universal navigation rules apply to all 7 DA specs.** Priorities (P0/P1/P2) are PM recommendations; engineering may re-prioritize during spike based on existing app capabilities and effort.

### Universal Rules

**R1. Modal/Sheet/Screen primitive [P0]** *(updated 2026-05-11)*
Every tap target's destination is explicitly typed: full-screen push, bottom sheet, modal overlay, or inline accordion.

**ⓘ icon convention (locked across all DA specs):** single-tap → bottom sheet (BS) with plain-English explanation + GAAP framing where applicable. **No inline tooltip. No long-press.** Replaces older "tap=tooltip + long-press=BS" dual pattern. See `[[_Build Sheet Generation Spec#15. ⓘ Icon Interaction Convention]]`.

Non-ⓘ tooltips (chip tooltips, micro-help icons inside accordions) remain at designer's discretion.

**R2. Back-stack semantics [P0]**
Back pops one navigation frame and restores the prior frame's filter chips, scroll position, accordion state, and selected segments. iOS swipe-back and Android system back behave identically.

**R3. Deep links + share sheet [P1]**
Every drill state is uniquely URL-addressable as `rentok://da-NN/<view>?<filters>`. Push notifications, WhatsApp reminders, and the in-app "Share this view" affordance generate these URLs. Cold-start with deep link bypasses tabs and lands directly in the drill state.
> Engineering note: existing app uses singular-form deep links (e.g., `rentok://collection`). DA-01 already has precedent: `rentok://dues?filter=overdue&property=PG123`.

**R4. Permission gating UX [P0]**
Sections without read permission HIDE (not gray). Disabled actions show a lock icon; tap → toast naming the missing permission. Cross-screen drill into a denied screen shows a full-screen denial; back returns to the source bottom sheet. Mobile must inspect `can_view_invoices` flag from homepage response. EC-10 (Meena example) is the canonical reference for this pattern.

**R5. Loading states [P0]**
Every screen has skeleton-on-load (not spinner). Cross-screen navigation transitions show the destination header immediately + skeleton rows for content. Pull-to-refresh shows the chevron without flashing skeletons.

**R6. State preservation [P1]**
DA tab switches preserve per-tab filter and scroll state. App background/foreground within 15 min restores exact drill state. Force-quit + reopen returns to homescreen. Token expiry triggers silent re-auth + return-to-drill.
> Engineering note: 15-min restoration threshold is a PM recommendation; calibrate during spike.

**R7. Multi-property scope inheritance [P0]**
Single-property drill OVERRIDES global scope for the entire descendant drill stack. Cross-screen CTAs inherit single-property scope. The scope chip is always visible in the worklist header.

**R8. Worklist filter-chip behavior [P0]**
Every pre-applied filter is a removable chip. Removal re-fetches without that filter; other filters stay. New filters are additive (AND). Worklist filter changes do NOT propagate back to the dashboard. (DA-01 already specifies this — the canonical reference for the suite.)

**R9. Shareable state [P2]**
Every dashboard, worklist, and detail has a "Share this view" affordance in the overflow menu (⋮). Generates a deep link + system share sheet. Excel export remains for deep-analysis; deep link is for live-data sharing.
> Engineering note: net-new feature; defer to Phase 2 if app router work is not in scope.

**R10. CA-screenshot discipline [P0]**
Every hero has a visible GAAP subtitle and basis label — never tap-only. DA-01's "Total Outstanding (Accounts Receivable)" subtitle (v3.2) is the canonical pattern for the suite. No critical context in tap-only tooltips.

**R11. Cross-screen back path [P1]**
Cross-screen navigation (DA-X → DA-Y via "View in DA-Y" CTA) pushes the destination as a CHILD of the source's bottom sheet. The destination shows a breadcrumb header "← From [Source DA name]". Tap breadcrumb → returns directly to the source bottom sheet.
> Engineering note: if app doesn't have breadcrumb support today, fall back to standard back-stack pop (R2) for v1.

### Priority Summary

| Priority | Rules | Engineering guidance |
|----------|-------|---------------------|
| **P0 (must have)** | R1, R2, R4, R5, R7, R8, R10 | Universal mobile expectations; broken UX or persona regression without |
| **P1 (should have)** | R3, R6, R11 | Significant UX value; depends on existing app router/persistence patterns |
| **P2 (defer if needed)** | R9 | Net-new feature beyond current Excel export; Phase 2 candidate if scope-constrained |

### Per-Spec Specifics (DA-01)

- **Tenant Dues Detail screen** = full-screen push with 4 internal sub-tabs (Open Bills · Payment History · Reminder History · Late Fee History). Pull-to-refresh refetches all 4 tabs.
- **Send Reminder action** shows a confirmation modal listing recipients before sending. Cancel/Send buttons. After Send: toast confirmation, return to row.
- **Adjust from Deposit action** opens an inline modal calculator (Deposit balance, Outstanding, Adjustment amount). On confirm: row updates in place; toast confirmation.
- **Quick Setup Checklist link** (in Setting-Up state banner) routes to a flow owned by another team. If that flow is unavailable, the banner fallback is to hide rather than break.
- **Defaulter Analysis Chart bars** are tappable → worklist filtered to that aging bucket (R7 multi-property scope inheritance applies).
- **Always-Live drills** (Active Tenants, Booking, Old Tenants, Property Breakdown rows, Action Card, By Category rows, Aging buckets, Top Defaulters): drill-down sends NO `start_date`/`end_date` — list shows current outstanding regardless of dashboard date filter. Class 1 elements per DA-01 §"Drilling Down" rule.
- **Range-mode urgency bar** drills inherit the date range (Class 2 elements).

---

### Per-Spec Specifics — Additions (post-orphan-audit)

- **Setting-Up banner dismiss-per-session:** first dismiss shows confirmation tooltip *"You can re-enable this from Settings → Onboarding"*; subsequent dismiss in same session is silent. Banner re-shows on next session if underlying data is still missing.
- **"View All Defaulters" CTA** (below Top Defaulters list): full-screen push to defaulters worklist with no aging-bucket chip pre-applied (all buckets). Inherits scope. Back returns to DA-01 with scroll restored.
- **EC-02 "Add Dues" empty-state CTA:** routes to existing Add Dues flow (cross-app feature). On completion: returns to DA-01 dashboard with refresh (re-fetches hero + sections).
- **Dashboard overflow (⋮) → "Email Dues Report":** confirmation modal showing recipient email + selected period → on confirm: toast *"Export sent to [email]. You'll receive it in a few minutes."* Stays on dashboard. **Note:** this is the existing `/reports/generateNewDuesReport` endpoint at `src/routes/reports.ts:89` — verify column shape during spike.
- **Dashboard overflow (⋮) → "Share this view":** R9 (P2). Generates deep link `rentok://dues?period=...&scope=...` + system share sheet.
- **Defaulter Analysis Chart stacked-bar sub-segment tap:** R12 — drills to worklist with both aging bucket AND tenant-status filter chips applied (e.g., "30-60 days" + "active tenants only").
- **Filter chip ✕ on worklist** (R8 clarification): each of the 6 worklist chips (Date / Property / Tenant Status / Aging Bucket / Recorded By / Custom) has explicit ✕; tapping removes that chip and re-fetches.

### Universal Rule Clarifications (post-orphan-audit)

Resolves interaction-primitive ambiguities surfaced during the post-master orphan-tap-target audit. Treat as additive to R1–R11 above.

**R1 clarification — explicit primitives for common UI:**
- **Hero ⓘ icon:** single-tap = inline tooltip (one-line plain-language definition); long-press = bottom sheet with full GAAP framing + basis label.
- **MoM chip:** tap = inline tooltip showing prior-period numbers + computation window. No drill.
- **Accordion section:** tap on row OR chevron toggles expand/collapse. Default state per section spec.
- **Information-only chips** (held-duration, possible-duplicate, settlement-readiness, repeat-tenant on a row): single-tap = inline tooltip. No drill unless explicitly listed in Per-Spec Specifics.

**R5 clarification — pull-to-refresh:** Dashboard re-fetches all sections in parallel; worklist re-fetches with current chips. Cross-screen drill destinations also support pull-to-refresh (re-fetches that child view). Chevron animates without flashing skeletons.

**R8 clarification — filter chip ✕ as explicit tap target:** Every chip has a discrete ✕ icon (44pt min hit area). Tapping ✕ removes that chip and re-fetches; other chips stay. Tapping the chip body opens the chip's edit affordance (date picker, multi-select) where applicable; otherwise body-tap is no-op.

**R12 (NEW) — Trend chart conventions [P0]:**
- **Bar single-tap:** inline tooltip showing exact values. No drill.
- **Tap-into-tooltip CTA "View [period] →":** drills to worklist filtered to that period (cross-period drill overrides parent hero period for descendant drills).
- **Stacked-segment tap** (e.g., yellow-within-green sub-segment): drills to worklist with the sub-segment's filter applied.
- **Period selector toggle (`6M` / `This Month`):** tap to switch chart range. Selected state preserved on back. Does NOT affect dashboard hero values.
- This supersedes earlier drafts that listed both "tap → tooltip" and "tap → worklist" as primary actions.

### Updated Priority Summary

| Priority | Rules | Engineering guidance |
|----------|-------|---------------------|
| **P0 (must have)** | R1, R2, R4, R5, R7, R8, R10, R12 | Universal mobile expectations |
| **P1 (should have)** | R3, R6, R11 | Significant UX value |
| **P2 (defer if needed)** | R9 | Net-new beyond Excel export |

### Permission Vocabulary Reality Check

Codebase has only **11 JWT-mirrored permission keys**: `appAccess, cashCollection, recordPayment, editInvoices, editTenants, viewInvoices, viewExpenses, deleteInvoices, addTenants, deleteTenants, viewTenants` (mirror sites: `src/v1/login/property/service.ts:79-91` + `src/controllers/property.ts:14883/15041/15767/17813` + `src/helpers/teamMember.ts:134`). DB-side via `checkAuthInDb`: ~70 snake_case columns on `team_member_property` (no enum wrapper).

Keys cited in this spec vs codebase reality:

| Cited key | Status | Recommended Phase 1 path |
|-----------|--------|--------------------------|
| `viewInvoices` | EXISTS — JWT key | Canonical read-gate for dashboard + worklist |
| `view_invoices_of_self_added_tenants` | EXISTS — DB column (via `checkAuthInDb`) | Already in use by dues service for fallback gating |
| `viewTenants` | EXISTS — JWT key | Use for "View Tenant Detail" cross-screen drill (replaces fictional `viewTenantDetails`) |
| `sendReminders` (cited) | MISSING | Reuse `viewInvoices` for read-only nudges, OR build new column for separate audit trail |
| `recordPayment` | EXISTS — JWT key | For "Mark as Paid" worklist action |

> **Decision owner: Jatin (Sr Backend).** Each MISSING key requires build-or-reuse decision: (a) add new DB column + JWT mirror at 5 sites + frontend toggle, OR (b) reuse existing key. Specs cannot ship until Jatin signs off. Recommendations are PM suggestions only.

---

## Time Filter Behavior

The date filter at the top of Financial Insights is **global**. By default, it flows into every drill-down. Rule 6 codifies this. The exceptions below are explicit and complete.

| Component | Behavior | Rationale |
|-----------|---------|-----------|
| Total Dues Hero | Always live — number does not change | "What is stuck right now" is the question, every time |
| Urgency Bar | Reslices: today mode = 4 buckets vs today; range mode = 3 buckets vs period | Same total, decomposed differently |
| MoM Chip | Always compares right-now to last-month-same-day | Independent of date filter |
| Population | Always live | Tenant status is current state |
| Property Breakdown | Always live | Property scope is current state |
| Action Card | Always live | "What should I do today" — not "what should I have done in March" |
| By Category | Always live | "Where is the pile-up right now" |
| By Defaulters & Aging | Always live | Aging is always counted from today |
| Top Defaulters | Always live | Current state of who owes most |
| Defaulter Analysis Chart | Always live | Same as above |
| Performance (Invoices Due, Current Dues, Collection Efficiency) | Respects date filter | Period activity question |
| Added By | Respects date filter | "Who created bills due in this period" |

**Why no balance-as-of-date view.** Operators don't ask "what was my outstanding balance at 11:59 PM on 31st March." They ask "what's stuck right now?" and "how much of that came from March's bills?" — both already covered by always-live hero + range-mode urgency bar. For exact past-timestamp balances (rare, formal accounting), accountants pivot Invoice + Payment exports in Excel/Tally.

**When Rajesh picks "Last Month":**
- Hero, urgency total, population, property breakdown, action card, categories, defaulters, aging chart → unchanged. Still showing right-now state.
- Urgency bar → reslices to 3 buckets (Carried Forward / Due in Period / Due After Period).
- Performance accordion → shows "March 1–31: ₹22L invoices due, ₹16L current dues, 27% efficiency."
- Added By accordion → shows creators of bills due in March.
- A small banner below the date filter: *"Live values reflect right now. Period values show March activity."*

---

## Edge Cases

**EC-01: Zero dues — all clear.** Hero shows ₹0 (0 Invoices). Urgency bar hidden. Empty state: *"All clear! No pending dues across your property."*

**EC-02: No invoices ever created.** Same as EC-01, different message: *"No bills created yet. Start by adding rent dues for your tenants."* with an "Add Dues" button (hidden if no permission).

**EC-03: Invoice partly paid.** ₹10,000 invoice with ₹6,000 collected → shows as ₹4,000 outstanding. Counts as 1 invoice. Worklist row: "₹4,000 outstanding (of ₹10,000)."

**EC-04: Bill created and due today.** Appears in "Due Today" bucket. Disappears from Dues on next refresh after payment.

**EC-05: Tenant moves out with unpaid bills.** Moves to Old Tenants row. Deposit context shown inline. "(Moved out)" badge in worklist. "Adjust from Deposit" action available.

**EC-06: Multi-property view (Priya).** All numbers aggregate across selected properties. Property Breakdown section appears. Worklist groups rows by property, with property filter chip to narrow.

**EC-07: Single-property tap from Property Breakdown.** Worklist scopes to one property and bypasses grouping. Filter chip shows `[Property: Sunshine PG ✕]`.

**EC-08: Large property (500+ unpaid invoices).** Computations unchanged. Worklist uses pagination (20 at a time). Top Defaulters still shows top 5. Large numbers formatted with K/L/Cr suffixes.

**EC-09: Stale data between homescreen and detail screen.** Homescreen shows ₹16L. Rajesh taps through. A payment was recorded in between. Detailed screen shows ₹15.5L — that's fine. Detail always fetches fresh on load. Pull-to-refresh always available.

**EC-10: Meena has accounting access, not full admin.** Dues section visible. Worklist actions: "Record Payment" only if she has Record Payments permission. "Adjust from Deposit" only if she has Edit Dues & Collection permission. Surfaces are hidden (not grayed out) when she lacks access.

**EC-11: Cancelled or voided invoices.** Never shown. Only active, non-void invoices with outstanding > 0 appear.

**EC-12: One tenant, multiple overdue invoices.** Vikram Joshi: ₹10K rent + ₹2K electricity + ₹3K food, all overdue. *(Tenant name. Distinct from the operator persona Amit — see Personas section.)*
- Worklist: 3 separate rows
- Top Defaulters: 1 row, ₹15K total
- Aging: each invoice ages independently

**EC-13: Range-mode period with zero data.** Rajesh picks a custom range with no bills due. Urgency bar shows "No bills due in this period" placeholder. Live elements (hero, population, etc.) still show today's state.

**EC-14: Period crosses a fiscal year boundary.** "Last 6 Months" overlapping FY boundaries — buckets work normally. Carried Forward includes ALL bills due before period start, regardless of FY.

---

## Words on the Screen

### Section Header
**"Dues (Live)"** — does not change with date filter.

### Empty States

| When | Message | CTA |
|------|---------|-----|
| No invoices ever | "No bills created yet. Start by adding rent dues for your tenants." | "Add Dues" (if permitted) |
| All bills paid | "All clear! No pending dues across your property." | — |
| Worklist: filter returns nothing | "No bills match your current filters." | "Clear Filters" |
| Range mode: no bills due in period | "No bills were due in this period." | — |

### Error States

| When | Message | Recovery |
|------|---------|----------|
| Network failure | "Couldn't load dues data. Check your connection and try again." | "Retry" button |
| Advanced Insights section fails | Core KPIs show normally. Failed accordion: "Couldn't load this section." | "Retry" on that accordion |
| Slow load on large dataset | "This is taking longer than usual for large properties." | Auto-retry once, then retry button |
| Worklist drill-down count mismatch | (Should never happen — bug.) Show error: "Numbers couldn't be loaded correctly. Tap to refresh." | Retry |

### Data Freshness
If data is more than 5 minutes old: *"Last updated X minutes ago"* (subtle, below Hero KPI).

If in range mode: info banner — *"Live values reflect right now. Period values show [period] activity."*

---

## Decisions That Override Lark PRD v1.1 / Figma

The Lark PRD v1.1 was written before the homescreen was built and is partially out of date. Where this document differs:

| Lark / Figma showed | What we're building instead | Why |
|---------------------|---------------------------|-----|
| 4 today-mode buckets: "Past Due / Due This Week / Rest of Mo. / Future Due" | "Already Due / Due Today / Due this week / Due Later" | "Due Today" as own bucket is more actionable. Aligns with homescreen code structure. |
| "Due This Week" boundary unclear | Calendar week (Mon–Sun) | Operators plan in weekly cycles. Rolling 7 days shifts every day and loses calendar meaning. |
| Range mode bucketing was implied but not specified | 3 buckets: Carried Forward / Due in Period / Due After Period | This logic already exists in `getFinancialsV2`. |
| Performance metric labeled "Invoiced Created" | "Invoices Due" | Semantic across the screen is `due_date`, not `created_at`. |
| Late Fee Section as Phase 1 | Phase 2 | Figma (newer than Lark) does not include it. |
| Property Breakdown as edge case | First-class section (multi-property only) | Multi-property managers (Priya) need property attribution as a primary view. |
| Defaulter Analysis chart inside an accordion | Always visible below accordions | Health scan at a glance. |
| "Send Bulk Reminder" as Phase 1 | Phase 2 | Needs deeper notification system integration. |
| Drill-down spec was a single flat table | 3-part structured spec (Matrix × Mode + Time Rules + Worklist) | Mode-aware, no ambiguity on filter inheritance. |
| Operator-facing language: "governance," "accountability" framing | Plain "By team member" / "Who on my team is creating bills" | Operator-first voice. |
| Only Rajesh covered as primary persona | All four personas (Rajesh, Priya, Amit, Meena) auditable | Section §9 Setting-Up State adds Amit. EC-06/07 add Priya. EC-10 covers Meena. |

---

## v3.3 → v3.4 Calibrations (spec audit + pre-edit verification, 2026-05-11)

After a misread of the Hero model in mid-conversation prompted a re-audit. Pre-edit verification agent ran a thorough production-code check before any spec changes. Findings led to:

| v3.3 (locked) | v3.4 (current) | Why |
|---|---|---|
| **HB6 — "production filter excludes Old Tenants AND Partially-Paid invoices"** as one combined launch blocker | **HB6 split into HB6 (Old Tenants — RESOLVED to widen) + HB6b (Partial-Paid — RESOLVED no-op).** HB6: single-line widen `t.status` from `IN (1, 2)` to `IN (0, 1, 2)` at `helpers.ts:61`. HB6b: production NEVER writes `i.status = 2` (only commented-out refs at `payment.ts:5358, 5517`); partial-paid is captured via destructive `i.amount` mutation at `payment.ts:2039-2048`. Hero formula stays `SUM(i.amount) WHERE status = 0` — DO NOT widen to `status IN (0, 2)`. | Pre-edit verification surfaced that the "widen status" proposal was pointless (no matching rows) and risked double-counting in future. Surgical resolution. |
| **Hero universe ambiguity** between "always-live" (PRD V3.3) and period-sliced (mid-conversation re-interpretation) | **Hero is always-live, period filter slices breakdowns / segments / worklist drills only.** Documented explicitly in §1 of Build Sheet and Hero row Plain Drill. | The mid-conversation re-read was wrong; the original PRD model was correct. The fix here is to make it MORE explicit so future readers don't mis-interpret. |
| HB7 not present | **HB7 NEW — homepage tile alignment.** Two divergent "Total Dues" numbers exist on homepage today: `getHeaderStats` tile is date-filtered (defaults to current month) at `homepage/service.ts:480-486`; `getFinancialsV2` Financials tile is always-live at `homepage/service.ts:2005-2019`. Decision: align both to always-live. | UX bug-in-waiting: two numbers labeled "Total Dues" with different semantics. |
| HB8 not present | **HB8 NEW — Aging/Top Defaulters/Defaulter Chart need future-period guard + UI empty state.** Add `due_date ≤ today` guard to `applyDefaulters` + bucket math at `service.ts:2009-2016`. Future-only period → empty state ("Future-dated bills aren't overdue yet."). Mixed: operate on past portion. | Aging concept doesn't apply to future-dated bills; current code returns silently empty without graceful UI fallback. |
| HB9 not present | **HB9 NEW (SECURITY) — `getDuesWidget` permission enforcement gap.** Service.ts:132-138 sets `self_added_team_uuid` for restricted team members but never applies it as WHERE clause. Worklist enforces it (`service.ts:96`); widget doesn't. Cross-tenant aggregate leak. Tracked as CSB-8 — likely affects DA-02/03/04 widget endpoints too. | Surfaced by pre-edit verification agent. Not DA-01-specific but DA-01 carries the surface. |
| **R1 (ⓘ icon convention)** said "tap = inline tooltip; long-press = bottom sheet with GAAP" | **R1 updated:** single-tap → bottom sheet only. No inline tooltip. No long-press. Per Generation Spec §15 (locked across all DA suite). | Per design simplification: one interaction primitive, one destination. Replaces dual pattern that was hard to discover (long-press) and bandwidth-limited (inline tooltip). |
| CSB-8 not present | **CSB-8 added** to Cross-Suite Engineering Blockers — widget endpoint permission enforcement gap (security). | App-wide pattern bug; affects multiple DA list-screen widget endpoints. |
| **Default filter at app open** unspecified explicitly | **Today (Live) confirmed** as default. Matches production at `homepage/service.ts:1982-1984`. No code change needed. | Audit confirmed production already aligns. |

**Production code changes required for DA-01 V1 launch:**

1. `src/v1/list_screens/dues/helpers.ts:61` — widen `t.status IN (1, 2)` to `IN (0, 1, 2)` (HB6).
2. `src/v1/homepage/service.ts:480-486` — align "Total Dues" header tile to always-live OR rename to distinct metric (HB7).
3. `src/v1/list_screens/dues/helpers.ts:295-313` (`applyDefaulters`) — add `i.due_date ≤ today` guard (HB8).
4. `src/v1/homepage/service.ts:2009-2016` — add `due_date ≤ today` guard for aging/defaulter buckets in urgency-bar math (HB8).
5. `src/v1/list_screens/dues/service.ts:132-138` (`getDuesWidget`) — apply `self_added_team_uuid` as WHERE clause (HB9 / CSB-8).
6. UI: build graceful empty state for Aging / Top Defaulters / Defaulter Chart when entirely future period selected (HB8).
7. Designer task: decide visual indication near Hero for old-tenant contribution (HB6 default fallback: Population row's Old Tenants treatment is sufficient).

**Out of DA-01 scope but flagged to Jatin/Nimit:**

- Destructive `i.amount` mutation pattern at `payment.ts:2039-2048` loses original billed amount from invoice rows. Long-standing data-modeling issue. Audit trail lives only on Payments side. Separate ticket.

---

## v3.1 → v3.2 Calibrations (post structured operator audit)

The 2026-05-07 audit found DA-01 had ~40% audit-pattern coverage despite v3.1 lock. v3.1 → v3.2 closes structural gaps and resolves internal contradictions.

| v3.1 (locked) | v3.2 (current) | Why |
|---|---|---|
| **Series line listed wrong DA suite** (Tenants/Complaints/Leads/Bookings — different homescreen-tile family) | Series corrected to Financial Detailed Analytics (DA-01 through DA-07) | Frontmatter-level orphan — wrong wikilink targets |
| **Aging bucket schema split** — §7C used 5 buckets (1-7/8-15/16-30/31-60/60+); §8 chart used 4 buckets (0-7/8-15/16-30/31+) | All canonicalized to §7C's 5-bucket schema | Two queries against same data must agree; recovery-probability calibration depends on 31-60 vs 60+ split |
| **Repeat badge contradicted itself** — §7C shipped V1; Engineering filter table marked Phase 2; Phase 2 list also listed it | Repeat badge **deferred to Phase 2**; stripped from §7C body, narrative, worklist row, Top Defaulter row | 6-month lookback requires materialized state that doesn't exist; Setting-Up state has no clean fallback |
| **MoM chip showed only %** | Chip shows BOTH % AND prior absolute ("▲8% vs ₹14.8L last month") | Operators reason in rupees; matches DA-04/06/07 |
| **MoM chip silent-hid in ÷0 / no-prior-data cases** | Explicit "Not enough prior data" string with calibrated triggers | Silent hide left operator confused; matches DA-04/07 |
| **Collection Efficiency ÷0 not handled** | If `Invoices Due = ₹0` shows "—" or "Not enough billed yet" | Setup-mode + quiet-month edge case |
| **Defaulter Analysis Chart "Blue = collected" definition opaque** | Explicit: "tenants who had at least one payment recorded in this aging window" | Was decorative-or-analytic ambiguity |
| **Only Rajesh persona narrative** | Added "A Morning with Priya" (multi-property triage) + "A Morning with the CA via WhatsApp" (GAAP screenshot test) | DA-07 has 3 personas; Priya was named but had no narrative |
| **No Pre-Launch Engineering Blockers section** | Section added below | DA-04/05/06/07 all have one; DA-01 ships V1 with engineering work pending (aging query, pagination, etc.) |
| **Hero subtitle had no GAAP framing** | Subtitle: "Total Outstanding (Accounts Receivable)" + "(across 84 bills · 3 properties)" | CA-screenshot reader needs accounting term |

---

## Pre-Launch Engineering Blockers (v3.2)

These items must be resolved before DA-01 ships. Not Phase 2 — Phase 1 launch dependencies. Structured per DA-07 v1.2 pattern.

### HARD blockers (mobile screen would be wrong)

| Item | Why hard-block | File:Line |
|------|----------------|-----------|
| **HB1. Aging windows query gap** | `applyDefaulters` is currently 60+ only. §7C and §8 require 5 buckets (1–7 / 8–15 / 16–30 / 31–60 / 60+). Without this, By Defaulters accordion + Defaulter Analysis Chart can't render. | `src/v1/list_screens/dues/applyDefaulters` |
| **HB2. Pagination implementation** | API hardcodes `limit=5000`. If a property has 5,001 invoices, hero count and worklist count will diverge — direct violation of Rule 4 (numbers must match). | (path TBD) |
| **HB3. Cross-screen reconciliation invariant — DA-01 ∩ DA-02 = ∅** | A paid invoice in DA-02 must NOT appear in DA-01. A partly-paid invoice appears in both for different amounts. No automated test exists per the spec. | (test infrastructure) |
| **HB4. Cross-screen reconciliation invariant — DA-01 deposit dues ∩ DA-06 held = ∅** | Same tenant cannot have an unpaid SD/CM invoice in DA-01 AND a held SD/CM in DA-06 for the same invoice. | (test infrastructure) |
| **HB5. Hero `Total Dues` query canonicality** | Inherits DA-02 HB1 question — DA-01 hero is gross unpaid, but the calc must match what DA-02 sees as "not yet collected." Tied to DA-02 v3.0 Rule 0 canonical aggregation. | (cross-screen) |
| **HB6. Production filter excludes Old Tenants from Dues Hero — RESOLVED to widen** (audit 2026-05-11) | Production dues hero filter at `src/v1/list_screens/dues/helpers.ts:61` is `t.status IN (1, 2)` — excludes old/checked-out tenants (`t.status = 0`). Per PRD spec audit 2026-05-11: **old tenants WITH unpaid dues MUST appear in Hero.** Single-line backend change: `t.status IN (0, 1, 2)`. Old-tenant contribution surfaces to operator via Population breakdown's Old Tenants row (existing). Visual indication treatment: designer's call — requirement is "operator must be able to see the old-tenant contribution clearly." Note: `t.status = 8` (interested-booking-lead) NOT included — leads don't typically carry unpaid dues. | `src/v1/list_screens/dues/helpers.ts:61` |
| **HB6b. Partial-paid invoices — RESOLVED, no change needed** (audit 2026-05-11) | Production NEVER writes `i.status = 2` — only commented-out references at `payment.ts:5358, 5517`. Partial payments destructively mutate `i.amount` in place at `payment.ts:2039-2048`. `SUM(i.amount) WHERE status = 0` already captures partial outstanding correctly. **DO NOT widen Hero formula to `status IN (0, 2)` or `amount - paid_amount`** — pointless today (no rows match) and could double-count if pattern is ever fixed in future. Hero formula stays as production currently has it. Separate concern (out of DA-01 scope): the destructive mutation loses original billed amount from the invoice row — long-standing data-modeling issue, flag to Jatin/Nimit. | `src/v1/list_screens/dues/helpers.ts` (no change); `src/controllers/payment.ts:2039-2048` (data-model concern, separate ticket) |
| **HB7. Two divergent "Total Dues" numbers on homepage — must align** (NEW, audit 2026-05-11) | Homepage shows TWO "Total Dues" tiles with different semantics: `getHeaderStats` tile at `src/v1/homepage/service.ts:480-486` is **date-filtered** (defaults to current month); `getFinancialsV2` Financials tile at `src/v1/homepage/service.ts:2005-2019` is **always-live** (no date filter on SUM, just bucketing). These can return DIFFERENT numbers on the same screen. **Per spec audit: align both to always-live.** Engineering change: remove the date filter at `service.ts:480-486` OR rename the header-stats tile to a different metric ("Dues This Month"). | `src/v1/homepage/service.ts:480-486` |
| **HB8. Aging buckets / Top Defaulters / Defaulter Chart — future-period graceful empty state needed** (NEW, audit 2026-05-11) | Current `applyDefaulters` at `helpers.ts:295-313` has NO `due_date ≤ today` guard. When user selects a future period (e.g., October), these sections naturally return empty results but with no graceful UI fallback. **Decision: bound these sections by `due_date ≤ today`. Future-period selection → graceful empty state** ("Future-dated bills aren't overdue yet. These sections show past-due bills only."). Engineering: add guard to `applyDefaulters` + urgency-bar bucket math at `homepage/service.ts:2009-2016` + build UI empty state. | `src/v1/list_screens/dues/helpers.ts:295-313` + `src/v1/homepage/service.ts:2009-2016` |
| **HB9. Widget self-added permission gap — SECURITY** (NEW, audit 2026-05-11) | `getDuesWidget` at `src/v1/list_screens/dues/service.ts:132-138` sets `self_added_team_uuid` for team members with only `view_invoices_of_self_added_tenants` permission BUT NEVER applies it as a WHERE clause to the widget query. Worklist path enforces it (`service.ts:96`); widget path does not. **Effect: cross-tenant aggregate leak to team members with restricted permission.** Likely affects DA-02/03/04 widget endpoints too — pattern issue, not DA-01-specific. **Tracked as Cross-Suite Engineering Blocker CSB-8.** | `src/v1/list_screens/dues/service.ts:132-138` (DA-01 surface); likely also `collections/service.ts`, `expenses/service.ts`, etc. |

### PARITY blockers (drill-downs and aggregates must match)

| Item | Why parity-block |
|------|------------------|
| **PB1. `paid_amount` per row in API response** | Worklist row shows "₹4,000 outstanding (of ₹10,000)" — partial-paid invoices need paid_amount in API response. |
| **PB2. Aging schema unification** | §7C and §8 must use the same 5 buckets. v3.2 spec aligns; engineering must verify. |
| **PB3. Always-live drill-downs MUST NOT pass `start_date`/`end_date`** | Class 1 elements at L517 explicit. Verify URL params have NO date range when tapping Active Tenants from a "Last Month" filter. |

### DEPENDENCY (owned by another spec)

| Item | Owner |
|------|-------|
| **DB1. DA-02 v3.0 Rule 0 canonical aggregation** | DA-02 owns the canonical query path; DA-01 hero recovery rate inherits this. |
| **DB2. Quick setup checklist routing** | L437 — "owned by a different team." If that flow doesn't exist or breaks, the Setting-Up state banner CTA is dead. |

### DEFERRABLE (Phase 1 hotfix safe)

| Item | Why deferrable |
|------|----------------|
| **D1. Amount range filter on Worklist** | Niche worklist filter; not blocking primary drill-downs. |
| **D2. Multi-property worklist grouping** | Verify but don't block. |
| **D3. Defaulter Analysis Chart blue-bar computation refinement** | Decorative chart; if rendering simpler (just outstanding-only bars) saves time, defer the layered logic. |

---

## Cross-Suite Engineering Blockers (DECISION NEEDED — Jatin)

Surfaced during the post-master-merge audit of all 7 DA specs. **NOT DA-suite-specific** — these affect every authenticated endpoint in the app. The DA suite *uses* these endpoints, so Phase 1 launch readiness depends on Jatin's verdict on each.

> **Decision owner: Jatin (Sr Backend).** PM has surfaced; Jatin decides: fix-now vs. fix-fast-follow vs. accept-and-document. Items marked with ⛔ block DA-suite launch until resolved.

| ID | Issue | Files | Risk | Recommendation |
|----|-------|-------|------|----------------|
| ⛔ **CSB-1** | `checkAuth()` returns `true` when `req.is_authenticated === false`. `HeaderValidator` falls through with `next()` (not 401) when Authorization header is missing. **Net result: any endpoint protected by `HeaderValidator + checkAuth` can be hit unauthenticated by simply omitting the Bearer header.** | `src/utils/commonFunctions.ts:1193-1196` (HeaderValidator fallthrough); `:1228` (checkAuth bypass); `:1252` (checkAuthInDb same shape) | Production exposure — every list/detail screen the DA suite drills into is reachable anonymously | Patch `HeaderValidator` to return 401 on missing header, OR fix `checkAuth` to fail-closed when `is_authenticated=false`. Default risk-acceptance is NOT viable for a financial-data app. |
| ⛔ **CSB-2** | DA-01 + DA-02 drill endpoints have **zero middleware** — no HeaderValidator, no JWT decode. `POST /invoices/fetchDueDetailsForTenants` and `POST /invoices/fetchPaymentSettlementDetails` accept body params and query directly. | `src/routes/invoices.ts:203-204` → `src/controllers/invoices.ts:3331, 3424` | Anonymous read of tenant payment data | Add `HeaderValidator` + `checkAuth(viewInvoices)`. Mandatory before DA-01/02 launch. |
| ⛔ **CSB-3** | Tenant Detail + Adjust Deposit + Invoice Detail + Receipt PDF drill destinations have NO `HeaderValidator`. The DA suite assumes these are safe drills. | `src/routes/tenant.ts:927` (`getTenantData`), `:931` (`getTenantStepperv2`), `:944` (`adjustDeposit` — financial mutation); `src/routes/invoices.ts:215` (`getInvoiceData`), `:216` (`generateReceipt`), `:225` (`generateReceiptWrapper`), `:227` (`getSpecificInvoiceData`) | Anonymous read of PII; **anonymous financial mutation** at `/tenant/adjustDeposit` | Add `HeaderValidator` + appropriate `checkAuth` to each. `/tenant/adjustDeposit` is an absolute P0. |
| ⛔ **CSB-4** | DA-03 + DA-04 detail services have HeaderValidator but NO inline `checkAuth`. Authenticated user with any property access can read any refund/expense by uuid. | `src/services/refunds/refunds.ts:488` (refunds, also re-used by `routes/reimbursement.ts:10` via duplicate controller); `src/services/expense/expense.ts:20` (expenses; also no `pg_id` validation) | Cross-property data leak between team members | Add `checkAuth(viewInvoices)` for refunds and `checkAuth(viewExpenses)` for expenses. Logged in DA-03 HB4 + DA-04 HB4. |
| **CSB-5** | Expired-JWT bypass — for non-tenant packages (Manager/Owner/Landlord apps), `HeaderValidator` decodes the **expired** payload, sets `req.is_authenticated = true`, calls `next()`. A stolen-but-expired Manager App token still authenticates indefinitely. | `src/utils/commonFunctions.ts:1104-1126` | Stolen/leaked tokens have indefinite life | Decide: (a) intentional grace window for offline mode (document the window length), OR (b) bug — return 419 for non-tenant packages too. |
| **CSB-6** | JWT permission-key shape diverges by code path. `helpers/teamMember.ts:136` writes 12 keys (includes `bankAccess`); `v1/login/property/service.ts:79-91` writes 11 (omits it). | `src/helpers/teamMember.ts:134-145` vs `src/v1/login/property/service.ts:79-91` | Permission gating becomes path-dependent | Align both sites to the canonical key set. |
| **CSB-7** | Refund/Reimbursement controller duplication — `routes/refunds.ts:10` and `routes/reimbursement.ts:10` both call `RefundsController.getRefundDetails`, but import from two **different files** (`controllers/refunds.ts` vs `controllers/reimbursement.ts`), both classes named `RefundsController`. | as cited | Auth fix in one is silently missed in the other | Dedupe controllers OR add lint check ensuring auth changes propagate. |
| ⛔ **CSB-8** (NEW 2026-05-11) | **Widget endpoint permission enforcement gap.** List-screen widget endpoints (`getDuesWidget` at `src/v1/list_screens/dues/service.ts:132-138`; likely same pattern in `collections/service.ts`, `expenses/service.ts`, `refunds/service.ts`, etc.) set up `self_added_team_uuid` for team members with restricted permission (only `view_invoices_of_self_added_tenants`) **BUT NEVER apply it as a WHERE clause to the widget query.** Worklist paths enforce it; widget paths do not. | `src/v1/list_screens/dues/service.ts:132-138` (verified); other list_screens files (suspected — needs audit) | **Cross-tenant aggregate data leak via widget endpoints.** Restricted team member sees correctly-scoped worklist but un-scoped widget totals (entire property's dues). Privacy/security concern on a financial app. | Add the same `andWhere('t.added_by_id = :self_team_uuid', ...)` enforcement to widget query paths. Audit DA-02/03/04 widget endpoints for the same pattern bug; likely shared issue. |

⛔ = blocks DA-suite launch. CSB-1, CSB-2, CSB-3, CSB-4 must be resolved before Phase 1 ships. CSB-5/6/7 are app-hygiene; can ship as fast-follow but should be on Jatin's queue.

---

## Phase 2 — Intentionally Deferred

| Feature | Why it's deferred |
|---------|------------------|
| **Late Fee Section** | Figma doesn't include it. Phase 2 — after validating that managers track late fees as a separate workstream. |
| **Category × Defaulter cross-view** | Cross-cutting two analytical dimensions. Phase 1 serves them independently. |
| **Partial Payment filter on Worklist** | Adds query complexity. Phase 1 shows all invoices with any outstanding amount. |
| **Bulk Reminder scheduling** | Sending reminders to many tenants with timing rules needs notification system integration. |
| **Collection timing analytics** | On-Time / Early / Late payment splits require new tracking. |
| **Inline property comparison** | Phase 2 adds a side-by-side comparison view (V2.0 P0-#7). |
| **Repeat defaulter detection** | Recurrence over 6-month lookback needs a subquery or materialized flag. |
| **Snapshot Reports (balance-as-of-date)** | Excel pivot of exports already serves accountants. |

---

## For Engineering

### Reconciliation Invariants

| Invariant | Formula |
|-----------|---------|
| Hero is always live | Total Dues = `SUM(i.amount) WHERE status=0 AND amount>=1 AND is_active=1 AND tenant.status IN (0,1,2)` — independent of date filter |
| Today-mode urgency sums to total | Already Due + Due Today + Due This Week + Due Later = Total Dues |
| Range-mode urgency sums to total | Carried Forward + Due in Period + Due After Period = Total Dues |
| Population sums to total | Active + Booking + Old Tenants = Total Dues |
| Property breakdown sums to total | Sum of all property rows = Total Dues (multi-property mode) |
| Category sums to total | Rent + Electricity + Food + Deposit + Maintenance + Others = Total Dues |
| Performance: Invoices Due ≥ Current Dues | Invoices Due − Current Dues = Collected for period (cannot be negative) |
| Cross-screen consistency | Homescreen Dues tile = Detailed Dues hero, when same property scope is applied |
| Drill-down count matches | List header count must equal worklist row count exactly |
| MoM chip is independent of date filter | Always (right-now total) vs (same-date-last-month total) |

### Filters the List API Must Support

| Filter | Required for | Status |
|--------|-------------|--------|
| `due_date` start/end range | All drill-downs | **EXISTS** (`applyDateRange` filters on `i.due_date`) |
| `outstanding > 0` | Default base universe | **EXISTS** (`buildBaseQuery`) |
| Tenant `status` (1=active, 2=booking, 0=old) | Population drill-downs | **EXISTS** (`applyTenantTypes`) |
| `added_by` | Added By drill-downs | **EXISTS** (`applyAddedBy`) |
| Computed aging windows | Aging bucket drill-downs | **NEW** — current `applyDefaulters` is 60+ only |
| `due_type` (category) | Category drill-downs | **EXISTS** (`applyDueTypes`) |
| `property_id` / `pg_number_filter` | Multi-property | **EXISTS** |
| Property grouping in result rows | Multi-property worklist | **EXISTS** (`buildBaseQuery(strings, 'property')`) |
| Amount range (min/max) | Worklist amount filter | **NEW** — small SQL addition |
| `paid_amount` per row | Partial-paid display | **VERIFY** |
| Repeat defaulter flag | Repeat badge | **PHASE 2** |

**Note:** `filter_codes` and `start_date`/`end_date`/`due_types`/`added_by` are mutually exclusive in the current API. All drill-downs in this PRD use the `start_date`/`end_date` path.

### Architecture Notes

1. **Hero is mode-independent.** Single query returns both today-mode and range-mode buckets in one pass.
2. **`buildBaseQuery()` is shared** between homescreen and list — foundation of the count-consistency invariant.
3. **Worklist groups by tenant in current code** (`GROUP BY i.payer`). Drill-downs may need invoice-level vs tenant-level rows depending on context.
4. **Pagination is currently disabled** — `limit=5000`. Phase 1 must implement real pagination.
5. **`tenant_types` filter is independent of `filter_codes`** — Population drill-downs layer cleanly.

### Figma Component Node IDs

| Component | Node |
|-----------|------|
| Full Dues Section | `7837:99662` |
| Hero + MoM Chip | `7837:99673` |
| Urgency Stacked Bar | `7837:99691` |
| Population Rows | `7837:99717` |
| Action Card | `7837:99741` |
| Advanced Insights Container | `7837:99747` |
| Defaulter Analysis Chart | `7837:99812` |

---

> [!NOTE] Codebase feasibility (full table)
> The local spec has a complete "Codebase Feasibility" appendix with line-level code references (`getFinancialsV2:1879-1893` etc.), gap effort estimates, and "what was considered and killed" entries. See the local spec file or [[DA-01 V2 Roadmap]] Appendix A.
