---
title: DA-04 Cover Note — Nimit
date: 2026-08-07
tags:
  - rentok
  - expenses
  - detailed-analytics
  - handoff
aliases:
  - Expense backend cover note
---

> [!INFO] What this is
> The message that accompanies [[DA-04 Expense — Handoff Sheet]] when it goes to Nimit for the backend build. Kept here so the reading map and the two pre-answered decisions survive outside chat.

# Expense analytics — backend handoff

One doc, self-contained: [[DA-04 Expense — Handoff Sheet]] (this folder).

Your sections: **1–11, 13, 14, 16** + the measured-figures appendix. Skip 12 and 15 (design's).

Read §14 (Build guidance) before writing anything — it has the traps, including the two paths in the old module that must not be copied.

The three older DA-04 docs are marked superseded at the top; where they disagree with this sheet, the sheet wins. Their code anchors are still fine for finding where things live.

Two decisions pre-answered so you're not blocked — flag if you disagree:

1. The screen gates on the **expense-viewing permission** (§11). The analytics-vs-list permission question stays open suite-wide, not here.
2. **Others sheets ship day one** — same screen, same permission. The deeper-page access question is app-wide and doesn't hold this build.

Everything else in Open items (§16) is either yours to own (the four new list filters, the Passbook period check) or logged suite-wide.