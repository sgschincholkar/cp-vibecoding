---
name: vibe-coding-db
description: |
  Database skill coordinator. Routes to the right per-database skill based on detected or chosen database.
  Covers schema design, migrations, queries, indexing, connection management, and security for all supported databases.
  Supported databases: SQLite, BetterSQLite3, PostgreSQL (local + Supabase), DuckDB, Convex.
  Use when: (1) User says "set up database", "design schema", "write migration", "query the database",
  (2) vibe-coding-build hits a database step in IMPLEMENTATION_PLAN,
  (3) User says "which database should I use", "help with [database name]",
  (4) vibe-coding-doc-backend needs schema or query guidance,
  (5) User says "optimize query", "add index", "fix N+1", "connection pool".
  Do NOT trigger for: API integration patterns (vibe-coding-api-connect), CLI migration commands
  only (vibe-coding-cli-runner handles those).
  Reads run_target and stack from progress.txt to determine context (local vs cloud, Node vs Python).
---

# Vibe Coding — Database

Route to the right database skill. Never implement database logic directly — always delegate.

## Skill Map

```
vibe-coding-db
  |-- SQLite (file-based, zero config)      --> vibe-coding-db-sqlite
  |-- BetterSQLite3 (Node.js, synchronous)  --> vibe-coding-db-bettersqlite
  |-- PostgreSQL (local or Supabase)        --> vibe-coding-db-postgres
  |-- DuckDB (analytics, OLAP)              --> vibe-coding-db-duckdb
  |-- Convex (reactive, serverless)         --> vibe-coding-db-convex
```

## Detection and Routing

**Run Steps 1-3 in order. Route and delegate as soon as the database is identified — do not read the decision table or sub-skill content before routing.**

### Step 1 — Detect from progress.txt

Read `progress.txt` for `database:` field set during IDEATE. Route immediately if found.

### Step 2 — Detect from project files

If no progress.txt or database field is missing:

```
package.json contains "better-sqlite3"       → vibe-coding-db-bettersqlite
package.json contains "convex"               → vibe-coding-db-convex
package.json contains "duckdb"               → vibe-coding-db-duckdb
requirements.txt contains "psycopg2"         → vibe-coding-db-postgres
requirements.txt contains "asyncpg"         → vibe-coding-db-postgres
*.db file exists in project root             → vibe-coding-db-sqlite
prisma/schema.prisma provider = "sqlite"     → vibe-coding-db-sqlite
prisma/schema.prisma provider = "postgresql" → vibe-coding-db-postgres
```

### Step 3 — Ask if still unclear

```
Which database is this project using?
1. SQLite         — file-based, zero config, great for local and desktop apps
2. BetterSQLite3  — synchronous Node.js SQLite driver (faster than sqlite3 package)
3. PostgreSQL     — full relational DB (local or Supabase)
4. DuckDB         — analytics and OLAP workloads, columnar storage
5. Convex         — reactive serverless backend, TypeScript-native

(1 / 2 / 3 / 4 / 5)
```

## When to Use Which Database

| Database | Best for | Avoid when |
|----------|----------|------------|
| SQLite | Local apps, desktop apps (Electron/Tauri), prototypes, single-user tools | High concurrency writes, multiple processes writing simultaneously |
| BetterSQLite3 | Node.js apps needing synchronous SQLite access, Electron, CLI tools | Async-first environments, high concurrency |
| PostgreSQL | Multi-user apps, production SaaS, complex queries, full-text search | Purely local desktop apps with no concurrent writes |
| DuckDB | Analytics dashboards, CSV/Parquet processing, OLAP queries, data science | OLTP (transactional writes), production web apps as primary DB |
| Convex | Real-time apps, collaborative tools, TypeScript-native backends, AI apps | Python backends, offline-first apps, projects avoiding vendor lock-in |

## Common Tasks — Routing Table

| Task | Routes to |
|------|-----------|
| "Design schema" | Per-database skill → schema section |
| "Write migration" | Per-database skill → migrations section |
| "Optimize slow query" | Per-database skill → query optimization / indexing |
| "Set up connection pool" | Per-database skill → connection management |
| "Add full-text search" | vibe-coding-db-postgres (pg) or vibe-coding-db-convex (Convex search) |
| "Enable RLS / row-level security" | vibe-coding-db-postgres |
| "Real-time subscriptions" | vibe-coding-db-convex |
| "Run analytics on CSV" | vibe-coding-db-duckdb |
| "Store files / blobs" | vibe-coding-db-convex (file storage) or PostgreSQL large objects |

## ORM Awareness

The database skills are ORM-aware. When a project uses an ORM, the skill adapts:

| ORM | Detected by | Adapts in |
|-----|-------------|-----------|
| Prisma | `prisma/schema.prisma` exists | db-sqlite, db-postgres |
| Drizzle | `drizzle.config.ts` exists | db-sqlite, db-postgres |
| SQLAlchemy | `requirements.txt` contains `sqlalchemy` | db-postgres |
| Alembic | `alembic.ini` exists | db-postgres |
| Django ORM | `manage.py` exists | db-postgres |

When ORM is detected, the skill outputs ORM-native patterns (Prisma schema, Drizzle schema, SQLAlchemy models) rather than raw SQL — unless the user explicitly asks for raw SQL.

If ORM detected is not in the table above (e.g., TypeORM, Sequelize, Peewee, Tortoise ORM) → output: "ORM detected: [name]. This skill natively covers Prisma, Drizzle, SQLAlchemy, Alembic, and Django ORM. Your ORM may require adaptation — patterns below use the closest equivalent. Consult your ORM's documentation for specifics." Then proceed with database routing using raw SQL patterns as the fallback.

## Progress Tracking

Database setup fields are appended to the active BUILD phase block in progress.txt — not as a separate phase:

```
PHASE: BUILD
STATUS: in_progress
database: [sqlite|bettersqlite|postgres|duckdb|convex]
orm: [prisma|drizzle|sqlalchemy|django|none]
schema_complete: true
migrations_run: true
timestamp: [ISO timestamp]
```

Write these fields after the per-database skill completes setup. `vibe-coding-build` reads `schema_complete` and `migrations_run` to confirm the database step is done before continuing to the next IMPLEMENTATION_PLAN step.
