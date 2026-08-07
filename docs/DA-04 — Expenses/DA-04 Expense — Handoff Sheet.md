---
title: DA-04 Expense — Handoff Sheet
date: 2026-08-07
tags: [rentok, expenses, financials, detailed-analytics, handoff]
status: v2 · developer handoff · full end-to-end pass applied
owner: Sanchay
---

# Expense — Handoff Sheet

Everything on the Expense analytics screen: what each number means, what window it covers, what happens when it is tapped, and what it does when there is nothing to show.

---

## What is in here

| | Section | For |
|---|---|---|
| **1** | Build status | everyone |
| **2** | Where this lives | everyone |
| **3** | What every number counts | backend |
| **4** | How the screen behaves | backend + design |
| **5** | Overview | backend + design |
| **6** | View all sheet | backend + design |
| **7** | Expense Breakdown | backend + design |
| **8** | Top Payers and Vendors | backend + design |
| **9** | Expense Trend | backend + design |
| **10** | Expenses by Property | backend + design |
| **11** | What each number opens | backend |
| **12** | Who can see this | backend |
| **13** | What each card shows when it is empty, healthy or broken | design + copy |
| **14** | What this screen is not | everyone |
| **15** | Build guidance | backend |
| **16** | Open items | everyone |
| **17** | Design file: what needs fixing | design |

**Building the backend?** Read everything except Empty states and the Design file section. Start with Build guidance before writing anything; it holds the traps.

**Working on the design?** Sections 4 to 10, Empty states, and the Design file section, which collects every fix in one list.

**Just need the decisions?** Sections 1, 14 and 16, plus the measured figures at the end.

---

## 1. Build status

The screen is **not built**. The backend has one empty placeholder block for Expense that returns no numbers. Nothing here is broken, it is unwritten. Every number below has to be built from scratch.

The expense list that this screen drills into **is** built, and it already supports filtering by property, date range, category, payer and payee. That is the list every number on this screen opens.

---

## 2. Where this lives

Manager app, property header, **Financials** tab, third sub-tab: **Dues · Collection · Expense**.

---

## 3. What every number counts

### What counts as spend

Every number on this screen counts money that left the business and was recorded as an expense.

- A deleted expense is not spend.
- A refund paid back to a tenant is **not** an expense, even though it is money going out. It belongs to refunds.
- Paying staff back for money they fronted is **not** new spend. It reduces what the business owes them.
- A deposit returned to a tenant **is** counted here when it was recorded as an expense rather than against an invoice.
- Rows with a negative or zero amount are left out of every total, so a stray minus figure cannot quietly shrink a real number.

### Which date decides the period

The date the money was **paid**. Not the date someone typed the expense in.

### One word to be careful with

⚠ **"Bill" means something different here than on Dues and Collection.** There, a bill is an invoice raised to a tenant, money coming in. Here it means the receipt or proof attached to an expense, money going out. This screen never uses "bill" to mean an expense record, and never uses it to count anything. Where the proof is meant, this document and the screen both say **"bill or receipt"** in full.

### The same total appears three times

Total Expense sits on the Overview, above the Expense Breakdown bars, and above the Top Payers and Vendors bars. That is deliberate, and each is an anchor for the card under it. **All three must always be the same number**, computed once and reused. Three cards each working it out their own way is how they drift apart, and a manager who spots two different totals on one screen stops trusting all of them.

---

## 4. How the screen behaves

### The time filter

One filter at the top of the screen: **All Time · This Month (default) · Last Month · Current FY · Custom**.

Changing it updates every section. Some cards carry their own small date dropdown, and three rules govern those:

1. A card's own dropdown offers the **same options** as the filter at the top.
2. Changing the filter at the top **pulls every card back into line**. A manager can set one card aside deliberately, but can never end up looking at four different periods without meaning to.
3. **The trend chart is exempt.** It answers "how far back", not "which period", and needs several periods to say anything at all. It keeps its own range control.

Two numbers keep a fixed window on purpose and say so on the card: **Current FY Expense** and **Still owed to staff**.

### What every number does on every filter setting

| Number | All Time | This Month | Last Month | Current FY | Custom |
|---|---|---|---|---|---|
| Total Expense | Everything ever | Follows | Follows | Follows | Follows |
| Current FY Expense | Unchanged | Unchanged | Unchanged | Unchanged, and equal to Total Expense | Unchanged |
| Still owed to staff | Unchanged on every setting | | | | |
| Number of expenses | Everything ever | Follows | Follows | Follows | Follows |
| View all sheet | Follows, except the two fixed rows | | | | |
| Category | Follows | Follows | Follows | Follows | Follows |
| Payment Mode | Follows | Follows | Follows | Follows | Follows |
| Paid by · Paid to | Follows | Follows | Follows | Follows | Follows |
| Expense Trend | Exempt on every setting. Keeps its own range | | | | |
| Expenses by Property | Follows | Follows | Follows | Follows | Follows |

