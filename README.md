# Vibe Coding Skills for Claude

Build software from idea to shipped product through **49 specialized skills** with interactive conversation, complete state tracking, and local-first deployment.

---

## What is This?

Vibe Coding is a collection of Claude skills that guide you through the entire software development lifecycle. Each skill does one thing. Skills are loaded only when needed, keeping token cost low and instructions focused.

```
IDEA → IDEATE → DOCUMENT → BUILD → DEBUG → SHIP → LIVE
                                              ↑
                                   REVERSE-ENGINEER
                                   (from existing code)
```

**Key features:**
- 49 focused skills — 60-70% fewer tokens than a monolithic approach
- Local-first — `npm run dev`, `flask run`, Electron, Tauri, or cloud deploy
- Design system enforcement — CSS tokens, fonts, and layout rules are checked before every page is generated
- run_target wiring — you choose your deployment target once in IDEATE; every downstream skill adapts to it
- Self-improving — skills propose their own fixes after gaps are detected, with your approval before any change is written

---

## Quick Start

```bash
# Clone
git clone https://github.com/aimonk2025/cp-vibe-coding.git

# macOS / Linux
cp -r skills/* ~/.claude/skills/

# Windows
xcopy /E /I skills\* %USERPROFILE%\.claude\skills\
```

Then in Claude:
```
You: I want to build a habit tracking app
```
The orchestrator activates, detects you're starting from scratch, and routes to ideate.

---

## How the Skills Are Wired

### Full wiring diagram

```
User message
    |
    v
vibe-coding-orchestrator
    |
    |-- New project ---------> vibe-coding-ideate
    |                               |
    |                               |-- topic 10: design --> vibe-coding-ui-ux
    |                               |
    |                               v
    |                          vibe-coding-document
    |                               |
    |                               |-- doc 1 --> vibe-coding-doc-prd
    |                               |-- doc 2 --> vibe-coding-doc-appflow
    |                               |-- doc 3 --> vibe-coding-doc-techstack
    |                               |-- doc 4 --> vibe-coding-doc-design
    |                               |-- doc 5 --> vibe-coding-doc-backend
    |                               |-- doc 6 --> vibe-coding-doc-frontend
    |                               |-- doc 7 --> vibe-coding-doc-implplan
    |                               |-- doc 8 --> vibe-coding-doc-claudemd
    |                               v
    |                          vibe-coding-build
    |                               |
    |                               |-- project start --> vibe-coding-css-setup
    |                               |                 --> vibe-coding-design-templates
    |                               |
    |                               |-- each step -----> vibe-coding-design-guard
    |                               |                 --> vibe-coding-tdd
    |                               |                 --> vibe-coding-code-review
    |                               |                       |-- React/Next.js --> vibe-coding-review-react
    |                               |                       |-- React Native  --> vibe-coding-react-native
    |                               |
    |                               |-- external API  --> vibe-coding-api-connect
    |                               |-- CLI tool      --> vibe-coding-cli-runner
    |                               |-- MCP server    --> vibe-coding-mcp-setup
    |                               |-- database step --> vibe-coding-db
    |                               |                       |-- SQLite        --> vibe-coding-db-sqlite
    |                               |                       |-- BetterSQLite3 --> vibe-coding-db-bettersqlite
    |                               |                       |-- PostgreSQL    --> vibe-coding-db-postgres
    |                               |                       |-- DuckDB        --> vibe-coding-db-duckdb
    |                               |                       |-- Convex        --> vibe-coding-db-convex
    |                               |
    |                               |-- build error   --> vibe-coding-build-fix
    |                               v
    |                          vibe-coding-debug
    |                               |
    |                               |-- before fix    --> vibe-coding-impact-analysis
    |                               |-- UI bug        --> vibe-coding-ui-review
    |                               |-- build error   --> vibe-coding-build-fix
    |                               v
    |                          vibe-coding-ship
    |                               |
    |                               |-- pre-flight    --> vibe-coding-doc-review
    |                               |               --> vibe-coding-security-review
    |                               |
    |                               |-- run_target: local   --> vibe-coding-local-runner
    |                               |-- run_target: vercel  --> vibe-coding-deploy-vercel
    |                               |-- run_target: netlify --> vibe-coding-deploy-netlify
    |                               |-- run_target: do      --> vibe-coding-deploy-digitalocean
    |                               |-- run_target: desktop --> vibe-coding-local-runner
    |
    |-- Existing codebase ---> vibe-coding-reverse-engineer
    |                               |
    |                               |-- stage 1 --> vibe-coding-re-scan
    |                               |-- stage 2 --> vibe-coding-re-analyze
    |                               |-- stage 3 --> vibe-coding-re-generate
    |
    |-- "improve skill" -----> vibe-coding-self-improve
    |
    |-- State management ----> vibe-coding-state (read/write progress.txt)
    |-- Code navigation -----> vibe-coding-explore (AST-powered, requires claude-mem)
    |-- Session resume ------> vibe-coding-recall (cross-session memory, requires claude-mem)
```

### run_target — chosen once, used everywhere

During IDEATE topic 6, Claude asks:
```
Where will this app run?
1. Local machine (terminal)
2. Vercel
3. Netlify
4. DigitalOcean
5. Desktop app (Electron / Tauri)
6. Other cloud — specify
```

The answer is saved to `progress.txt` as `run_target`. Every downstream skill reads it:

| Skill | How it uses run_target |
|-------|----------------------|
| `vibe-coding-doc-techstack` | Deployment section shows correct cost + setup commands |
| `vibe-coding-doc-implplan` | Final phase title and steps match target (Run locally / Deploy to Vercel / etc.) |
| `vibe-coding-doc-claudemd` | Start command in overview, Python/Desktop adapters applied if needed |
| `vibe-coding-ship` | Routes to correct deploy skill, runs matching pre-flight checklist |

No default is assumed. If run_target is missing, Claude asks once.

### progress.txt — single source of truth

