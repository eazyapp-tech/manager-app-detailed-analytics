---
title: Dues — Handoff Sheet
screen: DA-01 — Dues
status: v12 · closed · the suite's current reference sheet
owner: Sanchay
date: 2026-08-08
tags:
  - rentok
  - dues
  - financial
  - detailed-analytics
  - handoff
---
# Dues — Handoff Sheet

> [!NOTE] Corrected 2026-08-08, from the Inventory uplift's sibling check
> - The Active-not-under-notice bar now says "no confirmed leaving date", matching this sheet's own confirmed-only rule.
> - One banned time word swept; the two-kinds line scoped to this screen.

> [!NOTE] Corrected 2026-08-09, sibling debts paid
> - Added the filter-memory contract to section 4, copied verbatim from Expense v3 §4. Converted all 19 test lines to the suite's "Test it:" phrasing.

> [!NOTE] Corrected 2026-08-09, View all sheet re-audited against Figma
> - Section 6 rewritten as a Row | Meaning table, built from the current "Dues Overview" frame. It is no longer treated as a stale draft.
> - Design file fixes: items 2 and 3 renamed Defaulter to Overdue Breakup, item 7 notes its wrongly written amounts recur in the View all sheet, item 9 now covers a chip the View all sheet draws where §4 allows none, item 25 now covers the Duration segments hide-or-zero question. Items 26 to 28 added.
> - Build status no longer lists the view-all sheet as an exception.


> [!NOTE] Corrected 2026-08-13, how the change chip gets its comparison figure
> - Section 4 gains **Where the comparison number comes from**: nothing stores a daily record of dues, and none is needed. A bill carries its due date and its payment date, so what was owed on a past date is every bill due by then that had not been paid yet on that date. Part payments need no special handling, since a part-paid bill becomes two ordinary bills.
> - Named the trap: reading today's unpaid bills as last month's figure runs about a quarter too small, which would show a steady property red every month.
> - Two rules stated: nothing to compare against means no chip at all, and the past figure describes the period as we understand it today.
> - Build guidance 13 to 16 added, including the one known gap, an edited bill reads at its corrected amount.


Everything on the Dues analytics screen: what each number means, what window it covers, what happens when it is tapped, and what it shows when there is nothing to show.

## What is in here

