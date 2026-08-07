---
title: DA-02 Collections Detailed Analytics
date: '2026-05-01'
tags:
  - rentok
  - prd
  - spec
  - detailed-analytics
  - collections
  - financials
aliases:
  - DA-02
  - Collections Detailed Analytics
---
> [!INFO] Source of Truth
> This note mirrors the spec at `RentOk Manager Homescreen/spec/DA-02-collections-detailed-analytics.md`.
> **Engineers:** see [[DA-02 Build Sheet]] (9-column ticket-ready format) for implementation. This doc is the canonical "why."
> **Supporting refs:** [[_Ground Truth Field Map]] · [[_Build Sheet Generation Spec]]
> Related: [[DA-01 Dues Detailed Analytics]]

# Collections — Detailed Analytics

> **Product:** RentOk Manager App → Financial Insights → Collections
> **Version:** 3.1
> **Status:** Final — Locked (revised post 2 operator audit rounds 2026-05-07; v2.1 → v3.0 closes critical cross-screen ambiguities)
> **Last Updated:** 2026-05-07
> **Supersedes:** Lark PRD "Collections · Financial Detailed Insights v1.0"
> **Design Reference:** Figma `KgBQXiT7r7oGrcqZHCWxyU` → Financials → Collection container (`7837:99848`)
> **Parent Spec:** 07b-collections.md — homescreen collapsed/expanded card spec

---

## The One Thing This Screen Does

It answers: **"Kitna paisa aaya, kahan se aaya, kaun laya?"**
("How much money came in, from which source, and who collected it?")

Every component exists to answer one of those three questions. Collections is an income view — it shows money received, not money owed.

---

## A Morning with Rajesh

It's the 8th of May. Rajesh manages two PGs in Pune. He opens the app.

Collections shows ₹1.5L net. He taps the efficiency badge: 83%. Good. He checks the Source bar — 60% is from current month's dues, 28% from old dues recovery, 12% advance received. Old dues recovery is high — tenants are finally paying up from March.

He expands "Received By." Shiv collected ₹45K — all cash, 3 receipts. That's ₹15K per receipt. That's unusually large. He taps Shiv's row, sees the three transactions, and calls Shiv to verify. One was a tenant paying 3 months at once. Makes sense.

Done in 3 minutes. No spreadsheet. No calling the accountant.

---

## A Morning with Priya (multi-property end-of-month)

It's April 30. Priya manages 3 PGs in Mumbai. She opens DA-02. Hero ₹4.8L net (▲14% vs ₹4.2L Mar). Efficiency badge 87% — green.

By Property: Property 2 — ₹2.2L (₹2.8L collectible — 79%), Property 1 — ₹1.5L (94%), Property 3 — ₹1.1L (90%). Property 2's efficiency is below her 85% threshold. She taps.

Property 2 worklist filtered. By Category shows Rent collected ₹1.6L, Electricity ₹0.4L, Food ₹0.2L. By Received By: Manager Aakash 80%, Riya 20%. Adjustments row below Source bar shows ₹35K cleared via mode 211 — old deposit applied to new bills (matches the move-out cluster from April).

Done in 4 minutes. She knows Property 2 needs follow-up; Adjustments aren't masquerading as cash.

---

## A Morning with the CA via WhatsApp screenshot

Priya forwards her DA-02 hero to her CA. The CA opens the screenshot.

Hero shows ₹4.8L · "Net Collections" subtitle (GAAP framing for accountant) · "(48 Receipts · net of refunds)" subtitle · ▲14% MoM chip. CA understands: this is operator's actual cash receipts after refunds, excluding paper adjustments.

She replies: *"Send me the Excel — looks consistent with what I'd expect."* Priya goes to overflow menu (⋮) → Email Collections Excel → done.

Without the GAAP subtitle and the explicit "net of refunds" qualifier, the CA would have asked 3 clarifying questions about scope.

---

## Scope

**This section shows money received — actual collections only.**

| Included | Excluded |
|----------|----------|
| Cash, UPI, bank transfer, cheque payments | Dues (unpaid bills) |
| RentOk gateway payments | Expenses (money spent) |
| Deposit adjustments applied to invoices | Deposits Held (liability balance) |
| Advance adjustments applied to invoices | Discounts / waivers |
| Advance received (unapplied) | Future expected collections |
| Partial payments | |
| Refunds issued (shown as reduction from gross) | |

Collections is driven by **paid_date** (receipt date). A May rent bill paid in June appears in June's collections, not May's.

---

## The Six Rules

0. **Canonical aggregation path (v3.0).** Hero number = `SUM(payment.net_amount)` where `p.status = 1` AND `p.is_active = 1` AND `p.payment_mode NOT IN (211, 288)`, MINUS Σ refunds via `ref_agg` CTE pattern. This single canonical query is used by BOTH widget and list endpoints — no divergent paths. **DA-03 net-collection hint, DA-05 owner-funded reconciliation, DA-06 deposit-collection invariant, and DA-07 Section A inflow ALL depend on this number being stable.** Without Rule 0 fixed, four sibling specs ship with broken cross-screen reconciliation.

1. **Paid date governs.** All filters apply to `p.paid_date`. Exception: "By Category" uses `inv.due_date` month to classify what the payment was for — this is always surfaced to the operator.
2. **Net of refunds in hero.** Total shown = Gross Collections − Refunds Issued. A tooltip breaks down gross vs. net so the operator always knows what the number means.
3. **Mode 211 / 288 are paper transfers — excluded entirely from inflows AND from the Source bar.** Mode 211 (deposit-applied) and 288 (advance-applied) move no cash. They appear ONLY in a separate "Adjustments" row below the Source bar (and as separator rows in Payment Modes for completeness). They are NOT one of the four Source segments. Matches DA-07 Rule 3.
4. **The numbers must match.** Every count and amount shown here must exactly match the drill-down worklist when tapped. Same date range, same property scope, same filter logic.
5. **Unlinked payments are surfaced, not hidden.** If a payment was recorded without attributing it to a tenant, an alert appears so Rajesh can fix it before it distorts population reports.

---

## How Rajesh Gets Here

`Homescreen → Overview → "View Detailed Analytics" → Financial Insights → Collections tab`

Collections is the second tab in Financial Insights, after Dues. Swipe right from Dues or tap the Collections chip in the top nav.

---

## What Rajesh Sees

### 1. The Header

**"Collections"** — with a small ⓘ icon.

No "(Live)" suffix — Collections is always period-based. The active period (e.g., "Apr 1–8") is visible in the time filter chip above.

Tapping ⓘ shows:
> "Collections shows all money received during the selected period. Refunds are subtracted to show net collection. Deposit and advance adjustments are excluded from the net total — they're visible in the Source and Payment Mode breakdowns."

---

### 2. Hero KPI — Total Collection

**The question this answers:** "How much money actually came in, net of refunds?"

A large number — ₹1.5L — with a subtitle: *(48 Receipts).*

> **Figma note:** Design shows `(120 Invoices)` as the subtitle. This is incorrect — a single payment can cover multiple invoices, so invoice count is not meaningful here. Subtitle should show payment count (= `p.id` count), labeled "Receipts." Engineering follows this spec, not the Figma label.

Three elements in the hero:

**MoM comparison chip:** ▲12% vs last month → green. ▼8% → red. For collections, UP = good.

