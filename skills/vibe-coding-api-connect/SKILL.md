---
name: vibe-coding-api-connect
description: |
  REST and GraphQL API integration patterns — auth (API key, OAuth2, Bearer token), pagination, error handling, retry logic, response typing, and rate limit handling.
  Use when: (1) BUILD hits a step that integrates an external API,
  (2) User says "integrate [API name]", "connect to [service]", "call [API] endpoint", "fetch data from [service]",
  (3) IMPLEMENTATION_PLAN has a step involving an external data source (Stripe, GitHub, OpenAI, custom REST, etc.),
  (4) User says "add API integration", "how do I call [API]", "set up [service] client".
  Generates typed API client code, auth setup, error handling wrapper, and environment variable config.
  When this skill grows beyond 400 lines OR covers 3+ named APIs with dedicated setup (e.g., Stripe + Google Ads + Twilio),
  break into per-API skills: vibe-coding-api-[name] (e.g., vibe-coding-api-stripe).
---

# Vibe Coding — API Connect

Generate production-ready, typed API integration code.

## Entry Router

```
0. If API name not provided and no BUILD context → ask: "What API are you integrating with? (e.g., Stripe, GitHub, custom endpoint, GraphQL URL)" before routing.
1. Check if a dedicated per-API skill exists (vibe-coding-api-[name]) → use that if so
2. Detect backend language from progress.txt or TECH_STACK.md:
   Node.js/TypeScript → READ: ## TypeScript Client
   Python async (FastAPI) → READ: ## Python Async Client
   Python sync (Flask/Django) → READ: ## Python Sync Client
3. Detect auth method from API docs/user message:
   API Key → READ: ## Auth: API Key
   OAuth2 server-to-server → READ: ## Auth: OAuth2 Client Credentials
   OAuth2 user-facing → READ: ## Auth: OAuth2 User
4. Pagination needed? → READ: ## Pagination
5. GraphQL? → READ: ## GraphQL
6. Webhooks needed? → READ: ## Webhooks: [TypeScript | Python]
```

Split rule: if this skill exceeds 400 lines OR project uses 3+ named APIs with distinct auth, create `vibe-coding-api-[name]` per API.

**Jump directly to the section matched by the entry router above. Do not read all auth patterns — only the one that matches the detected auth type.**

---

## Auth: API Key

```typescript
// .env.local: [SERVICE]_API_KEY=your_key_here
const API_KEY = process.env.[SERVICE]_API_KEY
if (!API_KEY) throw new Error('[Service] API key not configured')
const headers = { 'Authorization': `Bearer ${API_KEY}`, 'Content-Type': 'application/json' }
// (check API docs — may use 'X-API-Key' or 'api-key' instead)
```

---

## Auth: OAuth2 Client Credentials

```typescript
// .env.local: [SERVICE]_CLIENT_ID=xxx, [SERVICE]_CLIENT_SECRET=xxx, [SERVICE]_TOKEN_URL=https://...
let cachedToken: { access_token: string; expires_at: number } | null = null

async function getAccessToken(): Promise<string> {
  if (cachedToken && Date.now() < cachedToken.expires_at - 60_000) return cachedToken.access_token
  const response = await fetch(process.env.[SERVICE]_TOKEN_URL!, {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body: new URLSearchParams({ grant_type: 'client_credentials',
      client_id: process.env.[SERVICE]_CLIENT_ID!,
      client_secret: process.env.[SERVICE]_CLIENT_SECRET! }),
  })
  if (!response.ok) throw new ApiError('[Service] token fetch failed', response.status)
  const data = await response.json()
  cachedToken = { access_token: data.access_token, expires_at: Date.now() + data.expires_in * 1000 }
  return cachedToken.access_token
}
```

---

## Auth: OAuth2 User

Use NextAuth.js (JS) or authlib (Python) — do not implement manually.

```typescript
// next-auth config
import NextAuth from 'next-auth'
import [Provider] from 'next-auth/providers/[provider]'
export const { handlers, auth } = NextAuth({
  providers: [[Provider]({ clientId: process.env.[PROVIDER]_CLIENT_ID, clientSecret: process.env.[PROVIDER]_CLIENT_SECRET })]
})
```
```python
# Python: pip install authlib
from authlib.integrations.flask_client import OAuth  # or django_client
oauth = OAuth()
oauth.register("[provider]", client_id=os.environ["[PROVIDER]_CLIENT_ID"], ...)
```

---

## TypeScript Client

```typescript
// lib/api-client.ts
export class ApiError extends Error {
  constructor(message: string, public status: number, public body?: unknown) {
    super(message); this.name = 'ApiError'
  }
}

export async function apiFetch<T>(url: string, options: RequestInit & { retries?: number } = {}): Promise<T> {
  const { retries = 2, ...fetchOptions } = options
  for (let attempt = 0; attempt <= retries; attempt++) {
    const response = await fetch(url, fetchOptions)
    if (response.status === 429) {
      await sleep(parseInt(response.headers.get('Retry-After') ?? '60') * 1000); continue
    }
    if (!response.ok) {
      const body = await response.json().catch(() => null)
      throw new ApiError(`API error ${response.status}`, response.status, body)
    }
    return response.json() as Promise<T>
  }
  throw new Error('Max retries exceeded')
}
function sleep(ms: number) { return new Promise(r => setTimeout(r, ms)) }
```

---

## Python Async Client

