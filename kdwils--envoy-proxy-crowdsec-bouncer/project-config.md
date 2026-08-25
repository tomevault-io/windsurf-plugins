---
trigger: always_on
description: Guidance for agents working in this repository. Module: `github.com/kdwils/envoy-proxy-bouncer` (Go 1.26).
---

# AGENTS.md

Guidance for agents working in this repository. Module: `github.com/kdwils/envoy-proxy-bouncer` (Go 1.26).

## Project Overview

Envoy ext_authz service that bounces malicious traffic using CrowdSec: streamed IP decisions (ban/captcha), AppSec WAF inspection, and CAPTCHA challenges (reCAPTCHA v2, Cloudflare Turnstile).

### Package Layout

- `bouncer/` — `Bouncer` type with the core `Check` hot path: `ParseCheckRequest`, `ExtractRealIP`, decision-cache / CAPTCHA / WAF orchestration, and the `WAF`, `DecisionCache`, `CaptchaService` interfaces.
- `bouncer/components/` — concrete components: decision cache, AppSec WAF client, CAPTCHA service with reCAPTCHA and Turnstile providers, HTTP client interface.
- `server/` — `ServeDual` runs three listeners from config: gRPC ext_authz (`envoy.service.auth.v3.Authorization/Check`, reflection + health service registered) on `:8080`, HTTP CAPTCHA verify endpoints with per-IP rate limiting on `:8081`, and Prometheus `/metrics` on `:9090`. Service interfaces (`Bouncer`, `TemplateStore`, `Notifier`) live in `server/services.go`.
- `pkg/cache/` — generic in-memory cache with TTL cleanup used by the decision cache and CAPTCHA sessions.
- `pkg/crowdsec/` — CrowdSec LAPI client (TLS, cert auth) and metrics reporting service.
- `config/` — config structs and validation only. **Defaults are not here** (see Config wiring).
- `template/` — HTML template store (`RenderDenied`, `RenderCaptcha`) for ban/captcha pages.
- `webhook/`, `recorder/`, `logger/` — webhook notifier, Prometheus recorder, `slog` helpers.
- `cmd/` — cobra CLI (`serve`, `bounce`, `version`).
- `version/` — `Version` is `go:embed`ded from `version/version.txt`; bump that file to change the version.
- `tests/functional/` — testcontainers-based functional tests behind the `functional` build tag.

## Commands

```bash
go build -o envoy-proxy-bouncer .                          # Build the binary
go test -race ./...                                        # Unit tests (skips functional-tagged code)
go test -tags functional -timeout 30m ./tests/functional   # Functional tests (needs Docker; 30m timeout matters — the default 10m kills the image matrix)
go test -bench=. -benchmem -run=^$ ./...                   # Benchmarks
go generate ./...                                          # Regenerate mocks — requires mockgen on PATH (go install go.uber.org/mock/mockgen@latest)
go vet ./... && gofmt -l .                                 # Lint + format check
```

Single package / single test: `go test -race ./bouncer -run TestBouncer_Check/specific_case`.

## Config wiring

- Defaults and viper wiring live in `config/config.go` — `config.GetViper(cfgFile)` returns a ready-to-use `*viper.Viper` with the env prefix (`ENVOY_BOUNCER_`), key replacer (`.`→`_`), AutomaticEnv, optional config file, and all defaults. When adding a config key, add the struct field in `config/` **and** the default in `config/defaults.go`, or the zero value silently applies.
- Env vars: prefix `ENVOY_BOUNCER_` with `.`→`_` mapping (e.g. `ENVOY_BOUNCER_BOUNCER_APIKEY`... verify exact key form against the replacer in `config/config.go`); `--config` flag accepts a yaml/json file.
- User-facing docs for config/captcha/webhooks/templates live in `docs/` — update them when changing behavior.

## Mocks

Mocks are generated with mockgen into `<pkg>/mocks/` via `//go:generate` directives next to each interface (`bouncer`, `bouncer/components`, `server`, `webhook`, `pkg/crowdsec`). After changing an interface, run `go generate ./...` and commit the regenerated mocks.

## Release / CI facts

- Push to `main` publishes a `ghcr.io/.../envoy-proxy-bouncer:sha-<sha>` image. Tagging `v*.*.*` re-tags that image, publishes the Helm chart, and runs goreleaser (skipped for `-rc` tags).
- Functional test workflow runs only on non-main pushes that touch Go files.
- Helm chart in `charts/envoy-proxy-bouncer`: `values.schema.json` (helm-values-schema-json, driven by `# @schema` comments in `values.yaml`) and chart `README.md` (helm-docs) are generated — regenerate both when editing `values.yaml`. Charts publish only from merged PRs whose branch starts with `charts/`.

## Code Style

- No comments unless asked.
- Never `else` — use early returns, or set a default value and override it conditionally.

## Testing Rules

- Use `t.Context()` for test contexts — never `context.Background()`. It is canceled when the test finishes, so it cannot leak goroutines past the test.

### Test structure

- **Tests go in the file they test** — `recaptcha.go` is tested by `recaptcha_test.go`, never by a shared/merged test file covering multiple sources.
- **Never write mocks in test files** — mocks are generated from interfaces (`go generate ./...`, go.uber.org/mock). Every mock used in a test comes from the generated packages (`bouncer/mocks`, `bouncer/components/mocks`, etc.); never hand-roll a mock, fake, stub, or `gomock.Matcher`. Capture and assert captured request arguments inline with `gomock.Any()` + `.Do(...)`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kdwils/envoy-proxy-crowdsec-bouncer](https://github.com/kdwils/envoy-proxy-crowdsec-bouncer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
