---
title: DA-04 Expenses — Brief
date: 2026-06-06
tags: [rentok, brief, expenses, financials, detailed-analytics]
status: Living document · refreshed from codebase grounding
owner: Sanchay
time_budget: Ship all core questions; cut only by owner pain
companion_to: DA-04 Ground-Truth Formula Map · DA-04 Build Sheet
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
> A **Brief** for the Expenses analytics screen. It says what the owner needs to understand about spending before the team decides how to show it.
>
> **What this is NOT:** an engineering spec or a design doc. The exact formulas live in the Formula Map. Build tasks live in the Build Sheet.

# DA-04 Expenses — Brief

## 1. In one line

A property owner records expenses, but still cannot answer the simple question: **where did the money go, who paid it, and what still has to be paid back to staff?**

This screen makes spending readable in one place: total spend for the selected period, category mix, vendor/person paid, person who paid, property split, bill gaps, and personal-fund exposure. It is a spending diagnosis page, not an expense entry page and not a cash-flow profit page.

*"Kitna kharcha hua, kis cheez par hua, kisne pay kiya, aur staff ko kitna wapas dena hai?"*

## 2. Who this is for

This screen is for the owner or finance admin when they want to understand spending. It is not for adding an expense, approving a pending task, or reading a full cash-flow statement.

**Rajesh, primary user.** He owns one or a few PGs. He opens this near month-end, after a large repair or vendor bill, or when the homescreen expenses number looks high. He wants to know whether the month is normal, which category moved, which vendor/person took the money, and whether staff used personal funds.

**Priya, secondary user.** She manages several properties. Her first question is which property is driving the spend. Her second question is whether that property has a real reason: salary, maintenance, food, electricity, rent, deposit-like payout, or many small "other" entries.

> **Deposit-like** here means a refund-shaped payout that was recorded as an expense row — money returned to a tenant (often a security deposit), but logged on the expense side, not the invoice side. This term recurs across all three DA-04 docs; this is the shared definition.

**Meena or another staff member, secondary user.** Staff may record or pay expenses. Staff see only the expenses their access already allows. The owner must not see a total that opens rows a staff member cannot.

Where it sits: the homescreen says the selected period has expense outflow; the expense list shows individual rows; Team Passbook shows fund movement by team member. This page says what shape the spending took and points the user into the right list or passbook detail.

Personas are product composites from the RentOk owner/admin base.

## 3. The problem

Three pains, same root cause — expense records exist, but the owner still has to assemble the spending picture himself:

1. **"I know money went out, but I don't know why it went out."** The list has rows. It does not answer whether salary, maintenance, food, electricity, rent, deposit-like payout, or small other entries caused the spend.
2. **"I don't know who is carrying the money."** Some expenses are paid from collected funds, some from petty cash, and some from staff personal funds. A personal-fund expense is not just spending; it is money the business owes back to staff.
3. **"I can't tell whether this is clean enough for review."** Missing bills, vague payee names, free-text categories, deleted/reversed rows, and deposit-like entries can make the number look right while the detail is not review-ready.

The bet is not to reduce expenses. The bet is to make spending explainable, and to show when spend has created a staff payback obligation.

**Why now:** Expenses already appear in the homescreen financial view and the expense list. The missing step is the page between them: the owner sees a number, then needs the shape before acting. Without it, high spend becomes a list-scrolling exercise.

## 4. What users do today

- **Scroll the expense list.** Works for a single vendor lookup. Fails when the question is "what changed this month?"
- **Use the homescreen breakdown.** It shows Maintenance, Salary, and Other, but not the richer shape needed for review.
- **Ask staff what they paid.** Gives a chat answer, not a tied number. Personal-fund spend can stay hidden until reimbursement time.
- **Pull reports or Excel.** Useful for accounting, but late and heavier than the in-app decision.
- **Open Team Passbook separately.** Shows fund movement, but the owner has to connect passbook rows back to expense categories and vendors.

The screen has to beat all five: faster than report export, more complete than the homescreen, more grouped than the list, clearer than staff chat, and less scattered than switching to passbook.

## 5. What we must ship, and what we cut if time runs short

The goal is to ship every question below. The tiers are for controlled cuts if the build slips. They are not the planned scope.

**Must ship:**

1. **How much was spent in the selected period?** Count only active, recorded expense rows governed by expense paid date.
2. **Where did the spend go?** Show the category mix from stored expense categories, with a fixed top group for Salary, Maintenance, Rent, Electricity, Food/Mess, Deposit-like, and a data-driven Other group.
3. **Who received the money?** Show payee/vendor/person names from the expense paid-to field.
4. **Who paid it?** Show payer names from the expense paid-by field.
5. **Which property caused the spend?** Needed for multi-property owners.
6. **How much spend used staff personal funds and remains open for reimbursement?** This is the key correction from the old DA-04 framing. Staff-fronted spend is both expense and owner liability.

