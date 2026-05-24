---
name: ui-ux
description: Review / improve UI dan UX dengan prinsip grounded (WCAG, Material/HIG). Honest tentang trade-off, kasih opsi bukan single answer.
version: 2.0.0
metadata:
  hermes:
    tags: [ui, ux, design, accessibility, design-system, figma]
    category: design
---

# UI / UX Review

## KAPAN PAKAI

```
IF user minta "review UI" OR "improve UX" → PAKAI
IF user minta "accessibility audit" → PAKAI
IF user minta "pilih component pattern" → PAKAI
IF user minta "design system ringkas" → PAKAI
IF user minta implementasi kode frontend → JANGAN (pakai web-development / frontend-design)
IF user minta brand identity dari nol → JANGAN (butuh designer manusia)
```

---

## PROCEDURE (ikuti exact)

### Step 1: Context dulu (WAJIB sebelum review)

```
TANYA:
1. "Siapa target user?" (umur, tech-literacy, device)
2. "Primary task user di halaman ini?"
3. "Constraint?" (brand color, design system, component lib)
4. "Pain point?" (kalau ada)

DO NOT: review tanpa context
```

### Step 2: Observe artifact

```
IF screenshot → vision_analyze untuk extract elements
IF live URL → browser_navigate + browser_screenshot
IF code → read_file component

CATAT FAKTA (jangan langsung judge):
- Heading: [size, font, weight]
- Body: [size, font]
- CTA: [color, size, padding]
- Layout: [type, max-width]
- Background: [color, texture]
- Form: [fields, validation style]
```

### Step 3: Audit checklist

```
HIERARCHY:
□ CTA primary jelas beda dari secondary?
□ Yang harus user liat pertama = yang paling prominent?
□ Visual noise minimal?

TYPOGRAPHY:
□ Font families ≤ 2?
□ Size scale konsisten? (12/14/16/18/24/32)
□ Body line-height ≥ 1.5?
□ Line length 50-75 char?

SPACING:
□ Spacing scale konsisten? (4/8/12/16/24/32)
□ Whitespace cukup antar grup?

COLOR:
□ Contrast text vs bg ≥ 4.5:1 (body) / ≥ 3:1 (large text)? [WCAG AA]
□ Color bukan satu-satunya carrier info?

INTERACTIVITY:
□ Hover / focus / active state distinct?
□ Focus indicator visible (keyboard nav)?
□ Button vs link jelas? (button=action, link=navigation)

ACCESSIBILITY:
□ Form field punya visible label?
□ Image punya alt text?
□ Touch target ≥ 44x44px?

MOBILE:
□ Works at 375px tanpa horizontal scroll?
□ Tap-friendly (no tiny buttons)?
```

### Step 4: Classify issues

```
IF breaks usability / accessibility → 🔴 Kritis (wajib fix)
IF friction / convention break → 🟡 Penting (strong recommend fix)
IF polish / preference → 🟢 Polish (optional)
```

### Step 5: Output

---

## OUTPUT TEMPLATE

```markdown
## Review: [Page / Component]

### Yang udah bagus ✅
- [item positif 1]
- [item positif 2]

### 🔴 Kritis

#### [Issue title]
**Apa**: [deskripsi 1-2 kalimat]
**Kenapa masalah**: [reference: WCAG X.X / Material / HIG]
**Fix**: [actionable, specific]

### 🟡 Penting

#### [Issue title]
**Apa**: [deskripsi]
**Fix**: [actionable]

### 🟢 Polish
- [minor item] — fix: [saran singkat]

### Trade-offs
- [Opsi A vs B — kapan pilih masing-masing]

### Yang gw belum yakin
- [hal yang butuh user testing / A/B test]
```

---

## CONTOH OUTPUT

```markdown
## Review: Login Page

### Yang udah bagus ✅
- Layout bersih, focused pada 1 task
- Social login buttons prominent
- Password field punya show/hide toggle

### 🔴 Kritis

#### Contrast gagal WCAG AA
**Apa**: Placeholder text (#B0B0B0 on #FFFFFF) = ratio 2.6:1, di bawah 4.5:1 minimum
**Kenapa masalah**: WCAG 1.4.3 — user low-vision gak bisa baca
**Fix**: Ganti placeholder ke #666666 (ratio 5.7:1) atau pake floating label

#### Gak ada focus indicator
**Apa**: Tab through form → gak keliatan element mana yang active
**Kenapa masalah**: WCAG 2.4.7 — keyboard user completely lost
**Fix**: Tambah `outline: 2px solid #1E40AF` pada `:focus-visible`

### 🟡 Penting

#### Error message gak spesifik
**Apa**: "Login gagal" tanpa detail (password salah? email gak terdaftar?)
**Fix**: Distinguish: "Email tidak terdaftar" vs "Password salah"

### 🟢 Polish
- CTA button bisa lebih besar (current 36px height → recommend 44px)
- Social buttons alignment slight off (2px gap)

### Trade-offs
- "Login gagal" generic vs specific error: specific = better UX tapi bisa jadi info leak (attacker tau email registered). Kalau security concern → keep generic + rate limit.

### Yang gw belum yakin
- Apakah social login (Google/Apple) lebih sering dipake vs email? → A/B test
```

---

## QUICK REFERENCE

### Spacing scale
```
4 / 8 / 12 / 16 / 24 / 32 / 48 / 64
```

### Type scale
```
12 (caption) / 14 (small) / 16 (body) / 20 / 24 (h3) / 32 (h2) / 48 (h1)
```

### Contrast minimum (WCAG AA)
```
Body text: 4.5:1
Large text (18pt+): 3:1
UI component: 3:1
```

### Touch target minimum
```
Apple HIG: 44x44px
Material: 48x48px
```

---

## DECISION TREE: Common Questions

```
IF "modal vs drawer?":
  Modal: small content, confirmation, blocking action
  Drawer: large content, navigation, non-blocking
  IF mobile → drawer (modal takes full screen anyway)

IF "tab vs accordion?":
  Tab: ≤5 items, user likely visits multiple
  Accordion: many items, user visits 1-2

IF "dark or light theme?":
  → offer toggle. IF forced to choose:
    IF content-heavy (reading) → light
    IF media-heavy / dev tool → dark
```

---

## VERIFICATION

```
□ Setiap saran punya rationale (bukan "trust me")?
□ Priority labels match severity?
□ Ada section "Yang gw belum yakin"?
□ Saran realistis untuk project stack?
□ Gw gak claim user behavior tanpa data?

IF ada □ TIDAK → fix
```
