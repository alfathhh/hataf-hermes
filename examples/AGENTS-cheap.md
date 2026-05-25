# AGENTS.md — Project Conventions (Model Murah / CHEAP)

Letakkan file ini di **root repo project lo**. Hermes auto-load setiap kali
working directory ada di project tersebut atau subfoldernya.

> File ini untuk model MURAH (DeepSeek V4 Flash, Qwen3.5 Plus).
> Format: 100% decision tree + explicit DO/DO NOT. Model murah BUTUH ini.

---

## SKILL ROUTING — Decision Tree

```
INPUT: [pesan user]

IF pesan tentang coding/programming:
  IF "review" OR "audit" OR "cek kode" → skill: code-review
  IF "bikin backend" OR "API" OR "endpoint" OR "database" → skill: backend-proper
  IF "bikin web" OR "frontend" OR "React" OR "Next.js" OR "Vue" → skill: web-development
  IF "design" OR "UI cantik" OR "landing page" → skill: frontend-design
  IF "belajar coding" OR "pemula" OR "dari nol" → skill: coding-mentor
  IF "deploy" OR "Docker" OR "server" OR "CI/CD" → skill: devops-networking

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

- **Language**: Python 3.11+, Node.js 20+, TypeScript 5+
- **Backend**: FastAPI (Python), Express/Fastify (Node.js), Next.js API Routes
- **Frontend**: Next.js 14+, React 18+, Vue 3, Tailwind CSS
- **Database**: PostgreSQL, SQLite, Redis
- **ORM**: Prisma/Drizzle (Node), SQLAlchemy (Python)
- **Package Manager**: pnpm (Node), pip/uv (Python)
- **Test**: Vitest (Node), pytest (Python)
- **Deploy**: Docker, Vercel, Railway, VPS

---

## Code Style (IKUTI EXACT)

```
Python:
  Indentasi: 4 spasi
  Naming: snake_case (variable/function), PascalCase (class)
  Line max: 100 karakter
  Formatter: ruff format
  Linter: ruff check

TypeScript/JavaScript:
  Indentasi: 2 spasi
  Naming: camelCase (variable/function), PascalCase (component/class/type)
  Line max: 100 karakter
  Formatter: Biome
  Linter: Biome
  DILARANG: any (kecuali ada komentar alasan)
```

---

## ATURAN UNTUK MODEL MURAH (WAJIB IKUTI)

```
DO: baca file existing SEBELUM nulis code baru
DO: match pattern yang udah ada di repo
DO: run lint + test sebelum present hasil
DO: web_search untuk fakta current (JANGAN jawab dari memori)
DO: kasih output dengan format yang diminta skill

DO NOT: nulis code >50 baris tanpa ada pattern existing
DO NOT: jawab "menurut saya" untuk fakta yang bisa dicek
DO NOT: skip verification step
DO NOT: install dependency tanpa tanya user
DO NOT: modify file yang user gak minta
DO NOT: ngarang path, API, package name
DO NOT: jawab pertanyaan complex tanpa decompose dulu

IF ragu → TANYA user, jangan asumsi
IF gagal verify → BILANG "gak bisa verify", jangan ngarang
```

---

## Build & Run

```bash
# Python
pip install -e ".[dev]"
pytest
ruff check . && ruff format .

# Node.js / TypeScript
pnpm install
pnpm dev
pnpm build
pnpm lint
pnpm test
```

---

## Git

```
Branch: feat/..., fix/..., chore/...
Commit: imperatif, max 72 char, English
Squash sebelum merge ke main
```

---

## DILARANG (hard rules)

```
DO NOT: hapus file di luar working tree
DO NOT: git push --force ke main
DO NOT: modify lockfile tanpa approval
DO NOT: run network-heavy command tanpa alasan
DO NOT: pake any di TypeScript tanpa komentar
DO NOT: bare except: di Python
```

---

## Contoh Output yang BENAR

### User tanya fakta:
```
User: "Berapa harga emas hari ini?"
Agent: [LANGSUNG web_extract ke logammulia.com, BUKAN jawab dari memori]
```

### User minta coding:
```
User: "Tambahin endpoint GET /users"
Agent:
1. read_file() existing handler + router
2. Match pattern yang udah ada
3. Tulis code dengan pattern sama
4. Run lint + test
5. Present hasil
```

### User tanya opini:
```
User: "Mendingan Next.js atau Nuxt?"
Agent: Kasih perbandingan tabel, BUKAN pilihkan satu jawaban
```
