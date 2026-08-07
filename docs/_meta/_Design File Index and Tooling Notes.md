---
title: Design File Index and Tooling Notes
date: 2026-08-07
tags:
  - rentok
  - detailed-analytics
  - figma
  - reference
owner: Sanchay
---
# Design file index and tooling notes

Working notes for the analytics suite. Saves rediscovering all of this on the next screen.

## The Figma file

**File key** `KgBQXiT7r7oGrcqZHCWxyU` — "Home-Screen — Manager App". **Two pages:**

| Page | Node | What it is |
|---|---|---|
| Insight Screens | `0:1` | All analytics work. 164 top-level items |
| Final Designs | `1:2` | The app-wide locked component library — **not** a rival version of any screen |

**The current build target for every analytics screen** is the section `Analytics Design Draft3- WIP` (`10764:116936`). Its children are named *"Dues insight - done"*, *"Collection insight - done"*, *"Expense insight - done"*, *"Occupancy insight - done"*, *"Tenant insight"*, *"Issues insight"*. **The "- done" suffix marks the designer's own sign-off** — Tenant insight lacked it when documented.

## Tenants (DA-09) — node map

| | Node |
|---|---|
| Section | `10767:28646` |
| Screen, 412×5443 | `10905:109826` |
| Tile row component | `10767:29583` |
| Designer notes, G1–G8 | `10908:110018` |
| Move-in / move-out note | `10914:110019` |
| Empty states, 14 cards | `11454:114344` |

**Component sets:** Journey `11016:112654` · lifecycle-expanded `11342:111402` (not the build target) · Tenancy Details `10997:28977` · Tenant Profile `11005:29354` · Tenant Details `11005:111277`

**Six prior versions**, all carrying the same eight cards under older names: `9747:109155` · `6428:96170` · `8233:99014` · `5398:77058` · `4940:71601` · `80:573`

**Also in the file and undocumented:** `TENANT'S ADVANCED INSIGHTS` (`6438:105459`, `9747:110061`) — a second-level screen concept holding Widgets · lifecycle · Tenant Demographics. **Issues has the same thing.** No screen in this project owns it.

## Reading the file without drowning

**Node trees blow the tool's response limit.** The Tenants section alone is 132KB; page `0:1` is 20MB. Two ways through:

1. **Call the MCP endpoint directly over HTTP** — `POST http://127.0.0.1:3845/mcp`, standard JSON-RPC: initialize, read the `mcp-session-id` response header, send `notifications/initialized`, then `tools/call`. Bypasses the transport limit entirely. Responses come back as SSE `data:` lines; screenshots arrive as base64 rather than a URL.
2. **Write the XML to a file and parse it**, never read it into the conversation.

**Text content lives in layer names**, so the copy can be extracted without rendering anything.

⚠ **Hidden-ness lives on an ancestor container, not on the text node.** Any check that reads text nodes alone reports switched-off elements as visible. **Walk the parents.** This produced three wrong claims on Tenants before it was caught — including "the chips are missing" when in fact five were present but switched off, which was the more interesting finding.

⚠ **Elements parked outside the phone frame are not part of the screen.** Compare each item's x against the frame's. On Tenants this correctly excluded a Deposit Dues card borrowed from the Dues screen, a ratio-style Property Wise variant, and loose copies of two cards.

## The developer's spreadsheet

`https://docs.google.com/spreadsheets/d/1XGFQxXyVhNaciM6VEMFpX7Ufj_3-VaaNOpJVOk2H9Po/edit`

**Fetch a tab by name, no gid needed:**
`.../gviz/tq?tqx=out:csv&sheet=<TabName>`

**Tabs that exist: Issues · Tenants · Dues · Collection · Expense.** That is all of them — confirmed by reading the tab bar out of the page HTML (`docs-sheet-tab-caption`). **Old Tenants and Bookings have no tab**, so those two screens have no informal source at all.

## Production querying

Metabase database **ID 2**. Column-name traps that cost time on Tenants:

- `"createdAt"` is camelCase and must be quoted; there is no `updated_at` on the tenant table
- The tenant table uses `property_id` and `room`, not `property`/`room_id`
- `stamp_agreements` joins on `tenant_id`; `tenant_eviction_details` and `tenant_booking_confirmation` join on `tenant_uuid`
- `agreement_period` is text in places and contains junk — sanitise before casting
- `lockin_period` is in **months** and contains dates pasted as numbers; clamp the range
- Several boolean-looking property flags are smallint — `::int = 1`, not `IS TRUE`

**Always check `information_schema.columns` for lookalike fields before concluding one is empty.** Tenant type has three candidates: the one holding the answers (23% filled), one that is 99% empty and holds different vocabulary, and one that is effectively unused. The first conclusion drawn was from the wrong field.
