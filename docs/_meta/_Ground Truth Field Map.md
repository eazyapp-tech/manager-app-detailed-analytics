---
title: DA Suite — Ground Truth Field Map
date: '2026-05-08'
tags:
  - rentok
  - prd
  - build-sheet
  - engineering
  - field-map
  - ground-truth
status: V1 — locked
for: Build Sheet generation (PMs + engineers)
companion_to: All 7 DA spec PRDs and forthcoming Build Sheets
---

# DA Suite — Ground Truth Field Map

> **Purpose:** Stop hallucinating field names, formulas, route paths, and permission keys when writing Build Sheets for the 7 DA specs (DA-01 Dues, DA-02 Collections, DA-03 Refunds, DA-04 Expenses, DA-05 Discounts, DA-06 Liabilities, DA-07 Cash Flow). **Every fact in this doc was verified against the codebase on 2026-05-08, post-master-merge.**
>
> **How to use:** Read this BEFORE writing any cell of a Build Sheet. If a fact you need isn't here, verify it in the codebase yourself and add it here. Treat this as the single source of truth for entity columns, status enums, route paths, permission keys, and production aggregation formulas.
>
> **When to update:** Whenever the codebase changes (new column, new enum value, new endpoint, renamed permission), update the relevant section. Each section has a "verified at" header so we can spot stale facts.
>
> **Convention:** All file:line citations are relative to `/Users/eazypg/rentok-backend/`. SQL snippets are copied verbatim from production code, not paraphrased. Status tags follow Build Sheet convention (`EXISTS / EXTEND / BUILD / COMPOSE / VERIFY`).

---

## Section 1 — Core Entities

> **Verified at:** 2026-05-08 against `master` (commit `728abfd68`).

### 1.1 Invoices (`src/entities/invoices.ts`)

Table: `invoices`. Backbone of DA-01 Dues and DA-02 Collections.

| Column | Type | Default / nullable | Decoded enum / comment | DA spec(s) using |
|--------|------|--------------------|--------------------|-------------------|
| `id` | uuid (PK) | generated | — | All |
| `is_active` | smallint | default `1` | `1` = active row, `0` = soft-deleted/voided. Always filter for `is_active=1` | DA-01, DA-02 |
| `payer` | string | nullable | tenant identifier (firebase_id usually) — see also `firebase_id` column | DA-01, DA-02 |
| `firebase_id` | string | nullable, indexed | tenant firebase identifier — same as `payer` in most rows but not guaranteed | DA-01, DA-02 |
| `property` | string | nullable | property identifier `${pgID}PG${PGNumber}` | All |
| `property_fk_id` | uuid | required | FK to Property | All |
| `paid_by_name` / `paid_by_number` / `paid_by_room` / `paid_by_relation` | string | nullable | who actually paid (may differ from tenant) | DA-02 |
| `tax` | number | default `0` | invoice-level tax | DA-01 |
| `added_by` | number | required | `0`=owner, `1`=rent manager, `2`=partner, `3`=customer, `4`=tenant, `5`=others. **Note:** value `5` ("others") may include RentOk-system auto-generated bills — verify per query | DA-01 |
| `discount` | number | default `0` | discount applied at creation | DA-01, DA-05 |
| `amount` | decimal | nullable | invoice headline amount | All |
| `net_amount` | decimal | nullable | net after tax/discount; semantic differs from Payments.net_amount | DA-01 |
| `paid_amount` | decimal | default `0` | **⚠️ NOT populated for partial-paid invoices in production** (the partial-paid branch at `payments.ts:2039-2048` mutates `i.amount` in place instead — see §4.1 "Partial-paid pattern") | DA-01 |
| `status` | number | default `0`, indexed (`invoices_status_index`) | **`0`=Not Paid, `1`=Paid, `2`=Partially Paid (NEVER SET in production — see §4.1), `3`=Refunded, `4`=Loss** (verbatim comment line 93). **For "outstanding" use `status = 0` only** — production NEVER writes `status = 2` (verified 2026-05-11; only commented-out references at `payment.ts:5358, 5517`). `SUM(i.amount) WHERE status = 0` already captures partial outstanding correctly because partial payments destructively shrink `i.amount` in place | All |
| `due_date` | timestamp | nullable | when invoice is due | DA-01 |
| `paid_date` | timestamp | nullable, indexed (`invoices_paid_date_index`) | when invoice was fully paid | DA-01, DA-02 |
| `invoice_date` | timestamp | default `CURRENT_TIMESTAMP`, nullable, indexed | when invoice was issued | DA-01 |
| `due_type` | string | nullable, indexed (`invoices_due_type_index`) | **String values, NOT integer enum.** Examples: `'Rent'`, `'Security Deposit'`, `'Caution Money'`, `'Advance'`, `'Electricity'`, `'Maintenance'`. See constants `DEPOSIT_DUE_TYPES` and `ADVANCE_DUE_TYE` (Section 2.2) | DA-01, DA-04, DA-06 |
| `description` / `automated_description` / `paid_description` | string / text | nullable | human-readable | DA-01 |
| `due_start_date` / `due_end_date` | timestamp | nullable | for billing period | DA-01 |
| `gst_percentage` | numeric(5,2) | nullable | GST rate applied | DA-01 |
| `partner_name` / `partner_id` / `partner_phone` | string | nullable | external party | DA-01 |
| `pdf_link` / `payment_link` / `tax_invoice_id` | string | nullable | document references | DA-01 |
| `reminder_contact` | text[] | nullable | who got reminders | DA-01 |
| `notif_sent` | number | default `0` | reminder send flag | DA-01 |
| `created_at` | timestamp | default `CURRENT_TIMESTAMP`, indexed (`created_at_index`) | row creation | All |
| `root_invoice_id` / `parent_invoice_id` | uuid | nullable | for hierarchical invoices (recurring chains) | DA-01 |

Relations:
- `payments`: many-to-many via `payments_invoices` join table
- `refunds`: one-to-many via `Refunds.invoice`

**Common pitfalls on Invoices:**
- ❌ `i.amount > 0` is right but doesn't filter loss/refunded — combine with `status` constraints
- ❌ Spec writers ASSUMING `status IN (0, 2)` is needed for "outstanding" — **WRONG.** Production NEVER sets `status = 2` (verified 2026-05-11). Partial-paid is handled by destructively mutating `i.amount` in place at `payment.ts:2039-2048`. `status = 0` only is the correct production semantic. Adding `status IN (0, 2)` is pointless (no rows match) and could DOUBLE-COUNT if engineering ever fixes the partial-paid pattern in future. See §4.1 "Partial-paid pattern" for full details
- ❌ Spec writers ASSUMING `SUM(i.amount - i.paid_amount)` for "outstanding" — **WRONG.** `paid_amount` is never populated on partial-paid rows in production. `SUM(i.amount)` is correct because `i.amount` has already been shrunk to the outstanding balance by the mutation pattern
- ❌ `paid_date` is nullable; queries filtering `WHERE paid_date BETWEEN x AND y` will silently exclude unpaid invoices
- ❌ `amount` for a partially-paid invoice has been destructively mutated (original billed amount LOST from the row). To reconstruct original billed amount, JOIN payments_invoices and sum payment_amounts back. Data-modeling concern — long-standing issue, flag to Jatin/Nimit

---

### 1.2 Payments (`src/entities/payments.ts`)

Table: `payments`. Backbone of DA-02 Collections.

| Column | Type | Default / nullable | Decoded enum / comment | DA spec(s) using |
|--------|------|--------------------|--------------------|-------------------|
| `id` | uuid (PK) | generated | — | DA-02 |
| `is_active` | smallint | default `1` | `1` = active, `0` = voided. Combine with `status=1` for "successful" payments | DA-02 |
| `property` | string | required | property uuid | DA-02 |
| `property_fk_id` | uuid | required | FK to Property | DA-02 |
| `credits_used` | decimal | default `0`, nullable | credits applied during this payment | DA-02, DA-05 |
| `advance_amount` | decimal | default `0`, nullable | advance portion of this payment | DA-02, DA-06 |
| `net_amount` | decimal | nullable | the per-payment amount used for sums (after gateway charges in default branch) | DA-02 |
| `gateway_charges` | decimal | nullable | charge to subtract from net_amount in collections sum | DA-02 |
| `external_gateway_charges` | decimal | nullable | (separate field) | DA-02 |
| `status` | number | required | **`0`=failed, `1`=success, `2`=pending, `3`=refunded** (verbatim comment line 64). Filter `status=1` for "Net Collected" | DA-02 |
| `payment_mode` | number | required | **NOT just 0-cash/1-paytm/2-razorpay (entity comment is stale).** Real values include: `202, 2041, 2040, 207` (cash variants per `CASH_PAYMENT_MODES`); `211` (deposit-applied — excluded from grand total); `213` (special — subtracts `payment_owner_credits`); `288` (advance-applied — excluded from grand total) | DA-02 |
| `eazypg_credits` / `owner_credits` | number | default `0` | for mode 213 special handling | DA-02 |
| `credit_keys` | text[] | nullable | credit keys used | DA-02, DA-05 |
| `received_by` | string | nullable | team member who recorded | DA-02 |
| `partner_id` / `partner_name` / `partner_phone` | string | nullable | external | DA-02 |
| `description` | string | nullable | — | DA-02 |
| `bank_reference_no` | string | nullable | external bank ref | DA-02 |
| `gateway_payment_method` | jsonb | nullable | gateway-specific metadata | DA-02 |
| `cf_payment_id` | string | nullable | Cashfree-specific | DA-02 |
| `payment_id` | string | nullable | external payment id | DA-02 |
| `payment_gateway` | string | nullable | which gateway | DA-02 |
| `firebase_id` | string | nullable | tenant firebase | DA-02 |
| `created_at` | timestamp | default `CURRENT_TIMESTAMP` | row creation | DA-02 |
| `paid_date` | timestamp | nullable | **when payment was actually paid** — date basis for collections aggregation | DA-02 |
| `settlement_id` | string | nullable | FK to Settlements (one-to-one) | DA-02 |
| `payout_id` | string | nullable | FK to Payout | DA-02 |
| `non_settlement_amount` | decimal | nullable | amount staying with RentOk | DA-02 |
| `non_settlement_amount_type` | smallint | nullable | `2`=owner bears, `1`=tenant bears | DA-02 |
| `slack_ts` | string | nullable | internal logging | — |

