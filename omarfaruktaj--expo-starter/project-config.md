---
trigger: always_on
description: > The single source of truth for any AI coding agent (Codex, Claude Code,
---

# AGENTS.md

> The single source of truth for any AI coding agent (Codex, Claude Code,
> Cursor, GitHub Copilot, etc.) working in this repository. Read this **first**,
> before writing or modifying any code.

This template is **expo-starter** — a production-ready Expo SDK 57 starter.
The goal of this document is to let you make correct, consistent changes
without guessing where code belongs or inventing new patterns.

Companion documents (read the relevant ones before non-trivial work):

| Doc                                                                    | When to read                    |
| ---------------------------------------------------------------------- | ------------------------------- |
| [`docs/AI_QUICK_REFERENCE.md`](./docs/AI_QUICK_REFERENCE.md)           | Always — the 1-page cheat sheet |
| [`docs/AI_ARCHITECTURE.md`](./docs/AI_ARCHITECTURE.md)                 | Before any structural change    |
| [`docs/AI_DECISION_RULES.md`](./docs/AI_DECISION_RULES.md)             | When deciding where code goes   |
| [`docs/AI_FEATURE_GUIDE.md`](./docs/AI_FEATURE_GUIDE.md)               | When adding a feature           |
| [`docs/AI_DATA_GUIDE.md`](./docs/AI_DATA_GUIDE.md)                     | When integrating an API         |
| [`docs/AI_UI_GUIDE.md`](./docs/AI_UI_GUIDE.md)                         | When building UI                |
| [`docs/AI_NAVIGATION_GUIDE.md`](./docs/AI_NAVIGATION_GUIDE.md)         | When adding screens/routes      |
| [`docs/AI_TESTING_GUIDE.md`](./docs/AI_TESTING_GUIDE.md)               | When writing tests              |
| [`docs/AI_SECURITY.md`](./docs/AI_SECURITY.md)                         | When touching auth/env/storage  |
| [`docs/AI_PRODUCTION_READINESS.md`](./docs/AI_PRODUCTION_READINESS.md) | Before declaring done           |
| [`docs/AI_PRODUCTION_AUDIT.md`](./docs/AI_PRODUCTION_AUDIT.md)         | Final audit                     |
| [`docs/AI_FEATURE_CHECKLIST.md`](./docs/AI_FEATURE_CHECKLIST.md)       | Per-feature checklist           |
| [`docs/AI_CONTEXT_MAP.md`](./docs/AI_CONTEXT_MAP.md)                   | Task → file map                 |
| [`docs/AI_MODULES.md`](./docs/AI_MODULES.md)                           | Opt-in module deep-dive         |
| [`docs/DEFINITION_OF_DONE.md`](./docs/DEFINITION_OF_DONE.md)           | "Done" definition               |
| [`docs/PROJECT_CONSTITUTION.md`](./docs/PROJECT_CONSTITUTION.md)       | Non-negotiable principles       |
| [`docs/AI_APP_BUILD_WORKFLOW.md`](./docs/AI_APP_BUILD_WORKFLOW.md)     | Building a whole app            |

> `docs/AI_MODULES.md` is produced by task DOCS-B; if the link 404s when you
> read this, it is being written in parallel — the entry paths below remain
> authoritative.

---

## 1. Project purpose

A modern, scalable, **production-ready Expo SDK 57 starter template** that an
AI agent (or human) can take and turn into a real application. It is **not** a
tutorial app and **not** a feature grab-bag — it is the minimal, opinionated
foundation (auth, data, forms, theming, i18n, navigation, dev tooling) that
every serious RN app needs, wired together consistently.

Built on the official Expo SDK 57 default template, with the most useful
architecture and patterns from the [Obytes RN template][obytes] adapted and
modernized — **not a copy** of either.

[obytes]: https://github.com/obytes/react-native-template-obytes

## 2. Tech stack

| Concern                  | Choice                                                                                                                                             | Notes                                                                                                   |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| Framework                | Expo SDK 57 / React Native 0.86 / React 19                                                                                                         | `expo ~57.0.14`                                                                                         |
| Routing                  | Expo Router 7 (typed routes)                                                                                                                       | `experiments.typedRoutes: true`                                                                         |
| Language                 | TypeScript 6 (strict)                                                                                                                              | `extends: expo/tsconfig.base`                                                                           |
| Styling                  | `StyleSheet.create` + typed design tokens                                                                                                          | **No NativeWind/Tailwind.**                                                                             |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [omarfaruktaj/expo-starter](https://github.com/omarfaruktaj/expo-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
