---
title: DA-05 Discounts — Codebase Exploration (Phase 2 LAYER 1 + LAYER 2)
date: 2026-05-16
tags: [rentok, brief, discounts, da-05, phase-2-grounding]
companion_to: "[[DA-05 Brief]] (pending — this doc feeds Phase 3 gather)"
status: Phase 2 grounding · ready for Phase 3 gather inputs
---

## 1. TL;DR — what V1 of the Brief should know

**"Discounts" in the codebase = `credits` table.** No `discounts` table exists. The same table also stores **owner-issued discounts**, **RentOk-funded credits (scratch cards)**, and **management/concession adjustments** — but does NOT store loyalty points, referral, or wallet (those live elsewhere: `kyc_credit_balance` on tenant, `wallet_entry`, FlexiPe).

**Two orthogonal axes operators conflate:**
1. `source` → who pays for it (0 = owner P&L hit, 1 = RentOk P&L hit; only matters for EazyPG-collected payments via mode 213)
2. `view_status` → UX gate (102 = unscratched/hidden, 101 = scratched/visible-to-tenant)

**Three distinct credit types live in this table, all surface as "discount" to operators:**
- **Owner-issued discounts** (`source=0`, `view_status=101` always on create, scratching skipped) — created via `addCredits` / `addCreditsFromTenantPhone` / `addBulkCreditsFromPhone`. Default 30-day expiry, free-text description, `initiated_by` = team member name.
- **RentOk-funded "scratch card" credits** (`source=1`, `view_status=102→101` flow) — created via Firebase sync, tenant must scratch to reveal.
- **Adjustment/Negotiation credits** — same shape as owner discounts, distinguished only by free-text `description`.

**Categorization is description-based ILIKE classification** — no enum, no category column. Operators have no UI to pick a category at creation time; the analytics screen re-classifies post-hoc by string match on `description`. This is fragile (PB1 candidate).

**Unit: `Credits.amount` is INT, treated as rupees everywhere** (no `type: 'decimal'`, no `* 100`). Distinct from Payments which use paise. HB5 confirmed correct.

**Permission model is broken for delete** (HB2): `deleteCredits` checks only `pg_id` and `source !== 1`. Anyone with property access + valid token can delete owner credits. No `delete_discount`, no `record_payment` gate, no `viewDiscounts`/`editDiscounts` keys exist.

**Activity log half-wired** (HB3): `ACTIVITY_LOG_DISCOUNT = 4` fires on add only (`credits.ts:516`, `credits.ts:609`). Not on delete, not on scratch, not on link-to-payment.

**Refund→credit unwind broken** (HB1 CONFIRMED): refund controllers / `services/refunds/` have ZERO references to Credits. A refund of a payment that used a credit silently leaves `credit.status=1, date_used=<past>`. Credit cannot be re-applied. No alert, no manual fix UI.

---

## 2. LAYER 1: Direct entity grounding

### 2.1 Credits entity — full schema

File: [src/entities/credits.ts](../../../../rentok-backend/src/entities/credits.ts)

```ts
@Entity('credits')
export class Credits extends BaseEntity {
  id:                     string (uuid, PK)
  property:               string  (required, format: `${pg_id}PG${pg_number}`)
  tenant:                 string  (nullable, firebase_id of tenant)
  firebase_id:            string  (nullable, mirrors id post-save for legacy sync)
  initiated_by:           string  (nullable, FREE-TEXT — team member NAME or "Owner" or "Admin")
  date_added:             timestamp (nullable, no default — set in code)
  is_minimum_full_amount: number  (default 0 — 0=pay full to use, 1=pay min to use)
  minimum_amount:         decimal (nullable, threshold for is_minimum_full_amount=1)
  expiry_date:            timestamp (nullable — null = never expires)
  date_used:              timestamp (nullable — set when status flips 0→1)
  description:            string  (nullable, free-text)
  source:                 number  (nullable — 0=owner, 1=eazypg/RentOk)
  status:                 number  (nullable — 0=unused, 1=used)
  view_status:            number  (nullable — 101=scratched/visible, 102=unscratched/hidden)
  amount:                 number  (required, INT, RUPEES not paise)
  payment:                ManyToOne→Payments (joined on payment_id)
  payment_id:             string  (nullable, FK to payments — set when credit applied)
}
```

