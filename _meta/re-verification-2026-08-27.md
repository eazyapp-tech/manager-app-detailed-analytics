# Re-verification, 27 August 2026

Every open item in [[01 — Build Verification Log]] checked again against the code as it stands on
`origin/master` today, not against the commit the log is pinned to.

**Why this run happened.** The log is pinned to commit `3a13e08ac`. Since then **27 commits have
touched `src/v1/analytics/`**, about 2,250 lines, every one of the five services. The log had
re-verified two of those 27. Twenty-five had never been checked.

**How each item was judged.** By reading the current code for the function the item names, never by
reading a commit message. Three commits this run sounded like they fixed something and had not:
"upcoming vacany room wise logic done" changed which rooms count and left the colour defect alone. Two
further defects read as already handled because the guard sits one layer outside the value it has to
change; both predate this review, and F55 and F65 already describe them correctly.

**What the outcomes mean.**

| Outcome | Meaning |
|---|---|
| Fixed | The code now does what the item asked. Line and function named. |
| Part done | Some of the item landed, some did not. What is left is stated. |
| Not started | The code is unchanged in the way the item cares about. |
| No longer applies | A ruling or a product change removed the item. |
| Waits on an answer | Not a code item. A question to Vivek or a ruling to the owner. |

---

## New ruling from this run

**D33. The Current FY tile keeps the full financial year; the chip says what it compares
(2026-08-27).** The tile goes on showing 1 April to 31 March. The chip beside it goes on comparing
1 April to today against 1 April to the same day last year, which is what the code already does, and
it now carries a note on its face saying it compares the year so far. Nothing about the numbers
moves.

The two items this closes were both asking for the same thing, that the financial year window stop
at today: the Current FY tile row (F1, and its second copy in the View all screen, F25) and the
Current FY filter option row (F9). Both are withdrawn. The mismatch they described is real and gets
answered by wording inside the chip rework, not by moving the window.

Rejected while ruling: making the chip compare a full year against a full year. It is honestly in
sync with the tile, and it would read heavily down every year from April through March, correcting
itself only on 31 March.

---

## Dues, checked 2026-08-27

Twenty-two rows. **One fixed, one part done, sixteen not started, two withdrawn by today's ruling, two
waiting on an answer from Vivek.**

| Item, and what it asked for | Outcome | What the code says today |
|---|---|---|
| Current FY tile: show `fy_ytd` instead of `current_fy` (F1, F25) | No longer applies | Withdrawn by D33 above. `duesService.ts:274` and `:914` still read `current_fy`, and that is now the intended behaviour |
| Current FY filter option: end at today, not 31 March (F9) | No longer applies | Withdrawn by D33. `resolveWindow` case `current_fy` at `:71` still runs to 31 March, intended |
| Projected Due, rent half: count living tenants and confirmed bookings (F2) | Not started | `projectedDueTotal` at `:841` still filters `t.status = 1` only. Bookings are still left out |
| Projected Due, package half: add a tenant filter (F2) | Not started | The `pkg` block at `:856` to `:863` joins `tenant_packages` with no tenant filter of any kind |
| Projected Due, rent day: fall back the way `chargeRentTenants` does (F3) | Not started | `:837` still uses its own chain, `rent_addition_date` then day of joining then the 1st, with no read of the property's setup |
| Upcoming Dues: the same three fixes, second copy of the query (F2, F3, D8) | Not started | `getUpcomingDues` at `:758` to `:767` repeats `t.status = 1` and the same rent-day chain, word for word |
| Upcoming Dues: add the other configured types, extend on a forward window (D8, D9) | Not started | `:819` still ships one tab, Rent. The window is fixed at tomorrow through month end and never reads the filter |
| Both Dues chips: comparison figure as it stood on that date (F5) | Not started | `overviewAggregate` windows `due_date` over today's unpaid pool, so a bill from last month that has since been paid has dropped out of the figure it is compared against |
| Both Dues chips: red for up, green for down (F6) | Not started | `chip()` at `:133` returns green for up and red for down. The comment two lines above it says "Up is BAD on Dues" |
| Dues Breakdown: three active bars, Active no eviction · Eviction pending · Eviction approved (D6) | Not started | `:439` to `:441` still split two ways on whether a leaving date exists, and the label at `:457` is still "Under notice", which D28 renamed suite-wide |
| Overdue Breakup: rename "By Amount" to "By ageing" (F15) | Not started | `:546` still reads `label: 'By Amount'` |
| Overdue Breakup: remove the one-option dropdown (F16) | Not started | `registry.ts:115` still passes `filter_values: ALL_TIME_ONLY`, a list with one entry in it |
| Overdue Breakup: add "As of today" (F18) | Not started | `registry.ts:105` title is still plain "Overdue Breakup" |
| Deposit Dues: add "As of today" (F18) | **Fixed** | `registry.ts:125` now reads "Deposit Dues (As of today)". Landed today in commit `49c63275f` |
| Deposit Dues: match categories case-insensitively and trimmed (F19) | Not started | `:644` and `:661` still match `due_type IN ('Security Deposit', 'Caution Money')` on exact text |
| Deposit Dues: say what `net_amount` means against `amount`, then add the departed-tenants line (F21, D10) | Not started | Still two different money columns, `net_amount` for held at `:635` and `i.amount` for due at `:661`. The payer filter at `:648` is still `status IN (1, 2)`, so departed tenants are still out and have no line |
| Breakup by Stay Duration: hide when the rule says, and add an empty state (F23, F24) | Part done | The empty state ships at `:680`. The hide does not exist, and the View all copy of it now carries a written-in `TODO` at `:918` |
| Dues by Property: hide on a single-property account (F23) | Not started | No hide rule in either `getDuesByProperty` or the registry entry |
| Forward window: carry the D9 rules onto the Custom path (F17, D9) | Not started | `resolveWindow` case `custom` at `:80` hands back the dates raw. No chip suppression, no dash for event counts, no forecast extension. The one good half: there are no dead `coming_up` branches left in this service |
| Bookings: confirm whether `is_confirmed = 0` means pending or cancelled (F4) | Waits on an answer | A question to Vivek, not a code change |
| Added By: say what creator codes 5 and 6 are (F26) | Waits on an answer | `addedByLabel` at `:99` still folds every unmapped code into "RentOk". The answer is owed before the fix can be written |

