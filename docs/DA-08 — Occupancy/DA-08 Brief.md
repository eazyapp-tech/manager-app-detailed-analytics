---
title: DA-08 Occupancy — Brief
date: 2026-06-03
tags: [rentok, brief, inventory, occupancy]
status: Living document · v0.3.3 (§2a flat model corrected — BHK = multi-room flat)
owner: Sanchay
time_budget: Open — tiers ranked by operator-pain
companion_to: DA-08 Build Sheet (post-design)
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
> A **Brief** for the Occupancy detailed-analytics screen — written before design. What the owner needs the screen to *do*, not how it looks.
> **What this is NOT:** an engineering spec or design doc. Those live in `[[DA-08 Build Sheet]]` and Figma frame `10611:113044`. First screen of the **Inventory** tab — the financial-tab series (DA-01…07) is done.

## 1. In one line

A PG owner can see his vacancy *number* but not its *shape* — and never its *cost*. The homescreen gives one percentage; the empty beds, their staleness, and their price sit in three different places or only in his head. So the bet is one thing: **make vacancy easy to see and priced.** This screen shows the whole picture in one glance — how full he is, where the gaps are, how long they've been open, who's leaving next, and what the vacancy costs him each month. An empty bed is the one leak with no back-collection: a bed empty in March is revenue gone forever.

*"Kitne bed khali hain, kab se khali hain, aur ye mujhe har mahine kitna nuksan kar rahe hain?"*

## 2. The operator

This screen is for the owner when he wants to **diagnose vacancy** — not when he wants to *edit beds or check a tenant in*. Diagnosis here; the fix happens in Bed Management.

**Rajesh (primary)** — owner-admin of a single 40–60 bed PG. Opens this weekly, or whenever the homescreen occupancy number looks off. His instinct: "empty bed = my money." He wants to see the empty beds, know how stale they are, and know which ones have a tenant already lined up versus which need a fresh lead.

**Priya (secondary)** — owns 3+ properties across cities. Her question is *"which property is dragging my portfolio down?"* — served by Property-Wise Occupancy.

**Amit (secondary)** — new, sparse inventory, some beds not yet configured. The screen must stay honest at 0% and during setup, not look broken.

**Meena (secondary)** — role-restricted manager. She earns a persona slot because of one occupancy-specific call: does a restricted manager see the **₹ loss number**, or only the occupancy counts? That's a per-role gate (footer test), not a generic permission line.

Where it sits: the homescreen says *one number* ("45% occupied"); Bed Management lets him *edit beds one room at a time*; this screen says *what shape the vacancy is and what it costs.* Tapping any segment, band, or row drills into the existing room/bed list, filtered.

*Personas are composites from the RentOk operator base, carried forward from the DA-01…07 series — not fresh field interviews for this screen.*

## 2a. The one idea this screen turns on — inventory model

RentOk inventory is not one shape. Two types, and **a single property can be both**:

- **Sharing model (PG / hostel):** a room (`unit_type='ROOM'`) holds multiple beds (`unit_type='BED'` children). The rented thing is a **bed**; `sharing_type` is its seater count. Bed ≠ unit, partial-fill is normal.
- **Whole-unit model (co-living / apartment):** the rented thing is a **unit**, not a bed. A **studio or private room** is a single room. A **BHK flat is usually several rooms grouped under one flat name** (each with its own beds), rented as one unit — so a multi-room flat *can* be partly filled (some rooms still empty). *(Verified: flats are grouped by room-name prefix, on the room-list side only — `getOccupancy` counts rooms/beds, not flats. See Formula Map Rule 6 + the flat-aware "bet" flag.)*

