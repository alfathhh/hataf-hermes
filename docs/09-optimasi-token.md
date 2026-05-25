# 09 — Optimasi Token (Hemat Banget)

Tujuan: bikin Hermes lo seoptimal mungkin secara biaya, tanpa ngorbanin quality di task yang penting.

> Sumber: [Tips & Best Practices](https://hermes-agent.nousresearch.com/docs/guides/tips), [Configuration docs](https://hermes-agent.nousresearch.com/docs/user-guide/configuration), [Memory docs](https://hermes-agent.nousresearch.com/docs/user-guide/features/memory).

---

## 1. Mental model: di mana token "bocor"

Sebelum optimasi, ngerti dulu sumber konsumsi token Hermes per turn:

| Sumber | Estimasi % | Bisa dihemat? |
|---|---|---|
| System prompt (SOUL.md, identity) | 5-15% | Sedikit — keep SOUL.md tight |
| Memory (MEMORY.md + USER.md) | ~10% | Tight cap (default 1.300 token kombinasi) |
| Skills list (level 0) | ~10% | Conditional skills (`requires_toolsets`) |
| Tool descriptions | 15-25% | Disable toolset yang gak kepake |
| Conversation history | bervariasi | Compression aktif, `/new` saat ganti topic |
| User message | bervariasi | User behavior |
| Model output | bervariasi | Concise style (di SOUL.md) |

Estimasi di atas **kasar**, gw nggak punya benchmark resmi. Kuncinya: liat `/usage` di tiap session untuk angka real lo.

---

## 2. Knob optimasi per kategori

### A. Pakai cheap model untuk turn ringan (smart routing)

Setup udah ada di `examples/config.yaml`:

```yaml
smart_model_routing:
  enabled: true
  max_simple_chars: 160
  max_simple_words: 28
  cheap_model:
    provider: opencode-go
    model: qwen3.5-plus
```

Cara kerja:

- Turn pendek + sederhana ("halo", "convert ini ke JSON", "translate this") → cheap_model
- Turn kompleks (code, debug, multi-step) → primary model (DeepSeek V4 Flash)

Estimasi penghematan: 30-60% kalau pattern lo banyak quick Q&A. **Belum gw verifikasi real**, tergantung dari distribusi turn lo.

> ⚠️ **Trade-off**: cheap model lebih gampang halu. Untuk turn faktual sensitif, akan tetep ke primary (smart router checks for "code/tool/debug heavy" cues). Tapi kalau ragu, force `/model deepseek-v4-flash` untuk turn tertentu.

### B. Compression (auto-summarize history lama)

```yaml
compression:
  enabled: true
  threshold: 0.50          # compress saat 50% context limit
  target_ratio: 0.2
  protect_last_n: 20

auxiliary:
  compression:
    provider: opencode-go
    model: qwen3.5-plus    # model untuk summarize history
```

Yang di-compress: middle section conversation (yang lama). Awal session + recent turns dipreserve.

Effect:
- Sebelum: conversation 50K token → kirim 50K tiap turn
- Sesudah: conversation 50K token → di-compress jadi summary 10K, total kirim ~10-20K

Trade-off:
- Detail lama hilang (cuma ada summary)
- Bayar 1x summary call (oleh model murah)

Kalau lo butuh full detail lama, jangan pake compression — pake `/new` saat topic ganti.

### C. Reasoning effort

```yaml
agent:
  reasoning_effort: ""    # default = medium
```

Opsi:

| Level | Output thinking tokens | Pakai untuk |
|---|---|---|
| `xhigh` | Paling banyak | Task super kompleks (1-2x sehari OK) |
| `high` | Banyak | Code review serius, debug nontrivial |
| `medium` | Default | Use case umum |
| `low` | Sedikit | Quick Q&A |
| `minimal` | Hampir gak ada | Script-y task |
| `none` | Disabled | Pure execution (translate, format) |

Switch on the fly: `/reasoning low` di TUI.

> Reasoning tokens itu "thinking" yang model lakuin sebelum jawab. DeepSeek V4 Flash thinking mode bisa pake 10-100x output token. Pakai sesuai kebutuhan.

### D. Memory cap (jangan dilonggarkan)

```yaml
memory:
  memory_char_limit: 2200      # ~800 token, default
  user_char_limit: 1375        # ~500 token, default
```

Cap ini **fitur**. Memaksa konsolidasi. Kalau lo longgarkan jadi 10.000 char, system prompt lo overhead +3.000 token PER TURN.

Default OK untuk 95% kasus. **Resist temptation** untuk naikin.

### E. Quick commands (zero LLM)

```yaml
quick_commands:
  status:
    type: exec
    command: systemctl status hermes-agent
  disk:
    type: exec
    command: df -h /
```

`/status`, `/disk`, dst — **gak burn token sama sekali**. Pure shell.

Cocok untuk:
- Status check sehari-hari
- Update agent (`/hermes_update`)
- Quick lookup (cek log, cek disk, cek uptime)

Dari Telegram = ops dashboard portable.

### F. Skills progressive disclosure

Skills cuma load level 0 (meta-list) di system prompt. Full content dimuat **saat dipanggil**.

Implication:
- Punya 50 skill = level 0 ~3-5K token (bukan 50x full skill)
- Skill yang gak relevan ke turn ini = gak bayar token-nya

Untuk ekstra hemat, pake conditional skill:

```yaml
# Di SKILL.md frontmatter
metadata:
  hermes:
    requires_toolsets: [terminal]    # cuma muncul kalau terminal aktif
    fallback_for_toolsets: [web]      # cuma muncul kalau web TIDAK aktif
```

### G. Auxiliary models — selalu pake yang murah

```yaml
auxiliary:
  vision:
    provider: openrouter
    model: google/gemini-2.5-flash    # bukan gpt-4o, kecuali kritis
  web_extract:
    provider: opencode-go
    model: qwen3.5-plus               # bukan deepseek
  approval:
    provider: opencode-go
    model: qwen3.5-plus
  compression:
    provider: opencode-go
    model: qwen3.5-plus               # summarize history — murah, cukup
```

Auxiliary di-call **frequent**. Murah-cepat = win.

### H. Disable toolset yang gak kepake

```bash
hermes tools
```

Interactive: disable toolset yang gak relevan ke use case lo. Setiap tool description = beberapa ratus token di system prompt.

Kalau lo gak pake browser pernah, disable browser toolset. Hemat ~1.000-2.000 token.

### I. Delegation untuk subtask murah

```yaml
delegation:
  provider: opencode-go
  model: qwen3.6-plus
```

Saat agent spawn subagent (`delegate_task`), subagent jalan dengan model ini, bukan primary. Cocok untuk subtask narrow scope (research 1 topic, summarize 1 file).

### J. Context length set tepat

```yaml
model:
  context_length: 1000000     # untuk DeepSeek V4 Flash 1M
```

Hermes pakai ini untuk hitung kapan compression fire (threshold = 50% dari ini).

Set akurat — terlalu kecil = compression terlalu sering (rusak detail), terlalu besar = compression telat (mungkin overflow).

### K. Reasoning display (jangan ditampilkan)

```yaml
display:
  show_reasoning: false       # default false, KEEP false
```

Reasoning tokens udah expensive sendiri, gak perlu juga dirender di TUI lo (token rendering = visual UI cost, bukan API cost — tapi tetep noise).

---

## 3. Strategi per use case

### Strategi 1: Daily personal use, modest budget

```yaml
# Primary: DeepSeek V4 Flash non-thinking
model:
  default: deepseek-v4-flash
agent:
  reasoning_effort: medium

# Cheap path aktif untuk Q&A pendek
smart_model_routing:
  enabled: true
  cheap_model:
    provider: opencode-go
    model: qwen3.5-plus

# Compression aggressive
compression:
  enabled: true
  threshold: 0.40
  target_ratio: 0.2
  protect_last_n: 20

auxiliary:
  compression:
    provider: opencode-go
    model: qwen3.5-plus

# Auxiliary semua murah
auxiliary:
  vision: { provider: openrouter, model: google/gemini-2.5-flash }
  web_extract: { provider: opencode-go, model: qwen3.5-plus }
```

Estimasi cost: $5-15/bulan untuk personal heavy use (kira-kira, perlu cek meter sendiri).

### Strategi 2: Heavy coding, context kepenuhan sering

```yaml
# Primary: thinking mode
agent:
  reasoning_effort: high

# Disable smart routing (semua coding context kompleks)
smart_model_routing:
  enabled: false

# Compression LATE — biar detail kepertahanin
compression:
  threshold: 0.65

# Pakai DeepSeek V4 Flash 1M context, jangan compress kalau gak perlu
```

### Strategi 3: Pure cron ops (background scrap & monitor)

```yaml
# Primary: model murah aja (cron prompt biasa simple)
model:
  provider: opencode-go
  default: qwen3.6-plus

# Fallback ke yang lebih kuat kalau perlu
fallback_model:
  provider: custom
  model: deepseek-v4-flash
  base_url: https://api.deepseek.com/v1

# Reasoning rendah
agent:
  reasoning_effort: low
```

---

## 4. Monitor real cost

```bash
hermes config | grep show_cost      # pastikan show_cost: true
```

Di TUI status bar: real-time $ estimate.

Per-session:

```
/usage                              # token breakdown + estimated cost
/insights --days 7                  # cost summary 7 hari terakhir
```

Cost **estimate** dari Hermes berdasarkan token + pricing meta. Kalau lo pakai custom endpoint, set pricing manually di `config.yaml` kalau mau show_cost akurat.

---

## 5. Anti-pattern (yang BIKIN BOROS)

❌ **Memory char_limit dilonggarkan jadi 10.000+**: linier naikin biaya tiap turn

❌ **Skill terlalu banyak (50+) tanpa conditional**: skills_list level 0 membengkak

❌ **Threshold compression 0.90+**: compression telat, overflow risk, dan saat fire jadi summary konteks tipis

❌ **Reasoning effort = xhigh selalu**: tiap turn 3-10x token output

❌ **Custom endpoint tanpa context_length set**: Hermes default ke 128K, compression-nya jadi salah timing

❌ **Pakai `delegate_task` untuk task simple**: setiap subagent ada overhead system prompt sendiri

❌ **Primary model = mahal-mahalan kayak Claude Opus**: kalau task lo 90% Q&A simple, mahal di yang gak kepake

---

## 6. Checklist cepat: apakah lo udah optimal?

- [ ] `smart_model_routing.enabled: true` dengan cheap_model murah
- [ ] `compression.enabled: true`, threshold 0.40-0.60
- [ ] `auxiliary.*` semua pake model murah (Gemini Flash atau Qwen Plus)
- [ ] `memory.*_char_limit` default (jangan dilonggarkan)
- [ ] `quick_commands` setup untuk ops check rutin
- [ ] Toolset gak kepake = di-disable (`hermes tools`)
- [ ] Skills conditional (kalau punya banyak)
- [ ] `display.show_cost: true` (lo monitor sendiri)
- [ ] `agent.reasoning_effort: ""` (default medium) atau low untuk daily use
- [ ] Primary model = DeepSeek V4 Flash non-thinking (cheap by design)
- [ ] Fallback configured (no manual switch saat rate-limit)

---

## 7. Yang gw belum yakin

- **Estimasi penghematan smart_model_routing**: gw bilang 30-60%, ini **kasar**, perlu lo benchmark di pattern aktual lo. Bisa lebih tinggi/rendah tergantung distribusi turn.
- **Cost real DeepSeek V4 Flash thinking mode**: thinking tokens dihitung sebagai output token. Per [api-docs.deepseek.com](https://api-docs.deepseek.com/), pricing thinking output spesifik bisa beda — cek halaman pricing resmi.
- **OpenRouter pricing untuk Gemini Flash**: rate OpenRouter punya markup vs direct Google. Untuk **paling murah**, pake direct Google API key (kalau punya). Hermes support set `provider: google` dengan `GOOGLE_API_KEY`.

---

## Lanjut

→ [10 — Mengajarkan Hermes](10-mengajarkan-hermes.md): bikin agent lo grow over time. Ini bagian terakhir, kunci jangka panjang.
