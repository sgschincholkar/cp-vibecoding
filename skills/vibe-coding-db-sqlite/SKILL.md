---
name: vibe-coding-db-sqlite
description: |
  SQLite database patterns — schema design, migrations, queries, and Prisma/Drizzle ORM integration.
  File-based, zero-config database ideal for local apps, desktop apps, prototypes, and single-user tools.
  Use when: (1) vibe-coding-db routes here after detecting SQLite,
  (2) User says "use SQLite", "file-based database", "no database server",
  (3) Project is Electron/Tauri desktop app,
  (4) run_target is local or desktop and no database server is desired.
  Covers: schema creation, raw SQL queries, Prisma with SQLite provider, Drizzle with SQLite,
  migrations, WAL mode, pragmas for performance, and common pitfalls.
---

# Vibe Coding — SQLite

File-based database, zero config, no server needed. The `.db` file IS the database.

## Entry Router

```
ORM detected in project?
  Prisma → READ: ## Prisma with SQLite
  Drizzle → READ: ## Drizzle with SQLite
  No ORM / raw SQL → READ: ## Raw SQLite

After setup: READ: ## Performance Pragmas (always apply)
Slow queries? → READ: ## Indexing
Migrations without ORM? → READ: ## Migrations
Electron/Tauri? → READ: ## .env Setup (uses app data directory, not project root)
```

## When SQLite is the Right Choice

- Local machine apps (run_target: local)
- Desktop apps (Electron / Tauri)
- Prototypes and development environments
- Single-user tools and CLI utilities
- Apps with low write concurrency (one writer at a time)
- Apps where portability matters (ship the .db file)

**Not ideal for:** Multiple processes writing simultaneously, high-concurrency web apps, apps needing PostGIS or advanced PostgreSQL extensions.

---

## Setup

### Raw SQLite (no ORM)

**Node.js — use `sqlite3` package (async) or `better-sqlite3` (sync):**
```bash
npm install sqlite3          # async
npm install better-sqlite3   # sync (recommended for simplicity) → see vibe-coding-db-bettersqlite
```

**Python — built-in, no install needed:**
```python
import sqlite3
conn = sqlite3.connect("app.db")   # creates file if not exists
```

### Prisma with SQLite

```bash
npm install prisma @prisma/client
npx prisma init --datasource-provider sqlite
```

`prisma/schema.prisma`:
```prisma
datasource db {
  provider = "sqlite"
  url      = "file:./dev.db"
}

generator client {
  provider = "prisma-client-js"
}

model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String?
  createdAt DateTime @default(now())
  posts     Post[]
}

model Post {
  id        Int      @id @default(autoincrement())
  title     String
  content   String?
  published Boolean  @default(false)
  authorId  Int
  author    User     @relation(fields: [authorId], references: [id])
  createdAt DateTime @default(now())
}
```

```bash
npx prisma migrate dev --name init    # create migration + apply
npx prisma generate                   # regenerate client after schema changes
npx prisma studio                     # visual browser at localhost:5555
```

### Drizzle with SQLite

```bash
npm install drizzle-orm better-sqlite3
npm install --save-dev drizzle-kit @types/better-sqlite3
```

`src/db/schema.ts`:
```typescript
import { sqliteTable, text, integer } from "drizzle-orm/sqlite-core";

export const users = sqliteTable("users", {
  id: integer("id").primaryKey({ autoIncrement: true }),
  email: text("email").notNull().unique(),
  name: text("name"),
  createdAt: integer("created_at", { mode: "timestamp" }).notNull().$defaultFn(() => new Date()),
});

export const posts = sqliteTable("posts", {
  id: integer("id").primaryKey({ autoIncrement: true }),
  title: text("title").notNull(),
  content: text("content"),
  published: integer("published", { mode: "boolean" }).notNull().default(false),
  authorId: integer("author_id").notNull().references(() => users.id),
});
```

`src/db/index.ts`:
```typescript
import Database from "better-sqlite3";
import { drizzle } from "drizzle-orm/better-sqlite3";
import * as schema from "./schema";

const sqlite = new Database("app.db");
export const db = drizzle(sqlite, { schema });
```

`drizzle.config.ts`:
```typescript
import type { Config } from "drizzle-kit";

export default {
  schema: "./src/db/schema.ts",
  out: "./drizzle",
  driver: "better-sqlite",
  dbCredentials: { url: "app.db" },
} satisfies Config;
```

```bash
npx drizzle-kit generate    # generate migration SQL
npx drizzle-kit migrate     # apply migrations
npx drizzle-kit studio      # visual browser
```

---

## Performance Pragmas

Run these pragmas on every new connection for best performance:

```javascript
// better-sqlite3
const db = new Database("app.db");
db.pragma("journal_mode = WAL");      // concurrent reads while writing
db.pragma("synchronous = NORMAL");    // safe + fast (vs FULL)
db.pragma("foreign_keys = ON");       // enforce FK constraints
db.pragma("cache_size = -32000");     // 32MB page cache
db.pragma("temp_store = MEMORY");     // temp tables in RAM
```

