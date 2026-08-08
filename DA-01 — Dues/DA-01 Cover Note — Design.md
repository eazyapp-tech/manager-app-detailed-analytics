---
title: DA-01 Cover Note — Design
date: 2026-08-08
tags:
  - rentok
  - dues
  - detailed-analytics
  - cover-note
  - design
owner: Sanchay
---
# Dues analytics: what design needs to do

[[DA-01 Dues — Handoff Sheet]] is the spec. Your sections: **21** (the full fix list), **17** (what each card shows when it is empty, healthy or broken), **6 and 10** (the view all sheet and the Overdue Breakup card both need real design work, not just a copy fix), and **3** if you are writing the info-icon content.

## The view all sheet needs a fresh pass, not a fix

The current frame for this sheet (§6) is an old draft built on different vocabulary ("Invoice Breakdown," a different total format) and shouldn't be used as a reference. Build it fresh off the six Overview tiles plus this month's category and stay-duration rows, fully drillable, same as every other list on this screen.

## The six biggest fixes

1. **Change chip colours are inverted.** A rising, worse number is drawn green; it must be red, up is bad on this whole card. This is the kind of thing that reads as "already confirmed correct" if nobody checks the actual colour.
2. **Chips sit on four tiles; only two should carry one** (This Month's Due, Current FY Dues). Remove the other two, and add a neutral state for an unchanged number.
3. **Seven cards' empty states carry leftover Complaints copy** ("You're all caught up! New maintenance requests…"). §17 has the replacement per card, including which zeros are good news and should read that way, not as setup.
4. **The ageing chart draws four buckets; the spec is five**: 1–7 · 8–14 · 15–21 · 22–90 · 90+. The old fourth bucket splits into two.
5. **Two overflow bottomsheets need drawing wherever a category view exists**, not only for Added By: tapping "Others" on any category chart should open the same pattern, list of what's left out, each row opens its own filtered list.
6. **The Overdue Breakup card loses its dropdown entirely.** It's Live now, no time filter. Wherever the file still shows "This Month" or "All Time" on that card's header, remove it.

## New to draw

The **Coming up** filter option and its date picker, plus the screen's state on it: the billed-versus-collected card sitting out, the Live cards still saying "as of today", no change chips anywhere, and an empty state naming the window the manager picked rather than a generic one. This is a new setting for this screen, the other analytics screens already carry it.

## Smaller fixes, still real

Wrongly written money on the gauge and Deposit headlines ("₹15,00,000 L", a full number with a stray unit letter), format as ₹15L. The forecast card's chart still carries the axis label "Overdue Timeline," left over from a copied chart, it's forward-looking and needs its own label. The global filter chip reads "Today," it should show one of the five locked options. Two loose cards (a second Deposit Dues, a duplicate Dues Breakdown) sit parked outside the phone frames, housekeeping. Hidden "Paid by / Paid to" labels sit inside Dues by Property, leftover from a different card. Dev notes on the canvas still name the tenant-status labels this sheet rules out ("Under Eviction" among them); update or remove them.

## The colour rule

Red only where somebody can be held to it: overdue money. Money not yet due stays plain. A property onboarded this morning must not open red.

## Yours to decide

The gauge's treatment when one slice (usually Overdue) is nearly everything, platform data shows this is the common case, not the exception. Whether the 90+ ageing bar needs a scale treatment, given it dominates most properties by design, not by error.
