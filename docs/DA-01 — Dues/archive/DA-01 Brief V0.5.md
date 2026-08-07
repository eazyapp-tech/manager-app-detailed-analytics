---
title: DA-01 Dues — Pitch
date: 2026-05-10
tags:
  - rentok
  - pitch
  - shape-up
  - operator-brief
  - dues
status: Living document · v0.2
owner: Sanchay (PM) · co-signed by Eng lead + Design lead before kickoff
time_budget: 6-week build cycle
companion_to: DA-01 Dues Detailed Analytics (legacy PRD V3.4 — historical), DA-01 Build Sheet V2 (engineering)
---

> [!INFO] What this is
> A **Pitch** for the Dues screen — written from the PG owner's point of view, before design. This is what the PM wants the screen to *do* for the operator, not how it should look.
>
> **What this is NOT:** an engineering spec or a design doc. Those live in `[[DA-01 Build Sheet]]` and Figma.
>
> **Status:** Living doc. No approval gates. If something changes mid-cycle, update this doc — don't freeze it.
>
> **About this version:** V0.2 was reverse-engineered from the existing PRD V3.4 + Build Sheet V2 to retrofit the new Pitch format. Future Pitches (DA-02 onward) will be written *before* design, not after.

---

# DA-01 Dues — Pitch

## In one line

Rajesh (PG owner) runs his cash flow blind. Who to remind, when to nudge, what's quietly building up — none of it is visible without pulling reports, applying filters, or leaning on staff to investigate. So most owners run on gut, the rhythm of reminders breaks, and dues pile up unnoticed until they're hard to recover. This screen makes the dues picture legible in one glance: where money is stuck, what shape the problem is, what to chase or delegate. 6-week build to ship V1.

---

## The operator

This screen is for the owner / admin / supervisor in **scan mode** — not the person actually making the calls. The homescreen tells you *something's off*. The worklist tells you *who exactly to chase*. This screen sits in between: it tells you *what shape the problem is*.

**Rajesh.** Single-property PG owner. 35–50 years old. Hindi-first. Runs a 20–40 bed property in Bengaluru, Pune, or Jaipur. He may collect personally on a small property, or have a Meena/Ramesh-bhaiya doing it. Either way, when he opens *this* screen, he's not building today's call list — he's checking the **state of his dues**.

**When he opens it:**
- 2–3 times a week, or before handing work to staff, or when something feels off
- Wants to know: *"Mera paisa kahaan-kahaan atak raha hai, aur sabse zyada kahaan?"* (Where is my money getting stuck, and where is it worst?)
- Leaves with: a picture of where dues are concentrated — and, if he has staff, a delegation call (*"Meena, you take the move-outs at Property A; main 60+ wale dekh leta hoon."*)

**Priya** — multi-property owner (5+ properties). Scan mode is her *primary* mode; she can't go per-tenant across her portfolio. She lives in this view.

**Meena / Ramesh-bhaiya** — adjacent. Receives the work that comes *out* of this screen (a filtered slice of the worklist handed off). Doesn't open this screen herself.

Any view, segment, or row on this screen drills into the corresponding filtered view of the existing worklist. This screen does not replicate the worklist's job — it points to the right slice of it.

---

## The problem

Three pains at the synthesis altitude — all flavors of the same black box:

1. **No idea who needs reminding, or when.** Some tenants need a nudge today. Some next week. Some have been quietly slipping for a month. Without seeing the cuts (urgency, type, segment), the owner either chases everyone (burns staff time) or chases by gut (misses pockets). The rhythm of reminders breaks.
2. **Information is locked behind filters, reports, and staff investigation.** To answer *"where is my money stuck right now"*, the owner has to scroll the worklist, apply multiple filters, pull reports, or send staff to find out. 20+ minutes per attempt, and most owners don't bother — they go by gut. The information exists; it just isn't legible.
3. **No sense of trend or rhythm.** Owner can see what's stuck today but can't tell whether the situation is getting better or worse than last month, or whether the same tenants keep slipping vs new ones. No way to spot a building problem before it's a hard one.

Dues that go unseen go unrecovered. The bet isn't to plug a leak — it's to make the picture legible. Recovery follows.

