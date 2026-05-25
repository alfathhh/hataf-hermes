# Hataf Hermes — Tutorial Lengkap Hermes Agent

Tutorial step-by-step buat ngebangun Hermes Agent (by [Nous Research](https://nousresearch.com)) jadi alat AI personal yang **jujur, anti-halu, hemat token, dan jalan 24/7 lewat Telegram**.

> Stack: Hermes Agent + DeepSeek V4 Flash (primary cheap) / Kimi K2.6 (primary balanced) + Telegram gateway + cronjob + RTK token optimizer.

---

## ⚠️ Sebelum mulai — baca ini dulu

### Hermes Agent itu apa

- Framework agent open-source dari Nous Research — repo: [github.com/NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent)
- Jalan di **Linux/macOS/WSL2** (Windows native = early beta)
- **Bukan model** — dia butuh provider LLM (OpenCode Go, DeepSeek, OpenRouter, dll)
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
  ├── Turn kompleks  → Primary model
  ├── Turn simpel    → Cheap model
  └── Fallback       → Secondary model
        │
RTK (Rust Token Killer) → compress terminal output 60-90%
```

---

## 📊 Model Strategy — CHEAP vs BALANCED

### Strategy 1: CHEAP (DeepSeek V4 Flash only — ~$5-15/bulan)

**Siapa yang cocok**: Budget ketat, personal use, task gak terlalu kompleks

```
Primary:      DeepSeek V4 Flash (1M context, murah banget)
Cheap route:  Qwen3.5 Plus (via OpenCode Go) — untuk "halo", translate, format
Fallback:     DeepSeek V4 Pro — kalau Flash rate limit
Vision:       Gemini 2.5 Flash (OpenRouter, ~$1-2/bulan)
Compression:  Qwen3.5 Plus
Delegation:   Qwen3.5 Plus
```

| Aspek | Detail |
|-------|--------|
| **Cost** | ~$5-15/bulan total |
| **Quality** | 70-80% dari Claude/GPT-4o untuk coding |
| **Context** | 1M token (panjang banget) |
| **Kecepatan** | Sangat cepat (Flash optimized) |
| **Kekurangan** | Bisa miss nuance, perlu skill instructions eksplisit |

**Config highlights**:
```yaml
model:
  provider: custom
  default: deepseek-v4-flash
  base_url: https://api.deepseek.com/v1
  context_length: 1000000

smart_model_routing:
  enabled: true
  cheap_model:
    provider: opencode-go
    model: qwen3.5-plus

agent:
  reasoning_effort: medium     # low untuk daily Q&A, high untuk coding
```

**⚠️ Penting untuk cheap strategy**:
- Skills HARUS optimized (decision tree format, explicit instructions)
- SOUL.md HARUS punya explicit rules (model murah gak bisa "infer" niat)
- Reasoning effort: set `medium` default, `high` cuma saat coding/debug
- Smart routing: WAJIB ON — 60-70% turn lo gak perlu Flash

---

### Strategy 2: BALANCED (OpenCode Go primary — ~$11-15/bulan)

**Siapa yang cocok**: Quality penting, budget tetap terkontrol, mixed use case

```
Primary:      Kimi K2.6 (near-Opus, SWE-Bench 58.6%)
Cheap route:  DeepSeek V4 Flash (31,650 credits = paling irit)
Fallback:     DeepSeek V4 Pro (3,450 credits)
Vision:       Gemini 2.5 Flash (OpenRouter)
Compression:  DeepSeek V4 Flash
Delegation:   Qwen3.6 Plus
```

| Aspek | Detail |
|-------|--------|
| **Cost** | ~$11-15/bulan (OpenCode Go $10 + OpenRouter $1-3) |
| **Quality** | Near-Opus level untuk coding tasks |
| **Context** | Tergantung model (K2.6 = 128K-256K) |
| **Kecepatan** | Medium (K2.6 lebih lambat dari Flash) |
| **Kekurangan** | Quota K2.6 terbatas (1,150 credits) |

**Config highlights**:
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

### Strategy 3: HYBRID (Best quality + aggressive cost control)

**Siapa yang cocok**: Power user, butuh quality tinggi tapi gak mau bayar $50+

```
Primary:      Kimi K2.6 (turn coding/complex SAJA)
Cheap route:  DeepSeek V4 Flash (SEMUA turn simple)
Fallback:     DeepSeek V4 Flash (bukan Pro — karena Flash free-ish)
Cron/automation: Qwen3.5 Plus (murah, cron prompts simple)
Research:     DeepSeek V4 Flash (web_search heavy)
```

**Kuncinya**: Smart routing yang ketat — K2.6 cuma dipanggil kalau:
- Turn ada keyword coding/debug/refactor
- Turn > 28 kata DAN complex pattern detected
- Sisanya SEMUA ke Flash

```yaml
smart_model_routing:
  enabled: true
  max_simple_chars: 200         # lebih longgar → lebih banyak ke cheap
  max_simple_words: 35          # naikin threshold
  cheap_model:
    provider: custom
    model: deepseek-v4-flash
    base_url: https://api.deepseek.com/v1
```

---

### Tabel Perbandingan Strategy

| | CHEAP | BALANCED | HYBRID |
|---|---|---|---|
| **Cost/bulan** | $5-15 | $11-15 | $12-20 |
| **Quality ceiling** | Medium-High | Very High | Very High |
| **Quality floor** | Low (Qwen3.5) | Medium (Flash) | Medium (Flash) |
| **Best for** | Daily chat, scraping, cron | Coding, analysis | Mixed heavy use |
| **Skill format needed** | Decision tree (explicit) | Prose OK | Decision tree preferred |
| **Smart routing** | WAJIB | WAJIB | WAJIB (aggressive) |
| **RTK benefit** | High | Medium | High |

---

### Cara Pilih Strategy

```
IF budget < $10/bulan → CHEAP
IF coding heavy + quality matters → BALANCED atau HYBRID
IF mostly cron + scraping + daily Q&A → CHEAP
IF need long context (1M token) → CHEAP (Flash punya 1M)
IF need best reasoning → BALANCED (K2.6 near-Opus)
IF mixed (coding + daily + research) → HYBRID
```

### File Config

| File | Strategy | Primary model |
|------|----------|---------------|
| [`examples/config-cheap.yaml`](examples/config-cheap.yaml) | CHEAP | DeepSeek V4 Flash |
| [`examples/config-balanced.yaml`](examples/config-balanced.yaml) | BALANCED | Kimi K2.6 |
| [`examples/config-hybrid.yaml`](examples/config-hybrid.yaml) | HYBRID | K2.6 (complex) + Flash (simple) |
| [`examples/config.yaml`](examples/config.yaml) | BALANCED (default) | Kimi K2.6 |

Cara pakai:
```bash
# Pilih salah satu, copy ke ~/.hermes/config.yaml
cp examples/config-cheap.yaml ~/.hermes/config.yaml       # budget ketat
cp examples/config-balanced.yaml ~/.hermes/config.yaml    # kualitas terbaik
cp examples/config-hybrid.yaml ~/.hermes/config.yaml      # balance quality + cost
```

---

## 🤖 Multi-Profile Telegram — Organized by Workflow

Setup multi-bot di 1 server. Tiap bot = 1 workflow/domain, personality beda, skill set beda.

### Rekomendasi Profile Organization

| Profile | Workflow | Bot | Model | Skills | Personality |
|---------|----------|-----|-------|--------|-------------|
| `main` | **Meta / General** | @hataf_main_bot | Strategy utama lo | Semua 25 skill | General purpose, anti-halu |
| `code` | **Development** | @hataf_code_bot | Kimi K2.6 atau Flash (high reasoning) | coding-mentor, web-dev, backend, frontend, code-review, devops, data-analysis | Temen coding, eksekutor |
| `research` | **Research & Analysis** | @hataf_research_bot | DeepSeek V4 Flash (1M context) | deep-analysis, research-citation, study-buddy, pdf-summarize, video-summary | Researcher, citation-enforcer |
| `islam` | **Islamic Study** | @hataf_islam_bot | DeepSeek V4 Flash (high reasoning) | islamic-study, research-citation | Asisten ilmu, anti-halu dalil |
| `finance` | **Finance & Monitoring** | @hataf_money_bot | DeepSeek V4 Flash + smart routing | financial-literacy, saham-syariah, harga-emas, marketplace-scrape | Edukator, kalkulator |
| `media` | **Content & Media** | @hataf_media_bot | Flash (low reasoning) | copywriting, music-download, social-media-scrape, video-summary | Content creator assistant |

### Kenapa organize per workflow (bukan random)?

```
SEBELUM (campur):
  1 bot → user minta coding → agent load semua 25 skill → token boros
  1 bot → context coding campur sama doa + harga emas → noisy

SESUDAH (per workflow):
  Bot coding → cuma load 7 skill coding → token hemat
  Bot coding → context pure coding → quality naik
  Bot finance → cronjob khusus finance → deliver ke grup yang tepat
```

### Resource per profile

```
Per gateway process: ~50-150 MB RAM
Total 6 profile: ~600-900 MB RAM

Rekomendasi VPS:
- Budget: 2 GB RAM (fit 4 profile comfortable)
- Recommended: 4 GB RAM (fit 6+ profile + headroom)
- Overkill: 8 GB RAM (kalau mau future-proof)
```

### Model allocation per profile (CHEAP strategy)

| Profile | Model | Reasoning | Why |
|---------|-------|-----------|-----|
| `main` | DeepSeek V4 Flash | medium | General purpose |
| `code` | DeepSeek V4 Flash | high | Coding butuh reasoning kuat |
| `research` | DeepSeek V4 Flash | high | Analysis butuh depth |
| `islam` | DeepSeek V4 Flash | high | Agama HARUS akurat (anti-halu) |
| `finance` | DeepSeek V4 Flash | medium | Mix screening + Q&A |
| `media` | Qwen3.5 Plus | low | Content generation gak perlu deep |

### Model allocation per profile (BALANCED strategy)

| Profile | Model | Reasoning | Why |
|---------|-------|-----------|-----|
| `main` | Kimi K2.6 | medium | Best quality for general |
| `code` | Kimi K2.6 | high | Top coding quality |
| `research` | DeepSeek V4 Flash | high | 1M context for long docs |
| `islam` | DeepSeek V4 Flash | high | Web search heavy, context long |
| `finance` | DeepSeek V4 Flash + routing | medium | Cron tasks mostly |
| `media` | Qwen3.6 Plus | low | Cheap, adequate for writing |

### Cronjob Distribution

```
Profile: finance
├── Harga emas harian (logammulia.com) — 09:00 WIB weekdays
├── Weekly screening saham syariah — Jumat 20:00
└── DES update check — 1 Mei & 1 November

Profile: main
├── Weekly self-improvement review — Minggu 20:00
└── System health check — setiap 6 jam

Profile: research
└── (On-demand, no cron)

Profile: code
└── Daily coding tip (optional) — 08:00
```

Setup detail lengkap: [docs/11-multi-profile-telegram.md](docs/11-multi-profile-telegram.md)

---

## 📚 Daftar Isi

### Docs Tutorial (14 docs)

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
| 11 | [multi-profile-telegram](docs/11-multi-profile-telegram.md) | Multi-bot Telegram per workflow |
| 12 | [wsl-linux-commands](docs/12-wsl-linux-commands.md) | Cheat sheet Linux/WSL untuk pemula |
| 13 | [rtk-token-saver](docs/13-rtk-token-saver.md) | RTK install — hemat 60-90% token terminal |
| 14 | [prefix-cache-optimization](docs/14-prefix-cache-optimization.md) | KV Cache / Prefix Cache (teknik DeepSeek Reasonix) — hemat 40-90% |

---

## 🎯 25 Skills (Optimized for Weak Models)

> Semua skill udah di-refine dengan format decision tree + explicit instructions + contoh output.
> Model murah (DeepSeek V4 Flash, Qwen3.5) bisa follow tanpa ambiguitas.

### Meta / Workflow
| Skill | Fungsi |
|-------|--------|
| [`claude-superpowers`](examples/skills/claude-superpowers/SKILL.md) | Multi-capability workflow (thinking + artifacts + citations + vision + code exec) |
| [`anti-hallucination-review`](examples/skills/anti-hallucination-review/SKILL.md) | Checklist review sebelum jawab high-stakes — classify evidence, verify claims |
| [`weekly-review`](examples/skills/weekly-review/SKILL.md) | Ritual mingguan: review sesi, update memory/skill, track improvement |

### Research & Analysis
| Skill | Fungsi |
|-------|--------|
| [`research-citation`](examples/skills/research-citation/SKILL.md) | Cari jawaban faktual, wajib citation per klaim, "tidak ditemukan" kalau gak ada |
| [`deep-analysis`](examples/skills/deep-analysis/SKILL.md) | Multi-perspektif: decompose, trade-off matrix, conditional recommendation |
| [`study-buddy`](examples/skills/study-buddy/SKILL.md) | NotebookLM-style — HANYA jawab dari bahan yang lo kasih, quiz, flashcard |

### Development
| Skill | Fungsi |
|-------|--------|
| [`coding-mentor`](examples/skills/coding-mentor/SKILL.md) | Belajar coding dari NOL, analogi sehari-hari, 6 roadmap career path |
| [`web-development`](examples/skills/web-development/SKILL.md) | Fullstack web dev — stack-aware, verify version, plan before code |
| [`backend-proper`](examples/skills/backend-proper/SKILL.md) | Backend serius — security, idempotency, observability, DB schema |
| [`frontend-design`](examples/skills/frontend-design/SKILL.md) | UI anti-generic — bold aesthetic, no AI slop |
| [`ui-ux`](examples/skills/ui-ux/SKILL.md) | Review UI/UX — WCAG/Material/HIG grounded, priority labels |
| [`code-review`](examples/skills/code-review/SKILL.md) | Code review severity-based (🔴🟠🟡🟢) — security, bug, perf |
| [`devops-networking`](examples/skills/devops-networking/SKILL.md) | Server, Docker, CI/CD, nginx, SSL, firewall, monitoring |
| [`data-analysis`](examples/skills/data-analysis/SKILL.md) | Python pandas + matplotlib — load→clean→explore→visualize→insight |

### Daily Tasks / Media
| Skill | Fungsi |
|-------|--------|
| [`pdf-summarize`](examples/skills/pdf-summarize/SKILL.md) | Summary PDF terstruktur per section, referensi halaman |
| [`video-summary`](examples/skills/video-summary/SKILL.md) | yt-dlp + Whisper STT + LLM — summary video + timestamp |
| [`music-download`](examples/skills/music-download/SKILL.md) | YouTube (yt-dlp), Tidal/lossless FLAC (tiddl), Spotify (spotdl) |
| [`copywriting`](examples/skills/copywriting/SKILL.md) | Artikel, caption IG, email, landing page, thread X, script video |

### Finance
| Skill | Fungsi |
|-------|--------|
| [`financial-literacy`](examples/skills/financial-literacy/SKILL.md) | Edukator keuangan — budgeting, compound interest, produk keuangan |
| [`saham-syariah`](examples/skills/saham-syariah/SKILL.md) | Screening saham syariah IDX — DES OJK, fundamental |
| [`harga-emas`](examples/skills/harga-emas/SKILL.md) | Harga emas Antam HANYA dari logammulia.com |

### Religion
| Skill | Fungsi |
|-------|--------|
| [`islamic-study`](examples/skills/islamic-study/SKILL.md) | Cari ayat/hadits/fiqh dari sumber terpercaya — DILARANG ngarang dalil |

### Scraping
| Skill | Fungsi |
|-------|--------|
| [`web-scrape`](examples/skills/web-scrape/SKILL.md) | General scraping schema-first, fallback browser, JSON output |
| [`marketplace-scrape`](examples/skills/marketplace-scrape/SKILL.md) | Tokopedia, Shopee, Bukalapak, Lazada — harga, rating, seller |
| [`social-media-scrape`](examples/skills/social-media-scrape/SKILL.md) | Instagram (instaloader), X/Twitter (gallery-dl), Facebook, TikTok |

---

## ⚙️ File Config

| File | Fungsi |
|------|--------|
| [`examples/SOUL.md`](examples/SOUL.md) | Identity + anti-halu + formatting rules + execution rules |
| [`examples/AGENTS.md`](examples/AGENTS.md) | Project-level instructions + skill routing decision tree |
| [`examples/config.yaml`](examples/config.yaml) | Config default (BALANCED strategy) |
| [`examples/config-cheap.yaml`](examples/config-cheap.yaml) | Config CHEAP — DeepSeek Flash primary, ~$10-15/bln |
| [`examples/config-balanced.yaml`](examples/config-balanced.yaml) | Config BALANCED — Kimi K2.6 primary, ~$11-15/bln |
| [`examples/config-hybrid.yaml`](examples/config-hybrid.yaml) | Config HYBRID — K2.6 complex + Flash simple, ~$12-20/bln |
| [`examples/config-prefix-cache.yaml`](examples/config-prefix-cache.yaml) | Config PREFIX CACHE — BALANCED + KV cache optimized, ~$8-12/bln |
| [`examples/.env.example`](examples/.env.example) | Template env vars |

---

## 🚀 Quickstart

```bash
# 1. Install Hermes
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
source ~/.bashrc

# 2. Copy config — pilih strategy lo
cp examples/SOUL.md ~/.hermes/SOUL.md
cp examples/.env.example ~/.hermes/.env
# Edit .env → isi OPENCODE_GO_API_KEY, OPENROUTER_API_KEY, TELEGRAM_BOT_TOKEN

# Pilih salah satu config:
cp examples/config-cheap.yaml ~/.hermes/config.yaml       # budget ketat (~$10-15/bln)
cp examples/config-balanced.yaml ~/.hermes/config.yaml    # kualitas terbaik (~$11-15/bln)
cp examples/config-hybrid.yaml ~/.hermes/config.yaml      # balance (~$12-20/bln)

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

## 🧠 Tips: Model Murah agar Perform Optimal

Kalau lo pake DeepSeek V4 Flash / Qwen3.5 sebagai primary (cheap strategy):

1. **Skill format decision tree** — model murah follow `IF X → DO Y` lebih baik dari prose
2. **SOUL.md explicit** — tulis rule sebagai `DO NOT: ...` dan `DO: ...`, bukan narasi
3. **Contoh output di setiap skill** — model murah perlu "template" untuk tau format expected
4. **Reasoning effort = medium** — `high` bikin thinking token meledak di Flash
5. **RTK aktif** — terminal output model murah sering verbose, RTK compress ini
6. **Smart routing aggressive** — set threshold tinggi (200 char, 35 words) biar lebih banyak turn ke cheap model
7. **AGENTS.md di root project** — kasih context project supaya model gak perlu "guess" stack

---

## 📖 Urutan baca

**Kalau baru pertama kali**: 01 → 03 → 02 → 08 → sisanya sesuai kebutuhan

**Kalau udah familiar**: install (01), copy SOUL.md + config (03 + 09), Telegram (08), install skills, selesai.

**Jangan setup semuanya hari pertama** — pake dulu 1 minggu, baru tambah fitur.

---

## 📝 Prinsip di seluruh repo ini

1. **Honesty over confidence** — agent bilang "gak tau" daripada ngarang
2. **Source-first** — setiap klaim harus punya sumber yang bisa dicek
3. **Token efficiency** — progressive disclosure skills, smart routing, RTK, compression
4. **Never invent** — fakta, path, package name, API behavior → verify dulu
5. **Mistake correction** — tiap salah → koreksi note (apa, kenapa, rule ke depan)
6. **No filler** — langsung ke poin, gak ada "Tentu! Pertanyaan bagus!"
7. **Explicit over implicit** — instructions harus bisa diikuti model murah tanpa "infer"

---

## 🔗 Referensi

- [Hermes Agent docs](https://hermes-agent.nousresearch.com/docs/) — dokumentasi resmi
- [GitHub repo](https://github.com/NousResearch/hermes-agent) — source code
- [DeepSeek API docs](https://api-docs.deepseek.com/) — model pricing & reference
- [OpenCode Go](https://opencode.ai/go) — subscription models list
- [RTK (Rust Token Killer)](https://github.com/rtk-ai/rtk) — token optimizer
- [agentskills.io](https://agentskills.io/specification) — SKILL.md open standard

---

## Lisensi

Hermes Agent: MIT — © Nous Research.

Tutorial ini disusun dengan referensi ke dokumentasi resmi. Konten dirumus ulang untuk konteks lokal.
