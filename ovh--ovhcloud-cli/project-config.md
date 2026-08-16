---
trigger: always_on
description: Guidance for AI coding tools working in this repository (GitHub Copilot, Claude, …).
---

# Copilot / AI agent instructions

Guidance for AI coding tools working in this repository (GitHub Copilot, Claude, …).
Keep it accurate: if you change a convention or helper, update this file.

## What this is

`ovhcloud` — a Go CLI (cobra) wrapping the OVHcloud public APIs (v1 and v2). Each API
"universe" (cloud, domain, baremetal, vps, me, …) is exposed as a top-level command.
Module path: `github.com/ovh/ovhcloud-cli`. Entry point: `cmd/ovhcloud/main.go` → `cmd.Execute()`.

## Commands (always run before handing back)

```bash
make fmt          # gofmt — mandatory
make build        # CGO_ENABLED=0 build of ./cmd/ovhcloud → ./ovhcloud
go test ./...     # all tests must pass
make doc          # regenerate doc/ (see Docs below)
```

Refresh a **v1** OpenAPI schema: `make schemas UNIVERSE=<name>` (e.g. `cloud`, `domain`, `vps`).
There is **no** automated refresh for v2 schemas (see "API schemas" below).

## Architecture — the two-file pattern

Adding or changing a command almost always touches exactly two files:

| Layer | Location | Responsibility |
|-------|----------|----------------|
| **Command** | `internal/cmd/<universe>.go` | cobra definitions, flags, arg validation. **No business logic.** |
| **Service** | `internal/services/<universe>/*.go` | HTTP calls + response handling. **No printing.** |
| Display | `internal/display/` | ALL output/formatting (JSON/YAML/interactive/custom). |
| OpenAPI | `internal/openapi/` | reads embedded schemas to build request-body skeletons and filter editable fields. |
| Assets | `internal/assets/api-schemas/*.json` | embedded OpenAPI schemas (`//go:embed`). |

### Command registration

Each `internal/cmd/<universe>.go` has a `func init()` that builds its command tree and ends
with `rootCmd.AddCommand(<universe>Cmd)`. Cobra wires everything at startup — there is no
central registry to edit.

**Cloud is the exception**: `internal/cmd/cloud_project.go` `init()` builds `cloudCmd`, then calls
every `initCloud<Feature>Command(cloudCmd)`, and finally `rootCmd.AddCommand(cloudCmd)`. A new cloud
feature = a new `initCloud<Feature>Command` **that you must add to that block**. Storage sub-commands
are wired one level deeper in `internal/cmd/cloud_storage.go`.

Reference examples to copy from:
- v1 resource with nested sub-commands: `internal/services/cloud/cloud_storage_block.go` + `internal/cmd/cloud_storage_block.go`
- **v2** resource (list/get/create/edit/delete + action): `internal/services/cloud/cloud_managed_rancher.go` + `internal/cmd/cloud_managed_rancher.go`

## Core helpers — prefer these over hand-rolling

Service layer (`internal/services/common`):
- `common.ManageListRequestNoExpand(endpoint, columns, flags.GenericFilters)` — list.
- `common.ManageObjectRequest(endpoint, id, template)` — get (renders a `.tmpl`).
- `common.CreateResource(cmd, schemaPath, endpoint, example, spec, schemaBytes, requiredFields)` — create.
- `common.EditResource(cmd, schemaPath, endpoint, spec, schemaBytes)` — edit.
- `httpLib.Client.Get/Post/Delete(...)` — raw calls for simple actions only.
- `getConfiguredCloudProject()` (cloud package) — resolve the target cloud project.

Command layer (`internal/cmd/`):
- `withFilterFlag(cmd)` — adds `--filter` to a list command.
- `addParameterFileFlags(cmd, skipInit, schemaBytes, path, method, defaultExample, replaceFn)` — adds `--from-file` and `--init-file`.
- `addInteractiveEditorFlag(cmd)` — adds `--editor`.
- `markFlagsMutuallyExclusive(cmd, "from-file", "editor")`.

Output (`internal/display`):
- `display.OutputInfo(&flags.OutputFormatConfig, details, message, params...)` — success/info.
- `display.OutputError(&flags.OutputFormatConfig, message, params...)` — errors.
- **Never** use `fmt.Print*` to talk to the user. Always go through `display`.

## Conventions (enforced in review)

- **Name commands after the API endpoint.**
- **Output only via `internal/display`** — never `fmt.Println`.
- **`url.PathEscape` every identifier** injected into a URL path.
- **List column order: `id`, `name`, `region`, `type` first** when those fields exist.
  Column mapping syntax is `"jsonPath alias"`, e.g. `"currentState.name name"`.
- **Declare a cobra flag for every field** of a create/edit `Spec` struct (kebab-case flag names).
- **Create/edit UX rule** (CONTRIBUTING.md): if the request body has **> 5 parameters** or **more
  than one level of nesting**, the command MUST offer `--editor`, `--from-file`, and `--init-file`
  (via the helpers above).
- **`--editor`/`--from-file` must stay wired**: these flags only take effect through
  `common.CreateResource`/`EditResource` (they set globals read nowhere else). If a create/edit
  handler builds the body by hand and calls `httpLib.Client` directly, those flags become silent
  no-ops — route through `CreateResource`/`EditResource` instead, or drop the flags.
- **Async polling**: when waiting on a task/operation, a task/sub-resource reported in `ERROR` is
  **logged and waiting continues** (transient errors resolve backend-side); only a top-level
  resource/operation error status is fatal. See `internal/services/cloud/utils.go`.
- Keep changes small and single-purpose; avoid commands that fan out into many HTTP calls.

## API schemas — what the JSON files are for


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ovh/ovhcloud-cli](https://github.com/ovh/ovhcloud-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
