---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Bitcoin-S is a Scala toolkit for building Bitcoin and Lightning applications on the JVM: protocol
primitives, a Neutrino light client node, wallet, chain/filter management, DLC (Discreet Log Contract)
support, an oracle server, and RPC clients for bitcoind/eclair/lnd/c-lightning. Package namespace is
`org.bitcoins.*`.

## Build & Test Commands

Build system is SBT. Modules follow the pattern `<module>` (main code) / `<module>-test` (tests) on
disk — e.g. `core` / `core-test`, `wallet` / `wallet-test` — but **sbt project IDs on the command line
are always the camelCase form with no hyphens**, derived from the directory name: `core-test` on disk is
`coreTest`/`coreTestJVM` as an sbt id, `wallet-test` is `walletTest`, `dlc-wallet-test` is
`dlcWalletTest`, `bitcoind-rpc-test` is `bitcoindRpcTest`, etc. A hyphenated id like `core-test/test` is
never valid sbt syntax — check `build.sbt` for the exact `lazy val <id> = ...` if unsure.

A few modules (`crypto`, `core`, `testkit-core`, `async-utils`) build for both JVM and JS via
`crossProject`, and **this applies to their test modules too**: there is no plain `coreTest` or
`cryptoTest` project you can run tasks against directly — only the JVM/JS instances exist as real sbt
projects, so it's always `coreTestJVM`/`coreTestJS`, `cryptoTestJVM`/`cryptoTestJS`,
`asyncUtilsTestJVM`/`asyncUtilsTestJS`. (`testkit-core` itself has no separate test module — it's a
fixtures library other tests depend on — so only `testkitCoreJVM`/`testkitCoreJS` exist, for compiling
it.) Every other domain/RPC/app module (`wallet`, `chain`, `node`, `dlc-wallet`, `bitcoind-rpc`, `cli`,
etc.) is a plain single-platform `project`, so `<module>Test` alone is correct for those — no JVM/JS
suffix, and no such suffix exists to add.

```bash
sbt compile                       # compile main sources
sbt Test/compile                  # compile test sources
sbt <module>Test/test             # run all tests in a single-platform module, e.g. sbt walletTest/test
sbt "<module>Test/testOnly *TestClassName"
sbt "<module>Test/testOnly *TestClassName -- -z \"test name pattern\""
sbt scalafmtCheckAll               # check formatting (CI-enforced)
sbt scalafmtAll                    # auto-format everything — run before committing
sbt coreTestJVM/test               # cross-platform module test invocation (JVM side) — NOT coreTest or core-test
sbt cryptoTestJVM/test             # same for crypto
sbt cryptoTestJS/test              # cross-platform module test invocation (JS side)
```

### Prefer the sbt thin client for repeated commands

This is a large multi-module build (8G heap per `.jvmopts`) — plain `sbt <command>` pays full JVM
startup plus build-definition loading on every invocation. When running more than one sbt command in a
session, use `sbt --client` instead so they share one persistent background sbt server:

```bash
sbt --client compile
sbt --client walletTest/test
sbt --client "coreTestJVM/testOnly *SomeSpec"
```

The first `--client` call boots the background server (slow, one-time per checkout); every subsequent
call reuses it, skipping JVM/classpath reinitialization. Restart it with `sbt --client shutdown` after
editing `build.sbt` or files under `project/` (the running server won't pick those up otherwise).

Notes:
- Tests use ScalaTest (with `FutureOutcome` for async fixtures) and ScalaCheck for property-based tests.
  Test classes extend base fixtures from `testkit`/`testkit-core` (e.g. `BitcoinSUnitTest`,
  `ChainDbUnitTest`).
- CI is split across many workflows by module group (see `.github/workflows/Linux_2.13_*.yml`,
  `Mac_2.13_*.yml`) because the full suite is too large for one job — mirror that grouping if running
  broad test sweeps locally.
- `secp256k1jni` provides native JNI bindings to libsecp256k1; native loading goes through
  `NativeLoader`. Set `DISABLE_SECP256K1=true` to fall back to pure-JVM crypto when the native lib is
  unavailable.
- Scala version is 2.13 (see `inThisBuild.sbt`) with `scala213source3` dialect for Scala 3 source
  compatibility; formatting rules live in `.scalafmt.conf`.
- `sbt docs/mdoc` compiles the Markdown docs in `docs/` with live code examples (Docusaurus site lives
  in `website/`).

## Architecture

### Module dependency layers

Dependencies flow strictly downward; higher layers depend on lower ones (never the reverse):

1. **`crypto`** — elliptic curve primitives, hashing, signature schemes (secp256k1-backed, with a
   pure-JVM/JS fallback). Depends on `secp256k1jni` on the JVM side.
2. **`core`** — Bitcoin protocol data structures (transactions, scripts, blocks, addresses, PSBT, DLC
   messages), serialization, script interpreter. No I/O, no actor system — pure protocol logic. Depends
   only on `crypto`.
3. **`async-utils`, `testkit-core`** — small cross-platform helpers/fixtures built on `core`.
4. **`app-commons`** — shared server/CLI concerns: `AppConfig`/`AppConfigFactory` (the config-loading
   base class every module-specific config extends), JSON models shared between server and CLI.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bitcoin-s/bitcoin-s](https://github.com/bitcoin-s/bitcoin-s) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
