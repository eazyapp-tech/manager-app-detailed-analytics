---
title: DA-01 Engineering Reference
date: '2026-05-01'
tags:
  - rentok
  - prd
  - engineering
  - homescreen
  - financial-insights
  - detailed-analytics
  - dues
aliases:
  - DA-01 Engineering
  - DA-01 Engineering Reference
status: SUPERSEDED — historical reference only
companion_to: DA-01 Spec
superseded_by: DA-01 Build Sheet (V2) + _Ground Truth Field Map
---
> [!WARNING] SUPERSEDED — May 1 vintage
> This doc predates the V2 Build Sheet (May 8) and the Ground Truth Field Map. Kept for historical reference only.
> **Active engineering reference:** [[DA-01 Build Sheet]] + [[_Ground Truth Field Map]]
> Do NOT use for current implementation.

> [!INFO] Source of Truth (May 1 vintage)
> Local: `/Users/eazypg/RentOk Manager Homescreen/spec/DA-01-engineering.md`
> **Companion to:** [[DA-01 Spec]] (the user-facing spec)
> **Codebase audit:** [[DA-01 Codebase Feasibility]]
> **For:** Engineering and QA

# DA-01 — Engineering Reference

This doc covers reconciliation invariants, list API filter requirements, data sources, architecture notes, and Figma node IDs. For *what the screen does* (components, drill-downs, edge cases, microcopy), read [[DA-01 Spec]].

---

## Reconciliation Invariants

These must always hold. If any breaks, it's a bug — not a rounding issue.

| Invariant | Formula |
|-----------|---------|
| Hero is always live | Total Dues = `SUM(i.amount) WHERE status=0 AND amount>=1 AND is_active=1 AND tenant.status IN (0,1,2)` — independent of date filter |
| Today-mode urgency sums to total | Already Due + Due Today + Due This Week + Due Later = Total Dues |
| Range-mode urgency sums to total | Carried Forward + Due in Period + Due After Period = Total Dues |
| Population sums to total | Active + Booking + Old Tenants = Total Dues |
| Property breakdown sums to total | Sum of all property rows = Total Dues (multi-property mode) |
| Category sums to total | Rent + Electricity + Food + Deposit + Maintenance + Others = Total Dues |
| Performance: Invoices Due ≥ Current Dues | Invoices Due − Current Dues = Collected for period (cannot be negative) |
| Cross-screen consistency | Homescreen Dues tile = Detailed Dues hero, when same property scope is applied |
| Drill-down count matches | List header count must equal worklist row count exactly |
| MoM chip is independent of date filter | Always (right-now total) vs (same-date-last-month total) |
| Setting-Up state preserves invariants | All invariants above hold even when the Setting-Up banner is shown |

---

## List API — Filter Requirements

The existing `getDuesList` API supports most of what's needed. Use the `start_date`/`end_date` path with optional `due_types` / `added_by` / `tenant_types` modifiers — never `filter_codes` (mutually exclusive with the others; see architecture notes).

| Filter | Required for | Status |
|--------|-------------|--------|
| `due_date` start/end range | All drill-downs (today buckets, range buckets, Performance, Added By) | **EXISTS** (`applyDateRange` filters on `i.due_date`) |
| `outstanding > 0` (active unpaid bills) | Default base universe | **EXISTS** (`buildBaseQuery`) |
| Tenant `status` (1=active, 2=booking, 0=old) | Population drill-downs | **EXISTS** (`applyTenantTypes`) |
| `added_by` | Added By drill-downs | **EXISTS** (`applyAddedBy`) |
| Computed aging windows (1–7d, 8–15d, 16–30d, 31–60d, 60+d) | Aging bucket drill-downs | **NEW** — current `applyDefaulters` is 60+ only. Needs `DATEDIFF(today, due_date)` ranges as filter codes. |
| `due_type` (category) | Category drill-downs | **EXISTS** (`applyDueTypes`) |
| `property_id` / `pg_number_filter` | Multi-property scope, property filter | **EXISTS** |
| Property grouping in result rows | Multi-property worklist grouped output | **EXISTS** (`buildBaseQuery(strings, 'property')`) — needs response shape that groups rows |
| Amount range (min/max) | Worklist amount range filter | **NEW** — small SQL addition |
| `paid_amount` per row (for partial-paid display) | Worklist row showing "₹X (of ₹Y)" | **VERIFY** |
| Repeat defaulter flag | Repeat badge on Top Defaulters | **PHASE 2** — recurrence detection across last 6 months is a separate concern |

