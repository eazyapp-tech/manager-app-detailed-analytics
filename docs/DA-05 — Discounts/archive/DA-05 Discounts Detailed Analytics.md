---
title: DA-05 Discounts Detailed Analytics
date: '2026-05-07'
tags:
  - rentok
  - prd
  - spec
  - detailed-analytics
  - discounts
  - financials
aliases:
  - DA-05
  - Discounts Detailed Analytics
---
> [!INFO] Source of Truth
> This note is the canonical spec for the Discounts Detailed Insights screen.
> **Engineers:** see [[DA-05 Build Sheet]] (9-column ticket-ready format) for implementation. This doc is the canonical "why."
> **Supporting refs:** [[_Ground Truth Field Map]] · [[_Build Sheet Generation Spec]]
> Supersedes: `RentOk Manager Homescreen/Detailed Analytics/PRD — Financial • Discount Insights.md`
> Related: [[DA-01 Dues Detailed Analytics]] · [[DA-02 Collections Detailed Analytics]] · [[DA-03 Refunds Detailed Analytics]] · [[DA-04 Expenses Detailed Analytics]]

# Discounts — Detailed Analytics

> **Product:** RentOk Manager App → Financial Insights → Discounts
> **Version:** 1.4
> **Status:** Final — Locked (codebase + 2 operator audit rounds + Figma audit 2026-05-07)
> **Last Updated:** 2026-05-07
> **Supersedes:** Old PRD "Financial • Discount Insights" (local file)
> **Design Reference:** Figma `KgBQXiT7r7oGrcqZHCWxyU` → Financials → Discounts (node ID `7837-100127`)
> **Parent Spec:** Homescreen Financial Insights container

---

## The One Thing This Screen Does

It answers: **"Kitna discount diya, kisko diya, kyun?"**
("How much discount did I give, who got it, why?")

Discounts is a **revenue-given-up view**. Every entry on this screen represents revenue the operator chose not to collect — money that would otherwise be on the books. Unlike Refunds (money paid out after collection), Discounts are reductions before/at collection. They show up here regardless of whether the discount was funded by the operator (Owner) or by RentOk as a platform offer (eazypg).

> **Why this distinction matters:** A ₹2,000 owner-funded discount is real revenue lost. A ₹2,000 RentOk-funded promotional discount is RentOk's marketing cost — your bottom line is unaffected. The screen splits these explicitly.

---

## A Morning with Rajesh

It's the 7th of May. Rajesh manages two PGs in Pune. He checks the app.

Discounts shows ₹15,000 given this month. ▲22% vs ₹12,300 in April — red, more revenue given up. He taps Reason. Joining Offer — ₹8,000 across 3 new tenants. Negotiation — ₹4,000 to one tenant (Rahul, who threatened to leave). Maintenance Adjustment — ₹3,000 across 4 tenants whose AC was broken last week. Makes sense — he approved each one.

Below: a repeat-tenant callout: *"Pawan Sharma got 3 discounts this month totalling ₹4,500. Tap to review."* Rajesh taps. Sees three separate ₹1,500 entries — all manually entered by Shiv with the description "Diwali offer." Should have ended in April. Rajesh deletes the May ones and notes to remind Shiv. Real ₹1,500 back on the books.

A green-funded note also: *"₹2,000 of discounts this month came from RentOk offers. No impact on your revenue."* He nods. Done in 2 minutes.

---

## A Morning with Priya

It's May 14. Priya manages 3 PGs in Mumbai.

Discounts shows ₹38,000 given this month. By Property: Property 1 — ₹8K (3% of collections), Property 2 — ₹24K (12% of collections), Property 3 — ₹6K (2% of collections). Property 2 is at 12% discount-to-collection — flagged amber. By Issuer: 80% of Property 2's discounts were applied by Manager Aakash. She's seen this before — he over-discounts to fill rooms quickly.

She also sees: 5 tenants at Property 2 each received multiple discounts this month — all manually added by Aakash. Could be legitimate (split bills, service issues) or could be over-discounting. She flags it for a 1:1.

Done in 4 minutes. Without this screen, she'd have seen a ₹38K total and called it normal.

---

## Scope

**This section shows discounts the operator (or their team) has applied to tenant bills.**

| Included | Excluded |
|----------|----------|
| Owner-funded discounts (`Credits.source = 0`) | RentOk plan-side discounts (`invoice_meta_data.discount_value` — internal subscription pricing) |
| RentOk-funded discounts (`Credits.source = 1`) — shown but flagged as "no revenue impact" | Refunds (see DA-03) — money paid out, not concession at billing |
| Joining offers (`Room.fixed_discount` → Credit at tenant creation) | Future-dated discounts (use the date the discount was applied) |
| Joining offer / Negotiation / Maintenance Adjustment / Other discounts (4-category locked taxonomy) | Deleted discounts (hard-deleted from Credits, no soft-delete) |
| Repeat discounts (same tenant, multiple entries in period — operator-driven repetition) | Dead schema: `Invoices.discount`, `tenant_packages.discount`, `Dues.discount` (all commented out / unused) |
| Bulk-issued discount cohorts | True recurring/auto-monthly discounts (the codebase does not support this — see Phase 2) |
| Bulk-issued discounts (via `addBulkCreditsFromPhone`) | KYC credits (`instaveritas_credits` — separate domain) |
| Manual discount entries via `addCredits` | Discount on RentOk Plans subscription invoices (internal pricing, not tenant-facing) |

> **Why "RentOk-funded" is included but flagged:** If you give a tenant ₹2,000 off and RentOk eats the cost, your tenant did get a discount — it shows on your records. But your revenue isn't affected. The screen surfaces the source split so operators don't double-count or misattribute.

---

## The Four Rules

1. **A discount entry = revenue you chose not to collect.** Every row represents a `Credits` record where the operator (or system on their behalf) reduced what a tenant owed. Whether it's a one-time offer, a manual repeat discount, or a joining sweetener — it's in scope.

2. **Source matters more than amount.** The single biggest split is `Owner` vs `RentOk-funded`. Owner-funded discounts hit your P&L. RentOk-funded discounts don't. Every total on this screen splits both — a single "Total" without source breakdown is misleading.

3. **Application date governs.** Filters use `Credits.date_used` for redeemed discounts and `Credits.date_added` for issued-but-unredeemed discounts. The hero defaults to "applied this period" (date_used). The lifecycle accordion separates issued, used, and expired views.

4. **Discounts are a revenue-leak view — up is bad.** Same color logic as Refunds and Expenses. Increase = red. Decrease = green. A discount spike means more revenue given up.

---

## How Rajesh Gets Here

`Homescreen → Overview → "View Detailed Analytics" → Financial Insights → Discounts tab`

Discounts is the fifth tab in Financial Insights (Dues · Collections · Refunds · Expenses · **Discounts**).

---

## What Rajesh Sees

### 1. The Header

**"Discounts"** — with a small ⓘ icon.

No "(Live)" suffix — Discounts is always period-based. Active period (e.g., "May 1–7") is visible in the time filter chip above.

Tapping ⓘ shows:
> "Shows all discounts applied to tenant bills during the selected period. Owner-funded discounts reduce your revenue. RentOk-funded discounts don't — they're shown separately. An increase vs last month is shown in red — more discount means less revenue."

---

### 2. Hero KPI — Total Discounts Given

**The question this answers:** "How much revenue did I give up this period?"

A large primary number — **₹13,000** — with subtitle: *(6 discounts · 5 tenants).*

This is **owner-funded discounts only** — what came out of your revenue. It's the number operators care about and reason with.

**RentOk-funded chip (conditional — only when amount > 0, below hero):**
*"+ ₹2,000 RentOk-funded · no impact on your revenue."*
A small neutral chip. Hidden when ₹0 (which it will be for most operators most months). When it appears, tappable → worklist filtered to RentOk-funded discounts.

> **Why single primary number with conditional chip:** Most operators have zero RentOk-funded discounts in a given period. A permanently-rendered "RentOk: ₹0" second number trains operators to ignore it, then they miss it on the months it matters. Single primary + conditional chip matches DA-03/DA-04 hero pattern and surfaces the source split exactly when it's relevant.

> **Hero asymmetry rule (v1.2):** When `RentOk-funded > Owner-funded × 2` (rare — happens during RentOk festival campaigns), promote BOTH numbers to co-primary at hero scale. The default "single primary + small chip" pattern visually misleads when the chip is bigger than the primary. Detection at render-time: if RentOk amount > Owner amount × 2 AND RentOk amount > ₹5,000, render two co-primary numbers stacked.

**MoM comparison chip:** ▲22% vs ₹10,650 Apr → **red** (more discount given). ▼ → green.
- Show **both** % AND prior absolute: *"▲22% vs ₹10,650 Apr"*. Operators reason in rupees.
- **Partial-month rule:** If today is May 7th, "vs April" is 7 days vs 30 days — meaningless. Compare same elapsed days: *"▲22% vs Apr 1–7"*. Tooltip explains.
- **Prior total = ₹0:** *"Not enough prior data"*.

**Outlier callout (conditional, below MoM chip):**
If the largest single owner-funded discount in the period is **≥ 40% of owner total AND ≥ ₹3,000**:
→ *"Largest: ₹4,000 to Rahul Sharma · May 3 · Negotiation. Tap to view."* — opens detail.

