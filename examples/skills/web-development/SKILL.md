---
name: web-development
description: Bikin / refactor / debug web app full-stack (frontend + minor backend glue). Stack-aware, framework-agnostic. Output kode siap-jalan, gak halu API yang gak ada, gak ngarang library version.
version: 1.0.0
metadata:
  hermes:
    tags: [web, frontend, fullstack, react, nextjs, vue, svelte, html, css]
    category: development
---

# Web Development

Skill untuk task web development end-to-end: feature implementation, refactor, debug, build/deploy. Skill ini **menolak ngarang** — kalau library version, API endpoint, atau syntax framework tidak yakin, harus verify (read source, web_search docs resmi) sebelum write code.

## When to Use

User minta:
- "Bikin halaman X dengan Next.js / Vue / Svelte / dst"
- "Tambahin feature Y di app yang ada"
- "Refactor component Z"
- "Debug kenapa tampilan rusak / data gak load"
- "Setup proyek dari scratch"

JANGAN pakai untuk:
- Backend API design murni (pake `backend-proper` skill)
- UX / desain visual yang butuh judgment estetis (pake `ui-ux` skill)
- Mobile native (iOS/Android — beda alur)

## Procedure

### 1. Stack discovery (MANDATORY sebelum nulis kode)

Sebelum buat satu baris code:

```bash
# Cek stack
ls package.json next.config.* astro.config.* svelte.config.* nuxt.config.* vite.config.*
cat package.json | head -30

# Atau untuk Python web (Django/FastAPI):
ls pyproject.toml requirements.txt manage.py
```

Catat:
- Framework + version exact (Next.js 14.2.5? 15.0?)
- Package manager (npm / yarn / pnpm / bun) — cek lockfile
- TypeScript atau JavaScript murni
- CSS approach (Tailwind / CSS modules / styled-components / vanilla)
- State management (Zustand / Redux / TanStack Query / Pinia / context API)
- Routing (App Router Next 14? Pages? React Router? File-based?)
- API layer (REST / GraphQL / tRPC / Server Actions)
- Database access (Prisma / Drizzle / TypeORM / raw SQL)
- Auth (NextAuth / Clerk / Supabase / custom JWT)
- Deployment target (Vercel / Cloudflare / VPS / Docker)

> **Cek baca file existing**: `read_file` minimum 2-3 component existing untuk match style. Indentasi, naming convention, import order — ikutin yang udah ada.

### 2. Verify framework version (anti-halu syntax)

LLM sering halu syntax yang valid di major version berbeda. Mitigasi:

| Risk | Cara cek |
|---|---|
| Next.js App Router vs Pages Router | Cek folder `app/` vs `pages/`. Sintaks beda jauh. |
| React 18 vs 19 | `cat package.json \| grep '"react"'` — Suspense, use(), action API beda. |
| Tailwind v3 vs v4 | v4 punya `@theme`, native CSS. Class palette beda. |
| TypeScript strict mode | Cek `tsconfig.json` — kalau strict, code generated harus bener-bener typed. |

Kalau ragu syntax versi tertentu — `web_search "next.js 14 app router server actions"` ke docs resmi (next.js.org/docs, react.dev). JANGAN tulis dari memori kalau ragu.

### 3. Plan sebelum kode

Untuk task non-trivial (>3 file affected), tulis plan singkat dulu:

```markdown
## Plan
- File baru: `app/dashboard/layout.tsx`, `app/dashboard/page.tsx`
- File modify: `lib/auth.ts` (add session helper), `app/layout.tsx` (add provider)
- Test plan: render integration test, manual smoke test di browser
- Rollback plan: revert file list

## Risks
- Auth helper migration bisa break existing /login page
- Tailwind class baru perlu rebuild
```

Kasih ke user, tunggu approve. Jangan langsung edit.

### 4. Implementation

- **Pake existing pattern**: kalau project udah ada `Button.tsx`, jangan bikin custom button baru — extend yang ada.
- **Hindari over-engineering**: kalau task butuh 10 baris, jangan bikin abstraction class 50 baris.
- **Type-safety first** (kalau TypeScript): no `any`, prefer narrow types. Kalau benar-benar gak bisa, kasih `// eslint-disable` dengan komentar alasan.
- **Accessibility minimum**: setiap interactive element punya label / aria. Image punya alt. Form inputs labeled.
- **Loading & error states**: data fetching jangan asumsi happy path. Kasih skeleton / spinner + error message + retry.

### 5. Test sebelum claim "selesai"

Test order (dari murah ke mahal):

