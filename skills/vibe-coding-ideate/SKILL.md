---
name: vibe-coding-ideate
description: |
  Interactive requirements gathering through adaptive conversation - asks ONE question at a time and builds context progressively.
  Covers 10 topics: core concept, users, features, data, auth, tech stack, business model, edge cases, integrations, and design.
  Use when: (1) Planning new features - user says "let's ideate", "plan features", "gather requirements",
  (2) Vague idea needs refinement - user says "I have an idea for an app", "help me plan",
  (3) Fast-track mode - user says "skip the questions", "I know what I want", "quick mode", "10 questions",
  (4) Adding features after reverse-engineering existing code.
  Outputs complete requirements transcript to progress.txt for handoff to vibe-coding-document.
---

# Vibe Coding IDEATE Phase

Requirements gathering through conversation. ONE question at a time.

## Mode Router

On activation, detect mode then jump directly to that section. Do not read all modes upfront.

```
1. Check progress.txt if it exists:
   IDEATE in_progress + topics_complete > 0  → READ: ## Resume Session
   mode: fast_track                           → READ: ## Fast-Track Mode
   mode: add_features                         → READ: ## Add Features Mode

2. Check user message:
   "fast" / "quick" / "skip" / "10 questions" → READ: ## Fast-Track Mode
   "add features" / "enhance existing"        → READ: ## Add Features Mode
   everything else                            → READ: ## Full Mode
```

---

## Full Mode

Write progress.txt before first message:
```
PHASE: IDEATE
STATUS: in_progress
mode: full
started: [timestamp]
questions_asked: 0
TOPICS: core-concept⊙ users✗ features✗ data✗ auth✗ tech-stack✗ business✗ edge-cases✗ integrations✗ design✗
NEXT ACTION: Ask opening question
```

Show:
```
IDEATE PHASE STARTED — Full Mode (~30-45 min, adaptive)

Topics: Core Concept | Users | Features | Data | Auth | Tech Stack | Business | Edge Cases | Integrations | Design

Q: What's the one-sentence description of what you want to build?
```

**[WAIT FOR ANSWER]**

If `references/main.md` not found → skip loading it and use inline adaptive questioning below.
Then load `references/main.md` for the full adaptive conversation engine if present.

### Rules
- ONE question per message, always
- Acknowledge the answer before asking the next question
- Reference prior answers naturally ("You mentioned X — does that also apply to...")
- Write progress.txt after EVERY exchange (append to INTERROGATION_TRANSCRIPT)
- When resuming: show one-line recap first: "Resuming IDEATE — [X]/10 topics covered. Last: [topic]. Next: [topic]."

### 10 Topics and Their References

If any reference file is missing, skip loading it — use inline adaptive questioning instead. References are optional enhancements, not required.

| # | Topic | Reference |
|---|-------|-----------|
| 1 | Core Concept | `references/core-concept.md` |
| 2 | Users & Personas | `references/users.md` |
| 3 | Features | `references/features.md` |
| 4 | Data & Storage | `references/data.md` |
| 5 | Authentication | `references/auth.md` |
| 6 | Tech Stack + Run Target | `references/tech-stack.md` |
| 7 | Business Model | `references/business.md` |
| 8 | Edge Cases | `references/edge-cases.md` |
| 9 | Integrations | `references/integrations.md` |
| 10 | Design & Brand | `references/design.md` + `references/brand-colors.md` |

### Topic 6 — Run Target (ask explicitly)

```
Q: Where will this app run?
   1. Local machine (npm run dev / python app.py)
   2. Vercel
   3. Netlify
   4. DigitalOcean (App Platform or Droplet)
   5. Desktop app (Electron or Tauri)
   6. Other cloud (Railway, Fly.io, Render — specify)
   7. Not sure yet
```

Save immediately:
```
run_target: [local | vercel | netlify | digitalocean | desktop | other_cloud | unknown]
run_target_detail: [e.g., "Railway", "Electron + Vite"]
```

