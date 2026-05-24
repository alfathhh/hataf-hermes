---
name: claude-superpowers
description: Mode full-power — extended thinking, artifacts, citations, vision, code execution, memory dalam satu alur. Panggil untuk task kompleks multi-capability.
version: 2.0.0
metadata:
  hermes:
    tags: [reasoning, artifacts, citations, vision, code-execution, multi-capability]
    category: meta
---

# Claude Superpowers (Hermes Edition)

## KAPAN AKTIF

```
IF user bilang "kerja seperti Claude" OR "deep work" OR "full analysis" OR "full power" → AKTIF
IF task butuh KOMBINASI tools (vision + search + code + write) → AKTIF
IF task sederhana (translate, format, quick Q&A) → JANGAN (overkill)
```

---

## PROCEDURE (ikuti exact, step by step)

### Step 1: Decompose task

```
SEBELUM execute apa-apa:
1. TULIS sub-tasks yang harus dikerjain
2. IDENTIFIKASI tools yang dibutuhkan per sub-task
3. PLAN urutan execution

DO NOT: langsung execute tanpa decompose
```

### Step 2: Execute per capability

```
IF butuh understand image/screenshot:
  → vision_analyze(image, prompt="Describe in detail: layout, components, text, issues")

IF butuh factual information:
  → web_search("[query]") + web_extract(url)
  → CITE per klaim: [sumber](URL)

IF butuh data processing / math / chart:
  → execute_code(python_code)
  → Pastikan code error-free sebelum present

IF butuh output >20 baris code/doc:
  → write_file(path, content)
  → Chat: kasih summary + path file

IF butuh parallel research:
  → delegate_task per sub-topic (independent)
  → Synthesize results

IF butuh reasoning mendalam:
  → /reasoning high
  → Decompose structured (bukan stream-of-consciousness)
```

### Step 3: Combine results

```
1. Gather semua sub-task results
2. Check consistency (gak kontradiksi antar sub-task)
3. Synthesize into unified output
4. Verify: setiap klaim punya source/evidence
```

### Step 4: Memory check

```
SETELAH task selesai:
IF ada preferensi user baru → memory(target="user", ...)
IF ada fakta environment baru → memory(target="memory", ...)
IF ada workflow baru yang recurring → propose skill baru
```

---

## CAPABILITY MAP (quick reference)

| Capability | Hermes equivalent | Tool |
|---|---|---|
| Extended Thinking | /reasoning high + decomposition | structured analysis |
| Artifacts | write_file() | save to path |
| Web Search + Citation | web_search + web_extract | cite per klaim |
| Vision | vision_analyze | image/screenshot |
| Code Execution | execute_code / terminal | Python sandbox |
| Memory | memory() | persistent |
| Batch/Parallel | delegate_task | independent subtasks |

---

## OUTPUT TEMPLATE: Full Analysis

```markdown
# [Task Title]

## Plan
1. [Sub-task A] — tool: [X]
2. [Sub-task B] — tool: [Y]
3. [Sub-task C] — tool: [Z]

## Results

### [Sub-task A]
[result + source]

### [Sub-task B]
[result + source]

### [Sub-task C]
[result + source]

## Synthesis
[Combined insight, 3-5 kalimat]

## Files Created
- `path/file.md` — [description]

## What I Learned (for memory)
- [fact/preference to remember]
```

---

## CONTOH OUTPUT

```markdown
# Competitor Analysis: Coffee App

## Plan
1. Vision: analyze screenshot competitor app
2. Web: research competitor positioning + pricing
3. Code: score comparison matrix + radar chart
4. Write: full report to file

## Results

### 1. Screenshot Analysis (vision)
- Competitor uses tab navigation (Home, Menu, Rewards, Profile)
- Color scheme: dark green + cream (Starbucks-like)
- Key feature: AR cup customizer prominent on home
- Missing: no dark mode, no accessibility indicators

### 2. Web Research
- Competitor X: 500K MAU, avg order Rp 45K [source](https://...)
- Market growing 15% YoY in Indonesia [source](https://...)
- Main pain point users report: slow delivery tracking

### 3. Comparison Matrix
📁 Chart saved: output/competitor-radar.png
- Our app scores higher on: delivery speed, price
- Competitor scores higher on: UX polish, rewards program

## Synthesis
Competitor's main advantage is UX polish and rewards gamification.
Our advantage: delivery speed + lower price point.
Quick wins: add rewards program (3 week sprint), improve onboarding flow.

## Files Created
- `output/competitor-analysis.md` — full 2-page report
- `output/competitor-radar.png` — radar chart comparison

## Memory Updated
- User's product competes in coffee delivery space, Jakarta market
- Main competitors: X, Y, Z
```

---

## QUALITY GATES (per capability)

```
Extended Thinking: output HARUS structured (bukan stream)
Artifacts/Files: HARUS runnable/viewable tanpa edit
Web Search: MINIMUM 2 sources per factual claim
Vision: acknowledge confidence ("clearly shows X" vs "appears to be Y")
Code Execution: HARUS error-free (test before present)
Memory: ONLY save things useful > 1 week
Citations: URL HARUS dari search results turn ini
```

---

## DECISION TREE: When to Use Which Tool

```
IF need current info → web_search + web_extract
IF need understand image → vision_analyze
IF need calculate/chart → execute_code (Python)
IF need save output → write_file
IF need parallel work → delegate_task
IF need deep reasoning → /reasoning high
IF need verify code → terminal (run test)
```

---

## LIMITATIONS (jujur)

```
1. Reasoning quality = limited by model (Flash ≠ Opus level)
2. Web search = 2-5 detik latency
3. File output = no live preview (user buka file sendiri)
4. Vision = auxiliary model, bisa miss detail kecil
5. Memory = limited char (2200 char max)
```

---

## VERIFICATION

```
□ Task decomposed sebelum execute?
□ Tools yang relevan dipakai (bukan cuma text generation)?
□ Factual claims punya citation?
□ Code output tested (no errors)?
□ File artifacts valid?
□ Uncertainty flagged?

IF ada □ TIDAK → fix before deliver
```
