---
name: vibe-coding-doc-frontend
description: |
  Generate FRONTEND_GUIDELINES.md — the engineering rules for building the frontend.
  Covers project structure, naming conventions, component architecture, layout template rules,
  state management, data fetching, form handling, responsive design, accessibility, testing, and performance.
  Use when: (1) vibe-coding-document delegates doc #6 (FRONTEND_GUIDELINES),
  (2) User says "generate frontend guidelines", "create frontend rules", "document component patterns".
  Reads TECH_STACK.md, DESIGN_SYSTEM.md, and APP_FLOW.md to tailor patterns to the actual stack.
---

# Vibe Coding — Frontend Guidelines Generator

Generate FRONTEND_GUIDELINES.md. Engineering rulebook for building all frontend code consistently.

## Prerequisites Check

1. Read `./docs/TECH_STACK.md` — framework, CSS approach, state library, data fetching library
2. Read `./docs/DESIGN_SYSTEM.md` — token names for examples
3. Read `./docs/APP_FLOW.md` — screens to build
4. Read `progress.txt` — project type for layout rules
5. Detect framework → read only the matching project structure section:

```
Next.js App Router   → READ: ## Structure: Next.js App Router
Next.js Pages Router → READ: ## Structure: Next.js Pages Router
Vite + React         → READ: ## Structure: Vite + React
Remix                → READ: ## Structure: Remix
SvelteKit            → READ: ## Structure: SvelteKit
Astro                → READ: ## Structure: Astro
Django + templates   → READ: ## Structure: Django Templates
Other/unlisted       → READ: ## Structure: Generic
```

6. DESIGN_SYSTEM.md missing → proceed without token examples; add note: "DESIGN_SYSTEM.md not found — update token references after generating it."

---

## Generation Steps

1. Write Overview (framework, styling, state, data, forms, auth)
2. Insert project structure (from matching Structure section)
3. Write Layout Template Rules (from project type in progress.txt)
4. Write Naming Conventions, Component Architecture, Component Template
5. Write State Management (adapted to actual state library)
6. Write Data Fetching (adapted to actual library)
7. Write Form Handling, Responsive Design, Accessibility, Testing, Performance
8. Save to `./docs/FRONTEND_GUIDELINES.md`
9. Show first 20 lines as preview
10. Run Validation Checklist
11. Ask: "FRONTEND_GUIDELINES generated. Approve or request revisions?"

## Core Rules (All Stacks)

**Layout:**
- Pages output content only — never re-declare header/footer/nav in a page file
- Layout chrome lives in `components/layouts/` exclusively
- New pages extend the correct layout group

**Components:**
- One component per file, PascalCase name
- Props interface above component named `ComponentNameProps`
- Always accept `className` prop, merge with `cn()`
- No inline styles, no hardcoded hex colors or font names
- No layout chrome inside feature or UI components

**State (adapt to actual library):**
- Local: useState for UI toggles scoped to one component
- Global: [chosen library] for user session, cart, theme, notifications
- Server: [chosen data library] for all API data — never useState for fetched data

**Accessibility (required for every interactive component):**
- All images have alt text
- Keyboard accessible interactions
- Focus rings visible (uses :focus-visible from globals.css)
- Color contrast meets WCAG AA (4.5:1)
- Form inputs have labels + aria-describedby for errors

**Performance:**
- Lazy-load routes with dynamic imports
- Use next/image or equivalent — never bare `<img>` for content
- Loading skeletons for content areas, not spinners

## Generation Rules

1. All code examples use actual token names from DESIGN_SYSTEM.md
2. Library patterns match exact libraries in TECH_STACK.md
3. Project structure matches the framework (use correct structure section)
4. State management section uses the actual chosen library

## Validation Checklist

```
FRONTEND_GUIDELINES VALIDATION:
✓/✗ Project structure matches framework (app router / pages router / Vite)
✓/✗ Layout template rules reference correct layout files
✓/✗ Code examples use real design token names
✓/✗ State section uses actual library from TECH_STACK.md
✓/✗ Data fetching section uses actual library from TECH_STACK.md
✓/✗ Accessibility checklist present
✓/✗ No generic placeholder library names
```

---

## Structure: Next.js App Router

