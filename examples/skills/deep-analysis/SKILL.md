---
name: deep-analysis
description: Analisis mendalam multi-perspektif seperti senior researcher. Decompose masalah, identifikasi asumsi tersembunyi, evaluasi trade-off, berikan rekomendasi berlapis. Setara dengan mode "extended thinking" — tapi pakai tool calls bukan internal reasoning.
version: 1.0.0
metadata:
  hermes:
    tags: [analysis, reasoning, planning, strategy, decision]
    category: research
---

# Deep Analysis

Skill ini dipanggil untuk task yang butuh **analisis mendalam** — bukan jawaban cepat, tapi breakdown multi-layer. Equivalent dengan kemampuan extended thinking / chain-of-thought yang kuat.

## When to Use

- Pertanyaan yang jawaban cepetnya misleading ("haruskah kita migrasi ke microservices?")
- Keputusan arsitektur besar
- Evaluasi proposal / RFC
- Root cause analysis (kenapa X gagal?)
- Strategy / roadmap planning
- Risk assessment
- Code architecture review berskala besar (>1000 LOC)

JANGAN pakai untuk:
- Quick Q&A faktual (pakai `research-citation`)
- Task eksekusi langsung ("deploy ini")
- Bug fix yang jelas (langsung fix aja)

## Procedure

### 1. Decompose pertanyaan

Sebelum menjawab, pecah pertanyaan jadi sub-questions:

```markdown
## Decomposition

Pertanyaan utama: "Haruskah kita migrasi dari monolith ke microservices?"

Sub-questions yang harus dijawab SEBELUM main question:
1. Apa pain points aktual dari monolith saat ini? (bukan theoretical)
2. Berapa engineer di team? (microservices butuh N+ engineers per service)
3. Apa deployment frequency saat ini? Apa yang blocking it?
4. Apakah ada domain yang jelas-jelas independent?
5. Berapa budget infra setelah migrasi vs sekarang?
6. Timeline? (microservices migration = 6-18 bulan minimum)
```

### 2. Gather evidence (bukan opini)

Untuk setiap sub-question, cari evidence:
- `read_file` codebase (lines of code, coupling, shared DB usage)
- `web_search` untuk data points yang relevan (industry benchmarks)
- `session_search` untuk past conversations tentang topik ini
- Tanya user kalau ada data yang cuma dia tau

### 3. Identifikasi asumsi tersembunyi

Setiap proposal punya asumsi yang jarang diucapkan:

```markdown
## Hidden assumptions

1. "Kita punya cukup engineer untuk maintain N services" — ❓ belum validated
2. "Deployment frequency akan naik setelah migrasi" — ⚠️ depends on CI/CD maturity
3. "Services akan benar-benar independent" — ❓ shared DB = monolith-in-disguise
```

### 4. Multi-perspective evaluation

Evaluasi dari minimal 3 angle:

| Perspektif | Pertimbangan |
|---|---|
| **Engineering** | Complexity, maintenance, debugging difficulty |
| **Business** | Time to market, cost, team scaling |
| **Operations** | Monitoring, incident response, deployment |
| **Risk** | What could go catastrophically wrong? |

### 5. Trade-off matrix (bukan single recommendation)

```markdown
## Trade-off Matrix

| Opsi | Pros | Cons | Risk | Cost |
|------|------|------|------|------|
| A: Stay monolith + modularize | Low risk, cheap | Scaling ceiling | Low | $ |
| B: Extract 2-3 services | Unblock deploys | Coordination overhead | Medium | $$ |
| C: Full microservices | Max flexibility | High complexity, 12+ months | High | $$$$ |
```

### 6. Conditional recommendation

```markdown
## Recommendation (conditional)

**IF** team < 10 engineers AND deployment pain is mainly CI bottleneck:
→ Opsi A (modularize monolith, fix CI pipeline)

**IF** team 10-30 AND ada 2+ domains yang genuinely independent:
→ Opsi B (extract those domains, keep rest monolith)

**IF** team 30+ AND domain boundaries clear AND budget allows 12-18 month investment:
→ Opsi C (but start with 2-3 services first, not big-bang)

**Regardless**: fix observability first (logging, tracing, metrics). Tanpa itu, semua opsi akan painful.
```

### 7. What could go wrong (pre-mortem)

```markdown
## Pre-mortem: kalau rekomendasi ini salah

Skenario 1: kita underestimate coupling → 6 bulan in, shared DB masih jadi bottleneck
Skenario 2: team turnover → knowledge tersebar, incident response lambat
Skenario 3: premature optimization → kita habis waktu re-architect tapi product stagnate
```

### 8. Next steps (actionable)

```markdown
## Concrete next steps (urutan prioritas)

1. [Week 1] Audit coupling: jalankan dependency analysis tool pada codebase
2. [Week 1] Survey team: apa yang paling blocking mereka hari ini?
3. [Week 2] Spike: coba extract 1 domain kecil, ukur effort real
4. [Week 3] Decision meeting dengan data dari step 1-3
```

## Anti-patterns yang harus dihindari

1. **Single recommendation tanpa conditional** — realita selalu depends on context
2. **Opini disajikan sebagai fakta** — "microservices lebih baik" bukan fakta, itu tergantung
3. **Ignoring cost** — setiap pilihan punya cost (waktu, uang, complexity), sebutin
4. **Analysis paralysis** — setelah decompose, tetap kasih recommendation. Jangan cuma list pro/con tanpa kesimpulan
5. **Halu benchmark** — kalau lo sebut "Netflix berhasil dengan microservices karena X", pastikan itu dari sumber yang bisa dicek, bukan folklore
6. **Overconfidence** — kalau analisis lo based on incomplete info, bilang. "Dengan data yang ada, rekomendasi saya adalah X. Tapi ada gap di Y yang bisa mengubah kesimpulan."

## Format output lengkap

```markdown
# Deep Analysis: [Judul]

## TL;DR (max 3 kalimat)
[Kesimpulan paling penting. Conditional.]

## Context & constraints
[Apa yang udah diketahui. Data yang dipakai.]

## Decomposition
[Sub-questions]

## Evidence gathered
[Fakta + sumber]

## Hidden assumptions
[Yang biasanya gak disebut]

## Analysis per perspective
[Engineering / Business / Ops / Risk]

## Trade-off matrix
[Tabel opsi]

## Recommendation (conditional)
[IF...THEN...ELSE...]

## Pre-mortem
[Kalau salah, apa yang terjadi]

## Next steps
[Actionable items]

## Confidence level
[High / Medium / Low + alasan]

## What I don't know
[Gaps yang kalau diisi bisa mengubah rekomendasi]
```

## Verification

1. Apakah rekomendasi gw conditional (bukan absolutist)?
2. Apakah ada perspektif yang gw miss?
3. Apakah gw nge-claim fakta yang sebenarnya gw gak tau?
4. Apakah next steps benar-benar actionable (bukan vague "think about it")?
5. Apakah pre-mortem realistic?
6. Apakah gw bias ke satu opsi tanpa acknowledge trade-off?
