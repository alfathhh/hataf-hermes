# 13 — lean-ctx — Hemat Token 89-99%

Tujuan: install lean-ctx biar shell output + file reads Hermes di-compress sebelum masuk context LLM. Hemat quota OpenCode Go lo secara signifikan.

> Sumber: [github.com/yvgude/lean-ctx](https://github.com/yvgude/lean-ctx), [leanctx.com](https://leanctx.com), [lib.rs/crates/lean-ctx](https://lib.rs/crates/lean-ctx)

---

## 1. Apa itu lean-ctx

**lean-ctx** = Hybrid Context Optimizer. Single Rust binary, zero dependencies. Dua mekanisme utama:

1. **Shell Hook** — transparent compress CLI output sebelum masuk LLM
2. **MCP Server** — 46+ tools untuk sophisticated context management

```
Tanpa lean-ctx:
  Hermes → jalanin `git log --oneline -50` → 500+ token masuk context

Dengan lean-ctx:
  Hermes → shell hook → lean-ctx compress → ~50 token masuk context
  (cached re-reads turun ke ~13 token)
```

**Klaim**: reduce token consumption 89-99% di shell output + file reads.

---

## 2. Kenapa lo butuh ini

Lo pake OpenCode Go (quota-limited). Terminal output + file reads = **sumber utama token bloat** di agent:

| Sumber | Tanpa lean-ctx | Dengan lean-ctx | Token saved |
|--------|---------------|-----------------|-------------|
| `git log --oneline -50` | 500+ token | ~50 token | ~90% |
| `npm install` output | 1000+ token | ~30 token | ~97% |
| File read (500 lines) | 2000+ token | ~100 token (relevant only) | ~95% |
| Cached file re-read | 2000+ token | ~13 token | ~99% |
| `docker ps` | 300+ token | ~40 token | ~87% |
| Large `ls -la` | 800+ token | ~60 token | ~92% |

Semua ini **langsung nge-save quota** Kimi K2.6 lo yang cuma 1,150 credits.

---

## 3. Fitur Utama

| Fitur | Deskripsi |
|-------|-----------|
| Shell Hook | Transparent compress — 90+ patterns untuk common commands |
| MCP Server | 46+ tools untuk context management via MCP protocol |
| Tree-sitter AST | Parse 18 bahasa — compress code secara cerdas (structure-aware) |
| Cross-session Memory (CCP) | Persistent knowledge antar session |
| Adaptive Compression | Thompson Sampling bandits — belajar optimal compression |
| Task-conditioned Scoring | Relevance scoring berdasarkan task saat ini |
| AAAK Compact Format | Custom format untuk minimal token usage |

---

## 4. Install lean-ctx

### Cargo (recommended kalau lo udah punya Rust)

```bash
cargo install lean-ctx
```

### Install script

```bash
curl -fsSL https://leanctx.com/install.sh | bash
```

### Arch Linux (AUR)

```bash
yay -S lean-ctx-bin
```

### Verify

```bash
lean-ctx --version
which lean-ctx
```

---

## 5. Cara Kerja — Shell Hook

Shell hook lean-ctx intercept output dari terminal commands secara transparent:

```
Hermes mau jalanin: git status
  → Shell hook intercept output
  → lean-ctx apply compression pattern untuk `git`
  → Compressed output (hanya info relevan) masuk ke context LLM
```

Lo gak perlu ubah cara Hermes jalanin command — hook transparent.

---

## 6. Cara Kerja — MCP Server

lean-ctx juga bisa jalan sebagai MCP server, expose 46+ tools ke Hermes:

- Adaptive file reading (10 read modes)
- Context-aware compression
- Cross-session memory management
- Multi-agent context sharing

Setup MCP di Hermes: lihat [docs/05-tools-dan-mcp.md](05-tools-dan-mcp.md)

---

## 7. Supported Commands (90+ patterns)

lean-ctx punya compression patterns untuk 90+ command categories:

| Category | Commands |
|----------|----------|
| Git | `git status`, `git log`, `git diff`, `git branch`, `git stash` |
| Node.js | `npm install`, `npm test`, `yarn`, `pnpm`, `npx` |
| Python | `pip install`, `pytest`, `python -m`, `poetry` |
| Docker | `docker ps`, `docker logs`, `docker build`, `docker compose` |
| System | `ls`, `find`, `df`, `ps`, `top`, `du`, `tree` |
| Build | `make`, `cargo build`, `go build`, `tsc`, `gradle` |
| K8s | `kubectl get`, `kubectl describe`, `kubectl logs` |
| Rust | `cargo build`, `cargo test`, `cargo clippy` |

Kalau command gak punya pattern: lean-ctx tetap apply generic compression (no harm, still saves).

---

## 8. Verify lean-ctx Jalan

### Test manual

```bash
# Jalanin command dan lihat compressed output
lean-ctx shell -- git log --oneline -20
# Output: compressed version (relevant info only)

# Compare token count
lean-ctx stats
# Menampilkan statistik compression ratio
```

### Test di Hermes

```bash
hermes -q "jalanin git status di folder ini"
# Cek: apakah token count per turn lebih rendah dari biasa?
# Lihat di show_cost: true
```

---

## 9. Troubleshoot

| Problem | Fix |
|---------|-----|
| `lean-ctx: command not found` | Install belum kelar. Cek PATH: `echo $PATH`. Restart terminal. |
| Output terlalu di-compress (info penting hilang) | Adjust compression level, atau bypass untuk specific command |
| MCP server gak connect | Cek config MCP di `~/.hermes/mcp.json`. Restart Hermes. |
| Cargo install gagal | Cek Rust version: `rustc --version` (butuh 1.70+) |

---

## 10. Kapan lean-ctx GAK berguna

- **Vision tasks** (gambar, screenshot) — lean-ctx cuma compress teks
- **Web search/extract** — output dari web tool sudah di-handle Hermes sendiri
- **Chat biasa** (Q&A tanpa tool call) — gak ada terminal/file yang dipanggil
- **Very short commands** — output sudah kecil, compression minimal

lean-ctx paling berguna kalau Hermes lo **sering pake terminal** (coding, devops, scraping via CLI, git ops) dan **baca banyak file** (code review, refactor, analysis).

---

## 11. Impact ke Setup lo

Dengan lean-ctx aktif:

| Tanpa lean-ctx | Dengan lean-ctx |
|----------------|-----------------|
| Terminal commands burn 200-500 token/call | ~20-50 token/call |
| File reads burn 500-2000 token/read | ~50-100 token/read (first), ~13 token (cached) |
| K2.6 quota 1,150 habis di ~20-30 coding sessions | Bisa stretch ke ~80-150 sessions |
| Smart routing save 60-70% turns | Smart routing + lean-ctx save 85-95% total token |

Ini **complementary** dengan smart_model_routing:
- Routing hemat di level "turn mana ke model mana"
- lean-ctx hemat di level "berapa token per turn"

---

## 12. Quick Setup (copy-paste)

```bash
# 1. Install lean-ctx
cargo install lean-ctx

# 2. Verify
lean-ctx --version

# 3. Restart Hermes gateway
sudo systemctl restart hermes-agent

# 4. Test — monitor token usage
hermes -q "git status"
# Cek apakah token count per turn lebih rendah dari sebelumnya
```

---

## 🔗 Sumber

- [github.com/yvgude/lean-ctx](https://github.com/yvgude/lean-ctx) — repo resmi
- [leanctx.com](https://leanctx.com) — official site + install
- [lib.rs/crates/lean-ctx](https://lib.rs/crates/lean-ctx) — crate info + docs
