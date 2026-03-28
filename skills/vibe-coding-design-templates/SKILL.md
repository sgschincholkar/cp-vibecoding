---
name: vibe-coding-design-templates
description: |
  Site-type layout template selector and scaffolder. Identifies the project type and generates
  the base layout structure with shared components. All pages extend this base.
  Use when: (1) vibe-coding-build is about to scaffold first pages,
  (2) User says "set up page templates", "create base layout", "scaffold the pages",
  (3) After vibe-coding-css-setup completes, before first page component is built.
  Reads PRD.md and APP_FLOW.md to identify project type.
---

# Vibe Coding — Design Templates

Scaffold the base layout before any page content is built. Every page extends the base — built once, reused everywhere.

## Entry Check

Read `progress.txt`:
- If `templates_scaffolded: true` → **STOP. Skip entirely.**
- If `has_frontend: false` → **STOP. Backend-only project — no templates needed.**

## Step 1 — Detect Project Type

Read `./docs/PRD.md` and `./docs/APP_FLOW.md`. Match indicators:

| Type | Indicators |
|------|------------|
| Content / Blog | Articles, posts, categories, author pages, RSS |
| Ecommerce | Products, cart, checkout, orders, product listings |
| SaaS / App | User auth, dashboard, settings, subscription, features |
| Landing Page | Single page, hero, features, pricing, CTA, no app logic |
| Dashboard / Admin | Data tables, charts, stats, CRUD, analytics |
| Portfolio | Projects showcase, about, contact, case studies |

If 2+ indicators match one type clearly → use it automatically.
If unclear or tied → ask: "What type of site? Content/Blog | Ecommerce | SaaS/App | Landing | Dashboard | Portfolio"

**After type confirmed:** Read APP_FLOW.md Screen Inventory. Create one page shell for EVERY screen listed — not just defaults below.

**Framework note:** Templates below use Next.js App Router paths. For Vite: replace `app/layout.tsx` → `src/App.tsx`, replace `app/[route]/page.tsx` → `src/pages/[Route].tsx`, replace route groups with react-router-dom nested routes.
For Remix/SvelteKit/Astro/other: adapt route conventions to your framework — the layout structure and component list remain the same, only the file paths differ.

## Step 2 — Scaffold by Type

Then READ: ## [Type] Template

---

## Content / Blog Template

Layout: Header + main + Footer. Optional sidebar.

Files:
- `components/layouts/Header.tsx` — logo, nav, search, optional auth
- `components/layouts/Footer.tsx` — links, copyright
- `components/layouts/ContentLayout.tsx` — main + optional sidebar
- `components/layouts/Sidebar.tsx` — if sidebar in APP_FLOW
- `app/layout.tsx` — RootLayout, fonts, globals.css import

Page shells: `app/page.tsx` | `app/[slug]/page.tsx` | `app/category/[slug]/page.tsx` | `app/about/page.tsx`

---

## Ecommerce Template

Layout: Header (with cart icon) + optional CategoryNav + main + Footer + CartDrawer (slide-over).

Files:
- `components/layouts/Header.tsx` — logo, search, cart icon with count, user icon
- `components/layouts/CategoryNav.tsx` — horizontal category links
- `components/layouts/Footer.tsx` — links, payment icons, copyright
- `components/layouts/CartDrawer.tsx` — slide-over cart panel
- `components/ui/ProductCard.tsx` — shared product card shell
- `app/layout.tsx` — RootLayout

Page shells: `app/page.tsx` | `app/products/page.tsx` | `app/products/[slug]/page.tsx` | `app/cart/page.tsx` | `app/checkout/page.tsx` | `app/account/page.tsx`

---

## SaaS / App Template

Layout: Public routes (no sidebar) + Auth routes (Sidebar + TopBar + main).

Files:
- `components/layouts/AppLayout.tsx` — authenticated shell (sidebar + topbar + main)
- `components/layouts/Sidebar.tsx` — nav items, logo, user, collapse toggle
- `components/layouts/TopBar.tsx` — breadcrumb, page title, actions, user menu
- `components/layouts/PublicLayout.tsx` — for landing/auth pages (no sidebar)
- `components/ui/Modal.tsx` — shared modal
- `components/ui/Toast.tsx` — notification toast
- `app/layout.tsx` | `app/(auth)/layout.tsx` | `app/(public)/layout.tsx`

Page shells: `app/(public)/page.tsx` | `app/(public)/login/page.tsx` | `app/(public)/signup/page.tsx` | `app/(auth)/dashboard/page.tsx` | [additional from APP_FLOW]

---

## Landing Page Template

Layout: Sticky header with anchor nav + CTA + sections as components + Footer.

Files:
- `components/layouts/Header.tsx` — sticky with anchor nav + CTA button
- `components/layouts/Footer.tsx` — links, copyright
- `components/sections/HeroSection.tsx` — hero shell
- `components/sections/FeaturesSection.tsx` — features shell
- `components/sections/PricingSection.tsx` — if in APP_FLOW
- `components/sections/CTASection.tsx` — CTA shell
- `app/layout.tsx` | `app/page.tsx` — imports all sections

---

## Dashboard / Admin Template

Layout: Sidebar (collapsible) + content area (Header + main).

Files:
- `components/layouts/DashboardLayout.tsx` — sidebar + content area
- `components/layouts/Sidebar.tsx` — collapsible nav, logo, user
- `components/layouts/Header.tsx` — page title bar, actions
- `components/ui/StatCard.tsx` — shared stats card
- `components/ui/DataTable.tsx` — shared table with sort/filter shell
- `app/layout.tsx` | `app/dashboard/page.tsx` | [additional from APP_FLOW]

---

## Portfolio Template

Layout: Minimal header + main + footer.

Files:
- `components/layouts/Header.tsx` — minimal nav
- `components/layouts/Footer.tsx` — contact, social
- `components/ui/ProjectCard.tsx` — shared project card
- `app/layout.tsx` | `app/page.tsx` | `app/projects/[slug]/page.tsx` | `app/about/page.tsx` | `app/contact/page.tsx`

---

## Step 3 — Component Shell Rules

All generated shells follow this pattern:
```tsx
export const [ComponentName]: FC<{ className?: string }> = ({ className }) => {
  return (
    <[element] className={`[design tokens only] ${className ?? ''}`}>
      {/* content slot — empty shell */}
    </[element]>
  )
}
```

Rules:
- Design tokens only (`bg-background`, `border-border`, `z-sticky`)
- No hardcoded colors or hex values
- Always include `className` prop
- Mobile-first padding: `px-4 sm:px-6 lg:px-8`

---

## Step 4 — Mark Complete

Write to `progress.txt`:
```
templates_scaffolded: true
project_type: [type]
template_date: [timestamp]
layout_files: [list]
page_shells: [list]
```

Show report:
```
LAYOUT TEMPLATES SCAFFOLDED

Type: [type]  |  Base: [layout description]
Shared components: [list]
Page shells: [list]

RULE: Every new page must use the base layout — never re-declare layout in a page.
```
