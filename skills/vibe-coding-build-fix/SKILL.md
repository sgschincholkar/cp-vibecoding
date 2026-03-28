---
name: vibe-coding-build-fix
description: |
  Specialized build/compile error resolver — diagnoses and fixes build failures, type errors, import errors, and dependency issues.
  Use when: (1) Build command fails (npm run build, tsc, pytest, go build),
  (2) User pastes a compile error, type error, or import error,
  (3) vibe-coding-debug activates this when error is a build/compile failure (not a runtime logic bug),
  (4) User says "build is broken", "type error", "import error", "cannot find module", "build failed".
  Different from vibe-coding-debug: this handles compile-time errors. Debug handles runtime errors.
  Outputs root cause + minimal fix with exact file:line changes.
---

# Vibe Coding — Build Fix

Resolve build failures fast. Compile-time and tooling errors only — not runtime bugs.

## Entry Router

```
Classify the error first:

TS[4-digit code]?            → READ: ## TypeScript Errors
"Cannot find module"?        → READ: ## Module Errors
Vite/Webpack/Next message?   → READ: ## Next.js Errors
Peer dep / ERESOLVE?         → READ: ## Dependency Conflicts
"Unknown at rule" / Tailwind? → READ: ## Tailwind Errors
ModuleNotFoundError (Python)? → READ: ## Python Errors
ESLint blocking build?       → READ: ## Linter Errors
"undefined: [name]" / Go?    → READ: ## Go Errors
"error[E...]: [msg]" / Rust? → READ: ## Rust Errors
```

Scope: does NOT handle runtime errors (use vibe-coding-debug), test failures (use vibe-coding-tdd), or security issues (use vibe-coding-security-review).

**Jump directly to the matching ## [Error Type] section. Do not read all error sections — only the one matched by the entry router above.**

---

## Diagnostic Process

1. Parse error: extract error code, file, line number, exact message
2. Read only relevant lines (line-10 to line+10) — not entire file
3. Apply the matching fix pattern below
4. Make the smallest change that resolves the error — no refactoring

---

## TypeScript Errors

**TS2345** — Argument not assignable: read function signature, find type mismatch. Fix: add type guard, fix source type, or `value ?? defaultValue`.

**TS2339** — Property does not exist: add property to interface, use optional chaining `obj?.prop`, or check wrong object.

**TS7006** — Implicit any: add explicit type annotation. `(x) => x.name` → `(x: User) => x.name`

**TS2307** — Cannot find module: check package installed (`grep "[m]" package.json`), install `@types/[module]`, check tsconfig paths, fix relative path depth.

**TS2322** — Type not assignable to type: align types — change variable type or change value being assigned.

**TS18047/18048** — Possibly null: use `value!` (if certain), `obj?.prop`, `value ?? fallback`, or early return guard.

---

## Module Errors

**Cannot find module '[package]':** check `grep "[p]" package.json` → if missing run `npm install [package]` → if installed but not found: delete node_modules, run `npm install`.

**Cannot find module '../[relative]':** verify file exists at path, check case sensitivity (Linux is case-sensitive), fix path depth.

**Path aliases (@/components/...):** check `tsconfig.json compilerOptions.paths` AND `vite.config.ts resolve.alias` — both must match.

```ts
// vite.config.ts
resolve: { alias: { '@': path.resolve(__dirname, './src') } }
```

---

## Next.js Errors

**Missing "use client" directive:** component uses browser-only API or hook in Server Component. Add `"use client"` at top of file.

**NEXT_PUBLIC_ undefined:** check `.env.local` has the var, restart dev server, verify no non-NEXT_PUBLIC_ vars accessed on client.

**Hydration mismatch:** server/client render differs. Fix pattern:
```tsx
const [mounted, setMounted] = useState(false)
useEffect(() => setMounted(true), [])
if (!mounted) return null
```
Cause: `Date.now()`, `Math.random()`, or `window`/`localStorage` in component body.

---

## Tailwind Errors

**Styles not applying:** check `tailwind.config.js content` array covers all file paths, check `globals.css` has `@tailwind` directives, restart dev server.

**Unknown at rule @apply:** add to `.vscode/settings.json`: `"css.customData": [".vscode/tailwind.json"]`

**CSS variable undefined:** check `:root` in globals.css defines the var, check `theme.extend.colors` in tailwind.config.js, verify exact name match.

---

## Dependency Conflicts

**ERESOLVE / peer dep conflict:**
1. Upgrade conflicting package to compatible version
2. Use `--legacy-peer-deps` (temporary workaround)
3. Add `overrides` in package.json: `"overrides": { "[pkg]": "[version]" }`

**Duplicate versions:** `npm ls [package]` then `npm dedupe`.

---

## Python Errors

**ModuleNotFoundError:** check `grep "[p]" requirements.txt` or `pip show [pkg]` → if missing: `pip install [pkg]` → if installed but not found: verify venv activated (`which python` should show `.venv` path) → if custom package: check `__init__.py` exists, run `pip install -e .`.

**IndentationError:** check for mixed tabs/spaces, use spaces only.

---

## Go Errors

**undefined: [name]:** symbol not in scope — check import path, check package name matches directory name, run `go mod tidy`.

**imported and not used:** remove the unused import or use `_` alias: `import _ "pkg"` (for side effects only).

**cannot use [type] as type [type]:** Go is strictly typed — check interface implementation, missing method, or wrong return type. Use explicit conversion.

**declared and not used:** all declared vars must be used — either use it, rename to `_`, or remove.

**go: [module] requires go >= [version]:** update `go.mod` `go` directive to required version.

**missing go.sum entry:** run `go mod tidy` to regenerate go.sum.

---

## Rust Errors

**error[E0308] mismatched types:** type mismatch — check function signature vs call site, use `.into()` for conversions, check `Option<T>` vs `T`.

**error[E0382] borrow of moved value:** value used after move — use `.clone()`, borrow (`&value`), or restructure to avoid move.

**error[E0502] cannot borrow as mutable:** mutable borrow while immutable borrow active — end borrow scope before taking mutable borrow.

**error[E0507] cannot move out of [place]:** cannot move out of reference — use `.clone()` or change to owned type.

**error[E0277] trait bound not satisfied:** type doesn't implement required trait — add `#[derive(Debug)]`/`Clone`/etc., or implement manually.

**Cargo.lock conflict after edit:** run `cargo build` to regenerate, or `cargo update [crate]` for specific dependency.

---

## Linter Errors

Run `npm run lint` locally. Fix each error or add inline disable for false positives only:
```ts
// eslint-disable-next-line [rule-name]
```
NEVER disable: `react-hooks/rules-of-hooks`, `react-hooks/exhaustive-deps`, `no-unused-vars` (remove the var instead).

---

## Fix Output Format

```
BUILD FIX REPORT
================
Error type: [classification]
File: [path:line]

ROOT CAUSE:
[One paragraph explaining why this broke]

FIX:
[file:line]
Before: [original code]
After:  [fixed code]

VERIFY:
Run: [build command]
Expected: Build succeeds with no errors
```

After fix verified: write to progress.txt:
```
BUILD_FIX:
  error_type: [classification]
  file: [path]
  resolved: true
  fix_summary: [one line]
```

Return control to vibe-coding-build at the step where the error occurred (or back to vibe-coding-debug if called from there).
