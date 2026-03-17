# Knowledge Base Index — Claude & Claude Code Best Practices

A decision guide: what to read and when. Start here every session.

---

## I'm starting a brand new project
→ `project-docs-structure.md` — vibe coding prompt, /documents/ folder, what to generate first

## I'm onboarding an existing codebase into Claude Code
→ `project-docs-structure.md` → Existing Project Onboarding section
→ `claude-md-best-practices.md` → rewrite CLAUDE.md after audit

## My CLAUDE.md is getting bloated / Claude performance is degrading
→ `claude-md-best-practices.md` → Keep It Short + Rewrite Periodically sections

## Claude keeps making wrong assumptions about my product
→ `project-docs-structure.md` → platform-docs.md + styleguide.md sections

## Agent keeps interrupting or going off-rails
→ `agentic-workflow-best-practices.md` → AGENTS.md section

## Building a non-trivial feature without breaking things
→ `agentic-workflow-best-practices.md` → Planning Before Coding + Spec-First sections

## A bug appeared — how should Claude handle it?
→ `agentic-workflow-best-practices.md` → Bug Handling section
→ Rule: write a reproducing test first, then fix, then prove with passing test

## I want to reuse a workflow across projects
→ `skills-best-practices.md` — skill authoring, skill-creator, building from project docs
→ `anthropic-skills-guide.md` — official patterns, YAML frontmatter, trigger descriptions

## My domain is too complex for a single skill file
→ `skill-graphs.md` — wikilink networks, MOCs, progressive disclosure, arscontexta

## I need Claude to search the web inside Claude Code
→ `mcp-tools.md` → Exa MCP section + CLAUDE.md policy to force it

## I need specialist agents (security, design, marketing, etc.)
→ `mcp-tools.md` → Agency Agents section

## I want to understand Skills vs MCP vs Agents vs CLAUDE.md
→ `mcp-tools.md` → Skills vs MCP section
→ `agentic-workflow-best-practices.md` → When to Use What section

## I want to learn agent building from scratch
→ `learning-resources.md` — books, papers, videos, courses, newsletters

---

## File Map
| File | What's Inside |
|------|--------------|
| `claude-md-best-practices.md` | Line limits, lazy loading, rewrite cadence, sub-file pattern |
| `agentic-workflow-best-practices.md` | AGENTS.md config, planning mode, parallel agents, spec-first, bug handling, multi-LLM review |
| `skills-best-practices.md` | Skill authoring, building from projects, skill-creator workflow |
| `anthropic-skills-guide.md` | Official 33-page guide distilled: YAML frontmatter, trigger descriptions, design patterns, progressive disclosure |
| `project-docs-structure.md` | /documents/ folder, platform-docs, ICPs, styleguide, SCHEMA.md, vibe coding prompt, global vs project config |
| `skill-graphs.md` | Wikilink networks, MOCs, progressive disclosure, arscontexta |
| `mcp-tools.md` | Exa MCP setup, agency agents, plugins, Skills vs MCP distinction |
| `learning-resources.md` | Books, papers, videos, repos, courses, newsletters for building agents |

---

*Workflow: paste new tweets here → ask Claude to update the relevant file + this index.*

