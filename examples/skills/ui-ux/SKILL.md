---
name: ui-ux
description: Review / improve / design UI dan UX dengan prinsip yang grounded (WCAG, Material/HIG, atau design system project). Honest tentang trade-off, gak halu best practice, kasih opsi bukan single "right answer".
version: 1.0.0
metadata:
  hermes:
    tags: [ui, ux, design, accessibility, design-system, figma]
    category: design
---

# UI / UX

Skill untuk task design-related: review interface existing, kasih saran improvement, design new flow, atau jelasin trade-off design choice.

> **Posisi gw jujur**: gw bukan designer manusia. Saran gw based on best practice umum + dokumentasi resmi (WCAG, Material Design, Apple HIG, Tailwind UI patterns). Untuk judgment estetis subtle (warna emosional, branding voice), output gw cuma starter — final call manusia.

## When to Use

User minta:
- Review UI screenshot / Figma / live page
- Saran improvement UX flow
- Pilih component pattern (modal vs drawer? tab vs accordion?)
- Accessibility audit
- Design system ringkas
- Critique layout / typography

JANGAN pakai untuk:
- Brand identity dari nol (butuh designer manusia + research customer)
- Animasi kompleks (butuh tooling khusus, skill design motion lain)

## Reference Frame (yang gw rujuk, bukan ngarang)

Setiap saran gw, harusnya di-back oleh salah satu:

| Source | Untuk |
|---|---|
| [WCAG 2.2](https://www.w3.org/TR/WCAG22/) | Accessibility — kontras, keyboard nav, screen reader |
| [Material Design 3](https://m3.material.io/) | Component pattern, spacing scale (Google ecosystem) |
| [Apple HIG](https://developer.apple.com/design/human-interface-guidelines) | iOS / macOS / VisionOS pattern |
| [Refactoring UI](https://www.refactoringui.com/) | Practical UI taste (book by Steve Schoger & Adam Wathan) |
| [Inclusive Components](https://inclusive-components.design/) | Pattern accessible (Heydon Pickering) |
| Existing design system project | Nano-decision (kalau ada Tailwind UI / shadcn / Radix / Chakra di project, ikutin) |

Kalau saran gw clearly "best practice", gw kutip framework atau spec-nya. Kalau itu opini gw / common sense → gw bilang "ini common practice, bukan rule formal".

## Procedure

### 1. Konteks dulu

Sebelum kritik / saran, tanya:

- "Siapa target user? (umur, tech-literacy, language, device primer)"
- "Apa primary task user di halaman / flow ini?"
- "Apa constraint? (brand color, design system, existing component lib)"
- "Apa pain point existing? (kalau ada — supaya saran spesifik)"

JANGAN review tanpa konteks. UI bagus untuk admin tool ≠ UI bagus untuk consumer app.

### 2. Liat artifact

Kalau ada Figma / screenshot / live URL:
- Screenshot: `vision_analyze` untuk extract elements + layout
- Live URL: `browser_navigate` + `browser_screenshot`
- Code: `read_file` component utama

Catat fakta visual, jangan langsung judge:

```markdown
## Pengamatan (fakta dulu)
- Heading: 32px Inter Bold
- Body: 16px Inter Regular
- Primary CTA: blue #1E40AF, 14px label, padding 8px 16px
- Layout: centered max-w-md
- Background: white, no decorative element
- Form: 4 fields stacked, no inline validation
```

### 3. Audit struktural

Pakai checklist berikut:

#### Hierarchy
- Apa yang harus user liat pertama? Apa yang prominent?
- Apakah CTA primary jelas beda dari secondary?
- Apakah ada visual noise (border, shadow, color berlebihan)?

#### Typography
- Berapa font family? (>2 = noise)
- Berapa size step? (gunakan scale konsisten, misal 12/14/16/18/24/32)
- Line-height untuk body ≥ 1.5?
- Line length 50-75 char untuk paragraf panjang?

#### Spacing
- Konsisten? (4 / 8 / 12 / 16 / 24 / 32 — pake scale, bukan random)
- Whitespace cukup antar grup vs dalam grup?

#### Color
- Kontras text vs background ≥ 4.5:1 (body) atau ≥ 3:1 (large text 18pt+)? [WCAG AA]
- Brand color cuma di accent, bukan blanket?
- Status color (error/warning/success) tidak overlap dengan brand?

#### Interactivity
- Hover / focus / active state distinct?
- Focus indicator visible dengan keyboard nav?
- Button vs link clear (button untuk action, link untuk navigation)?

#### Accessibility
- Form field punya label (visible atau aria)?
- Image punya alt text deskriptif?
- Color tidak satu-satunya carrier info (jangan red = error tanpa icon/text)?
- Touch target minimal 44x44 px (Apple HIG) atau 48x48 px (Material)?

#### Mobile
- Responsive di 375px width (iPhone SE) tanpa horizontal scroll?
- Tap-friendly (no tiny button tight to others)?

### 4. Output: critique + saran prioritized

Format:

```markdown
## Review: [Page / Component]

### Yang udah bagus
- [Item 1] — [alasan singkat]
- [Item 2]

### Issue (priority order)

#### 1. 🔴 Kritis — [Judul]
**Apa**: [Deskripsi issue]
**Kenapa masalah**: [Reference: WCAG 1.4.3, atau pattern dari Material, dst]
**Fix**: [Konkrit, actionable]
**Visual contoh**: [Sketch dengan teks ASCII / referensi lib]

#### 2. 🟡 Penting — [Judul]
...

#### 3. 🟢 Polish — [Judul]
...

### Trade-offs yang patut diketahui
- [Opsi A vs B — kapan pilih masing-masing]

### Yang gw belum yakin
- [Hal yang butuh user validation atau A/B test]
```

> 🔴 = breaks usability / a11y. Wajib fix.
> 🟡 = friction / convention break. Strong rekomendasi fix.
> 🟢 = polish / preference. Optional.

### 5. Self-check kejujuran

Sebelum kirim, cek:

- Apakah saran gw karena "best practice yang well-documented" atau "preferensi gw aja"?
- Apakah gw nge-claim "users prefer X" tanpa data?
- Apakah gw kasih satu jawaban di tempat yang sebenarnya ada multiple valid?

Kalau ada bias, balance dulu.

## Pitfalls

### Pitfall 1: Halu "research says..."

❌ "Penelitian menunjukkan user mayoritas prefer dark theme"
✅ "Dark mode is widely supported as preference toggle. Apakah mayoritas user prefer atau gak — depends on demographic dan use case, gak ada single global answer"

JANGAN kutip "research" tanpa link ke paper / studi yang bisa dicek.

### Pitfall 2: Cargo-cult pattern dari startup besar

"Stripe pakai modal X" / "Linear pakai shortcut Y" — itu valid untuk app mereka. Belum tentu untuk app lo.

Tanya: "User base lo similar?" Kalau gak, jangan copy paste pattern.

### Pitfall 3: Over-prescribing single solution

Banyak design choice = trade-off, bukan correct answer.

Contoh: tab vs accordion vs separate pages — tergantung jumlah konten, frequency, mobile vs desktop primer. Kasih opsi + when each fits.

### Pitfall 4: Ignoring context tech

Saran "tambahin micro-animation transition smooth" kalau project static HTML tanpa framework — over-engineered.

Saran "pake Floating UI untuk dropdown positioning" kalau project pake Bootstrap — adds dependency non-trivial.

Cek stack dulu (lihat skill `web-development` step 1).

### Pitfall 5: A11y sebagai afterthought

Jangan kasih design + "lalu nanti tinggal tambah aria". A11y harus integrated dari awal — keyboard order, focus management, semantic HTML.

### Pitfall 6: Asumsi monitor designer

Designer biasanya pake monitor kalibrasi 27" 4K. User lo? Mungkin layar laptop 1366×768 murah, atau HP layar 5", atau projector buram. Test di range device.

## Verification

1. Apakah setiap saran punya rationale (bukan "trust me")?
2. Apakah priority labels (🔴🟡🟢) match severity nyata?
3. Apakah ada section "Yang gw belum yakin"?
4. Apakah saran realistis untuk stack project (bukan minta library yang berat untuk SPA simple)?
5. Apakah gw nge-claim user behavior tanpa data?

## Cheatsheet Cepat

### Spacing scale (default rekomendasi)
```
4 / 8 / 12 / 16 / 24 / 32 / 48 / 64 / 96
```
Pake yang konsisten — jangan campur 7px, 13px, 22px random.

### Type scale (default rekomendasi)
```
12 (caption) / 14 (small) / 16 (body) / 18 / 20 / 24 (h3) / 32 (h2) / 48 (h1) / 64 (display)
```

### Color contrast minimum (WCAG AA)
```
Body text vs background:    4.5:1
Large text (18pt+):         3:1
UI component / icon:        3:1
```
Tools cek: webaim.org/resources/contrastchecker, atau Chrome DevTools Lighthouse.

### Touch target minimum
```
Apple HIG:     44 x 44 px
Material:      48 x 48 px
WCAG AAA:      44 x 44 css px (2.5.5)
```

### Form heuristics
- Label di atas field (kecuali inline minimal form)
- Required marker visible
- Inline validation on blur, bukan on every keystroke
- Error message konkret ("Password must be ≥8 chars") bukan "Invalid"
- Submit button labeled action ("Save changes" bukan "Submit")

### Responsive breakpoints (Tailwind default — sesuaikan dengan project)
```
sm:  640px
md:  768px
lg:  1024px
xl:  1280px
2xl: 1536px
```

## Untuk Design System ringan

Kalau project belum punya design system:

```markdown
## Quick Design System (1 halaman)

### Color
- Primary: <hex>
- Primary dark: <hex>  
- Neutral 50/100/200/.../900: <ladder>
- Success / Warning / Error: <hex hex hex>

### Typography
- Font: <Inter / Geist / system-ui>
- Scale: 12/14/16/18/24/32 (line-height 1.5)

### Spacing scale
- 4/8/12/16/24/32/48/64

### Radius
- sm: 4px
- md: 8px
- lg: 12px
- full: 9999px

### Shadow
- sm: 0 1px 2px rgba(0,0,0,0.05)
- md: 0 4px 6px rgba(0,0,0,0.1)
- lg: 0 10px 15px rgba(0,0,0,0.1)

### Component conventions
- Button: padding 8px 16px (md), radius md, transition 150ms
- Input: height 40px, radius md, border 1px neutral-300
- Card: padding 16/24px, radius lg, shadow sm
```

Itu starter. Refine berdasarkan brand + user research.
