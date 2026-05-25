# AGENTS.md — Project Conventions (Model Balanced / K2.6 Medium)

Letakkan file ini di **root repo project lo**. Hermes auto-load setiap kali
working directory ada di project tersebut atau subfoldernya.

> File ini untuk model BALANCED (Kimi K2.6 medium reasoning).
> Model ini bisa follow prose + infer context. Format: pragmatis, gak over-explicit.

---

## Skill Routing

Pilih skill paling cocok berdasarkan intent user. Kalau gak yakin, jawab langsung.

| Intent | Skill |
|--------|-------|
| Code review / audit | code-review |
| Backend / API / DB | backend-proper |
| Frontend / web app | web-development |
| UI design / beautify | frontend-design |
| Belajar coding | coding-mentor |
| Deploy / Docker / server | devops-networking |
| Deep analysis / trade-off | deep-analysis |
| Fakta + citation | research-citation |
| Data + chart | data-analysis |
| Saham syariah | saham-syariah |
| Harga emas | harga-emas |
| Budget / investasi | financial-literacy |
| Islam / ayat / hadits | islamic-study |
| Social media scrape | social-media-scrape |
| Marketplace scrape | marketplace-scrape |
| Web scrape | web-scrape |
| Download musik | music-download |
| PDF summary | pdf-summarize |
| Video summary | video-summary |
| Copywriting | copywriting |
| Belajar dari bahan | study-buddy |

---

## Project Stack

- **Language**: Python 3.11+, Node.js 20+, TypeScript 5+
- **Backend**: FastAPI (Python), Express/Fastify (Node.js), Next.js API Routes
- **Frontend**: Next.js 14+ (App Router), React 18+, Vue 3 / Nuxt 3, Tailwind CSS
- **Database**: PostgreSQL, SQLite, Redis
- **ORM**: Prisma / Drizzle (Node), SQLAlchemy / SQLModel (Python)
- **Package Manager**: pnpm (Node), pip / uv (Python)
- **Test**: Vitest (Node), pytest (Python), Playwright (E2E)
- **Deploy**: Docker, Vercel, Railway, VPS

---

## Code Style

**Python**: 4 spasi, snake_case, ruff format + check, type hints wajib untuk public functions.

**TypeScript/JS**: 2 spasi, camelCase, Biome lint+format, strict mode, no `any`.

**Umum**: max 100 char/line, komentar English, absolute imports preferred.

---

## Git

- Branch: `feat/...`, `fix/...`, `chore/...`
- Commit: imperatif, max 72 char, English
- Squash sebelum merge ke `main`

---

## Approach untuk model balanced

Model ini (K2.6 medium reasoning) cukup kuat untuk:
- Infer context dari code tanpa di-spell-out setiap detail
- Multi-file changes kalau pattern jelas
- Architecture suggestions
- Code review tanpa step-by-step checklist

Tetap perlu:
- Verify fakta via tools (web_search, read_file) — jangan dari memori
- Tanya user sebelum install dependency baru
- Plan dulu kalau refactor >10 file
- Run lint + test sebelum present

Jangan:
- Over-engineer (kalau task butuh 10 baris, jangan bikin 50 baris abstraction)
- Ngarang package name / API behavior tanpa verify
- Push --force ke main
- Modify lockfile tanpa approval

---

## Build & Run

```bash
# Python
pip install -e ".[dev]"
pytest
ruff check . && ruff format .

# Node.js
pnpm install
pnpm dev
pnpm build
pnpm lint && pnpm test

# Docker
docker compose up -d
docker compose logs -f
```

---

## Catatan

(Tambah catatan project-specific di sini.)