Relations:
- `invoices`: many-to-many via `payments_invoices`
- `credits`: one-to-many (Credits.payment)
- `settlement`: one-to-one (Settlements)
- `advance_created` / `advance_redeemed`: one-to-one (AdvancePayments)
- `payout`: many-to-one
- `wallet_entry`: one-to-one (Wallet)
- `attachments` / `meta_details`: child tables

**Common pitfalls on Payments:**
- ❌ There is no column called `payment_date` — use `paid_date`
- ❌ There is no column called `amount` — use `net_amount` (or `net_amount - gateway_charges` for collections sum)
- ❌ `payment_mode` entity comment "0-cash, 1-paytm, 2-razorpay" is **stale**. Real mode values are 3-digit + special. See Section 2.1
- ❌ Refund subtraction is NOT in current grand-total expression — confirmed gap (CSB note for DA-02)

---

### 1.3 Refunds (`src/entities/refunds.ts`)

Table: `refunds`. Backbone of DA-03 Refunds.

| Column | Type | Default / nullable | Comment | DA spec(s) using |
|--------|------|--------------------|------|--------|
| `id` | uuid (PK) | generated | — | DA-03 |
| `amount` | number | required | refund amount | DA-03, DA-06 |
| `refund_date` | timestamp | default `CURRENT_TIMESTAMP`, required | **the date basis for refund aggregation** | DA-03 |
| `refund_reason` | string | nullable | free-text or category | DA-03 |
| `refund_mode` | number | required | refund mode integer (similar enum space to payment_mode — verify per query) | DA-03 |
| `refunded_by` | string | nullable | team member id | DA-03 |
| `refunded_by_name` | string | nullable | display name | DA-03 |
| `refunded_by_phone` | string | nullable | partner phone if applicable | DA-03 |
| `refund_images` | json | nullable | array of image URLs | DA-03 |
| `invoice_id` | uuid | required | FK to Invoices | DA-03, DA-06 |

Relations:
- `invoice`: many-to-one to Invoices

**Common pitfalls on Refunds:**
- ❌ NO `is_active` column — every row is real. No soft-delete pattern
- ❌ NO `status` column — refunds are recorded as final
- ❌ NO `tenant_id` direct — derive via `invoice → invoice.payer` or `invoice.firebase_id`

---

### 1.4 Credits (`src/entities/credits.ts`)

Table: `credits`. Backbone of DA-05 Discounts. **In this schema, "discounts" are stored as `Credits` rows.**

| Column | Type | Default / nullable | Decoded enum / comment | DA spec(s) using |
|--------|------|--------------------|--------------------|-------------------|
| `id` | uuid (PK) | generated | — | DA-05 |
| `property` | string | required | property uuid | DA-05 |
| `tenant` | string | nullable | tenant firebase_id | DA-05, DA-06 |
| `firebase_id` | string | nullable | also tenant firebase | DA-05 |
| `initiated_by` | string | nullable | who issued the credit | DA-05 |
| `date_added` | timestamp | nullable | **when the credit was issued** | DA-05 |
| `date_used` | timestamp | nullable | **when the credit was applied to an invoice** | DA-05 |
| `expiry_date` | timestamp | nullable | when the credit expires | DA-05, DA-06 |
| `is_minimum_full_amount` | number | default `0`, nullable | **`0`=pay full to use, `1`=pay minimum to use** (verbatim comment) | DA-05 |
| `minimum_amount` | decimal | nullable | the threshold for is_minimum_full_amount=1 | DA-05 |
| `description` | string | nullable | free-text | DA-05 |
| `source` | number | nullable | **`0`=owner, `1`=eazypg** (verbatim comment — note "eazypg" is old branding for RentOk) | DA-05 |
| `status` | number | nullable | **`0`=unused, `1`=used** (verbatim comment) | DA-05, DA-06 |
| `view_status` | number | nullable | for scratched/unscratched UX. **Production filter at `paymentService.ts:getApplicableCredits` uses `view_status = 101`** for "applicable" credits | DA-05, DA-06 |
| `amount` | number (int by TypeORM default — no `type:'decimal'` declared) | required | credit amount. **⚠️ Unit ambiguity (DA-05 HB5 — VERIFY):** entity declares as plain `number` (likely `int`). Whether producer code writes rupees or paise here needs verification — **diverges from `Payments.net_amount` which is `decimal` (paise convention)**. If a Build Sheet sums Credits.amount alongside payment amounts, unit reconciliation is mandatory | DA-05, DA-06 |
| `payment_id` | string | nullable | FK to Payments (when redeemed) | DA-05 |
| `created_at` | timestamp | default `CURRENT_TIMESTAMP` | row creation | DA-05 |

Relations:
- `payment`: many-to-one to Payments

