---
title: DA-04 Expenses — Ground-Truth Formula Map
date: 2026-06-06
tags: [rentok, expenses, financials, formula-map, detailed-analytics]
status: Refreshed · plain-language source of truth
version: v0.2 — hardened against live code, owner decisions folded in
owner: Sanchay
companion_to: DA-04 Brief · DA-04 Build Sheet
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



> **Who this is for.** Anyone on the team can read this once and know what every DA-04 Expenses number means.
>
> This page defines the numbers, time behavior, matching rules, next-step behavior, empty states, and edge cases. Build details live in the Build Sheet.

# DA-04 Expenses — Ground-Truth Formula Map

## 1. The screen in one line

DA-04 answers: **how much did we spend, where did it go, who paid it, who received it, and how much staff-fronted money is still unpaid?**

It is an expense diagnosis page. It is not a profit page, a reimbursement form, or a refund page.

## 2. Rules that apply to every number

1. **Only live expense records count as spend.** Deleted expense records are not live spend.
2. **Paid date controls time.** A spend belongs to a period by the date the money was paid.
3. **Expenses without a paid date are cleanup items.** They do not belong to any selected period until the paid date is set.
4. **Refunds are separate from expenses.** A refund paid back to a tenant is not counted as an expense on this page.
5. **Reimbursements are separate from expenses.** Paying staff back is not new spend. It reduces the amount the business owes staff.
6. **Categories are shown as recorded.** The page groups them, but it does not silently rename or fix them.
7. **Every total must open the same matching rows.** If the user opens a number, the list they see must add back to that number.

## 3. Plain meanings

| Term | Meaning |
|---|---|
| **Expense** | A recorded spend event: salary, maintenance, rent, electricity, food, vendor payment, or another business cost. |
| **Paid date** | The date the spend counts against. |
| **Recorded date** | The date the expense was entered in RentOk. This does not decide the period. |
| **Category** | The expense type saved on the expense. Examples: Salary, Maintenance, Electricity. |
| **Paid by** | The person or source recorded as having paid the expense. |
| **Paid to** | The vendor, person, or business that received the money. |
| **Bill attached** | Whether the expense has a bill or receipt image attached. |
| **Collected funds** | Business money already collected and used for an expense. |
| **Petty cash** | Cash the owner/business gave staff for spending. |
| **Personal funds** | Staff used their own money. This creates money the business owes back to staff. |
| **Open staff payback** | Staff personal money used for expenses, minus the amount already paid back to staff. |
| **Deposit-like** | A refund-shaped payout recorded as an expense row — money returned to a tenant (often a security deposit), logged on the expense side, not the invoice side. |

### Where this lives in the code

The numbers on this page read from one main table and a small set of join tables. Pinning them here so the formulas below stay tied to real columns.

**Table `expenses` (`src/entities/expenses.ts:15-112`):**

| Column | Type / notes |
|---|---|
| `amount` | decimal, nullable, in **rupees**. Rounded only at display, never per row (see section 6). |
| `paid_date` | timestamp, nullable. The date a spend counts against. |
| `invoice_date` | timestamp. Do not use it for period filtering. |
| `expense_type` | free-text category string. Drives the category groups in section 7. |
| `payer` | the paid-by value (`:23`). |
| `paid_to` | the receiver/vendor value. |
| `bill_urls` | text array of bill/receipt links. Empty = missing bill. |
| `added_by` | small enum, values 0–5 (`:40`). There is **no "system" value** — do not infer paid-by from who created the row. |
| `status` | marked `//TODO-REMOVE STATUS` (`:14`, `:82-83`). Do not use it for anything, including pending. |
| `is_active` | smallint, default 1 (`:110-111`). `0` = deleted. |

**Live-spend predicate** (`src/v1/list_screens/expenses/helpers.ts:54-55,134-140`): `property = ANY(:strings) AND is_active = 1 AND paid_date >= start_of_day(start) AND paid_date < start_of_day(end + 1 day)`. The end is **exclusive next day**, not inclusive end-of-day.

**Total** = `Math.round(SUM(amount))` over the filtered set (`helpers.ts:187,196`) — round the total, not each row.

**Category prefix groups** (`helpers.ts:67,82-122`): `ILIKE 'Salary%' | 'Rent%' | 'Electricity%' | 'Mess%' | 'Food%' | 'Maintenance%' | 'Deposit%'`; Other = a row matching none of these prefixes.

