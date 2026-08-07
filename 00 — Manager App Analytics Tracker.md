
# Manager App Analytics — Project Tracker

**Read this first, every session.** It holds what's already been decided across all screens, so no session re-asks a settled question.

Method lives in the `screen-handoff-pipeline` skill. This file holds project state only.

---

## Screen inventory

One handoff document per screen. One screen per session.

| # | Tab | Screen | Status | Prior art | Notes |
|---|---|---|---|---|---|
| 1 | Financial | **Dues** | ✅ Closed — v8 · ⬜ backfill due | 3 hardened docs (reconciled) | [[DA-01 Dues — Handoff Sheet]] |
| 2 | Financial | **Collection** | ✅ Closed — v12 · ⬜ restructure due | 3 hardened docs + archived Drill Map + dev sheet | [[DA-02 Collection — Handoff Sheet]]. Every number walked through one at a time, three behaviour matrices, then a four-angle final audit (Figma, source docs, code, internal consistency) that found ~30 more issues. 3 owner decisions still open at build kickoff — see the doc's Open items. |
| 3 | Financial | **Expense** | ✅ **Closed** — v1, audited | 3 hardened docs (now marked superseded) + dev sheet | [[DA-04 Expense — Handoff Sheet]] + [[DA-04 Cover Note — Nimit]]. Tile row, no view toggle. Four screen decisions came from **querying production**, not judgement. Four corrections made to the hardened docs, which now carry superseded banners. |
| 4 | Inventory | **Inventory** | ✅ **Closed** — v1, audited | 5 hardened docs (now marked superseded) + **no dev sheet tab** | [[DA-08 Inventory — Handoff Sheet]] + [[DA-08 Cover Note — Nimit]] + [[DA-08 Cover Note — Ishika]]. Two design versions; the older is a real requirements source, not junk. Bed/Unit toggle audited: nothing breaks, almost everything silently changes meaning. ~20 owner decisions, 17 production measurements. First screen with a forward time setting, per-tile chip polarity, and healthy states. |
| 5 | People | **Tenants** | ✅ **Closed — v2, rewritten lean and stress-tested** | Dev sheet tab + a 46KB hardened PRD found late (now superseded in part) | [[DA-09 Tenants — Handoff Sheet]] + [[DA-09 Cover Note — Nimit]] + [[DA-09 Cover Note — Design]]. 26 sections. Six prior design versions. **Five suite-wide questions closed here**: the tab row, permissions, Leads, when a number is red, and trend content. New suite rule: today numbers vs window numbers. Three "engineering questions" answered from data instead of being asked. |
| 6 | People | **Old Tenants** | ⬜ Not started | Unknown | Moved-out tenants' dues are deliberately excluded from the Dues screen — this is likely where they belong |
| 7 | People | **Bookings** | ⬜ Not started | Unknown | |
| 8 | Complaints | **Complaints** | ⬜ Not started | Unknown | Empty-state copy leaking into Dues widgets originates from this module |

**Recommended order:** as numbered. Financial screens first, then Inventory, then the greenfield People and Complaints screens.

---

## Cross-screen decisions — locked

These apply to **every** screen. Inherit them; don't re-ask.

### How handoff sheets are written

The document states **what should be built**, not how it was decided. This is a hard standard, not a preference — a developer reading it shouldn't have to skip past reasoning to reach the spec.

- **No process archaeology.** No "locked this session," "checked against the dev sheet," "an earlier pass assumed," "independently confirmed by."
- **No cross-document references.** Not "per the Build Sheet," not "the Formula Map says." If a fact matters, state the fact. A handoff sheet that only works with four other documents open isn't a handoff sheet.
- **No code, and internal numeric codes count as code.** Say "a bill cleared using the tenant's deposit," never the mode number behind it.
- **Design-file problems go in short, clearly-marked side notes** — not woven into the definitions. The definitions outlive the Figma bugs. Collect every side note once more in a single list at the end, so design has one place to work from.
- **Plain punctuation, no em dashes** (locked 2026-08-07). Commas, colons, full stops. The dash survives only as the empty-table-cell placeholder and in document titles. No AI-tell vocabulary. The four earlier sheets still carry em dashes; sweep each at its next touch.
- **Every new sheet starts as a copy of [[_Handoff Sheet Template]]** (locked 2026-08-07). Never a blank page, never memory of the format. The template is the spine, the sub-section names, the table schemas and the fixed phrases in one place; the Dues and Collection restructures mean applying it. This exists because standards recalled during audit produced an incremental correction loop across three screens; standards copied at draft time cannot drift.
- **One grammar across the suite** (locked 2026-08-07). Same spine and section names, same table schemas with identical column headers (Drill: `Number | Opens | Ready?` · definitions and View all: `Row | Meaning` · number kinds: `Live` / `Time-scoped`, never a new coinage), same fixed phrases ("Test it:", "as of today", "from today onwards", "hides at zero", the Restricted copy). Tables wherever content is parallel. A sheet may drop what does not apply; it never renames or reorders what it keeps. **Two names settled 2026-08-07:** the access section is **"Who can see this"** everywhere, and the states section is **"What each card shows when it is empty, healthy or broken"** (renamed in Expense, Inventory and Tenants; Collection picks both up in its restructure). The filter-behaviour grid is titled **"What every number does on every filter setting"** on every sheet. Collection's "What is already broken in the code" section is deliberately dead and is never copied forward.
- **A booking has two states, confirmed and awaiting confirmation, and both are bookings** (owner, 2026-08-07). Only cancelling ends one, and **a cancelled booking lands on the Old Tenants list** even though the person never lived there. Consequences: booking counts show both states with awaiting as a layer; Inventory's booked-bed layer stays confirmed-only, so the two screens meet through the confirmed count; Move-outs never counts a cancelled booking. The Old Tenants screen must separate real departures from cancelled bookings or its headline will overstate departures.
- **Lean is a number, not a feeling** (locked 2026-08-07, after Tenants v1 ran to 12,400 words with 54 warnings and 81 platform statistics in the body). Budget per sheet: warnings under ten, platform figures only in the measured-figures appendix, reasoning in this tracker, definitions in one or two lines. Tenants v2 landed at 5,500 words against Expense's 8,100; that is the bar now. After any full rewrite, re-run the decision ledger against the new text.
- Still mark anything genuinely unsettled as a recommendation or an open question. Don't blur that with what's decided.

### Time filters

Two kinds of section:
- **Live** — always the current snapshot. No filter changes it.
- **Time-scoped** — shows a window.

One **global filter** per screen: **All Time, This Month (default), Last Month, Current FY, Custom.** Changing it updates every time-scoped section. A card with its own local filter overrides the global one **for that card only**.

**Naming is locked.** Where the backend says "Live" and "This Year," use "All Time" and "Current FY."

**Not every screen has a Live section.** Collection has none. Check per screen.

**The developer's rough sheet writes "Today" where the system means "All Time" / no date limit.** Confirmed twice (Dues, Collection) — an informal-notation quirk, not a real per-screen override.

### Cards with their own period (locked on Collection 2026-08-06, applies to every screen)

Some cards carry their own small date dropdown. Three rules:

1. **A card's own dropdown offers the same options as the filter at the top.** No per-card subsets, or managers hit "why can't I pick Current FY here?" with no good answer.
2. **Changing the filter at the top pulls every card back into line.** Any card on its own period snaps back. A manager can set one card aside deliberately, but can never end up on a screen showing four different periods without meaning to.
3. **Trend-style charts are exempt from the period filter entirely.** Every normal card answers "which period?" — one window, one set of numbers. A trend answers "how far back?" and needs several periods to say anything at all. A trend following a This Month filter would show a single bar. Give it its own range control, and let the view toggle still apply to it even though the filter doesn't.

### Unfinished and future periods (locked on Collection 2026-08-06, applies to every screen)

**Never compare an unfinished period against a finished one.** The default is This Month, so most of the time a manager is looking at a period still running. On the 3rd, that's three days. A change chip comparing three days against a whole finished month shows a collapse that isn't real — on the default view, for the first week of every month, on every property.

