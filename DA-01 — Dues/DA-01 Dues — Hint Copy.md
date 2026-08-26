# DA-01 Dues, Hint Copy

**What is in here:** the words an operator sees when she taps (i) on the Dues tab. All 9 blocks, all 25 numbers. For each one: what it means, how it is worked out, and what the date chips at the top do to it. Plus what today's shipped sentence says and why it changes.

**Who reads this:** engineering, because each row replaces a line in `src/v1/analytics/dues/duesHints.ts`. QA, because every "why it changes" row is testable today. Design, because the two lines are all the bottom sheet holds. Sanchay, because the rulings are here.

**Where the truth came from:** the Dues handoff sheet, the build verification log findings F1 to F26 and rulings D1 to D10, and the code read directly at commit `aed58caa8`. No sentence here was written from another sentence.

**The short version.** Of the 25 numbers on this tab, **17 get new copy that says something the old copy did not**. The old copy is not badly written. It fails in two repeated ways: it never says who is left out, and it never says that six of the nine blocks completely ignore the date chips sitting right above them.

---

## Contents

1. How these sentences are written
2. The five time phrases
3. What the date chips actually do on this tab
4. Block 1, Overview
5. Block 2, Dues (Live)
6. Block 3, Bills Summary
7. Block 4, Dues Breakdown
8. Block 5, Overdue Breakup
9. Block 6, Upcoming Dues
10. Block 7, Deposit Dues
11. Block 8, Breakup by Stay Duration
12. Block 9, Dues by Property
13. Copy that flips when a known defect is fixed
14. What this pass did not cover

---

## 1. How these sentences are written

Nobody reads a glossary. An operator taps (i) in one situation: **a number surprised her.** It looks too big, too small, or it disagrees with a screen she just left. She is on a phone, mid-task, and English is not her first language.

So the copy has one job: **remove the surprise.** Not teach, not cover every edge, not defend the formula.

| Rule | Why |
|---|---|
| First line under 12 words, answering "what is this counting" | She reads one line and leaves |
| If a group of people or money is left out, say it in the same breath | This is the single biggest failure in today's copy. "All unpaid bills" while people who left are dropped reads clean and misleads |
| Tiny exclusions stay out of the copy | Bills under one rupee are excluded everywhere. That can never surprise her. It belongs in this doc, not in her sheet |
| One fixed time phrase, reused everywhere | She learns it once and recognises it forever. See section 2 |
| No word she would not say to a tenant | Banned here: net of, gross, excluded, adjusted, attribution, recurring setup, active tenant as a defined state, period |
| Her words stay | dues, bill, deposit, advance, booked, moved out, under notice, short stay |
| Never restate the value, never coach | The number is already on the tile |
| Where two blocks disagree on purpose, say why on the one that is different | Her real question is "why does this not match", and a definition alone never answers it |

**One call worth naming.** Some of these numbers are wrong today, not just described wrongly. Until the number is fixed, the copy describes **today's number honestly**, because she is reading today's screen. Section 13 lists every sentence that flips when its defect is fixed.

---

## 2. The five time phrases

Every number on every analytics tab is one of five kinds. Say it the same way every time.

| Kind | The phrase |
|---|---|
| Follows the chips | "The dates you pick at the top decide this." |
| Ignores the chips | "The dates you pick at the top do not change this." |
| Dated by the bill's due date | "Counted by the date the bill was due." |
| Has its own range | "Uses the months on the graph, not the dates at the top." |
| Forecast | "This is expected money. It is not billed yet." |

On Dues, every number that is dated at all is dated by the bill's due date, so the third phrase only appears where it could genuinely surprise. The fourth is not used on this tab.

**Which blocks are which.** Verified against the code, and matching finding F18.

