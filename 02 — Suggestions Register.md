# Suggestions Register

Problems in the shipped product that are **not** failures against a handoff sheet. The sheet either
never covered them, or covered them and the underlying behaviour is wrong anyway. Found while
verifying the build, kept apart from [[01 — Build Verification Log]] so the defect list stays a defect
list.

**Nothing here is actioned without an owner ruling.** The owner is Sanchay. Each entry says what
happens today, why it looks wrong, and what is proposed. **The next number is S13.**

S10, S11 and S12 arrived on 27 August 2026 from the project tracker, which used to hold them as
unowned open questions and no longer holds live state of any kind. None of them is new; each is dated
from when it was first found.

| # | Suggestion | Status | Waits on |
|---|---|---|---|
| S1 | Stop the biller raising future bills for unconfirmed bookings | Proposed | Owner |
| S2 | Cancelling a booking should clear its unpaid dues and say so | Proposed | Owner |
| S3 | Rename "Under notice" to "Under eviction", with two named parts | Agreed, D28, 2026-08-25 | A ticket for the six shipped strings |
| S4 | A list, ageing and a prompt for deposits owed to tenants who have left | Proposed | Owner |
| S5 | Rename the rollup row so "Others" means one thing | Parked | Owner, alongside Dues and Collection |
| S6 | A way to record paying staff back, before the tile reads fifty crore | Proposed | Owner |
| S7 | Ask for the agreement length at move-in, so the gap stops growing | Proposed | Owner |
| S8 | Validate the property's agreement-length setting where it is typed | Proposed | Owner |
| S9 | Add Past their date to Journey's eviction split, so the rows add up to Active | Ruled out of the build; one question open | Owner, on the note against the row |
| S10 | Give every row that opens something a chevron, not just the Others rows | Proposed | Owner, on whether this is worth a shared-component change |
| S11 | Make the list a drill lands on say which slice it is showing | Proposed | Owner, on whether this ships with the analytics screens or after |
| S12 | Check permissions on the pages below the first list, the way the list itself does | Proposed | Owner, on whether the drills ship before this is fixed |

| Status | Meaning |
|---|---|
| Proposed | Written up, owner has not weighed in |
| Parked | Owner has seen it and chose to decide later, with the trigger named |
| Agreed | Owner wants it, needs a ticket |
| Rejected | Owner ruled it stays as is, with the reason kept |

---

## S1. The biller raises future bills for bookings nobody confirmed

**Status:** Proposed, 2026-08-23. Owner raised it
**Where:** `src/scripts/chargeRentTenants.ts`

`chargeRentTenants` charges `status IN (1, 2)` and never looks at the booking's approval state, so it
raises rent for bookings that are still awaiting approval and may never happen.

**Why it looks wrong.** The owner's reasoning on D1 in the log applies to the biller as much as to
the forecast: a future bill against a booking nobody has confirmed is a bill against something that
may never happen.

**Proposed:** the biller should not create future bills for unconfirmed bookings. Bills for the
current period stay, since those record something that already happened.

**This is a feature, not a bug fix.** It changes what gets billed, so it needs its own decision, a
view on bookings already billed forward, and a view on what an operator sees when a booking is later
confirmed.

**If accepted, D1 gets simpler**, because the forecast and the biller would agree automatically, one
rule shared instead of two kept in step by hand.

## S2. Cancelling a booking leaves its dues behind, and nobody is told

**Status:** Proposed, 2026-08-23. Owner raised it and asked whether it was already fixed
**Where:** `src/services/tenant/tenant.ts:3706` (`rejectBooking`), which calls
`src/controllers/tenant.ts:21248` (`deleteActiveTenant`)

**What happens today, verified:**

1. Rejecting a booking writes `is_confirmed = -1` on the confirmation row, then calls
   `deleteActiveTenant` with `tenant_uuid`, `pg_number`, `remark` and `pg_id`, and nothing else.
2. `deleteActiveTenant` moves the tenant to status 0 with `reason_of_eviction = 'Booking cancelled'`.
   So the person does leave the booking pool. Production agrees: only 2 tenants sit at status 2 with
   a cancelled confirmation.
3. Their unpaid bills are not touched. They survive the cancellation and reappear as **Old tenants**
   money on the Dues screen.

**On whether it was already fixed.** `deleteActiveTenant` does have a path that clears unpaid bills,
but it runs only when the request carries `delete_invoices: true`. The booking-cancel call never
sends it. The manager app appears to send it from a team-member permission of the same name when a
staff member deletes a tenant directly. So the dues go or stay depending on which screen the person
came from, not on what happened to the booking. That is very likely the shape of the problem Jatin
reported.

