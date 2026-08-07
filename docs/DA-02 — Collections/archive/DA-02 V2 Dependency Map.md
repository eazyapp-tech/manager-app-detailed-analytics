---
title: DA-02 V2 Scope — Settlement Lifecycle + Bank Routing Dependency Map
date: 2026-05-16
tags: [rentok, brief, collections, da-02, v2-scope, settlement, bank-routing]
companion_to: "[[DA-02 Brief]] (V0.7 → ships as V1; this doc scopes V2)"
status: V2 scoping note · ready for V2 brief upgrade
---

## TL;DR

V0.7 treats Collections as "what came in." Codebase reveals a parallel **settlement lifecycle** (Payment → `SettlementFlowService.scheduleSettlement()` → `settlement_scheduler` → `WalletPayoutService.createWalletPayouts()` → `wallet_payouts` → Razorpay webhook) that decides **when money hits the owner's bank account**. The Collections screen today shows ₹X received from the gateway; it doesn't show whether ₹X has settled to the bank yet — that gap is invisible until the operator manually reconciles their bank statement.

Plus **bank routing has 3 precedence levels** (DueTypeBankMapping → Room → Property) that determine which of an owner's bank accounts receives any given payment. Multi-bank routing is real and not rare, but **no bank-wise summary endpoint exists** today.

**V1 ships with the gap acknowledged. V2 adds the settlement + bank dimensions.**

---

## 1. Domain dependency map (codebase-grounded)

### Settlement lifecycle (V2 architecture, runs alongside legacy Payout)

```
   ┌──────────────────────────────────────────────────────────┐
   │ Tenant pays (controllers/payment.ts)                     │
   │   → payments row written                                 │
   └──────────────────┬───────────────────────────────────────┘
                      │
                      ▼
   ┌──────────────────────────────────────────────────────────┐
   │ SettlementFlowService.scheduleSettlement()               │
   │ src/services/settlementFlow/settlementFlow.ts:474        │
   │   → resolves getFundAccountId() → AccountDetails         │
   │   → creates settlement_scheduler row                     │
   └──────────────────┬───────────────────────────────────────┘
                      │
                      ▼
   ┌──────────────────────────────────────────────────────────┐
   │ settlement_scheduler (entity)                            │
   │ src/entities/settlement_scheduler.ts                     │
   │   status: 0 initiated · 1 success · 2 attempted · 3 fail │
   │   gross_amount, charges, rentok_charges, net_amount      │
   │   scheduled_at, success_at, failed_at, utr               │
   │   mode: 0 instant / 1 others                             │
   │   FK → payment_id, account_details_id, property_id       │
   └──────────────────┬───────────────────────────────────────┘
                      │
                      ▼
   ┌──────────────────────────────────────────────────────────┐
   │ WalletPayoutService.createWalletPayouts()                │
   │ src/services/settlementFlow/walletPayout.ts:20           │
   │   → groups N scheduler rows per account_id               │
   │   → respects Razorpay payout limit                       │
   │   → creates wallet_payouts row                           │
   └──────────────────┬───────────────────────────────────────┘
                      │
                      ▼
   ┌──────────────────────────────────────────────────────────┐
   │ wallet_payouts (entity)                                  │
   │ src/entities/walletPayout.ts:29                          │
   │   status: -1 RZP failed · 0 pending · 1 success · 2 gen  │
   │   payout_id, utr, gateway_charges, payout_mode           │
   │   FK → account_id                                        │
   └──────────────────┬───────────────────────────────────────┘
                      │
                      ▼
   ┌──────────────────────────────────────────────────────────┐
   │ Razorpay webhook                                         │
   │ src/controllers/payouts.ts                               │
   │   payout.processed / reversed / failed / rejected        │
   │   → updates wallet_payout                                │
   │   → bubbles UTR up to settlement_scheduler               │
   └──────────────────────────────────────────────────────────┘
```

**Settlement modes** (from `propertySettlementSetting.settlement_mode`, `src/entities/propertySettlementSettings.ts`):
- `1` = instant
- `2` = on-demand
- `3` = T+n (with `n_value`)
- `4` = 10-10 twice/day (with `settlement_time`)

