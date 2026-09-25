---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Nightingale (n9e) is an open-source alerting/monitoring platform. Go module `github.com/ccfos/nightingale/v6`, Go 1.25, gin + gorm. The frontend lives in a separate repo (`n9e/fe`) and is embedded at build time as a statik bundle. A commercial fork (`flashcatcloud/n9e-plus`) imports this module and calls into `center.Initialize`-style entry points, so **exported function signatures that plus calls (e.g. `alert.Start`, `centerrt.New`, `BuildNotifyContext`) are effectively public API** — prefer adding fields/setters over changing parameter lists (see comments in `center/center.go` and `alert/alert.go`).

## Commands

```bash
# Build (front/statik/statik.go is committed, so plain go build works without fe.sh)
go build ./...                       # ~1 min cold; verifies everything incl. tests compile
make build                           # ./n9e  (cmd/center, embeds version via ldflags)
make build-edge | build-alert | build-pushgw | build-cli
sh fe.sh                             # download latest n9e/fe release into ./pub and regenerate front/statik (needs $GOPATH/bin/statik)

# Run locally
./n9e --configs etc.local            # etc.local* dirs are git-ignored per-developer configs; etc/ is the shipped default
./n9e-edge --configs etc/edge        # edge/alert processes need [CenterApi] configured

# Tests (unit tests use in-memory sqlite via github.com/glebarez/sqlite; no external services needed)
go test ./...
go test ./models/ -run TestAlertRuleVerify -count=1      # single test
go test ./center/router/ -run 'TestQueryBatchV2' -v

# CI gates for builtin integration templates (run these when touching integrations/**)
go run ./cmd/integrations-alert-action check     # every builtin alert rule must carry a non-empty `action` annotation
go run ./cmd/integrations-alert-action update    # tighten integrations/alert_action_baseline.json after fixing rules
go run ./cmd/integrations-i18n check -scope all  # en_US rendering of templates must contain no Chinese
```

There is no lint config in the repo; CI only runs the two integration gates above and goreleaser on tags.

## Process topology

Three deployable binaries share one codebase and one config schema (`conf/`):

| Binary | Entry | Role |
|---|---|---|
| `n9e` (center) | `cmd/center` → `center.Initialize` | Owns the DB. Serves UI + `/api/n9e`, `/v1/n9e` (service API for edge/alert), pushgw, alert engine, optional embedded TSDB, ibex task system. |
| `n9e-edge` | `cmd/edge/edge.go` | Remote-datacenter node: pushgw + alert engine, **no DB**. |
| `n9e-alert` | `cmd/alert` → `alert.Initialize` | Standalone alert engine, no DB. |

`pkg/ctx.Context` carries `DB` and `IsCenter`. Model functions branch on it: with `IsCenter` they hit gorm directly, otherwise they call the center over HTTP via `pkg/poster` (`GetByUrls`/`PostByUrls` against `CenterApi.Addrs`, endpoints under `/v1/n9e/...` in `center/router`). **Any new model read path that edge/alert needs must implement both branches**, and the center must expose a matching `/v1/n9e` route.

Alert engines register in `alerting_engine` table with `engine_cluster` (= `Alert.Heartbeat.EngineName`, default `default`); rules are sharded across engines of the same cluster by a consistent hash (`alert/naming`). A datasource's `cluster_name` selects which engine cluster evaluates it.

## Request/data flow

- **`memsto/`**: in-memory caches (`*CacheType`) of DB tables, each polling on its own goroutine and refreshing when `count/max(update_at)` changes. Alert evaluation, dispatch and routers read from these caches, never from the DB on the hot path. Adding a cached table means: model `Stat()` + `GetsAll()` (both ctx branches), a cache in `memsto`, and threading it through `alert.Start`.
- **Alert pipeline** (`alert/`): `eval` (scheduler per rule, queries via `prom`/`datasource`, produces `AlertCurEvent`) → `process` (mute, relabel, dedupe against `alert_cur_event`, `pushEventToQueue`) → `queue` → `dispatch` (subscribe, notify rules → channels/templates, event pipelines from `alert/pipeline`) → `sender` (per-channel senders, `sender/provider` for the configurable HTTP/script channel types). Event pipeline processors live in `alert/pipeline/processor/*`, implement `models.Processor` (or `BranchProcessor` for if/switch/foreach), self-register via `models.RegisterProcessor(typ, ...)` in their package `init`, and are executed by `alert/pipeline/engine` over a `WorkflowContext`; a new processor package must also be blank-imported in `alert/pipeline/pipeline.go` so its `init` runs.
- **Datasources**: `datasource/` = per-type query plugins used by the engine and `/ds-query`; `dskit/` = lower-level SQL/log clients; `dscache/` = the live client map synced from the `datasource` table; `prom/` = Prometheus-like clients. Datasource type ids/categories are the table in `datasource/datasource.go`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ccfos/nightingale](https://github.com/ccfos/nightingale) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
