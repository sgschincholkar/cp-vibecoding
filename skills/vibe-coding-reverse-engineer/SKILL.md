---
name: vibe-coding-reverse-engineer
description: |
  Analyze existing codebase and generate complete 8-document suite through deep code analysis + business context Q&A.
  Scans code structure, detects framework/stack, analyzes 8 areas (features, architecture, database, auth, APIs, middleware, UX, logic), generates technical docs from code, fills business gaps through targeted questions.
  Use when: (1) Docs from code - user says "generate docs from my code/app/codebase", "document this project/app", "create documentation for this code", "reverse engineer and document",
  (2) Code analysis - user says "analyze my app/code", "reverse engineer this", "understand this codebase", "what does this code do?",
  (3) Auto-detection - orchestrator detects existing codebase (10+ code files + dependency file) and prompts user,
  (4) Explicit trigger - user says "reverse engineer", "docs from code", "analyze codebase", "scan my project".
  Supports Next.js, React, Vue, Django, FastAPI, Rails, Laravel, and more. Auto-detects framework and database.
  Generates 5 tech docs directly from code (TECH_STACK, BACKEND_STRUCTURE, FRONTEND_GUIDELINES, partial DESIGN_SYSTEM, partial APP_FLOW).
  Then asks ~10 business questions to create remaining 3 docs (PRD, enhanced DESIGN_SYSTEM/APP_FLOW, IMPLEMENTATION_PLAN).
  Outputs all 8 canonical docs, then offers: add features (→IDEATE), improve code (→BUILD), or done (→EXIT).
---

# Vibe Coding — Reverse Engineer

Generate docs from an existing codebase via 3 sequential stages.

## Entry Check

```
Count code files in project directory.
Fewer than 5 files?
→ Warn: "This codebase is very small (N files). Results may be limited. Continue? Y/N"

5+ files (or user confirms) → proceed through 3 stages in order:
  Stage 1: activate vibe-coding-re-scan
  Stage 2: activate vibe-coding-re-analyze
  Stage 3: activate vibe-coding-re-generate
```

---

## Stage 1 — vibe-coding-re-scan

Fast detection pass (~60 seconds): framework, language, database, CSS approach, auth, deployment.
Output: scan summary with confidence scores.

---

## Stage 2 — vibe-coding-re-analyze

Deep code reading across 8 areas: features, architecture, database, auth, APIs, middleware, UX, logic.
Shows verified analysis summary to user for confirmation before Stage 3 begins.

---

## Stage 3 — vibe-coding-re-generate

**Wave 1:** Generate 5 technical docs from analysis: TECH_STACK, BACKEND_STRUCTURE, FRONTEND_GUIDELINES, partial DESIGN_SYSTEM, partial APP_FLOW.

**Gap-filling Q&A:** 5-10 business context questions to fill what code can't reveal.

**Wave 2:** Generate 3 business docs: PRD, enhanced DESIGN_SYSTEM/APP_FLOW, IMPLEMENTATION_PLAN, CLAUDE.md.

---

## Progress Tracking

```
PHASE: REVERSE-ENGINEER
STATUS: [quick_scan | deep_analysis | tech_docs | gap_filling | complete]

PROGRESS:
✓/✗ Quick scan
✓/✗ Deep analysis (8/8 areas)
✓/✗ Tech docs (N/5)
✓/✗ Gap-filling Q&A
✓/✗ All 8 docs complete
```

---

## Completion Options

```
REVERSE-ENGINEERING COMPLETE — 8 docs generated.

Next steps:
  1. Add new features → IDEATE phase
  2. Improve existing code → BUILD phase
  3. Just keep the docs → done

Type 1, 2, or 3:
```

Write to progress.txt based on choice:
- Choice 1: `PHASE: IDEATE | mode: add_features`
- Choice 2: `PHASE: BUILD | mode: enhancement | existing_codebase: true`
- Choice 3: `STATUS: complete | user_choice: docs_only`
