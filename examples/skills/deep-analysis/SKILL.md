---
name: deep-analysis
description: Analisis mendalam multi-perspektif. Decompose masalah, identifikasi asumsi tersembunyi, evaluasi trade-off, rekomendasi conditional.
version: 2.0.0
metadata:
  hermes:
    tags: [analysis, reasoning, planning, strategy, decision]
    category: research
---

# Deep Analysis

## KAPAN PAKAI

```
IF pertanyaan yang jawaban cepetnya misleading → PAKAI
IF keputusan arsitektur besar → PAKAI
IF evaluasi proposal / RFC → PAKAI
IF root cause analysis → PAKAI
IF strategy / roadmap planning → PAKAI
IF quick Q&A faktual → JANGAN (pakai research-citation)
IF task eksekusi langsung → JANGAN (langsung kerjain)
IF bug fix yang jelas → JANGAN (langsung fix)
```

---

## PROCEDURE (ikuti exact, jangan skip step)

### Step 1: Decompose pertanyaan jadi sub-questions

```
TULIS:
- Pertanyaan utama: "[apa yang user tanya]"
- Sub-question 1: [apa yang HARUS dijawab dulu]
- Sub-question 2: [apa lagi]
- Sub-question 3: [apa lagi]
```

### Step 2: Gather evidence (bukan opini)

```
IF ada codebase → read_file() untuk data coupling, LOC, structure
IF butuh data market → web_search()
IF ada past conversation → session_search()
IF ada data yang cuma user tau → TANYA user
DO NOT: jawab sub-questions dari opini/memori kalau bisa verify
```

### Step 3: Identifikasi hidden assumptions

```
UNTUK setiap proposal/ide yang dianalisis:
  LIST asumsi yang GAK diucapkan tapi harus benar supaya ide ini jalan
  LABEL setiap asumsi:
    IF validated → ✅
    IF belum dicek → ❓
    IF likely wrong → ❌
```

### Step 4: Multi-perspective evaluation

```
EVALUATE dari MINIMAL 3 sudut pandang:
- Engineering: complexity, maintenance, debugging
- Business: time to market, cost, scaling
- Operations: monitoring, incident response, deploy
- Risk: apa yang bisa catastrophically wrong?
```

### Step 5: Trade-off matrix

```
BUAT TABEL:
| Opsi | Pros | Cons | Risk | Cost |
```

### Step 6: Conditional recommendation

```
FORMAT:
IF [kondisi A] → rekomendasi X
IF [kondisi B] → rekomendasi Y
IF [kondisi C] → rekomendasi Z

DO NOT: kasih single recommendation tanpa kondisi
DO NOT: bilang "X lebih baik" tanpa context-dependent qualifier
```

### Step 7: Pre-mortem

```
TULIS: "Kalau rekomendasi ini SALAH, apa yang terjadi?"
- Skenario 1: [worst case]
- Skenario 2: [likely failure mode]
```

### Step 8: Actionable next steps

```
TULIS concrete steps dengan timeline:
1. [Week 1] [action spesifik]
2. [Week 2] [action spesifik]
3. [Week 3] [decision point berdasarkan data dari 1-2]
```

---

## OUTPUT TEMPLATE

```markdown
# Deep Analysis: [Judul]

## TL;DR (max 3 kalimat, conditional)
[Kesimpulan. HARUS ada "IF...THEN" di dalamnya.]

## Decomposition
- Pertanyaan utama: [X]
- Sub-questions:
  1. [sub-q 1]
  2. [sub-q 2]
  3. [sub-q 3]

## Evidence
| Sub-question | Evidence | Source | Confidence |
|---|---|---|---|
| [sq1] | [data] | [file/url/user] | ✅/⚠️/❓ |

## Hidden Assumptions
1. "[asumsi 1]" — ❓ belum validated
2. "[asumsi 2]" — ✅ confirmed dari [source]
3. "[asumsi 3]" — ❌ likely wrong karena [reason]

## Analysis per Perspective
| Perspective | Assessment | Key concern |
|---|---|---|
| Engineering | [1 kalimat] | [biggest risk] |
| Business | [1 kalimat] | [biggest risk] |
| Operations | [1 kalimat] | [biggest risk] |

## Trade-off Matrix
| Opsi | Pros | Cons | Risk | Cost | Timeline |
|---|---|---|---|---|---|
| A | [list] | [list] | Low/Med/High | $/$$/$$$  | [weeks] |
| B | [list] | [list] | Low/Med/High | $/$$/$$$  | [weeks] |
| C | [list] | [list] | Low/Med/High | $/$$/$$$  | [weeks] |

## Recommendation (conditional)
IF [kondisi] → Opsi [X] karena [alasan]
IF [kondisi lain] → Opsi [Y] karena [alasan]
REGARDLESS: [hal yang harus dilakuin apapun pilihannya]

## Pre-mortem
IF rekomendasi salah:
- Skenario 1: [apa yang terjadi]
- Skenario 2: [apa yang terjadi]
- Mitigasi: [apa yang bisa dilakuin sekarang untuk reduce risk]

## Next Steps
1. [Week 1] [concrete action]
2. [Week 1] [concrete action]
3. [Week 2] [decision meeting dengan data dari step 1-2]

## Confidence Level
[High / Medium / Low] — karena [alasan]

## What I Don't Know
- [gap 1 yang kalau diisi bisa mengubah rekomendasi]
- [gap 2]
```

---

## CONTOH OUTPUT (ringkas)

```markdown
# Deep Analysis: Migrasi Monolith ke Microservices

## TL;DR
IF team < 10 engineer DAN pain point utama = CI bottleneck → modularize monolith, jangan migrasi.
IF team 15+ DAN ada 2+ domain genuinely independent → extract 2-3 service, keep sisanya monolith.

## Decomposition
- Pertanyaan utama: "Haruskah migrasi ke microservices?"
- Sub-questions:
  1. Apa pain point AKTUAL dari monolith saat ini?
  2. Berapa engineer di team?
  3. Apakah ada domain yang benar-benar independent?

## Trade-off Matrix
| Opsi | Pros | Cons | Risk | Cost |
|---|---|---|---|---|
| A: Modularize monolith | Low risk, 2 minggu | Scaling ceiling tetap | Low | $ |
| B: Extract 2-3 service | Unblock deploys | Coordination overhead | Med | $$ |
| C: Full microservices | Max flexibility | 12+ bulan, high complexity | High | $$$$ |

## Recommendation (conditional)
IF team < 10 → Opsi A
IF team 10-30 + domain boundaries clear → Opsi B
IF team 30+ + budget OK + timeline 12-18 bulan → Opsi C
REGARDLESS: fix observability dulu (logging, tracing, metrics)

## Next Steps
1. [Week 1] Audit coupling: dependency analysis tool
2. [Week 1] Survey team: apa yang paling blocking?
3. [Week 2] Spike: extract 1 domain kecil, ukur effort real
4. [Week 3] Decision meeting dengan data
```

---

## VERIFICATION

```
□ Rekomendasi conditional (bukan absolutist)?
□ Ada perspektif yang gw miss?
□ Gw nge-claim fakta yang gw gak tau?
□ Next steps actionable (bukan vague)?
□ Pre-mortem realistic?
□ Gw bias ke satu opsi tanpa acknowledge trade-off?

IF ada □ TIDAK → fix sebelum kirim
```
