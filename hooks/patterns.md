# Hook Pattern Library

Patterns ordered roughly by how often you'll reach for them. See [contract.md](./contract.md) for the JSON shapes referenced here.

---

## Pattern 1 — Auto-format on every edit (`PostToolUse`)

Cross-platform version that doesn't break on Windows (no `xargs`):

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

## Pattern 2 — Block destructive shell (`PreToolUse: Bash`)

**Rewrite when you can** instead of always blocking.

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

## Pattern 3 — Inject project state at session start (`SessionStart`)

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

## Pattern 4 — Verify before stop (`Stop` agent hook)

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

## Pattern 5 — Redact secrets from prompts (`UserPromptSubmit`)

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

## Pattern 6 — Protect sensitive files (`PreToolUse: Edit|Write`)

JSON-output version that gives Claude a useful nudge:

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

## Pattern 7 — Audit log everything (`PostToolUse`)

Cheap and quietly powerful. Write JSONL so you can `jq` it later.

```bash
#!/usr/bin/env bash
mkdir -p .claude/logs
jq -c '{ts: now|todate, session: .session_id, tool: .tool_name, input: .tool_input, ok: (.tool_response.error == null)}' \
  >> .claude/logs/tools.jsonl
exit 0
```

Add `.claude/logs/` to `.gitignore`. Useful for "what did Claude actually do in that 3-hour session?" post-mortems.

## Pattern 8 — Desktop notification when idle (`Notification`)

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

## Pattern 9 — Block prompt expansions you don't want (`UserPromptExpansion`)

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

## Pattern 10 — Save context before compaction (`PreCompact`)

Before Claude summarizes the conversation, dump the parts you don't want lost.

```bash
#!/usr/bin/env bash
mkdir -p .claude/snapshots
ts=$(date -Is | tr ':' '-')
jq -r '.transcript_path' < /dev/stdin | xargs -I{} cp "{}" ".claude/snapshots/transcript-$ts.jsonl"
exit 0
```

---

## Bootstrapping a Repo

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
