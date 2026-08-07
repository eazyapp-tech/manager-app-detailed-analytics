---
title: DA-05 Discounts — Ground-Truth Formula Map
date: 2026-06-06
tags: [rentok, discounts, ground-truth, formula-map]
status: New · product-locked
owner: Sanchay
---

> [!INFO] What this is
> This document defines every number on the Discounts detailed-analytics screen.
>
> It explains what each number means, how time affects it, how follow-up works, and what must add back to what. Build details live in the Build Sheet.

## 1. The Screen In One Line

DA-05 answers: **how much owner-funded discount was actually used, who gave it, why it was given, who received it, and what discount value can still be used later.**

It is a discount-cost diagnosis page. It is not a collections page, refund page, expense page, or discount-entry form.

## 2. Rules That Apply To Every Number

1. **Used discount is the main cost number.** A discount counts as cost only after it reduces a payment.
2. **Created-but-unused discount is separate.** It is open value that can still reduce future payments. It is not counted as cost yet.
3. **Owner-funded discount is the main view.** RentOk-funded credit is shown separately when present.
4. **Deleted discounts do not count in live totals.** If a row was deleted, it is not part of the live discount picture.
5. **Reason and team-member names are only as clean as saved data.** V1 shows the saved values or honest buckets. It does not silently fix names.
6. **Every total must open the same matching rows.** If the user opens a number, the rows shown must add back to that number.
7. **Refunds do not become discounts.** A refund remains a refund. A used discount on a refunded payment may need review, but it does not move into the refund total.
8. **Collections do not become discounts.** Collections show money received. Discounts show money not collected because a payment was reduced.

## 3. Plain Meanings

| Term | Meaning |
|---|---|
| **Discount** | A saved value that can reduce what a tenant pays. |
| **Owner-funded discount** | Discount value paid by the owner through lower collected revenue. |
| **RentOk-funded credit** | Credit value funded by RentOk. It may reduce what the tenant pays, but it is not owner revenue loss. |
| **Used discount** | Discount value that actually reduced a payment. |
| **Open discount value** | Discount value that is visible, unused, and not expired. It can still reduce a future payment. |
| **Created discount** | A discount entry that was added for a tenant. Created does not mean used. |
| **Reason** | The saved text explaining why the discount was given, grouped into plain buckets. |
| **Issuer** | The saved team-member name on the discount. |
| **Recipient** | The tenant who received the discount. |
| **Deleted discount** | A discount entry removed from live records. It does not count in live totals. |

## 4. How Time Works

### Main Period

A used discount belongs to a selected period when the discount was used on a payment during that period.

Example:

| Selected period | Counts |
|---|---|
| 1 June to 30 June | Owner-funded discounts used from 1 June through 30 June |
| Today | Owner-funded discounts used today |
| This month | Owner-funded discounts used from the first day of this month through today or through the chosen end date |

### Previous Period

The previous period is the same length immediately before the selected period.

Example:

| Current period | Previous period |
|---|---|
| 1 June to 30 June | 2 May to 31 May |
| 10 June to 16 June | 3 June to 9 June |

If previous discount cost is zero, show the rupee change but do not show a percent change.

### Open Balance Time

Open discount value is always as of now. It does not change when the owner picks last month, because it answers: **what can still be used?**

## 5. Base Sets

### Main Used-Discount Set

Every period-based number starts with the same set:

- live discounts,
- owner-funded,
- already used,
- used inside the selected period,
- attached to selected property or properties.

Do not include:

- unused discounts,
- expired unused discounts,
- deleted discounts,
- RentOk-funded credits,
- internal plan discounts,
- KYC credits,
- wallet or reward entries,
- refunds,
- expenses.

### Open-Discount Set

Open balance starts with a different set:

- live discounts,
- owner-funded,
- visible to be used,
- not yet used,
- not expired,
- attached to selected property or properties.

Open balance is not added to used discount cost.

## 6. Top Summary

