---
trigger: always_on
description: Go port of Vercel's AI SDK (`github.com/grafana/ai-sdk`), providing streaming
---

# AGENTS.md

Go port of Vercel's AI SDK (`github.com/grafana/ai-sdk`), providing streaming
LLM orchestration wire-compatible with `@ai-sdk/react`.

## Upstream Reference

This project is a Go port of [Vercel's AI SDK](https://github.com/vercel/ai).
The upstream TypeScript implementation is the canonical reference for behavior,
naming, and protocol details.

Every implementation, bug fix, refactor, test, fixture, and documentation task
should consider the matching upstream behavior before deciding the Go shape.
Use upstream as normal development context, not only for explicit parity work.
Always compare against the same upstream package versions registered in
`test/conformance/upstream.yaml`; do not compare local Go code against upstream
`main`, latest docs, or an arbitrary package version unless the task is
explicitly upgrading the baseline. When local upstream checkouts or online
source are used, resolve them to the registered version/tag first and note any
version mismatch in the work summary.

- **Wire compatibility with `@ai-sdk/react`**: The SSE wire format
  (`UIMessageChunk`) must stay compatible with `useChat`, `useCompletion`, and
  `useObject`. Any change to chunk types, serialization, or SSE framing must be
  validated against the TypeScript frontend hooks.
  Docs: https://ai-sdk.dev/docs
- **Provider interface mirrors LanguageModelV4**: `provider.LanguageModel`
  follows the upstream LanguageModelV4 spec. Keep the interface aligned.
  Spec: https://github.com/vercel/ai/tree/main/packages/provider/src/language-model/v4
- **Use upstream as reference**: When investigating bugs, designing features, or
  clarifying expected behavior, consult the Vercel AI SDK source and docs first.
  Source: https://github.com/vercel/ai

## Development Workflow

### Upstream Alignment

The upstream TypeScript source is the canonical reference for behavior,
naming, and protocol details. Upstream comparison is required for all normal
development work and at every development phase:

Upstream alignment means matching behavior, semantics, and wire format --
not translating TypeScript patterns into Go. Adapt the design to use Go
idioms (interfaces, channels, error returns, etc.) rather than mirroring
the TypeScript architecture directly.

Before implementing, identify the registered upstream package version from
`test/conformance/upstream.yaml` and use source/tests for that same version as
the reference. If the corresponding upstream source cannot be found, state the
gap and avoid substituting another version silently.

- **Planning**: Read the corresponding upstream implementation before designing.
  Trace the execution flow, understand edge cases and design decisions. Document
  any intentional deviations with rationale.
- **Implementation**: Reference upstream source while writing Go code. Read both
  implementation and tests before porting a feature. Preserve upstream semantics
  even where Go idioms differ from TypeScript patterns.
- **Review**: Compare the Go implementation against upstream to verify
  behavioral alignment. Check for drift in wire format, API surface, error
  handling, and edge cases.

### Parity Governance

The registered upstream baseline lives in `test/conformance/upstream.yaml`; the
coverage map lives in `test/conformance/PARITY.md`. Any parity-sensitive change
must identify the relevant baseline and coverage status before implementation
is considered complete.

Parity-sensitive work includes stream parts, UI chunks, SSE framing, provider
messages, provider request conversion, tool orchestration, output behavior,
provider options, frontend interop, and conformance fixtures.

- **Coverage map first**: Use `test/conformance/PARITY.md` to classify the
  work by layer: core orchestration, provider contract, provider implementation,
  frontend interop, or conformance harness. The required proof differs by layer;
  provider work usually needs request snapshots, while core stream work usually
  needs UI chunk snapshots.
- **Conformance-first TDD**: When a reported bug can be reproduced by recorded
  provider chunks or provider request snapshots, add or update the conformance
  fixture first, confirm the Go replay fails, then implement the fix. For new
  parity-sensitive features, record or import upstream behavior alongside the
  implementation so the fixture becomes the regression contract.
- **Baseline checks**: Run `mise run parity-check` when changing committed
  parity behavior. For metadata-only changes, run
  `mise run validate-parity-baseline`.
- **Fixture updates**: Wire-format or provider-boundary behavior changes must
  consider whether `expected.jsonl`, `expected-requests.jsonl`, or
  `expected-object.json` needs to be regenerated.
- **Divergence classification**: Every observed upstream difference must be
  classified as parity-preserving Go adaptation, intentional deviation,
  implementation bug, or coverage gap.
- **Documented gaps**: Intentional deviations and accepted coverage gaps must be
  recorded in `test/conformance/upstream.yaml` or `test/conformance/PARITY.md`.
- **Upstream upgrades**: Package/version bumps must update the baseline
  manifest, conformance dependency pins, generated snapshots, and lockfiles

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [grafana/ai-sdk](https://github.com/grafana/ai-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
