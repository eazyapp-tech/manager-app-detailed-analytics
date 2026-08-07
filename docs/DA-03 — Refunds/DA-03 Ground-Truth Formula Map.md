---
title: DA-03 Refunds - Ground-Truth Formula Map
date: 2026-06-06
tags: [rentok, refunds, ground-truth, formula-map]
status: Living document · v2.0
owner: Sanchay
---

> [!INFO] What this is
> This document defines every number on the Refunds detailed-analytics screen.
>
> It explains what each number means, how time affects it, how drill-down works, and what must add back to what. It is written so design, engineering, QA, and product all use the same meaning.

## 1. The base rule

DA-03 is about **completed refunds**.

**Testable rule: a refund counts when, and only when, its row exists in the `refunds` table.** There is no status, is_active, or deleted column on `refunds` (entity `refunds.ts:5-44`). Deletion is a hard `DELETE` — the row is physically removed. So "the row exists" is the whole filter. The base query has **no status filter** and must not pretend one exists.

When a refund is deleted, the delete flow (`invoices.ts:8351-8355`) first inserts a snapshot into the separate `deleted_refunds` table, then calls `.remove()` to physically delete the live row. The snapshot insert runs under a swallowed `.catch(() => null)` (`invoices.ts:8354`), so `deleted_refunds` can under-count real deletions — flag this for the audit panel; do not treat `deleted_refunds` as a complete deletion ledger.

**"Add back to" means: when you sum the parts, they equal the whole.** Reason lines, Tenant lines, Property lines, and Mode lines each sum to Total Refunds.

Pending obligations, deleted refund history, and expense entries are not live refunds.

### Base query (mirror cash-flow)

DA-03's base query mirrors the cash-flow report (`generateCashFlowReport.ts:261-272`), which is the bridge that matches DA-03's intended scope:

```sql
FROM refunds r
JOIN invoices i ON i.id = r.invoice_id
WHERE i.property = $1
  AND r.refund_date BETWEEN $2 AND $3
```

No invoice-status filter, no `r.amount > 0` filter — same as cash-flow. The collections and deposits-held bridges use narrower scopes (see §14, §15) and will **not** tie 1:1 to this base.

### `refunds` table schema (entity `refunds.ts:5-44`)

| Column | Type | Note |
|---|---|---|
| `id` | uuid PK | |
| `amount` | int | Stored as-is — format directly as INR. **NO /100.** |
| `refund_date` | timestamp (default now) | Time column for every period number. |
| `refund_reason` | text, nullable | Free remark. Not a clean category. |
| `refund_mode` | int | See mode table below. |
| `refunded_by` | nullable | **Never set on the save path** — see §12. |
| `refunded_by_name` / `refunded_by_phone` | nullable | Only these two are set, from `req.user.partnerName` / `partnerPhone`. Often null. |
| `refund_images` | json | |
| `invoice_id` | uuid FK → `invoices.id` | Links to the bill. `invoices.payer` is the firebase_id used to resolve the tenant. |

There is **no** `is_active`, `status`, or `deleted` column.

## 2. Verified facts

These facts were checked against the current backend. Evidence links stay in the Build Sheet.

| Fact |
|---|
| A refund row stores amount, refund date, reason text (free remark), payment mode, recorder name/phone, images, and one linked bill. It has no status, is_active, or deleted column — no pending, failed, or settled state exists. |
| Creating a refund needs a refund date, payment mode, how the amount is split across bills, and how the money source is split. |
| The final refund save sets amount, refund date, payment mode, reason text, and linked bill. It sets `refunded_by_name` and `refunded_by_phone` only — it never sets `refunded_by` (`invoices.ts:8133-8140`). Recorder name/phone are often null. |
| Mode 2049 ("Advance Credits") is **blocked at write** today (`invoices.ts:7951-7955` rejects it with "Refund with advance credits is coming soon."). It appears only in old data predating the block. |
| Deleting a refund is a hard `DELETE`: snapshot into `deleted_refunds`, physically `.remove()` the live row, then write money-trail reversal entries. The snapshot insert is best-effort (swallowed catch). |
| Deletion history stores the removed refund and the deletion time. |
| A refund detail page can still show a deleted refund if opened from its exact refund link, but that fallback is not part of live totals. |
| Collections subtract refunds from collected bill amount to compute net collection. |
| Deposits Held subtracts refunds from paid Security Deposit and Caution Money bills. |
| Cash-flow reporting reads refunds by refund date and shows them as Refunds (-). |
| The old expenses path can label Deposit-like expenses as deposit refunds, which can duplicate a real refund if staff entered both. |