1. **Type check** — `npm run typecheck` atau `tsc --noEmit`
2. **Lint** — `npm run lint`
3. **Unit / integration test** — `npm test`
4. **Build** — `npm run build` (catch SSR/build-time bug)
5. **Run manual** — minta user test di browser kalau visual

Kalau ada step yang gagal, **fix dulu** sebelum present hasil.

### 6. Output format

```markdown
## Implementasi: [Feature]

**Stack terdeteksi**: [Next.js 14.2 / TS / Tailwind v3 / Drizzle / Vercel]

**File baru**: 
- `app/dashboard/page.tsx` (server component, list users dengan suspense)

**File modified**: 
- `lib/db.ts` (add `getUsers` query)

**Test**: 
- ✅ tsc passes
- ✅ lint passes  
- ✅ build passes
- ⏳ Manual: silakan cek `/dashboard` di dev server

**Notes**:
- [Caveat / asumsi yang gw bikin]
- [Yang sebaiknya user verify lagi]
```

## Pitfalls

### Pitfall 1: Halu version-specific API

❌ "Pakai `useFormState` dari React" (itu ada di React 19, di React 18 namanya beda)

Fix: selalu cek package.json. Kalau ragu, tanya user atau search docs.

### Pitfall 2: Server vs Client component salah

Next.js App Router: default Server Component. Hooks (`useState`, `useEffect`) butuh `"use client"`. Kalau salah taro:
- Server component pake hook → build error
- Client component pake `await db.query()` → akses DB di browser, security disaster

Cek tiap component: butuh interactivity? client. Cuma data fetch? server.

### Pitfall 3: Hydration mismatch

Render server beda dari client (timestamp, random, locale). Hindari:
- `Date.now()` / `Math.random()` di render path
- Browser-only API (`window`, `localStorage`) tanpa guard
- Conditional render berdasarkan `typeof window`

Pake `useEffect` atau `'use client'` untuk dynamic content.

### Pitfall 4: Salah CSS spec

Tailwind v3 dan v4 beda fundamental (config approach, color palette). CSS modules vs styled-components vs vanilla CSS punya quirks beda.

Cek file CSS / Tailwind config dulu sebelum nulis class.

### Pitfall 5: Asumsi env tersedia

`process.env.NEXT_PUBLIC_API_URL` di client = harus prefix `NEXT_PUBLIC_`. `process.env.DATABASE_URL` di client = leak / undefined.

Verify env strategy framework dulu.

### Pitfall 6: Library yang gak ke-install

Jangan import library random tanpa cek package.json. Kalau perlu library baru:
- Tanya user dulu
- Justify kenapa perlu (apakah bisa solve native?)
- Cek bundle size (bundlephobia)

## Verification

Sebelum kasih hasil ke user:

1. Apakah semua import valid (cek package.json)?
2. Apakah type check passing?
3. Apakah ada `console.log` / `debugger` ketinggalan?
4. Apakah ada hardcoded secret / URL?
5. Apakah handling loading / error state sudah ada?
6. Apakah accessibility minimum terpenuhi?
7. Apakah lo modify file yang user gak minta diubah?

Kalau ada "ya" di #4, #7, atau "tidak" di lainnya — fix dulu.

## Untuk Setup Project Baru

Saat user minta "bikin project Next.js / Vite / Astro baru":

```bash
# JANGAN langsung scaffold — tanya dulu:
# 1. Package manager preference (npm/yarn/pnpm/bun)?
# 2. TypeScript yes/no?
# 3. Tailwind / styling preference?
# 4. Testing framework (Vitest / Jest / Playwright)?
# 5. Linter / formatter (ESLint+Prettier / Biome)?
```

Default rekomendasi gw (kalau user no opinion):

| Pilihan | Alasan |
|---|---|
| pnpm | Faster, disk-efficient |
| TypeScript | Catch bug compile-time |
| Tailwind | Productivity, ekosistem besar |
| Vitest | Fast, ESM-native |
| Biome | All-in-one (lint+format), 10x lebih cepat dari ESLint+Prettier |

> Caveat: ekosistem JS bergerak cepat. Per [biomejs.dev](https://biomejs.dev) Biome stable, tapi user yang udah biasa ESLint mungkin prefer stick. Tanya dulu.

## Untuk Debug Visual / Layout

```
1. Buka di browser → buka DevTools
2. Inspect element yang masalah
3. Cek computed styles vs expected
4. Cek box model (margin, padding, border)
5. Cek media query active
6. Cek z-index / stacking context
```

Kalau butuh otomatis:

```
> screenshot halaman /dashboard, area sidebar
```

Hermes browser tool + vision_analyze bisa visual diff.
