---
title: DA-08 Occupancy — Build Sheet
date: 2026-06-04
tags: [rentok, occupancy, inventory, build-sheet, engineering-spec]
status: v1.2 — plain language + flat model corrected (added an optional flat-aware Unit view)
owner: Sanchay
build_leads: Jatin (Sr. BE) · Parishi · Vivek
companion_to: DA-08 Ground-Truth Formula Map (v0.3.4) · DA-08 Brief (v0.3.2)
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



> [!INFO] What this is
> The engineering spec for **DA-08 Occupancy** — the first screen of the Inventory tab. Every fact and `file:line` here is carried from the [[DA-08 Ground-Truth Formula Map v0.2|Formula Map]] (v0.3.4 inside); don't re-derive it, just build. This screen *diagnoses* vacancy — it never edits beds, rooms, or rent (that's Bed Management). Every drill-down hands off to the existing `POST /rooms/list/filters` — **build no new list.**
>
> **Build order:** **A. Data Foundation** (build this first — everything else depends on it) → **C. Tasks** → **D. Two open decisions for kickoff.**

---

## Two rules every count obeys

Carried from the Formula Map (Rules 2 and 6). Both apply to every number on this screen:

1. **The headline rate is always counted per bed** — in both the Bed and Unit views. The view re-groups the same numbers; it never changes the headline %.
2. **Every total adds up the rentable units, never a room and its beds both.** A rentable unit is one **bed** (or a single-bed room like a studio). A BHK flat is **several rooms, each with beds** — counted as those beds, not as one flat. `unit_type='BED'` rows copy the room's rent down — add the room *and* its beds and you count the money twice.

---

## A. Data Foundation — the nightly snapshot table (build this first)

Build this first. The period numbers, the trend, and property-wise ranking all read from it.

**Why it has to exist.** "How full am I" is a state at a moment, not a thing you add up over time. So "how full was I last month" has no single answer until we pick one. We pick the **average fullness across the days**: add up the filled bed-days, then divide by the bed-days that could be rented. That's how hotels measure it. The current period code (`getOccupancyForPeriod`) can't give us that — it counts anyone who was there at *any* point in the period, so it counts the same bed more than once. It's neither a single-day reading nor an average, and it can't be patched into one.

The catch is the past. We *can* rebuild **how many beds were filled** each past day from `tenant_stay_history`. We **can't** rebuild **how many beds were rentable** each past day — `room` records when it was created but not when it changed, and there's no history table, so we can't ask "was this bed rentable on a past date?" The fix is what every property system does: **add up each day's occupancy once a night, from now on.**

> ⚠ **Flag before building:** this is a new table + a nightly cron — a schema change. Per our rules, confirm the table name, the segment-key format, and where nightly jobs run today **with Jatin** before the migration lands. Don't auto-apply.

### A1. The table — one row per `property × segment × day`

```
occupancy_daily_snapshot
  property_id
  segment            -- unit_type, plus sharing_type for multi-bed Rooms (see Task 5)
  day
  rentable           -- rentable units that day  (capacity − disabled)
  occupied           -- units with a living tenant that day  (status=1)
  disabled           -- units switched off that day  (rent_disable)
  overbooked         -- true-overbooking count that day  (see Task 7)
  PRIMARY KEY (property_id, segment, day)
```

This is **not** a full history of every room change — the screen only needs each day's totals, never a per-room log. With this table, every period number is a simple `AVG()` over rows, the old over-counting code is dropped (not patched), and "how many beds were rentable" is exact from launch day on.

### A2. Pre-launch history — show it, labelled approximate *(locked)*

