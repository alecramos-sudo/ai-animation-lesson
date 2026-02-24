# Interactive SVG Animation Workshop — Design

## Context
A 1-hour workshop for 4-5 designers. Hosted on GitHub Pages as a single public URL they can follow along on devices and keep as a reference afterward.

## Structure — Single Scrollable Page, Two Acts

### Act 1: AI for Designers (presentation backdrop)
Facilitator-led discussion. The page provides a clean visual backdrop with big text and whitespace.

- **Section 1: "What AI Can Do Today"** — landscape overview (Claude, ChatGPT, Midjourney, etc.), free vs paid, capabilities and limitations.
- **Section 2: "How It's Changing Design Work"** — key points about where things are headed. Brief, visual.
- **Transition:** "Let's get hands-on with one specific thing AI is great at..."

### Act 2: SVG Animation with AI (interactive)

- **Section 3: "Meet the SVG"** — show a static spiral icon. Visual anatomy: labeled parts (viewBox, circle, radius, stroke). No code visible.
- **Section 4: "One Icon, Many Motions"** — the spiral icon with 4 interactive playgrounds:
  - Pulse (sliders: duration, intensity)
  - Expanding radius / hypnotic zoom (sliders: duration, wave count)
  - Rotate / spin (sliders: duration, dash length, direction)
  - Draw-on / stroke reveal (sliders: duration, stagger)
  - Each playground: live SVG preview + labeled sliders. No code.
- **Section 5: "A Real Example"** — walkthrough of a complex animation (Caglia sequential draw-on). How it was built with AI. The prompt that produced it. Before/after comparison.
- **Section 6: "The AI Workflow"** — Figma → Claude → animated SVG pipeline. Figma MCP, good vs bad prompts, iteration examples.
- **Section 7: "Take It Home"** — quick reference card (animation patterns table), tool links, next steps.

## Visual Style
- Light, warm, educational — background ~#FAFAF8, dark text
- Clean sans-serif (Inter or system font stack)
- Generous whitespace, large section headers
- SVG demos on subtle cards (light border or shadow)
- Custom-styled slider controls (not raw browser defaults)
- Responsive — works on phones and laptops

## Technical Approach
- Single `index.html`, no build step, zero dependencies
- CSS custom properties for theming
- Vanilla JS: sliders bind to SVG attributes via `setAttribute()`
- Inline SMIL SVG for all animations
- GitHub Pages for hosting (public repo)

## Out of Scope
- No database, backend, or auth
- No code editors or visible code
- No poll/quiz widgets
- No Lottie/JSON output — all inline SMIL SVG
- No build tools or frameworks
