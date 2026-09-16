---
trigger: always_on
description: When you start chatting, write the very first sentence "I READ RULES" AND DO NOT WRITE COMMENTS.
---

# QR Code Generator Project Rules

# General Rules

DO
When you start chatting, write the very first sentence "I READ RULES" AND DO NOT WRITE COMMENTS.
Analyze the relevant code/files fully before changes.
ALWAYS USE CLEAN CODE PRINCIPLES.
Follow WCAG 2.1 AA for all UI.
Use semantic HTML and proper roles; provide accessible names (aria-label/labelledby/describedby).
Ensure all interactive elements are keyboard-operable (Tab + Enter/Space); never trap focus.
Keep visible focus styles; minimum target size 24×24px (touch targets aim 44×44px).
Implement roving tabindex for dense data (only active point tabindex="0").
Mark decorative/non-interactive SVG elements unfocusable (tabindex="-1", aria-hidden="true").
Expose chart container as role="group" + aria-roledescription="line chart" with a name.
Expose each series as role="group" + aria-roledescription="line series" with a name.
Support series keyboard nav: Left/Right ±1, Home/End, PageUp/PageDown ±5, Esc exits.
Announce focused points (e.g., update an aria-live="polite" region).
Provide a keyboard path to a data table or textual summary of the chart.
Respect prefers-reduced-motion; provide non-animated paths where needed.
Make hover info also available on focus/tap; provide tap toggles for legends/filters.
Use strict TypeScript: explicit types/interfaces; prefer literal unions/enums; narrow unknowns.
Keep styling consistent; if Tailwind is present, use Tailwind utilities only.
Use semantic structure for SEO (<main>, one <h1> per page); write descriptive titles/descriptions.
Use meaningful link text; provide alt for non-decorative images; add canonical where needed.
Optimize performance: responsive images (WebP/AVIF), lazy load, code-split, tree-shake.
Cache effectively (HTTP headers/Next SSG/ISR); defer non-critical JS; inline critical CSS when sensible.
Serve via HTTPS; set secure headers (CSP, HSTS, XFO, XCTO); validate/sanitize inputs.
Keep secrets out of the client; follow least-privilege for tokens/keys.
Enforce quality gates: ESLint/Prettier configs; small, single-responsibility components.
Use meaningful names; keep functions short; avoid duplication; document public APIs briefly.
In React/Next.js, prefer functional components + hooks; handle errors with boundaries; use Suspense/SSR/ISR appropriately.

DO NOT
DO NOT WRITE COMMENTS.
Do not change unrelated code or introduce unsolicited refactors.
Do not add new deps/imports without clear need and approval.
Do not remove or weaken accessibility features or focus outlines.
Do not put every data point in the Tab order for large charts.
Do not rely on color/animation alone to convey meaning.
Do not ship hover-only or gesture-only interactions without keyboard/tap equivalents.
Do not use any; avoid @ts-ignore except as reviewed exceptions.
Do not mix styling systems when Tailwind is in use; avoid ad-hoc inline styles.
Do not use multiple <h1>s, keyword stuffing, or hidden SEO text.
Do not ship broken links, empty anchors, or missing alt on images.
Do not bloat bundles (unused libs, un-split routes); avoid blocking the main thread.
Do not expose secrets in code, logs, URLs, or client bundles.
Do not disable linters/formatters to “make it pass”; fix the root cause.
Do not assume desktop/hover; do not lock orientation or break responsive layouts.

# Commit rules�

Commits (build; chore; ci; docs; feat; fix; perf; refactor; revert; style; test;)
build: Changes that affect the build system or external dependencies (e.g., npm, webpack).
chore: Miscellaneous tasks that don't modify src or test files (e.g., updating the .gitignore file).
ci: Changes to the continuous integration configuration files and scripts (e.g., GitHub Actions, GitLab CI).
docs: Documentation-only changes (e.g., updating the README or inline comments).
feat: A new feature or enhancement to an existing feature.
fix: A bug fix that corrects an issue in the code.
perf: Changes that improve performance.
refactor: Code changes that neither fix a bug nor add a feature, but improve the structure or readability of the code.
revert: Reverting a previous commit.
style: Changes that do not affect the meaning of the code (e.g., formatting, white-space, semi-colons).
test: Adding or modifying tests (e.g., unit tests, integration tests).

style > type:[taskNumber]: description, for example: FEAT[1033]: added a tooltip

## Project Overview
A simple, elegant QR code generator web application that creates QR codes from URLs with a clean, gradient-styled interface.

## Tech Stack
- Pure HTML, CSS, and JavaScript (no frameworks)
- QRCode.js library for QR code generation
- Modern CSS with gradients and animations

## Project Structure
```
free-qr-generator/
├── css/
│   └── styles.css      # All styling
├── js/
│   └── script.js       # All JavaScript logic
├── index.html          # Main HTML structure
├── LICENSE
└── README.md
```

## Code Style Guidelines

### HTML
- Use semantic HTML5 elements
- Keep structure clean and minimal
- Use meaningful class names (kebab-case)
- Include proper meta tags for responsiveness

### CSS
- Use modern CSS features (flexbox, gradients, transitions)
- Follow mobile-first approach with responsive design
- Use CSS custom properties for theme colors when adding new features

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Lashapor/free-qr-generator](https://github.com/Lashapor/free-qr-generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