DA-03 must preserve the current refund amount unit. `refunds.amount` is stored already in rupees and shown directly as INR by the existing path. **Do not divide or multiply by 100 inside DA-03** — a blind paise conversion would make refunds show 100x too big. Any paise standardization is a separate migration that must change the money data everywhere first.

## 3. Words that must stay separate

| Word | Meaning |
|---|---|
| **Refund** | A row in `refunds`. Money recorded as already returned. |
| **Recipient** | Whoever the linked bill's payer is (`invoices.payer`). May or may not match a current tenant record. Used consistently instead of "tenant" for the person who got the money. |
| **Pending refund obligation** | Money the owner may still owe back, usually deposit held. Not DA-03. |
| **Deleted refund** | A refund removed from live records, with a saved copy in deletion history. Audit history only. |
| **Refund reason** | Plain bucket derived from the linked bill type and, where needed, refund remarks. |
| **Refund mode** | How money went out: cash, digital, bank, card, cheque, or other. |
| **Gross collection** | Money collected before subtracting refunds. |
| **Net collection** | Collection after subtracting refunds. |
| **Deposits held** | Paid deposit money still held after refunds and adjustments. |

## 4. Duration modes

The screen uses refund date for every period-based refund number.

| Duration | Meaning |
|---|---|
| **Today** | Refunds dated today in India time. |
| **This Week** | Refunds dated inside the current week. |
| **This Month** | Refunds dated from the first day of this month through today. |
| **Last Month** | Refunds dated inside the previous calendar month. |
| **Current FY** | Refunds dated inside the current financial year. |
| **Custom** | Refunds dated inside the selected start and end dates. |

Future ranges return zero refunds and a plain empty state. Do not show projected refunds.

## 5. Time behavior matrix

| Section | Uses duration picker? | Time rule |
|---|---:|---|
| Total Refunds | Yes | Add live refunds by refund date. |
| Prior-period comparison | Yes | Compare to the previous matching period by refund date. For a partial month, compare the **same number of days from the month's start** — e.g. on day 6 of June (Jun 1–6), compare against May 1–6, not all of May. |
| Reason Breakdown | Yes | Group the same refunds by reason. |
| Tenant Breakdown | Yes | Group the same refunds by recipient. |
| Property Breakdown | Yes | Group the same refunds by property. |
| Mode Breakdown | Yes | Group the same refunds by payment method. |
| Review Signals | Yes | Compute from the same refunds. |
| Processed By | Yes | Group the same refunds by who recorded the refund, with money-trail attribution used only when reliable. |
| Fund Source | Yes | Group money-trail entries linked to live refunds in the period. |
| Deposit-held bridge | Yes | Only deposit/caution refunds in the same refunds. One-directional: DA-03's period deposit refunds are a subset of the lifetime refund amount deposits-held subtracts (deposits-held is un-dated). Not a per-period equality. |
| Cash-flow bridge | Yes | Same refund amount that DA-07 shows as Refunds (-) for the period. |
| Deleted-refund audit | Yes, if shipped | Uses deletion date as the audit period. Shows original refund date separately. Never enters live totals. |

## 6. Total Refunds

| Number | Plain formula | What it tells the owner |
|---|---|---|
| **Total Refunds** | Add `refunds.amount` for selected properties, where refund date is inside the selected duration (every row that exists — no status filter). | "This much money went back out." |
| **Refund Count** | Count refund rows in the same set. | "How many refund entries were recorded." |
| **Tenant Count** | Count distinct recipients in the same set. | "How many people received money." |
| **Prior-period change** | Current total minus previous matching-period total. | "Refunds went up or down." |

Direction: more refunds is worse for the owner in a money-out view. Higher than prior period should read as bad unless the design explicitly explains the exception.

