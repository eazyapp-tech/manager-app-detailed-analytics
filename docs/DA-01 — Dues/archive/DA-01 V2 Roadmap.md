---
title: DA-01 V2 Roadmap — Dues Detailed Analytics
date: '2026-05-01'
tags:
  - rentok
  - prd
  - roadmap
  - v2
  - homescreen
  - financial-insights
  - detailed-analytics
  - dues
aliases:
  - DA-01 V2 Roadmap
  - DA-01 Roadmap
  - Dues Analytics V2 Plan
status: Draft for PM Review
version: '1.1'
companion_to: DA-01 Dues Detailed Analytics
---
o> [!INFO] Source of Truth
> Local spec: `/Users/eazypg/RentOk Manager Homescreen/spec/DA-01-V2-roadmap.md`
> Companion to V1 spec: [[DA-01 Dues Detailed Analytics]] (v3.1, V1 locked)
> Codebase verified via graphify queries on `EazyPg2.0` graph + direct reads of `helpers.ts`, `service.ts`, `getFinancialsV2`, AI rules (AI-018, AI-019), business rules (FIN-135), homescreen spec (07a-pending-dues.md)

# DA-01 — Dues Detailed Analytics · V2 Roadmap

## 0. The One-Page Brief

**V1 ships a competent analytics dashboard.** It answers "what's stuck, where, and who do I call." The reconciliation invariants hold. The drill-downs are correct. Engineering can build it in one sprint.

**V1 does NOT yet do the things that make this a senior-PM-grade analytics product:**

- It is a **snapshot**, not a story. No trends, no decomposition, no "why."
- It is **reactive**, not predictive. No forecasts, no risk, no projections.
- It is **single-lens**, not multi-lens. No cohorts, custom ranges, saved views, or comparison.
- It is a **destination**, not a system. No digests, no alerts, no scheduled reports.

**V2 closes those gaps in three release windows over Q3–Q4 2026** without depending on RentOk AI (which is Tier 7 in the build order and ships later as its own product motion).

**Strategic frame:** V2 makes Dues Detailed Analytics the *trusted second-derivative* layer of Financial Insights. V1 tells you what is. V2 tells you what's changing, why, and what's coming.

---

## 1. V1 Status — Alignment Issue Resolved (2026-05-01)

**Status: RESOLVED.** PM has confirmed the simpler current-state model wins. Both DA-01 v3.1 and homescreen 07a have been updated to use identical 3-bucket semantics: Carried Forward / Due in Period / Due After Period — current-state decomposition by due-date cohort. No historical balance reconstruction.

### The original conflict (recorded for posterity)

**DA-01 v3.0 range-mode urgency (3 buckets):**
- Carried Forward: `due_date < period_start`
- Due in Period: `due_date IN [period_start, period_end]`
- **Due After Period: `due_date > period_end`** ← future-dated unpaid bills

**Homescreen 07a-pending-dues.md historical-mode buckets (3 buckets, original):**
- Carried Forward Dues: already unpaid before period started
- Due in Selected Period: became due during the period
- **Still Unpaid at Period End: open at period end** ← state-at-period-end

These were fundamentally different semantics — homescreen implied historical reconstruction, which DA-01 explicitly killed.

### Resolution applied

- **DA-01 v3.1:** range-mode buckets remain Carried Forward / Due in Period / Due After Period (current-state decomposition).
- **07a-pending-dues.md:** historical-mode buckets renamed and redefined to match — same labels, same semantics. Time Matrix updated to clarify total is always live regardless of period.
- **Code path:** `getFinancialsV2:1888-1890` is the source of truth for both screens. Single query, single semantic, no historical reconstruction.

