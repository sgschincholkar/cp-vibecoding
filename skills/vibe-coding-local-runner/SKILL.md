---
name: vibe-coding-local-runner
description: |
  Local development and local "deployment" — running apps from the terminal on your own machine.
  Covers: starting dev servers, running production builds locally, environment setup, process management, database startup, and port configuration.
  Frameworks: Next.js, React (Vite/CRA), Flask, Django, FastAPI, Express, and static sites.
  Databases: SQLite, BetterSQLite3, PostgreSQL (local), Convex (local dev).
  Use when: (1) User says "run locally", "start the app", "how do I run this", "start the dev server",
  (2) vibe-coding-ship detects run_target is local machine and delegates here,
  (3) User says "test before deploying", "run production build locally", "start database",
  (4) User wants to run an Electron or Tauri desktop app from terminal.
  Role distinction: local-runner = executes startup commands. vibe-coding-ship = phase coordinator that delegates here.
  Focus: terminal commands, process management, env file setup, port conflicts.
---

# Vibe Coding — Local Runner

Start your app locally. Detect framework and show exact startup sequence.

## Entry Router

```
Detect framework from TECH_STACK.md or package.json/requirements.txt:

Next.js → READ: ## Next.js
Vite + React → READ: ## Vite
Flask → READ: ## Flask
Django → READ: ## Django
FastAPI → READ: ## FastAPI
Express/Node.js → READ: ## Express
Electron → READ: ## Electron
Tauri → READ: ## Tauri
Go (net/http / Gin / Echo / Fiber) → READ: ## Go
Ruby on Rails → READ: ## Ruby on Rails
Rust (Axum / Actix / Warp) → READ: ## Rust
PHP (Laravel / Symfony / plain) → READ: ## PHP
Multiple services → READ: ## Multiple Services

Port conflict troubleshooting → READ: ## Port Conflicts
```

**Jump directly to the matched framework section. Do not read all framework sections.**

---

## Next.js

```bash
npm run dev                    # dev server at http://localhost:3000
npm run dev -- --port 3001    # custom port
npm run build && npm run start # production build (local test)
PORT=3001 npm run start        # production on custom port
cp .env.example .env.local     # env setup (fill in real values)
```

Env file load order (later overrides earlier): `.env` → `.env.local` → `.env.development` / `.env.production`

---

## Vite

```bash
npm run dev          # dev at http://localhost:5173
npm run build        # outputs to dist/
npm run preview      # serve dist/ at http://localhost:4173
```
Env: prefix public vars with `VITE_` in `.env.local`.

---

## Flask

```bash
python -m venv venv && source venv/bin/activate  # macOS/Linux
python -m venv venv && venv\Scripts\activate      # Windows
pip install -r requirements.txt
flask run                  # http://localhost:5000
flask run --port 5001      # custom port
```

`.env`:
```
FLASK_APP=app.py
FLASK_ENV=development
SECRET_KEY=dev-secret-change-in-production
DATABASE_URL=sqlite:///app.db
```
Needs `pip install python-dotenv` + `from dotenv import load_dotenv; load_dotenv()` in app.py.

---

## Django

```bash
python -m venv venv && source venv/bin/activate
pip install -r requirements.txt
python manage.py migrate          # always run before starting
python manage.py runserver        # http://localhost:8000
python manage.py runserver 8001   # custom port
python manage.py createsuperuser  # first run
```

`.env`: `SECRET_KEY=`, `DEBUG=True`, `DATABASE_URL=sqlite:///db.sqlite3`, `ALLOWED_HOSTS=localhost,127.0.0.1`

---

## FastAPI

```bash
pip install -r requirements.txt
uvicorn main:app --reload              # http://localhost:8000, docs at /docs
uvicorn main:app --reload --port 8001  # custom port
uvicorn main:app --host 0.0.0.0        # accessible on local network
```

---

## Express

```bash
npm install
npm run dev   # nodemon with auto-restart
PORT=4000 node server.js  # custom port
```
```json
"dev": "nodemon server.js",
"start": "node server.js"
```

---

## Electron