If prior total is zero and current total is above zero, do not show a percentage. Show "No prior refunds" plus the current amount.

## 7. Reason Breakdown

This view uses the selected duration.

Reason comes from the linked bill's `due_type` (`invoices.due_type`). **`due_type` is uncontrolled free text** — same idea spelled many ways, plus dynamic month strings like 'May Rent' or 'Pending July Rent'. So matching must be **case-insensitive substring / regex (`ILIKE`)**, not exact-equals. The real prod variants are enumerated below. Until tuned against a `SELECT DISTINCT due_type` pull from prod, expect "Other" to be non-trivial.

Deposit category is the one fixed pair: `due_type IN ('Security Deposit', 'Caution Money')` (`homepage/service.ts:2451`).

| Reason | Matching rule (case-insensitive) | What it tells the owner |
|---|---|---|
| **Deposit** | `due_type = 'Security Deposit'`. | "Deposit money returned after collection." |
| **Caution Money** | `due_type = 'Caution Money'`. | "Caution money returned." |
| **Advance** | `due_type = 'Advance'` (the **bill type**, not refund mode 2049 "Advance Credits" — do not conflate the two). | "Advance money returned or adjusted back." |
| **Rent** | `due_type ILIKE '%rent%'`, **excluding** 'Renewal Charges'. Catches dynamic strings: 'May Rent', 'Pending July Rent', 'Annual Rent Installment 2'. | "Rent money was corrected back." |
| **Late Fine** | `ILIKE '%late%'` — Late Fine, Manual Late Fine, Automatic Late Fine. | "A late fine was returned." |
| **Utility / Service Correction** | `ILIKE '%maint%'` (Maintenance Bill, typo "Maintanance charge"), Electricity / Electricity Bill / Electricity Recharge / Electricity Meter, Water, Food / Food Charges / Meals / Mess. | "Service-related collection was returned." |
| **Other named charges** | Joining Fee, Renewal Charges, Move Out Charges, Penalty Charges (each matched by its phrase). | "A specific named charge was returned." |
| **Goodwill / Manual** | **Open build item, not a settled bucket.** The remark-to-reason phrase map is unbuilt. Until a prod `refund_remarks` phrase-pull is done, these route to **Other**. | "Owner returned money outside a standard bill bucket." |
| **Other** | Anything not matched above, including unbuilt goodwill/manual. | "Refunds that need review or better labels." |

**Owner decision (locked this session):** the **Rent** bucket = every `due_type` containing 'rent' case-insensitive (`ILIKE '%rent%'`), so dynamic month strings roll in. **Exclude 'Renewal Charges' explicitly** so it does not false-match.

Show top reasons by amount, then Others if the visible line limit is reached.

Reason lines must add back to Total Refunds. Unmatched and unbuilt-goodwill lines stay under Other.

## 8. Tenant Breakdown

This view uses the selected duration.

| Number | Plain formula | What it tells the owner |
|---|---|---|
| **Tenant refunded amount** | Add refunds grouped by recipient. | "Who received the most money." |
| **Tenant refund count** | Count refunds for that recipient. | "Was this one refund or several entries?" |
| **Tenant state** | Match the recipient to the tenant record for the same property; prefer active, then booking, then moved-out, then removed record. | "Whether this refund is normal for the tenant journey." |

Tenant state labels:

| State | Meaning |
|---|---|
| **Active** | Tenant is currently living. Refunds here need review unless clearly a small correction. |
| **Moved Out** | Tenant has left. Deposit and caution refunds are common here. |
| **Booking** | Tenant has booking state. Refunds may be booking cancellation or advance return. |
| **Tenant record removed** | Recipient cannot be resolved to a tenant record. Refund still counts. |

Tenant lines must add back to Total Refunds.

## 9. Property Breakdown

This view appears when the owner is viewing more than one property.

| Number | Plain formula | What it tells the owner |
|---|---|---|
| **Property refund amount** | Add refunds linked to bills for one property. | "Which property returned the most money." |
| **Property share** | Property refund amount divided by total refunds across selected properties. | "How much of the refund load sits here." |
| **Property tenant count** | Distinct recipients for that property. | "Is this one large refund or many tenants." |

