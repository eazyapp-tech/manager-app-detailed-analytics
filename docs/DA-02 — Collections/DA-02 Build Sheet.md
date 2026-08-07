---
title: DA-02 Collections - Build Sheet
date: 2026-06-06
tags:
  - rentok
  - collections
  - build-sheet
status: v2.0 - ready for build planning
owner: Sanchay
companion_to: DA-02 Ground-Truth Formula Map, DA-02 Brief
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
> The engineering spec for **DA-02 Collections**. This doc carries facts from the formula map into build tasks. It does not re-argue the product bet.
>
> **Build rule:** finish the shared aggregation and drill safety work first. Every section after that depends on them.
>
> *"Operator" = the person using the screen — an owner, property manager, or finance-minded user.*

## A. Data foundation first

Two things have to be true before the screen is trustworthy:

1. **All DA-02 totals come from one shared collection definition.**
2. **Every drill stays inside pages that check the user's access.**

Without both, the screen may look finished and still be unsafe or misleading.

## B. Shared rules every task inherits

1. **Net collection means refund-aware received money.** Refund subtraction is mandatory.
2. **Adjustment modes are separate.** Modes `211`, `288`, and `291` are paper adjustments, not fresh cash.
3. **Paid date and due date do different jobs.** Paid date answers "when did money come in." Due date answers "what bill was it for."
4. **One screen, one answer.** Homepage collection, DA-02 overview, DA-02 cards, and DA-02 drills must agree when they claim to show the same thing.

## C. Tasks

| Task | Why | Files / queries | Acceptance check |
|---|---|---|---|
| **1. Build one shared collection aggregate for DA-02 and homepage reuse.** | The current list, widget, and homepage collection numbers are not based on one shared answer. | Start from `src/v1/list_screens/collections/helpers.ts:407-449`, `:713-724`, `:930-1045`, and `src/v1/homepage/service.ts:2447-2453`. **Headline base is payment-side** (`p.status = 1 AND p.is_active = 1`, paid date), NOT the legacy `status = 1` invoice base — that base undercounts partial receipts on open invoices and must not be reused. Collected amount per Formula Map §3.1. | The same property + period returns the same total in DA-02 overview, DA-02 cards, and homepage collection wherever the label claims the same meaning. A partial receipt on a still-open invoice (`status = 0`) is counted. |
| **2. Make the shared aggregate refund-aware.** | Net collection is wrong if refunds are left out in one path. | Refund model at `src/entities/refunds.ts:5-43`; current refund-aware widget logic at `src/v1/list_screens/collections/helpers.ts:713-724` and `:930-1045`. | A refunded invoice lowers net collection everywhere. A period with more refunds than inflow can go negative without breaking the response. |
| **3. Normalize adjustment-mode handling.** | DA-02 must not mix paper adjustments with fresh cash. | Mode labels at `src/helpers/payments.ts:781-822`; current exclusions at `src/v1/list_screens/collections/helpers.ts:615-624`, `:824-853`; homepage exclusions at `src/v1/homepage/service.ts:2449-2452`. | Modes `211`, `288`, and `291` are excluded from cash inflow totals and shown only in the adjusted-collection path. |
| **4. Replace the broken old adjustment filter logic before reuse.** | The existing filter case throws at query time, not just mis-counts. `helpers.ts:292` has malformed SQL: `query.andWhere('p.payment_mode IN (211');` — unbalanced paren, unterminated string. | Broken case at `src/v1/list_screens/collections/helpers.ts:288-293` (the throw is at `:292`). Note: `291` (Caution Money) has no standalone case in `CollectionFilterCode` (`:288-315`) — it must be built, not copied. | DA-02 adjusted-collection numbers come from cleaned shared logic, not the broken old condition. The `291` adjusted line exists as its own bucket. |
| **5. Build DA-02 overview widgets from the shared aggregate.** | The top of the screen must answer the period read in seconds. | Current widget titles in `src/v1/list_screens/collections/helpers.ts:1069-1105`; Figma direction uses total, current-period dues, past dues, future dues, current FY, and settlement pending. | Overview widgets render from the same base dataset and each drill into the existing collection list with matching numbers. |
| **6. Build source buckets using due-date classification on received payments.** | Operators need to know whether money came from current dues, old dues, future dues, or advance. | Paid-date scope at `src/v1/list_screens/collections/helpers.ts:335-350`; due-type fields at `src/entities/invoices.ts:98-114`. | For one selected period, every received-money row lands in exactly one source bucket or the separate adjustment bucket. |
| **7. Build collection-status buckets from billed-period math.** | Received money alone cannot answer whether the billed period is on track. | Invoice fields at `src/entities/invoices.ts:84-103`; dues patterns in `src/v1/homepage/service.ts:2428-2437`. | The status view clearly separates collected-current-period dues, collected-past dues, collected-future dues, advance, and unpaid amount for the selected period. |
| **8. Build breakup cuts: category, mode, status, received-by.** | The operator needs multiple cuts of the same received-money base, not four separate definitions. | Grouping inputs live in `src/entities/invoices.ts`, `src/entities/payments.ts`, and the tenant join in `src/v1/list_screens/collections/helpers.ts:151-167`. | Switching the grouping changes the buckets, not the base total. All tabs add back to the same received-money total. |
| **9. Define how unlinked payments behave in breakup tabs.** | Some payments can exist without a linked tenant row, which breaks status-based grouping if left vague. | Tenant join and null case at `src/v1/list_screens/collections/helpers.ts:151-167` and `:380-387`. | Unlinked payments stay in collection totals and mode/received-by cuts, and have an explicit behavior in status-based cuts instead of vanishing silently. |
| **10. Build payment-settlement summary from existing payout, wallet, and settlement scheduler state.** | Operators read "received" and "settled" together. | Settlement text logic at `src/v1/list_screens/collections/helpers.ts:511-574`; entities at `src/entities/settlement_scheduler.ts:56-105`, `src/entities/payout.ts:52-94`, `src/entities/wallet.ts:29-64`. | Every online collection row can be classified into settled, unsettled, or in-process without a second manual lookup. |
| **11. Build collection-by-property only on top of the same aggregate.** | Multi-property comparison is useful only if it does not introduce yet another definition. | Property scoping already supports multi-property strings at `src/v1/list_screens/collections/service.ts:36-40` and `:160-163`. | Property totals add back to the selected account total exactly. |
| **12. Build trend from the same net-collection definition, repeated by month.** | Trend is only useful if month-to-month numbers use the same rule set. | Use the same shared aggregate with month slices; do not reintroduce the old widget-only logic. | A month opened in the trend matches the month opened directly in the screen filter. |
| **13. Secure second-level drill endpoints or stop DA-02 at first-level drills.** | The current first-level list check is stronger than some detail endpoints behind it. | First-level auth at `src/v1/list_screens/collections/service.ts:21-34`. Review `src/controllers/invoices.ts:295-380`, `:3427-3468`, and `src/controllers/tenant.ts:3337-3368`. | No DA-02 drill lands on a detail page that does not check the user's access in production. |
| **14. Keep report export tied to the list service, not a parallel DA-02 export stack.** | Reuse reduces drift and keeps reporting aligned with the list. | `src/v1/list_screens/reports/services.ts:32-39` and `:96-124`. | Collection export still pulls from the list service, and DA-02 drills/export agree on row totals. |
| **15. Cross-check deposit and advance side-effects before launch.** | Collection numbers affect notice reports, tenant ledgers, and available deposit/advance math. | Deposit and advance helpers at `src/helpers/payments.ts:593-777`; notice report at `src/services/reports/generateNoticeReport.ts:390-427`; collection report at `src/services/reports/generateCollectionReport.ts:367-436`. | DA-02 adjustment and refund logic does not contradict deposit/advance balances shown elsewhere in the product. |

