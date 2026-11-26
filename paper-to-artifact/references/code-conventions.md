# Code Conventions — Paper to Interactive Artifact

Full component library, design tokens, and patterns extracted from the
"Attention Is All You Need" interactive explainer.

---

## File structure

Single `.jsx` file. Top-to-bottom order:
1. Font injection (guarded with `data-*` attribute)
2. CSS animations in the same style tag
3. Design tokens object `T`
4. Font constants `serif`, `sans`, `mono`
5. Utility functions (weight generators, color interpolators)
6. Shared UI primitives (Card, Callout, Analogy, etc.)
7. Tab config array `TABS`
8. Tab components (one function per tab)
9. `TAB_COMPONENTS` map
10. `App` default export

---

## Font injection (copy exactly)

```jsx
const styleTag = document.createElement("style");
styleTag.textContent = `
  @import url('https://fonts.googleapis.com/css2?family=Lora:ital,wght@0,400;0,600;1,400&family=DM+Sans:wght@400;500;600&family=DM+Mono:wght@400;500&display=swap');
  * { box-sizing: border-box; margin: 0; padding: 0; }
  button { font-family: inherit; cursor: pointer; }
  .fade-in { animation: fadeIn 0.35s ease forwards; }
  @keyframes fadeIn { from { opacity: 0; transform: translateY(8px); } to { opacity: 1; transform: translateY(0); } }
  .hover-lift:hover { transform: translateY(-2px); box-shadow: 0 6px 20px rgba(0,0,0,0.1) !important; }
`;
if (!document.head.querySelector("[data-pfa-style]")) {
  styleTag.setAttribute("data-pfa-style", "1");
  document.head.appendChild(styleTag);
}

const serif = "'Lora', Georgia, serif";
const sans  = "'DM Sans', system-ui, sans-serif";
const mono  = "'DM Mono', monospace";
```

**Font pairings that work well (pick one set):**
- `Lora` (headings) + `DM Sans` (body) + `DM Mono` (code) ← default
- `Playfair Display` (headings) + `Source Sans 3` (body) + `JetBrains Mono` (code) ← elegant
- `Fraunces` (headings) + `Nunito` (body) + `Fira Code` (code) ← playful

---

## Design tokens

```jsx
const T = {
  // Backgrounds
  cream:    "#faf8f4",   // page background
  white:    "#ffffff",   // card background
  faint:    "#f1f5f9",   // subtle inset background

  // Text
  ink:      "#1c1917",   // headings, primary text
  inkLight: "#44403c",   // body text
  muted:    "#78716c",   // secondary text, labels

  // Borders
  border:   "#e7e4df",   // default border
  borderDk: "#d6d3ce",   // slightly darker border

  // Accent palette (assign one per tab concept)
  blue:     "#2563eb",  blueL:   "#eff6ff",  blueMid: "#bfdbfe",
  teal:     "#0d9488",  tealL:   "#f0fdfa",
  orange:   "#ea580c",  orangeL: "#fff7ed",
  purple:   "#7c3aed",  purpleL: "#f5f3ff",
  green:    "#16a34a",  greenL:  "#f0fdf4",
  red:      "#dc2626",  redL:    "#fef2f2",
  yellow:   "#ca8a04",  yellowL: "#fefce8",
};
```

**Tab color assignment pattern:**
- Tab 1 (overview/motivation): `blue`
- Tab 2 (core vocabulary): `teal`
- Tab 3 (main mechanism): `orange`
- Tab 4 (scaling/variants): `purple`
- Tab 5 (technical detail): `green`
- Tab 6 (architecture/full picture): `red`

---

## Shared UI primitives

Copy these exactly. They ensure visual consistency across all tabs.

