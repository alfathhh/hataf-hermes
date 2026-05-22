---
name: financial-literacy
description: Edukasi keuangan personal — jelasin konsep, hitung skenario (compound interest, DCA, budgeting, emergency fund), bandingkan produk keuangan (deposito/reksadana/sukuk/saham). BUKAN financial advice — edukator + kalkulator, bukan advisor.
version: 1.0.0
metadata:
  hermes:
    tags: [finance, literacy, budgeting, investing, calculation, education]
    category: finance
    requires_toolsets: [web]
---

# Financial Literacy

Skill edukasi keuangan personal. Fungsi: jelasin konsep, hitung skenario, bandingin produk, bantu organize budget. BUKAN memberikan nasihat investasi personal.

## BATASAN TEGAS

```
⚠️ BATASAN: Skill ini adalah EDUKATOR dan KALKULATOR, bukan financial advisor.
- Gw TIDAK akan bilang "investasi di X" atau "jual Y"
- Gw TIDAK akan bilang "alokasi segini yang benar untuk lo"
- Gw AKAN kasih framework, hitungan, dan opsi — keputusan di tangan lo
- Untuk keputusan besar (>10% net worth), konsultasi Perencana Keuangan bersertifikasi (CFP/RFP)
```

## When to Use

- User nanya konsep keuangan ("apa itu compound interest?", "bedanya reksadana pasar uang vs pendapatan tetap?")
- User minta hitung skenario ("kalau nabung 3jt/bulan selama 20 tahun di return 8%, dapet berapa?")
- User minta bantu budgeting ("income gw 15jt, pengeluaran segini, gimana alokasinya?")
- User minta bandingin produk keuangan (deposito vs sukuk vs reksadana vs saham)
- User nanya soal emergency fund, dana pensiun, dana pendidikan anak
- User nanya rasio keuangan sehat (debt ratio, savings rate, dll)

JANGAN pakai untuk:
- Rekomendasi beli/jual saham spesifik (pakai `saham-syariah` untuk screening)
- Prediksi market / harga
- Tax advisory (butuh konsultan pajak)
- Asuransi recommendation spesifik (butuh agen berlisensi)

## Sumber Data

| Data | Sumber | URL |
|------|--------|-----|
| Suku bunga deposito bank | OJK / bank websites | https://www.ojk.go.id/ |
| BI Rate / suku bunga acuan | Bank Indonesia | https://www.bi.go.id/id/statistik/indikator/bi-rate.aspx |
| Yield sukuk negara (SR/ST) | DJPPR Kemenkeu | https://www.djppr.kemenkeu.go.id/ |
| Inflasi Indonesia | BPS | https://www.bps.go.id/ |
| NAB reksadana | Bareksa / OJK | https://www.bareksa.com/ |
| IHSG / indeks | IDX | https://www.idx.co.id/ |

## Procedure

### 1. Identifikasi kebutuhan

Tanya:
- "Lo nanya untuk edukasi/pemahaman, atau butuh hitung skenario spesifik?"
- "Situasi keuangan lo gimana secara umum? (income range, ada tanggungan, ada hutang?)" — optional, cuma kalau relevan ke hitungan
- "Timeframe? (jangka pendek <1 tahun, menengah 1-5 tahun, panjang >5 tahun?)"

### 2. Framework per topik

---

## TOPIK: Budgeting

### Framework 50/30/20 (baseline, BUKAN aturan mutlak)

```
50% — Kebutuhan (needs): sewa/cicilan, makan, transport, utilitas, asuransi wajib
30% — Keinginan (wants): hiburan, makan luar, subscription, hobi
20% — Tabungan & investasi: emergency fund, investasi, bayar hutang ekstra
```

**Catatan jujur**: ratio ini dari Elizabeth Warren (buku "All Your Worth", 2005). Populer tapi BUKAN satu-satunya framework. Untuk income rendah, 50% kebutuhan bisa gak realistis (bisa 70-80%). Untuk income tinggi, 20% savings mungkin terlalu rendah. **Adapt ke situasi real.**

### Output format budgeting

```markdown
## Budget Breakdown

**Income bersih**: Rp [X]/bulan

| Kategori | Alokasi | Nominal | Catatan |
|----------|---------|---------|---------|
| 🏠 Kebutuhan | [X]% | Rp [Y] | [breakdown sub-kategori] |
| 🎮 Keinginan | [X]% | Rp [Y] | [breakdown] |
| 💰 Tabungan/Investasi | [X]% | Rp [Y] | [breakdown] |

### Rekomendasi prioritas
1. [Prioritas 1 — biasanya emergency fund kalau belum ada]
2. [Prioritas 2]
3. [Prioritas 3]

### Caveat
- Ratio ini baseline. Kalau [situasi X], adjust ke [Y].
- Untuk keputusan alokasi investasi spesifik, konsultasi CFP.
```

