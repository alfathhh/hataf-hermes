# 08 — Telegram Gateway

Tujuan: pasang Hermes lo sebagai bot Telegram, lengkap dengan voice message, attachment, dan secure access.

> Sumber: [Telegram setup docs](https://hermes-agent.nousresearch.com/docs/user-guide/messaging/telegram), [Team Telegram Assistant tutorial](https://hermes-agent.nousresearch.com/docs/guides/team-telegram-assistant).

---

## 1. Apa yang lo dapet

Setelah setup, lo bisa:

- Chat sama Hermes dari HP / Telegram di mana aja
- Kirim voice message → auto-transkrip via Whisper → agent jawab
- Kirim file PDF / gambar → agent baca/analisa
- Receive cronjob results (daily brief, alert, dll)
- Approve/deny dangerous command via reply chat
- Authorization per user ID (cuma lo / tim lo yang bisa pake)

---

## 2. Prasyarat

- Telegram account (HP)
- Hermes terinstall + minimal 1 LLM provider configured (doc 01-02)
- Optional: `ffmpeg` (untuk voice message), `faster-whisper` (untuk STT lokal gratis)

---

## 3. Bikin bot di BotFather

### Step 1: Open BotFather

Buka [t.me/BotFather](https://t.me/BotFather) di Telegram.

### Step 2: `/newbot`

```
You: /newbot
BotFather: Alright, a new bot. How are we going to call it?
You: Hermes Hataf
BotFather: Good. Now let's choose a username for your bot.
You: hataf_hermes_bot
```

Username **harus** end dengan `bot` dan unique global.

### Step 3: Save token

BotFather kasih token kayak:

```
123456789:ABCdefGHIjklMNOpqrSTUvwxYZabc12345
```

**Simpan baik-baik** — siapa pun yang punya token ini bisa kontrol bot lo.

### Step 4 (opsional, recommended): customize

Masih di BotFather:

```
/setdescription      # "Personal AI assistant. Honest, no hallucination."
/setabouttext        # tampil di profile bot
/setuserpic          # upload avatar
/setcommands         # define menu /commands

# Untuk /setcommands, paste:
new - Mulai conversation baru
help - Bantuan
sethome - Set chat ini sebagai home channel cron
status - Cek status server
disk - Cek disk usage
```

---

## 4. Privacy mode (PENTING untuk grup)

Ini **sumber confusion paling sering**. Defaultnya: privacy mode ON.

| Privacy ON | Privacy OFF |
|---|---|
| Bot lihat: pesan dengan `/`, reply ke pesan bot, member events | Bot lihat: SEMUA pesan |

Buat **DM (chat 1-on-1)**: privacy mode gak relevan, bot dapat semua pesan lo.

Buat **group chat**: kalau lo mau bot ikut diskusi natural (gak harus selalu @mention), matikan privacy mode.

### Cara matiin

```
BotFather → /mybots → [pilih bot] → Bot Settings → Group Privacy → Turn off
```

> ⚠️ **Setelah ubah privacy**: lo HARUS **remove dan re-add bot** ke group yang udah ada. Telegram cache state ini saat bot join.

Alternatif: bikin bot jadi **admin** di group. Admin bots dapat semua pesan terlepas dari privacy setting.

---

## 5. Cari user ID lo

Hermes pakai numeric Telegram ID untuk authorize, **bukan username**.

Cara cari:

- DM ke [@userinfobot](https://t.me/userinfobot) — instant balas user ID lo
- Atau: [@get_id_bot](https://t.me/get_id_bot)

Save number-nya (kayak `123456789`).

---

## 6. Configure Hermes

### Cara 1: Wizard interaktif (recommended buat pemula)

```bash
hermes gateway setup
```

Pilih **Telegram**. Wizard nanya:
- Bot token → paste
- Allowed user IDs → user ID lo (comma-separated kalau multi-user)
- Mau set home channel? → optional, bisa diset nanti dengan `/sethome`

Wizard auto-write ke `.env` dan `config.yaml`.

### Cara 2: Manual

Tambah ke `~/.hermes/.env`:

```bash
TELEGRAM_BOT_TOKEN=123456789:ABCdefGHIjklMNOpqrSTUvwxYZabc12345
TELEGRAM_ALLOWED_USERS=123456789

# Optional: home channel untuk delivery cronjob (bisa diset belakangan via /sethome)
# TELEGRAM_HOME_CHANNEL=123456789       # Untuk DM, sama dengan user ID
# TELEGRAM_HOME_CHANNEL=-1001234567890  # Untuk group, mulai -100xxx
```

---

## 7. Start gateway

### Foreground (testing)

```bash
hermes gateway
```

Bot harusnya online dalam beberapa detik. Buka chat di Telegram, kirim `halo`. Harus dapet balasan.

### Background (production)

Install sebagai service:

```bash
# User service (jalan saat user lo login)
hermes gateway install

# System service (jalan saat boot — recommended di VPS)
sudo hermes gateway install --system

# Start
sudo systemctl start hermes-agent          # kalau system service
# atau
systemctl --user start hermes-agent        # kalau user service

# Cek status
hermes gateway status
journalctl -u hermes-agent -f             # tail logs
```

---

## 8. Test checklist

| Test | Cara | Expected |
|---|---|---|
| Bot online | Send `/start` di chat | Bot reply welcome |
| Authorization | Minta temen lo (tanpa ID di allowlist) chat bot | Bot reply "unauthorized" atau silent |
| Pesan biasa | "halo, perkenalkan dirimu" | Reply sesuai SOUL.md (anti-halu, gak filler) |
| Voice message | Record voice di Telegram, kirim | Auto-transkrip, agent reply text |
| File attachment | Kirim PDF | Agent acknowledge, mulai baca / minta klarifikasi |
| Slash command | `/new` | Reset session |
| Cronjob delivery | `/cron add "30s" "test, kirim ke Telegram"` | Tunggu 30 detik, dapet pesan |

Kalau ada yang gagal, lihat troubleshoot di section 11.

---

## 9. Voice message setup

### Incoming (lo kirim voice → bot transkrip)

3 pilihan STT (speech-to-text):

| Backend | Setup | Pricing | Best for |
|---|---|---|---|
| **local** (faster-whisper) | `pip install faster-whisper` | Gratis | Privacy-first, server lo punya CPU/RAM |
| **groq** | `GROQ_API_KEY=...` | Murah, cepat (mungkin gratis tier) | Speed prioritas |
| **openai** | `VOICE_TOOLS_OPENAI_KEY=...` | Per-minute | Kalau udah pake OpenAI ecosystem |

Set di `config.yaml`:

```yaml
stt:
  provider: local             # atau: groq / openai
  local:
    model: small              # tiny | base | small | medium | large-v3
```

> Catatan: model `large-v3` paling akurat tapi butuh RAM 10 GB+. Untuk VPS kecil, `small` (~2 GB) sweet spot.

### Outgoing (bot bales pake voice)

Kalau lo mau bot bales pake voice (TTS):

```yaml
tts:
  provider: edge              # gratis, default. atau: openai / elevenlabs
  edge:
    voice: "id-ID-ArdiNeural"  # voice Indonesia (cek list edge-tts untuk opsi lain)
```

Pakai `/voice tts` di chat untuk toggle.

> Edge TTS output MP3, perlu `ffmpeg` untuk convert ke voice bubble Telegram (Opus). Install: `sudo apt install ffmpeg`.

---

## 10. Group chat

### Allowed user di group

`TELEGRAM_ALLOWED_USERS` tetep berlaku di group — cuma user yang di allowlist yang bisa trigger bot.

### Trigger style

| Privacy ON | Privacy OFF (atau bot admin) |
|---|---|
| @mention bot atau reply ke pesan bot | Bot lihat semua pesan; bisa ngerespon natural |

### Per-user session di group

Default `group_sessions_per_user: true` (di `config.yaml`):

- Tiap user di group punya **conversation thread sendiri** dengan bot
- User A dan B gak bisa nyampurin context

Kalau lo mau group sebagai 1 collaborative session (semua user share context):

```yaml
group_sessions_per_user: false
```

> Caveat: shared session = shared cost dan shared confusion. Pake yang `true` aja kecuali use case spesifik.

---

## 11. Troubleshoot

| Problem | Penyebab | Fix |
|---|---|---|
| Bot gak respond sama sekali | Token salah / gateway gak jalan | Cek `journalctl -u hermes-agent -f`, verify token |
| Bot reply "unauthorized" | User ID lo gak di allowlist | `hermes config set TELEGRAM_ALLOWED_USERS your_id_here` |
| Bot gak dengar pesan group | Privacy mode ON | Disable di BotFather, **remove + re-add bot ke group** |
| Voice gak ke-transkrip | STT belum installed atau API key salah | `pip install faster-whisper` atau cek key Groq/OpenAI |
| Voice reply muncul sebagai file, bukan voice bubble | `ffmpeg` gak terinstall | `sudo apt install ffmpeg` |
| Token leak / curiga ke-compromise | Apa pun | BotFather → `/revoke` → bikin baru, update `.env`, restart gateway |

---

## 12. Security checklist

Sebelum deploy ke "real" use:

- [ ] `TELEGRAM_ALLOWED_USERS` di-set, gak kosong (kalau kosong, default = deny semua, tapi pastikan)
- [ ] `~/.hermes/.env` permissions `chmod 600` (cuma user yang bisa baca)
- [ ] VPS firewall: gak ada port Hermes yang exposed unnecessarily
- [ ] Gateway log di-rotate (default `journalctl` udah rotate)
- [ ] `approvals.mode: manual` (atau `smart`), JANGAN `off` di production
- [ ] Bot username gak ngandung info sensitif (jangan `internal_corp_bot`)

---

## 13. Quick command via Telegram (zero-token)

Lo bisa setup quick command di `config.yaml` (lihat doc 05) lalu pake langsung dari Telegram:

```
/status
/disk
/uptime
```

Mereka jalan **tanpa LLM call** — instant, zero token cost. Cocok buat ops check yang sering dipake.

---

## 14. Untuk team (multiple users)

```bash
TELEGRAM_ALLOWED_USERS=123456789,987654321,555555555
```

Kombinasi dengan `group_sessions_per_user: true` — tiap member tim punya conversation isolated.

Detail tutorial team setup: [Team Telegram Assistant guide](https://hermes-agent.nousresearch.com/docs/guides/team-telegram-assistant).

---

## 15. Yang gw belum yakin

- **Latency voice di provider Telegram tertentu**: voice-to-voice round-trip (lo kirim voice → STT → LLM → TTS → voice reply) tergantung banyak faktor (model size, network ke OpenRouter/DeepSeek, dll). Real-world latency belum gw benchmark — kemungkinan 5-15 detik untuk pesan singkat.
- **Streaming behavior di group chat**: di config gw enable `streaming: true`. Di DM jalan smooth (progressive edit). Di group dengan banyak user, behavior streaming bisa rame. Test di setting lo.

---

## Lanjut

→ [09 — Optimasi Token (HEMAT BANGET)](09-optimasi-token.md): semua knob untuk minimize biaya tanpa kurangin quality.