Also detect and save `has_frontend` based on what the user is building:
- Pure CLI tools, REST APIs only, scripts, backend services → `has_frontend: false`
- Apps with any UI (web, desktop, mobile) → `has_frontend: true` (default)
This flag tells downstream skills (css-setup, design-templates, DESIGN_SYSTEM, FRONTEND_GUIDELINES) whether to run or skip.

### Topic 10 — Design Hook

When user says "I don't know what style" / "help me pick" / "suggest something" / "give me options":
→ Activate `vibe-coding-ui-ux` for style/palette/typography selection.
→ Save selections to `UI_UX_SELECTIONS:` block in progress.txt.

If user already knows their style: continue with `references/design.md` directly.

### Completion

When all 10 topics complete, show summary covering: Problem | Users | Features | Data | Auth | Tech Stack | Run Target | Business | Success Metrics | Edge Cases | Design.

End with: "Does this accurately capture what you want to build? (yes / revise [topic] / missing [something])"

**[WAIT FOR APPROVAL]**

On approval: write `STATUS: complete`, `summary_approved: true`, activate `vibe-coding-document`, exit.

---

## Fast-Track Mode

Write progress.txt before first message:
```
PHASE: IDEATE
STATUS: in_progress
mode: fast_track
started: [timestamp]
questions_asked: 0
NEXT ACTION: Confirm fast-track
```

Show comparison (Full: ~50q/30-45min vs Fast-Track: 10q/10-15min). Ask: "Proceed? (yes/no)"

**[WAIT]**

Then ask questions 1-10, ONE at a time, waiting for each answer:

1. In 2-3 sentences, what are you building and what problem does it solve?
2. Who will use this? One specific person (age, role, context).
3. List the 3-5 must-have features (core functionality only).
4. What data does this app store? List the main "things" (users, tasks, projects...).
5. Does this need user accounts? If yes, what auth method?
6. Preferred tech stack? And where will this run? (show 7 options from Topic 6 above) — save `run_target` immediately after answer.
7. Timeline and budget constraint?
8. How will you measure success? 1-2 specific metrics.
9. Critical edge cases or error scenarios?
10. Visual style preference? (minimal / colorful / professional / you decide)

After Q10, score completeness (0-100%). If ≥70%: summarize and proceed to DOCUMENT. If <70%: list gaps and offer 3 options (fill gaps / proceed anyway / switch to full mode).

After each answer: append `Q[N]: [answer]` to progress.txt and update `questions_asked: [N]`.

---

## Add Features Mode

```
ADD FEATURES MODE

Existing app: [read from reverse-engineer results in progress.txt]
Framework: [existing] | Features: [existing] | Users: [existing]

Q: What new feature do you want to add?
```

Follow same ONE-question pattern, focused on:
- The new feature only
- How it integrates with existing features
- New data models needed
- Impact on existing users

---

## Resume Session

Show: "Resuming IDEATE — [X]/10 topics covered. Last: [most recent complete topic]. Next: [current topic]."

Read INTERROGATION_TRANSCRIPT from progress.txt to restore context.
Ask the next unanswered question. ONE question only.

---

## Already In IDEATE

If IDEATE is already in_progress when this activates:
```
Already in IDEATE — [X] topics covered. Mode: [full/fast-track].
Options: "continue" | "restart" | "switch to fast-track" | "status"
```

**Switching to fast-track mid-session:**
Map completed topics to fast-track questions (skip those already answered). List only remaining questions. Update progress.txt: `mode: fast_track`, `fast_track_switch: true`. Ask first unanswered fast-track question.

---

## Hand-off to DOCUMENT

```
PHASE: IDEATE
STATUS: complete
completed: [timestamp]
summary_approved: true

NEXT ACTION: Generate first document (PRD)

---
PHASE: DOCUMENT
STATUS: ready_to_start
source: ideate_complete
```

Activate `vibe-coding-document`. Exit.
