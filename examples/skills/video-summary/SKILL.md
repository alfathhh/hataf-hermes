---
name: video-summary
description: Summarize video (file lokal atau URL YouTube/etc) dengan pipeline yt-dlp + ffmpeg + Whisper STT + LLM summarize. Output disertai timestamp untuk klaim penting.
version: 1.0.0
metadata:
  hermes:
    tags: [video, audio, transcription, summary]
    category: daily-task
    requires_tools: [terminal]
---

# Video Summary

Skill untuk meringkas video lewat pipeline: download audio → transcribe → summarize. Output mencakup timestamp, jadi user bisa langsung lompat ke bagian yang relevan di video aslinya.

## Prerequisites (HARUS terinstall di sistem)

Cek dulu sebelum mulai:

```bash
which yt-dlp        # https://github.com/yt-dlp/yt-dlp
which ffmpeg
python3 -c "import faster_whisper" 2>&1   # untuk STT lokal
```

Kalau belum ada, kasih tau user untuk install:

```bash
# yt-dlp
pip install --upgrade yt-dlp
# atau: brew install yt-dlp / apt install yt-dlp

# ffmpeg
sudo apt install ffmpeg            # Linux
brew install ffmpeg                # macOS

# Whisper STT lokal (gratis, jalan di CPU/GPU)
pip install faster-whisper
```

JANGAN auto-install dependency tanpa konfirmasi user.

## When to Use

User memberikan:
- URL YouTube / TikTok / Vimeo / dst
- Path file video lokal (.mp4, .mkv, .webm, dst)
- Path file audio (.mp3, .wav, .m4a) — bisa skip download step

Permintaan eksplisit untuk:
- Summary
- Transkrip + summary
- Cari moment spesifik di video ("kapan dia ngomong tentang X")
- Translasi transkrip

JANGAN pakai untuk:
- Video tanpa audio (visual-only) — itu butuh frame analysis, beda alur
- Live stream

## Procedure

### 1. Validasi input

- Kalau URL: konfirmasi format yang yt-dlp support (cek `yt-dlp --list-extractors` kalau ragu)
- Kalau file lokal: cek file exist, durasi (`ffprobe`)
- Kalau durasi > 2 jam: warn user bahwa STT akan lama, tanya konfirmasi

### 2. Download audio aja (lebih cepat)

```bash
# YouTube — extract audio aja
yt-dlp -x --audio-format mp3 -o "/tmp/video-summary/%(id)s.%(ext)s" <URL>

# Hasil: /tmp/video-summary/<video_id>.mp3
```

Kalau file lokal sudah audio, skip langkah ini.

Kalau file lokal video, extract audio:

```bash
ffmpeg -i input.mp4 -vn -acodec libmp3lame /tmp/video-summary/audio.mp3
```

### 3. Speech-to-text dengan Whisper

Pakai `faster-whisper`. Pilih model size:

| Model | RAM | Kecepatan | Akurasi | Kapan pakai |
|---|---|---|---|---|
| `tiny` | ~1 GB | Sangat cepat | Kurang | Cuma untuk test |
| `base` | ~1 GB | Cepat | OK | Audio jelas, bahasa Inggris |
| `small` | ~2 GB | Sedang | Bagus | Default rekomendasi |
| `medium` | ~5 GB | Lambat | Bagus | Bahasa Indonesia, audio noisy |
| `large-v3` | ~10 GB | Sangat lambat | Terbaik | Akurasi prioritas, ada GPU |

Skrip Python yang dipakai (lewat `terminal`):

```python
from faster_whisper import WhisperModel

model = WhisperModel("small", device="cpu", compute_type="int8")
segments, info = model.transcribe("/tmp/video-summary/audio.mp3", beam_size=5)

print(f"Detected language: {info.language}")
for seg in segments:
    print(f"[{seg.start:.1f}s -> {seg.end:.1f}s] {seg.text}")
```

Output: array of `{start, end, text}` segments.

Save transkrip ke `/tmp/video-summary/<video_id>.transcript.txt`.

### 4. Summarize transkrip

Sekarang LLM (model utama Hermes) summarize transkrip yang udah diberi timestamp.

Format output:

```markdown
# Summary: [Judul kalau dari yt-dlp metadata, atau "Untitled"]

**Sumber**: [URL atau filename]
**Durasi**: [HH:MM:SS]
**Bahasa**: [hasil deteksi Whisper]

## TL;DR (max 4 kalimat)
[Paling penting]

## Topik utama
1. [Topik A] — `[start_time]–[end_time]`
2. [Topik B] — `[start_time]–[end_time]`
3. [Topik C] — `[start_time]–[end_time]`

## Per topik

### [Topik A] (`[mm:ss]`)
[2-4 kalimat ringkas. Quote langsung kalau ada statement penting:]
> "Quote eksplisit dari transkrip" (`[mm:ss]`)

### [Topik B] (`[mm:ss]`)
...

## Yang TIDAK dibahas (kalau user nanya)
- [Hal yang user tanya tapi video gak bahas]

## Catatan
- Transkrip otomatis (Whisper). Kemungkinan ada kata yang salah dengar — verifikasi quote langsung di video kalau krusial.
- [Kalau ada bagian yang Whisper bilang "low confidence", flag di sini]
```

### 5. Cleanup

Hapus file audio sementara setelah selesai:

```bash
rm -rf /tmp/video-summary/*.mp3 /tmp/video-summary/*.transcript.txt
```

(Kecuali user minta keep — tanya dulu.)

## Pitfalls

### Pitfall 1: Halu quote

Setelah summary, user mungkin nanya "tolong dikutip persis". STT bisa salah dengar (misal "sequel" jadi "sekuel" atau "DeepSeek" jadi "the seek"). 

**Aturan**: kalau quote ditampilkan, ambil **persis dari transkrip**, jangan "perbaiki" dari pengetahuan umum. Kalau ada kata yang clearly salah-dengar, flag dengan `[salah dengar?]`.

### Pitfall 2: Video bahasa campur

Whisper bisa salah deteksi kalau audio dual-language (bahasa Indo dengan istilah English campur). Kalau hasil tampak aneh, coba force language:

```python
segments, info = model.transcribe(audio, language="id")
```

### Pitfall 3: Music / non-speech sections

Whisper bisa "denger" lirik music sebagai ucapan dan kasih transkrip. Kalau ada section yang clearly musical interlude, sebutkan dan skip dari summary.

### Pitfall 4: YouTube auto-caption sebagai shortcut

YouTube punya auto-caption built-in. Kalau available (cek `yt-dlp --write-auto-sub --skip-download`), itu bisa **lebih cepat** daripada Whisper, tapi kualitas variable. Pakai sebagai fallback kalau Whisper terlalu lambat di hardware lo.

### Pitfall 5: Konten yang melanggar TOS

Jangan transkrip video yang user JELAS-JELAS gak punya hak akses ke transkripnya (misal scrape ribuan video private). Skill ini untuk konten yang user own, public domain, atau fair use research.

## Verification

Sebelum kirim summary:

1. Apakah setiap claim di summary ada di transkrip? (sample 2-3)
2. Apakah timestamp valid (dalam range durasi video)?
3. Apakah quote langsung exact match dengan transkrip?
4. Apakah ada caveat tentang STT bisa salah dengar?

## Untuk video panjang (> 1 jam)

- Transkrip bisa puluhan ribu token, overflow context.
- Strategy: chunk transkrip per 5-10 menit, summarize per chunk, lalu summary-of-summaries.
- Atau pakai delegate_task — spawn subagent dengan model murah untuk chunk-level summary, model utama untuk synthesis.
