# 10 — Mengajarkan Hermes (Bikin Lebih Pinter Tiap Hari)

Tujuan: pakai mekanisme self-improvement Hermes (skills auto-create, memory curation, session search) untuk bikin agent lo makin sesuai kebutuhan dari minggu ke minggu.

> Sumber: [Skills System docs](https://hermes-agent.nousresearch.com/docs/user-guide/features/skills), [Curator docs](https://hermes-agent.nousresearch.com/docs/user-guide/features/curator), [Memory docs](https://hermes-agent.nousresearch.com/docs/user-guide/features/memory).

---

## 1. Filosofi: agent yang grow vs agent statis

Default LLM stateless — tiap session ngulang dari nol. Hermes beda: punya **closed learning loop**:

```
┌─────────────────────────────────────────────────┐
│  Lo kerja sama Hermes → ada interaksi           │
└────────────────┬────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────┐
│  Hermes auto-curate:                            │
│  ─ Save fakta penting ke MEMORY.md              │
│  ─ Update profil di USER.md                     │
│  ─ Bikin skill baru kalau workflow non-trivial  │
│  ─ Index conversation di session DB             │
└────────────────┬────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────┐
│  Next session: agent inget, agent reuse skill,  │
│  agent search past conversation kalau perlu     │
└─────────────────────────────────────────────────┘
```

Yang bikin loop ini efektif: **lo aktif kasih feedback**, bukan tunggu agent nemuin sendiri.

---

## 2. 4 cara mengajar Hermes

### Cara 1: Eksplisit "remember this"

Paling simpel, paling reliable.

```
> remember bahwa server staging API saya butuh SSH port 2222 (bukan 22), dengan key di ~/.ssh/staging_ed25519
```

Hermes auto call `memory(action="add", target="memory", ...)`. Persisted ke `MEMORY.md`.

Kapan pake:
- Fakta env (paths, ports, conventions)
- Aturan project yang frequent muncul
- Lessons learned dari debugging

### Cara 2: Koreksi inline → memory

Saat agent salah, koreksi:

```
Hermes: Saya akan jalankan `sudo systemctl restart nginx` untuk apply config
You: jangan pake sudo. di server ini saya udah di group sudo dan systemd nerima command tanpa sudo. ingat itu.
Hermes: Noted. [Save to memory: "User has sudo permissions; systemctl works without sudo."]
```

Setelah koreksi, agent harusnya auto-save ke memory. Cek dengan `cat ~/.hermes/memories/MEMORY.md` setelah session.

### Cara 3: Bikin skill saat lo nemu workflow yang berulang

Setelah lo kerjakan task complex bersama agent (5+ tool calls, atau task yang butuh trial-and-error), tanya:

```
> tadi kita debug masalah pkg.go.dev bisa download module private dari Goproxy. itu workflow yang akan saya pakai lagi. tolong ekstrak jadi SKILL.md di ~/.hermes/skills/go-private-module/, dengan section When to Use, Procedure, Pitfalls, Verification.
```

Agent otomatis call `skill_manage(action="create", name="go-private-module", ...)`.

Lain kali muncul issue serupa, tinggal `/go-private-module` — instant load workflow.

### Cara 4: Edit langsung file (advanced)

Lo bisa edit `~/.hermes/SOUL.md`, `~/.hermes/memories/USER.md`, `~/.hermes/skills/<name>/SKILL.md` manual dengan editor. Restart Hermes untuk reload.

Kapan pake:
- Bersihin memory yang udah stale
- Refine SKILL.md setelah dipake beberapa kali (procedure-nya kurang akurat)
- Tambah USER.md baru info yang lo gak nyaman tunggu agent discover

---

## 3. Curator (background maintenance)

Hermes punya **Curator** subsystem yang tracking skill usage dan kualitas:

- Catat berapa kali tiap skill dipanggil
- Detect skill yang udah lama gak dipake (staleness)
- Periodic LLM-driven review: skill mana yang outdated / harus di-archive
- Auto-archive skill yang gak relevan lagi

Detail: [Curator docs](https://hermes-agent.nousresearch.com/docs/user-guide/features/curator).

Lo bisa cek manual:

```bash
hermes skills list                      # semua skill + last_used
hermes skills audit                     # security re-scan
```

---

## 4. Iteratif refine SKILL.md

Pattern yang gw rekomendasikan:

### Iterasi 1: Bikin draft

Lewat conversation, sebagaimana cara 3 di atas.

### Iterasi 2: Pake skillnya beberapa kali

Test di real task. Catat:

- Step mana yang agent skip (mungkin gak jelas di SKILL.md)
- Step mana yang agent ulang-ulang (mungkin redundant)
- Hasil yang gak sesuai expectation (procedur kurang specific)

### Iterasi 3: Edit

```
> aku notice bahwa skill /web-scrape sering gagal extract harga produk Tokopedia karena format harganya kompleks. update SKILL.md di ~/.hermes/skills/web-scrape/ dengan handling tambahan: kalau target site = tokopedia.com, parse harga dengan regex /Rp\s*([\d.]+)/ dan strip titik thousand-separator.
```

Agent edit SKILL.md (via `skill_manage(action="patch")`). Save.

### Iterasi 4: Repeat

Tiap kali skill ada gap, refine. Tujuannya: SKILL.md jadi **resep matang** yang reusable.

---

## 5. Session search untuk recall

Setelah pakai Hermes berminggu-minggu, lo akan lupa apa yang udah dibahas. Pakai session search:

```
> minggu lalu kita bahas optimasi nginx untuk websocket. apa kesimpulannya?
```

Agent call `session_search(query="nginx websocket optimization")` → cari conversation lama → summarize.

Untuk inisiatif:

```bash
hermes sessions list                    # semua sesi
hermes sessions search nginx            # search keyword
```

---

## 6. Workflow bulanan: review & cleanup

Sekali/sebulan:

### Step 1: Review memory

```bash
cat ~/.hermes/memories/MEMORY.md
cat ~/.hermes/memories/USER.md
```

Buang entry yang udah gak relevan (project yang udah selesai, server yang udah dimatikan).

### Step 2: Review skills

```bash
hermes skills list
```

- Skill yang last_used > 30 hari + irrelevant: hapus
- Skill yang last_used > 30 hari tapi mungkin relevan: keep, tapi cek apakah procedure-nya masih akurat
- Skill yang sering dipake: refine kalau ada gap

### Step 3: Backup

```bash
tar czf hermes-monthly-$(date +%F).tar.gz \
  ~/.hermes/config.yaml \
  ~/.hermes/SOUL.md \
  ~/.hermes/memories/ \
  ~/.hermes/skills/ \
  ~/.hermes/cron/jobs.json

# Upload ke cloud / external drive
```

---

## 7. Anti-pattern saat ngajar Hermes

### Anti-pattern 1: Spam memory dengan fakta tidak penting

Memory cap kecil. Jangan paksa simpan "Today I asked about Python".

Filter mental: "Apakah saya akan butuh fakta ini lagi 1 minggu dari sekarang?"

### Anti-pattern 2: Skill terlalu sempit

❌ "Skill khusus untuk handle URL X di hari Senin"
✅ "Skill untuk handle category URL secara umum"

Skill yang general lebih reusable. Kalau ada special case, tambahin di section "Edge cases" di SKILL.md.

### Anti-pattern 3: Edit SOUL.md tiap minggu

SOUL.md = identitas durable. Kalau lo ubah-ubah, behavior agent jadi inconsistent. Edit cuma kalau ada drift fundamental yang lo notice.

### Anti-pattern 4: Tugaskan everything ke 1 skill jumbo

Skill bagus = 1 file SKILL.md, 200-500 baris max. Skill jumbo (1000+ baris) → hard to maintain, dan progressive disclosure level 1-nya jadi mahal token.

Pecah jadi multiple skill: `pdf-summarize-academic`, `pdf-summarize-contract`, `pdf-summarize-regulation`.

### Anti-pattern 5: Gak pernah test perubahan

Tiap kali edit SOUL/SKILL/MEMORY, **test minimum 3 query** yang menguji perubahan. Kalau gak test, mungkin nyerusak behavior tanpa lo sadari.

---

## 8. Tracking improvement (optional, advanced)

Kalau lo serius ingin **measure** apakah agent makin pinter:

### Approach 1: Test set personal

Bikin 20-30 query yang representatif dari use case lo. Run mereka tiap bulan, track:

- Berapa yang dapet jawaban memuaskan first-try?
- Berapa yang harus lo koreksi?
- Berapa yang halu / sumber palsu?

Note hasilnya di tempat lain (Notion, simple text file).

### Approach 2: Log + manual review

Set `display.streaming: true` dan `display.tool_progress: all`. Save log session penting. Review weekly:

- Pattern halu yang masih muncul
- Tools yang sering salah panggil
- Memory yang mestinya tersimpan tapi gak

### Approach 3: Profile distribution

Hermes punya fitur "profile distribution" — package complete agent (config + skill + cron) sebagai git repo. Lo bisa **version control** agent lo sendiri:

```bash
hermes profile distribute --to git --repo github.com/lo/hermes-personal
```

Detail: [Profile Distributions docs](https://hermes-agent.nousresearch.com/docs/user-guide/profile-distributions).

Useful kalau lo:
- Setup di multiple machine (laptop, VPS)
- Tracking evolusi agent lo dari waktu ke waktu (git history)

---

## 9. Kapan agent lo "matang"

Tanda-tanda Hermes lo udah well-tuned:

- ✅ Lo jarang harus repeat instruksi yang sama
- ✅ Agent inget conventions project tanpa lo bilang
- ✅ Skill yang sering dipake jalan tanpa salah-step
- ✅ Memory MEMORY.md isinya 90% relevan, gak ada junk
- ✅ Halu rate < 5% (estimasi subjektif lo)
- ✅ Quick commands cover 80% ops check rutin
- ✅ Cronjob jalan reliable, alert akurat

Kalau belum, identifikasi gap mana yang paling sering muncul, fokus refine ke situ.

---

## 10. Yang gw belum yakin

- **Honcho effect terhadap quality**: Honcho (memory eksternal advanced) klaim better long-term user modeling, tapi gw belum benchmark vs default MEMORY.md. Kalau lo udah pakai Hermes 3+ bulan dan masih merasa agent gak "kenal" lo, baru pertimbangkan Honcho.
- **Curator behavior**: gw belum verifikasi seberapa aggressive Curator auto-archive skill. Kalau ada skill yang tiba-tiba "menghilang", cek `~/.hermes/skills/.archived/` (kalau ada folder seperti itu) atau `hermes skills audit`.

---

## Penutup tutorial

Lo udah selesai 10 docs. Hermes lo sekarang:

- ✅ Anti-halu (SOUL.md tegas)
- ✅ Multi-provider dengan fallback (DeepSeek + OpenCode Go)
- ✅ Hemat token (smart routing, compression, quick_commands)
- ✅ Skill yang reusable (research, PDF, video, scrape)
- ✅ Memory persisten (MEMORY.md, USER.md curated)
- ✅ Background automation (cronjob harian)
- ✅ Mobile-accessible (Telegram gateway)
- ✅ Tumbuh dari waktu ke waktu (feedback loop)

Step terakhir yang gw saranin:

1. **Set up dulu yang minimal** — install + DeepSeek + SOUL.md (doc 01-03)
2. **Jalanin 1 minggu** sebagai user normal — chat aja, pake naturally
3. **Review setelah 1 minggu** — cek memory, cek pattern, identifikasi pain point
4. **Tambah skill, MCP, cronjob** sesuai pain point yang muncul
5. **Iterate**

Jangan setup semuanya hari pertama — lo gak akan tau yang mana actually lo butuhin sebelum jalan beberapa hari.

---

## Sumber resmi (semua yang gw rujuk)

- [Hermes Agent docs](https://hermes-agent.nousresearch.com/docs/) — main reference
- [GitHub repo](https://github.com/NousResearch/hermes-agent) — source code, install script
- [DeepSeek API docs](https://api-docs.deepseek.com/) — model resmi
- [OpenCode docs](https://opencode.ai/docs) — provider OpenCode Go
- [agentskills.io](https://agentskills.io/specification) — open standard SKILL.md
- [models.dev](https://models.dev) — model registry untuk context length resolution

Cek sumber di atas untuk versi terbaru — banyak detail di tutorial ini akan **berubah** seiring update Hermes.