Property lines must add back to Total Refunds.

## 10. Mode Breakdown

This view uses the selected duration. `refund_mode` is an int (`helpers/invoices.ts:1299-1326`). The write path enforces 2040–2049.

| Code | Label | Bucket |
|---|---|---|
| 2040 | Cash | Cash |
| 202 | Cash by OTP | Cash (exists, was previously unhandled — `helpers/invoices.ts:1303`) |
| 2041 | G Pay | Digital |
| 2042 | Phone pe | Digital |
| 2043 | Paytm | Digital |
| 2044 | UPI | Digital |
| 2045 | Bank | Bank |
| 2046 | Card Machine | Card Machine |
| 2047 | Cheque | Cheque |
| 2048 | Others | Other |
| 2049 | Advance Credits (disabled) | Other (old data only) |
| default / any unmapped | "-" | Other |

| Mode | Matching rule | What it tells the owner |
|---|---|---|
| **Cash** | 2040 Cash, or 202 Cash by OTP. | Physical cash returned. |
| **Digital** | 2041 G Pay, 2042 Phone pe, 2043 Paytm, 2044 UPI. | Digital refund. |
| **Bank** | 2045 Bank. | Bank transfer. |
| **Card Machine** | 2046 Card Machine. | Card-machine refund. |
| **Cheque** | 2047 Cheque. | Cheque refund. |
| **Other** | 2048 "Others", any 2049 in old data, and **any unmapped code**. | Other / catch-all. |

**Mode 2049 = "Advance Credits", BLOCKED at write** (`getRefundModeForInvoice` returns "Advance Credits" at `helpers/invoices.ts:1321-1322`; the write block at `invoices.ts:7951-7955` rejects 2049 with "Refund with advance credits is coming soon."). It is **not** a defensive legacy "Other" — it is a real, disabled mode that appears only in old data predating the block. Label it "Advance Credits (disabled)".

Any unmapped `refund_mode` must fall into **Other** so Mode lines still add back to Total Refunds.

## 11. Review Signals

Signals are not separate money totals. They are flags on the live refunds.

| Signal | Plain formula | What it tells the owner |
|---|---|---|
| **Largest Refund** | Find the biggest single live refund in the selected duration. | "The one refund to verify first." |
| **Active-tenant Refunds** | Add refunds where resolved tenant state is Active. | "Money returned to people still living here." |
| **Repeat Refunds** | Same recipient has more than one refund in the selected duration. | "Could be normal split refund, or duplicate entry." |
| **Possible Duplicate** | Same recipient, same amount, same property, within a short time window. | "Check whether the same refund was entered twice." |
| **Unattributed Refunds** | Refund exists but recorder or money-trail detail is missing. | "Refund amount exists, but staff/fund trail is incomplete." |

Signal lines do not add together. One refund can trigger more than one signal.

## 12. Processed By

This view uses the selected duration and should ship only after attribution rules are reliable.

| Number | Plain formula | What it tells the owner |
|---|---|---|
| **Refunded By** | Group refunds by recorder. The save path sets only `refunded_by_name` / `refunded_by_phone`; `refunded_by` is **never set** (`invoices.ts:8133-8140`), so even name/phone are often null. Where the money trail has a stronger team-member match, use that. | "Who recorded or handled refunds." |
| **Unattributed** | Refund has no reliable recorder or money-trail match. Expect this to be common — `refunded_by` is mostly null. | "The refund exists, but attribution is missing." |

Unattributed is real and unavoidable. The passbook REFUND write sits in a try/catch that only `console.error`s (silent), is skipped for 4 hardcoded pg_ids, and is skipped when there is no `team_uuid` (`teamPassbook.ts:4313-4335`). So money-trail attribution will be missing for whole property sets. Do not hide unattributed refunds.

Processed By lines must add back to Total Refunds, including Unattributed.

## 13. Fund Source

This view uses the money trail linked to each refund.

