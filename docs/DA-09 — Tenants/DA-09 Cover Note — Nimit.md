---
title: DA-09 Cover Note — Nimit
date: 2026-08-07
tags:
  - rentok
  - tenants
  - detailed-analytics
  - cover-note
owner: Sanchay
---
# Tenants analytics — reading map

[[DA-09 Tenants, Handoff Sheet]] is the whole spec. This note says which parts are yours.

**Read:** section 23 (Build guidance, every trap has a test) → section 3 (the vocabulary) → sections 5–19 (the cards and what each number opens). **Skip:** 21 and 25, design's.

## The five things most likely to bite

1. **Every number is either Live or time-scoped.** Live always shows now, on every filter setting; time-scoped counts what happened inside the filter window. Section 4 has the table and a worked example. Five of the seven tiles are Live. *Test:* on an unchanged property, switching the filter changes only the time-scoped numbers.
2. **Under notice, approved eviction and past-their-date all sit inside Active.** 100 living, 12 under notice → Active shows 100. Must match the tenant list, the homescreen and Inventory.
3. **A leaving date is requested when a notice is raised, confirmed when approved.** Departure counts use confirmed only; Upcoming Eviction shows both, marked apart.
4. **Move-outs counts everyone who left, however their leaving was noted.** A property with known departures never shows zero, and the Old Tenants list must return the same people (open item 5).
5. **Every ❌ in the drill table is a filter to add, not history to start keeping.** All dates and states are already kept in full.

## Two things to size deliberately

**Police verification** is on file for a small minority, so at the seven-day deadline most tenants are overdue on day one. The card is built for that: the honest total, plus the workable queue of recent joiners. Not a bug.

**Agreement end dates are assumed at eleven months** wherever no duration is recorded, which is most tenants. The card says so in one line. If that assumption is wrong for reporting, say so before build.

Every figure behind a decision is in the appendix, measured against production. Open items: five, none blocking a start.
