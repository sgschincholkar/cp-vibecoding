---
name: vibe-coding-document
description: |
  Generate 8 canonical documentation files from requirements or existing codebase - ONE document at a time with approval workflow.
  Creates: PRD, APP_FLOW, TECH_STACK, DESIGN_SYSTEM, BACKEND_STRUCTURE, FRONTEND_GUIDELINES, IMPLEMENTATION_PLAN, and CLAUDE.md.
  Use when: (1) Generating documentation - user says "generate docs", "create PRD", "make documentation",
  (2) IDEATE phase complete - orchestrator delegates after requirements summary approved,
  (3) Importing PRD - user says "I have a PRD", "use this spec", "import requirements",
  (4) After reverse-engineering - generates remaining 3 business docs.
  Outputs all 8 docs to ./docs/ with sequential approval workflow before transitioning to BUILD.
---

# Vibe Coding DOCUMENT Phase

Generate 8 docs, ONE at a time. Each requires approval before the next starts.

## Entry Router

```
Check progress.txt:
  DOCUMENT in_progress + any docs approved → READ: ## Resume Documentation
  mode: post_reverse_engineer              → READ: ## Post-Reverse-Engineer Mode

Check how this was triggered:
  User provided a PRD file/text           → READ: ## PRD Import Mode
  IDEATE complete (standard)              → READ: ## Standard Mode
```

**Jump directly to the matched mode section. Do not read all modes before starting.**

---

## Standard Mode

Read INTERROGATION_TRANSCRIPT from progress.txt. This is the source for all 8 docs.

If INTERROGATION_TRANSCRIPT is missing or empty → offer:
- (A) Return to IDEATE to gather requirements
- (B) Answer 5 quick gap-filling questions now (core concept, users, features, tech stack, design)
- (C) Skip to IMPLEMENTATION_PLAN and generate from existing code (if codebase present)

Do not block — ask user which option to take.

Check `has_frontend` in progress.txt. If `has_frontend: false` → skip docs #4 (DESIGN_SYSTEM.md) and #6 (FRONTEND_GUIDELINES.md). Mark them as N/A in progress.txt. Set `docs_completed` target to 6 instead of 8.

Generate docs in this order. Use the delegate table below. ONE at a time.

### Document Sequence

| # | Doc | Output Path | Skill |
|---|-----|-------------|-------|
| 1 | PRD.md | `./docs/PRD.md` | `vibe-coding-doc-prd` |
| 2 | APP_FLOW.md | `./docs/APP_FLOW.md` | `vibe-coding-doc-appflow` |
| 3 | TECH_STACK.md | `./docs/TECH_STACK.md` | `vibe-coding-doc-techstack` |
| 4 | DESIGN_SYSTEM.md | `./docs/DESIGN_SYSTEM.md` | `vibe-coding-doc-design` |
| 5 | BACKEND_STRUCTURE.md | `./docs/BACKEND_STRUCTURE.md` | `vibe-coding-doc-backend` |
| 6 | FRONTEND_GUIDELINES.md | `./docs/FRONTEND_GUIDELINES.md` | `vibe-coding-doc-frontend` |
| 7 | IMPLEMENTATION_PLAN.md | `./docs/IMPLEMENTATION_PLAN.md` | `vibe-coding-doc-implplan` |
| 8 | CLAUDE.md | `./CLAUDE.md` (project root) | `vibe-coding-doc-claudemd` |

### Approval Loop

```
FOR each doc (starting from first unapproved):
  1. Announce: "Generating [doc name] ([N]/8)..."
  2. Activate the doc skill
  3. Doc skill shows preview and asks for approval
  4. WAIT for user response
  5. If approved: update progress.txt, move to next doc
  6. If revision: re-activate same doc skill for revision, then re-ask approval
```

After DESIGN_SYSTEM.md is approved, note in progress.txt:
```
css_setup_pending: true  (vibe-coding-css-setup runs at BUILD start)
```

---

## Resume Documentation

Check `DOCS COMPLETED` in progress.txt. Skip any doc with `Approved [timestamp]`.

```
Resuming documentation — [N/8] docs approved. Starting with [next doc].
```

Continue the approval loop from the first unapproved doc.

---

## PRD Import Mode

1. Read the provided PRD content.
2. Score quality (0-100):
   - Problem Statement: 0-20
   - Target Users: 0-20
   - Core Features: 0-20
   - Success Metrics: 0-15
   - Technical Constraints: 0-10
   - Scope Definition: 0-15

3. Act based on score:
   - 86-100: Complete → generate remaining 7 docs immediately
   - 61-85: Solid → ask gap-filling questions for sections < 10pts → generate 7 docs
   - 31-60: Basic → ask gap-filling questions for ALL weak sections → enhance PRD → generate 7 docs
   - 0-30: Stub → "This PRD lacks enough detail. Recommend starting IDEATE instead. Proceed? (yes/no)"

Gap-filling questions (ONE at a time, only for weak sections):
- Problem weak: "In one sentence, what specific problem does this solve and who has it?"
- Users weak: "Describe your primary user: role, goal, current frustration."
- Features weak: "List the 3-5 must-have features. What can't users do without them?"
- Metrics weak: "How will you know this is working? 1-2 measurable metrics."
- Constraints weak: "Any hard constraints? (offline, GDPR, specific hosting)"
- Scope weak: "What is explicitly OUT of scope for v1?"

Full validation logic: `references/prd-import.md`

---

## Post-Reverse-Engineer Mode

5 tech docs already generated. Generate only the 3 business docs.

Mark in progress.txt:
```
PHASE: DOCUMENT
STATUS: in_progress
mode: post_reverse_engineer

DOCS COMPLETED:
✓ TECH_STACK.md - from reverse-engineer
✓ BACKEND_STRUCTURE.md - from reverse-engineer
✓ FRONTEND_GUIDELINES.md - from reverse-engineer
✓ DESIGN_SYSTEM.md - from reverse-engineer (partial)
✓ APP_FLOW.md - from reverse-engineer (partial)

DOCS PENDING:
⊙ PRD.md
✗ IMPLEMENTATION_PLAN.md
✗ CLAUDE.md
```

Generate PRD first (from Q&A context gathered during reverse-engineer gap-filling), then IMPLEMENTATION_PLAN, then CLAUDE.md. Same approval loop applies.

---

## Revision Handler

When user says "revise [doc]" / "update [section] in [doc]" at any point:

1. Identify the doc. Mark `revision_requested` in progress.txt (keep existing approved status).
2. Re-activate the appropriate doc skill for that doc only.
3. After revised doc shown: ask for approval again.
4. If approved: update timestamp, resume sequence from NEXT unapproved doc.

Do NOT restart the full sequence. Do NOT re-generate already-approved docs.

---

## Progress Tracking

After each approval:
```
PHASE: DOCUMENT
STATUS: in_progress
docs_completed: [N]
current_doc: [next doc name]

DOCS COMPLETED:
✓ PRD.md - Approved [timestamp]
[...approved docs...]

DOCS PENDING:
⊙ [current] - Awaiting Approval
✗ [remaining...] - Not Started

NEXT ACTION: [wait for approval OR generate next]
```

---

## Hand-off to BUILD

When all 8 docs approved:
```
PHASE: DOCUMENT
STATUS: complete
completed: [timestamp]
docs_generated: 8
all_approved: true

NEXT ACTION: Start BUILD phase

---
PHASE: BUILD
STATUS: ready_to_start
```

Activate `vibe-coding-build`. Exit.