### One thing that moved the wrong way

**Dues by Property has lost each row's share of the account total.** `duesService.ts:711` now carries
the share calculation commented out, and `:704` computes a total that nothing but the empty check
reads. The log records this row's share as a pass, in "Passes worth recording": "Ranked high to low,
each row carrying its share of the account total." That is no longer true. Raised as a new finding in
this run rather than quietly dropped from the pass list.

---

## Collection, checked 2026-08-27

Eighteen rows. **Two fixed, two part done, twelve not started, two waiting on a ruling.** Two commits
landed here since the log's own re-verification, `3e6862174` "improve future window handling" and the
design-QA sweep.

| Item, and what it asked for | Outcome | What the code says today |
|---|---|---|
| Custom forward path: the D9 rules, and delete the dead `coming_up` branches (F27, F17) | Part done | Forward handling arrived in `3e6862174`: `computeWindows:178` now works out `isFuture`, `:179` drops the chips, and the money tiles dash instead of reading ₹0 at `:394` and `:401`. Two things are still owed. The dead `case 'coming_up'` is still there at `:161`, and `isFuture` only catches a window that starts after today, so a custom range that straddles today still behaves as though it were wholly past |
| Overview: Advance and Current FY keep their chips in Due Date view (F31) | Not started | Both are `trend_object: null` in Due Date view, `:394` and `:396`, while both carry chips in Paid Date view at `:353` and `:354` |
| Overview: cap the due window at today, both views one rule (F32, D11) | Not started | `computeWindows:182` caps the paid window at today and `:201` hands the due window back at the full period end. The comment at `:117` still describes the old rule as intended |
| Overview: Collected & Adjusted comparison bounded to the same elapsed moment (F33) | Not started | `:374` reads the current window to the end of the month while `:376` reads the previous one to the same elapsed day. The two sides still measure different lengths of time |
| Overview: adjustments-only line and an empty state for the tile row (F34) | **Fixed** | The adjustments-only line ships at `:339`, "No money received. ₹X of bills were cleared from deposits and advances." The tile row now has three empty states at `:318` to `:333`, telling apart a new account, one that has never collected, and one that is simply quiet |
| Breakup: case-insensitive, trimmed category grouping, as Dues does (F35) | Not started | `:512` and `:527` still group on `COALESCE(inv.due_type, 'Advance')` with no lower-casing or trimming. Dues got this treatment and Collection did not |
| Breakup: adjustment modes as rows on the Due Date Mode and Received by tabs (F36) | Not started | `foldModes:573` and `foldReceivers:598` still skip the three adjustment modes in both views, so the tabs still fall short of the card's own total in Due Date view |
| Breakup: the one-line no-tenant note on the Status tab (F37) | Not started | The comment at `:531` says payments with no tenant stay in the total but not in the bars. Nothing says it on the screen |
| Breakup: billed side per row on the Due Date Status tab (F38) | Not started | The Category tab did get its billed side, `:513` to `:522`. The Status tab at `:533` still returns collected only |
| Breakup and Dues Breakdown: three active bars (D6) | Not started | `:540`, `:541`, `:549` and `:550` still split two ways, and the label is still "Under notice", which D28 renamed suite-wide |
| Empty copy: ship §15's replacement copy on four blocks (F39) | Part done | Collection by Property is exact, `:67`. Adjusted Collection now names all four kinds of adjustment at `:65`, which was the substance, though not §15's sentence. The Mode tab at `:61` still promises cheque and leads with cash, and Collection Status at `:64` still promises a pending row the card does not have |
| Adjusted Collection: Discount reads the credits records, windowed by paid date (F40) | Not started | `:712` still sums `inv.discount`, the column the review measured as empty on production, and `:715` still windows it by due date rather than paid date. Both halves untouched |
| Payment Settlement: pick one account record per payout before summing (F87) | Not started | `:927` still joins `account_details` on bank account number or VPA with no single-row pick, so a payout matching two records is still counted twice. This is the 14% overstatement |
| Payment Settlement: read the payout's status again (F88) | Not started | `:889` still calls a payment settled on the mere existence of a payout id or a wallet entry. No status is read, so a failed transfer still reads as settled, and Flexi Pay balances still count as money that reached the bank |
| View all: Adjusted rows include discount (F45) | Not started | `adjustedTotal:1038` sums the three adjustment modes only. Discount is still not in it |
| Healthy states: the three §15 messages (F46) | **Fixed** | All three ship. "Everything billed for this period is collected." at `:392`, "All online money has reached your bank." on Unsettled at `:864` and on the Settlement Pending tile at `:343` |
| Total Collection agrees with neither the list nor the homescreen (F86) | Waits on a ruling | The refund netting is in `COLL_AMT` at `:238`, clamped at zero per bill, and the comment claims it matches the homescreen. Whether the three surfaces now agree is a question about live numbers, not about the code. It needs the owner's ruling before Vivek touches this tab again |
| The View all sheet now follows the toggle (F89) | Waits on a ruling | Still true. `:976` reads the toggle and branches on it throughout. Raised as going too far the other way, and still open |

