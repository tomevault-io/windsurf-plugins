---
trigger: always_on
description: GHC and Cabal come from [ghcup](https://www.haskell.org/ghcup/) (`~/.ghcup/env`). After a fresh VM, if `ghc` is missing from `PATH`, run `source "$HOME/.ghcup/env"` (or open a login shell that sources `~/.bashrc`).
---

# wireform Development Guidelines

## Cursor Cloud specific instructions

### Toolchain on the VM

GHC and Cabal come from [ghcup](https://www.haskell.org/ghcup/) (`~/.ghcup/env`). After a fresh VM, if `ghc` is missing from `PATH`, run `source "$HOME/.ghcup/env"` (or open a login shell that sources `~/.bashrc`).

Ubuntu packages required before the first `cabal build all` (match `.github/workflows/ci.yml` plus OpenSSL for `wireform-network`):

```bash
sudo apt-get install -y build-essential libgmp-dev libffi-dev libncurses-dev \
  zlib1g-dev libnuma-dev xz-utils pkg-config protobuf-compiler \
  libsnappy-dev liblz4-dev libzstd-dev libbrotli-dev librdkafka-dev libssl-dev
```

HLint for local lint: `sudo apt-get install -y hlint` (distro build; CI uses a newer pin via `haskell-actions/setup`).

### Build and test (default workflow)

From repo root:

```bash
cabal update
cabal configure --enable-tests --enable-benchmarks
cabal build all -j2 --ghc-options="-j2"   # first build ~25–40 min on 2-core VMs
cabal test wireform-test --test-show-details=streaming
```

Subsequent builds reuse `~/.cabal/store` and are much faster.

**Hello-world executables** (no external services):

- `cabal run example-derive` — one ADT encoded to proto, CBOR, MsgPack, JSON
- `cabal run example-msgpack` — schema-less roundtrip
- `cabal run payments-pipeline -- demo` — Kafka Streams topology in `TopologyTestDriver` (no broker)

### Optional services

| Service | Start | Notes |
|---------|--------|--------|
| Kafka (integration tests) | `docker compose -f wireform-kafka/test-integration/docker-compose.yml up -d` | Then `WIREFORM_KAFKA_BROKER=localhost:9092` for relevant `cabal test` targets |
| Docs site | `cd website && npm install && npm run dev` | <http://localhost:4321/wireform-/> |
| Nix dev shell | `nix develop` or `nix develop .#ghc98` | Alternative to ghcup; provides fourmolu, prek, native libs |

### Gotchas

- **`loadProto` splice sites need `DataKinds`:** the IDL bridge emits `Proto.Schema.HasField` instances whose field-name argument is a type-level string literal, so any module containing a `$(loadProto …)` splice must enable `{-# LANGUAGE DataKinds #-}`. Forgetting it surfaces as `Illegal type: "<field>" Perhaps you intended to use DataKinds` at the splice line. (`exe:wireform-conformance-runner`, `test:wireform-proto-derive-test`, and `bench:loadproto-bench` were previously missing this and now build.) Generated enums also carry a synthetic open-enum constructor named `<Enum>''Unrecognized !Int32` (two apostrophes — see `unknownConNameFor`); reference that, not `<Enum>'Unknown`.
- **First build is slow** — use `-j2` on small cloud VMs; see [Toolchain](#toolchain).
- **Heavy optional flags** (`+python-interop`, `+dataframe-bridge`, etc.) are off by default; see the Cabal flags table in [Cabal flags worth knowing](#cabal-flags-worth-knowing).

## Code Generation Principles

**All message types must come from the code generator.** This includes well-known
types (`Timestamp`, `Duration`, `Struct`, etc.), descriptor types, and benchmark
types. Hand-written wire encode/decode instances are not permitted because they
drift from what the code generator produces and mask codegen bugs.

- Well-known types live in `src/Proto/Google/Protobuf/*.hs` and are generated
  from the `.proto` files in `proto/google/protobuf/`.
- Supplementary logic (e.g. `packAny`, RFC 3339 formatting, `TypeRegistry`)
  belongs in companion modules like `Proto.Google.Protobuf.Any.Util` or
  `Proto.JSON.WellKnown`. These import the generated types but never define
  wire-level instances.
- Benchmark comparison types must also be code-generated so that benchmarks
  measure the *actual* codegen output, not idealised hand-written decoders.

### Never hand-edit a generated file

Generated files are **output**, not source. Editing them creates silent drift
between what the codegen produces and what the repo claims it produces; the
next regen pass clobbers the edit and the change disappears. The pattern that
broke this rule before:

- a generated module needed a tweak (an extra import, a missing instance,
    a fixed comment),
- the tweak was applied directly to `<Format>/Generated/Foo.hs`,
- the codegen kept generating the old shape,
- a later regen wiped the tweak and reintroduced the original bug.

**Always make the change in the codegen** (`<Format>.CodeGen.*` /
`<Format>/codegen/`) and **regenerate**. The regen output is what gets committed.

#### Audit before committing

Before committing changes that touch any `*/Generated/*.hs` file, run a
regen + diff to make sure the source tree exactly matches what the codegen
produces. For Kafka:

```
./scripts/regen-kafka-protocol.sh /path/to/kafka/clients/src/main/resources/common/message
git diff --stat wireform-kafka/src/Kafka/Protocol/Generated/
# expect zero non-codegen diff (only what your codegen change introduced)
```

If `git diff` shows changes you did not intend, you have a hand-edit somewhere
in the source tree (or a stale Generator output). Revert the hand-edit, fold
the intent into the codegen instead, and re-regen.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iand675/wireform-](https://github.com/iand675/wireform-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
