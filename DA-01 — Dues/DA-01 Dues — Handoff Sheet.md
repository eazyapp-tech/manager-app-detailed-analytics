---
screen: DA-01 — Dues
source: Figma "Home Screen — Manager App", frame "Dues insight - done" (node
  10764:112750) + existing DA-01 Brief/Formula Map/Build Sheet + developer's own
  rough sheet + owner review pass + grilling session on Restricted-state and
  hidden-breakdown threads
status: v8 — two cross-screen corrections, see top of doc
date: 2026-08-06
---

# DA-01 — Dues: Handoff Sheet

Plain-language definition of every number on the Dues widget and its detail screen. No code, no file names — this is what the number means and where it comes from, not how it's built.

> [!WARNING] Two corrections since v7, made 7 August 2026
> Both came out of checking this sheet against the Collection and Expense sheets. Anyone who built from v7 should re-read these two points.
>
> 1. **The change indicator colours were the wrong way round.** They now read red ▲ / green ▼. Dues going up is bad.
> 2. **A card's own date dropdown no longer survives a change to the filter at the top.** It snaps back. This was decided after this sheet closed and now applies to every analytics screen.

## Empty-state copy fixes needed

Nearly every empty state on this screen currently shows leftover copy from the Maintenance/Complaints module. Only Defaulter has correct, on-topic copy today — use its tone as the model for the rest.

| Widget | Current copy (wrong) | Recommended copy | CTA |
|---|---|---|---|
| Dues (Live) | "You're all caught up! New maintenance requests and…" *(cut off, Maintenance-domain leftover)* | **Title:** No dues yet **Subtitle:** Add a tenant to start tracking dues for this property. | "Add first tenant" *(already in Figma — keep)* |
| Bills Summary | Same leftover text | **Title:** No bills raised yet **Subtitle:** Bills you raise for this property will show up here. | — |
| Dues Breakdown | Same leftover text | **Title:** Nothing to break down yet **Subtitle:** Once bills are raised, you'll see them by category, tenant status, and who added them. | — |
| Overdue Breakup / Defaulter | ✅ Already correct: "No Defaulters this month" / "Every tenant has paid on time." | No change — this is the reference tone for every row above and below | — |
| Upcoming Dues | Same leftover text | **Title:** No upcoming dues **Subtitle:** Recurring bills due to be raised will show up here. | — |
| Deposit Dues | Same leftover text | **Title:** No deposit dues yet **Subtitle:** Deposits collected or pending will show up here. | — |
| Dues by Property | Same leftover text | **Title:** No dues to rank yet **Subtitle:** Once properties have outstanding dues, they'll be ranked here. | — |

## Build status

The backend for this screen exists as a scaffold today — every block (Overview, Dues Live, Bills Summary, Dues Breakdown, Overdue Breakup, Upcoming Rent, Deposit Dues, Stay Duration, Property-wise) is wired up and reachable, but every one currently returns placeholder numbers, not real data. Nothing below describes a live bug — it describes what the real logic should do once it's written.

## Where this widget lives

Manager App → property → **Financials** tab → **Dues** sub-tab (siblings: Collection, Expense).

## How the time filters work (applies the same way across every analytics section, not just Dues)

Two kinds of sections:
- **Live** sections — always show the current, real-time snapshot. No filter, global or local, ever changes them.
- **Duration-scoped** sections — show numbers for a time window, and follow the rules below.

One **global duration filter** sits at the top of the screen, with five options: **All Time, This Month, Last Month, Current FY, Custom**. ("All Time" is the same thing informal notes called "Today"/"Live" — it means no date limit, not literally today.)

- **Default: This Month.** Landing on "every due ever raised" (All Time) is a heavy, non-actionable first impression, especially for older properties — operators think in monthly cycles, so This Month is the useful landing state. (Recommendation, not yet user-tested — worth a quick sanity check once built.)
- Changing the global filter updates **every duration-scoped section** to that window.
- Some sections also have their **own local duration dropdown** (e.g. "This Month" shown on a section header), offering the same five options as the filter at the top. Changing a section's dropdown affects **only that section**.
- **Changing the filter at the top pulls every section back into line.** Any section sitting on its own period snaps back to follow it. A manager can set one section aside to check something, but can never end up with a screen showing four different periods without meaning to.

## Drill-down rule (applies across every breakdown section below)

