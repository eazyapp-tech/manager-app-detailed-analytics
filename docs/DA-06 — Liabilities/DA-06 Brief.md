---
title: DA-06 Liabilities — Brief
date: 2026-05-18
tags:
  - rentok
  - brief
  - liabilities
status: Living document · v0.2.2 (kickoff polish — 4 nice-to-haves from independent verification)
owner: Sanchay (PM) · co-signed by Eng lead + Design lead before kickoff
time_budget: 2-week build cycle
companion_to: DA-06 Liabilities Detailed Analytics (legacy PRD V1.4 — historical), DA-06 Build Sheet V1 (engineering), DA-06 Codebase Exploration (Phase 2 LAYER 1+2 findings)
---

> [!INFO] What this is
> A **Brief** for the Liabilities screen — written before design. What the PM wants the screen to *do*, not how it should look.
>
> **What this is NOT:** an engineering spec or a design doc. Those live in `[[DA-06 Build Sheet]]` and Figma.

---

# DA-06 Liabilities — Brief

## In one line

Rajesh holds tenants' money — security deposit, caution money, advance rent — every day. He owes most of it back at move-out. Today he can't tell you the total he's holding, who's exiting next month, or who's already left and waiting for a refund. He goes by gut, opens each tenant's passbook one by one, or pulls Excel at month-end. This screen shows the live liability picture: total held + by-composition breakdown + move-out forecast + refunds-pending. Drills land on existing tenant lists with new filter chips. **2-week build to ship V1.**

---

## The operator

The owner / admin / supervisor when they want to **see how much money they currently hold for tenants** — and which of that money is about to need refunding. The homescreen says *something's off*. The tenant passbook tracks each individual deposit. This screen sits between them: it aggregates the held money and flags the refund pipeline.

**Rajesh.** Single-property PG owner. 35–50, Hindi-first, 20–40 beds in Bengaluru / Pune / Jaipur. Holds deposits + caution money + occasional advance rent for every tenant. Opens this screen with a mixed rhythm:
- **Weekly review** — Sunday or Monday, scan move-out forecast and refunds-pending.
- **Event-triggered** — when a tenant gives notice, when planning a cash deployment, when a refund is queried.
- **Monthly deep audit** — month-end reconciliation, especially against the homepage Deposits Held card.

He leaves each visit with a picture of total liability + next-30-day exit risk + (if anything looks off) a follow-up: *"Iss tenant ka refund kab dena hai?"* or *"Itna deposit kaise jama hai ek banda pe?"*

**Priya** — multi-property owner (5+). Same mixed rhythm across her portfolio. Wants to know which property has the biggest liability concentration and where the next-30-day cash outflow is largest.

**Meena / Ramesh bhaiya** — the team members who handle tenant interactions and process refunds. What they see on this screen depends on their granted permissions — V1 reuses `viewInvoices` (composite view, no new key — see Traps).[^1]

[^1]: Personas are composites from ~6 field interviews (Mar–Apr 2026). Same trio as DA-01/02/03/04/05. RentOk permissions are atomic per phone number (per team member) — there's no fixed admin/staff role, just access keys the owner grants.

