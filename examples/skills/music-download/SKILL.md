---
name: music-download
description: Download lagu/video dari YouTube, Tidal, Spotify. Setting kualitas optimal. Personal use only.
version: 2.0.0
metadata:
  hermes:
    tags: [music, download, youtube, tidal, spotify, audio, video, yt-dlp]
    category: media
    requires_toolsets: [terminal]
---

# Music & Video Download

## ATURAN PENGGUNAAN

```
DO: download konten yang user PUNYA HAK AKSES (subscribe/beli/publik)
DO: personal offline listening only
DO NOT: redistribusi / piracy
DO NOT: bypass DRM (Apple Music, Spotify CDN)
```

---

## KAPAN PAKAI

```
IF user minta "download lagu dari YouTube" → PAKAI
IF user minta "download album dari Tidal" → PAKAI
IF user minta "download playlist Spotify" → PAKAI
IF user minta "download video YouTube" → PAKAI
IF user minta Apple Music download → BILANG "gak bisa" (DRM)
```

---

## PROCEDURE (ikuti exact)

### Step 1: Klarifikasi

```
TANYA:
1. "Platform mana?" (YouTube / Tidal / Spotify)
2. "Audio aja atau video?" (YouTube only untuk video)
3. "Kualitas?" (best / lossless / 320 / hemat)
4. "1 track, album, atau playlist?"
5. "Path output mau di mana?"
```

### Step 2: Cek prerequisites

```bash
which yt-dlp tiddl spotdl ffmpeg
# IF missing → kasih install command, tanya user mau install
```

### Step 3: Execute per platform

---

## PLATFORM 1: YouTube (yt-dlp)

```
IF audio MP3:
  yt-dlp -x --audio-format mp3 --audio-quality 0 --embed-thumbnail --add-metadata -o "~/Music/%(title)s.%(ext)s" "URL"

IF audio Opus (native, best real quality):
  yt-dlp -x -o "~/Music/%(title)s.%(ext)s" "URL"

IF video best quality:
  yt-dlp -o "~/Videos/%(title)s.%(ext)s" "URL"

IF video 1080p:
  yt-dlp -f "bv[height<=1080]+ba" --merge-output-format mp4 -o "~/Videos/%(title)s.%(ext)s" "URL"

IF playlist audio:
  yt-dlp -x --audio-format mp3 -o "~/Music/%(playlist_title)s/%(title)s.%(ext)s" "PLAYLIST_URL"

IF speed boost:
  yt-dlp --downloader aria2c "URL"
```

**⚠️ YouTube quality truth:**
| Source | Actual quality |
|---|---|
| Opus (WebM) | ~160 kbps (BEST YouTube punya) |
| MP3 320 (transcoded) | File 320kbps, SOURCE tetap 160 (NOT better) |

---

## PLATFORM 2: Tidal (tiddl) — TRUE LOSSLESS

```
REQUIREMENT: user HARUS punya akun Tidal aktif

IF first time:
  tiddl login   # OAuth browser login

IF track:
  tiddl track --quality lossless "URL"

IF album:
  tiddl album --quality lossless "URL"

IF playlist:
  tiddl playlist --quality lossless "URL"

IF master quality (HiFi Plus only):
  tiddl track --quality master "URL"

IF custom output:
  tiddl track --quality lossless --output "~/Music/Tidal/" "URL"
```

**Quality tiers:**
| Subscription | Quality | Format |
|---|---|---|
| HiFi Plus | Master 24-bit/192kHz | FLAC |
| HiFi | Lossless 16-bit/44.1kHz | FLAC (CD quality) |
| Any | High | AAC 320kbps |

---

## PLATFORM 3: Spotify (spotdl)

```
⚠️ TRUTH: spotdl download dari YOUTUBE, bukan Spotify CDN.
Quality = YouTube quality (~160kbps), BUKAN Spotify Premium quality.

IF track:
  spotdl "URL"

IF album:
  spotdl "URL"

IF playlist:
  spotdl "URL"

IF choose format:
  spotdl --format opus "URL"    # best real quality
  spotdl --format mp3 "URL"     # compatibility

IF custom output:
  spotdl --output "~/Music/Spotify/{artist}/{title}.{output-ext}" "URL"
```

**SELALU jelaskan ke user**: spotdl ≠ Spotify quality. Source = YouTube.

---

## PLATFORM 4: Apple Music

```
❌ TIDAK BISA download tanpa bypass DRM.

ALTERNATIF:
- Offline di app resmi (DRM-protected, gak bisa export)
- Beli di iTunes Store (DRM-free AAC 256kbps, lo own file-nya)
- Cari di YouTube → yt-dlp
- Cari di Tidal → tiddl (true FLAC)
```

---

## OUTPUT TEMPLATE

```markdown
## ✅ Download selesai

| | Detail |
|---|---|
| 🎵 Track | [nama] |
| 🎤 Artist | [artist] |
| 📀 Source | [platform] |
| 🔊 Quality | [format, bitrate] |
| 📁 Path | `[path]` |
| 📦 Size | [X MB] |

⚠️ [caveat quality kalau ada]
```

---

## CONTOH OUTPUT

```markdown
## ✅ Download selesai

| | Detail |
|---|---|
| 🎵 Track | Bohemian Rhapsody |
| 🎤 Artist | Queen |
| 📀 Source | Tidal (tiddl) |
| 🔊 Quality | FLAC 16-bit/44.1kHz (1411 kbps) |
| 📁 Path | `~/Music/Tidal/Queen - Bohemian Rhapsody.flac` |
| 📦 Size | 34.2 MB |

✅ True lossless — CD quality verified via ffprobe.
```

---

## DECISION TREE: Quality Recommendation

```
IF user mau quality terbaik → Tidal (true lossless FLAC)
IF user gak punya Tidal → YouTube Opus (~160kbps, best available free)
IF user minta MP3 320 dari YouTube → jelaskan: "File 320kbps tapi source 160, gak beneran better"
IF user minta FLAC dari YouTube/Spotify → jelaskan: "Container lossless tapi isi lossy, bukan true lossless"
IF user minta Apple Music → jelaskan: "DRM, gak bisa. Alternatif: [list]"
```

## DECISION TREE: Troubleshoot

```
IF yt-dlp throttled → tambah --downloader aria2c
IF Tidal auth expired → tiddl logout && tiddl login
IF YouTube geo-blocked → yt-dlp --geo-bypass "URL"
IF spotdl gak match lagu → "spotdl cocokkan via metadata, kadang salah match"
```

---

## VERIFICATION

```
□ File downloaded + ada di path?
□ Format sesuai request?
□ Quality limitation dijelaskan ke user?
□ User punya subscription (Tidal)?
□ Gak ngarang quality yang impossible dari source?

IF ada □ TIDAK → fix atau explain
```
