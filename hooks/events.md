# Hook Events

Anthropic groups events into three cadences. The tables below are the full list — far more than the 4 events in `claude-code-2026-update.md`.

The minimum useful subset for a serious project is: `PreToolUse`, `PostToolUse`, `UserPromptSubmit`, `SessionStart`, `Stop`, `Notification`.

---

## Once per session

| Event | Fires | Can block? | Common use |
|---|---|---|---|
| `SessionStart` | Session begins or resumes (matcher: `startup`, `resume`, `clear`, `compact`) | No (but stdout is injected into context) | Inject project state, load `state.md`, post-compaction context recovery |
| `SessionEnd` | Session terminates | No | Persist last-session summary, flush logs |
| `Setup` | One-shot init: `claude --init-only`, or `--init`/`--maintenance` in `-p` mode | No | CI/script preparation that should run before the first prompt |
| `PreCompact` | Before the conversation is compacted | No | Save important context to disk before it's summarized away |

## Once per turn

| Event | Fires | Can block? | Common use |
|---|---|---|---|
| `UserPromptSubmit` | User submits a prompt, before Claude sees it | Yes | Inject context, redact secrets, reject prompts that violate policy |
| `UserPromptExpansion` | A typed slash command expands into a prompt | Yes | Block dangerous slash commands; rewrite expansions |
| `Stop` | Main agent finishes a turn | Yes (force Claude to keep going) | Run tests / typecheck before declaring done |
| `StopFailure` | Turn ends in error | No | Alert / log |

## Per tool call (inside the agentic loop)

| Event | Fires | Can block? | Common use |
|---|---|---|---|
| `PreToolUse` | Before a tool runs | Yes | Block `rm -rf`, gate `git push --force`, rewrite tool input |
| `PostToolUse` | After tool succeeds | Yes (re-prompts Claude) | Lint/format, log, append context |
| `PostToolUseFailure` | After tool errors | Yes | Auto-retry policy, structured error feedback |
| `PostToolBatch` | After a parallel batch of tool calls fully resolves, before next model turn | Yes | Inject conventions once per batch instead of per call |
| `PermissionRequest` | A permission dialog would appear | Yes | Custom approval logic; auto-approve a known-safe pattern |
| `PermissionDenied` | Auto-mode classifier denied the call | No (return `{retry:true}`) | Tell Claude it may retry with a different approach |

## Subagents and tasks

| Event | Fires | Can block? |
|---|---|---|
| `SubagentStart` | A subagent is spawned | No |
| `SubagentStop` | A subagent finishes | Yes (force it to keep going) |
| `TaskCreated` | A task is created via `TaskCreate` | No |
| `TaskCompleted` | A task is marked completed | No |

## Notifications

| Event | Matcher value | Common use |
|---|---|---|
| `Notification` | `permission_prompt`, `idle_prompt`, `auth_success`, `elicitation_dialog`, `elicitation_complete`, `elicitation_response` | Desktop alerts, Slack pings when Claude is idle, OAuth-success logging |

---

Next: pick a [handler type](./handlers.md), then check the [JSON contract](./contract.md).
