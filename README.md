# Claude & Claude Code Best Practices

A living knowledge base of best practices for working with Claude, Claude Code, skills, MCP tools, and agentic workflows that I PERSONALLY collected from practitioners, researchers and the community.

This is different from ecosystem trackers or reference indexes. Everything here is there for people who are learning, not people who already know the landscape.

---

## What This Is

This repo is a personal knowledge base I've been building while learning Claude Code and agentic workflows. Everything here comes from real practitioners sharing what actually works.... Twitter threads, blog posts, Anthropic's official guides and my own experience.

It's not a tutorial. It's a **decision guide** organized so you can find the right practice for the situation you're in right now.

I'm keeping this updated as I learn more. If you've found something that works, contributions are welcome.

---

## How to Use This

**Start with [`00-index.md`](./00-index.md)** — it maps every common situation to the right file and section. Don't read everything at once. Find your situation, go to the relevant file.

**With Claude Code:** point Claude at this repo before starting work:
> "Read my knowledge base at github.com/AravindKurapati/claude-best-practices and use it to inform how we work together."

**With claude.ai Projects:** upload the `.md` files to your Project Knowledge so Claude can reference them in chat.

---

## Files

| File | What's Inside |
|------|--------------|
| [`00-index.md`](./00-index.md) | Decision guide. Start here |
| [`claude-md-best-practices.md`](./claude-md-best-practices.md) | CLAUDE.md line limits, lazy loading, rewrite cadence |
| [`agentic-workflow-best-practices.md`](./agentic-workflow-best-practices.md) | Planning, parallel agents, bug handling, multi-agent structure |
| [`skills-best-practices.md`](./skills-best-practices.md) | Skill authoring, skill-creator, self-improving skills |
| [`anthropic-skills-guide.md`](./anthropic-skills-guide.md) | Anthropic's official 33-page guide distilled |
| [`project-docs-structure.md`](./project-docs-structure.md) | /documents/ folder, onboarding existing projects |
| [`skill-graphs.md`](./skill-graphs.md) | Wikilink networks, MOCs, when single skill files aren't enough |
| [`mcp-tools.md`](./mcp-tools.md) | MCP server setup, Exa, GitHub, and more |
| [`learning-resources.md`](./learning-resources.md) | Books, papers, videos, courses, newsletters |
| [`claude-code-2026-update.md`](./claude-code-2026-update.md) | **NEW** - Hooks, Auto Mode, scheduled tasks, BMAD, Skills 2.0 evals, MCP lazy loading, agent teams, context management |

