---
trigger: always_on
description: Feature: a LaunchDarkly JSON flag (`inAppMessages`) that drives conditional, targeted user prompts
---

# In-App Messages — Agent Guide

Feature: a LaunchDarkly JSON flag (`inAppMessages`) that drives conditional, targeted user prompts
in `learn-card-app` and `scouts`. Targeting is evaluated **on-device** (platform / semver /
role), the flag value is just the payload. Read `README.md` in this folder for the authored-flag
reference; this file is for making **code** changes safely.

## Architecture (data flow)

```
LaunchDarkly `inAppMessages` (JSON)
   → useFlags()                       (launchdarkly-react-client-sdk)
   → parseInAppMessagesFlag()         (@learncard/types — never throws)
   → useInAppMessages()               (filter by targeting + frequency, pick highest priority)
        ├─ useRuntimeContext()        (platform + native/web/capgo versions + role)
        ├─ messageMatches()/evaluatePredicate()  (predicates.ts — PURE)
        └─ filterSuppressed()         (dismissalStore.ts)
   → InAppMessageHost                 (mounted in each app's FullApp.tsx)
        → InAppMessageModal | InAppMessageBanner | toast
             ├─ useInAppMessageActions()  (internalLink/externalLink/appStore/dismiss)
             └─ useCapgoUpdate()          (capgoUpdate action: OTA download + progress)
```

Schema is the single source of truth: `packages/learn-card-types/src/inAppMessages.ts`
(Zod → inferred TS types). The runtime lives here in `learn-card-base` so both apps share it.

## Golden rules

-   **Schema first.** Any new field/action/predicate starts in `@learncard/types/inAppMessages.ts`,
    then flows through `predicates.ts` (if it affects matching) and the UI. Keep `.passthrough()` /
    safe defaults so old clients never break on new flag fields.
-   **Fail closed.** Missing/unparseable data in a predicate must return `false`, never throw. A
    malformed flag must degrade to "show nothing" (see `parseInAppMessagesFlag`).
-   **`predicates.ts` stays pure** — no React, no Capacitor, no I/O. It is the unit-tested core.
    Every predicate/semver change requires a test in `__tests__/predicates.test.ts`.
-   **Native plugins are dynamic-imported** (`@capacitor/app`, `@capgo/capacitor-updater`,
    `@capacitor/browser`) and guarded, so the web bundle and non-native paths never break. Do **not**
    add static imports of these, and do **not** add `@capacitor/app-launcher` (not installed).
-   **UI follows `apps/*/AGENTS.md`**: only `grayscale-*/emerald-*/amber-*/red-*` tokens, `font-poppins`,
    `rounded-[20px]` pill buttons, `animate-fade-in-up`, Ionicons (no raw emoji). No `IonModal`/`IonInput`.
-   **No `any` / `@ts-ignore` / `@ts-expect-error`.** Keep comments minimal (a repo hook flags them).

## How to extend

### Add a new action type (e.g. `share`)

1. **Schema** — `@learncard/types/inAppMessages.ts`: add a validator and include it in
   `inAppMessageActionTargetValidator`:
    ```ts
    export const shareActionValidator = z
        .object({ type: z.literal('share'), url: z.string() })
        .passthrough();
    // add shareActionValidator to the z.union([...]) in inAppMessageActionTargetValidator
    ```
2. **Runner** — `useInAppMessageActions.ts`: add a `case 'share':` returning `'closed' | 'noop'`.
3. **UI** — usually nothing; the modal/banner render buttons generically. Add a progress/inline
   state only if the action is long-running (like `capgoUpdate`).

### Add a new predicate leaf (e.g. `tenant`)

1. **Schema**: add `tenantPredicateValidator` and put it in the `inAppMessagePredicateValidator`
   union; extend the `InAppMessagePredicate` TS union to match.
2. **Context**: add the field to `InAppMessageRuntimeContext` (`predicates.ts`) and populate it in
   `useRuntimeContext.ts`.
3. **Evaluator**: add an `if ('tenant' in predicate)` branch in `evaluatePredicate`.
4. **Tests**: cover match / no-match / missing-context (fail-closed) in `predicates.test.ts`.

### Add a new presentation (e.g. `fullscreen`)

1. **Schema**: extend `inAppMessagePresentationValidator`.
2. **Host**: `InAppMessageHost.tsx` — add a branch selecting the new component.
3. Build the component next to `InAppMessageModal.tsx` following the UI rules above.

## Store links

`appStore` actions resolve via `storeLinks.ts` from tenant `links.appStoreUrl` /
`links.playStoreUrl` (see `tenantConfigSchema.ts`), with optional per-message
`iosUrl`/`androidUrl`/`webUrl` overrides. Native opens the `https` store URL through
`@capacitor/browser` (device redirects into the store app); web opens a new tab.

## Version sources (targeting `version.source`)

| source   | read from                                | present on                     |
| -------- | ---------------------------------------- | ------------------------------ |
| `native` | `@capacitor/app` `App.getInfo().version` | native only                    |
| `capgo`  | `@capgo/capacitor-updater` `.current()`  | native only (else fail-closed) |
| `web`    | `__APP_VERSION__` (Vite `define`)        | all platforms                  |

`__APP_VERSION__` is defined in both `apps/*/vite.config.ts` and declared module-locally in
`useRuntimeContext.ts` (do not add it to app `global.d.ts` — that causes duplicate-declaration
conflicts because the package is type-checked with its own ambient declaration).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [learningeconomy/LearnCard](https://github.com/learningeconomy/LearnCard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
