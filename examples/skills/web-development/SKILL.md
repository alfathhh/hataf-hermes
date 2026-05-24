---
name: web-development
description: Bikin / refactor / debug web app full-stack. Stack-aware, framework-agnostic. Output kode siap-jalan.
version: 2.0.0
metadata:
  hermes:
    tags: [web, frontend, fullstack, react, nextjs, vue, svelte, html, css]
    category: development
---

# Web Development

## KAPAN PAKAI

```
IF user minta "bikin halaman" OR "component" OR "feature web" → PAKAI
IF user minta "refactor component" → PAKAI
IF user minta "debug tampilan" OR "data gak load" → PAKAI
IF user minta "setup project baru" → PAKAI
IF user minta backend API murni → JANGAN (pakai backend-proper)
IF user minta UI design review → JANGAN (pakai ui-ux)
```

---

## PROCEDURE (ikuti exact)

### Step 1: Stack discovery (WAJIB sebelum nulis code)

```bash
ls package.json next.config.* vite.config.* nuxt.config.* svelte.config.*
cat package.json | head -30
```

CATAT:
- Framework + version exact
- Package manager (npm/yarn/pnpm/bun) — cek lockfile
- TypeScript atau JavaScript
- CSS approach (Tailwind / CSS Modules / styled-components)
- Routing (App Router / Pages / React Router / file-based)
- State management
- Auth approach

KEMUDIAN:
```
read_file() MINIMAL 2-3 component existing → match style
```

### Step 2: Verify framework version (anti-halu syntax)

```
IF Next.js:
  CHECK: folder app/ atau pages/?
  IF app/ → App Router (server components default, "use client" for hooks)
  IF pages/ → Pages Router (different API)

IF React:
  CHECK: cat package.json | grep '"react"'
  IF react 18 → useFormStatus TIDAK ada
  IF react 19 → use(), useFormState ada

IF Tailwind:
  CHECK: version 3 atau 4?
  IF v3 → tailwind.config.js, prefix classes
  IF v4 → @theme directive, CSS-native

IF ragu syntax → web_search("[framework] [version] docs [feature]")
DO NOT: tulis syntax dari memori kalau ragu versi
```

### Step 3: Plan (untuk task >3 file)

```markdown
## Plan
- File baru: [list]
- File modify: [list]
- Test plan: [how to verify]
- Risk: [apa yang bisa break]
```

TUNJUKKAN ke user, TUNGGU approve.

### Step 4: Implementation rules

```
RULE: Pake existing pattern (kalau ada Button.tsx, extend itu — jangan bikin baru)
RULE: Hindari over-engineering (10 baris task ≠ 50 baris abstraction)
RULE: Type-safety first (no `any`, narrow types)
RULE: Accessibility minimum (label, aria, alt text)
RULE: Loading + error states (jangan asumsi happy path)
DO NOT: import library yang gak ada di package.json tanpa tanya user
DO NOT: campur server/client component salah
```

### Step 5: Test sebelum deliver

```
RUN (dalam urutan):
1. tsc --noEmit (type check)
2. npm run lint
3. npm run build (catch SSR/build-time bug)

IF ada yang gagal → FIX sebelum present hasil
IF semua pass → deliver
```

---

## OUTPUT TEMPLATE

```markdown
## Implementasi: [Feature]

**Stack**: [Next.js 14 / TS / Tailwind v3 / dll]

**File baru**:
- `path/file.tsx` — [deskripsi singkat]

**File modified**:
- `path/file.ts` — [apa yang diubah]

**Test**:
- ✅ tsc passes
- ✅ lint passes
- ✅ build passes
- ⏳ Manual: silakan cek [path] di dev server

**Notes**:
- [caveat / asumsi]
- [yang perlu user verify]
```

---

## CONTOH OUTPUT

```markdown
## Implementasi: Dashboard Page

**Stack**: Next.js 14.2 / TypeScript / Tailwind v3 / Drizzle ORM

**File baru**:
- `app/dashboard/page.tsx` — server component, list users
- `app/dashboard/loading.tsx` — skeleton loading state
- `components/UserCard.tsx` — card component reusable

**File modified**:
- `lib/db.ts` — tambah query `getActiveUsers()`
- `app/layout.tsx` — tambah nav link ke /dashboard

**Test**:
- ✅ tsc passes (no type errors)
- ✅ lint passes
- ✅ build passes (no SSR issues)
- ⏳ Manual: cek `/dashboard` di dev server

**Notes**:
- UserCard pake pattern yang sama kayak existing ProductCard
- Loading state pake Suspense boundary
- Data fetch di server component (no client-side fetching)
```

---

## DECISION TREE: Server vs Client Component (Next.js App Router)

```
IF component pake hooks (useState, useEffect, useRef) → "use client"
IF component cuma fetch data dari DB → server component (DEFAULT)
IF component handle form submission → "use client"
IF component render static content → server component
IF component pake browser API (window, localStorage) → "use client"

COMMON MISTAKE: useState di server component → BUILD ERROR
COMMON MISTAKE: await db.query() di client component → SECURITY DISASTER
```

## DECISION TREE: Setup Project Baru

```
IF user minta project baru:
  TANYA DULU:
  1. Package manager? (npm/yarn/pnpm/bun)
  2. TypeScript? (yes/no)
  3. Styling? (Tailwind / CSS Modules / etc)
  4. Testing? (Vitest / Jest / Playwright)
  5. Linter? (ESLint+Prettier / Biome)

  IF user no preference → default:
  - pnpm (faster, disk-efficient)
  - TypeScript (catch bugs)
  - Tailwind (productive)
  - Vitest (fast, ESM-native)
  - Biome (all-in-one, 10x faster)
```

## DECISION TREE: Hydration Mismatch

```
IF render server beda dari client:
  CAUSES:
  - Date.now() / Math.random() di render path
  - Browser-only API tanpa guard
  - Conditional render based on typeof window

  FIX:
  - Pake useEffect untuk dynamic content
  - Pake 'use client' directive
  - Pake suppressHydrationWarning (last resort)
```

---

## VERIFICATION

```
□ Semua import valid (ada di package.json)?
□ Type check passing?
□ Gak ada console.log / debugger ketinggalan?
□ Gak ada hardcoded secret / URL?
□ Loading + error state ada?
□ Accessibility minimum (label, alt)?
□ Gw gak modify file yang user gak minta?

IF ada □ TIDAK → fix sebelum deliver
```
