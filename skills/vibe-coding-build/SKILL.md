---
name: vibe-coding-build
description: |
  Step-by-step software implementation following IMPLEMENTATION_PLAN.md with protection rules to prevent regressions.
  Implements features sequentially, one phase/step at a time, with continuous progress tracking and testing.
  Use when: (1) Starting implementation - user says "let's build", "start coding", "implement", "begin development",
  (2) All 8 docs approved - orchestrator delegates BUILD phase after DOCUMENT phase complete,
  (3) Adding features - user says "add this feature", "implement [feature name]",
  (4) Continuing implementation - user says "continue building", "next step", "proceed",
  (5) Resuming after debug - orchestrator returns to BUILD after debug session complete.
  Requires IMPLEMENTATION_PLAN.md and all canonical docs.
---

# Vibe Coding BUILD Phase

Implement features step-by-step following IMPLEMENTATION_PLAN.md.

## Prerequisites Check

Before anything else:
- progress.txt exists?                     NO → check for IMPLEMENTATION_PLAN.md:
                                               Found → create minimal progress.txt, assume DOCUMENT complete, proceed
                                               Not found → activate `vibe-coding-document`
- `./docs/IMPLEMENTATION_PLAN.md` exists?  NO → activate `vibe-coding-doc-implplan`
- progress.txt shows DOCUMENT complete?    NO → stop, message "Complete DOCUMENT phase first"
- Returning from DEBUG?                    YES → READ: ## Resume from Debug

---

## Startup Sequence

Run once per project (check progress.txt flags — skip if already done):

1. **vibe-coding-recall** — every session start (if claude-mem installed). Restores context without reading files.
2. **vibe-coding-css-setup** — if `css_setup_complete: true` NOT in progress.txt AND `has_frontend: false` NOT in progress.txt
3. **vibe-coding-design-templates** — if `templates_scaffolded: true` NOT in progress.txt AND `has_frontend: false` NOT in progress.txt

---

## Build Loop

```
READ IMPLEMENTATION_PLAN.md
FIND current phase and step from progress.txt

FOR each phase:
  FOR each step:
    → READ: ## Step Execution
  Write phase complete to progress.txt

ALL phases done → READ: ## Hand-off to SHIP
```

---

## Step Execution

For each step:

1. Announce: "Phase [X], Step [Y]: [feature name]"
2. Read `references/protection-rules.md`
3. If step reads/modifies existing code file > 100 lines → use `vibe-coding-explore` (not Read)
4. If step creates a page or component → run `vibe-coding-design-guard` first
5. Implement the step
6. Run `vibe-coding-code-review` on new/modified files
7. Fix any MUST FIX violations before continuing
8. Test/verify

**If build/compile/type error occurs:**
→ Call `vibe-coding-build-fix` immediately with full error output
→ Do NOT proceed to next step until error is resolved
→ READ: ## Error Recovery

9. Update progress.txt (mark step complete)
10. Move to next step

---

## Quality Hooks

| Hook | When to call |
|------|-------------|
| `vibe-coding-design-guard` | Before generating any page or component |
| `vibe-coding-code-review` | After each step that creates/modifies code |
| `vibe-coding-impact-analysis` | Before modifying shared files (ui/ hooks/ utils/ layouts/) |
| `vibe-coding-ui-review` | When user shares a screenshot |
| `vibe-coding-tdd` | When user says "write tests first", "TDD", or "add tests" |
| `vibe-coding-build-fix` | When build command fails (compile/type/import errors) |
| `vibe-coding-api-connect` | When step involves an external API |
| `vibe-coding-cli-runner` | When step requires CLI tools (migrations, codegen, scaffolding) |
| `vibe-coding-mcp-setup` | When user wants Claude to query DB or external service |

---

## Error Recovery

When `vibe-coding-build-fix` is called:
- Pass full error output
- Wait for fix
- Re-run build command to verify
- If resolved: continue from same step
- If not resolved: log in progress.txt, ask user for guidance

---

## Resume from Debug

Read `return_to_step` from progress.txt:
```
Debug session complete. Resuming BUILD at Phase [X], Step [Y].
```
Continue from that exact step.

---

## Progress Tracking

After each step:
```
PHASE: BUILD
STATUS: in_progress
current_phase: Phase [X]
current_step: Step [Y]
feature: [Feature Name]

PHASES:
✓ Phase 1: [name] - Complete
⊙ Phase 2: [name] - Step [N]/[total]
✗ Phase 3+: Not Started

FILES CHANGED: [list]
NEXT ACTION: [exact next step description]
```

---

## Hand-off to SHIP

When all phases complete:
```
PHASE: BUILD
STATUS: complete
completed: [timestamp]
all_tests_passing: true

NEXT ACTION: Deploy

---
PHASE: SHIP
STATUS: ready_to_start
```

Activate `vibe-coding-ship`. Exit.
