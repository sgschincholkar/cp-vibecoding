---
name: vibe-coding-doc-prd
description: |
  Generate PRD.md — the Product Requirements Document.
  Covers problem statement, target users, features with priorities, user stories, acceptance criteria,
  success metrics, constraints, and out-of-scope items.
  Use when: (1) vibe-coding-document delegates doc #1 (PRD),
  (2) User says "generate PRD", "write the product requirements", "create the PRD",
  (3) Resuming document phase at PRD step.
  Reads interrogation transcript from progress.txt.
  Outputs PRD.md to ./docs/.
---

# Vibe Coding — PRD Generator

Generate the Product Requirements Document. Business foundation — every feature decision traces here.

## Prerequisites Check

1. Read `progress.txt` — get INTERROGATION_TRANSCRIPT for product requirements
2. Check if `./docs/PRD.md` already exists → if yes, confirm with user before regenerating

If transcript is thin on data → READ: ## Thin Data Handler

---

## Generation Steps

1. Extract problem statement, target users, features (must/should/nice), success metrics, constraints from transcript
2. Write user stories for each must-have feature
3. Define testable acceptance criteria for each user story
4. List explicit out-of-scope items
5. Save to `./docs/PRD.md`
6. Show first 20 lines as preview
7. Run Validation Checklist
8. Ask: "PRD generated. Approve or request revisions?"

## Output Format

```markdown
# [APP NAME] — Product Requirements Document

## Problem Statement
[2-3 sentences: what problem, who has it, why current solutions fail]

## Target Users
### Primary User
- Who: [role/type]  |  Goal: [what they want]  |  Pain Point: [frustration today]

## Features

### Must-Have (MVP)
| # | Feature | User Story | Acceptance Criteria |
|---|---------|------------|---------------------|
| 1 | [name] | As a [user], I want [action] so that [benefit] | - [verb criterion 1] |

### Should-Have (V1.1)
| # | Feature | Notes |

### Nice-to-Have (Future)
| # | Feature | Notes |

## Success Metrics
| Metric | Target | Measurement |

## Technical Constraints
- [Constraint — e.g., must work offline, GDPR required]

## Out of Scope (V1)
- [Feature explicitly not in this version]
```

## Generation Rules

1. Features from transcript only — never invent
2. Acceptance criteria are testable — each starts with an action verb
3. Out-of-scope section is mandatory — at least one item
4. Success metrics are measurable — no vague metrics like "users enjoy the product"
5. Scan transcript for compliance keywords (HIPAA, GDPR, PCI-DSS, SOC2, COPPA, CCPA, ADA/accessibility) → auto-add matching items to Technical Constraints. If found but details thin: add `[COMPLIANCE DETAIL NEEDED]` placeholder and flag in validation checklist.

## Validation Checklist

Output after generating with actual pass/fail:
```
PRD VALIDATION:
✓/✗ Problem statement concrete and specific
✓/✗ Target user specific (not "everyone")
✓/✗ All must-haves have user stories
✓/✗ Acceptance criteria testable (action verbs)
✓/✗ Out-of-scope list exists (min 1 item)
✓/✗ No invented features
✓/✗ Success metrics measurable
```

---

## Thin Data Handler

Load only when transcript is sparse. Match the condition:

**No acceptance criteria for a feature:**
Derive from the feature name: "User login" → "User can authenticate with valid email/password", "System rejects invalid credentials". Do not leave blank.

**No success metrics:**
Insert `[METRICS NEEDED]` placeholder. Mark as failing in validation checklist.

**Target user is "everyone" or non-specific:**
Use the most common persona described in the features as primary user. Add assumption: "Primary user identified from feature context — confirm with stakeholder."
