---
name: weekly-review
description: Rutinitas mingguan self-improvement agent. Review sesi terakhir, identifikasi pattern error, update memory/skill.
version: 2.0.0
metadata:
  hermes:
    tags: [meta, self-improvement, review, memory, skill-management]
    category: meta
---

# Weekly Self-Improvement Review

## KAPAN PAKAI

```
IF seminggu sekali (Jumat/Minggu sore) → PAKAI
IF setelah sprint intensif → PAKAI
IF user merasa agent "gak belajar" → PAKAI
IF manual trigger "/weekly-review" → PAKAI
```

---

## PROCEDURE (5 phases, ikuti exact)

### Phase 1: GATHER

```python
session_search(query="error OR mistake OR correction OR retry OR failed")
session_search(query="remember OR ingat OR save OR memory")
```

CARI:
- Agent salah + dikoreksi user
- Agent harus retry/debug
- User bilang "ingat ini"
- Tool/approach yang salah
- Pattern pertanyaan sering

### Phase 2: ANALYZE

ISI template ini:

```markdown
## 📊 Weekly Review — [Tanggal]

### 🔴 Repeated Mistakes
| # | Pattern | Frequency | Impact | Root cause |
|---|---|---|---|---|
| 1 | [mistake] | [Nx] | [High/Med/Low] | [why] |

### 💾 Missing Memories
| # | Info yang harusnya saved | Target | Alasan |
|---|---|---|---|
| 1 | [info] | [USER/MEMORY] | [muncul Nx, belum saved] |

### 🛠️ Workflows → Skills
| # | Workflow | Frequency | Proposed skill |
|---|---|---|---|
| 1 | [workflow] | [Nx] | [skill name] |

### 📦 Outdated Skills
| # | Skill | Issue | Fix |
|---|---|---|---|
| 1 | [skill] | [what broke] | [how to fix] |

### ⚠️ Risky Assumptions
| # | Assumption | Risk | Verify how |
|---|---|---|---|
| 1 | [assumption] | [Med/High] | [method] |
```

### Phase 3: PROPOSE

```markdown
## Proposed Updates

### Auto-apply (safe):
1. ADD memory: "[fact]"
2. PATCH skill [name]: [change]

### Needs user approval:
1. UPDATE SOUL.md: [change] — reason: [why]
2. CREATE skill [name]: [what it does]
```

### Phase 4: EXECUTE

```
IF update = memory add/replace → auto-apply
IF update = skill patch (non-destructive) → auto-apply
IF update = context file (SOUL.md, AGENTS.md) → ASK USER first
IF update = skill delete/archive → ASK USER first
IF update = memory remove → ASK USER first (mungkin masih relevan)
```

### Phase 5: REPORT

---

## OUTPUT TEMPLATE

```markdown
## 📊 Weekly Review — [Tanggal]

### Summary
- 🔴 Repeated mistakes: [N]
- 💾 Memory gaps filled: [N]
- 🛠️ New skills proposed: [N]
- 📦 Skills updated: [N]

### Auto-applied ✅
- [list apa yang udah di-update]

### Needs your decision ❓
1. [Update X] — reason: [Y]. Proceed? (yes/no)

### Key insight minggu ini 💡
[1-2 kalimat paling penting yang agent "pelajari"]
```

---

## CONTOH OUTPUT

```markdown
## 📊 Weekly Review — 24 Mei 2026

### Summary
- 🔴 Repeated mistakes: 2
- 💾 Memory gaps filled: 1
- 🛠️ New skills proposed: 1
- 📦 Skills updated: 1

### Auto-applied ✅
- ADD memory: "User prefer Tailwind v4 (bukan v3) di semua project baru"
- PATCH skill harga-emas: update extraction prompt (logammulia.com redesign)

### Needs your decision ❓
1. CREATE skill "cloudflare-deploy" — workflow deploy ke CF Pages muncul 4x minggu ini. Worth jadi skill? (yes/no)
2. UPDATE SOUL.md: tambah rule "always check Tailwind version before writing classes" — karena 2x salah versi. Proceed? (yes/no)

### Key insight minggu ini 💡
Paling sering error karena asumsi Tailwind v3 padahal project udah v4. Perlu cek `package.json` SETIAP kali sebelum nulis class.
```

---

## DECISION TREE: Apakah Worth Jadi Skill?

```
IF workflow muncul ≥ 3x dalam 2 minggu → worth jadi skill
IF workflow muncul 1-2x → cukup di memory
IF workflow butuh >20 step → worth jadi skill
IF workflow simple (3-5 step) → cukup di memory
```

## DECISION TREE: Progress Tracking

```
Week 1: baseline (identify all problems)
Week 2: fix top 2-3 problems
Week 3: verify fixes work, find new problems
Week 4: should be stabilizing

IF repeated mistakes MASIH muncul setelah 3 minggu:
  → systemic issue, perlu rule di SOUL.md (bukan cuma memory)

IF skill di-patch >3x:
  → approach fundamentally wrong, REWRITE dari scratch
```

---

## VERIFICATION

```
□ Semua 5 sections terisi (atau explicitly "gak ada")?
□ Ada minimal 1 actionable update?
□ Risky assumptions flagged?
□ Auto-applied updates non-destructive?
□ User-approval items clearly marked?

IF ada □ TIDAK → fix before send
```
