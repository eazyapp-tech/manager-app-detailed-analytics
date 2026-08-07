---
title: DA-06 Build Sheet
date: '2026-05-08'
tags:
  - rentok
  - prd
  - build-sheet
  - engineering
  - homescreen
  - financial-insights
  - detailed-analytics
  - liabilities
aliases:
  - DA-06 Build Sheet
  - DA-06 Dev Tasks
  - DA-06 Engineering Tasks Table
status: V1 — locked
version: '1.0'
for: Engineering ticket creation
companion_to: DA-06 Liabilities Detailed Analytics
---

> [!INFO] Source of Truth
> **Companion to:** [[DA-06 Liabilities Detailed Analytics]] — canonical "why" doc (V1.4)
> **Reference:** [[_Ground Truth Field Map]] — entity fields, formulas, endpoints, permissions
> **Format spec:** [[_Build Sheet Generation Spec]] — column conventions, plain-English rules, status values
> **For:** Engineers picking up tickets

# DA-06 — Build Sheet (for Engineering) — V1

Each row in a section table = one thing to build. Status column tells you whether to extend, build, compose, or wire UI. Plain columns are for PM / CS / QA review; Tech columns are for implementation.

For "why" or rationale, see [[DA-06 Liabilities Detailed Analytics]].
For entity fields, formulas, permissions, endpoints, see [[_Ground Truth Field Map]].
For format conventions, see [[_Build Sheet Generation Spec]].

> **DA-06 is fully a Live snapshot screen.** Every section ignores the global time filter (the chip is rendered for cross-tab consistency only). The Trend Chart has its own 6M / This Month toggle. Most rows therefore say "Same as live (always-live)" for past period and future contexts in their Plain Drill cells.

---

## Glossary

Plain-English definitions of terms used across this Build Sheet. Read before scanning rows.

| Term | Plain English |
|------|---------------|
| **paise** | Indian smallest currency unit. ₹1 = 100 paise. Always store amounts as integer paise, never as floats |
| **pg_id** | Property identifier — every record (invoice, tenant, payment, etc.) belongs to one property identified by `PG<n>` |
| **Held amount** | Money the operator currently holds for a tenant — sum of paid Security Deposit invoices + paid Caution Money invoices + paid Advance invoices, minus any refunds already processed against those invoices, plus net applicable credits. The screen's central concept |
| **Settlement** | The act of finalizing a moved-out tenant's books — checklist locked, deposit adjusted against any pending dues, refund (if any) processed |
| **Move-Out Forecast** | Tenants whose `date_of_eviction` falls in the next 30 days AND who currently hold a deposit. Treated as near-term cash outflow risk |
| **Refunds Pending** | Tenants who have already exited (`tenant.status NOT IN (1, 2)`) but still have held money on the books. Computed by deduction — there is no `refund_status` field |
| **Live** | Section ignores the date filter — always shows current state (always-live). DA-06 is fully Live. |
| **Class 1 / always-live** | Element doesn't change with date filter (every DA-06 component except the Trend Chart) |
| **Snapshot pill** | Static "Snapshot — updated live" chip in the header. Informational, not tappable |
| **Anomaly callout** | Conditional hero callout — fires when EITHER (a) a tenant's deposit has been held > 12 months without ledger activity, OR (b) tenant's held amount ≥ 2.5× property average AND ≥ ₹25K AND property has ≥ 5 active deposit-holders (maturity gate) |
| **Settlement-readiness flag** | Per-tenant badge — ✅ when `tenant_checklist.status = 6` (MOVE_OUT_APPROVED), ⏳ otherwise. Parent checklist row state, NOT item-level timestamps |
| **Long-pending badge** | Row badge "Possibly settled off-system?" — fires when `days_since_exit > 90 AND held_remaining > 0` regardless of partial refunds |
| **Aging bucket** | Days since exit, grouped: 0–7 (recent) / 8–14 (approaching delay) / 15–21 (delayed) / 22+ (significantly overdue, reputation risk) |
| **Open Settlement** | Composite action that chains 5 endpoints (`/checklist/move-out/items` → `mark-item` → `complete-checklist` → `/tenant/adjustDeposit` → invoice creation). Not an aggregate endpoint |
| **Process Refund** | Cross-screen drill into `POST /refunds/advanced-addition` — creates a Refund row that immediately reduces liability |
| `tenant.status` | **0** = old/checked-out, **1** = active, **2** = booking, **3** = lead, **4** = invitation/joining-request, **5** = permanently deleted, **6** = invitation-removed/accepted, **7** = soft-deleted lead, **8** = interested (self-service). Per wiki TEN-001 |
| `invoice.status` | **0** = Not Paid, **1** = Paid, **2** = Partially Paid, **3** = Refunded, **4** = Loss. Per `src/entities/invoices.ts:93` |
| `tenant_checklist.status` | **0** MOVE_IN_NOT_STARTED, **1** MOVE_IN_TENANT_COMPLETED, **2** MOVE_IN_LOCKED, **3** MOVE_IN_REOPENED, **4** MOVE_OUT_TENANT_COMPLETED, **5** MOVE_OUT_TEAM_COMPLETED, **6** MOVE_OUT_APPROVED, **7** NEXT_MOVE_IN_READY. Per `src/entities/tenantChecklist.ts:19-28` |
| `payment_mode = 211` | DEPOSIT_PAYMENT_MODE — deposit-applied (paper transfer from one deposit to another). Not real cash; must be excluded from canonical liability sum |
| `payment_mode = 288` | ADVANCE_PAYMENT_MODE — advance-applied. Excluded from collections grand total in DA-02 |
| `due_type` constants | `DEPOSIT_DUE_TYPE = 'Security Deposit'`; `DEPOSIT_DUE_TYPES = ['Security Deposit', 'Caution Money']`; `ADVANCE_DUE_TYE = 'Advance'` (typo "TYE" verbatim in source). Per `src/services/payment/constants.ts:5-7` |

---

## Engineering Architecture Reference

| Concern | Path / Reference |
|---------|------------------|
| List endpoint | `POST /v1/list_screens/liabilities/list/filters` — **NEW BUILD** (mirror `src/v1/list_screens/expenses/`) |
| Widget endpoint | `src/v1/list_screens/liabilities/service.ts:getLiabilitiesWidget` — **NEW BUILD** |
| Helpers (query builder) | `src/v1/list_screens/liabilities/helpers.ts` — **NEW BUILD** (mirror expenses pattern) |
| Filter codes | `src/v1/constants/filterCodes.ts` — `LiabilityFilterCode` enum, range **1700-1799 (NOT BUILT — reserve)** |
| Per-tenant deposit calc (correct) | `src/helpers/payments.ts:593-616` — `getSecurityDepositAmount` (excludes mode 211 via `payment_outflow`) — **EXISTS** |
| Per-tenant advance calc (correct) | `src/helpers/payments.ts:667+` — `getNetAdvanceAmount` — **EXISTS** |
| Per-tenant invoice-driven advance | `src/services/payment/paymentService.ts:2820-2858` — `getAvailableAdvance` — **EXISTS** |
| Per-tenant invoice-driven deposit | `src/services/payment/paymentService.ts:2860-2899` — `getAvailableDeposit` — **EXISTS** ⚠️ uses `DEPOSIT_DUE_TYPE` (singular = 'Security Deposit') only — does NOT include Caution Money |
| Per-tenant credits | `src/services/payment/paymentService.ts:1043-1122` — `getApplicableCredits` — **EXISTS** |
| Canonical homepage liability SQL | `src/v1/homepage/service.ts:1909-1922` — **EXISTS** (does NOT exclude mode 211 — see HB1/HB3) |
| Buggy variant — do not use | `src/helpers/payments.ts:619-645` — `getCumulativeSecurityDepositAmount` (mixes paid + unpaid) |
| Broken advance getter | `src/helpers/payments.ts:646-665` — `getAdvanceAmount` (returns 0 unconditionally) |
| Eviction-time deposit calc | `src/services/tenant/evictionService.ts:497-548` — **EXISTS** |
| Move-Out checklist endpoints | `src/routes/moveInMoveOutRoutes.ts:63-77` — `GET /checklist/move-out/items`, `PUT /checklist/move-out/mark-item`, `POST /checklist/move-out/complete-checklist` (all HeaderValidator-protected) |
| TenantChecklist parent entity | `src/entities/tenantChecklist.ts` — `status` enum 0-7; `MOVE_OUT_APPROVED = 6` is the canonical settlement-complete indicator |
| Refund creation endpoint | `POST /refunds/advanced-addition` — `src/routes/refunds.ts:9` (HeaderValidator-protected) |
| Adjust Deposit destination | `POST /tenant/adjustDeposit` — `src/routes/tenant.ts:944` ⚠️ **financial mutation, zero auth (CSB-3 P0)** |
| Tenant Detail destination | `POST /tenant/getTenantData` — `src/routes/tenant.ts:927` ⚠️ **zero HeaderValidator (CSB-3)** |
| Invoice Detail destination | `POST /invoices/getInvoiceData` — `src/routes/invoices.ts:215` ⚠️ **zero HeaderValidator (CSB-3)** |
| Excel report endpoint | `POST /reports/liability-report` — **NEW BUILD** (kebab-case per recent convention; mirror `/reports/cashflow-report`) |
| Property setting (refund block) | `is_refund_deposit_blocked_until_all_checklists_locked` — `src/controllers/property.ts:9326` |
| Production "under_notice" pattern | `src/v1/homepage/service.ts:798-803` — uses `t.status IN (1,2) AND t.date_of_eviction IS NOT NULL`, NOT `t.under_notice` boolean |
| Tenant entity | `src/entities/tenant.ts` — see Field Map §1.5 |
| Invoices entity | `src/entities/invoices.ts` — see Field Map §1.1 |
| Refunds entity | `src/entities/refunds.ts` — see Field Map §1.3 (no `is_active`, no `status`) |
| Constants | `src/services/payment/constants.ts:5-7` — `DEPOSIT_DUE_TYPE / DEPOSIT_DUE_TYPES / ADVANCE_DUE_TYE / DEPOSIT_PAYMENT_MODE / ADVANCE_PAYMENT_MODE` |
| Permission keys | Field Map §3 |
| Migrations | TBD (HB4 may need `liability_snapshots` table) |
| Dead code | `liabilitySum` in `src/controllers/reports.ts:7495` — must be cleaned up pre-launch |

