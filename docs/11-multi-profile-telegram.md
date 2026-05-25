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
# (profile 'default' = main kalau lo mau pake nama default)
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

```bash
# Masuk ke profile coding
hermes --profile coding config edit
```

**`~/.hermes/profiles/coding/.env`**:
```bash
TELEGRAM_BOT_TOKEN=<token hataf_code_bot>
TELEGRAM_ALLOWED_USERS=<user_id_lo>
OPENCODE_GO_API_KEY=<opencode_go_key>
OPENROUTER_API_KEY=<openrouter_key>    # untuk vision auxiliary
```

**`~/.hermes/profiles/coding/config.yaml`**:
```yaml
# Profile: coding — Development workflow
# Personality: temen coding, eksekutor, strict anti-hallucination untuk code
model:
  provider: opencode-go
  default: kimi-k2.6            # top quality untuk coding (SWE-Bench 58.6%)

agent:
  reasoning_effort: high        # coding butuh reasoning kuat

smart_model_routing:
  enabled: true
  max_simple_chars: 160
  max_simple_words: 28
  cheap_model:
    provider: opencode-go
    model: deepseek-v4-flash    # Q&A coding simpel → murah

fallback_model:
  provider: opencode-go
  model: deepseek-v4-pro

auxiliary:
  vision:
    provider: openrouter
    model: google/gemini-2.5-flash
  web_extract:
    provider: opencode-go
    model: qwen3.5-plus

compression:
  enabled: true
  threshold: 0.55               # coding sessions panjang
  summary_provider: opencode-go
  summary_model: deepseek-v4-flash

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
OPENCODE_GO_API_KEY=<opencode_go_key>
OPENROUTER_API_KEY=<openrouter_key>    # untuk vision auxiliary
FIRECRAWL_API_KEY=<firecrawl_key>      # untuk web extract hadits/quran
```

**`~/.hermes/profiles/islamic/config.yaml`**:
```yaml
# Profile: islamic — Islamic Study workflow
# Personality: asisten ilmu, anti-halu dalil, sumber wajib, multi-mazhab
model:
  provider: opencode-go
  default: deepseek-v4-flash    # 1M context, dibutuhkan untuk research dalil panjang

agent:
  reasoning_effort: high        # WAJIB — agama butuh ketelitian, jangan buru-buru

smart_model_routing:
  enabled: false                # JANGAN cheap routing untuk agama — halu risk tinggi

fallback_model:
  provider: opencode-go
  model: deepseek-v4-pro

auxiliary:
  vision:
    provider: openrouter
    model: google/gemini-2.5-flash
  web_extract:
    provider: opencode-go
    model: qwen3.5-plus

compression:
  enabled: true
  threshold: 0.60              # preserve context lebih lama — dalil jangan kehapus
  summary_provider: opencode-go
  summary_model: qwen3.5-plus

memory:
  memory_enabled: true
  user_profile_enabled: true

streaming:
  enabled: true
  edit_interval: 0.3
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
OPENCODE_GO_API_KEY=<opencode_go_key>
OPENROUTER_API_KEY=<openrouter_key>
FIRECRAWL_API_KEY=<firecrawl_key>      # untuk scrape logammulia, OJK, marketplace
```

**`~/.hermes/profiles/finance/config.yaml`**:
```yaml
# Profile: finance — Finance & Monitoring workflow
# Personality: edukator keuangan, kalkulator, BUKAN financial advisor
model:
  provider: opencode-go
  default: deepseek-v4-flash    # 1M context, cukup untuk analisis laporan keuangan

smart_model_routing:
  enabled: true
  max_simple_chars: 160
  max_simple_words: 28
  cheap_model:
    provider: opencode-go
    model: qwen3.5-plus          # Q&A simpel: "apa itu PER?", "jelaskan DCA"

fallback_model:
  provider: opencode-go
  model: deepseek-v4-pro

agent:
  reasoning_effort: medium

auxiliary:
  vision:
    provider: openrouter
    model: google/gemini-2.5-flash
  web_extract:
    provider: opencode-go
    model: qwen3.5-plus

compression:
  enabled: true
  threshold: 0.50
  summary_provider: opencode-go
  summary_model: qwen3.5-plus

streaming:
  enabled: true
  edit_interval: 0.3
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

#### Profile: `research`

**`~/.hermes/profiles/research/.env`**:
```bash
TELEGRAM_BOT_TOKEN=<token hataf_research_bot>
TELEGRAM_ALLOWED_USERS=<user_id_lo>
OPENCODE_GO_API_KEY=<opencode_go_key>
OPENROUTER_API_KEY=<openrouter_key>
FIRECRAWL_API_KEY=<firecrawl_key>
```

**`~/.hermes/profiles/research/config.yaml`**:
```yaml
# Profile: research — Research & Analysis workflow
# Personality: researcher serius, citation-enforcer, gak jawab tanpa sumber
model:
  provider: opencode-go
  default: deepseek-v4-flash    # 1M context — PDF panjang, dokumen tebal

