---
trigger: always_on
description: Multi-module Android wallet for the Polkadot/Kusama ecosystem. Kotlin 2.1, Dagger 2 (KSP), coroutines/Flow,
---

# Nova Wallet Android

Multi-module Android wallet for the Polkadot/Kusama ecosystem. Kotlin 2.1, Dagger 2 (KSP), coroutines/Flow,
View system + ViewBinding (no Compose), Room, substrate-sdk-android. Java 17, AGP 8.9, Gradle 8.11.

## Commands

```bash
./gradlew assembleDevelop                         # main CI build (build type, not a flavor)
./gradlew :feature-swap-impl:compileDebugKotlin   # fast compile check of one module
./gradlew runTest                                 # what CI runs = clean + ktlint + testDebugUnitTest
./gradlew :feature-staking-impl:testDebugUnitTest # fast test loop for one module
./gradlew ktlint                                  # style check (root JavaExec task)
./gradlew ktlintFormat                            # autofix
```

- `ktlint`/`ktlintFormat` are **root-level tasks**, not per-module. `:module:ktlintCheck` does not exist.
- `ktlint` only scans `**/src/main/**` — test sources are unchecked.
- `runTest` starts with `clean`, so it is slow; prefer per-module test tasks while iterating.
- `:module:runModuleTests` exists only in modules that `apply from: '../tests.gradle'`.
- Build types (no product flavors): `debug`, `debugLocal`, `develop`, `releaseTest`, `releaseMarket`,
  `releaseGithub`, `instrumentialTest`. `debug` signs with a shared committed keystore so Google services
  work; use `debugLocal` for a local key.

## Environment gotchas (build fails without these)

- `local.properties` must define every secret key listed in the README, `mock` values are fine.
  Error `Secret X is not found` means the README list is stale — add `X=mock`.
- Rust toolchain + NDK `29.0.14206865` are required: `bindings/hydra-dx-math` and `bindings/metadata_shortener`
  run `cargoBuild` as part of assembling. Add the four Android rustup targets (see README).
- `nova-wallet-dapp-js` / `nova-wallet-metamask-js` are **git submodules** consumed by `feature-dapp-impl`
  (npm build step). Run `git submodule update --init` or that module fails to build.

## Module map

Everything is an Android library except `app`. Common Gradle config is injected by `allmodules.gradle`
(no per-module plugin/compileSdk blocks — don't add them).

| Module | Purpose |
|---|---|
| `app` | Application class, root DI graph, **all navigation** (53 nav graphs, all `Router` impls), only module allowed to depend on `-impl` |
| `common` | Base classes, mixins, validation framework, utils, **all app strings/resources** |
| `core-api` | Tiny cross-cutting contracts (updaters, storage, ethereum) |
| `core-db` | Room `AppDatabase` (version 73), DAOs, migrations |
| `runtime` | Chain registry, connections, runtime metadata, extrinsics, storage access |
| `bindings/*` | Rust FFI: `hydra-dx-math` (swap math), `metadata_shortener` (Ledger) |
| `caip` | CAIP-2/CAIP-10 chain & account ID parsing (WalletConnect) |
| `web3names` | Web3 name resolution for addresses |
| `test-shared` | Shared unit-test helpers (`CoroutineTest`, Mockito/assert helpers) |

Feature modules follow `feature-<name>-api` / `feature-<name>-impl`:
**api** = domain models, interfaces, DI `FeatureApi`, reusable UI mixins; **impl** = data/domain/presentation.
Pairs: account, ahm, banners, buy, cloud-backup, crowdloan, currency, dapp, external-sign, gift, governance,
ledger, nft, onboarding, proxy, settings, staking, swap, versions, wallet, wallet-connect, xcm.

Single-module / non-obvious features:

| Module | Purpose |
|---|---|
| `feature-assets` | The Assets tab: balances, asset list, send/receive, operation history |
| `feature-vote` | Thin host tab that embeds governance + crowdloan screens |
| `feature-ahm-*` | Asset Hub Migration (chain migration flows) |
| `feature-swap-core` | Quoting primitives shared by swap api + impl (path/graph/direction types) |
| `feature-ledger-core` | Ledger transport shared by api + impl |
| `feature-multisig:operations` | Multisig operation flows |
| `feature-account-migration` | Legacy account migration |
| `feature-deep-linking` | Deep link parsing and dispatch |
| `feature-splash`, `feature-push-notifications` | Self-explanatory, no api/impl split |

## Hard architectural rules

1. **No feature module may depend on another feature's `-impl`.** Cross-feature access goes through `-api`.
   Only `app` wires `-impl` modules together (verified: 0 violations today).
2. **All user-facing strings live in `common/src/main/res/values/strings.xml`.** Feature modules have no
   `strings.xml` (only `app` and `feature-ahm-impl` are exceptions). Add the **English** string only —
   the 13 `values-*` locale files in `common` come from the translation pipeline, never hand-edit them.
3. **Navigation lives in `app`.** A feature declares a `Router` interface in its own `presentation` package;
   `app/root/navigation/navigators/Navigator.kt` implements it and a `*NavigationModule` binds it. Adding a
   screen means touching `app` (nav graph XML + navigator).
4. **Room schema changes need a migration** in `core-db/.../migrations/` plus a version bump in
   `core-db/.../AppDatabase.kt`.

## DI: two-level Dagger graph

Feature graph — lazily created and released per feature, keyed by the `-api` module's `FeatureApi` interface:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [novasamatech/nova-wallet-android](https://github.com/novasamatech/nova-wallet-android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
