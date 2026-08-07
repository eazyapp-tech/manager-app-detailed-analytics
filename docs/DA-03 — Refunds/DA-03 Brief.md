---
title: DA-03 Refunds - Brief
date: 2026-06-06
tags: [rentok, brief, refunds, detailed-analytics]
status: Living document · v3.0
owner: Sanchay
time_budget: Open - cut order ranked by owner pain
---

> [!INFO] What this is
> A product brief for the Refunds detailed-analytics screen.
>
> It explains what the owner needs to understand after money has gone back out. It is not a build spec and not a formula sheet.

## 1. In one line

A PG owner records refunds after money has already left, but he cannot see the full picture of that refunded money — how much, why, to whom, and whether it matches his other money screens.

He needs to know how much went out, why, how, who received it, which property's money went out, and whether it agrees with collections (DA-02), deposits held (DA-06), and cash-flow (DA-07).

So the bet is one thing: **make completed refunds readable as one money-out picture.**

*"Kitna refund gaya, kis wajah se gaya, kisko gaya, aur kya yeh baaki money screens se match karta hai?"*

## 2. The owner

This screen is for the owner when he wants to **review completed refunds**.

It is not for approving a refund. It is not for seeing tenants who still need their deposit back. It is not for editing a refund. Those jobs belong to the refund entry path, deposit-held liability view, and existing detail pages.

**Rajesh, primary user.** Owner of one PG. He opens this after move-outs, during weekly money review, or before month-end checking. His question is: *"Is this normal, and can I explain every big rupee amount?"*

**Priya, secondary user.** Multi-property owner. Her question is: *"Which property is returning the most money, and is it because of deposits, advances, rent corrections, or staff behavior?"*

**Meena, restricted manager.** She may record refunds, but should see only the data her access allows. The screen should show a clear restricted state, not a broken one.

These personas are RentOk owner and team composites, not fresh interview quotes.

Where this sits: collections says money came in; deposits held says deposit money is still owed later; DA-07 says money moved in and out; this screen says money has already gone back out as refunds.

## 3. The problem

Three pains, same root cause:

1. **Completed refunds disappear into scattered entries.** Rajesh can record a refund, but later he cannot quickly explain total refunded money for a period.
2. **Different refund reasons need different reads.** A security-deposit refund after move-out is normal. An active-tenant rent correction, duplicate-looking refund, or staff-fronted refund needs checking.
3. **Refunds touch other money screens.** Deposit refunds reduce deposits held, lower net collections, and appear as money out in DA-07. If these screens disagree, trust breaks.

The screen should turn refunded money into clear review lanes.

## 4. What Rajesh does today

He works through scattered views:

- opens bill or tenant details one refund at a time
- asks staff *"iss hafte kitna refund diya?"*
- checks deposit-held numbers separately
- checks collections and DA-07 separately
- misses deleted refund history unless he already knows which refund to inspect
- sometimes sees deposit-refund-like expenses mixed into expense reports

This is slow and easy to misread. The new screen should show the refund picture first, then send the owner to the matching refund list.

## 5. What must ship

**Must ship. V1 is not V1 without these:**

- **Q1 - How much was refunded?** Total amount, refund count, tenant count, and prior-period comparison.
- **Q2 - Why was money refunded?** Deposit, caution money, advance, rent or fee correction, utility or service correction, and Other. Goodwill/manual appears only after product approves the phrase mapping.
- **Q3 - Who received it?** Tenants ranked by refunded amount, with tenant state kept clear.
- **Q4 - Which property's money went out?** Multi-property split, with amount and share.
- **Q5 - How did money go out?** Cash, digital, bank, card, cheque, and other modes.
- **Q6 - Which refunds need review?** Big single refunds, active-tenant refunds, repeat refunds, and possible duplicate refunds.

These six answer: *how much -> why -> who -> where -> how -> what needs checking.*

**Good to ship if the cycle allows:**

- **Processed By.** Staff or owner who recorded the refund, plus unattributed refunds.
- **Fund source.** Petty cash, tenant-held funds, or staff personal money.
- **Deposit-held impact.** For deposit and caution-money refunds, show the liability reduction.
- **DA-07 bridge.** Show refunds as money out.

**Cut order if time runs short:**

1. DA-07 bridge - DA-07 still carries the money-out total, so Rajesh can wait one cycle.
2. Deposit-held impact - deposit refunds still appear in the Reason split, and DA-06 still owns the liability answer.
3. Fund source - important for staff trust, but amount, reason, tenant, and mode still let Rajesh review the money.
4. Processed By - useful for staffed PGs; less useful for owner-run PGs.

