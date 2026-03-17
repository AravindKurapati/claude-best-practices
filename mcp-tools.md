# MCP Tools — Servers, Setup, and When to Use Them

## Skills vs MCP — The Core Distinction
- **MCP** = connectivity. Secure, standardized access to external systems
- **Skills** = expertise. Domain knowledge and workflow logic on top of those connections
- MCP without skills: Claude has access but lacks procedural clarity → inconsistent output
- Skills without MCP: Claude has instructions but no live data or tools
- One skill can orchestrate multiple MCP servers. One MCP server can support dozens of skills.

Analogy: MCP is the kitchen (tools, ingredients). Skills are the recipes (what to do with them).

MCP is client-agnostic — configure a server once, it works in Claude Code, Cursor, Windsurf, VS Code, all of them.

---

## How to Pick Your MCP Stack
Ask yourself three questions before adding anything:
- What am I copy-pasting between Claude and other tools?
- What tabs do I keep switching between?
- Where do I lose 10 minutes every time?

Every MCP you keep should kill a specific friction point. Start with 1-2, get comfortable, then expand.

---

## The Config File
All MCP servers go in one place. In Claude Code: Settings → Developer → Edit Config. It opens a JSON file:

```json
{
  "mcpServers": {
    "exa": {
      "command": "npx",
      "args": ["-y", "exa-mcp-server"],
      "env": { "EXA_API_KEY": "your-key-here" }
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_PERSONAL_ACCESS_TOKEN": "your-token-here" }
    }
  }
}
```

Add each server as a new block inside `mcpServers`. Restart Claude Code after any changes.

---

## Recommended Servers by Category

### 🔍 Search & Research
**Exa MCP** ← install this first
- Better quality and faster than Claude's built-in search
- Has its own index with date filters for fresh results (not stale training data)
- Key tools: `web_search_exa`, `get_code_context_exa` (real code from GitHub/SO/docs), `company_research`, `crawling`, `deep_researcher`
- Get API key: `dashboard.exa.ai/api-keys` (free tier available)
- Add to global CLAUDE.md: "For all search and research, use Exa MCP. Do not use built-in web search."

**Context7**
- Injects up-to-date, version-specific docs and real code examples into context
- Kills hallucinated APIs — Claude uses accurate library info instead of stale training data
- Essential when working with any rapidly evolving framework or SDK

```json
"context7": {
  "command": "npx",
  "args": ["-y", "@upstash/context7-mcp@latest"]
}
```

### 💻 Code & Version Control
**GitHub MCP** ← most universally useful after Exa
- Claude gets direct access to your repos
- Read issues, review PRs, search across repos, automate workflows — without leaving terminal
- Supports read-only lockdown mode to prevent accidental changes to production

**Sentry MCP**
- Go from production alert to fix without switching tools
- Fetches full stack traces and breadcrumbs directly into context
- Remote hosted at `https://mcp.sentry.dev/mcp` (OAuth auth)

### 🗄️ Databases
**PostgreSQL MCP**
- Natural language queries — no switching to a SQL client
- Claude explores table structure, understands relationships, runs queries
- Use read-only mode for anything near production

**Supabase MCP**
- Combines Postgres, auth, and storage in one
- Best for full-stack developers building modern backend apps

### 🧠 Memory
**Memory MCP** ← most underrated category
- Claude goes from "tool I explain things to every session" → "collaborator who knows my projects"
- Changes the whole relationship — context carries over sessions
- Makes every other MCP more powerful

### 🎨 Design
**Figma MCP**
- Extract design specs, access component libraries, sync design tokens
- Ask: "convert this Figma layout to React components" → production-ready code
- Install: `npx @composio/mcp@latest setup figma --client claude`

### 📊 Product & Analytics
**PostHog MCP**
- Access feature flags, user funnels, A/B test results inside Claude Code
- Make code decisions based on actual user behavior, not assumptions

### 🔧 Project Management
**Atlassian MCP (Jira + Confluence)**
- Search tickets, update statuses, create tickets from PR descriptions or bug reports
- Pull Confluence docs alongside your code — no more IDE/Jira context switching

### ⚡ Automation
**Zapier MCP**
- Connect Claude to any app Zapier supports (Slack, Gmail, Trello, etc.)
- Trigger cross-app workflows from inside Claude Code

---

## Specialist Agents (not MCP, but related)
**Agency Agents** — 61 specialized sub-agents
- Engineering, design, marketing, product, testing, and more
- Each has unique personality, workflows, real deliverables
- Install: drop into `~/.claude/agents/`
- Repo: `github.com/msitarzewski/agency-agents`
- Works with Claude Code, Cursor, Windsurf, Aider, Gemini CLI

---

## Claude Code Plugins Worth Installing
- `mksglu/claude-context-mode` — bigger context window via reference loading
- `claude-warden` — protection against destructive actions
- `arscontexta` — builds skill graphs (249-file knowledge system)

---

## Multi-LLM Code Review Pattern
- Only ~1/3 of issues overlap across different LLMs — use multiple for broader coverage
- Connect Gemini + Codex as local review tools alongside Claude
- Push to GitHub early → Cursor Bug Bot reviews
- Fresh Claude context = better review (no bias toward its own code)
