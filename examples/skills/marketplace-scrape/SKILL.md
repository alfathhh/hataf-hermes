---
name: marketplace-scrape
description: Scrape data produk dari marketplace Indonesia (Tokopedia, Shopee, Bukalapak, Lazada). Extract harga, rating, jumlah terjual, seller info. Schema-first, JSON output.
version: 1.0.0
metadata:
  hermes:
    tags: [scrape, marketplace, tokopedia, shopee, ecommerce, price-monitor]
    category: automation
    requires_toolsets: [web, terminal]
---

# Marketplace Scrape

Skill untuk extract data produk dari marketplace Indonesia: Tokopedia, Shopee, Bukalapak, Lazada.

## ⚠️ Catatan penting

- Marketplace pake anti-bot (Cloudflare, rate limit, JS rendering)
- Scraping marketplace = **rapuh** (HTML bisa berubah kapan aja)
- Untuk monitoring harga jangka panjang, lebih reliable pake API resmi (kalau ada)
- Shopee dan Tokopedia load data via **JavaScript** — `web_extract` biasa bisa gagal

## When to Use

- "Scrape harga produk X di Tokopedia"
- "Bandingkan harga di Shopee vs Tokopedia"
- "Monitor harga seller Y"
- "Extract daftar produk dari search result marketplace"

## Tools yang dipake

| Situasi | Tool | Alasan |
|---------|------|--------|
| Default | `web_extract` | Coba dulu, gratis |
| JS-rendered / gagal | `browser_navigate` + `browser_screenshot` | Fallback buat dynamic page |
| Batch / API-based | `terminal` (Python script) | Untuk loop pagination |

---

## Platform 1: Tokopedia

### Approach: web_extract atau browser

```python
# Coba web_extract dulu
result = web_extract(
    url="https://www.tokopedia.com/search?q=mechanical+keyboard&ob=5",
    prompt="Extract semua produk: nama, harga (Rupiah), rating, jumlah terjual, seller name, URL produk. Format JSON array."
)
```

Kalau gagal (result kosong / HTML skeleton):

```python
# Fallback: browser
browser_navigate(url="https://www.tokopedia.com/search?q=mechanical+keyboard")
# Tunggu page load (2-3 detik)
browser_screenshot()
# Extract dari rendered page
```

### Schema output Tokopedia

```json
[
  {
    "name": "Keychron K2 V2 Wireless",
    "price": 1250000,
    "price_display": "Rp 1.250.000",
    "rating": 4.9,
    "sold": "500+ terjual",
    "seller": "Keychron Official",
    "seller_location": "Jakarta Selatan",
    "url": "https://www.tokopedia.com/..."
  }
]
```

### Tokopedia search URL pattern

```
https://www.tokopedia.com/search?q={query}&ob={sort}&pmin={min_price}&pmax={max_price}

Sort (ob):
- 5 = Ulasan (review)
- 4 = Harga tertinggi
- 3 = Harga terendah
- 2 = Terbaru
- 1 = Paling sesuai
```

---

## Platform 2: Shopee

### ⚠️ Shopee = paling susah di-scrape

Shopee render SEMUA via JavaScript + anti-bot agresif. `web_extract` hampir pasti gagal.

### Approach: browser atau API internal

```python
# Browser approach (lebih reliable)
browser_navigate(url="https://shopee.co.id/search?keyword=mechanical+keyboard")
# Tunggu load (Shopee lambat, 3-5 detik)
browser_screenshot()
# vision_analyze screenshot untuk extract data
```

### Shopee search URL pattern

```
https://shopee.co.id/search?keyword={query}&sortBy=sales&minPrice={min*100000}&maxPrice={max*100000}

sortBy: relevancy | sales | ctime (terbaru) | price (ascending)
```

### Alternative: Shopee punya internal API (public, tapi bisa berubah)

```bash
# Internal API — BISA BERUBAH KAPAN AJA
curl "https://shopee.co.id/api/v4/search/search_items?keyword=keyboard&limit=20&order=desc&sort_by=sales"
```

⚠️ API internal ini **gak official** dan bisa di-block/berubah tanpa notice. Gunakan sebagai fallback, bukan primary.

---

## Platform 3: Bukalapak

Bukalapak relatif lebih gampang di-scrape (less JS-heavy dibanding Shopee).

```python
result = web_extract(
    url="https://www.bukalapak.com/products?search%5Bkeywords%5D=mechanical+keyboard&search%5Bsort_by%5D=bestselling",
    prompt="Extract produk: nama, harga, rating, jumlah terjual, seller, URL."
)
```

---

## Platform 4: Lazada

Lazada pake JS rendering + Cloudflare.

```python
# Browser fallback biasanya diperlukan
browser_navigate(url="https://www.lazada.co.id/catalog/?q=mechanical+keyboard&sort=salescount")
browser_screenshot()
```

---

## Procedure

### 1. Klarifikasi

- Marketplace mana? (Tokped / Shopee / Bukalapak / Lazada / semua?)
- Keyword search apa?
- Filter: range harga? rating minimal? lokasi seller?
- Berapa item? (top 10? semua halaman 1? pagination?)
- Output format? (JSON / tabel / CSV)

### 2. Cek dulu pakai web_extract

Selalu coba `web_extract` dulu (murah, cepat). Kalau hasilnya kosong / gak match → fallback browser.

### 3. Parse harga

Marketplace Indonesia format harga: "Rp 1.250.000" atau "Rp1.250.000"

```python
# Parse harga Indonesian
price_str = "Rp 1.250.000"
price_int = int(price_str.replace("Rp", "").replace(".", "").replace(" ", ""))
# Result: 1250000
```

### 4. Output

```markdown
## 🛒 Hasil Scrape: [Marketplace] — "[keyword]"

**Tanggal**: [hari ini]
**Filter**: [yang dipakai]
**Total item**: [N]

| # | Produk | Harga | Rating | Terjual | Seller | Link |
|---|--------|-------|--------|---------|--------|------|
| 1 | ... | Rp ... | ⭐ 4.9 | 500+ | ... | [link] |
| 2 | ... | ... | ... | ... | ... | [link] |

⚠️ Data dari scraping, bisa berubah. Cek langsung di marketplace untuk harga terkini.
```

## Pitfalls

### Pitfall 1: Harga promo vs harga asli

Marketplace sering tampilkan "harga coret" (asli) dan harga promo. Pastiin yang di-extract = harga ACTUAL (yang user bayar), bukan harga coret.

### Pitfall 2: Rate limit

Jangan loop 100 halaman tanpa delay. Marketplace bakal block IP.

```python
import time
time.sleep(2)  # delay 2 detik antar request
```

### Pitfall 3: Lokasi filter

Harga bisa beda per lokasi (ongkir included?). Kalau user butuh harga untuk kota tertentu, sebutkan bahwa harga belum include ongkir.

### Pitfall 4: Shopee gagal terus

Shopee anti-bot paling agresif. Kalau browser juga gagal → bilang ke user bahwa Shopee memang susah di-scrape secara automated. Saran: cek manual atau pake third-party API (Apify, Crawlbase — berbayar).

### Pitfall 5: Produk gak ada

Kalau search result = 0 atau "Produk tidak ditemukan", BILANG. Jangan fabricate result.

## Verification

1. Apakah harga dalam format Rupiah yang benar?
2. Apakah URL produk valid (bisa diklik)?
3. Apakah gak ada data yang di-fabricate?
4. Apakah disclaimer "data bisa berubah" ada?
5. Apakah marketplace + keyword jelas di output?
