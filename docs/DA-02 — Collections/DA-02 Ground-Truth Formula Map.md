---
title: DA-02 Collections - Ground-Truth Formula Map
date: 2026-06-06
tags:
  - rentok
  - collections
  - formula-map
  - analytics
status: v2.0 - source of truth for DA-02 definitions
owner: Sanchay
companion_to: DA-02 Brief, DA-02 Build Sheet
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
> This file defines **what each DA-02 number means**. It says which fields matter, which date each number follows, and where the current product still disagrees with itself.
>
> The brief says **why** this screen exists. This file says **what each number is**. The build sheet says **how engineering should ship it**.
>
> Use this when you need the exact meaning of a DA-02 number, which date it follows, or where the current app disagrees with itself.

## 1. Read this first

If two cards look like they show the same collection number, they should match.
If a number includes refunds in one place, it should include refunds everywhere.
If money did not actually come in, it should not be shown as fresh collection.

## 2. Rules every number follows

### Rule 1 - Net collection is refund-aware received money

If DA-02 says **collection**, it means money received in the selected period after subtracting refunds tied to the same paid invoices.

- Refunds are real payment records, not just notes or comments.
- **Which path is refund-aware and which is not:**
  - The list grand-total path is **NOT refund-aware** — `executePaginated` (`collections/helpers.ts:419-429`) sums `raw_total_amount` with no refund join.
  - The widget aggregate **IS refund-aware** — it subtracts refunds via `ref_agg = inv.amount − COALESCE(total_refunded, 0)`, filtering `r.amount > 0` (`collections/helpers.ts:713-724`, `:930-960`).

**For DA-02:** use the refund-aware definition everywhere. Take the widget's refund-subtraction approach; do not reuse the list grand-total path, which omits refunds.

### Rule 2 - Paid date and due date do different jobs

Two dates matter:

- **Paid date** answers: when did money come in?
- **Due date** answers: what bill period was that money for?

**For DA-02:** overview totals, trend, settlement, mode, received-by, and property use **paid date**. The split between current dues, past dues, future dues, and advance should come from the bill date inside the selected paid-date set.

### Rule 3 - Adjustment modes are not cash inflow

These payment modes clear invoices without new cash arriving (labels at `helpers/payments.ts:812-819`):

- `211` - Adjusted from deposit
- `288` - Adjusted from advance
- `291` - Adjust from Caution Money

**Exactly how the three modes are handled today (the real split, not "inconsistent"):**

- **List grand-total path** (`resolveAdjustmentModes`, `collections/helpers.ts:615-624`; `executePaginated` `:419-429`) excludes only `211` and `288`, and **never references `291` at all**.
- **Widget aggregate** (`collections/helpers.ts:824`, `:971-1016`) and **homepage** (`v1/homepage/service.ts:2449-2452`) exclude all three: `211`, `288`, `291`.

**DECISION:** DA-02 standardizes on excluding all three `{211, 288, 291}` everywhere. The cash-exclusion target predicate is `payment_mode NOT IN (211, 288, 291)`.

> [!WARNING] Broken SQL in the old adjustment filter
> `helpers.ts:292` (inside `DEPOSIT_ADJUSTED_THIS_MONTH`) contains literally malformed SQL: `query.andWhere('p.payment_mode IN (211');` — an unbalanced paren and an unterminated string. This **throws at query time**, it does not merely mis-count. DA-02's adjustment logic must not inherit this path.

**For DA-02:** treat `211`, `288`, and `291` as **Adjusted Collection**, never as fresh cash collection.

### Rule 4 - One screen cannot change its starting point halfway through

Current product logic uses more than one starting point:

- One calculation starts from payments.
- Another starts from invoices.
- The homescreen uses its own grouped sum.

**For DA-02:** all numbers should come from one shared total, then be shown in different ways. The screen cannot use one meaning for the top card, another for trend, and a third for click-through detail.

### Rule 5 - Special mode `213` keeps its current money math unless payment rules change product-wide

The current product treats payment mode `213` as CirclePe transfer.

**For DA-02:** until payment logic changes across the product, keep the current `213` money math.

