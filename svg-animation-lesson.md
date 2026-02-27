# AI-Powered SVG Animation

**From Static Icons to Production Motion — Without After Effects**

*A practical guide to animating SVGs with Claude, SMIL, and Lottie*
*Willa Creative*

---

## Lesson 1: Understanding the SVG You're Animating

Before you ask AI to animate anything, you need to understand what you're working with. Every SVG is just XML — a text file with shapes described as code. When you open one, you'll see elements like `<circle>`, `<path>`, `<rect>`, and `<g>` (groups). Each one is a layer you can animate independently.

### Reading an SVG

Here's a real example — the spiral icon from our workshop, three concentric circles inside a 30×30 viewBox:

```xml
<svg width="30" height="30" viewBox="0 0 30 30" fill="none">
  <circle cx="15" cy="15" r="14.5" stroke="#F2F7F9"/>
  <circle cx="15" cy="15" r="9.5" stroke="#F2F7F9"/>
  <circle cx="15" cy="15" r="4.5" stroke="#F2F7F9"/>
</svg>
```

Reading this tells you everything you need:

- Three circles, all centered at (15,15), with radii 14.5, 9.5, and 4.5
- Stroke color is #F2F7F9 (near-white), no fill
- The viewBox is 30×30 units — this is the coordinate system animations operate in

The `viewBox` is critical. All animation values (positions, radii, translate distances) use these coordinates, not pixels. A circle at `r="14.5"` in a 30×30 viewBox fills nearly the entire icon regardless of whether it renders at 30px or 300px on screen.

### What to Look For

When you get an SVG from Figma, scan for these things before animating:

| Element | What It Means | Animation Potential |
|---------|---------------|---------------------|
| `<circle>` | Simple shape with cx, cy, r | Animate r (size), position, stroke-dasharray |
| `<path>` | Complex shape from d attribute | Draw-on via dasharray, translate, rotate |
| `<g>` | Group — acts like a folder | Transform the whole group at once |
| `<clipPath>` | Masks content outside boundary | Essential for expanding animations |
| `fill="none"` | Outline-only (stroked) | Required for draw-on effects |

> **TIP:** Export SVGs from Figma using right-click → Copy/Paste as → Copy as SVG. This gives you clean markup. The "Export" panel sometimes adds unnecessary wrappers.

---

## Lesson 2: The Animation Toolkit — SMIL

SMIL (Synchronized Multimedia Integration Language) is animation built directly into SVG. No JavaScript, no CSS, no external libraries. You add `<animate>` tags inside any SVG element and the browser does the rest.

This is the primary technique we use because it produces a single self-contained `.svg` file that works anywhere SVGs are supported.

### The Core Tags

#### `<animate>` — Animate Any Attribute

Changes a single attribute over time. This is the workhorse for most effects:

```xml
<circle cx="15" cy="15" r="4.5" stroke="#F2F7F9">
  <animate attributeName="r"
    values="4.5;9.5;14.5" dur="2s"
    repeatCount="indefinite"/>
</circle>
```

This grows the circle's radius from 4.5 → 9.5 → 14.5 over 2 seconds, looping forever.

#### `<animateTransform>` — Move, Rotate, Scale

Applies transform operations to an element or group:

```xml
<circle cx="15" cy="15" r="9.5" stroke="#F2F7F9"
        stroke-dasharray="15 45">
  <animateTransform attributeName="transform"
    type="rotate" from="0 15 15" to="360 15 15"
    dur="3s" repeatCount="indefinite"/>
</circle>
```

This rotates a dashed circle around center (15,15). The `stroke-dasharray` creates a partial arc that spins like a loading indicator.

### Key Attributes

| Attribute | Purpose | Example |
|-----------|---------|---------|
| `attributeName` | Which SVG attribute to animate | `r`, `cx`, `stroke-dashoffset`, `stroke-opacity` |
| `values` | Semicolon-separated keyframes | `"0;4.5;9.5;14.5"` (4 keyframes) |
| `keyTimes` | When each keyframe occurs (0–1) | `"0;0.25;0.75;1"` (hold in middle) |
| `dur` | Total animation duration | `"3s"`, `"500ms"` |
| `begin` | Delay before starting | `"1s"` (stagger waves) |
| `repeatCount` | How many times to loop | `"indefinite"`, `"3"`, `"1"` |
| `type` | Transform type | `translate`, `rotate`, `scale` |

### Controlling Timing with keyTimes

By default, values are evenly spaced. `keyTimes` lets you control exactly when each value is reached. Both lists must have the same number of entries.

```xml
values="1;1;0;0;1"     keyTimes="0;0.25;0.5;0.75;1"
```

