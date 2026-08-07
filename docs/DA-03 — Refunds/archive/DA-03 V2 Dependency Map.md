---
title: DA-03 V2 Scope — Refund Lifecycle + Passbook + Liability Cross-Screen Dependency Map
date: 2026-05-16
tags: [rentok, brief, refunds, da-03, v2-scope, lifecycle, passbook, liability]
companion_to: "[[DA-03 Brief]] (V0.2.3 → ships as V1; this doc scopes V2)"
status: V2 scoping note · ready for V2 brief upgrade
---

## TL;DR

V0.2.3 treats Refunds as a record of past disbursements. Codebase reveals **three under-explored dependencies**:

1. **Refunds have NO state machine.** No `status`, no `gateway_refund_id`, no settlement column. A refund row exists only if money is already disbursed. "Pending refund" is **off-domain** in V1 of the build — would need gateway integration + new schema.
2. **Refunds-from-PF (staff fronted personal money)** is a tracked flow via `getOpenPfForRefundUuid`, but the refund detail screen has `reimbursement_details: []` **hardcoded empty** (UI shell exists, never wired).
3. **DA-06 Liability cross-screen is real and live.** Every Security Deposit refund directly decrements `Deposits Held` via the homepage aggregator at `v1/homepage/service.ts:2118-2161`. V1 of DA-03 doesn't mention this; V2 should bridge it.

Plus: deleted refunds (JSONB snapshot in `deleted_refunds` + paired `REFUND_REVERSE` transaction in team passbook) are queryable for audit but invisible in V1.

**V1 ships with the gap acknowledged. V2 adds the lifecycle, passbook, and cross-screen liability dimensions.**

---

## 1. Domain dependency map (codebase-grounded)

### Refund entity — no state machine exists

```
   ┌──────────────────────────────────────────────────────┐
   │ refunds (entity) — disbursement record only          │
   │ src/entities/refunds.ts:5-43                         │
   │                                                       │
   │ Columns:                                              │
   │   id, amount, refund_date, refund_reason             │
   │   refund_mode, refunded_by_*, refund_images          │
   │   invoice_id (FK → invoices.id) — single invoice only│
   │                                                       │
   │ NO status. NO gateway_refund_id. NO settled_at.      │
   │ A refund row exists IFF money already disbursed.     │
   └──────────────────────────────────────────────────────┘
```

**Confirmed by absence:**
```
grep -rln "gateway_refund\|refundViaGateway\|initiateRefund\|refund_status\|isRefundSettled" src/
→ zero hits
```

**Implication:** No "pending settlement" cohort to query in V2. Settlement delay does not exist as a domain concept today. V2 questions like *"How many refunds are pending settlement?"* are **blocked at the schema level** — would need a separate domain extension (new entity `refund_status`, gateway-payout integration) before they can answer.

### Refund deletion = JSONB snapshot + paired REFUND_REVERSE

```
   ┌──────────────────────────────────────────────────────┐
   │ Delete flow                                          │
   │ src/controllers/invoices.ts:8351-8360                │
   │                                                       │
   │  await deletedRefundsRepository.insert({             │
   │    refund_uuid: refund_data.id,                      │
   │    refund_json: refund_data                          │
   │  });                                                  │
   │  await refund_data.remove();                         │
   │  await TeamPassbookService.deleteRefund({…});        │
   └──────────────────┬───────────────────────────────────┘
                      │
                      ▼
   ┌──────────────────────────────────────────────────────┐
   │ deleted_refunds (entity)                             │
   │ src/entities/deletedRefunds.ts:18                    │
   │  - refund_uuid                                       │
   │  - refund_json: jsonb (full snapshot)                │
   └──────────────────────────────────────────────────────┘
                      │
                      ▼ (passbook side)
   ┌──────────────────────────────────────────────────────┐
   │ team_member_transactions row (REFUND_REVERSE)        │
   │ src/services/teamPassbook/teamPassbookOperationFactories/
   │   teamPassbookRefundService.ts:117                   │
   │  - category = REFUND_REVERSE                         │
   │  - balance_type inverted from original REFUND row    │
   └──────────────────────────────────────────────────────┘
```