| Block | Reads the date chips | Says so on its face today |
|---|---|---|
| Overview | **No** | Only the block title, "Overview (Live)" |
| Dues (Live) | **No** | Only the block title, "Dues (Live)" |
| Bills Summary | Yes | Nothing needed |
| Dues Breakdown | Yes | Its own dropdown |
| Overdue Breakup | **No** | **Nothing at all** |
| Upcoming Dues | **No** | Subtitle, "Forecast" |
| Deposit Dues | **No** | **Nothing at all** |
| Breakup by Stay Duration | Yes | Nothing needed |
| Dues by Property | Yes | Its own dropdown |

**Six of the nine blocks ignore the chips, and today not one of their 19 sentences says so.** That is the largest single fix in this document.

---

## 3. What the date chips actually do on this tab

The same chip does not mean the same thing across the row. This is worth knowing before reading any sentence below.

| Chip | The window it actually sets | Note |
|---|---|---|
| This Month | The 1st of this month **up to today**, not to month end | This is a part month. Every other "month" chip is a whole month |
| Last Month | The whole of last month, 1st to last day | Whole month |
| Current FY | 1 April **to 31 March**, so it includes bills not yet due | Should end at today. Ruled in D2, not built. Findings F1 and F9 |
| All Time | No start, no end | |
| Custom | Exactly the dates picked, **and the end may be in the future** | The safeguards that were meant to come with a forward window were never built. Finding F17 |

**The three Overview windows do add up.** All Past covers everything before this month, this month's card covers the 1st to today, All Future covers after today. Together they equal All Time. An operator who adds them up will get the right answer, which is worth protecting.

---

## 4. Block 1, Overview

**Six cards, and the block ignores the date chips completely.** Each card carries its own fixed window worked out from today. Every card counts only tenants living here or booked.

| Card | What is this? | How is this calculated? |
|---|---|---|
| **All Time Dues** | Everything unpaid, from any date. | Only tenants living here or booked. People who have moved out are not counted. The dates you pick at the top do not change this. |
| **All Past Dues** | Unpaid bills that were due before this month started. | Only tenants living here or booked. The dates you pick at the top do not change this. |
| **{Month}'s Due** | Unpaid bills due from the 1st of this month up to today. | Only tenants living here or booked. Bills due later this month are not counted here. The dates you pick at the top do not change this. |
| **All Future Dues** | Unpaid bills due after today. | Only tenants living here or booked. The dates you pick at the top do not change this. |
| **{Month}'s Projected Due** | Rent expected to be billed between tomorrow and the end of this month. | This is expected money. It is not billed yet. Counts tenants living here whose rent is added automatically each month. Bookings are not counted. |
| **Current FY Dues** | Unpaid bills due in this financial year, 1 April to 31 March. | Includes bills dated later this year that are not due yet. Only tenants living here or booked. The dates you pick at the top do not change this. |

**What changed and why**

| Card | Today it says | Why it changes |
|---|---|---|
| All Time Dues | "Total of all unpaid bills, across every date." | Two silences. Money owed by people who have moved out is not in it, and the date chips above do it nothing. Base pool at `helpers.ts:84`, `t.status IN (1,2)` |
| All Past Dues | "Unpaid bills that came due before this month." | Same two silences |
| {Month}'s Due | "Unpaid bills that came due this month, up to today." plus "Counts active tenants and confirmed bookings only." | The best sentence on the tab today, and the only one that names its population. Two changes: plain words for who is counted, and it now says the chips do nothing. The "bills due later this month are not counted" line is new, because that is the surprise when this card is read next to All Future Dues |
| All Future Dues | "Bills that come due after today." | Same two silences |
| {Month}'s Projected Due | "What the recurring setup is expected to raise before month end." plus "Forecast, not yet billed." | "Recurring setup" is our word, not hers. Bookings being left out is finding F2 and she cannot see it. Only monthly rent counts: tenants on any other billing frequency are absent, `duesService.ts:825` |
| Current FY Dues | "Unpaid bills due within the current financial year." | It runs to next 31 March, so it holds bills nobody owes yet, and the change chip beside it measures a different window entirely. Findings F1 and F9, ruled in D2. Until D2 is built, the sentence must admit the future bills |

---

