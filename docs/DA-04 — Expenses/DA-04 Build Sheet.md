---
title: DA-04 Expenses — Build Sheet
date: 2026-06-06
tags: [rentok, expenses, financials, build-sheet, detailed-analytics]
status: Refreshed · ready for engineering review
version: v0.2 — hardened against live code, owner decisions folded in
owner: Sanchay
companion_to: DA-04 Brief · DA-04 Ground-Truth Formula Map
---

> [!WARNING] Superseded in places, as of 7 August 2026
> **The Expense Handoff Sheet in this folder overrides this document** on the points below. Where the two disagree, build what the handoff says. These corrections have not been folded back in here yet.
>
> 1. **Expenses with no paid date.** This document treats them as cleanup items needing their own list. There are none. Zero, across all 339,732 live expenses, on every property. The whole cleanup path is dropped.
> 2. **Review gaps.** Dropped as a section. Missing category, payee and payer total 21 rows out of 151,012. Missing bills went the other way: 83% of expenses have none, so it survives as a single figure showing money rather than a count.
> 3. **Fund sources.** This document names three. There is a fourth and it is large: **FlexiPe**, about one expense in seven. Petty cash is 0.35% and now appears only when used.
> 4. **"Fund source unavailable" is mostly not a silent failure.** Every FlexiPe expense has no fund record, because FlexiPe creates its expense on a path that never writes one. The genuine swallowed-failure residue is 213 rows.
> 5. **What "Other" means in the category card.** Here it is the catch-all group. On the screen it is the rollup of everything outside the top three, so a named group such as Salary can land inside it.
> 6. **The previous period.** Here it is the same length immediately before. The screen compares against the **same point** in the previous period, so an unfinished month is never measured against a finished one.
> 7. **Where "still owed to staff" opens.** Here it is a payback list. On the screen it opens the live Team Passbook.
> 8. **The negative and zero quarantine.** Still quarantined, but it appears as one row in the View all sheet only when there are any, rather than as a permanent data-quality line.
>
> Also missing here entirely, and specified in the handoff: the Expense Trend chart, the Current FY tile, and the whole **Payment Mode** breakdown. No source document establishes that expenses carry a payment mode at all.



> [!INFO] What this is
> The engineering build sheet for **DA-04 Expenses**. Build from the Ground-Truth Formula Map first, then implement tasks in the order below.
>
> This page diagnoses recorded expense spend. It does not add expenses, create reimbursements, or replace cash-flow reporting.

# DA-04 Expenses — Build Sheet

## 1. Code anchors