### The same thing has now happened twice

**Collection by Property has also lost each row's share of the account total**, `collectionService.ts:812`,
commented out exactly as in Dues at `duesService.ts:711`. Two blocks, same removal, same shape. Worth
asking Vivek whether this was deliberate before either is put back.

---

## Expense, checked 2026-08-27

Thirteen rows, including the one suite-wide item that lives here. **Three fixed, two part done, eight
not started.** The suite-wide one is the single biggest fix in this whole run.

| Item, and what it asked for | Outcome | What the code says today |
|---|---|---|
| Every chip, all five tabs: end the "same point last month" window at the end of the previous month (F47) | Not started | Checked in all five services and unchanged in all five: `duesService.ts:148`, `collectionService.ts:172`, `expenseService.ts:149`, `occupancyService.ts:74`, `tenantService.ts:157`. Every one still reads start-of-last-month plus today's day number, with no clamp, so on the 29th, 30th and 31st of a month longer than the one before it the comparison window still reaches into the current month |
| Four empty states: the in-window zero copy where the property has spent before, the setup line where it never has (F50) | **Fixed** | A `hasAnyExpense` probe now decides, and all four blocks use it: Breakdown at `:385`, Top Payers at `:473`, Trend at `:539`, By Property at `:582` |
| Overview row: give it the screen's own in-window zero (F51) | Not started | `getOverview` at `:215` returns its four tiles whatever the total. No empty state on the row |
| Expense healthy state: "Nothing owed to staff right now." on the tile and on the View all row (F51) | Part done | The tile has it, `:220`, word for word. The View all row that repeats the same number, `:289`, still has no healthy state |
| View all, quarantine row: take the absolute value before the two helpers that floor it at zero (F55) | Not started | A fix was attempted and lands outside the helper instead of inside it. `:299` reads `Math.abs(n(totals.neg_sum))`, and `n()` at `:54` floors a negative to zero before `Math.abs` ever sees it. The row still always reads ₹0, which is the one thing it exists to avoid |
| View all, no-bill row: print the amount once (F56) | Not started | `:293` still prints it as the value and again inside the sub-label |
| View all sheet: carry the window's own name, not today's date (F57) | Not started | `:302` still sends `window_label: w.today` |
| Every window: say which clock `paid_date` is stored on, convert if it is UTC (F58) | Not started | Every query still reads `e.paid_date::date` with no conversion and no note anywhere saying which clock it is on |
| Expense Breakdown, Category: group the catch-all on the lower-cased trimmed name (F59) | Not started | `CAT_GROUP` at `:38` now trims but still does not lower-case, so two spellings that differ only in capitals still draw two bars |
| Top Payers, Paid by: label FlexiPe spending "FlexiPe (Owner)", never the raw string (F62, D19) | Not started | The Payment Mode tab does name FlexiPe at `:435`, which is a different tab. `payerItems` at `:495` still returns whatever string sits in `e.payer`, trimmed and unmapped. The bracketed string is still on the bar |
| Expense Trend: end the range at today, like the other six blocks (F63) | Part done | The running bar is now marked `in_progress` and drawn in a lighter blue, `:535` to `:536`, so it no longer reads as a spending collapse. The window itself still runs to the end of the month at `:527`, so the tab still has one block on a different clock from the rest |
| Expenses by Property: send the empty state that is already written (F64) | **Fixed** | `:582` now sends one when the window's total is zero |
| Every block, all five tabs: an unresolved property must not fall back to the block's scaffold; fix once in `toBlock`, not 51 times (F52, F84) | **Fixed** | Done exactly where the item asked. `service.ts:161` holds a list of the five built prefixes, and `:172` to `:175` send an empty state instead of the registry's demo numbers whenever a built block returns null. The comment above it states the rule: fake demo numbers on a real account is a data bug. This closes all 51 places in one change |

