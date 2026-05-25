# 14 — Prefix Cache Optimization + lean-ctx

Tujuan: hemat biaya API DeepSeek 60-90% dengan memanfaatkan **KV Cache / Prefix Caching** dan **lean-ctx** (context compression tool).

> Sumber:
> - [DeepSeek KV Cache docs](https://api-docs.deepseek.com/guides/kv_cache)
> - [DeepSeek Context Caching on Disk announcement](https://api-docs.deepseek.com/news/news0802)
> - [lean-ctx — Hybrid Context Optimizer](https://github.com/yvgude/lean-ctx)
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

## 2. Prinsip Inti: "Prefix Stability"

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

compression:
  enabled: true
  threshold: 0.65    # Lebih tinggi dari default = compress lebih jarang
                     # Kenapa: setiap compression = restart conversation →
                     #          cache miss untuk history
  target_ratio: 0.2
  protect_last_n: 20

# Model untuk summarize ditaruh di auxiliary.compression
auxiliary:
  compression:
    provider: opencode-go
    model: qwen3.5-plus

# Memory jaga tetap pendek = less prefix disruption
memory:
  memory_enabled: true
  user_profile_enabled: true
  memory_char_limit: 1500        # Lebih ketat dari default 2200
  user_char_limit: 1000          # Lebih ketat dari default 1375
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

## 6. lean-ctx — Compress Tool Output (Complementary)

Prefix cache menghemat token di **system prompt yang berulang**.
lean-ctx menghemat token di **tool output (shell + file reads)**.

Keduanya **complementary** — bukan pilih salah satu.

### Apa itu lean-ctx

[lean-ctx](https://github.com/yvgude/lean-ctx) adalah Rust binary (single binary, zero dependencies) yang compress shell output + file reads sebelum sampai ke LLM.

**Fitur utama:**
- Shell hook: transparent compress CLI output (89-99% reduction)
- MCP server: 46+ tools untuk context management
- 90+ compression patterns untuk berbagai command
- Tree-sitter AST parsing untuk 18 bahasa (compress code cerdas)
- Cross-session memory (CCP) — persistent knowledge
- Cached re-reads turun ke ~13 token

### Install lean-ctx

```bash
# Opsi 1: Cargo
cargo install lean-ctx

# Opsi 2: Install script
curl -fsSL https://leanctx.com/install.sh | bash

# Verify
lean-ctx --version
```

### Kenapa lean-ctx melengkapi prefix cache

| Layer | Apa yang dihemat | Tool |
|-------|------------------|------|
| System prompt (prefix) | Token dari SOUL.md + tools + skills yang berulang tiap turn | DeepSeek KV Cache (otomatis) |
| Tool output | Token dari shell output + file reads yang masuk context | lean-ctx |
| Conversation history | Token dari history yang sudah lewat | Hermes compression |

**Tanpa lean-ctx**: `git log --oneline -50` → 500+ token masuk context.
**Dengan lean-ctx**: output di-compress → ~50 token masuk context.

### Impact gabungan

```
Tanpa optimasi:
  System prompt: 4000 token × full price × 50 turns = mahal
  Tool output: 200-500 token per call × banyak calls = mahal
  Total: $$$$

Dengan prefix cache + lean-ctx:
  System prompt: 4000 token × cache hit price (10x murah) × 50 turns = murah
  Tool output: 50-100 token per call (compressed) = murah
  Total: $  (70-90% saving)
```

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
  Total = $0.00021 + $0.001029 ≈ $0.001239/session

Penghematan prefix cache saja: ~88% untuk sistem prompt portion
```

Tambah lean-ctx (compress tool output 89-99%):
- **Estimasi total penghematan: 70-90%** dari total biaya input token

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

## 9. Config Khusus: Prefix-Cache + lean-ctx Optimized

Full config ada di: [`examples/config-prefix-cache.yaml`](../examples/config-prefix-cache.yaml)

Key settings yang berbeda dari default:

```yaml
model:
  provider: opencode-go
  default: kimi-k2.6
  api_mode: chat_completions

# Threshold compression lebih tinggi = prefix tetap stabil lebih lama
compression:
  enabled: true
  threshold: 0.65         # Default 0.50 → naik ke 0.65
  target_ratio: 0.2
  protect_last_n: 20

# Model untuk summarize
auxiliary:
  compression:
    provider: opencode-go
    model: qwen3.5-plus

# Memory tetap tight = kurangi prefix disruption
memory:
  memory_enabled: true
  user_profile_enabled: true
  memory_char_limit: 1500   # Lebih ketat dari default 2200
  user_char_limit: 1000     # Lebih ketat dari default 1375

# Explicit caching TTL
prompt_caching:
  cache_ttl: 5m
```

---

## 10. Monitoring Cache Hit Rate

Cara indirect untuk monitor cache performance:

```bash
# Di DeepSeek API response, ada field:
# "prompt_cache_hit_tokens" dan "prompt_cache_miss_tokens"

# Cara cek via Hermes:
# 1. Aktifkan show_cost: true di config
# 2. Monitor biaya per turn
# 3. Kalau turn ke-2+ jauh lebih murah dari turn ke-1 → cache hit
# 4. Kalau semua turn sama mahal → ada prefix yang berubah
```

Config wajib:
```yaml
display:
  show_cost: true    # WAJIB untuk monitoring
```

---

## 11. Summary: Hal yang Lo Perlu Lakukan

1. **Cek SOUL.md** — tidak ada dynamic content → ✅ sudah oke di repo ini
2. **Jaga compression threshold** — naik dari 0.50 → 0.65
3. **Install lean-ctx** — compress tool output 89-99%
4. **Jangan edit SOUL.md terlalu sering** — cache bust = cost spike 1-2 turn
5. **Monitor biaya** — `show_cost: true` dan perhatikan pattern

**Yang TIDAK perlu dilakukan:**
- Ubah provider atau model
- Konfigurasi manual cache — DeepSeek handle otomatis
- Install tool/agent terpisah selain lean-ctx

---

## 12. Quick Setup (copy-paste)

```bash
# 1. Copy config prefix-cache optimized
cp examples/config-prefix-cache.yaml ~/.hermes/config.yaml

# 2. Install lean-ctx
cargo install lean-ctx
# atau:
# curl -fsSL https://leanctx.com/install.sh | bash

# 3. Verify lean-ctx
lean-ctx --version

# 4. Restart Hermes
sudo systemctl restart hermes-agent

# 5. Monitor — cek cost per turn, turn ke-2+ harus lebih murah
hermes -q "halo"
hermes -q "apa kabar"
# Lihat: apakah turn kedua jauh lebih murah?
```

---

## 13. Stack Optimal untuk Cost Minimum

```
┌─────────────────────────────────────────────────────┐
│  lean-ctx          → compress tool output 89-99%    │
│  prefix cache      → hemat system prompt 88%        │
│  memory ketat      → prefix stabil, cache friendly  │
│  SOUL.md static    → no cache bust                  │
│  smart routing     → turn simpel ke cheap model     │
│  compression 0.65  → jarang compress, cache stable  │
└─────────────────────────────────────────────────────┘
  → Total estimasi saving: 70-90% vs default config
```

---

## 🔗 Sumber

- [DeepSeek KV Cache docs](https://api-docs.deepseek.com/guides/kv_cache) — cara kerja prefix cache
- [lean-ctx](https://github.com/yvgude/lean-ctx) — context compression tool
- [leanctx.com](https://leanctx.com) — official site
- [KV-Cache Aware Prompt Engineering](https://ankitbko.github.io/blog/2025/08/prompt-engineering-kv-cache/) — deep dive teknis
