---
tags: [claude-code, examples, claude-md]
type: example
related:
  - "[[CLAUDE-md]]"
  - "[[Context-Engineering]]"
created: 2026-08-02
status: stable
---

# CLAUDE.md — Complete Examples

> **Source**: notes from *AI Builders Meetup Guadalajara #1*, hosted by Wizeline (Guadalajara, 2026-08-02).

Three complete, functional `CLAUDE.md` examples for different project types. Each includes annotations explaining why each section is written the way it is. See [[CLAUDE-md]] for the general anatomy.

## Example 1: Backend Python/FastAPI Microservice

```markdown
# CLAUDE.md — Payments Service

## Project Overview
Payment processing microservice. Receives checkout requests, integrates
with Stripe, and publishes result events to RabbitMQ.

## Architecture
- FastAPI + PostgreSQL + Redis (idempotency keys) + Celery (async retries)
- Repository pattern: never direct ORM in endpoints
- Every payment is processed idempotently via the `Idempotency-Key` header

## Code Style
- `uv run ruff format .` + `uv run ruff check --fix .` before every commit
- Type hints required on every public function
- Tests with pytest, DB fixtures with automatic rollback per test

## Common Commands
| Command | Purpose |
|---|---|
| `uv run pytest -x` | Run tests, stop at first failure |
| `uv run alembic upgrade head` | Apply migrations |
| `uv run python scripts/replay_failed_payments.py` | Reprocess failed payments |

## MCP Servers
- `postgres` (staging only, read-only) — transaction debugging
- `stripe-docs` — Stripe API documentation lookup

## Key Files
- `app/services/payment_processor.py` — business core, >95% coverage required
- `app/repositories/idempotency_repository.py` — idempotency control, don't touch without understanding the Redis TTL

## Anti-patterns
- DO NOT retry a payment without checking the Idempotency-Key first
- DO NOT log card numbers or CVV, not even partially (PCI compliance)
- DO NOT modify `payment_processor.py` without running the integration suite against the Stripe sandbox

## External References
- Idempotency spec: `docs/specs/idempotency.md`
- Payment incident runbook: `docs/runbooks/payment-failures.md`
```

**Annotations**:
- *Project Overview* is deliberately short — a payments microservice's purpose doesn't need more than 2 lines to orient Claude
- *Anti-patterns* includes a compliance rule (PCI) because it's the kind of restriction Claude can't infer from the code alone — it's a business/legal rule, not a technical one
- *Key Files* explicitly flags which file requires extra care, guiding Claude to be more conservative there

---

## Example 2: Frontend React/TypeScript SPA

```markdown
# CLAUDE.md — Admin Dashboard

## Project Overview
Internal admin SPA for the operations team. User management, orders,
and reports. Consumes the main backend's REST API.

## Architecture
- React 18 + TypeScript + Vite
- Global state: Zustand (not Redux — deliberate decision for simplicity)
- Data fetching: TanStack Query, never direct fetch in components
- Components in `src/components/`, one file per component, co-located with its test

## Code Style
- Prettier + ESLint, config in `.eslintrc.cjs` (non-negotiable, already tuned)
- Functional components only, no class components
- Props typed with interfaces, no `any` under any circumstance

## Common Commands
| Command | Purpose |
|---|---|
| `npm run dev` | Local dev server |
| `npm run test` | Tests with Vitest |
| `npm run build` | Production build, fails on type errors |

## MCP Servers
- `figma` — reading designs to implement components faithful to the mockup

## Key Files
- `src/lib/api-client.ts` — single entry point to the API, every fetch goes through here
- `src/store/` — Zustand stores, one per domain (users, orders, reports)

## Anti-patterns
- DO NOT use `useEffect` for data fetching — always use TanStack Query
- DO NOT create new Zustand stores without checking if the state already belongs to an existing store
- DO NOT add new UI libraries — the design system already covers the needed components

## External References
- Design system: `src/components/ui/README.md`
- Testing conventions: `docs/testing-conventions.md`
```

**Annotations**:
- *Architecture* explicitly justifies why Zustand and not Redux — anticipates that Claude might suggest Redux as the more "standard" choice and blocks that suggestion with the reasoning already documented
- The *Anti-patterns* entry about `useEffect` targets a common anti-pattern in LLMs trained on older, generic React code — it's documented explicitly because Claude might otherwise "fix" the correct pattern thinking it's a bug

---

## Example 3: Multi-agent AI System

```markdown
# CLAUDE.md — Support Ticket Triage System

## Project Overview
Multi-agent system that classifies, prioritizes, and suggests responses
for incoming support tickets. See [[Graph-Engineering]] for the
orchestration pattern used.

## Architecture
- Orchestrator (`agents/orchestrator.py`) coordinates 3 workers:
  - `classifier-agent`: categorizes the ticket (billing, technical, account)
  - `priority-agent`: assigns severity (low/medium/high/critical)
  - `response-agent`: drafts a suggested response
- Communication via shared state (`TicketState` object), not message passing
- Each agent runs with its own versioned prompt in `agents/prompts/`

## Code Style
- Prompts versioned as separate `.md` files, apart from the Python code
- One eval test per agent in `evals/`, run in CI on every prompt change

## Common Commands
| Command | Purpose |
|---|---|
| `uv run python evals/run_suite.py` | Run evals for all agents |
| `uv run python scripts/replay_ticket.py --id X` | Debug the full flow for a specific ticket |

## MCP Servers
- `zendesk` — read/write access to real tickets (permissions limited to staging)

## Key Files
- `agents/orchestrator.py` — coordination logic, changes here affect the whole system
- `agents/prompts/classifier.md` — classifier prompt, high change sensitivity (affects downstream priority-agent)

## Anti-patterns
- DO NOT modify an agent's prompt without running `evals/run_suite.py` before and after, and comparing the score
- DO NOT give `response-agent` permission to send the response directly to the customer — always requires human approval (see [[Security]], HITL section)
- DO NOT mix business logic into the orchestrator — the orchestrator only coordinates, it doesn't decide content

## External References
- Triage system spec: `docs/specs/triage-system.md`
- Golden ticket dataset: `evals/golden_set.json`
```

**Annotations**:
- *Anti-patterns* explicitly prohibits sending automatic responses to the customer — it's the concrete application of human-in-the-loop from [[Security]] in this specific project
- *Key Files* flags the classifier prompt as sensitive because a change there has a cascading effect on the rest of the system (priority-agent depends on the assigned category)

## Related Notes

- [[CLAUDE-md]] — general anatomy and best practices
- [[Context-Engineering]] — why CLAUDE.md should stay concise and up to date
