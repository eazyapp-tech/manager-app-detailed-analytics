---
title: DA-08 Cover Note — Ishika
date: 2026-08-08
tags: [rentok, inventory, occupancy, detailed-analytics, handoff, cover-note, design]
owner: Sanchay
---

> [!INFO] What this is
> The message that goes with the Inventory handoff sheet, kept here so the reading map survives outside chat.

# Inventory analytics — design handoff

Everything is in one document: [[DA-08 Inventory — Handoff Sheet]] (v2, on the suite template).

**Your sections: 3 and 4 for the vocabulary and screen behaviour, 5 to 13 for what each card shows, 16 for every state, and 20, which collects all 41 file fixes in one list.** Skip 14, 15 and 18, which are Nimit's.

Start with section 20. It is the shortest route into what changes.

## The one thing that reframes everything else

There are two Occupancy designs in the file, and the newer one ("Occupancy insight - done") is the build target. The older draft is not junk: five things were lost between the two rather than cut, and all five are back in the spec. The giveaway was that the newer version's empty states still carry the older version's card titles.

1. **Healthy states.** This is the only screen in the suite where an empty card is usually good news; the current copy congratulates a full property on having no data. Four cards need a genuinely different message (section 16).
2. **The not-set-up screen.** Nearly half of all properties have no rooms at all; seven separate empty boxes is the wrong answer to one situation.
3. **Agreements ending soon**, as its own card.
4. **The Semi-Vacant slice** in the Unit View donut, without which a partly filled room has nowhere to sit.
5. **The per-card insight line**: positive is plain text, negative carries a button.

## Three things undrawn and needed

- **The View all sheet** (section 6): only the link exists today.
- **Bed View**: every card is drawn once, in Unit language. Labels, the "Total Beds" header and the donut slices all need their Bed View versions.
- **Coming up**: the forward filter setting, with a date picker. Section 4 says how each card reads when it is on.

## Decisions worth knowing

**The toggle is Bed View / Unit View**, Bed View default, and "unit" is a defined word: whatever is let as one lettable space. Labels switch with the toggle; "Units" above a bed count is a mislabel.

**Two change chips are inverted.** Vacant and Under Notice go green when rising, and both are bad news. Polarity is per tile here, including a neutral grey chip the shared component does not have yet.

**The tab row is settled suite-wide** and the file's row is stale: Financial · People (Leads · Bookings · Tenants · Old Tenants) · Inventory · Complaints. Complaints keeps its name; the floating Leads tab goes. Fix 13 in section 20.

## The info icons

Nobody in the suite has written what any info icon says. Section 3 is written to double as that content, and this screen is the natural place to start: it defines the vocabulary the other seven screens borrow.
