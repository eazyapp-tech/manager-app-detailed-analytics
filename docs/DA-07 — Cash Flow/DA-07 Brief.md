---
title: DA-07 Cash Flow — Brief
date: 2026-05-18
tags:
  - rentok
  - brief
  - cashflow
status: Living document · v0.2.2 (kickoff polish — GST legacy fix scope refined + methodology drill elevated to must-ship)
owner: Sanchay (PM) · co-signed by Eng lead + Design lead before kickoff
time_budget: 2-week build cycle
companion_to: DA-07 Cash Flow Detailed Analytics (legacy PRD V1.3 — historical), DA-07 Build Sheet V1 (engineering), DA-07 Codebase Exploration (Phase 2 LAYER 1+2 findings)
---

> [!INFO] What this is
> A **Brief** for the Cash Flow screen — written before design. What the PM wants the screen to *do*, not how it should look.
>
> **What this is NOT:** an engineering spec or a design doc. Those live in `[[DA-07 Build Sheet]]` and Figma.

---

# DA-07 Cash Flow — Brief

## In one line

Rajesh runs the property all month — collecting rent, paying salaries, returning deposits, refunding occasional disputes — and at month-end can't tell you what he actually made or lost. He pulls Excel, asks his CA, or just guesses. This screen shows the live cash picture for any period: inflows by source, outflows by category, deposit money tracked separately (because it's not his money), with drill into every line. **Last screen in the financial DA suite — sits on top of DA-02/03/04/05/06 and reconciles to all of them. 2-week build to ship V1.**

---

## The operator

The owner / admin / supervisor when they want to **see what the property actually made or lost in a period** — and whether the deposit money on the books is getting confused with operating cash. The homescreen says *something's off*. The other DA screens show inflows, outflows, refunds, expenses individually. This screen sits on top of all of them: it composes the cash-flow picture and flags the operator's "is this property making money?" question.

**Rajesh.** Single-property PG owner. 35–50, Hindi-first, 20–40 beds in Bengaluru / Pune / Jaipur. Runs the day-to-day; at month-end does basic accounting with whoever helps him (small CA, family member, or himself). Opens this screen with a rhythm distinct from the rest of the suite:
- **Monthly close** — month-end + first 5 days of next month, reconciles with CA, decides whether to deploy cash or hold.
- **Mid-month timing check** — when something feels off mid-month, opens to see whether the running tally is healthy.
- **Quarterly review** — every 3 months, longer-range planning (renewal pricing, hiring, expansion).

He leaves each visit with a picture of net cash flow + composition split + (if anything looks off) a follow-up: *"Yeh expense itna kyu?"* or *"Salary aur rent ke baad bachta kya?"*