| Area | Anchor | What matters |
|---|---|---|
| Expense entity | `src/entities/expenses.ts:15-112` | `amount` (decimal, nullable, rupees), `paid_date`, `invoice_date`, `expense_type`, `payer` (`:23`), `paid_to`, `bill_urls`, `added_by` (0–5, `:40`), `is_active` (`:110-111`); `status` (`:82-83`, marked `//TODO-REMOVE STATUS` at `:14`) is not safe for pending. |
| Expense list base query | `src/v1/list_screens/expenses/helpers.ts:25-55` | Selects listed columns and filters `property = ANY(...)` and `is_active = 1`. |
| Expense date range | `src/v1/list_screens/expenses/helpers.ts:134-140` | Uses `paid_date`; end date is exclusive next day. |
| Expense filters | `src/v1/list_screens/expenses/service.ts:42-90` | Filter codes override manual date/category/payer filters. Analytics must not reuse current-month filter codes for arbitrary date ranges. |
| Expense routes | `src/v1/list_screens/expenses/routes.ts:8-10` | Existing list/widget/filter endpoints. Build a dedicated analytics endpoint. |
| Legacy expense route | `src/routes/expenses.ts:29-52` | Existing add/edit/delete/get/widget/details routes. Some are legacy and use `SELECT *`; do not copy. |
| Add expense | `src/controllers/expenses.ts:500-562` | Creates expense, then registers passbook expense. |
| Delete expense | `src/controllers/expenses.ts:163-206` | Marks expense inactive, then reverses Team Passbook expense rows. |
| Team Passbook categories | `src/services/teamPassbook/constants.ts:3-25` | `CATEGORY_MAP` (`:3-25`): Expense, Expense Reversed, Refund, Reimbursement, and related categories. |
| Fund ids | `src/services/teamPassbook/constants.ts:28-33` | `FUND_ID` constants (`:28-33`): `PF`, `AF`, `NPNAF`, `EF`; expose only the first three on DA-04. `normalizeFundId` (`:37-40`) coerces missing/unknown to `NPNAF`, so no row has a null fund id. |
| Expense fund fan-out | `src/services/teamPassbook/teamPassbookOperationFactories/teamPassbookExpenseService.ts:38-50` | One expense can create multiple passbook rows, one per fund split. |
| Expense map table | `src/entities/tmTransactionExpenseMap.ts:20-24` | Joins passbook transactions back to expense UUID. |
| Passbook transaction table | `src/entities/teamMemberTransactions.ts:23-61` | Has `team_uuid`, amount, category, balance type, fund id, property id, and created date. |
| Reimbursement allocation | `src/services/teamPassbook/teamPassbookOperationFactories/teamPassbookReimbursement.ts:331-420` | Links reimbursements back to expense/refund targets. |
| Reimbursement transaction legs | `src/services/teamPassbook/teamPassbookOperationFactories/teamPassbookReimbursement.ts:423-450` | Payer outflow uses `EF`; receiver inflow uses `PF`. |
| Reimbursement metadata | `src/services/reimbursement/reimbursement.ts:360-584` | Existing open PF logic for expense and refund targets. |
| Homepage expenses | `src/v1/homepage/service.ts:2455-2462`, `src/v1/homepage/service.ts:2575-2579` | Uses `paid_date`; counts `paid_date IS NULL` as pending-like only in homescreen metadata. |
| Cash-flow report | `src/services/reports/generateCashFlowReport.ts:235-256`, `src/services/reports/generateCashFlowReport.ts:383-426` | Expenses and refunds are both cash-flow outflows; refund rows are separate from expense rows. |
| List report export | `src/v1/list_screens/reports/services.ts:39-40`, `src/v1/list_screens/reports/services.ts:96-100` | Expense list export can be reused for drill export. |

## 2. Build order

Build in this order:

1. **Analytics base query.** One shared base for active expenses in selected property scope and selected `paid_date` range.
2. **Raw expense sections.** Total, category groups, paid-to, paid-by, property split, review gaps.
3. **Drill payloads.** Each raw expense number must point to `POST /v1/expenses/list/filters` with matching filters.
4. **Team Passbook joins.** Add fund-source and open personal-fund payback only after raw spend is correct.
5. **Reimbursement boundary.** Subtract reimbursement allocations against expense targets only. Do not pull refund allocations into DA-04 expense payback totals.
6. **Comparison period.** Add prior equal-length period after the current-period numbers are stable.
7. **QA and smoke tests.** Validate totals against list results and passbook detail rows.

## 3. Endpoint shape

Create a dedicated analytics endpoint instead of overloading list widgets.

Recommended:

`POST /v1/expenses/analytics`

Query string:

```text
property_id=uuid
```

Body:

```json
{
  "pg_number_filter": [1, 2],
  "start_date": "2026-06-01",
  "end_date": "2026-06-30"
}
```

Response outline:

```json
{
  "status": 200,
  "message": "Success",
  "data": {
    "period": {},
    "summary": {},
    "category_breakdown": [],
    "paid_to_breakdown": [],
    "paid_by_breakdown": [],
    "property_breakdown": [],
    "fund_source": {},
    "open_personal_fund_payback": {},
    "review_gaps": {},
    "comparison": {},
    "drills": {}
  }
}
```

