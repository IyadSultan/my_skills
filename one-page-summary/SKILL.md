---
name: "one-page-summary"
description: "Use when asked for a one-page summary, one-pager, single-page brief or fact sheet on any subject: gather all available info, distill it, and deliver a colorful one-page PDF with simple diagrams."
---

# One-Page Summary → Colorful PDF

Goal: a reader who knows nothing about the subject understands it in under two minutes from ONE attractive, colorful, A4 page, with at least one simple diagram that explains it visually.

## Phase 1 — Collect everything (before any design)

Gather from every source available in the session, in this order, and keep notes in a scratch file:

1. **The conversation and attachments** — read every attached file fully (PDF, docx, xlsx, slides, images).
2. **The user's own knowledge stores** — memory files, a connected vault/wiki (e.g. the KHCC 2nd Brain `wiki/index.md` → relevant entity/project/source-summary pages), connected folders, Google Drive or other connected document apps.
3. **The web / literature** — WebSearch/WebFetch for current facts; PubMed/Consensus for clinical or scientific subjects. Anything describing the world *now* (roles, figures, status, guidelines) must be looked up, not recalled.
4. **Never use Gmail or email tools** to collect information. If a key fact seems to live only in email, ask the user to paste it.

Stop collecting when new sources stop adding new facts. Record source URLs/files for the footer.

If the subject is ambiguous (e.g. which project, which audience), ask ONE short question; otherwise proceed.

## Phase 2 — Distill

Write the content as a plain outline first. Budget hard — a page holds ~350–450 words plus visuals.

- **Title** (≤8 words) + **one-sentence subtitle** saying what it is and why it matters.
- **Bottom line / key message** — 1–2 sentences, the thing to remember.
- **3–5 key-point blocks**, each: icon/emoji-free short heading + 1–3 lines. Pick the blocks that fit the subject, e.g. What it is · Why it matters · How it works · Status / numbers · Risks / open issues · Next steps.
- **2–4 headline numbers** (KPI tiles) if real figures exist. Never invent numbers.
- **One or two simple diagrams** — choose the form that matches the content:
  - process / workflow → left-to-right flow of 3–6 boxes with arrows
  - timeline / milestones → horizontal line with dated dots
  - parts of a whole / stakeholders → hub-and-spoke
  - comparison → 2–3 column side-by-side or a tiny bar chart
  - hierarchy → simple tree
  Diagrams carry ≤6 elements, ≤4 words per label.
- **Footer**: sources (short), author/date line (today's real date, ISO).

Be ruthless: if it doesn't help the reader grasp the subject, cut it. Plain language, no jargon without a gloss.

## Phase 3 — Design & build (HTML → PDF)

Build a single self-contained HTML file and print it to PDF with headless Chromium (Playwright). This gives the best color, typography and SVG diagrams.

Design rules:
- Page: A4 portrait, `@page { size: A4; margin: 0 }`, content padded ~12mm. Use CSS grid: header band → key-message strip → 2-column body (text blocks + diagram) → KPI row → footer.
- Color: one confident primary + one accent + soft tinted backgrounds for cards (e.g. deep teal #0F6E74, coral #E8684A, sand #F6EFE6, ink #1E2A32). Each key block gets a colored left rule or tinted card. Keep text contrast ≥ 4.5:1.
- Type: one clean sans (Inter / Helvetica / system-ui) — title 26–30pt bold, headings 11–12pt bold colored, body 9–10pt, line-height 1.35. Google Fonts are fine if reachable; otherwise system fonts.
- Diagrams: hand-written inline SVG (rounded rects, arrows with markers, circles), using the same palette; labels in the body font. No external images needed. Mermaid only if it renders cleanly offline.
- KPI tiles: big number (20–24pt, colored) + small caption.
- Whitespace is a feature; no walls of text, no clip-art, no emoji unless the user wants them.
- Branding: neutral by default. Use an organization's logo/colors only if the user asks or supplies them.

Render:
```python
from playwright.sync_api import sync_playwright
with sync_playwright() as p:
    b = p.chromium.launch()
    pg = b.new_page()
    pg.goto(f"file://{html_path}")
    pg.pdf(path=pdf_path, format="A4", print_background=True,
           margin={"top":"0","bottom":"0","left":"0","right":"0"})
    b.close()
```
(If Chromium is unavailable, fall back to WeasyPrint; keep `print_background`-equivalent colors.)

## Phase 4 — Verify (mandatory)

1. **Exactly one page**: check with `pypdf` (`len(PdfReader(pdf).pages) == 1`). If 2+, cut words first, then tighten spacing, then drop body font by 0.5pt (never below 8.5pt). Re-render and re-check.
2. **Look at it**: rasterize (`pdftoppm -png -r 110`) and view the PNG. Check: nothing clipped or overlapping, diagram labels legible, colors print well, visual hierarchy obvious at a glance.
3. **Facts**: every number and claim traces to a collected source; dates correct.

## Phase 5 — Deliver

Send the PDF to the user (SendUserFile). If working in a connected folder, also save it there next to related material as `<subject>-one-pager-YYYY-MM-DD.pdf`. Reply with one or two sentences: what the page covers and any gap (e.g. a figure you couldn't confirm). Offer the editable HTML only if useful.