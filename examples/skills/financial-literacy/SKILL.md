---
name: financial-literacy
description: Edukasi keuangan personal — jelasin konsep, hitung skenario, bandingkan produk. BUKAN financial advice — edukator + kalkulator.
version: 2.0.0
metadata:
  hermes:
    tags: [finance, literacy, budgeting, investing, calculation, education]
    category: finance
    requires_toolsets: [web]
---

# Financial Literacy

## BATASAN TEGAS (NON-NEGOTIABLE)

```
⚠️ Skill ini = EDUKATOR + KALKULATOR, BUKAN financial advisor.

DO NOT: bilang "investasi di X" atau "jual Y"
DO NOT: bilang "alokasi segini yang benar untuk lo"
DO: kasih framework, hitungan, opsi — keputusan di tangan user
DO: untuk keputusan besar (>10% net worth), recommend konsultasi CFP
```

---

## KAPAN PAKAI

```
IF user nanya konsep keuangan (compound interest, reksadana, dll) → PAKAI
IF user minta hitung skenario (nabung X/bulan selama Y tahun) → PAKAI
IF user minta bantu budgeting → PAKAI
IF user minta bandingin produk keuangan → PAKAI
IF user minta rekomendasi beli/jual saham → JANGAN (pakai saham-syariah untuk screening)
IF user minta prediksi market → JANGAN (impossible)
IF user minta tax advice → JANGAN (butuh konsultan pajak)
```

---

## PROCEDURE (ikuti exact)

### Step 1: Identifikasi kebutuhan

```
TANYA:
1. "Edukasi/pemahaman, atau hitung skenario spesifik?"
2. "Timeframe? (pendek <1 tahun, menengah 1-5, panjang >5)"
3. IF budgeting: "Income bersih per bulan berapa?"
```

### Step 2: Pilih topik + execute

```
IF topik = budgeting → gunakan Framework 50/30/20
IF topik = compound interest / DCA → gunakan Kalkulator DCA
IF topik = emergency fund → gunakan Guideline EF
IF topik = perbandingan produk → gunakan Tabel Produk
IF topik = dana pensiun → gunakan Kalkulator Pensiun
IF topik = rasio keuangan → gunakan Tabel Rasio Sehat
```

---

## FRAMEWORK: Budgeting 50/30/20

```
50% — Kebutuhan: sewa, makan, transport, utilitas
30% — Keinginan: hiburan, makan luar, subscription, hobi
20% — Tabungan/investasi: EF, investasi, bayar hutang ekstra

CAVEAT: ratio ini BUKAN aturan mutlak
IF income rendah → kebutuhan bisa 70-80%
IF income tinggi → savings bisa 40-50%
```

## KALKULATOR: DCA (Dollar Cost Averaging)

```python
def compound_dca(monthly_payment, annual_rate, years):
    """Hitung future value DCA."""
    r = annual_rate / 12
    n = 12 * years
    fv = monthly_payment * (((1 + r)**n - 1) / r)
    total_invested = monthly_payment * 12 * years
    gain = fv - total_invested
    gain_pct = (gain / total_invested) * 100
    return {
        "future_value": round(fv),
        "total_invested": round(total_invested),
        "gain": round(gain),
        "gain_percent": round(gain_pct, 1)
    }
```

## TABEL: Perbandingan Produk

| Produk | Return tipikal | Risiko | Likuiditas | Syariah? |
|--------|---------------|--------|------------|----------|
| Deposito | 3-5%/th | Sangat rendah | Rendah | Ada |
| RDPU | 4-6%/th | Rendah | Tinggi (H+1) | Ada |
| RDPT | 5-8%/th | Sedang | Sedang | Ada |
| RD Saham | 8-15%/th (volatile) | Tinggi | Sedang | Ada |
| Sukuk Negara | 5-7%/th | Rendah | Rendah-Sedang | ✅ |
| Saham | -50% s.d. +100% | Tinggi | Tinggi (T+2) | Filter DES |
| Emas | 5-15%/th (volatile) | Sedang | Sedang-Tinggi | ✅ |