### The same thing has now happened three times

**Expenses by Property has also lost each row's share of the account total**, `expenseService.ts:576`,
commented out exactly as in Dues (`:711`) and Collection (`:812`). Three blocks, one shape of removal.
This is a pattern, not three accidents, and it is worth one question to Vivek rather than three
tickets.

### A scope change nobody recorded

**The Issues tab has been taken out of the navigation.** `masterConfig.ts:170` to `:181` comments the
whole tab entry out, with a note saying to re-enable by uncommenting. Its eleven blocks are still in
`registry.ts`. The log's "What is in scope" says Issues is declared with eleven blocks of sample data,
which is now only half true: the blocks exist and the manager cannot reach them. Landed in commit
`0625481ad`.

---

## Occupancy, checked 2026-08-27

Twenty rows. **Three fixed, four part done, thirteen not started**, and one shipped behaviour the owner
ruled correct has been undone. This is the tab with the most movement since the review: six
commits, 478 lines changed, and a whole new query, `bookedOnVacant`.

| Item, and what it asked for | Outcome | What the code says today |
|---|---|---|
| Every tile: build the chips at all; `chip()` is written and never called (F65) | Not started | `chip()` is still defined at `:56` and called nowhere in the file. `getOverview:323` now reads `trend_object: showChip ? null : null`, a ternary that returns null on both sides. Scaffolding went in; nothing was wired to it |
| Under eviction tile: take in pending notices too, both parts named on a second line (F66, D21) | Not started | The count at `:183` still tests only for a recorded leaving date, which is the approved half. There is no second line, and the label at `:320` is still "Under Notice", not the "Under eviction" D28 settled |
| Under eviction tile: Unit view counts units holding someone, not people (F66, D22) | **Fixed** | `:309` now picks `underNoticeRooms` in Unit view, and that is a room count built at `:192` |
| Booked, three surfaces: confirmed bookings only (F67, D20) | Not started | A new method, `bookedOnVacant` at `:148`, replaced the old counting and did the grain properly. It still takes every tenant at status 2, `:155`, `:171`, `:187`, `:238`, with no approval test anywhere. D20 says only an approved booking, or one on a property that needs no approval, counts as booked |
| Overview, a period: divide by the capacity as it stood, reusing the trend's own query; exclude future-scheduled stay rows (F68) | Not started | `avgOccupiedBeds` at `:257` still divides by today's rentable count, and its own comment at `:256` now says so out loud: "Denominator uses current rentable (approx)". No filter on pending stay rows. `trendMonths` at `:573` does rebuild capacity per day, so the hard half of the fix is written and sitting one function away, unused by the tile |
| Occupancy Status: two chips, not four; hide at zero; the Under eviction second line (F69) | Not started | Still four chips at `:377` to `:384`, and the comment at `:375` states the current rule as intended: "always shown incl. zero (matches homescreen)" |
| Occupancy Status: make the legend add up to the header, both directions (F70) | Part done | Over-occupancy is now explained rather than fixed: `:399` adds a note when the rate passes 100%, "N beds are over-occupied while others sit empty." The legend still does not add up to the header, and the Unit-view half, rooms with no declared capacity holding somebody, is untouched |
| Occupancy Status: count booked at bed grain, or say the count is by room (F71) | Not started | `:379` still prints `now.bookedRooms`, a room count, under the label "Booked Beds". The bed-grain number now exists, `bookedInv.beds`, and is used for the Vacant split at `:365` but not for this chip |
| Rentable units: exclude rooms that declare no capacity (F72) | Not started | `:141` is still total rooms minus disabled rooms |
| Vacant Room Status: red marks the largest of the four aging bars only (F73) | Not started | `:473` works out the maximum across all six bars, Never rented and Unknown included, so Never rented can still be painted red. That is the one thing §8 forbade |
| Vacant Room Status: hide Never rented and Unknown at zero, four aging bands keep showing (D30) | **Fixed** | `:470` and `:471` now add each bar only when it is above zero, and the four bands are unconditional |
| Four cards: the §16 healthy copy; stop shipping the sentence §16 forbade; add the not-set-up and no-beds states (F74) | Part done | The not-set-up state is built with its button, `:278` to `:285`, though the sentence is shortened from §16's. One of the four healthy lines is exact: Vacant Room Status, `:22`. Upcoming Vacancy, Agreements and Losing money still ship their old copy at `:23`, `:24` and `:20`. The forbidden sentence, "Vacancy duration by days will appear here once rooms are empty", is still in the file at `:21` and still reachable on a property with no rooms. The no-beds case at `:351` sends the generic status copy instead of §16's "No beds added yet." and has no Add beds button |
| Upcoming Vacancy: drop the red bar (F76) | Not started | `:524` still paints the largest bar red |
| Agreements ending soon: name the eleven-month fallback with its count and give it a drill (F77, D24) | Not started | The fallback is still in the query at `:541` to `:543`. The hint at `occupancyHints.ts:30` reads "Active long-term agreements ending in the next 30 / 60 / 90 days." and says nothing about the assumption, its count, or where to go and fix it |
| Losing money: fall back to the group average, so an empty room is not priced at zero (F78) | Not started | `:688` still prices a room from its own active tenants only, and `:697` turns a null into zero. A room with nobody in it is still worth nothing. What did land is the disclosure: `:783` now says "based on N of M vacant beds with rent set", so the hole is stated on the card while the number stays wrong |
| Losing money: bar and share from rupees, sort by revenue loss, build the coverage threshold (F79) | **Fixed** | All three. `:784` and `:785` compute from the raw rupee figure, `:757` sorts by revenue loss, `:783` carries the coverage line. The comments name the tickets, REN-466, REN-467 and REN-468 |
| Three silent blocks: "As of today" on Status, Property Wise and the View all sheet (F80) | Not started | None of the three carries it. Vacant Room Status, which was not on the list, did get one at `:482` |
| View all sheet: the seven missing rows, the missing third section, the never-rented number (F81) | Part done | The rows are largely there now: never rented `:864`, past their date `:874`, bookings with no space `:875`, disabled `:857`, semi-vacant `:860`, over-occupied `:865`, booked and available `:862` to `:863`. The sheet still has two sections where the item asks for three |
| Occupancy Trend: start the chart where the property's data starts (F82) | Not started | `:626` still walks back a fixed six, twelve or twenty-four months from today. The months before the property existed now draw as honest zeros rather than invented numbers, because `trendMonths` counts a room only from the day it was created, but the empty months are still on the chart |
| Custom path: D9's rules, and delete the dead `coming_up` branches (F83) | Part done | Custom now stops at today, `:90`. The dead `case 'coming_up'` is still at `:92`, and the rest of D9, dashing event counts and extending forecasts, is not there |