**Fund link** (`tmTransactionExpenseMap.ts:20-24`, `constants.ts:8-9,28-33`, `teamPassbookExpenseService.ts:38-51`): `expenses.id → tm_transaction_expense_map.expense_uuid → .tmt_id → team_member_transactions`, where `category in ('Expense','Expense Reversed')` and `fund_id in ('PF','AF','NPNAF')`. `EF` is excluded (reimbursement-internal). One expense can produce many passbook rows (a fund split).

**Fund ids** (`src/services/teamPassbook/constants.ts:28-33`; `normalizeFundId` at `:37-40`): `normalizeFundId` coerces any missing or unknown `fund_id` to **NPNAF**, and `normalizeSplitArray` defaults a missing split to `[{fund_id: NPNAF}]`. So **every passbook expense row carries a real fund id** (PF / AF / NPNAF / EF). `NPNAF` means **"Neither-PF-Nor-AF"** — collected/business funds — it is **not** an "unknown" bucket.

**Category map** (`src/services/teamPassbook/constants.ts:3-25`): `CATEGORY_MAP` at `:3-25`, `FUND_ID` constants at `:28-33`. `CATEGORY_MAP.REFUND` and `CATEGORY_MAP.REIMBURSEMENT` (`:16-21`) are excluded from spend, alongside the `refunds` table.

**Reimbursement legs** (`teamPassbookReimbursement.ts:434,449`): the payer's outflow leg uses `fund_id: EF` (`:434`); the receiver's inflow leg uses `fund_id: PF` (`:449`). `EF` is internal-only — never show it as an expense fund source.

**Open staff payback source** (`reimbursement.ts:21,405,435-442`): PF expense amount minus reimbursement allocations where `target_type = 'EXPENSE'` (never `'REFUND'`).

**Do not copy `propertyMonthlySummary`** (`propertyMonthlySummary.ts:243`): it sums expenses by `created_at`. DA-04 must use `paid_date`, not `created_at`.

**Expenses are created in-repo** (`addNewExpense → createNewExpense + registerExpense`, `controllers/expenses.ts:500-560`). The Firebase out-of-repo caveat that applies to some other domains does **not** apply to expenses.

## 4. How time works

### Selected period

An expense belongs to the selected period when its paid date is on or after the start date and on or before the end date.

Example:

| Selected period | What counts |
|---|---|
| 1 June to 30 June | Expenses paid from 1 June through 30 June |
| Today | Expenses paid today |
| This month | Expenses paid from the first day of this month through today or through the chosen end date |

### Previous period

The previous period is the same length immediately before the selected period.

Example:

| Current period | Previous period |
|---|---|
| 1 June to 30 June | 2 May to 31 May |
| 10 June to 16 June | 3 June to 9 June |

If previous spend is zero, show the rupee change but do not show a percent change. Percent from zero is not helpful.

### Expenses without paid date

Expenses without paid date:

- do not count in current spend,
- do not count in previous spend,
- appear under cleanup,
- open to a cleanup list instead of a dated expense list.

## 5. Base spend set

Every DA-04 spend number starts with the same set: live expenses for the selected property or properties, with a paid date inside the selected period.

Do not include:

- deleted expenses,
- refunds,
- reimbursements,
- expenses outside the selected paid-date period,
- expenses without paid date.

## 6. Top summary

| Number | What it means | Empty state |
|---|---|---|
| **Total spend** | Round of the sum of all live expenses in the selected period: `Math.round(SUM(amount))`. Round the **total**, not each row. Negative and zero rows are quarantined out (see below). | ₹0 spent in this period |
| **Expense count** | Number of live expense records in the selected period. | No expenses recorded in this period |
| **Average per expense** | Total spend divided by expense count. | Hide when count is 0 |
| **Change vs previous period** | Current spend minus previous-period spend. | Hide percent when previous spend is 0 |
| **Review gaps** | Expenses that are missing basic review data. | No review gaps found |
| **Open staff payback** | Staff personal money used for expenses and not yet paid back. | No staff payback open for this period |

> **Negative and zero amount rows are quarantined (owner decision, locked).** Exclude negative and zero `amount` rows from the headline total. Do not silently drop them: surface their **count and summed amount** in a small data-quality line so the owner can see the spend total was computed on clean rows only. They open into a separate audit list, not the main expense list.

## 7. Category grouping

The page uses fixed top groups first, then shows the real saved categories underneath Other.

| Group | What goes in it |
|---|---|
| Salary | Categories that start with Salary |
| Maintenance | Categories that start with Maintenance |
| Rent | Categories that start with Rent |
| Electricity | Categories that start with Electricity |
| Food / Mess | Categories that start with Food or Mess |
| Deposit Refunds | Categories that start with Deposit |
| Missing category | Expenses with no category saved |
| Other | Everything else |

