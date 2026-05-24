---
name: social-media-scrape
description: Scrape data publik dari Instagram, X (Twitter), Facebook, TikTok. Cuma data PUBLIK, gak bypass private account.
version: 2.0.0
metadata:
  hermes:
    tags: [scrape, instagram, twitter, x, facebook, social-media]
    category: automation
    requires_toolsets: [terminal, web]
---

# Social Media Scrape

## ATURAN KERAS (NON-NEGOTIABLE)

```
DO: scrape data PUBLIK saja
DO NOT: bypass private account
DO NOT: login ke akun orang lain
DO NOT: mass-scrape tanpa purpose jelas
DO NOT: scrape untuk stalking / harassment

IF intent user jelas stalking → REFUSE
IF profile private → bilang "profile private, gak bisa" dan STOP
```

---

## KAPAN PAKAI

```
IF user minta download foto/video dari IG/X/TikTok public → PAKAI
IF user minta scrape tweets dari @handle → PAKAI
IF user minta profile info publik → PAKAI
IF user minta monitor brand mention → PAKAI
IF target = private account → JANGAN
IF intent = stalking → JANGAN (refuse)
```

---

## PROCEDURE (ikuti exact)

### Step 1: Klarifikasi

```
TANYA:
1. "Platform mana?" (IG / X / Facebook / TikTok)
2. "Username/URL target?"
3. "Mau apa?" (media / teks post / profile info / semua)
4. "Berapa banyak?" (last 10 / semua / date range)
5. "Lo punya akun di platform itu?" (untuk login kalau perlu)
```

### Step 2: Cek profile PUBLIC

```bash
# Instagram
instaloader --no-pictures --no-videos profile [username] 2>&1 | head -5
# IF "Private profile" → STOP

# X/Twitter
# Coba akses URL langsung
web_extract(url="https://x.com/[username]", prompt="Is this profile public?")
```

```
IF private → output "❌ Profile private. Gak bisa scrape." dan STOP
IF public → lanjut
```

### Step 3: Execute per platform

```
IF platform = Instagram:
  TOOL: instaloader
  # Download posts
  terminal: instaloader profile [username] --count [N]
  # Profile info only
  terminal: instaloader profile [username] --no-pictures --no-videos --metadata-json

IF platform = X (Twitter):
  TOOL: gallery-dl atau yt-dlp
  # Media
  terminal: gallery-dl "https://x.com/[username]"
  # Video
  terminal: yt-dlp "https://x.com/[username]/status/[id]"
  # Text: web_extract dari tweet URL

IF platform = TikTok:
  TOOL: yt-dlp
  # Video
  terminal: yt-dlp "https://www.tiktok.com/@[username]/video/[id]"
  # All videos
  terminal: yt-dlp "https://www.tiktok.com/@[username]"

IF platform = Facebook:
  TOOL: yt-dlp (video) atau gallery-dl (foto)
  # Video
  terminal: yt-dlp "https://www.facebook.com/watch?v=[id]"
  # Foto: gallery-dl --cookies-from-browser chrome "[URL]"
  # Text: web_extract dari post URL publik
```

### Step 4: Handle failures

```
IF tool gagal (rate limit / block):
  1. Update tool: pip install --upgrade [tool]
  2. IF masih gagal → fallback: browser_navigate + vision_analyze
  3. IF masih gagal → bilang "platform lagi block scraper"
  DO NOT: fabricate data
```

---

## OUTPUT TEMPLATE

```markdown
## 📱 Scrape Result: [Platform] — @[username]

**Tanggal**: [hari ini]
**Profile**: Public ✅
**Data extracted**: [media / posts / profile info]

### Profile Info
| Field | Value |
|-------|-------|
| Username | @[username] |
| Followers | [N] |
| Following | [N] |
| Posts | [N] |
| Bio | "[bio]" |

### Media downloaded
📁 Path: `~/Downloads/[username]/`
- [N] foto
- [N] video

⚠️ Data publik saja. Copyright tetap milik pemilik akun.
```

---

## CONTOH OUTPUT

```markdown
## 📱 Scrape Result: Instagram — @coffeeshop_jkt

**Tanggal**: 24 Mei 2026
**Profile**: Public ✅
**Data extracted**: 10 post terbaru + profile info

### Profile Info
| Field | Value |
|-------|-------|
| Username | @coffeeshop_jkt |
| Followers | 45.2K |
| Following | 312 |
| Posts | 847 |
| Bio | "Specialty coffee since 2019 ☕ Kemang & Senopati" |

### Media downloaded
📁 Path: `~/Downloads/coffeeshop_jkt/`
- 8 foto (.jpg)
- 2 video (.mp4)
- 10 metadata (.json)

### Command used
```bash
instaloader profile coffeeshop_jkt --count 10
```

⚠️ Data publik saja. Redistribusi tanpa credit = pelanggaran copyright.
```

---

## DECISION TREE: Tool Selection

```
IF Instagram + media → instaloader
IF Instagram + stories/highlights → instaloader --login [user_own_account]
IF X/Twitter + media → gallery-dl
IF X/Twitter + video → yt-dlp
IF TikTok + video → yt-dlp
IF Facebook + video → yt-dlp
IF Facebook + foto → gallery-dl (butuh cookies biasanya)
IF any platform + text only → web_extract dari URL post
IF semua gagal → browser_navigate + vision_analyze (fallback)
```

## DECISION TREE: Login Required

```
IF tool bilang "login required":
  → tanya user: "Lo punya akun sendiri di platform ini?"
  IF yes → "Mau login pake akun lo? (session di-cache lokal)"
  IF no → "Tanpa login, scraping limited. Mau lanjut yang bisa diambil?"
  DO NOT: simpan credentials di memory/log
```

---

## VERIFICATION

```
□ Profile yang di-scrape PUBLIC?
□ Tools ter-update?
□ Output gak ngarang data?
□ Disclaimer "data publik saja" ada?
□ Gak ada credential terexpose?
□ Intent user legitimate (bukan stalking)?

IF ada □ TIDAK → fix atau refuse
```
