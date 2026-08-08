---
title: DA-04 Cover Note — Design
date: 2026-08-08
tags:
  - rentok
  - expenses
  - detailed-analytics
  - handoff
aliases:
  - Expense design cover note
---

> [!INFO] What this is
> The message that accompanies [[DA-04 Expense — Handoff Sheet]] when it goes to design. Kept here so the reading map survives outside chat.

# Expense analytics: design handoff

One doc, self-contained: [[DA-04 Expense — Handoff Sheet]] (this folder).

Your sections: **4 to 10, 13, 17**. Section 17 is the work list: 41 numbered fixes grouped Wrong, Missing, Remove, Decide. Section 13 holds the empty-state copy, the zeros router and the one healthy state word for word, so nothing needs writing from scratch.

Before opening the file: every value drawn in it is placeholder, and the section is titled "done" while 41 fixes are outstanding. Take shape from the file, never a number.

The biggest pieces of new drawing, in rough size order:

1. **The Overview row rebuilt.** Five tiles are drawn; the screen needs four different ones, and two of them (Still owed to staff, Number of expenses) exist nowhere in the file. Two tiles carry a fixed-window label where the others put their chip: "This financial year" and "as of today".
2. **The View all sheet.** The link exists in the header and opens nothing. Section 6 lists its eleven rows, including two that appear only when they have something to show.
3. **The states.** The Restricted lock, loading skeletons, the failed card, the healthy state, an Overview empty state, title lines and a button on every empty state that invites an action.
4. **The chip set.** Increase is red on the money tiles, the count tile takes a neutral grey the shared component does not have yet, chips hide entirely on All Time, and the unfinished-period marking is not drawn anywhere.

Three smaller things worth knowing before you open the file:

- **The time filter becomes a dropdown stopping at today.** It is drawn as a stepper labelled "Today", which is not one of the options, and the Custom picker must not offer future dates. This screen has no forward window at all.
- **Two whole cards come out**: "Where is my property losing money?" and Team Passbook. Their reasons are in the list.
- **Eleven info icons have no content.** Section 17 names the three that most need it: Still owed to staff, the fund rows, and the bill-or-receipt figure, because managers have never seen those numbers before.
