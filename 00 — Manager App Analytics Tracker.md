# Manager App Analytics — Method and History

**This file stopped being a tracker on 27 August 2026.** It holds two things now: the rules a handoff
sheet is written to, R1 to R100, and the record of what each screen close actually produced. It holds
no live state at all.

It keeps the number `00` and the word Tracker in its filename because that is how every other document
links to it. The name in the path is history; the heading above is what the file is.

## Where the live state went

State used to live here and in two other places at once, so it drifted. On 27 August 2026 the owner
ruled it into one place each. Nothing was deleted without being written down somewhere it can be acted
on.

| What | Where it lives now | Why there |
|---|---|---|
| Which screen is at which version | The sheet's own frontmatter, with [the README](README.md#the-screens) as the index that copies it | The sheet is the thing that has a version. This file carried a third copy and was wrong on three of six |
| What is owed on the built code | [The work queue in 01](01%20—%20Build%20Verification%20Log.md#the-work-queue) | Every row there says what is wrong, what to do and which file, and carries the commit it was checked against |
| Questions only the owner can answer | [What is waiting on you, in 01](01%20—%20Build%20Verification%20Log.md#what-is-waiting-on-you) | One queue the owner reads, rather than three lists in three files |
| Product problems that are not sheet failures | [02 Suggestions Register](02%20—%20Suggestions%20Register.md) | Its own charter, written before this split |
| Which screen gets written next | [The README's screens table](README.md#the-screens) | The one table a reader already opens to find a sheet |

**What this file kept:** the rules, the method, and what each session produced. All of it is either
permanent or dated history, so none of it can go stale in the way a status column does.

---

## What is in here

| Section | Answers |
|---|---|
| [Where the live state went](#where-the-live-state-went) | What moved on 27 August 2026, and to which file |
| [How to cite a rule](#how-to-cite-a-rule) | What an R number is for, and how to add one |
| [The rules](#the-rules) | R1 to R100, grouped by subject, each one citable on its own |
| [The sources, and which to trust](#the-sources-and-which-to-trust) | The dev sheet, the legacy module, the design file, and the traps in each |
| [What the next screen inherits](#what-the-next-screen-inherits) | What whoever writes Old Tenants should know before Phase 0 |
| [What each screen close produced](#what-each-screen-close-produced) | Per screen: the prior art, what was found, what it cost |
| [Scope decisions about the suite itself](#scope-decisions-about-the-suite-itself) | Which screens exist, which do not, and why |
| [What was still open when Tenants closed](#what-was-still-open-when-tenants-closed) | A dated snapshot, deliberately not re-checked since |
| [Session lineage](#session-lineage) | Every session, oldest first, and what it produced |
| [What was corrected in this rewrite](#what-was-corrected-in-this-rewrite) | The stale claims this file was carrying, and what is true instead |

---

## How to cite a rule

Every rule below carries an **R number**. The number exists so that a finding, a sheet or a later
session can contradict one rule by name instead of quoting a paragraph. It is a pointer to the
reasoning and never the carrier of the meaning: write "R29's one-set-of-options rule for a card's own
dropdown", never "per R29".

Numbers are permanent. A rule that is replaced keeps its number and gains a line saying what replaced
it, the way R90 replaced the old shared-permission line. New rules take the next free number, which is
**R101**.

---

## The rules

These apply to **every** screen. Inherit them, do not re-ask them.

### How a sheet is written

The document states **what should be built**, not how it was decided. This is a hard standard, not a
preference: a developer reading it should not have to skip past reasoning to reach the spec.

**R1. No process archaeology.** No "locked this session", "checked against the dev sheet", "an earlier
pass assumed", "independently confirmed by".

**R2. No cross-document references.** Not "per the Build Sheet", not "the Formula Map says". If a fact
matters, state the fact. A handoff sheet that only works with four other documents open is not a
handoff sheet.

**R3. No code, and internal numeric codes count as code.** Say "a bill cleared using the tenant's
deposit", never the mode number behind it.

**R4. Design-file problems go in short, clearly marked side notes**, not woven into the definitions.
The definitions outlive the Figma bugs. Collect every side note once more in a single list at the end,
so design has one place to work from.

**R5. Plain punctuation, no em dashes** (locked 2026-08-07). Commas, colons, full stops. The dash
survives only as the empty-table-cell placeholder and in document titles. No AI-tell vocabulary.

**R6. Every new sheet starts as a copy of [[_Handoff Sheet Template]]** (locked 2026-08-07). Never a
blank page, never memory of the format. The template is the spine, the sub-section names, the table
schemas and the fixed phrases in one place. This exists because standards recalled during audit
produced an incremental correction loop across three screens, and standards copied at draft time
cannot drift.

**R7. One grammar across the suite** (locked 2026-08-07). Same spine and section names, same table
schemas with identical column headers (tap matrix: `You tap | What opens | Arriving filtered to |
Ready?` · definitions and View all: `Row | Meaning` · number kinds: `Live` / `Time-scoped`, never a new
coinage), same fixed phrases ("Test it:", "as of today", "from today onwards", "hides at zero", the
Restricted copy). Tables wherever content is parallel. A sheet may drop what does not apply; it never
renames or reorders what it keeps. **Two names settled 2026-08-07:** the access section is **"Who can
see this"** everywhere, and the states section is **"What each card shows when it is empty, healthy or
broken"**. The filter-behaviour grid is titled **"What every number does on every filter setting"** on
every sheet. Collection's old "What is already broken in the code" section is gone as of v13 and is
never copied forward.

**R8. The reader test** (owner-stated 2026-08-08, the acceptance bar over every writing rule): a
developer or designer reads the sheet alone and builds from it without asking the PM. Words that fail
it get replaced even when technically correct. "Partition" failed it on Dues, because in a PG a
partition is a plywood wall.

**R9. One time vocabulary, cell level included** (locked 2026-08-08, owner-directed after Dues
drifted). The grammar locked section names and table headers but never the words inside cells, so
every new layer became fresh surface for coinages. "Follows", "pinned window" and "duration-scoped"
all slipped through a grammar that already said "never a new coinage". The fix is a closed list plus a
mechanical sweep, not better recall. **Allowed grid-cell phrases:** "As of today" · "Counted inside the
window" · "Each tile keeps its own fixed window; the filter does not change it" · "From today onwards" ·
"—". **Allowed kind labels:** Live · Time-scoped (", window fixed" where pinned) · Forecast. **Banned,
and grep-swept at every close-out:** follows, pinned, window-scoped, duration-scoped, period-scoped,
any new kind word. Write cells the way you would say them aloud to someone new.

**R10. GitHub readability** (locked 2026-08-08, from Dues v11). Every sheet's contents table links each
section name to its own heading anchor. The tap matrix, the design-fix list and the measured-figures
appendix each collapse behind a GitHub `<details><summary>` block, since each is read start to end by
one audience only. Section 3's simple one-line terms (no rule, no exclusion, no cross-reference) move
into a quick-terms table at the top of the section; anything carrying a real rule keeps its own
heading and prose below the table, never folded in just to shorten the section.

**R11. Every sheet carries a tap matrix** (locked 2026-08-07, same rank as the time-filter grid). One
row per tappable element, grouped by card: what opens, the filters already applied on arrival in plain
words, and a Ready? verdict. The destination varies with the record's state (arrivals open the
Bookings list, departures the Old Tenants list), so "opens a list" without naming which one and how
filtered is not a spec.

**R12. What travels with a tap depends on the window** (locked 2026-08-07, part of R11). Live taps
carry nothing: the list opens as of today. Period taps carry the window, and the list shows those
people as they are today, naming any difference on arrival ("showing 11 of the 14; the rest have since
moved out"), because a count of events and a list of current people drift apart on any window not
ending today. Forward-window taps re-route to where the people are today: scheduled arrivals open the
Bookings list, scheduled departures the Tenants list. Records-add-back holds absolutely only on
windows that include today.

The next five are the behaviour contracts, locked suite-wide 2026-08-07, found by walking the seams
after the tap-matrix work. Each is one to three lines on a sheet. None of them had ever been ruled,
and every one was a question a builder would otherwise have had to ask.

**R13. Custom stops at today.** The future belongs to the forward setting. One question never has two
answer models.

**R14. The filter stays where the manager put it while the app is open.** A fresh launch opens on the
default.

**R15. Coming back from a drill returns to the screen as it was left, with fresh numbers.** Close a
record, return, the tile has moved.

**R16. On Custom, chips compare the same number of days immediately before the range.**

**R17. A day runs midnight to midnight, India time.**

**R18. Every screen whose records carry future dates gets the forward setting** (locked 2026-08-08,
from the Dues backfill). R13 already required the future to belong to a forward setting, and Dues was
the one screen with none to hand it to: 8,703 bills worth ₹28.5 crore were already raised with due
dates more than a week out, and no period setting could reach them. **Coming up** is the name on every
screen. Two things differ by screen and must be said per sheet: whether the forward numbers are
projections (Tenants, Inventory: state at a future date, confirmed only, no guesses) or plain records
(Dues: bills already raised with a future due date, facts not estimates), and which cards sit the
setting out because their question is about the past (on Dues, the billed-versus-collected pair).
**Trap, verified in the shared date control:** the presets labelled "Next 7 days" and "Next 30 days"
resolve backward, to the *last* 7 and 30 days. Any sheet specifying a forward option carries the
outcome-plus-test for it.

**R19. The forward threshold is by-design, not exists-at-all** (carved out on Expense 2026-08-08). A
screen gets Coming up when its records carry future dates *by design*, the way bills do. Stray future
dates do not qualify: Expense holds 45 future-dated rows out of 340,524, data-entry artifacts, and
gets no forward setting. Its Custom stops at today and a stray joins totals when its date arrives.

**R20. Every sheet routes its zeros** (locked 2026-08-07, extending Inventory's healthy-state
discovery). One table tells the empty-looking situations apart on sight: never-set-up, **onboarding**
(bookings confirmed and nobody moved in: real zeros plus one quiet line, not the setup screen), **the
emptied property** (a true and alarming zero, never dressed as setup), the in-window zero with its
neutral chip, the good-news zero with healthy wording, the not-recorded gap with its coverage line,
and failure. Distinguishing good zero from bad zero from no-data-yet is per number, not per screen.

**R21. Drill reachability is read from the apps' shared filter drawer, not from the backend** (locked
2026-08-07). Both manager apps use one filter set, and that file is the truth for what a manager can
reach. On Tenants a late app-side check flipped nine tap-matrix verdicts in both directions: eviction
pending and approved, leaving-date windows with a custom range, and the already-expired agreement
window all exist as filters, while true disjoint bands (8 to 15 days) and valid-beyond-90 do not.
Corollary for the cards: **the lists filter by within-N windows, the cards draw disjoint bands**, so a
band drill opens the nearest window and says so until band filters exist.

**R22. A booking has two states, confirmed and awaiting confirmation, and both are bookings** (owner,
2026-08-07). Only cancelling ends one, and **a cancelled booking lands on the Old Tenants list** even
though the person never lived there. Consequences: booking counts show both states with awaiting as a
layer; Inventory's booked-bed layer stays confirmed-only, so the two screens meet through the
confirmed count; Move-outs never counts a cancelled booking. The Old Tenants screen must separate real
departures from cancelled bookings or its headline will overstate departures.

**R23. Lean is a number, not a feeling** (locked 2026-08-07, after Tenants v1 ran to 12,400 words with
54 warnings and 81 platform statistics in the body). Budget per sheet: warnings under ten, platform
figures only in the measured-figures appendix, reasoning in this file, definitions in one or two
lines. Tenants v2 landed at 5,500 words against Expense's 8,100, and that is the bar. After any full
rewrite, re-run the decision ledger against the new text.

**R24. Mark anything genuinely unsettled as a recommendation or an open question.** Do not blur that
with what is decided.

### Time and filters

**R25. Two kinds of section.** **Live** is always the current snapshot and no filter changes it.
**Time-scoped** shows a window.

**R26. One global filter per screen**, with five options: **All Time, This Month (default), Last
Month, Current FY, Custom.** Changing it updates every time-scoped section. A card with its own local
filter overrides the global one **for that card only**.

**R27. Naming is locked.** Where the backend says "Live" and "This Year", use "All Time" and "Current
FY". Read R43 before applying this on a state screen, where the two words mean opposite things.

**R28. Not every screen has a Live section.** Collection has none. Check per screen.

**R29. A card's own dropdown offers the same options as the filter at the top.** No per-card subsets,
or managers hit "why can't I pick Current FY here?" with no good answer.

**R30. Changing the filter at the top pulls every card back into line.** Any card on its own period
snaps back. A manager can set one card aside deliberately, but can never end up on a screen showing
four different periods without meaning to.

**R31. Trend-style charts are exempt from the period filter entirely.** Every normal card answers
"which period?", one window, one set of numbers. A trend answers "how far back?" and needs several
periods to say anything at all. A trend following a This Month filter would show a single bar. Give it
its own range control, and let the view toggle still apply to it even though the filter does not.

**R32. Never compare an unfinished period against a finished one.** The default is This Month, so most
of the time a manager is looking at a period still running. On the 3rd, that is three days. A change
chip comparing three days against a whole finished month shows a collapse that is not real, on the
default view, for the first week of every month, on every property. Compare against **the same point
in the previous period** and say so: *"▲ 12% vs same point last month."* The note drops away once the
period completes.

**R33. An unfinished period is always marked as unfinished** wherever it appears: the change chip, the
in-progress bar on a trend chart, anywhere else. One principle, not a per-component decision.

**R34. Future periods are allowed.** They are meaningful on any screen with a bill-side view, since
next period's bills already exist. Where a future period genuinely has nothing to show, say why in one
line and offer a way out rather than showing a blank. Collection's version: *"Nothing received yet.
Collection counts money as it arrives. Switch to Due Date to see what's billed for this period."* This
fires only when the **whole** period is in the future. A period straddling today has real data and
behaves normally.

**R35. All Time has no previous period, so change chips disappear there** (locked on Expense
2026-08-07). Every screen offers All Time and every screen puts a change chip on its headline numbers.
There is nothing before "everything". A chip comparing all history against a period that does not
exist is blank at best and invented at worst. Hide it whenever the window is All Time, whether that
came from the top filter or a card's own dropdown, and draw the tile state that has no chip.

**R36. A fixed-period card that duplicates a filter option stays visible** (locked 2026-08-07). Current
FY is both an option on the top filter and a fixed tile. Select it and two adjacent tiles show the
same number. Find this on every screen that has both, and **rule the same way every time: the tile
stays.** Hiding it shifts every tile after it under the manager's thumb at the exact moment they
change filters, and a row that reshuffles is worse than a number that repeats. The tile earns its
place at the other settings by putting the year next to the month. This was decided on Collection and
re-decided the opposite way on Expense before the two were compared. **Screens one tab apart cannot
answer the same question differently**, which is why R87 exists.

**R37. A screen may add a filter option its numbers call for; it never removes one of the five**
(owner, 2026-08-18, from Complaints). Complaints keeps Last 7 days beside the suite's five, because
complaints move in days where money moves in months and half are resolved inside two. Inventory added
Now, Tenants dropped it, so the precedent already existed. The sheet says why in its section 4, and
the shared date control does not gain the option on any other screen.

**R38. A number is either a today number or a window number, and the two never mix silently** (locked
on Tenants 2026-08-07). A **today number** answers *how many are there now*: it ignores the filter and
says "as of today" on its face. A **window number** answers *how many happened*: it follows the
filter. Cards holding both label **per number, not per card**. Only window numbers carry a change
chip, and nothing carries one on All Time (R35). **Before writing any filter behaviour, sort every
number on the screen into these two piles.** Inventory never needed this because every number there
was a today number. **And the filter options follow from the mix:** Inventory rests on Now because it
describes space; Tenants has no Now setting at all, because it is a screen about movement and four of
its seven tiles would have nothing to count.

**R39. Run the time matrix as a grid, not as a set of rules** (locked on Expense 2026-08-07). Stating
the filter rules is not the same as walking every number through every option. Expense stated the
rules, grilled the biggest question on that axis, and **still shipped a duplicate tile** that only
turned up in the final audit, plus two more gaps that only appeared when the grid was actually drawn.
Write the grid: every number down the side, every filter option across the top, and fill each cell.
The cells that are not simply "the same as the filter" are where the work is.

### Cards, states and zeros

**R40. An empty card is not always bad news** (locked on Inventory 2026-08-07). Every screen before
Inventory treated an empty card as a gap: zero collection, zero spend, nothing to show. On Inventory
an empty card is usually success, because no vacant rooms means the property is full. The suite
therefore needs a state nobody had defined, **healthy**, distinct from **empty**, plus, where a whole
screen can be empty for one reason, a single whole-screen state instead of N boxes each explaining
themselves.

**R41. An empty state that means the feature is not in use gets the obvious next step; one that means
success gets nothing** (clarified 2026-08-18, from Complaints). Inventory's healthy states carry no
button because there is nothing to do: a full house is success. Its whole-screen not-set-up state
carries one ("Let's set up my property") because setup is the next step. A screen with no complaints
almost never means perfection, it means the feature is not in use, so the whole-screen state carries
**Add Complaint** and reads plainly, never triumphantly.

**R42. Change-chip polarity can be per tile, not per screen** (locked on Inventory 2026-08-07). Dues,
Collection and Expense each settle direction once for the whole screen, "up is bad here". Inventory
cannot: occupied rising is good, vacant rising is bad, and rentable rising is neither. It needs three
behaviours in one row, including a **neutral grey chip the shared component does not have**. Treat
polarity as a per-number property, and treat the shared chip component as carrying work whenever a new
screen lands.

**R43. "Live" is already taken, and it means filter-immune** (locked on Inventory 2026-08-07). Dues
defines Live sections as always the current real-time snapshot that no filter ever changes, and
Expense's design fixes remove a "(Live)" label for the same reason. Inventory needed a word for the
*present-moment setting of the filter* and reached for the same one, which would have given it two
opposite meanings on adjacent tabs. **The word on a filter chip meaning "right now" is Now.** Live
stays reserved for sections no filter touches. Related, and the reason this surfaced: **R27's mapping
of the backend's "Live" to "All Time" does not hold on every screen.** On the money screens the two
mean the same thing, no date limit. On Inventory they are opposites: all-time occupancy is meaningless
and the present moment is the most-used setting. **Check what the word means on the screen before
applying the mapping.**

**R44. Every card loads, fails and retries on its own** (locked on Collection 2026-08-06). The screen
never waits for its slowest card. Cards are not equally fast: simple sums return at once, while trend
charts and joined breakdowns can take seconds, and All Time on an old property makes that worse. A
manager should not watch a blank page while a chart they may never scroll to finishes.

**R45. Skeletons match the shape of the card they become**, so nothing jumps position as the page
fills in. This is what makes independent loading feel calm rather than restless.

**R46. A failed card never renders a number.** Zero is real, meaningful data on every one of these
screens, so it can never double as "we do not know". A settlement card that failed and showed zeros
would send a manager to support about a failure that never happened.

**R47. Empty and failed must look different**, because a manager acts differently on each. Empty means
the query worked and there is nothing there, names what is missing, and offers no retry. Failed means
the query did not work, says only *"Couldn't load this,"* and offers a Retry. Error copy stays boring:
no apology, no cause, no codes.

**R48. A failed card keeps its heading and shrinks** to its message and Retry, rather than holding a
card-sized hole. **Retry refetches only that card**, never the screen.

**R49. When every card fails it is the connection, not the cards.** Replace them all with one message
and one Retry: *"Couldn't load this page. Check your connection."* Some cards failing stays per card.

**R50. Every property-wise component across all analytics screens sorts highest to lowest.**
Placeholder data in Figma often does not demonstrate it, and that is not a contradiction.

**R51. The entry point for each tab is the widget at the top of that tab.** Screens shown around it in
the design file are reference context only. **Not every screen has a single big hero number.** Dues
does; Collection does not, and its entry point is a tile row. This matters beyond layout: edge states
(negative totals, zero-activity periods, limited-view viewers) were all designed to live on a hero
card and need a new home when there is not one.

**R52. Empty-state tone.** Use the Defaulter widget's copy as the reference: short factual title plus
one reassuring line. A call to action only where there is a genuine next step.

**R53. Excluded records get exactly one home.** Where a category of record is deliberately excluded
from a screen's totals, it gets **one explicit row** where it does appear, not scattered, not hidden.
Established with moved-out tenants on Dues. The same principle covers unlinked payments on Collection:
they stay in the totals and get a visible row, or the card says plainly they are excluded there.

**R54. A state that describes the future is a layer, never a slice** (locked on Inventory 2026-08-07,
confirmed on Tenants). **A booked-but-empty bed is still vacant.** A booking is a promise, not a
person: until someone actually moves in the space is empty and earning nothing, and a property that
counted arrivals as occupancy could report itself full on the strength of move-ins that never happen.
So booked is reported as a **layer inside** whatever the bed's present state is, never as a slice
beside it. A booking on an empty bed sits inside **Vacant**, which then splits into booked and
available. A booking behind a tenant whose eviction is under way sits inside **Occupied**, as the
already-replaced line. **Available is the number a manager acts on**, so it is always stated rather
than left to subtraction. Three problems dissolved when this landed: the donut arithmetic closed
(occupied + semi-vacant + vacant = rentable, + disabled = total) where booked-as-a-slice had
double-counted every already-replaced bed; the vacant drill became fully reachable again; and the
vacancy-age card stopped disagreeing with the tiles. One exception proves it: bookings with no bed
allocated at all have nothing to be a layer on, so they stay their own row, and the sheet says why.
**Confirmed on a second screen:** a tenant under an eviction is still active, and an approved eviction
is still active. Both are layers inside Active, never slices out of it. That also settled a question
open since DA-02, because Dues and Collection never actually disagreed: both were describing the same
thing from two ends.

**R55. The layer follows the bed, never the room around it.** A booked bed inside a partly-filled unit
still counts in the booked layer, and the unit stays semi-vacant. No new state.

**R56. A flat is one unit only while a single tenancy holds all of its rooms.** One bedroom let
separately makes it room-wise, including the empty bedrooms. The owner's form of the argument is the
keeper: *if the tenant were taking the whole flat, they would be recorded against the whole flat;
holding one room is itself the answer.* One consequence found a round later: **the rooms list
disagrees with the layer model in both directions**, treating a booked bed as taken (so Vacant drills
come back short) and counting unconfirmed bookings as bookings (so booked numbers over-return there).
One meaning of vacant and booked across the app is a logged requirement; until it lands, the sheet
routes drills to wherever each number can be honoured and says so.

**R57. How many rows before Others is decided per screen, from how the categories spread** (locked
2026-08-18, from Complaints). The suite had six numbered rollup cards using three, one using four,
five stating no number at all, and only one measured reason anywhere (Collection: median two receivers
per property, so three plus Others fits). Complaints measured its own spread and landed on **four**,
because complaints spread wider across categories than money does across bill types, so three would
have left Others holding half of everything. **The count is set from the measured spread on that
screen and stated on the sheet with the figure behind it.** A fixed, closed set of values shows every
one and never rolls up, the way Expense's payment modes do.

**R58. Others rows are two steps.** Tapping Others opens a sheet with the full remainder; the drill
happens from a row inside it.

**R59. Every card's info icon carries written content in the screen's own plain language, or the icon
comes off.** An icon that opens nothing is worse than no icon. Logged on Expense 2026-08-07 as a
content deliverable nobody owned; **written and shipped by 27 August 2026**, 115 entries across the
five built tabs (Dues 25, Collection 29, Expense 10, Inventory 21, Tenants 30). The rule stays because
every new screen inherits the obligation.

### Taps and drills

**R60. Every breakdown widget opens the exact filtered list behind the number tapped.** The one
exception is forecast numbers, bills not yet raised, which have no records to open and so open an
explainer sheet.

**R61. A drill filters a list. It never re-scopes the screen.** Verified in code: the list endpoints
already take property, date range, payment mode, due type, received-by and tenant type as filters, so
a filtered list is the pattern the whole app uses. A card that instead narrowed the whole screen would
be an interaction existing nowhere else, and managers would have to learn it for one row type.

**R62. A number opens the screen that owns the kind of record it describes.** A number about payments
opens a payments list. A number about bills opens Dues, even when it sits on the Collection screen. On
Collection that means three numbers leave the screen: Billed, Still Unpaid, and the trend chart's Due
segment.

**R63. Cross-screen drills carry two things:** the destination opens on **the same period**, and the
back control **names where the person came from** ("Collection", not a bare arrow).

**R64. A drill from a trend chart moves the period rather than narrowing it**, because the chart has
its own range. That is the only place this happens, and it should stay that way.

**R65. Check the destination can express the filter before promising the drill.** Naming the
destination is not enough. On Expense, three whole families of drill (bill coverage, payment mode,
fund source) point at a list that has no such filter, so the number could never open its own records.
**Enumerate the destination's real filters and match them against the drill list, one by one**, before
calling drills done.

**R66. When the drill check fails, say which kind of failure it is.** The two answers look identical
in a drill table and are weeks apart in effort. On **Inventory**, nothing records when a room became
empty, so a fact has to start being recorded and whole cards are blocked: never-rented is 87 to 90% of
all vacant inventory and has nowhere to go. On **Tenants**, every date is recorded in full, leaving
dates, notice dates, approval times, and none can be asked for as a range, so a filter has to be added
over data that already exists. **Name which one it is, every time.** This is also why R65 runs before
the drill table is drafted rather than after.

**R67. The destination has to say what it is showing.** A shared dependency, worth doing once rather
than per screen. Checked against the live manager web app's collections list: a manager arriving from
a widget tap sees a page titled "Collections Overview", fixed, whatever filter brought them there.
Chips naming active filters exist but render only after the person scrolls, and filters that come from
tapping a widget produce no chip at all. An empty filtered slice shows one sentence, "There are no
collections present", which reads as "you have no collections at all" and is false for a property that
collected lakhs that month. Any list an analytics screen drills into needs three things: **name the
slice** in place of the fixed title, **name the filter in the empty state** with a way on, and **show
the active filter on arrival** without scrolling.

### Words and meanings

**R68. One word, one meaning, inside a document.** A term that means two things in one doc is a bug
even when every individual sentence is true. On Collection, "settled" meant "bill dealt with" in half
the doc and "money reached the bank" in the other half. Sweep the final doc for its load-carrying
words and check each has exactly one meaning; where the product itself uses one word two ways, say so
explicitly once.

**R69. One word can collide across screens, not just within one doc.** **"Bill" means an invoice
raised to a tenant on Dues and Collection, and a receipt attached to an expense on Expense.** Both are
correct on their own screen and a manager moves between them in seconds. Where a word is already taken
by another screen, say so explicitly and use the longer unambiguous phrase.

**R70. Naming a sibling screen is allowed in two places only** (owner, 2026-08-18, from Complaints).
R2's no-cross-document rule stands everywhere except section 3's Words to be careful with, where "on
Dues a bill is raised to a tenant" is the fact itself, and the design-fix list, where "copied from
Tenants" is the fact itself. Elsewhere, state the fact and not the sheet.

**R71. Never tell a developer their code is wrong, tell them what the operator must get** (locked
2026-08-07). The sheet says **what has to be true for the user**, never what is broken in the code. No
"the existing calculation counts wrongly", no "do not reuse X", no naming of tables, fields or
functions, and no citation of code as evidence. Where something today produces a wrong result, say it
as an **outcome plus a test**:

> ✗ *"The period calculation counts anyone present at any point of the window as present throughout."*
> ✓ *"A tenant present for three days of a month contributes three days. **Test it:** a property that
> emptied on the 5th and refilled on the 25th reports roughly a third full for that month."*

Three reasons this is more correct, not just politer. **A code claim goes stale the day someone edits
the code**, while a statement about what the operator must see stays true for the life of the screen.
**"That function is wrong" is not testable; an outcome is**, and the second version is an acceptance
criterion QA can run. **It protects against the writer being wrong**: stating the requirement and
being told "we already do that" costs nothing, while stating "your code is broken" and having misread
it sends someone to rewrite working code. What stays is scope: "the screen is not built yet" changes
the estimate and belongs in the sheet. What comes out is every characterisation of *how* existing code
misbehaves.

**R72. Write requirements, not derivations.** A recommendation that names a table, a field or a lookup
has stopped being a requirement. State what the user must see; engineering picks how it is computed.
Corrected mid-session by the owner: *"Why are you getting into how it should be calculated? Let the
engineers decide. We only want what we want."* This is R71 applied to computations instead of to
defects. One lens, both cases.

### Sources, prior art and what production tells you

**R73. Read the definitions doc twice.** Prior definitions docs get skimmed in Phase 0 for orientation
and then never re-opened, so whatever did not seem relevant on the first pass stays lost. **Re-read
the definitions doc after drafting, line by line, against the draft**, and walk the section lists both
directions. Why this is a rule: DA-02's definitions doc specified a five-part Collection Status card,
the design drew four, and the draft documented four. The missing part, what is still unpaid, was half
the card's value and had been sitting in a doc already read. **Drafting from the design file inherits
the design file's omissions, and nothing flags the gap.**

**R74. When the handoff supersedes its sources, say so in the sources.** Collection's handoff overrides
its Formula Map and Build Sheet on at least six points: the credit-payment maths, settlement states,
the unpaid row, date assignments, category rows, and a fourth adjustment type. Those docs still call
themselves the source of truth, so **whoever reads them next builds the wrong thing.** When a handoff
wins an argument with its sources, mark the losing sections as superseded, or log it as explicit debt.

**R75. Pay the superseded-sources debt in the same session.** DA-02 logged it as an open item and it
was never paid; DA-04 paid it in session, with a banner at the top of all three source documents
listing the eight points the handoff overrides. Make this part of closing a screen, not an open item,
or it never happens.

**R76. A frame called stale, superseded or not drawn needs a live re-check, not a recall** (locked
2026-08-09, from Dues' View all sheet). Dues' View all sheet cited its own design frame as an old
superseded draft and wrote the section from scratch instead of from it. The frame was current and
complete, close enough to the doc's own vocabulary to have been documented almost as-is, and the
dismissal let one real bug through unaudited: a change chip on Past Dues, a row the screen's own rule
says carries none. **Any sentence in any sheet that says a design frame is old, stale, superseded or
not yet drawn gets a live Figma check before that sheet is next touched, never a recall of when the
claim was first made.** Cheap to run, and the one time it was not run is the one time it mattered.
Swept across the other four closed sheets 2026-08-09 and found nothing: Collection's "the View all
sheet is not drawn yet" is an honest absence, and every other stale hit is the doc correctly flagging
old copy inside a current frame. Still to sweep on Old Tenants, Bookings and Leads once each has a
design file worth checking.

**R77. Prior art must be searched for by subject, never by naming convention.** A 46KB hardened PRD for
Tenants existed the whole time and Phase 0 missed it, because every check looked for `DA-` prefixed
folders and this one sits in a folder named for the screen. **The convention is the thing a stray
document is most likely to be missing.**

**R78. A stale document is not a wrong document.** That Tenants PRD was six months old, had eight
unresolved blockers, and was still the best statement of why the screen exists. Reconciling it took an
hour; re-deriving what it already knew would have taken a session and would have lost five things the
design does not draw: police-verification deadlines configurable per property, the overdue concept
that follows from them, the legal-versus-operational compliance split, the money-at-risk framing, and
a tenure breakdown as the only preventive number on the screen.

**R79. Empty tables are not evidence.** Three tables that look like they own the Tenants data hold
**zero rows**: the notices table, the lead-status table, the new-bookings table. The real notice
history sits in the eviction records, 119,655 rows, and lead state sits on the tenant record. A near
miss in the other direction too: booking approval looked unrecorded until a second table turned up
holding exactly it. **The first matching artifact is not the governing artifact, in either
direction.**

**R80. Measure a candidate number before specifying it, then judge it as a new feature** (locked on
Expense 2026-08-07). **Query production before writing a number into a spec.** On Expense this killed
three: a paid-date cleanup path (**zero** such rows across 339,732 expenses), a
missing-category/payee/payer section (21 rows out of 151,012), and a permanent petty-cash bucket
(0.35%). All three were specified in a hardened doc and would have been built. **But current data
measures behaviour under a product that never asked.** The same query said 83% of expenses have no
bill attached, and the first call was to cut the number as meaningless. That was wrong, and the owner
caught it: nothing has ever shown an owner that figure, which is *why* it is 83%. **A new number's
test is "would seeing this change what someone does", not "is this common today".** Low numbers can
still be dead cases: the paid-date cut held, because nothing an operator does can create that row.

**R81. Record the figures in the doc.** Whoever reads it next cannot re-check reasoning they cannot
see.

**R82. The snapshot grounds a decision; it never makes it** (locked on Expense, hardened on Inventory
2026-08-07 after being broken again). **A config value is the easiest thing in the system to change
and the easiest to mistake for a constraint.** Inventory's nav config declares the screen live-only.
That was read as the shape of the screen and used to recommend against a time dimension, which would
have shipped a read-only copy of a screen that already exists. The owner reversed it. Then the same
mistake repeated: a Bed View was recommended against because only 1% of properties are on the
structure that makes beds real, when in fact **everyone is migrating to it.** Both errors have the
same shape, **a transitional state read as a permanent property of the system.** Before recommending
against something because "the system does not do that today", ask what it would take to change and
who is already changing it. Where the snapshot and the user's need disagree, grill the owner rather
than silently deferring to either.

**R83. Before specifying a card, check whether the product already has a screen for it** (locked on
Expense 2026-08-07). Expense had a fully designed Team Passbook card. Team Passbook is already a
**live screen** with cash handover, give-money and settle-up, so a read-only analytics copy would have
been a weaker version of something that exists, and its one useful next step is banned on a diagnosis
screen. The pattern that replaced it generalises: **one number on the analytics screen, drilling into
the live screen that owns the subject.** Where half a card feels wrong for a screen, usually the whole
card is.

**R84. Answer it from evidence before asking it.** Three items sat on the Tenants open list as
engineering questions and all three dissolved under a query. Room changes update a tenant in place and
cannot inflate move-ins: 320 tenants in the whole system have more than one room record. The two
leaving dates were never in conflict: they agree in 4,497 cases out of 4,498 and simply mean
*requested* versus *approved*. Booking states did not need a ruling because the homescreen already
computes them both ways. **A question you can answer is not a question to ask.**

**R85. Push once on the answer you did not recommend.** The owner overruled the recommendation twice on
Tenants, and both times a single narrow follow-up made the decision better rather than reversing it:
red-for-missing narrowed to "only where somebody can be held to it", and Leads-as-its-own-screen
forced the useful question of where it lives, landing it inside People in lifecycle order. **Pushback
is not disagreement, it is finishing the decision.**

**R86. When the owner says "I don't understand", the doc is wrong, not the reader.** It happened twice
on Tenants and both times the confusion was real: two different questions had been mashed into one
card. Untangling "does an agreement document exist" from "when does the term end" dissolved a problem
that had taken three rounds. **Confusion is a finding.**

**R87. A number large enough to reshape a card gets its derivation checked before it is put to the
owner, not after.** The "188,568 expired agreements" figure was mostly the system assuming eleven
months for people with no agreement at all. The recommendation was made, approved, and then had to be
withdrawn. The lesson is not "check harder".

### The checks that run before a screen closes

**R88. The final audit, mandatory before calling a screen closed.** Run four independent checks once
the doc is "done". On Collection this pass found about 30 real issues in a doc that had already
survived eleven versions and five review rounds. **One, the decision ledger:** list every decision the
owner made this project (Collection had 45), then check each against the doc mechanically, which
catches silent drops during rewrites. **Two, design-file verification:** re-verify every visual claim
against the actual frames, claim by claim, verified / wrong / cannot tell, which catches everything
asserted from memory or reasoning. **Three, the source-document sweep:** list everything the
definitions docs require that the handoff never mentions, and every contradiction. This was the
largest angle on Collection: 29 dropped items and 10 contradictions, including a third live bug the
doc's own bugs section had missed and a whole non-goals section that had evaporated. **Four, verify
the doc's own rationale claims:** sentences like "because rent is due on the 1st" are claims too, and
Collection's weekly-trend rationale turned out to be a configurable setting, not a fact.

**R89. Audit agents are also wrong sometimes.** One confidently reported the property card sorted when
it is not. Re-check any agent finding that contradicts your own records before accepting it.

**R90. Full rewrites silently drop sections.** Collection's Copy Fixes section vanished in a v9 rewrite
and nobody noticed for three versions. **Diff the section list against the previous version after any
full rewrite.**

**R91. View-toggle audit, mandatory on every screen that has one.** Several screens carry a view toggle
separate from the time filter (Collection: Paid Date / Due Date. Inventory: Bed / Unit. Others
likely). **A toggle can change which sections exist and what surviving sections mean.** Walk every
card through both sides and write out its definition in words on each side. Three failure modes, in
increasing order of danger. **Collapses:** the card splits data by the very dimension the toggle
switches, so on one side it just restates the filter back, one row at 100% and the rest zero. Hide it.
**Goes dead or redundant:** permanent zeros, or a tile becomes an exact duplicate of its neighbour.
Swap it for something that answers what that view is good at. **Silently changes meaning:** looks fine
on both sides, no zeros, no duplicates, but now measures a different thing and nothing says so.
Sibling cards can also stop agreeing, because one can express the new meaning and another structurally
cannot. State the new meaning, and pin siblings to one definition so they still reconcile. The first
two announce themselves. **The third never does**, and only definition-by-definition comparison
catches it. "Unaffected" must be earned per card, never assumed for anything that does not look
broken. Worked example, Collection's toggle, 4 of 7 sections affected: Collection Status *collapsed*
and is hidden, since its split by bill timing can only return 100/0/0/0 in Due Date view; four of six
Overview tiles went *dead*, two duplicates and two permanent zeros, swapped for Billed / Collected /
Still Unpaid; and the trend chart's collection bar and the Breakup tabs both *silently changed
meaning*, so all four tabs had to be pinned to collected money or they would stop adding up. **Missed
twice on DA-02 and caught by the owner both times.** Run it as its own deliberate pass; it will not
surface from reading the design file.

**R92. Check a finished screen against its siblings before closing it** (locked 2026-08-07). A handoff
sheet can be internally perfect and still disagree with the screen next to it. Nothing inside one
document can catch that, and a manager moves between these tabs in seconds. Before closing any screen,
read the already-closed sheets beside it and compare, line by line, on: the change-chip direction and
colours, what a card's own date dropdown does when the top filter changes, fixed-period cards that
duplicate a filter option, what the View all sheet does, whether share-of-total is shown, the
permission wording, the Restricted copy, and the section names themselves. Running this for the first
time, on Expense against Dues and Collection, found five disagreements including **an inverted change
chip on Dues that had been signed off as "confirmed in scope"**, green when dues rise, on a screen
where rising dues are the problem.

**R93. Check the tab row itself, once, for every screen** (found on Inventory 2026-08-07, settled on
Tenants 2026-08-07). The design's tab row and the one the app serves did not match, and nobody had
noticed across four screens. **The system's structure is correct and the design is redrawn to match:**
Financial (Dues · Collection · Expense) · People (Leads · Bookings · Tenants · Old Tenants) · Inventory ·
Complaints. People keeps its grouping rather than flattening, the sub-tabs run in **lifecycle order**
rather than alphabetically, and **Complaints keeps its name**, so the design's "Issues" is dropped.
This unblocked all the documented screens; it had been logged as nobody's since Inventory.

**Not shipped yet, checked live 27 August 2026.** The analytics web build serves three major tabs,
`Financials · Occupancy · Tenant`. There is no People grouping, no Complaints tab, and no Leads,
Bookings or Old Tenants. Two of the three it does serve are labelled differently from their own
sheets, which is F109 in the verification log. R93 is a decision about a structure still to be built,
and this line exists so nobody reads it as a description of today.

**R94. Do not report a screen closed early.** Inventory went through a term sweep, an operator pass and
a decision ledger, and the sibling check and source sweep still found more than fifty issues between
them, including one the term sweep had created. Declaring completion is itself a claim needing one
more look.

### Who can see this

**R95. Each analytics tab follows the permission of the records it describes** (settled 2026-08-07).
Whoever can open the tenant list can read the Tenants screen; whoever can open dues can read Dues.
There is no separate analytics permission to invent and nothing to re-assign for people who already
have access. The argument that settled it: every present-moment number on an analytics tab is already
visible to anyone who can open the underlying list, so gating the summary harder than the records
hides a total from somebody who can count it by hand one tap away. **This replaced the earlier rule
that Dues and Collection unlock together on a shared analytics permission, with Expense separate.**
That line is superseded and must not be copied forward; DA-02 carries the correction.

**R96. Restricted shows a full-screen lock.** The copy is *"Analytics Restricted — You don't have
permission to view these analytics. Request access from your admin,"* with a Request Access button.
The dash in that string is shipped product copy, not prose, so R5 does not touch it.

**R97. There is no tenant-wise restriction for team members**, corrected 2026-08-06. Earlier sessions
recorded a "narrowed to their own added tenants" state as live; product says nobody is given that
permission, so it is not a state to design for on any screen. The mechanism is built and working in
code across Collection and Dues, which is why it kept surfacing. Verified in the analytics permission
code on 27 August 2026: **the analytics module does not read that permission at all.** A team member's
scope is the properties they can open and hold the matching view permission for, and nothing narrows
by who added a tenant. The question that used to sit open here, whether anyone holds it, is not a risk
to these screens. Whether it narrows the older list widgets is a separate question and belongs with
them.

**One known gap in R95, latent today.** The scope resolver maps `dues` and `collection` to invoices,
`expense` to expenses, `tenant` to tenants and `occupancy` to rooms, and returns nothing for
**`issues`**, so the Complaints tab is the one tab that is not gated. It is harmless right now,
because that tab is commented out of the navigation and all eleven of its blocks are sample data
(F108 in the verification log). **It stops being harmless the day Complaints is built**, so whoever
builds it owes the mapping first.

### What a record is, and what a number means

**R98. One record, three list screens.** **A booking, an active tenant and an old tenant are the same
record at three points of one life**, not three kinds of thing. A booking becomes an active tenant on
move-in day and an old tenant on move-out day. Consequence for every screen that counts movement: **a
move-in and a booking-converted are one event seen from two sides**, and the numbers must move
together. Settled from code on Tenants 2026-08-07, closing a question open since DA-02.

**R99. When a number is red** (owner, 2026-08-07). Most of these screens describe things that are not
recorded, and without a rule every screen becomes a wall of red on day one and managers stop reading
it.

| Red | Plain |
|---|---|
| An obligation not met: no police verification past the deadline, no agreement on record | A gap with no consequence: gender, food preference, tenant type not recorded |
| A date already passed: leaving date gone, agreement expired | A deadline not yet reached: joined three days ago, verification not yet due |

**Missing is red only where somebody can be held to it.** The owner chose red-for-missing over a
neutral treatment, then narrowed it twice under pushback: first to exclude anything still inside its
deadline, then to exclude gaps with no consequence. The final rule is stronger than either starting
position.

**R100. If a screen gets a trend chart, what it plots.** Shape was settled by precedent: stacked bars
per month, its own 6/12/24 range control, exempt from the time filter (R31). **Content is the
movement, not the level.** On Tenants that is move-ins against move-outs with the net stated. A
property can sit flat at 100 tenants all year while replacing forty people, and a headcount line draws
a straight line through all of it. Inventory is the suite's one exception and keeps plotting the
level, because space does not move the way people do.

---

## The sources, and which to trust

Reference facts about the material a sheet is written from. None of this is state, and none of it has
a version.

- **The developer's own sheet**, found 2026-08-06:
  `https://docs.google.com/spreadsheets/d/1XGFQxXyVhNaciM6VEMFpX7Ufj_3-VaaNOpJVOk2H9Po/edit`. One tab
  per screen (Issues, Tenants, Dues, Collection, Expense). Publicly viewable, and any tab exports as
  CSV via `.../export?format=csv&gid=<tab-id>`, where Collection is gid `865615981`. Genuinely
  useful: on Collection it confirmed five decisions and caught two wrong ones. **Pull it in Phase 0**,
  because reading it late cost an extra review round. **Old Tenants, Bookings and Leads have no tab
  in it.**
- **It writes "Today" where the system means "All Time"**, that is, no date limit. Confirmed twice, on
  Dues and Collection. An informal-notation quirk, not a real per-screen override.
- **A separate, older module powers the legacy list widgets.** It is not what these analytics screens
  are built on, so do not cite its behaviour as fact about them. **The exception is Collection**,
  whose definitions doc was deliberately built by reading that module's real logic, since it is the
  source the new module's maths was ported from. **Two bugs in it are confirmed live and must not be
  ported forward:** a deposit-adjustment filter written incorrectly that fails at run time, and
  RentOk-credit payments having the processing fee subtracted twice.
- **The design file** is checked live, never recalled. R76 is the rule; the index of frames and the
  tooling notes are in [[_Design File Index and Tooling Notes]].
- **The vault mirror** sits at `RentOk/PRDs/Homescreen Detailed Analytics/` and is a copy of this
  repo, not the other way round. Wikilinks in these files resolve inside Obsidian and not on GitHub,
  and are kept as written.

---

## What the next screen inherits

**Old Tenants is next.** These are the things worth knowing before Phase 0, all of them earned on a
previous screen.

- **No developer-sheet tab and no design states.** The workbook holds only Issues, Tenants, Dues,
  Collection and Expense. Old Tenants is the first screen with no informal source of any kind, so the
  design file and production carry the whole grounding load, as they did on Tenants.
- **A cancelled booking lands on the Old Tenants list** even though the person never lived there
  (R22). The screen must separate real departures from cancelled bookings or its headline overstates
  departures. Settled principle, unbuilt distinction.
- **Moved-out tenants' dues are deliberately excluded from Dues** and have one home, here (R53).
  Inherit the exact wording from Dues section 3.
- **The active-in-period model now has three consumers**: the homescreen Issues card (ticketed),
  Complaints Open Backlog (specified), and almost certainly a departed-in-period count here. Build
  once.
- **Two suite-wide checks Complaints spawned, worth running here:** sweep "follow" in every
  inflection rather than only "follows", and re-read any inherited rule against its source sheet
  before writing it as a fact. An Inventory no-button rule was recalled wrong and shipped.
- **Run the session card by card, one question at a time.** Complaints proved the cadence over eleven
  cards; the first consolidated report was rejected as overwhelming.
- **R76 is still owed here**, and on Bookings and Leads: any claim that a frame is stale or not drawn
  gets a live Figma check once each screen has a design file.

After Old Tenants comes **Bookings**, which inherits the booking definitions including R54's
booked-is-a-layer rule and the no-space row reserved on Inventory's View all sheet. Then **Leads**.

---

## What each screen close produced

The versions each sheet is at now live in the sheet's own frontmatter, indexed by
[the README](README.md#the-screens). What follows is what the work found, which does not change.

### Dues, closed 2026-08-06, backfilled to v11 and closed again at v12

Prior art: three hardened docs, reconciled and now superseded. The first screen, so most of the rules
above were born here, and the method was extracted into the `screen-handoff-pipeline` skill.

The backfill restructured it onto the template and production-sized it: 99.5% of unpaid money is
already overdue, and the 22-plus ageing bucket alone is 92% of that, split into 22 to 90 and 90 plus.
The tap matrix was rebuilt against the web app's Money filter drawer, which set a cross-tab drill
precedent. A sibling check against Tenants found and fixed a real gap Tenants had carried since it
closed, no share-of-total on its own property-wise card.

**Closed at v12 after four owner-driven rounds past "done":** the forward setting (Dues was the one
screen with none while the suite already required Custom to stop at today, and 8,703 bills worth ₹28.5
crore sat unreachable), GitHub navigation, one time vocabulary down to cell level, and the reader test
applied word by word, which killed "partition". The last check before close found what the greps could
not: removing one banned word had left five different phrasings for one idea, and the cover notes had
never been swept at all.

**Re-audited 2026-08-09** after an owner report that the View all sheet looked thinner than the rest.
It was: section 6 had been written from a frame the doc dismissed as an old draft, and the frame was
current and complete. That produced R76. Rewriting it picked up one live bug the dismissal had let
past, a change chip on Past Dues where the screen's own rule says there is none, plus three smaller
ones.

### Collection, closed 2026-08-06 at v11, restructured to v13 on 2026-08-08

Prior art: three hardened docs, an archived Drill Map and the dev sheet. The only screen with a view
toggle, and every card is defined on both sides. Most of the drill, loading and time-filter rules
above came from here.

Two findings came from checking the real thing rather than reasoning: the trend chart is a **stacked**
bar, not two bars, wrong in the doc for three versions and only surfaced when an owner answer did not
fit the description; and the payments list every drill lands on **does not name the filter that
brought you there**, which became R67.

The v13 restructure ran a 98-probe decision ledger against the new text and every ruling survived.
Production sizing, 18 measurements, killed nothing and corrected two claims: caution-money adjustments
are live since April rather than unbuilt, and RentOk-credit payments are near dead, 27 ever. The
filter-drawer enumeration settled every tap-matrix verdict and found four traps: widget-code arrivals
wipe every drawer filter, the drawer's cash option reaches one of two cash codes, the deposit-adjusted
widget on the live strip fails when tapped, and a six-state settlement filter sits fully built but
never enabled in the mobile app. **Biggest open item shipped to engineering:** a third of July's
online money, ₹48 crore, has no settlement record in any system, and the Unsettled tile cannot ship
until no-record is confirmed to mean not-settled.

The post-close source sweep found 12 real drops plus two overclaims, and the design verification
confirmed 21 of 25 fix claims while catching one stale one, because **design was already applying
v12's fixes and the file moved under the audit.**

### Expense, closed 2026-08-07, uplifted to v3 on 2026-08-08

Prior art: three superseded docs and the dev sheet. Two decisions were reversed mid-session by the
owner pushing back and both reversals were right: the Team Passbook card became a single tile (R83),
and killing bill coverage at 83% missing was wrong because current data measures behaviour under a
product that never asked (R80).

Four corrections to the hardened docs, all verified rather than reasoned: expenses with no paid date
do not exist at all; "fund source unavailable" is not a silent failure but every FlexiPe expense,
since FlexiPe writes its expense on a path that never creates a fund record; petty cash is 0.35%, not
a peer of the other two sources; and Receivable and Payable are two signs of one balance per person,
not two record types.

The final audit found about 40 more issues in a doc that had already been through a term sweep,
including a **duplicate tile whenever the filter is set to Current FY**, three drill families pointing
at filters that do not exist, eleven info icons with no content written, and a dead clipped copy of
the tile row inside the screen frame. The uplift settled the forward setting from production rather
than by copying a sibling: 45 of 340,524 expenses are future-dated, so no Coming up, and the suite
rule gained R19's by-design threshold.

### Inventory, closed 2026-08-07, uplifted to v2 on 2026-08-08

Prior art: five superseded docs, no dev-sheet tab. **Two design versions existed** and the older one
turned out to be a live requirements source rather than a superseded draft, which is how the drift was
found: the newer version's empty states still carry the older one's card titles. Five things came back
from it, including healthy states and the not-set-up screen.

**The owner reversed two recommendations, both times for the same reason**, and both are now R82.
Production sizing ran twice, 17 measurements: it killed the double-booked chip (5 cases in 36,000
identifiable beds), demoted three tiles to the View all sheet, moved the money card's axis from flat
type to sharing type (85% of rooms are typed plainly "room"), and found the session's biggest fact,
**87 to 90% of vacant inventory has never been occupied**, which no number anywhere showed.

The drill check found the two best numbers on the screen **cannot open their records at all**, because
nothing records when a room became empty. That produced R66. Firsts for the suite: a forward time
setting, per-tile chip polarity, healthy states, and a vocabulary (bed, unit, vacant, occupied,
booked, never rented) the other screens borrow.

The uplift's decision ledger earned its keep: 4 dropped rulings, 6 weakenings, and one true
regression, where the rewrite had reintroduced a behaviour a v1 audit round had explicitly removed.
**Design verification caught the file moving again:** a staged redesign of Agreements ending soon
carrying a **Notify Tenant** button that breaks the not-a-management-screen rule, now routed to the
tenants list. **The sibling check produced the biggest haul yet, 18 findings across all five sheets.**
One process scar: an argument-evaluation bug in a fix script truncated Collection's sheet to zero
bytes, restored from the repo clone, and the repo mirror paid for itself as a backup.

### Tenants, closed 2026-08-07, rewritten to v2 after the owner rejected v1

Prior art: a dev-sheet tab, plus a 46KB hardened PRD found late, which produced R77 and R78. The first
greenfield screen, and the one where the method changed most.

**Five suite-wide questions closed here**, none of them a Tenants question: the tab row (R93),
permissions (R95), where Leads lives, when a number is red (R99), and what a trend plots (R100). Four
had been open across multiple screens. **Three engineering questions were answered from data rather
than asked** (R84). One recommendation had to be **withdrawn after approval**, which produced R87.

**The owner rejected v1 for density and voice**: 12,400 words, 54 warnings, 81 platform statistics in
the body, coined vocabulary, code-flavoured framing. The sheet was rewritten whole to v2 at 6,200
words with all 34 rulings ledger-verified across the rewrite. **That rejection produced the suite's
biggest structural fixes:** the document grammar, the canonical template, the no-em-dash rule, the
lean budgets (R23), the per-setting filter grid, and the booking two-state correction (R22). Six
further defects were found after "done" was declared, each by one more check, which is R94.

Design-file facts worth carrying: **six prior versions of this screen exist**, not two, and nothing
was lost by name in the renames. "Tenant's Advanced Insights" is a second screen concept appearing
twice in the file that **no documented screen owns**. All fourteen Tenants empty states carry the
Complaints leak, *"You're all caught up! New maintenance requests and"*, truncated mid-sentence, which
is the third screen it has been confirmed on. Collection chrome sits as hidden layers in four Tenants
cards and is ruled dead, not deferred.

### Complaints, closed 2026-08-18 at v1, all three audits run

Prior art: a 62-requirement PRD filed off-convention (R77 firing a second time), the dev-sheet tab and
the homescreen's IO-01 spec. Greenfield with no drawn states, no design sign-off, and every number
defined from scratch across one owner session, card by card. 79 owner rulings ledger-verified.

Production sizing ran four passes and reshaped the screen: 38,365 open complaints with a median age of
444 days against a resolve time of two days, which is two populations wearing one word; only seven
status codes in use out of ten claimed; no Rejected ever written; 63% of stored ratings being the
value 6, raw average 4.99 against a real 3.3; a 5.5-hour clock mismatch between complaints and their
history; and escalation firing emails and recording nothing.

Owner overrules that changed the sheet: TAT drives overdue rather than a flat clock, keep the true
average rather than the median, Open rather than Unresolved, an all-complaints donut rather than
open-only, red on late ageing bars, admin separated from team, four categories not three, and filters
ship with the screen. Two rules proposed here were reversed by data mid-session and one by the owner's
own reasoning.

**Biggest single finding: most numbers cannot open their own records today**, so ten list filters ship
with the screen by owner decision. The Complaints empty-state copy leak was traced to its Figma
origin: five Expense cards duplicated from one Complaints card, layer names never renamed. **The three
audits ran the same session** and between them changed the sheet in 40 places, including two
definitional holes the author's own ledger could not see: the donut never stated its population, and a
past window's donut had no stability rule.

### The uplifts, all paid 2026-08-08

Collection's eight backfill items were applied at v13. Expense reached v3 and Inventory v2 in one
session each: cell vocabulary, contracts, tap matrix with arrival filters, zeros router, Test it
conversion, anchors and collapsed sections, both cover notes, repo synced. Inventory's uplift added a
**new closed phrase to the template**, "Averaged across the days of the window", for state numbers,
and swept 189 em dashes. **This queue is empty and has been since 8 August 2026.**

---

## Scope decisions about the suite itself

| Question | Ruling | When |
|---|---|---|
| How many screens are documented | **Nine.** Financial (Dues · Collection · Expense), People (Leads · Bookings · Tenants · Old Tenants), Inventory, Complaints. R93 carries the tab row itself | 2026-08-07 |
| Does Leads get its own screen | **Yes, and its home is the first sub-tab under People**, on the argument that a lead, a booking, a living tenant and a departed one are four points in one person's life, which is literally true of the records. Not folded into Bookings, not left out. Unwritten and unscoped, and it must not be built out of the Journey card on Tenants | 2026-08-07 |
| Does Refunds get a screen | **No, parked.** Hardened documentation exists from an earlier round, but there is no Refunds tab in the design file. Scope stayed Collection only. **The trigger for reopening it is a Figma frame to anchor it to**, and there is still none | asked 2026-08-06, still parked |
| Where does the source of truth live | **This repo.** The Obsidian vault at `RentOk/PRDs/Homescreen Detailed Analytics/` is a mirror of it. This reversed on 27 August 2026, after the repo ran ahead of the vault for three days | 2026-08-27 |
| What lands in the repo | Each screen's handoff sheet and reading maps, the template, this file, the verification log and the suggestions register. Briefs, formula maps, archives and superseded PRDs stay in the vault | 2026-08-07, owner's scope |

---

## What was still open when Tenants closed

**A dated snapshot, 7 August 2026, deliberately not re-checked since.** It is kept because deleting it
would drop open items nobody has ruled on. Anyone acting on it must re-check it first: Tenants has
been rewritten twice and verified against the built code since this was written.

**Five definitions that were blocking real cards:** what counts as a completed profile, what "renewal
rate after notice" means, whether the extra departed records count as move-outs, a usable source for
tenant type, and the lookback for Journey's two foot figures.

**Two decisions still open, both product rather than engineering:** whether the tenure breakdown gets
built, and whether this screen needs something showing change over time. R100 has since settled what a
trend would plot if one is added; whether Tenants gets one at all is still unanswered, and it is the
only screen in the suite without one.

---

## Session lineage

Sheet-writing sessions only, oldest first. The sessions that verified the built code have their own
lineage in [01](01%20—%20Build%20Verification%20Log.md). Transcripts sit at
`~/.claude/projects/-Users-eazypg-rentok-backend/<session-id>.jsonl`. **Never read a raw transcript
into context, they run past 2MB.** Use `@Past Chats` or the claude-history tools.

| Date | Session | Screen | What it produced |
|---|---|---|---|
| 2026-08-06 to 08-08 | `37fb37a0-57ee-4862-ad27-68d4335b8c5a` | Dues | Closed at v7, then resumed for the backfill to v11 and the four owner rounds to v12. The method was extracted into the `screen-handoff-pipeline` skill and into this file. Most of the rules above were born here |
| 2026-08-06 | `69b09043-0c95-43f9-a832-beb1bcfaf6cf` | Collection | Closed at v11 over five review passes and three behaviour matrices run as one-question-at-a-time interviews. Produced most of the drill, loading and time-filter rules, plus R73 and R91 |
| 2026-08-07 | `2da9684a-3efc-4454-98d8-b1e8552ede50` | Expense | Closed at v2. Two owner reversals, both right. Produced R80, R83, R92, R35 and R39, plus the backfill queue and a re-sequenced skill |
| 2026-08-07 | `45509081-b5cd-420e-a3cf-dd7b9b611b3d` | Tenants | Closed at v1, rejected for density and voice, rewritten to v2. Produced the template, R23, R38, R84 to R87, R93, R95, R99 and R100 |
| 2026-08-07 | `de9381a9-caf3-4e21-b534-04069a0fcfd4` | Inventory | Closed at v1, and the same session later ran the uplift to v2. Produced R40, R42, R43, R54, R66 and R82 |
| 2026-08-08 | `f164d8c6-2215-48be-bcab-f30719bbb0a1` | Collection backfill | Closed at v13. Filter-drawer enumeration, 18 production measurements, four traps found, both cover notes |
| 2026-08-08 | `8777dbce-be50-4586-b3b7-6e0a99e17c99` | Expense uplift | Closed at v3. Three own measurement errors in one pass, each of which read as "the prompt is stale" until re-measured. Repo synced at `7f4c407` |
| 2026-08-09 to 08-10 | `844bb8f1-c96c-4528-9ed3-41366e3b2525` | Dues, View all re-audit | Produced R76. Repo synced at `4a1293e` and `e8d467b` |
| 2026-08-17 to 08-18 | `9c5d7a87-11d2-4b8a-abd5-db4aad59ff15`, continued from `b4391da8-9ec4-4b65-9f7b-d86d63b724d4`, `db63ed2f-642d-4a44-93f7-18f6fa737e77` and `9f097f24-f726-4f6d-8e50-5b95d7ecadda` | Complaints | Closed at v1. The greenfield screen with the least going in and the most found. Produced R37, R41, R57 and R70 |
| 2026-08-27 | `c4a15499-d661-4359-a3e2-364d48c5c61b` | none | Audited this file, 02 and the README against the three structural failures fixed in 01, then rewrote this file as method and history on the owner's ruling |

---

## What was corrected in this rewrite

Recorded so nobody re-derives a claim this file used to make. Every one of these was found by the
sibling audit of 27 August 2026, kept at `_meta/sibling-audit-2026-08-27.md`.

| What this file used to say | What is true |
|---|---|
| "Nobody has written the contextual help. No screen has content for any of them", and separately "Info icons remain unwritten across all eight screens" | **Written and shipped.** 115 entries across the five built tabs. R59 keeps the obligation for new screens |
| The backend "returns placeholder data, scaffolded, not built", and Collection is "confirmed still a stub" | **Built and verified.** All five services shipped, and every block was read against its sheet in August 2026. The findings are in 01 |
| "Backfill queue below: Dues and Collection both need their own session", while the backfill section said the queue was empty | **The queue is empty**, and has been since 8 August 2026 |
| Tenants is "🟡 v1 drafted, audits partially run, not closed", while the inventory table on the same page said "Closed, v2" | **The sheet's own frontmatter is the answer**, and this file no longer carries a version at all |
| "Dues and Collection unlock together on one shared permission, Expense is separate" | **Superseded by R95** on 7 August 2026. Each tab follows the permission of the records it describes |
| "Which permission gates the analytics screens?" and "Tenant booking identification" listed as open questions | **Both were settled inside this same file**, on 7 August 2026, and are now R95 and R98 |
| The screen inventory listed eight screens | **Nine.** Leads was ruled a ninth screen on 7 August 2026, in this same file |
| The source of truth is the Obsidian vault | **This repo**, since 27 August 2026 |

**What was condensed rather than preserved.** The session lineage used to carry a 300 to 500 word
narrative per session. Those narratives are history, not rules, and the full text sits in this repo's
git history at commit `3522610` and in the transcripts. **No rule was dropped:** every locked decision
this file carried is above with a number, and the numbering exists so a later finding can say which
one it disagrees with.
