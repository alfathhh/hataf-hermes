# 11 — Multi-Profile Telegram (1 Server, Banyak Bot, Per Workflow)

Tujuan: setup Hermes dengan banyak profile — tiap profile = 1 workflow/domain. Bot Telegram terpisah, skill set fokus, personality konsisten. Kayak punya "tim AI" yang masing-masing spesialis.

> Sumber: [Hermes Profiles docs](https://hermes-agent.nousresearch.com/docs/user-guide/profiles), [Profile Distributions](https://hermes-agent.nousresearch.com/docs/user-guide/profile-distributions), [Telegram Gateway](https://hermes-agent.nousresearch.com/docs/user-guide/messaging/telegram).

---

## 1. Konsep: kenapa organize per WORKFLOW

| ❌ Masalah (1 bot untuk semua) | ✅ Solusi (multi-profile per workflow) |
|---|---|
| Semua topik di 1 chat, context noisy | Tiap workflow punya context clean |
| Semua 25 skill loaded tiap turn | Tiap bot cuma load 3-7 skill relevan |
| 1 personality untuk semua topik | Tiap profile punya personality & skill sendiri |
| Token boros (skill list besar) | Token hemat (skill list kecil per profile) |
| Cron output campur di 1 grup | Cron deliver ke grup yang tepat |

### Workflow categories (rekomendasi):

```
1. META / GENERAL      → semua skill, general purpose, DM personal
2. DEVELOPMENT         → coding, devops, review, data analysis
3. RESEARCH & ANALYSIS → deep analysis, study buddy, pdf, video summary
4. ISLAMIC STUDY       → islamic-study, research-citation (HIGH anti-halu)
5. FINANCE & MONITOR   → saham, emas, financial literacy, marketplace scrape
6. CONTENT & MEDIA     → copywriting, music download, social media scrape
```

Lo gak harus pakai SEMUA 6. Mulai dari 2-3, expand kalau perlu.

---

## 2. Arsitektur akhir

```
┌───────────────────────────────────────────────────────────────────────┐
│  VPS lo (1 server)                                                     │
│                                                                        │
│  hermes --profile main      → gateway → @hataf_main_bot     (DM)      │
│  hermes --profile code      → gateway → @hataf_code_bot     (Coding)  │
│  hermes --profile research  → gateway → @hataf_research_bot (Research)│
│  hermes --profile islam     → gateway → @hataf_islam_bot    (Islam)   │
│  hermes --profile finance   → gateway → @hataf_money_bot    (Finance) │
│  hermes --profile media     → gateway → @hataf_media_bot    (Media)   │
└───────────────────────────────────────────────────────────────────────┘
         │            │              │            │           │          │
         ▼            ▼              ▼            ▼           ▼          ▼
   ┌──────────┐ ┌──────────┐ ┌───────────┐ ┌──────────┐ ┌────────┐ ┌────────┐
   │ DM       │ │ Grup     │ │ Grup      │ │ Grup     │ │ Grup   │ │ Grup   │
   │ Personal │ │ Coding   │ │ Research  │ │ Islamic  │ │ Finance│ │ Media  │
   └──────────┘ └──────────┘ └───────────┘ └──────────┘ └────────┘ └────────┘
```

---

## 3. Step-by-step setup

### Step 1: Bikin bot di BotFather (1 per profile)

Buka [@BotFather](https://t.me/BotFather), bikin bot sesuai profile yang mau disetup:

```
/newbot → "Hataf Main"     → username: hataf_main_bot     → save token
/newbot → "Hataf Code"     → username: hataf_code_bot     → save token
/newbot → "Hataf Research" → username: hataf_research_bot → save token
/newbot → "Hataf Islam"    → username: hataf_islam_bot    → save token
/newbot → "Hataf Finance"  → username: hataf_money_bot    → save token
/newbot → "Hataf Media"    → username: hataf_media_bot    → save token
```

Lo sekarang punya 6 token. Simpan baik-baik di `.env` masing-masing profile.

---

### Step 2: Bikin profile di Hermes

```bash
# Bikin semua profile
hermes profile create main
hermes profile create coding
hermes profile create research
hermes profile create islamic
hermes profile create finance
hermes profile create media
```

Ini bikin struktur:

```
~/.hermes/profiles/
├── main/
│   ├── config.yaml
│   ├── .env
│   ├── SOUL.md
│   ├── memories/
│   └── skills/         ← semua 25 skill
├── coding/
│   ├── config.yaml
│   ├── .env
│   ├── SOUL.md
│   ├── memories/
│   └── skills/         ← 6 skill coding
├── research/
│   ├── config.yaml
│   ├── .env
│   ├── SOUL.md
│   ├── memories/
│   └── skills/         ← 5 skill research
├── islamic/
│   ├── config.yaml
│   ├── .env
│   ├── SOUL.md
│   ├── memories/
│   └── skills/         ← 2 skill
├── finance/
│   ├── config.yaml
│   ├── .env
│   ├── SOUL.md
│   ├── memories/
│   └── skills/         ← 5 skill finance
└── media/
    ├── config.yaml
    ├── .env
    ├── SOUL.md
    ├── memories/
    └── skills/         ← 5 skill media
```

> ⚠️ **Catatan jujur**: gw belum 100% confirm exact folder structure per profile dari dokumentasi. Yang gw tau pasti: profile punya config override sendiri. Cek `hermes profile --help` untuk detail exact. Kalau structure beda, adapt panduan ini.

---

### Step 3: Setup tiap profile

#### Profile: `coding`

**`~/.hermes/profiles/coding/.env`**:
```bash
TELEGRAM_BOT_TOKEN=<token hataf_code_bot>
TELEGRAM_ALLOWED_USERS=<user_id_lo>
OPENCODE_GO_API_KEY=<opencode_go_key>
OPENROUTER_API_KEY=<openrouter_key>
```

**`~/.hermes/profiles/coding/config.yaml`**:
```yaml
model:
  provider: opencode-go
  default: kimi-k2.6
  api_mode: chat_completions

agent:
  reasoning_effort: high
  max_turns: 90

smart_model_routing:
  enabled: true
  max_simple_chars: 160
  max_simple_words: 28
  cheap_model:
    provider: opencode-go
    model: deepseek-v4-flash

fallback_model:
  provider: opencode-go
  model: deepseek-v4-pro

auxiliary:
  vision:
    provider: openrouter
    model: google/gemini-2.5-flash
    timeout: 30
  web_extract:
    provider: opencode-go
    model: qwen3.5-plus
  compression:
    provider: opencode-go
    model: deepseek-v4-flash

compression:
  enabled: true
  threshold: 0.55
  target_ratio: 0.2
  protect_last_n: 20

memory:
  memory_enabled: true
  user_profile_enabled: true

display:
  show_cost: true

streaming:
  enabled: true
```

**`~/.hermes/profiles/coding/SOUL.md`**:
```markdown
Lo adalah temen belajar coding. Bahasa lo-gw, casual.

ATURAN:
- Selalu pakai analogi sehari-hari untuk konsep baru
- Gak boleh skip fundamental
- Kalau user belum ngerti prerequisite, bilang dulu
- Setiap penjelasan HARUS ada contoh code yang bisa diketik
- Jangan pakai jargon tanpa jelasin dulu

FORMAT OUTPUT:
- 📌 TL;DR di awal
- 💡 Analogi
- 🔧 Code contoh (dengan label bahasa)
- 🏋️ Exercise kecil
- ⚠️ Pitfall yang sering terjadi
- 🔗 Sumber (selalu di akhir)
```

**Install skill yang relevan**:
```bash
cp -r ~/.hermes/skills/coding-mentor ~/.hermes/profiles/coding/skills/
cp -r ~/.hermes/skills/web-development ~/.hermes/profiles/coding/skills/
cp -r ~/.hermes/skills/backend-proper ~/.hermes/profiles/coding/skills/
cp -r ~/.hermes/skills/frontend-design ~/.hermes/profiles/coding/skills/
cp -r ~/.hermes/skills/ui-ux ~/.hermes/profiles/coding/skills/
cp -r ~/.hermes/skills/code-review ~/.hermes/profiles/coding/skills/
```

---

#### Profile: `islamic`

**`~/.hermes/profiles/islamic/config.yaml`**:
```yaml
model:
  provider: opencode-go
  default: deepseek-v4-flash
  api_mode: chat_completions

agent:
  reasoning_effort: high
  max_turns: 90

smart_model_routing:
  enabled: false

fallback_model:
  provider: opencode-go
  model: deepseek-v4-pro

auxiliary:
  vision:
    provider: openrouter
    model: google/gemini-2.5-flash
    timeout: 30
  web_extract:
    provider: opencode-go
    model: qwen3.5-plus
  compression:
    provider: opencode-go
    model: qwen3.5-plus

compression:
  enabled: true
  threshold: 0.60
  target_ratio: 0.2
  protect_last_n: 20

memory:
  memory_enabled: true
  user_profile_enabled: true

streaming:
  enabled: true
```

**Install skill**:
```bash
cp -r ~/.hermes/skills/islamic-study ~/.hermes/profiles/islamic/skills/
cp -r ~/.hermes/skills/research-citation ~/.hermes/profiles/islamic/skills/
```

**`~/.hermes/profiles/islamic/SOUL.md`**:
```markdown
(Global SOUL.md berlaku. Section ini adalah tambahan untuk profile islamic.)

## FOKUS DOMAIN: Ilmu Islam

Persona: asisten pencarian ilmu — teliti, multi-mazhab, anti-halu dalil.

LARANGAN TAMBAHAN (domain-specific, di luar golden rules global):
- DILARANG mengarang ayat Al-Quran atau hadits (bahkan satu kata pun)
- DILARANG bilang "hukumnya halal/haram" tanpa rujukan ulama spesifik
- DILARANG tafsir sendiri — HARUS rujuk mufassir (Ibnu Katsir, Kemenag, dll)
- DILARANG keluarkan fatwa
- DILARANG tentukan derajat hadits tanpa verify sumber

KALAU GAK KETEMU SUMBER → bilang "gak ketemu". TITIK. Jangan isi dengan opini.

SUMBER YANG VALID:
- Al-Quran: quran.kemenag.go.id, quran.com, tanzil.net
- Hadits: sunnah.com, dorar.net, hadits.id
- Tafsir: tafsirweb.com, Ibnu Katsir (via quran.com)
- Fiqh: MUI (mui.or.id), NU Online, Muhammadiyah, islamqa.info, islamweb.net

HANDLING KHILAFIYAH:
- WAJIB present semua pendapat utama (multi-mazhab), bukan pilih satu
- Label setiap pendapat dengan mazhab/ulama

DISCLAIMER WAJIB di akhir setiap output:
"Untuk fatwa personal, konsultasi ustadz yang lo percaya."
```

---

#### Profile: `finance`

**`~/.hermes/profiles/finance/config.yaml`**:
```yaml
model:
  provider: opencode-go
  default: deepseek-v4-flash
  api_mode: chat_completions

agent:
  reasoning_effort: medium
  max_turns: 90

smart_model_routing:
  enabled: true
  max_simple_chars: 160
  max_simple_words: 28
  cheap_model:
    provider: opencode-go
    model: qwen3.5-plus

fallback_model:
  provider: opencode-go
  model: deepseek-v4-pro

auxiliary:
  vision:
    provider: openrouter
    model: google/gemini-2.5-flash
    timeout: 30
  web_extract:
    provider: opencode-go
    model: qwen3.5-plus
  compression:
    provider: opencode-go
    model: qwen3.5-plus

compression:
  enabled: true
  threshold: 0.50
  target_ratio: 0.2
  protect_last_n: 20

streaming:
  enabled: true
```

**Install skill**:
```bash
cp -r ~/.hermes/skills/financial-literacy ~/.hermes/profiles/finance/skills/
cp -r ~/.hermes/skills/saham-syariah ~/.hermes/profiles/finance/skills/
cp -r ~/.hermes/skills/harga-emas ~/.hermes/profiles/finance/skills/
cp -r ~/.hermes/skills/web-scrape ~/.hermes/profiles/finance/skills/
cp -r ~/.hermes/skills/research-citation ~/.hermes/profiles/finance/skills/
```

**`~/.hermes/profiles/finance/SOUL.md`**:
```markdown
(Global SOUL.md berlaku. Section ini adalah tambahan untuk profile finance.)

## FOKUS DOMAIN: Keuangan & Investasi

Persona: edukator keuangan — kasih framework dan hitungan, BUKAN nasihat investasi.

BATASAN TEGAS (domain-specific):
- BUKAN financial advisor — TIDAK bilang "invest di X" atau "jual Y sekarang"
- TIDAK memberikan prediksi harga atau return
- Untuk keputusan besar (>10% net worth) → arahkan ke CFP berlisensi
- Angka historis → flag bahwa past performance ≠ future results

DISCLAIMER WAJIB di setiap output yang menyebut produk/instrumen investasi:
"⚠️ Bukan nasihat investasi. Data screening otomatis. DYOR."
```

---

#### Profile: `research`

**`~/.hermes/profiles/research/config.yaml`**:
```yaml
model:
  provider: opencode-go
  default: deepseek-v4-flash
  api_mode: chat_completions

agent:
  reasoning_effort: high
  max_turns: 90

smart_model_routing:
  enabled: false

fallback_model:
  provider: opencode-go
  model: kimi-k2.6

auxiliary:
  vision:
    provider: openrouter
    model: google/gemini-2.5-flash
    timeout: 30
  web_extract:
    provider: opencode-go
    model: qwen3.5-plus
  compression:
    provider: opencode-go
    model: qwen3.5-plus

compression:
  enabled: true
  threshold: 0.65
  target_ratio: 0.2
  protect_last_n: 20

memory:
  memory_enabled: true
  user_profile_enabled: true

streaming:
  enabled: true
```

**Install skill**:
```bash
cp -r ~/.hermes/skills/deep-analysis ~/.hermes/profiles/research/skills/
cp -r ~/.hermes/skills/research-citation ~/.hermes/profiles/research/skills/
cp -r ~/.hermes/skills/study-buddy ~/.hermes/profiles/research/skills/
cp -r ~/.hermes/skills/pdf-summarize ~/.hermes/profiles/research/skills/
cp -r ~/.hermes/skills/video-summary ~/.hermes/profiles/research/skills/
cp -r ~/.hermes/skills/data-analysis ~/.hermes/profiles/research/skills/
```

**`~/.hermes/profiles/research/SOUL.md`**:
```markdown
(Global SOUL.md berlaku. Section ini adalah tambahan untuk profile research.)

## FOKUS DOMAIN: Research & Analisis

Persona: research assistant — decompose masalah, citation-first, no fluff.

TAMBAHAN ATURAN:
- Setiap klaim factual HARUS ada URL yang bisa dicek — bukan cuma "menurut X"
- JANGAN jawab dari memori untuk fakta time-sensitive (harga, regulasi, versi)
- Correlation ≠ causation — sebut ini kalau ada data yang di-present
- Kalau pertanyaan butuh expert domain (dokter, pengacara, akuntan) → arahkan ke expert

CARA KERJA WAJIB:
1. Decompose pertanyaan jadi sub-questions
2. Search per sub-question dengan web tools
3. Synthesize dengan attribution per klaim
4. List eksplisit "yang gw belum temukan" — jangan sembunyikan gap

FORMAT TAMBAHAN:
- ❓ Section "What I don't know / gaps" wajib ada di output research panjang
- ⚖️ Wajib kasih multiple perspectives untuk topik yang contested
```

---

#### Profile: `media`

**`~/.hermes/profiles/media/config.yaml`**:
```yaml
model:
  provider: opencode-go
  default: qwen3.6-plus
  api_mode: chat_completions

agent:
  reasoning_effort: low
  max_turns: 90

smart_model_routing:
  enabled: true
  max_simple_chars: 200
  max_simple_words: 40
  cheap_model:
    provider: opencode-go
    model: qwen3.5-plus

fallback_model:
  provider: opencode-go
  model: deepseek-v4-flash

auxiliary:
  vision:
    provider: openrouter
    model: google/gemini-2.5-flash
    timeout: 30
  web_extract:
    provider: opencode-go
    model: qwen3.5-plus
  compression:
    provider: opencode-go
    model: qwen3.5-plus

compression:
  enabled: true
  threshold: 0.45
  target_ratio: 0.2
  protect_last_n: 20

memory:
  memory_enabled: true
  user_profile_enabled: true

streaming:
  enabled: true
```

**Install skill**:
```bash
cp -r ~/.hermes/skills/copywriting ~/.hermes/profiles/media/skills/
cp -r ~/.hermes/skills/music-download ~/.hermes/profiles/media/skills/
cp -r ~/.hermes/skills/social-media-scrape ~/.hermes/profiles/media/skills/
cp -r ~/.hermes/skills/video-summary ~/.hermes/profiles/media/skills/
cp -r ~/.hermes/skills/web-scrape ~/.hermes/profiles/media/skills/
```

**`~/.hermes/profiles/media/SOUL.md`**:
```markdown
(Global SOUL.md berlaku. Section ini adalah tambahan untuk profile media.)

## FOKUS DOMAIN: Content & Media

Persona: content assistant — output siap pakai, langsung eksekusi, no filler.

TAMBAHAN ATURAN:
- Output = copy yang siap publish, bukan draft perlu revisi besar
- Kalau context sudah jelas → langsung eksekusi tanpa nanya dulu
- JANGAN ngarang statistik/fakta untuk dipakai dalam konten
- JANGAN janjiin hasil ("pasti viral", "pasti naik engagement")

GAYA NULIS yang diharapkan:
- Specific > generic ("hemat 2 jam/hari" bukan "hemat waktu")
- Hook kuat di kalimat pertama
- CTA jelas dan cuma 1 per piece
- DILARANG opener AI slop: "Di era digital...", "Sebagai makhluk sosial..."
```

---

#### Profile: `main` (default — General Purpose)

**`~/.hermes/profiles/main/config.yaml`**:
```yaml
model:
  provider: opencode-go
  default: kimi-k2.6
  api_mode: chat_completions

agent:
  reasoning_effort: ""
  max_turns: 90

smart_model_routing:
  enabled: true
  max_simple_chars: 160
  max_simple_words: 28
  cheap_model:
    provider: opencode-go
    model: deepseek-v4-flash

fallback_model:
  provider: opencode-go
  model: deepseek-v4-pro

auxiliary:
  vision:
    provider: openrouter
    model: google/gemini-2.5-flash
    timeout: 30
  web_extract:
    provider: opencode-go
    model: qwen3.5-plus
  approval:
    provider: opencode-go
    model: deepseek-v4-flash
  compression:
    provider: opencode-go
    model: deepseek-v4-flash

compression:
  enabled: true
  threshold: 0.50
  target_ratio: 0.2
  protect_last_n: 20

delegation:
  provider: opencode-go
  model: qwen3.6-plus

memory:
  memory_enabled: true
  user_profile_enabled: true
  memory_char_limit: 2200
  user_char_limit: 1375

streaming:
  enabled: true

display:
  show_cost: true
```

**Install skill** (semua 25):
```bash
cp -r ~/.hermes/skills/* ~/.hermes/profiles/main/skills/
```

**`~/.hermes/profiles/main/SOUL.md`**:
```markdown
(Global SOUL.md berlaku. Section ini adalah tambahan untuk profile main.)

## FOKUS DOMAIN: General Purpose

Profile ini adalah profile utama — semua 25 skill aktif, tidak ada batasan domain.
Global SOUL.md sudah mencakup semua yang dibutuhkan untuk use case general.

Tidak ada tambahan aturan khusus untuk profile ini.
```

---

### Step 4: Setup gateway per profile

```bash
hermes --profile main     gateway setup
hermes --profile coding   gateway setup
hermes --profile research gateway setup
hermes --profile islamic  gateway setup
hermes --profile finance  gateway setup
hermes --profile media    gateway setup
```

---

### Step 5: Jalanin semua gateway

#### Opsi A: Manual (testing)

```bash
hermes --profile main     gateway
hermes --profile coding   gateway
hermes --profile research gateway
hermes --profile islamic  gateway
hermes --profile finance  gateway
hermes --profile media    gateway
```

#### Opsi B: Systemd services (production, recommended)

```bash
for profile in main coding research islamic finance media; do
  sudo tee /etc/systemd/system/hermes-${profile}.service > /dev/null << EOF
[Unit]
Description=Hermes Agent - ${profile} Profile
After=network.target

[Service]
Type=simple
User=$(whoami)
ExecStart=$(which hermes) --profile ${profile} gateway
Restart=always
RestartSec=10
Environment=HOME=$HOME

[Install]
WantedBy=multi-user.target
EOF
done

sudo systemctl daemon-reload
sudo systemctl enable hermes-main hermes-coding hermes-research hermes-islamic hermes-finance hermes-media
sudo systemctl start  hermes-main hermes-coding hermes-research hermes-islamic hermes-finance hermes-media
```

Cek status semua:
```bash
for profile in main coding research islamic finance media; do
  echo "=== $profile ===" && sudo systemctl status hermes-${profile} --no-pager -l | head -5
done
```

---

### Step 6: Bikin grup Telegram per profile

```
Grup: "🏠 Hataf Main"      → add @hataf_main_bot     → DM personal + semua topik
Grup: "💻 Coding Hataf"    → add @hataf_code_bot     → coding, devops, review
Grup: "🔬 Research Hataf"  → add @hataf_research_bot → analisis, paper, PDF
Grup: "🕌 Islamic Study"   → add @hataf_islam_bot    → dalil, tafsir, fiqh
Grup: "💰 Finance Hataf"   → add @hataf_money_bot    → saham, emas, literasi finansial
Grup: "🎨 Media Hataf"     → add @hataf_media_bot    → copywriting, download, scrape
```

---

## 4. Cronjob per profile

```bash
# Harga emas → profile finance
hermes --profile finance cron create "0 9 * * 1-5" \
  "Ambil harga emas Antam dari logammulia.com. Alert kalau berubah >1%." \
  --skill harga-emas --name "Emas Harian"

# Weekly saham screening → profile finance
hermes --profile finance cron create "0 20 * * 5" \
  "Screening JII: PER<15, DY>4%, ROE>12%. Kirim top 5." \
  --skill saham-syariah --name "Weekly Screen"

# Coding tip of the day → profile coding
hermes --profile coding cron create "0 8 * * *" \
  "Kasih 1 coding tip/trick harian tentang topik random (Python/JS/Go/Git/Terminal). Format: 💡 Tip + contoh code 5 baris + penjelasan 2 kalimat." \
  --name "Daily Coding Tip"
```

---

## 5. Tabel ringkasan — 6 Profile

| Profile | Bot | Model | Reasoning | Skills | lean-ctx benefit |
|---|---|---|---|---|---|
| `main` | @hataf_main_bot | Kimi K2.6 | medium | Semua 25 | Medium |
| `coding` | @hataf_code_bot | Kimi K2.6 + Flash routing | high | coding-mentor, web-dev, backend, frontend, ui-ux, code-review | High |
| `research` | @hataf_research_bot | DeepSeek Flash (1M ctx) | high | deep-analysis, research-citation, study-buddy, pdf-summarize, video-summary, data-analysis | Medium |
| `islamic` | @hataf_islam_bot | DeepSeek Flash (1M ctx) | high | islamic-study, research-citation | Low |
| `finance` | @hataf_money_bot | DeepSeek Flash + routing | medium | financial-literacy, saham-syariah, harga-emas, web-scrape, marketplace-scrape | Medium |
| `media` | @hataf_media_bot | Qwen3.6 Plus | low | copywriting, music-download, social-media-scrape, video-summary, web-scrape | Low |

> Semua profile memakai **OpenCode Go** sebagai provider. Tidak ada API key DeepSeek atau OpenAI terpisah.
> Install **lean-ctx** (`cargo install lean-ctx`) untuk compress tool output 89-99% — terutama berguna di profile `coding`.

---

## 6. FAQ

**Q: Resource usage di VPS berapa?**

Per gateway process ≈ 50-150 MB RAM. 6 profile = ~400-900 MB idle. 2 GB VPS comfortable, 4 GB aman.

**Q: Kalau 1 profile crash, yang lain ikut?**

Tidak. Tiap profile = proses terpisah. Systemd auto-restart kalau crash.

**Q: Bisa share memory antar profile?**

Default: TIDAK. Tiap profile punya `memories/` sendiri. Kalau mau share fakta, tulis manual di MEMORY.md tiap profile.

**Q: Berapa biaya per bulan roughly?**

| Komponen | Estimasi |
|---|---|
| VPS 2-4 GB | $5-15/bulan |
| OpenCode Go | $10/bulan (fixed) |
| OpenRouter (auxiliary) | $1-5/bulan |
| Firecrawl (web scrape) | Free tier / $0-20 |
| **Total** | **~$16-50/bulan** |

---

## 7. Lanjut dari sini

1. `hermes profile create coding` → test dulu 1 profile
2. Confirm folder structure yang di-generate
3. Copy SOUL.md + config + skills ke profile
4. `hermes --profile coding gateway` → test di Telegram
5. Kalau jalan → scale ke profile lain
6. Install as systemd services
7. Set cronjob per profile

**Jangan setup 6 sekaligus** — start dari 1, pastiin jalan, baru expand.
