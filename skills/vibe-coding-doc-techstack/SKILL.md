---
name: vibe-coding-doc-techstack
description: |
  Generate TECH_STACK.md — exact frameworks, versions, hosting, and integrations.
  Covers frontend, backend, database, auth, storage, deployment, monitoring, and cost estimates.
  No ambiguity — every dependency named and version-pinned.
  Use when: (1) vibe-coding-document delegates doc #3 (TECH_STACK),
  (2) User says "generate tech stack doc", "document the tech stack", "write TECH_STACK.md".
  Do NOT trigger for: general "what tech should I use" (that is vibe-coding-ideate).
  Reads interrogation transcript and PRD.md for technology choices.
  Outputs TECH_STACK.md to ./docs/.
---

# Vibe Coding — Tech Stack Generator

Generate TECH_STACK.md. Every library gets a version, every service gets a cost estimate.

## Prerequisites Check

1. Read `progress.txt` — technology choices from Topic 6 + `run_target`
2. Read `./docs/PRD.md` — features that drive technology requirements
3. Detect stack type → read only the relevant deployment section:

```
run_target = local      → READ: ## Deployment: Local
run_target = vercel     → READ: ## Deployment: Vercel
run_target = netlify    → READ: ## Deployment: Netlify
run_target = digitalocean → READ: ## Deployment: DigitalOcean
run_target = desktop    → READ: ## Deployment: Desktop
run_target = other_cloud → READ: ## Deployment: Other Cloud
```

4. Detect backend language → read only the relevant setup section:

```
Python (FastAPI)  → READ: ## Setup: Python FastAPI
Python (Flask)    → READ: ## Setup: Python Flask
Python (Django)   → READ: ## Setup: Python Django
Node.js / Next.js → READ: ## Setup: Node.js
Desktop Electron  → READ: ## Setup: Electron
Desktop Tauri     → READ: ## Setup: Tauri
```

---

## Generation Steps

1. Extract all technology choices from transcript
2. Look up current stable versions
3. Build sections: Frontend | Backend | Database | Auth | Storage | Deployment | Monitoring | Dev Tools
4. Add integration details for each third-party service
5. Add cost estimates for all hosting/services
6. Add setup commands (from matching Setup section)
7. Add environment variables
8. Save to `./docs/TECH_STACK.md`
9. Show first 20 lines as preview
10. Run Validation Checklist
11. Ask: "TECH_STACK generated. Approve or request revisions?"

## Output Format

```markdown
# [APP NAME] — Tech Stack

## Overview
Architecture: [Monolith | Serverless | JAMstack | Desktop]
Primary Language: [Language + version]
Run Target: [value from progress.txt]

## Frontend
| Technology | Version | Purpose |
| React | 18.2.0 | UI framework |
...

## Backend
| Technology | Version | Purpose |

## Database
| Service | Plan | Purpose | Cost |

## Authentication
| Service | Version | Purpose |

## File Storage (if applicable)
| Service | Plan | Purpose | Cost |

## Deployment
[Content from matching Deployment section]

## Monitoring
[Content from matching Monitoring section]

## Dev Tools
| Tool | Version | Purpose |

## Environment Variables
[Content from matching Setup section]

## Setup Commands
[Content from matching Setup section]
```

## Generation Rules

1. Every technology has an exact version — no "latest"
2. Every paid service has a cost estimate (even if $0)
3. Setup commands must match the actual stack — do not default to Next.js commands for Python/desktop projects
4. If technology version not specified — use current LTS/stable

## Validation Checklist

```
TECH_STACK VALIDATION:
✓/✗ All versions exact (no "latest")
✓/✗ Cost estimates for all services
✓/✗ Environment variables complete
✓/✗ Setup commands correct for stack
✓/✗ No conflicting technologies (e.g., two ORMs)
✓/✗ Deployment section matches run_target
```

---

## Deployment: Local

```markdown
## Deployment / Run Target: Local Machine
| What | How | Cost |
|------|-----|------|
| Start app | [npm run dev / flask run / python manage.py runserver / uvicorn main:app --reload] | $0 |
| Database | [local PostgreSQL / SQLite file / npx convex dev] | $0 |
| Total monthly cost | Local machine only | $0/mo |
```
Monitoring: No service needed. Use browser devtools and terminal logs.

