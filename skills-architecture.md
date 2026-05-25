# Skill Graphs - Structured Knowledge for Agents

## The Problem with Single Skill Files
- One skill file = one capability (summarizing, code review, etc.)
- Fine for simple tasks, but real depth requires something else
- A therapy skill needs: CBT patterns, attachment theory, active listening, emotional regulation - can't fit in one file
- Same applies to: trading logic, legal reasoning, complex company knowledge, multi-product platforms

## What is a Skill Graph?
- A network of skill files connected with wikilinks
- Many small, composable pieces that reference each other
- Each file = one complete thought, technique, or skill
- Wikilinks between them create a traversable graph
- The agent follows relevant paths and skips what doesn't matter
- This is the difference between an agent that follows instructions and one that understands a domain

## The Key Primitives
- **Wikilinks woven into prose** - carry meaning, not just references. Agent knows *why* to follow them.
- **YAML frontmatter with descriptions** - agent scans without reading full files
- **MOCs (Maps of Content)** - organize clusters of related skills into navigable sub-topics
- **Progressive disclosure**: index -> descriptions -> links -> sections -> full content
- Most decisions happen *before* reading a single full file

## Index File - The Entry Point
The index isn't a lookup table. It's an entry point that points attention.

```markdown
# knowledge-work

[Brief description of the domain]

## Synthesis
- [[the-core-argument]] - one-line description of what this node covers
- [[architecture-overview]] - the foundational structure and why it works

## Topic MOCs
- [[graph-structure]] - how wiki links and topology create traversable graphs
- [[agent-cognition]] - how agents think through external structures
  - [[agent-cognition-hooks]] - hook enforcement and cognitive science
- [[discovery-retrieval]] - how descriptions and progressive disclosure enable finding content

## Cross-Domain Claims
- [[forced-engagement-produces-weak-connections]] - the social analog of shallow knowledge

## Explorations Needed
- Missing: comparison between human and agent traversal patterns
- Scaling limits: at what system size does human curation fail?
```

## Individual Node Structure
Each node is a standalone methodology claim. Wikilinks inside the prose carry context:

```markdown
---
description: How wiki links create small-world topology in knowledge graphs.
  Use when agent needs to understand graph traversal or link structure.
---

# Graph Structure

Knowledge graphs work because [[small-world-topology]] allows any two concepts 
to be connected in a few hops. This is why [[wikilinks-in-prose]] matter more 
than bare reference lists - the surrounding text tells the agent when and why 
to follow the link.

See also: [[discovery-retrieval]] for how agents find entry points.
```

## What Skill Graphs Enable
- **Trading**: risk management -> market psychology -> position sizing -> technical analysis, each linked
- **Legal**: contract patterns -> compliance -> jurisdiction specifics -> precedent chains
- **Company**: org structure -> product knowledge -> processes -> onboarding -> culture -> competitive landscape
- **Therapy**: CBT patterns -> attachment theory -> active listening -> emotional regulation frameworks
- None of these fit in one file. All of them work as graphs.

## How to Build One

### Easy way - arscontexta plugin
- Install `arscontexta` Claude Code plugin
- Pick the research preset, point it at a topic
- Sets up the markdown folder structure, then fill with `/learn` and `/reduce`
- 249 connected markdown files teach the agent how to build knowledge bases

### Manual way
- Doesn't need to live in `.claude/skills/` - just needs an index file
- Index tells the agent what exists and how to traverse it
- Each node links to others; graph goes as deep as the domain requires
- Simpler than it sounds - start with 5-10 nodes and grow it

## When to Use a Skill Graph vs a Single Skill
Use a single skill when: one file can hold the full workflow (< 5,000 words)
Use a skill graph when:
- Domain has multiple interconnected sub-topics
- Different tasks need different subsets of the knowledge
- You keep hitting the context limit on your skill file
- The domain requires depth that would bloat a single file

## The Evolution
- Skills = context engineering (curated knowledge injected where it matters)
- Skill graphs = next step: agent navigates a knowledge structure, pulling in exactly what the situation requires
- Stack skills over time - "it might just change your life"

