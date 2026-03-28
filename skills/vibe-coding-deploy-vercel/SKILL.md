---
name: vibe-coding-deploy-vercel
description: |
  Deploy projects to Vercel. Handles git remote detection, project linking, team selection, authentication (interactive + token-based), preview vs production deployments, environment variable management, and domain configuration.
  Use when: (1) vibe-coding-ship detects Vercel as the deployment target,
  (2) User says "deploy to Vercel", "push to Vercel", "create Vercel deployment",
  (3) User says "set up Vercel", "link this project to Vercel", "configure Vercel".
  Supports two auth modes: interactive (vercel login) for local machines, token-based (VERCEL_TOKEN) for CI/automated environments.
  Returns deployment URL and status to vibe-coding-ship on completion.
---

# Vibe Coding — Deploy to Vercel

Full Vercel deployment. Called by `vibe-coding-ship` when target is Vercel.

## Entry Router

```
Python full app (Flask/Django/FastAPI)?
→ STOP. Vercel does NOT support persistent Python servers (no long-running processes, no background workers).
  Options: (1) Migrate to FastAPI serverless functions (limited — see ## Python on Vercel), (2) Use Railway/Render/Fly.io for full server, (3) Use DigitalOcean App Platform.
  Redirect user to appropriate platform before proceeding.

VERCEL_TOKEN in env?
→ READ: ## Token Mode

Otherwise → READ: ## Interactive Mode
```

**Jump directly to the matched mode. Do not read both Token Mode and Interactive Mode.**

---

## Interactive Mode (Local Machine)

```bash
# Step 1: Check CLI
vercel --version
# If missing: npm install -g vercel

# Step 2: Auth
vercel whoami
# If not logged in: vercel login  (opens browser)

# Step 3: Check if linked
cat .vercel/project.json 2>/dev/null
# If exists: already linked, skip to Step 5

# Step 4: Link project
vercel link
# Follow prompts: team, project name, root directory

# Step 5: Check git remote
git remote -v
# No remote? Ask user: "Direct CLI deploy or connect GitHub for auto-deploys?"
```

**Deploy:**
```bash
vercel deploy         # preview (always safe)
vercel deploy --prod  # PRODUCTION — requires user confirmation
```

Always confirm before `--prod`:
```
About to deploy to PRODUCTION. URL: [project].vercel.app. This updates the live site. Confirm? (yes/no)
```

---

## Token Mode (CI/Automated)

```bash
vercel deploy --token=$VERCEL_TOKEN                      # preview
vercel deploy --prod --token=$VERCEL_TOKEN               # production
vercel deploy --prod --token=$VERCEL_TOKEN --scope=$VERCEL_TEAM_ID  # with team
```

Get project/team IDs:
```bash
vercel teams list --token=$VERCEL_TOKEN
cat .vercel/project.json | grep projectId
```

---

## Environment Variables

```bash
vercel env ls                              # list all
vercel env add DATABASE_URL production     # add one env
vercel env add DATABASE_URL preview
vercel env pull .env.local                 # sync Vercel env to local
```

Never commit `.env.local`. Verify `.gitignore` has: `.env.local`, `.env*.local`, `.vercel`.

---

## Common Errors

**Build fails locally but passes:** check Node.js version (`"engines": { "node": "20.x" }` in package.json), env vars missing (`vercel env ls`), wrong build command, package in devDependencies instead of dependencies.

**Function timeout:** add to `vercel.json`: `"functions": { "app/api/**": { "maxDuration": 30 } }`

**Wrong root directory:** `vercel link` to re-link with correct root.

---

## Python on Vercel (Serverless Only)

```python
# api/index.py — FastAPI as serverless function
from fastapi import FastAPI
app = FastAPI()
@app.get("/api/health")
def health(): return {"status": "ok"}
```

```json
// vercel.json
{
  "builds": [{ "src": "api/index.py", "use": "@vercel/python" }],
  "routes": [{ "src": "/api/(.*)", "dest": "api/index.py" }]
}
```
Requirements must be in `requirements.txt` at project root.

---

## Inspect and Monitor

```bash
vercel ls                      # list recent deployments
vercel logs [deployment-url]   # view logs
vercel inspect [url]           # deployment details
vercel alias [url] [domain]    # point domain to deployment
```

---

## Output to vibe-coding-ship

Success:
```
VERCEL DEPLOYMENT COMPLETE
URL: https://[project]-[hash].vercel.app
Production: https://[project].vercel.app  (if --prod)
Status: Ready
```

Failure:
```
VERCEL DEPLOYMENT FAILED
Error: [message]  |  Logs: vercel logs [url]
Fix: [specific fix]
```

Write to progress.txt:
```
DEPLOYMENT:
  platform: vercel  url: [url]
  environment: [preview|production]  status: [success|failed]
```