Compare against **the same point in the previous period** and say so: *"▲ 12% vs same point last month."* The note drops away once the period completes.

**An unfinished period is always marked as unfinished** wherever it appears — the change chip, the in-progress bar on a trend chart, anywhere else. One principle, not a per-component decision.

**Future periods are allowed.** They're meaningful on any screen with a bill-side view, since next period's bills already exist. Where a future period genuinely has nothing to show, say why in one line and offer a way out rather than showing a blank. Collection's version: *"Nothing received yet. Collection counts money as it arrives. Switch to Due Date to see what's billed for this period."* This fires only when the **whole** period is in the future. A period straddling today has real data and behaves normally.

### View-toggle audit — mandatory on every screen that has one

Several screens carry a **view toggle** separate from the time filter (Collection: Paid Date / Due Date. Inventory: Bed / Room. Others likely).

**A toggle can change which sections exist AND what surviving sections mean.** Walk every card through both sides and write out its definition in words on each side. Three failure modes, in increasing order of danger:

1. **Collapses** — the card splits data by the very dimension the toggle switches, so on one side it just restates the filter back (one row at 100%, rest zero). → **Hide it.**
2. **Goes dead or redundant** — permanent zeros, or a tile becomes an exact duplicate of its neighbour. → **Swap** for something that answers what that view is good at.
3. **Silently changes meaning** — looks fine on both sides, no zeros, no duplicates, but now measures a different thing and nothing says so. Sibling cards can also stop agreeing, because one can express the new meaning and another structurally can't. → **State the new meaning, and pin siblings to one definition so they still reconcile.**

The first two announce themselves. **The third never does** — only definition-by-definition comparison catches it. "Unaffected" must be earned per card, never assumed for anything that doesn't look broken.