Three things fall out of that grid that are not obvious from the rules above.

**All Time has no previous period, so every change chip disappears there.** There is nothing before "everything", and a chip comparing all history against a period that does not exist is either blank or wrong. Hide the chip on Total Expense and on Number of expenses whenever the window is All Time, whether that came from the top filter or from a card's own dropdown. This is the only setting where a tile shows a number with nothing beside it, and that is correct.

**On Current FY, the Current FY tile and Total Expense show the same number, side by side. That is accepted.** Hiding it would move every tile after it under the manager's thumb at the moment they change filters, and a row that reshuffles is worse than a number that repeats. The tile earns its place at the other four settings by letting someone see the year next to the month in one glance. The Collection screen makes the same call in the same situation, and the two tabs need to behave alike.

**On Last Month and on any past Custom range, the Current FY tile still includes months after the period being looked at.** That is intended, and it is why the tile carries "This financial year" on its face. Without that label a manager investigating April would read it as April's figure.

One more, on the trend's own control: its range options are **6 months · 12 months · 24 months**, defaulting to 6. The design draws only the default. A trend needs enough bars to show a shape, so nothing shorter than six is offered, and All Time is not offered at all because a bar per month over a property's whole life is unreadable.

### Comparing against the previous period

Most of the time a manager is looking at a period still running. On the 3rd of the month, that is three days.

**Never compare an unfinished period against a finished one.** Compare against the **same point** in the previous period and say so on the chip: *"▲ 12% vs same point last month."* The note drops away once the period completes.

**An unfinished period is always marked as unfinished** wherever it appears: on the change chip, on the in-progress bar of the trend chart, anywhere else.

**Up is bad on this screen.** Spending more than last period is a red chip, spending less is green. This is the opposite of Collection and has to be set deliberately.

Where the previous period had no spend at all, show the rupee change and drop the percentage. A percentage measured from zero tells the manager nothing.

### Future periods

A manager may select a period that has not happened yet. Say why in one line and **offer a way out**, rather than showing a blank:

> *"Nothing spent in this period yet. Expenses count from the day they are paid. Switch to This Month to see what has gone out so far."*

This fires only when the **whole** period is in the future. A period straddling today has real data and behaves normally.

### Loading and failure

**Every card loads, fails and retries on its own.** The screen never waits for its slowest card. Simple totals return at once, the trend chart and the joined breakdowns take longer, and All Time on an old property makes that worse.

**Skeletons match the shape of the card they become**, so nothing jumps position as the page fills in.

**A failed card never shows a number.** Zero is real, meaningful data here, so it can never double as "we do not know."

**Empty and failed look different.** Empty means the query worked and there is nothing there: it names what is missing and offers no retry. Failed says only *"Couldn't load this"* and offers a **Retry** that refetches that card alone. Error copy stays plain, with no apology, no cause and no codes.

**A failed card keeps its heading and shrinks** to its message and Retry rather than leaving a card-sized hole.

**When every card fails it is the connection, not the cards.** Replace them all with one message and one Retry: *"Couldn't load this page. Check your connection."*

### There is no view toggle on this screen

Collection switches between Paid Date and Due Date, and Inventory switches between Bed and Room. **Expense has nothing equivalent.** The Category / Payment Mode and Paid by / Paid to pairs are tabs inside a single card, not a mode that changes the whole screen. Each pair splits the same spend for the same period two different ways, so both sides of each pair always add to the same total. Nothing else on the screen changes when a tab is switched.

### Where a manager arrives

The entry point is the **Overview row at the top of the Expense tab**. Everything else is reached by scrolling. There is no single big number on this screen, so the states that would normally live on one, an empty period and a viewer seeing a narrowed slice, live on the Overview row instead.

### Sorting

Every list and every bar on this screen sorts **highest to lowest**. The placeholder data in the design file does not demonstrate this. Ignore the design file on this point.

---

## 5. Overview

Four tiles across the top, scrolling sideways, with a **View all** link in the header.

| Tile | What it means | Window |
|---|---|---|
| **Total Expense** | Everything spent in the selected period. | Follows the filter |
| **Current FY Expense** | Everything spent since the start of the current financial year, April to March. | Fixed. Says so on the card |
| **Still owed to staff** | Money staff paid out of their own pockets that the business has not paid back yet. | Fixed. A running total, not a period figure |
| **Number of expenses** | How many expenses were recorded in the selected period. | Follows the filter |

