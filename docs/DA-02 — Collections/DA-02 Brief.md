---
title: DA-02 Collections - Brief
date: 2026-06-06
tags:
  - rentok
  - brief
  - collections
status: Living document - v1.0
owner: Sanchay
time_budget: 2-week build cycle
companion_to: DA-02 Ground-Truth Formula Map, DA-02 Build Sheet
---

> [!WARNING] Superseded in places, as of 7 August 2026
> **The Collection Handoff Sheet in this folder overrides this document** on the points below. Where the two disagree, build what the handoff says. These corrections have not been folded back in here yet.
>
> 1. **The RentOk-credit maths.** The formula recorded here is copied from the live screen, which subtracts the processing charge twice. Only one charge is real. The handoff builds on the corrected maths, with finance signing off before the fix ships.
> 2. **There are four kinds of adjustment, not three.** Deposit, advance, caution money and discount all count toward Collected & Adjusted in Due Date view. Discount is its own total, never a payment method. And every rupee lands in exactly one category row: caution money sits in the deposits row only, not in two places.
> 3. **Collection Status has no unpaid row.** The "Unpaid dues" row specified here was removed: what is still owed is the Dues screen's job. Still Unpaid lives as an Overview tile in Due Date view and opens the Dues screen, carrying the period with it.
> 4. **Which date each measure follows changed with the two-view model.** The screen carries a Paid Date / Due Date toggle that changes what several cards mean, and it hides or reshapes cards that collapse in Due Date view. Advance, Current FY and Settlement Pending always count by when money arrived, in both views. The single paid-date model here predates all of that.
> 5. **The word "settled" is confined to the Payment Settlement card**, where it means money that has physically reached the owner's bank. Everywhere else a bill being dealt with is "collected or adjusted", never "settled". Banks are never predicted for unsettled money, and properties on the older settlement system show a message instead of bank rows.
>
> Also decided only in the handoff, with no equivalent here: the trend chart is one stacked bar per period, not two bars; the View all sheet's full contents; per-card loading and failure states; and change comparisons made against the same point of the previous period, so an unfinished month is never measured against a finished one.



> [!INFO] What this is
> A **Brief** for DA-02 Collections. It says what this screen must help the user understand before we talk about layout or tasks.
>
> **What this is NOT:** a design file, a task list, or a bank-reconciliation product. The detailed formulas live in `[[DA-02 Ground-Truth Formula Map]]`. Build tasks live in `[[DA-02 Build Sheet]]`.

## 1. In one line

Managers can already see that money came in. What they cannot see, in one trustworthy period view, is **what kind of money it was** — this month's dues, old dues recovery, early payment, advance, or a deposit/advance adjustment that cleared a bill without new cash coming in. **2 weeks to ship V1.**

*Main question:* *"Is this real collection, or did the number go up for some other reason?"*

## 2. Who this is for

This screen is for the owner, property manager, or finance-minded user when they want to **understand what kind of money came in**. It is not for recording payment. It is not the page for working through one payment at a time.

**Main user:** a single-property or small multi-property owner with invoice access. They already know the top-line collection number exists somewhere. Their problem is trust and meaning. They want to know whether the month is healthy, whether staff-recorded cash looks normal, and whether money has actually reached the bank.

**Second user:** a larger owner checking which property needs follow-up. They use this to compare properties, then open the existing collection list or speak to staff.

**Where it sits in the product:**
- The homescreen financials block says, "collection happened."
- The collection list says, "here are the individual payments."
- DA-02 says, "what shape did that money take in this period?"

**Click-through rule:** when people want detail, they should land on the main page people land on when they want detail — an existing collection page they already use. This screen should not become a second list of individual payments.

## 3. The problem

Three pains, same root cause: the app has collection data, but not one trusted reading of it.

1. **The total can be true and still be misleading.** A manager sees one number, but cannot tell whether it came from this month's dues, old arrears, advance, or an internal adjustment. That makes the number hard to act on.
2. **The app mixes real cash with bill adjustments too loosely.** Deposit and advance adjustments clear dues without new cash coming in. If these sit too close to cash collections, people read the period as healthier than it is.
3. **Money received is not the same as money settled.** For online collections, the user still needs to know whether the payout reached the bank, is in process, or is stuck.

The point is not to show more charts. The point is to remove ambiguity from a number people already watch.

**Why now:** Collections is one of the few numbers people review every cycle without fail. When they cannot explain the number, they fall back to WhatsApp, bank screenshots, export files, and memory. That is slow, and worse, it breaks trust.

## 4. What people do today

- **Checks the homescreen total, then stops there.** They get the number, not the reason behind it.
- **Scrolls the collection list and does mental grouping.** They try to separate rent, deposit, old dues, and staff handling by hand.
- **Pulls a collection report or tenant ledger export.** This works for month-close review, but not for a quick period read.
- **Asks staff who collected what.** Especially when cash is involved, people still rely on people, not the product.
- **Checks settlement messages separately.** Online payment received and money-in-bank are not read in one place.

Today people have data, but not a clean answer.

## 5. What we must ship - and what we cut if time runs short

**The goal is to ship all 7 questions below.** The tiers here are the fallback if the cycle slips, not the planned scope.