```jsx
function Card({ children, style = {} }) {
  return (
    <div style={{
      background: T.white, border: `1px solid ${T.border}`,
      borderRadius: 14, padding: "22px 24px", ...style
    }}>
      {children}
    </div>
  );
}

// icon: emoji, color: accent hex, bg: light bg hex
function Callout({ icon, color = T.blue, bg, children, style = {} }) {
  return (
    <div style={{
      background: bg || T.blueL, border: `1px solid ${color}30`,
      borderLeft: `4px solid ${color}`, borderRadius: "0 12px 12px 0",
      padding: "14px 18px", display: "flex", gap: 12, alignItems: "flex-start", ...style
    }}>
      {icon && <span style={{ fontSize: 20, flexShrink: 0, marginTop: 1 }}>{icon}</span>}
      <div style={{ fontSize: 13.5, color: T.inkLight, lineHeight: 1.75, fontFamily: sans }}>
        {children}
      </div>
    </div>
  );
}

// Yellow box — always used for analogies
function Analogy({ children }) {
  return (
    <div style={{
      background: T.yellowL, border: `1px solid #fde68a`,
      borderRadius: 10, padding: "12px 16px",
      fontSize: 13.5, color: T.inkLight, lineHeight: 1.75, fontFamily: sans
    }}>
      <span style={{ fontFamily: mono, fontSize: 10, letterSpacing: 1, color: T.yellow, marginRight: 8 }}>
        💡 ANALOGY
      </span>
      {children}
    </div>
  );
}

function SectionHeading({ label, color = T.blue }) {
  return (
    <div style={{ display: "flex", alignItems: "center", gap: 10, marginBottom: 18 }}>
      <div style={{ width: 4, height: 24, background: color, borderRadius: 2 }} />
      <span style={{ fontFamily: serif, fontSize: 20, color: T.ink, fontWeight: 600 }}>{label}</span>
    </div>
  );
}

// Mono label above sections — uppercase, spaced
function SubHeading({ children, color = T.blue }) {
  return (
    <div style={{
      fontFamily: mono, fontSize: 10.5, letterSpacing: 1.5,
      textTransform: "uppercase", color, marginBottom: 8
    }}>
      {children}
    </div>
  );
}

// Body paragraph — always use this instead of raw <p>
function Prose({ children, style = {} }) {
  return (
    <p style={{ fontSize: 14, color: T.inkLight, lineHeight: 1.85, fontFamily: sans, ...style }}>
      {children}
    </p>
  );
}
```

---

## Tab config array

```jsx
const TABS = [
  {
    id: "tab-id",
    icon: "📜",          // emoji
    label: "Tab Name",   // shown in nav button
    short: "tagline",    // shown under icon in nav
    color: T.blue,       // accent color for this tab
    intro: "2–4 sentence intro shown in the banner above tab content.",
  },
  // ...
];
```

---

## Tab intro banner (use verbatim)

```jsx
// In App component, before tab content:
const tab = TABS.find(t => t.id === active);

<div style={{
  background: T.white, border: `1.5px solid ${tab.color}30`,
  borderLeft: `5px solid ${tab.color}`,
  borderRadius: "0 14px 14px 0", padding: "18px 22px", marginBottom: 28,
  boxShadow: `0 2px 12px ${tab.color}12`,
}}>
  <div style={{ fontFamily: mono, fontSize: 10, color: tab.color, letterSpacing: 1.5, marginBottom: 6 }}>
    {tab.icon} {tab.short.toUpperCase()}
  </div>
  <div style={{ fontFamily: serif, fontSize: 20, color: T.ink, fontWeight: 600, marginBottom: 8 }}>
    {tab.label}
  </div>
  <Prose style={{ fontSize: 14 }}>{tab.intro}</Prose>
