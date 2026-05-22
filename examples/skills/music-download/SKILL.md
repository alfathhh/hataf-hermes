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

### Audio — command simpel

```bash
# Lagu → MP3 (best quality dari YouTube)
yt-dlp -x --audio-format mp3 --audio-quality 0 --embed-thumbnail --add-metadata -o "~/Music/%(title)s.%(ext)s" "URL"

# Lagu → Opus (native YouTube, file kecil, quality sama)
yt-dlp -x -o "~/Music/%(title)s.%(ext)s" "URL"

# Playlist → semua jadi MP3
yt-dlp -x --audio-format mp3 --audio-quality 0 -o "~/Music/%(playlist_title)s/%(title)s.%(ext)s" "PLAYLIST_URL"
```

### Video — command simpel

```bash
# Video best quality
yt-dlp -o "~/Videos/%(title)s.%(ext)s" "URL"

# Video 1080p (hemat storage)
yt-dlp -f "bv[height<=1080]+ba" --merge-output-format mp4 -o "~/Videos/%(title)s.%(ext)s" "URL"

# Video 720p (lebih hemat)
yt-dlp -f "bv[height<=720]+ba" --merge-output-format mp4 -o "~/Videos/%(title)s.%(ext)s" "URL"

# Playlist → video semua
yt-dlp -o "~/Videos/%(playlist_title)s/%(title)s.%(ext)s" "PLAYLIST_URL"
```

### Speed boost

```bash
# Pake aria2 (download paralel, 5-10x lebih cepet)
yt-dlp --downloader aria2c "URL"
```

### YouTube quality reality check

| YouTube source | Actual quality | Notes |
|---|---|---|
| Opus (WebM) | ~160 kbps | Best audio YouTube punya |
| AAC (M4A) | ~128 kbps | Fallback |
| MP3 320 (transcoded) | File 320kbps, SOURCE 160 | Bigger file, NOT better quality |

⚠️ **YouTube gak punya true lossless.** Max = ~160kbps Opus. Convert ke FLAC/320 = placebo (file besar, quality SAMA).

---

## Platform 2: Tidal (tiddl) — BEST quality option

**WAJIB punya akun Tidal aktif.** Lo bilang lo langganan — perfect.

### Setup (sekali aja)

```bash
tiddl login          # buka browser, OAuth login
```

### Quick commands

```bash
# Track (auto best quality sesuai subscription lo)
tiddl track "URL"

# Album lossless
tiddl album --quality lossless "URL"

# Playlist lossless
tiddl playlist --quality lossless "URL"

# Master quality (24-bit, HiFi Plus only)
tiddl track --quality master "URL"
```

### Quality options

```bash
tiddl track --quality master "URL"      # 24-bit/192kHz FLAC (HiFi Plus)
tiddl track --quality lossless "URL"    # 16-bit/44.1kHz FLAC CD quality (HiFi)
tiddl track --quality high "URL"        # AAC 320kbps
tiddl track --quality low "URL"         # AAC 96kbps
```

### Output folder

```bash
tiddl track --quality lossless --output "~/Music/Tidal/" "URL"
```

### Tidal quality tiers

| Subscription | Best quality | Format | Bitrate |
|---|---|---|---|
| HiFi Plus | Master | MQA / FLAC 24-bit/192kHz | ~3000-9000 kbps |
| HiFi Plus | HiRes | FLAC 24-bit | ~2500-4500 kbps |
| HiFi | Lossless | FLAC 16-bit/44.1kHz | ~1411 kbps |
| Any | High | AAC | 320 kbps |

### Troubleshoot

```bash
tiddl logout && tiddl login    # kalau auth expired
tiddl info                     # cek subscription tier lo
```

**INI satu-satunya platform yang kasih TRUE LOSSLESS.** Kalau lo mau quality terbaik → selalu prefer Tidal.

---

## Platform 3: Spotify (spotdl)

**Cara kerja**: metadata dari Spotify → match + download dari YouTube. BUKAN rip dari Spotify CDN.

### Quick commands (yang simpel)

```bash
# Track
spotdl "URL"

# Album
spotdl "URL"

# Playlist
spotdl "URL"

# Liked songs (butuh auth sekali)
spotdl --user-auth saved
```

### Pilih format

```bash
spotdl --format mp3 "URL"          # MP3
spotdl --format opus "URL"         # Opus (best real quality)
spotdl --format m4a "URL"          # M4A/AAC
spotdl --output "~/Music/Spotify/{artist}/{album}/{title}.{output-ext}" "URL"
```

### Spotify quality — HARUS JUJUR ke user

| Persepsi | Realita |
|---|---|
| "320kbps dari Spotify" | Source = YouTube ~160kbps, di-transcode ke 320 (file besar, quality SAMA) |
| "FLAC lossless" | Container lossless, isi lossy. BUKAN true lossless. |
| "Sama kayak Spotify Premium" | BUKAN. Premium = 320kbps OGG dari CDN Spotify. spotdl = YouTube audio. |

**SELALU jelaskan ini ke user.** Jangan biarin mereka mikir dapet quality Spotify asli.

### Kenapa gak bisa rip CDN Spotify langsung?

Tools yang rip CDN Spotify (berbasis librespot) = **violate TOS Spotify**. Risiko:
- Akun lo ke-ban permanent
- Legal exposure (DMCA)

Gw gak akan guide cara bypass DRM Spotify. Kalau lo mau quality asli Spotify → dengerin di app resmi (offline download mode di Premium udah 320kbps OGG).

**Alternatif kalau mau lossless**: pake Tidal (true FLAC via tiddl).

---

## Platform 4: Apple Music

```
❌ TIDAK BISA download/rip secara legitimate tanpa bypass DRM.

Tools yang ada di internet untuk rip Apple Music = bypass FairPlay DRM = illegal
di banyak jurisdiksi + violate TOS Apple → akun ke-ban.
```

**Alternatif yang BISA lo lakuin**:

| Opsi | Cara |
|------|------|
| Offline di app resmi | Apple Music app → download → dengerin offline (DRM-protected, gak bisa export) |
| Beli di iTunes Store | DRM-free AAC 256kbps. Lo OWN file-nya. Bisa copy ke mana aja. |
| Cari di YouTube | Banyak yang ada versi official. `yt-dlp` download. Quality ~160kbps. |
| Cari di Tidal | Kalau lo juga langganan Tidal, lagu yang sama biasanya ada. Download FLAC lossless via tiddl. |

**Rekomendasi gw**: kalau lo mau lossless yang bisa lo export → **Tidal via tiddl**. Itu satu-satunya yang legitimately kasih true FLAC tanpa DRM bypass.

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