---

## Deployment: Vercel

```markdown
## Deployment / Run Target: Vercel
| Service | Plan | Purpose | Cost |
|---------|------|---------|------|
| Vercel | Hobby (free) | Frontend + API hosting | $0/mo |
| [DB host] | [Plan] | Database | $0-$25/mo |
| Total estimated monthly cost | | | $[N]/mo |
```
Monitoring: Vercel Analytics (built-in) + Sentry free tier.

---

## Deployment: Netlify

```markdown
## Deployment / Run Target: Netlify
| Service | Plan | Purpose | Cost |
|---------|------|---------|------|
| Netlify | Free starter | Frontend + serverless | $0/mo |
| [DB host] | [Plan] | Database | $0-$25/mo |
| Total estimated monthly cost | | | $[N]/mo |
```
Monitoring: Netlify Analytics + Sentry free tier.

---

## Deployment: DigitalOcean

```markdown
## Deployment / Run Target: DigitalOcean
| Service | Plan | Purpose | Cost |
|---------|------|---------|------|
| DO App Platform / Droplet | Basic | App hosting | $5-$12/mo |
| DO Managed Postgres | Dev (1GB) | Database | $15/mo |
| Total estimated monthly cost | | | ~$20-$27/mo |
```
Monitoring: DO built-in metrics + Sentry free tier.

---

## Deployment: Desktop

```markdown
## Deployment / Run Target: Desktop App
| What | How | Cost |
|------|-----|------|
| Dev mode | [npm run electron:dev / npm run tauri dev] | $0 |
| Distribution | electron-builder / tauri build — outputs installer | $0 |
| Total cost | One-time build, no hosting | $0/mo |
```
Monitoring: No external service. Use Electron/Tauri crash reporters.

---

## Deployment: Other Cloud

```markdown
## Deployment / Run Target: [Platform from run_target_detail]
| Service | Plan | Purpose | Cost |
|---------|------|---------|------|
| [Platform] | [Plan] | App hosting | $[N]/mo |
| [DB if separate] | [Plan] | Database | $[N]/mo |
| Total estimated monthly cost | | | $[N]/mo |
```

---

## Setup: Node.js

```markdown
## Environment Variables (.env.local)
DATABASE_URL=
NEXTAUTH_URL=http://localhost:3000
NEXTAUTH_SECRET=
[API keys from integrations]

## Setup Commands
npm install
npx prisma migrate dev    # if using Prisma
npm run dev
```

---

## Setup: Python FastAPI

```markdown
## Environment Variables (.env)
DATABASE_URL=
SECRET_KEY=
DEBUG=True

## Setup Commands
python -m venv .venv
source .venv/bin/activate    # macOS/Linux
.venv\Scripts\activate       # Windows
pip install -r requirements.txt
alembic upgrade head         # if using Alembic
uvicorn main:app --reload
```

---

## Setup: Python Flask

```markdown
## Environment Variables (.env)
DATABASE_URL=
SECRET_KEY=
DEBUG=True

## Setup Commands
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
flask run
```

---

## Setup: Python Django

```markdown
## Environment Variables (.env)
DATABASE_URL=
SECRET_KEY=
DEBUG=True

## Setup Commands
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python manage.py migrate
python manage.py runserver
```

---

## Setup: Electron

```markdown
## Setup Commands
npm install
npm run electron:dev     # development with hot-reload
npm run electron:build   # build distributable installer
```

---

## Setup: Tauri

```markdown
## Prerequisites
Rust installed via rustup
macOS: xcode-select --install
Windows: Visual Studio Build Tools

## Setup Commands
npm install
npm run tauri dev     # development
npm run tauri build   # build distributable
```

---

## Setup: Other (Go, Ruby, Rust, PHP, or unlisted)

Generate setup commands based on the detected language and framework using the standard pattern: install dependencies, configure environment, run dev server. Reference official documentation for exact commands.

```markdown
## Setup Commands
[language-specific install command]
[configure environment]
[run dev server command]
```

Note: "Setup commands for [language/framework] — verify against official documentation."
