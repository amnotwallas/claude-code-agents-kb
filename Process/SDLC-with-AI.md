---
tags: [claude-code, process, sdlc, ai-augmented, workflow]
type: process
related:
  - "[[Spec-Driven-Development]]"
  - "[[Harness-Engineering]]"
  - "[[Observability]]"
  - "[[Security]]"
created: 2026-08-02
status: stable
---

# SDLC with AI

> **Source**: notes from *AI Builders Meetup Guadalajara #1*, hosted by Wizeline (Guadalajara, 2026-08-02).

## The SDLC Transformed by AI

### 1. Planning & Discovery
- **AI for**: effort estimation, risk analysis, tech stack evaluation, generating PRDs from meeting notes
- **Prompt template** for generating a PRD from bullet points:

```
Turn these meeting notes into a structured PRD with sections:
Problem, Goal, Affected Users, Functional Requirements,
Non-Functional Requirements, Risks, Out of Scope.

Notes: {meeting bullet points}
```

- **Expected metric**: reduces first-draft PRD time from days to hours — the human still validates and adjusts priorities

### 2. Design & Architecture
- **AI for**: reviewing architecture diagrams, generating ADRs (Architecture Decision Records), identifying trade-offs, documenting APIs
- Claude can review a proposed architecture by questioning assumptions: "why microservices instead of a modular monolith for this 4-person team?"
- **Expected metric**: more ADRs get documented (because the cost of writing them drops), better-justified decisions

### 3. Implementation
- Claude Code as pair programmer: write the high-value design/decision code yourself, delegate mechanical or repeatable-pattern code to Claude
- Workflow: spec (see [[Spec-Driven-Development]]) → Claude Code implements → human review → iterate
- Large projects: break into bounded tasks, use [[CLAUDE-md]] to keep architecture context consistent across sessions
- **Expected metric**: higher PR throughput, but only if review doesn't become the new bottleneck

### 4. Testing & QA
- Generating tests from specs (unit, integration, e2e) — the spec as source of truth, see [[Spec-Driven-Development]]
- Generating edge cases humans miss due to fatigue or happy-path bias
- Evals for features with AI components (see [[Harness-Engineering]])
- AI-assisted fuzzing: systematically generating adversarial inputs
- **Expected metric**: broader edge-case coverage, not necessarily less total QA time

### 5. Code Review
- Claude as first-pass reviewer: catches bugs, code smells, and security issues before human review, reducing the load of the trivial stuff
- **Prompt template** for effective code review:

```
Review this diff focusing on: (1) logic bugs, (2) security
vulnerabilities, (3) violations of the conventions in CLAUDE.md.
Do NOT comment on style already covered by the linter. Prioritize
findings by severity.
```

- **What NOT to delegate**: major architecture decisions (is this redesign worth it?), business trade-offs, final approval of high-risk changes

### 6. Deployment & Infrastructure
- Generating IaC (Terraform, Kubernetes manifests) from infrastructure specs
- Generating runbooks and operational playbooks from documented architecture
- Validating security configurations before applying changes (see [[Security]])
- **Expected metric**: fewer manual configuration errors, but requires human review on critical infrastructure changes

### 7. Monitoring & Maintenance
- Log analysis with Claude: detecting patterns a human would take a long time to notice across high log volume
- Generating alerts from postmortems — turning "this happened and we didn't see it coming" into a concrete alert
- Automatic change documentation (changelogs, release notes) from commit history
- Direct connection to [[Observability]]: AI-assisted log analysis depends on having structured, consistent logs in the first place

## AI-Augmented vs AI-Automated

- **Augmented**: human in control, AI accelerates work but doesn't decide autonomously. Use it when errors are costly or hard to reverse.
- **Automated**: AI executes autonomously without human approval at every step. Use it only when the action space is well bounded, reversible, and there's enough observability to detect misbehavior quickly.

| SDLC Activity | Augmented | Automated |
|---|---|---|
| Generate PRD from notes | ✓ (human adjusts priorities) | |
| Write feature code | ✓ (human reviews before merge) | |
| Format/lint code | | ✓ (low risk, reversible) |
| Generate tests from spec | ✓ (human validates they reflect the spec) | |
| Deploy to production | ✓ (human approval required) | |
| Auto-rollback on service health failure | | ✓ (reversible, well-bounded action) |
| Respond to security incidents | ✓ (human always in the loop) | |

## Productivity Metrics with AI

How to measure real impact, not just "it feels faster":

- **Lead time**: time from feature definition to production
- **Cycle time**: time from starting to code until the PR merges
- **Defect rate**: bugs found in production per unit of delivered feature
- **Review time**: time it takes to review and approve a PR

**Metrics that can worsen at first, and why**:
- **Review time can go up** initially: the volume of generated code grows faster than human review capacity, creating a new bottleneck
- **Defect rate can go up** if the team trusts generated code without the same rigor they'd apply to their own code — familiarity reduces skepticism, which is itself a risk

## Anti-patterns

- Measuring productivity only by "lines of code generated" — a metric that can be inflated without generating real value
- Automating irreversible steps (deploy, data deletion) without human-in-the-loop just because "Claude is usually right"
- Delegating major architecture decisions to Claude without business context, treating it as if it had the same strategic understanding as the team
- Not adjusting the review process as AI-generated code volume rises — the bottleneck just moves, it doesn't disappear

## Related Notes

- [[Spec-Driven-Development]] — the formalized Design/Requirements phase
- [[Harness-Engineering]] — how the Testing & QA phase gets systematized with AI
- [[Observability]] — the foundation needed for the Monitoring & Maintenance phase
- [[Security]] — validations required before Deployment
