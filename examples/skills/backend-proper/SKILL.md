---
name: backend-proper
description: Bikin / refactor / debug backend service dengan disiplin engineering yang serius — security, error handling, observability, idempotency, dan testing. Reject quick-and-dirty pattern yang ke-deploy tanpa mikir failure mode.
version: 1.0.0
metadata:
  hermes:
    tags: [backend, api, database, security, devops, observability]
    category: development
---

# Backend Proper

Skill untuk task backend serius: API design, database schema, integration, refactor, debug. Skill ini tidak akan bikin endpoint tanpa mikir auth, validasi, error handling, dan failure mode. Ngotot di dasar engineering yang sering di-skip.

## When to Use

User minta:
- Bikin API endpoint / service
- Design DB schema
- Tambah integrasi (payment, email, queue)
- Refactor existing backend
- Debug 500 error / data inconsistency / latency issue
- Security audit endpoint

JANGAN pakai untuk:
- Frontend / UI (pake `web-development` atau `ui-ux`)
- Pure scripting / one-off automation
- "Quick prototype throwaway" (skill ini overkill, user prefer raw)

## Filosofi

1. **Boring is good**. Standard pattern, well-tested library, predictable behavior.
2. **Failure mode pertama, happy path kedua**. Apa yang terjadi kalau DB down? Network split? Disk full?
3. **Idempotency by default**. Operasi yang affecting state harus survive retry.
4. **Observability built-in**, bukan ditambahin nanti.
5. **Security default-deny**, bukan default-allow.

## Stack-aware Procedure

### 1. Stack discovery

```bash
# Cek stack
ls go.mod pyproject.toml requirements.txt package.json Gemfile pom.xml

# Cek framework
grep -i "fastapi\|django\|flask\|express\|nestjs\|gin\|chi\|fiber\|axum\|rocket" \
  package.json pyproject.toml go.mod 2>/dev/null

# Cek DB layer
grep -i "prisma\|drizzle\|sqlx\|sqlc\|sqlalchemy\|gorm\|typeorm\|knex\|pg\|mongoose" \
  package.json pyproject.toml go.mod 2>/dev/null

# Cek deploy target
ls Dockerfile docker-compose.* .github/workflows/ Procfile vercel.json railway.toml
```

Catat:
- Language + version
- HTTP framework (FastAPI, chi, Express, Axum, ...)
- DB driver / ORM
- DB engine (Postgres / MySQL / SQLite / Mongo / Redis)
- Auth approach (JWT / session / OAuth)
- Secret management (env / Vault / secret manager)
- Logging (stdout / structured logger / Sentry)
- Deployment (container / serverless / VPS)

### 2. Spesifikasi sebelum kode

Untuk endpoint baru / non-trivial change:

```markdown
## Spec: POST /orders

### Request
- Method: POST
- Path: /api/v1/orders
- Auth: Bearer JWT (scope: `order:create`)
- Idempotency-Key header: required (UUID)
- Body schema:
  - product_id: string (uuid)
  - quantity: integer (1-1000)
  - shipping_address_id: string (uuid)

### Response
- 201 Created → Order DTO
- 400 → validation error
- 401 → no token
- 403 → insufficient scope
- 404 → product not found
- 409 → conflict (idempotency-key reused with different body)
- 422 → out of stock
- 429 → rate limit hit
- 5xx → server error (retryable kalau body bilang retryable: true)

### Side effects
- INSERT order, order_items
- DECREMENT product.stock atomically
- PUBLISH event order.created ke queue
- LOG dengan request_id

### Constraints
- Maximum 100 RPS per user
- Transactional (semua sukses atau semua rollback)
- Idempotent: retry dengan key sama → kembalikan response asli, bukan duplicate order
```

Tunjukkan spec ke user, dapat approve, baru implement.

### 3. Implementation checklist (per endpoint)

#### Auth + AuthZ
- ✅ Authentication (siapa user)
- ✅ Authorization (apa yang user boleh — scope / role / resource ownership)
- ✅ Token validation (signature, expiry, issuer, audience)
- ❌ Jangan trust client claims (`X-User-Id` header dari frontend = no-go)

#### Input validation
- ✅ Schema validation (Zod / Pydantic / class-validator) — strict, reject extra field
- ✅ Length limits (string max 1000 char default, lebih kalau dijustifikasi)
- ✅ Type coercion explicit (string "true" ≠ boolean true)
- ✅ Business rule validation (quantity > 0, email valid, dst)

#### Database access
- ✅ Parameterized queries (PREVENT SQL injection — never string concat)
- ✅ Transaction untuk multi-step write
- ✅ Index pada column yang di-WHERE / JOIN
- ✅ Connection pooling configured
- ❌ Jangan SELECT * di production code (explicit columns)
- ❌ Jangan loop-and-query (N+1) — pakai JOIN atau batch fetch