**Efficiency badge (inline, below the number):**
`Collection Efficiency: 83%`
Tap → tooltip: *"Collection Efficiency = Amount Collected ÷ Amount that was due by [end date]. Collectible: ₹1.8L · Collected: ₹1.5L · Efficiency: 83%"*

Color: ≥80% green · 50–79% amber · <50% red.

**Hero ⓘ tooltip breakdown:**
```
Gross Collection: ₹1.55L
Refunds Issued:   ₹5,000
Net Collection:   ₹1.5L
```

The entire hero block is tappable → opens full collections worklist for the period.

**Multi-property (Priya):** Subtitle reads *(48 Receipts · across 3 properties).*

---

### 3. Source Breakdown

**The question this answers:** "Is this money from current month's billing, or old dues recovery?"

A horizontal stacked bar — four color-coded segments — showing where the collected money came from:

| Segment | Color | What it covers | Tap goes to |
|---------|-------|----------------|-------------|
| [Month]'s Dues | Green | Collections against invoices billed for the current period | Worklist: current month invoice collections |
| Old Dues | Amber | Collections against invoices from previous months — recovery | Worklist: old dues collections |
| Advance | Blue | Advance/prepaid amounts received, not yet applied to invoices | Worklist: advance payments |
| Paid Early | Purple | Collections against invoices billed for future months — tenant paid before due date | Worklist: future dues collections |

> **Figma confirms this label:** Design uses "Paid Early" — clearer than "Future Dues" (both mean the same thing: payment against a future-dated invoice). Use "Paid Early."

Below the bar, four chips — one per segment — showing label + amount. Tap either the bar segment or the chip.

**Why this matters:** If 70% of collections are from Old Dues recovery, the current month's fresh billing isn't performing. That's a different problem than if 70% is current month. Same total, very different health signal.

**Label changes with time filter:**
- This Month → "[Apr]'s Dues"
- Last Month → "[Mar]'s Dues"
- Last 6 Months → "Period Dues"
- Custom → "Period Dues"

---

### 4. Population Breakdown

**The question this answers:** "Who is paying — current tenants, bookings, or old tenants?"

Three tappable rows:

| Row | Who | Tap goes to |
|-----|-----|-------------|
| Active Tenants | Currently living tenants | Worklist: active tenants only |
| Booking | Confirmed future move-ins who paid | Worklist: bookings only |
| Old Tenants | Moved-out tenants who made a payment (deposit adjustments etc.) | Worklist: moved-out tenants |

Each row: label + amount + count. The total of these three rows = Net Collection total.

---

### 5. Unlinked Payments Alert

**Shown only when:** One or more payments exist with no tenant attributed.

A high-contrast warning card:
`₹8,000 received — not linked to any tenant. Link now to keep reports accurate.`
CTA: **"Link Payment"** → opens Unlinked Payments flow.

**Unlinked Payments flow:**
1. List of unlinked payments (amount, date, mode, recorded by)
2. Tap a payment → search for tenant/booking by name, phone, room
3. Confirm linking → payment attributed, alert dismissed

**Why this exists:** Staff sometimes record cash without selecting the tenant — especially during busy move-in days. The payment sits in "Unlinked" and distorts the Population breakdown. Surface it early so it gets fixed.

---

### 6. Advanced Insights

A section divider. Four accordions.

**Default state:** Performance is **expanded by default**. The other three (By Category, Payment Modes, Received By) are collapsed.

> **Figma confirms this:** The main populated state shows Performance expanded on load. This is the right call — Performance is the first question an operator has after seeing the total.

---

#### 6A. Performance — "Am I billing well and collecting efficiently?"

Two chips at the top of the accordion:

| Chip | Label | What it means |
|------|-------|---------------|
| Left chip | **Invoices Created** ₹2.2L | Total billed in the period — invoices with `inv.due_date` falling within the selected period (not `inv.created_at`; auto-generated future invoices with due_date outside the period are excluded) |
| Right chip | **Collection** ₹1.5L | Net collection in the period |

Below the chips: a summary sentence — *"₹[X] collected between [start date] → [end date]."*

Below that, an inline bar: Collectible (full width) with Collected overlaid, plus:

| Row | What it means |
|-----|---------------|
| **Collection Efficiency** 83% | Collected ÷ Collectible |

> **Figma note:** Design uses "Invoiced Created" (typo). Copy spec is "Invoices Created." Engineering follows this spec.

An inline bar: Collectible (full width) with Collected overlaid.

Below: a **timing breakdown** — how payments aligned with due dates:

| Row | Definition | Color |
|-----|-----------|-------|
| On Time | `paid_date = due_date` | Green |
| Early | `paid_date < due_date` | Blue |
| Late | `paid_date > due_date` | Red |
| Adjustments | Deposit/advance adjustments (not timing-classifiable) | Grey |

Each row: tenant count + amount. Tap → worklist filtered to that timing bucket.

**Multi-invoice timing classification:** A single payment can clear multiple invoices with different due_dates (e.g., tenant pays April + May dues in one UPI transfer). Classify using `MIN(inv.due_date)` — the oldest obligation. If even the earliest due bill was paid late, the payment is Late. This ensures arrear-clearing payments are flagged correctly, not masked by a current-month invoice in the same payment.

Below timing: **Partial Payments** — *"₹25,000 across 12 invoices still have outstanding balance after this payment. These need follow-up."* Tap → dues worklist filtered to partially-paid invoices.

**Tooltips:**
- *Collectible:* "Total dues that were due on or before [date]. Excludes future dues not yet due."
- *Collection Efficiency:* "Amount Collected ÷ Collectible. 100% means every due bill was fully paid."
- *Timing:* "Payment timing vs invoice due date. On-Time = same day. Early = before. Late = after."

---

#### 6B. By Category — "Which bill types are being paid — and which aren't?"

A ranked list sorted by collected amount. Each row shows: category icon + label, collected (₹X) / billed (₹Y), progress bar.

| Category | Collected | Billed | Progress |
|----------|-----------|--------|----------|
| Rent | ₹80K | ₹1.1L | 73% |
| Electricity | ₹22K | ₹28K | 79% |
| Food | ₹18K | ₹25K | 72% |
| Security Deposit | ₹15K | ₹20K | 75% |
| Others | ₹15K | ₹22K | 68% |

Tap any row → worklist filtered to that category.

**Tooltip:** *"Collected: Amount received for [category] this period. Billed: Amount invoiced for [category] this period. Progress shows collection rate."*

**Note on categorization:** "Billed" uses `inv.due_date` month; "Collected" uses `p.paid_date`. A March rent bill paid in April appears in April's "Rent — Collected" but not in April's "Rent — Billed." This is correct behavior — the category row shows what was received this period and what was billed this period as separate numbers.

---

#### 6C. By Payment Mode — "How did the money come in?"

A ranked list sorted by amount. Each row: mode icon + label, amount, progress bar (share of total).

| Mode | What it covers |
|------|----------------|
| RentOk (Online) | RentOk payment gateway — modes 203, 205, 206, 210 |
| Bank Transfer | Direct NEFT/IMPS transfers |
| UPI | Third-party UPI apps |
| Cash | Physical cash |
| Others | Cheque, DD, other modes |

Separator below the five rows — two adjustment rows (not in total):

