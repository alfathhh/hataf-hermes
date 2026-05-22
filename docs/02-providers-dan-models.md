# 02 — Providers & Model Recommendation

Tujuan: pasang DeepSeek V4 Flash sebagai model utama, OpenCode Go sebagai bundle model coding sehari-hari, dan kasih tabel rekomendasi model per use case.

> Sumber: [Hermes Configuration docs](https://hermes-agent.nousresearch.com/docs/user-guide/configuration), [DeepSeek API docs](https://api-docs.deepseek.com/), [OpenCode Go](https://opencode.ai/go).

---

## 1. Konsep penting: provider vs model

| Istilah | Artinya |
|---|---|
| **Provider** | Layanan yang kasih akses LLM via API. Contoh: DeepSeek, OpenCode Go, OpenRouter, Anthropic. |
| **Model** | LLM spesifik. Contoh: `deepseek-v4-flash`, `kimi-k2.6`, `glm-4-plus`. Satu provider bisa ngehost banyak model. |
| **Endpoint** | URL API provider. DeepSeek: `https://api.deepseek.com/v1`. OpenCode Go: dikelola Hermes (provider native `opencode-go`). |
| **API mode** | Format request — `chat_completions` (OpenAI), `anthropic_messages` (Anthropic), atau `responses` (OpenAI Codex). |

Hermes **abstraksi** semua ini — lo set provider sekali, lalu pilih model dengan `/model`.

---

## 2. Setup provider 1: DeepSeek V4 Flash (PRIMARY)

DeepSeek pake API yang **OpenAI-compatible**, jadi setup-nya via custom endpoint di Hermes.

### Step 1: Daftar & dapetkan API key

1. Buka [platform.deepseek.com](https://platform.deepseek.com)
2. Daftar / login
3. Tambah saldo (top up) — DeepSeek pricing per token, **bukan subscription**
4. Buat API key di section "API Keys"

> **Pricing (perlu dicek ulang di halaman resmi)**: per beberapa sumber sekunder seperti [apidog](http://apidog.com/blog/deepseek-v4-api-pricing) dan [codersera](https://codersera.com/blog/deepseek-v4-flash-deep-dive), DeepSeek V4 Flash kurang lebih $0.14/M input dan $0.28/M output. Halaman resmi: [api-docs.deepseek.com/quick_start/pricing](https://api-docs.deepseek.com/quick_start/pricing). **Pricing bisa berubah, cek sumber resminya sebelum jadiin patokan budget.**

### Step 2: Tambahkan ke Hermes

Edit `~/.hermes/.env`:

```bash
# DeepSeek sebagai custom OpenAI-compatible endpoint
OPENAI_BASE_URL=https://api.deepseek.com/v1
OPENAI_API_KEY=sk-deepseek-xxxxxxxxxxxx
LLM_MODEL=deepseek-v4-flash
```

Edit `~/.hermes/config.yaml`:

```yaml
model:
  provider: custom
  default: deepseek-v4-flash
  base_url: https://api.deepseek.com/v1
  context_length: 1000000   # 1M tokens — cek halaman resmi DeepSeek
```

> **Tentang context length**: berbagai sumber sekunder ([openrouter](https://openrouter.ai/deepseek/deepseek-v4-flash), [codersera](https://codersera.com/blog/deepseek-v4-flash-deep-dive)) sebut DeepSeek V4 Flash punya 1M-token context. Cek halaman model resmi sebelum acuan akhir.

### Step 3: Test

```bash
hermes -q "halo, sebut dirimu pakai 1 kalimat"
```

Kalau jawab tanpa error: DeepSeek udah konek.

### Optional: Mode "thinking" (deeper reasoning)

DeepSeek V4 Flash punya 2 mode (sumber: [api-docs.deepseek.com](https://api-docs.deepseek.com/)):

- **Non-thinking** (alias lama: `deepseek-chat`) — cepat, untuk task biasa
- **Thinking** (alias lama: `deepseek-reasoner`) — lebih lambat, lebih baik buat task kompleks (debug, analisis, planning)

Cara enable thinking di Hermes:

```yaml
# Di config.yaml
agent:
  reasoning_effort: "high"   # xhigh | high | medium | low | minimal | none
```

Atau switch on the fly di TUI:

```
/reasoning high
```

---

## 3. Setup provider 2: OpenCode Go (SECONDARY)

OpenCode Go itu provider **native** di Hermes — gak perlu hack custom endpoint.

### Step 1: Subscribe

1. Buka [opencode.ai/go](https://opencode.ai/go)
2. Subscribe (per snippet halaman: $5 bulan pertama, $10/bulan setelahnya — **cek halaman buat info terbaru**)
3. Dapetkan API key dari dashboard

### Step 2: Tambahkan ke Hermes

```bash
hermes config set OPENCODE_GO_API_KEY oc-go-xxxxxxxxxxxx
```

Itu otomatis ditulis ke `~/.hermes/.env`. Verifikasi:

```bash
cat ~/.hermes/.env | grep OPENCODE
# OPENCODE_GO_API_KEY=oc-go-xxxxx
```

### Step 3: Test

```bash
hermes chat --provider opencode-go --model glm-5.1 -q "halo"
```

---

## 4. Konfigurasi dual-provider dengan fallback

Strategi yang gw rekomendasikan: **DeepSeek V4 Flash sebagai primary**, **OpenCode Go sebagai fallback** (kalau DeepSeek down/rate-limit).

Di `~/.hermes/config.yaml`:

```yaml
# Primary: DeepSeek V4 Flash via custom endpoint
model:
  provider: custom
  default: deepseek-v4-flash
  base_url: https://api.deepseek.com/v1
  context_length: 1000000

# Fallback: OpenCode Go bundle (Kimi K2.6 contohnya)
fallback_model:
  provider: opencode-go
  model: kimi-k2.6        # ganti sesuai pilihan lo dari bundle

# Smart routing: turn pendek → cheap model
smart_model_routing:
  enabled: true
  max_simple_chars: 160
  max_simple_words: 28
  cheap_model:
    provider: opencode-go
    model: qwen3.5-plus    # model paling murah/cepat di bundle
```

**Cara kerja**:

1. Setiap request masuk dulu ke router
2. Kalau pertanyaan pendek & sederhana (<160 char, <28 kata, gak keliatan tool/code) → ke `cheap_model` (Qwen3.5 Plus di OpenCode Go) — jauh lebih murah
3. Kalau kompleks → ke primary (DeepSeek V4 Flash)
4. Kalau primary error/rate-limit → otomatis fallback ke `kimi-k2.6` di OpenCode Go, **conversation lo gak ilang**

---

## 5. Tabel rekomendasi model per use case

> **Caveat**: angka kemampuan model di bawah ini gw sebut berdasarkan sumber yang gw akses; **mungkin berubah** seiring rilis baru. Selalu coba sendiri di task lo yang spesifik — benchmark publik **bukan** prediktor pasti untuk task spesifik lo.

### A. Coding (utama harian)

| Tugas | Rekomendasi 1 | Rekomendasi 2 | Catatan |
|---|---|---|---|
| **Refactor besar, debug kompleks** | `deepseek-v4-flash` (thinking on) | `kimi-k2.6` (OpenCode Go) | Thinking mode bantu di logic deep |
| **Generate boilerplate cepat** | `qwen3.6-plus` (OpenCode Go) | `mimo-v2.5` (OpenCode Go) | Murah, cepat |
| **Code review, security audit** | `deepseek-v4-flash` (thinking) | `glm-5.1` (OpenCode Go) | Butuh konteks panjang & reasoning |
| **Quick Q&A coding** | `qwen3.5-plus` via smart routing | — | Otomatis kalau smart_model_routing aktif |

### B. Daily task (PDF, summary, analisis teks)

| Tugas | Rekomendasi 1 | Rekomendasi 2 | Catatan |
|---|---|---|---|
| **Baca PDF panjang & summarize** | `deepseek-v4-flash` (1M context) | `glm-5.1` (OpenCode Go) | Context panjang penting kalau PDF >50 halaman |
| **Summary singkat artikel** | `qwen3.6-plus` (OpenCode Go) | smart routing → cheap | Hemat |
| **Translate ID↔EN** | `qwen3.5-plus` atau `qwen3.6-plus` | — | Qwen kuat di Asia languages |

### C. Vision (analisis gambar, screenshot, foto)

> **PENTING — gw harus jujur**: gw belum bisa konfirmasi mana model di OpenCode Go bundle yang **resmi** punya vision. DeepSeek V4 Flash sendiri **bukan** model vision (V4 Flash teks/coding-focused per sumber yang gw akses). Untuk vision, jalur paling aman: **auxiliary vision via OpenRouter ke Gemini Flash atau GPT-4o**.

Setting di `config.yaml`:

```yaml
auxiliary:
  vision:
    provider: openrouter
    model: google/gemini-2.5-flash      # atau openai/gpt-4o-mini
    timeout: 30
```

Plus tambah `OPENROUTER_API_KEY` di `.env`. Ini cuma kepake saat lo kasih gambar — request normal tetep ke DeepSeek/OpenCode Go.

Detail full di [docs/05-tools-dan-mcp.md](05-tools-dan-mcp.md).

### D. Cronjob scraping (background, ringan)

| Tugas | Rekomendasi | Alasan |
|---|---|---|
| **Scrape & summarize 1 halaman web** | `qwen3.5-plus` atau `mimo-v2.5` (OpenCode Go) | Murah, cukup pinter buat ekstraksi |
| **Daily news brief multi-sumber** | `glm-5.1` (OpenCode Go) atau `deepseek-v4-flash` non-thinking | Konteks lebih panjang |
| **Monitoring alert (deteksi anomali sederhana)** | `qwen3.5-plus` | Cepat, jalan tiap menit OK |

Cron config nanti dijelasin di [docs/07-cron-scraping.md](07-cron-scraping.md).

### E. Video summary

> **Caveat**: "video summary" itu bukan task primitif LLM. Yang sebenernya terjadi adalah:
>
> 1. **Audio extracted** dari video (lewat `ffmpeg`)
> 2. **Speech-to-text (STT)** — Whisper local atau Groq Whisper
> 3. **Summarize transkrip** — di sini baru LLM masuk
>
> Hermes punya pipeline ini built-in via voice/STT tools. Detail di [docs/05-tools-dan-mcp.md](05-tools-dan-mcp.md).

Untuk step summarize:

| Durasi video | Model rekomendasi |
|---|---|
| < 30 menit | `qwen3.6-plus` (OpenCode Go) |
| 30 menit – 2 jam | `glm-5.1` atau `deepseek-v4-flash` |
| > 2 jam | `deepseek-v4-flash` (1M context) |

---

## 6. Auxiliary models (selalu ada di background)

Hermes pake "auxiliary" model untuk task ringan yang **bukan** main reasoning:

| Slot | Default | Untuk apa |
|---|---|---|
| `auxiliary.vision` | Gemini Flash via OpenRouter | Analisis gambar / screenshot |
| `auxiliary.web_extract` | Gemini Flash via OpenRouter | Extract isi web page |
| `auxiliary.approval` | Gemini Flash via OpenRouter | Klasifikasi command bahaya |
| `compression.summary_model` | Gemini Flash | Compress conversation lama |

Lo bisa swap ke OpenCode Go atau DeepSeek kalau mau:

```yaml
auxiliary:
  vision:
    provider: openrouter
    model: google/gemini-2.5-flash
  web_extract:
    provider: opencode-go
    model: qwen3.5-plus

compression:
  summary_provider: opencode-go
  summary_model: qwen3.5-plus
```

> **Catatan biaya**: auxiliary calls itu **frequent**. Pake yang murah-cepat (Gemini Flash, Qwen Plus). Jangan pake DeepSeek V4 Flash thinking di sini — itu tipenya bukan auxiliary task.

---

## 7. Switching model on the fly

Di TUI:

```
/model                              # Buka picker interaktif
/model opencode-go:kimi-k2.6        # Switch ke Kimi
/model custom:deepseek-v4-flash     # Switch balik ke DeepSeek
```

Untuk one-shot:

```bash
hermes chat --provider opencode-go --model glm-5.1 -q "..."
```

---

## 8. Verifikasi setup lo

Jalankan ini, semuanya harus bisa balas:

```bash
# DeepSeek primary
hermes chat --provider custom --model deepseek-v4-flash -q "halo"

# OpenCode Go fallback
hermes chat --provider opencode-go --model kimi-k2.6 -q "halo"

# OpenRouter auxiliary (kalau lo set)
hermes chat --provider openrouter --model google/gemini-2.5-flash -q "halo"
```

Lihat config aktif:

```bash
hermes config            # tampilan ringkas
hermes config edit       # buka di editor
```

---

## 9. Yang gw belum yakin (jujur)

- **Apakah `kimi-k2.6` itu nama persis di OpenCode Go bundle?** Snippet halaman opencode.ai/go nyebut "Kimi K2.6" — gw asumsiin ID-nya `kimi-k2.6` atau `kimi-k2-6`. **Cek dengan `hermes model` interaktif** setelah set API key — itu list model yang available dari endpoint.
- **Apakah ada hard cap quota di OpenCode Go subscription?** Halaman bilang "generous limits" — gw nggak punya angka pasti. Cek dashboard OpenCode lo.
- **Apakah DeepSeek V4 Flash di endpoint resmi DeepSeek (api.deepseek.com) selalu point ke versi terbaru?** Per [api-docs.deepseek.com/updates](https://api-docs.deepseek.com/updates), alias `deepseek-chat` dan `deepseek-reasoner` di-deprecate 2026-07-24 dan saat ini point ke `deepseek-v4-flash`. Habis tanggal itu, lo harus pakai `deepseek-v4-flash` eksplisit. Cek halaman update DeepSeek mendekati tanggal tersebut.

---

## Lanjut

→ [03 — SOUL.md](03-soul-md.md): bikin agent lo gak halu. **Ini paling penting.**
