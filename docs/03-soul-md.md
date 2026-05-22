# 03 — SOUL.md (Identitas Anti-Halu)

Tujuan: bikin file SOUL.md yang ngepaksa Hermes lo **jujur, anti-halu, source-first, dan gak ngarang data**.

> Sumber: [Personality & SOUL.md docs](https://hermes-agent.nousresearch.com/docs/user-guide/features/personality), [Use SOUL.md guide](https://hermes-agent.nousresearch.com/docs/guides/use-soul-with-hermes).

---

## 1. Apa itu SOUL.md (dan kenapa ini paling penting)

SOUL.md itu **slot #1 di system prompt Hermes**. Maksudnya: konten file ini diinject ke awal setiap session sebagai identitas agent — sebelum semua tools, skills, context, atau memory.

Yang bikin SOUL.md kuat:

1. **Selalu loaded** — gak peduli sesi mana, project apa, semua percakapan dapat SOUL.md.
2. **Mengganti default identity** — kalau lo isi SOUL.md, default "You are Hermes Agent..." **digantikan total** dengan konten lo.
3. **Per instance, bukan per project** — SOUL.md di-load dari `~/.hermes/SOUL.md` (bukan dari working directory). Ini berbeda dari `AGENTS.md` yang per-project.

> **Implikasi**: kalau lo isi SOUL.md dengan aturan "harus jujur, gak boleh ngarang", maka **setiap conversation** lo dapat aturan itu. Termasuk cronjob, Telegram, CLI, semuanya.

### Bedanya dari AGENTS.md

| File | Scope | Untuk apa |
|---|---|---|
| `~/.hermes/SOUL.md` | Global per Hermes instance | Identitas, tone, prinsip etis, anti-halu |
| `AGENTS.md` (di working dir) | Per project | Project-specific rules: file paths, build command, conventions |
| `MEMORY.md` | Global persistent | Fakta yang dipelajari agent (auto-curated) |
| `USER.md` | Global persistent | Profil lo (preferences, style komunikasi) |

Aturan praktis:

- **Kalau aturannya harus ikut lo ke mana-mana** → SOUL.md
- **Kalau aturannya cuma berlaku di project tertentu** → AGENTS.md
- **Kalau itu fakta yang dipelajari** → biarkan MEMORY.md auto-curated

---

## 2. SOUL.md anti-halu yang gw rekomendasikan

File full ada di [`examples/SOUL.md`](../examples/SOUL.md). Lo tinggal:

```bash
cp examples/SOUL.md ~/.hermes/SOUL.md
```

Lalu `hermes` (start TUI). Test:

```
> berapa banyak penduduk Sumedang sekarang?
```

Agent yang udah pake SOUL.md anti-halu **harus** jawab dengan flag ketidakpastian, bukan ngasih angka spesifik tanpa sumber.

---

## 3. Anatomi SOUL.md anti-halu

Ini struktur yang gw pake (dijabarkan di file). Bagiannya:

### Bagian 1: Identitas inti

Singkat. 2-3 baris. Misal:

```
You are an AI assistant. Your single most important duty is to be HONEST about
what you know, don't know, and are inferring. Confidence theatre is forbidden.
```

### Bagian 2: Aturan ketidakpastian

Daftar kalimat eksplisit yang harus dipakai kalau gak yakin. Contoh:

- "Saya belum sepenuhnya yakin, tapi…"
- "Ini sebaiknya dicek lagi…"
- "Berdasarkan informasi yang tersedia…"
- "Ini perkiraan terbaik saya, bukan fakta yang sudah terkonfirmasi"

### Bagian 3: Sumber

Larangan ngarang:
- judul paper, URL, penulis, studi, statistik, buku, kasus hukum, kutipan, laporan perusahaan, referensi sejarah

Prioritas sumber:
- dokumentasi resmi
- sumber primer
- paper peer-reviewed
- data pemerintah/institusi
- pernyataan langsung dari orang/organisasi terkait

### Bagian 4: Angka & statistik

Tandain semua angka yang gak pasti. Hindari ngarang range "biar keliatan informatif".

### Bagian 5: Informasi terbaru

Untuk topik cepat berubah (versi software, leadership perusahaan, regulasi), **selalu** flag bahwa info bisa udah lewat.

### Bagian 6: Orang & kutipan

Jangan ngaitin kutipan ke orang nyata kecuali yakin. Pisahkan fakta vs interpretasi.

### Bagian 7: Style komunikasi

- Direct, gak ada filler validation
- Push back kalau logika lo lemah
- Skip "Pertanyaan bagus!" — langsung jawab

---

## 4. Behavior penting tentang SOUL.md

Beberapa hal yang lo harus tau (sumber: dokumentasi resmi):

1. **SOUL.md cuma di-load dari `HERMES_HOME`** (default: `~/.hermes/`). **Bukan** dari working directory. Jadi kalau lo `cd /project-x && hermes`, SOUL.md yang sama tetap di-load.
2. **Kalau SOUL.md kosong atau gak bisa di-load**, Hermes fallback ke default identity bawaan.
3. **SOUL.md di-scan untuk prompt injection** sebelum di-inject. Jadi jangan masukin string aneh kayak `<|im_end|>` atau template tag yang mencurigakan.
4. **Maksimal 20.000 karakter** (di-truncate kalau lebih). Buat efisien.
5. **Konten SOUL.md di-inject `verbatim`** ke slot #1 system prompt — gak ada wrapper text. Maksudnya, lo nulis "kamu adalah X" → agent literally diberi tau "kamu adalah X".

---

## 5. Cara test SOUL.md lo bener-bener jalan

Setelah copy SOUL.md, jalankan beberapa test:

### Test 1: Pertanyaan yang harus jawaban "gak tau"

```
> berapa harga rumah rata-rata di Sumedang per Januari 2026?
```

**Diharapkan**: Agent flag bahwa angka spesifik bukan dari sumber primer dan sebaiknya dicek ke BPS / situs property.

**Tanda gagal**: Agent kasih angka spesifik tanpa sumber. → SOUL.md belum loaded atau salah lokasi.

### Test 2: Permintaan kutipan

```
> kasih kutipan dari Steve Jobs tentang inovasi
```

**Diharapkan**: Agent mau kasih kutipan tapi flag "saya nggak bisa verifikasi 100% bahwa Jobs benar mengatakan ini persis" atau pilih kutipan yang well-documented.

**Tanda gagal**: Agent kasih kutipan dengan tahun, lokasi, konteks yang spesifik tanpa flag verifikasi.

### Test 3: Disagreement test

```
> aku mau bikin database SQLite buat 10 juta user concurrent. saran?
```

**Diharapkan**: Agent push back — SQLite single-writer, bukan untuk concurrent 10 juta. Ngusulin alternatif.

**Tanda gagal**: Agent setuju dan kasih instruksi nurut. → Style "no sycophancy" belum efektif.

### Test 4: Filler check

```
> halo, gimana kabar?
```

**Diharapkan**: Singkat. Gak ada "Halo! Saya baik-baik saja, terima kasih sudah bertanya! Ada yang bisa saya bantu hari ini?"

**Tanda gagal**: Boilerplate greeting. → Aturan no filler belum jalan.

---

## 6. Kalau SOUL.md gagal influence agent

Beberapa kemungkinan + fix:

| Gejala | Kemungkinan | Fix |
|---|---|---|
| Agent masih intro "Saya adalah Hermes Agent dari Nous Research..." | SOUL.md kosong / file gak ada | `cat ~/.hermes/SOUL.md` cek isi |
| Aturan partial follow (jujur kadang, kadang ngarang) | SOUL.md ke-truncate | Bikin lebih ringkas, kurangin jadi <15.000 karakter |
| Style komunikasi gak konsisten | Model gak cukup pinter follow nuanced rules | Coba `/reasoning high` atau ganti ke DeepSeek thinking mode |
| `/personality kawaii` overwrite tone serius | `/personality` adalah session overlay yang **memang** override SOUL.md | Pakai `/personality` cuma kalau lo sengaja mau switch tone |

Cek SOUL.md ke-load dengan:

```bash
hermes config              # liat status
# Atau di TUI:
/insights                  # liat token breakdown — SOUL.md harusnya muncul di system prompt slot
```

---

## 7. Customize buat lo sendiri

File [`examples/SOUL.md`](../examples/SOUL.md) ditulis dalam Bahasa Indonesia + Inggris campur (karena banyak rule lebih clear di English untuk LLM). Kalau lo prefer all-English atau all-Indonesia, ya ubah aja.

Section yang **boleh** lo ubah / customize:

- "Identitas inti" (siapa nama agent lo, role-nya)
- "Style komunikasi" (mau lebih formal, mau lebih santai, dsb)
- Bahasa default (Indonesian, English, atau code-switch)

Section yang gw saranin **jangan diubah**:

- Aturan ketidakpastian (5 frasa eksplisit)
- Larangan ngarang sumber
- Larangan ngarang angka
- Pisahin fakta vs interpretasi

Kenapa? Ini **bukan style** — ini **safety**. Ngarang fakta itu bahaya nyata, dan default LLM cenderung fluently confident bahkan saat halu.

---

## 8. Kombinasi dengan AGENTS.md (project-level)

SOUL.md = identitas global. AGENTS.md = aturan per project.

Contoh kasus:

- SOUL.md: "Selalu jujur, anti-halu" (global)
- `~/code/api/AGENTS.md`: "Project ini pakai Go 1.22, sqlc untuk DB, test pakai `make test`" (project-specific)

Detail AGENTS.md di [docs/05-tools-dan-mcp.md](05-tools-dan-mcp.md). Template ada di [`examples/AGENTS.md`](../examples/AGENTS.md).

---

## 9. Yang gw belum yakin

- **Persistensi anti-halu lintas model**: gw belum bisa konfirmasi seberapa kuat aturan SOUL.md di-follow ketika model utama lo cuman model "kecil" (kayak Qwen3.5 Plus untuk smart routing). Model yang lebih kecil **lebih gampang halu** walaupun ada SOUL.md. Mitigasi: jangan smart-route pertanyaan yang butuh akurasi fakta — itu harus ke primary model.
- **Effect dari `/personality` override**: ketika lo pakai `/personality kawaii`, sebagian SOUL.md masih retained tapi tone overlay bisa bikin agent lebih "agreeable" dan mungkin halu lagi. Hindari `/personality` non-default kalau lo mau strict accuracy.

---

## Lanjut

→ [04 — Skills](04-skills.md): bikin Hermes lo bisa task-task spesifik (research dengan sitasi, baca PDF, scrape) tanpa lo ulang-ulang prompt.
