---
tags: [claude-code, core, agents, framework, production]
type: core
related:
  - "[[Context-Engineering]]"
  - "[[Agent-Architecture]]"
  - "[[Memory-Strategies]]"
  - "[[Harness-Engineering]]"
  - "[[Observability]]"
  - "[[Extended-Thinking]]"
  - "[[MCP]]"
created: 2026-08-02
status: stable
---

# Six Pillars of Agents

A 6-pillar framework for evaluating how solid an AI agent system is — sourced from the talk "Building AI Agents that Matter" at **AI Builders Meetup Guadalajara #1**, hosted by Wizeline (2026-08-02). The companion talk that evening, "Building Better with Claude Code," covered Claude Code, Extended Thinking, Memory, MCP, and Skills — see [[Extended-Thinking]] and [[MCP]] for the notes covering the parts of that talk not already addressed by [[CLAUDE-md]] and [[Anatomy-of-a-Skill]].

This note synthesizes the framework as a navigation map for the rest of the KB. It doesn't introduce isolated new concepts: each pillar points to the note(s) where that topic is developed in depth.

It works as a maturity checklist: an agent system in production should have a clear answer for all 6, not just the ones that were most "fun" to build.

## The 6 Pillars

```
1. Context Engineering    → what information the model sees on each call
2. Agent Architecture     → how the system is structured (single-agent,
                             workflow, multi-agent)
3. Memory Strategies      → what persists across calls and sessions
4. Evals                  → how quality is measured objectively
5. AI-Harness              → the infrastructure that runs and compares evals
6. Production Best        → how the system is operated once live
   Practices                 (observe, govern, scale)
```

### 1. Context Engineering
What goes into the context window of each call, and how that finite token budget is managed. See [[Context-Engineering]] for the context hierarchy in Claude Code, compression strategies, sliding window, and the risk of context poisoning.

### 2. Agent Architecture
The system's structural shape: single-agent loop, deterministic workflow, or multi-agent graph — and the design dimensions (autonomy, granularity, tool surface) that determine which one fits. See [[Agent-Architecture]] for the full breakdown, and [[Workflow-Patterns]] for the 5 concrete composable patterns and the workflow-vs-agent distinction.

### 3. Memory Strategies
What information survives beyond a single call or session, and how it gets retrieved when relevant — distinct from context engineering, which manages only what's ephemeral to the current call. See [[Memory-Strategies]] for memory types (short-term, long-term, episodic, semantic, procedural) and implementation strategies.

### 4. Evals
How to measure, systematically rather than anecdotally, whether the agent produces correct results. Without evals, a prompt or architecture change is a bet, not a verifiable improvement. See [[Harness-Engineering]] for eval types (unit, integration, e2e, adversarial) and golden dataset construction.

### 5. AI-Harness
The concrete infrastructure that runs those evals repeatably: experiment tracking, prompt version comparison, CI/CD integration. It's the operational implementation of the Evals pillar — see [[Harness-Engineering]] (same note; evals and harness are, in practice, two sides of the same discipline: what to measure and what infrastructure to measure it with).

### 6. Production Best Practices (Observe, Govern, Scale)
How the system is operated once it's no longer a prototype — observability (see [[Observability]]), security and governance (see [[Security]]), and the priorities that shift as the system goes from one user to thousands. Detailed below.

## Production Best Practices: Humans, Governance & What Changes at Scale

### Human in the Loop
Human oversight isn't a temporary patch until the agent is "good enough" — it's a permanent design control for the highest-risk actions, even in mature systems. See [[Loop-Engineering]] (HITL section) for implementation patterns (approval gates, parallel review) and [[Security]] for the criteria on which actions require it (irreversible, low confidence, high stakes).

What changes as the system matures isn't *whether* there's HITL, but *where* it sits: a new system places the gate early and wide (review almost everything); a mature system with solid evals (pillar 4) can move the gate later and narrow it (review only what's genuinely anomalous), leaning on observability (pillar 6) to detect when something drifted from expected behavior.

### Governance
Explicit rules about what the agent is allowed to do, who's accountable when something goes wrong, and how the system's behavior is audited after the fact.

- **Permissions**: explicit allowlist of tools and their scope (see [[Security]], Tool Layer) — governance starts with least privilege
- **Traceability**: every tool call is logged immutably (see [[Observability]] and [[Security]], Audit section) — without this, governance is a paper policy with no way to verify it
- **Accountability**: when the agent takes an action with real-world effect, it must be clear who approved that class of action (the system design, a human at an approval gate, or both) — governance without assigned accountability isn't actionable during an incident
- **Regulatory compliance**: in regulated domains (payments, healthcare, personal data), agent governance must align with the same requirements already applying to the rest of the system — an agent isn't an exception to existing compliance

### Scale Shifts Priorities
Engineering priorities aren't static — they shift with the system's volume and maturity:

| Stage | Dominant priority | Most critical pillars |
|---|---|---|
| **Prototype** (1 user, exploration) | Iteration speed, validating the approach works | Agent Architecture, Context Engineering |
| **Pilot** (internal team, limited real cases) | Basic reliability, first evals | Evals, AI-Harness, Memory Strategies |
| **Early production** (real users, low-medium volume) | Observability, error handling, well-calibrated HITL | Production Best Practices, Evals |
| **Production at scale** (high volume, multiple teams) | Cost per task, governance, consistent latency, graceful degradation | Production Best Practices (all 3 axes: observe, govern, scale), Agent Architecture (revisited — what worked at low volume may not work at high volume) |

What worked in the prototype (e.g., a single monolithic agent with a large context) frequently stops being the best architecture at scale (where cost per call and p99 latency start mattering more than flexibility) — it's normal and expected to redesign the architecture (pillar 2) when the system changes stage, not a sign the original design was wrong.

## How to Use This Framework

When evaluating or designing an agent system, walk through the 6 pillars as a checklist:

1. Is it clear what context the model sees on each call, and why? (pillar 1)
2. Does the chosen architecture match the task's actual variability, not the trendiest architecture? (pillar 2)
3. What survives between sessions, and why that specifically? (pillar 3)
4. Does a golden dataset exist with an objective "this improved" criterion? (pillar 4)
5. Can that criterion run repeatably, not just manually once? (pillar 5)
6. Is the production system observable, governed, and was its architecture reviewed for current scale? (pillar 6)

A system weak across several of these pillars at once isn't an isolated prompting problem — it's a signal that foundational work is missing before iterating further on features.

## Related Notes

- [[Context-Engineering]] — Pillar 1
- [[Agent-Architecture]] — Pillar 2
- [[Memory-Strategies]] — Pillar 3
- [[Harness-Engineering]] — Pillars 4 and 5
- [[Observability]] — half of Pillar 6 (observe)
- [[Security]] — half of Pillar 6 (govern)
- [[Workflow-Patterns]] — deep dive on Pillar 2
- [[Loop-Engineering]] — human-in-the-loop, a core component of Production Best Practices