```
VIBE CODING PROGRESS
====================
CURRENT STATE:
phase: BUILD
step: implementing_auth
next_action: Create auth API routes
active_skill: vibe-coding-build
run_target: vercel
run_target_detail:

---
PHASE: IDEATE
STATUS: complete
mode: full
[conversation transcript...]

---
PHASE: DOCUMENT
STATUS: complete
docs_completed: 8/8
[doc list...]

---
PHASE: BUILD
STATUS: in_progress
current_step: 2.1
[step log...]
```

---

## All 49 Skills — What They Do

### Core Workflow

**vibe-coding-orchestrator**
Entry point for every session. Detects whether you're starting fresh, resuming, or have existing code. Routes to the correct skill. Reads progress.txt on activation to determine current state. Does not implement any phase itself — only coordinates. Auto-activates when opening Claude in a directory with 10+ code files and a dependency file.

**vibe-coding-ideate**
Requirements gathering conversation. 10 topics: app concept, target users, core features, must-have vs nice-to-have, tech stack preferences, run target, design style, data model, auth, and integrations. Fast-track mode compresses this to 6 questions for experienced builders. Saves full transcript and all decisions to progress.txt. Hooks into `vibe-coding-ui-ux` during topic 10 when design guidance is needed.

**vibe-coding-document**
Thin coordinator — does not generate any docs itself. Reads IDEATE output from progress.txt, then calls each of the 8 doc generator skills in sequence. Manages the approval loop between each doc: user approves, revises, or skips. Handles resume (skips already-approved docs). Triggers `vibe-coding-css-setup` after DESIGN_SYSTEM.md is approved.

**vibe-coding-build**
Implementation phase. Reads IMPLEMENTATION_PLAN.md and executes each step in order. At project start, calls `vibe-coding-css-setup` and `vibe-coding-design-templates`. Before generating each page or component, calls `vibe-coding-design-guard`. After each step, calls `vibe-coding-code-review`. Detects integration signals in step descriptions and calls `vibe-coding-api-connect`, `vibe-coding-cli-runner`, or `vibe-coding-mcp-setup` accordingly. Routes build errors to `vibe-coding-build-fix`.

**vibe-coding-debug**
Bug fixing. Before touching shared code, calls `vibe-coding-impact-analysis` to map blast radius. For UI bugs, calls `vibe-coding-ui-review`. For compile/type/import errors, calls `vibe-coding-build-fix`. Asks clarifying questions before reproducing vague bugs.

**vibe-coding-ship**
Deployment coordinator. Reads run_target from progress.txt. Runs pre-flight: `vibe-coding-doc-review` then `vibe-coding-security-review` (blocks deployment if CRITICAL or HIGH findings). Then routes to the correct deploy skill or local-runner. Never runs deployment commands itself.

**vibe-coding-reverse-engineer**
Coordinator for analyzing existing codebases. Checks for minimum codebase size, then calls re-scan → re-analyze → re-generate in sequence. Passes findings between stages via progress.txt.

---

### Document Generators (8)

Each skill generates one document. Loaded only for that document. All read from IDEATE output in progress.txt plus any previously generated docs.

**vibe-coding-doc-prd**
Generates PRD.md. Extracts must-have, should-have, and nice-to-have features from IDEATE transcript. Handles thin-data projects by asking 3 targeted questions. Shows preview, asks for approval before saving.

**vibe-coding-doc-appflow**
Generates APP_FLOW.md. Documents every screen with all 4 states: default, loading, error, empty. Maps navigation paths and user journeys. Handles SPAs (no page reload — documents state transitions instead). Falls back to progress.txt if PRD.md is missing.

**vibe-coding-doc-techstack**
Generates TECH_STACK.md. Every library gets an exact version. Every service gets a cost estimate. Deployment section has 6 variants keyed to run_target. Setup commands adapt to the actual stack (pip/uvicorn for Python, npm/prisma for Node).

**vibe-coding-doc-design**
Generates DESIGN_SYSTEM.md. Reads UI_UX_SELECTIONS from progress.txt if vibe-coding-ui-ux was used. Documents color tokens, typography scale, spacing, shadows, and border radius as CSS custom properties. Includes a globals.css scaffold section.

**vibe-coding-doc-backend**
Generates BACKEND_STRUCTURE.md. Documents database schema (Prisma or SQLAlchemy format based on stack), all API endpoints with request/response shapes, auth middleware, and error codes. Adapts pagination patterns to REST vs GraphQL.

**vibe-coding-doc-frontend**
Generates FRONTEND_GUIDELINES.md. Documents component structure, state management rules, data fetching patterns, layout template rules (pages extend base layout, never re-declare Header/Footer). Includes Vite SPA and Next.js App Router variants.

**vibe-coding-doc-implplan**
Generates IMPLEMENTATION_PLAN.md. Foundation-before-features order: Phase 0 setup, Phase 1 database, Phase 2 auth, Phase 3 shared layout, then one phase per must-have feature, final phase for polish and ship. Each step has a concrete verify outcome. Includes `vibe-coding-design-guard` call at the end of each feature phase. Final phase title and steps adapt to run_target.

**vibe-coding-doc-claudemd**
Generates CLAUDE.md and saves it to the project root (not docs/). Synthesizes all 7 preceding docs into the rulebook Claude reads on every invocation. Derives at least 3 project-specific gotchas from the actual tech choices and data model. Includes Python and desktop adapters. Warns when CLAUDE.md exceeds 150 lines and provides a split-to-docs/claude/ pattern.

---

### Reverse-Engineer Stages (3)

**vibe-coding-re-scan**
Fast framework and stack detection (~60 seconds). Reads package.json, requirements.txt, file extensions, and directory structure. Outputs: framework, database, ORM, auth, CSS approach, entry points, confidence score. Saves to progress.txt before displaying output.

**vibe-coding-re-analyze**
Deep 8-area analysis: framework patterns, database schema, API endpoints, auth flow, frontend components, business logic, third-party integrations, and deployment config. Assigns confidence score per area. Already at 100% baseline — no experiments needed.

