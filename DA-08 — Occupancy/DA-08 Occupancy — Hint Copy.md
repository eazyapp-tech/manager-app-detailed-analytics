# DA-08 Occupancy, Hint Copy

The words behind every (i) on the Occupancy tab. Two columns, matching the two text fields the sheet has: **What is this?** and **How is this calculated?** A dash means the definition already says everything. **Good to know** is one line per block, for the block-level line when the sheet gets one.

Checked against the code at rentok-backend `0e8cc713b` and the live account, Mumbai group, 22 properties, 27 Aug, in Now and This Month.

## Now against a period, before anything else

This tab's date filter does something no money tab does: it changes what kind of number you are looking at.

| You pick | What every period number means |
|---|---|
| Now | A live count, this moment. Matches the home screen and the rooms list. |
| Any period | The average across that period's days. A bed filled for half the month counts as half a bed. |

**On the live account this is a 9-point difference on the same screen:** Now shows 88% occupied, This Month shows 79%, because beds filled up through August. Neither is wrong. One is today, the other is the average day. This one sentence prevents more tickets than everything else on this tab.

**And two blocks on the same screen keep different clocks:** the Overview follows the filter, while Occupancy Status is always right now. So on a period, the two can show different rates one above the other. Their faces say so; the sheets below say it again.

The Bed and Unit toggle changes what is counted where a room holds several beds. The Occupancy Rate is always measured by beds, in both views.

## 1. Overview Snapshot

Five tiles. Follows the date filter; the title carries the mode, "(Now)" or the month.

> **Good to know:** Occupied plus Vacant always equals Rentable, in Now and on any period.

| Tile | What is this? | How is this calculated? |
|---|---|---|
| Rentable Beds | Beds you can rent: the total minus those switched off. | - |
| Occupied Beds | Beds with a tenant living in them. | On a period, this is the average across its days: a bed filled half the month counts as half. |
| Occupancy Rate | How full you are: occupied out of rentable. | Always by beds, even in Unit view. A past period is measured against today's rentable count. |
| Vacant Beds | Rentable beds with no tenant. | Rentable minus Occupied. |
| Under Notice Beds | Beds whose tenant has given notice to leave. | Notices a manager has accepted. Always as of today, whatever dates you pick. |

## 2. Occupancy Status

The donut, the chips and the two money tiles. Always right now; the date filter does not change this block.

> **Good to know:** always shows this moment, so on a period it can differ from the Overview above it. That is the two clocks, not a bug.

| Card | What is this? | How is this calculated? |
|---|---|---|
| Occupied | Beds or units with a tenant living in them. | - |
| Semi-Vacant | Rooms part-filled: some beds taken, some free. | Unit view only. |
| Vacant | Empty rentable beds or units, split into booked and available. | Booked means a booking is against it; available means still open to let. |
| Disabled for Rent | Beds or units switched off for rent. | Not counted in Rentable. |
| Under Notice | Tenants who have given notice. | Manager-accepted notices. |
| Booked Beds | Beds with an upcoming booking. | In Unit view this becomes units where every bed is booked. |
| Under Notice with Booking | Beds where the leaving tenant is already replaced by a booking. | - |
| Overbooked occupancy | More tenants than beds. | When this is above zero, the parts can add to more than the total. |
| Avg. Rent per Occupied | The average rent of one filled bed or unit. | Uses the rent on each tenant's profile. Tenants with no rent set are left out, so thin rent records read low. |
| Avg. Revenue per Rentable | The same rent spread across every rentable bed or unit, filled or not. | Lower than Avg Rent whenever anything is vacant: the gap is what vacancy costs. |

## 3. Vacant Room Status (Live)

| Card | What is this? | How is this calculated? |
|---|---|---|
| Vacancy aging | Empty rooms grouped by how long they have been empty since the last move-out. | Rooms only: a room with even one tenant does not appear here. Never rented means it has never had a stay; Unknown means it was let before tracking began. |

## 4. Upcoming Vacancy

| Card | What is this? | How is this calculated? |
|---|---|---|
| Upcoming move-outs | Rooms freeing up soon, grouped by the day the last tenant leaves. | Counts rooms that empty fully; a room where only some beds free up is not counted. Manager-accepted move-outs only. |

## 5. Agreements Ending Soon

| Card | What is this? | How is this calculated? |
|---|---|---|
| Agreements ending | Long-term agreements ending in the next 30, 60 or 90 days. | When no end date is recorded, one is estimated: the agreement period, else the property's default, else 11 months. |

## 6. Occupancy Trend

Has its own months dropdown.

> **Good to know:** Occupied and Vacant add up to each month's rentable total, so the rate never reads above 100%.

| Card | What is this? | How is this calculated? |
|---|---|---|
| Occupancy trend | How full you were, month by month: the average across each month's days. | Measured against the rooms that existed that month. Uses its own months dropdown, not the date filter above. |

## 7. Where am I losing money?

Follows the date filter: Now prices today's empty beds, a period prices the average across it.

| Card | What is this? | How is this calculated? |
|---|---|---|
| Revenue lost to vacancy | Empty beds by room type, and the monthly rent they would earn if filled. | Priced from the rent of tenants in the same room. The "based on X of Y" line says how many empty beds could be priced; the rest count as ₹0. |

## 8. Property Wise Occupancy

| Card | What is this? | How is this calculated? |
|---|---|---|
| Occupancy by property | Each property's occupied out of rentable, most empty space first. | Needs two or more properties. Always right now. |

---

## The pointers are checked, not guessed

Read off the live account on 27 Aug, in Now and in This Month.

| Claim | Checked |
|---|---|
| Occupied plus Vacant equals Rentable, both modes | Now: 1341 + 187 = 1528. This Month: 1212 + 316 = 1528 |
| A period is an average, not a reading | Same screen, same minute: Now 88%, This Month 79% |
| Status keeps its own clock | On This Month, Overview showed 79% while Status below it stayed at 88% |
| Total against rentable | Total Beds 1557 = 1528 rentable + 29 disabled |
| Trend agrees with the Overview's average | Trend's Aug bar reads 1.2K occupied; the Overview's Aug average is 1212 |
| Under Notice ignores the filter | 153 in Now and in This Month, faced "as of today" |
| The faces already carry their clocks | "Overview Snapshot (Aug)", "as of today", "by rooms · As of today", "when confirmed move-outs happen" all render live |

## Launch risks on this tab

1. **Where am I losing money prices 85% of vacant beds at ₹0** (finding F78): an empty room has no tenants to take a rent from, and the fallback to the group's average was documented but never built. The emptiest property, losing the most, reads ₹0. The coverage line is the only honest part. **This card should not launch before F78 is fixed.** Its bars also all draw at zero width (F79).
2. **The period rate divides by today's bed count** (F68), so on ~4,000 properties that added rooms recently, the Overview's period rate reads lower than the trend bar under it for the same month.
3. **A custom window ending in the future shows today's numbers under tomorrow's label** (F83).
4. **Unapproved bookings count in the booked layer** (F67); the Vacant card's "booked" split reads higher than approved reality. The copy above says "a booking is against it" deliberately; it becomes "a confirmed booking" when F67 lands.
5. **Pending evictions are invisible on the whole tab** (F66): a tenant who has asked to leave, unaccepted, appears nowhere. The Under Notice sheets say "manager-accepted" so the number is at least honest about itself.
6. **A missing property falls back to made-up scaffold numbers** in nine places here (F84).

**Cells that flip when fixes land:** Vacant's "booked" becomes confirmed-only (F67); the period rate line about today's rentable count drops (F68); more than half of agreement end dates stop being estimates as real dates are recorded (F77).
