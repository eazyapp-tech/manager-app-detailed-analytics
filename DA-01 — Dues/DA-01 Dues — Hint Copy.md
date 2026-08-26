# DA-01 Dues, Hint Copy

The words behind every (i) on the Dues tab. Two columns, matching the two text fields the sheet has: **What is this?** and **How is this calculated?** A dash means the definition already says everything.

**Good to know** is a third line, one per block, that answers what she asks after she understands the number: how these numbers relate, and why one will not match another. The sheet has no field for it yet. See the build ask at the bottom.

Checked against the code at rentok-backend `0e8cc713b` and against the live account, Mumbai group, 22 properties, on 27 Aug.

## The date filter

| Option | The dates it sets |
|---|---|
| This Month | The 1st to today. Not the full month. |
| Last Month | The whole of last month. |
| Current FY | 1 April to 31 March, so it reaches into the future. |
| All Time | No limits. |
| Custom | The dates you pick. The end can be in the future. |

## 1. Overview

Six tiles, each with its own window. Does not follow the date filter.

> **Good to know:** All Past, this month's and All Future add up to All Time Dues. Projected is not in that total, because it is not billed yet.

| Tile | What is this? | How is this calculated? | Its window |
|---|---|---|---|
| All Time Dues | Every unpaid bill of tenants living here or booked, from any date. Dues of tenants who moved out are not counted. | The dates you pick at the top do not change this block. | All dates |
| All Past Dues | Unpaid bills due before this month started. | - | Up to last month's end |
| {Month}'s Due | Unpaid bills due from the 1st of this month up to today. | Bills due later this month are in All Future, not here. | 1st to today |
| All Future Dues | Unpaid bills due after today. | - | After today |
| {Month}'s Projected Due | Rent and other recurring bills expected this month that have not been raised yet. | Rent of tenants on automatic monthly rent, plus scheduled packages like food and services. A tenant with no rent day set is counted on their joining day. | Tomorrow to month end |
| Current FY Dues | Unpaid bills due this financial year. | The year runs to 31 March, so bills not due yet are counted too. | 1 Apr to 31 Mar |

## 2. Dues (Live)

One total split four ways by due date. Does not follow the date filter.

> **Good to know:** the four parts add up to Total Dues, and Total Dues is the same number as All Time Dues above.

| Card | What is this? | How is this calculated? |
|---|---|---|
| Total Dues | Everything unpaid right now from tenants living here or booked, whatever the due date. | The dates you pick at the top do not change this block. |
| Overdue | Bills already past their due date. | - |
| Due Today | Bills due today. | - |
| Due This Week | Bills due in the next 7 days, starting tomorrow. | Today's bills are in Due Today, not here. |
| Due Later | Bills due after the next 7 days. | - |

## 3. Bills Summary

> **Good to know:** the gap between the two is what is still to collect from these bills.

| Card | What is this? | How is this calculated? |
|---|---|---|
| Bill Due | All bills raised for the dates you pick, from tenants living here or booked, paid or not. This is your total billing, not what is still pending. | Counted by the bill's due date, not the day the bill was made. |
| Received | Money received against those same bills, whenever it came in. | Payments made outside these dates count too. Bills cleared from a tenant's deposit or advance also count. |

## 4. Dues Breakdown

> **Good to know:** Category and Added By add up to the same total. Tenant Status runs higher, because it is the only view that counts tenants who moved out.

| Tab | What is this? | How is this calculated? |
|---|---|---|
| Category | Unpaid dues for the dates you pick, from tenants living here or booked, split by bill type. | The top 5 types are shown, the rest are grouped as Others. |
| Tenant Status | Unpaid dues split by tenant status: living here, under notice, booked, or moved out. | The only view that counts tenants who moved out. |
| Added By | Unpaid dues split by who raised the bill. | By role, not by person: Owner, RentOk, Partner, Customer or Tenant. |

## 5. Overdue Breakup

> **Good to know:** both tabs show the same overdue money, split two different ways, so both add up to Overdue in Dues (Live).

