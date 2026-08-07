---
title: DA Suite — Build Sheet Generation Spec
date: '2026-05-08'
tags:
  - rentok
  - prd
  - build-sheet
  - meta
  - generation-spec
status: V1 — locked
for: Phase 3 sub-agents and human Build Sheet authors
companion_to: All 7 DA spec PRDs, _Ground Truth Field Map
---

# DA Suite — Build Sheet Generation Spec

> **Purpose:** This is the deterministic prompt template for generating ticket-ready Build Sheets for each of the 7 DA specs (DA-01 Dues, DA-02 Collections, DA-03 Refunds, DA-04 Expenses, DA-05 Discounts, DA-06 Liabilities, DA-07 Cash Flow). Phase 3 sub-agents read this spec + the Field Map + the corresponding PRD, then produce a Build Sheet at the locked path with the locked structure.
>
> **Audience:** sub-agents (and humans) writing Build Sheets. Once locked, deviations require explicit user approval.
>
> **Goal:** Same format across all 7 Build Sheets. Same conventions. Same gloss rules. Engineers (Jatin, Parishi, Vivek, Harsh, Shiv, Ansh) read DA-N Build Sheet and DA-M Build Sheet identically — no cognitive switching cost between specs.

---

## 1. Sources of Truth (read these first, in order)

Before writing a single row, the agent must read:

