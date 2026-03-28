---
name: vibe-coding-db-postgres
description: |
  PostgreSQL patterns — local setup, Supabase, schema design, indexing, RLS, connection pooling,
  migrations, and ORM integration (Prisma, Drizzle, SQLAlchemy, Django ORM, Alembic).
  Best practices sourced from the official Supabase agent-skills postgres guide.
  Use when: (1) vibe-coding-db routes here after detecting PostgreSQL,
  (2) User says "PostgreSQL", "Postgres", "Supabase", "pgvector", "RLS",
  (3) Multi-user app, SaaS, or production backend needing full relational database,
  (4) Python backend using psycopg2, asyncpg, SQLAlchemy, or Django ORM.
  Covers: local setup, Supabase cloud setup, schema design, all index types, row-level security,
  connection pooling with PgBouncer, migrations, N+1 prevention, pagination, upserts.
---

# Vibe Coding — PostgreSQL

Full relational database for multi-user apps, SaaS, and production backends.

## Entry Router

```
Supabase detected (supabase in package.json, SUPABASE_URL in env, supabase init run)?
→ Skip ## Local Setup, jump to ## Supabase Setup

Local PostgreSQL (DATABASE_URL points to localhost)?
→ READ: ## Local Setup

ORM detected?
  Prisma → read ## Migrations (Prisma section)
  Drizzle → read ## Migrations (Drizzle section)
  SQLAlchemy/Alembic → read ## Migrations (Alembic section)
  Django ORM → read ## Migrations (Django ORM section)

PostgreSQL extensions detected (pgvector, PostGIS, timescaledb)?
→ Note: This skill covers core PostgreSQL. Extension-specific patterns require consulting the extension's documentation or a dedicated skill. Core setup still proceeds.
```

---

## Local Setup

```bash
# macOS (Homebrew)
brew install postgresql@16
brew services start postgresql@16
psql postgres

# Ubuntu / Debian
sudo apt install postgresql
sudo service postgresql start
sudo -u postgres psql

# Windows — download installer from postgresql.org or use WSL
net start postgresql-x64-16

# Create database
createdb myapp
# or inside psql:
CREATE DATABASE myapp;

# Check connection
pg_isready -h localhost
```

**.env.local:**
```
DATABASE_URL="postgresql://localhost:5432/myapp"
# With user/password:
DATABASE_URL="postgresql://user:password@localhost:5432/myapp"
```

## Supabase Setup

```bash
# Install Supabase CLI
npm install -g supabase

# Init local project
supabase init
supabase start     # starts local Postgres + Auth + Storage + Studio

# Or connect to Supabase cloud
# Get connection string from: Project Settings > Database > Connection string
DATABASE_URL="postgresql://postgres:[password]@db.[project-ref].supabase.co:5432/postgres"

# With connection pooler (use for serverless / Vercel):
DATABASE_URL="postgresql://postgres.[project-ref]:[password]@aws-0-[region].pooler.supabase.com:6543/postgres?pgbouncer=true"
```

---

## Schema Design Best Practices

### Primary Keys

```sql
-- Use BIGINT GENERATED ALWAYS AS IDENTITY (not serial, not UUID v4)
CREATE TABLE users (
  id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  email TEXT NOT NULL UNIQUE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- UUIDv7 if you need globally unique IDs (requires pg_uuidv7 extension)
CREATE EXTENSION IF NOT EXISTS pg_uuidv7;
CREATE TABLE items (
  id UUID DEFAULT uuid_generate_v7() PRIMARY KEY
);
```

**Why not UUIDv4?** Random UUIDs fragment the B-tree index — new rows don't land near related rows, causing cache misses. UUIDv7 is time-ordered and behaves like a BIGINT from an index locality standpoint.

### Data Types

```sql
-- Use these types
BIGINT          -- not INT (INT overflows at 2B rows)
TEXT            -- not VARCHAR(n), no performance difference, no length overhead
TIMESTAMPTZ     -- not TIMESTAMP (always store with timezone)
BOOLEAN         -- not VARCHAR or INT
NUMERIC(p,s)    -- not FLOAT (imprecise, never use for money)

-- Constraints
email TEXT NOT NULL CHECK (email ~* '^[^@]+@[^@]+\.[^@]+$'),
status TEXT NOT NULL CHECK (status IN ('active', 'inactive', 'pending')),
price NUMERIC(10,2) NOT NULL CHECK (price >= 0)
```

### Foreign Keys — Always Index Them

PostgreSQL does NOT auto-index foreign key columns. Missing FK indexes cause full table scans on every JOIN and CASCADE operation.

