---
name: vibe-coding-review-react
description: |
  React and Next.js code review — 65 rules across 8 categories covering async waterfalls, bundle size, server-side performance, client-side data fetching, re-render optimization, rendering performance, JavaScript performance, and advanced patterns.
  Also covers React composition patterns: compound components, render props, context providers, React 19 APIs.
  Use when: (1) vibe-coding-code-review detects React/Next.js project and delegates here,
  (2) User says "review my React code", "check my Next.js component", "optimize this component",
  (3) User says "check for performance issues", "review data fetching", "audit bundle size".
  Returns findings with severity (CRITICAL/HIGH/MEDIUM/LOW) and exact file:line references.
  Source: Vercel agent-skills react-best-practices + composition-patterns.
---

# Vibe Coding — React & Next.js Code Review

65 rules for production-quality React and Next.js. Called by `vibe-coding-code-review` when React/Next.js detected.

## Entry Router

```
Verify React project first:
  Check package.json for "react" — if NOT found → stop: "Not a React project. Use vibe-coding-code-review for the detected language."
  If Vue/Svelte/Angular detected → stop: "This skill is for React/Next.js only."

Run categories in priority order. Report CRITICAL issues immediately.

Category 1 (CRITICAL): Async Waterfalls → READ: ## Cat 1
Category 2 (CRITICAL): Bundle Size → READ: ## Cat 2
Category 3 (HIGH): Server-Side Performance → READ: ## Cat 3
Category 4 (MEDIUM-HIGH): Client-Side Data Fetching → READ: ## Cat 4
Category 5 (MEDIUM): Re-render Optimization → READ: ## Cat 5
Category 6 (MEDIUM): Rendering Performance → READ: ## Cat 6
Category 7 (LOW-MEDIUM): JavaScript Performance → READ: ## Cat 7
Category 8 (MEDIUM): Composition Patterns → READ: ## Cat 8
```

**Jump directly to each ## Cat N section in order. Do not read ahead to later categories before completing the current one.**

---

## Cat 1: Async Waterfalls (CRITICAL)

FAIL if: fetch() calls inside useEffect depend on previous fetch results, components fetch then render children that also fetch, sequential awaits that could be parallelized, client components fetching what Server Components could handle.

```tsx
// FAIL: waterfall
const user = await getUser(id)
const posts = await getPosts(user.id)  // waits for user
// PASS: parallel
const [user, posts] = await Promise.all([getUser(id), getPosts(id)])
```

Next.js: check for data fetching in layout.tsx blocking all children, missing Suspense boundaries, useEffect for data that should be in Server Components.

---

## Cat 2: Bundle Size (CRITICAL)

FAIL if: entire library imported for one function, no dynamic imports on large components, moment.js used, full icon library imports, `<img>` instead of `next/image`, "use client" on page-level components.

```tsx
// FAIL: import _ from 'lodash'
// PASS: import debounce from 'lodash/debounce'
// FAIL: import HeavyChart from './HeavyChart'
// PASS: const HeavyChart = dynamic(() => import('./HeavyChart'))
```

---

## Cat 3: Server-Side Performance (HIGH)

Next.js App Router checks:
- Missing `cache()` for repeated DB calls
- No revalidation strategy: `fetch(url)` caches forever — use `{ next: { revalidate: 60 } }` for dynamic data
- N+1 queries in Server Components (loop calling DB per item)
- Missing `generateStaticParams` for routes that could be static
- No Suspense + streaming for slow data

---

## Cat 4: Client-Side Data Fetching (MEDIUM-HIGH)

FAIL if: raw `useEffect + fetch` instead of SWR/React Query, missing loading/error states, no request deduplication, `useEffect` with missing deps (stale closures), polling without cleanup (`setInterval` without `clearInterval`).

---

## Cat 5: Re-render Optimization (MEDIUM)

FAIL if: object/array literals as props (new ref every render), inline function props on frequently-rendered components, context causing all consumers to re-render, missing `React.memo` on expensive pure components, `useMemo`/`useCallback` on cheap operations (adds overhead).

---

## Cat 6: Rendering Performance (MEDIUM)

FAIL if: lists >100 items without virtualization (`react-window`/`react-virtual`), array index as key (`key={index}`), expensive calculations on every render without `useMemo`.

```tsx
// FAIL: items.map((item, i) => <Item key={i} />)
// PASS: items.map(item => <Item key={item.id} />)
```

---

## Cat 7: JavaScript Performance (LOW-MEDIUM)

FAIL if: `.filter().map()` chained (two passes — use `.reduce()`), `JSON.parse(JSON.stringify(...))` in hot paths (use `structuredClone()` or immer), regex compiled inside loops, no debounce on search/filter inputs.

---

## Cat 8: Composition Patterns (MEDIUM)

FAIL if: boolean prop proliferation (>3 boolean props), component >8 props, missing compound component pattern for related UI groups, prop drilling >2 levels.

```tsx
// FAIL: <Button primary large rounded disabled loading />
// PASS: <Button variant="primary" size="large" state="loading" />
```

React 19 patterns — FAIL if:
- `forwardRef()` used (React 19: just accept `ref` as a prop directly)
- `useFormState` used without wrapping in `<form action={serverAction}>` (Server Actions pattern)
- `use()` hook called conditionally (violates Rules of Hooks — `use()` must be called unconditionally)
- `React.createContext()` used without wrapping in `<Context>` provider (React 19 allows `<Context>` directly)
- `createContext` consumed via `useContext` where the new `use(Context)` pattern is more readable for async
- PASS: `use(promise)` in Server Components, `useOptimistic` for optimistic UI, `useFormStatus` in form children

---

## Output Format

```
REACT CODE REVIEW
=================
File: [path]
Framework: [React | Next.js App Router | Next.js Pages Router]

CRITICAL: [N]  HIGH: [N]  MEDIUM: [N]  LOW: [N]

CRITICAL FINDINGS:
[1] [file:line] — [description]
    Fix: [exact fix]
    Impact: [effect on performance/bundle]

HIGH FINDINGS:
[2] [file:line] — [description] + fix

MEDIUM FINDINGS: [...]
LOW FINDINGS: [...]

PASSED: ✓ [list passing checks]
```
