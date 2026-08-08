---
title: DA-09 Tenants — Handoff Sheet
date: 2026-08-07
tags:
  - rentok
  - tenants
  - people
  - detailed-analytics
  - handoff
status: v2.1 · share-of-total retrofit from the Dues sibling check · developer handoff
owner: Sanchay
---
# Tenants — Handoff Sheet

> [!NOTE] Corrected 2026-08-08, from the Inventory uplift's sibling check
> - The booked-layer sentence claimed equality with Inventory's booked count; the two totals differ by design and the sentence now says how.
> - The chips rule and the two-kinds line were stated as suite laws; both are now scoped to this screen.

> [!NOTE] Corrected 2026-08-09, sibling debts paid
> - Section 4's filter-memory line was a paraphrase; replaced with Expense v3 §4's exact wording, and added the day-boundary sentence it was also missing. Converted all 7 test lines to the suite's "Test it:" phrasing.


Everything on the Tenants analytics screen: what each number means, what window it covers, what happens when it is tapped, and what it shows when there is nothing to show.

**Changed 8 August 2026:** share-of-total added as a requirement on Property Wise Active Tenants (section 18), matching a rule locked while building the Dues sheet. Anyone who already built section 18 without it needs this one addition.

---

## What is in here

|        | Section                              | For              |
| ------ | ------------------------------------ | ---------------- |
| **1**  | Build status                         | everyone         |
| **2**  | Where this lives                     | everyone         |
| **3**  | What every number counts             | backend + copy   |
| **4**  | How the screen behaves               | backend + design |
| **5**  | Overview Snapshot                    | backend + design |
| **6**  | View all sheet                       | backend + design |
| **7**  | Move-in & Move-out Metrics           | backend + design |
| **8**  | Journey                              | backend + design |
| **9**  | Tenant Verification                  | backend + design |
| **10** | Tenancy Details                      | backend + design |
| **11** | Upcoming Eviction                    | backend + design |
| **12** | Agreement Expiry Status              | backend + design |
| **13** | Tenant Profile                       | backend + design |
| **14** | Tenant Details                       | backend + design |
| **15** | Renewal & Retention                  | backend + design |
| **16** | Stay Type                            | backend + design |
| **17** | How long tenants have been here      | backend + design |
| **18** | Property Wise Active Tenants         | backend + design |
| **19** | What each number opens               | backend          |
| **20** | Who can see this                     | backend          |
| **21** | What each card shows when it is empty, healthy or broken | design + copy |
| **22** | What this screen is not              | everyone         |
| **23** | Build guidance                       | backend          |
| **24** | Open items                           | everyone         |
| **25** | Design file: what needs fixing       | design           |
|        | Where the measured figures came from | everyone         |

**Building the backend?** Read everything except sections 21 and 25. Start with Build guidance.

**Working on the design?** Sections 4 to 18, 21, and 25, which collects every fix in one list. Section 3 is the plain wording for the info icons.

**Just need the decisions?** Sections 1, 22 and 24, plus the measured figures at the end.

---

## 1. Build status

The screen is **not built**. The backend serves one empty placeholder block for Tenants. Nothing is broken; it is unwritten. Old Tenants and Bookings are in the same state.

What already exists and matters:

- The **tenants, bookings and old-tenants lists** are built and live. This screen drills into all three.
- **Booking confirmations, notices, approvals and renewals are all kept in full**, with who did what and when. The history this screen needs exists.
- What the lists **cannot yet do** is answer questions like "who left last month" or "whose agreement ends in 60 days", they can show who is here, not what happened in a window. Those filters have to be added. No new history needs to start being kept.

---

## 2. Where this lives

Manager app, property header, **People** section, **Tenants** tab.

The tab row, settled for the whole suite:

| | |
|---|---|
| Financial | Dues · Collection · Expense |
| **People** | **Leads · Bookings · Tenants · Old Tenants** |
| Inventory | — |
| Complaints | — |

People's sub-tabs run in lifecycle order. Complaints keeps its name. **Leads is a ninth screen, unwritten.** Nothing here depends on it, and it is not built from any card on this screen.

---

## 3. What every number counts

Shared by every card. Bed, unit, vacant, occupied and booked keep their Inventory meanings.

### Tenant

A person renting a space. Everyone is one of three things, moving one way:

| State | Meaning | List |
|---|---|---|
| **Booking** | Booked, not moved in yet | Bookings |
| **Active** | Living here now | Tenants |
| **Old** | Moved out | Old Tenants |

The same person throughout. A booking becomes active on move-in day; active becomes old on leaving day. So a move-in and a booking-converted are one event seen from two sides, and the numbers move together.

### Active tenant

Somebody living here today. **A tenant under notice is still active**, they live here; they have said they are leaving. Under notice is a layer inside Active, never subtracted from it, exactly as booked is a layer inside vacant on Inventory. The same goes for an approved eviction.

### Booking

A person who has booked and not moved in yet. A booking has two states, and both are bookings:

| State | Meaning |
|---|---|
| Confirmed | The property has said yes, or the property accepts bookings automatically |
| Awaiting confirmation | Booked, and the property has not said yes yet |

**Cancelled is not a booking any more.** It is excluded everywhere except the cancellation count, and the person lands on the Old Tenants list even though they never lived here.

A booking occupies no bed and earns nothing until the person arrives; the bed they will fill is still vacant, here and on Inventory. Inventory counts the confirmed ones as its booked layer, so the two screens meet through confirmed bookings.

