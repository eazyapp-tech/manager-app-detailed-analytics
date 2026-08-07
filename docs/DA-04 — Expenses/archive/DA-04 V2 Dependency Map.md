---
title: DA-04 V2 Scope — Team Passbook Dependency Map
date: 2026-05-16
tags: [rentok, brief, expenses, da-04, v2-scope, team-passbook]
companion_to: "[[DA-04 Brief]] (V0.2.1 → ships as V1; this doc scopes V2)"
status: V2 scoping note · ready for V2 brief upgrade
---

## TL;DR

V0.2.1 treats Expenses as `SUM(expenses)`. Codebase says Expenses is one node in a 5-entity fund-flow graph: Expense ↔ TeamMemberTransactions ↔ FundSplit (PF/AF/NPNAF/EF) ↔ TransactionExchange (reimbursement) ↔ TmTransactionExpenseMap. **V1 ships with the gap acknowledged.** V2 must add at minimum: (1) AF petty-cash balance per staff, (2) PF reimbursement burden owed-by-owner, (3) per-fund expense split, (4) cash-handover view this period, (5) expense → fund-source reconciliation. Recommended V2 effort: ~1.5x V1.

---

## 1. Domain dependency map (codebase-grounded)

```
                  ┌──────────────────────────────────────────────┐
                  │           expenses (entity)                  │
                  │  src/entities/expenses.ts                    │
                  │   - amount, paid_date, expense_type          │
                  │   - payer (free-text), paid_to, payment_mode │
                  │   - added_by (role enum)                     │
                  └────────────┬─────────────────────────────────┘
                               │ 1:N (via expense_uuid)
                               ▼
            ┌──────────────────────────────────────────────────────┐
            │ tm_transaction_expense_map                           │
            │ src/entities/tmTransactionExpenseMap.ts              │
            │   - tmt_id  ──► team_member_transactions.id         │
            │   - expense_uuid ──► expenses.id                     │
            └────────────┬─────────────────────────────────────────┘
                         │ 1:1
                         ▼
   ┌─────────────────────────────────────────────────────────────────┐
   │ team_member_transactions (TMT)                                  │
   │ entities/teamMemberTransactions.ts                              │
   │   - team_uuid, property_id, pg_id                               │
   │   - amount, balance_type (1=inflow, 2=outflow)                  │
   │   - fund_id ∈ {PF, AF, NPNAF, EF}  ← split_array fan-out        │
   │   - category ∈ CATEGORY_MAP (Expense, Handover, Reimbursement…) │
   │   - exchange_id (joins to TransactionExchange)                  │
   └────────┬─────────────────────────────────────────────────┬──────┘
            │                                                 │
            ▼                                                 ▼
  ┌────────────────────────────┐         ┌──────────────────────────────────┐
  │ transaction_exchange       │         │ FUND_ID semantics                │
  │ entities/transactionExch.. │         │ src/services/teamPassbook/       │
  │  - type = REIMBURSEMENT |  │         │   constants.ts:28-40             │
  │    HANDOVER (in handover)  │         │   teamPassbook.ts:88-94          │
  │  - from_team_uuid          │         │                                  │
  │  - to_team_uuid            │         │ AF    = "Petty cash"             │
  │  - amount                  │         │ PF    = "Personal funds"         │
  └────────────┬───────────────┘         │ EF    = "Exchange funds"         │
               │ 1:N                     │ NPNAF = "Collected funds"        │
               ▼                         └──────────────────────────────────┘
  ┌──────────────────────────────────────┐
  │ transaction_exchange_allocation      │
  │ entities/transactionExchangeAlloc..  │
  │  - exchange_id, target_type          │
  │    (EXPENSE | REFUND | PAYMENT)      │
  │  - target_uuid, fund_id, amount      │
  │  → links a reimbursement EVENT       │
  │    back to the expense(s) it paid off│
  └──────────────────────────────────────┘
```

**Cited write-paths:**
- Expense add → `services/teamPassbook/teamPassbookOperationFactories/teamPassbookExpenseService.ts:38-50` fans `split_array` into N TMT rows (one per fund_id).
- Reimbursement creates **two TMT rows** + N allocations: `teamPassbookReimbursement.ts:423-449` — `from_team` gets `EF` outflow, `to_team` gets `PF` inflow.
- Handover: `teamPassbookHandover.ts:111-145` — `from_team` (owner) outflow no fund_id; `to_team` (staff) inflow as **AF** (petty cash).
- Auto-allocate PF against open expenses: `teamPassbookReimbursement.ts:198-289`. The `getOpenPfForExpenseUuid` query (`teamPassbookReimbursement.ts:57-81`) computes per-expense un-reimbursed PF balance.