```sql
-- ALWAYS add this index when you add an FK
ALTER TABLE posts ADD COLUMN author_id BIGINT NOT NULL REFERENCES users(id);
CREATE INDEX idx_posts_author_id ON posts(author_id);   -- <-- required

-- Find missing FK indexes (run this diagnostic)
SELECT
  tc.table_name, kcu.column_name, ccu.table_name AS foreign_table
FROM information_schema.table_constraints tc
JOIN information_schema.key_column_usage kcu ON tc.constraint_name = kcu.constraint_name
JOIN information_schema.constraint_column_usage ccu ON tc.constraint_name = ccu.constraint_name
WHERE tc.constraint_type = 'FOREIGN KEY'
  AND NOT EXISTS (
    SELECT 1 FROM pg_indexes
    WHERE tablename = tc.table_name
    AND indexdef LIKE '%' || kcu.column_name || '%'
  );
```

---

## Index Types

| Index | Use for | Example |
|-------|---------|---------|
| B-tree (default) | =, <, >, BETWEEN, LIKE 'prefix%' | `CREATE INDEX ON users(email)` |
| GIN | JSONB, arrays, full-text search | `CREATE INDEX ON items USING GIN(tags)` |
| GiST | Geometric, range, K-nearest neighbor | `CREATE INDEX ON locations USING GIST(point)` |
| BRIN | Very large time-series tables (100M+ rows) | `CREATE INDEX ON events USING BRIN(created_at)` |
| Hash | Equality only, no range scans | rarely needed — B-tree covers this |

```sql
-- Composite index — equality columns first, range column last
CREATE INDEX idx_posts_status_created ON posts(status, created_at DESC);

-- Partial index — only index active rows (smaller, faster)
CREATE INDEX idx_users_active ON users(email) WHERE status = 'active';

-- Covering index — include extra columns to avoid table lookup
CREATE INDEX idx_orders_customer ON orders(customer_id)
  INCLUDE (status, total_amount);

-- GIN for JSONB
CREATE INDEX idx_items_metadata ON items USING GIN(metadata);
-- Query: SELECT * FROM items WHERE metadata @> '{"category": "electronics"}';

-- Full-text search
ALTER TABLE posts ADD COLUMN fts_vector TSVECTOR
  GENERATED ALWAYS AS (to_tsvector('english', title || ' ' || coalesce(content, ''))) STORED;
CREATE INDEX idx_posts_fts ON posts USING GIN(fts_vector);
-- Query: SELECT * FROM posts WHERE fts_vector @@ to_tsquery('english', 'postgresql & index');

-- Check if indexes are used
EXPLAIN (ANALYZE, BUFFERS) SELECT * FROM posts WHERE status = 'active' AND created_at > NOW() - INTERVAL '7 days';
-- Look for "Index Scan" or "Bitmap Index Scan" — "Seq Scan" means no index hit
```

---

## Row-Level Security (RLS)

RLS enforces tenant isolation at the database level — every query is automatically filtered.

```sql
-- Enable RLS on a table
ALTER TABLE posts ENABLE ROW LEVEL SECURITY;
ALTER TABLE posts FORCE ROW LEVEL SECURITY;   -- applies to table owner too

-- Policy: users can only see their own posts
CREATE POLICY "users_own_posts" ON posts
  FOR ALL
  USING (author_id = (SELECT auth.uid()))   -- (SELECT auth.uid()) evaluated once, not per row
  WITH CHECK (author_id = (SELECT auth.uid()));

-- Policy: published posts are public
CREATE POLICY "published_posts_public" ON posts
  FOR SELECT
  USING (published = TRUE);

-- Index on RLS column (critical for performance)
CREATE INDEX idx_posts_author_id ON posts(author_id);
```

**RLS performance rule:** Always use `(SELECT auth.uid())` not `auth.uid()` directly. The subquery form is evaluated once per query; the function form is called per row — 5-10x slower on large tables.

---

## Connection Pooling

PostgreSQL is not designed for thousands of simultaneous connections. Use PgBouncer or the Supabase connection pooler.

```bash
# Supabase — use the pooler URL (transaction mode) for serverless
DATABASE_URL="postgresql://postgres.[ref]:[pass]@pooler.supabase.com:6543/postgres?pgbouncer=true"

# Local PgBouncer
# pool_size = (CPU cores * 2) + spindle count
# For a 4-core machine: pool_size = 9
```

**Prisma with connection pooler:**
```
DATABASE_URL="postgresql://...?pgbouncer=true&connection_limit=1"
```
Set `connection_limit=1` per serverless function instance — PgBouncer manages the actual pool.

