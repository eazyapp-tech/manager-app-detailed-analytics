# Manager App Detailed Analytics

Handoff documentation for the RentOk Manager App analytics suite: nine screens, one self-contained handoff sheet per screen. Each sheet defines every number, every window, every tap and every empty state in plain language, so a developer can build from it and a non-technical reader can follow it. No code references anywhere.

## The screens

| Screen | Handoff sheet | Reading maps | Status |
|---|---|---|---|
| Dues | [DA-01](DA-01%20—%20Dues/DA-01%20Dues%20—%20Handoff%20Sheet.md) | — | Closed v8, restructure queued |
| Collection | [DA-02](DA-02%20—%20Collections/DA-02%20Collection%20—%20Handoff%20Sheet.md) | — | Closed v12, restructure queued |
| Expense | [DA-04](DA-04%20—%20Expenses/DA-04%20Expense%20—%20Handoff%20Sheet.md) | [engineering](DA-04%20—%20Expenses/DA-04%20Cover%20Note%20—%20Nimit.md) | Closed v2 |
| Inventory | [DA-08](DA-08%20—%20Occupancy/DA-08%20Inventory%20—%20Handoff%20Sheet.md) | [engineering](DA-08%20—%20Occupancy/DA-08%20Cover%20Note%20—%20Nimit.md) · [design](DA-08%20—%20Occupancy/DA-08%20Cover%20Note%20—%20Ishika.md) | Closed v1 |
| Tenants | [DA-09](DA-09%20—%20Tenants/DA-09%20Tenants%20—%20Handoff%20Sheet.md) | [engineering](DA-09%20—%20Tenants/DA-09%20Cover%20Note%20—%20Nimit.md) · [design](DA-09%20—%20Tenants/DA-09%20Cover%20Note%20—%20Design.md) | Closed v2, the current standard |
| Old Tenants | — | — | Next up |
| Bookings | — | — | Not started |
| Complaints | — | — | Not started |
| Leads | — | — | Not started, newest addition |

## How to read this repo

**Building a screen?** Open its handoff sheet. It is the whole spec; you need nothing else. The engineering reading map tells you which sections are yours and what will bite.

**Working on design?** The design reading map lists your sections and every fix, in one place.

**Running the project?** [The tracker](00%20—%20Manager%20App%20Analytics%20Tracker.md) holds every locked cross-screen rule, the backfill queue, the session history and the next action. Read it first, every time.

**Writing a new sheet?** Copy [the template](_meta/_Handoff%20Sheet%20Template.md). It is the canonical spine: section order, table shapes, fixed phrases. Never start from a blank page.

## Rules

- The Obsidian vault is the source of truth; this repo carries the closed deliverables and the two working artifacts (tracker, template). Briefs, formula maps, archives and superseded sources stay in the vault.
- Screens land here as they close, sheet plus reading maps, at close-out.
- `[[Wikilinks]]` resolve inside Obsidian, not on GitHub, and are kept as written.
- Dues and Collection predate the current standard; both are queued for restructure to the template and will be replaced here when that happens.
