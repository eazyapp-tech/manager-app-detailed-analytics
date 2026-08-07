---
title: DA-07 Cash Flow Detailed Analytics
date: '2026-05-07'
tags:
  - rentok
  - prd
  - spec
  - detailed-analytics
  - cashflow
  - financials
aliases:
  - DA-07
  - Cash Flow Detailed Analytics
---
> [!INFO] Source of Truth
> This note is the canonical spec for the Cash Flow Detailed Insights screen.
> **Engineers:** see [[DA-07 Build Sheet]] (9-column ticket-ready format) for implementation. This doc is the canonical "why."
> **Supporting refs:** [[_Ground Truth Field Map]] · [[_Build Sheet Generation Spec]]
> No old PRD existed for this screen — written from first principles + codebase audit.
> Related: [[DA-01 Dues Detailed Analytics]] · [[DA-02 Collections Detailed Analytics]] · [[DA-03 Refunds Detailed Analytics]] · [[DA-04 Expenses Detailed Analytics]] · [[DA-05 Discounts Detailed Analytics]] · [[DA-06 Liabilities Detailed Analytics]]

# Cash Flow — Detailed Analytics

> **Product:** RentOk Manager App → Financial Insights → Cash Flow
> **Version:** 1.3
> **Status:** Final — Locked (codebase + 2 operator audit rounds + Figma audit 2026-05-07)
> **Last Updated:** 2026-05-07
> **Supersedes:** No prior PRD. Existing Excel report `generateCashFlowReport.ts` continues to ship as the deep-analysis tool with mandated bug fixes (see Pre-Launch Engineering Blockers).
> **Design Reference:** Figma `KgBQXiT7r7oGrcqZHCWxyU` → Financials → Cash Flow (node ID `7826-96615`)
> **Parent Spec:** Homescreen Financial Insights container

---

## The One Thing This Screen Does

It answers: **"Kitna kamaaya, kitna gaya, kya net hua?"**
("How much earned, how much went out, what's the net?")

Cash Flow is the **synthesis screen** — it composes data from Collections (DA-02), Refunds (DA-03), Expenses (DA-04), Discounts (DA-05), and Liabilities (DA-06) into a single net direction signal. It answers one question no other screen answers: *"Did I make money this month, or did I lose it?"*

> **What this screen is NOT:** It is not a profit & loss statement. It is not a bank balance reconciliation. It is not an accountant's close-grade tool. It is a **mobile cash-flow summary** for the operator's monthly check, with an Excel report download for the accountant doing the actual close.

---

## A Morning with Rajesh (mid-month gut check, May 15)

It's the 15th of May. Rajesh opens Cash Flow.

Hero shows **−₹15,000** in red. He pauses. Last month was +₹54K. What changed?

Inflows: ₹1.4L. Outflows: ₹1.55L. He scrolls — Salary ₹50K, Electricity ₹40K, Rent paid ₹35K, Mess ₹15K, Maintenance ₹15K. All standard. Refunds ₹0. He thinks for a second — salary always pays in the first week, but rent collection peaks 20th–25th. He's seeing a mid-month timing illusion.

Below, the trend chart shows last 5 months all positive. He glances and exhales. This will turn green by month-end.

Done in 90 seconds. The screen prevented panic, gave him pattern context.

---

## A Morning with Priya (multi-property check, May 31)

Priya opens Cash Flow on May 31. Hero: **+₹2.1L** across 3 properties. Healthy.

By Property: Property 1 +₹85K · Property 2 −₹15K · Property 3 +₹140K. Property 2 is red.

She taps Property 2. Inflows ₹1.8L, Outflows ₹1.95K. Refunds ₹35K (5 move-outs end-April). She remembers — Aakash flagged a complaint cluster there. Move-outs hit cash. Expected.

She emails the Excel report to her CA for monthly close. *"Export sent to [email]."*

Done in 3 minutes. She knew what was wrong, why, and what to send to her accountant.

---

## A Morning with the CA (month-end validation)

Priya forwards the screenshot to her CA. The CA opens DA-07 on her own phone for the same period.

Looks at hero: **+₹2.1L net operating cash flow.** Below, sees the deposit movements section: **+₹1.2L net deposit flow** (deposits collected exceeded refunds). She nods — that's a net liability increase, not revenue.

She taps the Excel CTA. Receives the .xlsx. Opens it on laptop. Reconciles to bank statements. The mobile screen was her smell test; the Excel is her work surface.

---

## Scope

**This section shows period-based cash movement — what came in and what went out during the selected time range.**

| Included                                                                         | Excluded                                                                                          |
| -------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| Cash actually received in period (`p.paid_date` within period, `p.status = 1`)   | Mode 211 (deposit-applied) and Mode 288 (advance-applied) — paper transfers, no cash movement     |
| Cash actually paid out in period (expenses `e.paid_date`, refunds `refund_date`) | Discounts (concessions before billing — already netted in collections, NOT a cash outflow)        |
| Deposits collected (separated as Section B — non-revenue cash)                   | Bank balance (RentOk does not track operator cash position)                                       |
| Refunds disbursed                                                                | Owner personal withdrawals (no schema; off-system)                                                |
| Operating expenses across all DA-04 categories                                   | RentOk-funded discounts (`Credits.source = 1` — no operator revenue impact, no make-whole exists) |
| Conditional capex flagging (when "Other" expenses > 25% of outflow)              | Investing/Financing GAAP buckets (folded into Operating + Deposit Money In/Out)                   |
|                                                                                  | GST handling (residential PG typically GST-exempt; surface in Phase 2 if commercial)              |
|                                                                                  | Accrual revenue / unbilled amounts (cash basis only)                                              |

> **Why no opening/closing balance:** RentOk does not track an operator's cash position. The wallet balance in the system is RentOk's tenant-collection float, not the operator's bank balance. DA-07 shows **movement, not position**. Phase 2 may add operator-entered opening balance.

---

## The Four Rules

1. **Cash basis. Always.** Revenue and expenses recognised when money moves, not when bills are dated. Uses `p.paid_date`, `e.paid_date`, `refund.refund_date`. Matches DA-02/03/04 governing date rules.

2. **Deposits are cash but not revenue.** Deposits collected appear in Section B (Deposit Money In/Out), separated from Operating Cash Flow. The hero number is **Net Operating Cash Flow** — what the operator actually made or lost. Conflating deposits with revenue is the single biggest mental-math trap on this screen.

3. **Mode 211 and 288 are paper transfers — excluded entirely.** When a tenant settles a new bill by deducting from prior deposit (mode 211) or prior advance (mode 288), no cash moved. These never appear as inflows. They appear (if at all) only in DA-02 Collections' separate adjustment rows.

4. **Up is good — but only on Net Cash Flow.** The hero green/red color logic is on the SIGN of net cash flow (positive = green = good; negative = red = bad). This is opposite of DA-04 Expenses (where up = bad). MoM chips on individual lines are suppressed because composition matters more than direction of any single component.

---

## How Rajesh Gets Here

`Homescreen → Overview → "View Detailed Analytics" → Financial Insights → Cash Flow tab`

Cash Flow is the seventh and final tab in Financial Insights (Dues · Collections · Refunds · Expenses · Discounts · Liabilities · **Cash Flow**).

---

## What the User Sees

### 1. The Header

**"Cash Flow"** — with a small ⓘ icon.

No "(Live)" suffix — Cash Flow is period-based (matches DA-02/03/04/05). Active period (e.g., "May 1–31") is visible in the time filter chip above.

Tapping ⓘ shows:
> "Money in vs. money out for the selected period. The hero shows net operating cash flow — what you made after expenses and refunds. Deposits are tracked separately because they have to be returned. For full reconciliation, download the Excel report."

---

### 2. Hero KPI — Net for the Month

**The question this answers:** "Did I make money this period, or did I lose it?"

A large number — color-coded by sign:
- **Positive:** `+₹85,000` in green
- **Negative:** `−₹38,000` in red
- Zero: `₹0` in neutral

**Two-line subtitle:**
- Line 1 (small grey): *"Net Operating Cash Flow"* — GAAP-readable for CA screenshot use case
- Line 2 (small grey): *"₹2.85L in − ₹2L out"* — component totals (period dates already shown in time filter chip above)

> **Hero label "Net for the Month" + GAAP subtitle (v1.2 calibration):** Earlier v1.1 draft only had "Net for the Month" as visible label with GAAP framing in long-press tooltip. Second-round audit confirmed CA screenshot use case fails — accountant viewing a forwarded screenshot can't long-press, sees only operator-Hindi label. v1.2 promotes "Net Operating Cash Flow" to a visible secondary subtitle line, addressing both personas without sacrificing operator readability.

**Long-press tooltip on hero number:**
> "Net Operating Cash Flow. This is your operating cash profit. Security deposits aren't included because you have to give them back. For full reconciliation to your bank, download the Excel report (overflow menu)."

**MoM comparison chip — 3 states (v1.2 simplified from 4):**
- **Improvement** (same-sign growth, OR sign-reversal to positive): ▲25% vs ₹68K Apr → green
- **Regression** (same-sign decline > 10%): ▼12% vs ₹98K Apr → amber
- **Sign reversal to negative**: ▼ vs +₹54K Apr → red (now-negative net was previously positive)
- Sign-reversal glyph (e.g., ↻) prepended on either reversal direction so operator sees both magnitude and reversal

Show **both** % AND prior absolute: *"▲25% vs ₹68K Apr"*. Operators reason in rupees.
- **Partial-month rule:** Same-elapsed-days comparison: *"▲25% vs Apr 1–15"*. Tooltip on chip: *"Same days compared."*
- **Prior total = ₹0:** *"Not enough prior data."*

> **MoM 4→3 state collapse (v1.2):** Earlier v1.1 had 4 states including a "neutral grey" for ≤10% regressions. Operator audit caught: neutral grey is indistinguishable from "no chip rendered" (no prior data). v1.2 collapses to 3 distinguishable states (green / amber / red).

> **Sign-reversal-to-positive bug fix (v1.2):** Earlier v1.1 said "chip color goes red regardless of magnitude" on sign reversal. Bug: prior month −₹100, current month +₹500 → flipped POSITIVE → green, not red. v1.2 distinguishes reversal-to-positive (green) from reversal-to-negative (red).

