---
name: web-scrape
description: Scrape halaman web untuk extract data terstruktur (JSON). Schema-first, anti-bot fallback ke browser, kalau gagal bilang gagal.
version: 2.0.0
metadata:
  hermes:
    tags: [web, scrape, extract, monitoring]
    category: automation
    requires_toolsets: [web]
---

# Web Scrape

## KAPAN PAKAI

```
IF user minta "scrape" OR "extract data" OR "ambil info dari web" → PAKAI
IF user minta "pantau perubahan halaman" → PAKAI
IF user minta "ambil tabel dari URL" → PAKAI
IF halaman butuh login → JANGAN PAKAI (bilang butuh login)
IF mass scraping ribuan URL → JANGAN PAKAI (butuh worker terpisah)
```

---

## PROCEDURE (ikuti exact, jangan skip)

### Step 1: Klarifikasi schema

TANYA user (kalau belum jelas):
- "Field apa yang lo butuhkan?"
- "Output format apa? (JSON / CSV / tabel)"
- "Berapa item? (page 1 doang? sampe N halaman?)"

Bikin JSON schema DULU:
```json
{
  "title": "string",
  "price": "number",
  "url": "string (absolute URL)"
}
```

### Step 2: Cek robots.txt

```bash
curl -s https://example.com/robots.txt | head -50
```

```
IF target URL di-disallow → bilang ke user, tanya mau lanjut atau tidak
IF robots.txt OK → lanjut
```

### Step 3: Scrape (coba web_extract dulu)

```python
result = web_extract(
    url="[TARGET_URL]",
    prompt="Extract [FIELD LIST]. Format JSON array. URL field harus absolute."
)
```

### Step 4: Decision tree — kalau gagal

```
IF web_extract return data lengkap → lanjut Step 5
IF web_extract return kosong / partial:
  → coba browser_navigate(url="[TARGET_URL]")
  → browser_screenshot()
  → extract dari rendered HTML
IF browser juga gagal:
  → DO NOT fabricate data
  → output: "❌ Scraping gagal" + diagnosa + rekomendasi
```

### Step 5: Validasi hasil

```
CHECK: semua field terisi? (bukan banyak null)
CHECK: tipe data konsisten? (price semua number, bukan campur string)
CHECK: URL absolute? (bukan relative /products/123)
CHECK: jumlah item masuk akal? (halaman ada 24 produk tapi result 3 = fishy)

IF ada anomali → flag di output ATAU redo extract
```

### Step 6: Format output

---

## OUTPUT TEMPLATE: Success

```markdown
## 🌐 Scraping result: [URL]

**Tanggal akses**: [hari ini]
**Items extracted**: [N]
**Status**: ✅ Success

```json
[
  {"title": "Product A", "price": 150000, "url": "https://..."},
  {"title": "Product B", "price": 250000, "url": "https://..."}
]
```

### Catatan
- [field yang gagal: ...]
- [pagination: ada N halaman lagi, mau lanjut?]
```

## OUTPUT TEMPLATE: Gagal

```markdown
## ❌ Scraping gagal: [URL]

**Diagnosa**:
- Web extract: [error / empty response]
- Browser fallback: [juga gagal / Cloudflare blocking]

**Kemungkinan penyebab**:
1. Halaman butuh login
2. Cloudflare WAF blocking
3. HTML structure berubah

**Rekomendasi**:
- Cek manual di browser
- Coba API resmi situs (kalau ada)
- Self-hosted Firecrawl dengan Playwright
```

---

## CONTOH OUTPUT LENGKAP

```markdown
## 🌐 Scraping result: https://tokopedia.com/search?q=keyboard

**Tanggal akses**: 24 Mei 2026
**Items extracted**: 5
**Status**: ✅ Success

```json
[
  {
    "name": "Keychron K2 V2",
    "price": 1250000,
    "rating": 4.9,
    "sold": "500+ terjual",
    "seller": "Keychron Official",
    "url": "https://www.tokopedia.com/keychron/keychron-k2-v2"
  },
  {
    "name": "Rexus Daxa M71",
    "price": 450000,
    "rating": 4.8,
    "sold": "1rb+ terjual",
    "seller": "Rexus Official",
    "url": "https://www.tokopedia.com/rexus/daxa-m71"
  }
]
```

### Catatan
- Harga belum include ongkir
- Data bisa berubah, cek langsung di marketplace untuk harga terkini
- Pagination: ada 10+ halaman, yang di-extract cuma halaman 1
```

---

## DECISION TREE: Harga Indonesia

```
IF format harga = "Rp 1.250.000":
  1. Strip "Rp" dan spasi
  2. Strip titik (pemisah ribuan, BUKAN desimal)
  3. Convert ke integer: 1250000
  DO NOT: interpret titik sebagai desimal
```

## DECISION TREE: Pagination

```
IF user minta "semua halaman":
  → tanya "berapa halaman max?"
  → loop dengan delay 2 detik antar request
  → STOP kalau dapat 429/403

IF user gak sebut pagination:
  → ambil halaman 1 saja
  → bilang "ada N halaman lagi, mau lanjut?"
```

---

## VERIFICATION

```
□ JSON valid (parseable)?
□ Struktur sesuai schema user?
□ Gak ada item suspect (price=0, title="undefined")?
□ URL absolute?
□ Gak ada data yang gw fabricate?

IF ada □ TIDAK → redo atau flag
```
