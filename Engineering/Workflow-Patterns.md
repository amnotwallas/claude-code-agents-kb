---
tags: [claude-code, engineering, workflows, agents, design-patterns]
type: engineering
related:
  - "[[Agent-Architecture]]"
  - "[[Loop-Engineering]]"
  - "[[Graph-Engineering]]"
  - "[[Prompt-Engineering]]"
  - "[[Six-Pillars-of-Agents]]"
created: 2026-08-02
status: stable
---

# Workflow Patterns

> **Source**: notes from *AI Builders Meetup Guadalajara #1*, hosted by Wizeline (Guadalajara, 2026-08-02).

Extends [[Agent-Architecture]]: how to choose between a deterministic workflow and an autonomous agent, and the 5 composable patterns that cover most real use cases without needing an agent with an open-ended decision loop.

## Workflows vs Agents

| | Workflow | Agent |
|---|---|---|
| **Control flow** | Predefined by code — fixed steps in a known order | The LLM dynamically decides the next step |
| **Predictability** | High — same input, same execution path | Lower — the path varies with the model's reasoning |
| **When to use** | The process is known upfront and doesn't need step-by-step adaptation | The task requires the model to decide what to do based on what it discovers |
| **Cost/latency** | Predictable, generally lower | Variable, generally higher (more calls, more reasoning tokens) |
| **Example** | Content moderation pipeline: always classify → filter → respond | Support agent that decides which tools to use based on the user's query |

**Practical rule**: start with the simplest workflow that solves the problem. Move up in autonomy (workflow → agent) only when the real task's variability demands it — not by default, not "because agents are the future." A well-designed workflow is cheaper, easier to debug, and easier to test (see [[Harness-Engineering]]) than an equivalent agent.

## Five Composable Workflow Patterns

### 1. Prompt Chaining
Breaks a task into fixed sequential steps, where each step's output feeds the next step's input. Each step is a model call focused on one sub-task.

```
Input → [Step 1: extract data] → [Step 2: validate] → [Step 3: format] → Output
```

**When to use**: the task naturally decomposes into sub-steps where each one benefits from a focused prompt, rather than asking for everything in one complex call.

```python
def chain_generate_report(raw_notes: str) -> str:
    extracted = call_llm(EXTRACT_PROMPT, raw_notes)
    validated = call_llm(VALIDATE_PROMPT, extracted)
    formatted = call_llm(FORMAT_PROMPT, validated)
    return formatted
```

### 2. Routing
Classifies the input and directs it to one of several specialized paths, each optimized for its case type.

```
Input → [Classifier] ──┬──→ Path A (billing)
                        ├──→ Path B (technical support)
                        └──→ Path C (account/access)
```

**When to use**: inputs fall into clearly distinct categories that benefit from different prompts or even different models (e.g., a small, cheap model for simple cases, a large model for complex ones — model routing, see [[Observability]] for the cost impact).

### 3. Parallelization
Runs several independent sub-tasks at the same time and combines the results — same principle as the Parallel Fan-out in [[Graph-Engineering]], applied as a workflow pattern.

```
              ┌→ Sub-task A ─┐
Input ────────┼→ Sub-task B ─┼──→ Merge → Output
              └→ Sub-task C ─┘
```

