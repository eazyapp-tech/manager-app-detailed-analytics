---
title: DA-08 Occupancy — Build Sheet
date: 2026-06-04
tags: [rentok, occupancy, inventory, build-sheet, engineering-spec]
status: v1.0 — ready for build planning
owner: Sanchay
build_leads: Jatin (Sr. BE) · Parishi · Vivek
companion_to: DA-08 Ground-Truth Formula Map (v0.3.4) · DA-08 Brief (v0.3.2)
---

> [!INFO] What this is
> The engineering spec for **DA-08 Occupancy** — the first screen of the Inventory tab. Every fact and `file:line` here is carried from the [[DA-08 Ground-Truth Formula Map v0.2|Formula Map]] (v0.3.4 inside); don't re-derive it, just build. This screen *diagnoses* vacancy — it never edits beds, rooms, or rent (that's Bed Management). Every drill-down hands off to the existing `POST /rooms/list/filters` — **build no new list.**
>
> **Build order:** **A. Data Foundation** (the keystone — build this first) → **C. Tasks** → **D. Two open decisions for kickoff.**

---

## Two rules every count obeys

Carried from the Formula Map (Rules 2 and 6). Every number on this screen inherits both:

1. **The headline rate is always bed-level** — in both Bed and Unit views. The lens re-slices the picture; it never changes the headline %.
2. **Every rollup sums over leaves, never parent + child.** A leaf is a bed under a sharing room, *or* a childless whole-unit room. `unit_type='BED'` rows copy the room's rent down — sum the room *and* its beds and you double-count.

---

## A. Data Foundation — the nightly snapshot table (keystone)

Build this first. The period numbers, the trend, and property-wise ranking all read from it.

**Why it has to exist.** Occupancy is a state, not a sum. "How full was I last month" has no single meaning, so we define it as **average daily occupancy** — room-nights occupied ÷ room-nights available, the hotel/PMS standard. The existing period path `getOccupancyForPeriod` can't give us that: it counts anyone present at *any* point in the period (overlap-counting), which inflates the number and is neither a snapshot nor an average. It can't be patched into one.

The blocker is history. The numerator (occupied beds per day) is reconstructable from `tenant_stay_history`. The denominator (rentable beds per day) is **not** — `room` has a create-date but no update-date and no audit table, so we can't ask "was this bed rentable on a past date?" The fix is what every PMS does: **roll occupancy up nightly, going forward.**

> ⚠ **Flag before building:** this is a new table + a nightly cron — a schema change. Per our rules, confirm the table name, the segment-key encoding, and where nightly jobs run today **with Jatin** before the migration lands. Don't auto-apply.

### A1. The table — one row per `property × segment × day`

```
occupancy_daily_snapshot
  property_id
  segment            -- unit_type, plus sharing_type for multi-bed Rooms (see Task 5)
  day
  rentable           -- rentable leaves that day  (capacity − disabled)
  occupied           -- leaves with a living tenant that day  (status=1)
  disabled           -- leaves switched off that day  (rent_disable)
  overbooked         -- true-overbooking count that day  (see Task 7)
  PRIMARY KEY (property_id, segment, day)
```

This is **not** a per-room audit log — the screen only needs per-day aggregates, never per-room history. With this table, every period number is a plain `AVG()` over rows, the overlap-counting path is retired (not patched), and the denominator is exact from launch day forward.

### A2. Pre-launch history — show it, labelled approximate *(locked)*

