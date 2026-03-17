# CLAUDE.md Best Practices

## Core Philosophy
- Treat CLAUDE.md like a Skill — apply all Skill authoring best practices to it
- It level-sets the agent on your product so it "speaks the same language" — that's the real 10x
- Not a memory dump — architectural grounding for every session
- CLAUDE.md is injected into context on EVERY interaction — every token here costs you reasoning space

## Keep It Short (50–150 lines max)
- A 50-line file with clear architectural intent beats a 1,000-line file of "don't do X" patches
- Negations ("don't use X", "never do Y") dilute important directives and hurt performance
- The model has to parse through hundreds of negations — important directives get diluted
- Rule of thumb: over 150 lines → rewrite from scratch
- A/B tested: short + intentional >> long + patchy

## Rewrite Periodically
- CLAUDE.md has a "half life" — degrades over time as patches accumulate
- Recommended cadence: rewrite every ~2 weeks
- Start fresh, capture only current architectural intent
- Move outdated "don't do X" patches to Skills or delete them entirely
- Think of it like technical debt: don't keep patching, periodically refactor

## Lazy Loading via Sub-files
- Keep CLAUDE.md at ~100 lines by offloading domain-specific instructions
- Pattern: write "read API.md when touching API routes" inside CLAUDE.md
- Claude only loads sub-files when actually needed → saves context
- Example sub-files: `API.md`, `PAYMENTS.md`, `AUTH.md`, `SCHEMA.md`

## What to Reference from CLAUDE.md
- Link to all `/documents/*.md` files (platform-docs, styleguide, ICPs, roadmap, data-reference)
- Force code to be compliant with those docs
- Reference AGENTS.md for agent behavior rules
- Reference SCHEMA.md so Claude knows when to update it

## High-Value Rules to Include
**Bug handling:**
> "When I report a bug, don't start by trying to fix it. Instead, write a test that reproduces the bug. Then have subagents try to fix it and prove it with a passing test."

**Search policy (if using Exa MCP):**
> "For all search, retrieval, or web-based research: use the Exa MCP. Do not use built-in web search. If Exa is unavailable, ask the user how to proceed."

**Spec-first:**
> "For every non-trivial code change, write a spec file first (FEATURE_xyz.md). Include impact on database structure. Update SCHEMA.md when needed."

## Global vs Project CLAUDE.md
- `~/.claude/CLAUDE.md` — universal rules true for every project (Exa policy, bug-fix rule, ask-vs-act)
- `your-project/CLAUDE.md` — project-specific architecture, lazy-loads /documents/
- Skills in `~/.claude/skills/` — portable across all projects
- Skills in `.claude/skills/` — only for that project

## Per-Folder Instructions
- Create `discussion/` and `decisions/` subfolders per topic
- Instruct Claude to save a summary of discussion + decision after each commit

