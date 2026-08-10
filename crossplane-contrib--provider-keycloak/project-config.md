---
trigger: always_on
description: Instructions and context for AI coding agents working on provider-keycloak
---


# Agent Instructions for provider-keycloak

This page collects context and instructions for AI coding agents (GitHub Copilot,
Cursor, Claude, etc.) working on the provider-keycloak repository.

## What This Repository Is

`provider-keycloak` is a [Crossplane](https://crossplane.io/) provider that lets
you manage [Keycloak](https://www.keycloak.org/) resources as Kubernetes custom
resources. It is generated with [Upjet](https://github.com/crossplane/upjet) on
top of the [Keycloak Terraform Provider](https://github.com/keycloak/terraform-provider-keycloak).

**One-line flow:**
```
Keycloak Terraform Provider  →  Upjet (code generator)  →  Crossplane provider  →  Kubernetes CRDs
```

Users declare Keycloak resources as YAML (`spec.forProvider` maps to Terraform arguments), and the provider reconciles them continuously against a live Keycloak instance.

## Repository Layout

```
apis/               Crossplane API types (generated + hand-authored)
cmd/                provider and generator entry points
config/             Upjet resource configuration (external names, references, cross-resource refs)
docs/               Hugo (hextra) documentation site
examples/           Hand-authored example manifests for each managed resource
examples-generated/ Auto-generated example manifests (do not edit by hand)
package/crds/       Generated CRD YAML files (source of truth for field schemas)
internal/           Internal controller and reconciler logic
generate/           Generation scripts
cluster/            Uptest end-to-end test manifests and setup
dev/                Local development environment scripts
scripts/            Utility scripts
```

## Core Concepts

- **ProviderConfig** – holds connection details for a Keycloak instance (URL,
  client ID, credentials secret reference).
- **Managed Resources** – Kubernetes CRDs that map 1:1 to Keycloak objects.
  `spec.forProvider` maps to Terraform resource arguments.
- **Reconciliation** – the provider controller continuously ensures Keycloak
  matches the desired state expressed in `spec.forProvider`.
- **External Name** – the Keycloak-side identifier wired in `config/external_name.go`.
  This is the ID or name that Keycloak assigns to the resource.
- **References** – cross-resource references (e.g., `realmIdRef`) are configured
  in `config/<group>/config.go`. They wire one managed resource's external name
  into another resource's field.

## Key Files for Common Tasks

| Task | File(s) |
|------|---------|
| Add a new resource | `config/external_name.go`, `config/<group>/config.go` |
| Change reference resolution | `config/<group>/config.go` |
| Update docs for a resource | `docs/content/docs/using/resources/<resource>.md` |
| Add/update an example manifest | `examples/<group>/<resource>.yaml` |
| Modify CRD generation | `generate/*.go`, run `make generate` |
| Run unit tests | `make test` |
| Run e2e tests | `make e2e`, see `cluster/test/cases.txt` for covered resources |
| Regenerate llms.txt/llms-full.txt | `make docs-gen` |
| Verify docs freshness | `make docs-freshness-check` |

## Code Generation

Always run `make generate` after changing `config/` to regenerate CRDs and Go types. **Never** edit files in `apis/` or `package/crds/` by hand — they are generated outputs.

The generation pipeline:
1. `generate/main.go` calls Upjet with the Terraform provider schema.
2. Upjet writes Go type definitions into `apis/<group>/<version>/`.
3. `make generate` runs `go generate ./...` which invokes controller-gen to write CRDs into `package/crds/`.

## Testing

- Unit tests: `make test`
- E2E tests: `make e2e` (requires a running Keycloak and Crossplane cluster)
- E2E coverage is limited to resources listed in `cluster/test/cases.txt`

The E2E suite uses [uptest](https://github.com/crossplane/uptest). Only resources
explicitly listed in `cluster/test/cases.txt` receive e2e coverage.

## Adding a New Resource

1. Add an entry to `config/external_name.go` (the external name is the Keycloak-assigned ID).
2. Create or update `config/<group>/config.go` to configure references and any
   custom behaviors.
3. Run `make generate` to regenerate CRDs and Go types.
4. Add a hand-authored example to `examples/<group>/<resource>.yaml`.
5. Optionally add a docs page to `docs/content/docs/using/resources/<resource>.md`.

To allow a resource to be imported/observed by its properties (avoiding 409 on create),
wire its `external_name.go` entry to a `lookup.BuildIdentifyingPropertiesLookup` config
in the `config/<group>` package (see `config/openidclient/config.go` for an example).

## Cross-Resource References

References are wired in `config/<group>/config.go` using `r.References` on the
Upjet resource configuration. The reference resolver fills in the referenced
resource's external name at reconciliation time. Example pattern:

```go
r.References["realm_id"] = config.Reference{
    TerraformName: "keycloak_realm",
}
```

## Documentation Site

The docs use [Hugo](https://gohugo.io/) with the [Hextra](https://imfing.github.io/hextra/) theme.

```bash
cd docs && hugo server --buildDrafts   # local preview
make docs-gen                          # regenerate llms.txt and llms-full.txt
make docs-freshness-check             # CI: verify llms.txt is current
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [crossplane-contrib/provider-keycloak](https://github.com/crossplane-contrib/provider-keycloak) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