> **Why no standalone sign-flip callout:** Earlier v1.0 draft had a separate "⚠️ Cash flow turned negative this month" callout firing on prior-2-periods comparison. Would fire every mid-month for every operator (salary out, rent not yet in) — alarm fatigue. Replaced with sign-coloring on the existing MoM chip.

**Per-bed Net Cash Flow (multi-property only, inline below the hero number):**
*"+₹850 per occupied bed"* — only renders when multi-property is selected. Single-property operators don't see this row.

> **Why multi-property only (v1.1):** Per-bed has no comparison value for a single-property operator. Operator audit cut it for single-property views.

**Mid-month timing nudge (conditional, below MoM chip):**
Fires when ALL three conditions hold:
1. Current day-of-month < 25
2. AND time filter chip displays the **"This Month" preset** (NOT a custom range that happens to cover the current month — preset only)
3. AND the period contains today's date

→ *"💡 It's the [N]th — collections typically peak after the 20th. This may turn green by month-end."*

> **Custom range exclusion (v1.2 clarification):** If operator picks a custom range like "May 1–15" on May 15, the nudge does NOT fire — they explicitly chose a partial period and don't need to be told it's partial.

> **Why this is in v1.1 (promoted from Phase 2):** Zero engineering effort (string conditional). Saves Rajesh from mid-month panic. Highest-ROI deferred item — operator audit promoted it.

**Deposit-dependency callout (conditional, below mid-month nudge):**
Fires when ALL three conditions hold:
1. Deposits collected (Section B inflow) ≥ 30% of total cash inflows
2. AND property has ≥ 5 active deposit-holders (maturity gate — same pattern as DA-06)
3. AND current month is not a property's first 60 days from launch

→ *"⚠️ ₹35K of inflow is deposits (held as liability — see [[DA-06 Liabilities Detailed Analytics]]). Net for the month excludes this: +₹50K."*

> **Threshold rationale (recalibrated v1.1):** Earlier draft used `≥ 25%` flat. Setup-mode properties trigger every month for 3+ months. Maturity gate + ≥30% threshold + 60-day-newness bypass eliminates the false-positive without losing the real signal. Same discipline as DA-06's anomaly callout.

The entire hero block is tappable → opens detail bottom sheet showing full inflow/outflow breakdown (mirrors Section A + Section B in expanded form).

**Multi-property (Priya):** Subtitle reads *"₹2.85L in − ₹2L out · across 3 properties."*

---

### 2a. YTD Strip (placement varies by view)

A thin strip showing financial-year-to-date cumulative figures:

*"FY26 YTD: +₹4.2L net · ₹18.5L in − ₹14.3L out"*

