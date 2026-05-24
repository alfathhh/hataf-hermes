---
name: anti-hallucination-review
description: Checklist review anti-halu sebelum jawab. Classify evidence (verified/inferred/unknown), verify claims, remove unsupported statements.
version: 2.0.0
metadata:
  hermes:
    tags: [review, anti-hallucination, verification, fact-check, safety]
    category: meta
---

# Anti-Hallucination Review

## KAPAN AKTIF (auto-trigger)

```
IF pertanyaan involve salah satu:
  - Info current (versi software, harga, regulasi, berita) → AKTIF
  - External websites/repos/docs → AKTIF
  - Code behavior yang belum di-inspect → AKTIF
  - High-impact advice (legal, financial, medical, security) → AKTIF
  - Automation yang touch accounts/credentials → AKTIF
ELSE:
  - Skip review ini
```

---

## PROCEDURE (ikuti urutan, JANGAN skip)

### Step 1: List semua klaim faktual di draft jawaban

Tulis setiap klaim yang mau lo deliver:

```
Claim 1: "[tulis klaim]"
Claim 2: "[tulis klaim]"
Claim 3: "[tulis klaim]"
```

### Step 2: Label setiap klaim

```
IF klaim ada di tool result TURN INI → label: ✅ VERIFIED
IF klaim masuk akal tapi belum dicek → label: 🔍 INFERRED
IF klaim gak ada evidence sama sekali → label: ❓ UNKNOWN
IF klaim mungkin bener dulu tapi bisa berubah → label: ⚠️ OUTDATED-RISK
```

### Step 3: Action per label

```
IF ✅ VERIFIED → deliver as-is
IF 🔍 INFERRED → tambah "kemungkinan besar, tapi cek ulang"
IF ❓ UNKNOWN → DO NOT deliver. Pilih salah satu:
  - web_search untuk verify
  - read_file untuk verify
  - bilang "gw gak tau"
IF ⚠️ OUTDATED-RISK → tambah "info ini mungkin sudah berubah, cek sumber terkini"
```

### Step 4: Verify unknown claims

```
IF info current → web_search("[topic] [year]")
IF file/repo → read_file("path/to/file")
IF API behavior → web_extract(url="docs URL")
IF code behavior → terminal("test command")
IF SEMUA gagal → bilang "tidak ditemukan"
```

### Step 5: Hapus atau rewrite klaim yang gagal verify

```
IF klaim ❓ gak bisa di-verify → HAPUS dari jawaban
IF klaim ⚠️ → FLAG dengan "mungkin sudah berubah"
DO NOT biarkan klaim unverified lolos tanpa label
```

### Step 6: State uncertainty

```
⚠️ **Yang gw belum bisa verify**:
- [claim X] — belum dicek dari source
```

---

## DECISION TREE: Code Tasks

```
IF user tanya behavior code:
  1. read_file(path) DULU
  2. BARU jelasin behavior
  DO NOT jelaskan code tanpa baca file

IF user minta run command:
  IF sudah di-run → bilang "tested, jalan"
  IF belum di-run → tulis: "🔧 Command (untested, verify di env lo)"

IF user tanya "file ini ada?":
  1. terminal("ls path/to/file")
  2. BARU jawab ada/tidak
  DO NOT claim file exist tanpa ls/read
```

## DECISION TREE: Info Current

```
IF pertanyaan tentang versi/harga/regulasi/berita:
  1. web_search DULU
  2. Cite source dari result
  3. BARU jawab
  DO NOT jawab dari memori untuk topik time-sensitive

IF web_search return nothing:
  → bilang "tidak ditemukan di pencarian"
  DO NOT fabricate answer
```

## DECISION TREE: Automation

```
SEBELUM pake browser/terminal/tools:
  1. STATE apa yang akan diakses
  2. STATE apakah credentials ke-touch
  3. IF destructive → minta approval dulu
  4. IF non-destructive → langsung execute
```

---

## CONTOH OUTPUT: Evidence Check

```
📋 Evidence check:
- Source inspected: logammulia.com (web_extract)
- Directly confirmed: harga jual 1g = Rp 1.850.000
- Inferred: harga buyback ~Rp 1.750.000 (terlihat di page tapi format unclear)
- Unknown: harga member (gak terlihat di page publik)
```

---

## CONTOH OUTPUT: Jawaban dengan Uncertainty

```
## Harga Emas Antam — 24 Mei 2026

✅ Harga jual 1g: Rp 1.850.000 (dari logammulia.com)
✅ Harga buyback: Rp 1.750.000 (dari logammulia.com)
⚠️ Harga member: tidak tersedia di halaman publik

🔗 Sumber: https://www.logammulia.com/id/harga-emas-702
```

---

## VERIFICATION CHECKLIST (sebelum kirim)

```
□ Setiap klaim faktual punya: source / evidence / uncertainty label?
□ Gak ada invented paths, APIs, dates, repo behavior?
□ Uncertainty di-state explicitly?
□ Code claims di-back oleh file inspection?
□ Current info di-verify via tool?

IF ada satu □ yang TIDAK → FIX sebelum kirim
```