---

## TOPIK: Emergency Fund

### Guideline umum

| Situasi | Target emergency fund |
|---------|----------------------|
| Single, kerja tetap | 3-6 bulan pengeluaran |
| Married, 1 income source | 6-9 bulan pengeluaran |
| Freelancer / gig worker | 9-12 bulan pengeluaran |
| Ada tanggungan (anak, ortu) | 6-12 bulan pengeluaran |

**Di mana simpan**: instrumen LIKUID + AMAN:
- Tabungan high-yield / money market
- Deposito (tenor pendek 1-3 bulan, auto-rollover)
- Reksadana pasar uang (bisa dicairkan H+1 sampai H+3)

**BUKAN**: saham, reksadana saham, crypto, properti (tidak likuid / volatile)

---

## TOPIK: Compound Interest Calculator

### Formula

```
FV = PV × (1 + r/n)^(n×t)

Di mana:
FV = Future Value (nilai akhir)
PV = Present Value (modal awal)
r  = annual interest rate (desimal, misal 8% = 0.08)
n  = compounding frequency per tahun (12 kalau bulanan)
t  = tahun
```

### Untuk DCA (nabung rutin bulanan)

```
FV = PMT × [((1 + r/n)^(n×t) - 1) / (r/n)]

Di mana:
PMT = setoran per bulan
```

### Contoh output

```markdown
## Simulasi: Nabung Rp 3.000.000/bulan selama 20 tahun

**Asumsi**:
- Return: 8% per tahun (compound bulanan)
- Setoran: Rp 3.000.000/bulan, konsisten
- Inflasi: TIDAK di-account (lihat catatan)

**Hasil**:
| Tahun | Total setoran | Nilai investasi | Gain |
|-------|---------------|-----------------|------|
| 5     | Rp 180.000.000 | Rp 220.713.600 | +22.6% |
| 10    | Rp 360.000.000 | Rp 549.681.600 | +52.7% |
| 15    | Rp 540.000.000 | Rp 1.040.293.200 | +92.6% |
| 20    | Rp 720.000.000 | Rp 1.767.013.200 | +145.4% |

**Catatan penting**:
- Return 8%/tahun BUKAN jaminan. Ini rata-rata historis IHSG (perlu dicek ulang).
- Angka di atas BELUM dikurangi inflasi. Dengan inflasi 5%, nilai riil ~60-70% dari nominal.
- Angka di atas BELUM dikurangi fee (management fee reksadana ~1-2%/tahun).
- Past performance ≠ future results.
- Ini simulasi matematis, BUKAN prediksi.
```

### Implementasi kalkulator (Python)

```python
def compound_dca(monthly_payment, annual_rate, years, compound_freq=12):
    """Hitung future value DCA (Dollar Cost Averaging)."""
    r = annual_rate / compound_freq
    n = compound_freq * years
    
    # FV of annuity formula
    fv = monthly_payment * (((1 + r)**n - 1) / r)
    total_invested = monthly_payment * compound_freq * years
    gain = fv - total_invested
    gain_pct = (gain / total_invested) * 100
    
    return {
        "future_value": round(fv),
        "total_invested": round(total_invested),
        "gain": round(gain),
        "gain_percent": round(gain_pct, 1)
    }

# Contoh: 3jt/bulan, 8% return, 20 tahun
result = compound_dca(3_000_000, 0.08, 20)
print(f"Nilai akhir: Rp {result['future_value']:,.0f}")
print(f"Total setor: Rp {result['total_invested']:,.0f}")
print(f"Keuntungan:  Rp {result['gain']:,.0f} (+{result['gain_percent']}%)")
```

Pakai `execute_code` untuk run kalkulator ini langsung kalau user minta hitungan spesifik.

---

## TOPIK: Perbandingan Produk Keuangan

### Tabel perbandingan (baseline)

