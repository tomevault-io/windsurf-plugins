---
trigger: always_on
description: Build a frontend with a deliberate visual direction held through palette, typography, structure, and texture, with on-screen strings that name real information rather than fabricated demo data or filler labels. Use when building or restyling a frontend.
---


# Frontend Design Skill

Eight anchors. Each is a distinct aesthetic territory locked to specific CSS tokens. Pick one per brief. Match its tokens.

> **Reach for the unexpected. Fidelity to the anchor. Discipline on the content. Nothing left to default.**

---

## 1. How to work

Before writing code, run this sequence:

1. **Context** — Identify purpose, audience, domain, content density. State the problem in one sentence.
2. **Anchor** — Pick one. Lean unexpected. A Swiss punk record label, an Industrial florist, a Brutalist luxury watchmaker, an Aurora tax app, a Chaotic law firm, a Retro-Futuristic wedding photographer, an Organic trading terminal, a Lo-Fi luxury hotel — each more distinctive than its safe counterpart. Safe pairings produce generic work. Unexpectedness is surface-level — what a first-time viewer sees. Don't let "users are technical" or "it's really a data tool" route every brief to Industrial. State the choice and the reason in one line.
3. **Differentiator** — Define one memorable anchor-internal move: a signature interaction, a typographic gesture, a layout motif, or a material treatment. One sentence. Describable. Visible in the rendered output.
4. **System** — Match the chosen anchor's tokens exactly. Picking Swiss means white + sans + grid, not "some flavor of clean."
5. **Implementation** — Outline structure, then build. Content on screen is authored to the discipline in §2 — no fabrication, no filler, no themed replacement for standard UI copy.

Commit fully to one anchor. Hybridising ("Swiss with Brutalist edge") is a category error — each signature excludes the others by construction.

---

## 2. Content is not design

Design is visuals — palette, typography, structure, texture. **Content — every string, number, and label on screen — is authored separately and has its own discipline.** Token fidelity is not a defence against content slop.

The rule: every string on screen must either name real information from the product, or be authored content that knows what it is — headline, button label, legal body, form field name, sample data that reads as sample. What's forbidden is content pretending to be something it isn't.

**Forbidden:**

- **Fabrication posing as real data.** Invented session personas (`a.chen@grid.co`), fake telemetry (`GRID.FREQ 59.998 Hz`, `BUILD 8.2.0-rc3`). If a slot has no real content, leave it empty — do not fabricate to make the screen look alive.
- **Filler labels.** Mono-caps subtitles nobody asked for (`SECURE OPERATOR AUTHENTICATION` under a login masthead), `//`-prefixed kickers pretending to be code comments (`// INTELLIGENCE LAYER`). If removing the string removes no information, it was filler.
- **Themed replacement of standard UI copy.** `Authenticate Session` instead of `Next`; `Remember this operator` instead of `Remember me`. Standard copy for standard actions. Themed copy is a tax that must be paid with actual utility.
- **Unicode glyphs as icon substitutes.** `▣ Dashboard`, `◊ Market Navigator`. Either use a real icon set or use nothing. ASCII art is not iconography.
- **AI-slop register.** The model recognises this. Twee subcopy on serious surfaces (`Ask the grid.`), synth-sci-fi status strips on mundane B2B, ornamental "seam"/"joinery" flourishes pretending to be structural. Recognise it in your own output; cut it before the reviewer does.

---

## 3. The eight anchors

Each anchor locks specific CSS tokens. Picking the anchor commits to those tokens. If the rendered output drifts outside them, the anchor didn't hold.

### 1. Swiss

**Surface:** Pure white `#FFFFFF` or neutral `#F7F7F8`. **Typography:** Akzidenz-Grotesk, Helvetica Neue, or Söhne — sans display and body, one family. **Accent:** Swiss Red `#E4002B`, International Orange `#FF4F00`, or Yves Klein Blue `#002FA7` — one, used deliberately. **Structure:** visible grid lines or 1 px hairline rules. Left-aligned typography; asymmetric balance. Numerals as composition elements (dates, folio numbers, page markers set in condensed sans).

**Breaks if:** warm paper, serif display, grain texture, or centered typography appears.

### 2. Industrial

**Surface:** Pitch black `#000000` or warm-black `#0B0C0A`. **Typography:** IBM Plex Mono, JetBrains Mono, or Berkeley Mono — mono for display and body. **Signal color:** one semantic — green `#00E676`, red `#FF3B30`, amber `#FFB800`, or acid lime `#C6FF4A`. **Structure:** flat; 1 px borders instead of shadows. Tabular numerics via `font-variant-numeric: tabular-nums`.

**Breaks if:** serif typography, proportional fonts, warm paper, any grain, decorative shadows, or rounded corners appear.

### 3. Brutalist


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ilm-Alan/frontend-design](https://github.com/Ilm-Alan/frontend-design) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
