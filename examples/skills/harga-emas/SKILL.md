---
name: harga-emas
description: Ambil harga emas Antam HANYA dari logammulia.com (official). JANGAN pakai sumber lain.
version: 2.0.0
metadata:
  hermes:
    tags: [emas, gold, antam, logammulia, price-monitor]
    category: automation
    requires_toolsets: [web]
---

# Harga Emas Antam

## ATURAN MUTLAK (NON-NEGOTIABLE)

```
SUMBER: https://www.logammulia.com/id/harga-emas-702
TOOL: web_extract (LANGSUNG ke URL di atas)

DO NOT: web_search("harga emas")
DO NOT: ambil data dari harga-emas.com atau situs lain
DO NOT: ngarang harga kalau extraction gagal
DO NOT: fallback ke sumber lain

IF logammulia.com GAGAL → bilang "Gagal mengakses logammulia.com" dan STOP
```

---

## PROCEDURE (ikuti exact)

### Step 1: Fetch langsung (TANPA search)

```python
web_extract(
    url="https://www.logammulia.com/id/harga-emas-702",
    prompt="Extract tabel harga emas Antam. Ambil: tanggal update, harga jual per gram (1g, 5g, 10g, 25g, 50g, 100g), dan harga buyback. Format semua harga dalam Rupiah."
)
```

### Step 2: Decision tree kalau gagal

```
IF web_extract return data → lanjut Step 3
IF web_extract return kosong / error:
  → coba browser_navigate("https://www.logammulia.com/id/harga-emas-702")
  → browser_screenshot()
  → vision_analyze screenshot
  IF masih gagal → output "⚠️ Gagal ambil harga emas dari logammulia.com" dan STOP
  DO NOT: cari sumber lain
  DO NOT: ngarang angka
```

### Step 3: Validasi output

```
CHECK: tanggal update = hari ini atau kemarin? (kalau jauh = stale)
CHECK: harga per gram dalam range Rp 1.000.000 - Rp 5.000.000? (range wajar 2026)
  IF harga < Rp 100.000 → extraction salah, JANGAN deliver
  IF harga > Rp 10.000.000 → extraction salah, JANGAN deliver
CHECK: ada "Rp" prefix? Strip untuk angka comparison
```

### Step 4: Format output

---

## OUTPUT TEMPLATE

```markdown
## 🪙 Harga Emas Antam — [TANGGAL dari halaman]

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
[IF ada data kemarin di state file → hitung delta]
[IF gak ada → "Data kemarin tidak tersedia"]
```

---

## CONTOH OUTPUT

```markdown
## 🪙 Harga Emas Antam — 24 Mei 2026

**Sumber**: https://www.logammulia.com/id/harga-emas-702
**Diakses**: 24 Mei 2026, 10:30 WIB

### Harga Jual (per batang)
| Gram | Harga |
|------|-------|
| 1g   | Rp 1.875.000 |
| 5g   | Rp 9.125.000 |
| 10g  | Rp 18.100.000 |
| 25g  | Rp 45.000.000 |
| 50g  | Rp 89.750.000 |
| 100g | Rp 179.250.000 |

### Harga Buyback
Rp 1.773.000 /gram

### Perubahan vs kemarin
Jual 1g: +Rp 12.000 (+0.64%)
Buyback: +Rp 10.000 (+0.57%)
```

---

## UNTUK CRONJOB

Prompt cron yang BENAR:

```
CRITICAL: Do NOT use web_search. Use web_extract with this exact URL: https://www.logammulia.com/id/harga-emas-702

Steps:
1. web_extract URL di atas, extract harga jual 1g dan buyback
2. Baca file ~/.hermes/cron/output/harga-emas/last.json
3. IF harga berubah >1% dari kemarin → kirim Telegram: "🪙 Emas Antam [tanggal]: Jual 1g Rp X | Buyback Rp Y | [naik/turun X%]"
4. IF harga sama → DIAM (jangan kirim)
5. IF gagal extract → kirim: "⚠️ Gagal ambil harga emas dari logammulia.com"
6. Update last.json dengan data baru
```

State file (`~/.hermes/cron/output/harga-emas/last.json`):
```json
{
  "date": "2026-05-24",
  "sell_1g": 1875000,
  "buyback": 1773000,
  "source": "https://www.logammulia.com/id/harga-emas-702"
}
```

---

## VERIFICATION

```
□ Sumber yang dikutip = logammulia.com? (BUKAN harga-emas.com)
□ Tanggal update masuk akal?
□ Angka harga dalam range wajar?
□ Gak ada fallback ke sumber lain?

IF ada □ TIDAK → STOP, jangan deliver
```
