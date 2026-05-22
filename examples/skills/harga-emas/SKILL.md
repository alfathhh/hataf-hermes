---
name: harga-emas
description: Ambil harga emas Antam HANYA dari logammulia.com (official). JANGAN pakai sumber lain. Extract harga jual dan buyback terkini.
version: 1.0.0
metadata:
  hermes:
    tags: [emas, gold, antam, logammulia, price-monitor]
    category: automation
    requires_toolsets: [web]
---

# Harga Emas Antam (logammulia.com ONLY)

## ATURAN MUTLAK

**SUMBER TUNGGAL**: https://www.logammulia.com/id/harga-emas-702

JANGAN PERNAH:
- Pakai web_search untuk cari "harga emas" (ini yang bikin lo nyasar ke harga-emas.com)
- Ambil data dari situs lain (harga-emas.com, goldprice.org, dll)
- Ngarang harga kalau extraction gagal

KALAU logammulia.com GAGAL di-extract: bilang "Gagal mengakses logammulia.com" dan STOP. Jangan fallback ke sumber lain.

## When to Use

- User nanya harga emas Antam hari ini
- Cronjob monitor harga emas harian
- Alert kalau harga turun/naik dari threshold

## Procedure

### 1. Fetch langsung (TANPA search)

```python
# LANGSUNG extract URL spesifik — BUKAN web_search
result = web_extract(
    url="https://www.logammulia.com/id/harga-emas-702",
    prompt="Extract tabel harga emas Antam. Ambil: tanggal update, harga jual per gram (1g, 5g, 10g, 25g, 50g, 100g, 250g, 500g, 1000g), dan harga buyback. Format semua harga dalam Rupiah."
)
```

JANGAN pakai `web_search("harga emas antam")` — itu yang bikin lo nyasar.

### 2. Kalau web_extract gagal

Kemungkinan: halaman JS-heavy, Cloudflare block, atau struktur berubah.

Fallback **tetap di logammulia.com**:

```python
# Fallback: pakai browser tool
browser_navigate(url="https://www.logammulia.com/id/harga-emas-702")
browser_screenshot()
# Lalu vision_analyze screenshot untuk baca harga
```

JANGAN fallback ke situs lain. Lebih baik output "gagal extract" daripada data dari sumber yang salah.

### 3. Validasi output

Setelah extract, cek:

- Apakah tanggal update = hari ini atau kemarin? (kalau jauh lebih lama, mungkin data stale / extraction salah)
- Apakah harga dalam range masuk akal? (per Mei 2026, harga emas Antam ~Rp 1-2 juta/gram — angka ini PASTI berubah, tapi kalau result < Rp 100.000 atau > Rp 10.000.000 per gram, jelas salah extraction)
- Apakah ada "Rp" prefix di angka? Strip dan convert ke integer untuk perbandingan

### 4. Format output

```markdown
## Harga Emas Antam — [TANGGAL]

**Sumber**: https://www.logammulia.com/id/harga-emas-702
**Diakses**: [timestamp sekarang]

### Harga Jual (per batang)
| Gram | Harga |
|------|-------|
| 1g   | Rp X.XXX.XXX |
| 5g   | Rp X.XXX.XXX |
| 10g  | Rp X.XXX.XXX |
| 25g  | Rp X.XXX.XXX |
| 50g  | Rp X.XXX.XXX |
| 100g | Rp X.XXX.XXX |

### Harga Buyback
Rp X.XXX.XXX /gram

### Perubahan vs kemarin
[Kalau ada data kemarin di state file, hitung delta. Kalau gak ada, skip.]
```

## Untuk Cronjob

Contoh prompt cron yang BENAR:

```
Ambil harga emas Antam hari ini HANYA dari https://www.logammulia.com/id/harga-emas-702 menggunakan web_extract langsung ke URL tersebut, JANGAN web_search. Extract harga jual 1g dan harga buyback. Bandingkan dengan file ~/.hermes/cron/output/harga-emas/last.json. Kalau harga berubah >1% dari kemarin, kirim ke Telegram dengan format: "🪙 Emas Antam [tanggal]: Jual 1g Rp X | Buyback Rp Y | [naik/turun X%]". Update last.json. Kalau sama, silent (gak kirim apa-apa). Kalau gagal extract, kirim: "⚠️ Gagal ambil harga emas dari logammulia.com".
```

Kunci yang bikin ini jalan:
1. URL eksplisit (bukan query search)
2. Tool eksplisit (`web_extract`, bukan `web_search`)
3. Instruksi "JANGAN" yang tegas
4. Fallback behavior jelas (gagal = bilang gagal, bukan cari sumber lain)
5. Silent success (gak spam Telegram kalau gak ada perubahan)

## Pitfalls

### Pitfall 1: Agent tetap web_search walaupun disuruh jangan

Kalau ini terjadi, kemungkinan:
- Prompt cron terlalu ambigu → perjelas "LANGSUNG web_extract ke URL ini"
- Model cheap (yang dipake cron) gak cukup pinter follow instruction → ganti model cron ke yang lebih kuat

Fix: di config.yaml, pastikan cron gak didowngrade ke model terlalu lemah. Atau, tambah di awal prompt: "CRITICAL: Do NOT use web_search. Use web_extract with this exact URL."

### Pitfall 2: Struktur HTML logammulia.com berubah

Situs jualan sering redesign. Kalau extraction mulai gagal (return kosong / harga aneh), lo perlu:
1. Manual cek halaman di browser
2. Update prompt `web_extract` dengan schema baru
3. Atau switch ke browser+screenshot+vision approach

### Pitfall 3: Cloudflare block

logammulia.com mungkin pake anti-bot. Kalau `web_extract` selalu gagal:
- Coba `browser_navigate` (simulate real browser)
- Atau kalau pake self-hosted Firecrawl, Playwright fallback aktif

### Pitfall 4: Harga yang ditampilkan = harga member

logammulia.com kadang nampilkan harga berbeda untuk member vs non-member. Pastikan yang diextract adalah harga **non-member** (harga umum) kecuali user eksplisit bilang dia member.

## State file (untuk tracking perubahan)

Simpan di `~/.hermes/cron/output/harga-emas/last.json`:

```json
{
  "date": "2026-05-22",
  "sell_1g": 1850000,
  "buyback": 1750000,
  "source": "https://www.logammulia.com/id/harga-emas-702"
}
```

Cron job compare harga baru vs file ini untuk detect perubahan.

## Verification

1. Apakah sumber yang dikutip = logammulia.com? (BUKAN harga-emas.com atau lainnya)
2. Apakah tanggal update masuk akal?
3. Apakah angka harga dalam range wajar?
4. Apakah ada fallback ke sumber lain yang SEHARUSNYA tidak terjadi?
