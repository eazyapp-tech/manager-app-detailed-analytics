---
title: DA-03 Refunds - Build Sheet
date: 2026-06-06
tags: [rentok, refunds, build-sheet, engineering, detailed-analytics]
status: Living document · v4.0
owner: Sanchay
companion_to: DA-03 Ground-Truth Formula Map
---

> [!INFO] What this is
> Engineering build sheet for DA-03 Refunds.
>
> Build against `[[DA-03 Ground-Truth Formula Map]]`. Do not re-define product meaning here.

## 1. Data foundation

Build one **base query** first. Every section reads from it. (Called "base query" throughout — §1, §3, §5.)

**Base row:** every row that exists in `refunds`, joined to its linked `invoices` row. The `refunds` table has **no** is_active / status / deleted column (`refunds.ts:5-44`), so there is **no status filter** — deletion is a hard `DELETE`, so a row's mere existence is the whole live-vs-deleted test.

**Time column:** `refunds.refund_date`.

**Property scope:** linked invoice `property`, shaped as `<pg_id>PG<pg_number>`.

**Amount:** `refunds.amount`. The stored amount is already in **rupees** — the existing path formats it directly as INR. So multiplying by 100 would make refunds show 100x too big. Do not divide or multiply by 100 in DA-03 unless a separate money-unit migration changes the data everywhere first.

**Permission baseline:** use `viewInvoices` unless product explicitly adds a new refund-view key (the `viewInvoices` reuse is net-new for refund read; no read-permission gate exists on refund data today). Create/delete already use DB-backed `add_refund_access` and `delete_refund` — note `add_refund_access` has a hardcoded `moms_pg_ids` bypass (`invoices.ts:7937`).

## 2. Code anchors

