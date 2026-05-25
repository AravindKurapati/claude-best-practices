# Claude Code Hooks — Deep Dive

The hooks coverage in `agentic-workflow-best-practices.md` and `claude-code-2026-update.md` is correct as far as it goes, but it's a fraction of what hooks actually do. This directory is the reference: every event, every handler type, the JSON contract Claude Code actually expects, the schema gotchas the official docs are still inconsistent on, and patterns past "auto-format on save."

> If a rule must hold 100% of the time, it goes in a hook. CLAUDE.md is advisory (~80%). Hooks are deterministic.

---

## Read order

| File | When to open |
|---|---|
| [`events.md`](./events.md) | "What lifecycle points can I hook into?" — full event list with what each can/can't block |
| [`handlers.md`](./handlers.md) | "Should this be a shell command, a small-model prompt, a subagent, or an HTTP call?" |
| [`contract.md`](./contract.md) | "What JSON does my hook receive, and what shape does it need to return?" Includes the `PreToolUse` schema gotcha + settings-file precedence |
| [`patterns.md`](./patterns.md) | "Show me real hook implementations I can copy" — 10 patterns + a starter `.claude/settings.json` |
| [`pitfalls.md`](./pitfalls.md) | "It's not working" — the 10 traps that bite people repeatedly, plus reference links |

---

## Mental Model

Hooks are **shell commands, prompts, sub-agents, or HTTP endpoints** that Claude Code fires at lifecycle points. The flow is always:

```
event fires -> matcher matches -> handler receives JSON -> handler returns a decision
```

Three things to internalize:

- **Hooks run before permission checks.** A `PreToolUse` hook returning `deny` blocks the tool **even in `bypassPermissions` mode**. They are the sharpest tool you have.
- **Deny always wins.** When multiple hooks or settings rules apply: `deny > defer > ask > allow`. A hook returning `allow` does not override a deny rule from settings.
- **You pick exit codes OR JSON, not both.** If you exit `2`, Claude Code ignores any JSON you printed and treats stderr as the blocking message. JSON only works on exit `0`.

---

## Hooks vs CLAUDE.md vs Skills vs Permissions

| Need | Where it goes |
|---|---|
| "Claude should generally prefer X" | CLAUDE.md (advisory) |
| "Claude must always do X before Y" | Hook (deterministic) |
| "When asked to do X, follow this multi-step procedure" | Skill |
| "This file/command should never be touched, period" | Settings `permissions.deny` (cleaner than a hook for static rules) |
| "This file may be touched only after a human approves" | `PreToolUse` hook returning `permissionDecision: "ask"` |
| "On every edit, run the formatter" | `PostToolUse` hook |
| "Before declaring done, all tests must pass" | `Stop` agent hook |
| "Inject `state.md` into every session, including post-compaction" | `SessionStart` hook with `compact` matcher |

A working setup uses **all four**: CLAUDE.md sets intent, Skills encode procedures, Permissions handle static blocklists, Hooks enforce dynamic invariants and feedback loops.