</div>
```

---

## App shell (header + content + footer)

```jsx
export default function App() {
  const [active, setActive] = useState(TABS[0].id);
  const tab = TABS.find(t => t.id === active);
  const activeIdx = TABS.findIndex(t => t.id === active);

  return (
    <div style={{ minHeight: "100vh", background: T.cream, fontFamily: sans, display: "flex", flexDirection: "column" }}>

      {/* Sticky header */}
      <div style={{
        background: T.white, borderBottom: `1px solid ${T.border}`,
        position: "sticky", top: 0, zIndex: 100,
        boxShadow: "0 2px 10px rgba(0,0,0,0.06)",
      }}>
        <div style={{ maxWidth: 980, margin: "0 auto", padding: "14px 24px" }}>
          <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 14 }}>
            <div>
              <div style={{ fontFamily: serif, fontSize: 17, color: T.ink, fontWeight: 600 }}>
                [Paper Title]
              </div>
              <div style={{ fontFamily: mono, fontSize: 10, color: T.muted, marginTop: 2, letterSpacing: 0.5 }}>
                [Authors · Institution · Venue Year] · Interactive Explainer
              </div>
            </div>
            <div style={{ background: T.blueL, border: `1px solid ${T.blueMid}`, borderRadius: 8, padding: "5px 12px", fontFamily: mono, fontSize: 10, color: T.blue }}>
              {activeIdx + 1} / {TABS.length}
            </div>
          </div>
          <div style={{ display: "flex", gap: 4, overflowX: "auto", paddingBottom: 2 }}>
            {TABS.map(t => (
              <button key={t.id} onClick={() => setActive(t.id)} style={{
                background: active === t.id ? t.color : "transparent",
                border: `1.5px solid ${active === t.id ? t.color : T.border}`,
                borderRadius: 8, padding: "7px 14px", whiteSpace: "nowrap",
                color: active === t.id ? "#fff" : T.muted,
                fontSize: 13, transition: "all 0.18s",
                fontWeight: active === t.id ? 600 : 400,
              }}>
                {t.icon} {t.label}
              </button>
            ))}
          </div>
        </div>
      </div>

      {/* Content area */}
      <div style={{ flex: 1, maxWidth: 980, margin: "0 auto", width: "100%", padding: "28px 24px" }}>
        {/* Tab intro banner goes here */}
        <div className="fade-in" key={active} style={{
          background: T.white, border: `1px solid ${T.border}`,
          borderRadius: 16, padding: "28px 30px",
          boxShadow: "0 2px 16px rgba(0,0,0,0.05)",
        }}>
          {TAB_COMPONENTS[active]}
        </div>

        {/* Prev / Next navigation */}
        <div style={{ display: "flex", justifyContent: "space-between", marginTop: 24 }}>
          {activeIdx > 0 ? (
            <button onClick={() => setActive(TABS[activeIdx - 1].id)} style={{
              background: T.white, border: `1.5px solid ${T.border}`,
              borderRadius: 10, padding: "10px 18px", fontSize: 13, color: T.inkLight, fontFamily: sans,
            }}>
              ← {TABS[activeIdx - 1].label}
            </button>
          ) : <div />}
          {activeIdx < TABS.length - 1 ? (
            <button onClick={() => setActive(TABS[activeIdx + 1].id)} style={{
              background: tab.color, border: "none",
              borderRadius: 10, padding: "10px 18px", fontSize: 13, color: "#fff",
              fontFamily: sans, fontWeight: 600,
            }}>
              {TABS[activeIdx + 1].label} →
            </button>
          ) : (
            <button onClick={() => setActive(TABS[0].id)} style={{
              background: tab.color, border: "none",
              borderRadius: 10, padding: "10px 18px", fontSize: 13, color: "#fff", fontFamily: sans, fontWeight: 600,
            }}>↩ Back to start</button>
          )}
        </div>
      </div>

      {/* Dot footer */}
      <div style={{ background: T.white, borderTop: `1px solid ${T.border}`, padding: "14px 24px", display: "flex", justifyContent: "center", gap: 8, alignItems: "center" }}>
        {TABS.map(t => (
          <button key={t.id} onClick={() => setActive(t.id)} title={t.label} style={{
            width: active === t.id ? 32 : 8, height: 8, borderRadius: 4,
            background: active === t.id ? t.color : T.borderDk,
            border: "none", padding: 0, transition: "all 0.25s",
          }} />
        ))}
      </div>
    </div>
  );
}
```

---

## Common interactive patterns

### Expandable cards
```jsx
const [open, setOpen] = useState(null);

