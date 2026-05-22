# Hataf Hermes — Tutorial Lengkap Hermes Agent

Tutorial step-by-step buat ngebangun Hermes Agent (by [Nous Research](https://nousresearch.com)) jadi alat AI personal yang **jujur, anti-halu, hemat token, dan jalan 24/7 lewat Telegram**.

> Stack: Hermes Agent + OpenCode Go (subscription provider) + DeepSeek V4 Flash (primary) + Telegram gateway + cronjob scraping.

---

## Sebelum mulai — baca yang ini dulu

### 1. Hermes Agent itu apa, dan apa yang BUKAN

**Hermes Agent itu:**

- Framework agent open-source dari Nous Research. Repo: [github.com/NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent).
- Jalan di **terminal Linux/macOS/WSL2** (Windows native masih early beta — lo akan dapet rough edges).
- Bisa nyambung ke **berbagai model provider** — Hermes nggak ngunci lo ke satu model.
- Punya **memory persisten**, **skills system**, **cronjob**, **messaging gateway** (Telegram, Discord, dll).
- Sumber utama yang gw pake buat dokumentasi ini: [hermes-agent.nousresearch.com/docs](https://hermes-agent.nousresearch.com/docs/).

**Hermes Agent BUKAN:**

- Bukan model. Hermes nggak punya bobot (weights) sendiri — dia perlu nyambung ke LLM provider (OpenCode Go, DeepSeek, OpenRouter, Anthropic, dst).
- Bukan SaaS. Lo install dan jalanin sendiri.

### 2. Arsitektur akhir yang lo bangun

```
┌────────────────────────────────────────────────────────────┐
│  Lo (Telegram di HP)                                        │
└────────────────────────┬───────────────────────────────────┘
                         │ chat
┌────────────────────────▼───────────────────────────────────┐
│  Telegram Bot (BotFather token)                             │
└────────────────────────┬───────────────────────────────────┘
                         │
┌────────────────────────▼───────────────────────────────────┐
│  Hermes Gateway daemon (jalan di VPS/lokal lo)              │
│  ─ memuat SOUL.md (identity/anti-halu)                      │
│  ─ memuat MEMORY.md, USER.md (memory persisten)             │
│  ─ memuat AGENTS.md (project rules)                         │
│  ─ punya skills (research, pdf, video, scraping, dst)       │
│  ─ jalanin cronjob terjadwal                                │
└──┬───────────────────────────────────────────┬──────────────┘
   │                                            │
   │ (utama, kompleks)                          │ (cepat, ringan)
   │                                            │
┌──▼──────────────────────┐         ┌───────────▼──────────────┐
│  DeepSeek V4 Flash      │         │  OpenCode Go bundle      │
│  (custom endpoint OAI)  │         │  GLM-5.1 / Kimi K2.6 /   │
│  api.deepseek.com/v1    │         │  MiniMax M2 / Qwen3.6 +  │
│  thinking mode optional │         │  (untuk coding harian)   │
└─────────────────────────┘         └──────────────────────────┘
```

### 3. Caveat penting (gw harus jujur)

Beberapa hal di tutorial ini **mungkin sudah berubah** ketika lo baca, jadi cek dokumentasi resminya:

- **Model di OpenCode Go bundle**: per snippet [opencode.ai/go](https://opencode.ai/go) yang gw cek, isinya GLM-5.1, GLM-5, Kimi K2.5, K2.6, MiMo-V2.5-Pro, MiMo-V2.5, Qwen3.5 Plus, Qwen3.6 Plus, MiniMax M2. **DeepSeek V4 Flash TIDAK ada di bundle ini.** Kalau lo mau pakai DeepSeek V4 Flash sebagai model utama, lo butuh API key DeepSeek terpisah ($5 deposit minimum kalau setoran pertama, info bisa berubah).
- **Pricing DeepSeek V4 Flash**: per beberapa sumber ([apidog](http://apidog.com/blog/deepseek-v4-api-pricing), [codersera](https://codersera.com/blog/deepseek-v4-flash-deep-dive)), $0.14/M input dan $0.28/M output. Cek halaman resmi DeepSeek sebelum jadiin patokan budget.
- **Vision capability** (analisis gambar): gw **belum bisa konfirmasi** mana model di OpenCode Go bundle yang punya vision. Hermes default-nya pake **Gemini Flash via OpenRouter** untuk auxiliary vision — dan ini gampang di-swap. Jadi vision tasks (PDF dengan gambar, screenshot, foto) sebaiknya pakai jalur auxiliary terpisah, bukan model utama. Detailnya di [docs/05-tools-dan-mcp.md](docs/05-tools-dan-mcp.md).
- **SOUL.md / SKILL.md / MEMORY.md**: ini **konvensi native Hermes**, bukan nama yang lo karang. `tools.md` bukan native — gw petakan ke `~/.hermes/config.yaml` + MCP config.
- **`hataf-hermes` (nama repo lo)**: gw asumsiin "hataf" itu prefix personal lo. Nggak pengaruh ke setup.

---

## Daftar isi

1. [Instalasi & Quickstart](docs/01-instalasi-quickstart.md) — install Hermes, chat pertama dalam < 5 menit
2. [Providers & Model Recommendation](docs/02-providers-dan-models.md) — setup OpenCode Go + DeepSeek + tabel rekomendasi per use case
3. [SOUL.md — Identitas Anti-Halu](docs/03-soul-md.md) — file paling penting, agent gak ngarang
4. [Skills System](docs/04-skills.md) — knowledge on-demand, hemat token, 4 contoh skill custom
5. [Tools, MCP, dan Vision](docs/05-tools-dan-mcp.md) — file tools, web search, vision auxiliary
6. [Memory & USER Profile](docs/06-memory.md) — persistensi lintas sesi tanpa boros token
7. [Cronjob Scraping](docs/07-cron-scraping.md) — scheduled tasks tanpa lo pelototi
8. [Telegram Gateway](docs/08-telegram-gateway.md) — bot Telegram lo, lengkap
9. [Optimasi Token (HEMAT BANGET)](docs/09-optimasi-token.md) — semua knob hemat biaya
10. [Mengajarkan Hermes (Feedback Loop)](docs/10-mengajarkan-hermes.md) — bikin agent makin pinter dari waktu ke waktu

## File contoh siap pakai

Di folder [`examples/`](examples/) ada file template yang tinggal lo copy:

- [`examples/SOUL.md`](examples/SOUL.md) — identitas anti-halu (versi panjang)
- [`examples/AGENTS.md`](examples/AGENTS.md) — project-level instruction
- [`examples/config.yaml`](examples/config.yaml) — `~/.hermes/config.yaml` lengkap
- [`examples/.env.example`](examples/.env.example) — template environment variables
- [`examples/skills/research-citation/SKILL.md`](examples/skills/research-citation/SKILL.md)
- [`examples/skills/pdf-summarize/SKILL.md`](examples/skills/pdf-summarize/SKILL.md)
- [`examples/skills/video-summary/SKILL.md`](examples/skills/video-summary/SKILL.md)
- [`examples/skills/web-scrape/SKILL.md`](examples/skills/web-scrape/SKILL.md)

---

## Urutan baca yang gw rekomendasikan

Kalau lo **bener-bener nol**, baca berurutan dari `01` sampai `10`. Jangan loncat. Setiap file naik level.

Kalau lo udah pernah pake AI agent lain (Claude Code, Codex, OpenClaw, Cursor):

1. Baca `01-instalasi-quickstart` (dapet TUI jalan)
2. Loncat ke `03-soul-md` (ini yang bikin agent lo jujur)
3. Loncat ke `02-providers-dan-models` (set DeepSeek + OpenCode Go)
4. Loncat ke `08-telegram-gateway` (kalau emang prioritas Telegram)
5. Lainnya sesuai kebutuhan

---

## Prinsip yang dipegang di seluruh dokumen ini

Ini bukan template umum. Hermes lo bakal di-tuned khusus dengan prinsip:

1. **Honesty over confidence** — agent harus bilang "gak tau" daripada ngarang.
2. **Source-first** — setiap klaim harus punya sumber yang bisa dicek.
3. **Token efficiency** — pakai progressive disclosure (skills), smart routing (cheap model untuk tugas ringan), compression aktif.
4. **Provenance untuk angka** — angka, statistik, version number selalu ditandain "perlu cek ulang" kalau bukan dari sumber primer.
5. **No filler** — jangan ada "Tentu! Pertanyaan bagus!"-an.

Aturan-aturan ini di-encode keras di [`examples/SOUL.md`](examples/SOUL.md). Itu yang bikin Hermes lo beda sama default.

---

## Lisensi & atribusi

Hermes Agent: MIT — © Nous Research.

Tutorial ini disusun dengan referensi ke dokumentasi resmi Hermes Agent ([hermes-agent.nousresearch.com](https://hermes-agent.nousresearch.com)), repo [github.com/NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent), DeepSeek API docs ([api-docs.deepseek.com](https://api-docs.deepseek.com)), dan OpenCode ([opencode.ai](https://opencode.ai)). Konten dirumus ulang untuk konteks lokal — cek sumber asli untuk versi paling baru.
