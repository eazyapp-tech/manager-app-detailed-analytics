---
title: DA-03 Refunds Detailed Analytics
date: '2026-05-06'
tags:
  - rentok
  - prd
  - spec
  - detailed-analytics
  - refunds
  - financials
aliases:
  - DA-03
  - Refunds Detailed Analytics
---
> [!INFO] Source of Truth
> This note is the canonical spec for the Refunds Detailed Insights screen.
> **Engineers:** see [[DA-03 Build Sheet]] (9-column ticket-ready format) for implementation. This doc is the canonical "why."
> **Supporting refs:** [[_Ground Truth Field Map]] · [[_Build Sheet Generation Spec]]
> Supersedes: `RentOk Manager Homescreen/Detailed Analytics/Property Analytics → Finance → Refund Insights.md`
> Related: [[DA-01 Dues Detailed Analytics]] · [[DA-02 Collections Detailed Analytics]] · [[DA-04 Expenses Detailed Analytics]]

# Refunds — Detailed Analytics

> **Product:** RentOk Manager App → Financial Insights → Refunds
> **Version:** 1.3
> **Status:** Final — Locked (codebase + 2 operator audit rounds + Figma audit 2026-05-07)
> **Last Updated:** 2026-05-06
> **Supersedes:** Old PRD "Property Analytics → Finance → Refund Insights" (local file)
> **Design Reference:** Figma `KgBQXiT7r7oGrcqZHCWxyU` → Financials → Refunds container (node IDs TBD)
> **Parent Spec:** Homescreen Financial Insights container

---

## The One Thing This Screen Does

It answers: **"Kitna refund hua, kisko gaya, kab gaya?"**
("How much was refunded, who got it, when did it go out?")

