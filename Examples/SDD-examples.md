---
tags: [claude-code, examples, sdd, specs]
type: example
related:
  - "[[Spec-Driven-Development]]"
  - "[[Harness-Engineering]]"
created: 2026-08-02
status: stable
---

# Specs — Complete Examples

> **Source**: notes from *AI Builders Meetup Guadalajara #1*, hosted by Wizeline (Guadalajara, 2026-08-02).

Two complete specs following the anatomy defined in [[Spec-Driven-Development]], including how Claude used them to generate tests first.

## Spec 1: JWT Authentication API

````markdown
# Spec: JWT Authentication API

## Title and Context
JWT authentication for the public API. Currently there's no access
control; blocking issue for launching to external customers.

## Objective
Every endpoint (except /health and /auth/*) requires a valid
access_token JWT. Tokens expire in 15 minutes, renewable via a
7-day refresh_token with rotation.

## Functional Requirements
- Given valid credentials, When POST /auth/login,
  Then responds with access_token, refresh_token, expires_in
- Given invalid credentials, When POST /auth/login,
  Then responds 401 {"error": "invalid_credentials"}, without
  distinguishing whether the email exists (avoids user enumeration)
- Given a valid, unused refresh_token, When POST /auth/refresh,
  Then responds with a new access_token + new refresh_token, revokes the old one
- Given an already-used refresh_token (reuse), When POST /auth/refresh,
  Then revokes the user's ENTIRE token chain and responds 401
- Given a valid access_token, When POST /auth/logout,
  Then revokes the associated refresh_token

## Non-Functional Requirements
- JWT validation adds < 5ms of latency per request
- Refresh tokens are stored hashed (SHA-256), never in plain text
- Rate limit: 5 failed login attempts per IP every 15 minutes

## API Contract
```
POST /auth/login
  Request: {"email": str, "password": str}
  Response 200: {"access_token": str, "refresh_token": str, "expires_in": 900}
  Response 401: {"error": "invalid_credentials"}
  Response 429: {"error": "rate_limited", "retry_after": int}

POST /auth/refresh
  Request: {"refresh_token": str}
  Response 200: {"access_token": str, "refresh_token": str, "expires_in": 900}
  Response 401: {"error": "invalid_or_reused_token"}

POST /auth/logout
  Headers: Authorization: Bearer {access_token}
  Response 204
```

## Data Models
```
RefreshToken:
  - id: UUID
  - user_id: UUID (FK to users)
  - token_hash: str (SHA-256)
  - expires_at: datetime
  - revoked: bool (default false)
  - replaced_by: UUID | null (to trace the rotation chain)
```

## Edge Cases and Error Handling
- Reused refresh token: revoke the entire chain, force new login
  (token theft mitigation)
- User deleted with active tokens: invalidate all their tokens
  immediately
- Clock skew between servers: use a 30s tolerance when validating `exp`

## Out of Scope
- OAuth with external providers — separate future spec
- 2FA — not included in this iteration

## Acceptance Criteria
- [ ] All protected endpoints reject requests without a valid JWT
- [ ] Integration tests cover login, refresh, reuse detection, logout
- [ ] Rate limiting verified with a load test
- [ ] OpenAPI schema updated
````

### How Claude Used the Spec to Generate Tests First

From the Functional Requirements, Claude generated tests **before** touching the implementation:

```python
# tests/test_auth.py — generated from the spec, before writing the code

async def test_login_success_returns_tokens(client, test_user):
    response = await client.post("/auth/login", json={
        "email": test_user.email, "password": "correct-password",
    })
    assert response.status_code == 200
    body = response.json()
    assert "access_token" in body and "refresh_token" in body
    assert body["expires_in"] == 900

async def test_login_invalid_credentials_returns_generic_error(client, test_user):
    response = await client.post("/auth/login", json={
        "email": "nonexistent@example.com", "password": "whatever",
    })
    assert response.status_code == 401
    assert response.json() == {"error": "invalid_credentials"}

async def test_refresh_token_reuse_revokes_entire_chain(client, test_user):
    login = await client.post("/auth/login", json={
        "email": test_user.email, "password": "correct-password",
    })
    old_refresh = login.json()["refresh_token"]

    # First refresh: valid
    first_refresh = await client.post("/auth/refresh", json={"refresh_token": old_refresh})
    assert first_refresh.status_code == 200
    new_refresh = first_refresh.json()["refresh_token"]

    # Reuse the old token: should revoke the entire chain
    reuse_attempt = await client.post("/auth/refresh", json={"refresh_token": old_refresh})
    assert reuse_attempt.status_code == 401

    # The "new" token generated in the first refresh should also be revoked
    second_attempt_with_new = await client.post("/auth/refresh", json={"refresh_token": new_refresh})
    assert second_attempt_with_new.status_code == 401
```

The reuse-detection test exists **directly** because the spec's Edge Case demands it explicitly — without the spec, it's an easy case to miss when implementing from memory.

---

## Spec 2: Async Data Processing Pipeline

````markdown
# Spec: Async Data Processing Pipeline

## Title and Context
Pipeline for processing CSV files uploaded by customers (up to 500k
rows) without blocking the HTTP request. Processing is currently
synchronous and causes timeouts on large files.

## Objective
An uploaded file gets queued, processed by async workers, and the
client can poll the processing status. Partial failures don't
discard the whole file.

## Functional Requirements
- Given a valid file, When POST /uploads,
  Then responds 202 with {"job_id": str, "status": "queued"}
- Given an existing job_id, When GET /uploads/{job_id},
  Then responds {"status": "queued"|"processing"|"completed"|"failed",
  "progress": {"processed": int, "total": int, "failed_rows": int}}
- Given an individual invalid row within an otherwise valid file,
  When processed, Then that row goes to a dead letter queue and the
  rest of the file keeps processing

## Non-Functional Requirements
- A 500k-row file must complete in under 10 minutes
- The system supports at least 20 files queued simultaneously
- Automatic retries: 3 attempts with exponential backoff for
  transient failures (e.g., DB timeout), no retry for rows with
  validation errors

## API Contract
```
POST /uploads
  Request: multipart/form-data, file=csv
  Response 202: {"job_id": str, "status": "queued"}
  Response 400: {"error": "invalid_file_format"}

GET /uploads/{job_id}
  Response 200: {
    "status": "queued"|"processing"|"completed"|"failed",
    "progress": {"processed": int, "total": int, "failed_rows": int}
  }
  Response 404: {"error": "job_not_found"}
```

## Data Models
```
UploadJob:
  - id: UUID
  - status: enum(queued, processing, completed, failed)
  - total_rows: int
  - processed_rows: int
  - failed_rows: int
  - created_at: datetime
  - completed_at: datetime | null

DeadLetterRow:
  - id: UUID
  - job_id: UUID (FK)
  - row_number: int
  - raw_data: str
  - error_reason: str
```

## Edge Cases and Error Handling
- Empty file (0 rows): responds 400 immediately, doesn't get queued
- Worker crashes mid-processing: the job must be resumable from the
  last confirmed row, not reprocess everything from scratch
- More than 50% of rows fail: mark the entire job as `failed` instead
  of `completed`, even though it technically finished processing all rows

## Out of Scope
- Processing formats other than CSV (Excel, JSON) — future spec
- Push notifications on completion — the client polls in this iteration

## Acceptance Criteria
- [ ] A 500k-row file processes in < 10 min on staging
- [ ] An individual invalid row doesn't abort the rest of the file
- [ ] A crashed worker resumes without reprocessing already-confirmed rows
- [ ] A job with >50% failures gets marked as `failed`
````

### How Claude Used the Spec to Generate Tests First

```python
# tests/test_pipeline.py — generated from Edge Cases before implementing

async def test_partial_row_failure_does_not_abort_file(pipeline, csv_with_one_bad_row):
    job = await pipeline.submit(csv_with_one_bad_row)
    await pipeline.wait_until_done(job.id)

    result = await pipeline.get_status(job.id)
    assert result.status == "completed"
    assert result.progress.failed_rows == 1
    assert result.progress.processed == result.progress.total

async def test_majority_failure_marks_job_as_failed(pipeline, csv_mostly_invalid):
    job = await pipeline.submit(csv_mostly_invalid)
    await pipeline.wait_until_done(job.id)

    result = await pipeline.get_status(job.id)
    assert result.status == "failed"
    assert result.progress.failed_rows > result.progress.total / 2

async def test_worker_crash_resumes_without_reprocessing(pipeline, large_csv, simulate_crash_at_row):
    job = await pipeline.submit(large_csv)
    simulate_crash_at_row(job.id, row=1000)

    await pipeline.resume(job.id)
    await pipeline.wait_until_done(job.id)

    result = await pipeline.get_status(job.id)
    assert result.status == "completed"
    # Verifies the first 1000 rows weren't double-processed
    assert result.progress.processed == result.progress.total
```

The "worker crash resumes" test exists directly because the Edge Case specifies it — exactly the kind of case that gets forgotten without a spec written in advance.

## Related Notes

- [[Spec-Driven-Development]] — full anatomy and process of SDD
- [[Harness-Engineering]] — how these spec-generated tests feed the eval golden dataset
