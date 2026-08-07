---
title: DA-07 Build Sheet
date: '2026-05-08'
tags:
  - rentok
  - prd
  - build-sheet
  - engineering
  - homescreen
  - financial-insights
  - detailed-analytics
  - cashflow
aliases:
  - DA-07 Build Sheet
  - DA-07 Dev Tasks
status: V1 — locked
version: '1.0'
for: Engineering ticket creation
companion_to: DA-07 Cash Flow Detailed Analytics
---

> [!INFO] Source of Truth
> **Companion to:** [[DA-07 Cash Flow Detailed Analytics]] — canonical "why" doc (V1.3)
> **Reference:** [[_Ground Truth Field Map]] — entity fields, formulas, endpoints, permissions
> **Format spec:** [[_Build Sheet Generation Spec]] — column conventions, gloss rules, status values
> **For:** Engineers picking up tickets

# DA-07 — Build Sheet (for Engineering) — V1

Each row in a section table = one thing to build. Status column tells you whether to extend, build, compose, or wire UI. Plain columns are for PM / CS / QA review; Tech columns are for implementation.

For "why" or rationale, see [[DA-07 Cash Flow Detailed Analytics]].
For entity fields, formulas, permissions, endpoints, see [[_Ground Truth Field Map]].
For format conventions, see [[_Build Sheet Generation Spec]].

---

## Glossary

Plain-English definitions of terms used across this Build Sheet. Read before scanning rows.

| Term | Plain English |
|------|---------------|
| **paise** | Indian smallest currency unit. ₹1 = 100 paise. Always store amounts as integer paise, never as floats |
| **pg_id** | Property identifier — every record (invoice, payment, expense, refund) belongs to one property identified by `PG<n>` |
| **Operating Cash Flow** | What the operator actually made or lost — period inflows (rent, utilities, advance) minus period outflows (salary, expenses, non-deposit refunds). The hero number |
| **Deposit Money In/Out (Section B)** | Non-revenue cash — deposits collected and deposit refunds paid. Tracked separately because deposits are a liability the operator must return. NOT in the hero |
| **Net Cash Flow** | Inflows − Outflows for the selected period. Can be negative. Sign-coded green (positive) / red (negative) / neutral (zero) |
| **GAAP basis** | "Net Operating Cash Flow" subtitle on hero — accountant-readable label so a CA recognizes the number on a forwarded screenshot |
| **Mode 211 / 288** | Payment-mode integers for **deposit-applied** (211) and **advance-applied** (288). Paper transfers — no cash actually moved. EXCLUDED from all DA-07 sums per Field Map §2.1 |
| **Period-sensitive** | Section respects the dashboard date filter — number changes when user picks "Last Month" / "Custom range". DA-07 hero is period-sensitive (matches DA-02/03/04/05) |
| **Cash basis** | Inflows recognised on `payment.paid_date`; outflows on `expense.paid_date` and `refund.refund_date`. NOT on invoice date |
| **MoM chip (3 states)** | Improvement (▲ green: same-sign growth OR sign-reversal-to-positive) · Regression (▼ amber: same-sign decline > 10%) · Sign-reversal-to-negative (▼ red: was positive, now negative). v1.2 collapsed from 4 states |
| **Per-bed Cash Flow** | Hero ÷ live occupied beds. Multi-property only. Live count, not period-average — matches DA-04 pattern |
| **Mid-month timing nudge** | Conditional copy when (current day < 25) AND ("This Month" preset selected) AND (period contains today). Saves operator from mid-month panic |
| **Deposit-dependency callout** | Conditional warning when (deposit inflow ≥ 30% of total inflow) AND (≥ 5 active deposit-holders) AND (property age ≥ 60 days) |
| **Vendor concentration flag** | Conditional warning when (top vendor by `paid_to` > 30% of outflow) AND (≥ 5 expense entries) AND (top vendor amount ≥ ₹15K) |
| **Capex flag** | Conditional warning when ("Other" expenses > 25% of outflow) AND (any single expense > ₹50K). DA-04 has `is_capex` flag for explicit drill |
| **YTD strip** | Financial-year-to-date cumulative. Indian FY = April 1 – March 31. Auto-rolls based on current date |
| **Discoverability footer** | Static informational text at bottom of screen — "Need a detailed report for accounting? Email Cash Flow Excel from menu (⋮)." Visible in screenshots so CA reading forwarded screenshot knows Excel exists |
| `tenant.status` | DA-07 doesn't directly group by tenant.status (cash flow is composition-based). For any per-tenant drill into DA-02/DA-04/DA-06, full 9-value enum (0-8) at [[_Ground Truth Field Map]] §2.3 + wiki TEN-001 |

---

## Engineering Architecture Reference

| Concern | Path / Reference |
|---------|------------------|
| List endpoint | `POST /v1/list_screens/cashflow/list/filters` — **NEW BUILD** (mirror `src/v1/list_screens/expenses/`) |
| Widget / aggregator | **NEW BUILD** at `src/v1/list_screens/cashflow/service.ts` — composes from collections + expenses + refunds queries |
| Filter codes | `src/v1/constants/filterCodes.ts` — `CashFlowFilterCode` enum, range **1800-1899 (NEW BUILD)** |
| Helpers (query builder) | **NEW BUILD** at `src/v1/list_screens/cashflow/helpers.ts` |
| Excel report endpoint (existing) | `POST /reports/cashflow-report` — `src/routes/reports.ts:119` (HeaderValidator-protected) |
| Excel service (existing) | `src/services/reports/generateCashFlowReport.ts` — Phase 1 = column extension + B1/B2 bug fixes |
| Cross-screen drill: Collections | `POST /v1/list_screens/collections/list/filters` — `src/v1/list_screens/collections/routes.ts:8` |
| Cross-screen drill: Expenses | `POST /v1/list_screens/expenses/list/filters` — `src/v1/list_screens/expenses/routes.ts:8` |
| Cross-screen drill: Refunds | `POST /v1/list_screens/refunds/list/filters` — **NEW BUILD per DA-03** |
| Cross-screen drill: DA-06 Liabilities | **NEW BUILD per DA-06** with "viewing as of [period end]" pill |
| DA-04 worklist (vendor flag, capex flag) | `POST /v1/list_screens/expenses/list/filters` with `paid_to` / `is_capex` filters |
| DA-05 worklist (discount memo) | **NEW BUILD per DA-05** |
| Tenant entity | `src/entities/tenant.ts` — see Field Map §1.5 |
| Invoices entity | `src/entities/invoices.ts` — see Field Map §1.1 |
| Payments entity | `src/entities/payments.ts` — see Field Map §1.2 |
| Refunds entity | `src/entities/refunds.ts` — see Field Map §1.3 |
| Expenses entity | `src/entities/expenses.ts` — see Field Map §1.7 |
| Permission keys | Field Map §3 — `viewCashFlow` does NOT exist; recommend composite `viewInvoices AND viewExpenses` (Jatin gate) |
| DB Excel-export toggle | `cashflow_report` (DB column on `team_member_property` per Field Map §3.2) — gates Excel CTA only, NOT screen access |
| Migrations | TBD (filter code enum addition; possible new permission column for screen access) |

---

## Cross-Suite Blockers Affecting This Spec

Surfaced from `[[DA-07 Cash Flow Detailed Analytics#Cross-Suite Engineering Blockers]]`.

| ID | Issue (one-line) | Status | Affects |
|----|------------------|--------|---------|
| ⛔ CSB-1 | `checkAuth` returns true on missing Bearer header — list endpoint reachable anonymously | Jatin gate (P0) | All sections (any auth-protected endpoint) |
| CSB-3 | Cross-screen drill destinations (`/tenant/getTenantData`, `/invoices/getInvoiceData`) zero middleware | Jatin gate (P0) | Bottom-sheet sub-row drills (Top-5 tenant entity rows) |
| CSB-4 | `/refunds/advanced-details` and `/expenses/advanced-details` zero `checkAuth` — anonymous read by uuid | Jatin gate | Bottom-sheet sub-row drills (refund / expense detail) |
| CSB-5 | Expired-JWT bypass — stolen Manager App token authenticates indefinitely | App-hygiene fast-follow | All authenticated rows |
| CSB-6 | JWT permission-key shape diverges (11 vs 12 keys) | App-hygiene | Permission gating on screen entry |
| ⛔ CSB-8 (NEW 2026-05-11) | **Widget endpoint permission enforcement gap (security).** Pattern from `getDuesWidget` (`dues/service.ts:132-138`) — set `self_added_team_uuid` but never apply. DA-07 NEW BUILD cash-flow widget endpoints MUST enforce self-added filter from day 1 | Engineering (build correctly) | All NEW cash flow widget endpoints |

---

## Hard Blockers (HB)

Surfaced from PRD's Pre-Launch Engineering Blockers.

| ID | Issue | Status |
|----|-------|--------|
| HB-B1 | **Production double-count bug at `generateCashFlowReport.ts:393`** — `totalExpenses = expenseTotal + refunds`. When operator records a deposit return as both Refund row AND `expense_type ILIKE 'Deposit%'` Expense, same money counted twice. Pre-launch fix mandatory (excludes `'Deposit%'` from operating outflows; dedupes Refund/Expense in Section B per DA-03 EC-15 heuristic) | Engineering (blocks Section B) |
| HB-B2 | Existing Excel column shape doesn't match DA-07 sections + 14 bottom sheets — extend, don't replace | Engineering (Phase 1) |
| HB-P1 | `viewCashFlow` permission key does NOT exist as JWT key. DB has `cashflow_report` (Excel toggle, not screen perm). Build new column OR composite gate `viewInvoices AND viewExpenses` | Jatin gate (blocks all sections) |
| HB-F1 | `CashFlowFilterCode` enum does NOT exist — range 1800-1899 free, must build | Engineering (blocks list endpoint) |
| HB-E1 | `/v1/list_screens/cashflow/list/filters` does NOT exist — NEW BUILD mirroring expenses pattern | Engineering (blocks Section A/B drill targets that need cashflow-scoped worklist; NOTE — most drills route to source DA worklists, so this is for hero/property bottom sheet) |
| HB-D1 | DA-06 cross-screen drill ("View deposit balance →") needs "viewing as of [period end]" pill — DA-06 is live-snapshot, DA-07 is period-based. Requires DA-06 to support historical mode OR fall back to live with explanation | Cross-suite (blocks Section B CTA) |

