---
trigger: always_on
description: Polls the Tailscale (or Headscale) control plane and exports OpenTelemetry-native metrics and logs
---

# tailscale2otel

Polls the Tailscale (or Headscale) control plane and exports OpenTelemetry-native metrics and logs
over OTLP, tuned for Grafana Cloud. Single static Go binary. `README.md` is the user-facing pitch;
`docs/` is the published site.

This repository is PUBLIC. Keep lab-specific names, addresses, identifiers, credentials and
observability captures out of every tracked file, `backlog/` included - write the shape, not the
instance. Aggregate counts, timings and structural findings are fine.

## Task interface

`just check` is the gate. `just ci` adds the goreleaser cross-compile and the container image +
smoke legs. `just --list` is the authoritative recipe list; `just --show <recipe>` is what one runs.

- `just setup` once per clone: pinned `golangci-lint`, `govulncheck`, the two Helm generators, and
  `core.hooksPath` at `.githooks`. Git cannot run anything on clone, so nothing installs it for you.
- `just lint` and `just vuln` assert the invoked binary against the justfile pin before running. A
  local lint result is evidence only when that assertion passed; `just setup` repairs a stale tool.
- `prune-rules` and `bump-major` are `[confirm]` recipes that mutate outside the tree. Never pass
  `--yes` or `JUST_YES=1`; run `just` with stdin from `/dev/null`.
- `just gen <family>...` and `just gen-<family>` are the same thing. Test failure messages and CI
  job names use the spaced spelling.
- `just review-sharded` is the CodeRabbit path here - a whole-repo review exceeds the transport
  limit. See `docs/coderabbit-sharded-review.md`.
- `just run` starts the exporter against `config.yaml`; `otlp.protocol: stdout` prints signals to
  the console for local debug with no backend.

## Generated artifacts

Every committed generated artifact has a `gen-<family>` recipe and a CI fail-on-diff gate. `just gen`
reproduces the set; the `gen` group in `just --list` is the only place the set is written down.
`.githooks/pre-commit` regenerates only what your *staged* changes invalidate and re-stages it.

- Never hand-edit between `<!-- BEGIN GENERATED -->` and `<!-- END GENERATED -->` in
  `docs/metrics.md`. Prose outside the markers is safe.
- `internal/catalog/signal_dispositions.json` is the one generated-adjacent file you do NOT blindly
  regenerate, and regenerating it cannot turn a red coverage gate green.

## Grafana delivery

- **Pushing alert rules to Rob's Grafana stack is pre-authorized - do not ask.** That covers
  `gcx resources push -p deploy/alerts/grafana-managed` and deleting a rule the repo no longer
  ships. It does NOT extend to mutating the tailnet itself.
- **Do NOT push DASHBOARDS with `gcx`.** They are delivered into `m7kni/gc-gitsync-m7kni` by
  `.github/workflows/grafana-sync.yml`; an API push is an out-of-band edit the next sync undoes.
- Nothing under `deploy/grafana` or `deploy/alerts` is hand-maintained. The project is Grafana v2 /
  Grafana 13+ only and will never ship a Classic export.

## Modules and CI

Root module `github.com/rknightion/tailscale2otel/v5` plus four CI-only tool modules
(`tools/{configcheck,metricscatalog,apidrift,promqlcheck}`). **No `go.work`**, deliberately, so a
tool module can never affect the main module's build.

- `go test -race ./...` at the root is NOT "the test suite": it stops at the root module boundary.
  ci.yml's `module-verify` matrix covers the tool modules (build, vet, race test, `go mod tidy`
  diff, `govulncheck`) and is in `ci-success.needs`; the `lint` matrix covers all five modules.
  `internal/ci/workflowcontract_test.go` fails if a module drops out of either matrix, or if
  `module-verify` stops running a leg.
- `tools/promqlcheck` is the one tool module with no `replace ../..` - it needs nothing from the
  root module - and it pins `golang.org/x/text` against a transitive vulnerability (`GO-2026-5970`),
  so Renovate must keep that in step with the root module's. Invoke it as
  `go run -C tools/promqlcheck . -root "$PWD"`.
- A breaking change that cuts a new MAJOR needs the Go module path moved first: run `just bump-major`
  and land it on `main` before merging the release PR. release-please does not maintain the path,
  and a major tagged against a stale `/vN` fails the GoReleaser binaries job.
  `TestModulePathMatchesReleaseVersion` (`internal/config/modulepath_test.go`) catches it.
- Conventional Commits: Renovate and the release tooling assume `type(scope): subject`.

## Config and secrets

- Layered: built-in defaults < optional YAML file < environment. Passing no `-config` flag runs from
  defaults + env alone. `docs/configuration.md` is the full key reference; `config.example.yaml` is
  the committed starter.
- Env convention is `TS2OTEL_` + the dotted key path with `__` between levels, e.g.
  `tailscale.auth.oauth.client_secret` -> `TS2OTEL_TAILSCALE__AUTH__OAUTH__CLIENT_SECRET`. Env
  overrides the file. Keep secrets in env vars; they never need to appear in YAML.
- Prefer OAuth (`auth.method: oauth`, auto-refreshing) over API keys (expire in 90 days or less,
  user-bound; config WARNs about this).
- `config.local.yaml`, `config.smoke.yaml`, `config.lowlog.yaml`, `.env*`, `.secrets/`,
  `checkpoints.json` and `.capture/` are gitignored.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rknightion/tailscale2otel](https://github.com/rknightion/tailscale2otel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
