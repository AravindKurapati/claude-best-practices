# Claude Code Hooks — Deep Dive

The hooks coverage in `agentic-workflow-best-practices.md` and `claude-code-2026-update.md` is correct as far as it goes, but it's a fraction of what hooks actually do. This file is the reference: every event, every handler type, the JSON contract Claude Code actually expects, the schema gotchas the official docs are still inconsistent on, and patterns past "auto-format on save."

> If a rule must hold 100% of the time, it goes in a hook. CLAUDE.md is advisory (~80%). Hooks are deterministic.

---

## 1. The Mental Model

Hooks are **shell commands, prompts, sub-agents, or HTTP endpoints** that Claude Code fires at lifecycle points. The flow is always:

```
event fires -> matcher matches -> handler receives JSON -> handler returns a decision
```

Three things to internalize:

- **Hooks run before permission checks.** A `PreToolUse` hook returning `deny` blocks the tool **even in `bypassPermissions` mode**. They are the sharpest tool you have.
- **Deny always wins.** When multiple hooks or settings rules apply: `deny > defer > ask > allow`. A hook returning `allow` does not override a deny rule from settings.
- **You pick exit codes OR JSON, not both.** If you exit `2`, Claude Code ignores any JSON you printed and treats stderr as the blocking message. JSON only works on exit `0`.

---

## 2. Every Event (current as of April 2026)

Anthropic groups events into three cadences. The table below is the full list — far more than the 4 events in `claude-code-2026-update.md`.

### Once per session
| Event | Fires | Can block? | Common use |
|---|---|---|---|
| `SessionStart` | Session begins or resumes (matcher: `startup`, `resume`, `clear`, `compact`) | No (but stdout is injected into context) | Inject project state, load `state.md`, post-compaction context recovery |
| `SessionEnd` | Session terminates | No | Persist last-session summary, flush logs |
| `Setup` | One-shot init: `claude --init-only`, or `--init`/`--maintenance` in `-p` mode | No | CI/script preparation that should run before the first prompt |
| `PreCompact` | Before the conversation is compacted | No | Save important context to disk before it's summarized away |

### Once per turn
| Event | Fires | Can block? | Common use |
|---|---|---|---|
| `UserPromptSubmit` | User submits a prompt, before Claude sees it | Yes | Inject context, redact secrets, reject prompts that violate policy |
| `UserPromptExpansion` | A typed slash command expands into a prompt | Yes | Block dangerous slash commands; rewrite expansions |
| `Stop` | Main agent finishes a turn | Yes (force Claude to keep going) | Run tests / typecheck before declaring done |
| `StopFailure` | Turn ends in error | No | Alert / log |

### Per tool call (inside the agentic loop)
| Event | Fires | Can block? | Common use |
|---|---|---|---|
| `PreToolUse` | Before a tool runs | Yes | Block `rm -rf`, gate `git push --force`, rewrite tool input |
| `PostToolUse` | After tool succeeds | Yes (re-prompts Claude) | Lint/format, log, append context |
| `PostToolUseFailure` | After tool errors | Yes | Auto-retry policy, structured error feedback |
| `PostToolBatch` | After a parallel batch of tool calls fully resolves, before next model turn | Yes | Inject conventions once per batch instead of per call |
| `PermissionRequest` | A permission dialog would appear | Yes | Custom approval logic; auto-approve a known-safe pattern |
| `PermissionDenied` | Auto-mode classifier denied the call | No (return `{retry:true}`) | Tell Claude it may retry with a different approach |

### Subagents and tasks
| Event | Fires | Can block? |
|---|---|---|
| `SubagentStart` | A subagent is spawned | No |
| `SubagentStop` | A subagent finishes | Yes (force it to keep going) |
| `TaskCreated` | A task is created via `TaskCreate` | No |
| `TaskCompleted` | A task is marked completed | No |

### Notifications
| Event | Matcher value | Common use |
|---|---|---|
| `Notification` | `permission_prompt`, `idle_prompt`, `auth_success`, `elicitation_dialog`, `elicitation_complete`, `elicitation_response` | Desktop alerts, Slack pings when Claude is idle, OAuth-success logging |

The minimum useful subset for a serious project is: `PreToolUse`, `PostToolUse`, `UserPromptSubmit`, `SessionStart`, `Stop`, `Notification`.

---

## 3. The Four Handler Types

`type` in your hook config picks one. The first three are real, the fourth is undocumented in most secondary sources.

### `type: "command"` — shell command
Most common. Fast, predictable, no model spend. Receives JSON on stdin, communicates via exit code + stdout/stderr.

```json
{ "type": "command", "command": ".claude/hooks/block-dangerous.sh" }
```

