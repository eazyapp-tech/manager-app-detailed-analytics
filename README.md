# Manager App Detailed Analytics

Product documentation for the RentOk Manager App analytics suite: nine screens, one handoff sheet per screen, written so a developer can build from them and a non-technical reader can follow them. Plain language, no code references, every number defined.

## Where things are

| Folder | Screen | Status |
|---|---|---|
| [docs/DA-01 — Dues](docs/DA-01%20—%20Dues) | Dues | Closed v8, restructure queued |
| [docs/DA-02 — Collections](docs/DA-02%20—%20Collections) | Collection | Closed v12, restructure queued |
| [docs/DA-03 — Refunds](docs/DA-03%20—%20Refunds) | Refunds | Deferred, no screen in current design |
| [docs/DA-04 — Expenses](docs/DA-04%20—%20Expenses) | Expense | Closed v2 |
| [docs/DA-05 — Discounts](docs/DA-05%20—%20Discounts) | Discounts | Earlier round, not in current eight |
| [docs/DA-06 — Liabilities](docs/DA-06%20—%20Liabilities) | Liabilities | Earlier round, not in current eight |
| [docs/DA-07 — Cash Flow](docs/DA-07%20—%20Cash%20Flow) | Cash Flow | Earlier round, not in current eight |
| [docs/DA-08 — Occupancy](docs/DA-08%20—%20Occupancy) | Inventory | Closed v1 |
| [docs/DA-09 — Tenants](docs/DA-09%20—%20Tenants) | Tenants | Closed v2, the current standard |
| docs/Tenant Insights | Superseded PRD, kept with its banner | Superseded in part |
| docs/_meta | Template, tooling notes, cross-screen specs | — |

**Start here:** [the tracker](docs/00%20—%20Manager%20App%20Analytics%20Tracker.md). It holds every locked cross-screen rule, the backfill queue, and the next action. Read it before touching anything.

**Writing a new sheet?** Copy [the template](docs/_meta/_Handoff%20Sheet%20Template.md). Never start from a blank page.

## How each screen's folder reads

The current-standard sheets (Tenants, Inventory, Expense) each carry: the handoff sheet (the whole spec), cover notes for engineering and design (reading maps), and an `archive/` of superseded sources, each marked with what overrides it. Older folders still carry the earlier three-document structure (Brief, Ground-Truth Formula Map, Build Sheet) and get restructured to the template at their next touch.

## Rules of this repo

- **The Obsidian vault is the source of truth** (`RentOk/PRDs/Homescreen Detailed Analytics/`). This repo is its shareable mirror; sync is a plain copy because filenames match exactly.
- `[[Wikilinks]]` in the docs resolve inside Obsidian, not on GitHub. They are kept as-is.
- Documents follow the suite grammar: one spine, fixed section names, fixed table schemas, plain punctuation, no code references. The grammar lives in the tracker and the template.
- Superseded documents are never deleted; they carry a banner naming what overrides them.