| Row | What |
|-----|------|
| Adjusted from Deposit | Mode 211 — deposit applied to clear dues |
| Advance Adjusted | Mode 288 — advance applied to clear dues |

Below all rows: summary line — *"Verified Digital (RentOk): 35% · Manual Entry: 65%"*

**Performance nudge (conditional):** If RentOk % increased vs prior period → *"✨ More tenants are paying digitally this month!"* If decreased → *"💡 Push RentOk Pay links to reduce manual recording."*

**Settlement indicator on RentOk row:** *"₹12K unsettled"* if any online payments lack UTR. Tap → settlement worklist.

Tap any mode row → worklist filtered to that payment mode.

---

#### 6D. Received By — "Who on my team collected this money?"

A ranked list sorted by amount collected. Each row: avatar + name, total collected, receipt count.

System row first: **RentOk** (online self-payments by tenants — no staff involved). Trophy badge if RentOk is the top "collector."

Staff rows: each team member who recorded payments. Sorted by amount.

If >4 staff: remaining collapse into "Others (N staff) · ₹X" with tap-to-expand.

**Concentration flag (admin-only, conditional, calibrated v3.0):**
Fires only when ALL four conditions hold:
1. One staff member recorded > 50% of cash payments
2. AND ≥ 5 cash receipts in the period (maturity gate — no flag on 1-2 cash receipts)
3. AND total cash collected by that staff ≥ ₹10,000 (absolute floor)
4. AND property is ≥ 60 days old from launch (newness bypass — setup mode often has one staff doing everything)

→ *"⚠️ Riya recorded 68% of cash this month (₹X across N receipts). Worth a check-in."*

> **v3.0 calibration:** v2.1 fired on any single staff member at >50% with no maturity gate. Result: setup-mode properties (1 staff, 1 cash receipt = 100% concentration) fired the alarm constantly. Matches DA-06 v1.1 anomaly-callout discipline.

Tap any row → worklist filtered to `received_by = that person`.

**Tooltip:** *"Number of receipts recorded by this person. One receipt may cover multiple invoices."*

---

### 7. Collection Trend Chart (Always Visible)

Below the four accordions, a dual-bar chart is always visible — no tap required.

Each bar period shows two grouped bars:
- **Yellow bar:** New dues raised in that period (stacked: current period dues)
- **Green bar (stacked):** Collections received in that period — broken into source segments: [Month]'s Dues collected / Old Dues collected / Advance received

The green bar is stacked to show the source composition visually. The yellow bar is a single value.

4-item legend: Dues · Collection · Old Dues · Advance

**"This Month" view:** Single bar for the active date range (not multiple months).
**"6M" view:** One grouped pair per month for the last 6 months.

When green bar height approaches or exceeds yellow bar height, collection is keeping pace with billing. When yellow persistently towers over green, outstanding is growing.

**Time selector on the chart itself:** `This Month` (default) · `6M`

> **Figma update:** Design shows "This Month" as the default time selector on the trend chart, not "6M." The 6M view is an alternative. Both are valid — "This Month" makes more sense as default since operators check the current month first.

**Bars are tappable:** Tap any bar → tooltip showing exact dues raised + collected for that period. Tap the green bar → opens collections worklist for that period.

**Trend insight text (conditional):** Shown below chart when pattern is detectable:
- "Outstanding trending down over last 2 months." (green bars ≥ yellow)
- "Collections below billing for 3 months. Outstanding may be building up." (yellow consistently higher)

---

## Drilling Down: Collections Worklist

Every tappable element leads to a filtered payment list.

### Worklist Pre-sets

| Where tapped | Pre-set filter | Default sort | Header |
|-------------|---------------|--------------|--------|
| Hero total | No filter (all payments in period) | Paid date DESC | "All Collections · 48 Receipts · ₹1.5L" |
| Source: [Month]'s Dues | Invoice due_date in current period | Paid date DESC | "Apr Dues Collected · 29 Receipts · ₹90K" |
| Source: Old Dues | Invoice due_date before current period | Paid date DESC | "Old Dues Collected · 14 Receipts · ₹42K" |
| Source: Advance | Advance payments (unapplied) | Paid date DESC | "Advance Received · 5 Receipts · ₹18K" |
| Source: Paid Early | Invoice due_date after current period | Paid date DESC | "Paid Early · 3 Receipts · ₹10K" |
| Active Tenants row | Tenant status = active | Paid date DESC | "Active Tenants · 40 Receipts · ₹1.2L" |
| Booking row | Tenant status = booking | Paid date DESC | "Bookings · 4 Receipts · ₹18K" |
| Old Tenants row | Tenant status = moved out | Paid date DESC | "Old Tenants · 4 Receipts · ₹12K" |
| On Time timing row | paid_date = due_date | Paid date DESC | "On Time · 28 Receipts · ₹65K" |
| Late timing row | paid_date > due_date | Paid date DESC | "Late Payments · 12 Receipts · ₹30K" |
| Category row | That invoice category | Paid date DESC | "Rent Collected · 20 Receipts · ₹80K" |
| Payment mode row | That payment mode | Paid date DESC | "UPI · 18 Receipts · ₹48K" |
| Received By row | That `received_by` | Paid date DESC | "By Shiv · 8 Receipts · ₹45K" |
| Unsettled (settlement) | Online, no payout | Paid date DESC | "Unsettled · 3 Receipts · ₹12K" |

### Per-Payment Row

- Tenant name + room + status badge (Active / Booking / Moved Out)
- Payment amount (prominent)
- Paid date + "X days late" if `paid_date > due_date`
- Payment mode badge
- Settlement status chip for online payments (Settled / Processing / Unsettled)
- Receipt icon if `receipt_url` exists

### Per-Payment Actions

- **View Receipt** — PDF/image
- **View Tenant** — tenant profile
- **WhatsApp** — opens chat
- **Call** — dials tenant

### Worklist Filters

Operator can further filter: Date range · Payment mode · Recorded by · Category · Tenant type · Property (multi-property) · Status (Active / Voided)

### Bulk Actions

- Export to Excel

---

## Drill-Down Behavior

> **Universal navigation rules apply to all 7 DA specs.** Priorities (P0/P1/P2) are PM recommendations; engineering may re-prioritize during spike based on existing app capabilities and effort.

### Universal Rules

**R1. Modal/Sheet/Screen primitive [P0]** *(updated 2026-05-11)* — Every tap target's destination is explicitly typed: full-screen push, bottom sheet, modal overlay, or inline accordion. **ⓘ icon convention (locked across DA suite):** single-tap → bottom sheet. No inline tooltip. No long-press. See `[[_Build Sheet Generation Spec#15. ⓘ Icon Interaction Convention]]`.

**R2. Back-stack semantics [P0]** — Back pops one navigation frame and restores the prior frame's filter chips, scroll position, accordion state, and selected segments. iOS swipe-back and Android system back behave identically.

**R3. Deep links + share sheet [P1]** — Every drill state is uniquely URL-addressable as `rentok://da-NN/<view>?<filters>`. Push notifications and WhatsApp deep-links generate these URLs.
> Engineering note: existing app uses `rentok://collection` (singular). New drill states should align with existing router patterns.

**R4. Permission gating UX [P0]** — Sections without read permission HIDE (not gray). Disabled actions show a lock icon + toast on tap. Cross-screen drill into a denied screen shows full-screen denial; back returns to the source bottom sheet. Mobile must inspect `can_view_invoices` flag before showing drill targets as tappable. EC-08 "Self-added-only" pattern (with "(Limited view)" badge) is the canonical reference.