---

## Cross-Suite Blockers Affecting This Spec

Surfaced from `[[DA-06 Liabilities Detailed Analytics#Cross-Suite Engineering Blockers]]` (V1.4).

| ID | Issue (one-line) | Status | Affects |
|----|------------------|--------|---------|
| ⛔ CSB-1 | `checkAuth` returns true on missing Bearer header — entire list endpoint reachable anonymously | Jatin gate | All sections (every auth-protected endpoint) |
| ⛔ CSB-3 | `/tenant/getTenantData`, `/tenant/adjustDeposit`, `/invoices/getInvoiceData` — drill destinations have zero HeaderValidator. **`/tenant/adjustDeposit` is financial mutation** (P0) | Jatin gate (P0 for adjustDeposit) | §3 (Move-Out Forecast row), §4 (Refunds Pending row), §6 (Worklist row), Open Settlement composite |
| ⛔ CSB-4 | DA-03 detail service has HeaderValidator but NO inline `checkAuth` — any authenticated user can read any refund by uuid | Jatin gate | §3, §4 (View Linked Refund) |
| CSB-5 | Expired-JWT bypass — stolen Manager App token authenticates indefinitely | App-hygiene fast-follow | All authenticated rows |
| CSB-6 | JWT permission-key shape diverges (11 vs 12 keys) | App-hygiene | Permission gating |
| CSB-7 | Refund/Reimbursement controller duplication — auth fix in one silently missed in the other | App-hygiene | Refund creation flow |
| ⛔ CSB-8 (NEW 2026-05-11) | **Widget endpoint permission enforcement gap (security).** Pattern from `getDuesWidget` (`dues/service.ts:132-138`) — set `self_added_team_uuid` but never apply. DA-06 NEW BUILD widget endpoints (held amount, move-out forecast) MUST enforce self-added filter from day 1 | Engineering (build correctly) | All NEW liability widget endpoints |

---

## Hard Blockers (HB)

Surfaced from PRD's Pre-Launch Engineering Blockers (V1.4).

| ID | Issue | Status |
|----|-------|--------|
| HB1 | Aggregation source-of-truth unification — three divergent paths produce different "money owed" numbers. Wrap canonical query in `LiabilityService.getDepositsHeld()` and route all callers through it | Jatin gate (architectural) |
| HB2 | Move-out exit date precedence rule — three candidate fields, no canonical source. Engineering picks: `tenant.date_of_eviction` if set, else lease `end_date`, else nothing | Jatin gate (data-source ambiguity) |
| HB3 | `payment_mode = 211` filter on canonical liability query — phantom liability fix. Add filter; verify total drops appropriately | Jatin gate (canonical query) |
| HB4 | Trend chart historical data — Option A (backfill cron) vs Option B (on-demand re-derivation). Cannot ship without one chosen | Jatin gate (architectural) |
| HB5 | `tenant.notice_period_started_date` field confirmed or notice-window chip removed | Jatin gate (schema dependency) |
| HB6 | `liabilitySum` dead variable cleanup at `src/controllers/reports.ts:7495` | Engineering cleanup |
| **HB7 (NEW V1 Build Sheet finding)** | **`getAvailableDeposit` at `paymentService.ts:2860-2899` filters on `DEPOSIT_DUE_TYPE` (singular = 'Security Deposit') — does NOT include Caution Money.** PRD scope says liability = SD + CM. Either widen function to use `DEPOSIT_DUE_TYPES` (plural) OR carry the gap forward and document | Jatin/PM gate |
| **HB8 (NEW V1 Build Sheet finding)** | **No production query computes "tenant has held amount > 0 in batch"** — current per-tenant composite is on-demand. Worklist needs a NEW aggregated query that runs `available_deposit + available_advance + applicable_credits > 0` efficiently across all tenants in a property | Engineering (blocks §3, §4, §6) |
| **HB9 (NEW Phase 4 audit). `Credits.amount` unit ambiguity in held-amount composite** | DA-06's per-tenant held = `available_deposit + available_advance + applicable_credits`. The first two are **paise** (from invoice ledgers via `paymentService.ts`). `applicable_credits` is sourced from `Credits.amount` which **DA-05 HB5 declares is integer rupees**, NOT paise (entity declares as plain `number`, no `decimal` type — likely int per TypeORM default). **Without explicit unit normalization, DA-06's hero is off by 100× on the credits portion** — silent financial display error. Either (a) multiply `Credits.amount × 100` before adding to deposit/advance paise, OR (b) confirm via codebase audit that producer code already writes paise to `Credits.amount` (in which case DA-05 HB5 is wrong and should be retracted). Cross-spec: must align with DA-05 HB5 resolution. | DA-06 §1 (held-amount composite) + DA-05 HB5 cross-reference |

