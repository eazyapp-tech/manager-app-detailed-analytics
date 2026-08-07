---
title: DA-05 Discounts — Brief
date: 2026-06-06
tags: [rentok, brief, discounts, detailed-analytics]
status: Refreshed · product-locked
owner: Sanchay
time_budget: Open — cut order ranked by owner pain
---

> [!INFO] What this is
> A product brief for the Discounts detailed-analytics screen.
>
> It explains what the owner needs to understand about discounts before he reviews tenants, staff, or reports. It is not a build spec and not a formula sheet.

## 1. In One Line

A PG owner gives discounts to close bookings, keep tenants, fix service mistakes, or settle a one-off complaint.

Today each discount is saved, but the owner cannot easily see **how much revenue was actually given up**, who gave it, why it was given, who received it, and what discount value is still open.

So the bet is one thing: **make discount cost readable before it becomes invisible leakage.**

*"Kitna discount use hua, kisne diya, kis wajah se diya, aur kisko mila?"*

## 2. The Owner

This screen is for the owner when he wants to **diagnose discount cost**.

It is not the place to give a new discount. It is not the place to collect rent. It is not the tenant ledger. Those jobs happen elsewhere.

**Rajesh, primary user.** Owner of one PG. He opens this when revenue feels lower than expected, when staff have been offering discounts, or before month-end review.

**Priya, secondary user.** Multi-property owner. Her question is: *"Which property or manager is giving away too much?"*

Where this sits:

- the main screen says **money moved**
- the collections list says **which payments happened**
- the tenant ledger says **what happened for one tenant**
- this screen says **what shape discount cost has**

## 3. The Problem

Three pains, same root cause:

1. **The owner cannot tell how much revenue was actually given up.** A discount that was created but never used is not the same as a discount that reduced a payment.
2. **The owner cannot tell who is giving discounts.** One team member applying many discounts can quietly reduce revenue across many tenants.
3. **The owner cannot tell whether discounts are normal or a pattern.** A joining offer is different from repeated negotiation for the same tenant.

The screen should turn a scattered discount history into clear review lanes.

## 4. What Rajesh Does Today

He works through partial views:

- checks tenant credit history one tenant at a time
- opens collection receipts to see whether a discount touched the payment
- reads activity logs for new discount entries
- pulls a payments report at month-end
- asks staff why a tenant paid less than expected

## 5. What Must Ship

**Must ship. V1 is not V1 without these:**

- **Q1 — How much owner-funded discount was used in this period?** This is the main cost number.
- **Q2 — Who gave the used discounts?** Group by saved team-member name, with an Unknown row when the name is missing.
- **Q3 — Why were they given?** Group into plain reason buckets from the saved reason text.
- **Q4 — Which tenants received the most?** Rank tenants by used owner-funded discount amount.
- **Q5 — Which property is carrying the discount cost?** Required when the owner views more than one property.

**Good to ship if the cycle allows:**

- **Q6 — What discount value is still open?** Created, visible, unused, and not expired. This is not cost yet, but it can become cost.
- **Q7 — What discount value is affected by refunds or unclear payment links?** A review lane for rows that may make the used total hard to trust.

**Cut order if time runs short:**

1. Payments-list shortcut from a used-discount total. The owner can still reach the rows from the discount list.
2. Q7 review lane. The main number can ship only if QA proves no material mismatch for sampled properties.
3. Q6 open balance. It is useful planning context, but not the main cost story.
4. Property percentage against collections. Property amount must stay; the ratio can wait.

## 6. Product Decisions

**Used discount is the headline.** The main number is discount value that actually reduced a payment during the selected period. Created-but-unused discount is a separate open balance.

**Used date controls the main screen.** If the owner chooses This Month, the main views show discounts used this month, not discounts created this month.

**Created date controls open balance only.** Open balance is about discounts that exist and can still be used. It does not enter the cost total.

**Owner-funded is the main view.** RentOk-funded credits can appear as a separate neutral note, but they should not inflate the owner's revenue-loss number.

**Reason buckets are helpful, not perfect.** The current data has free-text reasons. The screen groups common words into Joining Offer, Negotiation, Maintenance Adjustment, Management Adjustment, and Other.

**Team-member names are shown as saved.** If the same person appears as "Jatin" and "Jatin K", V1 should not silently merge them.

**Deleted discounts do not count.** There is no live row after deletion, so deleted entries cannot be part of live totals.

**Refunds stay separate.** Refunds are money paid back after collection. Discounts are money not collected because a payment was reduced. A refunded payment can make a used discount hard to reason about, so it belongs in review, not in the refund total.

**Collections stays separate.** Collections shows money received. DA-05 shows discount value used on those payments.

## 7. What We Are Not Building This Cycle

- **No new discount form.** Creating discounts stays in the existing discount/tenant/payment paths.
- **No staff punishment view.** The page shows who gave discounts so the owner can review; it should not accuse people in the wording.
- **No fake category picker.** V1 reads saved reason text. A structured reason field can come later.
- **No settlement report replacement.** This page explains discount use. Settlement reports explain who funded money after payment settlement.
- **No expired-discount campaign tool.** Open discount value is a balance check, not a reminder system.
- **No invoice plan-side discounts.** Tenant-bill credits belong here. Internal subscription invoice discounts do not.

## 8. Traps And Risks

**Traps decided now:**

- **Created is not used.** Showing created discount as headline would overstate real revenue loss.
- **Open balance is not cost.** A discount can exist without reducing any payment yet.
- **RentOk-funded credit must not look like owner loss.** It may matter to the tenant, but it is not Rajesh giving up revenue.
- **Reason buckets will be noisy.** Other is not failure. It is honest about free text.
- **Team attribution is only as clean as the saved name.** V1 should show the saved name.
- **Deleted discounts are a trust risk.** If a team member can delete a discount without a clear permission and log, the screen can be gamed.
- **Every follow-up must match the number tapped.** If the owner opens a row and the list sum differs, trust breaks.

**Risks:**

| Risk | Read | Mitigation |
|---|---|---|
| Used total disagrees with payment reports | High | Pick and label the source clearly. QA must compare sampled rows before launch. |
| Owners confuse open balance with cost | Medium | Keep open balance secondary and label it as "can still be used." |
| Staff view feels accusatory | Medium | Use review language, not blame language. |
| Permissions leak discount history | High | Add a clear view permission or explicitly gate behind invoice view with owner approval. |
| Delete without audit hides patterns | High | Ship delete permission and delete log before enabling edit/delete actions from the new list. |

### Footer

**Things to test with owners before launch:**

- Do owners understand that the headline is used discount, not created discount?
- Do they understand open balance as "can still be used" rather than cost already lost?
- Do reason buckets match how they talk about discounts?
- Does the team-member view feel useful rather than accusatory?
- Does a multi-property owner prefer property amount first, or amount plus collection percentage?

**Locked decisions:**

- DA-05 is a diagnosis screen.
- Headline = owner-funded discount used in the selected period.
- Main date = date the discount was used on a payment.
- Open balance = visible, unused, not expired discount value as of now.
- Main cuts = team member, reason, tenant, property.
- Reason buckets = Joining Offer, Negotiation, Maintenance Adjustment, Management Adjustment, Other.
- RentOk-funded credit is separate and neutral.
- Deleted discounts do not count in live totals.

## Changelog

| Date | Version | Change |
|---|---|---|
| 2026-06-06 | v1.0 | Rebuilt as a self-contained product brief. Corrected headline from created discounts to used owner-funded discounts and separated open balance. |
