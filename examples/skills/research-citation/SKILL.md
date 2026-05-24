---
name: research-citation
description: Cari jawaban faktual dengan sumber yang bisa dicek. Wajib citation per klaim. Kalau gak ketemu → bilang "tidak ditemukan".
version: 2.0.0
metadata:
  hermes:
    tags: [research, fact-check, citation, anti-hallucination]
    category: research
    requires_toolsets: [web]
---

# Research with Citation

## KAPAN PAKAI

```
IF user tanya fakta (kapan, berapa, siapa, di mana) → PAKAI
IF user tanya regulasi / versi software / leadership → PAKAI
IF user minta data statistik / ranking / salary → PAKAI
IF user minta kutipan orang publik → PAKAI
IF user minta opini / brainstorm / coding → JANGAN PAKAI
```

---

## PROCEDURE (ikuti exact)

### Step 1: Search minimal 2 query berbeda

```
DO: web_search("[query bahasa user]")
DO: web_search("[query English / lebih spesifik]")
OPTIONAL: web_search("[query] site:official-domain")
```

### Step 2: Rank sumber by trust

```
TIER 1 (paling kuat): Sumber primer / official (api-docs, bps.go.id, who.int, github.com/owner/repo)
TIER 2: Dokumentasi resmi vendor (nodejs.org, react.dev)
TIER 3: Berita kredibel (reuters.com, kompas.com, wikipedia)
TIER 4 (lemah): Blog post, Medium, Stack Overflow

SKIP: Blog SEO "top 10 best", forum tanpa moderasi, halaman clickbait
```

### Step 3: Extract per klaim + tag confidence

```
IF klaim dari TIER 1 → tag: ✅ Confirmed
IF klaim dari TIER 2-3 → tag: ⚠️ Likely (cek ulang kalau krusial)
IF klaim gak ada sumber kuat → tag: ❓ Unverified
```

### Step 4: Handle konflik antar sumber

```
IF sumber A bilang X, sumber B bilang Y:
  → sebut keduanya
  → identifikasi mana yang lebih primary
  DO NOT: paksa jadi satu jawaban
```

### Step 5: Format output

---

## OUTPUT TEMPLATE

```markdown
## Jawaban singkat
[1-2 kalimat, jawab langsung]

## Detail dengan sumber

✅ [Klaim 1]. ([URL sumber])

⚠️ [Klaim 2 — likely tapi cek ulang]. ([URL sumber])

❓ [Klaim 3 — gak ada sumber kuat]. (tidak ditemukan)

## Yang TIDAK ditemukan
- [aspek yang user tanya tapi gak ada sumber]

## Catatan
- Tanggal akses: [date]
- [caveat lain]
```

---

## CONTOH OUTPUT YANG BENAR

```markdown
## Jawaban singkat
DeepSeek V3 memiliki context window 128K token berdasarkan dokumentasi resmi.

## Detail dengan sumber

✅ DeepSeek V3 context window: 128K token. (https://platform.deepseek.com/docs)

⚠️ DeepSeek V4 Flash dilaporkan 1M token oleh listing OpenRouter, belum dikonfirmasi docs resmi. (https://openrouter.ai/deepseek/deepseek-v4-flash)

## Yang TIDAK ditemukan
- Benchmark resmi DeepSeek V4 Flash vs V3 untuk coding tasks

## Catatan
- Tanggal akses: 24 Mei 2026
- Info model AI berubah cepat, verify ke docs resmi sebelum production decision
```

---

## CONTOH OUTPUT YANG SALAH (jangan kayak gini)

```
DeepSeek V4 Flash punya context window 1 juta token. Modelnya dirilis 2026
dengan arsitektur MoE dan menjadi salah satu model paling efisien.
```
❌ Tidak ada source. Klaim spesifik tanpa URL. "Salah satu paling efisien" tanpa benchmark.

---

## DECISION TREE: URL Handling

```
IF URL muncul di hasil web_search turn ini → boleh kutip
IF URL gak muncul di tool result → DO NOT fabricate URL
IF URL terlihat "terlalu rapi" (misal bps.go.id/data-exact-2025) → curigai, verify

DO NOT: invent URLs
DO NOT: guess URL patterns
```

## DECISION TREE: Angka & Statistik

```
IF angka ada di tool result → kutip + URL
IF angka gak ada di tool result → bilang "tidak ditemukan"
IF angka dari sumber sekunder (blog ngutip BPS) → coba buka BPS langsung
IF BPS gak bisa diakses → bilang "dari sumber sekunder [URL], belum verify primer"
```

---

## VERIFICATION

```
□ Setiap angka spesifik punya URL backing?
□ Setiap kutipan orang nyata punya URL backing?
□ Ada section "Yang TIDAK ditemukan" yang jujur?
□ Gak ada URL yang gw fabricate?

IF ada □ TIDAK → redo search atau drop klaim
```