This means: hold at 1 for the first quarter, transition to 0 by halfway, hold at 0 for the third quarter, then return to 1. This is how we built the Caglia sequential reveal — each shape's `stroke-opacity` toggled on/off at different points in the same 4-second cycle.

> **GOTCHA:** `keyTimes` must always start with 0 and end with 1, and must have exactly the same count as `values`. Mismatches silently break the animation.

---

## Lesson 3: Animation Patterns

These are the building blocks. Every complex animation is a combination of these patterns. Learn them individually, then layer them.

### Pattern 1: Expanding Radius (Hypnotic Zoom)

Grow circles outward from center. Use a `clipPath` to hide them past the outer boundary. Stack multiple waves with staggered `begin` times for continuous flow.

```xml
<defs><clipPath id="c"><circle cx="15" cy="15" r="14"/></clipPath></defs>
<circle cx="15" cy="15" r="14.5" stroke="#F2F7F9"/>
<g clip-path="url(#c)">
  <circle cx="15" cy="15" stroke="#F2F7F9" fill="none">
    <animate attributeName="r" values="0;4.5;9.5;14.5"
      dur="3s" repeatCount="indefinite"/>
  </circle>
  <!-- Wave 2: begin="1s" / Wave 3: begin="2s" -->
</g>
```

ClipPath radius (14) is slightly less than the outer ring (14.5) so circles vanish behind the border. Three waves at 1s offsets = seamless loop.

> **TIP:** Wave count = duration ÷ offset. 3s dur ÷ 1s offset = 3 waves for gapless looping.

### Pattern 2: Draw-On (Stroke Reveal)

Makes a shape appear as if drawn by hand. Set `stroke-dasharray` to the path's total length, then animate `stroke-dashoffset` from full to zero.

```xml
<!-- Circumference = 2πr = 2 × 3.14 × 14.5 ≈ 91.1 -->
<circle r="14.5" stroke="#F2F7F9"
  stroke-dasharray="91.1" stroke-dashoffset="91.1">
  <animate attributeName="stroke-dashoffset"
    values="91.1;0" dur="2s" repeatCount="indefinite"/>
</circle>
```

For paths, estimate the length or use JS: `document.querySelector("path").getTotalLength()`. Rounding to nearest 10 works fine.

> **GOTCHA:** `stroke-dasharray` only works on stroked elements (`fill="none"` with a stroke). Won't apply to filled shapes.

### Pattern 3: Translate (Sliding Motion)

Move elements left/right or up/down. Used for wave rolling, tide drift, and sway effects.

```xml
<animateTransform attributeName="transform" type="translate"
  values="0,0;-3,0;0,0;3,0;0,0" dur="4s"
  repeatCount="indefinite"/>
```

Values are x,y pairs. Opposite directions on adjacent elements (top: -3 then +3, middle: +3 then -3) create organic rolling motion. Always wrap in `clipPath`.

### Pattern 4: Rotate

Spin an element. Values: `"angle centerX centerY"`. Always rotate around icon center, not 0,0.

```xml
<animateTransform type="rotate"
  from="0 15 15" to="360 15 15" dur="3s"
  repeatCount="indefinite"/>
```

Combine with `stroke-dasharray` for partial arcs that spin. Counter-rotate nested elements for visual complexity.

### Pattern 5: Pulse / Breathe

Gently oscillate a value. Three keyframes: start, peak, start.

```xml
<animate attributeName="r" values="9.5;12;9.5"
  dur="3s" repeatCount="indefinite"/>
```

Apply to radius, translate, or stroke-width. Stagger `begin` times so nested elements breathe in sequence.

---

## Lesson 4: Composition — Combining Patterns

The real power comes from layering patterns. Here's how we composed the workshop variants.

### Waves — "Rolling" Variant

Three wave paths using horizontal translate with opposite directions and staggered starts:

- **Top wave:** translate -3 → +3 (left-to-right)
- **Middle wave:** translate +3 → -3 (opposite direction = rolling feel)
- **Bottom wave:** same as top, `begin="0.3s"` (cascade lag)

Opposing directions on adjacent waves = organic ocean motion. Without direction reversal, all waves slide like a rigid sheet.

### Spiral — "Spin" Variant

- Inner circles use `stroke-dasharray` for partial arcs (not full circles)
- Each arc gets `animateTransform rotate` in opposite directions
- Outer circle stays static as anchor

A full circle with `stroke-dasharray="15 45"` becomes a short arc. Spinning it creates a radar/loading effect. Counter-rotating doubles complexity with zero extra code.

### Caglia — "Draw On" Variant

Four shapes drawn sequentially with staggered `stroke-dashoffset`, orchestrated via `keyTimes`:

- **Outer circle** (≈1320 circumference): draws 0–25% of timeline
- **Top circle** (≈924): draws 15–40%
- **Bottom circle** (≈519): draws 30–55%
- **Middle oval** (≈1100): draws 45–70%
- All hold visible 70–80%, then erase to reset

Overlapping percentages (15–40, 30–55) mean the next shape starts before the previous finishes, creating smooth cascade.

> **TIP:** Sketch multi-element timelines on paper first. Map each element to a % range, overlap 10–15% for smooth transitions, and leave a "hold" period where everything is visible before resetting.

---

## Lesson 5: Using AI to Generate Animations

Now that you understand the mechanics, you can direct AI effectively. The key is specificity — not "animate this" but describing the structure, effect, and constraints.

### Anatomy of a Good Prompt

A strong animation prompt has four parts:

1. **Source:** Tell Claude what SVG to read and describe its structure
2. **Effect:** Name the motion pattern (draw-on, pulse, translate, rotate)
3. **Timing:** Duration, delays, easing, loop behavior
4. **Constraints:** What should NOT change (outer ring stays static, preserve colors)

#### Weak Prompt

```
"Animate this SVG"
```

*Claude will guess. You'll iterate 3–4 times.*

#### Strong Prompt

```
"Read waves.svg. It has an outer circle and three wave
paths inside a clipPath. Animate each wave with
horizontal translate — top/bottom move -3 to +3,
middle moves opposite. Stagger 0.3s. Keep outer
circle static. 4s duration, loop. Output single SVG."
```

References specific elements, names the technique, specifies direction/magnitude, defines constraints. Works first attempt.

### Prompt Templates

**Simple Icon:**
```
"Read [file]. Animate with [pattern] effect.
[X]s duration, loop. Keep [element] static."
```

**Sequential Multi-Element:**
```
"Read [file]. Contains [describe elements].
Animate sequentially with stroke-dashoffset draw-on.
[element 1] first, [element 2] after [delay].
Total cycle [X]s. Hold all visible [X]s before reset."
```

**Figma Pipeline:**
```
"Using Figma MCP, get design at [URL]. Export SVG.
Animate with [effect]. Save SVG + HTML preview."
```

**Batch Processing:**
```
"Read all SVGs in folder. For each, create variant
with consistent [pattern]. Same timing: [X]s, loop.
Save as [name]-animated.svg"
```

### Iterating on Results

When the first output isn't right, be specific:

- "Draw-on is too fast — increase dur from 2s to 4s" (not "make it slower")
- "Waves overflow clipPath — reduce translate from 5 to 3 units"
- "Stagger feels jerky — reduce begin offset from 1s to 0.3s"
- "Remove opacity animation, use position/size changes only"

> **GOTCHA:** Claude sometimes adds opacity fades by default. If you want hard cuts or clip-based hiding, say so up front.

---

## Lesson 6: Setup & Tools

### Claude Code (Terminal)

Most powerful option. Reads files, writes SVGs directly, creates HTML previews.

**Install:**
```bash
npm install -g @anthropic-ai/claude-code
```

**Daily Usage:**
```bash
cd ~/Desktop/my-icons
claude
> "List all SVG files here"
> "Read icon.svg and animate it with a pulse effect"
```

**Key Commands:**

| Command | What It Does |
|---------|-------------|
| `/mcp` | Manage MCP server connections (Figma, LottieFiles) |
| `/clear` | Clear conversation context |
| `Ctrl+C` | Cancel current operation |
| `exit` | Leave Claude Code session |

### Figma MCP Server

Connects Claude to Figma files. Reference designs by URL instead of manual SVG export.

```bash
claude mcp add --scope user --transport http figma https://mcp.figma.com/mcp
# Restart Claude Code, then: /mcp → figma → Authenticate
```

> **GOTCHA:** Figma MCP requires file access. For restricted client files, manually export (right-click → Copy as SVG) and paste into a file for Claude to read.

### HeroUI Studio (heroui.studio)

A visual AI canvas purpose-built for SVG animation — the most designer-friendly option. No terminal, no code.

**How it works:**

HeroUI Studio uses a node-based canvas (think: visual programming). You connect nodes to build an animation pipeline:

1. **SVG Source** — Upload an SVG, generate one from a prompt, or paste markup
2. **Text Prompt** — Describe what you want in plain English ("make the icon pulse", "add a hover scale effect")
3. **AI Generation** — The AI produces an animated SVG with a live preview
4. **Animation Prompt** — Fine-tune with presets and controls:
   - Presets: Entrance (Fade In), Hover (Scale), Loading (Spin), Attention (Bounce), Exit (Slide Out)
   - Controls: Duration, Delay, Easing curve, Loop toggle

**Iterative chaining:** Connect one generation's output to a new prompt node to refine. Example flow: upload icon → "make it bounce" → "now add a glow on hover" → "slow the bounce to 3s". Each step builds on the last.