**Must ship (V1 is not V1 without these):**
- **Q1 - What was net collection in the selected period?** One trusted total for the period, net of refunds.
- **Q2 - What made up that collection?** Split the period's received money into current-period dues, past dues, future dues paid early, and advance. Show paper adjustments separately.
- **Q3 - How much of the billed period was actually collected?** People need a bill-side reading, not only a money-in reading.
- **Q4 - What still has not settled?** Unsettled and in-process online money belongs inside the collection story, not in a separate accounting rabbit hole.

If we cannot ship those four with one shared definition, we should not ship DA-02.

**Nice to have (keep if the cycle holds):**
- **Q5 - What did the mix look like by category, mode, status, and received-by?**
- **Q6 - How is the period trending over time?**
- **Q7 - Which property is driving or dragging the number?**

These sharpen the picture after the core reading is trustworthy.

**If time runs short, cut in this order:**
1. **Q7 - Property comparison.** Useful for multi-property owners, but not needed for the single-property core read.
2. **Q6 - Trend.** Helpful for monthly review, not required to explain today's selected period.
3. **Received-by inside Q5.** Solo owners can live without it for one cycle.
4. **Status inside Q5.** It explains who paid, but not as directly as source mix or settlement state.

Do not cut Q1 to Q4.

**If Q5 must split by user type:**
- **Solo owner:** keep Mode, cut Received-by first.
- **Owner with staff:** keep Received-by, cut Mode first.
- **Default:** keep both.

## 6. What each question needs

Each line below describes the information, not the layout.

1. **Net collection.** One period total that subtracts refunds and does not count deposit or advance adjustments as fresh cash.
2. **Source mix.** A split of the same period total into current-period dues, past dues recovery, future dues paid early, and advance received.
3. **Adjusted collection.** A separate reading of money cleared by deposit, advance, or caution adjustments, so people do not confuse it with cash inflow.
4. **Collection status.** A period reading that compares what was billed for the period against what has been collected for that same billed period.
5. **Breakups.** The same money-received total shown by due type, payment mode, tenant status, and received-by.
6. **Settlement state.** Online collections grouped into settled, unsettled, and in-process states.
7. **Property view.** The same period reading shown property by property when more than one property is selected.
8. **Trend.** The same definition repeated month by month so people can compare periods without meaning drift.

## 7. What we are not building this cycle

- No new payment-recording flow. The existing payment flows already own data entry.
- No full bank ledger or cash-flow product. DA-02 explains collections; it does not replace finance reports.
- No predictive alerts or push notifications. People already know when to open this screen; the issue is clarity after opening it.
- No new duplicate list view. Clicks for more detail should reuse existing list and detail pages.
- No per-bank routing prediction. V1 should first show settlement state clearly before promising bank-by-bank allocation.

## 8. Things that can go wrong

*USER = a risk the user feels. TEAM = a risk only the build team sees.*

- **USER - One number, one definition.** Collections is currently calculated differently across the list, widget, and homescreen. DA-02 must not ship on top of three competing answers.

- **USER - Adjustments are not fresh cash.** Deposit adjustments, advance adjustments, and caution-money adjustments clear dues without new money coming in. DA-02 must treat all three as adjustments, never as cash inflow.

- **TEAM - Some old adjusted-collection logic is not safe to reuse as-is.** One of the old adjusted-collection methods is visibly broken today. That means the new DA-02 adjusted-collection logic should come from cleaned shared rules, not from old behavior copied forward.

- **USER - Refunds must reduce the story everywhere.** Refunds are a real part of the payment story. If DA-02 shows net collection, refund subtraction cannot be optional by section or endpoint.

- **USER - Settlement is part of the collection story.** People read "money received" and "money settled" together, so DA-02 should too.

- **TEAM - Some deeper pages have weaker access checks than the main collection list.** DA-02 should not send people to weaker pages and call that done.

| Risk | How sure are we? | Mitigation |
|------|------|------------|
| Will people use it? | HIGH | They already review collections. The missing piece is explanation, not demand. |
| Will they understand it? | MEDIUM | Keep the screen anchored on what money came in, what bills it relates to, and whether it settled. Separate adjustments clearly. |
| Can we build it in 2 weeks? | MEDIUM | Build one shared total first, then the views on top of it, and do not leave access cleanup for the end. |
| Is it worth it? | HIGH | A collection total that cannot be explained creates avoidable mistrust every cycle. This is the strongest case of the four — the cost of an unexplained number is paid every single cycle. |

**Footer**

This is a **read-and-understand screen**, not a page for taking action on one payment.
People will usually open it from the homescreen collection total, a collection report, or confusion about whether received money has settled.
After reading it, they should usually go to the collection list, a receipt, a settlement page, or a staff follow-up.
People will likely open it 2 to 3 times a week in normal operation, and more often near month-close.

Assumptions used here:
- `2-week` cycle taken from the prior DA-02 draft and kept unless product changes it.
- V1 keeps the existing collection list as the main page people land on when they want detail.
- V1 improves trust before it expands into full accounting coverage.

## Changelog

- 2026-06-06 - Rewritten from backend and Figma grounding. Reframed DA-02 as a period-reading screen for collection meaning, not a second collection list.
