---
trigger: always_on
description: This file provides guidance for AI coding agents working in this repository.
---

# AGENTS.md

This file provides guidance for AI coding agents working in this repository.

## Project Overview

Vipr Wallet is a Progressive Web App (PWA) that serves as an ecash wallet for Fedimint. Built with Vue 3, TypeScript, and Quasar Framework, it enables private and instant lightning transactions.

## Development Commands

### Core Commands

- `pnpm dev` - Start development server with hot reload (opens in Firefox by default)
- `pnpm build` - Build for production (PWA mode)
- `pnpm build:docker` - Build Docker image
- Use `pnpm build:docker` as the standard way to build the Docker image for this project.

### Code Quality

- `pnpm lint` - Run ESLint on source files
- `pnpm lint:fix` - Fix ESLint errors and warnings
- `pnpm format` - Format code with Prettier
- `pnpm format:check` - Verify formatting without modifying files
- `pnpm typecheck` - Run Vue TypeScript compiler checks
- `pnpm final-check` - Run all checks: format check, lint, typecheck, and tests
- After each coherent change set made by an agent, run `pnpm format`.
- After every code, config, or dependency change made by an agent, run `pnpm final-check` before finalizing the work.
- Documentation-only changes do not require `pnpm final-check`, but should still be formatted.

### Testing

Playwright is configured in `playwright.config.ts` (tests live under `test/e2e`).

- `pnpm playwright install` – Install/update Playwright browsers (run once or after upgrades)
- `pnpm test` - Run unit tests once (alias for `pnpm test:unit:ci`)
- `pnpm test:unit:ci` - Run tests once (CI mode)
- `pnpm test:unit:ui` - Run tests with Vitest UI
- `nix develop --accept-flake-config --command pnpm test:e2e` - Run end to end tests using playwright in nix dev shell
- After any updates (code, config, or dependencies), run tests and ensure they pass before finalizing changes.
- For UI verification/debugging tasks, use the Codex Playwright/Browser plugin and Playwright MCP tooling first (`browser_snapshot` first, then interactions/assertions). Use Playwright via CLI only as a fallback when the plugin is unavailable, cannot reach the target, or when running the committed E2E suite itself.

#### Test Authoring Best Practices

- Prefer behavior-focused tests over implementation-focused tests. Exercise components through public user-like interactions such as clicks, input updates, emitted child-component events, router navigation, Pinia actions, and visible DOM state.
- Treat `wrapper.vm` as a test smell in page and component tests. Avoid calling private component methods, reading local refs, or casting `wrapper.vm as any`; this couples tests to `<script setup>` internals and makes harmless refactors break tests.
- Use `wrapper.vm` only when intentionally testing a documented public component instance API, such as methods exposed via `defineExpose` and consumed by a parent component. Do not use it as a shortcut to trigger ordinary UI behavior.
- Use `data-testid` selectors for stable interaction points and state assertions. Avoid broad text assertions unless the copy itself is the behavior under test, such as a user-facing validation or error message.
- Keep business logic in pure utilities, services, or composables when possible, and test those with direct input/output unit tests. Page tests should then verify wiring: DOM event in, store/composable/router effect out.
- When stubbing child components, make the stub preserve the public contract the parent relies on: props, `v-model` events, click events, slots, and relevant emitted events. Avoid inert `true` stubs for children whose events drive the flow under test.
- For wallet/payment flows, assert functional outcomes rather than presentation details: invoice creation arguments, subscription lifecycle, balance refresh, route targets, cleanup behavior, and error handling.

### Design System

- Vipr has a mandatory design system documented in `docs/design-system.md`.
- All new or changed UI must use the shared `vipr-*` classes and CSS tokens from `src/css/app.scss`.
- Do not add ad hoc hardcoded colors, raw pixel radii, or Quasar layout/typography utility classes when an existing token or shared class covers the use case.
- Always register every Quasar icon name you add or change in templates in `src/boot/icon-map.ts`; otherwise the icon may render as raw text instead of a mapped symbol.

### TypeScript Code Style

- Prefer discriminated unions for result, state, and protocol variants instead of broad objects with `success: boolean` plus optional fields. Model success and failure as separate variants so TypeScript can narrow fields reliably at call sites.
- Prefer explicit variant tags such as `type: 'success' | 'error'` when a result has more than two states; boolean discriminants like `success: true | false` are acceptable for simple success/failure results.
- Prefer functional transformations (`map`, `filter`, `reduce`, object/array spreads, pure helper functions) over in-place mutation when deriving data. Keep imperative mutation limited to Vue refs, Pinia state updates, timers/subscriptions, and SDK lifecycle boundaries.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ngutech21/vipr-wallet](https://github.com/ngutech21/vipr-wallet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->
