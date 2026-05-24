---
name: code-review
description: Review kode seperti senior engineer. Cari bug, security issue, performance problem. Output prioritized by severity.
version: 2.0.0
metadata:
  hermes:
    tags: [review, code-quality, security, performance, refactor]
    category: development
    requires_toolsets: [core]
---

# Code Review

## KAPAN PAKAI

```
IF user minta "review" OR "audit" OR "cek kode" OR "PR review" → PAKAI
IF sebelum deploy kode penting → PAKAI
IF user minta refactor assessment → PAKAI
ELSE → jangan pakai
```

---

## PROCEDURE (ikuti urutan exact)

### Step 1: Baca SEMUA file dulu

```
DO: read_file() untuk SEMUA file yang di-review
DO NOT: komentar per baris sequential tanpa baca keseluruhan
DO NOT: review isolated function tanpa liat caller-nya
```

### Step 2: Pahami context

Jawab 3 pertanyaan ini (internal, gak perlu tulis ke user):
1. Apa tujuan kode ini?
2. Gimana flow datanya?
3. Apa yang berubah dari sebelumnya (kalau diff)?

### Step 3: Scan checklist per kategori

#### Security scan:
```
□ SQL injection (string concat ke query?)
□ XSS (user input ke HTML tanpa escape?)
□ Auth bypass (endpoint tanpa auth check?)
□ Secret hardcoded (API key, password di kode?)
□ Path traversal (user input ke file path?)
□ SSRF (user-controlled URL di server-side fetch?)
□ Mass assignment (user bisa set field restricted?)
```

#### Correctness scan:
```
□ Off-by-one error
□ Null/undefined not handled
□ Race condition (concurrent access shared state?)
□ Exception swallowed (catch tanpa rethrow/log?)
□ Type mismatch
□ Edge case: empty list, zero value, max int, unicode
```

#### Performance scan:
```
□ N+1 query (loop → DB call per iteration?)
□ Missing index (WHERE/JOIN tanpa index?)
□ Unbounded result set (SELECT tanpa LIMIT?)
□ Memory leak (event listener tanpa cleanup?)
□ Blocking main thread (sync IO di async?)
□ Large payload tanpa pagination
```

### Step 4: Classify severity

```
IF bug / security / data loss → 🔴 CRITICAL (harus fix sebelum deploy)
IF performance cliff / race condition / error handling hilang → 🟠 HIGH (sangat disarankan fix)
IF maintainability / unclear naming / missing test → 🟡 MEDIUM (fix kalau sempat)
IF style preference / minor optimization → 🟢 LOW (nice-to-have)
```

### Step 5: Format output

```
RULE: Lead dengan severity tertinggi
RULE: Setiap issue HARUS punya fix suggestion (bukan cuma "fix this")
RULE: Acknowledge yang bagus (bukan pure criticism)
RULE: IF gak ada issue serius → bilang "gak nemu critical/high issue"
DO NOT: bikin-bikin issue biar keliatan thorough
```

---

## OUTPUT TEMPLATE

```markdown
## Code Review: [file/PR name]

### Summary
[1-2 kalimat: overall impression + highest severity issue]

### 🔴 Critical

#### [Issue title]
**File**: `path/to/file.ts:42`
**Apa**: [deskripsi masalah, 1-2 kalimat]
**Impact**: [apa yang terjadi kalau gak difix]
**Fix**:
```[language]
// kode fix yang bisa langsung dipake
```

### 🟠 High

#### [Issue title]
**File**: `path/to/file.ts:78`
**Apa**: [deskripsi]
**Impact**: [impact]
**Fix**:
```[language]
// kode fix
```

### 🟡 Medium
- [issue singkat] — `file:line` — fix: [saran 1 kalimat]

### ✅ Yang udah bagus
- [item positif 1]
- [item positif 2]

### ❓ Questions (butuh klarifikasi)
- [hal yang gw gak sure tanpa lebih banyak context]
```

---

## CONTOH OUTPUT

```markdown
## Code Review: `src/handlers/order.ts`

### Summary
Ada SQL injection vulnerability di line 23. Selain itu, structure cukup bersih.

### 🔴 Critical

#### SQL Injection di createOrder
**File**: `src/handlers/order.ts:23`
**Apa**: String concat langsung ke SQL query dari user input
**Impact**: Attacker bisa dump/delete seluruh database
**Fix**:
```typescript
// SEBELUM (vulnerable):
const result = await db.query(`SELECT * FROM products WHERE id = '${req.body.productId}'`);

// SESUDAH (parameterized):
const result = await db.query('SELECT * FROM products WHERE id = $1', [req.body.productId]);
```

### 🟠 High

#### N+1 query di getOrderItems
**File**: `src/handlers/order.ts:45-52`
**Apa**: Loop fetch items per order, 100 orders = 101 queries
**Impact**: Endpoint jadi lambat 5-10x di production load
**Fix**:
```typescript
// SEBELUM (N+1):
for (const order of orders) {
  const items = await db.query('SELECT * FROM items WHERE order_id = $1', [order.id]);
}

// SESUDAH (batch):
const items = await db.query('SELECT * FROM items WHERE order_id = ANY($1)', [orderIds]);
```

### ✅ Yang udah bagus
- Error handling consistent (semua endpoint pake try-catch)
- TypeScript strict mode, no `any`
- Input validation pake Zod schema
```

---

## DECISION TREE: Edge Cases

```
IF review tanpa context (gak tau siapa caller-nya):
  → bilang "gw butuh liat file [X] juga untuk review lengkap"

IF gak ada issue serius:
  → bilang "gak nemu critical/high issue" — jangan fabricate

IF cuma style issues (indentation, naming preference):
  → JANGAN report sebagai masalah utama
  → taruh di 🟢 Low atau skip entirely

IF file terlalu besar (>500 baris):
  → fokus ke hot path (handler utama, business logic)
  → skip utility/helper yang gak dipanggil dari critical path
```

---

## VERIFICATION

```
□ Setiap issue punya severity label?
□ Setiap issue punya concrete fix (bukan cuma "fix this")?
□ Gw gak ngarang issue yang bukan issue?
□ Gw baca SEMUA file sebelum mulai comment?
□ Gw acknowledge yang bagus?

IF ada □ TIDAK → fix sebelum kirim
```
