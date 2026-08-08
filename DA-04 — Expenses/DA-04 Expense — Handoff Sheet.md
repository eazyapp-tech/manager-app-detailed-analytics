---
title: DA-04 Expense — Handoff Sheet
date: 2026-08-08
tags: [rentok, expenses, financials, detailed-analytics, handoff]
status: v3 · uplifted to the suite template · definitions carried from v2
owner: Sanchay
---

# Expense — Handoff Sheet

Everything on the Expense analytics screen: what each number means, what window it covers, what happens when it is tapped, and what it shows when there is nothing to show.

---

## What is in here

| | Section | For |
|---|---|---|
| **1** | [Build status](#1-build-status) | everyone |
| **2** | [Where this lives](#2-where-this-lives) | everyone |
| **3** | [What every number counts](#3-what-every-number-counts) | backend |
| **4** | [How the screen behaves](#4-how-the-screen-behaves) | backend + design |
| **5** | [Overview](#5-overview) | backend + design |
| **6** | [View all sheet](#6-view-all-sheet) | backend + design |
| **7** | [Expense Breakdown](#7-expense-breakdown) | backend + design |
| **8** | [Top Payers and Vendors](#8-top-payers-and-vendors) | backend + design |
| **9** | [Expense Trend](#9-expense-trend) | backend + design |
| **10** | [Expenses by Property](#10-expenses-by-property) | backend + design |
| **11** | [What each number opens](#11-what-each-number-opens) | backend |
| **12** | [Who can see this](#12-who-can-see-this) | backend |
| **13** | [What each card shows when it is empty, healthy or broken](#13-what-each-card-shows-when-it-is-empty-healthy-or-broken) | design + copy |
| **14** | [What this screen is not](#14-what-this-screen-is-not) | everyone |
| **15** | [Build guidance](#15-build-guidance) | backend |
| **16** | [Open items](#16-open-items) | everyone |
| **17** | [Design file: what needs fixing](#17-design-file-what-needs-fixing) | design |

**Building the backend?** Read sections 1 to 12, then 14 to 16, plus the measured figures at the end. Start with Build guidance before writing anything; it holds the traps.

**Working on the design?** Sections 4 to 10, section 13, and section 17, which collects every fix in one list.

**Just need the decisions?** Sections 1, 14 and 16, plus the measured figures at the end.

---

## 1. Build status

The screen is **not built**. The backend has one empty placeholder block for Expense that returns no numbers. Nothing here is broken, it is unwritten. Every number below has to be built from scratch.

The expense list this screen drills into **is** built. In the app, its filter drawer offers a **date range** and a **Paid to** filter today, plus search. The other arrivals this screen needs are listed in section 11, each named as a filter to add; every one of them is over data the system already records.

---

## 2. Where this lives

Manager app, property header, **Financial** tab, third sub-tab: **Dues · Collection · Expense**. The suite's full tab row: Financial (Dues · Collection · Expense) · People (Leads · Bookings · Tenants · Old Tenants) · Inventory · Complaints.

---

## 3. What every number counts

An **expense** is one recorded spend event: an amount, the date it was paid, a category, who paid it, and who received the money.

### What counts as spend

Every number on this screen counts money that left the business and was recorded as an expense.

- A deleted expense is not spend.
- A refund paid back to a tenant is **not** an expense, even though it is money going out. It belongs to refunds.
- Paying staff back for money they fronted is **not** new spend. It reduces what the business owes them.
- A deposit returned to a tenant **is** counted here when it was recorded as an expense rather than against an invoice.
- Rows with a negative or zero amount are left out of every total, so a stray minus figure cannot quietly shrink a real number.

### Which date decides the window

The date the money was **paid**. Not the date someone typed the expense in.

An expense dated in the future joins every total the day its date arrives. These are a handful of records at any time, almost all within the next week, and they need no screen of their own.

### Words to be careful with

⚠ **"Bill" means something different here than on Dues and Collection.** There, a bill is an invoice raised to a tenant, money coming in. Here it means the receipt or proof attached to an expense, money going out. This screen never uses "bill" to mean an expense record, and never uses it to count anything. Where the proof is meant, this document and the screen both say **"bill or receipt"** in full.

### The same total appears three times

Total Expense sits on the Overview, above the Expense Breakdown bars, and above the Top Payers and Vendors bars. That is deliberate, and each is an anchor for the card under it. **All three must always be the same number**, computed once and reused. Three cards each working it out their own way is how they drift apart, and a manager who spots two different totals on one screen stops trusting all of them.

---

## 4. How the screen behaves

### The time filter

Options: **This Month (default) · Last Month · Current FY · Custom · All Time.**

**There is no forward setting here, and Custom stops at today.** Dues and Collection carry Coming up because a bill exists before its date arrives. Spending does not: an expense is recorded after money has already left, so a forward window would have nothing to count.

Cards with their own date dropdown follow three rules: same options as the top filter, the top filter pulls every card back into line, and one card can be deliberately set aside until the top filter next changes. The filter stays where the manager put it while the app is open; a fresh launch opens on the default. Coming back from a drill returns to the screen as it was left, with fresh numbers. A day runs midnight to midnight, India time.

Two kinds of number, the suite's own two words:

| Kind | Meaning |
|---|---|
| **Live** | Always the current snapshot. No filter setting changes it. Says "as of today" on its face |
| **Time-scoped** | Counts expenses whose paid date falls inside the window the filter picks |

Expense has exactly one Live number, **Still owed to staff**, and no forecast: nothing on this screen describes the future.

Worked example, today being 8 August. A ₹6,300 electricity expense paid on 28 July:

| Number | Kind | Shows |
|---|---|---|
| Total Expense on This Month | Time-scoped | does not count it |
| Total Expense on Last Month | Time-scoped | counts it |
| Current FY Expense | Time-scoped, window fixed | counts it, July sits inside this financial year |
| Still owed to staff | Live | unchanged, unless staff fronted it and are still unpaid |
| Expense Trend | Time-scoped | counts it in July's bar; the chart keeps its own range |

### What every number does on every filter setting

| Number | This Month · Last Month · Current FY · Custom · All Time |
|---|---|
| Total Expense | Counted inside the window |
| Current FY Expense | Each tile keeps its own fixed window; the filter does not change it |
| Still owed to staff | As of today |
| Number of expenses | Counted inside the window |
| View all sheet | Counted inside the window, except the rows that state their own fixed window |
| Expense Breakdown, both tabs | Counted inside the window; its own dropdown can pick a different one |
| Top Payers and Vendors, both tabs | Counted inside the window; its own dropdown can pick a different one |
| Expense Trend | Each keeps its own range; the filter does not change it |
| Expenses by Property | Counted inside the window; its own dropdown can pick a different one |

Three things fall out of that grid that are not obvious from the rules above.

**On All Time, the chips hide**, whether All Time came from the top filter or from a card's own dropdown. There is nothing before "everything", so a chip comparing all history against a previous period has nothing fair to compare. Total Expense and Number of expenses show their number with nothing beside it there, and that is correct.

**On Current FY, the Current FY tile and Total Expense show the same number, side by side. That is accepted.** Hiding it would move every tile after it under the manager's thumb at the moment they change filters, and a row that reshuffles is worse than a number that repeats. The tile earns its place at the other settings by letting someone see the year next to the month in one glance. Collection makes the same call in the same situation, and the two tabs behave alike.

**On Last Month and on any past Custom range, the Current FY tile still includes months after the window being looked at.** That is intended, and it is why the tile carries "This financial year" on its face. Without that label a manager investigating April would read it as April's figure.

One more, on the trend's own control: its range options are **6, 12 and 24 months**, defaulting to 6. A trend needs enough bars to show a shape, so nothing shorter than six is offered, and All Time is not offered because a bar per month over a property's whole life is unreadable.

### Periods that have not finished

An unfinished period compares against the same elapsed days of the previous period, marked as unfinished on the chip: *"▲ 12% vs same point last month."* The note drops away once the period completes. On Custom, chips compare the same number of days immediately before the range. The unfinished period is marked wherever it appears, the chip and the trend chart's in-progress bar alike.

Where the previous period had no spend at all, show the rupee change and drop the percentage. A percentage measured from zero tells the manager nothing.

### Change chips

Direction is per number, not per screen:

| Number | Rising means | Chip |
|---|---|---|
| Total Expense | more money out | red up, green down |
| Current FY Expense | more money out. Its chip compares against the same point of the previous financial year | red up, green down |
| Number of expenses | more spend events recorded, neither good nor bad on its own | neutral grey |
| Still owed to staff | no chip. It is a balance, not a flow; "twelve percent more owed than last month" is not a sentence anyone acts on | — |

Nothing carries a chip on All Time. The neutral chip is a state the shared chip component does not have yet.

### When a number is red

Red only where somebody can be held to it. Nothing on this screen meets that bar: spending is a record, not an unmet obligation. The chip's red is direction, never alarm.

### There is no view toggle on this screen

Collection switches between Paid Date and Due Date, and Inventory switches between Bed and Unit. **Expense has nothing equivalent.** The Category / Payment Mode and Paid by / Paid to pairs are tabs inside a single card, not a mode that changes the whole screen. Each pair splits the same spend for the same window two different ways, so both sides of each pair always add to the same total, and nothing else changes when a tab is switched.

### Loading, failure, sorting, entry

Each card loads with a skeleton shaped like the card it becomes and fails alone with *"Couldn't load this"* and Retry, never a healthy message, never a zero. Retry refetches only that card. When every card fails it is the connection: one message, one Retry for the screen, reading *"Couldn't load this page. Check your connection."* Error copy stays plain everywhere: no apology, no cause, no codes. All Time is the heavy setting: on a property with years of history the joined breakdowns scan the lot, so the skeletons matter most there. Every list and every bar on this screen sorts highest to lowest; the placeholder data in the design file does not demonstrate this, so take the rule from here, not the file. The entry point is the Overview row at the top of the Expense sub-tab, and the screen-level states live on it: the in-window zero and the narrowed-access note both show there.

---

## 5. Overview

Four tiles across the top, scrolling sideways, with a **View all** link in the header.

| Tile | Counts | Kind |
|---|---|---|
| **Total Expense** | Everything spent in the window | Time-scoped |
| **Current FY Expense** | Everything spent since the start of the current financial year, April to March | Time-scoped, window fixed |
| **Still owed to staff** | Money staff paid out of their own pockets that the business has not paid back yet | Live |
| **Number of expenses** | How many expenses were recorded in the window | Time-scoped |

The two fixed tiles say so on their face, in the place the other tiles put their change chip: **Current FY Expense** reads *"This financial year"*, and **Still owed to staff** reads *"as of today"*. Without that, a fixed number sitting in a row of window numbers reads wrong to anyone scanning quickly.

**Still owed to staff is not spend, and never adds into any total on this screen.** It answers a different question: what does the business owe its own people right now. It is a running figure on purpose. A version that followed the filter would show zero for April simply because April's advances had since been settled, which reads as "nothing owed" when lakhs may be outstanding.

This matters more than it looks: **a quarter of all expenses are paid from staff's own money.** It is the most common way money leaves after collected funds.

**Total Expense and Number of expenses open the same list.** That is correct and not a mistake. They are two readings of one set of records, and the list shows both the count and the total at its head.

---

## 6. View all sheet

Opens from the **View all** link in the Overview header. A bottom sheet holding the numbers the four tiles have no room for.

| Row | Meaning |
|---|---|
| Total Expense | Everything spent in the window |
| Current FY Expense | Everything spent since April, whatever window is selected |
| Still owed to staff | Money staff fronted that has not been paid back, as of today |
| Number of expenses | How many expenses were recorded in the window |
| **Average per expense** | Total Expense divided by the number of expenses. Useful for spotting a month made of one very large expense versus a month made of many small ones. Hidden when there are no expenses |
| **Spend with no bill or receipt** | How much of this window's spend has no bill or receipt attached, shown as an amount and a share: *"₹8.2L of ₹9.8L, 84%"* |
| **Paid from collected money** | Spend that came out of money the business had already collected |
| **Paid from staff's own money** | Spend staff fronted personally. This is what creates the Still owed to staff figure |
| **Paid through FlexiPe** | Spend that left through a FlexiPe transfer |
| **Paid from petty cash** | Cash the business had already handed to staff. **Shown only when it has spend.** It is used on well under one percent of expenses, so a permanent row would suggest a distinction that barely exists |
| **Rows with a negative or zero amount** | How many, and what they add up to. **Shown only when there are any.** These are left out of every total on the screen, and this is the one place they appear, so the owner can see the totals were worked out on clean rows |

**The bill-or-receipt figure is money, never a count.** More than eight in ten expenses have no proof attached, because no screen has ever shown anyone that. "₹8.2L of ₹9.8L this month has no receipt" is a number an owner acts on before an accountant or a tax question forces the issue; a count of expenses is noise. Expect it to look alarming at launch and to fall as people respond. That fall is the point.

**None of the four fund rows can be worked out from the expense record itself.** Whose money paid for something is held in the team passbook, not on the expense. Three rules follow, and each one is a wrong number if missed:

- **An expense can be paid from more than one source.** Count it once in Total Expense and in Number of expenses; split only its amount across the fund rows. The four fund rows plus the small no-record remainder below always add back to Total Expense. If they do not, something is counted twice or dropped.
- **Deleting an expense leaves its passbook entry behind, reversed.** The reversal has to be subtracted from the fund rows. Skipping this overstates every fund row on any property that has ever deleted an expense.
- **Paying staff back creates its own passbook entry, and it is not spend.** It must never appear as a fifth source of money. If it does, the screen shows staff paybacks as though the business spent that money twice.

**A passbook record is missing on two kinds of expense, and they are treated differently.** FlexiPe expenses never have one, because FlexiPe writes its expense on a path that never creates it; they are identified another way and fill the FlexiPe row. The rest, a couple of hundred across every property, are ones where the passbook entry failed to write and the failure was swallowed. Those stay in Total Expense, sit outside all four fund rows, and their source is never guessed from who paid.

**There is no wider missing-data section, on purpose.** Expenses with no paid date number zero across every property; missing category, payee and payer names number in the low tens out of a hundred and fifty thousand, because the form already collects them. Nothing there earns a row.

**Empty states for this sheet:** where the window has no spend, the sheet still opens with every amount at ₹0 rather than refusing to open; Average per expense hides, and the two conditional rows, petty cash and negative-or-zero, hide at zero. Where nothing is owed to staff, that row reads *"Nothing owed to staff right now."*

---

## 7. Expense Breakdown

Two tabs: **Category** and **Payment Mode**. Both cover the same spend for the same window, so both totals are identical. The card states that total above the bars.

### Category

Answers **what the money was spent on**.

Categories are grouped as they were recorded, without silently renaming anything. Six named groups collect the common spellings:

Salary · Maintenance · Rent · Electricity · Food and Mess · Deposit Refunds

**Deposit Refunds** here means a deposit returned to a tenant that was recorded on the expense side. It is spend on this screen. It is not the same thing as a refund raised against an invoice, which stays off this screen entirely.

Each row shows its **amount and its share of the window's total**.

**The card shows the three largest groups, then one Others row.** Others means **everything outside the top three**, nothing more. Tapping it opens a sheet with the full remainder, and the drill happens from a row inside that sheet. Inside the sheet, the remaining named groups come first, then spend that matched no group, listed under its **actual saved category name** and sorted by amount. Nothing in the sheet is ever labelled "Other", so the word keeps one meaning on this screen. Spend recorded with no category at all, a handful of rows across every property, appears there as **"No category"**, never merged into anything.

### Payment Mode

Answers **how the money left**.

Modes are: Cash · G Pay · Phone pe · Paytm · UPI · Bank · Card Machine · Cheque · Others · FlexiPe

**Two catch-alls, and they are not the same thing.** "Others" is a mode a person chose when recording the expense. **"Online"** is what the screen shows when a mode was recorded that it does not recognise at all. Keep them apart: a mode the system knows and the person picked goes to Others, an unknown one goes to Online. A manager never sees a raw code either way. Online hides at zero.

**This tab has no Others rollup.** It lists every mode that has spend and hides the rest. Payment modes are a fixed set of about ten, so there is nothing to roll up.

⚠ **"Others" means two different things on this one card, one tab apart.** On Category it is a rollup, the leftovers after the top three, and it always sits last. On Payment Mode it is a real mode that a person chose when recording the expense, and it is not a rollup at all. To keep them apart: on Payment Mode, Others **sorts by amount like every other mode** rather than always sitting last, and it **never carries the chevron** that means "open the remainder". A manager who taps it gets the expenses recorded under that mode, not a sheet.

**FlexiPe needs its own handling.** A FlexiPe transfer creates a real expense, but it records its mode as "Others", so reading the mode alone would bury every FlexiPe payment in the wrong row and leave the FlexiPe row permanently empty. Every FlexiPe expense carries a link back to the FlexiPe wallet it was paid from, and no other expense does; that link is how they are identified, never the mode. Show them as their own row, and take them out of Others so nothing is counted twice.

This is not a small row. **Roughly one expense in seven leaves through FlexiPe.**

---

## 8. Top Payers and Vendors

Two tabs: **Paid by** and **Paid to**. Both cover the same spend for the same window, so both totals are identical, and the card states that total above the bars.

| Tab | Meaning |
|---|---|
| **Paid by** | Who paid, grouped by the person recorded as having paid |
| **Paid to** | Who received the money, grouped by vendor, person or business |

Rules for both:

- Extra spaces are trimmed before grouping.
- Names are only merged when they are exactly the same after trimming. Do not guess that two similar names are one vendor.
- Where no name was recorded, the spend still counts in the total and appears as its own row, named **"No payer recorded"** or **"No payee recorded"**, so the two sides still add up. In practice this is a handful of rows across every property.
- Each row shows its **amount and its share of the window's total**. The bar carries the share; the number beside it carries the amount.
- **Never work out who paid from who created the record.** They are different things and only one of them is recorded.

Both tabs show the three largest, then Others, with the same two-step behaviour as Category.

---

## 9. Expense Trend

Total spend per month, one bar per month.

- Range options: **6, 12 and 24 months**, defaulting to 6. The range control belongs to this card.
- **The screen's time filter does not apply here.** A trend following a This Month filter would show a single bar.
- Months run oldest to newest, left to right.
- The current month is still running, and its bar is **marked as in progress** so it is not read as a collapse in spending.

Tapping a bar **moves the screen to that month** rather than filtering within the current one. This is the only place on the screen where a tap changes the window, and it should stay that way.

---

## 10. Expenses by Property

Which property the money went out of. One row per property, with a bar and an amount, **sorted highest to lowest**, and a **View more** control when there are more properties than fit.

**Hidden entirely when only one property is in scope.** With one property selected the card can only show a single row equal to the total already at the top of the screen, which tells the manager nothing they did not just read. It comes back on its own as soon as more than one property is selected.

Properties with no spend in the window still appear, at zero. A manager reading a portfolio needs to see which properties spent nothing.

---

## 11. What each number opens

### The rules

A drill filters a list. It never re-scopes the screen; the one exception is the trend chart, whose bars move the window. The destination opens on the same window and the same properties, the records always add back to the number tapped, and the back control names this screen. Averages are not tappable: no set of records adds back to one.

**Every row that opens something has to look like it does.** In the design file only the Others rows carry a chevron. Every drillable row needs the affordance.

### When the window changes what a tap shows

Time-scoped taps carry the window with them. The one Live tap, Still owed to staff, carries nothing: it opens Team Passbook as it stands today, because a running balance has no window to hand over. There are no forward windows on this screen.

### The tap matrix

<details>
<summary>Every tappable element, what opens, and whether it is ready today</summary>

| You tap | What opens | Arriving filtered to | Ready? |
|---|---|---|---|
| **Overview** | | | |
| Total Expense | Expense list | This window, all expenses | ✅ |
| Current FY Expense | Expense list | This financial year so far | ✅ |
| Number of expenses | Expense list | Same records as Total Expense; the list heads with count and total | ✅ |
| Still owed to staff | Team Passbook | As it stands today; the back control reads "Expense" | ✅ |
| **View all sheet** | | | |
| Average per expense | Nothing. Read-only line | — | — |
| Spend with no bill or receipt | Expense list | This window, only expenses with nothing attached | ❌ filter to add |
| Paid from collected money | Expense list | This window, that fund source | ❌ filter to add |
| Paid from staff's own money | Expense list | This window, that fund source | ❌ filter to add |
| Paid through FlexiPe | Expense list | This window, FlexiPe payments | ❌ filter to add |
| Paid from petty cash | Expense list | This window, that fund source | ❌ filter to add |
| Rows with a negative or zero amount | A quarantined list of just those rows, never the main list | — | ❌ filter to add |
| **Expense Breakdown, Category tab** | | | |
| A named group row | Expense list | This window, that category group | ⚠ drawer group to add |
| Others | Remainder sheet, then the list from a row inside it | That row's category | ⚠ drawer group to add |
| **Expense Breakdown, Payment Mode tab** | | | |
| A mode row, including Others and Online | Expense list | This window, that mode | ❌ filter to add |
| FlexiPe | Expense list | This window, FlexiPe payments | ❌ filter to add |
| **Top Payers and Vendors** | | | |
| A Paid by name | Expense list | This window, that payer | ⚠ drawer group to add |
| A Paid to name | Expense list | This window, that payee | ✅ |
| Others, either tab | Remainder sheet, then the list from a row inside it | That row's person | As its tab above |
| **Expense Trend** | | | |
| A bar | This screen, moved to that month | — | ✅ |
| **Expenses by Property** | | | |
| A property row | Expense list | That property, this window | ✅ |

**What the list can show on arrival today:** a property scope, a date window, a payee, and search. **Two drawer groups to add**, for filters the list already understands but the drawer does not offer: category, and paid by. **Four new abilities to build**, for filters the list does not have at all: nothing attached, a single payment mode, a fund source, and the quarantined negative-or-zero rows. Every ❌ on this screen is a filter to add over data that already exists; nothing here waits on a fact the system does not record.

</details>

### What the destination says when you arrive

Most of these drills land on a list that will not name the slice the manager tapped, and an empty slice reads as "you have no expenses at all" when the property may have spent lakhs that month. The list can already name a slice for ten built-in shortcuts, but a named slice discards every other filter sent with it, and the names are fixed to this month and year to date. So today only two drills here can name themselves, and one of those only while the filter sits on its default. Widen it so any window combined with a category, mode, payer, payee or property produces a named slice that keeps its filters. Three things every arrival must do:

1. **Name the slice**, in place of a fixed title.
2. **Name the filter in the empty state**, with a way out. "No Rent expenses in April" beats "There are no expenses."
3. **Show the active filter on arrival**, without scrolling.

---

## 12. Who can see this

**Each analytics tab follows the permission of the records it describes.** Whoever can open the expense list can read this screen. There is no separate analytics permission to grant, and nothing to re-assign for people who already have access.

Someone without it sees a full-screen lock: **"Analytics Restricted"**, with *"You don't have permission to view these analytics. Request access from your admin"* and a **Request Access** button.

The permissions to add, edit and delete expenses are separate. This screen is read-only, so being unable to add or edit changes nothing about what a person sees here.

**Narrowed access still adds up.** Where someone's access covers only some properties, every number here is worked out over exactly the expenses their list would show them. An owner and a staff member looking at the same screen can legitimately see different totals. What can never happen is a total that counts expenses the person cannot open: that reads as data going missing, and it is the fastest way to lose trust in a money screen.

---

## 13. What each card shows when it is empty, healthy or broken

### The zeros, told apart

| Situation | What shows |
|---|---|
| Never set up: no expense ever recorded on the property | *"No expense recorded yet. Add your first expense to start tracking costs."* with **Add expense**. For spend there is no separate onboarding stage: recording is the setup |
| In-window zero: expenses exist, nothing in the window | The card draws zero, plain: *"No expenses recorded in this period."* Never congratulated: on a running property a silent month usually means nothing was recorded, not that nothing was spent |
| Recording stopped: real history, then months of nothing | The same plain zero, never the setup message. The never-set-up CTA appears only where no expense has ever existed |
| Nothing owed to staff | *"Nothing owed to staff right now."* Good news, no CTA. This is the screen's one healthy state |
| Conditional rows with nothing | Petty cash and the negative-or-zero rows drop out of the View all sheet. Hidden is hidden: no ghost row |
| Not recorded: fund source coverage | The fund rows draw what the passbook holds; the small remainder is named as its own line, never guessed from who paid |
| No property selected | *"Select a property to view expenses."* |
| Failed | *"Couldn't load this"* with Retry, never a healthy message, never a zero |

### The copy already drawn

The existing empty copy on this screen is correct and on-topic. Keep it.

| Card | Copy |
|---|---|
| Expense Breakdown | *No expense recorded yet. Add your first expense to start tracking costs.* |
| Top Payers and Vendors | *No expense data yet.* |
| Expense Trend | *No trend data yet. Expenses over time will show here once added.* |
| Expenses by Property | *No property level data yet. Expenses will be grouped by property once recorded.* |

Each empty state needs a **title line** above the body copy; today they are body copy only. And where the copy invites an action, as the Breakdown one does with "Add your first expense", it needs a button to do it with, or the sentence has to change. Tone: short factual title, one reassuring line, a next step only where there genuinely is one.

---

## 14. What this screen is not

Each of these was considered and ruled out. They are listed so nobody rebuilds the argument, and so nobody adds them by accident.

- **Not a place to pay anyone.** The screen points at Team Passbook, where settling up happens. It must never become a form that moves money itself.
- **No pending-expense number.** There is a homepage reminder worded "Pending Expenses", but it is a nudge to log expenses, not a state anything is stored in. Nothing on this screen may be built on it.
- **No profit figure.** Cash flow already nets spending against money in. This screen explains spending only.
- **No fixing of bad categories.** Categories are shown exactly as they were recorded. The screen never quietly renames or reclassifies what someone typed.
- **No refund analysis.** Refunds against invoices are their own subject. Deposits returned to tenants appear here only because they were recorded as expenses.
- **No deleted-and-reversed audit view.** Deleted expenses are simply not spend.
- **No description-quality figure.** Descriptions are free text with no agreed standard, so a count of weak ones would mean nothing.
- **No forward window.** Spending is recorded after it happens. The future belongs to no card here.

---

## 15. Build guidance

1. **The total is rounded once, never per row.** *Test it:* a property with thousands of expenses shows a total equal to the sum of its list, not a few rupees off.
2. **A category saved with stray spaces counts with its category.** *Test it:* expenses saved as "Maintenance" and "Maintenance " chart as one bar.
3. **Every category name filters to exactly its own expenses.** *Test it:* every saved category name in the system, including any matching internal shorthands, opens only its own rows.
4. **Every number uses the paid date.** *Test it:* an expense typed in today but paid on 28 July moves last month's totals and this month's not at all.
5. **The Paid to breakdown covers only the selected window.** *Test it:* a vendor last paid in March appears in March's window and in no other.
6. **Negative and zero amounts stay out of every total.** *Test it:* adding a ₹0 row and a −₹500 row changes no card; both appear in the quarantine row's count.
7. **A missing paid date breaks nothing, and gets no screen.** No such expense exists on any property, and the date is not enforced on save. *Test it:* if such a row ever appears, every card still renders and the row counts nowhere.
8. **An expense dated in the future joins totals the day its date arrives.** *Test it:* date one for next Monday; every total moves on Monday and not before.
9. **Every FlexiPe expense shows FlexiPe as its source.** *Test it:* a FlexiPe transfer of ₹5,000 raises the FlexiPe rows by ₹5,000 and never appears as source-unknown.
10. **The screen never waits on the fund figures.** *Test it:* on an account with years of history, the tiles and breakdowns render while the fund rows still load; where the fund rows cannot load, that card says fund information is not available yet, and shows no zeros.
11. **Every total respects the manager's selected dates.** *Test it:* a custom ten-day window's total equals the list's sum for those ten days, never this month's figure.
12. **One computed total feeds all three cards that show it.** *Test it:* Overview, Expense Breakdown and Top Payers state the same total to the rupee, on every window.
13. **The tapped number and its list agree.** *Test it:* any number's list adds back to it exactly, before launch, on every drill in section 11.
14. **An expense paid late on the window's last day counts inside it.** *Test it:* record one at 23:30 on the final day; the window's total includes it.
15. **This screen shows one staff figure and hands off.** The per-person breakdown belongs to Team Passbook. *Test it:* the tile equals the sum of every team member's open expense payback in Passbook.
16. **Only paybacks against expenses reduce the staff figure.** *Test it:* recording a payback against a refund changes nothing here; a ₹2,000 part-payback against ₹5,000 leaves ₹3,000.

---

## 16. Open items

1. **Who adds the list's missing arrivals.** Two drawer groups (category, paid by) and four new filters (nothing attached, payment mode, fund source, quarantined rows), plus widening the named-slice mechanism. Sits outside the analytics build; every screen that drills needs the naming. Owner: engineering, unassigned.
2. **Do the deeper pages check permissions the way the list does.** This screen puts a sheet below the first list, under the Others rows. Whether those ship before the access work is settled is a suite-wide owner call with no owner.
3. **Team Passbook as its own analytics screen.** Parked deliberately. There is no design for it, and the live Passbook screen may already be enough. Owner: product, when its turn comes.
4. **The three older Expense documents carry superseded banners but still hold stale sections.** Fold the corrections back in or retire them; two documents disagreeing is how the wrong thing gets built. Owner: PM, low urgency.

---

## 17. Design file: what needs fixing

None of these change what the numbers mean. **Read this first:** every value drawn in the file is placeholder. Every tile reads ₹26.3K with a 12% chip, every bar card uses the same four amounts, and the section is titled "done". Take shape and copy from the file. Take no number from it.

<details>
<summary>All 41 fixes: Wrong · Missing · Remove · Decide</summary>

### Wrong

1. **The change chip colours.** An increase draws green. On the money tiles an increase is red; the Number of expenses tile is neutral grey, a chip state the shared component does not have yet.
2. **The Overview draws five tiles; the screen needs four different ones.** Drawn on the screen: May Expenses, May Rent Expenses, May Maintenance, May Refund Expenses, Current FY Expense. The standalone tile row draws May Salary Expenses where the screen draws May Refund Expenses, so the two copies in the file disagree on the fourth tile; both get rebuilt. Only the first and last survive, the first renamed. The three category tiles duplicate the Expense Breakdown card, and refunds are not expenses. Still owed to staff and Number of expenses exist nowhere in the file yet. The drawn tiles also carry inconsistent info icons, one missing where the rest have one; the rebuild resolves it.
3. **A designer's note above the tile row still specifies the old model** ("current month expense, current fy expense, top 3 expense category"). Superseded; update or delete it.
4. **Month names in tile titles.** "May Expenses" becomes "Total Expense". The tiles move with the filter, so a hardcoded month contradicts the card beneath.
5. **The "(Live)" label on the Overview heading.** Three of the row's four tiles move with the filter, so the heading is wrong for the row; the suite reserves Live for filter-immune numbers, and the one tile that is Live says "as of today" on its own face.
6. **The time filter is drawn as a stepper labelled "Today"** while four cards read "This Month". "Today" is not an option, and a stepper is the wrong control for a five-option list.
7. **The property list is not sorted.** The file's own note requires highest to lowest.
8. **Bar lengths do not match their values on any bar card.** On six of the seven, a ₹36k bar draws shorter than a ₹22k one; on Expenses by Property, a ₹5.2L bar draws the same length as a ₹2.8L one.
9. **The rows do not add up to the stated total.** ₹32k + ₹22k + ₹36k + ₹32k under "₹54,000", on two separate cards.
10. **The trend months are out of order** (Jan, Feb, Apr, Mar), four bars are drawn for a six-month range, and the bars overshoot their own axis: ₹32k and ₹36k against a scale topping at ₹30k.
11. **The same total is labelled two ways**: "Total Expense" on one card, "Total Amount" on another. One number, one name.
12. **The card is named two different things**: "Expense Group" in the component and empty state, "Top Payers & Vendors" on the screen.
13. **Category labels fall short of the spec**: "Deposit" where the group is "Deposit Refunds", and no Salary, Maintenance or Food and Mess row drawn at all.
14. **The Paid to placeholder names are staff**, not vendors.
15. **The axis is labelled two ways on one card**: "Amount in INR (₹)" on one tab, "Amount" on the other, and the on-screen copy of that card has no axis caption at all.
16. **The chart axis ticks are unevenly spaced**: 10k, 30k, 50k, 80k, 100k.
17. **Layer names carry over from other screens**: the Expense Trend card is named "dues breakup", three different layers are all named "expense group", and every empty-state text layer is named with leftover complaints copy even though the rendered text is correct.
18. **The section is marked "done"** while this list is outstanding.

### Missing

19. **The View all sheet.** The link exists in the header; the sheet was never drawn.
20. **A tap affordance on every drillable row.** Only the Others rows carry a chevron today; every row on every card opens something.
21. **The Restricted full-screen lock** for Expense.
22. **The Custom range picker, stopping at today.** No picker is drawn at all today, and when it is, it must offer no future dates.
23. **Loading skeletons** and the **failed-card** state.
24. **The unfinished-period marking** on the change chip and the trend's in-progress bar.
25. **The tile with its chip hidden.** On All Time the chip disappears; the tile needs a drawn state with the number alone and no gap where the chip was.
26. **The neutral grey chip** for Number of expenses.
27. **The trend's range control with its other two options.** Only the default is drawn.
28. **The two fixed-window face labels**: "This financial year" on the Current FY tile, "as of today" on Still owed to staff.
29. **An empty state for the Overview row.** Every other card has one.
30. **A title line on each empty state**, and a button wherever the copy invites an action. The Breakdown copy says "Add your first expense" with nothing to tap.
31. **The healthy state**: *"Nothing owed to staff right now."*
32. **Info-icon content.** Eleven info icons across the cards and empty states, and no screen has written what any of them say. An icon that opens nothing is worse than no icon. The three that most need one: Still owed to staff, the fund rows, and the bill-or-receipt figure, because all three are numbers a manager has never seen before.

### Remove

33. **"Where is my property losing money?"** Exists only as an empty state, no filled version anywhere, and its copy is about vacant beds and lost revenue.
34. **The Team Passbook card.** A read-only copy of a live screen that can actually settle up; replaced by the Still owed to staff tile. Its layer is a re-skin of the Top Payers card carrying that card's placeholder data, under two tabs of its own reading Receivable and Payable.
35. **The "Others" rollup row on the Payment Mode tab.** Others there is a real recorded mode, not a rollup; a rollup row gives the word two meanings on one tab.
36. **Hidden Paid by / Paid to buttons inside Expenses by Property**, in both the filled and the empty version.
37. **A second, dead copy of the tile row inside the screen**, clipped so it never renders, holding the five tiles in a different order.
38. **The second, switched-off change chip inside every tile.** The component has two chip slots and uses one.
39. **Two stray leftover groups.** An icon group parked outside the screen frame, and a status-bar group sitting in the blank bottom quarter inside it. Neither renders anything the screen uses.
40. **A switched-off gridline row inside the Trend chart**, which is why the trend shows four gridlines where every other card shows five.

### Decide

41. **Bar colours change between tabs with nothing behind the change, and Expenses by Property is the only red card on the screen.** Red carries a warning meaning elsewhere in the app: commit to one palette for the whole cost screen or take the red off that one card. Related: Expenses by Property uses a different row pattern from every other bar card (name above the bar, amount outside it, no axis, no total line); decide whether that is deliberate.

</details>

---

## Where the measured figures came from

<details>
<summary>Every production figure a decision rests on</summary>

Across all properties, expenses paid in the twelve months to 8 August 2026 (150,901 of them). The table is written to continuously, so re-runs move slightly; that is the data changing, not an error.

| Measured | Result | What it decided |
|---|---|---|
| No bill or receipt attached | 83% | Kept the figure, as money rather than a count |
| Paid from staff's own money | 26% | Justified the Still owed to staff tile |
| Paid through FlexiPe | 13% | Gave FlexiPe its own row and its own fund bucket |
| Paid from petty cash | 0.35% | Made petty cash a row that only appears when used |
| Negative or zero amount | 0.16% | Kept the quarantine, shrank it to one conditional row |
| No category, payee or payer | 21 rows in total | Dropped the wider review-gap section |
| No paid date | zero rows, across all 340,524 live expenses | Dropped the cleanup path entirely |
| FlexiPe expenses with no fund record | 100% | Treated FlexiPe as its own bucket, not missing data. The genuine failures were 213 rows at the 7 August measurement |
| Expenses dated in the future | 45 of 340,524, one beyond the next seven days | No forward setting; Custom stops at today |

</details>
