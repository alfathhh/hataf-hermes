---
name: music-download
description: Download lagu/video dari YouTube, Tidal (via tiddl), dan Spotify (via spotdl) dengan setting kualitas. Butuh akun aktif untuk Tidal. Personal use only.
version: 1.0.0
metadata:
  hermes:
    tags: [music, download, youtube, tidal, spotify, audio, video, yt-dlp]
    category: media
    requires_toolsets: [terminal]
---

# Music & Video Download

Skill untuk download lagu atau video dari YouTube, Tidal, dan Spotify menggunakan tools CLI.

## ⚠️ ATURAN PENGGUNAAN

- HANYA untuk konten yang lo PUNYA HAK AKSES (subscribe/beli/publik)
- Personal offline listening/viewing only
- BUKAN untuk redistribusi / piracy

## Prerequisites

```bash
pip install --upgrade yt-dlp tiddl spotdl
sudo apt install ffmpeg aria2
```

## When to Use

- "Download lagu X dari YouTube"
- "Download album Y dari Tidal dalam FLAC"
- "Download playlist Z dari Spotify"
- "Download video YouTube 4K"

---

## Platform 1: YouTube (yt-dlp)

### Audio

```bash
# MP3 320kbps (best YouTube can offer after transcode)
yt-dlp -x --audio-format mp3 --audio-quality 0 -o "~/Music/YouTube/%(title)s.%(ext)s" "URL"

# Native quality (Opus ~160kbps — actual YouTube source)
yt-dlp -x --audio-format opus -o "~/Music/YouTube/%(title)s.%(ext)s" "URL"

# FLAC container (still from lossy source — NOT true lossless)
yt-dlp -x --audio-format flac -o "~/Music/YouTube/%(title)s.%(ext)s" "URL"

# With metadata + thumbnail embedded
yt-dlp -x --audio-format mp3 --audio-quality 0 --embed-thumbnail --add-metadata -o "~/Music/YouTube/%(title)s.%(ext)s" "URL"
```

### Video

```bash
# Best quality
yt-dlp -f "bestvideo+bestaudio" --merge-output-format mp4 -o "~/Videos/%(title)s.%(ext)s" "URL"

# 4K
yt-dlp -f "bestvideo[height<=2160]+bestaudio" --merge-output-format mp4 -o "~/Videos/%(title)s.%(ext)s" "URL"

# 1080p
yt-dlp -f "bestvideo[height<=1080]+bestaudio" --merge-output-format mp4 -o "~/Videos/%(title)s.%(ext)s" "URL"

# 720p (hemat storage)
yt-dlp -f "bestvideo[height<=720]+bestaudio" --merge-output-format mp4 -o "~/Videos/%(title)s.%(ext)s" "URL"

# Audio only dari video (tanpa video track)
yt-dlp -x --audio-format mp3 -o "~/Music/YouTube/%(title)s.%(ext)s" "URL"
```

### Playlist

```bash
# Playlist → MP3
yt-dlp -x --audio-format mp3 --audio-quality 0 --embed-thumbnail --add-metadata \
  -o "~/Music/YouTube/%(playlist_title)s/%(playlist_index)03d - %(title)s.%(ext)s" "PLAYLIST_URL"

# Playlist → Video 1080p
yt-dlp -f "bestvideo[height<=1080]+bestaudio" --merge-output-format mp4 \
  -o "~/Videos/%(playlist_title)s/%(playlist_index)03d - %(title)s.%(ext)s" "PLAYLIST_URL"
```

### Speed up download

```bash
# Pakai aria2 (parallel download, lebih cepat)
yt-dlp --downloader aria2c --downloader-args aria2c:"-x 16 -s 16" "URL"
```

### YouTube quality reality check

| YouTube source | Actual quality | Notes |
|---|---|---|
| Opus (WebM) | ~160 kbps | Best audio YouTube has |
| AAC (M4A) | ~128 kbps | Fallback |
| MP3 320 (transcoded) | File 320kbps, source 160 | Bigger file, NOT better quality |
| FLAC (transcoded) | Lossless container, lossy source | NOT true lossless |

---

## Platform 2: Tidal (tiddl)

**WAJIB punya akun Tidal aktif.**

### Setup

```bash
# Login (buka browser OAuth)
tiddl login
```

### Download

```bash
# Track — auto quality sesuai subscription
tiddl track "TIDAL_URL"

# Quality options
tiddl track --quality master "URL"      # MQA/FLAC 24-bit (HiFi Plus ONLY)
tiddl track --quality lossless "URL"    # FLAC 16-bit/44.1kHz CD quality (HiFi)
tiddl track --quality high "URL"        # AAC 320kbps
tiddl track --quality low "URL"         # AAC 96kbps

# Album
tiddl album --quality lossless "TIDAL_ALBUM_URL"

# Playlist
tiddl playlist --quality lossless "TIDAL_PLAYLIST_URL"

# Custom output
tiddl track --quality lossless --output "~/Music/Tidal/" "URL"
```

### Tidal quality tiers

