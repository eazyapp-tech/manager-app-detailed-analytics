---
title: DA-01 Dues — Pitch (V0.6)
date: 2026-05-13
tags:
  - rentok
  - pitch
  - shape-up
  - operator-brief
  - dues
status: Living document · v0.6 (Linear / Shape Up bar pass)
owner: Sanchay (PM) · co-signed by Eng lead + Design lead before kickoff
time_budget: 6-week build cycle
companion_to: DA-01 Dues Detailed Analytics (legacy PRD V3.4 — historical), DA-01 Build Sheet V2 (engineering)
---

> [!INFO] What this is
> A **Pitch** for the Dues screen — written before design. What the PM wants the screen to *do*, not how it should look.
>
> **What this is NOT:** an engineering spec or a design doc. Those live in `[[DA-01 Build Sheet]]` and Figma.

---

# DA-01 Dues — Pitch

## In one line

Rajesh (PG owner) runs his cash flow blind. To find out where his money is stuck he scrolls the worklist, applies filters, pulls reports, or sends staff to investigate. Most don't bother — they go by gut, and dues pile up unseen. This screen makes the dues picture legible in one glance: where money is stuck, what shape the problem is, what to chase or delegate. **6-week build to ship V1.**

---

## The operator

The owner / admin / supervisor in **scan mode** — not the person making the calls. The homescreen says *something's off*. The worklist says *who exactly to chase*. This screen says *what shape the problem is*.

**Rajesh.** Single-property PG owner. 35–50, Hindi-first, 20–40 beds in Bengaluru / Pune / Jaipur. May collect personally on a small property or have a Meena / Ramesh-bhaiya doing it. He opens this screen 2–3×/week, or before handing work to staff, or when something feels off — not to build a call list, but to check the **state of his dues**. He leaves with a picture and (if staffed) a delegation call: *"Meena, you take the move-outs at Property A; main 60+ wale dekhta hoon."*

**Priya** — multi-property (5+). Scan is her *primary* mode. She lives in this view.

**Meena / Ramesh-bhaiya** — receives the filtered worklist slice handed off. Doesn't open this screen herself.

Any cut or row drills into the existing worklist (filtered). This screen does not replicate the worklist; it points to the right slice of it.

---

## The problem

Three pains, same root cause — the dues picture isn't legible:

1. **No idea who needs reminding, or when.** Some need a nudge today, some next week, some have been slipping for a month. Without the cuts, the owner chases everyone (wastes staff time) or chases by gut (misses pockets). The rhythm of reminders breaks.
2. **Information is locked behind filters, reports, and staff investigation.** Answering *"where is my money stuck right now"* takes 20+ minutes of scrolling, filtering, report-pulling, or staff legwork. The information exists; it just isn't legible.
3. **No trend or rhythm.** Owner can't tell whether things are getting better or worse, or whether the same tenants keep slipping. No way to spot a building problem before it's a hard one.

Dues that go unseen go unrecovered. The bet isn't to plug a leak — it's to make the picture legible. Recovery follows.

**Why now:** Cash flow control is one of RentOk's core promises. Without this screen, owners stay in per-tenant mode and churn at 2–3 months. The 7-screen analytics suite is anchored on this one — get it wrong and the next six inherit the mistake.

---

## What Rajesh does today

- **Scrolls the worklist + applies multiple filters, clustering mentally.** Slow, error-prone.
- **Sends staff to investigate, or asks for a verbal summary.** Gets names or a vague answer — not a synthesis.
- **Exports Excel and filters there.** Power-user move; most don't.
- **Goes by gut.** Most common. Often wrong.

The screen has to beat all four: faster than scrolling-and-filtering, more complete than gut, more honest than the staff summary, less effort than a spreadsheet.

---

## The bet

### Fat marker