**This screen is fully a LIVE snapshot — unique among the DA suite.** Liabilities are stocks (current state), not flows (period activity). The time chip is rendered for cross-tab consistency but ignored. MoM comparison is meaningless and not shown — trajectory comes from the trend chart (deferred to V2; see Traps + What we're NOT building).

**Rhythm fit:** weekly review + event-triggered + monthly audit, ~6-8×/month per operator. Standard feature.

Tapping any row, segment, or section routes the operator into the relevant context. The rule (not the UI prescription — designer picks entry points):
- **Hero composition tap** (SD / Caution / Advance segment) → existing Current Tenants list filtered to *"Has Deposit"* / *"Has Advance"* chip for the right composition.
- **Move-Out Forecast row tap** → existing Current Tenants list filtered to *"Has Deposit"* + sorted by exit date ASC.
- **Refunds Pending row tap** → existing Old Tenants list filtered to *"Refundable"* (computed: `tenant.status NOT IN (1,2) AND held_amount > 0`).
- **By-property row tap** (Priya only) → drill into the same property's DA-06 view.
- **Per-tenant row tap** (within any section) → existing Tenant Passbook with deposit tab focused.

All drills land on EXISTING tenant list screens with NEW filter chips inherited from DA-06 V1 — see Traps for the cross-Brief filter-chip scope.

---

## The problem

Three pains, same root cause — there's no aggregate live view of liability today:

1. **No idea how much I'm holding right now.** Deposits + caution + advances accumulate one tenant at a time, then disappear into individual tenant ledgers. The operator can't tell at-a-glance *"we're holding ₹8.5L across 42 tenants — ₹6L deposit + ₹1.5L caution + ₹1L advance."* The total exists only in the operator's head or a month-end Excel pull.
2. **No idea who's exiting next month — can I refund them?** A move-out triggers refund obligation. Without a forward-looking view, operators learn about exits one-by-one as notice periods land in WhatsApp, then scramble to check if cash is available. No 30-day forecast.
3. **No idea who's already left and I haven't refunded.** When a tenant exits but the refund isn't processed (often because the operator forgot, or because settlement happened off-system), the deposit sits on the books indefinitely. There's no `refund_status` field — refunds-pending must be deduced (exited tenant AND held > 0). Operators discover the gap at month-end audit, or never.

Liabilities you can't see, you can't govern. The bet isn't to hold less — it's to make the held-money picture clear so refunds get processed on time and cash is planned for upcoming exits.

**Why now:** Cash flow control is one of RentOk's core promises, and held liability is the most opaque cost on the operator's balance sheet. Right now Rajesh holds significant tenant money but can't see it as one number — he learns about owed-refunds reactively when an ex-tenant calls. Every cycle without this screen is another month of reputation risk (delayed refunds) + cash-flow blind spots (unforecasted exits).

---

## What Rajesh does today

- **Goes by gut / mental tally.** Most common. Operator estimates total liability but rarely tracks it actively.
- **Pulls Excel report at month-end.** Manually sums deposits paid minus refunds processed across all tenants. Hours of work for 30+ tenants. Power-user move; most don't.
- **Checks each tenant's passbook one by one.** Opens Tenant Detail → sees deposit invoice + refund history. Slow, error-prone, only works for small properties.
- **Glances at homepage Deposits Held card.** The number exists on the homepage but is just an aggregate — no breakdown, no aging, no exits-in-30-days context. Operator gets the total but nothing actionable.

All four workarounds coexist — operators with different literacy levels use different combinations. None give the live + breakdown + forecast + refunds-pending picture in one place.

---

## What we must ship — and what we cut if time runs short

**Time budget: 2 weeks.** Why 2 and not 6: Rajesh holds tenant money every day with no aggregate live view — every cycle of delay is another month of reputation risk on delayed refunds and cash-flow blind spots on unforecasted exits. Two backend devs (1 senior + 1 junior) on this, plus separate frontend and separate web teams — capacity is real. **All endpoints are NEW BUILD** (list, widget, filter_options, detail, Excel) — heavier than DA-04 (extended existing) and similar to DA-05 (also fully NEW BUILD). **Added scope in V1: 5 new filter chips across 3 existing tenant-list endpoints (Bookings: Has Advance + Has Deposit; Current Tenants: Has Advance + Has Deposit; Old Tenants: Refundable) + auth fixes on those 3 list endpoints + held-amount column added to those 3 list screens.** ~5-7 extra dev-days. The cut order below is the contingency, not the expected outcome.

**Must ship (V1 is not V1 without these):**
- **Q1 — how much am I holding right now (Hero).** Live total of money the operator currently holds for tenants. Extended formula: `paid Security Deposit + paid Caution Money + paid Advance − refunds processed − Mode 211 paper-transfer adjustments`. Across multi-property if multi-property dashboard.
- **Q3 — who's exiting in the next 30 days (Move-Out Forecast).** Tenants with `date_of_eviction` in next 30 days AND held > 0. Sorted by exit date ASC. Answers the cash-deployment-planning question directly.
- **Q4 — who's already left and I haven't refunded (Refunds Pending).** Computed: `tenant.status NOT IN (1,2) AND held > 0`. Sorted by days-since-exit DESC. Answers the reputation-risk question directly.

These three answer 90% of *"what am I holding, what's about to leave, and what's overdue?"* **No-ship reconciliation gate (rewritten for precision):** the **SD + Caution sub-component** of DA-06 hero MUST match the homepage Deposits Held card to the rupee for the same property + same instant. The full DA-06 hero total may LEGITIMATELY differ from the homepage card by exactly the Advance amount — this is the deliberate extended-formula divergence, NOT a bug. The screen subtitle must surface the Advance contribution explicitly so operators can mentally back out the SD+Caution-only number when cross-checking against the homepage. **Pre-V1 QA gate (no-ship blocker): 10-property rupee-match sample on the SD+Caution sub-component before V1 ships.**

**Protect at stretch tier (last to cut):**
- **Q2 — by composition.** Tap hero → bottom sheet showing SD / Caution / Advance subtotals. Each segment taps to the filtered list. Answers *"where is my held money concentrated?"*
- **Q5 — by property (Priya only).** One row per property with per-property held total + next-30-day-exits subtitle. Skip for solo Rajesh.
- **Q6 — refunds-overdue callout.** Conditional callout when any ex-tenant has held > 0 AND days-since-exit > 21. Tap → Q4 list pre-sorted by oldest.
- **Q7 — move-out-forecast callout.** Conditional callout when any tenant exits in next 30 days AND has held > 0. Tap → Q3 list.

**If time tightens, cut in this order (operator-pain-ranked, least pain first):**
1. **Q5 — by property.** Niche for multi-property; solo Rajesh gets nothing. Hero still shows multi-property total.
2. **Q7 — move-out forecast callout.** The Q3 section list carries the same data; callout is redundant for operators who scroll past the hero.
3. **Q6 — refunds-overdue callout.** Same logic — Q4 list carries the data; callout is the alert overlay.
4. **Q2 — composition breakdown.** Operator loses the SD/Caution/Advance split bottom-sheet UI. Hero number still shows total; chip filter codes still work (they don't depend on Q2's UI). Cut LAST among stretch items because the hero drill affordance breaks without it — operator taps the hero expecting a breakdown and gets nothing. Q2 is cuttable but losing it degrades the hero's drill-into-action utility.

**Not cuttable: the 5 new filter chips on Bookings / Current Tenants / Old Tenants lists.** Operator-side cost of cutting them = no way to drill from DA-06 sections into the actionable tenant list; the screen becomes a read-only number with no path to action. Same reasoning as DA-05's "chips are MUST-SHIP" decision. Chips are part of V1 must-ship alongside Q1+Q3+Q4.

**Conditional cut: nothing is conditional in V1.** Unlike DA-05 Q6 (which depended on HB1 + NEW-1 fixes), DA-06's must-ship questions don't depend on bug fixes landing — they depend on the canonical aggregator being unified (HB1) and Mode 211 being excluded (HB3). Both fixes are inside V1 scope, not pre-conditions.

---

## What each question needs

Each line describes the *information* — not how it should look. **How** the screen shows it is the designer's call. Tapping routes per the drill chain in the Operator section.

1. **Hero — total held right now.** One live total: `SUM(paid SD + paid Caution + paid Advance − refunds processed − Mode 211 adjustments)` across the selected property/properties. Subtitle: tenant count + next-30-day exits count + (multi-property) property count. Live snapshot — no MoM comparison shown (liabilities are stocks, MoM is misleading).
2. **By composition.** Hero tap opens SD / Caution / Advance subtotals. Each segment tappable → drill to filtered tenant list (Current Tenants with appropriate chip).
3. **Move-Out Forecast.** Per-tenant rows for tenants with `date_of_eviction` in next 30 days AND held > 0. Columns: tenant name + room + held amount + days-to-exit. Sorted by exit date ASC.
4. **Refunds Pending.** Per-tenant rows for tenants with `status NOT IN (1,2)` AND held > 0. Columns: tenant name + room + held amount + days-since-exit. Sorted by days-since-exit DESC. **Computed filter** — no `refund_status` field exists; backend derives this from tenant status + canonical held-amount query.
5. **By property** (Priya only). One row per property with held total + next-30-day-exit count. Skip for single-property Rajesh.
6. **Refunds-overdue callout.** Conditional render: fires when any ex-tenant has held > 0 AND days-since-exit > 21. Amount + count + "Review" CTA → Q4 list.
7. **Move-out-forecast callout.** Conditional render: fires when any tenant exits in next 30 days AND held > 0. Amount ("Up to ₹X exiting") + count + "View" CTA → Q3 list.

---

## What we're NOT building this cycle

- **No trend chart.** Historical liability data doesn't exist — there's no `liability_snapshots` table anywhere in the codebase (verified). V1 cannot show trajectory without either building a backfill cron + new table (~3-4 dev-days, breaks the budget) or doing on-demand re-derivation (expensive, fragile). V2 ships the table + cron + chart together. V1 shows current snapshot only; honest framing in the screen header.
- **No "Open Settlement" composite (5-endpoint move-out chain).** The chain includes `/tenant/adjustDeposit` which has ZERO authentication AND is a financial mutation (P0 security bug — anyone with the URL can change deposit amounts on any tenant). **Owner: Jatin (Eng lead). Action item: file P0 ticket separately from DA-06 with target landing date.** DA-06 doesn't block on it. V1 is read-only. Operator finalizes settlements via the existing move-out flow externally. V1.1 ships the composite once `/tenant/adjustDeposit` is auth-fixed.
- **No "Process Refund" mutation surface from DA-06 either.** Even though `/refunds/advanced-addition` IS auth-safe, V1 stays read-only end-to-end for scope discipline — drill-to-action lives on the destination list screens, not on DA-06 itself. V1.1 may add the in-screen action.
- **No notice-window chip.** Build Sheet HB5 referenced `tenant.notice_period_started_date` field — codebase verification shows this field doesn't exist (closest is `notice_raised_on`). Rather than repoint to a different field whose semantics may differ from PRD intent, cut the chip entirely. V1.1 revisits if operators ask.
- **No anomaly callout** (12-month-no-activity OR 2.5×-property-avg outlier). Build Sheet has the logic but it requires 2 new aggregation queries + a maturity gate. Defer to V1.1; the Refunds-Pending list already surfaces stale deposits through the days-since-exit sort.
- **No per-tenant settlement readiness flag** (✅/⏳ badge based on `tenant_checklist.status = 6`). Adds row complexity for V1. Operators can check checklist status via tenant drill if needed. V1.1.
- **No multi-invoice refund modeling.** Schema constraint — `Refunds.invoice_id` is a single FK. Operator can't refund across multiple deposit invoices in one operation today. V2 schema change.
- **No notifications, no push, no badges, no popups.** The operator opens this screen when they want to.

If a feature being discussed mid-cycle would require a notification, popup, or mutation surface inside DA-06, cut it.

---

## Traps & risks

Decided in advance so engineering doesn't wrestle:

- **USER:** **Hero formula divergence from homepage canonical — partial reconciliation gate.** DA-06 hero = extended formula `paid SD + paid Caution + paid Advance − refunds processed − Mode 211 adjustments`. The existing homepage Deposits Held card uses canonical `paid SD + paid Caution − refunds − Mode 211` (NO Advance). **The two totals MUST legitimately differ** by exactly the Advance amount — this is the deliberate extended-formula decision, NOT a bug. **Reconciliation rule:** the SD + Caution **sub-component** of DA-06 hero MUST match the homepage card to the rupee for the same property + same instant. The full DA-06 total = SD+Caution sub-component + Advance contribution. **Honesty requirement:** the screen must surface the Advance contribution as a subtitle line (e.g., *"includes ₹X in advance balances"*) so operators can mentally back out the SD+Caution-only number when cross-checking against the homepage. **Pre-V1 QA gate (no-ship blocker): 10-property rupee-match sample on the SD+Caution sub-component (not the full total) before V1 ships.** **Cross-screen strategic punt (acknowledged):** the homepage Deposits Held card and DA-06 hero will disagree by the Advance amount forever unless the homepage card is also extended. Brief recommends extending the homepage card in a follow-on cycle for full reconciliation, but does NOT block V1 on that work. Until then, the screens disagree by design; subtitle disclosure carries the operator-facing explanation.

- **USER:** **`tenant.security_deposit` snapshot column is anonymously writable today — DA-06 must NEVER read it as a source of truth.** The `/tenant/adjustDeposit` endpoint (zero auth, financial mutation — P0) writes to `tenant.security_deposit` directly. If DA-06 reads that column as a fallback, anyone with the URL can change the displayed liability number. **All hero/breakdown/per-tenant calculations MUST derive from invoice + refund + payment tables only**, never from `tenant.security_deposit`. Verified at codebase-exploration: only `paymentService.ts` aggregators (getAvailableDeposit + getAvailableAdvance + getApplicableCredits) and the homepage SQL touch the canonical tables.

- **USER:** **No MoM comparison on this screen (different from every other DA screen).** Liabilities are stocks (current state), not flows (period activity). Showing ▲/▼ MoM on liability would compare two snapshots taken at different times — misleading and operator-confusing. **Header treatment:** "Live · updated now" pill always visible; no period chip drives the hero (the chip renders for cross-tab consistency but is ignored by DA-06). Designer must NOT add a "(Live)" suffix in the title — the pill carries it; the title says plain "Liabilities." **"Updated now" definition:** the number computes fresh on each screen open (no cache) — pull-to-refresh re-runs the query. If a transaction lands between two visits, the number reflects it on the next open. Defining "now" matters because operators may compare the screen to a number they saw 5 minutes ago and expect a stable reading; the pill must imply "this is the current live number" not "this is a 30-second snapshot." Designer + eng align on cache strategy before the pill ships.

- **USER:** **Priya (multi-property owner) may want MoM at the property-rollup level even though individual liability is a stock.** Cross-property comparison ("Property A's liability is up 30% MoM, Property B is flat") is a flow question riding on stock data. V1 does NOT ship this — keeps the "no MoM" framing universal. **V1.1 may add property-rollup trend if Priya operators ask** — needs the same `liability_snapshots` table that V2 trend chart needs. Flag in V2 scope.

- **USER:** **Refunds-Pending is a computed filter, not a stored field — operator-facing freshness remedy required.** No `refund_status` column exists on tenants or invoices. *"Refunds Pending"* = `tenant.status NOT IN (1,2) AND held_amount(t.id) > 0`. **Operator-side consequence:** the count can shift between visits not because a refund happened, but because the held-amount calculation re-runs each time and may pick up edge-case changes (a new mode-211 entry, an invoice status transition). **Mitigation (information needs — designer picks the UI):** operators need (a) a **freshness signal** — they should understand the count was computed at this moment, not stored — so they don't compare a 9am number to a 2pm number and think the DB lost data; (b) **operator education** on why the count can shift between visits even when no refund happened (mode-211 entries, invoice status transitions, held-amount recompute) — surfaced in plain language somewhere reachable from the section; (c) **source-of-truth treatment** in V1 since no alternative exists. V2 adds the stored `refund_status` field to make this a hard fact.

- **USER:** **HB1 unification — pick the canonical liability aggregator on Day 1, before any breakdown work.** Three divergent code paths produce different "money held" numbers today: (a) `src/v1/homepage/service.ts` CTE at lines ~2100-2180 (homepage Deposits Held card, currently missing Mode 211 filter); (b) `src/helpers/payments.ts:594-618` `getSecurityDepositAmount` (per-tenant, handles Mode 211 correctly); (c) `src/services/payment/paymentService.ts:2870-2899` `getAvailableDeposit` (only Security Deposit, **drops Caution Money** — HB7). **Decision before any breakdown work starts:** wrap the canonical aggregator in `LiabilityService.getDepositsHeld()` (add Mode 211 filter + use `DEPOSIT_DUE_TYPES` plural). Route DA-06 hero, breakdown, sections, and the homepage card through it. Day-3 lock.

- **USER:** **Mode 211 paper-transfer phantom liability — operator-facing remedy required on launch day.** `payment_mode = 211` is DEPOSIT_PAYMENT_MODE — a paper transfer that moves deposit balance internally (e.g., applied to clear a rent due) without real cash movement. The homepage canonical SQL currently does NOT exclude these, inflating liability. **HB3 fix:** add `payment_mode != 211` filter to the unified aggregator. **Operator-side consequence:** after the fix, the hero number may drop noticeably on properties that have done many internal deposit-to-rent transfers — and the homepage card will drop simultaneously (same canonical aggregator). **Mitigation (information needs — designer picks the surface):** (a) **one-time operator-facing explanation** on first visit per operator surfacing the math change in plain language — *"paper-transfer entries that didn't move cash are now excluded; your number may be lower than yesterday — this is the right number"* — the surface (banner / bottom sheet / inline note) is designer's call but the message must land before the operator's first hero-glance. (b) **Pre-launch operator-testing** with 3 operators on properties with high mode-211 history: confirm the explanation lands as honest correction, not as data loss. (c) **Support team briefing day-before-launch** with the new formula + per-property diff samples so they can answer "why did my liability drop ₹50K overnight" tickets confidently.

- **USER:** **All 3 drill-target list endpoints have ZERO authentication today — no-ship blockers (CSB-3 sweep), NOT cuttable from V1 scope.** `/fetchTenants` (line 928), `/fetchEvictedTenants` (line 929), `/fetchBookings` (line 956) all currently have no HeaderValidator. **DA-06 V1 ships only after all 3 get HeaderValidator + checkAuth + pg_id validation.** **Operator-side cost of cutting = PII regression strictly worse than today.** Today operators can leak tenant data via these endpoints if they share URLs; with DA-06, the screen actively maps "here are the tenants with the most money held → click to see PII list," turning a passive leak into an advertised one. **Cannot be moved to fast-follow ticket — must land in V1 scope window.** Eng owns the fixes; the fixes belong to DA-06's timeline.

- **USER:** **Held-amount column on Bookings + Current Tenants + Old Tenants lists — required for operator mental model.** When operator drills from DA-06's Move-Out Forecast section into Current Tenants list filtered to "Has Deposit," they're asking *"of these N tenants exiting, who has the most money on the books I need to refund?"* — a sorted, visible held-amount column is the answer. Without it, operator lands on a filtered list but can't see WHO has the most held without opening each tenant's passbook one by one — the same workaround the screen was built to eliminate. **MUST-SHIP. Cutting the column = cutting the drill's utility.** Backend cost: trivial (already computed for the chip filter). Frontend cost: column add + sort affordance per list screen.

- **USER:** **Permission gate: reuse `viewInvoices` (composite-view rationale) + tab-hide UX for operators without it.** PRD cites fictional `viewLiabilities` key — doesn't exist. Reusing `viewInvoices` is correct because DA-06 is a composite view over invoice + refund + payment data; any operator with invoice visibility legitimately needs the liability aggregate. **No new permission key built in V1.** V1.1 may revisit if operators ask for a separate "liability viewer" key. **UX of the gate:** operators without `viewInvoices` (e.g., checklist-only or expenses-only team members) see the DA-06 tab HIDDEN from navigation entirely — not "empty state," not "no permission" error. Hiding matches existing DA-suite permission-gating patterns and avoids leaking which tabs exist to operators who can't see them.

- **TEAM:** **All DA-06's own endpoints NEW BUILD in v1/list_screens architecture.** `POST /v1/list_screens/liabilities/list/filters`, `GET /v1/list_screens/liabilities/list/widget`, `GET /v1/list_screens/liabilities/list/filter_options`, `GET /liabilities/advanced-details`, `POST /reports/liability-report`. Mirror `src/v1/list_screens/expenses/` pattern. DA-06's own filter codes from the `1700-1799` range (free per Field Map §2.8 — codebase-wide grep confirmed only `q_1201`–`q_1219` exist today, so the entire 13xx-19xx range is open). **Architecture asymmetry to flag:** DA-06's own list endpoint uses the v1/list_screens pattern + `q_NNNN` codes, but the 3 drill-target tenant lists (`/fetchTenants`, `/fetchBookings`, `/fetchEvictedTenants`) are legacy routes that DON'T use the v1/list_screens pattern. Chip implementation on those lists uses ad-hoc query params, NOT `q_NNNN` (see chip Trap). User-side consequence: until base widget endpoint lands, the screen has no data.

- **TEAM:** **HB1/HB7 unification touches 5 files.** Today deposit logic is scattered across `helpers/payments.ts`, `paymentService.ts`, `evictionService.ts`, `homepage/service.ts`, and `controllers/invoices.ts`. **HB1 fix:** create `LiabilityService.getDepositsHeld(propertyIds)` in a new `src/services/liabilities/` directory. Route all 5 existing call sites through it over time (V1 just adds the new service + routes DA-06 + homepage card through it; legacy callers in eviction + invoices can migrate in V1.1). **HB7 fix inside the same service:** use `DEPOSIT_DUE_TYPES` (plural) constant — already exists at `src/services/payment/constants.ts:5-6`, just unused in `getAvailableDeposit`. One-line fix.

- **TEAM:** **`getAdvanceAmount` at `helpers/payments.ts:647-666` is broken — returns 0 unconditionally** (async/sync bug — `.then` returns inside callback while outer fn returns 0 synchronously). DA-06 must use `getAvailableAdvance` at `paymentService.ts:2820-2867` (which works correctly, subtracts refunds + mode-288). Adjacent bug ticket to file: deprecate `getAdvanceAmount` after V1.1 callers migrate.

- **TEAM:** **5 new filter chips across 3 list endpoints — MUST-SHIP, not cuttable.** Bookings (Has Advance + Has Deposit), Current Tenants (Has Advance + Has Deposit), Old Tenants (Refundable). Operator-side cost of cutting = DA-06 sections become read-only numbers with no actionable drill path; the screen becomes inert. Backend uses direct FK paths on Payments + Invoices + Refunds tables — no exotic JOINs needed (verified at codebase-exploration). **Filter mechanism gap (codebase-verified):** the `q_NNNN` filter-code system only exists in `src/v1/list_screens/` modules (verified: only `q_1201`–`q_1219` exist anywhere, all in Collections). The 3 target list endpoints (`/fetchTenants`, `/fetchBookings`, `/fetchEvictedTenants`) are LEGACY routes in `src/controllers/tenant.ts` — they don't use `q_NNNN` at all. **V1 chip implementation:** add ad-hoc query params (`has_deposit=1`, `has_advance=1`, `refundable=1`) to the legacy endpoints rather than introducing the filter-code system mid-flight. This keeps scope contained but creates a small architecture inconsistency the team should flag for the "migrate tenant lists to v1/list_screens/" follow-on cycle. User-side requirement: when an operator lands on a list from DA-06 drill, the active chip state must be visibly clear and easy to clear — operators arriving from a different entry point shouldn't find the list silently pre-filtered. Two separate chips were chosen over a tri-state per list (matches existing chip patterns).

- **TEAM:** **"Old Tenants Refundable" chip = computed filter, backend complexity.** No `refund_status` field exists. Filter logic: `tenant.status NOT IN (1,2) AND held_amount(t.id) > 0` — requires the same canonical aggregator from HB1. Means the Old Tenants list endpoint must call `LiabilityService.getDepositsHeld()` for the chip filter to work. Ties the 3 chip-list endpoints to the HB1 unification timeline. **NOT a chip cut — a conditional Day-5 degradation:** if HB1 unification slips past Day 5, the Refundable chip's standalone-drill-to-Old-Tenants utility degrades (DA-06's Refunds-Pending section still works internally because it can call the per-tenant composite directly). The chip itself ships in V1; what changes is whether operators can use it as a standalone filter when arriving on Old Tenants from a different entry point. Reframe to keep MUST-SHIP intact: chip is built, but acknowledges its standalone-utility depends on the canonical aggregator landing in V1.