**A second defect inside that path.** The comment above it says "update all invoices for this tenant
to status 4 if their existing status is 0", meaning write them off. The code below calls
`InvoicesRepository.delete(...)`, a hard delete. Comment and code disagree, and a hard delete
destroys the record of what was billed.

**Proposed:**

1. Cancelling or rejecting a booking clears that booking's unpaid dues as a rule of the cancellation,
   not as a flag some callers send.
2. The operator is told it happened, with the amount, rather than finding out from a total that moved.
3. Clearing means writing off, not hard deleting, so the history survives and the number can be
   explained later.

**Open question for the owner:** paid bills against a cancelled booking, a token or an advance, are a
refund question, not a deletion question. This proposal covers unpaid bills only.

## S3. Rename "Under notice" to "Under eviction", with two named parts

**Status:** Agreed, D28, 2026-08-25. Owner raised it, and ruled it once all five tabs were verified
**Where:** Tenants (DA-09), and any screen that shows the parent group

The owner's suggestion:

> Under notice can also be converted into under eviction, which can have two sub-slices: approved
> eviction, pending eviction.

**Why it is attractive.** D6 has already named the two parts, Eviction pending and Eviction approved,
and put them on Dues and Collection. If the parent group on Tenants is still called Under notice, the
suite has one word for the whole and a different word for both of its parts. Renaming it makes the
whole and its parts share the same first word, so a manager can see that the two bars on Dues add up
to the one group on Tenants without being told.

**Why it was parked.** Counted case-insensitively on 2026-08-24, "under notice" appeared 21 times in
the Inventory sheet, 15 in Tenants, 10 in Collection and 2 in Dues after the D6 edits, plus tile
notes and empty-state copy in shipped code. That is a suite-wide sweep and it should not ride along
inside a build verification. Deciding it after all five tabs were verified meant the sweep happened
once, against a settled picture.

**Settled 2026-08-25, D28 in the log.** The owner accepted the rename. The sheets are swept, DA-09,
DA-08 and DA-01, with historical narrative left as written. What remains is the shipped copy: six
visible strings on the Tenant screen (the Journey bar label, the View all row, the rent at risk
note, two empty messages, one hint), plus the D6 and D21 label changes already owed on the other
tabs. D28 in the log carries the reasoning, and the rule that the rename changes words and not
numbers: nobody moves between groups.

## S4. Nobody can see deposits owed back to tenants who have already left

**Status:** Proposed, 2026-08-24. Falls out of D10
**Where:** the product as a whole, not one screen

In the system of record, deposit is still held against 138,925 paid deposit invoices whose payer has
moved out, and 133,886 of those have no settlement of any kind recorded. Numbers, scope and the
owner's ruling that these are genuine are in F22 and D10 in the log.

That is money the business owes people who are no longer customers. Today no screen shows it. The
Deposit Dues card will gain a line for it under D10, but a line on one analytics card is a report,
not a workflow.

**What is missing is the work, not the number.** An operator seeing "held for tenants who have left"
has no way from there to see who they are, how long each has waited, or to act. Deposit refunds owed
are the kind of obligation that turns into a complaint, a bad review or a legal notice, and they get
older quietly.

**Proposed, in order of what it takes:**

1. A list behind the number: who, how much, how long since they left.
2. Ageing on it, the same idea as the Overdue Breakup buckets, because a deposit unreturned for nine
   months is a different problem from one unreturned for nine days.
3. A prompt when a tenant is moved out and their deposit has not been settled, so the number stops
   growing at the source.

**Expect pushback when the line goes live.** Some managers will say the deposit went back in cash and
was never recorded. The product's own exit flow reads the same two records, refunds and adjustment
payments, so there is nowhere else in the system a settlement could be. The line is right about the
system of record. Whether it is right about the cash is a recording question, and item 3 above is
what fixes it at the source.

## S5. Rename the rollup row so "Others" means one thing

**Status:** Parked, 2026-08-24. Owner ruled the collision stays for now (D18 in the log)
**Where:** Expense Breakdown, Category tab. Very likely Dues and Collection too, unchecked

The Category card shows the three largest groups and folds the rest into a row called **Others**.
Separately, people type a category called Other or Others: 4,566 expenses worth ₹3.97 crore over the
twelve months to 24 August 2026. So on 111 properties the card carries two rows reading Others, one
being what people typed and one being everything else, and they open different things. On another
362 the typed one turns up inside the Others sheet.

**Why it is parked, not fixed.** Ruled in D18 in the log, where the reasoning lives. In short: the
only repair that does not break §14 is to rename the rollup, and the owner ruled the anomaly is
tolerable and not worth holding the Expense build for.

**Why it should not be forgotten.** Dues Breakdown and Collection Breakup fold their own rollup into
a row called Others over a free-text field, exactly as this one does. Nobody has checked whether a
due type has ever been typed as "Others". If it has, three tabs have the same collision and the
rename becomes one decision made once rather than three made separately.

