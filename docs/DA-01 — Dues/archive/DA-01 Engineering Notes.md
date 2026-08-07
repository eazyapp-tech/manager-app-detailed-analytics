---
title: DA-01 Engineering Notes & Open Questions
date: '2026-05-01'
tags:
  - rentok
  - prd
  - engineering
  - open-questions
  - homescreen
  - financial-insights
  - detailed-analytics
  - dues
aliases:
  - DA-01 Engineering Notes
  - DA-01 Q&A
  - DA-01 Eng Gaps
status: SUPERSEDED — historical reference only
companion_to: DA-01 Build Sheet
superseded_by: DA-01 Build Sheet (V2) Hard Blockers section + Pre-Build Decisions Pending
---
> [!WARNING] SUPERSEDED — May 1 vintage
> Open questions and engineering gaps in this doc have been migrated to [[DA-01 Build Sheet]] V2 (May 8) under Hard Blockers + Pre-Build Decisions Pending sections, with full code citations and Status tags.
> Do NOT use this doc as the active engineering Q&A queue.

> [!INFO] Source of Truth (May 1 vintage)
> Local: `/Users/eazypg/RentOk Manager Homescreen/spec/DA-01-engineering-notes.md`
> **Active companion:** [[DA-01 Build Sheet]] (V2)

# DA-01 — Engineering Notes & Open Questions

This doc captures the questions a backend dev would ask before starting a ticket from the build sheet. Each gap has my PM recommendation. Some need dev confirmation (marked "Needs eng input"); most are PM calls already made.

**Status legend:**
- ✅ **Confirmed** — PM call, no further input needed
- ⚙️ **Needs eng input** — backend dev to verify against actual schema/code
- 🔄 **In discussion** — open for PM/eng debate before V1 lock

---

## 1. Timezone for "today" semantics ✅

**Question:** Every date predicate (`due_date < today`, `today − due_date`, `endOfThisCalendarWeekSunday`) assumes a "today" that isn't defined. Which timezone?

**PM recommendation:** Use **property timezone, midnight-to-midnight**. The `property` table has a timezone field — use it. Almost all RentOk properties are IST-based, but property-timezone-driven is the only correct approach.

**For multi-property selection (Priya):** When properties span timezones (rare), use the first selected property's timezone. Document this edge case.

**Why:** Operators experience "today" as their property's day. A bill due 11:30 PM IST should be "due today" until midnight IST, not based on server UTC.

---

## 2. Division-by-zero / null handling ✅

**Question:** Three places in the build sheet have division: MoM chip, Collection Efficiency, Property Breakdown share %. What if denominators are zero?

**PM recommendation:**

| Surface | Behavior when denominator = 0 |
|---------|-------------------------------|
| MoM chip | **Hide the chip entirely** (no "—", no "0%") |
| Collection Efficiency | Show **"—"** with tooltip "No bills due in this period" |
| Property Breakdown share % | **Hide the percentage** but keep the row (just amount) |

**Why:** Operators read "0%" as bad performance. "—" is correctly interpreted as N/A. Hiding the chip is cleaner than showing a placeholder.

---

## 3. Currency precision and equality ✅

**Question:** Reconciliation invariants say "strict equality." Stored in paise (int) or rupees (decimal/float)? How do we handle rounding from partial payments?

**PM recommendation:** **Store and compute in paise as integer.** Display in rupees with INR formatting (₹16L, ₹4,000, K/L/Cr suffixes). All reconciliation comparisons are exact integer equality. **No floating-point arithmetic anywhere in the sum chain.**

For partial-paid display ("₹4,000 of ₹10,000"), the underlying values are still integer paise; format at the display layer only.

**Verification needed (⚙️):** Confirm with backend that `i.amount` is currently stored as integer paise (or decimal — if decimal, file a separate ticket to migrate before V1 ships, OR codify "tolerance ±1 paise on cross-aggregate checks" as the QA rule).

---

## 4. `tenant.status` full enum ✅

**Question:** Base query filters `status IN (0,1,2)`. What are all the values, and which do we include?

**PM recommendation:** Per the tenant lifecycle wiki + RPT-026:

| Value | Meaning | Include in DA-01 base universe? |
|-------|---------|--------------------------------|
| 0 | Old (moved out, retained for dues recovery) | Yes |
| 1 | Active (currently checked-in) | Yes |
| 2 | Booking (confirmed but not yet checked-in) | Yes |
| 3 | Evicted-pending (in eviction workflow) | **No** — separate workflow surface |
| -1 / soft-delete flag | Deleted | **No** — never shown |

