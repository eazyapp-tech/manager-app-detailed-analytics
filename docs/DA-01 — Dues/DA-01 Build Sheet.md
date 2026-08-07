---
title: DA-01 Dues — Build Sheet
date: 2026-06-06
tags: [rentok, dues, build-sheet, engineering-spec, detailed-analytics]
status: v4.0
owner: Sanchay
build_leads: Jatin · Parishi · Vivek
---

> [!INFO] What this is
> Engineering handoff for DA-01 Dues.
>
> Build from the meanings in this sheet. The product rule: every amount shown on the Dues analytics screen must match the list or detail view opened from it. The real database fields live in Section 20 of the Ground-Truth Formula Map — build against those.

## 1. Build rules

1. **Base amount is the open invoice amount.** Only `invoices.status = 0` counts. Partial payments are already handled correctly: the manual payment flow reduces the open invoice's `amount`/`net_amount` in place and leaves it `status = 0` ([payment.ts:2048](</Users/eazypg/rentok-backend/src/controllers/payment.ts:2048>)), so summing `amount` gives the remaining owed amount. No split row is created and `parent_invoice_id`/`root_invoice_id` are never written.
2. **Old tenant dues count.** Moved-out tenants are `tenant.status = 0`. The current base filters `status IN (1, 2)` and hides them — that is the bug to fix.
3. **Future Dues and Projected Fixed Dues are different.** Future Dues are existing unpaid invoices with future due dates. Projected Fixed Dues are expected recurring invoices not yet created.
4. **Category and Added By use top rows + Others.** Do not hardcode the visible categories.
5. **Tenant Status follows the stage model.** Active, Notice Raised, Confirmed Eviction, Bookings, Old Tenants — keyed off `tenant.status` and the eviction tables.
6. **Every row needs drill parity — the list a row opens must total the same amount the row shows.**

## 2. Code anchors

| Area | Anchor | Note |
|---|---|---|
| Dues base query | [helpers.ts:52](</Users/eazypg/rentok-backend/src/v1/list_screens/dues/helpers.ts:52>) | `buildBaseQuery` |
| Unpaid filter | [helpers.ts:58](</Users/eazypg/rentok-backend/src/v1/list_screens/dues/helpers.ts:58>) | `status = 0 AND is_active = 1 AND amount >= 1` |
| Tenant filter excludes old tenants today | [helpers.ts:61](</Users/eazypg/rentok-backend/src/v1/list_screens/dues/helpers.ts:61>) | `t.status IN (1,2)` — change to `IN (0,1,2)` |
| Current-month widget filter includes the whole month | [helpers.ts:132](</Users/eazypg/rentok-backend/src/v1/list_screens/dues/helpers.ts:132>) | Do not reuse — DA-01 needs through-today |
| due_type category predicates | [helpers.ts:135](</Users/eazypg/rentok-backend/src/v1/list_screens/dues/helpers.ts:135>) | Real string values, incl. Deposit = Security Deposit + Caution Money |
| Date range helper | [helpers.ts:176](</Users/eazypg/rentok-backend/src/v1/list_screens/dues/helpers.ts:176>) | |
| Due type helper | [helpers.ts:185](</Users/eazypg/rentok-backend/src/v1/list_screens/dues/helpers.ts:185>) | |
| Added By helper | [helpers.ts:196](</Users/eazypg/rentok-backend/src/v1/list_screens/dues/helpers.ts:196>) | Groups on `i.added_by` (invoice creator) |
| Existing 60-day defaulter helper | [helpers.ts:295](</Users/eazypg/rentok-backend/src/v1/list_screens/dues/helpers.ts:295>) | Binary — replace with 1-7/8-14/15-21/22+ buckets |
| Invoice status enum | [invoices.ts:93](</Users/eazypg/rentok-backend/src/entities/invoices.ts:93>) | |
| Invoice creator enum (`added_by`) | [invoices.ts:74](</Users/eazypg/rentok-backend/src/entities/invoices.ts:74>) | No "system" value; auto-rent = `added_by = 1` |
| Tenant status enum | [tenant.ts:154](</Users/eazypg/rentok-backend/src/entities/tenant.ts:154>) | `0 = moved-out, 1 = active, 2 = booking` |
| Journey status predicates | [homepage service:2790](</Users/eazypg/rentok-backend/src/v1/homepage/service.ts:2790>) | **Tenant COUNT logic — reuse the status predicates, not the date-overlap counting** |
| Journey date-range variant | [homepage service:2867](</Users/eazypg/rentok-backend/src/v1/homepage/service.ts:2867>) | Omits `ted.status = 2` — DA-01 uses the status=2 version |
| Top-category + Others pattern | [homepage service:3342](</Users/eazypg/rentok-backend/src/v1/homepage/service.ts:3342>) | **This is the COMPLAINTS card — reuse the shape only, not the logic** |
| Others drill pattern | [homepage service:3469](</Users/eazypg/rentok-backend/src/v1/homepage/service.ts:3469>) | **Complaints card — shape reference only** |
| Filter codes | [filterCodes.ts:1](</Users/eazypg/rentok-backend/src/v1/constants/filterCodes.ts:1>) | DuesFilterCode + ADDED_BY_MAP |
| Dues list route | [routes.ts:8](</Users/eazypg/rentok-backend/src/v1/list_screens/dues/routes.ts:8>) | |
| Widget route | [routes.ts:9](</Users/eazypg/rentok-backend/src/v1/list_screens/dues/routes.ts:9>) | |
| List self-added restriction | [service.ts:95](</Users/eazypg/rentok-backend/src/v1/list_screens/dues/service.ts:95>) | Keys on `t.added_by_id` (tenant creator) |
| Widget self-added gap | [service.ts:132](</Users/eazypg/rentok-backend/src/v1/list_screens/dues/service.ts:132>) | Scope computed but never applied — launch blocker |
| Hardcoded list limit | [service.ts:27](</Users/eazypg/rentok-backend/src/v1/list_screens/dues/service.ts:27>) | `limit = 5000`, pagination ignored |