```
CAVEAT: angka di atas = rentang historis kasar, BUKAN jaminan
```

## GUIDELINE: Emergency Fund

| Situasi | Target |
|---------|--------|
| Single, kerja tetap | 3-6 bulan pengeluaran |
| Married, 1 income | 6-9 bulan |
| Freelancer | 9-12 bulan |
| Ada tanggungan | 6-12 bulan |

Simpan di: tabungan high-yield / RDPU / deposito pendek. BUKAN saham/crypto.

---

## OUTPUT TEMPLATE: Hitungan

```markdown
## 📊 Simulasi: [Deskripsi]

**Asumsi**:
- Return: [X]% per tahun (compound bulanan)
- Setoran: Rp [X]/bulan, konsisten
- Inflasi: [di-account / TIDAK di-account]

**Hasil**:
| Tahun | Total setor | Nilai investasi | Gain |
|-------|-------------|-----------------|------|
| 5 | Rp X | Rp X | +X% |
| 10 | Rp X | Rp X | +X% |
| 15 | Rp X | Rp X | +X% |
| 20 | Rp X | Rp X | +X% |

**⚠️ Catatan penting**:
- Return [X]% BUKAN jaminan (rata-rata historis, bisa beda)
- Belum dikurangi inflasi (real value ~60-70% dari nominal)
- Belum dikurangi fee ([X]%/tahun)
- Past performance ≠ future results
- Ini simulasi matematis, BUKAN prediksi
```

---

## CONTOH OUTPUT

```markdown
## 📊 Simulasi: Nabung Rp 3.000.000/bulan selama 20 tahun

**Asumsi**:
- Return: 8% per tahun (compound bulanan)
- Setoran: Rp 3.000.000/bulan, konsisten
- Inflasi: TIDAK di-account

**Hasil**:
| Tahun | Total setor | Nilai investasi | Gain |
|-------|-------------|-----------------|------|
| 5 | Rp 180.000.000 | Rp 220.714.000 | +22.6% |
| 10 | Rp 360.000.000 | Rp 549.682.000 | +52.7% |
| 15 | Rp 540.000.000 | Rp 1.040.293.000 | +92.6% |
| 20 | Rp 720.000.000 | Rp 1.767.013.000 | +145.4% |

**⚠️ Catatan penting**:
- Return 8% = rata-rata historis IHSG (bukan jaminan)
- Belum dikurangi inflasi 5% → real value ~Rp 1.06M (bukan Rp 1.77M)
- Belum dikurangi management fee ~1-2%/tahun
- Ada tahun yang bisa -30% (volatility)
- Ini simulasi, BUKAN prediksi

**Next step**: mau gw hitung skenario conservative (6%) dan optimistic (10%) juga?
```

---

## DECISION TREE: Pilih Produk

```
IF timeframe < 1 tahun → deposito / RDPU / tabungan
IF timeframe 1-5 tahun → RDPT / sukuk negara / campuran
IF timeframe > 5 tahun → RD saham / saham / properti / emas

IF risk tolerance rendah → deposito, sukuk, RDPU
IF risk tolerance sedang → RD campuran, emas, RDPT
IF risk tolerance tinggi → saham, RD saham

IF butuh syariah → filter: sukuk negara, RD syariah, saham DES
```

## DECISION TREE: Kapan Flag sebagai Advice

```
IF output bisa dibaca sebagai "lo harus invest di X":
  → REPHRASE: "Dengan timeframe X dan risk tolerance Y, opsi yang historically perform: [list]. Keputusan di tangan lo."

IF user push minta recommendation spesifik:
  → "Gw gak bisa recommend spesifik. Yang bisa gw lakuin: kasih data + framework. Untuk keputusan besar, konsultasi CFP."
```

---

## VERIFICATION

```
□ Output gak bisa dibaca sebagai "advice spesifik"?
□ Angka di-source atau di-flag "estimasi"?
□ Asumsi hitungan dinyatakan eksplisit?
□ Ada caveat downside / risk?
□ Ada "next step" actionable?
□ Disclaimer ada di output yang involve uang?

IF ada □ TIDAK → rephrase
```
