# DA-02 Collection, Hint Copy

The words behind every (i) on the Collection tab. Two columns, matching the two text fields the sheet has: **What is this?** and **How is this calculated?** A dash means the definition already says everything. **Good to know** is one line per block, for the sheet's block-level line when it exists.

Checked against the code at rentok-backend `0e8cc713b` and against the live account, Mumbai group, 22 properties, 27 Aug, in both views.

## The toggle comes first

This tab has a switch no other tab has, and it changes what every number means. Any sentence written for this tab has to say which side of the switch it lives on.

| | Paid Date | Due Date |
|---|---|---|
| Counts | Money, on the day it arrived | Bills, on the day they were due |
| Refunds | Already taken off every number | Not taken off: a bill paid and later refunded still counts as settled |
| Deposits and advance used to clear bills | Not counted | Counted |
| A future window | Shows ₹0, money cannot arrive in the future | Shows real bills due in the future |

**One more fact that explains most mismatches with the Dues tab:** Collection counts everyone who was billed or who paid, including tenants who have since moved out and payments with no matched tenant. Dues counts only tenants living here or booked. So Billed here can run higher than the Dues tab's bills for the same dates. On the live account: ₹2.33Cr here against ₹2.18Cr there.

## The date filter

Same five options as Dues: This Month is the 1st to today, Last Month is whole, Current FY runs 1 April to 31 March, All Time has no limits, Custom can end in the future.

## 1. Collection Overview

Different cards per view. Advance, Current FY and Settlement Pending appear in both.

**Paid Date view.**

> **Good to know:** Bills Collected, Past Dues Collected and Paid Early add up to Total Collection. Advance is separate money, not in the total.

| Card | What is this? | How is this calculated? |
|---|---|---|
| Total Collection | Money received in the dates you pick, after refunds are taken off. | Counted by the day the money arrived. Money moved from a tenant's own deposit, advance or caution money is not counted. |
| {Month}'s Bills Collected | Money received against bills due in these dates. | - |
| Past Dues Collected | Money received in these dates against bills due earlier. | - |
| Paid Early | Money received in these dates against bills due later. | - |
| Advance | Money received with no bill against it, paid ahead. | Not part of Total Collection. |
| Current FY | Money received since 1 April. | - |
| Settlement Pending | Online money collected but not yet in your bank or Flexi Pay. | - |

**Due Date view.**

> **Good to know:** Billed minus Collected & Adjusted is Still Unpaid.

| Card | What is this? | How is this calculated? |
|---|---|---|
| Billed | Every bill due in the dates you pick, paid or not. | Includes bills of tenants who have moved out, so it can run higher than the Dues tab shows. |
| Collected & Adjusted | What came in against those bills, as money or from deposits and advance. | A bill paid and later refunded still counts as settled here. |
| Still Unpaid | Billed minus Collected & Adjusted. | - |

## 2. Collection Breakup

Four tabs. Follows the toggle, and the total above the bars follows it too: Total Collection in Paid Date, Collected & Adjusted in Due Date.

> **Good to know:** Category adds up to the total shown above it, in both views. Status, Mode and Received by can fall short of it, and each says why on its own card.

| Tab | What is this? | How is this calculated? |
|---|---|---|
| Category | Collection for these dates split by bill type. | - |
| Status | Collection split by where the payer stands: living here, under notice, booked, or moved out. | A payment with no matched tenant stays in the total but not in these bars. |
| Mode | Collection split by how the money came in: cash, UPI, bank, online. | Money moved from deposits or advance is not in these bars. Online through RentOk shows as RentOk. |
| Received by | Collection split by who took the payment. | Online payments show as RentOk, not a person. |

## 3. Collection Status

Shows only in Paid Date view.

> **Good to know:** the first three bars are Total Collection regrouped by when the money was due.

| Bar | What is this? | How is this calculated? |
|---|---|---|
| {Month}'s Due | Money received against bills due in these dates. | - |
| Past Due | Money received against bills due earlier. | - |
| Future Due | Money received against bills due later. | - |
| Advance Payments | Money received with no bill. | - |

## 4. Adjusted Collection

Money already held being used, not new money arriving.

> **Good to know:** these are your tenants' own balances clearing bills, so none of this is in Total Collection.

| Card | What is this? | How is this calculated? |
|---|---|---|
| Advance | Bills cleared from a tenant's advance balance. | - |
| Security Deposit | Bills cleared from a tenant's security deposit. | - |
| Discount | Discount recorded against bills due in these dates. | Counted by the bill's due date, unlike the three cards beside it. |
| Caution Money | Bills cleared from caution money. | - |

## 5. Collection Trend

Has its own months dropdown.

> **Good to know:** uses the months on its own dropdown, not the date filter above. The second line flips with the toggle: Billed in Paid Date view, Uncollected in Due Date view.

| Line | What is this? | How is this calculated? |
|---|---|---|
| Collection | Money collected in each month. | - |
| Billed | What was billed in each month. | - |
| Uncollected | Billed but not collected, in each month. | - |

## 6. Payment Settlement

Does not follow the toggle.

> **Good to know:** same numbers in both views. Settlement is about money reaching your bank, not about billing. Settled amounts are what actually reached the bank, after online payment charges.

| Card | What is this? | How is this calculated? |
|---|---|---|
| Collected via RentOk | Online money collected through RentOk in these dates. | - |
| Total Settled | The part that has reached your bank or your Flexi Pay balance. | Flexi Pay money counts as settled even before you withdraw it. |
| Unsettled | The part not paid out yet. | - |

## 7. Collection by Property

Follows the toggle.

> **Good to know:** the ranking can change when you flip the toggle, because the two views date money differently.

| Card | What is this? | How is this calculated? |
|---|---|---|
| Collection by property | Collection per property for these dates, highest first. | - |

---

## The pointers are checked, not guessed

Read off the live account on 27 Aug, This Month selected, both views.

| Claim | Checked |
|---|---|
| The three cards add to Total Collection | ₹1.99Cr + ₹29.74L + ₹6.28L = ₹2.35Cr shown |
| Billed minus Collected & Adjusted is Still Unpaid | ₹2.33Cr − ₹2.17Cr against ₹15.86L shown |
| Breakup adds to its own header, both views | Category bars: ₹2.35Cr in Paid Date, ₹2.17Cr in Due Date |
| Status regroups Total Collection | ₹2Cr + ₹29.7L + ₹6.3L = ₹2.35Cr |
| Settlement identical in both views | ₹1.79Cr / ₹1.79Cr / ₹0 on both |
| By Property re-ranks on the flip | PH-MIDC ₹33.92L → ₹36.03L; positions 3 and 4 changed hands |
| Billed runs above the Dues tab's bills | ₹2.33Cr here, ₹2.18Cr there, same dates |

## Build asks

**1. One glossary serves both views.** The backend knows which view is selected when it builds the sheet, but ships the same sentences either way. The tables above are already split per view; serving them per view is a data edit, not new machinery. This is the single fix that makes this tab's copy honest.

**2. The block-level line**, same ask as Dues: Good to know needs one field on the sheet.

**3. Advance and Current FY lose their change chips in Due Date view** (finding F31), and Settlement Pending never gets one (F30).

**Cells that change when known fixes land:** the Discount card reads a column that is empty on production, so it shows ₹0 until the number is wired (F40); Current FY in Due Date view runs to 31 March instead of today (F9, F32). The old Total Collection sentence claimed "advance included"; the code keeps true advances out, and the copy above says so.
