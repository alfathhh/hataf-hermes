# 15 — lean-ctx: Context Optimizer (Pengganti RTK yang Lebih Powerful)

Tujuan: hemat 89-99% token dengan lean-ctx — hybrid optimizer yang compress terminal output DAN file reads. Pengganti RTK yang lebih lengkap.

> Sumber:
> - [github.com/yvgude/lean-ctx](https://github.com/yvgude/lean-ctx)
> - [lib.rs/crates/lean-ctx](https://lib.rs/crates/lean-ctx)
> - [leanctx.com](https://leanctx.com)

---

## 1. Apa itu lean-ctx?

**lean-ctx** = Hybrid Context Optimizer. Single Rust binary, zero dependencies. Punya 2 mode:

| Mode | Fungsi | Savings |
|------|--------|---------|
| **Shell Hook** | Compress terminal output (90+ patterns) — mirip RTK tapi auto-inject | 60-90% |
| **MCP Server** | 46 tools: file read compression, AST parsing, cross-session cache | 89-99% |

```
Tanpa lean-ctx:
  agent → read_file(500 baris) → 500 baris masuk context = 2000+ token
  agent → git status → 30 baris = 300 token

Dengan lean-ctx:
  agent → read_file(500 baris) → AST extract relevant = 50 token
  agent → git status → shell hook compress = 30 token
  agent → re-read same file → cache hit = ~13 token
```

---

## 2. lean-ctx vs RTK — Kenapa lean-ctx Lebih Bagus

| Kriteria | RTK | lean-ctx |
|----------|-----|----------|
| Shell compression | 100+ patterns | 90+ patterns |
| Auto-inject | ❌ Butuh prefix `rtk [cmd]` | ✅ Shell hook otomatis |
| File read compression | ❌ Tidak bisa | ✅ AST-aware (tree-sitter, 18 bahasa) |
| Cross-session cache | ❌ Tidak | ✅ Re-read = ~13 token |
| MCP Server | ❌ Tidak | ✅ 46 tools |
| Total savings | 60-90% (terminal only) | 89-99% (terminal + files) |
| Plugin Hermes | ✅ `rtk-hermes` ready | ⚠️ Manual MCP config |

**Rekomendasi**: ganti RTK dengan lean-ctx. Dapet semua yang RTK bisa + file compression + caching.

---

## 3. Install lean-ctx

```bash
# Option A: cargo install (kalau punya Rust)
cargo install lean-ctx

# Option B: binary download dari GitHub releases
curl -L https://github.com/yvgude/lean-ctx/releases/latest/download/lean-ctx-x86_64-unknown-linux-gnu -o /usr/local/bin/lean-ctx
chmod +x /usr/local/bin/lean-ctx

# Verify
lean-ctx --version
```

---

## 4. Setup Mode 1: Shell Hook (ganti RTK)

```bash
# Tambah ke ~/.bashrc atau ~/.zshrc
eval "$(lean-ctx hook bash)"    # untuk bash
eval "$(lean-ctx hook zsh)"     # untuk zsh

# Reload
source ~/.bashrc
```

Setelah ini, **semua terminal command otomatis di-compress** tanpa prefix. Lo gak perlu ketik `lean-ctx git status` — cukup `git status` biasa.

**Kalau sebelumnya pake RTK**: hapus RTK setup dan ganti:

```bash
# Hapus RTK
pip uninstall rtk-hermes
# Hapus dari .bashrc kalau ada rtk eval

# Ganti dengan lean-ctx
eval "$(lean-ctx hook bash)"
```

---

## 5. Setup Mode 2: MCP Server (untuk Hermes)

Tambah di `~/.hermes/config.yaml` under section `mcp:`:

```yaml
mcp:
  lean-ctx:
    command: lean-ctx
    args: ["mcp-server"]
    # lean-ctx MCP server kasih 46 tools:
    # - ctx_read_file (AST-aware, compress)
    # - ctx_search (relevance scoring)
    # - ctx_memory (cross-session cache)
    # - ctx_summarize (condense large output)
    # - ... dan banyak lagi
```

Restart Hermes:
```bash
sudo systemctl restart hermes-agent
```

Verify MCP tools loaded:
```bash
hermes tools
# Harusnya ada tools baru dari lean-ctx
```

---

## 6. Gimana file read compression bekerja

### Tanpa lean-ctx (default Hermes read_file):
```
read_file("src/handler/order.go")
→ Return: seluruh file 500 baris = ~2500 token
```

### Dengan lean-ctx MCP `ctx_read_file`:
```
ctx_read_file("src/handler/order.go", task="understand CreateOrder handler")
→ tree-sitter parse Go AST
→ Extract: function signatures + CreateOrder body + imports
→ Return: 50 baris relevan = ~250 token (90% hemat)
```

### Re-read (cross-session cache):
```
ctx_read_file("src/handler/order.go", task="check error handling")
→ Cache HIT (file belum berubah)
→ Return: hanya delta yang relevan = ~13 token (99% hemat)
```

---

## 7. Stack lengkap token optimization untuk Hermes

```
┌──────────────────────────────────────────────────────┐
│ Layer 1: System prompt → DeepSeek Prefix Cache (90%) │
│          SOUL.md + tools + skills = stabil            │
├──────────────────────────────────────────────────────┤
│ Layer 2: Terminal output → lean-ctx Shell Hook (80%)  │
│          git, npm, docker, ls, dll                    │
├──────────────────────────────────────────────────────┤
│ Layer 3: File reads → lean-ctx MCP Server (90-99%)   │
│          Code files, configs, docs, PDF               │
├──────────────────────────────────────────────────────┤
│ Layer 4: Conversation → Hermes compression (var)     │
│          History lama di-summarize                     │
├──────────────────────────────────────────────────────┤
│ Layer 5: Smart routing → Simple turns ke cheap model │
│          "halo" → Qwen3.5 Plus (bukan K2.6)          │
└──────────────────────────────────────────────────────┘

Estimasi total: 90-95% reduction input token
```

---

## 8. Config Hermes: model kuat + lean-ctx + prefix cache

Untuk yang pake model kuat (K2.6 / DeepSeek V4 Pro) dan mau optimize max:

```yaml
# ~/.hermes/config.yaml — STRONG + OPTIMIZED

model:
  provider: opencode-go
  default: kimi-k2.6            # model terkuat di bundle

fallback_model:
  provider: opencode-go
  model: deepseek-v4-pro        # fallback kuat juga

smart_model_routing:
  enabled: true
  max_simple_chars: 160
  max_simple_words: 28
  cheap_model:
    provider: opencode-go
    model: deepseek-v4-flash

agent:
  reasoning_effort: high        # model kuat, biarkan reasoning deep

auxiliary:
  vision:
    provider: openrouter
    model: google/gemini-2.5-flash
  web_extract:
    provider: opencode-go
    model: qwen3.5-plus

# Prefix cache optimized
compression:
  enabled: true
  threshold: 0.65              # lebih jarang compress = prefix cache stabil
  summary_provider: opencode-go
  summary_model: deepseek-v4-flash

# lean-ctx MCP Server
mcp:
  lean-ctx:
    command: lean-ctx
    args: ["mcp-server"]

delegation:
  provider: opencode-go
  model: qwen3.6-plus

memory:
  memory_enabled: true
  user_profile_enabled: true
  memory_char_limit: 2200
  user_char_limit: 1375
```

---

## 9. Perbandingan: kapan pakai apa

| Use case | RTK | lean-ctx Shell | lean-ctx MCP | Prefix Cache |
|----------|-----|---------------|--------------|--------------|
| Terminal output compress | ✅ | ✅ | - | - |
| File read compress | - | - | ✅ | - |
| System prompt savings | - | - | - | ✅ |
| Cross-session cache | - | - | ✅ | ✅ (auto) |
| Zero-config | ✅ (plugin) | ✅ (hook) | ⚠️ (MCP setup) | ✅ (auto) |
| Hermes compatible | ✅ | ✅ | ✅ | ✅ |

---

## 10. Migration dari RTK ke lean-ctx

```bash
# Step 1: Uninstall RTK
pip uninstall rtk-hermes

# Hapus dari .bashrc kalau ada:
# eval "$(rtk hook)" atau alias rtk-related

# Step 2: Install lean-ctx
cargo install lean-ctx
# atau download binary

# Step 3: Setup shell hook
echo 'eval "$(lean-ctx hook bash)"' >> ~/.bashrc
source ~/.bashrc

# Step 4: Setup MCP di Hermes config
nano ~/.hermes/config.yaml
# Tambah:
# mcp:
#   lean-ctx:
#     command: lean-ctx
#     args: ["mcp-server"]

# Step 5: Restart
sudo systemctl restart hermes-agent

# Step 6: Verify
lean-ctx --version
hermes tools | grep ctx
```

---

## 11. Conflict check

```
lean-ctx Shell Hook + RTK = ⚠️ CONFLICT (double compress, jangan bareng)
lean-ctx Shell Hook + lean-ctx MCP = ✅ OK (beda layer)
lean-ctx MCP + RTK = ✅ OK (RTK = terminal, MCP = file reads)
lean-ctx + Prefix Cache = ✅ OK (beda layer entirely)
lean-ctx + Hermes compression = ✅ OK (beda layer)

RULE: Pilih SATU untuk terminal compression (RTK ATAU lean-ctx hook, jangan dua-duanya)
```

---

## 12. Yang gw belum yakin

- **lean-ctx MCP server compatibility exact dengan Hermes**: Hermes support MCP, dan lean-ctx expose MCP tools — tapi gw belum verify apakah semua 46 tools langsung jalan tanpa config tambahan. Test dulu setelah setup.
- **Performance overhead shell hook**: lean-ctx hook tambah ~1-5ms per command execution. Untuk interactive use gak kerasa. Untuk batch scripting mungkin terasa.
- **AST parsing accuracy**: tree-sitter support 18 bahasa, tapi untuk bahasa niche atau file format aneh mungkin fallback ke raw text.

---

## 🔗 Sumber

- [github.com/yvgude/lean-ctx](https://github.com/yvgude/lean-ctx) — repo resmi
- [lib.rs/crates/lean-ctx](https://lib.rs/crates/lean-ctx) — Rust crate
- [leanctx.com](https://leanctx.com) — website resmi
- [skillsmp.com/lean-ctx](https://skillsmp.com/skills/cses8-lean-ctx-skills-lean-ctx-skill-md) — skill spec
