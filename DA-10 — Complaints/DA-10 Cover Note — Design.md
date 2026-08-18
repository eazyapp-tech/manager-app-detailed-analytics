---
title: DA-10 Cover Note — Design
date: 2026-08-18
tags:
  - rentok
  - complaints
  - detailed-analytics
  - cover-note
  - design
owner: Sanchay
---
# Complaints analytics: what design needs to do

[[DA-10 Complaints — Handoff Sheet]] is the spec. Your sections: **23** (the full fix list, 32 items), **19** (what each card shows when it is empty, healthy or broken), **4** (filter, colours, action bars), and **3**, where the info-icon copy is already written, one row per number, in the homescreen's voice.

Every number in the Figma file is placeholder. This section is not signed off, and most of the list below is work in progress rather than error.

## The six biggest

1. **Almost no states are drawn.** Two exist, both on Current Status, and the healthy one reads a good-news zero as a waiting room. Section 19 is the brief for all of them: one whole-screen state with an Add Complaint button, five healthy states with no button, four not-yet-in-use states, and the failed state on every card. On this screen an empty card is usually good news.
2. **The status donut contradicts itself.** The centre says Unresolved while a slice says Resolved. It shows every complaint open in the window; the centre names the total; the five slices sum to it.
3. **Two charts still carry a fifth bar labelled "Move-out"**, clipped past the right edge, copied from Tenants and never cleaned. Same leftover Inventory and Tenants had.
4. **The last ageing bar has no plus.** "31-45 Days" becomes 31+ days, and the three bars past a week turn red. Ageing ignores the filter and says "As of today" beside its title.
5. **Team Performance ranks by the wrong thing and draws the wrong chart.** It ranks by speed; it ranks by promises kept, "9 of 10 on time". Its Team tab is one stacked bar per person; it is two side by side, assigned against resolved, with an unassigned pair at the end. Its sample names are the engineering team.
6. **The filter chips drawn are not the set.** Today and This Week go; the options are Last 7 days, This Month, Last Month, Current FY, Custom, All Time.

## New to draw

The View all sheet. A time dropdown on Property Performance. Insight lines on Cost of Issues, Team Performance, Ratings and the View all sheet. An unassigned row on both Team tabs and an Unknown bar on Reporter. Room for three short lines: how many complaints the average is built on, that the overdue count is using the standard target, that the ageing bars are not tappable yet.

## The colour rule

Red only where somebody can be held to it: on this screen, a promise to a tenant broken. The overdue bar, the ageing bars past seven days, and a chip moving the wrong way. Nothing else. Cost of Issues draws its resolution time in red; that comes off.

## Yours to decide

Whether the face set has a face for no ratings, or the average and face both hide. Where the three short lines above fit, and which gives way if something must.

## One thing to delete

The older draft screen beside this one, with its Advanced Insights blocks. Everything of value has landed on the main screen. Before it goes, three things it still carries that the live screen lost and this sheet asks for: the "31-45 +" label, a Team legend reading "Assigned", and an insight line on the cost card.