**Still owed to staff is not spend, and never adds into any total on this screen.** It answers a different question: what does the business owe its own people right now. It is a running figure on purpose. A version that followed the filter would show zero for April simply because April's advances had since been settled, which reads as "nothing owed" when lakhs may be outstanding.

This matters more than it looks: **a quarter of all expenses are paid from staff's own money.** It is the most common way money leaves after collected funds.

**Which tiles carry a change chip, and against what:**

| Tile | Change chip | Compared against |
|---|---|---|
| Total Expense | Yes | The same point in the previous period |
| Current FY Expense | Yes | The same point in the previous financial year |
| Still owed to staff | **No** | Nothing. It is a balance, not a flow. "Twelve percent more owed than last month" is not a sentence anyone acts on |
| Number of expenses | Yes | The same point in the previous period |

The two fixed-window tiles say so on their face, in the place the other tiles put their change chip: **Current FY Expense** reads *"This financial year"*, and **Still owed to staff** reads *"Right now"*. Without that, a fixed number sitting in a row of period numbers is simply wrong to anyone scanning quickly.

**Total Expense and Number of expenses open the same list.** That is correct and not a mistake. They are two readings of one set of records, and the list shows both the count and the total at its head.

**Still owed to staff is a live number on a screen that otherwise has none.** No filter changes it. That is the only such number here, which is why it is a single tile rather than its own section, and why it carries "Right now" on its face.

---

## 6. View all sheet

Opens from the **View all** link in the Overview header. A bottom sheet holding the numbers the four tiles have no room for.

| Row | What it means |
|---|---|
| Total Expense | Everything spent in the selected period |
| Current FY Expense | Everything spent since April, whatever period is selected |
| Still owed to staff | Money staff fronted that has not been paid back, right now |
| Number of expenses | How many expenses were recorded in the selected period |
| **Average per expense** | Total Expense divided by the number of expenses. Useful for spotting a month made of one very large expense versus a month made of many small ones. Hidden when there are no expenses. |
| **Spend with no bill or receipt** | How much of this period's spend has no bill or receipt attached to it, shown as an amount and a share: *"₹8.2L of ₹9.8L, 84%"*. Not a count. A count of expenses tells the owner nothing about exposure, an amount tells them what they cannot account for. |
| **Paid from collected money** | Spend that came out of money the business had already collected. |
| **Paid from staff's own money** | Spend staff fronted personally. This is what creates the "still owed to staff" figure. |
| **Paid through FlexiPe** | Spend that left through a FlexiPe transfer. |
| **Paid from petty cash** | Cash the business had already handed to staff. **Shown only when it has spend.** It is used on well under one percent of expenses, so a permanent row would suggest a distinction that barely exists. |
| **Rows with a negative or zero amount** | How many, and what they add up to. **Shown only when there are any.** These are left out of every total on the screen, and this is the one place they appear, so the owner can see the totals were worked out on clean rows. They open a list of just those rows, not the main expense list. |

Every row opens the expense list for that slice, **except three**: Still owed to staff opens Team Passbook; the negative-or-zero row opens its own quarantined list, never the main one; and Average per expense opens nothing at all, because an average has no set of records that adds back to it.

**None of the four fund rows can be worked out from the expense record itself.** Whose money paid for something is held in the team passbook, not on the expense. Three rules follow from that, and each one is a wrong number if missed:

- **An expense can be paid from more than one source.** Count it once in Total Expense and in Number of expenses; split only its amount across the fund rows. The four fund rows plus the small no-record remainder below always add back to Total Expense. If they do not, something is counted twice or dropped.
- **Deleting an expense leaves its passbook entry behind, reversed.** The reversal has to be subtracted from the fund rows. Skipping this overstates every fund row on any property that has ever deleted an expense.
- **Paying staff back creates its own passbook entry, and it is not spend.** It must never appear as a fifth source of money. If it does, the screen shows staff paybacks as though the business spent that money twice.

**A passbook record is missing on two kinds of expense, and they are treated differently.** FlexiPe expenses never have one, because FlexiPe writes its expense on a path that never creates it; they are identified another way and fill the FlexiPe row. The rest, a couple of hundred across every property, are ones where the passbook entry failed to write and the failure was swallowed. Those stay in Total Expense, sit outside all four fund rows, and their source is never guessed from who paid.

**On bill coverage.** Across every property, **more than eight in ten expenses have no bill or receipt attached**. That is not a reason to leave the number off the screen, it is the reason to put it on. No screen has ever shown an owner this, which is why it got that high. An owner closing the month, handing books to an accountant, or answering a tax question cannot substantiate most of what they spent, and today nothing tells them so.