```bash
npm install
npm run electron:dev   # hot-reload dev mode
npm run electron:build # outputs installer to dist/
```
```json
"electron:dev": "concurrently \"vite\" \"electron .\"",
"electron:build": "vite build && electron-builder"
```
electron-builder config in package.json `"build"`: appId, mac/win/linux targets.

---

## Tauri

```bash
# Prerequisites: Rust via rustup, xcode-select (macOS), VS Build Tools (Windows)
npm install
npm run tauri dev    # starts Vite + Tauri window
npm run tauri build  # outputs to src-tauri/target/release/bundle/
```

---

## Go

```bash
go mod tidy               # install/sync dependencies
go run main.go            # run directly (or go run ./cmd/...)
go run . --port 8080      # custom port (if app accepts flag)
go build -o app && ./app  # compiled binary

# With live reload (install air first: go install github.com/cosmtrek/air@latest)
air
```
Default ports: 8080 (Gin/Echo/Fiber), 8000 (net/http default pattern).
Env: create `.env` and load via `godotenv` or read `os.Getenv` directly.

---

## Ruby on Rails

```bash
bundle install                     # install gems
bin/rails db:create db:migrate     # set up database
bin/rails server                   # http://localhost:3000
bin/rails server -p 4000           # custom port
bin/rails console                  # REPL for debugging
```
Env: use `.env` file with `dotenv-rails` gem, or set via `credentials.yml.enc`.

---

## Rust

```bash
cargo build                        # compile
cargo run                          # build and run
cargo run -- --port 8080           # args if app accepts
cargo watch -x run                 # live reload (install: cargo install cargo-watch)
```
Default ports: Axum/Actix typically 3000 or 8080. Check main.rs for the bind address.

---

## PHP

```bash
# Laravel
composer install
cp .env.example .env
php artisan key:generate
php artisan migrate
php artisan serve                   # http://localhost:8000
php artisan serve --port=8001       # custom port

# Symfony
composer install
symfony server:start                # http://localhost:8000
php bin/console doctrine:migrations:migrate

# Plain PHP (built-in server)
php -S localhost:8000 -t public/    # serve public/ directory
```

---

## Multiple Services

```bash
# Option 1: separate terminals
Terminal 1: npm run dev
Terminal 2: uvicorn main:app --reload
Terminal 3: npx convex dev

# Option 2: concurrently (one terminal)
npm install --save-dev concurrently
```
```json
"dev:all": "concurrently \"npm run dev\" \"uvicorn main:app --reload\" --names \"frontend,backend\""
```

---

## Database Startup

**SQLite/BetterSQLite3:** no server — file-based. `npx prisma db push` creates the file.

**PostgreSQL (local):**
```bash
brew services start postgresql@15  # macOS
sudo service postgresql start       # Ubuntu
net start postgresql-x64-15         # Windows
createdb [your-app-name]
pg_isready -h localhost             # verify running
```
Connection: `DATABASE_URL=postgresql://localhost:5432/[your-app-name]`

**Convex:** `npx convex dev` (dashboard at localhost:3210)

---

## Port Conflicts

```bash
# Find what's on port 3000
lsof -i :3000          # macOS/Linux
netstat -ano | findstr :3000  # Windows

# Kill by PID
kill -9 [PID]          # macOS/Linux
taskkill /PID [PID] /F # Windows

# Or just use different port: npm run dev -- -p 3001
```

Port defaults: Next.js=3000, Vite=5173, Flask=5000, Django/FastAPI=8000, Express=3000, PostgreSQL=5432, Convex=3210.

---

## Output Format

```
LOCAL RUNNER
============
Framework: [detected]  |  Database: [detected]

STARTUP SEQUENCE:
1. [first command]
2. [second command if needed]

App at: http://localhost:[port]
[Additional URLs: API docs at /docs, DB studio at localhost:5555, etc.]

Checklist:
[ ] .env.local exists with required values
[ ] Database running (if applicable)
[ ] Dependencies installed
```

After confirmed running, write to progress.txt:
```
LOCAL_RUNNER:
  framework: [name]
  start_command: [cmd]
  port: [port]
  status: running
```
