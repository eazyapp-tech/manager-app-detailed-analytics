---
title: DA-02 Collections — Drill Map (Companion doc)
date: 2026-05-19
tags:
  - rentok
  - drill-map
  - collections
  - operator-first
status: Living document · v0.3.1 (round-3 dual-angle audit PASS; small plain-English polish applied — row #40c tooltip text re-translated to operator-plain-English, row #1c jargon-bracket dropped, row #52b code-field reference removed, row #40b ordinal position added, frontmatter row count corrected. Canonical row count: 76.)
owner: Sanchay (PM)
companion_to: DA-02 Brief (v0.7.4), DA-02 Build Sheet V1, DA-02 Collections Detailed Analytics (V3.1 canonical "why" doc)
graph_rev: 2026-05-19 (8080 nodes / 17628 edges)
---

> [!INFO] What this is
> A **PM + designer-readable drill map** for DA-02 Collections. Every tap-able element → existing screen + filters (in plain English). Engineers find the technical wiring below the hard divider.
>
> **Drill map structure (lean per anti-pattern #19):** this map enumerates ONLY (a) tap targets, (b) situational display states affecting operator UX, (c) section visibility rules. Display labels, QA invariants, sort orders, visual specs, color rules live in the Build Sheet.

---

## Scope of this drill map (v0.2)

**In scope — full DA-02 UI surface (10 sections + multi-property §4.5 + empty/error CTAs):**
- §1 Hero & MoM Chip + Efficiency badge
- §2 Period chip + "Has discount" / "No discount" chips (Brief v0.7.4 MUST-SHIP)
- §3 Source Breakdown (4 segments + Adjustments row)
- §4 Population Breakdown
- §4.5 By Property breakdown row (NEW v0.2 — appears between §4 and §5 when multi-property; was missing from v0.1)
- §5 Unlinked Payments Alert (situational)
- §6A Performance Accordion (expanded by default)
- §6B By Category Accordion
- §6C By Payment Mode Accordion (includes RentOk settlement indicator + nudge)
- §6D Received By Accordion (includes concentration callout — permission-gated)
- §7 Collection Trend Chart (with internal time selector + cross-screen drill into DA-01)
- Empty / error state CTAs (Clear Filters, Retry)

**Out of scope — deferred to next session:**
- §8 Worklist Behavior (drill destination, not source — covered in DA-02 worklist spec)
- Pull-to-refresh + data freshness indicator (chrome — same pattern as DA-01)

---

## Ready-when legend

| Marker | Meaning |
|--------|---------|
| ✅ V1 | Works today — frontend just wires it up |
| ⚠️ V1 (second-level CSB-2) | First-level drill works; second-level drill from worklist row → payment detail goes through an unauthenticated endpoint (fix before launch) |
| 🚨 V1 BLOCKED (HB1 + HB6) | Cannot ship until canonical aggregation path locked AND refund-deduction CTE ported to list path — without these, hero ≠ worklist sum |
| 🔧 V1 hot-add | Small backend wiring needed (≤ 1 dev-day per dependency) |
| ⚠️ V1 perm-gate not yet built | Feature works but requires a new JWT permission key that hasn't been added to the seeder yet |

### Acronyms (plain English)
- **CSB-2** — the bill-detail endpoint that operators reach by tapping a payment row in the worklist is currently anonymous-readable. Anyone with the URL gets payment data. Fix before launch.
- **CSB-3** — the Tenant Detail screen and the Receipt PDF endpoint that operators reach from worklist row actions are anonymous-readable. PII exposure.
- **CSB-4** — the refund detail endpoint reached by long-press on the hero ⓘ gross/net breakdown has middleware but skips the inline permission check.
- **HB1** — the homescreen Collections tile uses one query path; this screen uses a different one. They return different numbers. Engineering must pick one canonical aggregation and route both through it. Launch blocker.
- **HB3** — the unusual per-row formula for one payment mode (EazyPG credits) subtracts gateway charges twice in production code but the PRD wording says once. PM + eng must confirm intent before launch.
- **HB4** — the "one staff handling too much cash" callout fires only when all four conditions hold (more than half the cash AND at least 5 cash receipts AND at least ₹10K AND property is at least 60 days old). Exact thresholds non-negotiable.
- **HB5** — the subtitle below the hero number must say "(N Receipts)", not "(N Invoices)". One word, big trust impact.
- **HB6** — the hero on this screen does NOT subtract refunds yet (the widget already does). Until both endpoints subtract refunds the same way, the hero number won't match what the operator can verify by summing the worklist. Launch blocker.
- **viewCashAudit** — a new permission key needed before the concentration callout can render. Owners get it by default; team members get it only if explicitly granted by owner. Not in production today.

---

## Architectural truths (8 — apply to DA-02 + suite-wide where noted)

Each truth has a **plain-English line** (for PM/designer) and a folded **technical detail** (for engineering).

**Truth #1 — DA-02 hero is PERIOD-SENSITIVE (different from DA-01 hero which is always-live).**
- *Plain English:* The big "Net Collections" number changes when the operator picks a different period chip. All cuts below (source bar, population, accordions) inherit the same period. No "(Live)" suffix on the header. Future-dated periods show ₹0 with a helper line.
- *Technical:* Brief line 147 + Build Sheet line 154. Period sent via `start_date`/`end_date` and applied to `p.paid_date::date` per `applyPaidDateRange` at `helpers.ts:335-342`.

**Truth #2 — Hero accuracy gate (HB1 + HB6) is the V1 launch blocker.**
- *Plain English:* Two backend gaps make the hero number unreliable today: HB1 (homescreen tile vs this screen disagree) + HB6 (hero doesn't subtract refunds). Per Brief lines 128-129, the hero MUST match the homescreen Collections tile to the rupee AND must be net of refunds. Without both fixes, the hero displays a number the operator can't verify by summing the worklist. **No-ship.**
- *Technical:* HB1 per Build Sheet line 121 (canonical aggregation pick). HB6 per Build Sheet line 126 (port widget's `ref_agg` CTE pattern into list path at `helpers.ts:60-181`).

**Truth #3 — Adjustment modes (211, 288) and mode 213 are special.**
- *Plain English:* When a deposit or advance is applied to clear a bill (modes 211 + 288), no new cash moves — these show in a separate "Adjustments" row, not in the hero or any source segment (HB2). When a payment uses EazyPG credits (mode 213), the per-row amount subtracts gateway charges TWICE per production code (HB3 — PRD wording disagrees; PM + eng must confirm). Keep verbatim for V1; revisit with payment ops in V1.1.
- *Technical:* HB2 per Build Sheet line 122 (CASE expression at `helpers.ts:423-429`). HB3 per Build Sheet line 123 (mode-213 branch at `helpers.ts:494-504`).

**Truth #4 — MoM chip is INVERTED from Dues (▲ green = up = good for collections).**
- *Plain English:* Per Brief line 208. Designer keep visually distinct from DA-01 Dues so operators don't misread cross-screen.

**Truth #5 — Empty-drill UX contract.**
- *Plain English:* When a drill lands on a destination with zero results, show the active filter context at the top (period + property + filter chip + sort) so the operator sees WHY empty. Copy: "No [type] payments in [period] for [property]."

**Truth #6 — Only one cross-screen drill in §1-§7: §6A → DA-01.**
- *Plain English:* Every drill stays in DA-02 except for three rows in §6A Performance + §7 Trend chart that jump to DA-01 Dues worklist: Invoices Created chip (#19), Partial Payments callout (#26), Trend yellow bar (#48). Back arrow on the destination says "← Collections" (R11). Cross-screen drill destinations carry their own auth review (DA-01 V1 audit covers this).

**Truth #7 — Drill destinations: first-level OK; CSB-2/CSB-3/CSB-4 affect second-level.**
- *Plain English:* When operator taps a DA-02 card or row, they go to the Collections worklist — that endpoint has proper authentication. But the second-level surfaces from worklist rows (payment detail, receipt PDF, tenant profile, refund detail) are currently anonymous-readable. Brief line 133 names this as non-negotiable to fix before V1.
- *Technical:* First-level = `POST /v1/list_screens/collections/list/filters` at `routes.ts:8` (HeaderValidator). Second-level CSB-2 = `/invoices/fetchPaymentSettlementDetails` at `routes/invoices.ts:204`. CSB-3 = `/tenant/getTenantData` at `routes/tenant.ts:927` + `/invoices/generateReceipt` at `routes/invoices.ts:216`. CSB-4 = refund detail lacks inline `checkAuth(viewInvoices)`. All Jatin gate P0.

**Truth #8 — Permission-driven action hiding affects every worklist drill row.**
- *Plain English:* Every "Primary row action" listed in the table (View Receipt, View Tenant, WhatsApp, Call, Excel Export) is HIDDEN — not greyed out — when the operator's role lacks the corresponding permission. Same for the §6D concentration callout (#45) which requires a NEW `viewCashAudit` JWT key that's not yet in the seeder. The drill itself still works; only the per-row action buttons / callout hide.
- *Technical:* Permission keys: `viewInvoices`, `recordPayment`, `viewTenants`, `money_reports`, `viewCashAudit` (new — owners bypass via `isOwner` at `commonFunctions.ts:1244`).

---

## ⚠️ Pending designer decisions

- **Hero ⓘ icon convention drift** (Brief vs Build Sheet) — Brief line 213 says single-tap → bottom sheet (R1 strict). Build Sheet line 167 says tap = inline tooltip, long-press = bottom sheet with gross/refunds/net breakdown. Pick one. Affects rows #2, #4a.
- **Row #26 "Others" tap behavior** (§6B By Category) — direct drill OR expand-then-drill? Build Sheet line 284 says direct drill; DA-01 §8 pattern was expand-then-drill. Pick one for consistency.
- **"Has discount" chip filter code allocation** (rows #6, #7) — next free integer after 1219 (where REFUND=1209 is already used). Day-3 cut-line per Brief.

---

# DA-02 Drill Map — tap-relevant rows only (10 of 10 UI sections + §4.5)

> **How to read the "Filters applied" column:** the **Primary row action** at the bottom of each filter cell is shown on each row of the destination worklist — but **hidden if the operator lacks permission** (Truth #8).
>
> **DA-02 vs DA-01 key differences:** Hero is period-sensitive (not always-live); MoM inverted (up = good); Hero block ENTIRE tappable (not just the ⓘ icon); Performance accordion expanded by default (not collapsed); multiple cross-screen drills into DA-01.
>
> **All hero-level numbers are 🚨 V1 BLOCKED on HB1 + HB6** until canonical aggregation + refund CTE land. First-level drills work, but the displayed numbers won't match the worklist sum until then.

| # | Tap on… | Goes to (existing screen) | Filters applied | Ready when? |
|---|---------|--------------------------|-----------------|-------------|
| **§1 HERO + MoM CHIP + EFFICIENCY BADGE** | | | | |
| | *Also rendered (display only, not tappable): receipt count subtitle "(N Receipts)" [HB5 — not "(N Invoices)"], "Net Collections" GAAP label, multi-property suffix, "Collections" header [no "(Live)" suffix], MoM color rule (▲ green = up = good), MoM visibility rule (hidden when account < 30d OR prior period = ₹0), Efficiency color thresholds (≥80% green / 50-79% amber / <50% red, >100% allowed and stays green), Hero reconciliation invariant.* | | | |
| 1 | Hero block (entire region — big number, subtitle, GAAP label all tappable) | Collections worklist | • Show: all payments in selected period<br>• Period: selected from chip<br>• Property: selected<br>• Sort: most recent first<br>• Header: "[Period] Collections · X Receipts · ₹Y"<br>• **Primary row actions: View Receipt / View Tenant / WhatsApp / Call** (permission-gated — Truth #8) | 🚨 V1 BLOCKED (HB1 + HB6) → ⚠️ second-level CSB-2 once unblocked |
| 1a | Hero negative-net state ("Net Collection: −₹X" with red styling) | — (situational display: when refunds exceed gross in the period, hero shows negative with red styling and tooltip clarifies gross vs net. Per Analytics PRD EC-02 line 562 + Build Sheet line 413.) | — | — |
| 1b | Hero all-adjustments ₹0 state | — (situational display: when ALL payments in the period are mode 211/288 paper transfers, hero shows ₹0 because adjustments are excluded; tooltip explains. Per Analytics PRD EC-03 line 565.) | — | — |
| 1c | Hero "(Limited view)" badge | — (situational display: when an operator can only see invoices for tenants they personally added, header shows "(Limited view)" badge and totals reflect the subset. Per Analytics PRD EC-08 line 580.) | — | — |
| 1d | Hero empty state ("No payments recorded in this period.") | — (situational display: when period has zero collections, hero shows ₹0 with this copy. Per Build Sheet line 462 microcopy.) | — | — |
| 2 | The "ⓘ" icon next to "Collections" header | Stays on DA-02 — opens explainer (designer decision pending — see top of doc) | • Per pending decision: strict R1 (single-tap → bottom sheet) OR DA-02 exception (single-tap → tooltip; long-press → bottom sheet with Gross / Refunds / Net breakdown) | ✅ V1 (pending designer decision) |
| 3 | MoM chip (▲/▼ % vs prior same-elapsed-days last month) | Stays on DA-02 — opens inline tooltip showing prior-period numbers + comparison day-range | • In-screen tooltip reveal | ✅ V1 |
| 4 | Efficiency badge ("Collection Efficiency: 83%") inline below hero | Stays on DA-02 — opens tooltip with Collectible / Collected / Efficiency breakdown. When >100%, tooltip expands with current dues / old dues recovery / advance breakdown | • In-screen tooltip reveal | ✅ V1 (denominator query is NEW BUILD) |
| 4a | Hero ⓘ long-press → full Gross / Refunds / Net bottom sheet | Stays on DA-02 — opens bottom sheet (conditional on designer decision per top of doc) | • In-screen reveal (only if DA-02 exception adopted; else folded into #2) | ✅ V1 (conditional) |
| **§2 PERIOD CHIP + DISCOUNT CHIPS** (top-of-screen filter row) | | | | |
| 5 | Period chip ("This Month" default; tap opens period picker: This Month / Last Month / Last 6M / Custom range) | Stays on DA-02 — opens period picker modal; on selection, ALL sections below the chip re-fetch with the new period | • In-screen modal reveal | ✅ V1 |
| 5a | Future-range helper text ("Collections are based on received payments. Future dates show ₹0.") | — (situational display: appears below hero when operator picks a custom range extending into the future. Per Build Sheet line 188 + Analytics PRD EC-14.) | — | — |
| 5b | (Period picker does NOT offer "Today" option) | — (render rule: period picker offers This Month / Last Month / Last 6M / Custom only. Per Brief line 147 — Collections is period-based, not always-live — there is no "today live" mode. Different from DA-01 which has a Today filter. Truth #1.) | — | — |
| 6 | "Has discount" chip (NEW V1 MUST-SHIP per Brief v0.7.4) | Re-filters DA-02 in place (all sections below re-fetch — hero, source, population, accordions all recompute) | • Show only: payments that have at least one discount applied (a credit attached)<br>• Period: inherited from period chip<br>• Removable chip "[Has discount ✕]" visible in §2 chip row | ✅ V1 (depends on backend filter code allocation by Day 3) |
| 7 | "No discount" chip (NEW V1 MUST-SHIP — peer toggle to #6, not tri-state) | Re-filters DA-02 in place — inverse of #6 | • Show only: payments with NO discount applied<br>• Period: inherited<br>• Removable chip "[No discount ✕]" | ✅ V1 (same dependency as #6) |
| 7a | Chip pre-applied state when arriving from DA-05 (Discounts) drill | — (situational display: when operator lands on DA-02 via cross-screen drill from DA-05 with "Has discount" chip pre-applied, chip state must be visibly clear and easy to clear. Per Brief line 141.) | — | — |
| **§3 SOURCE BREAKDOWN** (4 segments + Adjustments row; period-sensitive; **each segment has TWO tap targets: bar segment AND chip below it; chip is canonical 44pt min hit area per Build Sheet line 207**) | | | | |
| | *Also rendered (display only): per-segment chips with label + amount, period-aware label (e.g., "[Apr]'s Dues" in This Month mode flips to "Period Dues" in Last 6M / Custom), Source reconciliation invariant.* | | | |
| 8 | "[Month]'s Dues" segment OR its chip — payments against bills due in current period | Collections worklist | • Show only: payments against bills with due date in the selected period<br>• Adjustments excluded<br>• Period: selected<br>• Property: selected<br>• Sort: most recent first<br>• Header: "[Month] Dues Collected · X Receipts · ₹Y"<br>• **Primary row actions: View Receipt / View Tenant / WhatsApp / Call** | ⚠️ V1 (second-level CSB-2) |
| 9 | "Old Dues" segment OR its chip — payments against bills from prior periods (recovery) | Collections worklist | • Show only: payments against bills with due date BEFORE the selected period<br>• Adjustments excluded<br>• Period: selected<br>• Property: selected<br>• Sort: most recent first<br>• Header: "Old Dues Collected · X Receipts · ₹Y" | ⚠️ V1 (second-level CSB-2) |
| 10 | "Advance" segment OR its chip — advance/prepaid amounts not yet applied | Collections worklist | • Show only: advance payments (not yet applied to a specific bill)<br>• Adjustments excluded<br>• Period: selected<br>• Property: selected<br>• Sort: most recent first<br>• Header: "Advance Received · X Receipts · ₹Y" | ⚠️ V1 (second-level CSB-2) |
| 11 | "Paid Early" segment OR its chip — payments against bills due in future periods | Collections worklist | • Show only: payments against bills with due date AFTER the selected period (tenant paid before bill was due)<br>• Adjustments excluded<br>• Period: selected<br>• Property: selected<br>• Sort: most recent first<br>• Header: "Paid Early · X Receipts · ₹Y" | ⚠️ V1 (second-level CSB-2) |
| 12 | "Adjustments" row tap (below the Source bar — Truth #3) | Collections worklist | • Show only: paper-transfer entries (deposit-applied OR advance-applied — no new cash moved)<br>• Period: selected<br>• Property: selected<br>• Sort: most recent first<br>• Header: "Adjustments · X Receipts · ₹Y" | ⚠️ V1 (second-level CSB-2) |
| 12a | Per-segment long-press (any of #8-#11) | Stays on DA-02 — opens source-specific values tooltip per Analytics PRD line 480 | • In-screen tooltip reveal | ✅ V1 |
| **§4 POPULATION BREAKDOWN** (3 rows; period-sensitive — population grouping at payment time, within the period) | | | | |
| | *Also rendered (display only): per-row count + amount label, Population reconciliation invariant (Active + Booking + Old = hero, excludes unlinked which surface separately in §5).* | | | |
| 13 | "Active Tenants" row — payments from currently checked-in tenants | Collections worklist | • Show only: payments from active tenants<br>• Period: selected<br>• Property: selected<br>• Sort: most recent first<br>• Header: "Active Tenants · X Receipts · ₹Y"<br>• **Primary row actions: View Receipt / View Tenant / WhatsApp / Call** | ⚠️ V1 (second-level CSB-2) |
| 14 | "Booking" row — payments from confirmed future move-ins | Collections worklist | • Show only: payments from booking tenants<br>• Period: selected<br>• Property: selected<br>• Sort: most recent first<br>• Header: "Bookings · X Receipts · ₹Y" | ⚠️ V1 (second-level CSB-2) |
| 15 | "Old Tenants" row — payments from moved-out tenants (e.g., deposit adjustments) | Collections worklist | • Show only: payments from moved-out tenants<br>• Period: selected<br>• Property: selected<br>• Sort: most recent first<br>• Header: "Old Tenants · X Receipts · ₹Y" | ⚠️ V1 (second-level CSB-2) |
| **§4.5 BY PROPERTY BREAKDOWN — MULTI-PROPERTY ONLY** (NEW v0.2 — was missing in v0.1; visible only when `selected_properties > 1`; appears between §4 Population and §5 Unlinked Alert per Analytics PRD EC-07 line 577) | | | | |
| 15a | (Section visibility — entire §4.5 hidden when single-property dashboard) | — (render rule: visible only when operator has more than one property selected. Per Analytics PRD EC-07 line 577 + Build Sheet line 418.) | — | — |
| 16 | Per-property row (e.g., "Sunshine PG · ₹X · share %") | Collections worklist — **scopes to single property** regardless of multi-property dashboard scope (R7 in PRD) | • Property: scoped to single property (OVERRIDES dashboard multi-prop scope)<br>• Period: selected<br>• Sort: most recent first<br>• Header: "[Property Name] · X Receipts · ₹Y"<br>• Removable filter chip "[Property: X ✕]"<br>• Bypasses property grouping in the worklist<br>• **Primary row actions: View Receipt / View Tenant / WhatsApp / Call** | ⚠️ V1 (second-level CSB-2) |
| **§5 UNLINKED PAYMENTS ALERT** (situational — renders only when at least 1 payment in period has no tenant attached) | | | | |
| 17 | (Section visibility — Alert hidden when zero unlinked payments in period) | — (render rule: appears only when at least 1 payment has no tenant attributed. Per Build Sheet line 241.) | — | — |
| 18 | "Link Payment" CTA on alert card | Full-screen push — opens 3-step Link Payment flow: (1) list of unlinked payments, (2) search candidate tenant by name/phone/room, (3) confirm linking | • FS push, NOT a worklist drill<br>• Each step is a separate frame in back-stack (R2 universal)<br>• On confirm, payment attributed; alert dismisses if no more unlinked<br>• **Action permission: `record_payment`** (permission-gated — Truth #8) | ✅ V1 (Link Payment flow exists; 3-step UI is BUILD) |
| 18a | "All dues collected!" success banner state | — (situational display: when collection efficiency hits 100% [within ±0.5% rounding], a green success banner replaces the Unlinked Alert area or sits above it: "All dues collected! Everything is up to date." Per Analytics PRD EC-13 line 604 + Build Sheet line 424.) | — | — |
| **§6A PERFORMANCE ACCORDION** (period-sensitive; ALL metrics use "due date in period", NOT "created in period"; **EXPANDED by default — different from DA-01 §7 which is collapsed**) | | | | |
| | *Also rendered (display only): summary sentence "₹X collected between [start] → [end]", inline progress bar (Collectible full width with Collected overlaid), Efficiency color thresholds, Performance reconciliation invariants. Accordion expand-state preserves on back from drill per Build Sheet line 472.* | | | |
| 19 | Performance accordion header tap (expand/collapse) | Stays on DA-02 — toggles the accordion (default: EXPANDED) | • In-screen reveal | ✅ V1 |
| 20 | "Invoices Created" chip — total billed in selected period | **DA-01 Dues worklist** (CROSS-SCREEN per Truth #6) | • Show: ALL bills (paid + unpaid) with due date in selected period<br>• Period: passed to DA-01<br>• Property: selected<br>• Sort: due date ascending<br>• Header: "[Period] Invoices Created · X Bills · ₹Y"<br>• Back arrow: "← Collections" (R11)<br>• **Primary row action on DA-01: View Bill Detail** | ⚠️ V1 (cross-screen to DA-01; second-level CSB-2 on DA-01) |
| 21 | "Collection" chip (same value as Hero #1 — restated for side-by-side comparison) | Collections worklist (same drill as Hero #1) | • Same filter set as Hero #1 | 🚨 V1 BLOCKED (HB1 + HB6 — same as Hero #1) |
| 22 | Collection Efficiency row tap (or its ⓘ tooltip) | Stays on DA-02 — opens tooltip with Collectible / Collected / Efficiency breakdown | • In-screen tooltip reveal | ✅ V1 |
| 23 | "On Time" timing row — paid on the same day the bill was due | Collections worklist | • Show only: payments where the paid date matches the earliest bill's due date (for multi-invoice payments, oldest obligation wins)<br>• Period: selected<br>• Property: selected<br>• Sort: most recent first<br>• Header: "On Time · X Receipts · ₹Y" | ⚠️ V1 (second-level CSB-2) |
| 24 | "Early" timing row — paid before the bill was due | Collections worklist | • Show only: payments paid before the earliest bill's due date<br>• Period: selected<br>• Removable chip "[Paid before due ✕]" visible<br>• Sort: most recent first<br>• Header: "Early · X Receipts · ₹Y" | ⚠️ V1 (second-level CSB-2) |
| 25 | "Late" timing row — paid after the bill was due | Collections worklist | • Show only: payments paid after the earliest bill's due date (arrear-clearing payments correctly classified Late)<br>• Period: selected<br>• Sort: most recent first<br>• Header: "Late Payments · X Receipts · ₹Y" | ⚠️ V1 (second-level CSB-2) |
| 26 | "Adjustments" timing row — paper transfers not timing-classifiable | Collections worklist | • Show only: deposit-applied or advance-applied paper transfers (no due-date timing applies)<br>• Period: selected<br>• Sort: most recent first<br>• Header: "Adjustments · X Receipts · ₹Y" | ⚠️ V1 (second-level CSB-2) |
| 27 | "Partial Payments" callout ("₹X across N invoices still have outstanding balance after this payment.") | **DA-01 Dues worklist** (CROSS-SCREEN to DA-01) | • Show only: invoices that were paid in selected period AND still have outstanding balance<br>• Period: passed to DA-01<br>• Property: selected<br>• Removable chip "[Partial payments]"<br>• Sort: most overdue first<br>• Header: "Partial Payments · X Bills · ₹Y"<br>• Back arrow: "← Collections" (R11) | ⚠️ V1 (cross-screen to DA-01; second-level CSB-2 on DA-01) |
| 28 | Per-metric ⓘ tooltips × 3 (Collectible / Efficiency / Timing — each independently tappable) | Stays on DA-02 — opens inline tooltip per metric | • In-screen reveal | ✅ V1 |
| **§6B BY CATEGORY ACCORDION** (period-sensitive — "Collected" uses paid date in period; "Billed" uses due date in period; collapsed by default; top categories ranked by collected amount with rest in "Others") | | | | |
| | *Also rendered (display only): per-row progress bar (Collected / Billed %), Category sort order (biggest Collected first), Category reconciliation invariant.* | | | |
| 29 | By Category accordion header tap (expand/collapse) | Stays on DA-02 — toggles the accordion | • In-screen reveal | ✅ V1 |
| 30 | Category row tap (one per bill type — Rent, Electricity, Food, Security Deposit, etc.) | Collections worklist | • Show only: payments against bills in the tapped category<br>• Period: selected (Collected uses paid date)<br>• Property: selected<br>• Sort: most recent first<br>• Header: "[Category] Collected · X Receipts · ₹Y" | ⚠️ V1 (second-level CSB-2) |
| 31 | "Others" row tap (6th category onwards aggregated) — pending designer decision: direct drill OR expand-then-drill | Collections worklist (after expand if expand-then-drill pattern adopted) | • Show only: payments in non-top-5 categories (Caution Money, Maintenance, etc.)<br>• Period: selected<br>• Sort: most recent first<br>• Header: "Others · X Receipts · ₹Y" | ⚠️ V1 (designer confirm pattern) |
| 32 | Per-row ⓘ tooltip | Stays on DA-02 — opens tooltip: "Collected: amount received this period. Billed: amount invoiced this period. Progress = Collected ÷ Billed." | • In-screen reveal | ✅ V1 |
| 32a (NEW v0.3) | EC-09 worklist row count caveat | — (situational note: when one payment covers multiple bills across different categories, the worklist shows one row per payment but the category total uses sum-of-invoices. So worklist row count won't always equal sum-of-category-amounts ÷ avg row. Per Analytics PRD EC-09 line 583 + Build Sheet EC-09 line 420.) | — | — |
| **§6C BY PAYMENT MODE ACCORDION** (period-sensitive; collapsed by default; RentOk + Bank + UPI + Cash + Others as main rows; deposit-applied + advance-applied separator rows below; settlement indicator on RentOk row) | | | | |
| | *Also rendered (display only): Verified Digital % summary ("RentOk: X% · Manual Entry: Y%"), Mode reconciliation invariant.* | | | |
| 33 | By Payment Mode accordion header tap (expand/collapse) | Stays on DA-02 — toggles the accordion | • In-screen reveal | ✅ V1 |
| 34 | RentOk (Online) row body tap — RentOk payment gateway | Collections worklist | • Show only: payments via RentOk gateway (UPI/cards/netbanking through RentOk)<br>• Period: selected<br>• Property: selected<br>• Sort: most recent first<br>• Header: "RentOk · X Receipts · ₹Y" | ⚠️ V1 (second-level CSB-2) |
| 34a | "Unsettled" inline indicator on RentOk row (situational — shown only when payout missing for any RentOk payment in period) | Collections worklist | • Show only: RentOk payments with no UTR (not yet settled)<br>• Period: selected<br>• Sort: most recent first<br>• Header: "Unsettled · X Receipts · ₹Y"<br>• Note: separate tap target from row body #34 (anti-pattern #17 dual tap) | ⚠️ V1 (second-level CSB-2) |
| 34b | "Unsettled — Link Bank A/C" panel state (situational — when operator's bank account isn't linked to receive RentOk payouts) | — (situational display: settlement panel shows "Unsettled — Link Bank A/C" on RentOk row; UTR field reads "Link Bank A/C to receive the payment." Per Analytics PRD EC-06 line 574.) | — | — |
| 35 | Bank Transfer row tap — NEFT/IMPS direct transfers | Collections worklist | • Show only: payments via direct bank transfer<br>• Period: selected<br>• Sort: most recent first<br>• Header: "Bank Transfer · X Receipts · ₹Y" | ⚠️ V1 (second-level CSB-2) |
| 36 | UPI row tap — third-party UPI apps | Collections worklist | • Show only: payments via third-party UPI (not RentOk gateway)<br>• Period: selected<br>• Sort: most recent first<br>• Header: "UPI · X Receipts · ₹Y" | ⚠️ V1 (second-level CSB-2) |
| 37 | Cash row tap — physical cash | Collections worklist | • Show only: cash payments<br>• Period: selected<br>• Sort: most recent first<br>• Header: "Cash · X Receipts · ₹Y" | ⚠️ V1 (second-level CSB-2) |
| 38 | Others row tap — cheque/DD and other modes | Collections worklist | • Show only: cheque, DD, and other payment modes (not in main 4 categories)<br>• Period: selected<br>• Sort: most recent first<br>• Header: "Others · X Receipts · ₹Y" | ⚠️ V1 (second-level CSB-2) |
| 39 | Adjusted from Deposit row tap — separator row (NOT in main mode total) | Collections worklist | • Show only: deposit-applied paper transfers<br>• Period: selected<br>• Sort: most recent first<br>• Header: "Adjusted from Deposit · X Receipts · ₹Y" | ⚠️ V1 (second-level CSB-2) |
| 40 | Advance Adjusted row tap — separator row (NOT in main mode total) | Collections worklist | • Show only: advance-applied paper transfers<br>• Period: selected<br>• Sort: most recent first<br>• Header: "Advance Adjusted · X Receipts · ₹Y" | ⚠️ V1 (second-level CSB-2) |
| 40b (NEW v0.3) | EazyPG Credits row tap — separate row sitting between Cash (#37) and Others (#38) in the §6C list, NOT folded into Others | Collections worklist | • Show only: payments made via EazyPG credits<br>• Period: selected<br>• Sort: most recent first<br>• Header: "EazyPG Credits · X Receipts · ₹Y"<br>• **Note:** per-row amount uses the special HB3 formula — Truth #3 | ⚠️ V1 (second-level CSB-2) |
| 40c (NEW v0.3) | ⓘ tooltip on EazyPG Credits row — explains the credit-net math (Brief line 137 MUST-SHIP) | Stays on DA-02 — opens inline tooltip with operator-plain-English: "EazyPG Credit payments are shown after RentOk's processing fee. The amount you see reflects the net credit applied — slightly lower than the gross billed." | • In-screen reveal | ✅ V1 (microcopy: NEW BUILD per Brief line 137) |
| 40a | Performance nudge — ✨ ("More tenants paying digitally") OR 💡 ("Push RentOk Pay links to reduce manual recording") | — (situational display: shown below mode rows when RentOk share changes vs prior period — ✨ if up, 💡 if down. Per Build Sheet line 309 + microcopy lines 452-453.) | — | — |
| **§6D RECEIVED BY ACCORDION** (period-sensitive; collapsed by default; RentOk system row first; staff sorted by amount; "Others (N staff)" collapse for > 4 staff; concentration callout permission-gated) | | | | |
| | *Also rendered (display only): per-row avatar + name + amount + receipt count, trophy badge on RentOk row if top, Received By sort order (RentOk first then staff by amount DESC), Received By reconciliation invariant.* | | | |
| 41 | Received By accordion header tap (expand/collapse) | Stays on DA-02 — toggles the accordion | • In-screen reveal | ✅ V1 |
| 42 | RentOk system row tap — online self-payments by tenants | Collections worklist | • Show only: payments where no staff recorded the payment (tenant self-paid via RentOk gateway)<br>• Period: selected<br>• Property: selected<br>• Sort: most recent first<br>• Header: "By RentOk · X Receipts · ₹Y" | ⚠️ V1 (second-level CSB-2) |
| 43 | Per-staff row tap (one per team member who recorded payments) | Collections worklist | • Show only: payments recorded by the tapped team member<br>• Period: selected<br>• Sort: most recent first<br>• Header: "By [Name] · X Receipts · ₹Y" | ⚠️ V1 (second-level CSB-2) |
| 44 | "Others (N staff)" row tap (situational — appears when > 4 staff) | Stays on DA-02 — in-place expansion to reveal staff rows 5+ | • In-screen reveal (no drill until specific staff row tapped) | ✅ V1 |
| 45 | Concentration callout banner tap ("⚠️ [Name] recorded X% of cash this month (₹Y across N receipts). Worth a check-in.") | Collections worklist | • Show only: payments recorded by the named staff<br>• Period: selected<br>• Sort: by AMOUNT DESC (different from other Received By drills which sort by paid date)<br>• Header: "By [Name] · X Receipts · ₹Y"<br>• **Permission gate: `viewCashAudit` JWT key (NEW — not yet in seeder; owners bypass via isOwner)** | ⚠️ V1 perm-gate not yet built + ⚠️ second-level CSB-2 |
| | 💡 **Note (concentration callout — 3 stacked dependencies):** <br>**(a) HB4 4-condition trigger** — banner fires ONLY when ALL of [staff > 50% of cash, ≥5 cash receipts, ≥₹10K, property ≥60 days old]. Newness bypass + maturity gate prevent false-positives on new properties or sparse data. <br>**(b) `viewCashAudit` JWT permission key** — NEW; not yet in production seeder; owners get by default via isOwner check; team members get only if owner explicitly grants. ~0.5 dev-day to add. <br>**(c) Drill sort exception** — uses amount DESC (not paid date DESC like other Received By rows) so operator sees the largest recorded payments first. | | | |
| 45a | Concentration callout visibility (4 trigger conditions per HB4 + permission gate) | — (render rule: callout renders ONLY when all 4 HB4 conditions hold AND operator's role has `viewCashAudit` permission. Per Build Sheet line 331 + line 124.) | — | — |
| 45b (NEW v0.3) | Concentration callout 100% copy variant | — (situational display: when a single staff member recorded 100% of the cash in the period [EC-12], the callout copy adjusts to "⚠️ [Name] recorded 100% of cash this month (₹Y across N receipts). Worth a check-in." Per Analytics PRD line 602.) | — | — |
| 46 | Per-row ⓘ tooltip | Stays on DA-02 — opens tooltip: "Number of receipts recorded by this person. One receipt may cover multiple invoices." | • In-screen reveal | ✅ V1 |
| **§7 COLLECTION TREND CHART** (always visible; has INTERNAL time selector "This Month" / "6M" independent of dashboard period; yellow bar = Dues raised; green bar = Collections stacked by 4 source segments) | | | | |
| | *Also rendered (display only): 4-item legend (Dues · Collection · Old Dues · Advance · Paid Early), Trend reconciliation invariant.* | | | |
| 47 | Yellow bar single-tap (any month) — Dues raised | Stays on DA-02 — opens inline tooltip with exact dues-raised amount + "View [month] →" CTA | • In-screen tooltip reveal (per Universal Rule R12) | ✅ V1 |
| 48 | Yellow bar "View [month] →" CTA inside tooltip — CROSS-SCREEN drill | **DA-01 Dues worklist** (CROSS-SCREEN) | • Show: ALL bills with due date in the tapped month<br>• Period: chart's month (OVERRIDES dashboard period for the descendant drill)<br>• Property: inherited<br>• Header: "[Month] Invoices Due · X Bills · ₹Y"<br>• Back arrow: "← From DA-02 Collections" (R11 — breadcrumb) | ⚠️ V1 (cross-screen to DA-01; second-level CSB-2 on DA-01) |
| 49 | Green bar single-tap (any month) — Collections stacked | Stays on DA-02 — opens inline tooltip with exact totals + "View [month] →" CTA | • In-screen tooltip reveal | ✅ V1 |
| 50 | Green bar "View [month] →" CTA inside tooltip | Collections worklist (same screen but month override) | • Show: all payments in the tapped month<br>• Period: chart's month (OVERRIDES dashboard period)<br>• Property: selected<br>• Sort: most recent first<br>• Header: "[Month] Collections · X Receipts · ₹Y" | ⚠️ V1 (second-level CSB-2) |
| 51 | Stacked sub-segment tap (within green bar — e.g., Old Dues sub-segment) | Collections worklist (month + sub-segment) | • Show: payments matching the source sub-segment (same logic as §3 segments #8-#11) AND in the tapped month<br>• Period: chart's month override<br>• Property: selected<br>• Sort: most recent first<br>• Header: "[Month] [Sub-segment]" | ⚠️ V1 (second-level CSB-2) |
| 52 | Time selector toggle ("This Month" / "6M" — internal chart control) | Stays on DA-02 — chart re-renders with new time bucket; dashboard hero unaffected | • In-screen control | ✅ V1 |
| 52a | Trend insight text ("Outstanding trending down over last 2 months." OR "Collections below billing for 3 months. Outstanding may be building up.") | — (situational display: shown below chart when pattern is detectable. Per Build Sheet line 353 + microcopy lines 458-459.) | — | — |
| 52b (NEW v0.3) | Trend chart independence when dashboard period spans calendar months (EC-15) | — (render rule: chart uses its own internal selector independent of dashboard period. When the dashboard period spans more than 1 calendar month [e.g., "Last 6M" / Custom range], source segments still bucket correctly by the month the payment was received. Per Build Sheet EC-15 line 426 + Analytics PRD EC-15 line 606.) | — | — |
| **EMPTY / ERROR STATE CTAs** (cross-cutting — apply across §1-§7) | | | | |
| 53 | "Clear Filters" CTA on worklist empty state ("No payments match your current filters.") | Re-fetches worklist with all filter chips cleared | • In-screen reveal — chips cleared, list refreshes<br>• Per Build Sheet line 463 + Analytics PRD line 622 | ✅ V1 |
| 54 | "Retry" CTA on network failure error state ("Couldn't load collections. Check your connection.") | Re-fetches all sections in parallel | • In-screen reveal<br>• Per Build Sheet line 464 + Analytics PRD line 631 | ✅ V1 |
| 55 | "Retry" CTA on per-section failure ("Couldn't load this section.") | Re-fetches the failed accordion / section only | • In-screen reveal — only the failed section refetches<br>• Per Build Sheet line 465 + Analytics PRD line 632 | ✅ V1 |

---

## Status summary (full DA-02 audit, 10 of 10 UI sections + §4.5 + empty/error CTAs)

**Total tap-relevant rows: 76** (each row appears in exactly ONE bucket; 💡 Note rows are sub-row annotations and not counted)

| Status | Count | Row IDs |
|--------|-------|-------|
| ✅ V1 (in-screen reveal: tooltip / BS / accordion header / period picker / chip toggle / sub-flow / CTA) | 24 | #2, #3, #4, #4a, #5, #12a, #18, #19, #22, #28, #29, #31, #32, #33, #40c, #41, #44, #46, #47, #49, #52, #53, #54, #55 |
| ✅ V1 (depends on backend filter code allocation by Day 3) | 2 | #6, #7 |
| ⚠️ V1 (first-level drill OK, ⚠️ second-level CSB-2) | 27 | #8, #9, #10, #11, #12, #13, #14, #15, #16, #23, #24, #25, #26, #30, #34, #34a, #35, #36, #37, #38, #39, #40, #40b, #42, #43, #50, #51 |
| ⚠️ V1 (cross-screen to DA-01 — adds CSB-2 risk on DA-01 second-level drill) | 3 | #20, #27, #48 |
| 🚨 V1 BLOCKED (HB1 + HB6) | 2 | #1, #21 |
| ⚠️ V1 perm-gate not yet built (`viewCashAudit`) + ⚠️ second-level CSB-2 | 1 | #45 |
| Situational display states (operator-visible, no tap) | 12 | #1a, #1b, #1c, #1d, #5a, #7a, #18a, #32a, #34b, #40a, #45b, #52a |
| Section visibility / render rules | 5 | #5b (Today not offered), #15a (§4.5 multi-property only), #17 (§5 Unlinked Alert), #45a (Concentration callout), #52b (Trend chart independence across calendar months) |

**Sum:** 24 + 2 + 27 + 3 + 2 + 1 + 12 + 5 = **76** ✓

**Critical pending decisions (10 blockers):**
1. **HB1 canonical aggregation path** — Jatin picks invoice-first OR payment-first; route widget AND list endpoints through it.
2. **HB6 refund-deduction CTE in list path** — port widget's `ref_agg` CTE pattern. Blocks Hero #1 + Collection chip #21.
3. **HB3 Mode 213 formula** — confirm gateway_charges subtracted twice (production) vs once (PRD wording). Jatin + PM gate.
4. **CSB-1 / CSB-2 / CSB-3 / CSB-4 auth fixes** — all 4 destinations need HeaderValidator + checkAuth.
5. **Hero ⓘ icon convention** — designer picks R1 strict (single-tap → BS only) OR DA-02 exception (tap → tooltip + long-press → BS).
6. **`viewCashAudit` JWT key** — engineering adds to seeder + frontend permission map (~0.5 dev-day).
7. **Filter code allocation for "Has discount" / "No discount" chips** — next free integer (genuinely 1220 unless 1288/12881 typos renumbered).
8. **§6B "Others" tap behavior** — direct drill OR expand-then-drill? Pick one for consistency with DA-01.
9. **HB4 concentration calibration** — 4 conditions enforced exactly.
10. **HB5 "Receipts" subtitle** — replace any "(N Invoices)" wording in Figma with "(N Receipts)."

---

---

# Engineering Reference (technical specifics per row)

## Verification protocol applied (v0.2)

v0.2 = v0.1 + round-1 three-angle audit corrections. All 19 anti-patterns from DA-01 work applied from kickoff. Personal PM verification on all code line refs after round-1 sub-agent caught drift.

**Corrections applied vs v0.1:**
1. **10+ code line refs corrected:** `applyReceivedBy: 370-378` (was 370-372); `applyTenantTypes: 380-388` (was 374-382); `applyOrderBy: 390-403, default at 401` (was 389); `executePaginated: 407-449` (was 394-436); mode 211/288 CASE expression: `helpers.ts:423-429` (was 410-416); `formatRow: 453-605` (was 440-489); mode-213 branch: `helpers.ts:494-504` (was 481-489); `applyPaidDateRange: 335-342` (was 337-341); `Credits.payment_id: entities/credits.ts:68` (was 67); `commonFunctions.checkAuth isOwner bypass: line 1244` (was 1247).
2. **Tenant resolution priority CORRECTED:** code at `helpers.ts:164` ranks `status=2` (booking) FIRST, then `status=1` (active), then `status=0` (moved-out), then `status=5` (perma-deleted). v0.1 narrative was reversed.
3. **Build Sheet line ref attribution corrected:** lines 480, 483, 485 are in **Analytics PRD** (Per-Spec Specifics post-orphan-audit), NOT Build Sheet (which has smoke-test curls there). Cited correctly in v0.2.
4. **Status summary math:** v0.1 claimed 56 = 50+4+2 but actually 59 with double-counting. v0.2 = 65 rows with explicit reconciliation; no self-contradicting parentheticals.
5. **SQL stripped from PM-facing filter cells:** all `mode IN (...)`, `tenant_types: [N]`, `payment_owner_credits`, `MIN(inv.due_date)`, etc. translated to plain English; tech specifics quarantined to Eng Reference.
6. **11 Truths folded → 8:** combined HB1 + HB6 (Truth #2 Hero accuracy gate); combined mode 211/288 + mode 213 (Truth #3 Special payment modes); converted Truth #11 ⓘ drift to "Pending Designer Decision" callout (not architectural).
7. **EC-07 By Property breakdown — NEW §4.5 added** (was missing entire sub-section in v0.1 — critical coverage gap).
8. **5 high-severity situational states added:** #1a negative net (EC-02), #1b all-adjustments (EC-03), #1c Limited view (EC-08), #1d empty state (EC-01), #18a success banner (EC-13), #34b "Link Bank A/C" (EC-06).
9. **3 empty/error CTAs added:** #53 Clear Filters, #54 network Retry, #55 section Retry.
10. **Acronym section trimmed:** plain-English meaning only; file paths and CTE names moved to Eng Reference and individual Truth technical details.

---

## Per-row engineering details

### Row #1 — Hero block tap
**First-level endpoint:** `POST /v1/list_screens/collections/list/filters` at `routes.ts:8` (HeaderValidator-protected).
**Hero formula (PRD intent):** `Net Collected = Σ raw_total (excl. modes 211/288, mode 213 special formula) − Σ refunds_in_period`. **Drift:** production list at `helpers.ts:60-181` does NOT subtract refunds today (HB6); widget has the CTE, list doesn't.
**Second-level CSB-2:** worklist row → payment detail = `/invoices/fetchPaymentSettlementDetails` at `routes/invoices.ts:204` (zero middleware).

### Rows #1a-#1d — Hero situational states
- #1a EC-02 negative net: Analytics PRD line 562 + Build Sheet line 413.
- #1b EC-03 all-adjustments ₹0: Analytics PRD line 565 + Build Sheet line 414.
- #1c EC-08 "(Limited view)" badge: Analytics PRD line 580 + Build Sheet line 419. Triggered by `checkAuthInDb(view_invoices_of_self_added_tenants)` = true AND `viewInvoices` = false.
- #1d EC-01 empty state copy: Build Sheet line 462 microcopy + Analytics PRD line 621.

### Row #2 + #4a — Hero ⓘ convention drift (Designer Decision)
Brief line 213: "ⓘ icon: single-tap → bottom sheet. No tooltip, no long-press (per DA-01 convention)."
Build Sheet line 167: "Tap = inline tooltip; long-press = bottom sheet with full GAAP framing + gross/refunds/net breakdown."
**Designer decision pending.** If R1 strict → row #4a folds into #2 (single tap reveals full BS).

### Row #3 — MoM chip
Per Brief line 208: ▲ green = collections increased (good). ▼ red = decreased. Inverted from DA-01 Dues. Microcopy per Build Sheet line 442.

### Row #4 — Efficiency badge
`Collectible = SUM(invoice.amount WHERE invoice.status=0 AND invoice.due_date ≤ period_end AND invoice.is_active=1)`. **NEW BUILD** — no Collectible denominator query in collections service today; cross-query to invoices needed.

### Row #5 — Period chip
Sends `start_date`/`end_date` via `applyPaidDateRange` at `helpers.ts:335-342`. Future-dated ranges return ₹0 with helper text #5a.

### Rows #6, #7 — "Has discount" / "No discount" chips (MUST-SHIP)
Backend per Brief line 141: single LEFT JOIN `payments LEFT JOIN credits ON credits.payment_id = payments.id AND credits.status=1`. "Has discount" = `WHERE credits.id IS NOT NULL`; "No discount" = `WHERE credits.id IS NULL`. `Credits.payment_id` is a direct FK at `entities/credits.ts:68`. Filter codes from DA-02's range (next genuinely-free integer is 1220 — 1209 = REFUND is used; 1288/12881 are out-of-range typos). Day-3 cut-line: escalate if codes unallocated.

### Rows #8-#12 — Source segments (4 segments + Adjustments row)
**Dual tap targets per Build Sheet line 207:** each segment has a colored bar segment AND a chip below it. Chip is the canonical 44pt tap target; bar is convenience. Long-press shows source-specific values tooltip (#12a) per Analytics PRD line 480 (Per-Spec Specifics post-orphan-audit). Adjustments (mode 211/288) excluded per Truth #3 / HB2.

### Rows #13-#15 — Population Breakdown
Filters per row: `tenant_types: [1]` (Active) / `[2]` (Booking) / `[0]` (Old) via `applyTenantTypes` at `helpers.ts:380-388` (with null support for unlinked covered separately in §5).

### Row #15a + #16 — §4.5 By Property breakdown (NEW v0.2)
Per Analytics PRD EC-07 line 577 + Build Sheet EC-07 line 418: appears between §4 Population and §5 Unlinked Alert when `selected_properties > 1`. Each row tap → worklist scoped to single property (Universal R7 property override). Removable chip `[Property: X ✕]`. Bypasses property grouping in worklist (Analytics PRD line 593).

### Row #18 — Link Payment 3-step flow
Per Build Sheet line 243: full-screen push; each of 3 steps is a separate frame in back-stack (R2 universal). Action permission: `record_payment`. **Tenant resolution priority in step 2** picks best-match per code at `helpers.ts:164`: ORDER BY `CASE t.status WHEN 2 THEN 1 WHEN 1 THEN 2 WHEN 0 THEN 3 WHEN 5 THEN 4 ELSE 5 END ASC`. Translates to: **booking (status=2) first, then active (1), then moved-out (0), then permanently-deleted (5).** (v0.1 had this reversed.)

### Row #18a — EC-13 Success banner
Per Analytics PRD line 604 + Build Sheet line 424: when efficiency = 100% (within ±0.5% rounding), "All dues collected! Everything is up to date." banner replaces or sits above Unlinked Alert. Population still shows normally.

### Row #20 — "Invoices Created" cross-screen drill to DA-01
**Cross-screen pattern (Truth #6):** routes to DA-01 Dues worklist with `start_date=period_start, end_date=period_end, status IN (0,1,2)` filter set. Back arrow: "← Collections" per R11.

### Row #21 — Collection chip
Same value as Hero #1. Restated in Performance accordion for side-by-side with Invoices Created chip (#20). Drill identical to Hero #1 — both blocked by HB1 + HB6.

### Rows #23-#26 — Timing rows (On Time / Early / Late / Adjustments)
Per Brief line 262: "Multi-invoice timing classification uses `MIN(inv.due_date)` — the oldest obligation." Ensures arrear-clearing payments are flagged Late correctly. Early row drill adds `paid_before_due_date=1` chip per Analytics PRD line 481.

### Row #27 — Partial Payments callout cross-screen drill
**Cross-screen to DA-01** per Build Sheet line 267 + Analytics PRD line 483 (Per-Spec Specifics). NEW BUILD — cross-query needed. Back arrow: "← Collections" (R11).

### Rows #30-#32 — Category drills
`applyDueTypes` at `helpers.ts:358-363`; needs `GROUP BY inv.due_type` aggregation (BUILD). Set `effectiveInvoiceLevelSum=true` per `helpers.ts:174-178` to avoid double-counting on multi-invoice payments (EC-09). **Mutex (EC-16, Build Sheet line 427):** when `filter_codes` is set, all of `payment_mode`, `due_types`, `received_by`, `start_date`, `end_date` are discarded.

### Rows #34-#40a — Payment Mode rows + adjustment separators + nudge
`applyPaymentMode` at `helpers.ts:353-355`; needs `GROUP BY p.payment_mode` aggregation (BUILD).
- **RentOk (Online):** modes 203, 205, 206, 210
- **Cash:** modes 202, 2041, 2040, 207 per `CASH_PAYMENT_MODES` constant at `payment/constants.ts:8`
- **Adjustments:** mode 211 (`DEPOSIT_PAYMENT_MODE` at `constants.ts:4`) + mode 288 (`ADVANCE_PAYMENT_MODE` at `constants.ts:10`)
- **Mode 213 (EazyPG credits):** per-row formula at `helpers.ts:494-504` — subtracts `gateway_charges` TWICE per HB3
- **Drift:** PRD entity comment on `payment_mode` is stale (says 0=cash, 1=paytm, 2=razorpay); real values are 3-digit — use Field Map §2.1.
- **Performance nudge (#40a):** per Build Sheet line 309 + microcopy lines 452-453. [VERIFY] threshold (≥1pp swing? ≥5pp?) — PM decision.

### Row #34a + #34b — RentOk "Unsettled" indicator (dual tap target + situational state)
**#34a tap (Analytics PRD line 485 Per-Spec Specifics):** tap on Unsettled indicator → unsettled-only drill (filter `payment_mode IN (203,205,206,210) AND payout_status != 1`). Tap on row body (#34) → all RentOk drill. Two distinct tap targets per anti-pattern #17.
**#34b situational state (EC-06 Analytics PRD line 574 + Build Sheet line 417):** settlement panel shows "Unsettled — Link Bank A/C" when `payout.bank_account IS NULL`; UTR field shows "Link Bank A/C to receive the payment."

### Row #45 + #45a — Concentration callout (3 stacked dependencies)
**HB4 4 conditions per Build Sheet line 124 + line 331:**
- (1) one staff > 50% of cash
- (2) AND ≥ 5 cash receipts in period
- (3) AND total cash by staff ≥ ₹10,000 (= 1,000,000 paise)
- (4) AND property ≥ 60 days old from launch
**Permission gate (Truth #8 + new key):** `viewCashAudit` JWT key. Owners bypass via `isOwner` per `commonFunctions.ts:1244`. ~0.5 dev-day to add to seeder + frontend permission map.
**Drill sort:** AMOUNT DESC (different from other Received By drills which use paid_date DESC) per Build Sheet line 331.

### Rows #47-#52a — Trend chart
**Internal time selector** independent of dashboard period (per Universal Rule R12 + Build Sheet line 343).
**Per R12:** bar single-tap = inline tooltip; tap-into-tooltip CTA "View [month] →" = drill.
**Yellow bar drill (#48):** CROSS-SCREEN to DA-01 with breadcrumb "← From DA-02 Collections" (R11).
**Green bar drill (#50):** stays in DA-02 with month override.
**Stacked sub-segment (#51):** drills to worklist with sub-segment + month filter.
**#52a Trend insight text:** conditional pattern callouts per Build Sheet line 353 + microcopy lines 458-459.

### Rows #53-#55 — Empty / error CTAs
- #53 Clear Filters: Build Sheet line 463 + Analytics PRD line 622. Empty state CTA when no payments match filters.
- #54 Network Retry: Build Sheet line 464 + Analytics PRD line 631. Re-fetches all sections in parallel.
- #55 Section Retry: Build Sheet line 465 + Analytics PRD line 632. Re-fetches the failed accordion only.

---

## Cross-Brief impact

**DA-02 Brief v0.7.4 — Q-by-Q mapping (v0.2 status):**
- Q1 (how much came in + last-month comparison) ✓ §1 (rows #1, #3)
- Q2 (am I on track — collection rate) ✓ §1 (row #4 Efficiency badge) + §6A (row #22)
- Q3 (where the money came from) ✓ §3 (rows #8-#12)
- Q4 (who paid — population) ✓ §4 (rows #13-#15) + §4.5 (row #16 multi-property)
- Q5 (which bill types collected well) ✓ §6B (rows #30-#32)
- Q6 (how people paid) ✓ §6C (rows #34-#40 + #34a, #34b, #40a)
- Q7 (who on team collected) ✓ §6D (rows #42-#46) — concentration callout permission-gated
- Q8 (trend) ✓ §7 (rows #47-#52a)
- Q9 (any unlinked payments) ✓ §5 (rows #17, #18, #18a)
- **PLUS (Brief v0.7.4 MUST-SHIP):** "Has discount" / "No discount" chips ✓ §2 (rows #6, #7)
- **PLUS (Brief line 128):** Headline integrity — covered by HB1 + HB6 blocking flags on rows #1, #21

**Cross-screen impact:**
- **INCOMING:** DA-05 → DA-02 with "Has discount" chip pre-applied (#7a).
- **OUTGOING:** 3 cross-screen drills DA-02 → DA-01 — #20 (Invoices Created), #27 (Partial Payments), #48 (Trend yellow bar).

**DA-02 vs DA-01 differences flagged:**
- Hero period-sensitive (DA-02) vs always-live (DA-01) — Truth #1
- MoM inverted (DA-02 ▲ good) vs (DA-01 ▼ good) — Truth #4
- Hero block entire tappable (DA-02) vs hero number not tappable (DA-01)
- Performance accordion expanded by default (DA-02) vs collapsed (DA-01 §7)
- ⓘ icon convention possibly different per Designer Decision (pending)
- "Receipts" subtitle wording (DA-02 HB5) vs "Bills" (DA-01)

---

## Anti-patterns codified (carry from DA-01 work — apply to all DA suite from kickoff)

#11 Sub-agent verification ceiling · #12 Section-count math drift · #13 Build Sheet vs code drift goes silent · #14 [SUPERSEDED by #19] · #15 Deferred-section list ≠ comprehensive audit · #16 Drill-destination resolved by canonical PRD · #17 Dual tap targets per UI element · #18 Permission-driven action hiding is a Truth · #19 Drill maps must not become Build Sheet duplicates.

**DA-02-specific patterns surfaced in v0.1 → v0.2:**
- **#20 (NEW DA-02 v0.2) — Brief vs Build Sheet UX convention drift.** When Brief locks a universal convention but Build Sheet specifies a per-section exception, flag for designer resolution rather than silently picking one. Example: Brief line 213 ⓘ single-tap → BS; Build Sheet line 167 tap = tooltip + long-press = BS. v0.1 included both as architectural truth — v0.2 reclassified as Pending Designer Decision.
- **#21 (NEW DA-02 v0.2) — Hero-value blocking flags must propagate to derived rows.** HB1 + HB6 affect Hero #1 AND Collection chip #21 (same number restated in Performance accordion). When a single architectural fix blocks multiple rows, the status marker must cite both rows.
- **#22 (NEW DA-02 v0.2) — Tenant resolution priority must be quoted from code, not paraphrased.** v0.1 paraphrased the ORDER BY CASE statement and reversed the actual priority (booking vs active swap). v0.2 fix: when describing internal logic, copy the exact CASE expression's effect, then translate to plain English with verified mapping.

---

## Changelog

| Date | Version | Change |
|------|---------|--------|
| 2026-05-19 | v0.1 | Initial draft — all 19 anti-patterns from DA-01 work applied from kickoff. Lean structure. 56 rows claimed (actual 56 unique rows). Awaiting 3-angle audit. |
| 2026-05-19 | v0.3.1 | **Round-3 audit polish.** Round-3 verdict: correctness APPROVED-with-minor-flag (7/8 verified + 1 typo in frontmatter "75 → 76"); plain-English Maybe-with-edits (only #40c tooltip jargon, #1c bracket jargon, #52b code-field reference); coverage NEARLY COMPLETE (1 minor positional gap on #40b). 5 polish edits applied: **(1)** Row #40c tooltip text re-translated from engineering ("gateway charges subtracted twice in production code") to operator-plain-English ("EazyPG Credit payments are shown after RentOk's processing fee. The amount you see reflects the net credit applied — slightly lower than the gross billed."). **(2)** Row #1c bracket-jargon "[Meena pattern]" removed; reads as plain sentence. **(3)** Row #52b `paid_date` code-field reference replaced with "the month the payment was received." **(4)** Row #40b ordinal position added ("separate row sitting between Cash (#37) and Others (#38) in the §6C list"). **(5)** Frontmatter row count corrected 75 → 76. No new rows; no math changes (still canonical 76). |
| 2026-05-19 | v0.3 | **Round-2 three-angle audit corrections.** Round-2 result: correctness FLAG (all 11 code line refs + tenant priority + Build Sheet vs Analytics PRD attribution + all 10 new content items verified; only status math broken — actual rows were 70 not claimed 65, bucket sum was 68); plain-English 9 of 10 dimensions PASS (only status math criticism); coverage NEARLY COMPLETE (1 HIGH + 3 MED + 4 LOW remaining). All 8 items fixed in v0.3: **(1) Mode 213 tooltip (HIGH)** — added row #40c per Brief line 137 MUST-SHIP for credit-net math explanation. **(2) Mode 213 §6C placement (MED)** — added row #40b explicit "EazyPG Credits" row (was previously folded into Others NOT-IN list, invisible to operator). **(3) EC-09 row-vs-sum caveat (MED)** — added row #32a situational note. **(4) EC-15 trend chart calendar-span (MED)** — added row #52b. **(5) Period picker "Today" not offered (LOW)** — added row #5b explicit render rule. **(6) EC-12 100% concentration copy variant (LOW)** — added row #45b. **(7) Row #1c plain-English softened** — replaced raw permission key name with plain-English explanation. **(8) Status math reconciled** — recounted to canonical 76 rows; deleted self-contradicting "sanity check overshoots" footnote; each row in exactly one bucket; clean sum check. 6 new rows (70 → 76). No new anti-patterns. Awaiting round-3 verification for PASS confirmation. |
| 2026-05-19 | v0.2 | **Round-1 three-angle audit corrections.** Round-1 found 1 CRITICAL coverage gap (EC-07 By Property breakdown row missing entire sub-section), 5 HIGH coverage gaps (EC-02 negative hero, EC-13 success banner, EC-08 Limited view, EC-06 settlement copy, Worklist Clear Filters), 10+ code line refs drifted by 5-15 lines, tenant resolution priority narrative reversed (code ranks booking → active → moved-out → perma-del; v0.1 said active → moved-out → booking → perma-del), SQL leak in PM-facing filter cells, 11-truths overload (folded to 8), Build Sheet line ref attribution errors (lines 480/483/485 actually Analytics PRD content), status math wrong (claimed 56 = 50+4+2 but actually 59 with double-counting). All fixed: **(1)** §4.5 By Property breakdown sub-section added (rows #15a + #16). **(2)** 5 high-severity situational states added (#1a, #1b, #1c, #1d, #18a, #34b, plus #40a, #52a). **(3)** Empty/error CTAs added (#53, #54, #55). **(4)** 10+ code line refs corrected (helpers.ts, service.ts, commonFunctions.ts, entities/credits.ts). **(5)** Tenant resolution priority corrected. **(6)** SQL stripped from PM-facing cells; tech specifics quarantined to Eng Ref. **(7)** 11 Truths folded to 8 (combined HB1+HB6, mode 211/288+mode 213; converted ⓘ drift to Pending Designer Decision callout). **(8)** Build Sheet vs Analytics PRD line attribution fixed. **(9)** Status summary math reconciled (65 rows total, single canonical count, no double-counting). **(10)** 3 NEW anti-patterns codified (#20 Brief vs Build Sheet UX drift, #21 Blocking flags propagate, #22 Tenant resolution from code not paraphrase). Row count 56 → 65 (+9 from coverage additions). Awaiting round-2 verification. |