Do not use filter codes `1301-1310` for analytics totals. They are current-month-coded in `ExpenseListHelper.applyFilterCodes`.

## 4. Tasks

| # | Task | Why | Acceptance check |
|---|---|---|---|
| 1 | Add a DA-04 analytics service with a single expense base query. | All numbers must start from the same spend set. | Every expense total uses `is_active = 1`, selected properties, and `paid_date` range. |
| 2 | Resolve property scope from `property_id` and optional `pg_number_filter`. | Matches existing expense list behavior. | Single-property and multi-property requests match list totals for the same filters. |
| 3 | Return total spend and entry count. | The headline number anchors the page. | Total = `Math.round(SUM(amount))` (round the total, not each row); negative and zero `amount` rows are excluded and reported in a data-quality line. Sum equals `data.total_amount` from `POST /v1/expenses/list/filters` for the same date and property filters. |
| 4 | Return category groups. | Owner needs the reason for spend. | Salary, Maintenance, Rent, Electricity, Food/Mess, Deposit Refunds (`expense_type ILIKE 'Deposit%'`), Other add up to total. Use the live widget label "Deposit Refunds". |
| 5 | Return data-driven category rows under Other. | "Other" must be explainable. | Other detail lists stored `expense_type` values sorted by amount. |
| 6 | Return paid-to breakdown. | Shows vendor/person receiving money. | Aggregate over this period's expenses — do NOT reuse `getDistinctPaidTo` (`helpers.ts:342-351`, all-time, no date range). Blank paid-to is grouped as "Missing paid-to" and counted as a review gap. |
| 7 | Return paid-by breakdown. | Shows who paid or recorded the expense. | Uses `payer`; blank payer is grouped as "Missing paid-by". |
| 8 | Return property breakdown. | Multi-property owners need the driver property. | Rows include property id, name, amount, count, and drill payload. |
| 9 | Return review gaps. | Helps month-end cleanup. | Counts missing bill, paid-to, category, paid date, and description. Each gap has an expense-list drill. |
| 10 | Add fund-source query. | Shows collected funds, petty cash, and staff personal funds. | Expense passbook rows grouped by `fund_id` and linked through `tm_transaction_expense_map`. |
| 11 | Add open personal-fund payback. | Staff-fronted expenses create owner liability. | Open PF = active PF expense amount minus reimbursement allocations for target type `EXPENSE`. |
| 12 | Exclude reimbursement rows from spend totals. | Reimbursement pays back staff; it is not new expense spend. | `CATEGORY_MAP.REIMBURSEMENT` never contributes to DA-04 total spend. |
| 13 | Exclude refund rows from expense totals. | Refunds belong to refund analytics and cash flow. | `refunds` table and `CATEGORY_MAP.REFUND` do not contribute to DA-04 expense total. |
| 14 | Add previous-period comparison. | Shows whether spend is moving. | Prior period is the same length immediately before selected start date. Blank prior period returns no percent. |
| 15 | Add drill payloads for every row. | Diagnosis must lead to the existing list. | Each row can be opened through `POST /v1/expenses/list/filters` or Team Passbook detail where applicable. |
| 16 | Add permission checks. | Restricted users must not see wider totals than their list access. | Same user gets matching analytics and list rows; unauthorized users receive 401. |

## 5. Formula requirements

Use the Formula Map as the source for exact rules. Engineering summary:

- Amounts are stored in rupees as numeric/decimal in current expense code. Do not introduce paise conversion in this build unless the surrounding expense code switches to paise.
- Round display amounts to whole rupees, matching existing expense list behavior.
- Date filter is `paid_date >= start_of_day(start_date)` and `paid_date < start_of_day(end_date + 1 day)`.
- Rows with `paid_date IS NULL` are excluded from period spend and counted in review gaps.
- Current-period category groups use prefix matching for fixed groups, not exact equality, to match list filter behavior.
- Other = total minus fixed groups, with data-driven subrows from remaining categories.
- Fund source reads Team Passbook rows with categories `Expense` and `Expense Reversed`; the live fund-source amount subtracts the reversed rows from the expense rows.
- `EF` is internal to reimbursements. Do not show it as an expense fund source.

