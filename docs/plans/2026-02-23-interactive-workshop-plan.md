# Interactive SVG Animation Workshop — Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build a single-page interactive workshop site for 4-5 designers, hosted on GitHub Pages, covering AI for designers + hands-on SVG animation with slider controls.

**Architecture:** Single `index.html` with embedded CSS and vanilla JS. No build step, no dependencies. Sections scroll vertically. Interactive playgrounds use slider inputs bound to SVG attributes via JS. All four playgrounds share a single `renderPlayground(name)` function pattern. GitHub Pages serves it publicly.

**Tech Stack:** HTML, CSS (custom properties), vanilla JavaScript, inline SMIL SVG

**Conventions:**
- All SVG element IDs are prefixed by playground name to avoid collisions (e.g., `pulse-mid-anim`, `spin-inner-anim`, `expand-clip`)
- SMIL update strategy: each playground's generator returns a full SVG string; `renderPlayground()` replaces the container's innerHTML. After insertion, optionally call `beginElement()` on animate nodes to ensure they start. Do NOT use `setAttribute()` on existing animate nodes — always re-render.
- All sliders use `<label for="id">` and work via keyboard
- Reduced-motion support via JS (SMIL ignores CSS `animation-play-state`): on load, check `matchMedia('(prefers-reduced-motion: reduce)')`. If matched, call `svg.pauseAnimations()` on each playground SVG. Provide a "Play animations" toggle button that calls `unpauseAnimations()`/`pauseAnimations()` on all SVGs.
- External links use `rel="noopener noreferrer"`

---

### Task 1: Git Init + GitHub Repo

**Files:**
- Create: `.gitignore`

**Step 1: Initialize git and create .gitignore**

```gitignore
.DS_Store
.claude/
*.swp
*.swo
*~
.env
.env.*
node_modules/
```

**Step 2: Create GitHub repo and push**

```bash
cd /Users/alecramosnv/Projects/ai-animation-lesson
git add .gitignore svg-animation-lesson.md icon-variants.html spiral-preview.html docs/
git commit -m "chore: initial commit with lesson content and design docs"
gh repo create ai-animation-lesson --public --source=. --push
```

**Step 3: Enable GitHub Pages**

```bash
OWNER=$(gh api user -q .login)
gh api "repos/${OWNER}/ai-animation-lesson/pages" -X POST -f source.branch=main -f source.path=/ 2>/dev/null || true
# Verify Pages is actually enabled:
PAGES_URL=$(gh api "repos/${OWNER}/ai-animation-lesson/pages" -q .html_url)
echo "Pages enabled at: ${PAGES_URL}"
# If PAGES_URL is empty, something went wrong — check repo settings manually
```

**Step 4: Verify**

```bash
gh repo view --web
```

Expected: repo created, pages enabled on main branch.

---

### Task 2: Page Shell — HTML Structure + CSS Foundation

**Files:**
- Create: `index.html`

**Step 1: Create `index.html` with full page structure (empty sections) and all CSS**

The HTML has these sections in order:
1. Hero/title
2. Section 1: "What AI Can Do Today"
3. Section 2: "How It's Changing Design Work"
4. Transition moment
5. Section 3: "Meet the SVG"
6. Section 4: "One Icon, Many Motions" (4 playground containers)
7. Section 5: "A Real Example"
8. Section 6: "The AI Workflow"
9. Section 7: "Take It Home"

CSS requirements:
- CSS custom properties: `--bg: #FAFAF8`, `--text: #1a1a1a`, `--text-muted: #6b7280`, `--accent: #2563eb`, `--card-bg: #ffffff`, `--card-border: #e5e7eb`, `--card-shadow: 0 1px 3px rgba(0,0,0,0.08)`
- Font: `system-ui, -apple-system, 'Segoe UI', sans-serif` (no external dependency)
- Sections: `max-width: 800px`, centered, `padding: 80px 24px`
- Section headers: large (clamp-based responsive sizing), dark, generous margin-bottom
- Body text: 18px, line-height 1.6, `--text-muted` color
- Cards for playgrounds: white bg, subtle border + shadow, 24px padding, 12px border-radius
- Custom slider styling (`::-webkit-slider-thumb`, `::-moz-range-thumb`) — rounded thumb, accent track
- Responsive: stacks cleanly on mobile, sliders go full-width
- Smooth scroll between sections: `html { scroll-behavior: smooth }`
- Reduced-motion: JS-based (not CSS) — on load, if `matchMedia('(prefers-reduced-motion: reduce)').matches`, call `document.querySelectorAll('svg').forEach(s => s.pauseAnimations())`. A sticky "Play animations" toggle button calls `unpauseAnimations()`/`pauseAnimations()` on all SVGs.
- All `<input type="range">` elements have associated `<label for="...">` elements