#### Error handling
- ✅ Try-catch sekitar IO (DB, network, file)
- ✅ Distinguish business error (4xx) vs system error (5xx)
- ✅ Sanitize error response — JANGAN leak stack trace / DB query / internal path ke user
- ✅ Log dengan request_id biar bisa di-trace
- ❌ Jangan swallow error diam-diam (`except: pass` = bahaya)

#### Observability
- ✅ Structured logging (JSON, dengan request_id, user_id, latency_ms)
- ✅ Metrics (request count, latency p50/p95/p99, error rate)
- ✅ Trace untuk operasi penting (DB call, external API call)
- ✅ Log level tepat (INFO untuk normal, WARN untuk edge case, ERROR untuk failure)
- ❌ JANGAN log credential / token / PII ke logs

#### Idempotency (untuk POST/PUT/PATCH/DELETE)
- ✅ Endpoint yang affect state: support `Idempotency-Key` header
- ✅ Store key + response 24-48 jam, return cached response untuk retry
- ✅ Webhook handler: dedup by event ID

#### Rate limiting
- ✅ Per IP + per user
- ✅ 429 response dengan `Retry-After` header
- ✅ Different tier untuk auth vs anonymous

#### Async / queue
- ✅ Operasi panjang → queue (RabbitMQ, Redis, SQS, BullMQ, dst), bukan sync
- ✅ Worker idempotent
- ✅ DLQ untuk job yang gagal terus
- ✅ Retry dengan exponential backoff

#### Test
- ✅ Unit test pure logic
- ✅ Integration test dengan real DB (testcontainers atau test schema)
- ✅ E2E test happy path + 1-2 error path
- ✅ Test idempotency (call 2x dengan key sama → 1 effect)

#### Security checklist
- ✅ HTTPS only (HSTS header)
- ✅ CORS specific origin (jangan `*` di production)
- ✅ CSRF protection untuk session-based auth
- ✅ Password hash dengan bcrypt/argon2 (jangan MD5/SHA1)
- ✅ Secrets dari env / secret manager (jangan hardcode)
- ✅ Dependency scan (pip-audit / npm audit / govulncheck)
- ✅ SQL injection: parameterized queries
- ✅ XSS: escape user input di output
- ✅ Open redirect: validate redirect URL whitelist

### 4. Deploy considerations

- ✅ Health endpoint (`/health` atau `/healthz`) — return 200 kalau service ready
- ✅ Graceful shutdown (drain in-flight request)
- ✅ Migration strategy (forward-only? backward compat?)
- ✅ Rollback plan
- ✅ Resource limits (CPU/memory di container manifest)
- ✅ Backup strategy untuk data tier

## Pitfalls

### Pitfall 1: SELECT * + frontend pretend-it's-fine

DB schema berubah → response bocor field private. Selalu eksplisit kolom + DTO mapping.

### Pitfall 2: N+1 query

```python
# Bad
orders = db.query("SELECT * FROM orders WHERE user_id = ?", uid)
for order in orders:
    items = db.query("SELECT * FROM order_items WHERE order_id = ?", order.id)
```

10 order → 11 query. 1000 order → catastrophic.

Fix: JOIN, atau batch fetch (`WHERE order_id IN (...)`).

### Pitfall 3: Transaction boundary salah

```python
# Bad: race condition kalau check stock di luar transaction
if get_stock(product_id) > 0:
    decrement_stock(product_id)
    create_order(...)
```

Fix: gabung dalam transaction + lock row, atau atomic decrement (`UPDATE ... SET stock = stock - 1 WHERE id = ? AND stock > 0`).

### Pitfall 4: Webhook tanpa verifikasi signature

Stripe / GitHub / Discord webhook → siapa pun bisa fake call. Selalu verify HMAC signature pakai secret webhook.

### Pitfall 5: Env var salah scope

`DATABASE_URL` di client-side bundle (Next.js public env) → DB credential leak. Cek strategy framework dengan teliti.

### Pitfall 6: Caching tanpa invalidation strategy

Cache itu mudah, invalidate yang susah.

```
Sebelum bikin cache:
1. TTL berapa? (5 menit? 1 jam? sampai event tertentu?)
2. Cara invalidate kalau data berubah?
3. Cache miss path tetap aman / cepat?
4. Stampede protection (10K concurrent miss bareng)?
```

Kalau gak ada jawaban semua, **jangan cache dulu** — measure first.

### Pitfall 7: Authorization di middleware aja

```python
# Bad
@requires_login  # cek auth doang
def get_order(order_id):
    return db.query("SELECT * FROM orders WHERE id = ?", order_id)
```

User A authenticated bisa fetch order user B. **Authentication ≠ authorization**. Cek ownership / scope di handler.

### Pitfall 8: Error message bocor info

```
500 Internal Server Error
Error: connection to "db.internal:5432" failed: password authentication failed for user "app_user"
```