For exact past-timestamp balances (rare, formal accounting), accountants pivot Invoice + Payment exports in Excel/Tally. Frozen-period semantics for monthly book-closing arrives in V2.1 (P1-#16).

---

## 2. What V2 Is — and Isn't

**V2 is:** the analytics depth, predictive layer, scheduled distribution, and multi-lens analysis that turns this from a "dashboard" into an "analytics product."

**V2 is NOT:**
- A workflow tracker (assignments, follow-up queues, promised payments) — that's homescreen and worklist territory, not analytics
- An AI insights surface (RentOk AI is Tier 7, ships separately, V3+ for analytics integration)
- A two-way comm system (WhatsApp replies feeding back) — infrastructure project, not analytics
- A collaboration tool (notes, mentions, assignments) — cross-app feature
- A re-architecture — V2 builds on V1's foundation; the buckets, drill-downs, reconciliation invariants stay

**Persona scope for V2:**
- **Rajesh, Priya, Amit, Meena** — actively served by V2.0 → V2.2.
- **Accountant** — served via V2.1 frozen-period semantics + existing FY filter + Excel pivot path.
- **Owner-investor** — served exclusively via V2.2 WhatsApp digest (P2-#21). They are NOT expected to open the app. The digest is their full surface.

**A note on Meena's role-scope view:** Earlier consideration of a "My Tenants" default view for Meena is dropped — there is no tenant-to-team-member assignment concept in the codebase, and PM has confirmed it's not practically needed. Meena's needs in V2 are met via permission-gated surfaces (V1) plus saved views (V2.1 P1-#11) for self-curated subsets if she wants them.

---

## 3. Verified Codebase Inventory (the V2 substrate)

### What exists and is leverage-able

| Capability | Code reference | V2 use |
|-----------|---------------|--------|
| Dues 4-bucket today-mode urgency | `getFinancialsV2:1883-1886` | V1 baseline |
| Dues 3-bucket range-mode urgency | `getFinancialsV2:1888-1890` | V1 baseline |
| Property-level grouping in worklist | `buildBaseQuery(_, 'property')` | V1 multi-property |
| Multi-property scope filter | `pg_number_filter` param | V1 multi-property |
| 14 widget filter codes (1102-1114) | `DuesFilterCode`, `getWidgetFilterCodes` | V1 + extensible for new buckets |
| 60-day binary defaulter classification | `applyDefaulters`, FIN-135 | V1 (binary); V2 extends to 5-bucket |
| AI dues NL search (text → filter codes) | `SemanticParsingService`, `DuesParsingService`, AI-018, AI-019 | Future hook for V3+ AI insights |
| LLM tool call logging | `repositories/llmToolCallLog.ts` | V3+ AI observability |
| WhatsApp alert service | `services/whatsappAlert/sendWhatsappAlert.ts`, `sendWhatsapp7218()` | V2 push triggers |
| Cloud-task report scheduler | `cloudTaskReportScheduler` (`commonFunctions.ts:725`) | V2 scheduled digest |
| Daily report PDF generator | `services/reports/dailyReportPdf.ts`, RPT-050 | V2 scheduled PDF export |
| Email digest pattern | `services/complaints/emailDigestService.ts` | V2 reusable pattern for dues digest |
| Cron-scheduled campaign dispatch | REV-014 | V2 reusable pattern |
| Activity log infrastructure | `addActivityLog()` in evictionService etc. | V2 audit + history |

### What does NOT exist (V2 net-new)

| Capability | Status |
|-----------|--------|
| 5-bucket aging classification (1–7d, 8–15d, 16–30d, 31–60d, 60+d) | Net-new — DATEDIFF buckets |
| Forecast / projection logic for future dues | Net-new — no time-series infrastructure exists |
| Per-tenant risk scoring | Net-new |
| MoM "why" decomposition | Net-new |
| Saved views / saved filters | Net-new — verified via grep, zero hits |
| Custom date range support beyond fixed periods | Net-new — current API uses filter codes mostly |
| Multi-property side-by-side comparison view | Net-new — current data is aggregated, not parallelized |
| Trend visualization (sparkline, line chart) for KPIs | Net-new |
| Repeat defaulter detection (recurrence over 6-month lookback) | Net-new |
| Push notification triggers from analytics thresholds | Net-new (templates exist; trigger logic doesn't) |
| Cohort dimensions beyond population/category | Net-new |

---

## 4. The Four Investment Themes

### Theme 1 — Snapshot → Story
**Operator question evolves from:** "What's stuck right now?" → "How is it changing, and why?"
**Features:** Trend sparklines on every KPI · Historical line chart for Total Dues, Collection Efficiency, 60+ day exposure · MoM "why" decomposition · "First time you saw this" markers · Compare-to-self
**Success metric:** % of users tapping a sparkline / "why" / trend ≥ 1×/week.

### Theme 2 — Reactive → Predictive
**Operator question evolves from:** "What is?" → "What will be? What's at risk?"
**Features:** Month-end forecast · Quarter / FY forecast · At-risk amount · Per-tenant default-risk indicator · Recovery probability · Forward-looking projection mode
**Success metric:** Forecast accuracy. Target: ±5% by V2.2.
**Note:** Statistical, not ML. Linear regression on history. ML/AI versions are V3+.

### Theme 3 — Single-view → Multi-lens
**Operator question evolves from:** "Show me my dues." → "Show me my dues sliced by [X]."
**Features:** Custom date range picker · Saved views · Cohort dimensions (tenant tier, agreement type, booking source, room type, city/zone) · Multi-property comparison view · "Compare this period to..." selector · URL-shareable view state
**Success metric:** Avg distinct views/saved-views per power user per week. Target: 3+ for Priya persona.

### Theme 4 — Dashboard → System
**Operator question evolves from:** "I'll check the app when I have time." → "RentOk tells me when something matters."
**Features:** Daily/weekly scheduled email digest · Scheduled PDF export · Push notifications on threshold crossings · "Share this view" snapshot · WhatsApp owner digest · In-app proactive cards
**Success metric:** % of users with ≥ 1 active digest subscription. Target: 60% of multi-property accounts by V2.2.

---

## 5. Prioritized Feature List

### P0 — Must ship in V2.0 (~Q3 2026)

| # | Feature | Theme | Effort | Why P0 |
|---|---------|-------|--------|--------|
| 1 | **5-bucket aging classification** (1–7d, 8–15d, 16–30d, 31–60d, 60+d) | 1 | M | V1 specifies the visual. Filter is binary today (FIN-135). Most-asked operator analytic. New filter codes + DATEDIFF SQL. |
| 2 | **MoM "why" decomposition** on the chip | 1 | M | V1 shows ▲8% with no explanation. Decomposition into +new bookings / +late fees / +rebookings / −collections is purely SQL on existing tables. |
| 3 | **Custom date range** picker | 3 | S | Filter codes today have fixed periods. `applyDateRange` already supports arbitrary ranges; UI is the gap. |
| 4 | **Trend sparklines** on Total Dues, Collection Efficiency, 60+ day exposure | 1 | M | Add 12-month rolling KPI history table; render as compact line chart. New cron + cache. Establishes trend infra all Theme-1 features depend on. |
| 5 | **Repeat Defaulter detection** (3 of last 6 months) | 1 | M | V1 promises the badge but defers detection. Subquery on monthly overdue history. |
| 6 | **Late Fee Section** (analytical, not workflow) | 1 | S | Pulled forward from V1 deferred. 3-metric tile, not full module. Code exists (`DuesFilterCode.LATE_FINE`). |
| 7 | **Multi-property comparison view** (side-by-side) | 3 | M | Priya's #1 unmet need. V1 has aggregated rollup; V2 needs columnar comparison. New endpoint. |
| 8 | **Scheduled email digest** (weekly default, daily for accounts >₹5L overdue) | 4 | M | Leverages `cloudTaskReportScheduler` + reusable email digest pattern. Highest-leverage Theme-4 feature. |

### P1 — Ship in V2.1 (~Q4 2026)

| # | Feature | Theme | Effort | Why P1 |
|---|---------|-------|--------|--------|
| 9 | **Month-end forecast** (with confidence band) | 2 | M | Linear regression on last 6 months. Statistical, not ML. |
| 10 | **At-risk amount** ("₹X likely to cross 60 days in 14 days") | 2 | M | Per-tenant velocity model. Statistical heuristic. |
| 11 | **Saved views** | 3 | S | New `dues_saved_views` table. Stores filter combination + name. |
| 12 | **Cohort dimensions** (tenant tier, agreement type, booking source) | 3 | M | New filter clauses. Cohort tables already in other domains. |
| 13 | **Push notification triggers** (threshold-based) | 4 | M | New `analytics_alerts` table. Cron runs hourly. WhatsApp template exists. |
| 14 | **"Share this view"** export | 4 | S | Server-side HTML/PDF render. Signed link, 7-day TTL. |
| 15 | **Forward-looking mode** (future-dated projections) | 2 | S | Renders existing future invoices as a forward urgency bar. |
| 16 | **Frozen-period / book-closing semantics** | — | M | Accountant workflow. Mark March closed; retroactive March payments create adjustment entries. Cross-domain. |

### P2 — Ship in V2.2 (~Q1 2027)

| # | Feature | Theme | Effort | Why P2 |
|---|---------|-------|--------|--------|
| 17 | **Recovery probability** ("22% of 60+ day defaulters paid eventually") | 2 | M | Historical analysis on 6-month moving window. Real moat. |
| 18 | **Per-tenant risk indicator** on worklist | 2 | M | Composite score. Surface as Low/Med/High pill. |
| 19 | **In-app proactive cards** (anomaly nudges) | 4 | M | "Your 60+ bucket grew unusually fast this week." Statistical anomaly detection. |
| 20 | **Quarter / FY forecast** | 2 | S | Extension of P1 month-end forecast. |
| 21 | **WhatsApp owner digest** (end-of-week one-card) | 4 | S | Targeted at owners not on app. Single WhatsApp message. |
| 22 | **Compare to self** ("vs your 12-month avg," "vs last quarter") | 1 | S | Richer comparison baselines. |
| 23 | **First-time-seen anomaly markers** on trend chart | 1 | S | When a metric crosses a threshold for first time in 12 months, mark it. |

### V3+ — Explicitly Deferred

| # | Feature | Why deferred |
|---|---------|--------------|
| AI-augmented "Why" insights (RentOk AI integration) | RentOk AI is Tier 7. AI-018 / AI-019 plumbing exists for NL search, not insight generation. Wait until RentOk AI's analytics tools mature. |
| Conversational analytics ("Why up 8%?") | Same as above |
| ML-based default prediction | Statistical heuristics in V2 will be 80% as good. Not Phase 2. |
| Cohort benchmarking (your 27% vs industry median) | Requires anonymized cross-account data, fairness review, opt-in. |
| Two-way WhatsApp loop (replies feeding back) | Infrastructure project across WhatsApp domain. Not analytics-team scope. |
| Collaboration / assignments / notes | Worklist + homescreen territory. |
| Snapshot Reports (true historical reconstruction) | Excel pivot of exports already serves accountants. Build only if accountants ask for it. |

---

## 6. Sequencing & Release Windows

### V2.0 — "The Stories Behind the Numbers" (Q3 2026)
**Operator promise:** "Now you know not just what is stuck, but what's changing, why, and which categories matter."
**Ships:** P0 features 1–8.
**Build sequence:** 5-bucket aging → "Why" decomposition → trend sparklines → custom date range → late fee section → repeat defaulter → multi-property comparison → email digest.
**Critical path:** 5-bucket aging unblocks several other features by establishing aging-window query infrastructure.

### V2.1 — "What's Coming, What's at Risk" (Q4 2026)
**Operator promise:** "Now you can see what month-end will look like, who's about to slip, and you can save your favorite views."
**Ships:** P1 features 9–16.
**Critical path:** Forecast infrastructure → month-end forecast → at-risk → saved views → cohort dimensions → push triggers → share view → forward-looking mode → frozen periods.

### V2.2 — "The Trusted Second Brain" (Q1 2027)
**Operator promise:** "Now RentOk thinks ahead with you."
**Ships:** P2 features 17–23.
**Critical path:** Recovery probability + per-tenant risk → proactive cards → quarter/FY forecast → WhatsApp owner digest → polish features.

---

## 7. Cross-Domain Dependencies

| V2 capability | Depends on | Owner |
|--------------|------------|-------|
| Email digest | Email infrastructure (RPT domain), `cloudTaskReportScheduler` | Reports / Platform |
| Push notification triggers | WhatsApp templates, push infra (`sendWhatsapp7218`, mobile push) | WhatsApp & Comms |
| Forecast / risk | Tenant payment history (Payment domain) | Payment & Settlement |
| Cohort dimensions: tenant tier | Rewards & Credits (`tenant_membership`) | Rewards |
| Cohort dimensions: booking source | Booking & Leads | Booking |
| Frozen periods | Reports, Payment, Billing | Cross-domain |
| WhatsApp owner digest | WhatsApp templates (new ones needed) | WhatsApp & Comms |
| Trend sparklines | New `dues_kpi_history` table populated by daily cron | Platform / Reports |

---

## 8. Success Metrics — Measurement Plan

### North-star metrics

| Metric | Definition | Target by V2.2 |
|--------|-----------|----------------|
| **Detailed Analytics weekly active rate** | % of accounts opening Detailed Analytics ≥ 1× per week | 50% (V1 baseline TBD) |
| **Time-to-first-action** | App-open → first reminder/payment/call | <90 sec (matches V1 vignette promise) |
| **Collection efficiency improvement** | Cohort-level efficiency 30/60/90 days post-V2 rollout vs pre-V2 | +5pp at 90 days |
| **New-account setup velocity (Amit)** | Median days from signup → 70% of beds with active tenants AND ≥10 invoices created | ≤14 days |
| **New-account analytics-screen retention (Amit)** | % of accounts opening Detailed Analytics in week 2 (after week 1 first session) | ≥40% |

### Per-theme metrics

| Theme | Metric |
|-------|--------|
| Snapshot → Story | % of users tapping a sparkline / "why" decomposition / trend view ≥ 1× per week |
| Reactive → Predictive | Forecast accuracy: forecast vs actual at month-end (target ±5% by V2.2) |
| Single-view → Multi-lens | Avg distinct saved views per power user per week (target 3+ for Priya) |
| Dashboard → System | % of accounts with ≥ 1 active digest or alert (target 60% multi-property) |

### Counter-metrics

| Metric | Why we track it |
|--------|----------------|
| Support tickets containing "why is X showing" | Catch unclear UI early |
| False-positive risk-indicator rate | Risk pills wrongly flagging stable tenants kills trust fast |
| Email digest unsubscribe rate | Digest fatigue is a real risk |
| Push notification dismiss-without-tap rate | Threshold mis-calibration |

### Instrumentation Requirements

Every interaction emits an event with payload `{user_id, property_scope, view_state, action, timestamp, screen_position}`. New event types needed:
- `analytics_screen_opened`
- `analytics_drill_down` (with source element)
- `analytics_period_changed`
- `analytics_view_saved` / `view_loaded`
- `analytics_export` (with format)
- `analytics_digest_email_opened` / `clicked`
- `analytics_alert_triggered` / `dismissed` / `acted_on`

---

## 9. The Operator Promise Threading Through V1 → V2

V1's promise was: **"60-second answer to who to call today."**

V2 layers on three additional promises:

- **V2.0**: "60-second answer to **why this month is different**."
- **V2.1**: "60-second answer to **what month-end will look like**."
- **V2.2**: "60-second answer to **what RentOk noticed before I did**."

Each promise is short, falsifiable, and measurable.

---

## 10. Open Questions for PM Decision

1. ~~**V1 alignment with homescreen 07a (Section 1 above):** confirm the simpler current-state model. Update 07a.~~ **RESOLVED 2026-05-01.** PM confirmed simpler current-state model. Both DA-01 v3.1 and 07a updated.

2. **Forecast accuracy target.** I've proposed ±5% by V2.2. This is aggressive for a heuristic. ±10% in V2.1 tightening to ±5% by V2.2? Or ±5% from V2.1 launch?

3. **Risk indicator visibility.** Per-tenant risk pill on the worklist (P2-#18). Recommend: indicator visible only to operators with collections role, never on tenant-facing surfaces.

4. **Cohort dimensions in V2.1 (P1-#12).** Which 3–5 dimensions launch first? Recommendation: tenant tier, agreement type, booking source.

5. **Frozen-period semantics (P1-#16).** Manual operator-driven or automatic 5th-of-next-month auto-close? Recommend manual for V2.1, optional auto-close as V2.2.

6. **Email digest opt-in vs opt-out.** Auto-enable for accounts >₹5L overdue, or pure opt-in? Recommend auto-enable for first 30 days, then opt-out.

7. **Multi-property comparison view scope (P0-#7).** Top-level (3 KPIs side-by-side) for V2.0; full (every metric) for V2.2.

8. **Trend window length (P0-#4).** 12-month default with toggle to 3/6/12.

9. **What we explicitly DON'T build in V2** — confirm:
   - No RentOk AI integration (waits for RentOk AI maturity)
   - No two-way WhatsApp loop (different domain)
   - No collaboration/assignment features (workflow, not analytics)
   - No ML predictions (statistical only)
   - No cohort benchmarking (privacy review needed first)

---

## 11. Why I'm Now Satisfied (Conditional)

If V1 ships per v3.1 (now locked) and V2 follows this roadmap, this becomes a **best-in-class analytics product** for SMB property management — competitive with anything Stanza, NoBroker, or Zolo could build, and significantly ahead of Tally + Excel for this use case.

The codebase has the substrate (WhatsApp infra, scheduled cron, AI plumbing for later, report generation, activity logs, multi-property grouping). What we're missing is the **analytics layer specifically** — and that's exactly what this roadmap fills.

V3+ becomes the AI layer (RentOk AI deep integration), the ML layer (real default prediction, cohort benchmarking), and the cross-product layer (Dues + Collections + Refunds unified intelligence). That's a different conversation, and rightly so — we earn the right to build V3 by shipping V2 well.

---

## Appendix A — Codebase Findings That Shaped This Roadmap

- **`getFinancialsV2:1879-1893`** — single query computes both today-mode (4 buckets) and range-mode (3 buckets) urgency. V2 trend sparklines need a similar pattern with 12 monthly snapshots.
- **`getFinancialsV2:1881`** — Total Dues hero is mode-independent. Confirms V1 hero stays live always.
- **`buildBaseQuery()` shared between homescreen and list** — V2 forecast/risk should also use this base.
- **FIN-135** — defaulter classification is binary at 60 days. 5-bucket aging is genuinely net-new SQL.
- **AI-018 / AI-019** — dues NL search built on tool registry (`AI-001`) with execution planning (`AI-002`). When V3 RentOk AI lands, analytics insight tools register the same way.
- **`SemanticParsingService` filter codes 1102-1114** — when V2 adds new buckets, these codes should be extended so AI dues search can target them too.
- **`cloudTaskReportScheduler`** — V2 scheduled digest leverages this directly.
- **`emailDigestService` (complaints)** — pattern reusable for dues digest.
- **`sendWhatsapp7218()`** — actual WhatsApp send function. V2 push triggers wrap this with threshold logic.
- **`addActivityLog()`** in evictionService — pattern for logging dues actions.
- **REV-014 cron-scheduled campaign dispatch** — proven pattern for time-based dispatch.

## Appendix B — V2 Glossary

- **Aging window**: a bucket of days-overdue (e.g., 1–7, 8–15)
- **Cohort**: a subgroup of bills/tenants defined by an attribute (tier, agreement type, booking source)
- **Forecast**: extrapolated metric value at a future date based on historical pace
- **At-risk amount**: ₹ value of bills with above-threshold likelihood of crossing aging boundary
- **Saved view**: a stored combination of filters + period + property scope, named by user
- **Frozen period**: a past period explicitly marked "closed" — its summary numbers stop updating
- **Trend sparkline**: a compact 12-month line chart showing a metric's history
- **Anomaly marker**: a visual indicator on a trend chart for the first time a metric crossed a threshold
- **Discipline indicator**: % collected / % charged for a metric (Late Fee Section)