{items.map((item, i) => (
  <div key={i} className="hover-lift" onClick={() => setOpen(open === i ? null : i)} style={{
    background: open === i ? accentBg : T.white,
    border: `1.5px solid ${open === i ? accentColor : T.border}`,
    borderRadius: 12, padding: "16px 20px", cursor: "pointer", transition: "all 0.22s",
  }}>
    {/* Always show summary */}
    <div>{item.summary}</div>
    {/* Show affordance */}
    <div style={{ fontFamily: mono, fontSize: 10, color: accentColor, marginTop: 8 }}>
      {open === i ? "▲ less" : "▼ read more"}
    </div>
    {/* Expanded detail */}
    {open === i && (
      <div className="fade-in" style={{ marginTop: 14, paddingTop: 14, borderTop: `1px solid ${accentColor}30` }}>
        {item.detail}
      </div>
    )}
  </div>
))}
```

### Step-through cards (for multi-part concepts)
```jsx
const [step, setStep] = useState(0);
const steps = [ { key, label, icon, color, bg, plain, analogy, formula, formulaNote } ];
const s = steps[step];

{/* Selector */}
<div style={{ display: "grid", gridTemplateColumns: `repeat(${steps.length}, 1fr)`, gap: 8 }}>
  {steps.map((st, i) => (
    <button key={i} onClick={() => setStep(i)} className="hover-lift" style={{
      background: step === i ? st.bg : T.white,
      border: `2px solid ${step === i ? st.color : T.border}`,
      borderRadius: 12, padding: "12px 10px", transition: "all 0.2s",
    }}>
      <div style={{ fontSize: 22 }}>{st.icon}</div>
      <div style={{ fontFamily: mono, fontSize: 11, color: step === i ? st.color : T.muted }}>{st.key}</div>
    </button>
  ))}
</div>

{/* Detail (key={step} triggers fade-in on change) */}
<div className="fade-in" key={step} style={{ background: s.bg, border: `1.5px solid ${s.color}40`, borderRadius: 14, padding: "24px 26px" }}>
  ...
</div>
```

### Clickable block diagram
```jsx
const [sel, setSel] = useState(null);
const active = blocks.find(b => b.id === sel);

<div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 20, alignItems: "start" }}>
  {/* Left: block list */}
  <div style={{ display: "flex", flexDirection: "column", gap: 5 }}>
    {blocks.map((b, i, arr) => (
      <div key={b.id}>
        <button onClick={() => setSel(sel === b.id ? null : b.id)} className="hover-lift" style={{
          width: "100%", background: sel === b.id ? b.bg : T.white,
          border: `1.5px ${b.isDashed ? "dashed" : "solid"} ${sel === b.id ? b.color : T.border}`,
          borderRadius: 10, padding: "10px 14px", textAlign: "left",
          display: "flex", alignItems: "center", gap: 10, transition: "all 0.2s",
        }}>
          <span style={{ fontSize: 18 }}>{b.emoji}</span>
          <div style={{ flex: 1 }}>
            <div style={{ fontSize: 12, fontWeight: 600, color: sel === b.id ? b.color : T.ink }}>{b.label}</div>
            <div style={{ fontSize: 11, color: T.muted, marginTop: 3 }}>{b.simple.slice(0, 60)}…</div>
          </div>
          <span style={{ fontFamily: mono, fontSize: 10, color: sel === b.id ? b.color : T.muted }}>
            {sel === b.id ? "▲" : "▼"}
          </span>
        </button>
        {/* Connector arrow between blocks */}
        {i < arr.length - 1 && (
          <div style={{ display: "flex", justifyContent: "center", margin: "1px 0" }}>
            <div style={{ width: 2, height: 10, background: T.border }} />
          </div>
        )}
      </div>
    ))}
  </div>

  {/* Right: detail panel */}
  <div style={{ position: "sticky", top: 20 }}>
    {active ? (
      <div className="fade-in" style={{ background: active.bg, border: `1.5px solid ${active.color}50`, borderRadius: 14, padding: "22px 24px" }}>
        {/* Content */}
      </div>
    ) : (
      <div style={{ background: T.cream, border: `1.5px dashed ${T.border}`, borderRadius: 14, padding: 32, display: "flex", flexDirection: "column", alignItems: "center", justifyContent: "center", minHeight: 260, textAlign: "center" }}>
        <div style={{ fontSize: 40 }}>👈</div>
        <Prose style={{ fontSize: 13 }}>Click any block to see its explanation</Prose>
      </div>
    )}
  </div>