Bocor: hostname internal, port, username DB. Sanitize: `Internal error. Request ID: xyz`. Detail di log internal.

### Pitfall 9: Microservices premature

3-orang team butuh microservices? Probably not. Modular monolith dulu sampai pain points jelas justify split.

### Pitfall 10: ORM black-box

ORM nyaman, tapi generated SQL bisa horrible (N+1 hidden, full table scan). Inspect query yang dihasilkan untuk hot path.

## Verification

Sebelum mark "selesai":

1. Apakah semua endpoint require auth yang relevan?
2. Apakah authz dicek di handler (bukan cuma middleware)?
3. Apakah input validation strict?
4. Apakah error response gak leak internal detail?
5. Apakah log gak ngandung secret / PII?
6. Apakah unit + integration test ada untuk happy + error path?
7. Apakah migration backward-compat (kalau zero-downtime deploy)?
8. Apakah ada test idempotency untuk operasi state-changing?
9. Apakah perf tested untuk realistic load (kalau public-facing)?

Kalau ada satu "tidak" — tahan deploy, fix dulu.

## Database Schema Tips

### Naming
- Snake_case table & column
- Plural table (`users`, `orders`)
- Timestamps di setiap table (`created_at`, `updated_at`)
- Soft delete jika perlu (`deleted_at` nullable)

### Type
- ID: UUID v7 (time-ordered) atau bigint serial
- Timestamps: `timestamptz` (Postgres) — selalu UTC
- Money: `decimal(19,4)` atau pakai integer cents (jangan float — precision rusak)
- Enum: `enum` type Postgres atau check constraint, bukan `varchar`

### Indexing
- Primary key: auto-indexed
- Foreign key: index manual (DB tidak auto-index FK)
- Frequently filtered column: index
- Composite index urutan kolom matter (most-selective first)

### Migrations
- Forward-only (backward via revert deploy)
- Test rollback plan (kalau possible)
- Avoid long-running lock pada table besar (gunakan `CREATE INDEX CONCURRENTLY` di Postgres)
- Versioned (Flyway / Alembic / Prisma Migrate / sql-migrate)

## Security Reminders (constant)

- Secrets dari env, never commit
- Update dependency rutin (`npm audit fix`, `pip-audit`, dst)
- Logging: redact field sensitive (token, password, NIK, no kartu)
- Rate limit di edge (CDN / WAF) DAN di app — jangan andalkan satu layer
- Backup terenkripsi, restore tested berkala
- Insiden response plan tertulis (siapa pertama, kemana eskalasi)

## Contoh Output

```markdown
## Implementasi: POST /api/v1/orders

**Stack**: Go 1.22 + chi + sqlc + Postgres 16 + Redis (rate limit)

**File baru**:
- `internal/handler/order.go` (handler)
- `internal/service/order.go` (business logic)
- `internal/repo/queries/orders.sql` (sqlc query)
- `internal/repo/orders.go` (generated by sqlc)
- `internal/handler/order_test.go` (integration test)

**Migration**:
- `migrations/0042_create_orders.up.sql`
- `migrations/0042_create_orders.down.sql`

**Spec implemented**:
- ✅ Auth via JWT + scope `order:create`
- ✅ Input validated (go-playground/validator)
- ✅ Idempotency-Key handled (Redis cache 24h)
- ✅ Stock decrement atomic (UPDATE ... WHERE stock > 0)
- ✅ order.created event published ke RabbitMQ
- ✅ Structured log (zap) dengan request_id
- ✅ Metrics (Prometheus): request_count, latency_seconds, error_rate
- ✅ Rate limit 100 RPS/user (Redis sliding window)

**Test**:
- ✅ Unit test service
- ✅ Integration test dengan testcontainers Postgres + Redis
- ✅ Idempotency test (call 2x → 1 order, response sama)
- ✅ Race test (10 concurrent → stock konsisten)

**Deploy notes**:
- Migration zero-downtime (no destructive change)
- Konsumer event order.created harus deployed dulu (atau toleran event tanpa konsumer)

**Yang perlu user verify**:
- Rate limit value 100 RPS sesuai capacity production?
- Scope JWT `order:create` ada di OIDC config?
- RabbitMQ exchange `orders.events` udah dibuat di prod?
```

## Untuk Debug Production Issue

Workflow:

1. **Liat metric** — dimana lonjakan? Endpoint mana? Spike kapan?
2. **Cari log error** — request_id pada error → trace flow
3. **Reproduce dulu kalau bisa** — di staging dengan data anonymized
4. **Hipotesis sebelum fix** — apa root cause? bukan symptom
5. **Fix minimal** — jangan refactor 50 file untuk fix 1 bug
6. **Tambah test regresi** — biar bug yang sama gak balik

**JANGAN**:
- Push fix ke production tanpa test
- Restart service tanpa investigasi (just hides root cause)
- Add try-except global untuk silence error tanpa cari kenapa