```python
# Python
conn = sqlite3.connect("app.db")
conn.execute("PRAGMA journal_mode=WAL")
conn.execute("PRAGMA synchronous=NORMAL")
conn.execute("PRAGMA foreign_keys=ON")
conn.execute("PRAGMA cache_size=-32000")
conn.execute("PRAGMA temp_store=MEMORY")
```

**WAL mode is critical** — without it, readers block writers and writers block readers.

---

## Indexing

```sql
-- Index on columns used in WHERE clauses
CREATE INDEX idx_posts_author_id ON posts(author_id);
CREATE INDEX idx_posts_published ON posts(published);

-- Composite index — put equality columns first, range last
CREATE INDEX idx_posts_published_created ON posts(published, created_at);

-- Partial index — only index rows matching a condition
CREATE INDEX idx_posts_unpublished ON posts(created_at)
  WHERE published = 0;

-- Check if index is used (run EXPLAIN QUERY PLAN)
EXPLAIN QUERY PLAN
  SELECT * FROM posts WHERE published = 1 ORDER BY created_at DESC;
-- Look for "USING INDEX" — if you see "SCAN TABLE", add an index
```

---

## Common Queries

```sql
-- Pagination (cursor-based, avoid OFFSET for large tables)
SELECT * FROM posts
WHERE published = 1 AND id > :last_id
ORDER BY id ASC
LIMIT 20;

-- Upsert
INSERT INTO users (email, name)
VALUES (:email, :name)
ON CONFLICT(email) DO UPDATE SET
  name = excluded.name;

-- Full-text search (FTS5)
CREATE VIRTUAL TABLE posts_fts USING fts5(title, content, content='posts', content_rowid='id');

-- Populate FTS table
INSERT INTO posts_fts(rowid, title, content)
  SELECT id, title, content FROM posts;

-- Search
SELECT p.* FROM posts p
JOIN posts_fts ON posts_fts.rowid = p.id
WHERE posts_fts MATCH 'typescript';

-- JSON in SQLite (3.38+)
SELECT json_extract(metadata, '$.tags') FROM items;
SELECT * FROM items WHERE json_extract(metadata, '$.status') = 'active';
```

---

## Migrations (raw SQL, no ORM)

Maintain a `migrations/` folder with numbered files:

```
migrations/
  001_init.sql
  002_add_posts.sql
  003_add_fts.sql
```

Migration runner (Node.js):
```typescript
import Database from "better-sqlite3";
import fs from "fs";
import path from "path";

const db = new Database("app.db");

// Track applied migrations
db.exec(`CREATE TABLE IF NOT EXISTS _migrations (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  name TEXT UNIQUE NOT NULL,
  applied_at INTEGER NOT NULL DEFAULT (unixepoch())
)`);

const applied = new Set(
  db.prepare("SELECT name FROM _migrations").all().map((r: any) => r.name)
);

const migrationDir = path.join(__dirname, "migrations");
const files = fs.readdirSync(migrationDir).sort();

for (const file of files) {
  if (applied.has(file)) continue;
  const sql = fs.readFileSync(path.join(migrationDir, file), "utf8");
  db.transaction(() => {
    db.exec(sql);
    db.prepare("INSERT INTO _migrations (name) VALUES (?)").run(file);
  })();
  console.log(`Applied migration: ${file}`);
}
```

---

## Common Pitfalls

**Concurrent writes from multiple processes**
SQLite allows only one writer at a time. WAL mode helps for readers but still serializes writers. If you need concurrent writes from multiple Node.js processes (e.g., PM2 cluster), switch to PostgreSQL.

**Missing WAL mode**
Default journal mode blocks readers during writes. Always set `PRAGMA journal_mode=WAL`.

**Foreign key enforcement off by default**
SQLite does NOT enforce foreign keys by default. Always run `PRAGMA foreign_keys=ON` on every connection.

**INTEGER PRIMARY KEY vs ROWID**
`INTEGER PRIMARY KEY` is an alias for SQLite's internal rowid — do not use `AUTOINCREMENT` unless you specifically need to prevent rowid reuse. `AUTOINCREMENT` adds overhead.

**Storing dates**
SQLite has no native date type. Use `INTEGER` (Unix epoch) or `TEXT` (ISO 8601). Never use `REAL` for dates unless you understand Julian day numbers.

---

## .env Setup

```
DATABASE_URL="file:./dev.db"       # Prisma
DATABASE_PATH="./app.db"            # raw / Drizzle
```

For Electron/Tauri — use app data directory, not project root:
```typescript
import { app } from "electron";
import path from "path";

const dbPath = path.join(app.getPath("userData"), "app.db");
const db = new Database(dbPath);
```

---

## Progress.txt Update

After SQLite setup is complete, append these fields to the active BUILD phase block in progress.txt:

```
PHASE: BUILD
STATUS: in_progress
database: sqlite
orm: [prisma|drizzle|none]
schema_complete: true
migrations_run: true
timestamp: [ISO timestamp]
```

`vibe-coding-build` reads `schema_complete` and `migrations_run` to confirm the database step is done before proceeding to the next IMPLEMENTATION_PLAN step.
