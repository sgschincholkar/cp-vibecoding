---
name: vibe-coding-css-setup
description: |
  One-time project CSS foundation setup. Generates globals.css, verifies tailwind.config.js content paths,
  updates root layout to import fonts and globals.css, and ensures Tailwind directives are in place.
  Runs ONCE per project at the start of the BUILD phase before any component is written.
  Use when: (1) vibe-coding-build starts and css_setup_complete not in progress.txt,
  (2) User says "set up CSS", "fix my CSS setup", "Tailwind isn't working",
  (3) Fonts are not loading or typography looks wrong.
---

# Vibe Coding — CSS Setup

Run ONCE at BUILD start. Establishes the CSS foundation all components depend on.

## Entry Check

Read `progress.txt` — if `css_setup_complete: true` exists → **STOP. Skip this skill entirely.**

## Prerequisites

1. Read `./docs/DESIGN_SYSTEM.md` — CSS variable tokens and font names
2. Read `./docs/TECH_STACK.md` — framework and CSS approach
   If TECH_STACK.md missing → auto-detect CSS approach:
   - Check `tailwind.config.js` or `tailwind.config.ts` exists → Tailwind
   - Check `package.json` for "tailwindcss" → Tailwind
   - Check `vite.config.*` for CSS Modules → CSS Modules
   - If ambiguous → ask user: "Is this project using Tailwind CSS, plain CSS, or CSS Modules?"

Detect CSS approach and Tailwind version:
```
Tailwind CSS v3        → Run all 6 steps (full setup) using @tailwind directives
Tailwind CSS v4        → READ: ## Tailwind v4 Setup (uses @import, no tailwind.config.js)
Plain CSS / CSS Modules / SCSS → READ: ## Non-Tailwind Setup
```

Detect Tailwind version:
- `package.json` has `"tailwindcss": "^4.*"` or `">=4"` → v4
- `globals.css` contains `@import "tailwindcss"` → v4
- Otherwise → v3

**Non-Tailwind: jump directly to ## Non-Tailwind Setup and skip all Tailwind steps.**

---

## Setup Steps (Tailwind)

### Step 1 — Locate or create globals.css

Common paths: `src/styles/globals.css` | `src/app/globals.css` | `src/index.css` | `styles/globals.css`

If exists: check for 3 required sections (Tailwind directives, CSS variables, @layer base). Add missing ones.
If not exists: create with this content:

```css
/* FONT IMPORTS — only here, never in components */
/* [Font @import or handled via next/font in layout.tsx] */

@tailwind base;
@tailwind components;
@tailwind utilities;

:root {
  /* [All CSS variable tokens from DESIGN_SYSTEM.md] */
}

.dark {
  /* [Dark mode overrides from DESIGN_SYSTEM.md] */
}

@layer base {
  html {
    font-family: var(--font-sans);
    color: var(--color-text);
    background-color: var(--color-background);
    -webkit-font-smoothing: antialiased;
  }
  body { min-height: 100vh; }
  h1 { font-family: var(--font-heading, var(--font-sans)); font-size: 2.25rem; line-height: 2.5rem; font-weight: 700; }
  h2 { font-family: var(--font-heading, var(--font-sans)); font-size: 1.875rem; line-height: 2.25rem; font-weight: 700; }
  h3 { font-family: var(--font-heading, var(--font-sans)); font-size: 1.5rem; line-height: 2rem; font-weight: 600; }
  h4 { font-size: 1.25rem; line-height: 1.75rem; font-weight: 600; }
  h5 { font-size: 1.125rem; line-height: 1.75rem; font-weight: 600; }
  h6 { font-size: 1rem; line-height: 1.5rem; font-weight: 600; }
  a { color: var(--color-primary); }
  a:hover { color: var(--color-primary-hover); }
  :focus-visible { outline: 2px solid var(--color-primary); outline-offset: 2px; }
  @media (prefers-reduced-motion: reduce) {
    *, *::before, *::after { animation-duration: 0.01ms !important; transition-duration: 0.01ms !important; }
  }
}
```

### Step 2 — Verify tailwind.config.js

