---
title: Handoff Sheet Template — canonical spine
date: 2026-08-08
tags:
  - rentok
  - detailed-analytics
  - template
  - reference
owner: Sanchay
---
# Handoff Sheet Template

**Every new sheet starts as a copy of this file.** Never from a blank page, never from memory of the format. Fill the placeholders, delete what the screen does not have, and never rename or reorder what you keep. The rules live in the tracker and the pipeline skill; this file is the shape.

**The test every sentence must pass, owner-stated:** a developer or designer new to the project reads it once and knows what to build, no PM beside them, no dictionary. A sentence that needs the author standing next to it is wrong, however accurate.

Writing rules that bind every section: plain words, no code references, no naming what is broken in code (outcome plus test instead), no em dashes in prose (watch spaced en dashes too, same tell, different character), no jargon from the banned list, tables wherever content is parallel, platform statistics only in the appendix, warnings for things that bite a builder only.

**GitHub readability (added after Dues v11):** every section in the TOC links to its own heading anchor. The tap matrix (N+1), the design-fix list (N+7), and the measured-figures appendix each collapse behind a GitHub `<details><summary>` block, since each is read start-to-end by exactly one audience and none needs to be visible by default for someone scanning the sheet. Leave a blank line right after `</summary>` or the markdown inside renders as raw text.

---

# [Screen] — Handoff Sheet

One line: everything on the [Screen] analytics screen: what each number means, what window it covers, what happens when it is tapped, and what it shows when there is nothing to show.

## What is in here

Table: `| | Section | For |` listing every numbered section, **each section name a markdown link to its own heading anchor** (`#3-what-every-number-counts`, GitHub's own slug: lowercase, strip punctuation, spaces to hyphens). Then three reading maps: backend, design, decisions-only.

## 1. Build status

Built, scaffolded, or absent. What already exists that matters. What the lists cannot yet do, said as what a user cannot yet see, never as a code critique.

## 2. Where this lives

Navigation path. The suite tab row: Financial (Dues · Collection · Expense) · People (Leads · Bookings · Tenants · Old Tenants) · Inventory · Complaints.

## 3. What every number counts

Open with one line defining the section's core noun (a due, a tenant, a bill). Then a **quick-terms table** (`| Term | Meaning |`) for every one-or-two-line definition with no rule attached. Anything carrying a real rule, an exclusion, a cross-reference, or a warning stays its own heading with prose below the table, never folded in just to shorten the section, that is how a rule gets lost. Inherit the suite dictionary; never coin a word the suite already owns (Live, Time-scoped, layer, under notice, booking, Past their date). End with **Words to be careful with** for any word that means something different one tab away.

## 4. How the screen behaves

Fixed sub-heads, in this order, dropping what does not apply:

- **The time filter** — the options, the default, where Custom stops, what the filter remembers across drills and launches, and the two kinds of number: `| Kind | Meaning |` for **Live** (always the current snapshot, says "as of today" on its face) and **Time-scoped** (counts inside the window the filter picks), then a worked example with real dates.
- **What every number does on every filter setting** — the grid: every number down the side, the five period settings folded into one column (they behave identically), the forward setting broken out, every cell filled. Nothing on a forward setting ever invents an event. **The cells use the suite's fixed phrases and no others:** "As of today" (Live) · "Counted inside the window" (moves with the filter; add "; its own dropdown can pick a different one" where a card has one) · "Each tile keeps its own fixed window; the filter does not change it" (fixed-window cards) · "From today onwards" (forward-looking cards) · "—" (nothing agreed to count). Kind labels are **Live** · **Time-scoped** (add ", window fixed" where the window is pinned) · **Forecast**, nothing else. **Banned on this axis, sweep for them at close-out:** follows, pinned, window-scoped, duration-scoped, period-scoped, or any freshly coined kind word. Plain sentences, the way you would say it aloud, win over compact jargon every time.
- **Periods that have not finished** — compare against the same elapsed days of the previous period, marked as unfinished.
- **Periods that have not happened yet** — confirmed facts only; no chips, no action bars.
- **Change chips** — which numbers carry one, which direction counts as good per number, nothing on All Time.
- **When a number is red** — red only where somebody can be held to it (if the screen needs the rule).
- **The action bar** — names the problem, opens the filtered list where the work happens; never at zero (if the screen has one).
- **Loading, failure, sorting, entry** — the suite rules, restated in one paragraph.

## 5 to N. The cards

One numbered section per card, in screen order. Each: what it shows (`| Row | Meaning |` or `| Tile | Counts | Kind |`), what ignores the filter and says so on its face, the healthy state if zero is good news, and at most one warning if something genuinely bites.

## N+1. What each number opens

Fixed sub-heads, **the tap matrix and everything below it inside a collapsed `<details>` block** (see GitHub readability, above), the section heading itself stays outside it: **The rules** (a drill filters a list, never re-scopes the screen, trend bars excepted; the destination follows the record's state, so different taps open different lists; the destination opens on the same window and the same properties, and the back control names this screen; records add back to the number tapped; rates and averages are not tappable) · **When the window changes what a tap shows** (the three-row rule: Live taps ignore the window; period taps carry it and the list shows people as they are today, naming any difference on arrival; forward-window taps re-route to where the people are today) · **The tap matrix** (`| You tap | What opens | Arriving filtered to | Ready? |`, grouped by card with a bold divider row per card, one row per tappable element, arrival filters in plain words, verdicts ✅ ⚠ ❌, every ❌ named as filter-to-add or fact-not-recorded) · **What the destination says when you arrive** (name the slice, show the active filter, name the filter in the empty state).

## N+2. Who can see this

The permission rule: each analytics tab follows the permission of the records it describes. The Restricted lock copy, word for word. Narrowed access still adds up.

## N+3. What each card shows when it is empty, healthy or broken

Opens with **The zeros, told apart**: one router table covering every situation that produces an empty-looking card, told apart on sight — never-set-up, onboarding (something confirmed, nothing arrived yet), the emptied bad zero (never dressed as setup), the in-window zero, the good-news zero, the not-recorded gap, the failure. Then the sub-sections: **Not set up** (whole screen, one state, one CTA) · **Healthy** (zero as good news, no CTA) · **Empty** (nothing yet) · **Not recorded** (draws what exists, states its coverage) · **Failed** ("Couldn't load this" plus Retry, never a healthy message, never a zero).

## N+4. What this screen is not

Each exclusion with its reason, one line each.

## N+5. Build guidance

Numbered. Every non-obvious rule as an outcome plus a *Test it:* line QA can run. Never a description of what existing code does wrong.

## N+6. Open items

Numbered, each with an owner. "None outstanding" when closed; never ambiguous.

## N+7. Design file: what needs fixing

Four groups, one numbering end to end: **Wrong** · **Missing** · **Remove** · **Decide**. **The whole list, inside a collapsed `<details>` block** (see GitHub readability, above), section heading stays outside it.

## Where the measured figures came from

`| Measured | Result | What it decided |`. Every platform figure a decision rests on lives here and nowhere else. **Inside a collapsed `<details>` block** (see GitHub readability, above), section heading stays outside it.