**Max connections formula:**
```sql
-- Check your current max
SHOW max_connections;

-- Recommended formula: (RAM in MB / 5) - reserved_for_superuser
-- 1GB RAM → (1024/5) - 3 = ~202 max connections
-- Practical: keep actual connections to max_connections * 0.8
```

---

## Migrations

### Prisma

```bash
npx prisma migrate dev --name add_posts_table   # dev: create + apply
npx prisma migrate deploy                        # prod: apply pending only
npx prisma db push                               # dev: push schema without migration file
npx prisma migrate reset                         # wipe and reapply all (dev only)
```

### Drizzle

```bash
npx drizzle-kit generate    # generate SQL from schema
npx drizzle-kit migrate     # apply to database
npx drizzle-kit push        # push without migration file (dev only)
```

### Alembic (Python / SQLAlchemy)

```bash
alembic init alembic           # init alembic directory
alembic revision --autogenerate -m "add posts table"   # generate from models
alembic upgrade head           # apply all pending
alembic downgrade -1           # rollback one step
alembic history                # show migration history
```

### Django ORM

```bash
python manage.py makemigrations    # generate migration files from models
python manage.py migrate           # apply
python manage.py showmigrations    # list applied/pending
python manage.py sqlmigrate app 0001   # preview SQL
```

---

## N+1 Prevention

```sql
-- BAD: 1 query for list + 1 query per row = N+1
-- SELECT * FROM posts         -- 1 query
-- SELECT * FROM users WHERE id = ?   -- once per post

-- GOOD: single JOIN
SELECT p.*, u.name AS author_name
FROM posts p
JOIN users u ON u.id = p.author_id
WHERE p.published = TRUE
ORDER BY p.created_at DESC
LIMIT 20;
```

**Prisma — use `include`:**
```typescript
const posts = await prisma.post.findMany({
  where: { published: true },
  include: { author: { select: { name: true, email: true } } },
  orderBy: { createdAt: "desc" },
  take: 20,
});
```

---

## Pagination

```sql
-- BAD: OFFSET scans all skipped rows — O(n) cost
SELECT * FROM posts ORDER BY created_at DESC LIMIT 20 OFFSET 10000;

-- GOOD: keyset/cursor pagination — O(1) regardless of position
SELECT * FROM posts
WHERE (created_at, id) < (:last_created_at, :last_id)
ORDER BY created_at DESC, id DESC
LIMIT 20;
```

**Prisma cursor pagination:**
```typescript
const posts = await prisma.post.findMany({
  take: 20,
  skip: 1,
  cursor: { id: lastId },
  orderBy: { createdAt: "desc" },
});
```

---

## Upsert

```sql
-- Safe upsert — no race conditions
INSERT INTO users (email, name, updated_at)
VALUES ($1, $2, NOW())
ON CONFLICT (email) DO UPDATE SET
  name = EXCLUDED.name,
  updated_at = NOW()
RETURNING *;
```

---

## Monitoring

```sql
-- Slow queries (requires pg_stat_statements extension)
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

SELECT query, calls, total_exec_time::int, mean_exec_time::int
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;

-- Table bloat / vacuum status
SELECT relname, n_dead_tup, last_vacuum, last_autovacuum
FROM pg_stat_user_tables
ORDER BY n_dead_tup DESC;

-- Active connections
SELECT state, count(*) FROM pg_stat_activity GROUP BY state;

-- Running queries longer than 30s
SELECT pid, now() - pg_stat_activity.query_start AS duration, query
FROM pg_stat_activity
WHERE state = 'active' AND now() - query_start > interval '30 seconds';
```

---

## ORM Setup Cheatsheet

| ORM | Language | Install |
|-----|----------|---------|
| Prisma | TypeScript | `npm install prisma @prisma/client` |
| Drizzle | TypeScript | `npm install drizzle-orm postgres` + `npm i -D drizzle-kit` |
| SQLAlchemy | Python | `pip install sqlalchemy psycopg2-binary` |
| Alembic | Python | `pip install alembic` (use with SQLAlchemy) |
| Django ORM | Python | Built-in — `pip install django psycopg2-binary` |
| asyncpg | Python (async) | `pip install asyncpg` (raw async queries) |

---

## Progress.txt Update

After PostgreSQL setup is complete, append these fields to the active BUILD phase block in progress.txt:

```
PHASE: BUILD
STATUS: in_progress
database: postgres
orm: [prisma|drizzle|sqlalchemy|django|none]
schema_complete: true
migrations_run: true
connection_pooling: [true|false]
timestamp: [ISO timestamp]
```

`vibe-coding-build` reads `schema_complete` and `migrations_run` to confirm the database step is done before proceeding to the next IMPLEMENTATION_PLAN step.
