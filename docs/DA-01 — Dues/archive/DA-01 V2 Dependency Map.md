---
title: DA-01 V2 Scope — Reminder Cadence + Autopay Coverage + Eviction Pipeline Dependency Map
date: 2026-05-16
tags: [rentok, brief, dues, da-01, v2-scope, reminders, autopay, eviction]
companion_to: "[[DA-01 Brief]] (V0.10 → ships as V1; this doc scopes V2)"
status: V2 scoping note · ready for V2 brief upgrade
---

## TL;DR

V0.10 treats Dues as a static stock — *how much is owed, how old, who owes*. The codebase reveals **three live, operator-controlled flow systems** that shape that stock in real time and are invisible on V1:

1. **Reminder cadence engine** (`sendBulkPaymentReminder`, `sendBulkPaymentReminderForProperty`, property flags `automated_payment_reminders` + `last_bulk_tenant_reminder_date` + invoice `reminder_contact[]`) — Rajesh can't see "did the WA reminder go out this month, to whom, did the dues drop after?"
2. **Autopay V2 subscription engine** (`AutoPayServiceV2`, `autopay_debit_schedule`, `autopay_transaction`, `autopay_debit_schedule_invoice`) — fully built, status enums real, but DA-01 has zero surfacing of "how many bills are on autopay vs manual / how many autopay debits failed this period." This is a separately-billable revenue line operators want visibility on.
3. **Eviction pipeline that auto-creates penalty/move-out invoices** (`TenantEvictionService`, `PropertyEvictionCharges`, `tenant_eviction_invoices`, `tenant_eviction_details`) — when dues exceed thresholds, eviction notices fire and **new invoices get auto-generated and land in the same dues pool**. V1 shows them undifferentiated, so the operator sees a ballooning dues number with no signal that it's eviction-machinery output vs ordinary rent.

Plus three smaller real dependencies: **deposit adjustment at move-out** (`paymentService.adjustDeposit` writes a settle-from-deposit path that reduces dues silently), **invoice scheduler** that forward-creates next-month rent (`invoice_scheduler`), and **future-period guard gap** acknowledged in HB8 but not framed as an operator question.

**V1 ships with the gap acknowledged. V2 adds the action-loop dimensions: did the operator act, did automation act, and did the dues number respond?**

---

## 1. Domain dependency map (codebase-grounded)

### A. Reminder cadence engine (manual + automated)

```
   ┌──────────────────────────────────────────────────────────┐
   │ Operator taps "Send Reminder" CTA on homepage/Dues       │
   │   action: rentok://whatsapp-bulk-reminder                │
   │   homepage/service.ts:2325                               │
   └──────────────────┬───────────────────────────────────────┘
                      │
                      ▼
   ┌──────────────────────────────────────────────────────────┐
   │ POST /invoices/sendBulkPaymentReminder                   │
   │ controllers/invoices.ts:6812 (route :231 HeaderValidator)│
   │   → checks property.automated_payment_reminders flag     │
   │   → checks property.send_onboarding_payment_notifications│
   │   → respects property.last_bulk_tenant_reminder_date     │
   │   → iterates invoices status=0, due_today                │
   │   → writes WA via Meta / 7218 / 7510 templates           │
   └──────────────────┬───────────────────────────────────────┘
                      │
                      ▼
   ┌──────────────────────────────────────────────────────────┐
   │ Invoices.reminder_contact: string[]                      │
   │ src/entities/invoices.ts:153                             │
   │   → append-only ledger of who was reminded               │
   └──────────────────────────────────────────────────────────┘
```

**Sibling endpoint:** `POST /invoices/sendBulkPaymentReminderForProperty` at `controllers/invoices.ts:7169` — used for cron-driven daily reminder run (`scripts/propertyRentReminder.ts:29` hits this localhost). `override_last_sent_check = true` forces re-send.