Branching at `settlementFlow.ts:560,608,612,648`.

### Legacy `Payout` path (still live for old properties)

`Payout` (status: `0 failed / 1 success / 2 pending`) is M:N with `Settlements` (status: `0 scheduled / 1 success`). FK to property + `accont_fk_id` → `AccountDetails`. Atomic transaction with pessimistic lock at `src/controllers/payouts.ts`.

**Critical:** new flow + legacy flow coexist. Gate function: `SettlementFlowService.isSettlementV2FlowEnabled(property_id)`. Properties on either side don't talk to each other.

### Bank-routing graph (3 levels of precedence)

```
  Highest precedence (most specific):
  ┌─────────────────────────────────────────────────────────┐
  │ DueTypeBankMapping (entity)                             │
  │ src/entities/dueTypeBankMapping.ts                      │
  │   per-property × per-DueType × per-Account              │
  │   priority, amount_limit, amount_routed, reset_period   │
  │   routing_rules (jsonb): percentage_split, min/max,     │
  │                          allowed_rooms                  │
  │   settlement_charges                                    │
  └─────────────────────────────────────────────────────────┘
                            │ (if no match)
                            ▼
  Middle precedence:
  ┌─────────────────────────────────────────────────────────┐
  │ Room.account_details_id                                 │
  │ src/entities/room.ts:131                                │
  │   room-level override                                   │
  └─────────────────────────────────────────────────────────┘
                            │ (if no match)
                            ▼
  Lowest precedence (default):
  ┌─────────────────────────────────────────────────────────┐
  │ Property.account_details_id                             │
  │ src/entities/property.ts:431                            │
  │   property default                                      │
  └─────────────────────────────────────────────────────────┘

  All point to: AccountDetails entity
  src/entities/accountDetails.ts
   - owner_id, bank_account_no, ifsc, vpa, nick_name
   - customer_name, pan, verified
   - type: 1 bank / 2 upi / 3 IFT
   - Razorpay contact_id_v2, fund_id_v2
```

**Multi-bank is real and not rare.** Any property using mid-month switching, per-due-type splits, or rent-vs-deposit segregation has 2+ accounts in play.

### Credits (mode 213) — bypass the settlement flow

`Credits` entity (`src/entities/credits.ts`) — `source: 0 owner / 1 eazypg`, `status: 0 unused / 1 used`, with view_status, expiry_date, etc. Payment mode 213 explicitly bypasses settlement:

```ts
if (payment_mode == 213) return;  // settlementFlow.ts createWalletEntries
```

Credits never produce a payout. They're a P&L adjustment, not cash-in. V2 must classify mode-213 collections as **"Not applicable — paid via credit"** else they'll permanently sit in "pending settlement" and tank operator trust.

### TeamPassbookCollectionService

`src/services/teamPassbook/teamPassbookOperationFactories/teamPassbookCollectionService.ts`. Mirrors every Collection into `team_member_transactions` with `category=COLLECTION`, `balance_type: 1 inflow / 2 outflow`, `is_paid`, `payment_map.payment_uuid`.

Edit triggers a `COLLECTION_REVERSE` + new `COLLECTION` row tied by `edit_group_id` — full audit trail.

**Implication for V2:** "who collected" attribution (sub_user_id) and any cash-vs-online reconciliation drill should source from `team_member_transactions` joined back to `payments`, not from `payments` alone. The reverse rows mean **naively summing balance_type=1 double-counts edits** — must filter most-recent per `edit_group_id` or net `(1 inflow) − (reverse)`.

---

## 2. Existing operator-facing surfaces I missed

