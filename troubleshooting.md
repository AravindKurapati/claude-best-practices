# Troubleshooting — Symptom → Fix

When Claude Code "isn't doing the thing." Find your symptom, jump to the fix. Each entry links to the deeper file when you need the why.

---

## CLAUDE.md

### Claude ignores a rule in CLAUDE.md

- **Most likely**: CLAUDE.md is too long. Past ~2K tokens the model starts losing the middle. → Split into sub-files (`platform-docs.md`, `state.md`, `architecture.md`) and lazy-load. See `claude-md-best-practices.md`.
- **Then**: the rule is phrased as a preference, not an instruction. "Prefer X" gets ignored faster than "Always X. Never Y."
- **If the rule MUST hold 100% of the time**: it doesn't belong in CLAUDE.md. Move it to a hook. See `hooks/README.md` → "Hooks vs CLAUDE.md vs Skills vs Permissions."

### CLAUDE.md is bloating / performance is degrading

- → `claude-md-best-practices.md` → Keep It Short + Rewrite Periodically.
- Symptom check: run `/cost` mid-session. If the system prompt is >5K tokens, you're paying for it on every turn.

### Claude makes wrong assumptions about my product/domain

- CLAUDE.md is for *how*, not *what*. Move the product/domain facts to `platform-docs.md` + `styleguide.md` in `/documents/`. See `project-docs-structure.md`.

---

## Skills

### Skill exists but never triggers

- **Most likely**: the `description` in the YAML frontmatter doesn't include the verbs the user is actually typing. Descriptions are matched against user intent. → Rewrite to include 3-5 trigger phrases ("Use when user says X, Y, or Z").
- **Then**: another skill has a more compelling description and is winning. List skills (`/skills`) and check overlap.
- **Then**: the skill file is malformed YAML. The frontmatter must start at line 1, no BOM, three dashes exactly.

### Skill triggers when it shouldn't

- The description is too broad. Add an explicit `Do NOT use for: …` clause — Claude reads negative triggers. See `skills-official.md` for the official trigger-description format.

### Skill is huge, has too many sub-rules

- Split it. Multi-file skill with a short `SKILL.md` that loads the sub-files only when relevant sections fire. → `skills-architecture.md` (wikilinks, MOCs, progressive disclosure).

### I want a skill from project documentation

- Use the `skill-creator` skill. → `skills-authoring.md` → Building From Project Docs.

---

## Hooks

### Hook isn't firing

In order of likelihood:

1. The script isn't executable. `chmod +x .claude/hooks/*.sh`. On Windows, invoke `.py`/`.js` through their interpreter in the `command` field.
2. The matcher doesn't match. Run `/hooks` to see what Claude Code thinks your config is.
3. The hook lives in a settings file outside scope. Project hooks need `.claude/settings.json` in the project root, not nested.
4. `disableAllHooks: true` is set somewhere up the precedence chain. Check `~/.claude/settings.json` and `.claude/settings.local.json`.

### Hook fires but doesn't block

- **You used the wrong decision shape.** `PreToolUse` requires `hookSpecificOutput.permissionDecision: "deny"`. Other events use top-level `decision: "block"`. → `hooks/contract.md`.
- You exited `2` AND wrote JSON. The JSON gets ignored. Pick one mode per hook.

### `Stop` hook causes an infinite loop

- The hook is being re-invoked. Check for `stop_hook_active: true` in input and exit 0 immediately if true. → `hooks/patterns.md` → Pattern 4.

### Hook prints debug and pollutes Claude's context

- On `SessionStart` / `UserPromptSubmit`, anything on **stdout** is injected into context. Send debug to **stderr**.

---

## MCP

### MCP server doesn't appear in `/mcp`

- Run `claude mcp list` — it shows load errors per server. Usually a typo in `command` or missing env var.
- For stdio servers: run the `command` manually in a shell. If it crashes there, it'll crash for Claude Code too.

### Server connects but tools are never called

- The tool descriptions are weak. Claude picks tools by description; vague descriptions lose. → `mcp-patterns.md` → §5 Tool Design.
- Another server exposes a more compelling tool with the same effect. Disable the one you don't want, or namespace the tools.

### Context window blows up after adding a server

- Forgot `lazyLoad: true`. → `mcp-patterns.md` → §3 Lazy Loading.

### Hosted (OAuth) server keeps asking me to re-auth

- Tokens expired. `/mcp` → select server → "Re-authenticate." Some servers issue short-lived tokens; this is normal.

### Server worked yesterday, now it 500s

- Subprocess in a bad state. `claude mcp restart <name>`, or just restart Claude Code (kills + respawns all stdio servers).

---

## Agents & Subagents

### Subagent runs forever / loops

- Set tighter limits in the spawn call (or rely on the 50-turn default). Inspect the subagent's transcript via the returned task ID before re-running.

### Subagent returns useless output

- Underspec'd prompt. Subagents start cold — they don't see the parent conversation. The prompt must include: goal, what's been tried, what file paths to look at, what form the answer should take. → `agentic-workflow-best-practices.md` → Parallel Agents section.

### Parallel subagents stomp on each other

- They share the same working directory. Use git worktrees (one per agent) or `isolation: "worktree"` on the spawn call. → `agentic-workflow-best-practices.md`.

---

## Context & Compaction

### Claude forgets state across compaction

- The compaction summary throws away mid-conversation details. → Use a `SessionStart` hook with the `compact` matcher to re-inject `state.md`. → `hooks/patterns.md` → Pattern 3.
- Pre-emptively dump important context with a `PreCompact` hook. → `hooks/patterns.md` → Pattern 10.

### Context is full and Claude is slow

- Run `/cost` and `/compact`. If `/compact` keeps happening every few turns, the underlying issue is usually a giant CLAUDE.md, an MCP server without `lazyLoad`, or a tool that returns mega-blobs (raw HTML, full file contents).

---

## Permissions & Auto Mode

### Claude keeps stopping for permission on a safe command

- Promote the command to your allowlist. The `/fewer-permission-prompts` skill scans your transcript for repeated approvals and proposes a prioritized allowlist for `.claude/settings.json`.
- For a single project: add to `.claude/settings.json` `permissions.allow`. For all projects: `~/.claude/settings.json`.

### Auto mode auto-approved something it shouldn't have

- Add the pattern to `permissions.deny` (hard rule) or a `PreToolUse` hook (context-sensitive). Deny always wins, even in `bypassPermissions`.

---

## Search / Web

### Claude uses built-in search instead of Exa

- CLAUDE.md policy needs to be explicit and at the top of the file: "For all search and research, use Exa MCP. Do not use built-in web search." Repeat in the project CLAUDE.md too — global policies get diluted in long contexts.

---

## When all else fails

1. `claude --version` — are you on the version the docs assume?
2. `claude mcp list` and `/hooks` — your config might not be what you think it is.
3. Look at `.claude/logs/` if you have the audit-log hook installed.
4. Start a fresh session — stale context masks fixed bugs.
5. If a third-party skill or plugin is in the mix, disable it and re-test. Plugin skills update on their own schedule.