```
┌──────────────────────────────────────────────┐
│  ₹ Money stuck right now    ↑/↓ vs last mo   │   Q1 — load-bearing
│  (always live; never sliced by period)       │
└────────────────┬─────────────────────────────┘
                 │
                 ▼
┌──────────────────────────────────────────────┐
│  Urgency bands                                │   Q2 — load-bearing
│  overdue · due today · due later              │
│  (most urgent first)                          │
└────────────────┬─────────────────────────────┘
                 │ tap →  worklist (filtered)
                 ▼

[ Period filter ]  ─── slices everything below ─►

  • Segment: active / future / moved-out          — Q3
  • Aging bands: 1-7 / 8-15 / 16-30 / 31-60 / 60+ — Q6
  • Top of stack (biggest × overdue)              — Q7
  • Bill types: rent / electricity / deposits     — Q5
  • Period performance                            — Q8
  • Property breakdown                            — Q4 (Priya only)

  Tap anything → worklist (filtered)
```

### Load-bearing vs stretch

**V1 MUST ship: Q1 + Q2.** The headline number + the urgency split. Together they answer 80% of *"what should I do today."* If we can't ship these two, we don't ship the screen.

**V1 stretches (ship if time allows): Q3, Q6, Q7.** Segment / aging / top-of-stack — these answer *"where exactly is the leak."*

**Cuttable first if scope tightens:** Q9 (free — just a link to existing tenant detail) → Q4 (Priya-only, defer) → Q8 (period perf, defer) → Q5 (bill types, defer). Start cutting from the bottom of the stretch list before touching Q3/Q6/Q7.

### What each question needs

Each line below describes the *information* — not the UI. **How** it shows up is the designer's call. Tapping any cut or row drills into the corresponding filtered view of the worklist.

1. **Money stuck right now + trend.** One total amount, glanceable. Plus a higher/lower/similar signal vs last month.
2. **Urgency split.** Three bands minimum: overdue · due today · due later. Most urgent first.
3. **Who owes it.** Three counts: active tenants / future bookings / moved-out. Moved-out must not be hidden — biggest leak source.
4. **Property comparison row.** Priya only. One row per property.
5. **Bill-type split.** Rent / electricity / deposits / food.
6. **Aging bands.** Numeric: 1–7, 8–15, 16–30, 31–60, 60+. Copywriter adds operator-Hindi labels.
7. **Top of stack.** Short sorted list (biggest × overdue). For awareness or staff handoff. The screen doesn't try to *be* the call list; it points to it.
8. **Period performance.** When the operator picks a date range, the cuts (not the headline) slice to it.
9. **Tenant drill.** Tapping a name → existing tenant detail. No new screen.

---

## Boundaries

**What we're NOT building this cycle:**
- No AI / predictive defaulter scoring (V2 / V3)
- No automated reminder sending (manual tap; automation is a separate flow)
- No invoice generation or editing (Tenant module)
- No portfolio summary for Priya (one row per property, not a roll-up)
- No deposit settlement (Move-Out, DA-06)
- No recording payments inside this screen (existing Record Payment flow)
- No notifications, push, badges, popups

**The chai test:** would Rajesh want this notification / badge / popup / animation with chai in one hand? V1 ships **no** interruptions. The screen is something the operator chooses to open, not something that pings him. Cut anything that violates this.

---

## Traps & risks

Decided in advance so engineering doesn't wrestle:

- **Main number is always live; period filter slices everything below it.** This will read mildly weird at first (the headline number stays still while the chart below changes). Right answer anyway — operator wants to know what's stuck *right now*, regardless of which period they're reviewing. Designer to explore one visual cue showing the headline is independent of the period filter. Don't hold V1 on it.
- **Moved-out tenants are widened in via the backend filter task.** Today the filter excludes them. If that one task slips, ship V1 without the moved-out segment and label clearly — never stretch the cycle for it.
- **Partial-paid: inherits production behavior.** Production already shrinks the unpaid amount when a partial payment lands. The screen inherits this — no new modeling.
- **Aging bands are locked for V1** (1-7 / 8-15 / 16-30 / 31-60 / 60+). Tune in V2 if operators disagree.
- **Two homepage tiles disagree by a small amount today.** Out of scope to align in this cycle. This screen uses one source of truth; flag the divergence as a follow-up.
- **Compliance / reversibility: low risk.** Same tenant data the operator already sees. All backend changes are additive (widen filters, add queries) — rollback is clean.

