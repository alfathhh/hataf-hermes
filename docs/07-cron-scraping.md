# 07 — Cronjob untuk Scraping & Monitoring

Tujuan: bikin Hermes jalan otomatis di background — scraping harian, daily brief, monitoring server, alert via Telegram.

> Sumber: [Cron docs](https://hermes-agent.nousresearch.com/docs/user-guide/features/cron), [Automate with Cron guide](https://hermes-agent.nousresearch.com/docs/guides/automate-with-cron).

---

## 1. Konsep

Hermes punya **cron scheduler built-in** — jalan di gateway daemon, tick tiap 60 detik.

Cara kerja:

1. Lo bikin job (via slash command, CLI, atau natural language)
2. Job stored di `~/.hermes/cron/jobs.json`
3. Gateway daemon cek `next_run_at` setiap tick
4. Job due → spawn fresh agent session, jalanin prompt, deliver hasil
5. Hasil dikirim ke output destination (Telegram, file, dll)

Yang bagus:
- Setiap cron run = **fresh session** (gak pollute conversation utama lo)
- Bisa attach skill untuk inherit workflow
- Hasil otomatis di-deliver, gak perlu lo cek

Yang harus lo aware:
- **Cron job gak bisa create cron job lain** (anti recursion loop)
- Prompt cron HARUS self-contained — gak ada konteks dari conversation lo

---

## 2. Prasyarat

Pastikan gateway daemon jalan:

```bash
# Install sebagai user service (jalan saat lo login)
hermes gateway install

# Atau system service (jalan saat boot — recommended buat VPS)
sudo hermes gateway install --system

# Cek status
hermes gateway status
hermes cron status
```

Kalau jalan di VPS dan lo logout, **system service** (`--system`) yang lo butuh, bukan user service (yang ke-stop saat lo logout).

---

## 3. Format schedule

Hermes accept beberapa format:

### Relative delay (one-shot)

```
30m         # 30 menit dari sekarang
2h          # 2 jam
1d          # 1 hari
```

### Interval (recurring)

```
every 30m
every 2h
every 1d
```

### Cron expression (POSIX)

```
0 9 * * *       # Setiap hari jam 9 pagi
0 9 * * 1-5     # Hari kerja jam 9 pagi
0 */6 * * *     # Setiap 6 jam
30 8 1 * *      # Tanggal 1 setiap bulan jam 8:30
```

### ISO timestamp (one-time)

```
2026-06-15T09:00:00     # Sekali, di tanggal & jam tertentu
```

---

## 4. Bikin job — 3 cara

### Cara 1: Slash command (di chat / Telegram)

```
/cron add "every 1h" "Cek apakah ada PR baru di github.com/myorg/myrepo, kasih ringkasan dengan judul + author + URL. Kirim ke Telegram." --skill research-citation
```

### Cara 2: CLI

```bash
hermes cron create "every 1h" "..." --skill research-citation --name "PR Watch"
```

### Cara 3: Natural conversation

```
> Tiap pagi jam 8, scrape headline dari kompas.com dan tempo.co, summarize jadi 5 bullet utama, kirim ke Telegram
```

Hermes auto-call `cronjob` tool dengan parameter yang sesuai.

---

## 5. Resep cronjob (recipes siap pakai)

### Recipe 1: Daily news brief

**Tujuan**: tiap pagi dapet ringkasan berita yang relevan ke lo.

```
/cron add "0 7 * * *" "Cari headline tech news dari hari ini di Hacker News (top 10), TechCrunch (top 5), dan Detik Inet (top 5). Filter yang berhubungan dengan AI / startup Indonesia / cloud infra. Output: 7 bullet utama dengan link. Format jadi 1 pesan Telegram." --skill research-citation --name "Morning Brief"
```

Pakai skill `research-citation` — biar setiap link punya source. Jadwal 7 pagi WIB.

### Recipe 2: Monitor harga produk

**Tujuan**: cek harga produk tertentu, alert kalau drop > 10%.

Step 1, set up state file (sekali aja):

```bash
echo '{"last_price": null}' > ~/.hermes/cron/output/iphone-price/state.json
```

Step 2, bikin cron:

```
/cron add "every 6h" "Scrape harga 'iPhone 15 Pro 256GB' di tokopedia.com. Bandingkan dengan harga di ~/.hermes/cron/output/iphone-price/state.json. Kalau drop >10% dari last_price, kirim alert ke Telegram. Update state.json. Kalau sama atau naik, gak usah kirim apa-apa." --skill web-scrape --name "iPhone Price Watch"
```

> Catatan: scraping marketplace itu rapuh (struktur HTML bisa berubah). Test sendiri sebelum deploy.

### Recipe 3: Server health check

**Tujuan**: cek server lo masih hidup, alert kalau down.

```
/cron add "every 5m" "Test http://my-api.hataf.com/health. Kalau status code != 200, atau response time > 3 detik, kirim alert ke Telegram dengan diagnosa singkat. Kalau OK, gak usah kirim apa-apa (silent success)." --name "API Healthcheck"
```

### Recipe 4: Weekly review

**Tujuan**: tiap Senin pagi, review minggu lalu.

```
/cron add "0 9 * * 1" "Review session_search untuk percakapan 7 hari terakhir. Identifikasi: (1) topik yang sering muncul, (2) task yang belum selesai (kalau ada disebut), (3) pola pertanyaan saya. Kasih insight singkat 5 bullet. Kirim Telegram." --name "Weekly Review"
```

### Recipe 5: Backup config Hermes

**Tujuan**: backup config + memory + skills tiap minggu.

```
/cron add "0 3 * * 0" "Run shell command: tar czf /tmp/hermes-backup-$(date +%F).tar.gz ~/.hermes/config.yaml ~/.hermes/SOUL.md ~/.hermes/memories/ ~/.hermes/skills/ && rclone copy /tmp/hermes-backup-$(date +%F).tar.gz remote:hataf-backups/ . Hapus file sementara. Kirim notif sukses/gagal ke Telegram." --name "Weekly Backup"
```

> Asumsi `rclone` udah configured. Adapt ke cloud storage lo.

### Recipe 6: Monitor situs target spesifik

**Tujuan**: pantau halaman tertentu, alert kalau ada perubahan.

```
/cron add "every 2h" "Fetch https://example.com/pengumuman. Compute hash isi <main> tag. Bandingkan dengan ~/.hermes/cron/output/site-monitor/last-hash.txt. Kalau beda, kirim diff ringkas ke Telegram. Update last-hash.txt." --name "Pengumuman Watch"
```

### Recipe 7: PDF inbox processor

**Tujuan**: lo drop PDF ke folder, agent auto-summarize.

```
/cron add "every 30m" "Cek folder ~/inbox-pdf/. Untuk setiap PDF baru (yang belum ada di ~/inbox-pdf/.processed/), summarize dengan skill pdf-summarize, simpan summary di ~/inbox-pdf/.processed/<filename>.summary.md, kirim TL;DR ke Telegram, dan move PDF ke ~/inbox-pdf/.processed/." --skill pdf-summarize --name "PDF Inbox"
```

### Recipe 8: Multi-skill brief gabungan

```
/cron add "0 18 * * *" "Brief sore: (1) cuaca Bandung besok (search), (2) status server saya (terminal: curl health endpoints), (3) PRs baru di repo aktif. Gabungin jadi 1 pesan Telegram, format clear sections." --skill research-citation --skill web-scrape --name "Evening Brief"
```

Multiple skill di-load berurutan, prompt di-overlay di atasnya.

---

## 6. Manage jobs

```bash
# List semua
hermes cron list

# Lihat detail satu job
hermes cron status <job_id>

# Pause (sementara stop, gak ke-delete)
hermes cron pause <job_id>

# Resume
hermes cron resume <job_id>

# Force trigger sekali (testing)
hermes cron run <job_id>

# Edit
hermes cron edit <job_id> --schedule "every 4h"
hermes cron edit <job_id> --prompt "..."
hermes cron edit <job_id> --add-skill new-skill
hermes cron edit <job_id> --remove-skill old-skill

# Hapus
hermes cron remove <job_id>
```

Atau di chat: `/cron list`, `/cron pause <id>`, dst.

---

## 7. Output destination

Default delivery:

| Konteks | Default |
|---|---|
| Bikin job dari Telegram chat | `origin` (kembali ke chat itu) |
| Bikin job dari CLI | `local` (file di `~/.hermes/cron/output/<job_id>/`) |

Override eksplisit:

```
/cron add "every 1h" "..." --output telegram
/cron add "every 1h" "..." --output telegram:123456     # specific chat ID
/cron add "every 1h" "..." --output local
```

Untuk Telegram default channel: set `TELEGRAM_HOME_CHANNEL` di `.env` atau pake `/sethome` di chat target.

---

## 8. Self-contained prompt (penting)

Cron job jalan di **fresh session** — gak ada conversation history, gak ada konteks dari interaksi lo barusan.

❌ **Buruk** (gak self-contained):
```
"Cek server itu yang lagi bermasalah"
```

✅ **Bagus** (full context):
```
"SSH ke 192.168.1.100 dengan user deploy via key ~/.ssh/deploy_ed25519. Run 'systemctl status nginx postgresql redis'. Ambil output. Untuk setiap service: kalau status != 'active (running)', flag dan kirim ke Telegram dengan log error 20 baris terakhir. Kalau semua OK, kirim 1-line: 'All services OK'."
```

---

## 9. Token cost estimasi cron

Per cron run, biaya token tergantung kompleksitas. Rough estimate (gw belum benchmark presisi):

| Cron tipe | Estimasi token/run |
|---|---|
| Healthcheck (terminal command + assess) | ~2.000 - 5.000 |
| Daily brief (3-5 web search + summarize) | ~10.000 - 30.000 |
| PDF summarize (paper 20 halaman) | ~20.000 - 80.000 |
| Multi-source scraping + diff | ~10.000 - 50.000 |

Angka ini **kira-kira**, perlu lo benchmark sendiri di setting lo. Hindari cron yang sangat sering (every 1 menit) untuk task yang berat — bisa cepat boros.

Tips hemat:

- Pake cheap_model untuk monitoring sederhana (lewat smart_model_routing — tapi ini gak active di cron, jadi lo set delegation atau langsung kasih `--model qwen3.5-plus`)
- Pake quick_commands kalau bisa — zero LLM cost

---

## 10. Security cron

Cron prompt di-scan untuk:
- Prompt injection
- Credential exfiltration patterns
- Invisible Unicode tricks
- SSH backdoor patterns

Kalau prompt lo "kelihatan curiga" oleh scanner, dia direject saat create. Modify prompt lo, jangan paksa bypass.

> **Wajib**: jangan masukin secrets langsung di cron prompt. Pake env variable / file ref. Misal:
>
> ❌ `"login dengan token ghp_xxxxx"`
> ✅ `"login dengan token dari $GITHUB_TOKEN"` (gateway baca dari `.env`)

---

## 11. Yang gw belum yakin

- **Reliabilitas tick di high-load**: gateway tick tiap 60 detik. Kalau cron yang due lebih dari 60 detik durasinya, next tick mungkin ke-skip (ada file lock untuk anti-double-run). Belum gw stress-test.
- **Cron job persistance saat update Hermes**: setelah `hermes update`, jobs.json **harusnya** persisted (di `~/.hermes/cron/`), tapi schema versi config bisa berubah. **Backup `~/.hermes/cron/jobs.json` sebelum upgrade major version**.

---

## Lanjut

→ [08 — Telegram Gateway](08-telegram-gateway.md): pasang bot Telegram lo biar agent bisa diakses dari HP.
