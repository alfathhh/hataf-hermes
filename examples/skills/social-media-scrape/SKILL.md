---
name: social-media-scrape
description: Scrape data publik dari Instagram, X (Twitter), dan Facebook. Extract posts, profile info, media. Pakai tools legitimate (instaloader, gallery-dl, yt-dlp). Cuma data PUBLIK — gak bypass private account.
version: 1.0.0
metadata:
  hermes:
    tags: [scrape, instagram, twitter, x, facebook, social-media]
    category: automation
    requires_toolsets: [terminal, web]
---

# Social Media Scrape

Skill untuk scrape data publik dari Instagram, X (Twitter), dan Facebook. 

## ⚠️ ATURAN KERAS

- **CUMA data PUBLIK** — gak bypass private account
- **Gak login ke akun orang lain** — kalau butuh login, pake akun USER sendiri
- **Gak mass-scrape** tanpa purpose jelas (jangan scrape 10.000 profile random)
- **Respect rate limit** — delay antar request
- **Gak untuk stalking / harassment** — kalau intent user jelas buat nge-stalk individu, refuse

## Prerequisites

```bash
pip install --upgrade instaloader gallery-dl yt-dlp snscrape
```

## When to Use

- "Download semua foto dari Instagram @username"
- "Scrape tweets dari @handle tentang topik X"
- "Ambil profile info Instagram @username"
- "Download video dari post Facebook publik"
- "Monitor mention brand di Twitter"

---

## Platform 1: Instagram

### Tool: `instaloader`

⚠️ Instagram makin restrictive di 2026. Login mungkin diperlukan untuk akses lebih dari beberapa profile.

### Quick commands

```bash
# Download semua post dari profile publik
instaloader profile username

# Download post terbaru (last 10)
instaloader profile username --count 10

# Download cuma foto (skip video)
instaloader profile username --no-videos

# Download stories (butuh login)
instaloader --login your_username --stories username

# Download highlight (butuh login)
instaloader --login your_username :stories username

# Download dengan metadata (caption, timestamp, likes)
instaloader profile username --no-compress-json
```

### Login (kalau diperlukan)

```bash
# Login sekali, session di-cache
instaloader --login your_username

# Atau lewat session file
instaloader --login your_username --sessionfile ~/.config/instaloader/session-your_username
```

⚠️ **Pakai akun lo sendiri.** Instagram bisa rate-limit atau ban sementara kalau terlalu agresif.

### Extract profile info (tanpa download media)

```bash
instaloader profile username --no-pictures --no-videos --no-video-thumbnails --metadata-json
```

Output JSON: followers, following, bio, external URL, post count.

### Output folder

```
username/
├── 2026-05-20_12-30-00_UTC.jpg
├── 2026-05-20_12-30-00_UTC.json   (metadata)
├── 2026-05-18_09-15-00_UTC.mp4
└── ...
```

---

## Platform 2: X (Twitter)

### Tool: `gallery-dl` atau `yt-dlp`

Twitter/X sering berubah API. Tools yang kerja hari ini bisa gagal besok.

### Quick commands — gallery-dl

```bash
# Download semua media dari timeline user
gallery-dl "https://x.com/username"

# Download media dari 1 tweet
gallery-dl "https://x.com/username/status/123456789"

# Download dengan metadata
gallery-dl --write-metadata "https://x.com/username"

# Filter: cuma gambar
gallery-dl --filter "extension in ('jpg', 'png')" "https://x.com/username"
```

### Quick commands — yt-dlp (untuk video)

```bash
# Download video dari tweet
yt-dlp "https://x.com/username/status/123456789"

# Download semua video dari timeline (recent)
yt-dlp "https://x.com/username/media"
```

### Scrape teks tweet (tanpa media)

```bash
# Pakai snscrape (kalau masih jalan di 2026 — cek dulu)
snscrape --jsonl twitter-user username > tweets.json

# Atau web_search sebagai fallback
```

⚠️ **snscrape mungkin udah gak jalan** karena X/Twitter sering block scraper. Test dulu. Kalau gagal, pakai `web_extract` ke tweet URL langsung.

### Twitter auth (kalau diperlukan)

```bash
# gallery-dl pakai cookies
gallery-dl --cookies-from-browser firefox "https://x.com/username"

# Atau export cookies manual
gallery-dl --cookies ~/cookies-twitter.txt "https://x.com/username"
```

---

## Platform 3: Facebook

### ⚠️ Facebook = paling susah

Facebook anti-scrape paling agresif. Hampir semua tools butuh login + sering ke-block.

### Approach: yt-dlp untuk video

```bash
# Download video publik Facebook
yt-dlp "https://www.facebook.com/watch?v=123456789"

# Download video dari page publik
yt-dlp "https://www.facebook.com/pagename/videos/"
```

### Approach: gallery-dl untuk foto

