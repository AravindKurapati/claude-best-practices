# The JSON Contract & Settings Precedence

---

## Input — common fields

Every event sends at least:

```json
{
  "session_id": "abc123",
  "transcript_path": "/path/to/transcript.jsonl",
  "hook_event_name": "PreToolUse"
}
```

Plus event-specific fields:

| Event | Adds |
|---|---|
| `PreToolUse`, `PostToolUse`, `PermissionRequest` | `tool_name`, `tool_input` |
| `PostToolUse`, `PostToolUseFailure` | `tool_response` |
| `UserPromptSubmit` | `prompt` |
| `SessionStart` | `source` (one of `startup`, `resume`, `clear`, `compact`) |
| `Notification` | `message`, `type` |
| `Stop`, `SubagentStop` | `stop_hook_active` (set if the hook is being re-invoked — guard against loops) |

---

## Output — exit codes

| Code | Meaning |
|---|---|
| `0` | Allow. **For `UserPromptSubmit`, `UserPromptExpansion`, `SessionStart`: anything you write to stdout is added to Claude's context.** |
| `2` | Block. stderr is fed back to Claude as an error message. JSON in stdout is **ignored**. |
| any other | Error notice shown to user; tool proceeds. |

## Output — JSON (exit 0 with structured output)

Universal fields:

```json
{
  "continue": true,                  // false stops the entire turn; takes precedence over event-specific fields
  "stopReason": "shown to user when continue=false",
  "suppressOutput": false,           // hide stdout from debug log
  "systemMessage": "warning shown to user",
  "hookSpecificOutput": { "hookEventName": "PreToolUse", "...": "..." }
}
```

---

## The schema fragmentation gotcha

**Different events use different decision shapes.** This is documented inconsistently in Anthropic's own docs ([issue #19115](https://github.com/anthropics/claude-code/issues/19115)) — copying a `PostToolUse` example into a `PreToolUse` hook will silently not block.

| Event | How to block | Where the reason goes |
|---|---|---|
| `PreToolUse` | `hookSpecificOutput.permissionDecision: "deny"` | `hookSpecificOutput.permissionDecisionReason` |
| `PostToolUse`, `Stop`, `SubagentStop`, `UserPromptSubmit` | top-level `decision: "block"` | top-level `reason` |
| `PermissionRequest` | `hookSpecificOutput.decision.behavior: "deny"` (or `"allow"`, `"ask"`) | `permissionDecisionReason` |
| `PermissionDenied` | top-level `retry: true` to let Claude try again | n/a |

If you only remember one rule: **`PreToolUse` is the odd one out.** It uses the nested shape; the rest still use top-level `decision`. Anthropic is migrating everything to `hookSpecificOutput` but as of April 2026 the migration is partial.

### `PreToolUse` — the full shape

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "deny",            // or "allow" | "ask" | "defer" (TS SDK only)
    "permissionDecisionReason": "Blocked: rm -rf in repo root",
    "updatedInput": { "command": "rm -rf ./build" }   // optional input rewrite; requires permissionDecision: "allow"
  }
}
```

`updatedInput` is the underrated power move — you don't have to block, you can rewrite. Strip an `--unsafe` flag, append `--dry-run`, redirect a path. **Always return a new object; do not mutate `tool_input` in place.**

---

## Settings Files & Precedence

Hooks live in JSON settings files. There are four scopes; precedence runs **strict to lax**:

```
managed enterprise settings  (highest — overrides everything)
        |
        v
project    .claude/settings.json          (committed, team-shared)
        |
        v
project    .claude/settings.local.json    (gitignored, your machine only)
        |
        v
user       ~/.claude/settings.json        (lowest — global defaults)
```

You can also split hooks into a dedicated `.claude/hooks.json` file with a `description` field — useful when a single project has many hooks and a settings.json is getting unwieldy.

`disableAllHooks: true` in any settings file kills every hook in scope. Use it as a panic switch when debugging.

`/hooks` inside Claude Code lists every configured hook grouped by event — your single best debugging tool.

---

Next: copy-pasteable [patterns](./patterns.md), or check [pitfalls](./pitfalls.md) if something isn't working.
