---
name: vibe-coding-re-generate
description: |
  Generate all 8 documentation files from reverse-engineering analysis results.
  First generates 5 technical docs directly from code analysis. Then asks ~10 business
  context questions to fill gaps. Then generates remaining 3 business docs.
  This is Stage 3 of 3 in the reverse-engineering pipeline. Runs after vibe-coding-re-analyze.
  Use when: vibe-coding-re-analyze completes and hands off with confirmed analysis.
  Outputs all 8 canonical docs to ./docs/ and offers next steps.
---

# Vibe Coding — Doc Generation from Analysis (RE Stage 3)

Generate all 8 docs from confirmed analysis. Wave 1: 5 technical docs. Wave 2: Q&A. Wave 3: 3 business docs.

## Prerequisites

Read `progress.txt` — must show `STATUS: analysis_complete`. If not, redirect to `vibe-coding-re-analyze`.

---

## Wave 1 — 5 Technical Docs from Code

Generate without questions. Delegate to the relevant doc skill for each:

1. **TECH_STACK.md** → activate `vibe-coding-doc-techstack` with analysis data
2. **BACKEND_STRUCTURE.md** → activate `vibe-coding-doc-backend` with models + endpoints
3. **FRONTEND_GUIDELINES.md** → activate `vibe-coding-doc-frontend` with detected patterns (document ACTUAL patterns, not idealized)
4. **DESIGN_SYSTEM.md (partial)** → activate `vibe-coding-doc-design` with CSS vars + font names. Mark tokens as inferred vs. explicitly found.
5. **APP_FLOW.md (partial)** → activate `vibe-coding-doc-appflow` with route/page list

If a sub-skill fails: mark "FAILED — [reason]", continue with remaining docs, add to Q&A gap list.

After Wave 1:
```
WAVE 1 COMPLETE — 5 technical docs generated.
Generated: ✓ TECH_STACK.md | ✓ BACKEND_STRUCTURE.md | ✓ FRONTEND_GUIDELINES.md
           ✓ DESIGN_SYSTEM.md (partial) | ✓ APP_FLOW.md (partial)

Now I need ~[N] business context questions to complete the remaining 3 docs. Ready? (yes/no)
```

---

## Wave 2 — Business Context Q&A

Ask ONE question at a time. Required questions:

1. "What problem does this product solve? Who has this problem?"
2. "Who are your target users? Describe the primary persona."
3. "What are the 3 most important things this product does right now?"
4. "What's the business model? (SaaS subscription, ecommerce, freemium, one-time purchase, etc.)"
5. "What metrics define success for this product?"

Conditional (ask based on gaps from analysis):
- APP_FLOW low confidence: "Walk me through the main user journey from [entry] to [goal]."
- Auth roles unclear: "What user roles exist? What can each role do?"
- Complex logic purpose unclear: "What does [module] do from a business perspective?"
- Integrations purpose unclear: "I see [Stripe/etc] integrated — what features use it?"
- Incomplete features detected: "I see [TODO/incomplete]. Is this intentional or WIP?"
- Design system inconsistent: "Want to clean up the design system as part of this project?"

---

## Wave 3 — 3 Business Docs

After Q&A completes, generate docs ONE AT A TIME with approval between each:

1. **PRD.md** → activate `vibe-coding-doc-prd`. Flag features found in code not mentioned in Q&A — check if still needed. After generation: "PRD generated. Approve or request revisions?"

2. **Enhanced DESIGN_SYSTEM.md** → fill Wave 1 partial doc with Q&A answers. After generation: "DESIGN_SYSTEM.md updated. Approve or request revisions?"

3. **Enhanced APP_FLOW.md** → fill Wave 1 partial doc with Q&A answers. After generation: "APP_FLOW.md updated. Approve or request revisions?"

4. **IMPLEMENTATION_PLAN.md** → activate `vibe-coding-doc-implplan` with `mode: enhancement` (plan WHAT'S NEXT, not rebuilding from scratch). After generation: "IMPLEMENTATION_PLAN.md generated. Approve or request revisions?"

5. **CLAUDE.md** → activate `vibe-coding-doc-claudemd`. After generation: "CLAUDE.md generated. Approve or request revisions?"

Wait for user approval at each step before proceeding to the next doc.

---

## Completion

```
REVERSE-ENGINEERING COMPLETE — all 8 docs in ./docs/

What next?
  1. Add new features → IDEATE phase
  2. Improve existing code → BUILD phase
  3. Fix design inconsistencies → vibe-coding-doc-design + vibe-coding-css-setup
  4. Just keep the docs → Exit
```

Write to progress.txt:
```
PHASE: REVERSE-ENGINEER
STATUS: complete
docs_generated: 8
entry_point: [user's choice 1-4]
```
