---
title: DA-04 Expenses Detailed Analytics
date: '2026-05-06'
tags:
  - rentok
  - prd
  - spec
  - detailed-analytics
  - expenses
  - financials
aliases:
  - DA-04
  - Expenses Detailed Analytics
---
> [!INFO] Source of Truth
> This note is the canonical spec for the Expenses Detailed Insights screen.
> **Engineers:** see [[DA-04 Build Sheet]] (9-column ticket-ready format) for implementation. This doc is the canonical "why."
> **Supporting refs:** [[_Ground Truth Field Map]] · [[_Build Sheet Generation Spec]]
> Supersedes: `RentOk Manager Homescreen/Detailed Analytics/PRD — Financial Detailed Insights • Expenses.md`
> Related: [[DA-01 Dues Detailed Analytics]] · [[DA-02 Collections Detailed Analytics]]

# Expenses — Detailed Analytics

> **Product:** RentOk Manager App → Financial Insights → Expenses
> **Version:** 1.4
> **Status:** Final — Locked (revised post 2 operator audit rounds + Figma audit 2026-05-07)
> **Last Updated:** 2026-05-06
> **Supersedes:** Old PRD "Financial Detailed Insights • Expenses" (local file)
> **Design Reference:** Figma `KgBQXiT7r7oGrcqZHCWxyU` → Financials → Expenses container (node IDs TBD from Figma)
> **Parent Spec:** Homescreen Financial Insights container

---

## The One Thing This Screen Does

It answers: **"Kitna kharch hua, kahan gaya, kisne kiya?"**
("How much was spent, where did it go, and who spent it?")

Expenses is a cost view — it shows money going out, not coming in. Every component exists to help Rajesh understand whether his spending is in control, where it's going, and who is driving it.

---

## A Morning with Rajesh

It's the 5th of May. Rajesh manages two PGs in Pune. He checks the app.

Expenses shows ₹82,000. The MoM chip: ▲18% vs April. Red — costs are up. He taps Category. Salary is flat. Electricity jumped — ₹24,000 vs ₹14,000 last month. Two PGs, both spiked. He taps Electricity, sees 4 entries — all recorded by Shiv on the 3rd. Two properties, two bills. Makes sense — summer. He taps Paid To — "Maharashtra Electricity Board" at the top, ₹24K. Consistent with the bills. Nothing suspicious.

Done in 2 minutes. No spreadsheet. He knows it's seasonal, not a leak.

---

## A Morning with Priya (multi-property check, May 14)

Priya manages 3 PGs in Mumbai. She opens DA-04. Hero ₹2.3L this month (▲12% vs April). Per-bed: ₹540/bed. By Property: Property 2 — ₹1.1L (highest). She taps. Category breakdown shows Maintenance ▲₹35K vs Apr (red). Top expense category by MoM swing.

She taps Maintenance → worklist filters to 8 entries. Top 3 are at Property 2: AC repair ₹15K, plumbing ₹10K, electrical rework ₹8K. All recorded by Manager Aakash. She remembers the complaint cluster he flagged. Real, contextual, expected.

She also notices the "no bill" callout: *"4 expenses have no bill attached."* Three are at Property 2, one at Property 1. Aakash again. She'll text him for receipts.

Done in 4 minutes. The screen surfaced the cost driver, the property, and the staff accountability gap.

---

## A Morning with the CA via WhatsApp screenshot

Priya forwards her DA-04 hero to her CA. The CA opens the screenshot.

Hero shows ₹2.3L (▲12%), ₹540/bed, "4 expenses have no bill" amber chip. Below the hero, micro-label: *"Cash basis · paid date."* CA understands: this is what was actually paid out in May, not what was billed in May.

She replies: *"Send me the Excel — I need to see the entries."* Priya goes to overflow menu (⋮) → Email Cash Flow Excel → done.

Without the cash-basis micro-label, the CA would have spent 3 messages clarifying the basis.

---

## Scope

**This section shows money spent — recorded expenses only.**

| Included | Excluded |
|----------|----------|
| Operational costs (electricity, maintenance, rent) | Refunds issued to tenants (DA-03) |
| Salary and wage payments | Security deposit held (liability) |
| Vendor payments | Internal fund transfers |
| Food and kitchen costs | Negative-amount entries (shown as warning) |
| Deposit refunds paid out (`expense_type ILIKE 'Deposit%'`) | Future/scheduled expenses not yet recorded |
| Reimbursements recorded as expenses | Collections received |
| Complaint-linked maintenance expenses | |

> **"Deposit Refunds" category:** When a tenant's security deposit is returned and recorded as an outgoing expense entry (not through the deposit settlement flow), it appears here under Deposit. This is distinct from the Deposits module — it's cash paid out, recorded via the expense form.

---

## The Four Rules

1. **Paid date governs.** All filters apply to `e.paid_date`. `e.invoice_date` exists in the entity but is not used for filtering or display. A May electricity bill recorded in June appears in June's expenses, not May's.
2. **Categories are ILIKE-matched.** The system classifies expenses by `e.expense_type` using prefix matching (e.g., `'Salary%'`, `'Electricity%'`). Free-text entry means some expenses may not match any standard category and fall into "Other."
3. **The numbers must match.** Every tappable count and amount must exactly match the drill-down worklist. Same date range, same property scope, same filter. The four breakdown dimensions (Category, Payment Mode, Paid By, Paid To) must each sum to the hero total.
4. **Expenses is a cost view — up is bad.** MoM color logic is inverted vs. Collections: increase = red, decrease = green.

---

## How Rajesh Gets Here

`Homescreen → Overview → "View Detailed Analytics" → Financial Insights → Expenses tab`

Expenses is the fourth tab in Financial Insights (after Dues, Collections, Refunds). Swipe or tap the Expenses chip in the top nav.

---

## What Rajesh Sees

### 1. The Header

**"Expenses"** — with a small ⓘ icon.

No "(Live)" suffix — Expenses is always period-based. The active period (e.g., "May 1–5") is visible in the time filter chip above.

Tapping ⓘ shows:
> "Shows all recorded expenses for the selected period and properties. Uses the payment date — when money left the account — not the date the expense was entered. An increase vs last month is shown in red (costs are up)."

---

### 2. Hero KPI — Total Expense

**The question this answers:** "How much money went out this period?"

A large number — ₹82,000 — with a subtitle: *(34 Expenses).*

**Cash-basis micro-label (small grey text below subtitle):**
*"Cash basis · paid date"*

> **Why this micro-label (v1.3):** Second-round audit caught the CA-via-WhatsApp persona-stress-test failure. CAs viewing a forwarded screenshot can't tap the Hero ⓘ tooltip — they have no signal whether the number is cash-basis (paid_date) or accrual-basis (invoice_date). Permanent micro-label visible in screenshots removes 3 round-trip messages. Same discipline as DA-07's GAAP subtitle for accountants.

Three elements in the hero block:

