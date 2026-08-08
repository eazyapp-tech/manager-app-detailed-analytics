---
title: Collection — Handoff Sheet
screen: DA-02 — Collection
status: v13 · restructured to the suite template · definitions carried from v12
owner: Sanchay
date: 2026-08-08
tags:
  - rentok
  - collection
  - financial
  - detailed-analytics
  - handoff
---
# Collection — Handoff Sheet

Everything on the Collection analytics screen: what each number means, what window it covers, what happens when it is tapped, and what it shows when there is nothing to show.

## What is in here

| | Section | For |
|---|---|---|
| **1** | [Build status](#1-build-status) | everyone |
| **2** | [Where this lives](#2-where-this-lives) | everyone |
| **3** | [What every number counts](#3-what-every-number-counts) | everyone |
| **4** | [How the screen behaves](#4-how-the-screen-behaves) | backend + design |
| **5** | [Collection Overview](#5-collection-overview) | backend |
| **6** | [Collection Breakup](#6-collection-breakup) | backend |
| **7** | [Collection Status](#7-collection-status) | backend |
| **8** | [Adjusted Collection](#8-adjusted-collection) | backend |
| **9** | [Collection by Property](#9-collection-by-property) | backend |
| **10** | [Collection Trend](#10-collection-trend) | backend + design |
| **11** | [Payment Settlement](#11-payment-settlement) | backend + design |
| **12** | [View all sheet](#12-view-all-sheet) | backend + design |
| **13** | [What each number opens](#13-what-each-number-opens) | backend |
| **14** | [Who can see this](#14-who-can-see-this) | backend |
| **15** | [What each card shows when it is empty, healthy or broken](#15-what-each-card-shows-when-it-is-empty-healthy-or-broken) | design + copy |
| **16** | [What this screen is not](#16-what-this-screen-is-not) | everyone |
| **17** | [Build guidance](#17-build-guidance) | backend |
| **18** | [Open items](#18-open-items) | named owners |
| **19** | [Design file: what needs fixing](#19-design-file-what-needs-fixing) | design |
| | [Where the measured figures came from](#where-the-measured-figures-came-from) | anyone re-checking a decision |

**Building the backend?** Read 1, 3 to 13, 17, 18. **Working on the design?** Read 3, 4, 10, 11, 12, 15, 19. **Just need the decisions?** Read 3, 4 and the appendix.

## 1. Build status

The screen is designed and signed off in the design file; the View all sheet is the one exception, it is not drawn at all (§12). The backend serves this screen as a scaffold: the block is wired and reachable but answers with nothing while the real calculations wait to be written. Nothing in this sheet describes a live defect; it describes what each number must do once built. The collections list that most taps land on already exists and works, and its filter drawer already reaches most of the slices this screen needs (§13).

Today the numbers a manager can see disagree with each other: the homescreen's collection figure, the collections list and the old collection widgets do not produce one answer. Making them one answer is part of this build, and the tests in §17 hold every card to it.

If the build cycle runs short, cut in this order: the weekly trend range first, then Collection by Property, then Payment Settlement's per-destination rows. The core ships together and never cuts: the Overview tiles in both views, Collection Breakup, Collection Status, Adjusted Collection, the Paid Date / Due Date toggle, and the View all sheet. The View all sheet stays in core because every screen's overview rail opens one; a manager who learns the pattern on one screen must find it on the next.

## 2. Where this lives

Manager App → property → **Financial** tab → **Collection** sub-tab. The suite tab row:

| | |
|---|---|
| Financial | Dues · Collection · Expense |
| People | Leads · Bookings · Tenants · Old Tenants |
| Inventory | — |
| Complaints | — |

The entry point is the widget at the top of the Financial tab. Screens drawn around it in the design file are reference context only.

## 3. What every number counts

A **payment** is one record of money received: a tenant, an amount, a date it arrived, a way it was paid, and the bill or bills it went against. This screen is built from payments the way Dues is built from bills.

| Term | Meaning |
|---|---|
| Collection | Money actually received. Bills cleared without new money arriving are adjustments, never collection |
| Adjustment | A bill dealt with using money already held: the tenant's deposit, advance, caution money, or a discount the owner gave |
| Advance | Money received against no bill. It sits with the property until a bill uses it up |
| Paid early | A payment against a bill whose due date is still ahead |
| Online via RentOk | Payments made through RentOk's own payment gateway. Their journey to the owner's bank is the Payment Settlement card's subject |
| Money on this screen | Rupees, shown short: ₹23K, ₹2.45L |

### The base rule

A payment counts if it completed and is still active. A payment that failed, is pending, or was cancelled appears in no number here. The amount counted is what the owner receives: payment processing charges come off first, so a ₹10,000 online payment carrying a ₹200 charge counts as ₹9,800. **Refunds come off everywhere**, not only the headline: every tab, every bar, every property row and every View all row drops when a refund is made against its slice. Part payments count their paid amount; a tenant paying half their rent is collection.

### The two dates every payment carries

The date money **arrived**, and the date its bill was **due**. The whole screen pivots on which one you read by; §4 starts there.

### Words to be careful with

| | |
|---|---|
| **Settled** | Reserved for one meaning: money that has physically reached the owner's bank. Only the Payment Settlement card uses it. A bill being dealt with is "collected" or "adjusted", never "settled". |
| **Advance** | Two moments of one story. The Overview tile counts advance **received**. The Adjusted Collection row counts advance **used up** against a bill. Same money, entering and leaving. |
| **Bill** | On this screen and Dues, a bill is a due raised to a tenant. On Expense the same word means a receipt for spending. |
| **Under notice** | The suite's word for a tenant with a confirmed leaving date. The design's "Under Eviction" label is renamed to match (§19). |
| **Others** | Every overflow list on this screen opens the same way: a bottomsheet naming everything left out of the top rows, each row carrying its own total, each row drillable onward to its own filtered list. One pattern, used everywhere it applies. |

## 4. How the screen behaves

### The two views

A toggle at the top switches the whole screen between two readings of the same payments. It is never a per-card setting.

| View | Reads by | The question it answers |
|---|---|---|
| **Paid Date** (opens first) | The date money arrived | What came in, and what kind of money was it? |
| **Due Date** | The date the bill was due | How did this period's billing do? |

A bill due in May and paid in June counts in June under Paid Date and in May under Due Date. Both are correct; they answer different questions. The reverse too: a June bill paid early in May sits in May under Paid Date, labelled Paid early, and in June under Due Date, counted against June's billing.

What changes when the toggle flips:

| Card | In Due Date view |
|---|---|
| Collection Overview | Four tiles are replaced by three (§5) |
| Collection Breakup | Category and Status gain a billed comparison; Mode and Received by show Collected & Adjusted only (§6) |
| Collection Status | Hidden (§7) |
| Collection Trend | The collection bar counts differently (§10) |
| Adjusted Collection, Collection by Property, Payment Settlement | No change |

**In Due Date view, adjustments count as dealt with.** A tenant moving out has their ₹10,000 May rent cleared from their deposit: the bill is done and nobody owes anything. If that ₹10,000 sat in Still Unpaid, a manager would chase a tenant who owes nothing. So money cleared from deposit, advance, caution money or a discount all count towards **Collected & Adjusted**, all four, and a discounted bill is just as dealt with as a deposit-adjusted one.

**Three numbers never switch.** Advance, Current FY and Settlement Pending always count by when money arrived, in both views. None of them is about billing: advance has no bill, Current FY is a fixed year of money in, and Settlement Pending is about where money has got to on its way to the bank.

### The time filter

Options: **This Month (default) · Last Month · Current FY · Custom · All Time · Coming up.**

**Custom stops at today. The future belongs to Coming up**, the same forward setting Dues, Tenants and Inventory carry: pick a future date, see the window from tomorrow to that date, default 30 days. On Collection everything in it is a record, never a guess: bills already raised with future due dates, and the money already received against them. Coming up only means something in Due Date view, because money cannot arrive in the future; picking it in Paid Date view shows one line and a way across:

> **Nothing received yet.** Collection counts money as it arrives. Switch to Due Date to see what's billed for this window.

The switch is tappable. No window can span today: Custom ends at it, Coming up starts after it.

Three cards can be set to their own period: Adjusted Collection, Collection by Property and Payment Settlement, each with its own dropdown carrying the same options as the top filter. Changing the top filter pulls every card back into line. The filter stays where the manager put it while the app is open; a fresh launch opens on the default. Coming back from a drill returns to the screen as it was left, with fresh numbers. A day runs midnight to midnight, India time.

Two kinds of number on this screen:

| Kind | Meaning |
|---|---|
| **Time-scoped** | Counts inside the window the filter picks |
| **Time-scoped, window fixed** | Keeps its own fixed window whatever the filter says |

Collection has no Live number and no forecast. Everything is a record inside some window.

### What every number does on every filter setting

| Number | This Month · Last Month · Current FY · Custom · All Time | Coming up |
|---|---|---|
| Total Collection + the three source tiles + Advance tile | Counted inside the window | The Paid Date tiles sit out; the view switches to Due Date |
| Billed · Collected & Adjusted · Still Unpaid (Due Date tiles) | Counted inside the window | Counted inside the window: bills due tomorrow to the chosen date, money already in against them |
| Current FY tile | Each tile keeps its own fixed window; the filter does not change it | Each tile keeps its own fixed window; the filter does not change it |
| Settlement Pending tile | Counted inside the window | The card sits out; money cannot arrive in a future window |
| Collection Breakup, all four tabs | Counted inside the window | Counted inside the window, Due Date reading |
| Collection Status | Counted inside the window | Hidden, with the rest of Paid Date view |
| Adjusted Collection | Counted inside the window; its own dropdown can pick a different one | The card sits out; money cannot arrive in a future window |
| Collection by Property | Counted inside the window; its own dropdown can pick a different one | Counted inside the window, Due Date reading: which property has most landing, and how much is already in |
| Collection Trend | Each keeps its own range; the filter does not change it | Each keeps its own range; the filter does not change it |
| Payment Settlement | Counted inside the window; its own dropdown can pick a different one | The card sits out; money cannot arrive in a future window |
| View all sheet | Counted inside the window, except the rows that state their own fixed window | Counted inside the window, Due Date reading |

When the filter is on Current FY, the Current FY tile and Total Collection show the same figure. The tile stays: a row that reshuffles under the thumb is worse than a number that repeats, and at every other setting the tile puts the year beside the month.

### Periods that have not finished

An unfinished period compares against the same elapsed days of the previous one, marked on the chip: "▲ 12% vs same point last month." The note drops away once the period completes. On Custom, chips compare the same number of days immediately before the range. The unfinished period is always marked wherever it appears, the chip and the trend chart's in-progress bar alike.

### Change chips

Every Overview tile carries a chip except on All Time, which has no previous period, and on Coming up, where there is nothing fair to compare. Direction is per number, not per screen:

| Number | Rising means | Chip |
|---|---|---|
| Total Collection, the three source tiles, Advance, Current FY, Collected & Adjusted | more money in | green up, red down |
| Settlement Pending, Still Unpaid | more money stuck or owed | red up, green down |
| Billed | more billing, neither good nor bad | neutral grey |

The neutral chip is a state the shared chip component does not have yet; Inventory logged the same need.

### When a number is red

Red only where somebody can be held to it. A negative Total Collection is red with its explanation (§5). Money whose transfer was reversed is marked (§11). Nothing else on this screen is red by default.

### Loading, failure, sorting, entry

Each card loads with a skeleton shaped like the card it becomes and fails alone with "Couldn't load this" and Retry, never a healthy message, never a zero. Retry refetches only that card. When every card fails it is the connection: one message, one Retry for the screen. All Time is the heavy setting: on a property with years of history every card scans the lot, so the skeletons matter most there. Property-wise components sort highest to lowest. The entry point is the widget at the top of the Financial tab.

## 5. Collection Overview

Seven tiles in Paid Date view. Each shows an amount and a change chip per §4.

| Tile | Counts | Kind |
|---|---|---|
| Total Collection | all money received in the window, refunds off | Time-scoped |
| This Period's Bills Collected | the part that paid bills due inside the same window. The face carries the month's own name | Time-scoped |
| Past Dues Collected | the part that paid bills due before the window: old dues recovered | Time-scoped |
| Paid Early | the part that paid bills due after the window. Advance is not here, it has its own tile | Time-scoped |
| Advance | money received against no bill | Time-scoped |
| Current FY | money received since 1 April | Time-scoped, window fixed |
| Settlement Pending | online money received in the window not yet in the owner's bank | Time-scoped |

The middle four tiles add up to Total Collection exactly. Adjustments are never counted in any of them; they clear a bill without new money arriving and live in Adjusted Collection (§8).

**Due Date view** replaces the first four tiles with three; Advance, Current FY and Settlement Pending stay, so it shows six:

| Tile | Counts | Kind |
|---|---|---|
| Billed | everything billed for the window, paid or not. Bills written off or fully refunded are not billing anyone owes, so they do not count | Time-scoped |
| Collected & Adjusted | how much of that billing is dealt with: money received plus bills cleared from deposit, advance, caution money or discount | Time-scoped |
| Still Unpaid | Billed minus Collected & Adjusted: what is genuinely still owed | Time-scoped |

If "Collected & Adjusted" runs too long on a small tile, "Collected" with the full meaning in its info note is acceptable; the longer label is preferred because "Collected" alone would be untrue in this view.

Two situations the Total Collection tile handles:

- **Refunds larger than money received.** The negative amount shows in red with one line: *"Refunds were higher than collection this period."* Real and rare; without the line a manager assumes the app is broken.
- **Nothing but adjustments.** ₹0 with one line: *"No money received. ₹X of bills were cleared from deposits and advances."* Without it a real month reads as a dead one.

## 6. Collection Breakup

One card, four tabs, all splitting the same total for the window. The window's total sits above the rows on every tab, and the rows add up to it.

| Tab | Splits by | Rows |
|---|---|---|
| Category | bill type | top 3 by amount, rest in **Others** |
| Status | tenant state | Active, not under notice · Under notice · Bookings · Old tenants. All four, always |
| Mode | how it was paid | top 3 by amount, rest in **Others** |
| Received by | who took the payment | top 3 by amount, rest in **Others** |

Top three means purely by amount: whatever is largest in the window shows by its real name, everything else rolls into Others, nothing is guaranteed a place and nothing is barred. Others opens the suite's overflow bottomsheet (§3), and on Received by it says how many people it holds: "Others (4 staff)".

A badge marks the largest row on the Mode and Received by tabs, moving with the data. Category and Status carry none; a winning bill type means nothing.

**Under notice** here means an active tenant with a confirmed leaving date, split out as its own row. The Dues screen keeps these tenants inside its Active bar; on Collection the split is deliberate.

**Payments with no tenant attached stay in the total** and in the Mode and Received by tabs. The Status tab says in one line that they are not counted there; they are a handful a month platform-wide, and the list's own "Non Tenant" filter reaches them.

**Payments with no receiver recorded** get their own "Not recorded" line on the Received by tab whenever they exist in the window. Real money moves through them every month, and money with no name on it is worth seeing, not hiding.

### What the tabs show in Due Date view

All four tabs stay, all four split **Collected & Adjusted** for bills due in the window, and all four still add up to it.

**Category and Status also show what was billed**, row by row, so each reads as collected against billed: "Rent ₹8L of ₹10L". That is the question this view exists for. **Mode and Received by show Collected & Adjusted only**; an unpaid bill has no payment method and nobody received it, so there is no billed side to show.

The Mode tab earns its place in this view for one reason: it is the only place on the screen that answers how much of a month's billing was cleared with real money and how much was drawn out of deposits. For a property with heavy move-outs, that share is a real signal.

## 7. Collection Status

*Paid Date view only. Hidden in Due Date view.*

Not how much came in, but what kind of bills it paid. Four bars on one shared scale:

| Row | Meaning |
|---|---|
| This Period's Due | bills due in the window that were collected |
| Past Due | older bills recovered during the window |
| Future Due | bills due later, paid early |
| Advance Payments | money received as advance, against no bill |

These four are the same numbers as four Overview tiles. They appear twice on purpose: the tiles are the headline, and the bars let you compare the four against each other, which a sideways-scrolling tile row cannot do.

**No unpaid row here.** What is still owed is the Dues screen's job; this screen shows collection.

**Hidden in Due Date view** because the card's whole job is sorting money by when its bill was due, and in that view the filter has already decided it: the first row would show everything and the rest zero, every time.

## 8. Adjusted Collection

*The same in both views. Can be set to its own period.*

Bills dealt with without new money arriving, kept apart so they are never mistaken for collection. A four-item grid, in this order:

| Row | Meaning |
|---|---|
| Advance | a bill cleared using advance the tenant paid earlier |
| Security Deposit | a bill cleared using the tenant's deposit |
| Discount | the amount the owner waived off bills in the window |
| Caution Money | a bill cleared using caution money held for the tenant |

Each figure is the amount of bill that got cleared, not the size of any payment. All four are real and live in production, caution money included; it is the smallest and the newest, in use since April 2026.

**Discount works differently from the other three.** It is not a payment method. It is an amount recorded against a payment made any way at all, and it does not shrink the bill: a ₹10,000 bill with a ₹2,000 discount stays ₹10,000, the tenant pays ₹8,000, and ₹2,000 is recorded as discount. That ₹2,000 is a bill cleared without cash, exactly like a deposit adjustment, and it is counted as its own total.

Do not confuse the Advance row with the Advance tile (§3, Words to be careful with).

## 9. Collection by Property

*Multi-property accounts only. The same in both views. Can be set to its own period.*

The window's total split by property, highest to lowest, and **every row carries its share of the account total**: one property carrying 70% is a different risk picture from four at 25%, and only share-of-total makes that readable at a glance.

**Properties with zero collection stay in the list.** A property that collected nothing all window is the row most likely to need a phone call; sorting drops it to the bottom, so showing it costs nothing. Its drill opens an empty list whose empty state names the property and the window and offers the next question: *"See what was due there."*

The rows add up to Total Collection when both are on the same window; when this card is set to its own period, its dropdown says so.

## 10. Collection Trend

One stacked bar per period. **Collection sits at the bottom, in green; what sits above it is yellow.** Collection takes the bottom because it is the number managers compare across bars, and only the bottom segment starts from the same line every time.

**This chart does not follow the filter at the top.** Every other card answers "which window?"; a trend answers "how far back?" and needs several periods to say anything. It has its own range control, and the toggle still applies to it even though the filter does not.

| Range | Bars |
|---|---|
| Last 8 weeks | one per week |
| **Last 6 months** (default) | one per month |
| Last 12 months | one per month |

The axis says weeks or months, so the unit is never a guess. The weekly range shows how fast money comes in after it is due: a property collecting most of a month's billing in week one is healthy, one still collecting in week four has a follow-up problem no monthly number reveals. How sharp that curve reads depends on the property's billing setup; a property billing everyone on one day gets a clean curve, one billing on each tenant's own joining date gets a steadier rhythm. Both are real.

**How the bar is drawn.** Segments are sized to their values with a minimum height so a small segment never vanishes. Every segment carries its printed value, and those labels can never be dropped for a cleaner look: they carry the precision the shape deliberately does not. Units may differ between bars (₹90K beside ₹1.2L); the bars are sized to their values, so the shape settles what the digits confuse.

**What the two parts mean.** The yellow part always counts bills by when they were due. The green part changes with the toggle:

- **Paid Date view:** green is money that arrived in that period, yellow is what was billed in it, two independent numbers. Green can run past yellow: a month collecting heavy arrears shows more collected than billed, and that is the true story.
- **Due Date view:** green is money collected against that period's bills whenever it arrived, and **yellow is what is still uncollected of them**, so the whole bar is exactly what was billed. Green can never pass the top. The more yellow you see, the more is outstanding. The current period always shows heavy yellow because its bills are not due-and-paid yet; the in-progress bar is marked as unfinished, the same rule as the chips.

Yellow drawn as the full billed amount in Due Date view would double-count every collected rupee and draw a fully collected month as half done; the remainder is the only honest top segment.

**Tapping.** Each segment drills directly: green opens the collections list, yellow opens the Dues screen, both arriving on **the period of the bar tapped**, not the screen's. This chart is the one place on the screen where a drill changes the window rather than narrowing it. With the weekly range, tapping a week's bar carries that week.

If managers ever ask to pick their own range here: the chart should choose its own bar size to fit, days for a short stretch, weeks for a few months, months for a year or more, always landing between four and twelve bars, and always saying which unit it picked. Not for this build; the three ranges cover what managers ask a trend.

## 11. Payment Settlement

*The same in both views. Can be set to its own period.*

Where online money is on its way to the owner's bank.

| Tile | Meaning |
|---|---|
| Collected via RentOk | online money received in the window through RentOk |
| Total Settled | how much of it has reached the owner's bank |
| Unsettled | how much has not |

Collected via RentOk is the total; Settled and Unsettled are its two parts and add back to it. The bar below shows **two segments**, Settled and Unsettled; the total is not a segment of itself. Money that has started moving but not landed counts as Unsettled: a manager needs "in my bank or not", and the in-between states live one tap away. **Money whose transfer was reversed or failed is marked within Unsettled**: it is stuck, not on its way, and the manager waiting for it needs to see the difference.

**Below the bar, one row per destination shows where settled money went**, under a heading that says exactly that: *"Where settled money went."* A destination is usually a bank account but can be a UPI address. The rows break down Total Settled only and add up to it; the Unsettled tile accounts for the rest.

**The destination comes from the settlement record, whichever settlement system wrote it.** Most properties' records live on the older system and still name their destination; those rows draw. A row only falls back to the placeholder when its record does not resolve to a nameable destination: *"Bank details aren't available for this settlement. Contact support."* Never predict a destination for unsettled money; which account money lands in can depend on the order payments arrive, so a guess can turn out wrong. Unsettled money simply is not in the destination rows.

One thing engineering must confirm before the Unsettled tile ships (§18): a third of online money currently has no settlement record in any system, and this card would show all of it as Unsettled. Whether that is the truth or a record-keeping gap decides whether the tile can be trusted on day one.

## 12. View all sheet

Opens from **View all** in the Collection Overview header. A bottomsheet holding every collection number the main screen has no room for. It reads by the screen's window and toggle, except the rows that state their own fixed window on their face.

| Group | Rows |
|---|---|
| The window's collection | total collection · collection against the window's own bills |
| By category | every bill category present in the window, largest first |
| By stay duration | short-term tenants · long-term tenants |
| Financial year | collected this financial year · billed this financial year |
| Adjusted collection | this window · this financial year · all time |
| Settlement | unsettled amount |

Every row that can open a list does, arriving filtered to exactly that row's slice, the same as if tapped from a card. In Due Date view, every row that can show a billed side reads as collected against billed. Stay duration lives only here; on Collection it is a secondary cut, not a card.

## 13. What each number opens

<details>
<summary><strong>Expand:</strong> the drill rules, what travels with a tap, the full tap matrix with arrival filters, and the list filters that still have to be added</summary>


### The rules

A drill filters a list, it never re-scopes the screen. The destination follows the record's kind: payments open the collections list, bills open the Dues screen even from this screen, settled money's journey lives on the FlexiPe screen. The destination opens on the same window and the same properties, and the back control names this screen. Records add back to the number tapped. Every overflow is the one Others pattern (§3).

**Deeper pages are not part of this build's drills.** Some pages reachable below the first list do not check the viewer's permission the way the list does; drilling stops at the list until that is resolved (§18).

### When the window changes what a tap shows

| The screen's window | What travels to the list |
|---|---|
| A period, running or finished | The window travels. Paid Date numbers carry it as a collected-on range; Due Date numbers carry it as a due-on range |
| Coming up | The forward window travels as a due-on range, tomorrow through the chosen date. Real bills, real payments, a real list |
| A fixed-window row (Current FY, all time) | That row's own window travels, not the screen's |

### The tap matrix

| You tap | What opens | Arriving filtered to | Ready? |
|---|---|---|---|
| **Collection Overview, Paid Date** | | | |
| Total Collection | collections list | collected inside the window | ✅ |
| This Period's Bills Collected | collections list | collected inside the window, against bills due inside it | ✅ both date filters combine |
| Past Dues Collected | collections list | collected inside the window, bills due before it | ✅ |
| Paid Early | collections list | collected inside the window, bills due after it | ✅ |
| Advance | collections list | collected inside the window, Advance category | ✅ |
| Current FY | collections list | collected since 1 April | ✅ |
| Settlement Pending | collections list | online in the window, not settled | ❌ settlement-status filter to add |
| **Collection Overview, Due Date** | | | |
| Billed | Dues screen | bills due inside the window | ✅ |
| Collected & Adjusted | collections list | payments and adjustments against bills due inside the window | ✅ |
| Still Unpaid | Dues screen | bills due inside the window, still unpaid | ✅ |
| **Collection Breakup** | | | |
| a Category row | collections list | that category, the window | ✅ "Collection Types" |
| Status: Active, not under notice | collections list | active tenants' payments, the window | ✅ |
| Status: Under notice | collections list | under-notice tenants' payments | ❌ under-notice filter to wire in from the tenants list |
| Status: Bookings | collections list | bookings' payments | ✅ |
| Status: Old tenants | collections list | old tenants' payments | ✅ |
| a Mode row | collections list | that payment mode, the window | ✅ |
| a Received by row | collections list | that receiver, the window | ✅ matched by name |
| Received by: Not recorded | collections list | payments with no receiver | ❌ no-receiver filter to add |
| any Others row | overflow sheet, then the list per row | the remainder, then that one slice | ✅ |
| **Collection Status** | | | |
| any row | collections list | as the matching Overview tile | ✅ |
| **Adjusted Collection** | | | |
| Advance | collections list | advance-adjusted payments, the window | ⚠ add "Adjusted from Advance" to the mode filter options |
| Security Deposit | collections list | deposit-adjusted payments, the window | ✅ |
| Discount | collections list | payments with a discount applied, showing the discount amount per row | ❌ discount filter to add |
| Caution Money | collections list | caution-adjusted payments, the window | ✅ |
| **Collection by Property** | | | |
| a property row | collections list | that property, the window | ✅ |
| **Collection Trend** | | | |
| green segment | collections list | collected inside that bar's period | ✅ |
| yellow segment | Dues screen | bills due in that bar's period | ✅ |
| **Payment Settlement** | | | |
| Collected via RentOk | collections list | online modes, the window | ✅ "RentOk Transfer" |
| Total Settled | FlexiPe screen | settled transactions | ⚠ confirm FlexiPe can arrive pre-filtered |
| Unsettled | collections list | online in the window, not settled | ❌ settlement-status filter to add |
| a destination row | FlexiPe screen | that account's transactions | ⚠ confirm FlexiPe can arrive on one account |
| **View all sheet** | | | |
| tile and category rows | collections list | that row's slice | ✅ |
| stay rows | collections list | that stay type | ❌ stay-type filter to add; the Dues screen waits on the same one |
| adjusted rows | collections list | adjustment modes, that row's window | ⚠ needs "Adjusted from Advance" added, then ✅ |
| unsettled row | collections list | not settled | ❌ settlement-status filter to add |

Every ❌ is a filter over data that already exists. None is history to start recording, and none is a new screen to build: the one candidate for a new screen, settlements, turned out to already exist as the FlexiPe screen.

### What the lists can already do, and what has to be added

The collections list already filters by property, collected-on window, due-on window (future dates included), payment mode, receiver, bill category, and tenant type including payments with no tenant. The two date windows combine, which is what the source-bucket drills need.

| New filter to add | Which numbers wait on it |
|---|---|
| Settlement status | Settlement Pending tile, Unsettled tile, the View all unsettled row. Partly built already: a six-state settlement filter exists in the mobile app, constructed but never switched on. Finishing it also fixes the deeper problem that today's settlement arrival codes wipe every other filter, so "unsettled in May" is inexpressible |
| Discount applied / not applied | the Discount drill. The discount amount already travels with each row's data, so the list can show it per row when this filter is on |
| Under notice | the Breakup Status tab's Under notice row. The tenants list already has exactly this filter; it is not wired into collections |
| "Adjusted from Advance" as a mode option | the advance-adjusted drills. Its two siblings are already options |
| No receiver recorded | the Not recorded row |
| Stay type | the View all stay rows, the same filter Dues is already waiting on |
| A named forward window option | every drill taken from Coming up. The list's due-on filter already accepts future dates through a custom range; what is missing is a named option a manager would find |

### What the destination says when you arrive

The list names the slice it arrived filtered to, shows the active filter without scrolling, and names that filter in its empty state with a way on. None of that is true of the list today (§18); it is one shared fix for every screen in the suite.
</details>

## 14. Who can see this

**Anyone who can view collections.** The suite rule: each analytics tab follows the permission of the records it describes. Dues and Collection open and lock together; Expense is separate. No partial state.

Whoever lacks it sees the lock: *"Analytics Restricted, You don't have permission to view these analytics. Request access from your admin,"* with **Request Access**.

Narrowed property access counts only those properties, and the drill matches.

## 15. What each card shows when it is empty, healthy or broken

### The zeros, told apart

| Situation | What shows |
|---|---|
| Never set up: no tenants | *"No collection yet. Add your first tenant and this page fills in."* with **Add tenant** |
| Onboarding: tenants added, nothing received yet | *"Nothing collected yet. Payments you receive appear here."* |
| In-window zero: a live property with nothing received | the zero, plain, with the way on: *"No payments received in July. See what was due."* One live property in six has such a month; this state is ordinary, not an edge |
| Adjustments-only window | ₹0 with its one line (§5) |
| Refunds exceed collection | the negative total with its one line (§5) |
| Good-news zero | see Healthy below |
| Not recorded | see Not recorded below |
| Coming up with nothing due ahead | *"Nothing due in the next 30 days."* Named with the window the manager picked |
| Failed | "Couldn't load this" with Retry, never a healthy message, never a zero |

### Healthy: good news, no CTA

| Number | Reads |
|---|---|
| Still Unpaid at zero | *"Everything billed for this period is collected."* |
| Unsettled at zero | *"All online money has reached your bank."* |
| Settlement Pending at zero | same good news, tile form |

Zero collection is never good news on this screen; only zero *owed* and zero *stuck* are.

### Not recorded: draws what exists, states its coverage

| Where | Reads |
|---|---|
| A settlement destination that does not resolve | *"Bank details aren't available for this settlement. Contact support."* on that row only; resolvable rows still draw |
| Received by, payments with no receiver | the "Not recorded" line (§6) |

### Empty-state copy fixes

| Card | Currently says | Should say |
|---|---|---|
| Breakup, Mode tab | promises a split by "cash, UPI, bank transfer & cheque" | *"Once payments come in, you will see a split by mode: RentOk, bank transfer, UPI, cash and others."* The current copy promises modes the card does not show and omits the biggest one |
| Collection Status | "No status yet. Dues collected and pending will show up here." | *"No collection yet. Once payments come in, you will see what kind of bills they paid."* This card has no pending row by design; the copy promises one |
| Adjusted Collection | "No adjustments recorded. Advance payments and deposit adjustments will appear here." | *"Nothing adjusted yet. Bills cleared from a deposit, advance, caution money or a discount will appear here."* The current copy names two of four and "advance payments" reads as the Advance tile |
| Collection by Property | "No properties added yet. Add a property and collect dues to see how each property is doing." | *"No collection at any property in this period."* The card only renders when several properties exist, so the empty case is no collection, not no properties |
| Breakup Category, Status, Received by · Trend · Payment Settlement | fine as written | no change |

The Overview tile row has no empty state drawn at all and needs one; a window with no activity is ordinary (§19).

## 16. What this screen is not

- **Not a place to record a payment.** The existing payment flows own data entry.
- **Not a bank ledger or a cash-flow report.** This screen explains collection; FlexiPe and the finance reports own the money's ledger.
- **Not a second payments list.** Every drill goes to the list that already exists.
- **No alerts or notifications.** Managers know when to open this; the problem it solves is clarity once they do.
- **No guessing where money will land.** Settled money shows its real destination; unsettled money shows none.

## 17. Build guidance

1. **One total, many views.** Every card slices the same base. *Test it:* the Breakup tabs, the property rows and the window's trend bar each sum to Total Collection on the same window.
2. **The homescreen and this screen agree.** *Test it:* the same property and month show the same collection figure on the homescreen tile and this screen's Total Collection.
3. **A payment is fresh money or an adjustment, never both.** *Test it:* a deposit-adjusted bill moves Adjusted Collection and no Overview tile.
4. **Refunds come off everywhere.** *Test it:* refund ₹5,000 against a July rent payment; July's total, the Rent row, July's trend bar and that property's row all drop by ₹5,000.
5. **Every rupee lands in exactly one category.** *Test it:* a caution-money payment raises exactly one Breakup row, and the rows still sum to the total.
6. **The three adjustment methods behave identically.** *Test it:* clear three bills, one from deposit, one from advance, one from caution money; all three appear in Adjusted Collection and none in Total Collection.
7. **A credit payment counts what the owner receives, charges off exactly once.** *Test it:* a ₹10,000 credit payment with a ₹200 processing charge contributes ₹9,800, not ₹9,600. Rare money today; the maths still must be right, and finance signs off the correction since it changes a figure people already watch.
8. **Cash means all cash.** *Test it:* the Mode tab's cash figure and the cash drill return the same payments, including older cash entries recorded under the earlier cash code.
9. **Part payments count their paid amount.** *Test it:* pay ₹4,000 of a ₹10,000 bill; every total moves by exactly ₹4,000.
10. **Due Date view counts all four adjustment kinds as dealt with.** *Test it:* a May bill cleared by discount shows inside Collected & Adjusted and not in Still Unpaid.
11. **The Due Date trio adds up.** *Test it:* Collected & Adjusted plus Still Unpaid equals Billed, on every window.
12. **A drill adds back to the number tapped.** *Test it:* tap ₹30,000; the list arrives totalling ₹30,000.
13. **A trend bar equals its period opened directly.** *Test it:* March's green segment equals Total Collection with the filter on March.
14. **Settlement Pending and the Unsettled tile are one measure.** *Test it:* on the same window they show the same figure.
15. **Chips compare unfinished periods fairly.** *Test it:* on the 8th, the chip compares against the first 8 days of last month, marked unfinished, and the mark drops when the month completes.
16. **Chip direction is per number.** *Test it:* rising Total Collection shows green; rising Settlement Pending shows red; rising Billed shows the neutral chip.
17. **The top filter pulls every card back.** *Test it:* set Adjusted Collection to Last Month, change the top filter; the card snaps to the new window.
18. **Custom stops at today; Coming up starts tomorrow.** *Test it:* the Custom picker cannot select a future end date, and Coming up's window never includes today.
19. **Coming up counts records only.** *Test it:* on Coming up, Billed counts only bills already raised with due dates in the window; nothing is projected.
20. **The Paid Date family sits out Coming up with the switch line.** *Test it:* pick Coming up in Paid Date view; the sitting-out line appears with a tappable switch, and no tile renders a zero that reads as a collections failure.
21. **A window labelled forward looks forward.** *Test it:* a named forward option on the list shows bills due next month, not last month's. The shared date control's "Next 7 days" preset currently resolves backwards; this test exists to catch exactly that.
22. **A failed card never renders a number.** *Test it:* fail the settlement card's fetch; it shows "Couldn't load this" and Retry, not ₹0.
23. **Settled destinations draw from whichever settlement system holds the record.** *Test it:* a property whose settlements live on the older system still shows its destination rows; only a row with no resolvable destination shows the contact-support line.
24. **Export stays on the existing list.** No separate export path for this screen; an export that disagrees with the list it came from is another number nobody can explain.
25. **Knock-on check before launch.** Collection numbers feed notice reports, tenant ledgers and deposit and advance balances elsewhere. Discount in Adjusted Collection and adjustments counting as dealt with in Due Date view both touch those; nothing here may contradict a tenant's ledger.

## 18. Open items

1. **Engineering.** A third of online money has no settlement record in any system. Before the Unsettled tile ships, confirm that no-record genuinely means not-settled rather than a record-keeping gap; the tile would show it all as stuck money, and if that is wrong, managers will call support about failures that never happened.
2. **Engineering.** The live collections widget strip includes a deposit-adjusted shortcut that fails when tapped; its this-financial-year and all-time siblings work. Worth fixing ahead of this build since it is customer-visible today. *Test it:* tapping the deposit-adjusted widget returns a list, not an error.
3. **Engineering.** The tenant-type filter's "Old Tenants" option needs its meaning verified against real data before the Status tab builds on it; the backend's own notes call the same value "pre-joining", and both cannot be right.
4. **Owner + engineering.** Do deeper drills ship on day one? Pages below the first list do not all check permissions the way the list does. Ship or hold is undecided, and it blocks nothing else in this sheet.
5. **Engineering.** A permission exists that narrows a member's view to only tenants they added. Product says nobody is granted it. Confirm, or two people can see different totals on one property with nothing on screen saying so.
6. **Shared, unowned.** The payments list names none of its arriving filters (§13). One fix serves every screen in the suite.

## 19. Design file: what needs fixing

<details>
<summary><strong>Expand:</strong> 24 numbered design-file fixes, grouped Wrong / Missing / Remove / Decide</summary>


**Wrong**

1. Overview tiles "Past Bill" and "Future Bill" read as an old bill and an upcoming charge. They are **Past Dues Collected** and **Paid Early**.
2. The Status tab's "Under Eviction" bar is the suite's **Under notice**, and the first bar reads **Active, not under notice**.
3. The Mode tab's empty copy promises cash and cheque and omits RentOk; §15 has the replacement, and the built tab needs its Cash row.
4. Three more empty states are wrong (Collection Status, Adjusted Collection, Collection by Property); §15 holds all replacement copy.
5. The time filter is drawn as a stepper reading "Today". It is the six-option dropdown, defaulting to This Month.
6. The badge on Mode and Received by is stuck on the RentOk row; it follows whichever row is largest.
7. The trend chart draws Due as the bottom segment; Collection takes the bottom.
8. The Payment Settlement bar has three segments; it takes two, and its tile and row numbers do not add up (sample data).
9. Settlement rows are labelled bank rows though one is a UPI address; the heading "Where settled money went" covers both.
10. Collection by Property bars do not track their amounts and the rows are unsorted (sample data), and Breakup and Status bars have the same fault.
11. The Breakup tabs' period total does not match the rows beneath it (sample data), and the Category tab misspells Electricity.
12. Tiles hard-code "May's"; the label reads as the selected window's own month.
13. The trend's empty state shows a month dropdown instead of the range control.

**Missing**

14. The Advance tile, the three Due Date tiles, and the negative-total, adjustments-only and unfinished-window treatments.
15. The Coming up option, its date picker, and the Paid Date sitting-out state with the tappable switch.
16. The View all sheet: the link exists and opens nothing.
17. The weekly trend range, the Due Date trend, and the marked in-progress bar.
18. The neutral change chip, and "Not recorded" on Received by.
19. The Others bottomsheet for every overflow, with the staff count on Received by's.
20. An empty state for the Overview tile row, and the healthy states in §15.
21. Every card carries an info icon and no info copy is written anywhere; write it per card or drop the icons.

**Remove**

22. The trend legend's two pill chips look tappable; if they do not toggle the series, they should not look like buttons.

**Decide**

23. The trend card fits about four and a half bars at the six-month range: horizontal scroll or narrower bars.
24. Received by has no written spec in the design's own notes; this sheet is its first. Design should confirm the tab as specced here.
</details>

## Where the measured figures came from

<details>
<summary><strong>Expand:</strong> 18 production figures and the decision each one settled</summary>


Measured on production, 8 August 2026, on July 2026 unless stated. Successful, active payments only; per-property figures keyed on the property reference present on every payment. Amounts are stored in rupees. No junk exclusion was needed: July's largest single payment was ₹7 lakh.

| Measured | Result | What it decided |
|---|---|---|
| July scale | 236,464 payments, ₹238.9 crore | the scale every total must survive |
| Source split of July money | 85.5% against bills due in July · 9.6% before · 5.0% after · 0.008% no bill | all four source tiles carry real money |
| Adjustments in July | deposit ₹3.9 crore · advance ₹93 lakh · caution ₹1.2 lakh · discount ₹1.07 crore | all four Adjusted rows are real |
| Caution-money adjustments, all time | 116 payments since April 2026, still occurring | the "not built" claim from earlier documents is dead; the path is live and new |
| RentOk-credit payments | 27 ever, none since May 2026 | the counted-once rule stays as a test, not a headline |
| Properties where July refunds beat July collection | 27 | the negative-total state is real and rare |
| Adjustments-only properties in July | 17 | that state too |
| Unlinked payments | 10 of 236,464 | the Status tab states the exclusion in one line rather than carrying a ghost row |
| Payments with no receiver | 0.7% of July, about ₹1.7 crore | the "Not recorded" line on Received by |
| Mode mix of July money | online 60.8% · cash 21.5% · bank, UPI and others 17.7% | cash is the second-biggest mode; the missing Cash row matters |
| Receivers per property | median 2, one in ten properties above 3, max 13 | top 3 plus Others fits; the overflow count matters at the tail |
| Settlement of July's online ₹142.3 crore | 65.6% settled · 34.3% no record in any system | open item 1, and the coverage honesty in §11 |
| Newer settlement system reach | 160 properties ever, of 10,500 collecting online | destination rows must draw from the older system's records too |
| Older-system destinations | 8,326 properties with settled money naming an account, median 1 account, one in ten above 4 | the same decision, and the overflow size for destination rows |
| Live properties with a zero-collection July | 1,342 of 7,878, one in six | the in-window zero is an ordinary state, not an edge |
| Money already received against future-dated bills | ₹5.2 crore across 4,688 bills | Coming up counts real money today |
| Billing-day spread | 41% of properties bill on one day · a third spread across five or more | the weekly trend's caveat: a curve for some, a rhythm for others |
| Discount-carrying payments | 992 in July | the discount filter has real material |
</details>
