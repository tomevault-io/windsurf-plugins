---
trigger: always_on
description: validates every part as a slug, so a malformed reference can never reach
---

# Working on Cubeship

Self-hosted PaaS for one VPS. Go, module `cubeship`, no external services
beyond Docker. `README.md` is the operator's intro; this file is for
whoever (or whatever) edits the code.

**This file is the part that applies to every change.** How each area
actually works is in `docs/design/`, one file per area — indexed below,
and worth opening before touching the area it covers.

## Before you commit

```sh
make check
```

gofmt, `go vet` (including the build-tagged integration test, which a
plain `go vet ./...` never compiles), and the unit tests under `-race`.
`make help` lists the rest.

Commit messages: imperative subject saying what changes, body only when
the *why* isn't obvious. **Never credit an AI agent** — no
`Co-Authored-By: Claude`, no "generated with" footer, not in commits, PRs
or code comments.

Work happens on `master`, in the repository root. No worktrees.

## Where the rest is written down

Read the file for the area before changing it. Each one carries the
reasoning as well as the rule, including the things that were tried the
other way and were wrong — which is the part that stops a decision being
re-made by accident.

| Opening | Read |
| --- | --- |
| Anything under `web/` | [dashboard.md](docs/design/dashboard.md) — the layers, the navigation, the components, the look |
| Anything under `site/` | [site.md](docs/design/site.md) — cubeship.dev: the landing page, the docs, and where `install.sh` comes from |
| `internal/node`, `internal/mesh`, `internal/worker` | [cluster.md](docs/design/cluster.md) — placement, replicas, the agent, the network between machines, the one front door, autoscaling, limits |
| `internal/app` | [deploys.md](docs/design/deploys.md) — where an image comes from, the two builders, the GitHub App, who may build, what deleting takes |
| `internal/datastore` | [datastores.md](docs/design/datastores.md) — the engines, attaching, exposing, what is fixed after creation |
| `internal/objectstore` | [object-storage.md](docs/design/object-storage.md) — managed MinIO, linked S3, what a folder is |
| `internal/backup` | [backups.md](docs/design/backups.md) — dumps, the schedule, restoring |
| `internal/certificates`, `internal/firewall`, app domains | [networking.md](docs/design/networking.md) — where an app answers, TLS, and the host's ufw |
| `internal/metrics`, `internal/machine` | [monitoring.md](docs/design/monitoring.md) — what is sampled, and what 100% means |
| `internal/credential`, `internal/extregistry` | [credentials.md](docs/design/credentials.md) — one secret, named by everything that needs it |
| `internal/user`, `internal/setup` | [authentication.md](docs/design/authentication.md) — API keys, sessions, the setup token |
| `install.sh`, `internal/settings` | [installing.md](docs/design/installing.md) — the front door, the data directory, the instance's own settings |
| `internal/release`, `internal/update` | [releasing.md](docs/design/releasing.md) — tags, notes, and replacing the daemon that is running |
| `internal/platform/bootstrap` | [infrastructure.md](docs/design/infrastructure.md) — the config hash, and why state must be a bind mount |
| Adding or moving a route | [the-api-document.md](docs/design/the-api-document.md) — `Handle` vs `HandleInternal` |

## Layout

The code is organized by domain, not by technical layer. A module owns
everything about one concept — its entity, its persistence, its use
cases, and every surface it is reached through.

```
internal/
  user/         identities, the API keys they authenticate with, and the
                one authorization question on the instance
  project/      projects and the environments inside them
  app/          apps, deployments, and the deploy orchestrator
  datastore/    the databases the instance runs, and which apps are
                wired to which
  objectstore/  the buckets this instance can reach — the MinIO it runs
                and the S3 endpoints it holds keys for — and the files
                in them
  metrics/      what every container is using, sampled on a timer — the
                series apps, databases and managed stores are charted
                from
  machine/      what the box under all of them is doing: its CPU, its
                memory, the disk everything is kept on, and the bytes
                over its own interfaces
  node/         the machines this instance is made of — the control
                plane and the workers that dial it, and what each of
                them serves
  mesh/         the private network those machines share, and the
                firewall rules that let them reach each other
  worker/       the daemon running as somebody else's machine: the loop
                that calls home and does what it is told. It runs no
                proxy — every name arrives at the control plane
  registry/     who may docker push/pull, and the push webhook
  credential/   the secrets this instance holds — one secret, stored
                once, named by everything that needs it
  extregistry/  which registry Cubeship does not run, and which
                credential logs in to it
  dns/          which provider writes this instance's records, and the
                credential it writes them with

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cubeshipd/cubeship](https://github.com/cubeshipd/cubeship) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-12 -->
