---
name: vibe-coding-orchestrator
description: |
  Entry point and workflow coordinator for the complete software development lifecycle from idea to shipped product.
  Auto-detects context, routes to appropriate phase skills, validates transitions, and displays status.
  Use when: (1) Starting new projects - user says "build me an app", "new project", "I want to build", "create an app", "start a project",
  (2) Resuming work - user says "continue", "where were we?", "resume", "keep going", "what's next?",
  (3) Has existing materials - user says "I have a PRD", "I have code", "here's my spec", "use this requirements doc",
  (4) Checking status - user says "show status", "where am I?", "what phase?", "progress", "current state",
  (5) Viewing documents - user says "show PRD", "display docs", "view documentation", "show me the [doc name]",
  (6) Auto-activates when opening Claude in a directory with existing code (10+ files + dependency file).
  This orchestrator COORDINATES workflow and DELEGATES to phase-specific skills.
  Does NOT implement phases itself.
---

# Vibe Coding Orchestrator

Route requests to the right skill. Never implement phases directly.

## Skill Map

Phase skills: `vibe-coding-ideate` | `vibe-coding-document` | `vibe-coding-build` | `vibe-coding-debug` | `vibe-coding-ship` | `vibe-coding-reverse-engineer`
State: `vibe-coding-state`
Token efficiency: `vibe-coding-recall` | `vibe-coding-explore`
Design chain: `vibe-coding-ui-ux` | `vibe-coding-doc-design` | `vibe-coding-css-setup` | `vibe-coding-design-templates` | `vibe-coding-design-guard`
Quality: `vibe-coding-code-review` | `vibe-coding-ui-review` | `vibe-coding-impact-analysis` | `vibe-coding-doc-review` | `vibe-coding-security-review` | `vibe-coding-tdd` | `vibe-coding-build-fix` | `vibe-coding-self-improve`
Integration: `vibe-coding-api-connect` | `vibe-coding-cli-runner` | `vibe-coding-mcp-setup`
Database: `vibe-coding-db` | `vibe-coding-db-sqlite` | `vibe-coding-db-bettersqlite` | `vibe-coding-db-postgres` | `vibe-coding-db-duckdb` | `vibe-coding-db-convex`
Deployment: `vibe-coding-local-runner` | `vibe-coding-deploy-vercel` | `vibe-coding-deploy-netlify` | `vibe-coding-deploy-digitalocean`
Framework review: `vibe-coding-review-react` | `vibe-coding-react-native` | `vibe-coding-web-design-guidelines`

## Entry Router

On activation, execute this decision tree top to bottom. Stop at the first match. Do not read phase skill sections before routing is complete.

```
1. Does progress.txt exist?
   YES → Check user message for bug/error keywords first
         ("broken", "bug", "error", "crash", "not working", "failing", "fix", "exception", "TypeError")
         AND current phase is BUILD or SHIP?
         YES → Save return_to_step in progress.txt → READ: ## Bug Intercept
         NO  → READ: ## Resume Workflow

   NO  → Create minimal progress.txt and treat as new project:
             Write: `PHASE: NEW\nSTATUS: ready_to_start\nstarted: [timestamp]`
             Then parse user message as below.

             Does directory have 10+ code files + a dependency file?
         (package.json / requirements.txt / Gemfile / Cargo.toml / go.mod)
         YES → READ: ## Codebase Detected
         NO  → Parse user message intent:
               "build/create/make/new app/I want to/let's build" → Activate vibe-coding-ideate
               "I have a PRD/use this spec/import requirements"  → Activate vibe-coding-document (PRD import mode)
               "fast/quick/skip questions/10 questions"          → Activate vibe-coding-ideate (fast-track)
               "show status/where am I/progress"                → READ: ## Status Display
               "show [doc name]"                                 → READ: ## Show Document
               unclear                                           → READ: ## Welcome Prompt
```

---

## Bug Intercept

```
Bug detected — interrupting [PHASE] to start debug session.
Saving current position: [phase], [step]
```
Write to progress.txt: `return_to_step: [current step]`
Activate `vibe-coding-debug`. Exit.

---

## Resume Workflow

