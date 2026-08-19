---
trigger: always_on
description: Working notes for Peevee. Read this before changing anything; several decisions
---

# CLAUDE.md

Working notes for Peevee. Read this before changing anything; several decisions
here look arbitrary and are not.

## What this is

A read-only observability platform for PersistentVolumeClaim usage across many
Kubernetes clusters. It reads kubelet's `/stats/summary` through each cluster's
API server proxy, joins it to PVC inventory, serves a single-page UI, and remote
writes to Grafana Mimir.

Deep background: [docs/how-it-works.md](docs/how-it-works.md).

## Commands

```bash
make check          # vet + go test ./... + helm lint  (what CI runs)
make build          # ./peevee
make image          # container image
go test ./internal/store/... -run TestTotals -v
helm template peevee charts/peevee --set config.remoteWrite.enabled=true
```

No Go toolchain locally? Everything runs in a container:

```bash
docker run --rm -v "$PWD":/src -w /src golang:1.24-alpine go test ./...
```

## Layout

```
cmd/peevee/main.go        wiring, flags, graceful shutdown
internal/
  config/                 YAML config, defaults, validation, Duration type
  cluster/                kubeconfig discovery, client pool, hot reload, health
  collector/              per-cluster scrape; joins PVC inventory + kubelet stats
  store/                  in-memory snapshot, per-volume history, filter/aggregate
  metrics/                snapshot → Prometheus samples (text + remote write)
  remotewrite/            hand-rolled protobuf + snappy → Mimir
  api/                    REST, SSE, /metrics, embedded UI, config redaction
  app/                    the collection loop
  model/                  shared types
web/ui/                   hand-written HTML/CSS/JS, embedded — NOT build output
charts/peevee/            Helm chart
scripts/                  kubeconfig generation, install helper
docs/, examples/          reference material, alert rules, PromQL cookbook
```

## Invariants

Breaking any of these silently produces wrong answers, which is worse than
crashing. Each has a test.

**1. Never report "no data" as zero.**
A claim nobody mounts is not an empty claim. `unmounted`, `pending` and `block`
carry no usage numbers, publish no usage series, and are excluded from
aggregates. `store_test.go:TestUnmountedIsUnknownNotZero`,
`metrics_test.go:TestUnmountedPublishesNoUsageSeries`.

**2. Count each shared filesystem once in aggregates.**
`local-path` and friends report the whole host disk for every claim on it.
Summing per claim multiplies one disk by the number of claims. Dedupe key is
`(cluster, node, capacity)`. `store_test.go:TestTotalsCountSharedFilesystemOnce`.

**3. `peevee_pvc_info` carries only labels stable for a claim's lifetime.**
Anything changing — `status`, `phase` — mints a new series per transition, and
the stale one keeps resolving for the lookback window. Status is a state set
instead. `metrics_test.go:TestInfoCarriesOnlyStableLabels`.

**4. The state set emits every status, including the zeros.**
Emitting only the active one leaves the previous status stuck at 1.
`metrics_test.go:TestStatusIsAStateSetWithExactlyOneActive`.

**5. Redact before any config reaches a browser, and never mutate the original.**
The config struct's *yaml* tags include the remote write password that the json
tags drop, so the YAML view is the riskier path. `redact()` must copy — clearing
the password in place breaks remote write auth on the next push.
`api_test.go:TestRedactDoesNotMutateOriginal`.

**6. Remote write labels must be sorted by name.** Mimir rejects the request
otherwise. `proto_test.go`.

**7. Config is read-only in the UI.** Deliberate. A browser and `helm upgrade`
cannot both be the source of truth. Do not add write endpoints without
revisiting this with the user.

**8. Peevee never writes to an observed cluster.** Every verb is `get` or
`list`. Keep it that way; the RBAC in the chart and in
`scripts/create-remote-kubeconfig.sh` must stay in sync.

## Things that will bite you

- **Kubelet caches volume stats for 1 minute** (`volumeStatsAggPeriod`). Polling
  faster re-reads identical values and inflates the sparkline with duplicate
  points. One fresh sample per volume per minute is a hard ceiling.
- **`hostPath` PVs report nothing.** Kubelet has no metrics provider for that
  plugin. `local-path-provisioner` creates `local` volumes, which do report.
- **`[hidden]` in CSS.** Any rule setting `display` beats the browser's
  `display:none` for the attribute. There is a global
  `[hidden] { display: none !important; }` — do not remove it.
- **History is in memory.** Restart loses every sparkline. Mimir is the durable
  store.
- **Two logical clusters can point at one physical cluster** (the dev setup does
  this). The shared-filesystem dedupe is per cluster, so a shared node disk gets
  counted once per logical cluster.

## Known issues

**1. Misleading requested-percentage on shared filesystems.**
For a `shared` volume, `usedBytes` is the whole host disk, so `requestedPercent`
is arithmetic on unrelated numbers. The drawer currently says *"Against the
request it is at 1456%"*, which reads as if the claim blew past its quota. It
should say per-claim consumption is **unknowable** via `statfs`.
`web/ui/app.js` (drawer `sharedFilesystem` note), `model.Volume.RequestedPercent`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OdedNeuhaus/peevee](https://github.com/OdedNeuhaus/peevee) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