**MoM comparison chip:** ▲18% vs ₹69,500 Apr → red (cost increase). ▼8% → green (cost reduction). For expenses, DOWN = good.
- Show **both** the percentage AND the prior-period absolute: *"▲18% vs ₹69,500 Apr"*. Operators reason in rupees, not percentages.
- **Partial-month rule:** If today is May 5th, "vs April" is a 5-day vs 30-day comparison — meaningless. Compare same elapsed days: chip reads *"▲18% vs Apr 1–5"*. Tooltip on chip: *"Comparing May 1–5 vs Apr 1–5 (same 5 days) for a fair comparison."*
- **Prior total = ₹0:** Show *"Not enough prior data"* — avoid ÷0.

**Governance callout (below MoM chip, conditional):**
If ≥ 3 expenses in the period have `bill_urls IS NULL OR bill_urls = '{}'`:
→ *"[N] expenses have no bill attached — tap to review."* (amber chip)
- **At N = total expense count (100% missing):** copy reads *"No bill attached on any expense — tap to review."*
- **At < 3 expenses missing:** callout suppressed (noise threshold).
- **Placement:** when MoM chip is hidden (EC-01, EC-11), this callout moves up to be flush against the hero number.

CTA opens worklist filtered to `bill_urls IS NULL OR bill_urls = '{}'`.

This is the primary receipt-coverage control. Operators use it to chase staff for missing receipts.

> **Threshold added v1.3:** Earlier draft fired on any N ≥ 1. A single missing-bill expense in a 50-expense month triggered the chip — alarm fatigue. Floor of ≥ 3 missing focuses on real coverage gaps.

> **Amber banner stack rule (v1.3):** EC-02 (negative amounts), EC-13 (null paid_date), and this "no bill" callout can all co-fire. **Cap at one amber callout above the hero, in priority order: (1) null paid_date EC-13, (2) negative amount EC-02, (3) no bill governance callout.** Lower-priority callouts move into the worklist's filter-chip set.

**Per-Bed Expense (multi-property only, inline below the number):**
`₹820 per occupied bed`
Calculated as: Total Expense ÷ Currently Occupied Beds (live count from occupancy).
- Renders ONLY when multi-property is selected. Single-property operators don't see this row.
- Shows "—" if occupied beds = 0
- Tap → tooltip: *"₹[Total] ÷ [N] occupied beds = ₹[X] per bed. Occupancy count is current (live), not period-average."*
- **Multi-property:** Uses sum of occupied beds across all selected properties.

