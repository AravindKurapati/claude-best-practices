# Claude Code 2026 Update -- Hooks, Auto Mode, Scheduling, and New Patterns

Everything that changed since the original knowledge base was written. Covers March 2026 launches and ecosystem shifts.

---

## Hooks -- Deterministic Enforcement

CLAUDE.md is advisory (~80% compliance). Hooks are deterministic (100%). If something must happen every time, make it a hook.

Hooks are shell commands that fire at specific lifecycle points in Claude Code. Configure in `.claude/settings.json`:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          { "type": "command", "command": "npx prettier --write $FILEPATH" }
        ]
      }
    ]
  }
}
```

### Hook Events
- **PreToolUse** -- fires before a tool runs. Can deny, allow, or rewrite tool input.
- **PostToolUse** -- fires after a tool completes. Use for formatting, linting, logging.
- **Stop** -- fires when Claude finishes responding (not on user interrupts).
- **PermissionRequest** -- fires on permission prompts (does NOT fire in non-interactive `-p` mode).

### Three Hook Types
1. **Command hooks** (`type: "command"`) -- run a shell command. Fastest, most predictable.
2. **Prompt hooks** (`type: "prompt"`) -- send a yes/no question to a Claude model (Haiku by default). Use for judgment calls.
3. **Agent hooks** (`type: "agent"`) -- spawn a subagent that can read files, run commands, and verify conditions. 60-second timeout, up to 50 tool-use turns.

### Key Behaviors
- `PreToolUse` hooks fire BEFORE permission-mode checks. A hook that returns `deny` blocks the tool even in `bypassPermissions` mode.
- A hook returning `allow` does NOT bypass deny rules from settings.
- When multiple `PreToolUse` hooks rewrite the same tool's input, last one wins (non-deterministic order). Avoid overlapping rewrites.
- Run `/hooks` to browse all configured hooks grouped by event.
- `"disableAllHooks": true` in settings to kill all hooks at once.

### When to Use Hooks vs CLAUDE.md
| Requirement | Use |
|---|---|
| Must happen 100% of the time | Hook |
| Guidance Claude should consider | CLAUDE.md |
| Auto-format on every edit | PostToolUse hook |
| Run tests before Claude stops | Stop agent hook |
| Block dangerous commands | PreToolUse hook with deny |

---

## Auto Mode -- Smart Permission Handling

Launched March 2026 as research preview. A background classifier evaluates each action in real time:
- Safe actions: auto-approved
- Risky actions: blocked, Claude pushed toward safer alternatives

Start with: `claude --enable-auto-mode` or `--permission-mode auto`, or cycle to it with `Shift+Tab` during a session.

### What It Changes
- Eliminates the approve-every-action bottleneck (Anthropic says users approve 93% of prompts anyway)
- Two-layer check: input-layer prompt-injection probe + output-layer transcript classifier
- Currently works with Sonnet 4.6 and Opus 4.6

### Availability (as of April 2026)
- Team plan: available
- Enterprise and API: rolling out
- Pro and Max individual: not yet available

### Auto Mode + Hooks
Auto mode handles the middle ground. Hooks handle the hard boundaries. Use both:
- Auto mode for general workflow acceleration
- PreToolUse hooks for non-negotiable rules (never delete production branches, always lint, etc.)

---

## Scheduled Tasks and /loop -- Autonomous Recurring Work

Two distinct systems for recurring automation:

### /loop (CLI, session-scoped)
- Runs in your terminal session. Dies when you close the terminal.
- Max 50 concurrent tasks per session.
- Auto-expires after 3 days (safety cap).
- No catch-up: if Claude is busy when a task fires, missed runs are gone.
- Sub-minute intervals round up to 1 minute.
- Best for: deployment health checks, PR monitoring during active review, CI watching during builds.

### Scheduled Tasks (Desktop, persistent)
- Run locally on your machine via Claude Desktop app.
- Persist across sessions -- only skip if machine is asleep or app is closed.
- Catch-up: queues missed executions and runs when app reopens.
- Create via sidebar or conversationally: "set up a daily code review at 9am"
- Each run gets full access to files, MCP servers, skills, connectors, plugins.
- Enable worktree toggle to isolate scheduled runs from manual work.

### Cloud Scheduled Tasks
- Run on Anthropic-managed infrastructure -- works even when your machine is off.
- Each run starts from a fresh clone.
- Use `/schedule` in CLI pointing to cloud execution.
- Best for: true always-on automation (nightly deployments, daily regression tests).

### Decision Rule
- Losing a run on terminal close is acceptable --> use `/loop`
- Run must eventually complete --> use Desktop Scheduled Tasks
- Must run regardless of machine state --> use Cloud Scheduled Tasks

---

## BMAD Method -- AI SDLC Framework

Breakthrough Method for Agile AI-Driven Development. Turns Claude Code into a structured agile team.

### What It Is
- 21 specialized AI agents (Business Analyst, PM, Architect, Developer, QA, etc.)
- 50+ guided workflows across 4 phases: Analysis --> Planning --> Solutioning --> Implementation
- Scale-adaptive: adjusts from bug fixes to enterprise systems
- Documentation-first: specs before code, reducing hallucinations
- 100% free, MIT license

### Install
```bash
npx bmad-method install
```

Or as Claude Code skills:
```bash
git clone https://github.com/aj-geddes/claude-code-bmad-skills.git
cp -r claude-code-bmad-skills/bmad-skills ~/.claude/skills/bmad-skills
```

### When to Use BMAD vs Plan Mode
- **BMAD**: substantial projects with real users, external integrations, security surface area. Generates PRDs, architecture docs, user flows before any code.
- **Plan Mode**: smaller features where you can be the one asking the hard questions.
- BMAD found 36 user flows one practitioner didn't know they needed. Caught spec contradictions and security risks before implementation.

### The Workflow
1. "Initialize BMAD for this project" --> creates config + context folders
2. "Create a product brief" --> parallel subagents do market/competitive/technical/user research
3. "Create a PRD" --> product manager agent generates requirements
4. Architecture design --> system architect agent
5. Implementation --> developer agent follows the spec
6. QA --> quality engineer agent validates against spec

---

## Skills 2.0 and Skill Evals

### The Finding
Anthropic ran their eval system against their own public document-creation skills. Result: **5 of 6 skills degraded output quality vs baseline.** Skills they had shipped and users were presumably relying on.

### Why Skills Degrade
- Models improve; base behavior shifts
- Skills written for one model version fight the next version's defaults
- No one measures skill quality after initial creation

### What Changed
- Skills 2.0 includes a testable eval system
- Run evals on skills before trusting automation to run unsupervised
- Not a one-time exercise -- recurring checks, because the model underneath keeps changing

### Practical Takeaway
- Eval your skills periodically, especially after model updates
- If a skill constrains behavior the model now handles well by default, the skill may be hurting rather than helping
- The self-improving skills pattern from `skills-best-practices.md` is now backed by official tooling

---

## MCP Lazy Loading (Tool Search)

Claude Code now supports MCP Tool Search -- lazy loading for MCP servers.

### What It Does
- MCP server tool definitions load only when Claude actually needs them
- Reduces context usage by up to 95% when running many MCP servers
- Previously, every MCP server's full tool list was loaded into context on startup

### Why This Matters
- You can now run 10+ MCP servers without context bloat
- Removes the main argument against having a large MCP stack
- Makes the "start with 1-2, expand slowly" advice less necessary

### CLI Install Syntax (New Standard)
```bash
# Local stdio servers (most common)
claude mcp add <name> -- npx -y @package/server