**Implication:** A "refund reversal history" view is achievable today by reading `deleted_refunds` + matching `team_member_transactions WHERE category = REFUND_REVERSE`. **But the active `refunds` table alone tells V1's story — deleted refunds are invisible in V1 totals by design.** V2 audit panel must source from both tables.

### Passbook integration — the open-PF concept

```
   ┌──────────────────────────────────────────────────────┐
   │ services/refunds/refunds.ts:398-413                  │
   │   Fund priority for refunds: AF → NPNAF → PF         │
   │   PF (staff personal money) consumed LAST            │
   └──────────────────┬───────────────────────────────────┘
                      │
                      ▼ (when staff fronts PF)
   ┌──────────────────────────────────────────────────────┐
   │ getOpenPfForRefundUuid                               │
   │ services/teamPassbook/.../teamPassbookReimbursement.ts:83-121
   │                                                       │
   │ SQL parses team_member_transactions.remark for       │
   │   "refund id: <uuid>"                                │
   │ Sums PF-fund-id transactions in                       │
   │   (REFUND, REFUND_REVERSE) categories                │
   │ Returns GREATEST(0, -signed_sum)                     │
   └──────────────────┬───────────────────────────────────┘
                      │
                      ▼ (auto-allocated to reimbursement runs)
   ┌──────────────────────────────────────────────────────┐
   │ autoAllocatePf                                       │
   │ teamPassbookReimbursement.ts:198-235                 │
   │                                                       │
   │ When owner pays reimbursement to staff, this finds   │
   │ open PF refund rows and allocates them.              │
   └──────────────────────────────────────────────────────┘
```

**Plain-English:** Staff member fronted personal money (PF = Personal Fund) to refund a tenant. "Open PF" = how much of that the owner still owes the staff member back.

**Current refund detail screen state:** `services/refunds/refunds.ts:600-714` builds `payment_source_breakup = {AF_SUM, PF_SUM, NPNAF_SUM}` and surfaces `show_reimbursement = true` when PF is involved. **But:** `reimbursement_details: []` is HARDCODED EMPTY at `services/refunds/refunds.ts:715`. The UI block to show "still owed to staff" exists but is never populated. **V2 should wire this.**

### Linkage chain — single invoice only

`entities/refunds.ts:38-43`: Refund has exactly ONE `invoice_id` FK. **Refunds CANNOT span multiple invoices at the entity level.** A multi-invoice "spread refund" is modelled as N refund rows, one per invoice, with the same `refund_date / refunded_by` — the spread is implicit, **no `group_id` exists**.

Orchestrator: `services/refunds/refunds.ts:417-479` loops `invoice_bifurcations`, calls `addRefundToInvoice` per invoice. No correlation id is written.

**Chain:** `refund → invoices.invoice_id → payments_invoices (M:N) → payments` (and via payment → deposit/rent invoice via `due_type`).

Deposit refunds vs rent refunds **share the same entity, differentiated only by `invoice.due_type IN ('Security Deposit', 'Caution Money')`** — there is no separate `deposit_refund` table.

### DA-06 Liability cross-screen dependency — real and live

```
   ┌──────────────────────────────────────────────────────┐
   │ v1/homepage/service.ts:2118-2161                     │
   │                                                       │
   │ Deposits Held = SUM(                                 │
   │   gross_amt − refund_amt − adjustment_amt            │
   │ )                                                     │
   │                                                       │
   │ where refund_amt = (                                 │
   │   SELECT SUM(r.amount)                               │
   │   FROM refunds r                                     │
   │   WHERE r.invoice_id = inv.id                        │
   │ )                                                     │
   │                                                       │
   │ filtered on:                                         │
   │   due_type IN ('Security Deposit', 'Caution Money')  │
   └──────────────────────────────────────────────────────┘
```

**Every Security-Deposit refund directly decrements DA-06 Liability total in real time.** This is THE cross-screen dependency the V1 brief doesn't surface.

