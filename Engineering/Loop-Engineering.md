---
tags: [claude-code, engineering, agents, agentic-loop, tools]
type: engineering
related:
  - "[[Graph-Engineering]]"
  - "[[Context-Engineering]]"
  - "[[Harness-Engineering]]"
  - "[[Observability]]"
  - "[[Security]]"
created: 2026-08-02
status: stable
---

# Loop Engineering

> **Source**: notes from *AI Builders Meetup Guadalajara #1*, hosted by Wizeline (Guadalajara, 2026-08-02).

## What Is an Agentic Loop?

The fundamental cycle of an AI agent: the agent **perceives** the current state → **reasons** about what to do → **acts** by executing a tool → **observes** the result → repeats until the task is complete.

It exists because complex tasks require multiple steps and decisions that depend on intermediate results — they can't be solved with a single model call.

The key difference between an LLM call and a loop: **the loop has state and feedback**. A single call produces an output without seeing the effect of its own actions; a loop observes the result of each action and adjusts the next step accordingly.

## Anatomy of the Loop

```
┌─────────────────────────────────────────────────────┐
│                    AGENTIC LOOP                      │
│                                                       │
│  Input → [Perception] → [Reasoning] → [Action]        │
│                ↑                            │         │
│                └──── [Observation] ◄────────┘         │
│                                                       │
│  Exit condition: task complete / error /               │
│  step limit reached / human approval required          │
└─────────────────────────────────────────────────────┘
```

## Types of Loops

### ReAct Loop (Reason + Act)
The most common. Claude explicitly reasons before each action ("I need to see file X to understand Y"), executes the tool, observes the result, and repeats. Ideal for exploratory tasks where the next step depends on what gets discovered.

### Plan-then-Execute
Claude plans all the steps first (a complete list of actions), then executes them sequentially without re-planning at each step. More token-efficient for well-defined tasks, but less adaptive if an intermediate step reveals information that invalidates the original plan.

### Reflexion Loop
Claude executes an action, then self-critiques the result ("did this actually solve the problem?") and improves before continuing. Useful when the first pass tends to have subtle errors that a second review step catches.

### Tool-Use Loop
A loop centered specifically on the use of external tools (bash, APIs, file system) — this is the concrete shape the ReAct loop takes in Claude Code.

## Flow Control: When to End the Loop

- **Success condition**: the task was completed — detected by checking the final state against the goal (tests pass, file exists with expected content)
- **Step limit**: maximum iterations to avoid infinite loops from recurring errors or poor reasoning
- **Unrecoverable error**: the agent detects it can't continue (permission denied, resource doesn't exist) and should report it instead of retrying indefinitely
- **Human-in-the-loop**: pause and request human approval before a high-risk action (see next section)
- **Cost budget**: spending limit (tokens or money) per session, to bound the cost of long or poorly-behaved loops

## Human-in-the-Loop (HITL): When and How

Pause for human approval when:
- The action is **irreversible** (delete, deploy to production, sending emails/messages to third parties)
- The **agent's confidence is low** (multiple failed attempts, unresolved ambiguity)
- The **stakes are high** (affects production data, shared systems, real users)

Implementation patterns:
- **Approval gates**: the loop stops before the risky action and waits for explicit confirmation
- **Parallel review**: the action runs in an isolated environment (sandbox/staging) and a human reviews the result before promoting it to production

## Loop State and Persistence

- Track progress in long-running loops via an explicit task list (what's done, what's pending) rather than relying only on conversation history
- Periodically serialize state to be able to recover from failures without restarting from zero
- **Checkpointing**: save state every N steps, not only at the end — so an interruption mid-way doesn't lose all progress (see [[Context-Engineering]] for context-specific checkpointing)

## Error Handling in Loops

- **Retry logic with exponential backoff**: for transient errors (network timeouts, rate limits) — wait progressively longer between successive retries
- **Distinguish recoverable vs unrecoverable errors**: a network timeout is recoverable via retry; a permission denied or a missing file generally isn't, and blindly retrying wastes steps
- **Graceful degradation**: when a tool fails, can the loop continue with reduced capability (e.g., without that information source) or must it stop and report?

## Complete Example

A Python loop for a coding agent that: reads an issue → analyzes the code → writes a fix → runs the tests → iterates until they pass (max 5 iterations).

```python
from dataclasses import dataclass

MAX_ITERATIONS = 5

@dataclass
class LoopState:
    iteration: int = 0
    tests_passing: bool = False
    last_error: str | None = None

def run_fix_loop(issue: str, repo_path: str) -> LoopState:
    state = LoopState()

    while state.iteration < MAX_ITERATIONS and not state.tests_passing:
        state.iteration += 1

        # Perception: analyze code relevant to the issue
        context = analyze_codebase(repo_path, issue, previous_error=state.last_error)

        # Reasoning + Action: Claude proposes and applies a fix
        patch = generate_fix(issue, context)
        apply_patch(repo_path, patch)

        # Observation: run tests
        result = run_tests(repo_path)
        state.tests_passing = result.success
        state.last_error = None if result.success else result.error_output

        if state.tests_passing:
            break

        if state.iteration == MAX_ITERATIONS:
            # Step limit reached: report, don't retry further
            raise LoopExhausted(
                f"Issue not resolved in {MAX_ITERATIONS} iterations. "
                f"Last error: {state.last_error}"
            )

    return state
```

Key points from the example:
- Dual exit condition: success (`tests_passing`) or step limit (`MAX_ITERATIONS`)
- The previous iteration's error is passed as context to the next one (`previous_error`) — the loop learns from its own failure
- When the limit is exhausted, an explicit exception is raised instead of failing silently

## Anti-patterns

- Loops without a step limit — recurring poor reasoning can consume budget indefinitely
- Not passing the previous iteration's error to the next attempt — the agent repeats the same mistake without learning
- Executing irreversible actions without HITL just because "it's probably fine"
- Confusing "the loop ended" with "the task completed successfully" — always verify the success condition explicitly, not just the absence of errors

## Related Notes

- [[Graph-Engineering]] — when a single loop isn't enough and multiple coordinated agents are needed
- [[Context-Engineering]] — context management and checkpointing within the loop
- [[Harness-Engineering]] — how to systematically evaluate whether a loop produces good results
- [[Observability]] — instrumenting loops to detect excessive iterations or anomalous costs
- [[Security]] — risks of running automated actions without human supervision
