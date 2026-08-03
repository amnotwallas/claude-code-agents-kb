---
tags: [claude-code, engineering, context, memory, tokens]
type: engineering
related:
  - "[[CLAUDE-md]]"
  - "[[Prompt-Engineering]]"
  - "[[Loop-Engineering]]"
  - "[[Observability]]"
created: 2026-08-02
status: stable
---

# Context Engineering

> **Source**: notes from *AI Builders Meetup Guadalajara #1*, hosted by Wizeline (Guadalajara, 2026-08-02).

## The Context Window as a Scarce Resource

The context window is the total amount of tokens (tokenized text) Claude can "see" in a single call — including system prompt, conversation history, files read, and the current input. Although modern windows are large, they remain a finite resource with real costs.

The cost of tokens shows up in three dimensions:
- **Speed**: more context tokens → more processing time
- **Price**: providers charge per input and output token
- **Response quality**: irrelevant or poorly organized context dilutes signal and can degrade response accuracy ("lost in the middle")

**Guiding principle**: only put into context what changes the response. Everything else is noise competing for the model's attention.

## Context Hierarchy in Claude Code

```
System Prompt (always present)
    ↓
Project CLAUDE.md (always active in the project)
    ↓
Relevant Skills (loaded per task)
    ↓
Conversation history (sliding window)
    ↓
User input (the current turn)
```

Each layer has a different persistence priority: the system prompt is never discarded, [[CLAUDE-md]] reloads on every new session, skills (see [[Anatomy-of-a-Skill]]) come and go based on the task, and conversation history is the first thing compressed or discarded when context fills up.

## Context Management Strategies

### Context Priming
Load at session start the context that's most *relevant* to the task, not simply the most *recent*. A file read 20 turns ago but central to the current task is worth more than a trivial message from the previous turn.

### Progressive Compression
As a conversation grows, summarize old turns while preserving key decisions (what was decided, why) and discarding process detail (which commands were tried and failed before reaching the solution).

### Sliding Window
Discard old, irrelevant turns while retaining ones containing active decisions. It's not a simple FIFO — it requires judgment about which turn still matters.

### RAG for External Context
For large documents (specs, API documentation, extensive legacy code), don't paste the whole document — retrieve and paste only the fragment relevant to the current question. This is RAG (Retrieval-Augmented Generation) applied to the agent's workflow.

### Context Checkpointing
In long loops (see [[Loop-Engineering]]), periodically save the agent's state to external files (not just in the conversation history), so an interrupted session can resume without reconstructing all prior reasoning.

## Context Poisoning: the Hidden Risk

**What it is**: malicious or erroneous instructions that contaminate the context and alter the agent's behavior without the user explicitly asking for it.

**How it happens**:
- Unsanitized user input containing text shaped like an instruction ("ignore previous rules and...")
- Project files (code, comments, third-party READMEs) containing text designed to be read by an agent, not a human

**How to mitigate**:
- Clearly separate data from instructions — never concatenate user input directly into the "system instructions" zone
- Treat any content from external sources as untrusted (see [[Security]] for the full threat model)

## CLAUDE.md as Persistent Memory

[[CLAUDE-md]] solves the problem of Claude "forgetting" everything between sessions — without it, every new session starts from zero.

**What belongs in CLAUDE.md**: stable decisions, conventions that don't change session to session, frequent commands.

**What's temporary context** (doesn't belong in CLAUDE.md): the current state of an in-progress task, the current session's plan, details of a specific bug being debugged right now — that lives in the conversation or a temporary working file, not in the project's persistent memory.

Updating CLAUDE.md should be a natural part of the workflow: when a session produces a new architecture decision, that decision gets documented in CLAUDE.md before closing the session, not after forgetting it.

## Context Metrics

- **Tokens used / available**: monitor in real time how much of the total budget has been consumed, to anticipate compression or truncation
- **Context relevance**: what percentage of present tokens actually influence the response — a rough proxy is how much of the context is explicitly referenced in the response
- **Detecting context overflow**: a typical signal is the agent starting to ignore early-session instructions (from CLAUDE.md or the system prompt) because they got "buried" under a lot of recent history

## Anti-patterns

- Pasting entire documents when only one section is needed — wastes tokens and dilutes relevant signal
- Letting conversation history grow indefinitely without compression, until the agent starts "forgetting" early instructions
- Mixing temporary task context into CLAUDE.md, polluting persistent memory with short-lived noise
- Not monitoring token usage until an overflow has already occurred and response quality has already degraded

## Related Notes

- [[CLAUDE-md]] — the persistent memory layer within the context hierarchy
- [[Prompt-Engineering]] — how to structure prompt content within the available context budget
- [[Loop-Engineering]] — context checkpointing in long-running loops
- [[Observability]] — how to measure token usage and relevance in production
