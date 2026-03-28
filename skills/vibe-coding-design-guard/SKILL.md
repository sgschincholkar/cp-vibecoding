---
name: vibe-coding-design-guard
description: |
  Design consistency enforcer. Checks any component or page for design token violations
  before code is committed. Catches hardcoded hex colors, inline font declarations, arbitrary Tailwind values,
  missing base layout extension, and undefined CSS variables.
  Use when: (1) vibe-coding-build is about to generate a page or component,
  (2) User reports design inconsistency — "colors look wrong", "fonts not matching",
  (3) Code review finds hardcoded style values.
  Call before each new page or component during BUILD phase.
---

# Vibe Coding — Design Guard

Check components and pages for design token violations. One MUST FIX violation = block generation.

## Prerequisites

Detect CSS approach and component language before running checks:
```
Read TECH_STACK.md:
  Tailwind CSS            → run all 7 checks as written
  Plain CSS / CSS Modules / SCSS → READ: ## Non-Tailwind Checks
  Python HTML templates (Jinja/Django) → READ: ## Template Checks

Read framework:
  React / Next.js / Remix → JSX/TSX patterns (className=, style={{}})
  Svelte / SvelteKit      → READ: ## Svelte Checks (uses class=, not className=)
  Vue / Nuxt              → READ: ## Vue Checks (uses class=, :style, SFC syntax)
```

**For non-Tailwind or templates: jump directly to that section and skip the 7 Tailwind checks entirely.**

---

## The 7 Checks

### Check 1 — No Hardcoded Hex Colors (MUST FIX)

FAIL: `style={{ color: '#3B82F6' }}` | `className="text-[#3B82F6]"` | `background: '#1a1a2e'`
PASS: `className="text-primary"` | `style={{ color: 'var(--color-primary)' }}`

### Check 2 — No Inline Font Declarations (MUST FIX)

FAIL: `style={{ fontFamily: 'Inter, sans-serif' }}` | `className="font-['Playfair_Display']"`
PASS: `className="font-sans"` | `className="font-heading"`

### Check 3 — No Arbitrary Tailwind Values (SHOULD FIX)

FAIL: `className="mt-[23px] p-[13px] text-[15px]"` | `className="bg-[#ff6b35]"`
PASS: `className="mt-6 p-4 text-sm"` | `className="bg-accent"`

### Check 4 — Pages Extend Base Layout (MUST FIX)

FAIL: Page file contains its own `<header>`, `<nav>`, `<footer>` or sidebar
PASS: Page outputs content only — layout chrome in layout files only

### Check 5 — CSS Variables Defined in DESIGN_SYSTEM.md (MUST FIX)

If DESIGN_SYSTEM.md not found → skip this check, note "Design system not yet generated" in the report, and continue with remaining checks. Do not block.

Read DESIGN_SYSTEM.md, extract all `--color-*`, `--font-*`, `--space-*` tokens.
FAIL: Component uses `var(--color-brand-blue)` if that token is not in DESIGN_SYSTEM.md
PASS: All `var()` references exist in the token list

### Check 6 — No Inline Style Blocks (MUST FIX)

FAIL: `<style>{`.card { background: #fff; }`}</style>` | `style={{ marginTop: 24 }}`
PASS: All styles via Tailwind or CSS variable references
Exception: `style` prop allowed for dynamic values only (e.g., CSS custom properties for animations, JavaScript-calculated widths)

### Check 7 — Responsive Classes Are Mobile-First (SHOULD FIX)

FAIL: `className="p-8 sm:p-4"` (desktop-first)
PASS: `className="p-4 md:p-6 lg:p-8"` (mobile-first)

---

## Guard Report

```
DESIGN GUARD REPORT — [ComponentName]

PASS: [N] checks passed  |  FAIL: [N] violations

MUST FIX (blocks generation):
  1. [Check name] — [specific line/issue]
     Fix: [exact corrected code]

SHOULD FIX (non-blocking):
  1. [Check name] — [suggested improvement]

[All pass:]
DESIGN GUARD: ALL CHECKS PASSED — safe to generate/commit.

[MUST FIX violations:]
DESIGN GUARD: BLOCKED — fix MUST FIX violations first. Re-submit after fixes for re-check.
```

**After MUST FIX fixes:** Re-run all 7 checks from scratch. Output "RE-CHECK PASSED" only after all resolved.

**After all 7 checks pass (or only SHOULD FIX remain):** Delegate to `vibe-coding-web-design-guidelines` on the same files. Append findings under "WEB GUIDELINES FINDINGS". Accessibility issues from web-design-guidelines are always MUST FIX.

---

## Non-Tailwind Checks

For plain CSS / CSS Modules / SCSS:
- Skip checks 3 and 7 (Tailwind-specific)
- Check 1: look for hardcoded hex in CSS files, not className attributes
- Check 2: look for `font-family` property in CSS, not className
- Check 5: verify CSS properties use `var(--token)` references from DESIGN_SYSTEM.md
- Check 6: no inline `style=""` attributes with hardcoded values

---

## Svelte Checks

For Svelte / SvelteKit components:
- Use `class=` not `className=` — Svelte uses HTML syntax
- Check 1: FAIL on hardcoded hex in `style=""` or `<style>` block; PASS on `var(--token)` or Tailwind class
- Check 2: FAIL on `font-family` in `<style>` block; PASS on CSS variable references
- Check 4: pages use `<slot />` pattern for layout, not layout chrome in page components
- Check 6: no inline `style=""` with hardcoded values; dynamic binding `style:color={var}` is OK
- Tailwind checks (3, 7) apply if Tailwind is in the stack, adapted for `class=` syntax

---

## Vue Checks

For Vue / Nuxt single-file components (`.vue`):
- Use `class=` not `className=` — Vue uses HTML syntax in `<template>`
- Dynamic classes use `:class=` binding syntax, not string literals with hex
- Check 1: FAIL on hardcoded hex in `:style` binding or `<style>` block; PASS on `var(--token)` or utility class
- Check 2: FAIL on `font-family` in component `<style>` block
- Check 4: pages use layout via `<NuxtLayout>` or Vue Router — no layout chrome in page components
- Check 6: `:style="{ color: '#hex' }"` is FAIL; `:style="{ color: 'var(--color-primary)' }"` is PASS
- `<style scoped>` is allowed; check that properties use CSS variables

---

## Template Checks

For Python HTML templates (Jinja/Django templates):
- Check for inline `style=""` attributes containing hardcoded color/font values
- CSS rules in `<style>` blocks must use CSS variables from the design system
- Skip all Tailwind-specific checks

---

## Quick Reference

| Usage | Allowed | Blocked |
|-------|---------|---------|
| Colors | `text-primary`, `bg-surface`, `var(--color-error)` | `#3B82F6`, `rgb(59 130 246)` |
| Fonts | `font-sans`, `font-heading`, `font-mono` | `font-['Inter']`, `style={{ fontFamily: 'Inter' }}` |
| Spacing | `p-4`, `mt-8`, `gap-6` | `p-[13px]`, `mt-[1.3rem]` |
| Layout chrome | In `layout.tsx` files only | In page files |
| CSS variables | Defined in DESIGN_SYSTEM.md only | Undefined variables |
| Style objects | Dynamic values only | Static design values |