## 5. Block 2, Dues (Live)

**Five numbers, and the block ignores the date chips completely.** The title says "Live"; the sentences never do.

| Card | What is this? | How is this calculated? |
|---|---|---|
| **Total Dues** | Everything unpaid right now, whatever the due date. | Only tenants living here or booked. The dates you pick at the top do not change this. |
| **Overdue** | Unpaid bills whose due date has already passed. | Only tenants living here or booked. The dates you pick at the top do not change this. |
| **Due Today** | Unpaid bills due today. | Only tenants living here or booked. The dates you pick at the top do not change this. |
| **Due This Week** | Unpaid bills due in the next 7 days, starting tomorrow. | Today's bills are counted in Due Today, not here. The dates you pick at the top do not change this. |
| **Due Later** | Unpaid bills due more than 7 days from now. | Only tenants living here or booked. The dates you pick at the top do not change this. |

**What changed and why**

| Card | Today it says | Why it changes |
|---|---|---|
| Total Dues | "All unpaid dues right now, regardless of due date. Always live." | "Always live" is our shorthand and she has to work out what it rules out. The plain phrase says it. The population silence is the same one as everywhere |
| Overdue, Due Today, Due Later | Each names its bucket correctly, and ends "Always live" on two of the four | Correct as far as they go. Same two changes: plain time phrase, and who is counted |
| Due This Week | "Unpaid dues falling due within the next 7 days." | The window starts tomorrow, not today, `duesService.ts:305`. Read next to Due Today, "within the next 7 days" sounds like it includes today, and she would think a bill is double counted |

---

## 6. Block 3, Bills Summary

**Two cards, and this block follows the date chips.**

| Card | What is this? | How is this calculated? |
|---|---|---|
| **Bill Due** | Every bill that came due in the dates you picked, paid or not. | Counted by the date the bill was due. Only tenants living here or booked. |
| **Received** | Money received against those same bills, whenever it came in. | A payment made later still counts here, as long as the bill was due in the dates you picked. Bills cleared from a tenant's own deposit or advance also count as received. |

**What changed and why**

| Card | Today it says | Why it changes |
|---|---|---|
| Bill Due | "Everything that came due in the selected period, paid or not." plus "Grouped by bill due date." | Nearly right. Only the population line is added |
| Received | "Collected against the bills that came due in the period." plus "Payments recorded against those same bills." | **Two real surprises, neither of them stated.** First, the payment itself carries no date condition: money that arrived months later still lands in this window. That is finding F8, and it was ruled correct in D4, so the copy must carry it. Second, a bill cleared out of the tenant's own deposit or advance has a paid amount and counts here as though money came in. `duesService.ts:340`, `SUM(COALESCE(i.paid_amount, 0))` |

**A note for whoever reads this beside the money model work.** The second point is the same thing the money model calls a transfer counted as a collection. The copy above describes today's number. When Received is split so that money moved from a tenant's own deposit stops counting as money received, the second line drops.

---

## 7. Block 4, Dues Breakdown

**Three tabs, and this block follows the date chips.** The three tabs deliberately do not add to the same total.

| Tab | What is this? | How is this calculated? |
|---|---|---|
| **Category** | Unpaid bills in the dates you picked, split by bill type. | Only tenants living here or booked. The largest few types are shown, the rest are grouped as Others. |
| **Tenant Status** | Unpaid bills split by where the tenant stands: living here, under notice, booked, or moved out. | This is the only place on this tab that counts people who have moved out, so its total is higher than the other two tabs. |
| **Added By** | Unpaid bills grouped by who added the bill. | Grouped by role, not by person: Owner, RentOk, Partner or Tenant. |

**What changed and why**