Homescreen "Deposits Held" widget already shows a "Less: Refunds" line (`v1/homepage/service.ts:2399`) — the DA-03 → DA-06 mental model is established in the codebase, just not in V1 of the DA-03 brief.

### DA-02 Collections cross-screen — real but subtractive

`v1/list_screens/collections/helpers.ts:713-724`: Net collection = `SUM(inv.amount − total_refunded)` via the `ref_agg` join. Refunds reduce Net Collection by `refund.amount` per invoice. V1 brief already mentions this in the V0.2 critique; consistent with codebase.

### Bookkeeping — no separate accounting flow

`grep -rln "bookkeep\|accounting.*refund\|monthly.*pnl\|p_and_l"` returns nothing in refund code paths. Refunds affect P&L **only indirectly** via the Collections / Liability rollups above. No monthly-close / GL posting layer.

---

## 2. Existing operator-facing surfaces I may have missed

| Surface | File | Shows | Missing for V2 |
|---|---|---|---|
| Refund detail screen | `refunds.ts:488-826` | `payment_source_breakup` (AF/PF/NPNAF), `show_reimbursement` flag, single tenant card, single invoice link | No "open PF still owed to staff" rupee figure; `reimbursement_details: []` is **hardcoded empty**; no reversal/delete history; no link to the reimbursement run that paid the staff back |
| Refund metadata for add-flow | `refunds.ts:134-271` | `total_fund_balance` (AF/PF/NPNAF) + per-invoice `refundable_amount` | Operator never sees "refunded so far" as a standalone period number in this flow |
| Routes | `routes/refunds.ts` | Only `/metadata`, `/advanced-addition`, `/advanced-details` | **No list / aggregate / analytics endpoint exists today** — confirms DA-03 V1 is greenfield analytics |
| Homescreen Deposits widget | `v1/homepage/service.ts:2399` | "Less: Refunds" line under Deposits Held | Already wired — DA-03 → DA-06 mental model is established |

---

## 3. V2 question list — ranked by operator pain × codebase feasibility

| # | Question | Pain | Feasible today? | Effort |
|---|---|---|---|---|
| **Q10** | **"This deposit refund just decreased my liability by ₹X — show me the running liability impact"** (DA-03 ↔ DA-06 bridge) | High — real money trail | Yes — joinable on `invoice.due_type`; DA-06 already does the math | S |
| **Q11** | **"Who fronted refund money from their personal pocket, and how much do I still owe them?"** (per-team-member open-PF) | High — staff trust issue, money owed | Yes — `getOpenPfForRefundUuids` already aggregates | S |
| **Q12** | **"Show me deleted / reversed refunds in this period"** (audit trail) | High — fraud signal: refund deleted to hide an error | Yes — `deleted_refunds` JSONB + `category=REFUND_REVERSE` | S-M (jsonb parse) |
| Q13 | "Which refunds spanned multiple invoices — same date, same staff, same tenant?" (spread-refund detection) | Medium — operator can't see spread as one event today | Yes via heuristic (`refund_date + refunded_by_phone + invoice.payer`), but no `group_id` exists — false-positive risk | M |
| Q14 | "Deposit refunds vs rent-adjustment refunds split + average held-duration per category" | Medium — partly in V1 (Q2 + Q8) | Yes | S |
| **Off-scope** | "How many refunds are pending settlement / what's average settlement delay" | Low (today) | **No — domain concept does not exist.** All refunds are recorded post-disbursement | Blocked — needs gateway integration first |
| Q15 | "Refund reversal / void history per refund" (per-row trail) | Medium | Yes — same source as Q12, scoped per refund | S |

---

## 4. V1 → V2 architecture risks (decide before V2 lands)

1. **No `refund_status` column today.** If V2 ever needs "pending settlement" or "failed refund," a schema migration + new domain is required. **Decision needed:** does V2 stay manual-disbursement-only, or open the door to digital-refund-via-gateway? If yes, file a separate spec — it is not a DA-03 extension, it is a payments-platform change.

2. **No `refund_group_id` for spread refunds.** Today: N invoice rows = N refund rows, joined only by `(refund_date, refunded_by_phone, payer)`. **Decision needed:** add a `refund_group_id` column when V2 ships, or accept heuristic grouping in the analytics layer with explicit "we may misgroup" caveat.

