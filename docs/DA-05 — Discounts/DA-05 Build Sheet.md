---
title: DA-05 Discounts — Build Sheet
date: 2026-06-06
tags: [rentok, build-sheet, engineering, discounts, detailed-analytics]
status: Refreshed · ready for ticket slicing after engineering review
owner: Sanchay
companion_to: DA-05 Brief · DA-05 Ground-Truth Formula Map
---

> [!INFO] What this is
> Engineering build sheet for DA-05 Discounts.
>
> Product meaning lives in the Brief and Ground-Truth Formula Map. This sheet gives code anchors, endpoint shape, build order, QA, smoke tests, and launch blockers.

## 1. Product Lock

DA-05 is a Synthesis screen for discounts.

Locked product meaning:

- headline = owner-funded discounts actually used in the selected period,
- main date = `Credits.date_used`,
- open balance = owner-funded, visible, unused, unexpired credits as of now,
- RentOk-funded credit is separate,
- deleted credit rows are excluded from live totals,
- refunds and expenses are not discount totals,
- every row tap must open rows that add back to the tapped number.

Correctness pushback:

- Do not use `Credits.date_added` for the headline. Created credit is not revenue loss until used.
- Do not force DA-05 to match the old Payments Report owner-credit column without a product decision. That report recomputes the owner/RentOk split from settlement data; DA-05's row-level governance is based on credit rows.
- Do not repeat the older claim that all tenant credits become used through `linkCreditToPayment`. The broad update attaches a payment to all tenant/property credits, but `updateCreditData` flips `status` and `date_used` only for selected credit keys.

## 2. Codebase Facts Verified

