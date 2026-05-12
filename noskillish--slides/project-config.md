---
trigger: always_on
description: You are helping a human build a slide deck using the Slides framework. This file is everything you need to produce correct, on-brand slides.
---

# Slides — Agent Instructions

You are helping a human build a slide deck using the Slides framework. This file is everything you need to produce correct, on-brand slides.

## What this is

A minimalist HTML slide deck framework. One self-contained HTML file (`deck.html`), no build step, no dependencies beyond Google Fonts (Inter). Navigate with arrow keys, space, swipe, or on-screen buttons. Export to PDF with `P`.

## File structure

```
your-deck/
├── deck.html              ← the deck (edit this)
├── docs/
│   ├── USING.md           ← usage guide
│   ├── STORYTELLING.md    ← talk structure and tone
│   └── DESIGN.md          ← visual tokens and rules
├── media/                 ← images and videos
└── AGENTS.md              ← this file
```

---

## How slides work

Each slide is a `<section class="slide">` inside the `.deck` div. The first slide gets the class `active`. The JS handles navigation, counter, and progress bar automatically. Just add or remove `<section>` blocks.

```html
<section class="slide">
  <div class="slide-inner">
    <!-- content here -->
  </div>
</section>
```

Dark slides add the `.dark` class to `<section>`:

```html
<section class="slide dark">
```

---

## The headline pattern (use everywhere)

Bold anchor + dim extension. This is the visual identity of the system.

```html
<h1>Anchor. <span class="dim">Extension that fades.</span></h1>
```

- First phrase: weight 500, full color
- Second phrase: weight 300, color `#b5b5b0` (or `#888` on dark slides)
- Use on every headline that has the room

---

## Design tokens

### Colors

| Token              | Hex       | Use                              |
|--------------------|-----------|----------------------------------|
| Background         | `#f5f5f3` | Page/slide background            |
| Surface            | `#fafaf8` | Card backgrounds                 |
| Surface (white)    | `#ffffff` | Cards that need more contrast    |
| Ink / Hero         | `#1a1a1a` | Text primary, dark slides        |
| Border soft        | `#e0e0db` | Default borders                  |
| Border medium      | `#d5d5d0` | Emphasized borders               |
| Pill background    | `#eeeee9` | Context pills                    |
| Text dim           | `#a0a09a` | Subtitles, body copy             |
| Text very dim      | `#b5b5b0` | Dim spans in headlines, meta     |
| Text faint         | `#c5c5c0` | Labels, very low priority text   |
| On-dark text       | `#f5f5f3` | Primary text on dark backgrounds |
| On-dark dim        | `#ccc`    | Body text on dark backgrounds    |
| On-dark very dim   | `#888`    | Labels on dark backgrounds       |

### Typography

- **Font:** Inter (weights 300, 400, 500, 600)
- **H1 display:** `clamp(2.5rem, 6vw, 5rem)`, weight 500, tracking `-0.035em`
- **H2 section:** `clamp(1.75rem, 3.5vw, 2.6rem)`, weight 500, tracking `-0.025em`
- **H3 column:** ~1rem, weight 500
- **Body/subtitle:** 0.85–1rem, weight 400, line-height 1.5–1.6
- **Eyebrow:** 0.65–0.75rem, uppercase, weight 500, tracking `0.08–0.1em`, color `#a0a09a`

### Spacing

- Slide padding: `6vh 8vw`
- Max content width: `1100px`
- Card padding: `1rem–1.5rem`
- Border radius: `10px` cards, `4px` small elements
- Section gaps: `2–4rem`

---

## Component reference

Use these exact patterns. Copy the HTML structure. Change only the text content.

### 1. Cover slide

```html
<section class="slide active">
  <div class="slide-inner">
    <div class="eyebrow">Conference · Date</div>
    <h1>Your headline.<br><span class="dim">Continuation.</span></h1>
    <div class="meta">Speaker name · 20 minutes</div>
  </div>
</section>
```

### 2. Quote slide

A single bold statement. No subtitle. No supporting text.

```html
<section class="slide quote-slide">
  <div class="slide-inner">
    <h1>A bold statement <span class="dim">that opens the talk.</span></h1>
  </div>
</section>
```

For a dark quote (closing slide, mic-drop):

```html
<section class="slide dark quote-slide">
```

### 3. Eyebrow + Headline + Subtitle

The default text slide.

```html
<section class="slide">
  <div class="slide-inner">
    <div class="eyebrow">Section label</div>
    <h1>Headline. <span class="dim">One line that lands.</span></h1>
    <p class="subtitle">One or two sentences of nuance. Keep it short.</p>
  </div>
</section>
```

### 4. Two-column (problem / fix)

```html
<section class="slide">
  <div class="slide-inner">
    <div class="two-col">
      <div>
        <div class="eyebrow">The problem</div>
        <h2>What's broken.</h2>
        <p>Description of the pain.</p>
      </div>
      <div>
        <div class="eyebrow">The fix</div>
        <h2>What we built.</h2>
        <p>Description of the solution.</p>
      </div>
    </div>
  </div>
</section>
```

### 4b. Two-column with step stack

Steps with optional `.dim` (blocker), `.kill` (negative outcome), `.live` (positive outcome):

```html
<div class="col-stack">
  <div class="step">First step</div>
  <div class="step">Second step</div>
  <div class="step dim">Blocked step</div>
  <div class="step kill">Negative outcome</div>
  <div class="step live">Positive outcome</div>
</div>
```

### 5. Three-column

```html

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [noskillish/slides](https://github.com/noskillish/slides) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-11 -->
