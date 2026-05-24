---
name: frontend-design
description: Create distinctive, production-grade frontend interfaces. Anti-generic AI aesthetics. Bold design choices.
version: 2.0.0
metadata:
  hermes:
    tags: [frontend, design, ui, css, html, react, creative, web]
    category: development
---

# Frontend Design

## KAPAN PAKAI

```
IF user minta "bikin UI cantik" OR "landing page" OR "beautify" → PAKAI
IF user minta web component dengan emphasis di visual → PAKAI
IF user minta poster / visual artifact web → PAKAI
IF user minta "not generic" OR "looks designed" → PAKAI
IF user minta backend API → JANGAN (pakai backend-proper)
IF user minta UX review tanpa implementasi → JANGAN (pakai ui-ux)
```

---

## PROCEDURE (ikuti exact)

### Step 1: Clarify context

```
TANYA (kalau belum clear):
1. "Tujuan interface ini? Siapa user?"
2. "Preference estetika? (dark/light/retro/minimal/bold)"
3. "Framework? (React/Vue/Svelte/plain HTML)"
4. "Responsive? Target device?"
```

### Step 2: Commit to aesthetic direction

```
PILIH 1 aesthetic direction (JANGAN generic):
- Brutalist (raw typography, harsh contrast, mono)
- Maximalist (rich colors, layers, animation heavy)
- Minimalist refined (negative space, elegant type, subtle)
- Retro-futuristic (neon, gradients, terminal-vibe)
- Editorial/magazine (grid, large type, editorial layout)
- Organic/natural (earthy tones, soft curves, texture)
- Luxury/refined (gold accents, serif fonts, high contrast)
- Playful/toy-like (rounded, bright, bouncy animations)

STATE direction di output: "Aesthetic: [direction]"
```

### Step 3: Design choices (MANDATORY)

```
CHOOSE for each:
- Font display: [distinctive, NOT Inter/Arial/Roboto]
- Font body: [readable, pairs with display]
- Palette: [dominant → accent → neutral]
- Motion: [1+ animation that adds delight]
- Differentiator: [1 thing someone will remember]

DO NOT: use Inter, Roboto, Arial, system fonts (generic)
DO NOT: purple gradient on white background (AI slop)
DO NOT: same layout structure every time
```

### Step 4: Implement

```
RULES:
- Write COMPLETE runnable code (no TODO, no placeholder)
- Include font imports (Google Fonts / Fontsource)
- Include CSS variables at :root
- Include 1+ motion element (transition, animation)
- WCAG AA contrast minimum (4.5:1 body text)
- Responsive at 375px minimum

OUTPUT: write_file() (jangan paste 200+ baris di chat)
```

### Step 5: Self-check

```
□ Visually DISTINCT from generic templates?
□ A designer would say "this has a point of view"?
□ Code runs without errors?
□ Fonts loaded (import/link present)?
□ 1+ unexpected creative choice?
□ WCAG AA contrast met?
□ Works at 375px?

IF any □ TIDAK → revise
```

---

## OUTPUT TEMPLATE

```markdown
## Frontend: [Nama]

**Aesthetic**: [direction]
**Stack**: [HTML+CSS / React+Tailwind / etc]
**Key design choices**:
- Font: [display] + [body]
- Palette: [dominant → accent → neutral]
- Motion: [what animates, why]
- Differentiator: [the one memorable thing]

**Files**:
- `[path]` — [description]

**Preview notes**:
- Open [file] in browser to preview
- Responsive: tested at 375px, 768px, 1280px
```

---

## CONTOH OUTPUT

```markdown
## Frontend: Startup Landing Page

**Aesthetic**: Editorial magazine — large typography, asymmetric grid, dramatic white space
**Stack**: HTML + CSS (no framework, pure)
**Key design choices**:
- Font: "Playfair Display" (display) + "Source Sans 3" (body)
- Palette: #1a1a2e (dominant dark) → #e94560 (accent red) → #f5f5f5 (neutral)
- Motion: Hero text slides in with staggered delay; CTA button has magnetic hover
- Differentiator: Split-screen layout with text left, oversized image right bleeding off edge

**Files**:
- `output/landing-page/index.html` — full page
- `output/landing-page/style.css` — all styles + animations

**Preview notes**:
- Open index.html in browser
- Responsive: stacks vertically on mobile, grid on desktop
- Font loaded via Google Fonts CDN
```

---

## DECISION TREE: Aesthetic by Context

```
IF product = SaaS B2B → Clean, professional, data-forward. NOT playful.
IF product = Consumer app → Bold, memorable, personality-driven.
IF product = Portfolio/personal → Express personality. Can be experimental.
IF product = E-commerce → Trust signals, clean product display, fast-feeling.
IF product = Blog/content → Editorial, type-focused, readable.
IF user says "dark mode" → Dark bg, light text, careful with accent colors.
IF user says "minimal" → Restraint. Fewer colors, more whitespace, precision spacing.
IF user says "creative/bold" → Go maximalist. Overlap, asymmetry, unexpected.
```

## DECISION TREE: Framework Choice

```
IF project uses React → output JSX + CSS Modules or Tailwind
IF project uses Vue → output .vue SFC
IF project uses Svelte → output .svelte
IF project uses Tailwind → use Tailwind classes (NOT vanilla CSS)
IF no framework specified → plain HTML + CSS (most universal)
IF user says "quick prototype" → plain HTML + CSS
```

---

## ANTI-PATTERNS (JANGAN lakukan)

```
DO NOT: Inter font family
DO NOT: purple gradient on white
DO NOT: predictable card grid with rounded corners
DO NOT: same Space Grotesk across every generation
DO NOT: cookie-cutter Bootstrap look
DO NOT: same structure every time (VARY deliberately)
```

---

## VERIFICATION

```
□ Would this pass as "human-designed" (not "AI-generated")?
□ Aesthetic direction clear and consistent?
□ Code production-ready (no TODO, no placeholder)?
□ Fonts, colors, spacing intentional (not default)?
□ Responsive?
□ WCAG AA contrast on text?

IF ada □ TIDAK → revise before deliver
```