| Fact | Evidence |
|---|---|
| Discounts live in `credits`, not a discount table. | [credits.ts](/Users/eazypg/rentok-backend/src/entities/credits.ts:11) |
| `Credits.amount` is a plain number column and current credit flows treat it as rupees. | [credits.ts](/Users/eazypg/rentok-backend/src/entities/credits.ts:60), [credits.ts](/Users/eazypg/rentok-backend/src/controllers/credits.ts:417) |
| `source` distinguishes owner vs RentOk-funded credit. | [credits.ts](/Users/eazypg/rentok-backend/src/entities/credits.ts:48) |
| `status` distinguishes unused vs used. | [credits.ts](/Users/eazypg/rentok-backend/src/entities/credits.ts:52) |
| `view_status` is the scratch/visibility state. | [credits.ts](/Users/eazypg/rentok-backend/src/entities/credits.ts:56) |
| `date_added` and `date_used` are separate timestamps. | [credits.ts](/Users/eazypg/rentok-backend/src/entities/credits.ts:29), [credits.ts](/Users/eazypg/rentok-backend/src/entities/credits.ts:42) |
| `payment_id` links a credit to a payment. | [credits.ts](/Users/eazypg/rentok-backend/src/entities/credits.ts:63), [credits.ts](/Users/eazypg/rentok-backend/src/entities/credits.ts:67) |
| Existing credit routes do not provide DA-05 list/widget/report endpoints. | [credits.ts](/Users/eazypg/rentok-backend/src/routes/credits.ts:120) |
| `fetchTenantCredits` returns all tenant credits and sums all of them, including used/expired rows. Do not reuse its total as open balance. | [credits.ts](/Users/eazypg/rentok-backend/src/controllers/credits.ts:197), [credits.ts](/Users/eazypg/rentok-backend/src/controllers/credits.ts:208) |
| `fetchCreditsToApply` is the closest current open-credit filter: tenant, property, unused, unexpired, visible. | [credits.ts](/Users/eazypg/rentok-backend/src/controllers/credits.ts:296) |
| `addCredits` creates owner-funded, unused, visible credits and saves issuer name as free text. | [credits.ts](/Users/eazypg/rentok-backend/src/controllers/credits.ts:482) |
| `addCredits` is gated by `record_payment`, not discount-specific permission. | [credits.ts](/Users/eazypg/rentok-backend/src/controllers/credits.ts:442) |
| `addCreditsFromTenantPhone` creates `Management discount` rows. | [credits.ts](/Users/eazypg/rentok-backend/src/controllers/credits.ts:574), [credits.ts](/Users/eazypg/rentok-backend/src/controllers/credits.ts:583) |
| `deleteCredits` hard-deletes unused owner credits and has no discount-specific permission check. | [credits.ts](/Users/eazypg/rentok-backend/src/controllers/credits.ts:246), [credits.ts](/Users/eazypg/rentok-backend/src/controllers/credits.ts:280) |
| `linkCreditToPayment` broadly attaches a payment to all credits for tenant+property. | [credits.ts](/Users/eazypg/rentok-backend/src/controllers/credits.ts:175), [credits.ts](/Users/eazypg/rentok-backend/src/controllers/credits.ts:181) |
| `updateCreditData` updates only selected credit keys to used and sets `date_used`. | [payments.ts](/Users/eazypg/rentok-backend/src/helpers/payments.ts:1247), [payments.ts](/Users/eazypg/rentok-backend/src/helpers/payments.ts:1318) |
| Partial credit use splits one credit into used and remaining rows. | [payments.ts](/Users/eazypg/rentok-backend/src/helpers/payments.ts:1327), [payments.ts](/Users/eazypg/rentok-backend/src/helpers/payments.ts:1334) |
| Payments store `credits_used`, `owner_credits`, and `eazypg_credits`. | [payments.ts](/Users/eazypg/rentok-backend/src/entities/payments.ts:40), [payments.ts](/Users/eazypg/rentok-backend/src/entities/payments.ts:76) |
| The list-screen pattern already exists for dues, collections, expenses, and rooms. | [index.ts](/Users/eazypg/rentok-backend/src/v1/index.ts:2), [routes.ts](/Users/eazypg/rentok-backend/src/v1/list_screens/expenses/routes.ts:8) |
| Current collection totals treat mode 213 specially by subtracting owner credits. | [helpers.ts](/Users/eazypg/rentok-backend/src/v1/list_screens/collections/helpers.ts:413), [helpers.ts](/Users/eazypg/rentok-backend/src/v1/list_screens/collections/helpers.ts:423) |
| Old Payments Report recomputes owner and RentOk credit from settlement math. | [reports.ts](/Users/eazypg/rentok-backend/src/controllers/reports.ts:1888), [reports.ts](/Users/eazypg/rentok-backend/src/controllers/reports.ts:1894) |
| `viewDiscounts`, `editDiscounts`, and `deleteDiscounts` do not exist in current permission projection. | [teamMemberProperty.ts](/Users/eazypg/rentok-backend/src/entities/teamMemberProperty.ts:70), [service.ts](/Users/eazypg/rentok-backend/src/v1/login/property/service.ts:80) |
| `checkAuth` can return true when request authentication was not set; use HeaderValidator and fail-closed DB checks for new routes. | [commonFunctions.ts](/Users/eazypg/rentok-backend/src/utils/commonFunctions.ts:1238), [commonFunctions.ts](/Users/eazypg/rentok-backend/src/utils/commonFunctions.ts:1259) |
| Invoice and tenant detail routes used by follow-up are currently missing HeaderValidator. | [invoices.ts](/Users/eazypg/rentok-backend/src/routes/invoices.ts:215), [tenant.ts](/Users/eazypg/rentok-backend/src/routes/tenant.ts:927) |
| No `DiscountFilterCode` range exists today. Current ranges: dues 1101, collections 1201, expenses 1301, rooms 1401. | [filterCodes.ts](/Users/eazypg/rentok-backend/src/v1/constants/filterCodes.ts:1), [filterCodes.ts](/Users/eazypg/rentok-backend/src/v1/constants/filterCodes.ts:37), [filterCodes.ts](/Users/eazypg/rentok-backend/src/v1/constants/filterCodes.ts:87), [filterCodes.ts](/Users/eazypg/rentok-backend/src/v1/constants/filterCodes.ts:115) |
| Collections already has a self-added tenant fallback. DA-05 must either match that rule or explicitly reject partial access. | [service.ts](/Users/eazypg/rentok-backend/src/v1/list_screens/collections/service.ts:23), [service.ts](/Users/eazypg/rentok-backend/src/v1/list_screens/collections/service.ts:106) |
| The v1 homepage financial block currently assembles collections, dues, expenses, and deposits. No discount item is present in that block. | [service.ts](/Users/eazypg/rentok-backend/src/v1/homepage/service.ts:2464), [service.ts](/Users/eazypg/rentok-backend/src/v1/homepage/service.ts:2650) |

