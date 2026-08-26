# DA-01 Dues, Hint Copy

The words behind every (i) on the Dues tab. Two columns per number, matching the two fields on the sheet: **What is this?** and **How is this calculated?** A dash means the definition already carries it.

Facts true of a whole block live once, not under every number: the block's face says whether it follows the date chips (the "Live" and "Forecast" labels), and where a block ignores the chips, the first item on its sheet says so once.

Checked against the code at rentok-backend `aed58caa8`. Every cell describes today's number; the last line names the three cells that change when known fixes land.

## The date chips

Five chips, the same five on every block that follows them.

| Chip | The window it sets |
|---|---|
| This Month | The 1st to today. Not the full month. |
| Last Month | The whole of last month. |
| Current FY | 1 April to 31 March, so it reaches into the future. |
| All Time | No limits. |
| Custom | The dates you pick. The end can be in the future. |

## 1. Overview

Six cards, each with its own window worked out from today. Ignores the date chips.

| Card | What is this? | How is this calculated? | Its window |
|---|---|---|---|
| All Time Dues | Every unpaid bill of tenants living here or booked, from any date. Dues of tenants who moved out are not counted. | The date chips above do not change this block. | All dates |
| All Past Dues | Unpaid bills of tenants living here or booked, due before this month. | - | Up to last month's end |
| {Month}'s Due | This month's unpaid bills, due between the 1st and today. | Bills due later this month sit in All Future, not here. | 1st to today |
| All Future Dues | Unpaid bills due after today, from tenants living here or booked. | - | After today |
| {Month}'s Projected Due | Rent expected this month that has not been billed yet. | Worked out from each tenant's rent amount and rent day. Only tenants on automatic monthly rent. | Tomorrow to month end |
| Current FY Dues | Unpaid bills due this financial year, from tenants living here or booked. | The year runs to 31 March, so bills not yet due are included. | 1 Apr to 31 Mar |

## 2. Dues (Live)

One total, four slices by due date. Ignores the date chips.

| Card | What is this? | How is this calculated? |
|---|---|---|
| Total Dues | Everything unpaid right now from tenants living here or booked, any due date. | The date chips above do not change this block. |
| Overdue | The part already past its due date. | - |
| Due Today | The part due today. | - |
| Due This Week | The part due in the next 7 days, starting tomorrow. | Today's bills are in Due Today, not here. |
| Due Later | The part due after the next 7 days. | - |

## 3. Bills Summary

Follows the date chips.

| Card | What is this? | How is this calculated? |
|---|---|---|
| Bill Due | All bills raised for these dates, paid or not. This is your total billing, not what is still pending. | Counted by the bill's due date, not the day the bill was made. |
| Received | Money received against those same bills, whenever it came in. | Payments from outside these dates count too. Bills cleared from a tenant's deposit or advance also count. |

## 4. Dues Breakdown

Follows the date chips. Three tabs over the period's unpaid dues.

| Tab | What is this? | How is this calculated? |
|---|---|---|
| Category | The period's unpaid dues split by bill type: rent, electricity, deposit, and the rest. | Top 5 types shown; the rest group into Others. |
| Tenant Status | Unpaid dues split by where the tenant stands: living here, under notice, booked, or moved out. | The only view that counts moved-out tenants, so its total runs higher than the other two tabs. |
| Added By | The same dues split by who raised the bill. | By role, not by person: Owner, RentOk, Partner or Tenant. |

## 5. Overdue Breakup

Ignores the date chips.

| Tab | What is this? | How is this calculated? |
|---|---|---|
| By Amount | Overdue dues grouped by how late they are, counted from due date to today. | The date chips do not change this block. |
| By Category | The same overdue money, grouped by bill type instead. | - |

## 6. Upcoming Dues

Forecast. Ignores the date chips.

| Card | What is this? | How is this calculated? |
|---|---|---|
| Rent | Rent expected between tomorrow and month end, not billed yet. | From each tenant's rent day and amount. Rent already billed is left out, so nothing is counted twice. |

## 7. Deposit Dues

Ignores the date chips.

| Card | What is this? | How is this calculated? |
|---|---|---|
| Total Deposits | Received plus Due: all deposit money, held and pending. | The date chips do not change this block. |
| Received | Deposit money collected and still with you, from tenants living here or booked. | Refunds and deposit already used to clear bills are taken off. |
| Due | Deposit billed but not yet paid, from tenants living here or booked. | - |

## 8. Breakup by Stay Duration

Follows the date chips.

| Card | What is this? | How is this calculated? |
|---|---|---|
| Short Term | The period's unpaid dues from tenants marked short stay. | The short stay mark sits on the tenant's profile, not on the bill. |
| Long Term | Unpaid dues from everyone else. | Any tenant not marked short stay counts here. |

## 9. Dues by Property

Follows the date chips.

| Card | What is this? | How is this calculated? |
|---|---|---|
| Dues by property | Each property's unpaid dues from tenants living here or booked, highest first. | Share is that property's part of your total. |

---

Three cells change when known fixes land: Current FY Dues (the window moves to today), {Month}'s Projected Due (bookings join the count), Bills Summary Received (deposit-cleared bills leave). All tracked in the build verification log, findings F1, F2 and F8.