```bash
# Download foto dari album publik
gallery-dl "https://www.facebook.com/pagename/photos/"

# Butuh cookies biasanya
gallery-dl --cookies-from-browser chrome "https://www.facebook.com/pagename/photos/"
```

### Scrape post text Facebook

```python
# Paling reliable: web_extract langsung ke post URL publik
result = web_extract(
    url="https://www.facebook.com/pagename/posts/123456",
    prompt="Extract: post text, timestamp, like count, comment count, share count"
)
```

⚠️ Kalau gagal (login wall) → pakai `browser_navigate` + `browser_screenshot` + `vision_analyze`.

---

## Platform 4: TikTok (bonus)

```bash
# Download video TikTok (publik)
yt-dlp "https://www.tiktok.com/@username/video/123456789"

# Download tanpa watermark (kalau yt-dlp support)
yt-dlp --format best "https://www.tiktok.com/@username/video/123456789"

# Download semua video dari profile
yt-dlp "https://www.tiktok.com/@username"
```

---

## Procedure

### 1. Klarifikasi

- Platform mana?
- Username/URL target?
- Mau download apa? (media / teks post / profile info / semua)
- Berapa banyak? (last 10 / semua / date range)
- Lo punya akun di platform itu? (untuk login kalau diperlukan)

### 2. Cek apakah profile PUBLIK

```bash
# Instagram — cek tanpa login dulu
instaloader --no-pictures --no-videos profile username 2>&1 | head -5
# Kalau "Private profile" → STOP, bilang ke user
```

### 3. Execute

Pilih command dari reference per platform di atas.

### 4. Output

```markdown
## 📱 Scrape Result: [Platform] — @[username]

**Tanggal**: [hari ini]
**Profile**: [public/private]
**Data extracted**: [media / posts / profile info]
**Total items**: [N]

### Profile Info
| Field | Value |
|-------|-------|
| Username | @... |
| Followers | ... |
| Following | ... |
| Posts | ... |
| Bio | "..." |

### Media downloaded
📁 Path: `~/Downloads/[username]/`
- [N] foto
- [N] video
- [N] metadata JSON

⚠️ Data publik saja. Verifikasi langsung di platform untuk informasi terkini.
```

---

## Pitfalls

### Pitfall 1: Private account

JANGAN coba bypass private account. Kalau profile private:
```
❌ Profile @username bersifat private. Gw gak bisa scrape tanpa izin pemilik akun.
```

### Pitfall 2: Rate limit / temporary ban

- Instagram: max ~100 request/jam (tanpa login), bisa lebih rendah
- Twitter/X: sangat restrictive, bisa block setelah 10-20 request
- Facebook: block setelah beberapa request tanpa cookie

Mitigasi: delay, cookies, jangan greedy.

### Pitfall 3: Tools outdated

Social media sering update anti-bot. Tools yang jalan bulan lalu bisa gagal hari ini.

Kalau tool gagal:
1. Update tool (`pip install --upgrade instaloader`)
2. Cek GitHub issues tool tersebut
3. Fallback ke `browser_navigate` + `vision_analyze`
4. Bilang ke user bahwa platform lagi block

### Pitfall 4: Content yang inappropriate

Kalau user minta scrape profile yang jelas-jelas untuk stalking/harassment → refuse politely.

### Pitfall 5: Login credential security

Kalau user kasih login credentials:
- JANGAN simpan di memory atau log
- Pakai session file (bukan plain password di command)
- Remind user: "pake akun lo sendiri, jangan akun orang"

### Pitfall 6: Copyright media

Media yang di-download tetap copyright pemiliknya. Scraping = backup/research personal. Redistribusi / repost tanpa credit = pelanggaran.

---

## Tool comparison

| Tool | Platform | Media | Text | Login needed? |
|------|----------|-------|------|---------------|
| `instaloader` | Instagram | ✅ foto, video, stories | ✅ caption, metadata | Optional (untuk stories/highlights) |
| `gallery-dl` | Instagram, X, Facebook, TikTok, dll | ✅ foto, video | ⚠️ limited | Optional (cookies) |
| `yt-dlp` | X, Facebook, TikTok, YouTube | ✅ video | ❌ | Optional |
| `snscrape` | X (Twitter) | ❌ | ✅ tweet text | ❌ (tapi mungkin udah mati) |
| `web_extract` | Any public page | ❌ | ✅ text | ❌ |
| `browser` + `vision` | Any (fallback) | ✅ screenshot | ✅ via OCR | ❌ |

---

## Verification

1. Apakah profile yang di-scrape PUBLIC?
2. Apakah tools ter-update (latest version)?
3. Apakah output gak ngarang data (jumlah followers, post content)?
4. Apakah ada disclaimer "data publik saja"?
5. Apakah gak ada credential yang terexpose di output?
6. Apakah intent user legitimate (bukan stalking)?