**When to use**: the sub-tasks are genuinely independent of each other (none needs another's result) and total time matters — e.g., analyzing 3 different aspects of the same document in parallel.

### 4. Orchestrator-Worker
A central orchestrator breaks down the task and delegates sub-tasks to specialized workers, then integrates their results. Unlike Parallelization, here the orchestrator dynamically decides which sub-tasks to generate (they aren't fixed upfront).

```
Input → Orchestrator → [decides which sub-tasks to generate]
              ↓
      [Worker 1, Worker 2, ..., Worker N] → results → Orchestrator → Output
```

**When to use**: the number and nature of sub-tasks isn't known upfront and depends on the specific input — see [[Graph-Engineering]] for the full implementation of this topology at the multi-agent level.

### 5. Evaluator-Optimizer
A generator produces a result, an evaluator critiques it against explicit criteria, and the generator iterates until the evaluator approves or an attempt limit is reached.

```
Generator → Output → Evaluator ──┬──→ Approved → Final output
                ↑                 └──→ Rejected (with feedback)
                └─────────────────────────┘
```

**When to use**: there's a clear, verifiable quality criterion, and the model's first pass typically doesn't meet it without review (e.g., code generation that must pass tests, translation that must preserve tone).

```python
MAX_ATTEMPTS = 3

def evaluator_optimizer(task: str) -> str:
    output = call_llm(GENERATE_PROMPT, task)
    for attempt in range(MAX_ATTEMPTS):
        evaluation = call_llm(EVALUATE_PROMPT, output)
        if evaluation.approved:
            return output
        output = call_llm(REFINE_PROMPT, task, output, evaluation.feedback)
    return output  # last version, even if not approved
```

## Combining Patterns

The 5 patterns are **composable**: a real system frequently combines several — e.g., Routing to direct the input to the right path, followed by Prompt Chaining within that path, with a final Evaluator-Optimizer step before delivering the result. They aren't mutually exclusive or hierarchical relative to each other.

## Agent Loops: When Workflows Aren't Enough

When none of the 5 fixed patterns covers the task because the execution path genuinely can't be predefined — the next step truly depends on what the model discovers in the previous one — an **agent loop** is needed: the model decides on every turn which tool to use and when to stop (see [[Loop-Engineering]] for the full anatomy of the ReAct loop and flow control).

The transition from workflow to agent isn't binary — it's a spectrum (see [[Agent-Architecture]]). Many production systems are workflows with a single "agentic" step in the middle (e.g., Routing followed by an agent loop only within the most complex path), not pure end-to-end agents.

## Complex Prompt vs Simple Prompt

Related but distinct from choosing workflow vs agent: within a single step/call, should the prompt be simple (one focused instruction) or complex (multiple instructions, extensive context, several few-shot examples)?

| | Simple Prompt | Complex Prompt |
|---|---|---|
| **When** | Task with a single clear objective, no relevant ambiguity | Task with multiple simultaneous requirements, strict format, or high risk of misinterpretation |
| **Advantage** | Fast, cheap, easy to debug when it fails | Covers more edge cases in a single call |
| **Risk** | Fails on cases with nuances not covered | Instructions stepping on each other, or the model ignoring the ones further down (see [[Context-Engineering]]) |
| **Relation to patterns** | Fits well as one step in Prompt Chaining (each step simple) | Fits when splitting into Chaining isn't practical due to strong dependencies between requirements |

**Practical rule**: prefer several simple prompts chained together (Prompt Chaining) over one complex prompt trying to cover everything — easier to test each step individually (see [[Harness-Engineering]]), and each individual step is less prone to the model ignoring part of the instruction.

## Anti-patterns

- Using a full agent loop for a task that a 2-step Prompt Chaining solves just as well, paying unnecessary latency and cost
- One complex prompt with 10 simultaneous instructions when 3 chained simple prompts would be more reliable and easier to debug
- Choosing Orchestrator-Worker when the sub-tasks are actually fixed and known upfront — that's simply Parallelization with unnecessary extra coordination steps
- Not defining the Evaluator-Optimizer's approval criterion explicitly and verifiably — a vague evaluator produces refinement cycles that never converge

## Related Notes

- [[Agent-Architecture]] — the workflow-to-agent spectrum as an architectural design dimension
- [[Loop-Engineering]] — the full anatomy of the agent loop when fixed workflows aren't enough
- [[Graph-Engineering]] — Orchestrator-Worker and Parallelization taken to full multi-agent topologies
- [[Prompt-Engineering]] — how to structure each individual prompt within any of these patterns
- [[Six-Pillars-of-Agents]] — this content within the Agent Architecture pillar