| Source | Matching rule | What it tells the owner |
|---|---|---|
| **Petty Cash** | Owner/property cash source. | "Owner/property cash funded the refund." |
| **Tenant-held Funds** | Tenant-held money source. | "Tenant-held funds funded the refund." |
| **Staff Personal Money** | Team member's personal money source. | "A team member fronted money and may need reimbursement." |
| **Unattributed** | No matching money-trail entry. | "The refund exists, but source trail is missing." |

Fund source lines should add back to Total Refunds only after unattributed is included. If money-trail data is not reliable, hide this section or show it as partial with a clear label.

## 14. Deposit-held Bridge

This is a bridge, not a second refund total.

| Number | Plain formula | What it tells the owner |
|---|---|---|
| **Deposit Refunds** | Add refunds whose linked bill `due_type IN ('Security Deposit','Caution Money')`. | "Deposit-related money returned." |
| **Deposits Held Reduction** | The lifetime deposit refund amount that deposits-held subtracts. | "Deposit refunds reduce money still owed back later." |

**One-directional, not a per-period equality.** Deposits Held (`homepage/service.ts:2469-2484`) is live / no-date, runs on `net_amount`, dedups tenants via a CTE, and subtracts refunds **and** payments where `payment_mode NOT IN (211, 288, 291)`. Critically, its refund subtraction is **un-dated** — it subtracts lifetime refunds, not this period's. So the correct relation is:

> DA-03's period deposit refunds **⊆** the lifetime refund amount deposits-held subtracts.

Do not assert `DA-03 deposit refunds = deposits-held reduction` for a period. DA-03 should not show pending deposit obligations as refunds.

## 15. Collections Bridge

This is a bridge, not a second refund total.

| Number | Plain formula | What it tells the owner |
|---|---|---|
| **Refund impact on net collection** | Refund amount subtracted from linked paid bill amount in collection logic. | "The collection happened, then part of it went back out." |
| **Gross collection** | Paid bill amount before refund subtraction. | "Money collected before refund." |
| **Net collection** | Paid bill amount minus refunds. | "Money retained after refund." |

Net formula (`collections/helpers.ts:713-728`):

```
SUM( inv.amount − COALESCE(ref_agg.total_refunded, 0) )
WHERE inv.status = 1 AND inv.amount > 0 AND inv.is_active = 1
-- refund subquery: r.amount > 0, refund side UN-DATED
```

The refund side is **un-dated**: a refund dated next month still reduces a paid bill's net **today**. So this bridge will not tie 1:1 to DA-03's period-scoped, status-free base.

**Open cross-screen question (for the DA-02 owner):** whether DA-02 net collections uses this homepage `collections/helpers` query or its own query determines whether the collections bridge ties out. Leave unresolved here.

Do not say the collection never happened. Say the refund reduced the net amount kept.

## 16. Cash-flow Bridge

Cash flow reads refunds by refund date and shows them as money out. **This is the bridge that matches DA-03's intended base.**

| Number | Plain formula | What it tells the owner |
|---|---|---|
| **Refunds (-)** | Add refund amounts by refund date in the cash-flow period. | "Refunds are part of money out." |

Cash-flow (`generateCashFlowReport.ts:261-272`) calls `Refunds.find` by `refund_date Between`, scoped via the `invoice.property` relation, with **no** is_active / status filter and **no** `r.amount > 0` filter. DA-03's base query mirrors this exactly (refund_date + property, no invoice-status filter — see §1). The collections and deposits-held bridges use narrower scopes and will not tie 1:1.

Known risk: the cash-flow report adds refunds into total expenses, while expenses can also contain deposit-refund-like entries. QA must test for double count.

## 17. Deleted Refund Audit

Deleted refunds are optional audit scope. They never enter live refund totals.

| Number | Plain formula | What it tells the owner |
|---|---|---|
| **Deleted Refund Count** | Count refunds removed during the selected audit period. | "How many refunds were removed." |
| **Deleted Refund Amount** | Add amounts from saved deleted-refund copies for those removed refunds. | "How much deleted refund history exists." |
| **Reversal Trail** | Match the refund deletion to its money-trail reversal. | "Whether the money trail was reversed." |

`deleted_refunds` schema (`deletedRefunds.ts:9-23`): `id` (serial), `refund_uuid` (uuid), `refund_json` (jsonb — the full snapshot), `created_at` (= deletion time). The **original** `refund_date` lives inside `refund_json`, not as a column. Use `created_at` as the deletion period; read original refund date from `refund_json`.