## 3. Build order

1. Shared unpaid-dues base (status 0, tenant status 0/1/2)
2. Duration helper (incl. new current-month-through-today range)
3. Category and Added By grouping helper
4. Top Summary
5. Category, Journey, Added By
6. Ageing
7. Property-wise and stay-duration views
8. Projected Fixed Dues
9. Drill parity and permissions
10. QA checks

## 4. Foundation tasks

| # | Task | Why | Acceptance check |
|---|---|---|---|
| 1 | Create one shared unpaid-dues base query for DA-01. | All sections start from the same unpaid-money universe. | All DA-01 sections use `status = 0`, `is_active = 1`, `amount >= 1`, selected properties. |
| 2 | Include old tenants: change base from `t.status IN (1,2)` to `IN (0,1,2)`. | `status 0 = moved-out (Old Tenant)`, `1 = active`, `2 = booking`. Old tenants still owe money and are hidden today. | A moved-out tenant (`status = 0`) with an unpaid bill appears in All Dues and Old Tenants. |
| 3 | Treat the open invoice `amount` as the remaining owed amount. | The manual partial-payment flow reduces `amount`/`net_amount` in place and keeps the invoice `status = 0`. No `amount − paid_amount` math is needed. | A part-paid bill shows its remaining balance in DA-01 totals and the drill list matches. |
| 4 | Apply the same permission scope to analytics and list drills. | The widget computes the self-added scope but never applies it ([service.ts:132](</Users/eazypg/rentok-backend/src/v1/list_screens/dues/service.ts:132>)) — restricted users see a wider total than their list. | Self-added-only users see matching totals and list rows across every aggregate. |
| 5 | Decide endpoint shape. | The old widget endpoint may not fit the full analytics response. | Team chooses dedicated analytics endpoint or safe extension before frontend wiring. |

## 5. Duration helper

| # | Task | Why | Acceptance check |
|---|---|---|---|
| 6 | Build duration ranges for All Time, This Month, Last Month, Current FY, Custom. | All deeper views need one shared time rule. | Same duration input produces same invoice pool across Category, Journey, Added By, Ageing, Property. |
| 7 | Keep Top Summary outside the duration helper. | Summary numbers have fixed meanings. | Changing duration does not change Top Summary. |
| 8 | Keep Projected Fixed Dues outside the duration helper. | It is always tomorrow through month-end. | Changing duration does not change projection. |
| 9 | Build a new current-month-through-today range. Do not reuse `CURRENT_MONTH`. | The existing filter ([helpers.ts:132](</Users/eazypg/rentok-backend/src/v1/list_screens/dues/helpers.ts:132>)) covers the whole month including future days. DA-01 current-month numbers end today. | Current Month Dues and current-month category cuts exclude tomorrow onward. |

