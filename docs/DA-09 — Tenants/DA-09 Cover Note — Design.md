---
title: DA-09 Cover Note — Design
date: 2026-08-07
tags:
  - rentok
  - tenants
  - detailed-analytics
  - cover-note
  - design
owner: Sanchay
---
# Tenants analytics: what design needs to do

[[DA-09 Tenants — Handoff Sheet]] is the spec. Your sections: **25** (the full fix list), **21** (what each card shows when it is empty, healthy or broken), **4** (filter, colours, action bars), and **3** if you are writing the info-icon content.

Every number in the Figma file is placeholder. The fixes are structural.

## The six biggest

1. **The change chips sit on exactly the wrong two tiles.** Visible on Active Tenants and Active Bookings, which should never have one. Switched off on Approved Bookings and Notices Raised, which should. Swap them; delete the other five.
2. **All fourteen empty states carry truncated Complaints copy.** Section 21 has replacements, including healthy states: on this screen an empty card is often good news, and "no data" is the wrong thing to tell a property where nobody is leaving.
3. **Upcoming Eviction draws bars past its own edge.** The 31+ bar sits outside the card with one value where every bar carries two. A stray "Move-out" column sits further out; remove it (the same leftover Inventory had).
4. **Four cards hide Collection leftovers**: rupee rows, a "Received by" tab, "Paid by / Paid to" inside Property Wise, which is still internally named "Property Expense". Delete rather than hide.
5. **The two-tab Journey placed on screen is the build target.** Archive the three-tab Lifecycle variant with the Leads tab so nobody builds from it.
6. **Active is not a bar on Journey.** State the headcount once at the top; the three bars are what varies.

## New to draw

The Coming up filter chip and date picker. "As of today" on five tiles. The View all sheet. Coverage lines on five cards. The Already-expired bar. The tenure card (section 17). The net on Move-in & Move-out. Healthy, not-set-up, not-recorded, loading, failed and Restricted states.

## The colour rule

Red only where somebody can be held to it: an unmet obligation or a date already passed. Gaps with no consequence stay plain, and so does anything still inside its deadline. A property onboarded this morning must not open red.

## Yours to decide

Tile row overflow (scroll or wrap). Info icon and chevron placement: currently inconsistent, and no card has both.