| Area | Anchor | Use |
|---|---|---|
| Refund entity | [refunds.ts](/Users/eazypg/rentok-backend/src/entities/refunds.ts#L5) | Confirms refund row fields and no status lifecycle. |
| Refund routes | [refunds.ts](/Users/eazypg/rentok-backend/src/routes/refunds.ts#L8) | Existing metadata, create, and detail routes only. |
| Refund create service | [refunds.ts](/Users/eazypg/rentok-backend/src/services/refunds/refunds.ts#L273) | Existing multi-invoice create orchestrator. |
| Refund final write | [invoices.ts](/Users/eazypg/rentok-backend/src/controllers/invoices.ts#L8133) | Saves refund row; sets `refunded_by_name`/`refunded_by_phone` only — never `refunded_by`. |
| Refund mode block | [invoices.ts](/Users/eazypg/rentok-backend/src/controllers/invoices.ts#L7950) | Blocks mode 2049 at write time. |
| Refund detail | [refunds.ts](/Users/eazypg/rentok-backend/src/services/refunds/refunds.ts#L488) | Existing detail response and deleted-snapshot fallback. |
| Detail auth gap | [routes/refunds.ts](/Users/eazypg/rentok-backend/src/routes/refunds.ts#L10) | HeaderValidator only; no route-level checkAuth. |
| Reimbursement duplicate route | [reimbursement.ts](/Users/eazypg/rentok-backend/src/routes/reimbursement.ts#L10) | Same advanced-details route shape from different controller. |
| Deleted refund snapshot | [deletedRefunds.ts](/Users/eazypg/rentok-backend/src/entities/deletedRefunds.ts#L9) | Audit history table. |
| Delete refund flow | [invoices.ts](/Users/eazypg/rentok-backend/src/controllers/invoices.ts#L8273) | Permission, snapshot, remove, passbook reversal. |
| Passbook refund write | [teamPassbook.ts](/Users/eazypg/rentok-backend/src/services/teamPassbook/teamPassbook.ts#L4315) | REFUND transaction write and swallowed error risk. |
| Passbook reversal | [teamPassbookRefundService.ts](/Users/eazypg/rentok-backend/src/services/teamPassbook/teamPassbookOperationFactories/teamPassbookRefundService.ts#L65) | REFUND_REVERSE creation. |
| Open PF for refunds | [teamPassbookReimbursement.ts](/Users/eazypg/rentok-backend/src/services/teamPassbook/teamPassbookOperationFactories/teamPassbookReimbursement.ts#L153) | Staff-fronted refund reimbursement source. |
| Collections refund subtraction | [helpers.ts](/Users/eazypg/rentok-backend/src/v1/list_screens/collections/helpers.ts#L713) | Net collection subtracts refunds. |
| Deposits held refund subtraction | [service.ts](/Users/eazypg/rentok-backend/src/v1/homepage/service.ts#L2469) | Deposits Held subtracts refunds and adjustments. |
| Cash-flow refund line | [generateCashFlowReport.ts](/Users/eazypg/rentok-backend/src/services/reports/generateCashFlowReport.ts#L261) | DA-07 reads refunds by refund date. |
| Expense deposit-refund risk | [expenses.ts](/Users/eazypg/rentok-backend/src/controllers/expenses.ts#L366) | Expense category can duplicate deposit refunds. |
| Existing list-screen pattern | [service.ts](/Users/eazypg/rentok-backend/src/v1/list_screens/expenses/service.ts#L19) | Mirror service/controller/helper/routes pattern. |
| Filter code ranges | [filterCodes.ts](/Users/eazypg/rentok-backend/src/v1/constants/filterCodes.ts#L1) | Existing 1100/1200/1300/1400 ranges; reserve refunds range. |
| Auth helper behavior | [commonFunctions.ts](/Users/eazypg/rentok-backend/src/utils/commonFunctions.ts#L1238) | `checkAuth` behavior for JWT permission keys. |
| Refund action permissions | [teamMemberProperty.ts](/Users/eazypg/rentok-backend/src/entities/teamMemberProperty.ts#L199) | DB columns for add/delete refund. |

## 3. Build order

| Step | Build | Why | Acceptance check |
|---|---|---|---|
| 1 | Create `src/v1/list_screens/refunds/` with routes, controller, service, helpers, schemas. | DA-03 has no current list-screen endpoint. | `POST /v1/list_screens/refunds/list/filters` and `GET /v1/list_screens/refunds/list/widget` exist and return empty success for no data. |
| 2 | Add refund filter codes. | Drill-down needs stable filter names. | Codes do not collide with Dues, Collections, Expenses, or Rooms. |
| 3 | Build base query. | All sections read from one base query. | For a property and period, `SUM(refunds.amount)` equals a direct DB query on `refunds` by `refund_date` (no status filter — none exists). Mirror cash-flow scope (`generateCashFlowReport.ts:261-272`). |
| 4 | Enforce view permission. | Refund data is financial and tenant-linked. | User without permission gets 401/403; owner gets data. |
| 5 | Add detail auth guard. | Existing refund detail route is HeaderValidator only. | User with wrong property or missing view permission cannot read a refund by UUID. |
| 6 | Build Total Refunds widget. | Core screen value. | Amount, refund count, tenant count, and prior-period comparison match formula map. |
| 7 | Build Reason Breakdown. | Owner needs to know why money went out. | Reason rows add back to Total Refunds. |
| 8 | Build Tenant Breakdown. | Owner needs to know who got money. | Tenant rows add back to Total Refunds, including unresolved tenant bucket. |
| 9 | Build Property Breakdown. | Multi-property owners need location of refund load. | Property rows add back to Total Refunds. Hidden for single property. |
| 10 | Build Mode Breakdown. | Owner needs money-out method. | Mode rows add back to Total Refunds. 2049 ("Advance Credits", blocked at write) appears only if old data has it; 202 "Cash by OTP" maps to Cash; any unmapped code falls into Other. |
| 11 | Build Review Signals. | Owner needs the first rows to check. | Largest, active-tenant, repeat, duplicate, and unattributed signals use same base pool. |
| 12 | Build Processed By. | Staffed properties need attribution. | Rows add back to total only with Unattributed included. |
| 13 | Build Fund Source. | Staff-fronted refunds and cash source matter. | Petty Cash, Tenant-held Funds, Staff Personal Money, plus Unattributed add back to total where passbook data exists. (Fund-source codes: **AF** = property/agent funds = Petty Cash; **PF** = personal funds = Staff Personal Money; **NPNAF** = non-personal-non-agent funds = Tenant-held Funds. See passbook factories under `teamPassbookOperationFactories/`.) |
| 14 | Build Deposit-held bridge. | Deposit refunds reduce DA-06 liability. | Deposit refund total equals amount subtracted from deposits-held formula for the same deposit invoices. |
| 15 | Build Collections bridge. | Refunds reduce net collections. | Linked collection view shows gross, refund, and net without implying the collection never happened. |
| 16 | Build Cash-flow bridge. | Refunds appear as money out in DA-07. | Same property and period matches DA-07 `Refunds (-)` unless DA-07 rule differs and is documented. |
| 17 | Add deleted-refund audit entry point if cycle allows. | Deleted refunds are history, not live totals. | Deleted refund count/amount never changes Total Refunds. |
| 18 | Add export parity only after widgets pass. | Export must not drift from mobile numbers. | CSV/XLSX uses the same service methods as the screen. |

## 4. Endpoint shape

| Endpoint | Method | Purpose | Notes |
|---|---|---|---|
| `/v1/list_screens/refunds/list/filters` | POST | Filtered refund list for drill-down. | Mirror existing list-screen pagination shape. |
| `/v1/list_screens/refunds/list/widget` | GET or POST | Screen widgets and breakdowns. | Prefer POST if date/property body matches other analytics screens. |
| `/v1/list_screens/refunds/list/filter_options` | GET | Reason, mode, property, tenant, staff filters. | Optional for V1 if static filters suffice. |
| `/refunds/advanced-details` | GET | Existing refund detail. | Add auth guard before using as drill destination. |

Request must carry selected property or property set, date range, filters, pagination, and sort.

## 5. Query rules

- Use parameterized queries.
- Do not use `SELECT *`.
- Preserve the existing amount unit from `refunds.amount`. Do not introduce floats or 100x conversion unless a separate source-table migration is approved.
- Filter by `refunds.refund_date`, inclusive start and end.
- Join invoices by `refunds.invoice_id = invoices.id`.
- Scope property through `invoices.property`.
- No status / is_active filter on `refunds` — none exists. Deleted refunds are already physically gone from the table (hard `DELETE`); read `deleted_refunds` only for the audit panel, never the base query.
- Keep expenses out of the base query.
- Add indexes only through new migrations. Do not edit old migrations.

Suggested index plan before production load:

| Query need | Candidate index |
|---|---|
| Period + invoice join | `refunds(refund_date, invoice_id)` |
| Invoice property join | existing invoice property indexes first; add only if explain plan needs it |
| Deleted audit by date | `deleted_refunds(created_at)` if audit ships |

## 6. Matching rules

| Area | Rule |
|---|---|
| Reason | `due_type` is uncontrolled free text — match case-insensitive (`ILIKE`), not exact-equals. Deposit (`'Security Deposit'`), Caution Money (`'Caution Money'`), Advance (bill type `'Advance'`, **not** mode 2049). **Rent = `ILIKE '%rent%'` excluding 'Renewal Charges'** (owner-locked; catches 'May Rent', 'Pending July Rent', etc.). Late Fine (`'%late%'`), Utility/Service (`'%maint%'` incl. typo "Maintanance", Electricity*, Water, Food/Meals/Mess). Goodwill/Manual is unbuilt — route to Other until a prod `refund_remarks` phrase-pull is done. See Formula Map §7 for the full variant list. |
| Tenant | Resolve invoice `payer` (firebase_id) to tenant in the same property. Prefer active, then booking, then moved-out. |
| Property | Use linked invoice property string. |
| Mode | 2040 Cash + 202 "Cash by OTP" → Cash; 2041-2044 Digital; 2045 Bank; 2046 Card Machine; 2047 Cheque; 2048 "Others" → Other; 2049 "Advance Credits" (blocked at write, old data only) → Other; any unmapped code → Other. |
| Processed By | Prefer passbook team link when available; otherwise refund-by fields; otherwise Unattributed. |
| Fund Source | Parse passbook transactions linked by refund UUID and `CATEGORY_MAP.REFUND` / `REFUND_REVERSE`. |
| Deleted audit | Use `deleted_refunds.created_at` as deletion date, `refund_json.refund_date` as original refund date, plus REFUND_REVERSE trail. Never merge with live refunds. |

## 7. QA checklist

- Total equals direct sum of `refunds.amount` by property set and refund-date range (no status filter — none exists; mirrors cash-flow scope).
- Reason rows add back to total.
- Tenant rows add back to total.
- Property rows add back to total in multi-property mode.
- Mode rows add back to total.
- Processed By rows add back only when Unattributed is included.
- Fund Source rows add back only when Unattributed is included.
- Deleted refunds do not enter live total.
- Expense rows do not enter live total.
- Deposit refunds for the period are a subset of (not equal to) the lifetime amount deposits-held subtracts — deposits-held refund subtraction is un-dated. Do not assert per-period equality.
- Refunds reduce linked net collections, but the collections refund subtraction is un-dated (`collections/helpers.ts:713-728`): a next-month refund reduces a paid bill's net today. No 1:1 period tie.
- Cash-flow `Refunds (-)` matches the base 1:1 (same refund_date + property scope, no status filter) — this is the bridge DA-03's base mirrors.
- No prior-period percentage is shown when prior total is zero.
- Future period shows empty state, not projected refunds.
- Restricted user sees restricted state, not zero data.
- Detail route denies cross-property read by UUID.

## 8. Smoke tests

Use Postman-importable curls after local auth token is available.

```bash
curl -X POST "$BASE_URL/v1/list_screens/refunds/list/filters" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "property_id": "PROPERTY_UUID",
    "pg_id": "PG_ID",
    "start_date": "2026-06-01",
    "end_date": "2026-06-30",
    "limit": 20,
    "offset": 0
  }'
```

```bash
curl -X POST "$BASE_URL/v1/list_screens/refunds/list/widget" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "property_id": "PROPERTY_UUID",
    "pg_id": "PG_ID",
    "start_date": "2026-06-01",
    "end_date": "2026-06-30"
  }'
```

```bash
curl -X GET "$BASE_URL/refunds/advanced-details?refund_uuid=REFUND_UUID" \
  -H "Authorization: Bearer $TOKEN"
```

Manual DB checks:

```sql
SELECT COALESCE(SUM(r.amount), 0) AS total_refunds, COUNT(*) AS refund_count
FROM refunds r
JOIN invoices i ON i.id = r.invoice_id
WHERE i.property = $1
  AND r.refund_date::date BETWEEN $2::date AND $3::date;
```

```sql
SELECT i.due_type, COALESCE(SUM(r.amount), 0) AS amount
FROM refunds r
JOIN invoices i ON i.id = r.invoice_id
WHERE i.property = $1
  AND r.refund_date::date BETWEEN $2::date AND $3::date
GROUP BY i.due_type
ORDER BY amount DESC;
```

## 9. Launch blockers

| Blocker | Why it blocks | Fix |
|---|---|---|
| Refund detail lacks explicit permission check | The `/advanced-details` route is HeaderValidator only — no `checkAuth` (`routes/refunds.ts:10`). Any caller could read a refund by UUID. No read-permission gate exists on refund data today. | Add permission guard to refund detail and the duplicate reimbursement detail path. |
| Base total does not match direct DB sum | Trust failure. | Stop all UI work until base query matches. |
| Deleted refunds included in Total Refunds | Wrong product definition. | Split deleted audit from live query. |
| Expense deposit refunds included in Total Refunds | Double count. | Keep expense table out of base query; only use for cross-screen warning. |
| Deposit-held bridge disagrees with DA-06 | Cross-screen trust failure. | Reuse or match DA-06 deposits-held formula. |
| Collections bridge uses gross as net | Owner sees contradictory money. | Show gross, refund, and net separately. |
| Passbook attribution missing without Unattributed row | Hidden data loss. | Include Unattributed or hide attribution section. |
| 2049 shown as a normal mode | 2049 = "Advance Credits", blocked at write today (`invoices.ts:7951-7955`). It is a real disabled mode, not a defensive legacy bucket. | Bucket into Other; label "Advance Credits (disabled)"; appears only in old data predating the block. |
| Amount unit converted incorrectly | Cause: stored `refunds.amount` is already in rupees. Effect: multiplying by 100 makes refunds show 100x too big. | Preserve existing unit; treat any paise standardization as a separate migration. |

## 10. Implementation notes

- Keep service methods small: base query, breakdown mappers, signal builders, response formatter.
- Do not build a refund lifecycle state machine in DA-03.
- Do not introduce gateway-refund fields in this build.
- Do not add a new permission key unless product explicitly chooses it; document every frontend/backend touchpoint if added.
- Do not use passbook as the source of truth for the refund amount. Passbook is attribution and fund source.
- Treat `deleted_refunds.refund_json` as audit data; JSONB querying may need a later index plan if audit usage grows.

## Changelog

| Date | Version | Change |
|---|---|---|
| 2026-06-06 | v3.0 | Rebuilt against the new formula map. Added code anchors, build order, endpoint shape, query rules, matching rules, QA, smoke tests, launch blockers, and correctness pushback on pending/deleted/expense double-count scope. |
| 2026-06-06 | v4.0 | Hardened against live code, owner decisions folded in. Fixed inverted 2049 (Advance Credits, blocked at write — not defensive legacy). Removed all "active row" framing: `refunds` has no is_active/status/deleted, deletion is hard DELETE, base query mirrors cash-flow with no status filter. Folded in free-text `due_type` matching + locked Rent `ILIKE '%rent%'` rule, 202 Cash by OTP, anchored AF/PF/NPNAF fund codes, one base-query name, cause→effect 100x split, `refunded_by` always null, `moms_pg_ids` add-refund bypass, and the un-dated deposit/collections bridge caveats. |
