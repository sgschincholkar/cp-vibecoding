---
name: vibe-coding-db-duckdb
description: |
  DuckDB patterns — analytics, OLAP queries, CSV/Parquet/JSON processing, and embedding in Node.js or Python.
  In-process columnar database — zero server, zero config, runs inside your app like SQLite but for analytics.
  Use when: (1) vibe-coding-db routes here after detecting duckdb in package.json or requirements.txt,
  (2) User says "DuckDB", "analytics", "OLAP", "columnar", "query CSV files", "process Parquet",
  (3) Building a dashboard, data pipeline, or reporting tool,
  (4) Need to run complex aggregations or analytical queries on local data.
  Covers: Node.js and Python setup, SQL patterns, file querying (CSV/Parquet/JSON),
  aggregations, window functions, and combining with SQLite or PostgreSQL.
  NOT for: transactional OLTP apps (use SQLite or PostgreSQL for those).
---

# Vibe Coding — DuckDB

In-process analytics database. Query CSV, Parquet, and JSON files directly. No server needed.

## Entry Router

```
Language detected from TECH_STACK.md or package.json:
  Node.js/TypeScript → READ: ## Setup (Node.js section)
  Python → READ: ## Setup (Python section)
  Both → read Node.js section first, then Python section

DuckDB as primary DB (no other database)?
→ Note: DuckDB is optimized for analytics/OLAP. For transactional app data, also consider SQLite or PostgreSQL as primary store. Document in TECH_STACK.md and progress.txt: primary_db: duckdb (if truly standalone).

DuckDB alongside existing DB (SQLite/PostgreSQL)?
→ Proceed to setup, then see ## Combining DuckDB with Other Databases
```

---

## When DuckDB is the Right Choice

- Analytics dashboards and reporting
- Processing CSV, Parquet, or JSON data files
- Complex aggregations and window functions
- Data science pipelines in Python
- Combining data from multiple sources (CSV + Parquet + in-memory tables)
- Replacing pandas for data transformation

**Not for:** Transactional web apps, concurrent writes from multiple users. Use SQLite or PostgreSQL for OLTP.

---

## Setup

### Node.js

```bash
npm install duckdb
# Or the newer async client:
npm install @duckdb/node-api
```

```typescript
// Sync API (simpler)
import * as duckdb from "duckdb";

const db = new duckdb.Database(":memory:");       // in-memory
// const db = new duckdb.Database("analytics.db"); // persistent file

const conn = db.connect();

conn.all("SELECT 42 AS answer", (err, rows) => {
  console.log(rows);  // [{ answer: 42 }]
});

// Promise wrapper
const query = (sql: string, params?: any[]): Promise<any[]> =>
  new Promise((resolve, reject) =>
    conn.all(sql, ...(params ?? []), (err, rows) =>
      err ? reject(err) : resolve(rows)
    )
  );
```

### Python

```bash
pip install duckdb
```

```python
import duckdb

# In-memory (default)
conn = duckdb.connect()

# Persistent
conn = duckdb.connect("analytics.db")

# Query
result = conn.execute("SELECT 42 AS answer").fetchall()
# Returns as Pandas DataFrame:
df = conn.execute("SELECT * FROM my_table").df()
```

---

## Querying Files Directly

DuckDB's killer feature — query files without loading them into a table first.

```sql
-- CSV file
SELECT * FROM read_csv('data/sales.csv', header=True, auto_detect=True);
SELECT month, SUM(amount) FROM read_csv('data/sales.csv') GROUP BY month;

-- Multiple CSV files (glob)
SELECT * FROM read_csv('data/sales_*.csv');

-- Parquet file
SELECT * FROM read_parquet('data/events.parquet');
SELECT * FROM read_parquet('data/events/*.parquet');  -- folder of partitioned files

-- JSON file
SELECT * FROM read_json('data/users.json');
SELECT * FROM read_json_auto('data/users.json');      -- auto-detect schema

-- Remote files (HTTP)
SELECT * FROM read_parquet('https://example.com/data.parquet');

-- Create a persistent table from a file
CREATE TABLE sales AS SELECT * FROM read_csv('data/sales.csv', auto_detect=True);
```

---

## SQL Patterns

### Aggregations

```sql
-- Group by with rollup
SELECT
  category,
  region,
  SUM(revenue) AS total_revenue,
  AVG(revenue) AS avg_revenue,
  COUNT(*) AS order_count,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY revenue) AS median_revenue
FROM sales
GROUP BY ROLLUP(category, region)
ORDER BY category, region;

-- Date truncation
SELECT
  DATE_TRUNC('month', order_date) AS month,
  SUM(amount) AS monthly_revenue
FROM orders
GROUP BY 1
ORDER BY 1;

-- Histogram
SELECT
  width_bucket(amount, 0, 1000, 10) AS bucket,
  COUNT(*) AS count,
  MIN(amount) AS bucket_min,
  MAX(amount) AS bucket_max
FROM orders
GROUP BY bucket
ORDER BY bucket;
```

### Window Functions