Refunds is a money-out view. Every row on this screen represents money the operator has **already returned** to a tenant — a completed refund, not an obligation. Pending refund obligations (e.g., a tenant who moved out but hasn't received their deposit back yet) live on the **Deposits Held / Liabilities** screen, not here.

---

## A Morning with Rajesh

It's the 7th of May. Three tenants moved out at end of April. Rajesh has been processing deposit returns over the last week.

He opens Refunds. Hero shows ₹85,000 refunded MTD (▲42% vs ₹60,000 in Apr). Red — refunds are up. He taps Reason: Security Deposit — ₹78,000 across 4 entries. Makes sense, 4 move-outs paid back. Caution Money — ₹5,000 to one tenant. Advance — ₹2,000.

He scrolls — a callout: "Largest: ₹35,000 to Anita Sharma · May 3." That was the long-stay tenant from Property 2. He nods. Real.

But also a yellow note at the bottom: *"⚠️ 2 expenses recorded as 'Deposit Refunds' for the same tenants. May be double-counted in totals."* He taps it. Sees Shiv had also entered them in the expense form. Easy fix — delete the duplicate expense entries.

Done in 3 minutes. He knows exactly where the ₹85K went and why.

---

## A Morning with Priya

It's May 14. Priya manages 3 PGs in Mumbai — one luxury (Property 2), one budget (Property 1), one mid-range (Property 3).

She opens Refunds. Total ₹1.4L this month. By Property: Property 2 — ₹95K, Property 1 — ₹35K, Property 3 — ₹10K. Property 2 is 68% of total — but it's also her smallest in headcount. Why?

She taps. Most are Security Deposit returns. Move-out cluster. Then she sees By Processed By: 90% by Manager Aakash at Property 2. She remembers — he flagged a maintenance complaint surge there last month. Tenants exited. She'll call him to debrief.

---

## Scope

**This section shows refunds the operator has already processed — money that has already left their hands.**

| Included | Excluded |
|----------|----------|
| Security Deposit refunds | Pending refund obligations (see Deposits Held screen) |
| Caution Money returns | Refund-eligible balances not yet processed |
| Advance refunds | Gateway-failed refunds (no gateway integration exists in codebase) |
| Rent / fee overpayment returns | Future-dated refunds (refunds use the date money left) |
| Goodwill / waiver refunds (recorded as Refund rows) | Voided / deleted refunds (snapshot in `deleted_refunds` — separate audit view) |
| Manual refunds across any due_type | Wallet credits / in-system adjustments |
| | Refunds blocked at `refund_mode = 2049` (Advance Credits — feature flag-disabled) |

> **Why no "pending" or "failed" view?** The current codebase has no refund status field, no async lifecycle, and no gateway integration. A refund row exists ⇒ the operator has manually disbursed the money. There is no in-progress state to surface. The pending obligation question — "which tenants are still waiting for their deposit?" — is answered by the **Deposits Held / Liabilities** screen, not here. When gateway-integrated refunds are built (Phase 2), this scope will expand.

---

## The Four Rules

1. **A refund row = money already returned.** All entries on this screen are completed disbursements. There is no "pending" or "in-progress" state. If you don't see a refund here, the money hasn't been logged as paid out.

2. **Refund date governs.** All filters apply to `refund.refund_date` — the date entered by the operator at the time of recording. This is operator-entered (back-datable), not the system insert time.

3. **Reason comes from the linked invoice's `due_type`.** The refund table has no native category column. Categories displayed on this screen — Security Deposit, Caution Money, Advance, Rent, Other — are derived from `refund.invoice.due_type`. A refund's category cannot be changed without changing the underlying invoice.

4. **Refunds are a cost view — up is bad.** Same color logic as Expenses. Increase = red. Decrease = green. A refund spike usually means more move-outs.

---

## How Rajesh Gets Here

`Homescreen → Overview → "View Detailed Analytics" → Financial Insights → Refunds tab`

Refunds is the third tab in Financial Insights (Dues · Collections · **Refunds** · Expenses).

---

## What Rajesh Sees

### 1. The Header

**"Refunds"** — with a small ⓘ icon.

No "(Live)" suffix — Refunds is always period-based. Active period (e.g., "May 1–7") is visible in the time filter chip above.

Tapping ⓘ shows:
> "Shows all refunds processed for the selected period. Uses the date the refund was recorded — when money left. An increase vs last month is shown in red (more refunds = more money out)."

---

### 2. Hero KPI — Total Refunds

**The question this answers:** "How much money did I return this period?"

A large number — ₹85,000 — with a subtitle: *(7 refunds across 4 tenants).*

> **Why distinct-tenant count matters:** A single move-out often produces 2–3 refund rows (deposit + caution + advance). "7 refunds" sounds like 7 tenants got money — it's actually 4. Show both counts so operators see real move-out volume, not row volume.

**MoM comparison chip:** ▲42% vs ₹60,000 Apr → red (cost increase). ▼12% → green (cost reduction). For refunds, DOWN = good (less money flowing back).
- Show **both** % AND prior absolute: *"▲42% vs ₹60,000 Apr"*. Operators reason in rupees.
- **Partial-month rule:** If today is May 7th, "vs April" is a 7-day vs 30-day comparison — meaningless. Compare same elapsed days: chip reads *"▲42% vs Apr 1–7"*. Tooltip: *"Comparing May 1–7 vs Apr 1–7 (same 7 days) for a fair comparison."*
- **Prior total = ₹0:** Show *"Not enough prior data"* — avoid ÷0.

**Outlier callout (conditional, below MoM chip):**
If the largest single refund in the period is ≥ 30% of the total OR ≥ ₹25,000:
→ *"Largest: ₹35,000 to Anita Sharma · May 3. Tap to view."* — opens that refund's detail screen. Helps operators verify big-ticket refunds at a glance.

**Net Collection impact (always visible, small text below):**
*"This pulls ₹85,000 out of this month's Collections."* Links to DA-02. Reinforces the cross-screen relationship.

The entire hero block is tappable → opens full refund worklist for the period.

**Multi-property (Priya):** Subtitle reads *(7 refunds · across 3 properties).*

---

### 3. By Property (multi-property only — appears here, before Reason)

**The question this answers:** "Which property is generating refunds — and is the refund volume normal for what that property collects?"

For operators with >1 selected property, this section appears immediately below the hero — before all other breakdowns. This is Priya's first question. Single-property operators don't see this section.

**Each row contains:**
- Property name
- Refund total (₹)
- **Refund-to-collection ratio** for the same period — e.g., `38% of collections` — sourced from DA-02 per-property **GROSS collection** (before refund deduction). This is the operationally-defining metric: ₹95K refund at a property collecting ₹2.5L means a 38% drag, while the same ₹95K at a property collecting ₹6L is a 16% drag — operators care about the ratio, not the absolute.

> **Denominator clarification (cross-spec):** The denominator is **DA-02 GROSS collection** (`SUM(payment.net_amount)` before refund deduction via `ref_agg`), NOT DA-02's net hero. Using net would create a self-referential ratio where increasing refunds inflates the ratio twice (numerator up, denominator down). Same convention as DA-05's discount-to-collection ratio.
- Refund count + distinct tenant count: `7 refunds · 4 tenants`
- ▲/▼ MoM% (suppressed if either period has count ≤ 2 — see "MoM suppression rule" below)
- Proportional bar

Sorted by amount DESC.

**Color cue on the ratio:** ratio < 15% = neutral; 15–30% = amber; > 30% = red. The threshold reflects healthy PG operations (refunds are usually deposit returns; if you're refunding more than 30% of what you collect, something's off).

Tap any row → worklist filtered to that property + current period.

> **MoM suppression rule (applies to all per-row deltas in this PRD):** Refund volume is episodic. Showing "Security Deposit ▲100% vs April" when April had 1 deposit and May has 2 is mathematically right but operationally noise. **If either the current or prior period has count ≤ 2, suppress the % chip and show absolute count delta instead** — e.g., "+1 vs Apr" or "−2 vs Apr." Operators understand counts.

---

### 4. Breakdown by Reason

**The question this answers:** "Why are tenants getting refunded?"

A ranked list — sorted by amount DESC. Each row: reason label, amount, **MoM delta chip**, refund count, proportional bar.

**Reason categories (derived from `invoice.due_type` of the refund's linked invoice):**

| Reason | What it covers | `invoice.due_type` value(s) |
|--------|----------------|----------------------------|
| Security Deposit | Deposit returned at move-out | `Security Deposit` |
| Caution Money | Conditional refund of caution money | `Caution Money` |
| Advance | Pre-payment / advance returned | `Advance` |
| Rent Overpayment | Excess rent refunded | `Rent` |
| Late Fee Reversal | Manual or auto late fee refunded | `Manual Late Fine`, `Automatic Late Fine` |
| Utility / Service | Electricity, food, mess, etc. refunded | `Electricity*`, `Mess*`, `Food*`, `Maintenance*` |
| Other | Anything not in above buckets | All other `due_type` values |

**Category-level MoM delta (per row):** Each row shows its own ▲/▼% vs prior equivalent period — e.g., `Security Deposit · ₹78,000 · ▲52%` in red. Apply same partial-month same-days rule as hero.

**Display logic:**
- Show all reasons with amount > 0, sorted by amount DESC
- If >5 rows: show top 4 + collapsed "Other (N more reasons)" row
- Reasons with ₹0 hidden
- Tap row → worklist filtered to that reason (filter by `invoice.due_type` value(s))

**Collapsed state subtitle:** Top reason + amount. E.g., *"Security Deposit · ₹78,000"*

---

### 5. Breakdown by Refund Mode

**The question this answers:** "How was the money returned?"

A ranked list sorted by amount. Each row: mode icon + label, amount, share, count.

**Refund mode integer mapping** (codes 2040–2049, from `helpers/invoices.ts:1278`):

| Mode | DB value(s) | What it covers |
|------|-------------|----------------|
| Cash | 2040 | Physical cash returned |
| G Pay | 2041 | Google Pay |
| Phone Pe | 2042 | PhonePe |
| Paytm | 2043 | Paytm |
| UPI | 2044 | Generic UPI |
| Bank Transfer | 2045 | NEFT / IMPS / RTGS |
| Card Machine | 2046 | POS / card swipe |
| Cheque | 2047 | Physical cheque |
| Others | 2048 | Other modes |
| Advance Credits | 2049 | **Disabled** — hard-blocked at write time ("coming soon") |

> **Display grouping (locked for P1):** G Pay, Phone Pe, Paytm, UPI (2041–2044) are grouped under a single **"UPI / Digital"** row. Tooltip: *"Includes Google Pay, PhonePe, Paytm, and other UPI apps."* Final displayed modes: Cash · UPI / Digital · Bank Transfer · Card Machine · Cheque · Others. Backend mapping preserved regardless of display grouping.

> **2049 (Advance Credits) is feature-flagged off.** No refund will ever appear under this mode in production today. Do not surface as a row.

Tap any mode row → worklist filtered to that mode (by DB integer value(s)).

**Accordion default:** Collapsed. Most refunds in PG context will be cash or bank transfer — granular split is secondary.

---

### 6. Who Got the Refund

**The question this answers:** "Who received the refunds?"

A ranked list sorted by amount. Each row: tenant avatar (if available) + tenant name + status badge (Active / Moved Out / Cancelled before move-in), amount, refund count.

**Tenant resolution:** `refund.invoice.payer` (firebase_id) → join to `Tenant` table on `firebase_id + property + pg_number`. If multiple tenant rows exist for the same firebase_id (reactivated), pick `status = 1` (active) first, then most recent.

**Tenant status badge:**
- **Active** — tenant currently lives in the PG (`status = 1`). Unusual to refund an active tenant — surface this.
- **Moved Out** — tenant has checked out (`status = 0`). Most refunds will be here. No urgency badge.
- **Cancelled before move-in** — tenant cancelled before joining (`status = 2`). Plain-language label, not "Booking Cancelled."
- **Tenant record removed** — tenant cannot be resolved (deleted / archived). Softer than "Unknown" — operators don't think the screen is broken.

**Active-tenant alert (conditional, below header — tightened threshold):**
Fires only when **active-tenant refund total ≥ ₹2,000 OR ≥ 3 active-tenant refunds in the period**. A ₹50 overpayment correction does not trigger an amber alert.
→ *"⚠️ ₹15,000 went to [N] active tenants. Active tenants don't normally get refunds — check these are correct."*
CTA: "Review" → worklist filtered to active-tenant refunds.

**Repeat-refund flag (per row, conditional):**
If the same tenant has **≥ 2 refunds in the selected period**, show a small chip on their row: `2 refunds`. Two reasons this matters:
- Legitimate: deposit + caution + advance for a single move-out (3 rows per tenant)
- Suspicious: same refund entered twice by accident
Tap chip → worklist filtered to that tenant. Operator can scan for duplicates.

If >4 tenants: collapse remaining into "Others (N tenants) · ₹X" with tap-to-expand.

> **Data quality note:** Tenant resolution is a 3-hop traversal (refund → invoice → payer firebase_id → tenant). If a tenant record can't be resolved, the row falls under "Tenant record removed."

Tap any tenant row → worklist filtered to that tenant's refunds.

---

### 7. Processed By (collapsed by default; auto-hidden for single-staff PGs)

**The question this answers:** "Who on my team processed these refunds?"

A ranked list sorted by amount. Each row: staff name + role badge, amount, refund count.

**Source:** `team_member_transactions` joined on `category IN (REFUND)` with `remark ILIKE '%<refund_uuid>%'` and `is_active = 1`. `team_uuid` resolves to the team member name. Role: `team_member.added_by` (0=Owner, 1=Manager, 2=Partner, etc.).

**Auto-hide rule:** If the property has only one team member with refund attribution AND no "unattributed" fallback rows exist, hide this section entirely. Showing one row "Shiv · ₹85,000" adds nothing. For owner-only PGs (Rajesh recording everything himself), this section never renders.

> **Data quality note:** Two refund flows do NOT create passbook entries:
> 1. The 4 hardcoded `moms_pg_ids` bypass the team passbook entirely.
> 2. If the passbook write fails silently (no transactional atomicity — see Engineering Note 2), the refund row exists without a corresponding passbook entry.
>
> Refunds without a passbook entry appear under **"You / Owner"** (when the operator viewing is the owner) or **"Unattributed"** otherwise. A footnote below the list reads: *"[N] refunds had no staff attribution. If this is unexpected, contact support — a passbook write may have failed."*

If >4 staff: collapse into "Others (N people) · ₹X."

Tap any row → worklist filtered to that team member's refunds.

---

### 8. Where the Money Came From (conditional — see render rules)

**The question this answers:** "Where did the refund money come from?"

**Section render rules:**
- **Single-PG operator (Rajesh) AND no PF involvement:** entire section hidden. Almost all refunds will be from petty cash — operators don't think in fund-name terms; the breakdown is noise.
- **Multi-PG operator (Priya) OR any PF refund exists:** section visible (collapsed by default).
- **PF > 0 (any operator type):** the **Reimbursement chip** below the hero is shown regardless of whether the full breakdown is rendered — that chip is the operationally critical signal, even when the full accordion is hidden.

When rendered, the breakdown shows:

| Fund | Operator-friendly label | Source |
|------|--------------------------|--------|
| Petty Cash (AF) | "Petty cash" | Admin Fund — cash held at the property |
| Staff Personal (PF) | "Staff (reimbursable)" | Money fronted by a team member — needs reimbursement |
| Tenant Funds (NPNAF) | "Tenant funds" | Funds collected from tenants and held |

**Source:** Sum of `team_member_transactions.amount` per `fund_id` for `category = REFUND` minus `category = REFUND_REVERSE`, joined by `remark ILIKE '%<refund_uuid>%'`.

**Reimbursement chip (conditional, shown below hero — independent of section render):**
If PF > 0 → *"₹X owed back to staff for refunds they fronted from personal funds."*
CTA: "View" → Reimbursement screen filtered to refund-sourced PF entries. **Operationally critical** — staff being owed money is a real obligation operators forget about.

---

### 9. Refund Trend Chart (always visible)

Below all accordions, a bar chart is always visible — no tap required.

Each bar = one month's total refund. Color-stacked by top 3 reasons + Other (consistent with DA-04 trend chart).

Time selector: `6M` (default) · `This Month`

> **Default is 6M, not This Month.** Refunds are episodic (move-out clusters, end-of-lease cycles). A single bar for the current month tells you nothing about whether the spike is normal. 6M is the only useful default.

**Trend insight text (conditional):**
- *"Refunds have increased for 3 consecutive months."* (each month > prior)
- *"Most refunds in [Month] this year — likely a move-out cluster."* (peak month identified)
- *"Refunds have been steady across 6 months."* (variance < 15% across all bars)

Bars tappable → tooltip with exact total + reason breakdown for that month. Tap bar → opens worklist for that month's refunds.

---

## Drilling Down: Refund Worklist

Every tappable element opens a filtered refund list.

### Worklist Pre-sets

| Where tapped | Pre-set filter | Default sort | Header |
|-------------|---------------|--------------|--------|
| Hero total | No filter (all refunds in period) | refund_date DESC | "All Refunds · 7 · ₹85,000" |
| By Property row | That property's pg_number_filter | refund_date DESC | "[Property Name] · 4 · ₹56,000" |
| Reason row | `invoice.due_type IN ([values])` | refund_date DESC | "Security Deposit · 4 · ₹78,000" |
| Refund mode row | `refund_mode IN ([db integer values])` | refund_date DESC | "Cash · 3 · ₹15,000" |
| Who Got the Refund row | That tenant's firebase_id | refund_date DESC | "Anita Sharma · 1 · ₹35,000" |
| Processed By row | That team_uuid (via passbook join) | refund_date DESC | "By Aakash · 5 · ₹70,000" |
| Trend chart bar | That month's date range | refund_date DESC | "Apr Refunds · 5 · ₹60,000" |
| Outlier callout | Specific refund UUID → opens detail screen, not worklist | n/a | "Refund Detail" |
| Active-tenant alert | Active tenants only | refund_date DESC | "Refunds to active tenants · 2 · ₹15,000" |

### Per-Refund Row

- Tenant name + status badge (Active / Moved Out / Cancelled before move-in / Tenant record removed)
- Amount (prominent)
- **Full / Partial badge** — derived from `refund.amount` vs `invoice.amount` minus prior refunds:
  - *Full refund* — refund covers the entire remaining invoice balance
  - *Partial — ₹35K of ₹40K (₹5K withheld)* — partial refund with absolute breakdown shown inline
  - This is the single most-asked operator question: "did I refund everything they were owed?" Surfaces it without making them tap.
- Refund date (`refund_date`)
- **Held duration** — small text "Held 14 months" — computed as `refund_date − earliest payment date on the linked invoice`. Helps spot anomalies: a 2-day hold on a deposit is suspicious; a 14-month hold is normal end-of-stay.
  - **Negative-days guard (v1.2):** if `held_days < 0` (operator back-dated `refund_date` earlier than the earliest payment), hide the chip + log a data-integrity warning. Don't render "Held -2 months."
- Reason (derived from `invoice.due_type`)
- **Reason note (when present)** — `refund.refund_reason` truncated to ~40 chars. Operators write notes like "damage to mattress" or "early exit" — the spec previously dropped this; surface it. Tooltip shows full text. If null, this line is hidden.
- Linked invoice context: "Against: Invoice #1234 (₹50K) · paid Apr 12"
- Refund mode badge (resolved from integer to label)
- Processed by: team member name from passbook + role badge
- Property name (multi-property only)
- **Possible-duplicate hint (when applicable)** — chip "Possible duplicate?" appears if same tenant + same amount + within 24 hours of another refund in the worklist. Hint, not a block.

> **Note on "voided indicator":** Earlier draft had a "voided" red flag for live rows when a `deleted_refunds` snapshot existed for the same tenant in the period. This contradicted EC-11 (deleted refunds don't appear in DA-03 at all). **Removed.** Voided refund visibility lives entirely on the Phase 2 Refund Audit screen.

### Per-Refund Actions

- **View Detail** — opens `/refunds/advanced-details` for that refund (banner card + linked invoice + fund split)
- **View Linked Invoice** — opens the invoice this refund was processed against
- **View Tenant** — opens tenant detail screen (resolved via firebase_id)

### Worklist Filters

Operator can further filter: Date range · Reason (due_type) · Refund mode · Tenant status (Active / Moved Out / Cancelled) · Recorded by · Property (multi-property)

### Bulk Actions

- **Export to Excel** — **NEW BUILD.** No refund export endpoint exists in the codebase (`/generateExpenseReport` is expense-only). Phase 1 must build a refund export endpoint that emails a `.xlsx` to the operator's registered email. Fields: Date | Tenant | Tenant Status | Reason | Amount | Refund Mode | Processed By | Property | Linked Invoice | Fund Split (AF/PF/NPNAF — code-level fund IDs). Toast: *"Export sent to [email]. You'll receive it in a few minutes."*

---

## Drill-Down Behavior

> **Universal navigation rules apply to all 7 DA specs.** Priorities (P0/P1/P2) are PM recommendations; engineering may re-prioritize during spike.

### Universal Rules

**R1. Modal/Sheet/Screen primitive [P0]** *(updated 2026-05-11)* — Every tap target's destination is explicitly typed: full-screen push, bottom sheet, modal overlay, or inline accordion. **ⓘ icon convention (locked across DA suite):** single-tap → bottom sheet. No inline tooltip. No long-press. See `[[_Build Sheet Generation Spec#15. ⓘ Icon Interaction Convention]]`.

**R2. Back-stack semantics [P0]** — Back pops one navigation frame and restores prior frame's filter chips, scroll position, accordion state, selected segments. iOS swipe-back and Android system back behave identically.

**R3. Deep links + share sheet [P1]** — Every drill state is uniquely URL-addressable as `rentok://da-NN/<view>?<filters>`. Push notifications and WhatsApp deep-links generate these URLs.
> Engineering note: align with existing `rentok://<screen>` singular-form router pattern.

**R4. Permission gating UX [P0]** — Sections without read permission HIDE (not gray). Disabled actions show lock icon + toast on tap. Cross-screen drill into denied screen shows full-screen denial; back returns to source bottom sheet. Mobile must inspect `can_view_invoices` flag before showing drill targets.

**R5. Loading states [P0]** — Skeleton-on-load (not spinner). Cross-screen transitions show destination header immediately + skeleton rows. Pull-to-refresh shows chevron without flashing skeletons.

**R6. State preservation [P1]** — Tab switches preserve per-tab filter and scroll state. App background within 15 min restores exact drill state. Force-quit returns to homescreen.
> Engineering note: 15-min threshold is PM-suggested; calibrate during spike.

**R7. Multi-property scope inheritance [P0]** — Single-property drill OVERRIDES global scope for the entire descendant drill stack. Scope chip always visible in worklist header.

**R8. Worklist filter-chip behavior [P0]** — Every pre-applied filter is a removable chip. Removal re-fetches without that filter; others stay. New filters are additive (AND). Worklist filter changes do NOT propagate back to dashboard.

**R9. Shareable state [P2]** — "Share this view" affordance in overflow menu generates deep link + system share sheet.
> Engineering note: net-new feature; defer to Phase 2 if scope-constrained.

**R10. CA-screenshot discipline [P0]** — Every hero has a visible GAAP subtitle and basis label — never tap-only. No critical context in tap-only tooltips.

**R11. Cross-screen back path [P1]** — Cross-screen drill pushes destination as CHILD of source bottom sheet. Destination shows breadcrumb "← From [Source DA name]".
> Engineering note: if no breadcrumb support today, fall back to standard back-stack pop (R2).

### Priority Summary

| Priority | Rules | Engineering guidance |
|----------|-------|---------------------|
| **P0 (must have)** | R1, R2, R4, R5, R7, R8, R10 | Universal mobile expectations |
| **P1 (should have)** | R3, R6, R11 | Significant UX value; depends on existing app router/persistence patterns |
| **P2 (defer if needed)** | R9 | Net-new feature beyond Excel export |

### Per-Spec Specifics (DA-03)

- **Refund Detail screen** (`/refunds/advanced-details`) = full-screen push. **HB4 prerequisite:** auth check must be added before exposing.
- **Outlier callout tap** → directly to Refund Detail (skips worklist — operator already knows the specific refund).
- **View Linked Invoice** → invoice detail screen (existing app feature, full-screen push).
- **View Tenant** → tenant detail screen (existing, full-screen push).
- **Reimbursement chip "View"** → Reimbursement screen (owned by separate spec). Cross-spec wikilink in body text is for reader navigation only; in-app tap target opens Reimbursement.
- **Active-tenant alert "Review"** → worklist filtered to active-tenant refunds (R7 multi-property scope inheritance applies).
- **Repeat-tenant chip on row** → tap → worklist filtered to that tenant's refunds (matches DA-03 narrative — clear in spec).
- **Long-pending refund detection** is part of DA-06 EC-13 (cross-spec); DA-03 surfaces refunds via worklist filter, DA-06 surfaces stale liabilities.

---

### Universal Rule Clarifications (post-orphan-audit)

Resolves interaction-primitive ambiguities surfaced during the post-master orphan-tap-target audit. Treat as additive to R1–R11 above.

**R1 clarification — explicit primitives for common UI:**
- **Hero ⓘ icon:** single-tap → bottom sheet with plain-English explanation + GAAP framing + basis label. **No inline tooltip. No long-press.** Per Generation Spec §15.
- **MoM chip:** tap = inline tooltip showing prior-period numbers + computation window. No drill.
- **Accordion section:** tap on row OR chevron toggles expand/collapse. Default state per section spec.
- **Information-only chips** (held-duration on a row, possible-duplicate hint, repeat-refund chip, reason-note tooltip): single-tap = inline tooltip explaining the flag. No drill unless explicitly listed in Per-Spec Specifics.

**R5 clarification — pull-to-refresh:** Dashboard re-fetches all sections in parallel; worklist re-fetches with current chips. Cross-screen drill destinations also support pull-to-refresh. Chevron animates without flashing skeletons.

**R8 clarification — filter chip ✕ as explicit tap target:** Every chip has a discrete ✕ icon (44pt min hit area). Tapping ✕ removes that chip and re-fetches; other chips stay. Body-tap opens edit affordance (date picker, multi-select for Reason / Refund mode / Tenant status / Recorded by / Property) where applicable; otherwise no-op.

**R12 (NEW) — Trend chart conventions [P0]:**
- **Bar single-tap:** inline tooltip showing exact refund values + count. No drill.
- **Tap-into-tooltip CTA "View [period] →":** drills to refund worklist filtered to that period.
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
| `viewRefunds` (cited) | DOES NOT EXIST | Build (new column + JWT mirror at 5 sites + frontend toggle), OR reuse `viewInvoices`. **Recommend BUILD** — refund visibility is distinct from invoice visibility for compliance/audit purposes |
| `processRefunds` (cited) | DOES NOT EXIST as JWT key | DB has `add_refund_access` and `delete_refund` (snake_case, `checkAuthInDb` only). Use these instead of building new key |
| `viewInvoices` | EXISTS — JWT key | For "View Linked Invoice" cross-screen drill |
| `viewTenants` | EXISTS — JWT key | For tenant-detail drill (replaces fictional `viewTenantDetails`) |

> **Decision owner: Jatin (Sr Backend).** Each MISSING key requires build-or-reuse decision. Specs cannot ship until Jatin signs off.

> **Reimbursement controller duplication note:** `routes/reimbursement.ts:10` reuses `RefundsController.getRefundDetails` — same auth gap as `routes/refunds.ts:10` (HB4). When fixing HB4, must patch both controller files (`controllers/refunds.ts` + `controllers/reimbursement.ts`). See CSB-7.

---

## Time Filter Behavior

| Component | Behaviour |
|-----------|-----------|
| Hero total | `refund.refund_date` in selected period; uses live (non-deleted) `Refunds` rows only |
| MoM chip | **This Month / Last Month:** "vs [MonthName]". **Custom range:** "vs equivalent prior period" with same-days tooltip. **Prior period = ₹0:** show "Not enough prior data" (avoid ÷0). |
| Outlier callout | Largest refund in period (by amount) — recomputed per period change |
| Reason breakdown | `refund_date` in period, grouped by `invoice.due_type` |
| Refund mode breakdown | `refund_date` in period, grouped by `refund.refund_mode` |
| Who Got the Refund breakdown | `refund_date` in period, grouped by `invoice.payer` (firebase_id) |
| Processed By breakdown | `refund_date` in period, grouped by `team_member_transactions.team_uuid` (REFUND minus REFUND_REVERSE) |
| Where the Money Came From breakdown | `refund_date` in period, sum by `team_member_transactions.fund_id` |
| Trend chart | Has its own 6M / This Month selector — not governed by global time filter |

**Custom future dates:** Show ₹0 for future portion. Helper text: *"Refunds use the date money left. Future dates show ₹0."*

---

## Edge Cases

**EC-01: No refunds in period**
Hero shows ₹0 (0 refunds). MoM chip hidden. All accordions show *"No refunds processed for this period."* Secondary hint: *"Wrong period? Change the date filter."*

**EC-02: Single refund dominates the total (≥ 80%)**
Outlier callout fires automatically with that refund's tenant and amount. Reason breakdown will show one row with ~100%. Valid. Rajesh sees this as a single big move-out — expected.

**EC-03: Negative or zero amount refund entry**
Theoretical — runtime rejects `<= 0` at write time. If any legacy data leaks through, exclude silently from totals (no banner — this is too rare to surface as operator-visible noise).

**EC-04: Tenant resolution fails (deleted / orphaned firebase_id)**
The Who Got the Refund row groups under **"Tenant record removed"** at the bottom. Worklist row shows the same status badge. Refund itself still counts in the total — the data is real, only the tenant identity is missing.

**EC-05: Refund linked to deleted invoice**
Rare. The refund persists with a dangling FK. In the worklist, "Linked invoice" shows "Invoice deleted." Refund still counts in totals. Engineering should add a LEFT JOIN with NULL handling.

**EC-06: Partial refunds — multiple rows on same invoice**
Each row appears separately. Reason breakdown sums across all rows for that invoice's due_type. Reconciliation invariant: sum of all refund.amount per invoice ≤ invoice.amount.

**EC-07: Refund mode = 2049 (Advance Credits — feature-flagged off)**
Should never appear (write is blocked). If legacy data has any, group under "Others" with footnote: *"[N] entries with disabled refund mode shown as Others."*

**EC-08: MoM and refund-to-collection ratio — divide-by-zero or insufficient prior data**
- **Hero MoM:** if prior period total = ₹0, or property is < 30 days old, show *"Not enough prior data"* instead of a percentage. Don't show ▲∞%.
- **Refund-to-collection ratio (per property):** if DA-02 collection for that property in the same period = ₹0, hide the ratio entirely (show "—") with tooltip *"No collections recorded for this property this period."* Don't compute ÷0.
- **Newness bypass:** properties < 60 days old skip the ratio color cue entirely (matches DA-06 discipline).

**EC-09: Multi-property (Priya) but only one property has refunds**
By Property breakdown shows one row at 100% share. Other properties hidden (₹0). Valid.

**EC-10: Refund created without team passbook entry**
Two scenarios: (1) hardcoded `moms_pg_ids` bypass the passbook entirely; (2) passbook write failure was swallowed silently. Refunds without a passbook record appear under "You / Owner" (when viewer is owner) or "Unattributed" otherwise in Processed By, and contribute to Where the Money Came From breakdown as "Unattributed." Footnote: *"[N] refunds have no team-member attribution."* Engineering Note 2 covers the underlying integrity gap.

**EC-11: Refund deleted (snapshot in `deleted_refunds`)**
Deleted refunds do NOT appear in any breakdown — they're hard-deleted and snapshot to a separate table. If an operator wants to see voided/deleted refunds, that's a separate "Refund Audit" screen (Phase 2). DA-03 shows only live refunds.

**EC-12: Refund issued to active tenant (status = 1)**
Triggers the active-tenant alert in section 6. Common legitimate cases: rent overpayment correction, mistake adjustment. Worth surfacing because it's unusual.

**EC-13: (Removed)**
Previously documented CirclePe-blocked invoices. CirclePe is dead (per project memory) — dropping to reduce noise.

**EC-14: Same-day reverse (created and deleted within minutes)**
The deleted_refunds snapshot will exist for a few minutes' span. The live refund total reflects only what's currently live. No special handling needed.

**EC-15: Cross-screen double-count with DA-04 Expenses**
**Critical operator integrity issue.** If an operator both (a) creates a Refund row, AND (b) manually logs the same disbursement as an Expense with `expense_type ILIKE 'Deposit%'`, both totals appear in their respective screens — and the cash-flow report at `generateCashFlowReport.ts:393` adds them (`totalExpenses = expenseTotal + refunds`). Same money counted twice.

A footer warning appears on this screen if heuristic match detects this:
*"⚠️ [N] entries appear twice — once here as a refund and once in Expenses as 'Deposit Refund'. Your totals are inflated by ₹[X]."*
CTA: **"Fix"** → expense worklist filtered to the overlapping entries with bulk-delete option.

Heuristic for matching: same `pg_number` + `paid_to` matches tenant name from refund + amount within 5% + date within 7 days of `refund_date` + `expense_type ILIKE 'Deposit%'`.

**EC-16: Inverse double-count — deposit refund logged ONLY as expense**
The opposite of EC-15. If an operator records a deposit return only via the Expense form (never as a Refund row), this DA-03 screen will under-count their actual refund volume. The hero says ₹85K refunded; the operator actually returned ₹1.05L (₹20K hidden in Expenses).

Detection heuristic: scan `expenses` for `expense_type ILIKE 'Deposit%'` entries in the period whose tenant name (`paid_to`) does NOT have a matching Refund row in this screen. If any exist, footer banner:
*"₹[X] in 'Deposit Refund' expenses are not logged as refunds. Your refund total may be understated."*
CTA: "Review" → expense worklist filtered to those entries with "Convert to refund" option (Phase 2 — Phase 1 just surfaces the gap).

**EC-17: Same tenant, same amount, same day — possible duplicate**
Operator accidentally entered the same refund twice. Worklist row shows "Possible duplicate?" chip when same `payer firebase_id` + amount + `refund_date` (date only, time ignored) appear ≥ 2 times. Does NOT exclude from totals — operator decides if it's a real duplicate (rare but possible: two deposits of the same amount on the same day) or an entry error.

**EC-18: Refund period ≠ original invoice period**
A May refund for a deposit collected last September. The "Held duration" on the worklist row makes this transparent ("Held 8 months"). For analytics totals, the refund counts in May (its `refund_date` period). Operator gets context without confusion: large May spike = lots of long-held deposits being released, not a sudden cost surge.

**EC-19: Refund recorded against an unpaid invoice**
A refund row exists but the linked invoice has no payment (`invoice.payments` empty or all `payments.status != 1`). Mathematically odd — refunding money that wasn't collected. Surface in the worklist row with a small flag: "Linked invoice has no payment — review." Don't exclude from totals; the refund is real money out, but the audit trail is broken.

---

## Words on the Screen

### Empty States

| When | Message | CTA |
|------|---------|-----|
| No refunds in period | "No refunds in this period. Wrong period? Change the date filter." | "Change filter" |
| Custom future range | "Refunds use the date money left. Future dates show ₹0." | — |
| Worklist: filter returns nothing | "No refunds match your current filters." | "Clear Filters" |
| Who Got the Refund: all tenants unresolved | "Tenant records couldn't be resolved for any refund this period. Check tenant data integrity." | — |
| New property, no refunds yet | "No refunds yet. They'll appear here when you process deposit returns or other refunds." | — |

### Error States

| When | Message | Recovery |
|------|---------|----------|
| Network failure | "Couldn't load refunds. Check your connection." | "Retry" |
| Section fails to load | "Couldn't load this section." | "Retry" on that section |
| Cross-module check (double-count) fails | (silent — show breakdowns without the warning footer) | — |

### Hero ⓘ Tooltip
> "Shows all refunds processed for the selected period. Uses the date the refund was recorded. An increase vs last month appears in red (more money out)."

### Reason Breakdown ⓘ Tooltip
> "Reasons are taken from the invoice each refund was processed against. Most refunds will be deposit returns at move-out."

### Refund Mode ⓘ Tooltip
> "How the money was returned — cash, digital transfer, cheque, etc."

### Who Got the Refund ⓘ Tooltip
> "Who received the money. Most are tenants who moved out. Active tenants getting refunds is unusual — check those are correct."

### Processed By ⓘ Tooltip
> "Which team member processed each refund. Some refunds may show as 'You / Owner' or 'Unattributed' if staff attribution wasn't captured."

### Where the Money Came From ⓘ Tooltip
> "Which fund the refund came from — petty cash, money fronted by staff (needs reimbursement), or tenant funds held."

### Net Collection Hint (small text below hero)
> "This pulls ₹[X] out of this month's Collections."

### Refund-to-Collection Ratio ⓘ Tooltip (in By Property section)
> "Refunds as a percentage of what this property collected this period. Healthy is below 15%. Above 30% suggests something's off — complaint surge, quality issue, or data entry."

### Loading States

| Component | Loading behaviour |
|-----------|-------------------|
| Hero KPI | Skeleton matching hero — number placeholder + MoM chip placeholder |
| Each accordion | Header visible immediately; 3 skeleton rows in content area |
| Trend chart | Grey placeholder rectangle; loads after accordions |
| Overall failure | Hero shows error state; accordions show individual "Retry" |

---

## Critique of the Old PRD

> Old PRD: `Property Analytics → Finance → Refund Insights.md`. Mostly skeleton — many table cells empty. Where it was specific, it was often wrong.

### 1. Old PRD scoped out pending and failed refunds — but in our codebase, those don't exist anyway

The old PRD explicitly excluded `refund_status NOT IN ('success', 'settled')` from scope. The operator review flagged this as the biggest miss — operators want a pending queue.

**Reality:** The codebase has **no `refund_status` field**, no async lifecycle, no gateway integration. Every refund row is implicitly "completed" because that's the only state the schema supports. There is nothing to surface for "pending" or "failed" until gateway integration is built.

**Action:** The "pending refunds" question is genuinely valid for operators — but it belongs to a different domain entity (Deposits Held / Liabilities, where the operator owes money but hasn't disbursed yet). DA-03 explicitly directs operators there. Gateway-driven refund lifecycle moves to Phase 2.

---

### 2. Old PRD invented a `refund_status` enum with values "success / settled"

These string values do not exist in the codebase. The `Invoices.status` enum has a comment for `3-Refunded` and `Payments.status` has a comment for `3-refunded` — both are unused.

**Action:** Drop these from the spec. The only state model is "row exists" vs "row deleted (snapshotted)."

---

### 3. Old PRD had only 3 reason categories: Security Deposits, Advance Refunds, Other Refunds

This is too coarse for diagnosis. The codebase exposes far more via `invoice.due_type`: Security Deposit, Caution Money, Advance, Rent, Manual/Automatic Late Fine, Electricity, Mess, Food, Maintenance, plus more.

**Action:** Use the real `due_type` taxonomy, grouped sensibly into 7 displayable categories (Section 4 of this spec).

---

### 4. Old PRD had a "Refunded By" / "Staff Attribution" section with no source field identified

The `refunds` entity has `refunded_by_name` and `refunded_by_phone` fields — but they're populated **only** for partner-token sessions. For owner/manager flows (the majority), they're null.

**Action:** Drop the dedicated `refunded_by` columns. Use `team_member_transactions.team_uuid` (joined via passbook `category = REFUND` and `remark ILIKE '%<refund_uuid>%'`) as the actual staff attribution source. Caveat: hardcoded `moms_pg_ids` bypass the passbook entirely — those refunds will have no attribution.

---

### 5. Old PRD assumed gateway-specific refund tracking (Cashfree, Easebuzz, etc.)

There is **no gateway refund integration in the codebase**. All refund modes (2040–2048) are manual disbursements. 2049 (Advance Credits) is hard-blocked at write.

**Action:** Drop the gateway references. All refunds are operator-recorded manual events.

---

### 6. Old PRD's "By Property" section was buried at position 7 (between Staff and Audit)

For Priya, "Which property is generating refunds?" is the first question, not the seventh.

**Action:** Move By Property to position 3 — directly below the hero — for multi-property operators (mirror DA-04 EC-06).

---

### 7. Old PRD did not call out the cash-flow double-count risk

`generateCashFlowReport.ts:393` adds `expenseTotal + refunds` into `totalExpenses`. If an operator manually logs a deposit refund both as an Expense (under `expense_type ILIKE 'Deposit%'`) AND as a Refund row, they double-count.

**Action:** EC-15 and a UI footer warning surface this to operators in DA-03. Engineering Note 1 mandates a fix in the cash-flow logic.

---

## Decisions That Override Old PRD

| Old PRD | What We're Building | Why |
|---------|--------------------|----|
| Status enum: 'success' / 'settled' | No status field — every Refund row = completed | Codebase has no refund status field |
| Only completed refunds in scope; exclude pending/failed | Same scope (completed only), but explicitly redirect pending obligations to Deposits Held screen | No pending state exists in the codebase |
| 3 reason categories (Security/Advance/Other) | 7 categories derived from `invoice.due_type` | Real taxonomy from the joined invoice |
| `refunded_by` columns on refunds entity | `team_member_transactions.team_uuid` from passbook | Refund table fields are partner-only / mostly null |
| Gateway-specific tracking | No gateways involved | Refunds are manual in this codebase |
| By Property at position 7 | By Property at position 3 (multi-property only) | Priya's first question |
| Single MoM chip on hero | Hero MoM + per-row MoM on Reason and By Property | Diagnostic tool — operators need to see WHICH category changed |
| No outlier callout | "Largest refund this period" callout when ≥ 30% of total | Cheap to compute, high signal for big-ticket reviews |
| No cross-screen reconciliation language | Hero shows "Reduces net Collections by ₹X"; EC-15 surfaces double-count with Expenses | Required for trust — DA-02 hero math must match |
| No fund split visibility | Where the Money Came From breakdown (AF/PF/NPNAF) with reimbursement chip when PF > 0 | Operationally important for staff reimbursement |
| Refund mode display: granular 9 modes | Grouped: Cash · UPI/Digital · Bank · Card · Cheque · Others | Same UX rationale as DA-04 |
| MoM partial-month: full prior month | Same-elapsed-days comparison ("vs Apr 1–7") | Avoid misleading partial vs full comparison |
| Trend chart default: This Month | Default 6M | Refunds are episodic — single-month view is meaningless |
| Hero count: just refund count | "(7 refunds across 4 tenants)" — show distinct-tenant count too | One move-out → 2-3 refund rows; row count overstates real move-out volume |
| Worklist row: refund amount only | Add Full/Partial badge with absolute breakdown | Most-asked operator question: "did I refund everything?" |
| Worklist row: no held duration | Show "Held [N] months/days" — refund_date minus earliest invoice payment | Spotting a 2-day hold on a deposit is a fraud signal; 14-month hold is normal |
| `refund_reason` text dropped | Surface on worklist row (truncated) | Operators write notes there; spec previously hid their own data |
| Refund-to-collection ratio per property: P2 | Elevated to P1 in By Property | Priya's defining metric — without it the property comparison is half useful |
| Active-tenant alert: any active-tenant refund | Threshold: ≥ ₹2,000 OR ≥ 3 active-tenant refunds | A ₹50 correction should not trigger a yellow alert |
| Where the Money Came From breakdown: always shown | Auto-hidden for single-PG with no PF involvement; reimbursement chip remains | AF/PF/NPNAF jargon adds no value when 100% is petty cash |
| Processed By section: always shown | Auto-hidden if only one staff with attribution | "Shiv · ₹85,000" as the only row tells operator nothing |
| Per-row MoM %: always shown | Suppressed when count ≤ 2 in either period; show count delta instead ("+2 vs Apr") | Episodic refunds make small-N percentages meaningless |
| Repeat-refund detection | Chip on tenant row when ≥ 2 refunds in period; possible-duplicate chip on worklist row when same tenant + amount within 24h | Catches both legitimate splits (deposit + caution) and accidental duplicates |
| Voided indicator on live worklist rows | Removed | Contradicted EC-11; voided refund visibility is Phase 2 audit screen only |
| EC-15 copy: "may double-count" | "Your totals are inflated by ₹X" + "Fix" CTA | Operators need certainty, not hedging |
| Inverse double-count (deposit refund only logged as expense) | New EC-16 — surface but don't auto-fix in P1 | Without this, DA-03 silently under-reports refunds |
| "Recorded By" / "Money Source" / "Booking Cancelled" / "Unknown" | "Processed By" / "Where the Money Came From" / "Cancelled before move-in" / "Tenant record removed" | Operator-friendly language vs internal jargon |

---

## v1.0 → v1.1 → v1.2 Calibrations (post 2 operator audit rounds)

### v1.1 → v1.2 changes (second-round operator audit, 2026-05-07)

| Item | v1.1 | v1.2 |
|---|---|---|
| 18 orphan label references in subsidiary tables | "Refunded To" / "Recorded By" / "Money Source" / "Unknown" / "System / Owner" still appeared in Worklist Pre-sets, Time Filter Behavior, Empty States, Reconciliation Invariants, Codebase Feasibility, Architecture Notes | All swept to "Who Got the Refund" / "Processed By" / "Where the Money Came From" / "Tenant record removed" / "You / Owner" or "Unattributed" |
| EC-04 directly contradicted L239 | EC-04 said groups under "Unknown" + "Tenant not found" | EC-04 rewritten: groups under "Tenant record removed" matching L239 |
| Refund-to-collection ratio ÷0 unhandled | Ratio computed regardless of denominator | EC-08 expanded: when DA-02 collection for property = ₹0, hide ratio with "—" + tooltip "no collections this period" |
| Days held could go negative | `refund_date` operator-back-datable; held_days = refund_date − earliest_payment can be negative | Worklist row hides chip when held_days < 0 + logs data-integrity warning |
| Distinct-tenant copy ambiguous when 1 tenant | "(7 refunds across 4 tenants)" / no spec for single-tenant case | Add explicit copy: "(3 refunds to 1 tenant)" |
| Repeat-tenant + EC-17 duplicate hint stack on same row | Both can fire | Suppress repeat-refund chip when EC-17 "Possible duplicate?" fires |
| EC-16 inverse double-count detection logic vague | "scan expenses for tenant name match" | EC-16 explicitly = complement of EC-15 over deposit-named expenses in period |
| Pre-Launch Engineering Blockers section absent | Engineering risks scattered across 14 unranked Architecture Notes | Section added (see below) |

### v1.0 → v1.1 changes (first-round audit)
See "Decisions That Override Old PRD" table above.

---

## Pre-Launch Engineering Blockers

These items must be resolved before DA-03 ships. Structured per DA-07 v1.2 pattern.

### HARD blockers (mobile screen would be wrong)

| Item | Why hard-block | File:Line |
|------|----------------|-----------|
| **HB1. Cash flow expense+refund double-count** | Affects DA-03 EC-15 detection accuracy. Same fix as DA-04 HB3 + DA-07 HB1. **Note: this is DIFFERENT from DA-05's "Refund→Credit-status integrity bug" — they share the word "refund" but are at different file locations and require different fixes.** | `generateCashFlowReport.ts:393` |
| **HB2. No transactional atomicity on refund creation** | Three writes (refund row, invoice update, passbook call), no DB transaction. Passbook errors swallowed silently. Result: Processed By + Where the Money Came From breakdowns systemically broken at low rate. | `controllers/invoices.ts addRefundToInvoice` + `teamPassbook.ts:4333` |
| **HB3. `moms_pg_ids` hardcoded bypass auth + passbook** | 4 hardcoded property IDs skip both. Mobile screen will silently under-attribute refunds at those properties. Either remove the bypass list or document each property's expected behaviour. | `invoices.ts:7774-7779`, `7988-7997` |
| **HB4. `/refunds/advanced-details` endpoint has NO auth check** | Refund Detail screen (`getRefundDetails`) accepts only `pg_id + refund_uuid`, no permission validation. Any authenticated user can read any refund. **Add `checkAuth(viewInvoices)` or equivalent.** Confirmed by codebase audit Apr 2026 (master branch). | `src/services/refunds/refunds.ts:488` |

### PARITY blockers (mobile + Excel must match)

| Item | Why parity-block | File:Line |
|------|------------------|-----------|
| **PB1. New `/generateRefundReport` endpoint must use same aggregations as mobile widget** | Mobile + Excel render same fields; same query path required. | (NEW BUILD) |

### VERIFICATION (folded into other blockers)

| Item | Folded into |
|------|-------------|
| **Stale TODOs in `deleteRefundFromInvoice`** | HB2 QA — fix as part of refund-write transaction work. | `invoices.ts:8151, 8179, 8194` |

### DEPENDENCY (owned by another spec)

| Item | Owner |
|------|-------|
| **DA-02 hero refund-line reconciliation** | DA-02 spec — DA-03 hero must match DA-02's "Refunds Issued" line for same period |

### DEFERRABLE (post-launch hotfix safe)

| Item | Why deferrable |
|------|----------------|
| **D1. `lastIndexOf('PG')` parsing fragility** | Production audit recommended (properties named "PG Greenwood"); not a launch block. |
| **D2. `refundMultipleInvoices` HTTP self-call cleanup** | Two patterns coexist; pick one. Hotfix-safe. |
| **D3. Dead columns `refunded_by`, `refund_images`, `status`** | Schema cleanup; not user-facing. |

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
| **Refund status lifecycle (initiated → approved → processed → completed / failed)** | Requires schema additions: status, gateway, gateway_refund_id, failure_reason, initiated_at, processed_at, webhook_payload. Not built today. |
| **Pending refunds queue** | Depends on lifecycle above. Until then, "pending" is captured in Deposits Held, not Refunds. |
| **Failed refund callout + retry CTA** | Depends on lifecycle + gateway integration. No gateway today. |
| **Refund age / SLA buckets (0–2d / 3–7d / 7+d)** | Needs initiated-vs-completed timestamps. Single `refund_date` insufficient. |
| **Gateway-driven refund processing** | Cashfree / Easebuzz / PayU / RazorpayX integration. Net-new build. |
| **Tenant-side notifications (refund processed SMS/WhatsApp)** | Today only owner gets `refund_alert_3` WhatsApp. Tenant comms = net-new. |
| **Approval workflow (request → approve → process)** | Today: flat permissions only. Multi-step approval = new module. |
| **Refund Audit screen (deleted_refunds snapshots)** | Voided refund visibility. Separate audit module. |
| **Cross-module reconciliation engine (auto-detect double-count)** | EC-15 heuristic warning is P1. Auto-merge / auto-fix is P2. |
| **Refund cause taxonomy beyond `due_type`** | Free-text `refund_reason` is too dirty for analytics. Either add a structured cause enum or NLP-categorize. |
| **High-volume alert (2x trailing 3-month average)** | Performance concern; needs validation that alert is signal, not noise. |
| **Auto-convert: deposit-named expense → refund row (EC-16)** | Phase 1 detects and surfaces; Phase 2 offers a one-tap "Convert to refund" action that creates a Refund row and deletes the duplicate Expense in one transaction. |
| **Refund-to-collection ratio for hero / homescreen overview** | Per-property ratio is now Phase 1 (in By Property). Hero-level ratio across all properties is still Phase 2 (cross-screen aggregation). |
| **Refund mode 2049 (Advance Credits)** | Currently feature-flagged off in codebase. Build when ready. |
| **Reimbursement workflow integration** | PF-funded refunds become reimbursable items. P1 surfaces the chip; P2 deep-links to the full reimbursement queue. |
| **Concentration flag (one staff doing >60% of refunds)** | Same logic as DA-04 — defer to a Governance/Audit module. |

---

## For Engineering

### Reconciliation Invariants

| Invariant | Formula |
|-----------|---------|
| Reason sum = hero total | All `invoice.due_type` group sums = Total |
| Refund mode sum = hero total | All `refund_mode` group sums (incl. Others) = Total |
| Who Got the Refund sum = hero total | All tenant group sums (incl. "Tenant record removed") = Total |
| Processed By sum = hero total | All team_uuid sums (incl. "You / Owner" / "Unattributed" fallback) = Total |
| Where the Money Came From sum = hero total | AF + PF + NPNAF (live, post REFUND_REVERSE netting) = Total |
| By Property sum = hero total | All property group sums = Total |
| Worklist count = parent count | Header count → worklist row count must match exactly |
| **Cross-screen: DA-03 hero = DA-02 hero "Refunds Issued" line** | Same period + same property scope — must match |

### Codebase Feasibility

> **Source:** `src/entities/refunds.ts`, `src/services/refunds/refunds.ts`, `src/controllers/invoices.ts` (addRefundToInvoice / deleteRefundFromInvoice), `src/v1/constants/filterCodes.ts`
> **Reviewed:** 2026-05-06

#### What Already Exists

| PRD Feature | Code Reference | Status |
|-------------|---------------|--------|
| Refund creation (single invoice) | `addRefundToInvoice` (`invoices.ts:7748`) | **EXISTS** |
| Refund creation (multi-invoice spread) | `RefundsService.addRefund` (`refunds.ts:273`) | **EXISTS** |
| Refund detail view | `RefundsService.getRefundDetails` (`refunds.ts:488`) | **EXISTS** |
| Refund deletion + snapshot | `deleteRefundFromInvoice` (`invoices.ts:8119`) | **EXISTS** |
| Refund listing per invoice | `getRefundsFromInvoice` (`invoices.ts:8212`) | **EXISTS** |
| Permission checks | `add_refund_access`, `delete_refund` via `checkAuthInDb` | **EXISTS** |
| Where the Money Came From split (AF/PF/NPNAF — code-level fund IDs) | `team_member_transactions` joined by remark (`refunds.ts:586`) | **EXISTS** |
| Refund mode label resolution | `getRefundModeForInvoice` (`helpers/invoices.ts:1278`) | **EXISTS** |
| Refund-against-invoice cap | `invoice.amount - SUM(refund.amount)` cap (`invoices.ts:7971`) | **EXISTS** |
| Move-out checklist gate (deposit refunds) | `is_refund_deposit_blocked_until_all_checklists_locked` (`invoices.ts:7876`) | **EXISTS** |
| WhatsApp owner alert on refund | `refund_alert_3` template (`invoices.ts:8042`) | **EXISTS** |
| `deleted_refunds` snapshot table | `deletedRefundsRepository` | **EXISTS** |

#### What Needs New Build

| PRD Feature | Gap | Effort |
|-------------|-----|--------|
| **Analytics list endpoint (`/v1/refunds/list/filters`)** | No equivalent of expense/collection list_screens for refunds. Mirror the pattern from `expenses/helpers.ts` and `expenses/service.ts`. | **MEDIUM** |
| **Widget endpoint (`/v1/refunds/list/widget`)** | No widget endpoint. Build with reason/mode/property/recipient widget codes. | **MEDIUM** |
| **Refund filter codes enum (RefundFilterCode)** | No enum exists. `CollectionFilterCode.REFUND = 1209` is a no-op. Allocate a new range, e.g., 1500-1599. | **LOW** |
| **Reason aggregation (GROUP BY invoice.due_type)** | New JOIN + GROUP BY on `refunds → invoices.due_type`. | **LOW** |
| **Mode aggregation (GROUP BY refund_mode)** | New GROUP BY query. | **LOW** |
| **Who Got the Refund aggregation** | New GROUP BY `invoice.payer` + JOIN to Tenants for status badge. | **MEDIUM** — tenant join is 3-hop |
| **Processed By aggregation** | JOIN `team_member_transactions` ON `category = REFUND` AND `remark ILIKE '%<refund_uuid>%'` AND `is_active = 1`. Subtract REFUND_REVERSE. | **MEDIUM** |
| **Where the Money Came From aggregation** | Same join as Processed By; sum by `fund_id`. | **LOW** (after Processed By is built) |
| **By Property aggregation** | Parse `invoice.property` string ("{pg_id}PG{pg_number}") OR add a derived column. | **LOW** |
| **MoM comparison (per-row + hero)** | Prior-period query + delta calculation. | **MEDIUM** |
| **Trend chart aggregation (6M)** | `GROUP BY DATE_TRUNC('month', refund_date)` with reason stacking. | **MEDIUM** |
| **Outlier callout (largest refund detection)** | `MAX(amount)` query + threshold check (≥ 30% of total OR ≥ ₹25,000). | **LOW** |
| **Active-tenant alert detection** | JOIN tenant; filter where `tenant.status = 1`. | **LOW** |
| **Cross-screen double-count detector (EC-15)** | Heuristic JOIN: `expenses.expense_type ILIKE 'Deposit%'` + `paid_to ILIKE tenant_name` + amount within 5% + date within 7 days. | **MEDIUM** |
| **Refund Excel export (`/generateRefundReport`)** | No equivalent of `/generateExpenseReport`. New endpoint. Mirror the email-based pattern. | **MEDIUM** |
| **Net collection reconciliation chip** | Match DA-02 hero subtraction. Re-use DA-02's `ref_agg` CTE. | **LOW** |
| **Worklist drill-down filter (by reason via ILIKE/IN on invoice.due_type)** | Use exact `IN` match on `invoice.due_type` values listed in the spec. | **LOW** |

#### Critical Architecture Notes for Engineering

1. **CRITICAL — Cash-flow double-count must be fixed.** `generateCashFlowReport.ts:393` does `totalExpenses = expenseTotal + refunds`. If a deposit refund is logged BOTH as an Expense (`expense_type ILIKE 'Deposit%'`) AND as a Refund row, both totals land in `totalExpenses`. Either dedupe on creation (block the duplicate Expense), or expose deduplication at read time. EC-15 is a UI warning; this is the underlying engineering fix.

2. **CRITICAL — No transactional atomicity on refund creation.** `addRefundToInvoice` saves refund row, then invoice, then calls `TeamPassbookService.registerRefund` — three writes, no DB transaction. The passbook call is wrapped in a try/catch that swallows errors (`teamPassbook.ts:4333` — note the copy-paste bug "Error registering expense" in a refund handler). A refund row can persist with no passbook entry, breaking the Processed By and Where the Money Came From breakdowns. Wrap all three writes in a single DB transaction; surface passbook failures as 500s.

3. **`refunds.refund_mode` is integer (2040–2049).** No string enum. Source: `helpers/invoices.ts:1278`. Mode 2049 (Advance Credits) is hard-blocked at write — never appears in production.

4. **`refunds` has only one FK: `invoice_id`.** Tenant lookup is 3-hop (refund → invoice → payer firebase_id → Tenant). Property lookup is via parsing `invoice.property` string. Add a JOIN performance index on `refunds.invoice_id`. Consider denormalizing tenant_id and pg_number onto refunds for analytics if performance becomes an issue.

5. **`refunds` has no `is_active` flag.** Soft-delete doesn't exist; deletes are hard, with a snapshot to `deleted_refunds`. All analytics queries hit the live `refunds` table — voided refunds are simply absent.

6. **`refund_date` is operator-entered, not system-generated.** It's overridable from the request body and back-datable. Operators can also leave the time component meaningless (a backfill script `RefundTimeCorrectionScript` confirms this). For analytics, treat `refund_date` as date-only (truncate time).

7. **`refunds.refunded_by`, `refunds.refund_images`, and `refunds.status` are dead columns.** Never written. Do not return in the formatted analytics row.

8. **Stale TODO comments in `deleteRefundFromInvoice`.** Lines 8151, 8179, 8194 have `// TODO: check access`, `// TODO: add activity log`, `// TODO: add whatsapp alert`. Fix as part of DA-03 work — analytics depends on consistent delete handling.

9. **Hardcoded `moms_pg_ids` bypass auth AND passbook.** `invoices.ts:7774-7779` and `7988-7997` skip both. Refunds at these properties have NO passbook entry → NO Processed By or Where the Money Came From attribution. EC-10 surfaces this; the underlying bypass needs review.

10. **Tenant resolution `lastIndexOf('PG')` parsing is fragile.** `refunds.ts:574` uses `invoice.property.lastIndexOf('PG')` to derive `pg_number`. Will break on properties whose name contains "PG" (e.g., "PG Greenwood"). Add validation or migrate to a structured FK.

11. **`refund_reason` is free text and mostly null.** Useful only as a tooltip / detail field. Do NOT use as a primary breakdown dimension. Reason categories come from `invoice.due_type`.

12. **Deleted refunds (`deleted_refunds.refund_json`) are JSONB snapshots.** Useful for audit but not queryable for analytics. Phase 2 audit screen would denormalize these.

13. **Filter code allocation:** A `RefundFilterCode` enum doesn't exist. Allocate a fresh range (suggested 1500–1599) for refund widget codes. Do NOT extend `CollectionFilterCode` further.

14. **`refundMultipleInvoices` (`invoices.ts:7548`) makes HTTP self-calls.** It POSTs to `${RENTOK_BASE_URL}/invoices/addRefundToInvoice` per invoice. The newer `RefundsService.addRefund` uses an in-process mock req/res — two patterns coexist for the same operation. Pick one; align with the analytics list endpoint.
