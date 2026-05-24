# Hataf Hermes — Tutorial Lengkap Hermes Agent

Tutorial step-by-step buat ngebangun Hermes Agent (by [Nous Research](https://nousresearch.com)) jadi alat AI personal yang **jujur, anti-halu, hemat token, dan jalan 24/7 lewat Telegram**.

> Stack: Hermes Agent + OpenCode Go (subscription provider) + Kimi K2.6 (primary) + DeepSeek V4 Flash (smart routing) + Telegram gateway + cronjob + RTK token optimizer.

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
  ├── 25 Skills      (knowledge on-demand)
  └── Cronjob        (automation background)
        │
Smart Router
  ├── Turn kompleks  → Kimi K2.6 (primary, top quality)
  ├── Turn simpel    → DeepSeek V4 Flash (cheap routing)
  └── Fallback       → DeepSeek V4 Pro (kalau primary error)
        │
RTK (Rust Token Killer) → compress terminal output 60-90%
```

### Caveat penting

- **DeepSeek V4 Flash TIDAK ada di OpenCode Go bundle** — kalau mau pakai, butuh API key DeepSeek terpisah
- **Model di OpenCode Go** (per snippet opencode.ai/go, bisa berubah): GLM-5, Kimi K2.5/K2.6, MiMo-V2.5, Qwen3.5/3.6 Plus, MiniMax M2, DeepSeek V4 Flash/Pro
- **Vision**: model OpenCode Go belum dikonfirmasi punya vision — pakai OpenRouter Gemini Flash untuk auxiliary vision
- **SOUL.md / SKILL.md / MEMORY.md** = konvensi native Hermes, bukan nama yang lo karang

---

## 📚 Daftar Isi

### Docs Tutorial (13 docs)

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
| 11 | [multi-profile-telegram](docs/11-multi-profile-telegram.md) | Multi-bot Telegram per topik (coding/islam/finance) |
| 12 | [wsl-linux-commands](docs/12-wsl-linux-commands.md) | Cheat sheet Linux/WSL untuk pemula |
| 13 | [rtk-token-saver](docs/13-rtk-token-saver.md) | RTK install — hemat 60-90% token terminal |

---

## 🎯 25 Skills (tinggal copy-install)

### Meta / Workflow
| Skill | Fungsi |
|-------|--------|
| [`claude-superpowers`](examples/skills/claude-superpowers/SKILL.md) | Multi-capability workflow (thinking + artifacts + citations + vision + code exec) |
| [`anti-hallucination-review`](examples/skills/anti-hallucination-review/SKILL.md) | Checklist review sebelum jawab high-stakes — classify evidence, verify claims |
| [`weekly-review`](examples/skills/weekly-review/SKILL.md) | Ritual mingguan: review 20 sesi, update memory/skill, track improvement |

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
| [`frontend-design`](examples/skills/frontend-design/SKILL.md) | UI anti-generic (adapted from Anthropic) — bold aesthetic, no AI slop |
| [`ui-ux`](examples/skills/ui-ux/SKILL.md) | Review UI/UX — WCAG/Material/HIG grounded, priority labels |
| [`code-review`](examples/skills/code-review/SKILL.md) | Code review severity-based (🔴🟠🟡🟢) — security, bug, perf, maintainability |
| [`devops-networking`](examples/skills/devops-networking/SKILL.md) | Server, Docker, CI/CD, nginx, SSL, firewall, VPN, monitoring |
| [`data-analysis`](examples/skills/data-analysis/SKILL.md) | Python pandas + matplotlib + seaborn — load→clean→explore→visualize→insight |

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
| [`financial-literacy`](examples/skills/financial-literacy/SKILL.md) | Edukator keuangan — budgeting, compound interest calculator, produk keuangan |
| [`saham-syariah`](examples/skills/saham-syariah/SKILL.md) | Screening saham syariah IDX — DES OJK, fundamental, BUKAN financial advice |
| [`harga-emas`](examples/skills/harga-emas/SKILL.md) | Harga emas Antam HANYA dari logammulia.com (bypass web_search) |

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
| [`examples/SOUL.md`](examples/SOUL.md) | Identity + anti-halu + formatting rules + execution rules + mistake correction |
| [`examples/AGENTS.md`](examples/AGENTS.md) | Project-level instructions template |
| [`examples/config.yaml`](examples/config.yaml) | Full config — Balanced strategy (Kimi K2.6 primary, smart routing, compression) |
| [`examples/.env.example`](examples/.env.example) | Template env vars — OpenCode Go + OpenRouter + Telegram |

---

## 🚀 Quickstart (untuk yang udah tau Linux)

```bash
# 1. Install Hermes
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
source ~/.bashrc

# 2. Copy config
cp examples/SOUL.md ~/.hermes/SOUL.md
cp examples/config.yaml ~/.hermes/config.yaml
cp examples/.env.example ~/.hermes/.env
# Edit .env → isi OPENCODE_GO_API_KEY, OPENROUTER_API_KEY, TELEGRAM_BOT_TOKEN

# 3. Install semua skills
cp -r examples/skills/* ~/.hermes/skills/

# 4. Install RTK (hemat token)
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

## 📊 Model Strategy (Balanced, OpenCode Go only)

| Slot | Model | Alasan |
|------|-------|--------|
| 🧠 Primary | `kimi-k2.6` | Top quality (SWE-Bench 58.6%), near-Opus level |
| ⚡ Cheap routing | `deepseek-v4-flash` | 31,650 credits = paling irit, turn simpel ke sini |
| 🔄 Fallback | `deepseek-v4-pro` | Kalau K2.6 rate limit/error |
| 🗜️ Compression | `deepseek-v4-flash` | Summarize gak butuh pinter |
| 👁️ Vision | Gemini Flash (OpenRouter) | Satu-satunya yang vision-capable di setup ini |
| 🌐 Web extract | `qwen3.5-plus` | 10,200 credits, adequate |
| 🛡️ Approval | `deepseek-v4-flash` | Binary classify, gak perlu pinter |
| 🧒 Delegation | `qwen3.6-plus` | Balance quality/cost buat subagent |

**Total cost**: ~$11-13/bulan (OpenCode Go $10 + OpenRouter $1-3 untuk vision)

---

## 🏗️ Multi-Profile Telegram (Optional)

Buat 4 bot Telegram terpisah, tiap bot punya personality beda:

| Profile | Bot | Fokus | Primary model |
|---------|-----|-------|---------------|
| `coding` | @hataf_code_bot | Web dev, backend, coding mentor | Qwen3.6 Plus |
| `islamic` | @hataf_islam_bot | Islamic study, dalil verification | DeepSeek V4 Flash (high reasoning) |
| `finance` | @hataf_money_bot | Saham syariah, harga emas, financial literacy | DeepSeek V4 Flash |
| `default` | @hataf_main_bot | Semua skill, general purpose | Kimi K2.6 |

Setup detail: [docs/11-multi-profile-telegram.md](docs/11-multi-profile-telegram.md)

---

## 📖 Urutan baca

**Kalau baru pertama kali**: baca 01 → 03 → 02 → 08 → sisanya sesuai kebutuhan

**Kalau udah familiar dengan AI agent**: install dulu (01), lalu copy SOUL.md + config (03 + 09), setup Telegram (08), install skills semua sekaligus, selesai.

**Jangan setup semuanya hari pertama** — pake dulu 1 minggu, baru tambah fitur sesuai pain point yang muncul.

---

## 📝 Prinsip di seluruh repo ini

1. **Honesty over confidence** — agent bilang "gak tau" daripada ngarang
2. **Source-first** — setiap klaim harus punya sumber yang bisa dicek
3. **Token efficiency** — progressive disclosure skills, smart routing, RTK, compression
4. **Never invent** — fakta, path, package name, API behavior → verify dulu
5. **Mistake correction** — tiap salah → koreksi note (apa, kenapa, rule ke depan)
6. **No filler** — gak ada "Tentu! Pertanyaan bagus!" — langsung ke poin

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

Tutorial ini disusun dengan referensi ke dokumentasi resmi Hermes Agent, DeepSeek API docs, dan OpenCode. Konten dirumus ulang untuk konteks lokal — cek sumber asli untuk versi paling baru.
