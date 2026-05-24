---
name: anti-hallucination-review
description: Checklist review anti-halu sebelum jawab. Classify evidence (verified/inferred/unknown), verify claims, remove unsupported statements. Wajib dipake untuk info current, code behavior, high-impact advice, dan automation.
version: 1.0.0
metadata:
  hermes:
    tags: [review, anti-hallucination, verification, fact-check, safety]
    category: meta
---

# Anti-Hallucination Review

Checklist yang WAJIB dijalankan sebelum menjawab pertanyaan high-stakes. Ini bukan skill yang user panggil manual — ini internal review yang agent lakuin **sebelum kirim jawaban** untuk topik sensitif.

## When to Use (AUTO-ACTIVATE)

Jalankan review ini SEBELUM jawab kalau pertanyaan involve:

- ⏰ **Current information** (versi software, harga, regulasi, berita)
- 🌐 **External websites/repos/docs** (API behavior, config, file structure)
- 💻 **Code behavior yang belum di-inspect** (jangan claim behavior tanpa baca file)
- ⚖️ **High-impact advice** (legal, financial, medical, security)
- 🤖 **Automation** yang touch accounts, credentials, browsers, atau proxies

---

## Procedure

### Step 1: List factual claims di draft jawaban

Sebelum kirim, identify SEMUA klaim faktual:

```
Claim 1: "Next.js 15 support server actions natively"
Claim 2: "File ada di src/components/Button.tsx"
Claim 3: "DeepSeek V4 Flash punya 1M context"
Claim 4: "Harga emas Antam hari ini Rp 1.850.000/gram"
```

### Step 2: Classify setiap claim

| Label | Artinya | Action |
|-------|---------|--------|
| ✅ **Verified** | Ada di tool result turn ini | OK, deliver |
| 🔍 **Inferred** | Masuk akal tapi belum dicek | Flag sebagai "kemungkinan besar, tapi cek ulang" |
| ❓ **Unknown** | Gak tau, gak ada evidence | JANGAN deliver — bilang "gak tau" atau search dulu |
| ⚠️ **Outdated-risk** | Mungkin bener dulu, tapi bisa udah berubah | Flag "info ini mungkin sudah berubah, cek sumber terkini" |

### Step 3: Verify unknown/current claims

Untuk setiap ❓ dan ⚠️:

```python
# Info current → web search
web_search("next.js 15 server actions docs")

# File/repo → inspect langsung
read_file("src/components/Button.tsx")

# API behavior → cek docs resmi
web_extract(url="https://api-docs.deepseek.com/")

# Code behavior → run atau propose test
terminal("npm run test -- --filter=Button")
```

### Step 4: Remove atau rewrite unsupported claims

- Claim ❓ yang gak bisa di-verify → **HAPUS** dari jawaban, atau ganti dengan "gw belum bisa confirm ini"
- Claim ⚠️ → **FLAG** dengan "ini mungkin sudah berubah"
- JANGAN biarin claim unverified lolos tanpa label

### Step 5: State uncertainty clearly

Kalau ada gap:
```markdown
⚠️ **Yang gw belum bisa verify**:
- [claim X] — belum dicek dari source, kemungkinan bener tapi gak guaranteed
```

### Step 6: Untuk code tasks — verify

```bash
# Run test
npm test

# Atau propose verification command
echo "Coba jalanin: npm run build && npm test"
```

Jangan bilang "ini jalan" kecuali:
- Lo udah RUN command-nya, ATAU
- Lo EXPLICITLY mark "belum ditest"

### Step 7: Untuk repo tasks — inspect dulu

```python
# SEBELUM jelasin behavior code
read_file("path/to/file.py")
# BARU jelasin
```

JANGAN jelaskan code behavior tanpa baca file. JANGAN claim file exist tanpa ls/read.

---

## Untuk Coding (additional rules)

- ❌ Jangan explain code behavior sampai lo inspect file yang relevan
- ❌ Jangan bilang command works kecuali lo udah run atau clearly mark "untested"
- ✅ Kalau lo suggest command, mark:
  ```
  🔧 Command (untested, verify di env lo):
  ```bash
  npm run build
  ```
  ```

---

## Untuk Info Terbaru (additional rules)

- ✅ Untuk latest/current info → web search atau docs resmi DULU
- ✅ Cite source atau bilang "could not verify"
- ❌ Jangan kasih info dari memori untuk topik yang cepat berubah

---

## Untuk Automation (additional rules)

Sebelum pake browser/terminal/tools:

1. **State** apa yang akan lo akses
2. **State** apakah credentials/secrets mungkin ke-touch
3. **Least privilege** — jangan akses lebih dari yang dibutuhkan
4. **Ask** sebelum destructive actions

Format:
```markdown
🤖 **Automation plan**:
- Akses: [URL/file/service]
- Credentials touched: [ya/tidak]
- Destructive: [ya/tidak]
- Proceed? [ya, langsung / minta approval dulu]
```

---

## Evidence Classification Template

Sebelum jawab pertanyaan high-stakes, isi ini (internal, gak perlu ditampilin ke user kecuali user minta):

```
📋 Evidence check:
- Source/file inspected: [list]
- Directly confirmed: [list claims]
- Inferred (likely but unverified): [list]
- Unknown (cannot confirm): [list]
```

Lalu jawab berdasarkan classification itu.

---

## Pitfalls

### Pitfall 1: Trust memory untuk package names/commands/URLs

❌ "Jalanin `npx create-next-app@latest --typescript`"
✅ `web_search("create next app official docs")` → verify command dari docs resmi

### Pitfall 2: Claim file exists tanpa inspect

❌ "File-nya ada di `src/utils/helpers.ts`"
✅ `terminal("ls src/utils/")` → confirm dulu

### Pitfall 3: Assume latest version

❌ "React 19 support use() hook"
✅ `web_search("react 19 use hook docs")` → verify dari react.dev

### Pitfall 4: Present guesses as facts

❌ "API ini return JSON dengan field 'data' dan 'meta'"
✅ "Gw belum inspect API response-nya. Mau gw test dulu?"

### Pitfall 5: Skip review karena "udah yakin"

Justru saat lo paling yakin = saat lo paling perlu review. Overconfidence = source of hallucination.

---

## Verification (meta — review the review)

Jawaban final LULUS kalau:

1. ✅ Setiap major factual claim punya: source, file evidence, test result, ATAU uncertainty label
2. ✅ Gak ada invented paths, options, APIs, dates, atau repo behavior
3. ✅ Uncertainty di-state explicitly (bukan hidden)
4. ✅ Code claims di-back oleh file inspection atau marked "untested"
5. ✅ Current info di-verify via tool atau flagged "may have changed"
