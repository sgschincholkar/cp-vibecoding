---
name: vibe-coding-impact-analysis
description: |
  Blast-radius analysis before making code changes. Maps which functions, components, and files
  are affected by a proposed change. Identifies test gaps, risk level, and execution flow impact.
  Prevents unintended regressions by scoping the full impact before touching any code.
  Use when: (1) vibe-coding-debug is about to apply a fix and needs to scope the impact first,
  (2) vibe-coding-build is about to refactor shared code (layout, utility, hook, store),
  (3) User says "what will this change break?", "what is affected by changing X?",
  (4) Before modifying any file used by more than 2 other files.
  Input: the file or function about to be changed. Output: impact map with risk score and safe change path.
---

# Vibe Coding — Impact Analysis

Map blast radius before changing shared code.

## Entry Router

```
Change type is DELETION?
→ Force HIGH risk immediately. List all N consumers. Require user confirmation.

Otherwise run Steps 1-5:
  Step 1: Identify target (file, function, change type)
  Step 2: Find direct consumers (grep imports)
  Step 3: Build transitive impact map
  Step 4: Score risk → READ: ## Risk Scoring
  Step 5: Output safe change path
```

**Run steps sequentially. Jump to ## Risk Scoring only after completing Steps 1-3. Jump to ## HIGH Impact Response only if risk score is HIGH — skip it otherwise.**

Always run before modifying:
- JS/TS: `components/ui/`, `lib/hooks/`, `lib/utils/`, `stores/`, `components/layouts/`, `lib/api/`, `types/`
- Python: `utils/`, `models/`, `services/`, `middleware/`, base templates, shared decorators

---

## Steps 1-3

**Step 1 — Target:** File path, export name, change type (interface change | rename | logic change | deletion | addition)

**Step 2 — Direct consumers:** Grep using the pattern that matches the language:
```
JS/TS:   import { [name] } from '[path]'   or   from '@/[path]'
Python:  from [module] import [name]        or   import [module]
Go:      "[module_path]"  (import block)    or   [package].[Name]
Ruby:    require '[path]'                   or   require_relative '[path]'
PHP:     require/include/use [ClassName]
```
List every importing/referencing file.

**Step 3 — Transitive map:**
```
[Target file]
  └── Direct: components/ui/Button.tsx
        └── Used by: components/features/ProductCard.tsx
              └── Used by: app/products/page.tsx
        └── Used by: components/layouts/Header.tsx
              └── Used by: app/layout.tsx (ROOT — affects ALL pages)
```

---

## Risk Scoring

| Files affected | Level | Action |
|----------------|-------|--------|
| Root layout reached OR >10 files | HIGH | Require user confirmation. Do not proceed automatically. |
| 4-10 files | MEDIUM | Test all direct consumers after change |
| 1-3 files | LOW | Spot-check sufficient |
| 0 (new addition) | NONE | No risk |

Interface breaking change or deletion always bumps to HIGH.

Test gap: for each direct consumer, check if `[Component].test.tsx` exists and covers the changed behavior.

---

## Report Format

```
IMPACT ANALYSIS — [target]

CHANGE TYPE: [type]

BLAST RADIUS:
  Direct consumers: [N]  |  Transitive: [N]  |  Root layout: [YES/NO]
  Risk level: [HIGH | MEDIUM | LOW | NONE]

AFFECTED FILES:
  [path] — [what it uses] — [test: Y/N]

TEST GAPS: [N] affected files untested

SAFE CHANGE PATH:
  1. [If interface change] Create new version alongside old → migrate consumers one by one → remove old
  2. Update [target], then [consumer 1] → verify → [consumer 2] → verify...
  3. Rollback plan: [what to revert]

PROCEED: [YES | CAUTION — test after each step | BLOCKED — user confirmation required]
```

---

## HIGH Impact Response

```
HIGH IMPACT DETECTED — [N] files affected including [root layout/shared store].

Options:
  A) Proceed with safe change path (step-by-step, test each consumer)
  B) Create parallel version, migrate gradually
  C) Scope down — smaller change affecting only [N] files
  D) Review with user before touching anything

Recommended: [A|B|C based on situation]

Waiting for confirmation before proceeding.
```