```python
# pip install httpx; requirements.txt: httpx>=0.27
import httpx, os
from typing import Any

API_KEY = os.environ.get("[SERVICE]_API_KEY")
if not API_KEY: raise ValueError("[Service] API key not configured")

class ApiError(Exception):
    def __init__(self, message: str, status: int, body: Any = None):
        super().__init__(message); self.status = status; self.body = body

async def api_fetch(path: str, method: str = "GET", **kwargs) -> dict:
    async with httpx.AsyncClient() as client:
        r = await client.request(method, f"https://api.example.com{path}",
            headers={"Authorization": f"Bearer {API_KEY}"}, **kwargs)
        if r.status_code == 429: raise ApiError("Rate limited", 429, {"retry_after": r.headers.get("Retry-After", 60)})
        if not r.is_success: raise ApiError(f"API error {r.status_code}", r.status_code, r.json())
        return r.json()
```

---

## Python Sync Client

```python
# pip install requests
import requests, os
SESSION = requests.Session()
SESSION.headers.update({"Authorization": f"Bearer {os.environ['[SERVICE]_API_KEY']}", "Content-Type": "application/json"})

def api_fetch(path: str, method: str = "GET", **kwargs) -> dict:
    r = SESSION.request(method, f"https://api.example.com{path}", **kwargs)
    r.raise_for_status()
    return r.json()
```

---

## Pagination

**Offset/Limit:**
```typescript
async function fetchAllPages<T>(url: string, headers: HeadersInit, pageSize = 100): Promise<T[]> {
  const results: T[] = []; let offset = 0
  while (true) {
    const data = await apiFetch<{ items: T[]; total: number }>(`${url}?limit=${pageSize}&offset=${offset}`, { headers })
    results.push(...data.items); offset += pageSize
    if (offset >= data.total) break
  }
  return results
}
```

**Cursor-based (GitHub/Stripe):**
```typescript
async function fetchAllCursor<T>(url: string, headers: HeadersInit): Promise<T[]> {
  const results: T[] = []; let cursor: string | null = null
  while (true) {
    const data = await apiFetch<{ data: T[]; has_more: boolean; next_cursor?: string }>(
      cursor ? `${url}?after=${cursor}` : url, { headers })
    results.push(...data.data)
    if (!data.has_more || !data.next_cursor) break
    cursor = data.next_cursor
  }
  return results
}
```

---

## GraphQL

```typescript
export async function gqlFetch<T>(endpoint: string, query: string, variables = {}, headers: HeadersInit = {}): Promise<T> {
  const response = await apiFetch<{ data: T; errors?: Array<{ message: string }> }>(endpoint, {
    method: 'POST', headers: { 'Content-Type': 'application/json', ...headers },
    body: JSON.stringify({ query, variables }),
  })
  if (response.errors?.length) throw new Error(`GraphQL: ${response.errors.map(e => e.message).join(', ')}`)
  return response.data
}
```

---

## Webhooks: TypeScript

```typescript
// app/api/webhooks/[service]/route.ts
export async function POST(request: Request) {
  const body = await request.text()  // raw body for signature verification
  const signature = (await headers()).get('[service]-signature') ?? ''
  if (!verifyWebhookSignature(body, signature, process.env.[SERVICE]_WEBHOOK_SECRET!))
    return new Response('Invalid signature', { status: 401 })
  const event = JSON.parse(body)
  switch (event.type) {
    case '[event.name]': await handle[EventName](event.data); break
    // ignore unknown types — don't error (prevents retries)
  }
  return new Response('OK', { status: 200 })
}
```

---

## Webhooks: Python

```python
@app.route("/webhooks/[service]", methods=["POST"])
def handle_webhook():
    body = request.get_data()
    signature = request.headers.get("[Service]-Signature", "")
    expected = "sha256=" + hmac.new(os.environ["[SERVICE]_WEBHOOK_SECRET"].encode(), body, hashlib.sha256).hexdigest()
    if not hmac.compare_digest(expected, signature): return jsonify({"error": "Invalid signature"}), 401
    event = request.get_json()
    if event.get("type") == "[event.name]": handle_[event_name](event["data"])
    return jsonify({"received": True}), 200
```

---

## Custom Pagination

If API uses custom pagination not matching offset/limit or cursor patterns (keyset cursors with link headers, X-Next-Token, etc.) → adapt the nearest pattern and add a comment:
```typescript
// TODO: Adapt pagination to [API]'s actual format — see API docs for [pagination_method]
// [CUSTOM_PAGINATION_SPEC_HERE]
```

---

## Environment Variables

```
# .env.local (never commit)
[SERVICE]_API_KEY=
[SERVICE]_WEBHOOK_SECRET=
[SERVICE]_BASE_URL=https://api.example.com

# .env.example (commit this)
[SERVICE]_API_KEY=your_key_here
[SERVICE]_WEBHOOK_SECRET=your_secret_here
```

Rules: never prefix with `NEXT_PUBLIC_`, null-check at client init (not at call site), never hardcode in source.

---

## Output Format

```
API INTEGRATION: [Service Name]
================================
Files to create:
1. lib/[service]-client.ts — typed client + auth
2. types/[service].ts — response types

Env vars to add: [SERVICE]_API_KEY, [SERVICE]_WEBHOOK_SECRET (if webhooks)

[Generated file contents]

Checklist:
[ ] API key in .env.local + .env.example updated
[ ] Error handling covers 401/403/404/429/5xx
[ ] No API keys in client-side code
```

After integration, write to progress.txt:
```
API_INTEGRATION:
  service: [name]  type: [REST|GraphQL]  auth: [type]
  files_created: [list]  env_vars: [list]
```