### Notice

Somebody living here has a leaving date on record. Two things about it matter:

**Who put it on record.** A notice raised **by the tenant** needs the property's approval; this is the whole of the awaiting-approval number. A notice raised **by the property** is already settled. This says who recorded it, not whose idea it was, so the screen never labels these "voluntary" and "eviction", that would claim a reason nobody recorded.

**Which date it carries.** A leaving date is **requested** when the notice is raised, and **confirmed** when it is approved. Upcoming Eviction shows both, marked apart. Everything that counts real departures uses the confirmed date only, nobody is chased over a date that was never agreed.

A withdrawn notice stops counting everywhere, immediately.

### Past their date

An active tenant whose confirmed leaving date has passed. Usually the person has already gone and the record was never closed. This is common where automatic move-out is switched off. The action is **check and close**, not send a reminder. These tenants still count in Active, so this screen always matches the tenant list; closing the records is what corrects the count.

### Stayed after notice

Gave notice, leaving date has passed, still living here. The manager saved them. Counted only after the date passes; someone in their notice period hasn't been saved yet. **Never combined with Past their date**: one is a person who changed their mind, the other is a record nobody closed.

### Move-in / move-out

Started or stopped living here during the window. Counts of events, never a state. **Move-out counts everyone who left, however their leaving was noted**, a property where people have left never shows zero. **Moving between rooms is neither**: the person never left. **A cancelled booking is not a move-out either.** The person lands on the Old Tenants list, but they never lived here. *Test it:* cancel a booking; Move-outs is unchanged and Cancelled rises by one.

### Renewal

An agreement extended instead of ending. **Renewal due** = ends within 30 days, no decision yet. A renewal and a notice are opposite answers to "is this person staying", which is why both sit on the Journey card.

### Agreement end date

Every long-term tenant has one, worked out in this order, first available wins:

1. The recorded end date
2. Start date + the tenant's own agreement duration
3. Start date + the property's agreement duration
4. Start date + 11 months

Start date = last renewal date, else joining date. Where step 4 was used, the card says the date is assumed. Whether a **signed document** exists is a different question and lives on Tenancy Details, a tenant can have a known term and no document, or a document and no dates.

### Verification

Three separate things: **ID verification** (E-KYC, done automatically · manually verified · not done), **police verification** (done or not, with a deadline, section 9), and the **rent agreement** (how it was signed, or not).

### Profile completion

**Complete = all ten of these are filled in:** phone, date of birth, joining date, permanent address, father's name, mother's name, father's phone, father's occupation, local guardian's name, local guardian's phone. This is the same rule the tenant list uses, so the number and the list always agree.

### Tenant type

Working professional · Student · Family · Bachelor · Couple · Company · Property staff · Relatives & friends · Other. *Test it:* filter the tenant list by Student on a property with students, the count must match this card.

### Money on this screen

Two figures, both risk beside a count, never transactions:

| Figure | Meaning |
|---|---|
| **Rent at risk** | Monthly rent of everyone under notice: what stops arriving if nobody is replaced |
| **Leaving with dues** | What departing tenants still owe: money that walks out unless chased before the leaving date |

Anyone owing anything counts. Collections, settlements and refunds stay on the Financial screens.

### Words to be careful with

**"Under notice"** is the phrase, matching Dues and Inventory. **"Booking"** here is the person; on Inventory it is the layer inside a vacant bed. Inventory counts confirmed bookings only, so its booked layer draws only from this screen's confirmed bookings, never the awaiting ones. The two totals do not match: a confirmed booking behind a tenant under notice sits inside Inventory's Occupied as "already replaced", and a confirmed booking with no bed assigned sits outside every layer, so Inventory's booked layer is always the smaller number. **"Active"** describes both a tenant (living here) and a booking (waiting to arrive), never sum them.

---

## 4. How the screen behaves

### The time filter

**This Month (default) · Last Month · Current FY · Custom · All Time · Coming up.**

Two kinds of number share this screen, two of the suite's three kinds; screens with projections add Forecast:

| Kind | Meaning |
|---|---|
| **Live** | Always the current snapshot. No filter setting changes it. Says **"as of today"** on its face |
| **Time-scoped** | Counts what happened inside the window the filter picks |

A worked example. It is October, and the filter is on **Last Month**:

| Number | Kind | Shows |
|---|---|---|
| Active Tenants | Live | 360: the people living here **today**. September's headcount no longer exists to count |
| Move-ins | Time-scoped | 14: the people who joined **during September** |
| Notices Raised | Time-scoped | 6: the notices raised **during September** |
| Upcoming Eviction | Live, forward | Who is leaving, always counted **from today onwards** |

Change the filter to This Month and only the time-scoped numbers change. That is the whole rule.

**Why there is no "Now" chip.** Every number that answers "right now" (the headcount, the pipelines, the verifications) is Live, and Live numbers already show now on every filter setting. A manager asking "how many tenants are active right now" reads the Active Tenants tile, whatever the filter says. A Now chip would add nothing to those numbers and would blank Move-ins, Move-outs, Approved Bookings and Notices Raised, because "movement right now" has no window to count. Whoever wants today's movement picks Custom set to today.