**Common pitfalls on Credits:**
- ❌ "Discount" terminology in PRDs maps to `credits` table, not a separate `discounts` table
- ❌ `source` of `1` says "eazypg" in comment — same as RentOk-funded
- ❌ "Used credits in period" filter: `status=1 AND date_used IN [period]`. NOT `status=1 AND date_added IN [period]` (that's "issued in period")
- ❌ `expiry_date` is nullable — handle null as "never expires"

---

### 1.5 Tenant (`src/entities/tenant.ts`)

Table: `tenant`. Backbone of DA-01 (population breakdown), DA-06 (Move-Out Forecast).

| Column | Type | Default / nullable | Decoded enum / comment | DA spec(s) using |
|--------|------|--------------------|--------------------|-------------------|
| `id` | uuid (PK) | generated | — | All |
| `firebase_id` | string | nullable | tenant firebase identifier — primary tenant key in most queries | All |
| `name` / `email` / `phone` | string | nullable | identity | All |
| `status` | number | required | **`0`=old/checked-out, `1`=active, `2`=booking, `8`=interested-booking-lead** (verified production usage at `homepage/service.ts:798-803` and `tenant/interestedBooking.ts:92`). **Note: comment is missing on entity, derived from production code.** Other status values may exist | All |
| `property_id` | uuid | required | FK to Property | All |
| `security_deposit` | decimal | nullable | deposit amount per agreement (denormalized; actual held amount comes from invoice ledger — see Section 4) | DA-01, DA-06 |
| `under_notice` | boolean | nullable | **EXISTS but NOT used by production homepage.** Production uses `date_of_eviction IS NOT NULL` instead. Real divergence — both fields exist; not guaranteed to be in sync | DA-06 |
| `date_of_joining` | date | nullable | when tenant joined | All |
| `date_of_eviction` | date | nullable | **expected/effective move-out date** (canonical move-out timing field per production) | DA-06 |
| `notice_raised_on` | date | nullable | when notice was raised | DA-06 |
| `notice_period` | number | required, default `30` | notice period in days | DA-06 |
| `checkout_date` | date | nullable | actual date of checkout (post-move-out) | DA-06 |
| `checkout_time` | string | nullable | time of checkout | DA-06 |
| `last_agreement_renewal_date` | date | nullable | — | DA-06 |
| `room` | string | nullable | current room reference | DA-01 |
| `room_type` | string | nullable | — | DA-01 |
| `rent_amount` | decimal | nullable | current rent | DA-01 |
| `assigned_staff` / `staff_id` / `staff_name` / `staff_phone` | string | nullable | — | DA-01, DA-04 |
| `lead_status` / `lead_token_amount` / `lead_source` / `lead_remarks` | string | nullable | for booking pipeline | DA-01 |
| `lead_visit_date` / `lead_visit_time` / `lead_visit_type` | date / string | nullable | — | DA-01 |
| `reason_of_eviction` | string | nullable | — | DA-06 |
| `is_profile_locked` | boolean | default `false` | — | — |
| `collect_online_payment` | boolean | default `true` | — | DA-02 |
| `grace_period` | int | nullable | per-tenant grace days | DA-01 |
| `added_by_id` | uuid | nullable | team member who added | All |
| `checkin_seconds` | number | nullable | — | DA-01 |
| `checkin_timestamp` | date | nullable | — | DA-01 |
| `is_aadhar_verified` | boolean | default `false` | — | — |
| Many KYC / address / family fields (aadhar, parents, addresses, etc.) | various | mostly nullable | not relevant to DA suite | — |

**Common pitfalls on Tenant:**
- ❌ NO `exit_date` field — use `date_of_eviction`
- ❌ NO `held_amount` field — computed via Section 4.6
- ❌ NO `has_held_amount` field — same, derived
- ❌ `under_notice: boolean` exists but production homepage uses `date_of_eviction IS NOT NULL` instead. **Real divergence — DA-06 must use the production pattern (`date_of_eviction IS NOT NULL`) to avoid mismatch with homepage tile counts**
- ❌ `status` enum values are not exhaustive at 0/1/2 — `8` exists for "interested-booking-lead"

---

### 1.6 AdvancePayments (`src/entities/advancePayments.ts`)

Table: `advance` *(NOTE: table name is `advance`, NOT `advance_payments`)*. Used by DA-06 held-amount calculation.

| Column | Type | Default / nullable | Comment |
|--------|------|--------------------|---------|
| `id` | uuid (PK) | generated | — |
| `tenant` | string | nullable | tenant firebase_id |
| `property` | relation | required | FK to Property via `property_id` |
| `amount` | decimal | nullable | advance amount |
| `payment_inflow` | one-to-one | — | FK to Payments (when advance recorded) via `advance_payment_id` |
| `payment_outflow` | one-to-one | — | FK to Payments (when advance redeemed) |
| `created_at` | timestamp | default `CURRENT_TIMESTAMP` | row creation |

**Note:** in production logic (`paymentService.ts:getAvailableAdvance`), advance availability is computed from **Invoices** with `due_type = 'Advance'` AND `status = 1`, NOT directly from this table. The `AdvancePayments` table is more of a pointer/log between Payment rows; the canonical "available advance per tenant" is invoice-driven. See Section 4.6.

---

### 1.7 Expenses (`src/entities/expenses.ts`)

Table: `expenses`. Backbone of DA-04 Expenses.

| Column | Type | Default / nullable | Decoded enum / comment |
|--------|------|--------------------|--------------------|
| `id` | uuid (PK) | generated | — |
| `payer` | string | nullable | tenant identifier (for tenant-charge expenses) |
| `firebase_id` | string | nullable | tenant firebase |
| `property` | string | nullable, indexed | property uuid |
| `property_fk_id` | uuid | required | FK to Property |
| `added_by` | number | required | **`0`=owner, `1`=rent manager, `2`=partner, `3`=customer, `4`=tenant, `5`=others** (verbatim comment line 40) |
| `amount` | decimal | nullable | expense amount |
| `paid_date` | timestamp | nullable | when expense was paid |
| `invoice_date` | timestamp | default `CURRENT_TIMESTAMP`, nullable | when expense entry was created |
| `expense_type` | string | nullable | category like 'Maintenance', 'Utility', etc. |
| `description` / `automated_description` | string | nullable | — |
| `payment_mode` | number | nullable | mode (similar enum space to Payments.payment_mode) |
| `partner_name` / `partner_id` / `partner_phone` | string | nullable | external party |
| `status` | number | nullable | **comment says `//TODO-REMOVE STATUS`** — status field is being deprecated; verify usage per query |
| `paid_to` | string | nullable | — |
| `bill_urls` | text[] | nullable | array of bill image URLs |
| `wallet_id` | uuid | nullable | FK to Wallet (when paid from wallet) |
| `is_active` | smallint | default `1` | active flag |
| `created_at` | timestamp | default `CURRENT_TIMESTAMP` | — |

Relations:
- `complaint_expenses_map`: one-to-many (ComplaintExpenseMap) — links expenses to complaints
- `wallet`: one-to-one (Wallet)

**Common pitfalls on Expenses:**
- ❌ `status` field exists but marked TODO-REMOVE — don't rely on it
- ❌ Date basis: `paid_date` is the "when expensed" basis; `invoice_date` is row creation
- ❌ `payment_mode` may overlap with Payments' enum but is independent
- ❌ **`is_capex` column does NOT exist** (verified 2026-05-08 via grep across all entity files, list_screens/expenses, services/expense). DA-04 and DA-07 PRDs reference `is_capex=1` for capex flag callouts and bottom sheets. **NEW BUILD requirement** — must add column AND heuristic for classifying expenses as capex (e.g., one-off > ₹X threshold + manual flag). Affects: DA-04 §"Section A Footer Callouts" capex flag, DA-07 Section A footer "capex flag" cross-screen drill

---

### 1.8 TenantChecklist + TenantChecklistItem (`src/entities/tenantChecklist.ts`, `tenantChecklistItem.ts`)

Used by DA-06 Move-Out Forecast (settlement-readiness flag).

#### TenantChecklist (table: `tenant_checklist`) — **PARENT entity**

| Column | Type | Default / nullable | Comment |
|--------|------|--------------------|---------|
| `id` | integer (PK) | auto-increment | — |
| `tenant_id` | uuid | required, indexed | FK to Tenant |
| `property_id` | uuid | required, indexed | FK to Property |
| `room_id` | uuid | required, indexed | FK to Room |
| `checklist_type` | varchar(20) | required, indexed | type identifier |
| `status` | smallint | default `0` (MOVE_IN_NOT_STARTED), indexed | **8-value enum** — see TenantChecklistStatus below |
| `total_items` | integer | default `0` | total items in checklist |
| `completed_items` | integer | default `0` | how many done |
| `not_working_items` | integer | default `0` | how many flagged not-working |
| `started_at` | timestamp | nullable | when started |
| `completed_at` | timestamp | nullable | when completed |
| `deadline` | timestamp | nullable | when due |

**TenantChecklistStatus enum** (verbatim from `src/entities/tenantChecklist.ts:19-28`):
- `0` = `MOVE_IN_NOT_STARTED` — tenant has not marked anything
- `1` = `MOVE_IN_TENANT_COMPLETED` — tenant marked all items
- `2` = `MOVE_IN_LOCKED` — move-in locked, move-out pending
- `3` = `MOVE_IN_REOPENED` — admin resolved issues, tenant can re-mark
- `4` = `MOVE_OUT_TENANT_COMPLETED` — tenant marked all move-out items
- `5` = `MOVE_OUT_TEAM_COMPLETED` — team submitted move-out
- `6` = `MOVE_OUT_APPROVED` — admin approved, no further changes
- `7` = `NEXT_MOVE_IN_READY` — next move-in can begin

**For DA-06 settlement-readiness flag:**
- ✅ render when `tenant_checklist.status = 6` (MOVE_OUT_APPROVED) for the tenant's active checklist
- ⏳ render when `status < 6`
- Per-tenant lookup: `tenant_checklist.tenant_id = t.id`. There may be multiple checklist rows per tenant over time (one per stay) — use most recent active one

#### TenantChecklistItem (table: `tenant_checklist_item`) — child rows

| Column | Type | Default / nullable | Comment |
|--------|------|--------------------|---------|
| `id` | integer (PK) | auto-increment | — |
| `checklist_id` | integer | required | FK to TenantChecklist |
| `facility_name` | varchar(255) | required | inventory item name |
| `instance_number` | integer | default `1` | per-item instance |
| `damage_cost` | integer | default `0` | charge for damage |
| `category` | varchar(255) | nullable | item category |
| `description` | text | nullable | — |
| `move_in_condition` | enum | nullable | `'working' \| 'damaged' \| 'missing' \| 'pending'` |
| `move_in_notes` | text | nullable | — |
| `move_in_media` | text[] | nullable | photos at move-in |
| `move_in_marked_at` | timestamptz | nullable | when item was checked at move-in |
| `move_out_condition` | enum | nullable | same enum as move_in |
| `move_out_notes` | text | nullable | — |
| `move_out_media` | text[] | nullable | photos at move-out |
| `move_out_marked_at` | timestamptz | nullable | when item was checked at move-out |
| `inventory_id` | integer | nullable | FK to Inventory |
| `created_at` / `updated_at` | timestamptz | default | — |

**Common pitfalls:**
- ❌ NO `status` column on `tenant_checklist_item` — items don't have a separate status. The PARENT `tenant_checklist.status` is the canonical state machine
- ❌ Don't use item-level `move_out_marked_at IS NOT NULL` for settlement-readiness — use parent `tenant_checklist.status >= 6` (MOVE_OUT_APPROVED) instead. Item-level timestamps are for granular UI, not the readiness flag

---

## Section 2 — Constants and Enums

> **Verified at:** 2026-05-08 against `src/services/payment/constants.ts` and entity comments.

### 2.1 Payment Mode integer values (FULL canonical enum)

Sources (3 decoder functions in codebase — semantics may diverge per context):
- `src/helpers/payments.ts:781` — `getPaymentModeForActivityLog` (Payments activity log)
- `src/helpers/payments.ts:823` — `getPaymentModeForActivityLogForExpense` (Expenses activity log)
- `src/helpers/payments.ts:1172` — `getPaymentMode` (general/online-only modes)
- `src/helpers/invoices.ts:1278` — `getRefundModeForInvoice` (Refunds context)
- `src/services/payment/constants.ts` — DEPOSIT/ADVANCE/CASH constants

| Value | Meaning (canonical) | Refund context | Used in DA |
|-------|---------------------|----------------|------------|
| `201` | Only CREDITS | — | DA-02 |
| `202` | Cash (Payments + Expenses) | "Cash by OTP" (refunds) | DA-02, DA-04 |
| `203` | Razorpay Payment Link | "RentOk Bank Transfer" | DA-02 |
| `205` | Paytm UPI | "RentOk Bank Transfer" | DA-02 |
| `206` | Google Pay | "RentOk Bank Transfer" | DA-02 |
| `207` | Cash variant | — | DA-02 (CASH_PAYMENT_MODES) |
| `210` | Paytm | "RentOk Bank Transfer" | DA-02 |
| `211` | **Deposit-applied (DEPOSIT_PAYMENT_MODE)** — adjustment mode, EXCLUDED from Net Collected grand total | "Adjusted from deposit" | DA-02, DA-06 |
| `213` | **Context-dependent.** Collections: special — subtracts `payment_owner_credits` (EazyPG/RentOk credits handling). Refunds: "Circle Pe" | "Circle Pe" | DA-02, DA-03 |
| `288` | **Advance-applied (ADVANCE_PAYMENT_MODE)** — adjustment mode, EXCLUDED from Net Collected grand total | "Adjusted from advance" | DA-02, DA-06 |
| `2040` | Cash (default) | "Cash" | DA-02, DA-03, DA-04 |
| `2041` | G Pay | "G Pay" | DA-02, DA-03, DA-04 |
| `2042` | Phone pe | "Phone pe" | DA-02, DA-03, DA-04 |
| `2043` | Paytm | "Paytm" | DA-02, DA-03, DA-04 |
| `2044` | UPI | "UPI" | DA-02, DA-03, DA-04 |
| `2045` | Bank | "Bank" | DA-02, DA-03, DA-04 |
| `2046` | Card Machine | "Card Machine" | DA-02, DA-03, DA-04 |
| `2047` | Cheque | "Cheque" | DA-02, DA-03, DA-04 |
| `2048` | Others / Wallet | "Others" | DA-02, DA-03, DA-04 |
| `2382` | — | "Aliste Bank Transfer" | DA-03 |

Constants (`src/services/payment/constants.ts`):
```ts
export const DEPOSIT_PAYMENT_MODE = 211;
export const ADVANCE_PAYMENT_MODE = 288;
export const CASH_PAYMENT_MODES = [202, 2041, 2040, 207];   // ⚠️ note: includes 2041 (G Pay) which other functions classify separately
```

**⚠️ Pitfalls on payment_mode:**
- The entity comment on `Payments.payment_mode` says `0-cash, 1-paytm, 2-razorpay` — **STALE.** Real values are 3-digit (201-213) and 4-digit (2040-2048, 2382).
- `CASH_PAYMENT_MODES` constant includes `2041` which `getPaymentModeForActivityLog` decodes as "G Pay" — divergence between collections-grand-total cash exclusion and decoder function. Verify per query.
- Same integer can mean different things in different contexts — `213` is "owner_credits special handling" in Collections grand-total, but "Circle Pe" in Refunds decoder. **Context matters.**
- For UI display use the appropriate decoder; for SQL filtering use the integer directly.
- **Mode 213 row-vs-SUM divergence (DA-02 finding 2026-05-08):** SUM-level grand total at `helpers.ts:413` subtracts `payment_owner_credits` once: `raw_total_amount - payment_owner_credits`. Per-row total_amount at `helpers.ts:481-489` subtracts `gateway_charges` TWICE (once via `raw_total_amount` which is already `net_amount - gateway_charges`, then again explicitly at line 488). **Hero number and per-row sums can disagree on mode-213 payments.** Either intentional (preserved per comment "preserving original formula exactly (line 977)") OR a bug — Jatin gate. PRD v3.0 changelog claims this was fixed; production code suggests it wasn't.

### 2.2 Due Type strings (Invoices.due_type and Expenses.expense_type)

Source: `src/services/payment/constants.ts:5-7`.

| Constant | String value | Notes |
|----------|--------------|-------|
| `DEPOSIT_DUE_TYPE` | `'Security Deposit'` | Single primary deposit type |
| `DEPOSIT_DUE_TYPES` | `['Security Deposit', 'Caution Money']` | Both qualify as "deposit" for held-amount calculations |
| `ADVANCE_DUE_TYE` | `'Advance'` | **Note: typo "TYE" in source code, kept verbatim. Use the constant import, not literal string.** |

Other due_type values (not exhaustive — string-based, free-form historically):
- `'Rent'`
- `'Electricity'` / `'Electricity charges'` (variation)
- `'Maintenance'`
- `'Food'`
- Plus any custom strings configured per property

### 2.3 Tenant.status enum (full 9-value set)

Source: `docs/wiki/02-tenant-lifecycle/rules/TEN-001-tenant-status-codes.md` (verified 2026-04-27, the canonical Tenant Status Codes rule).

| Code | Meaning | Description |
|---|---|---|
| `0` | Inactive / Evicted | Tenant has left the property or been removed |
| `1` | Active | Tenant is currently living in the property and paying rent |
| `2` | Booking / Pending | Tenant has been booked into a room but hasn't checked in yet |
| `3` | Lead | A prospective tenant being tracked for follow-up |
| `4` | Invitation / Joining Request | Tenant has received an invite or submitted a joining request |
| `5` | Permanently Deleted | Record removed permanently — cannot be recovered |
| `6` | Invitation Removed / Accepted | Invitation was either accepted (moved forward) or cancelled |
| `7` | Soft-Deleted Lead | Lead removed from active lists but data preserved for re-engagement |
| `8` | Interested (Self-Service) | Tenant self-registered interest via the app or website |

**Default-on-creation:** `2` (booking, set by `initializeTenant` in `tenant.ts`).
**Eviction target:** `0` (set by `clearTenantData` helper).

**For DA suite filtering:**
- DA-01 Dues hero — production filters `t.status IN (1, 2)` (active + booking only). Old tenants (`status=0`) excluded
- DA-06 Move-Out Forecast — `t.status = 1 AND t.date_of_eviction IS NOT NULL` (active tenants under notice)
- "Tenant cohort" rows (active vs old vs booking) — use 1, 0, 2 respectively

### 2.4 Invoices.status enum

Verbatim comment at `src/entities/invoices.ts:93`:
- `0` = Not Paid
- `1` = Paid
- `2` = Partially Paid
- `3` = Refunded
- `4` = Loss

For "paid" filter: `status = 1`.

**✅ Production "outstanding" filter — RESOLVED (2026-05-11):** The original entity comment "use `status IN (0, 2)`" is **wrong**. Production `i.status = 0` only is **correct** because: (a) production NEVER writes `i.status = 2` — only commented-out references at `payment.ts:5358, 5517` exist; (b) partial payments destructively shrink `i.amount` in place at `payment.ts:2039-2048`, so `SUM(i.amount) WHERE status = 0` already includes partial outstanding correctly. **Build Sheet writers MUST use `status = 0` only.** Do not add `status IN (0, 2)` widening — it does nothing today and could double-count in future if the partial-paid pattern is ever fixed. See §4.1 for full details.

### 2.5 Payments.status enum

Verbatim comment at `src/entities/payments.ts:64`:
- `0` = failed
- `1` = success
- `2` = pending
- `3` = refunded

### 2.6 Credits.status + view_status

- `Credits.status`: `0` = unused, `1` = used (verbatim comment).
- `Credits.view_status`: production filter uses `= 101` for "applicable" credits at `paymentService.ts:getApplicableCredits`. Other values exist for scratched/unscratched UX states.
- `Credits.source`: `0` = owner-funded, `1` = eazypg-funded (RentOk-funded under old branding).
- `Credits.is_minimum_full_amount`: `0` = pay full to use, `1` = pay minimum to use.

### 2.7 ChecklistItemCondition enum (`tenant_checklist_item.move_in/out_condition`)

Verbatim from `src/entities/tenantChecklistItem.ts:18-23`:
- `WORKING` = `'working'`
- `DAMAGED` = `'damaged'`
- `MISSING` = `'missing'`
- `PENDING` = `'pending'`

### 2.7b TenantChecklistStatus enum (`tenant_checklist.status`)

Verbatim from `src/entities/tenantChecklist.ts:19-28`:
- `0` = `MOVE_IN_NOT_STARTED`
- `1` = `MOVE_IN_TENANT_COMPLETED`
- `2` = `MOVE_IN_LOCKED`
- `3` = `MOVE_IN_REOPENED`
- `4` = `MOVE_OUT_TENANT_COMPLETED`
- `5` = `MOVE_OUT_TEAM_COMPLETED`
- `6` = `MOVE_OUT_APPROVED` ← canonical "settlement complete" indicator for DA-06 readiness flag
- `7` = `NEXT_MOVE_IN_READY`

### 2.8 Filter Code enums (`src/v1/constants/filterCodes.ts`)

| Enum | Range | Status |
|------|-------|--------|
| `DuesFilterCode` | 1101-1114 | EXISTS |
| `CollectionFilterCode` | 1201-1219 + 2 outliers `1288` and `12881` (suspected typos) | EXISTS — fix the typos |
| `ExpenseFilterCode` | 1301-1310 | EXISTS |
| `RoomFilterCode` | 1401-1420 | EXISTS |
| `RefundFilterCode` | 1500-1599 (proposed range, free) | NOT BUILT — DA-03 needs new build |
| `DiscountFilterCode` | 1601-1699 (proposed range, free) | NOT BUILT — DA-05 needs new build |
| `LiabilityFilterCode` | 1700-1799 (proposed range, free) | NOT BUILT — DA-06 needs new build |
| `CashFlowFilterCode` | 1800-1899 (proposed range, free) | NOT BUILT — DA-07 needs new build |

---

## Section 3 — Permission Vocabulary (REAL keys only)

> **Verified at:** 2026-05-08 against `src/v1/login/property/service.ts`, `src/controllers/property.ts`, `src/helpers/teamMember.ts`, `src/entities/teamMemberProperty.ts`, `src/v1/homepage/service.ts`.

### 3.1 The 11 JWT-mirrored permission keys (camelCase)

Mirrored from DB to JWT permissions hash at 5 sites:
1. `src/v1/login/property/service.ts:79-91`
2. `src/controllers/property.ts:14883`
3. `src/controllers/property.ts:15041`
4. `src/controllers/property.ts:15767`
5. `src/controllers/property.ts:17813`

Plus a 6th rebuild helper: `src/helpers/teamMember.ts:134`.

| JWT key | Maps from DB column | Used in DA |
|---------|---------------------|------------|
| `appAccess` | `app_access` | foundational |
| `cashCollection` | `cash_collection` | DA-02 |
| `recordPayment` | `record_payment` | DA-02 |
| `editInvoices` | `edit_invoices` | DA-01, DA-05 |
| `editTenants` | `edit_tenants` | DA-01, DA-06 |
| `viewInvoices` | `view_invoices` | DA-01, DA-02, DA-03, DA-05 |
| `viewExpenses` | `view_expenses` | DA-04, DA-07 |
| `deleteInvoices` | `delete_invoices` | DA-01 |
| `addTenants` | `add_tenants` | DA-01 |
| `deleteTenants` | `delete_tenants` | DA-01 |
| `viewTenants` | `view_tenants` | DA-01, DA-06 |

**⚠️ Divergence (CSB-6):** `src/helpers/teamMember.ts:136` adds `bankAccess` mirroring `bank_access` — making it a 12-key shape that diverges from the 11-key login service. Jatin gate.

### 3.2 DB-side columns on `team_member_property` (used via `checkAuthInDb`)

NOT mirrored to JWT. Accessed at runtime per-request via `checkAuthInDb` async helper.

Verified via `src/entities/teamMemberProperty.ts`:

| DB column | Purpose | DA spec |
|-----------|---------|---------|
| `view_room` | view rooms screen | DA-01 (multi-property) |
| `view_complaint` | view complaints | DA-04 (link from expense) |
| `view_food` | view food | — |
| `view_invoices_of_self_added_tenants` | fallback view for self-added tenants only | DA-01, DA-02 (production fallback) |
| `record_payment` | record payment action | DA-02 |
| `record_payment_backdate` | record payment with past date | DA-02 |
| `record_payment_otp` | OTP-required record-payment flow | DA-02 |
| `people_reports` / `money_reports` / `cashflow_report` | Excel report exports | DA-07 |
| `dashboard_access` | overall dashboard access | foundational |
| `add_refund_access` | initiate refunds | DA-03, DA-06 |
| `delete_refund` | delete a refund | DA-03 |
| `submit_or_update_moveout_checklist_access` | submit move-out checklist | DA-06 |
| `approve_moveout_checklist_access` | approve move-out checklist | DA-06 |
| `edit_eviction_access` | edit eviction details | DA-06 |
| `cancel_eviction_access` | cancel eviction | DA-06 |
| `view_leads` | view leads | — |
| `delete_old_tenant` | permanently delete historical tenant records | DA-01 (Old Tenants row actions) |
| `change_prop_room` | move tenant between properties/rooms | DA-01 (cross-property reassignment) |
| `edit_tenants_rental_details` | change rent amount, deposit, billing details | DA-01 (rent/deposit edits) |
| `update_tenant_profile_lock_status` | lock/unlock tenant profile | — |

> **Source:** `src/entities/teamMemberProperty.ts` (full column list); `docs/wiki/02-tenant-lifecycle/rules/TEN-019-permission-keys-tenant-operations.md` (semantic documentation).

### 3.3 Derived `ctx` flags (homepage service)

Computed via `any(field)` higher-order function at `src/v1/homepage/service.ts:380` — true if ANY relevant property row has the underlying permission. Used per-row in tile rendering.

```ts
can_view_rooms     = any('view_room')
can_view_tenants   = any('view_tenants')
can_view_leads     = any('view_leads')
can_view_complaints = any('view_complaint')
can_view_invoices  = any('view_invoices')
can_view_expenses  = any('view_expenses')
can_view_food      = any('view_food')
```

These derived flags are what Build Sheet rows reference in the `view:` portion of the Permission cell.

### 3.4 Spec-cited permission keys vs reality

Reproduced from prior audit. **Each MISSING key is a Jatin decision (build new vs reuse existing).**

| Cited key | Status | Recommended reuse |
|-----------|--------|-------------------|
| `viewDues` (DA-01) | MISSING | reuse `viewInvoices` |
| `viewCollections` (DA-02) | MISSING | reuse `viewInvoices` |
| `viewRefunds` (DA-03) | MISSING | build new OR reuse `viewInvoices` |
| `viewDiscounts` (DA-05) | MISSING | build new OR reuse `viewInvoices` |
| `viewLiabilities` (DA-06) | MISSING | reuse `viewInvoices` (composite) |
| `viewCashFlow` (DA-07) | MISSING | build new OR composite `viewInvoices AND viewExpenses` |
| `editDiscounts` (DA-05) | MISSING | build new OR reuse `editInvoices` |
| `processRefunds` (DA-06) | MISSING | use existing DB `add_refund_access` via `checkAuthInDb` |
| `processMoveOuts` (DA-06) | MISSING | use existing DB `submit_or_update_moveout_checklist_access` (and siblings) |
| `sendReminders` (DA-01) | MISSING | reuse `viewInvoices` OR build new |
| `viewTenantDetails` (DA-01, DA-06) | MISSING | use `viewTenants` JWT key |

---

## Section 4 — Production Aggregation Patterns

> **Verified at:** 2026-05-08 by direct read of service files. SQL/QueryBuilder snippets are verbatim.

### 4.1 Dues hero (DA-01) — corrected 2026-05-11

**Files:**
- `src/v1/list_screens/dues/service.ts:26-50` (worklist) + `src/v1/list_screens/dues/helpers.ts:52-110` (query builder)
- `src/v1/homepage/service.ts:2005-2019` (Financials tile — the canonical DA-01 Hero source)
- `src/v1/homepage/service.ts:480-486` (separate header-stats "Total Dues" tile — divergent semantics, see ⚠️ below)

**Auth:** `checkAuth(req, pg, 'viewInvoices')` with fallback `checkAuthInDb(req, pg, 'view_invoices_of_self_added_tenants')` at `service.ts:36, 130`.

**Verbatim production filter** (from `src/v1/list_screens/dues/helpers.ts:60-66`, single source for both worklist and homepage Financials tile via `DuesListHelper.buildBaseQuery`):

```ts
.where('i.property = ANY(:property_strings)', { property_strings: propertyStrings })
.andWhere('i.status = 0')             // status=0 ONLY (Not Paid). Partial-paid invoices DO NOT exist as status=2 in prod — see "Partial-paid pattern" below
.andWhere('i.amount >= 1')
.andWhere('i.is_active = 1')
.andWhere('t.status IN (1, 2)');      // ⚠️ Excludes old tenants (t.status=0) — needs widening per DA-01 spec, HB6
```

**Sum expression:** `SUM(i.amount)` — uses `i.amount` directly. **Correct for both fully-unpaid AND partial-paid outstanding** because of the in-place mutation pattern (see below).

**Group-by:** `.having('SUM(i.amount) >= 1')` filters tenants with negligible totals.

**Partial-paid pattern (CRITICAL pitfall — verified 2026-05-11):**

Production does NOT use `i.status = 2` ("Partially Paid"). The only references to `status = 2` in `controllers/payment.ts:5358, 5517` are commented out. **Zero rows in production have `status = 2`.**

When a tenant makes a partial payment, the production code at `src/controllers/payment.ts:2039-2048` **mutates `i.amount` in place**, shrinking it to the new outstanding balance, while `i.status` stays `0` and `i.paid_amount` stays `0`:

```ts
// payment.ts:2039-2048 — partial-paid branch
invoices[i].net_amount = Number((Number(invoices[i].net_amount) - Number(totalAmount)).toFixed(2));
invoices[i].amount = Number((Number(invoices[i].net_amount)));
await invoices[i].save();
partiallyPaidInvoices.push(invoices[i]);
```

**Implication:** the current `SUM(i.amount) WHERE status = 0` already captures partial outstanding correctly — Riya's ₹10K invoice after she pays ₹3K is stored as `amount = ₹7K, status = 0, paid_amount = 0`. The Hero sums ₹7K (correct outstanding) without needing `status = 2` widening.

**Implication for Build Sheets and PRDs:** Hero formula stays `SUM(i.amount) WHERE status = 0`. **DO NOT add `status IN (0, 2)` widening or `amount - paid_amount` math** — both are pointless (no status=2 rows exist) and could silently DOUBLE-COUNT if engineering ever fixes the partial-paid pattern in the future.

**Separate concern (out of DA-01 scope, flag to Jatin/Nimit):** the destructive `amount` mutation loses the original billed amount from the invoice row. Audit trail lives only on the Payments side. Long-standing data-modeling issue.

**Old tenants (`t.status = 0`) — DA-01 HB6 actionable change:**

Currently excluded by `t.status IN (1, 2)`. Per DA-01 PRD v3.4 (post-2026-05-11 spec audit), old tenants WITH unpaid dues should appear in Hero. Engineering change: `helpers.ts:61` → `t.status IN (0, 1, 2)`. Single-line backend fix.

Note: `t.status = 8` ("interested-booking-lead" per wiki TEN-001) is NOT included in the widening — leads do not typically have unpaid dues. If they do, separate spec decision.

**Homepage tile vs Detailed Hero — divergence flagged (2026-05-11):**

Two different "Total Dues" numbers exist on the homepage today:

| Source | Date filter behavior | Default filter |
|---|---|---|
| `getFinancialsV2` Financials tile (canonical DA-01 Hero) at `homepage/service.ts:2005-2019` | **Live (no date filter on SUM)** — buckets by `due_date` via CASE-WHEN | Today |
| `getHeaderStats` "Total Dues" header tile at `homepage/service.ts:480-486` | **Date-filtered** — `i.due_date BETWEEN filterFrom AND filterTo` | Current month |

These can return DIFFERENT numbers on the same screen. **Per DA-01 spec audit 2026-05-11, decision: align both to be always-live.** Engineering change: remove the date filter at `homepage/service.ts:480-486` OR rename the header-stats tile to a different metric ("Dues This Month"). DA-01 PRD captures as HB7.

**Default filter at app open:** Today (Live) — already matches production at `homepage/service.ts:1982-1984`. ✅ No code change needed.

**Aging / Top Defaulters / Defaulter Chart bounded by `due_date ≤ today`:**

Current production at `helpers.ts:295-313` (`applyDefaulters`) is binary 60+ only (HB1) and has NO `due_date ≤ today` guard. Future-period selection naturally returns empty results but with no graceful UI empty-state branch.

Per DA-01 spec audit 2026-05-11, decision: bound these sections by `due_date ≤ today`. Future-period selection → graceful empty state ("Future-dated bills aren't overdue yet. These sections show past-due bills only.") Engineering work: add guard to `applyDefaulters`, add bucket-math guard to urgency-bar sections in `homepage/service.ts:2009-2016`, build UI empty state.

**Pagination:** list service hardcodes `limit = 5000, offset = 0` at `service.ts:27-28`. HB2 — must be surfaced as request params.

**⚠️ Widget self-added permission gap (security, surfaced 2026-05-11, CSB-8):**

`getDuesWidget` at `service.ts:132-138` sets `self_added_team_uuid` for team members with only `view_invoices_of_self_added_tenants` permission BUT NEVER applies it as a WHERE clause to the widget query. Worklist path enforces it (`service.ts:96`); widget path does not.

**Effect:** team member with restricted permission sees correctly-scoped worklist but UN-scoped widget totals (cross-tenant aggregate). Cross-property data leak via widget endpoint.

**Out of DA-01 scope** — pattern likely affects other DA list-screen services too. Documented as Cross-Suite Engineering Blocker CSB-8.

### 4.2 Collections hero — Net Collected (DA-02)

**File:** `src/v1/list_screens/collections/helpers.ts:67-71, 138, 170-171, 430-450`.

```ts
// Per-payment expression chosen at query build time:
if (effectiveInvoiceLevelSum) {
    query.addSelect('SUM(inv.amount)', 'raw_total_amount');
} else {
    query.addSelect('p.net_amount - p.gateway_charges', 'raw_total_amount');
}
```

```ts
// Filter conditions (helpers.ts:170-171):
.andWhere('p.status = 1')
.andWhere('p.is_active = 1');
```

```ts
// Join with invoice constraints (helpers.ts:138):
.leftJoin('p.invoices', 'inv', 'inv.is_active = 1 AND inv.status = 1 AND inv.amount > 0')
```

```ts
// Date filter on paid_date (helpers.ts:337-340):
query.andWhere('p.paid_date::date >= :paid_start', { paid_start: ... });
query.andWhere('p.paid_date::date <= :paid_end', { paid_end: ... });
```

```ts
// Grand total expression with mode handling (helpers.ts:430-450):
const totalExpr = `COALESCE(SUM(
    CASE
        WHEN ${exclusionClause}            THEN 0
        WHEN sub.payment_mode = 213        THEN CAST(sub.raw_total_amount AS numeric) - CAST(sub.payment_owner_credits AS numeric)
        ELSE                                    CAST(sub.raw_total_amount AS numeric)
    END
), 0)`;
// Where exclusionClause = adjustment modes (211, 288 unless explicitly filtered) → 0
```

Auth: `checkAuth(req, pg, 'viewInvoices')` at `collections/service.ts:21, 164` with fallback `checkAuthInDb(req, pg, 'view_invoices_of_self_added_tenants')`.

**Refunds are NOT subtracted in this expression.** PRD says "Net of refunds" — engineering gap or PRD revision (Jatin/PM gate).

### 4.3 Refunds detail (DA-03)

**File:** `src/services/refunds/refunds.ts:488` — `getRefundDetails`.

No aggregation. Single-row fetch by `refund_uuid` and `pg_id`.

**Auth gap:** no `checkAuth` call inside the service — anyone authenticated can read any refund by uuid (CSB-4 / DA-03 HB4).

### 4.4 Expenses list (DA-04)

**File:** `src/v1/list_screens/expenses/service.ts:32, 104`.

Auth: `checkAuth(req, pg, 'viewExpenses')`.

For "expense total in period," filter is:
```
e.is_active = 1
AND e.paid_date::date BETWEEN start AND end
AND status checks per query (NOTE: status field marked TODO-REMOVE)
```

### 4.5 Expense detail (DA-04)

**File:** `src/services/expense/expense.ts:20` — `getExpenseDetails`.

No `checkAuth` call. Also no `pg_id` validation — pure UUID lookup. Anyone authenticated can read any expense by uuid (CSB-4 / DA-04 HB4).

### 4.6 Liabilities held-amount (DA-06) — composite

**File:** `src/services/payment/paymentService.ts:getAvailableAdvance` + `getAvailableDeposit` + `getApplicableCredits`.

Per-tenant logic (NOT aggregated — DA-06 worklist needs new aggregated query):

```ts
// Available Deposit per tenant (paymentService.ts:getAvailableDeposit):
for (let invoice of all_invoices) {
    if (invoice.due_type == DEPOSIT_DUE_TYPE && invoice.status == 1) {
        net_available_security_deposit += Number(invoice.amount);
        for (let refund of invoice.refunds) {
            net_available_security_deposit -= Number(refund.amount);
        }
    }
}
```

```ts
// Available Advance per tenant (getAvailableAdvance):
// SAME PATTERN with invoice.due_type == ADVANCE_DUE_TYE
```

```ts
// Applicable Credits per tenant (getApplicableCredits):
let credits = await Credits.find({
    where: {
        tenant: tenant_firebase_id,
        property: property_uniq_key,
        status: 0,
        expiry_date: MoreThanOrEqual(getCreatedAt()),
        view_status: 101
    }
});
// Sum credits.amount with cash/min-amount logic per credit row
```

**Composite for "total held per tenant" = `available_deposit + available_advance + sum(applicable_credits.amount)`.**

DA-06 worklist needs a NEW aggregated query that runs this per-tenant composite efficiently. Production currently does it per-tenant on demand, not in batch.

**⚠️ Critical pitfall on `getAvailableDeposit` (corrected 2026-05-08 per DA-06 Build Sheet finding):** Production code at `paymentService.ts:2874` uses the **singular** constant `DEPOSIT_DUE_TYPE = 'Security Deposit'` — **NOT** the plural `DEPOSIT_DUE_TYPES` array `['Security Deposit', 'Caution Money']`. Despite both constants being defined at `payment/constants.ts:5-6`, the held-amount logic ignores the plural one. **Net result: Caution Money invoices are NOT counted in held-amount.** If a property charges Caution Money separately, DA-06 hero under-counts held funds. This is now logged as DA-06 HB7 — Jatin gate to either widen the production query (use `due_type IN DEPOSIT_DUE_TYPES`) OR explicitly document the scope as "SD-only."

Earlier Field Map versions (initial creation 2026-05-08 morning) incorrectly stated the formula uses the plural array. **Use singular `'Security Deposit'` only when citing the production formula in Build Sheets.** If a Build Sheet's PRD scope is "SD ∪ CM," it's a NEW BUILD requiring the widened filter.

### 4.7 "Under Notice" / Move-Out forecast pattern

**File:** `src/v1/homepage/service.ts:798-803`.

Production uses:
```sql
COUNT(t.id) FILTER (WHERE t.status IN (1,2) AND t.date_of_eviction IS NOT NULL) AS under_notice
```

NOT `t.under_notice = TRUE`. The `under_notice: boolean` column exists on Tenant entity but is NOT canonical for the homepage tile.

**For DA-06 Move-Out Forecast (next 30 days), the right filter is:**
```sql
t.status = 1
AND t.date_of_eviction IS NOT NULL
AND t.date_of_eviction::date BETWEEN today AND today + 30
AND held_amount(t.id) > 0
```

No production query does this exact filter today — DA-06 needs new query work.

### 4.8 Cash Flow Report (DA-07)

**File:** `src/routes/reports.ts:119` → `POST /reports/cashflow-report` (HeaderValidator-protected).

Existing endpoint generates Excel report for Cash Flow. **Spec earlier claimed "NEW BUILD" for `/generateCashFlowReport` — that's wrong. The endpoint exists.** Phase 1 work is column extension, not endpoint creation.

### 4.9 Discounts (DA-05) hero — does not exist

No aggregation logic for discounts hero exists. DA-05 requires new build at `src/v1/list_screens/discounts/` mirroring expenses pattern.

For "Owner-funded vs RentOk-funded" breakdown:
- `Credits` rows where `status = 1` (used) AND `date_used IN [period]`
- Group by `source` (`0` = owner, `1` = eazypg/RentOk)
- Sum `amount`

---

## Section 5 — Endpoint Map

> **Verified at:** 2026-05-08. Auth state per `src/utils/commonFunctions.ts` (HeaderValidator + checkAuth pattern).

### 5.1 Existing list endpoints (per spec)

| DA | Endpoint | Auth state | File:line |
|----|----------|-----------|-----------|
| DA-01 | `POST /v1/list_screens/dues/list/filters` | HeaderValidator + `checkAuth(viewInvoices)` (with self-added fallback) | `src/v1/list_screens/dues/routes.ts:8` |
| DA-02 | `POST /v1/list_screens/collections/list/filters` | HeaderValidator + `checkAuth(viewInvoices)` (with self-added fallback) | `src/v1/list_screens/collections/routes.ts:8` |
| DA-04 | `POST /v1/list_screens/expenses/list/filters` | HeaderValidator + `checkAuth(viewExpenses)` | `src/v1/list_screens/expenses/routes.ts:8` |

### 5.2 Existing detail endpoints (per spec)

| DA | Endpoint | Auth state | Notes |
|----|----------|-----------|-------|
| DA-03 | `GET /refunds/advanced-details` | HeaderValidator only (CSB-4: no checkAuth) | `src/routes/refunds.ts:10`; service at `src/services/refunds/refunds.ts:488` |
| DA-04 | `GET /expenses/advanced-details` | HeaderValidator only (CSB-4: no checkAuth, no pg_id validation) | `src/routes/expenses.ts:52`; service at `src/services/expense/expense.ts:20` |
| (reuse) | `GET /reimbursement/advanced-details` | HeaderValidator only — same handler as refunds (CSB-7 controller duplication) | `src/routes/reimbursement.ts:10` |

### 5.3 Cross-screen drill destinations

| Drill | Endpoint | Auth state | Notes |
|-------|----------|-----------|-------|
| Tenant Detail (default fetch) | `POST /tenant/getTenantData` | **NO HeaderValidator** (CSB-3) | `src/routes/tenant.ts:927` |
| Tenant Detail — passbook | `POST /tenant/getTenantPassbook` | HeaderValidator | `src/routes/tenant.ts:939` |
| Tenant Detail — stepper | `POST /tenant/getTenantStepperv2` | **NO HeaderValidator** (CSB-3) | `src/routes/tenant.ts:931` |
| Adjust from Deposit | `POST /tenant/adjustDeposit` | **NO HeaderValidator** — financial mutation (CSB-3 P0) | `src/routes/tenant.ts:944` |
| Invoice Detail | `POST /invoices/getInvoiceData` | **NO HeaderValidator** (CSB-3) | `src/routes/invoices.ts:215` |
| Invoice Detail (alt) | `POST /invoices/getSpecificInvoiceData` | **NO HeaderValidator** (CSB-3) | `src/routes/invoices.ts:227` |
| Receipt PDF | `POST /invoices/generateReceipt` | **NO HeaderValidator** (CSB-3) | `src/routes/invoices.ts:216` |
| Receipt PDF (wrapper) | `POST /invoices/generateReceiptWrapper` | **NO HeaderValidator** (CSB-3) | `src/routes/invoices.ts:225` |
| Refund Creation | `POST /refunds/advanced-addition` | HeaderValidator | `src/routes/refunds.ts:9` |
| Refund metadata (form prefill) | `GET /refunds/metadata` | HeaderValidator | `src/routes/refunds.ts:8` |
| Move-out checklist items | `GET /checklist/move-out/items` | HeaderValidator | `src/routes/moveInMoveOutRoutes.ts:63` |
| Move-out checklist mark item | `PUT /checklist/move-out/mark-item` | HeaderValidator | `src/routes/moveInMoveOutRoutes.ts:70` |
| Move-out checklist complete | `POST /checklist/move-out/complete-checklist` | HeaderValidator | `src/routes/moveInMoveOutRoutes.ts:77` |
| Send Reminder (bulk per tenant) | `POST /invoices/sendBulkPaymentReminder` | HeaderValidator | `src/routes/invoices.ts:231` |
| Send Reminder (bulk per property) | `POST /invoices/sendBulkPaymentReminderForProperty` | HeaderValidator | `src/routes/invoices.ts:232` |
| Complaint detail | `GET /complaints/fetch/:complaint_id` | HeaderValidator | `src/routes/complaints.ts:182` |

### 5.4 Drill endpoints with ZERO middleware (CSB-2)

| Endpoint | Used by | File:line |
|----------|---------|-----------|
| `POST /invoices/fetchDueDetailsForTenants` | DA-01 worklist row drill | `src/routes/invoices.ts:203` → `controllers/invoices.ts:3331` |
| `POST /invoices/fetchPaymentSettlementDetails` | DA-02 worklist row drill | `src/routes/invoices.ts:204` → `controllers/invoices.ts:3424` |

These have NO HeaderValidator AND NO checkAuth. Anonymous read of payment data. P0 fix before launch.

### 5.5 Excel report endpoints (existing)

| Endpoint | Pattern | File:line |
|----------|---------|-----------|
| `POST /reports/generateNewDuesReport` | DA-01 dashboard-overflow | `src/routes/reports.ts:89` |
| `POST /reports/generateCollectionReport` | DA-02 | `src/routes/reports.ts:91` |
| `POST /reports/generateCollectionReportV2` | DA-02 | `src/routes/reports.ts:107` |
| `POST /reports/generateExpenseReport` | DA-04 | `src/routes/reports.ts:86` |
| `POST /reports/generateSettlementReport` | DA-06 (move-out settlements) | `src/routes/reports.ts:90` |
| `POST /reports/cashflow-report` | DA-07 | `src/routes/reports.ts:119` |

Naming convention diverges: older reports are camelCase (`generateXReport`); newer (cashflow-report) is kebab-case. Recommend kebab-case for new builds.

### 5.6 NEW BUILD endpoints (Phase 1)

| Endpoint | DA | Pattern to mirror |
|----------|-----|-------------------|
| `POST /v1/list_screens/refunds/list/filters` | DA-03 | Pattern: `src/v1/list_screens/expenses/` |
| `POST /v1/list_screens/discounts/list/filters` | DA-05 | Same |
| `POST /v1/list_screens/liabilities/list/filters` | DA-06 | Same |
| `POST /v1/list_screens/cashflow/list/filters` | DA-07 | Same (or extend `/reports/cashflow-report` shape) |
| `POST /reports/discount-report` | DA-05 Excel | NEW — kebab-case per recent convention |
| `POST /reports/liability-report` | DA-06 Excel | NEW |

---

## Section 6 — Common Hallucinations / Anti-Patterns

> If a Build Sheet writer is tempted to use any of these, STOP and use the verified alternative instead.

| Tempting (WRONG) | Reality (RIGHT) |
|------------------|-----------------|
| `payment.payment_date` | `payment.paid_date` |
| `payment.amount` | `payment.net_amount` (or `net_amount - gateway_charges` for collections sum) |
| `payment.mode = 'rentok'` (string) | `payment.payment_mode IN/NOT IN (211, 213, 288, 202, 2041, 2040, 207)` (integer) |
| `tenant.exit_date` | `tenant.date_of_eviction` |
| `tenant.held_amount` | computed via Section 4.6 composite |
| `tenant.has_held_amount` | not a field; computed |
| `tenant.under_notice = TRUE` | use `tenant.date_of_eviction IS NOT NULL` (production canonical) |
| `tenant.status` enum exhaustive at 0/1/2 | actual enum has **9 values (0-8)** per wiki TEN-001 — see Section 2.3 for full table |
| `tenant_checklist_item.status = 'completed'` | NO `status` column on items. Use `tenant_checklist.status = 6` (MOVE_OUT_APPROVED) on PARENT entity for settlement-readiness |
| Settlement-readiness via item-level `move_out_marked_at IS NOT NULL` | Use parent `tenant_checklist.status >= 6` instead. Item timestamps are for granular UI only |
| `viewDues` permission key | reuse `viewInvoices` (`viewDues` does not exist) |
| `viewRefunds` / `viewDiscounts` / `viewLiabilities` / `viewCashFlow` | NONE EXIST as permission keys (Jatin gate per Section 3.4) |
| `processRefunds` permission key | use DB column `add_refund_access` via `checkAuthInDb` |
| `processMoveOuts` permission key | use DB column `submit_or_update_moveout_checklist_access` (and 3 siblings) |
| `viewTenantDetails` permission key | use `viewTenants` JWT key |
| Refunds entity has `is_active` or `status` | NO — every refund row is final |
| Invoice "outstanding" = `status IN (0, 2)` per entity comment | **WRONG. Use `status = 0` only.** Production NEVER writes `status = 2` (commented-out refs only at `payment.ts:5358, 5517`). Partial-paid is captured via destructive `amount` mutation at `payment.ts:2039-2048` — `SUM(i.amount) WHERE status = 0` already includes partial outstanding correctly. Adding `status IN (0, 2)` is pointless AND could double-count if pattern is ever fixed. See §4.1 |
| Outstanding math uses `SUM(i.amount - i.paid_amount)` | **WRONG.** `paid_amount` is never populated on partial-paid rows (mutation pattern leaves it at default `0`). Use `SUM(i.amount)` — it's already correct |
| `getHeaderStats` "Total Dues" tile = DA-01 detailed Hero | **WRONG. Two divergent numbers exist on homepage today.** Header stats is date-filtered (defaults to current month) at `homepage/service.ts:480-486`. Financials tile is live (no date filter) at `homepage/service.ts:2005-2019`. Per DA-01 spec audit 2026-05-11, both must align to always-live. DA-01 HB7 |
| DA-01 Hero excludes old tenants is intentional | **WRONG.** Per DA-01 spec audit 2026-05-11, old tenants WITH unpaid dues SHOULD appear in Hero. Current `t.status IN (1, 2)` is a bug. Fix: `helpers.ts:61` → `t.status IN (0, 1, 2)`. Population row's Old Tenants treatment surfaces the contribution to operators. DA-01 HB6 |
| Widget endpoint enforces self-added-tenant permission like worklist does | **WRONG. SECURITY GAP.** `getDuesWidget` at `service.ts:132-138` sets the self-added team UUID variable BUT NEVER applies it as a WHERE clause to the query. Worklist path enforces it (`service.ts:96`); widget path does not. Cross-tenant aggregate leak to team members with restricted permission. Likely affects DA-02/03/04 widget endpoints too. Cross-Suite Engineering Blocker CSB-8 |
| Dues hero includes old tenants (`t.status=0`) | Production filter is `t.status IN (1, 2)` — old tenants EXCLUDED from main dues query. DA-01 "Old Tenants" row likely uses separate query path |
| `getAvailableDeposit` sums both Security Deposit AND Caution Money | Production uses **singular** `DEPOSIT_DUE_TYPE = 'Security Deposit'` only at `paymentService.ts:2874`. **Caution Money invoices NOT counted in held-amount** despite plural `DEPOSIT_DUE_TYPES` constant being defined. Corrected 2026-05-08; DA-06 HB7 |
| `Expenses.is_capex` column exists | **DOES NOT EXIST** anywhere in codebase. PRDs reference it for capex flag callouts (DA-04, DA-07). NEW BUILD: add column + classification heuristic. Verified 2026-05-08 |
| `RefundFilterCode` / `DiscountFilterCode` / `LiabilityFilterCode` / `CashFlowFilterCode` exist | NONE EXIST — must be built |
| `/generateCashFlowReport` is NEW BUILD | exists at `/reports/cashflow-report` — Phase 1 is column extension |
| Adjust Deposit endpoint is auth-protected | NO — `/tenant/adjustDeposit` has zero middleware (CSB-3 P0) |
| Tenant Detail endpoint is auth-protected | NO — `/tenant/getTenantData`, `/getTenantStepperv2` have zero middleware (CSB-3) |
| Invoice Detail endpoint is auth-protected | NO — `/invoices/getInvoiceData`, `/getSpecificInvoiceData` have zero middleware (CSB-3) |
| `entity.payment_mode` enum is `0-cash, 1-paytm, 2-razorpay` (per entity comment) | STALE — actual values are 3-digit + special. See Section 2.1 |
| `is_active = 1` means "not voided" | not formally documented as void; means "active row" — combine with status filters |

---

## Section 7 — Maintenance Log

| Date | Change | By |
|------|--------|-----|
| 2026-05-08 | Initial creation. Verified against `master` post-merge (commit `728abfd68`). | PM-bot |
| 2026-05-08 | **Audit pass.** Patches: (a) Section 1.8 — added TenantChecklist parent entity + 8-value status enum (was only TenantChecklistItem); (b) Section 2.7b — added TenantChecklistStatus enum; (c) Section 4.1 — replaced conceptual dues filter with verbatim production filter (`status=0` only, `t.status IN (1,2)`); (d) Section 6 — added 2 new pitfalls (production uses `status=0` not `status IN (0,2)`; production excludes old tenants from dues hero); corrected settlement-readiness pitfall to use parent checklist status. | PM-bot |
| 2026-05-08 | **Graphify pass.** Used graphify queries to surface wiki rules (TEN-001, TEN-019, FIN-118, FIN-121). Patches: (a) Section 2.3 — replaced 4-value Tenant.status (0/1/2/8) with full 9-value enum from wiki TEN-001; (b) Section 3.2 — added 4 missing DB permission columns (`delete_old_tenant`, `change_prop_room`, `edit_tenants_rental_details`, `update_tenant_profile_lock_status`) sourced from TEN-019; (c) Section 6 — corrected exhaustiveness pitfall on tenant status enum. | PM-bot |
| 2026-05-08 | **Phase 3 Wave 1 back-port pass.** DA-02/03/04 Build Sheet agents surfaced new findings: (a) Section 2.1 — replaced 7-value `payment_mode` enum with full 19-value enum sourced from 4 decoder functions (`getPaymentMode`, `getPaymentModeForActivityLog`, `getPaymentModeForActivityLogForExpense`, `getRefundModeForInvoice`); (b) Section 2.1 — added pitfall on context-dependent decoding (213 = "owner_credits handling" in Collections vs "Circle Pe" in Refunds); (c) Section 2.1 — added pitfall on Mode 213 row-vs-SUM divergence at `helpers.ts:413` (single owner_credits subtraction) vs `:481-489` (gateway_charges subtracted twice) — PRD v3.0 claims fixed but code shows otherwise; (d) Section 2.1 — flagged `CASH_PAYMENT_MODES` constant contains `2041` which other decoders call "G Pay" — divergence. | PM-bot |
| 2026-05-08 | **Phase 3 Wave 2 back-port (DA-06).** DA-06 agent caught a Field Map hallucination: I had claimed `getAvailableDeposit` uses `DEPOSIT_DUE_TYPES` plural array (SD + CM). Production code at `paymentService.ts:2874` uses **singular** `DEPOSIT_DUE_TYPE = 'Security Deposit'` only. Caution Money is NOT counted in held-amount. Patched: (a) Section 4.6 — corrected formula to single 'Security Deposit'; (b) Section 6 — added pitfall row on the divergence; (c) DA-06 PRD HB7 added separately. **Lesson: even Field Map can hallucinate; agent verification is genuinely valuable.** | PM-bot |
| 2026-05-08 | **Phase 3 Wave 2 back-port (DA-05).** DA-05 agent flagged `Credits.amount` unit ambiguity — entity declares as plain `number` (no `type:'decimal'`), so likely int. Diverges from `Payments.net_amount` which is `decimal`. Whether producer code writes rupees or paise needs verification before any Build Sheet sums Credits.amount alongside payment amounts. Patched Section 1.4 with VERIFY tag. DA-05 PRD HB5 captures the launch-blocking aspect. | PM-bot |
| 2026-05-11 | **DA-01 spec audit + pre-edit verification pass.** Major corrections: (a) §1.1 Invoices.status comment — corrected from "use status IN (0, 2)" to "use status = 0 only" (production never writes status = 2; partial-paid handled via destructive `amount` mutation at `payment.ts:2039-2048`); (b) §1.1 pitfalls — replaced 2 wrong pitfalls + added 1 new pitfall about destructive `amount` mutation losing original billed amount; (c) §4.1 Dues hero — complete rewrite documenting: production filter, partial-paid pattern, homepage tile vs Detailed Hero divergence (DA-01 HB7), DA-01 HB6 actionable change (widen `t.status` to `IN (0,1,2)`), Aging/Defaulters future-bound rule, widget self-added permission gap (CSB-8); (d) §6 hallucination table — added 5 new pitfalls covering: status semantics, paid_amount math, header stats divergence, old tenants widening, widget perm gap. Source: pre-edit verification agent 2026-05-11 + DA-01 spec audit with user. | PM-bot |

---

## Related Documents

- `[[DA-01 Dues Detailed Analytics]]` — DA-01 PRD (canonical "why")
- `[[DA-02 Collections Detailed Analytics]]` — DA-02 PRD
- `[[DA-03 Refunds Detailed Analytics]]` — DA-03 PRD
- `[[DA-04 Expenses Detailed Analytics]]` — DA-04 PRD
- `[[DA-05 Discounts Detailed Analytics]]` — DA-05 PRD
- `[[DA-06 Liabilities Detailed Analytics]]` — DA-06 PRD
- `[[DA-07 Cash Flow Detailed Analytics]]` — DA-07 PRD
- `[[DA-01 Build Sheet]]` — DA-01 Build Sheet (V1, will be updated to v2 with new format)
- (Forthcoming) `[[DA-02 Build Sheet]]`, `[[DA-03 Build Sheet]]`, etc.
