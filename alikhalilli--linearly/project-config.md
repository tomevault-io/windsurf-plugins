---
trigger: always_on
description: You are working on a free, community-grown linear algebra course for people building AI,
---

# AGENTS.md: the rulebook for AI agents working on linearly

You are working on a free, community-grown linear algebra course for people building AI,
published at linearly.space as a reading companion to MIT 18.06. The quality bar is not
negotiable: this project aims to be the best material of its kind in the world, useful enough
that MIT could cite it. Every rule below was learned the hard way. Follow all of them.

## The one law above the others

**Never claim what you have not verified.** Never state a program output you did not run.
Never call a visual fixed without a rendered screenshot you looked at. Never cite an edition,
date, or fact you did not check against a primary source. Never invent a resource, video, or
citation: fabrication has been caught in this project before, and everything an agent asserts
is re-checkable. When you cannot verify, say so plainly instead of guessing.

## Project shape

- Astro (pinned version in `package.json`; do not upgrade it), MDX content collections,
  Tailwind 4 tokens in `src/styles/global.css`, KaTeX at build time (the `katex` package major
  version must match what `rehype-katex` expects; a mismatch silently breaks subscript
  rendering), Shiki, Pagefind. Package manager is pnpm, always.
- One lecture = one file: `src/content/lectures/lecture-NN.mdx` (7.5 is `lecture-07-5`).
  Frontmatter schema: `src/content.config.ts`.
- `src/pages/lectures/[...slug].astro` provides these components to every lecture: SlideViewer,
  CodeTabs, Callout, Figure, SpanPlayground, MatrixPlayground, and `pre` → CodeFigure. New
  lecture-specific components are imported inside the lecture's own MDX, never registered in
  the shared layout.
- Figures: inline SVG in the MDX plus a matching `.excalidraw` source in
  `drawings/lecture-NN/`. Slides: `public/slides/<deck>/` (never touched by agents).
- Model chapters: `lecture-01.mdx` and `lecture-02.mdx`. Read both before writing any lecture
  content. Model interactives: `SpanPlayground.astro`, `MatrixPlayground.astro`.

## Writing law

Four references define the standard; when in doubt, they outrank any checklist:
[Write Simply](https://paulgraham.com/simply.html),
[Good Writing](https://paulgraham.com/goodwriting.html), and
[How to Write Usefully](https://paulgraham.com/useful.html) by Paul Graham say what to do;
Wikipedia's [Signs of AI writing](https://en.wikipedia.org/wiki/Wikipedia:Signs_of_AI_writing)
says what must never appear, whether or not a machine wrote it.

The style is Paul Graham's "Write Simply". Ordinary words, short sentences, low friction.
Every symbol introduced in prose before it appears in a formula. Picture first, notation last.
Each chapter is a story with a hook and a payoff. Real numbers everywhere; every claim
checkable by hand or by running the shown code.

Hard bans, each one a failed review:

- Em dashes in article prose (use a comma, a colon, or two sentences). Middle-dot separators.
- "not X, but Y" constructions; habitual groups of three; "-ing" analysis clauses tacked onto
  sentences.
- Puffery: crucial, pivotal, delve, showcase, vibrant, testament, elegant, stunning (about the
  work itself). "Serves as", "features" (use "is", "has").
- "clearly", "obviously", "it is easy to see", "simply" as filler.
- Curly quotes or apostrophes in source files. Check: `grep -n "’\|“\|”" <file>` must be empty.
- English words inside display math. Tall constructs (`\begin{bmatrix}`) in inline math:
  tuples like $(1, 2)$ inline, matrices only in display.

Chapter anatomy: `## The idea in one sentence` → story sections → `## Where this is going` →
closing `<Callout type="check">` exercises, the last one: "Redraw Figure N from memory, labels
included. The drawing is the test."

## Notation canon (course-wide, binding)

$A$ matrix, $b$ right-hand side, $x$ unknown; $\hat{x}$ only ever the least-squares solution;
$w$ only an ML weight vector; row space written $C(A^\mathsf{T})$, column space $C(A)$, null
space $N(A)$, left null space $N(A^\mathsf{T})$; bare $R$ is the current factorization's
triangular/rref factor; $Q$ has orthonormal columns; eigenpairs $\lambda, x$ (or $q$ when
orthonormal). Real spaces $\R^n \to \R^m$, never $\C$. Never reuse a letter with two meanings
in one lecture. KaTeX macros available: `\R \C \T \norm{} \inner{}{} \rank \vv{}`.

## Figure law

- **Ink draws, color means, accent acts.** Structure (axes, frames, brackets, construction
  dashes) and ALL text stay `currentColor`; text is never colored. Strokes may additionally
  use `var(--color-accent)` (the action: mapping arrows, the vector being followed),
  `var(--color-stroke-warm)` (the before/input/contrast voice), and
  `var(--color-stroke-green)` (the after/output/achieved voice). Fills use the pastel part
  tokens `--color-part1..6` in two modes: chip mode (a cell/box carrying text on it:
  full-strength fill + that text in fixed dark ink `#141414`, the Callout convention) and
  region mode (large geometric fills: `fill-opacity` 0.30-0.5 so `currentColor` labels stay
  legible in dark). The dark chip ink applies ONLY to glyphs sitting on a part-token fill;
  text on the page background stays `currentColor`, or dark mode breaks the other way.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alikhalilli/linearly](https://github.com/alikhalilli/linearly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-04 -->
