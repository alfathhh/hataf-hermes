# AGENTS.md — Project Conventions

Letakkan file ini di **root repo project lo**. Hermes auto-load setiap kali
working directory ada di project tersebut atau subfoldernya.

> Versi ini untuk model yang cukup pintar (Kimi K2.6, DeepSeek V4 Flash/Pro).
> Model ini bisa follow prose + decision tree. Format: campuran pragmatis.

---

## SKILL ROUTING

Saat user kirim pesan, pilih skill yang paling cocok:

```
IF pesan tentang coding/programming:
  IF "review" OR "audit" OR "cek kode" → skill: code-review
  IF "bikin backend" OR "API" OR "endpoint" OR "database" → skill: backend-proper
  IF "bikin web" OR "frontend" OR "React" OR "Next.js" OR "Vue" → skill: web-development
  IF "design" OR "UI cantik" OR "landing page" OR "beautify" → skill: frontend-design
  IF "belajar coding" OR "pemula" OR "dari nol" → skill: coding-mentor
  IF "deploy" OR "Docker" OR "server" OR "CI/CD" OR "nginx" → skill: devops-networking

IF pesan tentang research/analisis:
  IF "analisis mendalam" OR "trade-off" OR "arsitektur" → skill: deep-analysis
  IF "fakta" OR "data" OR "sumber" OR "citation" → skill: research-citation
  IF "data" AND ("CSV" OR "pandas" OR "chart") → skill: data-analysis

IF pesan tentang keuangan:
  IF "saham" AND ("syariah" OR "DES" OR "JII") → skill: saham-syariah
  IF "emas" OR "antam" OR "logammulia" → skill: harga-emas
  IF "budget" OR "investasi" OR "compound" → skill: financial-literacy

IF pesan tentang Islam:
  IF "ayat" OR "hadits" OR "fiqh" OR "doa" → skill: islamic-study

IF pesan tentang scraping/download:
  IF "Instagram" OR "Twitter" OR "TikTok" → skill: social-media-scrape
  IF "Tokopedia" OR "Shopee" OR "marketplace" → skill: marketplace-scrape
  IF "scrape" OR "extract" OR "monitor web" → skill: web-scrape
  IF "download lagu" OR "YouTube" OR "Tidal" → skill: music-download

IF pesan tentang konten/dokumen:
  IF "PDF" OR "summarize dokumen" → skill: pdf-summarize
  IF "video" OR "transkrip" → skill: video-summary
  IF "nulis" OR "caption" OR "artikel" → skill: copywriting

IF pesan tentang belajar (bukan coding):
  IF user kasih bahan/file + minta diskusi/quiz → skill: study-buddy

IF tidak match → jawab langsung tanpa skill khusus
```

---

## Project Stack

**Languages**:
- Python 3.11+
- Node.js 20+ / TypeScript 5+
- JavaScript (browser + server)

**Backend Frameworks** (pilih sesuai project):
- FastAPI (Python — API utama)
- Express / Fastify (Node.js)
- Next.js API Routes (kalau fullstack)

**Frontend Frameworks**:
- Next.js 14+ (App Router, React Server Components)
- React 18+
- Vue 3 / Nuxt 3 (kalau project Vue-based)
- Tailwind CSS (styling utama)

**Database**:
- PostgreSQL (primary relational)
- SQLite (untuk project kecil / prototyping)
- Redis (caching, queue)
- Prisma / Drizzle ORM (Node.js)
- SQLAlchemy / SQLModel (Python)

**Package Manager**:
- pnpm (Node.js — default)
- pip / uv (Python)

**Testing**:
- Vitest (Node.js/TS — default)
- pytest (Python)
- Playwright (E2E kalau perlu)

**Deploy**:
- Docker + docker-compose
- Vercel (frontend/fullstack)
- Railway / Render (backend)
- VPS sendiri (kalau butuh kontrol penuh)

---

## Code Style

### Python
- Indentasi: **4 spasi**
- Formatter: `ruff format`
- Linter: `ruff check`
- Type hints: wajib untuk public functions
- Naming: `snake_case` (variable, function), `PascalCase` (class)
- Docstring: Google style

### TypeScript / JavaScript
- Indentasi: **2 spasi**
- Formatter: Biome atau Prettier
- Linter: Biome atau ESLint
- Naming: `camelCase` (variable, function), `PascalCase` (component, class, type)
- Prefer `const` over `let`, never `var`
- Strict mode TypeScript (no `any` kecuali justified)

### Umum
- Line length: **100 karakter** max (Python), **100** (TS/JS)
- Komentar: Bahasa Inggris
- Import: absolute > relative (kecuali co-located modules)

---

## Git Convention

- Branch: `feat/...`, `fix/...`, `chore/...`, `refactor/...`
- Commit message: imperatif, max 72 char subject, Bahasa Inggris
- Squash sebelum merge ke `main`
- PR review wajib sebelum merge (kalau ada reviewer)

---

## Build & Run

```bash
# Python project
pip install -e ".[dev]"           # atau: uv pip install -e ".[dev]"
pytest                            # run tests
ruff check . && ruff format .     # lint + format

# Node.js / TypeScript project
pnpm install
pnpm dev                          # dev server
pnpm build                        # production build
pnpm lint                         # lint (biome atau eslint)
pnpm test                         # vitest

# Docker
docker compose up -d              # start semua service
docker compose logs -f            # lihat logs
```

---

## Kebiasaan Project

- **Migration database**: selalu via migration tool (Prisma Migrate, Alembic) — jangan ALTER TABLE manual
- **Secrets**: `.env` di-gitignore. Jangan pernah commit secrets.
- **Dependency baru**: tanya user dulu sebelum install yang berat / unfamiliar
- **Refactor besar**: kalau affect >10 file, bikin plan dulu sebelum mulai
- **Error handling**: explicit — no bare `except:` (Python), no swallowed `.catch()` (JS)

---

## DILARANG (hard rules)

```
DO NOT: hapus file di luar working tree project ini
DO NOT: git push --force ke main
DO NOT: modify lockfile tanpa user approval
DO NOT: run command network-heavy tanpa alasan jelas
DO NOT: jawab "menurut saya" untuk fakta yang bisa dicek
DO NOT: skip verification step
DO NOT: install dependency tanpa tanya user
DO NOT: pake `any` di TypeScript tanpa komentar alasan
```

---

## Model-Aware Rules

### IF model = strong (Kimi K2.6, DeepSeek V4 Pro):
- BOLEH: complex coding, multi-file refactor, architecture decision
- BOLEH: nulis code >100 baris kalau pattern jelas
- TETAP: verify sebelum claim, tool-first untuk fakta current
- BOLEH: infer context dari code tanpa di-spell-out setiap detail

### IF model = medium (DeepSeek V4 Flash):
- BOLEH: coding standard, single-file changes, follow existing pattern
- HATI-HATI: multi-file refactor (cek impact dulu)
- TETAP: verify facts, tool-first
- PREFER: match existing patterns, jangan over-engineer

### IF model = cheap (Qwen3.5 Plus):
- JANGAN: attempt complex coding tanpa existing pattern
- JANGAN: jawab fakta current dari memori — HARUS web_search
- BOLEH: translate, format, simple Q&A, summarize, classify

---

## Quick Commands (zero token)

```bash
# Status
git status && git log -5 --oneline

# Diff sebelum commit
git diff --staged

# Lint sebelum commit
pnpm lint && pnpm test        # Node.js
ruff check . && pytest        # Python
```

---

## Catatan Personal

(Tambah catatan-catatan kecil di sini yang bantu agent inget context project.)
