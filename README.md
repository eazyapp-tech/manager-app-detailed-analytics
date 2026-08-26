# Manager App Detailed Analytics

Handoff documentation for the RentOk Manager App analytics suite. Nine screens are in scope and **six have a sheet today**; the other three are listed below with nothing behind them yet. A sheet is self-contained: Each sheet defines every number, every window, every tap and every empty state in plain language. No code references anywhere; a developer builds from it, a designer works from it, a non-technical reader follows it.

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
| **Running the project** | [The work queue in the verification log](01%20—%20Build%20Verification%20Log.md#the-work-queue) | Everything owed, one readable row per job. Live state lives there and nowhere else |
| **Writing or reviewing a sheet** | [The rules, R1 to R100](00%20—%20Manager%20App%20Analytics%20Tracker.md#the-rules) | Every rule the suite is written to, each one citable by number. That file is method and history now, and holds no live state |
| **Writing a new sheet** | [The template](_meta/_Handoff%20Sheet%20Template.md) | The canonical spine. Copy it; never start from a blank page |
| **Checking what was built** | [The verification log](01%20—%20Build%20Verification%20Log.md) | Every difference between what we asked for and what shipped, with what the owner ruled. Its [work queue](01%20—%20Build%20Verification%20Log.md#the-work-queue) is what is still owed, one readable row per job, re-checked against the live code on 27 Aug 2026 |
| **Fixing something that is not a spec failure** | [The suggestions register](02%20—%20Suggestions%20Register.md) | Product problems found while verifying, none of them actioned without a ruling |
| **Wanting engineering's own explanation** | [The calculation guide](<03 — Analytics Calculation Guide (Vivek).md>) | Vivek's block-by-block account of how each number is worked out. Read-only copy; the source lives in the backend repo |

## The screens

| # | Screen | Handoff sheet | Engineering map | Design map | Status |
|---|---|---|---|---|---|
| 1 | Dues | [DA-01 sheet](DA-01%20—%20Dues/DA-01%20Dues%20—%20Handoff%20Sheet.md) | [read me first](DA-01%20—%20Dues/DA-01%20Cover%20Note%20—%20Nimit.md) | [design map](DA-01%20—%20Dues/DA-01%20Cover%20Note%20—%20Design.md) | ✅ v12 · **the current reference** |
| 2 | Collection | [DA-02 sheet](DA-02%20—%20Collections/DA-02%20Collection%20—%20Handoff%20Sheet.md) | [read me first](DA-02%20—%20Collections/DA-02%20Cover%20Note%20—%20Nimit.md) | [design map](DA-02%20—%20Collections/DA-02%20Cover%20Note%20—%20Design.md) | ✅ v13 · the toggle screen |
| 3 | Expense | [DA-04 sheet](DA-04%20—%20Expenses/DA-04%20Expense%20—%20Handoff%20Sheet.md) | [read me first](DA-04%20—%20Expenses/DA-04%20Cover%20Note%20—%20Nimit.md) | [design map](DA-04%20—%20Expenses/DA-04%20Cover%20Note%20—%20Design.md) | ✅ v3.1 |
| 4 | Inventory | [DA-08 sheet](DA-08%20—%20Occupancy/DA-08%20Inventory%20—%20Handoff%20Sheet.md) | [read me first](DA-08%20—%20Occupancy/DA-08%20Cover%20Note%20—%20Nimit.md) | [design map](DA-08%20—%20Occupancy/DA-08%20Cover%20Note%20—%20Ishika.md) | ✅ v2.1 · uplifted to template, verified against the build |
| 5 | Tenants | [DA-09 sheet](DA-09%20—%20Tenants/DA-09%20Tenants%20—%20Handoff%20Sheet.md) | [read me first](DA-09%20—%20Tenants/DA-09%20Cover%20Note%20—%20Nimit.md) | [design map](DA-09%20—%20Tenants/DA-09%20Cover%20Note%20—%20Design.md) | ✅ v2.2 · voice exemplar |
| 6 | Old Tenants | — | — | — | 🟡 next up |
| 7 | Bookings | — | — | — | ⬜ |
| 8 | Complaints | [DA-10 sheet](DA-10%20—%20Complaints/DA-10%20Complaints%20—%20Handoff%20Sheet.md) | [read me first](DA-10%20—%20Complaints/DA-10%20Cover%20Note%20—%20Nimit.md) | [design map](DA-10%20—%20Complaints/DA-10%20Cover%20Note%20—%20Design.md) | ✅ v1.1 · audited |
| 9 | Leads | — | — | — | ⬜ newest addition |

## Jump straight into a sheet

Deep links into the sections readers ask for most.

| Section | Complaints (DA-10) | Tenants (DA-09) | Inventory (DA-08) | Expense (DA-04) | Dues (DA-01) | Collection (DA-02) |
|---|---|---|---|---|---|---|
| What every number counts | [§3](DA-10%20—%20Complaints/DA-10%20Complaints%20—%20Handoff%20Sheet.md#3-what-every-number-counts) | [§3](DA-09%20—%20Tenants/DA-09%20Tenants%20—%20Handoff%20Sheet.md#3-what-every-number-counts) | [§3](DA-08%20—%20Occupancy/DA-08%20Inventory%20—%20Handoff%20Sheet.md#3-what-every-number-counts) | [§3](DA-04%20—%20Expenses/DA-04%20Expense%20—%20Handoff%20Sheet.md#3-what-every-number-counts) | [§3](DA-01%20—%20Dues/DA-01%20Dues%20—%20Handoff%20Sheet.md#3-what-every-number-counts) | [§3](DA-02%20—%20Collections/DA-02%20Collection%20—%20Handoff%20Sheet.md#3-what-every-number-counts) |
| How the screen behaves | [§4](DA-10%20—%20Complaints/DA-10%20Complaints%20—%20Handoff%20Sheet.md#4-how-the-screen-behaves) | [§4](DA-09%20—%20Tenants/DA-09%20Tenants%20—%20Handoff%20Sheet.md#4-how-the-screen-behaves) | [§4](DA-08%20—%20Occupancy/DA-08%20Inventory%20—%20Handoff%20Sheet.md#4-how-the-screen-behaves) | [§4](DA-04%20—%20Expenses/DA-04%20Expense%20—%20Handoff%20Sheet.md#4-how-the-screen-behaves) | [§4](DA-01%20—%20Dues/DA-01%20Dues%20—%20Handoff%20Sheet.md#4-how-the-screen-behaves) | [§4](DA-02%20—%20Collections/DA-02%20Collection%20—%20Handoff%20Sheet.md#4-how-the-screen-behaves) |
| What each number opens | [§17](DA-10%20—%20Complaints/DA-10%20Complaints%20—%20Handoff%20Sheet.md#17-what-each-number-opens) | [§19](DA-09%20—%20Tenants/DA-09%20Tenants%20—%20Handoff%20Sheet.md#19-what-each-number-opens) | [§14](DA-08%20—%20Occupancy/DA-08%20Inventory%20—%20Handoff%20Sheet.md#14-what-each-number-opens) | [§11](DA-04%20—%20Expenses/DA-04%20Expense%20—%20Handoff%20Sheet.md#11-what-each-number-opens) | [§15](DA-01%20—%20Dues/DA-01%20Dues%20—%20Handoff%20Sheet.md#15-what-each-number-opens) | [§13](DA-02%20—%20Collections/DA-02%20Collection%20—%20Handoff%20Sheet.md#13-what-each-number-opens) |
| Build guidance | [§21](DA-10%20—%20Complaints/DA-10%20Complaints%20—%20Handoff%20Sheet.md#21-build-guidance) | [§23](DA-09%20—%20Tenants/DA-09%20Tenants%20—%20Handoff%20Sheet.md#23-build-guidance) | [§18](DA-08%20—%20Occupancy/DA-08%20Inventory%20—%20Handoff%20Sheet.md#18-build-guidance) | [§15](DA-04%20—%20Expenses/DA-04%20Expense%20—%20Handoff%20Sheet.md#15-build-guidance) | [§19](DA-01%20—%20Dues/DA-01%20Dues%20—%20Handoff%20Sheet.md#19-build-guidance) | [§17](DA-02%20—%20Collections/DA-02%20Collection%20—%20Handoff%20Sheet.md#17-build-guidance) |
| Open items | [§22](DA-10%20—%20Complaints/DA-10%20Complaints%20—%20Handoff%20Sheet.md#22-open-items) | [§24](DA-09%20—%20Tenants/DA-09%20Tenants%20—%20Handoff%20Sheet.md#24-open-items) | [§19](DA-08%20—%20Occupancy/DA-08%20Inventory%20—%20Handoff%20Sheet.md#19-open-items) | [§16](DA-04%20—%20Expenses/DA-04%20Expense%20—%20Handoff%20Sheet.md#16-open-items) | [§20](DA-01%20—%20Dues/DA-01%20Dues%20—%20Handoff%20Sheet.md#20-open-items) | [§18](DA-02%20—%20Collections/DA-02%20Collection%20—%20Handoff%20Sheet.md#18-open-items) |
| Design file fixes | [§23](DA-10%20—%20Complaints/DA-10%20Complaints%20—%20Handoff%20Sheet.md#23-design-file-what-needs-fixing) | [§25](DA-09%20—%20Tenants/DA-09%20Tenants%20—%20Handoff%20Sheet.md#25-design-file-what-needs-fixing) | [§20](DA-08%20—%20Occupancy/DA-08%20Inventory%20—%20Handoff%20Sheet.md#20-design-file-what-needs-fixing) | [§17](DA-04%20—%20Expenses/DA-04%20Expense%20—%20Handoff%20Sheet.md#17-design-file-what-needs-fixing) | [§21](DA-01%20—%20Dues/DA-01%20Dues%20—%20Handoff%20Sheet.md#21-design-file-what-needs-fixing) | [§19](DA-02%20—%20Collections/DA-02%20Collection%20—%20Handoff%20Sheet.md#19-design-file-what-needs-fixing) |

Every sheet also opens with its own **What is in here** table: the full section list with a reading map per audience.

**The version above is copied from each sheet's own frontmatter, which is the answer if the two ever disagree.** Checked against all six on **27 August 2026**. It used to be copied into two places and was wrong in both.

Screens 6, 7 and 9 have no sheet yet. Only three of the nine are built and reachable in the app today, and the app labels two of them differently from their sheets: it draws `Financials · Occupancy · Tenant`, where the sheets say Inventory and Tenants. That is [F109 in the verification log](01%20—%20Build%20Verification%20Log.md#findings), and it needs a ruling on which side changes.

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
| Source of truth | **This repo.** The Obsidian vault at `RentOk/PRDs/Homescreen Detailed Analytics/` is a mirror of it. This reversed on 27 August 2026, after the repo had run ahead of the vault for three days |
| What lands here | Each screen's handoff sheet and reading maps at close-out, the template, the rules and history in `00`, the verification log in `01` and the suggestions register in `02`. The last two are live working documents, not closed deliverables |
| What never lands here | Briefs, formula maps, archives, superseded PRDs. They stay in the vault |
| Where state lives | One place each: a sheet's version in its own frontmatter · what is owed on the code in `01`'s work queue · questions for the owner in `01` · product problems in `02` · which screen is next in the table above. Ruled 27 August 2026, after the same fact was found stored in three places and wrong in two |
| Wikilinks | `[[Like this]]` resolve inside Obsidian, not on GitHub, and are kept as written |
