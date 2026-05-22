---
name: saham-syariah
description: Analisis dan screening saham syariah Indonesia (IDX). Filter berdasarkan Daftar Efek Syariah (DES) OJK, analisis fundamental (PER/PBV/DY/DER), dan monitoring harga. BUKAN financial advice — data + screening only.
version: 1.0.0
metadata:
  hermes:
    tags: [saham, syariah, idx, investment, screening, fundamental, ojk]
    category: finance
    requires_toolsets: [web]
---

# Analisis Saham Syariah Indonesia

Skill untuk screening dan analisis saham yang **masuk Daftar Efek Syariah (DES) OJK**. Output = data terstruktur + screening result. BUKAN rekomendasi beli/jual.

## DISCLAIMER (WAJIB ditampilkan di setiap output)

```
⚠️ DISCLAIMER: Output ini BUKAN nasihat investasi. Ini adalah hasil screening
otomatis berdasarkan data publik. Selalu lakukan riset mandiri (DYOR) dan
konsultasi dengan penasihat keuangan berlisensi sebelum mengambil keputusan
investasi. Past performance ≠ future results.
```

## When to Use

- User nanya "saham syariah apa yang bagus?"
- User minta screening saham dengan kriteria tertentu
- User minta analisis fundamental saham tertentu
- User minta cek apakah saham X masuk DES
- Cronjob monitoring harga saham syariah
- Alert kalau saham target masuk/keluar DES

JANGAN pakai untuk:
- Rekomendasi beli/jual (skill ini BUKAN advisor)
- Analisis teknikal (candlestick, support/resistance) — itu butuh charting tool terpisah
- Saham luar negeri (skill ini fokus IDX)
- Crypto/forex (bukan saham)

## Sumber Data (HANYA dari sumber resmi/kredibel)

| Data | Sumber primer | URL |
|------|---------------|-----|
| **Daftar Efek Syariah (DES)** | OJK | https://www.ojk.go.id/id/kanal/syariah/data-dan-statistik/daftar-efek-syariah/ |
| **Indeks JII (Jakarta Islamic Index)** | IDX | https://www.idx.co.id/id/data-pasar/indeks-saham/ |
| **Indeks JII70** | IDX | https://www.idx.co.id/id/data-pasar/indeks-saham/ |
| **Indeks ISSI** | IDX | https://www.idx.co.id/id/data-pasar/indeks-saham/ |
| **Data fundamental (PER, PBV, dll)** | RTI Business / Sectors.app / IDX | https://www.rti.co.id/ atau https://sectors.app/ |
| **Harga real-time** | IDX / Yahoo Finance (ID) | https://finance.yahoo.com/ (append .JK untuk IDX) |
| **Statistik pasar syariah** | OJK | https://ojk.go.id/id/kanal/syariah/data-dan-statistik/saham-syariah/ |

**JANGAN** ambil data dari:
- Blog random / forum saham tanpa verifikasi
- Grup Telegram "signal"
- Situs yang claim "guaranteed profit"

## Procedure

### 1. Klarifikasi kebutuhan user

Tanya dulu:
- "Lo mau screening (cari saham baru) atau analisis (saham tertentu)?"
- "Kriteria apa? (dividen tinggi? undervalued? growth? low debt?)"
- "Timeframe investasi? (short term trading vs long term hold?)"
- "Budget range? (blue chip aja atau termasuk second liner?)"

### 2. Validasi status syariah

**LANGKAH PERTAMA SEBELUM ANALISIS APAPUN**: cek apakah saham masuk DES.

```python
# Cek DES terkini dari OJK
result = web_extract(
    url="https://www.ojk.go.id/id/kanal/syariah/data-dan-statistik/daftar-efek-syariah/",
    prompt="Extract daftar terbaru Keputusan DES (Daftar Efek Syariah). Ambil: nomor SK, tanggal berlaku, jumlah efek. Cari link download PDF terbaru."
)
```

Kalau saham TIDAK ada di DES:
```
❌ [KODE] TIDAK masuk Daftar Efek Syariah (DES) periode [terbaru].
Saham ini tidak memenuhi kriteria syariah OJK. Tidak dilanjutkan analisis.
```

