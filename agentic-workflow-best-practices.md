# Agentic Workflow Best Practices

## When to Use What
| You need... | Use |
|---|---|
| Universal behavior rules | Global `~/.claude/CLAUDE.md` |
| Project-specific context | Project `CLAUDE.md` + `/documents/` |
| Repeatable workflow | Skill |
| External tool/API access | MCP server |
| Specialist capability | Sub-agent or agency-agent |
| Complex domain knowledge | Skill graph |

---

## AGENTS.md - Configuration Over Babysitting
- Write clear rules for when agent should ask vs when to act
- Vague prompts = constant interruptions; frontload decisions into config
- Include: structured memory files, explicit tool permissions, checkpoint intervals
- The 10x productivity comes from the agent running uninterrupted, not from the model itself

---

## The 4-File Framework for Multi-Agent Systems
Most teams write SKILL.md and INSTRUCTIONS.md naturally. What teams consistently skip are Agent.md and AGENTS.md - because they feel administrative. But agents don't know what you know. Every new conversation starts from scratch.

```
repo/
├── AGENTS.md                  <- System map. Who exists. How they connect.
└── agents/
    └── analyst/
        ├── Agent.md           <- I am analyst-agent. I report to orchestrator.
        ├── INSTRUCTIONS.md    <- Step 1: validate. Step 2: compute. Never hallucinate.
        └── skills/
            └── SKILL.md       <- How to run pandas analysis specifically.
```

**AGENTS.md** - the org chart. Lives at project root. Every agent reads it so they have a mental model of the whole system, not just their corner. Update it whenever you add or remove an agent.

**Agent.md** - one per agent. Short (30 lines max). Answers: who am I, who spawned me, who do I hand off to, what do I NOT do. The "what I do NOT do" section is the most important - agents fill gaps that aren't explicitly closed.

**INSTRUCTIONS.md** - the daily routine. Numbered steps, not prose. Rules answer "always/never." Workflows answer "first/then/finally." Separate them - mixing creates ambiguity about which takes priority.

**SKILL.md** - the training certificate. One skill per capability, atomic and reusable. Include failure modes, not just sunny-day scenarios.

Mental model: AGENTS.md is the city map. Agent.md is your apartment address. INSTRUCTIONS.md is your daily routine. SKILL.md is your job training certificate.

---

## Parallel Sub-Agents
Most people run Claude Code linearly....don't.

**Level 1 - Multiple terminal sessions (start here)**
- Open multiple terminal tabs, run `claude` in each
- Manually coordinate by switching tabs
- Writer/Reviewer pattern: one agent writes, fresh agent reviews (no bias toward its own code)
- Only ~1/3 of issues overlap across different LLM sessions - broader coverage = fewer bugs

**Level 2 - Orchestrator + subagents (advanced)**
- One Claude Code agent becomes the orchestrator
- Orchestrator spawns other agents for specific parallel tasks
- Collects results, decides what to keep
- Cuts complex project time roughly in half
- Bottleneck becomes orchestration logic, not generation speed

```
Orchestrator
├── Research agent    (finds best approach)
├── Implementation agent  (writes the code)
└── Review agent     (checks the output)
```

**cmux** - terminal workspace manager that makes this manageable at scale
- Named workspaces with emojis instead of anonymous tabs
- Orchestrator can programmatically list, read, and send to any workspace
- Three commands: `cmux list-workspaces`, `cmux read-screen`, `cmux send`
- Pair with Obsidian Bases for a session dashboard (status, progress, outcomes)
- Only relevant once you're regularly running 3+ agents simultaneously

---

## Security Audit Pattern
- After big code changes: "Check this project for critical vulnerabilities. Ignore non-severe."
- Use parallel sub-agents per attack surface (secrets/keys, auth bypass, injection/XSS)
- Use strong models (Opus) to find issues, cheaper models (Sonnet/Haiku) to fix them
- Each agent runs independently - faster and doesn't pollute main context

## Planning Before Coding
- For any non-trivial feature: shift-tab into planning mode first
- Add "do deep research on best practices and known issues, using web search" to prompt
- READ the plan, adjust it, THEN execute
- Plans survive context compaction much better than vibe-prompted features

## Force Reasoning Before Code
Structure prompts like this:
1. Analyze the problem
2. Propose two possible solutions
3. Choose the best approach and explain why
4. Then implement

Prevents Claude from jumping straight into mediocre code - forces senior-engineer thinking.

## Spec-First Development
- For every code change, ask Claude to write a spec first
- Review and refine the spec with Claude, THEN ask it to implement
- Name files: `FEATURE_xyz.md` or `TWEAK_xyz.md`
- Every spec includes impact on database structure -> Claude knows when to update SCHEMA.md
- Claude is faster at implementation when spec + DB model are directly referenced

## Bug Handling (Single Biggest CLAUDE.md Improvement)
Add this rule to your global CLAUDE.md:
> "When I report a bug, don't start by trying to fix it. Instead, write a test that reproduces the bug. Then have subagents try to fix it and prove it with a passing test."

Why it works:
- Forces root cause understanding before any fix
- Subagents work in parallel without polluting main context
- Passing test = proof the fix works, not just assumed

## Auditing Your Own Habits (One-Time Setup)
Run this in Claude Code:
> "Scrape all of my Claude sessions on this computer. Give me a breakdown of all the things I do, things worth making into skills vs plugins vs agents vs CLAUDE.md entries."

Surfaces patterns you didn't know you had. Use output to bootstrap initial skills and CLAUDE.md.

