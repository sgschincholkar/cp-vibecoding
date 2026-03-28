---
name: vibe-coding-doc-claudemd
description: |
  Generate CLAUDE.md — the project-specific AI assistant rules file.
  Tells Claude exactly how to behave in this codebase: what to read first,
  project conventions, what to never do, and how to navigate the project.
  Use when: (1) vibe-coding-document delegates doc #8 (CLAUDE.md),
  (2) User says "generate CLAUDE.md", "create AI rules", "write the project instructions".
  Reads all 7 preceding docs to synthesize rules. Outputs to project ROOT (not ./docs/).
---

# Vibe Coding — CLAUDE.md Generator

Generate CLAUDE.md at project root. Loaded on every Claude invocation — keep it lean.

## Prerequisites Check

1. Read `progress.txt` — `run_target` field first (determines start command and deploy section)
2. Read `./docs/TECH_STACK.md` — exact technologies
3. Read `./docs/DESIGN_SYSTEM.md` — visual tokens
4. Read `./docs/FRONTEND_GUIDELINES.md` — component patterns
5. Read `./docs/BACKEND_STRUCTURE.md` — data models and API
6. Read `./docs/APP_FLOW.md` — screens and routes
7. Read `./docs/IMPLEMENTATION_PLAN.md` — build order

Detect stack adapters needed:
```
Python backend (Flask/Django/FastAPI) → READ: ## Python Adapter
Go / Ruby / Rust / PHP backend        → READ: ## Generic Backend Adapter
Desktop app (Electron/Tauri)          → READ: ## Desktop Adapter
run_target = desktop                  → READ: ## Desktop Adapter
has_frontend: false                   → omit Design Rules and Component Rules sections entirely
```

Check output size before finalizing → READ: ## Size Management

---

## Generation Steps

1. Write Overview (2-3 sentences, stack, type, run_target, start command)
2. Write Always Read First list (4 files)
3. Write Project Structure (abbreviated — key folders only)
4. Write Design Rules (5 critical rules)
5. Write Component Rules
6. Write API Rules (adapt for desktop if needed)
7. Write State Rules (adapt for Python if needed)
8. Write File Safety rules
9. Write Current Implementation Phase (how to resume)
10. Derive and write Common Gotchas (minimum 3, project-specific)
11. Write Testing section (adapt for Python/Desktop if needed)
12. Save to `./CLAUDE.md` (project root, NOT ./docs/)
13. Run Validation Checklist
14. Ask: "CLAUDE.md generated. Approve or request revisions?"

## Output Format

```markdown
# [APP NAME] — Claude Code Instructions

## Project Overview
[2-3 sentences: what this is, who it's for, what it does]

Stack: [Framework] + [CSS] + [Database] + [Auth]
Type: [Ecommerce | SaaS | Content | Dashboard | Landing | Portfolio | Desktop]
Run target: [Local machine | Vercel | Netlify | DigitalOcean | Desktop | Other cloud]
Start command: [npm run dev | flask run | python manage.py runserver | npm run tauri dev]

## Always Read First
1. ./docs/DESIGN_SYSTEM.md — colors, fonts, spacing tokens
2. ./docs/FRONTEND_GUIDELINES.md — component patterns and rules
3. ./docs/IMPLEMENTATION_PLAN.md — current phase and step
4. progress.txt — current build state

## Project Structure
[Abbreviated key folders — 15 lines max]

## Design Rules (CRITICAL)
1. Colors from tokens only — never hardcode hex
   - Tailwind: bg-primary, text-text-muted | CSS vars: var(--color-primary)
2. Fonts from globals.css only — never font-family in component files
3. Spacing via Tailwind scale — no arbitrary values like mt-[13px]
4. Pages extend base layout — never re-declare Header/Footer/Sidebar in page files
5. Run vibe-coding-design-guard before any page or component is committed

## Component Rules
- One component per file, PascalCase name
- Props interface named ComponentNameProps above component
- Always accept className, merge with cn()
- No inline styles (CSS var refs in style={{}} only for dynamic values)

## API Rules
[Standard HTTP rules OR Desktop IPC rules — from adapter if applicable]

## State Rules
[Standard JS rules OR Python ORM rules — from adapter if applicable]

## File Safety
- Never overwrite files without reading them first
- Never modify ./docs/ files — reference only
- Never modify globals.css design token values
- One change at a time — test before next step

## Current Implementation Phase
See ./docs/IMPLEMENTATION_PLAN.md and progress.txt for current phase and step.

To resume: read progress.txt → find current phase → continue from last incomplete step.

Start command: [from run_target]
App runs at: [URL or "opens desktop window"]

## Common Gotchas
- [Derived from actual tech choices — minimum 3, no placeholders]

## Testing
[Standard or adapted — from matching adapter section]

## When Unclear
1. Check ./docs/[relevant].md first
2. Check progress.txt for recent decisions
3. Ask the user — don't assume
```

