---
tags: [claude-code, core, extended-thinking, reasoning]
type: core
related:
  - "[[Prompt-Engineering]]"
  - "[[Loop-Engineering]]"
  - "[[Agent-Architecture]]"
  - "[[Observability]]"
created: 2026-08-02
status: stable
---

# Extended Thinking

> **Source**: notes from *AI Builders Meetup Guadalajara #1*, hosted by Wizeline (Guadalajara, 2026-08-02).

## What Is Extended Thinking?

A mode where Claude allocates a visible, budgeted reasoning pass before producing its final response — the model works through the problem step by step in a dedicated "thinking" block, then answers. Unlike a plain Chain-of-Thought prompt (asking Claude to "think step by step" in its regular output, see [[Prompt-Engineering]]), extended thinking is a first-class model capability with its own token budget, separate from the final answer.

It exists because some tasks benefit from more deliberate, longer reasoning than a single fast pass produces — the model gets space to consider approaches, catch its own mistakes, and backtrack before committing to an answer, without that scratch work cluttering the final output the user reads.

## How It Differs from Chain-of-Thought Prompting

| | CoT Prompting | Extended Thinking |
|---|---|---|
| **Mechanism** | A prompt instruction asking for step-by-step reasoning in the normal output | A distinct model capability with a dedicated reasoning budget |
| **Visibility to the user** | Reasoning is mixed into the final response unless manually stripped | Reasoning is a separate block, cleanly separated from the answer |
| **Reasoning depth** | Bounded by normal output length and the model's default effort | Can allocate a much larger budget specifically for reasoning |
| **Control** | Prompt-level only ("think step by step") | Explicit thinking budget/effort setting |

They're not mutually exclusive — a well-structured prompt (see [[Prompt-Engineering]]) still helps extended thinking reason well; extended thinking doesn't replace clear instructions, it gives the model more room to use them.

## When to Use It

- **Complex multi-step reasoning**: a task where the correct answer depends on correctly working through several dependent sub-problems (e.g., diagnosing a bug whose root cause is several layers removed from the symptom)
- **Planning before acting**: at the start of an [[Loop-Engineering|agentic loop]] or [[Agent-Architecture|architecture]] decision, where a wrong first move is expensive to walk back
- **Ambiguous requirements**: when the task has multiple plausible interpretations and picking the right one benefits from explicitly weighing them before committing
- **High-stakes single-shot answers**: when there's no cheap opportunity to iterate (e.g., a one-shot architecture recommendation), so getting more reasoning in before the answer is worth the cost

## When NOT to Use It

- **Simple, well-defined tasks**: formatting a file, running a known command, answering a factual lookup — extra reasoning budget adds latency and cost without improving the outcome
- **Latency-sensitive interactive flows**: a chat-like interaction where the user expects a fast response and the task doesn't warrant the delay
- **Tasks already broken down by a workflow**: if [[Workflow-Patterns|Prompt Chaining]] has already decomposed the task into simple, focused steps, each step may not need its own extended reasoning pass — the decomposition already did the heavy lifting

## Trade-offs

- **Latency**: a larger reasoning budget means more time before the final answer starts — a direct cost against responsiveness
- **Token cost**: reasoning tokens are still tokens; more thinking budget means more cost per call (see [[Observability]] for tracking this in production)
- **Not a substitute for good context**: extended thinking reasons better with the right context, but it can't invent information that isn't in [[Context-Engineering|context]] — more thinking time doesn't fix a starved or poisoned context
- **Diminishing returns**: past a certain budget, more reasoning tokens stop meaningfully improving the answer for a given task's actual complexity — sizing the budget to the task matters more than maximizing it

## How It Fits Agent Architecture

Extended thinking is a per-call capability, not an architectural pattern by itself — it composes with the patterns in [[Agent-Architecture]] and [[Workflow-Patterns]]:

- In a **single-agent loop** (see [[Loop-Engineering]]), extended thinking can be used at the reasoning step of each ReAct cycle, or more selectively — only when the agent's confidence is low or the next action is high-risk
- In an **Evaluator-Optimizer** pattern (see [[Workflow-Patterns]]), the generator step often benefits more from extended thinking than the evaluator step, since generation is where the hard reasoning happens
- In an **Orchestrator-Worker** setup (see [[Graph-Engineering]]), the orchestrator's task-decomposition decision is a natural candidate for extended thinking; individual workers doing narrow, well-defined sub-tasks usually aren't

## Anti-patterns

- Turning on extended thinking for every call by default, regardless of task complexity — pays latency and cost for tasks that don't need it
- Treating extended thinking as a substitute for a well-structured prompt or sufficient context — it improves reasoning over what's given, it doesn't compensate for what's missing
- Using extended thinking as the only quality lever while ignoring [[Harness-Engineering|evals]] — without measurement, there's no way to confirm the extra reasoning actually improved the outcome for your specific task
- Exposing the raw thinking block to end users as if it were the final answer — it's reasoning scratch work, not a polished response

## Related Notes

- [[Prompt-Engineering]] — how prompt structure and Chain-of-Thought relate to and complement extended thinking
- [[Loop-Engineering]] — where in the agentic loop extended thinking is most valuable
- [[Agent-Architecture]] — extended thinking as a per-call capability composing with architectural patterns
- [[Observability]] — tracking the latency and cost impact of extended thinking in production