**R5. Loading states [P0]** — Skeleton-on-load (not spinner) on every screen. Cross-screen transitions show destination header immediately + skeleton rows. Pull-to-refresh shows chevron without flashing skeletons.

**R6. State preservation [P1]** — Tab switches preserve per-tab filter and scroll state. App background within 15 min restores exact drill state. Force-quit returns to homescreen.
> Engineering note: 15-min threshold is PM-suggested; calibrate during spike.

**R7. Multi-property scope inheritance [P0]** — Single-property drill OVERRIDES global scope for the entire descendant drill stack. Cross-screen CTAs inherit single-property scope. Scope chip always visible in worklist header.

**R8. Worklist filter-chip behavior [P0]** — Every pre-applied filter is a removable chip. Removal re-fetches without that filter; others stay. New filters are additive (AND). Worklist filter changes do NOT propagate back to the dashboard.

**R9. Shareable state [P2]** — "Share this view" affordance in overflow menu generates deep link + system share sheet. Excel export remains for deep-analysis; deep link for live-data sharing.
> Engineering note: net-new feature; defer to Phase 2 if scope-constrained.

**R10. CA-screenshot discipline [P0]** — Every hero has a visible GAAP subtitle and basis label — never tap-only. DA-02's "Net Collections" subtitle (v3.0) is the canonical pattern. No critical context in tap-only tooltips.

**R11. Cross-screen back path [P1]** — Cross-screen drill pushes destination as CHILD of source bottom sheet. Destination shows breadcrumb "← From [Source DA name]". Tap → returns to source bottom sheet.
> Engineering note: if no breadcrumb support today, fall back to standard back-stack pop (R2).

### Priority Summary

| Priority | Rules | Engineering guidance |
|----------|-------|---------------------|
| **P0 (must have)** | R1, R2, R4, R5, R7, R8, R10 | Universal mobile expectations; broken UX or persona regression without |
| **P1 (should have)** | R3, R6, R11 | Significant UX value; depends on existing app router/persistence patterns |
| **P2 (defer if needed)** | R9 | Net-new feature beyond Excel export; Phase 2 candidate if scope-constrained |

### Per-Spec Specifics (DA-02)

- **Unlinked Payments flow** = full-screen push (not modal). Tenant search within the flow uses the same component as elsewhere in the app.
- **Receipt PDF** opens in system PDF viewer (not in-app). Per-payment row "View Receipt" action triggers system intent/handoff.
- **Tenant Detail target** = existing app feature (full-screen push). Cross-spec referenced from DA-01/03/04/05/06 worklist rows.
- **Performance accordion expand state** preserves on back from drill. Operator who expanded Performance, drilled, and returned sees Performance still expanded.
- **Trend chart yellow bar (Dues raised) drill** opens DA-01 Dues worklist for that month (cross-spec drill — R11 applies). Trend chart green bar drill opens DA-02 collections worklist for that month.
- **Adjustments row (mode 211/288) drill** opens worklist filtered to those modes only. Excluded from hero-tap drill (Rule 3 — Mode 211/288 excluded from hero math).

---

### Per-Spec Specifics — Additions (post-orphan-audit)

- **Source bar segment vs chip — disambiguation:** Both route to the same worklist filter; the **chip is the canonical tap target** (44pt min hit area). Bar segment is convenience and may be smaller. Long-press on either shows source-specific values tooltip. Worklist Pre-sets list source chips, not bar segments.
- **Early timing-row drill** (in Performance accordion): tap → worklist with `paid_before_due_date=1` chip pre-applied. Pre-set added.
- **Adjustments timing-row drill:** tap → worklist with `mode IN (211, 288)` chip (deposit-applied + advance-applied modes). Pre-set added.
- **Partial Payments callout cross-screen drill:** tap → leaves DA-02, enters **DA-01 worklist** with `payment_status=partial` chip + breadcrumb "← From DA-02 Collections" (R11). Back returns to DA-02 dashboard.
- **Trend chart yellow bar (Dues raised) cross-screen drill:** R12 specific — tap-into-tooltip "View [month] →" leaves DA-02, enters DA-01 worklist filtered to that month's `raise_period` + breadcrumb (R11).
- **RentOk row "Unsettled" indicator** (in Payment Mode breakdown): tap on "Unsettled" inline indicator → worklist with `mode=rentok AND settled=0` chip. Tap on the row body itself → worklist with `mode=rentok` chip only.
- **"Link Payment" CTA in Unlinked Payments alert:** full-screen push opens 3-step internal flow (list → search candidate invoice → confirm linking). Each step is a separate frame in the back-stack (R2).
- **Concentration flag callout "⚠️ Riya recorded 68%…"** (in Received By section): tap → worklist with `recorded_by=riya` chip + sorted by amount DESC.
- **Filter chip ✕ on worklist** (R8 clarification): each chip on the collections worklist (Date / Property / Source / Mode / Recorded by / Tenant Status / Custom) has explicit ✕; tapping removes that chip and re-fetches.

### Universal Rule Clarifications (post-orphan-audit)

Resolves interaction-primitive ambiguities surfaced during the post-master orphan-tap-target audit. Treat as additive to R1–R11 above.

**R1 clarification — explicit primitives for common UI:**
- **Hero ⓘ icon:** single-tap → bottom sheet with plain-English explanation + GAAP framing + basis label. **No inline tooltip. No long-press.** Per Generation Spec §15.
- **MoM chip:** tap = inline tooltip showing prior-period numbers + computation window. No drill.
- **Accordion section:** tap on row OR chevron toggles expand/collapse. Default state per section spec.
- **Information-only chips** (settlement indicator, concentration flag, repeat-tenant on a row): single-tap = inline tooltip. No drill unless explicitly listed in Per-Spec Specifics.

**R5 clarification — pull-to-refresh:** Dashboard re-fetches all sections in parallel; worklist re-fetches with current chips. Cross-screen drill destinations also support pull-to-refresh. Chevron animates without flashing skeletons.

**R8 clarification — filter chip ✕ as explicit tap target:** Every chip has a discrete ✕ icon (44pt min hit area). Tapping ✕ removes that chip and re-fetches; other chips stay. Body-tap opens edit affordance where applicable; otherwise no-op.

**R12 (NEW) — Trend chart conventions [P0]:**
- **Bar single-tap:** inline tooltip showing exact values. No drill.
- **Tap-into-tooltip CTA "View [period] →":** drills to worklist filtered to that period (cross-period drill overrides parent hero period for descendant drills).
- **Stacked-segment tap** (e.g., Old Dues / Advance sub-segment within green Collections bar): drills to worklist with sub-segment filter applied. **DA-02 specific: yellow bar (Dues raised) cross-screen drills into DA-01 with breadcrumb "← From DA-02 Collections".**
- **Period selector toggle (`6M` / `This Month`):** tap to switch range. Selected state preserved on back. Does NOT affect dashboard hero values.

### Updated Priority Summary

| Priority | Rules | Engineering guidance |
|----------|-------|---------------------|
| **P0 (must have)** | R1, R2, R4, R5, R7, R8, R10, R12 | Universal mobile expectations |
| **P1 (should have)** | R3, R6, R11 | Significant UX value |
| **P2 (defer if needed)** | R9 | Net-new beyond Excel export |