## 6. Category tasks

| # | Task | Why | Acceptance check |
|---|---|---|---|
| 10 | Build due-category normalization from the real `due_type` strings. | `due_type` is free-text. Deposit = `Security Deposit` + `Caution Money` (not `'Deposit'`); Electricity/Food/Mess are prefix matches. | Rent, electricity, food/mess, deposit, late fine, and custom values group into readable names matching Section 20. |
| 11 | Build top 4 + Others helper for categories. | Visible categories must be based on real unpaid amount. | Top 4 + Others equals selected-duration unpaid dues. |
| 12 | Build category drill filters. | Row and list must match. | Named category opens that category. Others opens selected duration excluding visible top categories. |

## 7. Tenant Journey tasks

| # | Task | Why | Acceptance check |
|---|---|---|---|
| 13 | Build Active Tenants dues. | Owners need the live recovery base. | Row opens `status = 1`, no confirmed eviction, no active notice, with unpaid dues in selected duration. |
| 14 | Build Notice Raised dues (eviction-details `is_active = 1 AND status = 2`). | Notice raised is an early exit-risk lane; `status = 2` is canonical (matches the home card). | Row opens active tenants with a `status = 2` eviction-details row and no confirmed move-out date. |
| 15 | Build Confirmed Eviction dues (`date_of_eviction IS NOT NULL`). | Confirmed move-out date is a higher-risk recovery lane. | Row opens active tenants with a confirmed eviction date. |
| 16 | Build Bookings dues (`status = 2`). | Booking dues are a different follow-up path. | Row opens booking tenants with unpaid dues in selected duration. |
| 17 | Build Old Tenants dues (`status = 0`). | Old tenant recovery is harder and should not be hidden. | Row opens moved-out tenants with unpaid dues in selected duration. |
| 18 | Mark Notice Raised and Confirmed Eviction as risk slices. | They sit inside the active-tenant stage and must not be treated as a strict sum. | API or UI metadata prevents a false "all rows add up" claim. |

## 8. Added By tasks

| # | Task | Why | Acceptance check |
|---|---|---|---|
| 19 | Build RentOk/System row as `added_by = 1` (rent-manager) when amount exists. | No separate system value exists. `ADDED_BY_MAP` aliases 'RentOk Manager' / 'EazyPG Rent Manager' → `1`. **Caveat:** the recurring-rent generator is a Firebase cloud function, not in this repo, so the `added_by` of auto-rent cannot be confirmed from the backend code. | The `added_by = 1` row appears labelled "RentOk/System" when its unpaid amount > 0. **Confirm first** with a DB check (`SELECT DISTINCT added_by FROM invoices WHERE due_type = 'Rent'` for system-created rows) or with Jatin. |
| 20 | Build top creator rows after RentOk/system. | Owners need to know who created the largest dues load. | Biggest creators appear until visible row limit is reached. |
| 21 | Build Others for remaining creators. | Long creator lists need a readable roll-up. | Visible rows + Others equals selected-duration unpaid dues by creator. |
| 22 | Build Other members detail. | Owners need to inspect the roll-up. | Detail shows creator name, unpaid amount, and bill count. |

## 9. Ageing tasks

| # | Task | Why | Acceptance check |
|---|---|---|---|
| 23 | Replace the 60-day binary defaulter logic with 1-7, 8-14, 15-21, 22+ buckets. | Owners need stale-money shape, not one old-defaulter filter. | Buckets sum to overdue unpaid dues in selected duration. |
| 24 | Exclude future dues from ageing. | Future dues are not overdue. | No due date after today appears in ageing buckets. |
| 25 | Build Ageing by Category using top 4 + Others. | Owners need to know what type of dues is getting stale. | Category ageing total equals Ageing by Amount total. |
| 26 | Build ageing drills. | Escalation starts from the list. | Each bucket opens the exact overdue-day range. |

## 10. Other section tasks

| # | Task | Why | Acceptance check |
|---|---|---|---|
| 27 | Build Short-stay vs Long-stay dues (`is_short_term`). | Stay type changes recovery behaviour. | Short + Long equals selected-duration dues for tenants with stay type known. |
| 28 | Build Deposit Dues (`Security Deposit` + `Caution Money`). | Deposit dues are unpaid money, separate from deposits held. | Only unpaid deposit-category dues appear. |
| 29 | Build Property-wise Dues. | Multi-property owners need to know where the load sits. | Property rows sum to selected-duration unpaid dues across selected properties. |

