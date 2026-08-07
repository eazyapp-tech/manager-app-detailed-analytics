---
title: DA-01 Dues — Ground-Truth Formula Map
date: 2026-06-06
tags: [rentok, dues, ground-truth, formula-map]
status: Living document · v4.2
owner: Sanchay
---

> [!INFO] What this is
> This document defines every number on the Dues detailed-analytics screen.
>
> It explains what each number means, how time affects it, what each amount opens when you tap it, and which numbers must add up to which totals. It is written so design, engineering, QA, and product all use the same meaning. Section 20 carries the real database fields so this stays a true contract, not a second brief.

## 1. The base rule

DA-01 is about **unpaid dues** — the money still owed.

A due counts if its invoice is still open (`invoices.status = 0`), is active (`is_active = 1`), and the amount is at least ₹1.

**Partial payments need no special rule — they are already counted right.** When a tenant pays part of a bill, the backend does **not** split it into two rows. It reduces the same invoice in place: `amount` and `net_amount` drop to the remaining balance, and the invoice stays `status = 0` ([payment.ts:2048](/Users/eazypg/rentok-backend/src/controllers/payment.ts#L2048)). So summing `amount` over `status = 0` already gives the correct *remaining* owed amount. (The `status = 2` "Partially Paid" value is not produced by the manual payment flow, and the `parent_invoice_id` / `root_invoice_id` split columns are never written — that split-and-rejoin design is dormant.)

**Every count in this doc is scoped by the viewer's permissions.** If the viewer has only `view_invoices_of_self_added_tenants`, only invoices from tenants *they* added are included. The widget endpoint currently misses this scope ([service.ts:132](/Users/eazypg/rentok-backend/src/v1/list_screens/dues/service.ts#L132)) — that is a security gap (cross-suite blocker CSB-8), covered by the build sheet. The list endpoint applies it correctly at [service.ts:95](/Users/eazypg/rentok-backend/src/v1/list_screens/dues/service.ts#L95). Every DA-01 query in every view must apply the same scope.

## 2. Words that must stay separate

| Word                     | Meaning                                                                             |
| ------------------------ | ----------------------------------------------------------------------------------- |
| **All Dues**             | All fully unpaid dues across selected properties, no matter when due.               |
| **Past Dues**            | Unpaid dues due before the current month.                                           |
| **Current Month Dues**   | Unpaid dues due from the first day of this month through today.                     |
| **Future Dues**          | Unpaid dues already created, with due date from tomorrow onward.                    |
| **Projected Fixed Dues** | Fixed recurring dues expected to be created later. These are not created bills yet. |
| **Deposit Dues**         | Deposit money still unpaid (`Security Deposit` or `Caution Money`).                 |
| **Deposits Held**        | Deposit money already collected and still held. Not part of DA-01 Deposit Dues.     |

## 3. Duration modes

The screen uses five duration modes for deeper views:

| Duration       | Meaning                                                                        |
| -------------- | ------------------------------------------------------------------------------ |
| **All Time**   | All unpaid dues in the selected properties.                                    |
| **This Month** | Unpaid dues whose due date is from the first day of this month through today.  |
| **Last Month** | Unpaid dues whose due date was in or before the previous calendar month.       |
| **Current FY** | Unpaid dues whose due date is inside the current financial year (April start). |
| **Custom**     | Unpaid dues whose due date falls inside the selected start and end dates.      |

> [!WARNING] Code divergence — must build new
> The live filter for "this month" (`DuesFilterCode.CURRENT_MONTH`, [helpers.ts:132](/Users/eazypg/rentok-backend/src/v1/list_screens/dues/helpers.ts#L132)) covers the **whole month including future days** (`due_date >= monthStart AND < nextMonth`). DA-01 needs **through today**. This is a new range to build, not a reuse. Decision locked: through today.
>
> "Through today" means `due_date < tomorrow 00:00 IST` (equivalently `due_date <= today 23:59:59 IST`). Use IST (UTC+5:30), not the server timezone.

## 4. Time behavior matrix

| Section                             | Uses duration picker? | Time rule                                                                                                                      |
| ----------------------------------- | --------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| **Top Summary**                     | No                    | Each number keeps its own fixed meaning.                                                                                       |
| **Current Month Category Breakups** | No                    | Always current month through today.                                                                                            |
| **Projected Fixed Dues**            | No                    | Always tomorrow through the end of the current month.                                                                          |
| **Category Breakdown**              | Yes                   | Group unpaid dues whose due date falls inside the selected duration.                                                           |
| **Tenant Journey/status Breakdown** | Yes                   | Select dues by due date duration, then group by the tenant's current stage.                                                    |
| **Added By Breakdown**              | Yes                   | Group unpaid dues whose due date falls inside the selected duration.                                                           |
| **Ageing by Amount**                | Yes                   | Select unpaid dues by due date duration, then bucket by how many days overdue they are today. Future dues do not enter ageing. |
| **Ageing by Category**              | Yes                   | Same overdue set, grouped by top 4 categories + Others.                                                                        |
| **Short-stay vs Long-stay**         | Yes                   | Select unpaid dues by due date duration, then split by tenant stay type.                                                       |
| **Deposit Dues**                    | Yes                   | Select unpaid deposit dues by due date duration.                                                                               |
| **Property-wise Dues**              | Yes                   | Select unpaid dues by due date duration, then group by property.                                                               |

## 5. Top Summary

These numbers do not change when the duration picker changes.

| Number                 | Plain formula                                                                        | What it tells the owner                                       |
| ---------------------- | ------------------------------------------------------------------------------------ | ------------------------------------------------------------- |
| **All Dues**           | Add all fully unpaid dues across selected properties.                                | "This is the total money still owed."                         |
| **Past Dues**          | Add unpaid dues with due date before the first day of the current month.             | "This is old unpaid money."                                   |
| **Current Month Dues** | Add unpaid dues with due date from the first day of the current month through today. | "This month's unpaid load so far."                            |
| **Total Future Dues**  | Add unpaid dues already created with due date from tomorrow onward.                  | "This money is not late yet, but it is already on the books." |
| **Current FY Dues**    | Add unpaid dues with due date inside the current financial year.                     | "This is unpaid money in the current financial year."         |

## 6. Current Month Category Breakups

These numbers stay tied to the current month through today. Each one matches a real `due_type` string (see Section 20).

The section header and each row label show the **actual month name** (e.g. "May Rent Dues", "May Electricity Dues"). The query range is still current month through today — the month name is a display label only.

| Number                             | Plain formula                                                                         | What it tells the owner                       |
| ---------------------------------- | ------------------------------------------------------------------------------------- | --------------------------------------------- |
| **Current Month Rent Dues**        | Current-month unpaid dues where `due_type = 'Rent'`.                                   | "Rent still unpaid this month."               |
| **Current Month Electricity Dues** | Current-month unpaid dues where `due_type LIKE 'Electricity%'`.                        | "Utility money still unpaid this month."      |
| **Current Month Food / Mess Dues** | Current-month unpaid dues where `due_type LIKE 'Food%' OR LIKE 'Mess%'`.               | "Food-related money still unpaid this month." |
| **Current Month Deposit Dues**     | Current-month unpaid dues where `due_type IN ('Security Deposit','Caution Money')`.    | "Deposit money still unpaid this month."      |

## 7. Projected Fixed Dues

| Number                   | Plain formula                                                                              | What it tells the owner                                      |
| ------------------------ | ------------------------------------------------------------------------------------------ | ------------------------------------------------------------ |
| **Projected Fixed Dues** | Add fixed recurring dues the system is expected to create from tomorrow through month-end. | "This money is expected, but the bills are not created yet." |

This can include rent, food, or other fixed recurring dues, depending on how the property is set up.

**Projected dues breakdown chart:** Below the total, show a chart with X axis = due date (tomorrow through month-end) and Y axis = projected amount, grouped by category (Rent, Food, and other recurring types). Each bar segment shows what category is expected on that date. This uses the same recurring schedule data as the total — it is not a separate query, just a breakdown of the same projected set.

> [!WARNING] New build — not in production today
> There is no existing production query for this number or its chart. Both must be built. The projection should be derived from each property's recurring schedule configuration (which `due_type` values repeat, at what frequency, and at what amount). The same schedule data that drives auto-rent creation is the source.
>
> If the schedule data is unavailable or ambiguous (e.g., a property with no active recurring schedule), show 0. Do not estimate or guess.

## 8. Category Breakdown

This view uses the selected duration.

| Row                  | Plain formula                                                                | What it tells the owner                        |
| -------------------- | ---------------------------------------------------------------------------- | ---------------------------------------------- |
| **Top 4 categories** | Group unpaid dues by category, sort by unpaid amount, show the four largest. | "These are the biggest types of unpaid money." |
| **Others**           | Add every remaining category after the top four.                             | "Smaller categories grouped together."         |

Category names are not fixed. Rent, electricity, deposit, food, late fine, or any custom category can appear if it is in the top four for that duration. Category is derived from the `due_type` string (Section 20).

## 9. Tenant Journey Breakdown

This view uses the selected duration.

This is a **stage view**, not a strict sum view. It follows the way owners already think about tenant movement. Each row keys off `tenant.status` and the eviction tables (Section 20).

| Row                    | Plain formula                                                                                    | What it tells the owner                     |
| ---------------------- | ------------------------------------------------------------------------------------------------ | ------------------------------------------- |
| **Active Tenants**     | Unpaid dues from tenants currently living in the property (`status = 1`, not on notice or eviction). | "Money owed by people still inside."        |
| **Notice Raised**      | Unpaid dues from active tenants where `eviction_details.is_active = 1 AND eviction_details.status = 2` and `tenant.date_of_eviction IS NULL`. | "Exit risk has started."                    |
| **Confirmed Eviction** | Unpaid dues from active tenants where `date_of_eviction IS NOT NULL`.                            | "Act before this becomes old-tenant money." |
| **Bookings**           | Unpaid dues from booked tenants (`status = 2`) who have not checked in, scoped to the selected duration. | "Move-in or booking-related dues."          |
| **Old Tenants**        | Unpaid dues from moved-out tenants (`status = 0`).                                               | "Harder recovery."                          |

Notice Raised and Confirmed Eviction sit inside Active Tenants. Do not make a total by adding all stage rows together unless the design clearly separates base rows from risk rows.

> [!WARNING] Notice Raised definition differs from homescreen tile
> This "Notice Raised" definition (eviction-details `status = 2`, no `date_of_eviction`) is narrower than the homescreen "Under Notice" tile, which uses `date_of_eviction IS NOT NULL` alone ([homepage/service.ts:798](/Users/eazypg/rentok-backend/src/v1/homepage/service.ts#L798)). Homescreen also uses a different label ("Under Notice" vs "Notice Raised"). These are separate product definitions by design — DA-01 splits the exit-risk stage into two rows (Notice Raised → Confirmed Eviction) where the homescreen has one. Build against DA-01's definition, not the homescreen's.

## 10. Added By Breakdown

This view uses the selected duration. It groups by the invoice creator (`invoices.added_by`), not by who added the tenant.

| Row                      | Plain formula                                                                  | What it tells the owner              |
| ------------------------ | ------------------------------------------------------------------------------ | ------------------------------------ |
| **RentOk / System**      | Unpaid dues where `added_by = 1` (rent-manager — this is how auto-created recurring rent is written). Show when amount > 0. | "How much load the system created."  |
| **Top actors**           | After RentOk/system, show the biggest other creators by unpaid amount.         | "Who created the biggest dues load." |
| **Others**               | Add every remaining creator after the visible rows.                            | "Smaller creators grouped together." |
| **Other members detail** | Show the people inside Others with unpaid amount and bill count.               | "Who sits inside the roll-up."       |

If RentOk/system has no unpaid amount, the visible rows can be the top 4 creators plus Others.

If RentOk/system has unpaid amount, keep it visible, then show the next biggest creators until the visible row limit is reached. **The visible row limit is 4 total** (RentOk/System counts as one if shown, then top creators until the visible set reaches 4; everything remaining rolls into "Others").

## 11. Ageing by Amount

This view uses the selected duration, but ageing is measured as of today.

Only overdue unpaid dues enter ageing. Future dues do not enter ageing.

A due is overdue only if its due date is before today. A due dated today is not yet overdue and does not enter ageing.

| Bucket         | Plain formula                        | SQL boundary | What it tells the owner |
| -------------- | ------------------------------------ | ------------ | ----------------------- |
| **1-7 days**   | Unpaid dues due 1 to 7 days ago.     | `due_date >= today − 7 AND due_date < today` | "Fresh overdue money."  |
| **8-14 days**  | Unpaid dues due 8 to 14 days ago.    | `due_date >= today − 14 AND due_date < today − 7` | "Needs follow-up."      |
| **15-21 days** | Unpaid dues due 15 to 21 days ago.   | `due_date >= today − 21 AND due_date < today − 14` | "Getting risky."        |
| **22+ days**   | Unpaid dues due 22 or more days ago. | `due_date < today − 21` | "Escalate."             |

All date arithmetic uses IST (UTC+5:30). `today` = current date at 00:00 IST.

> These four buckets are new. The only ageing logic in code today is a single 60-day defaulter check ([helpers.ts:295](/Users/eazypg/rentok-backend/src/v1/list_screens/dues/helpers.ts#L295)) — it is binary and must be replaced.

## 12. Ageing by Category

Use the same overdue set as Ageing by Amount (the "pool" of overdue unpaid dues).

Group it by category, show top 4 categories, then Others.

This reuses the same `due_type`-based category derivation as Category Breakdown (§8). It is the same grouping logic, applied to a different pool (overdue-only instead of all unpaid).

This answers: *what type of dues is becoming stale?*

## 13. Short-stay vs Long-stay

This view uses the selected duration. Short-stay is `tenant.is_short_term = true`.

| Row                 | Plain formula                        | What it tells the owner            |
| ------------------- | ------------------------------------ | ---------------------------------- |
| **Short-stay Dues** | Unpaid dues from short-stay tenants where `tenant.is_short_term = true`. | "Money from shorter-stay tenants." |
| **Long-stay Dues**  | Unpaid dues from all other tenants where `tenant.is_short_term = false`. | "Money from regular tenants."      |

Short-stay plus long-stay adds up to the unpaid dues for tenants whose stay type is known (`is_short_term IS NOT NULL`). Tenants where `is_short_term IS NULL` are excluded from both buckets — they fall outside the short-stay/long-stay split. If the matching rule in §17 does not balance, check whether NULL-stay-type tenants exist in the duration.

## 14. Deposit Dues

This view uses the selected duration.

| Number           | Plain formula                                                          | What it tells the owner     |
| ---------------- | --------------------------------------------------------------------- | --------------------------- |
| **Deposit Dues** | Unpaid dues where `due_type IN ('Security Deposit','Caution Money')`. | "Deposit money still owed." |

"Security Deposit" and "Caution Money" are the same thing in practice — they are both deposit money paid at move-in. The two `due_type` strings exist for historical reasons; always treat them as one group.

**Deposit comparison chart (Q7):** This section shows a side-by-side chart of dues vs received for deposit money.

| Side | Plain formula |
| ---- | ------------- |
| **Due** | Same as Deposit Dues above — unpaid invoices where `due_type IN ('Security Deposit','Caution Money')`, `status = 0`, `is_active = 1`. Scoped by the duration picker. |
| **Received / Held** | Total deposit payments collected and still held — payments where the linked invoice has `due_type IN ('Security Deposit','Caution Money')` and `payment.status = 1 AND payment.is_active = 1`. This uses the same definition as Deposits Held in DA-02. |

> [!WARNING] Duration scope for the "Received" side
> Confirm with the owner: does the "received" bar use the duration picker (collections in the selected period), or does it always show the total deposits held to date (a snapshot)? The "due" side is period-scoped; the "received" side may not be. Until confirmed, build the "received" side as a snapshot (total held, no duration filter) and flag it for UAT.

## 15. Property-wise Dues

This view uses the selected duration.

| Row                 | Plain formula                                                                 | What it tells the owner                |
| ------------------- | ----------------------------------------------------------------------------- | -------------------------------------- |
| **Property amount** | Add unpaid dues for one property.                                             | "Which property is carrying the load." |
| **Property share**  | Property unpaid dues divided by total unpaid dues across selected properties. | "How much of the problem sits here."   |

## 16. Drill-down table

What each amount opens when tapped. Every amount should open the exact slice it describes.

| Tap target                                        | Opens                       | Filter meaning                                                                                     |
| ------------------------------------------------- | --------------------------- | -------------------------------------------------------------------------------------------------- |
| All Dues                                          | Dues list                   | All unpaid dues across selected properties.                                                        |
| Past Dues                                         | Dues list                   | Due date before current month.                                                                     |
| Current Month Dues                                | Dues list                   | Due date from current month start through today.                                                   |
| Future Dues                                       | Dues list                   | Due date from tomorrow onward.                                                                     |
| Current FY Dues                                   | Dues list                   | Due date inside current financial year.                                                            |
| Current Month Rent / Electricity / Food / Deposit | Dues list                   | Current-month dues plus selected category.                                                         |
| Projected Fixed Dues                              | Detail sheet or explanation | Expected recurring dues, not existing bills. Do not open the dues list unless bills already exist. |
| Category row                                      | Dues list                   | Selected duration plus that category.                                                              |
| Category Others                                   | Dues list                   | Selected duration excluding the visible top categories.                                            |
| Active Tenants                                    | Dues list                   | Selected duration plus active tenants (`status = 1`).                                              |
| Notice Raised                                     | Dues list                   | Selected duration plus active tenants with eviction-details `status = 2`.                          |
| Confirmed Eviction                                | Dues list                   | Selected duration plus active tenants with `date_of_eviction IS NOT NULL`.                         |
| Bookings                                          | Dues list                   | Selected duration plus bookings (`status = 2`).                                                    |
| Old Tenants                                       | Dues list                   | Selected duration plus old tenants (`status = 0`).                                                 |
| Added By row                                      | Dues list                   | Selected duration plus that creator.                                                               |
| Added By Others                                   | Other members detail        | Creators hidden inside Others. From there, each creator opens the dues list.                       |
| Ageing bucket                                     | Dues list                   | Selected duration plus overdue-day bucket.                                                         |
| Ageing category row                               | Dues list                   | Selected duration plus overdue-only plus category.                                                 |
| Short-stay / Long-stay                            | Dues list                   | Selected duration plus stay type.                                                                  |
| Deposit Dues                                      | Dues list                   | Selected duration plus Deposit category.                                                           |
| Property row                                      | Dues list                   | Selected duration plus property.                                                                   |

## 17. Matching rules

| Check              | Rule                                                                                                                                   |
| ------------------ | -------------------------------------------------------------------------------------------------------------------------------------- |
| Category           | Top 4 categories + Others = selected-duration unpaid dues.                                                                             |
| Added By           | RentOk/system (`added_by=1`) + visible creator rows + Others = selected-duration unpaid dues.                                          |
| Ageing by Amount   | 1-7 + 8-14 + 15-21 + 22+ = overdue unpaid dues inside selected duration.                                                               |
| Ageing by Category | Top 4 stale categories + Others = same total as Ageing by Amount.                                                                      |
| Stay Duration      | Short-stay + Long-stay = selected-duration unpaid dues for tenants with known stay type. Tenants where `is_short_term IS NULL` are excluded from both buckets.                                               |
| Property-wise      | Sum of property rows = selected-duration unpaid dues across selected properties.                                                       |
| Tenant Journey     | Each row must match its drill. Do not force all rows to add to one total because Notice Raised and Confirmed Eviction are risk slices. |

## 18. Empty states and edge cases

| Case                                 | Behavior                                               |
| ------------------------------------ | ------------------------------------------------------ |
| No unpaid dues                       | Show zero state. Do not show fake categories.          |
| Fewer than 4 categories              | Show only real categories. Hide Others if it is zero.  |
| Fewer than 4 creators                | Show only real creators. Hide Others if it is zero.    |
| RentOk/system has no amount          | Do not pin an empty system row.                        |
| Projected Fixed Dues cannot be known | Hide or show unavailable state. Do not guess.          |
| Duration has no overdue dues         | Ageing view shows zero state for that duration.        |
| Old tenant has unpaid dues           | Include it in All Dues and Old Tenants (`status = 0`). |

## 19. Locked decisions

- All Dues means all unpaid dues (`status = 0`). Partial payments reduce the invoice in place and stay `status = 0`, so the remaining balance is counted automatically — no exclusion, no special math.
- Old Tenants = `tenant.status = 0` (moved-out). DA-01 base must include status 0.
- Future Dues are existing unpaid bills with future due dates.
- Projected Fixed Dues are expected recurring bills not yet created.
- Category uses top 4 + Others, derived from `due_type`.
- RentOk/System = `added_by = 1`. Keep visible when it has amount, then top creators + Others.
- Tenant Status follows the stage model; Notice Raised requires eviction-details `status = 2`.
- Current Month = first of month through today (new range, not the whole-month code filter).
- Ageing uses 1-7, 8-14, 15-21, 22+.
- Ground-truth totals and drill totals must match.

## 20. Schema ground-truth (the real fields)

This is what makes the map a contract. Build against these, not the prose above.

**Tenant status** — [tenant.ts:154](/Users/eazypg/rentok-backend/src/entities/tenant.ts#L154)
`0 = evicted / moved-out (Old Tenant) · 1 = active tenant · 2 = booking · 3 = lead · 4 = invite · 5 = permanently deleted · 6 = deleted invitation · 7 = deleted lead · 8 = deleted/rejected self-invite`
DA-01 base set = `status IN (0, 1, 2)`. Status 3 (lead) and 4 (invite) are excluded — leads and invited users do not have invoices raised against them; any that appear in data are a data anomaly and out of scope. The live dues base today filters `IN (1, 2)` ([helpers.ts:61](/Users/eazypg/rentok-backend/src/v1/list_screens/dues/helpers.ts#L61)) — that is the bug that hides Old Tenants.

**Invoice status** — [invoices.ts:93](/Users/eazypg/rentok-backend/src/entities/invoices.ts#L93)
`0 = Not Paid · 1 = Paid · 2 = Partially Paid · 3 = Refunded · 4 = Loss`
DA-01 base predicate = `status = 0 AND is_active = 1 AND amount >= 1` ([helpers.ts:58-60](/Users/eazypg/rentok-backend/src/v1/list_screens/dues/helpers.ts#L58)). Invoice amounts are stored in rupees. `amount >= 1` filters out zero-amount rows — it means "at least ₹1 still owed." Note: the manual partial-payment flow does **not** set `status = 2` — it reduces the open invoice's `amount` in place and leaves it `status = 0`, so the remaining balance is captured by the base. `parent_invoice_id` / `root_invoice_id` are present in the schema but written nowhere in `src/` (dormant split design).

**Invoice creator (`added_by`)** — [invoices.ts:74](/Users/eazypg/rentok-backend/src/entities/invoices.ts#L74)
`0 = owner · 1 = rent manager (= RentOk/System auto-rent) · 2 = partner · 3 = customer · 4 = tenant · 5 = others`
Label aliases in `ADDED_BY_MAP` ([filterCodes.ts:163](/Users/eazypg/rentok-backend/src/v1/constants/filterCodes.ts#L163)).

**`due_type` string values** (category is built from these) — [helpers.ts:135-161](/Users/eazypg/rentok-backend/src/v1/list_screens/dues/helpers.ts#L135)
- Rent = `'Rent'`
- Electricity = `LIKE 'Electricity%'`
- Food / Mess = `LIKE 'Food%' OR LIKE 'Mess%'` (real values include `'Mess'`, `'Food Charges'`)
- Deposit = `IN ('Security Deposit', 'Caution Money')` — **not** the literal `'Deposit'`
- Late Fine = `'Automatic Late Fine' OR 'Manual Late Fine'`

**Journey predicates (reuse the status logic, not the count math)** — [homepage/service.ts:2790-2807](/Users/eazypg/rentok-backend/src/v1/homepage/service.ts#L2790)
- Active = `status = 1 AND date_of_eviction IS NULL AND NOT EXISTS active eviction-details row`
- Notice Raised = `status = 1 AND date_of_eviction IS NULL AND EXISTS eviction-details (is_active = 1 AND status = 2)`
- Confirmed Eviction = `status = 1 AND date_of_eviction IS NOT NULL`
- Note: the date-range variant at [service.ts:2867](/Users/eazypg/rentok-backend/src/v1/homepage/service.ts#L2867) omits the `status = 2` check — for DA-01, the `status = 2` version is canonical.

**Date windows** — [helpers.ts:119-161](/Users/eazypg/rentok-backend/src/v1/list_screens/dues/helpers.ts#L119)
FY starts April (`month >= 3`). Future = `due_date >= tomorrow`. Current-Month-through-today is **new** (do not reuse `CURRENT_MONTH`).

**Permission scope** — list applies `t.added_by_id = team_uuid` ([service.ts:95](/Users/eazypg/rentok-backend/src/v1/list_screens/dues/service.ts#L95)); the widget computes the scope but never applies it ([service.ts:132](/Users/eazypg/rentok-backend/src/v1/list_screens/dues/service.ts#L132)). Every analytics aggregate must apply the same scope. This is the permission filter (on who added the *tenant*) and is separate from the Added-By breakdown (on who added the *invoice*).

## Changelog

| Date       | Version | Change                                                                                                                       |
| ---------- | ------- | ---------------------------------------------------------------------------------------------------------------------------- |
| 2026-06-06 | v3.0    | Rebuilt as a self-contained formula map with duration matrix, drill-down table, matching rules, and locked product meanings. |
| 2026-06-06 | v4.0    | Schema-grounding pass. Fixed frontmatter. Old Tenants = status 0 (base IN 0,1,2). Deposit = Security Deposit + Caution Money. Partial-paid excluded (locked). System = added_by 1. Current-month = through today (new range). Notice Raised = eviction-details status 2. Added Section 20 (real fields) to make this a true contract. |
| 2026-06-16 | v4.1    | Review pass. Added permission scope to base rule (§1). Clarified "through today" precision (§3). Flagged Projected Fixed Dues as new build with schedule-data guidance (§7). Added divergence warning on Notice Raised vs homescreen tile (§9). Clarified Booking row is duration-scoped (§9). Added explicit 4-row visible limit for Added By (§10). Made Ageing by Category's category-reuse explicit (§12). Clarified NULL `is_short_term` exclusion from both stay buckets (§13, §17). |
| 2026-06-17 | v4.2    | Figma-aligned review pass. Fixed timezone note to use IST (UTC+5:30) explicitly (§3). Added month-name label note for current-month categories (§6). Added projected dues breakdown chart definition — X = due date, Y = category (§7). Made Notice Raised predicate explicit: `is_active = 1 AND status = 2 AND date_of_eviction IS NULL` (§9). Added SQL boundaries to all ageing buckets with overdue threshold note (§11). Expanded Deposit Dues to cover comparison chart — due vs received/held sides, with UAT flag on duration scope of "received" side (§14). Clarified invoice amounts are stored in rupees, `amount >= 1` = at least ₹1 (§20). Added explanation for why status 3 and 4 are excluded from base set (§20). |