### Permission Vocabulary Reality Check

Codebase has only **11 JWT-mirrored permission keys**: `appAccess, cashCollection, recordPayment, editInvoices, editTenants, viewInvoices, viewExpenses, deleteInvoices, addTenants, deleteTenants, viewTenants` (mirror sites: `src/v1/login/property/service.ts:79-91` + `src/controllers/property.ts:14883/15041/15767/17813` + `src/helpers/teamMember.ts:134`). DB-side via `checkAuthInDb`: ~70 snake_case columns on `team_member_property`.

Keys cited in this spec vs codebase reality:

| Cited key | Status | Recommended Phase 1 path |
|-----------|--------|--------------------------|
| `viewInvoices` | EXISTS — JWT key | Canonical read-gate (codebase already uses for collections list service) |
| `viewCollections` (cited) | DOES NOT EXIST | Reuse `viewInvoices` |
| `recordPayment` | EXISTS — JWT key | Mutation gate for "Record Payment" worklist action |
| `view_invoices_of_self_added_tenants` | EXISTS — DB column | Fallback gating for limited-permission team members |

> **Decision owner: Jatin (Sr Backend).** Each MISSING key requires build-or-reuse decision. Specs cannot ship until Jatin signs off. Recommendations are PM suggestions only.

> **Filter code typo (engineering note):** `CollectionFilterCode` enum at `src/v1/constants/filterCodes.ts:53,56` has `ADVANCE_ADJUSTED_THIS_MONTH = 1288` and `ADVANCE_ADJUSTED_ALL_TIME = 12881` — both are out-of-stated-range (1201-1219). Suspected typos for 1220 / 1221. Fix in Phase 1 if mobile does any range validation; document the gap if not.

---

## Time Filter Behavior

| Component | Behavior |
|-----------|---------|
| Hero total | Net of refunds, paid_date in period, excl. modes 211/288 |
| Efficiency badge | Collectible = invoices due on or before period end |
| MoM chip | Prior equivalent period |
| Source breakdown | Invoice due_date vs period boundaries |
| Population | tenant.status at payment time |
| Unlinked alert | Unlinked payments with paid_date in period |
| Performance section | Collectible and timing based on invoice due_dates in period |
| Category breakdown | Invoice due_type, collected in period |
| Payment modes | paid_date in period |
| Received By | paid_date in period |
| Trend chart | Has its own This Month/6M selector — not governed by time filter |

**Custom future dates:** Collections values show ₹0 for future portion. A helper: *"Collections are based on received payments. Future dates show ₹0."*

---

## Edge Cases

**EC-01: No collections in period**
Hero shows ₹0. Efficiency badge shows 0%. Empty state: *"No payments recorded in this period."*

**EC-02: Refund exceeds gross collection**
Net collection is negative. Hero shows negative number with red styling. Rare — but can happen if a large refund was issued after the payment period. Tooltip clarifies gross vs. net.

**EC-03: All collections are deposit adjustments**
Default net total = ₹0 (modes 211/288 excluded). Source breakdown shows the adjustment amount in a separate segment. The hero tooltip explains why net = 0 when adjustment = non-zero.

**EC-04: Partial payment**
₹10,000 invoice, ₹6,000 paid → payment row shows ₹6,000. Invoice remains `status=0` in Dues. Partial Payments insight in Performance section surfaces the ₹4,000 remaining.

**EC-05: Payment reversed (status ≠ 1)**
Not shown anywhere in Collections. Reversals are not collections.

**EC-06: Online payment — bank account not linked**
Settlement panel shows "Unsettled — Link Bank A/C". UTR field: "Link Bank A/C to receive the payment."

**EC-07: Multi-property (Priya)**
All amounts aggregate. A "By Property" breakdown row appears between Population and Unlinked Alert — each property with amount + share. Tap → worklist filtered to that property.

**EC-08: Self-added-only access (Meena)**
`view_invoices_of_self_added_tenants` permission → sees only collections from her tenants. Total reflects her visible subset. "(Limited view)" badge on header.

**EC-09: Tenant pays multiple invoices in one payment**
One row in worklist (one `p.id`). Category breakdown uses `SUM(inv.amount)` for invoice-level sum when category filter is active — prevents double-counting.

**EC-10: Mode 213 (partial EazyPG credits)**
Amount formula handled by backend: `net_amount - gateway_charges - owner_credits + credits_used`. The correct computed total is surfaced — PRD doesn't expose this to the UI.

> **v3.0 BUG FIX:** Earlier v2.1 had `(net_amount - gateway_charges) - owner_credits + credits_used - gateway_charges` — `gateway_charges` subtracted twice. Engineering ship would have been ₹X off by gateway_charges magnitude per receipt. Fixed in v3.0.

**EC-11: Collection efficiency >100%** ✅ Locked
Possible when: (1) old dues from prior periods are recovered in the current period, (2) advance payments against future invoices are collected in the period, (3) tenant overpays (UPI rounding, multi-month lump sum).

**Decision:** Show the actual percentage (e.g., "124%") — not capped at 100%, not replaced with ">100%". Color thresholds unchanged: ≥80% green · 50–79% amber · <50% red. >100% stays green (positive signal — arrear recovery is healthy).

The efficiency badge tooltip expands to show the source breakdown:
> *"Collection Efficiency: 124% · ₹1,00,000 current period dues · ₹15,000 old dues recovery · ₹9,000 advance"*

This split is computed from existing data: `paid_date` month = `inv.due_date` month → current period; `paid_date` month > `inv.due_date` month → old dues recovery; `inv.due_date` after period end → advance/paid early. Advance invoices (`inv.due_type = 'Advance'`) are tagged separately. No new schema needed.

**EC-12: All payments by one staff member**
Concentration flag fires (admin only): *"⚠️ [Name] recorded 100% of cash this month."*

**EC-13: Collection efficiency = 100% — all dues collected**
A success banner replaces the unlinked payments alert (or appears above it if unlinked payments also exist): *"All dues collected! Everything is up to date."* with a celebratory illustration.
Population breakdown still shows amounts normally — there's nothing empty about this state.

> **Figma confirms this state:** Screen `8252:104083` is a distinct "all collected" variant with its own success banner and illustration.

> **Interaction with EC-11:** This banner fires only when efficiency = 100% (within ±0.5% rounding tolerance). Efficiency >100% does NOT trigger it — >100% means arrears or advances were collected, not that all current dues are cleared. At >100%, no banner; the efficiency badge shows the actual number in green.

---

## Words on the Screen

### Empty States

| When | Message | CTA |
|------|---------|-----|
| No payments in period | "No payments recorded in this period." | — |
| Custom future range | "Collections are based on received payments. Future dates show ₹0." | — |
| Worklist: filter returns nothing | "No payments match your current filters." | "Clear Filters" |
| No unlinked payments | Alert card hidden | — |

### Error States

| When | Message | Recovery |
|------|---------|----------|
| Network failure | "Couldn't load collections. Check your connection." | "Retry" |
| Section fails to load | "Couldn't load this section." | "Retry" on that section |

### Hero ⓘ Tooltip
> "Collections shows all money received during the selected period. Refunds are subtracted to show net collection. Deposit and advance adjustments are excluded from the net total — they're visible in the Source and Payment Mode breakdowns."

