---
title: DA-08 Inventory — Handoff Sheet
date: 2026-08-07
tags: [rentok, inventory, occupancy, detailed-analytics, handoff]
status: v1 · developer handoff · four audit rounds applied, shipped 2026-08-07
owner: Sanchay
---

# Inventory — Handoff Sheet

Everything on the Inventory analytics screen: what each number means, what the Bed / Unit toggle does to it, what window it covers, what happens when it is tapped, and what it shows when there is nothing to show.

---

## What is in here

| | Section | For |
|---|---|---|
| **1** | Build status | everyone |
| **2** | Where this lives | everyone |
| **3** | What every number counts | backend + copy |
| **4** | How the screen behaves | backend + design |
| **5** | Overview Snapshot | backend + design |
| **6** | View all sheet | backend + design |
| **7** | Occupancy Status | backend + design |
| **8** | Vacant Room Status | backend + design |
| **9** | Upcoming Vacancy | backend + design |
| **10** | Agreements ending soon | backend + design |
| **11** | Occupancy Trend | backend + design |
| **12** | Where is my property losing money? | backend + design |
| **13** | Property Wise Occupancy | backend + design |
| **14** | What each number opens | backend |
| **15** | Who can see this | backend |
| **16** | What each card shows when it is empty, healthy or broken | design + copy |
| **17** | What this screen is not | everyone |
| **18** | Build guidance | backend |
| **19** | Open items | everyone |
| **20** | Design file: what needs fixing | design |
| | Where the measured figures came from | everyone |

**Building the backend?** Read everything except section 16 and section 20. Start with Build guidance; it holds the traps.

**Working on the design?** Sections 4 to 13, section 16, and section 20, which collects every fix in one list. Section 3 too if you are writing the info-icon content — it is the plain wording for every term on the screen.

**Just need the decisions?** Sections 1, 17 and 19, plus the measured figures at the end.

---

## 1. Build status

The screen is **not built**. The backend serves one empty placeholder block for Inventory that returns no numbers. Nothing here is broken; it is unwritten.

Three things that already exist matter to the build:

- A **present-moment occupancy calculation** already powers the occupancy widget on the homepage and the rooms list. Its counting rules are the starting point for this screen's present-moment numbers, with the corrections in Build guidance.
- Something already produces an occupancy figure for a past period, and it does not answer the question this screen asks. A tenant who stayed three days of a month currently weighs the same as one who stayed the whole month, so a property that stood half empty can report as full. Section 4 says what a period number has to mean here; treat that as the requirement and check anything existing against it before building on it.
- The **rooms list screen** this screen drills into is built and live.

The company is migrating every property to the **new property structure**, where each bed is a real record with its own identity. Today about 1 in 100 properties is on it, and the share grows with every new cohort. Several numbers on this screen are exact on the new structure and approximate on the old; each such number says so where it is defined. Design for where this is going, not for today's split.

---

## 2. Where this lives

Manager app, property header, **Inventory** tab.

Unlike Financial, Inventory has no sub-tabs. The tab is the screen, and it is the only tab in the suite that carries a view toggle instead.

⚠ **The tab row in the design and the tab row the system serves do not match, and this is not an Inventory problem — it affects every screen in the suite.**

| | Tabs, in order |
|---|---|
| What the system serves | Financial *(Dues · Collection · Expense)* · People *(Tenants · Old Tenants · Bookings)* · **Inventory** · Complaints |
| What the design draws | Financial · **Inventory** · Tenant · Issues · Leads |

Four differences: Inventory is second in the design and third in the system; People is flattened, so Old Tenants and Bookings have no tab; Complaints is renamed Issues; and **Leads is a tab with nothing behind it** — it exists in neither the system's navigation nor the set of screens being documented. Someone has to reconcile this once, for all eight screens, before any of them ship.

---

## 3. What every number counts

These definitions are shared by every card. They are the vocabulary of the screen, and several of them exist nowhere else in the product yet — this sheet is where they get fixed.

### Bed

The smallest rentable space — one person's place to sleep.

- On a **migrated property** (new structure), a bed is a real record: it can be named, priced, and tracked over time.
- On a **property not yet migrated**, a bed is arithmetic: a room set to 3-sharing has three beds. They have no individual identity, no own rent, and no history. Counts still work; per-bed detail does not exist.

Bed View must work on both kinds of property. Where a per-bed detail is impossible on an unmigrated property, the card shows the count and omits the detail rather than guessing.

### Unit

**A unit is whatever is let as one lettable space.**

- A sharing room is one unit.
- A studio or 1RK is one unit.
- A flat is **one unit** only when a single tenancy holds **all** of its rooms.
- A flat is **one unit per room** the moment it does not — including when one bedroom is let and the rest stand empty. If the tenant were taking the whole flat, they would be recorded against the whole flat; holding one room is itself the answer.
- **Empty space counts as rooms.** An empty flat could be sold either way and nobody has decided yet, so counting it room-wise is the deliberate default until someone challenges it.

⚠ Two consequences to state, not hide.

**A flat let whole that becomes empty splits into as many units as it has rooms.** Both readings are true, but it means totals can move for reasons other than people arriving and leaving, and the trend chart has to say so rather than let someone discover it.

**Two properties with identical inventory must produce identical unit counts.** Today they can differ, and nothing on screen explains why. The rule above is what must be true; how it is arrived at is engineering's call.

### Occupied

A bed with a living tenant in it. A tenant under notice **still counts as occupied** — they live there today. Under notice is shown separately as an at-risk layer, never subtracted from occupied.

In Unit View, an occupied unit is a unit with **all** its beds occupied.

### Semi-vacant *(Unit View only)*

A unit with some beds occupied and some free. This is where a manager can still sell a bed without anything else changing.

Only exists in Unit View — a bed is full or empty, never partly. And only where the property has at least one unit holding more than one bed; for a property of singles the word is meaningless and the slice never appears.

### Vacant

Rentable space with **nobody living in it**.

- **Bed View:** a rentable bed nobody is in.
- **Unit View:** a rentable unit with no beds occupied. Partly-filled units are semi-vacant, not vacant.

**A bed with someone booked into it is still vacant**, and this is deliberate. A booking is a promise, not a person — until they actually move in, the bed is empty and earning nothing. Counting it as filled would let a property report itself full on the strength of arrivals that have not happened.

Vacant therefore splits into two layers, and both are shown:

- **Booked** — someone is due to arrive.
- **Available** — nothing lined up. **This is what a manager actually has to sell**, and it is the number that answers "what should I be working on today".

### Booked

A **confirmed** booking on a bed where the person has not moved in yet.

- **Pending requests are not booked.** They never appear inside occupancy; where they need showing, they are named as requests awaiting approval.
- **Rejected and cancelled bookings are not booked.** They are dead and excluded from every count.
- On a property set to accept bookings automatically, a booking with no separate confirmation step **is** confirmed.

**Booked is never a total of its own — it is always a layer inside something else**, because a booking describes a bed's future, not its present state. **The layer follows the bed, never the room or unit around it:**

- A booking on an **empty** bed is a layer inside **Vacant**. That bed is empty today. This holds whether the bed sits in a wholly empty unit or a partly-filled one — a booked bed inside a semi-vacant unit still counts as booked, and the unit stays semi-vacant.
- A booking behind a tenant **under notice** is a layer inside **Occupied**, reported as the "already replaced" line. That bed has someone in it today.

