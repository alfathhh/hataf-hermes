# AGENTS.md — Project Conventions (Model Hybrid / K2.6 + Flash)

Letakkan file ini di **root repo project lo**. Hermes auto-load setiap kali
working directory ada di project tersebut atau subfoldernya.

> File ini untuk HYBRID strategy (K2.6 untuk complex, Flash untuk simple).
> Smart routing aktif — turn simple otomatis ke Flash. Instructions harus
> cukup explicit supaya Flash juga bisa follow, tapi gak se-verbose cheap.

---

## Skill Routing

Pilih skill paling cocok berdasarkan intent user:

```
coding → code-review / backend-proper / web-development / frontend-design / devops-networking
research → deep-analysis / research-citation / data-analysis
finance → saham-syariah / harga-emas / financial-literacy
islam → islamic-study
scraping → social-media-scrape / marketplace-scrape / web-scrape / music-download
content → pdf-summarize / video-summary / copywriting
learning → coding-mentor / study-buddy
```

Kalau gak match → jawab langsung tanpa skill.

---

## Project Stack

- **Language**: Python 3.11+, Node.js 20+, TypeScript 5+
- **Backend**: FastAPI, Express/Fastify, Next.js API Routes
- **Frontend**: Next.js 14+, React 18+, Vue 3, Tailwind CSS
- **Database**: PostgreSQL, SQLite, Redis
- **ORM**: Prisma/Drizzle (Node), SQLAlchemy (Python)
- **Package Manager**: pnpm (Node), pip/uv (Python)
- **Test**: Vitest, pytest, Playwright
- **Deploy**: Docker, Vercel, Railway, VPS

---

## Code Style

**Python**: 4 spasi, snake_case, ruff, type hints.
**TypeScript/JS**: 2 spasi, camelCase, Biome, strict, no `any`.

---

## Hybrid-Specific Rules

Model ini punya 2 mode:
- **K2.6** (turn complex) → bisa infer, multi-file OK, architecture OK
- **Flash** (turn simple via routing) → butuh pattern matching, explicit better

Supaya KEDUA model bisa perform:

### Untuk semua turn:
- Verify fakta via tools — jangan dari memori
- Match existing patterns sebelum bikin yang baru
- Run lint + test sebelum present
- Tanya user sebelum install dependency baru

### Untuk coding tasks (ke K2.6):
- Multi-file refactor OK kalau impact jelas
- Architecture decision OK
- Boleh nulis code panjang kalau pattern ada

### Untuk simple tasks (ke Flash):
- Format output jelas (tabel, bullet, code block)
- Jangan over-think — jawab langsung
- Kalau butuh fakta → web_search, jangan asumsi

---

## Git

- Branch: `feat/...`, `fix/...`, `chore/...`
- Commit: imperatif, max 72 char, English
- Squash sebelum merge

---

## DILARANG

- Hapus file di luar working tree
- Push --force ke main
- Modify lockfile tanpa approval
- Pake `any` tanpa komentar (TS)
- Bare `except:` (Python)
- Install dependency tanpa tanya
- Ngarang facts tanpa verify

---

## Build & Run

```bash
# Python
pytest && ruff check .

# Node.js
pnpm lint && pnpm test && pnpm build
```

---

## Catatan

(Tambah catatan project-specific di sini.)
