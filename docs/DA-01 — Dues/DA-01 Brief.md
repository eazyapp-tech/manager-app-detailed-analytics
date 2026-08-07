---
title: DA-01 Dues — Brief
date: 2026-06-06
tags: [rentok, brief, dues, detailed-analytics]
status: Living document · v3.0
owner: Sanchay
time_budget: Open — cut order ranked by owner pain
---

> [!INFO] What this is
> A product brief for the Dues detailed-analytics screen.
>
> It explains what the owner needs to understand before he starts chasing unpaid money. It is not a build spec and not a formula sheet.

## 1. In one line

A PG owner can see that money is unpaid, but not the **breakdown** of that unpaid money — what kind, how old, from whom.

He needs to know what is old, what belongs to this month, and what is still ahead. He also needs to know what type of dues is stuck, who added it, what stage the tenant is at (still living there, on notice, evicted, booked, or moved out), and which property is carrying the load.

So the bet is one thing: **make unpaid dues readable as one money picture before the chase begins.**

*"Kitna paisa atka hai, kis type ka hai, kis stage mein hai, aur pehle kahan jaana hai?"*

## 2. The owner

This screen is for the owner when he wants to **diagnose dues**.

It is not for editing one bill. It is not for marking one payment. It is not for calling one tenant. Those actions happen after the owner understands the shape of the problem.

**Rajesh, primary user.** Owner of one PG. He opens this when the dues number feels high and he wants to know where to start.

**Priya, secondary user.** Multi-property owner. Her question is: *"Which property is carrying the unpaid money, and what kind of unpaid money is it?"*

**Meena, secondary user.** Restricted manager. She should either see the data she is allowed to act on, or a clear restricted state. The screen should never look broken.

These personas are built from real RentOk owners and team knowledge, not from new interviews.

Where this sits:

- the main screen says **money is unpaid**
- the dues list says **which rows match a filter**
- this screen says **what shape the unpaid-money problem has**

## 3. The problem

Three pains, same root cause:

1. **One number does not tell the owner where the problem is.** Unpaid rent, unpaid deposit, and unpaid old-tenant money are not the same problem.
2. **The owner cannot pick the first follow-up lane quickly.** Fresh unpaid dues, 22+ day dues, booking dues, and confirmed-eviction dues need different action.
3. **The owner cannot hand work to staff cleanly.** "Recover dues" is too broad. "Call 22+ day rent dues from confirmed-eviction tenants" is work.

The screen should turn a vague money pile into clear groups the owner can chase one by one.

## 4. What Rajesh does today

He works through partial views:

- reads one unpaid-dues number
- opens the dues list
- tries filters one by one
- mentally groups bills by type
- asks staff who added what
- guesses which tenant stage deserves attention

This is slow and easy to misread. The new screen should show the whole unpaid-money picture first, then hand the owner to the right filtered list.

## 5. What must ship

**Must ship. V1 is not V1 without these:**

- **Q1 — What is unpaid right now?** All unpaid dues, past dues, current-month dues, future dues, and current-year dues.
- **Q2 — What type of dues is stuck?** The top 4 due categories by unpaid amount, plus Others.
- **Q3 — Which tenant journey state is risky?** Active Tenants, Notice Raised, Confirmed Eviction, Bookings, and Old Tenants.
- **Q4 — Who added the dues load?** RentOk/system, top people or actors, and Others when the list is long.
- **Q5 — How old is the unpaid money?** Ageing buckets: 1-7, 8-14, 15-21, and 22+ days.
- **Q6 — Which property is carrying it?** Property-wise unpaid dues for multi-property owners.

These six answer: *how much → what type → which journey state → who added → how stale → where.*

**Good to ship if the cycle allows:**

- **Projected Fixed Dues.** Fixed recurring dues RentOk will add later this month. Usually rent. Some properties also have recurring food or other fixed dues.
- **Short-stay vs long-stay dues.** Useful when short-stay tenants create a different recovery pattern.
- **Deposit dues view.** Useful because deposit dues need a different follow-up than rent.
- **Other members sheet.** Useful detail for the Added By split.

**Cut order if time runs short:**

1. Other members sheet — the owner can still act from the main Added By rows.
2. Deposit dues view — deposit dues still appear in Category and Top Summary cuts.
3. Short-stay vs long-stay dues — useful, but the owner can still chase by category, age, journey state, and property.
4. Projected Fixed Dues — helpful for planning, but the owner can still recover existing unpaid dues without it.