1. Read progress.txt
2. Parse `CURRENT STATE` block for: phase, step, next_action, active_skill
3. Show:
```
RESUMING VIBE CODING PROJECT

Last active: [timestamp]
Current phase: [PHASE]

[If phase = BUILD]  Phase [X], Step [Y] — [feature name]. Status: [in_progress/complete].
[If phase = DOCUMENT]  [N/8] docs approved. Next: [doc name].
[If phase = IDEATE]  [X]/10 topics covered. Last: [topic]. Next: [topic].
[If phase = SHIP]  Deploying to [platform]. Last step: [step].
[If phase = DEBUG]  Session #[N] open. Issue: [description].

Next action: [next_action from progress.txt]

Type "continue" to proceed.
```
4. On "continue": check `current_step` status — announce "Resuming [PHASE] from [step]" — activate `active_skill`. Exit.

---

## Codebase Detected

```
I detected an existing codebase:

Framework: [detected]
Language: [detected]
Files: [count]
Database: [detected or "not detected"]

1. Generate docs from existing code (reverse-engineer)
2. Build a new project here
3. Tell me what you need

What would you like? (1 / 2 / 3 or describe)
```

Handle choice:
- 1 / "reverse" / "analyze" → Activate `vibe-coding-reverse-engineer`
- 2 / "new" / "fresh" → Warn about existing code → confirm → Activate `vibe-coding-ideate`
- 3 / other → Parse intent and route

---

## Welcome Prompt

```
VIBE CODING — Software Development Lifecycle

1. Build a new app (idea → deployed product)
2. Generate docs for existing code
3. Continue an in-progress project
4. Debug an issue
5. Deploy to production

What would you like to do?
```

Handle option 5 / "deploy": Ask "Where are you deploying? (Vercel / Netlify / DigitalOcean / local / other)" → Save as `run_target` in progress.txt → Activate `vibe-coding-ship`.

---

## Status Display

Read progress.txt. Display based on current phase:

**IDEATE:** Topics covered ✓/⊙/✗ for all 10, questions asked, mode, next action.
**DOCUMENT:** Docs approved ✓/⊙/✗ for all 8 with timestamps, next doc.
**BUILD:** Phase/step grid with ✓/⊙/✗, recently completed features, next action.
**DEBUG:** Session #N, issue description, investigation status, next action.
**SHIP:** Checklist ✓/⊙/✗ for pre-flight/env/deploy/smoke/monitor, platform, next action.
**REVERSE-ENGINEER:** Stage (scan/analyze/generate), current area, next action.

Always append:
```
Project: [directory]  |  File: progress.txt
Commands: "continue" | "show [doc]" | "status"
```

---

## Show Document

Parse document name (case-insensitive):
PRD | APP_FLOW/flow | TECH_STACK/tech | DESIGN_SYSTEM/design | BACKEND_STRUCTURE/backend | FRONTEND_GUIDELINES/frontend | IMPLEMENTATION_PLAN/plan | CLAUDE

Check: `./docs/[DOC].md` then `./[DOC].md`

If found: display full contents with location, last modified, size.
If not found: list what docs DO exist in ./docs/ and offer "continue" to generate remaining.

---

## Phase Transition Validation

**IDEATE → DOCUMENT:** All 10 topics complete + summary approved in progress.txt.
**DOCUMENT → BUILD:** All 8 docs approved (`docs_completed: 8, all_approved: true`).
**BUILD → SHIP:** All phases complete + no open debug sessions + user explicitly requests deploy.
**Any → DEBUG:** Always allowed. Save current state first.

If conditions not met: list what's missing and offer to continue current phase.

---

## Error Handling

**Corrupted progress.txt:**
Show raw content → offer: fix automatically / start fresh / manual edit.

**Invalid state:**
Describe violation → offer: auto-fix / force proceed / manual review.

**Missing progress.txt on "continue":**
Offer: start new project / check directory / restore from docs/ folder.

---

## Progress Tracking

Orchestrator reads progress.txt only. Does NOT write directly.
Exception: forced transitions — write CURRENT STATE block with `transition: forced_by_user`.
All other writes are done by phase skills via `vibe-coding-state`.
