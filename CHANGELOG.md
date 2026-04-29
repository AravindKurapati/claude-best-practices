# Changelog

## April 2026 (later)

- Added `hooks-patterns.md` — deep dive covering all ~16 hook events (vs the 4 in `claude-code-2026-update.md`), four handler types including HTTP, JSON output schema fragmentation (PreToolUse `hookSpecificOutput` vs root-level `decision` for everything else), settings precedence, 10 patterns (state injection on SessionStart, secret redaction, safer-bash with input rewriting, Stop agent verifier, etc.), and 11 pitfalls
- Updated `00-index.md` with 6 new decision entries: hooks (deterministic rules), state across compactions, scheduled tasks, auto mode, BMAD, MCP lazy loading — closing the gap the prior April changelog claimed but never delivered
- Updated `00-index.md` and `README.md` file tables to list `hooks-patterns.md` and reflect that `claude-code-2026-update.md` is no longer the only hooks reference

## April 2026

- Added `claude-code-2026-update.md` covering 10 new topics: hooks (command, prompt, agent types), auto mode, scheduled tasks (/loop + desktop + cloud), BMAD method, Skills 2.0 evals (5/6 degradation finding), MCP lazy loading + CLI install syntax, agent teams, context window management, universal workflow pattern (Research -> Plan -> Execute -> Review -> Ship), MCP ecosystem data (12K+ servers, Linux Foundation governance)
- Updated `00-index.md` with 5 new decision entries: hooks, scheduling, auto mode, BMAD, MCP lazy loading
- Updated `README.md` file table with new file

## March 2026

- Initial knowledge base published
- Added 9 core files covering CLAUDE.md, skills, MCP, agentic workflows, skill graphs, project docs structure and learning resources
- Added 4-file multi-agent framework (AGENTS.md, Agent.md, INSTRUCTIONS.md, SKILL.md)
- Added parallel agent patterns and cmux
- Added Superpowers skill pack reference
- Added self-improving skills pattern (cognee-skills)
- Added Exa MCP and GitHub MCP setup guides
- `<important if>` conditional attention pattern to claude-md-best-practices.md
- Security audit parallel agent pattern to agentic-workflow-best-practices.md
- Autoresearch practical implementation notes to skills-best-practices.md (Self-Improving Skills section)
- State + Architecture Files performance pattern: `architecture.md` and `state.md` as lazy-loaded sub-files; added to claude-md-best-practices.md, project-docs-structure.md Core Files + config tree, and 00-index.md
