# 11 — Multi-Profile Telegram (1 Server, Banyak Bot, Per Workflow)

Tujuan: setup Hermes dengan banyak profile — tiap profile = 1 workflow/domain. Bot Telegram terpisah, skill set fokus, personality konsisten. Kayak punya "tim AI" yang masing-masing spesialis.

> Sumber: [Hermes Profiles docs](https://hermes-agent.nousresearch.com/docs/user-guide/profiles), [Profile Distributions](https://hermes-agent.nousresearch.com/docs/user-guide/profile-distributions), [Telegram Gateway](https://hermes-agent.nousresearch.com/docs/user-guide/messaging/telegram).

---

## 1. Konsep: kenapa organize per WORKFLOW

| 1 bot untuk semua | Multi-profile per workflow |
|---|---|
| Semua topik di 1 chat, context noisy | Tiap workflow punya context clean |
| Semua 25 skill loaded | Tiap bot cuma load 3-7 skill relevan |
| 1 personality untuk semua | Personality optimized per domain |
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

Buka [@BotFather](https://t.me/BotFather), bikin 4 bot:

```
/newbot → "Hataf Code"    → username: hataf_code_bot    → save token
/newbot → "Hataf Islam"   → username: hataf_islam_bot   → save token
/newbot → "Hataf Finance" → username: hataf_money_bot   → save token
/newbot → "Hataf Main"    → username: hataf_main_bot    → save token
```

Lo sekarang punya 4 token. Simpan baik-baik.

---

### Step 2: Bikin profile di Hermes

```bash
# Bikin profile
hermes profile create coding
hermes profile create islamic
hermes profile create finance
# (profile 'default' udah ada otomatis)
```

Ini bikin struktur:

```
~/.hermes/profiles/
├── coding/
│   ├── config.yaml
│   ├── .env
│   ├── SOUL.md
│   ├── memories/
│   └── skills/
├── islamic/
│   ├── config.yaml
│   ├── .env
│   ├── SOUL.md
│   ├── memories/
│   └── skills/
└── finance/
    ├── config.yaml
    ├── .env
    ├── SOUL.md
    ├── memories/
    └── skills/
```

> ⚠️ **Catatan jujur**: gw belum 100% confirm exact folder structure per profile dari dokumentasi. Yang gw tau pasti: profile punya config override sendiri. Cek `hermes profile --help` untuk detail exact. Kalau structure beda, adapt panduan ini.

---

### Step 3: Setup tiap profile

#### Profile: `coding`

```bash
# Masuk ke profile coding
hermes --profile coding config edit
```

**`~/.hermes/profiles/coding/.env`**:
```bash
TELEGRAM_BOT_TOKEN=<token hataf_code_bot>
TELEGRAM_ALLOWED_USERS=<user_id_lo>
OPENAI_BASE_URL=https://api.deepseek.com/v1
OPENAI_API_KEY=<deepseek_key>
OPENCODE_GO_API_KEY=<opencode_go_key>
```

**`~/.hermes/profiles/coding/config.yaml`**:
```yaml
model:
  provider: opencode-go
  default: qwen3.6-plus        # model murah cukup buat ngajarin

agent:
  reasoning_effort: medium

smart_model_routing:
  enabled: false               # konsisten 1 model buat teaching

compression:
  enabled: true
  threshold: 0.50
  summary_provider: opencode-go
  summary_model: qwen3.5-plus

memory:
  memory_enabled: true
  user_profile_enabled: true

display:
  show_cost: true

streaming:
  enabled: true
  edit_interval: 0.3
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

GAYA:
- Kayak temen yang pinter ngajarin
- Celebrate progress tapi jujur soal gap
- Jangan fake enthusiasm ("Amazing!!!")
- Challenge kalau user mau loncat level
```

**Install skill yang relevan**:
```bash
# Copy skill coding-related ke profile
cp -r ~/.hermes/skills/coding-mentor ~/.hermes/profiles/coding/skills/
cp -r ~/.hermes/skills/web-development ~/.hermes/profiles/coding/skills/
cp -r ~/.hermes/skills/backend-proper ~/.hermes/profiles/coding/skills/
cp -r ~/.hermes/skills/frontend-design ~/.hermes/profiles/coding/skills/
cp -r ~/.hermes/skills/ui-ux ~/.hermes/profiles/coding/skills/
cp -r ~/.hermes/skills/code-review ~/.hermes/profiles/coding/skills/
```

---

#### Profile: `islamic`

**`~/.hermes/profiles/islamic/.env`**:
```bash
TELEGRAM_BOT_TOKEN=<token hataf_islam_bot>
TELEGRAM_ALLOWED_USERS=<user_id_lo>
OPENAI_BASE_URL=https://api.deepseek.com/v1
OPENAI_API_KEY=<deepseek_key>
OPENROUTER_API_KEY=<openrouter_key>
```

**`~/.hermes/profiles/islamic/config.yaml`**:
```yaml
model:
  provider: custom
  default: deepseek-v4-flash    # model KUAT — topik agama WAJIB akurat
  base_url: https://api.deepseek.com/v1
  context_length: 1000000

agent:
  reasoning_effort: high        # teliti, jangan buru-buru jawab

smart_model_routing:
  enabled: false                # JANGAN cheap model untuk agama (halu risk)

compression:
  enabled: true
  threshold: 0.60              # preserve context lebih lama (biar gak kehilangan dalil)

memory:
  memory_enabled: true
  user_profile_enabled: true

streaming:
  enabled: true
```

**`~/.hermes/profiles/islamic/SOUL.md`**:
```markdown
Lo adalah asisten pencarian ilmu Islam. Bahasa lo-gw, casual tapi hormat.

⛔ LARANGAN MUTLAK:
- DILARANG mengarang ayat Al-Quran
- DILARANG mengarang hadits (matan, sanad, perawi)
- DILARANG bilang "hukumnya halal/haram" tanpa rujukan ulama spesifik
- DILARANG tafsir sendiri — HARUS rujuk mufassir (Ibnu Katsir, Kemenag, dll)
- DILARANG keluarkan fatwa
- DILARANG tentukan derajat hadits tanpa verify sumber

KALAU GAK KETEMU SUMBER → bilang "gak ketemu". TITIK.

SUMBER YANG BOLEH:
- Al-Quran: quran.kemenag.go.id, quran.com, tanzil.net
- Hadits: sunnah.com, dorar.net, hadits.id
- Tafsir: tafsirweb.com, Ibnu Katsir (via quran.com)
- Fiqh: MUI (mui.or.id), NU Online (islam.nu.or.id), Muhammadiyah, islamqa.info, islamweb.net
- Doa: Hisnul Muslim, sunnah.com

HANDLING KHILAFIYAH:
- WAJIB present semua pendapat (multi-mazhab)
- JANGAN pilih satu sebagai "yang benar"
- Label setiap pendapat dengan mazhab/ulama

FORMAT OUTPUT:
- 📌 TL;DR
- 📖 Dalil (ayat + hadits dengan URL)
- ⚖️ Pendapat ulama (tabel multi-mazhab)
- ⚠️ Catatan (khilafiyah, disclaimer)
- 🔗 Sumber (WAJIB setiap output)

Disclaimer wajib di akhir: "Untuk fatwa personal, konsultasi ustadz yang lo percaya."
```

**Install skill**:
```bash
cp -r ~/.hermes/skills/islamic-study ~/.hermes/profiles/islamic/skills/
cp -r ~/.hermes/skills/research-citation ~/.hermes/profiles/islamic/skills/
```

---

#### Profile: `finance`

**`~/.hermes/profiles/finance/.env`**:
```bash
TELEGRAM_BOT_TOKEN=<token hataf_money_bot>
TELEGRAM_ALLOWED_USERS=<user_id_lo>
OPENAI_BASE_URL=https://api.deepseek.com/v1
OPENAI_API_KEY=<deepseek_key>
OPENCODE_GO_API_KEY=<opencode_go_key>
FIRECRAWL_API_KEY=<firecrawl_key>
```

**`~/.hermes/profiles/finance/config.yaml`**:
```yaml
model:
  provider: custom
  default: deepseek-v4-flash
  base_url: https://api.deepseek.com/v1
  context_length: 1000000

smart_model_routing:
  enabled: true
  max_simple_chars: 160
  max_simple_words: 28
  cheap_model:
    provider: opencode-go
    model: qwen3.5-plus          # buat quick Q&A ("apa itu PER?")

agent:
  reasoning_effort: medium

compression:
  enabled: true
  threshold: 0.50
  summary_provider: opencode-go
  summary_model: qwen3.5-plus

# Cronjob harga emas dan screening saham jalan di profile ini
streaming:
  enabled: true
```

**`~/.hermes/profiles/finance/SOUL.md`**:
```markdown
Lo adalah edukator keuangan personal. Bahasa lo-gw, casual.

⚠️ BATASAN TEGAS:
- BUKAN financial advisor. Lo TIDAK bilang "invest di X" atau "jual Y"
- BUKAN pengganti CFP (Certified Financial Planner)
- Lo kasih FRAMEWORK, HITUNGAN, dan OPSI — keputusan di tangan user
- Untuk keputusan besar (>10% net worth) → arahkan ke CFP berlisensi

BISA:
- Jelasin konsep (compound interest, DCA, diversifikasi, dll)
- Hitung skenario (pakai execute_code buat precision)
- Bandingkan produk (deposito vs reksadana vs sukuk vs saham)
- Screening saham syariah berdasarkan DES OJK
- Monitor harga (emas, saham) dari sumber resmi
- Organize budget

DISCLAIMER WAJIB di output yang melibatkan angka/produk:
"⚠️ Bukan nasihat investasi. Data screening otomatis. DYOR."

FORMAT:
- 📌 TL;DR
- 📊 Data / hitungan (pakai tabel)
- 💡 Insight
- ⚠️ Risiko / caveat
- 🔗 Sumber (WAJIB)
```

**Install skill**:
```bash
cp -r ~/.hermes/skills/financial-literacy ~/.hermes/profiles/finance/skills/
cp -r ~/.hermes/skills/saham-syariah ~/.hermes/profiles/finance/skills/
cp -r ~/.hermes/skills/harga-emas ~/.hermes/profiles/finance/skills/
cp -r ~/.hermes/skills/web-scrape ~/.hermes/profiles/finance/skills/
cp -r ~/.hermes/skills/research-citation ~/.hermes/profiles/finance/skills/
```

---

### Step 4: Setup gateway per profile

```bash
# Setup gateway tiap profile (interactive wizard per profile)
hermes --profile coding gateway setup     # pilih Telegram, paste token coding bot
hermes --profile islamic gateway setup    # paste token islamic bot
hermes --profile finance gateway setup    # paste token finance bot
```

---

### Step 5: Jalanin semua gateway

#### Opsi A: Manual (testing)

```bash
# Terminal 1
hermes --profile coding gateway

# Terminal 2
hermes --profile islamic gateway

# Terminal 3
hermes --profile finance gateway
```

#### Opsi B: Systemd services (production, recommended)

Bikin 1 service file per profile. Contoh untuk `coding`:

```bash
sudo cat > /etc/systemd/system/hermes-coding.service << 'EOF'
[Unit]
Description=Hermes Agent - Coding Profile
After=network.target

[Service]
Type=simple
User=<username_lo>
ExecStart=/home/<username_lo>/.local/bin/hermes --profile coding gateway
Restart=always
RestartSec=10
Environment=HOME=/home/<username_lo>

[Install]
WantedBy=multi-user.target
EOF
```

Repeat untuk `islamic` dan `finance` (ganti nama service + profile flag).

```bash
# Enable semua
sudo systemctl daemon-reload
sudo systemctl enable hermes-coding hermes-islamic hermes-finance
sudo systemctl start hermes-coding hermes-islamic hermes-finance

# Cek status
sudo systemctl status hermes-coding
sudo systemctl status hermes-islamic
sudo systemctl status hermes-finance

# Liat logs
journalctl -u hermes-coding -f
journalctl -u hermes-islamic -f
journalctl -u hermes-finance -f
```

---

### Step 6: Bikin grup Telegram per section

1. Bikin Telegram Group: "📚 Coding Hataf"
2. Add bot `@hataf_code_bot` ke group
3. Bikin Telegram Group: "🕌 Islamic Study"
4. Add bot `@hataf_islam_bot` ke group
5. Bikin Telegram Group: "💰 Finance & Investasi"
6. Add bot `@hataf_money_bot` ke group

Buat DM personal → pake `@hataf_main_bot` (profile default, semua skill).

---

### Step 7: Set home channel per profile (untuk cronjob delivery)

Di tiap group, kirim:

```
/sethome
```

Bot akan set group tersebut sebagai delivery target untuk cronjob profile itu.

Atau manual di `.env` profile:

```bash
# Di ~/.hermes/profiles/finance/.env
TELEGRAM_HOME_CHANNEL=-1001234567890    # ID grup Finance
```

---

## 4. Cronjob per profile

Cronjob attach ke profile masing-masing:

```bash
# Harga emas → jalan di profile finance, deliver ke grup Finance
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

## 5. Tabel ringkasan — PER WORKFLOW

| Profile | Workflow | Bot | Model (Cheap) | Model (Balanced) | Skills | Personality |
|---|---|---|---|---|---|---|
| `main` | **Meta / General** | @hataf_main_bot | Flash medium | K2.6 medium | Semua 25 | General purpose, anti-halu |
| `code` | **Development** | @hataf_code_bot | Flash high | K2.6 high | coding-mentor, web-dev, backend, frontend, ui-ux, code-review, devops, data-analysis | Temen coding, eksekutor |
| `research` | **Research & Analysis** | @hataf_research_bot | Flash high | Flash high (1M context) | deep-analysis, research-citation, study-buddy, pdf-summarize, video-summary | Researcher, citation-enforcer |
| `islam` | **Islamic Study** | @hataf_islam_bot | Flash high | Flash high | islamic-study, research-citation | Asisten ilmu, anti-halu dalil, multi-mazhab |
| `finance` | **Finance & Monitor** | @hataf_money_bot | Flash medium + routing | Flash medium + routing | financial-literacy, saham-syariah, harga-emas, web-scrape, marketplace-scrape | Edukator, kalkulator, disclaimer enforcer |
| `media` | **Content & Media** | @hataf_media_bot | Qwen3.5 low | Qwen3.6 low | copywriting, music-download, social-media-scrape, video-summary | Content creator assistant |

### Kenapa model allocation berbeda per workflow?

```
Development:  HIGH reasoning — coding butuh accuracy
Research:     HIGH reasoning + LONG context — analisis + PDF panjang
Islamic:      HIGH reasoning — HARAM ngarang dalil, harus careful
Finance:      MEDIUM + routing — mix Q&A simple + screening yang perlu akurat
Media:        LOW reasoning — content generation gak perlu deep thinking
```

### Cronjob distribution per workflow

```
Profile: finance
├── Harga emas harian (logammulia.com) — 09:00 WIB weekdays
├── Weekly screening saham syariah — Jumat 20:00
└── DES update check — 1 Mei & 1 November

Profile: main
├── Weekly self-improvement review — Minggu 20:00
└── System health check — setiap 6 jam

Profile: code
└── Daily coding tip (optional) — 08:00

Profile: media
└── (On-demand, no cron)

Profile: research
└── (On-demand, no cron)
```

---

## 6. FAQ

**Q: Resource usage di VPS berapa?**

Per gateway process ≈ 50-150 MB RAM (Python process). 4 profile = ~400-600 MB idle. 1 GB VPS tight, 2 GB comfortable, 4 GB aman.

> ⚠️ Angka di atas **perkiraan kasar** dari pengalaman umum Python web processes, bukan benchmark resmi Hermes. Test di setup lo.

**Q: Kalau 1 profile crash, yang lain ikut?**

Tidak. Tiap profile = proses terpisah. Systemd auto-restart kalau crash.

**Q: Bisa share memory antar profile?**

Default: TIDAK. Tiap profile punya `memories/` sendiri. Kalau lo mau share fakta (misal "user timezone Asia/Jakarta"), tulis manual di MEMORY.md tiap profile.

**Q: Bisa 1 user di multiple group?**

Bisa. `TELEGRAM_ALLOWED_USERS` di-set sama (user ID lo) di semua profile. Lo bisa chat di semua group.

**Q: Bisa gw tambahin profile baru nanti?**

```bash
hermes profile create <nama-baru>
# Setup .env, config.yaml, SOUL.md, skills
# Bikin bot baru di BotFather
# hermes --profile <nama-baru> gateway setup
# Bikin service baru
```

**Q: Berapa biaya per bulan roughly?**

| Komponen | Estimasi |
|---|---|
| VPS 2-4 GB | $5-15/bulan (DigitalOcean, Hetzner) |
| DeepSeek V4 Flash | $5-30/bulan tergantung usage |
| OpenCode Go | $10/bulan (fixed subscription) |
| OpenRouter (auxiliary) | $1-5/bulan (Gemini Flash murah) |
| Firecrawl (web scrape) | Free tier mungkin cukup, atau $0-20 |
| **Total** | **~$20-60/bulan** |

> ⚠️ Estimasi sangat kasar. Tergantung seberapa aktif lo chat + berapa cronjob jalan. Monitor di `/usage` tiap profile.

---

## 7. Yang gw belum yakin

- **Exact profile folder structure**: dari docs dan README, profiles override config — tapi exact path untuk `skills/`, `memories/`, `SOUL.md` per profile gw belum 100% confirm dari source code. **Jalanin `hermes profile create X` dulu**, liat apa yang di-generate, lalu adapt.
- **Multiple systemd services conflict**: secara teori gak conflict (beda PID, beda port kalau ada, beda bot token). Tapi kalau Hermes pake shared lockfile (misal `~/.hermes/state.db`), mungkin ada issue. Test dulu 2 profile sebelum scale ke 4.
- **Telegram rate limit**: kalau 4 bot lo di-spam barengan, Telegram punya rate limit per bot (~30 msg/detik). Untuk personal use, gak akan hit.
- **Apakah `hermes --profile X gateway install --system` otomatis bikin service name unik?** Kemungkinan ya, tapi kalau conflict → bikin manual pakai template systemd di atas.

---

## 8. Script setup cepat

Simpan ini buat reference:

```bash
#!/bin/bash
# setup-profiles.sh — jalanin setelah Hermes terinstall

PROFILES=("coding" "islamic" "finance")

for profile in "${PROFILES[@]}"; do
  echo "=== Creating profile: $profile ==="
  hermes profile create "$profile"
  
  echo "Copy SOUL.md..."
  # (lo harus udah siapin file SOUL.md per profile di ~/hataf-hermes/examples/profiles/)
  # cp examples/profiles/$profile/SOUL.md ~/.hermes/profiles/$profile/SOUL.md
  
  echo "Copy skills..."
  # cp -r examples/profiles/$profile/skills/* ~/.hermes/profiles/$profile/skills/
  
  echo "Setup gateway..."
  echo "  → Jalanin: hermes --profile $profile gateway setup"
  echo ""
done

echo "Done. Sekarang setup gateway per profile dan start services."
```

---

## Lanjut dari sini

1. `hermes profile create coding` → test dulu 1 profile
2. Confirm folder structure yang di-generate
3. Copy SOUL.md + config + skills ke profile
4. `hermes --profile coding gateway` → test di Telegram
5. Kalau jalan → scale ke profile lain
6. Install as systemd services
7. Set cronjob per profile

**Jangan setup 4 sekaligus** — start dari 1, pastiin jalan, baru expand.
