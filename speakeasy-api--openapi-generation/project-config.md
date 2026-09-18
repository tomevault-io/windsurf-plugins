---
trigger: always_on
description: Before creating or publishing any public artifact — PR titles/bodies, comments, review replies, commit messages, changesets, branch/file names, code comments and string literals, fixtures, docs — follow `.claude/skills/public-repo-communication/SKILL.md`: describe the change so it stands alone; never expose customer-derived identifiers, private paths/trackers, or workflow provenance.
---

Before creating or publishing any public artifact — PR titles/bodies, comments, review replies, commit messages, changesets, branch/file names, code comments and string literals, fixtures, docs — follow `.claude/skills/public-repo-communication/SKILL.md`: describe the change so it stands alone; never expose customer-derived identifiers, private paths/trackers, or workflow provenance.

## Definitions

### Targets

A `TARGET` is one of:
- cli
- csharp
- go
- javav2
- mcp-typescript
- php
- postman
- pythonv2
- ruby
- terraform
- typescriptv2
- unity

Their associated *templates* directory is located at `templates/templates/<TARGET>`

### Variants

A `VARIANT` is one of:
- basic-http
- client-credentials
- client-credentials-basic
- custom-http
- no-servers
- no-zod
- oauth2-password
- primary
- quaternary
- relative-servers
- review
- secondary
- security-options
- tertiary

Their associated *configuration* is located at: `tests/config/<VARIANT>/<TARGET>/.speakeasy/gen.yaml`

Note: in the following command `TARGET=review make test-typescriptv2`, the term `TARGET` is misleading and should have been called `VARIANT` instead (see Makefile#L35)

### Specs

- The main test specification is `tests/specs/uber.yaml` and directly drives the `primary` variant.
- Additional test fragments may be applied as configured in `tests/specs/fragments/`.
- Variants apply *overlay* files as defined in `tests/overlays`.


## How to debug an issue:

Workout what the issue is in the generator source code and fix it.

You can see what the issue is and verify you've fixed it by running

go run cmd/generate/main.go -s {{SOME_TMP_DIR}}/openapi.yaml -o {{SOME_TMP_DIR}} -l {{TARGET}} --license agpl-3.0-only

Note: first run will automatically create a gen.yaml under .speakeasy with default values.

## License election for local generation

Every generation needs an explicit license election. Run `./zero` once per
checkout: `./zero --license agpl-3.0-only` (OSS / agents) or `./zero` after
`speakeasy auth login` (commercial token). It writes a gitignored `.env` that
`make`, `mise run`, and `scripts/*.sh` load. When running `go run
cmd/generate/main.go` or `cmd/regen` directly, pass `--license agpl-3.0-only`
or have `SPEAKEASY_LICENSE_TOKEN` / `SPEAKEASY_GENERATED_LICENSE` in the
environment (a mise-activated shell loads `.env`).

Output written inside this repository (`zSDKs/`, `testSDKs/`) never carries
license headers, `LICENSE`, or `NOTICE` regardless of the election — the
repository LICENSE covers it — so regenerating fixtures produces the same
bytes for every contributor. Never commit `.env` or a license token.

### Regenerating an already-bootstrapped SDK

If the SDK directory already has a `.speakeasy/gen.yaml` (and optionally a
`workflow.yaml`), use `cmd/regen` which infers language and schema automatically:

```
go run cmd/regen/main.go {{SDK_DIR}}
```

- **Language** is inferred from the gen.yaml language key
- **Schema** is inferred from workflow.yaml, then by scanning for `openapi*`
  files in the SDK dir and up to 3 parent directories
- Override with `-s spec.yaml` or `-l go` if needed
- Patch gen.yaml before generation: `--set key=value` (e.g. `--set go.version=2.0.0`)

**Note:** `go run -C` changes the process cwd, so `.` resolves to the module
dir, not your shell cwd. Use absolute paths, or build a binary first:
`go build -o /tmp/regen ./cmd/regen && cd {{SDK_DIR}} && /tmp/regen .`


## You've fixed the issue, now run the tests

### SDK Testing

Run the tests for the SDKs:
  - `TARGET=review make test-{{TARGET}}` (outputs to `zSDKs/sdk-{{TARGET}}`)
  - `TARGET=primary make test-{{TARGET}}` (outputs to `testSDKs/sdk-{{TARGET}}-primary`)

You can see how tests are configured under `tests/*` eg the spec for the primary,secondary etc tests is in `tests/specs/uber.yaml` and for review tests it's in `tests/specs/review.yaml`. The gen.yaml for {{TARGET}} primary is
`tests/config/primary/{{TARGET}}/.speakeasy/gen.yaml`.

### Test Fragments

Isolated test cases can be added as OpenAPI Specification document fragments under `tests/specs/fragments/primary/` (or `tests/specs/fragments/uber/` for shared fragments). Fragments get merged into the base spec during build via `scripts/build-openapi-document.sh`.

- Use **kebab-case** filenames (e.g., `pagination-nullable-limit.yaml`).
- Include a `description` on the operation or schema under test explaining what the fragment is intended to check and why. Descriptions should be language-agnostic, focusing on the OpenAPI construct being tested rather than language-specific generated output.
- Ensure parameter and property names align with their testing purpose or defined characteristics (e.g., a schema with nullable typing should be named `NullableObject`).
- Include the standard security block matching other fragments.

### API Test Service


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [speakeasy-api/openapi-generation](https://github.com/speakeasy-api/openapi-generation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