**Step 2: Open in browser to verify empty shell renders correctly**

```bash
open /Users/alecramosnv/Projects/ai-animation-lesson/index.html
```

**Step 3: Commit**

```bash
git add index.html
git commit -m "feat: page shell with HTML structure and CSS foundation"
```

---

### Task 3: Act 1 — Content Sections (AI for Designers)

**Files:**
- Modify: `index.html`

**Step 1: Fill Section 1 — "What AI Can Do Today"**

Large heading. 3-4 brief content blocks, each with a bold label and 1-2 sentences:
- **Text & Code** — ChatGPT, Claude, Gemini. Conversational AI that writes, edits, summarizes, codes. Free tiers available, paid unlocks longer context and better models.
- **Images & Video** — Midjourney, DALL-E, Runway, Firefly. Generate and edit visuals from text prompts. Quality is production-ready for many use cases.
- **Design & Prototyping** — Figma AI, Galileo, v0. AI is entering the design tool workflow directly.
- **What they all share** — They work best when you're specific. Vague input = generic output. The skill is learning to direct them.

Style each as a card or a clean typographic block with generous spacing.

**Step 2: Fill Section 2 — "How It's Changing Design Work"**

3 key points, large text, minimal:
- "AI doesn't replace taste — it accelerates execution"
- "The gap between idea and prototype is shrinking to minutes"
- "Knowing what to ask for is the new core skill"

Each point gets its own visual block — big quote-style text, maybe with a subtle left border accent.

**Step 3: Fill Transition**

A single centered line, slightly larger, with visual breathing room:
> "Let's get hands-on with one specific thing AI is great at — turning static icons into animated ones."

**Step 4: Verify in browser, commit**

```bash
git add index.html
git commit -m "feat: act 1 content — AI for designers overview"
```

---

### Task 4: Section 3 — "Meet the SVG" (Anatomy Diagram)

**Files:**
- Modify: `index.html`

**Step 1: Create an annotated SVG anatomy visual**

Show the spiral icon (3 concentric circles) at large size (~200px) with visual labels pointing to:
- The outer boundary → labeled "viewBox (30x30)"
- The outer circle → labeled "circle r=14.5"
- The middle circle → labeled "circle r=9.5"
- The inner circle → labeled "circle r=4.5"
- The center point → labeled "center (15, 15)"

Implementation: Use an SVG that includes the 3 circles plus `<text>` or `<line>` elements for annotation. Or use HTML labels absolutely positioned over the SVG. Whichever is cleaner. Use id prefix `anatomy-` for all elements.

Below the diagram, 3 brief bullet points:
- "Every SVG is just shapes described in coordinates"
- "The viewBox is the coordinate system — animations use these numbers"
- "Each shape can be animated independently"

No code shown. Just the visual + plain language.

**Step 2: Verify in browser, commit**

```bash
git add index.html
git commit -m "feat: section 3 — SVG anatomy diagram with labels"
```

---

### Task 5: Section 4 — Shared Playground JS Pattern + Playground 1: Pulse

**Files:**
- Modify: `index.html`

**Step 1: Create the shared `renderPlayground()` helper**

