---
name: vibe-coding-web-design-guidelines
description: |
  Reviews UI/frontend code against Vercel's Web Interface Guidelines — fetched live from GitHub so rules stay current.
  Checks files for accessibility, visual hierarchy, interaction patterns, spacing, typography, and component API design.
  Use when: (1) vibe-coding-design-guard calls this after its 7 internal checks,
  (2) User says "check UI guidelines", "review against Vercel guidelines", "audit UI quality",
  (3) User says "review my UI", "check accessibility", "audit design", "check UX".
  Pass a file path or glob pattern. Returns findings in file:line format.
  Source: vercel-labs/agent-skills web-design-guidelines + vercel-labs/web-interface-guidelines.
---

# Vibe Coding — Web Design Guidelines

Review UI code against Vercel's Web Interface Guidelines.

## Entry Router

```
Fetch live guidelines first:
  URL: https://raw.githubusercontent.com/vercel-labs/web-interface-guidelines/main/command.md
  Fetch failed? → Note "Using cached guidelines (fetched [today's date])" and use fallback rules below.
  Fetch succeeded? → Note version/date from response if available; if no date available, note "Live fetch — no version date in response."

Then read target file(s) and run all 6 category checks.
Accessibility findings are always MUST FIX (highest severity).

Detect template language before writing fixes:
  React/Next.js → JSX fix syntax (className=, <button>, etc.)
  Vue/Nuxt      → Vue fix syntax (class=, <button>, v-bind, SFC template tags)
  Svelte        → Svelte fix syntax (class=, <button>, native HTML)
  Django/Jinja  → HTML fix syntax (class=, standard HTML attributes)
```

**Jump directly to the category relevant to the file being reviewed. Do not read all 6 categories before starting. Output only categories with findings — skip clean ones.**

---

## Accessibility (MUST FIX)

FAIL if:
- Interactive elements missing accessible labels: `<button><svg /></button>` → add `aria-label`
- Images missing `alt` (use `alt=""` for decorative images)
- Form inputs without associated `<label>` or `aria-labelledby`
- Color as only way to convey information (error needs icon/text too)
- Focus not visible: check `:focus-visible` in globals.css
- `tabIndex={-1}` on interactive elements without reason, modal not trapping focus
- `<div role="button">` — use actual `<button>`

---

## Visual Hierarchy (HIGH)

FAIL if:
- Multiple elements at same visual weight — each screen needs ONE primary CTA
- Heading levels skipped (h1 → h3, not h1 → h2 → h3)
- Text contrast below WCAG AA (4.5:1 normal, 3:1 large) — flag light gray on white
- Body text below 14px (0.875rem), line length with `max-w-none` on text blocks

---

## Spacing and Layout (MEDIUM)

FAIL if:
- Arbitrary spacing values (`padding: 13px`) — use design tokens (`p-3`, `var(--spacing-3)`)
- Content touching viewport edges on mobile — needs `px-4` or `px-safe`
- Touch targets below 44x44px — flag buttons with `py` less than `p-2` + small font
- `overflow-hidden` clipping content unexpectedly

---

## Interaction Patterns (MEDIUM)

FAIL if:
- No loading state on async operations (every async button needs loading feedback)
- Form errors shown only as color changes (need error message text)
- Destructive actions (delete/remove/clear) without confirmation step
- User actions with no visual feedback
- Hover states missing on interactive elements
- Disabled buttons with no explanation (need tooltip or nearby text)

---

## Typography (LOW)

FAIL if: 3+ font families in use, more than 4 font weights (use 400/500/600/700 only), `tracking-wide` on body paragraphs, ALL CAPS longer than 4 words, hardcoded `font-family` inline.

---

## Component API Design (LOW)

FAIL if: props accepting `any` where specific types possible, 3+ mutually exclusive boolean props (use `variant` enum instead), inconsistent prop naming across similar components, components accepting both `children` and a `content` prop.

---

## Output Format

```
WEB DESIGN GUIDELINES REVIEW
==============================
Files reviewed: [list]
Guidelines version: [live | cached fallback]

FINDINGS:

[file:line] [category] [severity] — [description]
  Found: [exact code]
  Fix:   [specific change]

Example:
src/components/Button.tsx:34 [accessibility] MUST FIX — Button missing aria-label
  Found: <button><CloseIcon /></button>
  Fix:   <button aria-label="Close"><CloseIcon /></button>

---
Total: [N] must-fix, [N] high, [N] medium, [N] low
Accessibility issues: [N]
```

**Integration:** Called by `vibe-coding-design-guard` after its 7 checks pass. Findings appended under "WEB GUIDELINES FINDINGS" in the design guard report. Also callable standalone.