---

## Data Sources

| What's shown | Source | Key columns |
|-------------|--------|-------------|
| Invoice outstanding | `invoices` (`i`) | `amount` (= outstanding when `status=0`), `due_date`, `status`, `is_active`, `due_type`, `added_by`, `created_at`, `partner_name`, `payer` |
| Tenant status | `tenants` (`t`) | `status` (0=old, 1=active, 2=booking), `name`, `phone`, `room`, `security_deposit`, `is_short_term` |
| Property | `property` (`prop`) | `id`, `pg_id`, `pg_number`, `pg_name` |
| Bill creator | `invoices.added_by` (numeric ID), `invoices.partner_name` (string for partner-created) | Mapped via `ADDED_BY_MAP` |
| Payment history | `payments` | `amount`, `payment_date`, `payment_mode`, `invoice_id` (via `payments_invoices` join) |
| Reminder history | `reminders` (or equivalent — verify location) | `sent_at`, `tenant_id`, `type` |

---

## Architecture Notes

1. **Hero is mode-independent.** The single query for `getFinancialsV2` already returns both today-mode (4 buckets) and range-mode (3 buckets) urgency in one pass — frontend picks which to display. No re-query when the operator switches modes.

2. **`buildBaseQuery()` is shared** between homescreen and list — this is the foundation of the count-consistency invariant (Rule 2). Do not fork it.

3. **Worklist groups by tenant in current code** (`GROUP BY i.payer`). Drill-downs may need either:
   - **Tenant-level rows** (matches current behavior — used by Top Defaulters, Population)
   - **Invoice-level rows** (specified by spec for most worklist drill-downs)
   Some drill-downs may need a parameter on the API to switch grouping.

4. **Pagination is currently disabled** — `limit=5000`, `offset=0` (`service.ts:27-28`). Phase 1 must implement real pagination (page size 20, infinite scroll).

5. **`tenant_types` filter is independent of `filter_codes`** — Population drill-downs can layer on top of any other filter cleanly.

6. **`filter_codes` and `start_date`/`end_date`/`due_types`/`added_by` are mutually exclusive** in the current API (`service.ts:60-75`). When `filter_codes` is present, the others are ignored. All drill-downs in DA-01 V1 use the `start_date`/`end_date` path — never `filter_codes`. The 14 widget tile filter codes remain available for the homescreen but are not used by the detailed analytics drill-downs.

7. **`filter_meta.mode`** in homescreen response is `'today'` or `'range'` (see `service.ts:2224`). Frontend uses this to pick which urgency-bar buckets to display.

8. **Setting-Up state is computed once on dashboard load** by frontend, based on account-age + invoice-count + bed-fill criteria. No backend flag.

---

## Figma Component Node IDs

| Component                   | Node         |
| --------------------------- | ------------ |
| Full Dues Section           | `7837:99662` |
| Hero + MoM Chip             | `7837:99673` |
| Urgency Stacked Bar         | `7837:99691` |
| Population Rows             | `7837:99717` |
| Action Card                 | `7837:99741` |
| Advanced Insights Container | `7837:99747` |
| Defaulter Analysis Chart    | `7837:99812` |

File: `KgBQXiT7r7oGrcqZHCWxyU` → Financials → Dues container (`7837:99662`).

---

## QA Checklist

Bare minimum verification per release:

- [ ] All 11 reconciliation invariants pass on a property with mixed tenant statuses (active + booking + old) and at least 5 categories of dues.
- [ ] Drill-down count equals worklist row count for every entry point in the spec drill-down matrix.
- [ ] Today mode → Last Month → Custom Range → This FY: hero number stays exactly the same across all four; only urgency bar reslices.
- [ ] Multi-property aggregated worklist: rows grouped by property; sum of property group counts equals header count.
- [ ] Setting-Up banner appears when ANY of the four criteria are met; disappears when ALL are unmet.
- [ ] Permission gating (Meena scenario): Adjust from Deposit hidden if user lacks `editDuesAndCollection`. Surface hidden, not grayed.
- [ ] Range-mode period with zero data: hero still shows live total; urgency placeholder shows "No bills due in this period."
- [ ] MoM chip: hidden in first month of operation; computed against same-day-last-month otherwise.

---

## Pointers

- **What does the screen DO?** → [[DA-01 Spec]]
- **What's already in code vs net-new?** → [[DA-01 Codebase Feasibility]]
- **What's the V2 plan?** → [[DA-01 V2 Roadmap]]