> **Multi-property gating (v1.3):** Earlier draft showed per-bed for single-property operators too. Second-round audit (matching DA-07's lesson) cut it for single-property — there's no comparison group to anchor the number. A single-PG operator seeing "₹820/bed" with no benchmark just ignores it. Multi-property operators benefit from cross-property comparison.

The entire hero block is tappable → opens full expense worklist for the period.

**Multi-property (Priya):** Subtitle reads *(34 Expenses · across 3 properties).*

---

### 3. Breakdown by Category

**The question this answers:** "Which type of expense is driving the spend?"

A ranked list — sorted by amount descending. Each row: category icon + label, amount, **MoM delta chip**, proportional bar.

**System-defined categories (aligned to codebase):**

| Category | What it covers | Code match |
|----------|----------------|------------|
| Salary | Staff wages, salaries | `expense_type ILIKE 'Salary%'` |
| Electricity | Power bills | `expense_type ILIKE 'Electricity%'` |
| Mess / Food | Kitchen, groceries, catering | `ILIKE 'Mess%' OR ILIKE 'Food%'` |
| Maintenance | Repairs, upkeep, AMC | `expense_type ILIKE 'Maintenance%'` |
| Rent | PG's own building rent | `expense_type ILIKE 'Rent%'` |
| Deposit Refunds | Security deposit returned as expense | `expense_type ILIKE 'Deposit%'`. **Cross-spec note:** DA-07 Cash Flow EXCLUDES this category from Operating Outflows (Section A); deposit returns appear in DA-07 Section B "Deposit Money In/Out" instead. DA-04 keeps it as a category for completeness — operators searching "where did this expense go" still find it here. |
| Other | Everything not matching above | NOT ILIKE any of above |

**Category-level MoM delta (per row):** Each row shows its own ▲/▼% vs the prior equivalent period alongside the amount — e.g., `Electricity · ₹24,000 · ▲71%` in red. This is the primary diagnostic tool: operators scan down and find the red number without tapping. Same inversion logic as hero (▲ = red = bad, ▼ = green = good). Apply partial-month same-days rule here too.

**Small-N suppression (v1.3):** Per-row MoM chip is suppressed when EITHER:
- Current period count < 3 expenses in that category, OR
- Prior period count < 3 expenses in that category, OR
- Absolute amount delta < ₹2,000

When suppressed, show absolute delta only: *"+₹500 vs Apr"* or no chip at all if delta < ₹500. Same discipline as DA-05 Reason rows. Prevents misleading ▲500% chips on 1-expense-vs-1-expense categories.

**Display logic:**
- Show all categories with amount > 0, sorted by amount DESC
- If >5 rows: show top 4 + collapsed "Other (N more categories)" row
- Categories with ₹0 hidden
- Tap any row → worklist filtered to that category (uses ILIKE, not IN)

**Collapsed state subtitle:** Top category + amount. E.g., *"Salary · ₹42,000"*

**Accordion default:** Collapsed. Operator expands on demand.

Tap any row → worklist filtered to `expense_type ILIKE '[category]%'`.

---

### 4. Breakdown by Payment Mode

**The question this answers:** "How was money paid out — and how much was cash?"

A ranked list sorted by amount. Each row: mode icon + label, amount, bar (share of total).

| Mode | DB value(s) | What it covers |
|------|-------------|----------------|
| Cash | 202, 2040 | Physical cash paid out (2040 = system default when mode not specified) |
| G Pay | 2041 | Google Pay |
| Phone Pe | 2042 | PhonePe |
| Paytm | 2043 | Paytm |
| UPI | 2044 | Generic UPI (any other UPI app) |
| Bank Transfer | 2045 | NEFT / IMPS / RTGS |
| Card Machine | 2046 | POS / card swipe |
| Cheque | 2047 | Physical cheque |
| Others / Wallet | 2048 | Any other mode; also set automatically for wallet-funded expenses |

> **Display grouping (locked for P1):** G Pay, Phone Pe, Paytm, and UPI (2041–2044) are grouped under a single **"UPI / Digital"** row. Tooltip on that row: *"Includes Google Pay, PhonePe, Paytm, and other UPI apps."* The 5 displayed modes are: Cash · UPI / Digital · Bank Transfer · Card Machine · Cheque · Others/Wallet. Underlying integer mapping preserved in the backend query regardless of display grouping.

Tap any mode row → worklist filtered to that payment mode (by DB integer value(s)).

> **Cash governance nudge — deferred to Phase 2.** PG operators pay cash for almost everything (salaries, vendor runs, maintenance). A nudge firing every month for virtually every operator adds noise and erodes trust in the screen. Moved to Phase 2 for validation with operators first.

---

### 5. Breakdown by Paid By

**The question this answers:** "Who on my team recorded these expenses?"

A ranked list sorted by amount. Each row: avatar + name + role badge, total amount, expense count.

**Role mapping for `added_by` field:**

| added_by value | Display label |
|----------------|---------------|
| 0 | Owner |
| 1 | Manager |
| 2 | Partner |
| 3 | Customer |
| 4 | Tenant |
| 5 | Other / System |

The row groups by `e.payer` — a **free-text name string** entered when the expense was recorded (e.g., "Jatin", "Shiv", "Owner"). It is NOT a firebase_id or system user ID. `added_by` is displayed as the role badge. Two entries with the same name string merge into one row; two people who entered different spellings appear separately.

> **Data quality note:** `payer` is free-text, same as `paid_to`. "Jatin" and "jatin" are separate rows. A footnote below the list reads: *"Names are as entered. Different spellings of the same person appear as separate rows."*

If >4 rows: remaining collapse into "Others (N people) · ₹X" with tap-to-expand.

Tap any row → worklist filtered to `payer = that name string` (exact match on the stored free-text value).

> **Concentration flag — deferred to Phase 2.** In a single-manager PG, one person will naturally account for 80%+ of recorded expenses — this is expected, not suspicious. The flag adds noise for the majority use case. Move to a dedicated Governance / Audit module in Phase 2 where it can be designed properly with context.

**Multi-property (Priya) — "By Property" section appears here:**
For operators with multiple properties, a **"By Property" breakdown appears immediately below the hero block** (before Category) — this is Priya's first question. Single-property operators don't see this section.

Each property row: property name, total expense, share of total, expense count. Sorted by amount DESC. Tap → worklist filtered to that property + current period.

---

### 6. Breakdown by Paid To

**The question this answers:** "Who received this money — which vendors or payees?"

A ranked list sorted by amount. Each row: payee name, amount, expense count.

> **Data quality note:** `paid_to` is a free-text field. "Maharashtra Electricity Board" and "MH Electricity Board" are treated as two separate payees — they won't be merged. Surface this to the operator: a footnote below the list reads *"Payee names are as entered. Different spellings of the same vendor appear separately."*

If >4 payees: remaining collapse into "Others (N payees) · ₹X" with tap-to-expand.

**Empty payee:** Expenses with no `paid_to` value → grouped as *"Not specified (N expenses)"* at the bottom.

Tap any row → worklist filtered to `paid_to = that payee name` (exact match).

---

### 7. Expense Trend Chart (Always Visible)

Below the four accordions, a bar chart is always visible — no tap required.

Each bar = one month's total expense, broken into category colour-stacked segments (top 3 categories + Other).

**"This Month" view:** Single bar for the active period.
**"6M" view:** One bar per month for the last 6 months.

Time selector on the chart itself: `6M` (default) · `This Month`

> **Default is 6M, not This Month.** On May 5th, "This Month" shows a single stub bar for 5 days — meaningless in isolation. 6M gives Rajesh immediate trend context: is this month on track, or already spiking? The partial-month bar in 6M view is shorter than complete months — operators understand this intuitively.

**Trend insight text (conditional):**
- "Expenses have increased for 3 consecutive months." (each month > prior)
- "Costs down vs last month — good sign." (current month < prior month)

Bars are tappable → tooltip showing exact total + category breakdown for that period. Tap bar → opens expense worklist for that month.

---

## Drilling Down: Expense Worklist

Every tappable element opens a filtered expense list.

### Worklist Pre-sets

| Where tapped            | Pre-set filter                                                             | Default sort   | Header                               |
| ----------------------- | -------------------------------------------------------------------------- | -------------- | ------------------------------------ |
| Hero total              | No filter (all expenses in period)                                         | paid_date DESC | "All Expenses · 34 · ₹82,000"        |
| Category row            | `expense_type ILIKE '[category]%'` (NOT `applyCategories()` which uses IN) | paid_date DESC | "Electricity · 4 · ₹24,000"          |
| Payment mode row        | `payment_mode IN ([db integer values])`                                    | paid_date DESC | "Cash · 12 · ₹18,000"                |
| Paid By row             | `payer = that name string` (exact match on free-text value)                | paid_date DESC | "By Shiv · 8 · ₹24,000"              |
| Paid To row             | `paid_to = that payee name` (exact match)                                  | paid_date DESC | "MH Electricity Board · 2 · ₹24,000" |
| Trend chart bar         | That month's date range (independent of global time filter)                | paid_date DESC | "Apr Expenses · 28 · ₹69,000"        |
| By Property row (EC-06) | `pg_number_filter = [that property's pg_number]`                           | paid_date DESC | "[Property Name] · N · ₹X"           |

### Per-Expense Row

- Category badge + description (`e.description`)
- Amount (prominent)
- Paid date (`e.paid_date`)
- Invoice date if different from paid date — shown as secondary line: *"Billed: [date]"* (uses `e.invoice_date`; only show when `invoice_date ≠ paid_date`)
- Paid to (payee name) — if blank, show *"Payee not specified"*
- Payment mode badge (resolved from integer to label)
- Recorded by: `e.payer` (name) + `e.added_by` (role badge)
- Bill photo icon if `bill_urls` is non-empty (array length > 0; treat null and `[]` identically — no icon)
- Complaint link icon if linked via `complaint_expense_map`

> **Multiple bills:** If `bill_urls` has >1 entry, tapping the icon opens a gallery/carousel showing all bills. Do not open just the first URL.

### Per-Expense Actions

- **View Bill** — opens bill_urls gallery (single image, or carousel if multiple)
- **View Complaint** — if complaint-linked, opens complaint detail
- **Call** — `partner_phone` is NOT returned in the formatted row (stripped at API layer). Remove this action from Phase 1. Defer to Phase 2 if needed.

> **Note on partner fields:** `partner_name`, `partner_id`, `partner_phone` exist in the entity but are stripped from `formatExpenseRow()` in the v1 list API. They are not available in the analytics list response. Do not spec UI that depends on them until the API is updated.

### Worklist Filters

Operator can further filter: Date range · Category · Payment mode · Recorded by · Paid to · Property (multi-property)

### Bulk Actions

- **Export to Excel** — triggers `POST /generateExpenseReport`. The export is **email-based**: an `.xlsx` is generated and emailed to the operator's registered email address. It is NOT an inline download. The exported file includes: Date | Amount | Category | Description | Payer | Paid To | Payment Mode | Bill URLs. A summary header shows: total amount, total count, top-4 categories by spend. Operator sees a toast: *"Export sent to [email]. You'll receive it in a few minutes."*

---

## Drill-Down Behavior

> **Universal navigation rules apply to all 7 DA specs.** Priorities (P0/P1/P2) are PM recommendations; engineering may re-prioritize during spike.

### Universal Rules

**R1. Modal/Sheet/Screen primitive [P0]** *(updated 2026-05-11)* — Every tap target's destination is explicitly typed: full-screen push, bottom sheet, modal overlay, or inline accordion. **ⓘ icon convention (locked across DA suite):** single-tap → bottom sheet. No inline tooltip. No long-press. See `[[_Build Sheet Generation Spec#15. ⓘ Icon Interaction Convention]]`.

**R2. Back-stack semantics [P0]** — Back pops one navigation frame and restores prior frame's filter chips, scroll position, accordion state, selected segments. iOS swipe-back and Android system back behave identically.

**R3. Deep links + share sheet [P1]** — Every drill state is uniquely URL-addressable as `rentok://da-NN/<view>?<filters>`. Push notifications and WhatsApp deep-links generate these URLs.
> Engineering note: align with existing `rentok://expenses` singular-form router pattern.

**R4. Permission gating UX [P0]** — Sections without read permission HIDE (not gray). Disabled actions show lock icon + toast on tap. Cross-screen drill into denied screen shows full-screen denial; back returns to source bottom sheet. Mobile must inspect `can_view_expenses` flag before showing drill targets. Master added auth checks to `/v1/expenses/list/filters` (Apr 2026); detail endpoint `/expenses/advanced-details` still missing auth (HB4).

**R5. Loading states [P0]** — Skeleton-on-load (not spinner). Cross-screen transitions show destination header immediately + skeleton rows. Pull-to-refresh shows chevron without flashing skeletons.

**R6. State preservation [P1]** — Tab switches preserve per-tab filter and scroll state. App background within 15 min restores exact drill state. Force-quit returns to homescreen.
> Engineering note: 15-min threshold is PM-suggested; calibrate during spike.

**R7. Multi-property scope inheritance [P0]** — Single-property drill OVERRIDES global scope for entire descendant drill stack. Scope chip always visible in worklist header.

**R8. Worklist filter-chip behavior [P0]** — Every pre-applied filter is a removable chip. Removal re-fetches without that filter; others stay. New filters are additive (AND). Worklist filter changes do NOT propagate back to dashboard.

**R9. Shareable state [P2]** — "Share this view" affordance in overflow menu generates deep link + system share sheet.
> Engineering note: net-new feature; defer to Phase 2 if scope-constrained.

**R10. CA-screenshot discipline [P0]** — Every hero has visible GAAP subtitle and basis label — never tap-only. DA-04's "Cash basis · paid date" micro-label (v1.3) is the canonical pattern.

**R11. Cross-screen back path [P1]** — Cross-screen drill pushes destination as CHILD of source bottom sheet. Destination shows breadcrumb "← From [Source DA name]".
> Engineering note: if no breadcrumb support today, fall back to standard back-stack pop (R2).

### Priority Summary

| Priority | Rules | Engineering guidance |
|----------|-------|---------------------|
| **P0 (must have)** | R1, R2, R4, R5, R7, R8, R10 | Universal mobile expectations |
| **P1 (should have)** | R3, R6, R11 | Significant UX value; depends on existing app router/persistence |
| **P2 (defer if needed)** | R9 | Net-new feature beyond Excel export |

### Per-Spec Specifics (DA-04)

- **Bill gallery** = full-screen modal with swipe between images and pinch-zoom. Close button + swipe-down to dismiss. Returns to expense row in worklist.
- **View Complaint** navigates to Complaints module (leaves Financial Insights — `rentok://complaints/{uuid}`). Permission-gated by complaints module's own permission key.
- **"No bill" governance callout** opens worklist with `bill_urls IS NULL OR bill_urls = '{}'` filter chip pre-applied. Tap on the chip → R8 filter-chip removal behavior.
- **Per-Bed Expense chip** tap → bottom sheet with explanation (per Generation Spec §15 — no inline tooltip, no long-press). No drill.
- **Expense Detail screen** (`/expenses/advanced-details`) = full-screen push. **HB4 prerequisite:** auth check must be added before exposing.
- **EC-02 amber banner CTA "View"** → worklist filtered to `amount < 0`. EC-13 banner CTA → worklist filtered to `paid_date IS NULL`. Both are R8 cases (filter chip pre-applied).
- **Trend chart bar tap** → opens worklist for that month's expenses (period override on destination, source filter chip stays unchanged on parent).

---

### Universal Rule Clarifications (post-orphan-audit)

Resolves interaction-primitive ambiguities surfaced during the post-master orphan-tap-target audit. Treat as additive to R1–R11 above.

**R1 clarification — explicit primitives for common UI:**
- **Hero ⓘ icon:** single-tap → bottom sheet with plain-English explanation + GAAP framing + basis label. **No inline tooltip. No long-press.** Per Generation Spec §15.
- **MoM chip (hero + per-row on Category breakdown):** tap = inline tooltip showing prior-period numbers + computation window. No drill.
- **Accordion section** (Category accordion): tap on row OR chevron toggles expand/collapse. Default-collapsed.
- **Information-only chips** (Per-Bed Expense chip, "Recorded by" badge): single-tap = inline tooltip. No drill unless explicitly listed in Per-Spec Specifics.

**R5 clarification — pull-to-refresh:** Dashboard re-fetches all sections in parallel; worklist re-fetches with current chips. Cross-screen drill destinations (Complaints module, Bill gallery) also support pull-to-refresh. Chevron animates without flashing skeletons.

**R8 clarification — filter chip ✕ as explicit tap target:** Every chip has a discrete ✕ icon (44pt min hit area). Tapping ✕ removes that chip and re-fetches; other chips stay. Body-tap opens edit affordance (date picker, multi-select for Category / Recorded by / Property) where applicable; otherwise no-op.

**R12 (NEW) — Trend chart conventions [P0]:**
- **Bar single-tap:** inline tooltip showing exact expense values + count. No drill.
- **Tap-into-tooltip CTA "View [period] →":** drills to expense worklist filtered to that period.
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
| `viewExpenses` | EXISTS — JWT key | Canonical read-gate (codebase already uses for expenses list service) |
| `view_complaint` | EXISTS — DB column (via `checkAuthInDb`) | For complaint-link drill from expense rows |
| `editExpenses` (cited if any) | MISSING | Reuse `editInvoices` pattern, OR build new |

> **Decision owner: Jatin (Sr Backend).** Each MISSING key requires build-or-reuse decision. Specs cannot ship until Jatin signs off.

---

## Time Filter Behavior

| Component | Behaviour |
|-----------|-----------|
| Hero total | `e.paid_date` in selected period, `e.is_active = 1` |
| Per-bed expense | Total ÷ live occupied beds (not period-scoped) |
| MoM chip | **This Month / Last Month filters:** shows "vs [MonthName]" with percentage. **Custom date ranges:** shows "vs equivalent prior period" with a tooltip clarifying the comparison window. **If prior period total = ₹0:** show "Not enough prior data" (avoid ÷0). |
| Category breakdown | `e.paid_date` in period, grouped by `e.expense_type` |
| Payment mode breakdown | `e.paid_date` in period, grouped by `e.payment_mode` |
| Paid By breakdown | `e.paid_date` in period, grouped by `e.payer` |
| Paid To breakdown | `e.paid_date` in period, grouped by `e.paid_to` |
| Trend chart | Has its own This Month / 6M selector — not governed by time filter |

**Custom future dates:** Shows ₹0 for future portion. Helper text: *"Expenses are based on recorded payment dates. Future dates show ₹0."*

---

## Edge Cases

**EC-01: No expenses in period**
Hero shows ₹0 (0 Expenses). Per-bed shows "—". MoM chip hidden. All accordions show *"No expenses recorded for this period."* Secondary hint: *"Wrong period? Change the date filter."* — prevents "app is broken" interpretation.

**EC-02: Negative-amount expense entry**
Excluded from all totals. If any exist in the period, an amber info banner above the hero: *"[N] refund or correction entries were excluded — they may affect your total."* CTA: "View" → worklist with `amount < 0` filter.

**EC-03: All expenses have no `paid_to` value**
Paid To accordion shows a single row: *"Not specified · ₹82,000"*. No other rows.

**EC-04: Single category dominates (100%)**
Category breakdown shows one row. All other categories hidden (₹0). No visual anomaly — this is valid.

**EC-05: `expense_type` free-text not matching any standard category**
Falls into "Other." If Other > 30% of total, a contextual note: *"A large portion of expenses are not categorised. Tap to see what's in 'Other.'"* CTA → opens worklist filtered to the "Other" category. Do NOT just tell them to improve future recording — let them see what's there now.

**EC-06: Multi-property (Priya)**
All amounts aggregate across selected properties. **"By Property" breakdown appears immediately below the hero block** — before Category. This is Priya's first question when opening the screen. Single-property operators don't see this section at all. Each property row: name, total, share, count. Tap → worklist filtered to that property. (Note: the Worklist Pre-sets table also has a "By Property" row.)

**EC-07: Occupied beds = 0 (vacant property)**
Per-bed shows "—". No divide-by-zero. Tooltip: *"No occupied beds in this period — per-bed figure unavailable."*

**EC-08: Same person appears in both Paid By and Paid To**
Valid — a staff member who recorded an expense AND received a reimbursement for it will appear in both. These are different dimensions and should not be conflated.

**EC-09: Expense linked to a complaint**
Row shows complaint icon. Tap → opens linked complaint. Useful for maintenance expense traceability.

**EC-10: `paid_to` payee appears with multiple spellings**
Each spelling = separate Paid To row. The footnote *"Different spellings of the same vendor appear separately"* is always visible. No auto-merge. Data quality fix is upstream (at the point of entry).

**EC-11: MoM — insufficient prior data or divide-by-zero**
If prior period total = ₹0 (would cause ÷0), or property is < 30 days old, show *"Not enough prior data"* instead of a percentage. Do not show ▲∞% or crash. A ₹0 prior period is more common than "< 3 entries" — this condition is the correct gate.

**EC-12: Expense linked to team member transaction (`tm_transaction_expense_map`)**
No special UI treatment in analytics view. The linkage is used for passbook reconciliation, not surfaced in the insights screen.

**EC-13: `paid_date` is null**
An expense with `paid_date IS NULL` fails to match any date range filter and is silently excluded from all analytics (hero total, all breakdowns, all worklist views). Do not surface it silently. If any `is_active = 1` expense in the selected properties has a null `paid_date`, show an amber info banner: *"[N] expense(s) have no payment date and are excluded from totals."* CTA: "View" → worklist with `paid_date IS NULL` filter. This protects operators from invisible data gaps.

**EC-14: Invoice date differs from paid date**
`e.invoice_date` is stored but is never the governing date. An electricity bill with `invoice_date = April 30` but `paid_date = May 3` appears in May's analytics, not April's. The worklist row shows both dates for operator reconciliation: "Paid: May 3 · Billed: Apr 30". If `invoice_date` is null or equals `paid_date`, show only the paid date.

**EC-15: Wallet-funded expense**
When an expense is funded from the RentOk wallet, `payment_mode = 2048` and `wallet_id` is set. In the Payment Mode breakdown, this appears under "Others / Wallet." No special treatment needed — it's already captured. Engineering note: do not attempt to aggregate wallet vs. non-wallet in Phase 1; surface this as Phase 2.

---

## Words on the Screen

### Empty States

| When | Message | CTA |
|------|---------|-----|
| No expenses in period | "No expenses recorded for this period. Wrong period? Change the date filter." | "Change filter" |
| Custom future range | "Expenses use payment dates. Future dates show ₹0." | — |
| Worklist: filter returns nothing | "No expenses match your current filters." | "Clear Filters" |
| Paid To: all blank | "No payee names recorded for this period." | — |
| No bills in period for "no bill" callout | (callout not shown — no expenses without bills = good, no action needed) | — |

### Error States

| When | Message | Recovery |
|------|---------|----------|
| Network failure | "Couldn't load expenses. Check your connection." | "Retry" |
| Section fails to load | "Couldn't load this section." | "Retry" on that section |

### Hero ⓘ Tooltip
> "Shows all recorded expenses for the selected period. Uses the payment date — when money left. An increase vs last month appears in red (costs are up). Decrease is green (cost control)."

### Per-Bed Expense ⓘ Tooltip
> "Total expenses ÷ currently occupied beds. Helps compare cost efficiency across properties of different sizes. Occupancy count is live — not a period average."

### Category Breakdown ⓘ Tooltip
> "Expenses grouped by type. Based on the category entered when the expense was recorded. Unrecognised types appear under 'Other.'"

### Payment Mode ⓘ Tooltip
> "Shows how money was paid out — cash, digital transfers, cheque, etc. Use this to see how much was paid in cash vs. traceable methods."

### Paid By ⓘ Tooltip
> "Shows who paid each expense — the name entered when the expense was recorded. Two people who entered different spellings of the same name appear as separate rows."

### Paid To ⓘ Tooltip
> "Shows who received the money — vendors, payees, and staff. Names are exactly as entered; different spellings of the same vendor appear as separate rows."

### Paid To Footnote (below list)
> "Payee names are as entered. Different spellings of the same vendor appear as separate rows."

### Loading States
| Component | Loading behaviour |
|-----------|-------------------|
| Hero KPI | Skeleton block matching hero dimensions — number placeholder + MoM chip placeholder |
| Each accordion | Accordion header visible immediately with title; content area shows 3 skeleton rows while loading |
| Trend chart | Chart area shows a grey placeholder rectangle; loads after accordions |
| Overall failure | Hero shows error state; accordions show individual "Couldn't load this section. Retry" |

---

## Critique of the Old PRD

> Treat the old PRD as a useful first draft. Identify what's wrong, over-specced, stale, or should be deferred.

### 1. `expense_date` Field Does Not Exist

Old PRD uses `expense_date` throughout as the governing date field.

**Reality:** The entity has `paid_date` and `invoice_date`. All filter codes in `helpers.ts` use `e.paid_date`. There is no `expense_date` column. Engineering following the old PRD would build against the wrong field.

**Action:** Replace every instance of `expense_date` with `paid_date`. This spec uses `paid_date` throughout.

---

### 2. Expense Approval Queue Is a Dead Feature

Old PRD specs an "Expense Approval Queue" with approve/reject/view actions. It treats `status` as a workflow field (pending/approved/rejected).

**Reality:** The `Expenses` entity has a comment: `//TODO-REMOVE STATUS`. No approval workflow logic exists anywhere in the codebase — no routes, no service methods, no controller logic. The `status` field was never implemented. The base query doesn't even filter on `status` (only `is_active = 1`).

**Action:** Drop the Expense Approval Queue entirely from Phase 1. If an approval workflow is ever built, it requires a full feature spec of its own — it's not an analytics feature.

---

### 3. Category Labels Don't Match the Codebase

Old PRD categories: Operational, Food & Kitchen, Staff, Salary, Maintenance, Marketing, Utilities, Others.

**Reality:** The widget filter codes and ILIKE matching in `helpers.ts` use: Salary, Rent, Electricity, Mess/Food, Maintenance, Deposit, Other. "Operational," "Marketing," "Utilities," and "Staff" are not code-defined categories — they'd all fall into "Other."

**Action:** Use the code-defined category set. Any expense not matching the 6 standard types lands in "Other." The old PRD's categories are aspirational taxonomy, not what the system actually classifies.

---

### 4. Per-Bed Expense Needs a Denominator Decision

Old PRD specifies "Current Occupied Beds" as denominator but doesn't clarify: at period start, period end, or live count?

**Decision:** Use **live occupied beds** (current count at time of API call). Rationale: simpler to query, and operators check this screen in the current context ("how am I doing right now?"). A period-average occupancy would require a daily snapshot history that doesn't exist.

**Engineering note:** This requires a cross-query to occupancy data. It's a new build.

---

### 5. "Paid By" Field Mapping Is Wrong (Partially)

Old PRD uses `paid_by = user_id` as the filter field.

**Reality:** `applyPaidBy()` in `helpers.ts` filters on `e.payer`. `payer` is a **free-text name string** entered at recording time (e.g., "Jatin") — it is NOT a firebase_id and requires NO external lookup. `firebase_id` is a separate column storing the Firebase Auth UID. `added_by` is the role enum (0–5).

**Action:** "Paid By" breakdown groups by `e.payer` (already a name — use directly). Show `added_by` as role badge. Don't group by `added_by` — that would show "Owner (₹30K)" vs "Manager (₹52K)" which loses individual accountability. Don't attempt firebase_id resolution — it's the wrong column.

---

### 6. Reconciliation Invariant Is Partially Wrong

Old PRD: "Category sum = Payment Mode sum = Paid By sum = Paid To sum = Total Expense."

**Issue:** Paid To can have empty values (expenses with no `paid_to`). If the worklist shows those as "Not specified," the Paid To sum still = Total Expense. But if the Paid To breakdown hides blank entries, the invariant breaks.

**Action:** "Not specified" is always a row in Paid To breakdown (at the bottom) so the invariant holds. Paid To sum = Total Expense always.

---

## Decisions That Override Old PRD

| Old PRD | What We're Building | Why |
|---------|--------------------|----|
| `expense_date` governs | `paid_date` governs | That's the actual DB field |
| Expense Approval Queue (P1) | Dropped entirely | `status` field is TODO-REMOVE; no workflow exists |
| Categories: Operational/Marketing/Utilities/Staff | Categories: Salary/Electricity/Mess-Food/Maintenance/Rent/Deposit/Other | Aligned to ILIKE prefixes in `helpers.ts notOtherTypes` |
| `paid_by = user_id` | Group by `e.payer` (already a name string) + show `added_by` as role badge | `payer` is free-text name, not a user ID |
| Paid To hides blank entries | "Not specified" row always shown | Maintains reconciliation invariant |
| MoM uses `expense_date` | MoM uses `paid_date` | Consistent with governing date rule |
| `payer` requires firebase_id → name lookup | No lookup needed — `payer` is already the name | Confirmed by reading `applyPaidBy()` and formatted row |
| Payment Mode: Cash / UPI / Bank / Cheque / Others | 9 distinct modes: 202+2040=Cash, 2041=G Pay, 2042=Phone Pe, 2043=Paytm, 2044=UPI, 2045=Bank, 2046=Card, 2047=Cheque, 2048=Others/Wallet | Integer mapping from `src/helpers/payments.ts:822` |
| Export to Excel = inline file download | Export triggers email of `.xlsx` to operator's registered address | Existing endpoint `POST /generateExpenseReport` is email-based |
| Category worklist filter uses `applyCategories()` | Category drill-down must use `ILIKE '[category]%'` | `applyCategories()` uses SQL `IN` — misses suffixed types like "Salary - April" |
| Payment mode display: 9 granular modes | Grouped for display: Cash · UPI/Digital · Bank · Card · Cheque · Others/Wallet | PG operators don't need G Pay vs PhonePe split in analytics |
| MoM chip for partial month: shows full prior month | Compare same elapsed days — "vs Apr 1–5" | Avoids misleading comparison of 5 days vs 30 days |
| Trend chart default: This Month | Default is 6M | Single stub bar on May 5th is meaningless; 6M gives immediate trend context |
| By Property breakdown placement: between Paid To and Trend | Appears immediately after hero (before Category) for multi-property | Priya's first question is "which property?", not "which category?" |
| Cash governance nudge in P1 | Deferred to Phase 2 | PG operators pay cash by default — nudge fires for everyone, adds noise |
| Concentration flag in P1 | Deferred to Phase 2 | Single-manager PGs trigger this constantly; needs governance module context |

---

## v1.0 → v1.1 → v1.2 → v1.3 Calibrations (post 2 operator audit rounds)

### v1.2 → v1.3 changes (second-round operator audit, 2026-05-07)

| v1.2 (first audit) | v1.3 (second-round corrections) | Why |
|---|---|---|
| Per-bed Expense always shown in hero | Per-bed Expense rendered ONLY when multi-property is selected | Single-property operator has no comparison group — ₹820/bed with no benchmark gets ignored. Inherits DA-07's discipline. |
| Category-level MoM delta with no small-N suppression | Suppression rule: hide chip when current/prior count < 3 OR amount delta < ₹2,000 | Without suppression, 1-expense-vs-1-expense categories render misleading ▲500% chips. Matches DA-05 Reason discipline. |
| "No bill" callout fires on any N ≥ 1 | Floor of ≥ 3 missing bills + special copy at 100%-missing | Single missing bill in 50-expense month was alarm-fatigue noise. ≥3 floor focuses on real coverage gaps. |
| EC-02 + EC-13 + "no bill" callout could all stack | Amber banner stack rule: priority order EC-13 > EC-02 > no-bill; cap at one above hero | Multiple amber banners = visual noise; lower-priority moves to worklist filter chips. |
| Pre-Launch Engineering Blockers section absent | Section added (see below) | Engineering needs explicit launch gates; structural parity with DA-06/DA-07. |
| Persona narratives: only "A Morning with Rajesh" | Added "A Morning with Priya" + "A Morning with the CA via WhatsApp" | DA-07 has 3 personas; DA-04 had 1. Priya's multi-property triage and CA's screenshot use case were unspecified. |

### v1.1 → v1.2 changes (first-round audit)

| Item | Change |
|---|---|
| Hero MoM chip | Added with prior-period absolute ("▲18% vs ₹69,500 Apr") |
| Partial-month MoM rule | Added same-elapsed-days comparison |
| "No bill" governance callout | Added below MoM chip |
| Per-Bed Expense | Added to hero (later cut for single-property in v1.3) |
| Category-level MoM delta | Added per row (later got small-N suppression in v1.3) |

---

## Pre-Launch Engineering Blockers

These items must be resolved before DA-04 ships. Not Phase 2 — Phase 1 launch dependencies. Structured per DA-07 v1.2 pattern.

### HARD blockers (mobile screen would be wrong)

| Item | Why hard-block | File:Line |
|------|----------------|-----------|
| **HB1. Category drill-down must use ILIKE, not `applyCategories()` IN** | `applyCategories()` uses SQL `IN` (exact match). Worklist drill-down would mis-count any `expense_type` with suffix (e.g., "Salary - Apr"). Architecture Note 10 already documents; promote to launch gate. | `helpers.ts` `applyCategories()` |
| **HB2. `e.paid_date IS NULL` exclusion banner (EC-13)** | Silent data loss on a financial screen. Operators recording expenses without paid_date will never see them in analytics. EC-13 specifies the banner; engineering must implement. | (query layer) |
| **HB3. Cash flow expense+refund double-count fix** | DA-07 hero math depends on DA-04's `expense_type ILIKE 'Deposit%'` deduction. Without it, DA-04 ↔ DA-07 reconciliation invariants fail. **Note: this is DIFFERENT from DA-05's "Refund→Credit-status integrity bug" — that's at `services/refunds/refunds.ts`. This one is at the cash flow aggregation line.** | `generateCashFlowReport.ts:393` (cross-screen) |
| **HB4. `/expenses/advanced-details` endpoint has NO auth check** | Expense Detail screen (`getExpenseDetails`) lacks permission validation. Any authenticated user can read any expense. **Add `checkAuth(viewExpenses)`.** List endpoint `/v1/expenses/list/filters` already has auth (added Apr 2026 master); detail endpoint was missed. | `src/services/expense/expense.ts:20` |
| **HB5 (NEW Phase 4 audit, 2026-05-08). `is_capex` column does NOT exist on `Expenses` entity** | DA-07 §4 Section A Footer Callouts ("Capex flag") and §9 Bottom Sheets row 10 reference `is_capex=1` filter for cross-screen drill into DA-04. Per Field Map §1.7 (verified via grep across all entity files, services, list_screens), the column does NOT exist anywhere in the codebase. **NEW BUILD requirement:** add `is_capex: boolean` column to `Expenses` entity + define classification heuristic (manual flag during expense creation OR threshold-based auto-detection > ₹X). **⚠️ Cross-spec impact:** blocks DA-07 §4 capex callout drill into DA-04 worklist. Without this, DA-07 capex flag cannot resolve. | `src/entities/expenses.ts` (column build) + DA-04 worklist filter + DA-07 cross-screen drill — coordinated work |

### PARITY blockers (mobile + Excel must match)

| Item | Why parity-block | File:Line |
|------|------------------|-----------|
| **PB1. Excel export categories must use ILIKE bucketing** | Existing Excel uses free-text `expense_type` strings as column keys. Mobile uses ILIKE buckets. Same fix as DA-07 PB1. | `generateCashFlowReport.ts:374-378` |

### DEPENDENCY (owned by another spec; DA-04 ships when this lands)

| Item | Owner |
|------|-------|
| **`partner_phone` strip-at-API-layer behaviour** | Confirm if Call action ships in Phase 1 (it's currently dropped — partner fields stripped in `formatExpenseRow()`) | API spec |

### DEFERRABLE (post-launch hotfix safe)

| Item | Why deferrable |
|------|----------------|
| **D1. `payer` typo de-duplication** | Phase 2 Payer Name Normalisation already covers; impact is row-count only, not amount accuracy. |
| **D2. `bill_urls` falsy-entry filtering** | Edge case (`['']` single empty string). Hotfix safe. |
| **D3. Trend chart partial-month tooltip** | Visual polish; not data-correctness. |

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
| **Expense approval workflow** | Requires building from scratch — `status` field was never implemented. Full feature, not an analytics add-on. |
| **Category auto-normalisation** | Merging "MH Electricity Board" + "Maharashtra Electricity Board" → requires NLP or a canonical vendor registry. |
| **Payer name normalisation** | Same free-text problem as `paid_to`. "Jatin" and "jatin" as separate rows. Requires canonical staff registry or fuzzy merge. |
| **YTD / FY comparison (financial year)** | Current YTD = calendar year (Jan–Dec). A financial-year YTD (Apr–Mar) requires a separate date range parameter. Requires 2+ FY of data for "vs last FY." |
| **Complaint-expense spend analysis** | "How much did I spend on maintenance linked to complaints?" — aggregate across `complaint_expense_map`. |
| **Wallet-funded expense analytics** | `getExpenseDetails` computes `payment_source_breakup` (Admin Fund / Partner Fund / Other). Surfacing this at aggregate level requires new GROUP BY on `wallet_id` presence. |
| **Budget vs actuals** | Expense tracking vs a set budget ceiling per category. Requires a budget-setting feature first. |
| **Staff salary breakdown** | Individual staff salary entries vs total salary spend. Requires staff module integration. |
| **Partner expense detail** | `partner_name`, `partner_id`, `partner_phone` are stored in entity but stripped from the v1 list API response (`formatExpenseRow()`). Surfacing partner metadata in the worklist row requires an API update. |
| **Per-bed denominator improvement** | Live occupied beds is epistemically wrong for a historical analytics screen — the denominator changes with every check-in/checkout. Period-average occupancy requires a daily snapshot history that doesn't exist yet. |
| **Cash governance nudge** | ">50% cash → suggest digital payments." Removed from P1 — PG operators pay cash for everything by default. Nudge fires every month and adds noise. Validate with operators before building. |
| **Concentration flag (Paid By)** | "One person recorded >60% of cash expenses." Removed from P1 — in single-manager PGs this fires for everyone. Build in a dedicated Governance/Audit module where context (staff size, property count) is available. |
| **Expense-to-collection ratio** | "You spent 58% of what you collected." Cross-screen metric — belongs on homescreen Overview, not Expenses tab. Requires joining Collections data. |
| **"Review staff-recorded expenses" quick filter** | One-tap filter to non-owner entries (`added_by != 0`). Useful for owner oversight. Low effort but secondary to core analytics. |

---

## For Engineering

### Reconciliation Invariants

| Invariant | Formula |
|-----------|---------|
| Category sum = hero total | Salary + Electricity + Mess/Food + Maintenance + Rent + Deposit + Other = Total |
| Payment mode sum = hero total | All modes sum = Total |
| Paid By sum = hero total | All payers sum = Total |
| Paid To sum = hero total | All payees (incl. "Not specified") sum = Total |
| Worklist count = parent count | Header count → worklist row count must match exactly |

### Codebase Feasibility

> **Source:** `src/v1/list_screens/expenses/helpers.ts`, `src/v1/list_screens/expenses/service.ts`, `src/entities/expenses.ts`
> **Reviewed:** 2026-05-06

#### What Already Exists

| PRD Feature | Code Reference | Status |
|-------------|---------------|--------|
| Base query: active expenses | `buildBaseQuery()` — `e.is_active = 1` | **EXISTS** |
| Filter by `paid_date` range | `applyDateRange()` | **EXISTS** |
| Filter by category (`expense_type`) | `applyCategories()` — exact match `IN` array (for worklist manual filters) | **EXISTS** (note: analytics aggregation must use ILIKE, not this function — see Architecture Note 10) |
| Filter by payment mode | (via `applyFilterCodes`) | **EXISTS** |
| Filter by paid by (`payer`) | `applyPaidBy()` — filters on `e.payer` | **EXISTS** |
| Filter by paid to | `applyPaidTo()` — filters on `e.paid_to` | **EXISTS** |
| Search (expense_type, description, paid_to) | `applySearch()` | **EXISTS** |
| Sort by paid_date, amount | `applyOrderBy()` | **EXISTS** |
| Pagination | `executePaginated()` — real pagination | **EXISTS** |
| Multi-property | `pg_number_filter` param | **EXISTS** |
| 10 widget filter codes | `getWidgetFilterCodes()` | **EXISTS** |
| Category widget amounts | `getWidgetItemForCode()` per category code | **EXISTS** |
| Complaint linkage | `complaint_expense_map` entity + join available | **EXISTS** (not surfaced in list yet) |
| Bill URLs | `e.bill_urls` array in base query | **EXISTS** |
| Partner details | `e.partner_name`, `e.partner_id`, `e.partner_phone` | **EXISTS in entity** (but stripped from `formatExpenseRow()` — not in v1 list API response) |
| Excel export | `POST /generateExpenseReport` — email-based `.xlsx` | **EXISTS** (`src/controllers/reports.ts:2111`) |
| `added_by` role | Selected in base query | **EXISTS** |

#### What Needs New Build

| PRD Feature | Gap | Effort |
|-------------|-----|--------|
| **Category aggregation (GROUP BY)** | `applyCategories` filters exist but no `GROUP BY e.expense_type` query for the breakdown. | **LOW** — straightforward aggregation |
| **Payment mode aggregation** | No `GROUP BY e.payment_mode` query. | **LOW** |
| **Paid By aggregation** | No `GROUP BY e.payer` query. `payer` is already a free-text name — no external lookup needed. Same shape as Paid To aggregation. | **LOW** — straightforward GROUP BY; no name resolution required |
| **Paid To aggregation** | No `GROUP BY e.paid_to` query. | **LOW** |
| **MoM comparison** | No prior-period query. Requires: same query for prior equivalent period, compute delta. | **MEDIUM** |
| **Per-bed expense** | No occupancy cross-query. Needs: `SUM(amount)` for period ÷ live occupied beds from tenants table (`status=1 AND property_id`). | **MEDIUM** |
| **Expense trend chart** | No time-series aggregation. `GROUP BY DATE_TRUNC('month', e.paid_date)` with category stacking. | **MEDIUM** |
| **"Not specified" Paid To row** | Current `applyPaidTo` filters on exact match. Need: `WHERE e.paid_to IS NULL OR e.paid_to = ''` for the blank group. | **LOW** |
| **Complaint icon in worklist row** | Base query doesn't join `complaint_expense_map`. Add LEFT JOIN + boolean flag. | **LOW** |
| **Concentration flag (Paid By)** | Compute `SUM(amount) WHERE e.payment_mode = cash GROUP BY e.payer`, check if any > 60%. Derivative of Paid By aggregation. | **LOW** |
| **"Other" category catch-all** | `CURRENT_MONTH_OTHER` filter code exists, but needs generalisation to any period (not just current month) for the detail screen's custom date ranges. | **LOW** |

#### Key Architecture Notes for Engineering

1. **`paid_date` is the single governing date.** `invoice_date` exists in entity and base query select but is never used for filtering. Do not use it for analytics date logic. It may be displayed in the worklist row as a secondary "Billed:" date when it differs from `paid_date`.
2. **`status` field — do not use.** The entity comment is `//TODO-REMOVE STATUS`. The base query does not filter on it. No approval workflow exists. Ignore `status` entirely; do not render it in any UI component.
3. **`added_by` vs `payer`:** `added_by` = role enum (0–5, see table). `payer` = **free-text name string** (e.g., "Jatin") entered at recording time — NOT a firebase_id. GROUP BY `payer` gives name-based rows directly. `firebase_id` (a separate column) stores the Firebase Auth UID but is not the Paid By grouping key. Don't confuse the three columns.
4. **`paid_to` is raw free text.** No FK, no normalisation. GROUP BY `paid_to` creates separate rows for spelling variants. Expected behaviour — document it in UI, don't fix at DB level.
5. **Category ILIKE matching — 7 prefixes.** `notOtherTypes` array: `['Salary', 'Rent', 'Electricity', 'Mess', 'Food', 'Maintenance', 'Deposit']`. "Other" = NOT ILIKE any of these with `%` appended. Mess and Food are stored as separate ILIKE checks but displayed as one combined row ("Mess / Food").
6. **filter_codes are mutually exclusive with manual filters.** When `filter_codes` is set, `start_date`, `end_date`, `categories`, `paid_by`, `paid_to_filter` are all discarded. CTA drill-downs from the analytics screen must use manual filters, NOT filter_codes — or the date range will lock to current month.
7. **Per-bed denominator:** `COUNT(*)` from `tenant` where `status = 1` (active) AND `property_id IN (selected properties)`. Live count at API call time.
8. **`payer` is free-text, same quality caveat as `paid_to`.** Spelling variants produce separate Paid By rows. No server-side merging. Same footnote pattern as Paid To: "Names are as entered."
9. **Payment mode integer values.** `payment_mode` stores integers — no string enum in the entity. Full mapping: 202/2040=Cash, 2041=G Pay, 2042=Phone Pe, 2043=Paytm, 2044=UPI, 2045=Bank Transfer, 2046=Card Machine, 2047=Cheque, 2048=Others/Wallet. Source: `src/helpers/payments.ts:822`. Cash governance check = `payment_mode IN (202, 2040)`.
10. **Category drill-down worklist must use ILIKE, not `applyCategories()`.** `applyCategories()` uses SQL `IN` (exact match). An expense with `expense_type = "Salary - April"` matches `ILIKE 'Salary%'` but NOT `IN ('Salary')`. If the worklist drill-down reuses `applyCategories()`, counts will be wrong for any suffixed type. Build a new ILIKE-based filter for the drill-down path.
11. **Export is already built — email-based.** `POST /generateExpenseReport` at `src/controllers/reports.ts:2111` generates an `.xlsx` and emails it. Do NOT build an inline download. Wire the "Export to Excel" button to this existing endpoint.
12. **`ACTIVE_CATEGORIES` (widget code 1302) returns COUNT DISTINCT, not SUM.** Every other widget code returns `SUM(e.amount)`. Code 1302 returns `COUNT(DISTINCT e.expense_type)`. Any UI rendering widget items uniformly as "₹X" will display an incorrect value for this widget. Branch on widget code type.
13. **YEAR_TO_DATE (widget code 1310) = calendar year.** Defined as `paid_date >= Jan 1 of current year AND paid_date < Jan 1 of next year`. This is NOT a rolling 12 months and NOT an Indian financial year (Apr–Mar). For an operator checking in March, YTD covers only Jan–Mar. Consider adding a tooltip: "Jan 1 – today."
