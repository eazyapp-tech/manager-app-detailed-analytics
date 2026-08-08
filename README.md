# Manager App Detailed Analytics

Handoff documentation for the RentOk Manager App analytics suite: nine screens, one self-contained handoff sheet per screen. Each sheet defines every number, every window, every tap and every empty state in plain language. No code references anywhere; a developer builds from it, a designer works from it, a non-technical reader follows it.

## Contents

- [Start here, by role](#start-here-by-role)
- [The screens](#the-screens)
- [Jump straight into a sheet](#jump-straight-into-a-sheet)
- [The rules every sheet follows](#the-rules-every-sheet-follows)
- [How this repo stays current](#how-this-repo-stays-current)

## Start here, by role

| You are | Open this | Then |
|---|---|---|
| **Building a screen** | Its handoff sheet, below | The engineering reading map tells you which sections are yours and what will bite |
| **Designing** | The design reading map, below | Every fix for your screen sits in one numbered list |
| **Running the project** | [The tracker](00%20—%20Manager%20App%20Analytics%20Tracker.md) | Every locked rule, the backfill queue, the next action. Read it first, every time |
| **Writing a new sheet** | [The template](_meta/_Handoff%20Sheet%20Template.md) | The canonical spine. Copy it; never start from a blank page |

## The screens

| # | Screen | Handoff sheet | Engineering map | Design map | Status |
|---|---|---|---|---|---|
| 1 | Dues | [DA-01 sheet](DA-01%20—%20Dues/DA-01%20Dues%20—%20Handoff%20Sheet.md) | [read me first](DA-01%20—%20Dues/DA-01%20Cover%20Note%20—%20Nimit.md) | [design map](DA-01%20—%20Dues/DA-01%20Cover%20Note%20—%20Design.md) | ✅ v12 · **the current reference** |
| 2 | Collection | [DA-02 sheet](DA-02%20—%20Collections/DA-02%20Collection%20—%20Handoff%20Sheet.md) | [read me first](DA-02%20—%20Collections/DA-02%20Cover%20Note%20—%20Nimit.md) | [design map](DA-02%20—%20Collections/DA-02%20Cover%20Note%20—%20Design.md) | ✅ v13 · the toggle screen |
| 3 | Expense | [DA-04 sheet](DA-04%20—%20Expenses/DA-04%20Expense%20—%20Handoff%20Sheet.md) | [read me first](DA-04%20—%20Expenses/DA-04%20Cover%20Note%20—%20Nimit.md) | — | ✅ v2 |
| 4 | Inventory | [DA-08 sheet](DA-08%20—%20Occupancy/DA-08%20Inventory%20—%20Handoff%20Sheet.md) | [read me first](DA-08%20—%20Occupancy/DA-08%20Cover%20Note%20—%20Nimit.md) | [design map](DA-08%20—%20Occupancy/DA-08%20Cover%20Note%20—%20Ishika.md) | ✅ v1 |
| 5 | Tenants | [DA-09 sheet](DA-09%20—%20Tenants/DA-09%20Tenants%20—%20Handoff%20Sheet.md) | [read me first](DA-09%20—%20Tenants/DA-09%20Cover%20Note%20—%20Nimit.md) | [design map](DA-09%20—%20Tenants/DA-09%20Cover%20Note%20—%20Design.md) | ✅ v2.1 · voice exemplar |
| 6 | Old Tenants | — | — | — | 🟡 next up |
| 7 | Bookings | — | — | — | ⬜ |
| 8 | Complaints | — | — | — | ⬜ |
| 9 | Leads | — | — | — | ⬜ newest addition |

## Jump straight into a sheet

Deep links into the sections readers ask for most.

| Section | Tenants (DA-09) | Inventory (DA-08) | Expense (DA-04) | Dues (DA-01) | Collection (DA-02) |
|---|---|---|---|---|---|
| What every number counts | [§3](DA-09%20—%20Tenants/DA-09%20Tenants%20—%20Handoff%20Sheet.md#3-what-every-number-counts) | [§3](DA-08%20—%20Occupancy/DA-08%20Inventory%20—%20Handoff%20Sheet.md#3-what-every-number-counts) | [§3](DA-04%20—%20Expenses/DA-04%20Expense%20—%20Handoff%20Sheet.md#3-what-every-number-counts) | [§3](DA-01%20—%20Dues/DA-01%20Dues%20—%20Handoff%20Sheet.md#3-what-every-number-counts) | [§3](DA-02%20—%20Collections/DA-02%20Collection%20—%20Handoff%20Sheet.md#3-what-every-number-counts) |
| How the screen behaves | [§4](DA-09%20—%20Tenants/DA-09%20Tenants%20—%20Handoff%20Sheet.md#4-how-the-screen-behaves) | [§4](DA-08%20—%20Occupancy/DA-08%20Inventory%20—%20Handoff%20Sheet.md#4-how-the-screen-behaves) | [§4](DA-04%20—%20Expenses/DA-04%20Expense%20—%20Handoff%20Sheet.md#4-how-the-screen-behaves) | [§4](DA-01%20—%20Dues/DA-01%20Dues%20—%20Handoff%20Sheet.md#4-how-the-screen-behaves) | [§4](DA-02%20—%20Collections/DA-02%20Collection%20—%20Handoff%20Sheet.md#4-how-the-screen-behaves) |
| What each number opens | [§19](DA-09%20—%20Tenants/DA-09%20Tenants%20—%20Handoff%20Sheet.md#19-what-each-number-opens) | [§14](DA-08%20—%20Occupancy/DA-08%20Inventory%20—%20Handoff%20Sheet.md#14-what-each-number-opens) | [§11](DA-04%20—%20Expenses/DA-04%20Expense%20—%20Handoff%20Sheet.md#11-what-each-number-opens) | [§15](DA-01%20—%20Dues/DA-01%20Dues%20—%20Handoff%20Sheet.md#15-what-each-number-opens) | [§13](DA-02%20—%20Collections/DA-02%20Collection%20—%20Handoff%20Sheet.md#13-what-each-number-opens) |
| Build guidance | [§23](DA-09%20—%20Tenants/DA-09%20Tenants%20—%20Handoff%20Sheet.md#23-build-guidance) | [§18](DA-08%20—%20Occupancy/DA-08%20Inventory%20—%20Handoff%20Sheet.md#18-build-guidance) | [§15](DA-04%20—%20Expenses/DA-04%20Expense%20—%20Handoff%20Sheet.md#15-build-guidance) | [§19](DA-01%20—%20Dues/DA-01%20Dues%20—%20Handoff%20Sheet.md#19-build-guidance) | [§17](DA-02%20—%20Collections/DA-02%20Collection%20—%20Handoff%20Sheet.md#17-build-guidance) |
| Open items | [§24](DA-09%20—%20Tenants/DA-09%20Tenants%20—%20Handoff%20Sheet.md#24-open-items) | [§19](DA-08%20—%20Occupancy/DA-08%20Inventory%20—%20Handoff%20Sheet.md#19-open-items) | [§16](DA-04%20—%20Expenses/DA-04%20Expense%20—%20Handoff%20Sheet.md#16-open-items) | [§20](DA-01%20—%20Dues/DA-01%20Dues%20—%20Handoff%20Sheet.md#20-open-items) | [§18](DA-02%20—%20Collections/DA-02%20Collection%20—%20Handoff%20Sheet.md#18-open-items) |
| Design file fixes | [§25](DA-09%20—%20Tenants/DA-09%20Tenants%20—%20Handoff%20Sheet.md#25-design-file-what-needs-fixing) | [§20](DA-08%20—%20Occupancy/DA-08%20Inventory%20—%20Handoff%20Sheet.md#20-design-file-what-needs-fixing) | [§17](DA-04%20—%20Expenses/DA-04%20Expense%20—%20Handoff%20Sheet.md#17-design-file-what-needs-fixing) | [§21](DA-01%20—%20Dues/DA-01%20Dues%20—%20Handoff%20Sheet.md#21-design-file-what-needs-fixing) | [§19](DA-02%20—%20Collections/DA-02%20Collection%20—%20Handoff%20Sheet.md#19-design-file-what-needs-fixing) |

Every sheet also opens with its own **What is in here** table: the full section list with a reading map per audience.

## The rules every sheet follows

| Rule | Meaning |
|---|---|
| Self-contained | One sheet is the whole spec for its screen. No other document needed to build |
| Plain language | The team's own words. No code, no file paths, no internal numeric codes |
| Outcome plus test | Anything that must hold carries a *Test it:* line QA can run |
| One grammar | Same spine, same section names, same table shapes across all sheets, from [the template](_meta/_Handoff%20Sheet%20Template.md) |
| Two kinds of number | **Live** always shows now; **Time-scoped** counts inside the filter window. Every sheet carries the grid |
| Honest gaps | A number that cannot open its own records says so; nothing silently opens a wrong list |

## How this repo stays current

| | |
|---|---|
| Source of truth | The Obsidian vault. This repo carries the closed deliverables only |
| What lands here | Each screen's handoff sheet and reading maps, at close-out, plus the tracker and template |
| What never lands here | Briefs, formula maps, archives, superseded PRDs. They stay in the vault |
| Wikilinks | `[[Like this]]` resolve inside Obsidian, not on GitHub, and are kept as written |