**Why this matters now:** Cash flow control is one of RentOk's core promises to owners. Without this screen the app fails on that promise — owners stay in per-tenant mode, can't see the shape of their dues, and churn after 2–3 months. The 7-screen analytics suite is anchored on this one — get it wrong and the next six inherit the mistake.

---

## What Rajesh does today (with or without us)

- **Scrolls the existing worklist and applies multiple filters trying to cluster mentally.** *"Bahut saare electricity dikh rahe hain… kuch move-outs bhi…"* Slow, gut-driven, error-prone.
- **Sends staff to investigate, or asks for a verbal summary.** *"Bhai, kahaan sabse zyada bakaya hai?"* Gets a list of names or a vague answer — not a synthesis.
- **Exports an Excel and filters there.** Power-user move. Most owners don't.
- **Goes by gut.** Most common. Owner *thinks* they know where the leak is. Often wrong.

The screen has to beat all four: **faster** than scrolling-and-mentally-clustering, **more complete** than gut, **more honest** than the staff summary, **less effort** than building a spreadsheet. If it doesn't beat all four, owners stay in gut mode.

---

## Time budget

**6-week build cycle.** That includes backend changes (widen the unpaid-bill filter to include moved-out tenants, add a how-long-overdue query, fix a permission gap on widgets), screen build, and handoff to the Hindi localization team.

If we can't ship V1 in 6 weeks, we push to the next cycle. We do **not** stretch the cycle to fit feature creep.

---

## The questions Rajesh needs answered

Ordered in the sequence Rajesh runs through when he opens the screen — not in the order the screen would lay them out. Each question gets a one-line answer describing the *information* we need to give him. **How** that information shows up is the designer's call. Any cut or row can be tapped to drill into the corresponding filtered view of the existing worklist.

1. **How much money is stuck right now, and is it getting better or worse?**
   → One total amount, glanceable. Plus a small signal showing whether it's higher, lower, or about the same as last month.

2. **How urgent is each chunk?**
   → The total split into urgency bands — what's overdue, what's due today, what can wait. Most urgent surfaced first.

3. **Who owes it — active tenants, future bookings, or moved-out tenants?**
   → Three counts. Each leads to a different action path. Moved-out folks especially must not be hidden, since they're often the biggest leak.

4. **Which property is losing the most money?** *(only for Priya — multi-property)*
   → A per-property comparison row. Skip this for Rajesh — he has one property.

5. **What types of bills are running late?**
   → A split by bill type — rent, electricity, deposits, food. So Rajesh knows whether to chase tenants or fix his electricity-bill workflow.

6. **How long has each been overdue?**
   → Numeric bands: **1–7 days**, **8–15**, **16–30**, **31–60**, **60+**. Plain numbers; copywriter to add operator-Hindi labels alongside. Cut-offs are the PM's first cut — worth one round of operator validation, but ship with sensible defaults.

7. **What's at the top of the stack right now?**
   → A short, sorted list (biggest × most overdue). For awareness — or to hand off to staff. Tapping any row drills into the filtered worklist. The screen does not try to *be* the call list; it points to it.

8. **For a time period I pick, how is my collection performance?**
   → When Rajesh picks a date range (this month, last month, custom), the breakdowns and lists slice to that range. The main "money stuck right now" number stays live — it's always today.

9. **One specific tenant — what's their full story?**
   → Tapping a name takes Rajesh to the tenant's existing detail screen. No new screen to build here.

These 9 questions are the operator's spec. The screen answers them. **How** is the designer's job.

---

## What success looks like

**The one thing that matters:**
> Rajesh can spot the biggest pocket of stuck money — or the worst-trending segment — within **5 seconds** of opening the screen. No filters, no reports, no staff calls. (If he then wants to act on it, one tap drills into that slice of the worklist.)

**What we can't break:**
- The numbers on screen match Rajesh's own register to the rupee. Zero divergence.
- A Hindi-first operator can use the screen without needing to read any English.

**When to stop and reconsider (kill signals):**
- **Mid-cycle:** if the backend change to include moved-out tenants isn't approved by end of week 2 → ship V1 without that segment and revisit in V2. Don't let one blocker stretch the cycle.
- **Post-launch:** if operators report "the number is wrong" in more than 5% of support tickets → trust is broken. Stop adding features and fix the data first.

---

## What we're NOT building this cycle

