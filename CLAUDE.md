# AI Animation Lesson — Workshop Site

## Project Overview
Interactive single-page workshop site for teaching designers about AI + SVG animation. Hosted on GitHub Pages.

- **Repo:** https://github.com/alecramos-sudo/ai-animation-lesson
- **Live:** https://alecramos-sudo.github.io/ai-animation-lesson/
- **Audience:** 4-5 designers, 1-hour workshop
- **Credit:** Willa Creative (no Peak Perspective Media)

## Architecture
- Single `index.html` — no build step, no dependencies, no frameworks
- Embedded CSS (custom properties) + vanilla JS
- Inline SMIL SVG for all animations
- GitHub Pages serves from `main` branch root

## Key Conventions
- Light, warm educational theme (#FAFAF8 bg, system fonts)
- SVG element IDs prefixed by section: `pulse-`, `expand-`, `spin-`, `drawon-`, `anatomy-`, `caglia-`, `ref-`
- Playground pattern: `generators[name]()` returns SVG string, `renderPlayground(name)` injects via innerHTML
- SMIL restart: re-render full SVG, optionally call `beginElement()`. Never `setAttribute()` on existing animate nodes.
- Reduced-motion: JS-based `pauseAnimations()`/`unpauseAnimations()` with global `animationsPaused` state
- Slider controls only (no code editors visible) — designer-friendly
- External links use `target="_blank" rel="noopener noreferrer"`

## Page Structure
1. Hero
2. Section 1: "What AI Can Do Today" — tool landscape
3. Section 2: "How It's Changing Design Work" — 3 quote blocks
4. Transition
5. Section 3: "Meet the SVG" — annotated anatomy diagram
6. Section 4: "One Icon, Many Motions" — 4 interactive playgrounds (Pulse, Hypnotic Zoom, Spin, Draw-On)
7. Section 5: "A Real Example" — Caglia before/after + prompt
8. Section 6: "The AI Workflow" — Figma MCP, tool comparison (incl. HeroUI Studio), good/bad prompts
9. Section 7: "Take It Home" — reference grid + tools list (incl. HeroUI Studio)
10. Footer — Willa Creative

## Reference Files (not linked from main page)
- `icon-variants.html` — dark-themed showcase of 20+ animated SVG variants
- `spiral-preview.html` — simple before/after preview
- `svg-animation-lesson.md` — full written lesson (8 lessons)