> **Back-port to PRD:** add HB7 and HB8 entries to [[DA-06 Liabilities Detailed Analytics#Pre-Launch Engineering Blockers]] under the HARD blockers section.

---

## Pre-Build Decisions Pending (Jatin)

Must be answered before Phase 1 begins:

| Decision | Recommendation | Owner |
|----------|----------------|-------|
| Fix CSB-1 (auth bypass) — patch HeaderValidator OR fix checkAuth fail-closed | Patch HeaderValidator to 401 on missing header (cleaner) | Jatin |
| Fix CSB-3 (`/tenant/adjustDeposit` zero auth + financial mutation) | Mandatory before Open Settlement composite can ship — P0 | Jatin |
| Fix CSB-4 (refund detail no `checkAuth`) — add `checkAuth(viewInvoices)` | Mandatory before Process Refund drill ships | Jatin |
| HB1 — unify aggregation in `LiabilityService.getDepositsHeld()` | Architecturally correct; pick Path C (homepage SQL) as canonical, ADD mode 211 filter, retire Path B | Jatin |
| HB2 — Move-out exit date precedence: `tenant.date_of_eviction` if set, else lease `end_date`, else nothing | Document in code comment + use throughout | Jatin |
| HB3 — add `payment_mode != 211` filter to canonical aggregator | Mandatory; verify total drops | Jatin |
| HB4 — Trend chart historical source: Option A (backfill cron) vs Option B (on-demand) | Option A (backfill cron + new `liability_snapshots` table); start fresh from launch | Jatin |
| HB5 — `tenant.notice_period_started_date` field exists or chip cut | If field doesn't exist, defer chip to v1.2 / Phase 2 | Jatin |
| HB7 (NEW) — `getAvailableDeposit` should use `DEPOSIT_DUE_TYPES` (plural) to include Caution Money | Widen the function; otherwise CM is silently dropped from per-tenant held amounts | Jatin |
| HB8 (NEW) — Build aggregated batched-tenant held-amount query for worklist | Required to back the §3/§4/§6 lists; spec the query shape during Phase 1 spike | Jatin |
| `viewLiabilities` permission cited in PRD does not exist — reuse `viewInvoices` | Yes — composite view, no new key needed | Jatin |
| `processRefunds` cited in PRD does not exist — use DB column `add_refund_access` via `checkAuthInDb` | Yes — exists in `team_member_property` | Jatin |
| `processMoveOuts` cited in PRD does not exist — use DB columns `submit_or_update_moveout_checklist_access`, `approve_moveout_checklist_access`, `edit_eviction_access`, `cancel_eviction_access` (per-action) | Yes — pick the right column per button | Jatin |
| `viewTenantDetails` cited in PRD does not exist — use `viewTenants` JWT key | Yes — `viewTenants` is the real key | Jatin |

---

## 1. Hero — Total Held + Snapshot Pill + ⓘ

> **Endpoint:** EXTEND/BUILD — canonical aggregator `src/v1/homepage/service.ts:1909-1922` exists; needs HB1/HB3 fixes (mode 211 exclusion, unify in `LiabilityService.getDepositsHeld()`); per-tenant composite breakdown is NEW
> **Default property context:** inherited from dashboard (single → single; multi → multi-property aggregate, header shows "across N properties")
> **Default time context:** **always-live** — Hero ignores the date filter (Class 1)
> **Cross-Suite Blockers:** CSB-1 (HeaderValidator bypass)
> **Hard Blockers:** HB1 (aggregation source-of-truth), HB3 (mode 211 filter), HB7 (`getAvailableDeposit` Caution Money gap)
> **Eng note:** all amounts in **paise** (integer) — EXCEPT `applicable_credits` which sources from `Credits.amount` declared as plain `number` (likely int rupees per DA-05 HB5). Held = `available_deposit + available_advance + applicable_credits` per tenant; aggregate is the SUM across active + moved-out-with-held tenants. **⚠️ Cross-spec unit normalization required (HB9):** before adding `applicable_credits` to the paise-based deposit/advance subtotals, either multiply by 100 OR confirm producer code writes paise (audit during spike). Production formula in PRD §Reconciliation Invariants: `SUM(inv.amount − SUM(refunds.amount per inv)) WHERE due_type IN ('Security Deposit', 'Caution Money') AND inv.status = 1 AND inv.amount > 0 AND payment_mode != 211`. **⚠️ Currently the homepage SQL omits `payment_mode != 211`** — see HB1/HB3. **⚠️ `getAvailableDeposit` uses singular `DEPOSIT_DUE_TYPE` only, dropping Caution Money** — see HB7.
> **⚠️ PRD vs codebase gap:** PRD invariant unions SD + CM under one "Held" number, but `getAvailableDeposit` at `paymentService.ts:2860` filters on `DEPOSIT_DUE_TYPE` (singular) — Caution Money is silently dropped from per-tenant breakdowns. See HB7.
> **Filter codes:** `LiabilityFilterCode` 1700-1799 NOT BUILT — reserve range.

| Component | Status | Task | Definition + [class] | Plain Formula | Formula (technical) | Permission | Plain Drill | Drill Behavior (technical) |
|-----------|--------|------|----------------------|---------------|---------------------|------------|-------------|----------------------------|
| Hero | EXTEND + [VERIFY] | Total Held number | Live total of money the operator currently holds for tenants. [always-live] | The total amount of deposit + caution money + advance balances the operator currently holds across the user's selected properties, minus any refunds already processed. This number does NOT change when the user picks a different time period. **[VERIFY]** confirm canonical aggregator excludes mode 211 (HB3) and unions SD + CM (HB7) before launch. | `SUM(inv.amount − SUM(refunds.amount per inv)) WHERE inv.due_type IN ('Security Deposit', 'Caution Money') AND inv.status = 1 AND inv.amount > 0 AND payment_mode != 211` *(`inv` = `invoices` table; `due_type` = string column with values like 'Security Deposit'/'Caution Money'/'Advance'/'Rent'; `inv.status=1` = Paid per entity comment; `payment_mode = 211` = DEPOSIT_PAYMENT_MODE per `src/services/payment/constants.ts:4` — paper-transfer mode that creates phantom liability and must be excluded; `refunds` = one-to-many child of Invoices, no `is_active` and no `status` columns — every refund row is final)* — currently at `src/v1/homepage/service.ts:1909-1922` (missing the mode 211 filter). **NEW BUILD:** wrap in `LiabilityService.getDepositsHeld(propertyIds)` per HB1. | view: `can_view_invoices` (PRD cites fictional `viewLiabilities`; reuse `viewInvoices` per Field Map §3.4) | → Live (today): Tap the hero block to open the SD vs CM vs Advance breakdown bottom sheet; tap any segment to drill into the worklist filtered to that composition. ⓘ icon: tap = inline tooltip; long-press = full GAAP framing.<br>→ Past period: Same as live (always-live) — Liabilities does not respect the global time filter; the hero number stays current.<br>→ Future: Same as live (always-live). | → Live: BS (composition breakdown) on body tap · T (inline) on ⓘ tap · BS (full definition) on ⓘ long-press · "Held composition"<br>→ Past period: SAME (always-live)<br>→ Future: SAME (always-live) |
| Hero | EXISTS | Tenant count subtitle | Count of tenants currently held against. [always-live] | Show how many separate tenants make up the total. For example, "across 28 tenants." | `COUNT(DISTINCT tenant_id) WHERE held_amount(t.id) > 0` over the canonical liability universe | view: inherits hero visibility | Display only — not tappable. | n/a — display only |
| Hero | BUILD | Move-out forecast count subtitle | "X exiting in next 30 days" inline subtitle. [always-live] | Append a quick subtitle showing how many of those tenants are exiting in the next 30 days. For example, "across 28 tenants · 3 exiting in next 30 days". | `COUNT(t.id) WHERE t.status = 1 AND t.date_of_eviction IS NOT NULL AND t.date_of_eviction::date BETWEEN today AND today + 30 AND held_amount(t.id) > 0` *(production "under-notice" pattern uses `date_of_eviction IS NOT NULL`, NOT `t.under_notice` boolean — see Field Map §1.5; `held_amount(t.id)` = the per-tenant composite from `paymentService.ts:getAvailableDeposit + getAvailableAdvance + getApplicableCredits`)* | view: inherits hero visibility | Display only. | n/a — display only |
| Hero | BUILD | Multi-property suffix | Append "across N properties" when multi-property dashboard. [always-live UI render] | When the user has more than one property selected, add the property count to the subtitle. | append `· across <count> properties` if `selected_properties > 1` (frontend render) | view: inherits hero visibility | Display only. | n/a — display only |
| Hero | BUILD | ⓘ icon | Tap = inline tooltip; long-press = bottom sheet with full plain-language framing. [always-live tooltip text] | Tap shows brief plain-English tooltip ("Money you're holding for tenants…"). Long-press shows expanded framing. | n/a — UI only. Tooltip text in §10 microcopy. | view: inherits hero visibility | → Live: Tap for quick tooltip; long-press for expanded definition.<br>→ Past period: Same as live (always-live).<br>→ Future: Same as live (always-live). | → Live: T (inline) on tap · BS on long-press<br>→ Past period: SAME<br>→ Future: SAME |
| Hero | BUILD | Header label "Liabilities" | Static section header. Never changes with filter. [always-live] | The screen's title. Plain "Liabilities" — no "(Live)" suffix (DA-01 uses that pattern with a Live/Range toggle; DA-06 has no toggle). | Static string `"Liabilities"` + ⓘ icon | view: inherits hero visibility | Display only — header text. | n/a — display only |
| Snapshot pill | BUILD | "Snapshot — updated live" pill | Static informational pill below header. [always-live UI render] | A small pill that reinforces this screen ignores the date filter. Visible always; not tappable. | Static string `"Snapshot — updated live"`. Render-rule: always visible. | view: inherits hero visibility | Display only — not tappable. | n/a — UI render only |
| Hero breakdown | BUILD | SD / CM / Advance segments | Tap on hero opens composition bottom sheet. [always-live] | Tap on the hero opens a bottom sheet showing the Security Deposit total, the Caution Money total, and the Advance balances total — three numbers that add to the hero. Each segment is tappable. | Three segment SUMs over the canonical universe: `SUM(inv.amount - refunds) WHERE due_type='Security Deposit'`, same for `'Caution Money'`, and `getAvailableAdvance` aggregated across tenants. **NEW BUILD** | view: inherits hero visibility | → Live (today): Tap a segment to open the worklist filtered to that composition (e.g., tap "Caution Money" → worklist with `composition=cm` chip).<br>→ Past period: Same as live (always-live).<br>→ Future: Same as live (always-live). | → Live: WL · filters: `composition=<sd\|cm\|advance>` · sort: held_amount DESC · "All Liabilities · [Composition] · N · ₹X"<br>→ Past period: SAME<br>→ Future: SAME |

> **Note:** No MoM chip on this screen. Liabilities is a snapshot — month-over-month comparison is misleading. Trajectory is shown via the Trend Chart (Section 7).

---

## 2. Hero Callouts (conditional) + Chips

> **Endpoint:** BUILD — each callout is a conditional render driven by query results from `LiabilityService.getDepositsHeld()` + per-tenant composite. NEW
> **Default property context:** inherited from dashboard
> **Default time context:** **always-live** — every callout reads "right now" data
> **Cross-Suite Blockers:** CSB-3 (Tenant Detail destination unauth, drill into anomaly tenant)
> **Hard Blockers:** HB5 (notice-window chip schema dependency — chip hidden if `tenant.notice_period_started_date` doesn't exist)
> **Eng note:** Callouts render only when their condition fires. Stack vertically below the hero in priority order (move-out forecast first, then anomaly, then refunds-overdue). Advance + notice-window chips render below callouts when count/amount > 0.

| Component | Status | Task | Definition + [class] | Plain Formula | Formula (technical) | Permission | Plain Drill | Drill Behavior (technical) |
|-----------|--------|------|----------------------|---------------|---------------------|------------|-------------|----------------------------|
| Move-Out Forecast callout | BUILD | "Up to ₹X exiting in next 30 days · N tenants" | Conditional callout when next-30-day forecast amount > 0. [always-live] | Show an amber callout when any tenant who's exiting in the next 30 days currently has held money. The number is "up to" because exit dates are soft. CTA "View" jumps to the Move-Out Forecast section. | Render condition: `EXISTS (t WHERE t.status=1 AND t.date_of_eviction::date BETWEEN today AND today+30 AND held_amount(t.id) > 0)`. Amount: `SUM(held_amount(t.id))` over that set. *(See Field Map §4.7 for the production under-notice pattern; `held_amount(t.id)` = composite from `paymentService.ts:getAvailableDeposit/Advance/Credits`.)* **NEW QUERY** | view: inherits hero visibility | → Live (today): Tap "View" CTA to scroll to the Move-Out Forecast section (§3). The callout itself opens the worklist filtered to `exit_within=30d`.<br>→ Past period: Same as live (always-live).<br>→ Future: Same as live (always-live). | → Live: WL · filters: `exit_within=30d` (chip removable) · sort: exit_date ASC · "Exiting in 30 days · N · ₹X"<br>→ Past period: SAME<br>→ Future: SAME |
| Anomaly callout | BUILD + [VERIFY] | "₹X deposit held for [Tenant] — unusual for this property" | Conditional callout when a single tenant trips the v1.4 reconciled anomaly definition. [always-live] | Show an amber callout when EITHER (a) a tenant's deposit has been held > 12 months without any ledger activity, OR (b) the tenant's held amount is ≥ 2.5× the property's average AND ≥ ₹25K AND the property has at least 5 tenants holding deposits (maturity gate). **[VERIFY]** PRD v1.4 reconciles two earlier definitions — ENG must implement the OR; do not pick one. | Either of two conditions:<br>(a) `MAX(t.held_since) - now > 365 days AND no_ledger_activity(t.id, last 365 days)` — needs new "ledger activity" definition (likely: no Payments / Refunds / Credits rows touching this tenant in 365 days)<br>(b) `t.held_amount >= 2.5 * property_avg AND t.held_amount >= 25000 AND COUNT(deposit_holders_in_property) >= 5` — GROUP BY tenant per property; compute property average; flag top-tenant per property if all 3 hold. **NEW QUERIES** | view: inherits hero visibility | → Live (today): Tap "View" CTA to open the worklist with `anomaly=1` chip + sorted by `days_held DESC`. The flagged tenant is at the top.<br>→ Past period: Same as live (always-live).<br>→ Future: Same as live (always-live). | → Live: WL · filters: `anomaly=1` (chip) · sort: days_held DESC · "Anomaly · 1 · ₹X"<br>→ Past period: SAME<br>→ Future: SAME |
| Refunds-overdue callout | BUILD | "₹X owed to ex-tenants for 21+ days · N tenants" | Conditional callout when any moved-out tenant has held > 0 AND days_since_exit > 21. [always-live] | Show an amber callout when any tenant who's already left still has held money AND has been waiting more than 21 days. Reputation risk threshold (recalibrated v1.1 from 30 → 21 days). | `EXISTS (t WHERE t.status NOT IN (1,2) AND held_amount(t.id) > 0 AND (today - t.date_of_eviction) > 21)`. Amount: `SUM(held_amount(t.id))` over that set. **NEW QUERY** | view: inherits hero visibility | → Live (today): Tap "Review" CTA to open the worklist filtered to refunds-overdue, sorted oldest first.<br>→ Past period: Same as live (always-live).<br>→ Future: Same as live (always-live). | → Live: WL · filters: `tenant_status NOT IN (1,2), held > 0, days_since_exit > 21` · sort: days_since_exit DESC · "Refunds Overdue · N · ₹X"<br>→ Past period: SAME<br>→ Future: SAME |
| Advance balance chip | BUILD | "+ ₹X advance balances held" | Conditional chip when total advance > 0. [always-live] | Show a small neutral chip when there is any held advance balance across tenants. Most operators have ₹0 — the chip surfaces only when relevant. | Render condition: `SUM(getAvailableAdvance per tenant) > 0`. Amount: same SUM. | view: inherits hero visibility | → Live (today): Tap chip to open worklist filtered to tenants with advance balance > 0, sorted by advance amount.<br>→ Past period: Same as live (always-live).<br>→ Future: Same as live (always-live). | → Live: WL · filters: `advance_amount > 0` · sort: advance_amount DESC · "Advance Balances · N · ₹X"<br>→ Past period: SAME<br>→ Future: SAME |
| Notice-window chip | BUILD + [VERIFY] | "N tenants in notice period · ₹X deposit held" | Conditional chip when any tenant is in their notice window AND has held money. [always-live] | Show a chip surfacing tenants who have given notice (notice period started but not yet exited) AND who hold a deposit. **[VERIFY]** depends on `tenant.notice_period_started_date` field (HB5). If field doesn't exist, chip is hidden in v1.2. | Render condition (HB5-gated): `EXISTS (t WHERE t.notice_period_started_date IS NOT NULL AND t.notice_period_started_date <= today AND t.status = 1 AND held_amount(t.id) > 0)`. **NEW QUERY + schema dependency** | view: inherits hero visibility | → Live (today): Tap chip to open worklist filtered to tenants in notice period.<br>→ Past period: Same as live (always-live).<br>→ Future: Same as live (always-live). | → Live: WL · filters: `notice_started=true` · sort: notice_started_date ASC · "In Notice · N · ₹X"<br>→ Past period: SAME<br>→ Future: SAME |

---

## 3. By Property (multi-property only)

> **Endpoint:** BUILD — aggregator at property granularity with the next-30-day exits column
> **Default property context:** **OVERRIDE** — each row scopes to a single property
> **Default time context:** **always-live** — counts and amounts reflect right now
> **Cross-Suite Blockers:** CSB-1 (auth bypass)
> **Eng note:** This section renders only when `selected_properties > 1`. Each row aggregates its property's held amount, next-30-days exit amount + count, refunds-pending amount + count, and active deposit-holder count. Sort: `held_amount DESC`. Color cue on "Exiting in next 30 days" column: < 5% neutral, 5–15% amber, > 15% red.

| Component | Status | Task | Definition + [class] | Plain Formula | Formula (technical) | Permission | Plain Drill | Drill Behavior (technical) |
|-----------|--------|------|----------------------|---------------|---------------------|------------|-------------|----------------------------|
| By Property | BUILD | Per-property held amount | Each property's total held. [always-live] | For each property the user has selected, show how much is held in total there. | `SUM(held_amount(t.id)) GROUP BY t.property_id` over the canonical universe. **NEW QUERY** | view: `can_view_invoices` (composite — same as hero) | → Live (today): Tap a row to open the worklist scoped to that property.<br>→ Past period: Same as live (always-live).<br>→ Future: Same as live (always-live). | → Live: WL · property: scoped to that property · filters: `pg_id=X` · sort: held_amount DESC · "[Property Name] · N · ₹X"<br>→ Past period: SAME<br>→ Future: SAME |
| By Property | BUILD | Exiting in next 30 days column | Per-property next-30-day exit amount + tenant count. [always-live] | For each property, show how much money is exiting in the next 30 days and how many tenants. The most operationally critical column — color-cued (red when > 15% of that property's held). | `SUM(held_amount(t.id)), COUNT(t.id) WHERE t.status=1 AND t.date_of_eviction::date BETWEEN today AND today+30 AND held_amount(t.id) > 0 GROUP BY t.property_id` | view: inherits | → Live (today): Tap the column value to open the worklist filtered to that property + exit_within=30d.<br>→ Past period: Same as live (always-live).<br>→ Future: Same as live (always-live). | → Live: WL · filters: `pg_id=X, exit_within=30d` · sort: exit_date ASC · "[Property Name] · Exiting · N · ₹X"<br>→ Past period: SAME<br>→ Future: SAME |
| By Property | BUILD | Refunds pending column | Per-property refunds-pending amount + count. [always-live] | For each property, show how much is owed to ex-tenants and how many. | `SUM(held_amount(t.id)), COUNT(t.id) WHERE t.status NOT IN (1,2) AND held_amount(t.id) > 0 GROUP BY t.property_id` | view: inherits | → Live (today): Tap to open worklist filtered to that property + refunds-pending.<br>→ Past period: Same as live (always-live).<br>→ Future: Same as live (always-live). | → Live: WL · filters: `pg_id=X, tenant_status NOT IN (1,2), held > 0` · sort: days_since_exit DESC · "[Property Name] · Refunds Pending · N · ₹X"<br>→ Past period: SAME<br>→ Future: SAME |
| By Property | BUILD | Tenant count column | Active deposit-holders per property. [always-live] | Number of currently-active tenants who have any held money against them at this property. | `COUNT(DISTINCT t.id) WHERE t.status = 1 AND held_amount(t.id) > 0 GROUP BY t.property_id` | view: inherits | Display only. | n/a — display |
| By Property | BUILD | Proportional bar | Visual share of total liability. [always-live UI render] | A small horizontal bar showing each property's share of total liability. Width by amount. | `width = (property_held / total_held) × bar_width`; min 4px floor | view: inherits | n/a — UI render. | n/a — UI render only |

---

## 4. Move-Out Forecast (next 30 days)

> **Endpoint:** BUILD — list endpoint `POST /v1/list_screens/liabilities/list/filters` with filter code 1702 (Exiting 30 days)
> **Default property context:** inherited from dashboard
> **Default time context:** **always-live** — rolling 30-day window from today
> **Cross-Suite Blockers:** CSB-1 (auth bypass), CSB-3 (Tenant Detail + adjustDeposit unauth)
> **Hard Blockers:** HB2 (exit date precedence rule), HB7 (`getAvailableDeposit` Caution Money gap), HB8 (batched per-tenant held query)
> **Eng note:** Sorted by `exit_date ASC` (soonest first). Each row shows tenant name, status badge, exit date with countdown, deposit owed (held minus refunds), pending dues if any (operator may deduct), property name (multi-property), settlement-readiness flag. Empty state: "No tenants exiting in the next 30 days. Liability is stable for now." Footer total: "Total exiting: ₹X across N tenants."

| Component | Status | Task | Definition + [class] | Plain Formula | Formula (technical) | Permission | Plain Drill | Drill Behavior (technical) |
|-----------|--------|------|----------------------|---------------|---------------------|------------|-------------|----------------------------|
| Forecast row | BUILD | Tenant row body | Per-tenant row with name, status badge, exit date, deposit owed. [always-live] | For each tenant exiting in the next 30 days who has held money, render a row with their name, a status badge ("Notice given" / "Contract ending" / "Booked exit"), the exit date with countdown ("May 18 · 11 days"), and the deposit owed (held minus refunds already processed). | `SELECT t.id, t.name, t.status, t.date_of_eviction, held_amount(t.id) AS deposit_owed FROM tenant t WHERE t.status=1 AND t.date_of_eviction IS NOT NULL AND t.date_of_eviction::date BETWEEN today AND today + 30 AND held_amount(t.id) > 0 ORDER BY t.date_of_eviction ASC` *(Field Map §4.7 — production "under-notice" pattern uses `date_of_eviction IS NOT NULL`, NOT `t.under_notice` boolean — DA-06 must follow production for parity)* **NEW QUERY** | view: `can_view_invoices` AND `can_view_tenants` (composite) | → Live (today): Tap row body to open Tenant Detail full-screen push (existing app feature).<br>⚠️ The destination `/tenant/getTenantData` does not currently check user permissions — engineering must fix before launch (CSB-3).<br>→ Past period: Same as live (always-live).<br>→ Future: Same as live (always-live). | → Live: FS (Tenant Detail at `POST /tenant/getTenantData`) · property: scoped to tenant's pg · filters: `tenant_uuid=X` · "Tenant Detail"<br>⚠️ unauth (CSB-3)<br>→ Past period: SAME<br>→ Future: SAME |
| Forecast row | BUILD | Pending dues subline | "⚠️ ₹X dues. Net refund: ₹Y." conditional. [always-live] | If the tenant has any unpaid bills, surface them inline: "⚠️ ₹3K dues. Net refund: ₹47K." Operators may deduct dues from deposit at settlement. | Render condition: `SUM(i.amount) > 0 WHERE i.payer = t.firebase_id AND i.status = 0 AND i.is_active = 1` per tenant. Net = `held - dues`. | view: inherits | Display only — informational. | n/a — display |
| Forecast row | BUILD | Settlement-readiness flag | ✅ when checklist approved, ⏳ otherwise. [always-live] | Show ✅ when the tenant's move-out checklist has been approved (status = MOVE_OUT_APPROVED), or ⏳ if it's still pending. | `tenant_checklist.status = 6 (MOVE_OUT_APPROVED) AS ready_flag` per `src/entities/tenantChecklist.ts:19-28` *(parent-checklist row state is the canonical settlement-complete indicator — NOT item-level `move_out_marked_at` per Field Map §1.8)* | view: inherits | Display only — chip with inline tooltip on tap. | → Live: T (inline) on tap<br>→ Past period: SAME<br>→ Future: SAME |
| Forecast row | COMPOSE + [VERIFY] | "Open Settlement" action button | Composite — chains 5 endpoints. [always-live action] | Tap "Open Settlement" to start the move-out settlement flow: open the checklist, mark items, complete the checklist, adjust the deposit against any pending dues, create a settlement invoice. **[VERIFY]** No aggregate "compute settlement amount" endpoint exists; engineering must compose `/checklist/move-out/items` → `mark-item` → `complete-checklist` → `/tenant/adjustDeposit` → invoice creation. | **COMPOSE** — chains: (1) `GET /checklist/move-out/items` (`src/routes/moveInMoveOutRoutes.ts:63`); (2) `PUT /checklist/move-out/mark-item` (`:70`); (3) `POST /checklist/move-out/complete-checklist` (`:77`); (4) `POST /tenant/adjustDeposit` (`src/routes/tenant.ts:944`) ⚠️ **financial mutation, zero auth (CSB-3 P0)**; (5) invoice creation per existing flow. | view: `can_view_tenants` · action: `submit_or_update_moveout_checklist_access` AND `approve_moveout_checklist_access` (DB-side via `checkAuthInDb`; PRD's `processMoveOuts` does NOT exist — see Field Map §3.4) | → Live (today): Tap "Open Settlement" → opens checklist screen → operator marks items → completes checklist → adjusts deposit against pending dues → creates settlement invoice. On completion: row is removed from the forecast and the hero recomputes live.<br>⚠️ The deposit-adjustment step `/tenant/adjustDeposit` is a financial mutation with zero auth currently (CSB-3 P0). Must be fixed before launch.<br>→ Past period: Same as live (always-live).<br>→ Future: Same as live (always-live). | → Live: FS (Move-Out checklist) → cross-screen drill chain · property: scoped to tenant's pg · "Move-Out Settlement"<br>⚠️ `/tenant/adjustDeposit` unauth + financial mutation (CSB-3 P0)<br>→ Past period: SAME<br>→ Future: SAME |
| Forecast row | EXISTS | "Process Refund" action button | Conditional — fires when exit date has passed AND held > 0. [always-live action] | Show "Process Refund" only if the exit date has already passed AND the tenant still has held money. Tap routes to the existing Refund Creation flow. | Render condition: `t.date_of_eviction < today AND held_amount(t.id) > 0`. Routes to `POST /refunds/advanced-addition` (`src/routes/refunds.ts:9`) — HeaderValidator-protected. | view: `can_view_invoices` · action: `add_refund_access` (DB-side; PRD's `processRefunds` does NOT exist — see Field Map §3.4) | → Live (today): Tap "Process Refund" → opens refund creation form (existing flow). On completion: returns to DA-06 with row updated (held reduced or row removed if fully refunded). On cancel: returns untouched.<br>→ Past period: Same as live (always-live).<br>→ Future: Same as live (always-live). | → Live: FS (Refund Creation at `POST /refunds/advanced-addition`) · property: scoped to tenant's pg · "Process Refund"<br>→ Past period: SAME<br>→ Future: SAME |
| Forecast | BUILD | Footer total | "Total exiting: ₹X across N tenants." [always-live] | Below the list, a footer showing the total amount + tenant count. | `SUM(held_amount), COUNT(*)` over the section query | view: inherits | Display only. | n/a — display |
| Forecast | BUILD | Empty state | "No tenants exiting in the next 30 days. Liability is stable for now." [always-live UI render] | When zero tenants match the filter, render the empty-state copy. | Render when query returns 0 rows | view: inherits | Display only. | n/a — render rule only |

---

## 5. Refunds Pending (already exited)

> **Endpoint:** BUILD — list endpoint with filter code 1703 (Refunds Pending) + aging-bucket grouping
> **Default property context:** inherited from dashboard
> **Default time context:** **always-live** — `days_since_exit` computed from today
> **Cross-Suite Blockers:** CSB-1 (auth bypass), CSB-3 (Tenant Detail unauth)
> **Hard Blockers:** HB2 (exit date precedence — same field used here)
> **Eng note:** Buckets: 0–7 / 8–14 / 15–21 / 22+ days since exit (recalibrated v1.1). Sort within bucket: `days_since_exit DESC`. Long-pending badge fires at `days_since_exit > 90 AND held_remaining > 0` regardless of partial-refund history. Cross-screen note: "Once a refund is processed in DA-03, the entry leaves this list."

| Component | Status | Task | Definition + [class] | Plain Formula | Formula (technical) | Permission | Plain Drill | Drill Behavior (technical) |
|-----------|--------|------|----------------------|---------------|---------------------|------------|-------------|----------------------------|
| Pending row | BUILD | Tenant row body | Per-tenant row with name, days since exit, held minus refunds, status. [always-live] | For each tenant who has already left (`tenant.status NOT IN (1, 2)`) but still has held money, render a row with their name (or "Tenant record removed" if archived), how many days since they exited, the held amount + held-minus-refunds, the property (multi-property), and a status string ("Settlement pending" / "Awaiting checklist lock" / "Refund flow stalled"). | `SELECT t.id, t.name, t.status, t.date_of_eviction, (today - t.date_of_eviction) AS days_since_exit, held_amount(t.id) AS deposit_owed FROM tenant t WHERE t.status NOT IN (1, 2) AND held_amount(t.id) > 0 ORDER BY days_since_exit DESC` **NEW QUERY** | view: `can_view_invoices` AND `can_view_tenants` | → Live (today): Tap row to open Tenant Detail full-screen push.<br>⚠️ Destination `/tenant/getTenantData` is unauth (CSB-3).<br>→ Past period: Same as live (always-live).<br>→ Future: Same as live (always-live). | → Live: FS (Tenant Detail) · property: scoped to tenant's pg · filters: `tenant_uuid=X` · "Tenant Detail"<br>⚠️ unauth (CSB-3)<br>→ Past period: SAME<br>→ Future: SAME |
| Pending row | BUILD | Aging bucket header | Header per bucket: "N tenants · ₹X · oldest Y days." [always-live] | Above each bucket of rows, a header summarising the bucket: tenant count, total amount, oldest age. | Per-bucket aggregate over the section query. Buckets defined as `days_since_exit BETWEEN 0 AND 7`, `8 AND 14`, `15 AND 21`, `>= 22`. | view: inherits | → Live (today): Tap header to expand bucket worklist (filtered to that bucket).<br>→ Past period: Same as live (always-live).<br>→ Future: Same as live (always-live). | → Live: WL · filters: `tenant_status NOT IN (1,2), held > 0, days_since_exit IN [bucket]` · sort: days_since_exit DESC · "Pending [bucket] · N · ₹X"<br>→ Past period: SAME<br>→ Future: SAME |
| Pending row | BUILD | Long-pending badge | "Possibly settled off-system?" — fires at days_since_exit > 90 AND held_remaining > 0. [always-live] | Show a small row badge "Possibly settled off-system?" when a tenant has been exited > 90 days AND still has held money — the operator may have given cash without recording a refund. Fires regardless of partial-refund history. | Render condition: `(today - t.date_of_eviction) > 90 AND held_amount(t.id) > 0`. Tooltip on tap: surfaces guidance to either record the refund or mark it written-off (Phase 2 cleanup). | view: inherits | → Live (today): Tap badge for inline tooltip explaining the heuristic.<br>→ Past period: Same as live (always-live).<br>→ Future: Same as live (always-live). | → Live: T (inline) on tap<br>→ Past period: SAME<br>→ Future: SAME |
| Pending | BUILD | Empty state | "All refunds are processed. Nothing owed to ex-tenants." [always-live UI render] | When zero rows match, show celebratory empty state. | Render when query returns 0 rows | view: inherits | Display only. | n/a — render rule only |

---

## 6. Liability Worklist (drill destination)

> **Endpoint:** BUILD — `POST /v1/list_screens/liabilities/list/filters` (NEW); mirror `src/v1/list_screens/expenses/`
> **Default property context:** inherited from caller (single from By Property; multi from hero)
> **Default time context:** **always-live** — worklist is filtered to current state
> **Cross-Suite Blockers:** CSB-1 (auth bypass), CSB-3 (Tenant Detail + Adjust Deposit unauth), CSB-4 (refund detail no `checkAuth`)
> **Hard Blockers:** HB1, HB7, HB8 (all required for worklist accuracy)
> **Eng note:** Worklist supports filter chips (Property, Tenant status, Held amount range, Days held, Days since exit, Has anomaly flag, Has pending dues), removable, additive (AND). Pagination: standard limit/offset. All filter chips map to `LiabilityFilterCode` enum codes (range 1700-1799 — to be reserved). Sort defaults vary by entry path (see PRD Worklist Pre-sets table).

| Component | Status | Task | Definition + [class] | Plain Formula | Formula (technical) | Permission | Plain Drill | Drill Behavior (technical) |
|-----------|--------|------|----------------------|---------------|---------------------|------------|-------------|----------------------------|
| Worklist | BUILD | Per-tenant row | Tenant row with held breakdown on tap. [always-live] | Each row shows tenant name + status badge (Active / Booking / Moved Out / Tenant record removed), held amount with components on tap (SD ₹X · CM ₹Y · Advance ₹Z), days held, property (multi-property), exit date if scheduled with countdown, refunds-already-processed amount if any ("Refunded ₹15K of ₹50K"), anomaly flag if applicable. | `SELECT t.id, t.name, t.status, t.property_id, t.date_of_eviction, held_amount(t.id), days_held(t.id), refunded_amount(t.id), anomaly_flag(t.id) FROM tenant t WHERE held_amount(t.id) > 0` + filter chip predicates. **NEW QUERY** | view: `can_view_invoices` OR `view_invoices_of_self_added_tenants` (DB-side fallback via `checkAuthInDb`; PRD's `viewLiabilities` doesn't exist) | → Live (today): Tap row body to open Tenant Detail full-screen push. Tap held amount to expand SD / CM / Advance breakdown inline.<br>⚠️ `/tenant/getTenantData` unauth (CSB-3).<br>→ Past period: Same as live (always-live).<br>→ Future: Same as live (always-live). | → Live: FS (Tenant Detail) on body tap · inline accordion on amount tap · "Tenant Detail"<br>⚠️ unauth (CSB-3)<br>→ Past period: SAME<br>→ Future: SAME |
| Worklist | BUILD | "View Linked Deposit Invoice" action | Opens the original Security Deposit invoice. [always-live action] | Tap action button to open the original deposit invoice for this tenant. | Routes to `POST /invoices/getInvoiceData` (`src/routes/invoices.ts:215`) ⚠️ **zero HeaderValidator (CSB-3)**. Pass `invoice_id` from tenant's deposit invoice lookup. | view: `can_view_invoices` | → Live (today): Tap to open the linked Security Deposit invoice detail.<br>⚠️ `/invoices/getInvoiceData` is unauth (CSB-3).<br>→ Past period: Same as live (always-live).<br>→ Future: Same as live (always-live). | → Live: FS (Invoice Detail) · property: scoped to tenant's pg · filters: `invoice_id=X` · "Bill Detail"<br>⚠️ unauth (CSB-3)<br>→ Past period: SAME<br>→ Future: SAME |
| Worklist | EXISTS | "Process Refund" action | Cross-screen drill into refund creation. [always-live action] | Same as §4 Process Refund — render condition: tenant exited AND held > 0. Routes to `POST /refunds/advanced-addition`. | See §4 row 5 for formula. | view: `can_view_invoices` · action: `add_refund_access` | → Live (today): Tap → opens Refund Creation flow → on completion returns to DA-06 with row updated.<br>→ Past period: Same as live (always-live).<br>→ Future: Same as live (always-live). | → Live: FS (Refund Creation) · "Process Refund"<br>→ Past period: SAME<br>→ Future: SAME |
| Worklist | COMPOSE + [VERIFY] | "Open Settlement" action | Composite — same chain as §4. [always-live action] | Same as §4 Open Settlement — composes the 5-endpoint chain. **[VERIFY]** see §4 row 4. | See §4 row 4 for formula. ⚠️ `/tenant/adjustDeposit` unauth + financial mutation (CSB-3 P0). | view: `can_view_tenants` · action: `submit_or_update_moveout_checklist_access` AND `approve_moveout_checklist_access` | → Live (today): See §4. ⚠️ CSB-3 P0.<br>→ Past period: Same as live (always-live).<br>→ Future: Same as live (always-live). | → Live: FS chain · "Move-Out Settlement"<br>⚠️ CSB-3 P0<br>→ Past period: SAME<br>→ Future: SAME |
| Worklist | BUILD | Filter chips (removable) | Property · Tenant status · Held amount range · Days held · Days since exit · Has anomaly · Has pending dues. [always-live UI] | Each chip is removable with an explicit ✕ icon (44pt min hit area). Removal re-fetches without that chip; others stay. New filters are additive (AND). | Filter codes from `LiabilityFilterCode` 1700-1799. **NEW BUILD** — reserve range. | view: inherits | → Live (today): Tap ✕ to remove a chip.<br>→ Past period: Same as live (always-live).<br>→ Future: Same as live (always-live). | → Live: re-fetch without removed chip · sort: per default<br>→ Past period: SAME<br>→ Future: SAME |
| Worklist | BUILD | Empty filter state | "No tenants match your current filters." [always-live UI render] | When filters return 0 rows, show empty state with "Clear Filters" CTA. | Render when query returns 0 rows | view: inherits | → Live: Tap "Clear Filters" → re-fetches without active chips.<br>→ Past period: SAME (always-live).<br>→ Future: SAME (always-live). | → Live: re-fetch with no chips<br>→ Past period: SAME<br>→ Future: SAME |
| Worklist | BUILD | Bulk action — Export to Excel | New `/reports/liability-report` endpoint. [always-live action] | New backend endpoint generates an `.xlsx` and emails it. Toast confirms send. | **NEW BUILD** — `POST /reports/liability-report` (mirror `/reports/cashflow-report` shape, kebab-case). Fields: Tenant · Tenant Status · Property · Held Amount · SD · CM · Advance · Days Held · Exit Date · Refunded · Pending Dues · Anomaly Flag · Last Activity Date. | view: inherits · action: `money_reports` (DB-side) | → Live (today): Tap to confirm → toast → email arrives in a few minutes.<br>→ Past period: Same as live (always-live).<br>→ Future: Same as live (always-live). | → Live: M (confirm) → toast · backend job emails xlsx<br>→ Past period: SAME<br>→ Future: SAME |

---

## 7. Liability Trend Chart (6M / This Month)

> **Endpoint:** BUILD — chart endpoint reads from `liability_snapshots` table (Option A) or computes on-demand (Option B). HB4 picks one.
> **Default property context:** inherited from dashboard
> **Default time context:** **always-live snapshot view** — chart shows month-end snapshots regardless of global time filter; has its own 6M / This Month toggle
> **Hard Blockers:** HB4 (Option A vs Option B chosen — chart cannot ship without one)
> **Eng note:** Single-color bars (consistent with DA-04 / DA-05 trend charts). 6M default; This Month renders one bar (today's snapshot). Tap bar = inline tooltip showing held composition (SD / CM / Advance) for that snapshot. Tap "View as of [month-end] →" CTA inside tooltip = bottom sheet "Held composition as of [month-end]" with snapshot pill — informational only, no descendant filtering since DA-06 is fully live (R12 P0).

| Component | Status | Task | Definition + [class] | Plain Formula | Formula (technical) | Permission | Plain Drill | Drill Behavior (technical) |
|-----------|--------|------|----------------------|---------------|---------------------|------------|-------------|----------------------------|
| Trend Chart | BUILD + [VERIFY] | Bar series (6M default) | 6 month-end snapshots of total held. [snapshot view] | A bar for each of the last 6 month-ends, showing the total held at that point in time. **[VERIFY]** HB4 — pick Option A (backfill cron + new `liability_snapshots` table) vs Option B (on-demand re-derivation from invoice/refund timestamps). | **NEW BUILD** — depends on HB4. Option A: `SELECT month_end, total_held FROM liability_snapshots WHERE property_id IN (...) ORDER BY month_end ASC LIMIT 6`. Option B: re-derive `SUM(inv.amount - refunds before month_end) WHERE due_type IN ('Security Deposit', 'Caution Money') AND inv.status = 1 AND inv.paid_date <= month_end AND payment_mode != 211` per snapshot. | view: `can_view_invoices` | → Live (today): Tap a bar for inline tooltip showing held composition (SD / CM / Advance) for that month-end. Tap "View as of [month-end] →" inside the tooltip to open a bottom sheet with the breakdown.<br>→ Past period: Same as live — the global time filter does not affect this chart; the chart has its own 6M / This Month toggle.<br>→ Future: Same as live — same reason. | → Live: T (inline) on tap · BS (composition as-of snapshot, no descendant filter) on CTA tap · "Held composition as of [month-end]"<br>→ Past period: SAME (chart is independent of global filter)<br>→ Future: SAME |
| Trend Chart | BUILD | This Month toggle | Single-bar view of today's snapshot. [snapshot view UI] | Toggle to "This Month" → chart renders a single bar showing today's snapshot value. Selected toggle state preserved on back. | UI control. Re-fetch with `range=this_month`. | view: inherits | n/a — control. | n/a — UI control |
| Trend Chart | BUILD | Trend insight text | Conditional sentence below chart. [snapshot view] | Below the chart, render one of:<br>(a) "Liability has grown for 3 consecutive months — driven by new deposits." (each month > prior)<br>(b) "Liability has dropped — exits outpacing new bookings. Watch cash." (current month < prior, downward trend ≥ 2 consecutive months)<br>(c) "Liability is stable across 6 months." (variance < 10% across all bars) | Compute trend signal client-side from the 6 bar values. | view: inherits | Display only. | n/a — display |
| Trend Chart | BUILD | Empty / partial-history state | "Trend data builds up over time. Full 6-month history available [date]." [snapshot view UI render] | If Option A is picked and snapshots haven't accumulated yet (first 6 months from launch), render this message inline. | Render when `COUNT(snapshots) < 6` | view: inherits | Display only. | n/a — display |

---

## 8. Permission Vocabulary Reality Check

| Cited key (PRD) | Status | Recommended substitute (Field Map §3.4) |
|-----------------|--------|-----------------------------------------|
| `viewLiabilities` | DOES NOT EXIST as JWT key | Reuse `viewInvoices` — composite view, no new screen permission |
| `processRefunds` | DOES NOT EXIST as JWT key | DB column `add_refund_access` via `checkAuthInDb` (also `delete_refund` for delete) |
| `processMoveOuts` | DOES NOT EXIST as JWT key | DB has 4 columns: `submit_or_update_moveout_checklist_access`, `approve_moveout_checklist_access`, `edit_eviction_access`, `cancel_eviction_access`. Pick per action |
| `viewTenantDetails` | DOES NOT EXIST as JWT key | Use `viewTenants` JWT key |

> Note: PRD V1.4 Permission Vocabulary Reality Check explicitly calls out the same gap and Jatin gate. This Build Sheet uses the recommended substitutes throughout; rows that involve a fictional key cite the substitute in their Permission cell.

---

## 9. Reconciliation Invariants (consolidated)

QA-runnable correctness checks. No drill-down, no permission — these are pure equalities or bounds.

| Task | Definition | Formula / rule | Source rows |
|------|-----------|----------------|-------------|
| Hero is filter-independent | Total Held never changes when global time filter changes | Same SUM regardless of `filter_meta.range`. DA-06 is fully Live | §1 row 1 |
| Hero = Σ (paid deposit invoices − refunds), excluding mode 211 | `SUM(inv.amount − SUM(refunds.amount per inv)) WHERE due_type IN ('Security Deposit', 'Caution Money') AND inv.status = 1 AND inv.amount > 0 AND payment_mode != 211` | Strict equality (conditional on HB1 + HB3 + HB7 resolution) | §1 row 1 |
| By Property sum = hero held | Sum of all property-row held = hero | Strict equality | §3 |
| Hero composition: SD + CM + Advance = hero | Three composition segments add to hero | Strict equality | §1 row 7 |
| Worklist sum (top-N + "Others") = hero held | All worklist tenant amounts (sorted by held DESC) = total | Strict equality | §6 |
| Move-Out Forecast subset ⊆ active tenants with held > 0 | Forecast tenants must have `t.status = 1 AND held_amount > 0 AND date_of_eviction in [today, today+30]` | Inclusion check | §4 |
| Refunds Pending ⊆ moved-out tenants with held > 0 | Pending tenants must have `t.status NOT IN (1, 2) AND held_amount > 0` | Inclusion check | §5 |
| Forecast and Pending are disjoint | A tenant cannot be in both Move-Out Forecast and Refunds Pending simultaneously | Set disjointness | §4 ∩ §5 = ∅ |
| Cross-screen: DA-06 Refunds Pending tenant set is disjoint from DA-03 completed refunds for same period | A specific refund event is in DA-03 once processed; before processing the tenant is in §5 | Set disjointness | §5 vs DA-03 |
| Cross-screen: DA-01 Dues `due_type='Security Deposit' AND status=0` ∩ DA-06 active deposits = ∅ | Same invoice cannot be both an unpaid due (DA-01) and a held liability (DA-06 needs status=1) | Set disjointness | §1 vs DA-01 |
| Cross-screen flow invariant (matches DA-07 I5) | ΔLiability over period = (DA-02 deposit collections in period) − (DA-03 deposit refunds in period). Equivalently: end-period DA-06 hero = start-period DA-06 hero + collections − refunds. Sign convention matches DA-07 I5. | Strict equality | §1 vs DA-02 vs DA-03 |
| Settlement-readiness flag uses parent checklist row | Flag is ✅ iff `tenant_checklist.status = 6 (MOVE_OUT_APPROVED)` for the tenant's active checklist; never derived from item-level `move_out_marked_at` | Definition check (per Field Map §1.8) | §4 row 3 |
| Anomaly callout fires per v1.4 reconciled definition | Either (a) held > 12 months without ledger activity OR (b) held ≥ 2.5× property avg AND ≥ ₹25K AND property has ≥ 5 active deposit-holders | Definition check (per PRD Anomaly Callout Reconciliation) | §2 row 2 |
| Long-pending badge fires regardless of partial refunds | Badge appears iff `days_since_exit > 90 AND held_remaining > 0` — partial-refund history does NOT suppress the badge | Definition check (per PRD V1.2 clarification) | §5 row 3 |
| Cross-screen: DA-07 Section B Net Deposit movement matches DA-06 ΔLiability | DA-07 Section B: deposit money in/out for period (using `DEPOSIT_DUE_TYPES` plural). DA-06: ΔLiability = end-period hero − start-period hero. **⚠️ Scope divergence note (HB7):** DA-06 hero uses singular `DEPOSIT_DUE_TYPE` (SD only), DA-07 Section B uses plural (SD + CM). Equality holds only after HB7 widens DA-06's scope to plural; until then, the equality has a known offset = sum of CM held + CM movements | Strict equality post-HB7 resolution | §1 vs DA-07 §5 |

---

## 10. Edge Cases

| ID | Definition | Behavior | Trigger |
|----|-----------|----------|---------|
| EC-01 | Zero liabilities (new property or fully refunded) | Hero shows ₹0; all sections show empty states; Net Collection language hidden | `total_held == 0` |
| EC-02 | Single tenant accounts for ≥ 50% of total liability | Anomaly callout fires (if maturity gate passed); worklist (sorted by held DESC) puts that tenant at the top | `tenant_held / total_held >= 0.5 AND maturity_gate` |
| EC-03 | All deposits paid via mode 211 | Excluded from liability per PRD Rule 2 (paper transfer, no cash held). Currently homepage SQL counts it — see HB1/HB3 | `payments[0].payment_mode = 211` per deposit invoice |
| EC-04 | Tenant moved out but deposit invoice was deleted | Refunds against deleted invoices are orphans; tenant won't show in Refunds Pending. No detection possible without audit trail. Phase 2 Settings → Data Health | invoice deleted, refund orphaned |
| EC-05 | Active tenant with ₹0 deposit recorded but tenancy > 6 months | Suspicious; doesn't change liability total; v1 doesn't auto-detect | `t.status=1 AND held_amount=0 AND tenancy_days > 180` |
| EC-06 | `tenant.security_deposit` (contractual) ≠ aggregated deposit invoices (recorded) | Aggregate-invoice number is the truth for liability. Phase 2 Data Health surfaces mismatches | `t.security_deposit != Σ(deposit_invoices)` |
| EC-07 | Refund recorded but linked invoice not found | LEFT JOIN with NULL handling required; do not crash on `inv.amount IS NULL` | `refund.invoice_id` points to deleted invoice |
| EC-08 | Multi-property, one property has zero liabilities | By Property row shows that property at the bottom with ₹0 (standard display) | `selected_properties > 1 AND property_held == 0` |
| EC-09 | Tenant with multiple deposit invoices (top-ups over time) | All invoices summed under that tenant's held amount; components shown on tap ("SD ₹15K + SD top-up ₹5K · Total ₹20K") | tenant has > 1 deposit invoice |
| EC-10 | Tenant exits but checklist setting blocks refund | Tenant appears in Refunds Pending with status "Awaiting checklist lock"; `is_refund_deposit_blocked_until_all_checklists_locked = true` (per `src/controllers/property.ts:9326`) | property setting + tenant exited |
| EC-11 | Caution Money vs Security Deposit not distinguished | Always unioned in canonical query (`due_type IN ('Security Deposit', 'Caution Money')`); appear as single "deposit" line. **⚠️ HB7 — `getAvailableDeposit` only filters singular** | spec-level decision |
| EC-12 | Tenant flipped status 1 → 0 without a Refund row | Standard exit before refund processing; tenant moves from active list → Refunds Pending; total liability unchanged. Normal flow | tenant status transition without refund |
| EC-13 | Off-system refund (operator gave cash, didn't record) | Tenant still appears in Refunds Pending forever; Long-pending badge fires at days_since_exit > 90; operator must manually record or Phase 2 mark-written-off | `days_since_exit > 90 AND held_remaining > 0` |
| EC-14 | Advance balance — broken `getAdvanceAmount` returns 0 | Use `getNetAdvanceAmount` instead (`src/helpers/payments.ts:667+`); broken function at `:646-665` must NOT be used | known bug |
| EC-15 | Deposit refunded to wrong tenant (data correction) | Once Refund row is in, Tenant A's liability decreases. Correction requires deleting misposted refund + creating correct one. Phase 2 surfaces this UX | manual misposting |

---

## 11. Microcopy

Static strings. PRD's "Words on the Screen" section is canonical; this section copies-with-attribution for engineering convenience.

| Component | Task | Exact string |
|-----------|------|-------------|
| Section header | Header label | "Liabilities" — never changes |
| Snapshot pill | Static pill | "Snapshot — updated live" |
| Hero | ⓘ tooltip | "Money you're holding for tenants — security deposits, caution money, and advance balances. Updates live as deposits come in or refunds go out." |
| Move-Out Forecast | ⓘ tooltip | "Tenants whose contract end date or notice period falls in the next 30 days. Their deposits become refundable on exit. Plan cash for these." |
| Move-Out Forecast | Callout copy | "⚠️ Up to ₹X exiting in next 30 days · N tenants. Plan cash." |
| Move-Out Forecast | Callout precision tooltip | "Based on contract end date or notice given. Actual exits may differ — confirm with tenants before settling cash." |
| Anomaly | Callout copy | "₹X deposit held for [Tenant] — unusual for this property (avg ₹Y). Confirm this matches the agreement." |
| Refunds Pending | ⓘ tooltip | "Tenants who've already moved out and are owed deposit money. Process oldest first — refunds older than 21 days are reputation risk." |
| Refunds-overdue | Callout copy | "⚠️ ₹X owed to ex-tenants for 21+ days. Reputation risk — process refunds." |
| Long-pending badge | Row badge | "Possibly settled off-system?" |
| Advance chip | Chip copy | "+ ₹X advance balances held." |
| Notice-window chip | Chip copy | "N tenants in notice period · ₹X deposit held." |
| By Property | ⓘ tooltip | "Each property's deposit liability and how many tenants are exiting in the next 30 days. The 'Exiting' column tells you which property has the biggest cash event coming." |
| Trend Chart | ⓘ tooltip | "How your total liability has moved over time. Rising = growing tenant base or larger deposits per tenant. Falling = exits outpacing new bookings — leading indicator of cash crunch." |
| Notice-window chip | ⓘ tooltip | "Tenants who have given notice of their intent to leave. Their deposits become refundable on exit — usually within their notice period." |
| Empty | No liabilities (new property) | "No deposits held right now. They'll appear here once tenants pay deposits." |
| Empty | No move-outs in next 30 days | "No tenants exiting in the next 30 days. Liability is stable for now." |
| Empty | No refunds pending | "All refunds are processed. Nothing owed to ex-tenants." |
| Empty | Worklist filter empty | "No tenants match your current filters." (CTA: "Clear Filters") |
| Empty | Trend chart no historical data | "Trend data builds up over time. Full 6-month history available [date]." |
| Error | Network failure | "Couldn't load liabilities. Check your connection." (CTA: "Retry") |
| Error | Section fails | "Couldn't load this section." (CTA: "Retry" on that section) |
| Permission denial | Process Refund | "You don't have permission to process refunds. Contact your admin." |

---

## 12. Smoke Test Commands

Per-endpoint curl. Substitute `<host>`, `<token>`, `<pg_id>` per environment.

```bash
# Liability list/widget (NEW BUILD — Phase 1)
curl -X POST 'https://<host>/v1/list_screens/liabilities/list/filters' \
  -H 'Authorization: Bearer <token>' \
  -H 'Content-Type: application/json' \
  -d '{ "pg_number": "<pg_id>", "filter_codes": [], "limit": 20, "offset": 0 }'

# Liability widget (NEW BUILD)
curl -X POST 'https://<host>/v1/list_screens/liabilities/widget' \
  -H 'Authorization: Bearer <token>' -H 'Content-Type: application/json' \
  -d '{ "pg_number": "<pg_id>" }'

# Move-Out Forecast filter (filter_code 1702 — NEW)
curl -X POST 'https://<host>/v1/list_screens/liabilities/list/filters' \
  -H 'Authorization: Bearer <token>' -H 'Content-Type: application/json' \
  -d '{ "pg_number": "<pg_id>", "filter_codes": [1702], "limit": 20, "offset": 0 }'

# Refunds Pending filter (filter_code 1703 — NEW)
curl -X POST 'https://<host>/v1/list_screens/liabilities/list/filters' \
  -H 'Authorization: Bearer <token>' -H 'Content-Type: application/json' \
  -d '{ "pg_number": "<pg_id>", "filter_codes": [1703], "limit": 20, "offset": 0 }'

# Move-Out checklist items (existing — HeaderValidator)
curl -X GET 'https://<host>/checklist/move-out/items?tenant_uuid=<tenant_uuid>' \
  -H 'Authorization: Bearer <token>'

# Mark move-out item (existing — HeaderValidator)
curl -X PUT 'https://<host>/checklist/move-out/mark-item' \
  -H 'Authorization: Bearer <token>' -H 'Content-Type: application/json' \
  -d '{ "item_id": <item_id>, "condition": "working", "notes": "" }'

# Complete move-out checklist (existing — HeaderValidator)
curl -X POST 'https://<host>/checklist/move-out/complete-checklist' \
  -H 'Authorization: Bearer <token>' -H 'Content-Type: application/json' \
  -d '{ "checklist_id": <checklist_id> }'

# Refund creation (existing — HeaderValidator)
curl -X POST 'https://<host>/refunds/advanced-addition' \
  -H 'Authorization: Bearer <token>' -H 'Content-Type: application/json' \
  -d '{ "tenant_uuid": "<tenant_uuid>", "invoice_id": "<invoice_id>", "amount": 5000, "refund_mode": 2040 }'

# ⚠️ Adjust from Deposit (FINANCIAL MUTATION — currently UNAUTH — CSB-3 P0)
# DO NOT TEST IN PROD without auth fix
curl -X POST 'https://<host>/tenant/adjustDeposit' \
  -H 'Content-Type: application/json' \
  -d '{ "tenant_uuid": "<tenant_uuid>", "amount": 0 }'

# ⚠️ Tenant Detail (currently UNAUTH — CSB-3)
curl -X POST 'https://<host>/tenant/getTenantData' \
  -H 'Content-Type: application/json' \
  -d '{ "tenant_uuid": "<tenant_uuid>" }'

# ⚠️ Invoice Detail (currently UNAUTH — CSB-3)
curl -X POST 'https://<host>/invoices/getInvoiceData' \
  -H 'Content-Type: application/json' \
  -d '{ "invoice_id": "<invoice_id>" }'

# Liability Excel report (NEW BUILD — Phase 1)
curl -X POST 'https://<host>/reports/liability-report' \
  -H 'Authorization: Bearer <token>' -H 'Content-Type: application/json' \
  -d '{ "pg_id": "<pg_id>" }'
```

---

## How to Use This Doc

- **Pick a row → that's a ticket** (or part of one). Status column tells you whether to extend, build, compose, or wire UI.
- **Plain Formula + Plain Drill columns** are for PM / CS / QA review without engineering vocabulary.
- **Formula (technical) + Drill Behavior (technical) columns** are for implementation. All code references cite file:line; first-occurrence-per-section gets a plain-English gloss in italics.
- **`[VERIFY]` tag** = needs spike-time confirmation. The cell will say what to confirm.
- **`⚠️` warnings** in cells flag CSB / HB issues. Resolve before launch.
- **Reconciliation Invariants (§9)** are correctness checks for QA — copy into test plans.
- DA-06 is fully Live — most rows say "Same as live (always-live)" in past/future drill contexts. Only the Trend Chart (§7) has its own time control, and even that is independent of the global filter.
- For "why" or rationale, see [[DA-06 Liabilities Detailed Analytics]].
- For entity fields, formulas, permissions, endpoints, see [[_Ground Truth Field Map]].
- For format conventions, see [[_Build Sheet Generation Spec]].
- If a row is ambiguous, the PRD is the source of truth. Don't guess — ask.

---

## Maintenance Log

| Date | Change | By |
|------|--------|-----|
| 2026-05-08 | V1 — initial Build Sheet per [[_Build Sheet Generation Spec]] V1. Locked 9-column structure. Verified all code references against [[_Ground Truth Field Map]] and codebase (master post-merge). New findings: HB7 (`getAvailableDeposit` filters singular `DEPOSIT_DUE_TYPE`, drops Caution Money) and HB8 (no production batched per-tenant held query — worklist needs new aggregator). Both flagged for back-port to PRD Pre-Launch Engineering Blockers. | PM |
| 2026-05-11 | DA-01 spec audit propagation: (a) ⓘ icon convention updated to "single-tap → BS only" per Generation Spec §15; (b) CSB-8 (widget self-added permission gap, security) added — DA-06 NEW BUILD widget endpoints must enforce self-added filter from day 1. | PM |