### Rule 6 - Clicks for more detail must not go to pages with weaker access checks

The current collection list is safer than some likely next-step detail pages. The safe entry point is the list service, which runs `checkAuth('viewInvoices')` and returns `401` on failure (`collections/service.ts:21-34`).

**For DA-02:** clicks for more detail should only go to pages with proper access checks, anchored on the `checkAuth('viewInvoices')` entry point above.

## 3. Base fields DA-02 reads

| Core field | Meaning |
|---|---|
| Successful payment only | Only completed money-received rows count. Payment side filters `p.status = 1 AND p.is_active = 1` (`helpers.ts:726-728`, `:954-965`). |
| Active payment row only | Inactive rows do not count (`p.is_active = 1`). |
| Received date | The date used for money-in-period questions (paid date). |
| Payment mode | The code used to separate cash-like modes from adjustment modes. |
| Received by | The recorder or receiver label shown to users. |
| Invoice amount | The billed amount on the invoice. |
| Invoice paid amount | The amount already cleared on the invoice. |
| Due date | The billed date used to classify current, past, and future dues. |
| Due type | The bill category. Exact strings folded into the measures below. |
| Invoice status | See the enum below. |
| Tenant status | See the tenant link set below. |
| Refund amount | The amount sent back after payment. |

### 3.1 The money primitive (define once, used everywhere)

**Collected amount** = `net_amount − gateway_charges` (in paise).

For mode `213` only, the collected amount is computed differently: `raw − owner_credits + credits_used − gateway_charges` (`collections/helpers.ts:494-503`).

This is the single definition of "money received on a payment row." Do not restate it as a vague "received amount."

### 3.2 Online modes and settlement state

- **Online modes** = `(203, 205, 206, 210)`.
- **Unsettled** = an online mode **AND** `payout_id IS NULL` **AND** `wallet_entry IS NULL` (`helpers.ts:317-321`, `:849-850`).
- **In-processing** = an online mode **AND** `payout_id IS NOT NULL` **AND** `payout.status != 1` (`helpers.ts:323-327`).
- **Settled** = `payout.status = 1` (`formatRow` `:515`).

`payout.status` codes (`formatRow:515-521`): `0` = reversed/failed, `1` = settled, `2` = processing.

### 3.3 Invoice status enum

`0` = Not Paid, `1` = Paid, `2` = Partial, `3` = Refunded, `4` = Loss (`invoices.ts:93`).

The legacy widget collection base filters `inv.status = 1 AND inv.amount > 0 AND inv.is_active = 1` (`helpers.ts:726-728`, `:824`, `:954-965`). See the headline-collection note in §4.1 — DA-02's headline does **not** reuse this `status = 1` invoice base.

### 3.4 Tenant link set

The tenant join uses `t.status IN (0, 1, 2, 5)` (`helpers.ts:157`, `:743`, `:961`). Status `5` = booking, so booking tenants **are** included. The status breakup (§4.2) inherits this same link set.

### 3.5 Due-type exact strings (fold in literally — never write 'Deposit')

| Category | Exact predicate | Anchor |
|---|---|---|
| Rent | `due_type = 'Rent'` | `helpers.ts:763-783` |
| Advance | `due_type = 'Advance'` | `helpers.ts:788`, `:996` |
| Deposit | `due_type IN ('Security Deposit', 'Caution Money')` | `helpers.ts:778`, `:989` |
| Electricity | `due_type ILIKE '%electricity%'` | `helpers.ts:763-783` |
| Food | `due_type ILIKE 'Mess%' OR due_type ILIKE 'Food%'` | `helpers.ts:763-783` |
| Late fine | `due_type IN ('Manual Late Fine', 'Automatic Late Fine')` | `helpers.ts:763-783` |

### 3.6 Adjustment mode labels

`211` = Adjusted from deposit, `288` = Adjusted from Advance, `291` = Adjust from Caution Money (`helpers/payments.ts:812-819`). Cash-exclusion target predicate: `payment_mode NOT IN (211, 288, 291)`.

## 4. Screen measures

### 4.1 Collection overview