Every duration-scoped breakdown widget — Dues Breakdown, Overdue Breakup, Dues by Property — opens the exact filtered bill list behind the number tapped (tap "Overdue" → see exactly those bills, tap a property → see exactly that property's dues, etc.). This was a hard rule in earlier hardening and carries forward here.

**Exception:** forecast-only numbers (This Month's Projected Due, Upcoming Dues) can't open a real bill list — the bills don't exist yet — so those open an explainer sheet instead, same as the existing "Dues Overview" sheet already does.

## Live sections (always real-time, ignore both filters)

### 1. Overview strip — 6 summary tiles

| Tile | Meaning |
|---|---|
| All Time Dues | Every unpaid bill right now, regardless of due date |
| All Past Dues | Unpaid bills whose due date has already passed |
| This Month's Due | Unpaid bills due between the 1st of this month and today |
| All Future Dues | Unpaid bills due from tomorrow onward |
| This Month's Projected Due | Forecast of recurring bills expected to be created before month-end, across every due type this tenant/property has set to auto-recur — not rent-only (see §7) |
| Current FY Dues | Unpaid bills due within the current financial year (April–March) |

Each tile shows a change indicator against the previous period. **Up is bad on this screen: an increase is red, a decrease is green.** Dues are money owed to you that has not arrived, so more of them is worse. This is the same polarity as Expense and the opposite of Collection, and it has to be set deliberately because the shared component defaults to the wrong way round.

**Where the selected period is still running, compare against the same point in the previous period**, not against the whole of it, and say so on the chip: *"▼ 12% vs same point last month."* On the 3rd of the month a whole-month comparison shows a collapse that is not real. The note drops away once the period completes.

**Base rule (applies everywhere on this page unless a section says otherwise):** a bill counts if it's unpaid and for at least ₹1, and the tenant it belongs to is active, under notice, or a confirmed booking. **Moved-out (old) tenants are deliberately excluded from every number on this page except one** — the "Old Tenants" row inside Dues Breakdown → Tenant Status (§5). That row is the only place their unpaid balance should ever surface; it should not be blended into All Time Dues, the Live gauge, FY Dues, or any other total. Build note: an older, currently-live widget elsewhere in the app excludes old tenants everywhere with no per-row exception — don't copy that pattern here, since it has no way to include them in just the one row this design calls for.

### 2. Dues (Live)

The breakdown directly under the Overview strip — same "Total Dues" headline as "All Time Dues" above, split into 4 buckets:

| Field | Meaning |
|---|---|
| Total Dues | Same figure as "All Time Dues" |
| Overdue | Portion whose due date has already passed |
| Due Today | Portion due today |
| Due This Week | Portion due in the current 7-day window (widget shows the actual date range, e.g. "01 Aug – 07 Aug") |
| Due Later | Portion due after this week (widget shows the actual start date, e.g. "After 08 Aug") |

The Figma file also has a tenant-status sub-breakdown (Active / Booking / Old Tenants, each with an amount) built into this same widget but switched off. **Dead, not deferred — do not build.** It duplicates the Tenant Status view already in §5; that's the one place this grouping should live.

### 3. Deposit Dues

| Field | Meaning |
|---|---|
| Received | Security/caution deposit already collected and still held |
| Due | Security/caution deposit still unpaid |

Live/running-balance by nature — deposits are evaluated as a running total, not a period.

## Duration-scoped sections (follow the global filter, or their own local override — see above)

### 4. Bills Summary (default This Month)

Redefined as a period snapshot, not a repeat of the Overview strip:

| Field | Meaning |
|---|---|
| Bill Due | Bills that came due within the selected window |
| Received | Money collected specifically against those same bills, within the same window |

Reads as a quick "how am I doing on collections this period" check — deliberately different from the Overview strip's all-time numbers.

### 5. Dues Breakdown (has its own local dropdown, default This Month)

Three views, switched by a toggle — same underlying bills, grouped differently:

| View | Groups by |
|---|---|
| Category | Top categories present in this window (typically Rent / Electricity / Deposit / Mess), rest rolled into Others |
| Tenant Status | Active / Under Notice / Bookings / **Old Tenants** — this is the one and only place moved-out tenants' dues appear anywhere on this screen (see Base Rule, §1) |
| Added By | Whoever raised the bill — Owner / Rent Manager / RentOk (auto-generated) / Tenant / Partner, top few shown, rest folded into an overflow list |

Tapping "Added By → Others" opens a separate list of every other person who added bills, each with their total and bill count (see §10).

### 6. Overdue Breakup / Defaulter (has its own local dropdown, default This Month)

Two views, same overdue bills grouped differently:

| View | Groups by |
|---|---|
| By Amount | How stale — 1–7 days / 8–14 days / 15–21 days / 22+ days overdue |
| By Category | Top categories among specifically the overdue bills, rest rolled into Others |

This is a **different slice** of bills than §5's Category view (all unpaid bills vs. only overdue ones) — so the top categories that surface can legitimately differ between the two (e.g. Joining Fee showing up here even though it isn't a top category overall). Not an inconsistency to fix — expected behavior of "top-of-this-slice."

A bill due today is not yet counted as overdue.

### 7. Upcoming Dues — recurring bills not yet raised (has its own local dropdown, default This Month)

Forecast of bills expected to be auto-raised before the window ends, grouped by type (Rent / Food / Others) and shown by due-date range.

**Logic:** driven by each tenant's recurring-dues-package setup — whatever due types that tenant has configured to auto-recur is what gets forecast, summed. Only rent configured → only rent shows. Rent + food configured → both show, cumulatively. "Others" is a catch-all for any further recurring type beyond Rent and Food, same top-categories-plus-rollup pattern used in §5/§6 — not a fixed third category.

Naming: use "Upcoming Dues" consistently (the live widget currently also carries the label "Upcoming Rent (to be added)" — pick one, recommend "Upcoming Dues" since it doesn't imply rent-only).

### 8. Breakup by Stay Duration

Splits outstanding dues into **Short Stay** vs **Long Stay** tenants. Stay-type is a required field on every tenant record, so in practice every tenant lands in one of the two buckets — no "unbucketed" case expected.

### 9. Dues by Property (has its own local dropdown, default This Month)

Outstanding dues per property, ranked highest to lowest. A "% share of total" next to each property was in the earlier spec but isn't visible in the current Figma — up to the developers to add if time allows; design will accommodate it.

## Detail sheets

### 10. "Added By → Others" detail sheet

Full list of everyone (besides the top few shown in §5) who added bills in the selected window, each with their total amount and bill count.

### 11. "Dues Overview" detail sheet (view-all)

Opens from the widget header. A read-only explainer, not a filtered bill list — restates every number from §1 (All Time / Past / This Month / Future / Projected / FY Dues) plus category and stay-duration breakdowns, all for the same fixed windows.

## Restricted (no-permission) state

Shows a locked overlay — "Analytics Restricted — You don't have permission to view these analytics. Request access from your admin." with a "Request Access" button. Clean copy, no issues.

**Scope of the lock:** matches the existing app-wide permission grouping — **Dues and Collection are locked or unlocked together** (same underlying permission today), **Expense is gated independently**. A user without Dues access sees this same lock on Collection too; their Expense access is unaffected either way.

**A note on narrowed views.** The system holds a permission that limits a team member to seeing only the tenants they added themselves. Product's position is that nobody is actually given it, so it isn't a state to design for. It is built and working in the code, though — see Open items.

## Build guidance (read before implementing)

1. **Old-tenant scope is a one-row exception, not a page-wide default.** Include moved-out tenants' dues only in Dues Breakdown → Tenant Status → Old Tenants (§5). Exclude them everywhere else. Don't reuse the older widget's shared query for this screen — that query has no way to make just one row an exception.
2. **Restricted-state permission check should read the same permission Dues and Collection already share**, plus the separate one Expense already has — don't invent a third, and don't build it as an all-or-nothing "whole Financials tab" lock.
3. **Don't build a narrowed self-added-tenant view.** Product says nobody is given that permission, so it isn't a state this screen needs to handle.
4. **Empty-state copy needs fixing** — see table at the top of this doc.
5. **Do not build the hidden tenant-status sub-breakdown inside Dues (Live)** — it's dead, superseded by §5.

## Open items

**One question for engineering.** The system has a permission that narrows a team member's view to only the tenants they added. It is built and working across both Dues and Collection. Product says nobody is actually given it. Confirm that — because if anyone does hold it, their numbers are being quietly narrowed today with nothing on screen saying so, and two people looking at the same property would see different totals with no way to explain the gap.

Everything else from this round is closed.