**Placement (v1.2):**
- **Single-property:** below hero (high prominence — single-property operators don't need property triage first)
- **Multi-property:** below the By Property section (operator's primary triage need is property-spread; YTD becomes secondary at month-end)

Tappable → opens YTD detail (rerenders Section A + Section B for the FY scope).

> **Why placement varies (v1.2):** Second-round audit caught reading-order friction for Priya — multi-property operators want to see property spread BEFORE the YTD context. Single-property operators don't have a spread concern, so YTD goes higher.

> **Why YTD is in v1.1 (promoted from Phase 2):** Operator audit identified this as the most-requested feature at FY-end (March 31). Zero new aggregation infrastructure — same queries with FY scope. High value at fiscal close, near-zero engineering cost.

> **FY definition:** Indian fiscal year (April 1 – March 31). Auto-rolls based on current date.

---

### 2b. Excel CTA (overflow menu) + screen footer hint

The Excel report download is accessed via the screen's overflow menu (⋮) — not a permanent hero chip. Menu item: *"📊 Email Cash Flow Excel"* — triggers existing `/generateCashFlowReport` endpoint. Toast on tap: *"Cash Flow Excel sent to [email]. You'll receive it in a few minutes."*

**Discoverability footer (small text at bottom of screen):**
*"Need a detailed report for accounting? Email Cash Flow Excel from menu (⋮)."*

> **Why overflow, not hero chip (v1.1 calibration):** 95% of operators don't email Excels. Of those who do, 90% only at month-end. Permanent hero chip wasted prime real estate. v1.1 moved it to overflow menu.

> **Why discoverability footer (v1.2 addition):** Second-round audit caught the CA-screenshot persona-stress-test failure — accountant viewing a forwarded screenshot has no way to know the Excel exists. Footer hint is visible in screenshots, addresses the case without re-cluttering the hero.

---

### 3. By Property (multi-property only — appears here, before Operating Cash Flow)

**The question this answers:** "Which property is profitable, and which is bleeding?"

For operators with >1 selected property, this section appears immediately below the hero. Single-property operators don't see it.

**Property spread one-liner (above the row list — adaptive format):**

| Case | Format |
|------|--------|
| ≥ 3 properties, mixed positive/negative | *"Best: Property 3 (+₹140K) · Worst: Property 2 (−₹15K) · Spread: ₹155K"* |
| ≥ 3 properties, all positive | *"Top: Property 3 (+₹140K) · Lowest: Property 1 (+₹50K) · Range: ₹90K"* (replaces "Worst" with "Lowest" when no losses) |
| ≥ 3 properties, all negative | *"Least bad: Property 3 (−₹5K) · Worst: Property 2 (−₹35K) · Spread: ₹30K"* |
| 2 properties | *"Property 1 (+₹85K) vs Property 2 (−₹15K)"* (collapses Spread; same info, less redundancy) |
| 1 property | (one-liner suppressed; By Property section also suppressed for single-property) |

Tappable → opens the property in the worst position (or "Lowest" / "Least bad" depending on case).

> **Edge case handling (v1.2):** Earlier v1.1 used "Best · Worst · Spread" universally. Second-round audit caught format degradation on all-positive (calling +₹50K "Worst" is misleading) and 2-property cases (Spread is just |difference|). v1.2 adapts the label per case.

Each row contains:
- Property name
- **Net Operating Cash Flow** for the period — sign-coded color
- Inflows / Outflows totals: *"₹95K in − ₹105K out"*
- Tenant-night count or occupancy share (informational)
- Proportional bar (relative magnitude)

Sorted by Net Cash Flow ASC (most negative first — operators triage red first).

**Color cue on Net Cash Flow column:**
- Positive ≥ 10% of inflows = green
- Positive < 10% = amber (thin margin)
- Negative = red

Tap any row → opens that property's full Cash Flow detail (rerenders hero + Section A + Section B for just that property).

---

### 4. Section A — Operating Cash Flow

**The question this answers:** "What were the components of my Net Operating Cash Flow?"

Two sub-blocks: Inflows (top), Outflows (bottom). Each line is tappable → bottom sheet drill.

#### Inflows (Operating)

| Line | What it covers | Source |
|------|----------------|--------|
| Rent collected | Long-stay + short-stay rent | DA-02 net, `due_type = 'Rent'`, `payment_mode NOT IN (211, 288)` |
| Tenant utility bills collected | Electricity, food/mess, maintenance bills paid by tenants — NOT operator's own utility expenses | DA-02 net, `due_type` NOT IN ('Rent', 'Security Deposit', 'Caution Money', 'Advance'), `payment_mode NOT IN (211, 288)` |
| Advance received (conditional) | New advance balances paid in by tenants — only renders when ≥ 10% of total inflows | DA-02 net, `due_type = 'Advance'`, `payment_mode NOT IN (211, 288)` |
| **Total Operating Inflows** | Sum of above | |

> **Display logic:** Only show lines with amount > 0. Advance is folded into "Tenant utility bills collected" when below 10% threshold (rare for it to dominate). Bottom sheet on Tenant utility line shows the sub-split.

> **"Tenant utility bills collected" naming (v1.1 disambiguation):** Earlier draft used "Other charges (Electricity, Food, Maintenance)" — operators saw "Electricity" on both inflow AND outflow sides and got confused. The renamed label disambiguates: utility bills *collected from tenants* (revenue) vs utility bills *paid by operator* (expense, in Outflows). Same concept, different direction.

> **Why advance is in operating inflows, not deposits:** Advance is a pre-payment for future rent — operationally, it converts to revenue when applied. Treating it as a liability separate from operating obscures the working-capital picture.

#### Outflows (Operating)

| Line | What it covers | Source |
|------|----------------|--------|
| Salary | Staff salaries paid | DA-04 `expense_type ILIKE 'Salary%'` |
| Electricity (paid) | Power bills | DA-04 `ILIKE 'Electricity%'` |
| Mess / Food (paid) | Kitchen, groceries | DA-04 `ILIKE 'Mess%' OR 'Food%'` |
| Maintenance (paid) | Repairs, AMC | DA-04 `ILIKE 'Maintenance%'` |
| Building rent paid | Operator's lease | DA-04 `ILIKE 'Rent%'` |
| Other expenses | Everything else (NOT 'Deposit%') | DA-04 NOT matching above patterns AND NOT 'Deposit%' |
| Tenant refunds | Non-deposit refunds (deposit refunds are in Section B) | DA-03 owner-funded, `invoice.due_type` NOT IN ('Security Deposit', 'Caution Money') |
| **Total Operating Outflows** | Sum of above | |

> **Critical: 'Deposit%' expenses are NOT in operating outflows.** They're either (a) a duplicate of a Refund row (the production double-count bug — see Pre-Launch Engineering Blocker B1) or (b) a real deposit refund that belongs in Section B. Either way, excluded from operating expenses. See EC-01.

> **Per-row MoM delta:** **Suppressed.** Operator audit on DA-05/06 confirmed per-row MoM for low-N data is noise. Operating Cash Flow has 6–8 lines; per-row chips would clutter without diagnostic value. Trend chart (Section 6) handles trajectory.

> **No Section A footer total (v1.1 calibration):** Earlier draft repeated "Net Operating Cash Flow: +₹85,000" at the end of Section A. Operator audit cut it as redundant — operator just looked at the same number on the hero.

#### Section A footer callouts (capped at ONE visible chip + ONE vendor flag)

Conditional, only when relevant. Tappable for full context. Priority: vendor concentration > capex flag > discount memo (most operationally critical first).

- **Vendor concentration flag (v1.2 promoted from Phase 2):** *"⚠️ [Vendor Name] received ₹X (Y% of total outflow this period). Worth checking."* — tap → opens DA-04 worklist filtered to that `paid_to`. Fires when:
  1. Top vendor (by `SUM(amount) GROUP BY paid_to`) > 30% of total outflow
  2. AND property has ≥ 5 expense entries in period (maturity gate — prevents firing on setup-mode properties with 1-2 expenses)
  3. AND amount ≥ ₹15,000 (absolute floor)
- **Capex flag chip:** *"⚠️ Large 'Other' expense — ₹50K furniture purchase"* — tap → opens that expense detail. Fires when "Other" expenses > 25% of total outflow AND any single expense > ₹50K.
- **Discount memo chip:** *"ⓘ ₹13K discounts already deducted"* — tap → reveals: *"Of which ₹2K were RentOk-funded — no impact on your revenue. See [[DA-05 Discounts Detailed Analytics]]."*

> **Vendor concentration promoted from Phase 2 (v1.2):** Second-round operator audit re-evaluated the v1.1 deferral and found it indefensible. Data is canonically available (`paid_to` field on Expenses). Effort is LOW (same query DA-04 already runs). Operator value is HIGH — single-vendor concentration is THE classic embezzlement pattern in property management; Priya's primary fraud signal at month-end.

> **One memo at a time:** Earlier v1.0 draft had multiple stacked memos. Operator audit caught the wall-of-text problem from DA-05 v1.0 repeating. Capped at one visible chip per fire condition. Vendor flag has its own slot because it's a distinct fraud-signal class — but in practice, all three rarely fire simultaneously.

---

### 5. Section B — Deposit Money In/Out

**The question this answers:** "How did my deposit obligations change this period?"

Section B subtitle: *"Not part of your earnings — you'll return this."*

This section is informational. It does NOT contribute to the hero number.

| Line | What it covers | Source |
|------|----------------|--------|
| Deposits collected | New SD/CM paid in by tenants | DA-02 net, `due_type IN ('Security Deposit', 'Caution Money')`, `payment_mode NOT IN (211, 288)` |
| Deposits returned | Deposit refunds disbursed | DA-03 owner-funded, `invoice.due_type IN ('Security Deposit', 'Caution Money')` (deduped against any 'Deposit%' expense entries — see EC-01) |
| **Net Deposit Flow** | Inflow − Outflow | |

A small "View deposit balance →" CTA at the right of Section B header opens [[DA-06 Liabilities Detailed Analytics]] for the same period. The header itself (label + Net Deposit Flow) is NOT tappable to prevent accidental navigation while operator is scanning.

> **Tappability scoping (v1.2):** Earlier v1.1 made the entire Section B header tappable. Second-round operator audit caught the side effect — operators scanning the section accidentally landed on DA-06 mid-task and had to navigate back. v1.2 limits navigation to an explicit CTA.

> **Title naming (v1.1 calibration):** Earlier draft used "Deposit Movements (Informational, Non-Revenue)" — operator audit caught the GAAP-speak. Renamed to operator-readable "Deposit Money In/Out" with the explanatory subtitle. The cross-screen reconciliation memo prose was replaced by making the section header tappable to DA-06.

> **Why this is its own section, not folded into operating:** Operators who collect ₹2L in deposits in a month and pay ₹3L in expenses see "Net Operating Cash Flow −₹1L (red)" and panic. But the deposit money is in the bank — they have liquidity, just not profit. Showing both numbers separately preserves both signals: profit (operating) and liquidity (deposits).

---

### 6. Cash Flow Trend Chart (always visible)

Below all sections, a chart is always visible.

**Stacked column chart (6 months):**
- Green column: Operating Inflows (deposits excluded)
- Red column: Operating Outflows
- Horizontal zero line for visual anchoring

Time selector: `6M` (default) · `This Month`

> **Why stacked, not stacked + overlay line (v1.1 simplification):** Earlier draft added an overlay net-cash-flow line on top of the stacked columns. Operator audit caught visual noise — operator can read "is green > red" visually as the gap. The trend insight text below already states the net direction in words. Dropped the overlay line.

**Trend insight text (conditional, below chart — only when applicable):**
- *"Costs are growing faster than collections — Outflows up 18% vs Inflows up 8% this period."* (When out-growth-rate > in-growth-rate by >5 pts over 6M)

> **Single insight only (v1.1 calibration):** Earlier draft had 4 conditional copy lines. Operator audit kept only the one that's genuinely actionable (cost-vs-revenue growth divergence). Other copy variants restated what bars already showed.

Bars tappable → tooltip showing exact inflow / outflow / net for that month. Tap bar → opens that month's full Cash Flow view (rerenders hero + sections for that month).

---

## Drilling Down: Bottom Sheets

**Every line item on the hero, Section A, Section B opens a bottom sheet** (per the user's design pattern).

### Bottom sheet structure (consistent across line items)

1. **Sheet header** — line item name + period total (matches what was tapped)
2. **Sub-categorization** (if any) — e.g., "Rent Collection" splits into Long-Stay / Short-Stay / Food-Rent
3. **Top 5 contributing entities** — top 5 paying tenants for inflow lines, top 5 expense categories or vendors for outflow lines, top 5 refund recipients for refund lines
4. **Cross-screen CTA** — "View all in [DA-02 / DA-03 / DA-04 / DA-06]" — opens the source DA screen with matching filter

> **No per-line 6M mini-trend (v1.1 calibration):** Earlier draft included a 6-month mini-trend bar in every bottom sheet. Operator audit caught the engineering scope creep — 14 sheets × mini-chart = brittle UI maintenance when sibling specs shift. Main trend chart on the screen already provides 6M context. Bottom sheets stay lean: top-5 entities + cross-screen CTA only.

### Per-line-item bottom sheet content

| Hero / line item | Bottom sheet content |
|-------------------|---------------------|
| Hero (Net Cash Flow) | Full Section A + Section B breakdown in expanded form |
| (MoM chip with sign reversal coloring) | (no separate sign-flip bottom sheet — the MoM chip itself encodes the reversal) |
| Deposit-dependency callout | DA-06 Liabilities snapshot for the period |
| **Section A: Rent collected** | Long-Stay / Short-Stay / Food-Rent split · Top 5 paying tenants · "View in DA-02" |
| **Section A: Tenant utility bills collected** | Sub-split by `due_type` (Electricity / Food / Maintenance / etc.) + Advance sub-line if applicable · Top 5 tenants · "View in DA-02" |
| **Section A: Advance received** (only when standalone — ≥10% of inflows) | Tenants who paid advance · "View in DA-02 with Advance filter" |
| **Section A: Salary outflow** | List of staff salary entries · "View in DA-04 with Salary filter" |
| **Section A: Electricity outflow** | List of electricity bill entries · "View in DA-04" |
| **Section A: Mess/Food outflow** | List of mess/food expense entries · "View in DA-04" |
| **Section A: Maintenance outflow** | List of maintenance entries · Top 5 vendors · "View in DA-04" |
| **Section A: Building rent outflow** | List of building rent payments · "View in DA-04" |
| **Section A: Other expenses** | Free-text expense entries not matching above · Capex flag if applicable · "View in DA-04 — Other category" |
| **Section A: Tenant refunds** | List of non-deposit refunds · Source split (owner / RentOk-funded) · "View in DA-03" |
| **Section B: Deposits collected** | Tenant-by-tenant list · Footnote: "Held as liability — see DA-06" · "View in DA-02 with Deposit filter" |
| **Section B: Deposits returned** | Tenant-by-tenant list · Owner-funded vs RentOk-funded split · "View in DA-03 with Deposit filter" |
| **Trend chart bar** | That month's hero + Section A + Section B breakdown |
| **By Property row** | That property's full Cash Flow rerendered |

> **Bottom sheet design constraint:** Each sheet is ≤ 8 visible items above the fold. Mobile screen real estate is limited; depth lives in the cross-screen DA navigation, not in cluttered sheets.

---

## Worklist Pre-sets (for cross-screen navigation)

Cash Flow doesn't have its own worklist — it dispatches to other DA screens for transaction-level views. Each tap from a bottom sheet opens the corresponding DA screen with a pre-set filter:

| Source | Destination | Filter |
|--------|-------------|--------|
| Hero / Operating Inflows | DA-02 Collections worklist | `payment_mode NOT IN (211, 288)`, period match |
| Section A: Rent | DA-02 worklist | `due_type = 'Rent'`, period match |
| Section A: Tenant utility bills collected | DA-02 worklist | `due_type` NOT IN deposit/advance/rent set |
| Section A: Advance received | DA-02 worklist | `due_type = 'Advance'`, `payment_mode NOT IN (211, 288)` (excludes mode-288 advance-applied) |
| Section A: Tenant refunds (non-deposit) | DA-03 worklist | `invoice.due_type` NOT IN ('Security Deposit', 'Caution Money') |
| Section A: Salary/Electricity/etc. | DA-04 worklist | category filter pre-set |
| Section B: Deposits collected | DA-02 worklist | `due_type IN ('Security Deposit', 'Caution Money')`, `payment_mode NOT IN (211, 288)` |
| Section B: Deposits returned | DA-03 worklist | `invoice.due_type IN ('Security Deposit', 'Caution Money')` |

> **No standalone DA-07 worklist.** The screen is composition + drill — transaction-level lists live in source DA screens. This prevents duplicating worklist UX across the suite.

### Bulk Actions (Excel CTA on hero)

- **Email Cash Flow Excel** — triggers existing `POST /reports/cashflow-report`. The current Excel is a transposed pivot (months as columns) with all categories. Pre-launch engineering must apply the bug fixes in this PRD before the Excel matches the screen. Toast: *"Cash Flow Excel sent to [email]. You'll receive it in a few minutes."*

---

## Drill-Down Behavior

> **Universal navigation rules apply to all 7 DA specs.** Priorities (P0/P1/P2) are PM recommendations; engineering may re-prioritize during spike based on existing app capabilities and effort.

### Universal Rules

**R1. Modal/Sheet/Screen primitive [P0]** *(updated 2026-05-11)*
Every tap target's destination is explicitly typed: full-screen push, bottom sheet, modal overlay, or inline accordion. **ⓘ icon convention (locked across DA suite):** single-tap → bottom sheet with plain-English explanation + GAAP framing. No inline tooltip. No long-press. See `[[_Build Sheet Generation Spec#15. ⓘ Icon Interaction Convention]]`. **Note for DA-07:** the v1.2 audit concern about CA-screenshot losing long-press content is moot under this convention — there is no long-press; GAAP framing must live in visible subtitle on the screen itself.

**R2. Back-stack semantics [P0]**
Back pops one navigation frame and restores the prior frame's filter chips, scroll position, accordion state, and selected segments. iOS swipe-back and Android system back behave identically.

**R3. Deep links + share sheet [P1]**
Every drill state is uniquely URL-addressable as `rentok://da-NN/<view>?<filters>`. Push notifications, WhatsApp reminders, and the in-app "Share this view" affordance generate these URLs. Cold-start with deep link bypasses tabs and lands directly in the drill state.
> Engineering note: existing app uses singular-form deep links (e.g., `rentok://collection`). New convention should align with existing router patterns or document migration path.

**R4. Permission gating UX [P0]**
Sections without read permission HIDE (not gray). Disabled actions show a lock icon; tap → toast naming the missing permission. Cross-screen drill into a denied screen shows a full-screen denial; back returns to the source bottom sheet. Mobile must inspect `can_view_*` flags from homepage response before showing drill targets as tappable; backend returns `{status: 401, data: <empty>}` if endpoint is hit without permission.

**R5. Loading states [P0]**
Every screen has skeleton-on-load (not spinner). Cross-screen navigation transitions show the destination header immediately + skeleton rows for content. Pull-to-refresh shows the chevron without flashing skeletons.

**R6. State preservation [P1]**
DA tab switches preserve per-tab filter and scroll state. App background/foreground within 15 min restores exact drill state. Force-quit + reopen returns to homescreen. Token expiry triggers silent re-auth + return-to-drill.
> Engineering note: 15-min restoration threshold is a PM recommendation; calibrate during spike based on session-data analytics and React Navigation persistence cost.

**R7. Multi-property scope inheritance [P0]**
Single-property drill OVERRIDES global scope for the entire descendant drill stack. Cross-screen CTAs inherit single-property scope. The scope chip is always visible in the worklist header.

**R8. Worklist filter-chip behavior [P0]**
Every pre-applied filter is a removable chip. Removal re-fetches without that filter; other filters stay. New filters are additive (AND). Worklist filter changes do NOT propagate back to the dashboard.

**R9. Shareable state [P2]**
Every dashboard, worklist, and detail has a "Share this view" affordance in the overflow menu (⋮). Generates a deep link + system share sheet. Excel export (where it exists) remains for deep-analysis; deep link is for live-data sharing.
> Engineering note: net-new feature beyond Excel export. Defer to Phase 2 if app router work is not in this release scope.

**R10. CA-screenshot discipline [P0]**
Every hero has a visible GAAP subtitle and basis label (cash/accrual/snapshot) — never tap-only. Every screen with an Excel CTA has a footer hint visible in screenshots pointing to it. No critical context in tap-only tooltips.

**R11. Cross-screen back path [P1]**
Cross-screen navigation (DA-X → DA-Y via "View in DA-Y" CTA) pushes the destination as a CHILD of the source's bottom sheet. The destination shows a breadcrumb header "← From [Source DA name]". Tap breadcrumb → returns directly to the source bottom sheet. After ≥3 hops, breadcrumb collapses to "← Back to start."
> Engineering note: introduces breadcrumb UX pattern. If app doesn't have breadcrumb support today, fall back to standard back-stack pop (R2) for v1; revisit in v1.1.

### Priority Summary

| Priority | Rules | Engineering guidance |
|----------|-------|---------------------|
| **P0 (must have)** | R1, R2, R4, R5, R7, R8, R10 | Universal mobile expectations; broken UX or persona regression without |
| **P1 (should have)** | R3, R6, R11 | Significant UX value; depends on existing app router/persistence patterns |
| **P2 (defer if needed)** | R9 | Net-new feature beyond current Excel export; Phase 2 candidate if scope-constrained |

### Per-Spec Specifics (DA-07)

- **Hero block tap** opens the detail bottom sheet showing full Section A + Section B expanded form. Bottom sheet is dismissible by swipe-down or tap-outside.
- **All 14 bottom sheets** follow the same template: header (line item + period total) → sub-categorization → top 5 contributing entities → cross-screen "View in DA-X" CTA. Bottom sheets do NOT navigate away on tap; only the explicit CTA triggers cross-screen drill.
- **Cross-screen CTAs from bottom sheets (R11 applies):** push destination DA worklist as a CHILD of the source bottom sheet. The destination shows breadcrumb "← From Cash Flow." Back from worklist returns to the bottom sheet (still open).
- **DA-07 Section B "View deposit balance →" → DA-06 (special case):** DA-07 is period-based, DA-06 is live snapshot. CTA opens DA-06 with **a "viewing as of [period end date]" pill** in the header; DA-06 displays held-as-of-period-end if historical data exists, else falls back to live with explanation.
- **Trend chart bar tap → opens that month's hero + sections** rerendered (period override on destination). Source filter chip stays unchanged on parent.
- **YTD strip tap → opens YTD detail screen** (re-renders Section A + B for the FY scope). Back returns to DA-07 hero.
- **Excel CTA in overflow menu (⋮)** → email-based export via existing `/reports/cashflow-report` endpoint. Toast confirmation, no navigation.
- **Discoverability footer** is informational only (no tap target) — visible in screenshots so CA reading screenshot knows the Excel exists.

---

### Universal Rule Clarifications (post-orphan-audit)

Resolves interaction-primitive ambiguities surfaced during the post-master orphan-tap-target audit. Treat as additive to R1–R11 above.

**R1 clarification — explicit primitives for common UI:**
- **Hero ⓘ icon:** single-tap = inline tooltip (one-line plain-language definition); long-press = bottom sheet with full GAAP framing + basis label.
- **Hero number long-press:** bottom sheet with full computation breakdown (Inflows ± Outflows = Net).
- **MoM chip with sign-reversal coloring:** tap = inline tooltip showing prior-period numbers + computation window. No drill (sign-flip is rendered inline, no separate sheet).
- **Per-bed Net Cash Flow chip:** informational only. No tap target.
- **Mid-month timing nudge text:** informational only. No tap target.
- **Discoverability footer:** informational only. No tap target.

**R5 clarification — pull-to-refresh:** Dashboard re-fetches all sections in parallel; bottom sheets re-fetch on pull. Cross-screen drill destinations also support pull-to-refresh. Chevron animates without flashing skeletons.

**R8 clarification — filter chip ✕ as explicit tap target:** Every chip has a discrete ✕ icon (44pt min hit area). Tapping ✕ removes that chip and re-fetches; other chips stay. Body-tap opens edit affordance where applicable; otherwise no-op.

**R12 (NEW) — Trend chart conventions [P0]:**
- **Bar single-tap (6M / This Month):** inline tooltip showing exact inflow / outflow / net for that month. No drill.
- **Tap-into-tooltip CTA "View [month] →":** drills to that month's full Cash Flow view (hero + sections re-rendered with that month's data + period override pill).
- **Period selector toggle (`6M` / `This Month`):** tap to switch chart range. Selected state preserved on back. Does NOT affect dashboard hero values.

### Updated Priority Summary

| Priority | Rules | Engineering guidance |
|----------|-------|---------------------|
| **P0 (must have)** | R1, R2, R4, R5, R7, R8, R10, R12 | Universal mobile expectations |
| **P1 (should have)** | R3, R6, R11 | Significant UX value |
| **P2 (defer if needed)** | R9 | Net-new beyond Excel export |

### Section A Footer Callouts — Distinct Tap Targets

The Section A bottom sheets render footer callouts that were under-spec'd. Disambiguated:

| Footer callout | Tap behavior | Destination |
|----------------|--------------|-------------|
| Vendor concentration "⚠️ 3 vendors = 60% of outflow" | Single-tap | DA-04 worklist filtered to those 3 vendors (cross-screen drill, R11 breadcrumb) |
| Capex flag "⚠️ ₹2.4L flagged as capex" | Single-tap | DA-04 worklist filtered to `is_capex=1` |
| Discount memo "Owner-funded discounts: ₹X" | Single-tap | DA-05 worklist for that period |

### Bottom Sheet Sub-Row Drills

Inside any of the 14 bottom sheets, sub-rows (e.g., "Long-Stay / Short-Stay / Food-Rent split" rows; "Top 5 contributing entities") are tappable:

- **Sub-categorization row tap** (Long-Stay/Short-Stay/Food-Rent): drills to the relevant DA worklist with that filter applied (R7 scope inheritance).
- **Top-5 entity row tap** (tenant/vendor/property): cross-screen drill to that entity's detail screen with breadcrumb "← From DA-07 Cash Flow" (R11).
- **"Property spread one-liner" tap:** opens By Property bottom sheet anchored to the worst-position property.

### Permission Vocabulary Reality Check

Codebase has only **11 JWT-mirrored permission keys**: `appAccess, cashCollection, recordPayment, editInvoices, editTenants, viewInvoices, viewExpenses, deleteInvoices, addTenants, deleteTenants, viewTenants` (mirror sites: `src/v1/login/property/service.ts:79-91` + `src/controllers/property.ts:14883/15041/15767/17813` + `src/helpers/teamMember.ts:134`). DB-side via `checkAuthInDb`: ~70 snake_case columns on `team_member_property`.

Keys cited in this spec vs codebase reality:

| Cited key | Status | Recommended Phase 1 path |
|-----------|--------|--------------------------|
| `viewCashFlow` (cited) | DOES NOT EXIST as screen permission | DB has `cashflow_report` (Excel export toggle, NOT screen perm). Build new column for screen access OR reuse composite gate `(viewInvoices AND viewExpenses)` since cash flow = invoices ± expenses |
| `cashflow_report` | EXISTS — DB column (via `checkAuthInDb`) | For Excel CTA gating only, NOT for screen access |
| `viewInvoices`, `viewExpenses` | EXIST — JWT keys | Recommended composite gate for screen access if Jatin chooses reuse path |

> **Decision owner: Jatin (Sr Backend).** Each MISSING key requires build-or-reuse decision. Specs cannot ship until Jatin signs off.

> **Filter code enum status:** `CashFlowFilterCode` does NOT exist. Range 1800-1899 is free. Phase 1 must build.

> **Excel report endpoint reality (corrects earlier draft):** `POST /reports/cashflow-report` ALREADY EXISTS at `src/routes/reports.ts:119`, HeaderValidator-protected. Phase 1 work on the Excel CTA is **column extension**, NOT new endpoint creation. Verify column shape against DA-07 sections + 14 bottom sheets — extend existing report rather than building parallel `/generateCashFlowReport`.

---

## Time Filter Behavior

| Component | Behaviour |
|-----------|-----------|
| Hero (Net Operating Cash Flow) | `paid_date` (inflows) and `paid_date` / `refund_date` (outflows) within selected period; `payment_mode NOT IN (211, 288)` |
| Per-bed Net Cash Flow (multi-property only) | Hero ÷ live occupied beds (live count, not period-scoped — matches DA-04 pattern). Hidden when single-property. |
| MoM chip | **This Month / Last Month:** "vs [MonthName]". **Custom range:** "vs equivalent prior period." **Prior period = ₹0:** "Not enough prior data." Same-elapsed-days rule applies. |
| Vendor concentration callout | Period-respecting (top vendor by `paid_to` aggregation within selected period) |
| Deposit-dependency callout | Period-respecting (deposit ratio computed within selected period) |
| By Property | Period-respecting; sorted ASC by net cash flow |
| Section A — Operating Inflows | `payment_mode NOT IN (211, 288)`, `paid_date` within period |
| Section A — Operating Outflows | `paid_date` (expenses) and `refund_date` (refunds) within period; expenses with `expense_type NOT ILIKE 'Deposit%'` |
| Section B — Deposit Money In/Out | Period-respecting; uses same date filters as Section A but for SD/CM `due_type` only |
| Trend Chart | Has its own 6M / This Month selector — not governed by global time filter |

**Custom future dates:** Show ₹0 for future portion. Helper: *"Cash Flow uses dates money moved. Future dates show ₹0."*

> **Period semantics match DA-02/03/04/05** (period-based). NOT live snapshot like DA-06. Cross-suite consistency.

---

## Edge Cases

**EC-01: Deposit refund logged as both Refund AND Expense (the production double-count bug)**
The most important edge case on this screen. The existing code at `generateCashFlowReport.ts:393` does `totalExpenses = expenseTotal + refunds` — same money counted twice when an operator records a deposit return as a Refund row AND as a `expense_type ILIKE 'Deposit%'` Expense.

DA-07 detection + handling:
- Section A Operating Outflows EXPLICITLY excludes `expense_type ILIKE 'Deposit%'`
- Section B (Deposit Money In/Out) unions Refunds + Deposit-named expenses but **dedupes** using DA-03 EC-15 heuristic (same pg_number + paid_to≈tenant_name + amount±5% + date±7d)
- If unmatched 'Deposit%' expenses exist (no corresponding Refund row), surface in Section B footer:
  → *"⚠️ ₹X in 'Deposit Refund' expenses don't have a matching Refund row. Your deposit outflow may be inflated by these — review in DA-04."*

**Engineering action:** Pre-launch fix at `generateCashFlowReport.ts:393` is mandatory. See Pre-Launch Engineering Blocker B1.

**EC-02: Mode 211 / 288 phantom inflow**
The existing report counts `invoice.amount` of mode-211/288 invoices in `Total Collection (+)` — these are paper transfers, no cash. DA-07 query MUST use `payment_mode NOT IN (211, 288)` filter. EC-02 is the spec's enforcement of this rule.

**EC-03: Mid-month timing illusion**
Salary pays first week, rent collection peaks 20–25th. Net Cash Flow on the 12th is unreliable. DA-07 doesn't auto-detect this, but the trend chart provides historical context — operator self-corrects. Phase 2: add "results unreliable until day 25" footer based on current day-of-month.

**EC-04: Discount memo when discounts > 0**
Memo annotation always shows below Section A when discounts in period > 0. Operator must understand that hero number is already net of discounts.

**EC-05: RentOk-funded discount with no make-whole**
RentOk-funded credits (`source = 1`) reduce what the tenant pays. There is NO settlement transferring cash from RentOk to the operator. Memo annotation: *"Of which ₹Y RentOk-funded — no impact on your revenue."* See codebase audit — no make-whole flow exists (Engineering Note 4).

**EC-06: Capex / large "Other" expense distortion**
A one-time furniture purchase (₹50K) makes Net Cash Flow look terrible that month and great the prior month. Conditional flag fires when "Other" > 25% of total outflow AND any single expense > ₹50K. Operator gets context.

**EC-07: All inflow is deposits, no operating revenue**
A property in setup mode with new tenants moving in: ₹2L in deposit collections, ₹0 rent. Net Operating Cash Flow may be negative (only expenses). Deposit-dependency callout fires. The screen correctly shows the operator hasn't made any operating revenue — this is the right signal for an early-stage property.

**EC-08: Net Cash Flow = exactly ₹0**
Inflows = Outflows. Hero shows `₹0` in neutral color. No MoM (would be ÷0 unless prior period also ₹0). Subtitle shows balanced totals: "₹X in − ₹X out." Doesn't error.

**EC-09: Period extends beyond today (custom range)**
Future dates show ₹0 contribution. Helper text in time filter chip clarifies. No special handling.

**EC-10: Property has no expenses but has collections**
Outflows = ₹0. Net Cash Flow = full inflow amount. Section A Outflows hidden. Per-bed cash flow shown. Edge case but legitimate (very small property, no recorded expenses yet).

**EC-11: Multi-property, one property has zero activity**
By Property breakdown shows that property at the bottom with ₹0 net cash flow. No special treatment.

**EC-12: Refund without linked invoice (DA-03 EC-05)**
Inherited from DA-03 — refund amount still counts. Linked invoice context shows "Linked invoice unavailable."

**EC-13: Same-period deposit collection AND refund for same tenant**
Tenant pays SD, immediately moves out, gets refund. Both events count: Section B inflow has the SD collection, Section B outflow has the refund. Net Deposit Flow = ₹0 for this tenant. Operating cash flow unaffected (deposits not in Section A). Correct accounting.

**EC-14: GST collected (commercial PG / >₹20L turnover)**
v1.0 doesn't surface GST. If operator is GST-registered, the inflow includes GST collected (which is a liability to government). Footer note (Phase 2): *"Of ₹X collected, ₹Y is GST — owed to government."*

**EC-15: Off-system owner withdrawal**
Operator takes ₹50K cash from the till for personal use, doesn't record. DA-07 cannot detect this. Hero overstates "money you made" by ₹50K. Disclaimer in tooltip: *"Doesn't include money you've withdrawn for personal use — record those as expenses if you want them tracked."*

---

## Words on the Screen

### Empty States

| When | Message | CTA |
|------|---------|-----|
| No transactions in period | "No cash movement in this period. Wrong period? Change the date filter." | "Change filter" |
| Custom future range | "Cash Flow uses dates money moved. Future dates show ₹0." | — |
| New property | "No cash movement yet. They'll appear here once you collect rent or pay expenses." | — |
| Excel report queued | "Cash Flow Excel sent to [email]. You'll receive it in a few minutes." | — |

### Error States

| When | Message | Recovery |
|------|---------|----------|
| Network failure | "Couldn't load Cash Flow. Check your connection." | "Retry" |
| Section fails | "Couldn't load this section." | "Retry" on that section |
| Excel generation fails | "Cash Flow Excel couldn't be generated. Please try again." | "Retry" |
| Cross-screen reconciliation diverges (Eng Note 1 unfixed) | (silent — show numbers as-is) | — |

### Hero ⓘ Tooltip
> "Money in vs. money out for the selected period. The hero shows net operating cash flow — what you made after expenses and refunds. Deposits are tracked separately because they have to be returned."

### Net Operating Cash Flow ⓘ Tooltip (long-press on hero number)
> "This is your operating cash profit — what's left after expenses and refunds. Security deposits aren't included because you have to give them back. For full reconciliation to your bank, download the Excel report."

### Per-Bed Cash Flow ⓘ Tooltip
> "Net cash flow ÷ currently occupied beds. Helps compare profitability across properties of different sizes. Occupancy count is current (live), not period-average."

### Operating Inflows ⓘ Tooltip
> "Cash actually received during the period — rent, electricity, food, advances. Excludes deposits (those are in Section B) and adjustments (no real cash moved)."

### Operating Outflows ⓘ Tooltip
> "Cash actually paid out — staff salaries, utilities, maintenance, refunds. Excludes deposit refunds (those are in Section B)."

### Deposit Money In/Out ⓘ Tooltip
> "Deposit money in vs. out. This is non-revenue cash — when you collect a deposit, you take a liability to return it. Tracked separately from operating cash flow."

### By Property ⓘ Tooltip
> "Each property's net cash flow this period. Sorted by lowest first so you can spot the property losing money. Tap any row to see that property's full breakdown."

### Trend Chart ⓘ Tooltip
> "Money in (green) and money out (red) for the last 6 months. Watch if the green is consistently above the red — that's a healthy business."

### Deposit-dependency callout copy
> "⚠️ ₹35K of inflow is deposits (held as liability). Net for the month excludes this: +₹50K."

### Vendor concentration callout copy (v1.2)
> "⚠️ [Vendor Name] received ₹X (Y% of total outflow this period). Worth checking."

### Mid-month timing nudge copy
> "💡 It's the [N]th — collections typically peak after the 20th. This may turn green by month-end."

### Loading States

| Component | Loading behaviour |
|-----------|-------------------|
| Hero | Skeleton matching number block + chip placeholders |
| By Property | 3 skeleton rows |
| Sections A & B | Header visible immediately; skeleton rows in content |
| Trend chart | Grey placeholder rectangle; loads after sections |
| Bottom sheets | Sheet skeleton + 5 placeholder rows |

---

## Decisions Documented (no old PRD to override)

Without an old PRD to critique, this section documents v1.0 decisions and the alternatives considered.

### 1. Persona — primary is operator, accountant is secondary reader

**Considered:** Build for accountant primary (matches user intuition).
**Decided:** Operator primary, accountant secondary.
**Why:** Mobile form factor = operator's pre-flight check. Accountant does the close in Excel on laptop. The Excel CTA is the bridge. Cross-suite consistency with DA-02–06 operator-Hindi-English voice.

### 2. Hero — single number, sign-coded

**Considered:** Two numbers (Operating + Deposit), or three (Inflow / Outflow / Net).
**Decided:** Single Net Operating Cash Flow.
**Why:** Mobile = one primary number. Composition surfaces in Sections A/B below. Sign is the actionable signal — magnitude is context.

### 3. Deposits — Section B (separated), not folded

**Considered:** Strict 3-bucket GAAP (Operating/Investing/Financing) — accountant-correct but UI-heavy. Or fully flat (deposits as just-another-inflow) — simpler but creates "I made ₹2L from deposits!" trap.
**Decided:** Hybrid 2-section structure (Operating + Deposit Money In/Out).
**Why:** Operators conflate deposit cash with revenue if not visually separated. CA-defensible separation, operator-readable simplicity.

### 4. Discounts — memo only, no cash flow line

**Considered:** Show as outflow line. Show as separate "revenue forgone" section.
**Decided:** Memo annotation only. No cash flow line.
**Why:** Discounts reduce revenue at point of sale (already netted in DA-02 hero). They are NOT cash leaving. Putting them as an outflow double-counts. Standard accounting also excludes them from cash flow statements.

### 5. Mode 211 / 288 — excluded entirely

**Considered:** Show as separate "Adjustments" line.
**Decided:** Excluded from inflows and outflows. Maybe surface in DA-02 Collections worklist as a separate row.
**Why:** No cash actually moved. Including them as a line would imply cash flow when there is none. Pure cash-basis discipline.

### 6. RentOk-funded discounts — zero impact, memo flag

**Considered:** Show as inflow (assumes RentOk pays operator).
**Decided:** No inflow line. Memo annotation only.
**Why:** Investigation confirmed no make-whole settlement exists. RentOk eats the cost as marketing; operator collects less. No cash transfer to operator's books.

### 7. Opening / closing balance — not feasible

**Considered:** Operator-entered opening balance. Bank API integration.
**Decided:** Not in v1. Frame screen as "movement" not "position."
**Why:** RentOk doesn't track operator cash position. Wallet balance is RentOk's collection float. Phase 2 if needed.

### 8. Trend chart — stacked dual columns (no overlay line, v1.1 simplified)

**Considered:** Single net line. Three separate lines (in/out/net). Stacked columns + overlay net line (initial v1.0 draft).
**Decided:** Stacked columns (Inflows green / Outflows red) only — no overlay line.
**Why:** Composition AND direction matter. Stacked alone shows both (operator reads "is green > red" visually = net direction). Initial v1.0 draft added an overlay net line; first-round operator audit caught visual noise on mobile, stripped the overlay. Trend insight text below chart states net direction in words. This is the one DA suite screen where stacking earns its keep (DA-04/05/06 use simpler bar charts).

### 9. Hero label naming — "Net for the Month" + GAAP subtitle

**Considered:** "Net Operating Cash Flow" only (GAAP, accountant-friendly). "Money you made this month" only (operator-natural). "Profit" / "P&L" framing (rejected — not a true P&L, no accruals).
**Decided (v1.2):** Visible primary label "Net for the Month" (operator-readable). Visible secondary subtitle "Net Operating Cash Flow" (CA screenshot use case). Long-press tooltip explains both terms.
**Why:** The bilingual approach landed across two operator audit rounds. v1.0 used "Net Operating Cash Flow" → too GAAP. v1.1 used "Net for the Month" with "Money you made" only in tooltip → CA screenshot reader couldn't long-press, lost GAAP framing. v1.2 keeps the operator-friendly headline AND surfaces GAAP as a small subtitle line — both personas served on a single screen.

### 10. Investing / Financing GAAP buckets

**Considered:** Separate Investing bucket for capex.
**Decided:** Folded into Operating Outflows; conditional flag if > 25% of outflow.
**Why:** Capex is rare in PG context (< 5% of outflow typically). Empty Investing bucket trains users to ignore. Conditional flag preserves anomaly detection.

### 11. GST handling

**Considered:** Net out GST consistently across all due_types. Surface as separate row.
**Decided:** Skip v1. Phase 2 if commercial PG.
**Why:** Residential PG (95%+ of operators) is GST-exempt below ₹20L turnover. Surfacing GST adds clutter for the dominant case.

### 12. Owner withdrawals

**Considered:** Manual entry feature.
**Decided:** Not in v1. Disclaimer only.
**Why:** No schema exists. Operators don't record drawings. Phase 2 idea: "Owner Withdrawal" expense category.

---

## v1.0 → v1.1 → v1.2 Calibrations (post 2 operator audit rounds + Figma audit)

### v1.1 → v1.2 changes (second-round operator audit)

| v1.1 (first audit fixes) | v1.2 (second-round corrections) | Why |
|---|---|---|
| Hero: "Net for the Month" only as visible label, GAAP framing in long-press tooltip | Hero: "Net for the Month" headline + visible "Net Operating Cash Flow" subtitle | CA screenshot persona-stress-test failed in v1.1 — accountant viewing forwarded screenshot can't long-press. v1.2 surfaces GAAP framing as visible subtitle. |
| MoM chip 4 states (improvement / regression >10% amber / regression ≤10% neutral / sign reversal red) | MoM chip 3 states (improvement green / regression amber / sign reversal to negative red) | Neutral-grey state was indistinguishable from "no chip rendered" (no prior data) — operator audit caught the ambiguity. |
| Sign reversal: chip color red regardless of magnitude | Sign reversal to positive → green; sign reversal to negative → red | **BUG FIX:** v1.1 said red regardless. Flip from −₹100 to +₹500 is good news (operator turned profitable) — should be green. |
| Vendor concentration deferred to Phase 2 | Vendor concentration callout in Section A footer (≥30% AND ≥5 entries AND ≥₹15K) | Second-round audit confirmed deferral was indefensible — data is canonically available, effort is LOW, operator value is HIGH (Priya's primary fraud signal). |
| Section B header tappable → DA-06 navigation | Section B header NOT tappable; explicit "View deposit balance →" CTA opens DA-06 | Operators scanning the section accidentally landed on DA-06 mid-task. |
| Property spread one-liner: "Best · Worst · Spread" universally | Adaptive format: "Top · Lowest · Range" when all-positive; collapses for 2-property; suppressed for 1-property | "Worst" labeling for a +₹50K property was misleading; 2-property spread is just \|difference\|. |
| Mid-month nudge: day < 25 AND filter = "This Month" | Same + explicit clarification: only "This Month" preset, not custom range covering current month | Custom-range case was unspecified in v1.1; v1.2 makes the gating explicit. |
| Per-bed Cash Flow row in Time Filter Behavior table without multi-property gating | Per-bed row notes "(multi-property only)" + hidden when single-property | v1.1 fix landed in hero spec but not in Time Filter Behavior table — orphan. |

### v1.0 → v1.1 changes (first-round operator audit)

| v1.0 (initial draft) | v1.1 (locked) | Why |
|---|---|---|
| Hero label "Net Operating Cash Flow" + "Money you made" tooltip + period date subtitle | Hero label "Net for the Month" + long-press tooltip with GAAP terms + subtitle drops period (already in time filter chip) | Bilingual whiplash; pick one operator-facing voice; reduce subtitle redundancy |
| Standalone sign-flip callout ("⚠️ Cash flow turned negative") | Sign reversal encoded as red color on existing MoM chip (no separate callout) | Sign-flip callout fired every mid-month for every operator (alarm fatigue); MoM chip carries the same info already |
| Per-bed Cash Flow always shown | Multi-property only | Per-bed has no comparison value for single-property operator |
| Deposit-dependency callout: ≥ 25% of inflow | ≥ 30% AND ≥ 5 active tenants AND not within first 60 days from launch | Setup-mode properties triggered every month (false-positive); maturity gate matches DA-06 discipline |
| MoM amber threshold: any % regression | ▼ > 10% only (smaller regressions stay neutral) | Tiny regressions amber-flagged were noise |
| Excel CTA always visible on hero | Moved to overflow menu (⋮) | 95% don't use it; permanent chip wasted hero space |
| Section B title "Deposit Movements (Informational, Non-Revenue)" | "Deposit Money In/Out" with subtitle "Not part of your earnings — you'll return this." | GAAP-speak operator-blind; renamed to operator-readable |
| Section A "Other charges (Electricity, Food, Maintenance)" | "Tenant utility bills collected" | Operators saw "Electricity" on both inflow and outflow sides → confusion |
| Section A "Tenant refunds (operating)" parenthetical | "Tenant refunds" | Parenthetical was defending against confusion operator never has |
| Advance received as standalone inflow line | Folded into "Tenant utility bills collected" with sub-split in bottom sheet (only standalone when ≥ 10% of inflows) | Advance is < 5% of inflow most months for most operators |
| Section A footer "= Net Operating Cash Flow: +₹85,000 (matches hero)" | Removed | Redundant with hero — operator just looked at it |
| Multiple stacked memo annotations under Section A | One tappable chip (discount or capex, mutually exclusive — capex priority when both fire) | DA-05 v1.0 wall-of-text problem repeating; cap at 1 visible chip per section |
| Section B cross-screen reconciliation memo prose | Section B header tappable → DA-06 (no prose) | Engineering-speak removed; same navigation in two taps |
| Trend chart: stacked columns + overlay net line | Stacked columns only (operator reads "is green > red" visually) | Overlay line was visual noise; insight text below already states net direction |
| Trend insight text: 4 conditional copy variants | One variant only (cost-vs-revenue growth divergence) | Other 3 restated what bars showed |
| Bottom sheet per line: top-5 + 6M mini-trend + cross-screen CTA | Top-5 + cross-screen CTA only (no mini-trend) | 14 mini-charts = brittle maintenance; main trend chart provides 6M context |
| YTD section as Phase 2 | Promoted to v1.1 (thin strip below hero: "FY26 YTD: +₹4.2L net · ₹18.5L in − ₹14.3L out") | Most-requested at FY-end; zero new aggregation infra (same queries with FY scope) |
| Mid-month timing nudge as Phase 2 | Promoted to v1.1 (conditional, day-of-month < 25 + period = "This Month") | Zero engineering effort (string conditional); saves operator from mid-month panic |
| By Property: just the row list | List + property spread one-liner above ("Best · Worst · Spread") | Priya's primary triage need is the spread, not the list |
| Hindi framing "Kitna paisa aaya, kitna gaya, kya bacha?" | "Kitna kamaaya, kitna gaya, kya net hua?" | "Kya bacha" implies the whole pile remains; deposits don't really "remain" |

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
| ⛔ **CSB-8** (NEW 2026-05-11) | **Widget endpoint permission enforcement gap (security).** List-screen widget endpoints set up `self_added_team_uuid` for team members with restricted permission (only `view_invoices_of_self_added_tenants`) but never apply it as a WHERE clause to the widget query. Worklist paths enforce it; widget paths do not. | `src/v1/list_screens/dues/service.ts:132-138` (verified DA-01); likely also `collections/service.ts`, `expenses/service.ts`, `refunds/service.ts` — needs audit | Cross-tenant aggregate data leak via widget endpoints. Restricted team member sees correctly-scoped worklist but un-scoped widget totals. Privacy/security concern. | Add the same `andWhere('t.added_by_id = :self_team_uuid', ...)` enforcement to widget query paths. Audit all DA list-screen widget endpoints for the same pattern bug. |

⛔ = blocks DA-suite launch. CSB-1, CSB-2, CSB-3, CSB-4 must be resolved before Phase 1 ships. CSB-5/6/7 are app-hygiene; can ship as fast-follow but should be on Jatin's queue.

---

## Phase 2 — Intentionally Deferred

Ordered by operator-demand priority. Top 3 are what operators will likely ask for within 90 days of launch.

| Feature | Why |
|---------|-----|
| **1. Operator-entered opening balance** | Enables true cash position reconciliation. Phase 2 — manual entry. Phase 3 — bank API integration. The screen's full payoff (turns "movement" into "position"). |
| **2. Capex / Owner-withdrawal expense category tagging** | New `expense_category` enum on Expenses entity. Allows separating Operating vs Investing vs Financing outflows correctly. v1.1 surfaces capex as a chip; Phase 2 makes it a structural concept. |
| **3. Bank-account integration for true cash position** | API integration with operator's bank for live cash balance. Combined with #1 (operator-entered opening balance) enables true bank reconciliation. Phase 3 territory. |
| **GST surfacing for commercial PG** | When `gst_amount > 0` exists on invoices, add memo line for GST collected (liability) and GST paid (Input Tax Credit). |
| **Bottom sheet "View underlying transactions" inline list** | Currently bottom sheets show top 5 + cross-screen CTA. Phase 2: full transaction list inline (mirror DA-04 worklist). |
| **RentOk make-whole flow** | If RentOk-funded discounts ever transition to operator make-whole, surface as a separate inflow line. Codebase audit confirmed no make-whole exists today. |
| **Multi-property comparison trend overlay** | Currently trend chart aggregates across selected properties. Phase 2: 3-property comparison overlay (one line per property). |
| **Cash flow forecast (next 30 days)** | Mirror DA-06's Move-Out Forecast pattern. Predict next 30 days based on contract end dates + recurring expenses. |
| **Consolidated bank reconciliation** | Match per-mode collection (UPI / Bank Transfer / Cash) to operator's bank statement. Requires bank integration. |
| **Anomaly detection — month-over-month outliers** | "This month's electricity is 3x the 6M average." Per-category anomaly flagging. |
| **Prior-6M running average comparison on hero** | Beyond MoM, compare current period to 6M trailing average to catch regression-vs-trend signals. |
| **Bank-account integration for true cash position** | API integration with operator's bank for live cash balance. Phase 3 territory. |

> **v1.1 / v1.2 Phase 2 changes:**
> - **Removed from Phase 2 (promoted to v1.1):** YTD strip, Mid-month timing nudge, By Property spread one-liner.
> - **Removed from Phase 2 (promoted to v1.2):** Vendor concentration flag (data was always available — DA-04 `paid_to`).
> - **Currently in Phase 2:** Prior-6M running average comparison (defensibly deferred — hero is dense; add only after live data shows MoM-only is missing seasonal-divergence signal).

---

## Pre-Launch Engineering Blockers (re-ranked v1.2)

Second-round operator audit re-ranked v1.1's 5+4+4+1 to **3 hard + 1 dependency + 1 verification + 4 parity + 4 deferrable**. Cleaner classification — HB3 is owned by DA-06 (DA-07 ships when DA-06 ships); HB5 is a verification task folded into HB1's QA.

### HARD blockers (mobile screen would be wrong without these)

| Item | Why hard-block | File:Line |
|------|---------------|-----------|
| **HB1. Cash flow expense+refund double-count** | Affects DA-07 mobile Section A outflow math. Refunds double-counted once in Section A Tenant refunds line, once inside Total Operating Outflows. Same underlying codebase bug surfaced in DA-03 EC-15 (heuristic detection) + DA-04 HB3 (deduplication fix). **Note: this is DIFFERENT from "Refund→Credit-status integrity bug" tracked in DA-05 HB1 / EC-07 — that's a separate bug at `services/refunds/refunds.ts` where Credits.status isn't flipped after refund.** **Fix at source:** deduct `expense_type ILIKE 'Deposit%'` from Expenses aggregator (DA-04 pattern), treat Refunds as separate outflow line. **HB1 QA must also verify HB5** (RentOk-funded payment-mode handling — `Credits.source=1` already reduces `payment.net_amount` at source; verify no double-adjustment). | `src/services/reports/generateCashFlowReport.ts:393` |
| **HB2. Mode 211/288 phantom inflows** | Affects DA-07 mobile hero. Existing report counts `invoice.amount` of mode-211/288 invoices in inflow without checking payment_mode. Mobile inflow query MUST use `payment_mode NOT IN (211, 288)` filter. | `src/services/reports/generateCashFlowReport.ts:330-366` |
| **HB3. Single canonical inflow aggregation path** | Today: DA-02 widget query is invoice-first; DA-02 list query is payment-first; cash-flow report is invoice-first via `invoice.amount`. Three paths produce different totals for the same period. DA-07 hero MUST match DA-02 hero exactly — pick ONE canonical path and route DA-02 + DA-07 through it. | (architectural — multiple files) |
| **HB-P1 (NEW Build Sheet finding 2026-05-08). `viewCashFlow` permission key does NOT exist** | DA-07 hero needs view-permission gating per universal R4. JWT key `viewCashFlow` was cited in earlier drafts but does not exist in codebase (Field Map §3.4). DB column `cashflow_report` exists (`team_member_property`) but it's an Excel-export toggle, NOT a screen permission. **Recommend:** composite gate `viewInvoices AND viewExpenses` (since cash flow = invoices ± expenses) OR build new `viewCashFlow` JWT key + DB column. Jatin gate. | `src/v1/login/property/service.ts:79-91` (JWT mirror) + `src/entities/teamMemberProperty.ts` (DB column) |
| **HB-E1 (NEW Build Sheet finding 2026-05-08). List endpoint `/v1/list_screens/cashflow/list/filters` does NOT exist** | NEW BUILD required. Mirror pattern at `src/v1/list_screens/expenses/`. Must include filter codes, multi-property scoping, pagination, auth chain. | (NEW endpoint) |
| **HB-F1 (NEW Build Sheet finding 2026-05-08). `CashFlowFilterCode` enum does NOT exist** | NEW BUILD required. Range 1800-1899 is free per Field Map §2.8. Must define filter codes for: hero/section/category drill targets + cross-screen drills into DA-02/DA-04/DA-05/DA-06. | `src/v1/constants/filterCodes.ts` |
| **HB-D1 (NEW Build Sheet finding 2026-05-08). DA-06 historical mode for "View deposit balance →" CTA** | DA-07 Section B has CTA "View deposit balance →" routing to DA-06. DA-06 is currently fully-live snapshot (no historical mode). For period-anchored cash flow drill from DA-07, DA-06 needs to surface "viewing as of [period end]" pill OR a historical query path. Owned by DA-06 (HB4 Trend chart historical data source resolution unblocks this). | (cross-spec dependency on DA-06 HB4) |
| **HB-B1 (NEW Build Sheet finding 2026-05-08). Excel `generateCashFlowReport.ts:393` double-count bug** | Same root cause as HB1 above, but specifically the Excel rendering path. PB4 already covers; elevated here as Hard Blocker since Excel CTA at `/reports/cashflow-report` is the primary monthly export and produces wrong totals until fixed. | `src/services/reports/generateCashFlowReport.ts:393` |

### DEPENDENCY (owned by another spec; DA-07 ships when this lands)

| Item | Why dependency | Owner |
|------|----------------|-------|
| **DA-06 canonical liability query unification** | Required for cross-screen reconciliation Invariant I5 (Section B Net Deposit Flow ≈ ΔDA-06 liabilities held). DA-06 already lists this as its own pre-launch blocker. The fix lives in DA-06; DA-07 inherits the readiness. | DA-06 spec |

### PARITY blockers (mobile + Excel must show same numbers)

| Item | Why parity-block | File:Line |
|------|------------------|-----------|
| **PB1. DA-04 ILIKE category bucketing in Excel** | Existing Excel uses free-text `expense_type` strings as column keys. Mobile uses DA-04 ILIKE buckets. Same period would show different category counts in mobile vs Excel. Apply DA-04 ILIKE prefix matching to Excel. | `src/services/reports/generateCashFlowReport.ts:374-378` |
| **PB2. DA-03 refund category bucketing in Excel** | Existing Excel shows single `Refunds (-)` scalar. Mobile splits Refunds into Section A (operating) vs Section B (deposit). Excel should split refunds by `invoice.due_type` category to match. | `src/services/reports/generateCashFlowReport.ts:384-389` |
| **PB3. Discount visibility in Excel** | Existing Excel doesn't load `Credits` entity. Mobile shows discount memo chip. Excel should add memo section "Discounts given (revenue forgone)" with owner-funded vs RentOk-funded split. Informational only. | `src/services/reports/generateCashFlowReport.ts` |
| **PB4. Excel must drop deposit-name expenses from Operating Outflows** | Same fix as HB1, applied in Excel rendering. Section A outflow total in mobile will not match Excel "Total Expenses" until Excel deduplicates. | `src/services/reports/generateCashFlowReport.ts:393` |

### DEFERRABLE (can ship as Phase 1 hotfix post-launch)

| Item | Why deferrable | File:Line |
|------|---------------|-----------|
| **D1. Multi-month accumulator scoping bug** | Affects only multi-month Excel exports (most operators run monthly). Mobile is single-period. Hotfix safe. | `src/services/reports/generateCashFlowReport.ts:317-325` |
| **D2. Net Profit formula vs label mismatch in Excel** | Excel-only label fix. Doesn't affect mobile hero math. | `src/services/reports/generateCashFlowReport.ts:428` |
| **D3. Catalog `featureArray` mismatch** | Excel metadata only. Doesn't affect rendered report or mobile. | `src/controllers/reports.ts:8546-8562` |
| **D4. Dead `liabilitySum` variable** | Feature was never built. Not a regression. Cleanup. | `src/controllers/reports.ts:7495` |

### NOT in DA-07 scope (owned by other specs)

- **DA-03 refund-discount integrity bug** (was originally B11) — affects DA-05 more than DA-07 (operator audit confirmed). Let DA-03 / DA-05 own the fix; DA-07 surfaces the consequence in the discount memo.

---

---

## For Engineering

### Reconciliation Invariants

| Invariant | Formula |
|-----------|---------|
| **I1. Hero = Section A total** | Net Operating Cash Flow = Total Operating Inflows − Total Operating Outflows |
| **I2. Operating Inflows = DA-02 hero − new deposits collected this period** | Σ(Section A inflow lines) = DA-02 net collection − DA-06 new deposits in period |
| **I3. Operating Outflows = DA-04 hero − Deposit Refund expenses** | Σ(Section A outflow lines) = DA-04 expense total − Σ(`expense_type ILIKE 'Deposit%'`) |
| **I4. Operating refunds = DA-03 hero − deposit refunds** | DA-07 operating refund line = DA-03 owner-funded refunds − Σ(refunds where `invoice.due_type IN ('Security Deposit', 'Caution Money')`) |
| **I5. Section B Net = ΔLiability** | (Deposits collected − Deposits returned) for period ≈ DA-06 `total_held(period_end) − total_held(period_start)` ± timing variance |
| **I6. Discounts NOT subtracted** | DA-07 hero math MUST NOT include any discount-based subtraction. DA-05 hero is informational only. |
| **I7. Mode 211 + 288 excluded** | All inflow queries use `payment_mode NOT IN (211, 288)` |
| **I8. RentOk-funded zero impact** | DA-07 inflows use `p.net_amount` (already discount-adjusted). No further adjustment for source=1 credits. |
| **I9. Cash basis only** | Uses `paid_date` (payments / expenses) and `refund_date` (refunds). Never `due_date` or `invoice_date`. |
| **I10. Multi-property aggregates by sum** | Σ(per-property cash flow) = aggregate cash flow. No averaging. |
| **I11. Bottom sheet line counts match parent line totals** | Tap any line → bottom sheet sub-totals must sum to exactly the parent line amount |
| **I12. Trend chart bar (current month) = current period hero** | The current-month bar in the 6M trend chart equals the hero for "This Month" filter |

### Codebase Feasibility

> **Source:** `src/services/reports/generateCashFlowReport.ts`, `src/controllers/reports.ts`, plus DA-02/03/04/05/06 list_screens
> **Reviewed:** 2026-05-07

#### What Already Exists

| PRD Feature | Code Reference | Status |
|-------------|---------------|--------|
| Cash Flow Excel report endpoint | `POST /reports/cashflow-report` | **EXISTS** (with bugs — see Pre-Launch Blockers) |
| Property scoping pattern | `${pg_id}PG${pg_number}` | **EXISTS** |
| Inflow source: `Invoices` + `payments` | filtered by `paid_date`, `status=1` | **EXISTS** |
| Outflow source: `Expenses` filtered by `paid_date`, `is_active=1` | | **EXISTS** |
| Refund source: `Refunds` filtered by `refund_date` | | **EXISTS** |
| Mode 211/288 detection | `payment_mode == 211/288` checks | **EXISTS** (but inconsistently applied) |
| GST extraction helpers | `getGSTPercentFromDescription`, `getAmountWithoutGST` | **EXISTS** (rent-only application) |
| DA-02 net collection logic | DA-02 widget query | **EXISTS** |
| DA-03 owner-funded refund total | DA-03 hero | **EXISTS** |
| DA-04 expense category ILIKE matching | DA-04 `notOtherTypes` pattern | **EXISTS** |
| DA-06 canonical liability query | `src/v1/homepage/service.ts:1909-1922` | **EXISTS** (with payment_mode 211 bug — DA-06 pre-launch blocker) |
| Excel email delivery pattern | `ReportHelper.sendReport` | **EXISTS** |

#### What Needs New Build

| PRD Feature | Gap | Effort |
|-------------|-----|--------|
| **Cash Flow detail screen API endpoint** (`/v1/cashflow/list/filters`, `/v1/cashflow/widget`) | No list_screens module exists. Mirror the pattern from `expenses/helpers.ts`. | **MEDIUM** |
| **CashFlowFilterCode enum** | No filter codes. Suggested range 1701–1730. | **LOW** |
| **Section A Operating Inflows aggregation** | New GROUP BY query: SUM(payment.net_amount) by `due_type` bucket, filtered by `payment_mode NOT IN (211, 288)`. | **MEDIUM** |
| **Section A Operating Outflows aggregation** | DA-04 ILIKE buckets + DA-03 refunds (operating only) − Deposit Refund dedup. | **MEDIUM** |
| **Section B (Deposit Money In/Out) aggregation** | Deposits collected query + Deposits returned query (refund + 'Deposit%' expense, deduped). | **MEDIUM** |
| **MoM sign-reversal detection** | Compare current period sign to prior period (single-period comparison; not 2-period). Encode in MoM chip color (red = reversal to negative; green = reversal to positive). | **LOW** |
| **Deposit-dependency detection** | Compute deposit ratio = Section B inflow / total inflow. | **LOW** |
| **Capex flag detection** | "Other" expense > 25% of total outflow AND any single expense > ₹50K. | **LOW** |
| **Per-bed Cash Flow (multi-property only)** | Hero ÷ live occupied bed count (DA-04 pattern). Render only when multi-property is selected. | **LOW** |
| **By Property aggregation** | Per-property hero rerendered. Sort ASC by net cash flow. | **LOW** |
| **MoM chip with 3 states** | Standard MoM chip rendering 3 states (improvement / regression / sign-reversal — see MoM sign-reversal detection above). | **LOW** |
| **Trend chart aggregation** | 6M of: monthly inflows, monthly outflows, monthly net. Stacked + overlay rendering. | **MEDIUM** |
| **Bottom sheet content per line** | Per-line drill: top 5 entities + cross-screen CTA only (no 6M mini-trend per spec v1.1). ~14 sheets to spec UI. Use shared template. | **MEDIUM** (UI-heavy but template-driven) |
| **Cross-screen filter pre-set integration** | DA-02/03/04 worklist support pre-sets — verify cash-flow's filter shape can be passed through. | **LOW** |
| **Excel export bug fixes** | All 12 Pre-Launch Engineering Blockers in the existing report. | **HIGH** (blocking work) |
| **Trend insight text** | Conditional copy based on 6M trend pattern. | **LOW** |
| **Discount memo annotation** | DA-05 hero owner-funded total + RentOk-funded sub-amount. | **LOW** |

### Critical Architecture Notes for Engineering

1. **CRITICAL — `generateCashFlowReport.ts:393` `totalExpenses = expenseTotal + refunds` is wrong.** Refunds are double-counted. Production Excel reports are wrong today. DA-07 mobile screen MUST NOT inherit this; it would create a credibility crisis when mobile and Excel disagree. Fix the Excel before shipping DA-07. See Pre-Launch Blocker B1.

2. **CRITICAL — Mode 211/288 must be excluded from inflows.** Verify `payment_mode NOT IN (211, 288)` filter is applied to ALL inflow aggregations. The existing report sums `invoice.amount` without checking payment_mode — over-counts. See Pre-Launch Blocker B2.

3. **CRITICAL — Discounts are NOT a cash flow line.** They are concessions BEFORE billing. `payment.net_amount` already reflects them. Putting Discounts as an outflow line would double-count. Memo annotation only.

4. **CRITICAL — RentOk-funded discounts have NO make-whole.** Investigation confirmed no code path transfers cash from RentOk to operator for `Credits.source = 1`. Memo annotation in DA-07 must say *"no impact on your revenue"* — operator collected exactly what tenant paid (post-discount).

5. **No opening/closing balance in v1.** RentOk doesn't track operator cash position. Frame DA-07 as "movement, not position." Phase 2 may add operator-entered opening balance.

6. **Cash basis discipline.** Use `paid_date` everywhere. Never `due_date` or `invoice_date`. Matches DA-02/03/04/05 governing date rules.

7. **Multi-property = aggregate sum, not average.** Σ(per-property) = total. No occupancy-weighting, no normalization.

8. **DA-04 ILIKE bucketing must be applied in DA-07 outflows AND in the Excel report.** Free-text `expense_type` rendered as separate rows is unusable. Apply Salary/Electricity/Mess-Food/Maintenance/Rent/Deposit/Other. Excludes 'Deposit%' from Operating Outflows entirely (those go to Section B).

9. **DA-03 refund category split (by `invoice.due_type`)** must be applied in the Excel and in DA-07's operating-vs-deposit refund split. Operating refunds (non-deposit due_type) go to Section A. Deposit refunds go to Section B.

10. **Property scoping fragility.** Same composite string `${pg_id}PG${pg_number}` issue flagged in DA-03/05/06 — `lastIndexOf('PG')` parsing breaks on properties named "PG Greenwood".

11. **No standalone DA-07 worklist.** Bottom sheets dispatch to existing DA-02/03/04 worklists with pre-set filters. Preserves single source of truth for transaction-level views.

12. **Cross-screen reconciliation is the screen's whole reason to exist.** If DA-02 hero ≠ Σ(DA-07 operating inflows + Section B deposit inflows) for the same period, the screen has failed. Build aggressive integration tests before shipping.

13. **Trend chart 6M historical data.** Currently the existing Excel report can compute any historical period on demand. The trend chart will use the same on-demand calc — no separate snapshot table needed (unlike DA-06 trend chart which had this discussion). DA-07 trend is cheaper because it derives from existing payment/expense/refund tables which are full-history.

14. **Bottom sheet UI complexity.** 14 distinct bottom sheets × ~5 components each = ~70 UI pieces to design. Use a shared bottom-sheet template with parameterized content. Recommend: `<CashFlowDrillSheet line={lineId} period={period} property={property} />` taking 4 props.

15. **GST handling deferred.** Phase 2 will surface GST when operator has commercial invoices. Until then, DA-07 hero is gross-of-GST (cash actually received), and the Excel surfaces GST splits — but mobile doesn't.

16. **Owner withdrawals are invisible.** Disclaimer in tooltip mandatory. Phase 2 idea: dedicated expense category.
