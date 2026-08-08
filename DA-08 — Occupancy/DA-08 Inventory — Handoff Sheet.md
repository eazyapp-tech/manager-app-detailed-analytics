---
title: DA-08 Inventory — Handoff Sheet
date: 2026-08-08
tags: [rentok, inventory, occupancy, detailed-analytics, handoff]
status: v2 · rewritten lean onto the suite template · developer handoff
owner: Sanchay
---

# Inventory — Handoff Sheet

Everything on the Inventory analytics screen: what each number means, what the Bed / Unit toggle does to it, what window it covers, what happens when it is tapped, and what it shows when there is nothing to show.

---

## What is in here

| | Section | For |
|---|---|---|
| **1** | [Build status](#1-build-status) | everyone |
| **2** | [Where this lives](#2-where-this-lives) | everyone |
| **3** | [What every number counts](#3-what-every-number-counts) | backend + copy |
| **4** | [How the screen behaves](#4-how-the-screen-behaves) | backend + design |
| **5** | [Overview Snapshot](#5-overview-snapshot) | backend + design |
| **6** | [View all sheet](#6-view-all-sheet) | backend + design |
| **7** | [Occupancy Status](#7-occupancy-status) | backend + design |
| **8** | [Vacant Room Status](#8-vacant-room-status) | backend + design |
| **9** | [Upcoming Vacancy](#9-upcoming-vacancy) | backend + design |
| **10** | [Agreements ending soon](#10-agreements-ending-soon) | backend + design |
| **11** | [Occupancy Trend](#11-occupancy-trend) | backend + design |
| **12** | [Where is my property losing money?](#12-where-is-my-property-losing-money) | backend + design |
| **13** | [Property Wise Occupancy](#13-property-wise-occupancy) | backend + design |
| **14** | [What each number opens](#14-what-each-number-opens) | backend |
| **15** | [Who can see this](#15-who-can-see-this) | backend |
| **16** | [What each card shows when it is empty, healthy or broken](#16-what-each-card-shows-when-it-is-empty-healthy-or-broken) | design + copy |
| **17** | [What this screen is not](#17-what-this-screen-is-not) | everyone |
| **18** | [Build guidance](#18-build-guidance) | backend |
| **19** | [Open items](#19-open-items) | everyone |
| **20** | [Design file: what needs fixing](#20-design-file-what-needs-fixing) | design |
| | [Where the measured figures came from](#where-the-measured-figures-came-from) | anyone re-checking a decision |

**Building the backend?** Read 1, 3 to 15, 18, 19, and the appendix. Start with Build guidance; it holds the traps.

**Working on the design?** Read 3, 4, 5 to 13, 16, 20.

**Just need the decisions?** Read 3, 4, 17 and the appendix.

---

## 1. Build status

The screen is **not built**. The backend serves one empty placeholder block for Inventory. Nothing here is broken; it is unwritten.

Three things that already exist matter to the build:

- A **present-moment occupancy calculation** already powers the homepage occupancy widget and the rooms list. Its counting rules seed this screen's Now numbers, with the corrections in Build guidance.
- Something already produces an occupancy figure for a past period, and it does not answer the question this screen asks: a tenant who stayed three days of a month weighs the same as one who stayed all month. Section 4 says what a period number means here; check anything existing against that before building on it.
- The **rooms list screen** this screen drills into is built and live.

The platform is migrating every property to the structure where each bed is a real record. About 1 in 100 properties is on it today and the share grows with every cohort. Numbers that are exact on migrated properties and approximate on the rest say so where they are defined. Build the exact version first and the approximation as the fallback.

---

## 2. Where this lives

Manager app, property header, the **Inventory** tab.

The suite's tab row: **Financial** (Dues · Collection · Expense) · **People** (Leads · Bookings · Tenants · Old Tenants) · **Inventory** · **Complaints**. Inventory stands alone, no sub-tabs, and is the only tab that carries a view toggle instead.

The design file draws an older row (Financial · Inventory · Tenant · Issues · Leads). The settled row above wins; the redraw is on the design-fix list.

---

## 3. What every number counts

The vocabulary of the screen. Several of these words are fixed here for the first time, and the other analytics screens borrow them.

| Term | Meaning |
|---|---|
| Bed | The smallest rentable space: one person's place to sleep |
| Rentable | Total beds or units, minus disabled ones and minus rooms that declare no capacity. The denominator of the occupancy rate |
| Occupancy rate | Occupied divided by rentable, **always counted per bed in both views**. The label says so: "Occupancy Rate (by beds)" |
| Booked | A confirmed arrival who has not moved in yet. Always a layer inside another number, never a total of its own |
| Available | Vacant space with nothing lined up. What a manager actually has to sell |
| Never rented | Rentable space that has never had a tenant. Not slow turnover: space that has never earned |
| Over-occupied | More living tenants in a room than its capacity. The reason a rate can read above 100% |

The terms below carry rules, so they keep their own headings.

### Bed, on the two structures

On a **migrated property** a bed is a real record: name, rent, history. On one not yet migrated it is arithmetic, a 3-sharing room has three beds with no individual identity. Counts work everywhere; per-bed detail exists only on migrated properties, and cards show the count and omit the detail rather than guess.

### Unit

**A unit is whatever is let as one lettable space.**

- A sharing room, a studio, a 1RK: one unit each.
- A flat is **one unit** only while a single tenancy holds all its rooms.
- A flat is **one unit per room** the moment it does not, including when one bedroom is let and the rest stand empty. If the tenant were taking the whole flat, they would be recorded against the whole flat; holding one room is itself the answer.
- **Empty space counts as rooms.** An empty flat could be sold either way and nobody has decided yet. Room-wise is the deliberate bet until someone challenges it.

Two consequences, stated rather than hidden: a whole-let flat that empties splits into as many units as it has rooms, so totals can move without anyone arriving or leaving, and the trend chart says so. And two properties with identical inventory must count identically; today they can differ. The rule is the requirement; how it is computed is engineering's call.

### Occupied

A bed with a living tenant. A tenant under notice **still counts as occupied**: they live there today. In Unit View, an occupied unit has all its beds occupied.

### Semi-vacant *(Unit View only)*

A unit with some beds occupied and some free: where a bed can be sold without anything else changing. Exists only in Unit View, and only where some unit holds more than one bed; for a property of singles the slice never appears.

### Vacant, and its two layers

Rentable space with **nobody living in it**. Bed View: a bed nobody is in. Unit View: a unit with no beds occupied (partly filled is semi-vacant, not vacant).

**A bed with someone booked into it is still vacant.** A booking is a promise, not a person; until they move in, the bed earns nothing, and counting it as filled would let a property report itself full on arrivals that never happen. Vacant splits into two layers, both always shown:

- **Booked**: someone is due to arrive.
- **Available**: nothing lined up. Stated, never left to subtraction.

### Booked, precisely

- **Confirmed only.** Pending requests are named as awaiting approval and never sit inside occupancy; rejected and cancelled bookings are excluded from every count. On a property that accepts bookings automatically, a booking with no separate confirmation step is confirmed.
- **The layer belongs to the bed, never the room around it.** A booking on an empty bed is a layer inside Vacant, even when that bed sits in a partly filled unit; the unit itself stays semi-vacant. A booking behind a tenant under notice is a layer inside Occupied, shown as the "already replaced" line. Counting booked as its own slice would pull beds out of the states that describe today, and would count one bed twice wherever both apply.

### Disabled for rent

Space deliberately switched off from renting. **This is also the maintenance state**: repair, owner use and storage are all this one flag. One name on screen, maintenance named in its explanation. Excluded from rentable and the rate; shown only when the property has any.

### Under notice

A living tenant with a **confirmed** move-out date: occupied today, free soon. A date raised but never approved does not count, matching every sibling. Two layers ride on it, shown with it:

- **Past their date**: the date has passed and the tenant still shows as living. Either the record is stale or they overstayed; both deserve attention.
- **Already replaced**: a confirmed arrival is lined up for that bed. The gap is handled.

### Unknown *(vacancy age only)*

Let before, but with no recorded emptying date, so it cannot be aged. A migration artifact: shrinks as properties migrate, hides at zero.

### Words to be careful with

| Word | On this screen | Elsewhere |
|---|---|---|
| Unit | A lettable space as defined above | The design file says "Unit View" where older docs said Room View; unit is the settled word, and it is not a synonym for room |
| Under notice | The suite's one word for a tenant with a confirmed leaving date | Collection's design label "Under Eviction" has been renamed to match. No second variant survives |
| Vacant | Nobody living there, booked included | The rooms list treats a booked bed as taken. Section 14 carries the consequence |
| Booked | Confirmed arrivals only | The rooms list counts any booking record, confirmed or not |
| Now | The filter's present-moment setting | "Live" stays a kind label only (a section no filter changes); this screen never uses it as a filter setting |

---

## 4. How the screen behaves

### The time filter

One filter at the top: **Now (default) · This Month · Last Month · Current FY · Custom · Coming up.**

Two deliberate differences from the siblings:

- **No All Time.** Occupancy averaged since a property opened answers nothing; the resting state of a screen about space is right now, and Now is that.
- **The word is Now, not Live.** The siblings use "live" for sections no filter can change; reusing it as a filter setting would give one word two opposite meanings on adjacent tabs.

**A period is an average of days.** "78% last month" means 78% of rentable beds were filled on an average day of it. Not a last-day snapshot; the only reading fair to a property that emptied and refilled mid-month.

**Coming up** asks for a future date (default 30 days out) and answers what the property will look like then: today's occupied, minus confirmed move-outs before that date, plus confirmed arrivals. Nothing on a forward setting ever invents an event.

Cards with their own dropdown follow the suite rules: same options, the top filter pulls every card back into line, one card can be deliberately set aside.

### What every number does on every filter setting

| Kind | Meaning |
|---|---|
| **Live** | Always the current snapshot. No filter setting changes it. Says "as of today" on its face |
| **Time-scoped** | Moves with the filter. On this screen these are state numbers, so a window shows their average, not a tally |
| **Forecast** | Counts forward from today, from confirmed facts only |

| Number | Now | This Month · Last Month · Current FY · Custom | Coming up |
|---|---|---|---|
| Overview tiles | As of today | Averaged across the days of the window | Projected at the chosen date |
| View all sheet | As of today | Averaged across the days of the window; the booked and already-replaced layers hide, and the three past rows never move | Projected at the chosen date, except the three past rows |
| Occupancy Status | As of today | Averaged across the days of the window; the booked and already-replaced layers hide | Projected at the chosen date |
| Vacant Room Status | As of today | As of today | As of today |
| Upcoming Vacancy | From today onwards | From today onwards | From today onwards |
| Agreements ending soon | From today onwards | From today onwards | From today onwards |
| Occupancy Trend | Each keeps its own range; the filter does not change it | Each keeps its own range; the filter does not change it | Each keeps its own range; the filter does not change it |
| Losing money: occupancy and revenue loss | As of today | Averaged across the days of the window; revenue loss is the window's summed cost | Projected at the chosen date |
| Losing money: Days to Fill | Each keeps its own range; the filter does not change it | Each keeps its own range; the filter does not change it | Each keeps its own range; the filter does not change it |
| Property Wise Occupancy | As of today | Averaged across the days of the window | Projected at the chosen date |

The three past rows, on every setting: Days to Fill, never rented, past their date. They describe the past and never move.

The four filter-exempt cards say so in the card header, beside the title: "As of today" on Vacant Room Status, "From today onwards" on Upcoming Vacancy and Agreements ending soon, its own range control on Occupancy Trend. Days to Fill is exempt inside a card that is not: it measures past turnovers and looks back on every setting.

### Periods that have not finished

A finished period compares against the previous finished period, like for like. An unfinished one compares against the **same elapsed days** of the previous period, marked as unfinished. On Custom, chips compare the same number of days immediately before the range. On the money screens "same point" is a running total to the same day; here it is the average over the same number of days, or a fortnight of this month gets compared against a whole month of last.

### Periods that have not happened yet

The future belongs to Coming up; **Custom stops at today**. Coming up shows projections with no change chip: a projection has nothing to be compared with.

### Change chips

**This screen has no single good direction. Polarity is per tile:**

| Number | Rising is |
|---|---|
| Rentable | Neutral: grey chip |
| Occupied | Good: green up, red down |
| Occupancy Rate | Good: green up, red down |
| Vacant | Bad: red up, green down |
| Under Notice | Bad: red up, green down |

⚠ This is component work, not a table. The shared chip component defaults to one direction per screen; this row needs good-up, bad-up and a neutral grey the shared component does not have yet; Collection and Expense log the same need. The design currently draws Vacant and Under Notice green when rising; both are inverted.

**On Now**, a chip compares against the same point one month ago and says so: "▲ 4% vs this day last month." A state number has a real predecessor, which is why this screen keeps chips on its default while Tenants does not.

### When a number is red

The suite rule: red for unmet obligations and passed dates only. Here that is the chips above, the past-their-date line, and one deliberate exception: the vacancy-age chart marks its largest **aging** bar red (section 8), never rented and unknown excluded so red stays rare enough to mean something.

### The action bar

The "Ask RentOk AI" bar at the bottom is app shell, not this screen's component; this sheet does not define it.

Each card may carry one insight line: positive is plain text, negative carries a button opening the AI chat pre-filled with "How can we improve this?". Ships on Inventory first because a rooms-scoped AI analyst already exists; not a suite pattern, and the siblings cannot copy it yet.

### Loading, failure, sorting, entry

Suite rules unchanged: every card loads, fails and retries alone; skeletons match their card; a failed card never shows a number, keeps its heading, shrinks to "Couldn't load this" with a per-card Retry; empty and failed look different; all cards failing means "Couldn't load this page. Check your connection."

Every property-wise and breakdown list sorts highest to lowest; each card states its own key. The screen is reached from the Inventory tab in the property header; screens drawn around it in the design file are reference context, not part of it.

### The view toggle: Bed View / Unit View

One toggle at the top: **Bed View (default) · Unit View**. Bed View is the default because the platform is migrating to bed-level records; the toggle is where the product is going.

The toggle changes what gets counted, not the facts; every card section states what it shows on each side. Where the two sides are identical (a property of all singles, appendix), **the toggle is hidden**: a control that does nothing teaches people to ignore controls.

Cards may carry their own copy of the toggle, behaving like a card's own date dropdown: same options, the top toggle pulls every card back into line, one card can be deliberately set aside to compare views.

Three cards ignore the toggle because their subject has no bed-level version: Vacant Room Status and Upcoming Vacancy count per room, Agreements ending soon counts agreements, which belong to tenancies. Each says so on its face.

---

## 5. Overview Snapshot

Five tiles in a row wider than the card; how it is navigated is an open design decision (section 20). The header reads **"Overview Snapshot"** with the active window beside it: "Now", "Last Month", "By 15 Sep".

| Tile | Bed View | Unit View |
|---|---|---|
| Rentable | Rentable beds | Rentable units |
| Occupied | Beds with a living tenant | Units with all beds occupied |
| Occupancy Rate | Occupied ÷ rentable beds | Same number, always by beds, labelled so |
| Vacant | Rentable beds nobody is living in | Units nobody is living in |
| Under Notice | Beds whose tenant has a confirmed move-out date | Units with at least one such tenant |

Labels switch with the toggle: "Rentable Beds" against "Rentable Units". Each tile carries a change chip per section 4's polarity table. **View all** opens the sheet in section 6.

All five tiles appear again on the Occupancy Status card below, deliberately: the tiles summarise, the donut explains. **One number each, computed once, shown twice.** Two rates disagreeing on one screen loses a manager's trust in every number on it.

---

## 6. View all sheet

Everything the tile row knows, as three plain questions. Indented rows are layers inside the row above, never separate totals; booked and available split Vacant exactly, the other layers overlap their parent and are not meant to sum. Rows follow the toggle, except Semi-vacant (Unit View only). Every row that can open a list does (section 14).

**What have I got**

| Row | Meaning |
|---|---|
| Total | Everything, before anything is subtracted |
| Disabled for rent | Switched off from renting, maintenance included, plus rooms declaring no capacity. Shown only above zero |
| Rentable | Total minus disabled |
| Occupied | As defined |
| Semi-vacant | Unit View only; hidden for all-singles properties |
| Vacant | As defined |
| — of which booked | Someone is due to arrive |
| — of which available | Nothing lined up: what there is to sell |
| — of which never rented | Never had a tenant. Space that has never earned |
| Over-occupied | More living tenants than capacity: the reason a rate can read above 100% |

**Who's moving**

| Row | Meaning |
|---|---|
| Under notice | Living tenants with a confirmed move-out date |
| — past their date | Move-out date passed, still recorded as living |
| — already replaced | A confirmed arrival lined up for the same bed |
| Bookings with no space allocated | Somebody confirmed and given nowhere to sleep. Counted as its own row because there is no bed to layer it on. Belongs to Inventory until the Bookings screen exists, which must then reference it, not rebuild it |

**How fast**

| Row | Meaning |
|---|---|
| Days to Fill | Across beds refilled recently: days between one tenancy ending and the next beginning. The same measure the losing-money card shows per group; this is the overall figure |

---

## 7. Occupancy Status

The donut card. Centre: the occupancy rate, always by beds, labelled so.

**Bed View slices:** Occupied · Vacant · Disabled for Rent (only when present).
**Unit View slices:** Occupied · Semi-Vacant · Vacant · Disabled for Rent (only when present). A partly filled unit is neither full nor empty; forcing it into either would misstate the property.

**Vacant carries its two layers in the legend, not as slices:** "Vacant 38: 17 booked, 21 available". Available is the number a manager acts on, so it is stated, never left to subtraction.

**The legend adds up to Total, and the rate divides by Rentable.** Occupied + Semi-Vacant + Vacant = rentable; adding Disabled for Rent brings it to total. The header carries both, switching with the toggle: "Total Beds: 326 · 318 rentable". A legend summing to one number beside a percentage computed from another is checked once, cannot be reconciled, and is never trusted again.

**Chips row**, each shown only above zero. Booked is not a chip; its number lives in the Vacant legend split, once:

| Chip | Meaning |
|---|---|
| Under Notice | As defined, with its layers as a second line when above zero: "4 under notice · 1 past their date · 2 already replaced" |
| Over-occupied | More living tenants than capacity |

Where the rate exceeds 100%, the card states both halves in one sentence: "104%: 3 rooms are over-occupied while 5 beds elsewhere sit empty." Seeing only the first reads as a data error. No double-booked chip; the case folds into over-occupied, and preventing it belongs to the booking flow (section 19).

**Money tiles:**

| Tile | Meaning |
|---|---|
| Avg. Rent per Occupied Bed / Unit | What one filled bed or unit earns on average. Label switches with the toggle |
| Avg. Revenue per Rentable Bed / Unit | What every rentable bed or unit earns on average, filled or not. Lower than its neighbour whenever anything is vacant; the gap is what vacancy costs. On an over-occupied property it can read higher, the same signal the over-100% sentence explains. The design labels this tile "Available", which is this screen's word for vacant space with nothing lined up; the label becomes Rentable |

**Insight line**, per the rule in section 4: "₹5.2K lost to vacancy this month" with the button; "Occupancy up 4% on last quarter" as plain text.

---

## 8. Vacant Room Status

How long currently-empty rooms have been empty. **Counted per room, always**; ignores the view toggle, says "by rooms" and "As of today" on its face.

**A room is empty when nobody is living in it**, booked rooms included: a room booked for next month has still stood empty sixty days. A room with even one tenant never appears here; its free beds still count as vacant on the tiles but have no room-level emptiness to measure. This card counts rooms while the tiles count beds or units, which is why it says "by rooms" on its face.

Six bars:

| Bar | Meaning |
|---|---|
| 0–7 days | Empty a week or less |
| 8–15 days | One to two weeks |
| 16–30 days | Two weeks to a month |
| 31+ days | Over a month: the turnaround problem |
| Never rented | Added to the system, never filled. No "empty since" date exists |
| Unknown | Let before, but nothing records when it last emptied, so it cannot be aged. Shrinks as properties migrate; hides at zero |

The design draws only the four aging buckets. In production those four together describe a small fraction of vacant inventory (appendix); never rented is by far the largest state, and without its bar the card reports the minority of the problem while looking complete.

Red marks the largest of the **four aging bars only**, the design's rule kept as a deliberate exception to the suite's red-for-obligations rule. Never rented and Unknown sit outside the contest: never rented is the biggest bar on almost every property, and a constant red stops pointing at anything.

⚠ **Nothing in the product records when a room became empty.** The four aging bars depend on that being built first and cannot open lists until it is. Never rented and Unknown are different: both can be computed from history that already exists (the appendix measured them), and their bars wait only on a rooms-list filter. See Build guidance.

---

## 9. Upcoming Vacancy

Who is confirmed to be leaving, how soon. Built only from tenants who have given notice, no predictions. Counted per room; ignores the view toggle; "From today onwards" on its face.

Four bars: leaving within 0–7 · 8–15 · 16–30 · 31+ days.

This card and Vacant Room Status are adjacent bar charts over day-buckets, one looking back at empty rooms, one forward at occupied ones. The subtitles carry the direction: "how long empty rooms have been waiting" against "when notice-given tenants leave". Bucket labels agree: **31+ days** on both, so day 30 falls in one bucket.

---

## 10. Agreements ending soon

Agreements reaching their end date, in 30 / 60 / 90-day bands, with a count and a way into those tenants.

The **earliest warning on the screen**: notice given is a decision made, an agreement running out is a decision not yet made. Most renew, but each is a conversation to have before the date. Agreed end dates are facts about paperwork, not confirmed departures, and the card says so.

Counts **agreements**, so it ignores both the view toggle and the time filter; "From today onwards" on its face. The axis says "No. of agreements": the suite's word is agreement, never lease.

⚠ One tenancy can sit on this card and Upcoming Vacancy at once, notice given and agreement ending. Two lenses on one departure, not double-counting, but the two cards must agree on the person: if they leave on the 15th, this card cannot claim a different date.

---

## 11. Occupancy Trend

Occupied against vacant over time, stacked bars per month. Its own range control: **6 / 12 / 24 months**, default 6. Exempt from the time filter.

This chart plots the **level** deliberately, the suite's one exception to trend-plots-movement: how full the property stood is the question only this screen can answer, and the movement version (move-ins against move-outs) belongs to Tenants, where it is specified.

- Switches with the view toggle: beds in Bed View, units in Unit View.
- A month in progress renders as an in-progress bar, marked per the suite rule.
- History before the property joined does not exist and is not invented; the chart starts where the data starts.
- Where the unit definition changed mid-history (a whole-let flat emptied and became room-wise), totals can move without anyone arriving or leaving; the card's explanation says so.

Insight line per section 4's rule.

---

## 12. Where is my property losing money?

The cost of the gaps, grouped by **sharing type**: Single · Double · Triple · 4+ sharing. Not by flat type: most rooms carry no flat-type label at all (appendix), so that axis would collapse to one giant row, while sharing type is populated everywhere and is how a PG owner already prices. Properties with genuine flat-type variety get those as their own groups alongside.

**Headline: the total.** The card answers its own question in one number at the top; the rows below break it down and must always reconcile with it.

Per group row:

| Field | Meaning |
|---|---|
| Occupancy % | Occupied ÷ rentable beds in this group |
| Days to Fill | Average refill time from recent turnovers, always with its open tail: "avg 14 days · 2 beds still open, longest 86 days". Shows "—" with "no recent turnover" where none exists |
| Revenue loss | The rent of this group's vacant beds, per month, with its share of the total. The bar carries the share; the number carries the rupees |

**Revenue loss changes meaning with the window, and the card says which.** On Now: what is being lost per month, the monthly rent of every currently-vacant bed. On a past period: what vacancy actually cost, each bed's empty days times its daily rent (monthly rent over that month's real day count, not 30); this needs the same missing history as the vacancy card and waits on it. The wording changes with it: "losing ₹200K a month" on Now, "vacancy cost you ₹185K last month", "₹190K a month at this rate" on Coming up.

**Pricing an empty bed:** its own configured rent first, then what the space was last let for, then the group's average. The middle step is unavailable for most beds today, same reason the vacancy card carries an Unknown bar; build it so it improves as the migration lands.

⚠ **Rent is sparsely configured** (appendix), so the figure carries its basis on its face: "based on 34 of 120 vacant beds with rent set", counting only beds with a rent actually set, not fallback-priced ones. Below the agreed coverage (threshold open, section 19) the figure gives way to the nudge: "Set rents on your vacant beds to see what vacancy costs you", which is itself what fixes the coverage.

Sorted by revenue loss, highest first. Moves with the time filter and the view toggle; groups re-slice, the money stays with the beds.

---

## 13. Property Wise Occupancy

**Hidden when only one property is in scope**, back on its own when more than one is selected. A large share of active properties belong to multi-property owners (appendix); for them this is the "which building is hurting" card.

Per property: name, occupied / rentable, a progress bar. **Sorted highest to lowest by vacant space**: vacant beds in Bed View, vacant units in Unit View, not by rate, which would put the fullest property on top and bury the worst under View more. A property with 200 empty beds outranks one with 5, the order a manager would work in.

Moves with the time filter (own dropdown per the suite rules) and the view toggle. "View more" expands in place. **A property row opens the rooms list for that property**, like every drill on this screen; it does not narrow the screen to that property. Properties with nothing rentable still appear, at zero: a missing property reads as an error, a zero reads as a fact.

⚠ However the account grouping is computed, every property the manager can open belongs to some row. The obvious grouping field is missing on a large share of active properties (appendix), and silently dropping them would make this card lie by omission.

---

## 14. What each number opens

### The rules

A drill filters a list, never re-scopes the screen, trend bars excepted. The destination follows the record's kind: space numbers open the rooms list, person numbers the tenants list, booking numbers the bookings list. The destination opens on the same view and the same properties, and the back control names this screen. Records add back to the number tapped. Rates and averages are not tappable: no set of records adds back to one.

In Bed View, a bed count opens the rooms holding those beds; no bed list exists, and the destination says so ("12 rooms holding 18 vacant beds") rather than letting a manager count rows and find a different number.

### When the window changes what a tap shows

A period average has no list behind it: no set of beds *is* "78% last month". A drill from any period number opens **today's** matching list and says so on arrival: "showing these rooms as they are now". Coming up taps re-route the same way. Only on Now do records add back exactly.

<details>
<summary><strong>Expand:</strong> the full tap matrix with arrival filters, the gaps, and what the destination says when you arrive</summary>

### The tap matrix

Checked one by one against the filters the rooms, tenants and bookings lists actually accept today.

| You tap | What opens | Arriving filtered to | Ready? |
|---|---|---|---|
| **Overview tiles and View all rows** | | | |
| Total | Rooms list | all rooms | ✅ |
| Rentable | Rooms list | all except disabled | ✅ |
| Occupied | Rooms list | fully occupied (Unit View) or has an occupied bed (Bed View) | ✅ |
| Vacant | Rooms list | fully vacant (Unit View) or has a vacant bed (Bed View) | ⚠ the list treats a booked bed as taken, so it returns fewer rows than the number said |
| Semi-vacant | Rooms list | semi-vacant | ✅ |
| Disabled for rent | Rooms list | disabled for rent | ✅ |
| Vacant — of which booked | Bookings list | confirmed | ✅ |
| Vacant — of which available | Rooms list | vacant, minus confirmed arrivals | ⚠ the list cannot subtract confirmed arrivals |
| — of which never rented | Rooms list | never occupied | ❌ filter to add, over history that already exists |
| Over-occupied | Rooms list | over-occupied | ✅ |
| Bookings with no space allocated | Bookings list | confirmed, no room assigned | ⚠ the filter reads the older room link and can misread on migrated properties |
| **Occupancy Status** | | | |
| Under Notice chip | Rooms list | under notice | ✅ |
| — already replaced | Bookings list | confirmed, against beds under notice | ⚠ reachable only where the booking is linked to the specific bed |
| — past their date | Tenants list | move-out date already passed | ✅ |
| Occupancy Rate | nothing: a percentage has no records | | — |
| **Vacant Room Status** | | | |
| Any aging bar (0–7 · 8–15 · 16–30 · 31+) | Rooms list | vacant that long | ❌ fact-not-recorded |
| Never rented bar | Rooms list | never occupied | ❌ filter to add, over history that already exists |
| Unknown bar | Rooms list | let before, age unknown | ❌ filter to add, over history that already exists |
| **Upcoming Vacancy** | | | |
| Any bar | Rooms list | vacating in that window | ✅ |
| **Agreements ending soon** | | | |
| 30 / 60 / 90 band | Tenants list | agreement ending in that window | ⚠ no 60-day filter exists, and the family silently excludes short-term tenants |
| **Occupancy Trend** | | | |
| Any bar | moves the screen's window to that month; opens no list | | — |
| **Losing money** | | | |
| Group row | Rooms list | that sharing type, vacant only | ⚠ same booked mismatch as Vacant |
| Days to Fill | nothing: an average of past gaps has no current list | | — |
| **Property Wise Occupancy** | | | |
| Property row | Rooms list | that property, same slice | ✅ |

The aging-bar ❌s are a class the siblings never had: they wait on **a fact the system does not record**, when a room became empty. History to start keeping, not a filter to add, and weeks apart in effort. The never-rented and unknown ❌s are ordinary filters over history that already exists. Either the emptied date becomes something the rooms list can hold and filter on, or the vacancy card ships without drills and says so on its face; a card whose bars promise a tap that does nothing is worse than one that promises nothing.

### The gaps, plainly

⚠ **The rooms list and this screen disagree about a booked bed, in both directions.** This screen counts a booked-empty bed as vacant; the rooms list treats it as taken, so a Vacant drill comes back short. And this screen counts confirmed arrivals only; the rooms list counts any booking record. The requirement is one meaning of each word across the app. Until then the booked layer opens the bookings list, and the Vacant drill says on arrival that it under-reports.

The agreements filter family needs its 60-day band built (or the card ships 30 / 90 only), and its short-term-tenant exclusion lifted. The no-space booking filter needs to read whichever room link the property actually uses.

### What the destination says when you arrive

Better than the collections list, still not enough. The tapped tile stays visibly selected, so the slice is legible without scrolling. But no title names the slice, filters applied from elsewhere show only as a number badge on the filter icon, and the empty state reads "No rooms found" even when a filter is on, telling a manager with 300 rooms that they have none. The suite's three requirements stand: name the slice, show the active filter on arrival, name the filter in the empty state with a way out. Shared work, still unowned.

</details>

---

## 15. Who can see this

**This screen follows the permission of the records it describes: whoever can open the rooms list can read Inventory.** No separate analytics permission, nothing to re-assign. Every present-moment number here is already visible to anyone who can open the rooms list; gating the summary harder than the records would hide a total from somebody who can count it by hand one tab away.

Someone without that permission sees the full-screen lock, word for word: **"Analytics Restricted, You don't have permission to view these analytics. Request access from your admin,"** with a **Request Access** button.

There is no partial state: nobody sees some cards and not others. **Narrowed access still adds up:** if a viewer's access covers certain properties, every number on this screen, Property Wise included, counts only those properties, and the totals still reconcile with what the drills return.

---

## 16. What each card shows when it is empty, healthy or broken

**The zeros, told apart.** An empty card here is usually good news, a distinction no sibling needs.

| Situation | What shows |
|---|---|
| Never set up: no rooms at all | The whole-screen state below. No cards render |
| Onboarding: rooms created, beds still going in | Real zeros, "—" wherever a rate would divide by nothing, one quiet line under the screen: *"Your numbers fill in as you add beds and tenants."* Occupancy Status shows its card empty state (no donut to draw), and Trend its own while history is under a month |
| Every bed disabled for rent | Real counts, "—" for the rate: zero percent would say empty when the truth is switched off on purpose |
| In-window zero: rentable beds, nobody in them | Real zeros and **0%**: a genuinely empty property is a real state, never dressed as setup |
| Good-news zero | The healthy states below |
| Not recorded | The Unknown bar on Vacant Room Status, worded as unknown, never claimed as never rented |
| Failed | "Couldn't load this" with Retry. Never a healthy message, never a zero |

### Not set up

When the property has no rooms at all: *"Your property is not set up yet. Finish setting up your property and you will see all the insight details right here."* with a **"Let's set up my property"** button. This is the most common state the screen will ever show (appendix); seven boxes each explaining their own emptiness is the wrong answer to one situation.

### Healthy

| Card | Reads |
|---|---|
| Vacant Room Status | "No vacant rooms. Your property is fully occupied." |
| Upcoming Vacancy | "Nobody is leaving soon. No tenants have given notice." |
| Agreements ending soon | "No agreements ending in the next 90 days." |
| Where is my property losing money? | "No revenue leaking. Every rentable bed is earning." |

No button on any: there is nothing to do, which is the point. The design's current copy reads a full house as a data gap ("Vacancy duration by days will appear here once rooms are empty") and must not ship.

### Empty

| Card | Reads | Button |
|---|---|---|
| Occupancy Status, rooms but no beds yet | "No beds added yet. Add beds to see how full this property is." | **Add beds** |
| Occupancy Trend, under a month of history | "No trend data yet. This chart fills in from your first full month." | none: waiting is the only action |

### Failed

Per the suite rule, per card, with Retry refetching that card alone.

---

## 17. What this screen is not

- **Not a management screen.** Nothing here edits a room, moves a tenant or changes a rent; actions live on the screens this one opens.
- **Not a duplicate of the rooms list**, which shows each room's state right now and lets you act. This screen answers what it cannot: how occupancy moved, what the gaps cost, what is coming. A present-moment-only version would be the rooms list widget, read-only, not worth a tab.
- **No maintenance-downtime number.** Disabled for Rent already is that state; a separate count would show one flag twice under two names.
- **No double-booked chip.** Real but rare (appendix); folded into over-occupied. Preventing it at booking time is a live need logged in Open items.
- **Booked is never a headline of its own.** A booking is a promise about a bed's future, always reported as a layer inside the state the bed is in today.
- **No guesswork in any forward number.** Upcoming Vacancy and Coming up use confirmed notices and arrivals only; Agreements ending soon uses agreed end dates, facts about paperwork, and says so. A maybe mixed with a certainty leaves nothing a manager can act on.
- **No market-rate comparison.** Revenue loss is priced at the property's own rents, never a benchmark.
- **No All Time setting.** Occupancy averaged since the property opened answers nothing; Now is the resting state.
- **No new list screens.** Every drill lands on a list that already exists.

---

## 18. Build guidance

Numbered; every non-obvious rule as an outcome plus a test QA can run.

1. **A period number is an average of days, not a count of everyone who passed through.** A tenant present three days contributes three days; capacity counts as it stood each day. *Test it:* a property that emptied on the 5th and refilled on the 25th reports roughly a third full for that month. If it reports full, the number is wrong, whatever it was built on.
2. **The occupancy percentage counts beds in both views, and its label says so.** The existing widget shows a bed-based rate under a unit heading; that mislabel must not be carried forward. *Test it:* switch views on a property with sharing rooms. The percentage stays the same; the words beside it change.
3. ⚠ **Every rentable thing counts exactly once, and so does its rent.** A room and its beds are one lot of space and one lot of money. *Test it:* 10 rooms of 3 beds is 30 rentable beds; fully empty, the loss is 30 beds' rent, never 40, never twice the money. The easiest error here and the hardest to catch, because a doubled total still looks plausible.
4. **Capacity zero exists** (appendix) and is excluded from rentable. *Test it:* one zero-capacity room on a full property leaves the rate at 100%.
5. **Compute each number once.** The rate appears on the tiles, the donut centre, and per property; one computation, reused.
6. **Under notice stays inside occupied.** Never subtracted, anywhere.
7. **A booking is not a notice.** *Test it:* five confirmed bookings and nobody leaving shows zero under notice.
8. **This screen agrees with the rest of the app about every bed.** One disagreement exists today, both directions of it named in section 14; until it closes, booking numbers route to the bookings list.
9. **The unit rule is a requirement, not a formula.** Whole-let flat one unit; room-let flat one per room; empty space counts as rooms. How it is derived is engineering's choice. About twenty properties currently grouped by an internal list will start counting differently; check them after the change.
10. **Per-bed detail degrades to counts on unmigrated properties**, and on most history even for migrated ones. The Unknown bar exists for exactly this.
11. **Build the migrated version first**, the approximation as fallback. Every old-structure approximation in this sheet is transitional.
12. ⚠ **Nothing records when a room became empty.** Three things wait on it: the vacancy-age buckets, Days to Fill, and the past-period revenue-loss figure. Never rented does not; it comes from absence of history and was measured on live data. The one prerequisite blocking whole cards; size it before committing any date.
13. **Never rented comes from absence of history, not a zero age.** *Test it:* a bed added yesterday and never filled appears under never rented, not under 0–7 days.
14. **A room change is not a turnover, and neither is a rent change.** Both write mid-stay records; counted naively, every internal move looks like a departure plus an arrival and Days to Fill collapses toward zero. *Test it:* a tenant moved between rooms changes no Days to Fill figure anywhere.
15. **Days to Fill is one measure**, computed once, shown per group and overall, and it looks back on every filter setting.
16. **Forward layers hide on any averaged period.** Booked and already-replaced appear on Now and Coming up only. Over-occupied is not forward: it describes the window being looked at and must stay wherever the rate can exceed 100%, because it is the only thing explaining why.
17. **The present-moment and period calculations must reconcile before change chips ship.** *Test it:* pick a property and a date, compute both; if they disagree, ship without chips rather than ship a comparison that lies.
18. **State chips and layer rows hide at zero** (most properties have none of them: appendix). Change chips do not hide at zero; an unmoved number shows a neutral chip.
19. **The two bar charts share bucket edges**: 0–7, 8–15, 16–30, 31+. *Test it:* day 30 falls in exactly one bucket on each card, and the same bucket on both.
20. **Revenue loss carries its basis on its face**; below the agreed coverage it gives way to the nudge.
21. **The zeros route per section 16's table.** *Test it:* a property with rooms but no beds shows "—" for the rate and real zeros for counts; a property with rentable beds and no tenants shows 0%.
22. **Rates and averages are not tappable**; nothing about them looks tappable.

---

## 19. Open items

1. **Nothing records when a room became empty.** The vacancy-age buckets, Days to Fill and the past-period revenue loss wait on it, drills included. The largest prerequisite on the screen; needs sizing and an owner before any date.
2. **Info icon content.** No screen in the suite has written what any info icon says. Section 3 is written to be reusable as that content. Suite-wide, unowned.
3. **The trustworthy-coverage threshold** for the revenue-loss figure. Recommendation: two thirds of vacant beds carrying a set rent.
4. **Bookings with no space allocated** lives here until the Bookings screen is designed, then is referenced, not rebuilt.
5. **The trend chart's default range** (6 months) is judgement, not evidence. Revisit once real data exists.
6. **How the Overview tile row is navigated** when wider than the card. Design.
7. **Do deleted rooms still count?** If removal only flags a room, it pads rentable and reads vacant forever. Engineering to confirm.
8. **Should a restricted manager see the rupee figures, or only counts?** The original brief raised money as a separate per-role gate and nobody answered; today it is decided by omission, which is not the same as decided.
9. **Blocking a double booking at the point it is made.** Cut as a chip; the guard belongs to the booking flow and must not vanish with it.

Four items from v1 are closed and recorded in the tracker, not here: the tab row (settled, section 2), the gating permission (settled, section 15), the Dues-versus-Collection under-notice disagreement (both described the same thing from two ends), and the Expense sheet's toggle wording (its v3 already says Bed and Unit).

---

## 20. Design file: what needs fixing

<details>
<summary><strong>Expand:</strong> 41 numbered fixes, grouped Wrong / Missing / Remove / Decide</summary>

**Read this first:** every number in the file is placeholder. The fixes are structural, not data.

### Wrong, needs correcting

1. Vacant chip: green when rising. Invert: red up, green down.
2. Under Notice chip: green when rising. Invert.
3. Filter chips row: **FlexiPe** and **Report** are not time filters. Remove.
4. Filter chips: replace Today · This Week · This Month · Last Month with **Now · This Month · Last Month · Current FY · Custom · Coming up**, Now selected; Coming up gets a date picker defaulting 30 days ahead.
5. Default view: the file shows Unit View selected; the default is **Bed View**.
6. Toggle labels: both toggles read **Bed View / Unit View** (in-card: Beds / Units). The file draws them in opposite states; draw them matching, and the set-aside state separately.
7. Occupancy Trend x-axis: the fifth column, "Move-out", is a leftover clipped bar where a month belongs. Remove it; a six-month default then needs two more columns.
8. Bucket labels: "30 Days +" against "31 Days +" on the two bar charts; both become **31+ days**.
9. The two bar charts need subtitles carrying their direction: "how long empty rooms have been waiting" against "when notice-given tenants leave".
10. "No. of lease" becomes **"No. of agreements"**.
11. Property Wise sorted arbitrarily; sort highest to lowest by vacant space (section 13).
12. Empty-state copy is stale on every card: previous-draft titles ("Occupancy (Live)", "How long are my rooms vacant", "Rooms to be vacated soon", one for a card only this sheet restored) and a truncated Complaints-module body line. Rewrite per section 16.
13. Tab row: redraw to the settled suite row (section 2); Issues and the floating Leads tab go.

### Missing, needs drawing

14. Semi-Vacant slice in the Unit View donut.
15. Never-rented and Unknown bars on Vacant Room Status.
16. The revenue-loss total at the top of the losing-money card, with its basis line.
17. Agreements ending soon: a redesigned card is already staged beside the phone (horizontal bars, 90/60/30) and needs three things before it goes in: the axis reads "No. of lease" and becomes "No. of agreements"; the subtitle typo "agreements endings soon" loses its s; and the **Notify Tenant** button comes off. This screen diagnoses; the tap opens the tenants list, where notifying already lives.
18. Healthy states for the four cards in section 16, and the whole-screen not-set-up state.
19. The Under Notice chip's second line: "N past their date · N already replaced".
20. The View all sheet: only the link exists today.
21. Change-chip states: unfinished-period marker, the "vs this day last month" label on Now, the no-chip state on Coming up, the neutral grey chip.
22. Loading skeletons, failed states, and the Restricted lock.
23. In-progress bar treatment on the trend chart.
24. Bed View variants of every card: tile labels, the "Total Beds" header, donut slices are all drawn in Unit language only.
25. Coming up, entirely undrawn: chip, date picker, 30-day default, each card's projected reading, tiles without chips.
26. The Vacant legend row's two-layer split, both views: "Vacant 38: 17 booked, 21 available".
27. Overview Snapshot header: the hardcoded window comes out of the title; the active window shows beside it.
28. The over-100% sentence on Occupancy Status, stating over-occupancy and vacancy together.
29. The per-card insight line, both states: plain text positive, button negative.
30. Days to Fill with its open tail, not a bare average.
31. The Occupancy Status header carrying both totals, total and rentable, so the legend and the percentage reconcile.

### Leftover content, needs removing

32. Two of the four Occupancy Status chips: "Overbooked occupancy" (cut) and "Booked Beds" (lives in the Vacant legend split). Under Notice and Over-occupied remain.
33. Hidden duplicate delta chips stacked under three Overview tiles, in-screen and in the orphan copy.
34. The orphan tile row parked outside the phone frame, with its own hidden chips.
35. The stale single-row card ("1 BHK") parked outside the phone, disagreeing with the live card's first row.
36. Hidden "Paid by / Paid to" buttons inside Property Wise: leftover expense chrome; the card's internal name still says Expense.
37. Stray artwork parked outside the frame edge.

### Needs deciding, then drawing

38. The Overview tile row and the chips row both overflow their card with no scroll affordance; decide the treatment.
39. Info icons sit on three of five Overview tiles; all five or none.
40. No card has both an info icon and a chevron; cards that do both need both.
41. The losing-money rows and Property Wise rows carry no chevron, though every one opens a list.

</details>

---

## Where the measured figures came from

<details>
<summary><strong>Expand:</strong> 21 production figures and the decision each one settled</summary>

Measured on live production data, 2026-08-07. Two figures were not measured and are stated as judgement: the coverage threshold and the trend default range; both sit in Open items.

| Measured | Result | What it decided |
|---|---|---|
| Properties on the new structure | 787 of 73,188 real properties (1.08%); 6.11% of tenants; rising every cohort; full migration planned | Bed View defaults on, designed as the exact path; old structure is the approximation |
| Properties where every room is single | 29.5% of active | The toggle hides where both views are identical |
| Vacant units never occupied | 87–90% of vacant inventory | Never rented gets its own bucket; the four aging buckets alone describe under a tenth of vacancy |
| Vacant space let before but with no recorded emptying date | ~36% of the never-rented count | The Unknown bar, hidden at zero as migration completes |
| Room type field | free text, 60+ spellings, 85% plainly "room" | Losing-money card groups by sharing type, not flat type |
| Rooms at sharing 1 / 2 / 3 | 89% | Sharing groups capped at Single / Double / Triple / 4+ |
| Rent set on vacant beds | 25% (old structure) · 42% (new) | Revenue loss carries its basis on its face; below threshold, the nudge |
| Rent set on occupied beds | 43–54% | Confirms sparseness is not caused by vacancy itself |
| Double-booked beds, where detectable | 5, of 36,169 beds carrying real identity | Double-booked chip killed, folded into over-occupied |
| Properties with an over-occupied room | 1,061, or 4.16% of active | Chip kept |
| Under notice with a replacement booking | 328 of 4,550 (7.2%) | A second line on Under Notice, not a chip |
| Under notice, past their date | 428 tenants; 46% over a month past | Second line, not a tile; the number exists partly because nothing has ever shown it |
| Semi-vacant rooms | 56.8% of active properties have one; 11.5% of empty capacity | Donut slice and View-all row, not a tile |
| Disabled for rent | 4.5% of active properties; 79.5% of those have exactly one | Legend entry and View-all row, shown only when present |
| Properties with any confirmed booking | 3.4% of active | The booked layer hides at zero |
| Confirmed bookings with no space assigned | 603 of 3,676 (16.4%) | View-all row, reserved against the future Bookings screen |
| Multi-property owners | 11–15% of owners; 33–43% of active properties | Property Wise kept, hidden for single-property scope |
| Properties with the account link missing | ~29% of active | Property Wise must not drop unlinked properties |
| Rooms declaring capacity zero | 2,550 | Excluded from rentable; guarded division |
| Real properties with no rooms at all | 34,087 (46.6%) | The whole-screen not-set-up state |
| Flat-type labelled rooms | under 2 in 10 | Sharing type is the axis a PG owner already prices by |

</details>