**Priya** — multi-property owner (5+). Same monthly-close rhythm across her portfolio. Wants to know which property is cash-positive and which is leaking — but the per-property comparison view is V2 (see What we're NOT building).

**Meena / Ramesh bhaiya** — typically don't open this screen. Cash flow is owner-territory (revenue + expense visibility together = full P&L view). V1 gates the screen behind composite `viewInvoices AND viewExpenses` permission — most team members have only one of those, so DA-07 is hidden from their navigation. See Traps for the permission decision.[^1]

[^1]: Personas are composites from ~6 field interviews (Mar–Apr 2026). Same trio as DA-01 through DA-06. RentOk permissions are atomic per phone number (per team member); composite gating means an operator needs BOTH `viewInvoices` AND `viewExpenses` for DA-07 to appear in navigation.

**This screen is period-sensitive — the time chip drives everything.** Unlike DA-06 Liabilities (always-live snapshot), DA-07 is cash flow OVER a period: pick "Last Month" or "Custom range," see what came in vs went out. Default period at app open: "This Month."

**Rhythm fit:** monthly close + mid-month check + quarterly review, ~2-4×/month per operator. Lower frequency than DA-02 (daily/weekly) but higher reconciliation weight (every visit ends in a decision or a CA conversation).

**Drill rule (designer picks entry points):** every line item on DA-07 drills into the source DA screen filtered to the same period + the same composition slice. The full drill map is in Traps + Footer Key Decisions Locked — every inflow row → DA-02 Collections filtered, every outflow row → DA-04 Expenses filtered, every refund row → DA-03 Refunds filtered, deposit balance → DA-06 Liabilities (live), discount-impact → DA-02 with "Has discount" chip, refund-impact-on-collection → DA-02 with NEW "Has refund" chip, dues-bridge → DA-01 Dues. DA-07 is the meta-aggregator; all the actionable detail lives in the sibling DA screens.

---

## The problem

Three pains, same root cause — no composed live cash-flow view exists today:

1. **No idea what I actually made or lost this period.** Rajesh sees collections in DA-02 (₹4L came in) and expenses in DA-04 (₹2.8L went out), but composing them mentally is error-prone — and refunds (DA-03), discounts redeemed (DA-05), and deposit movements (DA-06) all sit in separate screens. The net cash flow number exists nowhere in product; operator either pulls Excel or asks the CA.
2. **No idea if my deposit money is getting mixed up with operating cash.** Deposits are tenant money the operator must return; treating them as revenue inflates the "I made ₹5L this month" feeling and leads to over-deployment. Today there's no clean separation between operating cash flow and deposit movement — operators conflate them.
3. **Can't easily forward a clean cash picture to my CA.** A CA reading a forwarded screenshot expects to see "Net Operating Cash Flow" with GAAP-readable structure. The existing Excel report has bugs (deposit refunds counted twice, GST sign error) — operators don't trust it; CAs reject it.

Cash flow you can't see, you can't deploy or save against. The bet isn't to make more — it's to make the made-vs-spent picture clear so the operator (and their CA) can plan against a number they trust.

**Why now:** DA-07 is the LAST screen in the financial DA suite. Without it, DA-02/03/04/05/06 each show a slice and the operator does the math in their head (or in Excel, or via their CA). Every cycle without this screen is another month-end where the operator either doesn't reconcile (and over-deploys cash) or burns 2+ hours of CA time per property. The suite is incomplete without the meta-aggregator.

---

## What Rajesh does today

- **Pulls the existing cash flow Excel report at month-end.** Exists at `POST /reports/cashflow-report`. Has known bugs — deposit refunds counted twice (HB-B1) and GST sign error. Operators use it but the CA-reviewed numbers don't match the operator's expectation; trust is brittle.
- **Mental composition across DA-02 + DA-04.** Opens Collections, notes the number; opens Expenses, subtracts. Misses refunds (DA-03), discounts redeemed (DA-05), and deposit movement (DA-06). Result: a guess, off by 5-15% in either direction.
- **Asks the CA at month-end.** ₹500-2000/month per property in CA fees just for monthly cash-flow reconciliation. CA pulls the Excel, fixes the obvious bugs by hand, sends back a number. Round-trip = 2-5 days.
- **Goes by bank balance.** Crudest workaround: looks at bank balance start vs end of month, subtracts. Ignores credit-period dynamics (rent collected this month is for last month's stay, deposits sitting on books, etc.). Often wrong by ₹50K-1L per property.

All four workarounds coexist — operators with different literacy levels and CA relationships use different combinations. None give the live + composition + drill picture in one place.

---

## What we must ship — and what we cut if time runs short

**Time budget: 2 weeks.** Why 2 and not 6: DA-07 is the meta-aggregator the suite has been waiting for — without it, every other DA screen is half-useful (operator sees the slice, can't see the whole). Two backend devs (1 senior + 1 junior) on this, plus separate frontend and separate web teams. **DA-07's own endpoints are NEW BUILD** (list, widget, filter_options, Excel link) but the inflow/outflow aggregators already exist (Collections + Expenses widgets exclude mode 211/288 correctly — verified at Phase 2). DA-07 composes them; no parallel aggregator math needed for the major components. **Added scope in V1: 1 new filter chip on DA-02 Collections list — *"Has refund applied"* (parallel to DA-05's "Has discount" chip pattern) — for the drill from DA-07's "Refunds reducing past inflows" line item.** ~1 extra dev-day. The cut order below is the contingency, not the expected outcome.

**Must ship (V1 is not V1 without these):**
- **Q1 — Operating Cash Flow hero + methodology drill.** Period inflows (rent + utilities + advance, mode 211/288 excluded) − period outflows (salary + operating expenses, EXCLUDING Deposit-class expenses + non-deposit refunds). Sign-coded green/red/neutral. The number the CA recognizes. **Methodology drill is MUST-SHIP (not stretch)** — operator can tap the hero to see the formula breakdown (inflows by source + outflows by category + Mode 211/288 exclusions + discount reduction + refund adjustment + GST treatment). Without this, CA receiving a forwarded screenshot defaults to trusting the legacy Excel (which shows columns) because they can't trace the DA-07 number to its components. See Traps.
- **Q2 — Section A breakdown.** Inflows by source (Rent / Electricity / Utilities / Advance / − Discounts redeemed) + outflows by category (Salary / Rent / Electricity / Maintenance / Other / − Non-deposit Refunds). Every line drills to the source DA screen filtered.
- **Q3 — Section B Deposit Money In/Out.** Deposit collected + Deposit returned, tracked SEPARATELY from operating cash flow. Net deposit movement shown. "View deposit balance →" drills to DA-06 Liabilities (live snapshot, period-pill disabled per HB-D1).
- **Q4 — MoM chip (3-state).** Improvement (▲ green) / Regression (▼ amber) / Sign-reversal-to-negative (▼ red). Collapsed from 4 states in PRD v1.2.

These four answer 90% of *"what did I make this period + what happened with deposit money + is it getting better or worse?"* **No-ship gate (rewritten precise per v0.2 GST-in-legacy decision):** Q1 hero on DA-07 MUST use the new aggregator math with both HB-B1 dedup logic AND GST treated as subtraction. Legacy Excel report ALSO gets the GST sign fix in V1 (2-line code change at `generateCashFlowReport.ts:427-428`); legacy retains the HB-B1 double-count until V2 migration. **Result:** DA-07 + legacy Excel agree on GST treatment, disagree on deposit-refund treatment (off by the deposit-refund amount). See Traps for the reconciliation gate + the explicit footnote that discloses precisely which fixes are where.

**Protect at stretch tier (last to cut):**
- **Q5 — Per-bed Cash Flow** (multi-property only). Hero ÷ live occupied beds. Skip for solo Rajesh.
- **Q6 — YTD strip.** Current Indian FY (April 1 – March 31) cumulative cash flow. Helps annual planning.
- **Q7 — Conditional callouts.** Deposit-dependency (≥30% deposit inflow + ≥5 deposit-holders + property ≥60 days), Vendor concentration (top vendor >30% of outflow + ≥5 expenses + ≥₹15K), Capex flag ("Other" >25% + single expense >₹50K). Each is a maturity-gated warning.
- **Q8 — Mid-month timing nudge.** Conditional helper text when day < 25 AND "This Month" preset AND period contains today: *"Mid-month — partial-period view. Wait until day 25+ for a representative full-month picture."*

**If time tightens, cut in this order (operator-pain-ranked, least pain first):**
1. **Q8 — mid-month timing nudge.** Niche conditional (only fires under specific combination); operator can self-orient with the date chip.
2. **Q7 — conditional callouts.** Each callout is a maturity-gated warning that operator can spot in the Section A/B breakdowns anyway. Cut all three together, not individually.
3. **Q6 — YTD strip.** Annual lens; operator can switch the period chip to "Current FY" to get the same view.
4. **Q5 — per-bed cash flow.** Niche for multi-property; solo Rajesh gets nothing. Hero still shows total cash flow.

**Not cuttable: the new "Has refund applied" chip on DA-02 Collections list.** Operator-side cost of cutting = no way to drill from DA-07's "Non-deposit refunds" line into the underlying collections that got refunded; the screen becomes a read-only number for that line. Same reasoning as DA-05's chip-must-ship and DA-06's chips-must-ship decisions. Chip is V1 must-ship alongside the Q1-Q4 base.

**Conditional cut: nothing is conditional in V1.** Unlike DA-05 Q6 (depended on HB1 + NEW-1 fixes), DA-07's must-ship questions depend on building the new correct aggregator from already-existing reusable Collections + Expenses aggregators — all inside V1 scope, no pre-conditions.

---

## What each question needs

Each line describes the *information* — not how it should look. **How** the screen shows it is the designer's call. Tapping routes per the drill chain in Traps + Key Decisions Locked.

1. **Operating Cash Flow hero.** One signed number for the period: `(SUM rent + utilities + advance collected − discounts redeemed) − (SUM operating expenses EXCLUDING Deposit-class − non-deposit refunds)`. Mode 211/288 excluded (already handled by Collections + Expenses aggregators). Sign-coded. Subtitle: "Net Operating Cash Flow · [period]" — CA-readable label.
2. **Section A breakdown.** Inflows split by source + outflows split by category. Each line shows amount + drills to the source DA screen filtered to period + composition. Drill map in Key Decisions Locked.
3. **Section B Deposit Money In/Out.** Deposit collected this period + Deposit returned this period + net movement. Tracked separately from Q1 hero — these are not the operator's money, they're a liability. Drill to DA-02 (deposit inflows) + DA-03 (deposit outflows) + DA-06 (live balance, period-pill disabled).
4. **MoM chip (3-state).** Improvement (▲ green — same-sign growth OR sign-reversal-to-positive); Regression (▼ amber — same-sign decline > 10%); Sign-reversal-to-negative (▼ red — was positive, now negative). 3-state collapsed from 4 in PRD v1.2.
5. **Per-bed Cash Flow** (multi-property only). Hero ÷ live occupied beds. Helps Priya compare property economics. Live bed count, not period-average.
6. **YTD strip.** Current Indian FY (April 1 – March 31) cumulative cash flow. Auto-rolls based on current date.
7. **Conditional callouts.** Three maturity-gated warnings: deposit-dependency / vendor-concentration / capex. Each fires only when its three guardrails (composition % + count + amount) hold simultaneously.
8. **Mid-month timing nudge.** Helper text when day < 25 + "This Month" preset + period contains today. Saves operator from mid-month panic on partial data.

---

## What we're NOT building this cycle

- **No trend chart.** Same as DA-06: no `cashflow_snapshot` / `daily_cashflow` / `monthly_cashflow` table exists in the codebase (verified by codebase-wide grep). V1 cannot show trajectory without either backfill cron + new table (~3-4 dev-days, breaks the budget) or on-demand re-derivation per period (expensive, fragile). V2 builds the snapshot infrastructure for DA-06 AND DA-07 together — natural pairing. V1 shows current period only; honest framing in screen header.
- **Partial fix on the legacy Excel report (`generateCashFlowReport.ts`).** Production has two real bugs there: HB-B1 (deposit-refund double-count at line 393) AND a NEW GST sign bug at lines 427-428 (GST is *collected from tenant, owed to govt* — currently added to gross profit, which inflates; header label says "GST Collected (-)" but math is "+"; label contradicts code). **V1 strategy:** (a) **fix the GST sign bug in legacy in V1** — 2-line code change at L427-428 with two corrections: (i) at L427, GST should be SUBTRACTED from gross profit (currently `grossProfit + monthlyTotals.gst` — flip sign); (ii) at L428, the Net Profit line currently doesn't subtract GST at all AND also silently drops `adjustedFromAdvance` (verification: line reads `'Net Profit (GP - GST Collected)': grossProfit - adjustedFromDeposit` — header label promises GST subtraction but math doesn't include it, and `adjustedFromAdvance` from L427 is missing). Engineering fix: align L428's math with what the header label promises. No cron/email regression risk (same code path, corrected arithmetic). Operators stop forwarding wrong-CA-number Excel immediately. (b) **DO NOT fix HB-B1 in legacy** — the double-count fix requires bigger refactor (deduplication logic on the Refund/Expense table query); high regression risk on the cron path. HB-B1 fix lives in the new DA-07 aggregator only; legacy Excel stays HB-B1-buggy until V2 migration. (c) **Operator-facing footnote on DA-07:** *"GST math now correct in both DA-07 and Excel. Deposit-refund math correct in DA-07 only (legacy Excel still uses older math — being fixed)."*
- **No passbook double-debit fix.** Some operators record a single deposit return as BOTH a Refund row AND a separate Deposit-Refund Expense row — workflow gap, not system bug. When that happens, the `team_passbook` records TWO debit entries (one from `TeamPassbookRefundService`, one from `TeamPassbookExpenseService` — they're independent factories with no dedup). **V1 acknowledges the behavior is real (some operators do this) but does NOT fix the root cause.** DA-07 V1's correct aggregator simply does the math right at the screen level; operators who check passbook + DA-07 in the same session may notice a mismatch. **V2 fixes the two factory services to prevent double-debit at write-time.** This is the explicit deferral the user signed off on.
- **No multi-property roll-up across PGs.** V1 single-property OR multi-property-selected-from-dashboard. True cross-property aggregator (sum cash flow across N properties with reconciliation) is V2 — depends on new aggregator design.
- **No per-property comparison view.** Priya's "which property is cash-positive?" question lives in V2 (depends on roll-up).
- **No GST treatment toggle.** V1 follows the corrected math (GST excluded from operating cash flow as it should be). V2 may add a toggle for CAs who want different lens (cash-basis incl. GST vs ex. GST).
- **No multi-FY comparison.** V1 = current FY only. V2 may add FY-over-FY comparison if operators ask.
- **No cash forecast (next 30 days).** Forward-looking. Different mental model (forecast vs historical). V2 or never.
- **No "Send to CA" email integration.** Operator uses existing menu (⋮) → email Excel path. V2 may add CA-email integration.
- **No notifications, no push, no badges, no popups.** The operator opens this screen at month-end or mid-month.

If a feature being discussed mid-cycle would require a notification, popup, forecast, or mutation surface inside DA-07, cut it.

---

## Traps & risks

Decided in advance so engineering doesn't wrestle:

- **USER:** **Hero formula no-ship reconciliation gate — new aggregator is canonical; legacy Excel is known-wrong.** The legacy `generateCashFlowReport.ts` at `src/services/reports/generateCashFlowReport.ts:393` has the deposit-refund double-count (HB-B1: `totalExpenses = expenseTotal + refunds` — both include deposit-refunds, ₹20K becomes ₹40K). At lines 427-428 there's a NEW GST sign bug surfaced in Phase 2 grounding (GST is added to gross profit when it should subtract — header label contradicts column label). **V1 builds the new DA-07 aggregator with both bugs fixed.** **Pre-V1 QA gate (no-ship blocker): 10-property reconciliation test on the new aggregator.** Hero must match: (a) `Collections Net Total (DA-02 widget, period)` for inflow portion; (b) `Expenses Net Total EXCLUDING Deposit-class (DA-04 widget filtered, period)` for outflow portion; (c) `Refunds Total WHERE invoice.due_type NOT IN (Security Deposit, Caution Money) (DA-03 service, period)` for non-deposit-refund subtraction. **Disagreement with the legacy Excel is expected and is the right behavior** — surface it in a footnote on the screen: *"This number uses corrected math. Legacy Excel report (Email from menu) still uses older math; V2 fixes Excel."*

- **USER:** **Section B Deposit Money In/Out — separation discipline matters more than the math.** Deposits are tenant money the operator must return. Mixing them into operating cash flow inflates "I made ₹X" and leads to over-deployment. **Hard rule:** Section B totals NEVER fold into Q1 hero. Section B is a parallel, non-revenue tracker. Designer should NOT show a "Total cash movement" summing both sections — that would defeat the whole separation. Operator education in the section header: *"Deposit money is tenant money you'll return — tracked separately from your actual cash flow."*

- **USER:** **Permission gate composite — `viewInvoices` AND `viewExpenses` (BOTH required) + tab-hide UX. Tradeoff confirmed: accountant-types with invoice-only access are locked out, by design.** No `viewCashFlow` JWT key exists today (verified Phase 2). Composite gate is feasible — both keys exist + are populated from `team_member_property.view_invoices` / `view_expenses` per `src/v1/login/property/service.ts:86-87`. **UX of the gate:** operators with EITHER key but not BOTH see DA-07 tab HIDDEN from navigation entirely — not "empty state," not "no permission" error. **Acknowledged tradeoff:** the accountant persona (owner-delegated, often invoice-only access) loses DA-07 discoverability. Operators with that gap will need to ask owners to grant the second permission (`viewExpenses`) to see DA-07. Rationale: DA-07 is a composite view; partial visibility (e.g., expense-only operator seeing only the outflow side) = misleading number that an accountant would mis-trust more than the lockout. **V1.1 may revisit** with a separate `viewCashFlow` JWT key (would let owners grant DA-07 specifically without exposing raw expense list) if accountant-lockout surfaces in support tickets >5×/week post-launch.

- **USER:** **DA-07 → DA-06 drill is always live-snapshot; period-pill disabled with explanation.** DA-06 V1 has no `liability_snapshots` table (verified Phase 2) — historical mode not feasible. When operator on DA-07 ("Last Month" period) taps *"View deposit balance →"* and lands on DA-06, the period-pill is DISABLED with tooltip: *"Deposit balance is always current — historical balances available in V2."* **Operator's job-to-be-done at this drill** is *"do I have enough cash to cover refunds-in-flight?"* — which is a live question, not a period question. The drill mismatch is by design, not a bug; tooltip carries the explanation.

- **USER:** **Mode 211/288 paper-transfer exclusion — already correctly handled by Collections + Expenses aggregators; DA-07 must not re-introduce it.** Phase 2 confirmed: `CollectionListHelper.getConsolidatedWidgetItems` at `src/v1/list_screens/collections/service.ts:179` excludes modes 211+288 by default (line 623 of helpers.ts: `return [211, 288]`). Expense aggregator at `src/v1/list_screens/expenses/service.ts:112-133` already segregates Deposit-class. **DA-07 V1 reuses these aggregators directly; no parallel mode-exclusion logic needed.** Trap is: if a new contributor builds a fresh SQL query bypassing the helpers, mode 211/288 will re-leak in. Code review checkpoint.

- **USER:** **Cross-screen drill destinations have auth gaps for some targets — no-ship blockers per the relevant sibling Brief.** Phase 2 auth audit:
  - DA-02 Collections list: HeaderValidator YES, reusable ✓
  - DA-04 Expenses list: HeaderValidator YES, reusable ✓
  - DA-03 Refunds list: NEW BUILD per DA-03 Brief (no list endpoint exists today)
  - DA-05 Discounts worklist: NEW BUILD per DA-05 Brief
  - DA-06 Liabilities: NEW BUILD per DA-06 Brief (with HB-D1 period-pill workaround)
  - DA-01 Dues list: HeaderValidator already present ✓
  
  **V1 sequencing decision: DA-07 ships LAST in the suite — after DA-03, DA-05, DA-06 V1s land.** This guarantees every drill works on day-1. The alternative (DA-07 in parallel) risks drills landing on "screen not yet built" placeholders.

  **Degraded-launch ladder (per-dependency contingency, in case a sibling slips and DA-07 has shipped or is days from shipping):**
  - **If DA-06 Liabilities slips:** DA-07 can ship with the *"View deposit balance →"* drill DISABLED (greyed CTA + tooltip *"Deposit balance view coming with DA-06 next cycle"*). Section B totals still work; only the cross-screen drill is degraded. **Sacred elements preserved:** Q1 hero + Section A + Section B numbers.
  - **If DA-05 Discounts worklist slips:** DA-07's "Discounts redeemed" line can drill to DA-02 Collections filtered to "Has discount" chip (already shipped per DA-05 v0.4.1). Slight degradation (operator sees collections with discounts, not the discount worklist itself) but the drill works.
  - **If DA-03 Refunds list slips:** DA-07's "Non-deposit refunds" line falls back to existing Tenant Passbook (refund tab focused) per the existing legacy drill pattern. Less rich than the planned DA-03 worklist but functional.
  - **If DA-06 Liabilities AND DA-05 worklist BOTH slip:** DA-07 still ships — only the live-deposit-balance drill is degraded; everything else has a degraded-mode path.
  - **Sacred (cannot degrade):** the Q1 hero number itself. If the new aggregator math isn't correct, DA-07 doesn't ship — see the pre-V1 QA gate.

- **USER:** **Permission gate's middle-state operators — what they see.** Operators with neither `viewInvoices` nor `viewExpenses` already see no relevant screens; no change. Operators with both see DA-07 normally. **Edge case:** operators who had `viewInvoices` but lost it (or vice versa) mid-cycle — DA-07 hides immediately. Designer must NOT show a "you no longer have access" interstitial — silent hide matches the rest of the navigation discipline.

- **TEAM:** **All DA-07 own endpoints NEW BUILD.** `POST /v1/list_screens/cashflow/list/filters`, widget endpoint, `filter_options`, `cashflow/advanced-details`. Mirror `src/v1/list_screens/expenses/` pattern. Filter codes from `1800-1899` range (free per Field Map — codebase-wide grep confirms only `q_1201`–`q_1219` exist + new ranges allocated for DA-04 1300s, DA-06 1700s). User-side consequence: until base widget endpoint lands, the screen has no data.

- **TEAM:** **Legacy `generateCashFlowReport.ts` — GST sign bug fixed in V1, HB-B1 double-count stays until V2.** GST sign fix is 2-line code change at lines 427-428 (flip sign + correct header label); cron + scheduled email path at `src/services/reports/reportScheduler.ts:1498` continues to run, just with corrected math (low regression risk — same code path, corrected arithmetic). HB-B1 double-count fix requires bigger refactor (Refund/Expense dedup logic on the SQL query side) — high regression risk on the cron path; deferred to V2 migration. V1 builds NEW DA-07 aggregator in `src/v1/list_screens/cashflow/service.ts` as parallel correct implementation; V2 migrates legacy to call the new aggregator (one source of truth). **Side-effect:** legacy Excel after V1 has correct GST but still has HB-B1 double-count; DA-07 hero has both correct. Footnote on DA-07 (per USER trap above) discloses precisely which bugs are fixed where.

- **TEAM:** **NEW chip on DA-02 Collections list — "Has refund applied" — MUST-SHIP, not cuttable.** Parallel to DA-05's "Has discount" chip pattern. Backend: SQL JOIN via `payments_invoices` → `invoices` → `refunds` where refund-row exists for the invoice within the period. Filter code from DA-02's range (next free after `q_1219`, after the 2 DA-05 chips land). **Operator-side cost of cutting** = no way to drill from DA-07's "Non-deposit refunds" line into the underlying collections that got refunded; that DA-07 line becomes a number with no actionable path. Same logic as DA-05 v0.3 + DA-06 v0.2 chip decisions. **Chip-slip fallback (if chip backend slips past Day-3 cut-line):** DA-07's "Non-deposit refunds" line falls back to drilling into DA-03 Refunds list filtered to `invoice.due_type NOT IN ('Security Deposit','Caution Money')` + period. Less rich than the chip-on-DA-02 drill (operator sees refunds without seeing the parent collection), but the drill still works.

- **USER:** **No "Total cash movement" line — strict separation between Section A (operating) and Section B (deposits).** PM recommendation (with operator-testing belt-and-suspenders): show Q1 hero as the headline number; show Section B subtotals separately; NEVER show a sum of A + B. Operator-mental-model bet: clearer conceptual separation > friendly-with-caveat aggregate number. Three reasons: (1) the hero IS the number — adding anything to it implies the operator made more; (2) operators don't read greyed caveats (consistent finding across DA-02/05/06 testing — headlines are absorbed, qualifiers are background); (3) most operators don't compute totals themselves — they look at the hero and move on; designing the screen against the 10-15% who reach for a calculator hurts the 85-90% majority. **Mitigation:** Section B header copy is direct — *"Deposit money is tenant money you'll return — tracked separately from your actual cash flow."* **Belt-and-suspenders:** if 3-operator pre-launch testing shows operators consistently do the wrong mental math (add A + B as if it's profit), V1.1 adds the greyed total line. V1 doesn't pre-emptively defend against the hypothetical bad calculation.

- **USER:** **Methodology disclosure for CA-readability — drill-down on the hero number.** CA-canonicity claim ("DA-07 is canonical, legacy Excel uses older math on HB-B1") is fragile if CA can't see DA-07's work. **Information need:** operator can drill into the Q1 hero to see the formula breakdown (inflows by source + outflows by category + exclusions like Mode 211/288 + discount reduction + refund adjustment + GST treatment). Designer picks the surface (bottom sheet / inline expansion / methodology icon) — the requirement is that CA looking at the operator's phone screen can trace the hero number back to its components, not just see a single hero with footnote. Without this, CA defaults to trusting the buggy Excel because Excel shows columns.

- **TEAM:** **Indian FY helper extraction.** Current pattern (Indian FY = April 1 – March 31) is inlined in 4+ places — `src/v1/list_screens/collections/helpers.ts:204`, `dues/helpers.ts:120`, `helpers/payments.ts:2186`, `helpers/invoices.ts:1393`. Drift risk: if FY logic ever changes (it won't, but drift risk is real), 4 places to update. **V1 task:** extract to shared `getCurrentFY()` helper in `src/utils/` as part of DA-07's YTD strip work. Low cost; consolidation hygiene win.

- **TEAM:** **Passbook double-debit propagation — V2, not V1 (explicit operator-confirmed deferral).** When operators record a single deposit return as both Refund row + Deposit-Refund Expense row (workflow some operators do today), `TeamPassbookRefundService` + `TeamPassbookExpenseService` each create independent debit entries. Passbook shows ₹40K debits for ₹20K cash event. **DA-07 V1 fixes only the aggregator math** (the screen shows the correct ₹20K outflow). The underlying passbook double-debit stays — V2 fixes the two factory services with dedup logic. **User-side consequence for V1:** operators who check both passbook AND DA-07 in the same session may notice the mismatch — and this can happen for ANY operator who has logged a deposit return as both Refund + Expense, not just on Section B. **Mitigation (information needs — designer picks surface):** screen-level disclosure (footer or methodology bottom sheet, NOT just on Section B) explaining *"If you log deposit returns as both a Refund and an Expense, your passbook may show duplicate entries. DA-07 shows the correct cash impact; passbook fix coming in next version."* Surface choice matters — footer-level keeps the message away from misleading any single section, but high-visibility enough that operators checking passbook notice it.

- **TEAM:** **HB-B1 dedup logic in the new aggregator.** Even though "operators rarely double-record" is the gather decision, DA-07's aggregator should still defensively NOT double-count. Implementation: outflows = `(operating expenses EXCLUDING expense_type ILIKE 'Deposit%') + (refunds WHERE invoice.due_type NOT IN ('Security Deposit', 'Caution Money'))`. This dedups by source-of-record: Deposit-class expenses are NEVER counted (they're tracked in Section B), and refunds are split by what they refund (deposit refunds → Section B, non-deposit refunds → Section A operating outflow). No need for cross-table dedup; the schema-level partition does the work.

- **TEAM:** **Excel CTA gating uses existing `team_member_property.cashflow_report` column.** Verified Phase 2 at `entities/teamMemberProperty.ts:191`, default false, already used to gate the legacy Excel report. **V1 reuses it for the DA-07 Excel CTA** — no new column needed. Operators without this DB flag see the screen but not the Excel-export button.

- **USER:** **Future date ranges show ₹0 (period-sensitive logic).** Cash flow can't be future-dated (no payments / expenses recorded in the future). Helper text: *"Cash flow shows historical periods. Pick a past period to see data."*

- **USER:** **Empty + new-account states.** New properties (<30 days old) with no transactions show *"No payments or expenses recorded yet."* instead of ₹0 in a vacuum. MoM chip hides when there's no last-month baseline.

- **USER:** **YTD strip (Q6) silently inherits the new aggregator's GST + HB-B1 fixes — operator may notice disagreement with prior year-end Excel.** When operator runs Q6 YTD on DA-07 and compares to last year's CA-prepared annual summary (built from old buggy Excel), the numbers will differ (probably by 5-15% on properties with significant deposit-refund + GST activity). Methodology drill (see USER trap above) helps CA reconcile, but expect a wave of "why is my YTD different?" questions in the first quarter post-launch. Pre-launch operator-testing on Q6: 3-operator test with last-FY data to confirm operators understand the diff is correction, not regression.

- **USER:** **Default period choice — "This Month" vs "Last Month" before day 25.** Q8 (mid-month nudge) fires when default is active for half the month — internally tense ("default to this month then warn that this month is misleading"). **V1 keeps "This Month" as default for cross-suite consistency** (every DA screen defaults to This Month). V1.1 may consider dynamic default ("Last Month" before day 25, "This Month" after day 25) if Q8 nudge feels noisy in operator testing.

**The four risks:**

| Risk | Read | Mitigation |
|------|------|------------|
| Will operators use it? | HIGH yes | Four named pains + clear month-end + mid-month rhythm. The CA-readability angle (operators forward screenshots to their CA) is unique to DA-07 in the suite. Excel report exists today but is bug-known + brittle; DA-07 is the trusted replacement |
| Will they understand it? | MEDIUM | "Operating Cash Flow" / "Deposit Money In/Out (separately)" / "Net" need plain explanations. The Section A vs Section B separation is the conceptual cornerstone; if operators conflate them, the screen fails its main job. Operator-Hindi labels. Tamil / Telugu / Kannada test. CA-readability test with 2 small-firm CAs |
| Can we build it in 2 weeks? | MEDIUM | DA-07's own endpoints NEW BUILD + new aggregator (correct HB-B1 + GST fix) + 1 new chip on DA-02 + FY helper extraction + 3 conditional callout queries + permission composite gating. Inflow/outflow aggregators ALREADY EXIST (reuse) — that's the big cost-saver. Tight but tractable. **Day-7 mid-cycle go/no-go:** if hero + Section A + Section B + chip aren't behind a frontend-callable contract by Day 7, cut Q8 (mid-month nudge) and Q7 (callouts) per the operator-pain-ranked cut order — preserving Q5 + Q6 as last-to-cut |
| Is it worth it? | STRONG | DA-07 is the screen the entire suite is converging toward. Without it, the operator does the math in their head; with it, they have one trusted number to plan against. Every ₹500-2000/property/month in CA fees just for monthly cash-flow reconciliation is the existing-state-cost the screen displaces |

**When to stop and reconsider** (sequenced day-by-day, not all-end-of-week):
- **Day 1 (kickoff):** designer + eng align on the period chip + "Live · period" pill semantics for Section B drill into DA-06 (live snapshot vs period-pill-disabled). Defer past Day 1 risks the UX surfacing as open question Day-5.
- **Day 3:** if filter code allocation for the new "Has refund" chip on DA-02 isn't done → escalate to Jatin (Eng). Chip is MUST-SHIP for V1; chip-slip fallback (drill to DA-03 instead) only kicks in past Day 5.
- **Day 3:** if the new aggregator's HB-B1 dedup logic + GST sign fix design isn't agreed → escalate to Jatin (Eng) + Sanchay (PM). Do not start Section A / Section B breakdown work until aggregator math is decided; they'll have to be rebuilt otherwise.
- **Day 3:** if GST sign fix in legacy `generateCashFlowReport.ts` (lines 427-428) isn't scheduled → escalate to Jatin (Eng). 2-line change but needs to land in V1 — operators forwarding wrong Excel to CAs is an active trust problem.
- **Day 5:** if the FY helper extraction (`getCurrentFY()`) isn't merged → escalate to Jatin (Eng). YTD strip (Q6) depends on it.
- **Day 7 (mid-cycle gut check):** if hero + Section A + Section B + chip aren't behind a frontend-callable contract, **cut Q8 (mid-month nudge) and Q7 (callouts) per the operator-pain-ranked cut order — preserve Q5/Q6** (Q5/Q6 cut LAST per the rank because Q5 is multi-property niche cut and Q6 is YTD which operator can self-trigger via period chip). MUST-SHIP base (Q1+Q2+Q3+Q4+chip) is sacred; stretch tier flexes. Owner: Sanchay (PM) calls the cut.
- **End of week 1:** if DA-03 Refunds list endpoint OR DA-05 Discounts worklist OR DA-06 Liabilities screen hasn't shipped → DA-07 launch delays accordingly. DA-07 is sequenced LAST; sibling slip = DA-07 slip.
- **Post-launch:** if support tickets about "DA-07 number doesn't match Excel report" exceed 10/week, the footnote disclosure isn't landing. Strengthen the in-screen explanation or move legacy Excel fix earlier than V2.

---

## V2 scope (deferred from V1 — codebase-grounded gap)

V0.1 already scopes V1 lean (no trend, no per-property comparison, no GST toggle, no forecast, no CA-email, no passbook fix, no multi-FY). **The Phase 2 LAYER 2 exploration surfaced additional dependencies** the Brief explicitly defers to V2. See `[[DA-07 Codebase Exploration]]` for full grounding.

**Top V2 questions (operator-pain-ranked, with owner + candidate cycle):**

| # | Question | Owner | Candidate cycle |
|---|----------|-------|-----------------|
| 1 | **"Show me cash flow trend over the last 6 months."** No `cashflow_snapshot` table exists (verified). V2 builds it + nightly cron + backfill alongside DA-06's `liability_snapshots` (natural pairing — same infrastructure). | Jatin (Eng) + Sanchay (PM) | V2 cycle paired with DA-06 trend chart |
| 2 | **"Per-property comparison view for Priya."** Side-by-side cash flow per property. Requires multi-property aggregator. | Sanchay (PM) | V2 |
| 3 | **"Migrate legacy Excel report to use new aggregator."** Single source of truth — DA-07 hero + Excel report = same number. Touches cron + scheduled emails. | Jatin (Eng) | V2, after DA-07 V1 validates new aggregator |
| 4 | **"Fix passbook double-debit at the factory services."** `TeamPassbookRefundService` + `TeamPassbookExpenseService` get dedup logic. Prevents NEW double-debits + optional retroactive cleanup of historical bad entries. | Jatin (Eng) | V2 cycle (target: within 1 cycle after DA-07 V1 launch) — explicit operator-confirmed deferral; cycle locked to prevent perpetual deferral |
| 5 | **"GST treatment toggle."** Cash-basis incl. GST vs ex. GST lens for CAs. | Sanchay (PM) | V2 if operator demand validates |
| 6 | **"Multi-FY comparison (FY-over-FY)."** Annual comparison view. | Sanchay (PM) | V2 |
| 7 | **"Cash forecast (next 30 days)."** Predictive view based on collection patterns + recurring expenses. Different mental model. | Sanchay (PM) | V2 or never (validate demand) |
| 8 | **"Send to CA email integration."** Direct CA delivery from DA-07. | Jatin (Eng) | V2 |

**V1 → V2 architecture decisions (close before V2 build):**

- **Schema migration: `cashflow_snapshot` table** — daily per-property record of net cash flow + composition. Powers trend chart + future trajectory callouts. Paired with DA-06's `liability_snapshots`.
- **Multi-property aggregator service** — `CashFlowService.getMultiPropertyAggregate()`. Reused by per-property comparison (Q2) + multi-property dashboard hero.
- **Legacy Excel report migration** — `generateCashFlowReport.ts` rewritten to call new aggregator. Touches scheduled-email cron at `reportScheduler.ts:1498`.
- **Passbook factory dedup** — `TeamPassbookRefundService` + `TeamPassbookExpenseService` get shared dedup logic at write-time. Optional one-time backfill cleanup of historical double-debits.
- **GST handling pattern** — cash-basis incl./ex. GST decided at aggregator level + surfaced as user toggle.

See `[[DA-07 Codebase Exploration]]` for the full LAYER 1 + LAYER 2 dependency map and detailed architecture risks.

---

## Footer

**Things to test with operators** (don't block kickoff; settle before launch):
- 3-operator check on the Hero language — does *"Operating Cash Flow"* land, or does plainer *"Aapka net paisa is mahine"* land better? Test with 3 owners (1 solo, 1 multi-property, 1 CA-using).
- 3-operator check on Section B separation — does the "deposit money is not your money" framing land, or do operators still mentally add it to Q1 hero? High-stakes test; if it fails, V1's central conceptual move fails.
- 2-CA check on the CA-readability — share a DA-07 screenshot with 2 small-firm CAs. Do they recognize "Net Operating Cash Flow" subtitle as GAAP-shaped? Can they reconcile the hero against a balance-sheet exercise?
- Bharat-language testing — Tamil / Telugu / Kannada. Owner: Sanchay (PM); target = week 2.
- Mid-month timing nudge wording — does the "wait for day 25" framing land as helpful or condescending? 2-operator check, week 2.

**Cross-Brief propagation (DA-07 owns the cascade):**
- **DA-02 Brief / Build Sheet** — add 1 new chip ("Has refund applied") + backend filter param + held-amount column equivalent (none needed — refund presence is the data). **DA-02 Brief footnote should explicitly reference DA-07 as the chip's primary consumer** so DA-02 owner doesn't deprioritize the chip as a "nice-to-have." Owner: Sanchay (PM) for chip-naming + propagation note; Jatin (Eng) for chip + filter param. Pattern matches DA-05 v0.3 (chips on DA-02) + DA-06 v0.2.2 (chips on tenant lists). Expect DA-02 to bump to v0.7.5 (or similar) once chip lands.
- **DA-03 / DA-05 / DA-06 Briefs** — no scope changes from DA-07 V1; DA-07 depends on their V1s landing first (sequencing), not on changes to their scope.
- **DA-01 / DA-04 Briefs** — no scope changes; DA-07 reuses existing endpoints.
- **Legacy `generateCashFlowReport.ts`** — NOT touched in V1. V2 migrates. Footnote on DA-07 explicitly discloses the known-disagreement.
- **Pattern note:** DA-07 is the meta-aggregator + the last screen in the suite. Cross-Brief impact is light (only 1 new chip on DA-02) because most sequencing dependencies are on sibling V1s landing, not on changes to their scope.

**Related docs:**
- Engineering spec (post-design): `[[DA-07 Build Sheet]]`
- Legacy PRD (historical): `[[DA-07 Cash Flow Detailed Analytics]]`
- Codebase ground truth: `[[_Ground Truth Field Map]]`
- Phase 2 grounding: `[[DA-07 Codebase Exploration]]` (LAYER 1 + LAYER 2 findings — feeds V1 traps + V2 scope)
- Sibling Briefs: `[[DA-01 Brief]]` · `[[DA-02 Brief]]` · `[[DA-03 Brief]]` · `[[DA-04 Brief]]` · `[[DA-05 Brief]]` · `[[DA-06 Brief]]`
- Design system + Figma: designer to lock the frame by end of week 1 (frame ID added to this row on lock)
- Cross-suite engineering blockers: `[[DA-07 Cash Flow Detailed Analytics#Cross-Suite Engineering Blockers]]`

**Key decisions locked:**
- Hero formula: NEW correct aggregator (`(Collections inflow − discounts redeemed) − (operating expenses EXCLUDING Deposit-class − non-deposit refunds)`, mode 211/288 already excluded by source aggregators). Legacy Excel report stays buggy in V1; explicit footnote discloses disagreement.
- Period-sensitive screen (NOT live-snapshot like DA-06). Time chip drives everything. Default period at app open: This Month.
- Section A (Operating Cash Flow) and Section B (Deposit Money In/Out) are SEPARATE — Section B never folds into Q1 hero.
- MoM chip: 3-state (Improvement / Regression / Sign-reversal-to-negative). Collapsed from 4 in PRD v1.2.
- Permission gate: composite `viewInvoices AND viewExpenses` (BOTH required). Tab HIDDEN for operators missing either. No new JWT key built.
- DA-07 → DA-06 drill: always live snapshot, period-pill DISABLED with tooltip explanation (HB-D1 resolution; DA-06 V1 has no historical mode).
- Drill chain (rule, not UI prescription): every Section A inflow row → DA-02 Collections filtered to period + composition; every Section A outflow row → DA-04 Expenses filtered to period + category; "Discounts redeemed" line → DA-02 with "Has discount" chip + period; "Non-deposit refunds" line → DA-02 with NEW "Has refund" chip + period (chip is V1 MUST-SHIP); Section B Deposit In → DA-02 filtered to deposit due_types; Section B Deposit Out → DA-03 filtered to deposit due_types; "View deposit balance →" → DA-06 (live snapshot, period-pill disabled); vendor concentration callout → DA-04 filtered to top `paid_to`; capex callout → DA-04 filtered to Other or `is_capex`; dues bridge → DA-01 Dues filtered to overdue + period.
- New chip on DA-02 Collections list: *"Has refund applied"* — MUST-SHIP for V1 (drill from DA-07 "Non-deposit refunds" needs it). Filter code from DA-02 range. Backend SQL JOIN via Payments → Invoices → Refunds.
- DA-07 sequencing: ships LAST in the suite, after DA-03 + DA-05 + DA-06 V1s land. Drills work day-1; no broken-drill fallbacks.
- Legacy Excel report: NOT touched in V1. V2 migrates to use new aggregator.
- Passbook double-debit fix: V2 (explicit deferral). Some operators do double-record; DA-07 V1's correct aggregator math fixes the screen but not the underlying passbook. Operator-facing tooltip on Section B discloses.
- Indian FY helper: extract `getCurrentFY()` to `src/utils/` in V1 (currently inlined in 4+ places).
- Excel CTA: gated by existing `team_member_property.cashflow_report` column (reused, no new column).
- ⓘ icon: single-tap → bottom sheet. No tooltip, no long-press (per DA-01/02/03/04/05/06 convention).
- V1 must-ship: Q1 (hero) + Q1 methodology drill (formula breakdown surface — CA-trust handoff) + Q2 (Section A) + Q3 (Section B) + Q4 (MoM chip) + new "Has refund" chip on DA-02. Protect at stretch: Q5 (per-bed), Q6 (YTD), Q7 (callouts), Q8 (mid-month nudge). Cut order if time tightens (operator-pain-ranked): **Q8 → Q7 → Q6 → Q5** (Q5 cuts LAST among stretch — multi-property only). **Day-7 mid-cycle gate cuts top 2 from this order (Q8 + Q7) if hero/Section A/Section B/chip aren't behind a frontend-callable contract.**

**Changelog:**

| Date | Version | Change |
|------|---------|--------|
| 2026-05-18 | v0.2.2 | **Kickoff polish — 2 nice-to-haves from independent verification sub-agent.** v0.2.1 was approved-to-proceed; these are quality polish. (1) **GST legacy fix scope refined** — original wording said "2-line code change (flip sign + correct math)"; verification agent caught that L428's Net Profit line ALSO drops `adjustedFromAdvance` silently, so the fix is actually "flip sign at L427 + align L428 math with header label (subtract GST + restore adjustedFromAdvance subtraction)." Same 2-line scope but more precise eng instructions. (2) **Methodology drill (Q1 hero formula breakdown surface) elevated from Traps-only to Q1 must-ship description + KDL must-ship line.** Verification's pre-mortem: most-likely month-1 failure = CA receiving forwarded screenshot can't trace DA-07 number to components, defaults to trusting legacy Excel (which shows columns). Methodology drill is the CA-trust handoff protection — too important to leave buried in Traps. Now: Q1 description names it as MUST-SHIP; KDL must-ship line includes it; Trap continues to describe information need. |
| 2026-05-18 | v0.2.1 | **CPO final-read — single stale wording fix.** Line 89 (must-ship reconciliation gate) still said GST fix is *"NOT at the legacy report"* — stale from v0.1 before v0.2 changed the decision to fix GST in legacy too. Rewrote gate to precisely reflect v0.2 state: DA-07 + legacy Excel agree on GST treatment, disagree on deposit-refund treatment (off by deposit-refund amount) until V2 legacy migration. No other contradictions caught in CPO sweep. Cut-order consistent across 4 references (body + risk-table + Day-7 + KDL). Chip MUST-SHIP framing consistent across 6 references. Hero formula consistent across body + traps + KDL. Zero AI-slop, hedges, or UI-words. Body 7000 words / 10 sections — over targets but consistent with DA-06 (meta-aggregator scale, dense decisions, full changelog). |
| 2026-05-18 | v0.2 | **Critique pass — substantive blockers + cross-Brief + operator-lens completeness.** Sub-agent critique returned 3 blockers + 6 high + 5 medium. **3 BLOCKERS resolved with operator confirmation:** (1) **Permission gate composite AND tradeoff confirmed + acknowledged in Trap** — accountant persona (invoice-only access) locked out by design; V1.1 may revisit with separate `viewCashFlow` JWT key if accountant-lockout surfaces in >5 support tickets/week post-launch. Rationale: partial visibility for composite view = misleading number; lockout is the safer default. (2) **GST sign bug — fix in legacy AND new aggregator in V1** (2-line code change at `generateCashFlowReport.ts:427-428`, flip sign + correct header label). HB-B1 double-count fix stays in new aggregator only (legacy refactor too risky for V1 cron path). Operators stop forwarding wrong-CA-number Excel immediately; legacy still has HB-B1 until V2 migration. (3) **Degraded-launch ladder added** — per-dependency contingency if DA-03/DA-05/DA-06 V1 slips: DA-07 can still ship with degraded drills (DA-06 slip = greyed "View deposit balance" CTA; DA-05 slip = drill to DA-02 "Has discount" chip; DA-03 slip = drill to Tenant Passbook refund tab). Sacred: Q1 hero math correctness — DA-07 doesn't ship if aggregator math isn't right. **6 HIGH fixes applied:** (4) Day-7 gate language tightened — explicit "cut Q8 + Q7, preserve Q5/Q6" + owner named (Sanchay). (5) Chip-slip fallback specified — "Has refund" chip slip = drill to DA-03 Refunds list filtered. (6) Section A vs B separation — PM recommendation for strict no-total with reasoning baked in (3 reasons: hero IS the number; operators don't read greyed caveats; majority don't compute totals); belt-and-suspenders 3-operator pre-launch test; V1.1 adds greyed total only if testing shows operators do the wrong mental math. (7) Methodology disclosure — information need added for CA-readability (drill into Q1 hero to see formula breakdown); designer picks surface. (8) Passbook double-debit disclosure widened from "Section B tooltip only" to screen-level disclosure (footer or methodology bottom sheet) — the divergence affects any operator who double-records, not just Section B viewers. (9) Day-3 escalation owners named (Jatin for filter codes + aggregator math + GST legacy fix + FY helper; Sanchay co-owner on aggregator math). **5 MEDIUM applied:** (10) YTD strip GST inheritance flagged (operators may notice diff vs prior year-end Excel). (11) Default period "This Month" decision kept for cross-suite consistency, V1.1 dynamic-default revisit option flagged. (12) Empty-state copy softened ("No payments or expenses recorded yet" vs "operate the property"). (13) V2 question #4 (passbook fix) cycle locked ("V2 cycle, target within 1 cycle after DA-07 V1") to prevent perpetual deferral. (14) Cross-Brief footnote — DA-02 explicitly told DA-07 is primary consumer of the new chip (prevents deprioritization). |
| 2026-05-18 | v0.1 | **Initial Brief** — written using `/brief` skill with Phase 2 LAYER 1 + LAYER 2 discipline from kickoff (per 2026-05-16 learning) + 4 operator-lens questions framework on all new V1 scope (per 2026-05-18 learning). **Phase 2 caught:** (1) HB-B1 confirmed at exact line — `generateCashFlowReport.ts:393` `totalExpenses = expenseTotal + refunds` — real reproducible double-count when operator logs deposit-return as both Refund + Expense rows. (2) **NEW bug not in Build Sheet HB list: GST sign error** at lines 427-428 (GST added to gross profit instead of subtracted; header label contradicts column label). (3) **Passbook double-debit propagation** — `TeamPassbookRefundService` + `TeamPassbookExpenseService` independent factories; when operator double-records, both create debits → passbook silently wrong by 2× per event. (4) **Collections + Expenses aggregators already exclude mode 211/288** — DA-07 reuses, no parallel mode-exclusion logic needed. (5) **DA-03 + DA-05 + DA-06 drill destinations are NEW BUILD** per sibling Briefs — DA-07 has sequencing dependencies; ships LAST. (6) **HB-D1 resolved** — DA-06 V1 has no historical mode; DA-07 → DA-06 drill = always live snapshot with period-pill disabled + tooltip. (7) **Indian FY helper inlined in 4+ places** — extract to shared `src/utils/getCurrentFY()` as part of DA-07. (8) **`team_member_property.cashflow_report` column already exists** for Excel CTA gating — reused, no new column. **All gather decisions explicit operator-confirmed:** hero formula = DA-07 builds new correct aggregator, legacy Excel stays buggy until V2 (with footnote disclosure); trend chart deferred to V2 (matches DA-06 — no `cashflow_snapshot` table); DA-07 ships LAST in sequencing; permission = composite `viewInvoices AND viewExpenses` (BOTH required), no new JWT key; drill map covers DA-01/02/03/04/05/06 (every line item routes); passbook double-debit deferred to V2 with explicit "some operators do double-record" acknowledgment. **Cross-Brief impact:** 1 new chip on DA-02 ("Has refund applied"), MUST-SHIP, parallel to DA-05's "Has discount" pattern + DA-06's chips-on-tenant-lists pattern. Traps labeled USER: / TEAM: from line 1. V2 scope pointer with 8 deferred questions + 5 V1→V2 architecture decisions. |