Do not cut Q1 to Q6 before all four above are gone.

## 6. Product decisions

**Completed money-out events.** A refund entry means the money was recorded as already returned. V1 has no pending, failed, or in-progress refund state.

**Use refund date.** The screen uses the date entered for the refund, not bill paid date, due date, save time, or delete time.

**Pending obligations stay out.** A moved-out tenant whose deposit is still held belongs to Liabilities / Deposits Held, not Refunds.

**Deleted refunds stay out.** Deleted or reversed refunds are audit history. They should not make the live refund total larger or smaller.

**Reason from the linked bill.** Refund entries do not carry a clean reason category. The screen reads the linked bill type and maps it into plain refund reasons. Manual/goodwill labels need an approved remark mapping before they become their own line.

**Refunds reduce net collections.** A collection tied to a refunded bill should not be counted as if all money stayed collected.

**Deposit refunds reduce deposits held.** Security Deposit and Caution Money refunds lower the liability still held for tenants.

**Expenses are separate.** If staff also enter a deposit refund as an expense, DA-03 still counts the refund once. The duplicate risk is a QA and DA-07 issue, not a reason to count it twice.

## 7. What we are not building this cycle

- **No pending refund queue.** There is no pending refund state in V1. Building one would require a new refund lifecycle.
- **No refund approval workflow.** Owners record refunds after deciding and paying. Approval is separate.
- **No deleted-refund audit panel in the main total.** Deleted refunds matter, but they are history. Put them behind an audit entry point if the cycle allows; never mix them into live refund totals.
- **No new accounting ledger.** The screen explains links to collections, deposits held, and DA-07. It does not create a separate books layer.
- **No duplicate expense cleanup inside this screen.** The screen can flag that duplicate-looking deposit-refund expenses exist; cleanup remains in the existing expense path.

## 8. Traps and risks

**Traps decided now:**

- **A pending-refund label/tag would be wrong.** It would imply a lifecycle the product does not have.
- **Refund date is the time rule.** Mixing bill dates and refund dates will make DA-03 disagree with DA-07 and owner memory.
- **Live refunds and deleted refunds must stay separate.** Deleted refund history helps audit who removed a refund; it does not define current refunded money.
- **Deposit refunds need a bridge, not a second definition.** DA-03 says money went out. DA-06 says liability went down. Both can be true.
- **Collections must be named carefully.** Refunds reduce net collections, but the gross collection still happened. The screen should not make owners think the collection never existed.
- **Every follow-up must match the number tapped.** If a line says ₹X and the list opens a different slice, the screen fails.

**Risks:**

| Risk | Read | Mitigation |
|---|---|---|
| Owners expect pending refunds | Medium | Say clearly that this screen shows only money already returned; route unpaid deposit obligations to deposits held. |
| Collections and refunds feel contradictory | High | Label gross collection, refund, and net collection separately. |
| Deposit refunds are counted twice with expenses | High | QA must test deposit-like expenses plus refunds for the same tenant and period. |
| Staff attribution is incomplete | Medium | Show an unattributed state when the money trail is missing. |
| Deleted refunds are mistaken for live refunds | Medium | Keep deletion history behind audit language and separate totals. |

### Footer

**Things to test with owners before launch:**

- Do owners understand that Refunds means completed money already returned?
- Does the Reason split match how owners speak: deposit, advance, rent correction, service correction, and manual refunds?
- Does the active-tenant alert feel useful, or does it fire too often?
- Does the collections bridge read clearly: gross collection happened, refund reduced net collection?
- Does a restricted manager understand the restricted state?

**Locked decisions:**

- Refunds = completed money-out entries.
- Time basis = refund date.
- No pending refund lifecycle in V1.
- Pending deposit obligations belong to deposits held.
- Deleted refunds are audit history, not live refunds.
- Reason comes from linked bill type.
- DA-03 total must match live refunds for the selected period and property set.
- Deposit refunds bridge to DA-06; refund totals bridge to DA-02 net collections and DA-07.

## Changelog

| Date | Version | Change |
|---|---|---|
| 2026-06-06 | v3.0 | Rebuilt as a self-contained product brief. Locked completed-refund meaning, refund-date time rule, no pending lifecycle, deleted-refund separation, and bridges to DA-02, DA-06, DA-07, and expenses. |
