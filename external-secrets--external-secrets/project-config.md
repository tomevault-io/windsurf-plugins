---
trigger: always_on
description: Kubernetes operator that synchronizes secrets from external providers (AWS Secrets Manager, Vault, GCP Secret Manager, Azure Key Vault, etc.) into Kubernetes Secrets.
---

# External Secrets Operator

Kubernetes operator that synchronizes secrets from external providers (AWS Secrets Manager, Vault, GCP Secret Manager, Azure Key Vault, etc.) into Kubernetes Secrets.

## Build and Test

Use `make` targets — refer to the Makefile for available commands. Do not run `go test`, `golangci-lint`, or `helm` directly.

You must run `make test && make check-diff` before the PR is ready. (See also section Non-Obvious patterns for more explanations about the tests)

## Project Layout

Single binary built from `main.go`. The **controller** reconciles ExternalSecrets into K8s Secrets. The **webhook** (validates and defaults CRDs) and **certcontroller** (manages webhook TLS) are subcommands registered via `rootCmd.AddCommand()`.

Multi-module repo: `apis/`, `runtime/`, `e2e/`, and each `providers/v1/*/` have their own `go.mod`.

## Non-Obvious Patterns

- `make reviewable` is the gate for PRs. Run it, not individual checks.
- Helm chart is the source of truth for deploy manifests. `make manifests` generates static YAML from it.
- Provider docs `{% include %}` reusable YAML snippets from `docs/snippets/` (`macros` plugin). AWS authentication is documented once on the standalone `docs/provider/aws-access.md` page; the per-service pages (`aws-secrets-manager.md`, `aws-parameter-store.md`) link to it rather than transcluding it.
- CRD tests use snapshot testing. Run `make test.crds.update` to update snapshots after CRD changes.
- `make update-deps` updates dependencies across all modules at once.
- Add a `git notes add HEAD` entry on every non-trivial commit. Record key design decisions, trade-offs, and gotchas. Queryable via `git notes show <sha>`.
- If you discover a non-obvious pattern while implementing, add it here before the PR is merged. Keep entries general — applicable across the codebase, not specific to one provider or feature.
- Never edit `zz_generated.*` files by hand. They are owned by controller-gen. Modify the source types and run `make generate` (included in `make reviewable`).
- After everything is committed - **ALWAYS RUN `make check-diff`** - this is the first step where PRs fall apart that LLMs forget - there are a lot of generated code outside of the main `make reviewable` spec like helm chart tests, docs, etc.
- 

## Adding a Provider

A provider is its own Go module under `providers/v1/<name>/` with no build tags on the package itself.
Build tags live in `pkg/register/<name>.go`.

### API types

- New spec goes in `apis/externalsecrets/v1/secretstore_<name>_types.go`.
- Add a one-line slot to the discriminator union in `apis/externalsecrets/v1/secretstore_types.go`
  (the `SecretStoreProvider` struct). The JSON tag is the provider name; `apis/externalsecrets/v1/provider_schema.go`
  resolves it from the first JSON key of the marshaled union.
- Auth: nested `*<Name>Auth` struct. Multi-method auth uses `+kubebuilder:validation:MaxProperties=1`. Selector types
  are `esmeta.SecretKeySelector` and `esmeta.ServiceAccountSelector`.
- CA: include `CABundle []byte` and `CAProvider *CAProvider` if the backend speaks TLS.
- v1 API is frozen by default. Net-new provider slots are fine

### Runtime helpers (use these, do not roll your own)

- `runtime/esutils/resolvers.SecretKeyRef(ctx, kube, storeKind, namespace, ref)` for credential resolution. It enforces
  `ClusterSecretStore` vs `SecretStore` namespace scoping. Pass `store.GetKind()` and the ES namespace.
- `runtime/esutils.FetchCACertFromSource(ctx, esutils.CreateCertOpts{...})` for CA bundles.
- `runtime/esutils.ValidateSecretSelector` / `ValidateReferentSecretSelector` / `ValidateServiceAccountSelector` for spec validation.
- `runtime/esutils/metadata` for parsing `PushSecretMetadata` into a typed spec.
- `runtime/constants` for metric label values.

### `SecretsClient` contract

Defined at `apis/externalsecrets/v1/provider.go`. All eight methods are mandatory; `Close` may be a no-op.

- Return `esv1.NoSecretErr` from `GetSecret` when the secret is missing. The reconciler depends on this for `deletionPolicy`.
- Set `Capabilities()` honestly: `SecretStoreReadOnly`, `SecretStoreWriteOnly`, or `SecretStoreReadWrite`. Read-only
  providers still implement Push/Delete but return a sentinel error! Do _NOT_ return `nil`!
- `gjson` is the conventional path extractor for `ref.Property` on JSON payloads.

### Caching (skip unless construction is expensive)

- Per-Provider client cache: `runtime/cache.Must[T](size, cleanup)`. Keyed by `cache.Key{Name, Namespace, Kind}`,
  versioned by `store.GetObjectMeta().ResourceVersion`. Use this for OIDC, vault leases, token exchange, etc. Default to no cache.
- Per-secret cache (in the SecretsClient): `expirable.LRU[string, []byte]` with a user-facing `CacheConfig{TTL, MaxSize}`
  field on the spec.

### Feature flags

Pipeline: helm value to deployment `extraArgs` to cmd flag to `feature.Register` to `Initialize()`.

- Register flags from the provider's `init()` using `runtime/feature.Feature{Flags, Initialize}`.
- `cmd/controller/root.go` collects them and runs `Initialize` after manager startup.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [external-secrets/external-secrets](https://github.com/external-secrets/external-secrets) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
