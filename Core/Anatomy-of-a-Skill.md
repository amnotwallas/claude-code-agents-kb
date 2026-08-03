---
tags: [claude-code, core, skills, knowledge-modules]
type: core
related:
  - "[[CLAUDE-md]]"
  - "[[Harness-Engineering]]"
  - "[[Prompt-Engineering]]"
  - "[[Skill-examples]]"
created: 2026-08-02
status: stable
---

# Anatomy of a Skill

> **Source**: notes from *AI Builders Meetup Guadalajara #1*, hosted by Wizeline (Guadalajara, 2026-08-02).

## What Is a Skill?

A **skill** is a specialized knowledge module that Claude reads on demand, not always. Unlike [[CLAUDE-md]] (always active in the project), skills load only when the current task type matches their purpose. This lets the system's knowledge scale without saturating the context window (see [[Context-Engineering]]).

Think of skills as "procedure manuals" a specialist consults only when the task requires it — an engineer doesn't re-read the deployment manual on every line of code, only when actually deploying.

## Anatomy of SKILL.md

Every `SKILL.md` file has these mandatory sections:

### 1. Frontmatter
`name` and `description` are the fields Claude reads to decide whether this skill applies to the current task — the `description` is the single most important field in the whole file, since it acts as the trigger.

```yaml
---
name: generate-fastapi-endpoint
description: Use this skill when the user asks to create a new REST endpoint in a FastAPI project, including its Pydantic model, tests, and DB migration if applicable.
---
```

### 2. When to Use / Not Use
Explicit triggers with positive and negative examples.

```markdown
## When to use
- "create an endpoint for..."
- "I need a POST route that..."

## When NOT to use
- Modifying an existing endpoint (use direct editing, not this skill)
- Endpoints in frameworks other than FastAPI
```

### 3. Prerequisites
Setup, dependencies, required tools before running the skill.

### 4. Step-by-Step Process
Numbered, actionable instructions — not vague, but literally executable.

### 5. Example Code
Complete, functional snippets, never pseudocode.

### 6. Validation
How to verify the result is correct (tests to run, checks to make).

### 7. Troubleshooting
Common errors and their solutions.

## Skill System: Hierarchy and Types

- **Public skills** (`/mnt/skills/public/`) — shared globally, centrally maintained
- **Private skills** (`/mnt/skills/private/`) — specific to a workspace or organization
- **User skills** (`/mnt/skills/user/`) — personalized by an individual user
- **Example skills** (`/mnt/skills/examples/`) — templates and demos for learning the format

## CLAUDE.md + Skills: When to Use Each

| | CLAUDE.md | Skill |
|---|---|---|
| Frequency of use | Always relevant | Only for specific tasks |
| Typical content | Architecture, conventions, commands | Specialized step-by-step procedure |
| Ideal size | < 200 lines | Whatever the procedure requires |
| Example | "we use repository pattern" | "how to generate a formal ADR" |

**Practical rule**: if something applies to fewer than 30% of the project's tasks, it goes in a skill, not in `CLAUDE.md`. Putting it in `CLAUDE.md` dilutes the signal for the 70% of tasks where it doesn't apply.

## How a Skill Gets Invoked

1. The skill is referenced in the `<available_skills>` list of the system prompt, listed by `name` + `description`
2. Claude reads the `description` of each available skill and decides whether it's relevant to the current task
3. If it applies, Claude invokes the skill and its full content loads into context
4. The `description` is the most important field: it must act as a precise trigger — neither so generic that it fires always, nor so specific that it never fires

## Complete Skill Example

````markdown
---
name: generate-fastapi-endpoint
description: Use this skill when the user asks to create a new REST endpoint in a FastAPI project, including Pydantic model, tests, and migration if applicable.
---

# Generate FastAPI Endpoint

## When to use
- "create a GET/POST/PUT/DELETE endpoint for..."
- "I need to expose X as an API"

## When NOT to use
- Modifications to existing endpoints
- Projects that don't use FastAPI + Pydantic

## Prerequisites
- Project with `app/routers/`, `app/schemas/`, `app/repositories/` structure
- `uv` installed, dependencies synced (`uv sync`)

## Step-by-step process

1. Define the Pydantic model in `app/schemas/{resource}.py`
2. Define/reuse the repository in `app/repositories/{resource}_repository.py`
3. Create the router in `app/routers/{resource}.py`
4. Register the router in `app/main.py`
5. Write tests in `tests/routers/test_{resource}.py`
6. If the model requires a new table/column, generate a migration with Alembic

## Example code

```python
# app/schemas/product.py
from pydantic import BaseModel

class ProductCreate(BaseModel):
    name: str
    price: float
    sku: str

class ProductOut(ProductCreate):
    id: int
```

```python
# app/routers/product.py
from fastapi import APIRouter, Depends
from app.schemas.product import ProductCreate, ProductOut
from app.repositories.product_repository import ProductRepository

router = APIRouter(prefix="/products", tags=["products"])

@router.post("", response_model=ProductOut, status_code=201)
async def create_product(
    payload: ProductCreate,
    repo: ProductRepository = Depends(),
) -> ProductOut:
    return await repo.create(payload)
```

```python
# tests/routers/test_product.py
async def test_create_product(client):
    response = await client.post("/products", json={
        "name": "Widget", "price": 9.99, "sku": "WID-001",
    })
    assert response.status_code == 201
    assert response.json()["sku"] == "WID-001"
```

## Validation
- `uv run pytest tests/routers/test_{resource}.py -v` must pass
- `uv run ruff check app/routers/{resource}.py` no errors
- Verify the endpoint at `/docs` (Swagger UI) shows the correct schema

## Troubleshooting
- **422 Unprocessable Entity**: the Pydantic schema doesn't match the payload sent, check types
- **ImportError when starting the server**: router not registered in `app/main.py`
- **Test fails due to dirty DB**: verify the DB fixture rolls back between tests
````

## Anti-patterns

- Skill with a vague `description` ("helps with FastAPI") — never fires at the right moment, or fires always
- Mixing `CLAUDE.md` content inside a skill (general architecture isn't "on-demand knowledge")
- Skills without a Validation section — Claude runs the procedure but can't confirm it worked
- Pseudocode instead of functional code — forces Claude to invent syntax details

## Related Notes

- [[CLAUDE-md]] — always-active memory vs on-demand knowledge
- [[Harness-Engineering]] — how to validate that a skill produces correct results consistently
- [[Prompt-Engineering]] — a skill's `description` is, in essence, a triggering prompt
- [[Skill-examples]] — more complete, functional skills