## Generation Rules

1. Common Gotchas are project-specific — derived from actual docs, minimum 3:
   - Cart + Zustand: "Cart state in Zustand cartStore — never useState for cart items"
   - File uploads: "All uploads go through [storage service] — never store binary in DB"
   - Complex auth roles: "Role checking server-side via middleware — never trust client-side role values"
   - Derive from BACKEND_STRUCTURE.md complexity and APP_FLOW.md edge cases
2. No placeholder text like "[Project-specific gotcha 1]" — fill them in
3. File goes to project root — never ./docs/
4. Keep under 150 lines total

## Validation Checklist

```
CLAUDE.md VALIDATION:
✓/✗ Stack summary accurate
✓/✗ run_target present in overview
✓/✗ Start command correct for stack
✓/✗ Design rules section present
✓/✗ API rules match backend approach (HTTP or IPC)
✓/✗ State/testing rules adapted for stack (Python or Desktop if needed)
✓/✗ File saved to project root (not ./docs/)
✓/✗ Common Gotchas have 3+ project-specific items (no placeholders)
✓/✗ Under 150 lines total
✓/✗ No full code examples
```

---

## Generic Backend Adapter

For Go, Ruby, Rust, PHP, or any other backend not covered by specific adapters:

**Start command:** Derive from TECH_STACK.md (e.g., `go run main.go`, `rails server`, `cargo run`, `php artisan serve`)

**State Rules:**
```
- No client-side state library — server handles all data state
- Database ORM from TECH_STACK.md manages persistence
- Never cache server data in client memory
```

**Testing:**
```
- Use language-native test runner (e.g., go test, rspec, cargo test, phpunit)
- Test files follow language convention (e.g., *_test.go, spec/, tests/)
- API tests: use HTTP client appropriate for language
```

**Common Gotcha to include:**
- "This project uses [language] — do not suggest JS/TS npm commands or node_modules"

---

## Python Adapter

Replace these sections when backend is Python (Flask/Django/FastAPI):

**Start command:** `flask run` | `python manage.py runserver` | `uvicorn main:app --reload`

**State Rules:**
```
- Server data managed by SQLAlchemy / Django ORM — no client-side state library
- UI state: component-level state (React hooks or equivalent)
- Never duplicate server data in client state
```

**Testing:**
```
- Unit tests: pytest — test files in tests/ directory
- Run tests: pytest or python -m pytest
- API tests: use httpx.AsyncClient with FastAPI test client
```

---

## Desktop Adapter

Replace these sections when run_target is desktop (Electron/Tauri):

**Start command:** `npm run electron:dev` | `npm run tauri dev`

**API Rules:**
```
- Desktop apps use IPC — NOT HTTP APIs for core data
- Electron: ipcMain / ipcRenderer for main-to-renderer messaging
- Tauri: Tauri commands (#[tauri::command]) invoked via invoke() in frontend
- External API calls (third-party services) may still use fetch from renderer
```

**Testing:**
```
- E2E: Playwright with Electron launch config
- Unit: Vitest for renderer-side logic
- IPC handlers: Jest/Vitest mocking ipcMain
```

---

## Size Management

| Size | Status | Action |
|------|--------|--------|
| Under 150 lines | Healthy | No action |
| 150-250 lines | Getting heavy | Move content to ./docs/ |
| Over 250 lines | Too heavy | Split immediately |

**Split pattern:** Move heavy sections to `./docs/claude/[topic].md`. Replace with pointer line:
```markdown
## Design Rules
See ./docs/claude/design-rules.md — read before writing any component.
```

Keep CLAUDE.md under 100 lines after split. Trigger split when: >250 lines, >5 headers, full code examples, or user reports Claude forgetting rules.

To split: activate `vibe-coding-doc-claudemd` with instruction "split CLAUDE.md — it's too heavy".
