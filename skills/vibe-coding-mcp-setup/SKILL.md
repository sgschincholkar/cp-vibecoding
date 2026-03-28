---
name: vibe-coding-mcp-setup
description: |
  MCP (Model Context Protocol) server discovery, configuration, and usage guide.
  Helps Claude connect to external data sources during development — databases, GitHub, Slack, filesystem, browsers, and more.
  Use when: (1) User says "add MCP", "set up MCP server", "connect Claude to [database/GitHub/Slack]",
  (2) User says "I want Claude to query my database directly", "Claude should read my files", "connect to GitHub",
  (3) BUILD or DEBUG needs direct data access (query DB, read repo, check logs),
  (4) User asks "what MCPs are available", "which MCP should I use for [task]",
  (5) User says "configure MCP", "add MCP server to settings".
  This is a META skill — it configures Claude's own tools, not the app's code.
  Outputs configuration blocks for .claude/settings.json or claude_desktop_config.json.
---

# Vibe Coding — MCP Setup

Configure MCP servers so Claude can directly access external data sources during development.

## Entry Router

```
User asks which MCP to use?
→ READ: ## MCP Selection Guide

User wants to add a specific MCP?
→ Find config in ## MCP Catalog → READ: ## Setup Process

User wants a custom MCP server?
→ READ: ## Custom MCP
```

**Jump directly to the section matched above. Do not read ## MCP Catalog before you know which server the user wants — use ## MCP Selection Guide first if unsure.**

---

## Config Locations

**Claude Code** (CLI/IDE): `.claude/settings.json` in project root or `~/.claude/settings.json` globally.

**Claude Desktop**: `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) or `%APPDATA%\Claude\claude_desktop_config.json` (Windows).

```json
{
  "mcpServers": {
    "[server-name]": {
      "command": "[command]",
      "args": ["[arg1]"],
      "env": { "[ENV_VAR]": "[value]" }
    }
  }
}
```

---

## MCP Selection Guide

| Need | MCP |
|------|-----|
| Query dev database | `postgres` or `sqlite` |
| Read/write project files | `filesystem` |
| Read GitHub issues/PRs | `github` |
| Search Slack messages | `slack` |
| Test UI in browser | `playwright` |
| Hit live API endpoints | `fetch` |
| Remember things between sessions | `memory` |
| Complex multi-step planning | `sequential-thinking` |

---

## MCP Catalog

**PostgreSQL:**
```json
"postgres": { "command": "npx", "args": ["-y", "@modelcontextprotocol/server-postgres", "postgresql://[user]:[password]@[host]:[port]/[database]"] }
```
Use read-only DB user. Never use production credentials.

**SQLite:**
```json
"sqlite": { "command": "npx", "args": ["-y", "@modelcontextprotocol/server-sqlite", "--db-path", "./[database].db"] }
```

**Filesystem:**
```json
"filesystem": { "command": "npx", "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/project"] }
```
Only expose directories Claude needs — not home directory.

**GitHub:**
```json
"github": { "command": "npx", "args": ["-y", "@modelcontextprotocol/server-github"], "env": { "GITHUB_PERSONAL_ACCESS_TOKEN": "[token]" } }
```
Token scopes: `repo`, `read:org`, `issues`, `pull_requests`. Create at GitHub Settings → Developer Settings → Fine-grained tokens.

**Slack:**
```json
"slack": { "command": "npx", "args": ["-y", "@modelcontextprotocol/server-slack"], "env": { "SLACK_BOT_TOKEN": "xoxb-[token]", "SLACK_TEAM_ID": "T[id]" } }
```
Create Slack app at api.slack.com → Add Bot Token Scopes → Install to workspace.

**Playwright (browser automation):**
```json
"playwright": { "command": "npx", "args": ["-y", "@playwright/mcp"] }
```

**Browserbase (cloud browsers):**
```json
"browserbase": { "command": "npx", "args": ["-y", "@browserbasehq/mcp-server-browserbase"], "env": { "BROWSERBASE_API_KEY": "[key]", "BROWSERBASE_PROJECT_ID": "[id]" } }
```

**Fetch/HTTP:**
```json
"fetch": { "command": "npx", "args": ["-y", "@modelcontextprotocol/server-fetch"] }
```

**Memory:**
```json
"memory": { "command": "npx", "args": ["-y", "@modelcontextprotocol/server-memory"] }
```

**Sequential Thinking:**
```json
"sequential-thinking": { "command": "npx", "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"] }
```

---

## Setup Process

```
1. Identify which server (from catalog above)
2. Check .claude/settings.json — if file doesn't exist, create it with template: { "mcpServers": {} }
   Read existing config if present — do NOT overwrite existing servers
3. Guide user to get required credentials
4. Generate config block
5. Merge under "mcpServers" alongside existing servers
6. Run security checklist below
7. Tell user to restart Claude and verify with test query
```

**Verification queries:**
- Postgres: "list all tables in the database"
- GitHub: "list open issues in [repo]"
- Filesystem: "list files in [directory]"
- Fetch: "fetch https://[your-api]/health"

---

## Security Checklist

```
[ ] Credentials in env vars — NOT hardcoded in settings.json
[ ] Database MCP uses read-only user
[ ] Filesystem MCP scoped to project directory only
[ ] GitHub token uses fine-grained permissions
[ ] .claude/settings.json in .gitignore if it contains credentials
[ ] Dev/staging database only — never production
```

---

## Custom MCP

```typescript
// Minimal MCP server (Node.js)
import { Server } from '@modelcontextprotocol/sdk/server/index.js'
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js'
import { ListToolsRequestSchema, CallToolRequestSchema } from '@modelcontextprotocol/sdk/types.js'

const server = new Server({ name: '[your-server]', version: '1.0.0' }, { capabilities: { tools: {} } })

server.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [{ name: '[tool]', description: '[what it does]',
    inputSchema: { type: 'object', properties: { [param]: { type: 'string' } }, required: ['[param]'] } }]
}))

server.setRequestHandler(CallToolRequestSchema, async (request) => {
  if (request.params.name === '[tool]') {
    const result = await yourFunction(request.params.arguments)
    return { content: [{ type: 'text', text: JSON.stringify(result) }] }
  }
  throw new Error(`Unknown tool: ${request.params.name}`)
})

await server.connect(new StdioServerTransport())
```

Register in settings:
```json
"[your-server]": { "command": "node", "args": ["./mcp-servers/[your-server].js"] }
```

---

## Output Format

```
MCP SETUP: [Server Name]
=========================
What this enables: [1-2 sentences]

Required credentials: [CREDENTIAL]: Get from [URL/instructions]

Config to add to .claude/settings.json:
[json block]

Verify: Ask Claude: "[test query]"
Security notes: [any specific warnings]
```
