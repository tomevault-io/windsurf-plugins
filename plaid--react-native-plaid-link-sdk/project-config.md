---
trigger: always_on
description: Guidance for AI coding agents integrating `react-native-plaid-link-sdk`.
---

# AGENTS.md

Guidance for AI coding agents integrating `react-native-plaid-link-sdk`.
Applies to **v13.x**. For v11/v12 integrations, see `V13_MIGRATION_GUIDE.md`.

If you learned this SDK before v13, your recalled API is out of date. v13 is a
rewrite onto Expo Modules with a session-based API. Read the rename table below
before writing any Plaid code.

## The import

```ts
import { createPlaidLinkSession } from "react-native-plaid-link-sdk";
```

Every removed pre-v13 name is retained only as a migration diagnostic and is
not usable: `create`, `open`, `openLink`, `usePlaidEmitter`, `submit`,
`destroy`, `dismissLink`, `PlaidLink`, and `EmbeddedLinkView`.

```ts
import { create, open } from "react-native-plaid-link-sdk";
import { EmbeddedLinkView } from "react-native-plaid-link-sdk";
```

Using one reports `TS2349: This expression is not callable` followed by an exact
v13 replacement, such as `Removed in v13. Use await
createPlaidLinkSession(config), then call await session.open() on the returned
session.` Rendering `PlaidLink` or `EmbeddedLinkView` as JSX reports `TS2786`
carrying the same replacement. JavaScript callers receive the same guidance as a
runtime error.

Older v13 releases reported `TS2614 ... Did you mean to use 'import create from
"react-native-plaid-link-sdk"' instead?`. **That suggestion is wrong.** The
deprecated default export is the native module object, not any of these names.
Do not follow the default-import suggestion.

## If you were about to write pre-v13 code

| Pre-v13                                 | v13                                                          |
| --------------------------------------- | ------------------------------------------------------------ |
| `create(config)`                        | `await createPlaidLinkSession(config)`                       |
| `open({ onSuccess, onExit })`           | `await session.open()` — callbacks now go to the create call |
| `openLink({ tokenConfig, onSuccess })`  | `createPlaidLinkSession(...)` then `session.open()`          |
| `<PlaidLink>` component                 | Build your own button, call `createPlaidLinkSession`         |
| `usePlaidEmitter(listener)`             | Pass `onEvent` to the create call                            |
| `submit(data)`                          | `await session.submit(data)` on a Layer session              |
| `EmbeddedLinkView`                      | `PlaidEmbeddedSearchView`                                    |
| `syncFinanceKit(token, bool, bool, cb)` | `await syncFinanceKit({ token, ... })`                       |
| `iOSPresentationStyle: FULL_SCREEN`     | `await session.open(true)`                                   |
| `destroy()`                             | Not needed — each create call resets session state           |
| `dismissLink()`                         | No replacement in v13                                        |

The two changes that trip up most generated code: **callbacks moved from `open`
to the create call**, and **every create function is `async`**.

## Complete v13 export surface

Functions — `createPlaidLinkSession`, `createPlaidIdentityVerificationSession`,
`createPlaidLayerSession`, `createPlaidHeadlessSession`, `syncFinanceKit`.

Constant — `sdkVersion`.

Component — `PlaidEmbeddedSearchView` (with `PlaidEmbeddedSearchViewProps`).

Types and enums — `LinkSuccess`, `LinkExit`, `LinkEvent`, `LinkEventName`,
`LinkError`, `LinkTokenConfiguration`, `PlaidLinkSession`, `PlaidLayerSession`,
`PlaidHeadlessSession`, `SubmissionData`, `FinanceKitSyncBehavior`, and the
other types in `ReactNativePlaidLinkSdk.types`.

Default export — deprecated native module instance. Do not use it; use the named
exports, including `sdkVersion`. It will be removed in the next major version.

## Standard Link

```ts
import { createPlaidLinkSession } from "react-native-plaid-link-sdk";

const session = await createPlaidLinkSession({
  token: "#GENERATED_LINK_TOKEN#",
  onSuccess: (success) => console.log("Success", success),
  onExit: (exit) => console.log("Exit", exit),
  onEvent: (event) => console.log("Event", event),
  onLoad: () => console.log("Link loaded"),
});

await session.open();
```

`onSuccess`, `onExit`, and `onEvent` are required. `onLoad` is optional.
`session.open(true)` requests full-screen presentation on iOS.

## Identity Verification

Use the dedicated function — IDV does not emit `onLoad`, so do not wait for it.

```ts
import { createPlaidIdentityVerificationSession } from "react-native-plaid-link-sdk";

const session = await createPlaidIdentityVerificationSession({
  token: "#GENERATED_IDENTITY_VERIFICATION_LINK_TOKEN#",
  onSuccess: (success) => console.log("Success", success),
  onExit: (exit) => console.log("Exit", exit),
  onEvent: (event) => console.log("Event", event),
});

await session.open();
```

## Layer

Open Link when the Layer flow emits `LAYER_READY`.

```ts
import {
  LinkEventName,
  PlaidLayerSession,
  createPlaidLayerSession,
} from "react-native-plaid-link-sdk";

let session: PlaidLayerSession;

session = await createPlaidLayerSession({
  token: "#GENERATED_LINK_TOKEN#",
  onSuccess: (success) => console.log("Success", success),
  onExit: (exit) => console.log("Exit", exit),
  onEvent: async (event) => {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [plaid/react-native-plaid-link-sdk](https://github.com/plaid/react-native-plaid-link-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