> **Threshold rationale (recalibrated v1.1):** Earlier draft used `≥30% OR ≥₹5,000`. The OR fired on tiny totals (₹2K out of ₹6K = 33% triggered alert for routine ₹2K joining offers). Switching to AND with raised floor catches genuine outliers without alarm fatigue.

**Repeat-tenant callout (conditional, below outlier):**
Fires when a tenant has **≥ 2 discounts with the same description (case-insensitive match)**:

> **v1.2 tightening:** Earlier v1.1 also fired on "≥ 3 discounts any-description." Operator audit caught the regression: legitimate split-bill cases (joining + maintenance + festival = 3 entries with different descriptions) triggered the alert. Dropped the any-description branch; kept only same-description match (the actual leakage signal).
→ *"⚠️ Pawan Sharma got 3 discounts totalling ₹4,500 — all 'Diwali offer'. Tap to review."*
CTA: "Review" → worklist filtered to that tenant's discounts.

> **Threshold rationale (recalibrated v1.1):** Earlier `≥ 2 entries` fired for legitimate split bills (joining + maintenance comp = 2 entries). The same-description trigger catches the actual leakage pattern (manual re-issuance) — which is what the Rajesh narrative describes.

**Unused-discounts chip (conditional, below repeat callout — replaces dropped Lifecycle section):**
If any unredeemed discounts exist (issued or scratched, not yet expired):
→ *"3 unused discounts ready to apply · ₹3,000."* Tap → worklist filtered to unredeemed.
Plus expiring-soon variant: if any expire within 7 days → *"⚠️ ₹2,500 in discounts will expire this week."*

> **Why this replaces the Lifecycle section:** 99% of PG discounts are inline-at-billing — the Lifecycle accordion would show Used: 100% / Issued: 0 / Scratched: 0 / Expired: 0 forever. The chip surfaces the rare case (campaign-runners, joining-offer-issuers with advance credits) without permanently consuming screen real estate.

**Net Collection impact (always visible, small text below):**
*"This reduced Collections by ₹13,000 this month."* Links to DA-02.

The hero is tappable → opens worklist filtered to owner-funded discounts in the period.

**Multi-property (Priya):** Subtitle: *(6 discounts · 5 tenants · across 3 properties).*

---

### 3. By Property (multi-property only — appears here, before Type)

**The question this answers:** "Which property is giving up the most revenue?"

For operators with >1 selected property, this section appears immediately below the hero — before all other breakdowns. Single-property operators don't see this section.

Each row contains:
- Property name
- Owner-funded discount total
- **Discount-to-collection ratio** (owner-funded discount ÷ DA-02 **GROSS collection** in same period — before refund deduction) — e.g., `12% of collections`. **Denominator is gross, not net** — using net would be self-referential since DA-02 net already accounts for cross-screen reductions. Same convention as DA-03's refund-to-collection ratio.
- Discount count + distinct tenant count: `7 discounts · 5 tenants`
- Proportional bar

**Color cue on the ratio:** < 5% = neutral; 5–10% = amber; > 10% = red.
- **Newness bypass:** Properties < 60 days old skip the color cue and show: *"New property — onboarding offers expected."* Joining-offer-heavy onboarding routinely runs 8–12% — coloring those amber/red trains operators to ignore the color.

Sorted by amount DESC.

Tap any row → worklist filtered to that property + current period.

> **No per-row MoM here (v1.1 change):** Discount volume is episodic. At property level, MoM swings are usually one-tenant move-out variance, not a real signal. Per-row MoM is kept ONLY in By Reason where stable categories make it diagnostic.

---

### 4. By Reason / Type

**The question this answers:** "What kind of discount is driving the spend?"

A ranked list — sorted by owner-funded amount DESC. Each row: type label, amount, **MoM delta chip**, count, proportional bar.

**Discount type categories (P1 locked taxonomy — 4 categories, NOT 7):**

| Type | What it covers | Detection |
|------|----------------|-----------|
| Joining Offer | Discount given at tenant onboarding (well-defined trigger) | `Credits.date_added` within 7 days of tenant creation OR linked to `Room.fixed_discount` at tenant creation OR `Credits.description ILIKE '%joining%'` |
| Negotiation | Mid-stay retention, threat-to-leave discount | `Credits.description ILIKE '%negotiat%' OR ILIKE '%retain%' OR ILIKE '%retention%' OR ILIKE '%loyalty%'` |
| Maintenance Adjustment | Compensation for service issue (AC, water, etc.) | `Credits.description ILIKE '%maintenance%' OR ILIKE '%adjust%' OR ILIKE '%compensation%' OR ILIKE '%waiv%' OR ILIKE '%kharab%' OR ILIKE '%broken%'` |
| Other | Everything not matching above | NOT ILIKE any of the above patterns |

**Display logic:**
- Show all types with amount > 0, sorted by owner-funded amount DESC
- Types with ₹0 hidden
- Tap any row → worklist filtered to that type

**Per-row MoM delta:** Same partial-month same-days rule as hero. Suppressed when count ≤ 2.

> **Why 4 categories, not 7 (v1.1 change):** The earlier draft had 7 categories (Joining / Loyalty / Negotiation / Waiver / Maintenance / Promotional / RentOk Offer / Other). Empirical reality: PG operators don't type "loyalty" or "promotional" into `Credits.description` free text. They type "AC kharab tha", "early exit", "Diwali offer". Most of those landed in "Other" anyway, making the 7-category taxonomy a false-precision exercise.
>
> The collapsed taxonomy: Loyalty merged into Negotiation (operationally the same — keep tenant happy). Waiver merged into Maintenance Adjustment (waivers are usually for service issues). Promotional dropped (covered by Bulk-cohort detection in EC-09). RentOk Offer dropped from this section (already shown via the conditional chip below the hero — its own tappable signal).
>
> If post-launch data shows `Credits.description` text patterns that warrant additional categories, expand iteratively. Don't pre-engineer a taxonomy operators won't populate.

> **Phase 2: structured `discount_type` enum.** Adding a proper categorical column would make this screen cleaner. Flagged in Phase 2.

---

### 5. Who Got Discounts

**The question this answers:** "Which tenants are receiving discounts?"

A ranked list sorted by owner-funded amount DESC. Each row:
- Tenant avatar + name
- Status badge: Active / Booking / Moved Out / Tenant record removed
- Owner-funded discount total
- Discount count
- **Repeat-discount chip** if same tenant has ≥ 2 entries in period: "3 discounts"
- Mini reason summary: "Mostly Joining Offers" or "Negotiation + Maintenance"

**Tenant resolution:** `Credits.tenant` (firebase_id) → join to Tenant table on firebase_id + property + pg_number.

**Active-tenant high-discount alert (conditional, recalibrated v1.1):**
Fires only when **(a)** an individual active-tenant discount **exceeds 25% of that tenant's monthly rent**, OR **(b)** total active-tenant discounts in the period are **≥ ₹10,000**:

> **v1.2 tightening:** Earlier v1.1 also fired on "≥ 3 active-tenant discounts." Operator audit caught the false-positive: half-active PGs commonly issue 3+ small AC-kharab compensations across active tenants in a normal month. Dropped the entry-count branch; kept only per-tenant proportional + period-total magnitude triggers.
→ *"⚠️ ₹X to active tenants — including some unusually large ones. Worth a review."*

> **Recalibrated from v1.0:** Earlier `≥ ₹3,000` threshold fired for routine cases (₹500 off broken AC, ₹1,500 loyalty discount). Active-tenant discounts are common in PG operations. Current thresholds catch the actual outliers — anomalously large or unusually frequent.

**Booking-tenant discount note (conditional, info-only):**
If discounts went to bookings (`status = 2`):
→ *"₹X in joining-offer discounts to N new bookings. Tracked as acquisition cost."*

If >5 tenants: collapse remaining into "Others (N tenants) · ₹X" with tap-to-expand.

Tap any row → worklist filtered to that tenant's discounts.

---

### 6. Who Applied Discounts (collapsed by default; auto-hidden for single-staff PGs)

**The question this answers:** "Who on my team is approving these discounts?"

A ranked list sorted by amount DESC. Each row: staff name + role badge + total amount + count + share.

**Source:** `Credits.initiated_by` (team_uuid) — directly resolved to team member name. Unlike Refunds, this does NOT require a passbook join. **The data is right there on the Credits row.**

**Auto-hide rule:** If only one team member has applied discounts in the period (typical owner-only PG), hide this section entirely. Single-row sections add no value.

**Staff×property concentration callout (conditional, multi-property only):**
If one staff member applied ≥ 60% of discounts at one specific property:
→ *"⚠️ Aakash applied 80% of discounts at Property 2 (₹19,200). Worth a review."*

This is THE governance signal Priya wants. Cross-cuts staff and property — not visible in either dimension alone.

If >4 staff: collapse into "Others (N people) · ₹X."

---

### 7. Discount Trend Chart (always visible)

Below all accordions, a bar chart is always visible — no tap required.

Each bar = one month's **owner-funded** discount total. Single color (consistent with DA-04 Expenses and DA-06 Liabilities). No stacking by reason — for a screen most operators glance at, bar height is the only signal that matters. (DA-03 Refunds uses stacked-by-reason — a domain-specific decision; DA-05 does not inherit that pattern.)

Time selector: `6M` (default) · `This Month`

