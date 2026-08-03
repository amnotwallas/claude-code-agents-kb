# Claude Code & AI Agents KB

A cross-linked knowledge base on **Claude Code**, **AI agent engineering**, and **AI-assisted software development** — built as an [Obsidian](https://obsidian.md) vault, fully readable as plain Markdown on GitHub.

21 notes covering project memory (`CLAUDE.md`), skills, MCP, extended thinking, the six pillars of agent systems (context engineering, agent architecture, memory strategies, evals, AI harnesses, production best practices), workflow patterns, spec-driven development, observability, and security — each note understandable on its own, but more useful navigated alongside the others via `[[wiki-links]]`.

## Source

Several of these notes synthesize and expand on topics covered at **AI Builders Meetup Guadalajara #1**, hosted by [Wizeline](https://wizeline.com) (Guadalajara, August 2026) — talks *"Building Better with Claude Code"* and *"Building AI Agents that Matter."* Each note that draws on that event carries a `> **Source**` attribution line. This is an independent set of notes, not an official Wizeline publication.

## Structure

```
claude-code-agents-kb/
├── 00 - Index.md                       — start here
├── Core/
│   ├── CLAUDE-md.md                    — persistent project memory
│   ├── Anatomy-of-a-Skill.md           — on-demand knowledge modules
│   ├── Extended-Thinking.md            — budgeted reasoning before answering
│   ├── MCP.md                          — Model Context Protocol
│   └── Six-Pillars-of-Agents.md        — 6-pillar framework, vault map
├── Engineering/
│   ├── Prompt-Engineering.md
│   ├── Context-Engineering.md
│   ├── Agent-Architecture.md
│   ├── Memory-Strategies.md
│   ├── Workflow-Patterns.md            — workflows vs agents, 5 composable patterns
│   ├── Loop-Engineering.md             — the agentic loop
│   ├── Graph-Engineering.md            — multi-agent orchestration
│   └── Harness-Engineering.md          — testing and evals for agents
├── Process/
│   ├── Spec-Driven-Development.md
│   └── SDLC-with-AI.md
├── Cross-Cutting/
│   ├── Observability.md
│   └── Security.md
└── Examples/
    ├── CLAUDE-md-examples.md           — 3 complete CLAUDE.md files
    ├── Skill-examples.md               — 3 complete skills
    └── SDD-examples.md                 — 2 complete specs
```

## How to Use

**In Obsidian**: open this folder as a vault. `[[wiki-links]]` are clickable, the graph view shows how notes connect, and `00 - Index.md` is the entry point.

**On GitHub**: every note is plain Markdown and readable directly — `[[wiki-links]]` render as literal bracketed text rather than clickable links (GitHub doesn't resolve Obsidian's link syntax), but headers, tables, and code blocks all render normally.

Start with **[[00 - Index]]**, or read in this order:
1. `Core/CLAUDE-md.md`
2. `Engineering/Prompt-Engineering.md`
3. `Process/Spec-Driven-Development.md`

## Conventions

Every note follows the same shape: YAML frontmatter (`tags`, `type`, `related`, `created`, `status`), at least 3 wiki-links to related notes, real functional code examples (no pseudocode), and an anti-patterns section.

## License

Content shared as-is for reference and reuse. No official affiliation with Anthropic or Wizeline.