| Surface | Where | What it shows | What's missing for V2 |
|---|---|---|---|
| `POST /payment/getPendingSettlements` | `controllers/payment.ts:6371` + `routes/payments.ts:355` | Returns `pending_settlements[]` with `approval_status`, `fund_id_v2`, `personal_contact` | **Not consumed by app today** — only used internally by `SettlementFailureReminderService` to send WA reminders. Operator never sees a UI for "what's stuck" |
| `POST /payout/*` webhooks → push notif | `controllers/payouts.ts` | "₹X settled in your bank. UTR: …" Razorpay → updates payout, fires notification | Fire-and-forget push; no in-app dashboard for operator |
| `src/controllers/settlements.ts` | `src/controllers/settlements.ts` | **Entirely commented out** — `getSettlementsForProperty`, `getSettlementsForAccount` exist as dead code | No operator-facing settlement list endpoint exists today |
| `SettlementFailureReminderService` | `src/services/payment/settlementFailureReminder.ts` | WA messages to owner when verification pending or no bank linked | No in-app surface; owner gets a message, can't self-serve |
| Bank-wise summary endpoint | **Does not exist** | — | No route aggregates collections / payouts grouped by `account_details_id`. Gap |
| Settlement-state filter on Collections list | `src/v1/list_screens/collections/` | Filter codes 1201-1219, **none for settlement status** | Gap. List endpoint can't slice by "settled" vs "pending" |

**Net:** operators have zero in-app visibility into settlement state today. They (a) wait for a push notification per payout, or (b) reconcile their bank statement manually. This is exactly what V2 should fix.

---

## 3. V2 question list — ranked by operator pain × codebase feasibility

### Must-add to V2 (P0 — operator pain confirmed by absent surfaces above)

**Q10 — "How much of this period's collections is settled to my bank vs still pending?"**
- WHY: today operator sees ₹X collected in Collections screen but their bank shows ₹Y; gap is invisible until manual reconciliation. Mode 213 (credits) further confuses — no payout ever comes for those.
- WHAT: 3 buckets on the hero card → `Settled (status=1) · In transit (status=0 or 2) · Failed (status=3 or wallet_payout.status=-1)`. Plus credits-only collected, labeled "No payout — paid via credits."
- WHERE: aggregate `settlement_scheduler` joined `payment_id → payments` filtered by `paid_date` ∈ period. Status from `settlement_scheduler.status`, escalate via `wallet_payout.status` for `attempted (2)` rows. Mode 213 carved out by joining `payments.payment_mode != 213`.

**Q11 — "Per-bank breakdown: which bank account received how much this period?"**
- WHY: multi-property / multi-account owners need to know which account to check. With `DueTypeBankMapping` priority+limit routing, even a single property can split across 3 accounts mid-period.
- WHAT: table → bank `nick_name | last4 (bank_account_no)` | gross collected this period | settled | pending | failed. Two source paths:
  - **For settled money:** `settlement_scheduler.account_details_id` (authoritative — actual routing destination)
  - **For not-yet-routed money:** walk `payments → invoice → (DueTypeBankMapping || room || property).account_details_id` to predict destination. Flag this column "Predicted destination" until scheduler row appears.
- WHERE: `AccountDetails` join on `account_details_id`. Group by `account_id`. Show `nick_name`; fallback to "Bank acct ending **XXXX**" + `bank_name`.

### Should-add to V2 (P1 — clear value, smaller pain)

**Q12 — "Failed settlements I need to chase this period."**
- WHY: today these become an inbound WA reminder ("verification pending"). Operator has no list view; can't bulk-act.
- WHAT: count chip "N failed · ₹X stuck" → drill to list of `settlement_scheduler.status IN (3) OR wallet_payout.status = -1`, with reason (`remarks`, `failure`), suggested action (verify account / re-link bank).
- WHERE: union of `settlement_scheduler` failed rows + `wallet_payouts.status = -1` joined to `account_details_id`. Reason text from `wallet_payouts.remark_json` and `payout.failure`.

### Defer to V3 (P2 — interesting but cost > benefit)

**Q13 — "What's my average settlement delay?"**
- Low operational pain — most owners on T+n know their cadence; outliers covered by P0 + P1. Computable from `success_at - payment.paid_at` but doesn't drive action.

**Q14 — "Per-gateway breakdown (Razorpay / Cashfree / RentOk-Pay / UPI)."**
- Collections screen primarily organizes by *payment source* (online / cash / bank-transfer), which already approximates this. Operator pain is bank-level not gateway-level. **BUT verify with operators** — if field interviews show owners ask "did Razorpay come in yet?", reclassify P0.

---

## 4. V1 → V2 architecture risks (decide before V2 lands)