Worked example (Collection's Paid Date / Due Date toggle — 4 of 7 sections affected):
- *Collapsed:* Collection Status — its split by bill timing can only return 100/0/0/0 in Due Date view. Hidden.
- *Dead:* four of six Overview tiles — two became duplicates, two permanent zeros. Swapped for Billed / Collected / Still Unpaid.
- *Silent meaning change:* the Trend chart's collection bar (money received that month vs. money collected against that month's bills — same chart, different question), and the Breakup tabs (Mode and Received by can only ever describe collected money, since an unpaid bill has no payment method — so all four tabs had to be pinned to collected, or they'd stop adding up).

**Missed twice on DA-02, caught by the owner both times.** Run it as its own deliberate pass; it will not surface from reading the design file.

### The final audit — mandatory before calling a screen closed

Run four independent checks once the doc is "done". On Collection this pass found ~30 real issues in a doc that had already survived eleven versions and five review rounds:

1. **Decision ledger.** List every decision the owner made this project (Collection had 45), then check each against the doc mechanically. Catches silent drops during rewrites.
2. **Design-file verification.** Send an agent to re-verify every visual claim against the actual frames, claim by claim, VERIFIED/WRONG/CANNOT-TELL. Catches everything asserted from memory or reasoning.
3. **Source-document sweep.** Send an agent to list everything the definitions docs require that the handoff never mentions, and every contradiction. This was the largest angle on Collection: 29 dropped items, 10 contradictions — including a third live bug the doc's own "bugs" section missed and a whole non-goals section that had evaporated.
4. **Verify the doc's own rationale claims.** Sentences like "because rent is due on the 1st" are claims too. Collection's weekly-trend rationale turned out to be a configurable setting, not a fact.

Two cautions from running it: **audit agents are also wrong sometimes** — one confidently reported the property card sorted when it isn't; re-check any agent finding that contradicts your own records before accepting it. And **full rewrites silently drop sections** — Collection's Copy Fixes section vanished in a v9 rewrite and nobody noticed for three versions; diff the section list against the previous version after any full rewrite.

### When the handoff supersedes its sources, say so in the sources

Collection's handoff now overrides its Formula Map and Build Sheet on at least six points (the credit-payment maths, settlement states, the unpaid row, date assignments, category rows, a fourth adjustment type). Those docs still call themselves "source of truth". **Whoever reads them next builds the wrong thing.** When a handoff wins an argument with its sources, mark the losing sections as superseded, or log it as explicit debt. Both DA-02's and DA-04's sources now carry the marker.

### One word, one meaning

A term that means two things in one doc is a bug even when every individual sentence is true. On Collection, "settled" meant "bill dealt with" in half the doc and "money reached the bank" in the other half. Sweep the final doc for its load-carrying words and check each has exactly one meaning; where the product itself uses one word two ways, say so explicitly once.

### Read the definitions doc twice

Prior definitions docs get skimmed in Phase 0 for orientation and then never re-opened, so whatever didn't seem relevant on first pass stays lost. **Re-read the definitions doc after drafting, line by line, against the draft** — and walk the section lists both directions.

Why this is a rule: DA-02's definitions doc specified a five-part Collection Status card. The design drew four. The draft documented four. The missing part — what's still unpaid — was half the card's value, and it had been sitting in a doc already read. **Drafting from the design file inherits the design file's omissions, and nothing flags the gap.**

### Measure a candidate number before specifying it, then judge it as a new feature (locked on Expense 2026-08-07)

**Query production before writing a number into a spec.** On Expense this killed three: a paid-date cleanup path (**zero** such rows across 339,732 expenses), a missing-category/payee/payer section (21 rows out of 151,012), and a permanent petty-cash bucket (0.35%). All three were specified in a hardened doc and would have been built.

**But current data measures behaviour under a product that never asked.** The same query said 83% of expenses have no bill attached, and the first call was to cut the number as meaningless. That was wrong, and the owner caught it. Nothing has ever shown an owner that figure, which is *why* it is 83%. A new number's test is "would seeing this change what someone does", not "is this common today". Low numbers can still be dead cases: the paid-date cut held, because nothing an operator does can create that row.

**Record the figures in the doc.** Whoever reads it next cannot re-check reasoning they cannot see.

**And the snapshot never blocks (owner, 2026-08-07).** Users, adoption and data are all growing, and requirements arrive with every new cohort. Current code and current data ground and prioritize a decision; neither is ever the stopping point. Where the snapshot and the user's need disagree, grill the owner rather than silently deferring to either.

### Before specifying a card, check whether the product already has a screen for it (locked on Expense 2026-08-07)

Expense had a fully designed Team Passbook card. Team Passbook is already a **live screen** with cash handover, give-money and settle-up. A read-only analytics copy would have been a weaker version of something that exists, and its one useful next step is banned on a diagnosis screen.

The pattern that replaced it generalises: **one number on the analytics screen, drilling into the live screen that owns the subject.** Where half a card feels wrong for a screen, usually the whole card is.

### A fixed-period card that duplicates a filter option stays visible (locked 2026-08-07)

Current FY is both an option on the top filter and a fixed tile. Select it and two adjacent tiles show the same number. Find this on every screen that has both, then **rule the same way every time: the tile stays.** Hiding it shifts every tile after it under the manager's thumb at the exact moment they change filters, and a row that reshuffles is worse than a number that repeats. The tile earns its place at the other settings by putting the year next to the month.

This was decided on Collection and re-decided the opposite way on Expense before the two were compared. **Screens one tab apart cannot answer the same question differently**, which is the whole reason the cross-screen check below exists.

### Check a finished screen against its siblings before closing it (locked 2026-08-07)

A handoff sheet can be internally perfect and still disagree with the screen next to it. Nothing inside one document can catch that, and a manager moves between these tabs in seconds.

Before closing any screen, read the already-closed sheets beside it and compare, line by line, on: the change-chip direction and colours, what a card's own date dropdown does when the top filter changes, fixed-period cards that duplicate a filter option, what the View all sheet does, whether share-of-total is shown, the permission wording, the Restricted copy, and the section names themselves.

Running this for the first time, on Expense against Dues and Collection, found five disagreements including **an inverted change chip on Dues that had been signed off as "confirmed in scope"** — green when dues rise, on a screen where rising dues are the problem. It also found Dues still specifying that a card's own period survives a change to the top filter, a rule reversed two screens ago.

### All Time has no previous period, so change chips disappear there (locked on Expense 2026-08-07)

Every screen offers All Time and every screen puts a change chip on its headline numbers. **There is nothing before "everything".** A chip comparing all history against a period that does not exist is blank at best and invented at worst. Hide it whenever the window is All Time, whether that came from the top filter or a card's own dropdown, and draw the tile state that has no chip. Nobody has checked this on Dues or Collection.

### Run the time matrix as a grid, not as a set of rules (locked on Expense 2026-08-07)

Stating the filter rules is not the same as walking every number through every option. Expense stated the rules, grilled the biggest question on that axis, and **still shipped a duplicate tile** that only turned up in the final audit, plus two more gaps that only appeared when the grid was actually drawn. Write the grid: every number down the side, every filter option across the top, and fill each cell. The cells that are not simply "follows" are where the work is.

### Check the destination can express the filter before promising the drill

Naming the destination is not enough. On Expense, three whole families of drill (bill coverage, payment mode, fund source) point at a list that has no such filter, so the number could never open its own records. **Enumerate the destination's real filters and match them against the drill list, one by one**, before calling drills done.

### Info icons are a content deliverable nobody owns

Every card on Expense carries an info icon, on the filled state and the empty state alike, and no screen so far has written what any of them say. An icon that opens nothing is worse than no icon. Either the explanations get written per card, in the screen's own plain language, or the icons come off. This is almost certainly true on every screen in this project and has never been logged.

### One word can collide across screens, not just within one doc

The "one word, one meaning" rule was written for a single document. It needs to run **across** screens too. **"Bill" means an invoice raised to a tenant on Dues and Collection, and a receipt attached to an expense on Expense.** Both are correct in their own screen and a manager moves between them in seconds. Where a word is already taken by another screen, say so explicitly and use the longer unambiguous phrase.

### Pay the superseded-sources debt in the same session

DA-02 logged this as an open item and it was never paid. **DA-04 paid it**: all three source documents now carry a banner at the top listing the eight points the handoff overrides. Make this part of closing a screen, not an open item, or it never happens. DA-02's sources were marked on 2026-08-07, a day and two screens after the debt was logged.

### The snapshot grounds a decision; it never makes it (hardened on Inventory 2026-08-07)

Locked before on Expense, broken again on Inventory, so it is restated as a rule with its worked example.

**A config value is the easiest thing in the system to change and the easiest to mistake for a constraint.** Inventory's nav config declares the screen live-only. That was read as the shape of the screen and used to recommend against a time dimension — which would have shipped a read-only copy of a screen that already exists. The owner reversed it. Then the same mistake repeated: a Bed View was recommended against because only 1% of properties are on the structure that makes beds real, when in fact **everyone is migrating to it**.

Both errors have the same shape: **a transitional state read as a permanent property of the system.** Before recommending against something because "the system does not do that today", ask what it would take to change and who is already changing it.

**And write requirements, not derivations.** A recommendation that names a table, a field or a lookup has stopped being a requirement. State what the user must see; engineering picks how it is computed. Corrected mid-session by the owner: *"Why are you getting into how it should be calculated? Let the engineers decide. We only want what we want."*

### An empty card is not always bad news (locked on Inventory 2026-08-07)

Every screen before this one treated an empty card as a gap — zero collection, zero spend, nothing to show. **On Inventory, an empty card is usually success:** no vacant rooms means the property is full. The suite therefore needs a state nobody had defined — *healthy*, distinct from *empty* — plus, where a whole screen can be empty for one reason, a single whole-screen state instead of N boxes each explaining themselves.

Check every future screen for this. Complaints is the obvious next one: zero open complaints is good news, and its cards will currently say the data has not arrived yet.

### Change-chip polarity can be per tile, not per screen (locked on Inventory 2026-08-07)

Dues, Collection and Expense each settle direction once for the whole screen — "up is bad here". **Inventory cannot:** occupied rising is good, vacant rising is bad, and rentable rising is neither. It needs three behaviours in one row, including a **neutral grey chip that the shared component does not have**.

Treat polarity as a per-number property from now on, and treat the shared chip component as carrying work whenever a new screen lands.

### "Live" is already taken, and it means filter-immune (locked on Inventory 2026-08-07)

Dues defines Live sections as *"always the current, real-time snapshot. No filter ever changes them"*, and Expense's design fixes remove a "(Live)" label for the same reason. Inventory needed a word for the *present-moment setting of the filter* and reached for the same one, which would have given it two opposite meanings on adjacent tabs.

**The word on a filter chip meaning "right now" is Now.** "Live" stays reserved for sections no filter touches.

Related, and the reason this surfaced: **the rule that relabels the backend's "Live" as "All Time" does not hold on every screen.** On the money screens the two mean the same thing — no date limit. On Inventory they are opposites: all-time occupancy is meaningless and the present moment is the most-used setting. **Check what the word means on the screen before applying the mapping.**

### Check the tab row itself, once, for all eight screens (found on Inventory 2026-08-07)

The design's tab row and the one the app serves do not match, and nobody had noticed across four screens. The app serves **Financial · People (Tenants · Old Tenants · Bookings) · Inventory · Complaints**. The design draws **Financial · Inventory · Tenant · Issues · Leads**.

So: Inventory sits in a different position, People is flattened so two documented screens have no tab, Complaints is renamed Issues, and **Leads is a tab with nothing behind it** — not in the app's navigation and not one of the eight screens being documented. Unowned, and it blocks all eight.

### A number that cannot open its own records (reinforced on Inventory 2026-08-07)

Expense established the check: enumerate the destination's real filters and match them one by one. Inventory ran it and found the same failure, but larger — **the two most valuable numbers on the screen have nowhere to go.** Nothing in the product records when a room became empty, so neither vacancy age nor never-rented can open a list, and never-rented is 87–90% of all vacant inventory.

The new lesson: **when the drill check fails, ask whether the underlying fact is recorded at all.** On Expense the gaps were missing filters over existing data. Here the data itself does not exist, which makes it a build prerequisite rather than a filter to add — a much bigger thing, and one that only surfaced because the check was run before drafting the drill table rather than after.

### Never tell a developer their code is wrong — tell them what the operator must get (locked 2026-08-07)

The sheet says **what has to be true for the user**, never what is broken in the code. No "the existing calculation counts wrongly", no "do not reuse X", no naming of tables, fields or functions, and no citation of code as evidence.

Where something today produces a wrong result, say it as an **outcome plus a test**:

> ✗ *"The period calculation counts anyone present at any point of the window as present throughout."*
> ✓ *"A tenant present for three days of a month contributes three days. **Test it:** a property that emptied on the 5th and refilled on the 25th reports roughly a third full for that month."*

Three reasons this is more correct, not just politer:

1. **A code claim goes stale the day someone edits the code.** A statement about what the operator must see stays true for the life of the screen.
2. **"That function is wrong" is not testable; an outcome is.** The second version is an acceptance criterion QA can run. The first is an opinion engineering can argue with — and then the conversation is about the PM's code reading rather than about the screen.
3. **It protects against the writer being wrong.** Stating the requirement and being told "we already do that" costs nothing. Stating "your code is broken" and having misread it sends someone to rewrite working code. This has already nearly happened on this project.

**What stays:** "the screen is not built yet" is scope, not critique, and changes the estimate. Keep it. What comes out is every characterisation of *how* existing code misbehaves.

This is the same rule as *write requirements, not derivations*, applied to defects instead of to computations. One lens, both cases.

⚠ **Retroactive:** DA-02 Collection carries a whole section titled *"What is already broken in the code this will be built from"*. That section is the clearest violation in the suite and is now on the backfill queue.

### A state that describes the future is a layer, never a slice (locked on Inventory 2026-08-07)

Owner correction, and it simplified the sheet rather than complicating it. **A booked-but-empty bed is still vacant.** A booking is a promise, not a person — until someone actually moves in, the space is empty and earning nothing, and a property that counted arrivals as occupancy could report itself full on the strength of move-ins that never happen.

So booked is reported as a **layer inside** whatever the bed's present state is, never as a slice beside it:

- a booking on an empty bed sits inside **Vacant**, which then splits into *booked* and *available*
- a booking behind a tenant under notice sits inside **Occupied**, as the *already replaced* line
- **the layer follows the bed, not the room around it** — a booked bed inside a partly-filled unit still counts, and the unit stays semi-vacant

**Available is the number a manager acts on**, so it is always stated rather than left to subtraction.

Three problems dissolved when this landed: the donut arithmetic closed (occupied + semi-vacant + vacant = rentable, + disabled = total) where booked-as-a-slice had double-counted every already-replaced bed; the vacant drill became fully reachable again; and the vacancy-age card stopped disagreeing with the tiles. **The general form is worth carrying to every screen: a state that describes what will happen is a layer on a present state, not a peer of it.**

One exception proves it: bookings with no bed allocated at all have nothing to be a layer on, so they stay their own row — and the sheet says why.

Two boundary rulings settled with the owner, both derivable from the same principle:

- **The layer follows the bed, never the room around it.** A booked bed inside a partly-filled unit still counts in the booked layer; the unit stays semi-vacant. No new state.
- **A flat is one unit only while a single tenancy holds all of its rooms.** One bedroom let separately makes it room-wise — including the empty bedrooms. The owner's form of the argument is the keeper: *if the tenant were taking the whole flat, they would be recorded against the whole flat; holding one room is itself the answer.*

And a consequence found one round later: **the rooms list disagrees with the layer model in both directions** — it treats a booked bed as taken (so Vacant drills come back short) and it counts unconfirmed bookings as bookings (so booked numbers over-return there). One meaning of vacant and booked across the app is now a logged requirement; until it lands, the sheet routes drills to wherever each number can be honoured and says so.

### Permissions and the Restricted state

- **Dues and Collection unlock together** — one shared permission. **Expense is separate.**
- Restricted shows a full-screen lock: "Analytics Restricted — You don't have permission to view these analytics. Request access from your admin," with a Request Access button.
- **There is no tenant-wise restriction for team members** — corrected 2026-08-06. Earlier sessions recorded a "narrowed to their own added tenants" state as live; product says nobody is given that permission, so it isn't a state to design for on any screen. The mechanism is built and working in code across Collection and Dues, which is why it kept surfacing. Engineering still needs to confirm nobody holds it — see Open questions.

### Drill-down (expanded on Collection 2026-08-06)

Every breakdown widget opens the exact filtered list behind the number tapped. **Exception:** forecast numbers (bills not yet raised) have no records to open, so those open an explainer sheet.

**A drill filters a list. It never re-scopes the screen.** Verified in code: the list endpoints already take property, date range, payment mode, due type, received-by and tenant type as filters, so a filtered list is the pattern the whole app uses. A card that instead narrowed the whole screen would be an interaction existing nowhere else, and managers would have to learn it for one row type.

**A number opens the screen that owns the kind of record it describes.** A number about payments opens a payments list. A number about bills opens Dues, even when it sits on the Collection screen. On Collection that means three numbers leave the screen: Billed, Still Unpaid, and the trend chart's Due segment.

**Cross-screen drills carry two things:** the destination opens on **the same period**, and the back control **names where the person came from** ("Collection", not a bare arrow).

**A drill from a trend chart moves the period rather than narrowing it**, because the chart has its own range. That is the only place this happens, and it should stay that way.

**"Others" rows are two steps.** Tapping Others opens a sheet with the full remainder; the drill happens from a row inside it.

### The destination has to say what it is showing

⚠ **A shared dependency, worth doing once rather than per screen.**

Checked against the live manager web app's **collections list** (not yet checked on the others): a manager arriving from a widget tap sees a page titled **"Collections Overview"**, fixed, whatever filter brought them there. Chips naming active filters exist but only render **after the person scrolls**. Filters that come from tapping a widget produce **no chip at all**. And an empty filtered slice shows one sentence: **"There are no collections present."**

So the drill hands the manager to a screen that will not say which number they tapped, and an empty slice reads as "you have no collections at all," which is false for a property that collected lakhs that month.

That undoes on the next screen what these screens are built to do. Any list an analytics screen drills into needs three things:

1. **Name the slice** in place of the fixed title.
2. **Name the filter in the empty state**, with a way on. Where there is an obvious next question, offer it.
3. **Show the active filter on arrival**, without scrolling.

### Loading and error states (locked on Collection 2026-08-06, applies to every screen)

**Every card loads, fails and retries on its own.** The screen never waits for its slowest card. Cards are not equally fast: simple sums return at once, while trend charts and joined breakdowns can take seconds, and All Time on an old property makes that worse. A manager should not watch a blank page while a chart they may never scroll to finishes.

**Skeletons match the shape of the card they become**, so nothing jumps position as the page fills in. This is what makes independent loading feel calm rather than restless.

**A failed card never renders a number.** Zero is real, meaningful data on every one of these screens, so it can never double as "we do not know." A settlement card that failed and showed zeros would send a manager to support about a failure that never happened.

**Empty and failed must look different**, because a manager acts differently on each. Empty means the query worked and there is nothing there, names what is missing, and offers no retry. Failed means the query did not work, says only *"Couldn't load this,"* and offers a Retry. Error copy stays boring: no apology, no cause, no codes.

**A failed card keeps its heading and shrinks** to its message and Retry, rather than holding a card-sized hole. **Retry refetches only that card**, never the screen.

**When every card fails it is the connection, not the cards.** Replace them all with one message and one Retry: *"Couldn't load this page. Check your connection."* Some cards failing stays per card.

### Sorting

**Every property-wise component across all analytics screens sorts highest to lowest.** Placeholder data in Figma often doesn't demonstrate it — don't read that as a contradiction.

### Entry point

**The entry point for each tab is the widget at the top of that tab.** Screens shown around it in the design file are reference context only.

**Not every screen has a single big "Hero" number.** Dues does. Collection doesn't — its entry point is a tile row. This matters beyond layout: edge states (negative totals, zero-activity periods, limited-view viewers) were all designed to live on a Hero card, and need a new home when there isn't one.

### Empty states

Use the Defaulter widget's copy as the tone reference: short factual title plus one reassuring line. CTA only where there's a genuine next step.

⚠ Several Dues empty states show leftover Maintenance/Complaints copy. Check every widget on every screen — but don't assume the leak is universal: Collection's were audited fresh and are all correct. Confirm per screen either way.

### Excluded records get exactly one home

Where a category of record is deliberately excluded from a screen's totals, it gets **one explicit row** where it does appear — not scattered, not hidden. Established with moved-out tenants on Dues. Same principle covers unlinked payments on Collection: they stay in the totals and get a visible row, or the card says plainly they're excluded there.

---

## Shared context

- **Backend:** one analytics module serves all eight screens. As of 2026-08-06 every block is wired and reachable but returns placeholder data — **scaffolded, not built**. Nothing is "broken"; it's unbuilt. Re-check per screen. Collection confirmed still a stub, with *less* scaffolding than Dues.
- **A separate, older module** powers the legacy list widgets. Not what these screens are built on — don't cite its behaviour as fact about them. **Exception:** for Collection, the definitions doc was deliberately built by reading that module's real logic, since it's the source the new module's maths should be ported from. Two bugs in it confirmed still live: a deposit-adjustment filter written incorrectly that fails at run time, and RentOk-credit payments having the processing fee subtracted twice. Neither should be ported forward.
- **Developer's own sheet — found 2026-08-06:** `https://docs.google.com/spreadsheets/d/1XGFQxXyVhNaciM6VEMFpX7Ufj_3-VaaNOpJVOk2H9Po/edit` — one tab per screen (Issues, Tenants, Dues, Collection, Expense…). Publicly viewable; export any tab as CSV via `.../export?format=csv&gid=<tab-id>` (Collection gid = `865615981`). Genuinely useful — on Collection it confirmed 5 decisions and caught 2 wrong ones. **Pull it in Phase 0**; reading it late cost an extra review round.
- **GitHub mirror:** `https://github.com/eazyapp-tech/manager-app-detailed-analytics` (private). **This vault folder is the source of truth**; the repo mirrors it with identical filenames, so syncing is a plain copy of changed files plus a commit. Sync at every screen close. One file is deliberately held out of the repo: the archived Liabilities audit that documents a live security issue in detail; the owner decides if and when it goes in.
- **Prior documentation** for several screens exists in this folder. Always check before deriving anything — and check it again after drafting. **Search by subject, never by folder naming convention** — the Tenants PRD was missed in Phase 0 because it is not filed under a `DA-` prefix.
- **Design-file node map, Figma and spreadsheet access recipes, and production query traps** are all in [[_Design File Index and Tooling Notes]]. Read it before touching Figma or Metabase; it saves an hour per screen and lists the column-name traps that have already cost time.

---

## Backfill queue — what each closed screen still owes

**Decided 2026-08-07: each screen gets its own session, before it goes to engineering.** Not now, and not folded into a live screen's session. The risk being managed is that a screen gets built from a stale sheet, which is exactly how Dues shipped inverted change-chip colours through a full close-out.

**Expense is the shape to copy.** It is the only sheet that a backend engineer can be handed cold, with a two-line reading map naming which sections are theirs. That is what the other two need.

### Applies to both Dues and Collection

1. **Restructure to one spine, numbered end to end.** Expense runs Build status → Where this lives → Rules for the whole screen → each card → what each number opens → who can see it → empty states → what this screen is not → build guidance → design fixes → open items → measured figures. Dues has no numbering at all. Collection numbers its eight cards but leaves everything around them unnumbered. Neither can carry a reading map until they do.
2. **One name per section across all three.** The Restricted state is currently "Restricted (no-permission) state", "When someone does not have access" and "Who can see this". Pick one for the set.
3. **Separate the design-only sections** so a backend reader can be told to skip them, the way Expense separates its empty-state copy and its design-fix list.
4. **Query production for the numbers each sheet specifies.** The measure-before-specifying rule postdates both screens, and on Expense it changed four decisions and killed three specced sections describing cases that occur zero, twenty-one and 0.35 percent of the time. Neither Dues nor Collection has had this run against it.
5. **Add a measured-figures appendix** where anything rests on data rather than judgement, so the reasoning can be re-checked.
6. **Make share-of-total a requirement, not a maybe.** Expense requires it on every breakdown row. Dues leaves it "up to the developers to add if time allows", which is not a spec. Collection has not been checked.
7. **Write the cover note** — which sections are the reader's, which two decisions are pre-answered, and where the superseded sources sit.
8. **Strip every code critique and code reference**, per the rule above. Restate each as an outcome plus a test. Collection's *"What is already broken in the code this will be built from"* section is the largest single case in the suite; Dues has smaller ones.

### Dues only

8. **Three cross-cutting sections are missing entirely**, all locked after it closed: loading and failure, the unfinished-period comparison, and the rule that a drill destination names what it is showing.
9. **Check the change chips on the Live tiles.** The Overview strip puts a month-over-month chip on all-time and fixed-window tiles. On a balance that may be perfectly sound, unlike the flow numbers on Expense where All Time has no previous period. Worth deciding rather than assuming either way.
10. **No collected design-fix list.** Its empty-state copy fixes sit at the top, but Figma problems are not gathered anywhere for design to work from.

*(Already corrected in place at v8: change-chip polarity, and a card's own date dropdown surviving a change to the top filter.)*

### Collection only

11. **Sources are marked superseded** as of 2026-08-07. Nothing further owed there.

---

## Open questions across screens

- **Nobody has written the contextual help.** Every card on every screen carries an info icon. No screen has content for any of them. Surfaced on Expense 2026-08-07; applies to all eight.
- **Row tap affordances are missing across the suite.** On Expense only the "Others" rows carry a chevron, though every row on every card opens something. Likely a shared component, so likely wrong on the other screens too.
- **Refunds** has hardened documentation in this folder from an earlier round, but no visible Refunds tab in the current design file. Fold into Collection, its own screen later, or dropped? Asked 2026-08-06: scope stayed **Collection only** — deferred, still no Figma frame to anchor it to. Still open.
- **Tenant booking identification.** Two internal sources disagree on how a booking is identified in tenant data. Unresolved from code this session. Affects any screen with a tenant-status breakdown — Collection, Tenants, Bookings. Engineering needs to settle it once.
- **The payments list needs to say what it is showing.** Named slice instead of a fixed title, a filter-aware empty state with a way on, and the active filter visible without scrolling. Verified as missing on the collections list; not yet checked on the others. Shared by every screen that drills, so worth doing once. Nobody owns this yet.
- **Which permission gates the analytics screens?** The lock is specced as an analytics permission covering Dues and Collection together; the existing collection list gates on the invoice-viewing permission. Two different answers, nothing reconciles them. Surfaced on DA-02, applies to every screen.
- **Do deeper drills ship before the access fixes?** Pages below the first list don't all check permissions the way the list does. Ship day one or hold? Owner decision, no owner yet.
- **Self-added-tenant permission — is it granted to anyone?** Built and working across Collection and Dues; product says nobody has it. If anyone does, their numbers are being narrowed today with nothing on screen saying so, and two people looking at the same property would see different totals. Engineering to confirm, then either remove the code or design for the state.

---

## Session lineage

Full stack, oldest first. Transcripts at `~/.claude/projects/-Users-eazypg-rentok-backend/<session-id>.jsonl`.

| Date | Session ID | Screens | Outcome |
|---|---|---|---|
| 2026-08-06 | `37fb37a0-57ee-4862-ad27-68d4335b8c5a` | Dues | Closed at v7 (Figma pass → owner review → code verification → `grilling` on 2 final threads). Method extracted into the `screen-handoff-pipeline` skill (pushed to `eazyapp-tech/screen-handoff-pipeline`) and this tracker. Most cross-screen decisions above came from this session. |
| 2026-08-06 | `69b09043-0c95-43f9-a832-beb1bcfaf6cf` | Collection | Closed at v11. Began as five review passes on the definitions, then three behaviour matrices run as one-question-at-a-time interviews: time filter (4 rounds), drill destinations (4), loading and errors (3). Two findings came from checking the real thing rather than reasoning: the trend chart is a **stacked** bar, not two bars, which had been wrong in the doc for three versions and only surfaced when an owner answer did not fit the description; and the payments list every drill lands on **does not name the filter that brought you there**, with an empty state reading "There are no collections present." Most of this tracker's drill, loading and time-filter rules came from this session. Earlier detail: five passes on definitions, where the owner supplied the dev-sheet link after publish and pushed twice on the view toggle, each time finding real gaps. **P1** (Figma + hardened docs): confirmed two live bugs in the old module and the new module still a stub; found real drift from every historical doc. 5 threads locked. **P2**: owner supplied the dev-sheet link after publish — caught a doc gap and 2 wrong calls. 3 threads. **P3**: owner asked what Due Date view does to Collection Status — it and 4 of 6 Overview tiles collapse there. 2 threads. **P4**: owner stayed skeptical about the Trend chart — the "everything else is unaffected" claim was under-verified; Trend and Breakup both silently change meaning. 2 threads. **P5**: owner asked for a proper re-read of the definitions doc and a plain rewrite — found a missing fifth row on Collection Status, an unlinked-payments rule, a sixth bill category, and a tenant-status conflict; rewrote the whole sheet without process archaeology or cross-doc references. Three of this tracker's rules (writing standard, read-definitions-twice, view-toggle audit) came out of this session. |
| 2026-08-07 | `2da9684a-3efc-4454-98d8-b1e8552ede50` | Expense | Closed at v2. Tile row, and **no view toggle** (confirmed in code), so the toggle audit did not apply. Two decisions were reversed mid-session by the owner pushing back, and both reversals were right. First: a fully designed **Team Passbook card** was about to be placed on the screen before anyone checked whether Team Passbook already existed. It does, as a live screen with settle-up actions, so the card became a single tile drilling into it. Second: production was queried to size every candidate number, which correctly killed three but also killed **bill coverage at 83% missing**. The owner's correction — current data measures behaviour under a product that never asked — restored it as money rather than a count. Four corrections were made to the hardened docs, all verified rather than reasoned: expenses with no paid date do not exist at all; "fund source unavailable" is not a silent failure but every FlexiPe expense, since FlexiPe writes its expense on a path that never creates a fund record; petty cash is 0.35%, not a peer of the other two sources; and Receivable and Payable are two signs of one balance per person, not two record types. The final audit found ~40 more issues in a doc that had already been through a term sweep, including a **duplicate tile whenever the filter is set to Current FY**, three drill families pointing at filters that do not exist, eleven info icons with no content written, and a dead clipped copy of the tile row inside the screen frame. All three source documents now carry superseded banners, paying a debt DA-02 logged and left. Post-close, the same session ran four more rounds, each owner-prompted and each finding real issues: the **time matrix drawn as a grid** (All Time has no previous period so change chips hide there; trend ranges 6/12/24 specified); the **first sibling check** against Dues and Collection (Dues corrected to v8 for inverted chip polarity and stale local-override behaviour; Expense reversed its own Current-FY-hide ruling to match Collection); a **true end-to-end read after the last edit** (15 defects, seven of them contradictions created by the audit fixes themselves — now trap 22 and a Phase 7 checklist item); and **DA-02's superseded banners** written, paying the remaining debt. Also produced: the Backfill queue section, [[DA-04 Cover Note — Nimit]], and a re-sequenced skill — sibling inheritance into Phase 0, production sizing into Phase 3, behaviour grids as Phase 3.5 before drafting, sibling check as Phase 6.75, cover note and final-read in Phase 7. |
| 2026-08-07 | `45509081-b5cd-420e-a3cf-dd7b9b611b3d` | Tenants | Closed at v1. First greenfield screen, and the one where the method changed most. **A 46KB hardened PRD existed the whole time** and was missed in Phase 0 because it is filed under the screen's name rather than a `DA-` prefix — the owner surfaced it after the sheet was drafted and audited. It independently agreed on the hardest thing (Snapshot/Flow versus the re-derived today/window split) and carried five things the design does not draw. **Five suite-wide questions closed**: the tab row, permissions, where Leads lives, when a number is red, and what a trend plots — four had been open across multiple screens. **Three "engineering questions" were answered from data rather than asked**: room changes cannot inflate move-ins, the two leaving dates agree in 4,497 of 4,498 cases and simply mean requested versus approved, and booking states were already computed both ways on the homescreen. Two owner overrules both survived a single narrow push and came out sharper. One recommendation had to be **withdrawn after approval** — 188,568 "expired agreements" turned out to be the system assuming eleven months for people with no agreement at all. The final read found three contradictions created by the session's own patches. Also produced: two cover notes, [[_Design File Index and Tooling Notes]], and a superseded banner on the PRD plus a second on DA-02. **Post-close, the owner rejected v1 for density and voice** (12,400 words, 54 warnings, 81 platform statistics in the body, coined vocabulary, code-flavoured framing) and the sheet was rewritten whole to **v2**: 6,200 words, one warning, suite vocabulary, all 34 rulings ledger-verified across the rewrite. The rejection produced the suite's biggest structural fixes: the document grammar, the canonical template ([[_Handoff Sheet Template]]), the no-em-dash rule, the lean budgets, the per-setting filter grid (which exposed a real Coming-up spec hole), the Renewal overdue figure (owner-found gap), and the booking two-state correction (owner-caught, with the cancelled-bookings-on-Old-Tenants consequence). Six further defects were found after \"done\" was declared, each by one more check: the lesson recorded here is that declaring completion is itself a claim needing one more look. |
| 2026-08-07 | `de9381a9-caf3-4e21-b534-04069a0fcfd4` | Inventory | Closed at v1. **Two design versions existed** and the older one turned out to be a live requirements source, not a superseded draft — the newer version's empty states still carry the older one's card titles, which is how the drift was found. Five things came back from it: healthy states, the not-set-up screen, Agreements ending soon, the Semi-Vacant slice, and the per-card AI insight. **The owner reversed two recommendations, both times for the same reason** — a config value saying "live only" was read as the screen's shape, and a 1% adoption figure was read as permanent when everyone is migrating. Both are now the hardened snapshot rule above. The owner also corrected a requirement written as a derivation. **Production sizing ran twice** (17 measurements): it killed the double-booked chip (5 cases in 36,000 identifiable beds), demoted three tiles to the View all sheet, moved the money card's axis from flat type to sharing type (85% of rooms are typed plainly "room"), and found the session's biggest fact — **87–90% of vacant inventory has never been occupied**, which no number anywhere showed. The Bed/Unit toggle audit found nothing visibly broken and almost everything silently changed meaning; the partly-filled room had nowhere to sit in Unit View, which is why Semi-Vacant returned. The drill check found the two best numbers on the screen **cannot open their records at all** — nothing records when a room became empty. The sibling check found 16 disagreements including a self-contradiction created by my own patch edit an hour earlier, and the source sweep found 36 dropped items, 16 contradictions and 12 stale claims across five documents that had never carried a superseded banner. All five now do. Firsts for the suite: a forward time setting, per-tile chip polarity, healthy states, and a screen whose vocabulary (bed, unit, vacant, occupied, booked, never rented) the other seven will borrow. |

---
---

## Next action

Start **Old Tenants** (screen 6) — now the second sub-tab under People, after Leads.

1. **Read the Tenants sheet first.** Old Tenants is the same record one state later, and it inherits the whole vocabulary — today numbers versus window numbers, under-notice-as-a-layer, the colour rule, the two leaving dates. Almost nothing needs redefining.
2. **It has no developer-sheet tab.** The workbook holds only Issues, Tenants, Dues, Collection, Expense. First screen in the suite with no informal source at all — so production and the design file carry the whole grounding load.
3. **Search prior art by subject, not by folder name.** A 46KB hardened PRD for Tenants was missed in Phase 0 because it was not filed under a `DA-` prefix.
4. **The second departure marker matters most here.** 1,005 properties record every departure in it, and the Old Tenants list does not show them — so on those properties the list is empty while people have demonstrably left. This is Old Tenants' single biggest question and it is already sized.
5. **Move-outs on Tenants and the population of Old Tenants are the same people.** They must agree exactly. This is the first pair in the suite where one screen's window number is another screen's whole population.
6. **Check whether an empty screen is good news.** A property with no old tenants has either never had anyone leave, or is brand new. Two different states, and the Tenants screen already carries the pattern.
7. **The Old Tenants list contains cancelled bookings**, people who never lived there. Tenants' Move-outs already excludes them. This screen must separate real departures from cancelled bookings everywhere, or every headline overstates departures.

### Then

**Bookings** (screen 7), which inherits the booking definitions already settled here, and where the "awaiting confirmation" number disappearing under a filter needs a ruling. Then **Complaints**, then the new **Leads** screen, then the Dues and Collection backfills.

---

## Superseded — previous next action (Tenants, now closed)

Start **Tenants** (screen 5). It is the first of the three People screens and the first greenfield one — Dues, Collection, Expense and Inventory all had hardened prior art; Tenants has a developer-sheet tab and nothing else.

1. **Read the Inventory sheet before drafting.** It defines bed, unit, vacant, occupied, booked, under notice and never rented for the whole suite, and Tenants will reuse most of them. Reusing beats redefining.
2. **Watch for the duplicate-screen trap in its strongest form yet.** A live tenants list already exists and it can act. Inventory's answer was that the period and forecast views are what earn the tab — Tenants needs its own answer to the same question, and it needs it before any card is specced.
3. **The tab row is broken and it blocks every screen.** See above. Somebody has to reconcile it once.
4. **Two tenant questions are still open across screens** and both land squarely here: how a booking is identified in tenant data, and whether Dues and Collection actually disagree about counting an under-notice tenant as active. Settle both on this screen.
5. **Check for healthy states.** Tenants may have them; Complaints certainly will.
6. **Run the drill check before drafting the drill table, not after.** On Inventory that ordering is what turned "a missing filter" into "the fact is not recorded anywhere", which is a build prerequisite rather than a small addition.
7. **Query production to size every candidate number**, both directions, and remember the snapshot rule above — twice this session a recommendation was wrong because a transitional state was read as permanent.
8. **Mark the source documents superseded before closing**, not as an open item.

### Still owed

- **Backfill queue** below: Dues and Collection both need their own session before engineering sees them. Inventory has now widened the gap — it carries a measured-figures appendix, a reading map, two cover notes and superseded sources; neither of them has any of that.
- **Info icons** remain unwritten across all eight screens. Inventory's section 3 is written to double as that content and is the natural place to start.

**And do not report a screen closed early.** Inventory went through a term sweep, an operator pass and a decision ledger, and the sibling check and source sweep still found more than fifty issues between them — including one the term sweep had created.

---

## Session addendum — Tenants (2026-08-07)

**Screen 5 status:** 🟡 v1 drafted, audits partially run. [[DA-09 Tenants — Handoff Sheet]] at `DA-09 — Tenants/`. Not closed — see "What Tenants still owes" below.

### New cross-screen rules from this screen

#### A number is either a today number or a window number, and the two never mix silently

**Tenants is the first screen in the suite whose tile row holds both**, and the discovery came from the owner refusing a recommendation rather than from any check in the pipeline. The first proposal was "every tile averages across the period", copied from Inventory. Three of the seven tiles are counts of events — notices raised, bookings confirmed, move-outs — and averaging an event across days is meaningless.

- **A today number** answers *how many are there now*. It ignores the filter and says "as of today" on its face.
- **A window number** answers *how many happened*. It follows the filter.
- Cards holding both label **per number, not per card**.
- **Only window numbers carry a change chip**, and nothing carries one on All Time.

The general form for every future screen: **before writing any filter behaviour, sort every number on the screen into these two piles.** Inventory never needed this because every number there was a today number; Complaints will need it badly, since open-complaint counts and complaints-raised-this-month are the same two kinds side by side.

**And the filter options follow from the mix.** Inventory rests on "Now" because it describes space. Tenants has **no Now setting at all** — it is a screen about movement, and four of its seven tiles would have nothing to count. Options are This Month (default) · Last Month · Current FY · Custom · All Time.

#### A state that describes the future is a layer — confirmed on a second screen

Inventory's booked-is-a-layer rule held here unchanged, on different subject matter: **a tenant under notice is still active**, and an approved eviction is still active. Both are layers inside Active, never slices out of it. The rule has now survived two screens and should be treated as suite doctrine.

This also settled a question open since DA-02: **Dues and Collection do not actually disagree.** A tenant under notice stays inside Active, and Under Notice is a layer shown separately. Both docs were describing the same thing from two ends. Closeable.

#### One record, three list screens — how a booking is identified

Settled from code, closing the other long-open question. **A booking, an active tenant and an old tenant are the same record at three points of one life**, not three kinds of thing. A booking becomes an active tenant on move-in day and an old tenant on move-out day. Consequence for every screen that counts movement: **a move-in and a booking-converted are one event seen from two sides**, and the numbers must move together.

#### When the drill check fails, say which kind of failure it is

Inventory's lesson was "ask whether the fact is recorded at all". Tenants gives the other answer, and the distinction is worth stating because the two need very different write-ups:

- **Inventory:** nothing records when a room became empty. A fact has to start being recorded. Whole cards blocked.
- **Tenants:** every date is recorded in full — leaving dates, notice dates, approval times. **None can be asked for as a range.** A filter has to be added over data that already exists.

Both look identical in a drill table (❌ not reachable). They are weeks apart in effort. **Name which one it is, every time.**

#### Empty tables are not evidence

Three tables that look like they own this screen's data hold **zero rows** — the notices table, the lead-status table, the new-bookings table. The real notice history sits in the eviction records (119,655 rows) and lead state sits on the tenant record. A near-miss in the other direction too: booking approval looked unrecorded until a second table turned up holding exactly it. **Trap 23 fired twice in one session** — the first matching artifact is not the governing artifact, in both directions.

#### The suite has a screen with no trend chart and nobody noticed

Dues, Collection, Expense and Inventory all carry one. **Tenants carries none**, in the current design and in all five earlier versions. It is the screen most about change over time. Logged as an open item; worth a suite-level look rather than a Tenants decision.

### Design-file facts worth carrying

- **Six prior versions of this screen exist**, not two. Five of them are stable and identical in card set; the current one renames four cards and splits two. Nothing was lost by name — Lifecycle→Journey, Eviction Pipeline→Upcoming Eviction, Tenure Type→Stay Type, KYC & Compliance→Tenant Verification + Tenancy Details, Tenant Demographics→Tenant Profile + Tenant Details — and two cards are new.
- **"Tenant's Advanced Insights" is a second screen concept** appearing twice in the file, holding Widgets · lifecycle · Tenant Demographics. Issues has the same thing. **No documented screen in this project owns it.**
- **The Complaints empty-state leak is now confirmed on a third screen.** All fourteen Tenants empty states carry *"You're all caught up! New maintenance requests and"*, truncated mid-sentence. Assume it is on every remaining screen.
- **Collection chrome is embedded as hidden layers** in four Tenants cards — money rows and a "Received by" tab. Ruled dead, not deferred.
- The `Analytics Design Draft3- WIP` section names the four closed screens "— done". **Tenant insight has no "done" suffix.**

### Two questions closed this session

- ~~Tenant booking identification~~ — **settled.** One record, three states. See above.
- ~~Dues and Collection disagreeing on under-notice tenants~~ — **settled.** They never disagreed.

### What Tenants still owes before it can be called closed

1. ~~The design-file verification pass~~ — **run.** It overturned three of my own claims; see below.
2. ~~The source-document sweep~~ — **run** as a line-by-line re-read of the developer sheet against the draft (Tenants has no hardened prior art). All 59 defined numbers accounted for; two label rewordings reverted.
3. ~~The full sibling check~~ — **run.** Findings and fixes below.
4. **Five open definitions** that block real cards: what counts as a completed profile, what "renewal rate after notice" means, whether the extra departed records count as move-outs, a usable source for tenant type, and the lookback for Journey's two foot figures.
5. ~~No superseded-sources debt~~ — **wrong, and corrected.** A hardened PRD did exist; see below. It now carries a banner listing the seven points the handoff overrides, paid in-session rather than logged.
6. **Two decisions still open**, both product rather than engineering: whether the tenure breakdown gets built, and whether this screen needs something showing change over time (it is the only one in the suite without it).

### Sibling check — results (run 2026-08-07)

Six findings, all fixed.

| Finding | Where | Fix |
|---|---|---|
| **"State" meant two things** — the tenant states (booking / active / old) and the kind of number | Tenants, internal | Number kinds renamed **today number / window number**. "State" is now reserved for tenants |
| **View all sheet was a sub-section** (5.1), while Expense and Inventory both give it its own numbered section 6 — and it was missing from the contents table entirely | Tenants vs DA-04, DA-08 | Promoted to section 6; everything after renumbered; all cross-references re-pointed and verified |
| **All Time change-chip rule missing** — locked on Expense, never restated | Tenants | Added. On All Time the whole screen is chipless |
| **Share-of-total not required on breakdown rows** — Expense requires it on every one | Tenants vs DA-04 | Required, and distinguished from the coverage line, which is a different thing |
| **View all rows never said whether they drill** | Tenants | They do, on the same rules as the drill section |
| **Lock copy differed** — Collection said *"Analytics Restricted."* with a full stop; Dues, Expense, Inventory and Tenants all use *"Analytics Restricted — "* with a dash. One shared component, four against one | **DA-02** | **Corrected in DA-02**, with a banner at the top of that sheet saying what changed |

Two smaller corrections came out of re-reading the developer sheet against the draft: **B2B and Residential** had been reworded to "let to a company / let to an individual", and **E-KYC / Manually verified** to "verified electronically / verified by hand". Both are the app's own labels and both are back, glossed on first use. This is the mirror-the-UI-label rule — the plain-language standard applies to *the writer's* vocabulary, not to words the team and the product already share.

**What the sibling check did not find** is as useful as what it did: permission grouping, Restricted behaviour, sorting, loading and failure, empty-versus-failed, and the per-number chip polarity rule all matched the four closed sheets without adjustment. Inheriting them in Phase 0 rather than re-deriving them is why.

### Design-file verification — results (run 2026-08-07)

**It overturned three claims in my own draft, all made from reading an extract rather than from looking at the frame.** The pipeline says this every time and it was true again.

| I had written | What the frame shows |
|---|---|
| "Change chips sit on all seven Overview tiles; five come off" | All seven carry a chip, but **five are already switched off**. The two left visible are Active Tenants and Active Bookings — **the exact two that should never have one**. The two that should carry a chip, Approved Bookings and Notices Raised, are among the five switched off. **The design has it precisely backwards**, which is a better and more useful finding than the one I invented |
| "Notices Raised is currently green when rising" | It has no visible chip at all, so there was nothing to be green |
| "Upcoming Eviction is missing the 31+ bar" | The bar exists — it sits **past the right edge of the card** and cannot be seen, and it carries one value where every other bar carries two |

**And it found something I had not looked for.** That same card draws a **sixth column labelled "Move-out"**, far outside the card edge, carrying values. **Inventory found the identical leftover on its trend chart** — same word, same clipped-past-the-edge position. Same component lineage, second screen. **Check every remaining screen for it rather than finding it a third time.**

The lesson is the one already in the skill, in a new costume: an extract tells you what text exists, never whether it renders. **Hidden-ness lives on a container, not on the text**, so any check that reads text nodes alone will report a switched-off element as visible. Walk the ancestors, or look at the picture.

### Five suite-wide questions closed by grilling (2026-08-07)

All five had been open for multiple screens. None was a Tenants question; Tenants is just where they finally got asked.

#### The tab row — settled

**The system's structure is correct; the design is redrawn to match.**

| | |
|---|---|
| Financial | Dues · Collection · Expense |
| **People** | **Leads · Bookings · Tenants · Old Tenants** |
| Inventory | — |
| Complaints | — |

People keeps its grouping rather than flattening, the sub-tabs run in **lifecycle order** rather than alphabetically, and **Complaints keeps its name** — the design's "Issues" is dropped. This unblocks all eight documented screens; it had been logged as "nobody's" since Inventory.

#### Leads becomes a ninth screen, inside People

Not folded into Bookings, not left out. **Its home is the first sub-tab under People**, on the argument that a lead, a booking, a living tenant and a departed one are four points in one person's life — which is also literally true of the records. Unwritten and unscoped; nothing existing depends on it, and it must not be built out of the Journey card on Tenants.

#### Permissions — one rule for the whole suite

**Each analytics tab follows the permission of the records it describes.** Whoever can open the tenant list can read the Tenants screen; whoever can open dues can read Dues. No separate analytics permission to invent, nothing to re-assign for people who already have access.

The argument that settled it: every present-moment number on an analytics tab is already visible to anyone who can open the underlying list. Gating the summary harder than the records hides a total from somebody who can count it by hand one tap away. This replaces DA-02's "Dues and Collection unlock together on an analytics permission" — **that line is now superseded.**

#### When a number is red — a rule the suite did not have

Most of these screens describe things that are not recorded. Without a rule, every screen becomes a wall of red on day one and managers stop reading it.

| Red | Plain |
|---|---|
| An obligation not met — no police verification past the deadline, no agreement on record | A gap with no consequence — gender, food preference, tenant type not recorded |
| A date already passed — leaving date gone, agreement expired | A deadline not yet reached — joined three days ago, verification not yet due |

**Missing is red only where somebody can be held to it.** The owner chose red-for-missing over a neutral treatment, then narrowed it twice under pushback: first to exclude anything still inside its deadline, then to exclude gaps with no consequence. The final rule is stronger than either starting position.

**Worth carrying to Complaints especially**, where the same tension will appear in reverse.

#### If a screen gets a trend chart, what it plots

Shape was already settled by precedent — stacked bars per month, own 6/12/24 range control, exempt from the time filter. **Content on Tenants: move-ins against move-outs, with the net stated.** The reasoning generalises: plot the movement, not the level. A property can sit flat at 100 tenants all year while replacing forty people, and a headcount line draws a straight line through all of it.

### The prior art I missed in Phase 0 — and the rule that comes out of it

**A 46KB hardened PRD for this screen existed the whole time.** `Tenant Insights/Tenant Insights.md`, v1.1, dated 3 Feb 2026 — Part 0 operator brief plus Part 1 foundations, with a glossary, nineteen invariants, formulas, threshold definitions and eight blocking open items. The owner surfaced it after the sheet was drafted and audited.

**Why the pre-flight missed it:** every check looked for `DA-` prefixed folders, because that is how the other four screens are filed. This one sits in a folder named for the screen. **Prior art must be searched for by subject, never by naming convention** — the convention is the thing a stray document is most likely to be missing.

**What it cost:** less than it should have, because the two documents independently agreed on the hardest thing. It splits every number into **Snapshot** and **Flow**; the handoff had re-derived the same split as *today number* / *window number*, down to the rule that pipeline cards ignore the filter and say "as of today". Two people reaching the same structure separately is the strongest confirmation available. But it also carried five things the design does not draw and the handoff had missed entirely: **police-verification deadlines configurable per property**, the **overdue** concept that follows from them, the **legal-versus-operational compliance split**, the **money-at-risk** framing, and a **tenure breakdown** as the only preventive number on the screen.

**What the handoff overrides, and why it is now marked:** seven points, listed in a banner on the source. The largest is that **voluntary and eviction notices cannot be told apart** — the system records who raised a notice, not why — which was the document's own blocking item #2 and which three of its constructs depended on.

**The general rule: a stale document is not a wrong document.** This one was six months old, had eight unresolved blockers, and was still the best statement of why the screen exists. Reconciling it took an hour; re-deriving what it already knew would have taken a session and would have lost the five things above.

### How the grill should run — sharpened on this screen

Three things worked better here than on any previous screen, and all three are cheap to repeat.

**Answer it from evidence before asking it.** Three items sat on the open list as "engineering questions" and all three dissolved under a query. Room changes update a tenant in place and cannot inflate move-ins — 320 tenants in the whole system have more than one room record. The two leaving dates were never in conflict — they agree in 4,497 cases out of 4,498 and simply mean *requested* versus *approved*. Booking states did not need a ruling because the homescreen already computes them both ways. **A question you can answer is not a question to ask.**

**Push once on the answer you did not recommend.** The owner overruled the recommendation twice. Both times a single narrow follow-up made the decision better rather than reversing it — red-for-missing narrowed to "only where somebody can be held to it", and Leads-as-its-own-screen forced the useful question of where it lives, landing it inside People in lifecycle order. **Pushback is not disagreement; it is finishing the decision.**

**When the owner says "I don't understand", the doc is wrong, not the reader.** It happened twice, and both times the confusion was real: two different questions had been mashed into one card. Untangling "does an agreement document exist" from "when does the term end" dissolved a problem that had taken three rounds. **Confusion is a finding.**

And one that cost real credibility: **a recommendation was made, approved, and then found to rest on an artefact.** The "188,568 expired agreements" was mostly the system assuming eleven months for people with no agreement at all. It had to be withdrawn after the owner had already agreed to it. The lesson is not "check harder" — it is that **a number large enough to reshape a card deserves its derivation checked before it is put to the owner, not after.**

### Note for whoever runs the next screen

Tenants was the first greenfield screen and the working method changed because of it. With no hardened prior art, **the developer's sheet and production data carried the whole grounding load**, and both turned out to be stronger sources than expected — the dev sheet defined all twelve cards and independently flagged four of its own definitions as uncertain, and production settled six decisions. The design file's own version history did the rest.

**Old Tenants and Bookings have no developer-sheet tab** — the workbook holds only Issues, Tenants, Dues, Collection, Expense. Those two screens will be the first with no informal source at all.