DA-01 base query: `tenant.status IN (0,1,2) AND is_active = 1`.

**Why:** Matches `buildBaseQuery` in helpers.ts. Eviction-pending bills are a different operations surface.

---

## 5. Due-type LIKE matching for By Category ✅

**Question:** Section 8 says `due_type = X` for X in {Rent, Electricity, ...}. But code (`applyDueTypes` in helpers.ts) uses `LIKE 'X%'` to catch variants ("Electricity Bill", "Mess (Veg)"). Which is correct?

**PM recommendation:** Use **`LIKE` matching with these patterns** (mirroring existing `applyFilterCodes` switch):

| Spec category | Match pattern |
|---------------|---------------|
| Rent | `due_type = 'Rent'` (exact — single canonical name) |
| Electricity | `due_type LIKE 'Electricity%'` |
| Food | `due_type LIKE 'Mess%' OR due_type LIKE 'Food%'` |
| Security Deposit | `due_type IN ('Security Deposit', 'Caution Money')` |
| Maintenance | `due_type LIKE 'Maintenance%'` |
| Late Fine (Phase 2 section) | `due_type IN ('Automatic Late Fine', 'Manual Late Fine')` |
| Others | NOT IN any of above |

**Why:** This is what already works in the dues list filter. Stay consistent.

---

## 6. `i.amount` semantic when status=1 ⚙️

**Question:** Section 7 Performance "Invoices Due" sums `i.amount` over `status IN (0,1)` — both unpaid and paid bills. If `i.amount` is 0 when status=1 (paid), this query is wrong. What does `i.amount` represent across statuses?

**PM recommendation (pending eng confirmation):** Based on `getFinancialsV2` using `SUM(i.amount)` consistently and the base query restricting to `status=0`, my read is that **`i.amount` is the invoice's gross/original amount and stays the same regardless of status** — `status` separately tells you whether it's collected.

**To verify (eng):** Run on staging — `SELECT amount, status FROM invoices WHERE id = <a-paid-invoice-id>`. If amount > 0 for a paid invoice, recommendation stands. If amount = 0 for paid, we need a separate field for original amount (or change the metric formula).

**If recommendation breaks:** Performance metric becomes: `SUM(i.original_amount) WHERE status IN (0,1) AND due_date in period` — file a separate ticket.

---

## 7. Aging off-by-one (day 0 and chart vs table) ✅

**Question:** Two issues:
- Bills due today (day 0) — not in any aging bucket. Intentional?
- §9 aging table has 5 buckets (1–7, 8–15, 16–30, 31–60, 60+). §12 Defaulter Analysis Chart shows 4 bars (0–7, 8–15, 16–30, 31+). Inconsistent.

**PM recommendation:**

**Day 0 stays out of aging.** A bill due today isn't overdue. It belongs in the urgency bar's "Due Today" bucket, not aging.

**Align the chart to 5 bars matching the aging table:** 1–7, 8–15, 16–30, 31–60, 60+. Update `DA-01-spec.md` §8 chart spec and Figma when next iterating. The "84/52/88/88" example data is a placeholder — chart must show 5 values, not 4.

**Why:** Same data should never be visualized two different ways. Operators get confused.

---

## 8. Repeat defaulter detection — V1 or V2? ✅

**Question:** §10 formula assumes an `overdue_history` table. Either we build a heavy subquery against `invoices` per tenant, or we maintain a materialized table. Which?

**PM recommendation:** **Defer to V2.0 P0** (already in roadmap). V1 ships **without** the Repeat badge. Update [[DA-01 Spec]] and [[DA-01 Build Sheet]] to mark Repeat badge as "Phase 2 — V2.0 P0-#5" rather than V1.

**Why:** Detection is meaningfully complex (3-of-6-month recurrence) and not on the V1 critical path. The aging buckets already help operators identify chronic problems. The badge is a polish layer.

**Net change:** Remove "Repeat badge" mentions from V1 ship list. Keep the design slot for V2.0.

---

## 9. ADDED_BY_MAP constants ⚙️

**Question:** §11 Added By says `added_by = user_X` and refers to a "RentOk system row." What's the actual numeric value for each, including system/auto-generated?

**PM recommendation:** PM cannot know these. **Action for backend:** expose `ADDED_BY_MAP` (from `src/v1/constants/filterCodes.ts`) in the API response payload. Frontend should never hardcode IDs.

API response sketch:
```json
{
  "added_by_options": [
    { "id": 0, "label": "Owner", "type": "user" },
    { "id": 1, "label": "Self", "type": "tenant" },
    { "id": 2, "label": "Partner", "type": "partner" },
    { "id": null, "label": "RentOk", "type": "system" }
  ]
}
```

