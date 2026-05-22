# 04 — Skills (Knowledge On-Demand)

Tujuan: ngerti sistem skills Hermes, dan bikin 4 skill custom (research-citation, pdf-summarize, video-summary, web-scrape) yang langsung kepake.

> Sumber: [Skills System docs](https://hermes-agent.nousresearch.com/docs/user-guide/features/skills), [Working with Skills guide](https://hermes-agent.nousresearch.com/docs/guides/work-with-skills), [Creating Skills](https://hermes-agent.nousresearch.com/docs/developer-guide/creating-skills).

---

## 1. Konsep skill (kenapa hemat token)

Skill itu **dokumen instruksi yang di-load on-demand**, BUKAN selalu ada di system prompt.

Pola **progressive disclosure**:

| Level | Yang di-load | Token cost |
|---|---|---|
| **Level 0** | List skill (nama + deskripsi singkat) | ~3.000 token (selalu ada) |
| **Level 1** | Isi penuh `SKILL.md` 1 skill spesifik | bervariasi, di-load saat agent panggil `skill_view(name)` |
| **Level 2** | Reference file di dalam folder skill | bervariasi |

Jadi: lo bisa punya **50 skill**, dan agent cuma bayar token untuk yang **dipakai sekarang**, bukan semuanya.

---

## 2. Lokasi skills

```
~/.hermes/skills/
├── research-citation/
│   └── SKILL.md
├── pdf-summarize/
│   ├── SKILL.md
│   └── templates/
├── video-summary/
│   └── SKILL.md
└── web-scrape/
    ├── SKILL.md
    └── scripts/scrape_helper.py
```

Setiap skill = satu folder, **wajib punya `SKILL.md`** di dalamnya.

Hermes auto-discover semua skill di `~/.hermes/skills/` saat startup.

---

## 3. Format SKILL.md

```markdown
---
name: nama-skill
description: 1 kalimat jelas, ini yang dilihat agent saat skills_list()
version: 1.0.0
metadata:
  hermes:
    tags: [research, citation]
    category: research
---

# Judul Skill

## When to Use
Trigger condition. Kapan skill ini relevan?

## Procedure
1. Step
2. Step
3. Step

## Pitfalls
Yang gampang salah. Cara hindari.

## Verification
Cara cek hasilnya bener.
```

> **Penting**: `description` itu **PR copy** untuk agent. Tulis spesifik & action-oriented. "Read PDF and summarize" lebih bagus dari "PDF helper".

---

## 4. Cara pake skill

Setelah skill terdaftar, lo bisa:

```bash
# Di TUI atau Telegram:
/research-citation        # langsung trigger skill, agent tanya butuh apa
/pdf-summarize ~/Downloads/paper.pdf
/web-scrape https://news.ycombinator.com

# List semua skill:
/skills

# Atau minta agent natural language:
> bantu aku summarize PDF ini → agent auto-load skill pdf-summarize
```

---

## 5. Skill 1: Research dengan citation (anti-halu di task riset)

**Tujuan**: kalau lo nanya hal faktual, agent harus search → ekstrak → kasih jawaban dengan sumber yang bisa dicek. Bukan dari memori.

File lengkap: [`examples/skills/research-citation/SKILL.md`](../examples/skills/research-citation/SKILL.md).

Install:

```bash
mkdir -p ~/.hermes/skills/research-citation
cp examples/skills/research-citation/SKILL.md ~/.hermes/skills/research-citation/
```

Pakai:

```
/research-citation kapan UU ITE direvisi terakhir?
```

Agent bakal:
1. `web_search` ke beberapa sumber
2. Filter: prioritas situs pemerintah / news kredibel
3. Kasih jawaban + URL setiap klaim
4. Kalau gak nemu sumber primer → bilang "tidak ditemukan"

---

## 6. Skill 2: PDF summarize (chunking + citation)

**Tujuan**: kasih PDF (paper, laporan, regulasi) → dapet summary terstruktur dengan section/halaman reference.

File: [`examples/skills/pdf-summarize/SKILL.md`](../examples/skills/pdf-summarize/SKILL.md).

Install:

```bash
mkdir -p ~/.hermes/skills/pdf-summarize
cp examples/skills/pdf-summarize/SKILL.md ~/.hermes/skills/pdf-summarize/
```

Skill ini guide agent untuk:
1. Read PDF (built-in tool Hermes bisa baca PDF teks)
2. Chunk ke section
3. Per section: kasih summary 2-3 kalimat + halaman/section reference
4. Akhirnya: kasih TL;DR overall + main claims dengan refs

---

## 7. Skill 3: Video summary (audio extract + STT + summarize)

**Tujuan**: kasih file video / URL YouTube → dapet summary teks.

> **Caveat penting**: Hermes tidak punya YouTube downloader built-in. Lo butuh `yt-dlp` ter-install di sistem. Skill ini guide agent untuk pakai `terminal` tool jalanin yt-dlp + ffmpeg + Whisper STT.

File: [`examples/skills/video-summary/SKILL.md`](../examples/skills/video-summary/SKILL.md).

Install (setelah `yt-dlp` & `ffmpeg` terinstall):

```bash
# Install dependency dulu
pip install yt-dlp                       # atau: brew install yt-dlp
sudo apt install ffmpeg                  # atau: brew install ffmpeg
pip install faster-whisper               # untuk STT lokal

# Install skill
mkdir -p ~/.hermes/skills/video-summary
cp examples/skills/video-summary/SKILL.md ~/.hermes/skills/video-summary/
```

Pakai:

```
/video-summary https://www.youtube.com/watch?v=XXXX
```

Pipeline:
1. `yt-dlp` download audio aja (lebih cepet, bukan video)
2. `ffmpeg` (kalau perlu konversi)
3. `faster-whisper` STT → transkrip
4. Hermes (model utama) summarize transkrip

---

## 8. Skill 4: Web scraping (anti-bot + structured extract)

**Tujuan**: scrape halaman web → output JSON terstruktur. Untuk monitoring harga, harvest data, dst.

File: [`examples/skills/web-scrape/SKILL.md`](../examples/skills/web-scrape/SKILL.md).

Backbone: web tool Hermes (`web_search`, `web_extract`, `web_crawl`) — ini menggunakan Firecrawl/Tavily/Parallel.

Install:

```bash
mkdir -p ~/.hermes/skills/web-scrape
cp examples/skills/web-scrape/SKILL.md ~/.hermes/skills/web-scrape/
```

Pakai:

```
/web-scrape https://www.tokopedia.com/search?q=mechanical+keyboard
```

Skill ini bilang ke agent:
1. Identify target schema dulu (apa yang user mau di-extract: harga? rating? URL?)
2. `web_extract` dengan schema explicit
3. Output JSON valid (bukan teks campur)
4. Kalau halaman dynamic / Cloudflare → fallback ke `browser_navigate`
5. Kalau gagal → bilang "tidak ditemukan", **bukan ngarang**

---

## 9. Browse skill dari registry public

Hermes terintegrasi sama beberapa registry skill (dokumentasi: [Skills System](https://hermes-agent.nousresearch.com/docs/user-guide/features/skills)):

| Registry | Trust level | Source ID |
|---|---|---|
| Official optional skills (Nous) | builtin | `official` |
| Anthropic skills | trusted | `anthropics/skills` |
| OpenAI skills | trusted | `openai/skills` |
| skills.sh (Vercel) | community | `skills-sh` |
| ClawHub | community | `clawhub` |
| LobeHub | community | `lobehub` |

Browse & install:

```bash
hermes skills browse                        # liat semua
hermes skills search pdf                    # cari berdasarkan keyword
hermes skills inspect openai/skills/k8s     # preview sebelum install
hermes skills install official/security/1password
```

> **Hati-hati**: skill **community** (skills.sh, dll) lewat security scanner Hermes, tapi `--force` bisa override warning. Untuk production agent, prefer `official` & `trusted` sources.

---

## 10. Bikin skill custom dari scratch

Workflow yang gw rekomendasikan:

1. Identifikasi task yang lo ulang-ulang prompt manual → calon skill
2. Bikin folder `~/.hermes/skills/<nama>/`
3. Tulis `SKILL.md` dengan **frontmatter wajib** (`name`, `description`, `version`)
4. Test: `/<nama-skill>` di TUI
5. Iterate berdasarkan output yang gak sesuai

Atau biar agent yang bikin sendiri (Hermes punya tool `skill_manage`):

```
> bantu aku bikin skill yang setiap minggu cek status server X via SSH dan email aku kalau down. tulis sebagai SKILL.md di folder yang tepat.
```

Hermes bakal `skill_manage(action="create", ...)` dan generate file SKILL.md.

---

## 11. Kapan pake skill, kapan pake AGENTS.md

| Kebutuhan | Pake apa |
|---|---|
| Aturan project (build, test, file path) | `AGENTS.md` |
| Workflow task spesifik (riset, scrape, summarize) | Skill |
| Identitas global (anti-halu, style) | `SOUL.md` |
| Fakta personal (preferences, env) | Memory (auto, lewat `memory` tool) |

Kuncinya: **skill itu prosedural**, **AGENTS.md itu deklaratif**.

---

## 12. Yang gw belum yakin

- **Token cost level 0 (skills_list)**: dokumentasi resmi sebut "~3k tokens" untuk meta-list semua skills. Kalau lo punya 50+ skill, level 0 ini bisa membengkak. **Belum gw verifikasi sendiri**. Mitigasi: kalau punya banyak skill, beberapa di-set `requires_toolsets` atau `fallback_for_toolsets` biar conditional, jadi gak semua muncul di list.
- **Apakah skill bisa load model lain (cheap) untuk dirinya sendiri?** Skill bisa pake `delegate_task` untuk spawn subagent dengan model berbeda. Detail di doc 09.

---

## Lanjut

→ [05 — Tools, MCP, dan Vision](05-tools-dan-mcp.md): tools yang available, MCP servers, vision setup khusus.
