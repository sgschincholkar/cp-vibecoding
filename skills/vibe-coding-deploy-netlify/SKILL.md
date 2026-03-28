---
name: vibe-coding-deploy-netlify
description: |
  Deploy projects to Netlify. Handles site creation, git connection, build settings, environment variables, redirects, and domain configuration.
  Use when: (1) vibe-coding-ship detects Netlify as the deployment target,
  (2) User says "deploy to Netlify", "push to Netlify", "host on Netlify",
  (3) User says "set up Netlify", "configure Netlify", "create Netlify site".
  Supports interactive (netlify login) and token-based (NETLIFY_AUTH_TOKEN) auth modes.
  Returns deployment URL and status to vibe-coding-ship on completion.
---

# Vibe Coding — Deploy to Netlify

Full Netlify deployment. Called by `vibe-coding-ship` when target is Netlify.

## Entry Router

```
Python full app (Flask/Django/FastAPI)?
→ STOP. Netlify does NOT support persistent Python servers (no long-running processes, no background workers).
  Options: (1) Serverless Python functions only (limited — netlify/functions/*.py), (2) Use Railway/Render/Fly.io for full server, (3) Use DigitalOcean App Platform.
  Redirect user to appropriate platform before proceeding.

NETLIFY_AUTH_TOKEN in env?
→ READ: ## Token Mode

Otherwise → READ: ## Interactive Mode
```

**Jump directly to the matched mode. Do not read both Token Mode and Interactive Mode.**

---

## Interactive Mode (Local Machine)

```bash
# Step 1: Check CLI
netlify --version
# If missing: npm install -g netlify-cli

# Step 2: Auth
netlify login  # opens browser

# Step 3: Check if linked
cat .netlify/state.json 2>/dev/null
# If siteId exists: already linked, skip to Step 5

# Step 4: Init/link
netlify init    # new site or connect existing
netlify link    # link to existing by name, ID, or git remote
```

Build settings by framework:
- Next.js: build cmd `next build`, publish `.next`, install `@netlify/plugin-nextjs`
- Vite: `npm run build`, publish `dist`
- CRA: `npm run build`, publish `build`

**Deploy:**
```bash
netlify deploy        # draft URL (safe)
netlify deploy --prod # PRODUCTION — requires user confirmation
```

Confirm before `--prod`: "About to deploy to PRODUCTION. Site: [name].netlify.app. Confirm? (yes/no)"

---

## Token Mode (CI/Automated)

```bash
export NETLIFY_AUTH_TOKEN=your_token
export NETLIFY_SITE_ID=your_site_id   # from: netlify sites:list or cat .netlify/state.json

netlify deploy --auth=$NETLIFY_AUTH_TOKEN --site=$NETLIFY_SITE_ID
netlify deploy --prod --auth=$NETLIFY_AUTH_TOKEN --site=$NETLIFY_SITE_ID
```

---

## Environment Variables

```bash
netlify env:list
netlify env:set DATABASE_URL "postgresql://..."
netlify env:import .env.production    # bulk import
netlify env:unset VAR_NAME
```

---

## netlify.toml Reference

```toml
[build]
  command = "npm run build"
  publish = ".next"   # or "dist" (Vite) / "build" (CRA)

[build.environment]
  NODE_VERSION = "20"

# SPA redirect (React Router, Next.js static)
[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200

# API proxy to backend
[[redirects]]
  from = "/api/*"
  to = "https://your-backend.com/api/:splat"
  status = 200
  force = true

# Next.js plugin
[[plugins]]
  package = "@netlify/plugin-nextjs"
```

---

## Common Errors

**"next: command not found":** move `next` from devDependencies to dependencies.

**SPA redirects broken with React Router v6 nested routes:** verify `[[redirects]] from = "/*"` is the LAST rule in netlify.toml (Netlify processes rules top-down and stops at first match). Alternatively use a `_redirects` file at the publish root: `/* /index.html 200`

**Functions not deploying:** check `netlify/functions/` exists, function exports a `handler`, and netlify.toml `[build]` points to correct directory.

**Env vars missing at build:** `netlify env:list` → verify → `netlify deploy --build`.

---

## Serverless Functions

```javascript
// netlify/functions/hello.js
export const handler = async (event, context) => {
  return { statusCode: 200, body: JSON.stringify({ message: 'Hello' }) }
}
// Access at: /.netlify/functions/hello
```

---

## Domain Configuration

```bash
netlify domains:add yourdomain.com
netlify dns:list
```

---

## Output to vibe-coding-ship

Success:
```
NETLIFY DEPLOYMENT COMPLETE
Draft URL: https://[hash]--[site].netlify.app
Production URL: https://[site].netlify.app  (if --prod)
Status: Published
```

Failure:
```
NETLIFY DEPLOYMENT FAILED
Error: [message]  |  Debug: netlify deploy --debug
Fix: [specific fix]
```

Write to progress.txt:
```
DEPLOYMENT:
  platform: netlify  url: [url]
  environment: [draft|production]  status: [success|failed]
```
