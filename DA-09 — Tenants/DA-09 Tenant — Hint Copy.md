# DA-09 Tenant, Hint Copy

The words behind every (i) on the Tenant tab. Two columns, matching the two text fields the sheet has: **What is this?** and **How is this calculated?** A dash means the definition already says everything. **Good to know** is one line per block, for the block-level line when the sheet gets one.

Checked against the code at rentok-backend `0e8cc713b` and the live account, Mumbai group, 22 properties, 27 Aug.

## Most of this tab is right now

The date filter at the top changes only a handful of numbers: Notices Raised, Move-ins and Move-outs, the Bookings funnel, and Renewals completed. Everything else on this tab is a live count of today and ignores the filter. Ten blocks are silent about this on their faces (finding F92); the sheets below say it instead.

The filter options are the same five as Dues: This Month is the 1st to today, Last Month is whole, Current FY is April onward, All Time, and Custom.

## 1. Overview Snapshot

Nine tiles. Two of them ship without any definition today (finding F94); both are written here.

> **Good to know:** everything here is right now except Notices Raised, which counts the dates you pick.

| Tile | What is this? | How is this calculated? |
|---|---|---|
| Active Tenants | Tenants living in the property right now. | - |
| Active Bookings | Bookings not yet moved in, pending or approved. | - |
| Approved Bookings | Bookings a manager has approved. | - |
| Eviction Pending | Tenants who asked to leave, still pending approval. | - |
| Eviction Approved | Tenants whose leaving is approved, with a confirmed date. | - |
| Notices Raised | Notices given in the dates you pick. | The one tile here that follows the date filter. |
| Past Their Date | Tenants whose approved eviction date has passed, not yet moved out. | - |
| Rent at risk | The monthly rent of tenants who are leaving. | Counts tenants with a notice, pending or approved. Uses the rent on each tenant's profile, not billed amounts. |
| Leaving with dues | Money still owed by tenants who are leaving. | Worked out on its own, so it can differ slightly from the Dues tab. |

The shipped Active Bookings sentence says "Confirmed bookings"; the tile counts pending and approved together, so the word confirmed goes.

## 2. Move-in & Move-out Metrics

> **Good to know:** Move-ins minus Move-outs is the net change in tenants.

| Card | What is this? | How is this calculated? |
|---|---|---|
| Move-ins | Tenants who moved in during these dates. | - |
| Move-outs | Tenants who moved out during these dates. | Counted by the approved leaving date, else the checkout date. |
| Net change | Move-ins minus move-outs. | - |

## 3. Journey

| Tab | What is this? | How is this calculated? |
|---|---|---|
| Tenants | Where tenants stand right now: active, eviction pending or approved, renewals due, agreements expired. | Each bar's share is out of Active Tenants. |
| Bookings | The booking funnel for these dates: total, approved, cancelled, converted. | - |

## 4. Tenant Verification

> **Good to know:** both splits cover every active tenant, so each adds up to Active Tenants.

| Tab | What is this? | How is this calculated? |
|---|---|---|
| ID Verification | Active tenants by ID check: e-KYC, verified by hand, or not verified. | - |
| Police Verification | Active tenants by police verification: done, still in time, or overdue. | - |

## 5. Tenancy Details

> **Good to know:** both tabs cover every active tenant.

| Tab | What is this? | How is this calculated? |
|---|---|---|
| Rent Agreement | Active tenants by how the agreement was signed: stamp, simple, uploaded, or not signed. | - |
| Profile Completion | Active tenants with a complete profile, against those without. | - |

## 6. Upcoming Eviction

> **Good to know:** the approved bars add up to the Eviction Approved tile above.

| Card | What is this? | How is this calculated? |
|---|---|---|
| Upcoming evictions | Tenants leaving, grouped by eviction date; pending and approved shown separately. | Always right now, whatever dates you pick. |

## 7. Agreement Expiry

| Card | What is this? | How is this calculated? |
|---|---|---|
| Agreements expiring | Long-term agreements: already expired, ending in 30, 60 or 90 days, or valid. | Where no agreement length is recorded, 11 months is assumed, and those tenants can show as Already expired. |

## 8. Tenant Profile

> **Good to know:** each tab says how many tenants have this recorded; the split covers only them.

| Tab | What is this? | How is this calculated? |
|---|---|---|
| Gender | Tenants by gender, of those who have it recorded. | - |
| Age | Tenants by age band, of those recorded. | - |
| City | Tenants by home city, of those recorded. | The biggest few are shown; Others opens the full list. |

## 9. Tenant Details

> **Good to know:** each tab says how many tenants have this recorded; the split covers only them.

| Tab | What is this? | How is this calculated? |
|---|---|---|
| Food Pref | Tenants by food choice, of those recorded. | - |
| Tenant Type | Tenants by type: student, working, and the rest. | - |
| Institute | Tenants by college or company, of those recorded. | Others opens the full list. |

## 10. Renewal & Retention

| Card | What is this? | How is this calculated? |
|---|---|---|
| Renewal Due | Agreements due for renewal in the next 30 days. | Always right now. |
| Completed | Renewals done in the dates you pick. | - |

## 11. Stay Type

| Card | What is this? | How is this calculated? |
|---|---|---|
| Stay type | Tenants living here now, short term against long term. | A tenant not marked short term counts as long term. |

## 12. Renting Type

| Card | What is this? | How is this calculated? |
|---|---|---|
| Renting type | Tenants renting as a company (B2B) or as an individual (Residential). | Shows only when renting type is recorded; hidden otherwise. |

## 13. Tenure

| Card | What is this? | How is this calculated? |
|---|---|---|
| Tenure | How long current tenants have stayed. | - |

## 14. Property Wise Active Tenants

| Card | What is this? | How is this calculated? |
|---|---|---|
| Active tenants by property | Active tenants per property, most first. | Needs two or more properties. |

---

## The pointers are checked, not guessed

Read off the live account on 27 Aug, This Month selected.

| Claim | Checked |
|---|---|
| ID Verification adds up to Active Tenants | 1080 + 37 + 102 = 1219 |
| Rent Agreement adds up to Active Tenants | 1107 + 112 = 1219 |
| The eviction bars add to the Eviction Approved tile | 59 + 35 + 43 + 16 = 153, matching the tile |
| Journey shares are out of Active Tenants | Active 1219 shows 100%; Approved Eviction 153 shows 13% |
| A CTA already ships on this tab | "51 recent joiners overdue on police verification · View list" renders live, the pattern the other tabs' pointers can reuse |

## Launch risks on this tab

1. **Approved Bookings reads 233 beside Active Bookings 60 on the live account.** Approved higher than total, on the first block of the tab. F93: on auto-accept properties every added tenant counts as an approved booking. Nothing the sheet says can make 233 against 60 look sane.
2. **Three quarters of the Already expired bar rests on the 11-month assumption** (F100). The biggest, reddest bar on most properties is mostly invented dates. D26 already ruled the fix: those tenants become their own "No term recorded" group. The copy above describes today and flips when D26 lands.
3. **One junk property setting blanks the whole tab** (F90).
4. **A cancelled booking counts as a move-out** (F95), 36 of July's move-outs on production.
5. **"Under Notice" means the pending half on one block and both halves on another** (F96, F99): one phrase, two populations, same screen.
6. **Recorded answers that fit no bucket are dropped or mislabelled** against the coverage line's promise (F104).

**Cells that flip when fixes land:** Approved Bookings stops counting auto-accepted tenants (F93); Agreement Expiry gains the "No term recorded" group and loses its assumption clause (D26); Leaving with dues drops its mismatch clause when it reads the shared dues figure.