**HB4** in Build Sheet acknowledges "Send Reminder activity log — must wire `ACTIVITY_LOG_REMINDER_SENT` event for audit trail" — but the activity log doesn't exist yet. Today there is no DB-backed "X reminders sent on Y date by Z operator" — only the per-invoice `reminder_contact[]` array and the property-level `last_bulk_tenant_reminder_date` watermark.

**Implication for V2:** Rajesh has no answer to "did Meena send the reminders this week / how many got nudged / what dropped off after." If HB4 lands, the activity log becomes the source.

### B. Autopay V2 subscription engine (real, live, invisible to V1)

```
   ┌──────────────────────────────────────────────────────────┐
   │ Tenant opts in via app                                   │
   │ → controllers/autopayControllerV2.ts                     │
   │ → AutoPayServiceV2.createSubscription()                  │
   │   src/services/autoPay/autopayV2.ts:42                   │
   │   - tenant_id + due_type_id ('Rent' only, line 81)       │
   │   - status: "initialized" → Cashfree → "active"          │
   └──────────────────┬───────────────────────────────────────┘
                      │
                      ▼
   ┌──────────────────────────────────────────────────────────┐
   │ AutoPay entity (status: string — 'initialized'/'active'/ │
   │  'paused'/'cancelled')                                   │
   │ src/entities/autopay.ts                                  │
   └──────────────────┬───────────────────────────────────────┘
                      │
                      ▼
   ┌──────────────────────────────────────────────────────────┐
   │ autopay_debit_schedule (per-month execution row)         │
   │ src/entities/autopay_debit_schedule.ts:23                │
   │   AutopayDebitScheduleStatus enum (line 15):             │
   │     0 PENDING · 1 PROCESSING · 2 RESOLVED                │
   │     3 SKIPPED  · 4 INTERIM                               │
   │   debit_date · amount · last_attempt_at                  │
   └──────────────────┬───────────────────────────────────────┘
                      │
                      ▼
   ┌──────────────────────────────────────────────────────────┐
   │ autopay_debit_schedule_invoice (N:N → invoices)          │
   │ src/entities/autopay_debit_schedule_invoice.ts           │
   │   FK → invoice_id, amount, preferred_debit_date          │
   └──────────────────┬───────────────────────────────────────┘
                      │
                      ▼
   ┌──────────────────────────────────────────────────────────┐
   │ autopay_transaction (per-attempt result)                 │
   │ src/entities/autopay_transaction.ts:14                   │
   │   AutopayTransactionStatus enum:                         │
   │     SCHEDULED · PROCESSING · SUCCESS · FAILED            │
   │     CANCELLED · MANUAL_REVIEW                            │
   └──────────────────────────────────────────────────────────┘
```

**Critical gates:**
- `AutoPayServiceV2.chargeScheduledDebits()` (line 223) is the cron driver. Hit by `routes/autopayV2.ts` daily.
- `processSchedule()` (line 253) flips PENDING → PROCESSING; SKIPPED on cancellation, manual review on threshold breach.
- `handlePaymentSuccess()` (line 576) and `.webhook()` (line 462) bridge from Cashfree back to invoice paid-status.
- `createNextSchedule()` (line 145) auto-pre-creates the next-month row.

**Implication for V2:** The Dues hero today silently includes invoices that are about to be auto-debited and invoices for which autopay just failed. Operator can't tell "₹X of my ₹Y dues is on autopay so I shouldn't worry" vs "₹Z autopay just failed and I MUST chase manually." Today they're one undifferentiated number.

### C. Eviction pipeline → auto-generated penalty/move-out invoices