Check `content` paths cover all files with Tailwind classes:
```js
content: [
  './src/**/*.{js,ts,jsx,tsx,mdx}',
  './app/**/*.{js,ts,jsx,tsx,mdx}',
  './pages/**/*.{js,ts,jsx,tsx,mdx}',
  './components/**/*.{js,ts,jsx,tsx,mdx}',
]
```
Check `theme.extend` has design tokens from DESIGN_SYSTEM.md.
If tailwind.config.js missing: create with correct paths + full theme.extend.

### Step 3 — Verify root layout imports globals.css

- Next.js app router (`app/layout.tsx`): `import '@/styles/globals.css'`
- Next.js pages router (`pages/_app.tsx`): `import '@/styles/globals.css'`
- Vite (`src/main.tsx`): `import './styles/globals.css'`

If import missing: add it. Do not add it anywhere else.

### Step 4 — Verify font setup

Detect framework and apply correct font import method:

**Next.js (preferred — next/font):**
```tsx
// app/layout.tsx
import { [BodyFont], [HeadingFont] } from 'next/font/google'
const fontSans = [BodyFont]({ subsets: ['latin'], variable: '--font-sans', display: 'swap' })
const fontHeading = [HeadingFont]({ subsets: ['latin'], variable: '--font-heading', display: 'swap' })
// html element: className={`${fontSans.variable} ${fontHeading.variable}`}
```
If body and heading are the same font, use only one import.

**Vite / non-Next.js:** Add `@import url('https://fonts.googleapis.com/css2?family=[FontName]:wght@400;500;600;700&display=swap');` at top of globals.css.

Rule: fonts imported ONCE in root layout or globals.css. Never inside a component.

### Step 5 — Write CSS variables to globals.css

Copy the full CSS variables block from DESIGN_SYSTEM.md into the `:root {}` and `.dark {}` sections.

### Step 6 — Mark complete

Write to `progress.txt`:
```
css_setup_complete: true
css_setup_date: [timestamp]
globals_css_path: [actual path]
```

Show completion report:
```
CSS FOUNDATION SETUP COMPLETE

Files: [globals.css path] | [tailwind.config.js] | [root layout] | Font: [method]
CSS variable count: [N] tokens  |  Dark mode: [enabled/disabled]  |  Font(s): [names]

READY: All components can now use Tailwind tokens and CSS variables.
```

---

## Tailwind v4 Setup

Tailwind v4 uses `@import "tailwindcss"` in CSS — no `tailwind.config.js`, no `@tailwind` directives.

### Step 1 — Locate or create globals.css

If exists: check for `@import "tailwindcss"` and CSS variables block. Add missing parts.
If not exists: create with this content:

```css
@import "tailwindcss";

/* FONT IMPORTS — only here, never in components */
/* [Font @import or handled via next/font in layout.tsx] */

:root {
  /* [All CSS variable tokens from DESIGN_SYSTEM.md] */
}

.dark {
  /* [Dark mode overrides] */
}
```

### Step 2 — No tailwind.config.js needed

Tailwind v4 auto-detects content paths. If custom theme tokens are needed, use `@theme {}` block inside CSS instead:
```css
@theme {
  --color-primary: [hex];
  --font-sans: 'Inter', sans-serif;
}
```

### Step 3-6 — Same as Tailwind v3

Follow steps 3-6 from the main setup (verify root layout imports globals.css, font setup, write CSS variables, mark complete). Skip tailwind.config.js steps.

---

## Non-Tailwind Setup

For plain CSS / CSS Modules / SCSS projects:

1. Skip Tailwind directives (`@tailwind base/components/utilities`) and tailwind.config.js
2. Create globals.css with only `:root {}` CSS variables and base typography using standard CSS (no `@layer`)
3. Import fonts via `@import` at top of globals.css
4. Verify root file imports globals.css
5. Mark `css_setup_complete: true` in progress.txt

---

## Common Issues

| Issue | Fix |
|-------|-----|
| Tailwind classes not generating | Check `content` paths cover all src files |
| Custom colors not showing | Check `theme.extend.colors` in tailwind.config.js |
| Wrong font rendering | Check root layout has font variable class on `<html>` |
| Dark mode not working | Check `darkMode: 'class'` in tailwind.config.js |
| CSS variables undefined | Check globals.css imported in root layout |
| h1/h2 wrong size | Check @layer base in globals.css has h1-h6 styles |