### A ruling has been reversed

**Upcoming Vacancy no longer follows the view toggle, and D23 says it must.** D23, ruled on 24 August,
closed F75 as an accepted change: the build counted departing tenants in Bed view and room-and-day
vacancy events in Unit view, the owner agreed the build was right, and sheet §4 and §9 were corrected
to match. The card now reads "Always room-wise" at `occupancyService.ts:496` and never looks at the
toggle. Commit `21a506e36`, "upcoming vacany room wise logic done".

This is the only place in the whole re-verification where the code has moved away from a decision
rather than towards it. It needs the owner's word before anything else happens on this card, because
the sheet has already been rewritten around the old behaviour.

### The same thing has now happened four times

**Property Wise Occupancy has also lost each row's share**, `occupancyService.ts:832`, commented out
alongside its colour. That is four blocks with the same removal: Dues `:711`, Collection `:812`,
Expense `:576`, Occupancy `:832`. One question to Vivek, not four tickets.

---

## Tenant, checked 2026-08-27

Twenty-three rows. **Two fixed, six part done, fifteen not started.** Three commits landed here,
including the one that finally gave Renting Type a real column of its own.

| Item, and what it asked for | Outcome | What the code says today |
|---|---|---|
| Agreement end date, two services: guard the `agreement_period` cast, digits only, else null (F90) | Not started | `ENDS_ON` at `:170` still casts straight to integer, and `occupancyService.ts:541` does the same. The junk settings value that kills the whole Tenant page still kills it |
| Agreement end date, Tenant only: null means the No term recorded group, never eleven months (D26) | Not started | `:172` still falls back to 11 |
| Any failed block: catch it, so one bad value cannot kill the whole page (F90) | Not started | `service.ts:84` still runs every block through one `Promise.all`. One rejection still takes the page down |
| Custom path: D9's rules; delete the dead `coming_up` branches (F91, D9) | Part done | Custom stops at today, `:134`. `case 'coming_up'` is still at `:145`, and the rest of D9 is not there |
| Ten silent blocks: "As of today" on the live blocks, "From today onwards" on the two forward ones (F92) | Not started | No block in the service carries a face note. The registry has the phrase, but only on the demo numbers at `registry.ts:378` to `:384` |
| Approved Bookings and the Journey funnel: count real approval rows only (F93) | Part done | The scope was narrowed to statuses 0, 1 and 2 so leads no longer fold in on an auto-accept property, and the comment at `:351` explains why. The defect itself stands: `:366` still counts every tenant on an auto-accept property, dated by when the record was created, `:367`. `bookingBars` at `:679` does the same |
| Overview info sheet: add Eviction Approved and Past their date; fix the Active Bookings definition (F94) | Not started | The screen now ships seven tiles. `tenantHints.ts` still defines seven entries for a different seven: Eviction Approved and Past Their Date have no definition, and `:10` still says Active Bookings means "Confirmed bookings not yet moved in" while `:315` counts every booking, awaiting included |
| Move-outs: exclude cancelled bookings (F95) | Not started | `movedCounts:537` still counts any status-0 tenant with a leaving date. A cancelled booking gets both when it is deleted |
| Journey: rename the Under Notice bar to Eviction pending (F96, D29) | Part done | The bar was renamed, `:597`, but to "Pending Eviction" rather than D29's "Eviction pending", and its sibling at `:598` reads "Approved Eviction" rather than "Approved eviction" |
| Journey, Tenants tab: two named groups, six rows (D29) | Part done | Two of the new rows landed, Renewals in 30 days at `:599` and Agreements expired at `:600`, and Agreements expired does reuse the expression D29 named. The two headings do not exist, all five bars sit in one flat list, and "No notices" is commented out at `:596` |
| Journey strip: Churn as a share with the count beside it; left early reads `lockin_period` (F97) | Not started | `:637` is still a bare count of move-outs. `journeyFooters:655` still measures left-early against the agreement end date, and there is no coverage line |
| Upcoming Eviction: hide the Overdue bar at zero, four day bands keep showing (D30) | **Fixed** | `:832` adds the Overdue bar only above zero; the four bands at `:833` to `:836` are unconditional |
| Upcoming Eviction: fall back to the tenant-level date the tiles read (F99) | Not started | `:808` to `:813` still read the eviction-details rows alone. The service's own comment at `:182` says approval can set the tenant-level date without leaving an active row, and names the production property it was found on |
| Agreement Expiry: relabel the five bands (D31) | Part done | "Already expired" landed at `:877`. The three day bands still read "30 days", "60 days" and "90 days" rather than D31's "0–30 days", "31–60 days" and "61–90 days", and the last reads "Valid (90 days+)" rather than "Valid (90+ days)" |
| Agreement Expiry: hide Already expired and No term recorded at zero (D30) | Not started | `:877` is unconditional, and there is no No term recorded bar to hide |
| Agreement Expiry: no-term tenants leave the bands and become their own stated group (F100, D26) | Not started | The eleven-month fallback still puts them in the bands. Three quarters of the Already expired bar still rests on a date the product invented |
| Renewal figures count real terms only (D26) | Not started | `getRenewalRetention:1082` and `:1083`, and Journey's `renewals_30` at `:574`, all run on the same fallback |
| Renewal & Retention: Stayed after notice as a share of those whose date fell in the window and passed (F101) | Not started | `:1093` still counts every leaving date inside the window whether or not it has passed, and `:1114` still prints a bare count |
| Stay type always shows; renting type shows only when it has somebody in it (D32, D27, F102) | **Fixed** | Both halves, and better than asked. Stay Type always ships with an empty state at `:1140`. Renting Type now reads a real column, `t.renting_type`, at `:1153`, splits B2B against Residential, and returns `{ hidden: true }` when neither side has anybody, `:1161`. The interim donut that showed the stay split under another name is gone. Landed in commit `26896ec14` |
| Four colours (F98) | Not started | All four are as they were, and the swapped pair with them. Not verified is red, `:730`. Police "Not done, in time" is red, `:734`. The 30-day expiry band is red, `:878`. Not signed is orange, `#FF9567`, where it should be red. Profile Complete is pale yellow and Profile Pending is green, `tenantColors.ts:28` and `:29`, still the wrong way round |
| The rename, five strings: "Under eviction" replaces "Under notice" (D28, D29, S3) | Not started | The View all row still reads "Under notice", `:458`, and both Journey empty messages still say it, `:63` and `:64`. The Journey bar, the one string D29 deliberately excluded from this list, is the only one that changed |
| Empty and healthy states: five §21 healthy, six unsent empty, the not-set-up screen, the Bookings copy (F103) | Part done | The whole-screen not-set-up state is built with its button, `:261` to `:268`, and a second state for an account with bookings but no tenants, `:269`. The unsent empty states are wired: moves `:524`, renewals `:1117`, stay type `:1140`, tenure `:1201`, by property `:1233`, and every profile and details tab. Two of the five healthy states ship, Upcoming Eviction `:845` and Agreement Expiry `:887`. The Bookings-tab copy at `:64` is still the Tenants sentence with one word changed: it promises a booking's "renewals due in 30 days" |
| Three breakdowns: an Other bar for food, "girls" into the female list, no-joining-date tenants stated (F104) | Not started | All three. Food Pref still has four buckets and counts unmatched values in its coverage but on no bar, `:985` to `:989`. Gender still matches "girl" and not "girls", `:905`, so 10,184 tenants are still shown as "Prefer not to say". Police verification still tests a joining date that may be null, `:715` and `:717`, and Tenure still drops the same people at `:1187` |

