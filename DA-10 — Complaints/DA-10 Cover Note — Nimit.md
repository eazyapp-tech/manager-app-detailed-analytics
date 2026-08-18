---
title: DA-10 Cover Note — Nimit
date: 2026-08-18
tags:
  - rentok
  - complaints
  - detailed-analytics
  - cover-note
owner: Sanchay
---
# Complaints analytics — reading map

[[DA-10 Complaints — Handoff Sheet]] is the whole spec. This note says which parts are yours.

**Read:** section 21 (Build guidance, every trap has a test) → section 3 (the vocabulary and the info-icon copy) → sections 5–17 (the cards and what each number opens). **Skip:** 19 and 23, design's.

## The five things most likely to bite

1. **Complaints and their history are stamped on two different clocks, five and a half hours apart.** Every duration on this screen crosses that seam. Uncorrected, one complaint in eight appears resolved before it was raised. *Test:* no complaint ever shows resolved, assigned or responded to before it was raised.
2. **A complaint's own record does not say when it was resolved, assigned or reopened.** All of that is read from its history, which begins May 2024. Reopened in particular is a status moving out of Resolved, never the reopen label, which has caught a few dozen events in all of history against thousands of real ones.
3. **Open Backlog and the Current Status donut share one population: complaints open at any point in the window, still open today.** A past month shrinks as its complaints get resolved. The homescreen's Issues card is ticketed to move to the same model; build the filter once and both agree.
4. **Overdue and SLA Rate use each complaint's own target time, sub-category first, then category, then the default: one day for food, cleaning, housekeeping, power and internet, seven for the rest.** A past-target complaint counts as missed whether or not it has been resolved, so the rate cannot be gamed by never resolving the worst one.
5. **Most numbers cannot open their own records today.** Section 17 names ten filters the complaints list needs; they ship with the screen by decision. Two more wait on facts nobody records yet: escalation, and a way to mark a complaint Closed.

## Two things to size deliberately

**Ratings.** Nearly two thirds of stored ratings are the value 6 on a five-star scale, written by something other than the app. The average counts one to five only; a naive average ships 4.99. *Test:* the screen can never show a figure above five.

**A property is a small place.** The typical property raises a handful of complaints a quarter. Every average and rate shows how many complaints it is built on, and reads a dash, never zero, when there is nothing to average.

Every figure behind a decision is in the appendix, measured against production. Open items: ten, none blocking a start.