## 3. Build Scope

### New V1 Module

Add a new list-screen module:

- `src/v1/list_screens/discounts/routes.ts`
- `src/v1/list_screens/discounts/controller.ts`
- `src/v1/list_screens/discounts/service.ts`
- `src/v1/list_screens/discounts/helpers.ts`
- `src/v1/list_screens/discounts/schemas.ts`

Register it under `/v1/discounts` in [index.ts](/Users/eazypg/rentok-backend/src/v1/index.ts:22).

Mirror the existing expenses module shape, but do not copy its permission gaps blindly.

### Endpoint Shape

| Endpoint | Method | Purpose | Auth |
|---|---:|---|---|
| `/v1/discounts/list/filters` | POST | Paginated discount rows for every follow-up. | HeaderValidator plus discount view gate |
| `/v1/discounts/list/widget` | GET | Top summary, issuer, reason, tenant, property, open value, review signals. | HeaderValidator plus discount view gate |
| `/v1/discounts/list/filter_options` | GET | Issuer, reason, tenant, property, source, status options. | HeaderValidator plus discount view gate |
| `/v1/discounts/list/report` | POST | Export the same filtered rows. Prefer v1 reports pattern over adding a legacy `/reports/*` endpoint. | HeaderValidator plus discount view gate |

Avoid adding `/discounts/advanced-details` in V1 unless the mobile app already needs a separate detail page. The list row can link to tenant passbook or collection detail.

### Permissions

Decision needed before coding:

| Option | Product read | Build read |
|---|---|---|
| New `view_discounts`, `edit_discounts`, `delete_discounts` DB fields and JWT keys | Best product fit. Owners can separate dues/collections access from discount governance. | Requires migration, signup defaults, login projection, team-member UI, route checks. |
| Reuse `viewInvoices` and `record_payment` | Fastest path. | Leaks discount visibility to anyone with invoice access and keeps delete governance unclear. |

Recommendation: build new `view_discounts` for the analytics page and `delete_discounts` before any delete action is exposed. Keep edit/create on existing paths until a discount-form refresh exists.

If V1 reuses invoice permissions, mirror the Collections self-added fallback or block restricted users with a clear 401. Do not show all discount rows to a team member who only has access to tenants they added.

## 4. Data Definitions

### Base Used Discount Query

Use this base for headline, issuer, reason, tenant, and property sections:

```sql
credits.source = 0
AND credits.status = 1
AND credits.date_used >= :period_start
AND credits.date_used <= :period_end
AND credits.property IN (:property_strings)
```

Also require live rows only. There is no soft-delete flag; hard-deleted rows are absent.

### Open Balance Query

Use this base for open balance:

```sql
credits.source = 0
AND credits.status = 0
AND credits.view_status = 101
AND (credits.expiry_date IS NULL OR credits.expiry_date >= :today_end)
AND credits.property IN (:property_strings)
```

Note: current `fetchCreditsToApply` requires `expiry_date >= now`, so rows with null expiry may be missed by old tenant-level code. DA-05 product meaning treats null expiry as not expired unless engineering proves null means invalid in production data.

### RentOk-Funded Used Query

```sql
credits.source = 1
AND credits.status = 1
AND credits.date_used BETWEEN :period_start AND :period_end
AND credits.property IN (:property_strings)
```

Show as separate, neutral context.

### Reason Buckets

| Bucket | Match |
|---|---|
| Joining Offer | `description ILIKE any joining/onboarding/move-in/new booking words` |
| Negotiation | `description ILIKE any negotiation/retain/retention/loyalty/stay-back words` |
| Maintenance Adjustment | `description ILIKE any maintenance/service/repair/water/ac/electricity/waiver/compensation words` |
| Management Adjustment | `description ILIKE '%management discount%' OR '%management%' OR '%manual adjustment%'` |
| Other | no match or blank |