> **Back-port to PRD:** HB-P1 and HB-E1 are Build-Sheet findings — surface in [[DA-07 Cash Flow Detailed Analytics#Pre-Launch Engineering Blockers]].

---

## Pre-Build Decisions Pending (Jatin)

Must be answered before Phase 1 begins:

| Decision | Recommendation | Owner |
|----------|----------------|-------|
| HB-P1 — `viewCashFlow` build new column vs composite `viewInvoices AND viewExpenses`? | Composite gate (cash flow = invoices ± expenses, no new column needed). Excel CTA gated separately on `cashflow_report` DB column | Jatin |
| HB-B1 — fix double-count at `generateCashFlowReport.ts:393` | Mandatory pre-launch. Exclude `expense_type ILIKE 'Deposit%'` from operating outflows; route Deposit% expenses to Section B with EC-01 dedupe heuristic | Jatin |
| HB-E1 — `/v1/list_screens/cashflow/list/filters` build new vs reuse | New aggregator at `src/v1/list_screens/cashflow/`. Composes from collections + expenses + refunds query builders. Mirror expenses pattern | Jatin |
| HB-D1 — DA-06 historical mode for "View deposit balance →" | If DA-06 v1 supports `as_of_date` param, render historical. Else fall back to live with explanation pill | Jatin |
| ~~MoM chip prior-period method~~ | **RESOLVED** per [[_Build Sheet Generation Spec]] §14: "same-elapsed-days" locked globally | — |
| Vendor concentration query — top-N by `paid_to` aggregation per period | Add to expenses helpers; same query DA-04 already runs (Field Map §4.4) | Jatin |
| Capex flag — does Expenses entity have `is_capex` column today? | **[VERIFY]** PRD references `is_capex=1` filter for DA-04 drill but Field Map §1.7 does not document an `is_capex` column on Expenses. Either build it or use heuristic (Other category > ₹50K threshold) | Jatin |

---

## 1. Hero — Net for the Month

> **Endpoint:** BUILD — new aggregator at `src/v1/list_screens/cashflow/service.ts`. Composes period-scoped sums from collections (`p.paid_date`), expenses (`e.paid_date`), refunds (`refund.refund_date`)
> **Default property context:** inherited from dashboard (single → single; multi → "across N properties")
> **Default time context:** **period-sensitive** — Hero number changes with date filter (matches DA-02/03/04/05; NOT live-snapshot like DA-06)
> **Cross-Suite Blockers:** CSB-1 (HeaderValidator bypass), CSB-3 (drill destinations unauth)
> **Hard Blockers:** HB-P1 (permission key), HB-E1 (list endpoint), HB-B1 (double-count)
> **Eng note:** all amounts in **paise** (integer). Display via `fmt()` helper. Date basis: inflows `p.paid_date`, expense outflows `e.paid_date`, refund outflows `refund.refund_date`. Mode-211 (deposit-applied) and Mode-288 (advance-applied) **EXCLUDED** from all sums per Field Map §2.1 (paper transfers, no cash moved). Operating Outflows EXCLUDES `expense_type ILIKE 'Deposit%'` per HB-B1.
> **⚠️ PRD vs codebase gap:** PRD cites `viewCashFlow` permission — does NOT exist. Recommend composite `viewInvoices AND viewExpenses`. Jatin gate (HB-P1).
> **Filter codes:** `CashFlowFilterCode` 1800-1899 NOT BUILT (HB-F1).

| Component | Status | Task | Definition + [class] | Plain Formula | Formula (technical) | Permission | Plain Drill | Drill Behavior (technical) |
|-----------|--------|------|----------------------|---------------|---------------------|------------|-------------|----------------------------|
| Hero | BUILD + [VERIFY] | Net Cash Flow number | The big sign-coded number — period inflows minus period outflows. [period-sensitive] | The amount of money the operator actually made or lost during the selected period. Inflows (rent + utility bills + advance received) minus outflows (salary + expenses + non-deposit refunds). Deposits collected/refunded are NOT in this number — they go in Section B. **[VERIFY]** confirm `viewCashFlow` permission resolution per HB-P1. | `Operating_Inflows − Operating_Outflows` where `Operating_Inflows = SUM(p.net_amount - p.gateway_charges) WHERE p.status = 1 AND p.is_active = 1 AND p.paid_date BETWEEN [period] AND p.payment_mode NOT IN (211, 288) AND inv.due_type NOT IN ('Security Deposit', 'Caution Money')` *(`p` = `payments` table; `status=1` = success per entity comment 0/1/2/3 = failed/success/pending/refunded; `is_active=1` = active row; mode 211 = deposit-applied EXCLUDED, mode 288 = advance-applied EXCLUDED per Field Map §2.1; `inv` = `invoices` joined many-to-many via `payments_invoices`; deposit due_types EXCLUDED routed to Section B)* and `Operating_Outflows = SUM(e.amount) WHERE e.is_active = 1 AND e.paid_date BETWEEN [period] AND e.expense_type NOT ILIKE 'Deposit%' + SUM(r.amount) WHERE r.refund_date BETWEEN [period] AND r.invoice.due_type NOT IN ('Security Deposit', 'Caution Money')` *(`e` = `expenses` table; `r` = `refunds` table — refunds have NO `is_active` and NO `status` columns per Field Map §1.3, every row is final; `'Deposit%'` exclusion is HB-B1 fix)*. **NEW BUILD** at `src/v1/list_screens/cashflow/helpers.ts`. | view: composite `can_view_invoices AND can_view_expenses` (Jatin gate per HB-P1) | → Live (today): Tap the hero block to open the detail bottom sheet showing the full Section A + Section B breakdown in expanded form. Long-press the number for a tooltip explaining "Net Operating Cash Flow" and the Excel report.<br>→ Past period: Same as live — the hero updates to the period total, and tap behavior is identical. Bottom sheet shows the period's breakdown.<br>→ Future: Same as live — for custom ranges extending into the future, future portion shows ₹0 contribution (helper text clarifies). | → Live: BS (Net Cash Flow detail) on tap · T (long-press) on number · time: period inherited · property: inherited · "Net for the Month — [Period Total]"<br>→ Past period: SAME with period inherited<br>→ Future: SAME (future portion contributes ₹0) |
| Hero | BUILD | Sign-coded color logic | Positive = green; negative = red; zero = neutral. [period-sensitive UI render] | Show the number in green if positive (operator made money), red if negative (operator lost money), or neutral grey if exactly zero. Opposite of DA-04 Expenses where up = bad. | `if net > 0: green; elif net < 0: red; else: neutral` (frontend render) | view: inherits hero visibility | Display only — color encodes sign. | n/a — UI render only |
| Hero | BUILD | "Net for the Month" headline | Static visible primary label. [always-live label text] | The operator-readable label above the big number. Does not change with period. | Static string `"Net for the Month"` (frontend) | view: inherits hero visibility | Display only — label text. | n/a — display only |
| Hero | BUILD | "Net Operating Cash Flow" GAAP subtitle | Static visible secondary subtitle (line 1, small grey). [always-live label text] | A small accountant-friendly subtitle so a CA viewing a forwarded screenshot recognizes the number as cash-basis operating cash flow. | Static string `"Net Operating Cash Flow"` (frontend) | view: inherits hero visibility | Display only — visible subtitle for CA screenshot use case (R10). | n/a — display only |
| Hero | BUILD | Component subtitle | "₹Xin − ₹Yout" (line 2, small grey). [period-sensitive] | Show the period's total inflows and outflows below the headline so the operator sees the composition at a glance. Multi-property variant adds "· across N properties". | `"{fmt(inflows)} in − {fmt(outflows)} out"` + multi-prop suffix if `selected_properties > 1` | view: inherits hero visibility | Display only — subtitle. | n/a — display only |
| Hero | BUILD | ⓘ icon | Tap = inline tooltip; long-press = bottom sheet with full GAAP framing. [always-live tooltip text] | Tap shows brief plain-English tooltip ("Money in vs. money out for the selected period..."). Long-press shows expanded definition. | n/a — UI only. Tooltip strings in §14 microcopy. | view: inherits hero visibility | → Live: Tap for quick tooltip; long-press for the expanded "Net Operating Cash Flow" definition and Excel report pointer.<br>→ Past period: Same as live.<br>→ Future: Same as live. | → Live: T (inline) on tap · BS on long-press<br>→ Past period: SAME<br>→ Future: SAME |

---

## 2. MoM Chip + Per-Bed + Mid-Month Nudge + Deposit-Dependency Callout

> **Endpoint:** EXTEND — same hero query with prior-period computation pass added; per-bed uses live occupied-beds count from rooms service
> **Default property context:** inherited from dashboard
> **Default time context:** MoM chip is **period-sensitive** (compares current period to equivalent prior period); per-bed uses **live** occupancy count
> **Cross-Suite Blockers:** CSB-1, CSB-3
> **Eng note:** MoM chip has 3 states (v1.2 simplified from 4) — green improvement, amber regression > 10%, red sign-reversal-to-negative. Sign-reversal-to-positive (e.g., −₹100 → +₹500) is GREEN per v1.2 bug fix. Same-elapsed-days rule applies to partial-month comparisons. Deposit-dependency callout has 3-condition gate to prevent setup-mode false positives (matches DA-06 maturity discipline).

| Component | Status | Task | Definition + [class] | Plain Formula | Formula (technical) | Permission | Plain Drill | Drill Behavior (technical) |
|-----------|--------|------|----------------------|---------------|---------------------|------------|-------------|----------------------------|
| MoM Chip | BUILD | Comparison value | Compare current period net vs same-elapsed-days prior-period net. [period-sensitive] | Show how this period's net cash flow compares to last period's same elapsed days. For example: May 1–15 vs Apr 1–15 → ▲25% vs ₹68K Apr 1–15 (improvement) or ▼12% vs ₹98K Apr 1–15 (regression). Show both percentage AND prior absolute. Per Generation Spec §14, "same-elapsed-days" is locked globally. | `(current_net − prior_same_elapsed_days_net) / abs(prior_same_elapsed_days_net) × 100` per [[_Build Sheet Generation Spec]] §14. **NEW BUILD** — prior-period pass not implemented. | view: inherits hero visibility | → Live (today): Tap the chip to see prior-period numbers and which window is being compared. The chip shows ▲ or ▼ with % and prior absolute.<br>→ Past period: Same — chip always compares the selected period vs same-elapsed-days prior period.<br>→ Future: Same — for future-extending custom ranges, prior-period comparison still applies (uses elapsed portion). | → Live: T (inline tooltip on tap) — no separate sheet, sign-flip is rendered inline per R1 clarification<br>→ Past period: SAME<br>→ Future: SAME |
| MoM Chip | BUILD | 3-state color rule | Green improvement / amber regression / red sign-reversal-to-negative. [UI rule] | Three distinguishable states: ▲ green when net grew or flipped positive; ▼ amber when net declined > 10% in same direction; ▼ red when previously-positive net flipped to negative. Sign-reversal-to-positive is GREEN (operator turned profitable — good news). | `if (current > prior) OR (prior < 0 AND current >= 0): green;`<br>`elif (prior > 0 AND current < 0): red;`<br>`elif (current < prior AND |delta| > 10%): amber;`<br>`else: hide chip` | view: inherits hero visibility | n/a — display logic only. | n/a — UI render only |
| MoM Chip | BUILD | Sign-reversal glyph | ↻ glyph prepended on either reversal direction. [UI render] | Show a small ↻ glyph alongside the ▲/▼ when the sign flipped, so operator sees both magnitude and reversal at once. | Frontend render — prepend `↻` if `sign(current) != sign(prior)` | view: inherits hero visibility | n/a — UI render. | n/a — UI render only |
| MoM Chip | BUILD | Visibility rule | Hidden when no prior data OR prior = ₹0. [render rule] | Hide the chip if there's no prior period data (account too new) OR prior period was exactly ₹0 (avoids divide-by-zero). Tooltip: "Not enough prior data." | Hide if `prior_net is null` OR `prior_net == 0`. Render "Not enough prior data" empty state if explicitly tapped | view: inherits hero visibility | n/a — render rule. | n/a — UI render only |
| MoM Chip | BUILD | Partial-month rule | Same-elapsed-days comparison for current month-in-progress. [period-sensitive UI render] | If the user is mid-month (May 15) and "This Month" preset is selected, compare against prior-month same-elapsed-days (Apr 1–15). Tooltip on chip: "Same days compared." | If period preset is "This Month" AND current_day < end_of_month: `prior_window = (prior_month_start, prior_month_start + elapsed_days)` | view: inherits hero visibility | → Live: Tap for tooltip explaining the same-days comparison.<br>→ Past period: Doesn't apply — full-period comparison used.<br>→ Future: Doesn't apply. | → Live: T (inline)<br>→ Past period: N/A<br>→ Future: N/A |
| Per-bed chip | BUILD | Per-bed Net Cash Flow | Hero ÷ live occupied beds. Multi-property only. [period-sensitive numerator + always-live denominator] | Show net cash flow per occupied bed, like "+₹850 per occupied bed". Helps compare profitability across properties of different sizes. Only shows when multi-property dashboard. | `current_period_net / live_occupied_bed_count`. Live count from rooms service (NOT period-average). Hidden when `selected_properties == 1` | view: inherits hero visibility | Informational only — no tap target per R1 clarification. | n/a — display only |
| Mid-month nudge | BUILD | Conditional timing nudge text | "💡 It's the [N]th — collections typically peak after the 20th." [period-sensitive conditional render] | Fire a small informational nudge when (a) current day-of-month is < 25, (b) "This Month" PRESET is selected (not custom range), and (c) period contains today. Saves operator from mid-month panic when salary went out but rent hasn't yet come in. | Render if `today.day < 25 AND filter_meta.preset == 'this_month' AND period_contains(today)`. Custom ranges that happen to span current month do NOT trigger | view: inherits hero visibility | Informational only — no tap target per R1 clarification. | n/a — display only |
| Deposit-dependency callout | BUILD | Conditional warning text | "⚠️ ₹35K of inflow is deposits (held as liability). Net for the month excludes this: +₹50K." [period-sensitive conditional render] | Fire a small warning when (a) deposits collected ≥ 30% of total cash inflows, (b) property has ≥ 5 active deposit-holders (maturity gate to prevent setup-mode false positives), and (c) current month is not within property's first 60 days from launch. | Render if `(section_b_inflow / (section_a_inflow + section_b_inflow)) >= 0.30 AND active_deposit_holder_count >= 5 AND property_age_days >= 60` | view: inherits hero visibility | → Live: Tap to open the DA-06 Liabilities snapshot for the period.<br>⚠️ DA-06 cross-screen drill needs "viewing as of [period end]" pill (HB-D1).<br>→ Past period: Same — DA-06 falls back to live with explanation if historical not supported.<br>→ Future: Same. | → Live: BS (DA-06 snapshot) · time: period inherited · "Liabilities — as of [period end]"<br>⚠️ HB-D1<br>→ Past period: SAME<br>→ Future: SAME |

---

## 3. Section A — Operating Cash Flow (Inflows + Outflows)

> **Endpoint:** BUILD — new aggregator composes from collections (inflows) + expenses (outflows) + refunds (non-deposit refunds) query builders
> **Default property context:** inherited from dashboard
> **Default time context:** **period-sensitive** — all lines respect dashboard period
> **Cross-Suite Blockers:** CSB-1, CSB-2 (DA-02 worklist drill — `/invoices/fetchPaymentSettlementDetails` zero middleware), CSB-4 (refund/expense detail unauth)
> **Hard Blockers:** HB-B1 (Deposit% expense exclusion mandatory)
> **Eng note:** Inflows use `payment_mode NOT IN (211, 288)` — see Field Map §2.1. Rent line uses `due_type = 'Rent'` (string). Tenant utility line uses `due_type` NOT IN deposit/advance/rent set. Advance line conditional render: only shown when ≥ 10% of total inflows; otherwise folded into utility line. Outflows use `e.paid_date` for expenses, `r.refund_date` for refunds. **`expense_type ILIKE 'Deposit%'` EXCLUDED from operating outflows per HB-B1** — routed to Section B with EC-01 dedupe heuristic. Per-row MoM delta is **suppressed** (low-N noise per operator audit).
> **Filter codes:** `CashFlowFilterCode` 1800-1899 NOT BUILT.

### 3a. Inflows (Operating)

| Component | Status | Task | Definition + [class] | Plain Formula | Formula (technical) | Permission | Plain Drill | Drill Behavior (technical) |
|-----------|--------|------|----------------------|---------------|---------------------|------------|-------------|----------------------------|
| Rent collected | BUILD | Rent inflow line | Long-stay + short-stay rent collected in period. [period-sensitive] | All money received from tenants for rent during the period — long-stay AND short-stay AND food-rent combined. Excludes deposits, advance, paper transfers. | `SUM(p.net_amount - p.gateway_charges) WHERE p.status = 1 AND p.is_active = 1 AND p.paid_date BETWEEN [period] AND p.payment_mode NOT IN (211, 288) AND inv.due_type = 'Rent'` *(`p` = `payments`; `status=1` = success; `inv` joined via `payments_invoices`)* per Field Map §4.2 collections pattern with rent due_type filter | view: `can_view_invoices` (Jatin gate per HB-P1 composite) | → Live (today): Tap to open the Rent bottom sheet — Long-Stay / Short-Stay / Food-Rent split + Top 5 paying tenants + "View in DA-02" CTA.<br>→ Past period: Same — bottom sheet inherits period.<br>→ Future: Same — for future-extending custom range, future portion is ₹0. | → Live: BS (Rent detail) · time: period inherited · property: inherited · "Rent collected — [Period Total]"<br>→ Past period: SAME<br>→ Future: SAME |
| Tenant utility bills collected | BUILD | Utility inflow line | Electricity / food / maintenance bills paid by tenants — NOT operator's own utility expenses. [period-sensitive] | Money received from tenants for electricity, food/mess, maintenance bills they owed. Distinct from operator's own utility expenses (which are in Outflows). Renamed in v1.1 from "Other charges" to disambiguate. | `SUM(p.net_amount - p.gateway_charges) WHERE p.status = 1 AND p.is_active = 1 AND p.paid_date BETWEEN [period] AND p.payment_mode NOT IN (211, 288) AND inv.due_type NOT IN ('Rent', 'Security Deposit', 'Caution Money', 'Advance')` per Field Map §2.2 (DEPOSIT_DUE_TYPES + ADVANCE_DUE_TYE constants) | view: `can_view_invoices` | → Live: Tap to open the Tenant Utility bottom sheet — sub-split by `due_type` (Electricity / Food / Maintenance / etc.) + Advance sub-line if folded + Top 5 tenants + "View in DA-02".<br>→ Past period: Same.<br>→ Future: Same. | → Live: BS (Utility detail) · "Tenant utility bills collected — [Period Total]"<br>→ Past period: SAME<br>→ Future: SAME |
| Advance received | BUILD | Advance inflow (conditional) | New advance balances paid in by tenants. Only renders when ≥ 10% of total inflows. [period-sensitive conditional render] | Money received from tenants as advance pre-payment for future rent. Only shown as standalone line when it dominates inflows; otherwise folded into Tenant Utility line. | `SUM(p.net_amount - p.gateway_charges) WHERE p.status = 1 AND p.is_active = 1 AND p.paid_date BETWEEN [period] AND p.payment_mode NOT IN (211, 288) AND inv.due_type = 'Advance'` *(uses `ADVANCE_DUE_TYE` constant, typo "TYE" verbatim per Field Map §2.2)*. Render condition: `advance_total / total_inflows >= 0.10` | view: `can_view_invoices` | → Live: Tap to open Advance bottom sheet — list of tenants who paid advance + "View in DA-02 with Advance filter".<br>→ Past period: Same.<br>→ Future: Same. | → Live: BS (Advance detail) · "Advance received — [Period Total]"<br>→ Past period: SAME<br>→ Future: SAME |
| Total Operating Inflows | BUILD | Subtotal line | Sum of all inflow lines. [period-sensitive] | Adds up all the inflow lines above. Shown bold below the inflow block. | `Rent_collected + Tenant_utility + Advance_received` (advance folded into utility if below 10% threshold) | view: inherits | Display only — subtotal. | n/a — display only |

### 3b. Outflows (Operating)

| Component | Status | Task | Definition + [class] | Plain Formula | Formula (technical) | Permission | Plain Drill | Drill Behavior (technical) |
|-----------|--------|------|----------------------|---------------|---------------------|------------|-------------|----------------------------|
| Salary | BUILD | Salary outflow line | Staff salaries paid in period. [period-sensitive] | Money paid out to staff for salary during the period. | `SUM(e.amount) WHERE e.is_active = 1 AND e.paid_date BETWEEN [period] AND e.expense_type ILIKE 'Salary%'` *(`e` = `expenses` table; status field marked TODO-REMOVE per Field Map §1.7, don't filter on status)* | view: `can_view_expenses` | → Live: Tap to open Salary bottom sheet — list of staff salary entries + "View in DA-04 with Salary filter".<br>→ Past period: Same.<br>→ Future: Same. | → Live: BS (Salary detail) · "Salary — [Period Total]"<br>→ Past period: SAME<br>→ Future: SAME |
| Electricity (paid) | BUILD | Electricity outflow line | Operator's own power bills paid. [period-sensitive] | Money paid out for electricity bills the operator pays (not what tenants paid). | `SUM(e.amount) WHERE e.is_active = 1 AND e.paid_date BETWEEN [period] AND e.expense_type ILIKE 'Electricity%'` | view: `can_view_expenses` | → Live: Tap to open Electricity bottom sheet — list of electricity bill entries + "View in DA-04".<br>→ Past period: Same.<br>→ Future: Same. | → Live: BS · "Electricity (paid) — [Period Total]"<br>→ Past period: SAME<br>→ Future: SAME |
| Mess / Food (paid) | BUILD | Mess outflow line | Kitchen / groceries paid. [period-sensitive] | Money paid for kitchen ingredients, groceries, mess running costs. | `SUM(e.amount) WHERE e.is_active = 1 AND e.paid_date BETWEEN [period] AND (e.expense_type ILIKE 'Mess%' OR e.expense_type ILIKE 'Food%')` | view: `can_view_expenses` | → Live: Tap to open Mess/Food bottom sheet — list of entries + "View in DA-04".<br>→ Past period: Same.<br>→ Future: Same. | → Live: BS · "Mess / Food — [Period Total]"<br>→ Past period: SAME<br>→ Future: SAME |
| Maintenance (paid) | BUILD | Maintenance outflow line | Repairs / AMC paid. [period-sensitive] | Money paid for property repairs, annual maintenance contracts. | `SUM(e.amount) WHERE e.is_active = 1 AND e.paid_date BETWEEN [period] AND e.expense_type ILIKE 'Maintenance%'` | view: `can_view_expenses` | → Live: Tap to open Maintenance bottom sheet — list + Top 5 vendors + "View in DA-04".<br>→ Past period: Same.<br>→ Future: Same. | → Live: BS · "Maintenance — [Period Total]"<br>→ Past period: SAME<br>→ Future: SAME |
| Building rent paid | BUILD | Building rent outflow line | Operator's lease paid. [period-sensitive] | Money the operator paid for the building lease (NOT what tenants paid in rent — that's an inflow). | `SUM(e.amount) WHERE e.is_active = 1 AND e.paid_date BETWEEN [period] AND e.expense_type ILIKE 'Rent%'` | view: `can_view_expenses` | → Live: Tap to open Building Rent bottom sheet — list of payments + "View in DA-04".<br>→ Past period: Same.<br>→ Future: Same. | → Live: BS · "Building rent — [Period Total]"<br>→ Past period: SAME<br>→ Future: SAME |
| Other expenses | BUILD | Other outflow line | Everything else (NOT 'Deposit%'). [period-sensitive] | Free-text expense entries that don't match Salary / Electricity / Mess / Maintenance / Rent. Capex flag fires here when total > 25% of outflow. | `SUM(e.amount) WHERE e.is_active = 1 AND e.paid_date BETWEEN [period] AND e.expense_type NOT ILIKE 'Salary%' AND NOT ILIKE 'Electricity%' AND NOT ILIKE 'Mess%' AND NOT ILIKE 'Food%' AND NOT ILIKE 'Maintenance%' AND NOT ILIKE 'Rent%' AND NOT ILIKE 'Deposit%'` *('Deposit%' EXCLUSION mandatory per HB-B1 — routed to Section B)* | view: `can_view_expenses` | → Live: Tap to open Other Expenses bottom sheet — entries + capex flag if applicable + "View in DA-04 — Other category".<br>→ Past period: Same.<br>→ Future: Same. | → Live: BS · "Other expenses — [Period Total]"<br>→ Past period: SAME<br>→ Future: SAME |
| Tenant refunds | BUILD | Non-deposit refund outflow | Refunds disbursed for non-deposit invoices. [period-sensitive] | Money refunded to tenants for non-deposit reasons (e.g., overcharged rent, cancelled service). Deposit refunds are in Section B. | `SUM(r.amount) WHERE r.refund_date BETWEEN [period] AND r.invoice.due_type NOT IN ('Security Deposit', 'Caution Money')` *(`r` = `refunds` table; NO `is_active` and NO `status` columns per Field Map §1.3 — every row is final; filtered via FK `invoice` to `Invoices.due_type`)* per Field Map §2.2 (DEPOSIT_DUE_TYPES) | view: `can_view_invoices` (refunds use viewInvoices per Field Map §3.4) | → Live: Tap to open Tenant Refunds bottom sheet — list of non-deposit refunds + source split (owner / RentOk-funded) + "View in DA-03".<br>⚠️ DA-03 detail endpoint has zero `checkAuth` (CSB-4) — Jatin gate.<br>→ Past period: Same.<br>→ Future: Same. | → Live: BS · "Tenant refunds — [Period Total]"<br>⚠️ CSB-4 on detail<br>→ Past period: SAME<br>→ Future: SAME |
| Total Operating Outflows | BUILD | Subtotal line | Sum of all outflow lines. [period-sensitive] | Adds up all the outflow lines above. Shown bold below the outflow block. | `Salary + Electricity + Mess_Food + Maintenance + Building_rent + Other + Tenant_refunds` | view: inherits | Display only — subtotal. | n/a — display only |

> **Display logic note:** Only show lines with amount > 0. Per-row MoM delta is suppressed (operator audit confirmed low-N noise). No Section A footer total (operator just looked at the same number on the hero — redundant per v1.1).

---

## 4. Section A Footer Callouts (Vendor Concentration / Capex / Discount Memo)

> **Endpoint:** EXTEND — vendor query is same `paid_to` aggregation DA-04 already runs (Field Map §4.4); capex flag uses `is_capex` filter per PRD (verify column existence — see [VERIFY] below); discount memo composes from DA-05 NEW BUILD
> **Default property context:** inherited
> **Default time context:** **period-sensitive**
> **Cross-Suite Blockers:** CSB-4 (DA-04 expense detail unauth)
> **Eng note:** Cap is ONE visible chip + ONE vendor flag at a time. Priority: vendor concentration > capex flag > discount memo. Maturity gate on vendor flag (≥ 5 expense entries) and floor (≥ ₹15K) prevents setup-mode false positives. Each callout has a distinct tap target per "Section A Footer Callouts — Distinct Tap Targets" PRD table.

| Component | Status | Task | Definition + [class] | Plain Formula | Formula (technical) | Permission | Plain Drill | Drill Behavior (technical) |
|-----------|--------|------|----------------------|---------------|---------------------|------------|-------------|----------------------------|
| Vendor concentration flag | BUILD | Top-vendor warning | "⚠️ [Vendor] received ₹X (Y% of outflow). Worth checking." [period-sensitive conditional render] | Fire when one vendor took more than 30% of total outflow this period AND property has ≥ 5 expense entries AND vendor amount ≥ ₹15K. Classic embezzlement signal in property management. | Render if `top_vendor_amount / total_outflow >= 0.30 AND expense_count >= 5 AND top_vendor_amount >= 1500000` (paise). Vendor identified by `SUM(amount) GROUP BY paid_to` per Field Map §1.7 | view: `can_view_expenses` | → Live: Tap to open the DA-04 worklist filtered to that `paid_to` vendor.<br>⚠️ DA-04 detail endpoint zero `checkAuth` (CSB-4).<br>→ Past period: Same — vendor flag respects period.<br>→ Future: Same. | → Live: WL (DA-04) · time: period inherited · property: inherited · filters: `paid_to = '[Vendor Name]', period match` · sort: amount DESC · "[Vendor Name] — [Period]"<br>⚠️ CSB-4<br>→ Past period: SAME<br>→ Future: SAME |
| Capex flag | BUILD + [VERIFY] | Large "Other" warning | "⚠️ Large 'Other' expense — ₹50K furniture purchase." [period-sensitive conditional render] | Fire when "Other" expenses > 25% of total outflow AND any single expense > ₹50K. Helps operator distinguish a one-time capex from recurring spend. **[VERIFY]** Field Map §1.7 does NOT document `is_capex` column on Expenses entity — PRD references `is_capex=1` filter for drill. Either build the column or use heuristic. | Render if `other_expenses / total_outflow > 0.25 AND MAX(e.amount) > 5000000` (paise). **[VERIFY]** drill filter `is_capex=1` — verify column exists or build | view: `can_view_expenses` | → Live: Tap to open the DA-04 worklist filtered to `is_capex=1` (or heuristic equivalent if column doesn't exist).<br>⚠️ CSB-4 on detail.<br>→ Past period: Same.<br>→ Future: Same. | → Live: WL (DA-04) · filters: `is_capex=1, period match` · sort: amount DESC · "Capex flagged — [Period]"<br>⚠️ CSB-4 + [VERIFY] column existence<br>→ Past period: SAME<br>→ Future: SAME |
| Discount memo | BUILD | Discount info chip | "ⓘ ₹13K discounts already deducted." [period-sensitive conditional render] | Always show below Section A when discounts in period > 0. Helps operator understand hero is already net of discounts. Tap reveals owner-funded vs RentOk-funded split. | Render if `discounts_in_period > 0`. Sum from `Credits` rows where `status = 1 AND date_used IN [period]`, group by `source` (`0` = owner, `1` = RentOk) per Field Map §1.4 + §4.9 | view: `can_view_invoices` (discounts use viewInvoices per Field Map §3.4) | → Live: Tap to open the DA-05 Discounts worklist for the period — owner-funded vs RentOk-funded split.<br>→ Past period: Same.<br>→ Future: Same. | → Live: WL (DA-05) · time: period inherited · property: inherited · filters: `period match` · "Discounts — [Period]"<br>⚠️ DA-05 worklist NEW BUILD<br>→ Past period: SAME<br>→ Future: SAME |

> **Note:** `viewCashFlow` cited in PRD doesn't exist as JWT key — composite view perms used per HB-P1. Vendor flag promoted from Phase 2 in v1.2 (operator audit re-evaluation).

---

## 5. Section B — Deposit Money In/Out

> **Endpoint:** BUILD — composes from collections (deposit due_types) + refunds (deposit invoice due_types) with EC-01 dedupe heuristic against `expense_type ILIKE 'Deposit%'` rows
> **Default property context:** inherited
> **Default time context:** **period-sensitive**
> **Cross-Suite Blockers:** CSB-2 (DA-02 worklist drill unauth), CSB-4 (refund detail unauth)
> **Hard Blockers:** HB-B1 (dedupe mandatory), HB-D1 (DA-06 historical mode for "View deposit balance →" CTA)
> **Eng note:** Section B is INFORMATIONAL — does NOT contribute to hero. Section header label + Net Deposit Flow are NOT tappable (v1.2 calibration — accidental DA-06 navigation). Only the explicit "View deposit balance →" CTA navigates. Deposit refunds dedupe heuristic per DA-03 EC-15: same `pg_number` + `paid_to ≈ tenant_name` + `amount ± 5%` + `date ± 7d`.

| Component | Status | Task | Definition + [class] | Plain Formula | Formula (technical) | Permission | Plain Drill | Drill Behavior (technical) |
|-----------|--------|------|----------------------|---------------|---------------------|------------|-------------|----------------------------|
| Deposits collected | BUILD | Deposit inflow line | New SD/CM paid in by tenants. [period-sensitive] | Money received from tenants as security deposit or caution money during the period. Non-revenue — operator owes it back. | `SUM(p.net_amount - p.gateway_charges) WHERE p.status = 1 AND p.is_active = 1 AND p.paid_date BETWEEN [period] AND p.payment_mode NOT IN (211, 288) AND inv.due_type IN ('Security Deposit', 'Caution Money')` *(uses `DEPOSIT_DUE_TYPES` plural array from Field Map §2.2 — NOTE: differs from DA-06 held-amount which uses singular `DEPOSIT_DUE_TYPE` only per Field Map §4.6 / DA-06 HB7)* | view: `can_view_invoices` | → Live: Tap to open Deposits Collected bottom sheet — tenant-by-tenant list + footnote "Held as liability — see DA-06" + "View in DA-02 with Deposit filter".<br>⚠️ DA-02 worklist drill destination zero middleware (CSB-2).<br>→ Past period: Same.<br>→ Future: Same. | → Live: BS · time: period inherited · property: inherited · "Deposits collected — [Period Total]" → cross-screen CTA: WL (DA-02) · filters: `due_type IN ('Security Deposit','Caution Money'), payment_mode NOT IN (211,288)`<br>⚠️ CSB-2<br>→ Past period: SAME<br>→ Future: SAME |
| Deposits returned | BUILD | Deposit outflow line | Deposit refunds disbursed (deduped against 'Deposit%' expenses per EC-01). [period-sensitive] | Money refunded as security deposit or caution money to tenants leaving. Deduped against any `expense_type ILIKE 'Deposit%'` rows to avoid double-count (HB-B1 / EC-01). | `SUM(r.amount) WHERE r.refund_date BETWEEN [period] AND r.invoice.due_type IN ('Security Deposit', 'Caution Money')` UNIONED with `SUM(e.amount) WHERE e.expense_type ILIKE 'Deposit%' AND e.paid_date BETWEEN [period]` minus dedupe heuristic match (same pg + paid_to ≈ tenant + amount ± 5% + date ± 7d) per DA-03 EC-15 | view: `can_view_invoices` | → Live: Tap to open Deposits Returned bottom sheet — tenant-by-tenant list + owner-funded vs RentOk-funded split + unmatched 'Deposit%' expense warning if applicable + "View in DA-03 with Deposit filter".<br>⚠️ DA-03 detail endpoint zero `checkAuth` (CSB-4).<br>→ Past period: Same.<br>→ Future: Same. | → Live: BS · "Deposits returned — [Period Total]" → cross-screen CTA: WL (DA-03) · filters: `invoice.due_type IN ('Security Deposit','Caution Money')`<br>⚠️ CSB-4<br>→ Past period: SAME<br>→ Future: SAME |
| Net Deposit Flow | BUILD | Subtotal line | Inflow − Outflow for deposits. [period-sensitive] | The net change in deposit obligations this period. Positive = operator collected more deposits than refunded; negative = operator refunded more than collected. | `Deposits_collected − Deposits_returned` | view: inherits | Display only — subtotal. NOT tappable per v1.2 (header label not tappable to prevent accidental DA-06 nav). | n/a — display only |
| "View deposit balance →" CTA | BUILD | Cross-screen CTA to DA-06 | Right-aligned button at Section B header. [period-sensitive composite] | The only tappable element in Section B header that navigates. Opens DA-06 with a "viewing as of [period end]" pill. DA-06 is live-snapshot; this CTA forces historical view if supported. | Frontend render → navigate to DA-06 with `as_of_date = period_end_date`. **[VERIFY]** confirm DA-06 v1 supports historical mode (HB-D1) — else fall back to live with explanation pill | view: `can_view_invoices` (DA-06 uses viewInvoices composite per Field Map §3.4) | → Live: Tap to open DA-06 Liabilities snapshot for the period.<br>⚠️ DA-06 historical mode dependency (HB-D1) — falls back to live with explanation pill if not supported.<br>→ Past period: Same.<br>→ Future: Same. | → Live: BS (DA-06) · time: period inherited (as_of pill) · property: inherited · "Liabilities — viewing as of [period end]"<br>⚠️ HB-D1<br>→ Past period: SAME<br>→ Future: SAME |
| Section B footer warning | BUILD | Unmatched deposit expense flag | "⚠️ ₹X in 'Deposit Refund' expenses don't have matching Refund row." [period-sensitive conditional render] | Fire when there are `expense_type ILIKE 'Deposit%'` entries that didn't match a Refund row in the dedupe heuristic. Operator's deposit outflow may be inflated. | Render if `unmatched_deposit_expense_total > 0`. Compute from set difference: `'Deposit%' expenses MINUS deduped matches with Refund rows` | view: `can_view_expenses` | → Live: Tap to open DA-04 worklist filtered to unmatched 'Deposit%' expense entries.<br>⚠️ CSB-4 on detail.<br>→ Past period: Same.<br>→ Future: Same. | → Live: WL (DA-04) · filters: `expense_type ILIKE 'Deposit%', unmatched flag` · "Unmatched deposit refunds — [Period]"<br>⚠️ CSB-4<br>→ Past period: SAME<br>→ Future: SAME |

---

## 6. Cash Flow Trend Chart (6M / This Month)

> **Endpoint:** EXTEND — same hero query run 6 times for 6M view; or 1 time for current month. Has its own period selector — NOT governed by global time filter.
> **Default property context:** inherited from dashboard
> **Default time context:** **period-override** (chart has its own 6M / This Month toggle)
> **Eng note:** Stacked column chart — green column for Operating Inflows (deposits excluded), red column for Operating Outflows. Horizontal zero line. v1.1 dropped the overlay net line per operator audit (visual noise). Trend insight text below chart fires only when `outflow_growth_rate > inflow_growth_rate + 5pts` over 6M (single condition only — v1.1 simplified from 4 copy variants).

| Component | Status | Task | Definition + [class] | Plain Formula | Formula (technical) | Permission | Plain Drill | Drill Behavior (technical) |
|-----------|--------|------|----------------------|---------------|---------------------|------------|-------------|----------------------------|
| Trend chart | BUILD | Stacked column chart | 6M default · This Month toggle. [period-override] | Visual chart showing the last 6 months (or current month broken into weeks) — green column for money in (excluding deposits), red column for money out. Zero line for visual anchor. | Run hero aggregator 6 times with month-bracket periods; or 1 time for current month with weekly buckets. Returns `[{month, inflows, outflows, net}, ...]` | view: composite (inherits hero) | → Live (today): Tap a bar for an inline tooltip showing exact inflow / outflow / net for that month. Tap "View [month] →" CTA inside tooltip to drill into that month's full Cash Flow.<br>→ Past period: Same — chart is independent of global time filter.<br>→ Future: Same — future months show ₹0 contribution. | → Live: T (inline tooltip) on bar tap; BS (drill into month's full Cash Flow) on "View [month] →" CTA · time: period override (that month) · "Cash Flow — [Month Year]"<br>→ Past period: SAME (chart independent)<br>→ Future: SAME |
| Period selector | BUILD | 6M / This Month toggle | Tap to switch chart range. [UI control] | Two pills above the chart — `6M` (default) and `This Month`. Selected state preserved on back. Does NOT affect dashboard hero. | Frontend control — toggles chart `range_mode` between `6m` and `this_month`. Independent of dashboard `period` filter | view: inherits | → Live: Tap to switch.<br>→ Past period: Same — chart stays independent.<br>→ Future: Same. | → Live: chart re-render only (no navigation)<br>→ Past period: SAME<br>→ Future: SAME |
| Trend insight text | BUILD | Conditional copy | "Costs are growing faster than collections — Outflows up 18% vs Inflows up 8% this period." [conditional render] | Single insight line below chart. Only renders when outflow growth rate exceeds inflow growth rate by more than 5 percentage points over the 6-month window. | Render if `(outflow_pct_growth_6M − inflow_pct_growth_6M) > 5`. Compute via simple linear regression slope or end-vs-start delta on each series | view: inherits | Display only — informational. | n/a — display only |

---

## 7. YTD Strip

> **Endpoint:** EXTEND — same hero aggregator with FY (Apr 1 – Mar 31) period override
> **Default property context:** inherited
> **Default time context:** **period-override** (FY-to-date, auto-rolling)
> **Eng note:** Indian fiscal year. Placement varies (v1.2): single-property → below hero; multi-property → below By Property section. Tappable → opens YTD detail screen rerendered for FY scope.

| Component | Status | Task | Definition + [class] | Plain Formula | Formula (technical) | Permission | Plain Drill | Drill Behavior (technical) |
|-----------|--------|------|----------------------|---------------|---------------------|------------|-------------|----------------------------|
| YTD strip | BUILD | FY-to-date cumulative | "FY26 YTD: +₹4.2L net · ₹18.5L in − ₹14.3L out" [period-override] | A thin strip showing financial-year-to-date cumulative figures. Operators love this at March 31 close. Indian FY = April 1 to March 31, auto-rolling. | Run hero aggregator with `period = (fy_start, today)` where `fy_start = April 1 of current/prior calendar year based on today.month >= 4`. Display: `FY[YY] YTD: [net] · [inflows] in − [outflows] out` | view: composite (inherits hero) | → Live (today): Tap to open YTD detail (rerenders Section A + Section B for FY scope). Back returns to DA-07 hero.<br>→ Past period: Same — YTD is always FY-to-date, independent of dashboard period.<br>→ Future: Same. | → Live: FS (YTD detail) · time: period override (FY-to-date) · property: inherited · "FY[YY] YTD — Cash Flow"<br>→ Past period: SAME<br>→ Future: SAME |

---

## 8. By Property Section (Multi-Property Only)

> **Endpoint:** EXTEND — same hero aggregator, group by `property_fk_id`
> **Default property context:** all selected properties (not single)
> **Default time context:** **period-sensitive**
> **Eng note:** Single-property operators DON'T see this section. Sorted by Net Cash Flow ASC (most negative first — operators triage red first). Property spread one-liner has 5 adaptive formats (v1.2). Tap any row → opens that property's full Cash Flow rerendered with single-property scope (R7 inheritance).

| Component | Status | Task | Definition + [class] | Plain Formula | Formula (technical) | Permission | Plain Drill | Drill Behavior (technical) |
|-----------|--------|------|----------------------|---------------|---------------------|------------|-------------|----------------------------|
| Property spread one-liner | BUILD | Adaptive top-level summary | "Best · Worst · Spread" or variant per case. [period-sensitive] | A single line above the property list summarizing the spread. Format adapts: 3+ properties mixed = "Best · Worst · Spread"; 3+ all-positive = "Top · Lowest · Range"; 3+ all-negative = "Least bad · Worst · Spread"; 2 properties = "P1 (X) vs P2 (Y)"; 1 property = suppressed. | Frontend render based on per-property net cash flow array. Tap behavior: opens worst-position property | view: composite | → Live: Tap to open the worst-position property's Cash Flow (or "Lowest" / "Least bad" depending on case).<br>→ Past period: Same.<br>→ Future: Same. | → Live: FS (single-property Cash Flow) · time: period inherited · property: scoped to worst-position property · "[Property Name] — Cash Flow"<br>→ Past period: SAME<br>→ Future: SAME |
| Property row | BUILD | Per-property net cash flow row | Property name + net + inflows/outflows + occupancy share + bar. [period-sensitive] | One row per property showing: name, sign-coded Net Cash Flow, "₹X in − ₹Y out", tenant-night count or occupancy share, proportional bar (relative magnitude). | Per-property: run hero aggregator with single-property scope. Sort by `net ASC`. Bar width: `|net| / max(|all_nets|)` | view: composite | → Live: Tap any row to open that property's full Cash Flow (rerenders hero + Section A + Section B with single-property scope per R7).<br>→ Past period: Same.<br>→ Future: Same. | → Live: FS (single-property Cash Flow) · time: period inherited · property: scoped to tapped property · "[Property Name] — Cash Flow"<br>→ Past period: SAME<br>→ Future: SAME |
| Net column color cue | BUILD | Sign-coded color | Green (≥10% of inflows) / amber (<10%) / red (negative). [UI render] | Color the Net Cash Flow column: green if comfortably positive, amber if thin margin, red if negative. | `if net >= 0.10 * inflows: green; elif net >= 0: amber; else: red` | view: inherits | n/a — display logic. | n/a — UI render only |

---

## 9. Bottom Sheets — 14 Sheets (Component column distinguishes)

> **Endpoint:** Each sheet uses the line-item's source DA query with sub-grouping. Top-5 contributing entities via standard aggregation (`GROUP BY <entity>` ORDER BY `SUM` DESC LIMIT 5). Cross-screen CTA navigates to source DA worklist with filter pre-set per PRD "Worklist Pre-sets" table.
> **Default property context:** inherited
> **Default time context:** **period-sensitive** (all 14 sheets inherit dashboard period)
> **Cross-Suite Blockers:** CSB-2 (DA-02 worklist drill destinations unauth), CSB-3 (Tenant Detail unauth), CSB-4 (refund/expense detail unauth)
> **Eng note:** All 14 bottom sheets share template — header (line + period total) → sub-categorization → Top 5 entities → "View in DA-X" CTA. ≤ 8 visible items above the fold. NO per-line 6M mini-trend (v1.1 cut). Cross-screen CTAs apply R11 breadcrumb pattern ("← From Cash Flow"). Sub-rows tappable: sub-categorization rows drill to relevant DA worklist with filter; Top-5 entity rows drill to entity detail (R7 scope inheritance). All sheets dismissible by swipe-down or tap-outside; do NOT navigate away on body tap (only explicit CTA).

| Component (sheet) | Status | Task | Definition + [class] | Plain Formula | Formula (technical) | Permission | Plain Drill | Drill Behavior (technical) |
|-------------------|--------|------|----------------------|---------------|---------------------|------------|-------------|----------------------------|
| 1. Hero (Net Cash Flow) | BUILD | Full A+B breakdown | Expanded form of Section A + Section B. [period-sensitive] | Tap on hero opens this — shows full inflow lines, full outflow lines, full Section B, all in one expanded view. | Composes hero aggregator output into expanded sheet template | view: composite | → Live: Sub-row taps drill to source DA worklist per line.<br>→ Past period: Same.<br>→ Future: Same. | → Live: BS · sub-row drills to source DA worklist per line<br>→ Past period: SAME<br>→ Future: SAME |
| 2. Rent collected | BUILD | Long/Short/Food split + Top 5 tenants | Sub-categorization by stay type + top contributing tenants. [period-sensitive] | Splits rent into Long-Stay / Short-Stay / Food-Rent. Top 5 paying tenants with amounts. CTA: "View in DA-02". | `SUM(p.net_amount - p.gateway_charges) GROUP BY stay_type` *(`stay_type` derived from invoice metadata or due_type subcategory — verify implementation)* and `... GROUP BY p.firebase_id ORDER BY SUM DESC LIMIT 5` | view: `can_view_invoices` | → Live: Sub-rows tap → DA-02 worklist with stay type / tenant filter (R7 scope).<br>⚠️ DA-02 row drill `/invoices/fetchPaymentSettlementDetails` zero middleware (CSB-2).<br>→ Past period: Same.<br>→ Future: Same. | → Live: WL (DA-02) on sub-row · filters: `due_type='Rent'` + sub-filter · "Rent — [Period]"<br>⚠️ CSB-2<br>→ Past period: SAME<br>→ Future: SAME |
| 3. Tenant utility bills collected | BUILD | due_type sub-split + Advance sub-line + Top 5 tenants | Sub-categorization by `due_type` + Top 5 tenants. [period-sensitive] | Splits utility into Electricity / Food / Maintenance / etc. Includes Advance sub-line if folded (< 10%). Top 5 paying tenants. CTA: "View in DA-02". | `... GROUP BY inv.due_type` and `... GROUP BY p.firebase_id ORDER BY SUM DESC LIMIT 5` | view: `can_view_invoices` | → Live: Sub-rows → DA-02 worklist with due_type filter.<br>⚠️ CSB-2.<br>→ Past period: Same.<br>→ Future: Same. | → Live: WL (DA-02) on sub-row · filters: `due_type=<sub>` · "[Sub-type] — [Period]"<br>⚠️ CSB-2<br>→ Past period: SAME<br>→ Future: SAME |
| 4. Advance received (when standalone) | BUILD | Advance-paying tenants list | Only when ≥ 10% of inflows — list of tenants who paid advance. [period-sensitive conditional] | List of tenants who paid advance during period. CTA: "View in DA-02 with Advance filter". | `SELECT firebase_id, SUM(p.net_amount - p.gateway_charges) ... WHERE inv.due_type = 'Advance' GROUP BY firebase_id` | view: `can_view_invoices` | → Live: CTA → DA-02 worklist with `due_type='Advance', payment_mode NOT IN (211,288)`.<br>→ Past period: Same.<br>→ Future: Same. | → Live: WL (DA-02) · filters: `due_type='Advance', payment_mode NOT IN (211,288)` · "Advance received — [Period]"<br>→ Past period: SAME<br>→ Future: SAME |
| 5. Salary outflow | BUILD | Staff salary entries list | List of expense entries matching Salary%. [period-sensitive] | List of all staff salary entries during period. CTA: "View in DA-04 with Salary filter". | `SELECT * FROM expenses WHERE expense_type ILIKE 'Salary%' AND ...` | view: `can_view_expenses` | → Live: CTA → DA-04 worklist with Salary filter.<br>⚠️ DA-04 detail unauth (CSB-4).<br>→ Past period: Same.<br>→ Future: Same. | → Live: WL (DA-04) · filters: `expense_type ILIKE 'Salary%'` · "Salary — [Period]"<br>⚠️ CSB-4<br>→ Past period: SAME<br>→ Future: SAME |
| 6. Electricity outflow | BUILD | Electricity bill entries list | List of expense entries matching Electricity%. [period-sensitive] | List of all electricity bill entries. CTA: "View in DA-04". | `SELECT * FROM expenses WHERE expense_type ILIKE 'Electricity%' AND ...` | view: `can_view_expenses` | → Live: CTA → DA-04 with Electricity filter.<br>⚠️ CSB-4.<br>→ Past period: Same.<br>→ Future: Same. | → Live: WL (DA-04) · filters: `expense_type ILIKE 'Electricity%'` · "Electricity — [Period]"<br>⚠️ CSB-4<br>→ Past period: SAME<br>→ Future: SAME |
| 7. Mess/Food outflow | BUILD | Mess/Food entries list | List of Mess% or Food% entries. [period-sensitive] | List of mess and food expense entries. CTA: "View in DA-04". | `... WHERE expense_type ILIKE 'Mess%' OR ILIKE 'Food%'` | view: `can_view_expenses` | → Live: CTA → DA-04 with Mess/Food filter.<br>⚠️ CSB-4.<br>→ Past period: Same.<br>→ Future: Same. | → Live: WL (DA-04) · filters: `expense_type ILIKE 'Mess%' OR ILIKE 'Food%'` · "Mess/Food — [Period]"<br>⚠️ CSB-4<br>→ Past period: SAME<br>→ Future: SAME |
| 8. Maintenance outflow | BUILD | Maintenance entries + Top 5 vendors | List of Maintenance% entries + Top 5 by `paid_to`. [period-sensitive] | List of maintenance entries, with Top 5 vendors by `paid_to` aggregation. CTA: "View in DA-04". | `... WHERE expense_type ILIKE 'Maintenance%'` and `... GROUP BY paid_to ORDER BY SUM DESC LIMIT 5` | view: `can_view_expenses` | → Live: Sub-rows → DA-04 with vendor filter; CTA → full Maintenance worklist.<br>⚠️ CSB-4.<br>→ Past period: Same.<br>→ Future: Same. | → Live: WL (DA-04) on vendor sub-row · filters: `paid_to=<vendor>, expense_type ILIKE 'Maintenance%'` · "[Vendor] — [Period]"<br>⚠️ CSB-4<br>→ Past period: SAME<br>→ Future: SAME |
| 9. Building rent outflow | BUILD | Building rent entries list | List of Rent% expense entries (operator's lease). [period-sensitive] | List of building rent payments. CTA: "View in DA-04". | `... WHERE expense_type ILIKE 'Rent%'` | view: `can_view_expenses` | → Live: CTA → DA-04 with Rent filter.<br>⚠️ CSB-4.<br>→ Past period: Same.<br>→ Future: Same. | → Live: WL (DA-04) · filters: `expense_type ILIKE 'Rent%'` · "Building rent — [Period]"<br>⚠️ CSB-4<br>→ Past period: SAME<br>→ Future: SAME |
| 10. Other expenses outflow | BUILD | Other entries + capex flag | Free-text entries not matching above + capex flag if applicable. [period-sensitive] | Other expenses with capex flag highlighted if a single one exceeded ₹50K. CTA: "View in DA-04 — Other category". | `... WHERE expense_type NOT ILIKE 'Salary%' AND NOT 'Electricity%' AND NOT 'Mess%' AND NOT 'Food%' AND NOT 'Maintenance%' AND NOT 'Rent%' AND NOT 'Deposit%'` | view: `can_view_expenses` | → Live: CTA → DA-04 with Other filter; capex sub-row → `is_capex=1` filter [VERIFY column].<br>⚠️ CSB-4.<br>→ Past period: Same.<br>→ Future: Same. | → Live: WL (DA-04) · filters: Other category · "Other — [Period]"<br>⚠️ CSB-4 + [VERIFY] is_capex<br>→ Past period: SAME<br>→ Future: SAME |
| 11. Tenant refunds outflow | BUILD | Non-deposit refunds + source split | List of non-deposit refunds + owner-funded vs RentOk-funded split. [period-sensitive] | List of refunds where invoice is non-deposit. Split by `source` if applicable. CTA: "View in DA-03". | `SELECT r.* FROM refunds r JOIN invoices inv ON r.invoice_id = inv.id WHERE inv.due_type NOT IN ('Security Deposit','Caution Money') AND r.refund_date BETWEEN [period]` | view: `can_view_invoices` | → Live: CTA → DA-03 worklist with non-deposit filter.<br>⚠️ DA-03 detail zero `checkAuth` (CSB-4).<br>→ Past period: Same.<br>→ Future: Same. | → Live: WL (DA-03) · filters: `invoice.due_type NOT IN ('Security Deposit','Caution Money')` · "Tenant refunds — [Period]"<br>⚠️ CSB-4<br>→ Past period: SAME<br>→ Future: SAME |
| 12. Deposits collected (Section B) | BUILD | Tenant-by-tenant deposit list | Per-tenant deposits paid + footnote "Held as liability — see DA-06". [period-sensitive] | List of tenants who paid SD/CM during period. Footnote points to DA-06. CTA: "View in DA-02 with Deposit filter". | `SELECT firebase_id, SUM ... WHERE inv.due_type IN ('Security Deposit','Caution Money') GROUP BY firebase_id` | view: `can_view_invoices` | → Live: CTA → DA-02 with Deposit filter; sub-row tap → tenant detail.<br>⚠️ CSB-2 (worklist row drill) + CSB-3 (Tenant Detail).<br>→ Past period: Same.<br>→ Future: Same. | → Live: WL (DA-02) · filters: `due_type IN ('Security Deposit','Caution Money'), payment_mode NOT IN (211,288)` · "Deposits — [Period]"<br>⚠️ CSB-2 + CSB-3<br>→ Past period: SAME<br>→ Future: SAME |
| 13. Deposits returned (Section B) | BUILD | Tenant-by-tenant deposit refunds + source split | Per-tenant deposit refunds + owner vs RentOk-funded split. [period-sensitive] | List of tenants who got deposit refunds. Source split (owner / RentOk-funded). CTA: "View in DA-03 with Deposit filter". | `SELECT firebase_id, SUM(r.amount) ... WHERE r.invoice.due_type IN ('Security Deposit','Caution Money') GROUP BY firebase_id` and source split derived from RentOk-funded credit refund pattern | view: `can_view_invoices` | → Live: CTA → DA-03 with Deposit filter.<br>⚠️ CSB-4.<br>→ Past period: Same.<br>→ Future: Same. | → Live: WL (DA-03) · filters: `invoice.due_type IN ('Security Deposit','Caution Money')` · "Deposit refunds — [Period]"<br>⚠️ CSB-4<br>→ Past period: SAME<br>→ Future: SAME |
| 14. Trend chart bar | BUILD | Single-month full Cash Flow rerender | Tap a bar → that month's full hero + Section A + Section B. [period-override] | Tap any bar in trend chart to drill into that month's complete Cash Flow view (hero + sections rerendered with that month's data + period override pill). | Run hero aggregator with `period = (month_start, month_end)`. Render full DA-07 view with override pill | view: composite | → Live: Drill into month-scope full Cash Flow.<br>→ Past period: Same.<br>→ Future: Future months show ₹0. | → Live: BS (single-month Cash Flow) · time: period override (that month) · "Cash Flow — [Month Year]"<br>→ Past period: SAME<br>→ Future: SAME (₹0 for future months) |

> **Bottom Sheet sub-row drill discipline (PRD "Bottom Sheet Sub-Row Drills"):** sub-categorization rows (Long-Stay/Short-Stay/Food-Rent; due_type sub-splits) drill to relevant DA worklist with that filter applied (R7 scope inheritance). Top-5 entity rows (tenant/vendor/property) cross-screen drill to entity detail with breadcrumb "← From DA-07 Cash Flow" (R11). Property spread one-liner tap opens By Property sheet anchored to worst-position property.

---

## 10. Excel Export CTA (Existing Endpoint — Phase 1 Column Extension)

> **Endpoint:** EXTEND — `POST /reports/cashflow-report` ALREADY EXISTS at `src/routes/reports.ts:119` (HeaderValidator-protected). Service: `src/services/reports/generateCashFlowReport.ts`. Phase 1 work is **column extension** (HB-B2) + double-count bug fix (HB-B1), NOT new endpoint creation. ⚠️ Earlier "NEW BUILD" wording was wrong.
> **Default property context:** inherited from dashboard
> **Default time context:** **period-sensitive** (uses dashboard's selected period)
> **Cross-Suite Blockers:** none (HeaderValidator-protected)
> **Hard Blockers:** HB-B1 (double-count fix at `generateCashFlowReport.ts:393`), HB-B2 (column shape match DA-07 sections)
> **Eng note:** Excel CTA lives in overflow menu (⋮) — NOT permanent hero chip (v1.1 calibration: 95% of operators don't email Excels). Email-based delivery — async, no in-app download. Toast: "Cash Flow Excel sent to [email]. You'll receive it in a few minutes." DB toggle for Excel export: `cashflow_report` (per Field Map §3.2).

| Component | Status | Task | Definition + [class] | Plain Formula | Formula (technical) | Permission | Plain Drill | Drill Behavior (technical) |
|-----------|--------|------|----------------------|---------------|---------------------|------------|-------------|----------------------------|
| Excel CTA (overflow menu) | EXTEND | Email Cash Flow Excel menu item | Triggers existing `/reports/cashflow-report` for selected period. [period-sensitive action] | A menu item in the overflow (⋮) menu. Tapping triggers backend Excel generation; user gets a confirmation toast and receives the Excel via email. Pre-launch fixes: B1 (double-count) and B2 (column extension) mandatory. | `POST /reports/cashflow-report` with `{pg_number, start_date, end_date}` payload — service at `src/services/reports/generateCashFlowReport.ts`. **Phase 1 changes:** (a) `generateCashFlowReport.ts:393` — replace `totalExpenses = expenseTotal + refunds` with deduped formula per HB-B1; (b) extend column shape to match DA-07 14-bottom-sheet structure per HB-B2 | view: `can_view_invoices AND can_view_expenses` (composite — same as screen) · **action:** DB column `cashflow_report` via `checkAuthInDb` per Field Map §3.2 | → Live (today): Tap menu item → toast confirms email queued. No in-app navigation.<br>→ Past period: Same — uses dashboard's selected period.<br>→ Future: Same. | → Live: M (toast) · time: period inherited · property: inherited · "Cash Flow Excel sent to [email]"<br>→ Past period: SAME<br>→ Future: SAME |

---

## 11. Discoverability Footer (Informational)

> **Endpoint:** none — pure UI render
> **Default property context:** inherited
> **Default time context:** always-live label text
> **Eng note:** Static text at bottom of screen — visible in screenshots so a CA reading a forwarded screenshot knows the Excel report exists. v1.2 addition addressing the CA-screenshot persona-stress-test failure. NO tap target (informational only) per R1 clarification.

| Component | Status | Task | Definition + [class] | Plain Formula | Formula (technical) | Permission | Plain Drill | Drill Behavior (technical) |
|-----------|--------|------|----------------------|---------------|---------------------|------------|-------------|----------------------------|
| Discoverability footer | BUILD | Static informational text | "Need a detailed report for accounting? Email Cash Flow Excel from menu (⋮)." [always-live label text] | A small static line at the bottom of the screen pointing to the Excel CTA in the overflow menu. Critical for CA screenshot use case (R10) — visible in forwarded screenshots so accountant knows Excel exists. | Static string render at screen footer | view: inherits | Display only — no tap target. | n/a — display only |

---

## 12. Worklist Behavior (NEW BUILD — for hero/property bottom sheet drills only)

> **Endpoint:** **NEW BUILD** — `POST /v1/list_screens/cashflow/list/filters` (HB-E1). Mirror `src/v1/list_screens/expenses/` pattern.
> **Default property context:** inherited
> **Default time context:** **period-sensitive**
> **Cross-Suite Blockers:** CSB-1, CSB-2
> **Hard Blockers:** HB-E1 (endpoint NEW BUILD), HB-F1 (filter codes 1800-1899 NEW BUILD)
> **Eng note:** **Cash Flow does NOT have its own transaction-level worklist** — it dispatches to source DA worklists for transactions (per PRD: "No standalone DA-07 worklist"). However, the hero block tap and By Property row tap need a "single-month full Cash Flow" or "single-property full Cash Flow" rerender, which composes the same aggregator output. The list endpoint is for these composite views, not for transaction lists.
> **Filter codes:** `CashFlowFilterCode` 1800-1899 NOT BUILT.

| Component | Status | Task | Definition + [class] | Plain Formula | Formula (technical) | Permission | Plain Drill | Drill Behavior (technical) |
|-----------|--------|------|----------------------|---------------|---------------------|------------|-------------|----------------------------|
| Cash Flow list endpoint | BUILD | NEW endpoint for composite drill states | Returns full A+B breakdown for given period+property scope. [period-sensitive] | The endpoint that powers single-month and single-property Cash Flow rerenders. Wraps the same hero aggregator with optional filter code overrides. | `POST /v1/list_screens/cashflow/list/filters` with `{pg_number, start_date, end_date, filter_codes?, single_property?, limit, offset}`. Composes from collections + expenses + refunds query builders | view: composite (HB-P1) | → Live: Used internally for hero / by-property / trend-chart-bar drills.<br>→ Past period: Same.<br>→ Future: Same. | n/a — internal endpoint |
| `CashFlowFilterCode` enum | BUILD | NEW filter code enum | Range 1800-1899 free. [build-time constant] | New TypeScript enum for cash-flow-specific filter codes. Pattern matches DuesFilterCode / CollectionFilterCode. | `export enum CashFlowFilterCode { OPERATING_INFLOWS = 1801, OPERATING_OUTFLOWS = 1802, DEPOSIT_INFLOWS = 1803, DEPOSIT_OUTFLOWS = 1804, ... }` at `src/v1/constants/filterCodes.ts` | n/a — constant | n/a — build-time. | n/a |

---

## 13. Reconciliation Invariants

QA-runnable checks. Strict equalities or bounds.

| ID | Invariant | Tolerance |
|----|-----------|-----------|
| RI-1 | `Hero Net = Total_Operating_Inflows − Total_Operating_Outflows` | strict equality |
| RI-2 | `Total_Operating_Inflows = Rent + Tenant_Utility + Advance_Received` (advance folded if < 10%) | strict equality |
| RI-3 | `Total_Operating_Outflows = Salary + Electricity + Mess_Food + Maintenance + Building_Rent + Other + Tenant_Refunds` | strict equality |
| RI-4 | `Net_Deposit_Flow = Deposits_collected − Deposits_returned` | strict equality |
| RI-5 | Section B Net is INDEPENDENT of Hero — Hero must NOT include Section B values | strict (audit query: hero formula excludes deposit due_types) |
| RI-6 | Mode 211 / 288 transactions contribute ZERO to all DA-07 sums (paper transfers) | strict equality (test: insert mode-211 payment, verify hero unchanged) |
| RI-7 | `expense_type ILIKE 'Deposit%'` rows must NOT appear in any operating outflow line | strict equality (HB-B1 enforcement test) |
| RI-8 | Trend chart 6M sum across green columns ≥ 0; across red columns ≥ 0 | sign check |
| RI-9 | YTD Net = sum of monthly hero values from FY start to today (month-by-month aggregation reconciles to YTD strip) | strict equality |
| RI-10 | By Property: `SUM(per_property_net) = Hero_Net` (multi-property scope) | strict equality |
| RI-11 | Per-bed value = Hero / live_occupied_beds (when multi-property) — never aggregated period-average | strict (audit: bed count is `today` count) |
| RI-12 | Vendor concentration flag: when fired, `top_vendor_amount >= 0.30 * total_outflow AND expense_count >= 5 AND top_vendor_amount >= 1500000` paise | gate triple-check |
| RI-13 | Deposit-dependency callout: when fired, `(section_b_inflow / total_inflow) >= 0.30 AND active_deposit_holders >= 5 AND property_age_days >= 60` | gate triple-check |
| RI-14 | MoM chip sign-reversal: prior < 0 AND current > 0 → chip color = green (NOT red); prior > 0 AND current < 0 → chip color = red | bug-fix regression test (v1.2) |
| RI-15 | EC-01 dedupe: matched 'Deposit%' expense rows MUST NOT contribute to Section B inflow (already counted via Refund row) | strict equality (test scenario: insert paired Refund + matching Deposit% Expense; assert single contribution) |

---

## 14. Edge Cases

| ID | Description | Behavior | Trigger condition |
|----|-------------|----------|-------------------|
| EC-01 | Deposit refund logged as both Refund AND `expense_type ILIKE 'Deposit%'` Expense | Section A excludes 'Deposit%'; Section B unions Refunds + Deposit-named expenses with dedupe heuristic; unmatched 'Deposit%' surface as Section B footer warning | Bug-prone operator workflow; HB-B1 mandatory fix at `generateCashFlowReport.ts:393` |
| EC-02 | Mode 211 / 288 phantom inflow | Excluded entirely from all DA-07 sums; DA-02 surfaces as separate adjustment row | Production code at `helpers.ts:430-450` already excludes; DA-07 query enforces |
| EC-03 | Mid-month timing illusion | No auto-detect; trend chart provides historical context; mid-month nudge fires when conditions met | day-of-month < 25 AND "This Month" preset AND period contains today |
| EC-04 | Discount memo when discounts > 0 | Memo annotation always shows below Section A when `discounts_in_period > 0` | `SUM(Credits.amount) WHERE status=1 AND date_used IN [period] > 0` |
| EC-05 | RentOk-funded discount with no make-whole | No inflow line; memo annotation only ("Of which ₹Y RentOk-funded — no impact on your revenue") | `Credits.source = 1` per Field Map §1.4 |
| EC-06 | Capex / large "Other" expense distortion | Capex flag fires; tooltip provides context | `other_expenses > 25% of total_outflow AND MAX(single_expense) > 50000 INR` |
| EC-07 | All inflow is deposits, no operating revenue | Net Operating Cash Flow may be negative; deposit-dependency callout fires; correct signal for early-stage property | `section_a_inflow = 0 AND section_b_inflow > 0` |
| EC-08 | Net Cash Flow = exactly ₹0 | Hero shows ₹0 in neutral; no MoM chip (would be ÷0 unless prior also ₹0); subtitle shows "₹X in − ₹X out" | `inflows = outflows` |
| EC-09 | Period extends beyond today | Future dates show ₹0 contribution; helper text in time filter chip clarifies | custom range `end_date > today` |
| EC-10 | Property has no expenses but has collections | Outflows = ₹0; Section A Outflows hidden; per-bed cash flow shown (multi-property only) | `total_outflow = 0 AND total_inflow > 0` |
| EC-11 | Multi-property, one property has zero activity | By Property breakdown shows that property at the bottom with ₹0; no special treatment | `per_property_net = 0 AND per_property_inflows = 0 AND per_property_outflows = 0` |
| EC-12 | Refund without linked invoice (DA-03 EC-05 inheritance) | Refund amount still counts; linked invoice context shows "Linked invoice unavailable" | `r.invoice_id IS NULL` (rare — FK is required per Field Map §1.3, but legacy data may exist) |
| EC-13 | Same-period deposit collection AND refund for same tenant | Both events count: Section B inflow + Section B outflow; Net Deposit Flow = ₹0 for tenant; operating cash flow unaffected | tenant pays SD then moves out within same period |
| EC-14 | GST collected (commercial PG / >₹20L turnover) | v1.0 doesn't surface; Phase 2 footer note: "Of ₹X collected, ₹Y is GST — owed to government" | property is GST-registered |
| EC-15 | Off-system owner withdrawal | Cannot detect; disclaimer in tooltip; Phase 2: "Owner Withdrawal" expense category | operator takes cash from till for personal use, doesn't record |

---

## 15. Microcopy

Static strings used across DA-07. PRD's "Words on the Screen" section is canonical; this is copies-with-attribution.

| Component | Task | Exact string |
|-----------|------|--------------|
| Hero ⓘ tooltip (tap) | Brief plain-English | "Money in vs. money out for the selected period. The hero shows net operating cash flow — what you made after expenses and refunds. Deposits are tracked separately because they have to be returned." |
| Hero number long-press | Expanded GAAP | "This is your operating cash profit — what's left after expenses and refunds. Security deposits aren't included because you have to give them back. For full reconciliation to your bank, download the Excel report." |
| Per-bed ⓘ tooltip | Per-bed definition | "Net cash flow ÷ currently occupied beds. Helps compare profitability across properties of different sizes. Occupancy count is current (live), not period-average." |
| Operating Inflows ⓘ | Inflows definition | "Cash actually received during the period — rent, electricity, food, advances. Excludes deposits (those are in Section B) and adjustments (no real cash moved)." |
| Operating Outflows ⓘ | Outflows definition | "Cash actually paid out — staff salaries, utilities, maintenance, refunds. Excludes deposit refunds (those are in Section B)." |
| Section B subtitle | Section B framing | "Not part of your earnings — you'll return this." |
| Deposit Money In/Out ⓘ | Section B definition | "Deposit money in vs. out. This is non-revenue cash — when you collect a deposit, you take a liability to return it. Tracked separately from operating cash flow." |
| By Property ⓘ | By Property definition | "Each property's net cash flow this period. Sorted by lowest first so you can spot the property losing money. Tap any row to see that property's full breakdown." |
| Trend chart ⓘ | Trend definition | "Money in (green) and money out (red) for the last 6 months. Watch if the green is consistently above the red — that's a healthy business." |
| Deposit-dependency callout | Conditional warning | "⚠️ ₹{X} of inflow is deposits (held as liability). Net for the month excludes this: +₹{Y}." |
| Vendor concentration callout | Conditional warning | "⚠️ {Vendor Name} received ₹{X} ({Y}% of total outflow this period). Worth checking." |
| Mid-month nudge | Conditional informational | "💡 It's the {N}th — collections typically peak after the 20th. This may turn green by month-end." |
| MoM chip — partial month | Tooltip on chip | "Same days compared." |
| MoM chip — no prior data | Empty state | "Not enough prior data." |
| Empty: no transactions | Empty state | "No cash movement in this period. Wrong period? Change the date filter." (CTA: "Change filter") |
| Empty: future range | Empty state | "Cash Flow uses dates money moved. Future dates show ₹0." |
| Empty: new property | Empty state | "No cash movement yet. They'll appear here once you collect rent or pay expenses." |
| Excel toast | Action confirmation | "Cash Flow Excel sent to [email]. You'll receive it in a few minutes." |
| Discoverability footer | Static informational | "Need a detailed report for accounting? Email Cash Flow Excel from menu (⋮)." |
| Network error | Recovery | "Couldn't load Cash Flow. Check your connection." (CTA: "Retry") |
| Section error | Recovery | "Couldn't load this section." (CTA: "Retry" on that section) |
| Excel error | Recovery | "Cash Flow Excel couldn't be generated. Please try again." (CTA: "Retry") |

---

## 16. Smoke Test Commands

For every endpoint cited in the Endpoint column or its drill destinations, one curl example.

```bash
# Hero — Net Cash Flow for This Month, single property (after NEW BUILD)
curl -X POST 'https://<host>/v1/list_screens/cashflow/list/filters' \
  -H 'Authorization: Bearer <token>' \
  -H 'Content-Type: application/json' \
  -d '{ "pg_number": "PG123", "start_date": "2026-05-01", "end_date": "2026-05-31", "filter_codes": [1801, 1802, 1803, 1804], "limit": 20, "offset": 0 }'
```

```bash
# Excel Export — existing endpoint (HeaderValidator-protected)
curl -X POST 'https://<host>/reports/cashflow-report' \
  -H 'Authorization: Bearer <token>' \
  -H 'Content-Type: application/json' \
  -d '{ "pg_number": "PG123", "start_date": "2026-05-01", "end_date": "2026-05-31", "email": "operator@example.com" }'
```

```bash
# Cross-screen drill — Collections worklist (Operating Inflows) with mode 211/288 exclusion
curl -X POST 'https://<host>/v1/list_screens/collections/list/filters' \
  -H 'Authorization: Bearer <token>' \
  -H 'Content-Type: application/json' \
  -d '{ "pg_number": "PG123", "start_date": "2026-05-01", "end_date": "2026-05-31", "exclude_payment_modes": [211, 288], "limit": 20, "offset": 0 }'
```

```bash
# Cross-screen drill — Expenses worklist (Salary outflow)
curl -X POST 'https://<host>/v1/list_screens/expenses/list/filters' \
  -H 'Authorization: Bearer <token>' \
  -H 'Content-Type: application/json' \
  -d '{ "pg_number": "PG123", "start_date": "2026-05-01", "end_date": "2026-05-31", "expense_type_filter": "Salary%", "limit": 20, "offset": 0 }'
```

```bash
# Cross-screen drill — Refunds worklist (Tenant refunds, non-deposit) — NEW BUILD per DA-03
curl -X POST 'https://<host>/v1/list_screens/refunds/list/filters' \
  -H 'Authorization: Bearer <token>' \
  -H 'Content-Type: application/json' \
  -d '{ "pg_number": "PG123", "start_date": "2026-05-01", "end_date": "2026-05-31", "exclude_due_types": ["Security Deposit","Caution Money"], "limit": 20, "offset": 0 }'
```

```bash
# Vendor concentration drill — DA-04 worklist filtered to top vendor
curl -X POST 'https://<host>/v1/list_screens/expenses/list/filters' \
  -H 'Authorization: Bearer <token>' \
  -H 'Content-Type: application/json' \
  -d '{ "pg_number": "PG123", "start_date": "2026-05-01", "end_date": "2026-05-31", "paid_to": "ABC Vendor Pvt Ltd", "limit": 20, "offset": 0 }'
```

---

## How to Use This Doc

- Pick a row → that's a ticket (or part of one). Status column tells you whether to extend, build, compose, or wire UI.
- Plain Formula + Plain Drill columns are for PM / CS / QA review.
- Formula (technical) + Drill Behavior (technical) columns are for implementation.
- All code references cite file:line. First-occurrence-per-section gets a plain-English gloss in italics.
- For "why" or rationale, see [[DA-07 Cash Flow Detailed Analytics]].
- For entity fields, formulas, permissions, endpoints, see [[_Ground Truth Field Map]].
- For format conventions, see [[_Build Sheet Generation Spec]].
- If a row is ambiguous, the PRD is the source of truth. Don't guess — ask.

---

## Maintenance Log

| Date | Change | By |
|------|--------|-----|
| 2026-05-08 | Initial generation per [[_Build Sheet Generation Spec]] V1. Verified against Field Map V1 (post-master commit `728abfd68`). | Phase 3 sub-agent |
| 2026-05-08 | Phase 3 finding (HB-P1) — `viewCashFlow` JWT key does NOT exist; recommend composite `viewInvoices AND viewExpenses`. Back-port to PRD Pre-Launch Engineering Blockers. | Phase 3 sub-agent |
| 2026-05-11 | DA-01 spec audit propagation: (a) ⓘ icon convention updated to "single-tap → BS only" per Generation Spec §15; (b) CSB-8 (widget self-added permission gap, security) added — DA-07 NEW BUILD cash-flow widget endpoints must enforce self-added filter from day 1. | PM |
| 2026-05-08 | Phase 3 finding (HB-E1) — `/v1/list_screens/cashflow/list/filters` is NEW BUILD; mirror expenses pattern. Back-port to PRD. | Phase 3 sub-agent |
| 2026-05-08 | Phase 3 finding (HB-F1) — `CashFlowFilterCode` enum NOT BUILT; reserve range 1800-1899. Back-port to Field Map §2.8 status. | Phase 3 sub-agent |
| 2026-05-08 | Phase 3 [VERIFY] — capex flag drill cites `is_capex=1` filter on Expenses, but Field Map §1.7 does NOT document this column. Either column needs to be built OR drill uses heuristic (Other > ₹50K). Back-port to Field Map §1.7 if column exists; else flag for build. | Phase 3 sub-agent |
| 2026-05-08 | Phase 3 confirmation — Excel CTA endpoint `/reports/cashflow-report` EXISTS at `src/routes/reports.ts:119` per Field Map §4.8. Phase 1 work is column extension (HB-B2) + double-count fix (HB-B1), NOT new endpoint. Earlier "NEW BUILD" wording in some PRD drafts was wrong. | Phase 3 sub-agent |

---

## Related Documents

- `[[DA-07 Cash Flow Detailed Analytics]]` — DA-07 PRD V1.3 (canonical "why")
- `[[DA-01 Build Sheet]]` — V2 exemplar
- `[[DA-02 Collections Detailed Analytics]]` · `[[DA-03 Refunds Detailed Analytics]]` · `[[DA-04 Expenses Detailed Analytics]]` · `[[DA-05 Discounts Detailed Analytics]]` · `[[DA-06 Liabilities Detailed Analytics]]` — cross-screen drill destinations
- `[[_Ground Truth Field Map]]` — entity / formula / permission / endpoint reference
- `[[_Build Sheet Generation Spec]]` — column conventions, gloss rules, status values
