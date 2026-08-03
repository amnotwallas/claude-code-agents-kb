---
tags: [claude-code, index]
type: index
related:
  - "[[CLAUDE-md]]"
  - "[[Spec-Driven-Development]]"
created: 2026-08-02
status: stable
---

# Claude Code KB — Index

> **Source**: notes from *AI Builders Meetup Guadalajara #1*, hosted by Wizeline (Guadalajara, 2026-08-02).

## Introduction

This is a comprehensive knowledge base on Claude Code, AI agent engineering, prompt engineering, and modern AI-assisted software development. Each note is understandable on its own, but gains more value navigated alongside the others via wiki-links.

**How to navigate**: start with the "Quick Start" section below, follow the `[[wiki-links]]` at the end of each note toward related topics, or use Obsidian's graph view to see the connections visually.

**How to contribute**: every new note must use the standard frontmatter (`tags`, `type`, `related`, `created`, `status`), include at least 3 wiki-links, at least one real code block, and an anti-patterns section.

## Vault Map

```
Claude Code KB/
├── 00 - Index.md                       — this file
├── Core/
│   ├── CLAUDE-md.md                    — persistent project memory
│   ├── Anatomy-of-a-Skill.md           — on-demand knowledge modules
│   ├── Extended-Thinking.md            — budgeted reasoning before answering
│   ├── MCP.md                          — Model Context Protocol
│   └── Six-Pillars-of-Agents.md        — 6-pillar framework, vault map
├── Engineering/
│   ├── Prompt-Engineering.md           — how to write effective prompts
│   ├── Context-Engineering.md          — context window management
│   ├── Agent-Architecture.md           — structural design of agent systems
│   ├── Memory-Strategies.md            — what persists across sessions
│   ├── Workflow-Patterns.md            — workflows vs agents, 5 composable patterns
│   ├── Harness-Engineering.md          — testing and evals for agents
│   ├── Loop-Engineering.md             — the agent cycle: perceive-reason-act
│   └── Graph-Engineering.md            — multi-agent orchestration
├── Process/
│   ├── Spec-Driven-Development.md      — spec before code
│   └── SDLC-with-AI.md                 — the software lifecycle with AI
├── Cross-Cutting/
│   ├── Observability.md                — logs, metrics, traces for agents
│   └── Security.md                     — threat model and defenses
└── Examples/
    ├── CLAUDE-md-examples.md           — 3 complete CLAUDE.md files
    ├── Skill-examples.md               — 3 complete skills
    └── SDD-examples.md                 — 2 complete specs
```

## Notes Table

| Note | Category | Purpose | Key Links |
|---|---|---|---|
| [[CLAUDE-md]] | Core | Persistent project memory | [[Context-Engineering]], [[Anatomy-of-a-Skill]] |
| [[Anatomy-of-a-Skill]] | Core | On-demand specialized knowledge | [[CLAUDE-md]], [[Harness-Engineering]] |
| [[Extended-Thinking]] | Core | Budgeted reasoning before answering | [[Prompt-Engineering]], [[Loop-Engineering]] |
| [[MCP]] | Core | Standard protocol for external tools/data | [[CLAUDE-md]], [[Security]] |
| [[Six-Pillars-of-Agents]] | Core | 6-pillar framework, synthesis note | [[Agent-Architecture]], [[Memory-Strategies]], [[Harness-Engineering]] |
| [[Prompt-Engineering]] | Engineering | Prompting fundamentals and techniques | [[Context-Engineering]], [[CLAUDE-md]] |
| [[Context-Engineering]] | Engineering | Context window management | [[Prompt-Engineering]], [[Loop-Engineering]] |
| [[Agent-Architecture]] | Engineering | Structural design of agent systems | [[Loop-Engineering]], [[Graph-Engineering]], [[Workflow-Patterns]] |
| [[Memory-Strategies]] | Engineering | What persists across calls and sessions | [[Context-Engineering]], [[CLAUDE-md]] |
| [[Workflow-Patterns]] | Engineering | Workflows vs agents, 5 composable patterns | [[Agent-Architecture]], [[Loop-Engineering]], [[Graph-Engineering]] |
| [[Loop-Engineering]] | Engineering | The agentic loop cycle | [[Graph-Engineering]], [[Harness-Engineering]] |
| [[Graph-Engineering]] | Engineering | Multi-agent systems | [[Loop-Engineering]], [[Security]] |
| [[Harness-Engineering]] | Engineering | Systematic testing and evals | [[Loop-Engineering]], [[Observability]] |
| [[Spec-Driven-Development]] | Process | Specify before implementing | [[CLAUDE-md]], [[Harness-Engineering]] |
| [[SDLC-with-AI]] | Process | SDLC transformed by AI | [[Spec-Driven-Development]], [[Security]] |
| [[Observability]] | Cross-Cutting | Logs, metrics, traces | [[Loop-Engineering]], [[Harness-Engineering]] |
| [[Security]] | Cross-Cutting | Threat model and defenses | [[Graph-Engineering]], [[SDLC-with-AI]] |
| [[CLAUDE-md-examples]] | Example | 3 real CLAUDE.md files | [[CLAUDE-md]] |
| [[Skill-examples]] | Example | 3 real skills | [[Anatomy-of-a-Skill]] |
| [[SDD-examples]] | Example | 2 real specs | [[Spec-Driven-Development]] |

## Quick Start

If you're new to this KB, read in this order:

1. **[[CLAUDE-md]]** — understand how Claude maintains project context across sessions
2. **[[Prompt-Engineering]]** — the foundation of communicating effectively with Claude
3. **[[Spec-Driven-Development]]** — the recommended process for building features with AI

## Typical Workflow

```
CLAUDE.md (persistent context)
    ↓
Spec-Driven-Development (what to build)
    ↓
Prompt Engineering (how to ask for it)
    ↓
Agent Architecture + Loop Engineering (how the agent executes it)
    ↓
Harness Engineering (how to verify it)
    ↓
Observability (how to monitor it in production)
```

## Six Pillars of Agents

A 6-pillar checklist for evaluating how solid an agent system is — see [[Six-Pillars-of-Agents]] for the full framework mapped to this vault's notes:

1. Context Engineering → [[Context-Engineering]]
2. Agent Architecture → [[Agent-Architecture]], [[Workflow-Patterns]]
3. Memory Strategies → [[Memory-Strategies]]
4. Evals → [[Harness-Engineering]]
5. AI-Harness → [[Harness-Engineering]]
6. Production Best Practices (observe, govern, scale) → [[Observability]], [[Security]]

## All Notes

- [[CLAUDE-md]]
- [[Anatomy-of-a-Skill]]
- [[Extended-Thinking]]
- [[MCP]]
- [[Six-Pillars-of-Agents]]
- [[Prompt-Engineering]]
- [[Context-Engineering]]
- [[Agent-Architecture]]
- [[Memory-Strategies]]
- [[Workflow-Patterns]]
- [[Loop-Engineering]]
- [[Graph-Engineering]]
- [[Harness-Engineering]]
- [[Spec-Driven-Development]]
- [[SDLC-with-AI]]
- [[Observability]]
- [[Security]]
- [[CLAUDE-md-examples]]
- [[Skill-examples]]
- [[SDD-examples]]