> **Why "Deposit Refunds" (owner decision, locked).** In the live widget, `expense_type ILIKE 'Deposit%'` is labeled **"Deposit Refunds"** (`helpers.ts:118-122,292`): a deposit returned to a tenant, recorded as an expense row. These are deposit returns paid out, recorded as expenses — distinct from invoice-side refunds (DA-03).
>
> **No contradiction with the Security Deposit / Caution Money rule.** Locked decision #3 (`'Security Deposit'` / `'Caution Money'`) governs the **dues / invoice** domain via `due_type`, not `expense_type`. The `Deposit%` grouping here is purely on `expense_type`. The two never touch the same field, so a reviewer should not flag this as a conflict.

For each category group:

- category amount = sum of matching expense amounts
- category share = category amount / total spend
- category count = number of matching expenses

Other must not stay as a black box. It opens into saved category names sorted by amount.

If a category is tapped, the next list shows only the expenses that made that category number.

## 8. Paid-to and paid-by

### Paid-to

Paid-to answers: **who received the money?**

For each paid-to name:

- paid-to amount = sum of expenses paid to this name
- paid-to count = number of expenses paid to this name
- paid-to share = paid-to amount / total spend

Rules:

- trim extra spaces,
- blank names become Missing paid-to,
- do not merge different names unless they are exactly the same after trimming.

> **Compute the paid-to breakdown over this period's expenses, not from `getDistinctPaidTo`.** The existing `getDistinctPaidTo` helper (`helpers.ts:342-351`) has **no date range** — it returns all-time paid-to filter options. DA-04's paid-to amounts and counts must be a fresh aggregation scoped to the selected paid-date period, never a reuse of that all-time list.

### Paid-by

Paid-by answers: **who paid, or who was saved as the person who paid?**

For each paid-by name:

- paid-by amount = sum of expenses paid by this name
- paid-by count = number of expenses paid by this name
- paid-by share = paid-by amount / total spend

Rules:

- trim extra spaces,
- blank names become Missing paid-by,
- do not infer paid-by from creator role.

## 9. Property split

Property split answers: **which property caused the spend?**

For each property:

- property amount = sum of selected-period expenses for that property
- property count = number of selected-period expenses for that property
- property share = property amount / total spend

If the user selected one property, this section can collapse. If the user selected more than one property, it is required.

Each property row opens the expense list for that property and the same date period.

## 10. Fund source

Fund source answers: **whose money was used to pay the expense?**

There are three user-facing fund sources:

| Fund source | Meaning | Counts as spend? | Creates payback? |
|---|---|---|---|
| Collected funds | Business money already collected and used for spend. Pinned to code fund id `NPNAF`. | Yes | No |
| Petty cash | Money the owner/business had already given staff. Pinned to code fund id `AF`. | Yes | No, unless staff paid from personal money too |
| Personal funds | Staff used their own money. Pinned to code fund id `PF`. | Yes | Yes, until paid back |

> **Fund label map — confirm with eng.** These three UI labels are proposed mappings to the code fund ids: **Collected = NPNAF**, **Petty cash = AF**, **Personal = PF**. Confirm each label with engineering before launch. `NPNAF` means "Neither-PF-Nor-AF" (collected/business funds), not "unknown".

For each fund source:

- fund amount = expense amount paid from that fund source
- fund share = fund amount / total spend where fund source is known

> **"Fund source unavailable" is a missing passbook row, not a null fund id.** Every passbook expense row already carries a real fund id — the code coerces any missing or unknown `fund_id` to `NPNAF` (`normalizeFundId`, `constants.ts:37-40`), and a missing split defaults to `[{fund_id: NPNAF}]`. So there is no such thing as a null-fund-id row. The true "unavailable" case is an expense with **no `tm_transaction_expense_map` row at all** — the passbook fund fan-out failed silently when the expense was saved (see section 13 and the QA assertion in the Build Sheet). Such an expense exists with zero passbook rows.

For an expense with no passbook rows:

- keep it in total spend,
- show it under Fund source unavailable (defined as: zero `tm_transaction_expense_map` rows),
- do not guess the source from paid-by.

## 11. Open staff payback

When staff pay an expense from their own money, the business owes them back. **Open staff payback is how much of that is still unpaid.** This is also the most important correction in this refresh.

When staff pays an expense from personal funds, two things are true:

1. The business has spent money.
2. The business now owes staff money.

When staff is paid back, only the second number goes down. Total spend does not go up again.

Formula: personal-fund expense amount minus amount already paid back against those expense records = open staff payback.

