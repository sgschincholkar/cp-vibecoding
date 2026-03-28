---
name: vibe-coding-cli-runner
description: |
  CLI tool integration patterns — safe shell command execution, output parsing, piping, error code handling, environment setup, and scripting within generated projects.
  Use when: (1) BUILD step requires running a CLI tool (database migrations, code generators, build scripts, deployment tools),
  (2) User says "run [cli tool]", "execute [command]", "set up [tool] CLI", "add [tool] scripts",
  (3) IMPLEMENTATION_PLAN step involves CLI setup (Prisma, Drizzle migrations, shadcn/ui add, stripe CLI, aws CLI, docker, etc.),
  (4) User says "generate migrations", "scaffold components", "run codegen", "add CLI scripts to package.json".
  Outputs safe command patterns, npm scripts, and output parsing code.
---

# Vibe Coding — CLI Runner

Generate safe, correct CLI usage patterns for dev tools, databases, and scaffolding.

## Entry Router

```
Detect tool category from BUILD context or user message:

Database ORM (Prisma/Drizzle/Alembic/Django) → READ: ## Database CLIs
UI components (shadcn/ui/Radix) → READ: ## UI Component CLIs
Code generation (GraphQL Codegen/OpenAPI) → READ: ## Code Generation
Stripe CLI → READ: ## Stripe CLI
AWS CLI → READ: ## AWS CLI
Docker → READ: ## Docker
Python project → READ: ## Python CLIs
Node.js output parsing → READ: ## Output Parsing
Tool not in list above → READ: ## Generic CLI Pattern
```

Safety rules apply to ALL commands:
- NEVER: `rm -rf` with variables, commands from untrusted input, `sudo` without confirmation, force flags without warning
- ALWAYS: use `--` separator, quote paths with spaces, `npx` for one-off packages

**Jump directly to the matched ## [Tool] section. Do not read all CLI sections — only the one matching the detected tool category.**

---

## Database CLIs

**Prisma:**
```bash
npx prisma init --datasource-provider postgresql
npx prisma generate          # regenerate client types
npx prisma db push           # push schema to dev DB (no migration file)
npx prisma migrate dev --name [description]  # create + apply migration (always run generate after)
npx prisma migrate deploy    # apply pending (production)
npx prisma studio            # visual DB browser
npx prisma migrate reset     # DESTRUCTIVE — drops and recreates DB
```
```json
"db:generate": "prisma generate",
"db:push": "prisma db push",
"db:migrate": "prisma migrate dev && prisma generate",
"db:studio": "prisma studio",
"db:reset": "prisma migrate reset"
```

**Drizzle:**
```bash
npx drizzle-kit generate && npx drizzle-kit migrate
npx drizzle-kit push   # dev only (skip migration file)
npx drizzle-kit studio
```

**Alembic (SQLAlchemy):**
```bash
alembic init alembic
alembic revision --autogenerate -m "[description]"
alembic upgrade head
alembic downgrade -1   # rollback one
alembic current
```

**Django:**
```bash
python manage.py makemigrations && python manage.py migrate
python manage.py runserver
python manage.py createsuperuser
python manage.py collectstatic --noinput
```

---

## UI Component CLIs

**shadcn/ui:** Files are copied to `components/ui/` — treat as yours to edit.
```bash
npx shadcn@latest init
npx shadcn@latest add button
npx shadcn@latest add button card input dialog  # multiple at once
```

**Radix UI (direct):**
```bash
npm install @radix-ui/react-dialog @radix-ui/react-dropdown-menu
```

---

## Code Generation

**GraphQL Codegen:**
```bash
npm install --save-dev @graphql-codegen/cli @graphql-codegen/typescript @graphql-codegen/typescript-operations
npx graphql-codegen --config codegen.ts
```
```json
"codegen": "graphql-codegen --config codegen.ts",
"codegen:watch": "graphql-codegen --config codegen.ts --watch"
```

**OpenAPI TypeScript:**
```bash
npx openapi-typescript ./openapi.yaml -o ./types/api.d.ts
# or from URL:
npx openapi-typescript https://api.example.com/openapi.json -o ./types/api.d.ts
```

