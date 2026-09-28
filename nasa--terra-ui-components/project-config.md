---
trigger: always_on
description: This file contains **MANDATORY instructions** for AI coding assistants (like Claude, GPT, or Cursor Agents) working in this repo. **You MUST follow these guidelines.**
---

## Purpose

This file contains **MANDATORY instructions** for AI coding assistants (like Claude, GPT, or Cursor Agents) working in this repo. **You MUST follow these guidelines.**

Your main goals:

-   **Help build, refactor, and debug Terra UI components** (Lit-based web components, React wrappers, and Python widgets).
-   **ALWAYS use the existing documentation in `docs/` instead of re‑inventing APIs or behavior.**

**REQUIRED:** When in doubt, **you MUST read existing docs and code rather than guessing.**

---

## Where to Read Documentation

-   **Component docs (primary source)**
    -   Markdown docs for each component live at `docs/pages/components/*.md`.
    -   Examples:
        -   `docs/pages/components/map.md`
        -   `docs/pages/components/time-average-map.md`
        -   `docs/pages/components/browse-variables.md`
    -   These files usually contain:
        -   Example HTML usage blocks (often fenced as ` ```html:preview `).
        -   A `[component-metadata:terra-X]` tag; behavior is defined by the underlying component in `src/components/`.
-   **Getting started / framework integration**
    -   `docs/pages/getting-started/*.md` – overall usage, themes, installation, localization, etc.
    -   `docs/pages/frameworks/*.md` – using Terra UI with React, Vue, Angular, etc.
-   **Design tokens**
    -   `docs/pages/tokens/*.md` – color, typography, spacing, etc. **You MUST use these as the single source of truth for design decisions.**

**REQUIRED workflow when building or modifying a component:**

1. **MUST read the relevant `docs/pages/components/<component>.md` first.**
2. **MUST review its implementation in `src/components/<component>/<component>.component.ts` (and related controller/utility files).**
3. **MUST ensure proper tests exist** in `src/components/<component>/<component>.test.ts` covering:
    - Component rendering
    - Property changes
    - Event emission
    - User interactions (clicks, keyboard navigation)
    - Edge cases and error states
    - Accessibility features
4. **MUST keep the docs, implementation, and examples in sync when making behavior or API changes.**

You can also consult the built documentation site under `_site/` (mirrors `docs/`), but prefer editing/reading the Markdown sources in `docs/`.

---

## Core Project Layout

-   **Lit web components**
    -   Source: `src/components/**`
    -   Example complex components:
        -   `time-average-map` – `src/components/time-average-map/*.ts`
        -   `browse-variables` – `src/components/browse-variables/*.ts`
-   **React wrappers**
    -   `src/react/**` – React bindings around the web components.
-   **Python widgets / Jupyter**
    -   `src/terra_ui_components/**` – Python package for Jupyter widgets.
    -   `notebooks/playground.ipynb` – example notebook using components.
-   **Docs site**
    -   Source: `docs/**`
    -   Built site: `_site/**`
-   **Build & tooling scripts**
    -   `scripts/*.js` – Node build, theming, React wrapper generation, etc.
    -   `scripts/plop/*.js|*.hbs` – code generators for new components/widgets.

**REQUIRED:** When you need to understand how a feature works, **you MUST start from the component docs, then read the corresponding `src/components` files and any data services or utilities they depend on.**

---

## Common Commands (Node & Python)

From the repo root:

-   **Install JS deps**

```bash
npm install
```

-   **Run dev server (docs + components)**

```bash
npm start        # alias for `npm run serve`
```

This uses `node scripts/build.js --serve` and will:

-   Build the components and docs
-   Start a local dev server
-   Auto-reload the browser for most source changes (no full HMR due to custom elements constraints)

-   **Build for production**

```bash
npm run build          # builds Lit components, docs/assets, etc.
```

-   **Create a new component (scaffold)**

```bash
npm run create terra-my-component
```

This uses `plop` to scaffold:

-   Component source files under `src/components/`
-   Styles
-   Jupyter widget support
-   A docs page in `docs/pages/components/`
-   **A test file (`*.test.ts`)** – **MUST implement comprehensive tests** before considering the component complete.

After scaffolding, run `git status` to see all new/updated files and **update docs/examples as needed.**

-   **Python / Jupyter dev workflow (using `uv`)**

```bash
uv venv
source .venv/bin/activate
uv pip install -e ".[dev]"
npm run start:python   # launches Jupyter Lab via .venv
```

Then open `notebooks/playground.ipynb` to test components in Jupyter.

---

## Code Style & Conventions

-   **Formatting**
    -   **MUST use Prettier** with the shared config `@gesdisc/prettier-config`.
    -   **MUST run `npm run prettier`** or rely on `lint-staged` hooks for staged `.ts` and `.js` files.
-   **Languages & frameworks**
    -   Core components are **Lit 3** web components in TypeScript (`lit` package).
    -   Data-heavy UI uses utilities such as `leaflet`, `ol`, `plotly.js-dist-min`, `ag-grid-community`, etc. **MUST extend existing patterns rather than inventing new ones.**
-   **Component naming**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nasa/terra-ui-components](https://github.com/nasa/terra-ui-components) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-28 -->