All four playgrounds share this pattern:
```javascript
let animationsPaused = false;

function renderPlayground(name) {
  const container = document.getElementById(`${name}-svg-container`);
  const svgMarkup = generators[name]();  // returns an SVG string
  container.innerHTML = svgMarkup;
  // After injecting, call beginElement() on all animate/animateTransform children
  container.querySelectorAll('animate, animateTransform').forEach(el => {
    try { el.beginElement(); } catch(e) {}
  });
  // Respect reduced-motion state after re-render
  if (animationsPaused) {
    const svg = container.querySelector('svg');
    if (svg) svg.pauseAnimations();
  }
}

// Initialize reduced-motion preference
const motionQuery = matchMedia('(prefers-reduced-motion: reduce)');
if (motionQuery.matches) animationsPaused = true;
motionQuery.addEventListener('change', (e) => {
  animationsPaused = e.matches;
  document.querySelectorAll('svg').forEach(s => {
    e.matches ? s.pauseAnimations() : s.unpauseAnimations();
  });
});
```

Each playground registers a generator function in a `generators` object. Sliders call `renderPlayground(name)` on `oninput`.

**Step 2: Build the Pulse playground**

Card containing:
- Title: "Pulse" with a one-line description: "A gentle breathing motion — the shape grows and shrinks rhythmically."
- Left side (or top on mobile): SVG preview container `id="pulse-svg-container"`
- Right side (or bottom on mobile): Sliders
  - **Duration** — `id="pulse-dur"`, range 1 to 6, default 2, step 0.5. `<label for="pulse-dur">Duration: <span>2s</span></label>`
  - **Intensity** — `id="pulse-intensity"`, range 0.5 to 3, default 1, step 0.5. `<label for="pulse-intensity">Intensity: <span>1</span></label>`

Generator function:
- Reads slider values: `dur`, `intensity`
- Returns SVG string with:
  - Outer circle static: `r="14.5"`, id `pulse-outer`
  - Middle circle: `<animate id="pulse-mid-anim" attributeName="r" values="9.5;${9.5+intensity};9.5" dur="${dur}s" repeatCount="indefinite"/>`
  - Inner circle: same pattern with `r` base 4.5, `begin="0.3s"`, id `pulse-inner-anim`
- All SVG IDs prefixed with `pulse-`

**Step 3: Verify sliders update the animation in browser**

**Step 4: Commit**

```bash
git add index.html
git commit -m "feat: section 4 — shared playground pattern + pulse playground"
```

---

### Task 6: Section 4 — Playground 2: Expanding Radius (Hypnotic Zoom)

**Files:**
- Modify: `index.html`

**Step 1: Build the Expand playground**

Card containing:
- Title: "Hypnotic Zoom" — "Circles expand outward from center in continuous waves."
- SVG container: `id="expand-svg-container"`
- Sliders:
  - **Duration** — `id="expand-dur"`, 2 to 8, default 3, step 0.5
  - **Wave Count** — `id="expand-waves"`, 2 to 5, default 3, step 1

Generator function:
- Reads `dur`, `waveCount`
- Computes wave offset: `dur / waveCount`
- Returns SVG with:
  - `<defs><clipPath id="expand-clip"><circle cx="15" cy="15" r="14"/></clipPath></defs>`
  - Static outer circle
  - `<g clip-path="url(#expand-clip)">` containing `waveCount` sets of 3 expanding circles each, with `begin="${i * offset}s"`
- All IDs prefixed with `expand-`

**Step 2: Verify, commit**

```bash
git add index.html
git commit -m "feat: section 4 — expanding radius playground"
```

---

### Task 7: Section 4 — Playground 3: Spin

**Files:**
- Modify: `index.html`

**Step 1: Build the Spin playground**

Card containing:
- Title: "Spin" — "Dashed arcs rotate around center — counter-rotate for visual complexity."
- SVG container: `id="spin-svg-container"`
- Sliders:
  - **Duration** — `id="spin-dur"`, 1 to 8, default 3, step 0.5
  - **Arc Size** — `id="spin-arc"`, range 10 to 90 (percentage of circumference), default 25, step 5. Label shows percentage.
  - **Direction** toggle — `id="spin-dir"`, a `<button>` that toggles between "Same Direction" and "Opposite" on click.