**vibe-coding-re-generate**
Two-wave doc generation from analysis results. Wave 1: TECH_STACK, BACKEND_STRUCTURE, FRONTEND_GUIDELINES, APP_FLOW, DESIGN_SYSTEM from code evidence. Wave 2: PRD, IMPLEMENTATION_PLAN from user Q&A (5 business questions). Handles partial Wave 1 failures gracefully — marks failed docs, continues, adds to gap list.

---

### Design and CSS Chain (5)

These 5 skills solve the CSS inheritance problem. Without them, generated projects end up with fonts declared inline per component, hardcoded hex colors, and no consistent spacing. With them, all tokens flow from one source.

**vibe-coding-ui-ux**
Style selection tool. Presents 67 UI styles (minimal, glassmorphism, brutalist, etc.), 161 color palettes, 57 typography pairings, and UX guidelines by project type. Warns when style and audience mismatch (e.g., brutalist for a medical app). Can skip with project-type defaults. Saves selections to progress.txt for `vibe-coding-doc-design` to read.

**vibe-coding-css-setup**
One-time CSS foundation setup at project start. Generates: `globals.css` with Tailwind directives and `@layer base` typography (h1-h6, body, links), `tailwind.config.js` with full token mapping from DESIGN_SYSTEM.md, and root layout file with font import (Google Fonts CDN or next/font). Handles non-Tailwind projects by skipping Tailwind-specific sections and generating plain CSS variables only.

**vibe-coding-design-templates**
Site-type layout scaffolding. Detects project type from PRD.md and APP_FLOW.md (content/blog, ecommerce, SaaS, landing page, dashboard, portfolio). Scaffolds the base layout components — Header, Footer, Sidebar, AppLayout — with correct content slots. Pages extend the template; they never re-declare layout chrome. Includes Vite SPA and Next.js App Router structure variants.

**vibe-coding-design-guard**
Pre-generation consistency check. Runs 7 checks before any page or component is written: no hardcoded hex colors, no inline font declarations, no arbitrary Tailwind values (e.g., `mt-[13px]`), pages extend base layout, CSS variables referenced correctly, spacing uses scale multiples, no duplicate component definitions. Outputs MUST FIX (blocks generation) and SHOULD FIX (advisory) separately.

**vibe-coding-web-design-guidelines**
Live-fetches Vercel's web interface guidelines from GitHub. Checks: accessibility (contrast, focus, ARIA), visual hierarchy, interaction patterns, spacing consistency, and responsive behavior. Called by `vibe-coding-design-guard` after its 7 checks pass. Has built-in fallback rules if the fetch fails.

---

### Quality and Review (7)

**vibe-coding-code-review**
8-check review after each build step: naming conventions, single responsibility, dead code, TypeScript strictness, security anti-patterns, test coverage, import organization, and performance hot spots. Detects project language — skips TypeScript checks for Python files. Routes React/Next.js projects to `vibe-coding-review-react` and React Native to `vibe-coding-react-native`.

**vibe-coding-ui-review**
Screenshot-based UI analysis at 3 levels. Map level: layout structure and spacing. System level: color token usage, font consistency. Blueprint level: pixel-level measurements against DESIGN_SYSTEM.md. Generates fix suggestions that are framework-aware (Tailwind classes vs plain CSS). Handles missing DESIGN_SYSTEM.md by falling back to visible token values.

**vibe-coding-doc-review**
Checks that docs match actual code. 5 areas: TECH_STACK.md vs package.json/requirements.txt, DESIGN_SYSTEM.md vs globals.css, BACKEND_STRUCTURE.md vs actual route files, APP_FLOW.md vs implemented screens, IMPLEMENTATION_PLAN.md vs completed steps. Called by `vibe-coding-ship` before deployment as a pre-flight check.

**vibe-coding-impact-analysis**
Blast-radius analysis before modifying shared code. Traces which functions, hooks, and components call the target function. Assigns risk: HIGH (shared utility used in 5+ places), MEDIUM (used in 2-4 places), LOW (isolated). Flags deletion as always HIGH risk. Includes Python equivalent trigger paths. Called by `vibe-coding-debug` before applying any fix to shared code.

**vibe-coding-security-review**
OWASP Top 10 audit plus secrets scan. Checks: injection, auth weaknesses, sensitive data exposure, XXE, misconfiguration, XSS, insecure deserialization, vulnerable dependencies, logging gaps, SSRF. Scans for hardcoded API keys, passwords, and tokens. Outputs CRITICAL (blocks ship), HIGH (blocks ship), MEDIUM (advisory), LOW (advisory). Called by `vibe-coding-ship` during pre-flight — CRITICAL or HIGH findings block deployment.

**vibe-coding-tdd**
Test-first development cycle enforcement during BUILD. Writes failing test first, then implementation, then refactors until test passes. Supports Jest/Vitest (TypeScript) and pytest (Python). Reads progress.txt on direct activation to get context — asks user if progress.txt is missing.

**vibe-coding-build-fix**
Compile, type, and import error resolver. Classifies errors into 8 types: missing module, type mismatch, import path, missing export, syntax, config, environment, and Python-specific (import resolution, venv, pip). Provides targeted fix for each type. Includes Python module resolution section for common pip/venv/PYTHONPATH issues.

---

### Deployment (4)

**vibe-coding-local-runner**
Terminal startup for every supported framework and database. Next.js (`npm run dev`), React/Vite (`npm run dev`), Flask (`flask run`), Django (`python manage.py runserver`), FastAPI (`uvicorn main:app --reload`), Express (`nodemon server.js`), Electron (`npm run electron:dev`), Tauri (`npm run tauri dev`). Databases: SQLite (no server needed), BetterSQLite3 (file created on first connection), PostgreSQL (`brew services start` / `net start`), Convex (`npx convex dev`). Includes port conflict resolution for Windows and macOS/Linux. Writes a completion block to progress.txt when app starts successfully.

