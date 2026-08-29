---
trigger: always_on
description: Guidance for AI coding agents (and humans skimming for the rules) working in this
---

# AGENTS.md — OpenASR

Guidance for AI coding agents (and humans skimming for the rules) working in this
repository. For step-by-step contributor setup see [CONTRIBUTING.md](CONTRIBUTING.md);
this file is the short, load-bearing list of what must stay true.

OpenASR is a **local-first speech-to-text platform**, maintained as the public
**Apache-2.0 open core**. It ships a Rust CLI, a local OpenAI-compatible HTTP API
subset, a model-metadata registry, and native ggml-backed ASR execution.

## Repository map

```text
crates/
  openasr-core/          # engine: model families, ggml runtime, .oasr packs, catalog, trust boundaries
  openasr-cli/           # the `openasr` binary
  openasr-server/        # local HTTP API + pairing/remote-compute server
  openasr-client/        # client trust primitives (TOFU pinning, pairing safety codes)
  openasr-system-audio/  # system-audio capture backends (macOS/Windows/Linux)
docs/                    # ROADMAP, KNOWN_LIMITATIONS, FAQ, format contracts, design docs
tooling/publish-model/   # model-pack publishing pipeline + public model-card templates
model-registry/          # bundled catalog + signed manifest (public verification key only)
perf/                    # performance harness, suite.toml, committed baselines
```

## Model-family onboarding (read before changing `models/`)

The normative lifecycle is [Model-family lifecycle](docs/design/model-family-lifecycle.md).
Read it together with [Model Onboarding](docs/MODEL_ONBOARDING.md), the
[Model Onboarding Contract](docs/design/model-onboarding-contract.md), and the
[`.oasr` Package Contract](docs/format/OASR_PACKAGE_CONTRACT_V1.md) before adding
or migrating a family. The live Rust inventory is the authority; these documents
describe the gates and the required ownership boundaries.

Hard rules for a new family:

- Evolve the one `OpenAsrArchitectureDescriptor` inventory row and fill every
  required facet. Do not add a parallel registry, `Default` escape, wildcard
  match, or runtime `Deferred` state.
- Write through the shared `PackEnvelope` and verify once through `PackVerifier`.
  Product paths and the core execute request carry `VerifiedPack`/`AdmittedPack`;
  public import results carry the writer's `VerifiedPack` beside a diagnostic
  `output_path`. The public direct-path ingress must turn its candidate into the
  same proof exactly once before dispatch. A bare path or preflight alone is not
  proof of a valid package contract.
- Use generated dispatch, validator, eviction, force-link, and audit projections.
  Once a projection owns a behavior, delete the old hand-written table and its
  tests/docs rather than keeping two sources of truth.
- Keep family code limited to frontend/tensor binding/topology assembly. Reuse
  shared blocks, decode/cancel drivers, and backend-neutral placement; a
  dedicated graph requires a structural reason and conformance coverage. Family
  policy consumes typed backend capabilities; provider-name parsing belongs only
  at the shared runtime boundary.

Start and finish the weight-free integration through the repository-owned
commands; do not hand-scaffold a parallel path:

```bash
cargo xtask family new <module_slug> --profile-id <profile-id>
# implement the generated fail-closed skeleton, then:
cargo xtask family conformance --profile-id <profile-id>
```

Before a new model becomes a staged or public release candidate, complete the
scope and catalog handoff in [Model Onboarding, Step 5](docs/MODEL_ONBOARDING.md#step-5--choose-the-integration-scope-and-close-the-release-handoff).
Publishing metadata has a separate human-edited source of truth; never hand-edit
generated registry/catalog files, fabricate hashes/URLs, or treat a passing
weight-free gate as release evidence.

## Building from source (the part agents forget)

The ggml backend is a **git submodule** compiled from source, so a plain clone will
not build. The very first step is always:

```bash
git submodule update --init --recursive        # pulls crates/openasr-core/third_party/openasr-ggml
```

`crates/openasr-core/build.rs` shells out to **CMake** and compiles ggml C/C++/Metal,
so the host also needs a C/C++ toolchain, `cmake`, and on Linux `libasound2-dev`
(ALSA, for `cpal`). Rust is pinned by `rust-toolchain.toml` (edition 2024). Then:

```bash
cargo run -p openasr-cli -- --help
cargo run -p openasr-cli -- transcribe fixtures/jfk.wav --model whisper-small --backend mock --format json
```

## Invariants — do not regress these

These encode product promises ("no telemetry / fail-closed / no silent download")
and the open-core trust boundary. Treat them as hard constraints:

- **`native` is the default backend** and is fail-closed by stage. It runs local
  ggml `.oasr` packs; new code must fail closed with typed errors, never fabricate
  output, and never reach for the network silently. **`mock` is an opt-in
  deterministic stub** (`--backend mock`, hidden in `--help`) for plumbing/CI;
  default tests pass `--backend mock` explicitly and stay local, network-free, and
  weight-free.
- **No silent downloads.** Auto-install of a missing model happens **only in the
  CLI** `transcribe`/`live` handlers, **only through a visible consent prompt**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [QuintinShaw/openasr](https://github.com/QuintinShaw/openasr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
