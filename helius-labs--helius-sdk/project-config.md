---
trigger: always_on
description: <!-- Keep this file in sync with CLAUDE.md. Both describe the same codebase conventions; changes to one should be reflected in the other. -->
---

# GitHub Copilot Instructions — Helius TypeScript SDK

<!-- Keep this file in sync with CLAUDE.md. Both describe the same codebase conventions; changes to one should be reflected in the other. -->

TypeScript SDK for Helius APIs and Solana development. Built on `@solana/kit` with dual ESM/CJS output, lazy loading, and tree-shaking. For full contributor details see [CLAUDE.md](../CLAUDE.md).

## Critical: Use `@solana/kit`, Not `@solana/web3.js`

This SDK uses `@solana/kit` exclusively. Never import from `@solana/web3.js`. When writing Solana types, helpers, or transaction logic, always reach for `@solana/kit` equivalents.

## Critical: No Top-Level Side Effects

The SDK is marked `"sideEffects": false`. Never introduce runtime calls at module scope. Use type assertions instead:

```typescript
// Correct — no runtime call
"ComputeBudget111111111111111111111111111111" as Address

// Wrong — runtime call breaks tree-shaking
address("ComputeBudget111111111111111111111111111111")
```

## Lazy Loading Pattern

Every method and sub-client must use `defineLazyMethod` or `defineLazyNamespace` from `src/rpc/lazy.ts`. This is mandatory — never eagerly import method modules from `src/rpc/index.ts`.

```typescript
// In src/rpc/index.ts
defineLazyMethod<HeliusClient, YourMethodFn>(client, "yourMethod", async () => {
  const { makeYourMethod } = await import("./methods/yourMethod.js");
  return makeYourMethod(call);
});

// Sub-clients use defineLazyNamespace
defineLazyNamespace<HeliusClient, "ws">(client, "ws", async () => {
  const { makeWsClient } = await import("../websockets/client.js");
  return makeWsClient(options);
});
```

## Factory Pattern

Every method is a factory function that receives its dependencies and returns the method implementation. No classes. No default exports.

```typescript
// src/rpc/methods/yourMethod.ts
export const makeYourMethod = (call: RpcCaller): YourMethodFn => async (params) => {
  return call("yourMethod", params);
};
```

- Name factories `make*` (e.g., `makeGetAsset`, `makeWsClient`)
- Name the returned function type `*Fn` (e.g., `GetAssetFn`, `YourMethodFn`)
- Add a JSDoc comment on all public methods

## Code Style

- **Quotes:** Double quotes everywhere — `"value"` not `'value'`
- **Exports:** Named exports only — never `export default`
- **Functions:** Arrow functions — `const fn = () => {}` not `function fn() {}`
- **Variables:** `const` by default; `let` only when reassignment is required
- **Async:** `async/await` with `try/catch`; never throw non-`Error` objects
- **Imports:** Source files always use `.js` extensions — `import { foo } from "./bar.js"`. Tests (ts-jest) omit extensions.
- **Types:** Strict — no implicit any, no unused locals or parameters

## Project Structure

```
src/
  rpc/
    index.ts                      # HeliusClient entry point - add lazy registrations here
    lazy.ts                       # defineLazyMethod / defineLazyNamespace helpers
    caller.ts                     # RpcCaller abstraction
    wrapAutoSend.ts               # Auto-send transaction wrapper
    heliusRpcApi.ts               # Helius-specific RPC API definition
    methods/                      # One file per DAS/RPC method
      getAsset.ts
      getAssetBatch.ts
      getTransactionsForAddress.ts
      searchAssets.ts
      ...                         # (one file per method)
      tests/
  transactions/
    client.ts                     # makeTransactionsClient factory
    sendSmartTransaction.ts
    sendTransactionWithSender.ts
    createSmartTransaction.ts
    types.ts
    tests/
  webhooks/
    client.ts                     # makeWebhooksClient factory
    createWebhook.ts
    getWebhook.ts
    getAllWebhooks.ts
    updateWebhook.ts
    deleteWebhook.ts
    tests/
  websockets/
    wsAsync.ts                    # makeWsClient factory
  enhanced/
    index.ts                      # makeEnhancedClient factory
    lazy.ts                       # Lazy loader for enhanced methods
    types.ts
    tests/
  staking/
    client.ts                     # makeStakeClient factory
    createStakeTransaction.ts
    createUnstakeTransaction.ts
    types.ts
    tests/
  zk/
    client.ts                     # makeZkClient factory
    types.ts
    methods/                      # One file per ZK method (mirrors rpc/methods/)
      getCompressedAccount.ts
      getValidityProof.ts
      ...
      tests/
  wallet/
    client.ts                     # makeWalletClient factory
    getBalances.ts
    getHistory.ts
    types.ts
    tests/
  auth/
    client.ts                     # makeAuthClient — standalone import only
    agenticSignup.ts
    types.ts
    tests/
  types/
    das.ts                        # DAS API types (Asset, GetAssetRequest, etc.)
    enums.ts                      # Shared enums (Network, TransactionDetails, etc.)
    webhooks.ts                   # Webhook types
  http.ts                         # HTTP client with SDK User-Agent header
  version.ts                      # Auto-generated at build time — do not edit
```

## Adding a New RPC Method

1. `src/rpc/methods/yourMethod.ts` — `make*` factory + `*Fn` type
2. `src/rpc/index.ts` — `defineLazyMethod` registration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/helius-labs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
