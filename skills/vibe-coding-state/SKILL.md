---
name: vibe-coding-state
description: |
  Progress state management for the vibe-coding workflow. Reads, writes, and validates
  progress.txt — the single source of truth for where a project is in the pipeline.
  Handles phase transitions, resume logic, and state validation.
  Use when: (1) vibe-coding-orchestrator needs to read or update workflow state,
  (2) User says "where are we?", "what's the current status?", "resume the project",
  (3) A phase completes and needs to hand off to the next,
  (4) progress.txt is missing, corrupted, or ambiguous.
  Never modifies code files — only reads/writes progress.txt.
---

# Vibe Coding — State Manager

Single source of truth for project position in the pipeline.

## Operation Router

Match the requested operation and jump directly to that section. Do not read other operation sections.

```
READ STATE    → READ: ## Read State
WRITE STATE   → READ: ## Write State
VALIDATE      → READ: ## Validate State
RESUME        → READ: ## Resume
INITIALIZE    → READ: ## Initialize
RECOVER       → READ: ## Recover
```

---

## progress.txt Format

```
PROJECT: [project name]
CREATED: [ISO timestamp]
LAST_UPDATED: [ISO timestamp]

---
CURRENT STATE:
phase: [IDEATE|DOCUMENT|BUILD|DEBUG|SHIP|REVERSE-ENGINEER]
step: [current substep]
next_action: [what to do next]
active_skill: vibe-coding-[phase]

---
PHASE: IDEATE
STATUS: [not_started | in_progress | complete]
mode: [full | fast_track | add_features]
topics_complete: [0-10]
run_target: [local | vercel | netlify | digitalocean | desktop | other_cloud | unknown]

---
PHASE: DOCUMENT
STATUS: [not_started | in_progress | complete]
docs_completed: [N]
current_doc: [doc name]

---
PHASE: BUILD
STATUS: [not_started | in_progress | complete]
css_setup_complete: [true | false]
templates_scaffolded: [true | false]
current_phase: [Phase N]
current_step: [Step N.N]

---
PHASE: DEBUG
STATUS: [not_started | in_progress | complete]
session_count: [N]
return_to_step: [BUILD step to resume]

---
PHASE: SHIP
STATUS: [not_started | in_progress | complete]
environment: [production | staging]

---
UI_UX_SELECTIONS:
  [design selections from vibe-coding-ui-ux]

---
INTERROGATION_TRANSCRIPT:
  [full Q&A from IDEATE phase]
```

**Rule:** File is append-only. Never delete previous blocks. Last occurrence of each phase block is current.

---

## Read State

Returns:
```
Project: [name]  |  Last updated: [timestamp]

Active: [phase] — [status] — [current step]

History:
  IDEATE:    [not_started | complete — timestamp]
  DOCUMENT:  [not_started | N/8 docs | complete]
  BUILD:     [not_started | Phase N Step N.N | complete]
  DEBUG:     [N sessions | none]
  SHIP:      [not_started | complete]

Next action: [next_action from progress.txt]
```

---

## Write State

Rules:
1. Never overwrite — always append below the last `---`
2. Update LAST_UPDATED in the header
3. Only write to progress.txt, never to doc files

Example — marking DOCUMENT complete and signaling BUILD ready:
```
LAST_UPDATED: [timestamp]

---
PHASE: DOCUMENT
STATUS: complete
completed: [timestamp]
docs_completed: 8
all_approved: true

---
PHASE: BUILD
STATUS: ready_to_start
source: document_complete
```

---

## Validate State

**Valid transitions:**
```
not_started → in_progress → complete (forward only)
BUILD in_progress → DEBUG allowed at any time
DEBUG complete → BUILD resumes
```

**Invalid — output error and stop:**
- BUILD started without DOCUMENT complete
- DEBUG has no `return_to_step`
- SHIP started without BUILD complete
- Multiple phases `in_progress` simultaneously
- Unknown phase value (not in IDEATE|DOCUMENT|BUILD|DEBUG|SHIP|REVERSE-ENGINEER): output `STATE ERROR: Unknown phase '[value]'`. Valid phases: IDEATE | DOCUMENT | BUILD | DEBUG | SHIP | REVERSE-ENGINEER. Options: 1. Fix manually 2. Start fresh

Error format:
```
STATE ERROR: [description]

Found: [what progress.txt says]
Expected: [what should be true]

Options:
1. Fix automatically — set [phase] to [correct state]
2. Force proceed (not recommended)
3. Show progress.txt — review manually
```

---

## Resume

1. Check if progress.txt exists.
   - NOT found → check if ./docs/ has generated files
     - Docs found → infer DOCUMENT complete, create minimal progress.txt, report recovery
     - No docs → output:
       ```
       NO PROJECT FOUND

       No progress.txt and no ./docs/ folder here.
       Options: "new project" | check directory
       ```
   - Do NOT call Initialize — that is only for new projects.

2. Read progress.txt → find most recent `in_progress` phase → find most recent incomplete step.

3. Report:
```
RESUMING: [project name]
Last active: [timestamp]
Phase: [phase]  |  Resuming at: [step]

Activating: vibe-coding-[phase skill]
```

---

## Initialize

When progress.txt does not exist (new project only):
```
PROJECT: [infer from directory name or ask user]
CREATED: [current timestamp]
LAST_UPDATED: [current timestamp]

---
PHASE: IDEATE
STATUS: not_started
```

---

## Recover

When progress.txt is corrupted or ambiguous:

1. Show what is readable and what is unclear
2. Ask: "What phase is the project in? What was the last completed step?"
3. Rebuild from user's answer

```
STATE RECOVERY:

progress.txt appears [corrupted | incomplete | inconsistent].

Readable: [list]
Unclear: [list]

Tell me:
1. Current phase?
2. Last completed step?

I'll rebuild progress.txt from your answer.
```

---

## Phase Completion Tokens

| Token | Written by | Meaning |
|-------|-----------|---------|
| `css_setup_complete: true` | vibe-coding-css-setup | CSS foundation ready |
| `templates_scaffolded: true` | vibe-coding-design-templates | Layout templates placed |
| `UI_UX_SELECTIONS:` block | vibe-coding-ui-ux | Design choices captured |
| `docs_completed: 8` | vibe-coding-document | All 8 docs approved |
| `all_tests_passing: true` | vibe-coding-build | Build verified |
| `scan_complete` | vibe-coding-re-scan | Scan done |
| `analysis_complete` | vibe-coding-re-analyze | Analysis done |
