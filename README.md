# Pinecone — "guess the number" quiz

One standalone question and its answer. A single self-contained HTML file: no dependencies,
no build step, no network calls. Drop it on GitHub Pages and embed it in a lesson.

## Embed it

```html
<iframe
  src="https://YOURNAME.github.io/pinecone-quiz/"
  title="Compound growth guess"
  width="100%"
  height="360"
  style="border:0; display:block; background:transparent"
  loading="lazy"
  scrolling="no"></iframe>
```

Measured heights, every state at every common phone width:

| state | 300–390px wide | 430px |
|---|---|---|
| question | **353** | 330 |
| answer (any guess) | 280 | 258 |

**`height="360"` covers everything.** One caveat worth knowing: the answer state is about
70px shorter than the question, so after someone answers there's a gap at the bottom. If
that bothers you, auto-size instead — the widget posts its height on every state change, so
the card always ends right under its content:

```js
addEventListener('message', (e) => {
  if (e.data?.type === 'pinecone-quiz-height') iframe.style.height = e.data.height + 'px';
});
```

Given you wanted it cut off at the content with no dead space, auto-sizing is the closer
fit. Fixed 360 is the simpler one.

## Light and dark mode

Verified in both. The widget brings all its own colors — dark ink on a painted plate — so it
reads identically on a white lesson page or a near-black one. `color-scheme: only light`
stops Chrome and Safari auto-inverting it in forced dark mode, which would turn the pastels
muddy. Tested with `--force-dark-mode`: no change.

The page background is transparent, so the plate sits on whatever your lesson page uses.

## Deploy to GitHub Pages

```bash
git init && git add index.html && git commit -m "Pinecone quiz"
gh repo create pinecone-quiz --public --source=. --push
```

Then Settings → Pages → Source: `main` / root. One file at the root, so a push is a deploy.

## How it behaves

Question, four options spanning four orders of magnitude. Tapping one **locks the guess** —
no changing it, because the commitment is what makes the reveal land. The answer appears
immediately: their guess echoed back, then the number on the callout. That's the end of it.
No next question, no CTA, no follow-on.

`revealMode: 'after-video'` at the top of the file withholds the answer instead, with a
"Just tell me" escape hatch. I'd leave it on `'immediate'` — committing to a guess and then
being corrected is what makes this stick, and withholding an answer you invited someone to
guess reads as a broken promise rather than intrigue.

## Editing the content

Everything is in the `QUIZ` object at the top of `index.html`:

```js
{
  theme: 'sage',                       // sage | olive | rose | peri | amber
  kicker: 'Take a guess',
  prompt: 'In 1789, George Washington puts $1 into a trust earning 8% a year…',
  options: [
    { label: 'About $190,000' },
    { label: 'About $83 million', correct: true },   // exactly one correct
  ],
  reveal: {
    big:  '$83 million',               // the number on the callout
    line: 'One dollar, 237 years, 8% a year.',
    note: 'Optional. Leave it off and nothing renders.',
  },
}
```

**List options low to high.** The reveal uses their position to work out whether someone
guessed under or over the answer, and reacts accordingly:

| guess | reaction |
|---|---|
| below the answer | "Higher than you think" |
| the answer | "You got it" |
| above the answer | "Not quite that high" |

That last row matters here — "About $4 billion" sits *above* $83 million, so generic copy
like "it's bigger than that" would be wrong for it. Any option can override its reaction
with its own `verdict: '...'` if you want something specific.

Adding a second entry to `questions` automatically brings back a "Next question" button, and
setting `videoUrl` adds a CTA under the answer. Both are off for this one.

**Watch the height if you add copy** — a longer prompt pushes past 360 and will clip.

**The numbers, verified:** 1789 → 2026 is 237 years. $1 × 1.08²³⁷ = **$83,450,713**. "About
$8 million" is a deliberate distractor — it's roughly what the dollar was worth in 1996, so
it's the right answer to a different question. If you change the year or the rate, recompute:
those figures are in the copy, not calculated at runtime.

## The painted look

The plate is **not** a div with a gradient mask — that reads as an airbrush glow, which was
the problem with the first pass. It's an SVG rect run through two filters:

- **`#brush`** — `feTurbulence` with *anisotropic* frequency (`0.03 0.1`: low across, high
  down) feeding `feDisplacementMap` at `scale="5"`. Low horizontal frequency plus high
  vertical frequency makes the edge streak sideways the way a dragged dry brush does,
  instead of tearing evenly like paper.
- **`#grain`** — fine fractal noise desaturated and composited `in` the plate, so the speckle
  stays inside the torn shape instead of spilling past it. Multiply blend at 30%.

Tuning knobs, if you want it rougher or calmer: `scale` on the displacement map (currently
**5**) controls how far the edge wanders — it was 8, which was too chewy; `baseFrequency`
controls how fine the raggedness is, where higher means finer; grain `opacity` controls the
speckle. Each is one number.

The buttons keep the hand-drawn feel with asymmetric `border-radius` and offset hard
shadows, and the answer sits on an amber callout with a dark outline — the same treatment as
your "Net worth = Total assets − Total liabilities" cards.

One gap: the type is a system sans stack, so it won't match your illustrations' lettering.
If you know the real typeface, it's a one-line `@import` plus a change to `--sans`.