For each staff member: open payback amount = personal-fund expenses paid by that staff member minus paybacks already made to that staff member for those expense records.

Rules:

- Count only paybacks tied to expense records.
- Do not subtract paybacks tied to refund records.
- Do not add reimbursement amount into total spend.
- If a staff-fronted expense is partly paid back, show only the remaining open amount.

## 12. Review gaps

Review gaps answer: **which expense records are weak for month-end review?**

| Gap | Rule |
|---|---|
| Missing bill | No bill or receipt is attached |
| Missing paid-to | No receiver/vendor/person is saved |
| Missing paid-by | No paid-by name is saved |
| Missing category | No category is saved |
| Missing paid date | No paid date is saved |
| Missing description | No description is saved |

For every gap:

- gap count = number of matching live expenses
- gap amount = sum of matching live expense amounts

Date rule:

- Missing bill, paid-to, paid-by, category, and description use the selected paid-date period.
- Missing paid date has no period. It uses the selected property scope and all live rows without paid date.

Each gap opens the matching cleanup list.

## 13. Deleted and reversed expenses

Deleted expenses are not live spend.

If the audit section ships, it answers: **what was removed or reversed during this period?**

Rules:

- show it as audit context,
- do not add it to total spend,
- do not subtract it from total spend. Only exception: when a live edited expense corrects its own fund source,
- label it clearly so the user does not read it as new spend.

> **Why an expense can exist with no passbook rows.** The passbook fund fan-out can fail silently. `teamPassbookExpenseService.ts:54-59` catches the error and rolls back the fund rows with `// throw error;` commented out — so the failed fan-out is swallowed and the expense row still saves. That is exactly how an expense ends up with zero `tm_transaction_expense_map` rows, which the fund-source view labels "Fund source unavailable" (section 10). Surface the count of such expenses as a coverage gap (see Build Sheet QA).

## 14. Refund boundary

Refunds and expenses are both money going out, but they answer different questions.

| Money movement | Belongs here? | Why |
|---|---|---|
| Salary expense | Yes | It is a recorded expense. |
| Maintenance expense | Yes | It is a recorded expense. |
| Deposit-like expense saved as an expense | Yes | It is stored as an expense record. |
| Tenant refund linked to an invoice | No | It is a refund, not an expense. |
| Staff reimbursement | No | It pays back staff for earlier personal-fund spend. |

Boundary:

- expense analytics total = recorded expenses only
- refund analytics total = tenant refunds only
- cash-flow outflow = expenses + refunds, handled by cash-flow reporting

This avoids double counting.

## 15. Empty states

| Situation | Message |
|---|---|
| No selected property | Select a property to view expenses. |
| User lacks expense access | You do not have permission to view expenses. |
| No expenses in period | No expenses recorded in this period. |
| No category data | No categories found for this period. |
| No paid-to data | No paid-to names found for this period. |
| No fund-source data | Fund source is unavailable for some expenses (expenses with no passbook rows). |
| No staff payback open | No staff personal-fund payback is open for this period. |
| Expense records missing fund source | Some expenses are missing fund-source data. Review before launch. |

## 16. Next-step behavior

Every amount must open the matching rows.

| User taps | Opens |
|---|---|
| Total spend | Expense list with the same property and period |
| Category group | Expense list filtered to that category group |
| Saved category under Other | Expense list filtered to that saved category |
| Paid-to name | Expense list filtered to that paid-to name |
| Paid-by name | Expense list filtered to that paid-by name |
| Property row | Expense list for that property and period |
| Missing bill | Cleanup list for expenses without bills |
| Missing paid date | Cleanup list for live expenses without paid date |
| Fund source | Expense rows that make up that fund-source amount |
| Open staff payback | Staff payback list for expense records only |

The opened rows must add back to the number the user tapped.

## 17. Edge cases

- **Negative expense amount:** quarantine it. Exclude it from the headline total and surface its count and summed amount in the data-quality line (owner decision, section 6).
- **Zero expense amount:** quarantine it too. Exclude it from the headline total and surface it in the same data-quality line.
- **Blank category:** group as Missing category, not Other.
- **Spelling variants:** use the saved category text. Do not guess fuzzy matches in V1.
- **One expense paid from multiple sources:** count the expense once in total spend; split only the fund-source view.
- **Deleted expense:** exclude from live spend.
- **Refund paid from staff personal money:** keep it out of DA-04 expense payback.
- **Fund source missing:** raw spend still works; fund-source and payback must label missing coverage.
- **Current-month shortcuts:** do not use shortcuts that ignore the user's selected dates.