> **v1.1 simplification:** Earlier draft stacked bars by top-3 reasons + RentOk overlay. Operator audit flagged that as visual noise for an episodic-data screen. Single-color bars match DA-03/DA-04 pattern. Stacking can come back in Phase 2 if usage data shows it useful.

> **Default is 6M, not This Month.** Discount events are sporadic — single-month view tells you nothing.

**Trend insight text (conditional):**
- *"Discounts trending up for 3 consecutive months."*
- *"Costs of discounting are stable across 6 months."*
- *"⚠️ Largest discount month in the last year. Review what changed."* (if current month is the highest in the 6M window)

Bars tappable → tooltip with exact owner / RentOk split + reason breakdown. Tap bar → opens worklist for that month.

---

## Drilling Down: Discount Worklist

Every tappable element opens a filtered discount list.

### Worklist Pre-sets

| Where tapped | Pre-set filter | Default sort | Header |
|-------------|---------------|--------------|--------|
| Hero owner-funded total | Owner-funded only, all dates in period | date_used DESC | "Owner-Funded Discounts · 6 · ₹13,000" |
| Hero RentOk-funded total | RentOk-funded only | date_used DESC | "RentOk Offers · 2 · ₹2,000" |
| By Property row | That property's pg_number_filter | date_used DESC | "[Property Name] · N · ₹X" |
| Reason row | `description ILIKE '[type prefix]%'` (or computed type label) | date_used DESC | "Joining Offer · 3 · ₹8,000" |
| Hero RentOk-funded chip | `source = 1` | date_used DESC | "RentOk-Funded · 2 · ₹2,000" |
| Unused-discounts chip | `status = 0 AND expiry_date >= now` | date_added DESC | "Unused Discounts · 3 · ₹3,000" |
| Hero repeat-tenant callout | Tenants with ≥ 2 discount entries with same description | date_used DESC | "Repeat-Discount Tenants · N · ₹X" |
| Bulk-cohort filter (worklist filter only) | Cohort grouping (same initiated_by + 60s window + same description) | date_added DESC | "Bulk Campaign · N tenants · ₹X" |
| Tenant row | That tenant's firebase_id | date_used DESC | "Pawan Sharma · 3 · ₹4,500" |
| Issuer row | That team_uuid (`initiated_by`) | date_used DESC | "By Aakash · 5 · ₹19,200" |
| Trend chart bar | That month's date range | date_used DESC | "Apr Discounts · 5 · ₹10,650" |
| Outlier callout | Specific discount UUID → opens detail | n/a | "Discount Detail" |
| Repeat-tenant callout | That tenant's firebase_id + selected period | date_used DESC | "Pawan Sharma · 3 discounts · ₹4,500" |

### Per-Discount Row

- Tenant name + status badge (Active / Booking / Moved Out / Tenant record removed)
- Amount (prominent) + source badge (Owner / RentOk)
- Date applied (`date_used`) — fall back to `date_added` if unused
- Type label (derived from description ILIKE)
- Description text (truncated to ~40 chars, full text on tap) — operator-written reason
- Linked invoice context: "Against: Invoice #1234 (₹50K · Rent · Apr)" if applied
- Issuer: team member name + role badge
- Property name (multi-property only)
- **Lifecycle badge (only when status ≠ Used)** — for the rare cases where an issued/scratched/expired entry surfaces in the worklist. Used entries (the dominant case) get no badge.
- **Repeat chip** if same tenant appears multiple times in the worklist: "3 entries this period"
- **Cohort chip** if part of a bulk-issued batch: "Bulk · 5 tenants"
- **Refund-impact note (when applicable):** if the linked payment was refunded but Credit.status still = 1 (used), show: *"⚠️ Linked payment was refunded — discount usage may be inflated."* — surfaces the known engineering bug.

### Per-Discount Actions

- **View Detail** — opens discount detail screen with linked invoice + payment + tenant context
- **View Tenant** — opens tenant detail
- **View Linked Invoice** — opens the invoice the discount was applied to (if used)
- **View Cohort** (when row is part of a bulk-issued batch) — shows all entries from the same campaign
- **Delete Discount** (permission-gated, with confirmation) — hard-deletes the Credit row.
  - Permission required: `delete_discount` permission key (NEW — does not exist in code today; see Engineering Note 14).
  - Confirmation modal: *"This deletes the discount permanently. There's no undo and no audit trail. Continue?"*
  - If discount has `status = 1` (used), warn: *"This discount was already applied to a payment. Deleting will reverse it from the linked payment's `credits_used` field."*
  - On confirm: hard-delete + log to activity log (NEW — discount delete event currently not logged; see Engineering Note 3).

> **Why this is in the worklist (v1.1 addition):** The Rajesh narrative shows him "deleting the May ones" when he spots a manager re-issuing a discount that should have ended. The earlier draft showed this in the story but never specified the action. Operators need a tangible path from "I see this is wrong" to "I removed it." Soft-delete + audit trail is Phase 2 — current implementation is hard-delete with confirmation gate.

### Worklist Filters

Operator can further filter: Date range · Source (Owner / RentOk) · Type · Status (Used / Issued / Scratched / Expired) · Tenant status · Recorded by · Property · Repeat tenants only · Bulk cohorts only · Refund-impacted only

### Bulk Actions

- **Export to Excel** — **NEW BUILD.** No discount export endpoint exists in the codebase. Phase 1 must build a `/generateDiscountReport` endpoint that emails an `.xlsx`. Fields: Date Applied | Tenant | Tenant Status | Amount | Source (Owner/RentOk) | Type | Description | Issuer | Property | Linked Invoice | Lifecycle Status | Repeat-Tenant Flag | Cohort ID (heuristic). Toast: *"Export sent to [email]. You'll receive it in a few minutes."*

---

## Drill-Down Behavior

Defines how every tap target on this screen and its descendants navigates, what filters carry over, and how state is preserved.

### Universal Rules

**R1. Modal/Sheet/Screen primitive [P0]** *(updated 2026-05-11)*
Every tap target's destination is explicitly typed: full-screen push, bottom sheet, modal overlay, or inline accordion. **ⓘ icon convention (locked across DA suite):** single-tap → bottom sheet. No inline tooltip. No long-press. See `[[_Build Sheet Generation Spec#15. ⓘ Icon Interaction Convention]]`.

**R2. Back-stack semantics [P0]**
Back pops one navigation frame and restores prior frame's filter chips, scroll position, accordion state, selected segments. iOS swipe-back and Android system back behave identically.

**R3. Deep links + share sheet [P1]**
Every drill state is uniquely URL-addressable as `rentok://da-05/<view>?<filters>`. Push notifications and WhatsApp deep-links generate these URLs.
> Engineering note: align with existing `rentok://<screen>` singular-form router pattern.

**R4. Permission gating UX [P0]**
Sections without read permission HIDE (not gray). Disabled actions show lock icon + toast on tap. Cross-screen drill into denied screen shows full-screen denial; back returns to source bottom sheet.

**R5. Loading states [P0]**
Skeleton-on-load (not spinner). Cross-screen transitions show destination header immediately + skeleton rows. Pull-to-refresh shows chevron without flashing skeletons.

**R6. State preservation [P1]**
Tab switches preserve per-tab filter and scroll state. App background within 15 min restores exact drill state. Force-quit returns to homescreen.

**R7. Multi-property scope inheritance [P0]**
Single-property drill OVERRIDES global scope for entire descendant drill stack. Scope chip always visible in worklist header.

**R8. Worklist filter-chip behavior [P0]**
Every pre-applied filter is a removable chip. Removal re-fetches without that filter; others stay. New filters are additive (AND).

**R9. Shareable state [P2]**
"Share this view" affordance in overflow menu generates deep link + system share sheet.

**R10. CA-screenshot discipline [P0]**
Every hero has visible GAAP subtitle and basis label — never tap-only.

**R11. Cross-screen back path [P1]**
Cross-screen drill pushes destination as CHILD of source bottom sheet. Destination shows breadcrumb "← From [Source DA name]".

### DA-05 Specifics

- **Hero (Used Discounts)** — Tap opens *Owner-Funded vs RentOk-Funded* breakdown bottom sheet (R1). Tap any segment → worklist with `source` filter chip pre-applied (R8). ⓘ icon: single-tap → bottom sheet with plain-English + GAAP definition (per Generation Spec §15 — no inline tooltip, no long-press).
- **Outlier callout (largest single discount)** — Tap → Discount Detail full-screen push (R1). Back returns to DA-05 with scroll restored (R2).
- **Repeat-tenant callout** — Tap → worklist with `repeat_tenant=1` filter chip (R8). Removable to broaden view.
- **Trend chart bars** — Tap a bar → worklist filtered to that month with `selectedPeriod` overriding parent hero period for descendant drills (R8).
- **Worklist row (Discount entry)** — Tap → Discount Detail full-screen push showing: tenant, amount, source, type, description, issuer, linked invoice, lifecycle status, repeat-tenant flag (R1). Header carries scope chip (R7).
- **View Linked Invoice** (from Discount Detail) — Cross-screen drill → existing Invoice Detail screen with breadcrumb "← From DA-05 Discounts" (R11). Back returns to Discount Detail.
- **View Cohort** (from Discount Detail, when cohort heuristic detects ≥2 same-campaign entries) — Bottom sheet listing all entries from same campaign, scoped to current period (R1, R7). Tap any row pushes that Discount Detail as child (R11).
- **Delete Discount** (worklist row action) — Permission-gated; without `editDiscounts`, action is hidden, not grayed (R4). With permission: confirmation modal *"Delete this discount? This cannot be undone."* → on confirm: worklist row removed + audit log fires (HB3 dependency). Stays on worklist.
- **Bulk Actions → Export to Excel** — In overflow menu (R9). On tap: confirmation modal → toast *"Export sent to [email]"* → stays on worklist.
- **Empty states** — *"No discounts in this period"*: shows "Clear Filters" CTA → re-fetches without active chips (R8). *"Discount feature not enabled"*: routes to onboarding flow per Quick Setup pattern.
- **Permission denial cross-drill** — Tapping "View Linked Invoice" without `viewInvoices` shows full-screen *"You don't have access to invoices"* + back button to Discount Detail (R4).
- **Deep link** — `rentok://da-05/worklist?source=owner&period=this_month` opens DA-05 with chips pre-applied (R3).