**vibe-coding-deploy-vercel**
Full Vercel deploy workflow. Interactive login (`vercel login`) or token-based CI auth. Env var setup (`vercel env add`). Production deploy (`vercel --prod`). Custom domain config. Error handling for build failures, missing env vars, and function timeouts. Includes serverless function `vercel.json` config. Note: Python (Flask/Django/FastAPI) has limitations on Vercel — serverless functions only, no persistent server.

**vibe-coding-deploy-netlify**
Full Netlify deploy workflow. `netlify login` or `NETLIFY_AUTH_TOKEN`. `netlify.toml` config generation (build command, publish directory, redirects, headers). Serverless functions setup. Deploy with `netlify deploy --prod`. Note: Python persistent servers are not supported on Netlify — redirects to Railway/Render/Fly.io/DigitalOcean for Python backends.

**vibe-coding-deploy-digitalocean**
Two paths: App Platform (managed PaaS) or Droplet (VPS). App Platform: `doctl apps create` with `.do/app.yaml` spec, env var injection, deploy log monitoring. Droplet path: provision Ubuntu, install nginx + PM2 (Node) or gunicorn + nginx (Python), set up SSL with Certbot, systemd service config. Already at 100% baseline.

---

### Integration (3)

**vibe-coding-api-connect**
REST and GraphQL API integration patterns. Auth patterns: API key (header/query param), Bearer token, OAuth2 (authorization code + client credentials). Pagination: cursor-based, offset, link-header. Error handling: retry with backoff, error classification by status code. Typed responses: Zod schemas (TypeScript), Pydantic models (Python). Webhooks: signature verification, idempotency. Python clients: httpx (async) and requests (sync). Python OAuth2 via authlib. Has a breakout rule: split into `vibe-coding-api-[name]` when a single service grows beyond 3 distinct auth patterns or 400 lines.

**vibe-coding-cli-runner**
CLI tool patterns used during build. Node/TypeScript: Prisma (`npx prisma migrate dev`), shadcn/ui (`npx shadcn@latest add`), Stripe CLI, Docker compose. Python: venv activation, Alembic migrations (`alembic upgrade head`), Django management commands (`manage.py migrate`, `manage.py createsuperuser`), uvicorn/gunicorn startup, pytest, Makefile pattern for common tasks.

**vibe-coding-mcp-setup**
MCP server configuration for Claude itself (not app code). Configures `claude_desktop_config.json` or `.claude/settings.json` for: filesystem, memory, sequential-thinking, GitHub, Slack, browser (Puppeteer/Playwright), databases (SQLite, PostgreSQL). Includes JSON merge guidance for adding multiple servers without overwriting existing config. Security checklist: never commit config files with tokens, use environment variables, review permissions per server.

---

### Framework Review (3)

**vibe-coding-review-react**
65 rules for React and Next.js. Covers: avoiding waterfalls (parallel data fetching), bundle size (dynamic imports, tree shaking), unnecessary re-renders (memo, useMemo, useCallback), Server Components vs Client Components decision tree, React 19 patterns (use hook, actions, optimistic updates), composition patterns over prop drilling, and Next.js App Router best practices.

**vibe-coding-react-native**
React Native and Expo review. FlashList over FlatList for large lists, Reanimated 3 API (useSharedValue, useAnimatedStyle), Expo Router file-based navigation, Pressable over TouchableOpacity, MMKV for local storage over AsyncStorage, image optimization with expo-image, OTA updates with expo-updates.

**vibe-coding-web-design-guidelines**
Live-fetches Vercel's published web interface guidelines. Applied after `vibe-coding-design-guard` passes its 7 checks. Covers accessibility standards (WCAG 2.1 AA), visual hierarchy, interaction feedback, focus management, motion preferences, and spacing consistency. Falls back to embedded core rules if GitHub fetch fails.

---

### State and Memory (3)

