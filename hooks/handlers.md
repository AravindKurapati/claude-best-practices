# The Four Handler Types

`type` in your hook config picks one. The first three are well-documented; the fourth (`http`) is undocumented in most secondary sources.

---

## `type: "command"` — shell command
Most common. Fast, predictable, no model spend. Receives JSON on stdin, communicates via exit code + stdout/stderr.

```json
{ "type": "command", "command": ".claude/hooks/block-dangerous.sh" }
```

## `type: "prompt"` — small model judgment call
Sends a yes/no question to a Claude model (Haiku by default). Use when a regex isn't enough — e.g. "is this commit message a stub?" rather than "does this command match `rm -rf`?"

```json
{ "type": "prompt", "prompt": "Does this diff modify any test files? Answer yes or no.", "model": "claude-haiku-4-5-20251001" }
```

Cost: one tiny API call per fire. Don't use prompt hooks on `PostToolUse` for `Edit|Write` unless you genuinely need judgment — you'll burn tokens on every edit.

## `type: "agent"` — full subagent
Spawns a subagent that can read files, run commands, verify conditions. 60-second timeout, up to ~50 tool turns. Use for `Stop` hooks that need to actually verify work — e.g. "before declaring done, run the test suite and confirm it passes."

```json
{
  "type": "agent",
  "prompt": "Run `pytest tests/ -q`. If anything fails, return `{decision: 'block', reason: <failure summary>}`. Otherwise allow.",
  "tools": ["Bash", "Read"]
}
```

## `type: "http"` — POST to a URL
The same JSON body that a command hook sees on stdin is sent as a POST request body. The HTTP response body becomes the decision JSON. Useful when the policy lives in a service (auth check against your IAM, audit log to a SIEM, central rate limiter).

```json
{ "type": "http", "url": "https://hooks.internal.example.com/claude/pre-bash", "headers": { "Authorization": "Bearer ..." } }
```

Caveats: latency adds to every tool call; failures are treated as errors (Claude Code shows a hook-error notice and proceeds). Set tight timeouts on the server side.

---

## Picking between them

| If you need... | Use |
|---|---|
| Regex / path / static rule | `command` |
| A yes/no semantic judgment | `prompt` |
| Verification that requires running tools | `agent` |
| Centralized policy across many machines/users | `http` |

Next: [JSON contract](./contract.md) — how to read input and shape your output.