---

## 2. What EF is (the 4th fund I missed)

**EF = "Exchange Funds."** Only written in one place — `teamPassbookReimbursement.ts:434` — as the from-team's outflow leg when a reimbursement is paid out. It's a bookkeeping fund, not real money. It's the contra-account that balances the double-entry when owner reimburses staff: owner-side TMT debits `EF`, staff-side TMT credits `PF`.

**Operator implication for V2:** EF balances are zero-sum across owner+staff; **never expose EF to operators as a metric.** It's an internal ledger device. If V2 ever shows "per-fund expense split," exclude EF. Label this in the V2 brief so future eng/PM doesn't surface it.

---

## 3. Existing operator-facing surfaces V0.2.1 missed

| Surface | Endpoint | File:line | Shows | Missing |
|---|---|---|---|---|
| **Fund Balances** | `POST /teamPassbook/fund-balances` | `routes/teamPassbook.ts:24`, `services/teamPassbook/teamPassbook.ts:1413-1524` | Current PF/AF/NPNAF/EF balance per `team_uuid` + scoped property | No period filter; no link from Expenses screen; operator must navigate to passbook |
| **Team Passbook Widgets** | `GET /teamPassbook/widgets` | `routes/teamPassbook.ts:44`, `teamPassbook.ts:2393` | 8 widgets incl. Total Expenses, Handovers, Give-money, Receive-from-owner (constants.ts:111-225) | Period-bound, already aggregates expense but tied to passbook view, not analytics |
| **Detailed Screen** | `GET /teamPassbook/detailed-screen` | `routes/teamPassbook.ts:48`, `teamPassbook.ts:2770` | Per-TMT drill (`tmt_id`) — block/banner/remark structure | Operator-pain unclear; not periodized; not a list |
| **Passbook Details (v1+v2)** | `GET /teamPassbook/details` + `/details-v2` | `routes/teamPassbook.ts:20-22`, `teamPassbook.ts:1751,2129` | Per-staff TMT ledger | Staff-centric, not expense-centric |
| **Admin Ledger** | `GET /teamPassbook/admin-ledger` | `routes/teamPassbook.ts:36`, **service stub at `teamPassbook.ts:3529-3555` is commented out** | Nothing — controller calls a service that returns `{}` | **Endpoint is dead.** Not safe to depend on for V2. |
| **Handover Team List** | `GET /teamPassbook/handover-team-list` | `routes/teamPassbook.ts:31` | Staff eligible for cash handover | No outflow visibility |
| **Reimbursement add/list** | `POST /reimbursement/advanced-addition`, `GET /reimbursement/advanced-details` | `routes/reimbursement.ts:9-10`, `services/reimbursement/reimbursement.ts` | Add a reimbursement; fetch reimbursement detail by id | **No list-by-period endpoint, no "owed-to-staff" queue, no per-staff PF-open total.** This is the biggest operator-pain gap. |

**Punchline:** operators today see fund balances on Passbook, expenses on Expense worklist, reimbursements only when adding them. **There is no single screen showing "what I spent this period AND what I still owe my staff for it."** V0.2.1 perpetuates that.

---

## 4. V2 question list (ranked by operator pain)

### MUST add to V2 (daily/weekly pain)

| # | Operator question | Why painful | Data path |
|---|---|---|---|
| **V2-Q1** | *"How much do I owe my staff in reimbursement right now? To whom?"* | Owner uses gut today. Staff fronted cash from PF; until reimbursed, owner owes them. No screen surfaces this aggregate. | Sum across staff of `getOpenPfForExpenseUuids` (`teamPassbookReimbursement.ts:123-151`) MINUS already-reimbursed (`getAlreadyReimbursedPfByTargetUuids`:16-55). Aggregate per `to_team_uuid`. |
| **V2-Q2** | *"How much petty cash (AF) does each staff have right now? Who's running low?"* | Owner gives Meena ₹5K Monday morning; by Thursday he has no idea what's left. He either over-funds (cash sitting idle) or under-funds (staff tops up from PF, creating reimbursement debt). | `fund-balances` endpoint already returns AF per `team_uuid` — but not aggregated across staff for a property in one view. Roll up via `teamPassbook.ts:1413` per staff. |
| **V2-Q3** | *"Of this period's expense total, how much came from each fund source (AF / PF / NPNAF)?"* | Today the headline is just `SUM(expense.amount)`. Doesn't distinguish "₹40K from owner's petty cash" from "₹30K staff fronted from their pocket" from "₹10K from collected rent." These three scenarios have **completely different cash-management implications**, but V1 shows one number. | TMT rows for category=EXPENSE, grouped by `fund_id`. Existing query in `getFundBalances` (`teamPassbook.ts:1440-1450`) can be cloned with category+period filter. Exclude EF. |