## 11. Projected Fixed Dues tasks

| # | Task | Why | Acceptance check |
|---|---|---|---|
| 30 | Identify recurring fixed dues that RentOk will create later. | Projection must not be guessed. | Rule lists qualifying recurring due types. |
| 31 | Build tomorrow-to-month-end projection. | Owner needs expected fixed load before it is created. | Projection includes qualifying fixed dues only. |
| 32 | Add unavailable state. | If projection cannot be known, a blank or guessed number is harmful. | Unknown projection shows unavailable state or is hidden. |

## 12. Drill and permission tasks

| # | Task | Why | Acceptance check |
|---|---|---|---|
| 33 | Implement every drill in the Ground Truth drill table. | Every amount needs a follow-up path. | Row amount and opened list total match. |
| 34 | Protect weak drill routes before launch. | More analytics drill traffic increases risk. (Verify the `invoices.ts:203` / `tenant.ts:927/944` routes — not yet confirmed.) | Drill endpoints require the correct auth checks. |
| 35 | Revisit hardcoded list limit (`limit = 5000`, pagination ignored). | Large properties should not silently miss rows. | Pagination decision recorded and implemented if needed. |

## 13. Matching QA

| Check | Expected |
|---|---|
| Category | Top 4 + Others equals selected-duration unpaid dues. |
| Added By | RentOk/system (`added_by=1`) + creator rows + Others equals selected-duration unpaid dues by creator. |
| Ageing | 1-7 + 8-14 + 15-21 + 22+ equals overdue unpaid dues in duration. |
| Ageing by Category | Top 4 + Others equals Ageing by Amount total. |
| Property-wise | Sum of property rows equals selected-duration unpaid dues. |
| Short/Long | Short + Long equals selected-duration dues where stay type is known. |
| Tenant Journey | Each row matches its drill. Do not force risk slices to sum. |
| Top Summary | Values do not change when duration changes. |
| Drill parity | Opened list total matches tapped amount. |

## 14. Edge cases

| Case | Expected behavior |
|---|---|
| No unpaid dues | Show zero state. |
| Fewer than 4 categories | Show real categories only. Hide zero Others. |
| Fewer than 4 creators | Show real creators only. Hide zero Others. |
| RentOk/system has no amount | Do not show empty system row. |
| No overdue dues in duration | Ageing shows zero state. |
| Projection unknown | Hide or show unavailable; do not guess. |
| Old tenant (`status = 0`) has dues | Include in All Dues and Old Tenants. |
| Restricted user | Totals and drills are scoped to allowed data (apply scope to the widget too). |

## 15. Smoke tests

| Test | Expected |
|---|---|
| Change duration from This Month to Last Month | Category, Journey, Added By, Ageing, Property change; Top Summary does not. |
| Tap Category Others | List excludes visible top categories and matches Others amount. |
| Tap Confirmed Eviction | List shows unpaid dues for tenants with a confirmed eviction date. |
| Tap 22+ ageing | List shows overdue dues 22 or more days old. |
| Remove all overdue dues | Ageing becomes zero state. |
| Add old-tenant (`status = 0`) unpaid invoice | All Dues and Old Tenants include it. |
| Self-added-only user opens analytics | Analytics total matches scoped dues list (widget scope applied). |

## 16. Launch blockers

- Top Summary changes when duration changes.
- Any visible amount does not match its drill.
- Old-tenant dues (`status = 0`) are excluded from All Dues.
- Projection ships without a clear recurring-dues rule.
- Notice Raised or Confirmed Eviction is presented as a strict additive total with Active.
- Restricted user sees broader analytics than list access allows (widget scope not applied).

## Changelog

| Date | Version | Change |
|---|---|---|
| 2026-06-06 | v3.0 | Rebuilt as a self-contained build sheet with foundation tasks, duration behavior, drill parity, QA, edge cases, and launch blockers. |
| 2026-06-06 | v4.0 | Schema-grounding pass. Task 2 corrected (status 0 = Old Tenant; base → IN 0,1,2). Task 3 flipped to partial-paid excluded (locked). Task 14 Notice = eviction-details status 2. Task 19 System = added_by 1. Task 9 current-month through-today. Relabelled anchors: 3342/3469 = complaints card (shape only), 2790/2867 = tenant counts (reuse predicates only). Drill-parity glossed. |