STOP di sini. Jangan lanjut analisis kalau gak syariah.

### 3. Kriteria syariah (untuk edukasi user)

Per regulasi OJK, saham masuk DES kalau memenuhi:

**Kriteria kualitatif:**
- Emiten tidak melakukan kegiatan usaha yang bertentangan dengan prinsip syariah:
  - ❌ Perjudian
  - ❌ Perdagangan yang dilarang (gharar, maysir)
  - ❌ Jasa keuangan ribawi (bank konvensional, asuransi konvensional)
  - ❌ Produksi/distribusi barang haram (alkohol, babi, rokok — note: rokok kontroversial, cek DES aktual)
  - ❌ Hiburan yang bertentangan dengan syariah

**Kriteria kuantitatif:**
- Total utang berbasis bunga / total aset ≤ 45%
- Pendapatan non-halal / total pendapatan ≤ 10%

> Catatan: kriteria di atas berdasarkan regulasi OJK yang gw ketahui. **Cek SK DES terbaru** untuk konfirmasi — bisa berubah.

### 4. Ambil data fundamental

Untuk saham yang SUDAH dikonfirmasi syariah:

```python
# Ambil dari RTI Business atau Yahoo Finance
result = web_extract(
    url="https://finance.yahoo.com/quote/BBRI.JK/",
    prompt="Extract: harga terakhir, market cap, PER (trailing), PBV, dividend yield, 52-week high/low, volume rata-rata"
)
```

Atau dari Sectors.app (lebih lengkap untuk IDX):

```python
result = web_extract(
    url="https://sectors.app/id/stocks/BBRI",
    prompt="Extract data fundamental: PER, PBV, ROE, ROA, DER, dividend yield, EPS growth, revenue growth"
)
```

### 5. Framework screening

#### Screening: Value (undervalued)
| Metrik | Kriteria "murah" | Catatan |
|--------|-----------------|---------|
| PER | < 15x | Bandingkan vs rata-rata sektoral |
| PBV | < 1.5x | Di bawah 1x = sangat murah ATAU ada masalah |
| DY (Dividend Yield) | > 4% | Konsisten minimal 3 tahun |
| PEG Ratio | < 1 | PER / EPS growth rate |

#### Screening: Quality (fundamental kuat)
| Metrik | Kriteria "bagus" | Catatan |
|--------|-----------------|---------|
| ROE | > 15% | Konsisten 3-5 tahun |
| DER (Debt to Equity) | < 1.0 | Untuk syariah, utang ribawi < 45% aset |
| Revenue growth | > 10% YoY | Minimal 2-3 tahun berturut |
| EPS growth | > 10% YoY | Konsisten |
| Free Cash Flow | Positif | Minimal 3 tahun terakhir |

#### Screening: Syariah-specific
| Metrik | Batas syariah |
|--------|--------------|
| Interest-bearing debt / Total assets | ≤ 45% |
| Non-halal income / Total revenue | ≤ 10% |

### 6. Format output analisis

```markdown
## Analisis Saham Syariah: [KODE] — [Nama Perusahaan]

⚠️ DISCLAIMER: Bukan nasihat investasi. Data screening otomatis. DYOR.

### Status Syariah
✅ Masuk DES OJK Periode [X] (SK No. [Y], berlaku [tanggal])
✅ Masuk indeks: [JII / JII70 / ISSI] — sebutkan mana yang applicable

### Data Harga
| Metrik | Nilai |
|--------|-------|
| Harga terakhir | Rp X.XXX |
| 52-week high | Rp X.XXX |
| 52-week low | Rp X.XXX |
| Market cap | Rp X T |
| Volume rata-rata | X juta lembar |

### Fundamental
| Metrik | Nilai | vs Sektor | Penilaian |
|--------|-------|-----------|-----------|
| PER | Xx | rata-rata sektor Yx | [Murah/Wajar/Mahal] |
| PBV | Xx | rata-rata sektor Yx | [Murah/Wajar/Mahal] |
| ROE | X% | — | [Bagus/Cukup/Kurang] |
| DER | Xx | — | [Aman/Perlu perhatian] |
| DY | X% | — | [Menarik/Biasa] |
| EPS Growth (YoY) | X% | — | — |

### Screening Result
- Value score: [★★★☆☆] — [alasan singkat]
- Quality score: [★★★★☆] — [alasan singkat]
- Syariah compliance: ✅ / ⚠️ mendekati batas

### Risiko yang teridentifikasi
1. [Risiko 1 — misal: DER mendekati batas 45% syariah]
2. [Risiko 2 — misal: revenue turun 2 kuartal terakhir]
3. [Risiko 3 — misal: sektor cyclical, sensitif suku bunga]

### Catatan
- Data diambil [tanggal], bisa sudah berubah
- DES di-review OJK tiap Mei dan November — status syariah bisa berubah
- [Hal lain yang perlu user cek manual]

### Sumber
- [URL 1] (diakses [tanggal])
- [URL 2] (diakses [tanggal])
```

