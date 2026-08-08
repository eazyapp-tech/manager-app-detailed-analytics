---
title: DA-08 Cover Note — Nimit
date: 2026-08-08
tags: [rentok, inventory, occupancy, detailed-analytics, handoff, cover-note]
owner: Sanchay
---

> [!INFO] What this is
> The message that goes with the Inventory handoff sheet, kept here so the reading map survives outside chat.

# Inventory analytics — backend handoff

Everything is in one document: [[DA-08 Inventory — Handoff Sheet]] (v2, on the suite template). It is self-contained; you should not need to open anything else.

**Your sections: 1 to 15, 17, 18, 19, and the measured-figures appendix. Skip 16 and 20**, which are Ishika's.

**Read section 18, Build guidance, before writing anything.** Every non-obvious rule carries a test you can run.

## Three things to know before you estimate

**One prerequisite blocks whole cards, not single numbers.** Nothing in the product records when a room became empty. Vacancy age, never rented, Days to Fill and the past-period revenue-loss figure all wait on it, drills included. Size this first; it probably decides the shape of the whole delivery.

**A period number means an average of days, and nothing today produces that.** A tenant who stayed three days of a month weighs three days, not a month, or a property that stood half empty reports as full. Section 18 states it with a test. The older design documents assume this needs a new table and a nightly job; I have marked them superseded, because my read is that the history already carries what is needed. **That is a read, not a finding.** If you look and disagree, say so early; it moves the estimate a lot.

**Properties not yet migrated are the fallback, not the target.** Roughly 1 in 100 properties has real bed records today, but everyone is migrating and the share climbs every cohort. Build the exact bed-level version first and the approximation second.

## Two decisions pre-answered so you are not blocked

Both are mine; flag a disagreement rather than working around it.

1. **The rate is bed-based in both views**, and its label says so. A unit-based rate would call a four-sharing room with one tenant fully occupied.
2. **Booking numbers open the bookings list, not the rooms list.** "Booked" is stricter here than there; the drill goes where the number can be honoured until the two lists agree.

## What has been settled since v1

The tab row, the gating permission (this screen follows the rooms-list permission; section 15) and the old Dues-versus-Collection under-notice question are all closed by suite rulings. Section 19 is now ten genuinely open items; the ones needing an engineering answer are whether a removed room still pads the rentable count, the missing 60-day agreement filter, and the room-link read on the no-space booking filter.

## Older documents

The five other DA-08 documents in this folder carry superseded banners listing what this sheet overrides. Worth reading for how the thinking got here; their conclusions have been overtaken.