```
src/
├── app/
│   ├── (auth)/                 # Auth-required route group
│   │   ├── layout.tsx          # AppLayout with sidebar/topbar
│   │   └── dashboard/page.tsx
│   ├── (public)/               # Public route group
│   │   ├── layout.tsx          # PublicLayout (header + footer)
│   │   └── page.tsx
│   ├── layout.tsx              # Root layout — fonts + globals.css
│   └── globals.css
├── components/
│   ├── layouts/                # Header, Footer, Sidebar, TopBar, AppLayout
│   ├── ui/                     # Button, Input, Card, Modal, Toast
│   ├── features/               # Feature-specific: ProductCard, UserProfile
│   └── sections/               # Full-width page sections (landing pages)
├── lib/
│   ├── api/                    # API client functions
│   ├── hooks/                  # Custom hooks (useAuth, useCart)
│   ├── utils/                  # cn(), formatDate()
│   └── validations/            # Zod schemas
├── stores/                     # Zustand stores
├── types/                      # TypeScript types
└── styles/globals.css
```

---

## Structure: Next.js Pages Router

```
src/
├── pages/
│   ├── _app.tsx                # Global layout + globals.css import
│   ├── _document.tsx
│   ├── api/                    # API routes
│   └── [routes]/               # Page components
├── components/
│   ├── layouts/
│   ├── ui/
│   ├── features/
│   └── sections/
├── lib/
├── stores/
├── types/
└── styles/globals.css
```

---

## Structure: Vite + React

```
src/
├── components/
│   ├── layouts/                # Header, Footer, Sidebar, AppShell
│   ├── ui/                     # Button, Input, Card, Modal
│   ├── features/               # Domain-specific components
│   └── sections/               # Full-width landing page sections
├── pages/                      # Route-level components
├── lib/
│   ├── api/
│   ├── hooks/
│   └── utils/
├── stores/
├── types/
└── styles/globals.css
```

---

## Structure: Remix

```
app/
├── routes/                     # File-based routes (route.tsx)
│   ├── _index.tsx              # / route
│   ├── _auth.tsx               # Auth layout
│   └── dashboard._index.tsx    # Nested route
├── components/
│   ├── layouts/
│   ├── ui/
│   └── features/
├── lib/
│   ├── session.server.ts       # Cookie/session utils
│   ├── api/
│   └── utils/
└── styles/globals.css
```

---

## Structure: SvelteKit

```
src/
├── routes/                     # File-based routing
│   ├── +layout.svelte          # Root layout
│   ├── +page.svelte            # / route
│   └── [route]/
│       ├── +page.svelte        # Page component
│       └── +page.server.ts     # Server load function
├── lib/
│   ├── components/
│   │   ├── layouts/
│   │   └── ui/
│   ├── stores/                 # Svelte writable stores
│   └── utils/
└── app.css                     # Global styles
```

Note: SvelteKit uses `class=` not `className=`. Components are `.svelte` files, not `.tsx`.

---

## Structure: Astro

```
src/
├── pages/                      # File-based routing (.astro, .md, .tsx)
│   ├── index.astro
│   └── [route].astro
├── layouts/
│   └── BaseLayout.astro        # Shared HTML shell
├── components/
│   ├── ui/                     # .astro or React/Svelte islands
│   └── sections/
├── styles/
│   └── globals.css
└── content/                    # Content collections (if used)
```

Note: Astro uses `.astro` syntax for templates. React components run as islands with `client:load`.

---

## Structure: Django Templates

```
[project]/
├── templates/
│   ├── base.html               # Base template with blocks
│   ├── layouts/                # Partials (header.html, footer.html)
│   └── [app]/                  # App-specific templates
├── static/
│   ├── css/globals.css
│   └── js/
├── [app]/
│   ├── views.py
│   └── urls.py
└── manage.py
```

Note: Django templates use `{% block %}` and `{% extends %}`. No JSX or React component patterns.

---

## Structure: Generic

```
src/                            # Or project root, depending on framework
├── components/                 # Reusable UI elements
├── layouts/                    # Shared page shells
├── pages/ (or views/)          # Route-level templates
├── styles/                     # CSS/SCSS files
└── [framework-specific]/       # Follow framework conventions
```

Note: Adapt to actual framework conventions. Check official documentation for correct directory names.