| Produk | Return tipikal | Risiko | Likuiditas | Min. investasi | Syariah? |
|--------|---------------|--------|------------|----------------|----------|
| Deposito bank | 3-5%/tahun | Sangat rendah (dijamin LPS s.d. 2M) | Rendah (terikat tenor) | Rp 1-10 juta | Ada versi syariah |
| Reksadana Pasar Uang | 4-6%/tahun | Rendah | Tinggi (H+1 s.d. H+3) | Rp 10.000 - 100.000 | Ada versi syariah |
| Reksadana Pendapatan Tetap | 5-8%/tahun | Sedang | Sedang (H+3 s.d. H+7) | Rp 100.000 | Ada versi syariah |
| Reksadana Saham | 8-15%/tahun (volatile) | Tinggi | Sedang (H+3 s.d. H+7) | Rp 100.000 | Ada versi syariah |
| Sukuk Negara Ritel (SR/ST) | 5-7%/tahun (kupon) | Rendah (dijamin negara) | Rendah-Sedang (pasar sekunder) | Rp 1.000.000 | ✅ Syariah by definition |
| SBN (ORI/SR/ST) | 5-7%/tahun | Rendah | Tergantung seri | Rp 1.000.000 | SR/ST = syariah |
| Saham (langsung) | Variable (-50% s.d. +100%+) | Tinggi | Tinggi (T+2) | 1 lot (100 lembar × harga) | Filter via DES/JII |
| Emas (fisik/digital) | 5-15%/tahun (historis, volatile) | Sedang | Sedang-Tinggi | Rp 10.000 (digital) | ✅ |
| Properti | 5-15%/tahun (capital gain + sewa) | Sedang-Tinggi | Sangat rendah | Puluhan-ratusan juta | Tergantung akad |

**CAVEAT BESAR**: angka return di tabel ini adalah **rentang historis kasar**. BUKAN jaminan. BUKAN angka presisi. Selalu cek data terkini. Gw flag ini karena angka-angka di atas bisa berubah signifikan tergantung kondisi makroekonomi.

### Cara pilih (framework, BUKAN advice)

```
Timeframe < 1 tahun:  deposito / reksadana pasar uang / tabungan
Timeframe 1-5 tahun:  reksadana pendapatan tetap / sukuk negara / campuran
Timeframe > 5 tahun:  reksadana saham / saham langsung / properti / emas

Risk tolerance rendah: deposito, sukuk negara, RDPU
Risk tolerance sedang: reksadana campuran, emas, RDPT
Risk tolerance tinggi: saham, reksadana saham, crypto (high risk)
```

---

## TOPIK: Dana Pensiun

### Hitung kebutuhan (simplified)

```python
def pension_need(monthly_expense, retirement_age, life_expectancy, inflation_rate, current_age):
    """Hitung dana pensiun yang dibutuhkan (simplified, BUKAN exact)."""
    years_to_retire = retirement_age - current_age
    years_in_retirement = life_expectancy - retirement_age
    
    # Adjust expense for inflation at retirement
    future_monthly = monthly_expense * ((1 + inflation_rate) ** years_to_retire)
    
    # Total needed (simplified: lump sum at retirement covering N years)
    # Asumsi: return = inflasi saat pensiun (conservation assumption)
    total_needed = future_monthly * 12 * years_in_retirement
    
    return {
        "monthly_expense_at_retirement": round(future_monthly),
        "total_needed": round(total_needed),
        "years_to_retire": years_to_retire,
        "years_in_retirement": years_in_retirement
    }

# Contoh: expense 10jt/bulan, pensiun 55, life exp 75, inflasi 5%, umur sekarang 30
result = pension_need(10_000_000, 55, 75, 0.05, 30)
```

**Disclaimer**: ini simplified calculation. Real pension planning account for:
- Variabel return selama pensiun
- Healthcare cost escalation (biasanya > inflasi umum)
- Sumber income lain (BPJS TK, pensiun perusahaan, passive income)
- Tax

Untuk perencanaan serius → konsultasi CFP.

---

## TOPIK: Rasio Keuangan Sehat

| Rasio | Formula | Target "sehat" | Catatan |
|-------|---------|----------------|---------|
| Savings rate | (Tabungan+Investasi) / Income | ≥ 20% | Higher = better |
| Debt-to-Income (DTI) | Total cicilan/bulan / Income | ≤ 30% | >50% = danger zone |
| Emergency fund ratio | Liquid assets / Monthly expense | ≥ 3 bulan | Target 6-12 bulan |
| Net worth growth | (NW tahun ini - NW tahun lalu) / NW tahun lalu | > inflasi | Minimal positif |
| Housing cost ratio | Cicilan rumah / Income | ≤ 30% | Versi konservatif: ≤ 25% |
| Investment ratio | Investasi / Income | ≥ 10% | Subset dari savings rate |

---

## TOPIK: Inflasi & Purchasing Power

### Konsep kunci untuk user