- **No AI / predictive defaulter scoring.** Heuristics and clear data only. AI is V2 or V3.
- **No automated reminder sending.** Sending a reminder is a manual tap. Automation lives in a separate flow.
- **No invoice generation or editing.** Already in the Tenant module.
- **No portfolio summary for Priya.** She gets a per-property row, not a roll-up dashboard.
- **No "how to talk to tenants" tips.** That's wisdom, not data.
- **No deposit settlement.** That's Move-Out territory (DA-06).
- **No recording payments inside this screen.** Tap a name → existing Record Payment flow.
- **No notifications or push.** See chai test below.

---

## The chai test

> Would Rajesh want this notification / popup / badge / animation with chai in one hand?

For V1: **the screen is something Rajesh chooses to open, not something that pings him.** No push. No popups. No red dot badges. No "5 unread dues!" nudges. If he doesn't open the app today, that's his call — not a problem for us to engineer around.

If any feature in this cycle violates this, cut it.

---

## Rabbit holes (traps we've thought through, so the team doesn't fall in)

Pre-decided so engineering doesn't burn a week wrestling with them:

- **The main number vs the period filter.** The headline "money stuck right now" is always today's number — it does **not** change when Rajesh picks "last month." The period filter only slices the breakdowns and lists below it. Yes, this is mildly weird (the main number sits still while the chart below changes), but it's the right answer: Rajesh wants to know what's stuck *right now*, regardless of which period he's reviewing. Worth a 3-operator usability check before lock.

- **Moved-out tenants must be included.** Today the filter excludes them. They're often the biggest leak (deposit unsettled, last month's rent unpaid). The backend widen-filter task is small but real and must land before V1 — if not, ship without that segment and label it clearly.

- **Partial-paid handling — no new work needed.** Production already shrinks the unpaid amount when a partial payment lands (instead of splitting the bill). The screen inherits this. We do *not* try to model "this bill is 60% paid" — that's a separate, much bigger feature.

- **Aging buckets stay PM-picked for V1.** Cut-offs (1-7 / 8-15 / 16-30 / 31-60 / 60+) are first cut. Worth one operator validation pass, but if it doesn't happen we ship the defaults and tune in a later cycle — we do *not* hold V1 on this.

- **Don't try to fix the wider data plumbing.** Two homepage tiles (the dues tile and another aggregator) currently disagree on the total — by a small amount. Fixing that alignment is tempting but out of scope; flag it as a follow-up. This screen uses one source of truth and notes the divergence.

- **Compliance: nothing new exposed.** Same tenant data Rajesh already sees in the Tenant module. No new fields, no new PII surface. If that changes mid-cycle, flag it.

- **Reversibility: low risk.** All backend changes are widening filters and adding query support — additive, no destructive migrations. Worst case, V1 ships and we roll back the screen; the data layer stays.

---

## Risks (where this could go wrong)

- **Will Rajesh use it? — HIGH confidence yes.** Field interviews are unanimous. Operators ask for this view of their dues. Risk: low.
- **Will Rajesh understand it? — MEDIUM.** Hindi-first design, numerical literacy varies, words like "outstanding" or "aging" don't translate cleanly. Mitigation: plain-language copy in the operator's own vocabulary, big readable numbers, no English-only jargon, glossary in the design system. Test with Tamil/Telugu/Kannada operators too — not just Hindi.
- **Can we build it? — MEDIUM-HIGH.** Several real backend changes (widen the moved-out filter, add the how-long-overdue query, align homepage tiles, fix a widget permission gap). All documented in `[[DA-01 Build Sheet]]` with file:line citations. None are speculative — but each is one more thing that could slip.
- **Is it worth it? — STRONG.** Core RentOk value prop. The app's job is to make collections legible; this screen is where that happens. Retention follows.

---

## Open questions to test during the cycle

- **MoM comparison feels odd.** The main number is always today's, but the "vs last month" signal next to it isn't. Worth a 3-operator check — does the contrast confuse Rajesh, or does he get it?
- **Bharat-language coverage.** Test with Tamil, Telugu, Kannada operators. Some rupee-format assumptions may not localize cleanly.
- **Old tenant visual cue.** Moved-out folks show up in the segment breakdown. Worth checking if operators want an extra visual chip near the main number, or if the segment row is enough.

These are validation tasks for the cycle — they don't block kickoff but should be settled before launch.

---

## Related docs