## Commands for Regular Tasks
- Have Claude create commands for tasks you run repeatedly
- After each successful run, ask it to update the commands to reflect learnings
- Commands become self-improving over iterations

---

## Thinking Budget Fixes (Claude Code Regression)
The regression is real but fixable. Default thinking effort was made conservative — for complex multi-file work it's not enough. When thinking depth drops, Claude shifts from research-first to edit-first behavior.

Three fixes:
1. `/effort high` in your prompt (or `/effort max` on Opus for hard debugging)
2. `~/.claude/settings.json` -> `"showThinkingSummaries": true` (see what Claude is actually reasoning about)
3. Add to CLAUDE.md: `"Research the codebase before editing. Never change code you haven't read."`

---

## Claude Code Hooks
Hooks are automatic actions that fire on every file edit or command run. Unlike CLAUDE.md (followed ~80% of the time), hooks are deterministic.

Two types:
- **PreToolUse** — runs before Claude acts. Exit code 2 blocks the action and sends your message back to Claude.
- **PostToolUse** — runs after. Use for formatting, linting, logging.

Config location: `.claude/settings.json` (project, committed to git) or `~/.claude/settings.json` (global).

### Hook 1 — Auto-format on every edit
Runs your formatter after every Write or Edit. Swap `prettier` for `black`, `gofmt`, `rustfmt` etc.

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [{ "type": "command", "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write 2>/dev/null; exit 0" }]
      }
    ]
  }
}
```

### Hook 2 — Block dangerous commands
Create `.claude/hooks/block-dangerous.sh`:

```bash
#!/usr/bin/env bash
cmd=$(jq -r '.tool_input.command // ""')
dangerous_patterns=("rm -rf" "git reset --hard" "git push.*--force" "DROP TABLE" "DROP DATABASE" "curl.*|.*sh" "wget.*|.*bash")
for pattern in "${dangerous_patterns[@]}"; do
  if echo "$cmd" | grep -qiE "$pattern"; then
    echo "Blocked: '$cmd' matches '$pattern'. Propose a safer alternative." >&2
    exit 2
  fi
done
exit 0
```

```json
{
  "hooks": {
    "PreToolUse": [
      { "matcher": "Bash", "hooks": [{ "type": "command", "command": ".claude/hooks/block-dangerous.sh" }] }
    ]
  }
}
```

### Hook 3 — Protect sensitive files
Create `.claude/hooks/protect-files.sh`:

```bash
#!/usr/bin/env bash
file=$(jq -r '.tool_input.file_path // .tool_input.path // ""')
protected=(".env*" ".git/*" "package-lock.json" "yarn.lock" "*.pem" "*.key" "secrets/*")
for pattern in "${protected[@]}"; do
  if echo "$file" | grep -qiE "^${pattern//\*/.*}$"; then
    echo "Blocked: '$file' is protected." >&2; exit 2
  fi
done
exit 0
```

```json
{
  "hooks": {
    "PreToolUse": [
      { "matcher": "Edit|Write", "hooks": [{ "type": "command", "command": ".claude/hooks/protect-files.sh" }] }
    ]
  }
}
```

### Hook 4 — Log every command
Create `.claude/hooks/log-commands.sh`:

```bash
#!/usr/bin/env bash
cmd=$(jq -r '.tool_input.command // ""')
printf '%s %s\n' "$(date -Is)" "$cmd" >> .claude/command-log.txt
exit 0
```

Add `.claude/command-log.txt` to `.gitignore`. Useful for debugging Modal runs or any long agentic session.

### Combined settings.json
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          { "type": "command", "command": ".claude/hooks/block-dangerous.sh" },
          { "type": "command", "command": ".claude/hooks/log-commands.sh" }
        ]
      },
      {
        "matcher": "Edit|Write",
        "hooks": [{ "type": "command", "command": ".claude/hooks/protect-files.sh" }]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          { "type": "command", "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write 2>/dev/null; exit 0" }
        ]
      }
    ]
  }
}
```

Make scripts executable: `chmod +x .claude/hooks/*.sh`

Full docs: `code.claude.com/docs/en/hooks`

---

## Superpowers (Pre-built Skill Pack)
GitHub repo with 40.9k stars - a complete development methodology as skills.

What it enforces:
- Spec-first: agent stops and brainstorms before writing a single line
- Creates implementation plan detailed enough for a junior engineer to follow
- Subagent-driven development: fresh subagents per task, two-stage code review after each
- True TDD: write failing test -> watch it fail -> write minimal code -> watch it pass -> commit. Deletes code written before tests.
- Verify before declaring success, then presents options (merge, PR, keep, discard)

Philosophy: systematic over ad-hoc. Evidence over claims. Complexity reduction.

Install as a Claude Code plugin. Works with Codex and OpenCode too.

**5 practical daily skills worth knowing:**
- `/grill-me` - stress-test your ideas before building
- `/write-a-prd` - turn a goal into a proper spec
- `/prd-to-issues` - break a PRD into trackable tickets
- `/tdd` - enforce test-driven development workflow
- `/improve-my-codebase` - systematic codebase improvement pass

---

## Claude Code Shortcuts
- **`!` prefix** - runs bash inline, command + output land directly in context
- **`Ctrl+S`** - stashes current draft, pops back after you send something else
- **`Ctrl+G`** - opens prompt in `$EDITOR` for bigger edits

## Claude Code Plugins Worth Installing
- `mksglu/claude-context-mode` - bigger context window via reference loading
- `claude-warden` - protection against destructive actions
- `arscontexta` - builds skill graphs (249-file knowledge system)
