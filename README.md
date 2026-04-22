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
<!-- SVG renders inline on GitHub -->
<p align="center">
<svg width="720" viewBox="0 0 720 620" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="claude-best-practices knowledge map">
<title>claude-best-practices knowledge map</title>
<desc>Visual map of repo files grouped by theme: Agentic/Claude Code (teal), Skills (orange), Infrastructure and Learning (blue), centered on the index decision guide.</desc>
<defs>
  <marker id="arr" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="5" markerHeight="5" orient="auto-start-reverse">
    <path d="M2 1L8 5L2 9" fill="none" stroke="#484f58" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
  </marker>
  <style>
    .bg { fill: #0d1117; }
    .card { fill: #161b22; stroke: #30363d; stroke-width: 1; }
    .card-center { fill: #1a1f6e; stroke: #58a6ff; stroke-width: 1.2; }
    .card-teal { fill: #0d2626; stroke: #3fb950; stroke-width: 1; }
    .card-orange { fill: #2a1800; stroke: #d29922; stroke-width: 1; }
    .card-blue { fill: #0d1f3c; stroke: #388bfd; stroke-width: 1; }
    .card-gray { fill: #161b22; stroke: #30363d; stroke-width: 1; }
    .t-center { font: 500 13px ui-monospace,SFMono-Regular,monospace; fill: #79c0ff; }
    .t-sub-center { font: 400 11px ui-monospace,SFMono-Regular,monospace; fill: #8b949e; }
    .t-teal { font: 500 12px ui-monospace,SFMono-Regular,monospace; fill: #3fb950; }
    .t-sub-teal { font: 400 10.5px ui-monospace,SFMono-Regular,monospace; fill: #7ee787; opacity: 0.75; }
    .t-orange { font: 500 12px ui-monospace,SFMono-Regular,monospace; fill: #e3b341; }
    .t-sub-orange { font: 400 10.5px ui-monospace,SFMono-Regular,monospace; fill: #f0c060; opacity: 0.75; }
    .t-blue { font: 500 12px ui-monospace,SFMono-Regular,monospace; fill: #79c0ff; }
    .t-sub-blue { font: 400 10.5px ui-monospace,SFMono-Regular,monospace; fill: #a5d6ff; opacity: 0.75; }
    .t-gray { font: 500 11px ui-monospace,SFMono-Regular,monospace; fill: #8b949e; }
    .t-sub-gray { font: 400 10px ui-monospace,SFMono-Regular,monospace; fill: #6e7681; }
    .edge { stroke: #30363d; stroke-width: 1; fill: none; }
    .dot { fill: #30363d; }
    .legend-label { font: 400 11px ui-monospace,SFMono-Regular,monospace; fill: #8b949e; }
    .badge { font: 500 10px ui-monospace,SFMono-Regular,monospace; fill: #6e7681; }
  </style>
</defs>
<!-- Background -->
<rect class="bg" x="0" y="0" width="720" height="620" rx="8"/>
<!-- Grid dots (decorative) -->
<g class="dot" opacity="0.4">
  <circle cx="40" cy="40" r="1"/><circle cx="80" cy="40" r="1"/><circle cx="120" cy="40" r="1"/>
  <circle cx="40" cy="80" r="1"/><circle cx="80" cy="80" r="1"/><circle cx="120" cy="80" r="1"/>
  <circle cx="600" cy="40" r="1"/><circle cx="640" cy="40" r="1"/><circle cx="680" cy="40" r="1"/>
  <circle cx="600" cy="80" r="1"/><circle cx="640" cy="80" r="1"/><circle cx="680" cy="80" r="1"/>
  <circle cx="40" cy="540" r="1"/><circle cx="80" cy="540" r="1"/>
  <circle cx="640" cy="540" r="1"/><circle cx="680" cy="540" r="1"/>
</g>
<!-- ── CENTER ── -->
<g>
  <rect class="card-center" x="264" y="270" width="192" height="58" rx="8"/>
  <text class="t-center" x="360" y="292" text-anchor="middle">00-index.md</text>
  <text class="t-sub-center" x="360" y="313" text-anchor="middle">decision guide · start here</text>
</g>
<!-- ── TOP: AGENTIC / CLAUDE CODE ── -->
<g>
  <rect class="card-teal" x="44" y="38" width="188" height="54" rx="6"/>
  <text class="t-teal" x="138" y="58" text-anchor="middle">agentic-workflow</text>
  <text class="t-sub-teal" x="138" y="76" text-anchor="middle">planning · agents · bugs</text>
</g>
<g>
  <rect class="card-teal" x="266" y="38" width="188" height="54" rx="6"/>
  <text class="t-teal" x="360" y="58" text-anchor="middle">claude-code-2026</text>
  <text class="t-sub-teal" x="360" y="76" text-anchor="middle">hooks · auto mode · BMAD</text>
</g>
<g>
  <rect class="card-teal" x="488" y="38" width="188" height="54" rx="6"/>
  <text class="t-teal" x="582" y="58" text-anchor="middle">claude-md-best-practices</text>
  <text class="t-sub-teal" x="582" y="76" text-anchor="middle">line limits · lazy loading</text>
</g>
<!-- ── LEFT: SKILLS ── -->
<g>
  <rect class="card-orange" x="28" y="186" width="182" height="54" rx="6"/>
  <text class="t-orange" x="119" y="206" text-anchor="middle">skills-best-practices</text>
  <text class="t-sub-orange" x="119" y="224" text-anchor="middle">authoring · skill-creator</text>
</g>
<g>
  <rect class="card-orange" x="28" y="306" width="182" height="54" rx="6"/>
  <text class="t-orange" x="119" y="326" text-anchor="middle">anthropic-skills-guide</text>
  <text class="t-sub-orange" x="119" y="344" text-anchor="middle">official 33-page distilled</text>
</g>
<g>
  <rect class="card-orange" x="28" y="426" width="182" height="54" rx="6"/>
  <text class="t-orange" x="119" y="446" text-anchor="middle">skill-graphs</text>
  <text class="t-sub-orange" x="119" y="464" text-anchor="middle">wikilinks · MOCs · depth</text>
</g>
<!-- ── RIGHT: INFRA / LEARNING ── -->
<g>
  <rect class="card-blue" x="510" y="186" width="182" height="54" rx="6"/>
  <text class="t-blue" x="601" y="206" text-anchor="middle">mcp-tools</text>
  <text class="t-sub-blue" x="601" y="224" text-anchor="middle">Exa · GitHub · setup</text>
</g>
<g>
  <rect class="card-blue" x="510" y="306" width="182" height="54" rx="6"/>
  <text class="t-blue" x="601" y="326" text-anchor="middle">project-docs-structure</text>
  <text class="t-sub-blue" x="601" y="344" text-anchor="middle">/documents/ · onboarding</text>
</g>
<g>
  <rect class="card-blue" x="510" y="426" width="182" height="54" rx="6"/>
  <text class="t-blue" x="601" y="446" text-anchor="middle">learning-resources</text>
  <text class="t-sub-blue" x="601" y="464" text-anchor="middle">books · papers · courses</text>
</g>
<!-- ── BOTTOM: META ── -->
<g>
  <rect class="card-gray" x="220" y="524" width="280" height="54" rx="6"/>
  <text class="t-gray" x="360" y="545" text-anchor="middle">AravindKurapati/claude-best-practices</text>
  <text class="t-sub-gray" x="360" y="563" text-anchor="middle">started Mar 2026 · actively updated</text>
</g>
<!-- ── EDGES: center → top ── -->
<line class="edge" x1="306" y1="270" x2="186" y2="92" marker-end="url(#arr)"/>
<line class="edge" x1="360" y1="270" x2="360" y2="92" marker-end="url(#arr)"/>
<line class="edge" x1="414" y1="270" x2="534" y2="92" marker-end="url(#arr)"/>
<!-- ── EDGES: center → left ── -->
<line class="edge" x1="264" y1="288" x2="210" y2="213" marker-end="url(#arr)"/>
<line class="edge" x1="264" y1="299" x2="210" y2="333" marker-end="url(#arr)"/>
<line class="edge" x1="264" y1="314" x2="210" y2="453" marker-end="url(#arr)"/>
<!-- ── EDGES: center → right ── -->
<line class="edge" x1="456" y1="288" x2="510" y2="213" marker-end="url(#arr)"/>
<line class="edge" x1="456" y1="299" x2="510" y2="333" marker-end="url(#arr)"/>
<line class="edge" x1="456" y1="314" x2="510" y2="453" marker-end="url(#arr)"/>
<!-- ── EDGE: center → bottom ── -->
<line class="edge" x1="360" y1="328" x2="360" y2="524" marker-end="url(#arr)"/>
<!-- ── LEGEND ── -->
<rect fill="#0d2626" stroke="#3fb950" stroke-width="0.8" x="36" y="594" width="10" height="10" rx="2"/>
<text class="legend-label" x="52" y="603">Agentic / Claude Code</text>
<rect fill="#2a1800" stroke="#d29922" stroke-width="0.8" x="210" y="594" width="10" height="10" rx="2"/>
<text class="legend-label" x="226" y="603">Skills</text>
<rect fill="#0d1f3c" stroke="#388bfd" stroke-width="0.8" x="280" y="594" width="10" height="10" rx="2"/>
<text class="legend-label" x="296" y="603">Infrastructure / Learning</text>
</svg>
</p>

## Contributing

If you've found a good practice that belongs here — a workflow, a pattern, a tool, a tip from someone in the community — please feel free to open a PR or an issue.

This repo grows through people sharing what actually works. If something in here is wrong or outdated, flag it.

---

## Status

Actively updated. Started March 2026.

Sources include: Anthropic's official guides, practitioner tweets, community blogs and personal experience building with Claude Code.