### The same thing has now happened five times

**Property Wise Active Tenants has also lost each row's share**, `tenantService.ts:1227`. That is
every ranked list in the suite: Dues `:711`, Collection `:812`, Expense `:576`, Occupancy `:832`,
Tenant `:1227`. Five blocks, one removal, all commented rather than deleted. One question to Vivek
settles all five.

---

## The guide's own fixes, G1 to G15, checked 2026-08-27

The read-only copy in this repo was taken from commit `a9e5ab994` on 22 August. The source has moved
**seven commits and 192 lines** since, so these fifteen were checked against the live file in the
backend, `docs/analytics/ANALYTICS_GUIDE.md` at `origin/master`, not against our copy.

**One fixed, one part done, thirteen not started.** The 192 new lines are new content, mostly the
vacancy and inventory sections, not these corrections.

| Fix | Outcome | The guide today |
|---|---|---|
| G1 Upcoming Dues is written last; it is sixth on the screen | Part done | No longer last. It now sits at `:115`, after Deposit Dues at `:100`, so the two are simply swapped against the screen |
| G2 The status glossary lists tenant statuses 0 to 3 only | Not started | `:28` still stops at 3. Status 4, invite, and 5 to 8, the deleted states, are still missing |
| G3 The guide contradicts itself on Current FY | Not started, and it now needs rewriting rather than fixing | `:21` says "1 Apr → today" and `:57` says "within this financial year". **D33, ruled today, settles it the other way from D2**: the tile shows the full financial year and the chip compares the year so far. So `:57` is now the correct line and `:21` is the one that has to change, together with a sentence saying the chip's window differs from the tile's on purpose |
| G4 "Food/Others appear only when configured" is not true of the build | Not started | Still at `:119`, and still describes a card that ships rent only |
| G5 Settlement Pending marked "(Live)"; Collection Overview called "Period, with two live tiles" | Not started | `:144` and `:138`, both unchanged |
| G6 The Collection filter list still names Coming up | Not started | `:133` still lists it |
| G7 Expense Overview called "Period, with one live tile" | Not started | `:201` unchanged. Two of the four tiles ignore the filter |
| G8 The Expense View all sheet's rows are never described | Not started | There is still no View all section anywhere in the guide. A support reader asked where "Paid from staff's own money" comes from still has nothing to read |
| G9 "the window's upper end never goes past today" is true of five blocks and not the trend | Not started | `:198` still states it as a whole-tab assumption, and the trend still runs to month end |
| G10 The Occupancy losing-money section claims a group-average fallback the query does not have | **Fixed** | `:313` and `:314` now say the loss is priced at "that room's own tenants' average" and stop there, and `:322` describes the coverage line the card actually ships. The guide now matches the code. The underlying defect, F78, is still open: an empty room is still priced at zero. The guide is now honest about a number that is still wrong |
| G11 The Occupancy filter list names Coming up | Not started | `:238`, `:244` and `:250` |
| G12 The period rate and the trend disagree about the same month, and the guide never says so | Not started | `:253` still describes the approximation without saying the two blocks therefore differ |
| G13 The Tenant filter line ends "(+ Coming up for forward blocks)" | Not started | `:340` |
| G14 Tenant Overview lists "Active / Approved Bookings" among the live tiles | Not started | `:344`. Approved Bookings is a window number and the guide's own appendix says so |
| G15 Stayed-after-notice described as coming from `tenant_agreement_renewals` | Not started | `:927` still folds all three numbers into that one table. Only Completed and renewed read it |

