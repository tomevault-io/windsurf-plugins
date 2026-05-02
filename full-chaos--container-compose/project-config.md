---
trigger: always_on
description: Guidance for autonomous coding agents working in this repository.
---

# AGENTS.md — Container-Compose

Guidance for autonomous coding agents working in this repository.

This file is the canonical orientation for agents (and humans) joining the project.
Read it before exploring. It mirrors the structure of `CLAUDE.md` / `AGENTS.md`
conventions used in agent-driven workflows.

---

## 1. Project Summary

**Container-Compose** is a Swift 6.1 CLI that brings *limited* Docker Compose
support to [Apple Container](https://github.com/apple/container). It parses
`docker-compose.yml` and orchestrates services via Apple's `container` runtime
on macOS.

- **Language / toolchain:** Swift 6.1, SwiftPM, macOS 15+ (best on macOS 26 Tahoe).
- **CLI entry:** `container-compose <subcommand>` (driven by `swift-argument-parser`).
- **Distribution:** Homebrew (`brew install container-compose`) or `make build && make install`.
- **License:** MIT.

The project is **not** a Docker / Docker Compose wrapper. It directly
interprets the Compose schema and translates a (large) subset to
`container run` / `container build` / `container network create` invocations.

---

## 2. Repository Map

```
Sources/
  ContainerComposeApp/        ← thin executable target (just calls Application)
    application.swift
  Container-Compose/          ← library target `ContainerComposeCore`
    Application.swift         ← root AsyncParsableCommand wiring subcommands
    Errors.swift              ← YamlError, ComposeError enums
    Helper Functions.swift    ← env loading, var substitution, port parsing, paths
    PreSubcommandFlagPromotion.swift  ← global flag normalization
    ProjectFlags.swift        ← shared @OptionGroup for project flags
    Codable Structs/          ← Compose schema → Swift model layer (36 files)
    Commands/                 ← AsyncParsableCommand + per-concern argv builders (31 files)
    Runtime/                  ← ContainerClientProvider + RunCommandRunner seams

Tests/
  Container-Compose-StaticTests/   ← parsing + argv-shape tests (53 files, 724 @Test cases)
  Container-Compose-DynamicTests/  ← integration tests against real `container` runtime (11 @Test cases)
  TestHelpers/                     ← shared fixtures, RecordingRunner, RecordingContainerClientProvider

Sample Compose Files/         ← runnable example compose files
scripts/regen-coverage.sh     ← extracts coverage.json from coverage.html
.github/workflows/            ← CI workflows (Tests, gh-pages, CodeQL via default-setup)
coverage.html                 ← canonical compose-spec coverage matrix (source of truth)
coverage.json                 ← derived from coverage.html, gitignored
Package.swift                 ← SwiftPM manifest
Package.resolved              ← pinned deps
Makefile                      ← build, install, clean targets
```

### Dependencies (`Package.swift`)

| Package                | Source                                        | Purpose                          |
| ---------------------- | --------------------------------------------- | -------------------------------- |
| `swift-argument-parser`| github.com/apple/swift-argument-parser ≥1.5.1 | CLI parsing                      |
| `container`            | github.com/mcrich23/container (custom branch) | Apple Container client APIs      |
| `Yams`                 | github.com/jpsim/Yams ≥5.0.6                  | YAML decoder                     |
| `Rainbow`              | github.com/onevcat/Rainbow ≥4.0.0             | ANSI-colored per-service output  |

Note the `container` dependency points at `mcrich23/container` on branch
`add-command-option-group-function-macro` — this is an upstream-fork that
exposes the macro `Container-Compose` needs to compose subcommands. Keep this
in mind when bumping versions.

---

## 3. How the Code Is Organized

### 3.1 Codable Structs (Schema → Swift)

Every top-level Compose entity has a dedicated `Codable` struct. The decoder
goes through `Yams.YAMLDecoder().decode(DockerCompose.self, …)`.

| File                       | Type                  | Compose entity       |
| -------------------------- | --------------------- | -------------------- |
| `DockerCompose.swift`      | `DockerCompose`       | root document        |
| `Service.swift`            | `Service`             | `services.<name>`    |
| `Build.swift`              | `Build`               | `service.build`      |
| `Healthcheck.swift`        | `Healthcheck`         | `service.healthcheck`|
| `Deploy.swift`             | `Deploy`              | `service.deploy`     |
| `DeployRestartPolicy.swift`| `DeployRestartPolicy` | `deploy.restart_policy` |
| `DeployResources.swift`    | `DeployResources`     | `deploy.resources`   |
| `ResourceLimits.swift`     | `ResourceLimits`      | `…resources.limits`  |
| `ResourceReservations.swift`| `ResourceReservations`| `…resources.reservations` |
| `DeviceReservation.swift`  | `DeviceReservation`   | `…reservations.devices[]` |
| `Network.swift` / `ExternalNetwork.swift` | `Network`, `ExternalNetwork` | `networks.<name>` |
| `Volume.swift`  / `ExternalVolume.swift`  | `Volume`,  `ExternalVolume`  | `volumes.<name>`  |
| `Secret.swift`  / `ExternalSecret.swift`  | `Secret`,  `ExternalSecret`  | `secrets.<name>`  |
| `Config.swift`  / `ExternalConfig.swift`  | `Config`,  `ExternalConfig`  | `configs.<name>`  |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [full-chaos/container-compose](https://github.com/full-chaos/container-compose) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
