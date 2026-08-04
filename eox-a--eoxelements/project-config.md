---
trigger: always_on
description: Executable reference for AI agents working in this repo. Follow these rules literally.
---

# AGENTS.md for EOxElements

Executable reference for AI agents working in this repo. Follow these rules literally.

---

## 1. Before Writing Any Code

Answer these questions in order. Stop if you can't answer one.

1.  **Which element is changing?** Identify the folder in `/elements/` (e.g., `elements/map`).
2.  **Is this a single-element change?** If the request touches multiple elements, state that you will split it into atomic PRs/commits.
3.  **New/changed attributes or properties?**
    - Attributes: strings, numbers, booleans (HTML compatible).
    - Properties: Objects, Arrays, Functions (Complex types).
4.  **What is the minimum change?** State it in one sentence.
5.  **Which existing pattern does this follow?** (e.g., "following logic extraction pattern in `src/methods/`").

If any answer is "I don't know," stop. Ask a clarifying question. Do not guess.

---

## 2. Project Overview

**EOxElements** is an npm workspaces monorepo containing **multiple published web component packages** under the `@eox/` scope, plus a shared utilities package (`@eox/elements-utils`) and an MCP server (`@eox/elements-mcp-server`).

- **Package name:** `@eox/elements` (private, not published)
- **Module system:** ESM (`"type": "module"`)
- **Node requirement:** `>=24.0.0` (enforced via `engine-strict=true` in `.npmrc`)
- **Security Hardening**: `ignore-scripts=true` is enabled in `.npmrc`. This means:
  - `postinstall` scripts (like Cypress binary download) are skipped. Run `npx cypress install` manually for local testing.
  - `prepack` scripts are skipped. When publishing or testing packaging, manually run `npm run types:generate`.
- **Package Quarantine**: `min-release-age=7` (days) is set in `.npmrc` to prevent the use of brand-new, potentially unverified package versions.
- **Component framework:** [Lit](https://lit.dev/) (v3.2+)
- **Language:** JavaScript with JSDoc type annotations (NOT TypeScript source files)

## 3. Technology Stack

| Category            | Technology                                      |
| ------------------- | ----------------------------------------------- |
| Component framework | Lit 3.2+ (LitElement)                           |
| Language            | JavaScript + JSDoc (type-checked by TypeScript) |
| Bundler             | Vite (library mode per element)                 |
| Type checker        | TypeScript (checkJs mode, no .ts source)        |
| Test runner         | Cypress (component + E2E)                       |
| Component mounting  | cypress-lit                                     |
| Coverage            | Istanbul (vite-plugin-istanbul) + NYC           |
| Storybook           | @storybook/web-components-vite (v10)            |
| Linting             | ESLint (flat config) + @eox/eslint-config       |
| Formatting          | Prettier                                        |
| Release             | release-please (per-package automated releases) |
| CI/CD               | GitHub Actions                                  |
| Design system       | @eox/ui (CSS variables + fonts)                 |

---

## 4. Architectural Boundaries & Patterns

### Element Anatomy

Every element follows a consistent directory layout:

```
elements/<name>/
├── package.json              # @eox/<name>, main: ./src/main.js
├── vite.config.js            # Vite library build config
├── tsconfig.build.json       # TypeScript config for .d.ts generation
├── src/
│   ├── main.js               # Main LitElement class + customElements.define()
│   ├── types.ts              # TypeScript type definitions (if needed)
│   ├── style.js              # Base/functional CSS (always applied)
│   ├── style.eox.js          # Branded/themed CSS (unless `unstyled`)
│   ├── enums/                # Constants and defaults (e.g. index.js, frozen objects)
│   ├── helpers/              # Pure utility functions
│   ├── methods/              # Extracted component logic (e.g., domain/index.js)
│   └── components/           # Sub-components (if decomposed)
├── stories/                  # Storybook stories (*.stories.js)
└── test/                     # Cypress component tests (*.cy.js)
    ├── cases/                # Test case implementations (extracted logic)
    └── fixtures/             # JSON/image test fixtures
```

### Web Component Definition Pattern

Every element follows this structure in `src/main.js`:

```javascript
import { LitElement, html, nothing } from "lit";
// ... imports

/**
 * @element eox-example
 * @fires {CustomEvent} change - Fired when the value changes
 */
export class EOxExample extends LitElement {
  static get properties() {
    return {
      prop: { attribute: false, type: Array },
      unstyled: { type: Boolean },
    };
  }

  constructor() {
    super();
    this.prop = [];
    this.unstyled = false;
  }

  // Lifecycle and Logic...

  render() {
    return html`
      <style>
        ${style} ${!this.unstyled && styleEOX}
      </style>
      <div>...</div>
    `;
  }
}

customElements.define("eox-example", EOxExample);
```

### Key Rules for Implementation

- **No TypeScript Code**: Use JSDoc for types. Do NOT use TypeScript decorators (`@property`, `@state`) or `.ts` files for logic. TypeScript is used only for type definitions (`types.ts`, `@typedef`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EOX-A/EOxElements](https://github.com/EOX-A/EOxElements) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