**And the copy in this repo is stale.** [[03 — Analytics Calculation Guide (Vivek)]] is a 22 August
snapshot of a file that has moved seven times since. Re-copying it is a mechanical job and should
happen before anyone reads it again.

---

## The seven items waiting on the owner

None of these is a code question, so none could move on its own. All seven are still open.

| Item | The question | Entry |
|---|---|---|
| Breakup by Stay Duration | Hide when there are no short-term tenants, or no short-term dues | F23 |
| Forward window | What an event count shows on a window that straddles today | D9 |
| Approved Bookings | What the tile shows if auto-accepted bookings leave no trace, once Vivek answers | F93 |
| Added By | Name creator codes 5 and 6, once Vivek says what they are | F26 |
| Agreement length | Whether the product starts asking for it at move-in, so the gap stops reopening | S7 |
| Paying staff back | Whether a way to record it ships before the Still owed to staff tile does | S6 |
| The word "Others" | Whether the rollup row is renamed across Dues, Collection and Expense together | S5 |

Two of these are blocked twice over: Approved Bookings and Added By each wait on an answer from
Vivek before the owner can rule, and neither answer has arrived.

---

## What the whole run adds up to

**96 rows checked against the code, plus 15 guide fixes.**

| Outcome | Rows | Dues | Collection | Expense | Occupancy | Tenant |
|---|---|---|---|---|---|---|
| Fixed | 11 | 1 | 2 | 3 | 3 | 2 |
| Part done | 15 | 1 | 2 | 2 | 4 | 6 |
| Not started | 64 | 16 | 12 | 8 | 13 | 15 |
| Withdrawn by today's ruling | 2 | 2 | | | | |
| Waiting on an answer or a ruling | 4 | 2 | 2 | | | |
| **Total** | **96** | **22** | **18** | **13** | **20** | **23** |

