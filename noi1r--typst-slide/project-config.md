---
trigger: always_on
description: >
---


# Typst Slide Workflow (Touying)

Universal skill for academic Typst presentations via Touying. Full lifecycle:
create → compile → review → polish → verify.

**Why Typst over Beamer?** Typst compiles in milliseconds (no multi-pass), has sane error messages, built-in scripting, and automatic package management. Touying is the most mature Typst slide framework with rich theme support.

---

## 0. REFERENCE PREAMBLE

Default template for new presentations. Use this unless the user provides a custom template.

```typst
#import "@preview/touying:0.7.0": *
#import themes.university: *
#import "@preview/cetz:0.4.2"
#import "@preview/fletcher:0.5.8" as fletcher: node, edge
#import "@preview/theorion:0.5.0": *
#import cosmos.clouds: *
#show: show-theorion

// CeTZ and Fletcher bindings for Touying animations
// Inside CeTZ canvas, use (pause,) tuple syntax — NOT #pause
#let cetz-canvas = touying-reducer.with(reduce: cetz.canvas, cover: cetz.draw.hide.with(bounds: true))
// Inside Fletcher diagram, use bare pause (no #)
#let fletcher-diagram = touying-reducer.with(reduce: fletcher.diagram, cover: fletcher.hide)

// Semantic colors — use throughout for visual consistency
#let positive = rgb("#0173B2")    // blue (correct, advantage)
#let negative = rgb("#DE8F05")    // orange (limitation, drawback)
#let emphasis = rgb("#029E73")    // green (highlight, key finding)
#let neutral = luma(140)          // muted context
#let accent = rgb("#CC78BC")      // additional accent (purple)

// Semantic text helpers
#let pos(body) = text(fill: positive, body)
#let con(body) = text(fill: negative, body)
#let HL(body) = text(fill: emphasis, body)

#show: university-theme.with(
  aspect-ratio: "16-9",
  // handout: true — uncomment for PDF export (collapses #pause subslides)
  config-common(
    handout: true,
    frozen-counters: (theorem-counter,),
  ),
  config-info(
    title: [Title],
    subtitle: [Subtitle],
    author: [Presenter: \[name\]],
    date: datetime.today(),
    institution: [Shanghai Jiao Tong University],
    logo: emoji.school,
  ),
)

#title-slide()
```

**Rules:**
- **Always `"16-9"` aspect ratio** — modern projectors are 16:9.
- **Always `handout: true` for PDF export** — without this, `#pause` creates duplicate subslide pages in the PDF (e.g., a slide with 3 pauses becomes 3 pages). Set `handout: false` only when presenting live with pdfpc/pympress. This is the #1 source of inflated page counts and "empty-looking" slides.
- **NO heading numbering** — never use `numbly` or `#set heading(numbering: ...)`. Numbered headings (1.1, 2.3) make slides look like a textbook. Slide titles should be plain text.
- **Manual outline, not auto-generated** — never use `#outline()` or `#components.adaptive-columns(outline(...))`. These produce book-style TOC with page numbers and dotted leaders. Instead, write a simple bullet list:
  ```typst
  == Outline
  + Topic A
  + Topic B
  + Topic C
  ```
- **Default presenter**: `author: [Presenter: [name]]` and `institution: [Shanghai Jiao Tong University]`. Override only if user specifies otherwise.
- If user provides a custom template or theme: use theirs.
- **Domain macros go inside math, not as top-level `#let`** — avoid `#let GG = $bb(G)$` then `$#GG_1$` (breaks in math context). Instead write `$bb(G)_1$` directly, or define math-safe macros: `#let GG(sub) = $bb(G)_#sub$`.
- **Available built-in themes** (choose based on context):
  - `themes.university` — academic blue, good for seminars and defenses (default)
  - `themes.metropolis` — clean minimalist, popular at tech conferences
  - `themes.dewdrop` — light and fresh, good for workshops
  - `themes.simple` — bare-bones, maximum content control
  - `themes.aqua` — water-color tones, suitable for soft topics
  - `themes.stargazer` — dark elegant, good for keynotes
- **Theorem environments** via `theorion` (v0.5.0): provides `theorem`, `lemma`, `corollary`, `definition`, `example`, `proof` with auto-numbering. Use `config-common(frozen-counters: (theorem-counter,))` to prevent counter increments during animation subslides.
- **CeTZ** (`@preview/cetz:0.4.2`) for diagrams — Typst's TikZ equivalent.
- **Fletcher** (`@preview/fletcher:0.5.8`) for flowcharts and arrow diagrams — built on CeTZ, much easier for node-edge diagrams.
- **Heading-based slide splitting**: `=` creates section pages, `==` creates slides (for most themes at slide-level=2). Use `---` or `#pagebreak()` for manual page breaks.
- **Special heading tags**: `<touying:hidden>` (don't render), `<touying:skip>` (no section page), `<touying:unnumbered>` (no page number), `<touying:unoutlined>` (exclude from outline).
- **NEVER use `#set page(...)`** — Touying resets it. Use `config-page(...)` in the theme `.with()` call instead.

---

## 1. HARD RULES (Non-Negotiable)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Noi1r/typst-slide](https://github.com/Noi1r/typst-slide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
