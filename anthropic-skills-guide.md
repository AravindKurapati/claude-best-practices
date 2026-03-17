# Anthropic's Official Skills Guide — Distilled

Source: "The Complete Guide to Building Skills for Claude" (Feb 2026, 33 pages)
PDF: `resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skills-for-Claude.pdf`

## What the Guide Covers
- Fundamentals
- Planning and design
- Testing and iteration
- Distribution and sharing
- Patterns and troubleshooting
- YAML frontmatter reference

## The Single Most Important Insight
Skills are not prompt templates. They are reusable operational assets.
MCP is the kitchen. Skills are the recipes.
Without skills, users connect an MCP and stare at a blank screen. With skills, workflows activate automatically.

## YAML Frontmatter Reference
```yaml
---
name: your-skill-name              # kebab-case, lowercase, no spaces
description: |                     # max 1,024 characters
  What this skill does. Include exact phrases users are likely to say.
  Use when user says "X", "Y", or "Z". Avoid XML tags here.
compatibility: claude-code         # optional: claude-code | claude-ai | api | all
context: fork                      # optional: fork = run in isolated context
version: 1.0.0
---
```

`context: fork` is critical for token isolation — runs the skill in a separate context so it doesn't pollute the main window. Always use for search-heavy or output-heavy skills.

## Progressive Disclosure — How It Works in Practice
```
Level 1: YAML frontmatter (always loaded, globally)
         ↓ Claude decides if skill is relevant
Level 2: SKILL.md instructions (loaded when skill triggers)
         ↓ Claude decides if deep reference is needed
Level 3: references/ folder (loaded only when explicitly needed)
```

Most decisions happen at Level 1. Most execution happens at Level 2. Level 3 only for truly complex edge cases.

## SKILL.md Structure
```markdown
# Skill Name

## Instructions

### Step 1: [Action]
Specific, actionable. Not "validate the data" — write 
"Run python scripts/validate.py --input {filename}"

### Step 2: [Action]
...

## Examples
User says: "set up a new project"
1. Fetch existing projects via MCP
2. Create with provided parameters
Result: Project created with confirmation link

## Troubleshooting
Error: Connection refused
Cause: MCP server not running
Fix: Settings > Extensions > Reconnect
```

## Testing Your Skill
Track these three KPIs:
1. **Trigger Rate** — target 90%+ accuracy on relevant queries. If skill doesn't trigger, fix the description.
2. **API Call Success Rate** — target 0 failed calls per workflow
3. **Token Efficiency** — measure context reduction vs mega-prompts

## Anthropic Benchmarks
For complex project creation: well-structured skills reduced clarifications from 15 to 2 and cut token consumption from 12,000 to 6,000.

## Distribution
- Skills work identically across Claude.ai, Claude Code, and API — write once, use everywhere
- Published as an open standard — can be used with other AI agents (not locked to Claude)
- Global skills: `~/.claude/skills/` — available across all projects
- Project skills: `.claude/skills/` — scoped to one project

## What the Guide Says About CLAUDE.md
Apply all skill authoring best practices to CLAUDE.md. Same principles:
- Short, focused, intent-driven
- Progressive disclosure (lazy load sub-files)
- Description/trigger logic determines what Claude loads

