---
name: vibe-coding-db-bettersqlite
description: |
  BetterSQLite3 patterns for Node.js — synchronous SQLite driver with superior performance and simplicity.
  Use when: (1) vibe-coding-db routes here after detecting better-sqlite3 in package.json,
  (2) User says "BetterSQLite3", "better-sqlite3", "synchronous SQLite",
  (3) Building Electron or Tauri desktop app in Node.js/TypeScript,
  (4) Building a CLI tool that needs embedded local storage.
  Covers: connection setup, CRUD patterns, transactions, prepared statements, WAL pragmas,
  Drizzle ORM integration, Electron integration, and native module rebuild.
  Distinct from vibe-coding-db-sqlite: this skill is Node.js-only, synchronous, and optimized
  for performance-critical local apps. Use vibe-coding-db-sqlite for Python or raw SQL focus.
---

# Vibe Coding — BetterSQLite3

Synchronous SQLite for Node.js. Faster and simpler than the async `sqlite3` package.

## Entry Router

```
Electron/Tauri project?
→ READ: ## Setup (includes native module rebuild steps)

Using Drizzle ORM?
→ READ: ## Drizzle Integration

Raw SQL patterns needed?
→ READ: ## CRUD Patterns + ## Transactions

Performance/WAL setup?
→ READ: ## Performance Pragmas
```

## Why BetterSQLite3 over sqlite3

| Feature | better-sqlite3 | sqlite3 |
|---------|---------------|---------|
| API style | Synchronous | Callback/Promise |
| Performance | 2-3x faster | Baseline |
| Transactions | Native, synchronous | Manual, async |
| Prepared statements | First-class, reusable | Verbose |
| Electron support | With rebuild step | With rebuild step |

---

## Setup

```bash
npm install better-sqlite3
npm install --save-dev @types/better-sqlite3
```

**Electron / Tauri — rebuild for Electron's Node version:**
```bash
npm install --save-dev electron-rebuild
npx electron-rebuild -f -w better-sqlite3

# Or with electron-builder (add to package.json):
"build": {
  "extraMetadata": {},
  "npmRebuild": true
}
```

---

## Connection and Pragmas

```typescript
import Database from "better-sqlite3";

// Create or open database
const db = new Database("app.db");

// Essential pragmas — run once on connection
db.pragma("journal_mode = WAL");       // concurrent reads during writes
db.pragma("synchronous = NORMAL");     // safe + fast
db.pragma("foreign_keys = ON");        // enforce FK constraints
db.pragma("cache_size = -32000");      // 32MB cache
db.pragma("temp_store = MEMORY");      // temp ops in RAM
db.pragma("mmap_size = 268435456");    // 256MB memory-mapped I/O

// For Electron — use app data dir, not project root
import { app } from "electron";
import path from "path";
const dbPath = path.join(app.getPath("userData"), "app.db");
const db = new Database(dbPath);
```

---

## Schema and Migrations

```typescript
// Run schema setup on startup
db.exec(`
  CREATE TABLE IF NOT EXISTS users (
    id    INTEGER PRIMARY KEY,
    email TEXT    NOT NULL UNIQUE,
    name  TEXT,
    created_at INTEGER NOT NULL DEFAULT (unixepoch())
  );

  CREATE TABLE IF NOT EXISTS posts (
    id         INTEGER PRIMARY KEY,
    title      TEXT    NOT NULL,
    content    TEXT,
    published  INTEGER NOT NULL DEFAULT 0,
    author_id  INTEGER NOT NULL REFERENCES users(id),
    created_at INTEGER NOT NULL DEFAULT (unixepoch())
  );

  CREATE INDEX IF NOT EXISTS idx_posts_author ON posts(author_id);
  CREATE INDEX IF NOT EXISTS idx_posts_published ON posts(published, created_at);
`);
```

**Versioned migrations:**
```typescript
function migrate(db: Database.Database) {
  db.exec(`CREATE TABLE IF NOT EXISTS _schema_version (version INTEGER NOT NULL)`);

  const row = db.prepare("SELECT version FROM _schema_version").get() as { version: number } | undefined;
  let version = row?.version ?? 0;

  const migrations: (() => void)[] = [
    // v1
    () => db.exec(`ALTER TABLE users ADD COLUMN avatar_url TEXT`),
    // v2
    () => db.exec(`CREATE INDEX idx_users_email ON users(email)`),
  ];

  db.transaction(() => {
    for (let i = version; i < migrations.length; i++) {
      migrations[i]();
      version = i + 1;
    }
    if (row) {
      db.prepare("UPDATE _schema_version SET version = ?").run(version);
    } else {
      db.prepare("INSERT INTO _schema_version (version) VALUES (?)").run(version);
    }
  })();
}
```

---

## CRUD Patterns

```typescript
// Prepare statements once, reuse many times
const insertUser = db.prepare(
  "INSERT INTO users (email, name) VALUES (@email, @name) RETURNING *"
);
const getUserByEmail = db.prepare(
  "SELECT * FROM users WHERE email = ?"
);
const listPosts = db.prepare(
  "SELECT * FROM posts WHERE published = 1 ORDER BY created_at DESC LIMIT ? OFFSET ?"
);
const updatePost = db.prepare(
  "UPDATE posts SET title = @title, content = @content WHERE id = @id"
);
const deletePost = db.prepare("DELETE FROM posts WHERE id = ?");

// Usage
const user = insertUser.get({ email: "user@example.com", name: "Alice" });
const found = getUserByEmail.get("user@example.com");
const posts = listPosts.all(20, 0);       // .all() returns array
updatePost.run({ title: "New Title", content: "...", id: 1 });
deletePost.run(1);
```

