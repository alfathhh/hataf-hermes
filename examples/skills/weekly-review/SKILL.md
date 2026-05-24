---
name: weekly-review
description: Rutinitas mingguan self-improvement agent. Review 20 sesi terakhir, identifikasi pattern error, update memory/skill/context. Jalanin via cronjob atau manual /weekly-review.
version: 1.0.0
metadata:
  hermes:
    tags: [meta, self-improvement, review, memory, skill-management, maintenance]
    category: meta
---

# Weekly Self-Improvement Review

Ritual mingguan biar agent beneran berkembang — bukan cuma numpuk session tanpa belajar. Jalanin seminggu sekali (manual atau via cronjob).

## When to Use

- Seminggu sekali (Jumat/Minggu sore)
- Setelah sprint intensif (banyak session dalam waktu singkat)
- Kalau user merasa agent "gak belajar-belajar" dari kesalahan
- Manual: `/weekly-review`

## Cronjob setup

```
/cron add "0 20 * * 0" "Jalanin weekly self-improvement review. Load skill weekly-review dan ikuti prosedurnya lengkap. Kirim hasil review ke Telegram." --skill weekly-review --name "Weekly Self-Review"
```

---

## Procedure

### Phase 1: GATHER (kumpulin data)

```python
# Search 20 sesi terakhir
session_search(query="error OR mistake OR correction OR retry OR failed")
session_search(query="remember OR ingat OR save OR memory")
```

Cari:
- Momen agent salah dan dikoreksi user
- Momen agent harus retry/debug
- Momen user bilang "ingat ini" atau koreksi approach
- Momen agent pake tool yang salah atau approach inefficient
- Pattern pertanyaan yang sering muncul

---

### Phase 2: ANALYZE (identifikasi pattern)

Isi template ini:

```markdown
## 📊 Weekly Review — [Tanggal]

### 1. 🔴 Repeated Mistakes (error yang muncul >1x)

| # | Mistake pattern | Frequency | Impact | Root cause |
|---|---|---|---|---|
| 1 | [misal: salah source harga emas] | 3x | High | web_search default instead of web_extract |
| 2 | [misal: halu package name] | 2x | Medium | jawab dari memory tanpa verify |
| 3 | ... | ... | ... | ... |

### 2. 💾 Missing Memories (hal yang seharusnya di-remember tapi belum)

| # | Info yang harusnya di-save | Target | Alasan |
|---|---|---|---|
| 1 | [misal: user prefer Tailwind bukan Bootstrap] | USER.md | Muncul 3x di session, belum di-save |
| 2 | [misal: server user di sgp1 region] | MEMORY.md | Penting untuk devops tasks |
| 3 | ... | ... | ... |

### 3. 🛠️ Workflows that should become skills

| # | Workflow | Frequency | Current state | Proposed skill |
|---|---|---|---|---|
| 1 | [misal: deploy ke Cloudflare Pages] | 4x manual prompt | Gak ada skill | `cloudflare-deploy` |
| 2 | [misal: format data BPS jadi chart] | 3x | Gak ada | `bps-data-viz` |
| 3 | ... | ... | ... | ... |

### 4. 📦 Outdated Skills (skill yang perlu di-update)

| # | Skill | Issue | Fix needed |
|---|---|---|---|
| 1 | [misal: web-scrape] | Shopee endpoint berubah | Update API URL |
| 2 | [misal: harga-emas] | logammulia.com redesign HTML | Update extraction prompt |
| 3 | ... | ... | ... |

### 5. ⚠️ Risky Assumptions (asumsi yang gw bikin dan belum di-verify)

| # | Assumption | Risk level | How to verify |
|---|---|---|---|
| 1 | [misal: Kimi K2.6 support thinking mode] | Medium | Test /reasoning high |
| 2 | [misal: RTK plugin auto-activate] | Low | Check hermes plugins list |
| 3 | ... | ... | ... |
```

---

### Phase 3: PROPOSE (rencana update)

