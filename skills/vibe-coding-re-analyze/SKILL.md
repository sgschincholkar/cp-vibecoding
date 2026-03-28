---
name: vibe-coding-re-analyze
description: |
  Deep codebase analysis for reverse engineering. Reads source files to understand
  data models, API routes, component architecture, auth flow, business logic, and integrations.
  This is Stage 2 of 3 in the reverse-engineering pipeline. Runs after vibe-coding-re-scan.
  Use when: (1) vibe-coding-re-scan completes and hands off for deep analysis,
  (2) User says "analyze my codebase in detail", "deep dive the code",
  (3) Scan results have medium/low confidence areas that need code reading.
  Reads actual source files. Produces a verified analysis summary.
  Hands off to vibe-coding-re-generate after user confirms the analysis.
---

# Vibe Coding — Deep Codebase Analysis (RE Stage 2)

Read source files and build a verified understanding of 8 analysis areas.

## Prerequisites

Read `progress.txt` for scan results from `vibe-coding-re-scan`. Use `vibe-coding-explore` for all file reading — smart_outline files, smart_unfold key symbols only. Never read full code files.

## Analysis Areas

Work through all 8. For each: read relevant files, extract facts, score confidence (High/Medium/Low).

**Area 1 — Framework & Stack:** Read `package.json`/`requirements.txt`, framework config, `tsconfig.json`. Extract: exact versions, all major dependencies, TypeScript strict mode, build config.

**Area 2 — Database Schema:** Read `prisma/schema.prisma`, `drizzle/` schemas, `models/`, latest migration files. Extract: all models/tables, fields+types, relationships, indexes, enums.

**Area 3 — API Endpoints:** Read `app/api/`, `pages/api/`, `routes/`, `controllers/`. Extract: every route path, HTTP method, auth requirement, brief description, request/response shape.

**Area 4 — Authentication:** Detect framework then read matching files:
```
Next.js/React  → app/api/auth/, middleware.ts, auth config files, useAuth hooks
Vue/Nuxt       → composables/useAuth.ts, middleware/, plugins/auth.ts, Pinia auth store
Angular        → src/app/auth/, guards/, interceptors/, AuthService
Django         → urls.py auth paths, views.py login/logout, AUTHENTICATION_BACKENDS setting
FastAPI/Flask  → auth router/blueprint, dependency functions (Depends), JWT utilities
```
Extract: auth library, flow (email/password/OAuth/magic link), session storage, protected route mechanism, user roles.

**Area 5 — Frontend Component Tree:** Detect framework then read matching locations:
```
React/Next.js  → components/layouts/, components/ui/, components/features/, root layout
Vue/Nuxt       → components/, layouts/, composables/, Pinia stores/, app.vue
Angular        → src/app/components/, shared/, services/, app-routing.module.ts
SvelteKit      → src/lib/components/, src/routes/+layout.svelte, stores/
Django HTML    → templates/, (no JS component tree — map template inheritance instead)
```
Extract: shared layout structure, UI component inventory, state management approach, data flow.

**Area 6 — Business Logic:** Read `lib/`, `utils/`, `hooks/`, `services/`. Extract: core algorithms, business rules, data transformations, key custom hooks.

**Area 7 — Integrations:** Read `lib/[service].ts` files, env var references. Extract: all external services, initialization patterns, which features use them.

**Area 8 — CSS and Design System:** Read `globals.css`, `tailwind.config.*`, `variables.css`, sample 3-5 components. Extract: CSS approach, CSS variables in `:root`, color palette, typography, styling consistency.

---

## Confidence Levels

| Level | Meaning |
|-------|---------|
| High | Clearly documented or obvious from code |
| Medium | Inferred from patterns, may have gaps |
| Low | Minimal code, significant guessing |

---

## Verification Step

Show user analysis summary and wait for confirmation before proceeding:

```
DEEP ANALYSIS COMPLETE
======================
Framework: [name + version] — [confidence]
Database: [N models: list names] — [confidence]
API: [N endpoints] — [confidence]
Auth: [method + providers] — [confidence]
Frontend: [layout + N components] — [confidence]
Business logic: [key patterns] — [confidence]
Integrations: [list services] — [confidence]
Styling: [approach + token status] — [confidence]

LOW CONFIDENCE (may need your input): [list gaps]

Does this look correct? Any corrections before I generate the docs?
(Reply with corrections, or say "looks good" to continue)
```

---

## Save to progress.txt and hand off

```
PHASE: REVERSE-ENGINEER
STATUS: analysis_complete
framework: [exact + version]
database_models: [list]
api_endpoints: [count]
auth_method: [method]
components: [count]
integrations: [list]
css_approach: [approach]
has_design_tokens: [yes/no]
font_names: [list if found]
confidence_overall: [level]
gaps: [list needing Q&A]
```

Then activate `vibe-coding-re-generate`.
