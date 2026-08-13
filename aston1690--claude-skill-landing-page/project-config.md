---
trigger: always_on
description: Build complete, deployment-ready landing pages with an explicit frontend skill stack: Frontend Design for art direction, Design Taste Frontend for anti-slop implementation, Imagegen Frontend Web for section-level visual references when useful, and this skill for research, conversion structure, build, browser QA, and deployment. Use whenever the user asks for a landing page, sales page, campaign page, one-page marketing site, website redesign, or wants a URL/PDF/doc/brief converted into a web pag
---


# Landing Page Builder

Build high-quality, deployment-ready landing pages by extracting branding and content from client websites and documents, then producing a responsive HTML/CSS page with verified images and clean structure.

This skill exists because landing pages require careful coordination of branding, content, images, and responsive design — and the most common failure mode is treating source documents as text-only, missing the images that carry half the marketing message.

## The Golden Rules

**1. Images are content, not decoration.** When a marketing PDF shows a classroom photo next to a paragraph about student wellbeing, that photo IS the message. A landing page without those images is incomplete regardless of how good the copy is. Extract and integrate images in the first pass, every time.

**2. Keep styling maintainable within the chosen stack.** For a static page, put CSS in `styles.css`, not inline in HTML. For an existing framework project, preserve its established CSS modules, component styles, utility classes, or token system. Do not migrate styling architecture without a reason.

**3. Never invent content.** Use only what the source document provides. Do not make up image captions, visual directions, testimonial quotes, statistics, or client names. If content is missing, flag it and ask — do not fill gaps with AI-generated placeholder copy.

**4. Use the frontend stack deliberately.** Before coding, load the available supporting skills named below. They are complementary roles, not duplicate instructions to paste together blindly. If a named skill is unavailable, continue with the available stack and record the gap; never pretend it ran.

---

## Frontend Skill Stack (Required Routing)

The screenshot-provided frontend tools map to this landing-page workflow as follows. Read `references/frontend-skill-stack.md` for the complete routing and conflict rules.

1. **`frontend-design`** — establish the subject-specific visual thesis, typography, palette, layout concept, signature element, and motion intent. Use for new builds and meaningful visual reshaping.
2. **`design-taste-frontend`** — apply anti-slop implementation discipline, responsive layout mechanics, accessible interaction states, component choices, performance checks, and the final design pre-flight.
3. **`imagegen-frontend-web`** — use when generated visual references would materially reduce ambiguity: unknown visual direction, image-led campaigns, custom illustration/art direction, or a user asking for mockups. It generates references; it does not replace real brand assets or implementation.
4. **`design-taste-frontend-v1`** — legacy fallback only. Use it when an existing project explicitly depends on v1 behaviour; do not load v1 and current Design Taste together.
5. **`imagegen-frontend-mobile`** — not part of a website landing-page build. Use only for a separate mobile-app screen deliverable, never as the page's responsive-web design step.
6. **`Aidesigner Frontend`** — if this exact skill becomes installed and discoverable, use it as an optional second critique of the approved art direction. It is not currently assumed available and must never block the workflow.

Do not run every tool for ceremony. The minimum new-landing-page stack is `frontend-design` + current `design-taste-frontend` + `landing-page`. Add `imagegen-frontend-web` when visual-reference generation is justified. Responsive mobile web remains part of the implementation and QA; it is not delegated to the mobile-app image skill.

### Required pre-code design gate

Before implementation, produce an internal build brief containing:
- the page's audience, single conversion goal, and source-of-truth content;
- one coherent visual thesis tied to the actual subject, not a generic SaaS aesthetic;
- palette and typography roles with licensing/availability checked;
- section order and the job of every section;
- asset map: preserve, obtain, generate, or omit;
- one signature visual idea and a restrained motion plan;
- desktop, tablet, and mobile behaviour for the first fold and complex sections;
- explicit avoid-list based on the brand and reference audit.

If the user supplied a reference, extract principles without cloning its identity. If the source is an existing client site or approved template, preserve approved copy, brand assets, component grammar, and section order unless the user explicitly authorises a redesign.

---

## Anti-Slop Rules (MANDATORY)

These patterns make AI-generated pages look cheap and generic. NEVER use them:

### Banned Copy Patterns
- "Revolutionize", "Transform your", "Unleash the power of", "Leverage", "Empower"
- "Cutting-edge", "Next-generation", "State-of-the-art", "Best-in-class"
- "Seamlessly", "Effortlessly", "Supercharge"
- "In today's fast-paced world", "In an era of"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Aston1690/claude-skill-landing-page](https://github.com/Aston1690/claude-skill-landing-page) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
