---
name: vibe-coding-doc-appflow
description: |
  Generate APP_FLOW.md — every screen, every route, every user journey.
  Maps navigation structure, screen definitions, data requirements per screen, error states,
  empty states, modals, and complete user flows.
  Use when: (1) vibe-coding-document delegates doc #2 (APP_FLOW),
  (2) User says "generate app flow", "map the screens", "document user journeys".
  Reads PRD.md for features and progress.txt for screens and flows.
  Outputs APP_FLOW.md to ./docs/.
---

# Vibe Coding — App Flow Generator

Generate APP_FLOW.md — complete map of every screen and user journey.

## Prerequisites Check

1. Read `./docs/PRD.md` for features and user stories
   - Not found → READ: ## Fallback: No PRD
2. Read `progress.txt` for any screens mentioned during interrogation
3. Check if `has_frontend: false` in progress.txt → READ: ## API-Only Handler
4. Check if SPA (no multi-screen routing) → READ: ## SPA Handler

---

## Generation Steps

1. List every screen in the application
2. Define navigation structure (routes and connections)
3. Write screen definition for each screen
4. Map primary user journeys (happy path + error paths)
5. Document all modals and overlays
6. Save to `./docs/APP_FLOW.md`
7. Show first 20 lines as preview
8. Run Validation Checklist
9. Ask: "APP_FLOW generated. Approve or request revisions?"

## Output Format

```markdown
# [APP NAME] — Application Flow

## Overview
Total Screens: [N]  |  Entry Point: [screen]  |  Auth Required: [Yes - N screens | No]

## Screen Inventory
| Screen | Route | Auth | Purpose |

## Navigation Structure
[Entry Point]
├── [Screen A] /route-a
│   └── [Screen B] /route-b
└── [Screen C] /route-c (auth required)

## Screen Definitions

### [Screen Name]
Route: /[route]  |  Auth: [Yes/No]  |  Layout: [Header + main + Footer | etc.]

Purpose: [what this screen does]

Data Required:
- [item] — from [API endpoint or local state]

States:
- Loading: [skeleton for which area]
- Success: [normal view]
- Empty: [no data — show what]
- Error: [API fail — show what]

Primary Action: [what user does here]
Navigation From: [screens] → Navigation To: [screens, conditions]
Modals: [name] — triggered by [action]

## User Journeys

### [Journey Name]
Entry: [screen]  |  Goal: [what user achieves]

Step 1: [screen] — user [action] → Success: Step 2 | Error: [handling]
Step 2: [screen] — user [action] → Success: complete | Cancel: Step 1

## Modals & Overlays
| Modal | Trigger | Content | Actions |

## Error Handling
| Error | Screen | Display | Recovery |
| Network error | All | Toast: "Connection failed. Retry?" | Retry button |
```

## Generation Rules

1. Every PRD feature has at least one screen
2. Every screen has all 4 states (loading, success, empty, error)
3. Every screen has navigation from/to defined
4. User journeys cover happy path AND main error scenarios
5. No screens left as TBD — make reasonable decisions and note them

## Validation Checklist

```
APP_FLOW VALIDATION:
✓/✗ Every PRD feature maps to at least one screen
✓/✗ All 4 states defined per screen
✓/✗ Navigation complete (no dead-end screens)
✓/✗ All user journeys have error paths
✓/✗ Modal inventory complete
```

---

## Fallback: No PRD

Read `progress.txt` directly for feature list. Note at top of APP_FLOW.md:
"Generated without PRD.md — PRD should be created first for full accuracy."
Proceed with features from transcript.

---

## API-Only Handler

Project has no frontend (CLI, REST API, backend service). Replace screen inventory with endpoint inventory:

Note at top: "API-only project — no screens. Documenting endpoint flows instead of screen flows."

Replace:
- "Screen Inventory" → "Endpoint Inventory" (Method | Path | Auth | Purpose)
- "Navigation Structure" → "Request Flow" (entry → middleware → handler → response)
- "User Journeys" → "API Call Flows" (client call → server processing → response)
- Skip: modals, loading states, empty states

---

## SPA Handler

If the app has no multi-screen routing (single-page or single-view architecture):
Replace Navigation Structure tree with a State/View Transition Map:

```
[Initial State]
├── [View A] — triggered by [action]
│   └── [Sub-view] — triggered by [action]
└── [View B] — triggered by [action]
```

Note at top of navigation section: "Single-page architecture — transitions between views, not routes."
