---
name: paper-to-artifact
description: >
  Converts any research paper, academic concept, or complex technical document into a
  fully interactive educational React artifact — with plain-language explanations,
  analogies, clickable demos, and live visualizations. Use this skill whenever the user
  wants to "make a paper interactive", "explain a paper visually", "turn research into
  a tutorial", "build an explainer for [paper/concept/algorithm]", or says things like
  "help me understand this paper", "make this paper easy to understand", or uploads a
  PDF and asks for an interactive version. Also trigger for requests to explain any
  complex technical concept (attention mechanisms, neural networks, algorithms, protocols,
  statistical methods, etc.) in an interactive or visual way. Always use this skill for
  such requests — it will produce far better results than a generic approach.
---

# Paper → Interactive Artifact

Convert any research paper or complex technical concept into a polished, multi-tab
interactive React artifact that a non-expert can understand and explore.

## Quick-start: what to do first

1. **Extract the paper's core concepts** (see Phase 1 below)
2. **Design the tab structure** — one tab per major concept (see Phase 2)
3. **Build the artifact** following the code conventions in `references/code-conventions.md`
4. **Read `references/content-strategy.md`** for how to write explanations that actually land

---

## Phase 1 — Analyze the Paper

Before writing any code, read or analyze the paper and extract:

### 1.1 Core concepts (aim for 4–7)
Each concept becomes one tab. Ask: "what must someone understand before the next concept makes sense?" Order them as a learning path.

**Example — "Attention Is All You Need":**
- Overview (why this paper matters)
- Queries, Keys & Values (the vocabulary)
- Self-Attention (the core mechanism)
- Multi-Head Attention (scaling the mechanism)
- Positional Encoding (solving word order)
- Full Architecture (putting it together)

### 1.2 Key formulas or algorithms
Identify the 1–3 central equations. These need to be displayed *and* explained in plain English, not just shown.

### 1.3 The "aha moment" for each concept
What analogy or example makes each concept click? This is the most valuable thing you'll write. See `references/content-strategy.md` for analogy patterns.

### 1.4 What can be made interactive?
For each concept, identify if it has:
- **Parameters to explore** → slider / stepper
- **Data that changes with input** → live heatmap / chart
- **A process with discrete steps** → step-through cards
- **A structure to explore** → clickable diagram
- **Comparative options** → tabbed selector

---

## Phase 2 — Tab Structure Design

### Tab types (choose per concept)

| Concept type | Recommended tab pattern |
|---|---|
| Historical context / motivation | Expandable timeline cards |
| Vocabulary / definitions | Step-through cards with analogy box |
| Core algorithm with formula | Formula card + interactive demo |
| Mechanism with parameters | Live visualization with slider/selector |
| Architecture / structure | Clickable block diagram with detail panel |
| Comparative analysis | Toggle between options, shared canvas |

### Required elements per tab
Every tab must include:
1. **Tab intro banner** — 2–3 sentences: what you'll learn, why it matters
2. **Plain-language explanation** — prose paragraphs, no bullet soup
3. **An analogy** — see content strategy guide
4. **At least one interactive element** — the user must be able to *do* something
5. **A "key insight" callout** — the one thing to remember from this tab

### Navigation
- Prev / Next buttons at bottom of each tab
- Progress indicator (e.g., "3 / 6") in header
- Dot pagination in footer

---

## Phase 3 — Build the Artifact

Read `references/code-conventions.md` before writing any code.

### Architecture checklist
- [ ] Single `.jsx` file, default export `App`
- [ ] Google Fonts injected via `<style>` tag (inject once with `data-*` attribute guard)
- [ ] Design tokens object `T` at top (colors, never hardcode hex values in JSX)
- [ ] Reusable primitives: `Card`, `Callout`, `Analogy`, `SectionHeading`, `SubHeading`, `Prose`
- [ ] `useState` for all interactive state; `useEffect` for derived state on input change
- [ ] `key={activeTab}` on tab content div to trigger `fade-in` animation on tab switch
- [ ] Sticky header with tab navigation
- [ ] Prev/Next navigation + dot footer
- [ ] All interactive demos generate data deterministically (seed-based) so they don't flicker on re-render

### Interactive element patterns

**Heatmap (e.g., attention weights):**
```jsx
// Generate weights deterministically
function generateWeights(tokens, headIdx = 0) {
  const seeds = [0.9, 0.3, 0.7, 0.5, 0.2, 0.8];
  return tokens.map((_, i) => {
    let row = tokens.map((_, j) => {
      const d = Math.abs(i - j);
      return Math.exp(-d * (0.4 + headIdx * 0.15)) * seeds[(i+j+headIdx*3) % seeds.length]
        + (i === j ? 0.6 : 0);
    });
    const s = row.reduce((a,b) => a+b, 0);
    return row.map(v => v/s);
  });
}

// Color mapping: low → light, high → saturated
function wColor(w) {
  const r = Math.round(37 + (239-37)*(1-w));
  const g = Math.round(99 + (246-99)*(1-w));
  const b = Math.round(235 + (255-235)*(1-w));
  return `rgb(${r},${g},${b})`;
}
```

**Slider visualization:**
```jsx
const [pos, setPos] = useState(0);
// Always show current value next to slider
// Use accentColor on input[type=range]
// Animate bars with transition: "height 0.25s ease"
```

**Expandable cards:**
```jsx
const [open, setOpen] = useState(null);
// onClick={() => setOpen(open === i ? null : i)}
// Wrap expanded content in <div className="fade-in">
// Show "▼ read more" / "▲ less" affordance
```

**Clickable diagram:**
```jsx
const [sel, setSel] = useState(null);
// Two-column layout: diagram list left, detail panel right
// Detail panel: position: "sticky", top: 20
// Empty state: "👈 Click any block" prompt
```

---

## Phase 4 — Content Quality Checklist

Before finalizing:

- [ ] Every formula is explained in plain English *below* it
- [ ] Every tab has at least one analogy (yellow callout box)
- [ ] No tab uses bullet points as the primary explanation mode — use prose paragraphs
- [ ] The "why was this designed this way?" question is answered for key design choices
- [ ] A non-expert reader (e.g., a curious doctor or lawyer, not a ML engineer) could follow every tab
- [ ] Interactive elements have step labels: "STEP 1 — ...", "STEP 2 — ..."
- [ ] Color legend provided for any heatmap or color-coded visualization

---

## Design principles

- **White/cream background** — readable, non-fatiguing
- **Serif font for headings** (Lora, Playfair, etc.) — signals "this is worth reading"
- **Mono font for formulas and labels** — technical precision
- **Sans font for body prose** — comfortable reading at length
- **One accent color per tab** — use it consistently for that tab's headings, borders, highlights
- **Yellow callout = analogy** — users learn to look for these
- **Blue callout = key insight** — used at the end of sections

See `references/code-conventions.md` for the full design token system and reusable component library.

---

## Reference files

- `references/content-strategy.md` — How to write explanations, analogies, and plain-language descriptions
- `references/code-conventions.md` — Full component library, design tokens, animation patterns

---

## Output

A single `.jsx` file that:
1. Runs as a React artifact with no external dependencies beyond Google Fonts
2. Has 4–7 tabs covering the paper's core concepts in learning order
3. Includes at least one interactive demo (heatmap, slider, clickable diagram, etc.)
4. Can be understood by a curious non-expert
5. Is visually polished and consistent
