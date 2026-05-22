---
name: claude-superpowers
description: Replika workflow Claude Superpowers di Hermes — extended thinking, artifacts, citations, vision, code execution, memory dalam satu alur terintegrasi. Panggil untuk task kompleks yang butuh multi-capability.
version: 1.0.0
metadata:
  hermes:
    tags: [reasoning, artifacts, citations, vision, code-execution, multi-capability]
    category: meta
---

# Claude Superpowers (Hermes Edition)

Skill meta yang menggabungkan semua kapabilitas Hermes menjadi satu alur kerja terintegrasi — mereplikasi pengalaman Claude Superpowers (Extended Thinking, Artifacts, Web Search, Vision, Code Execution, Memory, Citations) di ekosistem Hermes.

## When to Use

Panggil skill ini untuk task yang butuh **kombinasi kemampuan**:
- "Analisa screenshot app ini, tulis improvement plan, implementasi, dan test"
- "Research topik X, tulis report dengan citation, generate chart dari data"
- "Baca PDF ini, extract data, run analisis Python, output summary"
- "Pahami codebase ini, propose refactor, implement, verify"

Atau ketika user bilang: "kerja seperti Claude" / "full power" / "deep work mode"

## Capability Map (apa yang tersedia)

### 1. Extended Thinking → `/reasoning high` + structured decomposition

Sebelum menjawab task kompleks:

```
INTERNAL CHECKLIST:
□ Apakah task ini butuh decomposition? (>1 sub-problem)
□ Apakah ada hidden assumptions yang harus di-surface?
□ Apakah ada multiple valid approaches?
□ Apakah output butuh verification?

Kalau ≥2 checked → decompose dulu, jangan langsung eksekusi.
```

**Action**: Set reasoning effort ke high/xhigh di awal task kompleks:
```
/reasoning high
```

Lalu decompose task eksplisit (lihat skill `deep-analysis` untuk framework detail).

### 2. Artifacts → File Output

Claude punya "Artifacts" panel untuk code/doc. Di Hermes, equivalent = **tulis ke file**.

Aturan:
- Code output > 20 baris → `write_file` ke path yang masuk akal
- Document/report → `write_file` ke `~/output/` atau working directory
- HTML preview → tulis file, kasih path ke user

**Pattern:**

```python
# Instead of pasting 200 lines in chat:
write_file(
    path="output/analysis-report.md",
    content="..."
)
# Then tell user: "Report tersimpan di output/analysis-report.md"
```

Untuk output yang user mau langsung liat di chat: kasih summary di chat + full version di file.

### 3. Web Search + Citations → `web_search` + `web_extract`

Untuk setiap klaim faktual:

1. `web_search` → ambil top results
2. `web_extract` → fetch detail dari sumber primer
3. Cite per klaim: `[sumber](URL)`
4. Kalau gak ketemu → "tidak ditemukan di pencarian"

**JANGAN:**
- Skip search dan jawab dari "pengetahuan umum" kalau pertanyaannya time-sensitive
- Fabricate URL

**Lihat skill `research-citation` untuk detail lengkap.**

### 4. Vision → `vision_analyze`

Saat user attach gambar / screenshot / diagram:

```python
vision_analyze(
    image=attached_image,
    prompt="Describe this UI in detail: layout, components, colors, text content, any issues visible"
)
```

Gunakan untuk:
- Screenshot review (bug visual, UI critique)
- Diagram understanding (architecture, flowchart)
- Document scan (receipt, handwriting, whiteboard)
- Chart/graph data extraction

**Limitations:**
- Vision model = auxiliary (Gemini Flash / GPT-4o), bukan model utama
- Bisa miss detail kecil — kalau kritis, minta user confirm

### 5. Code Execution → `execute_code` tool

Run Python langsung di sandbox:

```python
execute_code("""
import pandas as pd
import json

# Load data
data = json.loads(open('/tmp/data.json').read())
df = pd.DataFrame(data)

# Analysis
summary = df.describe()
print(summary.to_markdown())

# Plot
import matplotlib.pyplot as plt
df['price'].plot(kind='line')
plt.savefig('/tmp/price_chart.png')
print("Chart saved to /tmp/price_chart.png")
""")
```

**Kapan pakai code execution vs terminal:**
- Code execution: data processing, math, charting, validation logic
- Terminal: system commands, git, file management, install packages

### 6. Memory → Persistent across sessions

Setiap kali task selesai, evaluate:

```
MEMORY CHECK:
- Apakah ada preferensi user baru yang terungkap? → memory(target="user", ...)
- Apakah ada fakta environment/project yang learned? → memory(target="memory", ...)
- Apakah ada workflow yang berhasil yang worth saving as skill? → skill_manage(action="create", ...)
```

Proaktif save, jangan tunggu user minta "remember this".

### 7. Custom Style → Adapt dari SOUL.md + USER.md

Baca USER.md dan SOUL.md — ikuti style yang didefinisikan. Jangan default ke verbose/formal kalau user prefer concise/casual.

### 8. Batch/Parallel → `delegate_task`

Untuk task yang bisa dipecah ke independent subtasks:

```python
# Research 3 topik paralel
delegate_task(prompt="Research pricing trends for product A", ...)
delegate_task(prompt="Research competitor features for product B", ...)
delegate_task(prompt="Research market size for segment C", ...)

# Lalu synthesize results
```

### 9. Prompt Caching (automatic)

Hermes otomatis cache prefix system prompt (SOUL.md, tools, skills list). Lo gak perlu ngapa-ngapain. Tapi:
- Jangan `/compress` terlalu sering (breaks cache)
- Session yang long-running = cheaper per turn (prefix reused)

## Integrated Workflow (semua digabung)

Contoh task: "Analisa competitor app dari screenshot + web research, tulis strategic recommendation"

### Step 1: Understand (Vision + Decompose)
```
1. vision_analyze(screenshot) → understand current state
2. Decompose task:
   - Sub-task A: identify competitor features from screenshot
   - Sub-task B: web research competitor positioning
   - Sub-task C: gap analysis
   - Sub-task D: strategic recommendation
```

### Step 2: Research (Web Search + Citations)
```
3. web_search("competitor X features pricing 2026")
4. web_extract(competitor_url) → structured data
5. web_search("market trends [domain] 2026")
6. Compile evidence with [sources]
```

### Step 3: Analyze (Code Execution + Extended Thinking)
```
7. execute_code("""
   # Score comparison matrix
   features = {...}
   scores = calculate_weighted_scores(features)
   generate_radar_chart(scores)
""")
8. /reasoning high → deep analysis on findings
```

### Step 4: Output (Artifacts + Memory)
```
9. write_file("output/competitor-analysis.md", full_report)
10. Chat summary: TL;DR + key recommendations
11. memory(add, "User's product competes in [space], main competitors are X, Y, Z")
```

### Step 5: Verify
```
12. Self-check:
    - Setiap klaim ada citation?
    - Screenshot interpretation accurate?
    - Angka dari code execution consistent?
    - Recommendation conditional (bukan absolutist)?
```

## Mode Activation

Kalau user bilang salah satu dari ini, activate full superpowers mode:

- "kerja seperti Claude"
- "deep work"
- "full analysis"
- "gunakan semua tools"
- `/claude-superpowers`

Response pertama harus decomposition + plan, BUKAN langsung eksekusi.

## Per-Capability Quality Gates

| Capability | Quality gate |
|---|---|
| Extended Thinking | Output harus structured (bukan stream-of-consciousness) |
| Artifacts | File harus runnable/viewable tanpa edit |
| Web Search | Minimum 2 sources per factual claim |
| Vision | Acknowledge confidence level ("clearly shows X" vs "appears to be Y") |
| Code Execution | Code harus error-free (test before present) |
| Memory | Only save things useful > 1 week from now |
| Citations | URL must come from actual search results THIS turn |
| Delegation | Subtasks must be truly independent |

## What This Skill CANNOT Do (jujur)

1. **Match Opus reasoning quality** — kalau model lo DeepSeek V4 Flash, reasoning ceiling = Flash level. Skill ini optimize workflow, bukan upgrade model.
2. **Constitutional AI behavior** — Claude punya trained refusal patterns. Hermes punya SOUL.md rules, tapi enforcement depends on model compliance.
3. **Writing voice identical to Claude** — setiap model punya characteristic voice. Skill ini enforce structure, bukan mimic Claude prose.
4. **Real-time streaming artifacts** — Hermes tulis file, bukan render live preview di side panel.
5. **Sub-100ms web search** — depends on Firecrawl/Tavily latency, biasanya 2-5 detik.

## Verification (before delivering output)

1. ✅ Task decomposed sebelum eksekusi?
2. ✅ Tools yang relevan dipakai (bukan cuma text generation)?
3. ✅ Factual claims punya citation?
4. ✅ Code output tested (no errors)?
5. ✅ File artifacts valid dan accessible?
6. ✅ Memory updated kalau ada insight baru?
7. ✅ Output style match USER.md preferences?
8. ✅ Uncertainty flagged kalau ada?