**Coming up** is the forward setting, as on Inventory: pick a future date, see the property then: today's tenants, minus confirmed departures before that date, plus confirmed arrivals. Confirmed only, no guesses. Default 30 days. No change chips and no action bars; nothing can be done today about a state that hasn't arrived.

**Custom stops at today.** The past and the present belong to Custom; the future belongs to Coming up. A future window is a projection, not a count, and one question must never have two answer models on one screen.

The filter stays where the manager put it while the app is open; a fresh launch opens on the default. Coming back from a drill returns to the screen as it was left, with fresh numbers. A day runs midnight to midnight, India time.

### What every number does on every filter setting

| Number | This Month · Last Month · Current FY · Custom · All Time | Coming up |
|---|---|---|
| Active Tenants · Active Bookings | As of today | Projected at the chosen date, from confirmed departures and arrivals only |
| Eviction Pending · Eviction Approved · Past their date | As of today | As of today |
| Move-ins · Move-outs | Counted inside the window | Scheduled: confirmed arrivals and departures due before the chosen date |
| Approved Bookings · Notices Raised | Counted inside the window | Nothing agreed to count; the tile shows "—" |
| Journey rows | As of today | Projected |
| Verification · Tenancy Details · Profile · Details · Stay Type · Tenure | As of today | As of today. Paperwork and people's details cannot be projected |
| Upcoming Eviction · Agreement Expiry | From today onwards | From today onwards |
| Renewal & Retention: Completed, renewed, stayed | Counted inside the window | "—" |
| Property Wise | As of today | Projected |
| The two money lines | As of today | Projected from the same confirmed facts |

Nothing on Coming up ever invents an event. A tile with nothing agreed to count shows a dash, never a guess.

*Test it:* on a property unchanged for a year, switching This Month to Last Month changes every time-scoped number and no Live one.

Cards with their own date dropdown follow the three suite rules: same options as the top filter, the top filter pulls every card back in line, one card can be deliberately set aside.

### Change chips

Only time-scoped numbers carry one on this screen; an event count has nothing to compare on a live view. Inventory rules the other way for its state numbers, which do have a predecessor, and says so. Nothing carries one on All Time or Coming up. Which direction counts as good is per number: **up is good** for Move-ins, Approved Bookings, Renewals; **up is bad** for Move-outs and Notices Raised. Unfinished periods compare against the same elapsed days of the previous period, marked as such. On Custom, the comparison is the same number of days immediately before the range.

### When a number is red

**Red only where somebody can be held to it:**

| Red | Plain |
|---|---|
| An obligation unmet: police verification past its deadline, no agreement on record | A gap with no consequence: gender, food preference, tenant type not recorded |
| A date already passed: leaving date gone, agreement expired | A deadline not yet reached: joined three days ago |

A property onboarded this morning must not open red.

### The action bar

Some cards carry a footer naming what needs attention, with one control that **opens the filtered list where the work happens**. This screen never sends anything itself. Appears only when there is something to say; never at zero, never on Coming up.

### Loading, failure, sorting, entry

Suite rules unchanged: every card loads, fails and retries alone; skeletons match their card; a failed card shows "Couldn't load this" and Retry, never a number; empty and failed look different. Value breakdowns sort highest to lowest. Bars with a natural order (day buckets, tenure bands, expiry bands) keep that order. Entry is the Tenants tab; screens drawn around it in the design file are context only.

---

## 5. Overview Snapshot

Seven tiles. Header carries the active window.

| Tile | Counts | Kind |
|---|---|---|
| Active Tenants | People living here | Live |
| Active Bookings | Booked, not moved in. Confirmed and awaiting together | Live |
| Approved Bookings | Bookings confirmed in the window | Time-scoped |
| Eviction Pending | Notices awaiting approval | Live |
| Eviction Approved | Departures scheduled | Live |
| Notices Raised | Notices raised in the window | Time-scoped |
| **Past their date** | Confirmed leaving date passed, record not closed | Live |

Live tiles say "as of today" and carry no chip. **Move-outs is not a tile.** It lives on the card below, beside Move-ins; Past their date takes the slot, and no other card surfaces it.

Two money lines ride under the tiles: **rent at risk** under the notice tiles, and **leaving with dues** ("3 leaving owe ₹48K"). This is the one number that pairs a departure with a deadline nobody can extend.

Active Tenants includes everyone under notice and approved eviction, layers, not slices. Nothing on screen should suggest the tiles add up.

**View all** opens the sheet below.

## 6. View all sheet

Everything the tile row knows, grouped as three plain questions. Indented rows are layers inside the row above, never separate totals. Every row that can open a list does.

**Who is here**

| Row | Meaning |
|---|---|
| Active tenants | Everyone living here |
| — under notice | Have said they are leaving |
| — of which approved | Leaving date agreed |
| — of which awaiting approval | Leaving date not agreed yet |
| — past their date | Confirmed leaving date already passed |
| Short term / Long term | The stay-type split |

**Who is coming**

| Row | Meaning |
|---|---|
| Active bookings | Booked, not moved in |
| — of which confirmed | The property has said yes |
| — of which awaiting confirmation | Not said yes yet. Shown only when above zero |
| — arriving this month | Expected within the current month |

**What moved**

| Row | Meaning |
|---|---|
| Move-ins | Started living here in the window |
| Move-outs | Stopped living here in the window |
| Notices raised | Raised in the window |
| Bookings confirmed | Confirmed in the window |
| Bookings cancelled | Cancelled in the window |

---

