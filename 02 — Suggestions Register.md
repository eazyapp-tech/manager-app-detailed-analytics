# Suggestions Register

Problems in the shipped product that are **not** failures against a handoff sheet. The sheet either
never covered them, or covered them and the underlying behaviour is wrong anyway. Found while
verifying the build, kept apart from [[01 — Build Verification Log]] so the defect list stays a defect
list.

**Nothing here is actioned without an owner ruling.** The owner is Sanchay. Each entry says what
happens today, why it looks wrong, and what is proposed.

| # | Suggestion | Status | Waits on |
|---|---|---|---|
| S1 | Stop the biller raising future bills for unconfirmed bookings | Proposed | Owner |
| S2 | Cancelling a booking should clear its unpaid dues and say so | Proposed | Owner |
| S3 | Rename "Under notice" to "Under eviction", with two named parts | Parked | Owner, after all five tabs are verified |
| S4 | A list, ageing and a prompt for deposits owed to tenants who have left | Proposed | Owner |

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

**Status:** Parked, 2026-08-23. Owner raised it. Decide after the five tabs are verified
**Where:** Tenants (DA-09), and any screen that shows the parent group

The owner's suggestion:

> Under notice can also be converted into under eviction, which can have two sub-slices: approved
> eviction, pending eviction.

**Why it is attractive.** D6 has already named the two parts, Eviction pending and Eviction approved,
and put them on Dues and Collection. If the parent group on Tenants is still called Under notice, the
suite has one word for the whole and a different word for both of its parts. Renaming it makes the
whole and its parts share the same first word, so a manager can see that the two bars on Dues add up
to the one group on Tenants without being told.

**Why it is parked.** Counted case-insensitively on 2026-08-24, "under notice" appears 21 times in the
Inventory sheet, 15 in Tenants, 10 in Collection and 2 in Dues after the D6 edits, plus tile notes and
empty-state copy in shipped code. That is a suite-wide sweep and it should not ride along inside a build verification. Deciding it after all five tabs are
verified means the sweep happens once, against a settled picture.

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
