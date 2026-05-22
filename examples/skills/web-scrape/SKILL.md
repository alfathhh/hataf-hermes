---
name: web-scrape
description: Scrape halaman web untuk extract data terstruktur (JSON). Schema-first, anti-bot fallback ke browser, dan kalau gagal — bilang "tidak ditemukan", jangan ngarang.
version: 1.0.0
metadata:
  hermes:
    tags: [web, scrape, extract, monitoring]
    category: automation
    requires_toolsets: [web]
---

# Web Scrape

Skill untuk extract data terstruktur dari halaman web, dengan output JSON yang konsisten. Bisa dipake sekali atau dijadwalkan via cron.

## When to Use

User minta:
- "Scrape harga produk di [URL]"
- "Ambil daftar berita dari [URL]"
- "Pantau perubahan halaman [URL]"
- "Extract data tabel dari [URL]"

JANGAN pakai untuk:
- Scrape situs yang explicitly larang (cek robots.txt) tanpa konfirmasi user
- Scrape halaman login-required (gak ada login flow di skill ini)
- Mass scraping ribuan URL — itu butuh worker terpisah, beda alur

## Tools yang dipake

| Tool | Kapan |
|---|---|
| `web_search` | Cari URL kalau user kasih query, bukan URL spesifik |
| `web_extract` | Default — fetch HTML + extract dengan schema |
| `web_crawl` | Multi-page (pagination, nested links) |
| `browser_navigate` | Fallback kalau halaman dynamic / JS-heavy / Cloudflare |

Backend default: Firecrawl (set di config.yaml `web.backend`). Kalau Firecrawl gagal, kasih tau user untuk consider Tavily/Parallel atau self-host Firecrawl.

## Procedure

### 1. Klarifikasi schema sebelum scrape

JANGAN langsung jalan. Tanya dulu:

- "Field apa aja yang lo butuhkan dari halaman?"
- "Output dalam format apa? (JSON / CSV / list bullet)"
- "Berapa item maksimal? (page 1 doang? sampe N halaman?)"

Bikin JSON schema **eksplisit**:

```json
{
  "title": "string",
  "price": "number (IDR)",
  "rating": "number (0-5)",
  "url": "string (absolute URL)",
  "image_url": "string"
}
```

### 2. Robots.txt check

```bash
curl -s https://example.com/robots.txt | head -50
```

Kalau halaman target di-disallow buat scraping public, bilang user:
- "Halaman ini disallow di robots.txt. Mau lanjut scrape karena alasan tertentu (misal personal use) atau cari sumber lain?"

Jangan auto-skip robots tanpa user concious decision.

### 3. Scrape

```python
# Pakai web_extract dengan schema
result = web_extract(
    url="https://example.com/products",
    schema=MY_SCHEMA,
    prompt="Extract all products on the page. URL field harus absolute."
)
```

Kalau halaman static, ini cukup. Kalau gagal (empty result, atau struktur halaman ternyata di-render JS), fallback:

```python
# Browser fallback
browser_navigate(url="https://example.com/products")
browser_screenshot()  # liat halaman beneran rendering apa
# extract dari rendered HTML
```

### 4. Validasi hasil

Cek hasil sebelum present ke user:

- Apakah field schema ke-fill semua, atau banyak `null`?
- Apakah tipe data konsisten (semua price number, atau ada string "Rp 50.000"?)
- Apakah URL field absolute (bukan relative `/products/123`)?
- Apakah jumlah item plausible (kalau halaman jelas-jelas ada 24 produk tapi result cuma 3 — fishy)?

Kalau ada anomali, **flag di output** atau redo extract dengan prompt lebih spesifik.

### 5. Output

```markdown
## Scraping result: [URL]

**Tanggal akses**: [hari ini]
**Items extracted**: [N]
**Status**: ✅ Success / ⚠️ Partial / ❌ Failed

```json
[
  { ... },
  { ... }
]
```

## Catatan
- [Field yang gagal di-extract: ...]
- [Halaman butuh JS rendering, pakai browser fallback]
- [Pagination terdeteksi, ada N halaman lagi belum di-scrape]
```

### 6. Kalau gagal total

Skenario: web_extract dan browser keduanya nge-zero result.

JANGAN ngarang data. JANGAN substitute dengan "data perkiraan dari halaman serupa".

Output:

```markdown
## ❌ Scraping gagal: [URL]

**Diagnosa**:
- Web extract response: [paste error / empty]
- Browser fallback: [hasilnya juga kosong / gagal load / Cloudflare blocking]

**Kemungkinan penyebab**:
1. Halaman butuh login
2. Halaman di-block CDN (Cloudflare WAF, dll)
3. Struktur HTML berubah, schema gak match

**Rekomendasi**:
- Kasih API resmi situs target (kalau ada)
- Coba self-hosted Firecrawl (anti-bot lebih agresif tapi gak punya fire-engine cloud)
- Manual screenshot + vision_analyze sebagai workaround
```

## Pitfalls

### Pitfall 1: Halu data

Mengulang dari rule SOUL.md: jangan ngarang data biar output keliatan informatif. `null` lebih baik daripada placeholder.

### Pitfall 2: Salah parse harga

Banyak situs Indonesia pake format "Rp 1.250.000" — itu titik sebagai pemisah ribuan, bukan desimal. Setelah extract:
- Strip "Rp " 
- Strip titik (kalau format Indo)
- Convert ke int

Kalau ragu, simpan sebagai string asli dan biar user yang interpret.

### Pitfall 3: Encoding

Beberapa halaman pake encoding lain (windows-1252, gb2312). Kalau hasil ada `??` atau `Â`, kemungkinan encoding mismatch. Re-extract dengan explicit encoding atau pake browser fallback.

### Pitfall 4: Pagination

Banyak situs nge-load lebih banyak item via "Load More" button (JS) atau infinite scroll. `web_extract` cuma ambil halaman 1. Untuk multi-page:
- Cek struktur URL pagination (`?page=2`, `?offset=20`, dst)
- Loop manual dengan `web_extract`
- Atau pake `web_crawl` dengan max_pages limit

JANGAN ambil semua halaman secara default — bisa ribuan request. Tanya user dulu.

### Pitfall 5: Rate limit

Kalau loop scraping, kasih jeda (`time.sleep(2)` antar request). Kalau dapat 429 / 403, stop dan kasih tau user.

### Pitfall 6: Hak cipta & TOS

User yang tanggung jawab atas konten yang dia scrape. Tapi kalau jelas-jelas user mau scrape buat republikasi konten copyrighted (misal copy-paste artikel berita), beri caveat singkat.

## Verification

Sebelum kasih hasil ke user:

1. Apakah JSON valid (parse-able)?
2. Apakah struktur sesuai schema yang user mau?
3. Apakah ada item yang clearly suspect (price = 0, title = "undefined")?
4. Apakah URL absolute?
5. Apakah lo ngarang field yang gak ada di halaman?

Kalau ragu, run `web_extract` ulang dengan prompt yang lebih ketat.

## Untuk monitoring (cronjob)

Skill ini sering dipake dari cron. Detail di [docs/07-cron-scraping.md](../../../docs/07-cron-scraping.md).

Kuncinya:
- Output ke file JSON (path eksplisit)
- Compare hasil baru vs hasil lama (diff)
- Kirim notif Telegram cuma kalau ada perubahan signifikan