### SHOULD add to V2 (periodic but real pain)

| # | Operator question | Why painful | Data path |
|---|---|---|---|
| **V2-Q4** | *"What did I hand over to which staff this period?"* | Periodic (weekly/monthly cash distribution). Operator gives Meena ₹5K, Ramesh ₹3K, then forgets the totals. Widget id=5 (`Total Cash Handed over` in constants.ts:176-186) exists but isn't on Expenses. | TMT rows where `category=CASH_HANDOVER`, grouped by `to_team_uuid` (via `exchange_id` join to `transaction_exchange.to_team_uuid`). |
| **V2-Q5** | *"Reimbursement burden this period: how much did I pay out to staff this month vs last?"* | Owner wants to know reimbursement velocity — is staff fronting more cash this month, or am I catching up? | TMT rows where `category=REIMBURSEMENT`, period-bound. MoM compare. |
| **V2-Q6** | *"Per-expense: was it reimbursed yet? By how much?"* | When Rajesh drills into an expense, he can't tell if Meena's already been paid back or if it's still open against PF. | Join `expenses` → `tm_transaction_expense_map` → `transaction_exchange_allocation` (target_type=EXPENSE). Per-expense `remaining_pf = open_pf - already_reimbursed_pf`. |
| **V2-Q7** | *"Total inflow from owner this period (money I funded the property with)."* | Owner top-ups petty cash; he wants to see "I've put in ₹50K this month already." Today this lives on the passbook widget id=8 (`RECEIVE_MONEY_FROM_OWNER` in constants.ts:213-225) but not in expense context. | TMT rows where `category=RECEIVE_MONEY_FROM_OWNER`, period-bound. |

### MAYBE V3 (niche / defer)

- **V2-Q8 — Per-fund per-category breakdown** (e.g., "Salary paid from AF vs NPNAF"). Niche — only matters for advanced multi-property accounting.
- **V2-Q9 — Reimbursement-cycle-time** (avg days from expense add to reimbursement settle). Strategic only.
- **V2-Q10 — Expense "fronted-by-staff" alert** (callouts when one staff has fronted > ₹X uncovered for > N days). Useful but premature; build V2-Q1/Q6 first; if operators ask for it, ship in V3.

---

## 5. V1 → V2 architecture risks

These need decisions **before V2 build starts** (V1 ships fine without them; V2 will break if any are unresolved):