1. **`_Ground Truth Field Map.md`** (this folder) — entity columns, status enums, constants, permission keys, production aggregation formulas, endpoint paths with auth state, hallucination table. **Source of truth for everything technical.**
2. **The corresponding PRD** (e.g., `DA-02 Collections Detailed Analytics.md`) — the canonical "why" doc. Source of truth for **business intent, persona context, decisions, edge cases, microcopy strings**.
3. **The existing Build Sheet** if it exists (DA-01 has one; others don't yet) — for structure inspiration only. Conventions in this Generation Spec override anything in older Build Sheets.
4. **`src/v1/list_screens/<spec>/`** code (if it exists) — for verbatim production formulas not yet captured in the Field Map. If a formula is needed and not in Field Map, agent reads the source, adds the formula to Field Map (with file:line), then proceeds.

**Never write a code reference (table, column, function, file:line) that hasn't been verified.** If genuinely unknown, mark `[VERIFY]` with reason and move on.

---

## 2. Locked Column Structure (9 columns)

Every Build Sheet uses the exact same 9 columns, in this exact order:

| # | Column | Width hint | Required content |
|---|--------|------------|------------------|
| 1 | **Component** | narrow | Visual sub-grouping noun (e.g., `Hero`, `MoM Chip`, `Forecast row`, `Row badges`). Repeats for all rows in a group. |
| 2 | **Status** | narrow | One or compound: `EXISTS` / `EXTEND` / `BUILD` / `COMPOSE` / `[VERIFY]`. Compound with `+` (e.g., `EXTEND + [VERIFY]`). |
| 3 | **Task** | medium | Short ticket-able label. Imperative or noun-phrase. Atomic — one row = one ticket. |
| 4 | **Definition + [class]** | wide | Plain-English description of what this row is, ending with `[<class>]` tag where `<class>` is one of: `always-live`, `period-sensitive`, `today-mode-only`, `mode-bifurcated`, `always-live tooltip text`, `period-sensitive UI render`, `always-live + property override`, etc. Class tag is mandatory on all rows except UI-render-only rows. |
| 5 | **Plain Formula** | wide | 1–3 sentences in plain English explaining the calculation logic. Audience: PM / CS / QA / new dev. No SQL, no entity references, no jargon. If the row is UI-only, write `n/a — visual element only` or `n/a — render rule only`. |
| 6 | **Formula (technical)** | wide | SQL/QueryBuilder code, verbatim from production where possible. Includes inline plain-English glosses on first-occurrence code references in italics (see §5 below). Cite file:line at the end. If formula doesn't exist in production, write `**NEW BUILD**` + the recommended formula. |
| 7 | **Permission** | medium | Format: `view: <flag>` or `view: <flag> · action: <action_perm>`. Use derived ctx flags (`can_view_invoices`, `can_view_tenants`) for view; use action perms (`record_payment`, `add_refund_access`, etc.) for actions. See Field Map §3.3 for derived flags, §3.2 for DB columns. |
| 8 | **Plain Drill** | wide | **3-line plain-English template** mirroring the Tech Drill structure: `→ Live (today): <plain description>` / `→ Past period: <plain description OR "Same as live — ..." OR "Doesn't apply — reason">` / `→ Future: <plain description OR Same OR Doesn't apply>`. Each line in plain English — no SQL, no entity names. Include warning prose (`⚠️`) for unauth or composite drill destinations. If row is non-tappable, write `Display only — not tappable` once (no 3-line structure needed). Audience: PM / CS / QA reading without engineering vocabulary. |
| 9 | **Drill Behavior (technical)** | wide | 3-line template covering Live / Past period / Future contexts. Each line: `→ <Context>: <dest> · time: <tag> · property: <tag> · filters: <list> · sort: <order> · "<header>"`. Use `SAME` for repeated patterns; `N/A` for non-applicable. Cite CSB/HB warnings inline with `⚠️`. See §6 below for full template. |

**Markdown table format:** standard pipes; cells can be multi-line via `<br>` tags. Headers in the same order as the table above.

---

## 3. Section Header Callouts (above every table)

Every section in a Build Sheet starts with a quoted callout block, in this exact order:

```markdown
## <Section #>. <Section Name>

> **Endpoint:** <verb + path> — `EXISTS` / `EXTEND` / `NEW BUILD` (file:line if exists)
> **Default property context:** <inherited from dashboard | scoped to tenant's pg | always-live single | etc.>
> **Default time context:** <period-sensitive | always-live | mode-bifurcated>
> **Cross-Suite Blockers:** <list CSB IDs that affect this section, with one-line summary each>
> **Hard Blockers:** <list HB IDs from the PRD, with one-line summary>
> **Eng note:** <1-3 sentences on entity/table/column conventions, paise vs rupees, basis date, mode handling, etc.>
> **⚠️ PRD vs codebase gap:** <only when applicable — divergences requiring Jatin/PM gate>
> **Filter codes:** <enum and range; "EXISTS" or "NOT BUILT — reserve range XXXX-YYYY">
```

Sections without a meaningful Cross-Suite or Hard Blocker callout can omit those lines, but **Endpoint, Default property context, Default time context, and Eng note are mandatory.**

---

## 4. Status Values (Column 2) — When to Use Each

| Status | When to use | Example |
|--------|-------------|---------|
| **EXISTS** | Endpoint AND logic both exist; only UI wiring remains. No backend work. | "Hero — payment count subtitle" — COUNT exists in current query, just needs to be exposed in response shape |
| **EXTEND** | Endpoint exists; needs additive backend work (new field, new aggregation, new filter branch). | "Net Collected number" — main SUM exists; refund subtraction is the additive gap |
| **BUILD** | Net-new endpoint OR net-new query OR net-new UI element with no precedent. | "DA-05 Discount worklist" — no list endpoint, no aggregator |
| **COMPOSE** | Multiple existing endpoints chained client-side; no aggregate endpoint. | "Open Settlement" button on DA-06 forecast row — composes 5 existing endpoints |
| **DEFERRED** | Item explicitly deferred to a later release (V2.0+) per PRD/V2 Roadmap. NOT a blocker; layout reserves visual space if applicable. | "DA-01 Repeat badge" — V2.0 P0-#5 deferred per V2 Roadmap |
| **[VERIFY]** | Field Map's confidence is < 95% on this specific item, OR the spec has a gap that requires spike-time decision. **Always combined** with another status (e.g., `EXTEND + [VERIFY]`). | A specific row whose data shape needs spike-time confirmation against codebase reality |

**Status compounds with `+`:** e.g., `BUILD + [VERIFY]` means "build new AND verify a sub-question during spike."

**[VERIFY] discipline:** every `[VERIFY]` must include the specific question to answer in the Plain Formula or Formula cell. Generic "needs verification" is not enough.

---

## 5. Code Reference Rules (Column 6)

**Citation format:** `file_path:line` — relative to `/Users/eazypg/rentok-backend/`. Always cite at the end of the cell.

**First-occurrence-per-section gloss rule:** the **first** time a code reference appears in a section, add an inline plain-English gloss in *italics within parentheses*. Subsequent occurrences in the same section don't need re-glossing.

Example (first occurrence in section):
```
SUM(p.net_amount - p.gateway_charges) WHERE p.status = 1 AND p.is_active = 1 *(p = Payments table; status=1 = success per entity comment 0-failed/1-success/2-pending/3-refunded; is_active=1 = active row)*
```

Example (later occurrence in same section):
```
COUNT(*) over the same query (filtered by p.status = 1 AND p.is_active = 1)
```

**Glossable references:**
- Table aliases (`p`, `inv`, `t`, `c`, `r`)
- Column names with non-obvious semantics (`paid_date` vs `created_at`; `due_type = 'Advance'` typo `ADVANCE_DUE_TYE`)
- Status enum values (`status = 1` → "= success")
- Mode integer values (`payment_mode IN (211, 288)` → "deposit-applied + advance-applied modes, excluded from grand total")
- DB permission column names (`view_invoices_of_self_added_tenants` → "DB column via `checkAuthInDb` for self-added tenant fallback")

**Non-glossable** (skip the gloss):
- Standard SQL keywords (SUM, COUNT, WHERE, JOIN)
- Obvious column names (`id`, `name`, `created_at` once basis date is established)
- File:line citations themselves

---

## 6. Drill Behavior Template (Column 9)

Every row's Drill Behavior cell uses this 3-line structure:

```
→ Live (today): <dest> · property: <tag if differs from section default> · filters: <list> · sort: <order> · "<header>"
→ Past period: <dest> · property: <tag> · filters: <list with period inherited> · sort: <order> · "<[Period] header>"  OR  SAME (always-live)  OR  N/A
→ Future: <dest> · property: <tag> · filters: <list> · sort: <order> · "<header>"  OR  SAME  OR  N/A
```

**`<dest>` abbreviations:**
- `WL` = Worklist (full-screen list, with chip rail)
- `BS` = Bottom sheet — **the canonical destination for ⓘ icon tap across the DA suite (per 2026-05-11 audit). Replaces the old "tap=inline tooltip + long-press=BS" dual pattern.** Single interaction: tap on ⓘ → BS opens with plain-English explanation + GAAP framing where applicable
- `FS` = Full-screen push (e.g., Tenant Detail)
- `M` = Modal overlay
- `T` = Tooltip (inline). **⚠️ DEPRECATED for ⓘ icons across DA suite per 2026-05-11 audit** — use `BS` instead. `T` remains valid for other UI primitives (chip tooltips, micro-help icons inside accordions) where designer decides; verify with design system. **No long-press patterns** in DA suite — single-tap only
- `inline` = expand-in-place (accordion)

**`time:` tags:** `always-live` | `today only` | `period inherited` | `period override` | `n/a`

**`property:` tags:** Only specify if **differs from section default** (declared in section header callout). Otherwise inherit. Override examples: `scoped to tenant's pg`, `scoped to [PGx]`, `all-properties aggregate`, `inherited - locked`.

**`filters:` list:** comma-separated. Use real column names + values (with first-occurrence gloss if needed).

**`sort:` short:** `amount DESC` (or `↓`), `due_date ASC`, `most overdue first`, `n/a`.

**`"<header>"`:** exact destination header text in quotes. Use bracketed placeholders like `[Period]`, `[Property X]`, `[Tenant Name]` for dynamic substitutions.

**Inline warnings:** if drill destination has known unauth gap or HB-NEW dependency, prefix with `⚠️` and one-line summary referencing the CSB/HB ID. Example:
```
→ Live: FS (Tenant Detail at POST /tenant/getTenantData)
⚠️ destination /tenant/getTenantData is unauth (CSB-3) — Jatin gate
→ Past period: SAME
→ Future: SAME
```

**SAME and N/A discipline:**
- `SAME` = identical drill behavior for that context (write only when truly identical)
- `N/A` = the context doesn't apply (e.g., Future for collections — collections cannot be future-dated)
- Never leave a context line blank — explicit "SAME" or "N/A" is required

---

## 7. Permission Cell Format (Column 7)

Codebase has row-level permissions, NOT section-level. Every row's permission cell must use this format:

| Pattern | Example | When to use |
|---------|---------|-------------|
| `view: <flag>` | `view: can_view_invoices` | View-only row, single permission |
| `view: <flag> OR <flag>` | `view: can_view_invoices OR view_invoices_of_self_added_tenants` | Production fallback pattern (DA-01, DA-02 use this) |
| `view: <flag> · action: <action>` | `view: can_view_invoices · action: record_payment` | Row with mutation action button |
| `view: inherits hero visibility` | `view: inherits hero visibility` | Sub-element of a parent row whose visibility cascades |
| `view: always · action: <action>` | `view: always · action: owner_only` | Always-visible action gated only on action perm |

**Always use derived ctx flags (`can_view_X`) for the `view:` portion** when available — these are the canonical computed flags from `homepage/service.ts:380`. See Field Map §3.3.

**Use DB column names for action perms** when they're DB-only (snake_case). See Field Map §3.2.

**For fictional permission keys** (Field Map §3.4), use the recommended substitute and add a footnote at the bottom of the section: `Note: PRD cited 'X', codebase doesn't have it; using <substitute> per Field Map §3.4. Jatin gate: build new vs reuse.`

---

## 8. Per-Sheet Required Components

Every Build Sheet has these blocks at the top (before Section 1):

### 8.1 Frontmatter

```yaml
---
title: DA-NN Build Sheet
date: '2026-05-08'
tags: [rentok, prd, build-sheet, engineering, homescreen, financial-insights, detailed-analytics, <spec-keyword>]
status: V1 — locked
for: Engineering ticket creation
companion_to: DA-NN <Spec Name> Detailed Analytics
---
```

### 8.2 Source of Truth Callout

```markdown
> [!INFO] Source of Truth
> **Companion to:** [[DA-NN <Spec Name> Detailed Analytics]] — canonical "why"
> **Reference:** [[_Ground Truth Field Map]] — entity fields, formulas, endpoints, permissions
> **Generation spec:** [[_Build Sheet Generation Spec]] — column conventions, gloss rules, status values
> **For:** Engineers picking up tickets
```

### 8.3 Glossary

```markdown
## Glossary

Plain-English definitions of terms used in this Build Sheet. Read before scanning rows.

| Term | Plain English |
|------|---------------|
| ... | ... |
```

Glossary content is per-spec but always covers: paise, pg_id, GAAP basis terms used in heroes, mode/status enum values referenced in this spec, tenant.status values used, "Live" vs "Range" mode definitions, any custom terms from the PRD.

### 8.4 Engineering Architecture Reference

```markdown
## Engineering Architecture Reference

| Concern | Path / Reference |
|---------|------------------|
| Routes file | `src/routes/<x>.ts` (NEW) or `src/v1/list_screens/<x>/routes.ts` |
| Controller | TBD or `src/controllers/<x>.ts` |
| Service | `src/v1/list_screens/<x>/service.ts` (NEW) or existing path |
| Helpers | `src/v1/list_screens/<x>/helpers.ts` |
| Entity | `src/entities/<x>.ts` |
| Filter codes | `src/v1/constants/filterCodes.ts` — `<X>FilterCode` enum (range XXXX-YYYY) |
| Permission keys | Field Map §3 |
| Migrations | TBD |
```

### 8.5 Cross-Suite Blockers (link, don't duplicate)

```markdown
## Cross-Suite Blockers Affecting This Spec

Surfaced from `[[DA-NN <Spec Name> Detailed Analytics#Cross-Suite Engineering Blockers]]`:

| ID | Issue (one-line) | Status | Affects rows |
|----|------------------|--------|--------------|
| CSB-1 | ... | ⛔ Jatin gate | All rows with auth |
| CSB-X | ... | ... | Section N rows |
```

### 8.6 Hard Blockers (link, don't duplicate)

```markdown
## Hard Blockers (HB)

Surfaced from PRD's Pre-Launch Engineering Blockers section:

| ID | Issue | Status |
|----|-------|--------|
| HB1 | ... | ... |
```

### 8.7 Pre-Build Decisions Pending (Jatin)

```markdown
## Pre-Build Decisions Pending

These must be answered before Phase 1 begins:

| Decision | Recommendation | Owner |
|----------|----------------|-------|
| ... | ... | Jatin |
```

---

## 9. Plain English Discipline (Columns 5 and 8)

The Plain columns are the Build Sheet's multi-audience guarantee. Every PM / CS / QA / new-dev person should be able to scan them and understand the spec without engineering vocabulary.

### Rules

1. **No SQL.** No `SELECT`, `WHERE`, `JOIN`, no column names, no table names. If a column or table name slips in, it's a tech-cell concern, not plain.
2. **No jargon undefined.** "GAAP basis" is OK only if the Glossary defines it. "Paise" is OK only if Glossary defines it. Otherwise rewrite in plain language.
3. **Use the operator's vocabulary.** Refer to "the user's tenant," "the room," "the bill," "the discount." Avoid "the entity," "the record," "the row."
4. **1–3 sentences max.** Not paragraphs. If you need more, split into multiple rows or put detail in the Definition cell.
5. **Show concrete examples.** "If Riya paid 3 separate times in the period, that counts as 3 events" beats "count distinct payment events."
6. **Replace negative-only conditions with positive framing where possible.** "Show ✅ when checklist is approved" beats "Show ✅ unless checklist is not yet approved."

### Anti-patterns (reject)

- "Total of all invoices where status is 0 minus refunds." → has SQL semantics in plain prose, no value.
- "The result of the SUM aggregation function." → meaningless to non-engineers.
- "Cardinality of the resulting set." → math jargon. Use "how many."

### Plain Drill — special note (3-line template)

Plain Drill describes **what the user experiences across all 3 contexts, not what the system does.** Mirror the Tech Drill cell's 3-line structure but in plain language.

**Template:**
```
→ Live (today): Tap to open the list of overdue bills, sorted by most overdue first. Header: "Already Due · X Bills · ₹Y".
→ Past period: Same as live — the segment isn't sensitive to the date filter (always-live). The header still reads "Already Due...".  
              OR  Doesn't apply — this segment doesn't appear when a past period is selected (Range mode shows different segments — see Section 3).
→ Future: Same as live.
         OR  Doesn't apply — collections cannot be future-dated.
```

**Discipline:**
- Bad: "Routes to the worklist with `due_date < today` filter applied." (system-speak, references a column)
- Good: "Tap to open the list of all overdue bills, sorted by most overdue first."
- Bad in past-period: "Inherits dashboard period filter, applied to worklist with `start_date=period_start, end_date=period_end`."
- Good in past-period: "When the dashboard is set to a past period (like 'Last Month'), the same tap takes the user to the same list, but limited to bills due in that period. The header includes the period name."

**`Same as live` vs verbatim repeat:** if the behavior is genuinely identical, write `Same as live — <one-line reason>` rather than restating the full description. Saves vertical space and reinforces the always-live concept.

**`Doesn't apply` reasons:**
- "this segment doesn't render in Range mode" (mode-bifurcated)
- "collections cannot be future-dated" (data domain limit)
- "this row is always-live so future period doesn't change behavior" (variant of Same)

**Inline warnings (`⚠️`):** include in the most relevant context line. Example:
```
→ Live: Tap to open the bill detail screen.
       ⚠️ The destination screen does not currently check user permissions — engineering must fix before launch.
→ Past period: Same as live (warning still applies).
→ Future: Same.
```

---

## 10. Glosses on Code References (Column 6)

See §5 for the format. Additional discipline:

- **First-occurrence-per-section, not per-row.** A reader scanning all rows in Section 1 sees the gloss once at the top.
- **Gloss should explain the SEMANTIC, not the syntax.** `is_active=1 = active row` is OK; `is_active=1 = the value 1 in the is_active column` is not (says nothing).
- **Constants imported from `payment/constants.ts` get the constant name, not the literal.** Use `DEPOSIT_DUE_TYPES` in the SQL instead of writing out `('Security Deposit', 'Caution Money')` inline. The constant is verifiable; literals risk drift.
- **Mode integer ranges deserve the full decoded list once per section.** `payment_mode IN (211, 288)` → `(211 = deposit-applied; 288 = advance-applied; both excluded from Net Collected per production logic at helpers.ts:430-450)`.

---

## 11. Output Path Convention

Each Build Sheet is written to:

```
/Users/eazypg/Documents/Obsidian Vault/RentOk/PRDs/Homescreen Detailed Analytics/DA-NN Build Sheet.md
```

Existing `DA-01 Build Sheet.md` (May 1 vintage) gets overwritten by Phase 2. Other DA Build Sheets are net-new in Phase 3.

**File naming:** literal `DA-NN Build Sheet.md` — no `.v2.md`, no `_new.md`. Versioning is in the frontmatter `version` field and the in-doc maintenance log section.

---

## 12. Final Section Conventions

Every Build Sheet ends with these standard sections (after the per-section row tables):

### 12.1 Reconciliation Invariants

QA-runnable checks. Strict equalities or bounds. Same format as DA-01 Build Sheet §16. No drill-down, no permission, no plain — these are pure correctness checks.

### 12.2 Edge Cases

`EC-NN: Description | Behavior | Trigger condition`. Same format as DA-01 Build Sheet §17.

### 12.3 Microcopy

Static strings used across the spec. `Component | Task | Exact string`. PRD's microcopy section is canonical; this section copies-with-attribution.

### 12.4 Smoke Test Commands

For every endpoint cited in the Endpoint column (or its drill destinations), one curl example:

```bash
# Net Collected hero, This Month, single property
curl -X POST 'https://<host>/v1/list_screens/collections/list/filters' \
  -H 'Authorization: Bearer <token>' \
  -H 'Content-Type: application/json' \
  -d '{ "pg_number": "PG123", "filter_codes": ["1201"], "limit": 20, "offset": 0 }'
```

### 12.5 How to Use This Doc

Standard footer copy:

```markdown
- Pick a row → that's a ticket (or part of one). Status column tells you whether to extend, build, compose, or wire UI.
- Plain Formula + Plain Drill columns are for PM / CS / QA review.
- Formula (technical) + Drill Behavior (technical) columns are for implementation.
- All code references cite file:line. First-occurrence-per-section gets a plain-English gloss in italics.
- For "why" or rationale, see [[DA-NN <Spec Name> Detailed Analytics]].
- For entity fields, formulas, permissions, endpoints, see [[_Ground Truth Field Map]].
- For format conventions, see [[_Build Sheet Generation Spec]].
- If a row is ambiguous, the PRD is the source of truth. Don't guess — ask.
```

### 12.6 Maintenance Log

```markdown
| Date | Change | By |
|------|--------|-----|
| 2026-05-08 | Initial generation per [[_Build Sheet Generation Spec]] V1 | (agent name or PM) |
```

---

## 13. Phase 3 Sub-Agent Prompt Template

Use this exact prompt structure when spawning a Phase 3 sub-agent:

```
You are generating the Build Sheet for DA-NN <Spec Name>. This is a ticket-ready engineering spec.

Read in this order:
1. /Users/eazypg/Documents/Obsidian Vault/RentOk/PRDs/Homescreen Detailed Analytics/_Ground Truth Field Map.md — your source of truth for entity fields, formulas, endpoints, permissions.
2. /Users/eazypg/Documents/Obsidian Vault/RentOk/PRDs/Homescreen Detailed Analytics/_Build Sheet Generation Spec.md — the column structure, gloss rules, status values, plain-English discipline.
3. /Users/eazypg/Documents/Obsidian Vault/RentOk/PRDs/Homescreen Detailed Analytics/DA-NN <Spec Name> Detailed Analytics.md — the canonical PRD ("why").
4. /Users/eazypg/Documents/Obsidian Vault/RentOk/PRDs/Homescreen Detailed Analytics/DA-01 Build Sheet.md — the canonical exemplar (Phase 2 V2 version).

Then write the Build Sheet to:
/Users/eazypg/Documents/Obsidian Vault/RentOk/PRDs/Homescreen Detailed Analytics/DA-NN Build Sheet.md

Critical rules:
- Use the locked 9-column structure (Component | Status | Task | Definition + [class] | Plain Formula | Formula (tech) | Permission | Plain Drill | Drill Behavior (tech)).
- Every code reference must be cited from Field Map or the actual codebase. NO HALLUCINATION. If genuinely unknown, mark [VERIFY] with the specific question.
- Plain columns must be readable by PM / CS / QA without SQL or jargon.
- First-occurrence-per-section glosses on code references in Tech Formula column.
- Section headers include Endpoint / Default property context / Default time context / CSB / HB / Eng note callouts (mandatory) and PRD-vs-codebase gap callout (when applicable).
- End with: Reconciliation Invariants + Edge Cases + Microcopy + Smoke Test Commands + How To Use + Maintenance Log.
- All filenames, table names, column names, line numbers must match Field Map. If not in Field Map, verify via grep/Read on the codebase before citing.

Output: Write the file. Report a summary of what you wrote, citing any [VERIFY] tags or new findings that should be back-ported into Field Map.
```

---

## 14. MoM Prior-Period Convention (locked across all specs)

All specs that compute Month-over-Month (MoM) comparisons must use **"same-elapsed-days"** logic for prior-period:

- **For "Today" / "This Month" filter:** compare to the same-elapsed-days window in the prior month. Example on May 8 with month-to-date selected: compare May 1-8 to April 1-8 (8 days), NOT April 1-30 (full prior month).
- **For "Last Month" filter:** compare to "Two Months Ago" full month.
- **For Custom range (N days):** compare to the same N days immediately preceding the range start.
- **Hide the chip if:** account is < 30 days old OR prior-period value is ₹0 (avoid divide-by-zero).

This convention applies to: DA-01 hero MoM, DA-02 hero MoM, DA-03 hero MoM, DA-04 hero MoM + per-row Category MoM, DA-05 hero MoM + per-row Reason MoM, DA-07 hero MoM. **DA-06 has no MoM** (snapshot screen).

Build Sheet writers should remove `[VERIFY]` tags on MoM rows and cite this convention. Phase 4 audit identified inconsistent wording across sheets ("same-day-prior-month" vs "same-elapsed-days" vs "equivalent-prior-days") — this section locks the canonical phrasing.

---

## 15. ⓘ Icon Interaction Convention (locked across all specs, 2026-05-11)

For every `ⓘ` (info) icon in the DA suite:

- **Single interaction: tap → Bottom Sheet.**
- No inline tooltip on tap.
- No long-press primitive.
- Bottom sheet content: plain-English explanation up top; expanded GAAP-framed definition below (where applicable to the metric).
- Frontend agnostic: backend already serves `bottomsheet_title` / `bottomsheet_subtitle` payload shapes for similar surfaces — no new backend pattern needed.

**Applies to:** all hero ⓘ icons, all section-level ⓘ icons, all per-row ⓘ icons across DA-01 through DA-07.

**Where `T` (inline tooltip) remains valid:** chip tooltips, micro-help icons inside accordions, and any other non-ⓘ surfaces where the designer decides tooltip is appropriate. The DA suite ⓘ pattern is the only one explicitly locked here.

**Designer authority:** if a designer determines a specific ⓘ context warrants different behavior, they override locally. Document the override in the Build Sheet row's Drill cell.

---

## 16. Maintenance Log

| Date | Change | By |
|------|--------|-----|
| 2026-05-08 | Initial spec, V1 locked. | PM-bot |
| 2026-05-08 | **Phase 4 audit patch.** Added §14 MoM Prior-Period Convention locking "same-elapsed-days" globally — resolves [VERIFY] tags across DA-01/02/03/04/05/07 MoM rows that were waiting for spike resolution. | PM-bot |
| 2026-05-11 | **DA-01 spec audit patch.** Added §15 ⓘ Icon Interaction Convention — single-tap → BS only across all DA suite. Deprecated old "tap=inline tooltip + long-press=BS" dual pattern. Updated §6 Drill Behavior Template `BS` and `T` definitions accordingly. To be propagated to existing Build Sheets (DA-01 through DA-07) in P6 of the audit pass. | PM-bot |
