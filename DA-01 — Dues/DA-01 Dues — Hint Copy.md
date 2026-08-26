# DA-01 Dues, Hint Copy

The words behind every (i) on the Dues tab. Two columns, matching the two fields on the sheet: **What is this?** and **How is this calculated?** A dash means the definition already says everything.

The sheet shows a title and a list of items, nothing else, so there is no block-level line. Every fact a card needs is inside that card's own two fields. Where a fact belongs to the whole block, it rides on the block's first item.

Checked against the code at rentok-backend `aed58caa8`.

## The date filter

| Option | The dates it sets |
|---|---|
| This Month | The 1st to today. Not the full month. |
| Last Month | The whole of last month. |
| Current FY | 1 April to 31 March, so it reaches into the future. |
| All Time | No limits. |
| Custom | The dates you pick. The end can be in the future. |

## 1. Overview

Six cards, each with its own window. Does not follow the date filter.

| Card | What is this? | How is this calculated? | Its window |
|---|---|---|---|
| All Time Dues | Every unpaid bill of tenants living here or booked, from any date. Dues of tenants who moved out are not counted. | The dates you pick at the top do not change this block. | All dates |
| All Past Dues | Unpaid bills due before this month started. | - | Up to last month's end |
| {Month}'s Due | Unpaid bills due from the 1st of this month up to today. | Bills due later this month are in All Future, not here. | 1st to today |
| All Future Dues | Unpaid bills due after today. | - | After today |
| {Month}'s Projected Due | Rent expected this month that has not been billed yet. | Worked out from each tenant's rent amount and rent day. Counts tenants whose rent is added automatically every month. | Tomorrow to month end |
| Current FY Dues | Unpaid bills due this financial year. | The year runs to 31 March, so bills not due yet are counted too. | 1 Apr to 31 Mar |

## 2. Dues (Live)

One total split four ways by due date. Does not follow the date filter.

| Card | What is this? | How is this calculated? |
|---|---|---|
| Total Dues | Everything unpaid right now from tenants living here or booked, whatever the due date. | The dates you pick at the top do not change this block. |
| Overdue | Bills already past their due date. | - |
| Due Today | Bills due today. | - |
| Due This Week | Bills due in the next 7 days, starting tomorrow. | Today's bills are in Due Today, not here. |
| Due Later | Bills due after the next 7 days. | - |

## 3. Bills Summary

| Card | What is this? | How is this calculated? |
|---|---|---|
| Bill Due | All bills raised for the dates you pick, from tenants living here or booked, paid or not. This is your total billing, not what is still pending. | Counted by the bill's due date, not the day the bill was made. |
| Received | Money received against those same bills, whenever it came in. | Payments made outside these dates count too. Bills cleared from a tenant's deposit or advance also count. |

## 4. Dues Breakdown

| Tab | What is this? | How is this calculated? |
|---|---|---|
| Category | Unpaid dues for the dates you pick, from tenants living here or booked, split by bill type. | The top 5 types are shown, the rest are grouped as Others. |
| Tenant Status | Unpaid dues split by tenant status: living here, under notice, booked, or moved out. | The only view that counts tenants who moved out, so its total runs higher than the other two tabs. |
| Added By | Unpaid dues split by who raised the bill. | By role, not by person: Owner, RentOk, Partner or Tenant. |

## 5. Overdue Breakup

| Tab | What is this? | How is this calculated? |
|---|---|---|
| By Amount | Overdue bills grouped by how late they are. | Counted from the due date to today. The dates you pick at the top do not change this block. |
| By Category | Overdue bills grouped by bill type. | - |

## 6. Upcoming Dues

| Card | What is this? | How is this calculated? |
|---|---|---|
| Rent | Rent expected between tomorrow and month end, not billed yet. | Worked out from each tenant's rent amount and rent day. Rent already billed is left out, so nothing is counted twice. |

## 7. Deposit Dues

| Card | What is this? | How is this calculated? |
|---|---|---|
| Total Deposits | All deposit money of tenants living here or booked: what is collected plus what is still unpaid. | The dates you pick at the top do not change this block. |
| Received | Deposit collected from tenants living here or booked, and still with you. | Refunds, and deposit already used to clear bills, are taken off. |
| Due | Deposit billed but not yet paid. | - |

## 8. Breakup by Stay Duration

| Card | What is this? | How is this calculated? |
|---|---|---|
| Short Term | Unpaid dues for the dates you pick, from tenants marked short stay. | The short stay mark sits on the tenant's profile, not on the bill. |
| Long Term | Unpaid dues from tenants not marked short stay. | - |

## 9. Dues by Property

| Card | What is this? | How is this calculated? |
|---|---|---|
| Dues by property | Unpaid dues per property for the dates you pick, highest first. Counts tenants living here and booked. | Share is that property's part of your total. |

---

**One build ask.** The sheet has no block-level line, so a fact about a whole block has to ride on its first item. That works, but it means the population line can only be said once per block. If the sheet's title area could carry one optional line, every card would read shorter.

**Three cells change when known fixes land:** Current FY Dues (the window moves to today), {Month}'s Projected Due (bookings join the count), Bills Summary Received (deposit-cleared bills leave). Findings F1, F2 and F8 in the build verification log.
