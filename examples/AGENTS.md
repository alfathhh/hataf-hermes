# AGENTS.md — Project Conventions

Letakkan file ini di **root repo project lo**. Hermes auto-load setiap kali
working directory ada di project tersebut atau subfoldernya.

> Bedakan dari `~/.hermes/SOUL.md` (global identity, anti-halu rules). AGENTS.md
> ini cuma project-specific aturan teknis.

---

## Project: <NAMA_PROJECT>

**Stack**:
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

## Kebiasaan project

- **Migration database**: jangan ALTER TABLE manual — selalu via tools migration (sqlc, alembic, dst)
- **Secrets**: `.env` di-gitignore. Jangan pernah commit secrets.
- **Dependency baru**: kalau lo (Hermes) mau install dependency baru, **tanya dulu** ke user — jangan auto-install yang heavy (>10MB) tanpa konfirmasi.
- **Refactor besar**: kalau perubahan affect >10 file, bikin PR plan dulu di markdown sebelum mulai ngedit.

---

## Yang harus Hermes hindari

- Jangan hapus file di luar working tree project ini.
- Jangan jalankan `git push --force` ke `main`.
- Jangan modify `package-lock.json` / `go.sum` / `poetry.lock` tanpa user approval (itu menyiratkan dependency tree berubah).
- Jangan run command yang network-heavy tanpa alasan jelas (`curl` ke API random, dst).

---

## Tools yang sering dipake

```bash
# Lihat status
git status && git log -5 --oneline

# Diff sebelum commit
git diff --staged

# Cek lint sebelum commit
make lint && make test
```

---

## Catatan personal

(Lo bisa nambah catatan-catatan kecil di sini, misalnya "dengar bahwa endpoint
production lambat hari Jumat", "deployment lewat GitHub Actions, butuh approval
dari @nama" — apa pun yang membantu Hermes inget context.)