```
   ┌──────────────────────────────────────────────────────────┐
   │ Operator marks eviction OR auto-trigger via threshold    │
   │ → TenantEvictionService.evictTenant() etc                │
   │   src/services/tenant/evictionService.ts:13              │
   └──────────────────┬───────────────────────────────────────┘
                      │
                      ▼
   ┌──────────────────────────────────────────────────────────┐
   │ tenant_eviction_details                                  │
   │ src/entities/tenantEvictionDetails.ts                    │
   │   date_of_eviction · raised_by · approved_by             │
   └──────────────────┬───────────────────────────────────────┘
                      │
                      ▼
   ┌──────────────────────────────────────────────────────────┐
   │ PropertyEvictionCharges (per-property rule)              │
   │ src/entities/propertyEvictionCharges.ts                  │
   │   option: 'penalty_charges' | 'move_out_charges'         │
   │   rule_name · total_deposit_enabled · rent_months ·      │
   │   fixed_amount                                           │
   └──────────────────┬───────────────────────────────────────┘
                      │
                      ▼
   ┌──────────────────────────────────────────────────────────┐
   │ tenant_eviction_invoices (auto-created invoices)         │
   │ src/entities/tenantEvictionInvoices.ts                   │
   │   1:1 → invoices.id                                      │
   │   These are the penalty/move-out rows that land in the   │
   │   same dues pool the V1 hero sums                        │
   └──────────────────────────────────────────────────────────┘
```

**Existing surface that already cites this:**
- Homepage `getFinancialsV2` queries `tenant_eviction_details` for "under-notice" count: `homepage/service.ts:838, 1626, 1651, 1728, 1753`. So the eviction concept IS visible elsewhere on the homepage, but **not on DA-01 V1**.
- Penalty / waiver math is computed live in `services/tenant/tenant.ts:6056, 7200, 7415` (`net_penalty_amount`).

**Implication for V2:** Penalty invoices and move-out charge invoices inflate the Dues hero. Operator has no way today to say "show me dues *excluding* eviction-driven penalties" or "how much of my dues is eviction-machinery output this period." Build Sheet HB6 widens `t.status IN (0,1,2)` to include moved-out, which mechanically pulls more eviction-driven invoices into the hero.

### D. Deposit adjustment against dues (silent settle-from-deposit)

Two distinct paths:

1. **Naive endpoint:** `POST /tenant/adjustDeposit` at `controllers/tenant.ts:520` — literally `Tenant.update(...).security_deposit = X`. **CSB-3 P0** — zero auth on a financial mutation. Doesn't touch invoices or write a payment row.

2. **Real path (used by Payment flow):** `PaymentService.advancedRecordPayment` at `services/payment/paymentService.ts:1150` accepts `deposit_adjustment_amount` (line 560), debits deposit, marks invoices paid, writes a Payment row mode-tagged. Reconciled with `getApplicableCredits` (line 1043) and `net_available_advance_amount` (line 2843).

**Implication for V2:** When a tenant moves out with ₹15K dues and ₹20K deposit, the operator does adjust-from-deposit. The dues line drops by ₹15K, deposit liability drops by ₹15K. V1 attributes the dues drop to "they paid." It was actually "deposit applied." Operator can't tell apart actual collection from deposit-as-payment. This also breaks the cross-screen story with DA-06 Liability.

### E. Invoice scheduler (forward-creates next-month rent)

`InvoiceScheduler` entity (`src/entities/invoice_scheduler.ts`):
```
status: 0 pending · 1 completed · -1 failed · 2 cancelled
scheduled_date · invoice_uuid · tenant_package_id · next_scheduled
```

`AutoPayServiceV2.createNextSchedule()` is one driver; package-based renewals fire others. The implication for DA-01: **future-dated invoices already exist in the `invoices` table with `due_date > today`**. HB8 acknowledges this — `applyDefaulters` lacks `due_date ≤ today` guard. Build Sheet's fix is a guard plus an empty-state. **But the more interesting V2 question is the opposite — "what's coming due in the next 7 days that isn't overdue yet."** Helpers `total_future_dues` in `helpers/invoices.ts:1385,1438` already proves the aggregate is cheap.

### F. TeamPassbookCollectionService — cross-entity flow

