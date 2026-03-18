# Skills Best Practices

## What a Skill Is
- A folder that teaches Claude how to handle a specific task repeatedly
- Teach it once -> Claude uses it automatically, forever
- Open standard - works with Claude Code, Claude.ai, API, and other AI agents
- Skills share the context window - treat context as a public good, stay lean

## When to Build a Skill (vs other options)
| Situation | Solution |
|---|---|
| Repeatable workflow (same steps every time) | Skill |
| External tool/API needed | MCP + Skill on top |
| Consistent document output (reports, decks) | Skill |
| Agent behavior rules | AGENTS.md |
| Project-specific architecture | CLAUDE.md |
| Domain too complex for one file | Skill graph |

## Skill Architecture
```
your-skill-name/       <- kebab-case folder name
  SKILL.md             <- required. core instructions + YAML frontmatter
  scripts/             <- optional. executable scripts Claude can run
  templates/           <- optional. output templates
  references/          <- optional. deep docs loaded only when needed
```

## YAML Frontmatter - Most Critical Part
Two fields matter most:
- `name` - lowercase with hyphens, no spaces or capitals
- `description` - what it does + specific trigger phrases users are likely to say

```yaml
---
name: sprint-planning
description: Manages sprint planning workflows including task creation and capacity
  evaluation. Use when user mentions "sprint", "planning session", "create tickets",
  or "what should we build next".
context: fork
---
```

Nail the description -> skill triggers at the right moment.
Get it wrong -> skill sits there doing nothing.

`context: fork` - use for search-heavy or output-heavy skills. Runs in isolated context, doesn't pollute the main window.

## Core Design Principles
1. **Context window is a public good** - be lean
2. **Claude is already smart** - give it structure, not hand-holding
3. **Progressive disclosure** - minimal instructions first, reference deeper docs only when needed
4. **Keep SKILL.md under 5,000 words** - offload to `references/` if you exceed this

## The 3 Skill Categories
1. **Workflow Automation** - repeatable multi-step processes (sprint planning, onboarding, CI/CD)
2. **MCP Enhancement** - expertise layer on top of tool access
3. **Document & Asset Creation** - consistent outputs following your standards

## Five Design Patterns (Anthropic official guide)
1. **Sequential pipelines** - rigid multi-step. Add rollback instructions for step failures.
2. **Cross-service workflows** - multiple MCPs. Validate data from MCP-A before passing to MCP-B.
3. **Iterative refinement** - high-stakes outputs. Define stopping conditions.
4. **Dynamic routing** - decision-making based on file types or sizes.
5. **Parallel orchestration** - spawn task agents, keep main context clean.

## Building Skills from Your Own Projects
- At every milestone, tell Claude to document what it did and how in a markdown file
- For every bug: trace root cause, fix it, document the full breakdown
- When project is done, feed all documentation into `skill-creator`
- Result: a reusable skill grounded in your actual codebase patterns

## Using skill-creator
- Meta-skill from Anthropic to scaffold new skills
- When a skill fails on first run -> ask Claude to fix the skill doc at the end
- Self-healing loop: run -> fail -> fix skill doc -> run again -> improve

## Offloading from CLAUDE.md to Skills
- When CLAUDE.md grows with "don't do X" patches -> move those to a dedicated skill
- Skills handle edge cases and domain specifics; CLAUDE.md handles architectural intent

## Practical Daily Skills Worth Knowing
From experienced Claude Code users:
- `/grill-me` - stress-test your ideas before building
- `/write-a-prd` - turn a goal into a proper spec
- `/prd-to-issues` - break a PRD into trackable tickets
- `/tdd` - enforce test-driven development workflow
- `/improve-my-codebase` - systematic codebase improvement pass

---

## Self-Improving Skills (Advanced / Emerging Pattern)

> Graduate to this when static skills are working but you're hitting maintenance friction at scale.

### The Problem
Skills are static. The environment around them is not. A skill quietly fails when:
- The codebase changes
- The model behaves differently
- Task patterns shift over time

Failures are invisible until output quality visibly drops.

### The Self-Improvement Loop
```
Observe -> Inspect -> Amend -> Evaluate -> Update or Rollback
```

**Observe** - after each skill run, store: what task was attempted, which skill was selected, whether it succeeded, what error occurred, user feedback.

**Inspect** - once failures accumulate, trace recurring patterns behind bad outcomes. Use evidence to propose a better version.

**Amend** - system proposes a targeted patch. Might tighten the trigger, add a missing condition, reorder steps, fix a broken tool call.

**Evaluate** - verify the amendment actually improved outcomes. If not -> rollback. Every change is tracked with rationale. Original instructions are never lost.

### The Progression
```
Static SKILL.md
    - (skills drift as codebase changes)
Periodic manual rewrites every ~2 weeks  <- where most people should be
    - (many skills, maintenance becomes the bottleneck)
Self-improving skills via observation loop (cognee-skills)
```

### Tool: cognee-skills
- Stores skills as a semantic graph with enriched fields, task patterns, summaries, relationships
- Adds observation nodes after each run
- Proposes amendments when failure patterns accumulate
- Dynamic graph view: `cognee-graph-skills.vercel.app`

---
### Autoresearch in Practice (Karpathy method applied to skills)
- Define a checklist of 3–6 yes/no scoring questions - what "good output" looks like
- Agent runs skill -> scores output -> makes one change -> keeps if score improves, reverts if not -> repeats
- Stops at 95%+ score three times in a row
- Outputs: improved skill file, results log, changelog of every attempted change
- The changelog is the most durable artifact - hand it to a smarter model later and it picks up where the last one left off
- Works on anything scorable: prompts, cold outreach, newsletter intros, page load time
- Source: aisolo.beehiiv.com

## Official Reference
Full 33-page guide: `resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skills-for-Claude.pdf`
-> See `anthropic-skills-guide.md` in this knowledge base for distilled version

