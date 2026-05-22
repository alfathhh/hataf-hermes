---
name: frontend-design
description: Create distinctive, production-grade frontend interfaces with high design quality. Use this skill when the user asks to build web components, pages, artifacts, posters, or applications (examples include websites, landing pages, dashboards, React components, HTML/CSS layouts, or when styling/beautifying any web UI). Generates creative, polished code and UI design that avoids generic AI aesthetics.
version: 1.0.0
metadata:
  hermes:
    tags: [frontend, design, ui, css, html, react, creative, web]
    category: development
---

# Frontend Design

> Adapted from [anthropics/skills/frontend-design](https://github.com/anthropics/skills/blob/main/skills/frontend-design/SKILL.md) (Anthropic official skill). Reformatted for Hermes Agent compatibility.

This skill guides creation of distinctive, production-grade frontend interfaces that avoid generic "AI slop" aesthetics. Implement real working code with exceptional attention to aesthetic details and creative choices.

The user provides frontend requirements: a component, page, application, or interface to build. They may include context about the purpose, audience, or technical constraints.

## When to Use

- User asks to build web components, pages, or applications
- User wants a landing page, dashboard, React component, HTML/CSS layout
- User asks to "beautify" or "style" any web UI
- User asks for a poster, visual artifact, or creative web piece
- User mentions wanting something "not generic" / "looks designed" / "premium feel"

JANGAN pakai untuk:
- Backend API logic (pakai `backend-proper`)
- UX flow review tanpa implementasi (pakai `ui-ux`)
- Data visualization / charting only (pakai code execution)

## Design Thinking

Before coding, understand the context and commit to a BOLD aesthetic direction:

- **Purpose**: What problem does this interface solve? Who uses it?
- **Tone**: Pick an extreme: brutally minimal, maximalist chaos, retro-futuristic, organic/natural, luxury/refined, playful/toy-like, editorial/magazine, brutalist/raw, art deco/geometric, soft/pastel, industrial/utilitarian, etc. There are so many flavors to choose from. Use these for inspiration but design one that is true to the aesthetic direction.
- **Constraints**: Technical requirements (framework, performance, accessibility).
- **Differentiation**: What makes this UNFORGETTABLE? What's the one thing someone will remember?

**CRITICAL**: Choose a clear conceptual direction and execute it with precision. Bold maximalism and refined minimalism both work — the key is intentionality, not intensity.

Then implement working code (HTML/CSS/JS, React, Vue, Svelte, etc.) that is:

- Production-grade and functional
- Visually striking and memorable
- Cohesive with a clear aesthetic point-of-view
- Meticulously refined in every detail

## Frontend Aesthetics Guidelines

Focus on:

### Typography
Choose fonts that are beautiful, unique, and interesting. Avoid generic fonts like Arial and Inter; opt instead for distinctive choices that elevate the frontend's aesthetics — unexpected, characterful font choices. Pair a distinctive display font with a refined body font.

### Color & Theme
Commit to a cohesive aesthetic. Use CSS variables for consistency. Dominant colors with sharp accents outperform timid, evenly-distributed palettes.

### Motion
Use animations for effects and micro-interactions. Prioritize CSS-only solutions for HTML. Use Motion library (framer-motion) for React when available. Focus on high-impact moments: one well-orchestrated page load with staggered reveals (`animation-delay`) creates more delight than scattered micro-interactions. Use scroll-triggering and hover states that surprise.

### Spatial Composition
Unexpected layouts. Asymmetry. Overlap. Diagonal flow. Grid-breaking elements. Generous negative space OR controlled density.

### Backgrounds & Visual Details
Create atmosphere and depth rather than defaulting to solid colors. Add contextual effects and textures that match the overall aesthetic. Apply creative forms like gradient meshes, noise textures, geometric patterns, layered transparencies, dramatic shadows, decorative borders, custom cursors, and grain overlays.

## NEVER (Anti-Patterns)

NEVER use generic AI-generated aesthetics:

- Overused font families (Inter, Roboto, Arial, system fonts)
- Cliched color schemes (particularly purple gradients on white backgrounds)
- Predictable layouts and component patterns
- Cookie-cutter design that lacks context-specific character
- Same font (Space Grotesk) across every generation
- Same layout structure every time

Interpret creatively and make unexpected choices that feel genuinely designed for the context. No design should be the same. Vary between light and dark themes, different fonts, different aesthetics. NEVER converge on common choices across generations.

## Implementation Complexity

**IMPORTANT**: Match implementation complexity to the aesthetic vision:

- **Maximalist designs** need elaborate code with extensive animations and effects
- **Minimalist or refined designs** need restraint, precision, and careful attention to spacing, typography, and subtle details
- Elegance comes from executing the vision well, not from throwing effects at everything

## Procedure (Hermes-specific)

### 1. Clarify context

Ask (if not provided):
- "Apa tujuan interface ini? Siapa target user?"
- "Ada preference estetika? (dark/light, vibe tertentu, brand reference?)"
- "Framework apa? (React/Vue/Svelte/plain HTML?)"
- "Butuh responsive? Target device?"

### 2. Commit to aesthetic direction

State clearly in output:
```
Aesthetic direction: [e.g., "Brutalist editorial — raw typography, mono stack, harsh contrast, intentional asymmetry"]
```

### 3. Implement

- Write complete, runnable code
- Use `write_file` untuk output (jangan paste 200+ baris di chat)
- Include font imports (Google Fonts / Fontsource / local)
- Include CSS variables at root
- Include at least 1 motion element (transition, animation, scroll-trigger)

### 4. Output format

```markdown
## Frontend: [Nama]

**Aesthetic**: [direction]
**Stack**: [HTML+CSS / React+Tailwind / Vue+CSS / etc]
**Key design choices**:
- Font: [display] + [body]
- Palette: [dominant → accent → neutral]
- Motion: [what animates, why]
- Differentiator: [the one memorable thing]

**Files**:
- `[path]` — [description]
```

### 5. Self-check

Before delivering:
- [ ] Is this VISUALLY DISTINCT from generic templates?
- [ ] Would a designer say "this has a point of view"?
- [ ] Does the code actually run without errors?
- [ ] Are fonts loaded (import/link present)?
- [ ] Is there at least one unexpected creative choice?
- [ ] Does contrast meet WCAG AA? (4.5:1 body text)

## Pitfalls

### Pitfall 1: Defaulting to safe choices

If you catch yourself reaching for Inter + purple gradient + card grid — STOP. Restart the design thinking step. Choose something with character.

### Pitfall 2: All flash, no function

Creative ≠ broken. Every element must be accessible (keyboard nav, screen reader labels) and functional. Beauty that doesn't work = failed design.

### Pitfall 3: Same output every time

Vary deliberately: if last output was dark/minimal, next one should be light/maximal or retro/warm. Track your recent outputs (session context) and diverge.

### Pitfall 4: Ignoring project stack

If project uses Tailwind, output Tailwind classes — don't write vanilla CSS that conflicts. If project uses Vue, don't output React JSX. Stack discovery FIRST.

### Pitfall 5: Forgetting mobile

Unless specified desktop-only, every layout must work at 375px width minimum. Test mentally: would this grid collapse break on mobile?

## Verification

1. Would this pass as "human-designed" (not "AI-generated")?
2. Is the aesthetic direction clear and consistent throughout?
3. Is code production-ready (no TODO, no placeholder)?
4. Are fonts, colors, and spacing intentional (not default)?
5. Is there responsive behavior?
6. At least WCAG AA contrast on text?

## Attribution

This skill is adapted from the official Anthropic `frontend-design` skill ([source](https://github.com/anthropics/skills/blob/main/skills/frontend-design/SKILL.md)). Reformatted with Hermes-compatible frontmatter, added procedure steps, pitfalls, and verification sections per Hermes SKILL.md convention.
