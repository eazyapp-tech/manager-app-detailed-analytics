---
title: DA-01 Dues — Drill Map (Companion doc)
date: 2026-05-19
tags:
  - rentok
  - drill-map
  - dues
  - operator-first
status: Living document · v0.5.3 (lean restructure — drill map enumerates ONLY tap-relevant rows. Display labels folded into section headers; QA invariants, sort orders, visual styles, color rules removed from drill map and left to Build Sheet ownership. 53 rows → 33 rows. New anti-pattern #19 codified.)
owner: Sanchay (PM)
companion_to: DA-01 Brief (v0.10.1), DA-01 Build Sheet V2 (status V2-locked), DA-01 Dues Detailed Analytics (V3.4 canonical "why" doc)
graph_rev: 2026-05-19 (8080 nodes / 17628 edges)
---

> [!INFO] What this is
> A **PM + designer-readable drill map** for DA-01 Dues. Every tap-able element → existing screen + filters (in plain English). Engineers find the technical wiring below the hard divider.
>
> **What this is NOT:**
> - A Build Sheet (engineering ticket spec) — display rules, QA invariants, sort orders, color rules live there.
> - A re-opened Brief — the signed-off DA-01 Brief v0.10.1 stays frozen.
>
> **What changed in v0.5.3:** non-tappable display labels, QA invariants, sort/visual/render rules removed from this drill map. They belong in the Build Sheet, which already enumerates them per-section. This drill map now answers the question it's named after: *what happens when the operator taps X.*

---

## Scope of this drill map (v0.5.3)

**In scope (8 of 12 UI sections):**
- §1 Hero & MoM Chip
- §2 Urgency Bar — Today Mode
- §3 Urgency Bar — Range Mode
- §4 Population Breakdown
- §5 Property Breakdown (multi-property only)
- §6 Action Card
- §7 Performance Accordion (period-sensitive)
- §8 By Category Accordion

**Out of scope — deferred to v0.6:**
- §9 By Defaulters: Aging — Brief Q6 commitment, locked for V1
- §10 Top Defaulters — Brief Q7, drills to Tenant Dues Detail (different destination)
- §11 Added By Accordion
- §12 Defaulter Analysis Chart (non-tappable in V1)
- Screen chrome: date filter chip, property selector, Setting-Up banner CTAs, pull-to-refresh, empty/error state buttons, data freshness indicator

---

## Ready-when legend

| Marker | Meaning |
|--------|---------|
| ✅ V1 | Works today — frontend just wires it up |
| ⚠️ V1 (second-level CSB-2) | First-level drill works; second-level drill from worklist row → bill detail goes through an unauthenticated endpoint (fix before launch) |
| 🔧 V1 hot-add | Small backend wiring needed (≤ 1 dev-day per dependency) |
| 🔧 V2 | Needs a new screen or schema change |

### Acronyms (plain English)
- **CSB-2** — bill-detail endpoint missing auth. Anyone could read bill data anonymously. Affects every "tap a tenant row in the worklist → see their bills" path. Jatin gate P0.
- **CSB-3** — financial-mutation endpoint missing auth. Anyone could trigger "Adjust from Deposit" anonymously. Jatin gate P0 for adjustDeposit.
- **HB6** — backend filter widen needed (`t.status IN (1,2)` → `IN (0,1,2)`) so Old Tenants surface in queries. Build Sheet says RESOLVED but code hasn't been updated.

---

## Architectural truths (apply suite-wide)

Each truth has a **plain-English line** (for PM/designer) and a folded **technical detail** (for engineering).

**Truth #1 — `CURRENT_MONTH_*` filter codes lock the response to current month.**
- *Plain English:* Don't use the `CURRENT_MONTH_*` filter codes for anything other than "This Month" — they'll override the operator's period.
- *Technical:* `DuesFilterCode` enum range 1101-1114 at `src/v1/constants/filterCodes.ts:1-15`; codes 1103-1107 hardcode `monthStart`/`nextMonth` at `helpers.ts:117-118`.

**Truth #2 — Mutex between filter_code and three body params.**
- *Plain English:* If a `filter_code` is sent, three body params (date_range, due_types, added_by) are silently dropped. Tenant_types is the exception — it always applies.
- *Technical:* `service.ts:60-76` gates `applyDateRange`/`applyDueTypes`/`applyAddedBy` behind `!hasFilterCode`. `applyTenantTypes` at `service.ts:78-80` runs unconditionally.

**Truth #3 — OVERDUE filter (1102) is always current.**
- *Plain English:* Use the OVERDUE filter code freely — it's always "past due as of today" regardless of period.
- *Technical:* `helpers.ts:129-130` — `due_date < tomorrow`.

**Truth #4 — DA-01 hero is always-live; period chip slices the cuts BELOW it.**
- *Plain English:* The big "Total Dues" number never changes when the operator picks a different period. The urgency bar, performance accordion, and added-by accordion DO change; everything else stays current.
- *Technical:* Brief line 184 + Build Sheet line 148. Different from DA-07.

**Truth #5 — Empty-drill UX contract.**
- *Plain English:* When a drill lands on a destination with zero results, show the active filter context at the top (period + property + filter chip + sort) so the operator sees WHY empty. Copy template: "No [type] [bills] in [period] for [property]."

**Truth #6 — Only one cross-screen drill in §1-§8: §7 Collection callout → DA-02 Collections.**
- *Plain English:* Every drill in §1-§8 opens the Dues worklist EXCEPT the Performance accordion's "Collection callout" row, which jumps to the DA-02 Collections tab. Back arrow says "← Dues" (Today) or "← Dues ([Period])" (Range).
- *Technical:* Build Sheet line 277 + Analytics PRD line 542. DA-02 worklist has its own auth (separate audit). Open question: DA-02's second-level drill CSB risk — flag for DA-02 audit.

**Truth #7 — Drill destination resolved: first-level OK, second-level needs CSB-2 fix.**
- *Plain English:* When the operator taps a DA-01 card or row, they go to the Dues worklist (authenticated, OK). When they then tap a tenant row IN the worklist to see bill details, that goes through a legacy endpoint with no authentication. Fix the second-level endpoint before launch.
- *Technical:* First-level = `POST /v1/list_screens/dues/list/filters` with HeaderValidator at `routes.ts:8`. Second-level = `POST /invoices/fetchDueDetailsForTenants` with zero middleware at `routes/invoices.ts:203`.

**Truth #8 — Permission-driven action hiding affects every worklist drill row.**
- *Plain English:* Every "Primary row action" listed in the table (Send Reminder, View Bill Detail, Record Payment, Adjust from Deposit) is HIDDEN — not greyed out — when the operator's role lacks the corresponding permission. Limited-access staff (e.g., Meena with accounting-only access) won't see actions they can't perform. The drill itself still works; only the per-row action buttons hide.
- *Technical:* Analytics PRD line 800 (EC-10). Permission keys: `record_payments`, `edit_dues_and_collection`, `view_invoices`, `send_reminders`, role-derived flags.

---

## DA-01 vs DA-07 mode difference

DA-01 hero is **always-live** (period chip slices the cuts BELOW it). DA-07 hero is **period-sensitive**. So on DA-01, MoM chip tap IS operator-correct (different from DA-07 where the period chip is the path).

---

# DA-01 Drill Map — tap-relevant rows only (§1-§8 of 12 UI sections)

> **How to read the "Filters applied" column:** the **Primary row action** at the bottom of each filter cell is shown on each row of the destination worklist — but **hidden if the operator lacks permission** (Truth #8).
>
> **Where to find display rules, QA invariants, sort orders, color rules:** Build Sheet, per-section component tables.

| # | Tap on… | Goes to (existing screen) | Filters applied | Ready when? |
|---|---------|--------------------------|-----------------|-------------|
| **§1 HERO & MoM CHIP** | | | | |
| | *Also rendered (display only, not tappable): hero number, invoice count subtitle, multi-property suffix when applicable, "Dues (Live)" header label, MoM chip color/visibility rules. See Build Sheet lines 156-163 for display specs.* | | | |
| 2 | The "ⓘ" icon next to the Hero | Stays on DA-01 — opens a bottom sheet with plain-English explanation + GAAP framing ("Accounts Receivable") | • In-screen reveal | ✅ V1 |
| 3 | MoM chip (▼/▲ %, with prior absolute alongside per v3.2 — e.g., "▲8% vs ₹14.8L last month") | Stays on DA-01 — opens an inline tooltip showing prior-period numbers + which day-range is compared | • In-screen tooltip reveal | ✅ V1 |
| | 💡 **DA-01 ≠ DA-07:** DA-01 hero is always-live + period chip slices the cuts BELOW the hero, so the period chip is NOT the path for "see prior month for the headline." MoM tap IS the path on DA-01. | | | |
| 3c | MoM chip "Not enough prior data" state | — (situational display variant of #3 — shown instead of hiding silently when account < 30 days OR prior period = ₹0 OR same-day-last-month in Setting-Up state. Analytics PRD v3.2 lines 177-181.) | — | — |
| **§2 URGENCY BAR — TODAY MODE** (renders only when the operator has selected just today; **each segment has TWO tap targets per Analytics PRD line 228: the colored bar segment AND the label row below it**) | | | | |
| | *Also rendered (display only): bar segment proportions (4px min), Today-mode reconciliation invariant. See Build Sheet lines 184, 187.* | | | |
| 4 | "Already Due" red segment OR its label row | Dues worklist | • Show only: bills past their due date<br>• Period: anchored to today<br>• Property: selected<br>• Sort: most overdue first<br>• Header: "Already Due · X Bills · ₹Y"<br>• **Primary row action: Send Reminder** (permission-gated — Truth #8) | ✅ V1 (⚠️ second-level CSB-2) |
| 5 | "Due Today" amber segment OR its label row | Dues worklist | • Show only: bills with due date = today<br>• Period: anchored to today<br>• Property: selected<br>• Sort: amount, largest first<br>• Header: "Due Today · X Bills · ₹Y"<br>• **Primary row action: Send Reminder** | ✅ V1 (⚠️ second-level CSB-2) |
| 6 | "Due This Week" blue segment OR its label row | Dues worklist | • Show only: bills due between tomorrow and this Sunday (calendar week Mon-Sun)<br>• Period: anchored to today<br>• Property: selected<br>• Sort: due date soonest first<br>• Header: "Due This Week · X Bills · ₹Y"<br>• **Primary row action: Send Reminder** | ✅ V1 (⚠️ second-level CSB-2) |
| 7 | "Due Later" green segment OR its label row | Dues worklist | • Show only: bills due after this Sunday<br>• Period: anchored to today<br>• Property: selected<br>• Sort: due date ascending<br>• Header: "Due Later · X Bills · ₹Y"<br>• **Primary row action: View Bill Detail** (per Analytics PRD line 512) | ✅ V1 (⚠️ second-level CSB-2) |
| 7a | Per-segment ⓘ tooltip on Today-mode segments (#4-#7) | Stays on DA-01 — opens inline tooltip with plain-English explanation of that segment (4 distinct microcopy strings — Build Sheet lines 494-497) | • In-screen reveal | ✅ V1 |
| **§3 URGENCY BAR — RANGE MODE** (renders when the operator selects any period other than today; same dual-tap-target rule as §2) | | | | |
| | *Also rendered (display only): Range-mode reconciliation invariant. See Build Sheet line 206.* | | | |
| 8 | "Carried Forward" segment OR its label row | Dues worklist | • Show only: unpaid bills with due date before the selected period started<br>• Period: selected period from chip<br>• Property: selected<br>• Sort: most overdue first<br>• Header: "[Period] · Carried Forward · X Bills · ₹Y"<br>• **Primary row action: Send Reminder** | ✅ V1 (⚠️ second-level CSB-2) |
| 9 | "Due in Period" segment OR its label row | Dues worklist | • Show only: unpaid bills with due date inside the selected period<br>• Period: selected<br>• Property: selected<br>• Sort: due date ascending<br>• Header: "[Period] · Due in Period · X Bills · ₹Y"<br>• **Primary row action: Send Reminder** | ✅ V1 (⚠️ second-level CSB-2) |
| 10 | "Due After Period" segment OR its label row | Dues worklist | • Show only: unpaid bills with due date after the selected period ended<br>• Period: selected<br>• Property: selected<br>• Sort: due date ascending<br>• Header: "[Period] · Due After Period · X Bills · ₹Y"<br>• **Primary row action: View Bill Detail** | ✅ V1 (⚠️ second-level CSB-2) |
| 10a | Per-segment ⓘ tooltip on Range-mode segments (#8/#9/#10) | Stays on DA-01 — opens inline tooltip with plain-English explanation (3 distinct microcopy strings — Build Sheet lines 498-500) | • In-screen reveal | ✅ V1 |
| 10c | Range-mode info banner | — (situational display: "Live values reflect right now. Period values show [period] activity." Shown only in Range mode, below date filter. Build Sheet line 508 + Analytics PRD line 776.) | — | — |
| 10d | Range-mode zero-data placeholder | — (situational display: "No bills due in this period." Renders in place of the urgency bar when selected period has no bills. Live elements unaffected. Analytics PRD EC-13 line 809 + Build Sheet line 513.) | — | — |
| **§4 POPULATION BREAKDOWN** (always-live — period filter doesn't apply) | | | | |
| | *Also rendered (display only): deposit context inline on each Old Tenants row, Population reconciliation invariant. See Build Sheet lines 224, 225.* | | | |
| 11 | "Active Tenants" row (currently checked-in tenants with dues) | Dues worklist | • Show only: active tenants<br>• No period (always-live)<br>• Property: selected<br>• Sort: amount, largest first<br>• Header: "Active Tenants · X Tenants · ₹Y"<br>• **Primary row action: Send Reminder** | ✅ V1 (⚠️ second-level CSB-2) |
| 12 | "Bookings" row (confirmed future move-ins with dues) | Dues worklist | • Show only: booking tenants<br>• No period (always-live)<br>• Property: selected<br>• Sort: amount, largest first<br>• Header: "Bookings · X Tenants · ₹Y"<br>• **Primary row action: Send Reminder** | ✅ V1 (⚠️ second-level CSB-2) |
| 13 | "Old Tenants" row (moved-out tenants still owing) | Dues worklist | • Show only: old tenants (moved-out)<br>• No period (always-live)<br>• Property: selected<br>• Sort: days-since-checkout, longest first<br>• Header: "Old Tenants · X Tenants · ₹Y"<br>• **Primary row action: Adjust from Deposit** (DIFFERENT from other rows — uses CSB-3 financial-mutation endpoint) | 🔧 V1 hot-add + ⚠️ CSB-2 + ⚠️ CSB-3 |
| | 💡 **Old Tenants — 3 dependencies:** (a) HB6 widen, (b) new sort key support, (c) CSB-3 auth-fix on Adjust-from-Deposit. Brief degraded-launch fallback: "If the backend change slips, ship V1 without the moved-out segment." Full eng detail (file refs, dev-day estimates) in Engineering Reference below. | | | |
| 13b | Per-row ⓘ tooltip on Population rows (#11/#12/#13) | Stays on DA-01 — opens inline tooltip explaining tenant-status semantics (3 distinct microcopy strings — Build Sheet lines 505-507) | • In-screen reveal | ✅ V1 |
| **§5 PROPERTY BREAKDOWN — MULTI-PROPERTY ONLY** (always-live) | | | | |
| 14 | (Section visibility — entire §5 hidden when single-property dashboard) | — (render rule: visible only when account has more than one property AND more than one is currently selected. Build Sheet line 239.) | — | — |
| | *Also rendered (display only): mini urgency bar inside each row, property sort order, Property reconciliation invariant. See Build Sheet lines 240-242.* | | | |
| 15 | Per-property row (e.g., "Sunshine PG · ₹8L · 50% · [mini urgency bar]") | Dues worklist — **scopes to single property** regardless of multi-property dashboard scope | • Property: scoped to single property (OVERRIDES dashboard multi-prop scope — R7 in PRD)<br>• No period (always-live)<br>• Sort: most overdue first<br>• Header: "[Property Name] · X Bills · ₹Y"<br>• Removable filter chip "[Property: X ✕]" at top<br>• Bypasses property grouping in the worklist (Analytics PRD line 593)<br>• **Primary row action: Send Reminder** | ✅ V1 (⚠️ second-level CSB-2) |
| **§6 ACTION CARD** (situational) | | | | |
| 16 | (Section visibility — Action Card hidden when (a) zero overdue OR (b) property in Setting-Up state) | — (render rule with TWO triggers — Build Sheet line 256 + Analytics PRD line 294.) | — | — |
| | *Also rendered (display only): card text label "X Rent & Bills Overdue · View All >", dark high-contrast visual style. See Build Sheet lines 257, 259.* | | | |
| 18 | "View All >" card tap (whole card is the tap target per Analytics PRD line 292) | Dues worklist | • Show only: bills past due date<br>• Period: anchored to today (always-live — IGNORES dashboard period, even in Range mode)<br>• Property: selected<br>• Sort: most overdue first<br>• Header: "Past Due · X Bills · ₹Y" (note: different header text from #4 "Already Due")<br>• **Primary row action: Send Reminder** | ✅ V1 (⚠️ second-level CSB-2) |
| **§7 PERFORMANCE ACCORDION** (period-sensitive; all metrics use "due date in period", NOT "created in period"; collapsed by default) | | | | |
| | *Also rendered (display only): Collection Efficiency metric, color thresholds (Green > 70%, Amber 40-70%, Red < 40%). See Build Sheet line 276.* | | | |
| 19 | Performance accordion header tap (expand/collapse) | Stays on DA-01 — expands the accordion in-place (default state: collapsed per Analytics PRD line 302) | • In-screen reveal | ✅ V1 |
| 19a | Performance Setting-Up overlay | — (situational render state: when account is in Setting-Up state AND operator opens Performance accordion, an overlay says "Need 30 days of bills to show meaningful efficiency. You have X days." Replaces metrics until threshold met. Build Sheet line 387.) | — | — |
| 20 | "Invoices Due" metric (total of bills due in period, paid + unpaid) | Dues worklist | • Show: ALL bills (paid + unpaid) with due date inside selected period<br>• Period: selected<br>• Property: selected<br>• Sort: due date ascending<br>• Header: "[Period] Invoices Due · X Bills · ₹Y" (or "Today Invoices Due ..." in Today mode)<br>• **Primary row action: View Bill Detail** (per Analytics PRD line 540) | ✅ V1 (⚠️ second-level CSB-2) |
| 21 | "Current Dues" metric (unpaid portion of period's bills) | Dues worklist | • Show only: bills with due date inside selected period AND still unpaid<br>• Period: selected<br>• Property: selected<br>• Sort: most overdue first<br>• Header: "[Period] Current Dues · X Bills · ₹Y" (or "Today Current Dues ..." in Today mode)<br>• **Primary row action: Send Reminder** | ✅ V1 (⚠️ second-level CSB-2) |
| 22b | Collection Efficiency "Not enough billed yet" / "—" state | — (situational display variant: shown when Invoices Due = ₹0, Setting-Up state OR quiet month. Analytics PRD v3.2 line 316.) | — | — |
| 23 | Collection callout ("₹X collected against bills due [start] → [end]") — **CROSS-SCREEN DRILL (Truth #6)** | **DA-02 Collections tab** (NOT Dues worklist) | • Show: payments made against bills due in the selected period<br>• Period: selected<br>• Property: selected<br>• Back arrow: "← Dues" (Today mode) or "← Dues ([Period])" (Range mode)<br>• DA-02 worklist auth: separate audit (not CSB-2-affected at first level) | ✅ V1 (cross-screen to DA-02) |
| 23a | Per-metric ⓘ tooltips × 3 (Invoices Due / Current Dues / Collection Efficiency — each independently tappable) | Stays on DA-01 — opens inline tooltip per metric (3 distinct microcopy strings — Analytics PRD lines 326-328) | • In-screen reveal | ✅ V1 |
| **§8 BY CATEGORY ACCORDION** (always-live; collapsed by default; top 5 categories by amount, rest aggregate to "Others") | | | | |
| | *Also rendered (display only): category sort order (biggest first), By Category reconciliation invariant. See Build Sheet lines 294, 295.* | | | |
| 24 | By Category accordion header tap (expand/collapse) | Stays on DA-01 — expands the accordion in-place | • In-screen reveal | ✅ V1 |
| 25 | Category row (e.g., "Rent · ₹8L" / "Electricity · ₹2L" — one row per top-5 category) | Dues worklist | • Show only: bills in selected category (Rent, Electricity, Food, Security Deposit, Maintenance, etc.)<br>• No period (always-live — note: different from §7 Performance accordion)<br>• Property: selected<br>• Sort: amount descending<br>• Header: "[Category] · X Bills · ₹Y"<br>• **Primary row action: Send Reminder** | ✅ V1 (⚠️ second-level CSB-2) |
| 26 | "Others" row tap (aggregates 6th category onwards) — expand-then-tap pattern: first tap expands the breakdown inline, second tap on a sub-category drills to worklist (designer confirm) | Dues worklist (after expand) | • Show only: bills in "Others" categories<br>• No period (always-live)<br>• Property: selected<br>• Sort: amount descending<br>• Header: "Others · X Bills · ₹Y"<br>• **Primary row action: Send Reminder** | ✅ V1 (⚠️ second-level CSB-2) |
| 26c | "Others" tooltip ("Tap to expand and see breakdown of other categories like Security Deposit, Maintenance, etc.") | Stays on DA-01 — inline tooltip | • In-screen reveal | ✅ V1 |

---

## Status summary (§1-§8 — partial DA-01 audit, 8 of 12 UI sections)

**Total tap-relevant rows: 33** (26 tap targets + 5 situational display states + 2 section visibility rules)

| Status | Count | Notes |
|--------|-------|-------|
| ✅ V1 (in-screen reveal: tooltip / bottom sheet / accordion header) | 9 | #2, #3, #7a, #10a, #13b, #19, #23a, #24, #26c |
| ✅ V1 (first-level drill OK, ⚠️ second-level CSB-2) | 15 | #4, #5, #6, #7, #8, #9, #10, #11, #12, #15, #18, #20, #21, #25, #26 |
| ✅ V1 (cross-screen to DA-02) | 1 | #23 |
| 🔧 V1 hot-add + ⚠️ CSB-2 + ⚠️ CSB-3 | 1 | #13 (Old Tenants — HB6 widen + sort key + Adjust-from-Deposit auth) |
| Situational display states (operator-visible, no tap) | 5 | #3c MoM no-prior-data, #10c Range info banner, #10d Range zero-data placeholder, #19a Performance setup overlay, #22b ÷0 guard |
| Section visibility rules | 2 | #14 §5 multi-property only, #16 §6 hidden when no overdue OR setup |

**Critical pending decisions:**
1. **CSB-2 fix** — applies to second-level drill (worklist row → bill detail) for all 16 worklist-drill rows.
2. **CSB-3 fix** — applies to Old Tenants Adjust-from-Deposit (row #13 primary action).
3. **HB6 fix** — apply `IN (0,1,2)` widen at `helpers.ts:61` OR correct Build Sheet RESOLVED status tag.
4. **applyOrderBy `days_since_checkout` extension** for row #13 sort.
5. **DA-02 Collection-callout destination** — confirm DA-02 first-level drill passes its own auth review.
6. **Row #26 "Others" expand-then-tap pattern** — designer confirms.

---

---

# Engineering Reference (technical specifics per row)

## Verification protocol applied (v0.5.3)

Same as v0.5.2 (round-3 three-angle audit: correctness ✓ all 58 claims verified; coverage closed 4 HIGH gaps; plain-English overhaul). v0.5.3 adds: lean restructure per anti-pattern #19. All claims personally verified against code (`src/v1/list_screens/dues/*` + `src/routes/invoices.ts` + `src/routes/tenant.ts`) + Build Sheet + Analytics PRD V3.4 + Brief.

---

## Per-row engineering details

### Row #2 — Hero ⓘ icon tap
**Behavior:** Single-tap → bottom sheet per Build Sheet line 159. **Microcopy:** Build Sheet line 492.

### Row #3 + #3c — MoM chip
**Tap behavior:** inline tooltip per Build Sheet line 161. **Microcopy:** Build Sheet line 493 + Analytics PRD lines 174-175.
**Row #3c "Not enough prior data" state** — 3 conditions per Analytics PRD v3.2 lines 177-181: (a) account < 30 days, (b) prior period = ₹0 (÷0 case), (c) same-day-last-month in Setting-Up. Eng must implement explicit state, not silent hide.

### Rows #4-#10 — Urgency segments + dual tap targets
**First-level drill endpoint (Truth #7):** `POST /v1/list_screens/dues/list/filters` (HeaderValidator at `routes.ts:8`).
**Second-level CSB-2:** worklist row → bill detail = legacy `/invoices/fetchDueDetailsForTenants` at `routes/invoices.ts:203` (zero middleware).
**Dual tap targets (Analytics PRD line 228):** each segment has TWO tap surfaces — the colored bar AND the label row below (2×2 grid in Today, 1×3 in Range).
**Mode detection drift:** Build Sheet line 186 says "Driven by `filter_meta.mode` from API"; grep returns ZERO matches. Reality: frontend-derived from period chip. Drift to resolve.

### Rows #7a, #10a, #13b, #23a, #26c — Tooltips
Each tooltip is a separate tap target per UI element. Tooltip text per Build Sheet §18 line refs:
- #7a Today (4 strings): lines 494-497
- #10a Range (3 strings): lines 498-500
- #13b Population (3 strings): lines 505-507
- #23a Performance metrics (3 strings): Analytics PRD lines 326-328 (Build Sheet §18 should also have — verify)
- #26c Others (1 string): line 503

### Row #10d — Range-mode zero-data placeholder
Per Analytics PRD EC-13 line 809 + Build Sheet line 513. Frontend renders when range-mode segment counts all return zero.

### Rows #11-#13 — Population Breakdown
Filters per row: `tenant_types: [1]` (Active) / `[2]` (Booking) / `[0]` (Old). Verified `applyTenantTypes` at `helpers.ts:192`, body-param-wired at `service.ts:78-80`.

**Mutex exception (Truth #2):** `applyTenantTypes` is NOT gated behind `!hasFilterCode` — can be combined with filter_code.

**Row #13 — Old Tenants — 3 stacked dependencies:**
- **(a) HB6 backend widen** — `dues/helpers.ts:61` currently `t.status IN (1, 2)`; Build Sheet says "RESOLVED — widen to `IN (0,1,2)`" but code hasn't been updated. Build Sheet → code drift.
- **(b) Sort-key hot-add** — `applyOrderBy` at `helpers.ts:241-246` supports only `amount`/`room`/`MAX(due_date)`. Need to add `days_since_checkout`. ~0.5 dev-day.
- **(c) CSB-3 P0 auth fix** — Old Tenants primary row action "Adjust from Deposit" routes to `POST /tenant/adjustDeposit` at `routes/tenant.ts:944` — financial mutation with zero auth (verified). Jatin gate P0.
- **Brief degraded-launch fallback (Brief line 122):** "If the backend change slips, ship V1 without the moved-out segment and label that clearly."

### Row #14 — §5 Section visibility
Per Build Sheet line 239: render if `properties_in_account > 1 AND selected_properties > 1`. Single-property dashboards don't see §5.

### Row #15 — Per-property row
**Property override semantics (R7 in PRD):** drill scopes to single property regardless of dashboard multi-property scope. `pg_number_filter: [X]` passed; removable filter chip `[Property: X ✕]` shown in worklist header. Bypasses property grouping in the destination worklist (Analytics PRD line 593).

### Row #16 — §6 Action Card visibility
TWO triggers per Build Sheet line 256 + Analytics PRD line 294: hidden when (a) `overdue_count == 0` OR (b) property in Setting-Up state.

### Row #18 — Action Card "View All"
**Always-today-anchored** (Build Sheet line 250 + Analytics PRD line 292). Same filter shape as #4 Already Due but with header "Past Due · X · ₹Y" (Build Sheet line 258) — eng note: keep visually distinct from #4 since they drill to identical filter set.

### Row #19 + #19a — Performance Accordion
**Period-sensitivity:** ALL Performance metrics use `due_date IN [period]`, NOT `created_at` (Analytics PRD line 558 + Build Sheet line 269).
**#19 accordion header:** default state collapsed (Analytics PRD line 302).
**#19a Setting-Up overlay** per Build Sheet line 387 — replaces metrics until threshold met.

### Row #23 — Collection callout — CROSS-SCREEN to DA-02
Unique pattern: routes to DA-02 Collections tab (Build Sheet line 277 + Analytics PRD line 542). Back arrow `← Dues` (Today) or `← Dues ([Period])` (Range). DA-02 first-level has its own HeaderValidator. Open question: DA-02 second-level CSB risk — flag for DA-02 audit.

### Rows #24-#26c — By Category Accordion
**Always-live semantics** even though §8 is in "Advanced Insights" (Build Sheet line 286 + Analytics PRD line 336).
**Row #25 Category row:** dynamic count based on `due_type` values present. Top 5 by amount; 6th+ aggregate into #26.
**Mutex application (Truth #2):** §8 drill must send `due_types: [X]` body param WITH OMITTED `filter_code`.
**Row #26 "Others" expand-then-tap pattern** — Build Sheet line 293 + Analytics PRD line 347/351 — first tap expands inline; second tap on sub-category drills to worklist. Designer confirm.

---

## Cross-Brief impact

**DA-01 Brief v0.10.1 — Q-by-Q mapping (v0.5.3 status):**
- Q1 (always-live hero + MoM) ✓ §1 (rows #2, #3, #3c)
- Q2 (urgency split) ✓ §2 + §3 (rows #4-#10)
- Q3 (segment counts) ✓ §4 (rows #11-#13)
- Q4 (per-property comparison row) ✓ §5 (row #15)
- Q5 (bill-type split) ✓ §8 (rows #25, #26) — **flagged:** Security Deposit may only surface under Others if not top-5 by amount; designer/PM confirm acceptable.
- Q6 (aging bands) — deferred to v0.6 (§9)
- Q7 (biggest items still owed) — deferred to v0.6 (§10)
- Q8 (period performance) ✓ §7 (rows #20, #21) + §3 Range mode
- Q9 (tenant drill-in) — deferred to next session (§15)

§6 Action Card is not a Brief Q — it's a PRD/Build Sheet addition for "what should I do first" cue.

**Cross-screen impact:** only one cross-screen drill in §1-§8 — row #23 Collection callout → DA-02 Collections.

---

## Anti-patterns codified across DA-01 work

- **#11** Sub-agent verification ceiling.
- **#12** Section-count math drift.
- **#13** Build Sheet vs code drift goes silent.
- **#14** ~~Enumeration incompleteness via "not tappable" cherry-pick.~~ **SUPERSEDED by #19.** Original mitigation ("enumerate all non-tappable rows for consistency") created clutter that obscured tap targets. Right mitigation is in #19.
- **#15** Deferred-section list ≠ comprehensive audit.
- **#16** Drill-destination resolved by canonical PRD, not Build Sheet alone.
- **#17** Dual tap targets per UI element missed.
- **#18** Permission-driven action hiding is a Truth, not a per-row note.
- **#19 (NEW DA-01 v0.5.3) — Drill maps must not become Build Sheet duplicates.** v0.5.2 enumerated 27 non-tappable rows (51% of total) including QA invariants, sort orders, color rules, visual styles, render rules — all of which already live in Build Sheet per-section component tables. This cluttered the drill map and distorted its status summary. **Mitigation:** drill maps enumerate ONLY (a) tap targets, (b) situational display states that affect operator UX (e.g., zero-data placeholders), (c) section visibility rules. Everything else — display labels, QA invariants, sort orders, visual specs, color rules — stays in Build Sheet, optionally cross-referenced from drill map section headers as "Also rendered (display only): …". Lean drill map = ~22-35 rows per ~8-section audit, not 50+.

---

## Changelog

| Date | Version | Change |
|------|---------|--------|
| 2026-05-18 | v0.1 | Initial §1-§4. 17 rows. |
| 2026-05-18 | v0.2 | Round-1 sub-agent verification: 7 findings applied. |
| 2026-05-18 | v0.3 | Round-2 dual sub-agent PASS; later found to have 4C+3H+3M missed. |
| 2026-05-18 | v0.4 | REDRAFT §1-§4 with personal PM verification. 4 NEW anti-patterns (#11-#14). 26 rows. |
| 2026-05-18 | v0.5 | Expanded to §1-§8 per phased plan. ~49 rows. 3 new anti-patterns (#15-#17). |
| 2026-05-18 | v0.5.2 | Full-polish patch after round-3 three-angle audit. Truth #8 permissions, status math fix, legend acronyms, code-fragment translation, 3 HIGH coverage gaps, Architectural Truths plain-English split, row #13 note trim, bulleted walls. 53 rows. Anti-pattern #18. |
| 2026-05-19 | v0.5.3 | **Lean restructure — drill map enumerates ONLY tap-relevant rows.** Display labels folded into section header "Also rendered (display only): …" notes. QA invariants, sort orders, visual styles, color rules, render rules removed from drill map and left to Build Sheet ownership (cross-referenced where helpful). 53 rows → 33 rows (26 tap targets + 5 situational display states + 2 section visibility rules). Reader experience: designer scans tap targets in ~2 minutes. Anti-pattern #19 codified; #14 superseded. Going forward: same lean structure for v0.6 (§9-§12 + chrome) AND DA-02/03/04/05/06 from kickoff. |