Counting booked as its own slice would take beds out of the two states that describe what is true right now, and would count the same bed twice wherever both apply.

### Disabled for rent

A bed or unit deliberately switched off from renting. **This is also the maintenance state** — a room being repaired, a room the owner keeps, a room used for storage are all the same flag today. One name on screen: Disabled for Rent, with its explanation naming maintenance as the common reason.

Disabled space is excluded from rentable and from the occupancy rate. It appears in the donut legend and the View all sheet only when the property has any.

### Rentable

Total beds (or units) minus disabled ones, and minus any room that declares no capacity at all. The denominator of the occupancy rate, and the most you could ever have filled.

⚠ **When every bed is disabled, rentable is zero and there is no occupancy rate.** Show "—", never 0%. Zero percent says the property is empty; the truth is that nothing is being rented out on purpose.

### Under notice

A living tenant with a move-out date set. The bed is occupied today and free soon.

Two layers ride on this number and are shown with it rather than as separate top-level numbers:

- **Past their date:** the move-out date has passed and the tenant still shows as living. Either the person left and nobody updated the record, or they genuinely overstayed. Both are worth a manager's attention.
- **Already replaced:** a confirmed booking already exists for that bed. The gap is handled.

### Never rented

A rentable bed or unit that has **never had a tenant** — added to the system and never filled. This is not slow turnover; it has no "empty since" date, because it has no history at all.

This is the single largest fact about vacancy on the platform: measured in production, roughly **9 in 10 vacant units have never been occupied**. It gets its own bucket wherever vacancy age is shown, and its own row in the View all sheet.

On migrated properties, a portion of "never rented" is really **"cannot tell"** — the space has been let before, but nothing records when it last emptied, so it cannot be aged. Those are shown as their own bar, worded honestly as *unknown* rather than claimed as never rented. It shrinks as properties migrate and **hides at zero** rather than sitting there as a permanent empty bar.

### Over-occupied

More living tenants in a room than its capacity. Real and recurring — measured at about 1 in 25 active properties. This is the only state that can push occupancy above 100%, and the arithmetic is left uncapped: a manager seeing 104% is seeing the truth, and the over-occupied chip beside it says why.

### Occupancy rate

**Occupied beds divided by rentable beds. Always beds, in both views.**

A unit-based rate would call a 4-sharing room with one tenant "fully occupied", which is not how this business earns. The rate stays bed-based when the toggle is on Unit View, and the label says so: "Occupancy Rate (by beds)". The toggle re-slices everything else; it never changes this number. Today's live code mislabels a bed-based rate as unit-based on the Units tab — that mistake must not be carried forward.

### One word to be careful with

⚠ **"Under notice" is the phrase on this screen** — the same tenant state is called "Under Eviction" on Collection and "Under Notice" on Dues. Inventory uses under notice, matching Dues. No third variant.

---

## 4. How the screen behaves

### The time filter

One filter at the top: **Now (default) · This Month · Last Month · Current FY · Custom · Coming up.**

⚠ **Two deliberate differences from every other screen in the suite.**

**There is no All Time.** On the money screens "no date limit" means everything ever, which is a useful number. Occupancy averaged since a property opened is not a number anyone asks for. The natural resting state here is right now, and that is what **Now** is.

**The word is Now, not Live.** The other screens already use "live" to mean a section no filter can change. Here the same idea is a *setting* of the filter, so reusing the word would give it two opposite meanings on adjacent tabs.

**Coming up** is the one forward setting in the suite. It asks for a future date and answers *what will this property look like then* — today's occupied tenants, minus everyone with a confirmed move-out before that date, plus everyone with a confirmed booking starting before it. Confirmed only: no leads, no pending requests, no guesses. Default horizon 30 days.

**What a period means:** This Month shows the **average across the days of the month**. "78% last month" means that on an average day last month, 78% of rentable beds were filled — not a snapshot of the last day. This is how occupancy is reported everywhere in this industry, and it is the only reading that is fair to a property that emptied and refilled mid-month.

Cards with their own date dropdown follow the three suite rules: same options as the top filter; changing the top filter pulls every card back into line; a manager can set one card aside deliberately but can never end up on a mixed screen without meaning to.

### What every number does on every filter setting

| Number | Now | This Month · Last Month · Current FY · Custom | Coming up |
|---|---|---|---|
| Overview tiles | As they stand | Average over the period | Projected at the chosen date |
| View all sheet | As they stand | Average; booked and already-replaced layers hide | Projected |
| Occupancy Status | As they stand | Average. The booked and already-replaced layers hide | Projected |
| Vacant Room Status | As of today | As of today | As of today |
| Upcoming Vacancy | From today onwards | From today onwards | From today onwards |
| Agreements ending soon | From today onwards | From today onwards | From today onwards |
| Occupancy Trend | Its own range | Its own range | Its own range |
| Losing money — occupancy and revenue loss | Follows | Follows | Projected loss at that date |
| Losing money — **Days to Fill** | Looks back | Looks back | Looks back |
| Property Wise Occupancy | Follows | Follows | Projected |

Four things fall out of this grid that the rules above do not make obvious.

**Four cards never follow the filter, and each says so on its face** — in the card header, beside the title, not buried in an info icon.

| Card | Says |
|---|---|
| Vacant Room Status | "As of today" |
| Upcoming Vacancy | "From today onwards" |
| Agreements ending soon | "From today onwards" |
| Occupancy Trend | its own range control, which already names the window |

**Days to Fill is exempt inside a card that is not.** It measures past turnovers, so it has nothing to compute from the present or the future. Its card follows the filter around it; this one figure looks back regardless, and says so.

**Coming up has no change chip.** There is nothing to compare a projection against — the same date last month was not a projection, it was a fact. The tiles show the projected figure with no chip, the same way they would on a setting with no previous period.

**Coming up counts only what is already agreed.** Its whole value is that a manager can trust it: every arrival has a confirmed booking and every departure a confirmed notice. The moment a maybe enters, the number stops being a plan and becomes an opinion.

**Three rows in the View all sheet describe the past and never move** — Days to Fill, never rented, and past their date. They look back on every setting, not only on Coming up.

### The view toggle: Bed View / Unit View

One toggle at the top of the screen: **Bed View (default) · Unit View.** Bed View is the default because the platform is migrating to bed-level records; the toggle is where the product is going.

The toggle changes **the unit of counting**, not the facts. The same property, the same people — counted per bed or per lettable space. Every card's section below states what it shows on each side. Where the two sides would show the same thing (a property of all singles), **the toggle is hidden** — about 3 in 10 properties. A control that does nothing teaches people to ignore controls.

**Cards may carry their own copy of the toggle.** It behaves exactly like a card's own date dropdown: same two options, flipping the top toggle pulls every card back into line, and a manager can deliberately set one card aside to compare views. The Occupancy Status card carries one in the design; any card-level toggle follows this rule.

