---
trigger: always_on
description: Notes for automated contributors (and humans) working in this repo. Everything
---

# AGENTS.md

Notes for automated contributors (and humans) working in this repo. Everything
here is enforced by CI — see `.github/workflows/ci.yml` and `scripts/`.

## 1. Never hand-edit a generated file

This is the trap that costs a round trip: a hand-edit to generated code reads
fine in review, passes lint/typecheck/tests, and then fails the CI step
`Check the code coupled to spec/ is in sync with it`.

Generated, do **not** edit by hand:

| Path                             | Produced by                                    |
| -------------------------------- | ---------------------------------------------- |
| `src/low/generated/types.gen.ts` | `pnpm generate` (`@hey-api/openapi-ts`)        |
| `src/low/generated/zod.gen.ts`   | `pnpm generate`                                |
| `src/low/generated/index.ts`     | `pnpm generate`                                |
| `src/low/version.ts`             | `scripts/gen-version.mjs`, run by `pnpm build` |
| `parity/python-surface.json`     | `pnpm sync:python-surface` (see section 6)     |

`spec/openapi.yaml` is also off-limits: it is a vendored, one-way sync of the
canonical Comfy API v2 contract with internal operations stripped (see
`spec/README.md`). Fix the contract upstream; do not patch the vendored copy to
make codegen emit what you want. The same goes for `spec/router-openapi.yaml`,
the vendored Comfy Router contract — nothing is generated from it, but
`src/sdk/routerErrors.ts` is checked against it (section 2).

Everything else under `src/` is hand-written — including `src/low/index.ts`,
which is the aggregator for the whole low layer, **not** codegen output. Note
that `openapi-ts.config.ts` sets `clean: true` on the output directory, so any
file you add under `src/low/generated/` is deleted on the next regeneration.
That directory is also excluded from `oxlint` (`.oxlintrc.json`
`ignorePatterns`) and from `oxfmt` (`.prettierignore`); leave both exclusions
alone. Reformatting generated output makes it differ byte-for-byte from a fresh
generation, which is exactly what the drift check compares.

### Regenerating

```bash
pnpm generate          # rewrites src/low/generated/* from spec/openapi.yaml
pnpm check:spec-drift  # same check CI runs (node scripts/check-spec-drift.mjs)
```

`scripts/check-spec-drift.mjs` regenerates into a temp directory using the same
`openapi-ts.config.ts` and compares the three files above byte-for-byte against
what is committed. There is no tolerance for whitespace or ordering. It then
runs a second, unrelated check that needs no codegen — the Router route against
`spec/router-openapi.yaml` (section 2) — and reports both, so a stale
regeneration cannot hide a moved route.

## 2. Spec-coupled hand-written code

Seven hand-written things must be updated in lockstep when a vendored spec
changes. Three are coupled to `spec/openapi.yaml` and fail
`src/low/spec-coverage.test.ts`; the other four are coupled to
`spec/router-openapi.yaml` and fail `src/sdk/router-spec-coverage.test.ts` and
`src/sdk/router-spec-contract.test.ts` respectively. Only the route check and
the collect check (fourth and sixth below) name the constant to fix; for the
rest the failure message will not tell you this:

- **`src/low/transport.ts` — `OPERATION_IDS`.** Must equal, as a set, the
  `operationId` of every non-internal operation in `spec/openapi.yaml`. A spec
  sync that adds an operation is not done until a transport method exists for
  it.
- **`src/low/transport.ts` — `OPERATION_METHODS`.** Maps every operation ID to
  a method name that must actually exist on `ComfyLow.prototype`. The test
  checks both directions of the mapping and the method's existence.
- **`src/sdk/routerErrors.ts` — the closed `error_type` set.** Checked against
  a _different_ spec: `spec/router-openapi.yaml`, whose
  `components.schemas.RouterErrorType.x-comfy-error-types` names every bucket,
  its tier and its meaning. `src/sdk/router-spec-coverage.test.ts` asserts the
  buckets, their order, their tier split and the class per bucket in both
  directions. Nothing is generated from that spec, so this test is its drift
  check — a Router spec sync that adds a bucket is not done until a
  `RouterError` subclass exists for it, named the PascalCase of the wire value.
  Adding one usually also needs a `routerErrorClassesAheadOfPython` entry in
  `src/sdk/surface-parity.test.ts` until the Python twin lands (section 6).
- **`src/sdk/models.ts` — `RUN_ROUTE_TEMPLATE`, and `src/sdk/credentials.ts` —
  `COMFY_ROUTER_BASE_URL`.** The path `comfy.models.run` posts to and the host
  it posts to by default, checked against `spec/router-openapi.yaml`'s
  `runRouterModel` path, its path parameters and its `servers[0].url`. Checked
  in two places on purpose: `src/sdk/router-spec-contract.test.ts` compares the
  live constants in `pnpm test`, and `scripts/check-spec-drift.mjs` compares
  the same constants read out of their source text (plain Node, no TypeScript
  loader) so `pnpm check:spec-drift` reddens too. Both messages name the
  constant to update. The vendored spec is the side that is right — a Router
  sync that moves the route is not done until the constant follows, and until
  it does `comfy.models.run` 404s. The shared spec reader is

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Comfy-Org/comfy-typescript-sdk](https://github.com/Comfy-Org/comfy-typescript-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
