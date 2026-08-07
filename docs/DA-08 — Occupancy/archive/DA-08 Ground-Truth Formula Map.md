---
title: DA-08 Occupancy — Ground-Truth Formula Map
date: 2026-06-03
tags: [rentok, occupancy, inventory, ground-truth, formula-map]
status: Living document · v0.1
owner: Sanchay
companion_to: DA-08 Brief (v0.3.1)
---

> [!INFO] What this is
> Every number on the Occupancy screen, with **one plain-English formula**, **where the number actually comes from in our data**, and **what it tells the operator.** This is the shared contract — designer draws it, engineer queries it, PM signs it — so the same number can never mean two things. It is **not** the Build Sheet (no SQL, no UI, no edge-case matrix); it's the layer of truth the Build Sheet is written against.

---

## The 6 ground rules (every formula below inherits these)

1. **A bed with a tenant who gave notice is still OCCUPIED.** He's living there and paying until his last day. We count him as occupied and *separately* flag him as "leaving soon." We never call his bed vacant. *(`tenant.status = 1` stays 1 during notice.)*
2. **The headline rate is always bed-level — in both Unit and Bed views.** Switching to the Unit view re-slices the *picture*; it never changes the headline %. *(Avoids the current bug where the Unit tab silently shows a bed-based %.)*
3. **One canonical occupancy formula.** We use the **live** count (`getOccupancy`). Two other formulas exist in the code (a monthly-history one and a stale 2025 script) — we do **not** mix them in. The "last 6 months" trend is the only place we read history, and its definition of "on notice" must be reconciled to match the live one before it ships.
4. **Occupancy can read above 100%, and we show it honestly.** Overbooked beds (more tenants than capacity) push the number past 100; disabled beds are removed from the denominator. We don't hide it — we flag the overbooking so the owner understands *why*.
5. **The ₹ loss number only shows when we have enough rents.** Loss is built from each vacant bed's own configured rent. If too many beds have no rent set, we say "set your rents to see your loss" instead of showing a number built on guesses.
6. **The money axis bends to the property.** A PG is sliced by **sharing type** (Single / Two / Three Sharing); an apartment by **unit type** (1BHK / Studio). Picked per-property from the inventory actually there. Beds are the leaf of the inventory tree — everything (rooms, units, floors) rolls up from beds, which is why bed-level is our universal denominator.

---

## Section 1 — Overview Snapshot (Now)

| Number | Plain-English formula | What it tells the operator | Source (eng) |
|---|---|---|---|
| **Rentable Units** | Every bed that can be rented = all beds minus the ones marked "not for rent". | "This is the size of my rentable inventory." | `SUM(capacity)` − disabled · `helpers.ts:1721` |
| **Occupied Units** | Beds that have a living tenant right now (including tenants on notice). | "This many are earning me money today." | tenants `status=1` · `helpers.ts:1758` |
| **Vacant Units** | Rentable beds with no tenant. = Rentable − Occupied. | "This many are empty and earning nothing." | `GREATEST(capacity − occupied, 0)` · `helpers.ts:1756` |
| **Under-Notice Units** | Occupied beds whose tenant has given a move-out date. | "These are about to go empty — act before they do." | `under_notice` / `date_of_eviction` · `Tenant.ts:207` |
| **Occupancy Rate** | Occupied ÷ Rentable × 100, at bed level. | "What share of my rentable beds is filled." | `service.ts:1836` |
| **▲/▼ vs last period** *(Q1b — gated)* | Same number for the previous equal period, shown as the change. | "Am I filling up or emptying out?" | period path `getOccupancyForPeriod` · **gated on reconciliation** |

---

## Section 2 — Occupancy Status (donut + lens)

| Number | Plain-English formula | What it tells the operator | Source (eng) |
|---|---|---|---|
| **Occupancy Rate (donut centre)** | Same as the snapshot rate — Occupied ÷ Rentable beds. Identical number; never recomputed differently. | "My one headline fullness number." | `service.ts:1836` |
| **Occupied (slice)** | Beds with a living tenant. | green / filled | `status=1` |
| **Vacant (slice)** | Rentable beds with no tenant. | the gap to fill | `capacity − occupied` |
| **Disabled for Rent (slice)** | Beds the owner switched off — not counted in the rate at all. | "Off the books on purpose." | `rent_disable` · `room.ts:57` |
| **Under Notice (chip)** | Occupied beds with a move-out date. *(slice of Occupied, not added on top.)* | "Vacancy coming." | `Tenant.ts:207,232` |
| **Booked Beds (chip)** | Beds with a confirmed future tenant who hasn't moved in yet. | "Already promised — don't re-sell." | `status=2` · `helpers.ts:1668` |
| **Under-Notice-with-Booking (chip)** | A notice bed that already has a replacement booked. | "This gap is already solved." | notice ∩ booking |
| **Bed view vs Unit view** | Bed view = count beds. Unit view = count rooms/units, and a partly-filled sharing room shows as **Semi-Vacant**. The headline % stays bed-level in both. | "Same fullness, two ways to see the gaps." | lens re-slice · `service.ts:1859` |
| **Semi-Vacant (Unit view only)** | A sharing room with some beds filled and some empty. *(Does not exist for single-occupancy units — hidden there.)* | "Rooms I can still partly fill." | `0 < active < capacity` · `helpers.ts:1762` |

