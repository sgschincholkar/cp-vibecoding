---
name: vibe-coding-ship
description: |
  Deployment workflow with pre-flight checks, environment setup, deployment execution, and post-launch verification.
  Use when: (1) Deployment request - user says "deploy", "ship it", "launch", "go live", "push to production",
  (2) BUILD complete - orchestrator delegates SHIP after all phases finished,
  (3) Staging deployment - user says "deploy to staging", "test deployment",
  (4) Production deployment - user says "deploy to production", "push live".
  Runs pre-flight checklist (security review blocks if CRITICAL/HIGH found). Routes to platform-specific deploy skill.
---

# Vibe Coding SHIP Phase

Pre-flight → deploy → verify. Routes to the right platform skill.

## Prerequisites

- BUILD phase complete (all phases done, no open debug sessions)
- User explicitly requests deployment

---

## Step 1 — Platform Detection

Read `run_target` from progress.txt.
Not found → read `./docs/TECH_STACK.md` Run Target section.
Still unknown → ask once:
```
Where is this app running?
1. Local machine (terminal)
2. Vercel
3. Netlify
4. DigitalOcean
5. Desktop app (Electron / Tauri)
6. Other — specify
```

Save answer to progress.txt as `run_target`.

---

## Step 2 — Pre-Flight Checklist

Run `vibe-coding-doc-review` — verifies docs match code.
Run `vibe-coding-security-review` — **CRITICAL or HIGH findings BLOCK deployment.**

Then jump directly to the checklist that matches `run_target` — do not read all checklists:

### Local / Desktop Checklist

```
PRE-FLIGHT — LOCAL
App starts without errors (npm run dev / python app.py / etc.)
All required env vars in .env / .env.local
Database connection works (migrations run if applicable)
All tests passing
No missing packages (node_modules / venv up to date)

[Python only]
  Migrations run (alembic upgrade head / python manage.py migrate)
  Virtual environment active
  Required services running (Postgres, Redis, etc.)

[Desktop only]
  npm run [electron:build / tauri build] completes
  Output installer/binary exists in dist/ or target/release/bundle/
```

### Cloud Deploy Checklist (Vercel / Netlify / DigitalOcean / other)

```
PRE-FLIGHT — CLOUD
Production env vars set (no dev values / no hardcoded secrets)
API keys configured for production
All tests passing
No console.log / print debug statements
CORS configured properly
Rate limiting enabled
Build size acceptable

[Python only]
  DEBUG=False in production settings
  SECRET_KEY is a random value (not dev default)
  Migrations run
  Static files collected (python manage.py collectstatic)
  WSGI server configured (gunicorn in Procfile or platform config)
```

If any item fails: fix it before proceeding. Do not deploy with failing checks.

---

## Step 3 — Deploy

Route to the correct skill based on `run_target`:

| run_target | Skill |
|-----------|-------|
| `local` | `vibe-coding-local-runner` |
| `vercel` | `vibe-coding-deploy-vercel` |
| `netlify` | `vibe-coding-deploy-netlify` |
| `digitalocean` | `vibe-coding-deploy-digitalocean` |
| `desktop` | `vibe-coding-local-runner` (packaging mode) |
| `other_cloud` | `vibe-coding-cli-runner` with platform command from `run_target_detail` |

For `other_cloud` where `run_target_detail` is Railway / Fly.io / Render / Heroku: warn user that a platform config file is required before deployment:
- Railway → `railway.toml`
- Fly.io → `fly.toml` (generate with `fly launch`)
- Render → `render.yaml`
- Heroku → `Procfile`

Activate the skill and exit. That skill handles all deployment steps.

After deploy skill returns success → confirm smoke test: ask user to verify `[deployment_url]` loads correctly → if confirmed: write `smoke_test: passed` to progress.txt. Note that monitoring should be configured via platform dashboard.

---

## Progress Tracking

```
PHASE: SHIP
STATUS: in_progress
deployment_target: [platform]
started: [timestamp]

CHECKLIST:
✓ Doc review - Complete
✓ Security review - No blockers
✓ Pre-flight checks - Complete
⊙ Deploying - In Progress
✗ Smoke tests - Not Started
✗ Monitoring - Not Started

NEXT ACTION: [current step]
```

---

## Completion

```
PHASE: SHIP
STATUS: complete
completed: [timestamp]
deployment_url: [URL if applicable]
deployment_successful: true

PROJECT COMPLETE!
```