### Priority Summary
| Priority | Rules | Engineering guidance |
|----------|-------|---------------------|
| **P0 (must have)** | R1, R2, R4, R5, R7, R8, R10 | Universal mobile expectations |
| **P1 (should have)** | R3, R6, R11 | Significant UX value |
| **P2 (defer if needed)** | R9 | Net-new feature beyond Excel export |

---

### Universal Rule Clarifications (post-orphan-audit)

Resolves interaction-primitive ambiguities surfaced during the post-master orphan-tap-target audit. Treat as additive to R1–R11 above.

**R1 clarification — explicit primitives for common UI:**
- **Hero ⓘ icon:** single-tap = inline tooltip (one-line plain-language definition); long-press = bottom sheet with full GAAP framing + basis label.
- **MoM chip (hero + per-row on Reason breakdown):** tap = inline tooltip showing prior-period numbers + computation window. No drill.
- **Accordion section:** tap on row OR chevron toggles expand/collapse. Default state per section spec.
- **Information-only chips** (repeat-discount chip on tenant row, "Mostly Joining Offers" mini reason summary): single-tap = inline tooltip. No drill unless explicitly listed in DA-05 Specifics.

**R5 clarification — pull-to-refresh:** Dashboard re-fetches all sections in parallel; worklist re-fetches with current chips. Cross-screen drill destinations also support pull-to-refresh. Chevron animates without flashing skeletons.

**R8 clarification — filter chip ✕ as explicit tap target:** Every chip has a discrete ✕ icon (44pt min hit area). Tapping ✕ removes that chip and re-fetches; other chips stay. Body-tap opens edit affordance where applicable; otherwise no-op.

**R12 (NEW) — Trend chart conventions [P0]:**
- **Bar single-tap:** inline tooltip showing exact discount values + count. No drill.
- **Tap-into-tooltip CTA "View [period] →":** drills to discount worklist filtered to that period.
- **Period selector toggle (`6M` / `This Month`):** tap to switch range. Selected state preserved on back. Does NOT affect dashboard hero values.

### Updated Priority Summary

| Priority | Rules | Engineering guidance |
|----------|-------|---------------------|
| **P0 (must have)** | R1, R2, R4, R5, R7, R8, R10, R12 | Universal mobile expectations |
| **P1 (should have)** | R3, R6, R11 | Significant UX value |
| **P2 (defer if needed)** | R9 | Net-new beyond Excel export |

### Permission Vocabulary Reality Check

Codebase has only **11 JWT-mirrored permission keys**: `appAccess, cashCollection, recordPayment, editInvoices, editTenants, viewInvoices, viewExpenses, deleteInvoices, addTenants, deleteTenants, viewTenants` (mirror sites: `src/v1/login/property/service.ts:79-91` + `src/controllers/property.ts:14883/15041/15767/17813` + `src/helpers/teamMember.ts:134`). DB-side via `checkAuthInDb`: ~70 snake_case columns on `team_member_property`.

Keys cited in this spec vs codebase reality:

| Cited key | Status | Recommended Phase 1 path |
|-----------|--------|--------------------------|
| `viewDiscounts` (cited) | DOES NOT EXIST | Build (new column + JWT mirror at 5 sites + frontend toggle), OR reuse `viewInvoices`. Recommend BUILD if discount visibility differs from invoice visibility for compliance |
| `editDiscounts` (cited) | DOES NOT EXIST | Build (mutation gate for Delete Discount worklist action), OR reuse `editInvoices` |
| `viewInvoices` | EXISTS — JWT key | For "View Linked Invoice" drill from discount detail |

> **Decision owner: Jatin (Sr Backend).** Each MISSING key requires build-or-reuse decision. Specs cannot ship until Jatin signs off.

> **Filter code enum status:** `DiscountFilterCode` does NOT exist in `src/v1/constants/filterCodes.ts`. Range 1601-1699 is free (no collisions). Phase 1 must build this enum.

---

## Time Filter Behavior

| Component | Behaviour |
|-----------|-----------|
| Hero (used discounts) | `Credits.date_used` in selected period AND `status = 1` |
| Hero (issued/unused — for lifecycle) | `Credits.date_added` in selected period |
| MoM chip | **This Month / Last Month:** "vs [MonthName]". **Custom range:** "vs equivalent prior period" with same-days tooltip. **Prior period = ₹0:** "Not enough prior data" (avoid ÷0). |
| Outlier callout | Largest single owner-funded discount in period |
| Repeat-tenant callout | Within selected period only — no rolling window (avoids false-positive cross-period matches) |
| By Property | `date_used` in period, grouped by `property` (parsed string) |
| By Reason | `date_used` in period, grouped by description ILIKE classifier |
| RentOk-funded chip (hero) | `date_used` in period AND `source = 1` |
| Unused-discounts chip (hero) | `status = 0 AND expiry_date >= now` (timeless query — not period-bound) |
| Repeat tenants & Bulk cohorts | Tenant grouping within selected period (≥ 2 entries) + cohort grouping (60s window same `initiated_by`) |
| By Tenant | `date_used` in period, grouped by `Credits.tenant` |
| By Issuer | `date_used` in period, grouped by `Credits.initiated_by` |
| Trend chart | Has its own 6M / This Month selector — not governed by global time filter |

**Custom future dates:** Show ₹0 for future portion. Helper: *"Discounts use the date applied. Future dates show ₹0."*

---

## Edge Cases

**EC-01: No discounts in period**
Hero shows ₹0 (0 discounts). MoM chip hidden. All accordions show *"No discounts applied for this period."* Hint: *"Wrong period? Change the date filter."*

**EC-02: All discounts are RentOk-funded**
Owner-funded hero shows ₹0. RentOk-funded shows the actual amount. Subtitle on owner: *"No owner-funded discounts this period — your revenue wasn't affected."* MoM chip suppressed. Net Collection impact shows ₹0 with note: *"RentOk-funded only — your collections not reduced."*

**EC-03: All discounts are Owner-funded**
RentOk-funded section shows ₹0 with subtitle: *"No RentOk offers used this period."* Standard owner-funded display.

**EC-04: Single tenant accounts for >50% of owner-funded discounts**
Outlier callout fires automatically. By Tenant section shows that tenant at top with prominent share. Repeat-discount chip if applicable.

**EC-05: Reason "Other" > 30% of total**
Contextual note: *"A large share of discounts are uncategorised. Tap to see what's in 'Other.'"* CTA → worklist filtered to "Other" reason for review. Operators who don't use standard keywords in `description` end up here.

**EC-06: Multi-property (Priya)**
By Property breakdown appears immediately below hero. Each property shows discount-to-collection ratio with color cue.

**EC-07: Discount applied but linked payment was refunded (KNOWN BUG)**
The codebase does NOT flip `Credits.status` back to 0 when a payment is refunded. The discount usage stays on the books even though the underlying transaction was reversed.

Detection: scan refunds in the period; for each refund, find the linked payment; for each payment with `credits_used > 0`, list those Credit rows. If any have `status = 1` (used), surface as a warning:

→ *"⚠️ ₹X in discount usage may be inflated. [N] discounts were applied to payments that were later refunded — but the discount usage wasn't reversed. This is a known data integrity issue."*

CTA: "Review" → worklist filtered to refund-impacted discounts.

**Engineering action:** This requires fixing the refund flow to flip `Credits.status = 0` (and clear `payment_id`, `date_used`) when the linked payment is refunded. Logged as Engineering Note 1.

**EC-08: MoM — divide-by-zero or insufficient prior data**
If prior period total = ₹0 (would cause ÷0), or property is < 30 days old, show *"Not enough prior data"* instead of a percentage.

**EC-09: Bulk discount campaign issued (`addBulkCreditsFromPhone`)**
Bulk-issued discounts have no `batch_id` in the schema — campaign tracking is impossible at the data layer. They appear as N individual rows in the worklist, all with the same `date_added` timestamp (within seconds). Detection heuristic: ≥ 5 discounts created within a 60-second window with the same `initiated_by` = likely bulk campaign.

When detected, group them in the worklist via the bulk-cohort filter with a "Campaign" label. They appear in the standard 4-category Reason breakdown ("Other" most often, sometimes "Joining Offer" or "Negotiation" depending on description) — there is no dedicated "Promotional / Campaign" Reason category in v1.1+.

