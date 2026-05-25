# Hataf Hermes — Tutorial Lengkap Hermes Agent

Tutorial step-by-step buat ngebangun Hermes Agent (by [Nous Research](https://nousresearch.com)) jadi alat AI personal yang **jujur, anti-halu, hemat token, dan jalan 24/7 lewat Telegram**.

> Stack: Hermes Agent + OpenCode Go bundle (Kimi K2.6 / DeepSeek V4 Flash / Qwen) + OpenRouter (vision) + Telegram gateway + cronjob + RTK token optimizer.
>
> **Semua model via OpenCode Go** — tidak butuh API key DeepSeek, OpenAI, atau provider lain terpisah.

---

## ⚠️ Sebelum mulai — baca ini dulu

### Hermes Agent itu apa

- Framework agent open-source dari Nous Research — repo: [github.com/NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent)
- Jalan di **Linux/macOS/WSL2** (Windows native = early beta)
- **Bukan model** — dia butuh provider LLM (OpenCode Go, OpenRouter)
- Punya memory persisten, skills system, cronjob, messaging gateway (Telegram, Discord, dll)

### Arsitektur akhir

```
Lo (Telegram di HP)
        │
Telegram Bot (BotFather token)
        │
Hermes Gateway daemon (VPS/lokal)
  ├── SOUL.md        (identity + anti-halu + execution rules)
  ├── MEMORY.md      (fakta persisten)
  ├── USER.md        (profil lo)
  ├── 25 Skills      (knowledge on-demand, optimized for weak models)
  └── Cronjob        (automation background)
        │
Smart Router
  ├── Turn kompleks  → Primary model (K2.6 / Flash)
  ├── Turn simpel    → Cheap model (Qwen3.5 Plus / Flash)
  └── Fallback       → Secondary model
        │
RTK (Rust Token Killer) → compress terminal output 60-90%
```

### Provider yang dibutuhkan

| Provider | Kegunaan | Wajib? |
|----------|----------|--------|
| **OpenCode Go** | Semua model utama (K2.6, Flash, Qwen, dll) | ✅ Ya |
| **OpenRouter** | Vision auxiliary (Gemini Flash) | ✅ Ya (untuk gambar) |
| Firecrawl | Web scraping backend | Optional |

**Tidak perlu**: API key DeepSeek, OpenAI, Anthropic, atau provider lain.

---

## 📊 Model Strategy — CHEAP vs BALANCED

### Strategy 1: CHEAP (~$10-15/bulan)

**Siapa yang cocok**: Budget ketat, personal use, daily chat + scraping + cron

```
Primary:     DeepSeek V4 Flash  (OpenCode Go — 31,650 credits, 1M context)
Cheap route: Qwen3.5 Plus       (OpenCode Go — 10,200 credits, paling irit)
Fallback:    DeepSeek V4 Pro    (OpenCode Go — 3,450 credits)
Vision:      Gemini 2.5 Flash   (OpenRouter  — ~$1-2/bulan)
Compression: Qwen3.5 Plus
Delegation:  Qwen3.5 Plus
```

| Aspek | Detail |
|-------|--------|
| **Cost** | ~$10-15/bulan |
| **Quality** | Good — adequate untuk daily use + coding sederhana |
| **Context** | 1M token (Flash) |
| **Kecepatan** | Cepat |
| **Kekurangan** | Skill instructions HARUS eksplisit (decision tree format) |

**Config highlights** (`config-cheap.yaml`):
```yaml
model:
  provider: opencode-go
  default: deepseek-v4-flash

smart_model_routing:
  enabled: true
  cheap_model:
    provider: opencode-go
    model: qwen3.5-plus

agent:
  reasoning_effort: medium
```

---

### Strategy 2: BALANCED (~$11-15/bulan)

**Siapa yang cocok**: Coding serius, code review, arsitektur, mixed use case

```
Primary:     Kimi K2.6          (OpenCode Go — 1,150 credits, near-Opus)
Cheap route: DeepSeek V4 Flash  (OpenCode Go — 31,650 credits)
Fallback:    DeepSeek V4 Pro    (OpenCode Go — 3,450 credits)
Vision:      Gemini 2.5 Flash   (OpenRouter)
Compression: DeepSeek V4 Flash
Delegation:  Qwen3.6 Plus
```

| Aspek | Detail |
|-------|--------|
| **Cost** | ~$11-15/bulan (OpenCode Go $10 + OpenRouter $1-3) |
| **Quality** | Near-Opus untuk coding (SWE-Bench Pro 58.6%) |
| **Context** | K2.6 = 128K-256K |
| **Kekurangan** | Quota K2.6 terbatas (1,150 credits) — routing WAJIB ON |

**Config highlights** (`config-balanced.yaml`):
```yaml
model:
  provider: opencode-go
  default: kimi-k2.6

smart_model_routing:
  enabled: true
  cheap_model:
    provider: opencode-go
    model: deepseek-v4-flash

fallback_model:
  provider: opencode-go
  model: deepseek-v4-pro
```

---

### Strategy 3: HYBRID (~$12-20/bulan)

**Siapa yang cocok**: Power user, heavy coding + daily use, squeeze savings maksimal

```
Primary:     Kimi K2.6          (turn coding/complex saja)
Cheap route: DeepSeek V4 Flash  (semua turn simple — threshold lebih longgar)
Fallback:    DeepSeek V4 Flash  (bukan Pro — lebih hemat)
```

**Bedanya dari BALANCED**: threshold routing lebih longgar (200 char / 35 kata vs 160 / 28) → K2.6 lebih jarang dipanggil → lebih hemat.

**Config highlights** (`config-hybrid.yaml`):
```yaml
model:
  provider: opencode-go
  default: kimi-k2.6

smart_model_routing:
  enabled: true
  max_simple_chars: 200        # lebih longgar dari balanced (160)
  max_simple_words: 35         # lebih longgar dari balanced (28)
  cheap_model:
    provider: opencode-go
    model: deepseek-v4-flash

fallback_model:
  provider: opencode-go
  model: deepseek-v4-flash     # Flash, bukan Pro
```

---

### Tabel Perbandingan

| | CHEAP | BALANCED | HYBRID | STRONG |
|---|---|---|---|---|
| **Cost/bulan** | $10-15 | $11-15 | $12-20 | $13-20 |
| **Quality ceiling** | Good | Near-Opus | Near-Opus | Near-Opus |
| **Reasoning effort** | medium | medium | medium | **high** |
| **Quality floor** | Qwen3.5 Plus | Flash | Flash | Flash |
| **Best for** | Daily chat, cron | Coding, review | Mixed heavy | Deep coding, arsitektur |
| **lean-ctx MCP** | Optional | Optional | Optional | **Included** |
| **RTK benefit** | High | Medium | High | Replaced by lean-ctx |

### Cara Pilih

```
IF cuma chat + cron + scraping       → CHEAP
IF coding serius + code review       → BALANCED
IF mixed heavy (coding + daily)      → HYBRID
IF mau maksimalkan cache savings     → config-prefix-cache.yaml
IF mau quality terbaik + deep reason → STRONG (config-strong.yaml)
```

### File Config

| File | Strategy | Primary |
|------|----------|---------|
| [`examples/config-cheap.yaml`](examples/config-cheap.yaml) | CHEAP | DeepSeek V4 Flash |
| [`examples/config-balanced.yaml`](examples/config-balanced.yaml) | BALANCED | Kimi K2.6 |
| [`examples/config-hybrid.yaml`](examples/config-hybrid.yaml) | HYBRID | K2.6 + Flash aggressive routing |
| [`examples/config-prefix-cache.yaml`](examples/config-prefix-cache.yaml) | BALANCED + KV cache opt | Kimi K2.6 |
| [`examples/config-strong.yaml`](examples/config-strong.yaml) | STRONG | K2.6 high reasoning + lean-ctx |
| [`examples/config.yaml`](examples/config.yaml) | BALANCED (default lengkap) | Kimi K2.6 |

```bash
# Pilih salah satu:
cp examples/config-cheap.yaml ~/.hermes/config.yaml
cp examples/config-balanced.yaml ~/.hermes/config.yaml
cp examples/config-hybrid.yaml ~/.hermes/config.yaml
```

---

## 🤖 Multi-Profile Telegram — Per Workflow

Setup multi-bot di 1 server. Tiap bot = 1 workflow/domain, personality sendiri, skill set sendiri.

| Profile | Workflow | Bot | Model | Skills |
|---------|----------|-----|-------|--------|
| `main` | Meta / General | @hataf_main_bot | K2.6 medium | Semua 25 |
| `code` | Development | @hataf_code_bot | K2.6 high | coding-mentor, web-dev, backend, frontend, code-review, devops |
| `research` | Research & Analysis | @hataf_research_bot | Flash high (1M ctx) | deep-analysis, research-citation, study-buddy, pdf-summarize, video-summary |
| `islam` | Islamic Study | @hataf_islam_bot | Flash high | islamic-study, research-citation |
| `finance` | Finance & Monitor | @hataf_money_bot | Flash medium + routing | financial-literacy, saham-syariah, harga-emas, web-scrape |
| `media` | Content & Media | @hataf_media_bot | Qwen3.6 low | copywriting, music-download, social-media-scrape, video-summary |

Setup detail lengkap: [docs/11-multi-profile-telegram.md](docs/11-multi-profile-telegram.md)

---

## 📚 Daftar Isi

### Docs Tutorial (15 docs)

| # | File | Isi |
|---|------|-----|
| 01 | [instalasi-quickstart](docs/01-instalasi-quickstart.md) | Install dari nol, chat pertama < 5 menit |
| 02 | [providers-dan-models](docs/02-providers-dan-models.md) | Setup OpenCode Go + rekomendasi model per use case |
| 03 | [soul-md](docs/03-soul-md.md) | SOUL.md anti-halu + cara test |
| 04 | [skills](docs/04-skills.md) | Sistem skills, install, cara pakai |
| 05 | [tools-dan-mcp](docs/05-tools-dan-mcp.md) | Tools, MCP servers, vision |
| 06 | [memory](docs/06-memory.md) | MEMORY.md + USER.md, cap management |
| 07 | [cron-scraping](docs/07-cron-scraping.md) | Scheduled tasks, 8 resep cronjob |
| 08 | [telegram-gateway](docs/08-telegram-gateway.md) | Bot Telegram lengkap + voice + group |
| 09 | [optimasi-token](docs/09-optimasi-token.md) | Smart routing, compression, semua knob hemat |
| 10 | [mengajarkan-hermes](docs/10-mengajarkan-hermes.md) | Feedback loop, agent makin pinter over time |
| 11 | [multi-profile-telegram](docs/11-multi-profile-telegram.md) | Multi-bot Telegram per workflow (6 profil lengkap) |
| 12 | [wsl-linux-commands](docs/12-wsl-linux-commands.md) | Cheat sheet WSL/Linux — navigasi, buka file, edit, troubleshoot |
| 13 | [rtk-token-saver](docs/13-rtk-token-saver.md) | RTK install — hemat 60-90% token terminal |
| 14 | [prefix-cache-optimization](docs/14-prefix-cache-optimization.md) | KV Prefix Cache (teknik DeepSeek Reasonix) — hemat 40-90% |
| 15 | [lean-ctx-context-optimizer](docs/15-lean-ctx-context-optimizer.md) | lean-ctx: shell hook + MCP file compression — hemat 89-99% |

---

## 🎯 25 Skills (Optimized for Weak Models)

> Semua skill di-refine dengan format decision tree + explicit instructions + contoh output.

### Meta / Workflow
| Skill | Fungsi |
|-------|--------|
| [`claude-superpowers`](examples/skills/claude-superpowers/SKILL.md) | Multi-capability workflow (thinking + artifacts + citations + vision + code exec) |
| [`anti-hallucination-review`](examples/skills/anti-hallucination-review/SKILL.md) | Checklist anti-halu sebelum jawab high-stakes |
| [`weekly-review`](examples/skills/weekly-review/SKILL.md) | Ritual mingguan: review sesi, update memory/skill |

### Research & Analysis
| Skill | Fungsi |
|-------|--------|
| [`research-citation`](examples/skills/research-citation/SKILL.md) | Cari fakta, wajib citation, "tidak ditemukan" kalau gak ada |
| [`deep-analysis`](examples/skills/deep-analysis/SKILL.md) | Multi-perspektif: decompose, trade-off matrix, conditional recommendation |
| [`study-buddy`](examples/skills/study-buddy/SKILL.md) | NotebookLM-style — jawab HANYA dari bahan yang dikasih |

### Development
| Skill | Fungsi |
|-------|--------|
| [`coding-mentor`](examples/skills/coding-mentor/SKILL.md) | Belajar coding dari NOL, analogi, 6 roadmap |
| [`web-development`](examples/skills/web-development/SKILL.md) | Fullstack — stack-aware, verify version, plan before code |
| [`backend-proper`](examples/skills/backend-proper/SKILL.md) | Backend serius — security, idempotency, observability |
| [`frontend-design`](examples/skills/frontend-design/SKILL.md) | UI anti-generic, bold aesthetic |
| [`ui-ux`](examples/skills/ui-ux/SKILL.md) | Review UI/UX — WCAG/Material/HIG |
| [`code-review`](examples/skills/code-review/SKILL.md) | Code review severity-based 🔴🟠🟡🟢 |
| [`devops-networking`](examples/skills/devops-networking/SKILL.md) | Server, Docker, CI/CD, nginx, SSL |
| [`data-analysis`](examples/skills/data-analysis/SKILL.md) | pandas + matplotlib — load→clean→visualize→insight |

### Daily Tasks / Media
| Skill | Fungsi |
|-------|--------|
| [`pdf-summarize`](examples/skills/pdf-summarize/SKILL.md) | Summary PDF per section + referensi halaman |
| [`video-summary`](examples/skills/video-summary/SKILL.md) | yt-dlp + Whisper + LLM — summary + timestamp |
| [`music-download`](examples/skills/music-download/SKILL.md) | YouTube, Tidal lossless, Spotify |
| [`copywriting`](examples/skills/copywriting/SKILL.md) | Caption IG, artikel, email, landing page, thread X |

### Finance
| Skill | Fungsi |
|-------|--------|
| [`financial-literacy`](examples/skills/financial-literacy/SKILL.md) | Edukator — budgeting, compound interest, produk keuangan |
| [`saham-syariah`](examples/skills/saham-syariah/SKILL.md) | Screening saham syariah IDX — DES OJK, fundamental |
| [`harga-emas`](examples/skills/harga-emas/SKILL.md) | Harga emas Antam HANYA dari logammulia.com |

### Religion
| Skill | Fungsi |
|-------|--------|
| [`islamic-study`](examples/skills/islamic-study/SKILL.md) | Cari ayat/hadits/fiqh dari sumber terpercaya |

### Scraping
| Skill | Fungsi |
|-------|--------|
| [`web-scrape`](examples/skills/web-scrape/SKILL.md) | Scraping schema-first, fallback browser, JSON output |
| [`marketplace-scrape`](examples/skills/marketplace-scrape/SKILL.md) | Tokopedia, Shopee, Bukalapak, Lazada |
| [`social-media-scrape`](examples/skills/social-media-scrape/SKILL.md) | Instagram, X/Twitter, Facebook, TikTok |

---

## ⚙️ File Config

| File | Fungsi |
|------|--------|
| [`examples/SOUL.md`](examples/SOUL.md) | Identity + anti-halu + rules |
| [`examples/AGENTS.md`](examples/AGENTS.md) | Project conventions + skill routing |
| [`examples/config.yaml`](examples/config.yaml) | BALANCED lengkap (default) |
| [`examples/config-cheap.yaml`](examples/config-cheap.yaml) | CHEAP — Flash primary |
| [`examples/config-balanced.yaml`](examples/config-balanced.yaml) | BALANCED — K2.6 primary |
| [`examples/config-hybrid.yaml`](examples/config-hybrid.yaml) | HYBRID — K2.6 + Flash aggressive routing |
| [`examples/config-prefix-cache.yaml`](examples/config-prefix-cache.yaml) | BALANCED + KV cache optimized |
| [`examples/config-strong.yaml`](examples/config-strong.yaml) | STRONG — K2.6 high + lean-ctx MCP, ~$13-20/bln |
| [`examples/.env.example`](examples/.env.example) | Template env vars |

---

## 🚀 Quickstart

```bash
# 1. Install Hermes
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
source ~/.bashrc

# 2. Copy config
cp examples/SOUL.md ~/.hermes/SOUL.md
cp examples/config-balanced.yaml ~/.hermes/config.yaml   # atau pilih strategy lain
cp examples/.env.example ~/.hermes/.env
# Edit .env → isi OPENCODE_GO_API_KEY, OPENROUTER_API_KEY, TELEGRAM_BOT_TOKEN

# 3. Install semua skills
cp -r examples/skills/* ~/.hermes/skills/

# 4. Install RTK (hemat token terminal 60-90%)
curl -fsSL https://raw.githubusercontent.com/rtk-ai/rtk/master/install.sh | bash
pip install rtk-hermes

# 5. Setup Telegram
hermes gateway setup

# 6. Start
hermes gateway install --system
sudo systemctl start hermes-agent

# 7. Test
hermes -q "halo, perkenalkan dirimu"
```

---

## 🖥️ WSL Quick Reference

Lo pake Windows? Ini command yang paling sering dibutuhkan di WSL.

### Buka & baca file

```bash
cat ~/.hermes/config.yaml          # tampilkan isi file di terminal
cat ~/.hermes/SOUL.md              # baca SOUL.md
less ~/.hermes/config.yaml         # baca panjang (q untuk quit)
head -20 ~/.hermes/SOUL.md         # tampilkan 20 baris pertama
tail -20 ~/.hermes/SOUL.md         # tampilkan 20 baris terakhir
```

### Edit file

```bash
nano ~/.hermes/config.yaml         # edit (Ctrl+O save, Ctrl+X keluar)
nano ~/.hermes/SOUL.md
nano ~/.hermes/.env                # edit API keys

# Alternatif: buka di VS Code dari WSL
code ~/.hermes/config.yaml
code ~/.hermes/SOUL.md
```

### Buka folder di File Explorer Windows

```bash
explorer.exe .                     # buka folder WSL ini di Explorer
explorer.exe ~/.hermes             # buka folder .hermes di Explorer
```

### Navigasi

```bash
pwd                                # di mana sekarang?
ls                                 # isi folder
ls -la ~/.hermes                   # isi .hermes dengan detail
cd ~/.hermes                       # masuk ke folder hermes
cd ~/.hermes/skills                # masuk ke folder skills
cd ..                              # naik 1 level
cd ~                               # balik ke home
```

### Copy & install skill

```bash
# Install 1 skill
cp -r examples/skills/coding-mentor ~/.hermes/skills/

# Install semua skill sekaligus
cp -r examples/skills/* ~/.hermes/skills/

# Cek skill terinstall
ls ~/.hermes/skills/
hermes skills list
```

### Cek & restart Hermes

```bash
sudo systemctl status hermes-agent         # running?
sudo systemctl restart hermes-agent        # restart setelah edit config
journalctl -u hermes-agent -f              # lihat log real-time (Ctrl+C stop)
hermes doctor                              # diagnose masalah
```

### Windows ↔ WSL path

```bash
# File di Windows bisa diakses via /mnt/
ls /mnt/c/Users/nama/Downloads/
cp /mnt/c/Users/nama/Downloads/dokumen.pdf ~/

# Buka folder WSL dari Windows Explorer:
explorer.exe ~/.hermes
```

Detail lengkap: [docs/12-wsl-linux-commands.md](docs/12-wsl-linux-commands.md)

---

## 🧠 Tips: Skill + Model Murah

Kalau lo pake DeepSeek V4 Flash atau Qwen3.5 (cheap strategy):

1. **Skill format decision tree** — model murah follow `IF X → DO Y`, bukan prose
2. **SOUL.md explicit rules** — tulis sebagai `DO NOT: ...` dan `DO: ...`
3. **Contoh output di setiap skill** — model murah butuh "template"
4. **Reasoning effort = medium** default — `high` bikin thinking token meledak
5. **RTK aktif** — compress verbose terminal output
6. **SOUL.md jangan sering diedit** — tiap edit = cache bust, biaya naik 1-2 turn

---

## 📖 Urutan baca

**Baru pertama kali**: 01 → 03 → 02 → 08 → sisanya sesuai kebutuhan

**Sudah familiar**: install (01) → copy SOUL.md + config (03, 09) → Telegram (08) → install skills → done.

Jangan setup semuanya hari pertama — pake 1 minggu dulu, baru tambah fitur.

---

## 📝 Prinsip

1. **Honesty over confidence** — bilang "gak tau" daripada ngarang
2. **Source-first** — setiap klaim ada sumber yang bisa dicek
3. **Token efficiency** — routing, compression, RTK, prefix cache
4. **Never invent** — fakta, path, API → verify dulu
5. **Explicit over implicit** — instructions harus diikuti model murah tanpa "infer"

---

## 🔗 Referensi

- [Hermes Agent docs](https://hermes-agent.nousresearch.com/docs/)
- [GitHub repo](https://github.com/NousResearch/hermes-agent)
- [OpenCode Go](https://opencode.ai/go) — subscription + model list
- [RTK (Rust Token Killer)](https://github.com/rtk-ai/rtk)
- [agentskills.io](https://agentskills.io/specification) — SKILL.md spec

---

## Lisensi

Hermes Agent: MIT — © Nous Research.
Tutorial ini disusun dengan referensi ke dokumentasi resmi.