**The four risks:**

| Risk | Read | Mitigation |
|------|------|------------|
| Will operators use it? | HIGH yes | Field interviews unambiguous |
| Will they understand it? | MEDIUM | Operator-Hindi copy, big numbers, no English-only jargon, Tamil/Telugu/Kannada test |
| Can we build it in 6 weeks? | MEDIUM-HIGH | 4 real backend changes — moved-out filter, aging query, tile alignment, widget perm gap. All documented in `[[DA-01 Build Sheet]]` with file:line citations. Each is one more thing that could slip |
| Is it worth it? | STRONG | The app's job is to make collections legible; this screen is where that happens |

**Kill signals:**
- **Mid-cycle (week 2):** if the moved-out filter widen isn't approved → ship V1 without that segment.
- **Post-launch:** if operators report "the number is wrong" in support tickets at a rate that doubles the current Dues-related ticket baseline → trust is broken; fix the data before adding features.

---

## Footer

**Validation tasks** (don't block kickoff; settle before launch):
- 3-operator usability check on the main-number-vs-period-filter behavior
- Bharat-language testing (Tamil / Telugu / Kannada)

**Related docs:**
- Engineering spec (post-design): `[[DA-01 Build Sheet]]`
- Legacy PRD (historical): `[[DA-01 Dues Detailed Analytics]]`
- Codebase ground truth: `[[_Ground Truth Field Map]]`
- Design system + Figma: frame `7837:99662`
- Cross-suite engineering blockers: `[[DA-01 Dues Detailed Analytics#Cross-Suite Engineering Blockers]]`

**Key decisions locked:**
- Main number always-live; period filter slices below it only
- ⓘ icon: single-tap → bottom sheet only. No tooltip, no long-press
- Default filter at app open: Today (Live)
- Aging band cut-offs: 1-7 / 8-15 / 16-30 / 31-60 / 60+ — locked for V1
- V1 must-ship: Q1 + Q2. V1 stretch: Q3, Q6, Q7. Cuttable first if scope tightens: Q9 → Q4 → Q8 → Q5

**Changelog:**

| Date | Version | Change |
|------|---------|--------|
| 2026-05-13 | v0.1 | Initial Pitch — retroactive extraction from PRD V3.4 + Build Sheet V2 |
| 2026-05-13 | v0.2 | Critique pass — Q+A intent answers, Rabbit Holes section, jargon strip |
| 2026-05-13 | v0.3 | Sidecar experiment (`DA-01 Pitch V0.3.md`) — overbuilt with reconciliation gate / validation tables / handoff checkpoints. Shelved |
| 2026-05-13 | v0.4 | Altitude reframe — scan screen, not worklist. Priya promoted; drill-into-worklist anchor added |
| 2026-05-13 | v0.5 | Cash-flow framing — dropped 8% leak number; problem reframed around cash flow blindness, who/when-to-remind, locked-behind-filters/reports/staff |
| 2026-05-13 | v0.6 | **Linear / Shape Up bar pass.** Added fat-marker breadboard. Q1+Q2 named load-bearing; scope-cut ladder explicit (Q9 → Q4 → Q8 → Q5). Hedges resolved into decisions (aging bands locked, main-number visual deferred to designer). 15 sections collapsed to 7 + footer. Editorial pass — cut "synthesis altitude," "all flavors of the same black box," 3× "go by gut" repetition. Open Questions section killed (decisions promoted to Traps, validation moved to footer). Decision Log + Maintenance Log merged into Changelog. Risks collapsed to 4-row table. ~35% shorter than v0.5 |
