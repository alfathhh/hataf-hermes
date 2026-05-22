# 01 — Instalasi & Quickstart

Tujuan: dari nol sampai bisa chat dengan Hermes di terminal lo, < 5 menit.

> Sumber utama: [Quickstart resmi Hermes](https://hermes-agent.nousresearch.com/docs/getting-started/quickstart) dan [README repo](https://github.com/NousResearch/hermes-agent).

---

## 1. Prasyarat

Yang lo butuh sebelum mulai:

- **OS**: Linux, macOS, atau WSL2 di Windows. (Native Windows ada tapi early beta — gw saranin pake WSL2 dulu.)
- **Internet** untuk download installer + nyambung ke LLM provider.
- **Akun di salah satu LLM provider** — minimum salah satu dari:
  - DeepSeek API key (kalau mau pake DeepSeek V4 Flash sebagai utama) — daftar di [platform.deepseek.com](https://platform.deepseek.com)
  - OpenCode Go subscription — daftar di [opencode.ai/go](https://opencode.ai/go)
  - Atau OpenRouter (sebagai fallback yang fleksibel) — [openrouter.ai](https://openrouter.ai)
- **Telegram bot token** (kalau mau gateway Telegram) — bisa diambil belakangan via [@BotFather](https://t.me/BotFather), detail di [docs/08-telegram-gateway.md](08-telegram-gateway.md).

> **Catatan**: gw rekomendasiin lo punya **minimal 2 provider** dari awal — satu primary, satu fallback. Kalau primary lagi rate-limit atau down, Hermes auto-switch (ini fitur fallback bawaan, dijelasin di doc 09).

---

## 2. Install Hermes (Linux / macOS / WSL2)

Buka terminal, jalankan:

```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

Installer ini bakal:

- Download `uv` (Python package manager)
- Bikin Python 3.11 virtualenv di `~/.hermes/`
- Install Node.js, ripgrep, ffmpeg (kalau belum ada)
- Bikin command `hermes` di `~/.local/bin/`

Setelah selesai, reload shell:

```bash
source ~/.bashrc      # bash
# atau
source ~/.zshrc       # zsh
```

Verifikasi terinstall:

```bash
hermes --version
hermes doctor          # diagnose, harusnya semua hijau
```

> **Kalau ada error**: jalankan `hermes doctor` — itu bakal kasih tau missing dependency. Yang paling sering: `ffmpeg` belum keinstall (perlu untuk voice mode), atau Python < 3.11.

---

## 3. Setup wizard (recommended buat pemula)

Hermes punya wizard interaktif yang mandu lo dari awal:

```bash
hermes setup
```

Wizard bakal nanya:

1. Provider mana yang mau lo pake → pilih provider lo (DeepSeek custom endpoint, OpenCode Go, atau OpenRouter)
2. API key → paste key lo
3. Default model → kasih nama model (misal `deepseek-v4-flash`)
4. Mau setup messaging gateway sekarang? → **skip dulu** (kita atur di doc 08)

Hasil setup ditulis ke:

- `~/.hermes/config.yaml` (settings non-rahasia)
- `~/.hermes/.env` (API keys, rahasia)
- `~/.hermes/SOUL.md` (default identity — kita timpa nanti di doc 03)

---

## 4. Chat pertama

```bash
hermes
```

Lo masuk ke TUI (Terminal UI). Coba:

```
> halo, perkenalkan dirimu
```

Kalau jalan: selamat, agent lo udah hidup.

Kalau error:

| Error | Penyebab umum | Fix |
|---|---|---|
| `No provider configured` | Belum set API key | `hermes config set <PROVIDER>_API_KEY ...` |
| `Connection refused` | Endpoint salah / firewall | Cek `~/.hermes/.env`, ping endpoint manual |
| `Model not found` | Nama model typo | `hermes model` (interaktif, dia listing model yang available) |
| Stuck di "thinking" lama banget | Model lambat / context kepenuhan | `Ctrl+C` cancel, coba `/compress` |

Detail troubleshooting: [hermes-agent.nousresearch.com/docs/reference/faq](https://hermes-agent.nousresearch.com/docs/reference/faq).

---

## 5. Command dasar yang harus lo hafal

| Command | Fungsi |
|---|---|
| `hermes` | Buka TUI (chat interaktif) |
| `hermes -q "pertanyaan"` | One-shot query (langsung jawab, gak buka TUI) |
| `hermes model` | Ganti provider/model interaktif |
| `hermes config edit` | Buka `config.yaml` di editor lo |
| `hermes doctor` | Diagnose masalah |
| `hermes update` | Update Hermes ke versi terbaru |
| `hermes gateway` | Jalanin messaging gateway (Telegram, dll) |

Slash command (di dalem TUI):

| Slash | Fungsi |
|---|---|
| `/new` atau `/reset` | Mulai conversation baru |
| `/model` | Ganti model di tengah session |
| `/compress` | Kompres history kalau context kepenuhan |
| `/usage` | Liat token & biaya estimated |
| `/personality concise` | Switch mode bicara |
| `/skills` | List skills tersedia |
| `Ctrl+C` | Interrupt agent yang lagi mikir |

---

## 6. Struktur direktori `~/.hermes/`

Setelah install, ini yang ada di home Hermes lo:

```
~/.hermes/
├── config.yaml         # Settings (model, terminal, compression, dst)
├── .env                # API keys, secrets
├── auth.json           # OAuth credentials (kalau pake OAuth provider)
├── SOUL.md             # Identitas agent (slot #1 system prompt) ← KUNCI
├── memories/
│   ├── MEMORY.md       # Agent's notes (~800 token)
│   └── USER.md         # User profile (~500 token)
├── skills/             # Folder skills, satu folder = satu skill
├── cron/
│   ├── jobs.json       # Cronjob storage
│   └── output/         # Hasil cronjob
├── sessions/           # Conversation history per platform
└── logs/
    ├── errors.log      # Error log (secrets auto-redacted)
    └── gateway.log     # Gateway log
```

> **Pastiin**: jangan share folder `~/.hermes/` ke siapa pun — `.env` di situ ada API key.

---

## 7. Backup minimal (sebelum lo otak-atik banyak)

```bash
# Backup config + soul + memory + skills
tar czf hermes-backup-$(date +%F).tar.gz \
  ~/.hermes/config.yaml \
  ~/.hermes/.env \
  ~/.hermes/SOUL.md \
  ~/.hermes/memories/ \
  ~/.hermes/skills/

# Simpan di tempat aman (jangan commit ke git, ada secrets)
```

Restore tinggal `tar xzf` ke folder yang sama.

---

## 8. Lanjut ke mana

Sekarang Hermes lo udah jalan tapi **belum di-tune**. Default Hermes itu pinter, tapi belum diajarin **gak halu** dan belum tau cara hemat token.

Lanjut:

→ [02 — Providers & Models](02-providers-dan-models.md): set DeepSeek + OpenCode Go bareng, plus rekomendasi model per use case.

Atau loncat ke:

→ [03 — SOUL.md](03-soul-md.md): bikin agent lo jujur dan anti-halu (gw saranin baca ini cepet).

---

## Catatan jujur dari gw

- `hermes setup` ini **suka berubah flow-nya** antar versi. Kalau wizard kasih opsi yang beda dari yang gw tulis di sini, **percaya wizard**, bukan tutorial ini.
- Kalau lo pake VPS murah ($5/bulan kayak DigitalOcean basic), Hermes bisa jalan tapi kalau RAM cuma 512 MB bakal sering OOM saat compression. Minimum yang nyaman: 1 GB RAM, dan SSD (HDD parah lambat). Saya belum benchmark sendiri — angka ini perkiraan dari pengalaman umum running Python apps di VPS, bukan benchmark resmi Hermes.
