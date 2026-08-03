---
tags: [claude-code, core, memory, project-config]
type: core
related:
  - "[[Context-Engineering]]"
  - "[[Anatomy-of-a-Skill]]"
  - "[[Spec-Driven-Development]]"
  - "[[CLAUDE-md-examples]]"
created: 2026-08-02
status: stable
---

# CLAUDE.md

> **Source**: notes from *AI Builders Meetup Guadalajara #1*, hosted by Wizeline (Guadalajara, 2026-08-02).

## What Is CLAUDE.md?

`CLAUDE.md` is the project memory file that Claude Code automatically reads at the start of every session. It's the project's "external brain": persistent instructions you don't need to repeat in every prompt.

It exists at the project level (repo root) and can be inherited in subdirectories with their own `CLAUDE.md` files — for example, a `backend/CLAUDE.md` with service-specific conventions, and a `frontend/CLAUDE.md` with its own, both complementing the root `CLAUDE.md`.

## What Is It Used For?

- Persistent architecture and convention instructions
- Frequent commands (build, test, lint, deploy) so Claude can run them without asking
- Team context: who does what, how code gets reviewed, what tools to use
- Technical decisions already made: what NOT to change, what patterns to follow
- References to external documentation or project specs (see [[Spec-Driven-Development]])

## What Does It Contain? (Full Anatomy)

### 1. Project Overview
2-3 lines describing the project's purpose. Example:

```markdown
## Project Overview
Inventory management API for multi-store retail. Synchronizes stock
in real time between POS, e-commerce, and central warehouse.
```

### 2. Architecture
Stack and key patterns. Example:

```markdown
## Architecture
- FastAPI + PostgreSQL + Redis (cache) + Celery (async jobs)
- Repository pattern: never direct ORM in endpoints, always via repos/
- Event-driven between services: RabbitMQ, no synchronous HTTP calls between microservices
```

### 3. Code Style
Formatter, linter, naming conventions, file organization.

```markdown
## Code Style
- Formatter: ruff format. Linter: ruff check --fix
- snake_case for functions/variables, PascalCase for classes
- One file per endpoint in routers/, no monolithic routers
```

### 4. Common Commands
Table with command and purpose.

```markdown
## Common Commands
| Command | Purpose |
|---|---|
| `uv run pytest` | Run the full test suite |
| `uv run alembic upgrade head` | Apply pending migrations |
| `uv run ruff check --fix .` | Lint + autofix |
```

### 5. MCP Servers
Which MCP servers are configured and for what.

```markdown
## MCP Servers
- `postgres`: read-only access to the staging DB for debugging
- `github`: create/read PRs and issues for the repo
```

### 6. Key Files
Map of important files with their responsibility.

```markdown
## Key Files
- `app/core/config.py` — central configuration, reads from env vars
- `app/repositories/` — all data access logic
- `app/services/inventory_sync.py` — critical sync logic, high test coverage required
```

### 7. Anti-patterns
What NOT to do — crucial to prevent Claude from "fixing" things that are correct as-is.

```markdown
## Anti-patterns
- DO NOT use SQLAlchemy ORM directly in routers — always go through the repository
- DO NOT add new dependencies without updating pyproject.toml and running `uv lock`
- DO NOT modify migrations already applied in production — create a new one
```

### 8. External References
Links to PRDs, ADRs, API documentation.

```markdown
## External References
- Sync spec: `docs/specs/inventory-sync.md`
- ADR for choosing Celery over Arq: `docs/adr/0003-celery.md`
```

## Best Practices

- Keep it **under 200 lines** — if it grows past that, move detail to sub-`CLAUDE.md` files in subdirectories
- Update it with every significant architecture decision (don't let it go stale)
- Document the **"why"**, not just the "what" — the reasoning prevents Claude from reintroducing the original problem
- Use clear headers and lists, never dense paragraphs — Claude scans structure better than prose
- Version it in git like any other project file — it's configuration code
- Review it quarterly to remove obsolete sections

## Anti-patterns

- **Generic CLAUDE.md** that could belong to any project — provides no project-specific signal
- **Not documenting conventions** → Claude guesses, and guesses wrong in projects with history
- **Making it so long** that it becomes noise and competes with the current task's context
- **Contradictory instructions** between sections (e.g., "always use async" in one section, synchronous examples in another)
- **Including secrets or credentials** — use `.env` and reference it, never paste real values (see [[Security]])

## Complete Example

```markdown
# CLAUDE.md — Inventory Sync Service

## Project Overview
Backend service that synchronizes inventory between POS, e-commerce, and
central warehouse in real time. Exposes a REST API and consumes RabbitMQ
events.

## Architecture
- FastAPI + PostgreSQL (transactional data) + Redis (read cache)
- Celery + Redis as broker for async jobs (nightly reconciliation)
- Strict repository pattern: routers → services → repositories → DB
- Stock events via RabbitMQ, consumed by `workers/stock_consumer.py`

## Code Style
- `uv run ruff format .` before every commit
- Type hints required on public functions
- Tests with pytest + pytest-asyncio, fixtures in `tests/conftest.py`

## Common Commands
| Command | Purpose |
|---|---|
| `uv run uvicorn app.main:app --reload` | Start local server |
| `uv run pytest -x` | Tests, stop at first failure |
| `uv run alembic revision --autogenerate -m "..."` | New migration |
| `uv run celery -A app.worker worker` | Start local worker |

## MCP Servers
- `postgres` (staging only, read-only) — data debugging
- `github` — PR management

## Key Files
- `app/services/inventory_sync.py` — business core, >90% coverage
- `app/repositories/stock_repository.py` — sole entry point to the `stock` table
- `workers/stock_consumer.py` — RabbitMQ consumer, idempotent by design

## Anti-patterns
- DO NOT read/write the `stock` table outside `stock_repository.py`
- DO NOT make synchronous HTTP calls to other services — use events
- DO NOT deploy without running the integration suite against staging

## External References
- Sync spec: `docs/specs/inventory-sync.md`
- Incident runbook: `docs/runbooks/stock-mismatch.md`
```

## Related Notes

- [[Context-Engineering]] — CLAUDE.md as a layer in the context hierarchy
- [[Anatomy-of-a-Skill]] — when to use CLAUDE.md vs a skill
- [[Spec-Driven-Development]] — how to reference specs from CLAUDE.md
- [[CLAUDE-md-examples]] — more complete examples by project type
