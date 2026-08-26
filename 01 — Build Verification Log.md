# Build Verification Log

Analytics was specified across six handoff sheets, then built. This log records what shipped
against what we asked for, block by block, and what was ruled on each difference.

**People in this log.** The owner is Sanchay, who rules on every difference. Vivek built the code
and wrote the calculation guide. "This review" is the author of this log, who also wrote the sheets.

**How to use it.** Vivek: start at [Open items](#open-items), then read the F entries it points to.
Sanchay: start at the same table, then [Decisions ruled](#decisions-ruled). Nobody needs to read the
whole log to find their work.

Companion to [[00 — Manager App Analytics Tracker]], which records how the sheets were written.
Product ideas that are not defects live in [[02 — Suggestions Register]].

Last audited end to end on 2026-08-24, before the first commit: every cited line re-checked, every
quote re-read, two independent reviewers run. Where the audit found an earlier claim wrong, the entry
says so rather than quietly replacing it. Occupancy was added the same day and audited the same way.

---

## What is in here

| Section | Answers |
|---|---|
| [How this review works](#how-this-review-works) | What gets compared, what the verdicts mean, the words used throughout |
| [What is in scope](#what-is-in-scope) | Which blocks are checked, and what is deliberately left out |
| [Status board](#status-board) | How far the review has got, and the sweeps every remaining tab gets |
| [Open items](#open-items) | Everything still waiting, by who it waits on |
| [Findings](#findings) | Every difference found, F1 to F104: what is wrong and the fix |
| [Decisions ruled](#decisions-ruled) | What the owner decided and why, D1 to D32, dated |
| [Sheet edits made](#sheet-edits-made) | Exactly what changed in the handoff sheets because of this review |
| [Doc fixes owed to Vivek](#doc-fixes-owed-to-vivek) | Errors in the calculation guide, harmless to the product |
| [What shipped better than we specified](#what-shipped-better-than-we-specified) | Things to write back into the sheets as rules |
| [Passes worth recording](#passes-worth-recording) | What was checked and found right, so nobody re-checks it |

---

## How this review works

Three sources, and only one is authority on what shipped:

| Source | What it is |
|---|---|
| The handoff sheet | What we asked for |
| A line number in this log | **Every `file.ts:NNN` in here resolves against commit `3a13e08ac`**, the state of the code when each tab was reviewed. Line numbers move whenever anyone edits a file, so each finding also names the function or block it is about, and that never moves. If a cited line does not say what the entry says, check the function, not the number (D25) |
| Vivek's calculation guide | What Vivek says the code does. Owned by him in the backend repo at `docs/analytics/ANALYTICS_GUIDE.md`; a read-only copy sits here as [[03 — Analytics Calculation Guide (Vivek)]] with the commit it was taken from. The Google Docs export circulating as a file is lossy, it ate the `$` from every SQL placeholder in the appendix, so never cite the export |
| `src/v1/analytics/` in rentok-backend | What the code actually does |

Doc against doc would pass a block where both documents agree and the code does neither, so every
block is checked all three ways.

**The guide has two halves and both count.** Its prose describes each block; its appendix, "the actual
queries", restates the SQL per service. The Dues pass first read only the prose, and the appendix was
read afterwards, on 2026-08-24, when the owner asked whether it had been used. It overturned nothing,
independently corroborated F2's package defect, and its pointer to `i.added_by` led to F26. **Read
both halves per tab.** Where a claim depends on live data it is measured on production,
test and deleted properties excluded, and the query's scope is stated next to the number. Where an
Expense figure is a share of all twelve-month spend, the base is ₹185.1 crore; the table is written to
continuously, so separate runs of the same query move it by a few lakh, which is the data moving and
not an error.

**A difference is not automatically a defect.** Some were decided during the build with the owner in
the room. So each difference gets a view on which version is right, and the owner rules.

**Where the reasoning lives.** An F entry says what is wrong and the fix. A D entry says what was
ruled and why. Reasoning is written once, in the D entry, and the F entry points to it.

**Every pointer carries its own meaning.** A reference to another entry always brings a few words of
what is there, so a sentence can be read where it sits: "D26, which took the eleven-month guess off
this tab", not "per D26". The numbering is for finding things again, never for carrying the meaning.
Nobody reads this log in the order it was written, so the small repetition is deliberate.

**Every finding has the same four-field header:**

| Field | Values |
|---|---|
| Verdict | Pass · Build gap · Accepted change · Specification gap · Doc error · Withdrawn |
| Status | Open · Ruled · Closed |
| Owed by | Vivek · Owner · Sheet · Nobody |
| Note | Free text, when one is needed: which D settles it, what blocks it, what closed it |

Verdicts: a **build gap** is where the sheet asked for something and the code does something else.
An **accepted change** is where the code differs and is right, so the sheet changes. A
**specification gap** is where the code is fine and the sheet was silent or contradicted itself. A
**doc error** is where the code is right and the guide describes it wrong. **Withdrawn** means raised,
then shown to be wrong or superseded, kept so the reasoning survives.

Status: **Open** until the owner decides, **Ruled** once he has and work remains, **Closed** when the
edit or ticket is done.

**Words used throughout, so entries do not re-explain them:**

| Word | Meaning here |
|---|---|
| Tenant status | 0 moved out · 1 living here · 2 booking · 3 lead · 4 invite · 5 to 8 deleted states |
| Invoice status | 0 unpaid · 1 paid · 2 part paid · 3 refunded · 4 written off as a loss |
| The base pool | The set of unpaid bills every Dues number starts from: unpaid, active, at least ₹1, payer at status 1 or 2 (sheet §3). Built by `DuesListHelper.buildBaseQuery`, shared with the homescreen |
| Adjustment payment | A payment recorded in mode 211 (deposit), 288 (advance) or 291 (caution money): a bill cleared using money already held |
| The collection base | The set of successful, active payments joined to the paid bills they cleared, shared with the homescreen and the collections list. Built in `collectionService.ts` as `COLL_BASE` |
| Credits | The table where the payment flow records a discount the owner gives at payment time. The bill keeps its full amount; the waived part is a credits row on the payment |
| Confirmed booking | A booking at status 2 that is approved, or on a property where bookings need no approval. The owner ruled those two are the same thing (D1). The code that decides it is at `tenantService.ts:352` |
| View all screen | The panel that opens from the Overview strip header, sheet §6. Called a "screen" here so it is never confused with the handoff sheet |
| Team passbook | The record of money each staff member holds or is owed, per property. Rows carry a fund and a category. `team_member_transactions` |
| Fund | Whose money paid for something, recorded on the passbook row, not on the expense: `NPNAF` collected money · `PF` the staff member's own pocket · `AF` petty cash · `EF` exchange between staff. These are not the screen's four fund rows: FlexiPe is found by its wallet link and writes no passbook row at all, and `EF` never reaches the screen |
| Balance type | Which way a passbook row moves the balance: 1 money in, 2 money out. The passbook screen and the Expense tile read the same rows with opposite signs, on purpose |
| FlexiPe | A wallet spending leaves through. An expense paid this way carries a wallet link and no passbook row, so it is identified by that link and never by its payment mode |
| The two structures | How a property records its space. On the **new structure** every bed is its own record and a tenant is linked to one; on the **old** one a room carries a sharing number and a tenant carries the room's name as text. Almost every Occupancy query is written twice, once per structure |
| The rooms widget | The one live count of space, `RoomListHelper.fetchAllWidgetCounts`, shared by the homescreen, the rooms list and every Now number on Inventory. Filter codes name each count: 1402 under notice, 1406 booked rooms, 1408 vacant rooms, 1409 occupied rooms, 1415 total beds, and so on |
| Stay history | One row per stretch of time a tenant occupied a room or bed, with a start and an end. Every period average and the whole trend are rebuilt from it. A row flagged pending is a move scheduled for the future |
| Day-averaged | What a period means on Inventory and nowhere else in the suite. "78% last month" is 78% of an average day of it, not the last day of it |
| Rentable | Space that can be let: total, minus space switched off for rent, minus rooms that declare no capacity |
| Direct add | A tenant created straight onto the tenant list, never a booking. On an auto-accept property the data cannot tell one from an auto-accepted booking, which is what F93 is about |

---

## What is in scope

**43 blocks across five tabs**, all wired to real queries. Vivek's guide covers all 43, one for one.

| Tab | Blocks | Wired | Sample data |
|---|---|---|---|
| Dues | 9 | 9 | 0 |
| Collection | 7 | 7 | 0 |
| Expense | 5 | 5 | 0 |
| Occupancy (Inventory) | 8 | 8 | 0 |
| Tenant | 14 | 14 | 0 |
| Issues (Complaints) | 11 | 0 | 11 |

**Deliberately excluded, known and agreed:**

- **Issues / Complaints is not built.** The tab is declared in `masterConfig.ts` with 11 blocks in
  `registry.ts`, every one of them sample data. DA-10 closed on 2026-08-18 with 79 owner rulings and
  none of it is wired. Not re-flagged per block.
- **Redirection is not built.** No number opens its own records yet. The tap matrix in every sheet is
  unimplemented by design for now. Not re-flagged per block.
- **Old Tenants, Bookings and Leads were never specified**, so their absence is expected, not a gap.

A block being wired does not prove every field inside it is. That is checked per block.

---

## Status board

| Tab | Blocks | Reviewed | Findings | State |
|---|---|---|---|---|
| Dues | 9, plus the View all screen | All | 26 | Complete |
| Collection | 7, plus the View all sheet | All | 25 | Complete; re-verified against the fix commits |
| Expense | 5, plus the View all sheet and the Others sheet | All | 18 | Complete; fixes with Vivek |
| Occupancy | 8, plus the View all sheet | All | 20 | Complete; fixes with Vivek |
| Tenant | 14, plus the View all sheet | All | 15 | Complete; ruled same day, D26 to D32; fixes with Vivek |

**All five tabs are reviewed, and the owner has ruled on everything this review raised.** Tenant returned 15
findings, F90 to F104. Two are visible on almost every property: the Approved Bookings tile is really
a count of tenants added rather than bookings approved (F93), and three quarters of the Already expired bar rested on dates the product invented (F100,
settled by D26, which moved those tenants into their own stated group). One property's junk settings
value crashes the whole Tenant page (F90).

**Seven rulings came out of this tab, D26 to D32.** Three closed things this review had raised: no-term tenants get their
own stated group instead of the red expired bar (D26), Renting Type hides rather than showing the
stay split under a false name (D27), and the suite renames the parent group to Under eviction,
settling S3, his own rename suggestion from 23 August, which was parked until all five tabs were
done (D28). Four came from the owner reading the built screen and asking for
better: Journey grows to six rows in two named groups (D29), a bucket outside a run of days hides at
zero (D30), every agreement expiry label carries its own days (D31), and stay type and renting type
get opposite showing rules (D32). What remains open waits on Vivek's answers, on the fix lists below, and
on the register's proposals.

**Sweeps to run on every remaining tab.** Each was born from a finding on a tab already checked, and
the entry it came from is named beside it.

| Sweep | Born from |
|---|---|
| Does the code implement every rule the sheet's own base section states, rule by rule | F85 |
| Does a number agree with the surface its drill lands on | F86 |
| Is the chip's comparison figure worked out as it stood on that date, or as it stands today | F5 |
| Is each chip's colour right for what that tile measures | F6 |
| Does the Current FY option end at today | F9. Already answered: only Dues and Collection get it wrong |
| Does every block that ignores the top filter say so on its face | F18 |
| Does any filter carry exactly one option | F16 |
| Does every hiding rule in the sheet have a `hidden` return in the code | F23 |
| Does every zero go somewhere: empty, healthy, or hidden | F24 |
| Does any block match a free-named category by exact text | F19 |
| Does any forecast read a status code as if it already contained the approval step | F2, F4 |
| Does any forecast leave out a tenant filter altogether | F2 |
| Does every column the code sums actually hold data on production | F40 |
| Do all rows of one card count on one clock | F40 |
| Does a tile keep its chip on both sides of a view toggle | F31 |
| Do a card's tabs still sum to the card's total in every view | F36 |
| Is a number clamped so it can never go below zero, where a negative is the real answer | F55 |
| Does every card on a screen end its window on the same day | F63 |
| Is an empty state defined in the code and never sent | F64 |
| Does any of our own code save an internal string that ends up on a bar the manager reads | F62 |
| Is a helper written, correct, and never called | F65 |
| Does a bar's width or share come from a number already formatted for display | F79 |
| Where a rule bounds which bars may be coloured, does the code honour the bound | F73 |
| Does a fallback the sheet and the guide both describe actually exist in the query | F78 |

---

## Open items

Everything still waiting, by who it waits on. An item leaves this table when its entry is Closed.

**Collection has been re-verified against the two fix commits**, `4bce93d21` "payment settlement
logic changed" and `a2d839ea1` "collection due view all sheet data", on 2026-08-25. What they did:
refunds now come off collected money, a rule the sheet always carried and this review missed (F85);
the View all sheet now follows the toggle, closing F44 (F89); the destination rows now cover nearly
all settled money, closing F42's coverage but overstating the total by 14% (F87); and "settled" was
redefined to include Flexi Pay balances (F88). The refund fix moved Total Collection off the
collections list it drills into without landing on the homescreen figure, so the screen now agrees
with neither (F86). Two of the four need an owner ruling before Vivek does anything else on this
tab.

**Waits on Vivek, Dues and Collection**

| Item | One-line action | Entry |
|---|---|---|
| Current FY tile | Show `fy_ytd` instead of `current_fy`, in both `getOverview` and `getViewAll` | F1, F25 |
| Current FY filter option | End at today instead of 31 March, in `duesService` and `collectionService` | F9 |
| Projected Due, rent half | Count living tenants and confirmed bookings | F2 |
| Projected Due, package half | Add a tenant filter: living tenants and confirmed bookings | F2 |
| Projected Due, rent day | Fall back the way `chargeRentTenants` does, per property setup | F3 |
| Upcoming Dues | Same three fixes, second copy of the query | F2, F3, D8 |
| Upcoming Dues | Add the other configured types; extend to the chosen end date on a forward window | D8, D9 |
| Both Dues chips | Comparison figure as it stood on that date, not as it stands today | F5 |
| Both Dues chips | Red for up, green for down; copy any sibling tab's chip code | F6 |
| Dues Breakdown, Collection Breakup | Three active bars: Active, no eviction · Eviction pending · Eviction approved | D6 |
| Overdue Breakup | Rename "By Amount" to "By ageing"; remove the one-option dropdown; add "As of today" | F15, F16, F18 |
| Deposit Dues | Match categories case-insensitively and trimmed | F19 |
| Deposit Dues | Say what `net_amount` means against `amount`; then add the departed-tenants line | F21, D10 |
| Deposit Dues | Add "As of today" | F18 |
| Breakup by Stay Duration | Hide when the rule says; add an empty state | F23, F24 |
| Dues by Property | Hide on a single-property account | F23 |
| Forward window | Carry the D9 rules onto the Custom path: no chip, dash for event counts, forecasts extend | F17, D9 |
| Bookings | Confirm whether `is_confirmed = 0` means pending or cancelled; two places in the code disagree | F4 |
| Added By | Say what creator codes 5 and 6 are; they are shown as RentOk today | F26 |
| Collection Custom forward path | The D9 rules: no chip past today, dash the money-arrived numbers, delete the dead `coming_up` branches; confirm the Paid Date picker stops at today | F27, F17 |
| Collection Overview | Advance and Current FY keep their chips in Due Date view | F31 |
| Collection Overview | Cap the due window at today in `computeWindows`, both views one rule | F32, D11 |
| Collection Overview | Collected & Adjusted comparison bounded to the same elapsed moment | F33 |
| Collection Overview | Adjustments-only line and an empty state for the tile row | F34 |
| Collection Breakup | Case-insensitive, trimmed category grouping, as Dues does | F35 |
| Collection Breakup | Adjustment modes as rows on the Due Date Mode and Received by tabs | F36 |
| Collection Breakup | The one-line no-tenant note on the Status tab | F37 |
| Collection Breakup | Billed side per row on the Due Date Status tab | F38 |
| Collection empty copy | Ship §15's replacement copy on four blocks | F39 |
| Adjusted Collection | Discount reads the credits records, windowed by paid date | F40 |
| Payment Settlement | Pick one account record per payout before summing; the rows overstate settled money by 14% | F87 |
| Payment Settlement | Read the payout's status again, so a failed transfer can be marked inside Unsettled as §11 asks | F88 |
| Collection View all | Adjusted rows include discount | F45 |
| Collection healthy states | Emit the three §15 healthy messages | F46 |

**Waits on Vivek, Expense**

| Item | One-line action | Entry |
|---|---|---|
| Every chip, all five tabs | End the "same point last month" window at the end of the previous month. The same two lines sit in duesService, collectionService, expenseService, tenantService and occupancyService | F47 |
| Four empty states | Send the in-window zero copy where the property has spent before; keep the setup line for a property that never has, in `expenseService.ts` | F50 |
| Overview row | Give it an empty state, the screen's own in-window zero | F51 |
| Expense healthy state | Emit "Nothing owed to staff right now." on the tile and on the View all row that repeats it | F51 |
| View all, quarantine row | Take the absolute value before the two helpers that floor it at zero | F55 |
| View all, no-bill row | Print the amount once, not as value and sub-label both | F56 |
| View all sheet | Carry the window's own name, not today's date | F57 |
| Every window on the tab | Say which clock `paid_date` is stored on; convert if it is UTC | F58 |
| Expense Breakdown, Category | Group the catch-all on the lower-cased trimmed name, display the first spelling | F59 |
| Top Payers, Paid by | Label FlexiPe spending "FlexiPe (Owner)"; never show the raw `(Owner)` | F62, D19 |
| Expense Trend | End the range at today, like the other six blocks | F63 |
| Expenses by Property | Send the empty state that is already written, when the window's total is zero | F64 |

**Waits on Vivek, everywhere**

| Item | One-line action | Entry |
|---|---|---|
| Every block, all five tabs | An unresolved property must not fall back to the block's scaffold; fix once in `toBlock`, not 51 times in the services | F52, F84 |

**Waits on Vivek, Occupancy.** Every row is in `occupancyService.ts` unless another file is named.

| Item | One-line action | Entry |
|---|---|---|
| Every tile | Build the chips at all: `chip()` at `:54` is written and never called, and `:204` to `:208` hardcode null. Fix `:72`'s comparison window in the same change | F65 |
| Under eviction tile | Take in pending notices too, with the two parts always named on the second line | F66, D21 |
| Under eviction tile | Unit view counts units holding someone under eviction, not people, at `:208` | F66, D22 |
| Booked, three surfaces | Confirmed bookings only, in `helpers.ts` `fetchAllWidgetCounts` (`:1929`, `:1945`, `:2023` to `:2025`), in the rooms list filters, and here | F67, D20 |
| Overview, a period | Divide by the capacity as it stood at `:196`, reusing `trendMonths` at `:440`; exclude future-scheduled stay rows at `:158` | F68 |
| Occupancy Status | Two chips, not four, at `:256` to `:261`; hide them at zero; add the Under eviction second line | F69 |
| Occupancy Status | Make the legend add up to the header, both directions: over-occupancy in Bed view, zero-capacity rooms holding somebody in Unit view | F70 |
| Occupancy Status | Count booked at bed grain at `:243`, or say the count is by room | F71 |
| Rentable units | Exclude rooms that declare no capacity at `:139`, and give them somewhere to go | F72 |
| Vacant Room Status | Red marks the largest of the four aging bars only, at `:348` | F73 |
| Four cards | Send the healthy copy §16 already holds; stop shipping the sentence §16 forbade, at `:22`; add the not-set-up and no-beds states | F74 |
| Upcoming Vacancy | Drop the red bar at `:390` | F76 |
| Agreements ending soon | Name the eleven-month fallback in `occupancyHints.ts` with its count, and give that count a drill to the tenants missing a length. Same treatment on the Tenant tab's Agreement Expiry | F77, D24 |
| Losing money | Fall back to the group average at `:547` and `:554`, so an empty room is not priced at zero | F78 |
| Losing money | Compute the bar and the share from rupees at `:571`, not from the formatted string; sort by revenue loss at `:566`; build the coverage threshold | F79 |
| Three silent blocks | "As of today" on Status, Property Wise and the View all sheet | F80 |
| Vacant Room Status | Hide Never rented and Unknown at zero; the four aging bands keep showing at zero. The Unknown half is not new: DA-08 §8 already says that bar hides at zero and the build renders it always, a gap the Occupancy pass missed | D30 |
| View all sheet | The seven missing rows at `:621` to `:639`, the missing third section, the never-rented number `getVacantRooms` already computes | F81 |
| Occupancy Trend | Start the chart where the property's data starts, at `:497` | F82 |
| Custom path | D9's rules in `occupancyService.ts` `computeWindow`; delete the dead `coming_up` branches | F83 |

**Waits on Vivek, Tenant.** Every row is in `tenantService.ts` unless another file is named.

| Item | One-line action | Entry |
|---|---|---|
| Agreement end date, two services | Guard the `agreement_period` cast: digits only, else null | F90 |
| Agreement end date, Tenant only | Null means the No term recorded group, never eleven months. Inventory keeps its eleven-month step | D26 |
| Any failed block, `service.ts` | Catch it, so one bad settings value cannot kill the whole page | F90 |
| Custom path | D9's rules; delete the dead `coming_up` branches at `:149`, `:260`, `:265`, `:311`, `:470`, `:1003` | F91, D9 |
| Ten silent blocks | "As of today" on the live blocks, "From today onwards" on Upcoming Eviction and Agreement Expiry | F92 |
| Approved Bookings, Journey funnel | Say whether an auto-accepted booking leaves any trace; until then, count real approval rows only, as `movedCounts` already does | F93 |
| Overview info sheet, `tenantHints.ts` | Add Eviction Approved and Past their date entries; fix the Active Bookings definition, it counts awaiting too | F94 |
| Move-outs | Exclude cancelled bookings, the delete path stamps them with a leaving date | F95 |
| Journey, Under Notice bar | Rename it Eviction pending; the number it already computes is right, and D29 gives Approved eviction its own row beside it | F96, D29 |
| Journey, Tenants tab, group 1 | Heading "Who is staying, who is leaving": Active tenants · No notices · Eviction pending · Approved eviction. The last three add up to Active on any property where nobody is past their date; where somebody is, they fall short by that count, which S9 proposes closing | D29 |
| Journey, Tenants tab, group 2 | Heading "Whose paperwork needs a decision": Renewals in 30 days · Agreements expired. These overlap group 1, and exclude each other. Parallel bars only. Agreements expired reuses the `renewal_overdue` expression `getRenewalRetention` already runs | D29 |
| Journey strip | Churn as a share with the count beside it; left early reads `lockin_period`, not the agreement end date, with a coverage line | F97 |
| Upcoming Eviction | Hide the Overdue bar at zero; the four day bands keep showing at zero | D30 |
| Agreement Expiry | Relabel to Already expired · 0–30 days · 31–60 days · 61–90 days · Valid (90+ days) | D31 |
| Agreement Expiry | Hide Already expired and No term recorded at zero; the four day bands keep showing | D30 |
| Stay Type, Renting Type | Stay type shows always, zero side included. Renting type shows only when either side is above zero, which replaces D27's untestable "until the field is captured"; today that hides the block everywhere, and the interim stay-split donut is removed rather than renamed | D32, D27, F102 |
| Four colours | Police in-time plain (the default colour, not red), ID not-verified plain, the 30-day band plain, Not signed red; look at the swapped profile pair while there | F98 |
| Upcoming Eviction | Fall back to the tenant-level date the tiles read, and say which record is authoritative for the list too | F99 |
| Agreement Expiry | No-term tenants leave the five bands and become the stated "No term recorded" group with its count and its drill to the list | F100, D26 |
| Renewal figures | Renewal overdue, Renewal Due and Journey's Renewals in 30 days all count real terms only, moving with the bands above | D26 |
| Renewal & Retention | Stayed after notice as a share of those whose date fell in the window and passed, count beside it | F101 |
| The rename, five strings | "Under eviction" replaces "Under notice" in the View all sheet's row, the rent at risk note, both Journey empty messages, and the Journey hint in `tenantHints.ts`. The Journey bar label is deliberately not in this list: D29 renames it Eviction pending instead | D28, D29, S3 |
| Empty and healthy states | The five §21 healthy messages; wire the six unsent empty states; the whole-screen not-set-up state; fix the Bookings-tab copy-paste | F103 |
| Three breakdowns | An Other bar for unmatched food values; "girls" into the female list; no-joining-date tenants stated, not dropped | F104 |

**Waits on the owner**

| Item | The question | Entry |
|---|---|---|
| Breakup by Stay Duration | Hide when there are no short-term tenants, or no short-term dues | F23 |
| Forward window | What an event count shows on a window that straddles today | D9 |
| Approved Bookings | What the tile shows if auto-accepted bookings leave no trace, once Vivek answers | F93 |
| Added By | Name creator codes 5 and 6, once Vivek says what they are | F26 |
| Agreement length | Whether the product starts asking for it at move-in, so the gap stops reopening | S7 in the register |
| Paying staff back | Whether a way to record it ships before the Still owed to staff tile does | S6 in the register |
| The word "Others" | Whether the rollup row is renamed across Dues, Collection and Expense together | S5 in the register |

**Waits on the other sheets**

| Sheet | Change | When |
|---|---|---|
| DA-09 Tenants | §4: drop Coming up, rewrite the forward column per D9, from that tab's code | Done, 2026-08-25 |
| DA-04 Expense | §4 explains its own no-forward rule by pointing at Coming up, an option D9 deleted. The rule is right; the reason needs rewriting without the dead name | Done, 2026-08-24 |
| DA-08 Inventory | §4's forward column, the under-eviction naming after D6, and the Occupancy booking question | Done, 2026-08-24 |

---

## Findings

### F1. Current FY Dues: the tile and its own chip measure different windows

**Verdict:** Build gap · **Status:** Ruled · **Owed by:** Vivek · **Note:** D2 settles it; chip also needs F5

The tile shows 1 April through 31 March, so it counts bills dated in the future. The chip beside it is
worked out on 1 April through today, against the same point last financial year. The percentage does
not describe the number it sits under.

`duesService.overviewAggregate()` computes both `current_fy` (1 April to 31 March) and `fy_ytd`
(1 April to today). `getOverview()` displays the first and chips on the second.

**Fix:** display `fy_ytd`. The figure is already calculated. The same choice is written separately in
`getViewAll`, see F25. Reasoning in D2.

**Audit note.** This entry first said the chip "becomes correct with a one word change". Wrong: the
figure the chip compares against is also worked out the wrong way, see F5. Both fixes are needed.

### F2. Projected Due: the rent half leaves out confirmed bookings, the package half filters nobody

**Verdict:** Build gap · **Status:** Ruled · **Owed by:** Vivek · **Note:** D1 settles who counts

`projectedDueTotal` adds two halves, and each is wrong in its own way:

| Half | Who it counts today | What D1 says it should count |
|---|---|---|
| Rent (`t.status = 1`) | Living tenants only | Living tenants and confirmed bookings |
| Packages (no tenant condition at all) | Every pending scheduler row on the property, including bookings of any state and departed tenants with live rows | Living tenants and confirmed bookings |

So the tile undercounts rent and overcounts packages, and neither half matches the other.

**Fix:** the same tenant test on both halves: status 1, or a confirmed booking. The code that decides
"confirmed" already exists at `tenantService.ts:352`; reuse it rather than write a second one.
`getUpcomingDues` holds a near copy of the rent half and needs the same fix, see D8.

**Audit note.** This entry first described the rent half only. The fact-check found the package half.

### F3. Projected Due can pick the wrong rent day

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek

When a tenant has no `rent_addition_date`, the projection falls back to their joining day. The job
that actually raises the rent, `chargeRentTenants`, falls back differently by property setup:

| Property setup | What the job falls back to | What the projection falls back to | Agree |
|---|---|---|---|
| Joining date properties | Day of joining | Day of joining | Yes |
| First of month properties | The 1st | Day of joining | No |
| Configured date properties | The property's `rent_addition_cycle` | Day of joining | No |

On two of the three property types, rent lands on the wrong side of "still to raise before month end"
for any tenant missing that field.

**Fix:** fall back the way the job does. F2 and F3 are the same kind of mistake: the forecast counts a
different set of tenants, on a different day, from the job that will create those bills. Any forecast
should be checked against the job that does the work. Applies to `getUpcomingDues` too.

### F4. Status 2 does not mean confirmed

**Verdict:** Withdrawn for Dues · **Status:** Ruled · **Owed by:** Owner for Occupancy, Vivek for one check · **Note:** D5 withdrew the Dues half

This review first treated tenant status 2 as "confirmed booking". The owner corrected that:
confirmation is a separate step, recorded in `tenant_booking_confirmation`:

| `is_confirmed` | Means, per the analytics code and the table's default |
|---|---|
| 1 | Approved by a named team member, with a timestamp |
| 0 or no row | Waiting for approval |
| -1 | Cancelled or rejected, with who, when, and the reason |

A property setting, `auto_accept_booking`, removes the step entirely; the owner ruled auto-accepted
and confirmed are the same thing (D1).

**What D5 settled.** A raised bill is owed whatever the booking's approval state, so every Dues block
that counts bills is correct with plain status 1 or 2. Only the forecast needs the confirmed test, and
that is F2. The Dues half of this finding is withdrawn.

**Still open, for the owner.** Occupancy is not a money question. Its forward projection uses plain
status 2, and its Booked layer reads a homescreen widget not traced here. Whether an unapproved
booking's bed reads as filled, blocked or vacant is ruled at DA-08.

**Still open, for Vivek.** `tenant.ts:3657` treats `is_confirmed = 0` as "already cancelled";
`tenantService.ts:16` treats 0 as pending. Both cannot be right.

**Audit note.** This entry first claimed a cancelled booking "stays at status 2 forever". Wrong:
`rejectBooking` calls `deleteActiveTenant`, which moves the tenant to status 0 (moved out). On
production only 2 tenants sit at status 2 with a cancelled confirmation, owing ₹2,350 between them.

### F5. Both chips compare against the wrong figure, and always read worse than reality

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek · **Note:** also the View all screen, F25

Sheet §4 sets the rule: what was owed on a past date is every bill that had not been paid yet on that
date, so a bill counts if it is still unpaid today or was paid after that date. It names the trap in
the same breath: asking what is still unpaid from last month gives a figure roughly a quarter too
small, because a month of collections has happened since, so a property collecting exactly as well as
before shows worse every month.

The code does the thing the sheet warns against. `this_month_prev` and `fy_ytd_prev` are worked out
on the base pool, which only ever holds bills still unpaid today, so the comparison counts only what
is still unpaid from the past window. Everything collected since has left the comparison.

**Fix:** the comparison figure needs bills due in the comparison window that were unpaid as at the
comparison date: unpaid today, or paid after that date. Part payments need no special handling; a
part paid bill splits into a paid bill and an unpaid one with the same due date.

### F6. The chips are coloured the wrong way round, and Dues is the only tab that gets it wrong

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek · **Note:** also the View all screen

Sheet §4: "Up is bad here: a rising chip shows red, a falling one green." The Dues chip code at
`duesService.ts:136` colours every chip green for up and red for down. On a screen about money owed to
you, that is backwards.

Every other tab decides per tile which direction is good, each in its own piece of code:

| Service | Chip code | Which way is good |
|---|---|---|
| `duesService.ts:131` | `chip(cur, prev, note)` | Not decided. Always green for up |
| `collectionService.ts:91` | `chip(cur, prev, kind: good, bad, neutral, note)` | Per tile |
| `expenseService.ts:83` and `:91` | `chipMoney`, red for up; `chipCount`, grey | Per piece of code, correct for expense |
| `occupancyService.ts:54` | `chip(cur, prev, kind, note)` | Per tile |
| `tenantService.ts:49` | `chip(cur, prev, goodDir: up or down, note)` | Per tile |

**Fix:** port any sibling's. Six pieces of code for one idea is its own small finding: the next tab
that needs a chip will write a seventh.

Correct as built: a chip is suppressed when there is nothing to compare against, which is what §4
asks for a property in its first month.

**Audit note.** This entry first said Collection and Expense share the fault. They do not.

### F7. Dues (Live) date captions: the sheet's worked example contradicted its own table

**Verdict:** Accepted change · **Status:** Closed · **Owed by:** Nobody · **Note:** D3; sheet §7 corrected

The numbers are correct. Only the two date captions differed, and the code was right both times.

| Caption | Sheet §7 example said | Code does |
|---|---|---|
| Due This Week | "01 Aug–07 Aug", starting today | Tomorrow through today plus 7 |
| Due Later | "After 08 Aug", the first day of the range | "After 30 Aug", the last day of the week |

Due Today is its own slice directly above, so a week that includes today would double count. And
"After 08 Aug" read literally means from the 9th.

### F8. Bills Summary: Received is not limited to money that arrived inside the window

**Verdict:** Accepted change · **Status:** Closed · **Owed by:** Nobody · **Note:** D4; sheet §8 and §3 corrected

Sheet §8 said "money collected against those same dues, inside the window". The code windows only the
bill's due date; the paid amount carries no date condition, so Received is every rupee ever collected
against bills that came due in the window. The code is right, reasoning in D4.

### F9. The Current FY filter option runs to 31 March, on Dues and Collection only

**Verdict:** Build gap · **Status:** Ruled · **Owed by:** Vivek · **Note:** D2 settles the window; this is a second place to apply it

The `current_fy` case inside `resolveWindow()` (`duesService.ts:66`) resolves the filter option to
1 April through 31 March, and `collectionService.ts:149` does the same. Expense, Occupancy and Tenant
all end at today.

This is F1 one level up. The same wrong window sits inside the dropdown that Bills Summary, Dues
Breakdown, Breakup by Stay Duration, Dues by Property and their Collection counterparts read.

**Fix:** end at today. The pattern: a definition was fixed where it was noticed and left wrong in the
shared place it came from.

**Collection audit note, 2026-08-24.** Still present at `collectionService.ts:149`. Paid Date view
hides it, because money windows cap at today anyway; the bite is Due Date view, where Billed runs to
next 31 March. The F32 ruling inherits it.

### F10. Refunded and written off dues vanish with nothing said about them

**Verdict:** Specification gap · **Status:** Closed · **Owed by:** Nobody · **Note:** sheet §3 now names both states

Both states are absent from every Dues number, which is right. The sheet never said so, so a reader
adding up a total by hand had no way to know those states exist.

### F11. Dues Breakdown's three tabs do not add to the same total, and the sheet said they did

**Verdict:** Specification gap · **Status:** Closed · **Owed by:** Nobody · **Note:** sheet §9 corrected

§9 opened with "three views over the same dues". Category and Added By run on the base pool, which
excludes moved-out tenants. Tenant Status deliberately widens to include them, their one home on the
screen. The three tabs differ by exactly the old-tenant money, by design.

### F12. "Under notice" meant something different on Dues and on Tenants

**Verdict:** Withdrawn · **Status:** Closed · **Owed by:** Nobody · **Note:** widened into the census in F14, ruled in D6

First raised as a two-screen collision. The owner asked for every definition in the suite instead.

### F13. "Notice Raised" was proposed as a rename, and the phrase is already taken

**Verdict:** Withdrawn · **Status:** Closed · **Owed by:** Nobody · **Note:** D6 adopted neither proposed name

The Tenant Overview already carries a tile called Notices Raised, a period count. A live headcount one
letter away would mean something different. D6 used the product's existing words instead.

### F14. "Under notice" has five live meanings in the code and two in our own sheets

**Verdict:** Build gap · **Status:** Ruled · **Owed by:** Vivek · **Note:** D6; Inventory's count still open

**In the code:**

| # | Where | What it asks for | Counts an unapproved notice | Counts a date already passed |
|---|---|---|---|---|
| 1 | Dues Breakdown bar | active, leaving date exists | No | Yes |
| 2 | Collection Breakup bar | identical to Dues | No | Yes |
| 3 | Occupancy Upcoming Vacancy | active, date after today, inside the card's look-ahead | No | No |
| 4 | Occupancy confirmed-leaving count | active, date today or later | No | No |
| 5 | Tenants `UNDER_NOTICE` | date today or later, or no date but a notice awaiting approval | Yes | No, it is a separate group called Past their date |
| 6 | Occupancy Under notice tile | homescreen widget 1402, `UNDER_EVICTION` | Not traced | Not traced |

They disagree on three things: whether a pending notice counts, whether a passed date counts, whether
a look-ahead applies.

**In our sheets:** DA-01, DA-02 and DA-08 all say confirmed leaving date only, and DA-08 adds "no
second variant survives". DA-09 says "has said they are leaving", which takes in the unapproved
notice, exactly as its code does. Two definitions, and one sheet claims there is only one.

**A third contradiction.** DA-02 says "The Dues screen keeps these tenants inside its Active bar; on
Collection the split is deliberate." False: Dues has its own bar, in its sheet and its code.

**Measured on production**, living tenants (status 1) on live properties, by notice state. Tenant
counts only; a dues column was tried and dropped because duplicate tenant rows inflated it.

| State | Tenants |
|---|---|
| No notice | 346,358 |
| Approved, date still ahead | 4,688 |
| Approved, date already passed | 322 |
| Raised, never approved | 1,183 |

**Fix:** the bars in D6. The test for a pending notice is already written at `tenantService.ts:200`.

### F15. The first Overdue tab is called "By Amount", but both tabs are amounts

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek

One tab splits by how late the bill is, the other by bill type. "By Amount" names the measure rather
than the split. Sheet §10 calls it By ageing, and the block's own axis label already reads "Overdue
Timeline".

Minor, in both documents: day 90 falls in the 22-90 bucket, so the bar labelled "90+ Days" begins at
91.

### F16. A dropdown with one option, on a block the sheet says has no dropdown

**Verdict:** Build gap · **Status:** Ruled · **Owed by:** Vivek · **Note:** D7

The build gives this block a dropdown with a single option, All Time (`registry.ts:115`). Sheet §10:
"no dropdown of its own". A control with one option invites a tap and does nothing, the option it
shows contradicts the card's own face, as of today, and §4 requires a card's own dropdown to carry the
same options as the top filter.

### F17. "Coming up" was dropped from the filter list and its behaviours were left behind

**Verdict:** Build gap · **Status:** Ruled · **Owed by:** Vivek · **Note:** D9. The largest finding in this review

Every sheet's §4 carries a Coming up column, the forward setting. The option is missing from the two
lists the app builds its filter from (`FIN_DURATION`, `OCCUPANCY_DURATION`), so the app can never send
it. What that left behind:

| Service | `coming_up` in the code | Effect |
|---|---|---|
| Dues | Never had one | The sheet's Coming up column was never built |
| Collection | Yes, four places | Dead code |
| Occupancy | Yes | Dead code |
| Tenant | Yes, and it gates the projected tiles, two forward tile variants, the "what moved" window and the dash for numbers that cannot be computed forward | All unreachable. Numbers that should dash now render real looking values |
| Expense | Never had one, and its sheet never asked | Correct |

The Custom option now lets a window end in the future (`allow_future_date: true` on four tabs), but
without the safeguards Coming up was meant to carry. Expense alone keeps Custom at today, matching its
sheet.

**Fix:** the rules in D9, on the Custom path. DA-01 §4 is already rewritten; the other sheets are owed.

### F18. Two Dues blocks ignore the top filter and say nothing about it

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek · **Note:** against D9

Every Dues block, checked for whether it reads the top filter and whether it says so on its face:

| Block | Reads the top filter | Says so on its face |
|---|---|---|
| Overview | No | "Overview (Live)" |
| Dues (Live) | No | "Dues (Live)" |
| Bills Summary | Yes | Not needed |
| Dues Breakdown | Yes | Its own dropdown |
| Overdue Breakup | No | Nothing, once D7 removes the dead dropdown |
| Upcoming Dues | No | Subtitle "Forecast" |
| Deposit Dues | No | Nothing |
| Breakup by Stay Duration | Yes | Not needed |
| Dues by Property | Yes | Its own dropdown |

**Fix:** `subtitle: 'As of today'` on the two silent blocks. Upcoming Dues already does it with
`subtitle: 'Forecast'`.

### F19. Deposit Dues matches its categories by exact text, so 161 properties get a short figure

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek

Both queries filter `due_type IN ('Security Deposit', 'Caution Money')`, exact, case sensitive,
untrimmed. Sheet §3: "names that differ only by capitals group as one". The Category view on the same
screen honours that; this card does not.

Measured on production, active invoices on live properties:

| Written as | Invoices |
|---|---|
| Security Deposit | 798,793 |
| Caution Money | 935 |
| Security deposit | 391 |
| Caution Money, trailing space | 51 |
| caution money, trailing space | 1 |
| Security deposit, trailing space | 1 |

444 invoices across 161 live properties are invisible to this card. None of those properties reads
zero, since each also has correctly spelled deposits, but every one shows a figure that is short.

**Fix:** `LOWER(TRIM(due_type))`, as the Category view does.

### F20. The card shows a total the sheet never defined

**Verdict:** Specification gap · **Status:** Closed · **Owed by:** Nobody · **Note:** sheet §12 now defines Total Deposits as Received plus Due

Adding the two is only safe once F21 is answered.

### F21. Received and Due are added up from two different money columns on the bill

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek · **Note:** blocks D10's new line

Received sums `net_amount`. Due sums `amount`. The total adds them.

Measured on production: `net_amount` differs from `amount` on 430,350 of 800,172 deposit invoices,
54%. It is not discount; the discount column is zero on every one of them. Every other block on the
screen sums `amount`; only this one reaches for `net_amount`.

**Fix:** say what each column means, then use one of them for both rows.

### F22. Deposits held for tenants who have moved out were missing from the card

**Verdict:** Specification gap · **Status:** Ruled · **Owed by:** Vivek · **Note:** D10; sheet §12 and §3 corrected; blocked on F21

The Received row keeps only invoices whose payer is at status 1 or 2. Sheet §12 said Received is
"deposit collected and still held", which describes the money; §3 said old tenants are excluded from
every number but one. A deposit not returned is still held, so the two sections disagreed.

Measured on production, paid deposit invoices on live properties, netting off refunds and adjustment
payments:

| Payer | Paid deposit invoices | Still held after refunds and adjustments | With no settlement of any kind recorded |
|---|---|---|---|
| Living or booked, counted today | 124,530 | 124,142 | 123,989 |
| Moved out, not counted | 148,953 | 138,925 | 133,886 |
| Other statuses | 5,802 | 5,635 | 5,577 |

**Audit note.** The figure first reported here, 148,946, was the "paid deposit invoices" column from
an earlier run of the same query, half an hour before, presented as if it were the "still held"
column. Production moves, so the two runs differ by seven rows; the error was the column, not the
run. The number the card should carry is 138,925.

**What the numbers can and cannot say.** In the system of record, 138,925 deposits from people who
have left are still held. The product's own exit flow, `evictionService.ts:520`, computes "deposit
available" from the same two records, refunds and adjustment payments, so there is no third place in
the system where a settlement could hide. Whether cash went back offline and was never recorded is
not knowable from data. Ruling and caution in D10.

### F23. Dues is the only tab that never hides a card, and it has two cards that must

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek, and the owner for one rule

Sheet §19: "Breakup by Stay Duration hides when the property has no short-term tenants. Dues by
Property renders only for multi-property accounts. Hidden is hidden: no ghost card, no explainer."
Neither is built. A property with no short-term tenants gets one bar equal to the total; a
single-property account gets a one-row ranking of itself.

The code already has a standard way to hide a card, a `hidden` return, and the other four services
use it: `collectionService.ts:587` and `:714`, `expenseService.ts:545`, `occupancyService.ts:582`,
`tenantService.ts:1117`. Dues never does.

**For the owner first:** §13 says the card shows when the property has a short-term tenant. The
natural code test is whether short-term dues are above zero. A property with short-term tenants who
owe nothing hides under one test and shows a single restating bar under the other.

### F24. Breakup by Stay Duration has no empty state at all

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek

When both figures are zero the block returns an empty list of segments with no empty state, no
healthy state and no hidden flag. Every other block on the screen sends its zero somewhere; Dues by
Property, directly beneath, does it correctly.

### F25. The View all screen repeats two Overview defects, and one fix has to be applied twice

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek · **Note:** implementation note on F1 and F5

`getViewAll` restates the six tiles from the same aggregate as the card, so it inherits F5; one fix
there corrects both. But the choice of `current_fy` over `fy_ytd` is written separately in
`getOverview` and `getViewAll`, so F1's fix has to land in both or card and screen will disagree.

It also carries `// TODO: hide this section when the property has no short-stay tenants.` Third site
for the F23 rule, and the only one where it was read and deferred rather than missed.

### F26. Two creator codes have no name and are silently shown as RentOk

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek, and the owner for the naming

DA-01 §20, open item 2, asked engineering to settle this before the view shipped: "Dues worth about
₹2.8 crore carry a creator category that has no display name. Added By needs a name for it before
that view ships."

It shipped with `addedByLabel`'s `default: return 'RentOk'`, so every unnamed code is folded into
RentOk's own bar rather than named.

**Measured on production**, unpaid dues on live properties, counted with `EXISTS` rather than a join
so duplicate tenant rows cannot inflate it, and capped at ₹1 crore a bill to keep outlier rows out:

| Code | Shown as | Unpaid bills | Unpaid amount | Properties |
|---|---|---|---|---|
| 1 | RentOk | 3,391,377 | ₹314.4 Cr | 21,864 |
| 0 | Owner | 85,756 | ₹89.7 Cr | 13,878 |
| 2 | Partner | 44,601 | ₹52.2 Cr | 3,125 |
| **6** | **RentOk, unnamed** | **3,725** | **₹2.78 Cr** | **264** |
| **5** | **RentOk, unnamed** | **2,665** | **₹2.07 Cr** | **271** |
| 4 | Tenant | 9 | ₹55,060 | 8 |

**There are two unnamed codes, not one.** The sheet found code 6 and measured it at "about ₹2.8
crore", which matches exactly. Code 5 is a second one, ₹2.07 crore across 271 properties, and neither
document mentions it. Together they put ₹4.85 crore of bills into RentOk's bar that RentOk did not
raise, on roughly 500 properties.

**Two things are needed.** Vivek should say what codes 5 and 6 are, and the owner should name them.
Folding an unknown into an existing named bucket is the one resolution that cannot be checked later,
because the bar stops being able to tell you it is wrong.

**Also correct as built, checked in the same pass:** Added By takes the top four and folds the rest
into Others, matching §20's open item 5. `foldTop` suppresses the Others row when there is nothing to
fold, which is §9's "fewer than four real contributors, no empty Others row". Each bar carries its
bill count. RentOk gets a bar only when it has an amount, which holds by construction since the base
pool requires at least ₹1.

**How this was found.** By reading the guide's appendix, which this review had not opened. See the
audit note under How this review works.

Findings from F27 are the Collection tab, reviewed 2026-08-24 in sheet order: the toggle and §4
first, then the seven blocks and the View all sheet.

### F27. The forward window rules never landed on Collection's Custom path

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek · **Note:** D9 rules it; F17 logged the suite-wide cause

Due Date view lets a Custom window end after today (`masterConfig.ts:126`), which D9 makes the
forward view. None of D9's safeguards exist here:

| D9 says | The code does |
|---|---|
| No chip on a window ending after today | `computeWindows` builds a comparison window for every Custom range (`collectionService.ts:154`), so the three Due Date tiles chip against "vs preceding period" |
| A number counting money arriving shows a dash with a reason | On a fully future window the Advance tile's paid window ends before it starts, so it renders ₹0, and Settlement Pending renders ₹0. A window that straddles today renders partial figures instead, the case D9 left open |
| Records keep counting | Correct as built: Billed and Collected & Adjusted count the forward window's bills |

Also for Vivek to confirm: the Custom option carries `allow_future_date: true` at the tab level
while the Paid Date view carries `false`; the app must let the view win, or Paid Date reaches a
future window the service caps silently.

**Fix:** the D9 rules on the Custom path: suppress chips when the window ends after today, dash the
money-arrived numbers with the one-line reason, and delete the four dead `coming_up` branches while
there (F17). The check that decides whether a tile sits out reads `filter_key === 'coming_up'`
(`collectionService.ts:343`), an option the app can no longer send.

### F28. Total Collection leaves advance money out, and the sheet says the middle four tiles add up to it

**Verdict:** Accepted change · **Status:** Closed · **Owed by:** Nobody · **Note:** D12; sheet §5 corrected

Sheet §5: Total Collection is "all money received in the window", and the four tiles beside it,
Advance included, "add up to Total Collection exactly". The code counts only money applied to bills:
a payment against no bill contributes nothing to Total Collection and lives in its own Advance tile
outside the sum (`COLL_AMT`, `collectionService.ts:247`). Deliberate, commented, and checked against
production during the build: counting it made the tile disagree with the homescreen's collection
figure, which §17.2 forbids.

So the sheet asks for two things at once: §5 wants advance inside the total, §17.2 wants agreement
with a homescreen that keeps it out. The build chose §17.2. The same choice repeats on the Current
FY tile. Advance is small: 0.008% of July's money, per the sheet's own appendix.

Ruled as built: D12 has the reasoning. Sheet §5 corrected.

### F29. Collection is counted at the bill amount, not at money net of processing charges

**Verdict:** Accepted change · **Status:** Closed · **Owed by:** Nobody · **Note:** D13; sheet §3 and §17.7 corrected

Sheet §3, the base rule: "a ₹10,000 online payment carrying a ₹200 charge counts as ₹9,800", and
§17.7 repeats it as a test. The code values every payment against a bill at the bill's own amount
(`inv.amount`), gross of gateway charges, matching the homescreen and the collections list. Only the
Advance tile and the whole Payment Settlement card are net of charges.

Ruled as built: D13 has the reasoning, including the one deliberate exception, the Advance tile
counting net. Sheet §3 and §17.7 corrected.

### F30. Settlement Pending carries no chip, and the sheet asks for one

**Verdict:** Accepted change · **Status:** Closed · **Owed by:** Nobody · **Note:** D14; sheet §4 chip table corrected

Sheet §4's chip table: Settlement Pending rising is bad, red up, green down. The build sends no chip
on the tile in either view (`collectionService.ts:319`).

Ruled right as built: D14 has the reasoning, and §4's chip table now carries it on the row.

### F31. Advance and Current FY lose their chips when the view flips

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek

In Paid Date view both tiles carry a good-kind chip (`collectionService.ts:317-318`). In Due Date
view the same two tiles, same numbers, same windows, send no chip (`collectionService.ts:352-353`). §4 says
the never-switching tiles do not change between views; their chips should not either.

**Fix:** compute the same two chips in `overviewDueDate`.

### F32. In Due Date view the window runs to the period's end, and the chip compares elapsed days

**Verdict:** Build gap · **Status:** Ruled · **Owed by:** Vivek · **Note:** D11 settles it; F33 is separate and stands

The build read This Month in Due Date view as the full month, 1st to 31st: the due window kept the
whole period (`collectionService.ts:183`) while the paid window caps at today, and Current FY in
this view ran to next 31 March (F9). So mid-month, Billed counted the whole month's bills, Still
Unpaid included bills not due yet, and the chip compared that full period against the previous
month's first N days while saying "vs same point last month": the same mistake as F1, a chip not
describing the number it sits under. The sheet never said where this window ends; Vivek's guide
called the full period deliberate.

D11 ruled it: the window ends at today in both views, one cap in `computeWindows`. Every chip on the
view then compares like with like automatically. The design argument, and where already-raised
future bills go, live in D11.

### F33. The Collected & Adjusted comparison is measured as it stands today

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek · **Note:** the same mistake as F5; D11 sets the windows it compares

D4 defines collected-against-bills as payments whenever they arrived, which is right for the tile.
The chip's comparison figure is computed the same way (`collectionService.ts:340`), so last month's
figure has had a whole extra month of collecting. A property collecting exactly as well as before
reads worse every month: the defect F5 logged on Dues, in the opposite direction.

**Fix:** the comparison counts only payments received by the equivalent moment of the previous
period. Applies to the Collected & Adjusted and Still Unpaid chips. Billed does not accumulate
payments and only needs D11's cap.

### F34. The Overview's special states are half built

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek

The negative-total note is built and worded exactly as §5 asks. Not built: the adjustments-only
state, ₹0 with "No money received. ₹X of bills were cleared from deposits and advances.", and any
empty state for the tile row (§15: a window with no activity is ordinary, one live property in six).
Every other Collection block sends its zeros somewhere; the screen's headline block does not.

### F35. Category rows split by spelling, and Dues already solved it

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek · **Note:** found by the F19 sweep

`breakupCategory` groups by raw `inv.due_type` (`collectionService.ts:469`, `:481`); Dues groups the
same field with `LOWER(TRIM(...))` (`duesService.ts:220`). Measured on production, July 2026, live
non-test properties, successful active payments joined to their paid bills: at least 20 category
names carry more than one spelling. Security Deposit alone is ₹32.5 crore across two spellings in
one month, 34,677 payment rows; Electricity Bill ₹2.6 crore across two.

A split spelling splits its row, misranks the top 3, and pads Others with fragments of real
categories. The same code path feeds the Due Date billed side and the View all category rows, so one
fix lands everywhere; note the billed lookup joins on the label and must normalise both sides.

**Fix:** group by `LOWER(TRIM(inv.due_type))` and display `MIN(inv.due_type)`, exactly as
`duesService.ts:217`.

### F36. In Due Date view the Mode and Received by tabs leave the adjustment money out

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek

Sheet §6, Due Date view: all four tabs split Collected & Adjusted "and all four still add up to it".
`foldModes` and `foldReceivers` skip the three adjustment modes unconditionally
(`collectionService.ts:529`, `:554`). Harmless in Paid Date view, where an adjustment is worth ₹0.
In Due Date view an adjustment carries the bill's amount, so both tabs run short by every deposit,
advance and caution clearance, and stop adding up to the card's total.

It also empties the Mode tab of its stated purpose: §6 keeps it in this view precisely to show how
much of the billing was cleared with real money against drawn from deposits. As built the tab can
never show that.

**Fix:** in Due Date view only, keep modes 211, 288 and 291 as their own rows, labelled as §8
labels them.

### F37. The Status tab's promised one-line note is missing

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek

§6: payments with no tenant attached stay in the total, and "the Status tab says in one line that
they are not counted there". `breakupStatus` drops them silently. Ten payments in July platform-wide
per the sheet's appendix, so the money is small; the sheet's point was never the money, it was that
the tab's rows visibly sum short of the card's total.

### F38. The Status tab has no billed side in Due Date view

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek

§6, Due Date view: "Category and Status also show what was billed, row by row". The Category tab
does; the Status tab returns collected values only (`collectionService.ts:494`). "Rent ₹8L of ₹10L"
works; its Status twin does not exist.

### F39. Four blocks ship the empty-state copy the sheet replaced

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek

§15 lists the current copy and its replacement for each. The build ships the current, wrong copy
verbatim (`EMPTY`, `collectionService.ts:61`):

| Block | Ships | Wrong because |
|---|---|---|
| Breakup, Mode tab | "cash, UPI, bank transfer & cheque" | Promises modes the card does not show and omits RentOk, the biggest. Also carries an em dash |
| Collection Status | "Dues collected and pending will show up here" | The card has no pending row by design |
| Adjusted Collection | "Advance payments and deposit adjustments" | Names two of four kinds, and reads as the Advance tile |
| Collection by Property | "No properties added yet" | The card only renders on multi-property accounts; the empty case is no collection, not no properties |

§15 already holds the exact replacement copy for all four.

### F40. The Discount figure reads a column that is empty on production

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek · **Note:** the largest finding on this tab

The Adjusted Collection card's Discount figure sums `invoices.discount`
(`collectionService.ts:631`), and Vivek's guide describes the same, so the two documents agree with
each other. Production does not: exactly one invoice on the whole platform has ever carried a value
in that column, worth roughly nothing. The real discounts live in the credits records the payment
flow writes, one `credits` row per discounted payment: July 2026, live non-test properties,
successful active payments, 683 payments carrying ₹89 lakh. The sheet's own appendix measured ₹1.07
crore and 992 payments for the same month through the same flow, on a wider scope: taken 8 August,
without excluding test properties. The two are different queries and are not expected to match.

So the Discount figure will read ₹0 for every property on every window, while §8 calls the number
real and the appendix proves it is. This is the exact false pass the three-source method exists to
catch: sheet, guide and code all in agreement, and the data elsewhere.

Two smaller defects die with the fix: the discount query windows by the bill's due date while the
card's other three rows window by paid date, one card on two clocks, and it reads any active
invoice, paid or not.

**Fix:** sum the discount on the payment, windowed by `p.paid_date` like the other three rows. The
View all sheet's Adjusted rows need the same source once F45 adds discount there.

**Correction, 2026-08-25: read `payments.owner_credits`, not the `credits` table.** This entry
originally named the credits rows as the source. They are a partial copy. Measured on production,
July 2026, live properties, test and deleted excluded, on the collection base: 683 payments carry
both a credits row and an `owner_credits` value and the two agree to the rupee; **299 payments worth
₹16.0 lakh carry `owner_credits` with no credits row at all**, and none carry a credits row without
`owner_credits`. So credits is a subset, and a fix reading it would undercount discount by roughly a
sixth every month and could never agree with the homescreen, which reads `owner_credits`
(`homepage/service.ts:586`). Full numbers and what it means for the three surfaces are in F105.

### F41. Collection by Property follows the toggle, and the sheet says it does not change

**Verdict:** Accepted change · **Status:** Closed · **Owed by:** Nobody · **Note:** D15; sheet §4 and §10 corrected

Sheet §4's card table says the card does not change in Due Date view, and §10 says "the same in both
views". The build switches with the view: money arrived per property in Paid Date, collected and
adjusted against the window's bills per property in Due Date (`collectionService.ts:719`).

Ruled as built: D15 has the reasoning. Sheet §4, §10 and test §17.1 corrected.

### F42. The destination rows cover under 3% of settled money

**Verdict:** Build gap · **Status:** Closed · **Owed by:** Nobody · **Note:** coverage fixed in `4bce93d21`; the rewrite brought its own defect, F87

§11: "the destination comes from the settlement record, whichever settlement system wrote it", and
the rows break down Total Settled and add up to it. The build reads destinations from
`settlement_scheduler` alone (`collectionService.ts:800`).

Measured on production, July 2026, live non-test properties, online modes, successful active
payments: ₹141.55 crore reads settled under the build's own test, and scheduler rows carry ₹3.69
crore of it. ₹137.86 crore of settled money, 97%, would sit under "Where settled money went" with no
row at all.

**Fix:** destinations from the payout and wallet records too, falling back to the contact-support
line only when no system names an account.

### F43. The settled test says yes before money reaches the bank

**Verdict:** Build gap · **Status:** Closed · **Owed by:** Nobody · **Note:** answered by `4bce93d21` and superseded by F88; rewrote §18's open item 1

The build marks a payment settled when its payout row has status 1, its wallet entry has a UTR, or a
scheduler row succeeded (`collectionService.ts:776`). Measured on production, live non-test
properties, online modes: money received on 24 August, the day of this review, already reads 100%
settled, 21 payments, ₹0.7 lakh; the last three days read 98.7% settled, ₹3.92 crore of ₹3.97
crore. Same-day bank landing for every rupee is not plausible, so at least one signal fires when
settlement is recorded or initiated, not when money lands.

Two consequences. Settlement Pending and Unsettled will read near ₹0 always, so the §15 healthy
state would show permanently and mean nothing. And §18's open item 1 held the opposite worry from an
earlier measurement, that a third of online money had no settlement record and would all show as
stuck. Both cannot describe the same bank reality, and neither has been checked against a bank
statement.

Also absent: §11's marking of reversed and failed transfers inside Unsettled.

**Fix:** engineering states what payout status 1 and a wallet UTR each mean, for each step of the
transfer, before this card ships, and adds the reversed marking. The card's shape is right; its yes
test is what needs proving.

### F44. The View all sheet half follows the toggle

**Verdict:** Build gap · **Status:** Closed · **Owed by:** Nobody · **Note:** fixed in `a2d839ea1`, verified at F89

§12: the sheet "reads by the screen's window and toggle". In Due Date view the category rows switch
correctly and carry their billed side. The rest does not: the "This window's collection" rows and
the stay-duration rows still read by paid date (`collectionService.ts:827`), so the sheet's headline
rows disagree with the tiles the manager just left.

**Fix:** in Due Date view the window rows show Billed and Collected & Adjusted, and the stay rows
split Collected & Adjusted.

### F45. The View all Adjusted rows leave Discount out

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek

`adjustedTotal` sums the three adjustment modes only (`collectionService.ts:879`). §3 defines
adjustment as four kinds, discount included, and the Adjusted Collection card shows four. The View
all rows labelled "Adjusted collection" run short of the card by exactly the discount. After F40,
the discount source is the credits records.

### F46. The healthy states are not emitted

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek

§15's healthy table: Still Unpaid at zero reads "Everything billed for this period is collected",
Unsettled at zero reads "All online money has reached your bank", Settlement Pending at zero says
the same in tile form. The service emits empty states and nothing else, so a zero that is good news
renders as a bare zero. Blocked in spirit by F43: until the settled test is proven, Unsettled at
zero is the default, not news.

Findings from F47 are the Expense tab, reviewed 2026-08-24 in sheet order: §4 and the filters
first, then the five blocks and the View all sheet.

### F47. The "same point last month" chip reaches into the current month seven days a year

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek · **Note:** all five services carry the same two lines

The comparison window is built as "first of last month, plus today's date minus one"
(`expenseService.ts:158`). When the previous month is shorter than today's date, that overflows past
the end of it. On 31 May the comparison runs 1 April to 1 May, so it counts a day of the month it is
supposed to be measured against, and the chip flatters or damns the current month with its own
money.

It bites on seven days a normal year, six in a leap year: 31 May, 31 July, 31 October, 31 December,
and 29, 30 and 31 March. Every other day of the year the window is right.

**Fix:** stop the comparison at the end of the previous month. **This is not an Expense mistake.**
The same two lines are written into all five services: `duesService.ts:151`,
`collectionService.ts:177`, `expenseService.ts:158`, `tenantService.ts:167` and
`occupancyService.ts:72`. The fix lands five times or four tabs keep it.

**Audit note, two errors in this entry.** It first said Dues built its window differently and was not
affected. Wrong, and asserted without opening the file. The citations for Collection, Tenant and
Occupancy were then wrong as well: they were read off a multi-file `awk` using `NR`, which counts
lines across all files rather than restarting at each one, so three of the five addresses were
cumulative offsets and pointed past the end of their files. The claim itself held; the addresses did
not. Any citation gathered from more than one file at a time is re-checked per file from now on.

On the Dues half: it feeds `lastMonthSamePoint` straight into `this_month_prev` at
`duesService.ts:207`, the same figure F5 is about, so the Dues chip carries both defects at once and
F5's fix must not be written as if this one were already handled.

### F48. The sheet says three totals must always match, and its own filter rule lets them differ

**Verdict:** Specification gap · **Status:** Closed · **Owed by:** Nobody · **Note:** sheet §3 corrected

§3 read, before this edit: Total Expense on the Overview, on Expense Breakdown and on Top Payers
"must always be the same number". §4: a card can be deliberately set aside on its own dropdown until the top filter next
changes. When a card is set aside the three are correctly different, and the card says which window
it is on.

The build is right: the three are separate queries over one and the same set of rows, so they cannot drift while
they share a window. §3 now carries the exception.

### F49. No chip where the sheet asked for a rupee change

**Verdict:** Accepted change · **Status:** Closed · **Owed by:** Nobody · **Note:** D16; sheet §4 corrected

§4 asked for the rupee change with the percentage dropped where the previous period had no spend.
`chipMoney` and `chipCount` both return null when the previous figure is zero or less
(`expenseService.ts:84`, `:92`), so no chip is sent. Ruled as built, reasoning in D16.

### F50. Four properties in five are told to add their first expense

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek · **Note:** the largest finding on this tab

§13 separates two zeros and states the rule outright: a property with real history and a quiet month
gets the plain line, and the setup message "appears only where no expense has ever existed". The
build ships one empty state per card and never two (`EMPTY`, `expenseService.ts:64`), so no card can
tell the two zeros apart. On one card the single message is the setup line: Expense Breakdown sends
"No expense recorded yet. Add your first expense to start tracking costs." whenever its window
returns no rows (`expenseService.ts:389` and `:391`). The other three send a neutral line each, which
is not wrong for either zero and is not right for either.

**Measured on production**, live properties with test and deleted excluded, counting only properties
that have ever recorded an expense:

| Window | Properties reading zero | Share |
|---|---|---|
| This month to date | 4,197 of 5,183 | 81% |
| Last month, whole month | 4,087 of 5,183 | 78.9% |

So the wrong message is the one almost everybody sees, and it tells an operator with years of
records to start recording. It also invites an action, "Add your first expense", on a card that has
no button (§17.30).

**Fix:** two states per card. The setup copy needs a second test, whether the property has any
expense at all, which is one existence check per card and not per row.

### F51. The Overview row sends its zero nowhere

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek · **Note:** the F24 sweep; same shape as F34

§4 puts the screen-level in-window zero on the Overview row in as many words: "the entry point is
the Overview row... the in-window zero and the narrowed-access note both show there". §17 item 29
lists the same thing as a missing design state. `getOverview`
never emits an empty state, so a window with no spend renders four bare zeros with nothing said.
Every other block on the tab sends its zero somewhere; the screen's headline block does not.

The healthy state is missing too. §13's one piece of good news, "Nothing owed to staff right now.",
is not emitted on the tile or on the View all row that repeats it (§6). 20 live properties read
exactly zero owed today.

### F52. A property the app cannot find shows made-up numbers as though they were real

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek · **Note:** suite-wide, 51 places across five services

Ask for a property that has been deleted, or send a stale property number, and the screen fills with
₹26.3K, 128 expenses, "Jatin" and "Sharma PG". §13 asks for "Select a property to view expenses."

Every block returns null when no live property matches the request
(`if (props.length === 0) return null`), and the caller falls back to the sample data the block was scaffolded with
(`service.ts:174`, `block_data: data ?? def.mock_data`). So a filter naming a property that has been
deleted, or a stale pg_number, renders ₹26.3K, 128 expenses, "Jatin" and "Sharma PG" as though they
were real. §13 asks for "Select a property to view expenses."

The request already refuses an empty property list, so this is the unresolved case, not the
empty one. Counted by service: Dues 11, Tenant 15, Collection 9, Occupancy 9, Expense 7. The lock
path is handled properly and returns a real restricted payload, so only this one branch leaks.

**Fix:** one guard in `toBlock`, not 51 in the services: a null from a real service is an empty
state, and only an unbuilt block falls back to mock.

### F53. Still owed to staff will read fifty crore and can only grow

**Verdict:** Specification gap · **Status:** Closed · **Owed by:** Nobody · **Note:** the same shape as D10; sheet §5 carries the caution; S6 proposes the fix at the source

The tile is built correctly. It is the exact negative of the Personal funds balance the Passbook
screen shows for the same people, same rows, same formula (`teamPassbook.ts:1444` against
`expenseService.ts:260`), so §15.15's test passes properly rather than by luck.

What the sheet did not say is what the number does in practice. The subtracting side barely exists:
reimbursement was first recorded on 19 March 2026 and has been used 23 times platform-wide since,
₹1,100 of it on live non-test properties.

**Measured on production**, live properties with test and deleted excluded, every property with any
personal-funds activity:

| | |
|---|---|
| Platform total | ₹50.77 crore |
| Properties with a positive figure | 1,165 |
| Properties reading exactly zero | 20 |
| Properties reading negative | 0 |
| Median | ₹60,100 |
| Largest on one property | ₹1.90 crore |

Not one property has ever settled below zero. So the tile is honest about the system of record and
alarming about the business, exactly as the held-deposit line is under D10, and the same "I paid him
back in cash" conversation is coming. Sheet §5 now carries the caution and the measurement.

### F54. The staff tile counts refunds staff fronted, and the sheet said two things

**Verdict:** Accepted change · **Status:** Ruled · **Owed by:** Owner for one question · **Note:** D17 settles the refunds; the passbook rows with no property are still open

₹2.32 crore of the ₹50.77 crore is tenant refunds a staff member paid from their own pocket. §5
defines the tile as money staff fronted that has not been paid back, which takes them in; §15.15
called it their "open expense payback", which leaves them out; §3 separately says a refund is not an
expense. The code counts them. Ruled as built, reasoning in D17.

**Also correct as built, checked in the same pass.** The tile reads no date bound of any kind, so it
is genuinely live. It nets off deleted expenses through the reversal rows. It counts the fund the
passbook calls Personal funds and no other, so petty cash and collected money cannot leak into it.

**One small gap left standing.** 46 passbook rows worth ₹70.1 lakh across 30 accounts carry no
property at all, written by the account-level passbook, and the tile reads by property so it cannot
see them. Small, and it needs a decision about what an account-level row belongs to before it can be
fixed.

### F55. The quarantine row always reads ₹0, which is the one thing it exists to avoid

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek

"Rows with a negative or zero amount" is §6's proof that the totals were worked out on clean rows.
The row is built as `inr(Math.abs(n(...)))` (`expenseService.ts:300`). `n()`
(`expenseService.ts:57`) floors the value at zero before `Math.abs` ever sees it, so the absolute
value is taken of a zero; `inr()` (`expenseService.ts:55`) would floor it a second time anyway. So the value is ₹0 on every property and every window. The row count
beside it is correct.

**Measured on production**, live properties with test and deleted excluded, expenses paid in the
twelve months to 24 August 2026: 225 excluded rows across 50 properties, of which 94 carry a
strictly negative amount worth negative ₹10.33 lakh. Those 94 are the ones rendered wrong; the other
131 are exactly ₹0 and would read ₹0 correctly either way.

**Fix:** take the absolute value before either helper, or let this one row through unclamped.

### F56. The no-bill row prints its amount twice

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek

§6 asks for one presentation, an amount and a share together: "₹8.2L of ₹9.8L, 84%". The build sends
the amount as the row's value and the whole phrase again underneath (`expenseService.ts:294`), so the
row says its own amount twice.

**And it is the wrong kind of number.** That row uses `inr()` (`expenseService.ts:54`), which writes
every digit, so it reads "₹1,69,42,000 of ₹1,85,00,000, 92%" where §6 asked for "₹8.2L of ₹9.8L,
84%". The whole point of the short form here is that this is a number a manager has never seen before
and has to take in at a glance.

**Audit note.** This entry first quoted the row as reading "₹1.69Cr", which the code cannot produce.
The wrong quote hid the second defect.

### F57. The View all sheet says today whatever window it is on

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek · **Note:** against D9

The sheet carries `window_label: w.today` (`expenseService.ts:303`), so a Last Month sheet announces
today's date. D9 requires a block that does not simply follow the filter to declare its own time
correctly, and the Dues View all screen already does this properly, carrying the real month name and
financial year on its face.

**Fix:** the window's own name, as Dues builds it.

**Sheet gained a label it never had.** The code adds a row called "Source not recorded" for the
remainder. §6 described that remainder without naming it, and §13 required only that it be "named as
its own line, never guessed from who paid". Good label, now written into §6 so it does not get
renamed later.

### F58. Which clock a paid date is stored on has not been settled

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek · **Note:** a question to answer, not a defect to fix yet

§4 says a day runs midnight to midnight, India time, and §15.14 tests it: an expense recorded at
23:30 on the window's last day counts inside that window. Every Expense query compares
`e.paid_date::date` against dates worked out in India time, but `paid_date` is a timestamp with no
timezone on a database running UTC. If the stored clock is UTC, an expense paid just after midnight
India time falls into the previous day, and at a month boundary into the previous month.

**Five paths write that column and they do not agree with each other**, which is the finding rather
than the timezone itself:

| Where | What it stores |
|---|---|
| `controllers/expenses.ts:460` | `moment(...).add(330, 'minutes')`, an explicit shift to India time |
| `controllers/expenses.ts:52` | `new Date(req.body.paid_date)` |
| `controllers/expenses.ts:1061` | `new Date(req.body.paid_date)`, or now when the field is absent |
| `controllers/expenses.ts:504` and `:634` | the request value assigned raw, with no conversion at all |

Line 460 is proof that at least one path deliberately stores India time. Whether the others land on
the same clock is what needs answering.

**Measured on production**, twelve months, all properties including test and deleted, which is why
this count is larger than F59's: 120,130 of 153,672 rows carry no time of day at all and are
unambiguous whatever the answer. 6,799 rows, 4.4%, sit between 18:30 and 23:59 on
the stored clock, which is exactly the band where the two readings disagree.

**The data alone cannot settle it**, because an evening entry stored in India time and a late-night
entry stored in UTC look identical in the column. Engineering says which it is, and if it
is UTC every window on the tab needs the conversion. Same treatment as F43: the shape is right, the
yes test needs proving.

### F59. Outside the six named groups, capitals split the bar, and a tenth of the money is in there

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek · **Note:** found by the F19 sweep; third tab with this fault

The six named groups match on a case-insensitive prefix and are fine. Everything else falls to the
catch-all, which trims but does not lower-case (`CAT_GROUP`, `expenseService.ts:41`), so a typed
category only merges with itself when the capitals match. Dues groups the same kind of field with
`LOWER(TRIM(...))` at `duesService.ts:220`; Collection was caught with the identical fault as F35.

**Measured on production**, live properties with test and deleted excluded, expenses paid in the
twelve months to 24 August 2026: **323 typed category names differ from another only by capitals,
carrying ₹18.48 crore across 30,302 expenses**, out of ₹185.1 crore and 151,552 expenses on the
tab. A tenth of the money is in a bar that has been split in two or three.

| Split across | Rows | Amount |
|---|---|---|
| Flat Rent · FLAT RENT · flat rent | 174 | ₹1.24 Cr |
| New Setup · NEW SETUP | 1,294 | ₹1.73 Cr |
| Security Refund · security refund · Security refund | 1,371 | ₹1.42 Cr |
| Vegetables · vegetables · VEGETABLES | 2,592 | ₹88.4 L |

A split spelling splits its bar, pushes the real category out of the top three, and pads the Others
row with pieces of it.

**Fix:** group on `LOWER(TRIM(...))` and display `MIN(...)`, exactly as `duesService.ts:217` does.
The named groups need no change.

### F60. The rollup row and a category people actually type share the word "Others"

**Verdict:** Accepted change · **Status:** Closed · **Owed by:** Nobody · **Note:** D18; sheet §7 corrected; the repair is parked as S5

§7 read, before this edit: "Nothing in the sheet is ever labelled 'Other', so the word keeps one meaning on this
screen", and §14 said the screen never renames what somebody typed. Both cannot hold, because people
type it: **4,566 expenses worth ₹3.97 crore over twelve months are saved under a category literally
called Other or Others**, on live properties.

`foldTop3` appends a bar labelled "Others" after the top three (`expenseService.ts:108`), so where a
typed "Others" is one of those three, the card carries two rows of that name and they open different
things.

**Measured on production**, twelve months to 24 August 2026, per property:

| | Properties |
|---|---|
| A typed Other or Others sits in the top three | 207 |
| ...and a rollup row shows as well, so the card reads Others twice | 111 |
| A typed Other or Others falls inside the Others sheet | 362 |

Ruled as built, reasoning in D18.

### F61. FlexiPe is a third of the money and the sheet calls it one in seven

**Verdict:** Specification gap · **Status:** Closed · **Owed by:** Nobody · **Note:** sheet §7 corrected

§7 read, before this edit: "This is not a small row. Roughly one expense in seven leaves through FlexiPe." True by count and
it undersells the row. Measured on live properties over twelve months: 13.4% of expenses, and
**₹60.2 crore of ₹185.1 crore, 32.5% of the money**. It is the largest row on the Payment Mode tab,
ahead of Cash. A reader deciding how much care the FlexiPe path deserves would take the wrong view
from "one in seven".

### F62. A third of all spending shows on the Paid by tab as a bracketed string nobody typed

**Verdict:** Specification gap · **Status:** Ruled · **Owed by:** Vivek · **Note:** D19; sheet §8 corrected

`payerItems` reads `e.payer` and trims it (`expenseService.ts:489`), which is exactly what §8 asks.
The problem is upstream: the FlexiPe path writes the payer as the literal `(Owner)`, brackets
included, so that string becomes a bar on the Paid by tab.

**Measured on production**, live properties with test and deleted excluded, expenses paid in the
twelve months to 24 August 2026. The match is exact, not approximate:

| | |
|---|---|
| Expenses whose payer is `(Owner)` | 20,301 |
| Of those, paid through FlexiPe | 20,301, all of them |
| FlexiPe expenses with any other payer | 0 |
| What it is worth | ₹60.2 crore, 32.5% of all spending |
| Properties showing it | 833 |
| ...that also have a plain `Owner` payer, so the card carries two owner bars | 123 |

§8 defines the tab as "who paid, grouped by the person recorded as having paid". For this money no
person paid it. Ruled as `FlexiPe (Owner)` in D19.

**The F19 capitals sweep was run here and deliberately not raised.** §8 merges names only when
identical, on purpose, so that two different vendors are never fused. Measured so it is not
re-opened: money sitting in a minority spelling is ₹5.67 crore of ₹185.1 crore on Paid to, 3.1%, and
₹17.36 lakh on Paid by. That does not overturn a deliberate rule. Category folds capitals and names
do not, because a category is a label people reuse from a short list and a name is a person.

### F63. The trend's running bar counts days that have not happened

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek · **Note:** found by the F40 one-clock sweep

Every block on this tab caps its window at today. The trend does not: its query runs to
`endOf('month')` (`expenseService.ts:521`). So an expense dated later this month sits in the running
bar while the Overview's This Month total leaves it out, and two cards on one screen disagree about
the same month. §3: a future dated expense "joins every total the day its date arrives".

It also argues with the bar's own marking. The bar is drawn as in progress precisely because the
month is unfinished, and then counts days that have not arrived.

**Measured on production**, live properties with test and deleted excluded, on 24 August 2026:

| | |
|---|---|
| Expenses dated after today, right now | 1, worth ₹1,414 |
| Recorded before their paid date, last twelve months | 4,796, worth ₹8.16 crore |
| ...of those, dated into a later month than the one they were entered in | 267 |

So the money in flight at any moment is small, and the situation recurs a few hundred times a year.

**Fix:** end the trend's range at today, as the other six blocks do.

### F64. Expenses by Property has its empty copy written and never sends it

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek · **Note:** the F24 sweep; sheet §10 and §13 reconciled

`EMPTY.by_property` is defined in full at `expenseService.ts:68` and is the only one of the four
empty states never used. `getByProperty` returns rows and a hint and nothing else
(`expenseService.ts:571`), so a multi-property account with no spend in the window gets a list of ₹0
rows with no explanation.

**The sheet argued with itself and is now settled.** §10 says properties with no spend still appear at
zero; §13 lists empty copy for the card. Both hold under one rule: list every property including the
zeros while anything at all was spent, and show the empty state only when the window's total is zero.
Sheet §10 now says so.

Findings from F65 are the Occupancy (Inventory) tab, reviewed 2026-08-24 in sheet order: §3 and §4
first, then the eight blocks and the View all sheet.

### F65. Not one change chip is built on this tab, and the code for them is written and never called

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek · **Note:** corrects F6's table; absorbs F47's Occupancy half

`chip()` sits at `occupancyService.ts:54` with the correct per-tile polarity written into it, good for
a rising Occupied, bad for a rising Vacant, neutral for Rentable. **It is never called once in the
file.** Every tile hardcodes `trend_object: null` (`occupancyService.ts:204` to `:208`), and the
Occupied tile carries `showChip ? null : null` (`:205`), which returns null on both branches. The
comparison window built to feed it, `prevFrom` and `prevTo` (`:71`, `:72`, `:78`), is computed and
never read.

So §4's polarity table and §5's "each tile carries a change chip" are unbuilt, and the "▲ 4% vs this
day last month" line §4 specifies for Now does not exist.

**This corrects F6.** That entry's table recorded `occupancyService.ts:54` as deciding direction "per
tile", which implied a working chip. The signature is per tile; the function is dead.

**F47 lands here and is currently harmless.** The overflowing "same point last month" window is at
`occupancyService.ts:72`, exactly as F47 said, re-checked in this file alone. Because `prevTo` is
never read, it cannot mislead anybody today. It has to be fixed while the chips are built, not
before, or the first chip this tab ever shows carries the defect on day one.

**Fix:** build the chips §4 asks for, and fix line 72 in the same change.

### F66. Pending evictions are invisible on this entire screen

**Verdict:** Build gap · **Status:** Ruled · **Owed by:** Vivek · **Note:** D21 names it, D22 sets the unit-view number; this is F14's sixth meaning, now traced

F14 row 6 left the Occupancy Under notice tile untraced. Traced here: it reads widget 1402,
`UNDER_EVICTION`, and **the two property structures build it differently**, which the code says out
loud at `helpers.ts:1990`: "old=status IN(1,2), new=status=1".

| Structure | Built at | Counts |
|---|---|---|
| Old | `helpers.ts:1927` | Living tenants **and bookings** carrying a leaving date |
| New | `helpers.ts:1943` | Living tenants carrying a leaving date |

A leaving date is written only when a manager approves, so **the tile is approved evictions only** on
both, and on both it includes people whose date has passed. The booking half of the old-structure
branch is real and empty: **measured on production on 24 August 2026, exactly 1 booking on a
live old-structure property carries a leaving date**, so it moves no number today. It is written down
because it is a difference between two branches of one count, and the next person to touch either
branch should know the two are not the same test.

That makes it the same people, to the person, as the "Eviction approved" bar D6 gave Dues and
Collection. D6 had assumed Inventory's count was date-bound; only Upcoming Vacancy is.

**Measured on production**, living tenants on live properties, test and deleted excluded, on
24 August 2026:

| State | Tenants | On the Inventory tile today |
|---|---|---|
| Approved, date still ahead | 4,772 | Yes |
| Approved, date already passed | 318 | Yes |
| Raised, nobody approved | 1,193 | No, and nowhere else on the screen either |
| Both at once | 3 | Yes, through the approved side |

The 1,193 appear in no tile, no donut slice, no card. The screen that most needs early warning about
space carries the least.

**Two smaller defects on the same tile.** It shows a headcount in both views while labelling itself
"Under Notice Units" in Unit view (`occupancyService.ts:208`), so two tenants leaving one 3-sharing
room read as 2 units. And that label is built with `unit === 'Beds' ? 'Beds' : 'Units'`, a test whose
two branches say the same thing as the variable already holds.

**Fix:** D21 and D22.

### F67. The booked layer and the Unit view's Occupied count take in bookings nobody approved

**Verdict:** Build gap · **Status:** Ruled · **Owed by:** Vivek · **Note:** D20 settles it; this is the F4 question DA-08 was left to answer

§3 is explicit twice over: a booked bed is still vacant, and Booked means confirmed only. The build
reads the homescreen widget, which answers differently in each view:

| Where | What the build does | What §3 asks for |
|---|---|---|
| Bed view, is a booked bed vacant | Yes. `vacant_beds` counts capacity minus status-1 tenants (`helpers.ts:2017`) | Yes |
| Unit view, is a booked room vacant | No. `vacant` and `occupied` count `active_count`, which is status 1 **or 2** (`helpers.ts:2023` to `:2025`), so a room holding only bookings reads Occupied | Yes, it should read vacant |
| The booked layer | Every booking, approved or not: `has_new_booking` is `BOOL_OR(t.status = 2)` (`helpers.ts:1929`, `:1945`) | Confirmed only |
| The forward projection | Every status-2 tenant with a future joining date (`occupancyService.ts:219`) | Confirmed only, and D1 already ruled forecasts this way |

**Measured on production**, bookings on live properties, test and deleted excluded, on 24 August
2026: 4,776 bookings, of which 2,880 are confirmed, being 112 with an explicit approval row and 2,838
on properties that accept automatically, which D1 ruled are the same thing. **1,896, 40%, are waiting
for approval.** D1's own note carries the reconciliation, because counting explicit approvals alone
gives a very different picture.

**Fix:** D20. It reaches past this tab, by the owner's instruction.

### F68. The period rate divides by today's capacity, and the trend beside it does not

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek · **Note:** the F40 one-clock sweep, inside one tab

§18 item 1 sets the rule: "capacity counts as it stood each day". §18 item 5 adds: compute each
number once.

Two blocks on this screen answer "how full was I last month" and they do not agree.

| Block | Numerator | Denominator |
|---|---|---|
| Overview, a period | Day-averaged occupied beds from stay history (`occupancyService.ts:151`) | **Today's** rentable count, from the live widget (`:196`) |
| Occupancy Trend | Day-averaged occupied beds, clipped to the rentable set | Capacity **as it stood that month**, rooms born at `createdAt`, removed at deletion, disabled windows honoured (`:461` to `:465`) |

The code says so itself at `occupancyService.ts:150`: "Rentable denominator = current rentable
(approx; day-averaging it via room_state_log is a refinement)". So the fix was known and deferred,
and the trend went on to write it. The trend is the one that matches the sheet.

**Measured on production**, live properties, test and deleted excluded, on 24 August 2026: **38,999
rentable rooms were created since the start of last month, 7.5% of the 520,052 that exist today,
across 3,902 properties**, and 5,304 rooms were deleted in the same stretch. On every one of those
properties, Last Month's rate on the Overview divides by rooms that did not exist in Last Month, so
the tile reads lower than the trend bar directly beneath it for the same month.

**Fix:** the Overview's period path uses the trend's historised capacity. The query is already
written; §18 item 5 asks for exactly this, one computation reused.

**One small thing worth fixing while there.** The Overview's day-average counts stay rows flagged as
future-scheduled moves, where the trend excludes them (`occupancyService.ts:158` against `:471`). It
is 32 rows platform-wide over last month, so it moves nothing; it is listed here so the fix is made
once and not rediscovered.

### F69. Occupancy Status ships four chips where the sheet asks for two, and two of them were cut by name

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek

§7's chip table has two rows, Under Notice and Over-occupied. §20's design-fix item 33 names the
other two and says to remove them: "Overbooked occupancy" (cut) and "Booked Beds" (lives in the
Vacant legend split). §17 states the same twice: no double-booked chip, and Booked is never a
headline of its own.

The build ships all four in Bed view, the default (`occupancyService.ts:256` to `:261`), including one
labelled literally "Overbooked occupancy", the cut chip's own name, and "Under Notice with Booking",
which §7 asks for as a second line rather than a chip of its own. Unit view gets three: Booked Beds is
correctly gated behind the view at `:258`, so that half of design fix 33 is already honoured on one
side of the toggle.

**And they never hide.** The code comment at `:253` says "always shown incl. zero (matches the
homescreen widget)". §18 item 18: "State chips and layer rows hide at zero", with the sheet's own
appendix noting most properties have none of them. So most properties get a row of four zeros in the
default view.

**Also missing from the same block:** the second line §7 specifies under the tile, which as the sheet
now reads is "24 under eviction · 19 approved · 5 pending · 1 past their date · 2 already replaced".
Past their date is not computed anywhere in `occupancyService.ts`, although the same test is written
on the Tenant tab at `tenantService.ts` in `liveTiles`.

**Fix:** two chips, hidden at zero, with the layers as the second line under Under Notice.

### F70. In Bed view the donut legend does not add up to Total wherever a room is over-occupied

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek

§7 states it as a trust rule in as many words: the legend adds up to Total, and "a legend summing to
one number beside a percentage computed from another is checked once, cannot be reconciled, and is
never trusted again."

In Bed view the legend is occupied beds plus vacant beds plus disabled beds. The widget builds
vacant beds as capacity minus occupied, floored at zero per room (`helpers.ts:2017`), while occupied
beds is the raw headcount (`:2019`). On an over-occupied room the floor holds vacant at zero while
occupied runs past capacity, so the legend comes out **above** the header's Total.

**Measured on production**, live rooms on live properties, test and deleted excluded, on 24 August
2026: **2,166 beds of excess occupancy across the platform**, and the sheet's own appendix puts an
over-occupied room on 1,061 properties, 4.16% of active ones. On each of those the legend overshoots
the header sitting directly above it.

**Unit view has its own version of the same hole, from the opposite direction.** A room is Occupied
when its tenants reach its capacity, Semi-vacant when they are below it, and Vacant when it has none.
A room whose capacity is zero and which holds somebody satisfies none of the three, so it falls out of
the legend and the legend reads **below** the header. Measured the same day: **780 of the 5,958
zero-capacity rooms hold a living tenant or a booking**. F72's fix removes them from Rentable, which
settles the rate; this legend still needs them to land somewhere or be visibly absent.

**Fix:** the legend carries over-occupancy as its own entry, and zero-capacity rooms holding somebody
are shown rather than dropped. Total is what the header promises, and moving Total to absorb the
excess would break §7's own trust rule from the other side. §7's over-100% sentence already exists to
explain the situation in words, so the number has somewhere to sit beside it. The owner confirms the
labels; everything else here is arithmetic.

### F71. The booked layer is a room count printed inside a bed number

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek

§7 asks the Vacant legend to carry its split: "Vacant 38: 17 booked, 21 available". The build
computes booked as `min(max(bookedRooms − underNoticeWithBooking, 0), vacant)`
(`occupancyService.ts:243`). In Bed view `vacant` holds beds while `bookedRooms` holds rooms, widget
1406, one per room with any booking. So a room with three booked beds contributes 1, and the split
reads "Vacant 38: 1 booked, 37 available" where the truth is 3 and 35.

Available is derived by subtraction from the same wrong number, and §7 says available is the number a
manager acts on, which is why it is stated rather than left to subtraction. The subtraction is still
there; only the arithmetic moved.

The code names the limitation itself at `occupancyService.ts:242`, "Room-level approx (no booked-beds
widget)", and Vivek's guide says the same in its Status section: "the booked split is room-level
(there's no booked-beds signal)". Both are honest, and neither makes the number right.

**Audit note.** This entry first cited that guide sentence as though it sat in the service file, at
line 264 of both. It is line 264 of the guide. The same class of mistake as F47's wrong addresses:
a line number carried across from one file to another.

**Fix:** count booked at bed grain where the property can answer it, and say "by rooms" on the split
where it cannot. On a migrated property `tenant_room` carries the bed, so the exact answer exists; on
the rest the count degrades to rooms, which is exactly what §3 already permits: "Counts work
everywhere; per-bed detail exists only on migrated properties, and cards show the count and omit the
detail rather than guess." What is not allowed is today's silent mixture of the two.

### F72. Rooms that declare no capacity count as rentable units

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek

§3 defines rentable as total "minus disabled ones and minus rooms that declare no capacity", and §18
item 4 makes it a test: "one zero-capacity room on a full property leaves the rate at 100%".

Rentable units is built as total rooms minus disabled rooms (`occupancyService.ts:139`). Nothing
excludes a room whose capacity is zero. In Bed view the rule holds by accident, since a zero-capacity
room contributes no beds; in Unit view the room joins Rentable and, having nobody in it, joins Vacant
as well.

**Measured on production**, live rooms on live properties, test and deleted excluded, on 24 August
2026: **5,958 rooms declare no capacity, across 384 properties**. The sheet's appendix measured 2,550
on 7 August; the count has grown, and the two runs are the same query seventeen days apart.

**Fix:** exclude them from rentable units, as §3 already says. **And decide where they go instead**,
because the sheet answers that twice and differently: §3 keeps them out of both Rentable and Disabled,
while §6's View all row folds them into "Disabled for rent". Folding them in tells a manager a room
was switched off on purpose when nobody ever gave it a capacity. The recommendation, now written into
the sheet, is a row of their own named "No capacity set", so the number is visible and can be fixed.

### F73. Vacant Room Status paints Never rented red on almost every property, the one thing §8 forbade

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek

§8 grants this card the suite's one exception to red-for-obligations, and bounds it in the same
paragraph: red marks the largest of the **four aging bars only**, "never rented and Unknown sit
outside the contest: never rented is the biggest bar on almost every property, and a constant red
stops pointing at anything."

The build takes the maximum across all six bars and paints it red (`occupancyService.ts:348`,
`:351`). The sheet's own appendix measured never-rented at 87 to 90% of vacant inventory, so on
almost every property that bar is the maximum and the card ships a permanent red.

§4's red rule states it a second time, as the screen's one granted exception. Two statements, one line
of code.

**Fix:** take the maximum over the first four bars only.

### F74. The one empty-state sentence §16 forbade by name ships verbatim, and no card has a healthy state

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek · **Note:** the same shape as F50 on Expense

§16 separates a card's empty state from its healthy state, and says an empty card here "is usually
good news, a distinction no sibling needs". It then names the design's current copy and rules it out:
it "reads a full house as a data gap ('Vacancy duration by days will appear here once rooms are
empty') and must not ship."

`EMPTY.vacant_rooms` (`occupancyService.ts:22`) ships: "No vacant rooms found. Vacancy duration by
days will appear here once rooms are empty." The forbidden sentence, word for word, inside it.

All four cards §16 gives healthy copy send an empty state instead:

| Card | Ships | §16's healthy copy |
|---|---|---|
| Vacant Room Status | "No vacant rooms found. Vacancy duration by days will appear here once rooms are empty." | "No vacant rooms. Your property is fully occupied." |
| Upcoming Vacancy | "No rooms are being vacated soon. Tenants with upcoming exit dates will show up here." | "Nobody is leaving soon. No confirmed move-outs on the books." |
| Agreements ending soon | "No agreements ending soon." | "No agreements ending in the next 90 days." |
| Where is my property losing money? | "No data yet. Once beds are vacant, you will see where your revenue is being lost." | "No revenue leaking. Every rentable bed is earning." |

Every one of them tells a full house that data is missing. §16's two other states are absent as well:
the whole-screen not-set-up state, which the sheet's appendix calls the most common state this screen
will ever show at 46.6% of real properties, and the rooms-but-no-beds state with its **Add beds**
button.

**Fix:** §16 already holds every word. This is copy that exists and is not wired.

### F75. Upcoming Vacancy follows the view toggle, and the sheet says it must not

**Verdict:** Accepted change · **Status:** Ruled · **Owed by:** Sheet · **Note:** D23; §9 and §4 corrected

§4 and §9 both say this card ignores the toggle and counts per room. The build switches with it:
Bed view counts departing tenants, Unit view counts distinct room-and-day vacancy events, and the
axis label switches with them (`occupancyService.ts:378`, `:389`).

The build is right. Reasoning in D23.

### F76. Upcoming Vacancy paints its largest bar red, an exception the sheet granted only to its neighbour

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek

§4's red rule: red for unmet obligations and passed dates only, with **one** deliberate exception,
the vacancy-age chart in §8. The build gives the same treatment to Upcoming Vacancy
(`occupancyService.ts:390`).

It reads as an alarm about the wrong thing. The tallest bar here is simply the week most people
happen to be leaving in, which on most properties is 31+ days out, the furthest away and the least
urgent. A card about confirmed departures that shades the calmest bucket red teaches an operator to
ignore red on this screen, which is what §4's rule exists to protect.

**Fix:** drop the red on this card, so it keeps §4's rule and the exception stays the single one §8
was granted. The related design question, whether 0 to 7 days should be marked instead as the bucket
with no time left to refill, is logged as design fix 43 in DA-08 §20 rather than left inside this
paragraph.

### F77. More than half of the agreement dates are invented, on the one card the sheet says carries facts

**Verdict:** Build gap · **Status:** Ruled · **Owed by:** Vivek · **Note:** D24 settles it; S7 proposes the fix at the source

§10 is careful about what this card claims: "Agreed end dates are facts about paperwork, not
confirmed departures, and the card says so." §17 makes it a prohibition: "No guesswork in any forward
number... Agreements ending soon uses agreed end dates, facts about paperwork, and says so."

The end date is built in four steps (`occupancyService.ts:404` to `:409`): a recorded renewal date,
then the tenant's own agreement length, then the property's default, then **11 months** where nothing
is recorded. The first three are facts. The fourth is a guess, and it is not marked as one anywhere:
not on the bar, not in the footer, not in the hint copy.

**Measured on production**, living long-term tenants on live properties, test and deleted excluded,
on 24 August 2026:

| | Tenants | Share |
|---|---|---|
| Long-term living tenants, the ones this card can place on a bar | 351,108 | |
| Of those, no renewal record, no agreement length, no property default | **190,609** | **54.3%** |

So for the majority of tenants the card places a bar using a date the product never recorded, and a
manager cannot tell which bars are paperwork and which are arithmetic. It is the same shape as F43 on
Collection: the card's structure is right, its yes test is what needs proving.

**Fix:** D24. The eleven-month fallback stays and is named in the info sheet with its count, and that
count opens the list of the tenants it applies to, so the number that reports the gap is also the way
to close it. This review had recommended dropping those tenants from the card; the owner ruled the
other way and the reasoning is in D24.

### F78. An empty room is priced at zero, so 85% of vacant beds cost nothing

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek · **Note:** the largest finding on this tab; also a doc fix, G10

§12 sets the pricing ladder in three steps: "its own configured rent first, then what the space was
last let for, then the group's average."

The code builds a room's per-bed rent as the average rent of the **living tenants in that room**, and
falls back with `COALESCE(avg_rent, 0)` (`occupancyService.ts:547`, `:554`). There is no second step
and no third. A room with nobody in it has no living tenants, so it has no average, so it prices at
zero. **The rooms that cost the most are the ones priced at nothing.**

**Measured on production**, live rooms on live properties, test and deleted excluded, on 24 August
2026:

| | Vacant beds | Share |
|---|---|---|
| Vacant beds on the card's own definition | 846,655 | |
| Sitting in a room with no living tenant, so priced at ₹0 | **712,694** | **84.2%** |
| Sitting in a room where no tenant has a rent recorded, so priced at ₹0 | **724,028** | **85.5%** |

So "Losing per month" will read a fraction of the truth on most properties, and on a fully empty
property it reads exactly ₹0, which is the property losing the most. The coverage line beside it,
"based on N of M rooms with rent set", is the only thing that hints at it, and it counts rooms where
§12 asks for beds.

**This is also a doc fix.** Vivek's guide describes the built behaviour as "the room's per-bed rent
(its own tenants' average, **falling back to the group average**)". The code's own comment at `:533`
says the same. The fallback is written in two documents and in neither case in the SQL. Logged as
G10.

**Fix:** the group average as the last step, as §12 and the guide both already say. The middle step,
what the space was last let for, stays unavailable for now for the reason §12 gave, and the rent
snapshot on the stay-history row is where it will come from when it lands.

### F79. Every bar on the losing-money card is drawn at zero width, and every share reads 0%

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek

The revenue-loss figure is formatted for display first, into a string like "₹1.24Cr", and the bar and
the share are then computed by stripping the currency out of that string and comparing what is left
against a raw rupee number (`occupancyService.ts:571`). "₹1.24Cr" becomes 1.24, which is then
measured against a maximum in the tens of lakhs, so **the bar percentage and the share both round to
0 on every group on every property.**

The only rows that escape are groups losing under ₹1,000, where the short form prints the rupees in
full; those compare a rupee figure against a rupee figure and read low rather than zero.

§12: "The bar carries the share; the number carries the rupees." Today the number carries the rupees
and the bar carries nothing.

**Fix:** compute the bar and the share from the rupee figures before formatting. The formatted string
is for display only.

**Two smaller defects on the same card.** It sorts by vacant beds where §12 says "Sorted by revenue
loss, highest first", so the ranking answers a different question from the card's title. And the
coverage threshold with its nudge, §12's rule for when the figure is too thin to show and §19's open
item 3, is not built, so a figure resting on two rooms out of ninety shows as confidently as one
resting on all ninety.

### F80. Three blocks ignore the top filter and say nothing at all about it

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek · **Note:** against D9; the F18 sweep

D9's rule: a block that does not follow the top filter says so on its face, in one of three ways.
Every Occupancy block, checked for both halves:

| Block | Reads the top filter | Says so on its face |
|---|---|---|
| Overview Snapshot | Yes | Not needed; carries the window name |
| Occupancy Status | No | Nothing |
| Vacant Room Status | No | "by rooms · As of today" |
| Upcoming Vacancy | No | "From today onwards" |
| Agreements ending soon | No | "From today onwards" |
| Occupancy Trend | No | Its own range control |
| Where is my property losing money? | No | A plain line: "this is the current monthly loss" |
| Property Wise Occupancy | No | Nothing |
| View all sheet | No | Nothing |

**The three silent ones are Occupancy Status, Property Wise Occupancy and the View all sheet.** Each
offers the filter, ignores it, and says nothing, which is the exact case D9 was written for.

**The losing-money card is not one of them**, and this entry first said it was. It ships a plain line
naming the period it follows (`occupancyService.ts:573`), which is the third of D9's three permitted
ways. §4's table asks it to average across the window, which it does not do; that is defensible on
its own terms, because §12 says the past-period cost waits on the room-emptied date and §19 open item
1 has not delivered it. Declaring itself is what makes the gap honest, and it does declare itself.

**Fix:** the three silent blocks carry "As of today", the way Vacant Room Status already does. Then
Property Wise and the View all sheet follow the window once the Overview's period path is fixed under
F68, since they read the same counts.

### F81. The View all sheet is missing seven of its eighteen rows and one of its three sections

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek

§6 lays the sheet out as three plain questions. As §6 now reads after this review's edits it carries
eighteen rows; the build ships eleven of them and two of the three sections
(`occupancyService.ts:621` to `:639`).

| §6 asks for | Built |
|---|---|
| What have I got: Total, Disabled, No capacity set, Rentable, Occupied, Semi-vacant, Vacant, of which booked, of which available, of which never rented, Over-occupied | All but **never rented** and the **No capacity set** row this review added under F72 |
| Who's moving: Under eviction with its approved, pending, past-their-date and already-replaced rows, and Bookings with no space allocated | The headline and already replaced only. **Approved, pending and past their date** are D21's new rows; **Bookings with no space allocated** was always asked for and is missing |
| How fast: Days to Fill | **The whole section is missing** |

Never rented is already computed one block away, in the Vacant Room Status query, so that row costs a
number that exists. Bookings with no space allocated is the row §6 says belongs to Inventory until the
Bookings screen exists, and the sheet's appendix measured it at 603 of 3,676 confirmed bookings,
16.4%, so it is not a rare case. Days to Fill is blocked by open item 1 and should show "—" with its
reason, the way the losing-money card already does for the same measure on the same screen.

The sheet also carries no empty state and no window on its face, which are F74 and F80 landing here.

**Fix:** the seven missing rows, the third section showing "—" with its reason, and the never-rented
number wired from the query that already produces it. Four of the seven are D20 and D21 landing here
rather than pre-existing gaps, so this row grows with those rulings and is one build with them.

### F82. The trend draws empty months from before the property existed

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek

§11: "History before the property joined does not exist and is not invented; the chart starts where
the data starts."

`getTrend` always emits one group per month of the chosen range (`occupancyService.ts:497`), filling
any month the query returned nothing for with zeros. A property two months old on a six-month range
gets four bars reading zero occupancy, which is not "no data", it is a claim that the property stood
empty.

**Fix:** start the chart at the first month with a capacity, which the query already reports as
`rent_room` per month.

### F83. A window ending in the future shows today's numbers under tomorrow's label

**Verdict:** Build gap · **Status:** Ruled · **Owed by:** Vivek · **Note:** F17 logged the cause, D9 the rules; DA-08 §4 rewritten here

The app lets a Custom window end after today, `allow_future_date: true` at the tab level
(`masterConfig.ts:153`). The service then caps the end date back to today (`occupancyService.ts:88`)
and says nothing. So a manager who asks what September looks like is shown August's answer with
September's dates written above it. Neither rule alone would do this: an app that refused the future
date would be honest, and a service that answered it would be useful.

**Underneath sits the dead code F17 logged**, and deleting it is part of the same change.
`OCCUPANCY_DURATION` (`masterConfig.ts:84`) carries Now, This Month, Last Month, Current FY and
Custom, and no Coming up, so `filter_key` can never arrive as `coming_up`. That strands the
`coming_up` case in the window builder (`occupancyService.ts:90`), the whole forward branch of the
Overview (`:184` to `:190`), the `projectedOccupiedBeds` query (`:214`), and the "By 15 Sep" window
label §5 shows as an example (`:92`).

**Fix:** D9's rules on the Custom path, and delete the `coming_up` branches while there. Whether the
forward projection returns at all is a design decision for the Custom path, and the query is written
if it does. Note the projection has its own defect if it is revived: it counts unconfirmed bookings
as arrivals, which D20 now rules against.

### F84. Occupancy holds 9 of the 51 places where a missing property falls back to the scaffold

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek · **Note:** F52 is the finding; this records where they are

Every one of the nine blocks returns `null` when no live property matches
(`occupancyService.ts:167`, `:228`, `:284`, `:363`, `:399`, `:488`, `:531`, `:581`, `:609`), and the
caller ships the registry's scaffold instead. **Eight of the nine then show made-up numbers**: 360
rentable units, a 25% occupancy rate, "Sharma PG" and "Jai Dada PG" (`registry.ts:290` to `:369`).
The ninth, the View all sheet, was scaffolded empty (`registry.ts:661`), so it shows an empty sheet
rather than an invented one.

The fix is F52's one guard in the caller, not nine here.


Findings from F85 verify Collection against the two fix commits, `4bce93d21` "payment settlement
logic changed" and `a2d839ea1` "collection due view all sheet data", 2026-08-25.

### F85. Refunds never came off collected money, and this review missed it. They do now

**Verdict:** Build gap · **Status:** Closed · **Owed by:** Nobody · **Note:** fixed in `4bce93d21`; the miss is this review's, see the audit note

Sheet §3 says refunds come off everywhere, "every tab, every bar, every property row and every View
all row", and §17.4 makes it a test. The build did not do it. `COLL_AMT` valued every payment at the
bill's full amount. A fully refunded bill did drop out, because refunding moves the bill to status 3
and the base join keeps only status 1, but a **part refund left the bill at status 1 and its full
amount still counted**.

`4bce93d21` adds a refunds subquery grouped by bill and values each payment at
`GREATEST(bill amount − refunds, 0)`.

**Measured on production**, July 2026, live non-test properties, successful active payments against
paid bills, adjustment modes excluded: netting refunds moves Total Collection from ₹238.90 crore to
₹238.25 crore, ₹64.6 lakh across 1,095 payment rows. No bill is refunded beyond its own amount, so
the clamp at zero never fires today and is there for safety.

The refunds table carries no status and no active column, so every row is a completed refund and
needs no filter. One latent flaw, not worth fixing today: the refund is subtracted once per payment
linked to the bill, so a bill paid by two payments would net its refund twice. On production 107 of
the 302,114 bills paid in July carry more than one payment, and exactly one of those is refunded.

**Audit note.** This review checked Collection block by block and did not notice that a rule stated
in §3 and tested in §17.4 was absent from the code. It read the code for what each block claimed to
do and never asked which of the base rules had been implemented at all. The sweep that would have
caught it did not exist; it does now, see the status board.

**The fix breaks two other rules.** See F86.

### F86. Collection now matches neither the list it drills into nor the homescreen

**Verdict:** Build gap · **Status:** Open · **Owed by:** Owner first · **Note:** the §1 goal, now measurable

DA-02 §1 states the goal: "the homescreen's collection figure, the collections list and the old
collection widgets do not produce one answer. Making them one answer is part of this build." Three
answers still exist, and after F85 the analytics figure sits between two of them, matching neither.

Measured on production for July 2026, live non-test properties, on one shared base: successful
active payments against paid bills, adjustment modes excluded.

| Surface | Refunds | Discount, held as `owner_credits` | July |
|---|---|---|---|
| Collections list, the total under every drill | not taken off | only on mode 213, which is ₹0 in July | ₹238.90 Cr |
| Analytics Total Collection, after `4bce93d21` | taken off | not taken off | ₹238.25 Cr |
| Homescreen collection, the figure on the header | taken off | taken off | ₹237.08 Cr |

The two gaps are exactly the two rules: ₹64.6 lakh of refunds separates analytics from the list,
₹1.17 crore of discount separates it from the homescreen.

Before `4bce93d21`, analytics agreed with the collections list to the rupee. That is what
`collectionService.ts` opens by claiming it does, and what D12 and D13 rested on. It no longer does.
**Test §17.12, "a drill adds back to the number tapped", now fails by ₹64.6 lakh a month across the
platform**: tap Total Collection, land on the collections list, read a bigger number than the one
tapped.

The homescreen carries both definitions itself, in two different places: the figure on its header
takes refunds and discount off, and its Financials block does neither. So this is not analytics
disagreeing with the product. It is the product disagreeing with itself, which is what §1 said this
build was for.

**This review's view:** the homescreen header's definition is right and the other two should move to
it. A refund is money given back and a discount is money never taken, so neither is collection.
It is also the only one of the three that implements §3 in full. The discount is then the last piece,
and F40 has to be fixed anyway.

### F87. The destination rows now overstate settled money by 14%

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek · **Note:** F42's coverage is fixed; this is new

F42 found "Where settled money went" reading one settlement table and covering under 3% of settled
money. `4bce93d21` rewrites it to three sources: direct bank payouts, wallet money moved to a bank
through the scheduler, and a "Flexi Pay" row for wallet money not yet moved. **The coverage problem
is fixed.**

The rows no longer add up. §11: "The rows break down Total Settled only and add up to it."

**Measured on production**, live non-test properties, online modes, successful active payments
received 18 to 24 August 2026: Total Settled reads ₹10.33 crore and the destination rows sum to
₹11.78 crore, ₹1.45 crore too much, 14% over.

All of the overstatement is in the direct-payout branch, and the cause is the join that names the
account: it matches an account record on the account number **or** the UPI address, and more than one
account record can carry the same number. 7,617 payments become 9,367 rows after that join, and every
duplicate counts its payment again, drawing ₹6.52 crore of direct payouts as ₹7.97 crore.

**Fix:** choose one account record per payout before summing, rather than joining and then grouping.
Nothing else in the block needs to change. The weighting used to split one payment across several
banks is right, and masking the account number to its last four digits is better than the sheet asked
for.

### F88. "Settled" now counts money sitting in Flexi Pay, and the sheet reserves that word for the bank

**Verdict:** Build gap · **Status:** Open · **Owed by:** Owner first · **Note:** supersedes F43

Sheet §3 reserves the word: "**Settled**, reserved for one meaning: money that has physically reached
the owner's bank." §11 repeats it for the tile.

`4bce93d21` replaces the settled test with "the payment has a payout record or a wallet record", and
rewrites the hint text to match: a Flexi Pay balance now counts as settled "even before withdrawal".

**Measured on production**, July 2026, live non-test properties, online modes: of ₹141.55 crore
counted as settled, ₹90.19 crore went out as a direct bank payout, ₹3.69 crore moved from the wallet
to a bank, and **₹47.67 crore, 34%, is a Flexi Pay balance that has not moved to a bank.** On money
received in the last seven days it is ₹3.52 crore of ₹10.33 crore.

The new meaning is defensible: a Flexi Pay balance is the owner's money and they can withdraw it. It
is also honest on the card, because the rewrite gives it its own named row rather than blending it
into a bank total. But it is not what the sheet's word means, and that row sits under the heading
"Where settled money went", which is untrue of money that has gone nowhere.

One rule is lost. §11 says "money whose transfer was reversed or failed is marked within Unsettled".
The payout's own status is no longer read, so a failed payout now counts as settled purely by
existing. On July's data no payout carries a failed status, so nothing is mis-stated today. This is a
rule that will bite the first time a payout fails, not a live defect.

**What this settles about F43.** F43 reported that money received the same day already read as
settled, and said the yes test could not be describing the bank. That was right, and the mechanism
now has a name: a wallet entry is written when the money arrives. F43 asked whether the test meant
the bank. The answer is no, and the product has now decided that it does not have to.

### F89. The View all sheet now follows the toggle

**Verdict:** Pass · **Status:** Closed · **Owed by:** Nobody · **Note:** F44 fixed in `a2d839ea1`

F44 found the View all sheet following the toggle only halfway: the category rows switched with the
view, the headline rows and the stay rows stayed on money received. `a2d839ea1` fixes both. In Due
Date view the window group now reads Collected & Adjusted against billed, with Still Unpaid beneath
it, and the stay rows split Collected & Adjusted and carry their billed side, which is what §12 asks
and what the category rows already did.

Checked in the same pass and untouched by either commit: F45 stands, the Adjusted rows in this same
sheet still leave discount out.

Findings from F90 are the Tenant tab, reviewed 2026-08-25 in sheet order: §4 and the filters first,
then the fourteen blocks and the View all sheet. Tenant line numbers in these entries resolve
against commit `6797eec6d`, the head this review ran on. The only change to the tenant files since
the pinned `3a13e08ac` is comments (`950d46d1e`), so behaviour is identical and every earlier
finding's function name still holds.

### F90. One junk property setting kills the whole Tenant page, and an Occupancy card with it

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek · **Note:** S8 proposes the fix at the source

The agreement end date rule reads the property's default agreement length with a plain cast to a
number, inside `ENDS_ON` (`tenantService.ts:176`). The column is text and the settings screen
accepts anything. One live property has "." stored there. On that property the cast throws, the
query fails, and nothing catches it: the page resolves every block in one `Promise.all`
(`service.ts:84`), so one failed block fails the whole page. The Tenant tab dies for that property,
because Journey, Agreement Expiry and Renewal & Retention all read `ENDS_ON`, and the Agreements
ending soon card on Inventory carries the identical cast (`occupancyService.ts:407` to `:408`).

**Measured on production**, live properties, test and deleted excluded, on 25 August 2026:

| Stored as the property's agreement length | Properties | Active tenants on them |
|---|---|---|
| "." , the cast throws | 1 | 37 |
| A negative number, "-5" or "-37" | 61 | 1,415 |

The negative ones do not crash; they mislead quietly. A default of -37 puts a fallback tenant's end
date about three years before they joined, so every such tenant reads Already expired.

**Fix:** guard the cast in both services, digits only and otherwise null. What null falls back to now
differs by tab, because of D26: on Inventory the eleven-month step takes over as before, and on
Tenant there is no eleven-month step any more, so the tenant joins the No term recorded group. Add a
catch in the caller too, so one bad block can never take a page down. S8 proposes validating the
field where it is typed, which is where this class of defect dies.

### F91. The forward window rules never landed on Tenant's Custom path, and F17's dead code is still here

**Verdict:** Build gap · **Status:** Ruled · **Owed by:** Vivek · **Note:** D9 rules it; F17 logged the cause; the Tenant twin of F27 and F83

The app lets a Custom window end after today (`allow_future_date: true`, `masterConfig.ts:165`).
The service caps the end back to today and says nothing (`tenantService.ts:137`), so a manager who
asks about September is shown August-to-date under September's dates, F83's exact shape. Move-ins,
Move-outs, Notices Raised, Approved Bookings and the View all's What moved section all quietly read
the capped window.

Underneath sits the dead code F17 already recorded as the most on any tab: the `coming_up` case in
the window builder (`tenantService.ts:149`), the projected tile variants (`projectedTiles`, `:311`),
the scheduled Move metrics branch (`:470`), the two dash tiles (`:260`, `:265`) and the Renewal
dashes (`:1003`). The filter list carries no Coming up, so none of it can run.

**Fix:** D9's rules on the Custom path, and delete the dead branches while there. Worth saying: the
dead code is the best forward path in the suite, dashes with reasons and a projection built from
confirmed facts only, close to what D9 asks Custom to do. The work is mostly moving it, not writing
it. Sheet §4 is rewritten onto the Custom path in this pass, the edit the open items owed.

### F92. Ten blocks ignore the top filter and not one of them says so on its face

*Ten blocks, plus Journey's live bars, whose card as a whole does follow the filter through its
window strip.*

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek · **Note:** against D9; the F18 and F80 sweep

Most of this screen is live by design, and §4 requires a live number to say "as of today" on its
face. Only the Overview tiles and the Renewal overdue card do; the block registry, the code that
wraps every card, adds no subtitle to any Tenant block. Checked block by block: Verification, Tenancy Details, Profile, Details, Stay Type,
Renting Type, Tenure, By Property and Journey's live bars offer the filter, ignore it, and say
nothing. Upcoming Eviction and Agreement Expiry ignore it and are owed "From today onwards" by §11
and §12.

**Fix:** the D9 face rule, the way Vacant Room Status already does it on Inventory: "As of today" on
the live blocks, "From today onwards" on the two forward cards.

### F93. Approved Bookings counts every tenant added on an auto-accept property, and the Journey funnel is built on the same mistake

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek, then the owner · **Note:** the largest finding on this tab

How the tile decides "a booking confirmed in the window" (`approvedBookings`,
`tenantService.ts:330`): take every tenant at status 0, 1 or 2; on properties that need approval,
keep those whose confirmation row is approved, dated by the approval; on auto-accept properties,
keep everyone, dated by the day the record was created. A confirmation row is only written when
somebody acts on a booking (`acceptBooking.ts:531`, `tenant.ts:3621`), and on an auto-accept
property nobody acts. So there the query cannot tell a booking from a tenant added straight to the
list, a direct add, and it counts them all.

Most confirmed bookings are auto-accepted (D1's note: 2,838 of 2,880). **Measured on production**,
July 2026, live properties, test and deleted excluded, the tile's own query run platform-wide:

| | Count |
|---|---|
| What the tile counts for July | 43,909 |
| Of those, a real approval row | 1,189 |
| Of those, on an auto-accept property with no booking record of any kind | 42,720 |
| Of the 42,720, still a booking today | 185 |
| July's Move-ins, for scale | 44,203 |

The tile is within rounding of Move-ins. It is counting tenants added, not bookings approved.

The same test is reused three more times and each inherits the fault:

| Where | Claims | Counts |
|---|---|---|
| Journey, Total (`bookingBars`, `:633`) | Booked in the window | Every tenant record created in the window, on every property, no booking test at all |
| Journey, Approved (`:635`) | Confirmed | Created in the window, on auto-accept or with an approval row |
| Journey, Converted (`:637`) | A booking became a tenant | Created in the window and now living or moved out, which is every direct add |
| View all, Bookings confirmed (`movedCounts`, `:518`) | Confirmed in the window | Real approval rows only. Correct, the only one of the four that is |

**What is needed, in order.** Vivek says whether a booking on an auto-accept property leaves any
trace at all: a marker on the tenant, a row some flow writes, anything. If a trace exists, all four
numbers count it. If none exists, the honest tile counts explicit approvals only and says so on its
face, and whether that is acceptable is the owner's call. The clean interim already exists:
`movedCounts` counts real approval rows only, and the tile could reuse it today.

### F94. The Overview info sheet defines seven of its nine numbers, and one definition is wrong

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek

`tenHints.overview` (`tenantHints.ts:62`) has no entry for Eviction Approved or Past their date, the
two tiles whose meanings most need stating: §3's whole point about Past their date is that the
action is check and close, not chase. And the Active Bookings entry reads "Confirmed bookings not
yet moved in" (`tenantHints.ts:10`), which is wrong. The tile counts confirmed and awaiting together
(sheet §5, and the code counts plain status 2). The definition contradicts the number it defines.

### F95. A cancelled booking counts as a move-out, failing a test the sheet spells out

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek · **Note:** found by the F85 rule-by-rule sweep; S2 is the same flow

§3: "A cancelled booking is not a move-out either", with its own test: cancel a booking, Move-outs
is unchanged and Cancelled rises by one. The cancellation flow fails it: rejecting a booking calls
the shared delete path, which stamps the tenant with a leaving date
(`controllers/tenant.ts:21500`, `UPDATE tenant SET status = 0, date_of_eviction = ...`). Move-outs
counts status-0 tenants by exactly that date (`movedCounts`, `tenantService.ts:513`), so the
cancelled booking joins Move-outs, and Journey's Churn figure, built on the same count, rises with
it.

**Measured on production**, live properties, July 2026: 36 cancelled bookings sit inside the month's
28,783 move-outs. Small, and it is the one departure the sheet names as never belonging there.

**Fix:** exclude tenants whose latest booking confirmation is cancelled from Move-outs, or exclude
the delete path's own marker, `reason_of_eviction = 'Booking cancelled'`. The Bookings cancelled
count already reads the confirmation rows correctly.

### F96. Journey's Under Notice bar counts only the pending half, so one label carries two meanings on one screen

**Verdict:** Build gap · **Status:** Ruled · **Owed by:** Vivek · **Note:** D29 rebuilt this card and settles it: the bar becomes Eviction pending under its own name

Sheet §8: the bar is "Living here, has said they are leaving", the parent group, and the three bars
"overlap each other, parallel bars only". The build's bar tests only a pending notice with no
approved date (`tenantService.ts:544` to `:545`), so it is Eviction Pending wearing the parent's
name, sharing nobody with the Approved Eviction bar beside it.

The View all sheet's Under notice row reads the parent test, `UNDER_NOTICE` (`:402`), pending plus
approved. So "Under notice" means the pending half on Journey and the whole group on the View all
sheet, on the one screen D6 left unchanged because its labels were consistent.

**Fix, superseded and simpler.** This entry proposed the bar read the parent test `UNDER_NOTICE`
(already written at `:190`), so that one label meant one thing. D29 then rebuilt the card with both
states as their own rows, which settles it a better way: the bar keeps the number it already computes
and gains the honest name, Eviction pending, beside a sibling row for Approved eviction. The View all
sheet's Under notice row keeps the parent test and gains the D28 name, Under eviction.

### F97. The Journey strip: Churn is a bare count, and "left early" measures against the wrong date

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek

**Churn** ships as the raw move-out count (`tenantService.ts:586`). §8 defines it as "left, against
who was here", and the tap matrix calls the strip figures rates. Fix: the share, with the count
beside it, which is design fix 23's own rule.

**Tenants who left early**: §3 defines it as left before the lock-in period ended, and the tenant
record carries `lockin_period`. The build compares the move-out date against the agreement end date
instead, `ENDS_ON` with its eleven-month fallback (`journeyFooters`, `:607` to `:611`). Lock-in and
agreement length are different facts: a one-month lock-in inside an eleven-month agreement is
ordinary, and F100 measures the fallback at more than half of long-term tenants. So the figure reads
"left before an agreement date the product half invented", overcounting heavily, with no coverage
line although the sheet's own measurement puts lock-in on 37% of departures.

### F98. Four colour-rule breaches, including a red that every new tenant triggers on arrival

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek · **Note:** the F73 sweep, does the code honour the colour bound

§4 bounds red to unmet obligations and passed dates, and adds "A property onboarded this morning
must not open red." Checked on every bar and legend:

| Where | Ships | The rule says |
|---|---|---|
| Police verification, "Not done, in time" | Red (`tenantService.ts:690`) | Plain. §9 marks only Overdue red. Every tenant in their first week shows red on arrival |
| ID verification, "Not verified" | Red (`:686`) | Plain. §4's red list names police past deadline and no agreement, not identity |
| Agreement Expiry, "30 days" band | Red (`:830`) | Plain. A deadline not yet reached |
| Tenancy Details, "Not signed" | Orange `#FF9567` (`tenantColors.ts:27`) | Red. §10 marks it in as many words and §4's list carries it |

One oddity to fix in the same file: Profile Completion colours Completed `#FFD99A` and Pending
`#10B981` (`tenantColors.ts:28` to `:29`), a yellow for done and a green for not done, backwards to
read even though Pending is rightly not red.

### F99. The eviction tiles and the Upcoming Eviction card read two different records, and §11's one-number rule was impossible as written

**Verdict:** Build gap and Specification gap · **Status:** Open · **Owed by:** Vivek for the source; sheet §11 corrected

**The build gap.** The tiles read the tenant record: an approved eviction is the tenant's own
confirmed date (`liveTiles`, `tenantService.ts:297`). The card reads only the notice records
(`getUpcomingEviction`, `:764`). The code's own comment at `:186` names the trap: approval can set
the tenant's date and leave no active notice row. Measured on production, live properties, 25 August
2026: **275 living tenants carry a future confirmed leaving date and no active notice row with a
date.** All 275 are in the Eviction Approved tile and none is on the card. The card needs the
tenant-level fallback the tiles already use; with it, the card's approved side and the tiles agree
by construction. The tenant list's eviction tags read the notice records too, so Vivek should say
which record is authoritative and align all three, the F86 sweep's question.

**The specification gap.** §11 said "The Overdue bar and the Past-their-date tile are one number,
computed once." §3 and build guidance 9 say a pending notice whose requested date has passed appears
in Upcoming Eviction and never in Past their date. Both cannot hold: the Overdue bar splits into
pending and approved, and its pending part is exactly who the tile must exclude. Measured the same
day: **463 living tenants sit in that pending slice of Overdue.** The build follows guidance 9,
which is the right half of the contradiction to keep. §11 now says the tile equals the approved part
of the bar, and names the pending gap.

### F100. Three quarters of the Already expired bar rests on a date the product invented

**Verdict:** Build gap · **Status:** Ruled · **Owed by:** Vivek · **Note:** D26 settles it; F77 and D24 are the same defect one screen over

The sheet knew about the fallback and asked for a line, and the line ships: "Dates are assumed at 11
months where no duration is recorded" (`tenantService.ts:835`, and the hint says the same). What D24
ruled for Occupancy is not here: no count of the tenants it applies to, and no drill from the count
to the list where the lengths get filled in.

And this card has a band Occupancy does not. On Inventory an invented date mostly drops a tenant off
the 90-day card. Here it lands them on the first bar, drawn first and red. **Measured on
production**, live properties, living long-term tenants with a joining date, 25 August 2026:

| | Tenants | Share |
|---|---|---|
| On the card | 346,930 | |
| On the eleven-month fallback | 187,385 | 54.0% |
| The Already expired bar | 167,001 | 48.1% |
| Of that bar, on the fallback | **123,287** | **73.8%** |
| The 30 days band | 7,504 | |
| Of that band, on the fallback | 3,208 | 42.8% |

Any tenant with no recorded length who joined more than eleven months ago reads Already expired.
That is most long-stay tenants, which is why the bar is the biggest on most properties, and Renewal
overdue, the same number by §15, inherits all of it.

The sheet also argues with itself here: §12 says the five bars cover everyone under §3's rule, while
the measured-figures table says the Already-expired bar is "honestly computed" at about 22,000 from
recorded and derived terms, "not the 188,568 the blanket assumption gives". The build built §12.
D26 ruled for the measured-figures reading: the fallback tenants become their own stated
"No term recorded" group with the count and drill, and every dated band is a fact.

### F101. Stayed after notice ships as a count where the sheet defines a share, and counts a day early

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek

§15: "Of those whose leaving date fell in the window, the share still here." The build counts living
tenants whose confirmed leaving date falls in the window (`tenantService.ts:996` to `:998`) and
ships that count alone: no denominator, no share. The window also runs through today, so a tenant
whose date is today counts as having stayed while §3 counts a person "only after the date passes".

**Fix:** the base is everyone whose confirmed leaving date fell in the window and has already passed,
whatever their status now; the figure is the share of them still living here, with the count beside
it.

### F102. Renting Type renders the stay split under another name, which the sheet forbade in as many words

**Verdict:** Build gap · **Status:** Ruled · **Owed by:** Vivek · **Note:** D27 settles it; the build says out loud that it is interim

§16: the B2B and Residential split renders only where renting type is recorded, "a missing value is
not an answer." The field is unpopulated on production, and the build substitutes the short against
long term split, marked interim in the code (`tenantService.ts:1040` to `:1042`) and in Vivek's
guide. So the screen carries the same two numbers twice in a row: Stay Type as two cards, then the
identical split again as a donut named Renting Type. A manager who trusts the donut's name learns
something false from a card that is restating its neighbour. D27 ruled the card hides until the
field is captured.

### F103. No healthy state exists, six empty states are written and never sent, and the not-set-up screen is missing

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek · **Note:** the F74 and F50 shape, third tab running; the F64 sweep

§21 separates seven empty-looking situations. The build has one kind of state, per-block empty copy,
and even that is half wired:

- **None of the five healthy messages exists**: Upcoming Eviction's "Nobody is leaving soon.",
  Agreement Expiry's "No agreements ending in the next 90 days.", Renewal & Retention's "Nothing due
  for renewal.", Past their date's "Every move-out is closed off.", Journey's "All quiet." A fully
  quiet property gets the empty copy instead, which reads good news as missing data: "Once an
  eviction is approved, you will see the tenant count by days remaining." on the healthiest card the
  screen has.
- **Six blocks define empty copy that is never sent** (the F64 sweep): `EMPTY.move`,
  `EMPTY.renewal`, `EMPTY.stay_type`, `EMPTY.renting_type`, `EMPTY.tenure` and `EMPTY.by_property`
  (`tenantService.ts:86` to `:91`) have no `empty_state` branch in their blocks, so those cards show
  bare zeros.
- **The whole-screen not-set-up state does not exist**: "No tenants yet. Add your first tenant and
  this page fills in.", the most common state on the platform by the sheet's own measurement. A
  never-used property renders fourteen cards of zeros instead. The Overview row itself sends its
  zero nowhere, the same shape as F51 on Expense and F34 on Collection.
- **One copy-paste error**: the Journey Bookings tab's empty message describes the Tenants tab,
  "Once bookings come in, you will see their status ... active, under notice, approved for eviction
  & renewals due in 30 days." (`tenantService.ts:64`).
- §21's exact wording ships nowhere; the shipped copy is the build's own. §21 already holds every
  word, so this is copy that exists and is not wired, exactly as F74 found on Inventory.
- Small, in the same area: §23.14 hides layer rows at zero. Only awaiting-confirmation hides
  (`:445`); the approved, awaiting and past-their-date rows render at zero.

### F104. Recorded answers that fit no bucket are dropped or mislabelled, against the card's own coverage line

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek

Three cases, one class, measured on production, live properties, living tenants, 25 August 2026:

| Where | What happens | Scale |
|---|---|---|
| Food Pref | A recorded value matching none of the four patterns is counted as recorded and drawn on no bar | 1,824 of 11,040 recorded |
| Gender | Anything that is not a male or female spelling is shown as "Prefer not to say" | 10,184 tenants |
| Police verification | A tenant with no joining date fits neither "in time" nor "overdue" | 4,310 active tenants, 1.2% |

On Food Pref the bars then sum to 83.5% of the card's own "based on N recorded" line. On Gender the
label claims a choice the tenant never made, and the swept-up values include "girls", a female
spelling the pattern list missed, alongside junk like "any" and "not sure". On police verification
the tenant stays in the card's base while fitting no bar, so the bars sum short of the base; Tenure
drops the same people silently.

Deliberate, commented, checked and left alone: a date of birth implying an age under 14 is treated
as invalid and excluded from the age coverage (1,154 tenants), which keeps the bars a true split of
their coverage.

**Fix:** an "Other" bar for recorded but unmatched food values; "girls" joins the female list and
junk gender values leave the coverage rather than being relabelled; tenants with no joining date
become a stated line or leave the base, following §21's not-recorded pattern, where a card draws what
exists and states its coverage rather than folding the gap into a bucket.


### F105. Discount is stored twice, and the two stores disagree by a sixth

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek · **Note:** corrects F40's fix line; grounds the F86 ruling

Found while preparing the F86 ruling, which turns on whether discount comes off collection. The
product records a payment-time discount in two places:

| Where | What it is |
|---|---|
| `payments.owner_credits` | A column on the payment. What the homescreen's collection figure subtracts (`homepage/service.ts:586`) |
| the `credits` table | One row per discounted payment. What F40 told Vivek to read |

**Measured on production**, July 2026, live properties, test and deleted excluded, on the collection
base of successful active payments against paid bills with adjustment modes excluded:

| | Payments | Rupees |
|---|---|---|
| Carry both, and the two agree to the rupee | 683 | ₹89.0 lakh |
| Carry `owner_credits` with **no** credits row | **299** | **₹16.0 lakh** |
| Carry a credits row with no `owner_credits` | 0 | ₹0 |
| `owner_credits` in total | 982 | **₹1.05 crore** |

So the credits table is a partial copy, missing about a sixth of the money and never holding anything
the column does not. That explains why F40 measured ₹89 lakh for July and F86 measured a larger
figure for the same month: they read different stores, and both were right about the store they read.

**Two consequences.** F40's fix as written would leave the Adjusted Collection card permanently short
and unable to agree with the homescreen, so its source line is corrected in that entry. And the F86
ruling, if it takes discount off collection, has to name `owner_credits` as the one store, or the
three surfaces will still disagree after the work is done.

**Not answered here:** why 299 payments never got a credits row. It could be an older flow, a failed
write, or a second path that only sets the column. Vivek should say which, because if the credits
table is meant to be complete it is dropping rows, and if it is not meant to be complete then it has
no reader left once F40 is fixed.

---

## Decisions ruled

| # | Date | Rules on | What was decided |
|---|---|---|---|
| D1 | 23 Aug | F2 | Projected Due counts confirmed bookings only; auto-accepted means confirmed |
| D2 | 23 Aug | F1, F9 | Current FY means 1 April to today, tile and filter option alike; the chip stays |
| D3 | 23 Aug | F7 | The Dues (Live) captions were a sheet error; the sheet was fixed |
| D4 | 23 Aug | F8 | Received on Bills Summary means collected against these bills, whenever it arrived |
| D5 | 23 Aug | F4 | A raised bill is owed whatever the booking's approval state; only forecasts need the confirmed test |
| D6 | 23 Aug | F12, F13, F14 | Money screens show Eviction pending and Eviction approved as separate bars; Tenants keeps Under notice |
| D7 | 23 Aug | F16 | The Overdue Breakup dropdown is removed |
| D8 | 23 Aug | Block 6 | Upcoming Dues becomes the day-by-day breakdown of the Projected Due tile |
| D9 | 24 Aug | F17 | The forward window rules, and every block declares its own time on its face |
| D10 | 24 Aug | F22 | Deposits held for departed tenants get their own line under the gauge |
| D11 | 24 Aug | F32 | The Due Date window on a running period ends at today, like every money window |
| D12 | 24 Aug | F28 | Advance stays outside Total Collection, which matches the homescreen |
| D13 | 24 Aug | F29 | Collection is counted at the bill amount; processing charges live on the settlement side |
| D14 | 24 Aug | F30 | Settlement Pending carries no chip; nothing records what it stood at before |
| D15 | 24 Aug | F41 | Collection by Property follows the toggle, like Breakup |
| D16 | 24 Aug | F49 | No chip where the previous period had no spend, on every tab |
| D17 | 24 Aug | F54 | Still owed to staff counts everything staff fronted, refunds included |
| D18 | 24 Aug | F60 | The two meanings of "Others" stay for now; renaming the rollup is parked |
| D19 | 24 Aug | F62 | FlexiPe spending is its own payer row, named "FlexiPe (Owner)" |
| D20 | 24 Aug | F67, F4 | Only a confirmed booking counts as booked; the homescreen and rooms list are fixed alongside |
| D21 | 24 Aug | F66, F14 | Inventory's tile becomes "Under eviction" and takes in both approved and pending, with the parts always named |
| D22 | 24 Aug | F66 | In Unit view the tile counts units holding someone under eviction, not people |
| D23 | 24 Aug | F75 | Upcoming Vacancy follows the view toggle, and the sheet was wrong |
| D24 | 24 Aug | F77 | The eleven-month assumption stays, named in the info sheet with its count, and the count opens the list of tenants missing it |
| D25 | 25 Aug | All citations | Line numbers are pinned to the commit each tab was reviewed against, not re-mapped as the code moves |
| D26 | 25 Aug | F100 | On Tenant's Agreement Expiry, fallback tenants leave the dated bands and become one stated "No term recorded" group with the D24 count and drill |
| D27 | 25 Aug | F102 | Renting Type hides until the field is captured; the interim stay-split donut is removed, not renamed |
| D28 | 25 Aug | S3, F96 | The parent group is named "Under eviction" suite-wide; Tenants adopts it, and no number changes |
| D29 | 25 Aug | F96, Journey (block 3) | Journey's Tenants tab carries six rows in two named groups: the eviction split under Active tenants, and the paperwork pair that overlaps it |
| D30 | 25 Aug | Upcoming Eviction and Agreement Expiry (blocks 6, 7) | A bucket outside the run of days hides at zero; a bucket inside the run always shows. Applies on Tenant and on Inventory |
| D31 | 25 Aug | Agreement Expiry (block 7) | Agreement Expiry's labels each carry their own days, and Valid becomes "Valid (90+ days)" |
| D32 | 25 Aug | F102, D27 | Stay type always shows, zero side included; renting type shows only when either side is above zero |

### D1. Projected Due counts confirmed bookings only (2026-08-23)

Owner: "For bookings, the projected due should include confirmed bookings. That is a decision that we
want. Unconfirmed bookings should be left out." And later: "Auto-accept equals 1, and confirmed
equals 1. They are the same thing."

**Why.** A projection is a claim about money that does not exist yet. Between now and the date a
future bill would be raised, somebody can cancel or delete the booking. A confirmed booking is one
where a human has taken an extra step, which is what makes it reliable enough to forecast against. An
unconfirmed one is not, even if the biller will raise a bill for it anyway.

Applies to both halves of `projectedDueTotal` and to `getUpcomingDues`, and to nothing else on Dues
(see D5).

This review had argued for counting unconfirmed bookings too, because the biller charges them and on
production 1,895 bookings sat unapproved against 28 carrying an explicit approval row. The owner's
standing rule is that current data grounds a decision and never makes it, and he held the ruling. The
biller's behaviour became S1 in the register.

**Re-measured 24 August 2026, and the two counts read differently for a reason.** F67 reports 2,880
confirmed against 1,896 unapproved on the same bookings, which looks like it contradicts the 28
above. It does not: 28, now 112, is the count with an **explicit** approval row, and 2,838 more sit on
properties that accept automatically, which this ruling defines as confirmed. Counting only explicit
approvals makes confirmed bookings look vanishingly rare; counting auto-accept as the ruling requires
makes them the majority. D20 leans on the second reading.

### D2. Current FY means 1 April to today, and the chip stays (2026-08-23)

Owner: "A works."

Applies to the tile (F1), to the View all screen (F25), and to the Current FY filter option on Dues
and Collection (F9). The chip is right only once F5 is also fixed.

**Why.** The owner first leaned the other way: some properties generate future dues early, and money
genuinely owed inside this financial year seemed to belong in a tile named after it. Three arguments
decided it, kept because they apply to any future period tile on any screen:

1. **The money was never excluded.** A future dated bill inside this financial year already appears in
   All Future Dues and in All Time Dues on the same strip. Counting it here would count the same rupee
   a third time, in three of six tiles.
2. **A full-year tile measures a billing setting, not the business.** Two identical properties, one
   generating bills three months ahead and one monthly, would show very different figures with
   nothing about the business differing. One property would also grow through the year mostly because
   generation catches up.
3. **A full-year tile breaks the chip.** It could only be compared with last year's full year. This
   year is not finished, so that compares an unfinished period with a finished one, which sheet §4
   and the tracker both already rule against.

**Deferred, not dropped.** The accounting view, how much of this financial year's book is unpaid, is a
real need. It is a different tile with a different chip, to be specified properly if wanted.

### D3. The Dues (Live) captions were a sheet error (2026-08-23)

Owner: "A, fix the sheet."

First case in this review where the build was right and the sheet was not. Precedent: a worked example
that contradicts the table above it is a specification defect, and the sheet gets corrected.

### D4. Received on Bills Summary means collected against these bills, whenever it arrived (2026-08-23)

Owner: "A."

**Why.** Sheet §8 states the card's purpose one line below the wording it corrected: "how much of what
came due has actually been collected". Window the payment as well and that purpose breaks for every
past window: July's bills were mostly paid in August, so July would read as badly collected when it
was in fact collected.

**The word collision.** "Received" on the Collection screen means money that arrived in the period.
The tracker's rule is one word, one meaning. Resolved by naming it rather than by changing the
number: §3 now states both meanings, says the two will not match for the same window, and that this
is intended. The card carries an info icon saying so.

Second case where the build was right and the sheet was not. Both times the sheet's own statement of
purpose agreed with the build.

### D5. A raised bill is owed whatever the booking's approval state (2026-08-23)

Owner: "In this particular dues breakdown bookings bar, we include both the confirmed and unconfirmed
bookings dues because those are real. We only do not include unconfirmed bookings when we are talking
about projections."

**The rule, generalised.** A bill that exists is owed, and the confirmed-only test belongs to
forecasts alone. A bill list records what happened; a forecast claims what will, so only a forecast
needs the extra assurance.

Consequences: plain status 1 or 2 is correct throughout Dues. Sheet §3 and §9 no longer say "confirmed
booking". Occupancy is not a money question and is not settled by this.

### D6. The money screens show the two eviction states as separate bars; Tenants keeps Under notice (2026-08-23)

Owner, across two exchanges. Three rulings in one.

**On tenants whose approved leaving date has passed.** They have not physically moved out, so they
are still living there, still being billed for rent, food, electricity and everything else. On a
money screen they belong with the approved evictions. No date bound on Dues or Collection. Only
Inventory bounds the date, because only Inventory asks when a bed frees up. This review had
recommended excluding them; the owner ruled otherwise.

**On vocabulary.** Owner: "For us, voluntary eviction and forced eviction are the same thing. We
don't care. Eviction is just eviction... that has no negative connotation in our terms. That
distinction gets fetched from who raised the notice." So eviction is the product's own neutral word,
and Dues and Collection adopt the two labels Tenants already displays. Nothing on Tenants moves.

**On structure.** Dues and Collection go from two active bars to three.

| Screen | Tenant status bars |
|---|---|
| Dues, Collection | Active, no eviction · Eviction pending · Eviction approved · Bookings · Old tenants |
| Tenants | Unchanged. Under notice stays the parent group inside Active |
| Inventory | Still open. Its count is approved and date-bound, so neither label fits as written. Ruled at DA-08 |

**Why three and not two.** Two bars would leave 1,183 tenants who have raised a notice under a label
saying they have none, and they are precisely the people Tenants already shows as under notice.

**On the first bar's label.** It does not read plain "Active". On Dues each due sits in exactly one
bar, so the first bar leaves out the two eviction bars. On Tenants, Active includes them. The label
has to say which one it is. "Active, no eviction" is the weakest word in this ruling; a better one is
welcome.

**For Vivek:** a new bar on two services. The test for a pending notice is already written at
`tenantService.ts:200`. The Tenant Status tab grows from four bars to five; chart height and label
truncation need a look.

### D7. The Overdue Breakup dropdown is removed (2026-08-23)

Owner: "A."

The card already says "as of today" on its face, which is the right one of the three ways a block may
declare its time (D9). A dropdown was the wrong one, not a missing one. Reusable rule: a filter with a
single option is a defect.

### D8. Upcoming Dues becomes the breakdown of the Projected Due tile (2026-08-23)

Owner: "A."

The card gains the other configured due types rather than being renamed Upcoming Rent. The tile's
total then equals the sum of the card's bars, so an owner who doubts the forecast can check it against
itself. The package data already reaches the tile.

**For Vivek:** `getUpcomingDues` and the rent half of `projectedDueTotal` are two copies of one query.
D1, F2 and F3 each land twice or one copy drifts. Sheet §11 no longer says "not built".

### D9. The forward window, and every block declares its own time on its face (2026-08-24)

Owner, across three exchanges. Replaces the Coming up column in every sheet's §4.

**The rule underneath everything.** A block that follows the top filter needs to say nothing. A block
that does not follow it says so on its face, in one of three ways the product already uses:

1. A word in the title or in brackets after it: Live, as of today, Forecast
2. Its own dropdown
3. A plain static line naming the period it follows, with no dropdown

**What each kind of number does.** Time-scoped numbers come in two kinds: those that count records
carrying a date (a bill due in September, an agreement expiring in September) and those that count
things that happened (a payment received, a notice raised, a move-in). Once you know which kind a
number is, its rule for a future window follows.

| Kind | Window in the past | Window ending after today |
|---|---|---|
| Live | Ignores the filter, says so on its face | Unchanged |
| Time-scoped, records by their date | Counts records in the window | Counts them, no chip |
| Time-scoped, things that happened | Counts them in the window | Dash, with a reason |
| Forecast | Stays on its own forward window, unchanged | Extends to the chosen date |

**A dash carries a reason.** One short line, "not available for a future period", or the operator
reads it as broken.

**The chip rule, as three cases.** The third is the second taken to its limit: "same elapsed days"
works for a period that has started, and a period that has not started has no elapsed days to
compare.

| The window | The chip |
|---|---|
| Finished and in the past, like Last Month | Compare the full previous period |
| Started but not finished, like This Month or Current FY | Same elapsed days, marked unfinished |
| Ends after today | No chip |

**Block rulings inside the framework:**

- **Bills Summary runs on a forward window.** This review had it sitting out; the owner ruled it stays,
  and D4 is why: Received means money collected against those bills whenever it arrived, and advance
  payment against a future month is real. The info icon says the gap means "not yet collected, and
  not yet due" on a forward window.
- **Upcoming Dues extends to the chosen date.** Costs more than it looks: the query works out one rent
  date per tenant for one month only, so a window of several months is a rewrite, not a setting
  change.
- **Forecasts stay on their own forward window when the window is in the past** rather than sitting
  out, because they already declare themselves on their face, exactly as Live cards do.

**Not yet ruled:** a window that straddles today, 1 August to 30 September. The "things that happened"
half has real data for August. The table above would dash it.

### D10. Deposits held for departed tenants are counted, on their own line (2026-08-24)

Owner: "A - population is real."

The card counts deposit still held for tenants who have moved out, and the owner confirmed from his
knowledge of the business that the people in that count are genuine unreturned deposits, not a gap in
how refunds were recorded.

**The shape follows from §3.** Old-tenant money is never blended into a total, so it cannot fold into
Received. It is a separate line, "Held for tenants who have left", beneath the gauge and outside it.
Sheet §12 and §3 now say so.

**A caution to carry into the launch.** 133,886 of those invoices have no settlement of any kind
recorded. The line will be large on many properties, and some operators will say "I returned that in
cash". That is a conversation about recording practice, not a defect in the number, but it will
arrive as a complaint about the number. S4 in the register proposes the workflow that would make the
line useful rather than alarming.

**Does not unblock the card.** F21 stands: two money columns, one total.

### D11. The Due Date window on a running period ends at today (2026-08-24)

Owner: "A."

Applies to This Month and Current FY in Due Date view on Collection, and inherits F9's fix there:
Current FY in that view reads 1 April through today. The paid window already capped at today, so
after this ruling both views share one cap.

**Why.** Three reasons, the third raised by the owner's own probe:

1. Every chip on the view repairs itself: Billed, Collected & Adjusted and Still Unpaid compare the
   same elapsed days of the previous period automatically, which D9's chip table already requires.
2. Still Unpaid stays money someone actually owes right now. On a full-month window it mixed overdue
   with not-yet-due, and a mid-month reader would chase money nobody owes yet.
3. A full-month Billed measures a property's billing setup, not its performance: a property raising
   the whole month on the 1st and one raising per tenant through the month would read very
   differently with nothing real between them. D2 cut future-dated bills from Current FY Dues on
   exactly this argument.

**The probe, kept because it will come up again.** Bills already raised with due dates later in the
month do not vanish: they enter Billed when their date arrives, they are one Custom window away
today (the D9 forward view counts them immediately), and money already received against them shows
today in Paid Date view as Paid Early.

**For Vivek:** cap the due window at today in `computeWindows`, both views one rule. F33 is a
separate defect on the same chips and is not settled by this.

### D12. Advance stays outside Total Collection (2026-08-24)

Owner: "A."

Total Collection counts money applied to bills, matching the homescreen and the collections list.
Advance is its own tile beside it, and the sheet's "middle four add up" line becomes three.

**Why.** One answer across surfaces is the point of this build (sheet §1), §17.2 makes it a test,
and the build already passes it. The Advance tile still shows the money by name next to the total,
so nothing is hidden. Advance was 0.008% of July's money, but the ruling is about the principle:
where the sheet's own two rules collide, the cross-surface one wins. The same reading applies to the
Current FY tile, which also counts money against bills.

No code change. Sheet §5 corrected: the three source tiles add up to Total Collection; Advance sits
beside them, counted separately.

### D13. Collection is counted at the bill amount; charges live on the settlement side (2026-08-24)

Owner: "A."

A payment against a bill counts at the bill's amount, gross of processing charges, everywhere on
Dues, Collection and the homescreen. The Payment Settlement card stays net of charges, and the gap
between Total Collection's online part and Collected via RentOk is the charge, visible rather than
smeared into every number.

**Why.** A fully paid ₹10,000 bill must read ₹10,000 collected or Billed minus Collected stops
equalling what is unpaid, which §17.11 tests. §17.2 requires agreement with the homescreen, which is
gross. The charge is a cost of collecting, not uncollected rent.

**One deliberate exception.** The Advance tile counts the payment net of charges: an advance has no
bill, so net is the only amount it has, and net is what the owner actually holds.

No code change. Sheet §3's base rule and test §17.7 corrected.

### D14. Settlement Pending carries no chip (2026-08-24)

Owner: "A."

**Why.** The tile measures where money stands right now. A past window's pending figure, measured
today, has mostly settled since, so any comparison always flatters the present: a chip that trends
good regardless is worse than no chip. A fair chip would need the figure as it stood at that
moment, which nothing records. This is the reasoning F5 established and D9's no-chip cases already
carry; the ruling writes it onto this tile so the next reviewer does not re-raise it.

No code change. Sheet §4's chip table corrected, with the reason on the row.

### D15. Collection by Property follows the toggle (2026-08-24)

Owner: "A."

In Paid Date view the card ranks properties by money arrived; in Due Date view by collected and
adjusted against the window's bills. In each view the rows sum to the total above them, which
§17.1 tests.

**Why.** A ranking that reconciles with the screen it sits on beats one frozen while everything
around it switches. The sheet's own forward-window cell already described the Due Date reading, so
"the same in both views" was the sheet disagreeing with itself, the same shape as F11 on Dues.

No code change. Sheet §4's card-change table and §10 corrected.

### D16. No chip where there is nothing to compare against (2026-08-24)

Owner: "A."

§4 had asked for a rupee change with the percentage dropped where the previous period had no spend.
The build sends no chip. Ruled as built, on all five tabs alike, and §4 corrected.

**Why.** A rupee chip is a third state the shared chip component does not have, and it already lacks
the second one, the neutral grey the count tile needs (§17.26). More importantly, "no chip" already
means "nothing to compare against" on the four other tabs, so building this would give one blank
situation two different looks across the suite. The tile shows its number alone, the same state it
already uses on All Time.

First Expense case where the build was right and the sheet was not.

### D17. Still owed to staff counts everything staff fronted, refunds included (2026-08-24)

Owner: "A."

The tile counts money a staff member paid from their own pocket whatever they paid it for, so a
tenant refund they fronted counts alongside an expense they fronted. ₹2.32 crore of today's ₹50.77
crore. Sheet §15 items 15 and 16 corrected.

**Why.** Two reasons, and the second is the one that would have broken.

1. The tile is a debt to a person, not a spend figure. §5 already says it never adds into any total
   on the screen, so §3's rule that a refund is not spend is not in play here: nothing about this
   tile is spend.
2. It has to keep matching the screen it hands off to. The tile is the Passbook's own Personal funds
   balance with the sign flipped, and the Passbook counts fronted refunds. Excluding them here would
   make the tile disagree with the Passbook for the same staff member, and the manager finds that out
   by tapping the tile.

**The general rule, for the next tile like it.** Where a number's whole job is to hand off to another
screen, that screen's definition wins over the local one. The same reading settled D12 on Collection,
where matching the homescreen beat the sheet's own adding-up rule.

### D18. The two meanings of "Others" stay for now (2026-08-24)

Owner: "I am kind of okay with having everything else named as Others only, and upon clicking that,
Other also comes as a line item. I think I am still okay. That is an anomaly we don't want to solve
for. However, note it down for later on that something should happen as per the sheet. Maybe we
should rename, as you suggest, everything else, but for now I am okay with it."

The rollup row keeps the name "Others". A category somebody typed as Other or Others keeps its own
name too, so on 111 properties the card carries the word twice and on 362 the typed one appears
inside the Others sheet. Sheet §7 corrected to describe what happens rather than to forbid it.

**Why.** §14's promise is the stronger one: the screen never renames what somebody typed. Given that,
the only repair left is to rename the rollup, and that is a change to a word Dues and Collection also
use for their own rollups, so it is a suite decision rather than an Expense one. The owner ruled it
is not worth holding this build for.

**Not dropped, parked with its trigger.** S5 in the register carries the rename. It should be decided
alongside the same question on Dues and Collection, where a due type typed as "Others" would produce
the identical collision and nobody has looked yet.

### D19. FlexiPe spending is its own payer, named "FlexiPe (Owner)" (2026-08-24)

Owner: "Show it as flexi pay, and in brackets mention owner. That makes the most sense to me."

The Paid by tab shows a row named **FlexiPe (Owner)** for every expense paid through the wallet. It
never merges into the plain Owner bar, and the raw `(Owner)` string never reaches the screen.

**Why.** Two readings were on the table and both were worse. Merging it into Owner tells a manager
the owner personally paid ₹60.2 crore when the wallet did. Leaving `(Owner)` as it stands puts a
bracketed internal string on the largest bar of the card, and on 123 properties splits the owner into
two bars that do not add up. Naming the wallet first and the person second says both true things in
one label.

**It is the third place FlexiPe is named the same way**, after its own row on the Payment Mode tab and
its own fund row in the View all sheet, so the tab is consistent with the rest of the screen rather
than inventing a fourth treatment.

**For Vivek:** the fix belongs in the analytics label, not in the FlexiPe writer. Changing what the
writer stores would rewrite history on 20,301 records and every other screen reading that field.

### D20. Only a confirmed booking counts as booked, and the other two screens are fixed with it (2026-08-24)

Owner: "As you recommend, however, I want the home screen and the room list widgets to be fixed and in
sync as well."

**What it settles.** This is the question F4 was left open for and DA-08 was chosen to answer. A bed
with an unapproved booking on it is **vacant and not booked**. The booked layer counts confirmed
bookings only, in both views. The forward projection, if the Custom path revives it, counts confirmed
arrivals only, which is D1's rule already.

**Why.** A booking is a promise; approval is the step that makes the promise worth planning against.
§3 said so from the start, and D1 settled the same question for money on the same reasoning. Counting
an unapproved booking as filled space lets a property report itself full on arrivals that may never
happen, and 40% of live bookings are unapproved right now, so this is not a rounding case.

**The instruction that goes wider than this tab.** This review had proposed that analytics compute its
own counts and leave the homescreen widget alone for now. The owner ruled against the fork: the
homescreen widget and the rooms list are corrected too, so all three surfaces answer the same. That is
the standing cross-surface rule, §17.2 on Collection and D17 on Expense, applied to space instead of
money.

**For Vivek, three separate places:**

| Where | What changes |
|---|---|
| `helpers.ts:1929`, `:1945` | `has_new_booking` counts a confirmed booking only, not any status-2 tenant |
| `helpers.ts:2023` to `:2025` | `active_count` drives vacant, occupied and semi-vacant off living tenants; an unapproved booking must not make a room read Occupied |
| Rooms list filters | The same word means the same thing, which §14 records as a live disagreement in both directions |

The test for confirmed is already written at `tenantService.ts:352`, including the auto-accept case
D1 ruled equal to confirmed. Reuse it rather than write a fourth one.

**One consequence to expect.** Occupied and Vacant will both move on the homescreen the day this
ships, on the 3.4% of properties the sheet's appendix found carrying any confirmed booking and on
however many carry unconfirmed ones. It is a correction, and it will read as a change.

### D21. Inventory's tile is "Under eviction", covering both states, with the parts always named (2026-08-24)

Owner: "Let's call it under eviction, which includes both the approved eviction and pending eviction."

**What it settles.** The tile F14 left as the sixth untraced meaning of under notice. It is renamed
**Under eviction** and its number takes in both states: 6,283 today, being 5,090 approved and 1,193
pending.

**Why.** The 1,193 pending were invisible on this whole screen. A manager reading Inventory is asking
how much space is at risk, and a raised notice is the earliest signal of that; D6 already gave those
people a bar on the money screens, so the space screen was the one place they did not appear.

**The condition this ruling carries.** An approved eviction has a date and a pending one does not, so
a single merged number can say space is at risk but cannot say when it frees. **The second line always
names the two parts, never only above zero:** "6,283 under eviction · 5,090 approved · 1,193 pending".
With that the headline is a risk number and the parts are the planning number.

**Two consequences accepted deliberately:**

1. **Upcoming Vacancy stays approved-only and date-bound**, because it buckets by day and a pending
   notice has no day. So the tile reads 6,283 while that card totals 4,772. The gap is 1,193 with no
   date yet plus 318 whose date has passed, and both cards say so rather than look broken.
2. **This effectively decides S3.** With Inventory saying Under eviction, Tenants becomes the only
   screen still calling the same idea Under notice. The suite-wide sweep should follow at the Tenants
   review rather than be left half done.

### D22. In Unit view the tile counts units, not people (2026-08-24)

Owner: "A."

Bed view counts people, which is one bed each. Unit view counts units holding at least one person
under eviction, which is what §5 asked for and smaller than the headcount wherever two people share a
room.

**Why.** Every other tile in that row changes grain with the toggle. A tile that keeps one grain under
two labels is the one that gets checked by hand, found wrong, and takes the other four down with it.

### D23. Upcoming Vacancy follows the view toggle (2026-08-24)

Ruled as built.

§4 and §9 both say this card ignores the toggle because "their subject has no bed-level version". That
reason does not hold: a departing tenant is a bed freeing up, which is as bed-level as anything on the
screen. The build counts departing tenants in Bed view and distinct room-and-day vacancy events in
Unit view, and switches its own axis label with them.

**Why the build wins.** The sheet's own §9 pairs this card with Vacant Room Status as two lenses on the
same day-buckets, and Vacant Room Status genuinely has no bed version, because an empty room's waiting
time belongs to the room. This one does. The build defended §4's "the toggle changes what gets counted"
against §4's own exemption list, which is the pattern this review keeps finding.

Sheet §4 and §9 corrected. Vacant Room Status and Agreements ending soon stay exempt as written.

### D24. The eleven-month assumption stays, and the number it rests on becomes the way to fix it (2026-08-24)

Owner: "Keep the assumption, but mark it... we'll mark the info sheet with a drill-down ability so
that users can filter the list, get to a filtered list, and then also fill the data one by one if they
want to. That way, coverage also gets 100% in reality."

**What it settles.** Every long-term tenant stays on the card. Where no agreement length is recorded
anywhere, eleven months is used, and that is stated rather than hidden. Three parts:

1. **The card keeps its bars.** No tenant is dropped for missing paperwork.
2. **The info sheet names the assumption and carries the count**: how many of these tenants are on the
   eleven-month fallback rather than a recorded length.
3. **That count opens the list.** Tapping it lands on the tenants list filtered to exactly those
   people, where the length can be filled in one at a time.

**Why this beats the alternative, which this review had recommended.** Taking those tenants off the
card makes every remaining bar a fact and leaves the manager with a thinner card and nothing to do
about it. This ruling keeps the card whole, tells the truth about which half is assumed, and puts the
repair one tap from the number that reports the problem. The count then falls as people use it, so the
caveat is temporary by design rather than permanent by disclosure.

**It is the homescreen's own pattern**, which is why it needs no new invention: an info sheet that
explains a number and drills into the records behind it already exists there.

**Eleven months is a reasonable fallback**, and the owner confirmed it. The defect was never the
number, it was that nothing said it was a fallback.

**For Vivek:** the assumption and its count go into `occupancyHints.ts`, where this tab's other 23
definitions already live, and the count needs a drill target: the tenants list filtered to long-term,
living, with no recorded agreement length. The same four-step end date runs on the Tenant tab's
Agreement Expiry card, so the same treatment belongs there.

**What this does to S7.** It does not replace it. Filling the back catalogue one tenant at a time is
the repair; asking for the length where the agreement is created is what stops the gap reopening. S7
keeps items 1 and 2 and loses its urgency.

### D25. Line numbers are pinned to the reviewed commit, not chased (2026-08-25)

Owner: "A."

**What it settles.** Every `file.ts:NNN` in this log resolves against **`3a13e08ac`**, the state of
the code each tab was reviewed against. They are not re-mapped as the code moves. Each finding names
the function or block as well, and that is what to trust when a number and the code disagree.

**Why this came up.** A day after the Occupancy commit, `950d46d1e`, "fix: improved comments",
rewrote comments across six analytics service files. It changed no behaviour, so every finding
survived untouched, and it moved **79 of this log's 98 citations** off the lines they name.

**Why pin rather than chase.** Two edits in one week moved these files. Re-mapping produces a doc
that is right on the day it is written and wrong again by the next commit, with no signal that it has
gone wrong. Worse, re-mapping by hand is the exact mistake this review has already made twice, once
with a multi-file `awk` using `NR` (F47) and once carrying a guide's line number onto a code file
(F71). A named baseline cannot rot: it either matches or it visibly does not.

**What this asks of Vivek.** Nothing extra. He works from the open-items tables, which name the file
and the function; the line number is a convenience for finding it faster, not the finding itself.

### D26. Fallback tenants leave the dated bands and become their own stated group (2026-08-25)

Owner: "B." Of the two put to him: A, keep every tenant on the dated bands with the eleven-month
fallback named and counted; B, take the fallback off this card and give those tenants their own
stated group.

**What it settles.** On the Tenant tab's Agreement Expiry card, a tenant whose agreement length is
recorded nowhere is never placed on a dated band. The five bands, Already expired through Valid, are
computed from recorded and derived terms only, so each is a fact about paperwork. The fallback
tenants become one stated group, **No term recorded**, carrying D24's treatment: the count on its
face, and the count opens the tenants list filtered to exactly those people so the lengths can be
filled in. Renewal overdue and Renewal Due count real terms only, so they stay equal to their bands
as §15, Renewal & Retention, requires. Journey's Renewals in 30 days moves with them: one number, computed once, shown
three times.

**Why.** The bar claims paperwork ran out. For 123,287 of the 167,001 tenants it draws, 74% of the
bar measured on 25 August 2026, the product holds no paperwork to claim it from, and a red bar that
is three quarters false alarms teaches a manager to ignore the bar. The group keeps every tenant
visible, keeps D24's repair path of a count that opens the list, and makes every dated bar
trustworthy. It also shrinks as lengths are filled, so the caveat is temporary by design, which was
D24's own argument.

**Two shares get quoted for this gap, and they measure different things.** 74% is the share of the
Already expired bar resting on an invented date. 54% is the share of all long-term living tenants
with no recorded length, 187,385 of 346,930, which is the figure §12 of the sheet carries and the
one S7 in the register uses. Neither is wrong; they have different bases.

**The boundary.** D24 stands unchanged on Inventory: its card looks forward 90 days, an invented
date mostly keeps a tenant off it, and its info sheet carries the count and drill as ruled. Only
this tab's card gains the group, because only here does the fallback paint the biggest, red bar.
The eleven-month step therefore stops being used anywhere on the Tenant tab, and stays on Inventory.

**For Vivek:** exclude the no-term tenants from the five bands and return them as one group with its
count; the count drills to the tenants list filtered to long-term, living, no recorded agreement
length, the same target D24 already asks for. The left-early figure stops reading `ENDS_ON` under
F97, which moves the left-early figure onto the lock-in date, so after both fixes the Tenant tab
computes no eleven-month date at all.

### D27. Renting Type hides until the field is captured (2026-08-25)

Owner: "A." Of the three put to him: A, hide the block until the field is recorded; B, keep the
interim stay-split donut under the Renting Type name; C, show the card with a not-recorded state.

**What it settles.** The Renting Type block does not render while renting type is unrecorded, which
today is everywhere. The interim short against long term donut is removed rather than renamed: it
duplicated the Stay Type card directly above it, and its name told a manager a lettings fact the
data does not hold. The block returns `hidden`, the same mechanism Property Wise uses on one
property, and starts rendering the real B2B against Residential split when the field starts being
captured.

**Why.** §16 wrote the rule for exactly this: "a missing value is not an answer." A card that
restates its neighbour under a different name is worse than an absent card, because the name is the
only thing it adds and the name is wrong. Hiding costs nothing a manager can use today and removes
the one way this screen could actively mislead about who a property lets to.

**For Vivek:** `getRentingType` returns `{ hidden: true }` until the renting-type field carries
data; the real split, and where that field gets captured, stay as the implementation doc's open
item. Whether a capture flow for renting type is worth building is a product question for another
day, the same shape as S7 in the register, where the product never asks for the fact at move-in.

### D28. The parent group is named "Under eviction" suite-wide (2026-08-25)

Owner: "A." Of the two put to him: A, rename the parent group to Under eviction everywhere; B, leave
Tenants saying Under notice. It was his own suggestion from 23 August, parked as S3 until all five
tabs were verified.

**What it settles.** The group of living tenants who have said they are leaving is called **Under
eviction** on every screen, always shown with its two parts named, **Eviction pending** and
**Eviction approved**. Tenants was the last screen saying "Under notice" for it; it adopts the suite
word. Eviction is the product's own neutral word, which D6 established when it gave the money screens
their two eviction bars, and the parts already carried it on this very screen's tiles, so the whole
and its parts finally share a first word.

**What does not change.** Nobody moves between groups; the rename changes words, not numbers. Each
screen keeps its own rules as already ruled:
Dues and Collection count a passed approved date inside Eviction approved, as D6 ruled for the money
screens; Inventory's tile includes both parts and the passed dates, as D21 ruled for the space
screen; and Tenants keeps **Past their date** as a separate sibling layer, never folded in. The rename is words, not numbers.

**Where the word changes, shipped copy.** Five visible strings on the Tenant screen: the View all
sheet's Under notice row, the rent at risk note "N under notice", the two Journey empty messages, and
the Journey hint text. The Journey bar label is a sixth string carrying the old word, and it is
deliberately excluded: D29 rebuilt that card and renames the bar Eviction pending, not Under
eviction, so the suite word lands on the other five only. The tenant
list's own filter names are the list's business and follow separately. Counted on 25 August 2026:
52 lines of analytics code carry the old phrase in some form, 76 occurrences counting every spelling,
most of them in comments or already owed under D6, the money screens' two eviction bars, and D21,
Inventory's tile; the five strings above are the
manager-visible remainder.

**Where the word changes, sheets.** Swept in this pass: DA-09's sixteen mentions, sixteen by sweep
day where S3 counted fifteen the day before, DA-08's two forward-looking ones, and DA-01's one. Historical narrative that records what things used to be called
stays as written, or the log stops being readable.

### D29. Journey's Tenants tab carries six rows in two named groups (2026-08-25)

Owner, naming the rows he wanted: "Active tenants, No notices, Eviction pending, Approved eviction,
Renewals in 30 days, Agreements expired. That would make the journey whole and more correct."

**What it settles.** The card goes from three bars to six rows, and the six sit in two named groups
because they answer two different questions:

| Group | Rows | How they relate |
|---|---|---|
| Who is staying, who is leaving | Active tenants · No notices · Eviction pending · Approved eviction | The last three account for everybody except tenants past their date, so they add up to Active on any property where nobody is past their date, and fall short by that count where somebody is |
| Whose paperwork needs a decision | Renewals in 30 days · Agreements expired | Overlap the first group, and exclude each other: an agreement has either ended or not. Parallel bars only, never stacked |

Agreements expired is the same number as Renewal overdue on Renewal & Retention and the Already
expired bar on Agreement Expiry: one computation, shown three times, which build guidance 5, compute
each number once, already requires.

**Why groups rather than six siblings.** A tenant under eviction can also have an expired agreement,
so those rows overlap. Six equal bars would tell a manager they are comparable slices of one thing.
Naming the groups costs a line and stops the misreading. The old rule that the three bars "overlap
each other, never slices of a whole" was written when every bar overlapped. The first group's rows no
longer overlap, so that rule now belongs to the second group alone.

**The repetition was raised by the owner and is deliberate.** Three of the six rows are also Overview
tiles. A tile answers "how many"; a row here answers "what share of my house", and this is the only
card that splits Active tenants by the eviction states the tile strip already shows, and its rows add
up where the tiles never do, because §5 forbids the tile strip from suggesting its tiles add up to
anything. Other cards split Active tenants too, Stay Type and Verification among them; none of them
splits it this way.
The precedent is the owner's own homescreen, where the tile row and the journey never restate each
other because the journey there is a pipeline rather than a second scoreboard. The Bookings tab of
this same card is already a funnel; this makes the Tenants tab its tenant-side equal.

**A systems note the owner asked for.** The overlap is a signal about the tile strip, not about this
card. Seven tiles is already an open design problem, §24 item 3, tile row overflow. If Journey owns
the eviction split with its shares, the strip has a real case for getting shorter. Not decided here.

**What is deliberately not in it.** Tenants whose leaving date has passed with the record still open.
They belong to the split arithmetically, but §5 keeps Past their date on its own tile and nowhere
else, and the owner ruled the addition is a proposal rather than a requirement. It is S9 in the
register, which measures the gap: 318 tenants on 151 properties.

**For Vivek:** every piece already exists in the service, and two of them move into the Journey
query. The bar now called Under Notice becomes the pending count under its own name and approved
eviction is unchanged, both already there. No notices is Active minus the three eviction states, so
the past-their-date count comes across from `liveTiles`. Agreements expired is the `renewal_overdue`
expression `getRenewalRetention` already runs. Nothing new needs measuring; it needs arranging.

### D30. A bucket outside the run of days hides at zero (2026-08-25)

Owner: "the upcoming eviction overdue bar should be hidden if zero in any period."

**A suite rule, not one bar.** On any card whose bars are a run of
time, a bucket sitting **outside** that run hides when it is zero; a bucket **inside** the run always
shows, even at zero.

| Card | Hides at zero | Always shows |
|---|---|---|
| Upcoming Eviction, Tenant | Overdue | 0–7 · 8–15 · 16–30 · 31+ |
| Agreement Expiry, Tenant | Already expired · No term recorded | 0–30 · 31–60 · 61–90 · Valid (90+) |
| Vacant Room Status, Inventory | Never rented · Unknown | 0–7 · 8–15 · 16–30 · 31+ |

**Why the two behave differently.** A run of days is read as a shape, so a missing band in the middle
of it reads as a broken chart and the reader stops trusting the axis. A bucket outside the run has no
neighbours holding a place for it; at zero it is a row saying nothing, and on a healthy property it
is the only row saying nothing. The rule generalises without needing a list: **if removing the bar
leaves a gap in a sequence, keep it; if it leaves nothing, drop it.**

**Where it reaches beyond the owner's example.** He named one bar. The same test finds four more: two
on Agreement Expiry, one of which is the group D26 created hours earlier, and two on Inventory's
Vacant Room Status. The Inventory pair is not all new ground: DA-08 already says the Unknown bar
hides at zero, and the build renders it always, so that half is a build gap the Occupancy pass
missed rather than a rule made here. Never rented is genuinely new. All are in the open items.

**Not affected:** a card where every bar is zero still shows its healthy or empty state as §21, the
empty and healthy states, words it, rather than a card with no bars at all.

### D31. Every Agreement Expiry label carries its own days (2026-08-25)

Owner: "All agreement expiry status labels should have 'valid' in brackets, beyond 90 days or
something. I am not sure about the copy... Whatever you decide on this."

**What it settles.** The bars read **Already expired · 0–30 days · 31–60 days · 61–90 days ·
Valid (90+ days)**, plus No term recorded from D26.

**Why, and the owner's instinct was pointing at something bigger than the one word.** "Valid" alone
invites the reading that every other bar is invalid. That is false: an agreement ending in twenty
days is perfectly valid today. Putting the days in brackets turns the word from a verdict on the
other bars into a statement about this one, how long it still runs. Once that is fixed the same
problem shows on the three bars beside it: "60 days" never said whether it meant within sixty days or
between thirty-one and sixty, and the only way to know was to read the bar next to it. Each label now
carries its own range, so no bar has to be explained by its neighbour.

**The range style is not invented here.** Upcoming Eviction, directly above on the same screen,
already labels its bands 0–7, 8–15, 16–30, 31+. This makes the two cards read the same way, which is
what §11's "bucket edges match Inventory exactly" was already reaching for. Agreement Expiry's own
edges are not Inventory's and were never meant to be; only the style of the label is shared.

### D32. Stay type always shows; renting type shows only when it has somebody in it (2026-08-25)

Owner: "the first split should be short term and long term. It should always be shown, even if short
term is zero or long term is zero. In the second split, which is basically the type of B2B or
residential, it should only come when both or any one is greater than zero."

**What it settles.** Two splits, two opposite showing rules:

| Split | Shows |
|---|---|
| Short term · Long term | Always, including when one side is zero |
| B2B · Residential | Only when either side is above zero |

**Why they differ, and this is the general rule.** Every tenant is either short term or long term.
Measured on production on 25 August 2026, live properties, test and deleted excluded: the flag is set
on all 352,886 living tenants, none unset, and the code counts an unset flag as long term anyway, so
the two sides cover everybody either way. That makes a zero on one side a real answer worth reading:
"none of my tenants are short term" tells a manager something, and it is the common case, since short
term is 1,645 tenants platform wide, under half a percent. Renting type is a field often not filled
in at all, so a zero there is not an answer, it is an absence, and §16's own line already said a
missing value is not an answer. **The test is whether a zero means "none" or "not recorded".** A zero
meaning none is shown; a zero meaning not recorded is hidden.

**This sharpens D27 rather than replacing it.** D27 said Renting Type hides "until the field is
captured", which is a state of the world nobody can test in code. This gives it a testable form: show
when either side is above zero. Today that hides the block across production, the same outcome D27
described, but now it turns itself back on property by property as data arrives rather than waiting
for somebody to declare the field captured.

All in DA-01 Dues, 2026-08-23 and 2026-08-24:

| Section | Change | From |
|---|---|---|
| §3 base rule | Bookings count whether or not approved; Projected Due named as the one exception | D5, D1 |
| §3 base rule | Refunded and written off dues excluded from every number | F10 |
| §3 old tenants | Second home added: the held-deposit line under Deposit Dues | D10 |
| §3 "Under notice" | Replaced by "Eviction: raised, then approved", two states, no date bound, eviction defined as neutral | D6 |
| §3 Received | Both meanings stated; the two screens will not match for the same window, by intent | D4 |
| §3 Active, not under notice | Renamed Active, no eviction | D6 |
| §4 options | Coming up removed | D9 |
| §4 forward rule | "Custom stops at today" replaced by Custom may end in the future, with the three things that change | D9 |
| §4 matrix | Last column is now Custom ending after today; every row rewritten from the code | D9 |
| §5 Projected Due | Living tenants and confirmed bookings only, with the reason | D1 |
| §7 Dues (Live) | Week starts tomorrow; Due Later names the last day of the week | D3 |
| §8 Bills Summary | Received ignores when the money arrived, with the reason; info icon named | D4 |
| §9 Dues Breakdown | Three tabs do not share one total, by design | F11 |
| §9 Tenant status | Five bars, labels and meanings | D6 |
| §11 Upcoming Dues | Marked built; other types being added so bars add to the tile | D8 |
| §12 Deposit Dues | Total Deposits defined; held-for-departed line added beneath the gauge | F20, D10 |
| §15 tap matrix | Tenant status rows now the five bars | D6 |
| §15, §17, §19, §21 | Every "Coming up" reference rewritten to the forward Custom window | D9 |
| §21 design fixes | Item 1 says the design's "Under Eviction" bar splits into pending and approved; item 11 updated | D6 |

All in DA-02 Collection, 2026-08-24. First the edits already ruled before this tab's review:

| Section | Change | From |
|---|---|---|
| Note block | Correction note added, naming this review's edits | |
| §1 | Build status: the backend now serves every block from real queries; the scaffold wording removed | This review |
| §3 words | The "Under notice" row replaced by "Eviction": two states, no date bound, the product's neutral word | D6 |
| §4 options | Coming up removed; the forward view is a Due Date Custom window ending after today | D9 |
| §4 forward rules | Chip suppression, dash with a reason on money-arrived numbers, records keep counting | D9 |
| §4 matrix | Last column rewritten as Custom ending after today, row by row | D9 |
| §4 chips | The no-chip case renamed from Coming up to a window ending after today | D9 |
| §6 Status tab | Five bars: Active, no eviction · Eviction pending · Eviction approved · Bookings · Old tenants | D6 |
| §6 | The false line "The Dues screen keeps these tenants inside its Active bar" removed | F14, D6 |
| §13 | Status tap rows are the five bars; the new-filter row renamed Eviction state; the window-travel and forward-option rows rewritten | D6, D9 |
| §15 | The forward empty-state row renamed from Coming up | D9 |
| §17 tests 18 to 20 | Rewritten for the one-sided forward Custom window | D9 |
| §19 items 2 and 15 | The eviction split and the forward treatment | D6, D9 |

Then the edits ruled out of this tab's own findings, and the corrections from the pre-commit review
pass:

| Section | Change | From |
|---|---|---|
| Note block | D11 to D15 named; pointer to where rulings live | Review pass |
| §1 | The five rulings named as done, not waiting | Review pass |
| §3 base rule | Counted at the bill amount; charges live on the settlement side; Advance the one net exception; reshaped one rule per line | D13 |
| §4 windows | A running period ends at today in both views, with where later-dated bills go | D11 |
| §4 chips | Settlement Pending row: no chip, with the reason | D14 |
| §4 card table, §10 | Collection by Property follows the toggle; rows sum to the view's own total | D15 |
| §5 | Three bill tiles add up to Total Collection; Advance beside them, counted separately; Current FY reads money against bills | D12 |
| §11, §18 item 1 | The settlement worry re-measured and inverted; both sections now point at F43 | F43 |
| §13 | The settlement-filter cell shortened; its deeper defect moved to a line under the table | Review pass |
| §17 test 1 | Each card sums to the view's own total | D15 |
| §17 test 7 | Rewritten: bill amount to Total Collection, net to Collected via RentOk | D13 |
| §17 test 16 | Settlement Pending shows no chip | D14 |

All in DA-04 Expense, 2026-08-24:

| Section | Change | From |
|---|---|---|
| §1 build status | "The screen is not built ... every number below has to be built from scratch" replaced: the screen is built, verified, and the sheet is the rule while the log holds what differs | This review |
| Note block, frontmatter | Correction note naming every changed section; date moved to 24 Aug, status to v3.1 | This review |
| §3 words | The team passbook defined, since §6's fund rows lean on it and nothing defined it | This review |
| §4 forward rule | The no-forward reason no longer explains itself by naming Coming up, which D9 removed | D9 |
| §17 item 32 | Closed: the info-icon words ship in the build | B5 |
| §3 three totals | "Always the same number" becomes the same number whenever they cover the same window, with the set-aside card named as the one exception | F48 |
| §4 chips | Where the previous period had no spend, no chip, replacing the rupee-change chip the sheet asked for | D16 |
| §5 tile table | Still owed to staff counts what staff fronted whatever they fronted it for | D17 |
| §5 tile | Launch caution: the tile will read fifty crore and can only grow, with the measured figures and who answers "I paid him back" | F53 |
| §6 fund rows | The remainder row named "Source not recorded"; the adding-back rule confirmed against production | F57 |
| §7 category groups | How the six groups match: start of the name, ignoring capitals, with the thirty deposit spellings as the worked example | B3 |
| §7 Others | The two meanings of Others stated as accepted, with the measured scale and the parked repair | D18 |
| §7 FlexiPe | One in seven by count and a third of the money, the largest row on the tab | F61 |
| §8 rules | FlexiPe spending is its own payer row, "FlexiPe (Owner)", never merged and never shown raw | D19 |
| §10 | The empty state replaces a list of zeros when the window's total is zero | F64 |
| §15 test 2 | Extended from stray spaces to capitals, with the measured scale, and which spelling labels the bar | F59 |
| §15 test 15 | The tile equals the sum of every member's open personal-funds balance in Passbook, to the rupee | D17 |
| §15 test 16 | New: a named group never swallows a category belonging elsewhere, with RentOk Software as the worked case | B3 |
| §15 test 17 | Everything a staff member fronted counts, whatever they fronted it for; was test 16 before test 16 was inserted | D17 |

All in DA-08 Inventory, 2026-08-24:

| Section | Change | From |
|---|---|---|
| Frontmatter, note block | Date to 24 Aug, status to v2.1, correction note listing every changed section and ending "Nothing else moved", with the one unruled item named | This review |
| §1 build status | "The screen is not built ... it is unwritten" replaced: the screen is built and verified, the sheet is the rule, the log holds what differs, and the two things built better than asked are named | This review, B6, B7 |
| §3 Booked | A room holding only unapproved bookings is vacant, in both views; the same rule lands on the homescreen widget and the rooms list | D20 |
| §3 Under notice | Replaced by "Under eviction": two states, both always named beside the total, the two layers kept, eviction defined as the product's neutral word | D21 |
| §3 words to be careful with | The Under eviction, Vacant and Booked rows rewritten; the false claim that "no second variant survives" removed | D20, D21 |
| §4 options | Coming up removed from the filter list | D9 |
| §4 forward rule | New: the forward window is a Custom range ending after today, with what each kind of number does and the no-chip rule | D9 |
| §4 matrix | Last column rewritten as Custom ending after today | D9 |
| §4 capacity | New line: capacity is day-averaged like occupancy, not taken from today | F68, §18.1 |
| §4 face rule | Every card that does not follow the filter says so on its face, three ways | D9, F80 |
| §4 chips | Under Notice row renamed Under eviction | D21 |
| §4 toggle | Upcoming Vacancy moved out of the exempt list, with the reason; two cards stay exempt | D23 |
| §5 tile table | Fifth tile is Under eviction; Unit View counts units, not people, with the worked example | D21, D22 |
| §6 View all rows | Rooms with no capacity get their own row instead of being folded into "Disabled for rent", which said they were switched off on purpose | F72 |
| §8 aging bars | Rewritten: the bars are built from the last move-out on the room's stay history, and what still waits is the drills | B6 |
| §18 item 12 | Rewritten: two things still wait on the emptying date, not three | B6 |
| §19 item 1 | Rewritten from "the largest prerequisite" to what is actually left: Days to Fill, the past-period cost, and the drills | B6 |
| §19 item 2 | The info-icon copy is written on this screen and on Expense; the item stays open only for the rest | B8 |
| §20 design fix 43 | New: whether 0 to 7 days should be marked once the build's red bar is removed | F76 |
| §7 chips | Two chips, not four, with Booked Beds and Under notice with booking named as the two that are not chips | F69 |
| §7 Under eviction chip | Carries its two states always, and its two layers as a second line | D21 |
| §9 Upcoming Vacancy | Follows the view toggle, with the reason and the switching axis label | D23 |
| §9 Upcoming Vacancy | New ⚠: this card and the tile will not match, and why | D21 |
| §10 Agreements | Every tenant stays on the card; the eleven-month fallback is named in the info sheet with its count, and the count opens the list of tenants missing a length | D24 |
| §14 tap matrix | New row: the assumed-length count opens the tenants list filtered to those people | D24 |
| §12 pricing | New ⚠: the group average is the step that matters most, because an empty room has no tenant to read a rent from | F78 |
| §12 basis line | Counted in beds, not rooms, matching the number above it | F79 |
| §12 sorting | Sorted by revenue loss with the reason; the bar and share come from rupees, never from the shortened text | F79 |
| §14 tap matrix | Under eviction rows rewritten, approved and pending each with their own destination | D21 |
| §17 no guesswork | Rewritten: Agreements leaves out tenants whose length was never recorded rather than assuming one | F77 |
| §18 items 6, 7, 16 | Under eviction wording; the forward-layer rule reworded off Coming up | D21, D9 |
| §19 open items | New item 10, agreement length not recorded; two items closed by D20 and D21; the v1 under-notice closure marked as reopened and settled | F77, D20, D21, F14 |
| §20 design fixes 2, 4, 20, 22, 26, 33 | Under eviction wording, the filter chip list, the forward window, and the four-chip note | D9, D21, F69 |
| Appendix | Two Under notice rows renamed | D21 |

All in DA-09 Tenants, 2026-08-25:

| Section | Change | From |
|---|---|---|
| Frontmatter, note block | Date to 25 Aug, status to v2.2, correction note naming every changed section, including §3 and §12 (D26) and §16 (D27), ruled and edited the same day | This review |
| §1 build status | "The screen is not built" replaced: built and verified, the sheet is the rule, the log holds what differs, redirection still suite-wide unbuilt | This review |
| §4 options | Coming up removed from the filter list | D9 |
| §4 forward rule | New: the forward window is a Custom range ending after today, with the per-kind table, the no-chip rule, the dash-with-reason rule, and the projection named as an open design decision with its query already written | D9, F91 |
| §4 matrix | Last column rewritten as Custom ending after today, row by row from D9's kinds | D9 |
| §4 chips, action bar | "Never on Coming up" rewritten to a window ending after today, both places | D9 |
| §11 | The one-number line scoped to the Overdue bar's approved part, with the pending gap named and the old sentence withdrawn | F99 |
| §19 window table | The Coming up row renamed to a window ending after today | D9 |
| §25 design fix 11 | The Coming up chip and picker replaced by the forward Custom window's chipless state and dashes | D9 |
| §3 agreement end date | Three steps, not four; no term recorded is a stated state, never an assumed date | D26 |
| §12 | Bars cover everyone with a term; the No term recorded group beside them with its count and drill, measured at 54% | D26 |
| §19 tap matrix | New row: the No term recorded count opens the tenants list filtered to those missing a length | D26 |
| Measured figures | The end-date sources row now points at the group instead of the assumption line | D26 |
| §16 | Renting Type hides until the field is captured, stated as settled | D27 |
| §8 Journey | Rewritten: six rows in two named groups, the eviction split under Active tenants and the paperwork pair that overlaps it, with why the tile repetition is deliberate and the gap S9 measures, tenants past their date, named | D29 |
| §11 Upcoming Eviction | Overdue hides at zero, the four day bands do not, with the reason | D30 |
| §12 Agreement Expiry | Labels carry their own days, Valid becomes Valid (90+ days), with why "valid" alone misleads; Already expired and No term recorded hide at zero | D31, D30 |
| §16 Stay Type | The two splits get opposite showing rules, with the zero-means-none against zero-means-not-recorded test | D32 |
| §3, §5, §6, §8, §19, §21, §23, measured figures | Every "under notice" label and copy line renamed Under eviction, parts always named; historical mentions kept | D28 |

Also from D28, outside DA-09: DA-08's two forward-looking lines now say Tenants carries the word
too, and DA-01 §3's "this screen never says under notice" line is rewritten around the new name.

---

## Doc fixes owed to Vivek

These do not affect the product. They matter because the guide is written for support and owners.

| # | Fix |
|---|---|
| G1 | Upcoming Dues is written last in the guide. It is sixth on the screen and sixth in the sheet |
| G2 | The status glossary lists tenant statuses 0 to 3 only. The table also has 4 (invite) and 5 to 8 (deleted states). A support reader tracing a status 4 person will not find them |
| G3 | The guide contradicts itself on Current FY: the glossary says 1 Apr to today, the tile description says within this financial year. D2 settled it as year to date; both places should say so |
| G4 | Upcoming Dues: "Food/Others appear only when configured" is not true of the build, which is rent only. After D8 it will be true; until then the line describes a card that does not exist |
| G5 | The guide marks Settlement Pending "(Live)" and calls Collection Overview "Period, with two live tiles". The tile is window-scoped in the sheet and the build; Current FY keeps a fixed window, which is not the same as live |
| G6 | The guide's Collection filter list still names Coming up, an option the app cannot send (F17) |
| G7 | The guide calls Expense Overview "Period, with one live tile". Two of the four tiles ignore the filter, and Current FY keeps a fixed window, which is not the same as live. Same fault as G5 |
| G8 | The guide's Expense section describes the five cards. It mentions the View all sheet once, inside a whole-tab assumption, and never describes its rows. Its four fund rows are the least obvious numbers on the tab, and §6 attaches three rules to them that are each a wrong number if missed. A support reader asked where "Paid from staff's own money" comes from has nothing to read |
| G9 | The guide states as a whole-tab assumption that "the window's upper end never goes past today". True of five blocks and not of the trend, which runs to the end of the month (F63). Once F63 is fixed the sentence becomes true |
| G10 | The Occupancy losing-money section says revenue loss is priced at "the room's per-bed rent (its own tenants' average, falling back to the group average)". There is no group-average fallback in the query; an empty room prices at zero (F78). The guide should describe what ships, whichever way F78 is fixed |
| G11 | The guide's Occupancy filter list names Coming up, an option the app cannot send (F83), and its Overview section describes the Coming-up projection as though a reader could reach it. Same fault as G6 on Collection |
| G12 | The guide says the Occupancy period rate uses "today's rentable count for past periods" as an assumption, and separately says the trend uses capacity "as it stood that month". Both are true of the code and the guide never says the two blocks therefore disagree about the same month (F68). A support reader asked why the tile and the bar differ has nothing to read |
| G13 | The guide's Tenant filter line ends "(+ Coming up for forward blocks)", an option the app cannot send, and the Move Metrics and Overview sections describe the Coming-up behaviour as though a reader could reach it. Same fault as G6 and G11 |
| G14 | The guide's Tenant Overview prose lists "Active / Approved Bookings" among the live tiles. Approved Bookings is a window number, and the guide's own appendix says so |
| G15 | The guide says stayed-after-notice comes "from `tenant_agreement_renewals`". It is computed from the tenant records, living tenants whose confirmed leaving date fell in the window; only Completed and renewed read the renewals table |

---

## What shipped better than we specified

| # | What | Why it matters |
|---|---|---|
| B1 | The Dues base pool is not a fresh query. It calls the homescreen's own `DuesListHelper.buildBaseQuery` | Dues on the homescreen and Dues in analytics cannot drift apart. The sheet never asked for this. Worth writing into DA-01 §3 as a rule the next screen inherits |
| B2 | The View all screen's code refuses to build a chip the design draws on Past Dues, with the comment "Design's chip on Past Dues is a flagged bug (§21.9)" | The sheet caught a design defect, the developer read the sheet and honoured it over the drawn file. The handoff process working as intended, and worth saying to the team |
| B3 | The six named expense categories match on the start of the saved name, ignoring capitals, rather than on exact text | The sheet asked only that the groups "collect the common spellings" and left the how open. This answer collects thirty spellings of a returned deposit into one bar, including the misspelling "Deposite Refund". It is the rule Dues and Collection should have had, and it is the reason F59 is only about the catch-all and not about the six groups. Its one measured cost: matching the start of a name also catches a name that merely begins the same way, so "RentOk Software", our own subscription fee, lands in the Rent bar, ₹25.5 lakh across 178 expenses in twelve months, 0.014% of the tab. Named in §15 as a test rather than fixed |
| B4 | Every FlexiPe expense is found by its wallet link, never by its payment mode | §7 warned that reading the mode would bury FlexiPe in Others and leave its own row permanently empty. The build took the warning. Verified on production: all 20,301 FlexiPe expenses in twelve months are identified this way, and none is double counted in Others |
| B5 | Every tile and every card ships its own definition text, ten of them, in `expenseHints.ts` | §17 item 32 flagged eleven info icons with nobody having written a word of what they say, and named Still owed to staff, the fund rows and the bill-or-receipt figure as the three that most needed one. All three have copy. The sheet listed this as outstanding and it was quietly done |
| B6 | The vacancy aging bars were built, and the sheet said they could not be | §8 and §19 open item 1 both said nothing records when a room became empty, so the four aging buckets were blocked. The build found an answer the sheet had not: the last move-out on the room's stay history, floored at the date a disabled room was switched back on, with rooms that have history but no recorded move-out kept honestly in the Unknown bar rather than folded into never rented. It is the transitional answer §18 item 11 asked for, and it improves on its own as the migration lands. Days to Fill and the past-period revenue loss stay blocked, correctly, because both need the emptying date per turnover rather than the last one |
| B7 | The trend rebuilds capacity as it stood on each day of history | §18 item 1 asked for it and every other block was allowed the approximation. The trend honours it in full: a room counts from the day it was created, stops on the day it was deleted, and drops out for the days it was switched off, all read from the room state log. Occupied is then clipped to that same set, so Occupied plus Vacant equal the month's total by construction and the rate cannot exceed 100% on a chart that has no room to explain why it did. It is also the query that fixes F68 on the Overview, so the hard half of that fix is already written |
| B8 | Every block on this tab ships its own definition text, 23 of them, in `occupancyHints.ts` | §19 open item 2 says "No screen in the suite has written what any info icon says", and calls it suite-wide and unowned. It is written here: all eight blocks carry a hint block, and the Overview and Status blocks define every tile and every slice inside them. §19 is written to be reusable as that content and now largely is. This is the second time the same thing has been quietly done and left unrecorded, after B5 on Expense, so the suite-wide open item is now false on two screens. Sheet §19 item 2 corrected |

---

## Passes worth recording

**Overview Snapshot.** All Time, All Past, This Month and All Future match the sheet exactly. No tile
reads the top filter. Exactly two tiles carry a chip. The three add up: All Past plus This Month plus
All Future equals All Time, no overlap, no gap. The rent-only mislabel on the Projected row that the
tracker logged in August is fixed; the projection counts scheduled packages as well as rent.

**Dues (Live).** Total Dues is the same computation as All Time Dues. The four slices add up to it:
overdue is strictly before today, so a due dated today is not yet overdue, matching §3. The empty
state fires at zero.

**Bills Summary.** Bill Due counts everything that came due in the window whether since paid or not,
and is always at least as large as the This Month's Due tile, which is correct. Empty state fires when
both are zero. This Month resolves as 1st through today, matching the tile and the dues list.

**Dues Breakdown.** Category is top 5 plus Others, Added By is top 4 plus Others, both on the base
pool. Tenant Status widens to include moved-out tenants, their one home on the screen. The
under-notice bar read the confirmed leaving date, which matched the sheet as written at the time; D6
has since changed the bars. Each tab has its own empty state.

**Overdue Breakup.** Buckets 1-7, 8-14, 15-21, 22-90 and over 90, as today minus due date on the base
pool with due date before today. By Category runs on the same overdue pool. The zeros are told apart
as §17 asks: overdue money shows the bars, none but billed shows the healthy "all paid on time"
message, never billed shows the empty state.

**Deposit Dues.** Due runs on the base pool filtered to deposit categories. Received nets off refunds
and adjustment payments, so money returned or adjusted stops counting. Empty state fires when both
rows are zero.

**Breakup by Stay Duration.** The split runs on the base pool inside the window using the tenant's
short-term flag, and the two bars add up to the window's total when both are present.

**Dues by Property.** Ranked high to low, each row carrying its share of the account total. Bar width
is relative to the top property, zero-dues properties stay at the bottom, a zero total fires the
empty state, View more appears past five rows.

**View all screen.** All six tiles restated, each carrying the real month name and financial year on
its face. Chips on exactly the two allowed rows. Category rows in the specified form, "Aug Rent
Dues", top five with Others folded below and suppressed at zero. Stay rows carry their exact labels.

### Collection, verified 2026-08-24

**The toggle and §4.** The toggle is screen-level, Paid Date default; only Due Date may take a Custom
window past today. Every block answers to the view. The five filter options match the app's list, and
no Collection block carries a one-option filter (F16 sweep). The three cards the sheet gives their
own dropdown have one, following the top filter's options; the Trend's range control is 8 weeks, 6
months default, 12 months. A chip is suppressed when there is nothing to compare against; Custom
compares an equal-length window immediately before the range. The Current FY tile itself is 1 April
to today with "This financial year" on its face; only the filter option carries F9's 31 March.
Per-tile chip direction is right everywhere it is built: collection tiles green for up, Still Unpaid
red for up, Billed neutral grey.

**Collection Overview.** The three source buckets add up to Total Collection exactly, each payment
in exactly one bucket by the due date of the bill it cleared, and the bucket boundary agrees with
§3's "due date still ahead" reading of Paid Early. The negative-total state is built with §5's exact
line. Part payments hold together: a part payment splits its bill, so the paid split counts and the
unpaid split stays on Dues. Measured on production, live non-test properties: exactly one part-paid
invoice exists, so Billed, which reads only unpaid and paid bills, statuses 0 and 1, loses nothing.
Billed excludes written off and refunded bills as §5 asks, and Billed minus Collected & Adjusted
equals the unpaid amount exactly.

**Collection Breakup.** Top 3 plus Others fold, Others carrying its drill and, on Received by, its
staff count. Old cash codes 202 and 207 fold into Cash (§17.8). Online modes collapse to RentOk on
both tabs. Blank receivers read "Not recorded" (§6). Adjustment modes excluded in Paid Date view,
where they are worth ₹0.

**Collection Status.** Paid Date only, hidden in Due Date with a real hidden return
(`collectionService.ts:587`, F23 sweep). The four bars are the same four numbers as the tiles. The
first bar carries the window's own name, matching design-fix item §19.12. Empty state fires when all
four are zero.

**Adjusted Collection.** Grid order matches §8. Deposit, advance and caution each sum the bill
amounts their mode cleared, by paid date. Empty state fires on four zeros; its copy is wrong (F39)
and its Discount figure reads a dead column (F40).

**Collection Trend.** Own range, the top filter has no hold. Collection is the bottom segment in
both views. Paid Date green counts money arrived, capped at today on the running period, yellow the
period's billing, two independent numbers as §9 says. Due Date yellow is the remainder, so the bar's
top is exactly Billed, the reading §9 recommends. The running bar is marked in progress, weekly bars
are calendar weeks, and the empty state fires on no data.

**Collection by Property.** Hidden below two properties with a real hidden return
(`collectionService.ts:714`, F23 sweep). Zero-collection properties stay listed at the bottom. Each
row carries its share of the account total, §19.21 done right; bars are relative to the top
property; View more appears past five rows.

**Payment Settlement.** Collected, Settled and Unsettled add up; the bar is two segments, §19.8 done
right. Destination naming falls back to the contact-support line per row. The card's yes test and
coverage are F42 and F43.

**View all sheet.** Six groups as §12 lists them. Category rows are every category, largest first,
with the billed side in Due Date view. FY rows carry the year on their face. Adjusted rows carry
window, FY and all time, short of discount (F45).

### Expense, verified 2026-08-24

**The filter and the screen's behaviour.** Five options matching the app's list, This Month default.
Custom is stopped at today twice over, once in the config the app reads and once in the service, and
Expense is the only tab that does this. Current FY runs 1 April to today as a tile and as a filter
option, so F9 genuinely does not reach here. The three cards with their own dropdown carry the top
filter's five options; the trend carries 6, 12 and 24 months and survives a page load on a top filter
it does not recognise. No card carries a one-option dropdown. Chips hide on All Time. Chip direction
is right per tile: red for rising spend, neutral grey for the count, none on Still owed to staff.
There is no view toggle, so the F31 and F36 toggle sweeps do not apply.

**Overview.** Four tiles in the sheet's order with the sheet's labels, the two fixed ones carrying
their face note where the others put a chip. The block is titled "Overview", not "Overview (Live)",
so design fix §17.5 is already done. Still owed to staff reads the passbook rather than re-inventing it, so
§15.15 passes properly; the check and its citations are in F53. §5's claim that a quarter of spending
comes from staff pockets is true as a share of money: ₹48.65 crore of ₹185.1 crore over twelve
months, 26.3%. The sheet's own appendix records 26% as a share of expenses, a different measurement
that this review did not re-run.

**View all sheet.** Eleven rows in §6's order. Average per expense hides at no expenses; petty cash
and the quarantine row hide at zero; the sheet opens on a dead window with zeros rather than refusing
to open, as §6 asks. The fund rows are the part most likely to be wrong and are right: an expense
paid from two sources is counted once in the total and split only across the fund rows, no passbook
row is attached to more than one expense so nothing is double counted, and deleted expenses drop out
of the window entirely along with both their passbook rows. **Measured on production**, twelve months,
live properties: across 2,847 properties that spent anything, not one has fund rows overshooting its
own total, and the whole platform's unexplained remainder is ₹1.67 lakh out of ₹185.1 crore. The
no-bill test is sound: no expense carries an array of blank strings that would pass as a receipt.

**Expense Breakdown.** Payment Mode passes in every detail. All nine codes in §7's list map to §7's
labels and all nine hold real money (the F40 sweep). Unrecognised modes fall to "Online" and are
almost nothing: 197 expenses carry mode code 0 and a further 4 carry no mode at all, ₹1.22 lakh
between them, 0.01% of the money. So "Online hides at zero" means most properties never see the row. The tab has no rollup, every mode sorts by
amount, and the real Others mode carries no drill, which is all three of the rules §7 wrote specially
for this tab. Category and Payment Mode run the same test over the same set of rows, so the two tabs cannot
disagree on the card's total (the F36 sweep). The Others sheet lists the remaining named groups first
and then the typed names by amount, and spend with no category at all appears as "No category" rather
than folded into anything.

**Top Payers and Vendors.** The tab never derives the payer from the record's creator, which was §8's
flat prohibition. Names are trimmed and merged only when identical. Blank names get their own row and
stay in the total, and §8's "a handful of rows across every property" is exactly right: over twelve
months, 4 expenses worth ₹3,000 have no payer and 7 worth ₹4,900 have no payee. Three largest then
Others, each row with its amount and share, both tabs summing to one card total.

**Expense Trend.** Its own range, the top filter has no hold. Months oldest to newest, the running
month marked in progress and drawn lighter, an all-zero range firing the empty state.

**Expenses by Property.** Hidden below two properties with a real `hidden` return
(`expenseService.ts:545`), the only hiding rule this tab has and the F23 sweep passes on it.
Zero-spend properties stay listed at the bottom, rows sort high to low, bars are relative to the top
property, each row carries its share of the account total, and View more appears past five rows.

**Windows, settled earlier and re-checked here.** F9 and F17 both exempt this tab and both exemptions
hold in the code: Current FY ends at today, Custom caps at today, and there has never been a forward
branch to leave behind because the sheet never asked for one.

### Occupancy (Inventory), verified 2026-08-24

**The filter and the toggle.** Five options matching §4 exactly, Now default, and no All Time, which
is this tab's own deliberate difference (`masterConfig.ts:84`). **F9 does not reach this tab**: the
Current FY case ends at today (`occupancyService.ts:83`), re-checked rather than taken from the
earlier pass. The toggle is Bed and Unit with Bed default (`masterConfig.ts:150`), matching §4 and
design fix 5. No block carries a one-option dropdown, so the F16 sweep passes. The toggle does not
hide on an all-singles property, which §4 asks for and the sheet's appendix put at 29.5% of active
properties; small enough to sit inside F80's group of face-level gaps rather than earn its own entry.

**The two toggle sweeps, which do apply here.** F31, does a tile keep its chip across the toggle:
moot, no tile has a chip either side (F65). F36, do a card's tabs still sum to the card's total in
every view: the donut is the only card with parts, and it sums correctly in Unit view. In Bed view it
does not, which is F70, and that is a rounding of over-occupancy rather than a toggle fault.

**Overview Snapshot.** Five tiles in §5's order with §5's labels, switching with the toggle. The rate
is computed from beds in both views and says "by beds" on its face, which is §18 item 2 and design fix
14's sibling. Occupied, Vacant and Rentable are read from the widget on Now, so the screen and the
rooms list cannot drift. The window's name rides beside the title as §5 asks.

**Occupancy Status.** The donut carries §7's slices in each view, Semi-Vacant appearing in Unit view
only and only above zero, Disabled only when present. The header carries both totals, total and
rentable, which is design fix 32 done. The over-100% sentence exists (`occupancyService.ts:276`),
though it states the over-occupied half without the vacant half §7 asked for. The money tiles are the
two §7 specifies, with §7's own labels, and the second one reads "per Rentable", which is design fix
14 done. **Measured on production**, living tenants on live properties: 93.7% carry a rent amount,
330,532 of 352,750, so the averages read about 6% low rather than half, and the sheet's appendix
figure of 43 to 54% was about beds carrying a rent, a different measurement.

**Vacant Room Status.** Six bars in §8's order including never rented and unknown, which the design
never drew. Counted per room in both views, with "by rooms · As of today" on its face, matching §8 and
§4. Bucket edges are 0-7, 8-15, 16-30 and 31+, so day 30 falls in exactly one bucket and both bar
charts agree, which is §18 item 19 and design fix 8. Never rented comes from the absence of any stay
history rather than from a zero age, which is §18 item 13 exactly. The subtitle carries the card's
direction, design fix 9.

**Upcoming Vacancy.** Built from approved leaving dates only, dates from today onwards, so a tenant
whose date has passed is correctly absent, which is F14's row 3 holding. Bookings cannot leak in:
status 1 only, which is §18 item 7's test. Buckets and subtitle agree with its neighbour.

**Agreements ending soon.** Counts agreements and says so on the axis, ignores both the toggle and the
filter with "From today onwards" on its face, all three matching §10 and design fixes 10 and 29. The
footer restates the 90-day total. The end date itself is this tab's largest question and is F77; the
same four-step rule runs on the Tenant tab, so both screens carry it and both are fixed together.

**Occupancy Trend.** Its own range, 6, 12 and 24 months with 6 default, and the top filter has no hold
on it, matching §11 and §4. Months run oldest to newest, the running month is marked in progress, and
the chart plots the level rather than movement, which is §11's deliberate exception. Its capacity work
is B7. The empty state fires on an all-zero range.

**Property Wise Occupancy.** Hidden below two properties with a real `hidden` return
(`occupancyService.ts:582`), so the F23 sweep passes on this tab. Sorted highest to lowest by vacant
space rather than by rate, which is §13's whole point and design fix 11. Properties with nothing
rentable still appear at zero, and every property the manager selected produces a row, so §13's
account-grouping warning does not bite: the scope comes from the request, not from a grouping field
that is missing on 29% of properties.

**Empty states, the F64 sweep.** All eight are defined and all eight are wired. Not one is written and
left unsent, which is the only tab so far where that sweep passes cleanly. What is wrong with them is
their words, not their plumbing, and that is F74.

**The F55 clamping sweep.** Vacant is floored at zero on a period (`occupancyService.ts:195`), which
is right here: a bed cannot be negatively empty, and over-occupancy is its own number on the same
screen. The rate is deliberately not clamped, so it can read above 100% as §3 requires.

**Two things checked here that turned out not to be passes**, recorded so the correction is visible
rather than quiet. The Unit view donut was first written up as summing correctly; it does not, for
zero-capacity rooms holding somebody, and that is now inside F70. And the losing-money card was first
written up as silent about its window; it is not, it carries a plain line, and that is now inside F80.

**The F62 sweep, an internal string reaching the manager.** Nothing on this tab prints a stored string
from our own writers. Every label is written in the analytics layer, and the only free text near this
screen, the room type field the sheet's appendix found with 60-plus spellings, is deliberately not
used as an axis.

### Tenant, verified 2026-08-25

**The filter and §4.** Five options matching the app's list, This Month default, no view toggle, so
the F31 and F36 toggle sweeps do not apply. **F9 does not reach this tab**: Current FY ends at today
(the `current_fy` case, `tenantService.ts:127` to `:129`), re-checked in the code rather than taken
from the earlier pass. No block
carries a one-option dropdown (F16 sweep). Chips hide on All Time. **The chip helper is real and
called**, unlike Occupancy's (F65 sweep): `chip(cur, prev, goodDir, note)` at `:44` runs on Approved
Bookings up-good, Notices Raised down-good, Move-ins up-good and Move-outs down-good, exactly §4's
polarity table, and returns null when there is nothing to compare against. Chip comparisons are
event counts recomputed from dated records, so the F5 as-it-stood trap does not bite here.
**F47 re-checked in this file alone**: the same two overflow lines sit at `:160` to `:161` at head
`6797eec6d`, so the open item's five-service fix list stays right.

**Overview Snapshot.** Seven tiles in §5's order with §5's labels, Past their date in the Move-outs
slot as §5 moves it, live tiles carrying "as of today" and no chip, the two window tiles carrying
chips (§25's design fix 1 done right in the build). The two money lines ride under the tiles with
the sheet's own note format, and leaving-with-dues only appears when somebody owes. Rent at risk
reads the monthly rent of everyone under notice on the parent test, past-their-date correctly
outside it. The unpaid balance behind leaving-with-dues reads the invoice pool through the
homescreen's own property key and payer join.

**View all sheet.** Three sections as §6 lays them out, indented rows as layers, every row keyed for
its drill. Under notice equals approved plus awaiting by construction, one `UNDER_NOTICE` test.
Awaiting-confirmation hides at zero as §6 asks. Arriving this month reads the calendar month
whatever the filter says, which is what "expected within the current month" means. Bookings
confirmed reads real approval rows only, the one correct count of the four the F93 entry examined.

**Move metrics.** Move-ins count status 0 and 1 by joining date, so someone who joined and left in
the window still counts, and a room change moves neither number (320 multi-room tenants platform
wide, the sheet's own measure). Move-outs read the confirmed leaving date, else the checkout date;
only 198 of 636,873 moved-out tenants on live properties carry neither, 0.03%, so the "never shows
zero" test holds everywhere it matters. Net change is stated with the cards as §7 asks, chips run in
opposite directions correctly.

**Verification.** The card states its base, both tabs split over all active tenants, and the action
bar names only the workable queue, overdue among the last 90 days of joiners, exactly §9's rule,
with the full overdue count still on the card. The e-KYC test reads the latest verified KYC row per
tenant, manual as the remainder, unverified as no row at all.

**Tenancy Details.** The four agreement buckets cover the base with no overlap: measured on
production, zero active tenants fall between them. Profile completion checks exactly the ten fields §3 lists,
and the tenant list's own completed-profile test reads the same ten
(`getAllTenants.ts:1324` to `:1333`), so the number and the list agree by construction, the F86
sweep's question answered yes here.

**Upcoming Eviction.** Five bands with Inventory's edges, 0-7, 8-15, 16-30, 31+, each split pending
against approved, pending bucketed by the requested date and approved by the confirmed one, §3's
rule exactly. A withdrawn notice leaves the card because only active notice rows count.

**Agreement Expiry.** Verified against §3 and §12 **as they stood before D26**: the four-step end
date rule matched that §3 step for step, start date as last renewal else joining, and the footer
carried the assumption line that §12 then asked for. D26 and F100 supersede both halves, so the
build now owes the No term recorded group and the footer's assumption line goes. The 30-day band and
Journey's Renewals in 30 days and Renewal Due are one expression three times, so they cannot
disagree (§23.5 honoured in effect). Bands do not overlap; day 30 falls in exactly one.

**Profile and Details.** Every tab carries the coverage line in §3's own words, built from recorded
over base. City groups on the trimmed lower-cased district and displays it title-cased, top 5 plus
Others, the F19 sweep passing here. Tenant Type reads `working_type`, the field with real coverage,
26.6% of active tenants measured today against the sheet's 23%, not the lookalike field the sheet
warned is 99% empty. Age bands cover everyone once, and an invalid under-14 date of
birth is excluded from coverage on purpose, with the comment saying why.

**Tenure.** Bands on the joining date with no gap and no overlap, and the code comment names the
off-by-one this fixed. Future joining dates land in Under 1 month rather than vanishing.

**Property Wise Active Tenants.** Hidden on one property with a real `hidden` return (`:1100`), the
F23 sweep passing. Every property in scope appears including zeros, sorted highest first, each row
carrying its share of the account total and a bar relative to the top property, §18 exactly.

**The F79 sweep.** Every share and bar percent on the tab is computed from raw counts before
formatting; nothing reads a display string back.

**The F52 count re-confirmed.** Fifteen null returns, one per block, all falling back through the
caller to the shared empty-card shell the registry holds, as F52 already records. Nothing new here.