**EC-10: Joining offer discount auto-created via `Room.fixed_discount`**
When a tenant is created with a room that has `fixed_discount > 0`, the system auto-creates a Credit with 1-month expiry. Detection: Credit was created within 24 hours of tenant creation AND amount matches `Room.fixed_discount`. Label as "Joining Offer (auto)" in the type breakdown.

**EC-11: Discount marked used but `payment_id` is null**
Data integrity issue. Should not happen but legacy data may have it. Show in worklist with flag: *"Discount marked used but no linked payment — review."*

**EC-12: Tenant resolution fails (deleted / orphaned firebase_id)**
By Tenant row groups under "Tenant record removed" at bottom. Discount itself still counts in totals.

**EC-13: Discount expired but not actually given up**
Issued + expired without redemption = ₹0 actual revenue impact. Worklist row for these entries shows: *"Expired Unused — discount you offered that wasn't taken. No revenue impact."* Excluded from hero owner-funded total (since `status = 0`).

**EC-14: Repeat-tenant flag — legitimate vs suspicious**
A tenant getting 2+ discounts in a period might be legitimate (split bills with different reasons — e.g., joining offer + maintenance compensation) or suspicious (manual re-entry leakage). The flag fires either way. The breakdown view shows reasons inline so operators can self-classify. No alarm color — just a chip.

**EC-15: Discount in period but linked invoice in different period**
A May 7 discount applied to a March invoice. The discount counts in May (its `date_used`). Linked invoice context shows: *"Against: Invoice from Mar 15"* — operator gets context.

**EC-16: Cross-screen integrity — DA-01 Dues shows gross dues, ignoring unused credits**
If a tenant has unused credits (`status = 0`, `expiry_date >= now`), DA-01 Dues hero shows their full obligation without subtracting available discounts. The tenant could pay less than the dues number suggests.

This is by design (dues = what's billed, not what's net of available discounts). DA-05 surfaces this as a footer note when significant unused credits exist:
*"₹X in unused discounts is available across [N] tenants. DA-01 Dues shows gross obligation; net of these discounts, outstanding is ₹Y less."*

Engineering Note 2 covers the underlying gap.

---

## Words on the Screen

### Empty States

| When | Message | CTA |
|------|---------|-----|
| No discounts in period | "No discounts applied in this period. Wrong period? Change the date filter." | "Change filter" |
| All RentOk-funded | "No owner-funded discounts this period — your revenue wasn't affected." | — |
| All Owner-funded | "No RentOk offers used this period." | — |
| Custom future range | "Discounts use the date applied. Future dates show ₹0." | — |
| Worklist filter empty | "No discounts match your current filters." | "Clear Filters" |
| New property | "No discounts yet. They'll appear here when you or your team applies a discount on a tenant's bill." | — |

### Error States

| When | Message | Recovery |
|------|---------|----------|
| Network failure | "Couldn't load discounts. Check your connection." | "Retry" |
| Section fails | "Couldn't load this section." | "Retry" on that section |
| Cross-module reconciliation fails | (silent — show breakdowns without integrity warnings) | — |

### Hero ⓘ Tooltip
> "Shows all discounts applied to tenant bills. Owner-funded discounts reduce your revenue. RentOk-funded discounts don't — they're shown separately."

### Owner-Funded Hero ⓘ Tooltip
> "Discounts you gave from your revenue. Each ₹100 here is ₹100 less in your collections."

### RentOk-Funded ⓘ Tooltip
> "Promotional discounts funded by RentOk. Tenants got the benefit; your bottom line wasn't affected."

### By Reason ⓘ Tooltip
> "Discount types based on what was written when the discount was given. Common ones are joining offers, negotiations, and maintenance compensation. Unrecognised reasons appear under 'Other.'"

### Unused-Discounts Chip ⓘ Tooltip (when shown)
> "Discounts you've issued but tenants haven't applied yet. They're not pulling from your revenue until they're used."

### Who Got Discounts ⓘ Tooltip
> "Tenants who received discounts this period. A tenant with multiple discounts is flagged — could be a single bill split or repeat discount."

### Who Applied Discounts ⓘ Tooltip
> "Which team member applied each discount. If one person applied a disproportionate share at one property, it's worth a check-in."

### Discount-to-Collection Ratio ⓘ Tooltip (in By Property)
> "Owner-funded discounts as a percentage of what this property collected. Healthy is below 5%. Above 10% is worth a closer look."

### Net Collection Hint (small text below hero)
> "This reduced Collections by ₹[X] this month."

### Loading States

| Component | Loading behaviour |
|-----------|-------------------|
| Hero (Owner + RentOk) | Skeleton matching both number blocks + MoM chip placeholder |
| Each accordion | Header visible immediately; 3 skeleton rows in content area |
| Trend chart | Grey placeholder rectangle; loads after accordions |
| Overall failure | Hero shows error state; accordions show individual "Retry" |

---

## Critique of the Old PRD

> Old PRD: `Property Analytics → Finance → Discount Insights.md`. Treats discounts as a marketing-platform redemption dashboard. Most of the spec doesn't fit PG context.

### 1. Old PRD frames discounts as voucher analytics

Old PRD's hero shows "Total Given" as voucher face value. Lifecycle (Active / Used / Expired) is the dominant axis. "Utilization Rate" is a P0 metric.

**Reality:** PG operators don't issue vouchers in the marketing sense. They give discounts inline at billing. The lifecycle axis is real (Credits has status / view_status / expiry_date) but secondary. The dominant axis is **revenue impact** — owner-funded vs RentOk-funded.

**Action:** Reframed. Hero shows owner-funded as the single primary number. RentOk-funded surfaces as a conditional chip below the hero only when amount > 0. Lifecycle is dropped as a section entirely — surfaced as conditional "unused discounts" chip below hero (only when count > 0).

---

### 2. Old PRD makes Type / Reason a P2 feature ("if categories configured")

The single most-asked operator question — "what kinds of discounts are these?" — is gated behind "if categorized."

**Reality:** Even free-text descriptions can be classified via ILIKE prefix matching (same pattern as DA-04 expense categories). However, PG operators don't type "loyalty" or "promotional" — they type "AC kharab tha" or "Diwali offer." A simple 4-category taxonomy (Joining / Negotiation / Maintenance / Other) catches the practical patterns without false-precision.

**Action:** Reason breakdown is P0 with **4-category taxonomy** (NOT the 7 categories in earlier drafts). If post-launch data shows additional patterns warrant more categories, expand iteratively.

---

### 3. Old PRD assumes a `category` column on vouchers

`Credits` has no `category` or `type` column. Free-text `description` is the only signal.

**Action:** Spec uses ILIKE matching on description. Phase 2 adds a structured `discount_type` enum.

---

### 4. Old PRD has no By Property breakdown

For Priya (multi-property), this is the screen's most useful entry point.

**Action:** By Property at position 3 (immediately below hero) for multi-property operators, with discount-to-collection ratio per property.

---

### 5. Old PRD has no repeat-tenant or bulk-cohort detection

**Reality check:** The codebase does NOT support recurring/monthly discounts. Every Credit is a one-time event. `Room.fixed_discount` creates ONE credit at tenant creation with 1-month expiry — no cron regenerates it. There is no "drip leak" pattern from automated infrastructure.

**What IS real:** (a) operators sometimes manually re-issue similar discounts to the same tenant (worth a flag), (b) bulk campaigns issued via `addBulkCreditsFromPhone` create N rows in seconds (worth grouping for cohort analysis).

**Action:** Repeat-tenant pattern surfaced as conditional callout in hero block (≥ 3 entries OR ≥ 2 same-description). Bulk-cohort detection used for worklist filter only (no dedicated section). Phase 2 adds proper recurring discount infrastructure if product wants it.

---

### 6. Old PRD has no source split (Owner vs RentOk-funded)

The codebase explicitly tracks `source` (0 = owner, 1 = eazypg). Reports already split owner_credit vs eazypg_credit. Old PRD treats discounts as a single bucket.

**Action:** Hero shows owner-funded as single primary number. RentOk-funded is a conditional chip below hero (only when > 0). MoM and Net Collection impact use owner-funded only. By Source as a dedicated section was tried in v1.0 but dropped — redundant with the hero pattern.

---

### 7. Old PRD has no cross-screen reconciliation

Discounts intersect with DA-01 (gross dues), DA-02 (net collections), and DA-03 (refund-discount integrity bug). Old PRD treats discounts as a silo.

**Action:** Hero shows "This reduced Collections by ₹X." EC-16 surfaces DA-01 gross-vs-net-of-discount issue. EC-07 surfaces the refund-discount integrity bug.

---

### 8. Old PRD's "Reconciliation Invariant" assumes data model that doesn't exist

Old PRD: "Total Given = Active + Used + Expired." This invariant assumes Credits.status fully covers the lifecycle. In reality, status only has 0/1; "Active vs Expired" is a date comparison, and "scratched vs unscratched" is a separate axis (`view_status`).

**Action:** Reconciliation invariants section uses the actual data model — see "For Engineering" section.

---

## Decisions That Override Old PRD