agent:
  reasoning_effort: high        # analisis mendalam butuh reasoning kuat

smart_model_routing:
  enabled: false                # semua research butuh model yang sama kuat

fallback_model:
  provider: opencode-go
  model: kimi-k2.6

auxiliary:
  vision:
    provider: openrouter
    model: google/gemini-2.5-flash    # baca chart/diagram di paper
  web_extract:
    provider: opencode-go
    model: qwen3.5-plus

compression:
  enabled: true
  threshold: 0.65              # preserve lebih lama — dokumen panjang butuh context
  summary_provider: opencode-go
  summary_model: qwen3.5-plus

memory:
  memory_enabled: true
  user_profile_enabled: true

streaming:
  enabled: true
  edit_interval: 0.3
```

**`~/.hermes/profiles/research/SOUL.md`**:
```markdown
Lo adalah research assistant. Bahasa lo-gw, direct, no fluff.

MISI:
Bantu user riset topik apapun — temukan fakta, bongkar asumsi, kasih
analisis multi-perspektif. BUKAN opini — data + reasoning.

ATURAN WAJIB:
- Setiap klaim HARUS ada sumber (URL yang bisa dicek)
- Kalau gak ada sumber → bilang "gak ketemu", STOP
- JANGAN jawab dari memori untuk fakta time-sensitive
- Correlation ≠ causation — SELALU sebut ini kalau relevan
- Kalau pertanyaan butuh expert (dokter, pengacara) → arahkan ke expert

CARA KERJA:
1. Decompose pertanyaan jadi sub-questions yang lebih kecil
2. Search per sub-question
3. Kasih jawaban PER sub-question dengan citation
4. Synthesize jadi jawaban utama
5. List "yang gw belum tau" secara eksplisit

FORMAT:
- 📌 TL;DR (conditional — bukan absolute)
- 🔍 Evidence per claim (dengan URL)
- ⚖️ Multiple perspectives (kalau ada)
- ❓ What I don't know / gaps
- 🔗 Sumber (WAJIB di akhir)

DILARANG:
- "Menurut pengetahuan saya..." tanpa sumber
- Jawab soal prediksi masa depan dengan konfiden
- Single perspective untuk topik yang contested
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

---

#### Profile: `media`

**`~/.hermes/profiles/media/.env`**:
```bash
TELEGRAM_BOT_TOKEN=<token hataf_media_bot>
TELEGRAM_ALLOWED_USERS=<user_id_lo>
OPENCODE_GO_API_KEY=<opencode_go_key>
OPENROUTER_API_KEY=<openrouter_key>
```

**`~/.hermes/profiles/media/config.yaml`**:
```yaml
# Profile: media — Content & Media workflow
# Personality: content creator assistant, creative, casual, efisien
model:
  provider: opencode-go
  default: qwen3.6-plus         # cukup untuk content generation

agent:
  reasoning_effort: low         # content creation gak butuh deep thinking

smart_model_routing:
  enabled: true
  max_simple_chars: 200
  max_simple_words: 40
  cheap_model:
    provider: opencode-go
    model: qwen3.5-plus          # edit caption, reformat, translate → murah

fallback_model:
  provider: opencode-go
  model: deepseek-v4-flash

auxiliary:
  vision:
    provider: openrouter
    model: google/gemini-2.5-flash    # analyze screenshot/image untuk konten
  web_extract:
    provider: opencode-go
    model: qwen3.5-plus

compression:
  enabled: true
  threshold: 0.45              # agresif — media session gak perlu history panjang
  summary_provider: opencode-go
  summary_model: qwen3.5-plus

memory:
  memory_enabled: true
  user_profile_enabled: true   # ingat preferensi tone/style user

streaming:
  enabled: true
  edit_interval: 0.3
```

**`~/.hermes/profiles/media/SOUL.md`**:
```markdown
Lo adalah content assistant. Bahasa lo-gw, casual, creative.

MISI:
Bantu bikin konten yang engaging — caption, artikel, script, download media.
Output = copy yang siap pakai, bukan draft perlu direvisi besar.

CARA KERJA PER REQUEST:
- Caption IG → tanya: audience? goal? tone? CTA?
- Artikel → tanya: platform? panjang? target reader?
- Script video → tanya: durasi? format (talking head / slideshow)?
- Download media → tanya: platform? format? kualitas?
- Kalau context udah jelas → langsung eksekusi, gak perlu nanya

GAYA NULIS:
- Specific > generic ("hemat 2 jam/hari" bukan "hemat waktu")
- Benefit > feature
- Hook kuat di kalimat pertama
- CTA clear dan CUMA 1 per piece
- DILARANG opener AI slop: "Di era digital...", "Sebagai makhluk sosial..."

FORMAT OUTPUT:
- Langsung kasih konten (gak perlu intro panjang)
- Kasih 1-2 variasi kalau relevan
- Note singkat: "ini untuk [platform], [tone], [CTA]"

DILARANG:
- Ngarang fakta/statistik buat dipake di konten
- Janjiin hasil tertentu ("caption ini pasti viral")
- Fake enthusiasm di konten ("AMAZING PRODUCT!!!")
```

