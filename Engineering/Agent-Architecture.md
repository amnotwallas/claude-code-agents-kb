---
tags: [claude-code, engineering, agent-architecture, design-patterns]
type: engineering
related:
  - "[[Loop-Engineering]]"
  - "[[Graph-Engineering]]"
  - "[[Workflow-Patterns]]"
  - "[[Memory-Strategies]]"
  - "[[Six-Pillars-of-Agents]]"
created: 2026-08-02
status: stable
---

# Agent Architecture

> **Source**: notes from *AI Builders Meetup Guadalajara #1*, hosted by Wizeline (Guadalajara, 2026-08-02).

Pillar 2 of the [[Six-Pillars-of-Agents|6 pillars of agents]]. It decides the structural shape of the system: how many agents, how they communicate, how much autonomy each has — before writing any concrete prompt or loop.

## Why Architecture Gets Decided Before the Prompt

A bad prompt gets fixed by iterating. Bad architecture (a single monolithic agent trying to handle 6 different responsibilities, or 5 agents coordinating for a task one would have solved just as well) gets paid for in every future session — more latency, more cost, more failure surface. Architecture sets the quality ceiling that prompting afterward can only approach, never exceed.

## Spectrum of Architectures

```
Single-agent loop          Deterministic workflow      Multi-agent graph
(one ReAct loop,             (fixed steps, no             (several specialized
 decides everything)          reasoning between             agents, dynamic
                              steps — see                   coordination — see
                              [[Workflow-Patterns]])        [[Graph-Engineering]])
     ↑                            ↑                             ↑
  more flexible,              more predictable,            more capability,
  less predictable            less flexible                more complexity
```

There's no universally "best" architecture — it's a trade-off between flexibility, predictability, cost, and maintenance complexity. See [[Workflow-Patterns]] for when a deterministic workflow is enough and when a decision-making agent loop is needed.

## Design Dimensions

### 1. Autonomy
How much freedom does the agent have to decide its own next step? One extreme is the fixed workflow (zero autonomy, predefined sequence); the other is an agent that decides on every turn which tool to use and when to stop (see [[Loop-Engineering]]).

### 2. Responsibility Granularity
A generalist agent that does everything vs several specialized agents each with a narrow responsibility. Specialization improves per-task quality but increases coordination cost (see [[Graph-Engineering]] for multi-agent topologies).

### 3. State and Memory
Does the agent maintain state between invocations (persistent memory) or is every call independent (stateless)? See [[Memory-Strategies]] for the full breakdown of this dimension.

### 4. Tool Surface
How many and which tools the agent has available. More tools = more capability, but also more error surface (the agent picks the wrong tool) and more security risk (see [[Security]]).

### 5. Human Oversight
Where the human approval points sit in the flow — see [[Workflow-Patterns]] (Production Best Practices section) for the details of human-in-the-loop.

## Common Architectural Patterns

| Pattern | When to use it |
|---|---|
| **Single-agent with tool use** | Bounded task, one tool domain, simple iteration (see [[Loop-Engineering]]) |
| **Deterministic workflow** | Known, repeatable process, no need for adaptive per-step reasoning (see [[Workflow-Patterns]]) |
| **Orchestrator-worker** | Parallelizable sub-tasks or ones requiring different specialization (see [[Graph-Engineering]], [[Workflow-Patterns]]) |
| **Evaluator-optimizer** | Output quality benefits from a second critical pass before acceptance (see [[Workflow-Patterns]]) |

## Anti-patterns

- Designing a multi-agent architecture before confirming a single-agent loop isn't enough — coordination complexity is hard to walk back once installed
- Giving an agent full autonomy without defining a step limit or exit condition (see [[Loop-Engineering]] for flow control)
- Copying another use case's architecture without validating that its design dimensions (autonomy, granularity, state) apply to your own problem
- Not deciding on a memory strategy as part of the initial architectural design — persistence ends up improvised mid-project

## Related Notes

- [[Loop-Engineering]] — the basic unit of behavior for a single agent within the architecture
- [[Graph-Engineering]] — when the architecture requires multiple coordinated agents
- [[Workflow-Patterns]] — concrete composable patterns and the workflow-vs-agent distinction
- [[Memory-Strategies]] — the state/memory dimension of the architecture
- [[Six-Pillars-of-Agents]] — this pillar in the context of all 6 pillars
