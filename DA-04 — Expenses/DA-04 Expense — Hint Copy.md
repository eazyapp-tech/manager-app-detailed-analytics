# DA-04 Expense, Hint Copy

The words behind every (i) on the Expense tab. Two columns, matching the two text fields the sheet has: **What is this?** and **How is this calculated?** A dash means the definition already says everything. **Good to know** is one line per block, for the block-level line when the sheet gets one.

Checked against the code at rentok-backend `0e8cc713b` and the live account, Mumbai group, 27 Aug.

## The date filter

Almost the same five options as Dues and Collection, with one difference that matters: **Custom cannot go past today.** Money cannot be spent in the future, and this is the one tab that enforces it.

| Option | The dates it sets |
|---|---|
| This Month | The 1st to today. Not the full month. |
| Last Month | The whole of last month. |
| Current FY | 1 April to today. |
| All Time | Everything up to today. |
| Custom | The dates you pick, capped at today. |

Every number on this tab is counted by the day the expense was paid.

## 1. Overview

Four tiles, three different clocks.

> **Good to know:** Total and Number of expenses follow the dates you pick. Current FY is 1 April to today. Still owed to staff is right now.

| Tile | What is this? | How is this calculated? |
|---|---|---|
| Total Expense | Money spent in the dates you pick. | Corrections, entries at zero or below, are not counted. |
| Current FY Expense | Money spent since 1 April. | - |
| Still owed to staff | Money the business still owes staff who paid from their own pocket. | What staff fronted minus what has been paid back. The dates you pick do not change this. |
| Number of expenses | How many expense entries in these dates. | Corrections, entries at zero or below, are not counted. |

## 2. Expense Breakdown

> **Good to know:** both tabs split the same spend, so each adds up to the same total.

| Tab | What is this? | How is this calculated? |
|---|---|---|
| Category | Spend for these dates split by expense type: salary, maintenance, and the rest. | The top few are shown, the rest are grouped as Others. |
| Payment Mode | Spend split by how it was paid: cash, UPI, bank, FlexiPe. | An entry with no recorded mode shows as Online. |

## 3. Top Payers & Vendors

> **Good to know:** the same spend twice, once by who paid it, once by who received it.

| Tab | What is this? | How is this calculated? |
|---|---|---|
| Paid by | Spend grouped by who paid it. | - |
| Paid to | Spend grouped by who received it, vendor or payee. | - |

## 4. Expense Trend

> **Good to know:** uses its own months dropdown, not the date filter above.

| Line | What is this? | How is this calculated? |
|---|---|---|
| Monthly | Total spend in each month. | The lighter bar is the current month, still filling in. |

## 5. Expenses by Property

> **Good to know:** the properties together add up to Total Expense for the same dates.

| Card | What is this? | How is this calculated? |
|---|---|---|
| Expenses by property | Spend per property for these dates, highest first. | Share is that property's part of the total. |

---

## The pointers are checked, not guessed

The live account holds almost no expense data, so most checks here are code-side rather than arithmetic on screen. What the screen could confirm, it did.

| Claim | Checked |
|---|---|
| Current FY is April to today, not to 31 March | ₹8.30K shown equals the trend's April ₹7.3K plus June ₹1K, and the code windows FY at today. This tab does not have the 31 March problem Dues and Collection have |
| The three clocks on the Overview strip | Live: Total ₹0 for This Month while Current FY showed ₹8.30K and owed-staff sat at ₹0 "as of today" |
| The current month renders lighter | The trend block marks the current month in progress with its own lighter colour |
| Share still renders here | The code computes it for this tab; it was removed from Dues and Collection by-property, not from Expense |

## Launch risks on this tab

Copy cannot absorb these; they generate the tickets themselves.

1. **Still owed to staff can only grow** on accounts with old passbook data and will read absurd figures at scale (finding F53). A tile that reads ₹50Cr owed to staff is instant distrust.
2. **A property the app cannot find shows made-up numbers as though they were real** (F52).
3. **Four properties in five get "add your first expense"** onboarding copy despite having expense history elsewhere (F50).
4. **The change chip's comparison window reaches into the current month seven days a year** (F47). Small, but it is a wrong percentage on the first tile.
