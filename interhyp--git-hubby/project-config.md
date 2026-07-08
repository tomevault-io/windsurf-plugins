---
trigger: always_on
description: These instructions help AI agents work productively in this **Kubebuilder v4**-based Kubernetes operator that manages GitHub organizations, repositories, and teams via CRDs.
---

# AI Coding Guide for git-hubby

These instructions help AI agents work productively in this **Kubebuilder v4**-based Kubernetes operator that manages GitHub organizations, repositories, and teams via CRDs.

## Architecture & Data Flow
- Operator runs a controller-manager that registers three controllers and webhooks; see [cmd/main.go](cmd/main.go).
- **Core CRDs**: 
  - [api/v1alpha1/organization_types.go](api/v1alpha1/organization_types.go) - GitHub organization management
  - [api/v1alpha1/repository_types.go](api/v1alpha1/repository_types.go) - GitHub repository management
  - [api/v1alpha1/team_types.go](api/v1alpha1/team_types.go) - GitHub team management across organizations
- **Configuration CRDs**:
  - [api/v1alpha1/rulesetpreset_types.go](api/v1alpha1/rulesetpreset_types.go) - Reusable ruleset configurations
  - [api/v1alpha1/webhookpreset_types.go](api/v1alpha1/webhookpreset_types.go) - Reusable webhook configurations
  - [api/v1alpha1/codesecurityconfiguration_types.go](api/v1alpha1/codesecurityconfiguration_types.go) - Standalone code security configurations (referenced by organizations)
- **Controllers delegate to reconcilers** in dedicated packages via factory pattern (see [internal/reconciler/reconcilerfactory/factory.go](internal/reconciler/reconcilerfactory/factory.go)):
  - [internal/reconciler/orgrec](internal/reconciler/orgrec) for organizations - manages org settings, custom properties, rulesets, code security configurations, and actions settings
  - [internal/reconciler/reporec](internal/reconciler/reporec) for repositories - manages repo settings, webhooks, and rulesets
  - [internal/reconciler/teamrec](internal/reconciler/teamrec) for teams - manages team creation, membership, and multi-organization support
- GitHub integration via a cached client factory that supports **multiple GitHub Apps** — each organization can reference its own credentials secret via `spec.githubAppConfig`; see [internal/ghclient/factory.go](internal/ghclient/factory.go), [internal/ghclient/wrapper.go](internal/ghclient/wrapper.go), [internal/ghclient/interface.go](internal/ghclient/interface.go).
  - `CachingGitHubClientFactory` caches credentials per secret name and clients per organization (`cacheKey`); rate-limit state is shared per GitHub App ID.
  - `SecretProviderFunc` now takes a `secretName string` parameter so any secret in the credentials namespace can be fetched on demand.
  - `GetClient(ctx, cacheKey, app v1alpha1.GitHubAppConfig)` and `GetGitHubClientAndCheckRateLimit(...)` accept a `GitHubAppConfig` struct (containing `InstallationId` and `CredentialsSecretName`).
  - When `spec.githubAppConfig` is set on an `Organization`, it takes precedence; otherwise the operator falls back to `spec.githubAppInstallationId` (deprecated) combined with the default secret name from `--app-credentials-secret-name`.
- Status conditions are set consistently via helpers; see [internal/conditions/conditions.go](internal/conditions/conditions.go). Finalizers gate deletion; see reconciler files.
- **Validation-only webhooks** enforce spec rules (e.g., organization custom properties, repository references); see [internal/webhook/v1alpha1/organization_webhook.go](internal/webhook/v1alpha1/organization_webhook.go) and [internal/webhook/v1alpha1/repository_webhook.go](internal/webhook/v1alpha1/repository_webhook.go). **Mutating webhooks have been removed**; labels are now applied in reconcilers. **Note**: Team CRD has no webhook currently.

## Startup Spreading & Parallel Reconciliation

### Startup Spreading Mechanism
The operator implements a sophisticated startup spreading system ([internal/reconciler/spreading](internal/reconciler/spreading)) to prevent GitHub API rate limit exhaustion during pod restarts:

- **Purpose**: Distributes warm-start reconciliations over time to avoid thundering herd problem when all resources reconcile simultaneously after pod restart.
- **Spread Period** (default 5 min): Grace period after startup during which reconciliations may be delayed. Controlled via `STARTUP_SPREAD_PERIOD_MINUTES` env var.
- **Spread Interval** (default 180 min): Time window across which reconciliations are distributed. Controlled via `SPREAD_INTERVAL_MINUTES` env var.
- **Smart Detection**: Only delays reconciliations for:
  - Resources with unchanged spec (`generation == observedGeneration`)
  - Healthy resources (`Ready` condition is `True`)
  - Resources not being deleted
  - Resources with unchanged sub-resource generations (e.g., referenced RulesetPresets, WebhookPresets)
- **Immediate Processing**: Bypasses spreading for:
  - Spec changes (generation mismatch)
  - Unhealthy/degraded resources
  - Deletions
  - Sub-resource changes
- **Implementation**: 
  - `SpreadingManager` created in [cmd/main.go](cmd/main.go) via `spreading.NewDefaultManager()`
  - Factory methods (`CreateForOrg()`, `CreateForRepo()`, `CreateForTeam()`) call `SpreadingManager.Spread()` before creating reconciler
  - Returns `RequiresSpreadError` with calculated delay if spreading is needed
  - Controllers handle this error via `handleRequeueError()` in [internal/controller/shared.go](internal/controller/shared.go)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Interhyp/git-hubby](https://github.com/Interhyp/git-hubby) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
