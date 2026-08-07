---
title: Handoff Sheet Template — canonical spine
date: 2026-08-07
tags:
  - rentok
  - detailed-analytics
  - template
  - reference
owner: Sanchay
---
# Handoff Sheet Template

**Every new sheet starts as a copy of this file.** Never from a blank page, never from memory of the format. Fill the placeholders, delete what the screen does not have, and never rename or reorder what you keep. The rules live in the tracker and the pipeline skill; this file is the shape.

Writing rules that bind every section: plain words, no code references, no naming what is broken in code (outcome plus test instead), no em dashes in prose, no jargon from the banned list, tables wherever content is parallel, platform statistics only in the appendix, warnings for things that bite a builder only.

---

# [Screen] — Handoff Sheet

One line: everything on the [Screen] analytics screen: what each number means, what window it covers, what happens when it is tapped, and what it shows when there is nothing to show.

## What is in here

Table: `| | Section | For |` listing every numbered section. Then three reading maps: backend, design, decisions-only.

## 1. Build status

Built, scaffolded, or absent. What already exists that matters. What the lists cannot yet do, said as what a user cannot yet see, never as a code critique.

## 2. Where this lives

Navigation path. The suite tab row: Financial (Dues · Collection · Expense) · People (Leads · Bookings · Tenants · Old Tenants) · Inventory · Complaints.

## 3. What every number counts

One term per heading, one or two lines each, `| Row | Meaning |` tables for parallel content. Inherit the suite dictionary; never coin a word the suite already owns (Live, Time-scoped, layer, under notice, booking, Past their date). End with **Words to be careful with** for any word that means something different one tab away.

## 4. How the screen behaves

Fixed sub-heads, in this order, dropping what does not apply:

- **The time filter** — the options, the default, and the two kinds of number: `| Kind | Meaning |` for **Live** (always the current snapshot, says "as of today" on its face) and **Time-scoped** (counts inside the window the filter picks), then a worked example with real dates.
- **What every number does on every filter setting** — the grid: every number down the side, the period settings and any forward setting across the top, every cell filled. Nothing on a forward setting ever invents an event.
- **Periods that have not finished** — compare against the same elapsed days of the previous period, marked as unfinished.
- **Periods that have not happened yet** — confirmed facts only; no chips, no action bars.
- **Change chips** — which numbers carry one, which direction counts as good per number, nothing on All Time.
- **When a number is red** — red only where somebody can be held to it (if the screen needs the rule).
- **The action bar** — names the problem, opens the filtered list where the work happens; never at zero (if the screen has one).
- **Loading, failure, sorting, entry** — the suite rules, restated in one paragraph.

## 5 to N. The cards

One numbered section per card, in screen order. Each: what it shows (`| Row | Meaning |` or `| Tile | Counts | Kind |`), what ignores the filter and says so on its face, the healthy state if zero is good news, and at most one warning if something genuinely bites.

## N+1. What each number opens

Fixed sub-heads: **The rules** (a drill filters a list, never re-scopes the screen; a number opens the list of the kind of record it counts; the destination opens on the same window and the back control names this screen; records add back to the number tapped; rates and averages are not tappable) · **The table** (`| Number | Opens | Ready? |`, verdicts ✅ ⚠ ❌, every ❌ named as filter-to-add or fact-not-recorded) · **What the destination says when you arrive** (name the slice, show the active filter, name the filter in the empty state).

## N+2. Who can see this

The permission rule: each analytics tab follows the permission of the records it describes. The Restricted lock copy, word for word. Narrowed access still adds up.

## N+3. What each card shows when it is empty, healthy or broken

**Not set up** (whole screen, one state, one CTA) · **Healthy** (zero as good news, no CTA) · **Empty** (nothing yet) · **Not recorded** (draws what exists, states its coverage) · **Failed** ("Couldn't load this" plus Retry, never a healthy message, never a zero).

## N+4. What this screen is not

Each exclusion with its reason, one line each.

## N+5. Build guidance

Numbered. Every non-obvious rule as an outcome plus a *Test it:* line QA can run. Never a description of what existing code does wrong.

## N+6. Open items

Numbered, each with an owner. "None outstanding" when closed; never ambiguous.

## N+7. Design file: what needs fixing

Four groups, one numbering end to end: **Wrong** · **Missing** · **Remove** · **Decide**.

## Where the measured figures came from

`| Measured | Result | What it decided |`. Every platform figure a decision rests on lives here and nowhere else.