When a payment lands, `teamPassbookCollectionService.ts` (already documented in DA-02 V2) writes a `team_member_transactions` row with `category=COLLECTION`, `is_paid`, `payment_map.payment_uuid`. **For DA-01 the question is the inverse**: when staff records a cash collection, both `payments` and `invoice.paid_amount` mutate (via `applyPaymentToDuesAndInvoices` at `services/payment/paymentService.ts:2518`). The dues hero responds *immediately*. So the team_member_transactions ledger is the right source for the V2 question "**who collected** the dues that just disappeared from my screen this hour" — same source DA-02 uses, but mirrored direction.

---

## 2. Existing operator-facing surfaces V1 missed

| Surface | Where | What it shows | What V1 misses |
|---|---|---|---|
| **`/invoices/sendBulkPaymentReminder` + ForProperty** | `controllers/invoices.ts:6812, 7169` · routes `:231,:232` | Triggers bulk WA reminder run; respects automated-flag and last-sent-date | V1 has no "reminder activity log" view. Operator can't see "ran today / yesterday / never this month" or "how many got reminded." |
| **`Property.last_bulk_tenant_reminder_date`** | `entities/property.ts` | When the property last fired a bulk reminder | V1 doesn't show this on the Dues screen. Should be a chip near hero: "last reminded · 3 days ago." |
| **`Invoices.reminder_contact: string[]`** | `entities/invoices.ts:153` | Per-invoice append-only contact ledger | V1 doesn't expose "this bill has been reminded 3× and still unpaid" — exactly the chronic-defaulter signal HB3 was deferred for. |
| **Autopay V2 subscription registry** | `entities/autopay.ts` · `autopay_debit_schedule` · `autopay_transaction` | Who is on autopay, debit cadence, success/fail history | V1 zero visibility. Dues hero silently includes autopay-pending and autopay-failed amounts as one number. |
| **`autopay_debit_schedule.status IN (3 SKIPPED)`** | `entities/autopay_debit_schedule.ts:15` | This month's debit got skipped (subscription paused, validation failed, etc.) | Operator must chase manually — no signal on Dues screen that autopay just stopped for tenant X. |
| **`tenant_eviction_details.date_of_eviction`** | `entities/tenantEvictionDetails.ts` | Eviction notice raised / scheduled date | DA-01 V1 doesn't tag dues as "from a tenant under eviction notice." Already used at `homepage/service.ts:838,1626` for the under-notice count. |
| **`tenant_eviction_invoices` + `property_eviction_charges`** | entities/* | Auto-generated penalty + move-out invoices | These land in the Dues hero undifferentiated. Operator can't see "₹40K of my ₹2L dues is eviction-machinery penalty." |
| **`InvoiceScheduler` forward-creation** | `entities/invoice_scheduler.ts` | Next month's rent already in `invoices` with future `due_date` | V1 surfaces "Due Later" only in Today-mode urgency bar. No "what's coming due in next 7 days" chip — but this is exactly what Rajesh wants before a Friday review. |
| **`total_future_dues` aggregate** | `helpers/invoices.ts:1385,1438,1488` | Already-computed sum of future-dated dues | Built; not surfaced on DA-01 V1. Tile reuse, not a new query. |
| **Property toggle `automated_payment_reminders`** | `entities/property.ts` (referenced in `controllers/invoices.ts:6831`) | Is automated daily reminder cron on? | V1 doesn't show on the Dues screen. Should at minimum say "auto-reminders: OFF" if Rajesh disabled them — most common reason dues balloon silently. |

**Net:** V1 treats Dues as photograph of a stock. Codebase shows two live machines (reminder cron + autopay debit cron) and one event-driven machine (eviction) actively trying to drain the stock. V1 makes them invisible.

---

## 3. V2 question list — ranked by operator pain × codebase feasibility

### Must-add to V2 (P0 — operator pain confirmed by absent surfaces)

**Q10 — "Are my auto-reminders on, and when did they last run?"**
- WHY: most common silent failure — owner disables `automated_payment_reminders` during a test, forgets, dues balloon 3 weeks later. Today the screen looks identical whether auto-reminders run or not.
- WHAT: status chip near hero: "Auto-reminders ON · last ran 2 hrs ago" or "Auto-reminders OFF · enable" with deep-link to property settings. If `last_bulk_tenant_reminder_date > 7 days ago` and flag is ON → red badge.
- WHERE: `Property.automated_payment_reminders` boolean + `Property.last_bulk_tenant_reminder_date` timestamp. Both already present, no aggregation needed. Single property-row lookup.

**Q11 — "How many of my dues are on autopay (will auto-collect) vs manual (I must chase)?"**
- WHY: autopay-covered invoices are *not the same operator problem* as manual ones. Mixing them inflates the chase list. Roomsoom and others actively sell autopay — operator wants the split visible.
- WHAT: split chip beneath hero: "₹X on autopay (Y bills) · ₹Z manual (W bills)." Drill on autopay → list with last-attempt-status; drill on manual → ordinary worklist.
- WHERE: LEFT JOIN `invoices` → `autopay_debit_schedule_invoice` → `autopay_debit_schedule` filtering `status IN (PENDING, PROCESSING, INTERIM)` AND parent autopay `status='active'`. Anything else = manual. Add `autopay_transaction.status = FAILED` carve-out → see Q12.

**Q12 — "Autopay debits that failed this period — needs my action."**
- WHY: autopay failure is a *new* kind of urgency invisible today. Today these dues sit in the same overdue bucket as never-reminded ones. Cashfree mandate revoked, insufficient funds, validation reject — all need different follow-up than a normal reminder.
- WHAT: red chip "N autopay debits failed · ₹X stuck" → drill to list with failure reason from `autopay_transaction.failure_reason`. CTA: "Convert to manual / Retry / Notify tenant."
- WHERE: `autopay_transaction.status = 'failed'` joined to active `autopay_debit_schedule.status = PENDING` rows whose `last_attempt_at >= period_start`. Reason from `failure_reason` column.

### Should-add to V2 (P1 — clear value, smaller pain)

**Q13 — "What's coming due in the next 7 days?" (forecast — different from overdue)**
- WHY: V1 Urgency Bar in Today mode shows "Due This Week" already — but only for already-due, and only in today-mode. V1 doesn't answer "what's the chase load this Friday?" The helper `total_future_dues` already aggregates this for the homepage. Cheap to reuse.
- WHAT: "Upcoming · next 7 days · ₹X across Y bills" chip beneath hero. Range-mode safe. Forecast-mode safe.
- WHERE: same hero universe + `due_date BETWEEN today+1 AND today+7`. Existing index `invoices_due_date_idx` covers it (`entities/invoices.ts:20`).

**Q14 — "How much of my dues was settled from deposit this period vs actually paid?"**
- WHY: deposit-as-payment masks the real-collection picture. Solo-owner Rajesh closes month thinking ₹2L collected; really ₹1.6L was cash and ₹40K was just deposit ledger entry. Also breaks DA-06 Liability story.
- WHAT: small chip on Period Performance accordion: "Settled from deposit: ₹X across Y move-outs."
- WHERE: `payments` joined where `mode=deposit_adjustment` OR via `paymentService.advancedRecordPayment` path with `deposit_adjustment_amount > 0`. Need a discriminator column or payment_mode code review with Jatin. Worth deferring if discriminator unclear.

**Q15 — "How much of my dues is eviction-machinery (penalty + move-out charges) vs rent?"**
- WHY: when HB6 widens to include moved-out tenants, eviction-driven invoices balloon into the hero. Owner sees "dues +30% MoM" — believes it's chase-failure, it's actually one eviction. Wrong story → wrong action.
- WHAT: bill-type split (already V1 stretch Q5) augmented with "Eviction penalty" and "Move-out charge" rows. Or a separate chip "Of which eviction-machinery: ₹X."
- WHERE: join `invoices.id → tenant_eviction_invoices.invoice_id`. Anything in that table is auto-eviction-generated.

### Defer to V3 (P2 — cost > benefit)

**Q16 — "Chronic-defaulter list" (HB3 deferred to V2.0 in Build Sheet).**
- Pain real but `Invoices.reminder_contact[]` array is awkward to query, no count column. Needs a real `reminders_sent_count` migration or activity-log table (HB4). Wait for HB4 to land.

**Q17 — "Late-fee accrued this period."**
- No standalone `late_fee` ledger today. `PropertyEvictionCharges.penalty_charges` rule fires at eviction; not a continuous accrual. Operator pain low — most properties don't actually charge daily late fees. Skip unless field interviews surface demand.

---

## 4. V1 → V2 architecture risks (decide before V2 lands)

1. **No `reminders_sent_count` column on Invoices.** HB4 calls for activity-log wire; until it exists, "reminded N times" requires `len(reminder_contact[])` which is per-invoice + can't be aggregated cheaply. **Decision:** does HB4 land in V1 or V2? Q10/chronic-defaulter Q16 both depend on it.

2. **Autopay-status read model is non-trivial.** Q11/Q12 need a 3-table join (invoice → autopay_debit_schedule_invoice → autopay_debit_schedule → autopay_transaction) for every dues query. Either add a denormalized `invoices.autopay_status` column (cleaner, write-amplification on autopay flips) OR build a materialized view. **Decision:** decide before V2 starts; affects whether DA-01 V2 endpoint extends `getDuesWidget` or spawns a new endpoint.

3. **Deposit-adjustment payment_mode code is undocumented.** Q14 needs a clean filter for "payment came from deposit." `paymentService.advancedRecordPayment` writes a Payment row but the `payment_mode` discriminator for deposit-adjustment isn't named in the codebase audit. **Decision needed from Jatin:** which `payment_mode` codes flag deposit-adjustment? If none, add one (migration) before V2.

4. **Eviction-invoice tagging.** Q15 needs a fast "is this invoice eviction-generated" check. `tenant_eviction_invoices.invoice_id` is the FK back, but querying "of N dues invoices, which are in tenant_eviction_invoices" is a LEFT JOIN. Acceptable performance — `tenant_eviction_invoices` table is small. No migration needed.

5. **`AutoPayServiceV2` is rent-only.** Hardcoded check at `services/autoPay/autopayV2.ts:81` — `due_type_id` must be Rent. Q11's "% on autopay" is meaningful only for Rent invoices. **Decision:** scope Q11 to Rent-only or label as "of Rent dues, X% on autopay." Recommend the latter — accurate, future-proof.

6. **Permission story is untouched.** Today `getDuesWidget` uses `viewInvoices` + `view_invoices_of_self_added_tenants` fallback (`service.ts:130-138`). Autopay data has no separate permission key — anyone who can see dues can see autopay status. Likely fine but flag for Jatin.

7. **Future-period guard (HB8) interacts with Q13.** HB8 adds `due_date ≤ today` guard to `applyDefaulters`. Q13 explicitly inverts this. Make sure the guard is per-component, not global, else Q13 returns empty.

---

## 5. Recommended V2 scope for DA-01 Brief

Add a new section **"V2 additions — Reminder cadence, Autopay coverage, Eviction tagging"** with 3 questions:

| # | Operator question | Eng effort | Source |
|---|---|---|---|
| Q10 | "Are auto-reminders on, last run when?" | S — single property-row read. UI chip near hero. | `Property.automated_payment_reminders` + `last_bulk_tenant_reminder_date` |
| Q11 | "How much of my dues is on autopay vs manual?" | M — new 3-table join. Decide: denormalized `invoices.autopay_status` column vs live join | `autopay_debit_schedule_invoice` × `autopay_debit_schedule.status` × `autopay.status` |
| Q12 | "Autopay debits failed this period — chase manually." | S-M — `autopay_transaction.status=failed` filter + reason text. Drill into worklist filtered to failed-autopay subset | `autopay_transaction.status` + `failure_reason` |

**Cut from V2 (defer to V3):** chronic-defaulter (Q16, blocked on HB4), late-fee accrued (Q17, no operator pain confirmed).

**Stretch P1 if eng capacity allows:** Q13 (next-7-days forecast — cheap reuse of `total_future_dues`), Q15 (eviction-machinery tagging — single LEFT JOIN).

**Pre-build decision for Jatin (must close before V2 starts):**
1. Does HB4 (reminder activity log) land in V1 or V2? Blocks Q10's "auto-reminders OFF for N days" red-state and entirely blocks Q16 (chronic defaulter).
2. Denormalized `invoices.autopay_status` column vs live 3-table join? Pick one before any UI ticket. Recommend denormalized — Q11 will be the hot read path.
3. Deposit-adjustment payment_mode code — does one exist or add one? Blocks Q14 cleanly.

---

## 6. Verdict

- **Did V1 of DA-01 have the same blind spot pattern as DA-02/03/04?** Yes, with high confidence. V0.10 treats Dues as a stock to *display*; codebase has three live flow systems (reminder cron, autopay V2, eviction pipeline) and two ledger systems (deposit-adjust, invoice-scheduler) that *change* that stock without the operator seeing why. Same pattern as DA-02's settlement lifecycle blindness, DA-03's refund-lifecycle blindness, DA-04's reimbursement blindness.

- **How serious is the gap (compared to the other 3)?** Equally serious or worse. DA-01 is the highest-traffic Dues screen — Rajesh opens it 2-3×/week, Priya lives in it. The autopay + reminder gap means V1 ships a screen where the operator can't tell why dues went up or down. The eviction-tagging gap means a single eviction skews the MoM chip red and the operator chases the wrong tenants.

- **Is DA-01 V0.10 + V2 pointer the right path, or does V1 need a rewrite?** V0.10 + V2 pointer is right. V1's must-ships (Q1 hero + Q2 urgency split) are independently load-bearing and don't require the V2 dimensions to be useful. Add one paragraph to the V0.10 Brief acknowledging "V2 will add: reminder-cadence visibility, autopay split, eviction tagging — see DA-01 V2 Dependency Map." Do not block V1. Do, however, treat Q10 (auto-reminder status chip) as a candidate for V1.1 fast-follow since it's a single property-row read with zero new joins — could land within the 6-week cycle if Q5 or Q4 get cut.

---

## 7. Cross-doc traceability

- V1 Brief: `[[DA-01 Brief]]` (V0.10 — ships with the gap acknowledged in a "V2 scope deferred" paragraph)
- Build Sheet: `[[DA-01 Build Sheet]]` (V2 locked at engineering scope; does NOT include reminder/autopay/eviction layers)
- Sibling V2 maps: `[[DA-02 V2 Dependency Map]]` · `[[DA-03 V2 Dependency Map]]` · `[[DA-04 V2 Dependency Map]]`
- Related blockers: HB3 (chronic-defaulter, deferred), HB4 (reminder activity log), HB8 (future-period guard)

---

## Changelog

| Date | Change | By |
|------|--------|-----|
| 2026-05-16 | Initial V2 scope note — generated from deep codebase exploration sub-agent. Confirmed three live flow systems invisible to V0.10: reminder cadence engine (sendBulkPaymentReminder + Property.last_bulk_tenant_reminder_date + reminder_contact[]); Autopay V2 subscription engine (AutoPayServiceV2 + autopay_debit_schedule + autopay_transaction); eviction pipeline auto-generating invoices (TenantEvictionService + PropertyEvictionCharges + tenant_eviction_invoices). Two secondary deps: deposit-adjust silent-payment path, InvoiceScheduler forward-creation. Same blind-spot pattern as DA-02/03/04. Recommended V2 P0: Q10 reminder-status chip, Q11 autopay split, Q12 autopay failure list. | Sanchay (PM) + Claude sub-agent |