| Subscription | Best quality | Format | Bitrate |
|---|---|---|---|
| HiFi Plus | Master | MQA / FLAC 24-bit/192kHz | ~3000-9000 kbps |
| HiFi Plus | HiRes | FLAC 24-bit | ~2500-4500 kbps |
| HiFi | Lossless | FLAC 16-bit/44.1kHz | ~1411 kbps |
| Any | High | AAC | 320 kbps |
| Free | Low | AAC | 96 kbps |

### Troubleshoot tiddl

```bash
# Login expired
tiddl logout && tiddl login

# Cek subscription tier
tiddl info
```

---

## Platform 3: Spotify (spotdl)

**Cara kerja**: metadata dari Spotify → match + download dari YouTube. BUKAN rip dari Spotify CDN.

### Download

```bash
# Track
spotdl "SPOTIFY_TRACK_URL"

# Album
spotdl "SPOTIFY_ALBUM_URL"

# Playlist
spotdl "SPOTIFY_PLAYLIST_URL"

# Format options
spotdl --format mp3 "URL"
spotdl --format opus "URL"      # closest to YouTube native
spotdl --format m4a "URL"
spotdl --format flac "URL"      # NOT true lossless (YouTube source)

# Bitrate
spotdl --bitrate 320k "URL"     # still from ~160kbps source

# Output template
spotdl --output "~/Music/Spotify/{artist}/{album}/{title}.{output-ext}" "URL"

# Liked songs
spotdl --user-auth saved
```

### Spotify quality honesty

| What user thinks | Reality |
|---|---|
| "320kbps dari Spotify" | Source = YouTube ~160kbps, transcoded to 320 |
| "FLAC lossless" | Lossless container, lossy YouTube source inside |
| "Same as Spotify Premium" | NO — Premium = 320kbps OGG from Spotify CDN, spotdl ≠ that |

**Selalu jelaskan ini ke user.**

---

## Platform 4: Apple Music

```
❌ TIDAK BISA download secara legitimate tanpa bypass DRM.

Alternatif:
- Pakai fitur offline download di app Apple Music resmi
- Beli track di iTunes Store (DRM-free AAC 256kbps)
- Cari versi yang sama di YouTube (yt-dlp)
```

---

## Quality Comparison

| Platform | Tool | Best quality | True lossless? |
|---|---|---|---|
| Tidal HiFi Plus | tiddl | 24-bit/192kHz FLAC | ✅ YES |
| Tidal HiFi | tiddl | 16-bit/44.1kHz FLAC (CD) | ✅ YES |
| YouTube | yt-dlp | ~160 kbps Opus | ❌ |
| Spotify (spotdl) | spotdl | ~160 kbps (YouTube) | ❌ |
| Apple Music | ❌ | N/A | N/A |

**Ranking quality**: Tidal Master >>> Tidal Lossless >> Spotify official app > YouTube = spotdl

---

## Procedure

### 1. Klarifikasi

- Platform mana?
- Audio aja atau video? (YouTube only)
- Kualitas? (best / lossless / 320 / hemat)
- 1 track, album, atau playlist?
- Path output?

### 2. Cek prerequisites

```bash
which yt-dlp tiddl spotdl ffmpeg
```

### 3. Execute

Pilih command dari reference di atas.

### 4. Verify

```bash
ls -la ~/Music/[folder]/
ffprobe "file.flac" 2>&1 | grep -E "Stream|Duration|bitrate"
```

### 5. Output

```
✅ **Download selesai**

| | Detail |
|---|---|
| 🎵 Track | [nama] |
| 🎤 Artist | [artist] |
| 📀 Source | [platform] |
| 🔊 Quality | [format, bitrate] |
| 📁 Path | `~/Music/...` |
| 📦 Size | [X MB] |

⚠️ [caveat quality kalau ada]
```

---

## Pitfalls

### Pitfall 1: spotdl ≠ Spotify quality
SELALU jelaskan bahwa spotdl download dari YouTube, bukan Spotify CDN.

### Pitfall 2: Transcode lossy → "lossless" = placebo
MP3 → FLAC = bigger file, SAME quality. Jelaskan ini.

### Pitfall 3: YouTube geo-block
```bash
yt-dlp --geo-bypass "URL"
# atau pake proxy:
yt-dlp --proxy "socks5://127.0.0.1:1080" "URL"
```

### Pitfall 4: Tidal auth expired
```bash
tiddl logout && tiddl login
```

### Pitfall 5: yt-dlp throttled
```bash
yt-dlp --downloader aria2c "URL"
```

### Pitfall 6: Apple Music request
Bilang langsung gak bisa. Arahkan ke alternatif (iTunes Store / YouTube).

### Pitfall 7: Copyright content
Jangan download konten yang user JELAS gak punya hak. Kalau ragu, tanya dulu.

---

## Verification

1. File downloaded + ada di path yang benar?
2. Format sesuai request?
3. Quality limitation dijelaskan ke user?
4. User punya subscription (Tidal)?
5. Gak ngarang quality yang impossible dari source?