## 7. Move-in & Move-out Metrics

Move-ins and Move-outs side by side, with the **net stated**, the card exists because the two only mean something against each other. Both are time-scoped. Chips run in opposite directions: green when Move-ins rise, red when Move-outs rise.

*Test it:* on a property where people have demonstrably left, Move-outs is never zero, and tapping it returns exactly that many people.

## 8. Journey

Where people are in their life with the property. **Two tabs: Tenants (default) · Bookings.** No Leads tab, a lead is not a tenant at any stage.

**Tenants tab.** The headcount is stated once at the top, "360 tenants", never drawn as a bar. Three bars below, each Live, each a share of that headcount:

| Bar | Meaning |
|---|---|
| Under Notice | Living here, has said they are leaving |
| Approved Eviction | Living here, departure scheduled |
| Renewals in 30 days | Agreement ends within 30 days |

These overlap each other, parallel bars only, never stacked, never slices of a whole.

Below, all time-scoped: **Churn** (left, against who was here) · **Renewal** (agreements extended) · **Net change**. At the foot, also time-scoped: **Tenants who left early** (left before their lock-in period ended) and **Avg. days to raise notice** (joining to first notice).

Footer: "N eviction pending → View more", opening the list.

**Bookings tab.** Total · Approved · Cancelled · Converted, all counted in the window, with conversion and cancellation shown as counts, not only rates. Converted and Cancelled describe finished bookings; Total includes ones still in progress, they will not add up, and the card must not imply they do.

## 9. Tenant Verification

Two tabs, both Live, across active tenants. Every row carries its share, and the card says how many tenants it describes.

**ID Verification:** E-KYC · Manually verified · Not verified.

**Police Verification:**

| Row | Colour |
|---|---|
| Done | plain |
| Not done, still within 7 days of moving in | plain |
| **Overdue**, more than 7 days in, no check | **red** |

The deadline is **seven days from move-in, same for every property**. The action bar names only the overdue among tenants who joined in the last 90 days: the queue somebody can actually work through. The full overdue count still shows.

Police verification and the rent agreement are the two things an inspector asks for; the card groups them apart from good-practice items and says they carry outside risk.

## 10. Tenancy Details

Two tabs, Live, across active tenants.

**Rent Agreement:** Stamp signed · Simple signed · Manually uploaded · **Not signed (red)**. Only "not signed" is distinguishable today; the three signed kinds need the signing route to be kept (open item 2).

**Profile Completion:** Completed · Pending, by the ten-detail rule in section 3. Pending is plain, not red: an incomplete profile is a gap, not an obligation.

## 11. Upcoming Eviction

Who is confirmed to be leaving, and how soon. Notices only, no predictions. Live, forward-looking, "from today onwards" on its face.

Five bars, **Overdue · 0–7 · 8–15 · 16–30 · 31+ days**, each split into awaiting approval and approved, because one can still be changed and the other is a plan. Bucket edges match Inventory exactly.

The Overdue bar and the Past-their-date tile are one number, computed once.

## 12. Agreement Expiry Status

When agreement periods run out, using the end-date rule from section 3. Five bars, covering everyone:

**Already expired · 30 days · 60 days · 90 days · Valid.**

Already expired is drawn first because it is the biggest group on most properties. Bands do not overlap: 30 means within thirty days, 60 means thirty-one to sixty, 90 means sixty-one to ninety. One line on the card says dates are assumed at 11 months where no duration is recorded.

The 30-day bar and Journey's "Renewals in 30 days" are the same people and can never disagree, one sits beside its timeline, the other beside the notices, which is the point.

Footer: "N agreements expiring → Notify Tenant", opening the tenants list filtered to them, where messaging a group already works.

## 13. Tenant Profile

Three tabs, Live, across active tenants: **Gender** (Male · Female · Prefer not to say) · **Age** (14–22 · 23–35 · 36–50+) · **City** (most common, plus Others).

Each tab carries its coverage, "based on N of M tenants who have this recorded", because these details exist for a minority today and a chart without that line pretends to describe everyone. Unrecorded is excluded from the split, never folded into a bucket, and is plain, not red.

## 14. Tenant Details

Three tabs, Live: **Food Preference** (Pure veg · Eggetarian · Non-veg · No food) · **Tenant Type** (per section 3) · **Institute** (most common, plus Others).

Ships as drawn. Coverage is low today because nothing ever asked, properties will backfill and new properties collect from the start, so the card is built to improve on its own. The coverage line is this card's most important element.

## 15. Renewal & Retention

| Figure                  | Meaning                                                              | Kind      |
| ----------------------- | -------------------------------------------------------------------- | -------------- |
| Renewal Due             | Agreement ends within 30 days, no decision yet                       | Live           |
| **Renewal overdue** | Agreement already ended, tenant still here, not renewed | Live |
| Completed               | Agreements extended in the window                                    | Time-scoped |
| Tenants who renewed     | Extended at least once                                               | Time-scoped |
| **Stayed after notice** | Of those whose leaving date fell in the window, the share still here | Time-scoped |

Renewal overdue is the same number as the Already-expired bar on Agreement Expiry, computed once and shown in both places, exactly as Renewal Due is the 30-day bar.

**Stayed after notice** is one question: *they said they were leaving; are they still here?* However they came to stay does not change the answer. Never combined with Past their date.

## 16. Stay Type

