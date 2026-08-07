---
title: DA-01 Dues — Pitch (V0.3)
date: '2026-05-13'
tags:
  - rentok
  - pitch
  - shape-up
  - operator-brief
  - dues
status: Living document · v0.3 (critique pass applied)
owner: Sanchay (PM)
co_signers: <Eng lead — TBD> · <Design lead — TBD> · sign before kickoff
designer: <name TBD — must be assigned before week 1>
figma_locked_by: end of week 2
time_budget: 6-week build cycle
companion_to: DA-01 Dues Detailed Analytics (legacy PRD V3.4 — historical), DA-01 Build Sheet V2 (engineering)
---

> [!INFO] What this is
> A **plan** for the Dues screen — written from the PG owner's point of view, before design. This is what the PM wants the screen to *do* for the operator, not how it should look.
>
> **What this is NOT:** an engineering spec or a design doc. Those live in `[[DA-01 Build Sheet]]` and Figma.
>
> **Status:** Living doc. No approval gates. If something changes mid-cycle, update this doc — don't freeze it.
>
> **About this version:** V0.3 was reverse-engineered from PRD V3.4 + Build Sheet V2 to retrofit a new front-of-lifecycle format. Future plans (DA-02 onward) will be written *before* design, not after.

---

# DA-01 Dues — Plan

## In one line

Rajesh[^1] (PG owner) loses roughly 8% of monthly rent[^2] to forgotten unpaid bills. This screen gives him a daily *"what money is stuck, where, and who do I call first"* answer at one glance — before the chai is over. 6-week build to ship V1.

[^1]: "Rajesh", "Priya", "Meena" are composite personas drawn from ~6 field interviews (Mar–Apr 2026) plus support-ticket review. Names anonymized; details are an amalgam, not any single operator. A "real-named" interview list (e.g., Suresh from Marathahalli, Anita from Kothrud) is in the operator-research folder.

[^2]: PM estimate. Not statistically validated. Worth confirming with a 20-operator survey during the cycle — but directionally what every operator we spoke to agreed with.

---

## The operator

**Rajesh.** Single-property PG owner. 35–50 years old. Hindi-first. Runs a 20–40 bed property in Bengaluru, Pune, or Jaipur. Knows every tenant by face and first name — but can't remember who owes what.

**A typical morning:**
- 8:30am — opens the app over chai before driving to the property
- Wants to know: *"Aaj kis-kis se lena hai, aur sabse pehle kiska phone ghumaun?"* (Who do I need to collect from today, and whose number do I dial first?)
- What he does today: checks a paper register, scrolls WhatsApp, calls his staff
- What he leaves with: a mental list of 3–5 names to chase by 11am

**Secondary and adjacent operators (one-liners — bet is built for Rajesh):**
- **Priya** — multi-property owner (2 or 5+ properties[^3]). Same morning ritual, scans across portfolio at month-end. Gets a per-property comparison row.
- **Meena/Ramesh-bhaiya** — collections coordinator hired by Rajesh or Priya[^4]. Works off a call list the screen feeds.

[^3]: In Bharat PG context, operators tend to cluster at 1 property or 5+; the 2-4 range is rare operationally. Worth checking in user research before designing for the middle.

[^4]: Field-check pending: in PG context this role is more often male (Ramesh-bhaiya) than female. Update persona name after confirming.

---

## The problem

Three pains, observed in the field:

1. **Forgotten unpaid bills cost about 8% of monthly rent.** Rajesh remembers the big tenants who haven't paid but loses track of the small 1–2 day cases — exactly the ones easiest to collect if caught early.
2. **It takes 20+ minutes to figure out the day's call list.** Most of that is cross-referencing register, WhatsApp, and staff — not actual decision-making.
3. **No sense of trend.** Rajesh can see "what's stuck today" but can't answer "is it getting better or worse than last month?" without doing math in his head.

**Why this matters now:** This is core to RentOk's promise. Without it, operators churn after 2–3 months because the app doesn't deliver on the basic "less stuck money, less work" pitch. The next 7-screen analytics suite is anchored on this — DA-02 through DA-07 reuse its patterns. Get this one wrong and the next six inherit the mistake.

---

## What Rajesh does today (with or without us)

- **Paper register.** One row per tenant. Updated 1–2 times a week. Often a month behind.
- **WhatsApp threads.** Tenants ping when they pay. Rajesh tries to remember which ones.
- **Calling staff.** *"Bhai, X tenant ka payment aaya kya?"* Burns 10–15 min/day.
- **Memory.** Works for tenants he sees daily. Misses bookings and tenants who've moved out.