## D. Open decisions for kickoff

These are real owner calls, not lazy placeholders.

1. **If Q5 must be trimmed, which tab survives first?**
Recommendation: staffed owners keep `Received by`; solo operators keep `Mode`.

2. **Do we allow second-level drills on day one if auth fixes slip?**
Recommendation: no. Stop at the authenticated collection list until detail endpoints are safe.

## E. Launch checks

- Homepage financials collection total and DA-02 period total match whenever the label means the same thing.
- Refunds reduce net collection in every total shown on the screen.
- Adjustment modes never inflate fresh-cash totals.
- The same payment cannot appear as both cash inflow and adjustment.
- Trend, property, and breakup tabs all add back to the same received-money base.
- No drill reaches a detail endpoint that does not check the user's access.
- **Do NOT inherit the homepage `other` bucket bug.** At `src/v1/homepage/service.ts:2452`, the `deposits` bucket includes `('Security Deposit', 'Caution Money')` but the `other` bucket only excludes `('Rent', 'Security Deposit')` — so Caution Money double-counts into both `deposits` and `other`. DA-02's shared aggregate must exclude Caution Money from `other` so each rupee lands in exactly one category. (Flagged for eng — fix in the shared aggregate, do not copy the homepage grouping as-is.)

## Changelog

- 2026-06-06 - v2.0 - Hardened against live code, owner decisions folded in. Headline base rebased to payment-side (partial receipts count); broken SQL at `helpers.ts:292` stated as a throw; `291` flagged must-be-built; homepage `other` bucket Caution-Money double-count flagged in launch checks; "operator" glossed; "reconcile" → "add back to", "surface" → "page".
- 2026-06-06 - Rewritten around one shared aggregate, adjustment-mode cleanup, and drill safety as the three main build themes.
