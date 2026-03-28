---
name: vibe-coding-doc-implplan
description: |
  Generate IMPLEMENTATION_PLAN.md — the step-by-step build roadmap.
  Breaks the entire project into sequential phases and steps, each small enough to implement
  and test in one session. Ordered to build foundation first, features on top.
  Use when: (1) vibe-coding-document delegates doc #7 (IMPLEMENTATION_PLAN),
  (2) User says "generate implementation plan", "create the build plan", "write IMPLEMENTATION_PLAN.md".
  Reads all 6 preceding docs to derive a complete, conflict-free build order.
  Outputs IMPLEMENTATION_PLAN.md to ./docs/.
---

# Vibe Coding — Implementation Plan Generator

Generate IMPLEMENTATION_PLAN.md. The ordered build roadmap that vibe-coding-build follows step by step.

## Prerequisites Check

1. Read `./docs/PRD.md` — features to build
2. Read `./docs/TECH_STACK.md` — setup and tooling steps
3. Read `run_target` from `progress.txt` — determines final phase title and steps
4. Read `./docs/BACKEND_STRUCTURE.md` — data models and API endpoints
5. Read `./docs/FRONTEND_GUIDELINES.md` — component structure
6. Read `./docs/APP_FLOW.md` — screens and dependencies

Then detect stack and run_target:
```
Stack detection → READ: ## Phase 0: [matching stack]
  Next.js         → ## Phase 0: Next.js
  FastAPI         → ## Phase 0: FastAPI
  Flask           → ## Phase 0: Flask
  Django          → ## Phase 0: Django
  Vite + React    → ## Phase 0: Vite + React
  Electron        → ## Phase 0: Electron
  Tauri           → ## Phase 0: Tauri
  Go/Ruby/Rust/PHP → ## Phase 0: Generic Backend

has_frontend: false in progress.txt → omit Phase 3 (Shared Layout + UI Kit) entirely
Auth in PRD?    → Phase 2 present | READ: ## Phase 2: No Auth
run_target      → READ: ## Final Step: [matching target]
```

---

## Planning Principles

1. Foundation before features — setup, DB, auth, layout before feature screens
2. One testable outcome per step — each step ends with something verifiable
3. Dependencies drive order — Feature B needs Feature A's data → A comes first
4. Backend before frontend — API endpoint before the screen that uses it
5. Shared components before pages — layout and UI kit before feature pages

---

## Phase Structure

```
Phase 0: Project Setup        → READ: ## Phase 0: [stack]
Phase 1: Database Setup       (Prisma/SQLAlchemy/Drizzle schema + migrations)
Phase 2: Authentication       → READ: ## Phase 2: Auth OR ## Phase 2: No Auth
Phase 3: Shared Layout + UI Kit
Phase 4+: Features (one phase per must-have from PRD, in dependency order)
Final Phase: Integration, Polish, and Ship → READ: ## Final Step: [run_target]
```

---

## Generation Steps

1. Phase 0 setup from matching stack section
2. Phase 1 DB + migrations
3. Phase 2 auth (or skip)
4. Phase 3 layout + UI kit
5. One phase per must-have PRD feature (backend endpoint → hook → component → page → design-guard)
6. Final integration + polish + ship step
7. Save to `./docs/IMPLEMENTATION_PLAN.md`
8. Show first 20 lines as preview
9. Run Validation Checklist
10. Ask: "IMPLEMENTATION_PLAN generated. Approve or request revisions?"

## Generation Rules

1. Every must-have PRD feature gets its own phase
2. Each step has a concrete verify/test outcome
3. css-setup is always Step 0.2 (for frontend projects only — skip if has_frontend: false)
4. design-templates is always Step 3.1 (for frontend projects only — skip if has_frontend: false)
4a. Phase 3 (Shared Layout + UI Kit) is omitted entirely if has_frontend: false
5. design-guard is called at the end of each feature phase
6. Should-have and nice-to-have features are NOT in the plan
7. Final phase uses run_target — never hardcode "Deploy to Vercel" when run_target is local

## Validation Checklist

```
IMPLEMENTATION_PLAN VALIDATION:
✓/✗ Every must-have PRD feature has its own phase
✓/✗ Phase 0 setup commands match actual stack
✓/✗ Phase 2 present if auth required, skipped if not
✓/✗ Every step has a verify/test outcome
✓/✗ css-setup and design-templates steps present (frontend projects only)
✓/✗ Phase 3 omitted for backend-only projects (has_frontend: false)
✓/✗ design-guard called after each feature phase
✓/✗ No circular dependencies
✓/✗ Final phase uses correct run_target variant
```

---

## Phase 0: Next.js

```
Step 0.1 — Initialize project
- [ ] npx create-next-app@latest [app-name] --typescript --tailwind --app
- [ ] Install deps from TECH_STACK.md
- [ ] Verify: dev server runs at localhost:3000

Step 0.2 — CSS and design tokens
- [ ] Run vibe-coding-css-setup → generates globals.css and tailwind.config.js
- [ ] Verify: design tokens accessible in dev tools

Step 0.3 — ESLint, Prettier, env vars
- [ ] Create .eslintrc.json, .prettierrc
- [ ] Create .env.local from TECH_STACK.md env vars
- [ ] Verify: npm run lint passes
```

---

## Phase 0: FastAPI

```
Step 0.1 — Initialize project
- [ ] python -m venv .venv && source .venv/bin/activate (.venv\Scripts\activate on Windows)
- [ ] pip install fastapi uvicorn sqlalchemy alembic pydantic python-dotenv
- [ ] pip freeze > requirements.txt
- [ ] Verify: uvicorn main:app --reload starts at localhost:8000
```

