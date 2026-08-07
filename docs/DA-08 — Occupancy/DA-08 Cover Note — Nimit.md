---
title: DA-08 Cover Note — Nimit
date: 2026-08-07
tags: [rentok, inventory, occupancy, detailed-analytics, handoff, cover-note]
owner: Sanchay
---

> [!INFO] What this is
> The message that goes with the Inventory handoff sheet, kept here so the reading map survives outside chat.

# Inventory analytics — backend handoff

Everything is in one document: [[DA-08 Inventory — Handoff Sheet]]. It is self-contained — you should not need to open anything else.

**Your sections: 1–15, 17, 18, 19, and the measured figures at the end. Skip 16 and 20** — those are Ishika's.

**Read section 18, Build guidance, before writing anything.** It holds the traps, and three of them will cost real rework if they are found late.

## Three things to know before you estimate

**One prerequisite blocks whole cards, not single numbers.** Nothing in the product records when a room became empty. Vacancy age, never-rented and Days to Fill all need it, and none of them can be built — or drilled into — until it exists. That is one full card plus parts of two others. Size this first; it probably decides the shape of the whole delivery.

**A period number means an average of days, and nothing today produces that.** A tenant who stayed three days of a month has to weigh three days, not a month — otherwise a property that stood half empty reports as full. Section 18 states it with a test you can run. The older design documents assume this needs a new table and a nightly job; I have marked them superseded, because my read is that the history already carries what is needed. **That is a read, not a finding** — if you look and disagree, say so early, because it moves the estimate a lot.

**Properties not yet migrated are the fallback, not the target.** Roughly 1 in 100 properties has real bed records today, but everyone is migrating and the share climbs every cohort. Build the exact bed-level version first and the approximation second, not the other way round.

## Two decisions pre-answered so you are not blocked

Both are mine, and I would rather you flag a disagreement than work around them.

1. **The rate is bed-based in both views.** Unit View re-slices everything under it, but the headline percentage stays beds and says so on its label. A unit-based rate would call a four-sharing room with one tenant fully occupied.
2. **Booking numbers open the bookings list, not the rooms list.** "Booked" is stricter here than on the rooms list, so drilling there would show a manager more rows than the number promised. The two should end up meaning the same thing; until they do, the drill goes where the number can be honoured.

## Open items that are yours or nobody's

Section 19 has the full list. The ones that need an engineering answer rather than a product one: which permission actually gates this screen, whether a removed room still counts toward what is rentable, and the tab row disagreeing with what the app actually serves — that last one affects all eight analytics screens, not just this one.

## Older documents

The five DA-08 documents in this folder now carry a superseded banner listing the eleven points this sheet overrides, plus four things in them that are stale on fact. They are still worth reading for how the thinking got here; their conclusions have been overtaken.