### 7. Screening batch (cari saham baru)

Kalau user minta "cariin saham syariah yang bagus":

```python
# Step 1: Ambil konstituent JII (30 saham paling likuid syariah)
result = web_extract(
    url="https://www.idx.co.id/id/data-pasar/indeks-saham/",
    prompt="Extract daftar konstituent Jakarta Islamic Index (JII) terbaru: kode saham dan nama perusahaan"
)

# Step 2: Untuk top candidates, ambil fundamental satu-satu
# (pakai delegate_task kalau mau paralel)
```

Output screening batch:

```markdown
## Screening Saham Syariah — [Tanggal]

**Kriteria**: [yang user minta, misal: DY > 4%, PER < 15, ROE > 15%]
**Universe**: JII (30 saham) / JII70 (70 saham) / ISSI (semua syariah)
**Lolos filter**: [N] saham

| # | Kode | Nama | PER | PBV | ROE | DY | DER | Score |
|---|------|------|-----|-----|-----|-----|-----|-------|
| 1 | XXXX | ... | ... | ... | ... | ... | ... | ★★★★★ |
| 2 | YYYY | ... | ... | ... | ... | ... | ... | ★★★★☆ |
| ... |

⚠️ DISCLAIMER: Bukan rekomendasi. Screening otomatis berdasarkan kriteria numerik.
Selalu riset mandiri sebelum investasi.
```

## Untuk Cronjob

### Monitor harga saham watchlist

```
/cron add "0 16 * * 1-5" "Cek harga penutupan hari ini untuk saham: BBRI, TLKM, UNVR, ANTM (semua .JK di Yahoo Finance). Bandingkan dengan ~/.hermes/cron/output/saham-syariah/watchlist.json. Kalau ada yang naik/turun >3% hari ini, kirim alert ke Telegram dengan format: '📊 [KODE] [naik/turun] [X%] → Rp [harga]. Volume: [X]M.' Update watchlist.json." --skill saham-syariah --name "Saham Syariah Daily"
```

### Alert DES update (2x setahun)

```
/cron add "0 9 1 5,11 *" "Cek halaman OJK Daftar Efek Syariah (https://www.ojk.go.id/id/kanal/syariah/data-dan-statistik/daftar-efek-syariah/) apakah ada SK DES baru. Kalau ada update, kirim ke Telegram: 'DES baru terbit: [No SK], berlaku [tanggal]. Cek apakah saham watchlist masih masuk.'" --skill saham-syariah --name "DES Update Check"
```

### Weekly screening

```
/cron add "0 20 * * 5" "Jalankan screening saham JII dengan kriteria: PER < 15, PBV < 2, ROE > 12%, DY > 3%. Kirim top 5 result ke Telegram." --skill saham-syariah --name "Weekly Syariah Screen"
```

## Pitfalls

### Pitfall 1: Saham "syariah" tapi mendekati batas

Beberapa saham masuk DES tapi DER-nya 40-44% (batas 45%). Ini berisiko keluar di review DES berikutnya. **Flag ini eksplisit** di output.

### Pitfall 2: DES itu lagging

DES di-review 2x setahun (Mei dan November). Di antara review, emiten bisa berubah fundamental (ambil utang baru, mulai bisnis non-halal) tapi masih "secara resmi" di DES sampai review berikutnya. **Cek laporan keuangan terbaru** kalau ragu.

### Pitfall 3: Halu angka fundamental

