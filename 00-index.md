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
→ `skills-authoring.md` — skill authoring, skill-creator, building from project docs
→ `skills-official.md` — official patterns, YAML frontmatter, trigger descriptions

## My domain is too complex for a single skill file
→ `skills-architecture.md` — wikilink networks, MOCs, progressive disclosure, arscontexta

## I need Claude to search the web inside Claude Code
→ `mcp-tools.md` → Exa MCP section + CLAUDE.md policy to force it

## I need specialist agents (security, design, marketing, etc.)
→ `mcp-tools.md` → Agency Agents section

## I want to understand Skills vs MCP vs Agents vs CLAUDE.md
→ `mcp-tools.md` → Skills vs MCP section
→ `agentic-workflow-best-practices.md` → When to Use What section

## I want to learn agent building from scratch
→ `learning-resources.md` — books, papers, videos, courses, newsletters

## I'm ready to make a project public / before submitting take-home assignment
→ `portfolio-best-practices.md` — what hiring managers look for in 5 min
→ **Skill**: `project-launch-checklist` — automated audit (README, tests, CI/CD, eval, config, deployment)

## I want a rule enforced 100% of the time (not 80% like CLAUDE.md)
→ `hooks/README.md` — overview + when-to-use-hook table, then drill into `events.md`, `handlers.md`, `contract.md`, `patterns.md`, `pitfalls.md`
→ `claude-code-2026-update.md` → Hooks section for the high-level overview

## Claude keeps forgetting state across compactions / new sessions
→ `hooks/patterns.md` → Pattern 3 (`SessionStart` + `compact` matcher injects `state.md` automatically)

## I want background agents on a schedule (deploy checks, PR triage)
→ `claude-code-2026-update.md` → Scheduled Tasks and /loop section (CLI vs Desktop vs Cloud)

## I want Claude to auto-approve safe actions and only stop me on risky ones
→ `claude-code-2026-update.md` → Auto Mode section
→ Pair with `hooks/patterns.md` for non-negotiable hard boundaries

## I want a structured AI-SDLC framework (PRDs, architecture docs, dev/QA agents)
→ `claude-code-2026-update.md` → BMAD Method section

## I want the deep mechanics of MCP (lifecycle, lazy loading, building servers, debugging)
→ `mcp-patterns.md` — deep dive: transport modes, lazy loading, tool design, auth, debugging, pitfalls
→ `claude-code-2026-update.md` → MCP Lazy Loading section + new `claude mcp add` CLI syntax

## Something isn't working (skill not triggering, hook not firing, MCP not loading, etc.)
→ `troubleshooting.md` — symptom → fix mapping across CLAUDE.md, skills, hooks, MCP, subagents, context

---

*File catalog lives in [`README.md`](./README.md). Workflow: paste new tweets here → ask Claude to update the relevant file + this index.*