Generator function:
- Reads `dur`, `arcPct`, `opposite` (boolean)
- Computes per-circle: `dashLength = circumference * (arcPct / 100)`, `gapLength = circumference - dashLength`
  - Middle circle circumference: `2 * Math.PI * 9.5 ≈ 59.7`
  - Inner circle circumference: `2 * Math.PI * 4.5 ≈ 28.3`
- Returns SVG with:
  - Static outer circle
  - Middle circle: `stroke-dasharray="${dashMid} ${gapMid}"`, `<animateTransform type="rotate" from="0 15 15" to="360 15 15" dur="${dur}s">`
  - Inner circle: same, but if `opposite` → `from="360 15 15" to="0 15 15"`, else same direction
- All IDs prefixed with `spin-`

**Step 2: Verify, commit**

```bash
git add index.html
git commit -m "feat: section 4 — spin playground"
```

---

### Task 8: Section 4 — Playground 4: Draw-On

**Files:**
- Modify: `index.html`

**Step 1: Build the Draw-On playground**

Card containing:
- Title: "Draw-On" — "Shapes appear as if drawn by hand, one stroke at a time."
- SVG container: `id="drawon-svg-container"`
- Sliders:
  - **Duration** — `id="drawon-dur"`, 2 to 8, default 3, step 0.5
  - **Stagger** — `id="drawon-stagger"`, 0 to 1, default 0.15, step 0.05. Label shows value.

Generator function:
- Reads `dur`, `stagger`
- Circle circumferences: outer=91.1, mid=59.7, inner=28.3
- Computes keyTimes per circle using this formula:
  - `drawDuration = 0.25` (each circle takes 25% of total to draw)
  - Circle 0 (outer): starts at t=0, ends draw at t=drawDuration
  - Circle 1 (mid): starts at t=stagger, ends at t=stagger+drawDuration
  - Circle 2 (inner): starts at t=stagger*2, ends at t=stagger*2+drawDuration
  - Hold all visible until t=0.8, then erase to reset by t=1.0
  - Clamp all values to [0, 1], enforce monotonically increasing
- For each circle, `values` and `keyTimes` arrays must have equal length
- Example for outer (stagger=0.15): `keyTimes="0;0.25;0.8;1"` `values="91.1;0;0;91.1"`
- Example for mid (stagger=0.15): `keyTimes="0;0.15;0.4;0.8;1"` `values="59.7;59.7;0;0;59.7"`
- Example for inner (stagger=0.15): `keyTimes="0;0.15;0.3;0.55;0.8;1"` `values="28.3;28.3;28.3;0;0;28.3"`
- Runtime guard: after generating arrays, assert `keyTimes.length === values.length` and all keyTimes are monotonic. If not, fall back to defaults.
- All IDs prefixed with `drawon-`

**Step 2: Verify, commit**

```bash
git add index.html
git commit -m "feat: section 4 — draw-on playground"
```

---

### Task 9: Section 5 — "A Real Example" (Caglia Walkthrough)

**Files:**
- Modify: `index.html`

**Step 1: Build the real example section**

Layout:
- Before/after: static Caglia icon on the left, animated draw-on version on the right (pull directly from `icon-variants.html` Caglia Draw On variant)
- Use unique IDs: prefix all clipPath/def IDs with `caglia-` to avoid collisions
- Below: a "How it was built" walkthrough with 3-4 steps:
  1. "Exported the icon from Figma as SVG"
  2. "Identified the 4 shapes and their stroke lengths"
  3. "Asked Claude to animate them sequentially with draw-on"
  4. "Iterated: adjusted timing overlaps so shapes cascade smoothly"
- Below that: the actual prompt used (displayed as a styled blockquote/card, not a code block)

Use the Caglia SVGs from `icon-variants.html` (static + Draw On variants, viewBox 0 0 421 422).

**Step 2: Verify, commit**

```bash
git add index.html
git commit -m "feat: section 5 — caglia real example walkthrough"
```

---

### Task 10: Section 6 — "The AI Workflow"

**Files:**
- Modify: `index.html`

**Step 1: Build the AI workflow section**

Content blocks:
- **The Pipeline** — visual flow: Figma → Export SVG → Claude → Animated SVG → Preview
  - Render as a horizontal flow of labeled boxes/pills connected by arrows (can be simple CSS flexbox with → characters or SVG arrows)
