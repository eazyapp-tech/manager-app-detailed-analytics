---
title: DA-08 Occupancy — Ground-Truth Formula Map (v0.3, time-aware + adversarially hardened)
date: 2026-06-03
tags: [rentok, occupancy, inventory, ground-truth, formula-map, time-dimension]
status: Living document · v0.3.5 (flat model corrected; "leaf" → "rentable unit")
owner: Sanchay
companion_to: DA-08 Brief (v0.3.2) · plain companion: DA-08 Ground-Truth Formula Map v0.4 (plain) · supersedes v0.1
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
> Every number on the Occupancy screen, with **one plain-English formula**, **where the number comes from in our data**, **what it tells the operator**, and — new in v0.2 — **how each number behaves in every time mode** (Now / Past / Range / Ahead). This is the shared contract: designer draws it, engineer queries it, PM signs it, so the same number can never mean two things. It is **not** the Build Sheet (no SQL, no UI, no edge-case matrix); it's the layer of truth the Build Sheet is written against.
>
> **What changed from v0.1:** v0.1 silently assumed "Now." We are shipping a period filter, so every number now carries a defined behavior per time mode — and the matrix at the end proves 100% coverage.

---

## The core idea behind the whole time dimension (read this first)

**Occupancy is a STATE, not a SUM.** Money screens add a period up ("₹2.1L collected in May"). Occupancy is "full or empty at a moment" — so "occupancy for last month" has **no single obvious meaning** until we *define* it. v0.2 defines it: **a past or range period reports the AVERAGE daily occupancy** ("on average, 62% full last month"), while **Now reports an instant snapshot** ("62% full right this second").

We chose average over month-end snapshot deliberately, and it's flagged for UAT — see the two flags at the bottom. The one-line reason: a bed empty 20 days then filled on the 31st reads ~100% on a month-end snapshot, which over-reports health and silently contradicts the ₹-loss number. Average is the only definition that stays honest about revenue actually earned.

This is not our invention — it is the **hotel / PMS industry standard**: period occupancy = *room-nights sold ÷ room-nights available*, which is exactly the average daily occupancy with a per-day rentable count (OPERA, Cloudbeds, Mews all compute it this way; out-of-order rooms leave the available count per day, our disabled-bed analogue).

---

## Data grounding — what's reconstructable (schema-verified 2026-06-03)

> Verified directly against the entities before locking the time model. The averaging promise is only as good as the data behind it — here's exactly what the data supports.

| Ingredient | Reconstructable? | Evidence |
|---|---|---|
| **Occupied beds per past day** (occupied count) | ✅ **Yes** | `tenant_stay_history.started_at / ended_at / bed_id`; `ended_at=NULL` = still in. On-notice stays occupied (`tenant_eviction_details.is_active=1`). |
| **Rentable beds per past day** (rentable count) | ❌ **No history** | `room.rent_disable`, `sharing_type`, capacity are **current-state only**. `room` has `@CreateDateColumn` but **no `@UpdateDateColumn`**, no audit/history table. Cannot answer "was this bed rentable on a past date?" |
| **Vacant-bed rent for past ₹-cost** | 🟡 **Partial** | `tenant_stay_history.rent_amount` snapshots rent *per stay segment* (occupied-side past cost is real history). Room base `room.rent` has no history. |
| **`getOccupancyForPeriod` (existing period path)** | ⚠️ **Overlap-counts → inflates** | `started_at <= end AND (ended_at IS NULL OR ended_at >= start)` counts anyone present at *any* point in the period — neither snapshot nor average. |
| **Existing daily snapshot table** | ❌ None | No `occupancy_history` / daily-stats / cron roll-up exists. |

### The foundation decision — a lean nightly snapshot, not historical reconstruction

The rentable count gap **cannot be fixed retroactively** (no audit trail to rebuild from; backfilling it would be guessing, which violates the truth principle). So the truthful path forward is the same one every PMS uses: **roll up occupancy nightly.**

> **Build a nightly job that writes one row per `property × segment × day`: `{ rentable, occupied, disabled, overbooked }`.**

This is deliberately **not** a per-room audit log (that would be over-engineering — the screen never needs per-room history, only per-day aggregates). With this small table:
- Every period number (headline, per-segment, property-wise, trend) is a trivial `AVG()` over rows.
- It **replaces** the `getOccupancyForPeriod` rewrite — you read the snapshot instead of looping dates at query time.
- The rentable count becomes **exact from launch day forward**.

### Pre-launch history — show it, labelled approximate *(locked)*