| Tab | What is this? | How is this calculated? |
|---|---|---|
| By Amount | Overdue bills grouped by how late they are. | Counted from the due date to today. The dates you pick at the top do not change this block. |
| By Category | Overdue bills grouped by bill type. | - |

## 6. Upcoming Dues

> **Good to know:** only Rent shows here today. Food and other recurring bills appear once they are set up.

| Card | What is this? | How is this calculated? |
|---|---|---|
| Rent | Rent expected between tomorrow and month end, not billed yet. | From each tenant's rent amount and rent day; if no rent day is set, their joining day is used. Rent already billed is left out, so nothing is counted twice. |

## 7. Deposit Dues (As of today)

> **Good to know:** Available plus Due is the Total. Available is the money you could return today.

| Card | What is this? | How is this calculated? |
|---|---|---|
| Total Deposits | All deposit money of tenants living here or booked: what is collected plus what is still unpaid. | The dates you pick at the top do not change this block. |
| Available | Deposit collected from tenants living here or booked, and still with you. | Refunds, and deposit already used to clear bills, are taken off. |
| Due | Deposit billed but not yet paid. | - |

## 8. Breakup by Stay Duration

> **Good to know:** the two sides add up to the same dues the Category tab shows for these dates. Both sides always show, even at ₹0.

| Card | What is this? | How is this calculated? |
|---|---|---|
| Short Term | Unpaid dues for the dates you pick, from tenants marked short stay. | The short stay mark sits on the tenant's profile, not on the bill. |
| Long Term | Unpaid dues from tenants not marked short stay. | - |

## 9. Dues by Property

> **Good to know:** this needs two or more properties. All the properties together add up to the dues for these dates.

| Card | What is this? | How is this calculated? |
|---|---|---|
| Dues by property | Unpaid dues per property for the dates you pick, highest first. Counts tenants living here and booked. | - |

---

## The pointers are checked, not guessed

Read off the live account on 27 Aug, This Month selected. Every "Good to know" above holds to the rupee.

| Claim | Checked |
|---|---|
| Overview's three windows add to All Time | ₹35.24L + ₹13.50L + ₹15.94L = ₹64.68L against ₹64.69L shown |
| Live total equals All Time Dues | ₹64.69L on both |
| Live's four parts add to its total | 48,40,581 + 34,033 + 11,87,117 + 4,07,040 = ₹64.69L |
| The Bills Summary gap is what is still to collect | ₹2.18Cr billed less ₹2.04Cr received is ₹14L, against Aug's Due of ₹13.50L |
| Category adds to the period's dues | six bars total ₹13.50L, matching Aug's Due |
| Stay split adds to the same | ₹5.81K + ₹13.44L = ₹13.45L |
| Available plus Due is Total Deposits | ₹56.72L + ₹35.83L = ₹92.55L |

## Build asks, in order of size

**1. The Overview tiles have hint copy that nobody can reach.** The block ships six hint sentences from the API, but the card header carries "View all" and no (i). Six of the tab's twenty-five definitions are unreachable, including All Time Dues, the largest number on the screen.

**2. The sheet needs one block-level line.** Today it is a title plus a list of items. Both the block facts and every "Good to know" above need that one line. Without it they have to ride on the first item, where they read as part of that item's own calculation.

**3. The change chips are green when dues go up.** Aug's Due showed a green 256% rise and Current FY a green 15% rise on the live account. On dues, up is bad. This is finding F6 and it is visible on the first screen an owner opens.

**4. "Bills Created" is the wrong rename.** The label was changed from Bill Due to Bills Created, but the window still runs on the bill's **due date**. A bill created in July for August rent lands in August. The old name was wrong about what the number counts; the new one is wrong about when. Either the label goes neutral or the query moves to the creation date. The copy above still says Bill Due, which is what the live build renders.

**Three cells change when known fixes land:** Current FY Dues (the window moves to today), {Month}'s Projected Due (bookings join the count), Bills Summary Received (deposit-cleared bills leave). Findings F1, F2 and F8.
