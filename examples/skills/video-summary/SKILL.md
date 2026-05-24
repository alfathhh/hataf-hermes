---
name: video-summary
description: Summarize video via pipeline yt-dlp + ffmpeg + Whisper STT + LLM. Output dengan timestamp.
version: 2.0.0
metadata:
  hermes:
    tags: [video, audio, transcription, summary]
    category: daily-task
    requires_tools: [terminal]
---

# Video Summary

## KAPAN PAKAI

```
IF user kasih URL YouTube/TikTok/Vimeo + minta summary → PAKAI
IF user kasih file video lokal + minta summary → PAKAI
IF user kasih file audio (.mp3/.wav) → PAKAI (skip download step)
IF video tanpa audio (visual-only) → JANGAN (butuh frame analysis)
IF live stream → JANGAN
```

---

## PROCEDURE (ikuti exact)

### Step 1: Cek prerequisites

```bash
which yt-dlp ffmpeg
python3 -c "import faster_whisper" 2>&1
```

```
IF missing → kasih install command:
  pip install --upgrade yt-dlp faster-whisper
  sudo apt install ffmpeg
```

### Step 2: Download audio

```
IF input = URL:
  yt-dlp -x --audio-format mp3 -o "/tmp/video-summary/%(id)s.%(ext)s" "[URL]"

IF input = file video lokal:
  ffmpeg -i [input.mp4] -vn -acodec libmp3lame /tmp/video-summary/audio.mp3

IF input = file audio:
  → skip, langsung ke Step 3

IF durasi > 2 jam:
  → warn user "STT akan lama, lanjut?"
```

### Step 3: Speech-to-text (Whisper)

```python
from faster_whisper import WhisperModel

model = WhisperModel("small", device="cpu", compute_type="int8")
segments, info = model.transcribe("/tmp/video-summary/audio.mp3", beam_size=5)

print(f"Language: {info.language}")
for seg in segments:
    print(f"[{seg.start:.1f}s -> {seg.end:.1f}s] {seg.text}")
```

```
Model selection:
IF audio jelas + English → "base" (fast)
IF audio noisy / Bahasa Indonesia → "medium" (better accuracy)
IF akurasi prioritas + ada GPU → "large-v3" (best)
DEFAULT → "small" (balance speed/quality)
```

### Step 4: Summarize transkrip

```
RULE: Setiap klaim di summary HARUS ada di transkrip
RULE: Quote langsung ambil PERSIS dari transkrip (jangan "perbaiki")
RULE: Timestamp HARUS dalam range durasi video
RULE: IF kata jelas salah-dengar → flag [salah dengar?]
```

### Step 5: Cleanup

```bash
rm -rf /tmp/video-summary/
# Kecuali user minta keep
```

---

## OUTPUT TEMPLATE

```markdown
# Summary: [Judul]

**Sumber**: [URL / filename]
**Durasi**: [HH:MM:SS]
**Bahasa**: [detected language]

## TL;DR (max 4 kalimat)
[Paling penting]

## Topik utama
1. [Topik A] — `[mm:ss]–[mm:ss]`
2. [Topik B] — `[mm:ss]–[mm:ss]`
3. [Topik C] — `[mm:ss]–[mm:ss]`

## Detail per topik

### [Topik A] (`[mm:ss]`)
[2-4 kalimat ringkas]
> "[Quote dari transkrip]" (`[mm:ss]`)

### [Topik B] (`[mm:ss]`)
[2-4 kalimat]

## Catatan
- Transkrip otomatis (Whisper) — bisa ada kata yang salah dengar
- [Flag bagian low-confidence kalau ada]
```

---

## CONTOH OUTPUT

```markdown
# Summary: "Cara Deploy Next.js ke VPS" (YouTube)

**Sumber**: https://youtube.com/watch?v=abc123
**Durasi**: 00:24:37
**Bahasa**: Indonesian

## TL;DR
Tutorial deploy Next.js ke VPS Ubuntu menggunakan Docker + Nginx reverse proxy + SSL Let's Encrypt. Cocok untuk production small-medium.

## Topik utama
1. Setup VPS + SSH — `00:00–03:45`
2. Install Docker + docker-compose — `03:45–08:20`
3. Dockerfile Next.js — `08:20–14:10`
4. Nginx reverse proxy + SSL — `14:10–20:30`
5. CI/CD dengan GitHub Actions — `20:30–24:37`

## Detail per topik

### Setup VPS (`00:00`)
Pakai Ubuntu 22.04 di DigitalOcean $6/bulan. SSH key setup, disable password auth.
> "Jangan pernah pake password buat SSH, selalu key-based" (`01:23`)

### Dockerfile Next.js (`08:20`)
Multi-stage build: stage 1 build, stage 2 production. Final image ~150MB.
> "Ini hasilnya 150 mega, bukan 1.2 giga kayak kalau gak multi-stage" (`12:45`)

### Nginx + SSL (`14:10`)
Certbot auto-renewal. Config proxy_pass ke container port 3000.

## Catatan
- Transkrip Whisper — beberapa nama library mungkin salah dengar
- Timestamp approximate (±5 detik)
```

---

## DECISION TREE: Video Panjang (>1 jam)

```
IF durasi > 1 jam:
  → chunk transkrip per 10 menit
  → summarize per chunk
  → synthesis: summary-of-summaries
  → OR: delegate_task ke subagent untuk chunk-level summary
```

## DECISION TREE: Bahasa Campur

```
IF video campur bahasa (Indo + English terms):
  → force language: model.transcribe(audio, language="id")
  → IF masih jelek → coba tanpa force, let Whisper detect
```

## DECISION TREE: Music Sections

```
IF ada musik/jingle tanpa speech:
  → skip dari summary
  → flag: "[mm:ss–mm:ss] musik, tidak ada speech"
```

---

## VERIFICATION

```
□ Setiap klaim di summary ada di transkrip?
□ Timestamp valid (dalam range durasi)?
□ Quote exact match dari transkrip?
□ Caveat "STT bisa salah" ada?
□ Gak ada info yang gw tambahin dari luar video?

IF ada □ TIDAK → fix
```