Because the occupied count *is* reconstructable, the 6-month trend and any "last month" view can be **backfilled at launch** from `stay_history` (real occupied) over an approximate rentable count (today's rentable config). Decision: **show the full history on day one with a subtle "estimated before \<launch date\>" marker.** It is the best available truth, fixes itself to exact as nightly snapshots accumulate, and the error only appears when inventory actually changed mid-window (rare; detectable via `created_at`). We never silently backfill — the marker is mandatory (it's the same trust contract as Rule 7's period label).

---

## The 8 ground rules (every formula below inherits these)

1. **A bed with a tenant who gave notice is still OCCUPIED.** He's living there and paying until his last day — counted as occupied, *separately* flagged "leaving soon." Never called vacant. *(`tenant.status = 1` stays 1 during notice.)*
2. **The headline rate is always bed-level — in both Unit and Bed views.** Switching to Unit view re-slices the *picture*; it never changes the headline %. *(Avoids the current bug where the Unit tab silently shows a bed-based %.)*
3. **One official occupancy formula.** Live count is `getOccupancy`. Two other formulas exist (a monthly-history one, a stale 2025 script) — we do **not** mix them in. History is read only for the trend and for past/range averages, and its "on notice" definition must be matched to the live one before it ships.
4. **Occupancy can read above 100% — from OVER-OCCUPANCY, not "overbooking."** The rate exceeds 100 when more *living* tenants (`status=1`) are assigned to a room than its capacity (3 people in a 2-bed room). Disabled beds leave the rentable count. We surface this as **over-occupied** and explain *why*. ⚠️ The codebase names this `OVERBOOKED_BEDS` (filter 1414) — a **wrong name** that already ships to the frontend: it computes `status=1 occupied − capacity`, i.e. over-occupancy. Bookings (`status=2`) are **never** in the rate and never push it past 100. **→ Correctness pushback (see box below): rename this to "Over-occupied" in UI and ideally in code.**
5. **The ₹ loss number only shows when we have enough rents.** Built from each vacant bed's own configured rent. Below the coverage threshold we say "set your rents to see your loss" instead of a guessed number.
6. **The rentable unit is the bed; the money axis groups beds by kind (verified).** The thing we count and sum ₹ over — the **rentable unit** — is the **bed** (new structure) or a single room treated as one bed (old structure). **Every total (counts, rate, ₹ loss) sums over rentable units, never a room *and* its beds both** — `BED` rows copy the room's rent down, so summing both double-counts. The money axis groups these by **`unit_type`** (Rooms / Studio / 1BHK …), with a **`sharing_type` sub-split only under multi-bed Rooms** (Single / Double / Four). A property can hold sharing Rooms, Studios, and BHK flats at once (`unit_types_available` is an array). *(Old binary "PG → sharing type XOR apartment → unit type" was the collapsed case, not the rule.)*
   - **A BHK flat is usually several rooms, not one (verified — corrects an earlier mistake).** Flats are built by grouping `room` rows that share a **name prefix** (`"101 Master"`, `"101 Hall"` → flat "101"), for BHK-typed rooms and **19 hardcoded flat-group properties (+1 property/pg_number combo)** (`helpers.ts:1271`, `roomConstants.ts → FLAT_GROUP_PG_IDS`; full list in the [[DA-08 Flat-Group Properties (tracking)]] note). Each room carries its own beds. "Whole flat rented" = every bed across the flat's rooms held by one tenant (no special field — just N beds, same tenant). **This grouping lives only in the room-list today — `getOccupancy` counts individual rooms and beds, never flats.** A studio is a single room with its bed(s); a sharing room is one room with several beds.
   - **Flat-aware Unit view = a bet (open, lean ship-and-watch).** The headline rate and all totals stay bed-level — that's the floor and always works. The **Unit view** *additionally* groups a flat's rooms into one unit (reusing the existing name-prefix grouping), so the owner sees "2BHK-101: fully rented / 1 room empty". This is an assumption: the grouping already works for these users in the room-list, so we expect it to carry over. **Open to pointers; the screen must work fully without it** — flat-awareness is extra, never required.
7. **(NEW) A global period filter is the default; every period-sensitive tile can be overridden; the effective period is always badged.** The top-of-screen period control (Now / This Week / This Month / Last Month / custom range / look-ahead) is the **default** for all period-sensitive tiles. But each such tile **can carry its own time override** — change one tile without moving the rest. There is only ever **one headline occupancy rate**, always period-labelled, so two headline rates can never silently disagree. The UX primitive that makes this coherent: **every tile wears a small time badge of its effective context** ("Last Month" / "as of today" / "next 30 days") — so a tile that didn't follow the global control reads as *deliberately scoped*, never as *stuck*. Some tiles are always-live or always-forward by definition (Rule 7a).

   **7a. Fixed-time tiles** (badge it, ignore the global default): **Vacancy Aging = always "as of today,"** **Upcoming Vacancy = always Ahead (look-ahead window),** **6-month Trend = always its own historical series,** **per-segment Days-to-Fill = always historical (measured over past turnovers).** These don't take a global or per-tile period override — their badge states their fixed framing.

   **7b. In-progress periods (This Week / This Month) = average so far.** A current, unfinished period is the **average so far**, badged with the day count ("this month so far · 12 days") — never projected to a full month. The Q1b ▲/▼ comparison for an in-progress period compares against the **same elapsed window** of the prior period (first 12 days of last month), not the full prior period — comparing 12 days to 30 is a false comparison.

8. **(NEW — systemic) Two property generations behave differently: `property.new_property_structure` (verified).** This flag forks core math — **capacity** comes from child-`BED`-row count when `true`, from `sharing_type` when `false`; and **under-notice** is counted as `status=1` (new) vs `status IN (1,2)` (old). The screen must branch on this flag, and **every formula below assumes the new-structure definitions are official** (bed-count capacity, `status=1` under-notice — see Ground Rule 1 and the under-notice pushback). Old-structure properties are matched to the same meaning. Treat the flag as a first-class build input, not an edge case — it touches capacity, occupancy, and the under-notice group across the whole screen.

---

## ⚠ Three states people (and our code) confuse — defined precisely

These three get muddled — including in the codebase, where one is literally misnamed. The screen must keep them distinct or every count downstream lies.

| State | Plain meaning | Who's on the bed | Data | Pushes rate >100%? |
|---|---|---|---|---|
| **Occupied** | Someone is *living* there now | 1 living tenant (`status=1`) | `occupied_only_count` | — |
| **Over-occupied** | More *living* people than beds | ≥2 living tenants on a bed/room over capacity | `status=1 occupied − capacity` (code mislabels this `OVERBOOKED_BEDS` 1414) | ✅ **Yes** — this is the only thing that does |
| **Booked** | A *future* tenant holds the bed; not living yet | `status=2` booking — can be on a **vacant** bed (arrival) **or** an **occupied** bed whose tenant is on notice (transition) | `has_new_booking` (1406) | ❌ Never in the rate |
| **Overbooked (true)** | Two *future bookings* clash on one bed for overlapping dates | ≥2 `status=2` with overlapping stay dates, same bed | **Partially detected** — `new_bed_status=4` flags count>1 (no date/confirmed check); DA-08 refines | ❌ |

### A "booking" is not one state — Confirmed / Pending / Rejected (verified)

`status=2` only means *a booking record exists*. The real sub-state lives in **`tenant_booking_confirmation.is_confirmed`**:

| Sub-state | `is_confirmed` | Holds a bed? | What it means to the operator |
|---|---|---|---|
| **Confirmed** | `1` (or property `auto_accept_booking=1` with no confirmation row) | ✅ **Yes** | "This bed is promised to someone arriving." |
| **Pending** | `0` | ❌ No — a *request* | "Someone asked; you haven't approved yet." → an **action**, not a held bed. |
| **Rejected / cancelled** | `-1` (still `status=2`, `is_active=1`) | ❌ No | Dead — must be **excluded** from every count. |

> **⚠ Verified bug this exposes:** `NEW_BOOKING` (1406) / the `status=2` flag (`helpers.ts:1668`) counts **all three** — confirmed + pending + **rejected** — because it never checks `is_confirmed`. So today's "booked beds" is inflated by dead and not-yet-approved bookings. **Wherever this map says "booked = a held bed" it means CONFIRMED only** (`is_confirmed=1`, or `auto_accept_booking` with no row). This drives the forecast, the under-notice-with-booking "already solved" flag, and true-overbooking detection — all must use **confirmed** bookings, never raw `status=2`. *(Pending bookings are worth surfacing separately as "N requests awaiting approval" — an action count, not occupancy.)*

### `new_bed_status` — the per-bed status machine (corrected, main-tree enum `rooms.ts:6695`)

`room.new_bed_status` is a **live-derived** rollup of `tenant.status` (not a drift-prone cache), with six values:

| Value | Meaning |
|---|---|
| 0 | Vacant |
| 1 | Occupied |
| 2 | Under-notice |
| 3 | Booking (one booking tenant on the bed) |
| **4** | **Overbooked — ≥2 booking tenants on one bed** |
| 5 | Notice + Booking (occupied-on-notice with a replacement booked) |

**Two important facts this nails down:**

1. **True overbooking is already partially detected.** `new_bed_status=4` is set wherever `booking_tenants.length > 1` (`rooms.ts:1560, 3389, 5664`, `roomRecommender.ts:463`), and the bed is locked from new bookings (`show_add_booking` excludes 4). **But it's coarse:** the trigger is *count > 1 only* — **no date-overlap check and no `is_confirmed` filter** — so two *sequential* bookings (Jan + March) on one bed flag as "overbooked" even though they never clash, and rejected/pending bookings can inflate it. So DA-08's true-Overbooked flag **refines an existing signal** (add date-overlap + confirmed-only), it doesn't build from zero.

2. **"Overbooked" is used for two different things in the code** — `OVERBOOKED_BEDS` (filter 1414) = over-*occupancy* (living `status=1` > capacity), while `new_bed_status=4` = over-*booking* (≥2 bookings). Same word, two meanings, two places. Both feed the rename pushback (#1).

> **No out-of-service / maintenance / reserved state exists.** A bed under repair has nowhere to live in the data today — the only rentable count exclusion is `rent_disable`. **If owners need "mark a bed out of service" (temporary, off the rate but not deleted), that's a brand-new feature**, not a wiring fix — flagged as a product gap below, consistent with the hotel "out-of-order" concept. *(An earlier sweep misread 4/5 as reserved/maintenance from a stale worktree; corrected here against the main-tree enum.)*

### ⚑ Correctness pushback — definitions/code we recommend changing (not just documenting)

The brief gives us authority to fix wrong definitions, not inherit them. Three calls:

1. **Rename "Overbooked" → "Over-occupied" (UI now; constant when feasible).** `OVERBOOKED_BEDS` (1414) measures living tenants over capacity — that's over-occupancy, and the word "overbooked" already reaches operators. It's wrong by plain English and by hotel-industry usage (where "overbooked" means *reservations sold beyond inventory*). Shipping the wrong word on a trust screen is exactly the trust-loss Rule 4 warns about. **Recommend: show as "Over-occupied beds"; file a code-rename so the constant stops lying.**
2. **Refine the existing overbooking signal — a coarse one exists (`new_bed_status=4`). → IN DA-08 SCOPE (decided 2026-06-04).** `new_bed_status=4` already flags ≥2 booking tenants on a bed, but on **count only — no date-overlap, no `is_confirmed` filter** — so sequential or rejected bookings false-flag. For a PG a *real* clash means *two tenants show up for one bed* — an operational disaster. **Scope (folded into this screen's build):** (a) a refined query — ≥2 **confirmed** bookings with **overlapping stay dates** on one bed — surfaced as a true **"Overbooked"** flag (distinct from over-occupancy); (b) **strongly recommended companion** — a validation at booking *creation* that blocks/warns on a real overlap, so the screen reports a shrinking problem, not a growing one. (a) is in DA-08; (b) is the upstream fix DA-08 should trigger.
3. **Unify "vacant" on a single basis across both lenses (status=1).** Today vacancy is computed two different ways (see Section 2 trap): bed-level uses `status=1` (a booked-empty bed *is* vacant), room-level uses `status IN (1,2)` (the same room is *not* vacant). The screen's job is "which beds are physically empty," and the truthful, industry-aligned answer is **physical occupancy = `status=1`; bookings are a forward arriving marker, not a fill.** **Recommend: standardize vacancy on the `status=1` basis in both lenses, and render "booked" as a separate "arriving" flag** — so Bed and Unit views can never disagree on what's empty. This is a definition fix to the existing room-level query, made deliberately, not a bug we route around.

4. **Pick one official under-notice definition — `status=1` (verified split).** Under-notice is counted two ways, forked by `property.new_property_structure`: old = `status IN (1,2) AND date_of_eviction IS NOT NULL` (wrongly counts *bookings* as under-notice), new = `status=1 AND date_of_eviction IS NOT NULL`. A booking isn't "under notice" — only a *living* tenant can be leaving. **Recommend: official = `status=1 AND date_of_eviction` (the new-structure definition) everywhere; match old-structure properties to it** (Ground Rule 8). Authority is the denormalized `tenant.date_of_eviction`; `tenant_eviction_details` is period-scoped, not the source of truth for this count.

6. **Refine the existing overbooking signal (don't rebuild it) + add a real out-of-service state (brand-new).** Two parts: **(a)** `new_bed_status=4` already flags ≥2 bookings on a bed but on *count only* — DA-08's true-Overbooked flag should add **date-overlap + confirmed-only** so sequential/rejected bookings stop false-flagging. **(b)** There is **no maintenance/out-of-service state today** — a bed under repair can't be expressed, so it sits in the rentable count and reads vacant. **Recommend a brand-new "Out-of-Service" flag** (owner marks a bed temporarily off-market; removed from the rate rentable count like disabled, shown as its own slice) — the hotel "out-of-order" standard. This is a feature, not a fix; size it as brand-new. *(`new_bed_status` is live-derived from `tenant.status`, so there's no cache-drift to match — that earlier worry is resolved.)*

5. **Filter "booked" to CONFIRMED — `NEW_BOOKING` (1406) currently includes rejected & pending.** Verified: the `status=2` flag never checks `tenant_booking_confirmation.is_confirmed`, so a **rejected** booking (`is_confirmed=-1`, still `status=2`) and a **pending** one (`is_confirmed=0`) both inflate the count. A rejected booking promising a bed is a straight bug. **Recommend: every "held bed / booked" count on this screen uses `is_confirmed=1` (or `auto_accept_booking` with no confirmation row); exclude `-1` always; treat `0` (pending) as a separate "awaiting approval" action count, not occupancy.** The forecast, the under-notice-with-booking "already solved" flag, and true-overbooking detection all inherit this — confirmed-only.

---

## How to read each section in v0.2

Every section below now carries a small **time behavior** line: what the number means under each mode.

- **Now (live):** instant snapshot — right this moment, from `getOccupancy`.
- **Past:** a finished period (e.g. Last Month) — **average daily occupancy**.
- **Range:** any custom start–end — same average-daily definition as Past.
- **Ahead (forecast):** what's coming — today's occupied − confirmed move-outs before the target date + confirmed bookings before it. *(Verified supported: move-outs via `tenant.date_of_eviction`, bookings via `tenant.date_of_joining` (tenant.ts:216) — both forward-filterable; the codebase already queries `date_of_joining <= targetDate`.)*

---

## Section 1 — Overview Snapshot

| Number | Plain-English formula | What it tells the operator | Source (eng) |
|---|---|---|---|
| **Rentable Units** | Every bed that can be rented = all beds minus "not for rent". | "The size of my rentable inventory." | `SUM(capacity)` − disabled · `helpers.ts:1721` |
| **Occupied Units** | Beds with a living tenant (incl. on-notice). | "This many earn me money." | tenants `status=1` · `helpers.ts:1758` |
| **Vacant Units** | Rentable beds with no tenant = Rentable − Occupied. | "This many are empty." | `GREATEST(capacity − occupied, 0)` · `helpers.ts:1756` |
| **Under-Notice Units** | Occupied beds whose living tenant has filed a move-out date. | "About to go empty — act first." | `status=1` + `date_of_eviction` · `tenant.ts:229` |
| **Occupancy Rate** | Occupied ÷ Rentable × 100, bed level. | "What share of rentable beds is filled." | `service.ts:1836` |
| **▲/▼ vs prior period** *(Q1b — gated)* | Same rate for the previous equal period, shown as the change. | "Filling up or emptying out?" | period path `getOccupancyForPeriod` · **gated on agreement check** |

> **⏱ Time behavior — Section 1**
> - **Now:** instant snapshot of all five counts + rate (`getOccupancy`).
> - **Past / Range:** the rate becomes **average daily occupancy** over the period; the counts (occupied / vacant / under-notice) become **period averages** ("averaged ~31 occupied last month"), not end-of-period totals. Period-labelled per Rule 7.
> - **Ahead:** the rate becomes **projected occupancy** at the target date (Rule 7a / Section 5 logic); counts become the projected occupied/vacant at that date. Under-Notice is naturally the forward group here.
> - **Q1b (▲/▼):** the delta is always "this period's value vs the prior equal period," and stays **gated** until live and period formulas match (see Brief §8). When gated out, no arrow shows.

---

## Section 2 — Occupancy Status (donut + lens)

| Number | Plain-English formula | What it tells the operator | Source (eng) |
|---|---|---|---|
| **Occupancy Rate (donut centre)** | Same as the snapshot rate — Occupied ÷ Rentable beds. Identical number, never recomputed differently. | "My one headline fullness number." | `service.ts:1836` |
| **Occupied (slice)** | Beds with a living tenant. | green / filled | `status=1` |
| **Vacant (slice)** | Rentable beds with no tenant. | the gap to fill | `capacity − occupied` |
| **Disabled for Rent (slice)** | Beds switched off — not in the rate at all. | "Off the books on purpose." | `rent_disable` · `room.ts:57` |
| **Under Notice (chip)** | Living tenant who has filed a move-out date. *(true sub-slice of Occupied — all still `status=1`.)* | "Vacancy coming." | `date_of_eviction` · `tenant.ts:229` |
| **Over-occupied (chip)** | Rooms with more *living* tenants than capacity. *(This is what code mislabels `OVERBOOKED_BEDS`.)* | "More people than beds — fix or it skews your rate." | `status=1 − capacity` · `helpers.ts:1769` |
| **Overbooked (chip — TRUE)** | ≥2 **confirmed** future bookings with overlapping stay dates on one bed. *(pending/rejected don't count.)* | "You promised one bed to two people — resolve before arrival." | refine `new_bed_status=4` (today count-only) + date-overlap + `is_confirmed=1` |
| **Booked (chip — arriving marker, CONFIRMED only)** | A *confirmed* future tenant holding a bed; **may be on an empty bed OR an occupied-on-notice bed.** Not living yet. *(Pending = separate "awaiting approval" count; rejected excluded.)* | "Someone's arriving — don't re-sell." | `status=2 ∧ is_confirmed=1` (or auto-accept) · `helpers.ts:1668` ⚠ today unfiltered |
| **Under-Notice-with-Booking (chip)** | A notice bed that already has a **confirmed** replacement. *(proof booked can sit on an occupied bed.)* | "This gap is already solved." | notice ∩ confirmed booking · code 1418 |
| **Bed view vs Unit view** | Bed view = count beds. Unit view = count rooms/units; a partly-filled sharing room shows as **Semi-Vacant**. Headline % stays bed-level in both. | "Same fullness, two ways to see gaps." | lens re-slice · `service.ts:1859` |
| **Semi-Vacant (Unit view only)** | A sharing room with some beds filled, some empty. *(Hidden for single-occupancy units.)* | "Rooms I can still partly fill." | `0 < active < capacity` · `helpers.ts:1762` |

> **Booked is an OVERLAY, not a sub-slice (corrected — earlier draft was wrong).** A booking (`status=2`) is a *future* tenant. It can fall on a **vacant** bed (someone arriving into an empty bed) **or** on an **occupied** bed whose tenant is on notice (the transition / under-notice-with-booking case). So booked is **not** a subset of vacant — render it as a cross-cutting "arriving" flag, never folded inside the vacant slice. *(Contrast: Under-Notice **is** a clean sub-slice of Occupied — those beds all have a living `status=1` tenant.)*

> **⚠ Trap — "vacant" is computed two different ways across the lenses (verified; recommend unifying).** Bed-level vacant (`VACANT_BEDS` 1403) = `capacity − status=1 occupied`, so an **empty-but-booked bed *is* vacant**. Room-level vacant (`VACANT` 1408) = `active_count = 0` where `active_count = status IN (1,2)` — so the **same room is *not* vacant** (the booking fills it), and Semi-Vacant likewise treats a booking as a fill. → The lens silently changes what "empty" means: a booked-empty bed reads as a *vacant bed* in Bed view but a *booked/non-vacant room* in Unit view. This is the exact "same number, two meanings" failure this map exists to kill. **Recommendation (pushback): standardize vacancy on the `status=1` physical basis in both lenses, and show "booked" as a separate arriving-arriving marker** (see the Correctness Pushback box). Until that lands, every vacancy count must declare its basis.

> **⏱ Time behavior — Section 2**
> - **Now:** live donut — every slice is the current count.
> - **Past / Range:** the donut reads as the **average split over the period** (average occupied / vacant / disabled). The forward-looking chips (**Booked**, **Under-Notice-with-Booking**) are **hidden in past mode** — they describe the future, so on a "Last Month" donut they're noise, not signal. (Cleaner than averaging a future state across a past window.)
> - **Ahead:** donut shows the **projected split** at the target date; Under-Notice and Booked chips are the live forward groups driving the projection.
> - **Lens (Bed/Unit):** time-independent — the lens re-slices whatever the active period produced; it never changes the headline rate.
> - **Over-occupancy ↔ vacant agreement check:** when *living* tenants (`status=1`) exceed capacity in some rooms, the rate can read >100% (Rule 4) while other beds sit empty. Not a contradiction — surface together: *"104% — 3 rooms are over-occupied (more tenants than beds) while 5 beds elsewhere are empty."* **Never call this "overbooking" to the operator** — that word means future-booking conflicts (see definitions box), not too-many-living-tenants.

---

## Section 3 — Rent yield *(Q8 — design-driven, pending validation)*

| Number | Plain-English formula | What it tells the operator | Source (eng) |
|---|---|---|---|
| **Avg Revenue per Occupied Unit** | Total configured rent of occupied beds ÷ number of occupied beds. | "What an average filled bed earns." | `room.rent` · `room.ts:63` |
| **Avg Revenue per Available Unit** | Total configured rent of all rentable beds ÷ number of rentable beds. | "My ceiling if every rentable bed were full." | `room.rent` over rentable |

> *Q8 is design-driven and pending validation (Brief §5). Listed for completeness; confirm it earns a slot before building.*

> **⏱ Time behavior — Section 3**
> - These are **configured-rent ratios, not time series.** Rent is a current property setting, not a historical fact we store per day.
> - **Now:** computed on the current occupied / rentable sets.
> - **Past / Range / Ahead:** **N/A — present-config only.** When the global filter is on a past period, these two tiles read "current rents" and label themselves as not period-bound (or are hidden), rather than faking a historical average. If Q8 is cut (likely), this row drops entirely.

---

## Section 4 — Vacant Room Status (how long empty)

| Number | Plain-English formula | What it tells the operator | Source (eng) |
|---|---|---|---|
| **Aging bands (0–7 / 8–15 / 16–30 / 31+ days)** | Each currently-vacant bed sorted by days empty (today − the day it went vacant). | "Fresh vacancies vs stale ones I've ignored." | days since last move-out · stay history |

> **⏱ Time behavior — Section 4 — FIXED-TIME (Rule 7a)**
> - **Always "as of today,"** in every global mode. Aging only makes sense relative to now: "this bed has been empty 24 days." There is no honest "aging as of last month."
> - **Now:** native.
> - **Past / Range:** **ignores the filter**; keeps the today-anchored bands and labels them "as of today" so the owner doesn't read them as historical.
> - **Ahead:** N/A — a bed that isn't vacant yet has no age. (Future gaps live in Section 5.)
>
> **Never-rented exception (honesty for new properties):** a bed that has **never been occupied** has no "went vacant" date — it isn't "stale vacancy you've ignored," it's un-activated inventory. Put it in a separate **"never rented"** bucket (or exclude from aging), never in the X-day-stale bands. This protects the Brief's new-property honesty requirement (Amit's just-configured beds must not read as a wall of stale red).

---

## Section 5 — Upcoming Vacancy (who's leaving next)

| Number | Plain-English formula | What it tells the operator | Source (eng) |
|---|---|---|---|
| **Forecast bands (0–7 / 8–15 / 16–30 / 31+ days)** | Beds with a **confirmed** notice, sorted by days until the move-out date. | "My vacancy pipeline — find replacements now." | `date_of_eviction` forward · confirmed notices only |
| **…with replacement booked** | Of those, the ones with a **confirmed** future tenant already booked. | "Already covered — skip these." | notice ∩ confirmed booking (`is_confirmed=1`) |

> **⏱ Time behavior — Section 5 — FIXED-TIME (Rule 7a)**
> - **Always Ahead.** This section *is* the forecast. The global filter, when set to a look-ahead window, sets the **look-ahead window** (next 7 / 30 days); it never pushes this section into the past.
> - **Now / Past / Range:** the section keeps its forward framing regardless, defaulting to a 30-day look-ahead window, and labels itself "next 30 days."
> - **Confirmed notices only** — no lead-based projection (Brief §7).

---

## Section 6 — Where is my property losing money?

> **Segment axis = a two-level tree, not either/or (Ground Rule 6, verified).** One property can hold sharing **Rooms** (with beds), **Studios**, and **1BHK/2BHK** whole-units at once. So:
> - **Primary group = `unit_type`** → *Rooms · Studio · 1BHK · 2BHK · …* (whatever this property actually has).
> - **Secondary split, under the multi-bed "Rooms" group only = `sharing_type`** → *Single · Double · Four-sharing.* Whole-units (Studio/BHK) get **no** sub-split — sharing_type isn't meaningful there.
> - **Everything sums over rentable units** (a bed, or a single-bed room). **Never sum a room *and* its beds both** — `unit_type="BED"` rows carry the room's rent copied down, so summing both double-counts. The rentable unit is bed-level, which keeps bed-level the universal rentable count.
>
> A pure-PG property collapses to one "Rooms" group sliced by sharing type (the simple case); a pure-apartment property is unit-type groups with no sub-axis; a mixed property shows Rooms-with-sharing-sub-rows **next to** Studio/1BHK/2BHK rows.
>
> **Flats (BHK) are usually several rooms grouped by name prefix, not one room** (Ground Rule 6). For ₹-loss this doesn't change the math — we still sum the rent of every vacant bed, and the flat's rooms all carry the BHK `type`, so they land in the same group. The **flat-aware Unit view** (showing a whole flat as one unit) is the extra bet — see the flag below; the money math here works bed-level regardless.

**Example — mixed property, "where am I losing money":**

| Unit type | Sharing sub-split | Vacant units | ₹ loss/mo |
|---|---|---|---|
| **Rooms** | Four-sharing | 3 beds | ₹15,000 |
| **Rooms** | Double | 1 bed | ₹6,000 |
| **Studio** | — | 2 units | ₹16,000 |
| **2BHK** | — | 1 unit | ₹15,000 |

| Number | Plain-English formula | What it tells the operator | Source (eng) |
|---|---|---|---|
| **Occupancy % (per segment)** | Occupied ÷ rentable **units** within that unit-type (and sharing sub-group) × 100. | "Which kind of inventory is emptiest." | grouped `status=1`/capacity over units |
| **Days to Fill (per segment)** | Average days a unit in that segment sat empty between one tenant leaving and the next moving in. "—" if no past turnover. | "How slow this type is to re-let." | move-out→move-in deltas · stay history |
| **₹ Revenue Loss (per segment)** | Sum of monthly rent of every **vacant unit** in that segment. *(Gated: "based on N of M units"; below threshold → "set rents to see your loss.")* | "Rupees this empty type costs me." | `SUM(rent)` over vacant units, `unit_type` group · `room.ts:63` |

> **⏱ Time behavior — Section 6** *(the section where the time definition matters most)*
> - **Occupancy % (per segment):** Now = snapshot; **Past / Range = average daily occupancy within the segment**; Ahead = projected. Same definition as the headline, scoped to the segment.
> - **Days to Fill:** **fixed historical** — a measured average over past turnovers, *not* re-scoped by the global filter (a single month rarely has enough turnover to be stable). Always "based on past turnover"; "—" when none.
>   - **⚠ Don't-hide-the-worst-beds fix:** counting only beds that *eventually refilled* hides the worst beds — a bed empty 200 days and **still empty** never finished a turnover, so it never enters the average, and the segment bleeding most reads *healthy*. Fix: count still-empty beds at their current age, and **always print the open tail beside the average**: *"avg 14 days · 2 beds still open, longest 86 days."* The whole job of this number is to show stale beds — it must never hide them.
>   - **⚠ Turnover caveat (verified):** the gap = next stay's `started_at` − prior stay's `ended_at` on the same `bed_id`, **but** `tenant_stay_history` writes new rows mid-stay for `ROOM_CHANGE` / `RENT_CHANGE`. Filter on `end_event_type` (count only true move-out events, not transfers) or an internal transfer reads as a fake turnover. *(verified: `tenantStayHistory.ts` — `bed_id`, `ended_at`, `start/end_event_type`.)*
> - **₹ Revenue Loss — the two-faced number:**
>   - **Now → current monthly rate:** current vacant beds × their monthly rent → "these empty beds are costing you **₹X / month right now**." Forward-looking burn.
>   - **Past / Range → actual cost:** each bed's **vacant-days × daily rent**, summed → "vacancy **actually cost you ₹X** last month." The truer number owners have never been shown. *(Daily rent = monthly ÷ actual days in that month, not ÷30.)*
>   - **Ahead → projected loss:** current monthly rate applied to the beds projected vacant over the look-ahead window.
>   - The rent-coverage gate (Rule 5) applies in all modes; the "N of M beds" caption travels with the number.
>   - **Rent sourcing (schema note):** the occupied-side rent has real history in `tenant_stay_history.rent_amount`; the **vacant** bed's rent uses current `room.rent`. Optional truth-boost: price an empty bed by its **last tenant's `rent_amount`** from stay history before falling back to the sharing-type average (Brief §8) — closer to what the bed truly earns than a default.

---

## Section 7 — Occupancy Trend (last 6 months)

| Number | Plain-English formula | What it tells the operator | Source (eng) |
|---|---|---|---|
| **Occupied vs Vacant per month** | For each of the last 6 months, how many beds were occupied vs vacant. | "Is my fullness trending up or down?" | history path · **definition must match live before ship** |

> **⏱ Time behavior — Section 7 — FIXED-TIME (Rule 7a)**
> - **Always its own 6-month series**, in every global mode — it *is* the time view, so it ignores the single-period filter entirely.
> - **Each monthly point uses the same "average daily occupancy" definition** as Past/Range (Decision 1) — so a hover on "May = 62%" matches what the filter shows when set to Last Month = May. This consistency is the whole point of locking average: the trend and the period filter must tell the same story.
> - **Must-match pre-req:** the history path's "on notice" handling must match the live one before this ships (Ground Rule 3).

---

## Section 8 — Property-Wise Occupancy (multi-property)

| Number | Plain-English formula | What it tells the operator | Source (eng) |
|---|---|---|---|
| **Rank value = Occupancy Rate %** | Each property's Occupied ÷ rentable beds × 100. Ranked by this. | "Which property is emptiest — fair across PGs and apartments." | per-property `status=1`/capacity |
| **Count = Occupied / Total (bed-level)** | "30 / 50" = 30 of 50 rentable **beds** filled. Bed-level on every row. | "The raw size behind the %." | bed-level both sides |

> **⏱ Time behavior — Section 8**
> - **Now:** each property's live rate, ranked.
> - **Past / Range:** each property's **average daily occupancy** over the period, ranked on that — so "who dragged me down last month" is answerable, not just "who's empty right now."
> - **Ahead:** each property's projected rate at the look-ahead window, ranked.
> - Ranking metric stays **rate %** in all modes (the only fair cross-property type comparison — UAT flag carried from v0.1); the bed-level count is the supporting number.

---

## Drill-down (every tap lands on an existing list — we build no new list)

| Tap | Lands on | Filter |
|---|---|---|
| Vacant slice / Vacant Units | Bed list | `VACANT_BEDS (1403)` |
| Occupied slice | Bed list | `OCCUPIED_BEDS (1405)` |
| Under Notice | Bed list | `UNDER_EVICTION (1402)` |
| Semi-Vacant | Room list | `SEMI_VACANT (1410)` |
| Over-occupied *(code: `OVERBOOKED_BEDS 1414` — wrong name, rename recommended)* | Bed list | `status=1 occupied − capacity` |
| Disabled | Bed list | `DISABLED_FOR_RENT_BEDS (1420)` |
| Booked *(arriving marker — may overlap occupied)* | Bed list | `NEW_BOOKING (1406)` |

> Source: `POST /rooms/list/filters` · `filterCodes.ts:115-136`. Each tap carries property + **the active period** + active lens.
> **⏱ Time note:** drill-downs are **always a present-state list** (these beds, right now). A tap from a *past/average* number lands on today's matching beds, and the list header must say so ("vacant beds as of today") — an averaged number has no single bed-list behind it, and pretending otherwise would reintroduce the trust gap this screen exists to close.

---

## ⏱ The Time × Number matrix — 100% coverage

Every number on the screen, with a defined behavior for every mode. No blanks — an "N/A" is an explicit decision with a reason, not an omission.

| Number | Now (live) | Past (finished period) | Range (custom) | Ahead (forecast) |
|---|---|---|---|---|
| **Occupancy Rate (headline & donut)** | Instant snapshot | **Avg daily occupancy** | **Avg daily occupancy** | Projected @ target date |
| **Occupied count** | Snapshot | Period average | Period average | Projected occupied |
| **Vacant count** | Snapshot | Period average | Period average | Projected vacant |
| **Under-Notice count** | Snapshot | Period average | Period average | Forward group (native) |
| **Disabled count** | Snapshot | Period average | Period average | Projected (rare to change) |
| **▲/▼ vs prior (Q1b)** | vs prior equal period — **gated** | vs prior equal period — **gated** | vs prior equal range — **gated** | N/A — forecast has no "prior" |
| **Booked (arriving marker) / Under-Notice-w-booking chips** | Snapshot | **Hidden in past** (forward-only) | **Hidden in past** (forward-only) | Drives the projection |
| **Over-occupied (status=1 > capacity; code mislabels "overbooked")** | Snapshot | Period average | Period average | Projected |
| **Overbooked (TRUE — ≥2 confirmed bookings clash on a bed; refines `new_bed_status=4`)** | Current conflicts | **Forward-only** (hidden in past) | **Forward-only** | Conflicts within look-ahead window |
| **Bed ⇄ Unit lens, Semi-Vacant** | Re-slice of live | Re-slice of period avg | Re-slice of period avg | Re-slice of projection |
| **Avg revenue / occupied & available (Q8)** | Current-config ratio | **N/A — present-config only** | **N/A — present-config only** | **N/A — present-config only** |
| **Vacancy Aging bands** | As-of-today | **As-of-today** (ignores filter) | **As-of-today** (ignores filter) | N/A — no age before vacant |
| **Upcoming Vacancy bands** | Next 30d (fixed-time) | Always Ahead (ignores filter) | Always Ahead (ignores filter) | Horizon = filter window |
| **…with replacement booked** | Next 30d | Always Ahead | Always Ahead | Horizon = filter window |
| **Per-segment occupancy %** | Snapshot | **Avg daily occupancy** | **Avg daily occupancy** | Projected |
| **Per-segment Days-to-Fill** | Historical avg (fixed-time) | Historical avg (fixed-time) | Historical avg (fixed-time) | N/A — measures the past |
| **Per-segment ₹ Revenue Loss** | **Current monthly rate** (₹/month now) | **Actual cost** (vacant-days × daily rent) | **Actual cost** | Projected loss over look-ahead window |
| **6-month Trend series** | Own 6-mo series (fixed-time) | Own 6-mo series (fixed-time) | Own 6-mo series (fixed-time) | Own 6-mo series (fixed-time) |
| **Property-Wise rate & count** | Snapshot, ranked | **Avg daily occupancy**, ranked | **Avg daily occupancy**, ranked | Projected, ranked |
| **Drill-down lists** | Beds as-of-today | Beds as-of-today (labelled) | Beds as-of-today (labelled) | Forward group lists |

**Behavior classes (the same coverage, summarized):**

| Class | Numbers |
|---|---|
| Snapshot @ Now / **average** over Past·Range / projected Ahead | Occupancy rate, Occupied, Vacant, Disabled, Over-occupied, per-segment occupancy, Property-Wise |
| Intrinsic as-of-today (ignores filter) | Vacancy Aging |
| Forward-only (hidden in past mode) | Booked (arriving marker), Overbooked (true), Under-Notice-w-booking chips |
| Always Ahead (filter sets the look-ahead window) | Upcoming Vacancy, projected occupancy |
| Always historical series (ignores filter) | 6-month Trend, per-segment Days-to-Fill |
| Current monthly rate @ Now / **actual-cost** over Past·Range / projected Ahead | Per-segment ₹ Revenue Loss |
| Present-config only (no time behavior) | Q8 avg revenue per occupied / available |
| Gated (ships only after agreement check) | Q1b ▲/▼ comparison |

---

## ⚑ Flagged for UAT / user review (not engineer-decidable)

Carried forward from v0.1, plus the two new time-dimension flags. All are correct by logic but turn on **operator preference**, so confirm with real owners before lock.

1. **(NEW — time) Past/Range "how full" = AVERAGE daily occupancy, not month-end snapshot.**
   *Reasoning:* snapshot is simpler and matches naive "how full am I," but **hides mid-month churn** — a bed empty 20 days then filled on the 31st reads ~100%, over-reporting health and contradicting the ₹-loss number. Average is truest to revenue earned and the only definition consistent with ₹-loss and the trend. *UAT to confirm:* owners reading "62% last month" intuitively hear **"average,"** not "end of month."
2. **(NEW — time) The filter drives the whole screen, with a mandatory dynamic period label.**
   *Decision:* one period filter governs every number; the active period is always shown and dynamic; fixed-time-time sections (Aging / Upcoming / Trend) keep their own framing and say so. *UAT to confirm:* owners **never misread a past number as current** — the label does its job.
3. **(v0.1) Multi-property money section — property-first, not pooled segments.**
   Rank **properties** by ₹ loss; tap to see segment breakdown; show a **portfolio total** on top. No pooling across properties in V1. *Watch:* a future **chain operator (10+ near-identical PGs)** may want "Two-Sharing is empty across all PGs" — revisit pooling only if that segment grows.
4. **(v0.1) Property-Wise ranking — by rate %, not rupees.**
   Rank by **occupancy rate %** (only fair cross-property type comparison); bed-level "30/50" supports it. *Risk:* a small property at 50% ranks "worse" than a big one at 90% losing more absolute rent — confirm owners read the health-scan (rate) and money (₹) sections as **different jobs**.
5. **(NEW) Flat-aware Unit view — a bet, lean ship-and-watch.**
   *Decision:* the bed-level rate and all totals are the floor and always work. The **Unit view** *additionally* groups a BHK flat's rooms into one unit (reusing the existing room-list name-prefix grouping), so the owner sees "2BHK-101: fully rented / 1 room empty." *The bet:* this grouping already works for these users in the room-list, so we expect it to carry over to occupancy. *Open to pointers; the screen must work fully without it* — flat-awareness is extra, never required, and only applies to BHK-typed inventory + the **19 flat-group properties (+1 combo)** — tracked in the [[DA-08 Flat-Group Properties (tracking)]] note.

---

## ⚙ Build-effort flag (scope this as foundation, not wiring)

The existing period path `getOccupancyForPeriod` (`service.ts:1912`) counts **anyone present at any point in the period** (overlap) — neither snapshot nor average; it **inflates** occupancy. So "average daily occupancy" (Decision 1) is **not a date-filter add** on the existing path.

**The right foundation (verified against schema):** the occupied count (occupied/day) is reconstructable from `tenant_stay_history`, but the rentable count (rentable/day) has **no history** and can't be rebuilt retroactively. The fix is the **nightly `property × segment × day` snapshot table** (see "Data grounding" above) — a lean aggregate roll-up, not a per-room audit log. Once it exists, period occupancy is `AVG()` over rows and the overlap-counting path is *retired*, not patched. This is the single biggest build item and the foundation the whole time dimension rests on.

**Denominator storage — resolved (industry-grounded).** The two measures change at different rates, so they take different standard patterns (Kimball "Daily Snapshot vs SCD-2"):
- **Occupied/day → periodic daily snapshot** (changes daily; daily grain correct). This is the foundation above and makes the screen truthful from launch forward.
- **Rentable/day → effective-dated `room_state_history` (SCD-2):** add `@UpdateDateColumn` to `room` + write one effective-dated row *only when* `rent_disable` / `capacity` / `sharing_type` changes. Nearly free because PG inventory changes rarely (a daily snapshot of an unchanging rentable count = redundant rows forever, the anti-pattern). Buys an exact, auditable rentable count and exact pre-launch backfill if we ever retire the "estimated" marker.
- **Do not build** full event-sourcing / per-field audit — answers questions the screen never asks (over-engineering line).

Both are cut-order-able relative to v1 scope, but the periodic snapshot is the floor — without it, period occupancy is not truthful.

**Re-grounding targets for the build:** `getOccupancyForPeriod` (`service.ts:1912`), `getOccupancy` (`service.ts:1836`), `tenant_stay_history` (`started_at/ended_at/bed_id/rent_amount`), `tenant_eviction_details` (`is_active`), `room` (`rent_disable`/`sharing_type`/`rent` — current-state only).

---

## Changelog

| Date | Version | Change |
|------|---------|--------|
| 2026-06-03 | v0.1 | Initial formula map — 8 sections, 6 ground rules, drill-down codes, 2 UAT flags. Grounded against `getOccupancy` / `helpers.ts` / `room.ts` / `filterCodes.ts`. |
| 2026-06-03 | v0.1 + time-spec | Appended locked spec for the v0.2 time rewrite (core principle, 4 modes, 5 locked decisions, behavior matrix, build-effort flag, kickoff checklist). |
| 2026-06-03 | **v0.2** | **Full time dimension to 100% coverage.** New doc. Added Ground Rule 7 (one filter drives the screen + dynamic label) and 7a (fixed-time-time exceptions). Added per-section ⏱ time-behavior blocks for all 8 sections. Locked: average daily occupancy for Past/Range; current monthly rate-vs-actual-cost for ₹ loss; projected-occupancy forecast; fixed-time time for Aging / Upcoming / Trend / Days-to-Fill; Q8 as present-config-only. Completed the **Time × Number matrix** (every number × 4 modes, no blanks). Carried 2 v0.1 UAT flags + added 2 new time flags. Restated the `getOccupancyForPeriod` build-effort flag as rework-not-wiring. |
| 2026-06-03 | **v0.3** | **Adversarial pass + schema verification.** Verified reconstructability against entities: occupied count (occupied/day) ✅ from `tenant_stay_history`; rentable count (rentable/day) ❌ no history; `getOccupancyForPeriod` confirmed overlap-counting; no snapshot table exists. Grounded averaging in hotel/PMS room-nights standard. Locked the **foundation: lean nightly `property×segment×day` snapshot** (not a per-room audit log). Locked **pre-launch history shown + "estimated before launch" marker**. Fixes folded in: Days-to-Fill **censoring** (survivorship), **forward chips hidden in past mode**, **never-rented** aging bucket, Rule 7 reworked to **global-default + per-tile override + time badge**, Rule 7b **in-progress = to-date average** + same-elapsed comparison, ₹ rent sourcing from `stay_history.rent_amount`. Build-effort flag reframed as foundation (snapshot retires the overlap path). |
| 2026-06-05 | **v0.3.5** | **Flat model corrected + "leaf" renamed.** (1) Renamed **"leaf" → "rentable unit"** everywhere (plain word). (2) **Corrected the whole-unit/flat model:** an earlier pass wrongly said a BHK flat is one childless room. Verified: a flat is usually **several rooms grouped by a shared name prefix** (`helpers.ts:1271`, `FLAT_GROUP_PG_IDS` — BHK-typed rooms + 19 properties +1 combo), each room with its own beds; "whole flat rented" = all those beds to one tenant (no special field). **This grouping lives only in the room-list — `getOccupancy` counts rooms/beds, never flats.** (3) Locked the **flat-aware Unit view as an extra bet** (open, lean ship-and-watch): the bed-level rate and totals are the floor and always work; the Unit view *additionally* groups a flat into one unit, reusing the existing grouping; the screen must work without it. Rule 6, §6, and the flags updated. |
| 2026-06-04 | **v0.3.4** | **Correction — `new_bed_status` enum + overbooking reality.** The v0.3.3 "maintenance(5)/reserved(4)" reading was **wrong** (stale-worktree misread). Main-tree enum (`rooms.ts:6695`): 0 vacant·1 occupied·2 under-notice·3 booking·**4 overbooked (≥2 booking tenants on a bed)**·5 notice+booking — **no maintenance/reserved/out-of-service state exists.** Two consequences: (1) **true overbooking is partially detected already** via `new_bed_status=4` (`rooms.ts:1560/3389/5664`, `roomRecommender.ts:463`) but **count-only — no date-overlap, no confirmed filter**; DA-08 *refines* it (pushback #2 reworded from "build" to "refine"). (2) "Overbooked" names two different things in code — filter 1414 (over-occupancy) vs `new_bed_status=4` (over-booking). Out-of-service/maintenance is a **brand-new feature** if owners need it (pushback #6 reworded). `new_bed_status` is live-derived from `tenant.status` → no cache-drift. |
| 2026-06-04 | **v0.3.3** | **Final state-machine sweep.** Confirmed tenant.status 3 (lead) / 4 (joining-request) correctly excluded from occupied & booked; gender tags don't affect rentable count; eviction keeps `status=1` until executed (matches Rule 1). **New finding:** `room.new_bed_status` carries 6 values (0 vacant/1 occupied/2 under-notice/3 booked/**4 reserved**/**5 maintenance**); maintenance & reserved are an out-of-service concept the occupancy math ignores (keys off `tenant.status`) and currently leaves in the rentable count. Added the bed-availability box + pushback #6 (treat maintenance/reserved as Out-of-Service, removed from rentable count like disabled; match `new_bed_status` vs `tenant.status`). Flagged lower verification confidence (stale worktree) — re-confirm rentable count treatment + room soft-delete in Build Sheet. |
| 2026-06-04 | **v0.3.2** | **Inventory hierarchy + booking sub-states + forward-query verification.** (1) **Money axis rewritten to a two-level tree** (Ground Rule 6 + §6): one property mixes sharing **Rooms** (+beds), **Studio**, **1BHK/2BHK** whole-units — verified `unit_types_available` is an array; primary group = `unit_type`, secondary `sharing_type` split only under Rooms; **₹ loss sums over rentable units, never room+beds** (double-count guard); worked example added. (2) **New Ground Rule 8 — `property.new_property_structure`** forks capacity (bed-count vs sharing_type) AND under-notice (`status=1` vs `status IN (1,2)`); new-structure defs are official. (3) **Overbooked (true)** added to matrix, Section 2 chip, behavior-classes (was dropped) — distinct from over-occupied. (4) **Booking sub-states** modelled: `tenant_booking_confirmation.is_confirmed` 1/0/−1 = Confirmed/Pending/Rejected; verified bug — `NEW_BOOKING` (1406) counts rejected+pending too; "booked" now means **confirmed only** across booked chip, forecast, under-notice-with-booking, overbooking. (5) Temporal verifications folded: Days-to-Fill needs `end_event_type` filter (exclude transfers); forecast confirmed via `tenant.date_of_joining`. Pushbacks #4 (official under-notice) and #5 (filter booked to confirmed) added. |
| 2026-06-04 | **v0.3.1** | **State-semantics correction (schema-verified) + correctness pushback.** Caught & fixed a domain error: **"overbooked" ≠ "over-occupied."** Verified in code: `OVERBOOKED_BEDS` (1414) computes `status=1 − capacity` (over-occupancy of *living* tenants), **mislabeled "overbooked"** and already shipping that word to the frontend. Corrected Rule 4, Section 2, drill-down, and the >100% story to **over-occupancy**. Added the **"Three states people confuse" definitions box** (Occupied / Over-occupied / Booked / true-Overbooked). Reversed the earlier **"Booked ⊂ Vacant"** claim — verified booked (`status=2`) can sit on occupied (on-notice) **or** vacant beds → booked is an **arriving marker**, not a sub-slice. Found & flagged a new **lens-vacancy divergence**: bed-level vacant uses `status=1`, room-level vacant uses `status IN (1,2)` → same booked-empty bed reads vacant in Bed view, not-vacant in Unit view. **Three correctness pushbacks added** (we have authority to fix definitions, not inherit them): (1) rename Overbooked→Over-occupied in UI + code; (2) build real booking-overlap detection — *it does not exist today*, two `status=2` bookings can clash on one bed silently; (3) unify vacancy on the `status=1` basis across both lenses. Fixed source refs (`tenant.ts:229`). |