This is verified, not assumed: `property.unit_types_available[]` and `property.sharing_types_enabled[]` are arrays, so one property can mix both, and there is no `is_pg` flag to switch on ([property.ts:412-416](src/entities/property.ts#L412)).

**Consequence — three of the screen's main features must adapt per-property, not assume one model:**

| Feature | Sharing model | Whole-unit model |
|---|---|---|
| Bed ⇄ Unit lens | both meaningful (bed ≠ unit) | collapses — Bed = Unit; don't show a toggle that does nothing |
| Semi-Vacant slice | valid (a 3-seater with 1 bed filled) | none for a single studio/room; a **multi-room flat can be partly filled** (the flat-aware view — a bet) |
| "Losing money" axis | by **sharing type** (Single / Two / Three Sharing) | by **unit type** (1BHK / Studio / …) |

The screen picks the axis from the inventory actually present. And because **a single property can hold sharing Rooms, Studios, and BHK units at the same time**, the money breakdown isn't sharing-type *or* unit-type — it's **two-level**: group first by unit type (Rooms / Studio / 1BHK …), then — *under the multi-bed Rooms group only* — by sharing type. A pure-PG property collapses to one Rooms group sliced by sharing type; a pure-apartment property to unit-type groups with no sub-split. This idea governs §5, §6, and three traps in §8.

## 3. The problem

Three pains, same root cause — **the owner can see the vacancy number but not the shape of it, and never the cost:**

1. **"I know I'm 45% full, but I don't know which beds are empty or for how long."** The percentage tells him there's a problem, not where it is or whether it's getting worse.
2. **"I find out a tenant left only when I check, and by then the bed's been empty for weeks."** Notice periods are visible per-tenant, never rolled up into "here's who's leaving in the next 30 days."
3. **"Nobody puts a rupee figure on my empty beds."** He intuits that vacancy costs him, but the number lives only in his head — so it loses to more urgent, more visible problems like dues.

The bet isn't "fill the beds" — it's *make the vacancy easy to see and priced.* Once he can see which beds are stale and what they cost per month, the filling follows.

**Why now:** The Financial Insights tab is complete; Inventory is the next tab, and occupancy is the owner's single biggest way to grow revenue. Unlike dues — deferred revenue he can still chase — a vacant bed is unrecoverable. It's the only leak on the platform with no way to collect it back, and today it's the least measured.

## 4. What Rajesh does today

He's in **fragmented partial views** — the data exists, but no place reconciles it, so he can't trust the sum:

- **Reads the homescreen occupancy %.** One number, no shape. Tells him *that* there's vacancy, never *where* or *since when*.
- **Exports the Excel vacancy report** (the bed/room occupancy reports). Point-in-time, batched, and rupee-blind — by the time he opens it the picture has moved, and it never says what the empty beds cost.
- **Opens Bed Management and scrolls room by room** to spot empties. Works for 10 rooms, breaks at 50; no roll-up, no aging, no "leaving soon."
- **Does the loss math in his head, or not at all.** "5 beds empty at ₹8K… roughly ₹40K a month?" — vague, so it never competes with the dues number that's right in his face.

The screen has to beat all four: faster than scrolling Bed Management, more complete than the homescreen number, more current than the Excel export, and it has to do the one thing none of them do — **put a rupee number on the gap.**

## 5. What we must ship — and what we cut if time runs short

**The bet is to ship the full question set below.** The tiers describe a simpler fallback if the cycle slips — not the planned scope. Cut order applies only when time forces a choice.

**Must ship (V1 is not V1 without these):**
- **Q1 — How full am I right now?** Live, point-in-time: occupancy rate plus the snapshot counts — rentable, occupied, vacant, under-notice. The rate is bed-level; when overbooking pushes it past 100%, the headline itself must flag *why* — an unexplained ">100%" is the fastest way to lose an owner's trust, so this caveat is must-ship even though the full overbooking flag (Q9) is not.
- **Q2 — Where are the empty beds?** The occupied / vacant / under-notice / disabled decomposition, in whichever lenses the property's inventory supports (Bed and Unit when beds ≠ units; a single view when they're the same), each segment drilling to the matching bed or room list.
- **Q5 — What is the vacancy costing me?** Per inventory segment (sharing type, or unit type where present): occupancy %, days-to-fill, and a monthly ₹ revenue-loss number. The rupee number is **gated on rent-coverage** — it shows only when enough vacant beds carry a configured rent, always states "based on N of M beds," and below the threshold falls back to "set rents to see your loss." That turns thin data into an action (set your rents) instead of a number the owner can't trust.

These three are the core: *state → location → cost.* Q5 is what converts "vacancy" into "rupees," which is the thing that actually moves an owner.

**The must-haves are narrower still.** The live snapshot (Q1) and the existing-drill-down breakdown (Q2) are cheap and ship no matter what. The two *new* pieces fall back rather than fail all-or-nothing: Q5's rupee number falls back to occupancy-% + days-to-fill when rent-coverage is thin; the up-or-down comparison is pulled out of Q1 into the gated Q1b. So "if we can't ship all three" means we don't ship without *some* priced view — a thin rent table doesn't sink the screen.

**Nice to have (ship if time allows):**
- **Q1b — Is it up or down?** Period-over-period comparison on each snapshot count. *Gated:* ships only once the live and period formulas reconcile to an agreed tolerance (§8 trap) — a comparison that disagrees with the live headline is worse than none.
- **Q3 — How long have the empty beds been open?** Vacancy aging bands (0–7 / 8–15 / 16–30 / 31+ days).
- **Q4 — Who's leaving next?** Upcoming vacancy projected from *confirmed* notice periods, and which of those already have a **confirmed** replacement booked. *(A booking has states — confirmed / pending / rejected. Only a **confirmed** booking counts as a held bed; a pending one is a "request awaiting your approval," a separate signal, not a covered gap.)*
- **Q9 — Any booking or occupancy anomalies?** Three *distinct* flags, each a different problem with a different fix: **booked-not-checked-in**; **over-occupied** (more living tenants than beds — the thing that makes the rate read >100%, fixed by reassigning); **overbooked** (one bed promised to two arriving tenants — a clash to resolve before move-in). Plus under-notice-with-booking. *(These are genuinely separate — "overbooked" and "over-occupied" were one muddled flag before; they aren't the same problem.)*

These answer the next-tier question: *is the vacancy getting worse, and is any of it already solved?*

**If time runs short, cut in this order** *(ranked by operator-pain, not build-cost):*
1. **Q6 — Occupancy trend (6-month).** Historical and informational; it explains the past, it doesn't drive today's action. The owner misses this least.
2. **Q7 — Property-wise occupancy.** Single-property owners don't need it at all; only the multi-property owner does — see paired guidance below.
3. **Q8 — Avg revenue per occupied vs available unit.** Lowest priority and design-driven — it's on the Figma tile, but no persona actually asks it, and it sits next to Q5, which answers "what's it costing" more directly. Confirm it earns a slot (e.g. as an "are my occupied units underpriced?" signal) before building — otherwise it's a canvas-verify like the Property-Expense layers.

Don't touch Q1–Q5 or Q3/Q4/Q9 until all three above are cut.

**Paired-cut guidance (scale axis):**
- Single-property owner (Rajesh) → cut **Q7**; property-wise ranking is meaningless with one property.
- Multi-property owner (Priya) → keep **Q7**; it's her primary question. For her, cut **Q8** first instead.

## 6. What each question needs

Each line is the *information* — not how it looks. Tapping any segment, band, or row drills into the existing room/bed list, filtered to that subgroup, carrying property + period + active lens.

1. **Q1 How full now.** Rentable bed count, occupied count, vacant count, under-notice count, and the rate — point-in-time for the live view, each headline count paired with its change versus the prior equivalent period.
2. **Q2 Where the gaps are.** Decomposition of the same total by the lenses the property supports: by bed (occupied / vacant / under-notice / disabled) and, where beds differ from units, by unit (occupied / semi-vacant / vacant), with under-notice and disabled as counts.
3. **Q3 Aging.** Each vacant bed bucketed by how many days it's been empty.
4. **Q4 Leaving next** *(official home for the under-notice group)*. Beds with a confirmed notice, bucketed by days until the bed frees up, split by whether a **confirmed** replacement is already booked (a pending, not-yet-approved request doesn't count as covered). The Q1 under-notice count and the Q9 under-notice-with-replacement flag are *derived from this same group* — design should render them as one population sliced, not three independent-looking widgets, or the owner can't tell if they're the same beds.
5. **Q5 Cost of the gap.** Per inventory segment (sharing type, or unit type where present): its occupancy %, average days to fill a vacancy, and an estimated monthly rupee loss.
6. **Q6 Trend.** Occupied vs vacant counts across the last several months.
7. **Q7 Property comparison.** Per property: occupied-of-total and rate, ranked.
8. **Q8 Yield per unit** *(design-driven, pending validation — see §5)*. Average revenue across occupied units and across all available units.
9. **Q9 Flags.** Three separate anomalies, each shown only when non-zero, each a different action: **booked-not-arrived**; **over-occupied** beds (more living tenants than capacity — the cause of any >100% reading, fix by reassigning); **overbooked** beds (two arriving tenants promised the same bed — resolve the clash before move-in). Plus the under-notice-with-replacement count (a slice of Q4's group, not a separate population).

## 7. What we're NOT building this cycle

- **No editing beds, rooms, or rent from this screen.** The owner comes here to diagnose; he acts in Bed Management, which already owns enable/disable and rent edits. A second place to edit would split the source of truth.
- **No external market-rent benchmark.** V1's revenue-loss uses the property's *own* configured rents, not an outside market rate — owners distrust outside numbers, and we have no market-rent data source to stand behind anyway.
- **No BHK-typed segmentation build.** Where a property is whole-unit, the money section groups by the unit-type text that already exists; we are not introducing a new typed BHK field this cycle. Owners who never wrote a unit type still get the reliable sharing-type axis, so no one is left with an empty section.
- **No lead-based vacancy prediction.** Upcoming Vacancy counts *confirmed* notices only. Projecting off unconfirmed leads would make the forecast wobble and erode the owner's trust in the one forward number he has.
- **No new vacant-bed list.** Every drill-down hands off to the existing room/bed list; a parallel list would split the counts and reintroduce the trust problem this screen exists to solve.
- **No balance-as-of-any-date reconstruction.** The owner asks "how full now" and "how full last month," not "at 11:59 PM on the 14th." A past period reports **average occupancy across the period** (the hotel/PMS room-nights standard), *not* an end-of-period snapshot and not an arbitrary point-in-time rebuild. *(This supersedes the earlier "period = snapshot" phrasing — average is the truer number and the one consistent with the ₹-loss figure; flagged for user testing.)*
- **No "out-of-service" bed state this cycle.** Marking a bed temporarily off-market (under repair) has no representation in the data today; if owners need it, it's a new feature, not part of V1's occupancy math.

## 8. Traps & risks

**Traps (decided in advance):**

- **(USER+TEAM) The money axis and the lens adapt to the property's inventory — they are not universal.** BHK is not a real field; it lives as free-text inside `Room.type` and is read by string-sniffing, so a "1BHK/2BHK/Studio" breakdown is empty for PG/hostel stock.
  → Verified at [room.ts:52-53](src/entities/room.ts#L52) (`type` is free text), [propertyDetails.ts:58-68](src/repositories/propertyDetails.ts#L58) (substring sniff for "bhk"); `sharing_type` is a populated integer at [room.ts:59-60](src/entities/room.ts#L59), rendered by `getSharingTypeText()` ([commonFunctions.ts:1976](src/utils/commonFunctions.ts#L1976)). Decision: **default the money-section axis to sharing type; use unit-type only where the property has whole-unit inventory** ([property.ts:412-416](src/entities/property.ts#L412) confirms a property can be mixed). The Bed/Unit lens and the Semi-Vacant slice follow the same rule (§2a) — hide the lens toggle when bed = unit, drop Semi-Vacant when no room has capacity > 1.
- **(USER) Under-notice beds count as occupied in the rate.** A tenant who's given notice is still living there and paying — calling his bed "vacant" would make this screen's number contradict the homescreen and every tenant list.
  → Verified at [service.ts:L1836](src/v1/homepage/service.ts#L1836) — occupied = tenants with `status=1`, and under-notice tenants remain `status=1`. Decision: they stay counted as occupied; show them as a separate **at-risk** slice/chip so "vacancy coming" is visible without faking today's number.
- **(USER) The rate stays bed-level in both lenses; the lens only re-slices.** Today the Unit tab ships a *bed-based* percentage while labelled as units — a hidden mistake.
  → Verified at [service.ts:L1859](src/v1/homepage/service.ts#L1859) — `roomOccupancyRate` is computed but commented out; both tabs return `bedOccupancyRate`. Decision: lock the headline rate as bed-level in both lenses (per the 06 lens model); the lens changes the *decomposition*, not the headline.
- **(TEAM) Revenue-loss uses each vacant room's own configured rent — not an average of occupied units.** Averaging-of-occupied breaks for the most-vacant segment (zero occupied comparables = no number for the type that's leaking most).
  → Rent is stored per room at [room.ts:63-64](src/entities/room.ts#L63) (`rent`, default 0). Decision: monthly loss = sum of configured `rent` across vacant rentable rooms/beds; where `rent` is 0/unset, fall back to the average configured rent of the same sharing type. This removes the zero-occupied hole and needs no external data.
- **(TEAM) Three occupancy formulas exist in production and won't reconcile.** Live uses `status=1 / capacity`; the period path uses `tenant_stay_history` + `tenant_eviction_details`; `getHighestOccupancyMonth` uses `status IN (0,1,2) / sharing_type` and hardcodes year 2025.
  → [service.ts:L1836](src/v1/homepage/service.ts#L1836), [service.ts:L1912](src/v1/homepage/service.ts#L1912), [getYearlyStats.ts:L545](src/services/property/getYearlyStats.ts#L545). Decision: the **live `getOccupancy` path is official** for V1. Reconciling the period path is a *named engineering pre-req with its own acceptance test* — live and period must agree within an agreed tolerance for a fixed property/date before the period-over-period comparison (Q1b) ships. "Align the definitions" is a reconciliation spike, not a tweak; until that test passes, Q1b stays gated out. The highest-month script is stale and offline — do not wire it.
- **(USER) Occupancy can read above 100%.** Overbooked beds inflate the numerator while disabled beds are removed from the denominator.
  → [helpers.ts:L1769](src/v1/list_screens/rooms/helpers.ts#L1769) (occupied can exceed capacity), [service.ts:L1837](src/v1/homepage/service.ts#L1837) (disabled removed). Decision: don't cap the math — but the *minimal* overbooking caveat on the headline is must-ship with Q1, even though the full overbooking flag panel (Q9) is nice-to-have. A >100% number with no explanation is the single most credibility-destroying thing this screen could show.
- **(TEAM) The trend block ships empty today; trend and forecast are new queries.** `trend_text` is hardcoded `null` on the live path.
  → [service.ts:L1860](src/v1/homepage/service.ts#L1860). Decision: treat Q4 (forecast) and Q6 (trend) as new build, not wiring — reflected in their stretch-tier placement.
- **(TEAM) Drill-downs already exist — consume them.** Vacant / occupied / under-notice / overbooked / semi-vacant buckets all map to existing `RoomFilterCode`s on `POST /rooms/list/filters`.
  → [filterCodes.ts:L115-136](src/v1/constants/filterCodes.ts#L115). Decision: every drill-down hands off here; build no new list.
**The four risks:**

| Risk | Read | Mitigation |
|------|------|------------|
| Will owners use it? | MED–HIGH | Occupancy is the #1 way to grow revenue; the existing vacancy-report export proves recurring demand. Usage is weekly/periodic, not daily — scoped accordingly. |
| Will they understand it? | MED | The lens split, >100% overbooking, under-notice semantics, and the per-property axis are subtle. Mitigation: one bed-level headline number, an axis chosen automatically from the property's own inventory, flags that explain anomalies, plain tooltips. |
| Can we build it (open budget)? | MED | Live snapshot (`getOccupancy`) and all drill-downs exist. New build: per-property axis selection, period reconciliation (`getOccupancyForPeriod`), aging bands, confirmed-notice forecast, the configured-rent loss calc, and the trend series. Each tiered so a short cycle still ships the core. |
| Is it worth it? | STRONG | An empty bed is the only leak you can't claw back — no back-collection. Putting a price on it has big impact. |

**Data-readiness / empty experience:** This screen shows inventory another actor configured. At 0% occupancy (a real state — e.g. a just-opened property) show real zeros, not an error. When *all* beds are disabled there is no denominator — show "—", not 0%. **Days-to-fill needs vacancy history** — for a brand-new property or a segment that has never turned over, show "—" with a one-line "not enough history yet", never a fake 0. During setup (sparse inventory) keep numbers honest and add a "your numbers fill in as you add beds and tenants" note, never fake data.

**When to stop and reconsider:** Mid-cycle, if the configured-rent revenue-loss baseline tests as untrustworthy with owners (rents widely unset), drop the rupee number from Q5 and ship occupancy-% + days-to-fill only — Q5 falls back to the middle path rather than slipping the whole screen.

**Footer**

**Things to test with operators during the cycle** (don't block kickoff; settle before launch):
- Does the sharing-type axis read right to PG owners, and the unit-type axis to apartment owners — and what shows for a *mixed* property? (Owner: Sanchay; before money-section design lock.)
- Rent-coverage threshold: at what % of vacant beds with configured rent does the Q5 rupee number become trustworthy enough to show (vs "set rents to see your loss")?
- Per-role: should a restricted manager (Meena) see the ₹ loss number, or only the occupancy counts?
- Bed lens vs Unit lens — for single-occupancy properties, is hiding the toggle the right call, or do owners still expect to see it?

**Related docs:** `[[DA-08 Build Sheet]]` (post-design) · homescreen card spec `06-occupancy` + lens model `06a/06b/06c` · superseded draft `08-01-OCCUPANCY-DRAFT` (mine its INV-2 + Amara vignette) · Figma frame `10611:113044`.

**Key decisions locked** *(index of the §8 trap decisions — §8 is official):*
- DA-08, Inventory tab group (continues the DA series).
- Q5 rupee number gated on rent-coverage; shows "N of M beds," falls back to "set rents" below threshold.
- Period-over-period comparison (Q1b) gated on a live/period reconciliation acceptance test.
- Minimal overbooking caveat on the >100% headline is must-ship; full Q9 flag panel is nice-to-have.
- Under-notice beds are occupied in the headline rate + shown as a separate at-risk slice.
- Headline rate is bed-level in both lenses; the lens re-slices only.
- Money section (Q5) is V1 must-ship; revenue-loss baseline = each vacant room's own configured rent, fallback = sharing-type average.
- Money axis, Bed/Unit lens, and Semi-Vacant slice adapt per-property to the inventory model present; default money axis = sharing type, unit type only where whole-unit stock exists.
- Snapshot carries period-over-period comparison on each headline count.
- Live `getOccupancy` is the official formula; period view must align under-notice; highest-month script not wired.
- Drill-downs consume existing `POST /rooms/list/filters`; no new list.

**Changelog:**

| Date | Version | Change |
|------|---------|--------|
| 2026-06-03 | v0.1 | Initial Brief — altitude (Synthesis), 4 calls locked, codebase-grounded traps |
| 2026-06-03 | v0.2 | Figma-coverage patch (MoM comparison + 5-tile snapshot in Q1, "revenue" label in Q8, Property-Expense canvas-verify trap). Added §2a inventory-model idea: money axis / lens / semi-vacant adapt per-property (sharing-type default, BHK-text unreliable — [room.ts:52,59](src/entities/room.ts#L52), [property.ts:412-416](src/entities/property.ts#L412)). Revenue-loss baseline switched to per-room configured rent ([room.ts:63](src/entities/room.ts#L63)). Days-to-fill empty-state added. |
| 2026-06-03 | v0.3 | Phase 5 six-lens critique applied. Q5 rupee number gated on visible rent-coverage (was silent fallback). Period-over-period comparison split into gated Q1b behind a live/period reconciliation acceptance test. Overbooking caveat promoted to must-ship on Q1. Under-notice group given one official home (Q4); Q1/Q9 marked derivative. Q8 demoted to design-driven/pending-validation. §1 tightened to the "easy to see and priced" thesis. Meena tied to the real ₹-visibility per-role call. |
| 2026-06-03 | v0.3.1 | Removed the "Property Expense / Paid by / Paid to" canvas-verify trap — confirmed by PM as stray Figma layers, not a real element. |
| 2026-06-04 | v0.3.2 | Reconciled to the schema-verified formula map (v0.3.4) — WHAT-level only, no engineering shape. §7: period = **average** occupancy (room-nights standard), not snapshot (supersedes old "period = snapshot" line; flagged for user testing). §7: out-of-service bed state is new/out-of-scope (no data today). Q9 (§5 & §6): split the one muddled flag into **over-occupied** (living > capacity, cause of >100%) vs **overbooked** (one bed promised twice) — distinct problems, distinct actions. Q4 (§5 & §6): "replacement booked" = **confirmed** only; pending = separate "awaiting approval" signal. §2a: money breakdown is **two-level** (unit-type, with sharing-type sub-split under multi-bed Rooms) since one property mixes both. |
| 2026-06-05 | v0.3.3 | §2a flat model corrected — a BHK flat is usually **several rooms grouped under one flat name**, not one room; a multi-room flat **can** be partly filled. Flat grouping is room-list-side only (`getOccupancy` counts rooms/beds). Flat-aware Unit view noted as a **bet** (detail in Formula Map + Build Sheet Task 18). |
