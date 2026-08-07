---
title: Detailed Analytics — Index
date: 2026-06-06
tags:
  - rentok
  - prd
  - spec
aliases:
  - DA Index
  - Detailed Analytics Index
---

> [!INFO] How this folder is organised
> One folder per screen (`DA-01 … DA-08`). Each screen has **three canonical docs** and an `archive/` for old versions and pre-split monoliths.
> - **Brief** — the WHY (what the operator must see, plain language)
> - **Ground-Truth Formula Map** — the truth (how each number is actually computed)
> - **Build Sheet** — the HOW (schema, columns, queries for engineers)
> Anything in `archive/` is superseded — kept for history, not current.

## Status

| Screen | Topic | Brief | Ground-Truth | Build Sheet | Notes |
|--------|-------|:-----:|:------------:|:-----------:|-------|
| DA-01 | Dues | ✅ | ✅ | ✅ | also has `Spec` + `Engineering` (pilot — richer than others) |
| DA-02 | Collections | ✅ | ✅ | ✅ | — |
| DA-03 | Refunds | ✅ | ✅ | ✅ | — |
| DA-04 | Expenses | ✅ | ✅ | ✅ | — |
| DA-05 | Discounts | ✅ | ✅ | ✅ | — |
| DA-06 | Liabilities | ✅ | ❌ | ✅ | Ground-Truth not written yet |
| DA-07 | Cash Flow | ✅ | ❌ | ✅ | Ground-Truth not written yet |
| DA-08 | Occupancy | ✅ | ✅ | ✅ | also has Flat-Group Properties tracking doc |

## Conventions

- **Current = the un-suffixed filename.** `DA-0X Brief.md`, `DA-0X Ground-Truth Formula Map.md`, `DA-0X Build Sheet.md`. No `V0.x` / `(plain)` suffixes at top level anymore.
- **New version?** Edit the canonical file in place. Only move the old copy to `archive/` if you need to keep the prior state for reference — don't spawn `… V0.7.md` next to it.
- **`_meta/`** holds the cross-screen templates: Build Sheet Generation Spec, Ground Truth Field Map, and the DA-01 §1 two-table pilot.
- **`DA Suite — Cross-Screen Figma Audit.md`** stays at top level (spans all screens).

## Gaps

- **DA-06 Liabilities** and **DA-07 Cash Flow** have no Ground-Truth Formula Map. They are the two heaviest screens — worth writing one so the build sheet numbers are auditable.