### Efficiency Badge ⓘ Tooltip (≤100%)
> "Collection Efficiency = Amount Collected ÷ Amount that was due by [end date]. Collectible: ₹[X] · Collected: ₹[Y] · Efficiency: [Z]%"

### Efficiency Badge ⓘ Tooltip (>100%)
> "Collection Efficiency: [Z]% · ₹[A] current period dues · ₹[B] old dues recovery · ₹[C] advance received. You collected more than what was due — arrear recovery or advance payments in this period."

---

## Critique of the Lark PRD

> Same lens applied to Dues: treat Lark as a useful first draft, not truth. Identify what's wrong, over-specced, stale, or should be deferred.

### 1. The "Operations" Section Is Scope Creep

Lark PRD has a detailed section for 8 payment operations: Record Payment, Edit Payment, Void Payment, Link Unlinked, Revert Reversal, Download Statement, Generate Receipt, Export to Excel.

**Problem:** These are not analytics features — they're transaction management. An insights/analytics screen is for reading and diagnosing, not for recording or editing. These operations belong in the Payments section of the app, not inside a Financial Insights modal.

**What to keep:** "Export to Excel" is valid as a worklist bulk action. "Link Unlinked" is valid as a contextual CTA inside the Unlinked Payments Alert. Everything else is out of scope for this PRD.

**Action:** Drop Record / Edit / Void / Revert / Download Statement / Generate Receipt from this PRD. They exist elsewhere in the app and don't belong in the analytics screen.

---

### 2. "Summary Cards" Clutter the Hero

Lark PRD shows 6 summary cards above the hero: Total Collected, Pending Settlements, Refunds Given, Adjustments Reconciled, New Tenants, Tenant Turnover.

**Problems:**
- "Pending Settlements" and "Refunds Given" are valid but belong in their own detail sections (settlement status on Payment Mode row; refunds in Refunds DA-03), not floating cards competing with the hero.
- "New Tenants" and "Tenant Turnover" have nothing to do with collections. They are a Tenants section metric, not a Collections metric.
- "Adjustments Reconciled" — this number is only meaningful to an accountant. Rajesh doesn't use this word.

**Action:** Collapse the valid signals into the hero (refunds as deduction tooltip) and Payment Mode section (settlement unsettled indicator). Drop New Tenants and Turnover entirely.

---

### 3. Source Breakdown Labels Don't Match How Operators Think

Lark uses: "Rent Collections / Deposit Collections / Utility Collections / Advance Collections."

**Problem:** This is a category breakdown, not a source breakdown. Lark conflates two separate dimensions:
- **What kind of billing was it?** (Rent / Deposit / Utility → this is Category, section 6B)
- **Was it for current or old dues?** (Current month / Old dues / Advance / Future → this is Source, section 3)

Operators care about *"is this money from this month's billing or old debt recovery?"* — that's the Source question. The Category question is *"what type of bill was it?"*. Mixing these into one chart loses the primary insight.

**Action:** Keep both dimensions but separate them. Source breakdown (section 3) = current/old/advance/future. Category breakdown (section 6B) = by due_type. Lark's labels apply to 6B only.

---

### 4. "Invoices Created" as a Hero Metric Is Misleading

Lark shows "Invoices Created" in the hero alongside Total Collected.

**Problem:** Invoices Created ≠ Collectible. An invoice may be created for a future month (e.g., June invoice created on May 1 via auto-generation). Showing it next to May's collections implies those future invoices are the denominator for efficiency, which is wrong.

**Action:** "Invoices Created" belongs in the Performance accordion (section 6A) as a secondary number. The efficiency denominator is `Collectible` — invoices due on or before period end — not all invoices created.

---

### 5. "Received By" Is Marked as Phase 2 in Lark — Demote It

Lark phases "Received By" as a later enhancement.

**Problem:** `received_by` already exists in the payments table and `applyReceivedBy()` filter code already exists in the backend. The aggregation (`GROUP BY received_by`) is a LOW-effort new build (same effort as payment mode aggregation). The concentration flag is a real governance need for PGs with cash-heavy staff.

**Action:** Bring "Received By" into Phase 1. It's one of the most actionable signals for a PG owner managing staff.

---

### 6. Settlement Tracking Over-Specced for Phase 1

Lark has an entire section for settlement lifecycle: Initiated → Processing → Confirmed → Reversed, with UTR editing, bank account linking, and payout reversal.

**Problem:** UTR status enrichment and settlement status display on the worklist row — EXISTS in code (`formatRow()` in helpers.ts). But the full lifecycle management (initiating payouts, handling reversals, linking bank accounts mid-session) requires payout microservice integration that's out of scope.

**What's in scope for Phase 1:** Show settlement status badge on RentOk rows. Surface unsettled count on the Payment Mode row. Let operator tap to see the worklist of unsettled payments. That's it.

**Action:** Keep the settlement indicator (exists in code). Defer full payout lifecycle to Phase 2. The deferred items are already listed in Phase 2 section.

---

### 7. Month-to-Month Comparison Uses Wrong Baseline in Some Cases

Lark's comparison is always "previous month same period MTD."

**Problem:** For "Last 6 Months," comparing to "previous 6 months" is correct but Lark shows this as a single number — *"vs last 6 months: +12%"*. A single percentage hides whether May drove the improvement or whether all 6 months were better.

**Action:** MoM chip applies only to This Month / Last Month views. For Last 6 Months and Custom ranges, the chip shows "vs previous equivalent period" with a tooltip clarifying the comparison window. No further drill-down on the chip for multi-period ranges — that's what the Trend chart is for.

---

### 8. "Partial Payments Outstanding" Needs Clarification on Scope

Lark says: *"Show invoices that are partially paid."*

**Problem:** This is ambiguous. Does it mean: (a) invoices where at least one payment hit within the period but balance > 0, or (b) all invoices ever partially paid regardless of when the payment was made?

The correct answer is (a) — invoices relevant to the selected period where collection was partial. All partially paid invoices ever is a dues screen problem (it belongs in Dues Outstanding).

**Action:** Partial Payments in the Performance accordion = invoices where a payment was received in the period AND `invoice.status = 0` after that payment (balance remaining). Tap → dues worklist pre-filtered to those invoice IDs.

---

### 9. Void Payment in the Worklist Is Out of Place

Lark lists "Void Payment" as an action on each worklist row.

**Problem:** Voiding a payment changes the financial record retroactively — a destructive action. A financial analytics screen is for reading and acting on insights, not for deleting data. Void belongs in the Payments section with a proper confirmation flow and audit trail.

**What to keep:** Per-row read actions (View Receipt, View Tenant, WhatsApp, Call) are fine — they're non-destructive and directly useful from the context. Void is a write action that should require deliberate navigation, not be a tap away while analyzing analytics.

**Action:** Remove Void Payment from the analytics worklist row actions. Admin can void from the tenant's payment history or the Payments section.

---

## Decisions That Override Lark PRD

| Lark PRD | What we're building | Why |
|---------|--------------------|----|
| Refunds included in scope table | Refunds shown as deduction from gross in hero, NOT in scope as separate line items | Refunds section handles refund detail; here they're a reconciliation deduction only |
| "Advance Adjusted" as a separate section | Advance adjustment (mode 288) shown in Payment Modes as its own row, excluded from net total | Simpler — same data, right place |
| Trend chart inside Advanced Insights accordion | Always visible below accordions | Same rationale as Dues defaulter chart — a health scan shouldn't require a tap |