The screen we ship has to beat all four: **faster** than the register, **more complete** than memory, **more current** than WhatsApp, **less effort** than calling staff. If it doesn't beat all four, it won't replace any of them.

---

## Time budget

**6-week build cycle.** Includes backend changes, screen build, and handoff to the Hindi localization team. If we can't ship V1 in 6 weeks, we push to the next cycle. We do **not** stretch the cycle to fit feature creep.

### Backend tasks — stack-ranked by critical path

| Rank | Task | Why critical | If it slips |
|------|------|--------------|-------------|
| 1 | Widen the unpaid-bill filter to include moved-out tenants (HB6) | Biggest source of stuck money is moved-out folks; without this, screen looks incomplete from day one | **Ship V1 without the moved-out segment**, label clearly, target V1.1 |
| 2 | Reconcile homepage tile vs screen total (HB7) | Trust-killer if numbers disagree (see Reconciliation Gate below) | **Hide the divergent homepage tile** for the cycle; do not ship with two contradicting numbers |
| 3 | Add the how-long-overdue query (HB1 / aging buckets) | Powers Q6 (urgency-band breakdown) | **Ship V1 with simple "overdue / due today / upcoming" three-bucket view**; defer fine-grained bands to V1.1 |
| 4 | Fix widget self-add permission gap (CSB-8) | Security; not user-facing for V1 | **Push to next cycle** (security review can happen in parallel) |

Owner: **Jatin** (Sr. Backend). If two slip simultaneously, the call is: **always sacrifice fine-grained aging buckets first, then moved-out segment**. Never sacrifice tile reconciliation.

---

## The questions Rajesh needs answered

Ordered in the sequence Rajesh runs through at 8:30am — not in the order the screen would lay them out. Each question gets a one-line answer describing the *information* we need to give him. **How** that information shows up is the designer's call.

1. **How much money is stuck right now, and is it getting better or worse?**
   → One total amount, glanceable. Plus a small signal showing whether it's higher, lower, or about the same as last month.

2. **How urgent is each chunk?**
   → The total split into urgency bands — what's overdue, what's due today, what can wait. Most urgent surfaced first.

3. **Who owes it — active tenants, future bookings, or tenants who've moved out?**
   → Three counts. Each leads to a different action path. Moved-out folks especially must not be hidden, since they're often the biggest leak.

4. **Which property is losing the most money?** *(only for Priya — multi-property)*
   → A per-property comparison row. Skip this for Rajesh — he has one property.

5. **What types of bills are running late?**
   → A split by bill type — rent, electricity, deposits, food. So Rajesh knows whether to chase tenants or fix his electricity-bill workflow.

6. **How long has each been overdue?**
   → Numeric bands: **1–7 days**, **8–15 days**, **16–30 days**, **31–60 days**, **60+ days**. Plain numbers; copywriter to add operator-Hindi labels alongside (e.g., *abhi-abhi late*, *do hafte late*, *mahina hone wala*, *do mahine late*, *purana bakaya*) — to be validated with 3 operators before lock.

7. **Who are my top 5 to call right now?**
   → A short, sorted list. Sorted by some sensible default (biggest + most overdue), but Rajesh decides who he calls.

8. **For a time period I pick, how is my collection performance?**
   → When Rajesh picks a date range (this month, last month, custom), the breakdowns and lists slice to that range. The main "money stuck right now" number stays live — it's always today.

9. **One specific tenant — what's their full story?**
   → Tapping a name takes Rajesh to the tenant's existing detail screen. No new screen to build here.

These 9 questions are the operator's spec. The screen answers them. **How** is the designer's job.

---

## What success looks like

**The one thing that matters:**
> Rajesh can identify his top 3 people to call within **5 seconds** of opening the screen — no scrolling, no tapping, no filtering.

**What we can't break:**
- The numbers on screen match Rajesh's own register to the rupee. Zero divergence.
- A Hindi-first operator can use the screen without needing to read any English.

### Reconciliation gate (pre-ship, non-negotiable)

This is the trust gate. If we ship with the homepage Dues tile disagreeing with the screen total, operators will spot it in week 1 and trust never lands.

Before ship, three things must be green:

1. **One source of truth named.** The aggregator behind the screen total is the canonical number. The homepage tile must either consume the same aggregator or be hidden for the cycle. Decision owner: **Jatin + Sanchay, week 1**.
2. **Reconciliation test passing.** One engineer owns a test that compares: screen total ↔ homepage tile ↔ a sample of 5 real properties' register sums. Must match to the rupee for all 5. Owner: **<eng — TBD>, run by end of week 5**.
3. **Rollback plan.** If reconciliation fails post-launch, the screen returns a "data refresh in progress" state, not a wrong number. Owner: **<eng — TBD>**.

**When to stop and reconsider:**
- **Mid-cycle (week 2):** if HB7 tile reconciliation isn't on track → escalate; do not ship with two contradicting numbers anywhere in the app.
- **Mid-cycle (week 2):** if HB6 (moved-out filter) isn't approved → ship V1 without that segment per the stack-rank above. Don't stretch the cycle.
- **Post-launch (week 0 baseline required):** measure today's Dues-related support ticket rate in week 0. If that rate **more than doubles** post-launch, trust is broken — stop adding features and fix the data first.

---

## What we're NOT building this cycle

- **No AI / predictive defaulter scoring.** Heuristics and clear data only. AI is V2 or V3.
- **No automated reminder sending.** Sending a reminder is a manual tap. Automation lives in a separate flow.
- **No invoice generation or editing.** Already in the Tenant module.
- **No portfolio summary for Priya.** She gets a per-property row, not a roll-up dashboard.
- **No "how to talk to tenants" tips.** That's wisdom, not data.
- **No deposit settlement.** That's Move-Out territory (DA-06).
- **No recording payments inside this screen.** Tap a name → existing Record Payment flow.
- **No notifications or push.** See the chai test below.

---

## The chai test

> Would Rajesh want this notification / popup / badge / animation at 8:30am with chai in one hand?

For V1: **the screen is something Rajesh chooses to open, not something that pings him.** No push. No popups. No red dot badges. No "5 unread dues!" nudges. If he doesn't open the app today, that's his call — not a problem for us to engineer around.

If any feature in this cycle violates this, cut it.

---

## Traps we've thought through

Pre-decided so engineering doesn't burn a week wrestling with them:

- **The main number vs the period filter.** The headline "money stuck right now" is always today's number — it does **not** change when Rajesh picks "last month." The period filter only slices the breakdowns and lists below it. Yes, this is mildly weird (main number sits still while the chart below changes), but it's the right answer: Rajesh wants to know what's stuck *right now*, regardless of which period he's reviewing. Worth a 3-operator usability check (see Validation Schedule).

- **Moved-out tenants must be included.** Today the filter excludes them. They're often the biggest source of stuck money (deposit unsettled, last month's rent unpaid). Backend task ranked #1 above. If it slips, ship without that segment and label clearly.

- **Partial-paid handling — no new work needed.** Production already shrinks the unpaid amount when a partial payment lands (instead of splitting the bill). The screen inherits this. We do *not* try to model "this bill is 60% paid" — that's a separate, much bigger feature.

- **Don't try to fix the wider data plumbing.** Multiple aggregators across the app disagree by small amounts; fixing all of them is tempting but out of scope. The Reconciliation Gate above covers the one alignment we MUST do. Everything else is flagged as follow-up.

- **Compliance: nothing new exposed.** Same tenant data Rajesh already sees in the Tenant module. No new fields, no new PII surface. If that changes mid-cycle, flag it.

- **Reversibility: low risk.** All backend changes are widening filters and adding query support — additive, no destructive migrations. Worst case, V1 ships and we roll back the screen; the data layer stays.

---

## Risks

- **Will Rajesh use it?** HIGH confidence yes. Field interviews are unambiguous. Operators ask for this daily.
- **Will Rajesh understand it?** MEDIUM. Hindi-first design, numerical literacy varies, words like "outstanding" or "aging" don't translate cleanly. Mitigation: plain operator-vocabulary copy, big readable numbers, no English-only jargon, glossary in the design system. Validation scheduled with Tamil/Telugu/Kannada operators too.
- **Can we build it?** MEDIUM-HIGH. Four backend changes (now stack-ranked above). Documented in `[[DA-01 Build Sheet]]` with file:line citations. None speculative — but each is one more thing that could slip.
- **Is it worth it?** STRONG. Core RentOk value prop. Pays for itself in retention if it cuts the 8% leak by even a third.

---

## Validation scheduled this cycle

