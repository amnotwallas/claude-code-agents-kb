---
tags: [claude-code, engineering, memory, state, persistence]
type: engineering
related:
  - "[[Context-Engineering]]"
  - "[[CLAUDE-md]]"
  - "[[Agent-Architecture]]"
  - "[[Six-Pillars-of-Agents]]"
created: 2026-08-02
status: stable
---

# Memory Strategies

> **Source**: notes from *AI Builders Meetup Guadalajara #1*, hosted by Wizeline (Guadalajara, 2026-08-02).

Pillar 3 of the [[Six-Pillars-of-Agents|6 pillars of agents]]. Distinct from [[Context-Engineering]]: context engineering manages what goes into a single call's window; memory strategies decide what information survives **between** calls and sessions, and how it gets retrieved when needed.

## Memory vs Context: the Distinction

- **Context window**: what the model sees in this specific call — ephemeral by definition, rebuilt every time (see [[Context-Engineering]])
- **Memory**: information that persists beyond a single call or session, and that gets actively decided to retrieve back into context when relevant

An agent without a memory strategy "forgets" everything once the session ends — every new conversation starts from zero, no matter how much work happened before. [[CLAUDE-md]] is one concrete form of memory (at the project level), but not the only one.

## Types of Memory

### Short-term memory (working memory)
The current conversation's history — lives and dies with the session. In practice, it's an extension of the context window (see [[Context-Engineering]]).

### Long-term (persistent) memory
Information that survives across sessions, stored in external storage (files, DB, vector store) and retrieved explicitly when applicable. [[CLAUDE-md]] is the canonical example in Claude Code: project memory, always loaded at session start.

### Episodic memory
A record of specific past events or interactions ("the last time this module was touched, the bug was X"). Useful for avoiding repeating already-solved mistakes.

### Semantic memory
Distilled general knowledge, not tied to a specific event ("this project uses the repository pattern"). This is typically what lives in CLAUDE.md or in a skill (see [[Anatomy-of-a-Skill]]).

### Procedural memory
"How to" do something — the equivalent of a skill: a reusable procedure, not a one-off fact.

## Implementation Strategies

### 1. Versioned Plain File
`CLAUDE.md` and skills — simple, git-versionable, readable by humans and by the agent alike. Ideal for a project's semantic and procedural memory. Limitation: doesn't scale well to large volumes of granular information (thousands of past events).

### 2. Retrieval (Vector Store / RAG)
For high-volume memory that doesn't fit, or shouldn't stay, always loaded — it gets indexed and only the fragment relevant to the current query is retrieved (same principle as RAG in [[Context-Engineering]], applied to long-term memory instead of external documents).

### 3. Structured State (DB / Key-Value)
For high-volume episodic memory that needs structured querying (e.g., "all decisions made about module X in the last 3 months"). Requires write discipline — the agent must explicitly decide what's worth persisting.

### 4. Task-State Checkpointing
Not knowledge memory but progress memory — saving what step a long-running task is on so it can be resumed without restarting (see [[Loop-Engineering]], Loop State section).

```python
# Example: simple episodic memory backed by a JSON file
import json
from pathlib import Path
from datetime import datetime, UTC

MEMORY_FILE = Path("agent_memory/episodes.jsonl")

def record_episode(summary: str, tags: list[str]) -> None:
    entry = {
        "timestamp": datetime.now(UTC).isoformat(),
        "summary": summary,
        "tags": tags,
    }
    with MEMORY_FILE.open("a") as f:
        f.write(json.dumps(entry) + "\n")

def recall_episodes(tag: str, limit: int = 5) -> list[dict]:
    if not MEMORY_FILE.exists():
        return []
    episodes = [json.loads(line) for line in MEMORY_FILE.read_text().splitlines()]
    matching = [e for e in episodes if tag in e["tags"]]
    return matching[-limit:]

# Usage: before touching the payments module, retrieve relevant episodic memory
past_issues = recall_episodes(tag="payments-module")
```

## What's Worth Persisting (and What Isn't)

| Persist | Don't persist |
|---|---|
| Stable architecture decisions | Current task's temporary state (use checkpointing, not memory) |
| Project conventions | Details of an already-resolved bug with no future value |
| Recurring error patterns and their fix | Full document content retrievable on demand via RAG |
| Confirmed user/team preferences | Unverified assumptions |

This is the same criterion that applies to [[CLAUDE-md]]: if something applies to fewer than 30% of tasks, it goes in a skill or gets retrieved on demand — it doesn't get loaded by default as always-on persistent memory.

## Anti-patterns

- Confusing long-term memory with simply "not truncating the conversation history" — that's context management, not real persistent memory (see [[Context-Engineering]])
- Persisting everything indiscriminately "just in case" — poorly curated memory becomes as damaging as noise as having no memory at all
- Not versioning persistent memory (CLAUDE.md, skills) — loses traceability of when and why it changed
- Storing episodic memory with sensitive information without access control — same exposure risk as any other persisted data (see [[Security]])

## Related Notes

- [[Context-Engineering]] — memory gets retrieved into the context window; both work together but solve different problems
- [[CLAUDE-md]] — the concrete implementation of project-level semantic/procedural memory in Claude Code
- [[Agent-Architecture]] — memory strategy as an architectural design dimension
- [[Six-Pillars-of-Agents]] — this pillar in the context of all 6 pillars