---

## Section 3 — Rent yield (two numbers in the status block)

| Number | Plain-English formula | What it tells the operator | Source (eng) |
|---|---|---|---|
| **Avg Revenue per Occupied Unit** | Total configured rent of occupied beds ÷ number of occupied beds. | "What an average filled bed earns me." | `room.rent` · `room.ts:63` |
| **Avg Revenue per Available Unit** | Total configured rent of all rentable beds ÷ number of rentable beds. | "What I'd earn per bed if every rentable bed were full — my ceiling." | `room.rent` over rentable |

> *Q8 is design-driven and pending validation (see Brief §5). Listed for completeness; confirm it earns a slot before building.*

---

## Section 4 — Vacant Room Status (how long empty)

| Number | Plain-English formula | What it tells the operator | Source (eng) |
|---|---|---|---|
| **Aging bands (0–7 / 8–15 / 16–30 / 31+ days)** | Each currently-vacant bed sorted by how many days it's been empty (today − the day it went vacant). | "Fresh vacancies vs stale ones I've been ignoring." | days since last move-out · stay history |

---

## Section 5 — Upcoming Vacancy (who's leaving next)

| Number | Plain-English formula | What it tells the operator | Source (eng) |
|---|---|---|---|
| **Forecast bands (0–7 / 8–15 / 16–30 / 31+ days)** | Beds with a **confirmed** notice, sorted by how many days until the tenant's move-out date. | "My vacancy pipeline — start finding replacements now." | `date_of_eviction` forward · confirmed notices only |
| **…with replacement booked** | Of those, the ones that already have a future tenant booked. | "Already covered — skip these." | notice ∩ `status=2` |

---

## Section 6 — Where is my property losing money?

> Segment axis = **sharing type** (PG) or **unit type** (apartment), per property (Ground Rule 6).

| Number | Plain-English formula | What it tells the operator | Source (eng) |
|---|---|---|---|
| **Occupancy % (per segment)** | Occupied ÷ rentable beds **within that segment** × 100. | "Which room/unit type is emptiest." | grouped `status=1`/capacity |
| **Days to Fill (per segment)** | Average days a bed in that segment stayed empty between one tenant leaving and the next moving in. Shows "—" if there's no past turnover to measure. | "How slow this type is to re-let." | move-out→move-in deltas · stay history |
| **₹ Revenue Loss (per segment)** | Add up the monthly rent of every **vacant** rentable bed in that segment. *(Gated: shows "based on N of M beds"; if too few have rents set → "set rents to see your loss.")* | "Rupees this empty type costs me every month." | `SUM(room.rent)` over vacant · `room.ts:63` |

---

## Section 7 — Occupancy Trend (last 6 months)

| Number | Plain-English formula | What it tells the operator | Source (eng) |
|---|---|---|---|
| **Occupied vs Vacant per month** | For each of the last 6 months, how many beds were occupied vs vacant. | "Is my fullness trending up or down over time?" | history path · **definition must match live before ship** |

---

## Section 8 — Property-Wise Occupancy (multi-property)

| Number | Plain-English formula | What it tells the operator | Source (eng) |
|---|---|---|---|
| **Rank value = Occupancy Rate %** | Each property's Occupied ÷ rentable beds × 100. Ranked by this. | "Which property is emptiest — a fair comparison even across PGs and apartments." | per-property `status=1`/capacity |
| **Count = Occupied / Total (bed-level)** | "30 / 50" = 30 of 50 rentable **beds** filled. Bed-level on every row so the numbers are comparable. | "The raw size behind the %." | bed-level both sides |

---

## Drill-down (every tap lands on an existing list — we build no new list)

| Tap | Lands on | Filter |
|---|---|---|
| Vacant slice / Vacant Units | Bed list | `VACANT_BEDS (1403)` |
| Occupied slice | Bed list | `OCCUPIED_BEDS (1405)` |
| Under Notice | Bed list | `UNDER_EVICTION (1402)` |
| Semi-Vacant | Room list | `SEMI_VACANT (1410)` |
| Overbooked | Bed list | `OVERBOOKED_BEDS (1414)` |
| Disabled | Bed list | `DISABLED_FOR_RENT_BEDS (1420)` |
| Booked | Bed list | `NEW_BOOKING (1406)` |

> Source: `POST /rooms/list/filters` · `filterCodes.ts:115-136`. Each tap carries property + period + active lens.

---

## ⚑ Two items flagged for UAT / user review (not engineer-decidable)

These two are *correct by logic* but should be confirmed with real owners before lock — they're about operator preference, not data truth:

1. **Multi-property money section — property-first vs pooled segments.**
   Decision: in multi-property view, rank **properties** by ₹ loss; tap a property to see its segment breakdown. Show a **portfolio total ₹ loss** on top. We do **not** pool segments across properties **in V1**.
   *UAT to confirm:* does a 2–5 property owner think property-first (expected: yes)? *Watch:* a future **chain operator (10+ near-identical PGs)** may genuinely want "Two-Sharing is empty across all my PGs" — revisit pooling only if that segment grows.

2. **Property-Wise ranking — by rate %, not by rupees.**
   Decision: rank properties by **occupancy rate %** (the only fair cross-archetype comparison); show bed-level "30/50" as the supporting count.
   *UAT to confirm:* owners understand this section is a **health scan** (who's emptiest by proportion), while the **money section** is where they **prioritize by ₹**. *The risk:* a small property at 50% ranks "worse" than a big one at 90% that's actually losing more absolute beds/rent — confirm owners read the two sections as different jobs and aren't misled.

---

## ⏭ Pending v0.2 — the TIME dimension (next session picks this up)

> v0.1 silently assumes "Now." We are shipping a period filter (Today / This Week / This Month / Last Month / range / forecast), so every number needs a defined behavior **per time mode**. This block is the locked spec for that rewrite. **Objective for v0.2: 100% dependency coverage — a Time × Number matrix where every number has a defined behavior for every mode, or an explicit "N/A — why".**

### Core principle (anchor the whole rewrite on this)
**Occupancy is a STATE, not a SUM.** Money screens add up a period; occupancy is "full or empty at a moment." So "occupancy for last month" has no single obvious meaning — we must *define* it. (This is why the snapshot says "(Now)".)

### The 4 time modes (plain words)
- **Now (live)** — right this moment. Instant snapshot.
- **Past** — a finished period (Last Month) or a past point.
- **Range** — any custom start–end.
- **Ahead (forecast)** — what's coming, built from confirmed notices + confirmed bookings.

### Locked decisions for v0.2
1. **Past/Range "how full" = AVERAGE daily occupancy (day-weighted); Now = instant snapshot.** → **UAT-flag.**
   *Reasoning (for the flag):* snapshot (month-end) is simpler and matches "how full am I" intuition, but it **hides mid-month churn** — a bed empty 20 days then filled on the 31st reads ~100%, over-reporting health and under-reporting lost revenue. Average ("on average X% full") is **truest to revenue actually earned** and is the only definition that stays consistent with the ₹-loss number. We lean average; UAT must confirm owners reading "62% last month" intuitively hear "average," not "end of month."
2. **₹ loss by mode:** *Now* = run-rate (current vacant beds × monthly rent, "costing ₹X/month right now"); *Past/Range* = actual (each bed's vacant-days × daily rent, "vacancy actually cost me ₹X"). The past version is the truer number owners have never seen.
3. **Forecast is a real capability:** projected occupancy = today's occupied − confirmed move-outs before the target date + confirmed bookings before it. We have all three.
4. **Filter drives the WHOLE screen; default Now; the active period label is always shown and dynamic** (never a bare "(Now)" while showing the past) → **UAT-flag #2** (confirm owners never misread a past number as current). One occupancy rate on screen, always period-labeled — avoids two-rates-disagree.
5. **Intrinsic-time exceptions** (ignore the single-period filter, keep own framing): **Aging = always "as of today," Upcoming Vacancy = always Ahead, Trend = always the 6-month series.**

### Per-number time behavior (the matrix to complete in v0.2)
| Behavior class | Numbers |
|---|---|
| Now-only (past filter N/A) | Vacancy aging |
| Snapshot @ Now / average over Past·Range | Occupancy rate, Occupied, Vacant, per-segment occupancy, Property-Wise |
| Always Ahead (filter = look-ahead window) | Upcoming Vacancy, projected occupancy |
| Always historical series | 6-month Trend |
| Run-rate @ Now / actual-cost over Past·Range | ₹ Revenue Loss |

### Build-effort flag (not wiring)
The existing period path `getOccupancyForPeriod` counts *anyone present at any point in the period* (overlap) — neither snapshot nor average; it **inflates** occupancy. Locking "average" means this path is **rework, not a date-filter add**. Scope accordingly.

### New-session kickoff checklist
Read, in order: this file → `DA-08 Brief.md` (v0.3.1) → then rewrite this map to v0.2 applying the locked decisions above and completing the Time × Number matrix to 100% coverage, with the same plain-English + operator-reading style. Re-ground any new claim against `getOccupancyForPeriod` (`service.ts:1912`), `tenant_stay_history`, `tenant_eviction_details`, and `room.rent`.

---

## Changelog

| Date | Version | Change |
|------|---------|--------|
| 2026-06-03 | v0.1 | Initial formula map — 8 sections, 6 ground rules, drill-down codes, 2 UAT flags. Grounded against `getOccupancy` / `helpers.ts` / `room.ts` / `filterCodes.ts`. |
| 2026-06-03 | v0.1 + time-spec | Appended locked spec for the v0.2 time-dimension rewrite (core principle, 4 modes, 5 locked decisions, behavior matrix, build-effort flag, kickoff checklist). Rewrite to run in a fresh session. |