- **TEAM:** **`notice_period_started_date` field does NOT exist in `tenant` entity.** Build Sheet HB5 referenced this field — codebase verification (Phase 2) shows the closest is `notice_raised_on` (line 232 of `entities/tenant.ts`). V1 cuts the notice-window chip entirely (see What we're NOT building) — not worth repointing to a field whose semantics may not match PRD intent.

- **TEAM:** **Currency unit convention (system-wide).** Rent invoices + refunds + credits in this codebase are all in RUPEES (verified via writer trace). CLAUDE.md's "paise (integer) internally" rule applies only to gateway-facing payment processing, not to invoice/refund/credit aggregates. DA-06 V1 hero composes `SD-held + Caution-held + Advance-held` from invoice/refund tables — all rupees, no normalization needed. Build Sheet HB9 (which flagged a potential 100× error if credits got added to the hero) is RESOLVED — `Credits.amount` is rupees; if V1.1 ever adds it, no `× 100` needed.

- **TEAM:** **No `liability_snapshots` table exists — trend chart cannot ship in V1.** Verified by codebase-wide grep. V2 needs to build the table + backfill cron + chart together as one feature. V1 ships hero + sections only; the "Trend coming soon — collecting data from launch" line in the screen header is the honest framing.

- **USER:** **Future date ranges show ₹0 (n/a for live screen).** Time chip is rendered for cross-tab consistency but DA-06 ignores it; helper text confirms "Liabilities is always-live; period filter doesn't apply here."

- **USER:** **Empty + new-account states.** New properties (<30 days old) with no held deposits show *"No tenants currently held against"* instead of ₹0 in a vacuum. Move-Out Forecast and Refunds Pending sections hide entirely when their respective lists are empty.

**The four risks:**

| Risk | Read | Mitigation |
|------|------|------------|
| Will operators use it? | HIGH yes | Three named pains. Operators hold significant tenant money daily with no aggregate view today. Excel month-end is the existing workaround — slow, reactive, post-hoc |
| Will they understand it? | MEDIUM | "Held amount," "Move-Out Forecast," "Refunds Pending" need plain explanations. Operator-Hindi labels. Tamil / Telugu / Kannada test. The "live, no MoM" framing is unusual for this DA suite — header pill carries the explanation |
| Can we build it in 2 weeks? | MEDIUM | Honest scope math: 5 NEW BUILD endpoints + HB1/HB3/HB7 unification in new `LiabilityService` + 5 new chips on 3 sibling list endpoints + 3 auth-fix sweep + held-column added to 3 list screens + Mode 211 launch-banner + Refunds-Pending freshness UX. With 1 senior + 1 junior backend over 10 working days = ~20 dev-days available against ~22-25 dev-days estimated scope. **Day-7 mid-cycle go/no-go is mandatory:** if hero + Q3 + Q4 + at least 3 of 5 chips aren't behind a frontend-callable contract by Day 7, cut Q5 (by-property) and Q7 (move-out callout) per the operator-pain-ranked cut order — NOT Q2 (Q2 cuts LAST because it carries the hero drill affordance). The MUST-SHIP base (hero + Q3 + Q4 + chips + auth + held-column) is sacred; the stretch tier flexes. |
| Is it worth it? | STRONG | Liability is the most opaque cost on the operator's balance sheet. Without visibility, reputation risk (delayed refunds) and cash-flow risk (unforecasted exits) accumulate silently. One late refund = a 1-star review = lost prospect inquiries |

**When to stop and reconsider** (sequenced day-by-day, not all-end-of-week):
- **Day 3:** if HB1 unification (`LiabilityService.getDepositsHeld()` wrapping the canonical aggregator) isn't designed → escalate. Do not start breakdown / section work until the canonical aggregator is decided; they'll have to be rebuilt otherwise. **Fallback path while waiting on HB1:** parallelize on the 3 auth-fix sweep (independent of aggregator) + frontend scaffolding for hero + section shells (calls to be wired Day 4-5). Backend doesn't sit idle — the auth fixes are real V1 work.
- **Day 3:** if filter code allocation for the 5 new chips across Bookings / Current Tenants / Old Tenants lists isn't done → escalate. Chips are MUST-SHIP for V1 (drills don't work without them).
- **Day 3:** if the bookings-deposit business-logic check (do bookings typically pay deposit before move-in, or just advance/booking-token?) isn't done with 2 operator interviews → escalate. Chip naming may need to drop "Bookings: Has Deposit" if bookings rarely have deposit pre-move-in. Don't let this drift to Day-7 — backend will have built the wrong filter by then.
- **Day 1 (kickoff conversation):** designer + eng align on the "Live · updated now" pill cache strategy. Defer past Day-1 risks the pill UX surfacing as an open question in Day-5 review when the screen is already wired.
- **Day 5:** if HB1 unification + Mode 211 filter (HB3) isn't merged → ship V1 with the Refundable chip whose standalone-drill utility is degraded (DA-06's Refunds-Pending section still works internally because it calls the per-tenant composite directly). Chip itself stays MUST-SHIP; what changes is whether operators arriving on Old Tenants from a different entry point can rely on the chip's filter being canonical-correct.
- **Day 7 (mid-cycle gut check):** if hero + Q3 + Q4 + at least 3 of 5 chips aren't behind a frontend-callable contract by Day 7, **cut Q5 (by-property) and Q7 (move-out callout) per the operator-pain-ranked cut order** — NOT Q2 (Q2 cuts LAST because it carries the hero drill affordance). The MUST-SHIP base is sacred; the stretch tier flexes. If even Q3+Q4 lists aren't contracted by Day 7, full reset conversation with eng lead — not a "push harder" moment, a "scope is wrong" signal.
- **End of week 1:** if the 3 auth-fix sweep on `/fetchTenants` + `/fetchEvictedTenants` + `/fetchBookings` isn't on track → flag and escalate. The screen drilling to anonymous endpoints = no-ship blocker (user PII at risk, actively advertised by the screen).
- **End of week 1:** if held-amount column hasn't been added to the 3 list screens → flag. The chips work without the column but the operator's drill loses utility (lands on filtered list, can't see who has the most held).
- **Post-launch:** if support tickets about "the liability number is wrong" or "the screen disagrees with the homepage" double the current baseline → trust is broken. Fix the data path before adding features.

---

## V2 scope (deferred from V1 — codebase-grounded gap)

V0.1 already scopes V1 lean (no trend, no Open Settlement composite, no anomaly callout, no notice chip, no settlement-readiness badge, no multi-invoice refunds). **The Phase 2 LAYER 2 exploration surfaced additional dependencies** the Brief explicitly defers to V2. See `[[DA-06 Codebase Exploration]]` for the full grounding.

**Top V2 questions (operator-pain-ranked, with owner + candidate cycle):**

| # | Question | Owner | Candidate cycle |
|---|----------|-------|-----------------|
| 1 | **"Show me liability trend over the last 6 months."** No `liability_snapshots` table exists; V2 builds table + nightly cron + backfill + chart together. Codebase grounding: zero hits on `liability_snapshot\|deposit_balance_history\|daily_balance` across `src/entities/`. | Jatin (Eng) + Sanchay (PM) | V2 cycle after V1 launch + 30 days of snapshot data |
| 2 | **"Open Settlement from inside DA-06."** 5-endpoint composite (`/checklist/move-out/items` → `mark-item` → `complete-checklist` → `/tenant/adjustDeposit` → invoice creation). Blocked on P0 auth fix on adjustDeposit. Codebase grounding: `routes/tenant.ts:944`. | Jatin (Eng) | V1.1, after P0 auth ticket lands |
| 3 | **"Anomaly callout — surface stale or outlier deposits."** > 12 months no ledger activity OR ≥ 2.5× property avg AND ≥ ₹25K AND property ≥ 5 deposit-holders. | Sanchay (PM) | V1.1, after V1 validates canonical aggregator |
| 4 | **"Property-rollup MoM for Priya"** — cross-property trend even though individual liability is a stock. Needs same `liability_snapshots` table as #1. | Sanchay (PM) | V2 (paired with #1) |
| 5 | **"Process Refund mutation from inside DA-06"** — V1 is read-only end-to-end; in-screen refund button is V1.1 if operators ask. | Jatin (Eng) | V1.1 |

**V1 → V2 architecture decisions (close before V2 build):**

- **Schema migration: `liability_snapshots` table** — daily per-property record of held total + composition. Powers the trend chart + future "growth rate" callouts. Low risk (write-only nightly cron).
- **Schema migration: `refund_status` field on tenants** — eliminates the computed-filter complexity of Refunds-Pending. Operators get a hard fact instead of a derived one.
- **Schema migration: `multi_invoice_refund` support** — `Refunds` table gets a many-to-many relationship to `Invoices` (or a junction table). Lets operators refund across multiple deposit invoices in one operation. Touches the refund creation flow.
- **`/tenant/adjustDeposit` auth fix (P0, separate ticket).** Add HeaderValidator + checkAuth + pg_id validation. Unblocks the Open Settlement composite for V1.1.
- **Anomaly query infrastructure.** Per-tenant deposit-aging + per-property average computation. Needs the canonical aggregator from V1's HB1 unification.

See `[[DA-06 Codebase Exploration]]` for the full LAYER 1 + LAYER 2 dependency map and detailed architecture risks.

---

## Footer

**Things to test with operators** (don't block kickoff; settle before launch):
- 3-operator check on the Hero language — does *"Total Held"* land, or do plainer Hindi phrasings (*"Tenants ka jama paisa"*) land better?
- 3-operator check on the Refunds-Pending framing — does it land as a *"my outstanding refund queue"* (helpful) or as *"why haven't you refunded these yet?"* (accusatory)?
- 3-operator check on the homepage-card divergence — when operators see DA-06 hero include Advance but homepage card excludes it, do they read the subtitle disclosure correctly? Or do they assume the numbers disagree because of a bug?
- Bharat-language testing — Tamil / Telugu / Kannada. Owner: Sanchay (PM) coordinates with operator network; target = week 2 of build cycle.
- Chip discoverability on Bookings / Current Tenants / Old Tenants lists — operators landing from DA-06 drill should recognize within 5 seconds that the list is filtered (chip state visible, easy to clear).

**Cross-Brief propagation (DA-06 owns the cascade):**

**Reality check:** the 3 target tenant list screens (Bookings, Current Tenants, Old Tenants) do NOT have existing Briefs in this vault (verified — no `*Booking Brief.md` / `*Tenant List Brief.md` etc). DA-05's chip-propagation pattern went into DA-02's existing Brief; DA-06's chip-propagation has nowhere existing to land. **V1 propagation strategy:**

- **Document the chip + auth + held-column scope additions directly in the DA-06 Build Sheet** (not a separate Brief per list screen — overkill for the chip scope). Build Sheet becomes the single source of truth for what changes across the 3 list endpoints.
- **Bookings list** — adds 2 chips (Has Advance, Has Deposit) + auth fix on `/fetchBookings` + held-amount column. Owner: Jatin (Eng) for auth + chip + column; Sanchay (PM) for chip-naming validation. **Open question for designer + 2 operator interviews:** do bookings in this property model typically pay a deposit BEFORE move-in, or just an advance/booking-token? If bookings rarely pay deposit, "Bookings: Has Deposit" may have low utility — cut to single "Has Advance" chip on Bookings.
- **Current Tenants list** — adds 2 chips (Has Advance, Has Deposit) + auth fix on `/fetchTenants` + held-amount column. Standard expected pattern.
- **Old Tenants list** — adds 1 chip (Refundable, computed via `tenant.status NOT IN (1,2) AND held_amount > 0`) + auth fix on `/fetchEvictedTenants` + held-amount column. Computed-filter complexity flagged in chip Trap.
- **Homepage Deposits Held card** — strategic punt: card and DA-06 hero will disagree by Advance amount until card is also extended. Brief recommends extending the card in a follow-on cycle for full reconciliation; doesn't block V1.
- **DA-02 / DA-03 / DA-04 / DA-05 Briefs** — no scope changes required from DA-06 V1; confirmed no cross-impact beyond shared `viewInvoices` permission key (already used).
- **Pattern divergence from DA-05:** DA-05 v0.3 propagated to DA-02's existing Brief (bilateral updates v0.7.2/3/4). DA-06's targets have no Briefs — propagation lives in DA-06 Build Sheet. The "create Brief for each list screen" alternative was rejected as scope creep — those list screens are legacy plumbing, not features warranting their own PM artifact.

**Related docs:**
- Engineering spec (post-design): `[[DA-06 Build Sheet]]`
- Legacy PRD (historical): `[[DA-06 Liabilities Detailed Analytics]]`
- Codebase ground truth: `[[_Ground Truth Field Map]]`
- Phase 2 grounding: `[[DA-06 Codebase Exploration]]` (LAYER 1 + LAYER 2 findings — feeds V1 traps + V2 scope)
- Sibling Briefs: `[[DA-01 Brief]]` · `[[DA-02 Brief]]` · `[[DA-03 Brief]]` · `[[DA-04 Brief]]` · `[[DA-05 Brief]]`
- Design system + Figma: designer to lock the frame by end of week 1 (frame ID added to this row on lock)
- Cross-suite engineering blockers: `[[DA-06 Liabilities Detailed Analytics#Cross-Suite Engineering Blockers]]`

**Key decisions locked:**
- Hero formula: extended `paid SD + paid Caution + paid Advance − refunds − Mode 211` (diverges from homepage canonical which excludes Advance; subtitle discloses Advance contribution for honest reconciliation).
- Live snapshot — DA-06 ignores the global time chip; no MoM comparison shown (liabilities are stocks, not flows). Header pill "Live · updated now."
- Hero source: derive from invoice + refund + payment tables ONLY. Never read `tenant.security_deposit` snapshot column (anonymously writable today).
- HB1 unification: wrap canonical aggregator in `LiabilityService.getDepositsHeld()`. Route DA-06 hero + breakdown + sections + homepage card through it. Day-3 lock.
- HB3 fix: add `payment_mode != 211` filter to canonical aggregator. Fix lives in `LiabilityService`.
- HB7 fix: use `DEPOSIT_DUE_TYPES` (plural) — includes Caution Money. One-line fix in `getAvailableDeposit`.
- Refunds-Pending = computed filter (`tenant.status NOT IN (1,2) AND held > 0`); no `refund_status` field exists. V2 may add the field.
- Trend chart: deferred to V2 entirely (no `liability_snapshots` table exists). Honest "Trend coming soon" framing in V1 header.
- Open Settlement composite: deferred to V1.1. Blocked by P0 auth fix on `/tenant/adjustDeposit` (separate backend ticket).
- Process Refund mutation surface: deferred to V1.1. V1 is read-only end-to-end.
- Notice-window chip: cut entirely (`notice_period_started_date` field doesn't exist).
- Anomaly callout: deferred to V1.1.
- Per-tenant settlement readiness flag (✅/⏳): deferred to V1.1.
- Multi-invoice refund: V2 schema work.
- Permission gate: reuse `viewInvoices` (composite-view rationale). No new key built in V1.
- Reconciliation: hero SD+Caution portion must match homepage Deposits Held card to the rupee. **Pre-V1 QA gate:** 10-property sample on SD+Caution portion.
- ⓘ icon: single-tap → bottom sheet. No tooltip, no long-press (per DA-01/02/03/04/05 convention).
- Default property context: inherited from dashboard (single → single; multi → multi-property aggregate).
- V1 must-ship: Q1 (hero) + Q3 (Move-Out Forecast) + Q4 (Refunds Pending) + 5 new filter chips on Bookings/Current/Old Tenants lists + 3 auth fixes (CSB-3 sweep on those list endpoints) + held-amount column on 3 list screens. Protect at stretch: Q2 (composition), Q5 (by-property), Q6 (refunds-overdue callout), Q7 (move-out callout). Cut order if time tightens (operator-pain-ranked): **Q5 → Q7 → Q6 → Q2** (Q2 cuts LAST — carries hero drill affordance). **Day-7 mid-cycle gate cuts the top 2 from this order (Q5 + Q7) if hero/Q3/Q4 + 3-of-5 chips aren't behind a frontend-callable contract.**
- Auth fix on `/fetchTenants` + `/fetchEvictedTenants` + `/fetchBookings`: no-ship blockers (DA-06 actively advertises which tenants to attack; drilling to anonymous endpoints regresses PII surface).
- Drill chain (rule, not UI prescription — designer picks entry points): hero composition tap → Current Tenants list filtered; Move-Out Forecast row → Current Tenants list filtered + sorted by exit date; Refunds Pending row → Old Tenants list filtered to Refundable; per-tenant row → Tenant Passbook (deposit tab). All drills land on EXISTING tenant list screens with NEW filter chips.

**Changelog:**

| Date | Version | Change |
|------|---------|--------|
| 2026-05-18 | v0.2.2 | **Kickoff polish — 4 nice-to-haves from independent verification sub-agent.** v0.2.1 was approved-to-proceed by the verification agent; these are quality polish, not substantive fixes. (1) **Risk label MEDIUM-LOW → MEDIUM.** Verification agent quibbled the math: 22-25 dev-days vs ~20 available with 1 senior + 1 junior (junior <1.0 effective velocity) + 3 frontend touches + cross-team coordination = real overrun risk. Day-7 gate stays as scope-safety mechanism; label honest-disclosed as MEDIUM. (2) **Day-3 deadline for bookings-deposit business-logic check.** Without it, the open question ("do bookings pay deposit pre-move-in?") drifts to Day-7 and backend builds the wrong chip filter. Now: Day-3 escalate if 2 operator interviews haven't happened. (3) **Day-1 kickoff conversation for "Live · updated now" cache strategy.** Deferring past Day-1 risks the pill UX surfacing as open question in Day-5 review when wiring is done. (4) **KDL cut-order line gained a one-line addendum** naming the Day-7 gate behavior (top 2 from cut order = Q5 + Q7) — makes the cross-reference bulletproof for the eng-lead skim-read. |
| 2026-05-18 | v0.2.1 | **CPO final-read pass — caught 5 residual issues the v0.2 critique missed.** Same pattern as DA-05 v0.4.1: critique sub-agent caught the major substantive issues; manual systematic re-read with codebase verification caught the architectural/consistency issues that needed grep to detect. **Fixes:** (1) **Day-7 cut-order contradicted operator-pain-rank** — risk-table + Day-line escalation said "cut Q2 and Q5" but the body cut order has Q2 cutting LAST (per operator-pain-rank: Q5 → Q7 → Q6 → Q2). Day-7 now cuts Q5 + Q7 (top 2 per the rank), preserving Q2 as last-to-cut. Same drift pattern that bit DA-04 v0.2 and DA-05 v0.4 — caught BEFORE kickoff this time. (2) **Filter code mechanism mismatch** — Brief said "filter codes from each list endpoint's own range" but codebase grep confirmed only `q_1201`–`q_1219` exist anywhere, all in `src/v1/list_screens/collections`. The 3 target tenant list endpoints (`/fetchTenants`, `/fetchBookings`, `/fetchEvictedTenants`) are LEGACY routes in `src/controllers/tenant.ts` — they don't use the `q_NNNN` system at all. Reframed: V1 chip implementation uses ad-hoc query params on the legacy endpoints (not `q_NNNN`); architectural asymmetry flagged for follow-on "migrate tenant lists to v1/list_screens/" cycle. (3) **Cross-Brief propagation overstated existence of targets** — Brief said "Bookings list Brief / Build Sheet — add chips" but no such Briefs exist (verified by ls). Reframed: propagation lives in DA-06 Build Sheet (not separate Briefs per list screen — those legacy lists don't warrant their own PM artifact). Added bookings business-logic open question: do bookings typically pay deposit before move-in, or just advance/booking-token? (4) **Refunds-Pending freshness mitigation crossed into UI prescription** — was specifying *"timestamp the count visibly as 14:32"* + *"expandable help row above the list"*. Reframed to information needs (freshness signal + operator education + source-of-truth treatment) — designer picks the surface. (5) **Mode 211 banner UI prescription same scrub** — was specifying "one-time in-app banner" UI. Reframed to information needs (one-time operator-facing explanation surface — banner/bottom-sheet/inline-note designer's call). **Pattern note:** v0.2 critique caught substantive blockers (cut-order, hero gate, cross-Brief propagation) but the agent didn't grep the actual codebase for filter-code mechanism or check whether the target Briefs existed — those needed manual verification. CPO final-read is non-negotiable for catching these. |
| 2026-05-18 | v0.2 | **Critique pass — blockers + cross-Brief + operator-lens completeness.** Sub-agent critique returned 3 blockers + 6 high + 6 medium. Fixes applied: **(3 BLOCKERS)** (1) Cut-order inconsistency resolved — Refundable chip stays MUST-SHIP; Day-5 reframed as "standalone-drill utility degrades" not "chip cuts." (2) Hero formula no-ship gate rewritten — was logically circular (asked for total-match while extended formula guarantees total-divergence); now precisely tests SD+Caution sub-component match, with full total legitimately differing by Advance. Cross-screen strategic punt on homepage card acknowledged explicitly. (3) Cross-Brief propagation moved from changelog into body — new Footer section names Bookings + Current Tenants + Old Tenants list Briefs as scope-cascade targets with owners; flags homepage card divergence as strategic punt. **(6 HIGH)** (4) Mode 211 phantom-liability mitigation specified — one-time in-app banner + operator-testing + support-team briefing. (5) Refunds-Pending count fluctuation mitigated — timestamped count + inline "Why did this change?" help. (6) HB1 Day-3 lock got a fallback path (parallelize auth-fix sweep + frontend scaffolding). (7) Scope honesty — risk-table downgraded MEDIUM → MEDIUM-LOW; Day-7 mid-cycle go/no-go added. (8) Permission gate UX specified — tab HIDDEN for operators without `viewInvoices`, not empty-state. (9) "Live · updated now" pill defined — fresh on each open, pull-to-refresh, no cache; Priya MoM-at-property-rollup flagged as V2 follow-up. **(OPERATOR-LENS)** (10) 3 auth fixes reframed with operator-side cost ("PII regression worse than today" — cannot be moved to fast-follow). (11) Held-amount column on 3 list screens given explicit operator-mental-model justification (operator needs to see WHO has the most, not just WHICH tenants match). **(MEDIUM)** (12) Credits.amount-is-rupees trap trimmed to one-liner (credits not in V1 scope). (13) Q2 cut-LAST reasoning expanded (hero drill affordance breaks without Q2). (14) P0 adjustDeposit auth-fix ticket owner named (Jatin). (15) Bharat-language testing owner named (Sanchay). (16) V2 scope got owner + candidate-cycle table (was bulleted list — drifts without owners). |
| 2026-05-18 | v0.1 | **Initial Brief** — written using `/brief` skill with Phase 2 LAYER 1 + LAYER 2 discipline from kickoff (per 2026-05-16 learning). **Phase 2 LAYER 2 caught dependencies the Build Sheet doesn't fully resolve:** (1) Hero formula divergence — homepage canonical excludes Advance, Build Sheet stated includes Advance; resolved as extended formula with honest subtitle disclosure on Advance contribution. (2) HB9 RESOLVED — `Credits.amount` is in rupees not paise (verified at `entities/credits.ts:67` + writers at `controllers/credits.ts:494,587`); Build Sheet HB9 was false alarm. (3) HB5 correction — `notice_period_started_date` field doesn't exist in tenant entity (closest is `notice_raised_on`); cut the notice chip entirely. (4) `tenant.security_deposit` snapshot column is anonymously writable (via P0 auth bug on `/tenant/adjustDeposit`); Brief mandates derivation from invoice/refund/payment tables only. (5) `getAdvanceAmount` at `helpers/payments.ts:647` is broken (returns 0 unconditionally); DA-06 must use `getAvailableAdvance` at `paymentService.ts:2820` instead. (6) No `liability_snapshots` table exists; trend chart deferred to V2. (7) No `DepositService` exists — deposit logic scattered across 5 files; HB1 unification creates new `LiabilityService`. (8) 3 drill-target list endpoints (`/fetchTenants` + `/fetchEvictedTenants` + `/fetchBookings`) all have ZERO auth — no-ship blockers. **All 8 prescribed Phase 3 gather questions asked (4 critical scoping + 3 follow-up batches; persona + time budget inherited from DA-01-05 suite — defaults).** Persona = same Rajesh-led trio. Rhythm = weekly + event-triggered + monthly (~6-8×/month). Scope = LIVE snapshot only (unique in DA suite — no MoM). Time budget = 2 weeks. NOT building V1 = trend chart (V2), Open Settlement composite (V1.1, blocked on P0 auth fix), Process Refund mutation surface (V1.1), notice chip (cut — field doesn't exist), anomaly callout (V1.1), settlement readiness badge (V1.1), multi-invoice refund (V2). Drill chain: hero composition + Move-Out Forecast + Refunds Pending all drill to EXISTING tenant lists with 5 new filter chips inherited as MUST-SHIP V1 scope across 3 sibling list endpoints (Bookings: Has Advance + Has Deposit; Current Tenants: Has Advance + Has Deposit; Old Tenants: Refundable). Traps labeled USER: / TEAM: from line 1. V2 scope pointer with top 3 deferred questions (trend chart, Open Settlement composite, anomaly callout) + 5 V1→V2 architecture decisions. **Cross-Brief impact:** Bookings + Current Tenants + Old Tenants list screens get new chips + auth fixes + held-column in V1 — needs separate Brief or Build Sheet update for those screens. Same scope-expansion pattern as DA-05 v0.3 (chips on DA-02). |
