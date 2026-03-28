---
name: vibe-coding-security-review
description: |
  Security audit skill — scans code for OWASP Top 10 vulnerabilities, hardcoded secrets, injection flaws, insecure auth patterns, and exposed API keys.
  Use when: (1) Before SHIP phase — automatic pre-flight security gate,
  (2) User says "security check", "audit security", "check for vulnerabilities", "security review",
  (3) After BUILD completes a phase that touches auth, API, or database code,
  (4) vibe-coding-build delegates before any deployment step.
  Outputs a structured security report with CRITICAL/HIGH/MEDIUM/LOW severity ratings and exact file:line references.
---

# Vibe Coding — Security Review

Scan for vulnerabilities before shipping. CRITICAL or HIGH findings block deployment.

## Scanning Process

Run all 10 checks. For each: grep for patterns, collect file:line findings, classify severity, note exact fix.

Skip: `.env`, `.env.local`, `.env.*`, `node_modules/`, `*.test.*`

**Run checks sequentially. Jump directly to each ## Check N section — do not read all checks upfront. Report CRITICAL findings immediately without waiting for remaining checks to complete.**

---

## Check 1 — Hardcoded Secrets (CRITICAL)

Scan for env var patterns even if .env file is absent — look for `process.env.*`, `os.environ*`, `VITE_*` patterns in code. Do not block scanning if .env is absent.

Scan `**/*.{js,ts,jsx,tsx,py,rb,go,env.example}` for:
- API keys: `sk-[a-z0-9]{32,}`, `AIza[0-9A-Za-z-_]{35}`, `AKIA[0-9A-Z]{16}`
- Passwords: `password\s*=\s*["'][^"']{4,}["']`
- Tokens: `token\s*=\s*["'][^"']{8,}["']`
- Connection strings with embedded credentials: `mongodb://[^@]+:[^@]+@`, `postgres://[^@]+:[^@]+@`
- Private keys: `-----BEGIN (RSA|EC|OPENSSH) PRIVATE KEY-----`

Fix: move to environment variable, reference via `process.env.VAR_NAME`.

## Check 2 — SQL Injection (CRITICAL)

FAIL if: string concatenation in DB queries in any language.
```
"SELECT * FROM users WHERE id = " + userId
db.execute(f"SELECT ... WHERE name = '{name}'")
```
Fix: parameterized queries, prepared statements, ORM query builders.

## Check 3 — XSS (HIGH)

FAIL if: `dangerouslySetInnerHTML` with unsanitized input, `innerHTML = userInput`, `eval(userInput)`, server-side `res.send(userInput)` without escaping.
Fix: DOMPurify for sanitization, React JSX default escaping, template engine auto-escaping.

## Check 4 — Insecure Auth (HIGH)

FAIL if: `jwt.decode()` instead of `jwt.verify()`, JWT without `expiresIn`, weak secret (`"secret"`, `"password"`), disabled HTTPS in production, missing CSRF on state-changing endpoints, plaintext passwords or MD5/SHA1 hashing.

## Check 5 — IDOR (HIGH)

FAIL if: API endpoints fetch by ID without ownership check, file paths from user input (`fs.readFile(req.params.filename)`).
Fix: always verify authenticated user owns or has permission for the requested resource.

## Check 6 — Sensitive Data Exposure (MEDIUM)

FAIL if: password hash in API response, logging sensitive fields (`console.log(user.password)`), `res.send(err.stack)`, debug endpoints left active, unmasked PII in responses.

## Check 7 — Dependency Vulnerabilities (MEDIUM)

Read package.json/requirements.txt for known vulnerable versions:
- Node.js: lodash <4.17.21, axios <0.21.1, jsonwebtoken <9.0.0
- Python: django <4.1, flask <2.2, requests <2.28.0

Advisory: suggest `npm audit` or `pip-audit` for full scan.

## Check 8 — Env Variable Exposure (MEDIUM)

FAIL if: secrets prefixed with `NEXT_PUBLIC_`, client-side code accessing server-only vars, `.env` committed to git (check .gitignore).

## Check 9 — Path Traversal (MEDIUM)

FAIL if: `fs.readFile(path.join(baseDir, userInput))` without checking result starts with allowed base dir, `req.params.filename` passed directly to file ops.

## Check 10 — Rate Limiting (LOW)

FAIL if: login/auth endpoints without rate limiting, no request throttling on public APIs.
Fix: `express-rate-limit`, Django's `django-ratelimit`, FastAPI's `slowapi`.

---

## Report Format

```
SECURITY REVIEW REPORT
======================
CRITICAL: [N]  HIGH: [N]  MEDIUM: [N]  LOW: [N]  PASSED: [N]

CRITICAL FINDINGS:
[1] [check name] — [file:line]
    Code: [offending code]
    Fix:  [specific fix]

HIGH FINDINGS: [same pattern]
MEDIUM FINDINGS: [same pattern]
LOW FINDINGS: [same pattern]

PASSED: ✓ [list clean checks]

VERDICT: [BLOCKED | APPROVED]
[BLOCKED]: Resolve all CRITICAL and HIGH before SHIP. Re-run after fixes.
[APPROVED]: No CRITICAL/HIGH issues. MEDIUM/LOW are advisory — address before v1 launch.
```

Verdict rules: any CRITICAL or HIGH = BLOCKED. Only MEDIUM/LOW or clean = APPROVED.

---

## Integration

Write to progress.txt after scan:
```
SECURITY_REVIEW:
  status: [blocked | approved]
  critical: [N]  high: [N]  medium: [N]  low: [N]
```

On BLOCKED: return to vibe-coding-build for fixes. Do NOT proceed to deploy.
On APPROVED: return to vibe-coding-ship with report summary.