Order matters. Apply first match in the table.

## 5. Widget Response Shape

Suggested response:

```ts
interface DiscountWidgetResponse {
  status: number;
  message: string;
  data: {
    period: {
      start_date: string;
      end_date: string;
      timezone: 'Asia/Kolkata';
    };
    summary: {
      owner_used_amount: number;
      owner_used_count: number;
      owner_used_tenant_count: number;
      prior_owner_used_amount: number;
      change_amount: number;
      change_percent: number | null;
      rentok_used_amount: number;
      rentok_used_count: number;
      open_owner_amount: number;
      open_owner_count: number;
      expiring_soon_amount: number;
      expiring_soon_count: number;
    };
    by_issuer: DiscountBreakdownRow[];
    by_reason: DiscountBreakdownRow[];
    by_tenant: DiscountTenantRow[];
    by_property: DiscountPropertyRow[];
    signals: DiscountSignal[];
  };
}
```

Keep money values in rupees to match `Credits.amount`. Do not divide by 100.

## 6. List Filters

Add `DiscountFilterCode` in a new range after rooms. Use `1501-1599` unless engineering wants to reserve that range for another module before coding.

Suggested filters:

| Code | Filter | Query |
|---:|---|---|
| 1501 | Owner-funded used in period | base used query |
| 1502 | RentOk-funded used in period | RentOk-funded used query |
| 1503 | Open owner-funded value | open balance query |
| 1504 | Expiring soon | open balance plus expiry inside next seven days |
| 1505 | Unknown issuer | base used query plus missing issuer |
| 1506 | Other reason | base used query plus no reason match |
| 1507 | Refund-touched payment | base used query plus linked payment has refund/refunded state |
| 1508 | Payment-link mismatch | review query where credit/payment link is unclear |

Issuer, reason, tenant, and property row taps can use explicit filter payload fields rather than allocating one code per value.

## 7. Build Order

| Order | Task | Done when |
|---:|---|---|
| 1 | Lock permission path. | Product and engineering choose new discount permission or reuse invoice permission. |
| 2 | Add module registration and schemas. | `/v1/discounts/*` routes compile and reject bad input. |
| 3 | Build base used query and list endpoint. | Filter 1501 returns rows and totals for selected period/properties. |
| 4 | Build widget summary. | Headline, count, tenant count, prior-period change, RentOk-funded used, and open balance return. |
| 5 | Build issuer/reason/tenant/property breakdowns. | Each breakdown adds back to headline. |
| 6 | Build open balance and expiring-soon logic. | Open balance ignores selected period and is labelled live. |
| 7 | Build review signals. | Unknown issuer, Other share, refund-touched, and payment-link mismatch are exposed. |
| 8 | Add report export. | Export uses the same filtered row service as list. |
| 9 | Harden cross-screen follow-up. | Any linked invoice/tenant destination used by DA-05 is authenticated. |
| 10 | QA reconciliation pass. | Sample properties match row sums and known payment/tenant records. |

## 8. QA Invariants

| Invariant | Check |
|---|---|
| Headline equals list sum | Sum row amounts from `/v1/discounts/list/filters` with owner-used filter equals widget `owner_used_amount`. |
| Issuer rows add back | Sum issuer row amounts equals headline, including Unknown. |
| Reason rows add back | Sum reason row amounts equals headline, including Other. |
| Tenant rows add back | Sum tenant row amounts equals headline, including Tenant record removed. |
| Property rows add back | Sum property row amounts equals headline for multi-property selection. |
| Open value stays separate | `open_owner_amount` is never added to `owner_used_amount`. |
| RentOk-funded stays separate | `rentok_used_amount` is never added to owner headline. |
| Previous period is same length | Current and previous windows have equal day count. |
| Missing prior period has no percent | If prior amount is zero, percent is null. |
| Hard-deleted rows absent | Deleted credits do not appear in live totals. |
| Rupee unit preserved | A known `Credits.amount = 500` displays as ₹500, not ₹5 or ₹50,000. |

