---
trigger: always_on
description: Rúnar compiles a strict subset of TypeScript into Bitcoin SV Script. Developers write smart contracts as TypeScript classes extending `SmartContract` (stateless) or `StatefulSmartContract` (stateful), and the compiler produces Bitcoin Script locking scripts.
---

# Rúnar — TypeScript-to-Bitcoin Script Compiler

## Project Overview

Rúnar compiles a strict subset of TypeScript into Bitcoin SV Script. Developers write smart contracts as TypeScript classes extending `SmartContract` (stateless) or `StatefulSmartContract` (stateful), and the compiler produces Bitcoin Script locking scripts.

Seven independent compiler implementations (TypeScript, Go, Rust, Python, Zig, Ruby, Java) ship in the repo. Two invariants are deliberately separate:

1. **Frontend parity (no exceptions).** All seven compilers parse all nine `.runar.{ts,sol,move,go,rs,py,zig,rb,java}` extensions for every fixture in the conformance suite. Contracts can be written in TypeScript, Solidity-like, Move-style, Go, Rust DSL, Python, Zig, Ruby, or Java syntax — every frontend lowers them to the same AST.
2. **Stack-IR + hex parity (scoped).** For any conformance fixture whose `source.json` does **not** declare a `"compilers"` allowlist, all seven compilers produce byte-identical Stack IR and byte-identical Bitcoin Script hex. Fixtures that carry a `"compilers"` allowlist explicitly opt out of one or more tiers — the listed tiers are still required to match each other.

A small set of crypto codegen modules — Baby Bear field, KoalaBear, Poseidon2, BN254 witness / Groth16, Merkle, FRI / SP1 FRI verifier, FiatShamir-KB — ship Stack-IR codegen in the **Go tier only**. They power Mode-3 STARK / FRI verification flows that haven't been ported to the other six tiers. Fixtures that exercise those primitives carry an explicit `"compilers": ["go"]` allowlist in `source.json` so the non-Go tiers are skipped at the codegen layer (their parsers are still exercised by `multi-format.test.ts`). A few hybrid fixtures (e.g. `state-covenant`, `stateful-bytestring`) parse cleanly through every frontend but depend on a Java Stack-IR pass that is still deferred — those carry an allowlist that omits `"java"` while keeping the other six tiers locked together. See `conformance/README.md` ⇒ "Per-fixture compiler allowlist" for the current opt-outs.

## Repository Structure

```
packages/
  runar-lang/          # Language: base classes, types, builtins (developer imports)
  runar-compiler/      # TypeScript compiler: parser → validator → typecheck → ANF → stack → emit
  runar-ir-schema/     # Shared IR type definitions and JSON schemas
  runar-testing/       # TestContract API, Script VM, interpreter, fuzzer
  runar-sdk/           # Deployment SDK (providers, signers, contract interaction)
  runar-cli/           # CLI tool
  runar-go/            # Go package: types, mock crypto, real hashes, CompileCheck(), deployment SDK
  runar-rs/            # Rust crate: prelude types, mock crypto, real hashes, compile_check(), deployment SDK
  runar-rs-macros/     # Rust proc-macro crate: #[runar::contract], #[public], #[readonly]
  runar-py/            # Python package: types, mock crypto, real hashes, EC operations, deployment SDK
  runar-zig/           # Zig package: types, mock crypto, real hashes, deployment SDK
  runar-rb/            # Ruby gem: types, mock crypto, real hashes, deployment SDK
  runar-java/          # Java package: types, mock crypto, real hashes, deployment SDK, contract simulator
compilers/
  go/                 # Go compiler implementation
  rust/               # Rust compiler implementation
  python/             # Python compiler implementation
  zig/                # Zig compiler implementation
  ruby/               # Ruby compiler implementation
  java/                # Java compiler implementation (Java 17, javax.tools + com.sun.source.tree)
conformance/          # Cross-compiler conformance test suite (multi-format)
examples/
  ts/                 # TypeScript contracts + vitest tests
  go/                 # Go contracts + go test (native Go tests + Rúnar compile checks)
  rust/               # Rust contracts + cargo test (native Rust tests + Rúnar compile checks)
  sol/                # Solidity-like contracts + vitest tests
  move/               # Move-style contracts + vitest tests
  python/             # Python contracts + pytest tests
  java/                # Java contracts + JUnit 5 tests (native Java tests + Rúnar compile checks)
  sdk-usage/          # SDK usage reference docs (not runnable)
  end2end-example/    # End-to-end example (ts, go, rust, sol, move, python, ruby, zig, webapp, webapp-blackjack)
spec/                 # Language specification (grammar, semantics, type system)
docs/                 # User-facing documentation
  formats/            # Format-specific guides (solidity.md, move.md, go.md, rust.md, python.md, java.md)
integration/          # On-chain integration tests (ts, go, rust, python, ruby, zig, java) + regtest tooling
tests/                # Repo-root research vectors (babybear/koalabear/bn254/merkle/FRI) + vitest tests that consume them
go.work              # Go workspace: compilers/go + conformance + examples/end2end-example/go + examples/end2end-example/webapp + examples/end2end-example/webapp-blackjack + examples/go + integration/go + packages/runar-go
```

## Build & Test


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [icellan/runar](https://github.com/icellan/runar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
