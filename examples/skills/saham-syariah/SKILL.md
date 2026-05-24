---
name: saham-syariah
description: Screening dan analisis saham syariah Indonesia (IDX). Filter DES OJK, analisis fundamental. BUKAN financial advice — data + screening only.
version: 2.0.0
metadata:
  hermes:
    tags: [saham, syariah, idx, investment, screening, fundamental, ojk]
    category: finance
    requires_toolsets: [web]
---

# Analisis Saham Syariah Indonesia

## DISCLAIMER (WAJIB di setiap output)

```
⚠️ DISCLAIMER: Output ini BUKAN nasihat investasi. Hasil screening otomatis
berdasarkan data publik. DYOR. Konsultasi penasihat keuangan berlisensi.
Past performance ≠ future results.
```

---

## KAPAN PAKAI

```
IF user minta screening saham syariah → PAKAI
IF user minta analisis fundamental saham IDX → PAKAI
IF user minta cek apakah saham masuk DES → PAKAI
IF user minta rekomendasi beli/jual → JANGAN (bilang "gak bisa recommend")
IF user minta prediksi harga → JANGAN (impossible)
IF user minta saham luar negeri → JANGAN (skill ini fokus IDX)
```

---

## PROCEDURE (ikuti exact)

### Step 1: Validasi status syariah (LANGKAH PERTAMA, WAJIB)

```python
# Cek DES dari OJK
web_extract(
    url="https://www.ojk.go.id/id/kanal/syariah/data-dan-statistik/daftar-efek-syariah/",
    prompt="Extract SK DES terbaru: nomor, tanggal berlaku, jumlah efek"
)
```

```
IF saham TIDAK di DES:
  OUTPUT: "❌ [KODE] TIDAK masuk DES. Tidak dilanjutkan analisis."
  STOP. Jangan lanjut.

IF saham di DES:
  → lanjut Step 2
```

### Step 2: Ambil data fundamental

```python
# Yahoo Finance
web_extract(
    url="https://finance.yahoo.com/quote/[KODE].JK/",
    prompt="Extract: harga, market cap, PER, PBV, dividend yield, 52-week high/low"
)

# ATAU Sectors.app (lebih lengkap untuk IDX)
web_extract(
    url="https://sectors.app/id/stocks/[KODE]",
    prompt="Extract: PER, PBV, ROE, ROA, DER, DY, EPS growth, revenue growth"
)
```

```
IF data gak ketemu → bilang "data tidak tersedia", DO NOT perkirakan
IF angka dari 2 sumber beda → flag discrepancy
```

### Step 3: Screening criteria

```
VALUE (undervalued):
- PER < 15x (vs rata-rata sektor)
- PBV < 1.5x
- DY > 4% (konsisten 3 tahun)

QUALITY (fundamental kuat):
- ROE > 15% (konsisten 3-5 tahun)
- DER < 1.0
- Revenue growth > 10% YoY
- Free Cash Flow positif

SYARIAH-SPECIFIC:
- Interest-bearing debt / total assets ≤ 45%
- Non-halal income / total revenue ≤ 10%
- IF DER 40-44% → ⚠️ FLAG "mendekati batas, risiko keluar DES"
```

### Step 4: Output

---

## OUTPUT TEMPLATE: Analisis Individual

```markdown
## Analisis Saham Syariah: [KODE] — [Nama]

⚠️ DISCLAIMER: Bukan nasihat investasi. DYOR.

### Status Syariah
✅ Masuk DES OJK Periode [X] (SK No. [Y])
✅ Indeks: [JII / JII70 / ISSI]

### Harga
| Metrik | Nilai |
|--------|-------|
| Harga terakhir | Rp X |
| 52-week high | Rp X |
| 52-week low | Rp X |
| Market cap | Rp X T |

### Fundamental
| Metrik | Nilai | Penilaian |
|--------|-------|-----------|
| PER | Xx | [Murah/Wajar/Mahal] |
| PBV | Xx | [Murah/Wajar/Mahal] |
| ROE | X% | [Bagus/Cukup] |
| DER | Xx | [Aman/Perlu perhatian] |
| DY | X% | [Menarik/Biasa] |

### Risiko
1. [Risiko 1]
2. [Risiko 2]

### Sumber
- [URL] (diakses [tanggal])
```

## OUTPUT TEMPLATE: Screening Batch

```markdown
## Screening Saham Syariah — [Tanggal]

**Kriteria**: [yang user minta]
**Universe**: [JII / JII70 / ISSI]
**Lolos filter**: [N] saham

| # | Kode | Nama | PER | PBV | ROE | DY | DER |
|---|------|------|-----|-----|-----|-----|-----|
| 1 | XXXX | ... | ... | ... | ... | ... | ... |
| 2 | YYYY | ... | ... | ... | ... | ... | ... |

⚠️ DISCLAIMER: Screening otomatis. Bukan rekomendasi. DYOR.
```

---

## CONTOH OUTPUT

```markdown
## Analisis Saham Syariah: TLKM — Telkom Indonesia

⚠️ DISCLAIMER: Bukan nasihat investasi. DYOR.

### Status Syariah
✅ Masuk DES OJK Periode Mei 2026
✅ Indeks: JII, JII70, ISSI

### Harga
| Metrik | Nilai |
|--------|-------|
| Harga terakhir | Rp 3.850 |
| 52-week high | Rp 4.200 |
| 52-week low | Rp 3.100 |
| Market cap | Rp 381 T |

### Fundamental
| Metrik | Nilai | Penilaian |
|--------|-------|-----------|
| PER | 14.2x | Wajar (sektor telco avg 15x) |
| PBV | 2.8x | Wajar |
| ROE | 19.7% | Bagus |
| DER | 0.52x | Aman (jauh dari batas 45%) |
| DY | 4.8% | Menarik |

### Risiko
1. Revenue growth melambat (5% vs 8% tahun lalu)
2. Kompetisi harga data dari Starlink entry
3. CAPEX tinggi untuk fiber expansion

### Sumber
- https://finance.yahoo.com/quote/TLKM.JK/ (diakses 24 Mei 2026)
- https://sectors.app/id/stocks/TLKM (diakses 24 Mei 2026)
```

---

## DECISION TREE: User Minta Rekomendasi

```
IF user bilang "saham apa yang bagus buat dibeli?":
  → "Gw gak bisa recommend beli/jual. Yang bisa gw lakuin: screening berdasarkan kriteria lo. Mau screening dengan kriteria apa? (DY tinggi? PER murah? ROE bagus?)"

IF user push "pasti untung":
  → "Gak ada saham yang pasti untung. Saham bisa turun 50% walaupun syariah. Yang bisa gw kasih: data + screening. Keputusan di tangan lo."

IF user minta target harga:
  → "Gw gak bisa prediksi harga. Yang bisa: kasih data historis + fundamental. Untuk target harga, konsultasi analis berlisensi."
```

## DECISION TREE: Indeks

```
DES = Daftar SEMUA efek syariah (ratusan)
ISSI = Indeks semua saham DES di IDX
JII70 = 70 saham syariah paling likuid
JII = 30 saham syariah paling likuid (subset JII70)

Hierarki: DES ⊃ ISSI ⊃ JII70 ⊃ JII
```

---

## VERIFICATION

```
□ SETIAP saham dikonfirmasi masuk DES?
□ Angka fundamental dari tool result (bukan memori)?
□ Disclaimer ada?
□ Gak ada statement yang bisa dibaca "rekomendasi beli"?
□ Sumber dikutip dengan URL + tanggal?
□ Risiko disebutkan (bukan cuma positif)?

IF ada □ TIDAK → fix
```
