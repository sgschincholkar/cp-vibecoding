---
name: vibe-coding-db-convex
description: |
  Convex database and backend patterns — reactive queries, mutations, actions, schema, file storage,
  real-time subscriptions, cron jobs, HTTP actions, migrations, and AI agent integration.
  Sourced from official Convex agent-skills and convexskills repos.
  Use when: (1) vibe-coding-db routes here after detecting convex in package.json,
  (2) User says "Convex", "real-time database", "reactive queries", "convex mutations",
  (3) Building a collaborative or real-time app (chat, multiplayer, live dashboard),
  (4) TypeScript-native backend with no separate API server wanted,
  (5) Building AI agents that need persistent state and tool execution.
  Covers: setup, schema, queries, mutations, actions, HTTP actions, file storage,
  cron jobs, real-time subscriptions, migrations, security, and AI agent patterns.
  Convex runs in the cloud — not a local file-based database. Requires a Convex account (free tier available).
---

# Vibe Coding — Convex

Reactive serverless backend for TypeScript. Real-time queries, zero infra.

## Entry Router

```
New project setup?
→ READ: ## Setup → ## Schema → ## Queries → ## Mutations

Adding real-time features to existing project?
→ READ: ## Setup (install + provider), then jump to relevant section

Need external API calls?
→ READ: ## Actions

Need webhooks (Stripe, etc.)?
→ READ: ## HTTP Actions

Need scheduled jobs?
→ READ: ## Cron Jobs

Need file uploads?
→ READ: ## File Storage

Evolving existing schema?
→ READ: ## Migrations

Building AI agents?
→ READ: ## AI Agent Integration
```

**Jump directly to the matched section. Do not read all sections before starting.**

---

## When Convex is the Right Choice

- Real-time collaborative apps (chat, multiplayer games, live dashboards)
- TypeScript-first teams who want no separate API server
- Apps where every client auto-updates when data changes
- AI apps needing persistent agent state and tool execution
- Projects that want to skip writing REST endpoints

**Not for:** Python backends, offline-first apps, projects with complex raw SQL requirements, or teams avoiding cloud vendor lock-in.

---

## Setup

```bash
npm install convex
npx convex dev    # starts local dev server + dashboard at localhost:3210

# First time — creates convex/ folder and project
npx convex init   # follow prompts to create account + project
```

**Project structure after init:**
```
convex/
  _generated/    # auto-generated types (never edit)
  schema.ts      # database schema
  users.ts       # functions for users table
  posts.ts       # functions for posts table
  http.ts        # HTTP actions (webhooks)
  crons.ts       # scheduled jobs
```

**.env.local:**
```
CONVEX_DEPLOYMENT=dev:your-project-name
NEXT_PUBLIC_CONVEX_URL=https://your-project.convex.cloud
```

**React setup (`app/layout.tsx` or `main.tsx`):**
```typescript
import { ConvexProvider, ConvexReactClient } from "convex/react";

const convex = new ConvexReactClient(process.env.NEXT_PUBLIC_CONVEX_URL!);

export default function RootLayout({ children }) {
  return <ConvexProvider client={convex}>{children}</ConvexProvider>;
}
```

---

## Schema

```typescript
// convex/schema.ts
import { defineSchema, defineTable } from "convex/server";
import { v } from "convex/values";

export default defineSchema({
  users: defineTable({
    email: v.string(),
    name: v.string(),
    imageUrl: v.optional(v.string()),
    role: v.union(v.literal("admin"), v.literal("member")),
    createdAt: v.number(),  // Unix timestamp
  })
    .index("by_email", ["email"])
    .index("by_role", ["role"]),

  posts: defineTable({
    title: v.string(),
    content: v.optional(v.string()),
    published: v.boolean(),
    authorId: v.id("users"),   // typed foreign key reference
    tags: v.array(v.string()),
    metadata: v.optional(v.object({
      views: v.number(),
      likes: v.number(),
    })),
    createdAt: v.number(),
  })
    .index("by_author", ["authorId"])
    .index("by_published", ["published", "createdAt"])
    .searchIndex("search_posts", {
      searchField: "title",
      filterFields: ["published"],
    }),
});
```

**Validator types:**
```typescript
v.string()        // text
v.number()        // float64
v.int64()         // integer (use for IDs)
v.boolean()
v.null_()
v.id("tableName") // typed document reference
v.array(v.string())
v.object({ key: v.string() })
v.union(v.literal("a"), v.literal("b"))
v.optional(v.string())  // can be undefined
v.any()           // escape hatch
```

---

## Queries (Real-Time, Read-Only)

Queries are reactive — UI auto-updates when underlying data changes.

