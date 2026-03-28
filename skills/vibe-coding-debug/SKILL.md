---
name: vibe-coding-debug
description: |
  Systematic bug investigation and surgical fixing with investigation logging and lessons learned.
  Creates debug sessions, reproduces bugs, analyzes root cause, applies minimal fixes, verifies resolution, then resumes BUILD.
  Use when: (1) Bug reports - user says "it's broken", "fix this bug", "error", "not working",
  (2) Test failures - user says "tests failing", "CI is red",
  (3) Debug requests - user says "help me debug", "investigate this", "find the bug",
  (4) BUILD phase encounters blocking error - orchestrator temporarily interrupts BUILD.
  Returns to BUILD phase after verification complete.
---

# Vibe Coding DEBUG Phase

Systematic bug investigation. Minimal fixes. Always return to BUILD.

## Session Setup

1. Assign session ID (increment from last in progress.txt)
2. Record issue description
3. Save current BUILD state as `return_to_step`
4. Run `vibe-coding-recall` — search memory for this bug or similar symptoms before investigating

Write to progress.txt:
```
PHASE: DEBUG
STATUS: in_progress
session_count: [N]
current_session: [N]
issue: [description]
return_to_step: [BUILD Phase X, Step Y]
started: [timestamp]
```

---

## Investigation Router

Classify the bug immediately:

```
Vague description (no error message, no steps to reproduce)?
  → READ: ## Vague Bug Handler

Build/compile/type error?
  → READ: ## Build Error Route

Visual/UI bug?
  → READ: ## UI Bug Route

Runtime/logic bug (app runs but behaves wrong)?
  → READ: ## Runtime Bug Investigation

Bug in shared code (ui/ hooks/ utils/ layouts/)?
  → READ: ## Shared Code Bug (run impact analysis first)
```

**Jump directly to the matched section. Do not read all bug handler sections before starting.**

---

## Vague Bug Handler

Do not start investigating. Ask first:
```
To investigate effectively, I need more detail:
1. What exactly happened? (expected vs actual behavior)
2. Was there an error message? (paste it here)
3. What steps lead to this?
```
Wait for answer. Then re-classify and continue.

---

## Build Error Route

Pass full error output to `vibe-coding-build-fix`. Wait for fix.
After fix applied: verify build passes. Return to BUILD.

---

## UI Bug Route

Ask user to share a screenshot.
Run `vibe-coding-ui-review` with the screenshot.
Apply the reported design violations as fixes.
Verify visually. Return to BUILD.

---

## Runtime Bug Investigation

Use `vibe-coding-explore` (smart_search / smart_outline / smart_unfold) to navigate to the bug.
Never read whole files during debug.

```
INVESTIGATION LOG:
[timestamp] - Reproduced: [how]
[timestamp] - Found in: [file:line]
[timestamp] - Root cause: [explanation]

FIX:
[minimal change — what was changed and why]

VERIFICATION:
✓ Bug resolved
✓ Tests passing
✓ No regressions
```

---

## Shared Code Bug

Before touching anything:
→ Run `vibe-coding-impact-analysis` to map blast radius.
Review the impact report. Proceed with the safest change path.
Then follow Runtime Bug Investigation steps.

---

## Closure

Write to progress.txt:
```
DEBUG SESSION #[N] - COMPLETE
completed: [timestamp]
bug_fixed: true
lesson_learned: [what to avoid in future]

---
PHASE: BUILD
STATUS: in_progress
Resume from: [return_to_step value]
```

Run `vibe-coding-code-review` on the fix to verify no new issues introduced.
Activate `vibe-coding-build`. Exit.