## 6. QA plan

### Raw expense QA

1. Pick one property with expenses in the selected period.
2. Call `POST /v1/expenses/list/filters` with the same date range.
3. Confirm DA-04 total spend equals list `total_amount`.
4. Confirm category amounts add to total.
5. Confirm paid-to, paid-by, and property rows add to total.
6. Confirm blank fields land in review gaps.

### Fund-source QA

1. Create or find an expense with split funds: NPNAF, AF, PF.
2. Confirm DA-04 fund-source rows match Team Passbook rows linked through `tm_transaction_expense_map`.
3. Create a reimbursement against the PF expense.
4. Confirm total spend stays unchanged.
5. Confirm open personal-fund payback reduces by the reimbursed amount.
6. Confirm refund reimbursement allocations do not reduce DA-04 expense payback.
7. **Fund-source coverage assertion.** Count period expenses with **no `tm_transaction_expense_map` row** (passbook fan-out was swallowed — `teamPassbookExpenseService.ts:54-59` has `// throw error;` commented out). That count is the fund-source coverage gap. Surface it; it must equal the "Fund source unavailable" total.

### Data-quality QA (negative / zero rows)

1. Create one negative-amount and one zero-amount expense in the period.
2. Confirm neither is in the headline total.
3. Confirm both appear in the data-quality line with correct count and summed amount.

### Delete/reversal QA

1. Delete an expense.
2. Confirm raw expense total excludes it because `expenses.is_active = 0`.
3. Confirm fund-source net does not leave the original expense as live spend after reversal.
4. Confirm audit context can show the reversed event if that section ships.

### Permission QA

1. Owner request: analytics total matches owner expense list.
2. Restricted team request: analytics total matches that team member's expense list.
3. User without expense view: analytics endpoint returns 401.

## 7. Smoke-test commands

Replace IDs before running.

```bash
curl -X POST "$BASE_URL/v1/expenses/analytics?property_id=$PROPERTY_ID" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "start_date": "2026-06-01",
    "end_date": "2026-06-30",
    "pg_number_filter": [1]
  }'
```

```bash
curl -X POST "$BASE_URL/v1/expenses/list/filters?property_id=$PROPERTY_ID" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "limit": 10000,
    "offset": 0,
    "start_date": "2026-06-01",
    "end_date": "2026-06-30",
    "pg_number_filter": [1]
  }'
```

## 8. Launch blockers

- Analytics total does not match the expense list for the same filters.
- Any refund table amount is included in DA-04 total spend.
- Reimbursement amount increases total spend.
- Personal-fund exposure cannot be tied to the correct expense target.
- Restricted user sees a wider analytics total than list total.
- Period totals use `created_at` or `invoice_date` instead of `paid_date`.
- Current-month filter codes are used for arbitrary date ranges.
- Deleted expenses remain in live spend.

## 9. Known codebase risks

- Legacy `getExpenses` uses `SELECT *`; do not copy it into analytics.
- `status` exists on `expenses` but is marked for removal; do not use it for pending.
- Homepage label "Pending Expenses" is a weekly logging reminder, not a data state.
- `propertyMonthlySummary` currently sums expenses by `created_at` (`propertyMonthlySummary.ts:243`), unlike list/homepage/cash-flow code. DA-04 must use `paid_date` and must not inherit that.
- Expenses are created **in-repo** (`addNewExpense → createNewExpense + registerExpense`, `controllers/expenses.ts:500-560`). The Firebase out-of-repo creation caveat that applies to some other domains does not apply here — the full create path is visible in this repo.
- Team Passbook write paths catch some errors without rethrowing. Fund-source coverage needs a "missing passbook rows" diagnostic during QA.
- Reimbursement controller is named like refunds in places. Trust what the code does, not what the class is called, to set definitions.