```typescript
// convex/posts.ts
import { query, internalQuery } from "./_generated/server";
import { v } from "convex/values";

// Public query
export const listPublished = query({
  args: { limit: v.optional(v.number()) },
  handler: async (ctx, args) => {
    return ctx.db
      .query("posts")
      .withIndex("by_published", q => q.eq("published", true))
      .order("desc")
      .take(args.limit ?? 20);
  },
});

// Query with auth
export const myPosts = query({
  args: {},
  handler: async (ctx) => {
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) return [];
    const user = await ctx.db
      .query("users")
      .withIndex("by_email", q => q.eq("email", identity.email!))
      .unique();
    if (!user) return [];
    return ctx.db
      .query("posts")
      .withIndex("by_author", q => q.eq("authorId", user._id))
      .collect();
  },
});

// React usage — auto-subscribes, re-renders on change
import { useQuery } from "convex/react";
import { api } from "@/convex/_generated/api";

function PostList() {
  const posts = useQuery(api.posts.listPublished, { limit: 10 });
  if (posts === undefined) return <Spinner />;  // loading
  return posts.map(post => <PostCard key={post._id} post={post} />);
}
```

---

## Mutations (Transactional Writes)

```typescript
// convex/posts.ts
import { mutation } from "./_generated/server";
import { v } from "convex/values";
import { ConvexError } from "convex/values";

export const createPost = mutation({
  args: {
    title: v.string(),
    content: v.optional(v.string()),
    tags: v.array(v.string()),
  },
  handler: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) throw new ConvexError("Not authenticated");

    const user = await ctx.db
      .query("users")
      .withIndex("by_email", q => q.eq("email", identity.email!))
      .unique();
    if (!user) throw new ConvexError("User not found");

    return ctx.db.insert("posts", {
      title: args.title,
      content: args.content,
      published: false,
      authorId: user._id,
      tags: args.tags,
      createdAt: Date.now(),
    });
  },
});

export const updatePost = mutation({
  args: { id: v.id("posts"), title: v.string(), published: v.boolean() },
  handler: async (ctx, { id, title, published }) => {
    const existing = await ctx.db.get(id);
    if (!existing) throw new ConvexError("Post not found");
    await ctx.db.patch(id, { title, published });
  },
});

export const deletePost = mutation({
  args: { id: v.id("posts") },
  handler: async (ctx, { id }) => {
    await ctx.db.delete(id);
  },
});

// React usage
import { useMutation } from "convex/react";

function CreatePost() {
  const createPost = useMutation(api.posts.createPost);
  return (
    <button onClick={() => createPost({ title: "New Post", tags: [] })}>
      Create
    </button>
  );
}
```

---

## Actions (External API Calls)

Actions can call external APIs and run arbitrary async code, but cannot directly read/write the DB — they call mutations/queries for that.

```typescript
// convex/posts.ts
import { action, internalMutation } from "./_generated/server";
import { v } from "convex/values";
import { internal } from "./_generated/api";

export const generateSummary = action({
  args: { postId: v.id("posts") },
  handler: async (ctx, { postId }) => {
    // Read via query
    const post = await ctx.runQuery(internal.posts.getById, { id: postId });
    if (!post) throw new Error("Post not found");

    // Call external API
    const response = await fetch("https://api.openai.com/v1/chat/completions", {
      method: "POST",
      headers: {
        Authorization: `Bearer ${process.env.OPENAI_API_KEY}`,
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        model: "gpt-4o-mini",
        messages: [{ role: "user", content: `Summarize: ${post.content}` }],
      }),
    });
    const data = await response.json();
    const summary = data.choices[0].message.content;

    // Write result via internal mutation
    await ctx.runMutation(internal.posts.saveSummary, { id: postId, summary });
    return summary;
  },
});
```

---

## HTTP Actions (Webhooks)

```typescript
// convex/http.ts
import { httpRouter } from "convex/server";
import { httpAction } from "./_generated/server";
import { internal } from "./_generated/api";

const http = httpRouter();

// Stripe webhook
http.route({
  path: "/webhooks/stripe",
  method: "POST",
  handler: httpAction(async (ctx, request) => {
    const sig = request.headers.get("stripe-signature");
    const body = await request.text();

    // Verify signature
    const event = verifyStripeSignature(body, sig!, process.env.STRIPE_WEBHOOK_SECRET!);

    if (event.type === "checkout.session.completed") {
      await ctx.runMutation(internal.orders.markPaid, {
        sessionId: event.data.object.id,
      });
    }

    return new Response("OK", { status: 200 });
  }),
});

export default http;
```

---

## Cron Jobs