| Number | What it means | Empty state |
|---|---|---|
| **Owner-funded discount used** | Add owner-funded discount amounts that were used in the selected period. | No owner-funded discounts used in this period |
| **Used discount count** | Count matching used discount entries in the same set. | No discount entries used |
| **Tenant count** | Count distinct tenants in the same set. | Hide when count is zero |
| **Change vs previous period** | Current used discount amount minus previous matching-period amount. | Hide percent when previous amount is zero |
| **RentOk-funded credit used** | Add RentOk-funded credits used in the selected period. | Hide when zero |
| **Open owner-funded value** | Add owner-funded discount value that can still be used as of now. | Hide when zero or show "No open discount value" in detail |

Direction: more owner-funded discount used is worse for the owner in a revenue-loss view. Higher than the previous period should read as bad unless the design explains a known exception.

## 7. Issuer Breakdown

This view uses the selected period and the main used-discount set.

| Number | Plain formula | What it tells the owner |
|---|---|---|
| **Issuer amount** | Add used owner-funded discounts grouped by saved team-member name. | "Who gave discounts that were actually used." |
| **Issuer count** | Count used discount entries for that saved name. | "Was this one entry or a pattern." |
| **Issuer tenant count** | Count distinct tenants touched by that saved name. | "How widely this person gave discounts." |

Rules:

- Blank or missing names become Unknown.
- Do not merge spelling variants in V1.
- Sort by amount, biggest first.
- Issuer lines must add back to Owner-funded discount used, including Unknown.

## 8. Reason Breakdown

This view uses the selected period and the main used-discount set.

Reason comes from saved text. V1 groups common words into simple buckets.

| Reason | Plain matching rule | What it tells the owner |
|---|---|---|
| **Joining Offer** | Text points to joining, onboarding, move-in offer, or new-booking discount. | "Discount used to close or start a tenant." |
| **Negotiation** | Text points to negotiation, retention, loyalty, stay-back, or tenant asking for a lower amount. | "Discount used to keep or negotiate with a tenant." |
| **Maintenance Adjustment** | Text points to maintenance, service issue, repair, water, AC, electricity issue, waiver, or compensation. | "Discount used because service was not right." |
| **Management Adjustment** | Text points to management discount or manual owner adjustment. | "Discount entered as a broad management concession." |
| **Other** | Anything not matched above. | "Discounts that need review or better saved reasons." |

Rules:

- Reason lines must add back to Owner-funded discount used.
- Other must open to its rows.
- Do not hide Other just because it is large.
- If a discount matches more than one reason, use the first approved order: Joining Offer, Negotiation, Maintenance Adjustment, Management Adjustment, Other.

## 9. Tenant Breakdown

This view uses the selected period and the main used-discount set.

| Number | Plain formula | What it tells the owner |
|---|---|---|
| **Tenant amount** | Add used owner-funded discounts grouped by tenant. | "Who received the most discount value." |
| **Tenant count of entries** | Count used discount entries for that tenant. | "Was this one discount or several." |
| **Reason summary** | Show the biggest reason bucket for that tenant, or the top two when close. | "Why this tenant received discounts." |
| **Tenant state** | Match the tenant to the current tenant record where possible. | "Whether the tenant is active, booked, moved out, or missing from live tenant records." |

Tenant state labels:

| State | Meaning |
|---|---|
| **Active** | Tenant is currently living. |
| **Booking** | Tenant is booked but not yet an active resident. |
| **Moved Out** | Tenant has left. |
| **Tenant record removed** | The discount remains, but the tenant record cannot be matched. |

Tenant lines must add back to Owner-funded discount used, including Tenant record removed.

## 10. Property Breakdown

This view appears when the owner is viewing more than one property.

| Number | Plain formula | What it tells the owner |
|---|---|---|
| **Property amount** | Add used owner-funded discounts for one property. | "Which property gave up the most revenue through discounts." |
| **Property share** | Property discount amount divided by total selected-property discount amount. | "How much of the discount cost sits here." |
| **Property tenant count** | Count distinct tenants who used owner-funded discounts at that property. | "Is this one tenant or a wider property pattern." |

Property lines must add back to Owner-funded discount used.