---

## v2.1 → v3.0 Calibrations (post structured operator audit)

The 2026-05-07 audit found DA-02 was the most cross-screen-critical spec but had the lowest audit-pattern coverage (~30-35%). v2.1 → v3.0 closes critical structural gaps + bugs.

| v2.1 (locked) | v3.0 (current) | Why |
|---|---|---|
| **Hero canonicality silent** | New Rule 0 names canonical aggregation: `SUM(payment.net_amount)` with explicit filters + `ref_agg` CTE pattern | DA-07 HB3 + DA-05 owner-funded reconciliation + DA-06 deposit-collection invariant + DA-03 net-collection hint ALL depended on this. Without Rule 0, four sibling specs ship with broken cross-screen reconciliation. |
| **Mode 211/288 included in Source bar** (per L633 invariant) but excluded from hero | Mode 211/288 EXCLUDED from Source bar entirely; surfaced in dedicated "Adjustments" row below the bar (matches DA-07 Rule 3) | Source bar segments must equal hero. Old design had Source bar ≠ hero — internal contradiction. |
| **Only Rajesh persona narrative** | Added "A Morning with Priya" (multi-property end-of-month) + "A Morning with the CA via WhatsApp screenshot" | DA-07 has 3 personas; CA-screenshot stress-test specifically caught gaps in DA-04/05/07. Priya was buried in EC-07 — now top-level. |
| **EC-10 mode-213 formula has duplicate `gateway_charges`** | Fixed: `net_amount - gateway_charges - owner_credits + credits_used` (single subtraction) | **Real bug.** Engineering would have shipped wrong total per receipt — magnitude of gateway_charges off. |
| **Concentration flag (>50% cash) has no calibration** | Calibrated: ≥ 5 cash receipts AND ≥ ₹10K floor AND property ≥ 60 days old | v2.1 fired on any 1-staff 1-receipt = 100% concentration in setup mode. Matches DA-06 v1.1 discipline. |
| **No Pre-Launch Engineering Blockers section** | Section added (see below) | DA-02 is upstream for 4 specs; engineering needs explicit launch gates |
| **No v2.1 → v3.0 changelog** | This table | v2.1 had no traceable iteration history |
| **Hero subtitle just `(120 Invoices)`** | Subtitle: `Net Collections` + `(N Receipts · net of refunds)` | CA-screenshot reader needs GAAP framing; DA-07 v1.2 pattern |
| **Trend chart "Paid Early" missing from green stack** | Trend chart green bar stacks 4 segments matching Section 3 Source breakdown | Internal consistency |
| **Performance section had duplicate inline bar** (L216 vs L221 in v2.1) | Single bar; bar duplication removed | Editing artifact from v2.0 → v2.1 |

---

## Pre-Launch Engineering Blockers (v3.0)

These items must be resolved before DA-02 ships. Structured per DA-07 v1.2 pattern.

### HARD blockers (mobile screen would be wrong without these)

| Item | Why hard-block | File:Line |
|------|----------------|-----------|
| **HB1. Canonical aggregation path** | Widget query (invoice-first) and list query (payment-first) produce different hero totals for same period. Pick ONE per Rule 0 and route both endpoints through it. **This is the same item as DA-07 HB3 — owned by DA-02.** | `helpers.ts buildBaseQuery` + `service.ts list endpoint` |
| **HB2. Mode 211/288 excluded from Source bar** | Source bar segments include 211/288 in v2.1 invariant but hero excludes them — internal contradiction. v3.0 fixes spec; engineering must apply the filter. | `helpers.ts applyFilterCodes` |
| **HB3. EC-10 formula `gateway_charges` duplicate fix** | v2.1 had bug: `gateway_charges` subtracted twice. v3.0 fixes spec; engineering must verify code. | `helpers.ts mode-213 amount calc` |
| **HB4. Concentration flag calibration** | v2.1 fires constantly in setup mode. v3.0 adds maturity gate + floor + newness bypass. | `helpers.ts concentration detection` |
| **HB5. Hero subtitle "(120 Invoices)" is wrong term** | Should be "(N Receipts)" or "(N Collections)" — Figma had typo flagged in v2.1 but not promoted to launch gate. | `Figma node 7837:99848` |

### DEPENDENCY (cross-screen owned by DA-02; sibling specs unblock when this lands)

| Item | Sibling unblocked |
|------|-------------------|
| **HB1 above** | DA-03 net-collection hint, DA-05 owner-funded reconciliation, DA-06 deposit-collection invariant, DA-07 Section A inflow |

### PARITY blockers (mobile + Excel must match)

| Item | Why parity-block |
|------|------------------|
| **PB1. List query `ref_agg` CTE for refund deduction** | Widget has it; list does not. Cross-tap from hero shows different total than worklist sum. |
| **PB2. Performance "Invoices Created" uses `inv.due_date` not `inv.created_at`** | QA tests must verify; otherwise auto-generated future invoices distort the chip. |
| **PB3. Property scoping `${pg_id}PG${pg_number}` parsing fragility** | Same as DA-03/05/06/07 — properties named "PG Greenwood" break parsing. |

### DEFERRABLE (Phase 1 hotfix safe)

| Item | Why deferrable |
|------|----------------|
| **D1. "Old Tenants" terminology unification** | DA-06 reframed as "Refunds Pending"; DA-02 still uses "Old Tenants" — terminology fix, not data correctness. |
| **D2. Tooltip register expansion** | v2.1 has 3 tooltips; gold standard DA-07 has 9. Add post-launch with copy iteration. |
| **D3. Trend chart Paid Early surfacing** | Visual polish; not data correctness. |

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

| Feature | Why |
|---------|-----|
| **Full payout/settlement dashboard** | Payout lifecycle (initiated → confirmed → reversed) needs payout microservice integration. Phase 2. |
| **Bulk receipt generation** | Generate receipts for all payments in a date range. Phase 2. |
| **Tenant payment pattern analysis** | "Tenants who consistently pay late" — requires multi-period historical analysis. |
| **Cash vs digital trend over time** | Shift from cash to UPI is a governance metric — valuable but not blocking. Phase 2. |
| **Collection velocity vs prior year** | Requires 2+ years of data. Phase 2. |

---

## For Engineering

### Reconciliation Invariants

| Invariant | Formula |
|-----------|---------|
| Hero total = gross − refunds | Net ≠ gross when refunds exist in period |
| Hero total ≠ sum of source segments | Source segments include modes 211/288; hero excludes them |
| Source segments sum = total incl. adjustments | Current + Old + Advance + Future = gross (incl. adj.) |
| Population rows sum = net total | Active + Booking + Old = Net Collection |
| Category rows sum = net total | Rent + Electricity + Food + Deposit + Others = Net |
| Payment mode rows sum = gross (incl. adj.) | All modes including 211/288 = gross |
| Drill-down count = parent count | Header count → worklist row count must match exactly |
| Homescreen card = detail total | Same period + same scope → same net number |

### Codebase Feasibility

> **Source:** `src/v1/list_screens/collections/helpers.ts`, `src/v1/list_screens/collections/service.ts`
> **Reviewed:** 2026-05-01

#### What Already Exists