**The trigger:** decide it when the Dues and Collection category grouping is next opened, which is
already owed there for F35 and F19.

## S6. Nothing records the business paying its staff back

**Status:** Proposed, 2026-08-24. Falls out of F53
**Where:** Team Passbook, not the analytics screen

Staff front a quarter of all spending from their own pockets, ₹48.65 crore over twelve months. What
the business has not paid back reads **₹50.77 crore across 1,185 live properties**, and almost nothing has ever
been taken off it: 1,165 properties positive, 20 at zero, not one negative. The reason is that the way to
record settling up only appeared in March 2026 and has been used 23 times platform-wide since.

So the new Still owed to staff tile will show a very large number that only grows, and the first
thing many operators will say is "I paid him back months ago, in cash".

**This is the same shape as S4.** A real obligation, correct about the system of record, with no
workflow behind it, so it ages quietly and arrives as a complaint about the number.

**Proposed, in order of what it takes:**

1. Make recording a payback easy and obvious from the passbook, so the tile can fall.
2. A prompt when the amount owed to one person crosses a threshold or gets old.
3. Ageing on it, the same idea as the Overdue Breakup buckets.

Item 1 alone decides whether the tile is a live figure or a growing monument. It is worth settling
before the tile launches, not after.

## S7. Nothing asks how long an agreement runs, so the gap keeps growing

**Status:** Proposed, 2026-08-24. Falls out of F77, narrowed by D24
**Where:** the move-in flow, not the analytics screen

Two screens now show agreements by when they end: Agreements ending soon on Inventory, and Agreement
Expiry on Tenants. Both work out the end date the same way, in four steps: a recorded renewal date,
then the length written on the tenant, then the property's default length, then **11 months** where
none of those exist.

**Measured on production**, living long-term tenants on live properties, test and deleted excluded, on
24 August 2026: **190,609 of 351,108, or 54.3%, have no agreement length recorded anywhere**, so the
product falls back to a flat eleven months and puts them on a bar anyway.

**Why it looks wrong.** An eleven-month default is a reasonable guess about the Indian rental market
and a poor claim about a named tenant. §17 of the Inventory sheet forbids guesswork in any forward
number in as many words, and §10 describes these dates as "facts about paperwork". For the majority of
tenants they are not.

**The back catalogue is already handled**, by D24 in the log: the Agreements card's info sheet names
the eleven-month fallback, carries the count of tenants on it, and that count opens the list so a
manager can fill the lengths in one at a time. That fixes the tenants who already exist.

**What is left is the tap that stops it happening again.**

**Proposed:**

1. Ask for the agreement length where the agreement is created, with the property's default filled in
   so the common case is one tap and nothing new to type.
2. A prompt on a tenant record being edited for another reason, where no length is recorded.

**Why it still matters after D24.** D24 gives managers a way to fill the gap; it does not stop the gap
growing. Every move-in recorded without a length adds another tenant to the list D24 built to empty.
Item 1 is where the number stops rising, and it is the cheapest of the three suggestions in this
register to build.

**This is a milder version of S4 and S6.** Same shape, a fact the product never captured and a
workflow missing where it should have been captured, but unlike those two the repair path now exists,
so this is about the inflow rather than the backlog.

## S8. The agreement-length setting accepts anything, and one "." took a whole tab down

**Status:** Proposed, 2026-08-25. Falls out of F90
**Where:** the property settings screen, not the analytics screen

The property's default agreement length is stored as free text. On live properties today it holds
"." on one property and negative numbers, "-5" and "-37", on 61 more. The "." crashes every
analytics block that computes an agreement end date, which takes the whole Tenant page and an
Inventory card down for that property (F90 in the log). The negatives quietly put tenants' computed
end dates years before they joined.

The analytics fix is a guard in two queries, and it is owed under F90 regardless. This suggestion is
the source fix: the settings field accepts whole months in a sensible range and nothing else. Same
shape as S7: a fact captured badly where capturing it right costs one input rule.

**Measured on production**, live properties, 25 August 2026: 62 properties carry a value that is not
a plain positive number, with 1,452 active tenants on them.

## S9. Journey's eviction split leaves out the people whose date has passed

**Status:** Ruled out of the build, 2026-08-25, with one question left open. Falls out of D29, the
ruling that gave Journey its six rows
**Where:** Journey, Tenants tab

D29 gave the card four rows that are meant to add up: Active tenants, and its three parts, No
notices, Eviction pending and Approved eviction. There is a fourth part it does not show: tenants
whose approved leaving date has passed and whose record was never closed. They are still living
there and still counted in Active, which is what §3 of the sheet says they are.

**Measured on production**, live properties, test and deleted excluded, 25 August 2026:

| | Tenants | Properties |
|---|---|---|
| Active tenants | 352,886 | 26,435 |
| No notices | 346,621 | |
| Eviction pending | 1,198 | |
| Approved eviction | 4,749 | |
| **Past their date, not on the card** | **318** | **151** |

The three rows under Active add up to it on every property where nobody is past their date, which today is
26,284 of 26,435. On the other 151 the rows visibly fall short, and a manager who adds them up finds
the gap with no explanation for it.

**Why it is a suggestion and not a requirement.** The owner ruled it that way. §5 of the sheet gives
Past their date the Overview tile slot and says no other card surfaces it, so adding a row here is a
change to a rule that was made on purpose, not a defect to fix. The card is also already carrying
repetition from the tile strip by design, and this would add a fifth repeat.

**Open for you, one choice.** Leave the card as ruled and add a one-line note saying the rows exclude
people whose leaving date has passed, or add a fifth row, Past their date, inside the first group so
the rows add up on every property. **My pick is the note**: it closes the arithmetic gap without
breaking §5's rule that Past their date lives on one tile and nowhere else, and it costs a line
instead of a row on a card that is already carrying six.

## S10. Only some rows that open something look like they open something

**Status:** Proposed. First found on Expense 2026-08-07, moved here from the tracker 2026-08-27
**Where:** the analytics screens as a family, most likely one shared row component

On Expense, only the **Others** rows carry a chevron, while every row on every card opens something.
A manager reading the screen has no way to tell which numbers are doors and which are just numbers,
so the ones without a chevron read as dead ends and go untapped.

**Why it looks wrong.** The whole point of these screens is that a number opens the records behind
it, which is what the tap matrix on every sheet exists to specify. An affordance shown on one row
type and not the others teaches the manager the opposite of the rule.

**Why it is probably one fix, not five.** The rows are almost certainly a shared component, which
means it is likely wrong on every screen in the same way and right in one place.

**Not verified beyond Expense.** This was seen on one screen and never checked on the other four.
That check is the first thing to do, because it decides whether this is a component change or five
screen changes.

**Proposed:** every row that opens something carries the affordance, and the tap matrix on each sheet
becomes the list of which rows those are.

## S11. The list a drill lands on does not say what it is showing

**Status:** Proposed. First found on Collection 2026-08-06, moved here from the tracker 2026-08-27
**Where:** the manager web app's list screens, not the analytics screens

Checked against the live collections list: a manager arriving from a widget tap sees a page titled
**"Collections Overview"**, fixed, whatever filter brought them there. Chips naming the active filters
exist but render only **after the person scrolls**, and filters that arrive from tapping a widget
produce **no chip at all**. An empty filtered slice shows one sentence, **"There are no collections
present."**

**Why it looks wrong.** The manager tapped a specific number and landed somewhere that will not say
which number they tapped. Worse, the empty state reads as "you have no collections at all", which is
false for a property that collected lakhs that month. That undoes on the next screen exactly what the
analytics screens are built to do.

**Not verified beyond the collections list.** The other list destinations have never been checked, and
the same component very likely serves several of them.

**Proposed**, and it is the same three things the tracker's R67 already requires of any drill
destination: name the slice in place of the fixed title, name the filter in the empty state with a way
on, and show the active filter on arrival without scrolling.

**Why it is a suggestion and not a defect against a sheet.** No handoff sheet specifies the
destination screen; each one specifies only what its own numbers open. The gap sits in the space
between two documents, which is why it went unowned for three weeks.

## S12. Pages below the first list do not all check permissions the way the list does

**Status:** Proposed. First found on Collection 2026-08-06, moved here from the tracker 2026-08-27
**Where:** the drill destinations, below the first list

The first list a drill opens checks the viewer's permission. Pages deeper than that do not all do the
same. Nobody has enumerated which ones do and which do not.

**Why this one is different from S10 and S11.** Those two are about a manager not being told enough.
This one is about a manager possibly being shown records they are not entitled to, which is a
different order of problem and should not sit in the same queue as a missing chevron.

**Unverified, and that is the finding.** This has been carried as an open question since 6 August and
nobody has checked it. **The first action is the enumeration, not the fix:** list every page reachable
below a first list, and record for each whether it checks and against what.

**Open for the owner, one choice:** do the deeper drills ship on day one with this unresolved, or does
the enumeration run first? **My pick is the enumeration first**, because it is a few hours of reading
and the alternative is shipping a permission question nobody has an answer to.

**What is already settled, and is not this.** The analytics screens themselves are gated correctly:
each tab follows the permission of the records it describes, and a team member's scope is narrowed to
the properties they can open and hold the matching view permission for. That was verified in the
analytics permission code on 27 August 2026. This entry is only about what sits below the lists those
screens drill into.
