---
screen: DA-02 — Collection
status: v12 — audited against Figma, source docs and code. Ready for build.
date: 2026-08-06
---
> [!WARNING] Superseded 2026-08-07 — permissions rule changed
> This sheet says Dues and Collection unlock together on a shared analytics permission, and Expense is separate. **That is no longer the rule.** Each analytics tab now follows the permission of the records it describes — whoever can open the collections list can read the Collection screen. No separate analytics permission exists. Settled suite-wide; see the tracker.

> [!WARNING] Corrected 2026-08-07 — one line changed
> The lock message read *"Analytics Restricted."* with a full stop. Every other screen in the suite uses *"Analytics Restricted — "* with a dash, and this is one shared component, so the copy has to be identical. Corrected in place. Nothing else on this page changed.


# DA-02 — Collection: Handoff Sheet

What every number on the Collection screen means. No code, no file names. This is what the number is and where it comes from, not how it gets built.

## What this screen is for

Managers can already see that money came in. What they cannot see is **what kind of money it was**. This month's rent, old dues finally recovered, someone paying early, advance, or a deposit adjustment that cleared a bill without any new money arriving.

The screen answers one question: *is this real collection, or did the number go up for some other reason?*

## Where it lives

Manager App → property → **Financials** tab → **Collection** sub-tab. Its siblings are Dues and Expense.

## What this screen is not

- **Not a place to record a payment.** The existing payment flows own data entry.
- **Not a bank ledger or a cash flow report.** This screen explains collection. It does not replace finance reporting.
- **Not a second payments list.** Every drill goes to the list that already exists.
- **No alerts or notifications.** Managers already know when to open this. The problem is clarity once they do.
- **No guessing where money will land.** Show which bank settled money went to. Never predict a destination for money that has not settled.

---

## What counts as collection

Every number on this screen is built from the same base. Three rules define it, and they apply to every card, not only the totals.

**Only completed, active payments count.** A payment that failed, is still pending, or has been deactivated is not collection. It appears in no number here.

**The amount is what the owner actually receives.** Payment processing charges come off first. A ₹10,000 online payment with a ₹200 processing charge counts as ₹9,800, because ₹9,800 is what reaches the owner.

**Refunds come off everywhere.** Not only from the headline total, but from every breakdown row, every trend bar, every property row and every row in the View all sheet. A refund that lowered the total but not the category it came from would leave the two disagreeing, and one of them would be wrong.

Everything else on this screen is a different way of slicing that same base.

---

## The two views

A toggle sits at the top of the screen. It switches the whole screen between two ways of reading the same data. Every card follows it. It is not a per-card setting.

| View | What it filters by | The question it answers |
|---|---|---|
| **Paid Date** (opens first) | Money actually received in the selected period, whatever bill it was for | What came in, and what kind of money was it? |
| **Due Date** | Bills whose due date falls in the selected period, whenever they were paid | How did this period's billing do? |

A bill due in May and paid in June counts in **June** under Paid Date, because that is when the money arrived. It counts in **May** under Due Date, because that is the billing it belongs to. Both are correct. They answer different questions.

The same works in reverse. A June bill that someone pays early in May shows up in **May** under Paid Date, labelled Paid Early. It shows up in **June** under Due Date, counted against June's billing. One payment, two honest homes.

### What changes between the views

| Card | In Due Date view |
|---|---|
| Collection Overview | Four of the seven tiles are replaced by three |
| Collection Breakup | Category and Status gain a billed comparison. Mode and Received by show collected and adjusted money only |
| Collection Status | Hidden |
| Collection Trend | The collection bar counts differently |
| Adjusted Collection, Collection by Property, Payment Settlement | No change |

### Three numbers that never follow the due date filter

**Advance, Current FY and Settlement Pending always count by when money arrived, in both views.**

None of them is about billing. Advance is money received against no bill at all. Current FY is a fixed year of money coming in. Settlement Pending is about where money has got to on its way to the bank. Asking "how much advance was due in May" is a question with no meaning. So is "online money against May's bills that has not settled."

These three sit in both views, unchanged, counted the same way. Everything else on the screen switches.

### The one rule that does change: adjustments count as collected

Everywhere on this screen, a bill cleared from a deposit is never counted as collection. No new money arrived. In Due Date view that rule would cause real harm.

A tenant is moving out. Their ₹10,000 May rent is cleared from their deposit. The bill is dealt with and nobody owes anything. If that ₹10,000 sat in Still Unpaid, a manager would go chasing a tenant who owes nothing.

So in Due Date view, money cleared from **deposit, advance, caution money or a discount** all count towards Collected & Adjusted. All four. A discounted bill is just as dealt with as a deposit adjusted one.