Two independent splits of today's active tenants: **Short term · Long term**, and **B2B · Residential** (let to a company · let to an individual). Not alternatives: a short-term company letting is both. Where renting type is unrecorded the split does not render; a missing value is not an answer.

## 17. How long tenants have been here

Five bars, Live: **Under 1 month · 1–3 months · 3–6 months · 6–12 months · Over a year.**

The only card describing people who have not decided anything yet. Every other departure number starts from a notice already raised. The cut points match the lock-in periods people actually sign, so each band is a group approaching a decision point. Each bar opens the tenants list for that joining range.

## 18. Property Wise Active Tenants

Hidden with one property in scope; returns automatically with more. Per property: name, active count, bar, and **its share of the account's total active tenants**, sorted highest to lowest. Properties with zero still appear: a missing property reads as an error, a zero reads as a fact. Tapping a row opens that property's tenant list; it never re-scopes the screen.

---

## 19. What each number opens

### The rules

- **A drill filters a list. It never re-scopes the screen.** The one exception: a trend bar moves the screen's window.
- **The destination follows the person's state.** Living here: the Tenants list. Arriving: the Bookings list. Left, or a cancelled booking: the Old Tenants list.
- The destination opens on **the same properties** the screen was showing, and the back control reads "Tenants". Whether the window travels depends on the kind of number; the table below is the rule.
- **Records add back to the number tapped** on any window that includes today. On a fully past window the list shows the people as they are now, and names the difference on arrival.
- **Coming back from a list returns to the screen exactly as it was left, with fresh numbers.** *Test it:* close a stale record from Past their date, come back; the tile has moved.
- Rates, averages and percentages are not tappable. Everything tappable looks tappable.
- The destination names the slice, shows the active filter on arrival, and names the filter in its empty state (the suite-wide requirement, still unowned).

### When the window changes what a tap shows

| The screen's window | What travels to the list |
|---|---|
| A Live number, on any window | Nothing. The number was as-of-today, so the list opens as-of-today |
| A period, running or finished | The window travels, and the list shows those people **as they are today**. Where some have since moved on, the list says so on arrival: "showing 11 of the 14; the rest have since moved out" |
| Coming up | The destination itself moves to where those people are **today**: scheduled arrivals open the **Bookings list**, scheduled departures open the **Tenants list**. Nothing on Coming up opens the Old Tenants list |

*Test it:* filter Last Month, tap Move-ins showing 14. The list opens on last month's joiners as they stand today, and if only 11 remain it says so on arrival. Eleven silent rows under a tile saying 14 is a failed test.

### The tap matrix

Every tappable thing, what it opens, and what is already applied on arrival. ✅ works today · ⚠ partly · ❌ the filter has to be added first. **Every ❌ is a filter over data that already exists, never history to start keeping.**

