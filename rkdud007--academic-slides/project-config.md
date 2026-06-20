---
trigger: always_on
description: Create Beamer-inspired academic HTML presentations from scratch or by converting PowerPoint files. Use when the user wants to build slides for a conference talk, lecture, seminar, or thesis defense. Supports theorem environments, KaTeX equations, algorithm pseudocode, and citations. Helps academics discover their preferred visual theme through Beamer-style previews rather than abstract choices.
---


# Academic Slides Skill

Create zero-dependency, Beamer-inspired HTML presentations that run entirely in the browser. This skill helps academics and researchers build professional presentations with proper theorem environments, equation rendering via KaTeX, and structured frame navigation. Users discover their preferred theme through visual exploration ("show, don't tell"), then the skill generates production-quality frame decks.

## Core Philosophy

1. **Zero Build Dependencies** -- Single HTML files with inline CSS/JS. KaTeX loaded via CDN for equations. No npm, no build tools.
2. **Show, Don't Tell** -- People don't know what they want until they see it. Generate visual theme previews for academic users.
3. **Academic Authenticity** -- Beamer-inspired structure, Computer Modern typography, proper theorem environments. Not PowerPoint-with-serif-fonts.
4. **Production Quality** -- Accessible, responsive, well-commented code.
5. **Viewport Fitting (CRITICAL)** -- Every frame MUST fit exactly within the viewport. No scrolling within frames, ever. This is non-negotiable.

---

## CRITICAL: Viewport Fitting Requirements

**This section is mandatory for ALL presentations. Every frame must be fully visible without scrolling on any screen size.**

### The Golden Rule

```text
Each frame = exactly one viewport height (100vh/100dvh)
Content overflows? -> Split into multiple frames or reduce content
Never scroll within a frame.
```

### Content Density Limits

To guarantee viewport fitting, enforce these limits per frame:

| Frame Type | Maximum Content |
|------------|-----------------|
| Title Frame | 1 title + 1 subtitle + author/institute/date block |
| Content Frame | 1 heading + 4-5 bullet points OR 1 heading + 2 short paragraphs |
| Theorem/Proof Frame | 1 theorem box (max 4 lines) + 1 proof sketch (max 5 lines) |
| Equation Frame | 1 heading + 1-3 display equations with optional annotation |
| Algorithm Frame | 1 heading + max 12 lines of pseudocode |
| Definition Frame | 1 heading + 1-2 definition boxes (max 3 lines each) |
| Citation/References Frame | 1 heading + max 8 reference entries |
| Section Divider Frame | 1 section number + 1 section title + optional outline |

**If content exceeds these limits -> Split into multiple frames**

### Required CSS Architecture

Every presentation MUST include this base CSS for viewport fitting:

```css
/* ===========================================
   VIEWPORT FITTING: MANDATORY BASE STYLES
   These styles MUST be included in every presentation.
   They ensure frames fit exactly in the viewport.
   =========================================== */

/* 1. Lock html/body to viewport */
html, body {
    height: 100%;
    overflow-x: hidden;
}

html {
    scroll-snap-type: y mandatory;
    scroll-behavior: smooth;
}

/* 2. Each frame = exact viewport height */
.frame {
    width: 100vw;
    height: 100vh;
    height: 100dvh; /* Dynamic viewport height for mobile browsers */
    overflow: hidden; /* CRITICAL: Prevent ANY overflow */
    scroll-snap-align: start;
    display: flex;
    flex-direction: column;
    position: relative;
}

/* 3. Content container with flex for centering */
.frame-content {
    flex: 1;
    display: flex;
    flex-direction: column;
    justify-content: center;
    max-height: 100%;
    overflow: hidden; /* Double-protection against overflow */
    padding: var(--frame-padding);
}

/* 4. ALL typography uses clamp() for responsive scaling */
:root {
    /* Titles scale from mobile to desktop */
    --title-size: clamp(1.5rem, 5vw, 3.5rem);
    --h2-size: clamp(1.25rem, 3.5vw, 2.25rem);
    --h3-size: clamp(1rem, 2.5vw, 1.75rem);

    /* Body text */
    --body-size: clamp(0.75rem, 1.5vw, 1.125rem);
    --small-size: clamp(0.65rem, 1vw, 0.875rem);

    /* Spacing scales with viewport */
    --frame-padding: clamp(1rem, 4vw, 4rem);
    --content-gap: clamp(0.5rem, 2vw, 2rem);
    --element-gap: clamp(0.25rem, 1vw, 1rem);

    /* Frame chrome */
    --chrome-height-top: clamp(2.5rem, 5vh, 3.5rem);
    --chrome-height-bottom: clamp(1.5rem, 3vh, 2.5rem);
}

/* 5. Theorem/definition/proof boxes use viewport-relative max sizes */
.theorem-box, .definition-box, .example-box, .lemma-box, .corollary-box {
    border-left: 4px solid var(--theorem-border);
    background: var(--theorem-bg);
    padding: clamp(0.5rem, 1.5vw, 1rem) clamp(0.75rem, 2vw, 1.5rem);
    margin: clamp(0.25rem, 0.5vw, 0.5rem) 0;
    max-height: min(40vh, 350px);
    overflow: hidden;
}

.theorem-box .env-title, .definition-box .env-title {
    font-weight: 700;
    font-size: var(--body-size);
    color: var(--theorem-border);
    margin-bottom: clamp(0.2rem, 0.5vw, 0.5rem);
}

.proof-box {
    border-left: 2px solid var(--proof-border);
    background: var(--proof-bg);
    padding: clamp(0.4rem, 1vw, 0.75rem) clamp(0.75rem, 2vw, 1.5rem);
    font-style: italic;
}

.proof-box::after {
    content: '\25A1'; /* QED square */
    float: right;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rkdud007/academic-slides](https://github.com/rkdud007/academic-slides) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
