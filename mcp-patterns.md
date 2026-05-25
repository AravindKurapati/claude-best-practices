# MCP — Deep Dive

`mcp-tools.md` is the catalog ("what servers exist, when to install them"). This file is the mechanics: how servers actually run, why your context bloats and how to fix it, when to build a server vs. write a script, and how to debug the failures the docs don't cover.

> Rule of thumb: every MCP server you keep should kill a *specific* friction point and earn its context cost. Default to fewer servers, lazy-loaded.

---

## 1. The Mental Model

An MCP server is a **separate process** that exposes tools, resources, and/or prompts over a JSON-RPC protocol. Claude Code (the client) launches and talks to it.

```
Claude Code  <-- JSON-RPC (stdio or HTTP) -->  MCP server process
                                                |
                                                v
                                     external system (API, DB, file, etc.)
```

Three things to internalize:

- **Servers run on your machine** (or a remote host you point to). Claude Code spawns them; the API never sees them.
- **Tool schemas count against context.** Every server you connect loads its tool list into the system prompt — even tools you never call. Lazy loading (§3) is the fix.
- **Connection ≠ capability.** A connected server only matters when Claude calls one of its tools. Permissions, naming, and trigger descriptions decide whether that happens.

---

## 2. Transport Modes

| Mode | When to use | Trade-offs |
|---|---|---|
| `stdio` (subprocess) | Default. Local servers (Postgres, GitHub, Exa, custom scripts). | Restarts when Claude Code restarts; no shared state across clients. |
| `http` (remote) | Hosted servers (Sentry MCP, Linear, vendor-provided). OAuth-protected services. | Adds network latency to every tool call; survives client restarts. |
| `sse` (deprecated) | Old hosted servers transitioning to streamable HTTP. | Avoid in new configs. |

The 2026 default for vendor-hosted MCP is **streamable HTTP** with OAuth. Old `sse` configs still work but the spec is moving on.

---

## 3. Lazy Loading — The Single Biggest Context Win

Default behavior (pre-2026): every connected server's full tool catalog is loaded into context at session start. Five servers with ~10 tools each = ~50 tool schemas you may never use, eating thousands of tokens.

