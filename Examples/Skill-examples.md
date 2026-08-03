---
tags: [claude-code, examples, skills]
type: example
related:
  - "[[Anatomy-of-a-Skill]]"
  - "[[Harness-Engineering]]"
created: 2026-08-02
status: stable
---

# Skills — Complete Examples

> **Source**: notes from *AI Builders Meetup Guadalajara #1*, hosted by Wizeline (Guadalajara, 2026-08-02).

Three complete, functional skills, each with its full `SKILL.md` and an example invocation + expected output. See [[Anatomy-of-a-Skill]] for the general anatomy.

## Skill 1: Generate FastAPI Endpoint

````markdown
---
name: generate-fastapi-endpoint
description: Use this skill when the user asks to create a new REST endpoint in a FastAPI project, including Pydantic model, tests, and DB migration if applicable.
---

# Generate FastAPI Endpoint

## When to use
- "create an endpoint for..."
- "I need a POST/GET/PUT/DELETE route that..."

## When NOT to use
- Modifying an already-existing endpoint
- Projects that don't follow the routers/schemas/repositories structure

## Prerequisites
- FastAPI project with standard structure (`app/routers/`, `app/schemas/`, `app/repositories/`)
- `uv sync` already run

## Step-by-step process
1. Define the Pydantic model in `app/schemas/{resource}.py`
2. Define or reuse the repository in `app/repositories/{resource}_repository.py`
3. Create the router in `app/routers/{resource}.py`
4. Register the router in `app/main.py`
5. Write tests in `tests/routers/test_{resource}.py`
6. If it requires a new table/column, generate a migration with `alembic revision --autogenerate`

## Example code
```python
# app/schemas/order.py
from pydantic import BaseModel

class OrderCreate(BaseModel):
    customer_id: int
    total: float

class OrderOut(OrderCreate):
    id: int
    status: str
```

```python
# app/routers/order.py
from fastapi import APIRouter, Depends
from app.schemas.order import OrderCreate, OrderOut
from app.repositories.order_repository import OrderRepository

router = APIRouter(prefix="/orders", tags=["orders"])

@router.post("", response_model=OrderOut, status_code=201)
async def create_order(payload: OrderCreate, repo: OrderRepository = Depends()) -> OrderOut:
    return await repo.create(payload)
```

## Validation
- `uv run pytest tests/routers/test_order.py -v` passes
- `uv run ruff check app/routers/order.py` no errors

## Troubleshooting
- 422: schema doesn't match the payload, check types
- ImportError on server start: router not registered in `main.py`
````

**Example invocation**:

> User: "create a POST /orders endpoint to register a new order"

**Expected output**: Claude invokes the skill, generates `app/schemas/order.py`, `app/routers/order.py`, registers the router, and generates `tests/routers/test_order.py` — following exactly the step-by-step process defined above.

---

## Skill 2: Code Review Checklist

````markdown
---
name: code-review-checklist
description: Use this skill when the user asks to review a PR or code diff systematically, applying a consistent rubric instead of ad hoc comments.
---

# Code Review Checklist

## When to use
- "review this PR"
- "do a code review of this diff"

## When NOT to use
- High-level design/architecture reviews (this is for concrete diffs)

## Prerequisites
- Access to the full diff (via git diff or PR URL)
- Project CLAUDE.md available to check conventions

## Step-by-step process
1. Read the full diff before commenting on anything
2. Check correctness: does the code do what the PR title claims?
3. Check security: unvalidated inputs, hardcoded secrets, SQL injection
4. Check project conventions (against CLAUDE.md)
5. Check test coverage for the change
6. Prioritize findings: blocker > important > minor suggestion
7. Generate the report in the output format

## Example code
```python
# Severity rubric applied consistently
SEVERITY_LEVELS = {
    "blocker": "Logic bug or security vulnerability — do not merge without fixing",
    "important": "Violates a project convention or introduces significant tech debt",
    "minor": "Style suggestion or non-critical improvement",
}

def format_finding(file: str, line: int, severity: str, issue: str, fix: str) -> str:
    return f"{file}:{line} [{severity.upper()}] {issue}. Suggested fix: {fix}"
```

## Validation
- Every finding has a location (file:line), severity, and suggested fix
- No style comments if the project's linter already covers it

## Troubleshooting
- If the diff is very large, review by module/file instead of all at once, prioritizing files with business logic over configuration files
````

**Example invocation**:

> User: "review this PR: [200-line diff adding a refunds endpoint]"

**Expected output**:
```
app/routers/refund.py:34 [BLOCKER] Doesn't validate that the refund
amount doesn't exceed the original order total. Suggested fix: add
validation `if refund.amount > order.total: raise ValueError(...)`.

app/routers/refund.py:52 [IMPORTANT] Doesn't follow the repository
pattern documented in CLAUDE.md — accesses the DB directly via ORM.
Suggested fix: move the query to RefundRepository.
```

---

## Skill 3: Write ADR (Architecture Decision Record)

`````markdown
---
name: write-adr
description: Use this skill when the user describes a technical decision informally and asks to document it formally as an ADR (Architecture Decision Record).
---

# Write ADR

## When to use
- "document this decision as an ADR"
- "we need an ADR for choosing X over Y"

## When NOT to use
- Trivial decisions with no relevant trade-offs (not everything deserves an ADR)

## Prerequisites
- Clear context on which alternatives were considered and why they were discarded

## Step-by-step process
1. Identify the next sequential ADR number (check `docs/adr/`)
2. Extract from the informal description: context, decision, alternatives considered, consequences
3. Draft using the standard template
4. Save to `docs/adr/{NNNN}-{kebab-case-title}.md`

## Example code
````markdown
# ADR-0007: Use Celery instead of Arq for async jobs

## Status
Accepted

## Context
We need to process async jobs (sending emails, nightly inventory
reconciliation). We evaluated Celery and Arq.

## Decision
We use Celery with Redis as the broker.

## Alternatives Considered
- **Arq**: simpler, natively async, but the monitoring ecosystem
  (Flower equivalent) is less mature
- **Celery**: heavier, but the team already has operational experience
  and Flower for production monitoring

## Consequences
- Positive: operational monitoring already solved, low learning curve
  for the existing team
- Negative: Celery isn't natively async, requires wrappers for
  existing async code
````

## Validation
- The ADR includes all 4 sections: Status, Context, Decision, Consequences
- Discarded alternatives are documented with their reason for discard, not just mentioned

## Troubleshooting
- If the user doesn't mention alternatives considered, ask explicitly before generating the ADR — an ADR without alternatives loses half its value as a decision document
`````

**Example invocation**:

> User: "we decided to use Celery instead of Arq because we already have Flower set up for monitoring, document this as an ADR"

**Expected output**: Claude generates `docs/adr/0007-use-celery-instead-of-arq.md` with the content from the code example above, adapted to the real context given by the user.

## Related Notes

- [[Anatomy-of-a-Skill]] — general anatomy and best practices for designing skills
- [[Harness-Engineering]] — how to systematically evaluate whether a skill produces consistent results