```typescript
// convex/crons.ts
import { cronJobs } from "convex/server";
import { internal } from "./_generated/api";

const crons = cronJobs();

// Run every hour
crons.interval("cleanup expired sessions", { hours: 1 }, internal.auth.cleanExpired);

// Run daily at 9am UTC
crons.cron("daily digest email", "0 9 * * *", internal.emails.sendDigest);

export default crons;
```

---

## File Storage

```typescript
// convex/files.ts
import { mutation, query } from "./_generated/server";
import { v } from "convex/values";

// Step 1: generate upload URL (called from client)
export const generateUploadUrl = mutation({
  handler: async (ctx) => ctx.storage.generateUploadUrl(),
});

// Step 2: save file reference after upload
export const saveFile = mutation({
  args: { storageId: v.id("_storage"), name: v.string() },
  handler: async (ctx, { storageId, name }) => {
    await ctx.db.insert("files", { storageId, name, uploadedAt: Date.now() });
  },
});

// Step 3: get URL for display
export const getFileUrl = query({
  args: { storageId: v.id("_storage") },
  handler: async (ctx, { storageId }) => ctx.storage.getUrl(storageId),
});
```

```typescript
// React upload component
function FileUpload() {
  const generateUrl = useMutation(api.files.generateUploadUrl);
  const saveFile = useMutation(api.files.saveFile);

  const handleUpload = async (file: File) => {
    const uploadUrl = await generateUrl();
    const { storageId } = await (await fetch(uploadUrl, {
      method: "POST", headers: { "Content-Type": file.type }, body: file,
    })).json();
    await saveFile({ storageId, name: file.name });
  };
}
```

---

## Migrations (Schema Evolution)

Convex uses an optional-field-first migration pattern — no migration files, no downtime.

```typescript
// Adding a new required field — 3-step process:

// Step 1: Add as optional
// schema.ts: bio: v.optional(v.string())

// Step 2: Backfill existing documents (run as mutation)
// convex/migrations.ts
export const backfillBio = mutation({
  handler: async (ctx) => {
    const users = await ctx.db.query("users").collect();
    await Promise.all(
      users.filter(u => u.bio === undefined).map(u =>
        ctx.db.patch(u._id, { bio: "" })
      )
    );
  },
});

// Step 3: Change to required after all docs have value
// schema.ts: bio: v.string()
```

---

## Security

```typescript
// auth helper — use in every query/mutation that needs auth
async function requireAuth(ctx: QueryCtx | MutationCtx) {
  const identity = await ctx.auth.getUserIdentity();
  if (!identity) throw new ConvexError("Authentication required");
  return identity;
}

// Row-level access control
export const updatePost = mutation({
  args: { id: v.id("posts"), title: v.string() },
  handler: async (ctx, { id, title }) => {
    const identity = await requireAuth(ctx);
    const post = await ctx.db.get(id);
    if (!post) throw new ConvexError("Not found");
    if (post.authorEmail !== identity.email)
      throw new ConvexError("Permission denied");
    await ctx.db.patch(id, { title });
  },
});

// Internal functions — not callable from client, only from other server functions
import { internalMutation, internalQuery } from "./_generated/server";
export const adminOnlyOperation = internalMutation({ /* ... */ });
```

---

## AI Agent Integration

Convex is purpose-built for AI agents — persistent state, real-time updates, tool execution.

```typescript
// Install @convex-dev/agent
npm install @convex-dev/agent

// convex/agent.ts
import { Agent } from "@convex-dev/agent";
import { components } from "./_generated/api";
import { openai } from "@ai-sdk/openai";

const agent = new Agent(components.agent, {
  model: openai("gpt-4o"),
  instructions: "You are a helpful assistant.",
  tools: { /* tool definitions */ },
});

export const createThread = mutation(agent.createThread());
export const sendMessage = action(agent.continueThread());
export const listMessages = query(agent.listMessages());
```

---

## Function Type Decision Tree

```
Need to read data? → query()
  Exposed to client? → query()
  Internal only?    → internalQuery()

Need to write data? → mutation()
  Exposed to client? → mutation()
  Internal only?    → internalMutation()

Need external API/fetch? → action()
  Exposed to client? → action()
  Internal only?    → internalAction()

Need HTTP endpoint? → httpAction() in http.ts

Need scheduled job? → cronJobs() in crons.ts
```

---

## Progress.txt Update

After Convex setup is complete, append these fields to the active BUILD phase block in progress.txt:

```
PHASE: BUILD
STATUS: in_progress
database: convex
schema_complete: true
auth_configured: [true|false]
file_storage_configured: [true|false]
timestamp: [ISO timestamp]
```

`vibe-coding-build` reads `schema_complete` to confirm the database step is done before proceeding to the next IMPLEMENTATION_PLAN step.