# Remote HTTP servers
claude mcp add --transport http <name> <url>

# With environment variables
claude mcp add <name> -e API_KEY=your-key -- npx -y @package/server

# Scoped to all projects (user-wide)
claude mcp add --scope user <name> -- npx -y @package/server
```

Manage with: `claude mcp list`, `claude mcp get <name>`, `claude mcp remove <name>`.
Verify inside Claude Code with `/mcp`.

---

## Agent Teams

First-class support for spawning multiple Claude Code instances working in parallel.

- Not just sub-agents -- full parallel sessions with isolated contexts
- Code Kit v5.0 released specifically for Agent Teams
- Pattern: Research --> Plan --> Execute --> Review --> Ship, with different agents handling each phase
- Writer/Reviewer pattern: one Claude writes, a fresh Claude reviews (no bias toward own code)

---

## Context Window Management -- Operational Hygiene

The #1 tip from experienced users: **fresh sessions per task.**

### Rules
- `/clear` between unrelated tasks
- Compact proactively at ~70% context usage
- Delegate research to subagents (keeps main context clean)
- If you've corrected Claude more than twice on the same issue, clear and start fresh
- `claude --continue` resumes most recent conversation
- `claude --resume` opens a session picker
- `Esc+Esc` (or `/rewind`) opens checkpoint history -- restore code, conversation, or both

### What to Fully Delegate
Test generation, boilerplate, migrations, documentation, formatting -- these don't need your main context.

### CLAUDE.md Slot Budget
Claude Code's system prompt uses ~50 of the ~150-200 effective instruction slots before compliance degrades. That leaves ~100 slots for your rules. Lines near the bottom of a long CLAUDE.md lose influence first.

---

## The Universal Workflow Pattern

From shanraisshan's trending repo and confirmed across multiple practitioners:

```
Research --> Plan --> Execute --> Review --> Ship
```

This is the Command --> Agent --> Skill architecture:
- **Commands** trigger workflows
- **Agents** handle specialized phases
- **Skills** encode reusable knowledge

Every major Claude Code workflow converges on this pattern regardless of project type.

---

## New MCP Ecosystem Data (March 2026)

- 12,000+ MCP servers across GitHub, npm, PyPI, and registries
- MCP donated to Linux Foundation's Agentic AI Foundation (co-founded by Anthropic, Block, OpenAI)
- Context7 is the #1 most popular server by usage (11K views on FastMCP)
- Browser automation is the fastest-growing category (Playwright #2, Puppeteer #6)
- 66% of community-built MCP servers had security vulnerabilities in a recent audit -- stick with vendor-maintained servers
- MCP is no longer Anthropic-only: OpenAI, Microsoft, Google, Amazon all support it

---

## References
- Anthropic hooks guide: `code.claude.com/docs/en/hooks-guide`
- Anthropic best practices: `code.claude.com/docs/en/best-practices`
- shanraisshan repo: `github.com/shanraisshan/claude-code-best-practice`
- BMAD Method: `github.com/bmad-code-org/BMAD-METHOD`
- BMAD Claude Code skills: `github.com/aj-geddes/claude-code-bmad-skills`
- Builder.io 50 tips: `builder.io/blog/claude-code-tips-best-practices`
- Skills 2.0 analysis: `medium.com/@ai_transfer_lab` (April 2026)
