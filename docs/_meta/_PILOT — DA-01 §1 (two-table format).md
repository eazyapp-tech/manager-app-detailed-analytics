---
title: PILOT — DA-01 §1 Hero & MoM Chip (Two-Table Format) — V2
date: '2026-05-11'
tags:
  - rentok
  - pilot
  - build-sheet
  - format-experiment
status: PILOT V2 — corrected post spec audit
companion_to: DA-01 Build Sheet (V2) §1
supersedes: V1 (2026-05-08)
---

> [!INFO] PILOT V2 — Format Experiment, corrected
> This pilot was rewritten 2026-05-11 after the DA-01 spec audit corrected several assumptions. V1 had:
> - Wrong Hero universe (excluded old tenants)
> - Wrong partial-paid framing (proposed `status IN (0, 2)` widening — confirmed pointless)
> - Old ⓘ icon convention (tap=tooltip + long-press=BS) — replaced with single-tap → BS
>
> **Compare against:** [[DA-01 Build Sheet#1. Hero & MoM Chip]] (V2 9-column, post-audit)
> **Reviewing for:** does the two-table split feel cleaner and lighter to read?
> **Scope:** ONE section (DA-01 §1) only. Don't propagate until format approved.

---

# DA-01 — §1 Hero & MoM Chip

> **What this section answers:** the live total of unpaid bills + month-over-month comparison.
> **Endpoint:** EXTEND — `src/v1/homepage/service.ts:2005-2019` (Financials tile, canonical Hero source) + `src/v1/list_screens/dues/helpers.ts:52-110` (shared `buildBaseQuery`)
> **Default property context:** inherited from dashboard (single → single; multi → multi-property aggregate)
> **Default time context:** **always-live** — Hero number does NOT change with date filter. Period filter slices segments / breakdowns / worklist drills, not the Hero number itself.
> **Default filter at app open:** Today (Live) — matches production default at `homepage/service.ts:1982-1984`
> **Figma:** frame `7837:99662` (Financials → Dues container) — header, hero number, subtitle, ⓘ icon, MoM chip layouts
> **Cross-Suite Blockers:** CSB-1 (HeaderValidator bypass), CSB-2 (drill destination unauth), CSB-8 (widget self-added permission gap — security)
> **Hard Blockers:** HB6 (widen `t.status` to `IN (0, 1, 2)` — RESOLVED, single-line fix at `helpers.ts:61`); HB6b (partial-paid — RESOLVED no-op); HB7 (homepage tile alignment); HB9 (widget perm gap — CSB-8)
> **Eng note:** all amounts in **paise** (integer). Post-HB6 production filter: `i.status = 0 AND i.amount >= 1 AND i.is_active = 1 AND t.status IN (0, 1, 2)`. Partial-paid invoices are captured silently via destructive `i.amount` mutation at `payment.ts:2039-2048` — DO NOT widen to `status IN (0, 2)` or `amount - paid_amount` math (see HB6b).

---

## Table A — Figma owns

> **Purpose:** reference list of visible UI elements in this section. Build Sheet does NOT restate these.
> **QA checklist use:** does Figma contain all items here? Anything in Figma not here?

| UI Element | Notes |
|---|---|
| Header label "Dues (Live)" | Static H2; primary text colour |
| Hero big number (₹X) | Display-large; primary colour |
| Hero subtitle text ("Across N bills") | Secondary text style below number. "Bills" not "invoices" — operator vocabulary. |
| Multi-property suffix ("· across N properties") | Appended to subtitle when condition fires (logic in Table B) |
| Old-tenant contribution indicator | Designer's call — operator must see how much of Hero is from moved-out tenants. Default: rely on Population row's Old Tenants row treatment. Designer may add chip / inline split / other treatment. |
| ⓘ icon | Right of header; primary colour. **Single-tap → bottom sheet** per Generation Spec §15. No tooltip. No long-press. |
| MoM chip — pill + ▲/▼ + colour + bold % | Pill shape, ▲ or ▼ arrow per sign, **▼ green / ▲ red (INVERTED for Dues**: down = good); bold % styling. Confirm with designer that semantics differ from DA-02 Collections. |
| Bottom sheet style (ⓘ tap destination) | Global BS component. Backend serves `bottomsheet_title` / `bottomsheet_subtitle` payload. |

**Frame:** `7837:99662` (single frame for entire §1).

**Items NOT yet locked in Figma** (designer ASK list):
- Setting-Up state's effect on MoM chip (force-hide overlay)
- Loading skeleton for hero (specific shimmer pattern)
- Error state styling (network failure on hero)
- Visual indication for old-tenant contribution near Hero (per designer's call on HB6)

---

## Table B — Build Sheet owns

> **Purpose:** formulas, permissions, drill data context, edge cases. What Figma can't show.
> **Row rule:** one row per atomic unit (its own formula OR drill OR permission). Decorations fold into Definition cell sub-bullets.

| Component | Status | Task | Plain Formula | Tech Formula | Permission | Plain Drill | Tech Drill |
|---|---|---|---|---|---|---|---|
| Hero | EXTEND | Total Dues number (+ subtitle + multi-prop suffix + ⓘ tap behaviour + old-tenant indication) | The total amount of unpaid bills across the user's selected properties, **as of right now**. The number does NOT change when the user picks a different time period — period filter slices segments and worklist drills, not the Hero number. Includes bills from active, booking, AND moved-out tenants (the Population row's Old Tenants entry breaks down the moved-out contribution clearly).<br><br>**Sub-elements:**<br>• Subtitle: count of those bills (`COUNT(*)` over same query)<br>• Multi-property suffix: append "across N properties" when more than one property selected<br>• ⓘ icon tap: bottom sheet with plain-English explanation + GAAP framing ("Accounts Receivable"). Per Generation Spec §15 — no inline tooltip, no long-press<br>• Old-tenant visual indication: designer's call (see Table A) | Hero number: `SUM(i.amount) WHERE i.status = 0 AND i.amount >= 1 AND i.is_active = 1 AND t.status IN (0, 1, 2)` *(`i` = `invoices`; `status = 0` = Not Paid per entity comment — DO NOT widen to `status IN (0, 2)`; production NEVER writes `status = 2`. Partial-paid is captured because `i.amount` is destructively shrunk on partial payment at `payment.ts:2039-2048`. `is_active = 1` = active row, not soft-deleted. `t` = `tenants`. `t.status IN (0, 1, 2)` = active + booking + old cohorts — post-HB6 change at `src/v1/list_screens/dues/helpers.ts:61` from prior `IN (1, 2)`)*.<br><br>Subtitle count: `COUNT(*)` over same query.<br>Multi-prop suffix render trigger: `selected_properties > 1` (frontend only). | view: `can_view_invoices` OR `view_invoices_of_self_added_tenants` *(derived ctx flags from `src/v1/homepage/service.ts:380, 392`. ⚠️ Widget endpoint at `service.ts:132-138` does NOT currently enforce the self-added clause — security gap CSB-8; worklist enforces it at `service.ts:96`)* | → Live (today): The big number is not tappable. Tap the ⓘ icon → bottom sheet opens with plain-English + GAAP explanation.<br>→ Past period: Same as live — Hero number itself doesn't change; ⓘ behaves the same.<br>→ Future: Same as live. | → Live: ⓘ tap → BS. Number itself n/a.<br>→ Past period: SAME (always-live).<br>→ Future: SAME. |
| MoM Chip | BUILD | Comparison value (+ visibility rule) | Compare the current Hero total to the same-elapsed-days window in the prior month. For example, on May 8 with month-to-date selected, compare May 1–8 vs April 1–8 (8 days each). Per Generation Spec §14, "same-elapsed-days" is locked globally across DA suite.<br><br>**Sub-elements:**<br>• Visibility: hide chip if account is < 30 days old OR if same window last month was ₹0 (avoids divide-by-zero)<br>• Setting-Up override: when Setting-Up state is active (see §13), force-hide regardless of data<br>• Color INVERTED for Dues: ▼ green = down = good (collections rising = good, opposite of Dues) | Comparison value: `(today_total − prior_same_elapsed_days_total) / prior_same_elapsed_days_total × 100`. <br><br>**NEW BUILD** — prior-period computation pass not implemented in `src/v1/list_screens/dues/`. Phase 1 must add second-pass query running same hero filter against prior-elapsed-days window.<br><br>Visibility logic: `hide if account_age_days < 30 OR prior_same_elapsed_days_total == 0 OR setting_up_state == true`. | view: inherits hero visibility | → Live (today): Tap chip → bottom sheet shows prior-period numbers and the exact days being compared. (Single-tap → BS, per Generation Spec §15. No inline tooltip.)<br>→ Past period: Same as live — chip always compares current vs same-elapsed-days last month, regardless of dashboard period selection.<br>→ Future: Same as live. | → Live: ⓘ tap → BS (prior numbers + computation window).<br>→ Past period: SAME (always-live comparison).<br>→ Future: SAME. |

---

## Cross-checks (this section's contribution to global §16 Reconciliation Invariants)

- Hero is mode-independent (Total Dues number does not change when date filter changes) — strict equality across all `filter_meta.mode` values.
- Hero universe = Active (`t.status = 1`) + Booking (`t.status = 2`) + Old (`t.status = 0`) tenant contributions — Population breakdown rows must sum to Hero strictly (post-HB6).
- MoM chip independence — comparison is always vs same-elapsed-days-last-month, never period-adjusted.
- Cross-screen: Homepage Financials tile = Detailed §1 Hero number (same property scope, same filter universe). **HB7:** also align with Homepage `getHeaderStats` tile if applicable.

*(Full invariant table for DA-01 lives in [[DA-01 Build Sheet#16. Reconciliation Invariants]].)*

---

## What changed from PILOT V1 (2026-05-08)

- **Hero universe widened** — `t.status IN (0, 1, 2)` (was `IN (1, 2)`). Old tenants now part of Hero. Reconciliation invariants hold cleanly.
- **Partial-paid framing corrected** — V1 had `status IN (0, 2)` widening flagged as `[VERIFY]`. Audit found production NEVER writes `status = 2`; current `SUM(i.amount) WHERE status = 0` already captures partial outstanding via destructive `amount` mutation. **Hero formula stays `SUM(i.amount)`.** Removed the widening from Table B Tech Formula. Documented why in eng note.
- **ⓘ icon convention simplified** — V1 had "tap = inline tooltip; long-press = bottom sheet." Now: **single-tap → BS only.** Per Generation Spec §15. Removed long-press references.
- **Reconciliation cross-check** — V1 referenced "Homescreen Dues tile" generically. Now distinguishes Financials tile (canonical, always-live) from `getHeaderStats` tile (currently date-filtered, must align per HB7).
- **Status simplified** — Hero row was `EXTEND + [VERIFY]`. Now `EXTEND` — `[VERIFY]` removed because the spec audit resolved the question. MoM chip stays `BUILD` (still net-new prior-period pass).
- **Old-tenant indication** — V1 didn't address how operator sees old-tenant contribution. Now Table A flags it as a designer concern (default fallback: Population row's Old Tenants treatment). Table B Sub-elements lists it explicitly.

---

## Reviewer questions

1. **Does Table B feel light enough?** 2 rows for entire Hero & MoM section. Genuine ticket-able units, not decorations.
2. **Does Table A feel like a useful checklist?** Or noise? Compared to V1, Table A is consolidated (MoM pill+arrow+color+styling = 1 row, not 5).
3. **Sub-bullet structure in Definition cells** — does it render well in Obsidian? Need visual check.
4. **The `t.status IN (0,1,2)` widening is a real production change** — comfortable that Hero changes for existing operators on rollout? Should we add a one-time changelog message ("We've updated Dues to include moved-out tenants too — these now show in your dashboard")?

---

## My honest verdict

PILOT V2 reads cleaner than V1 because:
- The Hero row carries fewer sub-bullets (some decorations dropped to Figma table)
- No `[VERIFY]` clutter (spec audit resolved them)
- ⓘ convention is one line, not a dual-pattern explanation
- Status column is tighter (single value per row, not compounds)

PILOT V2 still has open trade-offs:
- Definition cell with 4 sub-bullets is dense — depends on Obsidian rendering
- Table A's "Items NOT yet locked in Figma" list assumes coordination with designer — not all teams have this loop
- The HB6 / HB6b / HB7 / HB8 / HB9 list in section header is long; could collapse into a small ID-only list with link out to the full HB table

If V2 reads well, we can propagate this format to remaining sections of DA-01 + all other 6 Build Sheets.

---

## Maintenance log

| Date | Change | By |
|------|--------|-----|
| 2026-05-08 | V1 — Initial pilot with two-table format proposal. | PM |
| 2026-05-11 | V2 — Rewritten post DA-01 spec audit. Corrected Hero universe (widened to `t.status IN (0,1,2)`), removed pointless partial-paid widening, simplified ⓘ convention to single-tap → BS, removed `[VERIFY]` tags that audit resolved, added HB7/HB8/HB9 references, distinguished Financials tile vs `getHeaderStats` for HB7 alignment. | PM |