| Tab | Today it says | Why it changes |
|---|---|---|
| Category | "Unpaid dues in the period, split by bill category (rent, deposit, etc.)." | The population line is added, and the Others fold is stated because she can see a bar called Others and cannot tell what is in it |
| Tenant Status | "Unpaid dues split by tenant state, active, under notice, bookings, and old tenants." plus "The only breakdown that includes old (moved-out) tenants." | **The best sentence shipped anywhere in this layer, and the change is small.** The old version says this tab is different; the new one says what that does to the number she is looking at. That is finding F11 answered in her words instead of ours |
| Added By | "Unpaid dues grouped by who added the bill." plus "Role-level attribution (owner, RentOk, partner, tenant)." | "Role-level attribution" is our phrase. "Grouped by role, not by person" says the same thing and answers the question she actually has, which is why she cannot see a staff name |

**A defect this copy deliberately does not paper over.** Two creator codes have no name and are silently folded into RentOk's bar, about ₹4.85 crore of bills across roughly 500 properties. That is finding F26 and it is open. No honest sentence can be written about a bar whose contents are unknown, so the fix comes first and the copy stays as above.

---

## 8. Block 5, Overdue Breakup

**Two tabs, and this block ignores the date chips.** It also carries a dropdown with a single option, which does nothing.

| Tab | What is this? | How is this calculated? |
|---|---|---|
| **By Amount** | Overdue bills grouped by how late they are. | Counted from the bill's due date to today. The dates you pick at the top do not change this. |
| **By Category** | Overdue bills grouped by bill type. | Only bills whose due date has already passed. The dates you pick at the top do not change this. |

**What changed and why**

| Tab | Today it says | Why it changes |
|---|---|---|
| By Amount | "Overdue dues grouped by how long they are past the due date. Always live." plus "Days overdue = today minus due date." | The calculation line was written as arithmetic. She does not need the minus sign, she needs to know the clock runs to today. The plain time phrase replaces "Always live", which matters more here than anywhere: this block has **no** other sign that it ignores the chips |
| By Category | "Overdue dues grouped by bill category. Always live." | Same change, plus a line saying only past due bills are in it, because this tab sits beside Dues Breakdown's Category tab which counts everything |

**Two build defects worth carrying to whoever fixes this block.** The tab named "By Amount" splits by lateness, not by amount, and both tabs are amounts, so the name misleads (finding F15). And the bar labelled "90+ Days" actually starts at 91 days, because day 90 falls in the bucket below it.

---

## 9. Block 6, Upcoming Dues

**One number, forecast, and it ignores the date chips.**

| Card | What is this? | How is this calculated? |
|---|---|---|
| **Rent** | Rent expected between tomorrow and the end of this month. | This is expected money. It is not billed yet. Rent already billed is not counted again. Counts tenants living here whose rent is added automatically each month. |

**What changed and why**

| Card | Today it says | Why it changes |
|---|---|---|
| Rent | "Recurring rent expected before month end that has not been billed yet." plus "Forecast from tenant rent setup; excludes rent already raised." | Three of our words go: recurring, rent setup, excludes. The window is named plainly, tomorrow to month end. And the population is stated, because tenants on any billing frequency other than monthly are absent from this number and nothing on screen says so, `duesService.ts:756` |

---

## 10. Block 7, Deposit Dues

**Three numbers, and this block ignores the date chips while saying so on only two of the three.**

| Card | What is this? | How is this calculated? |
|---|---|---|
| **Total Deposits** | Deposit money held plus deposit still unpaid. | The dates you pick at the top do not change this. |
| **Received** | Deposit money collected and still held. | Refunds are taken off, and so is deposit already used to clear a bill. Only tenants living here or booked, so deposits of people who have moved out are not counted here. |
| **Due** | Deposit billed but not yet paid. | Only tenants living here or booked. The dates you pick at the top do not change this. |

**What changed and why**

| Card | Today it says | Why it changes |
|---|---|---|
| Total Deposits | "All deposit money, held plus still due. Always live." | Plain time phrase only. The sentence is right |
| Received | "Deposit collected and still held." plus "Net of refunds and deposit adjustments." | **Two changes, one of them large.** "Net of" is our phrase, and "adjustments" means nothing to her; both become plain. The large one: deposits belonging to people who have already moved out are not in this number, and on production that is 138,925 deposits still held. That is finding F22, ruled in D10, and it is the biggest silence on this block. It also carries no time phrase today, while the two cards beside it do |
| Due | "Deposit billed but not yet paid. Always live." | Plain time phrase, plus the population line |