**vibe-coding-state**
The only skill that reads and writes progress.txt directly. All other skills request state operations through it. Validates phase transitions (can't go to BUILD before DOCUMENT is complete). Handles resume: reads CURRENT STATE block to find active phase and step. Handles missing progress.txt by creating a new one with blank state. Provides WRITE STATE, READ STATE, VALIDATE TRANSITION, and RECOVER operations.

**vibe-coding-explore**
AST-powered code navigation (requires claude-mem plugin). Replaces reading large files — instead navigates directly to specific functions, classes, or exports. Uses smart_search, smart_outline, and smart_unfold tools. Dramatically reduces tokens when working in files over 100 lines. Already at 100% baseline.

**vibe-coding-recall**
Cross-session memory recall at session start (requires claude-mem plugin). Restores context from previous sessions without re-reading all project files. Called by the orchestrator when resuming work. Already at 100% baseline.

---

### Database (6)

One coordinator detects the database and routes to the right per-database skill. Skills sourced from [Convex agent-skills](https://github.com/waynesutton/convexskills) and [Supabase agent-skills](https://github.com/supabase/agent-skills).

| Skill | Database | Best for |
|-------|----------|----------|
| `vibe-coding-db` | Coordinator | Detection, routing, "which database should I use" |
| `vibe-coding-db-sqlite` | SQLite | Local apps, Python + Node.js, file-based zero config |
| `vibe-coding-db-bettersqlite` | BetterSQLite3 | Node.js sync SQLite, Electron/Tauri, 2-3x faster than sqlite3 |
| `vibe-coding-db-postgres` | PostgreSQL | Multi-user SaaS, Supabase, RLS, full-text search, OLTP |
| `vibe-coding-db-duckdb` | DuckDB | Analytics, OLAP, querying CSV/Parquet files directly |
| `vibe-coding-db-convex` | Convex | Real-time apps, TypeScript-native, AI agents, live subscriptions |

**vibe-coding-db** (coordinator)
Detects database from progress.txt, package.json, or requirements.txt and routes instantly. If ambiguous, presents a decision table and asks once. ORM-aware — adapts output to Prisma, Drizzle, SQLAlchemy, Alembic, or Django ORM when detected. Called by `vibe-coding-build` when a database step is reached in IMPLEMENTATION_PLAN.

**vibe-coding-db-sqlite**
File-based, zero config. Covers raw SQL (Python and Node.js), Prisma with SQLite provider, Drizzle with SQLite, WAL mode pragmas, indexing, FTS5 full-text search, versioned migration runner, and common pitfalls (foreign keys off by default, no boolean type, WAL required).

**vibe-coding-db-bettersqlite**
Synchronous Node.js SQLite driver — 2-3x faster than the async `sqlite3` package. Covers connection setup, essential pragmas, CRUD with prepared statements, transactions (synchronous, nested), Drizzle ORM integration, batch insert pattern, TypeScript types, and native module rebuild for Electron.

**vibe-coding-db-postgres**
Full relational database for multi-user apps. Sourced from Supabase's official postgres best practices across 8 categories: query performance (index types, composite, partial, covering, FTS), connection management (pooling, max connections, idle timeout), row-level security (policies, performance), schema design (primary keys, data types, FK indexes), concurrency (deadlock prevention, SKIP LOCKED), data access (N+1, cursor pagination, upserts, batch inserts), monitoring (EXPLAIN ANALYZE, pg_stat_statements, vacuum), and advanced features (JSONB indexing). Covers local PostgreSQL and Supabase cloud. ORM support: Prisma, Drizzle, SQLAlchemy, Alembic, Django ORM.

**vibe-coding-db-duckdb**
In-process columnar analytics database. No server, no config. Runs inside your app. Covers: querying CSV/Parquet/JSON files directly (no import needed), complex aggregations, window functions, pivoting, combining with SQLite or PostgreSQL via ATTACH, exporting to CSV/Parquet/Pandas/Polars, and performance tips (parallel execution, persistent cache, Parquet column pruning). Node.js and Python APIs.

**vibe-coding-db-convex**
Reactive serverless backend sourced from official Convex agent-skills. Covers: schema definition with typed validators, reactive queries (auto-updating UI subscriptions), transactional mutations, actions for external API calls, HTTP actions for webhooks (Stripe, Clerk), cron jobs, file storage (upload → store → serve URL), optional-field-first migrations, RBAC security patterns, and AI agent integration with `@convex-dev/agent`. Includes function type decision tree (query/mutation/action/httpAction/cron).

---

### Self-Improvement (1)

**vibe-coding-self-improve**
Reflects on workflow gaps, proposes surgical SKILL.md diffs, and logs every change. Triggers on user request ("improve this skill", "the skill missed X") or via orchestrator routing. Reads the current SKILL.md, proposes a targeted diff with risk level (LOW/MEDIUM/HIGH), waits for explicit user approval, then applies the change with Edit tool. Logs every improvement to `autoresearch/autoresearch-results.md`. Out of scope without explicit user instruction: changing workflow order, merging/splitting skills, removing existing functionality. Also runs 5-point autoresearch checks (trigger conflicts, handoff integrity, progress.txt alignment, orchestrator registration, file/path accuracy) on individual or all skills.

---

## Autoresearch Results

All 49 skills have been autoresearched against 5 test inputs x 5 binary evals using the Karpathy autoresearch method. One mutation per experiment. Loop until 100% for 3 consecutive stability runs.

### Summary

| # | Skill | Baseline | Final | Experiments | Key Fix |
|---|-------|----------|-------|-------------|---------|
| 1 | vibe-coding-orchestrator | 80% | 100% | 3 | Bug/error intercept before routing; routing announcements; step status check on resume |
| 2 | vibe-coding-state | 80% | 100% | 3 | Concrete WRITE STATE format; VALIDATE error messages; RESUME missing-file handler |
| 3 | vibe-coding-ideate | 68% | 100% | 4 | Mid-session fast-track switch; progress.txt write on activation; resume recap; handoff announcement |
| 4 | vibe-coding-document | 84% | 100% | 4 | Resume skip-approved-docs; revision handler; post-RE format variant; PRD import gap-filling |
| 5 | vibe-coding-doc-prd | 76% | 100% | 4 | Thin data handler; target user refinement; existing PRD check; preview + approval prompt |
| 6 | vibe-coding-doc-appflow | 72% | 100% | 3 | Preview + approval; missing PRD fallback to progress.txt; SPA handler |
| 7 | vibe-coding-doc-techstack | 72% | 100% | 2 | Preview + approval; stack-adaptive setup commands (pip/uvicorn vs npm/prisma) |
| 8 | vibe-coding-doc-design | 76% | 100% | 3 | Preview + approval; UI_UX_SELECTIONS lookup in progress.txt; validation checklist output |
| 9 | vibe-coding-doc-backend | 72% | 100% | 3 | Preview + approval; ORM adapter rule; API style adapter for pagination |
| 10 | vibe-coding-doc-frontend | 72% | 100% | 2 | Preview + approval; Vite + Next.js Pages Router structure variants |
| 11 | vibe-coding-doc-implplan | 72% | 100% | 3 | Preview + approval; no-auth handler; Python stack setup variants (FastAPI/Flask/Django) |
| 12 | vibe-coding-doc-claudemd | 56% | 100% | 2 | Specific gotchas derivation + preview prompt; Python stack adapter for state/testing |
| 13 | vibe-coding-doc-review | 84% | 100% | 3 | globals.css path discovery; requirements.txt/pyproject.toml fallback; missing-file handler |
| 14 | vibe-coding-build | 96% | 100% | 1 | Explicit error handler in implementation loop routing to vibe-coding-build-fix |
| 15 | vibe-coding-css-setup | 96% | 100% | 1 | Non-Tailwind handler — skip directives for plain CSS/SCSS projects |
| 16 | vibe-coding-design-templates | 88% | 100% | 3 | Type detection scoring; Vite page structure variant; read APP_FLOW.md screen inventory |
| 17 | vibe-coding-design-guard | 64% | 100% | 3 | MUST FIX vs SHOULD FIX severity levels; explicit DESIGN_SYSTEM.md token read; re-check trigger |
| 18 | vibe-coding-ui-ux | 88% | 100% | 3 | Style-audience mismatch warning; skip path with project-type defaults; custom color expansion |
| 19 | vibe-coding-ui-review | 92% | 100% | 2 | DESIGN_SYSTEM.md missing handler; framework-aware fix suggestions for non-Tailwind |
| 20 | vibe-coding-code-review | 96% | 100% | 1 | Language detection — skip TypeScript strictness check for Python files |
| 21 | vibe-coding-debug | 96% | 100% | 1 | Vague bug handler — asks clarifying questions before reproducing |
| 22 | vibe-coding-impact-analysis | 92% | 100% | 2 | Python-equivalent trigger paths; deletion always HIGH risk |
| 23 | vibe-coding-reverse-engineer | 92% | 100% | 1 | Minimum codebase check (warn if fewer than 5 code files) |
| 24 | vibe-coding-re-scan | 80% | 100% | 1 | Save to progress.txt before displaying scan output |
| 25 | vibe-coding-re-analyze | 100% | 100% | 0 | Already perfect at baseline |
| 26 | vibe-coding-re-generate | 96% | 100% | 1 | Wave 1 partial failure handler — mark failed docs, continue, add to gap list |
| 27 | vibe-coding-ship | 88% | 100% | 2 | Platform-specific deployment commands; Python pre-flight checklist items |
| 28 | vibe-coding-explore | 100% | 100% | 0 | Already perfect at baseline |
| 29 | vibe-coding-recall | 100% | 100% | 0 | Already perfect at baseline |
| 30 | vibe-coding-security-review | 100% | 100% | 0 | Already perfect at baseline |
| 31 | vibe-coding-tdd | 92% | 100% | 1 | Direct-activation handler — read progress.txt or ask user for context |
| 32 | vibe-coding-build-fix | 76% | 100% | 1 | Python error classification + Python module resolution section |
| 33 | vibe-coding-api-connect | 80% | 100% | 1 | Python httpx/requests client; Python OAuth2 via authlib; Flask webhook handler |
| 34 | vibe-coding-cli-runner | 68% | 100% | 1 | Python CLI: venv, Alembic, Django management, uvicorn/gunicorn, pytest, Makefile |
| 35 | vibe-coding-mcp-setup | 84% | 100% | 1 | JSON merge guidance for multiple servers; security checklist in setup flow |
| 36 | vibe-coding-deploy-vercel | 80% | 100% | 1 | Python on Vercel limitation note + serverless function vercel.json config |
| 37 | vibe-coding-deploy-netlify | 80% | 100% | 1 | Python persistent server limitation — redirect to Railway/Render/Fly.io/DO |
| 38 | vibe-coding-deploy-digitalocean | 100% | 100% | 0 | Already perfect at baseline |
| 39 | vibe-coding-local-runner | 100% | 100% | 0 | Already perfect at baseline |
| 40 | vibe-coding-review-react | 100% | 100% | 0 | Already perfect at baseline |
| 41 | vibe-coding-react-native | 100% | 100% | 0 | Already perfect at baseline |
| 42 | vibe-coding-web-design-guidelines | 100% | 100% | 0 | Already perfect at baseline |
| 43 | vibe-coding-self-improve | 40% | 100% | 1 | Fixed handoff claims, progress.txt format, orchestrator registration, created results file |
| 44 | vibe-coding-db | 80% | 100% | 1 | Progress block used non-canonical PHASE: DB_SETUP — changed to append under active BUILD phase |
| 45 | vibe-coding-db-sqlite | 80% | 100% | 1 | No progress.txt update section — added with BUILD phase append format |
| 46 | vibe-coding-db-bettersqlite | 80% | 100% | 1 | No progress.txt update section — added with BUILD phase append format |
| 47 | vibe-coding-db-postgres | 80% | 100% | 1 | No progress.txt update section — added with connection_pooling flag |
| 48 | vibe-coding-db-duckdb | 80% | 100% | 1 | No progress.txt update section — added with primary_db field for secondary analytics pattern |
| 49 | vibe-coding-db-convex | 80% | 100% | 1 | No progress.txt update section — added with auth_configured and file_storage_configured flags |

### Stats

| Metric | Value |
|--------|-------|
| Total skills | 49 |
| Already at 100% baseline | 11 |
| Improved | 38 |
| Lowest baseline | 40% (self-improve — post-creation) |
| Average baseline | 84.2% |
| Total experiments run | 120 |
| All skills final score | 100% |

### Most Common Failure Patterns Found

**1. Missing preview + approval prompt (12 skills)**
Doc generator skills produced output and moved on without showing a preview or asking for confirmation. Fixed by adding a preview step to each doc skill.

**2. JS/TypeScript-only patterns breaking Python stacks (11 skills)**
Skills assumed npm, package.json, and Node.js. Python projects failed because pip, venv, Alembic, gunicorn, and pytest were not covered. Fixed with Python adapters per skill.

**3. No handler for missing prerequisite files (8 skills)**
Skills with dependencies on PRD.md, DESIGN_SYSTEM.md, or progress.txt had no fallback when those files were absent. Fixed by adding "if file missing, fall back to X or ask user" per skill.

**4. No edge case for direct activation (5 skills)**
Skills designed to be called by BUILD or SHIP had no handling for direct user invocation without context. Fixed by adding "read progress.txt first; if missing, ask user."

**5. Missing severity levels in review output (3 skills)**
design-guard, code-review, and security-review did not distinguish blocking issues from advisory ones. Fixed by adding MUST FIX / SHOULD FIX and CRITICAL / HIGH / MEDIUM / LOW output sections.

Full experiment logs: `autoresearch/RESULTS.md` and per-skill `autoresearch/autoresearch-[name]/` folders.

### Progressive Disclosure Audit

All 49 skills audited against 4 binary criteria:
1. Entry router / decision tree at the top
2. Sections clearly labeled (Claude can jump directly)
3. Heavy content gated behind named sections
4. No wall of code before context is established

**Result: all 49 skills at 4/4 (100%).**

Skills that required fixes:

| Skill | Disclosure Baseline | Disclosure Final | Autoresearch Baseline | Autoresearch Final |
|-------|--------------------|-----------------|-----------------------|-------------------|
| vibe-coding-web-design-guidelines | 1/4 | 4/4 | 100% | 100% |
| vibe-coding-ui-review | 1/4 | 4/4 | 92% | 100% |
| vibe-coding-api-connect | 1/4 | 4/4 | 80% | 100% |
| vibe-coding-security-review | 2/4 | 4/4 | 100% | 100% |
| vibe-coding-build-fix | 2/4 | 4/4 | 76% | 100% |
| vibe-coding-review-react | 2/4 | 4/4 | 100% | 100% |
| vibe-coding-react-native | 2/4 | 4/4 | 100% | 100% |
| vibe-coding-impact-analysis | 2/4 | 4/4 | 92% | 100% |
| vibe-coding-tdd | 2/4 | 4/4 | 92% | 100% |
| vibe-coding-mcp-setup | 2/4 | 4/4 | 84% | 100% |
| vibe-coding-ui-ux | 2/4 | 4/4 | 88% | 100% |
| vibe-coding-cli-runner | 2/4 | 4/4 | 68% | 100% |
| vibe-coding-orchestrator | 3/4 | 4/4 | 80% | 100% |
| vibe-coding-ideate | 3/4 | 4/4 | 68% | 100% |
| vibe-coding-document | 3/4 | 4/4 | 84% | 100% |
| vibe-coding-debug | 3/4 | 4/4 | 96% | 100% |
| vibe-coding-ship | 3/4 | 4/4 | 88% | 100% |
| vibe-coding-state | 3/4 | 4/4 | 80% | 100% |
| vibe-coding-explore | 3/4 | 4/4 | 100% | 100% |
| vibe-coding-recall | 3/4 | 4/4 | 100% | 100% |
| vibe-coding-self-improve | 3/4 | 4/4 | 40% | 100% |
| vibe-coding-code-review | 3/4 | 4/4 | 96% | 100% |
| vibe-coding-design-guard | 3/4 | 4/4 | 64% | 100% |
| vibe-coding-css-setup | 3/4 | 4/4 | 96% | 100% |
| vibe-coding-local-runner | 3/4 | 4/4 | 100% | 100% |
| vibe-coding-deploy-vercel | 3/4 | 4/4 | 80% | 100% |
| vibe-coding-deploy-netlify | 3/4 | 4/4 | 80% | 100% |
| vibe-coding-deploy-digitalocean | 3/4 | 4/4 | 100% | 100% |
| vibe-coding-db | 3/4 | 4/4 | 80% | 100% |
| vibe-coding-build | 4/4 | 4/4 | 96% | 100% |
| vibe-coding-design-templates | 4/4 | 4/4 | 88% | 100% |
| vibe-coding-reverse-engineer | 4/4 | 4/4 | 92% | 100% |
| vibe-coding-re-scan | 4/4 | 4/4 | 80% | 100% |
| vibe-coding-re-analyze | 4/4 | 4/4 | 100% | 100% |
| vibe-coding-re-generate | 4/4 | 4/4 | 96% | 100% |
| vibe-coding-doc-prd | 4/4 | 4/4 | 76% | 100% |
| vibe-coding-doc-appflow | 4/4 | 4/4 | 72% | 100% |
| vibe-coding-doc-techstack | 4/4 | 4/4 | 72% | 100% |
| vibe-coding-doc-design | 4/4 | 4/4 | 76% | 100% |
| vibe-coding-doc-backend | 4/4 | 4/4 | 72% | 100% |
| vibe-coding-doc-frontend | 4/4 | 4/4 | 72% | 100% |
| vibe-coding-doc-implplan | 4/4 | 4/4 | 72% | 100% |
| vibe-coding-doc-claudemd | 4/4 | 4/4 | 56% | 100% |
| vibe-coding-doc-review | 4/4 | 4/4 | 84% | 100% |
| vibe-coding-db-sqlite | 4/4 | 4/4 | 80% | 100% |
| vibe-coding-db-bettersqlite | 4/4 | 4/4 | 80% | 100% |
| vibe-coding-db-postgres | 4/4 | 4/4 | 80% | 100% |
| vibe-coding-db-duckdb | 4/4 | 4/4 | 80% | 100% |
| vibe-coding-db-convex | 4/4 | 4/4 | 80% | 100% |

### Progressive Disclosure Rewrite Pass

A deeper structural rewrite pass was applied to 24 skills after the jump directive audit. This pass enforced a strict 3-tier content model and collapsed actual line counts by 44-71%.

**3-tier structure enforced:**
- Tier 1: Entry Router (10-20 lines max) — detects context, emits exactly one `→ READ:` cue
- Tier 2: Named `##` sections — one per variant — loaded only when matched
- Tier 3: Heavy code blocks and full configs — only inside matched Tier 2 sections

**Line count reductions:**

| Skill | Before | After | Reduction | Autoresearch Score |
|-------|--------|-------|-----------|-------------------|
| vibe-coding-api-connect | 568 | ~200 | 65% | 100% |
| vibe-coding-cli-runner | 503 | ~150 | 70% | 100% |
| vibe-coding-local-runner | 513 | ~150 | 71% | 100% |
| vibe-coding-deploy-vercel | 355 | ~130 | 63% | 100% |
| vibe-coding-deploy-netlify | 314 | ~120 | 62% | 100% |
| vibe-coding-deploy-digitalocean | 335 | ~130 | 61% | 100% |
| vibe-coding-tdd | 372 | ~150 | 60% | 100% |
| vibe-coding-review-react | 268 | ~150 | 44% | 100% |
| vibe-coding-react-native | 268 | ~130 | 51% | 100% |
| vibe-coding-web-design-guidelines | 244 | ~100 | 59% | 100% |
| vibe-coding-code-review | 258 | ~120 | 53% | 100% |
| vibe-coding-ui-review | 212 | ~100 | 53% | 100% |
| vibe-coding-build-fix | 414 | ~150 | 64% | 100% |
| vibe-coding-mcp-setup | 400 | ~130 | 68% | 100% |
| vibe-coding-impact-analysis | 210 | ~80 | 62% | 100% |
| vibe-coding-security-review | 279 | ~100 | 64% | 100% |
| vibe-coding-explore | 148 | ~80 | 46% | 100% |
| vibe-coding-recall | 165 | ~80 | 52% | 100% |
| vibe-coding-self-improve | 276 | ~110 | 60% | 100% |

**Content additions in this pass:**
- `vibe-coding-build-fix`: added Go and Rust error sections (undefined symbol, borrow checker, trait bounds, Cargo.lock)
- `vibe-coding-tdd`: added RSpec (Ruby) and Rust `#[test]` templates alongside Vitest/pytest/Go
- `vibe-coding-local-runner`: added Go, Ruby on Rails, Rust, and PHP startup sections
- `vibe-coding-code-review`: added language adaptation rules for Python, Go, Ruby, and PHP
- `vibe-coding-design-guard`: added Svelte and Vue single-file component check sections
- `vibe-coding-css-setup`: added Tailwind v4 setup section (`@import "tailwindcss"`, `@theme {}` block, no config file)
- `vibe-coding-react-native`: added Expo New Architecture check (SDK 51+ `newArchEnabled` flag)
- `vibe-coding-review-react`: added React 19 patterns (ref-as-prop, `use()` hook, Server Actions, `<Context>` provider)
- `vibe-coding-explore`: added "no file/query specified" guard at entry
- `vibe-coding-mcp-setup`: added settings.json create-if-missing step
- `vibe-coding-db`: added unsupported ORM fallback message (TypeORM, Sequelize, Peewee, Tortoise)

**Aggregate token savings:** 40-70% per skill invocation when only one variant is needed (the common case).

---

## Architecture

```
cp-vibe-coding/
├── skills/
│   ├── vibe-coding-orchestrator/
│   ├── vibe-coding-ideate/
│   ├── vibe-coding-document/
│   ├── vibe-coding-doc-prd/
│   ├── vibe-coding-doc-appflow/
│   ├── vibe-coding-doc-techstack/
│   ├── vibe-coding-doc-design/
│   ├── vibe-coding-doc-backend/
│   ├── vibe-coding-doc-frontend/
│   ├── vibe-coding-doc-implplan/
│   ├── vibe-coding-doc-claudemd/
│   ├── vibe-coding-build/
│   ├── vibe-coding-debug/
│   ├── vibe-coding-ship/
│   ├── vibe-coding-reverse-engineer/
│   ├── vibe-coding-re-scan/
│   ├── vibe-coding-re-analyze/
│   ├── vibe-coding-re-generate/
│   ├── vibe-coding-css-setup/
│   ├── vibe-coding-design-templates/
│   ├── vibe-coding-design-guard/
│   ├── vibe-coding-ui-ux/
│   ├── vibe-coding-ui-review/
│   ├── vibe-coding-web-design-guidelines/
│   ├── vibe-coding-code-review/
│   ├── vibe-coding-review-react/
│   ├── vibe-coding-react-native/
│   ├── vibe-coding-doc-review/
│   ├── vibe-coding-impact-analysis/
│   ├── vibe-coding-security-review/
│   ├── vibe-coding-tdd/
│   ├── vibe-coding-build-fix/
│   ├── vibe-coding-api-connect/
│   ├── vibe-coding-cli-runner/
│   ├── vibe-coding-mcp-setup/
│   ├── vibe-coding-local-runner/
│   ├── vibe-coding-deploy-vercel/
│   ├── vibe-coding-deploy-netlify/
│   ├── vibe-coding-deploy-digitalocean/
│   ├── vibe-coding-state/
│   ├── vibe-coding-explore/
│   ├── vibe-coding-recall/
│   └── vibe-coding-self-improve/
├── autoresearch/
│   ├── RESULTS.md                     # All autoresearch scores and key fixes
│   ├── autoresearch-results.md        # Self-improve change log
│   └── autoresearch-[skill]/          # Per-skill experiment logs and changelogs
└── README.md
```

---

## Token Efficiency

| Before | After |
|--------|-------|
| `vibe-coding-document`: 5,383 lines per doc run | Each doc skill: 300-500 lines, loaded only for that doc |
| `vibe-coding-reverse-engineer`: 4,381 lines always | Scan ~200, analyze ~800, generate ~600 — load only the active stage |
| Design mixed into document skill | `vibe-coding-doc-design` + `vibe-coding-css-setup` called once with full focus |
| No design enforcement during build | `vibe-coding-design-guard` is under 200 lines, called before each page only |

**Estimated reduction per typical workflow run: 60-70%**

---

## Requirements

- Claude Code v1.0.33+ or Claude.ai
- Any OS (Windows, macOS, Linux)
- For token-efficient code navigation: [claude-mem](https://github.com/anthropics/claude-mem) plugin (optional — needed only for `vibe-coding-explore` and `vibe-coding-recall`)

---

## Troubleshooting

**Skills not activating:**
```bash
ls ~/.claude/skills/vibe-coding-orchestrator
```
Try explicit phrases: "build me an app", "let's ideate", "generate docs from my code"

**Context not transferring between sessions:**
- Check `progress.txt` exists in project directory
- Say: "continue where we left off"

**CSS not inheriting correctly:**
- Ensure `vibe-coding-css-setup` ran at project start (Step 0.2 in IMPLEMENTATION_PLAN)
- Run `vibe-coding-design-guard` before generating any new page

**Python project failing build steps:**
- Skills have Python adapters but need the correct stack signal in progress.txt
- Ensure IDEATE captured the backend framework (Flask/Django/FastAPI)

---

## Contributing

1. Fork the repository
2. Create a feature branch
3. Test with the core workflow skills
4. Submit a pull request

Issues: [github.com/aimonk2025/cp-vibe-coding/issues](https://github.com/aimonk2025/cp-vibe-coding/issues)

---

## License

MIT
