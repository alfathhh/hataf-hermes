---
name: research-citation
description: Cari jawaban faktual dengan sumber yang bisa dicek. Wajib citation per klaim. Kalau sumber primer tidak ditemukan, jawab "tidak ditemukan" — jangan ngarang.
version: 1.0.0
metadata:
  hermes:
    tags: [research, fact-check, citation, anti-hallucination]
    category: research
    requires_toolsets: [web]
---

# Research with Citation

Skill ini dipanggil ketika user butuh **jawaban faktual yang bisa diverifikasi**, bukan opini atau best-guess. Output harus selalu menyertakan sumber primer atau dokumentasi resmi yang URL-nya bisa diklik.

## When to Use

Pakai skill ini bila request user:

- Menanyakan fakta yang bisa salah/benar (kapan, berapa, siapa, di mana)
- Menyebut hukum, regulasi, kebijakan, version software, leadership perusahaan
- Meminta data statistik, market size, salary, ranking
- Meminta kutipan dari orang publik
- Meminta info berita / current events

JANGAN pakai skill ini untuk:
- Permintaan opini ("menurutmu mana lebih bagus?")
- Brainstorming kreatif
- Coding task (pakai default)

## Procedure

### 1. Klarifikasi kebutuhan akurasi

Kalau pertanyaan ambigu, tanya dulu:
- "Lo butuh angka resmi terkini, atau perkiraan kasar OK?"
- "Lo nyari sumber tertentu (BPS, Kominfo, paper akademis), atau bebas?"

### 2. Search dengan multiple queries

Lakukan **minimal 2 query berbeda** ke `web_search`:
- Query 1: bahasa user, persis seperti yang ditanyakan
- Query 2: bahasa Inggris atau lebih spesifik (kalau topiknya teknis)
- Query 3 (opsional): tambah qualifier "official", "site:gov.id", "site:.edu"

### 3. Filter prioritas sumber

Ranking source berdasarkan trust:

| Tier | Sumber | Contoh |
|---|---|---|
| **1 (paling kuat)** | Sumber primer / official | api-docs.deepseek.com, bps.go.id, who.int, github.com/owner/repo |
| **2** | Dokumentasi resmi vendor | nodejs.org/en/docs, react.dev |
| **3** | Berita kredibel / wikipedia (untuk pointer) | reuters.com, bbc.com, kompas.com, en.wikipedia.org |
| **4 (lemah)** | Blog post, Medium, Stack Overflow answer | gunakan sebagai supporting only, bukan primary claim |

Skip total:
- Blog SEO (hasil "top 10 best...")
- Forum tanpa moderasi
- Halaman dengan banyak ad / clickbait

### 4. Extract claim per claim dengan URL

Setiap fakta yang lo sebut HARUS punya URL backing-nya:

```
DeepSeek V4 Flash punya context window 1M token [1].

[1] https://openrouter.ai/deepseek/deepseek-v4-flash (diakses [DATE_HARI_INI])
```

Kalau ada **konflik antar sumber** (sumber A bilang X, sumber B bilang Y):
- Sebut keduanya
- Identifikasi mana yang lebih primary
- Jangan paksain ada satu jawaban

### 5. Tag confidence

Untuk setiap klaim, kasih level:
- ✅ **Confirmed** — dari sumber tier 1
- ⚠️ **Likely** — dari tier 2-3, masih kuat tapi cek ulang kalau krusial
- ❓ **Unverified** — gak ketemu sumber kuat, anggap perkiraan

### 6. Format output

```markdown
## Jawaban singkat
[1-2 kalimat, jawab langsung pertanyaan user]

## Detail dengan sumber

✅ [Klaim 1]. ([Sumber 1])

✅ [Klaim 2]. ([Sumber 2])

⚠️ [Klaim 3 — likely tapi cek ulang]. ([Sumber 3])

## Yang TIDAK ditemukan
- [Aspek yang user tanya tapi gak ada sumber kuat]

## Catatan
- Tanggal akses sumber: [date]
- [Caveat lain kalau ada, misal: "Halaman ini di-update terakhir 2024, mungkin sudah berubah"]
```

## Pitfalls

### Pitfall 1: Halu URL

LLM cenderung **bikin URL plausible** yang sebenernya gak ada. Mitigasi:
- HANYA kutip URL yang muncul DI HASIL TOOL `web_search` di turn ini
- Kalau gak ada, JANGAN bikin URL — bilang "sumber tidak ditemukan"
- Test cepat: kalau URL terlihat "terlalu rapi" (misal `bps.go.id/data-sumedang-2025`), curigai

### Pitfall 2: Date bias

Hasil search bisa nyampurin info lama dan baru. Mitigasi:
- Cek `published_date` / footer halaman
- Untuk topik time-sensitive, sebutkan tanggal data eksplisit
- Kalau gak ada date di sumber, bilang "publish date unknown"

### Pitfall 3: Statistik dari sumber sekunder

Banyak artikel kutip statistik tanpa link ke primary source. Jangan rantai kutipan:
- ❌ "Per artikel kompas.com (yang ngutip BPS), populasi adalah X"
- ✅ Open BPS asli → cek angkanya → kutip BPS langsung
- Kalau primary source gak diakses (paywall, butuh login), bilang gitu

### Pitfall 4: User tekan untuk jawaban pasti

Kalau user push ("kasih aja angkanya, gak usah disclaimer"), JANGAN drop standar. Tetap kasih jawaban dengan flag uncertainty. Kalau user mau angka pasti, dia bisa ke sumber primer sendiri.

## Verification

Setelah jawaban siap, self-check:

1. Apakah setiap angka spesifik punya URL backing? (Y/N)
2. Apakah setiap kutipan dari orang nyata punya URL backing? (Y/N)
3. Apakah ada klaim "X sudah berubah jadi Y di [tahun]" yang URL-nya gak nyatain perubahan tersebut? (kalau ya — fix)
4. Apakah ada section "Yang TIDAK ditemukan" yang jujur? (kalau seluruh jawaban "smooth", curigai diri sendiri — biasanya ada gap)

Kalau ada satu "N" di check 1-3, **tahan jawaban**, redo search atau drop klaim.

## Output Examples

### Contoh OK

User: "Berapa context window DeepSeek V4 Flash?"

```
## Jawaban singkat
Berdasarkan beberapa sumber sekunder, DeepSeek V4 Flash punya context window 1M token. Konfirmasi dari sumber primer DeepSeek perlu dicek terpisah.

## Detail
⚠️ DeepSeek V4 Flash dilaporkan punya 1M-token context window oleh listing pihak ketiga seperti OpenRouter dan beberapa blog tinjauan ([1], [2]). Saya belum temukan halaman model resmi DeepSeek yang jelas mengkonfirmasi angka 1M, jadi sebaiknya verifikasi langsung di halaman pricing/model official DeepSeek sebelum mengandalkan angka ini untuk konfigurasi production.

[1] https://openrouter.ai/deepseek/deepseek-v4-flash (diakses [DATE])
[2] https://codersera.com/blog/deepseek-v4-flash-deep-dive (diakses [DATE])

## Yang TIDAK ditemukan
- Halaman dokumentasi DeepSeek resmi yang menyatakan 1M context spesifik untuk Flash
```

### Contoh BURUK (jangan kayak gini)

```
DeepSeek V4 Flash punya context window 1 juta token. Modelnya dirilis tahun 2026 dengan arsitektur Mixture-of-Experts dan menjadi salah satu model paling efisien.
```
(Tidak ada source. Klaim spesifik tahun. "Salah satu paling efisien" tanpa benchmark. Halu.)
