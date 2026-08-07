---
title: DA-08 Cover Note — Ishika
date: 2026-08-07
tags: [rentok, inventory, occupancy, detailed-analytics, handoff, cover-note, design]
owner: Sanchay
---

> [!INFO] What this is
> The message that goes with the Inventory handoff sheet, kept here so the reading map survives outside chat.

# Inventory analytics — design handoff

Everything is in one document: [[DA-08 Inventory — Handoff Sheet]].

**Your sections: 4 to 13 for what each card shows, 16 for every state, and 20 which collects all 34 file fixes in one list.** Skip 3, 14, 15 and 18 — those are Nimit's.

Start with section 20. It is the shortest route into what changes.

## The one thing that reframes everything else

There are two Occupancy designs in the file, and the newer one — "Occupancy insight - done" — is the build target. But **the older draft is not junk, and several things in it were lost rather than cut.** The giveaway: the newer version's empty states still carry the *older* version's card titles, and include an empty state for a card the newer version does not have. That is drift, not a decision.

So five things from the draft are coming back, each for a reason:

1. **Healthy states.** This is the only screen in the suite where an empty card is usually *good news* — no vacant rooms means the property is full. The current copy congratulates a full property on having no data ("Vacancy duration by days will appear here once rooms are empty"). Four cards need a genuinely different message.
2. **The not-set-up screen.** Nearly half of all properties have no rooms at all. For them, seven separate empty boxes is the wrong answer to one situation that needs one sentence and one button.
3. **Agreements ending soon**, as its own card.
4. **The Semi-Vacant slice** in the Unit View donut — without it, a partly-filled room has nowhere to sit and Unit View stops adding up.
5. **The per-card insight line**, with its rule: a positive insight is plain text, a negative one gets a button.

## Three things that are undrawn and needed

- **The View all sheet.** Only the link exists today; the sheet itself has never been drawn. Contents are in section 6.
- **Bed View.** Every card is drawn once, in Unit language. All of it needs its Bed View version — tile labels, the "Total Beds" header, the donut slices.
- **Coming up.** A new forward setting on the time filter, with a date picker. Section 4 describes how each card reads when it is on.

## Two decisions worth knowing about

**The toggle is Bed View / Unit View**, and "unit" is now a defined word — whatever is let as one lettable space, so a flat let whole is one unit and a flat let room by room is several. Tile labels have to switch with the toggle; "Units" above a bed count is a mislabel.

**Two change chips are inverted.** Vacant and Under Notice both go green when they rise, and both are bad news. This screen is the first with no single good direction, so polarity is per tile — including one neutral grey chip, which the shared component does not have today.

## Nobody has written the info icons

Every card carries one and no screen in this project has content for any of them. Section 3 of the sheet is written to be reusable as that content — plain definitions, no jargon. Worth doing here first, since this screen invents most of the vocabulary the other seven will borrow.
