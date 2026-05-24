---
name: pdf-summarize
description: Baca PDF (paper/laporan/regulasi) dan kasih summary terstruktur dengan referensi halaman. Setiap klaim traceable ke section PDF.
version: 2.0.0
metadata:
  hermes:
    tags: [pdf, summary, document]
    category: daily-task
---

# PDF Summarize

## KAPAN PAKAI

```
IF user kasih PDF file/URL + minta summary → PAKAI
IF user minta "ringkas paper/report/kontrak" → PAKAI
IF user kasih HTML article → JANGAN (pakai web_extract langsung)
IF user kasih image-heavy PDF (scan KTP) → JANGAN (butuh OCR, beda alur)
```

---

## PROCEDURE (ikuti exact)

### Step 1: Klarifikasi tujuan

```
TANYA (kalau gak jelas):
1. "Summary umum, atau fokus aspek tertentu?" (metodologi/hasil/risiko)
2. "Format: bullet / narrative / executive summary?"
3. "Panjang: 1 paragraf / 1 halaman / detailed?"
```

### Step 2: Load PDF

```python
# File lokal
read_file("path/to/document.pdf")

# URL
terminal("curl -o /tmp/doc.pdf [URL]")
read_file("/tmp/doc.pdf")
```

```
IF hasil kosong / garbage → PDF image-only → bilang user: "PDF ini scan/image, butuh OCR dulu"
IF file terlalu besar (>50 halaman) → gunakan strategy di bawah
```

### Step 3: Detect structure

```
IF paper akademis → sections: Abstract, Intro, Methods, Results, Discussion, References
IF laporan teknis → sections: Executive Summary, Sections, Appendices
IF kontrak → sections: Pasal 1, 2, 3..., Definitions, Termination
IF regulasi → sections: Bab, Pasal, Ayat
IF buku → sections: Chapter, Subchapter
```

### Step 4: Summarize per section (2-4 kalimat each)

```
RULE: Setiap klaim HARUS dari teks PDF (bukan dari pengetahuan umum)
RULE: Angka/data KUTIP PERSIS (jangan paraphrase "sekitar separuh" kalau PDF bilang 47.3%)
RULE: Catat halaman/section untuk setiap poin
DO NOT: tambah informasi yang GAK ADA di PDF
```

### Step 5: Output

---

## OUTPUT TEMPLATE

```markdown
# Summary: [Judul Dokumen]

**Sumber**: [path/URL] | **Halaman**: [N total]
**Tanggal akses**: [hari ini]

## TL;DR (max 4 kalimat)
[Inti dokumen]

## Klaim utama
1. [Klaim 1] (hal. X)
2. [Klaim 2] (hal. Y)
3. [Klaim 3] (hal. Z)

## Per section

### [Section Name] (hal. X–Y)
[2-4 kalimat key takeaway]

### [Section Name] (hal. X–Y)
[2-4 kalimat]

## Yang TIDAK ada di dokumen
- [Gap 1: hal yang mungkin user expect tapi gak dibahas]
- [Gap 2]

## Catatan
- [Limitasi: tabel hal X gak ke-extract bersih]
- [Uncertainty: "sekitar hal. X" kalau gak yakin exact page]
```

---

## CONTOH OUTPUT

```markdown
# Summary: "Laporan Keuangan PT XYZ Q1 2026"

**Sumber**: ~/Documents/lapkeu-xyz-q1-2026.pdf | **Halaman**: 32
**Tanggal akses**: 24 Mei 2026

## TL;DR
Revenue PT XYZ Q1 2026 naik 15% YoY ke Rp 4.7T. Net profit margin turun dari 12% ke 9% karena ekspansi warehouse. Cash position masih sehat (Rp 2.1T).

## Klaim utama
1. Revenue Rp 4.7 Triliun (+15% YoY) (hal. 5)
2. Net profit margin 9% (turun dari 12% Q1 2025) (hal. 8)
3. CAPEX Rp 800M untuk 3 warehouse baru (hal. 14)
4. Cash & equivalents Rp 2.1T (hal. 19)

## Per section

### Ikhtisar Keuangan (hal. 5-9)
Revenue naik 15% didorong segment e-commerce logistics (+23%). Gross margin stabil 28%. Net margin turun ke 9% karena beban depresiasi warehouse baru.

### Operasional (hal. 10-16)
Volume pengiriman naik 20% ke 45 juta paket. Ekspansi ke 3 kota baru (Makassar, Balikpapan, Manado). Hiring 1,200 kurir baru.

### Posisi Keuangan (hal. 17-22)
Total aset Rp 12.3T. DER 0.4x (sehat). Cash Rp 2.1T cukup untuk 18 bulan operasi tanpa revenue.

## Yang TIDAK ada di dokumen
- Proyeksi Q2-Q4 2026 (gak disebutkan)
- Detail per-kota revenue breakdown
- Strategi pricing ke depan

## Catatan
- Tabel hal. 20 (aging receivables) gak ke-extract bersih, cek manual
- Angka dikutip persis dari dokumen
```

---

## DECISION TREE: PDF Panjang (>50 halaman)

```
IF PDF > 50 halaman:
  1. Read overview dulu: TOC, abstract, intro, conclusion
  2. Identifikasi section yang user paling butuh
  3. Read section spesifik itu detail
  4. Summary progressive: overview → key sections → conclusion
  IF context masih overload → chunk per section, summarize each
```

## DECISION TREE: Angka & Data

```
IF PDF tulis "47.3%" → summary tulis "47.3%" (EXACT)
DO NOT: paraphrase "sekitar separuh"
IF PDF punya tabel penting → kutip angka key dari tabel
IF tabel gak ke-extract bersih → flag: "tabel hal. X perlu dicek manual"
```

## DECISION TREE: Klaim dari luar PDF

```
IF klaim ada di teks PDF → deliver
IF klaim dari pengetahuan umum gw → DO NOT include, atau flag:
  "⚠️ Ini BUKAN dari dokumen — ini konteks umum: [klaim]"
IF user tanya hal yang gak ada di PDF:
  → "Hal ini TIDAK dibahas di dokumen yang lo kasih."
```

---

## VERIFICATION

```
□ Setiap klaim di summary ada di teks PDF?
□ Angka kunci sama persis dengan PDF?
□ Ada section "Yang TIDAK ada"?
□ Halaman/section reference akurat?
□ Gak ada info yang gw tambahin dari luar PDF?

IF ada □ TIDAK → fix
```