Do not cut Q1 to Q6 before all four above are gone.

## 6. Product decisions

**All Dues means all unpaid dues.** Paid money is not part of this screen. When a tenant pays part of a bill, the remaining balance still shows — the system reduces that bill to what is left owed, so it stays in the picture.

**Future Dues and Projected Fixed Dues are different.** Future Dues are unpaid bills already created with future due dates. Projected Fixed Dues are fixed recurring bills expected to be created later.

**Top summary numbers keep their own time meaning.** They do not change just because the owner changes the duration for deeper views.

**Breakdowns use the selected duration.** Category, Tenant Journey, Added By, Ageing, and Property views answer the duration the owner picked.

**Category is top 4 + Others.** The category names are not fixed. The screen shows the four biggest unpaid categories for the selected duration. Everything else rolls into Others.

**Added By is RentOk/system + top actors + Others.** RentOk/system should stay visible when it has unpaid amount, because system-created dues are a different story from staff-created dues. After that, show the biggest people or actors and roll the rest into Others.

**Tenant Status follows the Journey model.** This is a journey-risk view, not a strict "parts of total" view.

- Active Tenants = living tenants with unpaid dues
- Notice Raised = active tenants where notice has been raised but the move-out date is not confirmed
- Confirmed Eviction = active tenants with a confirmed move-out date
- Bookings = booked tenants with unpaid dues
- Old Tenants = moved-out tenants who still owe money

Notice Raised and Confirmed Eviction are risk slices inside the active-tenant journey. They are shown because the owner acts on them differently.

**Ageing uses 1-7, 8-14, 15-21, and 22+ days.** Once unpaid money crosses three weeks, the owner needs escalation more than finer labels.

## 7. What we are not building this cycle

- **No recovery work here.** Calls, reminders, edits, and payment marking stay in the dues list and tenant pages.
- **No fake fixed category list.** Rent and electricity may appear often, but the category view is driven by real unpaid amount.
- **No paid-dues reporting.** This screen stays about unpaid money.
- **No hidden mix of deposit dues and deposits held.** Deposit dues are unpaid. Deposits held are already collected.
- **No duplicate dues list.** Every follow-up opens the existing dues list or tenant page.

## 8. Traps and risks

**Traps decided now:**

- **A fixed category view would be wrong.** The biggest unpaid categories can change by property and duration.
- **Journey-risk dues must be visible without lying about the sum.** Notice Raised and Confirmed Eviction sit inside the active-tenant journey. Show them clearly, but do not make the owner think every journey row adds to one total.
- **Old-tenant dues belong in the picture.** If moved-out tenants are missing, the owner sees a cleaner number than reality.
- **Projected Fixed Dues must be trustworthy.** If the system cannot explain what will be created later, cut it.
- **Every follow-up must match the number tapped.** If a row says ₹X and the list shows a different slice, trust breaks.

**Risks:**

| Risk | Read | Mitigation |
|---|---|---|
| Owners misunderstand duration | Medium | Keep top summary numbers fixed and make duration-driven views clearly labelled. |
| Categories feel unstable | Medium | Explain that rows show the biggest unpaid categories for the selected duration. |
| Projected Fixed Dues feels like a guess | High | Ship only after the recurring-dues rule is clear. |
| Old-tenant money is hidden | High | Make old-tenant dues part of the main unpaid-money story. |

### Footer

**Things to test with owners before launch:**

- Do they understand that category rows can change because they show the biggest unpaid categories?
- Do Notice Raised and Confirmed Eviction help them act faster?
- Does 22+ days feel like the right escalation bucket?
- Does Projected Fixed Dues read clearly when it includes recurring food or other fixed dues?

**Locked decisions:**

- All Dues = all unpaid dues.
- Category = top 4 + Others.
- Added By = RentOk/system plus top actors plus Others.
- Ageing = 1-7, 8-14, 15-21, 22+.
- Tenant Status = Journey model.
- Future Dues and Projected Fixed Dues are separate.

## Changelog

| Date | Version | Change |
|---|---|---|
| 2026-06-06 | v3.0 | Rebuilt as a self-contained product brief. Locked unpaid-dues meaning, top 4 + Others, Journey-style tenant status, 22+ ageing, and Projected Fixed Dues wording. |
| 2026-06-06 | v3.1 | Schema-grounding pass. Plain-language glosses (breakdown/stage/groups). Confirmed partial payments reduce the bill in place and stay in the unpaid picture (no exclusion). |
