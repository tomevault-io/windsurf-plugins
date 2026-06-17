---
trigger: always_on
description: This file provides Codex guidance for this repository.
---

# AGENTS.md

This file provides Codex guidance for this repository.

## Project Overview

React Native + Expo template with Feature-Sliced Design (FSD) architecture and an AI agent harness for full lifecycle app development.

## Tech Stack

- Framework: React Native 0.81 + Expo 54
- Routing: Expo Router
- State: Zustand for client state, TanStack Query for server state
- Styling: NativeWind
- Forms and validation: React Hook Form + Zod
- API: Axios with token auto-refresh
- TypeScript: strict mode

## Codex Harness Rules

This repository includes Claude-era harness files under `.claude/`. Codex does not execute Claude agents, Claude settings, or Claude permission files as native runtime features. Treat those files as project reference documents only.

When a task maps to a harness role, read the relevant `.claude/agents/*.md` file before implementation and apply the domain rules manually:

| Task Type | Reference |
| --- | --- |
| Idea and market research | `.claude/agents/idea-researcher.md` |
| Product planning and PRD | `.claude/agents/product-planner.md` |
| Specs and task breakdown | `.claude/agents/spec-planner.md` |
| Design system and theme | `.claude/agents/design-architect.md` |
| FSD module scaffolding | `.claude/agents/feature-builder.md` |
| API and state integration | `.claude/agents/api-integrator.md` |
| UI and screens | `.claude/agents/ui-developer.md` |
| Code quality review | `.claude/agents/qa-reviewer.md` |
| Functional and UX inspection | `.claude/agents/app-inspector.md` |

Use Codex skills when they are available for the same workflow:

| Workflow | Codex Skill |
| --- | --- |
| App ideation | `ideate` |
| App planning | `plan-app` |
| Design system | `design-system` |
| FSD feature creation | `create-feature` |
| Entity creation | `create-entity` |
| Screen creation | `create-screen` |
| App inspection | `inspect-app` |
| Full lifecycle app development | `orchestrate` |
| Store deployment | `store-deploy` |

For full app development, follow this pipeline and do not skip QA:

1. Ideation
2. Product planning (must define KPIs — north-star metric + acquisition/activation/retention/monetization)
3. Spec planning and task breakdown
4. Design
5. Implementation
   - 5a feature scaffolding
   - 5b API integration
   - 5c UI screens
   - 5d Firebase Analytics + Crashlytics integration (KPI events from PRD)
6. QA and app inspection
7. Iteration, up to 3 fix loops
8. Deployment through `store-deploy`

Use `_workspace/` for handoff artifacts between phases. Before moving to a later phase, read the previous phase outputs from `_workspace/` and continue from them.

## Hard Thresholds

These are fail conditions:

| Check | Threshold |
| --- | --- |
| `npm run typecheck` errors | 0 |
| `npm run lint` errors | 0 |
| `any` types in production code | 0 |
| FSD layer dependency violations | 0 |
| Missing safe area handling in screens | 0 |
| Missing barrel exports | 0 |
| Broken NativeWind setup | 0 |
| `toISOString().split('T')[0]` for local date | 0 |
| Tokens or secrets stored in AsyncStorage/MMKV/plaintext | 0 |
| `requestReview()` calls bypassing the policy engine | 0 |

### Secure Storage And Sensitive Data

`AsyncStorage` and `MMKV` write to plaintext stores that are readable on rooted/jailbroken devices. Auth tokens and any other sensitive data must live in iOS Keychain / Android Keystore-backed encrypted storage. Use `expo-secure-store` as the standard.

Install:

```bash
npx expo install expo-secure-store
```

Storage boundary:

| Class | Examples | Storage |
| --- | --- | --- |
| Sensitive | access token, refresh token, OAuth/session tokens, API secret keys, payment tokens, passwords/PINs, premium license keys, PII-bound tokens | `expo-secure-store` |
| Semi-sensitive | push tokens, device secrets, recoverable user pseudo-IDs | `expo-secure-store` |
| Non-sensitive | UI theme, locale, last-visited screen, onboarding flags, non-identifying cache, non-sensitive Zustand slices | `@react-native-async-storage/async-storage` or `react-native-mmkv` |

Forbidden:

- Persisting tokens through Redux/Zustand `persist` into `AsyncStorage`. `persist` is only for non-sensitive slices.
- Putting secrets in `app.config.ts` `extra`, `.env` files shipped to the client bundle, or plaintext JSON.
- Logging tokens or PII to console, Crashlytics, or Analytics params — mask even in `__DEV__`.

FSD layout:

```text
src/shared/secure-storage/
├── client.ts        # expo-secure-store wrapper
├── keys.ts          # SECURE_KEYS constants + TSecureKey
├── types/index.ts
└── index.ts         # barrel export
```

Required wrapper pattern (`src/shared/secure-storage/client.ts`):

```ts
import * as SecureStore from 'expo-secure-store';
import { SECURE_KEYS, TSecureKey } from './keys';

const DEFAULT_OPTIONS: SecureStore.SecureStoreOptions = {
  keychainAccessible: SecureStore.WHEN_UNLOCKED_THIS_DEVICE_ONLY,
};

export const setSecureItem = (key: TSecureKey, value: string) =>
  SecureStore.setItemAsync(key, value, DEFAULT_OPTIONS);

export const getSecureItem = (key: TSecureKey) =>
  SecureStore.getItemAsync(key, DEFAULT_OPTIONS);

export const deleteSecureItem = (key: TSecureKey) =>
  SecureStore.deleteItemAsync(key, DEFAULT_OPTIONS);

export const clearAllSecure = () =>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [seungmanchoi/react-native-fsd-agent-template](https://github.com/seungmanchoi/react-native-fsd-agent-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
