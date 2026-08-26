# Sibling audit: 00, 02 and the README, 27 August 2026

The three structural failures fixed in `01 — Build Verification Log.md` on 27 August were audited for
in its three sibling docs. **Nothing was rewritten.** This file records what was found, so the owner
can rule on what to fix.

The pattern being tested for is written up in
`~/.claude/skills/module-redesign-pipeline/references/patterns.md`, last entry.

## Count

| | Fault 1 verbs with no objects | Fault 2 state in more than one place | Fault 3 no last-checked record | Siblings | Total |
|---|---|---|---|---|---|
| 00 Tracker | 1 | 5 | 2 | 5 | 13 |
| 02 Suggestions | 1 | 2 | 1 | 4 | 8 |
| README | 0 | 2 | 1 | 2 | 5 |

Checked against repo state at 27 August 2026 and backend `origin/master` commit `0e8cc713b`.

## 00 — Manager App Analytics Tracker

Last modified 18 August 2026. It contains zero references to `01`, to `02`, or to any D or F number,
while calling itself the file that holds what has already been decided across all screens.

| # | Fault | What was found | Proof |
|---|---|---|---|
| T1 | 2 | Decisions now live in two systems with no link between them | 00 mentions "verification log" 0 times, "suggestions register" 0 times, D1-D33 0 times, F1-F108 0 times |
| T2 | 2 | Screen version lives in three places, 00 wrong on three of six | Sheets say Expense v3.1, Inventory v2.1, Tenants v2.2; 00 says v3, v2, v2 |
| T3 | 2 | Self-contradiction inside the file | "Still owed" says the backfill queue is owed on Dues and Collection; the Backfill queue section says "The queue is now empty" |
| T4 | 2 | The live owed-work list sits under a heading that says it is retired | "### Still owed" is nested inside "## Superseded — previous next action (Tenants, now closed)" |
| T5 | 2 | Two copies of one claim, false in both | "Info icons remain unwritten across all eight screens" and "No screen has content for any of them". 115 hint entries ship today: dues 25, collection 29, expense 10, occupancy 21, tenant 30 |
| T6 | 3 | No last-checked stamp anywhere in the file | Locked rules carry a locked-on date; no status, question or shared fact carries one |
| T7 | 3 | Shared context still says the backend is unbuilt | "returns placeholder data, scaffolded, not built", Collection "confirmed still a stub". 27 commits, roughly 2,250 lines, all five services, all read line by line since |
| T8 | 1 | No work table exists. Owed work is prose in three places | "Next action" six bullets, "Still owed" two stale bullets, "Backfill queue" three sections all paid |
| T9 | sibling | 8 open questions, no ID, no answer column, no owner, no date. Two are settled inside the same file | Permissions settled 2026-08-07; tenant booking identification listed under "Two questions closed this session" |
| T10 | sibling | 40 locked cross-screen rules, none numbered, so none can be cited or contradicted by name | 01 numbers everything and F106 contradicts P8 by name |
| T11 | sibling | Locked rules live in two sections | The locked block, and "Session addendum — Tenants → New cross-screen rules from this screen" |
| T12 | sibling | The screen inventory has 8 rows; the same file rules Leads a ninth screen inside People, and README lists 9 | "#### Leads becomes a ninth screen, inside People" |
| T13 | sibling | One live locked rule still carries the word D28 retired | "a booking behind a tenant under notice sits inside Occupied". The other three uses in 00 are historical narrative and are correct as written |

Also: 645 lines, 99KB, 60-plus headings, no contents table, in the file every session is told to read first.

## 02 — Suggestions Register

The Suggestion column is good: every row reads where it sits. The fault is the column beside it.

| # | Fault | What was found |
|---|---|---|
| S1 | 1 | "Waits on" reads "Owner" on 7 of 9 rows with no statement of the decision being asked. S3 and S9 carry meaning and show the shape the rest should take |
| S2 | 2 | Status is in the summary table and again in each entry's own "Status:" line, and for S3 and S5 a third time as a ruling in 01 (D28, D18) |
| S3 | 2 | The status legend defines four values; the table uses five. S9 reads "Ruled out of the build; one question open" |
| S4 | 3 | No last-checked stamp, and code citations carry no pinned commit. `controllers/tenant.ts:21248` for `deleteActiveTenant` is line 21331 at `0e8cc713b`, 83 lines out. `tenant.ts:3706` still holds. 01 solved this with D25; 02 never inherited it |
| S5 | sibling | Un-run checks with no answer slot read clean. S5: "Nobody has checked whether a due type has ever been typed as Others", and "Very likely Dues and Collection too, unchecked" |
| S6 | sibling | A defect hides inside a suggestion. S2 carries "A second defect inside that path": the comment says write off to status 4, the code calls `InvoicesRepository.delete(...)`, a hard delete. No F number, no work-queue row, no home in 01 |
| S7 | sibling | Two owner questions have nowhere to go. 01's "What is waiting on you" pulls in S5, S6 and S7 correctly. S2's question (paid bills against a cancelled booking) and S9's question (a note or a fifth row on Journey) are in neither list |
| S8 | sibling | No contents table, and no line saying the next number is S10. That fact lives only in project memory |

## README

| # | Fault | What was found |
|---|---|---|
| R1 | 2 | Version drift, and README and 00 disagree with each other. Sheets say Expense v3.1, Inventory v2.1, Tenants v2.2; README says v3, v2, v2.1; 00 says v3, v2, v2 |
| R2 | 2 | "Source of truth: the Obsidian vault. This repo carries the closed deliverables only." The repo is ahead of the vault (repo README 27 Aug, vault README 24 Aug), and 01 and 02 are live working docs, not closed deliverables. The line sends the next reader to edit the wrong copy |
| R3 | 3 | One stamp exists, on the verification-log row, and it is the right shape. Six version numbers, nine screen statuses and six rules carry none |
| R4 | sibling | "The rules every sheet follows" is six unnumbered rules; nothing can cite or contradict them by number |
| R5 | sibling | The opening line claims "nine screens, one self-contained handoff sheet per screen"; its own table shows three screens with no sheet |

## What came back clean

**Links.** 66 internal links with section anchors in README, 18 wikilinks in 00, 1 in 02. Every file
resolves and every anchor exists. 01's contents-table link to a section that did not exist has no
sibling in these three.

## The recommendation put to the owner

Rewrite 00 first and hard, give 02 a light pass, give the README ten minutes.

00 is stale at the centre, not at the edges: the file that claims to hold every decision has been
blind to the whole build-verification workstream for nine days, and three of its present-tense facts
are now false. The 40 unnumbered rules are the deeper problem, because until they carry IDs the two
files that cite decisions cannot cite them.

**The one question for the owner:** does 00 stay the decision store, or does it become the
method-and-history file and hand live state to 01? The recommendation is to hand state to 01. 00 has
not kept up for nine days; 01 has a work queue that does.

02 needs two real things and the rest is cosmetic: fill "Waits on" with the actual decision, and move
S2's hard-delete defect into 01 where defects live.