### `type: "prompt"` — small model judgment call
Sends a yes/no question to a Claude model (Haiku by default). Use when a regex isn't enough — e.g. "is this commit message a stub?" rather than "does this command match `rm -rf`?"

```json
{ "type": "prompt", "prompt": "Does this diff modify any test files? Answer yes or no.", "model": "claude-haiku-4-5-20251001" }
```

Cost: one tiny API call per fire. Don't use prompt hooks on `PostToolUse` for `Edit|Write` unless you genuinely need judgment — you'll burn tokens on every edit.

### `type: "agent"` — full subagent
Spawns a subagent that can read files, run commands, verify conditions. 60-second timeout, up to ~50 tool turns. Use for `Stop` hooks that need to actually verify work — e.g. "before declaring done, run the test suite and confirm it passes."

```json
{
  "type": "agent",
  "prompt": "Run `pytest tests/ -q`. If anything fails, return `{decision: 'block', reason: <failure summary>}`. Otherwise allow.",
  "tools": ["Bash", "Read"]
}
```

### `type: "http"` — POST to a URL
The same JSON body that a command hook sees on stdin is sent as a POST request body. The HTTP response body becomes the decision JSON. Useful when the policy lives in a service (auth check against your IAM, audit log to a SIEM, central rate limiter).

```json
{ "type": "http", "url": "https://hooks.internal.example.com/claude/pre-bash", "headers": { "Authorization": "Bearer ..." } }
```

Caveats: latency adds to every tool call; failures are treated as errors (Claude Code shows a hook-error notice and proceeds). Set tight timeouts on the server side.

---

## 4. The JSON Contract (and the gotcha)

### Input — common fields
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

### Output — exit codes
| Code | Meaning |
|---|---|
| `0` | Allow. **For `UserPromptSubmit`, `UserPromptExpansion`, `SessionStart`: anything you write to stdout is added to Claude's context.** |
| `2` | Block. stderr is fed back to Claude as an error message. JSON in stdout is **ignored**. |
| any other | Error notice shown to user; tool proceeds. |

### Output — JSON (exit 0 with structured output)
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

### The schema fragmentation gotcha
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

## 5. Settings Files & Precedence

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

## 6. Pattern Library

Patterns are ordered roughly by how often you'll reach for them.

### Pattern 1 — Auto-format on every edit (`PostToolUse`)
The example already in `agentic-workflow-best-practices.md` is fine but uses bash + jq. Cross-platform version that doesn't break on Windows (no `xargs`):

```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Write|Edit",
      "hooks": [{
        "type": "command",
        "command": "node .claude/hooks/format.js"
      }]
    }]
  }
}
```

```js
// .claude/hooks/format.js
const { execSync } = require("node:child_process");
let input = "";
process.stdin.on("data", c => input += c);
process.stdin.on("end", () => {
  const { tool_input } = JSON.parse(input);
  const file = tool_input?.file_path;
  if (!file) return process.exit(0);
  try { execSync(`npx --no-install prettier --write "${file}"`, { stdio: "ignore" }); } catch {}
  process.exit(0);
});
```

Why Node and not bash: Windows users without WSL can't run the bash version. Node ships with Claude Code's most common project types anyway.

### Pattern 2 — Block destructive shell (`PreToolUse: Bash`)
Beyond the pattern in the existing KB, **rewrite when you can** instead of always blocking.

```python
#!/usr/bin/env python3
# .claude/hooks/safer-bash.py
import json, sys, re
data = json.load(sys.stdin)
cmd = data.get("tool_input", {}).get("command", "")

DENY = [r"\brm\s+-rf\s+/", r"git\s+push.*--force-with-lease.*\s(main|master|prod)\b", r"DROP\s+(TABLE|DATABASE)"]
for p in DENY:
    if re.search(p, cmd, re.I):
        print(json.dumps({"hookSpecificOutput": {
            "hookEventName": "PreToolUse",
            "permissionDecision": "deny",
            "permissionDecisionReason": f"Blocked by safer-bash: matched {p}"
        }}))
        sys.exit(0)

# Soft rewrite: append --dry-run to terraform apply
if re.match(r"^\s*terraform\s+apply", cmd) and "--auto-approve" not in cmd:
    new = cmd + " -auto-approve=false"
    print(json.dumps({"hookSpecificOutput": {
        "hookEventName": "PreToolUse",
        "permissionDecision": "allow",
        "updatedInput": {**data["tool_input"], "command": new}
    }}))
    sys.exit(0)

sys.exit(0)
```

### Pattern 3 — Inject project state at session start (`SessionStart`)
The single highest-value hook for medium/large codebases. Anything your `state.md` would tell Claude, inject automatically — including after compaction.

