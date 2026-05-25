# AGENTS.md — Project Conventions (Model Kuat / STRONG)

Letakkan file ini di **root repo project lo**. Hermes auto-load setiap kali
working directory ada di project tersebut atau subfoldernya.

> File ini untuk model KUAT (Kimi K2.6 high reasoning, DeepSeek V4 Pro).
> Model ini bisa deep infer, multi-file refactor, architecture decision.
> Format: concise, trust model judgment, focus on constraints bukan hand-holding.

---

## Skill Routing

Route berdasarkan intent. Model kuat bisa infer tanpa decision tree rigid:

| Domain | Skills |
|--------|--------|
| Code | code-review, backend-proper, web-development, frontend-design, devops-networking |
| Research | deep-analysis, research-citation, data-analysis, study-buddy |
| Finance | saham-syariah, harga-emas, financial-literacy |
| Islam | islamic-study |
| Scraping | web-scrape, marketplace-scrape, social-media-scrape, music-download |
| Content | pdf-summarize, video-summary, copywriting |
| Meta | coding-mentor, weekly-review, claude-superpowers |

---

## Project Stack

- **Language**: Python 3.11+, Node.js 20+, TypeScript 5+
- **Backend**: FastAPI, Express/Fastify, Next.js API Routes
- **Frontend**: Next.js 14+ (App Router), React 18+, Vue 3 / Nuxt 3, Tailwind CSS
- **Database**: PostgreSQL, SQLite, Redis
- **ORM**: Prisma / Drizzle (Node), SQLAlchemy / SQLModel (Python)
- **Package Manager**: pnpm (Node), pip / uv (Python)
- **Test**: Vitest (Node), pytest (Python), Playwright (E2E)
- **Deploy**: Docker, Vercel, Railway, VPS

---

## Code Style

**Python**: 4 spasi, snake_case, ruff, type hints wajib.
**TypeScript/JS**: 2 spasi, camelCase, Biome strict, no `any`.
**Max line**: 100 char. Comments: English.

---

## Model Kuat — What You Can Do

Lo dipercaya untuk:
- Multi-file refactor tanpa step-by-step approval per file
- Architecture decisions (propose, explain trade-offs)
- Infer context dari codebase tanpa di-explain setiap detail
- Deep reasoning: root cause analysis, system design
- Long code output kalau justified

---

## Constraints (tetap berlaku walaupun model kuat)

1. **Verify facts** — jangan jawab fakta current dari memori. Tool-first.
2. **Dependency baru** → tanya user dulu sebelum install
3. **Refactor >10 file** → present plan dulu, tunggu approve
4. **Secrets** → jangan pernah commit, selalu dari .env
5. **Testing** → run sebelum present. Code tanpa test = incomplete.
6. **Lockfile** → jangan modify tanpa approval
7. **No force push** ke main. Ever.
8. **No `any`** di TypeScript tanpa komentar alasan.
9. **No bare `except:`** di Python.

---

## When to Use lean-ctx

Config STRONG include lean-ctx MCP. Gunakan `ctx_read_file` untuk:
- File besar (>200 baris) → AST extract relevant sections
- Re-read file yang sudah pernah dibaca → cache hit (~13 token)
- Multi-file understanding → relevance scoring per task

Jangan override lean-ctx ke `read_file` biasa kecuali butuh full file verbatim.

---

## Git

- Branch: `feat/...`, `fix/...`, `chore/...`, `refactor/...`
- Commit: imperatif, concise, English
- PR description: what + why + testing done

---

## Build & Run

```bash
# Python
pytest && ruff check . && ruff format --check .

# Node.js
pnpm lint && pnpm test && pnpm build

# Full check
pnpm lint && pnpm test && pytest && ruff check .
```

---

## Catatan

(Tambah catatan project-specific di sini.)