If a collection percentage is shown, it must use gross collection for the same property and period. It is a comparison line, not part of discount math.

## 11. Open Discount Value

This view is always as of now.

| Number | Plain formula | What it tells the owner |
|---|---|---|
| **Open owner-funded value** | Add owner-funded discounts that are visible, unused, and not expired. | "This much discount can still reduce future collections." |
| **Open count** | Count those open discount entries. | "How many entries are still waiting." |
| **Expiring soon** | Add open value that expires within the next seven days. | "Discount value that may vanish soon." |
| **Open by tenant** | Group open value by tenant. | "Who can still use discount value." |

Open value must never add to Owner-funded discount used.

When a period other than now is selected, keep the open-value label clearly live: **open as of today**.

## 12. RentOk-Funded Credit

RentOk-funded credit is separate from owner-funded discount.

| Number | Plain formula | What it tells the owner |
|---|---|---|
| **RentOk-funded credit used** | Add RentOk-funded credits used in the selected period. | "Tenant got a credit, but the owner did not fund it." |
| **RentOk-funded count** | Count matching RentOk-funded used credit entries. | "How often RentOk-funded credit was used." |

Show this only when amount is above zero. Do not mix it into the owner-funded headline.

## 13. Review Signals

Signals are flags on the main used-discount set or on open value. They are not separate totals to add together.

| Signal | Plain formula | What it tells the owner |
|---|---|---|
| **Largest used discount** | Find the largest single used owner-funded discount in the selected period. | "The one entry to check first." |
| **Repeat tenant, same reason** | Same tenant has more than one used owner-funded discount with the same saved reason in the selected period. | "Possible repeated concession." |
| **High Other share** | Other reason bucket is a large share of used owner-funded discount. | "Saved reasons are too unclear to trust categories." |
| **Unknown issuer** | Used owner-funded discount has no saved issuer. | "The cost exists, but attribution is missing." |
| **Refund-touched payment** | A used discount is linked to a payment that also has a refund or refunded state. | "Review whether the discount still tells the right story after refund." |
| **Payment-link mismatch** | Discount row and payment row do not clearly point to the same used discount value. | "Review before trusting row-level payment follow-up." |

One discount can trigger more than one signal.

## 14. Follow-Up Rules

| Tap | Opens | Required match |
|---|---|---|
| Owner-funded discount used | Discount list | Same properties, same selected period, owner-funded, used |
| Issuer row | Discount list | Same used set plus selected issuer name |
| Reason row | Discount list | Same used set plus selected reason bucket |
| Tenant row | Tenant passbook or discount list | Same tenant and same used set; the visible sum must match the row |
| Property row | Discount list | Same used set plus selected property |
| Open owner-funded value | Discount list | Owner-funded, visible, unused, not expired, as of today |
| RentOk-funded credit used | Discount list | RentOk-funded, used, selected period |
| Refund-touched signal | Discount or collection review list | Same rows that triggered the signal |

The first screen after a tap must make the active filters visible.

## 15. Empty And Edge States

| State | Behavior |
|---|---|
| No used owner-funded discounts | Show "No owner-funded discounts used in this period." |
| No previous-period discount | Show current amount and "No prior discount use." Do not show percent. |
| Future-only selected period | Show zero used discounts and explain that future discounts are not counted until used. |
| Open value exists while used total is zero | Show used total as zero and open value separately. |
| Deleted discount | Excluded from live totals. Audit risk belongs in Build Sheet. |
| Missing issuer | Count under Unknown. |
| Missing reason | Count under Other. |
| Missing tenant match | Count under Tenant record removed. |
| Expired unused discount | Excluded from open value. It may be a future cleanup view, not V1 cost. |
| Refunded payment with used discount | Keep the discount in used totals, but flag for review until the refund rule is fixed. |

## 16. Changelog

| Date | Version | Change |
|---|---|---|
| 2026-06-06 | v1.0 | Created the DA-05 ground-truth map. Locked used owner-funded discount as the headline, separated open discount value, and defined follow-up and edge rules. |
