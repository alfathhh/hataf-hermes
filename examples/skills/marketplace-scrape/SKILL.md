---
name: marketplace-scrape
description: Scrape data produk dari marketplace Indonesia (Tokopedia, Shopee, Bukalapak, Lazada). Extract harga, rating, seller. JSON output.
version: 2.0.0
metadata:
  hermes:
    tags: [scrape, marketplace, tokopedia, shopee, ecommerce, price-monitor]
    category: automation
    requires_toolsets: [web, terminal]
---

# Marketplace Scrape

## KAPAN PAKAI

```
IF user minta "scrape harga di Tokopedia/Shopee/Bukalapak/Lazada" → PAKAI
IF user minta "bandingkan harga marketplace" → PAKAI
IF user minta "monitor harga seller" → PAKAI
IF user minta scrape non-marketplace → JANGAN (pakai web-scrape)
```

---

## PROCEDURE (ikuti exact)

### Step 1: Klarifikasi

```
TANYA:
1. "Marketplace mana?" (Tokped / Shopee / Bukalapak / Lazada / semua?)
2. "Keyword search apa?"
3. "Filter: range harga? rating minimal? lokasi?"
4. "Berapa item?" (top 10 / halaman 1 / pagination?)
5. "Output format?" (JSON / tabel / CSV)
```

### Step 2: Pilih platform + approach

```
IF platform = Tokopedia:
  1. COBA web_extract dulu (gratis, cepat)
  2. IF gagal → browser_navigate + browser_screenshot
  URL: https://www.tokopedia.com/search?q={query}&ob=5

IF platform = Shopee:
  ⚠️ Shopee = PALING SUSAH (full JS rendering + anti-bot agresif)
  1. LANGSUNG browser_navigate (web_extract hampir pasti gagal)
  2. browser_screenshot + vision_analyze
  URL: https://shopee.co.id/search?keyword={query}&sortBy=sales

IF platform = Bukalapak:
  1. COBA web_extract (relatif lebih gampang)
  URL: https://www.bukalapak.com/products?search%5Bkeywords%5D={query}

IF platform = Lazada:
  1. COBA browser_navigate (JS rendering + Cloudflare)
  URL: https://www.lazada.co.id/catalog/?q={query}&sort=salescount
```

### Step 3: Extract data

```python
web_extract(
    url="[URL dari Step 2]",
    prompt="Extract semua produk: nama, harga (Rupiah integer), rating (0-5), jumlah terjual, seller name, URL produk. Format JSON array."
)
```

### Step 4: Parse harga Indonesia

```python
# "Rp 1.250.000" → 1250000
price_str = "Rp 1.250.000"
price_int = int(price_str.replace("Rp", "").replace(".", "").replace(" ", ""))
```

### Step 5: Validate + output

```
CHECK: harga dalam format Rupiah yang benar?
CHECK: URL produk valid (bisa diklik)?
CHECK: gak ada data fabricated?
CHECK: jumlah item masuk akal?

IF data kosong / gagal:
  → bilang "scraping gagal" + kenapa + rekomendasi
  DO NOT: fabricate result
```

---

## OUTPUT TEMPLATE

```markdown
## 🛒 Hasil Scrape: [Marketplace] — "[keyword]"

**Tanggal**: [hari ini]
**Filter**: [yang dipakai]
**Total item**: [N]

| # | Produk | Harga | Rating | Terjual | Seller | Link |
|---|--------|-------|--------|---------|--------|------|
| 1 | [nama] | Rp X | ⭐ 4.9 | 500+ | [seller] | [link] |
| 2 | [nama] | Rp X | ⭐ 4.8 | 1rb+ | [seller] | [link] |

⚠️ Data dari scraping, bisa berubah. Cek langsung di marketplace untuk harga terkini.
Harga belum include ongkir.
```

---

## CONTOH OUTPUT

```markdown
## 🛒 Hasil Scrape: Tokopedia — "mechanical keyboard"

**Tanggal**: 24 Mei 2026
**Filter**: sort by review, semua harga
**Total item**: 5

| # | Produk | Harga | Rating | Terjual | Seller |
|---|--------|-------|--------|---------|--------|
| 1 | Keychron K2 V2 Wireless | Rp 1.250.000 | ⭐ 4.9 | 500+ | Keychron Official |
| 2 | Rexus Daxa M71 Pro | Rp 455.000 | ⭐ 4.8 | 1rb+ | Rexus Official |
| 3 | Royal Kludge RK84 | Rp 520.000 | ⭐ 4.7 | 800+ | RK Store |
| 4 | Fantech Maxfit67 | Rp 650.000 | ⭐ 4.8 | 400+ | Fantech Indonesia |
| 5 | Leopold FC660M | Rp 1.850.000 | ⭐ 4.9 | 200+ | MechaKeys |

⚠️ Data dari scraping, bisa berubah. Harga belum include ongkir.
```

---

## DECISION TREE: Gagal Scrape

```
IF web_extract return kosong:
  → coba browser_navigate + screenshot
  IF browser juga gagal:
    IF Shopee → "Shopee anti-bot paling agresif, cek manual atau pake third-party API"
    IF lainnya → "Mungkin Cloudflare blocking / halaman JS-heavy. Coba lagi nanti."
  DO NOT: fabricate data

IF dapat partial result (3 item dari expected 20):
  → deliver yang ada + flag "partial result, ada lebih banyak di halaman"
```

## DECISION TREE: Harga Promo

```
IF halaman tampilkan "harga coret" + harga promo:
  → extract harga ACTUAL yang user bayar (harga promo)
  → OPTIONAL: include harga asli sebagai kolom terpisah
```

---

## VERIFICATION

```
□ Harga format Rupiah benar?
□ URL produk valid?
□ Gak ada data fabricated?
□ Disclaimer "bisa berubah" ada?
□ Marketplace + keyword jelas di output?

IF ada □ TIDAK → fix atau flag
```