---
Knowledge Map
<p align="center">
<svg width="720" viewBox="0 0 720 640" xmlns="http://www.w3.org/2000/svg">
<!-- Background -->
<rect x="0" y="0" width="720" height="640" rx="8" fill="#0d1117"/>
<!-- Grid dots -->
<circle cx="40" cy="40" r="1" fill="#21262d"/><circle cx="80" cy="40" r="1" fill="#21262d"/><circle cx="120" cy="40" r="1" fill="#21262d"/>
<circle cx="40" cy="80" r="1" fill="#21262d"/><circle cx="80" cy="80" r="1" fill="#21262d"/><circle cx="120" cy="80" r="1" fill="#21262d"/>
<circle cx="600" cy="40" r="1" fill="#21262d"/><circle cx="640" cy="40" r="1" fill="#21262d"/><circle cx="680" cy="40" r="1" fill="#21262d"/>
<circle cx="600" cy="80" r="1" fill="#21262d"/><circle cx="640" cy="80" r="1" fill="#21262d"/><circle cx="680" cy="80" r="1" fill="#21262d"/>
<circle cx="40" cy="560" r="1" fill="#21262d"/><circle cx="80" cy="560" r="1" fill="#21262d"/>
<circle cx="640" cy="560" r="1" fill="#21262d"/><circle cx="680" cy="560" r="1" fill="#21262d"/>
<!-- CENTER NODE -->
<rect x="264" y="272" width="192" height="58" rx="8" fill="#161b6e" stroke="#58a6ff" stroke-width="1.2"/>
<text x="360" y="295" text-anchor="middle" font-family="ui-monospace,SFMono-Regular,monospace" font-size="13" font-weight="500" fill="#79c0ff">00-index.md</text>
<text x="360" y="316" text-anchor="middle" font-family="ui-monospace,SFMono-Regular,monospace" font-size="11" fill="#8b949e">decision guide · start here</text>
<!-- TOP: AGENTIC / CLAUDE CODE -->
<rect x="44" y="40" width="188" height="54" rx="6" fill="#0d2626" stroke="#3fb950" stroke-width="1"/>
<text x="138" y="62" text-anchor="middle" font-family="ui-monospace,SFMono-Regular,monospace" font-size="12" font-weight="500" fill="#3fb950">agentic-workflow</text>
<text x="138" y="80" text-anchor="middle" font-family="ui-monospace,SFMono-Regular,monospace" font-size="10" fill="#7ee787">planning · agents · bugs</text>
<rect x="266" y="40" width="188" height="54" rx="6" fill="#0d2626" stroke="#3fb950" stroke-width="1"/>
<text x="360" y="62" text-anchor="middle" font-family="ui-monospace,SFMono-Regular,monospace" font-size="12" font-weight="500" fill="#3fb950">claude-code-2026</text>
<text x="360" y="80" text-anchor="middle" font-family="ui-monospace,SFMono-Regular,monospace" font-size="10" fill="#7ee787">hooks · auto mode · BMAD</text>
<rect x="488" y="40" width="188" height="54" rx="6" fill="#0d2626" stroke="#3fb950" stroke-width="1"/>
<text x="582" y="62" text-anchor="middle" font-family="ui-monospace,SFMono-Regular,monospace" font-size="11" font-weight="500" fill="#3fb950">claude-md-best-practices</text>
<text x="582" y="80" text-anchor="middle" font-family="ui-monospace,SFMono-Regular,monospace" font-size="10" fill="#7ee787">line limits · lazy loading</text>
<!-- LEFT: SKILLS -->
<rect x="28" y="188" width="182" height="54" rx="6" fill="#2a1800" stroke="#d29922" stroke-width="1"/>
<text x="119" y="210" text-anchor="middle" font-family="ui-monospace,SFMono-Regular,monospace" font-size="12" font-weight="500" fill="#e3b341">skills-best-practices</text>
<text x="119" y="228" text-anchor="middle" font-family="ui-monospace,SFMono-Regular,monospace" font-size="10" fill="#f0c060">authoring · skill-creator</text>
<rect x="28" y="308" width="182" height="54" rx="6" fill="#2a1800" stroke="#d29922" stroke-width="1"/>
<text x="119" y="330" text-anchor="middle" font-family="ui-monospace,SFMono-Regular,monospace" font-size="12" font-weight="500" fill="#e3b341">anthropic-skills-guide</text>
<text x="119" y="348" text-anchor="middle" font-family="ui-monospace,SFMono-Regular,monospace" font-size="10" fill="#f0c060">official 33-page distilled</text>
<rect x="28" y="428" width="182" height="54" rx="6" fill="#2a1800" stroke="#d29922" stroke-width="1"/>
<text x="119" y="450" text-anchor="middle" font-family="ui-monospace,SFMono-Regular,monospace" font-size="12" font-weight="500" fill="#e3b341">skill-graphs</text>
<text x="119" y="468" text-anchor="middle" font-family="ui-monospace,SFMono-Regular,monospace" font-size="10" fill="#f0c060">wikilinks · MOCs · depth</text>
<!-- RIGHT: INFRA / LEARNING -->
<rect x="510" y="188" width="182" height="54" rx="6" fill="#0d1f3c" stroke="#388bfd" stroke-width="1"/>
<text x="601" y="210" text-anchor="middle" font-family="ui-monospace,SFMono-Regular,monospace" font-size="12" font-weight="500" fill="#79c0ff">mcp-tools</text>
<text x="601" y="228" text-anchor="middle" font-family="ui-monospace,SFMono-Regular,monospace" font-size="10" fill="#a5d6ff">Exa · GitHub · setup</text>
<rect x="510" y="308" width="182" height="54" rx="6" fill="#0d1f3c" stroke="#388bfd" stroke-width="1"/>
<text x="601" y="330" text-anchor="middle" font-family="ui-monospace,SFMono-Regular,monospace" font-size="11" font-weight="500" fill="#79c0ff">project-docs-structure</text>
<text x="601" y="348" text-anchor="middle" font-family="ui-monospace,SFMono-Regular,monospace" font-size="10" fill="#a5d6ff">/documents/ · onboarding</text>
<rect x="510" y="428" width="182" height="54" rx="6" fill="#0d1f3c" stroke="#388bfd" stroke-width="1"/>
<text x="601" y="450" text-anchor="middle" font-family="ui-monospace,SFMono-Regular,monospace" font-size="12" font-weight="500" fill="#79c0ff">learning-resources</text>
<text x="601" y="468" text-anchor="middle" font-family="ui-monospace,SFMono-Regular,monospace" font-size="10" fill="#a5d6ff">books · papers · courses</text>
<!-- BOTTOM: META -->
<rect x="200" y="528" width="320" height="54" rx="6" fill="#161b22" stroke="#30363d" stroke-width="1"/>
<text x="360" y="550" text-anchor="middle" font-family="ui-monospace,SFMono-Regular,monospace" font-size="11" font-weight="500" fill="#8b949e">AravindKurapati/claude-best-practices</text>
<text x="360" y="568" text-anchor="middle" font-family="ui-monospace,SFMono-Regular,monospace" font-size="10" fill="#6e7681">started Mar 2026 · actively updated</text>
<!-- EDGES: center to top -->
<line x1="306" y1="272" x2="186" y2="94" stroke="#30363d" stroke-width="1"/>
<line x1="360" y1="272" x2="360" y2="94" stroke="#30363d" stroke-width="1"/>
<line x1="414" y1="272" x2="534" y2="94" stroke="#30363d" stroke-width="1"/>
<!-- EDGES: center to left -->
<line x1="264" y1="290" x2="210" y2="215" stroke="#30363d" stroke-width="1"/>
<line x1="264" y1="301" x2="210" y2="335" stroke="#30363d" stroke-width="1"/>
<line x1="264" y1="316" x2="210" y2="455" stroke="#30363d" stroke-width="1"/>
<!-- EDGES: center to right -->
<line x1="456" y1="290" x2="510" y2="215" stroke="#30363d" stroke-width="1"/>
<line x1="456" y1="301" x2="510" y2="335" stroke="#30363d" stroke-width="1"/>
<line x1="456" y1="316" x2="510" y2="455" stroke="#30363d" stroke-width="1"/>
<!-- EDGE: center to bottom -->
<line x1="360" y1="330" x2="360" y2="528" stroke="#30363d" stroke-width="1"/>
<!-- LEGEND -->
<rect x="36" y="606" width="10" height="10" rx="2" fill="#0d2626" stroke="#3fb950" stroke-width="0.8"/>
<text x="52" y="615" font-family="ui-monospace,SFMono-Regular,monospace" font-size="10" fill="#8b949e">Agentic / Claude Code</text>
<rect x="210" y="606" width="10" height="10" rx="2" fill="#2a1800" stroke="#d29922" stroke-width="0.8"/>
<text x="226" y="615" font-family="ui-monospace,SFMono-Regular,monospace" font-size="10" fill="#8b949e">Skills</text>
<rect x="286" y="606" width="10" height="10" rx="2" fill="#0d1f3c" stroke="#388bfd" stroke-width="0.8"/>
<text x="302" y="615" font-family="ui-monospace,SFMono-Regular,monospace" font-size="10" fill="#8b949e">Infrastructure / Learning</text>
</svg>
</p>
## Contributing

If you've found a good practice that belongs here — a workflow, a pattern, a tool, a tip from someone in the community — please feel free to open a PR or an issue.

This repo grows through people sharing what actually works. If something in here is wrong or outdated, flag it.

---

## Status

Actively updated. Started March 2026.

Sources include: Anthropic's official guides, practitioner tweets, community blogs and personal experience building with Claude Code.