**Credits:** Each generation/regeneration costs credits (10 per run). Free tier available.

**Best for:** Designers who want to experiment with animation without touching code. Great for quick prototyping and exploring effects before committing to a production approach.

> **TIP:** HeroUI Studio is ideal for exploring "what if" — try 5 different effects on the same icon in minutes, then recreate the winner in SMIL for production.

### Claude.ai Chat

Upload `.svg` files directly in chat and describe what you want. Simpler than Claude Code but no batch processing or Figma MCP connection.

### Preview & QA

- Ask Claude to create HTML preview files alongside every animation
- Upload `.json` Lottie files to lottiefiles.com for cross-platform testing
- Test at actual render size — effects visible at 200px may be invisible at 24px

---

## Lesson 7: Output Formats

Choosing the right format depends on where the animation is going and what it needs to do.

| Format | Best For | Pros | Cons | File |
|--------|----------|------|------|------|
| **SMIL SVG** | Icons, web UI, Shopify | Self-contained, tiny, no JS | No iOS native, limited easing | `.svg` |
| **CSS SVG** | Hover states, scroll triggers | CSS easing, familiar syntax | Needs HTML wrapper or inline | `.svg + CSS` |
| **Lottie JSON** | Apps, cross-platform, complex | iOS/Android native, rich effects | Requires player library | `.json` |
| **dotLottie** | Production Lottie | Compressed, multi-animation | Newer format, less tooling | `.lottie` |

### When to Use What

- **Shopify icons, web badges, simple loops** → SMIL SVG (just drop the `.svg` file in)
- **Interactive hover/scroll animations** → CSS SVG with Liquid wrapper
- **Mobile apps, complex multi-layer** → Lottie JSON
- **Production Lottie deployment** → dotLottie via LottieFiles

### Embedding on Shopify

**SMIL SVG — Direct Inline:**
```liquid
{% render 'icon-waves-animated' %}
```
Where `icon-waves-animated.liquid` is a snippet containing the raw SVG markup.

**Lottie JSON — With Player:**
```html
<script src="https://unpkg.com/@lottiefiles/lottie-player@latest/dist/lottie-player.js"></script>
<lottie-player src="/path/to/anim.json"
  background="transparent" speed="1"
  loop autoplay style="width:200px;height:200px">
</lottie-player>
```

---

## Lesson 8: Hands-On Exercise

Pick one of these exercises to practice. Each builds on the patterns from this lesson.

### Exercise A: Animate Your Own Icon (Beginner)

1. Export any icon from Figma as SVG (right-click → Copy as SVG)
2. Save it as a `.svg` file on your desktop
3. Open Claude Code (or upload to Claude.ai chat)
4. Prompt: *"Read [file]. Add a gentle pulse animation to the main shape. 3s duration, loop forever. Keep the outer border static."*
5. Ask Claude to create an HTML preview. Open it in your browser.

### Exercise B: Create 3 Variants (Intermediate)

1. Take the same icon from Exercise A
2. Ask Claude for three distinct animation variants:
   - Variant 1: Draw-on (stroke reveal)
   - Variant 2: Rotate with dashed stroke
   - Variant 3: Sequential element fade (if multi-element)
3. Ask Claude to build a single HTML showcase page displaying all three side-by-side

### Exercise C: Full Pipeline (Advanced)

1. Connect Figma MCP to Claude Code
2. Pick a frame from a Figma file you have access to
3. Prompt Claude to pull the design via MCP, export SVG, and create an animated Lottie JSON
4. Preview the Lottie in an HTML file and upload to lottiefiles.com
5. Embed the Lottie on a Shopify test page using the `lottie-player` web component

---

## Quick Reference Card

| Pattern | Key Attribute + Values |
|---------|----------------------|
| Expand / Shrink | `attributeName="r" values="4;14;4"` |
| Draw-On | `attributeName="stroke-dashoffset" values="91;0"` |
| Slide | `type="translate" values="0,0;-3,0;0,0;3,0;0,0"` |
| Rotate | `type="rotate" from="0 15 15" to="360 15 15"` |
| Pulse | `attributeName="r" values="9.5;12;9.5"` |
| Opacity Toggle | `attributeName="stroke-opacity" values="1;0;1" keyTimes="0;0.5;1"` |
| Stagger | `begin="0s"` / `begin="0.3s"` / `begin="0.6s"` |
| ClipPath Mask | `<clipPath id="c"><circle r="14"/></clipPath>` + `clip-path="url(#c)"` |

---

*Remember: every one of these animated SVGs is just a text file. You can open it in any code editor, tweak a number, and refresh the browser to see the change. That's the entire workflow.*
