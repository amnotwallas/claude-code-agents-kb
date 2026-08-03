---
tags: [claude-code, process, sdd, specs, tdd, requirements]
type: process
related:
  - "[[CLAUDE-md]]"
  - "[[SDLC-with-AI]]"
  - "[[Harness-Engineering]]"
  - "[[Prompt-Engineering]]"
  - "[[SDD-examples]]"
created: 2026-08-02
status: stable
---

# Spec-Driven Development

> **Source**: notes from *AI Builders Meetup Guadalajara #1*, hosted by Wizeline (Guadalajara, 2026-08-02).

## What Is Spec-Driven Development?

A paradigm where the specification gets written **before** any code. With AI, this matters even more: Claude is very good at following a clear spec, and very bad at guessing unstated intent.

Relationship to TDD: the spec generates the tests, and the tests guide the implementation — SDD is, in a sense, a layer that precedes TDD, ensuring the tests themselves represent correct behavior, not just the behavior a developer hastily imagined while writing them.

## Why SDD Is Critical with AI

- **Without a spec**: Claude implements the most literal interpretation of your prompt, which is rarely exactly what you want — it fills ambiguity gaps with reasonable but not necessarily correct assumptions
- **With a spec**: Claude has a clear contract it can verify its own work against, reducing the correction cycle
- The spec is also a **team communication artifact** — human and agent work from the same reference document
- It enables objectively evaluating whether Claude's output delivered what was promised (direct connection to [[Harness-Engineering]]: the spec is the source of the golden set)

## The Complete SDD Process

```
Requirements (conversation/discovery)
    ↓
Spec Document (write before any code)
    ↓
Tests (generated from the spec with Claude)
    ↓
Implementation (Claude implements guided by tests + spec)
    ↓
Review (does the implementation meet the spec?)
    ↓
Update CLAUDE.md (if new architecture decisions emerged)
```

## Anatomy of a Complete Spec

### 1. Title and Context
What problem it solves, why now.

```markdown
## Title and Context
JWT authentication for the public API. Currently there's no access
control, any client can call any endpoint. Blocking issue for
launching to external customers.
```

### 2. Objective
What must be true when this is done — the **what**, not the **how**.

```markdown
## Objective
Every endpoint (except /health and /auth/*) requires a valid JWT.
Tokens expire in 15 minutes and are renewable via refresh token.
```

### 3. Functional Requirements
In Given/When/Then or User Story format.

```markdown
## Functional Requirements
- Given a user with valid credentials, When they POST to /auth/login,
  Then they receive an access_token (15 min) and a refresh_token (7 days)
- Given an expired access_token, When used on a protected endpoint,
  Then the API responds 401 with error code "token_expired"
```

### 4. Non-Functional Requirements
Performance, security, compatibility, limits.

```markdown
## Non-Functional Requirements
- JWT validation must add < 5ms of latency per request
- Refresh tokens are stored hashed in the DB, never in plain text
- Rate limit of 5 failed login attempts per IP every 15 minutes
```

### 5. API Contract
Endpoints, request/response schemas, error codes.

```markdown
## API Contract
POST /auth/login
  Request: {"email": str, "password": str}
  Response 200: {"access_token": str, "refresh_token": str, "expires_in": int}
  Response 401: {"error": "invalid_credentials"}
```

### 6. Data Models
Entities, relationships, validations.

```markdown
## Data Models
RefreshToken:
  - id: UUID
  - user_id: UUID (FK to users)
  - token_hash: str (SHA-256 of the token)
  - expires_at: datetime
  - revoked: bool (default false)
```

### 7. Edge Cases and Error Handling
What happens when something fails.

```markdown
## Edge Cases and Error Handling
- Refresh token already used (rotation): revoke the user's entire
  token chain, force new login (mitigates token theft)
- User deleted with active tokens: all their tokens must be
  invalidated immediately, not wait for natural expiration
```

### 8. Out of Scope
Explicitly what this spec does NOT include.

```markdown
## Out of Scope
- OAuth with external providers (Google, GitHub) — separate future spec
- 2FA — not included in this iteration
```

### 9. Acceptance Criteria
Verifiable "done" checklist.

```markdown
## Acceptance Criteria
- [ ] All protected endpoints reject requests without a valid JWT
- [ ] Integration tests cover login, refresh, logout, and expiration
- [ ] Rate limiting verified with a load test
- [ ] API documentation updated (OpenAPI schema)
```

## Iterating on Specs with Claude

Actively use Claude to refine the spec before implementing: detect ambiguities, generate edge cases you hadn't thought of, question unclear requirements.

```
You are a senior QA engineer. Review this JWT authentication spec and
find: (1) ambiguities a developer could interpret in more than one
way, (2) cases not covered in Edge Cases, (3) possible security flaws
in the proposed design. Don't implement anything, just flag problems
in the spec itself.
```

## Integration with CLAUDE.md

- Reference specs in [[CLAUDE-md]] — don't copy their content, just point to the location (`docs/specs/auth-jwt.md`)
- Update CLAUDE.md when the spec produces new architecture decisions that should persist beyond this specific feature (e.g., "all tokens are hashed with SHA-256 before storage" becomes a general convention, not just for this spec)

## Anti-patterns

- Starting to code with a half-written spec "to not waste time" — the time lost re-implementing due to misunderstandings is greater
- A spec that describes the **how** (implementation) instead of the **what** (expected behavior) — takes away Claude's freedom to find the best implementation
- Non-verifiable Acceptance Criteria ("must be fast" instead of "p95 latency < 200ms")
- Not keeping the spec synchronized with the final implementation — it becomes lying documentation

## Related Notes

- [[CLAUDE-md]] — where to reference specs from persistent project memory
- [[SDLC-with-AI]] — SDD as the Design phase within the full lifecycle
- [[Harness-Engineering]] — the spec is the source of truth for building the eval golden dataset
- [[Prompt-Engineering]] — how to use Claude to review and refine the spec itself
- [[SDD-examples]] — complete example specs (JWT auth, async pipeline)
