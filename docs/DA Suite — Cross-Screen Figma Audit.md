---
title: DA Suite — Cross-Screen Figma Audit
date: '2026-05-06'
tags:
  - rentok
  - design-audit
  - figma
  - detailed-analytics
aliases:
  - DA Figma Audit
  - Cross-Screen Audit
---

# Detailed Analytics Suite — Cross-Screen Figma Audit

> **Scope:** All 4 Detailed Analytics screens — DA-01 Dues · DA-02 Collections · DA-03 Refunds · DA-04 Expenses
> **File:** Figma `KgBQXiT7r7oGrcqZHCWxyU` (Home Screen — Manager App)
> **Date:** 2026-05-06
> **Status:** Pre-engineering-handoff blocker list

---

## TL;DR

The 4 screens were designed by the same person and exhibit the same class of systemic bugs. Most of these can be fixed by building **3 design-system primitives** (polarity-aware MoM chip, polarity-aware delta chip, time-filter chip) plus a global token / label cleanup. The single most damaging issue is the **MoM chip color polarity** — it's wrong on 3 of 4 screens.

**Score per screen** (against locked PRD coverage):

| Screen | Coverage | Critical bugs | Missing components |
|--------|----------|---------------|-------------------|
| DA-01 Dues | ~55% | MoM color, "Defaulter Analysis" accordion violation, missing Collection Efficiency, missing Property Breakdown | Time filter, worklist, tooltips, range-mode urgency, setting-up state |
| DA-02 Collections | ~60% | No Collection Efficiency badge in hero, no Adjustment rows in Payment Modes, "Invoices" not "Receipts", Old Dues = red instead of amber | Time filter, hero tooltips, worklist, loading/error states |
| DA-03 Refunds | ~25% | MoM color, old 3-bucket Reason taxonomy, "RentOk" placeholder mode | Trend chart, By Property, Money Source, Active-tenant alert, double-count warning, worklist |
| DA-04 Expenses | ~30% | MoM color, wrong category labels (Operational/Marketing/Utilities/Staff), "RentOk" placeholder, "(120 Bills)" subtitle, "Advanced Insights" wrapper not in spec | Time filter, full-screen frame, trend chart, per-bed metric, worklist, loading/empty states |

---

## The 6 Systemic Bugs (Same Pattern, Multiple Screens)

### Bug 1: MoM chip color polarity — wrong on 3 of 4 screens

| Screen | What Figma shows | What spec requires | Status |
|--------|------------------|---------------------|--------|
| DA-01 Dues | ▼ 12% in **RED** | ▼ in dues = good = should be **GREEN** | ❌ |
| DA-02 Collections | ▲ 12% in **GREEN** | ▲ in collections = good = **GREEN** | ✅ |
| DA-03 Refunds | ▲ 12% in **GREEN** | ▲ in refunds = bad (cost out) = **RED** | ❌ |
| DA-04 Expenses | ▲ 12% in **GREEN** | ▲ in expenses = bad = **RED** | ❌ |

**Root cause:** The designer used a single MoM chip component that maps ▲ to green and ▼ to red unconditionally. This works only for revenue views (Collections). It's wrong for cost views (Refunds, Expenses) and inverted views (Dues, where the metric being tracked is outstanding receivables — decrease is good).

**Fix (one-time):**
Build `MoMChip` component with two variants:
```
polarity: "revenue" | "cost"
direction: "up" | "down"
```
- `revenue + up` → green
- `revenue + down` → red
- `cost + up` → red
- `cost + down` → green

Then audit all 4 screens to assign correct polarity:
- DA-01 Dues = `cost` (outstanding dues — increase is bad)
- DA-02 Collections = `revenue`
- DA-03 Refunds = `cost`
- DA-04 Expenses = `cost`

### Bug 2: Old taxonomy / wrong labels still in Figma

The PRDs explicitly override old labels in the Lark / Figma source. Figma hasn't picked up the changes:

| Screen | What Figma shows | Spec-locked label |
|--------|------------------|---------------------|
| DA-01 Dues | "Invoiced created" | **"Invoices Due"** (spec line 691 explicit override) |
| DA-02 Collections | "(120 Invoices)" subtitle | **"(N Receipts)"** (spec §2 explicit override) |
| DA-02 Collections | "Invoiced created" (typo) | **"Invoices Created"** |
| DA-03 Refunds | 3-bucket reasons: Security Deposits / Advance / Others | **7 reasons** from `invoice.due_type`: Security Deposit · Caution Money · Advance · Rent Overpayment · Late Fee Reversal · Utility/Service · Other |
| DA-04 Expenses | "Operational / Food & Kitchen / Staff" | **Salary · Electricity · Mess/Food · Maintenance · Rent · Deposit · Other** (ILIKE prefix matched) |

**Fix:** Single text-replacement pass across all 4 screens against the locked spec labels. Designer should have a short "label glossary" they apply to every variant.

### Bug 3: Placeholder text leaked into "real" content

The Figma still has obvious placeholders that were never replaced:

| Screen | Placeholder | Issue |
|--------|-------------|-------|
| DA-02, DA-03, DA-04 | "RentOk" appearing as a payment mode / refund mode row | Brand name; not a valid mode in code |
| DA-02 | "230 bills" repeated identically across 4 different staff rows | Same number on every row — placeholder leak |
| DA-02 | Trophy badge hardcoded to RentOk row regardless of who's actually top collector | Badge logic should be conditional on rank |
| DA-02 | Source bar segments at equal width despite different amounts | Equal-width placeholder; real data should be proportional |
| DA-04 | "(120 Bills)" subtitle | "Bills" implies receipts; should be "(N Expenses)" |
| DA-01 | Stale dates "19 Jan'26 → 21 Jan'26" | Test data not updated |

**Fix:** Replace all placeholders with realistic varied data. Designer review before any handoff.

### Bug 4: Spec-violating sub-containers

The spec for DA-04 explicitly does NOT have an "Advanced Insights" wrapper around accordions. The spec for DA-01 explicitly says Defaulter Analysis is always-visible (not behind a tap):

| Screen | Violation |
|--------|-----------|
| DA-01 | "Defaulter Analysis" rendered as a 5th accordion (collapsible). Spec §8: "always visible below accordions, no tap required to see it." |
| DA-04 | "Advanced Insights" container header wraps the breakdowns. Spec doesn't have this — breakdowns appear directly below the hero. |

**Fix:** Remove these sub-containers. Render the components directly per spec.

### Bug 5: Missing components — same 5 categories on every screen

Every screen is missing the same 5 component categories:

| Component | DA-01 | DA-02 | DA-03 | DA-04 |
|-----------|-------|-------|-------|-------|
| Time filter chip above hero | ❌ | ❌ | ❌ | ❌ |
| ⓘ tooltip popovers on accordion headers | ❌ | ❌ | ❌ | ❌ |
| Loading skeleton states | ❌ | ❌ | ❌ | ❌ |
| Empty / error state variants | partial | partial | partial | ❌ |
| Worklist drill-down screens | ❌ | ❌ (14 presets in spec, 0 designed) | ❌ | ❌ |

**Fix:** Build these as design-system primitives once, apply to all 4 screens.

### Bug 6: Inconsistent section naming

The "By X" prefix is used inconsistently within and across screens:

| Screen | Examples of inconsistency |
|--------|---------------------------|
| DA-01 | "Performance" / "By Category" / "By Defaulters" / "Added by" (mixed prefix + lowercase 'b') |
| DA-02 | "Category" (no prefix) / "Payment Modes" (plural, no prefix) / "Received by" (lowercase 'b') |
| DA-03 | "By Category" vs "Category" used in different states |
| DA-04 | Multiple naming variants across collapsed/expanded states |

**Fix:** Standardize on one pattern. **Recommendation:** Use **"By X"** prefix for all breakdown sections; reserve no-prefix naming for non-breakdown sections (Hero, Header, Trend Chart). Apply consistently across all 4 screens.

---

## Per-Screen P0 Fix Lists

### DA-01 Dues — P0

1. Flip MoM chip color: ▼ decrease = GREEN (currently RED)
2. Remove "Defaulter Analysis" 5th accordion — render chart always-visible per spec
3. Rename "Invoiced created" → "Invoices Due" in Performance accordion
4. Add Collection Efficiency % as third KPI in Performance (currently missing — only Current Dues + Invoiced created shown)
5. Design Property Breakdown section (multi-property — entirely absent)
6. Add Range-mode urgency bar variant (3-segment Carried Forward / Due in Period / Due After Period)
7. Add global time filter chip
8. Add Old Tenants deposit context line (₹X deposit held · Covered by deposit ✓)
9. Fix Defaulter Analysis chart legend — colors may currently misrepresent collected vs outstanding
10. Add Maintenance category row + re-sort By Category by amount DESC

### DA-02 Collections — P0

1. Replace "(120 Invoices)" → "(N Receipts)" across populated, empty, success variants
2. Add Collection Efficiency badge below hero number with v2.1 locked behavior:
   - Show actual % (not capped); >100% allowed
   - Color thresholds: ≥80% green / 50–79% amber / <50% red
   - Tooltip when ≤100% and >100% (breakdown by current/old/advance)
   - Success banner only at 100% (±0.5%)
3. Add hero ⓘ tooltip with v2.1 locked copy ("Deposit and advance adjustments are excluded from the net total — they're visible in the Source and Payment Mode breakdowns.")
4. Add gross / refund / net 3-line tooltip on hero tap
5. Add Adjustments grey timing row in Performance (4th row)
6. Add inline Collectible vs Collected progress bar inside Performance
7. Add Collection Efficiency row inside Performance accordion
8. Add separator + "Adjusted from Deposit" + "Advance Adjusted" rows in Payment Modes
9. Recolor Old Dues from red to amber in Source bar AND Trend 6M stacks (red is reserved for Late timing only)
10. Fix typo "Invoiced created" → "Invoices Created"
11. Fix label casing: "Received By" / "By Payment Mode" / "By Category"
12. Add time-filter chip with forward-only date picker

