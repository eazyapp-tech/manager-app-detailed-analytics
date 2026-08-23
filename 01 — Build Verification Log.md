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
says so rather than quietly replacing it.

---

## What is in here

| Section | Answers |
|---|---|
| [How this review works](#how-this-review-works) | What gets compared, what the verdicts mean, the words used throughout |
| [What is in scope](#what-is-in-scope) | Which blocks are checked, and what is deliberately left out |
| [Status board](#status-board) | How far the review has got, and the sweeps every remaining tab gets |
| [Open items](#open-items) | Everything still waiting, by who it waits on |
| [Findings](#findings) | Every difference found, F1 to F46: what is wrong and the fix |
| [Decisions ruled](#decisions-ruled) | What the owner decided and why, D1 to D15, dated |
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
| Vivek's calculation guide | What Vivek says the code does. Owned by him in the backend repo at `docs/analytics/ANALYTICS_GUIDE.md`; a read-only copy sits here as [[03 — Analytics Calculation Guide (Vivek)]] with the commit it was taken from. The Google Docs export circulating as a file is lossy, it ate the `$` from every SQL placeholder in the appendix, so never cite the export |
| `src/v1/analytics/` in rentok-backend | What the code actually does |

Doc against doc would pass a block where both documents agree and the code does neither, so every
block is checked all three ways.

**The guide has two halves and both count.** Its prose describes each block; its appendix, "the actual
queries", restates the SQL per service. The Dues pass first read only the prose, and the appendix was
read afterwards, on 2026-08-24, when the owner asked whether it had been used. It overturned nothing,
independently corroborated F2's package defect, and its pointer to `i.added_by` led to F26. **Read
both halves per tab.** Where a claim depends on live data it is measured on production,
test and deleted properties excluded, and the query's scope is stated next to the number.

**A difference is not automatically a defect.** Some were decided during the build with the owner in
the room. So each difference gets a view on which version is right, and the owner rules.

**Where the reasoning lives.** An F entry says what is wrong and the fix. A D entry says what was
ruled and why. Reasoning is written once, in the D entry, and the F entry points to it.

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
| Collection | 7, plus the View all sheet | All | 20 | Complete; fixes with Vivek |
| Expense | 5 | 0 | 0 | Not started |
| Occupancy | 8 | 0 | 0 | Not started |
| Tenant | 14 | 0 | 0 | Not started |

**Next:** Expense. All five Collection rulings are done, D11 to D15; the build fixes sit with Vivek
in the open items.

**Sweeps to run on every remaining tab**, each born from a Dues finding:

| Sweep | Born from |
|---|---|
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

---

## Open items

Everything still waiting, by who it waits on. An item leaves this table when its entry is Closed.

**Waits on Vivek**

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
| Payment Settlement | Destination rows from the payout and wallet records too | F42 |
| Payment Settlement | Say what payout status 1 and a wallet UTR each mean; mark reversed and failed transfers | F43 |
| Collection View all | Window rows and stay rows follow the toggle; Adjusted rows include discount | F44, F45 |
| Collection healthy states | Emit the three §15 healthy messages | F46 |

**Waits on the owner**

| Item | The question | Entry |
|---|---|---|
| Breakup by Stay Duration | Hide when there are no short-term tenants, or no short-term dues | F23 |
| Forward window | What an event count shows on a window that straddles today | D9 |
| Occupancy | What an unapproved booking's bed reads as | F4 |
| Under notice rename | Whether the Tenants label becomes Under eviction with two named parts | S3 in the register |
| Added By | Name creator codes 5 and 6, once Vivek says what they are | F26 |

**Waits on the other sheets**

| Sheet | Change | When |
|---|---|---|
| DA-08 Inventory, DA-09 Tenants | §4: drop Coming up, rewrite the forward column per D9, from each tab's code | At each tab's review |
| DA-08 Inventory | What its under-notice count is called after D6, and the Occupancy booking question | At the Occupancy review |

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

**Fix:** sum the credits rows joined to their payments, windowed by `p.paid_date` like the other
three rows. The View all sheet's Adjusted rows need the same source once F45 adds discount there.

### F41. Collection by Property follows the toggle, and the sheet says it does not change

**Verdict:** Accepted change · **Status:** Closed · **Owed by:** Nobody · **Note:** D15; sheet §4 and §10 corrected

Sheet §4's card table says the card does not change in Due Date view, and §10 says "the same in both
views". The build switches with the view: money arrived per property in Paid Date, collected and
adjusted against the window's bills per property in Due Date (`collectionService.ts:719`).

Ruled as built: D15 has the reasoning. Sheet §4, §10 and test §17.1 corrected.

### F42. The destination rows cover under 3% of settled money

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek

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

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek · **Note:** rewrites §18's open item 1, the day-one worry inverted

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

**Verdict:** Build gap · **Status:** Open · **Owed by:** Vivek

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
production 1,895 bookings sit unapproved against 28 approved. The owner's standing rule is that
current data grounds a decision and never makes it, and he held the ruling. The biller's behaviour
became S1 in the register.

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

---

## Sheet edits made

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

---

## What shipped better than we specified

| # | What | Why it matters |
|---|---|---|
| B1 | The Dues base pool is not a fresh query. It calls the homescreen's own `DuesListHelper.buildBaseQuery` | Dues on the homescreen and Dues in analytics cannot drift apart. The sheet never asked for this. Worth writing into DA-01 §3 as a rule the next screen inherits |
| B2 | The View all screen's code refuses to build a chip the design draws on Past Dues, with the comment "Design's chip on Past Dues is a flagged bug (§21.9)" | The sheet caught a design defect, the developer read the sheet and honoured it over the drawn file. The handoff process working as intended, and worth saying to the team |

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