```bash
#!/usr/bin/env bash
# .claude/hooks/inject-state.sh
# stdout on exit 0 is added to Claude's context for SessionStart/UserPromptSubmit

if [ -f state.md ]; then
  echo "## Current project state (from state.md)"
  cat state.md
fi

if [ -f .claude/last-session-summary.md ]; then
  echo "## Last session summary"
  cat .claude/last-session-summary.md
fi
```

```json
{
  "hooks": {
    "SessionStart": [{
      "matcher": "startup|resume|compact",
      "hooks": [{ "type": "command", "command": ".claude/hooks/inject-state.sh" }]
    }]
  }
}
```

The `compact` matcher is the killer feature — re-injects state after Claude's context is summarized away.

### Pattern 4 — Verify before stop (`Stop` agent hook)
The single most reliable way to enforce "don't claim done until tests pass."

```json
{
  "hooks": {
    "Stop": [{
      "matcher": "",
      "hooks": [{
        "type": "agent",
        "prompt": "Run `npm test --silent` and `npm run typecheck`. If either fails, return JSON: {\"decision\":\"block\",\"reason\":\"<short summary of what failed>\"}. Otherwise exit 0 with no output.",
        "tools": ["Bash"]
      }]
    }]
  }
}
```

Guard against infinite loops: agent hooks see `stop_hook_active: true` on re-invocation. Your prompt should check for it and let Claude through on the second pass if needed.

### Pattern 5 — Redact secrets from prompts (`UserPromptSubmit`)
Stops you from accidentally pasting an API key into the chat.

```python
#!/usr/bin/env python3
import json, sys, re
data = json.load(sys.stdin)
prompt = data.get("prompt", "")

PATTERNS = [
    (r"sk-[A-Za-z0-9]{32,}", "sk-***REDACTED***"),
    (r"AKIA[0-9A-Z]{16}", "AKIA***REDACTED***"),
    (r"ghp_[A-Za-z0-9]{36}", "ghp_***REDACTED***"),
]
redacted = prompt
hits = 0
for pat, rep in PATTERNS:
    new, n = re.subn(pat, rep, redacted)
    redacted, hits = new, hits + n

if hits:
    # Block + tell Claude what happened. Don't pass the secret through.
    print(f"Blocked: detected {hits} likely secret(s) in prompt. Redact and resubmit.", file=sys.stderr)
    sys.exit(2)
sys.exit(0)
```

### Pattern 6 — Protect sensitive files (`PreToolUse: Edit|Write`)
The example in `agentic-workflow-best-practices.md` uses bash; here's a JSON-output version that gives Claude a useful nudge:

```python
#!/usr/bin/env python3
import json, sys, fnmatch
data = json.load(sys.stdin)
path = data.get("tool_input", {}).get("file_path", "")

PROTECTED = [".env*", ".git/*", "*.pem", "*.key", "secrets/*", "infra/prod/*"]
for pat in PROTECTED:
    if fnmatch.fnmatch(path, pat):
        print(json.dumps({"hookSpecificOutput": {
            "hookEventName": "PreToolUse",
            "permissionDecision": "deny",
            "permissionDecisionReason": f"{path} is protected. Edit it via your secret manager / infra pipeline, not here."
        }}))
        sys.exit(0)
sys.exit(0)
```

### Pattern 7 — Audit log everything (`PostToolUse`)
Cheap and quietly powerful. Write JSONL so you can `jq` it later.

```bash
#!/usr/bin/env bash
mkdir -p .claude/logs
jq -c '{ts: now|todate, session: .session_id, tool: .tool_name, input: .tool_input, ok: (.tool_response.error == null)}' \
  >> .claude/logs/tools.jsonl
exit 0
```

Add `.claude/logs/` to `.gitignore`. Useful for "what did Claude actually do in that 3-hour session?" post-mortems.

### Pattern 8 — Desktop notification when idle (`Notification`)
Stop checking the terminal every minute.

```json
{
  "hooks": {
    "Notification": [{
      "matcher": "permission_prompt|idle_prompt",
      "hooks": [{
        "type": "command",
        "command": "powershell -c \"[reflection.assembly]::loadwithpartialname('System.Windows.Forms') | Out-Null; [System.Windows.Forms.MessageBox]::Show('Claude needs you')\""
      }]
    }]
  }
}
```

macOS: `osascript -e 'display notification "Claude needs you"'`. Linux: `notify-send "Claude" "Awaiting input"`.

### Pattern 9 — Block prompt expansions you don't want (`UserPromptExpansion`)
A custom slash command pulled in from a plugin you didn't audit yet? Reject the expansion before it reaches Claude.