LLM SANGAT sering halu angka keuangan. Mitigasi:
- SELALU ambil dari tool (`web_extract` / `web_search`), BUKAN dari memori
- Cross-check: kalau PER Yahoo beda jauh dari RTI → flag discrepancy
- Kalau data gak ketemu → bilang "data tidak tersedia", JANGAN perkirakan

### Pitfall 4: Confuse JII vs ISSI vs DES

| Istilah | Artinya |
|---------|---------|
| **DES** | Daftar lengkap SEMUA efek yang qualify syariah (ratusan saham) |
| **ISSI** | Indeks semua saham DES yang listing di IDX |
| **JII70** | 70 saham syariah paling likuid |
| **JII** | 30 saham syariah paling likuid (subset JII70) |

Hierarki: DES ⊃ ISSI ⊃ JII70 ⊃ JII

### Pitfall 5: Survivorship bias

Jangan kasih impresi "saham syariah selalu naik". Tunjukkan risiko:
- Saham bisa turun 50%+ walaupun syariah
- Syariah = filter moral/kepatuhan, BUKAN jaminan profit
- Past dividend ≠ future dividend

### Pitfall 6: Data stale dari Yahoo Finance

Yahoo Finance kadang delay 15-20 menit untuk IDX. Untuk intraday, sebutkan bahwa data mungkin bukan real-time. Untuk analisis end-of-day, data closing biasanya akurat.

### Pitfall 7: User minta "pasti untung"

Kalau user push untuk prediksi profit:
```
Saya tidak bisa memprediksi pergerakan harga saham. Yang bisa saya berikan:
data historis, screening berdasarkan kriteria, dan identifikasi risiko.
Keputusan investasi sepenuhnya di tangan Anda.
```

JANGAN kasih target harga, jangan bilang "saham ini bagus buat dibeli".

## Verification

Sebelum kirim output:

1. Apakah SETIAP saham yang dibahas sudah DIKONFIRMASI masuk DES? (cek, jangan asumsi)
2. Apakah angka fundamental dari tool result (bukan dari memori)?
3. Apakah disclaimer ada di output?
4. Apakah ada statement yang bisa dibaca sebagai "rekomendasi beli"? (kalau ya, rephrase)
5. Apakah sumber dikutip dengan URL + tanggal akses?
6. Apakah risiko disebutkan (bukan cuma yang positif)?

## Indeks referensi cepat

| Indeks | Jumlah saham | Review | Cocok untuk |
|--------|-------------|--------|-------------|
| JII | 30 | 2x/tahun (Mei, Nov) | Blue chip syariah, paling likuid |
| JII70 | 70 | 2x/tahun | Mid-large cap syariah |
| ISSI | 400+ | mengikuti DES | Universe lengkap syariah |
| IDX-MES BUMN 17 | 17 | berkala | BUMN syariah |

## MCP alternatif (advanced)

Kalau lo mau data yang lebih real-time dan structured, ada MCP server untuk saham Indonesia:

- **baguskto-saham** (MCP server) — Node.js based, akses data saham IDX
  - Source: https://lobehub.com/mcp/baguskto-saham
  - Setup di `config.yaml` under `mcp:` section

- **datasaham.io** (API) — 50+ endpoint, termasuk bandarmology
  - https://datasaham.io/
  - Butuh API key terpisah

- **Sectors.app** — screener IDX dengan filter lengkap
  - https://sectors.app/
  - Ada free tier

Ini opsional — skill ini bisa jalan cukup dengan `web_extract` ke Yahoo Finance / RTI.

## Yang gw belum yakin

- **Apakah Yahoo Finance .JK selalu up-to-date untuk semua saham IDX?** Beberapa saham kecil (third liner) kadang datanya sparse di Yahoo. Untuk saham JII/JII70, biasanya OK.
- **Format halaman OJK DES**: OJK sering redesign website. Kalau `web_extract` ke halaman DES gagal, coba search "Keputusan Daftar Efek Syariah [tahun] OJK" → biasanya ada PDF yang bisa di-extract.
- **Sectors.app free tier limits**: gw belum cek berapa query/hari free tier-nya. Kalau hit limit, fallback ke Yahoo Finance.
