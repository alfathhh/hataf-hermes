# AGENTS.md — Project Conventions (Optimized for Weak Models)

Letakkan file ini di **root repo project lo**. Hermes auto-load setiap kali
working directory ada di project tersebut atau subfoldernya.

> File ini di-optimize agar model murah (DeepSeek V4 Flash, Qwen3.5) bisa
> follow instruksi dengan benar. Format: decision tree, bukan prose.

---

## SKILL ROUTING — Decision Tree

Saat user kirim pesan, ikuti tree ini untuk pilih skill:

```
INPUT: [pesan user]

IF pesan tentang coding/programming:
  IF "review" OR "audit" OR "cek kode" → skill: code-review
  IF "bikin backend" OR "API" OR "endpoint" OR "database" → skill: backend-proper
  IF "bikin web" OR "frontend" OR "React" OR "Next.js" OR "Vue" → skill: web-development
  IF "design" OR "UI cantik" OR "landing page" OR "beautify" → skill: frontend-design
  IF "belajar coding" OR "pemula" OR "dari nol" → skill: coding-mentor
  IF "deploy" OR "Docker" OR "server" OR "CI/CD" OR "nginx" → skill: devops-networking

IF pesan tentang research/analisis:
  IF "analisis mendalam" OR "trade-off" OR "arsitektur" OR "keputusan besar" → skill: deep-analysis
  IF "fakta" OR "data" OR "sumber" OR "citation" OR "berapa" OR "kapan" → skill: research-citation
  IF "data" AND ("CSV" OR "pandas" OR "chart" OR "visualisasi") → skill: data-analysis

IF pesan tentang keuangan:
  IF "saham" AND ("syariah" OR "DES" OR "JII") → skill: saham-syariah
  IF "emas" OR "antam" OR "logammulia" → skill: harga-emas
  IF "budget" OR "investasi" OR "compound" OR "nabung" OR "pensiun" → skill: financial-literacy

IF pesan tentang Islam:
  IF "ayat" OR "hadits" OR "fiqh" OR "doa" OR "hukum islam" → skill: islamic-study

IF pesan tentang scraping/download:
  IF "Instagram" OR "Twitter" OR "X" OR "TikTok" OR "social media" → skill: social-media-scrape
  IF "Tokopedia" OR "Shopee" OR "marketplace" OR "ecommerce" → skill: marketplace-scrape
  IF "scrape" OR "extract" OR "monitor web" → skill: web-scrape
  IF "download lagu" OR "YouTube" OR "Tidal" OR "Spotify" → skill: music-download

IF pesan tentang konten/dokumen:
  IF "PDF" OR "summarize dokumen" OR "ringkas paper" → skill: pdf-summarize
  IF "video" OR "transkrip" OR "ringkas video" → skill: video-summary
  IF "nulis" OR "caption" OR "artikel" OR "copywriting" OR "email" → skill: copywriting

IF pesan tentang belajar (bukan coding):
  IF user kasih bahan/file AND minta diskusi/quiz → skill: study-buddy

IF pesan tentang UI/UX review (bukan implementasi):
  IF "review UI" OR "UX" OR "accessibility" OR "design system" → skill: ui-ux

IF pesan tentang meta/agent:
  IF "weekly review" OR "self-improvement" → skill: weekly-review
  IF "full power" OR "deep work" OR "semua tools" → skill: claude-superpowers

IF tidak match satupun → jawab langsung tanpa skill khusus
```

---

## Project: <NAMA_PROJECT>

**Stack** (ISI SESUAI PROJECT LO):
- Language: <Go 1.22 / Python 3.11 / Node.js 20 / dst>
- Framework: <chi / FastAPI / Next.js 14 / dst>
- Database: <PostgreSQL 16 / SQLite / dst>
- Deploy: <Vercel / Railway / VPS sendiri / dst>

**Direktori penting**:
- `cmd/` — entrypoint
- `internal/` — non-exported code
- `pkg/` — shared library
- (sesuaikan)

---

## Convention

### Code style
- Indentasi: **tab** untuk Go, **4 spasi** untuk Python, **2 spasi** untuk JS/TS
- Line length: **120 karakter** max
- Naming: `camelCase` untuk JS, `snake_case` untuk Python, `PascalCase` untuk Go exports
- Komentar: jelas, dalam Bahasa Inggris

### Git
- Branch: `feat/...`, `fix/...`, `chore/...`
- Commit message: imperatif, max 72 karakter subject
- Squash sebelum merge ke `main`

### Test
- Run all tests: `make test` (atau `go test ./...`, `pytest`, `npm test`)
- Coverage minimum: 70% untuk paket baru
- Test naming: `Test<Function>_<Case>`

---

## Build & Run

```bash
# Install deps
make install              # atau: go mod download / pip install -r requirements.txt / npm i

# Run dev server
make dev                  # atau: go run ./cmd/api / uvicorn main:app --reload

# Build production
make build

# Lint
make lint                 # atau: golangci-lint run / ruff check . / npm run lint

# Tipe check (kalau ada)
make typecheck            # atau: mypy . / tsc --noEmit
```

---

## Model-Aware Rules

### IF model = cheap (DeepSeek V4 Flash / Qwen3.5):
- JANGAN attempt deep analysis — delegasikan ke primary model
- JANGAN jawab pertanyaan fakta current dari memori — HARUS web_search
- JANGAN nulis code >50 baris tanpa existing pattern di repo
- BOLEH: translate, format, simple Q&A, summarize, classify

### IF model = primary (Kimi K2.6 / DeepSeek V4 Pro):
- BOLEH: complex coding, architecture decision, multi-file refactor
- TETAP: verify sebelum claim, tool-first

---

## Kebiasaan project

- **Migration database**: jangan ALTER TABLE manual — selalu via tools migration
- **Secrets**: `.env` di-gitignore. Jangan pernah commit secrets.
- **Dependency baru**: kalau mau install dependency baru, **tanya dulu** ke user
- **Refactor besar**: kalau perubahan affect >10 file, bikin PR plan dulu

---

## DILARANG (hard rules)

```
DO NOT: hapus file di luar working tree project ini
DO NOT: git push --force ke main
DO NOT: modify lockfile tanpa user approval
DO NOT: run command network-heavy tanpa alasan jelas
DO NOT: jawab "menurut saya" untuk fakta yang bisa dicek
DO NOT: skip verification step
```

---

## Quick Commands (zero token, langsung execute)

```bash
# Lihat status
git status && git log -5 --oneline

# Diff sebelum commit
git diff --staged

# Cek lint sebelum commit
make lint && make test
```

---

## Contoh Output yang BENAR (untuk model murah)

### Kalau user tanya fakta:
```
User: "Berapa harga emas hari ini?"
Agent: [LANGSUNG web_extract ke logammulia.com, BUKAN jawab dari memori]
```

### Kalau user minta coding:
```
User: "Tambahin endpoint GET /users"
Agent:
1. Baca file existing (handler, router, model)
2. Match pattern yang udah ada
3. Tulis code dengan pattern sama
4. Run lint + test
5. Present hasil
```

### Kalau user tanya opini:
```
User: "Mendingan Next.js atau Nuxt?"
Agent: Kasih perbandingan tabel, BUKAN pilihkan satu jawaban
```

---

## Catatan personal

(Tambah catatan-catatan kecil di sini yang bantu agent inget context.)
