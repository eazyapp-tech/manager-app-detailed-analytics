---
title: DA-08 — Flat-Group Properties (tracking)
date: 2026-06-05
tags: [rentok, occupancy, inventory, flat-group, tracking]
owner: Sanchay
companion_to: DA-08 Ground-Truth Formula Map v0.2 (Rule 6 + flag #5) · DA-08 Build Sheet (Task 18)
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



> **What this is.** The flat-aware Unit view (a 2BHK shown as one unit) is a **bet** — it only fires for BHK-typed inventory *and* for the hardcoded properties below. This is the list to **watch**: confirm the flat grouping reads right for these owners, and add notes as we learn.
>
> **Where it's set:** `src/v1/constants/roomConstants.ts` → `FLAT_GROUP_PG_IDS` (a Set) + `FLAT_GROUP_PG_ID_WITH_NUMBER`. Used in `src/v1/list_screens/rooms/helpers.ts:1271`.
>
> **Count: 19 property IDs + 1 (property, pg_number) combo = 20 entries.** (Plus: any room whose `type` is a BHK/2RK gets grouped regardless of this list.)

## The 19 flat-group property IDs

| # | pg_id | Known name | Flat view reads right? (fill in) |
|---|---|---|---|
| 1 | `mfPmPUg4JQYoktjNEAKD1pNV2sj1` | Meridian | |
| 2 | `F8Y0fI1DhxZlDAIxcsJuGqdBsep2` | | |
| 3 | `YpV3LGfiHRg1iNbjwGEoR012dFt2` | | |
| 4 | `QbN73z8J2tPgNYVszrhr9Ix3JBP2` | | |
| 5 | `zRQUEcPMpsVy6423soxCelWHCFb2` | | |
| 6 | `uFGY6vcHTfNmUwoS0ZgjsmvokW22` | | |
| 7 | `cFsXs6I8mzTWAMWHYzHJOjnlnHW2` | | |
| 8 | `kHSrFYnhNreWecRZIDkGHKZJ5yn1` | | |
| 9 | `nDKuIW7sURXPGoRGmJzlINEi0Iu1` | | |
| 10 | `dlOP7p9SZsbQdByNDxo9Oo3UNLr2` | | |
| 11 | `XjxUGY4FtRdgYcWjUPpaaCb1suM2` | | |
| 12 | `Cy3zJrZltkQVbEb95BvxpCplQFH2` | | |
| 13 | `8IPRDFWbJcRK3hmS6ybkEAib0UU2` | | |
| 14 | `eHI66jbj3SenZNcVjCdEmjHEZvv1` | | |
| 15 | `6QxkPy5hztgY5gVwxuD7AGMGtku2` | | |
| 16 | `8dMS0In3DUhpLsiEMJEEjjEXDMz2` | | |
| 17 | `wpyugesIv8b18Upo7Pc9FkD3r6G3` | | |
| 18 | `f12fNfKKhocgzmoM7540gB3NkFk1` | | |
| 19 | `LXsYHq6iRcc43uF3gFdXj3CJagr1` | | |

## The 1 property + pg_number combo

| pg_id | pg_number | Known name | Notes |
|---|---|---|---|
| `waBJcohBXJMsoI6w276wOYJONP72` | 1 | | flat-grouping only for pg_number = 1 |

## Watch-items

- Only **Meridian** carries a name in code; the rest are bare IDs — map them to owner names when watching the bet.
- This list is **hardcoded**. Any new property that wants flat-grouping must be added here (or have BHK-typed rooms). Flag if this needs to become a property setting instead of a code list.
- The bet is additive: even for these properties, the bed-level rate/totals work without flat-awareness. This list only affects the **Unit-view grouping**.
