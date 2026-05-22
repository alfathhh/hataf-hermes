---
name: pdf-summarize
description: Baca PDF (paper/laporan/regulasi) dan kasih summary terstruktur dengan referensi halaman. Setiap klaim harus traceable ke section/halaman PDF.
version: 1.0.0
metadata:
  hermes:
    tags: [pdf, summary, document]
    category: daily-task
---

# PDF Summarize

Skill untuk membaca dokumen PDF (paper akademik, laporan resmi, kontrak, regulasi) dan menghasilkan summary terstruktur. Setiap poin di summary harus bisa di-trace ke halaman atau section spesifik di PDF.

## When to Use

User memberikan:
- File PDF lokal (path file)
- URL PDF
- Permintaan "summarize this paper / report / contract"

JANGAN pakai skill ini untuk:
- HTML / web article (pakai `web_extract`)
- Image-heavy PDF (scan KTP, dll) — itu butuh OCR + vision, beda alur

## Procedure

### 1. Klarifikasi tujuan summary

Tanya user (kalau gak jelas):
- "Mau summary umum, atau fokus aspek tertentu? (metodologi / hasil / risiko / dst)"
- "Format mau bullet, narrative, atau executive summary?"
- "Berapa panjang? (1 paragraf / 1 halaman / detailed)"

### 2. Extract isi PDF

Pakai built-in tool. Cara:

```python
# Hermes punya tool untuk read file. PDF teks bisa langsung di-read.
# Untuk PDF yang lokasinya remote (URL):
#   1. Download dulu via terminal: curl -o /tmp/doc.pdf <URL>
#   2. Baru read /tmp/doc.pdf
```

Kalau PDF ternyata image-only (scan), tool read bakal kasih hasil kosong/garbage. Kasih tau user dan stop — minta versi text-extractable atau OCR dulu.

### 3. Structure detection

Identifikasi struktur dokumen:
- **Paper akademis**: Abstract, Introduction, Methods, Results, Discussion, References
- **Laporan teknis**: Executive Summary, Sections, Appendices
- **Kontrak**: Pasal/Article 1, 2, 3, ..., Definitions, Termination clause
- **Regulasi**: Bab, Pasal, Ayat
- **Buku**: Chapter, Subchapter

Catat halaman/section dari section yang penting.

### 4. Summarize per section

Untuk setiap section utama, generate 2-4 kalimat ringkas. Format:

```markdown
### [Nama Section] (hal. X – Y)
[2-4 kalimat key takeaway. Setiap klaim harus akurat, jangan "kira-kira".]
```

Kalau bagian penting punya angka/data, **kutip persis** dari PDF, jangan paraphrase angka.

### 5. Hasil overall

Output structure:

```markdown
# Summary: [Judul Dokumen]

**Sumber**: [path/URL] | **Halaman**: [N total]
**Tanggal akses**: [hari ini]

## TL;DR
[1 paragraf, max 4 kalimat]

## Klaim utama
1. [Klaim 1] (hal. X)
2. [Klaim 2] (hal. Y)
3. [Klaim 3] (hal. Z)

## Per section
[Loop section]

## Hal yang harus dicek pembaca
- [Kalau ada angka kritis, asumsi penting, atau klaim yang patut diverifikasi sendiri]

## Yang TIDAK ada di dokumen
[Penting kalau user nanya hal spesifik tapi gak dibahas — bilang gak ada]
```

### 6. Section "Yang TIDAK ada"

Ini krusial untuk anti-halu. Setelah summary, evaluate:
- Apakah dokumen ini punya gap? (misal: paper bahas hasil tapi gak bahas limitations)
- Apakah ada klaim asumsi yang gak di-back?
- Apakah ada referensi ke dokumen lain yang user butuh?

List itu di section "Yang TIDAK ada di dokumen" atau "Yang sebaiknya pembaca cek".

## Pitfalls

### Pitfall 1: Salah halaman

LLM bisa salah inget halaman. Mitigasi:
- Kalau ragu, sebut "sekitar hal. X" atau "di awal/tengah/akhir dokumen"
- Lebih baik vague tapi accurate daripada spesifik tapi salah

### Pitfall 2: Paraphrase angka

JANGAN paraphrase angka. "Sekitar separuh" bukan pengganti "47.3%". Kalau PDF tulis 47.3%, summary tulis 47.3%.

### Pitfall 3: Halu klaim yang gak ada di dokumen

LLM cenderung "lengkapin" dengan informasi general yang masuk akal tapi gak di PDF. Self-check:
- Apakah klaim ini ADA di teks PDF, atau gw tambahin sendiri dari pengetahuan umum?
- Kalau yang kedua, drop atau flag eksplisit "berdasarkan konteks umum, bukan dari dokumen"

### Pitfall 4: PDF tabel / chart kecil

PDF yang banyak tabel angka, tool read teks bisa nge-flatten tabel jadi messy. Kalau angka di tabel kritis, beritahu user "tabel di hal. X gak ke-extract bersih, sebaiknya cek manual".

### Pitfall 5: PDF dengan track changes / komentar

Kalau ada strikethrough atau revision marks, sebutkan eksplisit. Jangan diam-diam kasih versi final tanpa flag bahwa ada draft state.

## Verification

Self-check sebelum kirim:

1. Apakah setiap klaim di summary ada di teks PDF? (cek 2-3 sample)
2. Apakah angka kunci sama persis dengan PDF? (cek 1-2 sample)
3. Apakah ada section "Yang TIDAK ada" — atau jujur "saya gak nemu gap penting"?
4. Apakah dokumen punya tanggal/version, dan udah disebutkan?

## Untuk PDF panjang (> 50 halaman)

PDF besar bisa overflow context. Strategy:

1. Read overview (TOC, abstract, intro, conclusion) dulu — beberapa kilo token
2. Identifikasi section yang user paling butuh
3. Read **section spesifik** itu detail
4. Summary disusun progressive: overview → key sections → conclusion

Kalau context masih kepenuhan, jalankan `/compress` antar tahap.

## Untuk PDF non-Indonesia/non-Inggris

Default: summary keluar dalam bahasa user (Indonesia kalau user Indo). Kalau dokumennya bahasa lain (Mandarin, Arab, Belanda), kerjakan di bahasa asli dulu (jangan auto-translate sembrono — istilah teknis bisa salah), summary akhir di bahasa user dengan caveat "diterjemahkan dari [bahasa asal]".