**Caveat:** the snapshot insert during delete runs under a swallowed `.catch(() => null)` (`invoices.ts:8354`). If it fails, the live row is still physically removed but no snapshot is written — so `deleted_refunds` can under-count real deletions. State this on the audit panel; do not present deleted counts as complete.

If shipped, label as "Deleted refund history" and gate it as audit data. Do not combine with Total Refunds.

## 18. Empty states

| State | Display rule |
|---|---|
| No refunds in period | "No refunds recorded in this period." |
| Future period | "Refunds are recorded after money goes out. No refunds are scheduled here." |
| No prior-period refunds | Show current amount and "No prior refunds"; avoid percentage. |
| No permission | Show restricted state. Do not show zeros. |
| Tenant not resolved | Keep amount in total; label tenant line "Tenant record removed." |
| Money trail missing | Keep amount in total; show "Unattributed" where attribution or fund source is needed. |

## 19. Drill behavior

Every amount opens the exact slice it describes.

| Tap target | Opens | Filter meaning |
|---|---|---|
| Total Refunds | Refund list | Live refunds in selected properties and duration. |
| Reason line | Refund list | Same refunds filtered by reason matching rule. |
| Tenant line | Refund list or tenant money page | Same refunds filtered by recipient. |
| Property line | Refund list | Same refunds filtered by property. |
| Mode line | Refund list | Same refunds filtered by payment method. |
| Largest Refund | Refund detail | Exact refund. |
| Active-tenant signal | Refund list | Same refunds filtered to active-tenant refunds. |
| Repeat-refund signal | Refund list | Same refunds filtered to tenants with more than one refund. |
| Deposit-held bridge | Deposit-held screen or filtered refund list | Deposit/caution refunds only. |
| Collections bridge | Collections screen | Linked paid bills with refund subtraction visible. |
| Cash-flow bridge | Cash-flow report | Same period, refunds shown as money out. |
| Deleted refund history | Audit view | Deleted refund history by deletion date; original refund date shown beside it. |

## 20. QA invariants

- Total Refunds = sum of Reason lines.
- Total Refunds = sum of Tenant lines, including Tenant record removed.
- Total Refunds = sum of Property lines in multi-property mode.
- Total Refunds = sum of Mode lines, including Other (2048 "Others", any old 2049, and any unmapped code).
- Processed By = Total Refunds only when Unattributed is included.
- Fund Source = Total Refunds only when Unattributed is included.
- Deposit Refunds for the period are a **subset** of the lifetime refund amount deposits-held subtracts (deposits-held is un-dated) — do **not** assert per-period equality.
- Refund impact on Net Collection equals the refund amount the collections query subtracts, but that subtraction is un-dated — a next-month refund reduces a paid bill's net today. Do not expect a 1:1 period tie.
- Cash-flow Refunds (-) must match live refunds for the same property and refund-date period, unless DA-07 has a known broader property/date rule.
- Deleted refund totals must not change Total Refunds.
- Expense entries must never increase DA-03 Total Refunds.

## Changelog

| Date | Version | Change |
|---|---|---|
| 2026-06-06 | v1.0 | Created ground-truth formula map. Locked live refund entry as the base unit, refund-date time behavior, reason/mode/property/tenant formulas, cross-screen bridges, deleted-refund separation, empty states, drill behavior, and QA invariants. |
| 2026-06-06 | v2.0 | Hardened against live code, owner decisions folded in. Fixed the inverted 2049 mode (Advance Credits, blocked at write — not a defensive legacy bucket). Corrected the base rule: `refunds` has no is_active/status/deleted; deletion is a hard DELETE, so the base query has no status filter and mirrors cash-flow. Folded in `refunds`/`deleted_refunds` schema, full refund-mode map (incl. 202 Cash by OTP), free-text `due_type` reason matching with the locked Rent `ILIKE '%rent%'` rule (excl. Renewal Charges), one-directional deposit-held + un-dated collections bridges, `refunded_by` always null, and the silent passbook skip. |