---

## Transactions

Transactions are synchronous and fast — use them for any multi-step operation:

```typescript
// Simple transaction
const createUserWithProfile = db.transaction((email: string, name: string) => {
  const user = db.prepare(
    "INSERT INTO users (email, name) VALUES (?, ?) RETURNING *"
  ).get(email, name) as User;

  db.prepare(
    "INSERT INTO profiles (user_id, bio) VALUES (?, '')"
  ).run(user.id);

  return user;
});

const newUser = createUserWithProfile("user@example.com", "Alice");

// Nested transactions (uses savepoints)
const outer = db.transaction(() => {
  // ...
  inner();  // inner transaction uses SAVEPOINT
});
const inner = db.transaction(() => { /* ... */ });
```

---

## Drizzle ORM with BetterSQLite3

```typescript
// src/db/index.ts
import Database from "better-sqlite3";
import { drizzle } from "drizzle-orm/better-sqlite3";
import * as schema from "./schema";

const sqlite = new Database("app.db");
sqlite.pragma("journal_mode = WAL");
sqlite.pragma("foreign_keys = ON");

export const db = drizzle(sqlite, { schema });

// Query with Drizzle
import { eq, desc, and } from "drizzle-orm";

// Select
const users = await db.select().from(schema.users);

// Insert
const [user] = await db.insert(schema.users)
  .values({ email: "user@example.com", name: "Alice" })
  .returning();

// Update
await db.update(schema.posts)
  .set({ published: true })
  .where(eq(schema.posts.id, 1));

// Delete
await db.delete(schema.posts).where(eq(schema.posts.id, 1));

// Join
const postsWithAuthors = await db
  .select({ post: schema.posts, author: schema.users })
  .from(schema.posts)
  .leftJoin(schema.users, eq(schema.posts.authorId, schema.users.id))
  .where(and(eq(schema.posts.published, true)))
  .orderBy(desc(schema.posts.createdAt))
  .limit(20);
```

---

## Performance Tips

**Batch inserts — use transactions:**
```typescript
const insertMany = db.transaction((rows: { email: string; name: string }[]) => {
  const stmt = db.prepare("INSERT INTO users (email, name) VALUES (?, ?)");
  for (const row of rows) stmt.run(row.email, row.name);
});
insertMany(largeArray);  // 100x faster than individual inserts
```

**Read-heavy queries — use `.pluck()` for single column, `.raw()` for arrays:**
```typescript
const emails = db.prepare("SELECT email FROM users").pluck().all();
// Returns: ["a@x.com", "b@x.com"] instead of [{ email: "a@x.com" }]

const rows = db.prepare("SELECT id, email FROM users").raw().all();
// Returns: [[1, "a@x.com"], [2, "b@x.com"]] — faster, less memory
```

**WAL checkpoint** (for long-running apps):
```typescript
// Periodically checkpoint WAL to main DB file
setInterval(() => {
  db.pragma("wal_checkpoint(TRUNCATE)");
}, 60_000);
```

---

## TypeScript Types

```typescript
import Database from "better-sqlite3";

interface User {
  id: number;
  email: string;
  name: string | null;
  created_at: number;
}

interface Post {
  id: number;
  title: string;
  content: string | null;
  published: number;   // 0 or 1 — SQLite has no boolean
  author_id: number;
  created_at: number;
}

// Type your prepared statements
const getUser = db.prepare<[number], User>("SELECT * FROM users WHERE id = ?");
const user = getUser.get(1);  // typed as User | undefined
```

---

## Common Pitfalls

**Native module in monorepo / pnpm**
If you get "Cannot find module 'better-sqlite3'" in Electron, run `electron-rebuild` — the native `.node` file must be compiled for Electron's exact Node version.

**Boolean columns**
SQLite stores booleans as integers. `published = true` in JS needs to be `1` in SQLite. Drizzle handles this automatically with `{ mode: "boolean" }`. Raw SQL: use `0`/`1` literals.

**Dates**
Store as `INTEGER` (Unix epoch via `unixepoch()`) or `TEXT` (ISO 8601). Avoid `REAL`. Drizzle `{ mode: "timestamp" }` handles the conversion automatically.

**WAL and multiple processes**
WAL mode is per-connection. If multiple Node.js processes open the same `.db` file, each must set WAL separately. Writers still serialize — only one write at a time.

---

## Progress.txt Update

After BetterSQLite3 setup is complete, append these fields to the active BUILD phase block in progress.txt:

```
PHASE: BUILD
STATUS: in_progress
database: bettersqlite
orm: [drizzle|none]
schema_complete: true
migrations_run: true
timestamp: [ISO timestamp]
```

`vibe-coding-build` reads `schema_complete` and `migrations_run` to confirm the database step is done before proceeding to the next IMPLEMENTATION_PLAN step.
