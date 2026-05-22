# 06 — Memory & USER Profile

Tujuan: setup memory persisten Hermes — yang inget preferensi lo dan fakta penting lintas sesi, **tanpa overload context tiap turn**.

> Sumber: [Memory docs](https://hermes-agent.nousresearch.com/docs/user-guide/features/memory).

---

## 1. Konsep

Hermes punya 2 memory file persisten:

| File | Lokasi | Cap | Untuk apa |
|---|---|---|---|
| `MEMORY.md` | `~/.hermes/memories/MEMORY.md` | **2.200 char (~800 token)** | Catatan agent: env facts, conventions, lessons learned |
| `USER.md` | `~/.hermes/memories/USER.md` | **1.375 char (~500 token)** | Profil lo: preferensi, style, role |

Plus session search:

| Fitur | Storage | Untuk apa |
|---|---|---|
| Past conversations | `~/.hermes/state.db` (SQLite + FTS5) | Search percakapan lama on-demand |

---

## 2. Kenapa cap ketat segitu

Cap kecil itu **fitur, bukan bug**. Alasan:

- Memory di-inject ke setiap session sebagai **frozen snapshot** di system prompt.
- Tiap turn = bayar ~1.300 token cuma untuk memory.
- Kalau memory gak di-cap, cepet membengkak jadi 5.000+ token, biaya naik linier.
- Cap memaksa **konsolidasi**: agent reject duplicate, gabungin entries serupa, hapus yang stale.

Jadi: **terima cap-nya, jangan dilonggarkan**. Default OK untuk 99% kasus.

---

## 3. Cara kerja memory tool

Agent punya `memory` tool dengan 3 action:

| Action | Fungsi |
|---|---|
| `add` | Tambah entry baru |
| `replace` | Ganti entry existing (substring match via `old_text`) |
| `remove` | Hapus entry (substring match) |

Tidak ada `read` action — agent baca memory dari system prompt yang udah di-inject saat session start.

### Behavior penting (gw verifikasi dari dokumentasi resmi)

1. **Frozen snapshot**: snapshot diambil di awal session. Perubahan mid-session ke-persist ke disk, tapi **gak muncul di system prompt** sampai session berikutnya. Ini disengaja biar prefix cache LLM tetep jalan.
2. **Substring match unik**: untuk `replace`/`remove`, lo gak perlu kasih full text — cukup substring yang unik identify entry tersebut.
3. **Auto reject duplicate**: kalau lo `add` content yang persis sama, gak akan double-saved.
4. **Security scan**: setiap entry di-scan untuk prompt injection / credential exfiltration sebelum diterima.

---

## 4. Apa yang DISIMPAN, apa yang JANGAN

### Simpan (auto, tanpa lo minta)

Agent harusnya auto-save kalau dia belajar:

| Tipe | Target | Contoh |
|---|---|---|
| Preferensi user | `user` | "User prefer TypeScript over JavaScript" |
| Env facts | `memory` | "VPS lo Ubuntu 22.04, ada Docker + Podman" |
| Koreksi user | `memory` | "Jangan pake `sudo` untuk Docker, user di group docker" |
| Conventions | `memory` | "Project ~/code/api pakai Go 1.22, sqlc, chi router" |
| Completed work | `memory` | "Migrasi DB MySQL→Postgres pada 2026-01-15" |
| Eksplisit dari user | `memory` | "Inget bahwa rotasi API key bulanan" |

### Jangan disimpan

- Trivial / obvious info ("user nanya soal Python")
- Fakta yang gampang re-discover via search
- Raw data dump (code panjang, log file)
- Session-specific ephemera (path /tmp, debug output)
- Info yang udah ada di SOUL.md / AGENTS.md

---

## 5. USER.md — bootstrap awal (sangat recommended)

Default Hermes biarin agent yang nemuin profil lo from scratch. Itu lambat dan boros — bisa 5-10 conversation sebelum dia "ngerti" lo.

**Shortcut**: tulis `USER.md` initial sendiri.

Edit `~/.hermes/memories/USER.md`:

```markdown
Nama: <nama lo>
Role: <data engineer / web dev / mahasiswa / dst>
Timezone: Asia/Jakarta (UTC+7)
Bahasa default: Bahasa Indonesia, switch ke English untuk istilah teknis
OS: Ubuntu 22.04 / macOS 14 Sonoma / WSL2
Editor: VS Code with Vim keybindings
Shell: zsh + oh-my-zsh

Style komunikasi:
- Singkat, langsung ke poin
- Hindari "Tentu! Pertanyaan bagus!"-an
- Kalau ada flaw di pertanyaan saya, push back, jangan iya-iya doang
- Kasih opsi alternatif kalau ide saya gak optimal

Pet peeves:
- Filler validation
- Halu sumber/angka
- Saran "best practice" tanpa konteks

Workflow habits:
- Commit kecil-kecil, squash sebelum merge
- Test dulu sebelum push
- Branching: feat/, fix/, chore/
```

Cap 1.375 karakter — keep it tight. Yang penting agent tau lo siapa, gak perlu auto-biography.

---

## 6. MEMORY.md — bootstrap juga

Sama, kasih starter:

`~/.hermes/memories/MEMORY.md`:

```markdown
Server utama: VPS DigitalOcean 4GB RAM Ubuntu 22.04 di sgp1 region. Domain hataf.com point ke 64.xxx.xxx.xxx.

Project aktif:
- ~/code/scrapper-news (Python, scrape harian, deploy lewat cron)
- ~/code/personal-blog (Astro, deploy ke Cloudflare Pages)

Tools yang udah terpasang: Docker, Podman, yt-dlp, ffmpeg, faster-whisper, ripgrep.

Conventions: file output cron disimpan di ~/.hermes/cron/output/. Backup config tiap minggu via cron job ke s3://hataf-backups/.
```

Cap 2.200 karakter — sama, keep tight. Detail spesifik bisa di-discover by search nanti.

---

## 7. Verifikasi memory di-load

Setelah edit, restart Hermes. Lalu:

```
/insights        # liat token breakdown system prompt
```

Lo harusnya liat section MEMORY dan USER PROFILE di prompt allocation. Kalau gak ada:

```bash
hermes config | grep -i memory
```

Pastikan `memory.memory_enabled: true` dan `memory.user_profile_enabled: true` di config.

---

## 8. Best practice nulis entry

Format yang efisien (compact + dense info):

✅ **Bagus**:
> User runs Ubuntu 22.04 with Docker & Podman. zsh + oh-my-zsh. Editor: VS Code Vim mode. Domain hataf.com on Cloudflare.

✅ **Bagus** (specific, actionable):
> Project ~/code/api uses Go 1.22, sqlc for DB queries, chi router. Run tests: `make test`. CI via GitHub Actions.

❌ **Buruk** (vague):
> User has a project.

❌ **Buruk** (terlalu panjang):
> On January 15, 2026, the user came to me with a request about their project located at ~/code/api which they explained is using Go version 1.22 and they prefer to use sqlc for database queries because...

---

## 9. Capacity full — apa yang terjadi

Kalau memory penuh dan agent coba `add`, dia dapet error:

```json
{
  "success": false,
  "error": "Memory at 2,100/2,200 chars. Adding 250 chars exceeds limit. Replace or remove first.",
  "current_entries": ["..."],
  "usage": "2,100/2,200"
}
```

Best practice agent (otomatis kalau pinter):

1. Liat current entries
2. Identifikasi yang bisa di-konsolidasi (3 entry "project X uses ..." → 1 entry comprehensive)
3. `replace` entry-entry tersebut jadi 1 entry pendek
4. Baru `add` yang baru

Kalau lo merasa Hermes lo jelek di management memory, lo bisa:

- Edit `~/.hermes/memories/MEMORY.md` manual (dia text file biasa, separator `§`)
- Restart Hermes untuk reload

---

## 10. Session search (cari past conversation)

Selain frozen memory, Hermes simpan **semua conversation** di SQLite dengan FTS5 full-text search.

Saat agent lupa hal yang pernah dibahas weeks ago, dia bisa:

```python
session_search(query="docker compose error redis 2026 januari")
```

Hasil: snippet conversation lama + summary (di-summarize via Gemini Flash auxiliary).

| | Persistent Memory | Session Search |
|---|---|---|
| **Cap** | ~1.300 token total | Unlimited (semua sesi) |
| **Latency** | Instant (di system prompt) | Search + LLM summarize |
| **Cost** | Fixed ~1.300 token/session | On-demand |
| **Use case** | Fakta penting selalu ada | "Pernah bahas X minggu lalu, apa ya?" |

Agent biasanya pinter milih kapan pake yang mana. Lo gak perlu otak-atik.

---

## 11. Honcho (optional, advanced)

Untuk agent yang **multi-platform** (CLI + Telegram + Discord) dan butuh model user yang dalam, ada Honcho — eksternal service untuk dialectic user modeling.

```bash
hermes honcho setup
```

> Honcho berjalan paralel dengan MEMORY/USER bawaan, bukan menggantikan. Detail: [Honcho docs](https://hermes-agent.nousresearch.com/docs/user-guide/features/honcho).

Skip dulu kalau lo baru mulai. Bisa di-enable nanti kalau memang butuh.

---

## 12. Yang gw belum yakin

- **Apakah memory entries leak antar profile**: kalau lo punya beberapa Hermes profile (`hermes profile`), gw belum verifikasi apakah memory shared atau per-profile. Test sendiri kalau use case lo butuh isolasi.
- **Honcho biaya**: pricing Honcho gw belum cek angkanya. Cek [honcho.dev](https://honcho.dev) sebelum subscribe.

---

## Lanjut

→ [07 — Cron Scraping](07-cron-scraping.md): bikin Hermes jalan otomatis di background untuk scraping & monitoring.
