---
name: vibe-coding-code-review
description: |
  Code quality review after each build step. Checks for clean code principles, maintainability,
  naming clarity, single responsibility, dead code, over-engineering, and security issues.
  Gives a pass/fail report with specific line-level fixes.
  Use when: (1) vibe-coding-build completes a step and runs a post-step quality check,
  (2) User says "review this code", "check code quality", "is this clean?",
  (3) Before marking a build step as complete.
  Input: one or more files just written/modified. Output: quality report with fixes.
  Focused on code quality and maintainability — not design/CSS (that's vibe-coding-design-guard).
---

# Vibe Coding — Code Review

Review code for quality and correctness. Fast, surgical checks after each build step.

## Entry Router

```
Simple config file or generated file with no logic?
→ Skip, mark as passed.

React/Next.js detected in package.json (and NOT react-native)?
→ Run 8 checks below AND delegate to vibe-coding-review-react. Combine findings.

"react-native" or "expo" in package.json?
→ Delegate to vibe-coding-react-native only. Do NOT run checks below.

Python/Go/plain HTML or other?
→ Run 8 checks below only (with language-specific adaptations noted per check).
```

**For React Native: delegate immediately and stop — do not run the 8 checks. For all other targets: run checks 1-8 sequentially, jump to each ## Check N section in turn.**

**Language adaptations:**
- Python: Check 1 = underscores not camelCase, no CamelCase for private vars. Check 2 = one class/function per file is idiomatic. Check 5 = use constants module or settings. Check 7 = type hints on params + return types, bare `except:` = FAIL.
- Go: Check 1 = short names OK in narrow scope (Go idiom). Check 6 = explicit error returns required, no `panic()` in library code. Check 7 = all exported functions must have godoc comment. Check 8 = no `fmt.Sprintf` to build SQL strings.
- Ruby: Check 1 = snake_case methods/vars, SCREAMING_SNAKE constants, no abbreviated names. Check 2 = one class per file. Check 3 = no `binding.pry` or `puts` debug left in. Check 6 = use rescue blocks on external calls. Check 7 = Sorbet type sigs on public methods if project uses Sorbet. Check 8 = no string interpolation in SQL queries (use `?` placeholders).
- PHP: Check 1 = camelCase methods, PascalCase classes, snake_case DB fields. Check 3 = no `var_dump`/`print_r` left in. Check 5 = use config constants or .env, not hardcoded strings. Check 7 = PHPDoc on public methods. Check 8 = no raw `$_GET`/`$_POST` in queries (use PDO prepared statements).

---

## Check 1 — Naming Clarity

FAIL: single-letter vars (except `i`,`j`), non-standard abbreviations (`usr`, `cfg`), booleans not starting with `is/has/can/should`, event handlers not starting with `handle`.

```tsx
// FAIL
const u = await getUser(id); const loading = false
// PASS
const user = await getUser(id); const isLoading = false
```

## Check 2 — Single Responsibility

FAIL: component fetches data AND renders AND handles logic in one place.

```tsx
// FAIL: ProductCard fetches + renders + cart logic
// PASS: ProductCard({ product, onAddToCart }) — receives data as props
```

## Check 3 — No Dead Code

FAIL: unused variables/functions/imports, commented-out blocks (>2 lines), `console.log` left in.

## Check 4 — No Premature Abstraction

FAIL: helper function called only once, generic utility for one-time use, component split before it's complex enough.

```tsx
// FAIL: capitalizeFirstLetter() used only once 3 lines later
// PASS: inline it directly
```

## Check 5 — No Hardcoded Values

FAIL: magic numbers inline, hardcoded URL strings in components, magic delays.

```tsx
// FAIL: if (items.length > 10), setTimeout(refresh, 5000)
// PASS: const MAX_ITEMS = 10; const REFRESH_MS = 5000
```

## Check 6 — Error Handling at Boundaries

FAIL: API calls with no error handling, async functions without try/catch or error state, form submissions ignoring server errors.

## Check 7 — Type Strictness

```
TypeScript (.ts/.tsx): FAIL if `any` used, type assertions without comment, return types missing on public functions, optional chaining missing where null possible.

Python (.py): FAIL if type hints missing on params, return type annotation missing, bare `except:`, `# type: ignore` without explanation.

JavaScript (.js/.jsx): skip this check entirely.
```

## Check 8 — No Security Anti-Patterns

FAIL: `dangerouslySetInnerHTML` with unsanitized content, user input concatenated into SQL/shell, hardcoded secrets, `eval()` usage.

---

## Report Format

```
CODE REVIEW — [filename(s)]

RESULT: PASS | FAIL | PASS WITH WARNINGS

MUST FIX (blocks step completion):
  1. [Check name] in [file:line]
     Problem: [description]
     Fix: [exact corrected code]

SHOULD FIX (fix before SHIP):
  1. [issue + fix]

PASSED CHECKS:
  - Naming: clear
  - Single responsibility: clean
  - [list passing checks]

[All pass:] CODE REVIEW: CLEAN — step complete.
```

Security issues are always MUST FIX regardless of scope.
