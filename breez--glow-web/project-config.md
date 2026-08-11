---
trigger: always_on
description: Glow is a Bitcoin/Lightning wallet web app built with React + TypeScript + Vite, using the Breez Spark SDK (WASM).
---

# Claude Code Guidelines

## Project Overview

Glow is a Bitcoin/Lightning wallet web app built with React + TypeScript + Vite, using the Breez Spark SDK (WASM).

## Comment Style

Same standard as spark-sdk's CLAUDE.md "Comment Style" section, condensed for this repo. Applies to any new or modified comment, JSDoc, commit message, or PR description.

0. **No em-dashes or en-dashes** anywhere (code, comments, commits, PRs). Use a colon for an aside, a period between independent clauses, comma + conjunction for contrast, parentheses for a parenthetical, "to" for ranges (`3 to 5 lines`).
1. **Cut what the code already says.** Default to no comment. Add one only for a non-obvious WHY: a hidden constraint, a workaround, a subtle invariant, surprising behavior. When a signature changes, audit nearby JSDoc for params/fields that no longer exist.
2. **Compact what stays.** Target 3 to 5 lines. Longer means it wants to be a linked doc, not an inline essay. No multi-paragraph docstrings.
3. **Calibrate to the reader.** Internal `//` comments: why the code is shaped this way, for whoever maintains the file. Commit messages and PR bodies: what changed and why, not what the code looks like.
4. **Don't leak internal-looking specifics.** No production identifiers, stack-trace excerpts, or one-off debugging breadcrumbs in committed comments.
5. **Strip narrative; keep implementation facts.** No development history ("we used to..."), no PR-credit ("added for #1234"). State workarounds as present-tense facts: "Uses `bar()`: `foo()` deadlocks on the main thread." Pointers to active context are fine (an open upstream bug, a spec section, the issue a defense exists for, e.g. `(#213)`). Decision history belongs in the commit message.
6. **Frame what something IS,** not what happens downstream and not what lives elsewhere. Don't restate the type; document what `undefined`/absence means at the domain level.

## Key Paths (Hardcoded)

Assume these repos are checked out locally:

```
App:     ~/Documents/GitHub/glow-web
SDK:     ~/Documents/GitHub/spark-sdk
WASM:    ~/Documents/GitHub/spark-sdk/packages/wasm
Types:   ~/Documents/GitHub/spark-sdk/packages/wasm/bundler/breez_sdk_spark_wasm.d.ts
```

## SDK Integration

The app uses `@breeztech/breez-sdk-spark` for all wallet functionality. The SDK is a WASM module loaded at startup in `src/main.tsx`.

**Architecture — direct SDK pattern (no wrappers):**
- `src/hooks/useBreezSdk.ts` — owns the full SDK lifecycle: connect, disconnect, event listeners, mnemonic storage, data fetching
- `src/contexts/WalletContext.tsx` — provides `WalletProvider` (React context) and `useWallet()` hook
- `src/App.tsx` — wraps the app in `<WalletProvider client={sdk.sdk}>`
- Components call `useWallet()` to get the `BreezSdk` instance and call SDK methods directly

**How it works:**
```tsx
// In any component rendered after wallet connection:
import { useWallet } from '@/contexts/WalletContext';

const wallet = useWallet(); // Returns BreezSdk — guaranteed non-null

// Call SDK methods directly — no wrappers
const info = await wallet.getInfo({});
const parsed = await wallet.parse(input);
await wallet.sendPayment(preparedPayment);
```

**Key files:**
- `src/hooks/useBreezSdk.ts` — SDK lifecycle, state, event handling
- `src/contexts/WalletContext.tsx` — WalletProvider + useWallet()
- `src/main.tsx` — WASM init + app bootstrap

## Local SDK Development

When testing unreleased SDK changes (PRs, feature branches):

### Quick Setup (One Command)
```bash
# Build SDK and link to app
cd ~/Documents/GitHub/spark-sdk && git checkout <branch-name> && git pull origin <branch-name> && cd packages/wasm && make build && cd ~/Documents/GitHub/glow-web && npm link @breeztech/breez-sdk-spark
```

### Verify Link
```bash
ls -la node_modules/@breeztech/breez-sdk-spark
# Should show symlink → ../../../spark-sdk/packages/wasm
```

### After SDK Changes
```bash
cd ~/Documents/GitHub/spark-sdk/packages/wasm && make build
```

### Unlink (restore npm version)
```bash
npm unlink @breeztech/breez-sdk-spark && npm install
```

### Check SDK Types
```bash
# Find specific type definition
grep -A 10 "export interface TypeName" ~/Documents/GitHub/spark-sdk/packages/wasm/bundler/breez_sdk_spark_wasm.d.ts

# Find method signature
grep "methodName" ~/Documents/GitHub/spark-sdk/packages/wasm/bundler/breez_sdk_spark_wasm.d.ts
```

## Branch Strategy

| Branch | SDK Source | Deployment |
|--------|------------|------------|
| `main` | npm release | glow-app.co (prod) |
| `staging` | npm pre-release | breez-glow-staging.vercel.app |
| feature branches | `npm link` local | Local dev |

## Staging Environment

- **URL**: breez-glow-staging.vercel.app
- **Password**: Set via `VITE_STAGING_PASSWORD` env var in Vercel (Preview only)
- SDK version should track latest pre-release for integration testing

## Common Tasks

### Testing an SDK PR
1. Create feature branch: `git checkout -b feat/sdk-pr-XXX-description staging`
2. Build & link SDK (use Quick Setup above)
3. Fix any breaking changes in app code
4. Test locally with `npm run dev`
5. Open **draft** PR against `staging` branch
6. Once SDK PR merges and releases, update package.json and convert to ready

### PR Naming Convention

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [breez/glow-web](https://github.com/breez/glow-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