| Old PRD | What We're Building | Why |
|---------|--------------------|----|
| Hero shows single "Total Given" | Hero shows owner-funded primary number + conditional RentOk-funded chip (only when >0) | Single primary matches DA-03/DA-04. Most operators have ₹0 RentOk-funded — chip surfaces it only when relevant. |
| Voucher lifecycle (Active/Used/Expired) as dominant axis | Lifecycle dropped as a section entirely; surfaced as conditional "unused discounts" chip below hero only | 99% of PG discounts are inline-at-billing. Lifecycle accordion would show empty/trivial data permanently. |
| Type / Reason is P2 ("if configured") | Type / Reason is P0 with **4-category** ILIKE-matched taxonomy (not 7) | Operators don't type "loyalty" or "promotional" — collapsed taxonomy catches actual patterns |
| By Source as a dedicated section | Dropped as a section — replaced by hero's conditional chip | Section 5 was redundant with the hero |
| Repeat Tenants & Bulk Cohorts as a section | Dropped as a section — surfaced as hero callout (repeat) and worklist filter (cohort) | Patterns are real but don't deserve a section — chips are the right altitude |
| No By Property | By Property at position 3 (multi-property) | Priya's first question |
| No repeat / bulk detection | Repeat tenants (≥ 2 entries) + Bulk cohorts (60s window) as a dedicated section | Codebase doesn't support recurring discounts; these are the real patterns operators care about |
| Issuer = "system or user" | Issuer = `Credits.initiated_by` resolved to staff name | Real data, no passbook join needed |
| Voucher expiry-soon as P0 callout | "Unused discounts ready to apply" conditional chip below hero (only when count > 0); expiring-soon variant when expiry within 7 days | Most operators don't issue advance discounts — chip is right altitude |
| Utilization Rate as primary metric | Removed entirely | Not meaningful for inline discounts |
| Beneficiary count + Avg per beneficiary as 3 metrics | Single "distinct tenant count" in hero subtitle | Three metrics for the same question is overkill |
| Top Beneficiaries as P2 | "Who Got Discounts" section as P0 | Most-tapped section in operator usage |
| MoM chip uses voucher-issued total | MoM uses owner-funded applied total | Revenue-impact view |
| No cross-screen language | "This reduced Collections by ₹X" + EC-07 + EC-16 | Required for trust |
| Single MoM on hero | Hero MoM + per-row MoM on Reason only (not By Property — too noisy at small N) | Diagnostic on stable categories; suppress where false-signal risk is high |
| MoM partial-month: full prior month | Same-elapsed-days comparison ("vs Apr 1–7") | Avoid misleading partial-vs-full |
| Trend chart default: This Month or absent | Default 6M, single-color bars (matches DA-03/DA-04) | Discounts are episodic; bar height is the only signal |
| No bulk-campaign detection | EC-09 heuristic + worklist filter (no dedicated section) | Surfaces campaigns without permanent section real estate |
| No joining-offer detection | EC-10 detects auto-created Joining Offer credits | Recognizes the Room.fixed_discount → Credit flow |
| No refund-discount integrity warning | EC-07 + Engineering Note 1 | Catches a real codebase bug operators would otherwise be confused by |
| No reverse/delete UX | Worklist row "Delete Discount" action with permission gate + confirmation modal | Narrative shows Rajesh deleting; spec must define the path |

---

## v1.0 → v1.1 → v1.2 Calibrations (post 2 operator audit rounds)

### v1.1 → v1.2 changes (second-round operator audit, 2026-05-07)