| You tap | What opens | Arriving filtered to | Ready? |
|---|---|---|---|
| **Overview Snapshot** | | | |
| Active Tenants | Tenants list | Everyone living here | ✅ |
| Active Bookings | Bookings list | All bookings, confirmed and awaiting together | ✅ |
| Approved Bookings | Bookings list | Confirmed during the window shown | ❌ no confirmation-state or confirmation-date filter |
| Eviction Pending | Tenants list | The "Pending Eviction" filter | ✅ |
| Eviction Approved | Tenants list | The "Approved Eviction" filter | ✅ |
| Notices Raised | Tenants list | Notice raised during the window | ❌ no notice-date range |
| Past their date | Tenants list | Confirmed leaving date already passed | ✅ |
| Rent at risk line | Tenants list | Everyone under notice | ✅ |
| Leaving with dues line | Tenants list | "Under Notice" plus "Unpaid Dues", together | ✅ |
| View all | The View all sheet | — | — |
| **View all sheet rows** | | | |
| Under notice | Tenants list | Has a leaving date | ✅ |
| Of which approved / awaiting | Tenants list | "Approved Eviction" / "Pending Eviction" | ✅ |
| Past their date | Tenants list | Leaving date already passed | ✅ |
| Short term / Long term | Tenants list | That stay type | ✅ |
| Bookings, confirmed / awaiting | Bookings list | That state | ❌ no confirmation-state filter |
| Arriving this month | Bookings list | Joining date inside this month | ✅ |
| Move-ins | Tenants list | Joined during the window | ✅ |
| Move-outs | Old Tenants list | Left during the window; the list offers past windows and a custom range | ✅ |
| Bookings cancelled | Old Tenants list | Cancelled bookings only | ❌ the list cannot single them out |
| **Move-in & Move-out** | | | |
| Move-ins | Tenants list | Joined during the window | ✅ |
| Move-outs | Old Tenants list | Left during the window, as above | ✅ |
| The net | Nothing; two populations, no single list | — | — |
| **Journey, Tenants tab** | | | |
| Under Notice bar | Tenants list | Has a leaving date | ✅ |
| Approved Eviction bar | Tenants list | The "Approved Eviction" filter | ✅ |
| Renewals in 30 days bar | Tenants list | Agreement ending within 30 days | ✅ |
| Churn · Renewal · Net change | Nothing; rates and nets are not tappable | — | — |
| Tenants who left early | Old Tenants list | Left before their lock-in ended | ❌ no such filter |
| Eviction pending footer | Tenants list | The "Pending Eviction" filter | ✅ |
| **Journey, Bookings tab** | | | |
| Total | Bookings list | Booked during the window | ✅ |
| Approved | Bookings list | Confirmed | ❌ no confirmation-state filter |
| Cancelled | Old Tenants list | Cancelled bookings only | ❌ as above |
| Converted | Tenants list | Joined during the window | ⚠ the list cannot tell converted bookings from direct joins |
| **Tenant Verification** | | | |
| E-KYC · Manually verified | Tenants list | "Digitally Verified Profile" / "Documents Verified" | ✅ |
| Not verified | Tenants list | Identity not confirmed | ✅ |
| Police verification: done | Tenants list | Check on file | ✅ |
| Police: not done, still in time | Tenants list | No check, joined within 7 days | ✅ |
| Police: overdue | Tenants list | No check, joined over 7 days ago | ✅ |
| Action bar (overdue, recent) | Tenants list | No check, joined in the last 90 days | ✅ |
| **Tenancy Details** | | | |
| Stamp · Simple · Manually uploaded | Tenants list | That signing route | ⚠ only on-file-or-not today |
| Not signed | Tenants list | No agreement on record | ✅ |
| Profile completed / pending | Tenants list | The list's own completed-profile filter | ✅ |
| **Upcoming Eviction** | | | |
| Overdue bar | Tenants list | Leaving date already passed | ✅ |
| 0–7 · 8–15 · 16–30 · 31+ bars | Tenants list | Leaving within that many days | ⚠ the list offers within-N windows (3, 7, 15, 30 days), not the card's bands. A band drill opens the nearest window and says so, until band filters exist |
| **Agreement Expiry Status** | | | |
| Already expired | Tenants list | The agreement-window "Past" filter | ✅ |
| 30 days | Tenants list | Agreement ending within 30 days | ✅ |
| 60 · 90 days · Valid | Tenants list | That band | ⚠ the list offers within-N windows, not bands, and nothing expresses "valid beyond 90" |
| Notify Tenant footer | Tenants list | The tapped band, ready to message | Per the band above |
| **Tenant Profile** | | | |
| Any gender row | Tenants list | That gender | ✅ |
| Any age band | Tenants list | That age band | ❌ the list filters exact values, not ranges |
| Any city row | Tenants list | That city | ❌ no city filter |
| **Tenant Details** | | | |
| Any food preference row | Tenants list | That preference | ✅ |
| Any tenant type row | Tenants list | That type | ⚠ the list's filter reads a different place than this card today; counts will not match until both read one place |
| Any institute row | Tenants list | That institute | ❌ no institute filter |
| **Renewal & Retention** | | | |
| Renewal Due · Renewal overdue | Tenants list | Ending within 30 days · the "Past" agreement filter | ✅ |
| Completed · Tenants who renewed | Tenants list | Renewed during the window | ❌ no renewal-date filter |
| Stayed after notice | Nothing; a share, not tappable | — | — |
| **Stay Type** | | | |
| Short term / Long term | Tenants list | That stay type | ✅ |
| B2B / Residential | Tenants list | That renting type | ✅ |
| **How long tenants have been here** | | | |
| Any tenure bar | Tenants list | Joined inside that band | ✅ |
| **Property Wise** | | | |
| Any property row | Tenants list | That property, everyone living there | ✅ |


### What the lists can already do, and what has to be added

Checked against the filter drawer both apps share; the web list and the phone list use one filter set. The lists already filter by: who is living, booked or gone; joining, record-created and **leaving** dates in windows, including a custom range; eviction state (pending or approved) and leaving-date windows; agreement-end windows, including already ended; identity verification **and its route**; agreement signed or not; police check uploaded or not; profile completion; dues unpaid or cleared; stay type; food preference; gender; autopay; app downloaded. That covers every ✅ above.

What remains, and nothing else:

| New filter to add | On which list | Which numbers wait on it |
|---|---|---|
| Notice-raised date range | Tenants | Notices Raised |
| Confirmation state and date | Bookings | Approved Bookings, the confirmed and awaiting layers |
| Band filters for the day buckets | Tenants | The lists offer within-N windows; the eviction bars and the 60 and 90 expiry bands need true bands |
| Valid beyond 90 days | Tenants | The Valid bar |
| Renewal date range | Tenants | Completed, Tenants who renewed |
| Cancelled bookings only | Old Tenants | Bookings cancelled |
| Left before lock-in ended | Old Tenants | Tenants who left early |
| Age band, city, institute | Tenants | Those Profile and Details rows |

The remaining ⚠ checks: the three signing kinds still show only signed-or-not, and need the signing route kept from now on; the tenant-type filter and its card must read one place; and telling a converted booking from a direct join needs the link kept when a booking converts.

---

## 20. Who can see this

**Anyone who can view tenants.** The suite rule: each analytics tab follows the permission of the records it describes. No partial state. If someone's access is narrowed to certain properties, every number counts only those properties and still matches what the drill returns. Without permission: the standard lock, *"Analytics Restricted, You don't have permission to view these analytics. Request access from your admin,"* with Request Access.

## 21. What each card shows when it is empty, healthy or broken

### The zeros, told apart

Seven situations produce an empty-looking card, and they are never allowed to look alike:

| Situation | What shows |
|---|---|
| Never had anyone: no tenants, no bookings, no history | The not-set-up screen below, and nothing else |
| Bookings confirmed, nobody has moved in yet | The cards with real zeros, and one quiet line: "Tenants appear here from your first move-in." |
| Had tenants; everyone has left | The cards with real zeros. A true and alarming number, never dressed up as setup |
| Zero inside the window: no move-ins last month | The zero, with a neutral chip. Zero is real |
| Zero that is good news | The healthy wording below, no CTA |
| Nothing recorded for a breakdown | The not-recorded state: the card draws what exists and states its coverage |
| The card failed | "Couldn't load this" and Retry. Never a zero, never a healthy message |

### Not set up

No tenants, no bookings, no history: no cards render. One full-screen state: *"No tenants yet. Add your first tenant and this page fills in."* with an **Add tenant** button. The most common state on the platform.

### Healthy: good news, no CTA

| Card | Reads |
|---|---|
| Upcoming Eviction | "Nobody is leaving soon. No tenants have given notice." |
| Agreement Expiry Status | "No agreements ending in the next 90 days." |
| Renewal & Retention | "Nothing due for renewal." |
| Past their date at zero | "Every move-out is closed off." |
| Journey, all three bars at zero | Headcount stays; the bars give way to "All quiet. Nobody is under notice and nothing is due." |

### Empty: nothing yet

| Card | Reads |
|---|---|
| Tenant Profile or Tenant Details, nothing recorded | "No details recorded yet. These fill in as you add tenant details." |
| Journey, under a month of history | "Not enough history yet." |
| Property Wise, one property in scope | The card does not render at all |

### Not recorded

Its own state, not empty, and the common case on this screen: the card draws its chart over whatever is recorded and states the coverage. Only where nothing at all is recorded does it fall back to the empty wording above.

### Failed

"Couldn't load this" plus Retry. Never a healthy message, never a zero.

---
## 22. What this screen is not

- **Not a management screen.** Nothing here edits, approves or sends. It diagnoses; every action lives on the lists it opens.
- **Not the tenant list.** The list answers "who is here" and acts. This screen answers what the list cannot: what moved, what is coming, how this month compares. That is why it earns a tab.
- **Not Old Tenants, Bookings or Leads**, each has its own tab and its own sheet.
- **No predictions.** Confirmed notices and confirmed bookings only. A forecast nobody can act on is worse than none.
- **No transactions.** Two risk figures only; every rupee that moved lives on Financial.
- **No individual tenant.** Counts and breakdowns only.

## 23. Build guidance

1. **Live and time-scoped numbers must never blur.** *Test it:* unchanged property, switch This Month → Last Month; every Live number identical, every other changes.
2. **Under notice, approved eviction and past-their-date sit inside Active.** *Test it:* 100 living, 12 under notice → Active shows 100.
3. **A withdrawn notice stops counting everywhere at once.** *Test it:* raise and withdraw same day → every number back where it started.
4. **A booking converting moves four numbers together**, Active Bookings −1, Active Tenants +1, Move-ins +1, Converted +1.
5. **Compute each number once.** Past-their-date is a tile and the Overdue bar; Renewal Due is a figure and the 30-day band; Renewal overdue is a figure and the Already-expired bar. One computation, shown twice.
6. **This screen agrees with the tenant list, the homescreen and Inventory** about the same people. *Test it:* pick a property, compare Active here, the list's count, and Inventory's occupied beds.
7. **Move-outs counts every departure however it was noted.** *Test it:* a property with known departures never shows zero, and the tapped number equals the rows returned. The Old Tenants list must show the same people.
8. **A room change is not a move-out.** *Test it:* move a tenant between rooms → Move-ins and Move-outs unchanged.
9. **Use the confirmed leaving date for departures, the requested date only inside Upcoming Eviction.** *Test it:* a tenant whose requested date passed unapproved appears in Upcoming Eviction, not in Past their date.
10. **Coverage lines are part of the number** on every sparse breakdown. Unrecorded is excluded, never defaulted.
11. **Coverage will grow.** These cards are near-empty by intent today; build them to improve on their own as details are backfilled and new properties collect from day one, not to need rewriting when the data arrives.
12. **Overlapping bars are never stacked.** Journey's three rows overlap; parallel only.
13. **Bands do not overlap and match Inventory's edges**, 0–7, 8–15, 16–30, 31+.
14. **Layer rows hide at zero** (awaiting confirmation, past-their-date, the pending/approved split). Change chips do not; an unmoved number shows a neutral chip.
15. **Zero is real.** A property with no notices this month shows zero with the healthy wording, never "no data".
16. **A day runs midnight to midnight, India time**, on every card and every drill. *Test it:* a tenant joining at 11:55pm counts as that day's move-in, and their list row agrees.

## 24. Open items

1. **Info icon content**, every card carries one; no screen in the suite has written them. Section 3 is reusable as that content. Suite-wide.
2. **The three signed-agreement kinds** need the signing route kept from now on; until then those rows show signed-or-not. The identity-verification routes are already told apart.
3. **Tile row overflow**, scroll or wrap. Design.
4. **Trend chart**, specified, not committed: move-ins vs move-outs per month with net, own 6/12/24 range, exempt from the filter, matching the suite. Ships if design and engineering size it in.
5. **Some departures do not appear on the Old Tenants list** even though the people have left. Move-outs counts them regardless; the list must catch up so the number and the list agree. Engineering.

## 25. Design file: what needs fixing

Every number in the file is placeholder; these are structural.