Three cards do not follow the toggle at all, because their subject has no bed-level version: Vacant Room Status and Upcoming Vacancy are counted per room (a bed's own empty-since date does not exist on unmigrated properties and is missing from most history even on migrated ones), and Agreements ending soon counts agreements, which belong to tenancies, not beds. Each says "by rooms" or "agreements" on its face and ignores the toggle.

### Change chips

⚠ **This screen has no single good direction — polarity is per tile, not per screen.** Expense could say "up is bad" once. Here:

| Number | Rising is |
|---|---|
| Rentable Beds / Units | Neutral — grey chip |
| Occupied | Good — green up, red down |
| Occupancy Rate | Good — green up, red down |
| Vacant | **Bad — red up, green down** |
| Under Notice | **Bad — red up, green down** |

The design currently shows Vacant and Under Notice green when rising. Both are inverted and must be corrected.

⚠ **This is component work, not a table.** The shared change-chip component defaults to one direction for a whole screen, and Dues has already been corrected once for it. Inventory needs three behaviours in a single row — good-up, bad-up, and a **neutral grey chip** that no other screen defines.

**On Now**, a chip compares against **the same point one month ago**, and says so: "▲ 4% vs this day last month." Unlike All Time on the money screens, a present-moment number does have a real predecessor, and the manager landing on the default deserves context. **On Coming up there is no chip at all** — a projection has nothing to be compared with. On period settings, chips compare like-for-like: a finished month against the previous finished month; an unfinished period against the **same elapsed days** of the previous period, marked as unfinished. On the money screens "same point" means a running total to the same day; here, where a period is an average, it means the average over the same number of days — otherwise a fortnight of this month is compared against a whole month of last.

### Loading, failure, empty

The suite rules apply unchanged: every card loads, fails and retries on its own; skeletons match the card they become; a failed card never shows a number, keeps its heading, shrinks to "Couldn't load this" and a Retry that refetches only that card; empty and failed look different; when every card fails, one message — "Couldn't load this page. Check your connection."

What is different on this screen is that **an empty card is often good news** — see section 16 for the three-way split between empty, healthy, and not-set-up.

### Sorting

Every property-wise and every breakdown list sorts **highest to lowest**. For the losing-money card specifically, sorted by revenue loss, highest first.

### Entry point

A manager reaches this screen from the Inventory tab in the property header. Other screens drawn around it in the design file are reference context, not part of it.

---

## 5. Overview Snapshot

Five tiles in a row wider than the card. How that row is navigated is a design decision still open — see section 20. The header reads **"Overview Snapshot"** and carries the active window beside it — "Now", "Last Month", "By 15 Sep". The design's hardcoded "(Now)" in the title comes out; the window is what the filter says, not part of the card's name. (The older draft carried "(Live)" in the same place, for the same reason.)

| Tile | Bed View | Unit View |
|---|---|---|
| **Rentable Units** | Rentable beds | Rentable units |
| **Occupied Units** | Beds with a living tenant | Units with all beds occupied |
| **Occupancy Rate** | Occupied ÷ rentable beds | Same number — always by beds, labelled so |
| **Vacant Units** | Rentable beds nobody is living in | Units nobody is living in |
| **Under Notice Units** | Beds whose tenant gave notice | Units with at least one such tenant |

- Tile labels switch with the toggle: "Rentable Beds" in Bed View, "Rentable Units" in Unit View. A label reading Units above a bed count is a mislabel.
- Each tile carries a change chip per the polarity table in section 4.
- **View all** opens the sheet in section 6.

⚠ **All five of these tiles appear again on the Occupancy Status card directly below** — in its header, legend, rate and chips. That repetition is deliberate: the tiles summarise, the donut explains the same split visually and adds what the tiles have no room for. **They are one number each, computed once and shown twice.** They can never disagree. The design file currently shows 25% on the tile and 45% in the donut, which is placeholder data, but two occupancy rates on one screen is the single fastest way to lose a manager's trust in every other number on it.

---

## 6. View all sheet

Everything the tile row knows, grouped as three plain questions. Indented rows are layers inside the row above, never separate totals. Booked and available split Vacant exactly; the other layers are overlapping views of their parent and are not meant to sum. Rows follow the toggle exactly as the tiles do, except Semi-vacant, which exists only in Unit View. Every row that can open a list does — see section 14.

**What have I got**

| Row | Meaning |
|---|---|
| Total | Everything, before anything is subtracted. Beds or units, following the toggle like every row here |
| Disabled for rent | Switched off from renting (includes maintenance), plus any room declaring no capacity. Shown only when more than zero |
| Rentable | Total minus disabled |
| Occupied | As defined |
| Semi-vacant | Unit View only; hidden for all-singles properties |
| Vacant | As defined |
| — of which booked | Someone is due to arrive |
| — of which available | Nothing lined up. What there is to sell |
| — of which never rented | Never had a tenant. Not slow turnover — space that has never earned |
| Over-occupied | Rooms holding more living tenants than capacity. The reason a rate can read above 100% |

**Who's moving**

| Row | Meaning |
|---|---|
| Under notice | Living tenants with a move-out date |
| — past their date | Move-out date passed, still recorded as living |
| — already replaced | A confirmed arrival is lined up for the same bed |
| Bookings with no space allocated | Somebody has been confirmed and given nowhere to sleep. Counted here because it is the one booking figure that is not a layer under Vacant — there is no bed to layer it on. ⚠ Belongs to Inventory until the Bookings screen exists, which must then reference it rather than rebuild it |

**How fast**

| Row | Meaning |
|---|---|
| Days to Fill | Across beds refilled recently: days between one tenancy ending and the next beginning. The same measure the losing-money card shows per group; this is the overall figure |

---

## 7. Occupancy Status

The donut card. Centre: **Occupancy Rate — always by beds, in both views, labelled so.**

**Bed View slices:** Occupied · Vacant · Disabled for Rent (last one only when present).
**Unit View slices:** Occupied · Semi-Vacant · Vacant · Disabled for Rent (last one only when present). Semi-vacant is the reason Unit View needs its own slice set: a partly-filled unit is neither full nor empty, and forcing it into either would misstate the property.

**Vacant carries its two layers with it**, wherever it appears — as a split in the legend, not a slice of its own: *"Vacant 38 — 17 booked, 21 available"*. Available is the one a manager acts on, so it is never left to be worked out by subtraction.

**The legend adds up to Total, and the rate divides by Rentable.** Occupied + Semi-Vacant + Vacant = rentable; adding Disabled for Rent — where rooms declaring no capacity are also counted — brings it to total. The card carries both numbers in its header — "Total Beds: 326 · 318 rentable" — because a legend summing to one number beside a percentage computed from another is the kind of thing a manager checks once, cannot reconcile, and never trusts again.

The header count switches with the toggle: "Total Beds: N" in Bed View, "Total Units: N" in Unit View.

**Chips row** (each shown only when non-zero). Booked is not a chip — its number lives in the Vacant legend split, once:

| Chip | Meaning |
|---|---|
| Under Notice | As defined. Carries its two layers as a second line when either is above zero — "4 under notice · 2 already replaced · 1 past their date" |
| Over-occupied | Rooms with more living tenants than capacity. This is why the rate can read above 100% |

Where the rate does exceed 100%, the card states both halves in one sentence rather than leaving the manager to work it out: *"104% — 3 rooms are over-occupied while 5 beds elsewhere sit empty."* Over-occupancy and vacancy almost always coexist, and seeing only the first reads as a data error.

There is **no separate double-booked chip.** Two confirmed bookings on one bed is a real conflict, but it was measured at five cases across the 36,169 beds where the structure makes it detectable at all. It folds into over-occupied rather than owning screen space.

**Money tiles** (bottom of the card):

| Tile | Meaning |
|---|---|
| Avg. Rent per Occupied Bed / Unit | What one filled bed — or one filled unit — earns on average. The label follows the toggle like every other |
| Avg. Revenue per Available Bed / Unit | What every rentable bed — or unit — earns on average, filled or not. Lower than the tile beside it whenever anything is vacant, and the gap between the two is what vacancy is costing. On an over-occupied property it can read higher, which is the same signal the over-100% sentence explains |

**AI insight bar** (bottom of the card): one sentence about the biggest thing on this card. A positive insight is plain text, no link. A negative one — revenue being lost, a dip — carries a button opening the AI chat with the question pre-filled: "How can we improve this?" This ships on Inventory only; no other analytics screen has an AI destination behind it yet, and this pattern must not be copied to them until one exists.

---

## 8. Vacant Room Status

How long currently-empty rooms have been empty. **Counted per room, always — this card ignores the view toggle** and says "by rooms" on its face. Always as of today; the time filter does not move it.

**A room is empty here when nobody is living in it** — the same test as Vacant everywhere else on this screen, including rooms with somebody booked to arrive. A room booked for next month has still stood empty for sixty days and earned nothing in them. A room with even one tenant in it never appears on this card, however many free beds it has. Those free beds are still counted as vacant on the tiles — they simply have no room-level emptiness to measure.

Six bars:

| Bar | Meaning |
|---|---|
| 0–7 days | Empty a week or less |
| 8–15 days | Empty between one and two weeks |
| 16–30 days | Empty between two weeks and a month |
| 31+ days | Empty over a month — the turnaround problem |
| **Never rented** | Added to the system, never filled. No "empty since" date exists |
| **Unknown** | The room has been let before, but nothing records when it last became empty, so its age cannot be worked out. Shrinks as properties migrate; hides at zero |

⚠ **"Vacant" on this card is not the same number as "Vacant" on the tiles.** They apply the same test — nobody living there — but count different things: the tiles count beds or units and follow the toggle, this card counts rooms and never does. The card carries "by rooms" on its face precisely so nobody reads the two as one figure.

⚠ The design draws only the four aging buckets. Measured in production, the four together describe **under a tenth of vacant inventory** — never-rented is nine tenths. Without its bar this card silently reports the small minority of the problem while looking complete.

⚠ **Nothing in the product currently records when a room became empty**, so every bar on this card depends on that being built first, and none of them can open a list until it is. This card is the one place on the screen where a prerequisite blocks the whole thing rather than one number — see Build guidance.

Red marks the largest of the **four aging bars** — the design's existing rule, scoped deliberately. Never rented and Unknown are left out of it: never rented is the biggest bar on almost every property, so colouring it red would make the marker constant and stop it doing its job, which is pointing at the turnaround problem.

---

## 9. Upcoming Vacancy

Who is confirmed to be leaving, how soon. Built **only from tenants who have actually given notice** — no predictions. Counted per room; ignores the view toggle; always forward-looking.

Four bars: leaving within 0–7 · 8–15 · 16–30 · 31+ days.

⚠ Both this card and Vacant Room Status are bar charts over day-buckets, adjacent on the screen. One looks backward at empty rooms, one looks forward at occupied ones. The subtitles must carry the direction plainly: "how long empty rooms have been waiting" versus "when notice-given tenants leave". The bucket labels must also agree — the design currently draws "30 Days +" on one card and "31 Days +" on the other, so day 30 falls in two different buckets depending on which chart you read. Both cards use **31+ days**.

---

## 10. Agreements ending soon

Agreements reaching their end date, in 30 / 60 / 90-day bands, with a count and a way into the list of those tenants.

This is the **earliest warning on the screen**: notice given is a decision already made, an agreement running out is a decision not yet made. Most renew — but each one is a conversation a manager should have before the date, not after.

Counts **agreements**, so it ignores the view toggle and the time filter. The axis label says "No. of agreements" — the word is agreement everywhere in this suite, never lease.

⚠ One tenancy can appear on this card and on Upcoming Vacancy at once — notice given *and* agreement ending. That is not double-counting, it is two lenses on one departure; but the two cards must agree on the person. If they leave on the 15th, the agreement card cannot claim a different date.

---

## 11. Occupancy Trend

Occupied versus vacant over time, as stacked bars per month. Exempt from the time filter; keeps its own range control: **6 / 12 / 24 months**, defaulting to 6.

- Follows the view toggle: bars count beds in Bed View, units in Unit View.
- A month in progress renders as an in-progress bar, marked per the suite's unfinished-period rule.
- History before the property joined the platform does not exist and is not invented; the chart starts where the data starts.
- Where the unit definition changed mid-history (a whole-let flat emptied and became room-wise), the total can move without anyone arriving or leaving — the card's info explanation says so.

The insight sentence follows the same AI rule as section 7.

---

## 12. Where is my property losing money?

The cost of the gaps, grouped by **sharing type** — Single · Double · Triple · 4+ sharing — not by flat type. Measured in production, more than 8 in 10 rooms carry no flat-type label at all, so a flat-type grouping would collapse to one giant row for almost every property. Sharing type is populated on essentially all rooms and is how a PG owner already thinks about pricing. Where a property genuinely has flat-type variety (studios, BHKs), those appear as their own groups alongside sharing groups.

**Headline: the total.** ⚠ Not in the design yet — the card shows four group rows and never answers its own question in one number. The total monthly revenue loss sits at the top; the rows below break it down. The total and the rows must always reconcile.

Per group row:

| Field | Meaning |
|---|---|
| Occupancy % | Occupied ÷ rentable beds in this group |
| Days to Fill | Average time a bed in this group takes to refill, from recent turnovers, **always shown with its open tail**: "avg 14 days · 2 beds still open, longest 86 days". Shows "—" with "no recent turnover" where none exists |
| Revenue loss | The rent of this group's vacant beds, per month, **with its share of the total loss**. The bar carries the share; the number beside it carries the rupees |

**Revenue loss means two different things depending on the window, and the card says which.**

- **On Now:** what is being lost *per month* right now — the monthly rent of every currently-vacant bed.
- **On a past period:** what vacancy *actually cost* — each bed's empty days in that window multiplied by its daily rent, summed. Daily rent is the monthly rent divided by the real number of days in that month, not by 30. ⚠ This needs the same missing history as the vacancy card: how many days each bed stood empty. Until that exists, the past-period figure cannot be built either — see Build guidance.

The wording changes with it: "losing ₹200K a month" on Now, "vacancy cost you ₹185K last month" on a past period, "₹190K a month at this rate" on Coming up.

**Revenue loss uses each vacant bed's own configured rent.** Where a bed has none, fall back to **what that space was last let for**, and only then to the group's average. ⚠ The middle step is unavailable for most beds today, for the same reason the vacancy-age card carries an Unknown bar. Build it so it improves on its own as the migration lands, rather than leaving out a step that will soon work.

**The basis line counts only beds with a rent actually set**, not beds priced by a fallback: "based on 34 of 120 vacant beds with rent set". The fallbacks keep the figure from being wrong; the basis line keeps the manager from over-trusting it. Below the agreed coverage the figure is replaced by the setup nudge — the threshold itself is still open, see section 19. ⚠ **Rent is sparsely configured** — measured at a quarter of empty old-structure rooms and under half of empty new-structure beds. The card must therefore carry its basis on its face: "based on N of M vacant beds with rent set". The nudge reads "Set rents on your vacant beds to see what vacancy costs you" — which is itself what fixes the coverage.

Sorted by revenue loss, highest first. Follows the time filter and the view toggle (groups re-slice; the money follows the beds).

---

## 13. Property Wise Occupancy

**Hidden entirely when only one property is in scope**, rather than showing a one-row comparison, and it comes back on its own as soon as more than one property is selected. Measured in production, a third to well over two-fifths of active properties belong to a multi-property owner; for them this is the "which building is hurting" card.

Per property: name, occupied / rentable, and a progress bar. **Sorted highest to lowest by vacant space** — vacant beds in Bed View, vacant units in Unit View — not by occupancy rate.

The rule across the suite is highest first, and the card's question is "which property is hurting". Ranking by occupancy rate satisfies the rule but puts the fullest property on top and buries the worst under a View more tap. Ranking by vacant space keeps the rule exactly as written and puts the property that needs attention first. A property with 200 empty beds outranks one with 5, which is also the order a manager would work in.

Follows the time filter (its own dropdown per the suite rules) and the view toggle. "View more" expands the list in place. **Tapping a property row opens the rooms list for that property**, the same as every other drill on this screen — it does not narrow the whole screen to that property. Apart from a trend bar, which moves the window on every analytics screen, nothing in this app re-scopes from a row — and a card that did would be an interaction a manager had to learn for one row type only.

**Properties with nothing rentable still appear, at zero.** A property missing from a comparison reads as an error; a property showing zero reads as a fact.

⚠ The account grouping must not silently drop properties whose account link is missing — measured at 3 in 10 active properties. However the grouping is computed, every property the manager can open belongs to some row.

---

## 14. What each number opens

### The rules

- **A drill filters a list. It never re-scopes the screen** — with exactly one exception, named below: a trend bar. That exception exists on the money screens too, and it stays the only one.
- **A number opens the list of the kind of record it counts.** A number about *space* — vacant, occupied, semi-vacant, disabled, under notice, over-occupied — opens the rooms list, because the thing being examined is the room. A number about a *person* — past their date, agreements ending — opens the tenants list. A number about a *booking* opens the bookings list.
- The destination opens on the **same view**, and the back control **names where the person came from** ("Inventory", not a bare arrow).
- **Records behind a number must add back to the number tapped** — on Now.
- ⚠ **In Bed View, a bed count opens the rooms that hold those beds.** There is no bed list in the product. The destination says so — "12 rooms holding 18 vacant beds" — rather than letting a manager count rows and find a different number.
- ⚠ **A period average has no list behind it.** "78% last month" is an average over thirty days; there is no set of beds that *is* that number. So a drill from any period number opens **today's** matching list and says so on arrival: "showing these rooms as they are now". Pretending otherwise would hand a manager a list that does not add up to the figure they tapped, which is precisely the trust gap this screen exists to close.
- Every row that opens something has to look like it does.

### The table

Checked one by one against the filters the rooms, tenants and bookings lists actually accept today.

| Number | Opens | Ready? |
|---|---|---|
| Total | Rooms list, all rooms | ✅ |
| Rentable | Rooms list, all rooms except disabled | ✅ |
| Occupied | Rooms list, fully occupied (Unit View) or has-occupied-bed (Bed View) | ✅ |
| Vacant | Rooms list, fully vacant (Unit View) or has-vacant-bed (Bed View) | ⚠ **Partial** — the list treats a booked bed as taken, so it returns **fewer** rows than the number said |
| Semi-vacant | Rooms list, semi-vacant | ✅ |
| Disabled for rent | Rooms list, disabled for rent | ✅ |
| Under notice | Rooms list, under notice | ✅ |
| — already replaced | Bookings list, confirmed, against beds under notice | ⚠ **Partial** — reachable only where the booking is linked to the specific bed |
| — past their date | Tenants list, move-out date already passed | ✅ |
| Over-occupied | Rooms list, over-occupied | ✅ |
| Occupancy Rate | Nothing. A percentage has no records — not tappable | — |
| Days to Fill | Nothing. An average of past gaps has no current list — not tappable | — |
| Losing-money group row | Rooms list, that sharing type, vacant only | ⚠ **Partial** — same booked mismatch as Vacant above |
| Property Wise row | Rooms list, that property, same slice | ✅ |
| Vacant — of which booked | Bookings list, confirmed | ✅ |
| Vacant — of which available | Rooms list, vacant, minus those with a confirmed arrival | ⚠ **Partial** — the list cannot subtract confirmed arrivals |
| Bookings with no space allocated | Bookings list, confirmed with no room | ⚠ **Partial** — the filter reads the older room link, so it can misread on migrated properties |
| **Agreements ending, 30 / 60 / 90 days** | Tenants list, agreement ending in window | ⚠ **Partial** |
| **Vacancy age buckets (0–7, 8–15, 16–30, 31+)** | Rooms list, vacant that long | ❌ **Not reachable** |
| **Never rented** | Rooms list, never occupied | ❌ **Not reachable** |
| **Unknown (vacancy age)** | Rooms list | ❌ **Not reachable** |
| Upcoming Vacancy buckets | Rooms list, vacating in window | ✅ |
| Trend bar | **Moves the screen's window to that month.** The one drill that re-scopes rather than filters, matching the other analytics screens. Does not open a list | — |

### The gaps, plainly

⚠ **Nothing in the product can list "rooms vacant for 8–15 days" or "rooms never rented".** There is no record anywhere of when a room became empty — not a field, not a filter, not a derived value. Every bar on Vacant Room Status, including the never-rented bar that covers nine tenths of vacant inventory, is a number that cannot open itself.

This is the largest build item on the screen and it is not optional. A vacancy card whose bars do nothing is a poster. Either the vacant-since date becomes something the rooms list can hold and filter on, or the card ships without drills and the sheet says so on its face rather than promising a tap that does nothing.

⚠ **The rooms list and this screen disagree about a booked bed, in both directions.**

This screen says a bed with somebody booked into it is **still vacant** — nobody is living there, it is earning nothing, and the arrival is a promise rather than a person. The rooms list treats that bed as taken. So a Vacant number opened there comes back **short**, and a manager who counts the rows finds fewer than the number promised.

Separately, this screen counts **confirmed** arrivals only, while the rooms list counts any booking record. So the *booked* layer opens the bookings list, where confirmed-only can be expressed; *available* opens the rooms list, as its table row says.

**The requirement is that the two agree** — one meaning of vacant, one meaning of booked, across the app. Until they do, the Vacant drill will under-report and must say so on arrival rather than quietly showing a shorter list.

⚠ **Agreements ending has no 60-day bucket**, only 30 and roughly 90. And the whole agreement-ending filter family silently excludes short-term tenants. The 60-day band either gets built or the card uses 30 / 90 only.

⚠ **The confirmed-booking-with-no-room filter reads the old room field**, so on migrated properties it can misclassify. It needs to read whichever link the property actually uses.

### What the destination says when you arrive

Better than the collections list, and still not good enough.

Tapping a widget on the rooms or tenants list **does** leave the tapped tile visibly selected at the top, so a manager can see which slice they are in without scrolling. That much works.

But neither screen has a title that names the slice — they have no title bar at all. Filters applied from anywhere other than the tile row show only as a **number badge on the filter icon**, naming nothing. And the empty state reads **"No rooms found"** whether or not a filter is applied, which tells a manager with 300 rooms that they have none.

The three requirements are the same as every screen in this suite: name the slice, show the active filter on arrival, and name the filter in the empty state with a way out. This is shared work across every analytics screen and still has no owner.

---

## 15. Who can see this

The screen is gated by an analytics permission. Someone without it sees the full-screen lock: **"Analytics Restricted — You don't have permission to view these analytics. Request access from your admin,"** with a Request Access button.

⚠ Which permission that is remains the suite-wide open question — the lock is specced as an analytics permission, while the existing list screens gate on their own viewing permissions, and nothing reconciles the two. Inventory adds one wrinkle: its present-moment numbers are already visible to anyone who can open the rooms list, so gating the analytics tab harder than the rooms list would hide numbers here that the same person can see one tab away. Engineering owns the reconciliation; it is logged in Open items.

There is no partial state: nobody sees some cards and not others.

**What can never happen is a total that counts space the person cannot open.** If a viewer's access is narrowed to certain properties, every number on this screen — including Property Wise — counts only those properties, and the totals still reconcile with what the drill returns.

---

## 16. What each card shows when it is empty, healthy or broken

This screen needs a three-way distinction no other screen in the suite has, because **an empty card here is usually good news.**

### Not set up — the whole screen

When the property has **no rooms at all**, no cards render. One full-screen state: *"Your property is not set up yet. Finish setting up your property and you will see all the insight details right here."* with a **"Let's set up my property"** button into property setup. Measured in production, nearly half of live properties are in this state — it is the most common thing this screen will ever show, and seven separate empty boxes each explaining their own emptiness is the wrong answer to it.

### Healthy — the card worked, and the news is good

| Card | Healthy state reads |
|---|---|
| Vacant Room Status | "No vacant rooms — your property is fully occupied." |
| Upcoming Vacancy | "Nobody is leaving soon. No tenants have given notice." |
| Agreements ending soon | "No agreements ending in the next 90 days." |
| Where is my property losing money? | "No revenue leaking — every rentable bed is earning." |

No CTA on any of these. There is nothing to do; that is the point. The current design copy — "No vacant rooms found. Vacancy duration by days will appear here once rooms are empty" — reads a full house as a data gap and must not ship.

### Empty — the card worked, and there is nothing yet

Genuinely-nothing-yet cases keep the factual tone, with a button only where there is a real next step:

| Card | Reads | Button |
|---|---|---|
| Occupancy Status, rooms created but no beds yet | "No beds added yet. Add beds to see how full this property is." | **Add beds** |
| Occupancy Trend, under a month of history | "No trend data yet. This chart fills in from your first full month." | none — waiting is the only action |
| Every other card, rooms created but no beds yet | renders with real zeros, and "—" wherever a rate would divide by nothing | none — the screen-level line covers it |

**Half-set-up properties are the awkward middle**, and they are common — rooms added, beds and tenants still going in. The numbers stay honest rather than being suppressed, with one quiet line under the screen: *"Your numbers fill in as you add beds and tenants."* A property that is genuinely 0% occupied is a real state and shows real zeros; only a property with nothing set up gets the setup screen.

### Failed

Per the suite rule: "Couldn't load this" plus Retry. Never a healthy message, never a zero.

---

## 17. What this screen is not

- **Not a management screen.** Nothing here edits a room, moves a tenant, or changes a rent. Every action lives on the screens this one opens. This screen diagnoses.
- **Not a duplicate of the rooms list.** The rooms list answers "what is the state of each room right now" and lets you act. This screen answers what the rooms list cannot: how occupancy moved, what the gaps cost, what is coming. That is why the period and forecast settings exist — a present-moment-only version of this screen would be the rooms list widget, read-only, and not worth a tab.
- **No maintenance-downtime number.** Disabled for Rent already is that state — one flag covers repair, owner use, and every other reason. A separate maintenance count would show one flag twice under two names.
- **No double-booked chip.** Five real cases among the 36,169 beds where it is detectable at all; folded into over-occupied. Preventing it when the booking is made is a separate and current need — see Open items.
- **Booked is never a headline of its own.** A booking is a promise about a bed's future, not a description of it today, so it is always reported as a layer inside the state the bed is actually in.
- **No guesswork in any forward number.** Upcoming Vacancy and the Coming up setting use confirmed notices and confirmed bookings only — no leads, no pending requests, no predicted demand. Agreements ending soon uses agreed end dates, which are facts about the paperwork rather than decisions to leave, and the card says so. The moment a maybe mixes in with a certainty, a manager can no longer act on the number, and a forecast nobody acts on is worse than no forecast.
- **No market-rate comparison.** Revenue loss is priced at the property's own configured rents, never at an external benchmark.
- **No new list screens.** Every drill lands on a list that already exists.
- **The "Ask RentOk AI" bar at the bottom of the screen is app shell**, present on other tabs, and not defined by this sheet.

---

## 18. Build guidance

**Counting**

1. **A period number is an average of days, not a count of everyone who passed through.** A tenant present for three days of a month contributes three days. Capacity counts as it stood on each day, not as it stands today.

    *Test it:* a property that emptied on the 5th and refilled on the 25th reports roughly a third full for that month. If it reports full, the number is wrong, whatever it was built on.
2. **The occupancy percentage counts beds in both views, and its label says so.** A manager who switches to Unit View and sees the same percentage under a heading that says units has been told something untrue.

    *Test it:* switch views on a property with sharing rooms. The percentage stays the same; the words beside it change.
3. ⚠ **Every rentable thing counts exactly once, and so does its rent.** A room and the beds inside it are one lot of space and one lot of money, not two.

    *Test it:* a property with 10 rooms of 3 beds has 30 rentable beds, and if all of them are empty the loss is 30 beds' worth of rent — never 40 beds, never twice the money. This is the easiest error to make here and the hardest to catch, because a doubled total still looks plausible.
4. **Capacity zero exists.** Thousands of rooms declare zero capacity. Every rate divides by rentable; guard the division and exclude zero-capacity rooms from rentable rather than letting one bad room poison a property's percentage.
5. **Compute each number once.** The occupancy rate appears on the tiles and in the donut centre, and Property Wise shows the same idea per property. One computation, reused. Two cards disagreeing on one screen is how trust dies.
6. **Under notice stays inside occupied.** Never subtract it. The donut, the tiles and the rate all count an under-notice tenant as occupying their bed.
7. ⚠ **A booking is not a notice.** Under notice means someone living there who has said they are leaving. Someone who has booked and not arrived has not given notice — they have not arrived.

    *Test it:* a property with five confirmed bookings and nobody leaving shows zero under notice.
8. **This screen's numbers must agree with the rest of the app.** A bed this screen calls vacant has to read the same on the homepage occupancy widget, the rooms list and the tenant lists. Two surfaces disagreeing about one bed is worse than either being slightly wrong.

    ⚠ One disagreement exists today: **the rooms list counts a booking more loosely than this screen does.** There, a booking record is a booking; here, only a confirmed one is. So a booked number opened against the rooms list returns more rows than the number promised. Until the two agree, booking numbers open the bookings list instead — see section 14. **One meaning of "booked" across the app is the requirement**; routing around it is temporary.

**The structures and the migration**

9. **The unit rule is a requirement, not a formula.** Whole-let flat = one unit; room-let flat = unit per room; empty = rooms. How it is derived is engineering's choice. The hardcoded property list that currently decides flat grouping does not satisfy the rule — and about twenty properties currently on that list will start counting differently, so check them after the change.
10. **Bed identity is absent on unmigrated properties and missing from most stay history even on migrated ones.** Anything per-bed must degrade to counts on those properties. The vacancy-age "unknown" bar exists for the same reason: the space has been let before, but nothing says when it last emptied, so its age cannot be worked out.
11. **The migration is coming.** Every old-structure approximation in this sheet is transitional. Build the exact version for new-structure properties and the approximation as the fallback, not the other way around.

**History and time**

12. ⚠ **Nothing records when a room became empty.** Four things need it and none can be built until it exists: vacancy age, never rented, Days to Fill, **and the past-period revenue-loss figure**, which multiplies each bed's empty days by its daily rent. This is the one prerequisite that blocks whole cards rather than single numbers — size it before committing to a date.
13. **Never rented must come from absence of history, not a zero age.** A bed with no history is not "empty 0 days" and not "empty 31+ days" — it is its own state.
14. ⚠ **A room change is not a turnover.** Moving a tenant between rooms, or changing their rent, writes a new stay record mid-stay. Counted naively, every internal move looks like someone leaving and someone arriving, and Days to Fill collapses toward zero. Only real departures count.
15. **Days to Fill and the View-all Days to Fill are one measure**, computed once — per group on the losing-money card, overall in the sheet. And it is **always historical**: it looks back on every setting, because a fill can only be measured after it has happened.
16. **Forward-looking numbers hide on any averaged period.** Booked and already-replaced describe what is coming; under a period heading — this month, last month, any range — today's figure is noise at best and wrong at worst, so the layers hide there and appear only on Now and Coming up. **Over-occupied is not one of them** — it describes the period being looked at, and it must stay wherever the rate can exceed 100%, because it is the only thing explaining why.
17. ⚠ **The present-moment and period calculations must reconcile before change chips ship.** A chip comparing a number computed one way against a number computed another way will be wrong in a way nobody can see. Pick a property and a date, compute both, and require them to agree before the chips ship. If they do not agree, ship the numbers without chips rather than shipping a comparison that lies.

**On screen**

18. **State chips and layer rows hide at zero.** The booked layer, over-occupied, under notice, disabled-for-rent and the unknown vacancy bar render only when above zero. Three were measured against properties — booked 3.4%, over-occupied 4.2%, disabled for rent 4.5% — so for the large majority these are permanent zeros, and a row of zeros teaches managers to stop reading. **Change chips are a different thing and do not hide at zero**; a number that has not moved shows a neutral chip.
19. **The two forward/backward bar charts must share bucket edges.** 0–7, 8–15, 16–30, 31+ on both. Day 30 falls in exactly one bucket.
20. **Revenue loss carries its basis.** "Based on N of M vacant beds with rent set" is part of the number, not a footnote. Below a trustworthy coverage, show the setup nudge instead of a figure.
21. **Three different situations produce a zero, and they get three different answers.** Getting this wrong is the fastest way to make the screen look broken.

    | Situation | The rate shows | The counts show | The screen shows |
    |---|---|---|---|
    | No rooms at all | — | — | the not-set-up screen |
    | Rooms created, no beds yet | "—" | real zeros | the cards, plus the quiet fill-in line. Occupancy Status has nothing to draw a donut from, so that one card shows its own empty state |
    | Every bed disabled for rent | "—" | real counts | the cards |
    | Rentable beds, nobody in them | **0%** | real zeros | the cards |

    The last row is the important one: **0% is a real, correct number** and must never be suppressed or dressed up as an error. The three rows above it are cases where there is no denominator, and "—" says that honestly where 0% would lie.
22. **Occupancy Rate is not tappable.** A percentage has no records behind it. It carries no chevron and does nothing on tap.
23. **"Booked" is stricter here than on the rooms list.** This screen counts confirmed bookings; the rooms list counts any booking record. Booking numbers open the bookings list, not the rooms list, or the records will not add back to the number tapped.

---

## 19. Open items

1. **Which permission gates this screen** — the suite-wide question, plus the Inventory wrinkle that the rooms list shows the same present-moment numbers under its own gate. Engineering owns it.
2. **Info icon content.** Most cards carry an info icon — inconsistently, see section 20 — and no screen in the suite has written what any of them say. Suite-wide, unowned. The definitions in section 3 are written to be reusable as that content.
3. **The vacancy-age and never-rented drills have no destination**, because nothing records when a room became empty. Section 14 marks them; this is the same prerequisite as item 7 and the reason it blocks a whole card.
4. **The trustworthy-coverage threshold** for the revenue-loss number (at what rent-coverage the rupee figure shows) needs a decided value. Recommendation: two thirds.
5. **Booked-with-no-space** lives here until the Bookings screen is designed, then must be referenced, not rebuilt.
6. **The tab row disagrees with itself** — see section 2. Inventory's position, the flattening of People, Complaints renamed to Issues, and a Leads tab with nothing behind it. Suite-wide, needs one answer, and it is nobody's yet.
7. **Nothing records when a room became empty.** Vacancy age, never rented, Days to Fill **and the past-period revenue-loss figure** all depend on it, and none can be built or drilled into until it exists. The largest single prerequisite on this screen — needs sizing and an owner before a date is set.
8. **The trend chart's default range** is stated as 6 months by judgement, not by evidence. Worth a look once real data exists.
9. **How the Overview tile row is navigated** when it is wider than the card — scroll, wrap, or something else — is undecided. Design.
10. **Do deleted rooms still count?** If a removed room is only flagged rather than actually gone, it pads rentable and reads as vacant forever. Never confirmed. Engineering.
11. **Should a restricted manager see the rupee figures, or only the counts?** This sheet says access is all-or-nothing per screen; the original brief raised money as a separate per-role gate and nobody answered. Decided by omission today, which is not the same as decided.
12. **Blocking a double booking at the point it is made.** The double-booked chip was cut because the case is rare — but rare partly because nothing prevents it and nothing surfaces it. A guard when the booking is created belongs to the booking flow, not to this screen, and should not disappear along with the chip.
13. **Two screens disagree about a tenant under notice.** Collection's sheet states that Dues keeps such tenants inside Active; Dues lists Under Notice as its own row. One of those is wrong, and it affects any breakdown that splits by tenant status.
14. **The Expense sheet describes this screen's toggle as "Bed and Room".** It is Bed and Unit, and unit is deliberately not the same thing as room. One-line fix on that sheet.

---

## 20. Design file: what needs fixing

**Read this first:** every number in the file is placeholder. The fixes below are structural, not data.

### Wrong, needs correcting
1. Vacant Units chip: green when rising. Invert — red up, green down.
2. Under Notice Units chip: green when rising. Invert.
3. Filter chips row: **FlexiPe** and **Report** are not time filters and do not belong on this screen. Remove.
4. Filter chips: replace Today · This Week · This Month · Last Month with **Now · This Month · Last Month · Current FY · Custom · Coming up**, Now selected. Coming up needs a date picker, defaulting to 30 days ahead.
5. Default view: the file shows Unit View selected; the default is **Bed View**.
6. Toggle labels: screen-level toggle and in-card toggle must both read **Bed View / Unit View** (in-card may shorten to Beds / Units). The file currently draws them in opposite states with nothing explaining it; draw them matching, and draw the deliberately-set-aside state separately.
7. Occupancy Trend x-axis: the fifth column is labelled **"Move-out"** — a leftover bar, clipped at the card edge, carrying values, sitting where a month belongs. Remove it. That leaves four month columns, so a six-month default needs two more drawn.
8. Vacant Room Status bucket "30 Days +" vs Upcoming Vacancy "31 Days +": both become **31+ days**.
9. The two adjacent bar charts need subtitles carrying their direction — "how long empty rooms have been waiting" against "when notice-given tenants leave" — or they read as the same chart twice.
10. "No. of lease" on Agreements ending soon becomes **"No. of agreements"**.
11. Property Wise Occupancy is sorted arbitrarily in the file; sort **highest to lowest by vacant space** — vacant beds in Bed View, vacant units in Unit View — so the property needing attention is first (section 13).
12. Empty-state copy is stale on every card: titles reference the previous draft's card names ("Occupancy (Live)", "How long are my rooms vacant", "Rooms to be vacated soon"), one card ("Agreements ending soon") had no live card until this sheet restored it, and the shared body line "You're all caught up! New maintenance requests and…" is truncated mid-sentence and belongs to the Complaints module. Rewrite all per section 16.

### Missing, needs drawing
13. Semi-Vacant slice in the Unit View donut.
14. Never-rented and Unknown bars on Vacant Room Status.
15. The revenue-loss **total** at the top of the losing-money card, with its "based on N of M" basis line.
16. Agreements ending soon card (restored from the previous draft — redraw against current card chrome).
17. Healthy states for the four cards in section 16, and the whole-screen not-set-up state (both existed in the previous draft).
18. The Under Notice chip's second line — "N past their date" and "N already replaced".
19. The View all sheet (section 6) — currently only the link exists; the sheet itself is undrawn.
20. Change-chip states: unfinished-period marker, the "vs this day last month" label on Now, the no-chip state on Coming up, and a neutral grey chip for Rentable.
21. Loading skeletons, failed states, and the Restricted lock — none exist for this screen.
22. In-progress bar treatment on the trend chart.
23. Bed View variants: the file draws every card once, in Unit language — tile labels, "Total Beds: 326" header, donut slices all need their Bed View versions.
24. **Coming up** is entirely undrawn: the chip, its date picker, the default 30-day horizon, how each card reads a projected figure, and the tiles in their no-chip state.
25. The **Vacant legend row** needs its two-layer split drawn in both views: "Vacant 38 — 17 booked, 21 available".
26. The Overview Snapshot header: drop the hardcoded window from the card's name and show the active window beside it instead.
27. The over-100% explanation sentence on the Occupancy Status card, stating over-occupancy and vacancy together.
28. The per-card insight line and its two states — plain text when positive, with a button when negative.
29. Days to Fill shown with its open tail ("avg 14 days · 2 beds still open, longest 86 days"), not as a bare average.
30. The Occupancy Status header carrying both totals — total and rentable — so the legend and the percentage reconcile.

### Leftover content, needs removing

31. Two of the four chips in the Occupancy Status row come out: "Overbooked occupancy" (the double-booked chip is cut) and "Booked Beds" (booked moves into the Vacant legend split). Under Notice and Over-occupied remain.
32. Hidden duplicate delta chips on three Overview tiles (a dead chip stacked under each visible one, in-screen and in the orphan copy).
33. The orphan tile row parked outside the phone frame, with its own hidden chips.
34. The stale single row card ("1 BHK") parked outside the phone, disagreeing with the live card's first row.
35. Hidden "Paid by / Paid to" buttons inside Property Wise Occupancy — expense-card chrome under the occupancy list; the card's internal name still says Expense.
36. Stray artwork group parked outside the frame edge.

### Needs deciding, then drawing

37. The five-tile Overview row and the three-tile chips row both overflow their card with no scroll affordance drawn — decide and draw the scroll treatment.
38. Info icons sit on only three of the five Overview tiles; Rentable Units and Vacant Units have none. Either all five or none.
39. No card has both an info icon and a chevron — three cards explain themselves, four open something, none do both. Cards that do both need both.
40. The four losing-money rows and the four Property Wise rows carry no chevron at all, though every one of them opens a list.
41. Tab row: see section 2. Inventory's position, the missing People grouping, Issues versus Complaints, and the Leads tab.

---

## Where the measured figures came from

Decisions on this screen rest on live production data, measured 2026-08-07. Recorded so the reasoning can be re-checked.

Two figures were **not** measured and are stated as judgement: the coverage threshold below which the revenue-loss figure is hidden, and the trend chart's default range. Both are in Open items.

| Measured | Result | What it decided |
|---|---|---|
| Properties on the new structure | 787 of 73,188 real properties (1.08%); 6.11% of tenants; rising every cohort; full migration planned | Bed View defaults on, designed as the exact path; old structure is the approximation |
| Properties where every room is single | 29.5% of active | The toggle hides where both views are identical |
| Vacant units never occupied | 87–90% of vacant inventory | Never-rented gets its own bucket; the four aging buckets alone describe under a tenth of vacancy |
| Vacant space that has been let before but has no recorded emptying date | ~36% of the never-rented cases | The Unknown bar, hidden at zero as migration completes |
| Room type field | free text, 60+ spellings, 85% "room" | Losing-money card groups by sharing type, not flat type |
| Rooms at sharing 1 / 2 / 3 | 89% | Sharing groups capped at Single / Double / Triple / 4+ |
| Rent set on vacant beds | 25% (old structure) · 42% (new) | Revenue loss carries its basis on its face; below threshold, the setup nudge |
| Rent set on occupied beds | 43–54% | Confirms sparseness is not caused by vacancy itself |
| Double-booked beds, where detectable | 5, of 36,169 beds carrying real identity | Double-booked chip killed, folded into over-occupied |
| Properties with an over-occupied room | 1,061, or 4.16% of active | Chip kept |
| Under notice with a replacement booking | 328 of 4,550 (7.2%) | A second line on Under Notice, not a chip of its own |
| Under notice, move-out date already passed | 428 tenants; 46% over a month past | Second line on Under Notice, not a tile; the number exists partly because nothing has ever shown it |
| Semi-vacant rooms | 56.8% of active properties have one; 11.5% of empty capacity | Donut slice and View-all row, not a tile |
| Disabled for rent | 4.5% of active properties; 79.5% of those have exactly one | Legend entry and View-all row, shown only when present |
| Properties with any confirmed booking | 3.4% of active | The booked layer under Vacant hides at zero rather than showing zero to the rest |
| Confirmed bookings with no space assigned | 603 of 3,676 (16.4%) | View-all row, reserved against the future Bookings screen |
| Multi-property owners | 11–15% of owners; 33–43% of active properties | Property Wise kept, hidden for single-property accounts |
| Properties with account link missing | ~29% of active | Property Wise must not drop unlinked properties |
| Rooms declaring capacity zero | 2,550 | Guarded division, excluded from rentable |
| Real properties with no rooms at all | 34,087 (46.6%) | The whole-screen not-set-up state restored |
