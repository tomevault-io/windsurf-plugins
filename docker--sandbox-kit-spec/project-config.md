---
trigger: always_on
description: Guidance for AI coding agents (and the humans steering them) making
---

# Working in this repository

Guidance for AI coding agents (and the humans steering them) making
changes to the Docker Sandbox Kit Specification. It is the operational
companion to [CONTRIBUTING.md](.github/CONTRIBUTING.md) and
[GOVERNANCE.md](GOVERNANCE.md): those say what a change must carry, this
says how to produce and verify one without surprising a reviewer.

## What this repository is

Three things that move together:

| Part | Where | What it is |
|---|---|---|
| The specification | `docs/spec/SPEC-v3.md`, `docs/spec/capabilities/`, `docs/spec/conformance.md` | Normative text. In SPEC-v3 and the capability pages every MUST/SHOULD is anchored and accounted for by a check or a waiver; `conformance.md` binds adapters and the suite itself and carries no anchors |
| The reference implementation | `spec/`, `resolve/`, `assemble/`, `fetch/`, `cmd/frontend/`, `schema/` | Go types and validation (the grammar's source of truth), the resolver, the BuildKit frontend, and the JSON Schema pinned to the Go types |
| The conformance suites | `tck/`, `cmd/kit-tck/` | `kit-tck validate` judges a published Kit; `kit-tck runtime` judges a runtime through an adapter. `tck/sandbox` also judges the suite itself against a fake adapter |

Plus `examples/` (every descriptor must decode and validate against the
current grammar; tests enforce it), `skills/` (tool-agnostic agent skills
for authoring and migrating Kits; nothing tests them), and
`scripts/kit-version.sh` (the map of which example Kits track an upstream
release).

When two representations disagree — Go types, JSON Schema, normative
prose, an example — the code in `spec/` wins, and the fix is to bring the
others back in line, not to special-case the reader.

## Before you start

- Read the relevant normative page before touching behavior. A change to
  what a runtime must do is a change to a capability page first, and to
  the code second.
- When authoring or porting a Kit under `examples/`, read and follow
  `skills/create-kit-v3/SKILL.md` or `skills/migrate-kit-to-v3/SKILL.md`.
  They are the repository's own instructions for that job, and they were
  verified against real artifacts.
- `task --list` names every task with its purpose. Prefer a Task target
  over the underlying `go`/`docker` invocation: the targets carry guards
  and pinned versions that a hand-typed command does not.

Toolchain the tasks assume: Go at the version in `go.mod`,
[Task](https://taskfile.dev), `golangci-lint` v2, Docker with buildx (for
`lint:md`, `kit:dev`, `test:e2e`), and the `sbx` CLI only for
`tck:runtime:sbx`.

## Verifying a change

Run what CI runs, from the fast signal outward. CI runs `validate`,
`lint`, `test:unit`, and `test:tck` on every pull request, so a clean
local run is the same verdict rather than a different one.

| Task | What it does | When |
|---|---|---|
| `task validate` | `gofmt -l` + `go vet` | Every Go change. Seconds |
| `task lint` | `golangci-lint` and `markdownlint` with the repo configs (`lint:go`, `lint:md`) | Every change. `lint:md` needs Docker; it runs a digest-pinned image over every `**/*.md` |
| `task test:unit` | Every Go test except the sandbox conformance suite. Includes decoding and validating every descriptor under `examples/` and the schema-to-spec pins | Every change. Under a minute |
| `task test:tck` | The sandbox conformance suite against the fake adapter, including every fake-adapter mutation | Any change under `tck/`, `docs/spec/`, or to a capability page. Minutes |
| `task test` | `test:unit` and `test:tck` together, as one `go test ./...`. It does not run `validate` or `lint`; those are separate | Before opening a pull request, alongside `validate` and `lint` |
| `task kit:dev KIT=hello` | Builds the frontend from the current source under a unique tag and builds one example Kit against it | Any change under `cmd/frontend/`. Needs Docker with buildx |
| `task kits:dev` | The same for every Kit in `KITS` | A frontend change that touches what every descriptor goes through (decoding, staging, export) |
| `task test:e2e` | Publishes two Kits to a throwaway registry and builds a set over them | A change to set building, `fetch/`, or `assemble/`. Needs Docker; CI does not run it |
| `task tck:kit REF=<ref>` | `kit-tck validate` against a published Kit | When you changed what the artifact rules check and have a real Kit to judge |
| `task tck:runtime:sbx` | The full runtime suite against Docker Sandboxes | Do not run unprompted: it needs a signed-in `sbx`, drives hundreds of sandbox lifecycles, and takes up to two hours |

Narrowing is fine while iterating (`go test ./spec/...`,
`go test ./tck/sandbox/ -run TestEveryStatement`), but the tasks above
are what a reviewer expects to have passed.

Do not iterate on frontend changes by rebuilding `docker/sandbox-kit:3`
locally: BuildKit caches frontend resolution per reference and can keep
dispatching the stale binary. `kit:dev` and `frontend:dev` exist for
exactly this reason.

## Branches

Use semantic branch names in the form `<type>/<short-description>`,
with a type from the commit conventions below and a lowercase,
hyphen-separated description of the change. Examples:
`fix/resolver-cancellation`, `feat/kit-inspection`, and
`docs/branch-naming`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [docker/sandbox-kit-spec](https://github.com/docker/sandbox-kit-spec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