</div>
```

### Heatmap with color legend
```jsx
// Color: light (low) → saturated blue (high)
function wColor(w) {
  const r = Math.round(37  + (239-37)  * (1-w));
  const g = Math.round(99  + (246-99)  * (1-w));
  const b = Math.round(235 + (255-235) * (1-w));
  return `rgb(${r},${g},${b})`;
}
function wText(w) { return w > 0.42 ? "#fff" : T.ink; }

{/* Legend */}
<div style={{ display: "flex", gap: 6, alignItems: "center", marginTop: 14 }}>
  <div style={{ display: "flex", gap: 2 }}>
    {[0.05, 0.2, 0.4, 0.65, 0.9].map(v => (
      <div key={v} style={{ width: 22, height: 12, borderRadius: 3, background: wColor(v) }} />
    ))}
  </div>
  <span style={{ fontFamily: mono, fontSize: 10, color: T.muted }}>low → high</span>
</div>
```

### Slider visualization
```jsx
const [pos, setPos] = useState(0);

<div style={{ display: "flex", alignItems: "center", gap: 16, marginBottom: 20 }}>
  <div style={{ fontFamily: mono, fontSize: 12, color: accentColor, minWidth: 80 }}>
    {labelName}: {pos}
  </div>
  <input type="range" min={min} max={max} value={pos}
    onChange={e => setPos(Number(e.target.value))}
    style={{ flex: 1, accentColor: accentColor, height: 6 }} />
  <div style={{ fontFamily: mono, fontSize: 11, color: T.muted, minWidth: 40 }}>of {max}</div>
</div>
```

---

## Formula display

```jsx
{/* Formula card */}
<div style={{ background: T.yellowL, border: "1.5px solid #fde68a", borderRadius: 12, padding: "16px 20px" }}>
  <SubHeading color={T.yellow}>THE FORMULA</SubHeading>
  {/* Formula in mono */}
  <div style={{ fontFamily: mono, fontSize: 15, color: T.ink, letterSpacing: 0.4, margin: "10px 0 12px" }}>
    Result = f(X) = ...
  </div>
  {/* Variable legend */}
  <div style={{ display: "flex", flexDirection: "column", gap: 6 }}>
    {[["X", "input description", T.blue], ["f", "function description", T.teal]].map(([sym, desc, col]) => (
      <div key={sym} style={{ display: "flex", gap: 10, background: T.white, borderRadius: 8, padding: "8px 12px" }}>
        <span style={{ fontFamily: mono, fontSize: 13, color: col, minWidth: 24 }}>{sym}</span>
        <Prose style={{ fontSize: 12.5 }}>{desc}</Prose>
      </div>
    ))}
  </div>
</div>
```

---

## Numbered step list (for algorithm walkthroughs)

```jsx
{[
  ["Step name", "Step description"],
].map(([title, desc], i) => (
  <div key={i} style={{ display: "flex", gap: 12, alignItems: "flex-start", background: T.cream, borderRadius: 8, padding: "10px 14px" }}>
    <div style={{ fontFamily: mono, fontSize: 12, color: accentColor, flexShrink: 0, minWidth: 20 }}>{i + 1}</div>
    <div>
      <div style={{ fontFamily: mono, fontSize: 11, color: T.ink, marginBottom: 3 }}>{title}</div>
      <Prose style={{ fontSize: 12.5 }}>{desc}</Prose>
    </div>
  </div>
))}
```

---

## Anti-patterns to avoid

| ❌ Don't | ✅ Do instead |
|---|---|
| Bullet point lists as explanation | Prose paragraphs with `<Prose>` |
| Hardcoded hex colors | Use `T.blue`, `T.orange`, etc. |
| `useState` with `Math.random()` in initial value | Seed-based deterministic generation |
| Giant monolithic tab component | Extract sub-components with clear names |
| Raw `<p>` tags | `<Prose>` component |
| Raw `<div>` with bg + border | `<Card>` component |
| Font strings inline | Use `serif`, `sans`, `mono` constants |
| Forgetting `key={active}` on tab content | Always add it — enables fade-in |
| No step labels on interactive elements | Always add "STEP N — ..." labels |
| Color legend omitted | Always include for heatmaps |
