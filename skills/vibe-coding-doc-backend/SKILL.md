---
name: vibe-coding-doc-backend
description: |
  Generate BACKEND_STRUCTURE.md — complete database schema and API endpoint documentation.
  Covers all data models with field types, relationships, and constraints.
  Documents every API route with method, auth requirement, request/response shape, and error codes.
  Use when: (1) vibe-coding-document delegates doc #5 (BACKEND_STRUCTURE),
  (2) User says "generate backend structure", "document the API", "create the data model".
  Reads PRD.md and APP_FLOW.md to derive data models and endpoints.
  Outputs BACKEND_STRUCTURE.md to ./docs/.
---

# Vibe Coding — Backend Structure Generator

Generate BACKEND_STRUCTURE.md. Every data model and API endpoint documented with exact types and contracts.

## Prerequisites Check

1. Read `progress.txt` — check `has_frontend: false` flag
2. Read `./docs/PRD.md` — features determine data models
3. Read `./docs/APP_FLOW.md` — screens determine API endpoints
4. Read `./docs/TECH_STACK.md` — detect ORM and API style:

```
Frontend-only project (no backend in TECH_STACK.md) → READ: ## Frontend-Only Handler

ORM detection:
  Prisma      → READ: ## Schema Format: Prisma
  SQLAlchemy  → READ: ## Schema Format: SQLAlchemy (Python)
  Drizzle     → READ: ## Schema Format: Drizzle
  Mongoose    → READ: ## Schema Format: Mongoose
  TypeORM     → READ: ## Schema Format: TypeORM
  Other/unlisted → READ: ## Schema Format: Generic

API style detection:
  REST    → READ: ## Pagination: REST
  tRPC    → READ: ## Pagination: tRPC
  GraphQL → READ: ## Pagination: GraphQL
```

---

## Generation Steps

1. Derive all data models from PRD features
2. Define fields, types, relationships, constraints
3. Generate schema in ORM format (from matching Schema Format section)
4. Derive all API endpoints from APP_FLOW screens
5. Document each endpoint: method, path, auth, request, response, errors
6. Add Business Logic Rules
7. Add Database Indexes
8. Save to `./docs/BACKEND_STRUCTURE.md`
9. Show first 20 lines as preview
10. Run Validation Checklist
11. Ask: "BACKEND_STRUCTURE generated. Approve or request revisions?"

## Output Format

```markdown
# [APP NAME] — Backend Structure

## Overview
Database: [PostgreSQL | MySQL | SQLite | MongoDB]
ORM: [Prisma | Drizzle | SQLAlchemy | Mongoose | TypeORM]
API Style: [REST | tRPC | GraphQL]

## Data Models

### [Model Name]
Purpose: [what this represents]

| Field | Type | Required | Unique | Default | Notes |
|-------|------|----------|--------|---------|-------|
| id | uuid | Yes | Yes | auto | Primary key |
| createdAt | DateTime | Yes | No | now() | Auto-set |

Relationships:
- [Model] has many [Other] (via [foreignKey])
- [Model] belongs to [Other] (via [foreignKey])

## Database Schema
[Schema in ORM format — from matching Schema Format section]

## API Endpoints

### [Resource]

#### [METHOD] /api/[resource]
Auth required: [Yes (role) | No]
Query params: [if applicable]
Request body: [JSON if applicable]
Response 200: [JSON shape]
Errors: [code: reason]

## Business Logic Rules
- [Rule 1 — e.g., prices stored in cents]
- [Rule 2 — e.g., soft deletes via deletedAt]

## Database Indexes
| Model | Fields | Type | Reason |
```

## Generation Rules

1. Every PRD feature has supporting data model(s)
2. Every APP_FLOW screen has supporting API endpoints
3. All list endpoints are paginated (format from matching Pagination section)
4. Auth requirement explicit for every endpoint — never assumed
5. Response shapes are complete JSON examples — no "...rest of fields"

## Validation Checklist

```
BACKEND_STRUCTURE VALIDATION:
✓/✗ All PRD features have data models
✓/✗ All APP_FLOW screens have endpoints
✓/✗ Schema valid and complete
✓/✗ Every endpoint has auth stated
✓/✗ Response shapes complete
✓/✗ Error codes cover main failures
```

---

## Frontend-Only Handler

Project has no backend (static site, client-only SPA, or Jamstack with external APIs only).

Output a minimal BACKEND_STRUCTURE.md noting:
```markdown
# [APP NAME] — Backend Structure

## Overview
Architecture: Frontend-only
Backend: None (uses [external API / BaaS / no backend])

## External Data Sources
| Service | Purpose | Auth |
|---------|---------|------|
| [API/service name] | [what data] | [API key / OAuth] |

## Notes
No custom backend — no data models or API endpoints to document.
Data contracts governed by external API documentation.
```

---

## Schema Format: Generic

For ORMs not listed above (Sequelize, Peewee, ActiveRecord, GORM, Ecto, etc.):

```
[ModelName]:
  Fields:
    - id: primary key (uuid or serial)
    - [field]: [type], required: [yes/no], unique: [yes/no]
    - created_at: datetime, auto
  Relationships:
    - has_many: [RelatedModel]
    - belongs_to: [ParentModel]
```

Note: "Schema shown in generic format — adapt to [detected ORM] syntax from official documentation."

---

## Schema Format: Prisma

```prisma
model [ModelName] {
  id        String   @id @default(cuid())
  [field]   [Type]   [modifiers]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  // Relations
  [relation]  [RelatedModel][]

  @@map("[table_name]")
}
```

---

## Schema Format: SQLAlchemy (Python)

```python
class [ModelName](Base):
    __tablename__ = "[table_name]"
    id = Column(UUID, primary_key=True, default=uuid4)
    [field] = Column([Type], nullable=False)
    created_at = Column(DateTime, default=datetime.utcnow)
    # Relations
    [relation] = relationship("[RelatedModel]", back_populates="[field]")
```

---

## Schema Format: Drizzle

```typescript
export const [tableName] = pgTable('[table_name]', {
  id: uuid('id').primaryKey().defaultRandom(),
  [field]: [type]('[field]').notNull(),
  createdAt: timestamp('created_at').defaultNow(),
})
```

---

## Schema Format: Mongoose

```typescript
const [ModelName]Schema = new Schema({
  [field]: { type: [Type], required: true },
  createdAt: { type: Date, default: Date.now },
})
```

---

## Schema Format: TypeORM

```typescript
@Entity()
class [ModelName] {
  @PrimaryGeneratedColumn('uuid') id: string
  @Column() [field]: [type]
  @CreateDateColumn() createdAt: Date
  @OneToMany(() => [Related], (r) => r.[field]) [relation]: [Related][]
}
```

---

## Pagination: REST

All list endpoints use `page` and `limit` query params:
```json
{
  "data": [...],
  "pagination": { "page": 1, "limit": 20, "total": 100, "totalPages": 5 }
}
```

---

## Pagination: tRPC

Cursor-based pagination:
```typescript
input({ cursor: z.string().optional(), limit: z.number().default(20) })
// output includes nextCursor
```

---

## Pagination: GraphQL

Connections pattern with `edges`, `node`, `pageInfo`, and `cursor`.