We *can* rebuild the filled-beds count, so the 6-month trend and "last month" views can be filled in at launch from `tenant_stay_history` (real filled beds) over an estimated rentable count (today's config). Show full history on day one, with a quiet **"estimated before \<launch date\>"** marker. It corrects itself to exact as the nightly rows pile up. The marker is a must — we never fill in past numbers silently.

### A3. The two measures change at different rates *(resolved)*

| Measure | How to store it | Why |
|---|---|---|
| **Filled beds per day** | The daily snapshot above | Changes every day, so store it every day. This is the must-have — without it, period numbers aren't honest. |
| **Rentable beds per day** | A small `room_state_history` table *(new, can be cut if time runs short — see Task 4)* | One row written only when `rent_disable` / `capacity` / `sharing_type` changes. Nearly free, because PG inventory rarely changes — and it gives an exact rentable count for any past date. |

Do **not** build a full change-log of every field — it answers questions this screen never asks. The daily snapshot is the must-have; the small history table can be cut against v1 (Section D).

---

## B. Two open questions to settle during build

Both change the math. Settle them with code in front of you, not by guessing.

### B1. `property.new_property_structure` — a real either/or to handle

This true/false flag changes core math two ways. Handle both everywhere. Treat the new-structure definitions as the correct ones; make old-structure properties match them.

| What it changes | `new_property_structure = true` (the correct one) | `= false` (make it match) |
|---|---|---|
| **Capacity** | count of child `BED` rows | derived from `sharing_type` |
| **Under-notice group** | `status=1 AND date_of_eviction` | `status IN (1,2) AND date_of_eviction` (wrongly counts bookings as under-notice) |

### B2. Room soft-delete — confirm during build

The rentable count includes only rentable units, so we need to know how a deleted room shows up. Does `room` carry a soft-delete flag (`is_active` / `deleted_at`), and does the live `getOccupancy` path already leave it out? If not, deleted rooms pad the rentable count and read as vacant forever. Verify against `getOccupancy` (`service.ts:1836`) before locking the `rentable` definition, and report back. *(Carried open from the Formula Map's v0.3.3 re-confirm note.)*

---

## C. Tasks

One row per task — pick one up and own it end to end. **"Why" is the reason in one line; all `file:line` anchors live in the "Files / queries" column.**

### Group 1 — Data Foundation *(build first)*

| # | Task | Why | Files / queries | Acceptance check |
|---|---|---|---|---|
| **1** | Build `occupancy_daily_snapshot` + a nightly cron writing one row per `property × segment × day`: `{rentable, occupied, disabled, overbooked}`. | The foundation — makes every period number a simple `AVG()` and the rentable count exact going forward. | New migration in `src/migrations/`; cron reuses `getOccupancy` (`service.ts:1836`) and `rent_disable` (`room.ts:57`); segment = Task 5 axis. | One row per property×segment for the run date; `disabled` matches the live disabled count; re-running the same day overwrites cleanly (no dupes). Occupied *may* exceed rentable when rooms are over-occupied (`status=1` > capacity) — that's expected, not a bug. |
| **2** | Replace the overlap-counting period path with **average daily occupancy** = `AVG()` over snapshot rows, for Past and Range. | The old path counts the same bed more than once, so it inflates; it can't be patched into an average. *(Average-vs-snapshot is UAT-flagged — confirm with owners before launch; see Formula Map UAT flag #1.)* | Retire / rewrite `getOccupancyForPeriod` (`service.ts:1912`); read the Task 1 table. | For a finished month, the rate equals the mean of that month's daily `occupied/rentable`. A bed empty most of the month and filled only at month-end reads roughly its true fraction (low %), not ~100% as a month-end snapshot would. |
| **3** | Backfill pre-launch history from `tenant_stay_history`, tagged with an **"estimated before \<launch\>"** marker. | We can rebuild the filled-beds count, so we can show history on day one without lying — and it fixes itself as real rows land. | `tenant_stay_history` (`started_at` / `ended_at` / `bed_id`); marker on the snapshot rows or the API response. | Pre-launch months carry the marker in the payload; post-launch months don't. |
| **4** | *(Can be cut)* `room_state_history` — an update-date on `room` + a row on every `rent_disable` / `capacity` / `sharing_type` change. | Gives an exact rentable count for any past date, and exact pre-launch history if we ever drop the estimated marker. Nearly free. | `room` entity; new effective-dated table. | Toggling `rent_disable` writes exactly one new row; querying rentable as-of a past date returns the config that was live then. |

### Group 2 — Money axis & ₹-loss

| # | Task | Why | Files / queries | Acceptance check |
|---|---|---|---|---|
| **5** | Build the **two-level money axis**: group by `unit_type` (Rooms / Studio / 1BHK …), then split by `sharing_type` *only under the multi-bed Rooms group*. Whole-units get no sub-split. | One property can hold sharing Rooms, Studios, and BHK units at once — the old "PG or apartment" split was just the simple case. A BHK flat's rooms all carry the BHK `type`, so they land in the same group and the bed-sum is correct without flat-awareness. | `property.unit_types_available[]` (`property.ts:412-416`); `unit_type ∈ {ROOM, 1RK, STUDIO, 1BHK…5BHK, BED}`; `sharing_type` (`room.ts:59`) via `getSharingTypeText()` (`commonFunctions.ts:1976`). | Pure-PG → one Rooms group sliced by sharing type; pure-apartment → unit-type groups, no sub-split; mixed → Rooms-with-sharing-rows beside Studio/BHK rows. |
| **6** | Build the **rent-loss query**: monthly loss = `SUM(rent)` over vacant units in the segment. Only show the loss when enough rents are set: "based on N of M units", and below the threshold show "set your rents to see your loss". | Bed rows copy the room's rent down, so adding room + beds counts the money twice. This keeps a guessed number off a screen owners must trust. | `room.rent` (`room.ts:63`); vacant units from Task 1 or live. Optional: price an empty bed by its last tenant's `tenant_stay_history.rent_amount` before falling back to the sharing-type average. | On a mixed property, total loss adds up vacant units only (hand-check: no double-count). Below the coverage threshold, the rupee number is replaced by the "set rents" copy. |

### Group 3 — Bookings & overbooking

| # | Task | Why | Files / queries | Acceptance check |
|---|---|---|---|---|
| **7** | Build the **real-overbooking** query: ≥2 *confirmed* bookings with *overlapping* stay dates on one bed. This refines the existing `new_bed_status=4` (which is count-only — no date or confirmed check). | A real clash is two tenants arriving for one bed. Today, back-to-back or rejected bookings get wrongly flagged. Also recommend an upstream fix: warn or block on a real overlap at booking *creation*. | `new_bed_status=4` enum (`rooms.ts:6695`; set at `rooms.ts:1560,3389,5664`, `roomRecommender.ts:463`); `tenant_booking_confirmation.is_confirmed`. | Two non-overlapping confirmed bookings on one bed → not flagged; two overlapping confirmed bookings → flagged; a rejected booking never contributes. |
| **8** | Filter every "booked = held bed" count to **confirmed only** (`is_confirmed=1`, or `auto_accept_booking` with no row). Exclude rejected (`-1`) always; show pending (`0`) as a separate "N requests awaiting approval". | Today's "booked" never checks `is_confirmed`, so it's inflated by rejected + pending. A rejected booking holding a bed is a bug. | `tenant_booking_confirmation.is_confirmed`; `NEW_BOOKING` (1406) / `status=2` flag (`helpers.ts:1668`). | "Booked" drops to confirmed-only; a known rejected booking no longer counts; pending shows as its own "awaiting approval" number, never inside occupancy. |

### Group 4 — Correctness pushbacks

We have authority to fix wrong definitions, not inherit them.

| # | Task | Why | Files / queries | Acceptance check |
|---|---|---|---|---|
| **9** | Rename **"Overbooked" → "Over-occupied"** in the UI now; file a code-rename of the constant for later. | `OVERBOOKED_BEDS` measures living tenants over capacity — that's over-occupancy, and the wrong word already reaches owners. | `OVERBOOKED_BEDS` (filter 1414); occupied-over-capacity (`helpers.ts:1769`). | UI reads "Over-occupied beds"; the >100% explanation uses "over-occupied", never "overbooked"; rename ticket filed. |
| **10** | Unify **"vacant" on `status=1`** across both views; show "booked" as a separate *arriving* marker, not a filled bed. | Today bed-level vacancy uses `status=1` and room-level uses `status IN (1,2)`, so the same booked-empty bed reads two different ways. That's the exact contradiction this screen exists to kill. | `VACANT_BEDS` (1403) vs `VACANT` (1408); `SEMI_VACANT` (1410). | A booked-empty bed reads vacant in *both* Bed and Unit views; "booked" shows as an arriving flag, not folded into the vacant slice. |
| **11** | Make **`status=1 AND date_of_eviction`** the one under-notice definition everywhere; make old-structure properties match it. | A booking isn't "under notice" — only a living tenant can be leaving. The real signal is `tenant.date_of_eviction`. | `tenant.ts:229`; fork on `new_property_structure` (B1). | Under-notice excludes all `status=2`; old- and new-structure properties return the same set of beds for a fixed dataset. |
| **12** | *(New — flag only, not in V1)* An **out-of-service** bed state for beds temporarily off-market (e.g. under repair). | No such state exists today, so a bed under repair sits in the rentable count and reads vacant. This is a feature, not a wiring fix — the Brief scopes it out this cycle. | None today; would mirror how `rent_disable` is left out of the rentable count, shown as its own slice. | Documented as a product gap with a rough size; **not built** in v1 unless Section D pulls it in. |

### Group 5 — Per-section wiring *(mostly assembly over Groups 1–4)*

| # | Task | Why | Files / queries | Acceptance check |
|---|---|---|---|---|
| **13** | **Q1 / Q2:** the five counts (rentable / occupied / vacant / under-notice) + the bed-level rate, with the existing drill-downs. When the rate exceeds 100%, the headline must say *why* (over-occupancy). | The core: what state each bed is in, then where it is. These are cheap (live + existing filters). A >100% number with no explanation is the fastest way to lose trust. | `getOccupancy` (`service.ts:1836`); drill-downs via `POST /rooms/list/filters` (`filterCodes.ts:115-136`). | Five counts match live; rate is bed-level in both lenses; a >100% property shows the over-occupancy caveat inline. |
| **14** | **Q3 Vacancy Aging** (0–7 / 8–15 / 16–30 / 31+ days), always as-of-today, with never-rented beds in their own bucket. | Aging only makes sense relative to now. A new property's just-configured beds must not read as a wall of stale red. | Days since last move-out from `tenant_stay_history`. | Bands are computed from today; a never-occupied bed lands in "never rented", not 31+; a past-period filter does not move the bands. |
| **15** | **Q4 Upcoming Vacancy:** confirmed notices by days-to-free, split by whether a *confirmed* replacement is booked. Show Q1 under-notice, Q4, and the Q9 flag as the same beds, grouped different ways. | The forward view — who's leaving and who's coming, confirmed only (no lead guesses). All three views are the same beds — show it, or the owner can't tell. | `tenant.date_of_eviction` forward; `date_of_joining` (`tenant.ts:216`); confirmed via Task 8. | A pending replacement does not mark a gap "covered"; the under-notice number in Q1, Q4, and Q9 all trace to the same beds. |
| **16** | **Q9: three separate flags** — booked-not-arrived · over-occupied (Task 9) · overbooked (Task 7) — each shown only when non-zero. | These used to be one muddled flag. They're different problems with different fixes (reassign vs resolve-the-clash). | Tasks 7, 8, 9. | Three distinct chips; over-occupied and overbooked never collapse into one number. |
| **17** | *(Stretch)* **Q6 Trend** + **Q7 Property-Wise** + **Q8 yield**, all `AVG()` over the Task 1 snapshot. Rank Q7 by rate %. | The trend ships empty today (`trend_text=null`); all three just read the snapshot. Cut order if time runs short: Q6, then Q7, then Q8 (Brief §5). | Snapshot table (Task 1); trend null at `service.ts:1860`. | A trend month uses the same average-daily definition as the period filter — hovering "May = 62%" matches Last-Month = May. Q7 ranks by rate %, with the count as support. |
| **18** | *(An add-on — extra, ship small and watch)* **Flat-aware Unit view:** in the Unit view only, group a BHK flat's rooms into one unit and show it as fully-rented / part-empty (e.g. "2BHK-101: 1 room empty"). The headline rate and totals stay per-bed. | Owners think of a 2BHK as one thing. The grouping already works in the room-list, so we expect it carries over. **The screen must work without this** — it's an add-on, never required. | Reuse the room-list flat grouping (`helpers.ts:1271`, by `room.name` prefix); on for BHK types + the 19 `FLAT_GROUP_PG_IDS` (+1 combo) in `roomConstants.ts` — full list in the [[DA-08 Flat-Group Properties (tracking)]] note. | With the bet ON: a fully-rented 2BHK reads as one rented unit; a part-empty one shows which room is open. With it OFF: the screen still works, counting beds. The bed-level headline % is identical either way. |

**Two wiring rules that span the tasks above:**

- **Forward chips are hidden in the past.** Booked, Overbooked, and Under-Notice-with-booking describe the future — on a "Last Month" view they're noise, so hide them there. Aging is always as-of-today; Trend and Days-to-Fill are always their own historical series. The period filter drives everything else, and every tile shows a small time badge for its context.
- **Days-to-Fill, if built (Q5):** (a) count still-open vacancies at their current age and always print the open tail ("avg 14d · 2 beds still open, longest 86d") — otherwise the worst beds hide; (b) measure a gap as next `started_at` − prior `ended_at` on the same `bed_id`, but filter on `end_event_type` so a `ROOM_CHANGE` / `RENT_CHANGE` mid-stay doesn't read as a fake turnover.

---

## D. Two decisions for kickoff *(left open on purpose)*

### D1. If the cycle tightens, what's the cut order for the new foundation pieces?

The daily snapshot (Task 1) is the must-have — not cuttable. These three *can* be cut, and need ranking **only if time forces it**. This is just a list, not a recommendation:

- **Exact past rentable count (Task 4)** — an exact rentable count for any past date; the snapshot already makes the screen honest going forward, so this is the easiest to defer.
- **Out-of-service feature (Task 12)** — a product feature, not wiring; the Brief already scopes it out this cycle.
- **Vacancy unification (Task 10)** — a definition fix; low build cost, but it touches both lenses.

Rank these with the team once the cycle length is known.

### D2. Does this format work?

This sheet uses one task table (*task · why · files/queries · acceptance check*) with a Data Foundation section up top. Confirm it reads cleanly for Jatin, Parishi, and Vivek — or adjust the grouping/columns before kickoff.

---

## Where the facts live

- **Every verified fact, `file:line`, and the correctness pushbacks:** [[DA-08 Ground-Truth Formula Map v0.2|Formula Map]] (v0.3.4)
- **The why, the personas, and the cut-order:** [[DA-08 Brief]] (v0.3.2)
- **Drill-downs:** `POST /rooms/list/filters` · `filterCodes.ts:115-136` — build no new list.
