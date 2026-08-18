---
title: DA-10 Complaints — Handoff Sheet
date: 2026-08-18
tags:
  - rentok
  - complaints
  - detailed-analytics
  - handoff
status: v1 · developer handoff
owner: Sanchay
---
# Complaints — Handoff Sheet

Everything on the Complaints analytics screen: what each number means, what window it covers, what happens when it is tapped, and what it shows when there is nothing to show.

---

## What is in here

|        | Section                              | For              |
| ------ | ------------------------------------ | ---------------- |
| **1**  | [Build status](#1-build-status) | everyone |
| **2**  | [Where this lives](#2-where-this-lives) | everyone |
| **3**  | [What every number counts](#3-what-every-number-counts) | backend + copy |
| **4**  | [How the screen behaves](#4-how-the-screen-behaves) | backend + design |
| **5**  | [Overview Snapshot](#5-overview-snapshot) | backend + design |
| **6**  | [View all sheet](#6-view-all-sheet) | backend + design |
| **7**  | [Current Status](#7-current-status) | backend + design |
| **8**  | [Ageing Distribution](#8-ageing-distribution) | backend + design |
| **9**  | [Response Time](#9-response-time) | backend + design |
| **10** | [Cost of Issues](#10-cost-of-issues) | backend + design |
| **11** | [Efficiency & Quality](#11-efficiency--quality) | backend + design |
| **12** | [Team Performance](#12-team-performance) | backend + design |
| **13** | [Issues Sources](#13-issues-sources) | backend + design |
| **14** | [Issues Trend](#14-issues-trend) | backend + design |
| **15** | [Ratings & Feedback](#15-ratings--feedback) | backend + design |
| **16** | [Property Performance](#16-property-performance) | backend + design |
| **17** | [What each number opens](#17-what-each-number-opens) | backend |
| **18** | [Who can see this](#18-who-can-see-this) | backend |
| **19** | [What each card shows when it is empty, healthy or broken](#19-what-each-card-shows-when-it-is-empty-healthy-or-broken) | design + copy |
| **20** | [What this screen is not](#20-what-this-screen-is-not) | everyone |
| **21** | [Build guidance](#21-build-guidance) | backend |
| **22** | [Open items](#22-open-items) | everyone |
| **23** | [Design file: what needs fixing](#23-design-file-what-needs-fixing) | design |
|        | [Where the measured figures came from](#where-the-measured-figures-came-from) | everyone |

**Building the backend?** Read everything except sections 19 and 23. Start with Build guidance. Section 3 is the wording for every info icon, one row per number.

**Working on the design?** Sections 4 to 16, 19, and 23, which collects every fix in one list. Nothing for this screen's empty, healthy or failed states is drawn yet; section 19 is the brief for all of them.

**Just need the decisions?** Sections 1, 20 and 22, plus the measured figures at the end.

---

## 1. Build status

**Not built.** The screen's data block is wired and reachable and returns nothing for every request, whatever the property or filter. Nothing here is broken; it is unbuilt.

**The complaints list cannot yet open most of these numbers.** A manager tapping a number expects to land on exactly the complaints behind it. Today the list can filter by when a complaint was raised, its status, its category, its priority, and who it is assigned to. It cannot filter by when a complaint was closed, how long it has been waiting, whether it is past its target, whether it has money attached, who raised it by type, its rating, or its room. Section 17 lists every gap. **These filters ship with the screen, not after it**: a number that cannot open its own records is a report, and managers already have reports.

**Three things this screen depends on that do not exist yet:**

- **A way to mark a complaint Closed.** The status exists in the system and is read by the complaints widget, the homescreen and the list. Nothing lets a tenant or a manager set it. Until that ships, the Closed slice reads zero on every property.
- **A record of when a complaint escalates.** The system sends the escalation notifications and keeps no record that it did. Escalation Rate cannot be built until each escalation is written down.
- **The homescreen's Issues card counts differently from this screen.** It counts complaints raised in the period; this screen counts complaints active in the period. Both are meant to move to the same model, already ticketed as a priority fix on the homescreen. Build it once and both agree.

**The design is not signed off.** Every other analytics section in the design file carries a "done" marker. This one does not, and it has no empty, healthy or failed states drawn.

---

## 2. Where this lives

Manager app, home, **Complaints** tab. The screen opens from the Issues card at the top of that tab.

The suite tab row: Financial (Dues · Collection · Expense) · People (Leads · Bookings · Tenants · Old Tenants) · Inventory · **Complaints**. The design file's tab row still reads "Issues"; the settled name is Complaints.

One property at a time, or every property the manager can see. Property-wise cards appear only when more than one property is in view.

---

## 3. What every number counts

**A complaint** is one problem reported at a property: by a tenant, by a team member, or through the complaint bot. It carries a category, the room it belongs to where recorded, a status, and whether it is urgent.

| Term | Meaning |
|---|---|
| **Raised** | The complaint was created |
| **Resolved** | The problem was fixed. The complaint's status moved to Resolved |
| **Closed** | The complaint ended without a fix: raised by mistake, not a real complaint, or a duplicate. Either the tenant or a manager can close one. Different from Resolved, see below |
| **Open** | Not yet Resolved and not yet Closed |
| **New** | Raised, nobody has picked it up yet |
| **In Progress** | Being worked on. Every status that is not New, Resolved, Closed or Reopened |
| **Reopened** | Was Resolved, then came back |
| **Urgent** | Marked urgent when raised or afterwards |
| **Assigned** | A named team member is in charge of it |
| **Category** | What the complaint is about, from the property's list: Plumbing, Electrical Appliances, Cleaning and so on |
| **Target time** | How long a complaint of that category should take to fix. Set per property and per category, with a default where none is set |
| **Overdue** | Open and past its target time |
| **Live** | A number that ignores the date filter and always shows now. Says "as of today" on its face |
| **Time-scoped** | A number that counts inside the window the filter picks |

### Resolved and Closed

Resolved means somebody did the work. Closed means nobody needed to: the tenant raised it by mistake, it was never a real complaint, or it duplicated one already open. A tenant can close their own; a manager can close any. The two are never merged, and Issues Resolved never counts a Closed complaint. Nothing lets anyone mark a complaint Closed yet; see section 1.

### Repeat Complaints

A complaint counts as a repeat when the same room already had a complaint in the same category, that earlier one was resolved, and the new one arrived within 30 days of it resolving. It counts from the moment of resolving, so a complaint that comes back the same day counts. Category means the top level, Plumbing rather than the particular tap. Complaints with no room recorded cannot be repeats and are left out. Repeat and Reopened measure different things: Reopened is the same complaint coming back, Repeat is a new complaint for the same problem in the same place.

### Open Backlog

Complaints that were open at any point during the period, and are still open today. A past month's number shrinks over time as those complaints get resolved. On All Time it is simply everything open now.

### Target time and Overdue

Every complaint has a target time, from its category. Where the property has set one, that is used, sub-category first, then category. Where none is set, the default: **one day** for daily services (food, cleaning, housekeeping, power, internet), **seven days** for everything else. A complaint is overdue once open past its target. The escalation time that comes after the target is a separate clock and does not define overdue.

### Ratings

A tenant may rate a complaint after it is resolved, one to five stars, and leave a comment. Only ratings of one to five count. **Test it:** the average can never display a figure above five.

### Money on this screen

Total Impacted is money **spent** fixing complaints: the expenses linked to them. Money billed back to a tenant for damage, and money later collected against it, are separate questions and are not in this number.

### What the info icons say

Every card carries an info icon that opens a sheet, one row per number, in the house voice: what is counted, one sentence, no coaching, no "you". A named measure's row shows its live value beside the definition; a slicing row (categories, people, sources) explains the cut and carries no value. Where a card has tabs, the sheet has the same tabs.

**Overview Snapshot**, two tabs. *Overview:* Issues Created, New complaints raised in the selected period. · Issues Resolved, Complaints resolved in the selected period, whenever raised. · Open Backlog, Complaints active in the selected period, still open today. · Reopened, Complaints that were resolved and came back. · Repeat Complaints, Same room, same category, within 30 days of the last one resolving. · Average Resolution Time, Average time from raised to resolved. Counts only resolved complaints. *View all:* Net flow, Complaints raised minus complaints resolved. Positive means the pile grew. · Median resolution time, The middle complaint's time to resolve. · Oldest open complaint, The longest anything has been waiting. · Reopened rate, Reopened as a share of complaints resolved. · Repeat rate, Repeats as a share of complaints raised. · Tenant complaint rate, Share of tenants who raised at least one complaint. · Worst rooms, Rooms with the most complaints. Rooms with none do not appear.

**Current Status.** New, Raised, nobody has picked it up yet. · In Progress, Being worked on. Covers every status that is not New, Resolved, Closed or Reopened. · Resolved, The problem was fixed. · Reopened, Resolved, then came back. · Closed, Ended without a fix: raised by mistake, not a real complaint, or a duplicate. · Urgent, Open complaints marked urgent. · Non-Urgent, Open complaints not marked urgent. · Overdue, Open complaints past their target time.

**Ageing Distribution.** Open complaints grouped by how long they have been waiting. Bars past 7 days show red. Always live.

**Response Time.** Time to Assign, Time before someone is put in charge of a complaint. · First Response, Time before anything first happens on it: picked up, a note added, or the status moved.

**Cost of Issues.** Total Impacted, Money spent on complaints in the selected period. · Open Issues, Of those complaints, how many are still open. · Avg Cost, Average spend per complaint that had money on it. · Resol. Time, Average time to resolve, for complaints with money on them.

**Efficiency & Quality.** Avg Resolution Time, Average time from raised to resolved. · Escalation Rate, How often complaints move up to the next person. · SLA Rate, How often complaints are finished within their target time.

**Team Performance**, two tabs. *Top Performer:* Team members ranked by how often they finish within the target. *Team:* Complaints given to each person against complaints they resolved, over the same period. Work with nobody assigned appears separately.

**Issues Sources**, two tabs. *Reporter:* Complaints grouped by where they came from. *Category:* Complaints grouped by category. Top 4 appear individually; the rest fall under Others.

**Issues Trend.** Complaints raised against complaints resolved, month by month.

**Ratings & Feedback.** Average Rating, Average tenant rating on complaints resolved in the selected period. · By stars, Ratings grouped by score. · Pending Rating, Resolved complaints the tenant has not rated.

**Property Performance.** Complaints by property, with the most open first.

Three rows carry a collapsed "How is this calculated?" underneath, rationed the way the homescreen rations them, for behaviour that surprises rather than merely interests. **Open Backlog:** Counts complaints that were open at any point in the period, then checks which are still open today. A past period shows fewer as complaints get resolved. **Closed:** Resolved means the problem was fixed. Closed means it ended without a fix: raised by mistake, not a real complaint, or a duplicate. **SLA Rate:** Each complaint has its own target based on its category. A complaint past its target counts as missed, whether or not it has been resolved.

The homescreen's Issues card says "Unresolved" and "Active" where this screen says Open; its three hints and one overview hint change to Open so the two screens agree.

### Words to be careful with

**Open** here means not yet Resolved or Closed. The homescreen's Issues card says "Unresolved" and "Active" for the same thing; both should become Open. **Resolved** and **Closed** are two different endings, above. **Assigned** means a named person is in charge; a complaint can be In Progress with nobody assigned. **Bill** does not appear on this screen: money here is an expense, spent fixing something. On Dues and Collection a bill is raised to a tenant; on Expense a bill is a receipt. Say expense, never bill.

---
## 4. How the screen behaves

### The time filter

Options: **Last 7 days · This Month (default) · Last Month · Current FY · All Time · Custom.** Custom stops at today: a complaint is raised when someone complains, so there is nothing ahead of today to show, and this screen has no forward setting. The filter stays where the manager put it while the app is open; a fresh launch opens on This Month. Coming back from a drill returns to the screen as it was left, with fresh numbers. A day runs midnight to midnight, India time.

Two kinds of number sit side by side on this screen.

| Kind | Meaning |
|---|---|
| **Live** | Always the current snapshot. No filter setting changes it. Says "as of today" on its face |
| **Time-scoped** | Counts what happened inside the window the filter picks |

Worked example, filter on Last Month (July), looked at on 18 August: Issues Created counts complaints raised in July. Issues Resolved counts complaints resolved in July, including ones raised in May. Open Backlog counts complaints open at any point in July that are still open on 18 August. The ageing chart ignores July entirely and shows everything open on 18 August, by how long it has been waiting.

**A property is a small place.** The typical property raises a handful of complaints a quarter. Every card must read sensibly with single digits in it, and every average and rate says how many complaints it is built on.

### What every number does on every filter setting

| Number | Kind | Last 7 days · This Month · Last Month · Current FY · Custom · All Time |
|---|---|---|
| Issues Created | Time-scoped | Counted inside the window |
| Issues Resolved | Time-scoped | Counted inside the window |
| Open Backlog | Time-scoped | Counted inside the window: open at any point in it, still open today |
| Reopened | Time-scoped | Counted inside the window |
| Repeat Complaints | Time-scoped | Counted inside the window |
| Average Resolution Time | Time-scoped | Counted inside the window |
| Everything on the View all sheet | Time-scoped | Counted inside the window |
| Current Status, all slices | Time-scoped | Counted inside the window |
| Urgent, Non-Urgent, Overdue | Time-scoped | Counted inside the window: open complaints from it |
| Ageing Distribution | Live | As of today |
| Response Time, both | Time-scoped | Counted inside the window |
| Cost of Issues, all four | Time-scoped | Counted inside the window |
| Efficiency & Quality, all three | Time-scoped | Counted inside the window: complaints raised in it |
| Team Performance, both tabs | Time-scoped | Counted inside the window |
| Issues Sources, both tabs | Time-scoped | Counted inside the window |
| Issues Trend | Time-scoped, window fixed | Each tile keeps its own fixed window; the filter does not change it |
| Ratings & Feedback | Time-scoped | Counted inside the window: complaints resolved in it |
| Property Performance | Time-scoped | Counted inside the window; its own dropdown can pick a different one |

The one card that ignores the filter says so beside its title: "As of today" on Ageing Distribution. Issues Trend carries its own 6 · 12 · 24 month control. Property Performance carries its own dropdown, offering the same options as the filter at the top; changing the top filter pulls it back into line.

### Periods that have not finished

On the default view most of the time the period is still running. Change chips compare against the same number of elapsed days in the previous period and say so: "vs same point last month". The note drops away once the period completes. On Custom, chips compare the same number of days immediately before the range.

### Change chips

The six Overview tiles, the four Cost of Issues numbers, and SLA Rate carry one. Nothing else does: the breakdowns, the ageing bars and the trend already show a shape, and an arrow on a shape is noise. Direction is set per number: **Issues Resolved and SLA Rate are green when rising; every other number is red when rising.** Nothing carries a chip on All Time, and the View all sheet carries none.

### When a number is red

Red where a promise to a tenant is broken. The overdue bar. The ageing bars past seven days, because no target time on this screen is longer than seven days, so anything still waiting past a week is late whatever its category. A change chip moving the wrong way. Nothing else: a slow average is a fact, not a missed promise, and a property just starting out must not open red.

### The action bar

Two, both on cards where the number names the work. On Current Status: the overdue count with **Send Reminder**, which nudges the tenants and assignees of every overdue complaint in view; never at zero. On Ratings & Feedback: the pending-rating count with **Send Reminder**, which asks the tenants of resolved, unrated complaints in view to rate; never at zero, and never the same person about the same complaint twice in a week. Both act on the complaints the filter is showing, so the manager sees the number before pressing.

### Loading, failure, sorting, entry

Each card loads with a skeleton shaped like the card it becomes and fails alone with "Couldn't load this" and Retry, never a healthy message, never a zero. Retry refetches only that card. When every card fails it is the connection: one message, one Retry for the screen, reading "Couldn't load this page. Check your connection." Error copy stays plain everywhere: no apology, no cause, no codes. Every property-wise and breakdown list sorts highest to lowest; the placeholder data in the design file does not demonstrate this, so take the rule from here, not the file. The entry point is the Issues card at the top of the Complaints tab.

---

## 5. Overview Snapshot

Six tiles. All Time-scoped. All carry a change chip.

| Tile | Counts | Kind |
|---|---|---|
| **Issues Created** | Complaints raised in the window | Time-scoped |
| **Issues Resolved** | Complaints resolved in the window, whenever raised. A Closed complaint is not counted | Time-scoped |
| **Open Backlog** | Complaints open at any point in the window, still open today. Shrinks over time for a past window | Time-scoped |
| **Reopened** | Complaints that were resolved and came back, in the window | Time-scoped |
| **Repeat Complaints** | Same room, same category, within 30 days of the last one resolving. Section 3 | Time-scoped |
| **Average Resolution Time** | Average from raised to resolved, in hours, over complaints resolved in the window. Counts only resolved complaints; a complaint open for a year is not in it. Shows how many it is built on | Time-scoped |

The design labels the fourth tile "Reopen Rate" in one place and "Reopen Issues" in another; it is a count and reads **Reopened**.

Created and Resolved are not two halves of one set: a complaint resolved this month may have been raised months ago. Say it once here so nobody tries to make them add up.

## 6. View all sheet

Opens from the Overview Snapshot. Everything that earns a line but not a tile. Every row is Time-scoped, counted inside the window; no row carries a change chip.

| Row | Meaning |
|---|---|
| **Net flow** | Created minus Resolved for the window. Positive means the pile grew |
| **Median resolution time** | The middle complaint's time from raised to resolved, beside the average. A wide gap means a few slow ones are dragging the average |
| **Oldest open complaint** | The one complaint that has waited longest, and for how long. Nothing else on the screen names a single complaint |
| **Reopened rate** | Reopened as a share of complaints resolved in the window |
| **Repeat rate** | Repeats as a share of complaints raised in the window |
| **Tenant complaint rate** | Tenants who raised at least one complaint, as a share of tenants living there. Tells everyone-is-unhappy apart from one-person-is-loud |
| **Worst rooms** | The five rooms with the most complaints in the window, worst first, each with its count and what it was mostly about. View more when there are others. Rooms with no complaint do not appear; complaints with no room recorded are not attributed |

The insight for the Overview lives here, not on the tile row, which has no room for a sentence: when repeats are rising while resolution time is falling, work is being closed faster than it is being fixed. It stays silent when the counts are too small to say so.

---
## 7. Current Status

Every complaint in the window, grouped by where it stands now, in a donut. Below it, the urgency split of the open ones. At the bottom, the overdue count with its action.

| Row | Meaning |
|---|---|
| **New** | Raised, nobody has picked it up yet |
| **In Progress** | Being worked on. Every status that is not New, Resolved, Closed or Reopened |
| **Resolved** | The problem was fixed |
| **Reopened** | Was resolved, then came back. A reopened complaint sits here, never in New |
| **Closed** | Ended without a fix. Reads zero until a way to mark it exists, section 1 |
| **Urgent** | Open complaints from the window marked urgent |
| **Non-Urgent** | Open complaints from the window not marked urgent |
| **Overdue** | Open complaints from the window past their target time, with Send Reminder |

The centre of the donut names the total complaints in the window. Where the property is using the default target time, one line under the overdue count says so: "Using the standard target." Where the property has set its own, no line.

**Test it:** a complaint raised in the window, resolved, then reopened appears once, in Reopened. A food complaint open for two days at a property with no target set is overdue; a plumbing complaint open for two days at the same property is not.

## 8. Ageing Distribution

Every open complaint, by how long it has been waiting, in four bars: **0–7 days · 8–15 days · 16–30 days · 31+ days**, newest on the left. Live: it ignores the filter and says "As of today" beside its title, with no dropdown of its own. It does not add up to Open Backlog, and should not: one asks what came in this window and is not done, the other asks how long everything still open has been sitting.

For a complaint that was resolved and came back, the wait counts from when it came back, not from when it was first raised. The three bars past seven days show red: no target time on this screen is longer than a week. The first bar can hold late complaints, since a food complaint has a one-day target, so a plain first bar does not mean nothing is late.

**Test it:** a complaint raised 200 days ago, resolved after a week, and reopened 10 days ago sits in the 8–15 bar.

## 9. Response Time

Two numbers, both over complaints raised in the window, both showing how many complaints they are built on. Target: **one hour**, both, fixed by the system, the same for every category, nobody configures it. The badge is green when the average beats an hour and red when it does not; nothing in between.

| Row | Meaning |
|---|---|
| **Time to Assign** | Time from raised until a named team member is put in charge. Only complaints that were assigned; where none were, a dash, never zero |
| **First Response** | Time from raised until anything first happens: someone picks it up, adds a note, or moves the status. Where the first thing that happens is the resolution, that counts |

The clock counts every hour, day and night. A complaint raised at 11pm and assigned at 8am waited nine hours. Whether to count working hours only is an open item, section 22.

## 10. Cost of Issues

Only complaints with money spent on them: an expense linked to the complaint. All four Time-scoped, and all carry a change chip. No red on this card.

| Row | Meaning |
|---|---|
| **Total Impacted** | Money spent fixing complaints in the window. Shows how many complaints it covers |
| **Open Issues** | Of those complaints, how many are still open |
| **Avg Cost** | Total Impacted divided by the complaints it covers |
| **Resol. Time** | Average from raised to resolved, for complaints with money on them |

Insight line, where the card has room: whether these complaints are taking longer to close than the rest, since money usually means an approval is waiting. Silent when too few to say.

## 11. Efficiency & Quality

Three numbers over complaints raised in the window.

| Row | Meaning |
|---|---|
| **Avg Resolution Time** | Average from raised to resolved. The same number as the Overview tile, repeated on purpose: this section is incomplete without it |
| **Escalation Rate** | Complaints that moved up to the next responder, as a share of complaints raised. Cannot be built until escalations are recorded, section 1 |
| **SLA Rate** | Complaints finished within their target time, as a share of every complaint whose outcome is decided. Carries a change chip, green when rising |

SLA Rate counts a complaint the moment its outcome is known: resolved inside the target is a keep, past the target is a miss whether or not it has been resolved. Complaints still inside their target are not yet counted either way, so a recent window's figure settles a few days after it ends. Shows how many complaints it is built on. **Test it:** a complaint raised on the 28th with a seven-day target has not passed or kept by the 31st, and is not in the month's figure until the 4th.

## 12. Team Performance

Two tabs. Both Time-scoped.

**Top Performer.** Team members ranked by promises kept: complaints finished within their target, shown as a count out of a count, "9 of 10 on time", never a bare percentage. A complaint past its target counts against whoever holds it, resolved or not. A complaint that moves between people counts for whoever holds it now. Someone who has left the team gets no row; their open complaints move to unassigned.

**Team.** Two bars per person: complaints that became theirs in the window, and complaints they resolved in the window. Both cover the same window, so the pair says whether someone is keeping up with what they are given, not what is on their desk right now: the two bars are different complaints. One more pair at the end for **Unassigned**: complaints that arrived in the window and never got an owner, and complaints resolved with nobody assigned.

Both tabs carry the unassigned line, because a ranking of the people who have work says nothing about the work nobody has. When nobody at the property has been assigned anything, neither tab draws a chart; the card says so, section 19.

Insight line, where the card has room: how much of the window's work never reached anyone on the team.

## 13. Issues Sources

One card, two tabs. Both Time-scoped; both sort highest to lowest and carry each row's share of the total.

**Reporter.** Where complaints come from: **Tenant App · Complaint Bot · Team Members · Admin**, plus **Unknown** for complaints with no recorded source, hidden at zero. Each complaint lands in exactly one bar: a tenant complaint that came through the bot is Complaint Bot only. Admin and Team Members are told apart by role.

**Category.** The four largest categories in the window, then **Others**, which holds everything else and opens in two steps: tapping it lists the full remainder, and each row inside drills on its own. Four rather than the suite's usual three because complaints spread wider across categories than money does across bill types; four named rows cover about three fifths of everything.

The category names in the design are placeholders. Real ones are Electrical Appliances, Plumbing, Furnishings/Carpentry, Electricity and Power, Cleaning, Internet, House Keeping. Maintenance is not a category anyone uses and not a separate module.

## 14. Issues Trend

Complaints raised against complaints resolved, month by month, stacked. Its own range control: **6 · 12 · 24 months**. It ignores the filter and says so beside its title. Plot the movement, not the level: a property can sit at forty open complaints all year while handling four hundred, and a line of the level draws straight through all of it.

## 15. Ratings & Feedback

Ratings tenants left on complaints resolved in the window.

| Row | Meaning |
|---|---|
| **Average Rating** | Average of one-to-five star ratings, with a face set by the average: five bands, under 2, 2 to under 3, 3 to under 4, 4 to under 5, and 5. Shows how many ratings it is built on |
| **By stars** | Ratings grouped by score, one row per star |
| **Pending Rating** | Resolved complaints from the window the tenant has not rated, with **Send Reminder** |

Only ratings of one to five count. **Test it:** the average can never display above five. Where a property has resolved complaints and no ratings, no face and no average, one line saying so, and the reminder button stays.

## 16. Property Performance

Only when more than one property is in view. Each property with its complaints resolved out of its total for the window, sorted by open complaints, most first, and each row's share of the total. Its own dropdown, offering the same options as the filter at the top; the top filter pulls it back into line. Raw counts, not per-bed: the question here is where to send someone this week, and forty open complaints is forty pieces of work whatever the building's size. Shows how many complaints each row is built on, since one month at one property is often one or two.

---
## 17. What each number opens

<details><summary>The rules, the window rule, the tap matrix, and what the list can already do</summary>

### The rules

A drill filters the complaints list. It never re-scopes the screen; the trend bars are the one exception and they move the period. The destination opens on the same window and the same properties, and the back control names this screen. Records add back to the number tapped. Rates and averages are not tappable; the counts beside them are.

### When the window changes what a tap shows

Live taps carry nothing: the list opens as of today. Period taps carry the window, and the list shows those complaints as they are today, naming any difference on arrival: "showing 11 of the 14; the rest have since been resolved."

### The tap matrix

Ready? reads ✅ where the list can already land, ❌ where it cannot, and every ❌ names what has to be added. **These filters ship with the screen, not after it.**

| You tap | What opens | Arriving filtered to | Ready? |
|---|---|---|---|
| **Overview Snapshot** | | | |
| Issues Created | Complaints list | raised in the window | ✅ |
| Issues Resolved | Complaints list | resolved in the window | ❌ filter to add: resolved date |
| Open Backlog | Complaints list | open at any point in the window, still open | ❌ filter to add: active in period, the same one the homescreen fix needs |
| Reopened | Complaints list | reopened in the window | ❌ filter to add: reopened. The phone app's current "Reopened" option returns nothing every time |
| Repeat Complaints | Complaints list | the repeats | ❌ filter to add: repeat |
| Average Resolution Time | not tappable | — | — |
| **View all sheet** | | | |
| Oldest open complaint | that complaint | — | ✅ |
| Worst rooms, any row | Complaints list | that room, in the window | ❌ filter to add: room |
| Other rows | not tappable | — | — |
| **Current Status** | | | |
| New, In Progress, Resolved | Complaints list | that status, in the window | ✅ |
| Reopened | Complaints list | reopened, in the window | ❌ filter to add: reopened |
| Closed | Complaints list | closed, in the window | ❌ fact-not-recorded: nothing can be marked Closed yet |
| Urgent, Non-Urgent | Complaints list | open, that priority, in the window | ✅ |
| Overdue | Complaints list | open, past target, in the window | ❌ filter to add: past target |
| **Ageing Distribution** | | | |
| Any bar | Complaints list | open, waiting that long | ❌ filter to add: waiting time. Bars are not tappable until it lands, and the card says so |
| **Response Time** | | | |
| Either number | not tappable | — | — |
| **Cost of Issues** | | | |
| Total Impacted, Open Issues | Complaints list | with money linked, in the window; Open Issues adds still open | ❌ filter to add: has money linked |
| Avg Cost, Resol. Time | not tappable | — | — |
| **Efficiency & Quality** | | | |
| SLA Rate | Complaints list | past target, in the window | ❌ filter to add: past target |
| Escalation Rate | Complaints list | escalated, in the window | ❌ fact-not-recorded: escalations are not written down |
| Avg Resolution Time | not tappable | — | — |
| **Team Performance** | | | |
| Any person, either tab | Complaints list | assigned to them, in the window | ✅ |
| Unassigned | Complaints list | no assignee, in the window | ⚠ works on the phone app, not on web |
| **Issues Sources** | | | |
| Any reporter bar | Complaints list | raised by that kind of reporter, in the window | ❌ filter to add: reporter type. The list filters by a named person, not by type |
| Any category row | Complaints list | that category, in the window | ✅ |
| Others | the remainder sheet, then the list per row | that category | ✅ |
| **Issues Trend** | | | |
| Raised segment | Complaints list, period moved to that month | raised that month | ✅ |
| Resolved segment | Complaints list, period moved to that month | resolved that month | ❌ filter to add: resolved date |
| **Ratings & Feedback** | | | |
| Any star row | Complaints list | resolved in the window, that rating | ❌ filter to add: rating |
| Pending Rating | Complaints list | resolved in the window, unrated | ✅ |
| **Property Performance** | | | |
| Any property | Complaints list | that property, in the window | ✅ |

### What the list can already do, and what has to be added

Today the complaints list filters by: when raised, status, category and sub-category, priority, assignee, who raised it by name, and property. The phone app also offers unassigned; the web app does not.

**To add, in one place, so every number above can open:** resolved date · active in period · reopened · repeat · room · past target · waiting time · has money linked · reporter type · rating. Unassigned on web. Escalated once escalations are recorded, and Closed once it can be set.

**Three faults in the list itself, found on the way, all affecting where taps land:** the "Issue Time" filter is offered on both apps and changes nothing about the rows returned. The phone app's "Reopened" status option matches no complaint. Unassigned exists on the phone and not on web.

### What the destination says when you arrive

The suite's three requirements stand: name the slice in place of the fixed title, show the active filter without scrolling, and name the filter in the empty state with a way on. Shared work, still unowned.

</details>

---

## 18. Who can see this

**Anyone who can view complaints.** The suite rule: each analytics tab takes the permission of the records it describes. No partial state; every card, including Team Performance, is visible to anyone who can see complaints. If someone's access is narrowed to certain properties, every number counts only those properties and still matches what the drill returns. Without permission: the standard lock, *"Analytics Restricted, You don't have permission to view these analytics. Request access from your admin,"* with Request Access.

---
## 19. What each card shows when it is empty, healthy or broken

An empty card here is usually good news, the opposite of every money screen. Nothing for this screen's states is drawn yet; this section is the brief.

### The zeros, told apart

| Situation | What shows |
|---|---|
| Never raised a complaint here | The whole-screen state below. No cards render. For complaints there is no separate setup stage: raising one is the setup |
| In-window zero: nothing raised this window, complaints exist elsewhere in time | Cards draw. Time-scoped numbers read zero, plain, no message. Live numbers show what is genuinely open |
| Complaints came in and none were resolved | A real zero on Issues Resolved with its chip red. Never dressed as good news |
| Good-news zero | The healthy states below |
| Not yet in use: no ratings, no money linked, nobody assigned | The empty states below. Not a gap and not success: a part of the product this property has not started on. Shrinks as it does |
| Not recorded | Rooms, categories and sources that were never captured, named as unknown, never guessed into a bucket |
| Failed | "Couldn't load this" with Retry, never a healthy message, never a zero |

### Not set up

When the property has never had a complaint: *"No complaints have been raised here."* with an **Add Complaint** button, the same action the complaints list already offers. This is the most common state the screen will ever show; eleven boxes each explaining their own emptiness is the wrong answer to one situation. Plainly factual, no exclamation: an empty complaints screen almost never means perfection, it means the feature is not in use.

### Healthy: good news, no CTA

| Card | Reads |
|---|---|
| Open Backlog at zero | *"Nothing open right now."* |
| Overdue at zero | *"Nothing is overdue."* |
| Ageing Distribution at zero | *"Nobody is waiting."* |
| Reopened at zero | *"Nothing has come back."* |
| Repeat Complaints at zero | *"No problem has come back in the same room."* |

No button on any: there is nothing to do, which is the point.

### Empty: not yet in use

| Card | Reads | Button |
|---|---|---|
| Ratings & Feedback, resolved complaints but none rated | *"No ratings yet. Tenants are asked to rate a complaint once it's closed."* | **Send Reminder**, when there are resolved complaints to remind about |
| Cost of Issues, no money linked | *"No costs linked to complaints yet."* | none |
| Team Performance, nobody assigned | *"Nobody has been assigned a complaint here yet."* No chart on either tab | none |
| Response Time, Time to Assign with nothing assigned | A dash in that half, never a zero. First Response still draws | none |

### Not recorded

Reporter draws an Unknown bar for complaints with no recorded source, hidden at zero. Worst rooms and Repeat Complaints leave out complaints with no room recorded and say so. Never claimed as a category or a source.

### Failed

Per the suite rule, per card, with Retry refetching that card alone.

---

## 20. What this screen is not

- **Not the complaints list.** Nobody assigns, changes a status or replies to a tenant here; every number opens the list, where that happens. Three exceptions, all bulk or create actions with nowhere else to live: Send Reminder on overdue, Send Reminder on unrated, Add Complaint on the empty screen.
- **Maintenance is not a separate section.** About 5 complaints in 10,000 carry a maintenance-shaped category; what people call maintenance is Electrical, Plumbing and Carpentry, already inside Complaints.
- **Not what a complaint is about.** No descriptions, photos or conversation. Category is as far as this screen goes.
- **Not the escalation ladder.** How often complaints escalate, once recorded; who it went to and at what level belongs to the complaint.
- **Not money recovered from tenants.** Total Impacted is money spent. Billing damage back and collecting it is parked, section 22.
- **Not where target times are set.** The screen measures against them; they are set when a property is onboarded onto the complaint bot.
- **Not leads or enquiries.** Those belong to the Leads screen.
- **Not a staff management tool.** Team Performance shows who is keeping promises; reassigning work happens in the list.

---

## 21. Build guidance

1. **Complaints and their history are stamped on two different clocks, five and a half hours apart.** Every duration on this screen crosses that seam. *Test it:* no complaint on the screen ever shows as resolved, assigned or responded to before it was raised.
2. **Repeat Complaints, Reopened, resolution time and first response all read from a complaint's history, not from the complaint alone.** A complaint's own record does not carry when it was resolved, assigned or reopened. History begins May 2024; nothing before that can be measured, and the sheet says so where it bites.
3. **Reopened is read from a status moving out of Resolved, never from a reopen label.** *Test it:* on a property with recent reopens, the tile shows more than the handful the label would find.
4. **The ageing wait for a reopened complaint counts from when it came back.** *Test it:* raised 200 days ago, resolved, reopened 10 days ago, still open: it sits in the 8–15 bar.
5. **Unassigned is a layer inside every status, never a slice beside them.** *Test it:* New plus In Progress plus Reopened plus Closed equals every open complaint, with unassigned counted once, inside those.
6. **The average rating counts only one to five.** *Test it:* the screen can never show a figure above five.
7. **Category rows group on the name people see, not on the stored string.** *Test it:* Internet with and without a trailing space is one row; Waterproofing & Paint however its ampersand is stored is one row.
8. **The reporter bars never overlap.** *Test it:* the bars sum to Issues Created; a bot complaint sits in Complaint Bot and not in Tenant App.
9. **The overdue count and SLA Rate use each complaint's own target time, sub-category first, then category, then the default.** *Test it:* two complaints of the same category but different sub-categories with different targets go overdue at different times.
10. **SLA Rate counts a past-target complaint as missed whether or not it has been resolved.** *Test it:* a property that never resolves its worst complaint cannot keep a perfect rate.
11. **Team Performance counts a complaint for whoever holds it now.** *Test it:* a complaint moved from one person to another shows on the second person only.
12. **The Send Reminder on ratings never messages the same person about the same complaint twice within seven days.**
13. **Every average and rate shows how many complaints it is built on.** The typical property raises a handful a quarter; a bare average of three complaints is not a trend.
14. **The Last 7 days option must count backward.** The shared date control offers presets that read forward and resolve backward. *Test it:* a complaint raised yesterday appears; one raised three weeks ago does not.
15. **The whole-screen empty state fires only when the property has never had a complaint.** A quiet week with history draws plain zeros, not the takeover.

---

## 22. Open items

1. **Renaming the New slice to "Not started".** Most complaints in it are weeks or months old. Product.
2. **Working hours only, on Response Time.** Whether the one-hour target should skip the night. Internal discussion and UAT.
3. **Money billed to tenants and recovered.** A separate line beside money spent, once wanted. Product.
4. **Resolution time for money complaints against the rest.** Delivered as an insight line for now; a side-by-side figure if the line proves too quiet. Product.
5. **The complaint bot's per-category target times collapse to one per category on the way in.** The screen asks for sub-category targets; where the property set them, they should be kept. Engineering.
6. **A record of each escalation**, so Escalation Rate can be built. Engineering.
7. **A way to mark a complaint Closed**, for tenants and managers. Product and engineering.
8. **The homescreen's Issues card and this screen count differently** until its ticketed fix lands. Engineering.
9. **The complaints list's Issue Time filter changes nothing; its Reopened option matches nothing; Unassigned is missing on web.** Engineering.
10. **Ratings are being written with the value 6, still, this year.** Something other than the app is writing them; find it. Engineering.
11. **The Expense screen's empty states are built from a duplicated Complaints card**, and one Inventory card sits among them. Goes back to those two sheets. Design.

---

## 23. Design file: what needs fixing

<details><summary>The full list, one numbering end to end</summary>

**Wrong**

1. The last ageing bar reads "31-45 Days" with no plus; anything older has nowhere to go. Becomes **31+ days**, matching Inventory.
2. The Current Status donut centre says "Unresolved Complaints" while one slice is Resolved. The centre names the total.
3. The Issues Sources axis stops at 40 while three of four bars exceed it.
4. The Category tab title is typed "Issues Sourcesc".
5. The Trend chart says "Last 6 Months" and draws five points.
6. The ratings gradient has four colour bands for five stars.
7. A loose copy of the Overview tiles outside the frame says "Reopen Issues"; the live one says "Reopen Rate"; both become **Reopened**.
8. SLA Rate is a dial with no mark on it.
9. The Trend chart's legend reads Resolved and Pending; it becomes Raised and Resolved, section 14.
10. The Team tab's legend reads Created; it means assigned, section 12.
11. The category examples are placeholders, section 13.
12. Team Performance's sample names are the engineering team.

**Missing**

13. Every empty, healthy and failed state, section 19. Nothing is drawn.
14. The View all sheet, section 6.
15. A time dropdown on Property Performance.
16. Insight lines on Cost of Issues, Team Performance and Ratings, and one on the View all sheet.
17. An unassigned row on both Team Performance tabs; an Unknown bar on Reporter.
18. Add Complaint on the whole-screen empty state; Send Reminder on the empty Ratings card.
19. Room for three short lines, section by importance: how many complaints Average Resolution Time is built on; that the overdue count is using the standard target; that the ageing bars are not tappable yet.

**Remove**

20. The Trend chart's fifth bar labelled "Move-out", clipped past the right edge, copied from Tenants.
21. The Team chart's fifth bar labelled "Move-out", the same.
22. The hidden "Paid by / Paid to" switch inside Property Performance, and its layer name "Property Expense", both from Expense.
23. The hidden text about short-term guest approvals inside the Overview Snapshot.
24. The onboarding tooltip about swipe gestures rendering over the Team chart.
25. The Advanced Insights frames beside the screen. Everything of value in them has landed on the main screen; deleted.

**Decide**

26. Whether the design's face set has a face for no ratings, or the average and face both hide.

</details>

---

## Where the measured figures came from

<details><summary>Every platform figure a decision rests on</summary>

| Measured | Result | What it decided |
|---|---|---|
| Complaints ever, and in 90 days | 257,647 · 44,170 | Volume is not a question on this screen |
| Status spread | Resolved 85%, Unresolved 13%, Team is Working 1.2%, three others 0.37% together | The donut's grouping over the raw status list |
| The Closed status | Written once in the system's history | Closed reads zero until it can be set |
| Open complaints, and their age | 38,365 · median 444 days · ninetieth percentile 1,122 days | The shape of Open Backlog and the ageing card |
| Age of the New bucket | 78% older than three months, 5% under a week | The Not-started rename logged as open |
| Ageing spread today | 2,424 · 1,661 · 1,450 · 1,189 · 31,641 beyond 45 days | The last bar becomes open-ended |
| Resolution time | median 47 hours · mean 137 · ninetieth percentile 375 | Average kept over median; 22% take over a week |
| Resolution spread | 36% inside a day · 22% beyond a week | The seven-day default flags the slowest fifth |
| The two clocks | Complaints in IST, their history in UTC, 5.5 hours apart | 13% appear resolved before raised without the correction; build guidance 1 |
| Unassigned against status | 95% of Unresolved unassigned; 40% of Team is Working has no team member | Unassigned is a layer; as a slice it would report 72,752 against a true 38,365 |
| Escalation records | none: no column, no status, no history event | Escalation Rate waits on recording |
| A stored due date | null on all 257,648 | No due date exists to measure against |
| Urgent flag | true on 62% of recent open complaints | Not a priority signal; shown as it is |
| Room recorded | 62% ever, 87% in 90 days | Repeats are buildable and getting more so |
| Repeats, room and category, 30 days | 10,152 in 90 days, about one in four | The repeat definition |
| The same at property level | 67% of complaints would count | Confirms the room is the right grain |
| Repeats against reopens | 321 are both; repeats eight times more common | Different signals, both kept |
| Reopens | 4.8% of resolved; the reopen label captures 31 events in all history | Read from status changes, never the label |
| Reopened and still open | 638, under 2% of open | A thin, real slice |
| Tenants complaining twice | half of all complaining tenants; one of them 73 times | Tenant complaint rate earns its place |
| Ratings | 7% of resolved rated; 63% of stored ratings are the value 6; raw average 4.99, real 3.3 | Only one to five count |
| Rating spread | one star the second most common | Star counts drill to the list |
| Money linked | 0.41% of recent complaints; median ₹266 to ₹435 | Real and early; kept, with coverage shown |
| Category recorded | 98%, from about 42 values | The category card is well supported |
| Category faults | a trailing space undercounts Internet by a third; an encoding fault splits one category | Build guidance 7 |
| Maintenance as a category | 21 of 44,170 | Not a separate module |
| Who raises | tenants 83% · staff 14% · unrecorded 3% | The Unknown bar |
| The complaint bot | 41 complaints ever, all since late July | Its own bar, because it is the channel being rolled out |
| Assignment | 41% of complaints; the median property has nobody assigned; 12% have three or more people | The empty state on Team Performance |
| First response | for 51% the first event is the resolution | Resolving counts as first response |
| Property scale | median four complaints a quarter, three open; the busiest 2,156 | Every card reads with single digits; every average shows its count |
| Dormant backlog | 46% of open complaints sit on properties with no complaint in three months | Why Open Backlog needed the active-in-period model |
| Target coverage | set for 324 properties, 10% of active ones, 30% of complaints | The default target and its one-line notice |
| Category rollup | four categories cover about three fifths of complaints | Four plus Others rather than three |

</details>
