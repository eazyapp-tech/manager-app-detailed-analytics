---
title: DA-04 Cover Note — Nimit
date: 2026-08-08
tags:
  - rentok
  - expenses
  - detailed-analytics
  - handoff
aliases:
  - Expense backend cover note
---

> [!INFO] What this is
> The message that accompanies [[DA-04 Expense — Handoff Sheet]] when it goes to Nimit for the backend build. Kept here so the reading map and the pre-answered decisions survive outside chat.

# Expense analytics: backend handoff

One doc, self-contained: [[DA-04 Expense — Handoff Sheet]] (this folder).

Your sections: **1 to 12, then 14 to 16**, plus the measured figures at the end. Skip 13 and 17, they are design's.

Read section 15 (Build guidance) before writing anything. Every rule there is an outcome with a *Test it:* line QA can run, and several exist because the obvious way to build the number produces a wrong one.

The three older DA-04 documents are marked superseded at the top; where they disagree with this sheet, the sheet wins. Their code anchors are still fine for finding where things live.

Decisions already settled so you are not blocked:

1. **Permissions need nothing new.** The suite ruling is that each analytics tab follows the permission of the records it describes, so whoever can open the expense list reads this screen. Nothing to invent, nothing to re-assign.
2. **No forward window, and Custom stops at today.** Measured: 45 future-dated expenses out of 340,524. Nothing to build for the future here.
3. **Others sheets ship day one.** Same screen, same permission. The deeper-page access question is suite-wide and does not hold this build.
4. **The Still owed to staff tile opens Team Passbook as it stands today**, carrying nothing. A running balance has no window to hand over, so Passbook needs no change to receive it.

What does need building outside the analytics module, all listed with their numbers in section 11: two drawer groups the list already understands (category, paid by), four new filters it does not have (nothing attached, payment mode, fund source, quarantined rows), and the named-slice widening every screen in the suite needs.
