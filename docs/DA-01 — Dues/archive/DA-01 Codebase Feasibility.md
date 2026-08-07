---
title: DA-01 Codebase Feasibility Audit
date: '2026-05-01'
tags:
  - rentok
  - prd
  - codebase
  - feasibility
  - homescreen
  - financial-insights
  - detailed-analytics
  - dues
aliases:
  - DA-01 Codebase Feasibility
  - DA-01 Codebase Audit
  - DA-01 Codebase
status: SUPERSEDED — May 1 point-in-time audit; codebase has changed
audit_date: '2026-05-01'
companion_to: DA-01 Spec
superseded_by: _Ground Truth Field Map (May 8, post-master-merge audit)
---
> [!WARNING] SUPERSEDED — May 1 codebase snapshot
> The codebase has changed since this audit (master merge May 8 brought 30+ commits including HeaderValidator changes, payment_mode enum corrections, etc.).
> **Active codebase reference:** [[_Ground Truth Field Map]] (verified 2026-05-08 post-master).
> Do NOT use this doc for current implementation feasibility — it may show outdated line numbers, missing endpoints, or pre-merge code shapes.
> [!INFO] Source of Truth
> Local: `/Users/eazypg/RentOk Manager Homescreen/spec/DA-01-codebase.md`
> **Companion to:** [[DA-01 Spec]] (the user-facing spec)
> **Engineering reference:** [[DA-01 Engineering]]
> **For:** Engineering planning sequencing, PM tracking build readiness

# DA-01 — Codebase Feasibility Audit

This is a **point-in-time** audit of what exists in the codebase versus what's needed to ship DA-01 V1. As engineering builds, this doc will go stale — refresh before any V1 sprint planning.

**Reviewed:** 2026-04-30, refreshed 2026-05-01.

**Sources read:**
- `src/v1/list_screens/dues/helpers.ts` (DuesListHelper)
- `src/v1/list_screens/dues/service.ts` (DuesListService)
- `src/v1/homepage/service.ts` — `getFinancialsV2()`
- `docs/wiki/01-financial-operations/rules/FIN-135-dues-defaulters-tiers.md`
- `docs/wiki/14-ai-and-parsing/rules/AI-018, AI-019` (semantic dues parser, dues tool registration)
- `docs/wiki/_cross-domain-map.md`

---

## What Already Works

| Spec feature | Code reference | Status |
|--------------|----------------|--------|
| Base query: unpaid, active invoices for property | `buildBaseQuery()` | EXISTS |
| Today-mode urgency (4 buckets) | `getFinancialsV2` lines 1883-1886 | EXISTS — but `due_next_7` uses rolling 7d, needs alignment to calendar week (Mon–Sun) |
| Range-mode urgency (3 buckets: CF / DiP / DAP) | `getFinancialsV2` lines 1888-1890 | EXISTS |
| Hero always live (single SUM regardless of mode) | `getFinancialsV2` line 1881 | EXISTS |
| Filter by `due_date` range | `applyDateRange()` | EXISTS |
| Filter by `due_type` (category) | `applyDueTypes()` | EXISTS |
| Filter by tenant `status` (Population) | `applyTenantTypes()` | EXISTS |
| Filter by bill creator (Added By) | `applyAddedBy()` | EXISTS |
| Property grouping in worklist | `buildBaseQuery(strings, 'property')` | EXISTS — needs response shape that groups rows |
| Multi-property scope | `pg_number_filter` param | EXISTS |
| Old tenant deposit context (deposit value) | `t.security_deposit` selected in base query | EXISTS — shortfall arithmetic to be added in row formatter |
| Tenant search (name/phone) | `applySearch()` | EXISTS |
| Sort by amount/room/due_date | `applyOrderBy()` | EXISTS |
| Auth checks (view + self-added-only) | `checkAuth`, `checkAuthInDb` | EXISTS |
| AI-powered dues NL search (text → filter codes) | `SemanticParsingService`, `DuesParsingService`, AI-018, AI-019 | EXISTS — for NL search; not analytics insights |
| Filter codes 1102-1114 (NL-recognized) | `DuesFilterCode` | EXISTS |

---

## What Needs to Be Built