1. **Settlement-state ambiguity** — `settlement_scheduler` (new path), `wallet_payouts` (intermediate), `settlements + payout` (legacy) coexist. **Decision before V2:** is DA-02 V2 strictly V2-flow-only, or does it also surface legacy `Payout.status` for older properties? `SettlementFlowService.isSettlementV2FlowEnabled(property_id)` is the gate. Without a unified read model, the dashboard will say "₹0 settled" for properties still on legacy.

2. **Mode 213 (credits) classification** — the new "Settled vs Pending" bucket must explicitly classify mode-213 as "Not applicable — paid via credit." Else it sits in "pending" forever and tanks trust. Build Sheet `formatRow:481-489` already proves this pattern works.

3. **`DueTypeBankMapping` predicted-destination accuracy** — `amount_routed` running total + `priority` cap means a payment's destination depends on order-of-arrival within the reset period. Predicting destination for unsettled payments requires replaying mapping rules. **Recommend:** show "destination pending" rather than predict.

4. **TeamPassbook double-count** — bank-wise aggregation sources from `settlement_scheduler` (authoritative bank). Operator "who collected cash" label sources from `team_member_transactions` (cash-collection ledger, no bank context). Two different sources for two different columns.

5. **No bank-wise endpoint exists today.** New endpoint required: `POST /v1/list_screens/collections/by_bank` returning the per-bank rollup. Auth must mirror the existing collections endpoint (`viewInvoices` + `view_invoices_of_self_added_tenants` fallback).

6. **Legacy `Payout.account_details: string`** (not FK) — old payouts store the bank account *number string* not UUID (`src/entities/payout.ts:30`). Joining legacy payouts to AccountDetails requires `account_details_no` matching, prone to nulls. Bank-wise breakdown will be incomplete for legacy-flow properties unless backfilled.

---

## 5. Recommended V2 scope for DA-02 Brief

Add a new section **"V2 additions — Settlement & Bank"** with 3 questions:

| # | Operator question | Eng effort | Source |
|---|---|---|---|
| Q10 | "Of ₹X collected this period, how much has hit my bank?" | M — new aggregation over `settlement_scheduler` + escalate via `wallet_payouts`, mode-213 carve-out. New endpoint or extend existing collections endpoint with `?include=settlement_summary` | `settlement_scheduler.status`, `wallet_payouts.status`, `payments.payment_mode` |
| Q11 | "Per-bank account: how much routed where this period?" | M-L — group-by on `settlement_scheduler.account_details_id` + join AccountDetails. Legacy-flow properties show partial data with a banner | `settlement_scheduler` × `account_details` × `payments` |
| Q12 | "Failed settlements stuck — who's blocking and why?" | S — union of `settlement_scheduler.status=3` + `wallet_payouts.status=-1` with reason text. Bonus: deep-link to "Re-verify bank" CTA in account settings | `settlement_scheduler`, `wallet_payouts.remark_json`, `payout.failure` |

**Cut from V2 (defer to V3):** average settlement delay, per-gateway breakdown.

**Pre-build decision for Jatin (must close before V2 starts):**
Legacy-path properties (`isSettlementV2FlowEnabled = false`) — show legacy `Payout` data with caveat banner, OR scope DA-02 V2 to V2-flow properties only with a "your property is on legacy settlement, contact support" placeholder? **Recommend the latter** — cleaner data model, surfaces the migration debt, avoids dual-read complexity.

---

## 6. Cross-doc traceability

- V1 Brief: `[[DA-02 Brief]]` (V0.7 — ships with the gap acknowledged in a "V2 scope deferred" section)
- Build Sheet: `[[DA-02 Build Sheet]]` (V1 — locked at engineering scope; does NOT include settlement layer)
- Sibling V2 maps: `[[DA-03 V2 Dependency Map]]` · `[[DA-04 V2 Dependency Map]]`

---

## Changelog

| Date | Change | By |
|------|--------|-----|
| 2026-05-16 | Initial V2 scope note — generated from deep codebase exploration sub-agent (settlement lifecycle, bank-routing precedence, mode-213 bypass, TeamPassbookCollectionService, dead-code surfaces). Findings to inform a future DA-02 V2 brief upgrade. | Sanchay (PM) + Claude sub-agent |