(Actual values to be confirmed by dev. The principle: backend is source of truth; frontend is presenter.)

---

## 10. Defaulter Analysis Chart: 4 vs 5 bars ✅

Resolved in §7 above. **Use 5 bars matching aging table.**

---

## 11. Multi-property response shape ✅

**Question:** Worklist groups by property in multi-property mode. Is the response a flat list with `property_id` per row, or pre-grouped?

**PM recommendation:** **Flat list with `property_id` and `property_name` per row, plus a separate `property_summary` block for section headers.**

API response sketch:
```json
{
  "rows": [
    { "invoice_id": ..., "property_id": "PG123", "property_name": "Sunshine PG", "tenant_name": "...", "amount": 1200000, ... },
    ...
  ],
  "property_summary": [
    { "property_id": "PG123", "property_name": "Sunshine PG", "count": 12, "total": 35000000 },
    { "property_id": "PG456", "property_name": "Green Hostel", "count": 8, "total": 20000000 }
  ],
  "grand_total": { "count": 20, "total": 55000000 }
}
```

**Why:**
- Frontend handles grouping → can change UX without backend churn
- Section headers come from `property_summary` directly (no client-side aggregation)
- Pagination is simpler on flat rows
- Filter-chip removal is trivial on frontend

---

## 12. Setting-Up state — formal definitions ✅

**Question:** `bed_fill_ratio < 0.3` and `billing_history < 30d` need explicit definitions.

**PM recommendation:**

| Criterion | Formal definition |
|-----------|-------------------|
| `account_age` | `(today − property.created_at)` in days |
| `total_invoices` | `COUNT(*) FROM invoices WHERE property = X` (any status, any time, including auto-generated) |
| `bed_fill_ratio` | `active_tenants / total_beds_configured`, where `active_tenants = COUNT(*) WHERE tenant.status = 1 AND property = X` and `total_beds_configured = SUM(rooms.capacity) FROM rooms WHERE property = X` |
| `billing_history` | `(today − MIN(invoices.created_at))` in days, including auto-generated invoices |

Setting-Up state TRUE if **ANY** of:
- `account_age < 30`
- `total_invoices < 10`
- `bed_fill_ratio < 0.3`
- `billing_history < 30`

Permanent dismiss when **ALL** of these are FALSE AND user has tapped through the flow once. Per-session dismiss does not require permanent criteria.

---

## 13. Auth gating per action ✅

**Question:** EC-10 says Meena hides actions she can't perform. Which permissions gate which actions?

**PM recommendation:** Add the following permissions reference. All gated surfaces **hide** (not gray out) when permission is missing.

| Action / surface | Required permission | Source |
|------------------|---------------------|--------|
| Viewing the Dues section at all | `viewInvoices` | `checkAuth(req, pg_number, 'viewInvoices')` |
| Worklist scoped to user's added bills only | `view_invoices_of_self_added_tenants` (when set) | `checkAuthInDb` |
| Send Reminder action | `sendReminder` (verify name with backend) | — |
| Record Payment action | `recordPayments` | — |
| Adjust from Deposit action | `editDuesAndCollection` | — |
| Add Dues button (empty state) | `addDues` | — |
| Bulk Reminder | `sendBulkReminder` (Phase 2) | — |
| Export to Excel | `exportInvoices` (verify) | — |

**To verify (⚙️):** Confirm exact permission key strings with backend. The patterns are clear; the strings need a 5-minute audit.

---

## How to use this doc

1. **Backend dev opens a ticket from [[DA-01 Build Sheet]].**
2. If a row is unclear, dev checks here first.
3. If still unclear, dev pings PM. Answer gets added here, status flips to ✅.
4. New gaps discovered during build → append a new section here, don't bury in Slack.
5. Once V1 ships, this doc gets archived (or merged into [[DA-01 Engineering]] as ground truth).

**Open items requiring eng input before V1 ships:**
- ⚙️ Gap #6 — `i.amount` semantic across statuses
- ⚙️ Gap #9 — ADDED_BY_MAP constants
- ⚙️ Gap #13 — exact permission key strings (final pass)

The other 10 gaps are PM-confirmed; engineering can build to those answers without further input.

---

## Pointers

- **Tickets:** [[DA-01 Build Sheet]]
- **Why behind decisions:** [[DA-01 Spec]]
- **Invariants & API contracts:** [[DA-01 Engineering]]
- **What's already built:** [[DA-01 Codebase Feasibility]]