```markdown
## 📋 Proposed Updates

### Memory updates (safe, auto-apply):
1. ADD to MEMORY.md: "[fact]"
2. REPLACE in USER.md: "[old]" → "[new]"
3. REMOVE from MEMORY.md: "[stale fact]"

### Skill updates (auto-apply kalau non-destructive):
1. PATCH skill `[name]`: [what to change]
2. CREATE skill `[name]`: [what it does]
3. ARCHIVE skill `[name]`: [why outdated]

### Context file changes (ASK user dulu):
1. Update AGENTS.md: [what to add/change]
2. Update SOUL.md: [what to add/change]

### New verification rules (add to anti-hallucination-review):
1. "[new rule based on repeated mistake]"
```

---

### Phase 4: EXECUTE (apply updates)

**Auto-apply (safe):**
- Memory add/replace/remove → `memory(action="add/replace/remove", ...)`
- Skill patch → `skill_manage(action="patch", ...)`

**Ask first (potentially destructive):**
- Context file changes (SOUL.md, AGENTS.md)
- Skill delete/archive
- Memory removal yang mungkin masih relevan

Format:

```markdown
## ✅ Auto-applied:
- [list updates yang udah dijalanin]

## ❓ Needs approval:
1. [Update yang butuh confirm dari user]
   - Alasan: [kenapa ini risky]
   - Proceed? [yes/no]
```

---

### Phase 5: REPORT (kirim summary ke user)

Output final ke Telegram/chat:

```markdown
## 📊 Weekly Review — [Tanggal]

### Summary
- 🔴 Repeated mistakes found: [N]
- 💾 Memory gaps filled: [N]
- 🛠️ New skills proposed: [N]
- 📦 Skills updated: [N]
- ⚠️ Risky assumptions flagged: [N]

### Auto-applied ✅
- [list]

### Needs your decision ❓
- [list with options]

### Key insight minggu ini 💡
[1-2 kalimat: apa yang paling penting yang agent "pelajari" minggu ini]

---

🔗 **Sumber**: session_search 20 sesi terakhir
🤖 **Model**: [model] | reasoning: high
```

---

## Scoring: apakah agent berkembang?

Track progress across weeks:

```markdown
## Progress tracker (update tiap minggu)

| Minggu | Repeated mistakes | Memory gaps | New skills | Improvement trend |
|--------|-------------------|-------------|------------|-------------------|
| W1 | 5 | 3 | 2 | baseline |
| W2 | 3 | 1 | 1 | ↗️ improving |
| W3 | 2 | 0 | 0 | ↗️ stabilizing |
| W4 | 1 | 1 | 1 | ✅ mature |
```

Target: repeated mistakes trend ke 0 dalam 4-6 minggu. Kalau stuck → ada systemic issue yang perlu di-address.

---

## Pitfalls

### Pitfall 1: Review tanpa action

Review yang cuma "list masalah" tapi gak update memory/skill = sia-sia. SELALU ada Phase 4 (execute).

### Pitfall 2: Over-aggressive memory cleanup

Jangan hapus memory yang "belum dipake baru-baru ini" — mungkin masih relevan nanti. Hapus cuma yang CONFIRMED outdated.

### Pitfall 3: Bikin skill untuk everything

Gak semua workflow perlu jadi skill. Threshold: **muncul ≥3x dalam 2 minggu** baru worth jadi skill. Kalau cuma 1-2x → mungkin cukup di memory.

### Pitfall 4: Skip risky assumptions

Jangan ignore "risky assumptions" section — ini yang bikin agent halu di masa depan. Verify atau flag.

### Pitfall 5: Review jadi terlalu panjang

Max 20 sesi. Jangan review 100 sesi sekaligus — too noisy. Fokus ke recent patterns.

---

## Verification

Review LULUS kalau:

1. ✅ Semua 5 analysis sections terisi (atau explicitly "gak ada")
2. ✅ Ada minimal 1 actionable update (memory/skill/context)
3. ✅ Risky assumptions di-flag atau di-verify
4. ✅ Auto-applied updates gak destructive
5. ✅ User-approval items clearly marked
6. ✅ Summary terkirim ke user (Telegram/chat)

---

## Advanced: Cross-week pattern detection

Setelah 4+ minggu review, cari meta-patterns:

```
session_search(query="weekly review")
```

- Mistake yang MASIH muncul setelah 3 minggu → systemic issue, perlu rule di SOUL.md
- Skill yang di-patch >3x → mungkin approach-nya fundamentally wrong, rewrite
- Memory yang sering di-update → volatile fact, mungkin better as cron-fetched data