Not "open questions" — concrete validation tasks with owner and week. These don't block kickoff but **must** be settled before launch.

| # | What to validate | How | Owner | Week |
|---|------------------|-----|-------|------|
| V1 | "Main number always-live + period filter slices below" — is this weird, or does Rajesh get it? | 3-operator usability test with prototype | Design lead | Week 3 |
| V2 | Aging-band labels in operator-Hindi (Q6) — do *abhi-abhi late*, *do hafte late*, etc. land? | Same 3-operator session | Design lead + copywriter | Week 3 |
| V3 | Bharat-language coverage — does the screen work for Tamil, Telugu, Kannada operators? | 1 operator per language | Localization lead | Week 5 |
| V4 | Moved-out segment visibility — do operators want a separate cue near the main number, or is the segment row enough? | Same usability session as V1 | Design lead | Week 3 |
| V5 | Dues-related support ticket baseline | Pull 4 weeks of ticket data, tag dues-related | Support lead | Week 0 |

If any validation owner is unnamed by end of week 1, the validation slips. Flag it.

---

## Handoff checkpoints

To stop ownership leaking between PM → Design → Eng:

- **End of week 1:** designer assigned, baseline measured (V5), source-of-truth aggregator named.
- **End of week 2:** Figma frame `7837:99662` locked. Reconciliation test scaffolded. Mid-cycle kill decisions made (HB7, HB6).
- **End of week 3:** 3-operator usability tests done (V1, V2, V4). Copy locked in operator-Hindi.
- **End of week 5:** Reconciliation test green. Bharat-language testing done (V3).
- **End of week 6:** Ship.

If a checkpoint slips, the next checkpoint moves the same amount — we do not silently absorb slip.

---

## Related docs

- **Engineering spec (post-design):** `[[DA-01 Build Sheet]]` (V2, ticket-ready)
- **Legacy PRD (historical):** `[[DA-01 Dues Detailed Analytics]]` (V3.4 — kept as historical reference; not maintained for new features)
- **Codebase ground truth:** `[[_Ground Truth Field Map]]`
- **Design system + Figma:** frame `7837:99662` (designer to lock by end of week 2)
- **Cross-suite engineering blockers:** `[[DA-01 Dues Detailed Analytics#Cross-Suite Engineering Blockers]]`
- **Earlier versions:** `DA-01 Pitch.md` (V0.2)

---

## Decision log

| Date | Decision | Owner |
|------|----------|-------|
| 2026-05-11 | Main number stays always-live; period filter slices breakdowns and lists only. Moved-out tenants widened in via filter change. | PM |
| 2026-05-11 | ⓘ icon convention locked: single-tap opens a bottom sheet. No inline tooltip. No long-press. | PM + Eng |
| 2026-05-11 | Default filter at app open: Today (Live). | PM |
| 2026-05-13 | V0.2 — initial Pitch (retroactive extraction). | PM |
| 2026-05-13 | V0.3 — critique pass applied. Reconciliation gate, backend stack-rank, validation schedule, handoff checkpoints added. Q6 buckets to numeric + operator-Hindi. Hindi quote tightened. Personas footnoted as composite. | PM |
| 2026-05-13 | Backend critical-path order locked: moved-out filter > tile reconciliation > aging buckets > widget perm gap. | PM + Jatin |
| 2026-05-13 | If two backend tasks slip simultaneously: sacrifice fine-grained aging buckets first, then moved-out segment. Never sacrifice tile reconciliation. | PM + Jatin |

---

## Changelog

| Date | Change | By |
|------|--------|-----|
| 2026-05-13 | V0.2 — initial Pitch (retroactive extraction). | Sanchay |
| 2026-05-13 | V0.3 — critique pass applied (Shape Up + Bharat-operator + pre-mortem lenses). Major additions: Reconciliation Gate, backend stack-rank table, Validation Schedule with owners/weeks, Handoff Checkpoints. Major fixes: Q6 numeric bucket labels (dropped "lost cause maybe"), tightened Hindi quote (*phone ghumaun* not *call karna*), demoted Priya/Meena to footnotes, footnoted personas as composite, expanded "Why now" with propagation risk, restated post-launch kill criterion with week-0 baseline. Jargon strip: "leak/bleeding" → "stuck money", "defaulters" → "tenants who haven't paid", "Rabbit holes" → "Traps we've thought through", "kill signals" → "stop and reconsider", dropped Zerodha doctrine name-drop. | Sanchay |