- **Good Prompt vs Bad Prompt** — two cards side by side:
  - Bad: "Animate this SVG" — with a note: "Claude will guess. You'll iterate 3-4 times."
  - Good: "Read spiral.svg. It has an outer circle and two inner circles centered at (15,15). Add a gentle pulse to the inner circles — animate r ±1 unit, 3s duration, stagger 0.3s. Keep outer circle static. Loop forever." — with a note: "Specific elements, named technique, exact values, clear constraints."
- **Iteration Tips** — 3 brief bullet points from the markdown's "Iterating on Results" section
- All external links use `target="_blank" rel="noopener noreferrer"`

**Step 2: Verify, commit**

```bash
git add index.html
git commit -m "feat: section 6 — AI workflow and prompting guide"
```

---

### Task 11: Section 7 — "Take It Home" (Reference Card)

**Files:**
- Modify: `index.html`

**Step 1: Build the reference section**

- **Quick Reference Table** — styled as a clean card grid (not a raw HTML table). Each pattern gets a small card:
  - Pattern name (bold)
  - What it does (one line)
  - A tiny inline SVG showing the effect at ~30px

Patterns: Pulse, Expand, Draw-On, Rotate, Translate, Opacity Toggle
- All tiny SVG IDs prefixed with `ref-` to avoid collisions

- **Tools & Links** — brief list (all links `target="_blank" rel="noopener noreferrer"`):
  - Claude Code: `npm install -g @anthropic-ai/claude-code`
  - Figma MCP: one-line setup command
  - Claude.ai: upload SVGs directly in chat
  - LottieFiles: for cross-platform testing

- **Footer** — "Peak Perspective Media + Willa Creative" credit, subtle.

**Step 2: Verify, commit**

```bash
git add index.html
git commit -m "feat: section 7 — take it home reference card"
```

---

### Task 12: Polish + Push

**Files:**
- Modify: `index.html`

**Step 1: Cross-browser and mobile test**

Open in Safari and Chrome. Check:
- Sliders work and update animations
- All sliders have associated `<label>` elements and work via keyboard (tab + arrow keys)
- Layout is responsive (resize to phone width)
- SMIL animations play (note: SMIL doesn't work in IE but that's irrelevant)
- Scroll between sections is smooth
- Typography and spacing feel right
- Reduced-motion preference triggers JS `pauseAnimations()` and remains paused after slider-driven re-renders
- No duplicate SVG IDs in the DOM (check with: `document.querySelectorAll('[id]')` in console, verify no duplicates)

**Step 2: Fix any issues found**

**Step 3: Review staged files before push**

```bash
git status
```

Verify only expected files are staged. No `.env`, credentials, or `.claude/` directory.

**Step 4: Final commit and push**

```bash
git add index.html
git commit -m "polish: final cleanup and responsive fixes"
git push
```

**Step 5: Verify GitHub Pages is live**

```bash
OWNER=$(gh api user -q .login)
PAGES_URL="https://${OWNER}.github.io/ai-animation-lesson/"
echo "Site should be live at: ${PAGES_URL}"
# Wait ~60s for first deployment, then verify:
curl -s -o /dev/null -w "%{http_code}" "${PAGES_URL}"
```

Expected: HTTP 200, site is live.

---

## Notes for Implementer

- **SMIL update strategy:** Each playground generator returns a complete SVG string. `renderPlayground(name)` replaces the container's innerHTML with the new string. After insertion, optionally call `beginElement()` on animate nodes. Never use `setAttribute()` on existing animate elements — always re-render the full SVG.
- **Shared playground pattern:** All four playgrounds use the same `renderPlayground(name)` function + per-playground generator in a `generators` object. This avoids duplicated restart logic.
- **SVG ID collisions:** Every playground prefixes all IDs (`clipPath`, `animate`, elements) with the playground name. The reference card section uses `ref-` prefix. The Caglia section uses `caglia-` prefix.
- **Keep the existing files** (`icon-variants.html`, `spiral-preview.html`, `svg-animation-lesson.md`) in the repo as reference material. They don't need to be linked from the main page.
