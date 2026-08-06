---
trigger: always_on
description: Contributor and AI-agent guide for the **solace-prometheus-exporter** codebase. It captures the conventions,
---

# AGENTS.md

Contributor and AI-agent guide for the **solace-prometheus-exporter** codebase. It captures the conventions,
structure and invariants a change should respect. Read it alongside [`CONTRIBUTING.MD`](CONTRIBUTING.MD) and
[`docs/CONFIG.md`](docs/CONFIG.md).

## 1. Technology Stack

* **Language:** Go (see `go` directive in [`go.mod`](go.mod); module name `solace_exporter`).
* **Metrics:** `github.com/prometheus/client_golang` (the exporter implements `prometheus.Collector`).
* **CLI / logging:** `github.com/alecthomas/kingpin/v2` for flags; `github.com/prometheus/common/promslog` +
  `log/slog` for structured logging; `github.com/prometheus/common/version` for build info.
* **Config:** `gopkg.in/ini.v1` for the INI file; `os.Getenv` for environment overrides.
* **Auth / TLS:** `golang.org/x/oauth2/clientcredentials` for broker OAuth;
  `software.sslmate.com/src/go-pkcs12` for PKCS#12 keystores; standard `crypto/tls`.
* **Concurrency:** `golang.org/x/sync/semaphore` bounds concurrent SEMP connections.
* **SEMP transport:** standard `net/http`, `encoding/xml` (SEMP v1), `encoding/json` (SEMP v2).
* **Build & CI:** [`Makefile`](Makefile), multi-stage [`Dockerfile`](Dockerfile) producing a `scratch` image,
  GitHub Actions running `golangci-lint`, `go mod tidy` checks and `go test`.

## 2. Project Structure

```
cmd/solace-prometheus-exporter/   Entry point: flag parsing, HTTP handler wiring, /solace param parsing
internal/exporter/                Config, the Exporter collector, per-request config, HTTP client, auth, TLS, async prefetch
  config.struct.go                Config struct + ParseConfig (INI + env), endpoint alias parsing
  exporter.struct.go              Exporter type; wires a *semp.Semp per request
  exporter.collect.go             CollectPrometheusMetric: the scrape-target dispatch switch + Collect()
  exporter.describe.go            Describe() over the metric registry
  dataSource.struct.go            DataSource{Name, VpnFilter, ItemFilter, MetricFilter}
  auth.go / http.go / tlsServer.go  OAuth/basic visitor, HTTP client, HTTPS listener
  asyncFetcher.go                 Prefetch collector used when prefetchInterval > 0
internal/semp/                    SEMP access layer
  getXxxSemp1.go                  One file per SEMP v1 target (POST XML to /SEMP)
  getQueueStatsSemp2.go           SEMP v2 target (GET JSON from /SEMP/v2/monitor/...)
  metricDesc.go                   MetricDesc registry: label sets + all metric Descriptions
  semp.desc.struct.go             Desc, Descriptions, V2Result, field-selection helpers
  prometheusMetric.struct.go      PrometheusMetric wrapper (dedupe key, cardinality checks)
  http.go / helper.go / types/    HTTP verbs, filter escaping, shared XML types
internal/web/                     Index page handler/template + exporter Basic Auth wrapper
configs/                          Sample INI config (also baked into the Docker image)
docs/CONFIG.md                    Full settings + scrape-target reference
examples/                         Grafana dashboards and an NGINX reverse-proxy Helm chart
test/                             Captured SEMP fixtures (test/data) and an OAuth e2e stack (test/oauth)
```

## 3. Coding Standards & Conventions

* **Formatting/linting:** code must be `gofmt`-clean and pass `golangci-lint run` (`make lint`). Keep imports
  grouped (stdlib, then third-party) and use tabs for indentation.
* **Logging:** use the injected `*slog.Logger` with key/value pairs (`logger.Error("msg", "err", err, "broker",
  uri)`), not `fmt`/`log`. Do not `panic` in a scrape path (see §7).
* **Errors:** wrap with `%w` (`fmt.Errorf("...: %w", err)`); return early. Config parsing returns
  `(..., *Config, error)` and fails fast on invalid/incomplete input.
* **Metrics namespace:** every metric name is prefixed with `solace_` — this is applied once in
  `NewSemDesc` (do not repeat the prefix in call sites).
* **Value types:** pick `prometheus.CounterValue` for monotonic counters and `prometheus.GaugeValue` for gauges
  when emitting via `semp.NewMetric`.
* **No magic values:** durations, page sizes and channel capacities are named constants (`longQuery`,
  `capMetricChan`, `metricCacheChunkSize`, ...).
* **Security exceptions:** the SEMP HTTP client intentionally uses `InsecureSkipVerify` (broker certs are often
  self-signed); it is annotated `//nolint:gosec`. Do not remove the annotation without changing the behaviour.

## 4. Scrape Handlers & SEMP Access

* **Handler wiring (`main.go`):** `/metrics` serves process metrics; `/solace` parses `m.<Target>` GET params into
  `[]exporter.DataSource`; each `[endpoint.<alias>]` config section becomes its own handler. Every handler is
  wrapped with `web.WrapWithAuth` (exporter Basic Auth). When `prefetchInterval > 0`, alias handlers are served by
  an `AsyncFetcher` from cache instead of scraping inline.
* **Parameter grammar:** a `/solace` value is `vpnFilter|itemFilter[|metricFilter,...]`; fewer than two
  `|`-separated parts is skipped/rejected with a log. The metric filter (third part) is SEMP v2 only.
* **Dispatch (`exporter.collect.go`):** `CollectPrometheusMetric` switches on `DataSource.Name`. Each target has a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [solacecommunity/solace-prometheus-exporter](https://github.com/solacecommunity/solace-prometheus-exporter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
