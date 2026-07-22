---
trigger: always_on
description: > This document is mainly for agents and LLMs to follow when adding or changing
---

# App Bridge Actions

**Version 1.0.0**  
Saleor  
June 2026

> **Note:**  
> This document is mainly for agents and LLMs to follow when adding or changing  
> app-to-Dashboard communication in `@saleor/app-sdk`. Humans may also find it  
> useful, but guidance here is optimized for AI-assisted workflows.

---

## Abstract

Guide for extending the App Bridge communication layer in `@saleor/app-sdk`. The app, running inside
the Saleor Dashboard iframe, sends commands through exactly one channel: an `actions.*` action sent
via `appBridge.dispatch()`. This document explains how to add a new action, how to wrap it in helpers
and hooks, and how to ship it without breaking the public API. Contains 3 rules in 1 category.

**Why it exists:** apps are third-party web apps embedded via a cross-origin `<iframe>`, so the
browser's only sanctioned transport is `window.postMessage`. App Bridge is the typed wrapper over
that single link — app → Dashboard = actions (`dispatch`), Dashboard → app = events (`subscribe`).
There is one physical pipe, so a second message protocol is just an ungoverned use of it; every
command must be an action.

---

## Table of Contents

1. [App Bridge](#1-app-bridge) — **CRITICAL**
   - 1.1 [Use dispatch, not manual postMessage](#11-use-dispatch-not-manual-postmessage)
   - 1.2 [Adding an App Bridge Action](#12-adding-an-app-bridge-action)
   - 1.3 [Release Safety & Public API](#13-release-safety--public-api)

---

## 1. App Bridge

**Impact: CRITICAL**

App Bridge is the contract between an app and the Saleor Dashboard. Getting it wrong fragments the
protocol, confuses app authors, and creates breaking changes that ripple across the SDK, the
Dashboard, and every installed app.

### 1.1 Use dispatch, not manual postMessage

The app talks to the Dashboard through **exactly one** channel: an `actions.*` action sent via
`appBridge.dispatch()`. Never call `window.parent.postMessage` directly or add a second message
protocol.

> **Source**: `src/app-bridge/app-bridge.ts` (`AppBridge.dispatch`), `src/app-bridge/actions.ts`

## The Smell

A raw `window.parent.postMessage(...)` from the SDK, or a new `"saleor:foo"` message constant, is
what this rule exists to stop.

```ts
// ❌ parallel protocol
export const FOO_MESSAGE = "saleor:foo";
window.parent.postMessage({ type: FOO_MESSAGE, value }, "*");

// ✅ one channel
appBridge.dispatch(actions.Foo({ value }));
```

## Why

- One mental model for app authors: `import { actions }` → `dispatch`.
- One greppable, documented surface instead of scattered message strings.
- `dispatch` already gives `actionId` correlation and a Dashboard `ok`/error response.
- The Dashboard listens in one place; a second protocol needs a second listener and its own versioning.

## Helpers and Hooks Wrap Dispatch

Ship ergonomic helpers, but they take an `AppBridge` instance (or pull one from `useAppBridge()`)
and dispatch internally. App authors should never touch raw `postMessage` or import a message
constant.

```ts
export const reportWidgetHeight = (appBridge: AppBridge, height: number): void => {
  if (SSR || !isPositiveFiniteHeight(height)) return;
  appBridge.dispatch(actions.WidgetResize({ height })).catch((error: unknown) => {
    console.warn("WidgetResize dispatch failed:", error);
  });
};
```

## Dispatch Rejects — Always Catch

`dispatch()` returns a promise that **rejects on negative status or after the 10s timeout**, so an
un-caught dispatch can surface as an unhandled rejection. For best-effort commands (resize, route
propagation) you don't need the result: dispatch without `await` and attach a `.catch`. Existing
fire-and-forget call sites (`useRoutePropagator`, `sendNotifyReadyAction`) log in the catch:

```ts
appBridge.dispatch(actions.UpdateRouting({ newRoute })).catch(() => {
  console.error("Error dispatching action");
});
```

For a **high-frequency** signal like resize, use `console.warn` in the catch (not rethrow): failures
are expected on older Dashboards without a handler, but developers need a signal in the console.

## Anti-patterns

❌ **Don't call `window.parent.postMessage` from the SDK** — Dispatch an action instead  
❌ **Don't export a `*_MESSAGE` constant for a new command** — The action `type` is the contract  
❌ **Don't make app authors construct the message envelope** — Provide an `actions.*` creator  
❌ **Don't leave a best-effort dispatch un-caught** — It rejects on timeout/negative; attach a `.catch`

### 1.2 Adding an App Bridge Action

All edits live in `src/app-bridge/actions.ts` (plus a test and a changeset). Copy an existing action
(`WidgetResize`, `PopupClose`) — don't invent a new shape.

> **Source**: `src/app-bridge/actions.ts`, `src/app-bridge/actions.test.ts`

## Recipe

```ts
// 1. ActionType string — camelCase, with JSDoc + the Saleor version it needs
widgetResize: "widgetResize",

// 2. Payload type
export type WidgetResizePayload = { height: number };

// 3. Action type
export type WidgetResize = ActionWithId<"widgetResize", WidgetResizePayload>;

// 4. Creator — withActionId injects a crypto.randomUUID() actionId. Never hand-roll the envelope.
function createWidgetResizeAction(payload: WidgetResizePayload): WidgetResize {
  return withActionId({ type: "widgetResize", payload });
}


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [saleor/app-sdk](https://github.com/saleor/app-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