```python
#!/usr/bin/env python3
import json, sys
data = json.load(sys.stdin)
expanded = data.get("expandedPrompt", "")
if "rm -rf" in expanded or "force-push" in expanded.lower():
    print(json.dumps({"decision": "block", "reason": "Slash-command expansion contains a dangerous primitive. Review the command file before running."}))
    sys.exit(0)
sys.exit(0)
```

### Pattern 10 — Save context before compaction (`PreCompact`)
Before Claude summarizes the conversation, dump the parts you don't want lost.

```bash
#!/usr/bin/env bash
mkdir -p .claude/snapshots
ts=$(date -Is | tr ':' '-')
jq -r '.transcript_path' < /dev/stdin | xargs -I{} cp "{}" ".claude/snapshots/transcript-$ts.jsonl"
exit 0
```

---

## 7. Pitfalls

These bite people repeatedly. Most are not in Anthropic's docs.

| Pitfall | What happens | Fix |
|---|---|---|
| Mixing `exit 2` with JSON output | JSON ignored; only stderr shows up | Pick one mode per hook. Default to JSON output (exit 0) — more flexible. |
| Two `PreToolUse` hooks both rewriting `tool_input` | Last one wins, **non-deterministic order** | Put rewrites in a single hook. Hooks should be additive (deny / log) when running in parallel. |
| `Stop` hook calls a tool that triggers `PreToolUse` | Infinite loop | Check `stop_hook_active` in input; exit 0 immediately if true. |
| Hook prints debug to stdout on `SessionStart` / `UserPromptSubmit` | Debug noise gets injected into Claude's context | Send debug to **stderr**, not stdout, on these events. |
| Hook script not executable | Silent fail or "hook error" notice; turn proceeds without the guard | `chmod +x .claude/hooks/*.sh`. On Windows, prefer `.py`/`.js` invoked through their interpreter. |
| Bash hook on Windows | `jq`/`grep` may be missing; paths use backslashes | Write hooks in Python or Node — both ship cross-platform. |
| Trusting `allow` to bypass deny rules | A hook returning `allow` does **not** override settings-level deny rules | Don't model permission policy in hooks alone. Use settings `permissions.deny` for hard rules; hooks for context-sensitive blocks. |
| Slow `PreToolUse` hook on a hot tool (e.g. `Bash`) | Adds latency to every shell call | Keep PreToolUse hooks under ~100ms. Push expensive checks to `Stop` (once per turn). |
| HTTP hook with no timeout | A flaky service hangs Claude Code | Set short server-side timeouts; fail-closed only when the policy genuinely requires it. |
| Hooks committed with secrets in env vars | Leaks via `git log` | Reference env vars by name in `command`; set them in `settings.local.json` (gitignored) or your shell profile. |

---

## 8. Hooks vs CLAUDE.md vs Skills vs Permissions

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

---

## 9. Bootstrapping a Repo

A starter `.claude/settings.json` that covers the high-value cases without being noisy:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup|resume|compact",
        "hooks": [{ "type": "command", "command": ".claude/hooks/inject-state.sh" }]
      }
    ],
    "UserPromptSubmit": [
      {
        "matcher": "",
        "hooks": [{ "type": "command", "command": "python .claude/hooks/redact-secrets.py" }]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [{ "type": "command", "command": "python .claude/hooks/safer-bash.py" }]
      },
      {
        "matcher": "Edit|Write",
        "hooks": [{ "type": "command", "command": "python .claude/hooks/protect-files.py" }]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [{ "type": "command", "command": "node .claude/hooks/format.js" }]
      },
      {
        "matcher": ".*",
        "hooks": [{ "type": "command", "command": ".claude/hooks/audit-log.sh" }]
      }
    ],
    "Stop": [
      {
        "matcher": "",
        "hooks": [{
          "type": "agent",
          "prompt": "If `package.json` exists, run `npm test --silent`. If it fails, block with the failure summary. Otherwise allow.",
          "tools": ["Bash"]
        }]
      }
    ]
  }
}
```

Build it up incrementally — turn on one event at a time, watch `/hooks` and `.claude/logs/tools.jsonl`, only add the next when the previous is quiet.

---

## 10. References

- Hooks reference: [docs.claude.com/en/docs/claude-code/hooks](https://docs.claude.com/en/docs/claude-code/hooks)
- Hooks how-to guide: [docs.claude.com/en/docs/claude-code/hooks-guide](https://docs.claude.com/en/docs/claude-code/hooks-guide)
- Agent SDK hooks: [code.claude.com/docs/en/agent-sdk/hooks](https://code.claude.com/docs/en/agent-sdk/hooks)
- Schema fragmentation tracker: [github.com/anthropics/claude-code/issues/19115](https://github.com/anthropics/claude-code/issues/19115)
- Related in this KB: `agentic-workflow-best-practices.md` (basic hook examples), `claude-code-2026-update.md` (auto mode + hooks)
