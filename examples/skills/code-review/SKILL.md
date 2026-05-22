---
name: code-review
description: Review kode seperti senior engineer yang peduli quality. Cari bug, security issue, performance problem, maintainability issue. Prioritized output, bukan nitpick random.
version: 1.0.0
metadata:
  hermes:
    tags: [review, code-quality, security, performance, refactor]
    category: development
    requires_toolsets: [core]
---

# Code Review

Skill untuk review kode serius — bukan nitpick style, tapi cari masalah yang **berdampak nyata** ke production.

## When to Use

- User minta review file / PR / diff
- Sebelum deploy kode penting
- Audit keamanan
- Refactor assessment (mana yang perlu, mana yang bisa ditunda)

## Procedure

### 1. Baca keseluruhan dulu

JANGAN komentar per baris sequential. Baca SEMUA file yang di-review dulu. Pahami:
- Apa tujuan kode ini?
- Gimana flow datanya?
- Apa yang berubah dari sebelumnya (kalau diff)?

### 2. Cek berdasarkan severity tier

| Tier | Label | Artinya |
|------|-------|---------|
| 🔴 **Critical** | Bug / security / data loss | HARUS fix sebelum deploy |
| 🟠 **High** | Performance cliff / race condition / error handling hilang | Sangat disarankan fix |
| 🟡 **Medium** | Maintainability issue / unclear naming / missing test | Fix kalau sempat |
| 🟢 **Low** | Style preference / minor optimization | Nice-to-have |

FOKUS ke 🔴 dan 🟠. Jangan habiskan space untuk 🟢 kalau ada 🔴.

### 3. Review checklist (scan tiap point)

#### Security
- [ ] SQL injection (string concat ke query?)
- [ ] XSS (user input ke HTML tanpa escape?)
- [ ] Auth bypass (endpoint tanpa auth check?)
- [ ] Secret hardcoded (API key, password di kode?)
- [ ] Path traversal (user input ke file path?)
- [ ] SSRF (user-controlled URL di server-side fetch?)
- [ ] Mass assignment (user bisa set field yang seharusnya gak bisa?)

#### Correctness
- [ ] Off-by-one error
- [ ] Null/undefined not handled
- [ ] Race condition (concurrent access ke shared state?)
- [ ] Exception swallowed (catch tanpa rethrow/log?)
- [ ] Type mismatch (implicit coercion yang salah?)
- [ ] Edge case: empty list, zero value, max int, unicode

#### Performance
- [ ] N+1 query (loop → DB call per iteration?)
- [ ] Missing index (WHERE/JOIN pada column tanpa index?)
- [ ] Unbounded result set (SELECT tanpa LIMIT?)
- [ ] Memory leak (event listener, interval, tanpa cleanup?)
- [ ] Blocking main thread (sync IO di async context?)
- [ ] Large payload tanpa pagination

#### Maintainability
- [ ] Function > 50 baris (terlalu panjang?)
- [ ] Deeply nested (>3 level indent?)
- [ ] Magic number/string (hardcoded tanpa named constant?)
- [ ] Unclear naming (apa itu `data`, `info`, `temp`, `x`?)
- [ ] Dead code (unreachable path, unused import?)
- [ ] Missing error handling documentation

### 4. Format output

```markdown
## Code Review: [file/PR]

### Summary
[1-2 kalimat: overall impression + highest severity issue]

### 🔴 Critical

#### [Issue title]
**File**: `path/to/file.ts:42`
**Apa**: [Deskripsi masalah]
**Impact**: [Apa yang terjadi kalau gak difix]
**Fix**:
```suggestion
// code fix
```

### 🟠 High
...

### 🟡 Medium
...

### ✅ Yang udah bagus
- [Acknowledge hal positif — jangan cuma kritik]

### Questions (bukan issue, tapi gw perlu klarifikasi)
- [Hal yang gw gak sure tanpa lebih banyak context]
```

### 5. Kalau gak ada issue serius

Bilang langsung: "Gw gak nemu critical/high issue di review ini." Jangan bikin-bikin issue biar keliatan thorough. False positive = noise.

## Pitfalls

### Pitfall 1: Review tanpa context

JANGAN review isolated function. Liat bagaimana dia dipanggil, oleh siapa, di path apa. Bug sering ada di caller, bukan callee.

### Pitfall 2: Style nitpick dominasi

Kalau ada 1 security bug dan 20 style issues, laporan HARUS lead dengan security bug. Jangan buried di antara "use const instead of let" noise.

### Pitfall 3: Saran tanpa contoh

❌ "This function is too complex"
✅ "Split jadi 2 function: `validateInput()` dan `processOrder()`. Contoh:"

### Pitfall 4: Assumsi behavior tanpa cek

Kalau gw bilang "ini bisa race condition", gw harus bisa explain scenario konkret. Bukan theoretical — concrete: "thread A read X, thread B write X, A write stale X back".

### Pitfall 5: Missing the forest for the trees

Kadang kode technically correct per function, tapi arsitektur keseluruhan salah. Step back: apakah approach ini make sense? Atau ada fundamental design issue?

## Verification

1. Apakah setiap issue punya severity label?
2. Apakah setiap issue punya concrete fix suggestion (bukan cuma "fix this")?
3. Apakah gw gak ngarang issue yang bukan issue?
4. Apakah gw baca SEMUA file sebelum mulai comment?
5. Apakah gw acknowledge yang bagus (bukan pure criticism)?
