# Content Strategy — Paper to Interactive Artifact

How to write explanations that actually make complex ideas click for non-experts.

---

## The core writing philosophy

The goal is not to summarize the paper. The goal is to **transfer understanding**.
These are different things. A summary tells you what the paper says. Understanding
transfer makes you feel like you could explain it to someone else.

Every concept needs three layers:
1. **What it is** — the definition in plain English
2. **Why it matters** — the problem it solves, why the authors designed it this way
3. **How to feel it** — an analogy or interactive demo that makes it intuitive

---

## Analogy patterns that work

Analogies are the highest-leverage content in any explainer. A good analogy can
do more work than three paragraphs of prose. Use the yellow `<Analogy>` component
for all analogies so readers learn to scan for them.

### Pattern 1: The familiar process analogy
Map the technical process to something the reader does every day.

**Example — Self-Attention:**
> Think of how you resolve the word "bank" in two sentences: "The river bank was steep"
> vs. "The bank approved my loan." You don't look up "bank" in isolation — you scan the
> whole sentence for clues. Self-attention formalizes this exact process mathematically
> for every word simultaneously.

**Works well for:** Mechanisms, algorithms, processes

### Pattern 2: The physical object analogy
Map abstract data structures to tangible objects.

**Example — Query/Key/Value:**
> Imagine a library. Your search slip is the Query (what you're looking for).
> Each book's spine label is the Key (what it advertises). The book's contents
> are the Value (what you actually get when you read it). Attention is the
> process of matching your slip to every spine, then reading the matching books
> in proportion to how well they matched.

**Works well for:** Data structures, abstractions with multiple parts

### Pattern 3: The human team analogy
Map parallel processes to people doing specialized jobs.

**Example — Multi-Head Attention:**
> Imagine reading a legal contract. You might read it once to understand the
> logical argument, again to spot the obligations, a third time for deadlines,
> and a fourth time for contradictions. Multi-head attention does all of these
> reads simultaneously, each one highlighting different aspects of the text.

**Works well for:** Parallel processes, ensemble methods, multiple perspectives

### Pattern 4: The before/after analogy
Contrast the old approach with the new one to make the improvement tangible.

**Example — RNNs vs. Transformers:**
> Imagine trying to understand a novel by reading it one sentence at a time and
> then immediately forgetting earlier sentences — that's what RNNs did. The
> Transformer instead gives you the whole book open in front of you, so any
> sentence can directly reference any other without forgetting anything.

**Works well for:** Papers that introduce improvements over prior work

---

## Prose writing rules

### Do
- Write in paragraphs, not bullet points (bullets fragment understanding)
- Use **bold** for the key term the first time it appears in an explanation
- Use *italics* for emphasis and examples ("In 'The cat sat...'")
- Answer "why was this designed this way?" for every major design choice
- Use active voice: "the model learns" not "it is learned by the model"
- State the insight before the formula: explain what the formula computes, then show it

### Don't
- Don't start an explanation with the formula — it creates a wall of symbols
- Don't use jargon without immediately defining it in plain terms
- Don't say "simply" or "just" — it makes readers feel dumb if they don't get it
- Don't use bullet point lists as the primary explanation mode for a concept
- Don't assume the reader has read the paper — explain from first principles

---

## Formula presentation pattern

Always present formulas in this order:

1. **Plain English statement** — "This formula computes how much attention word A pays to word B."
2. **The formula** — displayed in monospace
3. **Variable legend** — each symbol explained in one line
4. **The intuition** — "Notice that dividing by √dₖ prevents the dot products from getting so large that softmax returns near-zero gradients everywhere."

Example:
```
Attention is computed by asking: "how similar is my query to each key, and
how much of each value should I take?"

  Attention(Q, K, V) = softmax( QKᵀ / √dₖ ) · V

  Q = Query vector (what am I looking for?)
  K = Key matrix (what does each token advertise?)
  V = Value matrix (what does each token contain?)
  √dₖ = scaling factor to prevent gradient vanishing

The division by √dₖ is subtle but important: without it, when dₖ is large,
the dot products grow large in magnitude, pushing softmax into regions with
very small gradients — making training unstable.
```

---

## Tab intro copy formula

Each tab intro should answer three questions in 2–4 sentences:
1. What concept does this tab cover?
2. Why does it matter / what problem does it solve?
3. What will the reader be able to do on this tab?

Template:
> **[Concept name]** is [one-sentence definition]. [Why it matters / what problem it solves].
> On this tab, [what the reader will do / see interactively].

Example:
> **Self-attention** is the mechanism that lets every word in a sentence simultaneously
> look at every other word to understand its context. It replaced sequential RNN processing,
> enabling parallelism and capturing long-range dependencies. On this tab you can click
> any word in a sentence and see — as percentages — which other words it pays most
> attention to.

---

## The "why designed this way?" pattern

For every major architectural or algorithmic choice, explain the reasoning:

Structure:
- **The naive approach** and its problem
- **The insight** that led to the design
- **The tradeoff** accepted

Example — LayerNorm placement:
> The naive approach would be to normalize once at the end of training. But deep
> networks suffer from unstable gradients — the signal can vanish or explode as
> it passes through many layers. The insight is to normalize after every sub-layer
> (attention and FFN), keeping the activation scale consistent throughout training.
> The tradeoff is a small computational cost, but the training stability benefit
> far outweighs it.

---

## Difficulty calibration

Target the **curious non-expert**: someone intelligent, motivated, but not a specialist.
Think of a practicing physician, a lawyer, a product manager at a tech company.

- Assume: can read English well, follows logical arguments, has heard of "neural networks"
- Don't assume: knows what a gradient is, what a matrix multiplication does, what softmax means
- Rule of thumb: if you use a term that wouldn't appear in a newspaper article about AI, define it

Calibration test: read your explanation aloud. If you'd be embarrassed saying it to a
smart 16-year-old, it's either too jargony or too condescending. Aim for the middle.

---

## Interactive element copy

For every interactive element, provide three pieces of copy:

1. **Step label** (monospace, small caps): "STEP 1 — CHOOSE A SENTENCE"
2. **Instruction sentence**: explains what to do and what to expect
3. **Interpretation sentence**: below the output, explains what the user just saw

Example:
```
STEP 2 — CLICK A WORD TO SEE WHERE IT LOOKS

Each button below is a word in the sentence. Click any word and you'll see — as
percentages — how much of its attention it directs to every other word. Words
always attend to themselves somewhat, but interesting patterns emerge for
ambiguous words like "bank."

[interactive heatmap]

What you just saw: "bank" attended strongly to "money" and "holds" — context
clues that tell the model this is a financial institution, not a riverbank.
```