**Wrong:** 1. Chips are visible on exactly the two tiles that should never have one (Active Tenants, Active Bookings) and switched off on the two that should (Approved Bookings, Notices Raised). Swap them, and delete the other five. 2. The Move-outs tile is replaced by **Past their date**. 3. Notices Raised chip red when rising. 4. The card internally named "Property Expense" is Property Wise Active Tenants; rename it and remove the hidden Paid by / Paid to. 5. All fourteen empty states carry truncated Complaints copy; rewrite per section 21. 6. Tenant Details third tab: "Office" on two variants, "Institute" on one. It is Institute. 7. Two Tenancy Details variants both named "Default"; name them for their tabs. 8. Journey foot labels: keep the on-screen wording ("Tenants who left early", "Avg. days to raise notice"); fix the component. 9. Upcoming Eviction: the 31+ bar sits outside the card edge with one value instead of two. Bring it in. A stray "Move-out" column sits further out; remove it (the same leftover as Inventory's trend chart). 10. The Renewal & Retention card draws **"Renewal rate after notice"**; the figure is renamed **Stayed after notice**.

**Missing:** 11. Coming up chip, its date picker, and the screen's chipless state. 12. "As of today" on the five Live tiles. 13. The View all sheet. 14. Coverage lines on five cards. 15. The net on Move-in & Move-out. 16. The pending and approved split inside each eviction bar. 17. The Already-expired bar. 18. The **Renewal overdue** figure on Renewal & Retention. 19. The tenure card (section 17), which is new. 20. **The two money lines under the Overview tiles**: rent at risk, and leaving with dues. 21. Healthy, not-set-up, not-recorded, loading, failed and Restricted states. 22. The unfinished-period marker. 23. Counts beside Journey's rates. 31. Section 18's share-of-total figure, drawn on the parked ratio-style Property Wise card; wire it into the shipped card rather than building it fresh.

**Remove:** 24. Hidden Collection leftovers in four cards (rupee rows, the "Received by" tab). 25. Hidden duplicate chips under five tiles. 26. The three-tab Lifecycle component with a Leads tab; archive it, the two-tab Journey placed on screen is the build target. 27. Parked cards outside the frame not needed here: Deposit Dues, loose copies. **The parked ratio-style Property Wise card is no longer on this list** — section 18 now requires a share-of-total figure, and that card already draws one; use it as the reference for section 18's rebuild instead of removing it (see Missing, 31).

**Decide:** 28. Tile row overflow, scroll or wrap. 29. Info icon and chevron placement; currently inconsistent, and no card has both. 30. Which cards carry their own date dropdown: the expanded Journey variant draws one, the placed card does not.

---

## Where the measured figures came from

Production, measured 2026-08-07. Platform-wide; recorded so decisions can be re-checked. The screen itself always shows one property's numbers.

| Measured | Result | What it decided |
|---|---|---|
| Active tenants | 386,943 across 26,602 properties | The base for every coverage line; the not-set-up screen is the most common state |
| Active bookings | 3,719 on 920 properties (3.5%) | Booking layers hide at zero |
| Booking confirmations | 7,194 confirmed · 403 cancelled · 27 awaiting | Both booking tiles real; awaiting hides at zero |
| Live notices | 1,313 awaiting approval · 4,482 approved | Two eviction tiles; the split is real |
| Notices raised by the tenant vs the property | 20,664 vs 99,011; only tenant-raised ever await approval | "Raised by", never "voluntary/eviction" |
| The two leaving dates | agree in 4,497 of 4,498 cases | Requested vs confirmed, two meanings, no conflict |
| Past confirmed leaving date | 472 (941 by requested date) | The tile, and why it uses the confirmed date |
| Properties with auto move-out off | 996 of 26,656 (4%) | Where stale records pile up; "check and close" |
| Key-handover marker | never set; required by 1 property | The tile cannot split "gone" from "still here" yet |
| Departures on a second marker | 47,048; sole marker on 1,005 properties | Move-outs counts both; open item 5 |
| Tenants with >1 room record | 320 of ~1.1M | Room changes cannot inflate move-ins/outs |
| Agreement end-date sources | recorded 9,067 · own duration 102,823 · property duration 43,280 · assumed 231,000 | The four-step rule; the assumption line |
| Genuinely expired (recorded/derived terms) | ~22,000, not the 188,568 the blanket assumption gives | Already-expired bar, honestly computed |
| Expiring in 30 days | 9,674 | Renewals-in-30-days appears twice, same number |
| Police verification on file | 6% of active; 88% overdue at 7 days (barely moves at 14 or 30) | Two-number card; action bar names recent joiners |
| Profile complete (ten details) | 18% | Pending is plain, not red |
| Tenant type recorded | 89,819 (23%), a lookalike field is 99% empty | Card buildable; the list-match test |
| Gender / DOB / address / food recorded | 24% / 22% / 18% / 2.6% | Coverage lines; demographics never red |
| Rent set | 84.5% of active; 97.3% of under-notice | Money figures ungated |
| Lock-in recorded | 37% of departures (1/3/6/11/12 months common) | Tenure cut points; "left early" carries coverage |
| Tenure bands | 11% · 15% · 11% · 15% · 47% | All five bands live |
| Renewal history | 14,910 records | Renewal & Retention buildable |
| Leads | 199,786 | Real volume; ninth screen, not this one |
| Where cancelled bookings go | Onto the Old Tenants list | Move-outs excludes them; the Old Tenants screen must separate them from real departures |
