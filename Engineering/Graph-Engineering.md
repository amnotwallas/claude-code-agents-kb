---
tags: [claude-code, engineering, multi-agent, graphs, orchestration]
type: engineering
related:
  - "[[Loop-Engineering]]"
  - "[[Context-Engineering]]"
  - "[[Observability]]"
  - "[[Security]]"
  - "[[Agent-Architecture]]"
  - "[[Workflow-Patterns]]"
created: 2026-08-02
status: stable
---

# Graph Engineering

> **Source**: notes from *AI Builders Meetup Guadalajara #1*, hosted by Wizeline (Guadalajara, 2026-08-02).

See also [[Agent-Architecture]] for where a multi-agent graph fits within the full spectrum of architectures, and [[Workflow-Patterns]] for the Orchestrator-Worker and Parallelization patterns as simpler workflow-level versions of this (no full graph required).

## What Is an Agent Graph?

A system where multiple specialized agents collaborate, each responsible for part of the workflow — as opposed to a single agent with a generic [[Loop-Engineering|loop]] trying to do everything.

**Why**: some problems benefit from parallelism (several independent tasks at once), specialization (an agent expert in security reasons differently than one expert in UI), or hierarchical supervision (an orchestrator validates workers' output before accepting it).

**When NOT to use a graph**: for tasks a single agent with good context can solve sequentially just as well. A graph adds coordination complexity (state serialization, inter-agent communication, partial-failure handling) — it's only worth it when that complexity is paid off by real parallelism or specialization that improves result quality.

## Key Concepts

- **Node**: an individual agent with a specific role (Planner, Coder, Reviewer, Tester)
- **Edge**: the communication channel between nodes — what information flows and in which direction
- **Orchestrator**: the agent that coordinates the others. Not necessarily the most capable one in the system, but the most strategic — its job is deciding which worker to use and when, not doing the underlying work itself
- **Worker**: a specialized agent that executes specific tasks within its domain
- **Handoff**: the transfer of control and context from one agent to another — includes what information carries over and what gets dropped

## Graph Topologies

### 1. Linear Pipeline
```
A → B → C → D
```
Each agent transforms the previous one's output. Simple, predictable, but no parallelism — total time is the sum of each stage.

### 2. Parallel Fan-out
```
              ┌→ Worker A ─┐
Orchestrator ─┼→ Worker B ─┼→ Merge
              └→ Worker C ─┘
```
Several workers work in parallel on independent parts of the problem, and their results get combined at the end. Reduces total time when sub-tasks are genuinely independent.

### 3. Hierarchical (Supervisor)
```
Supervisor → Planner → [Worker 1, Worker 2, ...] → Reviewer → Supervisor
```
A supervisor delegates to a planner that breaks down the task, workers execute the parts, and a reviewer validates before the supervisor considers the task complete.

### 4. Peer-to-peer
```
Agent A ⇄ Agent B ⇄ Agent C
```
Agents that consult each other without a fixed hierarchy — any of them can initiate communication with any other. More flexible, but harder to reason about the flow and to debug.

## Communication Between Agents

| Pattern | Description | When to use |
|---|---|---|
| **Shared state** | All agents read/write a shared state object | When several agents need visibility into the same context at all times |
| **Message passing** | Agents send each other structured messages (events) | When communication is point-to-point and coupling must be minimal |
| **Blackboard** | A shared board where each agent publishes and reads results | When contribution order isn't fixed and several agents can contribute to the same problem |

**Pros/cons**:
- Shared state is simple to implement but prone to race conditions with concurrent writes
- Message passing scales better and isolates failures, but requires explicit message contracts
- Blackboard is flexible for exploratory problems, but hard to trace which agent caused which effect

## Claude as Orchestrator

To instruct Claude to coordinate sub-agents, the orchestrator's prompt should:
1. List the available workers and their specialty
2. Define the criteria for when to delegate to each
3. Specify the handoff format (what context passes to each worker)
4. Define how results get combined/validated before reporting as complete

```
You are the orchestrator for a code review system. You have available:
- research-agent: investigates the purpose and historical context of
  undocumented legacy code
- coding-agent: implements code changes
- qa-agent: verifies that changes don't break existing tests

For each task:
1. If it requires understanding undocumented legacy code, delegate to
   research-agent first and use its output as context for coding-agent
2. All implementation goes through coding-agent
3. Every change from coding-agent is validated with qa-agent before
   being reported as complete
4. If qa-agent reports failures, go back to coding-agent with the
   failure details, max 3 cycles before escalating to human review
```

## Complete Example: Multi-Agent Code Review

System: **Fetcher** (gets the PR) → **Static Analyzer** → **Security Scanner** → **Logic Reviewer** → **Report Generator**.

```python
from dataclasses import dataclass, field

@dataclass
class ReviewState:
    pr_diff: str = ""
    static_issues: list[str] = field(default_factory=list)
    security_issues: list[str] = field(default_factory=list)
    logic_issues: list[str] = field(default_factory=list)

def fetcher_node(pr_url: str, state: ReviewState) -> ReviewState:
    state.pr_diff = fetch_pr_diff(pr_url)
    return state

def static_analyzer_node(state: ReviewState) -> ReviewState:
    # Specialized worker: linting, complexity, dead code
    state.static_issues = run_static_analysis(state.pr_diff)
    return state

def security_scanner_node(state: ReviewState) -> ReviewState:
    # Specialized worker: secrets, SQL injection, vulnerable deps
    state.security_issues = scan_for_vulnerabilities(state.pr_diff)
    return state

def logic_reviewer_node(state: ReviewState) -> ReviewState:
    # Specialized worker: change correctness, edge cases
    state.logic_issues = review_logic(state.pr_diff)
    return state

def report_generator_node(state: ReviewState) -> str:
    # Merge: combines all workers' output into a single report
    return build_report(
        static=state.static_issues,
        security=state.security_issues,
        logic=state.logic_issues,
    )

def run_review_pipeline(pr_url: str) -> str:
    state = ReviewState()
    state = fetcher_node(pr_url, state)

    # Fan-out: static + security run in parallel (independent of each other)
    state = static_analyzer_node(state)
    state = security_scanner_node(state)

    # Logic reviewer depends on the diff but not on the other two — could
    # be parallelized too, shown sequential here for clarity
    state = logic_reviewer_node(state)

    return report_generator_node(state)
```

In production, `static_analyzer_node` and `security_scanner_node` would actually run in parallel (e.g., with `asyncio.gather`), since they don't depend on each other's output — only on the initial `pr_diff`.

## Anti-patterns

- Using a multi-agent graph for a simple sequential task that a single [[Loop-Engineering|loop]] solves just as well, paying for coordination complexity without benefit
- An orchestrator that also does the underlying work — mixes responsibilities and loses the advantage of specialization
- Shared state without concurrency control when there's real parallel writing
- Not defining a cycle limit between agents that feed back into each other (worker → reviewer → worker → ...) — risk of an infinite loop between nodes, same as in a single-agent loop (see [[Loop-Engineering]])

## Related Notes

- [[Loop-Engineering]] — the single-agent loop is the building block of each node in the graph
- [[Context-Engineering]] — what context gets passed in each handoff between agents
- [[Observability]] — tracing multi-agent flows requires distributed tracing
- [[Security]] — a multi-agent graph widens the attack surface (more entry points, more handoffs that can be poisoned)
