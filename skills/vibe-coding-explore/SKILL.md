---
name: vibe-coding-explore
description: |
  Token-efficient codebase navigation using claude-mem's smart-explore tools.
  Replaces full file reads with AST-powered outline + unfold pattern.
  Uses smart_search, smart_outline, and smart_unfold instead of Read for code files over 100 lines.
  Saves 4-8x tokens on code understanding during BUILD, DEBUG, and REVERSE-ENGINEER phases.
  Use when: (1) About to read a code file over 100 lines — use this instead of Read,
  (2) Need to find a function, component, or class across the codebase,
  (3) Need to understand a file's structure before modifying it,
  (4) vibe-coding-build or vibe-coding-debug needs to navigate the codebase.
  Requires claude-mem plugin to be installed. Falls back to standard Read if not available.
---

# Vibe Coding — Token-Efficient Code Exploration

Navigate the codebase without loading whole files. Index first, fetch only what you need.

## Decision Rule

If called without a file path or search query → output:
```
EXPLORE: No file or query specified.
Usage: "explore [filename]" or "find [symbol/function name]"
```
Then wait for the user to provide context.

Apply this rule before every file read. Do not skip to ## 3 Tools before checking file size.

```
File size check before any Read on a code file:

Under 100 lines → Read directly (small, no overhead)
100-500 lines → smart_outline → smart_unfold only needed symbols
Over 500 lines → smart_search first → smart_outline → smart_unfold
Config, JSON, CSS, markdown → Read directly (AST doesn't help)
```

---

## 3 Tools (require claude-mem)

**`smart_search`** — Find files and symbols (use FIRST to locate before reading anything)
```
smart_search("UserCard component")
smart_search("POST /api/products")
smart_search("--color-primary CSS variable")
```
Returns: ranked file list + symbol names with line numbers (~2-6k tokens). Don't read files yet.

**`smart_outline`** — Get file skeleton without loading body
```
smart_outline("src/components/features/ProductCard.tsx")
```
Returns: all function/component/class signatures with line numbers (~1-2k tokens). Decide which symbols you need.

**`smart_unfold`** — Load one specific symbol
```
smart_unfold("src/components/features/ProductCard.tsx", "ProductCard")
smart_unfold("lib/hooks/useCart.ts", "addItem")
```
Returns: full source of just that symbol (~400-2100 tokens).

---

## Workflow

Old (heavy): `Read ProductCard.tsx` → 400 lines = ~3,000 tokens

New (lean): `smart_outline ProductCard.tsx` (~800) → `smart_unfold "ProductCard"` (~600) = ~1,400 tokens total (55% savings)

---

## When to Use Per Phase

**BUILD:** `smart_outline` before modifying existing component, `smart_search` before changing shared code.

**DEBUG:** `smart_search("error message")` to find bug location, `smart_unfold` only the buggy function.

**REVERSE-ENGINEER (vibe-coding-re-analyze):** `smart_search + smart_outline` for every area. Only `smart_unfold` key business logic. Saves 6-12x tokens on large codebases.

---

## Fallback (claude-mem not installed)

```
1. Grep function/class name to find exact line number
2. Read file with offset + limit (±30 lines around target)
3. Never read a full file when you only need one function
```

---

## Token Reference

| Operation | Approximate tokens |
|-----------|-------------------|
| Read 500-line file | ~3,500 |
| smart_outline 500-line file | ~800 |
| smart_unfold one function | ~400-600 |
| smart_search query | ~2,000 |
| **Savings per file** | **~60-70%** |
