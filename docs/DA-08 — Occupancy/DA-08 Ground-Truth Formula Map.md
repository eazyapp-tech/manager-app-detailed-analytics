---
title: DA-08 Occupancy — What Every Number Means (plain)
date: 2026-06-04
tags: [rentok, occupancy, inventory, formula-map, plain]
status: Living document · v0.4
owner: Sanchay
companion_to: DA-08 Brief · DA-08 Build Sheet
---

> [!WARNING] Superseded on 2026-08-07 by [[DA-08 Inventory — Handoff Sheet]]
> This document is kept for its reasoning and its code references. Where it disagrees with the handoff sheet, **the handoff sheet wins.** It overrides this document on the following points, each decided against production data or by the owner:
>
> 1. **A booked-empty bed is not vacant.** This doc standardises vacancy on the physical basis and renders booked as a separate flag; the handoff counts a booked bed as unsellable and therefore not vacant, with Booked as its own donut slice.
> 2. **No separate over-booked chip.** Two confirmed bookings on one bed measured five cases across 36,000 identifiable beds; it folds into over-occupied. A guard at booking creation is still wanted, and is logged as an open item.
> 3. **The money card groups by sharing type only**, not unit type with a sharing sub-split. 85% of live rooms are typed plainly as "room" and the field is uncontrolled free text.
> 4. **Vacancy age and upcoming vacancy are counted per room** and ignore the view toggle, because bed identity is absent on unmigrated properties and missing from most stay history even on migrated ones.
> 5. **Never rented is its own bucket**, with a separate "unknown" bar. It is 87–90% of vacant inventory, not an edge case.
> 6. **Drill-downs go to the rooms, tenants and bookings lists**, checked filter by filter. Vacancy-age and never-rented drills are **not reachable today** and are logged as a prerequisite.
> 7. **The view toggle is Bed View / Unit View**, and a unit is defined by how the space is let, not by a hardcoded property list.
> 8. **The occupancy rate is bed-based in both views**, and labelled as such.
> 9. **Property Wise ranks by empty units**, not by occupancy rate.
> 10. **The filter is Now · This Month · Last Month · Current FY · Custom · Coming up.** There is no All Time, and the word "Live" is not used.
> 11. **Disabled for Rent is the maintenance state.** This doc states no out-of-service state exists; the owner confirms the existing flag serves that purpose, so there is no separate maintenance number.
>
> **Stale on fact, do not build from these:** the proposed nightly snapshot table and room-state history are **not required** — a corrected query over existing stay history serves the period numbers. The yearly-stats "highest month" script described here as offline **is** wired in, and counts evicted tenants. The differing tenant-status usage across views described here as a bug is **deliberate**.


	₹
> **Who this is for.** Anyone on the team — engineer, designer, CPO, EM — can read this once and understand every number on the Occupancy screen, what it means, and why the owner needs it.
>
> No code, no table names, no how-to-build detail — that's in the **Build Sheet**. The **Brief** says *why* we're building it. This says *what each number is*.

---

## The screen in one line

A PG owner can see *that* he has empty beds, but not which ones, how long they've been empty, who's leaving next, or what the empty beds cost him. This screen shows all of that in one place — and shows what the empty beds are costing him in rupees.

---

## Things that are always true

These apply to every number below. Read once.

1. **A tenant on notice still fills his bed.** He's living there and paying until his last day. We show "leaving soon" separately, but his bed is not empty yet.
2. **The big % at the top is always counted by beds** — in both the Bed view and the Room view. Switching views changes the breakdown below, never the headline %.
3. **One way to count "how full."** We never mix two methods on the same screen.
4. **The % can go above 100%.** That happens when more people are actually living in a room than it has beds (we call this *over-occupied*). We show why — we never hide it.
5. **The "money lost" number only shows when enough beds have a rent set.** If too few do, we say "set your rents to see your loss" instead of guessing.
6. **Money is grouped by the kind of room you have** (see the money section).
7. **One time filter at the top sets the period for the whole screen.** You can also change a single card on its own. And a few cards always show their own time — they say so on the card.