1. **Headline arithmetic divergence.** V1 headline = `SUM(expenses.amount)` for the period. V2-Q3 will compute the same number from the TMT side as `SUM(amount where category=EXPENSE)`. **These two numbers will not match** in legacy data (expenses created before `team_passbook` migration won't have TMT rows; expenses with `is_active=0` may still have TMT rows; manual ledger corrections diverge). Decide: does V2 headline switch to TMT-source-of-truth, or keep `expenses` table as source and treat fund-split as a metadata overlay? **Recommend: keep `expenses` table as headline source; show fund-split as a stacked-bar overlay with a clear "N expenses missing fund-split data" footnote when divergence exists.**

2. **EXPENSE_REVERSE handling in V2-Q3/Q5.** `CATEGORY_MAP.EXPENSE_REVERSE` (constants.ts:9) is an INFLOW. V1 headline doesn't subtract reversals (Brief V0.2.1 says "Refunds and reversals do NOT appear here"). But V2 fund-split breakdown using TMT will naturally include EXPENSE_REVERSE rows unless filtered. **Decide before V2: do reversals count? Same answer as V1 (no) keeps the screens consistent. Document explicitly.**

3. **NPNAF "Collected funds" — is it really an expense source?** When operator pays a vendor from collected rent, that's TMT `category=EXPENSE, fund_id=NPNAF`. But the operator never deposited that money — it's tenant rent re-deployed. V2-Q3 needs a clear label: is NPNAF outflow shown as "expense from collected rent" (operator-meaningful) or as a separate category? **Recommend: explicit label "From collected rent" not "NPNAF."**

4. **Reimbursement-owed bucket overlaps with expenses.** V2-Q1 shows owed-PF total. The expense that created that PF debt is already counted in V1 headline. **Operator may double-count mentally:** "I spent ₹100K AND I owe staff ₹20K, so total cost is ₹120K?" No — the ₹20K is part of the ₹100K. UX/copy must make this obvious. **Decide before V2: do we show owed-PF as a sub-line on the headline, or as a separate "money you owe" card?** Recommend separate card with explicit copy *"of the ₹100K spent, ₹20K is owed back to staff."*

5. **Admin ledger endpoint is dead.** `/teamPassbook/admin-ledger` controller hits a commented-out service (`teamPassbook.ts:3529-3555`) and returns nothing. **Do not depend on it for V2.** If V2 needs an admin ledger view, build the service. Flag this to eng before V2 planning.

6. **Free-text payer field overlaps with `team_uuid`.** V1 Q3 (By payer) uses `expenses.payer` (free text, e.g., *"Jatin"*). V2-Q1/Q2 use `team_uuid` (system identity). **These don't reconcile.** "Jatin K" the free-text payer may or may not equal `team_uuid=abc-123`. V2 should prefer `team_uuid` (via TMT join) over free-text payer for reimbursement/handover questions. Document that V1 Q3 and V2-Q1 are answering different operator questions even though both name "the team member."

---

## 6. Recommended V2 scope (slot into Brief)

### V2 must-ship (operator daily-pain answers)

- **V2-Q1: Reimbursement owed-to-staff.** Per-staff total + grand-total + drill into open expenses. Answers *"who do I owe right now?"* Engineering: 1 new aggregation query joining `team_member_transactions` + `transaction_exchange_allocation`; ~2 days.
- **V2-Q2: Per-staff petty-cash (AF) balance.** Live snapshot (NOT period-bound). Engineering: extend existing `fund-balances` endpoint to return per-staff array for a property; ~1 day.
- **V2-Q3: Expense source breakdown (AF / PF / NPNAF, exclude EF).** Period-bound stacked breakdown of the V1 headline. Answers *"where did the money come from?"* Engineering: TMT-side aggregation with reconciliation footnote when sum diverges from `expenses` table; ~2 days incl. divergence handling.

### V2 should-ship

- **V2-Q4: Cash handover view this period.** Per-staff handed-over total + MoM. Reuses existing widget aggregation logic.
- **V2-Q5: Reimbursement-paid this period + MoM.** Mirror of V2-Q1 but on settled side.
- **V2-Q6: Per-expense reimbursement state on Expense detail drill.** Not a new tile — add a status badge ("Reimbursed ₹X of ₹Y" / "Fully reimbursed" / "Owner-paid") on the Expense detail screen.

### What V2 adds to the operator picture

V1 answers *"what did I spend?"* V2 answers *"where did that money come from, and what do I still owe staff?"* These are **two halves of the same operator decision** — without V2, the screen tells half the truth. Operators currently fill the other half with gut + WhatsApp asks ("Meena, how much did you front this week?"). V2 closes that loop.

### Eng-effort sketch

- V2 must-ship (Q1+Q2+Q3): **~6 dev-days** (1 senior + 1 junior, parallelized: ~4 calendar days).
- V2 should-ship (Q4+Q5+Q6): **~5 dev-days** (~3.5 calendar days).
- **Total V2 ~ 1.5 weeks of build.** Frontend integration is the long pole — V2 layers atop V1 tiles, so design needs a coherent "Fund flow" section (likely below "By category" and above "Trend chart").

### V1 → V2 sequencing

V1 ships standalone. V2 starts the cycle after V1's headline-integrity QA gate passes. **Do not start V2-Q3 until V1 headline = worklist sum is proven on the 10-property sample** — V2-Q3 amplifies that integrity dependency (any V1 divergence becomes a V2 divergence × number of fund buckets).

### V2 trap (carry into Brief)

- **EF is invisible.** V2 must explicitly exclude EF from any operator-facing aggregation. Cite `teamPassbookReimbursement.ts:434` so eng knows where it comes from.
- **Reimbursement-owed (V2-Q1) and headline (V1 Q1) overlap by design.** UX must explain *"the ₹20K owed-to-staff is a subset of the ₹100K spent."* No subtraction or addition.
- **Admin ledger endpoint is dead code** — do not surface or extend; flag to backend lead.

---

## 7. What I did NOT change in V0.2.1

V0.2.1 is operator-pain-correct for the V1 cut. The gap is **scope completeness**, not framing. V0.2.1's traps about MoM color, free-text payer, profit-loss cross-query, and headline integrity all remain valid in V2.

**Next step:** lift this V2 scope into a new Brief version (V0.3 if continuing the doc, or a "DA-04 Brief V2.md" if you want the V1/V2 split cleanly). Recommend the latter — V1 brief is sacred for the in-flight build.