It has to be shown as **money, not a count**. "Twelve thousand expenses without bills" is noise. "₹8.2L of ₹9.8L this month has no receipt" is a number someone acts on. Expect it to look alarming at launch on nearly every property, and expect it to fall as people respond to it. That fall is the point.

**Why there is no wider missing-data section.** The other candidates were measured and do not exist as problems: expenses with no paid date number **zero** across every property. Expenses missing a category, payee or payer number in the low tens out of a hundred and fifty thousand, because the form already collects them. Rows with a negative or zero amount are real but rare: they stay out of every total, and the one conditional row above is the only place they surface.

**Empty states for this sheet:** where the period has no spend, the sheet still opens with every amount at ₹0 rather than refusing to open; Average per expense hides, and the two conditional rows, petty cash and negative-or-zero, drop out. Where nothing is owed to staff, that row reads *"Nothing owed to staff right now."*

---

## 7. Expense Breakdown

Two tabs: **Category** and **Payment Mode**. Both cover the same spend for the same period, so both totals are identical. The card states that total above the bars.

### Category

Answers **what the money was spent on**.

Categories are grouped as they were recorded, without silently renaming anything. Six named groups collect the common spellings:

Salary · Maintenance · Rent · Electricity · Food and Mess · Deposit Refunds

**Deposit Refunds** here means a deposit returned to a tenant that was recorded on the expense side. It is spend on this screen. It is not the same thing as a refund raised against an invoice, which stays off this screen entirely.

Each row shows its **amount and its share of the period total**.

**The card shows the three largest groups, then one Others row.** Others means **everything outside the top three**, nothing more. Tapping it opens a sheet with the full remainder, and the drill happens from a row inside that sheet. Inside the sheet, the remaining named groups come first, then spend that matched no group, listed under its **actual saved category name** and sorted by amount. Nothing in the sheet is ever labelled "Other", so the word keeps one meaning on this screen. Spend recorded with no category at all, a handful of rows across every property, appears there as **"No category"**, never merged into anything.

### Payment Mode

Answers **how the money left**.

Modes are: Cash · G Pay · Phone pe · Paytm · UPI · Bank · Card Machine · Cheque · Others · FlexiPe

**Two catch-alls, and they are not the same thing.** "Others" is a mode a person chose when recording the expense. **"Online"** is what the screen shows when a mode was recorded that it does not recognise at all. Keep them apart: a mode the system knows and the person picked goes to Others, an unknown one goes to Online. A manager never sees a raw code either way. Where Online is empty, hide it.

**This tab has no Others rollup.** It lists every mode that has spend and hides the rest. Payment modes are a fixed set of about ten, so there is nothing to roll up.

⚠ **"Others" means two different things on this one card, one tab apart.** On Category it is a rollup, the leftovers after the top three, and it always sits last. On Payment Mode it is a real mode that a person chose when recording the expense, and it is not a rollup at all. To keep them apart: on Payment Mode, Others **sorts by amount like every other mode** rather than being pinned to the bottom, and it **never carries the chevron** that means "open the remainder". A manager who taps it gets the expenses recorded under that mode, not a sheet.

**FlexiPe needs its own handling.** A FlexiPe transfer creates a real expense, but it records its mode as "Others", so reading the mode alone would bury every FlexiPe payment in the wrong row and leave the FlexiPe row permanently empty. Every FlexiPe expense carries a link back to the FlexiPe wallet it was paid from, and no other expense does — that link is how they are identified, never the mode. Show them as their own row, and take them out of Others so nothing is counted twice.

This is not a small row. **Roughly one expense in seven leaves through FlexiPe.**

---

## 8. Top Payers and Vendors

Two tabs: **Paid by** and **Paid to**. Both cover the same spend for the same period, so both totals are identical, and the card states that total above the bars.

| Tab | What it means |
|---|---|
| **Paid by** | Who paid, grouped by the person recorded as having paid. |
| **Paid to** | Who received the money, grouped by vendor, person or business. |

Rules for both:

- Extra spaces are trimmed before grouping.
- Names are only merged when they are exactly the same after trimming. Do not guess that two similar names are one vendor.
- Where no name was recorded, the spend still counts in the total and appears as its own row, named **"No payer recorded"** or **"No payee recorded"**, so the two sides still add up. In practice this is a handful of rows across every property.
- Each row shows its **amount and its share of the period total**. The bar carries the share; the number beside it carries the amount.
- **Never work out who paid from who created the record.** They are different things and only one of them is recorded.

