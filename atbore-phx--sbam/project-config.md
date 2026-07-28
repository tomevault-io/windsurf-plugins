---
trigger: always_on
description: This project is a **Go (Golang) application** that intelligently manages Fronius Gen24+ solar battery charging using weather forecasts.
---

# Claude Code Instructions – sbam (Smart Battery Advanced Manager)

## Project Overview

This project is a **Go (Golang) application** that intelligently manages Fronius Gen24+ solar battery charging using weather forecasts.

The system:

- Retrieves daily solar production forecasts from the Solcast API
- Monitors battery state of charge via the Fronius local Solar API (REST)
- Controls battery charging via Fronius Modbus TCP protocol (register writes)
- Dynamically decides when and how much to charge the battery from the grid based on:
  - Weather forecasts (solar production estimates)
  - Daily household consumption
  - Current battery charge level
  - Configurable battery reserve thresholds
  - Time windows for grid charging
  - Net power calculations (solar production − consumption)

Deployment targets:

- Standalone CLI application
- Home Assistant add-on
- Docker container

The application must be:

- Modular
- Testable
- Secure
- Idiomatic Go
- Deployable as a Home Assistant add-on and Docker container

---

## Tech Stack

- Go 1.26+
- net/http (default HTTP client for Solcast and Fronius Solar API)
- github.com/simonvetter/modbus (Modbus TCP client for Fronius inverter control)
- github.com/spf13/cobra (CLI framework)
- github.com/spf13/viper (configuration: YAML, env vars, CLI flags)
- go.uber.org/zap (structured logging)
- github.com/robfig/cron/v3 (cron scheduling for recurring charge cycles)
- github.com/stretchr/testify (testing assertions)
- github.com/tbrandon/mbserver (mock Modbus server for testing)
- Docker support (standalone + Home Assistant add-on)

---

## Project Structure

```
.claude/
.github/
  ISSUE_TEMPLATE/
    bug_report.yml            - GitHub issue form for bug reports
    feature_request.yml       - GitHub issue form for feature requests
    config.yml                - Issue template chooser policy (blank issues disabled)
  dependabot.yml              - Dependabot updates for Go modules, GitHub Actions, and Dockerfiles
  workflows/                  - CI/CD workflow definitions
    test.yml                  # Go tests on every push
    release.yml               # Tag-based release (HA add-on + GoReleaser binaries)
    dev-image.yml             # Dev Docker image on release branches
    docs-deploy.yml           # MkDocs build + GitHub Pages deploy (main) / preview artifact (branches)
  actions/
    build-ha-addon/           - Composite action: build Go binary + HA add-on Docker image

mkdocs.yml                    # MkDocs configuration (Material theme, GitHub Pages deploy)
docs/
  site/                         # MkDocs source — user-facing documentation
    index.md                    # Landing page
    prerequisites.md            # Inverter setup and Solcast prerequisites
    installation.md             # HA add-on, Docker, and standalone installation
    configuration.md            # Single config reference with deployment annotations
    mqtt.md                     # MQTT guide (topic map, payloads, commands, discovery)
    cli.md                      # Full CLI command and flag reference
    changelog.md                # Release history
  vibe/                       - Contributor workflow docs for Compound Engineering skills
  brainstorms/                # CE brainstorm artifacts
  plans/                      # CE implementation plans
  implementations/            # Archived pre-CE implementation plans

main.go                      # Entry point, version vars, delegates to pkg/cmd
main_test.go                 # CLI-level tests

pkg/
  cmd/
    root.go                   - Cobra root command, Viper config loading
    root_test.go              - Unit tests for the root command
    configure.go              - `configure` command: battery defaults & force charge
    configure_test.go         - Unit tests for the configure command
    estimate.go               - `estimate` command: display forecast & battery state
    estimate_test.go          - Unit tests for the estimate command
    schedule.go               - `schedule` command: main intelligent charging workflow
    schedule_runner.go        - Single-goroutine runner that serializes schedule ticks and command intents
    schedule_runner_test.go   - Unit tests for runner command handling and queue behavior
    schedule_test.go           - Unit tests for the schedule command
    schedule_validation_test.go - Unit tests for schedule parameter validation
    schedule_mqtt_wiring_test.go - Unit tests for MQTT command subscription wiring and latest-state re-publish behavior
    schedule_lifecycle_test.go - Unit tests for runner lifecycle behavior in no-cron MQTT/no-MQTT modes
    schedule_cron_test.go     - Unit tests for cron-based scheduling
    precedence_test.go        - Unit tests for flag > env > yaml viper precedence
    config_schema_test.go     - Unit tests for HA add-on config.json schema regex validation
  fronius/
    types.go                  - Fronius struct definitions
    handler.go                - Main battery control dispatcher
    modbus.go                 - Modbus TCP client (open, read, write, close)
    classify.go               - Battery charge classification logic

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [atbore-phx/sbam](https://github.com/atbore-phx/sbam) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