If these six do not ship, the page does not answer the core question.

**Nice to have:**

7. **Which entries are weak for review?** Missing bills, blank payee, blank category, blank paid date, and vague description.
8. **What changed versus the prior comparable period?** Show spend movement by total and top categories when the prior period is valid.
9. **Which deleted or reversed expense events affected trust?** Show as audit context only, not live spend.

**Cut in this order if forced, ranked by user pain:**

1. **Deleted/reversed audit summary.** Useful for trust checks, but not needed for the main spending picture.
2. **Prior-period movement.** Helpful for diagnosis, but the user can still read the current shape without it.
3. **Review weakness count.** Important at month-end, but secondary to seeing amount, category, payee, payer, property, and personal-fund exposure.
4. **Property split.** Cut only for a single-property launch. Keep it for multi-property accounts.

Do not cut personal-fund exposure before the items above. It changes what the owner owes, not just what he spent.

## 6. What does each question need to know?

Each line describes information, not layout.

1. **Total spend.** Sum of active expense amounts in the selected date range.
2. **Category mix.** Amount and share by expense category group.
3. **Top paid-to.** Amount and count by paid-to name.
4. **Top paid-by.** Amount and count by payer name.
5. **Property split.** Amount and share by property.
6. **Fund source.** Amount by collected funds, petty cash, and personal funds where Team Passbook rows exist.
7. **Open personal-fund payback.** Personal-fund amount minus reimbursement allocations for expense targets.
8. **Review gaps.** Counts and amounts for rows missing bill, paid-to, category, paid date, or description.
9. **Period movement.** Current period versus previous equal-length period for total and top categories.
10. **Clean next step.** Every number must open the matching filtered expense list or passbook detail, with the same permissions.

## 7. What we are not building this cycle

- No new expense approval queue. The code does not support an expense approval state that can safely drive analytics. A homepage task uses the phrase "Pending Expenses" as a reminder to log expenses; that is not a stored pending-expense state.
- No profit statement. Cash flow already subtracts refunds and expenses in its own report. DA-04 explains expenses. Profit belongs to cash flow.
- No reclassification engine for bad categories. We show the stored categories and flag category quality. We do not silently recode spend.
- No reimbursement creation from this screen. The page can point to the reimbursement path. It must not turn a diagnosis page into a money-movement form.
- No separate refund analytics inside DA-04. Invoice-linked refunds are their own domain. Deposit-like expense categories are included only because they are stored as expense rows.

## 8. Traps and risks

**Traps decided in advance:**

- **USER — "Pending expenses" is not a real expense state.** The product currently has expense reminders, not a dependable pending-expense queue. Decision: DA-04 does not show a pending-expense headline.
- **USER — paid date controls the page.** DA-04 uses the date the money was paid for every period number. Rows without a paid date become review gaps, not period spend.
- **USER — refunds must not be double-counted as expenses.** Real tenant refunds stay out of DA-04. Deposit-like expense categories are included only when they were recorded as expense rows.
- **USER — reimbursement is not new spend.** Reimbursement pays staff back for money they fronted. Decision: show it as payback status and owner liability, not as added expense.
- **TEAM — fund splits need passbook history.** Expense records alone do not fully explain whether money came from collected funds, petty cash, or staff personal funds. Decision: fund-source and open-payback sections use the passbook history.
- **TEAM — deleted expenses are not live spend.** Deleted expenses do not count in the headline. Reversal history belongs in audit context, not the main total.

| Risk | Read | Mitigation |
|---|---|---|
| Will users use it? | HIGH | Expenses already sit on the homescreen and list. This page fills the missing "why did spend move?" step. |
| Will they understand it? | MED | Keep spend, fund source, and reimbursement clearly separated. Do not label payback as extra spend. |
| Can we build it? | MED | Raw spend is available now. Fund-source and open PF need passbook joins and performance checks. |
| Is it worth it? | STRONG | It closes a trust gap in a money screen: spending shape plus staff payback. |

**Stop and reconsider mid-build if:** fund-source joins are too slow for multi-property accounts, or if real data shows many expenses without Team Passbook rows. In that case, ship raw spend first and label fund-source coverage honestly.

## Scrub log

- Replaced old "profitability snapshot" framing with expense-only diagnosis. Profit belongs to cash flow.
- Removed reliance on the old `status` column as pending behavior.
- Replaced fixed top-category-only thinking with fixed-plus-data-driven grouping.
- Reframed reimbursements from expense totals to owner payback status.
- Removed design references and historical doc dependencies.