The numerator *is* reconstructable, so the 6-month trend and "last month" views can be backfilled at launch from `tenant_stay_history` (real occupied) over an approximate denominator (today's rentable config). Show full history on day one, with a quiet **"estimated before \<launch date\>"** marker. It self-corrects to exact as nightly rows accumulate. The marker is mandatory — we never backfill silently.

### A3. The two measures change at different rates *(resolved)*

| Measure | How to store it | Why |
|---|---|---|
| **Occupied per day** | The daily snapshot above | Changes daily, so a daily grain is right. This is the floor — without it, period occupancy isn't truthful. |
| **Rentable per day** | Effective-dated `room_state_history` *(net-new, cut-order-able — see Task 4)* | A row written only when `rent_disable` / `capacity` / `sharing_type` changes. Nearly free, because PG inventory rarely changes — and it buys an exact, forensic denominator. |

Do **not** build full event-sourcing or per-field audit — it answers questions this screen never asks. The daily snapshot is the floor; the effective-dated layer is cut-order-able against v1 (Section D).

---

## B. Two forks to settle during build

Both change the math. Settle them with code in front of you, not by guessing.

### B1. `property.new_property_structure` — a first-class fork

This boolean forks core math two ways. Branch on it everywhere. The new-structure definitions are canonical; old-structure properties reconcile to them.

| What it forks | `new_property_structure = true` (canonical) | `= false` (reconcile to canonical) |
|---|---|---|
| **Capacity** | count of child `BED` rows | derived from `sharing_type` |
| **Under-notice cohort** | `status=1 AND date_of_eviction` | `status IN (1,2) AND date_of_eviction` (wrongly counts bookings as under-notice) |

### B2. Room soft-delete — confirm during build

The denominator counts rentable leaves, so we need to know how a deleted room is represented. Does `room` carry a soft-delete flag (`is_active` / `deleted_at`), and does the live `getOccupancy` path already exclude it? If not, deleted rooms inflate the denominator and read as vacant forever. Verify against `getOccupancy` (`service.ts:1836`) before locking the `rentable` definition, and report back. *(Carried open from the Formula Map's v0.3.3 re-confirm note.)*

---

## C. Tasks

One row per task — pick one up and own it end to end. **"Why" is the reason in one line; all `file:line` anchors live in the "Files / queries" column.**

### Group 1 — Data Foundation *(build first)*

| # | Task | Why | Files / queries | Acceptance check |
|---|---|---|---|---|
| **1** | Build `occupancy_daily_snapshot` + a nightly cron writing one row per `property × segment × day`: `{rentable, occupied, disabled, overbooked}`. | The keystone — makes every period number a plain `AVG()` and the denominator exact going forward. | New migration in `src/migrations/`; cron reuses `getOccupancy` (`service.ts:1836`) and `rent_disable` (`room.ts:57`); segment = Task 5 axis. | One row per property×segment for the run date; `disabled` matches the live disabled count; re-running the same day overwrites cleanly (no dupes). Occupied *may* exceed rentable when rooms are over-occupied (`status=1` > capacity) — that's expected, not a bug. |
| **2** | Replace the overlap-counting period path with **average daily occupancy** = `AVG()` over snapshot rows, for Past and Range. | The old path overlap-counts and inflates; it can't be patched into an average. *(Average-vs-snapshot is UAT-flagged — confirm with owners before launch; see Formula Map UAT flag #1.)* | Retire / rewrite `getOccupancyForPeriod` (`service.ts:1912`); read the Task 1 table. | For a finished month, the rate equals the mean of that month's daily `occupied/rentable`. A bed empty most of the month and filled only at month-end reads roughly its true fraction (low %), not ~100% as a month-end snapshot would. |
| **3** | Backfill pre-launch history from `tenant_stay_history`, tagged with an **"estimated before \<launch\>"** marker. | The numerator is reconstructable, so we can show history on day one without lying — and it self-corrects as real rows land. | `tenant_stay_history` (`started_at` / `ended_at` / `bed_id`); marker on the snapshot rows or the API response. | Pre-launch months carry the marker in the payload; post-launch months don't. |
| **4** | *(Cut-order-able)* Effective-dated `room_state_history` — an update-date on `room` + a row on every `rent_disable` / `capacity` / `sharing_type` change. | Gives an exact, forensic denominator and exact pre-launch backfill if we ever drop the estimated marker. Nearly free. | `room` entity; new effective-dated table. | Toggling `rent_disable` writes exactly one new row; querying rentable as-of a past date returns the config that was live then. |

### Group 2 — Money axis & ₹-loss

| # | Task | Why | Files / queries | Acceptance check |
|---|---|---|---|---|
| **5** | Build the **two-level money axis**: group by `unit_type` (Rooms / Studio / 1BHK …), then split by `sharing_type` *only under the multi-bed Rooms group*. Whole-units get no sub-split. | One property can hold sharing Rooms, Studios, and BHK units at once — the old "PG or apartment" binary was just the collapsed case. | `property.unit_types_available[]` (`property.ts:412-416`); `unit_type ∈ {ROOM, 1RK, STUDIO, 1BHK…5BHK, BED}`; `sharing_type` (`room.ts:59`) via `getSharingTypeText()` (`commonFunctions.ts:1976`). | Pure-PG → one Rooms group sliced by sharing type; pure-apartment → unit-type groups, no sub-split; mixed → Rooms-with-sharing-rows beside Studio/BHK rows. |
| **6** | Build the **leaf-based ₹-loss query**: monthly loss = `SUM(rent)` over vacant leaves in the segment. Gate on rent-coverage: show "based on N of M leaves", and below the threshold show "set your rents to see your loss". | Bed rows copy the room's rent down, so summing room + beds double-counts. The gate keeps a guessed number off a trust screen. | `room.rent` (`room.ts:63`); vacant leaves from Task 1 or live. Optional: price an empty bed by its last tenant's `tenant_stay_history.rent_amount` before falling back to the sharing-type average. | On a mixed property, total loss sums over leaves only (hand-check: no double-count). Below the coverage threshold, the rupee number is replaced by the "set rents" copy. |

### Group 3 — Bookings & overbooking

| # | Task | Why | Files / queries | Acceptance check |
|---|---|---|---|---|
| **7** | Build the refined **true-overbooked** query: ≥2 *confirmed* bookings with *overlapping* stay dates on one bed. This refines the existing `new_bed_status=4` (which is count-only — no date or confirmed check). | A real clash is two tenants arriving for one bed. Today sequential or rejected bookings false-flag. Recommend a companion fix upstream: warn/block on overlap at booking *creation*. | `new_bed_status=4` enum (`rooms.ts:6695`; set at `rooms.ts:1560,3389,5664`, `roomRecommender.ts:463`); `tenant_booking_confirmation.is_confirmed`. | Two non-overlapping confirmed bookings on one bed → not flagged; two overlapping confirmed bookings → flagged; a rejected booking never contributes. |
| **8** | Filter every "booked = held bed" count to **confirmed only** (`is_confirmed=1`, or `auto_accept_booking` with no row). Exclude rejected (`-1`) always; surface pending (`0`) as a separate "N requests awaiting approval". | Today's "booked" never checks `is_confirmed`, so it's inflated by rejected + pending. A rejected booking holding a bed is a bug. | `tenant_booking_confirmation.is_confirmed`; `NEW_BOOKING` (1406) / `status=2` flag (`helpers.ts:1668`). | "Booked" drops to confirmed-only; a known rejected booking no longer counts; pending shows as its own "awaiting approval" number, never inside occupancy. |

### Group 4 — Correctness pushbacks

We have authority to fix wrong definitions, not inherit them.

| # | Task | Why | Files / queries | Acceptance check |
|---|---|---|---|---|
| **9** | Rename **"Overbooked" → "Over-occupied"** in the UI now; file a code-rename of the constant for later. | `OVERBOOKED_BEDS` measures living tenants over capacity — that's over-occupancy, and the wrong word already reaches owners. | `OVERBOOKED_BEDS` (filter 1414); occupied-over-capacity (`helpers.ts:1769`). | UI reads "Over-occupied beds"; the >100% explanation uses "over-occupied", never "overbooked"; rename ticket filed. |
| **10** | Unify **"vacant" on `status=1`** across both lenses; render "booked" as a separate *arriving* overlay, not a fill. | Today bed-level vacancy uses `status=1` and room-level uses `status IN (1,2)`, so the same booked-empty bed reads two different ways. That's the exact contradiction this screen exists to kill. | `VACANT_BEDS` (1403) vs `VACANT` (1408); `SEMI_VACANT` (1410). | A booked-empty bed reads vacant in *both* Bed and Unit views; "booked" shows as an arriving flag, not folded into the vacant slice. |
| **11** | Make **`status=1 AND date_of_eviction`** the one under-notice definition everywhere; reconcile old-structure properties to it. | A booking isn't "under notice" — only a living tenant can be leaving. The source of truth is `tenant.date_of_eviction`. | `tenant.ts:229`; fork on `new_property_structure` (B1). | Under-notice excludes all `status=2`; old- and new-structure properties return the same cohort for a fixed dataset. |
| **12** | *(Net-new — flag only, not in V1)* An **out-of-service** bed state for beds temporarily off-market (e.g. under repair). | No such state exists today, so a bed under repair sits in the denominator and reads vacant. This is a feature, not a wiring fix — the Brief scopes it out this cycle. | None today; would mirror `rent_disable`'s denominator exclusion as its own slice. | Documented as a product gap with a rough size; **not built** in v1 unless Section D pulls it in. |

### Group 5 — Per-section wiring *(mostly assembly over Groups 1–4)*

| # | Task | Why | Files / queries | Acceptance check |
|---|---|---|---|---|
| **13** | **Q1 / Q2:** the five counts (rentable / occupied / vacant / under-notice) + the bed-level rate, with the existing drill-downs. When the rate exceeds 100%, the headline must say *why* (over-occupancy). | The spine: state → location. These are cheap (live + existing filters). A >100% number with no explanation is the fastest way to lose trust. | `getOccupancy` (`service.ts:1836`); drill-downs via `POST /rooms/list/filters` (`filterCodes.ts:115-136`). | Five counts match live; rate is bed-level in both lenses; a >100% property shows the over-occupancy caveat inline. |
| **14** | **Q3 Vacancy Aging** (0–7 / 8–15 / 16–30 / 31+ days), always as-of-today, with never-rented beds in their own bucket. | Aging only makes sense relative to now. A new property's just-configured beds must not read as a wall of stale red. | Days since last move-out from `tenant_stay_history`. | Bands are computed from today; a never-occupied bed lands in "never rented", not 31+; a past-period filter does not move the bands. |
| **15** | **Q4 Upcoming Vacancy:** confirmed notices by days-to-free, split by whether a *confirmed* replacement is booked. Render Q1 under-notice, Q4, and the Q9 flag as one cohort sliced. | The forward pipeline, confirmed-only (no lead guesses). All three views are the same beds — show it, or the owner can't tell. | `tenant.date_of_eviction` forward; `date_of_joining` (`tenant.ts:216`); confirmed via Task 8. | A pending replacement does not mark a gap "covered"; the under-notice number in Q1, Q4, and Q9 all trace to the same beds. |
| **16** | **Q9: three separate flags** — booked-not-arrived · over-occupied (Task 9) · overbooked (Task 7) — each shown only when non-zero. | These used to be one muddled flag. They're different problems with different fixes (reassign vs resolve-the-clash). | Tasks 7, 8, 9. | Three distinct chips; over-occupied and overbooked never collapse into one number. |
| **17** | *(Stretch)* **Q6 Trend** + **Q7 Property-Wise** + **Q8 yield**, all `AVG()` over the Task 1 snapshot. Rank Q7 by rate %. | The trend ships empty today (`trend_text=null`); all three just read the snapshot. Cut order if time runs short: Q6, then Q7, then Q8 (Brief §5). | Snapshot table (Task 1); trend null at `service.ts:1860`. | A trend month uses the same average-daily definition as the period filter — hovering "May = 62%" matches Last-Month = May. Q7 ranks by rate %, with the count as support. |

**Two wiring rules that span the tasks above:**

- **Forward chips are hidden in the past.** Booked, Overbooked, and Under-Notice-with-booking describe the future — on a "Last Month" view they're noise, so hide them there. Aging is always as-of-today; Trend and Days-to-Fill are always their own historical series. The period filter drives everything else, and every tile shows a small time badge for its context.
- **Days-to-Fill, if built (Q5):** (a) count still-open vacancies at their current age and always print the open tail ("avg 14d · 2 beds still open, longest 86d") — otherwise the worst beds hide; (b) measure a gap as next `started_at` − prior `ended_at` on the same `bed_id`, but filter on `end_event_type` so a `ROOM_CHANGE` / `RENT_CHANGE` mid-stay doesn't read as a fake turnover.

---

## D. Two decisions for kickoff *(left open on purpose)*

### D1. If the cycle tightens, what's the cut order for the net-new foundation pieces?

The daily snapshot (Task 1) is the floor — not cuttable. These three *are* cut-order-able, and need ranking **only if time forces it**. This is an unranked menu, not a recommendation:

- **Effective-dated denominator (Task 4)** — exact/forensic denominator; the snapshot already makes the screen truthful forward, so this is the easiest to defer.
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