| PRD Feature | Code Reference | Status |
|-------------|---------------|--------|
| Base query: active paid payments | `buildBaseQuery()` — `p.status=1, p.is_active=1` | **EXISTS** |
| Filter by paid_date range | `applyPaidDateRange()` | **EXISTS** |
| Filter by invoice due_date range | `applyDueDateRange()` | **EXISTS** |
| Filter by payment mode | `applyPaymentMode()` | **EXISTS** |
| Filter by category (due_type) | `applyDueTypes()` | **EXISTS** |
| Filter by received_by | `applyReceivedBy()` | **EXISTS** |
| Filter by tenant type | `applyTenantTypes()` — with null/booking support | **EXISTS** |
| Adjustment mode exclusion | `resolveAdjustmentModes()` — excludes 211, 288 by default | **EXISTS** |
| 18 predefined filter codes incl. settlement | `getWidgetFilterCodes()` + consolidated query | **EXISTS** |
| Settlement status enrichment (UTR strings) | `formatRow()` payout_status → UTR logic | **EXISTS** |
| Receipt URL | `MIN(inv.pdf_link)` in base query | **EXISTS** |
| Search by tenant name/phone | `applySearch()` | **EXISTS** |
| Sort by paid_date, room, net_amount, due_date | `applyOrderBy()` | **EXISTS** |
| Multi-property | `pg_number_filter` param | **EXISTS** |
| Pagination | `executePaginated()` — real, not hardcoded | **EXISTS** |
| Vendor/unlinked payment fallback | `formatRow()` — `paid_by_name/number/room` fallback | **EXISTS** |
| Auth: view invoices + self-added-only | `checkAuth` + `checkAuthInDb` | **EXISTS** |

#### What Needs New Build

| PRD Feature | Gap | Effort |
|-------------|-----|--------|
| **Source breakdown (current/old/advance/future dues)** | No `due_date vs period boundary` classification. Needs: JOIN payments to invoices, compare `inv.due_date` to period start/end to classify each payment into 4 buckets. | **MEDIUM** |
| **Net total = gross − refunds** | Collections service doesn't query `refunds` table at all. Refund amounts need: `LEFT JOIN refunds ON refunds.invoice_id = inv.id`, `SUM(refunds.amount)` per period. | **MEDIUM** — widget query already does this (see `buildWidgetQueryForCode` with `ref_agg` CTE) — replicate for detail |
| **Collection efficiency (collectible denominator)** | No "total invoices due by period end" in collections service. Cross-query to invoices: `SUM(amount) WHERE due_date <= period_end AND status = 0`. | **MEDIUM** |
| **MoM comparison chip** | No prior-period query. | **MEDIUM** |
| **Timing breakdown (on-time/early/late)** | No `paid_date vs due_date` comparison. Needs: join payments → invoices, CASE WHEN `paid_date <= due_date` THEN 'on_time' etc. | **MEDIUM** |
| **Source breakdown amounts (4 buckets)** | Needs bucket logic: `inv.due_date BETWEEN period_start AND period_end` = current; `inv.due_date < period_start` = old; `inv.due_type = 'Advance'` = advance; `inv.due_date > period_end` = future. | **LOW** once join is done |
| **By Category aggregation** | `applyDueTypes` for filtering exists, but no `GROUP BY inv.due_type` query. | **LOW** |
| **By Payment Mode aggregation** | `applyPaymentMode` for filtering exists, but no `GROUP BY p.payment_mode` query. | **LOW** |
| **By Received By aggregation** | `applyReceivedBy` for filtering exists, but no `GROUP BY p.received_by` query. | **LOW** |
| **Unlinked payments query** | Payments with no tenant match (`t.id IS NULL`). Worklist filter: `WHERE t.id IS NULL`. | **LOW** — base query already has LEFT JOIN to tenant |
| **Collection Trend chart** | No time-series aggregation. `GROUP BY DATE_TRUNC('month', p.paid_date)` on payments + similar on invoices for "dues raised". | **MEDIUM** |
| **Partial payments insight** | Needs: invoices with `status = 0 AND amount > 0` that have at least one payment in the period. Cross-query. | **MEDIUM** |
| **Concentration flag** | `GROUP BY p.received_by` on cash payments (mode = cash), check if any one person > 50%. | **LOW** — derivative of Received By aggregation |
| **"X days late" in worklist row** | `paid_date` and `due_date` both selected in base query. Delta not computed in `formatRow()`. | **LOW** |

#### Staleness Flags

| PRD Statement | Reality | Action |
|--------------|---------|--------|
| "Adjustment modes excluded from net total" | `resolveAdjustmentModes()` confirms — modes 211/288 excluded by default | **CONFIRMED** |
| "Widget already has refund deduction" | `buildWidgetQueryForCode()` has `ref_agg` CTE subtracting refunds from invoice amounts | **CONFIRMED** — replicate this pattern for detail screen |
| "Worklist is per-payment (not per-invoice)" | `buildBaseQuery()` from `Payments` entity, grouped by `p.id` | **CONFIRMED** |
| "Advance Adjusted is mode 288" | Code confirms: `ADVANCE_ADJUSTED_THIS_MONTH` = `p.payment_mode = 288` | **CONFIRMED** |
| "Category uses due_date, not paid_date" | Filter codes use `EXTRACT(MONTH FROM inv.due_date)` for category filters | **CONFIRMED** |
| Source breakdown "4 buckets" | Code has no bucket logic yet. The homescreen `getFinancialsV2` has carried_forward / due_in_period / due_after_period for dues, but nothing equivalent on the collections side. | **NEW BUILD** |

#### Key Architecture Notes for Engineering

1. **Widget query vs list query produce different totals** — `buildConsolidatedWidgetQuery()` is invoice-first (prevents double-counting); the list query is payment-first. For the same filter code, these can differ by small amounts. Align both to invoice-first for analytics views.
2. **filter_codes mutually exclusive with manual filters** — when `filter_codes` is set, all of `payment_mode`, `due_types`, `received_by`, `start_date`, `end_date` are discarded. CTA routing for source breakdown drill-downs cannot use both together.
3. **`effectiveInvoiceLevelSum` flag** — category breakdown must set this to true (force `SUM(inv.amount)`) to avoid payment-level amount anomalies when filtering by due_type.
4. **Pagination is real** — unlike dues (hardcoded 5000), collections uses `params.limit ?? 10`. Large properties need real pagination.
5. **Tenant resolution priority** — base query picks the best-match tenant: `status DESC` → active (1) first, then moved-out (2), then pre-join (0), then exited (5). Unlinked payments have `t.id IS NULL`.

### Figma Component Node IDs

| Component | Node |
|-----------|------|
| Collections Section (outer) | `8252:102991` |
| Main screen — data populated | `8252:103565` |
| Main screen — empty/zero state | `8252:103819` |
| Main screen — all collected success | `8252:104083` |
| Advanced Insights — Performance expanded | `8252:103046` |
| Advanced Insights — Payment Modes expanded | `8252:103128` |
| Advanced Insights — By Category expanded | `8252:103230` |
| Advanced Insights — Received By expanded | `8252:103326` |
| Advanced Insights — all collapsed | `8252:103416` |
| Collection Trend — This Month variant | `8252:102992` |
| Collection Trend — 6M detailed variant | `8252:103469` |
| No pending dues illustration | `8277:104452` |
