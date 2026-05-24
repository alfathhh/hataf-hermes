---
name: backend-proper
description: Bikin / refactor / debug backend service dengan disiplin engineering — security, error handling, observability, idempotency, testing.
version: 2.0.0
metadata:
  hermes:
    tags: [backend, api, database, security, devops, observability]
    category: development
---

# Backend Proper

## KAPAN PAKAI

```
IF user minta "bikin API" OR "endpoint" OR "service" → PAKAI
IF user minta "design DB schema" → PAKAI
IF user minta "tambah integrasi" (payment, email, queue) → PAKAI
IF user minta "refactor backend" → PAKAI
IF user minta "debug 500 / data inconsistency" → PAKAI
IF user minta frontend/UI → JANGAN (pakai web-development / frontend-design)
IF user minta quick prototype throwaway → JANGAN (overkill)
```

---

## PROCEDURE (ikuti exact)

### Step 1: Stack discovery

```bash
ls go.mod pyproject.toml requirements.txt package.json
grep -i "fastapi\|express\|gin\|chi\|nestjs" package.json pyproject.toml go.mod 2>/dev/null
grep -i "prisma\|drizzle\|sqlx\|sqlalchemy\|gorm" package.json pyproject.toml go.mod 2>/dev/null
```

CATAT:
- Language + version
- HTTP framework
- DB driver / ORM
- Auth approach (JWT / session / OAuth)
- Logging (stdout / structured / Sentry)

### Step 2: Spec sebelum kode (untuk endpoint baru)

```
TULIS spec ini, TUNJUKKAN ke user, TUNGGU approve:

## Spec: [METHOD] [PATH]
- Auth: [required/optional, scope apa]
- Input: [body schema dengan types]
- Response: [status codes + body]
- Side effects: [DB writes, events, logs]
- Constraints: [rate limit, idempotency, transaction]
```

### Step 3: Implementation checklist

```
UNTUK setiap endpoint, cek SEMUA ini:

AUTH:
□ Authentication (siapa user) → verified
□ Authorization (apa yang user boleh) → verified
□ Token validation (signature, expiry) → verified
DO NOT: trust client claims (X-User-Id header dari frontend)

INPUT VALIDATION:
□ Schema validation (Zod/Pydantic/validator) → strict
□ Length limits → set (default max 1000 char)
□ Type coercion explicit → no implicit
□ Business rules → validated

DATABASE:
□ Parameterized queries → ALWAYS (prevent SQL injection)
□ Transaction untuk multi-step write → yes
□ Index pada WHERE/JOIN columns → verified
□ Connection pooling → configured
DO NOT: SELECT * (explicit columns only)
DO NOT: loop-and-query N+1 (pakai JOIN/batch)

ERROR HANDLING:
□ Try-catch sekitar IO → yes
□ Business error (4xx) vs system error (5xx) → distinguished
□ Error response sanitized → no stack trace leak
□ Log dengan request_id → yes
DO NOT: swallow error diam-diam (except: pass = BAHAYA)

OBSERVABILITY:
□ Structured log (JSON, request_id, user_id, latency_ms)
□ Metrics (request count, latency p50/p95, error rate)
□ Log level tepat (INFO normal, WARN edge, ERROR failure)
DO NOT: log credential / token / PII

IDEMPOTENCY (untuk POST/PUT/PATCH/DELETE):
□ Support Idempotency-Key header
□ Store key + response 24-48 jam
□ Return cached response untuk retry
```

### Step 4: Write code

```
RULE: Match existing patterns di repo (read 2-3 existing handlers dulu)
RULE: Buat file baru sesuai convention project
RULE: Include error handling dari awal (bukan nanti)
```

### Step 5: Test

```
MINIMAL:
□ Unit test pure logic
□ Integration test happy path + 1 error path
□ Test idempotency (call 2x → 1 effect)
```

### Step 6: Verify sebelum deliver

---

## OUTPUT TEMPLATE

```markdown
## Implementasi: [METHOD] [PATH]

**Stack**: [language + framework + DB]

**File baru**:
- `path/handler.go` — handler
- `path/service.go` — business logic
- `path/queries.sql` — DB queries
- `path/handler_test.go` — tests

**Spec implemented**:
- ✅ Auth via [method]
- ✅ Input validated ([library])
- ✅ Idempotency-Key handled
- ✅ Structured log dengan request_id
- ✅ Rate limit [N] RPS/user

**Test**:
- ✅ Unit test service
- ✅ Integration test
- ✅ Idempotency test

**Yang perlu user verify**:
- [hal yang butuh confirm dari user]
```

---

## CONTOH OUTPUT

```markdown
## Implementasi: POST /api/v1/orders

**Stack**: Go 1.22 + chi + sqlc + Postgres 16

**File baru**:
- `internal/handler/order.go` — handler
- `internal/service/order.go` — business logic
- `internal/repo/queries/orders.sql` — sqlc query
- `internal/handler/order_test.go` — tests

**Spec implemented**:
- ✅ Auth via JWT scope `order:create`
- ✅ Input validated (go-playground/validator)
- ✅ Idempotency-Key handled (Redis 24h cache)
- ✅ Stock decrement atomic (UPDATE WHERE stock > 0)
- ✅ Event order.created → RabbitMQ
- ✅ Structured log (zap) + request_id
- ✅ Rate limit 100 RPS/user (Redis sliding window)

**Test**:
- ✅ Unit test service layer
- ✅ Integration test (testcontainers Postgres + Redis)
- ✅ Idempotency test (call 2x → 1 order)
- ✅ Race test (10 concurrent → stock konsisten)

**Yang perlu user verify**:
- Rate limit 100 RPS sesuai capacity production?
- RabbitMQ exchange `orders.events` udah ada di prod?
```

---

## DECISION TREE: Database

```
IF butuh create table:
  - Naming: snake_case, plural (users, orders)
  - SELALU include: id, created_at, updated_at
  - ID: UUID v7 atau bigint serial
  - Timestamps: timestamptz (UTC)
  - Money: decimal(19,4) atau integer cents (BUKAN float)
  - Enum: DB enum type, bukan varchar

IF butuh migration:
  - Forward-only
  - Test rollback plan
  - Avoid long-running lock (CREATE INDEX CONCURRENTLY)
  - Versioned (Flyway / Alembic / Prisma Migrate)

IF butuh index:
  - Primary key: auto-indexed
  - Foreign key: index MANUAL (DB gak auto-index FK)
  - WHERE/JOIN column: index
  - Composite: most-selective column first
```

## DECISION TREE: Error Response

```
IF validation error → 400 + detail field mana yang salah
IF no auth → 401
IF forbidden → 403
IF not found → 404
IF conflict (idempotency reuse) → 409
IF business error (out of stock) → 422
IF rate limit → 429 + Retry-After header
IF server error → 500 + request_id (DO NOT leak internal detail)
```

---

## VERIFICATION

```
□ Semua endpoint require auth yang relevan?
□ Authz dicek di handler (bukan cuma middleware)?
□ Input validation strict?
□ Error response gak leak internal detail?
□ Log gak ngandung secret / PII?
□ Test ada untuk happy + error path?
□ Migration backward-compat?

IF ada □ TIDAK → fix sebelum deliver
```