The fifteen guide fixes sit outside that table: one fixed, one part done, thirteen not started.

Eleven of ninety-six is not a story of nothing happening. Vivek shipped 27 commits and about 2,250
lines in five days, and most of that work went somewhere else: a new booking-grain query on
Occupancy, a real column behind Renting Type, colours, icons, selfies, design QA. The fix list and
the work queue have been running on separate tracks.

**The four things worth acting on before anything else.**

1. **One fix closes 51 places and it is already done.** `service.ts:161` to `:175` stops a built block
   falling back to demo numbers when a property will not resolve. It was done exactly where the item
   asked, once, in the shared function. Nothing else in this run comes close for leverage.
2. **The next one closes five.** F47's chip window is the same two lines in all five services,
   unchanged in all five. It is a small, mechanical, high-value fix, and every chip on the suite is
   wrong for three days a month until it lands.
3. **A ruling has been reversed.** Upcoming Vacancy no longer follows the view toggle, and D23 ruled
   that it must. The sheet has already been rewritten around the old behaviour. This needs the owner's
   word before anything else happens on that card.
4. **Two defects look like a fix and are not one.** The Expense quarantine row wraps `Math.abs`
   around a helper that has already floored the negative to zero, so it still always reads ₹0.
   Occupancy's Overview carries `showChip ? null : null`, a ternary with the same answer on both
   sides. **Neither is a recent attempt**: both predate this review, and F55 and F65 described them
   accurately at the time. They belong together because they share a shape, code sitting one layer
   outside the thing it needs to change, which reads as handled to anyone skimming.

**And one thing to ask Vivek before filing anything.** Every ranked list in the suite has had its
per-row share of the account total commented out: Dues, Collection, Expense, Occupancy and Tenant,
five blocks, same shape, all commented rather than deleted. That is a decision somebody made, not
five accidents. One question settles whether it goes back.
