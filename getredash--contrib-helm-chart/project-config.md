---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A single community Helm chart (`charts/redash`) that deploys [Redash](https://github.com/getredash/redash) on Kubernetes. There is no application code here — everything is Helm templates, values, and generated docs. Upstream is `github.com/getredash/contrib-helm-chart`; the published repo index lives on the `gh-pages` branch.

## Commands

Dependencies must be fetched before `helm template`/`install` will work (`helm lint` only warns):

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
cd charts/redash && helm dependency build .
```

```bash
helm lint charts/redash                 # required to pass by CONTRIBUTING
helm template test charts/redash        # renders fine with stock values.yaml
```

Nothing gates rendering with default values — `values.yaml` ships placeholder `redash.secretKey: test` / `cookieSecret: test`, and the `required` guard in `NOTES.txt` is effectively defanged (its `or` chain falls through to the empty `externalPostgreSQLSecret` dict, and `required` only rejects nil and the empty string, so `{}` passes). Values are required for a *working install*, not for rendering. The canonical minimal working value set is the `test-values.yaml` heredoc in `.github/workflows/ci.yml`.

Full functional test = the minikube flow in `.github/workflows/ci.yml` (install → `helm test` → delete → reinstall → upgrade → `helm test`, across four Kubernetes minor versions). To reproduce one iteration locally you need a cluster; then follow that same sequence.

Regenerating the chart README (see the trap below):

```bash
cd charts/redash && helm-docs --dry-run | prettier --parser markdown > README.md
```

## Docs are generated — do not hand-edit

`charts/redash/README.md` is produced from `charts/redash/README.md.gotmpl` plus the `# key -- description` comments in `values.yaml`. Edit the source, then regenerate.

Two traps:

- **The pre-commit hook is wrong for the current layout.** `.pre-commit-config.yaml` runs `helm-docs --dry-run > README.md` from the repo root, but the chart moved out of the top level, so that writes the *root* README rather than `charts/redash/README.md`. Run helm-docs from inside `charts/redash`.
- **Regenerate from `charts/redash`, and check the diff.** The README had drifted from the template before (a stale version line, plus value rows for the `postgresqlMigration` block 4.0.0 removed), because the hook above was writing to the wrong file. Content hand-added to the generated README is silently lost on the next regeneration — the `### From 3.1 to 3.2` upgrade notes had to be moved into `README.md.gotmpl` to survive.

The root `README.md` deliberately uses absolute GitHub URLs because it is synced to `gh-pages`. `templates/` is in `.prettierignore`.

## Release mechanics

`charts/redash/Chart.yaml` `version` must be bumped in every change that should be published — the `chart-releaser` job only runs on pushes to `master` and only publishes versions it hasn't seen (commit `a56f7a8` exists purely to bump the version and trigger it). Chart version is independent of `appVersion` (the Redash release); it follows semver on its own. Breaking changes go in `CHANGELOG.md` and in the "Upgrading" section of `README.md.gotmpl`.

## Architecture

Five workloads render from one shared env helper:

- `server-deployment.yaml` — the web server, fronted by `service.yaml` / optional `ingress.yaml`.
- `worker-deployment.yaml` — **one Deployment per key under `workers.*`** (`adhoc`, `scheduled`, `generic`), produced by a single `range`. Each worker's config is `mergeOverwrite (deepCopy .Values.worker) $config`, so `worker.*` holds shared defaults and `workers.<name>.*` overrides them. Adding a worker means adding a values key, not a template.
- `scheduler-deployment.yaml` — `strategy: Recreate`, single instance.
- `hook-migrations-job.yaml` — a `post-install,post-upgrade` hook Job running `create_db`; this is how schema migrations happen.
- `tests/test-connection.yaml` — the `helm test` pod, curls the service and greps for "Welcome to Redash".

### `redash.env` in `_helpers.tpl`

An ~80-entry block mapping `redash.*` values to `REDASH_*` env vars, shared by every component. Key points:

- The block is delimited by `## Start primary Redash configuration` / `## End primary Redash configuration` markers that mirror the same markers in `values.yaml` and `secrets.yaml`. Keep the three in sync.
- `CONTRIBUTING.md` says to regenerate this with `python scripts/update-env-config.py` — **that script no longer exists in the repo**. Adding a Redash setting today means hand-editing `values.yaml` (with its helm-docs comment) and `_helpers.tpl`, plus `secrets.yaml` if the value is a secret.
- Secret-valued settings are additionally gated on `redash.selfManagedSecrets` and `redash.existingSecret`; follow the existing `{{- if not .Values.redash.selfManagedSecrets }}` pattern rather than inventing a new one.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [getredash/contrib-helm-chart](https://github.com/getredash/contrib-helm-chart) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
