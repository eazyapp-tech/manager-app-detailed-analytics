---
title: DA-02 Cover Note — Nimit
date: 2026-08-08
tags:
  - rentok
  - collection
  - detailed-analytics
  - handoff
aliases:
  - Collection backend cover note
---
> [!INFO] What this is
> The message that accompanies [[DA-02 Collection — Handoff Sheet]] when it goes to Nimit for the backend build. Kept here so the reading map and the pre-answered decisions survive outside chat.

# Collection analytics: backend handoff

One doc, self-contained: [[DA-02 Collection — Handoff Sheet]] (this folder).

Your sections: **1 to 14, 16 to 18** plus the measured-figures appendix. Skip 15 and 19 (design's).

Read §17 (Build guidance) before writing anything. It carries the traps as outcome-plus-test, including the charges-off-exactly-once rule for credit payments, the cash-means-all-cash test, and the add-up tests that hold every card to one shared total.

The three older DA-02 docs (Brief, Ground-Truth Formula Map, Build Sheet) are marked superseded at the top; where they disagree with this sheet, the sheet wins. Two of their claims are now known wrong: caution-money adjustments are live in production since April, and the settlement destination records on the older payout system are real and usable, not to be ignored.

Four decisions pre-answered so you're not blocked, flag if you disagree:

1. **The whole screen pivots on the Paid Date / Due Date toggle** (§4), and in Due Date view all four adjustment kinds, deposit, advance, caution money and discount, count towards Collected & Adjusted. Leave any out and already-cleared bills land in Still Unpaid, which sends a manager chasing a tenant who owes nothing.
2. **Coming up exists here and only means anything in Due Date view.** Custom stops at today, the suite rule. About ₹5.2 crore is already received against future-dated bills, and this screen is the only place in the suite that money can be seen: Dues counts unpaid dues only, so a future bill already paid appears nowhere there.
3. **Bank-destination rows draw from whichever settlement system holds the record.** The newer system covers 160 properties ever; the older one names a destination for 8,326. A row falls back to the contact-support line only when its own record does not resolve.
4. **The Status tab's unlinked payments are excluded with one stated line, not an Unlinked row.** Ten payments out of 236,464 in July; the list's own Non Tenant filter reaches them.

The single most important open item is yours: **a third of July's online money has no settlement record in any system** (open item 1). The Unsettled tile would show it all as stuck money. Confirm whether no-record genuinely means not-settled before that tile ships; if it is a record-keeping gap, the tile lies at ₹48 crore scale.

Two drill families leave the screen: bill-side numbers (Billed, Still Unpaid, the trend's yellow segment) open the **Dues screen** carrying their window, and settled money opens the **FlexiPe screen**. Confirm FlexiPe can arrive pre-filtered, ideally to one account, before wiring those taps. The filters-to-add table inside §13 is the work list beyond the cards themselves; the settlement-status filter at the top of it also fixes the wipe problem where today's widget arrivals clear every other filter.

Everything else in Open items (§18) is either yours to confirm (the live broken deposit-adjusted widget, the Old Tenants filter's real meaning, the self-added permission) or already logged as shared suite work (the list naming its arriving filters).