**Three defects under this block that copy cannot fix.** Received and Due are summed from two different money columns on the bill, which differ on 54% of deposit bills, so the total adds two unlike things (finding F21). The category match is case sensitive, so 444 bills across 161 properties are invisible (finding F19). And deposit used to clear a bill counts only one of the two ways that can happen, so caution money settled against dues stays counted as held. Fix these before trusting the sentences above to hold.

---

## 11. Block 8, Breakup by Stay Duration

**Two numbers, and this block follows the date chips.**

| Card | What is this? | How is this calculated? |
|---|---|---|
| **Short Term** | Unpaid bills from short stay tenants, in the dates you picked. | Short stay comes from the tenant's own profile, not from the bill. Only tenants living here or booked. |
| **Long Term** | Unpaid bills from everyone else, in the dates you picked. | A tenant not marked short stay counts as long stay. Only tenants living here or booked. |

**What changed and why**

| Card | Today it says | Why it changes |
|---|---|---|
| Short Term | "Unpaid dues from short-term tenants in the period." | Where "short stay" comes from is the whole question. It is a flag on the tenant, so changing it moves historic bills between the two bars, `duesService.ts:242` |
| Long Term | "Unpaid dues from long-term tenants in the period." | "Long term" is not a flag anyone sets; it is everyone the short stay flag does not catch, including tenants where it was never filled in. Saying so stops her hunting for a long stay setting that does not exist |

---

## 12. Block 9, Dues by Property

**One number per property, and this block follows the date chips.**

| Card | What is this? | How is this calculated? |
|---|---|---|
| **Dues by property** | Unpaid bills for each property in the dates you picked, highest first. | Share is that property's part of your total. Only tenants living here or booked. |

**What changed and why**

| Card | Today it says | Why it changes |
|---|---|---|
| Dues by property | "Unpaid dues for each property in the selected period, ranked high to low." plus "Share = the property's portion of the account total." | Close to right already. "Ranked high to low" becomes "highest first", the equals sign goes, and the population line is added |

---

## 13. Copy that flips when a known defect is fixed

Every sentence above describes the number as it stands today. These are the ones that change when the defect under them is fixed, so nobody has to rediscover it.

| Card | Defect | The line that goes, once fixed |
|---|---|---|
| Current FY Dues | Runs to 31 March instead of today. F1, F9, ruled D2 | "Includes bills dated later this year that are not due yet." The first line becomes "Unpaid bills due from 1 April up to today." |
| {Month}'s Projected Due | Bookings are not counted. F2 | "Bookings are not counted." |
| Bills Summary, Received | Bills cleared from deposit or advance count as money received | "Bills cleared from a tenant's own deposit or advance also count as received." |
| Deposit Dues, Received | Deposits of people who have moved out are not counted. F22, ruled D10 | "so deposits of people who have moved out are not counted here" |
| Every card carrying the population line | Old tenant dues get their own home | The population line stays, but it should then point at where that money now lives |

---

## 14. What this pass did not cover

| Not covered | Why |
|---|---|
| The View all sheet behind Overview | It repeats two Overview defects and needs the same fix applied twice, finding F25. Its copy should follow this document, not be written separately |
| The Others overflow sheets on Category and Added By | They carry no hint text today. Worth deciding whether they need any |
| Empty and healthy state messages | Written inline in the service rather than in the hint layer. They are copy, they were reviewed in the build verification pass, and they are not part of the (i) sheet |
| The change chips on Overview | Two of them are worked out against the wrong figure and coloured the wrong way round, findings F5 and F6. Explaining a chip that is being rebuilt would be writing copy for a number that is about to change |
| Whether these sheets are ever opened | Nobody knows. There is no measurement on the (i) sheet, on any tab |
