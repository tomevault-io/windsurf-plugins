---
trigger: always_on
description: finesse-ui design skill — never-cheap, high-craft web interfaces. Routes by register: brand (landing pages, hero sites, portfolios) → soul + spectacle engine; product (dashboards, admin, analytics) → component system + data viz. Enforces premium substrate, anti-slop blacklist, and pre-flight check on every output.
---


# finesse — Technically Spectacular · Soul-Distinct · Never Cheap

> **finesse builds two kinds of interface and routes by register:**
> - **brand** — design IS the product: landing pages, brand sites, launches, portfolios, hero pages. Optimize for **spectacle + soul + first impression**.
> - **product** — design SERVES the product: dashboards, admin panels, analytics, data tables, app shells. Optimize for **clarity + density + usability** — and still never cheap.
>
> The through-line is identical: **high craft, zero AI-slop.** The premium substrate, the cheapness blacklist, and the pre-flight apply to **both**.

---

## COMMANDS — targeted iteration (don't rebuild for one complaint)

A full build runs STEP 0–8. To iterate on an existing page, map the request to one verb and adjust only that — keep the substrate (STEP 3) and soul intact, then re-run the relevant pre-flight gates.

| Verb | Intent it matches | Focus |
|------|-------------------|-------|
| `init` | "new project / set up finesse" | write `PRODUCT.md` at project root — register, soul, locked dials, anti-references; every later page reads it |
| `document` | "capture the existing design system" | extract a `design-model.yaml` (palette/type/substrate/engine) from real code; report drift |
| `audit` | "is this any good? / review it" — **read-only**, never edits | run the blacklist + spectacle-shown + pre-flight, report findings |
| `bolder` | "too plain / boring" | SPECTACLE +2, engine up a tier (CSS → GSAP → Canvas → Three.js) |
| `quieter` | "too flashy / janky" | SPECTACLE −2, engine down a tier |
| `soul` | "feels generic / wrong vibe" | re-pick persona (STEP 2), re-lock the accent |
| `animate` | "make it move / different animation" | add/swap the engine only, motion alone |
| `densify` | "too sparse / too dense" | DENSITY ± — add/remove content |
| `redesign` | "improve / fix this page" | audit-first; change the responsible module, not the whole page |

---

## STEP 0 — BRAND READ (Before Any Code)

Most AI design output is bad because the model jumps to a default aesthetic. Don't.

**Read project memory first.** If `PRODUCT.md` (the locked brief) or `design-model.yaml` (locked palette/type/substrate) exist at the project root, read them — they override your guesses and keep this page consistent with the rest. Missing on a multi-page project? Run `init` (and `document` for existing code) first.

**Determine the register:**
- **brand** — landing page, brand site, launch, portfolio, campaign, hero page. Bold, opinionated, spectacular.
- **product** — dashboard, admin, analytics, data table, app shell, settings. Clarity, density, usability.

**Output a one-line Design Read before generating:**
`Design Read: {industry} · {soul in 2-3 words} · register={brand|product} · SPECTACLE={n} · hero-engine={type}`

**Anti-Default Discipline (two altitudes):** Name the lazy default, then beat it — and check the *second* reflex too.
- **First-order:** could someone guess theme+palette from the category alone? "Coffee → warm-beige + brass serif." Reject it.
- **Second-order:** could they guess it from category + your anti-reference? "AI tool that's *not* SaaS-cream → editorial-typographic." "Fintech *not* navy-gold → terminal dark mode." Avoiding the first default and landing on the obvious alternative is just slop one tier up. Rework until both are non-obvious.

---

## STEP 1 — THE THREE DIALS

Set these explicitly from the Design Read.

| Dial | 1–3 | 4–6 | 7–10 |
|------|-----|-----|------|
| **SOUL** | neutral, system-default | a clear vibe | unmistakable identity |
| **SPECTACLE** | static + CSS only | GSAP scroll, Canvas 2D accents | Three.js / GLSL / WebGL-FBO hero |
| **DENSITY** | airy, one idea/screen | balanced | editorial, data-rich |

- Astronomy / music / game / crypto → SPECTACLE 7–10
- Law / finance / healthcare / B2B → SPECTACLE 3–5
- Heritage / luxury / editorial → SOUL 8–10, SPECTACLE 4–6
- **Any product register** → SPECTACLE 1–4, DENSITY 6–9. Skip soul/engine sections.

**SPECTACLE ≥ 7 = must ship a working engine.** A gradient blob claiming SPECTACLE 8 is broken. Drop to 4 and ship a polished static page instead.

---

## STEP 2 — PICK A SOUL (brand register only)

Soul diversity is the job — the same method must yield visually unrelated pages for different briefs.

- **Cinematic tech** (cyan/magenta, Inter + JetBrains Mono, Three.js particles) — astronomy, AI, crypto
- **Phosphor terminal** (single neon-green, mono-forward, Canvas data viz) — quant, fintech, security
- **Editorial publication** (cream/ink, Playfair + Spectral, GSAP scroll-reveal) — magazines, film, journals
- **Warm heritage** (amber/copper, Fraunces/EB Garamond, Canvas fire) — whisky, coffee, craft
- **Brutal typographic** (bone/black + hot accent, Anton/Bebas, mix-blend-mode) — fashion, music, culture
- **Quiet luxury minimal** (off-white/forest, Raleway 100–900, CSS-only parallax) — architecture, hotels


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mouse-lin/finesse-skill](https://github.com/mouse-lin/finesse-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