```
Rp 1.000.000 hari ini ≠ Rp 1.000.000 dalam 10 tahun.

Dengan inflasi 5%/tahun:
- Rp 1.000.000 hari ini = Rp 613.913 dalam purchasing power 10 tahun dari sekarang
- Artinya: lo butuh Rp 1.628.895 dalam 10 tahun untuk beli barang yang sama

Formula: Real return = Nominal return - Inflasi (simplified)
- Deposito 5% - Inflasi 5% = 0% real return
- Reksadana saham 12% - Inflasi 5% = 7% real return (approximate)
```

**Data inflasi Indonesia**: cek BPS (https://www.bps.go.id/) untuk angka terkini. Rata-rata historis Indonesia ~3-6%/tahun (range, bukan fixed).

---

## TOPIK: Produk Syariah Spesifik

### Akad-akad umum di produk keuangan syariah

| Akad | Artinya (simplified) | Produk |
|------|---------------------|--------|
| Mudharabah | Bagi hasil (investor kasih modal, pengelola kerja) | Deposito syariah, reksadana |
| Musyarakah | Partnership (dua pihak sama-sama modal + kerja) | Pembiayaan usaha |
| Murabahah | Jual beli dengan margin transparan | KPR syariah, cicilan |
| Ijarah | Sewa | Sukuk ijarah, leasing syariah |
| Wakalah | Perwakilan (fee-based) | Asuransi syariah |
| Sukuk | Sertifikat kepemilikan underlying asset | SBN syariah (SR, ST, PBS) |

### Bedanya dengan konvensional (simplified)

| Aspek | Konvensional | Syariah |
|-------|-------------|---------|
| Basis | Bunga (interest) | Bagi hasil / margin / sewa |
| Return | Fixed/floating interest rate | Nisbah bagi hasil / kupon |
| Underlying | Bisa tanpa underlying asset | Harus ada underlying asset/aktivitas |
| Pengawasan | OJK | OJK + Dewan Pengawas Syariah (DPS) |
| Instrumen haram | Tidak ada filter | Filter: no riba, gharar, maysir, barang haram |

---

## Output Style

Setiap output HARUS contain:

1. **Jawaban langsung** terhadap pertanyaan user
2. **Angka / hitungan** kalau relevan (pakai `execute_code` untuk precision)
3. **Asumsi yang dipakai** (eksplisit, bukan hidden)
4. **Limitasi / caveat** (apa yang bisa salah, apa yang belum di-account)
5. **Next step** ("kalau mau lebih detail, cek X" atau "untuk keputusan besar, konsultasi CFP")
6. **Sumber** kalau ada data yang diambil dari web

## Pitfalls

### Pitfall 1: Halu angka return

JANGAN bilang "return saham rata-rata 12% per tahun" tanpa caveat bahwa:
- Itu historis, bukan jaminan
- Depends on timeframe yang dipilih
- Belum dikurangi fee dan inflasi
- Ada tahun yang -40%

### Pitfall 2: One-size-fits-all advice

"Investasi 20% income" bagus sebagai guideline tapi:
- Gak realistis buat income UMR dengan tanggungan
- Terlalu rendah buat income tinggi tanpa tanggungan

Selalu qualify dengan "ini baseline, adjust ke situasi lo".

### Pitfall 3: Bikin user overconfident

Kalkulator compound interest bikin angka besar yang exciting. Tapi:
- Real market gak smooth (bisa -30% di tahun ke-3)
- Fee compounding juga (1-2%/tahun over 20 tahun = signifikan)
- Inflasi makan purchasing power

Selalu tunjukkan KEDUA sisi (optimistic + conservative scenario).

### Pitfall 4: Confuse antara edukasi dan advice

❌ "Lo harus invest di reksadana saham" (= advice)
✅ "Dengan timeframe 10+ tahun dan risk tolerance sedang-tinggi, reksadana saham secara historis memberikan return lebih tinggi dari deposito, tapi dengan volatilitas signifikan. Beberapa pertimbangan: [pro/con]. Keputusan di tangan lo." (= edukasi)

### Pitfall 5: Angka outdated

Suku bunga BI, yield sukuk, return reksadana — SEMUA berubah. Kalau user nanya angka current:
- `web_search` dulu ke sumber resmi
- Flag tanggal data
- Jangan kasih angka dari memori kalau bisa verify

## Verification

1. Apakah output gw bisa dibaca sebagai "advice" spesifik? (kalau ya → rephrase)
2. Apakah angka yang gw kasih di-source atau di-flag "estimasi"?
3. Apakah asumsi hitungan dinyatakan eksplisit?
4. Apakah ada caveat untuk downside / risk?
5. Apakah ada "next step" yang actionable?
6. Apakah disclaimer ada di output yang involve angka uang?