The 2026 `lazyLoad` flag flips this: tool schemas load **only when Claude decides to query the server**.

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_PERSONAL_ACCESS_TOKEN": "..." },
      "lazyLoad": true
    }
  }
}
```

Pair it with a **good `description`** field — Claude reads descriptions to decide whether to load the tools.

```json
{
  "mcpServers": {
    "sentry": {
      "url": "https://mcp.sentry.dev/mcp",
      "lazyLoad": true,
      "description": "Production error tracking. Use for stack traces, breadcrumbs, alert investigation."
    }
  }
}
```

Without a description, lazy loading defeats itself — Claude doesn't know what's behind the door, so it conservatively loads everything anyway.

The new `claude mcp add` CLI sets `lazyLoad: true` by default:

```bash
claude mcp add github --transport stdio -- npx -y @modelcontextprotocol/server-github
```

---

## 4. Build vs. Install vs. Skip

Before installing a new server, work down this ladder:

| Need | Reach for |
|---|---|
| Hit a public API a couple of times per session | Just let Claude write `curl` / `fetch` calls. Don't add a server. |
| Hit a private/auth'd service repeatedly across sessions | Install or write an MCP server. |
| One-off custom workflow (your build, your repo) | Skill + bash, not an MCP server. Skills are cheaper to maintain. |
| Domain expertise + multi-step procedure | Skill. (Skills can *call* MCP servers; they're not alternatives.) |
| Connectivity to a system that already has a community server | Install it. Don't rewrite. |

**Custom servers are surprisingly easy** (the SDKs are thin), but every server you write is a process you have to keep alive, version, and debug. Default to skill + script unless you'll reach for it weekly.

---

## 5. Tool Design (if you're building a server)

The schemas the model sees are part of your UX. Patterns that work:

- **Names like English.** `search_issues` reads better than `gh_issues_query`. Claude pattern-matches on name + description more than schema.
- **One tool, one job.** Don't bundle "search, comment, close" into a single tool with a `mode` enum. Three tools with three descriptions reads better.
- **Descriptions answer "when would Claude pick this?"** Lead with the trigger condition, not the implementation. Bad: "Calls /api/v2/issues/search." Good: "Search issues by title, label, or assignee. Use for triage questions."
- **Return structured JSON, not prose.** Claude is better at parsing structure than re-parsing its own narration. Save the prose for when you're the one rendering output.
- **Errors are tools too.** A clear error message ("rate limited, retry in 30s" vs. "500") changes whether Claude retries blindly or backs off.
- **Tool count: keep it small.** Servers exposing >15 tools start crowding context. If you need more, split into multiple servers behind `lazyLoad`.

---

## 6. Auth & Secrets

| Setup | Where the secret lives |
|---|---|
| API key in env | `.claude/settings.local.json` (gitignored) or your shell profile. **Never** in `.claude/settings.json`. |
| OAuth (hosted MCP) | Tokens stored by Claude Code in its own credential store. Re-auth via `/mcp` -> server name. |
| Service account (Postgres, etc.) | DSN in env. Keep credentials read-only for anything near prod. |

A server config committed to git is a server config the next contributor inherits — but env values are *not* committed. The `env` block in `settings.json` should reference `${ENV_VAR}` shell substitutions, not literals.

```json
"github": {
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-github"],
  "env": { "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}" }
}
```

---

## 7. Debugging

| Symptom | First thing to check |
|---|---|
| Server doesn't show up in `/mcp` | Config typo or path issue. Run `claude mcp list` — it shows load errors per server. |
| Tool calls hang or time out | For stdio: the subprocess crashed. Run the `command` manually in a shell and watch stderr. For HTTP: server down or auth expired. |
| Claude has the server but never calls the tools | Bad/missing `description` on tools, or another server's tool has a more compelling name. Check `/mcp` -> tools. |
| Context window blowing up after adding a server | Forgot `lazyLoad: true`. Move it behind lazy loading; restart. |
| Auth loop on OAuth server | Re-run `/mcp` -> select server -> "Re-authenticate." Tokens expire silently. |
| Tool works once, fails after | Rate limit, expired token, or the server caches a bad response. Restart the server (it's a subprocess — `claude mcp restart <name>`). |

`/mcp` inside Claude Code is your `/hooks` equivalent — it shows every connected server, its status, its tool catalog, and last error.

---

## 8. Pitfalls

| Pitfall | What happens | Fix |
|---|---|---|
| Many servers, no `lazyLoad` | Massive system prompt bloat, slow first-token latency | Audit `claude mcp list`. Set `lazyLoad: true` on everything you don't use every session. |
| Auth tokens in `settings.json` | Leaked via `git log` forever | Reference env vars; keep literals in `settings.local.json` or shell profile. |
| Multiple servers exposing same tool name (e.g. two `search`) | Last-loaded wins; ambiguous behavior | Namespace tools when you build them (`<server>_<verb>`). Rename in client config if you can't change the server. |
| Trusting MCP for auditable actions | MCP runs locally — no central log of what Claude did | Wrap risky tools in a custom server that logs to your SIEM, or add a `PostToolUse` hook to JSONL-log MCP calls. |
| Heavy server in `stdio` mode shared across many clients | Each client spawns its own process; resource contention | Move to HTTP transport and run a single shared instance. |
| MCP server proxying a model API (e.g. "ask GPT-4") | You're now paying twice and the model in the loop has no context | Use Claude for what you need; don't proxy LLMs through MCP. |
| Letting Claude write to prod via MCP | One bad call and you've shipped | Read-only credentials by default. Promote to write only inside a specific skill that includes the safety procedure. |
| Untyped string blobs as tool inputs | Claude over-formats; you get JSON-in-strings | Use real JSON schemas. The SDK makes this easy and it dramatically improves call quality. |

---

## 9. MCP vs Hooks vs Skills vs Subagents

| Need | Where it goes |
|---|---|
| "Give Claude access to system X" | MCP server |
| "Force Claude to do Y before every Z" | Hook |
| "Encode a multi-step procedure Claude follows when asked" | Skill |
| "Run a long, isolated task with its own context" | Subagent (`Task` tool) |
| "Read live data from system X *inside* a procedure" | Skill that *calls* the MCP server |
| "Block a destructive MCP call" | `PreToolUse` hook matching the tool name |

MCP gives Claude **capabilities**. The other three decide **when and how** Claude uses them.

---

## 10. References

- MCP spec: [modelcontextprotocol.io](https://modelcontextprotocol.io)
- Server SDKs: [github.com/modelcontextprotocol](https://github.com/modelcontextprotocol)
- Anthropic's MCP overview: [docs.claude.com/en/docs/claude-code/mcp](https://docs.claude.com/en/docs/claude-code/mcp)
- Related in this KB: `mcp-tools.md` (server catalog), `claude-code-2026-update.md` → MCP Lazy Loading section, `hooks/patterns.md` (audit-log pattern works for MCP calls too)
