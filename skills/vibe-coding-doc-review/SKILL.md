---
name: vibe-coding-doc-review
description: |
  Documentation consistency checker. Ensures that docs in ./docs/ match the actual code after changes.
  Checks that IMPLEMENTATION_PLAN.md steps reflect what was built, TECH_STACK.md reflects actual dependencies,
  FRONTEND_GUIDELINES.md patterns match actual component code, and DESIGN_SYSTEM.md tokens match globals.css.
  Use when: (1) A build step modifies something that should update documentation,
  (2) User says "check docs", "are docs up to date", "do the docs match the code?",
  (3) Before SHIP phase as part of pre-flight checks.
  Returns a report of doc/code gaps with specific update instructions.
---

# Vibe Coding — Doc Review

Check that `./docs/` matches actual code. Find gaps before they cause confusion.

## Entry Check

If any referenced file is missing: note "File missing — check skipped" and continue. Do not abort.

Run all 5 checks. This skill is **read-only** — it reports gaps, does not update docs.

---

## Check 1 — TECH_STACK.md vs dependency file

Find dependency file: `package.json` (Node.js) or `requirements.txt`/`pyproject.toml` (Python).

FAIL if: library in TECH_STACK.md not in dependency file, version mismatch, major library in dependency file not documented.

```
TECH_STACK CHECK:
  Listed but not installed: [list]
  Version mismatch: [doc says X, package has Y]
  Not documented: [major libs in package.json missing from TECH_STACK.md]
```

## Check 2 — DESIGN_SYSTEM.md vs globals.css

Find globals.css in: `src/styles/globals.css`, `src/app/globals.css`, `app/globals.css`, `src/index.css`.

FAIL if: CSS variable in globals.css doesn't match DESIGN_SYSTEM.md token name, token in docs missing from CSS, color value differs.

```
DESIGN_SYSTEM CHECK:
  Token missing from globals.css: [list]
  Value mismatch: [doc says #hex1, CSS has #hex2]
  Extra CSS vars not in docs: [list]
```

## Check 3 — FRONTEND_GUIDELINES.md vs component patterns

Sample 3-5 files from `src/components/`.

FAIL if: components missing documented template (no `className` prop, no `FC` type), naming convention violated, state management differs from documented approach.

```
FRONTEND_GUIDELINES CHECK:
  Naming violations: [list]
  Pattern violations: [list]
  State management: [code uses X, docs say Y]
```

## Check 4 — IMPLEMENTATION_PLAN.md vs progress.txt

FAIL if: step marked complete in progress.txt but relevant files don't exist, plan references feature that was changed/removed.

```
IMPLEMENTATION_PLAN CHECK:
  Marked complete but files missing: [Phase X Step Y]
  Plan references removed feature: [describe]
```

## Check 5 — BACKEND_STRUCTURE.md vs actual API routes

Detect API style from TECH_STACK.md, then scan the correct location:

```
Next.js (REST)  → scan src/app/api/ or src/pages/api/
Express/Fastify → scan src/routes/ or routes/
FastAPI         → scan main.py or routers/ for @app.get/@app.post decorators
Django          → scan urls.py files for urlpatterns
Flask           → scan for @app.route decorators in *.py
tRPC            → scan src/server/routers/ for .query/.mutation procedures
Go              → scan for http.HandleFunc or router.GET/POST patterns
Ruby on Rails   → scan config/routes.rb
has_frontend: false AND no backend in stack → skip check with note "API-only project — no routes to verify"
```

FAIL if: endpoint documented but route file/handler missing, route exists but undocumented, HTTP method mismatch.

```
BACKEND_STRUCTURE CHECK:
  Documented but missing: [GET /api/products/[id]]
  Exists but undocumented: [POST /api/webhooks]
  Method mismatch: [docs say GET, route is POST]
```

---

## Report Format

```
DOC REVIEW REPORT

TECH_STACK.md: [PASS | N issues]
DESIGN_SYSTEM.md: [PASS | N issues]
FRONTEND_GUIDELINES.md: [PASS | N issues]
IMPLEMENTATION_PLAN.md: [PASS | N issues]
BACKEND_STRUCTURE.md: [PASS | N issues]

MUST UPDATE BEFORE SHIP: [critical gaps]
SHOULD UPDATE: [minor gaps]

SPECIFIC FIXES:
  1. Update [doc]: [what to change]
  2. [etc.]

[All pass:] DOCS ARE IN SYNC — ready for SHIP.
```

If docs are significantly out of date, recommend re-running `vibe-coding-document` for that specific doc.