---

## How time works

This is the one idea worth getting right. The owner can look at **now**, a **past period**, or **what's coming up**:

- **Now** — right this second.
- **Last month / a date range** — the average across all those days, not how it looked on the last day. *Example: a bed empty 20 days, then filled on the 31st, wasn't full all month — so it shouldn't read 100%. The average is honest.*
- **Coming up** — a best guess of what's ahead, built only from confirmed move-outs and confirmed bookings.
- **A few cards ignore the filter and say so:** "how long empty" always counts up to today; "who's leaving next" always looks ahead; the trend always shows the last 6 months.

---

## The numbers, section by section

Each one: what it is, and why the owner cares.

### 1. Top summary
- **Rentable beds** — all the beds you can rent (minus the ones switched off). *The most you could ever rent out.*
- **Occupied** — beds with someone living in them (including people on notice). *These earn money today.*
- **Vacant** — rentable beds with nobody in them. *These earn nothing.*
- **Leaving soon** — occupied beds whose tenant has given a move-out date. *Act before they go empty.*
- **How full (%)** — of your rentable beds, how many are filled. The one headline number.
- **Up or down vs last period** — the same % for the period before, shown as a change. *Filling up or emptying out?* (Shows only once we trust the past number — see fixes.)

### 2. The donut (the fullness picture)
- **Occupied / Vacant / Switched-off** — the split of your beds. Switched-off beds aren't counted in the %.
- **Bed view vs Room view** — count by bed, or count by room. A part-filled sharing room shows as half-full in Room view. The headline % stays bed-based in both.
- **Chips on top** (each shown only when it applies):
  - **Leaving soon** — occupied beds with a move-out date.
  - **Booked** — someone's booked this bed but hasn't moved in (an empty bed, or one the current tenant is leaving). *Don't re-sell it.*
  - **Booked + leaving soon** — a leaving bed that already has a replacement lined up. *This gap is handled.*
  - **Over-occupied** — more people living in a room than it has beds. *This is what pushes the % above 100% — fix it.*
  - **Overbooked** — the same bed promised to two arriving people at overlapping times. *A mistake to fix before they show up.*

### 3. Rent per bed *(maybe — still being confirmed)*
- **Average rent of a filled bed** and **average rent if every bed were full.** *Are my filled beds underpriced, and what's the most I could earn if full?* (Lowest priority; confirm it's worth a spot before building.)

### 4. How long beds have been empty
- **Empty for 0–7 / 8–15 / 16–30 / 31+ days** — every empty bed sorted by how long it's sat. *Fresh empties vs stale ones I've ignored.*
- **Never rented** beds (just set up, never filled) show separately — they're not stale, they're new.

### 5. Who's leaving next
- **Freeing up in 0–7 / 8–15 / 16–30 / 31+ days** — beds with a confirmed move-out date, sorted by when they free up. *Start finding replacements now.*
- **…already covered** — the ones that already have a confirmed replacement booked. *Skip these.*

### 6. Where am I losing money
Grouped by the kind of unit you have (see the money breakdown below). For each kind:
- **How full it is** — *which kind of room is emptiest.*
- **How long it takes to fill** — average days a bed of this kind sits empty between tenants. Shows "—" if there's no history yet, and always shows any beds still empty right now so the worst ones aren't hidden.
- **Rupees lost each month** — the rent of every empty bed of this kind, added up. *Now:* what they're costing you per month right now. *Past:* what the empty beds actually cost you over that period — a number owners have never seen.

### 7. Up or down over time (last 6 months)
- **Filled vs empty each month** — *is my fullness going up or down over time?*

### 8. All my properties (for multi-property owners)
- **Each property's % and "30 / 50" count, ranked** — *which property is dragging me down.* Ranked by the %. (We're checking with owners that ranking by % reads fairly — a small property at 50% ranks below a big one at 90% that may be losing more rent.)