- **Engineering spec (post-design):** `[[DA-01 Build Sheet]]` (V2, ticket-ready)
- **Legacy PRD (historical):** `[[DA-01 Dues Detailed Analytics]]` (V3.4 — comprehensive doc kept as historical reference; not maintained for new features)
- **Codebase ground truth:** `[[_Ground Truth Field Map]]` (entity fields, formulas, endpoints, permissions)
- **Design system + Figma:** frame `7837:99662` (designer to lock)
- **Cross-suite engineering blockers:** `[[DA-01 Dues Detailed Analytics#Cross-Suite Engineering Blockers]]`

---

## Decision log (living)

| Date | Decision | Owner |
|------|----------|-------|
| 2026-05-11 | Main number stays always-live; period filter slices breakdowns and lists only. Moved-out tenants widened in via filter change. | PM |
| 2026-05-11 | ⓘ icon convention locked: single-tap opens a bottom sheet. No inline tooltip. No long-press. | PM + Eng |
| 2026-05-11 | Default filter at app open: Today (Live). Matches current production. | PM |
| 2026-05-13 | Pitch created from retroactive extraction of PRD V3.4 + Build Sheet V2. | PM |
| 2026-05-10 | V0.2 — language stripped of jargon, Q+A intent answers added, Rabbit Holes section added per Shape Up. | PM |

---

## Maintenance log

| Date | Change | By |
|------|--------|-----|
| 2026-05-13 | v0.1 — Initial Pitch. Retroactively extracted from existing PRD V3.4 + Build Sheet V2. First instance of new Shape Up-style Pitch format adopted post-PM-doc-research. | Sanchay (PM) |
| 2026-05-10 | v0.2 — Critique pass applied. Added Q+A intent answers per question. Added Rabbit Holes section. Dropped Razorpay-heavy 3-Solutions / Recoverable-Reversible / Compliance subsections (folded as one-liners into Rabbit Holes and Risks). Fixed Hyderabad → Jaipur (Hindi-belt). Trimmed metrics to 1 primary + 2 guardrails + 2 kill. Added mid-cycle kill criterion. Cited 8% leak stat as PM estimate with source. Stripped jargon (TL;DR, Hero, MoM, GAAP, Appetite, Kill criteria, Drill, Outstanding, etc.) — operator-friendly language throughout. Acknowledged retroactive extraction in body. | Sanchay (PM) |
| 2026-05-13 | v0.3 — sidecar experiment (`DA-01 Pitch V0.3.md`) — overbuilt with reconciliation gate / validation tables / handoff checkpoints. Shelved as too much; smart team doesn't need it spelled out. | Sanchay (PM) |
| 2026-05-13 | v0.4 — **Altitude reframe.** The Operator / Problem / What-Rajesh-does sections were worklist-coded (daily call-list builder over chai). Rewrote at the right altitude — this is a diagnostic *scan* screen for the owner/admin/supervisor, sitting between the homescreen (signal) and the worklist (per-tenant action). Frequency shifted from daily ritual → 2–3×/week or trigger-based. Priya promoted as primary scan-mode user; Meena demoted to "receives delegated slice." Q7 reframed: "top 5 to call" → "top of the stack" (drills to worklist). Success metric reframed: "spot top 3 to call in 5 sec" → "spot the biggest pocket in 5 sec." Added drill-down-to-worklist note in operator section + questions intro. In-one-line rewritten to drop "daily" + "who to call first." | Sanchay (PM) |
| 2026-05-13 | v0.5 — **Cash-flow framing pass.** Dropped the 8% rent-leak number from headline + footnote + Problem closing + Risks "is it worth it" (PM estimate, arbitrary, doesn't anchor the doc). Reframed the problem around running cash flow blind: not knowing who to remind / when, information locked behind filters + reports + staff investigation, no rhythm. In One Line rewritten around cash flow blindness. The Problem rewritten with the three pains mapped to the new framing + closing line "Dues that go unseen go unrecovered. The bet isn't to plug a leak — it's to make the picture legible." What-Rajesh-does bullet 1 sharpened to "applies multiple filters"; bullet 2 sharpened to "sends staff to investigate." Success signal kept "biggest pocket" anchor but appended "no filters, no reports, no staff calls" — inverting the current alternate. Why-this-matters-now reframed around cash flow control. Footnote `[^1]` dropped entirely. | Sanchay (PM) |
