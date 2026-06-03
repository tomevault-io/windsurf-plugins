---
trigger: always_on
description: <img align="right" width="180" height="auto"  src="./.github/docs/docker.svg" alt="Docker in the Half-Life Colours">
---

# Agents

<img align="right" width="180" height="auto"  src="./.github/docs/docker.svg" alt="Docker in the Half-Life Colours">

> [!NOTE]
> This document is intended for AI agents and tools such as GitHub Copilot. If you're a human, check out the [Getting Started guide](README.md) or the [Contributing guide](CONTRIBUTING.md) instead.

## Repository Overview 📖

This repository provides a Dockerized solution for running the **Half-Life Dedicated Server (HLDS)**, supporting all classic GoldSrc games and mods. The project uses Docker to simplify server setup, with support for custom configurations, plugins, and mods. Pre-built images are published to Docker Hub and GitHub Container Registry via GitHub Actions CI/CD pipelines.

## Technologies 🔧

- **Docker** — Containerizes the HLDS server. The [`Dockerfile`](container/Dockerfile) lives in `container/` and supports build arguments (`GAME`, `FLAG`, `VERSION`, `IMAGE`).
- **Docker Compose** — Two compose files: [`docker-compose.yml`](docker-compose.yml) (root, for end-users pulling pre-built images) and [`container/docker-compose.yml`](container/docker-compose.yml) (for building custom images locally).
- **GitHub Actions** — CI/CD workflows in `.github/workflows/` for validation, beta publishing, production publishing, sponsor management, and PR labeling.
- **Shell Scripting** — [`container/entrypoint.sh`](container/entrypoint.sh) handles runtime initialization (mod syncing, config syncing, server startup).
- **SteamCMD** — Downloads HLDS game files during the Docker build via the [`container/hlds.txt`](container/hlds.txt) script.

## Project Structure 📂

```
├── AGENTS.md                        # This file
├── ARCHITECTURE.md                  # Architecture documentation with diagrams
├── .github/
│   ├── workflows/
│   │   ├── validate.yml             # CI: builds and validates all 12 game variants
│   │   ├── publish.yml              # CD: version bump → build → test → push to registries → GitHub release
│   │   ├── beta.yml                 # CD: builds and pushes beta-tagged images
│   │   ├── sponsors.yml             # Updates README with GitHub Sponsors
│   │   └── label.yml                # Auto-labels PRs via conventional commits
│   ├── ISSUE_TEMPLATE/              # Bug report form and config
│   ├── PULL_REQUEST_TEMPLATE.md
│   ├── CODEOWNERS                   # @JamesIves owns all files
│   ├── dependabot.yml               # Weekly updates for Actions and Docker
│   ├── release.yml                  # Changelog categories for releases
│   └── FUNDING.yml                  # GitHub Sponsors
├── container/                       # Docker build context
│   ├── Dockerfile                   # Ubuntu base, SteamCMD, HLDS
│   ├── entrypoint.sh                # Runtime: validates args, syncs mods/config, starts hlds_run
│   ├── hlds.txt                     # SteamCMD install script (app 90, runs 3x for reliability)
│   ├── docker-compose.yml           # For building custom images locally
│   ├── config/                      # Default configs baked into the image
│   │   ├── server.cfg               # Default hostname and contact
│   │   ├── autoexec.cfg             # Executes default.cfg
│   │   ├── default.cfg              # Empty placeholder for user customization
│   │   └── motd.txt                 # HTML message of the day
│   └── mods/                        # Empty by default; mods baked into custom builds go here
├── config/                          # User-provided configs (volume-mounted at runtime, gitignored)
├── mods/                            # User-provided mods (volume-mounted at runtime, gitignored)
├── docker-compose.yml               # End-user compose file pulling pre-built images
├── docs/
│   └── index.html                   # Retro-styled web UI for generating Docker commands
├── README.md
├── CONTRIBUTING.md
├── SECURITY.md
├── CODE_OF_CONDUCT.md
└── LICENSE                          # MIT
```

## Supported Games 🎮

All images use SteamCMD app ID `90` with a `mod` config to select the game variant:

| Game Identifier | Game Name                      | Legacy Available |
| --------------- | ------------------------------ | ---------------- |
| `valve`         | Half-Life Deathmatch           | Yes              |
| `cstrike`       | Counter-Strike                 | Yes              |
| `czero`         | Counter-Strike: Condition Zero | Yes              |
| `dmc`           | Deathmatch Classic             | No               |
| `gearbox`       | Half-Life: Opposing Force      | No               |
| `ricochet`      | Ricochet                       | No               |
| `dod`           | Day of Defeat                  | No               |
| `tfc`           | Team Fortress Classic          | Yes              |

Legacy variants use the `-beta steam_legacy` flag to install the pre-25th Anniversary Edition of the game.

## Key Build Arguments 🏗️

| Argument  | Purpose                                                         | Default   |
| --------- | --------------------------------------------------------------- | --------- |
| `GAME`    | GoldSrc game/mod identifier passed to SteamCMD                  | `valve`   |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JamesIves/hlds-docker](https://github.com/JamesIves/hlds-docker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
