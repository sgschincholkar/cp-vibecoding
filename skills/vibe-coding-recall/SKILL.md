---
name: vibe-coding-recall
description: |
  Cross-session memory recall using claude-mem's mem-search tools.
  Searches persistent memory for prior decisions, bug fixes, patterns, and gotchas
  before re-reading the codebase or asking the user to re-explain context.
  Use when: (1) Starting a new session on an existing project — recall what was last worked on,
  (2) About to debug an issue — check if it was encountered before,
  (3) About to implement something — check how similar things were done before,
  (4) User says "remember when we...", "how did we fix...", "what did we decide about...",
  (5) progress.txt is missing or incomplete — recall from memory instead.
  Requires claude-mem plugin to be installed. Falls back to reading progress.txt if not available.
---

# Vibe Coding — Cross-Session Memory Recall

Check memory before re-reading the codebase or asking the user to re-explain context.

## Entry Router

```
Session start on existing project?
→ Run Step 1 (session start recall) BEFORE reading any files.

Debugging an issue?
→ READ: ## Targeted Recall (debug path)

Implementing a feature?
→ READ: ## Targeted Recall (feature path)

claude-mem not installed?
→ READ: ## Fallback
```

**Jump directly to the matched path. Do not read all recall sections before starting.**

---

## Step 1 — Session Start Recall

```
search(query="recent work", project="[project name]", limit=10, sort="recent")
```

Read results — you now know: current phase, last built/changed, key decisions, gotchas. Resume without reading progress.txt.

---

## Targeted Recall

**Before debugging:**
```
search(query="[error message or symptom]", project="[project]", type="bug-fix")
search(query="[component/file name] bug", project="[project]")
```

**Before implementing:**
```
search(query="[feature type] implementation", project="[project]")
search(query="[component pattern] how we built", project="[project]")
```

**Before modifying shared code:**
```
search(query="[function/component name] decisions", project="[project]", type="decision")
search(query="why [architecture decision]", project="[project]")
```

**Timeline drill-down** (if search result looks relevant but needs more context):
```
timeline(anchor_id="[ID from search result]", depth=3)
```

**Fetch full details** (only after timeline confirms relevance):
```
get_observations(ids=["ID1", "ID2"])
```
Don't fetch everything — filter first (search → timeline → fetch). Full observations cost 500-1000 tokens each.

---

## Recall Report

```
MEMORY RECALL — [project name]

Last session: [timestamp]
Last worked on: [what was built/fixed]
Recent decisions: [key decisions]
Known gotchas: [gotcha observations]

RELEVANT TO CURRENT TASK: [matching observations]
RESUMING FROM: [where to pick up]
```

---

## Fallback (claude-mem not installed)

1. Read `progress.txt` for phase and last step
2. Read `./docs/IMPLEMENTATION_PLAN.md` for current phase context
3. Check recent git log: `git log --oneline -10`
4. Ask user: "What were we last working on?"

If no project name provided and running standalone → ask: "Which project? (leave blank to use current directory)" before running search queries.

---

## What claude-mem Stores Automatically

| Type | Example |
|------|---------|
| `feature` | "Implemented product listing with pagination" |
| `bug-fix` | "Fixed cart state — was useState instead of Zustand" |
| `decision` | "CSS variables over Tailwind because dark mode toggle" |
| `gotcha` | "next/font variables must be on `<html>`, not `<body>`" |
| `pattern` | "All API errors return `{ error: string, code: string }`" |

Token savings: session start recall saves ~3,500 tokens. Prior bug fix recall saves ~14,000 tokens vs re-investigating from scratch.