---

## Phase 0: Flask

```
Step 0.1 — Initialize project
- [ ] python -m venv .venv && source .venv/bin/activate
- [ ] pip install flask flask-sqlalchemy flask-migrate python-dotenv
- [ ] pip freeze > requirements.txt
- [ ] Verify: flask run starts at localhost:5000
```

---

## Phase 0: Django

```
Step 0.1 — Initialize project
- [ ] python -m venv .venv && source .venv/bin/activate
- [ ] pip install django djangorestframework python-dotenv
- [ ] django-admin startproject [name] .
- [ ] Verify: python manage.py runserver starts at localhost:8000
```

---

## Phase 0: Vite + React

```
Step 0.1 — Initialize project
- [ ] npm create vite@latest [app-name] -- --template react-ts
- [ ] npm install + packages from TECH_STACK.md
- [ ] Verify: npm run dev starts at localhost:5173
```

---

## Phase 0: Electron

```
Step 0.1 — Initialize project
- [ ] npm create vite@latest [app-name] -- --template react-ts
- [ ] npm install --save-dev electron electron-builder concurrently
- [ ] Add electron/main.ts entry point
- [ ] Add package.json scripts: electron:dev, electron:build
- [ ] Verify: npm run electron:dev opens Electron window
```

---

## Phase 0: Tauri

```
Step 0.1 — Initialize project
- [ ] Prerequisites: Rust via rustup; xcode-select --install (macOS); VS Build Tools (Windows)
- [ ] npm create vite@latest [app-name] -- --template react-ts
- [ ] npm install --save-dev @tauri-apps/cli
- [ ] npx tauri init
- [ ] Verify: npm run tauri dev opens Tauri window
```

---

## Phase 0: Generic Backend

For Go, Ruby, Rust, PHP, or any unlisted language/framework:

```
Step 0.1 — Initialize project
- [ ] Install language toolchain (verify with [go version / ruby -v / rustc --version / php -v])
- [ ] Create project directory and initialize package manager ([go mod init / bundle init / cargo init / composer init])
- [ ] Install web framework from TECH_STACK.md
- [ ] Verify: dev server starts at [localhost:PORT]

Step 0.2 — Project structure and dependencies
- [ ] Install all packages from TECH_STACK.md
- [ ] Create .env from TECH_STACK.md env vars
- [ ] Verify: app compiles/starts cleanly with no errors
```

Note: "Setup commands for [language/framework] — verify against official documentation."

---

## Phase 2: Auth

```
Step 2.1 — Set up auth [use correct variant for stack]

Next.js: Configure [NextAuth | Clerk | Supabase Auth] with providers from TECH_STACK.md. Create auth API routes. Verify: can log in and out.

Django: Django has built-in auth — configure in settings.py. Install django-allauth if social login needed. Verify: /admin login works.

Flask: Install Flask-Login (pip install flask-login). Create User model + login/logout routes. Verify: session persists.

FastAPI: Install python-jose + passlib. Create JWT auth utilities in auth/dependencies.py. Verify: /auth/login returns valid JWT.

Desktop: Use OS keychain for credentials (keytar for Electron, Tauri credential store). Verify: user state persists between sessions.

Step 2.2 — Auth route protection
- [ ] Create auth-protected layout group
- [ ] Create public layout group
- [ ] Verify: unauthenticated user redirected to login
```

---

## Phase 2: No Auth

```
## Phase 2: Authentication
N/A — this project does not require user authentication.
```

---

## Final Step: Local

```
Step [N].5 — Ready to run locally
- [ ] All must-have features from PRD complete
- [ ] All user journeys in APP_FLOW work end to end
- [ ] App starts cleanly: [npm run dev / flask run / python manage.py runserver / uvicorn main:app --reload]
- [ ] Activate vibe-coding-local-runner for startup instructions
```

---

## Final Step: Desktop

```
Step [N].5 — Package desktop app
- [ ] All must-have features complete
- [ ] npm run [electron:dev / tauri dev] opens window correctly
- [ ] npm run [electron:build / tauri build] produces distributable
- [ ] Activate vibe-coding-local-runner (packaging mode)
```

---

## Final Step: Vercel

```
Step [N].5 — Deploy to Vercel
- [ ] All must-have features complete
- [ ] npm run build passes with no errors
- [ ] All env vars from TECH_STACK.md recorded for Vercel dashboard
- [ ] Activate vibe-coding-ship → deploys via vibe-coding-deploy-vercel
```

---

## Final Step: Netlify

```
Step [N].5 — Deploy to Netlify
- [ ] All must-have features complete
- [ ] npm run build passes, dist/out folder correct
- [ ] All env vars recorded for Netlify dashboard
- [ ] Activate vibe-coding-ship → deploys via vibe-coding-deploy-netlify
```

---

## Final Step: DigitalOcean

```
Step [N].5 — Deploy to DigitalOcean
- [ ] All must-have features complete
- [ ] Start command confirmed ([npm run build / gunicorn / uvicorn])
- [ ] Dockerfile or .do/app.yaml present if needed
- [ ] All env vars recorded for DO App Platform
- [ ] Activate vibe-coding-ship → deploys via vibe-coding-deploy-digitalocean
```

---

## Final Step: Other Cloud

```
Step [N].5 — Deploy to [platform from run_target_detail]
- [ ] All must-have features complete
- [ ] Build/start command confirmed for platform
- [ ] All env vars recorded
- [ ] Activate vibe-coding-ship → routes to correct deploy skill
```