```sql
-- Running total
SELECT
  order_date,
  amount,
  SUM(amount) OVER (ORDER BY order_date) AS running_total
FROM orders;

-- Rank within group
SELECT
  product_id,
  category,
  revenue,
  RANK() OVER (PARTITION BY category ORDER BY revenue DESC) AS rank_in_category
FROM products;

-- Month-over-month change
SELECT
  month,
  revenue,
  LAG(revenue, 1) OVER (ORDER BY month) AS prev_month,
  revenue - LAG(revenue, 1) OVER (ORDER BY month) AS change,
  ROUND(100.0 * (revenue - LAG(revenue, 1) OVER (ORDER BY month)) / LAG(revenue, 1) OVER (ORDER BY month), 2) AS pct_change
FROM monthly_revenue;

-- Moving average
SELECT
  date,
  value,
  AVG(value) OVER (ORDER BY date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS moving_avg_7d
FROM time_series;
```

### Pivoting

```sql
-- PIVOT syntax (DuckDB 0.8+)
PIVOT sales
ON category
USING SUM(amount)
GROUP BY month;

-- Manual pivot with FILTER
SELECT
  month,
  SUM(amount) FILTER (WHERE category = 'Electronics') AS electronics,
  SUM(amount) FILTER (WHERE category = 'Clothing') AS clothing,
  SUM(amount) FILTER (WHERE category = 'Books') AS books
FROM sales
GROUP BY month;
```

---

## Combining DuckDB with Other Databases

```python
# Python — attach SQLite database and query alongside DuckDB
conn = duckdb.connect()
conn.execute("ATTACH 'app.db' AS sqlite_db (TYPE sqlite)")
result = conn.execute("""
  SELECT u.name, COUNT(o.id) AS order_count
  FROM sqlite_db.users u
  JOIN read_parquet('data/orders.parquet') o ON o.user_id = u.id
  GROUP BY u.name
""").df()
```

```sql
-- Attach PostgreSQL (requires httpfs + postgres extension)
INSTALL postgres;
LOAD postgres;
ATTACH 'postgresql://localhost:5432/myapp' AS pg (TYPE postgres);
SELECT * FROM pg.orders LIMIT 10;
```

---

## Exporting Results

```sql
-- Export to CSV
COPY (SELECT * FROM sales WHERE year = 2025) TO 'output/sales_2025.csv' (HEADER, DELIMITER ',');

-- Export to Parquet
COPY sales TO 'output/sales.parquet' (FORMAT parquet);

-- Export to JSON
COPY (SELECT * FROM users) TO 'output/users.json' (FORMAT json);
```

```python
# Export to Pandas
df = conn.execute("SELECT * FROM sales").df()

# Export to Polars (fast)
import polars as pl
df = conn.execute("SELECT * FROM sales").pl()
```

---

## In-Memory Tables

```sql
-- Create table from Python data
conn.execute("CREATE TABLE events AS SELECT * FROM ?", [events_list])

-- Create from Pandas
conn.execute("CREATE TABLE events AS SELECT * FROM df")  # df is a pandas DataFrame in scope

-- From Polars
conn.register("events", polars_df)
conn.execute("SELECT * FROM events")
```

---

## Performance Tips

**Use Parquet for large datasets** — columnar storage means DuckDB only reads columns it needs:
```sql
-- Only reads 'amount' and 'date' columns from file, skips everything else
SELECT SUM(amount) FROM read_parquet('data/orders.parquet') WHERE date > '2025-01-01';
```

**Parallel execution** — DuckDB uses all CPU cores by default:
```python
import duckdb
conn = duckdb.connect()
conn.execute("SET threads = 8")          # explicit thread count
conn.execute("SET memory_limit = '4GB'") # memory cap
```

**Persistent database for repeated queries:**
```python
# First run: loads and processes data, saves to .db file
conn = duckdb.connect("analytics.db")
conn.execute("CREATE TABLE IF NOT EXISTS sales AS SELECT * FROM read_parquet('data/*.parquet')")

# Subsequent runs: query from .db file directly (fast)
result = conn.execute("SELECT month, SUM(revenue) FROM sales GROUP BY month").df()
```

---

## Use with Vibe Coding Workflow

DuckDB is typically used alongside a primary database (SQLite or PostgreSQL), not as a replacement.

**Pattern 1 — Analytics layer over SQLite:**
```
SQLite (app data) → export to CSV/Parquet → DuckDB (analytics queries)
```

**Pattern 2 — Analytics layer over PostgreSQL:**
```
PostgreSQL (app data) → DuckDB ATTACH postgres → analytics queries
```

**In TECH_STACK.md** — document as:
```
Primary DB: PostgreSQL (app data)
Analytics DB: DuckDB (reporting + dashboards, in-process)
```

---

## Progress.txt Update

After DuckDB setup is complete, append these fields to the active BUILD phase block in progress.txt:

```
PHASE: BUILD
STATUS: in_progress
database: duckdb
primary_db: [sqlite|bettersqlite|postgres|none]
schema_complete: true
timestamp: [ISO timestamp]
```

Note: DuckDB is typically a secondary analytics layer alongside a primary transactional database. Document `primary_db` to indicate what the primary store is. `vibe-coding-build` reads `schema_complete` to confirm setup is done.