### Tapping any number
Tap any number to open exactly those beds as a list. The count always matches what you tapped.

---

## The money breakdown — how it groups

A single property can have several kinds of inventory at once — sharing Rooms (with beds inside), Studios, 1BHK, 2BHK. So the breakdown is two levels:

1. **First, by kind of unit:** Rooms · Studio · 1BHK · 2BHK …
2. **Then, under sharing Rooms only, by sharing type:** Single · Double · Four-sharing. (Studios and flats don't split further.)

Example for one mixed property:

| Kind | Split | Empty | Lost / month |
|---|---|---|---|
| Rooms | Four-sharing | 3 beds | ₹15,000 |
| Rooms | Double | 1 bed | ₹6,000 |
| Studio | — | 2 units | ₹16,000 |
| 2BHK | — | 1 unit | ₹15,000 |

A pure-PG property shows one "Rooms" group split by sharing type. A pure-apartment property shows just the unit kinds. A mixed one shows both, side by side.

**One thing about flats:** a 2BHK or 3BHK is usually **several rooms under one flat name** (like "101"), each with its own beds — not a single room. The empty-bed math stays the same: we add up the rent of every empty bed. Showing a whole flat as **one unit** in the Room view is something we're trying, because that grouping already works elsewhere in the app. It's optional — the screen works fine without it. (See "Still asking owners".)

---

## Three things people mix up — keep them separate

| Word | What it really means | The owner's action |
|---|---|---|
| **Booked** | Someone booked the bed but hasn't moved in. Only a confirmed booking counts. | Don't re-sell it |
| **Over-occupied** | More people *living* in a room than it has beds. This is what pushes the % above 100%. | Reassign people |
| **Overbooked** | The same bed promised to two arriving people at overlapping times. | Fix the clash before they arrive |

---

## What's decided

- **"Last month" = the average across the month**, not how it looked on the last day. (Truer, and matches the money-lost number.)
- **"Booked" means confirmed only.** A pending request shows separately as "awaiting your approval" — it's not a held bed.
- **We'll show past months from day one, marked "estimate"** until we've recorded enough daily data, then it becomes exact.
- **Money lost is grouped by kind of unit, then sharing type** (above).
- **Multi-property:** rank properties by their fullness %; show a total rupees-lost across all of them.

## Still asking owners

- Does "62% last month" naturally read as *an average*?
- Do they read the "who's emptiest" section and the "what's it costing" section as two different jobs?
- Does ranking properties by % (not by rupees lost) feel fair to them?
- Should a whole flat (a 2BHK made of several rooms) show as **one unit** here, the way it does in the room list? We think yes — but it's something we're trying, and we'd like your input. The screen still works if we don't do it.

## Bugs to fix in the build

(Plain summary — full detail is in the Build Sheet.)

- The word **"overbooked"** is used for two different things in the system today. Rename one to **"over-occupied"** so the screen shows the right word.
- The **"booked" count** today includes cancelled and not-yet-approved bookings — it should count confirmed only.
- **"Leaving soon"** should count only people actually living there who gave notice — not future bookings. Make this consistent.
- **Beds under repair can't be marked "out of service" today**, so they look empty and drag the number down. Add a way to mark them — they shouldn't count as rentable while being fixed.
- **"Vacant"** is counted two slightly different ways in the Bed view vs the Room view — make them one.
- There's already a rough **"two people on one bed"** check, but it's too loose (it flags beds that aren't really clashing). Tighten it to catch real clashes, and warn at booking time, not after.

---

## Changelog

| Date | Version | Change |
|------|---------|--------|
| 2026-06-04 | v0.4 | Plain-language rewrite of the detailed map (v0.3.4). Removed all code references, table names, and jargon; cut length roughly in half. Kept every number's meaning and the key decisions; technical detail moved to the Build Sheet. The detailed, verified version stays as `DA-08 Ground-Truth Formula Map v0.2.md` for engineering reference. |
