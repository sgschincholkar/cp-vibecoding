---
name: vibe-coding-self-improve
description: |
  Post-workflow skill improvement system inspired by Hermes Agent self-reflection.
  After any workflow phase completes, reflects on what went wrong or felt clunky,
  proposes a targeted surgical diff to the relevant SKILL.md, gets user approval,
  writes the update, and logs it in autoresearch/autoresearch-results.md.
  Use when: (1) User says "improve this skill", "this skill was wrong", "update the skill",
  "the skill missed X", "add this to the skill", "the prompt was off",
  (2) A workflow phase completes and Claude notices a gap in the skill instructions,
  (3) User says "self-improve", "run self-improve", "let's improve the skills",
  (4) Called automatically after a workflow produces an unexpected result.
  Do NOT trigger for: general code fixes, project changes, or non-skill improvements.
  Outputs a proposed diff, waits for approval, then writes the change and logs it.
---

# Vibe Coding — Self-Improve

Targeted surgery on skill files. One problem, one fix, one diff.

## Entry Router

```
User-triggered? → READ: ## Reflection Protocol

Post-workflow automatic? → Run internal check:
  "Did any skill instruction lead to wrong output?"
  "Was there a gap the skill should have covered?"
  "Did I improvise something not in the skill?"
  YES to any → READ: ## Reflection Protocol
  NO → skip

User says "run autoresearch on all skills"? → READ: ## Batch Mode
```

**Jump directly to the matched section. If NO gap detected, stop here — do not read further.**

---

## Reflection Protocol

**Step 1 — Identify the gap:**
- Which skill? What was actual behavior? What should have happened?
- Is this: missing case / wrong instruction / outdated pattern / trigger conflict?

**Step 2 — Read `skills/[skill-name]/SKILL.md`** before proposing.
If file not found → output: "Skill file not found at skills/[name]/SKILL.md. Verify the skill exists before proposing changes." and stop.

**Step 3 — Propose surgical diff:**

```
SELF-IMPROVE PROPOSAL
=====================
Skill: vibe-coding-[name]
Problem: [1 sentence]
Root cause: [missing case | wrong instruction | outdated pattern | trigger conflict]
Risk: LOW (adds new case) | MEDIUM (changes existing behavior) | HIGH (changes triggers/handoffs)

--- REMOVE ---
[exact text to remove, or "nothing" if addition only]

+++ ADD +++
[exact text to add]

Location: [section name or "after line containing: [text]"]
Impact on other skills: [none | affects: skill-a, skill-b]

Approve this change? (yes / no / revise)
```

**Step 4 — Wait for user approval.** Do NOT write until user says "approve", "yes", "apply it", or "looks good". Rejected → log as rejected and stop.

**Step 5 — Apply:** Re-read SKILL.md (ensure no concurrent edits) → Edit tool to apply diff → verify change → log.

**Step 6 — Log to `autoresearch/autoresearch-results.md`:**
```markdown
## [Skill Name] — [Date]
**Problem:** [1 sentence]  **Root cause:** [type]
**Change type:** Addition | Modification | Removal  **Risk:** LOW|MEDIUM|HIGH
**Change summary:** [2-3 sentences what changed and why]
**Before:** [key phrase]  **After:** [key phrase]
**Tested:** [yes — user confirmed | no — pending]
---
```

---

## Autoresearch Criteria (5-point check)

Run against any skill on demand:

1. **Trigger conflicts** — do trigger phrases overlap with other skills?
2. **Handoff integrity** — predecessors/successors named correctly and exist in `skills/`?
3. **progress.txt alignment** — block format matches what `vibe-coding-state` expects?
4. **Orchestrator registration** — listed in orchestrator's skill registry with correct trigger mapping?
5. **File/path accuracy** — referenced files/paths exist? GitHub URLs valid?

Score: 5/5=100% (no action), 4/5=97% (one fix), <3/5=needs full pass.

---

## Batch Mode

```
1. Read all SKILL.md files in skills/ directory
2. Run 5-point check on each
3. Present summary table:

AUTORESEARCH SWEEP
==================
| Skill | Score | Top Issue |
|-------|-------|-----------|
| vibe-coding-[name] | 97% | [issue] |

Propose fixes for all below 100%? (yes/no/select)

4. Fix in order lowest→highest score. Log each to autoresearch-results.md.
```

---

## Scope Limits

**In scope:** trigger phrase refinements, adding missing cases, fixing outdated commands, correcting handoff skill names, updating progress.txt block format.

**Out of scope (surface as recommendation, never auto-apply):** changing workflow order, removing existing functionality, changing skill's fundamental purpose, merging or splitting skills.

---

## Progress Tracking

```
PHASE: SELF_IMPROVE
STATUS: [proposed | approved | applied | rejected]
skill: [name]  problem: [brief]  risk: LOW|MEDIUM|HIGH
```