**Critical caveats:**
- `initiated_by` is **not a UUID** — it's the team member's display name resolved at write time via `TeamMemberService.getTeamUuid` then `team_data?.name` ([credits.ts:485](../../../../rentok-backend/src/controllers/credits.ts#L485)). Means renaming a team member breaks historical attribution. Means filter-by-creator must use string equality, not UUID.
- No `created_at` / `updated_at` from BaseEntity surfaced (BaseEntity has `created_at` per project pattern). `date_added` is the operator-facing creation timestamp.
- No `is_active` / soft-delete. `deleteCredits` does `credit.remove()` — hard delete. Once deleted, no audit trail (compounds HB3).
- No `category` column. No `discount_type` enum. ILIKE-on-description is the only categorization signal.

### 2.2 Existing endpoints over Credits

File: [src/routes/credits.ts](../../../../rentok-backend/src/routes/credits.ts)

| Endpoint | Controller | Surface | Auth |
|---|---|---|---|
| `POST /addCreditFromFirebase` | addCreditFromFirebase | Firebase mobile sync | None |
| `POST /updateCreditsFromFirebase` | updateCreditsFromFirebase | Firebase mobile sync | None |
| `POST /deleteCreditsFromFirebase` | deleteCreditsFromFirebase | Firebase mobile sync | None |
| `POST /updateCreditsPaymentDetails` | updateCreditsPaymentDetails | Internal | None |
| `POST /linkCreditToPayment` | linkCreditToPayment | Payment flow | None (!!) |
| `POST /validateCredits` | validateCredits | Pre-apply check | HeaderValidator |
| `POST /scratchCredit` | scratchCredit | Tenant app | HeaderValidator |
| `POST /fetchTenantCredits` | fetchTenantCredits | Tenant credits list (all) | None |
| `POST /fetchUnscratchedTenantCredits` | fetchUnscratchedTenantCredits | Tenant scratch-card surface | None |
| `POST /fetchCreditsToApply` | fetchCreditsToApply | Operator collect-payment flow | None |
| `POST /deleteCredits` | deleteCredits | Operator app | HeaderValidator only |
| `POST /addCredits` | addCredits | Operator: add discount to tenant | HeaderValidator only |
| `POST /addCreditsFromTenantPhone` | addCreditsFromTenantPhone | Operator alt | HeaderValidator |
| `POST /addBulkCreditsFromPhone` | addBulkCreditsFromPhone | Bulk campaign | HeaderValidator |

**No DA-05 list/widget/filter_options/detail/Excel endpoints exist** — all NEW BUILD confirmed.

### 2.3 Key flows verified

**`addCredits` ([credits.ts:413-528](../../../../rentok-backend/src/controllers/credits.ts#L413)):**
- Hardcodes `source = 0`, `status = 0`, `view_status = 101` (no scratch needed for operator-added)
- Default expiry: 30 days from now (`moment().add(30, 'days')`) when not provided
- `initiated_by` = team member display name (or "Owner" / "Admin")
- Fires `ACTIVITY_LOG_DISCOUNT` post-save ([credits.ts:516](../../../../rentok-backend/src/controllers/credits.ts#L516))
- Tied to single tenant — no multi-tenant API

**`fetchCreditsToApply` ([credits.ts:285-340](../../../../rentok-backend/src/controllers/credits.ts#L285)):**
- Filter: `tenant + property + status=0 + expiry_date >= now + view_status=101`
- Splits return into `owner_credits` + `eazypg_credits` by `source` — confirms the operator-facing split is real, not invented by the Build Sheet.

**`linkCreditToPayment` ([credits.ts:175-189](../../../../rentok-backend/src/controllers/credits.ts#L175)):**
- **Mass-updates ALL of a tenant's credits at given property** to set `payment_id` — does NOT filter by which credits were actually selected. `Credits.update({tenant, property}, {payment})`.
- Does NOT flip `status=1` or set `date_used` here. Status flip happens elsewhere (paymentService) — split state.
- No `ACTIVITY_LOG_DISCOUNT` fires here.

**`scratchCredit` ([credits.ts:666-705](../../../../rentok-backend/src/controllers/credits.ts#L666)):**
- Updates `view_status = 101` (was 102). Returns success.
- Validates: not already used, not expired.
- No activity log fires.

**`deleteCredits` ([credits.ts:246-283](../../../../rentok-backend/src/controllers/credits.ts#L246)):**
- Guards: must find by `pg_id+pg_number+credit_id`; reject if `status=1`; reject if `source=1` (can't delete RentOk-funded).
- **NO permission key check** — HB2 confirmed. Any team member with HeaderValidator + property scope can delete unused owner credits.
- Hard delete via `credit.remove()`. No activity log fires — HB3 confirmed.

### 2.4 HB1 verification — refund DOES NOT flip credit

Searched `src/services/refunds/` and `src/controllers/refunds.ts` for any reference to `credit` / `Credits`. **Zero hits.** Confirmed:
- When a refund processes against a payment that consumed credits, the credit rows retain `status=1, date_used=<original>, payment_id=<original>`.
- Credit cannot be auto-re-issued — operator must manually re-create via `addCredits`.
- No notification, no flag, no entry in any analytics screen highlighting this drift.
- Net effect: collections analytics still show the credit as "used", refund analytics show the cash-out, the gap is invisible.

---

## 3. LAYER 2: Domain dependency map

### 3.1 Sibling uses of the Credits table

The same table powers **three distinct operator mental models** that all surface as "discount" only sometimes:

**A. Owner-issued discounts** (the DA-05 target)
- Created via operator action — record-payment flow, tenant detail screen.
- `source=0, view_status=101` always, no scratching.
- Hits P&L of the property owner / RentOk customer.

**B. RentOk-funded scratch cards** (`source=1`)
- Created via Firebase sync from marketing/growth flows (`addCreditFromFirebase`).
- `view_status=102` initially (hidden) → tenant must "scratch" in tenant app to reveal (101).
- Operator sees them in `fetchTenantCredits` but cannot delete.
- These are RentOk-paid promotional credits — typically tied to mode 213 collection settlements where RentOk recovers via settlement adjustments.
- **Operators often don't realize these are a separate species** — they look identical in tenant credit history.

**C. Management/concession adjustments** (`source=0` but semantically different)
- `addCreditsFromTenantPhone` hardcodes `description = 'Management discount'`, `initiated_by = "Owner"`, 30-day expiry. ([credits.ts:530-665](../../../../rentok-backend/src/controllers/credits.ts#L530))
- `addBulkCreditsFromPhone` loops the above over phone-array → this **IS the bulk campaign primitive** the Build Sheet asks about (no separate endpoint).
- Indistinguishable from regular owner discount except by `description` text.

**No loyalty / referral / wallet uses Credits table.** Those are separate:
- `tenant.kyc_credit_balance` — KYC verification cash credits (different domain entirely)
- `wallet_entry` / `wallet.ts` controller — RentOk wallet ledger
- FlexiPe (`reports.ts:8854`) — credit/debit settlement product

### 3.2 Owner-vs-RentOk P&L flow ([reports.ts:1880-1944](../../../../rentok-backend/src/controllers/reports.ts#L1880))

In the existing **owner credits vs eazypg credits report**:
- For online-payment modes (settled by RentOk): `eazypg_credit = settlement_amount - paid_amount`. `owner_credit = payment.credits_used - eazypg_credit`.
- For cash/non-online: `owner_credit = payment.credits_used` entirely.
- This means a single payment can split its `credits_used` total across owner_credit + eazypg_credit at settlement time — the split is computed at report-time, not stored.

**Operator implication:** "owner-funded" vs "RentOk-funded" view on DA-05 should reconcile to this report's logic, NOT to a naive `credits.source` filter. Two different splits exist:
- `Credits.source` = who **issued** the credit
- `payment.owner_credits` / `payment.eazypg_credits` = who **paid** for it at settlement

These usually agree for `source=0` credits, but mode 213 payments can shift the boundary.

### 3.3 Mode 213 (EazyPG/Circle Pe credits handling)

Confirmed in [src/v1/list_screens/collections/helpers.ts:416-426](../../../../rentok-backend/src/v1/list_screens/collections/helpers.ts#L416):
> "mode 213: subtracts owner_credits from the total"

Formula: `WHEN sub.payment_mode = 213 THEN raw_total_amount - payment_owner_credits`.

Means DA-02 Net Collected already deducts owner credits for mode 213. **DA-05 must NOT double-count those credits as "discount issued" in periods where the credit was both issued AND consumed via mode 213** — they're already showing up as a deduction in DA-02.

### 3.4 Credit state machine (reconstructed)

```
[CREATE]
  via addCredits/addCreditsFromTenantPhone (operator)
    → source=0, status=0, view_status=101, date_added=now, expiry=now+30d (or input)
  via addCreditFromFirebase (RentOk sync)
    → source=1, status=0, view_status=102 (unscratched)

[SCRATCH] (RentOk-funded only)
  scratchCredit → view_status: 102 → 101

[APPLY-TO-PAYMENT]
  Step A: fetchCreditsToApply returns applicable list (operator selects)
  Step B: payment flow assigns credit.payment_id via linkCreditToPayment
          (BROAD UPDATE — links ALL credits for tenant+property, not just selected)
  Step C: paymentService flips status=1, date_used=now (separate code path)

[USE]
  status=1, date_used=set, payment_id=set

[DELETE] (operator)
  deleteCredits → hard remove; only if status=0 AND source=0

[REFUND BREAKAGE — HB1]
  refund of payment → NO update to linked credits
  → credit remains status=1, date_used=set forever
  → operator must manually re-create

[EXPIRY] — NO CRON
  Only enforced lazily at fetchCreditsToApply via expiry_date >= now filter.
  Expired credits remain in DB with status=0 forever.
  Operators NOT notified. No "expired credits" surface.
```

### 3.5 Tenant credit balance surface

`fetchTenantCredits` returns:
- All credits (used + unused, expired + active, scratched + unscratched) for tenant
- `total_credit_amount` = naive sum of ALL `credit.amount`, including used and expired — **misleading "balance"**. Operators using this number get inflated figures.

No endpoint exists that returns "unused applicable balance" — DA-05 could fill this gap.

### 3.6 TeamPassbook integration

Searched `team_passbook` / `TeamPassbook` in services — **no discount/credit entries found**. Discounts don't move cash through TeamPassbook. This means: no per-team-member discount accountability via passbook today. DA-05's `initiated_by` slice IS the only path to "who's giving away how much".

### 3.7 Activity log — only add

```
ACTIVITY_LOG_DISCOUNT = 4  (src/helpers/constants.ts:8)
```

Call sites:
- `credits.ts:516` — fires in `addCredits` after save
- `credits.ts:609` — fires in `addCreditsFromTenantPhone` after save
- `services/cron/agreementRenewal.ts:14` — imported but verify usage (likely auto-renewal discount)

**Not fired on:** scratch, delete, link-to-payment, status flip. Audit trail for discounts is creation-only.

### 3.8 Bulk campaign primitive (cohort heuristic)

The Build Sheet's "bulk campaign" heuristic (≥5 credits, 60s window, same initiated_by, same description) is the only retrospective way to detect bulk creation because **`addBulkCreditsFromPhone` writes one row at a time via `addCreditsFromTenantPhone` in a loop** — no campaign_id, no batch_id, no parent record. The heuristic IS the only signal. No way to know if 5 single-add calls in 60s are a campaign vs coincidence.

---

## 4. Existing operator-facing surfaces (what they ALREADY see)

| Surface | Where | What they see |
|---|---|---|
| Tenant detail → credits tab | Mobile app (calls `fetchTenantCredits`) | Full credit history, used + unused mixed, naive total |
| Collect Payment flow | Mobile (calls `fetchCreditsToApply`) | Only applicable, split owner/eazypg |
| Owner Credits vs EazyPG Credits report | [reports.ts:1880](../../../../rentok-backend/src/controllers/reports.ts#L1880) | Per-payment split for settlement reconciliation |
| Scratch card UX (tenant side) | Tenant app (`fetchUnscratchedTenantCredits` + `scratchCredit`) | Tenant scratches to reveal RentOk-funded credit |
| Add discount action | Operator app (`addCredits`) | Free-text description, amount, optional expiry |
| Activity log entry | Property activity feed | "Discount of Rs X added to tenant Y" — add only |

**No existing surface shows:** total discounts issued per period, per-team-member breakdown, per-category breakdown, expired credits, broken-by-refund credits, bulk-campaign detection.

---

## 5. Dead code / hardcoded-empty / broken-state findings

| ID | Finding | File:Line | Severity |
|---|---|---|---|
| HB1 (confirmed) | Refund doesn't flip credit status. Silent data corruption. | services/refunds/ — zero credit refs | Critical |
| HB2 (confirmed) | `deleteCredits` has no permission key check beyond HeaderValidator | [credits.ts:246](../../../../rentok-backend/src/controllers/credits.ts#L246) | High |
| HB3 (confirmed) | `ACTIVITY_LOG_DISCOUNT` only on add. Delete/scratch/link silent. | [credits.ts:516,609](../../../../rentok-backend/src/controllers/credits.ts#L516) | High |
| NEW-1 | `linkCreditToPayment` mass-updates all tenant credits at property — does NOT filter to selected credits. Race-prone. | [credits.ts:175-189](../../../../rentok-backend/src/controllers/credits.ts#L175) | High |
| NEW-2 | `fetchTenantCredits.total_credit_amount` sums ALL credits incl. used + expired — misleading "balance" | [credits.ts:228](../../../../rentok-backend/src/controllers/credits.ts#L228) | Medium |
| NEW-3 | No expiry cron. Expired credits sit in DB with `status=0` forever. Lazy-filtered only at fetch. | services/cron/ — no expiry job | Medium |
| NEW-4 | `initiated_by` is display name not UUID. Renaming team member breaks attribution. | [credits.ts:485](../../../../rentok-backend/src/controllers/credits.ts#L485) | Medium |
| NEW-5 | Firebase-sync endpoints (`addCreditFromFirebase`, etc.) have no auth at all. | [routes/credits.ts:122-124](../../../../rentok-backend/src/routes/credits.ts#L122) | Security (out of DA-05 scope) |
| NEW-6 | `addBulkCreditsFromPhone` has no zod validation schema, unlike siblings. | [routes/credits.ts:135](../../../../rentok-backend/src/routes/credits.ts#L135) | Low |

No DA-04-style "hardcoded `{}` admin-ledger" pattern found in credit endpoints. Closest analog: `fetchTenantCredits.total_credit_amount` returns a number that's technically wrong (NEW-2) — semantic dead value, not literal empty.

---

## 6. Candidate questions for Phase 3 gather

LAYER 2 surfaces these NEW operator questions the Brief should answer (in addition to the Build Sheet's known PBs/HBs):

**On the credit type taxonomy (Section 3.1):**
- Q-A: Does the operator see "scratch card credits (source=1)" as a discount on DA-05, or should source=1 be filtered out entirely? (They didn't issue it; they don't own the P&L; but it shows on the tenant ledger.)
- Q-B: Should `addCreditsFromTenantPhone` (description="Management discount") be merged into the regular discount stream or surfaced as its own category?

**On categorization (Section 2.1, 3.8):**
- Q-C: ILIKE-on-description is fragile. Should DA-05 V1 expose category at all, or just show free-text description and defer categorization to V2?
- Q-D: Should we add a `category` column to credits to make this robust? (Schema change — flag.)

**On bulk campaigns (Section 3.8):**
- Q-E: Without a campaign_id, the 60s/5-credit heuristic will both miss real campaigns (slow operators) and false-positive coincidences. Is the operator value worth the noise?
- Q-F: Should V1 just show "bulk created" badge using the heuristic and defer real campaign tracking to V2?

**On the broken-state surfaces (Section 5):**
- Q-G: Should DA-05 V1 surface the HB1 "refund-orphaned credits" as a flag? (Adds value to operator; doesn't fix the bug.)
- Q-H: Should DA-05 show expired-but-unused credits (NEW-3)? Operators currently have zero visibility.

**On balance / outstanding (Section 3.5):**
- Q-I: Should DA-05 expose "total active applicable credits per property" — the number that doesn't exist today? This is a natural fit for the widget.

**On settlement reconciliation (Section 3.2, 3.3):**
- Q-J: Mode 213 already deducts owner_credits in DA-02 Net Collected. Should DA-05 explicitly call out mode-213 credits as "already reconciled" to avoid operator double-counting?

**On scope of "discount":**
- Q-K: Does the operator consider `invoices.discount` (the line-item discount on invoices, used by DA-01) part of the same mental model as Credits-table discounts? Two separate surfaces today; one PRD or two?

---

## 7. V1 → V2 architecture decisions surfaced

**V1 (current Build Sheet scope) — design constraints:**
- Description-based ILIKE category is the only option (no schema change in V1)
- `initiated_by` filter must use display-name string, not UUID
- Bulk-campaign detection via heuristic only
- HB1 surfaced as flag, not fixed
- Activity-log coverage stays creation-only

**V2 candidates to flag (don't build yet):**
1. **Schema: add `category` enum + `campaign_id` to credits.** Unlocks reliable category filter + real bulk-campaign tracking. Medium migration cost (242 entities extend BaseEntity, no synchronize).
2. **Refund→credit unwind.** Fix HB1 in services/refunds/. Touches financial logic — needs separate spec.
3. **Permission keys: `view_discounts`, `edit_discounts`, `delete_discounts`.** Add to teamMemberProperty. Touches signup defaults and login projections.
4. **Expiry cron.** Daily job to mark expired credits + notify operators of upcoming expiries.
5. **Credit ledger view per tenant.** Replace `fetchTenantCredits` with a properly-bucketed (active / used / expired / refund-orphaned / unscratched) endpoint.
6. **`linkCreditToPayment` selective update.** Fix the mass-update bug (NEW-1) — only link credits operator actually picked.
7. **Activity log full coverage.** Fire on delete + scratch + link + status flip.
8. **TeamPassbook integration for discount accountability** (if Operations wants per-team-member P&L impact tracking).

**Sequence recommendation:** Ship V1 with HB1/HB2/HB3 as visible flags + Q-I balance widget (high operator value, no schema risk). Then V2 schema change (category + campaign_id) → V2.1 refund unwind → V2.2 permissions.

---

**End of exploration.** Phase 3 gather should drive question set in Section 6, plus the existing Build Sheet PB/HB list.