| Spec feature | Gap | Effort |
|--------------|-----|--------|
| Calendar-week boundary for "Due this week" | Code uses rolling 7 days (`today+1..today+7`); align to Mon–Sun | LOW — date math change in `getFinancialsV2` |
| Aging bucket filter codes (1–7d, 8–15d, 16–30d, 31–60d, 60+d) | Current `applyDefaulters` is 60+ binary only (FIN-135) | MEDIUM — new filter codes + SQL CASE on `DATEDIFF(today, due_date)` |
| MoM comparison chip | No comparison logic anywhere | MEDIUM — needs same-day-last-month query + delta + format |
| Collection Efficiency metric | Cross-domain join (payments × invoices for period) | MEDIUM — query lives in collections domain or a new joined query |
| Amount range filter on worklist | Add `min_amount` / `max_amount` params | LOW — small SQL clause |
| Real pagination (20/page, infinite scroll) | Currently `limit=5000`, `offset=0` hardcoded (`service.ts:27-28`) | LOW — surface params already exist |
| Filter chip rendering on worklist | Frontend feature; backend already exposes filter state | LOW |
| Property-grouped worklist response shape | Underlying query supports grouping; needs response reshape | LOW |
| Repeat defaulter detection (3 of last 6 months) | No recurrence detection | MEDIUM — Phase 2 candidate |
| Setting-Up state computation (frontend) | Computed by frontend on load — no backend support needed | LOW (frontend only) |

---

## What Was Considered and Killed

| Once-considered feature | Decision |
|-------------------------|----------|
| Historical balance reconstruction (date-rewinding hero) | NOT BUILT. Operators don't ask "what was outstanding at 11:59 PM on March 31." Range-mode urgency answers period-cohort questions; balance-as-of-date is an Excel pivot job for accountants. The 07a1 solution doc proposed this using `paid_date` reconstruction; the homescreen direction went live-only and the build was not pursued. |
| `date_field=created_at` query param | NOT NEEDED. Performance metric semantic is `due_date IN period`, not `created_at IN period`. The list API's existing `due_date` filter is the right tool throughout. |
| Late Fee dedicated section | PHASE 2. Figma (newer than Lark) does not include it. May be promoted to V2.0 P0 — see [[DA-01 V2 Roadmap]]. |
| Snapshot Reports (one-click "as of date" export) | PHASE 2 if accountants ask. Excel pivot of Invoice + Payment exports already serves the use case. |
| "My Tenants" view for Meena (collections coordinator) | NOT BUILT. There is no tenant-to-team-member assignment concept in the codebase. PM confirmed it's not practically needed. Saved views (V2.1) cover the "self-curated subset" need. |

---

## Staleness Flags vs Lark PRD v1.1

The Lark PRD v1.1 was written before the homescreen. Where it contradicts the code or this spec:

| Lark statement | Reality | Resolved by |
|----------------|---------|-------------|
| 3 forward buckets: "Due This Week / Rest of Mo. / Future Due" | Code has 4 today-mode buckets (`already_overdue`, `due_today`, `due_next_7`, `due_later`) | Spec documents 4 forward buckets, with calendar-week alignment as the engineering change |
| 2 historical buckets: "Carried Forward / Became Due in Period" | Code has 3 range-mode buckets (`carried_forward`, `due_in_period`, `due_after_period`) | Spec documents 3 range-mode buckets |
| Worklist columns include `outstanding`, `check_in_status`, `created_by` | Real columns are `i.amount`, `t.status`, `i.added_by` | Engineering Data Sources table corrected |
| Deposit source: `security_deposits` table | Code reads `t.security_deposit` directly | Corrected |
| Per-invoice rows | Code groups by tenant currently (`GROUP BY i.payer`) | Engineering note — some drill-downs may need invoice-level grouping change |
| Late Fee Section in Phase 1 | Figma omits it | Moved to Phase 2 with rationale |
| "Operational mode + Historical mode" framing for past periods | Code only has operational + range; no historical reconstruction | Spec uses "today / range" framing matching code |

---

## Cross-Domain Dependencies (V1)

| What V1 reads from | Domain |
|--------------------|--------|
| Invoice records | Billing & Invoicing (Domain 1) |
| Payment history | Payment & Settlement (Domain 1 — sub-area) |
| Tenant status, name, room, deposit | Tenant Lifecycle (Domain 2) |
| Property scope and grouping | Property & Rooms (Domain 3) |
| Reminder history | WhatsApp & Communications (Domain 7) |
| Auth checks | Platform & Auth (Domain 13) |

V2 expands this — see [[DA-01 V2 Roadmap]] §7 Cross-Domain Dependencies for the full V2 dependency matrix.

---

## Pointers

- **What does the screen DO?** → [[DA-01 Spec]]
- **Engineering reference (invariants, API, data sources):** → [[DA-01 Engineering]]
- **V2 roadmap:** → [[DA-01 V2 Roadmap]]
- **Original v3.1 spec (untouched):** → [[DA-01 Dues Detailed Analytics]]
