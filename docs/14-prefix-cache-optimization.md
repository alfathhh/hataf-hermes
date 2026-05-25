# 14 — Prefix Cache Optimization (Teknik dari DeepSeek Reasonix)

Tujuan: hemat biaya API DeepSeek 60-90% dengan memanfaatkan **KV Cache / Prefix Caching** — teknik yang jadi inti arsitektur [DeepSeek Reasonix](https://github.com/esengine/DeepSeek-Reasonix).

> Sumber:
> - [DeepSeek KV Cache docs](https://api-docs.deepseek.com/guides/kv_cache)
> - [DeepSeek Context Caching on Disk announcement](https://api-docs.deepseek.com/news/news0802)
> - [Reasonix — DeepSeek-native agent](https://github.com/esengine/DeepSeek-Reasonix)
> - [DeepSeek Hermes integration](https://api-docs.deepseek.com/quick_start/agent_integrations/hermes)
> - [KV-Cache Aware Prompt Engineering](https://ankitbko.github.io/blog/2025/08/prompt-engineering-kv-cache/)

---

## 1. Apa itu Prefix Cache?

DeepSeek API secara otomatis **menyimpan (cache) bagian awal setiap request** ke disk. Kalau request berikutnya punya prefix yang sama, DeepSeek **skip rekomputasi** dan langsung pake cache.

```
Request A: [SYSTEM_PROMPT (2000 token)] + [user message A]
                ↑
                cache ini → disimpan ke disk

Request B: [SYSTEM_PROMPT (2000 token)] + [user message B]
                ↑
                HIT! → system prompt gak dihitung ulang
                → lo bayar jauh lebih murah untuk 2000 token itu
```

**Biaya cache hit** (per DeepSeek pricing):
| | Normal input | Cache hit |
|---|---|---|
| DeepSeek V4 Flash | $0.07/M token | $0.007/M token (**10x lebih murah**) |
| DeepSeek V4 Pro | $0.27/M token | $0.027/M token (**10x lebih murah**) |

Dengan SOUL.md + skills list ~3000-5000 token per session: **efek cache = signifikan**.

---

## 2. Prinsip Inti Reasonix: "Prefix Stability"

DeepSeek Reasonix dibangun di sekitar satu insight kunci:

> **"Keep the system prompt stable across all turns — the cache hit pays for itself."**

Kalau system prompt lo berubah tiap turn (dinamis, inject waktu/date/status), **cache MISS terus** → lo bayar full price setiap turn.

Kalau system prompt lo **frozen** (identik antar turn), DeepSeek cache-nya → lo bayar 10x lebih murah untuk bagian itu.

---

## 3. Kondisi untuk Cache Hit di DeepSeek

Dari [docs resmi DeepSeek](https://api-docs.deepseek.com/guides/kv_cache):

```
Cache hit TERJADI kalau:
  1. Prefix (bagian awal request) IDENTIK dengan request sebelumnya
  2. Prefix itu sudah "persisted" di disk DeepSeek

Cache miss TERJADI kalau:
  1. Ada APAPUN yang berubah di prefix (1 karakter pun)
  2. Cache belum ada (request pertama)
  3. Prefix di-reorder atau di-shuffle

Yang di-cache: PREFIX = semua yang ada SEBELUM konten yang berubah
```

**Untuk Hermes agent**, yang masuk prefix = system prompt (SOUL.md, tools, skills list).

---

## 4. Dua Cara Cache Miss yang Sering Terjadi

### Problem 1: Dynamic content di awal system prompt

```
❌ BURUK — cache miss setiap turn:
---
You are Hermes. Current time: {datetime.now()}    ← INI BERUBAH TIAP TURN
User preferences: {user.load_preferences()}
---

✅ BAGUS — cache hit:
---
You are Hermes. Bahasa lo-gw, casual, anti-halu.  ← STATIC
[tools list]
[skills list]
---
... (dynamic content ditaro SETELAH prefix stabil)
```

### Problem 2: /compress mengubah conversation history prefix

Setelah `/compress`, format conversation history berubah → cache miss untuk semua history.

Ini trade-off yang perlu dikelola (lihat section 6).

---

## 5. Cara Implementasi di Hermes + DeepSeek

### 5.1. Jaga SOUL.md agar TIDAK mengandung dynamic content

```markdown
# ❌ JANGAN: Dynamic content di SOUL.md

Lo adalah Hermes.
Tanggal sekarang: {date}          ← berubah tiap hari
Session dimulai: {session_start}  ← berubah tiap session
Memory terbaru: {latest_memory}   ← berubah setiap update

# ✅ LAKUKAN: SOUL.md 100% static

Lo adalah Hermes. Bahasa lo-gw.

ATURAN ANTI-HALU:
- DO: verify sebelum klaim
- DO NOT: ngarang
...
```

**SOUL.md di repo ini sudah benar** — tidak ada dynamic injection.

### 5.2. Urutan konten yang optimal (cache-friendly)

DeepSeek cache bekerja dari **awal** request. Urutkan konten dari yang PALING STABIL ke yang PALING DINAMIS:

```
Optimal ordering untuk prefix cache:
┌─────────────────────────────────────────┐  ← STABIL (cache ini)
│  1. SOUL.md / identity rules            │
│  2. Tool descriptions (jarang berubah)  │
│  3. Skills list level 0                 │
├─────────────────────────────────────────┤  ← SEMI-STABIL
│  4. Memory + User profile               │
│     (berubah jarang, tapi berubah)      │
├─────────────────────────────────────────┤  ← DINAMIS (gak di-cache per turn)
│  5. Conversation history                │
│  6. Current user message                │
└─────────────────────────────────────────┘
```

### 5.3. Config Hermes untuk prefix cache awareness

```yaml
# config.yaml — tambahan untuk prefix cache

# PASTIKAN: compression tidak merusak prefix stabilitas
compression:
  enabled: true
  threshold: 0.60    # Lebih tinggi dari default = compress lebih jarang
                     # Kenapa: setiap compression = restart conversation →
                     #          cache miss untuk history
                     # Trade-off: sedikit lebih mahal per session,
                     #            tapi prefix system prompt TETAP di-cache

# PASTIKAN: memory tidak di-inject ke bagian awal system prompt
# (Hermes by default taruh memory DI BAWAH tools, ini udah benar)
memory:
  memory_enabled: true
  user_profile_enabled: true
  memory_char_limit: 2200        # Jaga tetap pendek = less prefix disruption
```

### 5.4. Jaga SOUL.md tight dan konsisten

Setiap edit ke SOUL.md = **cache bust** untuk semua session. Jadi:

```
DO: edit SOUL.md hanya kalau benar-benar perlu
DO: setelah edit, tau bahwa 1-2 turn pertama akan cache miss
DO NOT: tambah dynamic timestamp, user-specific content ke SOUL.md
DO NOT: sering rotate SOUL.md (weekly update = oke, daily = tidak oke)
```

---

## 6. Reasonix vs Hermes: Analisis Komparasi

| Aspek | Reasonix | Hermes + DeepSeek |
|-------|----------|-------------------|
| **Prefix stability** | Built-in by design | Bisa dioptimasi (panduan ini) |
| **System prompt** | Frozen, minimal | SOUL.md statis ✅ |
| **Cache strategy** | Explicit cache-first loop | Auto oleh DeepSeek API |
| **Tool repair** | Auto tool-call repair | Hermes punya retry built-in |
| **Context window** | Agresif hemat | Compression threshold |
| **Provider** | DeepSeek API only | OpenCode Go + OpenRouter |
| **Unique value Hermes** | — | Skills system, memory persisten, Telegram, multi-profile |

**Kesimpulan**: Reasonix fokus di satu hal (prefix cache stability untuk coding agent). Hermes lebih general, tapi **bisa adopt prinsip yang sama** dengan adjustments di panduan ini.

---

## 7. Berapa Penghematan yang Bisa Diharapkan?

Asumsi session Hermes tipikal:
- System prompt (SOUL.md + tools + skills level 0): ~3000-5000 token
- Turns per session: 20-50 turns

```
Tanpa prefix cache optimization:
  3000 token × 50 turns = 150,000 token input (system prompt saja)
  @ $0.07/M = $0.0105/session

Dengan prefix cache (stabil, cache hit):
  3000 token × 1 turn = 3,000 token (full price, pertama)
  3000 token × 49 turns = 147,000 token @ $0.007/M (cache hit)
  Total = 3000 × $0.07/M + 147,000 × $0.007/M
        = $0.00021 + $0.001029 ≈ $0.001239/session

Penghematan: ~88% untuk sistem prompt bagian
```

Dalam konteks penggunaan real (banyak session/hari):
- **Estimasi total penghematan: 40-70%** dari total biaya input token
- Makin panjang session → makin besar penghematan

---

## 8. SOUL.md Audit Checklist

Cek SOUL.md lo dengan checklist ini sebelum deploy:

```
□ Tidak ada {datetime} / {date} / {time} injection
□ Tidak ada user-specific data yang berubah tiap session
□ Tidak ada random/UUID generator
□ Tidak ada "current session ID" di awal
□ Tidak ada "latest news" / "trending" content
□ Konten 100% identik untuk setiap user yang sama di tiap session
□ Memory/USER.md diletakkan SETELAH tool descriptions (bukan sebelum)

IF semua ✅ → prefix cache aktif optimal
IF ada ❌ → cache miss setiap turn → perbaiki
```

---

## 9. Config Khusus: Prefix-Cache Optimized

Tambahkan file config baru untuk setup yang paling hemat dengan prefix cache:

```yaml
# Gunakan di: ~/.hermes/config.yaml
# Cocok untuk: daily heavy usage, panjang session, budget ketat

model:
  provider: opencode-go
  default: deepseek-v4-flash

# Threshold compression lebih tinggi = prefix tetap stabil lebih lama
compression:
  enabled: true
  threshold: 0.65         # Default 0.50 → naik ke 0.65
                          # Artinya: compress hanya kalau 65% context penuh
                          # Trade-off: session sedikit lebih panjang sebelum compress,
                          #            tapi cache hit rate lebih tinggi

# Memory tetap tight = kurangi prefix disruption
memory:
  memory_enabled: true
  user_profile_enabled: true
  memory_char_limit: 1500   # Lebih ketat dari default 2200
  user_char_limit: 1000     # Lebih ketat dari default 1375
```

---

## 10. Monitoring Cache Hit Rate

Sayangnya Hermes belum expose cache hit rate langsung di UI. Cara indirect:

```bash
# Di DeepSeek API response, ada field:
# "prompt_cache_hit_tokens" dan "prompt_cache_miss_tokens"

# Cara cek via Hermes:
# 1. Aktifkan show_cost: true di config
# 2. Monitor biaya per turn
# 3. Kalau turn ke-2+ jauh lebih murah dari turn ke-1 → cache hit
# 4. Kalau semua turn sama mahal → ada prefix yang berubah

display:
  show_cost: true    # WAJIB untuk monitoring
```

---

## 11. Summary: Hal yang Lo Perlu Lakukan

1. **Cek SOUL.md** — tidak ada dynamic content → ✅ sudah oke di repo ini
2. **Jaga compression threshold** — naik dari 0.50 → 0.60-0.65
3. **Jangan edit SOUL.md terlalu sering** — cache bust = cost spike 1-2 turn
4. **Monitor biaya** — `show_cost: true` dan perhatikan pattern
5. **(Advanced)** Pisah konten statis dan dinamis di system prompt — statis duluan

**Yang TIDAK perlu dilakukan:**
- Install Reasonix (itu agent terpisah, bukan plugin Hermes)
- Ubah provider atau model
- Konfigurasi manual cache — DeepSeek handle otomatis

---

## 12. Catatan: Reasonix vs Hermes

Reasonix adalah **agent tersendiri** yang dibangun dari scratch untuk terminal dengan fokus prefix cache. **Bukan plugin** yang bisa dipasang di Hermes.

Yang bisa diadopsi dari filosofi Reasonix ke Hermes:
1. ✅ **Principle: stable system prompt** → implementasi via SOUL.md yang static
2. ✅ **Principle: cache-first cost control** → implementasi via compression threshold yang lebih tinggi
3. ❌ **Auto tool-call repair loop** → Hermes sudah punya retry mechanism sendiri
4. ❌ **Direct DeepSeek API optimization** → Hermes via OpenCode Go (indirect)

**Hermes kelebihan yang Reasonix tidak punya:**
- Skills system (25 skills)
- Memory persisten lintas session
- Telegram gateway
- Multi-profile
- Anti-hallucination framework
- Cronjob automation

Rekomendasi: **tetap pakai Hermes**, apply prinsip prefix cache dari Reasonix. Dua-duanya complementary, bukan exclusive.
