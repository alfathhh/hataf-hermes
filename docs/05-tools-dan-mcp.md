# 05 — Tools, MCP, dan Vision

Tujuan: ngerti tools yang available di Hermes, cara nambah MCP servers, dan setup vision analysis untuk image/PDF berbasis gambar.

> Sumber: [Tools docs](https://hermes-agent.nousresearch.com/docs/user-guide/features/tools), [MCP docs](https://hermes-agent.nousresearch.com/docs/user-guide/features/mcp), [Vision docs](https://hermes-agent.nousresearch.com/docs/user-guide/features/vision).

---

## 1. Apa itu "tools" di Hermes

Tool = function yang LLM bisa panggil. Contoh: `read_file`, `terminal`, `web_search`, `memory`.

Hermes punya **40+ built-in tools** (per dokumentasi resmi). Mereka digrupin ke "toolsets":

| Toolset | Tools utama | Default |
|---|---|---|
| **core** | `read_file`, `write_file`, `edit_file`, `find`, `grep` | enabled |
| **terminal** | `terminal` (run shell command) | enabled |
| **web** | `web_search`, `web_extract`, `web_crawl` | butuh API key (Firecrawl/Tavily/Parallel) |
| **browser** | `browser_navigate`, `browser_screenshot`, `browser_click`, dll | butuh Browserbase atau local Chrome |
| **memory** | `memory` (add/replace/remove) | enabled |
| **skills** | `skills_list`, `skill_view`, `skill_manage` | enabled |
| **cron** | `cronjob` | enabled (di gateway) |
| **delegation** | `delegate_task` | enabled |
| **media** | `vision_analyze`, `image_generate`, `text_to_speech` | butuh API key per fitur |
| **session** | `session_search` | enabled |

Lo bisa lihat & manage:

```bash
hermes tools                  # interactive: enable/disable per toolset
```

---

## 2. Tools yang gw saranin enable, dan yang disable

| Toolset | Status | Alasan |
|---|---|---|
| `core` | ✅ Enable | File ops, gak ada agent tanpa ini |
| `terminal` | ✅ Enable | Tapi pake `approvals.mode: manual` (default) untuk safety |
| `web` | ✅ Enable | Untuk research-citation skill, dll. Butuh Firecrawl key. |
| `memory` | ✅ Enable | Identitas Hermes ada di sini |
| `skills` | ✅ Enable | Default, gak usah otak-atik |
| `cron` | ✅ Enable | Untuk scraping otomatis |
| `delegation` | ✅ Enable | Hemat token dengan subagent murah |
| `browser` | ⚠️ Optional | Enable kalau lo butuh JS-rendered scraping. Setup ada cost (Browserbase) atau perlu Chrome lokal |
| `media.image_generate` | ❌ Disable awalnya | Kecuali lo butuh generate image. Butuh FAL key. |
| `media.text_to_speech` | ⚠️ Optional | Untuk reply voice. Default `edge-tts` gratis. |
| `session.session_search` | ✅ Enable | Cari past conversation, sangat useful |

---

## 3. AGENTS.md — project-level tool preferences

Selain settings global, lo bisa kasih guidance per-project lewat `AGENTS.md` di root project. Hermes auto-load.

Template lengkap: [`examples/AGENTS.md`](../examples/AGENTS.md).

Section yang penting buat tools:

- "Yang harus Hermes hindari" — daftar command/operation berbahaya
- "Tools yang sering dipake" — convention command untuk lint/test/build
- "Build & Run" — biar Hermes tau cara restart dev server, dll

---

## 4. MCP (Model Context Protocol)

MCP = standard protocol untuk extend tool dengan **server eksternal**.

Use case populer:
- **GitHub MCP** — agent bisa create issue, comment, list PR
- **Linear / Jira MCP** — manage tickets
- **Database MCP** — query DB langsung
- **Custom internal MCP** — endpoint internal company lo

### Cara tambah MCP server

Edit `~/.hermes/config.yaml`:

```yaml
mcp:
  github:
    type: stdio
    command: npx
    args: ["-y", "@modelcontextprotocol/server-github"]
    env:
      GITHUB_PERSONAL_ACCESS_TOKEN: ${GITHUB_TOKEN}    # baca dari .env
  
  postgres:
    type: stdio
    command: npx
    args: ["-y", "@modelcontextprotocol/server-postgres", "${POSTGRES_URL}"]
```

Plus tambah secrets ke `.env`:

```bash
GITHUB_TOKEN=ghp_xxxxxxxxxxxx
POSTGRES_URL=postgresql://user:pass@host/db
```

Restart Hermes setelah edit config. Tools dari MCP akan muncul di `hermes tools`.

### Filter MCP tools (optional)

Gak semua tool dari MCP server harus aktif. Bisa filter:

```yaml
mcp:
  github:
    # ... command/args
    allowed_tools:
      - create_issue
      - list_issues
      - create_pull_request
    # tools lain seperti delete_repo, transfer_repo, dll tidak di-load
```

> **Penting buat security**: kalau MCP punya tool destruktif (delete repo, drop table), pakai `allowed_tools` whitelist. Detail di [MCP Config Reference](https://hermes-agent.nousresearch.com/docs/reference/mcp-config-reference).

---

## 5. Vision (analisis gambar)

Hermes pake **auxiliary model terpisah** untuk vision — bukan model utama lo. Default: Gemini Flash via OpenRouter.

### Kenapa beda?

- Model utama lo (DeepSeek V4 Flash) **bukan model vision** (per sumber sekunder yang gw cek)
- Bundle OpenCode Go: gw belum bisa konfirmasi mana model di sana yang punya vision capability resmi
- Auxiliary vision dipanggil cuma saat ada gambar — gak bayar token vision setiap turn

### Setup vision via OpenRouter (recommended)

Di `.env`:

```bash
OPENROUTER_API_KEY=sk-or-xxxxx
```

Di `config.yaml`:

```yaml
auxiliary:
  vision:
    provider: openrouter
    model: google/gemini-2.5-flash    # cepat, murah, cukup pinter
    timeout: 30
```

Atau kalau lo prefer lebih akurat (lebih mahal):

```yaml
auxiliary:
  vision:
    provider: openrouter
    model: openai/gpt-4o              # mahal tapi top-tier
    # atau: anthropic/claude-sonnet-4.5
```

### Tools yang pake vision

- `vision_analyze` — analyze gambar yang lo lampirin
- Browser screenshot analysis — kalau pake `browser_navigate` lalu screenshot
- PDF dengan gambar — kalau PDF text-extraction kasih hasil minim, agent fallback ke vision per halaman

### Limit vision

- Resolusi: kebanyakan provider auto-resize. Gambar gigantis (>10 MB) bisa ditolak.
- Bahasa di gambar: Gemini Flash bagus untuk teks Bahasa Indonesia di gambar (OCR-style).
- Kompleks reasoning di gambar: GPT-4o lebih kuat untuk diagram/chart.

### Vision di luar auxiliary

Lo juga bisa **manual call** vision untuk task spesifik:

```
> [attach gambar] tolong baca isi receipt ini, kasih JSON {merchant, total, date, items}
```

Hermes auto-route ke `vision_analyze` tool.

---

## 6. Web search backend pilihan

Tool `web_search` / `web_extract` butuh backend API. Hermes support:

| Backend | Pricing model | Strength | Weakness |
|---|---|---|---|
| **Firecrawl** | Subscription (free tier ada) | Fire-engine anti-bot, paling lengkap | Paling mahal kalau heavy use |
| **Tavily** | Per-search pricing | Fokus search-quality, hasil terstruktur | Crawl tidak sebanyak Firecrawl |
| **Parallel** | Per-search pricing | Search modes (fast / agentic) | Belum support `web_crawl` |
| **Self-hosted Firecrawl** | Free (lo provide infra) | Privacy, no per-page cost | Tidak punya Fire-engine cloud (Cloudflare bypass terbatas) |

Set di `config.yaml`:

```yaml
web:
  backend: firecrawl     # atau: tavily / parallel
```

API key di `.env`:

```bash
FIRECRAWL_API_KEY=fc-xxxxx
# atau:
TAVILY_API_KEY=tvly-xxxxx
PARALLEL_API_KEY=para-xxxxx
```

Kalau set 2-3 backend keys, Hermes auto-detect (urutan: Tavily → Parallel → Firecrawl, kalau cuma 1 yang tersedia).

### Self-host Firecrawl

Kalau heavy use dan ingin hemat:

```bash
git clone https://github.com/firecrawl/firecrawl
cd firecrawl
# di .env Firecrawl: USE_DB_AUTHENTICATION=false, HOST=0.0.0.0, PORT=3002
docker compose up -d

# Point Hermes ke instance lokal lo
hermes config set FIRECRAWL_API_URL http://localhost:3002
```

> Caveat: self-hosted Firecrawl pake basic fetch + Playwright. Cloudflare-protected sites bisa gagal. Untuk sites yang ekstrim anti-bot, tetap pake cloud Firecrawl atau Tavily.

---

## 7. Browser automation

Buat scrape JS-heavy sites atau interactive workflow (login flow, fill form), enable browser tools.

3 opsi backend:

### Opsi 1: Browserbase (cloud, paling reliable)

```bash
# .env
BROWSERBASE_API_KEY=bb_xxxxx
BROWSERBASE_PROJECT_ID=xxxxx
```

### Opsi 2: Browser Use (local)

Install:

```bash
pip install browser-use
```

Default chrome lokal. Lebih murah tapi gak portable ke server headless.

### Opsi 3: Local Chrome via CDP

```yaml
browser:
  inactivity_timeout: 120
  record_sessions: false
```

Detail per opsi: [Browser feature page](https://hermes-agent.nousresearch.com/docs/user-guide/features/browser).

---

## 8. Quick commands (zero LLM cost)

Selain tools, Hermes punya **quick commands** — shell shortcut yang dijalankan langsung TANPA LLM.

Edit `config.yaml`:

```yaml
quick_commands:
  status:
    type: exec
    command: systemctl status hermes-agent
  disk:
    type: exec
    command: df -h /
  uptime:
    type: exec
    command: uptime
```

Pakai:

```
/status            # langsung jalan, gak burn token
/disk
/uptime
```

Sangat berguna buat ops check via Telegram. Detail di doc 09 (token optimization).

---

## 9. "tools.md" — kemana mappingnya

Lo nyebut "tools.md" di permintaan awal. **Ini bukan file native Hermes**. Yang ada:

| Yang lo expect | Yang sebenarnya |
|---|---|
| `tools.md` di root project | `~/.hermes/config.yaml` (`mcp:` section + `quick_commands:`) |
| Tools list dokumentasi | `hermes tools` (interactive) atau [Tools Reference](https://hermes-agent.nousresearch.com/docs/reference/tools-reference) |
| Tool config per project | `AGENTS.md` (yang lo tulis aturan tool usage) |

Jadi: **gak ada `tools.md`**. Konfigurasinya tersebar di config.yaml, AGENTS.md, dan `hermes tools` CLI.

Ini bukan kelemahan — ini desain. Tools = secrets (API keys) + permissions (per project), jadi gak masuk akal di-flatten ke 1 file markdown.

---

## 10. Yang gw belum yakin

- **MCP server availability**: ekosistem MCP cepet berkembang. Server-server populer (GitHub, Postgres, Slack) tersedia, tapi nama package npm-nya bisa berubah. Cek [modelcontextprotocol.io](https://modelcontextprotocol.io) untuk listing terbaru.
- **Vision quality untuk PDF Indonesia**: gw belum benchmark Gemini Flash vs GPT-4o spesifik untuk dokumen formal Bahasa Indonesia (KTP, akta, regulasi). Kalau lo punya use case spesifik, test keduanya dulu di sample dokumen.
- **Browser cost di Browserbase**: pricing per session minute, bisa cepet naik kalau loop scraping. Estimasi gw: $1 untuk ~100-500 page navigation. **Cek halaman pricing Browserbase** untuk angka pasti.

---

## Lanjut

→ [06 — Memory & USER Profile](06-memory.md): bikin agent inget preferensi lo lintas sesi tanpa boros token.
