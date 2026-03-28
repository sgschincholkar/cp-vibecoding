---
name: vibe-coding-ui-review
description: |
  Screenshot-based UI analysis using ClearShot methodology. Analyzes UI screenshots at three levels:
  Map (spatial layout), System (design patterns), and Blueprint (layout architecture).
  Gives precise measurements, color values, spacing, and component identification.
  Identifies design token violations against DESIGN_SYSTEM.md.
  Use when: (1) User shares a screenshot during BUILD phase for UI feedback,
  (2) User says "check this design", "does this match the design system", "review this screenshot",
  (3) A UI component has been generated and user wants visual verification,
  (4) vibe-coding-debug is investigating a visual/layout bug.
  Requires an image to be provided. Returns structured analysis with specific actionable fixes.
---

# Vibe Coding — UI Review (Screenshot Analysis)

Analyze UI screenshots with precision. Identify violations and give exact fixes.

## Entry Check

```
No image provided?
→ READ: ## No Screenshot

Image provided → run all 3 levels below, then READ: ## Design System Comparison
```

**Read only the level sections you need. Skip to ## Design System Comparison and ## Fix Suggestions after running the levels — do not read all sections upfront.**

---

## Level 1 — MAP (Spatial Analysis)

Read the screenshot like a grid. Identify layout zones, element positions, spacing.

```
MAP ANALYSIS:
  Layout: [single column | two column | sidebar + main | grid]
  Header height: ~[Npx]
  Content max-width: ~[Npx] (centered? Y/N)
  Page padding: ~[Npx] left/right
  Primary CTA: [position description, visible Y/N]
  Mobile consideration: [how this looks on 375px wide]
```

---

## Level 2 — SYSTEM (Design Pattern Analysis)

Check colors, typography, spacing, components against the design system.

```
SYSTEM ANALYSIS:
  Colors:
    - Background: looks like #XXXXXX — [matches token | MISMATCH]
    - Primary: looks like #XXXXXX — [matches token | MISMATCH]
    - [any violations listed]
  Typography:
    - Heading font: [correct | MISMATCH]
    - Size hierarchy: [correct | ISSUE]
  Spacing: [consistent | ISSUE: describe]
  Components seen: [list]
  Pattern issues: [list]
```

---

## Level 3 — BLUEPRINT (Architecture Analysis)

Evaluate layout architecture, component reuse, responsive readiness, accessibility.

```
BLUEPRINT ANALYSIS:
  Layout template: [correct | ISSUE: page re-declares its own header]
  Component reuse: [consistent | ISSUE: describe]
  Responsive: [good | CONCERNS: describe what breaks on mobile]
  Accessibility: [good | CONCERNS: list]
```

Touch targets: flag any below 44px. Horizontal scroll risk: flag if max-width exceeds viewport.

---

## Design System Comparison

Check if `./docs/DESIGN_SYSTEM.md` exists:
- Missing: output "Design system file not found — comparison skipped. Run vibe-coding-document to generate docs."
- Exists: read and extract token names/hex values for comparison.

Check TECH_STACK.md for CSS approach and template language before writing fixes:
- Tailwind + React/Next.js: provide className fixes (`className="text-primary"`)
- Tailwind + Vue/Svelte: provide class fixes (`class="text-primary"`)
- Tailwind + Django/Jinja: provide class fixes (`class="text-primary"`)
- Plain CSS/SCSS: provide CSS property fixes (`color: var(--color-primary)`)
- Do NOT write JSX syntax (className=) for Vue, Svelte, or Django templates

```
DESIGN SYSTEM COMPARISON:

MATCHES:
  - [correctly uses design tokens]

VIOLATIONS:
  1. Color: [element] uses [hex] instead of [correct token]
     Fix: className="bg-primary"
  2. Typography: [element] wrong font
     Fix: ensure font-sans class, check root layout font variable
  3. Spacing: [element] ~13px — use gap-3 (12px) or gap-4 (16px)
```

---

## Final Report

```
UI REVIEW REPORT

SCORE: [N]/10 design token compliance

TOP 3 ISSUES:
  1. [Most impactful + fix]
  2. [Second + fix]
  3. [Third + fix]

WORKING WELL:
  - [positive observations]

NEXT STEP: [fix violations | looks good, proceed | investigate in vibe-coding-debug]
```

---

## No Screenshot

```
UI REVIEW: No screenshot provided.

To analyze your UI:
  1. Take a screenshot of the component or page
  2. Share it in the next message
  3. I will run all 3 analysis levels

Or describe what you're seeing and I can provide targeted guidance.
```