Both tabs show the three largest, then Others, with the same two-step behaviour as Category.

---

## 9. Expense Trend

Total spend per month, one bar per month.

- Range options: **6, 12 and 24 months**, defaulting to 6. The range control belongs to this card.
- **The screen's time filter does not apply here.** A trend following a This Month filter would show a single bar.
- Months run oldest to newest, left to right.
- The current month is still running, and its bar is **marked as in progress** so it is not read as a collapse in spending.

Tapping a bar **moves the screen to that month** rather than filtering within the current one. This is the only place on the screen where a tap changes the period, and it should stay that way.

---

## 10. Expenses by Property

Which property the money went out of. One row per property, with a bar and an amount, **sorted highest to lowest**, and a **View more** control when there are more properties than fit.

**Hidden entirely when only one property is in scope.** With one property selected the card can only show a single row equal to the total already at the top of the screen, which tells the manager nothing they did not just read. It comes back on its own as soon as more than one property is selected.

Properties with no spend in the period still appear, at zero. A manager reading a portfolio needs to see which properties spent nothing.

---

## 11. What each number opens

Every number opens the records behind it, and **those records always add back to the number that was tapped.**

| Tapped | Opens |
|---|---|
| Total Expense | Expense list, same property and period |
| Current FY Expense | Expense list for the financial year to date |
| Number of expenses | Expense list, same property and period |
| **Still owed to staff** | **Team Passbook** |
| Any other View all sheet row | Expense list for that slice |
| **Spend with no bill or receipt** | Expense list, same period, showing only expenses with nothing attached |
| Rows with a negative or zero amount | The quarantined list of just those rows, never the main list |
| Any category row | Expense list for that category |
| Any Others rollup row (Category, Paid by, Paid to) | Sheet with the full remainder, then the list from a row inside it |
| Any payment mode row | Expense list for that mode |
| FlexiPe row | Expense list for FlexiPe expenses |
| Any Paid by or Paid to name | Expense list for that person or vendor |
| Any trend bar | Moves the screen to that month |
| Any property row | Expense list for that property and period |

**A drill filters a list. It never re-scopes the screen.** The one exception is the trend chart, above.

**Every row that opens something has to look like it does.** In the design file only the Others rows carry a chevron, so a manager has no way to tell that a category, a payment mode, a person or a property row opens anything at all. Every row on every card here opens something, and every one needs the affordance.

**Four of these drills need list support that does not exist yet.** The expense list filters by property, date range, category, payer and payee, and nothing else. It cannot show expenses with nothing attached, a single payment mode, a fund source, or the quarantined negative-and-zero rows. Those have to be added, or the no-bill-or-receipt figure, the whole Payment Mode tab, all four fund rows and the quarantine line become numbers that cannot open the records behind them.

**Still owed to staff leaves this screen.** It opens Team Passbook, which is where settling up actually happens. Two things travel with it: Passbook opens on **the same period** the manager was looking at, and the back control **names where they came from** and reads "Expense", not a bare arrow.

⚠ **Engineering check:** confirm Team Passbook accepts a period when opened this way. If it does not, that is a small change on the Passbook side, not a reason to drop the rule.

### The list has to say what it is showing

Today, most of these drills land on a list that will not name the slice the manager tapped, and an empty slice reads as "you have no expenses at all" when the property may have spent lakhs that month.

The mechanism to fix this already exists. The expense list can already return a **named slice** and does so for ten built-in shortcuts such as "Current Month Rent Expenses" and "Year-to-Date Expenses". It has two limits:

- A named slice **discards every other filter** sent with it, so it is all-or-nothing.
- The named slices are **fixed to this month and year to date**. There is no "Last Month Rent".

So of every drill on this screen, only two can name themselves today, and one of those only while the filter sits on its default.

**Widen the existing naming so that any period combined with a category, mode, payer, payee or property produces a named slice, and so that a named slice keeps its filters instead of discarding them.** This is widening something already built and already reaching the screen. It is the difference between landing on "Expenses" and landing on "Rent, April".

Three things every list arrived at from this screen must do:

1. **Name the slice**, in place of a fixed title.
2. **Name the filter in the empty state**, with a way out. "No Rent expenses in April" beats "There are no expenses."
3. **Show the active filter on arrival**, without scrolling.

---

## 12. Who can see this

Expense analytics is gated by the **expense-viewing permission**, which is separate from the one covering Dues and Collection. A team member can hold one without the other, and often does.

Someone without it sees a full-screen lock: **"Analytics Restricted"**, with *"You don't have permission to view these analytics. Request access from your admin"* and a **Request Access** button.

The permissions to add, edit and delete expenses are separate again. This screen is read-only, so being unable to add or edit changes nothing about what a person sees here.

