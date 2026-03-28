---
name: vibe-coding-re-scan
description: |
  Fast initial codebase scan for reverse engineering. Detects framework, language, database,
  and rough complexity in under 60 seconds. Produces a confidence-scored summary.
  This is Stage 1 of 3 in the reverse-engineering pipeline.
  Use when: (1) vibe-coding-reverse-engineer initiates and needs initial scan,
  (2) User says "scan my codebase", "what stack is this?", "quick analysis of this project",
  (3) Before deep analysis to scope what vibe-coding-re-analyze needs to do.
  After scan, always hands off to vibe-coding-re-analyze unless user says stop.
---

# Vibe Coding — Codebase Scan (RE Stage 1)

Fast pattern detection only — no deep file reading. Goal: understand the stack in 60 seconds.

## Scan Steps

Run all 7 checks. Save results to progress.txt BEFORE displaying output.

**Step 1 — File count:** Count `.ts/.tsx`, `.js/.jsx`, `.py`, other files. Exclude `node_modules/`, `.git/`, `dist/`, `build/`. Identify key dirs: `src/app/`, `api/routes/controllers/`, `prisma/migrations/`, `public/static/`, `tests/__tests__/spec/`.

**Step 2 — Framework detection** (check in order):
- Next.js: `next.config.js` or `next.config.ts`
- Vite+React: `vite.config.ts` + `src/App.tsx`
- Vue: `vue.config.js` or `@vue/cli` in package.json
- Django: `manage.py` + `settings.py`
- FastAPI: `main.py` + `fastapi` in requirements.txt
- Express: `express` in package.json + no React
- Rails: `Gemfile` with `rails` | Laravel: `artisan` | Go: `go.mod` | Rust: `Cargo.toml`

**Step 3 — Database detection:** `prisma/schema.prisma` → Prisma, `drizzle.config.ts` → Drizzle, `mongoose` → MongoDB, `alembic/` → SQLAlchemy, `@supabase/supabase-js` → Supabase, `firebase` → Firebase.

**Step 4 — CSS detection:** `tailwind.config.*` → Tailwind, `*.module.css` → CSS Modules, `styled-components`/`@emotion` → CSS-in-JS, `*.scss` → SCSS.

**Step 5 — Auth detection:** `next-auth` → NextAuth, `@clerk/` → Clerk, `@supabase/auth` → Supabase Auth, `passport` → Passport.js, `jsonwebtoken` → Custom JWT.

**Step 6 — Deploy detection:** `vercel.json` → Vercel, `fly.toml` → Fly.io, `Dockerfile` → Docker, `netlify.toml` → Netlify, `.github/workflows/` → GitHub Actions.

**Step 7 — Complexity:** <30 files=Small, 30-100=Medium, 100-300=Large, >300=Very Large.

---

## Scan Output

```
CODEBASE SCAN COMPLETE
======================
Project: [folder name]
Complexity: [level] ([N] files)

STACK DETECTED:
  Framework: [name + version] (confidence: High/Medium/Low)
  Language: [TypeScript | JavaScript | Python | etc.]
  CSS: [approach]
  Database: [name + ORM]
  Auth: [library | none detected]
  Deployment: [config detected | unknown]

KEY DIRECTORIES: [list with purpose]

CONFIDENCE SUMMARY:
  Framework: [level]  Database: [level]  Auth: [level]  Overall: [level]

LOW CONFIDENCE: [areas needing deep analysis]

READY FOR DEEP ANALYSIS → activating vibe-coding-re-analyze
```

---

## Save to progress.txt

```
PHASE: REVERSE-ENGINEER
STATUS: scan_complete
framework: [detected]
language: [detected]
css: [detected]
database: [detected]
auth: [detected]
deployment: [detected]
file_count: [N]
complexity: [level]
low_confidence_areas: [list]
```

Then activate `vibe-coding-re-analyze`.
