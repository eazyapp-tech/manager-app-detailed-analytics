---
title: DA-01 Cover Note — Nimit
date: 2026-08-08
tags:
  - rentok
  - dues
  - detailed-analytics
  - handoff
aliases:
  - Dues backend cover note
---
> [!INFO] What this is
> The message that accompanies [[DA-01 Dues — Handoff Sheet]] when it goes to Nimit for the backend build. Kept here so the reading map and the two pre-answered decisions survive outside chat.

# Dues analytics — backend handoff

One doc, self-contained: [[DA-01 Dues — Handoff Sheet]] (this folder).

Your sections: **1–16, 18–20** + the measured-figures appendix. Skip 17 and 21 (design's).

Read §19 (Build guidance) before writing anything, it carries the traps as outcome-plus-test, including the junk-amount cap and the one-row rule for old tenants.

The three older DA-01 docs (Brief, Ground-Truth Formula Map, Build Sheet) are marked superseded at the top; where they disagree with this sheet, the sheet wins. Their schema notes are still fine for finding where things live.

Two decisions pre-answered so you're not blocked, flag if you disagree:

1. **Overdue Breakup (§10) is Live**, not scoped to a time filter. An earlier design had it following "This Month"; production data showed that leaves the ageing buckets structurally empty for most of every month, since a bill due the 1st can't be more than a week overdue by the 7th. Ageing answers "how stale, as of today," which is a Live question.
2. **Old tenants' dues appear in exactly one place**, the Old tenants row inside Dues Breakdown → Tenant status (§9). Every other number on the screen, including the six Overview tiles, excludes them entirely. This isn't a gap to fix by including them elsewhere.

One more worth flagging directly: the two **Received** figures (§8, §12) open the **Collection screen's** list, not this screen's. First cross-tab drill in the suite, tap lands where the records live even off-tab. Confirm the Collection list can accept a due-date window and, for §12, a deposit-category filter, before wiring the tap.

Everything else in Open items (§20) is either yours to confirm (the unnamed creator code, the drill-route permission check, list pagination) or a product call already logged (the Added-By row count, item 5).