| | Section | For |
|---|---|---|
| **1** | [Build status](#1-build-status) | everyone |
| **2** | [Where this lives](#2-where-this-lives) | everyone |
| **3** | [What every number counts](#3-what-every-number-counts) | everyone |
| **4** | [How the screen behaves](#4-how-the-screen-behaves) | backend + design |
| **5** | [Overview Snapshot](#5-overview-snapshot) | backend |
| **6** | [View all sheet](#6-view-all-sheet) | backend + design |
| **7** | [Dues (Live)](#7-dues-live) | backend |
| **8** | [Bills Summary](#8-bills-summary) | backend |
| **9** | [Dues Breakdown](#9-dues-breakdown) | backend |
| **10** | [Overdue Breakup](#10-overdue-breakup) | backend + design |
| **11** | [Upcoming Dues](#11-upcoming-dues) | backend |
| **12** | [Deposit Dues](#12-deposit-dues) | backend |
| **13** | [Breakup by Stay Duration](#13-breakup-by-stay-duration) | backend |
| **14** | [Dues by Property](#14-dues-by-property) | backend |
| **15** | [What each number opens](#15-what-each-number-opens) | backend |
| **16** | [Who can see this](#16-who-can-see-this) | backend |
| **17** | [What each card shows when it is empty, healthy or broken](#17-what-each-card-shows-when-it-is-empty-healthy-or-broken) | design + copy |
| **18** | [What this screen is not](#18-what-this-screen-is-not) | everyone |
| **19** | [Build guidance](#19-build-guidance) | backend |
| **20** | [Open items](#20-open-items) | named owners |
| **21** | [Design file: what needs fixing](#21-design-file-what-needs-fixing) | design |
| | [Where the measured figures came from](#where-the-measured-figures-came-from) | anyone re-checking a decision |

**Building the backend?** Read 1, 3 to 15, 19, 20. **Working on the design?** Read 3, 4, 6, 10, 17, 21. **Just need the decisions?** Read 3, 4 and the appendix.

## 1. Build status

The screen is designed and mostly signed off in the design file, including the view-all sheet. The backend serves every card as a scaffold: each block is wired and reachable but answers with placeholder numbers while the real calculations wait to be written. Nothing in this sheet describes a live defect; it describes what each number must do once built. The dues list screen that every tap lands on already exists and works.

If the build cycle runs short, cut in this order: Breakup by Stay Duration first, then Deposit Dues, then Upcoming Dues. The six core cards (Overview strip, Dues (Live), Bills Summary, Dues Breakdown, Overdue Breakup, Dues by Property) and their overflow sheets ship together.

## 2. Where this lives

Manager App → property → **Financial** tab → **Dues** sub-tab. The suite tab row:

|---|---|
| Financial | Dues · Collection · Expense |
| People | Leads · Bookings · Tenants · Old Tenants |
| Inventory | — |
| Complaints | — |

The entry point is the widget at the top of the Financial tab. Screens drawn around it in the design file are reference context only.

## 3. What every number counts

A **due** is one bill someone owes: a tenant, an amount still unpaid, a due date, a category. Where the screen's copy says "bill", it means a due.

| Term | Meaning |
|---|---|
| Overdue | The due date has passed. A due dated today is not yet overdue |
| Added by | Who raised the bill. Rent and late fines the system raises on schedule carry the manager's name |
| Deposit | Dues in the Security Deposit and Caution Money categories |
| Short stay, long stay | Every tenant is one or the other. There is no third state |
| Money on this screen | Rupees, shown short: ₹23K, ₹2.45L |

### The base rule

A due counts if it is unpaid, at least ₹1, and belongs to a tenant who is living here, has an eviction raised or approved, or is a booking. Bookings count whether or not the booking has been approved: the bill was raised, so the money is owed. The one forecast on this screen, Projected Due (§5), is the exception and counts confirmed bookings only. **Old tenants are excluded from every number on this screen except two places**: the Old tenants row in Dues Breakdown (§9), and the held-deposit line under Deposit Dues (§12). Those are their only homes; they are never blended into any total. Refunded dues and dues written off as a loss are excluded from every number on this screen, including Bill Due: they are neither owed nor collected.

### Category

The due's type as the property configured it: Rent, Electricity, Security Deposit, and so on. Types are free-named, so every category chart shows the top few by amount and folds the rest into **Others**, a bottomsheet listing every remaining category with its own total. Names that differ only by capitals group as one.

### Upcoming and Projected

Dues the property's recurring setup will raise but has not raised yet. A forecast, never blended with real dues, and nothing about it ever invents an event.

### Eviction: raised, then approved

An eviction has two states and this screen shows both, separately. **Eviction pending** is one that has been raised and nobody has approved yet. **Eviction approved** is one a manager has accepted, which is what puts a confirmed leaving date on record. A withdrawn notice stops counting immediately.

Eviction here means any eviction, whoever raised it. A tenant giving notice and a manager asking someone to leave are the same event to this screen; who raised it is recorded separately.

**Approved carries no date limit.** A tenant whose approved date has passed but who has not physically moved out is still living here, still being billed for rent and everything else, so they stay in Eviction approved. That is a small group and an anomaly, but their money is real.

**This screen never shows the two states as one bar.** On Tenants the one group covering both at once is called Under eviction (D28; "Under notice" until 25 August 2026). Here the two states are separate bars, so each is named for what it is.

### Words to be careful with

|---|---|
| **Active, no eviction** | On this screen every due sits in exactly one tenant-status bar, so this bar excludes anyone with an eviction raised or approved. On the Tenants screen, Active includes them, because there it is a layer and not a slice. The label says so on its face. |
| **Received** | Payments against the bills in the window, whenever they arrived. Payment records live on the Collection screen, where Received means money that arrived inside the window. The two will not match for the same window, and that is intended: this screen measures how much of what was billed has come in, Collection measures what came in. |
| **Late Fine** | Auto-raised. Expect it near the top of every category chart: that is real money, not a bug. |
| **Others** | Every overflow list on this screen, categories and added-by alike, opens the same way: a bottomsheet naming everyone or everything left out, each row drillable to its own filtered list. One pattern, used everywhere it applies. |

## 4. How the screen behaves

### The time filter

Options: **This Month (default) · Last Month · Current FY · Custom · All Time.**

**Custom may end in the future.** There is no separate forward setting; a Custom window whose end date is after today is the forward view. Nothing on this screen is projected by it: a bill raised today with a due date next month is a fact with a date on it, so a forward window counts real records, not estimates. Bills that have not been raised yet stay where they belong, in Upcoming Dues (§11), never mixed in.

A window that ends after today changes three things and nothing else: no number carries a change chip, because a period that has not finished has nothing of the same shape to compare against; a number that counts events that have happened shows a dash with a one-line reason instead of a figure; and the forecast card extends to the chosen date. Every block that does not follow this filter says so on its face, in a word in its title such as Live or as of today or Forecast, or in its own dropdown.

Cards with their own date dropdown follow three rules: same options as the top filter, the top filter pulls every card back in line, and one card can be deliberately set aside until the top filter next changes. The filter stays where the manager put it while the app is open; a fresh launch opens on the default. Coming back from a drill returns to the screen as it was left, with fresh numbers. A day runs midnight to midnight, India time.

Two of the suite's three kinds of number on this screen:

| Kind | Meaning |
|---|---|
| **Live** | Always the current snapshot. No filter setting changes it. Says "as of today" on its face |
| **Time-scoped** | Counts dues whose due date falls inside the window the filter picks |

Worked example, today being 8 August. A ₹6,300 rent due dated 28 July, still unpaid:

| Number | Kind | Shows |
|---|---|---|
| All Time Dues | Live | counts it |
| All Past Dues | Time-scoped, window fixed | counts it, its due date sits before this month |
| This Month's Due | Time-scoped, window fixed | does not count it |
| Dues (Live) gauge | Live | counts it under Overdue |
| Overdue Breakup | Live | 11 days overdue, sits in the 8–14 bucket |
| Dues Breakdown on This Month | Time-scoped | does not count it |
| Dues Breakdown on Last Month | Time-scoped | counts it |
| Dues Breakdown on a Custom window ending next month | Time-scoped | counts it only if the window's start is on or before 28 July |

### What every number does on every filter setting

| Number | This Month · Last Month · Current FY · Custom · All Time | Custom ending after today |
|---|---|---|
| Overview strip, 6 tiles | Each tile keeps its own fixed window; the filter does not change it | Unchanged |
| Dues (Live) gauge | As of today | As of today. What is outstanding right now cannot be projected forward without guessing what gets paid |
| Bills Summary | Counted inside the window | Counted inside the window. Bill Due is what is already billed for that period; Received is what has already been paid against it, in advance. The gap reads "not yet collected, and not yet due" rather than "still to collect", and the info icon says so |
| Dues Breakdown, all 3 views | Counted inside the window; its own dropdown can pick a different one | Bills due inside the window, split the same three ways |
| Overdue Breakup | As of today | As of today. Nothing is overdue in the future |
| Upcoming Dues | Stays on its own forward window, tomorrow to month end. A forecast has nothing to say about a period that is over, and it says Forecast on its face | Tomorrow to the chosen end date |
| Deposit Dues | As of today | As of today |
| Breakup by Stay Duration | Counted inside the window | Bills due inside the window |
| Dues by Property | Counted inside the window | Bills due inside the window. The planning view: which property has most landing, and when |

Nothing on a forward window ever invents an event. Every number there counts bills that already exist with a future due date, except Upcoming Dues, which is labelled a forecast on its face.

### Periods that have not finished

An unfinished period compares against the same elapsed days of the previous period, marked as unfinished on the chip: "▼12% vs same point last month." The note drops away once the period completes.

### Change chips

Exactly two numbers carry one: **This Month's Due** and **Current FY Dues**. Up is bad here: a rising chip shows red, a falling one green, an unchanged number shows a neutral chip. No other number on this screen carries a chip, and nothing carries one on a window that ends after today. The other four tiles have no previous period of the same shape, and a chip on a forecast would be a guess.

**When there is nothing to compare against, there is no chip.** A property in its first month shows the number alone, never a neutral chip and never a green one.

A rising chip means dues grew. It cannot say whether collections slipped or the property simply filled up and billed more people. Both read red, and the screen does not claim to tell them apart.

### Where the comparison number comes from

The chip needs one figure the screen does not store: what was owed at the same point last month. Nothing keeps a daily record of dues, and nothing needs to.

Every bill carries the date it was due and, once paid, the date it was paid. That is enough to look back:

> **What was owed on a past date** = every bill due on or before that date that had not been paid yet on that date. A bill counts if it is still unpaid today, or if it was paid after that date.

Part payments need no special handling. When somebody pays half a bill, it becomes two bills: the unpaid part, and a paid part carrying the same due date and the date it was paid. Both are ordinary bills, so the rule above already covers them.

**The trap this avoids.** Asking what is still unpaid from last month gives the wrong answer, because a month of collections has happened since. That figure is roughly a quarter too small, so a property collecting exactly as well as before would show red every month. See the measured figures.

**What the past figure means.** It describes the period as we understand it today, not as the screen looked back then. A bill since deleted does not count. A bill added later, but dated to that period, does count.

### When a number is red

Red only where somebody can be held to it. Overdue money is red. Money not yet due is plain. A property onboarded this morning must not open red.

### Loading, failure, sorting, entry

Each card loads with a skeleton and fails alone with "Couldn't load this" and Retry, never a healthy message, never a zero. Property-wise components sort highest to lowest. The entry point is the widget at the top of the Financial tab.

## 5. Overview Snapshot

Six tiles, always visible, none moves with the filter.

| Tile | Counts | Kind |
|---|---|---|
| All Time Dues | every unpaid due, any due date | Live |
| All Past Dues | unpaid dues due before the 1st of this month | Time-scoped, window fixed |
| This Month's Due | unpaid dues due from the 1st through today | Time-scoped, window fixed |
| All Future Dues | unpaid dues due from tomorrow onwards | Live |
| This Month's Projected Due | what the recurring setup will raise before month end, every configured type, not rent alone. Counts living tenants and confirmed bookings only; a booking still awaiting approval is not reliable enough to forecast against, even though any bill already raised for it counts everywhere else on this screen | Forecast |
| Current FY Dues | unpaid dues due since 1 April | Time-scoped, window fixed |

Every due sits in exactly one of All Past, This Month's Due and All Future, and the three add up to All Time. Chips per §4: This Month's Due and Current FY Dues only.

## 6. View all sheet

Opens from the strip header. Restates the six Overview tiles, then breaks this month's dues down by category and by stay length. Chips per §4: This Month's Due and Current FY Dues only, same as the strip. Every row that can open a list does, arriving filtered to exactly that row's dues, the same as if tapped from its card. The Projected row opens the forecast explainer, not a list.

**The six tiles, restated**

| Row | Meaning |
|---|---|
| All Time Dues | Same as the tile (§5) |
| Past Dues | Same as All Past Dues (§5) |
| [Month] Dues | Same as This Month's Due (§5), the actual month name on its face |
| Total Future Dues | Same as All Future Dues (§5) |
| [Month]'s Projected Dues | Same as This Month's Projected Due (§5), the actual month name on its face. Counts every configured due type, never rent alone |
| FY Dues | Same as Current FY Dues (§5), the actual financial year on its face |

**This month, by category**

| Row | Meaning |
|---|---|
| [Month] [Category] Dues | This month's top categories by unpaid amount, one row each, the real month and category name on every row ("May Rent Dues"). The rest fold into Others, same rule as Dues Breakdown (§9) |

**By stay length**

| Row | Meaning |
|---|---|
| Total Short Stay Dues | This month's unpaid dues from short-stay tenants |
| Total Long Stay Dues | This month's unpaid dues from long-stay tenants |

Hidden together when the property has no short-stay tenants, the same rule as the card these restate (§13).

## 7. Dues (Live)

The gauge under the strip. Says "as of today" on its face.

| Row | Meaning |
|---|---|
| Total Dues | the same figure as All Time Dues, one computation shown twice |
| Overdue | the portion whose due date has passed, red |
| Due Today | the portion due today |
| Due This Week | the portion due in the next 7 days, counting from tomorrow, the face shows the dates ("24 Aug–30 Aug") |
| Due Later | the portion due after that week, the face names the last day of the week ("After 30 Aug") |

Every due sits in exactly one slice, and the four add up to Total Dues. The week starts tomorrow, not today, because Due Today is already its own slice.

## 8. Bills Summary

Time-scoped pair: how billing went in the window.

| Row | Meaning |
|---|---|
| Bill Due | every due that came due inside the window, whether since paid or still unpaid |
| Received | money collected against those same dues, whenever it arrived, not only inside the window |

Bill Due is not the same figure as This Month's Due on the default window: This Month's Due counts only what remains unpaid, Bill Due counts everything that came due including what has since been paid. That gap is the card's purpose, it shows how much of what came due has actually been collected.

Received deliberately ignores when the money arrived. Look at Last Month today and most of those bills were paid in the weeks after they fell due; counting only payments made inside the window would show a month as badly collected when it was in fact collected. This is the one place on the suite where Received does not mean money that arrived in the period, so the card carries an info icon saying so.

## 9. Dues Breakdown

Time-scoped, own dropdown. Three views of the window's dues, switched by a toggle.

The three tabs do not add up to the same total, and they are not meant to. Category and Added by count the screen's base pool, which leaves out old tenants. Tenant status widens to include them, because this is their one home on the screen (§3).

**Category.** Top categories in the window by unpaid amount, the rest in Others. Expect Rent and Late Fine to lead nearly everywhere. Top plus Others always sums to the window's total.

**Tenant status.** Five bars splitting the window's money by where the payer stands today. Every due sits in exactly one bar:

| Row | Meaning |
|---|---|
| Active, no eviction | living here, nothing raised against them |
| Eviction pending | living here, an eviction raised, nobody has approved it yet |
| Eviction approved | living here, an eviction a manager has approved. Includes the few whose date has already passed but who have not physically moved out: they still live here and are still being billed |
| Bookings | bookings with dues, whether confirmed or still awaiting approval |
| Old tenants | moved out still owing. Their only home on this screen |

**Added by.** The top four contributors by amount, the rest in an Others overflow sheet, listing each remaining person or RentOk with their total and bill count. RentOk's own row is shown only when it has an unpaid amount, never given a row with nothing to count. Fewer than four real contributors: show only the real ones, no empty Others row.

The categories that surface here can differ from the categories in Overdue Breakup (§10). That is expected: this view counts every unpaid due, that one counts only the overdue ones, two different pools that can each have their own top few.

## 10. Overdue Breakup

Live, says "as of today" on its face, no dropdown of its own. Two views over everything overdue right now.

**By ageing.** Five buckets by days past the due date: **1–7 · 8–14 · 15–21 · 22–90 · 90+**. The first three are the chasing steps: remind, follow up, escalate. 22–90 is serious and still recoverable. 90+ is a review-for-write-off pile, and its bar being the biggest is the message, not a rendering fault.

**By category.** Top categories of the overdue pool, the rest folded into Others, the same overflow bottomsheet pattern as §9, each remaining category drillable to its own filtered list. Top plus Others always sums to the overdue total.

## 11. Upcoming Dues

"From today onwards": what the recurring setup will raise from tomorrow through month end, or through the chosen end date when the filter is a Custom window ending after today, grouped Rent · Food · Others, laid out by due date. This is the one number on the screen that is a forecast rather than a record, and it says so on its face. Others is the fold of every further configured type, not a fixed third group. Only rent configured, only rent shows. If the recurring setup can't be read cleanly, the card shows zero rather than a guess, never an estimate.

Built. Its taps open an explainer, no real dues exist to list. Shipped rent-only at first; the other configured types are being added so the card's bars add up to the Projected Due tile (§5).

## 12. Deposit Dues

Live running balance, not a period.

| Row | Meaning |
|---|---|
| Received | deposit collected and still held from tenants living here or booked. Money returned or adjusted no longer counts |
| Due | deposit still unpaid |
| Total Deposits | Received plus Due, the gauge's headline |

Under the gauge, one more line, never inside it:

| Line | Meaning |
|---|---|
| Held for tenants who have left | deposit collected from people who have since moved out and not yet returned or adjusted. Old tenants' money, so it sits outside the gauge and is never blended into its total (§3) |

Received opens the Collection screen's list filtered to deposit categories (§15).

## 13. Breakup by Stay Duration

Time-scoped. Short stay and long stay, two bars that always sum to the window's total. The card shows only when the property has at least one short-term tenant; on the rest of the platform it would only ever restate the total.

## 14. Dues by Property

Time-scoped. Renders only when the account has more than one property with data. Ranked highest to lowest, and **every row carries its share of the account total**: the share is what turns a ranking into a staffing decision.

## 15. What each number opens

<details>
<summary><strong>Expand:</strong> the drill rules, what travels with a tap, the full tap matrix with arrival filters, and the list filters that still have to be added</summary>


### The rules

A drill filters a list, it never re-scopes the screen: the screen behind stays exactly as it was. The destination follows the record's kind: **a tap lands on the list where its records live, even when that list sits on a sibling tab**, so the two Received figures open the Collection screen's list. The destination opens on the same window and the same properties, and the back control names this screen. Records add back to the number tapped. The forecast numbers open an explainer, nothing real exists to list.

**Every overflow, one pattern.** Category Others and Added By Others both open a bottomsheet naming everyone or everything left out of the top few shown, each with its own total. Every row inside that bottomsheet is itself drillable, opening the dues list filtered to that one category or that one person.

### When the window changes what a tap shows

| The screen's window | What travels to the list |
|---|---|
| A Live number, on any window | Nothing, the list opens as of today |
| A period, running or finished | The window travels as a due-date range, and the list shows tenants as they are today, naming any difference on arrival |
| Custom ending after today | The forward window travels as a due-date range. These are real bills, so they open a real list |
| A forecast number | No travel, the explainer opens instead. Nothing has been raised yet to list |

### The tap matrix

| You tap | What opens | Arriving filtered to | Ready? |
|---|---|---|---|
| **Overview strip** | | | |
| All Time Dues | dues list | all unpaid dues | ✅ |
| All Past Dues | dues list | due before this month | ✅ |
| This Month's Due | dues list | due 1st through today | ✅ |
| All Future Dues | dues list | due after today | ✅ |
| This Month's Projected Due | forecast explainer | — | ✅ |
| Current FY Dues | dues list | due since 1 April | ✅ |
| **View all sheet** | | | |
| tile rows | as their tiles above | ″ | as above |
| category rows | dues list | that category, this month | ✅ |
| stay rows | dues list | that stay type | ❌ stay-type filter to add |
| **Dues (Live)** | | | |
| Overdue | dues list | due before today | ✅ |
| Due Today | dues list | due today | ✅ |
| Due This Week | dues list | due inside the shown dates | ✅ |
| Due Later | dues list | due after the shown date | ✅ |
| **Bills Summary** | | | |
| Bill Due | dues list | came due in the window | ✅ |
| Received | Collection list | payments in the window against those dues | ⚠ Collection list must accept the window |
| **Dues Breakdown** | | | |
| category bars | dues list | those categories in the window | ✅ |
| category Others | overflow sheet, then dues list per row | remaining categories, then that one category | ✅ |
| Active, no eviction | dues list | that tenant state | ❌ tenant-state filter to add |
| Eviction pending | dues list | that tenant state | ❌ tenant-state filter to add |
| Eviction approved | dues list | that tenant state | ❌ tenant-state filter to add |
| Bookings | dues list | that tenant state | ✅ |
| Old tenants | dues list | that tenant state | ❌ tenant-state filter to add |
| added-by bars | dues list | that person's bills in the window | ✅ |
| added-by Others | overflow sheet, then dues list per row | remaining people, then that one person's bills | ✅ |
| **Overdue Breakup** | | | |
| ageing buckets | dues list | the bucket's due-date range | ✅ |
| category bars | dues list | those categories, due before today | ✅ |
| category Others | overflow sheet, then dues list per row | remaining categories, then that one category, due before today | ✅ |
| **Upcoming Dues** | forecast explainer | — | ✅ |
| **Deposit Dues** | | | |
| Due | dues list | deposit categories, unpaid | ✅ |
| Received | Collection list | deposit payments still held | ⚠ Collection list must accept deposit categories |
| **Breakup by Stay Duration** | | | |
| either bar | dues list | that stay type in the window | ❌ stay-type filter to add |
| **Dues by Property** | | | |
| property row | dues list | that property, the window | ✅ |

Every ❌ is a filter over data that already exists, never history to start keeping.

### What the lists can already do, and what has to be added

The dues list filters today by property, due-date window, who added the due, category, defaulter group, tenant type (active and booking only), and free search.

| New filter to add | On which list | Which numbers wait on it |
|---|---|---|
| Tenant state, four values matching §9's bars | dues list | Three of the four tenant-status bars |
| Stay type | dues list | Both stay bars, two view-all rows |
| A due-date range that may end after today | dues list | Every drill taken while the screen is on a Custom window ending after today |

The forward window is the cheapest of the three. The date control both apps share already understands forward ranges, and its custom two-date picker already accepts future dates, so the dues list can land on a future window today. What it does not do is offer one as a named option, which is what a manager reaching for it would expect. See build guidance for the trap waiting there.

### What the destination says when you arrive

The list names the slice it arrived filtered to, shows the active filter, and names that filter in its empty state.
</details>

## 16. Who can see this

**Anyone who can view dues.** The suite rule: each analytics tab follows the permission of the records it describes. Dues and Collection open and lock together; Expense is separate. No partial state.

Whoever lacks it sees the lock: *"Analytics Restricted, You don't have permission to view these analytics. Request access from your admin,"* with **Request Access**.

Narrowed property access counts only those properties, and the drill matches.

## 17. What each card shows when it is empty, healthy or broken

### The zeros, told apart

| Situation | What shows |
|---|---|
| Never set up: no tenants | *"No dues yet. Add your first tenant and this page fills in."* with **Add tenant** |
| Onboarding: tenants added, nothing billed yet | *"No bills raised yet. Bills you raise appear here."* |
| In-window zero on a time-scoped card | the card draws zero, plain, no message |
| Everything paid | good news, see Healthy below |
| Upcoming with nothing configured | *"Nothing set to auto-raise."* |
| Forward window, nothing due in it | *"Nothing due between 24 Aug and 30 Sep."* Named with the window the manager actually picked, and good news, not a gap |
| Failed | "Couldn't load this" with Retry, never a healthy message, never a zero |

### Healthy: good news, no CTA

| Card | Reads |
|---|---|
| Dues (Live) at zero with tenants present | *"Every bill is paid."* |
| Overdue Breakup at zero | *"Nobody is overdue."* |
| Dues Breakdown at zero | *"Nothing outstanding to break down."* |
| Deposit Due at zero | *"No deposit pending."* |
| Bills Summary, window with nothing billed | *"No bills came due in this window."* |
| Dues by Property, no property carrying dues | *"No property has outstanding dues."* |

Zero unpaid dues is real but rare, about one live property in fourteen, so the healthy states must exist and must not be dressed as setup.

### Empty and hidden

Breakup by Stay Duration hides when the property has no short-term tenants. Dues by Property renders only for multi-property accounts. Hidden is hidden: no ghost card, no explainer.

## 18. What this screen is not

- **Not a collections screen.** Payments live on Collection; this screen only pairs Received beside what was billed.
- **Not a chase list.** Tenant-by-tenant follow-up happens on the dues list the numbers open.
- **Not an income forecast.** Projected counts bills that will be raised, never money promised.
- **Not a ledger of old tenants.** Their dues have one row (§9) and the rest of that story lives on Old Tenants.

## 19. Build guidance

1. **Totals must survive junk amounts.** A bill of ₹1 crore or more is data entry gone wrong, not money. *Test it:* a property holding one ₹500 crore bill reports totals matching its real bills.
2. **A part-paid due counts its remaining amount.** *Test it:* pay ₹4,000 of a ₹10,000 due; every total moves by exactly ₹4,000.
3. **Old tenants appear in one row only.** *Test it:* a moved-out tenant owing ₹5,000 raises the Old tenants bar by ₹5,000 and changes no tile, no gauge, no other bar.
4. **Every due sits in exactly one tenant-status bar.** *Test it:* the four bars add up to the window's total exactly, and no due appears in two bars.
5. **Category names that differ only in capitals count as one.** *Test it:* two types differing only by capitals chart as one bar.
6. **Ageing runs from due date to today.** *Test it:* a bill due the 1st sits in 1–7 on the 7th and in 8–14 on the 9th.
7. **A due dated today is not overdue.** *Test it:* it appears in Due Today and in no ageing bucket.
8. **Deposit Received means still held.** *Test it:* refund a ₹10,000 deposit; Received drops by ₹10,000 the same day.
9. **Received counts only payments against the window's dues.** *Test it:* a payment against last month's bill moves nothing in this month's Bills Summary.
10. **Future windows must reach the list.** *Test it:* tap Due Later; the list opens showing bills due after the shown date.
11. **Chips compare unfinished periods fairly.** *Test it:* on the 8th, the This Month's Due chip compares against the first 8 days of last month, marked as unfinished, and drops the mark once the month completes.
12. **Chip direction is up-is-bad on this screen.** *Test it:* a rising This Month's Due shows red; a falling one shows green.
13. **The comparison figure counts what was unpaid then, not what is unpaid now.** *Test it:* a ₹10,000 bill due the 5th of last month, paid in full on the 20th, counts its whole ₹10,000 towards last month's comparison figure and nothing towards this month's.
14. **A part-paid bill counts in full for a date before the payment.** *Test it:* a ₹10,000 bill due the 5th of last month that took ₹4,000 on the 20th contributes ₹10,000 to the figure for the 8th, and ₹6,000 to today's.
15. **No previous period, no chip.** *Test it:* a property in its first month shows the number with no chip at all, not a neutral one.
16. **A corrected bill reads at its corrected amount.** Editing a bill's amount after the fact leaves no record of the earlier one, so past figures use the new amount and read slightly small. Rare, and always in that direction. *Test it:* raise a bill, edit its amount down, and confirm nothing else on the screen breaks.
17. **The stay card hides rather than restating.** *Test it:* a property with zero short-term tenants shows no stay card.
18. **The property card needs two properties with data.** *Test it:* a single-property account never sees it; a two-property account sees both rows with shares summing to 100%.
19. **Cross-tab arrivals carry their slice.** *Test it:* tap deposit Received; the Collection list opens on deposit payments, back control names this screen.
20. **A window labelled forward must look forward.** *Test it:* set the list to a "Next 30 days" option on a property with bills due next month; the list shows those bills. Seeing last month's bills instead is the failure this test exists to catch.
21. **A forward window counts bills that exist, never bills that might.** *Test it:* on a Custom window ending 60 days out, the breakdown cards count only bills already raised with due dates in that window; anything the recurring setup has yet to raise appears in Upcoming Dues and nowhere else.
22. **Bills Summary stays in on a forward window, and its info icon changes what the gap means.** *Test it:* set a Custom window to next month; the card shows what is already billed and what is already paid in advance, and the info icon reads "not yet collected, and not yet due", not "still to collect".
23. **Every category chart shows the top few plus one Others row.** *Test it:* a property with six categories shows the top four or five and one Others row, and the top rows plus Others sum to the card's total.
24. **The tenant-status sub-breakdown that once sat inside the gauge is not built.** It duplicates §9's Tenant status view; that view is its only home.
25. **No narrowed self-added-tenant view is built for this screen.** Product has confirmed nobody is granted that permission; if that changes, this screen needs the same scoping the dues list already has, see open item 1.

## 20. Open items

1. **Engineering.** A permission exists that narrows a member's view to only the tenants they added. Product says nobody is granted it. Confirm, because if anyone holds it, two people can see different totals on the same property with nothing on screen saying so.
2. **Engineering.** Dues worth about ₹2.8 crore carry a creator category that has no display name. Added By needs a name for it before that view ships.
3. **Engineering.** Some drill routes behind this screen's numbers have not been confirmed to carry the same permission check as the rest of the screen. Confirm before launch.
4. **Engineering.** The dues list currently returns everything in one page with no real pagination. Decide whether that holds once this screen sends it real traffic.
5. **Product.** The Added By overflow's row count is specified here as four, including RentOk's own row when it has an amount. Worth a final confirm before build, a different count was mentioned in passing this session.

## 21. Design file: what needs fixing

<details>
<summary><strong>Expand:</strong> 28 numbered design-file fixes, grouped Wrong / Missing / Remove / Decide, plus Also found</summary>


**Wrong**

1. The tenant-status view draws four bars with one labelled "Under Eviction". It is five bars now (§9): **Active, no eviction · Eviction pending · Eviction approved · Bookings · Old tenants**. The design's "Under Eviction" was the right idea with the wrong shape: it has to be split into pending and approved.
2. Seven cards' empty states carry copy from the Complaints module ("You're all caught up! New maintenance requests…"): Dues (Live), Bills Summary, Dues Breakdown, Overdue Breakup, Upcoming Dues, Deposit Dues, Dues by Property. §17 holds the replacement copy per card.
3. The Overdue Breakup card's healthy copy says "No Defaulters this month". The card is Live now: *"Nobody is overdue."*
4. The forecast card is titled "Upcoming Rent (to be added)". It counts every configured type: **Upcoming Dues**. Its chart also carries the axis label "Overdue Timeline", left over from a copied chart; it is a forward-looking card and needs its own label.
5. Change chips are drawn on four tiles (May's Due, May's Projected Due, All Past Dues, Current FY Dues). Only two carry a chip: This Month's Due and Current FY Dues.
6. **Change chip colours are inverted.** A rising, worse number is drawn green; it must be red. Up is bad on this whole card.
7. The gauge and Deposit Dues headlines show wrongly written amounts ("₹15,00,000 L", a full number plus a stray unit letter). Format as ₹15L. The View all sheet writes nearly every one of its own amounts the same wrong way.
8. The global filter chip at the top of the screen reads "Today". It should show one of the five locked options, defaulting to This Month.
9. The View all sheet draws a change chip on Past Dues, a number §4 says should carry none. The arrow is also the wrong colour, red for a fall, the same inversion as item 6.
10. The Restricted-state mockup's gauge shows a different Due Later window ("08 Aug–31 Aug") than the live card ("After 08 Aug"). Sync the two.
11. Dev notes left on the canvas still name tenant-status labels this sheet has since replaced ("Active, not under notice" among them). Update or remove them so they stop reading as current guidance.

**Missing**

12. The ageing view draws four buckets. The spec is five: 1–7 · 8–14 · 15–21 · 22–90 · 90+.
13. Property rows need their share-of-total figure.
14. Change chips need a neutral state for an unchanged number.
15. Live cards need "as of today" on their face.
16. Every card carries an info icon and no info copy is written anywhere. Either write the copy per card or drop the icons.
17. The Category Others and Added By Others overflow sheets need their bottomsheet drawn wherever a category view exists, not only for Added By.
18. The **Custom** date picker allowing an end date after today, and the screen's state on such a window: the Live cards still saying "as of today", no chips anywhere, Upcoming Dues extended to the chosen date.

**Remove**

19. The Overdue Breakup card's own dropdown control, wherever it still appears in the file. The card is Live.
20. The switched-off tenant-status block inside the gauge. Superseded by the Dues Breakdown view.
21. Two loose cards parked outside the phone frames in the Dues section: a Deposit Dues card and a duplicate Dues Breakdown card. Housekeeping.
22. Hidden leftover "Paid by / Paid to" labels sitting inside the Dues by Property card, left over from a different card.

**Decide**

23. The gauge when one slice is nearly everything: minimum sliver width or value-first treatment.
24. The 90+ ageing bar will dominate on most properties: whether the bar scale needs a treatment or the labels carry it.
25. Whether the View all sheet's Duration segments hide entirely when the property has no short-stay tenants, matching the card they restate (§13), or show a zero row instead.

**Also found**

26. The View all sheet's Projected row is labelled "May's Projected Rent Dues". It counts every configured due type, never rent alone, the same naming slip already flagged on the Upcoming card (item 4).
27. A bottomsheet sitting in the Dues section of the file holds an unrelated screen's content, a property's staff list. Misfiled, not part of this screen.
28. Every Overview tile carries two stacked change-chip layers, one hidden, one shown. Never wrong on screen, but worth flattening to one before build so nobody wires both by mistake.
</details>

## Where the measured figures came from

<details>
<summary><strong>Expand:</strong> 15 production figures and the decision each one settled</summary>


Measured on production, 7 August 2026 evening, amounts in rupees. Totals exclude test properties and bills of ₹1 crore and above (22 such bills sum to an impossible figure and would poison every headline).

| Measured | Result | What it decided |
|---|---|---|
| Unpaid dues, live tenants and bookings | 3.56 million dues, ₹3,294 crore | the scale every total must survive |
| Bills already raised with a due date more than a week out | 8,703 dues, ₹28.5 crore | The forward window: real money with future dates that no past-period setting could reach |
| Bills due in the first eight days of last month: owed at the time, against still unpaid today | ₹173.0 crore against ₹133.5 crore, 23% smaller | the comparison figure has to be rolled back, never read off today's unpaid bills |
| Paid bills with no payment date recorded | 74 out of 6.56 million, 0.001% | the roll-back rests on the payment date, and it is there |
| Bills split by a part payment | 186,716 since January, 37,510 in the last 30 days | part payments leave both halves on file, so they need no special handling |
| Share already overdue | 99.5% of unpaid money | the gauge's dominant-slice treatment (§21.23) |
| Ageing: 22+ days | 92% of overdue money | the fifth bucket: 22–90 split from 90+ |
| Old tenants' unpaid dues | 0.23% of the total, median due ₹10 | one row is enough, never a full card |
| Category mix | Rent 67.5% + Late Fine 29.8% = 97.3%; Electricity 0.18%; Mess absent from the top 12 | dynamic top-N, Late Fine surfacing is real |
| Distinct category names | 785, with duplicates differing only in capitals | names differing only in capitals must count as one |
| Added-by mix | one creator code holds 95.9% of dues | expect one dominant bar; the card serves the manual remainder |
| Unnamed creator code | ₹2.8 crore of dues | open item 2 |
| Stay type fill | zero unset across 1.08 million tenants; short-term 0.46% of active | no unbucketed case; the stay card hides at zero short-term |
| Recurring setup beyond rent | 56% of active tenants have at least one non-rent configured type | the Upcoming card has real material |
| Bills Summary pair, August | ₹330 crore billed, ₹128 crore received a week in | the card is viable and meaningful |
| Properties with zero unpaid dues | 7.2% of live properties | healthy states are real but rare (§17) |
| Accounts with dues on one property only | 94% | the property card renders multi-property only |
| Top property's share, multi-property accounts | median 60.4% | share-of-total is a requirement, not a maybe |
</details>