3. **`deleted_refunds` is JSONB.** Querying by tenant / property / refund_mode requires `refund_json->>'…'` paths — slow at scale, no indexes. **Decision needed:** is reversal-history a V2.1 feature with a real schema, or a V2 read-only "best effort" surface off the snapshot?

4. **`reimbursement_details: []` in refund detail is hardcoded.** `refunds.ts:715` returns an empty array — the UI block exists but is never populated. **Decision needed:** does V2 wire this to `transaction_exchange_allocation` (the reimbursement-run table touched by `autoAllocatePf`) and surface "refund X was reimbursed to staff in run Y"?

5. **No cross-screen contract with DA-06.** V1 of DA-03 talks to DA-02 (Collections) for drill-down. V2 needs an equivalent contract with DA-06 (Liability) — at minimum, tapping a deposit-refund row should route to the same tenant in DA-06 with the deposit's gross / net visible.

---

## 5. Recommended V2 scope for DA-03 Brief

**Top 3 questions, in this order:**

**Q10 — Refunds-from-PF: "Who am I about to reimburse, for what?"**
- Per-team-member rollup of `open_pf_by_refund` in the period
- Tap row → list of refunds that funded it, with tenant + invoice context
- Eng sketch: new endpoint reusing `getOpenPfForRefundUuids` (`teamPassbookReimbursement.ts:153`); ~2 days backend, ~3 days frontend
- Operator framing: *"Meena fronted ₹12K across 4 refunds last week — I still owe her ₹8K back"*

**Q11 — Deposit-refund → Liability bridge**
- Per-deposit-refund row, show the "liability decrement" alongside (and the resulting Deposits-Held tail value)
- Tap row → DA-06 with tenant filter prefilled
- Eng sketch: join exists; the math already runs in `homepage/service.ts:2118` — extract to shared helper, expose per-refund
- Operator framing: *"This ₹15K deposit refund dropped my liability from ₹4.2L → ₹4.05L. Healthy."*

**Q12 — Deleted / reversed refunds audit panel**
- Period view + per-refund history surface, sourced from `deleted_refunds` + `team_member_transactions WHERE category = REFUND_REVERSE`
- Permission-gated under the same `viewCashAudit` key as V1 audit signals
- Eng sketch: jsonb query on `deleted_refunds`; ~3 days backend (jsonb path indexes will be needed if volume is high)
- Operator framing: *"Ramesh deleted 3 refunds yesterday — totalling ₹4K. Why?"*

**Defer to V2.1:**
- Spread-refund grouping (needs `refund_group_id` migration first)
- Per-refund reimbursement-run linkage (needs the empty `reimbursement_details: []` to be wired)

**Off-scope for V2 — flag as PRD-level decision:**
- Settlement-delay / pending-refunds. Domain does not exist. Document explicitly in the V1 Brief's *"What we're NOT building"* section so the next PM doesn't re-litigate it.

---

## 6. Cross-doc traceability

- V1 Brief: `[[DA-03 Brief]]` (V0.2.3 — ships with the gap acknowledged in a "V2 scope deferred" section)
- Build Sheet: `[[DA-03 Build Sheet]]` (V1 — locked at engineering scope; does NOT include passbook integration or liability bridge)
- Sibling V2 maps: `[[DA-02 V2 Dependency Map]]` · `[[DA-04 V2 Dependency Map]]`
- Cross-screen dependency: `[[DA-06 Build Sheet]]` (Liability — V2 Q11 needs an explicit data contract with DA-06)

---

## Changelog

| Date | Change | By |
|------|--------|-----|
| 2026-05-16 | Initial V2 scope note — generated from deep codebase exploration sub-agent (refund-entity has no state machine, deleted_refunds JSONB pattern, open-PF flow for refunds, DA-06 liability cross-screen, hardcoded-empty `reimbursement_details`). Findings to inform a future DA-03 V2 brief upgrade. | Sanchay (PM) + Claude sub-agent |
