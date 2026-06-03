---
trigger: always_on
description: A browser-only ESM TypeScript package: `@proof.com/proof-vc-web`. Ships a single Web Component, `<proof-verify-id>`, that consumers drop into any markup — React, Vue, Svelte, plain HTML — and that kicks off a Proof verifiable-credentials flow on click. Depends on `@proof.com/proof-vc-common` for `init`, `getAuthorizationRequestURL`, and the `transactionData` builder.
---

# Proof VC Web - AI Assistant Guide

A browser-only ESM TypeScript package: `@proof.com/proof-vc-web`. Ships a single Web Component, `<proof-verify-id>`, that consumers drop into any markup — React, Vue, Svelte, plain HTML — and that kicks off a Proof verifiable-credentials flow on click. Depends on `@proof.com/proof-vc-common` for `init`, `getAuthorizationRequestURL`, and the `transactionData` builder.

## Hard Rules

1. **Files under `src/` MUST NOT import `node:*` or any Node-only package.** This package runs in browsers only. There is no Node entry. Type-only imports (`import type`, `export type *`) are safe because `verbatimModuleSyntax: true` erases them at emit.

2. **The SSR guards in `src/proof_verify_id.ts` and `src/index.ts` MUST stay.** Three sites would otherwise throw when an SSR framework (Next, etc.) evaluates the imported module in Node:
   - `class extends HTMLElement` → guarded by a conditional `Base` constant.
   - `new CSSStyleSheet()` at module scope → guarded by `typeof CSSStyleSheet !== "undefined"`.
   - `customElements.define(...)` → guarded by `typeof customElements !== "undefined"`.

3. **`src/react.ts` MUST NOT be referenced from `src/index.ts`.** It is the sub-path types entry only (`@proof.com/proof-vc-web/react`). Importing it from the main entry would force every non-React consumer (Vue, Svelte, vanilla) to install `@types/react`.

4. **ALWAYS prompt the user before publishing to npm.** Never bump version, push tags, create a GitHub Release, or trigger the publish workflow without explicit confirmation. Publishes are effectively permanent.

5. **Run `yarn check-all` before any commit or push.** It composes format, lint, typecheck, publint. The global pre-commit rule applies; this repo's equivalent of "tests + lint" is the full check suite.

6. **Do not change `yarn publint` to use any flag other than `--pack npm`.** Default `--pack auto` selects yarn-1 mode and reports false-positive "file not published" errors.

7. **Do not widen `engines.node` below `>=22.0.0`.** Matches proof-vc-common.

## Essential Commands

| Command               | Purpose                                       |
| --------------------- | --------------------------------------------- |
| `yarn check-all`      | Full check: format, lint, typecheck, publint  |
| `yarn build`          | `tsc` emit to `dist/`                         |
| `yarn typecheck`      | `tsc --noEmit`                                |
| `yarn lint:check`     | eslint, no fix                                |
| `yarn lint`           | `eslint --fix`                                |
| `yarn format:check`   | `prettier --check`                            |
| `yarn format`         | `prettier --write`                            |
| `yarn publint`        | `publint --pack npm` — do not change the flag |
| `cd site && yarn dev` | Webpack dev server on http://localhost:4000   |

## Architecture

### Files

| File                     | Role                                                                                                               |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------ |
| `src/index.ts`           | Public entry. Registers `<proof-verify-id>` as an import side effect; re-exports `init`, `transactionData`, types. |
| `src/proof_verify_id.ts` | The `ProofVerifyId` class extending `HTMLElement` (via guarded `Base`).                                            |
| `src/styles.ts`          | CSS as a tagged template literal. No SCSS, no sass build step.                                                     |
| `src/react.ts`           | `declare module "react"` JSX augmentation. Sub-path types entry only.                                              |
| `site/`                  | Local test app. Webpack-served, imports parent source via `../../src/index.ts`. Not published, not a workspace.    |
| `docs/`                  | README assets (`button.svg`, `buttons.svg`).                                                                       |

### Element model

`<proof-verify-id>` attaches an open shadow root containing a single `<button>`, the seal `<svg>`, and (unless `size="icon"`) a `<span>` label.

- One module-scope `CSSStyleSheet` is built from `styles.ts` and adopted into every shadow root via `adoptedStyleSheets`.
- `static observedAttributes = ["size"]` — only `size` changes a structural state (label visibility + `aria-label`). `theme` is pure CSS. `nonce`, `state`, `login-hint`, `transaction-data` are read at click time, not observed.
- There is no `connectedCallback`. `attributeChangedCallback` fires during the parser upgrade for parsed HTML, on programmatic `setAttribute`, and on framework-driven attribute updates — that covers React/Vue/Svelte too.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [proof/proof-vc-web](https://github.com/proof/proof-vc-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