*(A note on the word "settled". On the Payment Settlement card it means something different: money that has physically reached the owner's bank. That card is the only place it carries that meaning. Everywhere else on this screen a bill being dealt with is described as collected or adjusted, never settled.)*

---

## How the time filter works

One filter at the top: **All Time, This Month (default), Last Month, Current FY, Custom.** Changing it updates every card.

**Three cards can be set to their own period.** Adjusted Collection, Collection by Property and Payment Settlement each carry a small date dropdown with the same five options as the filter above. Setting one changes that card alone.

**Changing the filter at the top pulls every card back into line.** Any card sitting on its own period snaps back to follow it. A manager can set one card aside to check something, but can never end up with a screen showing four different periods without meaning to.

**The Trend chart is not part of this.** It has its own range control and the filter at the top never touches it. See section 6. Every other card follows the filter. The toggle above governs all of them, Trend included.

**One number ignores the filter completely.** The Current FY tile always shows April to March, whatever period is selected. When the filter is itself set to Current FY, that tile and Total Collection show the same number. That is accepted. The tile earns its place the rest of the time by letting a manager see the year alongside the month in one glance.

### Periods that have not happened yet

Custom lets a manager pick a period in the future, and that is allowed. It is genuinely useful in Due Date view, where next month's bills already exist and some tenants have already paid early.

In **Paid Date view** a period entirely in the future is ₹0, and always will be. Money cannot be received before the time arrives. So show the zero with an explanation and a way out:

> **Nothing received yet.** Collection counts money as it arrives. Switch to Due Date to see what's billed for this period.

The switch should be tappable. One tap to the answer they were actually looking for.

**This applies only when the whole period is in the future.** A period that straddles today, say 25 May to 25 June, has real money in it and behaves normally. No message.

### Periods that have not finished

The default view is This Month, so most of the time a manager is looking at a period still running. On the 3rd, "This Month" means three days.

**The change indicator compares against the same point in the previous period, not the whole of it.** Three days against the previous month's first three days.

> Total Collection ₹2.4L · ▲ 12% *vs same point last month*

Comparing three days against a finished month would show a collapse that is not real. On the default view. For the first week of every month. On every property. The note saying what is being compared drops away once the period is complete.

**A period that has not finished is always marked as unfinished.** That is the same idea behind the in-progress bar on the Trend chart. One principle, two places.

---

## Drill-down rule

Tapping any row or number opens the existing Collection payments list, filtered to exactly what was tapped, newest first.

Actions on those rows, such as view receipt, view tenant, WhatsApp and call, appear or hide depending on what the person is allowed to do.

**Never send someone to a page with weaker access checks than the list itself.** Some pages reachable below the first payments list do not check the viewer's permission the way the list does. Until they do, drilling stops at the list. **This is an open decision with no owner:** ship the deeper drills on day one, or hold them until the access checks are fixed. It needs answering before build, not during.

**"Others" rows work in two steps.** Tapping Others opens a bottom sheet with the full remaining breakdown. Tapping an item inside that sheet is what drills to the payments list. This matches how the Dues screen already handles its own Others rows.

**Numbers about bills go to the Dues screen instead.** Anything describing bills rather than payments has no payment list to open. On this screen that means three places: the **Billed** and **Still Unpaid** tiles in Due Date view, and the **Due** segment of the Trend chart.

When a number sends someone to Dues, two things have to be true:

- **The Dues screen opens on the same period.** Landing on This Month after tapping a Last Month number would be its own small betrayal.
- **The back control names where they came from**, reading "Collection" rather than a bare arrow. A manager who has landed on an unfamiliar screen should be able to see how they got there.

### What the destination has to say

The payments list today does not tell a manager which number brought them there. Its title is fixed as "Collections Overview" whatever filter is applied. Filter chips exist but only appear once the person scrolls, so on arrival there are none. Filters that come from tapping a widget produce no chip at all. And when a filtered slice is empty, the whole message is *"There are no collections present."*

So a manager taps Past Dues Collected, lands on a page headed "Collections Overview", and nothing on it says these are payments against bills due before May. If the slice is empty, the page appears to say they have no collections at all, which for a property that collected ₹4L this month is simply untrue.

That undoes on the next screen what this one is built to do. Three things fix it, and none of them is large:

1. **The destination names the slice**, in place of the fixed title. *"Past Dues Collected · May"*.
2. **The empty state names the filter and offers a way on.** Not *"There are no collections present"* but *"No payments match this view"*, saying what the view is, with a way to clear it. Where there is an obvious next question, offer it: a property that collected nothing should offer *"See what was due there"*.
3. **The active filter is visible on arrival**, without scrolling. A chip that appears only after the person scrolls cannot do its job at the moment they land.

This is work on the payments list rather than on Collection, and it applies to every analytics screen that drills into a list.

---

## 1. Collection Overview

Seven tiles across the top. Each shows an amount and a change indicator against the previous period.

**Paid Date view:**

| Tile | What it means |
|---|---|
| Total Collection | All money received in the period, after subtracting refunds |
| This Month's Bills Collected | The part of it that paid bills due inside the same period |
| Past Dues Collected | The part that paid bills due before the period started. Old dues recovered. |
| Paid Early | The part that paid bills due after the period ends. Advance is not counted here, since it has its own tile. |
| Advance | The part received as advance, not against any bill |
| Current FY | Money received so far this financial year, April to March |
| Settlement Pending | Online money that has not reached the bank yet |

The middle four tiles add up to Total Collection exactly.

Deposit, advance and caution money adjustments are never counted here. They clear a bill without new money arriving, so they belong in Adjusted Collection.

**Due Date view.** Four of those tiles stop working. Total Collection and This Month's Bills Collected become the same number, and Past Dues Collected and Paid Early are always zero. Those four are replaced by three:

| Tile | What it means |
|---|---|
| Billed | Everything billed for the period, paid or not. Bills that were written off or fully refunded are not billing anyone owes, so they do not count here. |
| Collected & Adjusted | How much of that billing has been dealt with. Money received, plus bills cleared from deposit, advance, caution money or discount. |
| Still Unpaid | Billed minus Collected & Adjusted. What is genuinely still owed. |

*(If "Collected & Adjusted" reads too long on a small tile, "Collected" with the explanation in its ⓘ is an acceptable fallback. The longer label is preferred, because "Collected" alone would be untrue in this view.)*

Advance, Current FY and Settlement Pending stay exactly as they are. So Due Date view shows six tiles.

**Two situations the Total Collection tile has to handle:**

- **Refunds larger than money received.** Show the negative amount in red, with one line under it: *"Refunds were higher than collection this period."* Without that line a manager assumes the app is broken.
- **Nothing but adjustments.** Show ₹0, with one line: *"No money received. ₹X of bills were cleared from deposits and advances."* Without it, a real month reads as a dead one.

**Missing in Figma:** the labels "Past Bill" and "Future Bill" should read **Past Dues Collected** and **Paid Early**. The current wording reads like an old bill and an upcoming charge. The Advance tile, the Due Date version of the row, and both situations above are not drawn yet.

---

## 2. Collection Breakup

One card, four tabs. All four split the same total. Only the grouping changes.

| Tab | Splits by | Rows shown |
|---|---|---|
| Category | Bill type | Top 3 by amount, rest in **Others** |
| Status | Tenant stage | Active, Under Eviction, Bookings, Old Tenants. All four, always. |
| Mode | How it was paid | Top 3 by amount, rest in **Others** |
| Received by | Who took the payment | Top 3 by amount, rest in **Others** |

**Top three means purely by amount.** Whichever three are largest in the selected period show individually, under their real names. Rent, electricity, food, deposit, late fine, advance, whatever it happens to be. Everything else rolls into Others. Nothing is excluded from showing individually and nothing is guaranteed a place. For most properties the same three will come up month after month. When they do not, the card follows the money.

**Each tab carries the period total above its rows**, so the rows visibly belong to something. Whichever tab is open, that figure stays the same and the rows under it add up to it.

The **Others** row on the Received by tab should say how many people it is holding, such as *"Others (4 staff)"*. A bare "Others" hides whether that is one person or twelve.

**A badge marks the largest row on the Mode and Received by tabs.** It moves with the data. It does not appear on Category or Status, where a winning row would not mean anything. In the design it is drawn as a gold shield.

**Under Eviction** means an active tenant who has been served notice. It is not a separate stage in the data. It is a slice of Active, pulled out as its own row here. The Dues screen keeps these tenants inside Active. That difference is deliberate.

**Payments not linked to any tenant must never disappear.** They stay in the total and in the Mode and Received by tabs. On the Status tab they either get their own **Unlinked** row, or the card says plainly that they are not counted there. Dropping them silently would make the tabs stop adding up with nothing explaining why.

### What the tabs show in Due Date view

All four tabs stay. Every one of them splits **Collected & Adjusted**, the same figure as the tile above: money received plus bills cleared from deposit, advance, caution money or discount, for bills due in the period.

**Category and Status also show what was billed**, row by row, so each reads as collected against billed:

> Rent ₹8L of ₹10L · Electricity ₹1.2L of ₹3L

That is the question this view exists for. A bare collected figure per category does not answer it.

**Mode and Received by show Collected & Adjusted only.** They cannot show a billed side, because an unpaid bill has no payment method and nobody received it. That is not a gap. Those two describe how money arrived, and money that has not arrived has no answer.

All four still add back to the same Collected & Adjusted total. Category and Status simply carry extra context alongside it.

The Mode tab earns its place here for one specific reason. It is the only place on the screen that answers this: **how much of a month's billing was cleared with real money, and how much was drawn out of deposits?** Paid Date view cannot show that. It leaves adjustments out of its total by design. For a property with heavy move-outs, that share is a real signal.

**Missing in Figma:** the Mode tab has no Cash row, though its own empty message promises one. Cash is one of the most common ways rent gets paid. Add the row and rewrite the message to match the real list. The trophy badge is also pinned to the RentOk row and should follow whichever row is largest.

**No written spec exists for the Received by tab.** The designer's notes describe both views as Category, Payment Mode and Tenant Status only. Received by is a real tab in the built card that nobody has specified, so its details are decided here for the first time rather than inherited.

**For engineering:** two internal sources disagree on how a booking is identified in tenant data. Settle this before building the Status tab, or the Bookings row will be wrong.

---

## 3. Collection Status

*Paid Date view only. Hidden in Due Date view.*

This card answers a different question from the tiles. Not how much came in, but what kind of bills it paid.

| Row | What it means |
|---|---|
| This Month's Due | Bills due in the period that were collected |
| Past Due | Older bills recovered during the period |
| Future Due | Bills due later, paid early |
| Advance Payments | Money received as advance, not against any bill |

Four bars on one shared scale, so their relative sizes are readable at a glance. All four are money that came in.

**These four are the same numbers as four of the Overview tiles above.** They appear twice on the screen, once as tiles and once as bars. That is accepted. The tiles are the headline. The bars let you compare the four against each other, which a sideways scrolling tile row cannot do.

**No unpaid row here.** What is still owed is the Dues screen's job. This screen shows collection.

**Hidden in Due Date view.** The card's whole job is sorting money by when each bill was due. In Due Date view the filter has already decided that, because everything in scope is due inside the period. The first row would show everything and the rest would show zero, every time. The card would just repeat the filter back.

*(The design file has a note headed "Collection Efficiency" describing a five part stacked bar. Its parts line up with this card plus an unpaid segment, so it is being treated as an earlier name for this same card. Recorded in case that assumption ever needs revisiting.)*

---

## 4. Adjusted Collection

*The same in both views. Can be set to its own period.*

Bills cleared without any new money arriving. Kept separate so they are never mistaken for real collection.

| Row | What it means |
|---|---|
| Advance | A bill cleared using advance the tenant paid earlier |
| Security Deposit | A bill cleared using the tenant's deposit |
| Discount | Amount the owner waived off bills in the period |
| Caution Money | A bill cleared using caution money held for the tenant |

Each figure is the **amount of bill that got cleared**, not the size of the payment that cleared it. Shown as a four item grid, in the order above.

The first three happen when a payment is recorded using that specific method.

**Discount works differently.** It is not a payment method. It is an amount recorded against a payment made any way at all, and it does not reduce the bill. A ₹10,000 bill with a ₹2,000 discount stays ₹10,000, the tenant pays ₹8,000, and ₹2,000 is marked as discount. So that ₹2,000 is a bill cleared without cash, exactly like a deposit adjustment. It needs counting as its own total rather than as a fourth payment method.

Tapping Discount opens payments that had a discount applied. Tapping the other three opens payments made using that method.

Do not confuse the Advance row here with the Advance tile above. Here it means advance being **used up** against a bill. In the tile it means advance being **received**.

**Not built yet:** caution money adjustments have no working path in the system today. Deposit and advance adjustments do. This one has to be built from scratch, not copied from the other two.

---

## 5. Collection by Property

*Multi-property only. The same in both views. Can be set to its own period.*

The period's total split by property, highest to lowest.

**Each bar shows that property's share of the total**, not its share of the largest property. The sort order already tells you the ranking, so the bar's job is showing how concentrated collection is. One property carrying 70% is a very different risk picture from four at 25% each, and only the share of total version makes that readable.

**Properties with zero collection stay in the list.** A property that collected nothing all period is the row most likely to need a phone call. Sorting drops it to the bottom, so it costs nothing to show.

The property rows add back to Total Collection **when both are on the same period.** If this card has been set to its own period, its dropdown says so.

**Tapping a property row opens the payments list for that property**, same as every other drill on the screen. It does not narrow the whole screen to that property. The payments list already takes a property filter, so this is the pattern the app uses everywhere, and one card behaving as a mode switch would be something managers had to learn for one row type only.

A property showing ₹0 is the row most likely to be tapped, and it opens an empty list. That is a job for the empty state, which should name the property and the period and offer the obvious next question rather than saying nothing was found.

**Missing in Figma:** the bars do not match their amounts in the current frame. One property shows a longer bar than another with double the money. The rows are also not sorted. Both look like sample data rather than intent, but worth stating, since a bar whose length does not track its number is worse than no bar.

---

## 6. Collection Trend

One bar per period, split into two parts stacked on top of each other. **Collection sits at the bottom. Due sits above it.**

Collection takes the bottom for a reason. It is the number managers compare from month to month, and only the bottom segment starts from the same line in every bar. Anything stacked above begins at a different height each time, which makes its length hard to judge across bars.

**This chart does not follow the filter at the top.** It has its own range control, because it answers a different kind of question. Every other card answers "which period?", meaning one window and one set of numbers. Trend answers "how far back?", and it needs several periods to say anything at all. Set the screen to This Month and a chart that followed it would show a single bar.

Three ranges:

| Option | What you get |
|---|---|
| **Last 8 weeks** | One bar per week |
| **Last 6 months** *(default)* | One bar per month |
| **Last 12 months** | One bar per month |

The axis says weeks or months, so the unit is never a guess.

The weekly view answers something the monthly one cannot: **how fast money comes in after it is due**. A property that collects most of a month's billing in the first week is healthy. One still collecting in week four has a follow-up problem that no monthly number reveals.

How sharp that reading is depends on the property. Billing day is a setting: it can be a fixed day of the month for everyone, or each tenant's own joining date, and it can be overridden tenant by tenant. Tenants who pay quarterly or yearly are always on their own dates. So a property billing everyone on the 1st gets a clean picture of the collection curve. A property billing on joining dates has billing spread through the month, and the weekly chart reads more like a steady rhythm than a curve. Both are useful. They are not the same picture, and nobody should promise the first one to a property running the second.

The toggle at the top still applies to this chart, even though the filter does not.

### How the bar is drawn

Segments are sized to their values, with a **minimum height** so that a small segment never vanishes. Only genuinely tiny values get lifted to that minimum, and lifting them from invisible to small is the right impression anyway.

**Every segment carries its value printed on it.** Those labels cannot be dropped later for a cleaner look. They carry the precision the shape deliberately does not.

**Units may differ from bar to bar**, so one may read ₹90k and its neighbour ₹1.2L. That is fine here. The bars are sized to their values, so the taller bar is genuinely the larger one, and the shape settles any confusion the digits create.

**In Paid Date view, bar heights are not comparable between periods**, because the total is Collection plus Due, which is not a real quantity. With Collection on the bottom this costs little. The segment people actually compare starts from the same line in every bar.

### What the two parts mean in each view

The Due segment always counts bills by when they were due. The toggle does not change it. **The Collection segment does.**

Take March. ₹1,00,000 billed. The tenant pays ₹60,000 in March and the last ₹40,000 late, in April. April bills ₹1,00,000 and it is all paid in April.

**Paid Date view.** Collection is money that arrived in that period. The two parts are independent numbers.

| Month | Collection (bottom) | Due (top) |
|---|---|---|
| March | ₹60,000 | ₹1,00,000 |
| April | ₹1,40,000 | ₹1,00,000 |

April's collection runs to ₹1,40,000 against ₹1,00,000 billed. That is not a bumper month. ₹40,000 of it is March's money arriving late.

**Due Date view.** Collection is money collected against that period's bills, whenever it arrived. Here Collection is a **part of** what was billed, not a separate number.

So in this view the whole bar is what was billed, and **the segment above Collection is what is still uncollected**, not the full billed amount:

| Month | Collection (bottom) | Still uncollected (top) | Whole bar |
|---|---|---|---|
| March | ₹1,00,000 | ₹0 | ₹1,00,000 |
| April | ₹1,00,000 | ₹0 | ₹1,00,000 |

Both months show as fully collected, a solid bar with no yellow above it.

This matters more than it looks. If the top segment showed the full billed amount instead, a fully collected March would draw a ₹2,00,000 bar that is half green and half yellow, reading as half collected when it is completely collected. The collected money would be counted twice.

*(Recommendation, not yet reviewed by design: the top segment in Due Date view is the uncollected remainder. Flag if it should be read another way.)*

Two things follow in Due Date view:

1. **The whole bar is what was billed, and the yellow part is what is still owed.** You cannot collect more against March's bills than March billed, so the green can never fill past the top. That makes the chart very easy to read: the more yellow you see, the more is outstanding.
2. **The current period always shows a lot of yellow.** Its bills mostly are not paid yet. That is timing, not a collection problem, but it will look alarming every month for every property. The unfinished period must look different from the finished ones. Same rule as the change indicator on the tiles.

### Tapping

**Each segment drills directly.** The Collection segment opens the payments list. The Due segment opens the Dues screen, because it describes bills rather than payments.

Both open on **the period of the bar tapped**, not the period the screen is on. This is the one place on this screen where a drill changes the period instead of narrowing it, and it follows from the chart having its own range. Tap March while the screen sits on This Month and you land on March.

With the weekly range the same applies to a week. Tap the bar for 5 to 11 May and the destination opens on those seven days.

**For later, if managers ask to pick their own range here:** the chart should choose its own bar size to fit. Days for a short stretch, weeks for a few months, months for a year or more, always landing between four and twelve bars so the shape stays readable. Whatever size it picks, the chart has to say so, because a weekly bar and a monthly bar look identical. Not for V1. The three ranges above cover the questions managers actually bring to this chart.

**Missing in Figma:** the weekly range, the Due Date version of the chart, and the treatment for the unfinished period.

---

## 7. Payment Settlement

*The same in both views. Can be set to its own period.*

| Tile | What it means |
|---|---|
| Collected via RentOk | Online money received in the period through RentOk |
| Total Settled | How much of it has reached the owner's bank |
| Unsettled | How much has not |

Collected via RentOk is the total. Settled and Unsettled are its two parts and add back to it.

Money that has started moving to the bank but has not landed counts as **Unsettled**. The system tracks that as its own state internally, but a manager only needs to know whether the money is in the bank or not.

**Money can start moving in two ways**, either through a payout or through a wallet entry. Either one means it has begun. Only money with neither has not started at all.

**Money whose transfer was reversed or failed also counts as Unsettled**, because it is not in the bank. But it is stuck rather than on its way, and a manager waiting for it needs to know the difference. Mark it, rather than letting it sit silently among money that is still in transit.

**The bar below shows two segments, not three.** Settled and Unsettled splitting the total. The total is not a segment of itself.

**Below the bar, one row per destination shows where settled money went.** A destination is usually a bank account, but it can also be a UPI address, so the rows are not always banks. These rows break down **Total Settled only**, so they will not add up to Collected via RentOk. The Unsettled tile above accounts for the difference.

**These rows need a heading saying what they are**, something like *"Where settled money went."* In the current frame they have no label at all, which makes them read as the full picture and leaves a manager thinking money has gone missing.

**Where the bank comes from.** For settled money, use the settlement record. That is the real account it landed in. Never predict a bank for money that has not settled. Which account a payment ends up in can depend on the order payments arrive and the limits set on each account, so a guess can turn out wrong.

**Properties on the older settlement system show a message instead of bank rows**, reading *"Bank details aren't available on the older settlement system. Contact support."* Only the newer system reliably records which bank received the money. Half reliable rows would leave a manager unable to tell which numbers to trust.

**Missing in Figma:** the bar has three segments and should have two. The tile numbers do not add up and the bank rows sum to more than the total, both sample data. The bank rows also have no heading.

---

## 8. The View all sheet

Opens from the **View all** link in the Collection Overview header. A bottom sheet holding every collection number the main screen has no room for.

It follows the screen's selected period and the Paid Date / Due Date toggle, except for the rows that state their own window.

| Group | Rows |
|---|---|
| The period's collection | Total collection, and collection against this period's bills |
| By category | Every bill category present in the period, largest first |
| By stay duration | Short-term tenants, long-term tenants |
| Financial year | Collection this financial year, amount billed this financial year |
| Adjusted collection | This period, this financial year, all time |
| Settlement | Unsettled amount |

**In Due Date view, every row that can show a billed side does so**, reading as collected against billed, the same way the Category tab works on the main card. This sheet is the fuller answer to the same question, so it should answer it the same way.

**Every row is tappable**, opening the payments list filtered to itself. The equivalent sheet on the Dues screen is read only, because its numbers are forecasts with no records behind them. Every number here has real payments behind it, so dead-ending them would waste the sheet.

**The fixed window rows say their window on the row**, such as "this financial year" and "all time". They are the only things in the sheet not following the filter above, and silently mixing windows is how a sheet like this loses trust.

**Stay duration lives only here.** The Dues screen has short stay versus long stay as a full section on its main screen. On Collection it is a secondary cut and belongs in this sheet.

**Missing in Figma:** the sheet is not drawn. The View all link exists in the header and currently opens nothing.

---

## What each number opens

Everything in the payments list opens filtered to the slice named, newest first, on the period the screen is showing.

**Collection Overview, Paid Date view**

| Tile | Opens | Showing |
|---|---|---|
| Total Collection | Payments list | Everything received in the period |
| This Month's Bills Collected | Payments list | Payments against bills due in the period |
| Past Dues Collected | Payments list | Payments against bills due before the period |
| Paid Early | Payments list | Payments against bills due after the period |
| Advance | Payments list | Advance received in the period |
| Current FY | Payments list | Everything received this financial year |
| Settlement Pending | Payments list | Online payments in the period not yet in the bank |

**Collection Overview, Due Date view**

| Tile | Opens | Showing |
|---|---|---|
| Billed | **Dues screen** | Every bill due in the period, paid or not |
| Collected & Adjusted | Payments list | Payments and adjustments against the period's bills |
| Still Unpaid | **Dues screen** | Bills due in the period with nothing against them |
| Advance, Current FY, Settlement Pending | Payments list | As above. These three do not change between views. |

**Everything else**

| Element | Opens | Showing |
|---|---|---|
| Any Collection Breakup row | Payments list | That slice of the period |
| Breakup "Others" | Bottom sheet first, then the payments list from a row inside it | |
| Any Collection Status row | Payments list | That slice of the period |
| Adjusted Collection: Advance, Security Deposit, Caution Money | Payments list | Payments recorded using that method |
| Adjusted Collection: Discount | Payments list | Payments with a discount applied |
| Collection by Property row | Payments list | That property, same period |
| Collection by Property "View more" | Expands the card in place to show the rest | |
| Trend chart, Collection segment | Payments list | **That bar's period**, not the screen's |
| Trend chart, Due segment | **Dues screen** | Bills due in that bar's period |
| Payment Settlement: Collected via RentOk | Payments list | Online payments in the period |
| Payment Settlement: Total Settled | Payments list | Online payments already in the bank |
| Payment Settlement: Unsettled | Payments list | Online payments not yet in the bank |
| Payment Settlement: a bank row | Payments list | Payments settled into that account |
| Payment Settlement "View more" | Expands the card in place to show the rest | |
| Any View all sheet row | Payments list | That row's slice |

The Trend chart is the only place a drill **changes** the period rather than narrowing it, because the chart has its own range. Everywhere else the destination inherits the period the screen is on.

---

## While it loads, and when it breaks

**Every card loads on its own.** The screen never waits for its slowest part. The tiles are a simple sum and appear almost at once. The trend chart pulls twelve periods, and the per-bank breakdown joins payments to settlement records, so both are slower. With All Time selected on a property with years of history, the heavy ones can take seconds. A manager should not watch a blank page while a chart they may never scroll to finishes.

**Each card shows its own skeleton while it works**, shaped and sized like the card it will become, so nothing jumps position as the page fills in. That is what makes independent loading feel calm instead of restless.

**A card that failed must never show ₹0.** This is the rule that matters most here. Zero is a real and meaningful number on this screen, so it can never double as "we do not know". A settlement card that failed and rendered zeros would tell a manager their money has not reached the bank, and they would start calling support about a failure that never happened.

**Empty and failed have to look different**, because a manager acts differently on each:

| State | What happened | What it says | Retry? |
|---|---|---|---|
| Empty | It worked. There is nothing to show. | What is not there, plainly: *"No payments received in May."* | No. Nothing to retry. |
| Failed | It did not work. We do not know what is there. | *"Couldn't load this."* | Yes |

Confuse the two and a manager chases tenants when they should be tapping retry, or the reverse.

**A failed card keeps its heading and shrinks** to fit its message and a Retry. It stays in place, so nothing disappears without explanation, but it does not hold a card-sized hole for one line of text. Copy stays boring on purpose: no apology, no explanation of what went wrong, no error codes. A manager wants the button, not the reason.

**Retrying a card refetches only that card.** Nobody should lose six working cards, or sit through the heavy queries again, to fix one.

### What each card says when it is empty

Half of these are currently wrong in the design, and three of the four wrong ones say something actively misleading.

| Card | Currently says | Should say |
|---|---|---|
| Breakup, Mode tab | Promises a split by "cash, UPI, bank transfer & cheque" | Name the real rows: *"Once payments come in, you will see a split by mode: RentOk, bank transfer, UPI, cash and others."* The current copy promises cash and cheque, and the built card has neither, while RentOk is missing from the list entirely. |
| Collection Status | "No status yet. Dues collected and pending will show up here." | Drop "pending". This card has no unpaid row by design, so the copy promises something that will never appear. *"No collection yet. Once payments come in, you will see what kind of bills they paid."* |
| Adjusted Collection | "No adjustments recorded. Advance payments and deposit adjustments will appear here." | Names two of the four. And "advance payments" reads as the advance received tile, the exact confusion this card has to avoid. *"Nothing adjusted yet. Bills cleared from a deposit, advance, caution money or a discount will appear here."* |
| Collection by Property | "No properties added yet. Add a property and collect dues to see how each property is doing." | Wrong failure. This card is only shown when several properties exist, so the empty case is no collection, not no properties. *"No collection at any property in this period."* |
| Breakup Category, Status and Received by; Trend; Payment Settlement | Fine as written | No change |

There is no empty state at all for the Overview tile row. It needs one, since a period with no activity is a real and ordinary case.

**When every card fails, it is the connection, not the cards.** Replace all of them with a single message and one Retry that refetches the screen:

> **Couldn't load this page.** Check your connection. *Retry*

If some cards fail and others load, keep it per card. The working ones are worth showing. All failed means the connection. Some failed means those cards.

---

## When someone does not have access

The screen shows a lock: *"Analytics Restricted — You don't have permission to view these analytics. Request access from your admin,"* with a Request Access button.

Dues and Collection unlock together. One permission covers both. Expense is separate.

---

## What is already broken in the code this will be built from

This screen has not been built yet. The connection exists but returns nothing. An older, still running Collection screen already does most of this maths and is the obvious place to copy from. Five things in it are wrong, and copying them forward would carry the problems into a screen built to fix them.

1. **A filter for deposit adjustment payments is written incorrectly** and fails the moment it runs. It is not miscounting. It breaks.
2. **For payments made using RentOk credit, the processing charge is subtracted twice.** Only one charge is taken, so every one of these payments is reported lower than it really was. It should come out once. This changes a number people already look at, so finance should sign off before the fix ships. But this screen should be built on the corrected version.
3. **Caution money is counted twice on the homescreen.** It sits in the deposits group and in the "other" group at the same time, so the same rupee appears in both. The Category tab on this screen has to put every rupee in exactly one place.
4. **Refunds come off in one calculation and not in another.** One path subtracts them, another does not. Since this screen needs refunds off everywhere, the path that skips them cannot be reused as it stands.
5. **Caution money adjustments are handled inconsistently.** Some calculations exclude them alongside deposit and advance adjustments. Others never mention them. This screen treats all three the same way.

Underneath all five sits the real problem: **the list, the widget and the homescreen do not currently produce one answer.** That is what this screen is being built on top of, and it is why every number here has to come from one shared total.

---

## What has to agree with what

This screen exists because managers do not trust a number they cannot explain. Each of these is a place where two numbers claiming the same thing could drift apart.

**The homescreen collection number and this screen's total must match.** The homescreen says money came in. This screen explains what kind. If the two disagree, the explanation is worthless. They have to be the same number wherever the label means the same thing.

**Every number here comes from one shared total**, not from separate calculations that happen to agree today. The tiles, the breakdowns, the trend, the property split and every drill destination are different views of one base. Separate calculations drift the moment one of them is touched.

**A payment is either fresh money or an adjustment, never both.** Every payment lands in exactly one bucket.

**A drill destination adds up to the number that was tapped.** Tap ₹30,000 and the list totals ₹30,000. If it does not, one of the two is wrong.

**A period in the Trend chart equals that period opened directly on the screen.** The chart has its own range, which makes it the easiest place for a second definition to creep in.

**Settlement Pending in the tile row and Unsettled on the Payment Settlement card are the same measure**, so they show the same number whenever both are on the same period.

**Within a view:** in Paid Date view the four middle Overview tiles add to Total Collection, and each Breakup tab adds to the same total. In Due Date view, Collected & Adjusted plus Still Unpaid equals Billed, and each Breakup tab adds to Collected & Adjusted. All of it holds when the cards are on the same period.

---

## Build guidance

1. Build the deposit adjustment filter and the RentOk credit maths fresh. Do not copy either from the older screen.
2. Caution money adjustments need building from scratch. Deposit and advance adjustments cannot be reused for them.
3. Count Discount as its own total, not as a fourth payment method.
4. Build Under Eviction as active tenants who have been served notice.
5. Settle how bookings are identified in tenant data before building the Status tab.
6. Give unlinked payments a visible row on the Status tab, or say plainly that they are excluded there.
7. Sort every property list highest to lowest, and keep zero collection properties in it.
8. Include part payments against bills still open. A tenant paying half their rent counts as collection.
9. Payment Settlement bank rows break down settled money only, and need a heading saying so.
10. **Advance, Current FY and Settlement Pending never follow the due date filter.** They always count by when money arrived, in both views. Everything else on the screen switches.
11. In Due Date view, all four kinds of adjustment (deposit, advance, caution money and discount) count towards Collected & Adjusted. Leaving any of them out puts already-cleared bills into Still Unpaid and sends managers chasing tenants who owe nothing.
12. In Due Date view, Category and Status carry a billed comparison per row. Mode and Received by show Collected & Adjusted only. All four still add to the same Collected & Adjusted total.
13. **Changing the filter at the top resets every card to follow it.** The Trend chart is the exception. The filter never touches it.
14. **Never compare an unfinished period against a finished one.** The change indicator compares against the same point in the previous period, and says so. The same rule marks the in-progress bar on the Trend chart.
15. **Three pieces of genuinely new work:** the Billed, Collected & Adjusted and Still Unpaid tiles; the Due Date version of the Trend chart; and the View all sheet. All three need designing, and the first two need counting from bills. Everything on this screen today counts from payments.
16. The numbers that must always add up: in Paid Date view, the four middle Overview tiles add to Total Collection, and each Breakup tab adds to the same total. In Due Date view, Collected & Adjusted plus Still Unpaid equals Billed, and each Breakup tab adds to Collected & Adjusted. All of this holds when the cards are on the same period.
17. **On the Trend chart, Collection is the bottom segment.** Only the bottom segment shares a baseline across bars, and Collection is the number managers compare. Segments are sized to their values with a minimum height so nothing disappears, and every segment keeps its printed value.
18. **In Due Date view the Trend chart's top segment is what is still uncollected, not the full billed amount.** Otherwise collected money is counted twice and a fully collected month draws a half yellow bar.
19. **Three numbers leave this screen for Dues:** the Billed and Still Unpaid tiles in Due Date view, and the Due segment of the Trend chart. Each carries its period across, and the back control says "Collection".
20. **Every card loads, fails and retries on its own.** Skeletons match the shape of the card they become. A failed card shrinks to its heading, a short message and a Retry, and refetches only itself. A failed card never renders ₹0.
21. **Empty and failed must not look alike.** Empty names what is not there and offers no retry. Failed says it could not load and offers one.
22. **When every card fails, show one screen-level message with one Retry** instead of seven. Some cards failing stays per card.
23. **Export stays on the existing list.** Do not build a separate export path for this screen. An export that disagrees with the list it came from is another number nobody can explain.
24. **Check the knock-on effects before launch.** Collection numbers feed notice reports, tenant ledgers and the deposit and advance balances shown elsewhere. Adding Discount to Adjusted Collection, and counting adjustments as collected in Due Date view, both touch those. They must not contradict what a tenant's ledger says.
25. **Fix what the payments list says on arrival.** It needs to name the slice in place of its fixed title, name the filter in its empty state with a way on, and show the active filter without the manager having to scroll. This is work on the list screen, and it applies to every analytics screen that drills into it.

---

## Everything missing or wrong in Figma, in one list

| Where | What |
|---|---|
| Overview tiles | "Past Bill" should read "Past Dues Collected". "Future Bill" should read "Paid Early". |
| Overview tiles | The Advance tile is not drawn |
| Overview tiles | The Due Date version of the row is not drawn. Three tiles needed. |
| Overview tiles | No treatment for a negative total, an adjustments only period, or an unfinished period |
| Breakup → Mode | No Cash row, though the empty message promises one |
| Breakup → Mode | Empty message lists categories that do not match the real ones |
| Breakup → Received by | **No Others row exists at all.** Four staff are named. The spec is top 3 plus Others, and the Others row needs to say how many people it holds. |
| Breakup | The period total line above the rows does not match the rows below it (sample data) |
| Breakup → Category | Typo: "Electricty" |
| Breakup and Status | Bar lengths do not track their values, same as the property card |
| Overview tiles | Tile reads "May's Bills Collected". It should read as the current period, not a hard-coded month. Same for "May's Due" on Collection Status. |
| Overview tiles | No empty state for the tile row |
| Time filter | The filter is drawn as a stepper reading "Today". It should be the five option dropdown, defaulting to This Month. Design is stale here. |
| Collection Trend | Only about four and a half bars fit the card at the six month range, so it needs horizontal scrolling or narrower bars |
| Collection Trend | The legend is drawn as two pill chips that look tappable. If they toggle the series, that is unspecified. If they do not, they should not look like buttons. |
| Collection Trend | The empty state frame shows a "This Month" dropdown instead of the range selector |
| Payment Settlement | One row is a UPI address, not a bank account. The copy calls them all bank rows. |
| Every card | Each header carries an ⓘ. What each one opens is unspecified. |
| By Property and Settlement | "View more" is a chevron that expands in place. It is not a link to a separate full list. |
| Empty states | Four of the eight are wrong. See the empty state table above. |
| Breakup | Trophy badge is pinned to the RentOk row. It should follow whichever row is largest. |
| Breakup | No billed comparison version of Category and Status for Due Date view |
| Collection by Property | Bar lengths do not match their amounts, and rows are not sorted (sample data) |
| Collection Trend | Due is drawn as the bottom segment. Collection should be at the bottom instead. |
| Collection Trend | No weekly range, no Due Date version, no treatment for the unfinished period |
| Payment Settlement | Bar has three segments. It should have two. |
| Payment Settlement | Bank rows have no heading |
| Payment Settlement | Tile and bank row numbers do not add up (sample data) |
| Future periods | No empty state for a future period in Paid Date view, and no switch to Due Date prompt |
| View all sheet | Not drawn at all. The link exists and opens nothing. |

---

## Open items

Three things need an owner before build starts.

1. **Do the deeper drills ship on day one?** Some pages below the first payments list do not check the viewer's permission the way the list does. Either those checks get fixed first, or drilling stops at the list until they do. Nobody has decided.
2. **Which permission gates this screen?** The lock is described as an analytics permission covering Dues and Collection together. The existing collection list gates on the invoice viewing permission instead. Those are two different answers and nothing reconciles them.
3. **Is anyone actually given the narrowed view?** The system has a permission limiting a team member to only tenants they added. It is built and working across Collection and Dues. Product says nobody has it. If anyone does, their numbers are being quietly narrowed today with nothing on screen saying so, and two people looking at the same property would see different totals with no way to explain the gap.

One thing sits outside this screen:

- **The payments list needs to say what it is showing.** Named slice in place of its fixed title, a filter aware empty state with a way on, and the active filter visible without scrolling. Every analytics screen drills into this list, so it is worth doing once rather than per screen.

## A note on scope

The original plan was a two week build with a stated order for what to cut if time ran short. That scope has since grown: a second whole view, a View all sheet, a weekly trend range, three new bill side tiles, per card loading and failure states, and the per bank settlement breakdown.

None of that is wrong. All of it was decided deliberately. But the cut order is gone, and if the cycle slips there is now no written answer to what goes first. Worth settling before build rather than during.

## Assumptions worth knowing

**All Time is the expensive option.** On a property with years of history it makes every card scan the lot at once.

**How useful the weekly trend view is depends on the property.** It shows the collection curve clearly when everyone is billed on the same day. It reads flatter at a property billing tenants on their own joining dates. Both are real, and neither is a fault.