| Item | v1.1 | v1.2 |
|---|---|---|
| 28 orphan references in subsidiary tables | "Section 5 (By Source)", "Section 6 (Lifecycle)", "Section 7 (Repeat & Bulk)", old 7-cat names ("Loyalty / waiver / Promotional / Campaign"), "concession" leakage in 4 places, trend chart "type stacking + RentOk overlay", active-tenant alert ≥₹3K orphan | All swept to current section structure, 4-cat taxonomy, "discount" copy, single-color owner-funded only trend, recalibrated alert thresholds |
| Repeat-tenant callout had 2 OR-branches | Fired on (a) ≥3 entries any-description OR (b) ≥2 entries same-description | Dropped (a); kept only (b) — same-description match. Legitimate split bills (joining + maintenance + festival) no longer trigger alarm. |
| Active-tenant alert had 3 OR-branches | (a) >25% of monthly rent OR (b) period total ≥₹10K OR (c) ≥3 entries | Dropped (c); kept (a) + (b). PG operators commonly issue 3+ small AC-kharab compensations — entry-count branch was constant false-positive. |
| Hero asymmetry case unaddressed | When RentOk-funded > Owner-funded, conditional chip dominates the "primary" hero number | Added rule: when `RentOk > Owner × 2 AND RentOk > ₹5K`, render BOTH as co-primary. |
| Lifecycle badge on EVERY worklist row | Used / Issued / Scratched / Expired badge always rendered | Renders only when status ≠ Used (the 99% case gets no badge) |
| EC-09 referenced "Promotional / Campaign" Reason category | Stale reference to dropped 7-cat taxonomy | Updated: bulk cohorts surface in standard 4-cat ("Other" most often, sometimes Joining/Negotiation depending on description) |
| Phase 2 had duplicate "Refund-discount integrity fix" entries | Listed twice (rows #2 and #11) | Deduplicated; only listed in Pre-Launch Blockers |
| "Loyalty discount ROI" Phase 2 row used dropped category name | "Loyalty discount ROI tracking" | "Negotiation/retention discount ROI tracking" |
| "Concession" leaked 4× in operator-facing copy (Rule 1, MoM rationale, type description, tooltip) | Internal-jargon residue | All replaced with "discount" |
| Pre-Launch Engineering Blockers section absent | 8 implicit blockers scattered across Engineering Notes, EC-07, Phase 2 rows | Section added (see below) — HARD / DEPENDENCY / PARITY / DEFERRABLE structure |
| Permission `delete_discount` flagged as Engineering Note 16 only | Buried in 16-item engineering-notes list | Promoted to Pre-Launch Hard Blocker — governance regression at launch without it |

### v1.0 → v1.1 changes (first-round audit)
See "Decisions That Override Old PRD" table above.

---

## Pre-Launch Engineering Blockers (re-ranked v1.2)

> These items must be resolved before DA-05 ships. Not Phase 2 — Phase 1 launch dependencies. Structured per DA-07 v1.2 pattern.

### HARD blockers (mobile screen would be wrong)

| Item | Why hard-block | File:Line |
|------|----------------|-----------|
| **HB1. Refund→Credit-status integrity bug (EC-07)** | When a payment is refunded, refund flow does NOT flip `Credits.status = 0`. Discount usage tally stays inflated post-refund. Silent data corruption on a financial screen. EC-07 surfaces consequence; fix is at refund flow. **Note: this is DIFFERENT from "Cash flow expense+refund double-count" tracked in DA-03 EC-15 / DA-04 HB3 / DA-07 HB1 — that's a separate bug at `generateCashFlowReport.ts:393` where Refunds are double-counted in totalExpenses.** | `src/services/refunds/refunds.ts` |
| **HB2. `delete_discount` permission missing** | v1.1 worklist Delete UX requires this permission. Today only `record_payment` gates discount creation. Without `delete_discount`, anyone with `record_payment` can delete any discount → governance regression at launch. | (NEW permission) |
| **HB3. Discount delete activity log not wired** | `ACTIVITY_LOG_DISCOUNT = 4` fires on add only. Worklist Delete UX promises audit; without delete-event logging, "deleted" entries are silently lost. | `src/helpers/activityLog.ts` |
| **HB4. `/generateDiscountReport` Excel endpoint NEW BUILD** | Worklist Bulk Actions has Excel CTA in spec; endpoint doesn't exist. Must build before launch. | (NEW endpoint) |
| **HB5. Rupee vs paise unit ambiguity on `Credits.amount`** | `Credits.amount` is `int` but no unit enforcement in code. Likely rupees (operator-entered values like ₹500), but if mis-assumed as paise the math is 100× off. Document explicitly + add validation. | `src/entities/credits.ts` |

### PARITY blockers (mobile + Excel must match)

| Item | Why parity-block | File:Line |
|------|------------------|-----------|
| **PB1. `view_status` default value ambiguity** | Code uses 101 (scratched) and 102 (unscratched). Default value on insert isn't explicit. Affects unused-chip aggregation accuracy. Document + set default = 102 explicitly. | `src/entities/credits.ts` |
| **PB2. `Credits.property` `lastIndexOf('PG')` parser fragility** | Same as DA-03 EC-10 — properties named "PG Greenwood" break parsing. Affects multi-property aggregation. | `src/services/credits.ts` |

### DEPENDENCY (owned by another spec)

| Item | Owner |
|------|-------|
| **DA-01 Dues "net of available credits" gross-vs-net handling** | EC-16 surfaces; full fix is DA-01 work. DA-05 ships when DA-01 hero math accommodates unused credits. | DA-01 spec |

### DEFERRABLE (post-launch hotfix safe)

| Item | Why deferrable |
|------|----------------|
| **D1. >50% "Other" category in-product nudge** | If empirical data shows operators write descriptions that don't match 4 ILIKE prefixes, surface a one-time tooltip suggesting structured `discount_type` enum (Phase 2). Current spec just has EC-05 firing. |
| **D2. Bulk-cohort heuristic refinement** | 60s window may miss bulk imports done in chunks; tune post-launch with real data. |

---

## Cross-Suite Engineering Blockers (DECISION NEEDED — Jatin)

Surfaced during the post-master-merge audit of all 7 DA specs. **NOT DA-suite-specific** — these affect every authenticated endpoint in the app. The DA suite *uses* these endpoints, so Phase 1 launch readiness depends on Jatin's verdict on each.

> **Decision owner: Jatin (Sr Backend).** PM has surfaced; Jatin decides: fix-now vs. fix-fast-follow vs. accept-and-document. Items marked with ⛔ block DA-suite launch until resolved.

| ID | Issue | Files | Risk | Recommendation |
|----|-------|-------|------|----------------|
| ⛔ **CSB-1** | `checkAuth()` returns `true` when `req.is_authenticated === false`. `HeaderValidator` falls through with `next()` (not 401) when Authorization header is missing. **Net result: any endpoint protected by `HeaderValidator + checkAuth` can be hit unauthenticated by simply omitting the Bearer header.** | `src/utils/commonFunctions.ts:1193-1196` (HeaderValidator fallthrough); `:1228` (checkAuth bypass); `:1252` (checkAuthInDb same shape) | Production exposure — every list/detail screen the DA suite drills into is reachable anonymously | Patch `HeaderValidator` to return 401 on missing header, OR fix `checkAuth` to fail-closed when `is_authenticated=false`. Default risk-acceptance is NOT viable for a financial-data app. |
| ⛔ **CSB-2** | DA-01 + DA-02 drill endpoints have **zero middleware** — no HeaderValidator, no JWT decode. `POST /invoices/fetchDueDetailsForTenants` and `POST /invoices/fetchPaymentSettlementDetails` accept body params and query directly. | `src/routes/invoices.ts:203-204` → `src/controllers/invoices.ts:3331, 3424` | Anonymous read of tenant payment data | Add `HeaderValidator` + `checkAuth(viewInvoices)`. Mandatory before DA-01/02 launch. |
| ⛔ **CSB-3** | Tenant Detail + Adjust Deposit + Invoice Detail + Receipt PDF drill destinations have NO `HeaderValidator`. The DA suite assumes these are safe drills. | `src/routes/tenant.ts:927` (`getTenantData`), `:931` (`getTenantStepperv2`), `:944` (`adjustDeposit` — financial mutation); `src/routes/invoices.ts:215` (`getInvoiceData`), `:216` (`generateReceipt`), `:225` (`generateReceiptWrapper`), `:227` (`getSpecificInvoiceData`) | Anonymous read of PII; **anonymous financial mutation** at `/tenant/adjustDeposit` | Add `HeaderValidator` + appropriate `checkAuth` to each. `/tenant/adjustDeposit` is an absolute P0. |
| ⛔ **CSB-4** | DA-03 + DA-04 detail services have HeaderValidator but NO inline `checkAuth`. Authenticated user with any property access can read any refund/expense by uuid. | `src/services/refunds/refunds.ts:488` (refunds, also re-used by `routes/reimbursement.ts:10` via duplicate controller); `src/services/expense/expense.ts:20` (expenses; also no `pg_id` validation) | Cross-property data leak between team members | Add `checkAuth(viewInvoices)` for refunds and `checkAuth(viewExpenses)` for expenses. Logged in DA-03 HB4 + DA-04 HB4. |
| **CSB-5** | Expired-JWT bypass — for non-tenant packages (Manager/Owner/Landlord apps), `HeaderValidator` decodes the **expired** payload, sets `req.is_authenticated = true`, calls `next()`. A stolen-but-expired Manager App token still authenticates indefinitely. | `src/utils/commonFunctions.ts:1104-1126` | Stolen/leaked tokens have indefinite life | Decide: (a) intentional grace window for offline mode (document the window length), OR (b) bug — return 419 for non-tenant packages too. |
| **CSB-6** | JWT permission-key shape diverges by code path. `helpers/teamMember.ts:136` writes 12 keys (includes `bankAccess`); `v1/login/property/service.ts:79-91` writes 11 (omits it). | `src/helpers/teamMember.ts:134-145` vs `src/v1/login/property/service.ts:79-91` | Permission gating becomes path-dependent | Align both sites to the canonical key set. |
| **CSB-7** | Refund/Reimbursement controller duplication — `routes/refunds.ts:10` and `routes/reimbursement.ts:10` both call `RefundsController.getRefundDetails`, but import from two **different files** (`controllers/refunds.ts` vs `controllers/reimbursement.ts`), both classes named `RefundsController`. | as cited | Auth fix in one is silently missed in the other | Dedupe controllers OR add lint check ensuring auth changes propagate. |
| ⛔ **CSB-8** (NEW 2026-05-11) | **Widget endpoint permission enforcement gap (security).** List-screen widget endpoints set up `self_added_team_uuid` for team members with restricted permission (only `view_invoices_of_self_added_tenants`) but never apply it as a WHERE clause to the widget query. Worklist paths enforce it; widget paths do not. | `src/v1/list_screens/dues/service.ts:132-138` (verified DA-01); likely also `collections/service.ts`, `expenses/service.ts`, `refunds/service.ts` — needs audit | Cross-tenant aggregate data leak via widget endpoints. Restricted team member sees correctly-scoped worklist but un-scoped widget totals. Privacy/security concern. | Add the same `andWhere('t.added_by_id = :self_team_uuid', ...)` enforcement to widget query paths. Audit all DA list-screen widget endpoints for the same pattern bug. |

⛔ = blocks DA-suite launch. CSB-1, CSB-2, CSB-3, CSB-4 must be resolved before Phase 1 ships. CSB-5/6/7 are app-hygiene; can ship as fast-follow but should be on Jatin's queue.

---

## Phase 2 — Intentionally Deferred

> Ordered by operator-demand priority. Top 3 are what operators will likely ask for within 90 days of launch.

| Feature | Why |
|---------|-----|
| **1. Joining offer ROI tracking** | The single most operator-demanded follow-up: "Did the tenant who got a ₹2K joining offer stay long enough to justify the cost?" Requires tenancy duration linkage. |
| ~~Refund-discount integrity fix (EC-07)~~ | **Promoted to Pre-Launch Hard Blocker HB1 in v1.2.** Was misclassified as Phase 2 in v1.1; the v1.1 entry itself flagged "should be hard pre-launch fix." |
| **3. Discount approval workflow with cap** | Priya's actual control mechanism. PG operators commonly have informal rules ("> ₹2K needs owner OK"). DA-05 surfaces governance issues but gives no enforcement today. Net-new module. |
| **Soft-delete + audit trail for discounts** | Today, delete is hard with no audit. Soft-delete + audit screen would let operators see voided discounts and undo deletes. |
| **Recurring/monthly discount infrastructure** | Today, every Credit is a one-time event. `Room.fixed_discount` creates one credit at tenant creation only — no cron regenerates. Net-new feature requires: `recurring_until` field, regeneration cron, "active recurring policies" view. |
| **Structured `discount_type` enum on Credits** | Requires schema migration. ILIKE matching with 4 categories covers practical patterns; enum cleans up the long tail. |
| **Campaign / batch tracking for bulk-issued discounts** | `addBulkCreditsFromPhone` has no `batch_id`. EC-09 heuristic catches most cases; explicit batch tracking needs schema work. |
| **Tenant-side notifications (discount applied)** | Today no SMS / WhatsApp / email sent on discount events. Net-new comms wiring. |
| **Negotiation/retention discount ROI tracking** | "Did the retention discount actually keep the tenant? Did they stay longer than 3 months after?" — requires churn data linkage. |
| **Discount approval bypass detection** | If policy is "discounts >₹2K need owner approval," surface breaches. Requires policy config. |
| ~~Refund-discount integrity fix~~ | **Promoted to Pre-Launch Hard Blocker HB1 in v1.2** — see Pre-Launch Engineering Blockers section above. |
| **DA-01 Dues "net of available discounts" view** | Show outstanding net of unused credits, not just gross. EC-16 surfaces; full fix is cross-PRD. |
| **Concentration flag (one staff doing >60% of discounts at one property)** | EC-09's staff×property cross-cut is the closest approximation. A formal governance module is Phase 2. |
| **Discount Audit screen (deleted Credits)** | Today, deletes are hard. No history. Soft-delete + audit screen = Phase 2. |
| **Auto-classification of "Other" descriptions** | NLP / ML to extract better discount types from free-text. |

---

## For Engineering

### Reconciliation Invariants

| Invariant | Formula |
|-----------|---------|
| Reason sum = owner-funded hero | All ILIKE-classified types (incl. "Other") sum = Owner Total |
| Source sum = total | Owner + RentOk = Total Discounts |
| Used count = hero entry count | All `status = 1` rows in period sum to hero owner-funded total (RentOk-funded counted separately) |
| By Property sum = owner-funded hero | All property group sums = Owner Total |
| By Tenant sum = owner-funded hero | All tenant group sums (incl. "Removed") = Owner Total |
| By Issuer sum = owner-funded hero (less unattributed) | All issuer group sums + unattributed = Owner Total |
| Worklist count = parent count | Header count → worklist row count must match |
| **Cross-screen: Owner-funded discount total = DA-02 owner_credits subtraction line** | Same period + same property scope — must match |

### Codebase Feasibility

> **Source:** `src/entities/credits.ts`, `src/controllers/credits.ts`, `src/services/payment/paymentService.ts`, `src/controllers/reports.ts`
> **Reviewed:** 2026-05-07

#### What Already Exists

| PRD Feature | Code Reference | Status |
|-------------|---------------|--------|
| Credit creation (single + bulk) | `addCredits`, `addCreditsFromTenantPhone`, `addBulkCreditsFromPhone` | **EXISTS** |
| Permission gate | `record_payment` / `add_discount_access` via `checkAuthInDB` | **EXISTS** |
| Lifecycle status (used flag) | `Credits.status` 0/1, `date_used`, `expiry_date` | **EXISTS** |
| Source split (owner vs RentOk) | `Credits.source` (0/1) — used in `reports.ts:1894–1944` | **EXISTS** |
| Multi-discount per payment | Array support in `paymentService.ts:561–770, 1164–1530` | **EXISTS** |
| Issuer attribution | `Credits.initiated_by` (team_uuid string) | **EXISTS** — direct, no passbook join needed |
| Tenant linkage | `Credits.tenant` (firebase_id) | **EXISTS** — same 3-hop pattern as Refunds |
| Property scoping | composite `${pg_id}PG${pg_number}` string | **EXISTS** |
| Activity log | `ACTIVITY_LOG_DISCOUNT = 4` (on add only) | **EXISTS** (but only on add — see Engineering Note 3) |
| Owner vs RentOk reporting bifurcation | `reports.ts:1894–1944` | **EXISTS** |

#### What Needs New Build

| PRD Feature | Gap | Effort |
|-------------|-----|--------|
| **Analytics list endpoint (`/v1/discounts/list/filters`)** | No equivalent of expense/collection list_screens for discounts. Mirror the pattern from `expenses/helpers.ts`. | **MEDIUM** |
| **Widget endpoint (`/v1/discounts/list/widget`)** | No widget endpoint. | **MEDIUM** |
| **DiscountFilterCode enum** | No filter code range allocated. **Suggested 1601–1699** (corrected v1.2: earlier draft proposed 1401–1410 but that range collides with `RoomFilterCode` 1401–1420 in `src/v1/constants/filterCodes.ts`). | **LOW** |
| **Type aggregation via ILIKE on description** | New JOIN + GROUP BY with description ILIKE classifier. Mirror DA-04 category logic. | **MEDIUM** |
| **Source aggregation** | New GROUP BY `source`. Trivial. | **LOW** |
| **By Property aggregation** | Parse composite property string OR add derived column. | **LOW** |
| **By Tenant aggregation** | GROUP BY `Credits.tenant` + JOIN to Tenant for status badge. | **MEDIUM** |
| **By Issuer aggregation** | GROUP BY `Credits.initiated_by` + JOIN to team_members for name + role. | **LOW** (no passbook join needed) |
| **Unused-discounts chip query** | `COUNT + SUM` where `status = 0 AND expiry_date >= now`. Plus expiring-soon variant: `expiry_date BETWEEN now AND now + 7 days`. | **LOW** |
| **Repeat-tenant detection** | GROUP BY tenant within period; flag tenants with COUNT ≥ 2. Simple aggregation. | **LOW** |
| **MoM comparison (per-row + hero)** | Prior-period query + delta. | **MEDIUM** |
| **Trend chart aggregation (6M)** | `GROUP BY DATE_TRUNC('month', date_used) WHERE source = 0` (owner-funded only). Single-color bars (matches DA-03/DA-04/DA-06). NO type stacking, NO RentOk overlay (v1.1 simplified). | **MEDIUM** |
| **Outlier callout (largest single discount)** | `MAX(amount) WHERE source=0` query + threshold check. | **LOW** |
| **Bulk-cohort detection** | Window function: ≥ 5 credits within 60s same `initiated_by` AND same `description`. Tag as cohort. | **LOW** |
| **Refund-impact detection (EC-07)** | JOIN Refunds → Payments → Credits where credits.status=1 + refund.invoice = payment.invoices. | **MEDIUM** |
| **Bulk-campaign detection (EC-09)** | Window function: ≥ 5 credits within 60s same `initiated_by`. | **LOW** |
| **Joining-offer auto-detection (EC-10)** | JOIN Tenant; check Credit.date_added within 24h of Tenant.created_at + amount = Room.fixed_discount. | **LOW** |
| **Cross-screen warning footer (DA-01 net-of-discount)** | SUM(unused, unexpired Credits) per property. | **LOW** |
| **Discount Excel export (`/generateDiscountReport`)** | New endpoint mirroring `/generateExpenseReport`. | **MEDIUM** |
| **Active-tenant high-discount alert** | JOIN tenant; filter `tenant.status = 1`. Fire when (a) any single discount > 25% of monthly rent OR (b) period total to active tenants ≥ ₹10,000. (Note: v1.1 ≥₹3K threshold superseded — see body Section 8.) | **LOW** |
| **Staff×property concentration callout** | Cross-cut aggregation: GROUP BY initiated_by + property. | **LOW** |

### Critical Architecture Notes for Engineering

1. **CRITICAL — Refund-Discount integrity bug (`services/refunds/refunds.ts`).** When a payment is refunded, the refund flow does NOT flip `Credits.status` back to 0, does NOT clear `payment_id`, does NOT clear `date_used`. A discount marked "used" stays used even after the underlying payment is reversed. This inflates discount-usage tallies post-refund. Either: (a) fix the refund flow to reverse credit status (preferred), OR (b) DA-05 must permanently surface EC-07 warning to operators. Recommend fix.

2. **CRITICAL — DA-01 Dues shows gross obligation, ignoring unused credits.** Adding a Credit does NOT mutate `Invoices.amount`. Net-of-discount outstanding diverges from DA-01 hero by `SUM(status=0, expiry_date >= now Credits)`. EC-16 surfaces this on DA-05; full fix is a cross-screen change to DA-01 hero or a "net of available discounts" view.

3. **Activity log only fires on add, not on use/scratch/delete.** `ACTIVITY_LOG_DISCOUNT = 4` is logged in `addCredits` only. Use/scratch/delete are silent. Audit history is incomplete. Phase 2 fix.

4. **No transactional atomicity on credit use.** When a payment with `discount_id` is processed, the Credit row is updated separately from Payment write. If the Payment fails after Credit is consumed, the Credit may be orphaned. Wrap in a DB transaction.

5. **`view_status` semantics are ambiguous.** Code uses 101 (scratched) and 102 (unscratched). Default value on insert isn't explicit. Lifecycle aggregation must verify default — recommend documenting in Credits entity comment + setting default = 102 explicitly.

6. **`Credits.amount` is `int` but no paise/rupee enforcement.** Verify against existing `addCredits` callers — likely **rupees**, not paise. Document explicitly. (This is unlike payment amounts which are paise.)

7. **`Credits.tenant` is firebase_id, not tenant_uuid.** Same 3-hop pattern as Refunds. Tenant resolution is brittle; archived tenants may not resolve.

8. **`Credits.property` is composite string `${pg_id}PG${pg_number}`.** Same pattern as Refunds. Multi-property aggregation needs the same parsing logic. Property names containing "PG" will break `lastIndexOf('PG')` parsing — see DA-03 Engineering Note 10 (same bug).

9. **Description ILIKE classification (Type breakdown).** Mirror the `notOtherTypes` array pattern from DA-04 expenses. Order-independent matching. "Other" = NOT ILIKE any of the 6 keyword sets.

10. **Owner-funded vs RentOk-funded math in DA-02 Collections (cross-screen).** `reports.ts:1894-1944` already splits these. DA-05 hero MUST match DA-02's owner_credits subtraction line for the same period. Reconciliation invariant.

11. **Bulk-issue has no `batch_id`.** `addBulkCreditsFromPhone` doesn't tag entries. Campaign tracking impossible at the data layer. EC-09 heuristic detection is the workaround for P1.

12. **No tenant notification on discount events.** WhatsApp templates reference `discount_amount` / `credit_amount` but they're not wired to discount-add or discount-apply events. If product wants tenant-side comms, this is net-new wiring.

13. **`Credits` is independent from `invoice_meta_data.discount_breakup`.** The latter is for RentOk Plans subscription pricing, NOT tenant discounts. DA-05 explicitly excludes it. Don't accidentally JOIN them in queries.

14. **Permission key is `record_payment` (not a dedicated `add_discount_access`).** Anyone who can record payments can add unlimited owner-funded discounts. This is a governance gap — add explicit `add_discount_access` permission as Phase 2.

15. **Dead schema to ignore:** `Invoices.discount`, `tenant_packages.discount` (commented), `Dues.discount` (commented), `packages.discountAmount` (commented). Do NOT use these in any analytics query.

16. **NEW PERMISSION REQUIRED — `delete_discount`.** v1.1 worklist row adds a "Delete Discount" action. Today, `Credits` only has the `record_payment` permission gate for create. Delete is currently performed by the same write controller without a dedicated permission key. Add `delete_discount` permission to `team_member_property` (parallel to existing `delete_refund` on Refunds). Without this, anyone who can record payments can delete any discount, which conflicts with operator governance expectations.

17. **NEW EVENT TO LOG — discount delete.** v1.1 adds delete UX. Currently, `ACTIVITY_LOG_DISCOUNT = 4` is logged only on add. Delete events are silent. Wire activity log to fire on delete with payload (discount_uuid, deleted_by team_uuid, reason if provided). Required for the worklist row's promise that delete is recorded.

18. **Trend chart simplification:** v1.1 trend chart is single-color bars (matches DA-03/DA-04). Drop the earlier stacked-by-reason + RentOk overlay design.