### DA-03 Refunds — P0

1. Flip MoM chip color: ▲ increase = RED (currently GREEN)
2. Replace 3-bucket reasons with locked 7-category list (Security Deposit / Caution Money / Advance / Rent Overpayment / Late Fee Reversal / Utility/Service / Other)
3. Remove "RentOk" placeholder refund mode; use locked 6 modes (Cash / UPI/Digital / Bank Transfer / Card Machine / Cheque / Others/Wallet)
4. Add per-row MoM delta chips on Reason rows
5. Design outlier callout below hero ("Largest: ₹X to [tenant] · [date]")
6. Design Net Collection reminder below hero ("This pulls ₹X out of this month's Collections")
7. Design Money Source breakdown (AF/PF/NPNAF) + Reimbursement chip
8. Design Trend Chart (6M default, stacked by reason)
9. Split single "Refunded by" section into two: "Who Got the Refund" (tenants) + "Processed By" (staff)
10. Add tenant status badges (Active / Moved Out / Cancelled before move-in / Tenant record removed)
11. Design active-tenant alert chip (amber)
12. Design EC-15 double-count footer warning (with "Fix" CTA)
13. Add full-vs-partial refund badge on worklist row
14. Add subtitles on collapsed accordions

### DA-04 Expenses — P0

1. Flip MoM chip color: ▲ increase = RED (currently GREEN)
2. Replace category labels (Operational / Food & Kitchen / Staff) with locked 7 (Salary / Electricity / Mess/Food / Maintenance / Rent / Deposit / Other)
3. Replace "RentOk" placeholder payment mode; use locked 5 (Cash / UPI/Digital / Bank Transfer / Card Machine / Cheque / Others/Wallet)
4. Fix subtitle "(120 Bills)" → "(N Expenses)"
5. Remove "Advanced Insights" sub-container wrapper
6. Design the full-screen mobile frame (currently component-level only — no complete screen exists)
7. Design Expense Trend Chart (always visible, 6M default)
8. Add time filter chip
9. Add Per-Bed Expense line in hero ("₹X per occupied bed")
10. Add prior-period absolute on MoM chip ("▲18% vs ₹69,500 Apr")
11. Add per-row category MoM delta chips
12. Add "N expenses without bills" governance callout below hero

---

## Single Design-System Cleanup (Highest Leverage)

If the design team has limited time, these 4 systemic fixes deliver ~70% of the value:

### Fix #1: Build polarity-aware MoM chip variant
**One component fix → all 4 screens improve.** See Bug 1 above for the spec.

### Fix #2: Build polarity-aware row-level delta chip
Used by per-row MoM on DA-01 by Defaulters, DA-03 by Reason, DA-04 by Category, DA-02 by Property. Same polarity logic as the hero chip but smaller form factor.

### Fix #3: Build global time-filter chip
Forward-only date picker per DA-02 spec. Used on all 4 screens above the hero.

### Fix #4: Build a label glossary, apply once across all 4 screens
Hand-written list of every spec-locked label. Replace placeholder/old text in one focused pass.

---

## What Wasn't a Bug (Worth Calling Out)

To set expectations correctly:

- **DA-02 hero chip color is correct** — designer got it right by accident because revenue view matches the default ▲=green pattern. After the polarity-aware fix, this still works.
- **DA-03 has no spec violations** (no pending queues, no status badges, no gateway names) — Figma was conservative because most of DA-03 is undesigned. Watch this when DA-03 gets fleshed out — same designer, same risk.
- **Source bar / Population breakdown / Unlinked alert on DA-02** — present and accurate.
- **Timing rows on DA-02 Performance** — present (just missing the Adjustments row).
- **Empty + Success variants on DA-02** — exist (with internal inconsistencies, but exist).
- **DA-01 urgency bar Today-mode + Aging chart + Action card** — well-executed, accurate.

---

## Recommended Process for Design Team

**Sequencing:**

1. **Day 1 (~2-3 hours):** Build the 4 design-system primitives (polarity MoM chip, row delta chip, time-filter chip, label glossary). Apply across all 4 screens in one pass.
2. **Day 2-3:** Per-screen P0 fixes (top 3 of each list above).
3. **Day 4:** Worklist screens for all 4 (use a single template pattern).
4. **Day 5:** Loading / empty / error states (design-system primitives).
5. **Day 6:** Multi-property variants (Priya persona).

**Total estimate:** ~1 week of focused design time before the suite is genuinely engineering-ready.

**Reviewer:** PM should review the polarity logic + label glossary specifically — these are the two issues where "looks right" doesn't equal "is right" without checking the spec.

---

## Quick Stats

- **Locked PRDs:** 4 (DA-01 v3.1 · DA-02 v2.1 · DA-03 v1.1 · DA-04 v1.2)
- **Figma node IDs audited:** 8 across 4 screens
- **Critical bugs found:** 6 systemic + ~40 per-screen P0
- **Components to build as design-system primitives:** 4
- **Estimated design effort to engineering-ready:** ~1 week focused

The PRDs are correct. The Figma needs a focused fix pass. After that, the suite ships clean.