**What must match: the totals on this screen and the rows behind them, for the person looking.** Where someone's access covers only some properties, every number here is worked out over exactly the expenses their list would show them. An owner and a staff member looking at the same screen can legitimately see different totals. What can never happen is a total that counts expenses the person cannot open. That reads as data going missing, and it is the fastest way to lose trust in a money screen.

⚠ **Still open across all analytics screens:** the lock is specced as an analytics permission, while the existing lists gate on their own viewing permissions. Nothing reconciles the two. This is not an Expense-specific question and needs settling once.

---

## 13. What each card shows when it is empty, healthy or broken

The existing empty copy on this screen is **correct and on-topic**, unlike some other screens. Keep it.

| Card | Copy |
|---|---|
| Expense Breakdown | *No expense recorded yet. Add your first expense to start tracking costs.* |
| Top Payers and Vendors | *No expense data yet.* |
| Expense Trend | *No trend data yet. Expenses over time will show here once added.* |
| Expenses by Property | *No property level data yet. Expenses will be grouped by property once recorded.* |

Three are missing and need writing:

| Card | Situation | Copy |
|---|---|---|
| Overview | No spend in the selected period | *No expenses recorded in this period.* |
| Still owed to staff | Nothing outstanding | *Nothing owed to staff right now.* |
| Any card | No property selected | *Select a property to view expenses.* |

Each empty state needs a **title line** above the body copy; today they are body copy only. And where the copy invites an action, as the Breakdown one does with "Add your first expense", it needs a button to do it with, or the sentence has to change.

Tone reference: short factual title, one reassuring line, and a next step only where there genuinely is one.

---

## 14. What this screen is not

Each of these was considered and ruled out. They are listed so nobody rebuilds the argument, and so nobody adds them by accident.

- **Not a place to pay anyone.** The screen points at Team Passbook, where settling up happens. It must never become a form that moves money itself. A diagnosis screen that can spend money is a different and much riskier thing.
- **No pending-expense number.** There is a homepage reminder worded "Pending Expenses", but it is a nudge to log expenses, not a real state anything is stored in. Nothing on this screen may be built on it.
- **No profit figure.** Cash flow already nets spending against money in. This screen explains spending only.
- **No fixing of bad categories.** Categories are shown exactly as they were recorded. The screen never quietly renames or reclassifies what someone typed.
- **No refund analysis.** Refunds against invoices are their own subject. Deposits returned to tenants appear here only because they were recorded as expenses.
- **No deleted-and-reversed audit view.** It was a nice-to-have and is cut. Deleted expenses are simply not spend.
- **No description-quality figure.** Descriptions are free text with no agreed standard, so a count of weak ones would mean nothing.

---

## 15. Build guidance

1. **Round the total, never each row.** Rounding every row and then adding them up drifts away from the real figure on a property with thousands of expenses.
2. **Match categories on the recorded text, allowing for stray spaces.** An exact match undercounts, because a category saved with a trailing space silently drops out.
3. **Do not reuse the existing all-time vendor list to build the Paid to breakdown.** It carries no date range and would show a year of vendors against a one-month total.
4. **Do not copy the monthly summary that already exists elsewhere.** It groups spend by the date the expense was typed in, not the date it was paid, and this screen uses the paid date everywhere.
5. **The category filter has a reserved word left in it from an older feature.** One particular word, passed as a category, does not filter to a category of that name at all: it means "everything except Salary and Maintenance". Engineering knows which word. It works, but it is a trap sitting in live filter logic, and there is already a proper mechanism for the same idea. Do not build on it, and make sure a real category name can never reach it.
6. **Leave negative and zero amounts out of every total.** They are rare but they are real, and one of them landing in a headline is a support ticket.
7. **Do not build a cleanup path for expenses with no paid date.** There is not one such expense on any property. The date is not enforced when an expense is saved, so it could happen in future, which means a number should not break if it ever does. But it has never happened, and a cleanup screen for it would be a screen nobody ever opens.
8. **The expense list needs four new filters before every number here can open its rows:** nothing attached, a single payment mode, a fund source, and the quarantined negative-or-zero rows. Today it filters by property, date range, category, payer and payee, and that is all.
9. **FlexiPe expenses have no fund record at all.** They are created on a path that never writes one. Any fund-source figure must treat them as their own bucket rather than as missing data, or one expense in seven becomes an unexplained gap.
10. **Watch the fund-source query on multi-property accounts.** It is the one number on this screen that needs a heavy join, and it is the first thing to simplify if the screen is slow.
11. **Never use the list's built-in current-month shortcuts to work out a total.** They carry their own fixed window and ignore the dates the manager selected, so a number built on one is silently wrong for every period except the default. They are for labelling a destination, not for computing.
12. **Work the period total out once and hand the same figure to all three cards that show it.** Three cards each computing it separately is how they drift apart.
13. **The analytics total and the expense list must match for the same filters, before launch.** If tapping a number opens a list that adds to something else, nothing else on the screen matters.
14. **Where a range has an end date, an expense paid late on that last day still counts in it.** The obvious way to write this check quietly excludes everything paid after midnight-start on the final day.
15. **The per-person breakdown of what is owed to staff belongs to Team Passbook, not here.** This screen shows one figure and hands off.
16. **When working out what is still owed to staff, count only paybacks made against expenses.** A payback made against a refund must never reduce this figure, and a partly repaid expense shows only what is left.
17. **If the fund figures are too slow on a multi-property account, ship the rest without them** and say plainly on the card that fund information is not available yet. Slow is worse than absent; wrong is worse than both.

