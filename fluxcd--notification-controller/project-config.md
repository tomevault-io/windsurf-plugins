---
trigger: always_on
description: Guidance for AI coding assistants working in `fluxcd/notification-controller`. Read this file before making changes.
---

# AGENTS.md

Guidance for AI coding assistants working in `fluxcd/notification-controller`. Read this file before making changes.

## Contribution workflow for AI agents

These rules come from [`fluxcd/flux2/CONTRIBUTING.md`](https://github.com/fluxcd/flux2/blob/main/CONTRIBUTING.md) and apply to every Flux repository.

- **Do not add `Signed-off-by` or `Co-authored-by` trailers with your agent name.** Only a human can legally certify the DCO.
- **Disclose AI assistance** with an `Assisted-by` trailer naming your agent and model:
  ```sh
  git commit -s -m "Add support for X" --trailer "Assisted-by: <agent-name>/<model-id>"
  ```
  The `-s` flag adds the human's `Signed-off-by` from their git config — do not remove it.
- **Commit message format:** Subject in imperative mood ("Add feature X" instead of "Adding feature X"), capitalized, no trailing period, ≤50 characters. Body wrapped at 72 columns, explaining what and why. No `@mentions` or `#123` issue references in the commit — put those in the PR description.
- **Trim verbiage:** in PR descriptions, commit messages, and code comments. No marketing prose, no restating the diff, no emojis.
- **Rebase, don't merge:** Never merge `main` into the feature branch; rebase onto the latest `main` and push with `--force-with-lease`. Squash before merge when asked.
- **Pre-PR gate:** `make tidy fmt vet && make test` must pass and the working tree must be clean after codegen. Commit regenerated files in the same PR.
- **Flux is GA:** Backward compatibility is mandatory. Breaking changes to CRD fields, status, CLI flags, metrics, or observable behavior will be rejected. Design additive changes and keep older API versions round-tripping.
- **Copyright:** All new `.go` files must begin with the boilerplate from `hack/boilerplate.go.txt` (Apache 2.0). Update the year to the current year when copying.
- **Spec docs:** New features and API changes must be documented in `docs/spec/` under the current version — `v1/receivers.md` for Receiver, `v1beta3/alerts.md` and `v1beta3/providers.md` for Alert and Provider. Update the relevant file in the same PR that introduces the change.
- **Tests:** New features, improvements and fixes must have test coverage. Add unit tests in `internal/controller/*_test.go` and other `internal/*` packages as appropriate. Follow the existing patterns for test organization, fixtures, and assertions. Run tests locally before pushing.

## Code quality

Before submitting code, review your changes for the following:

- **No secrets in logs or events.** Anything derived from a Secret (tokens, passwords, webhook URLs with embedded secrets) must be scrubbed via `fluxcd/pkg/masktoken` before logging or surfacing in conditions. Never log the receiver webhook path at info level — it is effectively a secret.
- **No unchecked I/O.** Close HTTP response bodies and file handles in `defer` statements. Check and propagate errors from I/O operations.
- **Bounded request bodies.** Both HTTP servers enforce `maxRequestSizeBytes` (3 MiB). Always read through `io.LimitReader` when extending handlers. Do not introduce new readers without bounds.
- **Single HTTP client for notifiers.** `internal/notifier/client.go` is the only sanctioned way to make outbound HTTP calls from a notifier. It handles proxies, TLS, retries, and response validation. Do not add your own HTTP client or retry loops.
- **No hardcoded defaults for security settings.** TLS verification must remain enabled by default; proxy settings come from user-provided secrets. Receiver signature verification must never be short-circuited.
- **Error handling.** Wrap errors with `%w` for chain inspection. Do not swallow errors silently. Return actionable error messages that help users diagnose the issue without leaking internal state.
- **No duplicate rate limiting.** The event server already applies a token-bucket rate limiter keyed by event fingerprint. Do not add a second layer of deduplication.
- **Concurrency safety.** Do not introduce shared mutable state without synchronization. Both HTTP servers and the reconcilers run concurrently.
- **No panics.** Never use `panic` in runtime code paths. Return errors and let the reconciler or handler handle them gracefully.
- **Minimal surface.** Keep new exported APIs, flags, and environment variables to the minimum needed. Every export is a backward-compatibility commitment.

## Project overview

notification-controller is the eventing edge of the [Flux GitOps Toolkit](https://fluxcd.io/flux/components/). It reconciles three custom resources under `notification.toolkit.fluxcd.io` — `Provider`, `Alert`, and `Receiver` — and runs two HTTP servers alongside the controller manager:

- An inbound **event sink** (default `:9090`) that ingests `eventv1.Event` payloads from the other Flux controllers and dispatches them to external notifier backends (Slack, MS Teams, PagerDuty, Git commit status, …) according to matching `Alert` resources.
- An inbound **webhook receiver** (default `:9292`) that translates third-party webhooks (GitHub, GitLab, Harbor, image registries, CDEvents, …) into reconcile requests on Flux objects.

It does not reconcile source or workload state itself.

## Repository layout


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fluxcd/notification-controller](https://github.com/fluxcd/notification-controller) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
