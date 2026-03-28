---
name: vibe-coding-doc-design
description: |
  Generate DESIGN_SYSTEM.md — the complete visual language for a project.
  Covers colors (full 50-950 scale + 3 gradients), typography, spacing, shadows, breakpoints, animation, z-index, and component tokens.
  Produces ready-to-use CSS variables block, Tailwind config snippet, and globals.css with font inheritance.
  Use when: (1) vibe-coding-document delegates doc #4 (DESIGN_SYSTEM),
  (2) User says "generate design system", "create design tokens", "build the design system".
  Reads TECH_STACK.md to decide CSS approach. Reads progress.txt for brand colors and style choices.
  Outputs DESIGN_SYSTEM.md to ./docs/ AND globals.css scaffold to src/styles/.
---

# Vibe Coding — Design System Generator

Generate DESIGN_SYSTEM.md + globals.css. Single source of truth for all visual decisions.

## Prerequisites Check

1. Read `progress.txt` — brand colors, style direction, font preferences from IDEATE
2. Check for `UI_UX_SELECTIONS:` block in progress.txt → if present → READ: ## Use UI/UX Selections
3. Read `./docs/TECH_STACK.md` — CSS approach (Tailwind vs CSS Modules vs plain CSS vs SCSS)
4. Read `./docs/PRD.md` — app type (ecommerce, SaaS, dashboard, landing) — affects component tokens

---

## Generation Steps

1. Extract design inputs from progress.txt (colors, style, fonts, dark mode)
2. Generate complete color token tables (brand, primary palette 50-950, 3 gradients, neutral, semantic)
3. Build typography system (font stack, type scale, weights)
4. Build spacing, radius, shadow, breakpoint, animation, z-index scales
5. Build component tokens (buttons, inputs, cards)
6. Output complete CSS variables block (no #XXXXXX placeholders)
7. Detect CSS approach from TECH_STACK.md:
   - Tailwind in stack → output Tailwind config snippet AND globals.css with `@tailwind` or `@import "tailwindcss"` (v4)
   - NOT Tailwind → skip Tailwind config step entirely; globals.css uses plain CSS `:root {}` block only (no `@tailwind` directives)
8. Output globals.css scaffold
9. Save `./docs/DESIGN_SYSTEM.md`
10. Save globals.css to `./src/styles/globals.css` (or equivalent root CSS file)
11. Show first 20 lines as preview
12. Run Validation Checklist
13. Ask: "DESIGN_SYSTEM generated. Approve or request revisions?"

## Output Structure

The document includes these sections in order:
1. Overview (philosophy, style reference, default theme)
2. Color Tokens (brand, primary 50-950 scale, 3 gradients, neutral, semantic)
3. Typography (font stack, type scale)
4. Spacing Scale (4px base unit, space-1 through space-20)
5. Border Radius (none through full)
6. Shadows (sm through xl + inner)
7. Breakpoints (sm 640px through 2xl 1536px, mobile-first)
8. Animation (duration + easing tokens)
9. Z-Index Scale (dropdown through tooltip)
10. Component Tokens (buttons: variants + sizes, inputs: states + sizes, cards: variants)
11. CSS Variables block (complete :root {} + .dark {})
12. Tailwind config snippet (if applicable)
13. Color Usage Rules (enforcement block — BUILD must read this)

Then separately output globals.css scaffold.

## Generation Rules

1. All hex values are exact — from brand-colors interrogation or UI/UX selections. No #XXXXXX in final output.
2. Full primary palette: generate actual hex values for all 11 shades (50-950)
3. Dark mode: define light/dark pairs for all neutral tokens
4. globals.css always generated even if CSS variables are incomplete
5. Tailwind config uses CSS variable references, not hardcoded hex — dark mode needs only `.dark` class toggle
5a. If project does NOT use Tailwind: omit Tailwind config entirely from DESIGN_SYSTEM.md; globals.css is plain CSS only
6. Font family declared once in globals.css — never in component-level CSS
7. Include Color Usage Rules section in DESIGN_SYSTEM.md:
   ```
   ALWAYS: var(--color-primary) or className="text-primary"
   NEVER: #6366F1 or color: blue (hardcoded values)
   ```

## Validation Checklist

```
DESIGN_SYSTEM VALIDATION:
✓/✗ All brand color hex values filled in (no #XXXXXX)
✓/✗ Full primary palette scale (50-950) with real hex values
✓/✗ 3 gradient presets defined as CSS variables
✓/✗ Dark mode values for all neutral tokens
✓/✗ Typography has exact font names
✓/✗ globals.css generated and saved
✓/✗ Tailwind config uses CSS variable references (not hardcoded hex) [skip if non-Tailwind stack]
✓/✗ Color Usage Rules section included
✓/✗ No placeholder text remains
```

---

## Use UI/UX Selections

When `UI_UX_SELECTIONS:` block exists in progress.txt:
Extract palette, typography, and component style from that section.
These selections take priority over all other defaults.
Proceed with generation using these as the source for color and font decisions.