> [!IMPORTANT] Headline collection = ALL money received (payment-side), locked this session
> DA-02 collection total is the **sum of successful payment rows in the period**, regardless of the linked invoice's paid status. The base is the **payment side** (`p.status = 1 AND p.is_active = 1`, scoped by paid date), **not** the legacy `status = 1` invoice base.
>
> This matters because partial receipts on still-open invoices (invoice `status = 0`, locked decision #1) **must count**. The legacy widget `status = 1` invoice base **undercounts partial receipts** and must not be reused for the headline.

Each "Formula" cell below is a stack of conditions — every line must hold.

| Measure | Plain meaning | Formula | Current product note |
|---|---|---|---|
| **Total collection in period** | Net money received in the selected period | • successful, active payment rows (`p.status = 1 AND p.is_active = 1`)<br>• paid date inside the selected received-date range<br>• collected amount per §3.1 (`net_amount − gateway_charges`; `213` special)<br>• refund-aware (subtract per Rule 1)<br>• exclude `payment_mode IN (211, 288, 291)` | List and homepage are not refund-aware everywhere; the list grand-total path also omits `291`. The widget is closest to target. |
| **Current-period dues collected** | Money received in the selected period for bills due inside that same selected period | • base set above<br>• then classify to due dates inside the selected period | The app does not group this today. Must be built. |
| **Past dues collected** | Money received in the selected period for bills due before the selected period | • base set above<br>• then classify to due dates before the period start | The app does not group this today. Must be built. |
| **Future dues collected** | Money received in the selected period for bills due after the selected period, excluding advance invoices | • base set above<br>• classify to due dates after the period end<br>• exclude `due_type = 'Advance'` | The app does not group this today. Must be built. |
| **Current FY collection** | Refund-aware received money inside the current April-March year | • base set above<br>• paid date within the FY range | Some views already show FY totals, but not yet with one shared DA-02 meaning. |
| **Settlement pending** | Online money received in the selected period that is not yet fully settled | • base set above<br>• online modes `(203, 205, 206, 210)`<br>• classify by payout / wallet / scheduler state per §3.2 | Pieces of this already exist in the product today. |

### 4.2 Collection breakup

All breakup tabs use the same **money-received total** for the selected paid-date period. Only the split changes.

| View | Formula | Notes |
|---|---|---|
| **Category** | Split the money-received total by invoice due type, using the exact strings in §3.5 (Deposit = `due_type IN ('Security Deposit', 'Caution Money')`, never literal `'Deposit'`) | These categories already exist in the collection data and should stay stable in DA-02. |
| **Mode** | Split the money-received total by payment mode, keeping adjustments out of cash modes | Use the payment mode labels already used in the product. |
| **Status** | Split the money-received total by tenant status from the linked tenant row, inheriting the §3.4 link set `t.status IN (0, 1, 2, 5)` (status `5` = booking is included) | This depends on the tenant row linked to the payment. |
| **Received by** | Split the money-received total by received-by label | This is the recorder or receiver field already used in collection pages. |

**If a payment is not linked to a tenant:** keep it in total collection and in mode / received-by views. For the status view, either show an `Unlinked` line or clearly say these rows are left out there. They must never disappear silently.

### 4.3 Collection status

This section answers a different question from inflow. It is about **what happened to the billed period**.

| Part | Plain meaning | Formula |
|---|---|---|
| **Current-period dues collected** | Selected-period bills that were collected | Received-money base classified to due dates inside the selected period |
| **Past dues collected** | Older bills recovered during the selected period | Received-money base classified to due dates before the period start |
| **Future dues collected** | Bills due later but paid now | Received-money base classified to due dates after the period end, excluding advance due type |
| **Advance collected** | Money received as advance | Received-money base classified to advance due type |
| **Unpaid dues** | Bills due in the selected period that remain unpaid | Sum of active invoice amounts still unpaid for due dates inside the selected period |

**If the UI shows collected vs billed, or a percentage:** the top line must use the same money-received meaning as DA-02 total collection, and the billed side must be the dues for the selected period.

### 4.4 Adjusted collection

Adjusted collection is its own card because it is real invoice clearance but not fresh cash.

| Measure | Formula | Build status |
|---|---|---|
| **Deposit-adjusted collection** | Sum of cleared invoice amount tied to payment mode `211` | Has a case in `CollectionFilterCode` (`DEPOSIT_ADJUSTED`, `helpers.ts:288-315`) — but note the broken SQL at `:292` (Rule 3). |
| **Advance-adjusted collection** | Sum of cleared invoice amount tied to payment mode `288` | Has a case (`ADVANCE_ADJUSTED`, `helpers.ts:288-315`). |
| **Caution-money-adjusted collection** | Sum of cleared invoice amount tied to payment mode `291` | **Must be built — no standalone path exists today.** |

> [!WARNING] Caution-money-adjusted (291) has no standalone path
> The `CollectionFilterCode` enum (`helpers.ts:288-315`) has `DEPOSIT_ADJUSTED` (`211`) and `ADVANCE_ADJUSTED` (`288`) cases but **no `291` case**. The widget only ever lumps `211`, `288`, `291` together (`helpers.ts:1010-1017`) — it never breaks `291` out on its own. So the caution-money line is the same status as the other to-build buckets, **not** at parity with `211` / `288`.

The current product does not treat all three modes consistently across list, widget, and homepage. DA-02 must.

### 4.5 Trend

Trend repeats the same **net collection** definition month by month.

**For DA-02:** a month bar in trend must equal the value shown if a user opens that same month directly.

Do not use one monthly rule for trend and another for the screen body.

### 4.6 Collection by property

Property view uses the same selected-period total, split by property.

**For DA-02:** the property rows must add back to the selected total exactly.

### 4.7 Payment settlement

Settlement is not a separate finance product in DA-02. It is another way of reading the same received payments.

Online modes for all three states = `(203, 205, 206, 210)` (§3.2).

| State | Plain meaning | Exact condition |
|---|---|---|
| **Settled** | Online collection with final settlement reference present | `payout.status = 1` (`formatRow:515`) |
| **In processing** | Online collection with payout started but not finished | `payout_id IS NOT NULL AND payout.status != 1` (`helpers.ts:323-327`) |
| **Unsettled** | Online collection received, but no payout or wallet entry yet | `payout_id IS NULL AND wallet_entry IS NULL` (`helpers.ts:317-321`, `:849-850`) |

`payout.status` codes (`formatRow:515-521`): `0` = reversed/failed, `1` = settled, `2` = processing.

## 5. Time behavior

| Measure family | Date used |
|---|---|
| Total collection, settlement, mode, received-by, property, trend | Received date |
| Current-period / past-due / future-due / advance split | Received date for membership in the selected period, then due date or due type for the split |
| Unpaid dues in collection status | Due date and unpaid invoice state |
| Adjusted collection | Received date plus adjustment mode code |

If a number cannot be explained with one row in this table, it should not ship.

## 6. Where the current product still disagrees with itself

1. **List total vs widget total vs homepage total are not one answer.**
2. **Refund subtraction exists in the widget path but not in the list grand-total path** (`executePaginated :419-429` sums `raw_total_amount`, no refund join).
3. **Adjustment-mode handling splits on `291`:** the list grand-total path excludes only `211`, `288` and never references `291`; the widget and homepage exclude all three. DA-02 standardizes on excluding `{211, 288, 291}`.
4. **One old adjustment path is broken SQL that throws** (`helpers.ts:292`, unbalanced paren / unterminated string).
5. **Some deeper pages have weaker access checks than the main collection list** (safe entry point: `checkAuth('viewInvoices')`, `service.ts:21-34`).

These are not small polish issues. They change what the number means.

## Changelog

- 2026-06-06 - v2.0 - Hardened against live code, owner decisions folded in. Real `291` exclusion split stated precisely; broken SQL at `helpers.ts:292` flagged as a throw; money primitive, online/settlement contract, invoice/payout/tenant enums, and due-type exact strings folded in; headline collection rebased on payment side (partial receipts count); `291` adjusted-collection marked must-be-built.
- 2026-06-06 - Created the missing DA-02 formula map from live backend behavior and current screen direction.
