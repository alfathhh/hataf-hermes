# 13 — RTK (Rust Token Killer) — Hemat Token 60-90%

Tujuan: install RTK biar terminal output Hermes di-compress sebelum masuk context LLM. Hemat quota OpenCode Go lo secara signifikan.

> Sumber: [github.com/rtk-ai/rtk](https://github.com/rtk-ai/rtk), [pypi.org/project/rtk-hermes](https://pypi.org/project/rtk-hermes), [mer.vin explainer](https://mer.vin/2026/05/cut-ai-coding-token-costs-80-with-rtk-cli-proxy-how-rust-token-killer-works/).

---

## 1. Apa itu RTK

**RTK** = CLI proxy yang duduk di antara Hermes dan terminal lo. Setiap kali Hermes jalanin shell command (`git status`, `npm install`, `ls -la`, dll), RTK **filter dan compress output-nya** sebelum masuk ke context window LLM.

```
Tanpa RTK:
  Hermes → jalanin `git status` → output 50 baris masuk context → 500 token

Dengan RTK:
  Hermes → jalanin `rtk git status` → RTK filter → output 5 baris masuk context → 50 token
```

**Klaim**: reduce token consumption 60-90% di terminal commands.

---

## 2. Kenapa lo butuh ini

Lo pake OpenCode Go (quota-limited). Terminal output = **sumber utama token bloat** di agent:

| Command | Output tanpa RTK | Output dengan RTK | Token saved |
|---------|-----------------|-------------------|-------------|
| `git status` | 30-50 baris | 3-5 baris | ~80% |
| `npm install` | 100+ baris (progress bar, deps tree) | 2-5 baris (errors only) | ~95% |
| `docker ps` | Header + semua container | Compact summary | ~70% |
| `ls -la` (folder besar) | 200+ baris | Filtered to relevant | ~90% |
| `pytest` output | 50+ baris per test | Summary + failures only | ~85% |

Semua ini **langsung nge-save quota Kimi K2.6 lo** yang cuma 1,150 credits.

---

## 3. Install RTK

### Linux / macOS / WSL

```bash
curl -fsSL https://raw.githubusercontent.com/rtk-ai/rtk/master/install.sh | bash
```

### Cargo (kalau lo udah punya Rust)

```bash
cargo install rtk
```

### Verify

```bash
rtk --version
which rtk
```

---

## 4. Install plugin Hermes (`rtk-hermes`)

Plugin ini auto-rewrite terminal commands Hermes lewat RTK.

```bash
pip install rtk-hermes
```

### Cara kerja plugin:

```
Hermes mau jalanin: git status
Plugin intercept → rewrite ke: rtk git status
RTK filter output → compressed version masuk context
```

Lo gak perlu ngapa-ngapain manual — plugin handle otomatis.

---

## 5. Verify RTK jalan

### Test manual (tanpa Hermes dulu)

```bash
# Tanpa RTK
git log --oneline -20
# Output: 20 baris

# Dengan RTK
rtk git log --oneline -20
# Output: compressed (mungkin 5-10 baris yang paling relevan)
```

### Test di Hermes

```bash
hermes -q "jalanin git status di folder ini"
# Cek: apakah output yang Hermes terima sudah compressed?
# Liat di display.tool_progress apakah command udah di-prefix rtk
```

---

## 6. Supported commands (100+)

RTK punya filter rules untuk 100+ common commands:

| Category | Commands |
|----------|----------|
| Git | `git status`, `git log`, `git diff`, `git branch` |
| Node.js | `npm install`, `npm test`, `yarn`, `pnpm` |
| Python | `pip install`, `pytest`, `python -m` |
| Docker | `docker ps`, `docker logs`, `docker build` |
| System | `ls`, `find`, `df`, `ps`, `top` |
| Build | `make`, `cargo build`, `go build`, `tsc` |
| K8s | `kubectl get`, `kubectl describe`, `kubectl logs` |
| Misc | `curl`, `wget`, `cat` (large files) |

Kalau command gak di-support: RTK pass-through (output sama kayak tanpa RTK, no harm).

---

## 7. Config RTK (optional, default OK)

```bash
# Buka config
rtk config

# Atau edit langsung
nano ~/.config/rtk/config.toml
```

Default settings biasanya udah optimal. Tweak kalau lo nemu case di mana RTK terlalu aggressive filter.

---

## 8. Troubleshoot

| Problem | Fix |
|---------|-----|
| `rtk: command not found` | Install belum kelar. Cek PATH: `echo $PATH`. Restart terminal. |
| Output terlalu di-filter (info penting hilang) | `rtk config` → adjust filter rules, atau bypass: `command` langsung tanpa `rtk` |
| Plugin `rtk-hermes` gak ke-load | Cek: `pip show rtk-hermes`. Restart Hermes. |
| Hermes masih verbose output | Plugin mungkin gak aktif. Cek Hermes plugin list: `hermes plugins list` |

---

## 9. Kapan RTK GAK berguna

- **Vision tasks** (gambar, screenshot) — RTK cuma filter teks terminal
- **Web search/extract** — output dari web tool bukan terminal command
- **Chat biasa** (Q&A tanpa tool call) — gak ada terminal command yang dipanggil
- **Compression already active** — kalau conversation udah di-compress, RTK savings di-stack tapi diminishing

RTK paling berguna kalau Hermes lo **sering pake terminal** (coding, devops, scraping via CLI, git ops).

---

## 10. Impact ke setup Balanced lo

Dengan RTK aktif:

| Sebelum RTK | Sesudah RTK |
|-------------|-------------|
| Terminal commands burn 200-500 token/call | ~50-100 token/call |
| K2.6 quota 1,150 habis di ~20-30 complex coding sessions | Bisa stretch ke ~50-80 sessions |
| Smart routing save 60-70% turns | Smart routing + RTK save 80-90% total token |

Ini **complementary** dengan smart_model_routing — routing hemat di level "turn mana ke model mana", RTK hemat di level "berapa token per turn".

---

## 11. Yang gw belum yakin

- **Exact savings di setup lo**: klaim 60-90% dari repo RTK. Gw belum benchmark sendiri di Hermes + OpenCode Go. Test sendiri pake `/usage` sebelum dan sesudah enable RTK.
- **`rtk-hermes` plugin maintenance status**: gw nemu di PyPI tapi belum verify siapa maintainer, last update kapan, dan apakah compatible dengan Hermes version terbaru. Cek `pip show rtk-hermes` untuk detail.
- **Apakah ada edge case di mana RTK bikin agent "salah tangkep" output**: secara teori bisa — kalau error message penting ke-filter. Monitor beberapa hari pertama.

---

## 12. Quick setup (copy-paste)

```bash
# 1. Install RTK
curl -fsSL https://raw.githubusercontent.com/rtk-ai/rtk/master/install.sh | bash

# 2. Verify
rtk --version

# 3. Install Hermes plugin
pip install rtk-hermes

# 4. Restart Hermes gateway
sudo systemctl restart hermes-agent

# 5. Test
hermes -q "git status"
# Cek apakah output lebih compact dari biasanya
```

Done. Gak perlu edit config.yaml atau .env — plugin auto-activate.

---

## 🔗 Sumber

- [github.com/rtk-ai/rtk](https://github.com/rtk-ai/rtk) — repo resmi, install instructions
- [pypi.org/project/rtk-hermes](https://pypi.org/project/rtk-hermes/) — plugin Hermes
- [mer.vin explainer](https://mer.vin/2026/05/cut-ai-coding-token-costs-80-with-rtk-cli-proxy-how-rust-token-killer-works/) — how RTK works internally
- [wavespeed.ai](https://wavespeed.ai/blog/posts/what-is-rtk-token-efficiency/) — context on token efficiency