---

## 16. Open items

1. **Which permission gates the analytics screens.** The lock is written as an analytics permission, while the existing lists each gate on their own viewing permission. Nothing reconciles the two. Shared across every screen in this project. Nobody owns it.
2. **Does Team Passbook accept a period when opened from here.** Engineering check. Small either way.
3. **Who widens the drill-list naming, and who adds the four missing list filters** (nothing attached, payment mode, fund source, negative-or-zero rows). Both sit outside the analytics work, and every screen that drills needs the first one.
4. **Do the deeper pages check permissions the way the list does.** This screen puts a sheet below the first list, under the Others rows. Whether those deeper pages ship before the access work is settled is an owner call with no owner.
5. **Team Passbook as its own analytics screen.** Parked deliberately. There is no design for it, and the live Passbook screen may already be enough.
6. **The three older Expense documents are now out of date in places and still call themselves the source of truth.** Each has been marked at the top with what this document overrides. Someone should either fold those corrections back into them or retire them, because two documents disagreeing is how the wrong thing gets built.

---

## 17. Design file: what needs fixing

Collected in one place so design has a single list. None of these change what the numbers mean.

**Read this first:** every value drawn in the file is placeholder. Every tile reads ₹26.3K with a 12% chip, every bar card uses the same four amounts, and the section is titled "done". Take shape and copy from the file. Take no number from it.

### The Overview row

1. **It draws five tiles; the screen needs four different ones.** Drawn: May Expenses, May Rent Expenses, May Maintenance, May Refund Expenses, Current FY Expense. Of those, only the first and last survive, the first renamed. **Remove** May Rent Expenses, May Maintenance and May Refund Expenses. The three category tiles duplicate the Expense Breakdown card below them, and refunds are not expenses at all. **Add** two tiles that exist nowhere in the file yet: Still owed to staff, and Number of expenses.
2. **A designer's note above the row still specifies the old model** ("current month expense, current fy expense, top 3 expense category"). It is superseded by the four tiles above and should be updated or deleted, so the next person reading the file does not rebuild the version that was replaced.
3. **Remove the "(Live)" label** from the Overview heading.
4. **Remove month names from tile titles.** "May Expenses" becomes "Total Expense". The tiles follow the filter now, so a hardcoded month would contradict the card beneath it.
5. **A second, dead copy of the tile row sits inside the screen** at the same position, clipped so it never renders, holding the same five tiles in a different order. Delete it.
6. **Every tile carries a second, switched-off change chip.** The component has two chip slots and only one is used. Remove the spare so nobody wires it up by accident.
7. **The info icon is missing on the Maintenance tile** and present on the others. Once the tiles change this resolves itself, but the component should be consistent.
8. **Two tiles need a fixed-window label** where the others put their change chip: "This financial year" and "Right now".

### Cards that need removing

9. **"Where is my property losing money?"** It exists only as an empty state, has no filled version anywhere, its copy is about vacant beds and lost revenue rather than spending, and it is the only card in the file with no date control.
10. **The Team Passbook card.** It is a read-only copy of a screen that already exists and can actually settle up. It is replaced here by the "Still owed to staff" tile. Its layer is a re-skinned copy of the Top Payers card and carries that card's placeholder data.
11. **The "Others" rollup row on the Payment Mode tab.** That tab lists every mode instead, and Others there is a real recorded mode, not a rollup, so a rollup row would give the word two meanings on one tab.
12. **Hidden Paid by / Paid to buttons** left inside Expenses by Property, in both the filled and the empty version.
13. **A stray icon group parked outside the screen frame.** Leftover assets.
14. **A switched-off gridline row inside the Trend chart**, which is why the trend shows four gridlines where every other card shows five.