**Install skill**:
```bash
cp -r ~/.hermes/skills/copywriting ~/.hermes/profiles/media/skills/
cp -r ~/.hermes/skills/music-download ~/.hermes/profiles/media/skills/
cp -r ~/.hermes/skills/social-media-scrape ~/.hermes/profiles/media/skills/
cp -r ~/.hermes/skills/video-summary ~/.hermes/profiles/media/skills/
cp -r ~/.hermes/skills/web-scrape ~/.hermes/profiles/media/skills/
```

---

#### Profile: `main` (default — General Purpose)

**`~/.hermes/profiles/main/.env`** (atau `~/.hermes/.env` kalau profile default):
```bash
TELEGRAM_BOT_TOKEN=<token hataf_main_bot>
TELEGRAM_ALLOWED_USERS=<user_id_lo>
OPENCODE_GO_API_KEY=<opencode_go_key>
OPENROUTER_API_KEY=<openrouter_key>
FIRECRAWL_API_KEY=<firecrawl_key>
```

**`~/.hermes/profiles/main/config.yaml`**:
```yaml
# Profile: main — General purpose, DM personal
# Semua 25 skill aktif, primary model terbaik
model:
  provider: opencode-go
  default: kimi-k2.6            # terbaik di OpenCode Go untuk general use

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

agent:
  reasoning_effort: medium

auxiliary:
  vision:
    provider: openrouter
    model: google/gemini-2.5-flash
  web_extract:
    provider: opencode-go
    model: qwen3.5-plus
  approval:
    provider: opencode-go
    model: deepseek-v4-flash

compression:
  enabled: true
  threshold: 0.50
  summary_provider: opencode-go
  summary_model: deepseek-v4-flash

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
  edit_interval: 0.3

display:
  show_cost: true
```

**Install skill** (semua 25):
```bash
cp -r ~/.hermes/skills/* ~/.hermes/profiles/main/skills/
```

---

### Step 4: Setup gateway per profile

```bash
# Setup gateway tiap profile (interactive wizard per profile)
hermes --profile main     gateway setup    # paste token main bot
hermes --profile coding   gateway setup    # paste token coding bot
hermes --profile research gateway setup    # paste token research bot
hermes --profile islamic  gateway setup    # paste token islamic bot
hermes --profile finance  gateway setup    # paste token finance bot
hermes --profile media    gateway setup    # paste token media bot
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

Buat semua profile (ganti `coding` → nama profile masing-masing):

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

> Tips: lo bisa mulai dari 2-3 grup dulu, tambah sisanya kalau udah nyaman.

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

## 5. Tabel ringkasan — 6 Profile

| Profile | Bot | Grup | Model | Reasoning | Skills | Personality |
|---|---|---|---|---|---|---|
| `main` | @hataf_main_bot | 🏠 DM personal | Kimi K2.6 | medium | Semua 25 | General purpose, anti-halu, semua bisa |
| `coding` | @hataf_code_bot | 💻 Coding | Kimi K2.6 + Flash routing | high | coding-mentor, web-dev, backend, frontend, ui-ux, code-review, devops, data-analysis | Temen coding, eksekutor, strict |
| `research` | @hataf_research_bot | 🔬 Research | DeepSeek Flash (1M ctx) | high | deep-analysis, research-citation, study-buddy, pdf-summarize, video-summary, data-analysis | Researcher, citation-enforcer |
| `islamic` | @hataf_islam_bot | 🕌 Islamic | DeepSeek Flash (1M ctx) | high | islamic-study, research-citation | Asisten ilmu, anti-halu dalil, multi-mazhab |
| `finance` | @hataf_money_bot | 💰 Finance | DeepSeek Flash + routing | medium | financial-literacy, saham-syariah, harga-emas, web-scrape, marketplace-scrape | Edukator, kalkulator, disclaimer enforcer |
| `media` | @hataf_media_bot | 🎨 Media | Qwen3.6 Plus | low | copywriting, music-download, social-media-scrape, video-summary, web-scrape | Content creator assistant, creative |

> Semua profile memakai **OpenCode Go** sebagai provider. Tidak ada API key DeepSeek atau OpenAI terpisah — cukup `OPENCODE_GO_API_KEY` + `OPENROUTER_API_KEY` untuk vision.

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

```bash
#!/bin/bash
# setup-profiles.sh — jalanin setelah Hermes terinstall

PROFILES=("main" "coding" "research" "islamic" "finance" "media")

for profile in "${PROFILES[@]}"; do
  echo "=== Creating profile: $profile ==="
  hermes profile create "$profile"
  echo "  → Copy SOUL.md, config.yaml, .env, skills ke ~/.hermes/profiles/$profile/"
  echo "  → Jalanin: hermes --profile $profile gateway setup"
  echo ""
done

echo "Done. Setup gateway per profile, lalu start services."
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