## 9. Launch Blockers

| Blocker | Why it blocks |
|---|---|
| Permission decision unresolved | Discount history is sensitive. Do not ship unclear access. |
| Delete action exposed without `delete_discounts` and activity log | Staff can hide discount patterns. |
| Headline/list mismatch | Trust breaks immediately. |
| Reason/issuer/tenant/property sums do not add back | The page becomes a set of disagreeing numbers. |
| Cross-screen invoice or tenant follow-up remains unauthenticated | Discount rows can expose tenant/payment details through unsafe destinations. |
| Restricted user access is not defined | A self-added-only team member could see discounts for tenants outside their scope. |
| Refund-touched rows are silently mixed without review | A refunded payment can make the used-discount story misleading. |
| Credits amount unit not verified in QA | A 100x money error can ship. |

## 10. Smoke Tests

Replace placeholders before running.

### Widget

```bash
curl --request GET \
  'http://localhost:3000/v1/discounts/list/widget?pg_number_filter=1&filter_key=this_month' \
  --header 'Authorization: Bearer <TOKEN>'
```

Expected:

- `status` is 200.
- `data.summary.owner_used_amount` is a rupee number.
- `open_owner_amount` exists but is not added to owner used amount.
- If prior amount is zero, `change_percent` is null.

### Owner-Used List

```bash
curl --request POST \
  'http://localhost:3000/v1/discounts/list/filters' \
  --header 'Authorization: Bearer <TOKEN>' \
  --header 'Content-Type: application/json' \
  --data '{
    "limit": 20,
    "offset": 0,
    "filter_codes": [1501],
    "filter_key": "this_month",
    "pg_number_filter": [1]
  }'
```

Expected:

- Rows are owner-funded, used, and inside the selected period.
- Sum of returned rows plus pagination total matches widget headline.
- No RentOk-funded rows appear.

### Open Balance

```bash
curl --request POST \
  'http://localhost:3000/v1/discounts/list/filters' \
  --header 'Authorization: Bearer <TOKEN>' \
  --header 'Content-Type: application/json' \
  --data '{
    "limit": 20,
    "offset": 0,
    "filter_codes": [1503],
    "filter_key": "last_month",
    "pg_number_filter": [1]
  }'
```

Expected:

- Rows are unused, visible, owner-funded, and not expired.
- Results are labelled as live/open as of today, not last month.

## 11. Adjacent Boundaries

| Area | Boundary |
|---|---|
| Dues | DA-05 does not change unpaid-dues math. Open discounts may reduce future payable amount, but they are not unpaid dues. |
| Collections | DA-05 reads discounts used on payments. Collections remains money received. Do not add both as cash movement. |
| Refunds | Refunds are cash returned. A discount on a refunded payment is a review signal, not refund amount. |
| Expenses | Discounts are not spend records and do not belong in expense totals. |
| Occupancy | Joining discounts may relate to filling rooms, but DA-05 does not measure occupancy. |
| Reports | Use the same row service for DA-05 export. Treat old Payments Report settlement split as adjacent, not automatic source of truth. |
| Homescreen | There is no existing v1 financial Discounts item to rely on. Add a separate homescreen item only if product wants a signal layer; DA-05 itself remains the deeper diagnosis page. |

## 12. Remaining Engineering Questions

These need owner/engineering sign-off before tickets are final:

1. Build new discount permissions, or reuse invoice permissions for V1?
2. Should DA-05 reconcile to credit-row owner-funded meaning or settlement-funded meaning when old Payments Report disagrees?
3. Should null `expiry_date` count as never-expiring open value, or be treated as invalid data?
4. Should the report endpoint live under `/v1/discounts/list/report` or the legacy `/reports` namespace?
5. Should invoice and tenant destination auth be fixed inside DA-05 scope or as a shared security ticket before DA-05 launches?

## 13. Changelog

| Date | Version | Change |
|---|---|---|
| 2026-06-06 | v1.0 | Rebuilt against current code. Corrected used-date headline, separated open balance, replaced old endpoint assumptions, and added permission/security launch blockers. |
