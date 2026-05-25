# Hook Pitfalls & References

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

## References

- Hooks reference: [docs.claude.com/en/docs/claude-code/hooks](https://docs.claude.com/en/docs/claude-code/hooks)
- Hooks how-to guide: [docs.claude.com/en/docs/claude-code/hooks-guide](https://docs.claude.com/en/docs/claude-code/hooks-guide)
- Agent SDK hooks: [code.claude.com/docs/en/agent-sdk/hooks](https://code.claude.com/docs/en/agent-sdk/hooks)
- Schema fragmentation tracker: [github.com/anthropics/claude-code/issues/19115](https://github.com/anthropics/claude-code/issues/19115)
- Related in this KB: `agentic-workflow-best-practices.md` (basic hook examples), `claude-code-2026-update.md` (auto mode + hooks)