### Wrong, needs correcting

15. **The change chip is the wrong colour.** It shows an increase in green. On a spending screen an increase is bad and must be red.
16. **The property list is not sorted.** The file's own note requires highest to lowest.
17. **Bar lengths do not match their values on all seven bar cards.** A ₹36k bar is drawn shorter than a ₹22k one, everywhere.
18. **The rows do not add up to the stated total.** Four rows of ₹32k, ₹22k, ₹36k and ₹32k sit under a total of ₹54,000, on two separate cards.
19. **The trend months are out of order** (Jan, Feb, Apr, Mar), only four bars are drawn for a six-month range, and **the bars overshoot their own axis**: ₹32k and ₹36k bars against a scale topping out at ₹30k.
20. **The same total is labelled two ways**: "Total Expense" on one card, "Total Amount" on another. It is one number and needs one name everywhere.
21. **The card is named two different things**: "Expense Group" in the component and empty state, "Top Payers & Vendors" on the screen.
22. **Category labels are short of the spec**: the tab draws "Deposit" where the group is "Deposit Refunds", and draws no Salary, Maintenance or Food and Mess row at all.
23. **The Paid to placeholder names are staff**, not vendors.
24. **The axis is labelled two ways on the Top Payers card**: "Amount in INR (₹)" on one tab, "Amount" on the other. The on-screen version of that card has no axis caption at all.
25. **The chart axis ticks are unevenly spaced**: 10k, 30k, 50k, 80k, 100k.
26. **Bar colours change between tabs** with nothing behind the change, and **Expenses by Property is the only red card on the screen**. Red carries a warning meaning elsewhere in the app. Either commit to red for the whole cost screen or take it off this one card.
27. **Expenses by Property uses a different row pattern from every other bar card**: name above the bar, amount outside it, no axis, no ticks, no total line. Decide whether that is deliberate.
28. **The time filter is drawn as a stepper labelled "Today"**, while four cards below it read "This Month". "Today" is not one of the options, and a stepper is the wrong control for a five-option list. Every other date control on the screen is a dropdown.
29. **Layer names carry over from other screens**: the Expense Trend card is named "dues breakup", and two different cards are both named "expense group". Component variants are still named after their old screens.
30. **The section is marked "done"** while this list is outstanding.

### Missing, needs drawing

31. **The View all sheet.** The link exists in the header; the sheet was never drawn.
32. **A tap affordance on every drillable row.** Today only the Others rows carry a chevron, so a manager has no way to know that a category, a mode, a person or a property row opens anything. Every row on this screen opens something.
33. **The Restricted full-screen lock** for Expense.
34. **Loading skeletons** and the **failed-card** state.
35. **The unfinished-period marking** on the change chip and on the trend chart's current bar.
36. **The tile without a change chip.** On All Time the chip disappears, so the tile needs a drawn state with the number alone and no gap where the chip was.
37. **The trend's range control with its other two options.** Only the default is drawn.
38. **The future-period state**, with its one-line explanation and its way out.
39. **An empty state for the Overview row.** Every other card has one.
40. **A title line on each empty state.** They are body copy only. The Breakdown copy says "Add your first expense" but there is no button to do it with.

### Content that does not exist yet

41. **Every card carries an info icon, and nobody has written what any of them say.** There are eleven of them, on all six cards and on every empty state. An info icon that opens nothing is worse than no icon at all. Either write the explanation for each card, in the plain language this screen uses, or remove the icons. The cards that most need one are Still owed to staff, the fund rows, and the no-bill-or-receipt figure, because all three are numbers a manager has never seen before.

---

## Where the measured figures came from

Four decisions on this screen rest on live data rather than on judgement, so the figures are recorded here and can be re-checked.

Across all properties, expenses paid in the last twelve months (151,012 of them):

| Measured | Result | What it decided |
|---|---|---|
| No bill or receipt attached | 83% | Kept the figure, as money rather than a count |
| Paid from staff's own money | 26% | Justified the Still owed to staff tile |
| Paid through FlexiPe | 13% | Gave FlexiPe its own row and its own fund bucket |
| Paid from petty cash | 0.35% | Made petty cash a row that only appears when used |
| Negative or zero amount | 0.16% | Kept the quarantine, shrank it to one conditional row |
| No category, payee or payer | 21 rows in total | Dropped the wider review-gap section |
| No paid date | zero rows, across all 339,732 live expenses | Dropped the cleanup path entirely |
| FlexiPe expenses with no fund record | 100% | Corrected the earlier explanation that this was a silent failure. The genuine failures are 213 rows |