---

## Stripe CLI

```bash
brew install stripe/stripe-cli/stripe  # macOS
stripe login
stripe listen --forward-to localhost:3000/api/webhooks/stripe
stripe trigger payment_intent.succeeded
```
```json
"stripe:listen": "stripe listen --forward-to localhost:3000/api/webhooks/stripe"
```

---

## AWS CLI

```bash
aws configure
aws s3 ls s3://[bucket]
aws s3 sync ./dist s3://[bucket] --delete
aws sts get-caller-identity  # verify current identity
```
Never hardcode credentials — use env vars: `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_REGION`.

---

## Docker

```bash
docker build -t [app]:[tag] .
docker run -p 3000:3000 --env-file .env.local [app]:[tag]
docker compose up -d
docker compose down        # stop
docker compose down -v     # DESTRUCTIVE — removes volumes
docker compose logs -f
```

Next.js Dockerfile pattern: multi-stage build (deps → builder → runner). Use `standalone` output.

---

## Python CLIs

```bash
# Venv setup
python -m venv .venv && source .venv/bin/activate  # macOS/Linux
python -m venv .venv && .venv\Scripts\activate      # Windows
pip install -r requirements.txt

# pytest
pytest -v --tb=short
pytest --cov=src --cov-report=term-missing
pytest --lf  # rerun only failed tests

# FastAPI dev
uvicorn main:app --reload --port 8000
gunicorn main:app -w 4 -k uvicorn.workers.UvicornWorker  # production
```

Makefile convention for Python (instead of npm scripts):
```makefile
.PHONY: dev test migrate lint
dev: uvicorn main:app --reload
test: pytest -v --tb=short
migrate: alembic upgrade head
lint: ruff check . && mypy .
```

---

## Output Parsing

```typescript
import { execSync, spawn } from 'child_process'

// Sync (simple build steps)
function runCommand(cmd: string): string {
  try { return execSync(cmd, { encoding: 'utf-8', stdio: ['pipe', 'pipe', 'pipe'] }).trim() }
  catch (e: any) { throw new Error(`Command failed: ${cmd}\n${e.stderr}`) }
}

// Async streaming (long-running commands)
function runCommandStream(command: string, args: string[]): Promise<void> {
  return new Promise((resolve, reject) => {
    const proc = spawn(command, args, { stdio: 'inherit' })
    proc.on('close', code => code === 0 ? resolve() : reject(new Error(`${command} exited ${code}`)))
  })
}
```

Exit codes: 0=success, 1=general error, 127=command not found (install or check PATH), EACCES=permissions, ENOENT=path not found.

---

## Generic CLI Pattern

For tools not in the categories above:

1. Identify the command structure: `[tool] [subcommand] [flags] [args]`
2. Apply safety checklist:
   - Does any flag destroy data? (reset, drop, delete, purge, clean) → warn user before including
   - Does it require authentication? → note env var or login step
   - Is it idempotent? → note if it can be safely re-run
3. Generate npm script wrapper:
   ```json
   "[tool]:[action]": "[full command here]"
   ```
4. Output format:
   ```
   CLI SETUP: [Tool Name]
   =======================
   Commands to run:
   1. [command]
   Safety notes: [any destructive flags or auth requirements]
   npm script: [json]
   Verification: [how to confirm it worked]
   ```

---

## npm Script Conventions

```json
"dev": "[framework dev cmd]",
"build": "[framework build cmd]",
"lint": "[linter cmd]",
"type-check": "tsc --noEmit",
"check": "npm run type-check && npm run lint",
"ci": "npm run check && npm run build"
```
Naming: `[tool]:[action]` for tool-specific scripts (`db:migrate`, `stripe:listen`).

---

## Output Format

```
CLI SETUP: [Tool Name]
=======================
Commands to run now:
1. [command 1]
2. [command 2]

npm scripts to add: [json block]
Files generated/modified: [list]
Verification: Run [check cmd] — expected: [what success looks like]
```
