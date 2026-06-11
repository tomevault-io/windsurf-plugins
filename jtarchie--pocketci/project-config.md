---
trigger: always_on
description: Trust these instructions. Only fall back to repo-wide search if something here
---

# PocketCI — Agent Onboarding Instructions

Trust these instructions. Only fall back to repo-wide search if something here
is demonstrably wrong or missing.

## What this repo is

PocketCI is a **local-first CI/CD runtime**. Pipelines are written in
JavaScript/TypeScript (and YAML, for Concourse-CI backwards compatibility) and
executed inside containers/VMs by one of several orchestration **drivers**
(Docker, Kubernetes, Fly.io, DigitalOcean, Hetzner, QEMU, macOS VZ, native).
State is persisted to a single SQLite database. There is also a web server with
HTMX-driven UI, REST API, MCP server, and webhook receivers.

- Primary language: **Go 1.26** (`go.mod` →
  `module github.com/jtarchie/pocketci`). `main.go` wires the CLI via
  [alecthomas/kong](https://github.com/alecthomas/kong).
- Pipeline scripting language: **TypeScript/JavaScript**, executed in-process by
  [dop251/goja](https://github.com/dop251/goja) (a pure-Go ES5+ runtime).
  `runtime/jsapi/` exposes Go ↔ JS bindings.
- Front end: HTMX + Tailwind v4 + esbuild bundle in `server/static/`.
- Docs: VitePress in `docs/` (also embedded into the server at build time).
- E2E: Playwright in `e2e/`.
- Repo size: ~50k+ LoC of Go across ~30 top-level packages, plus a TS/Node front
  end and docs site. `go.sum` is large (~66 KB) — many third-party deps
  including Docker, AWS SDK v2, k8s client-go, fly.io machines, goja, esbuild,
  VZ, qemu.

The **CLAUDE.md** at the repo root is a symlink to **this file** — anything you
add here is what both Copilot and Claude Code agents will see.

## Toolchain (always required)

Install via Homebrew on macOS (`brew bundle` in repo root reads `Brewfile`) or
the equivalents on Linux. CI installs them via `setup-*` actions.

| Tool                                   | Where it's used                                                                                                                                                                                                   | Notes                                                                                                                    |
| -------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| `go` (1.26+, `stable`)                 | All Go code                                                                                                                                                                                                       | `setup-go@v6 with go-version: stable, check-latest: true` in CI                                                          |
| `task` (go-task)                       | The single entry point for all build/test/lint                                                                                                                                                                    | `Taskfile.yml`. Always invoke commands through `task <name>`, not bare `go build`/`npm run` — tasks chain prerequisites. |
| `deno` (v2.x)                          | TS formatting, linting, type checking of `examples/`. Also bundles `backwards/` (Concourse YAML compat).                                                                                                          |                                                                                                                          |
| `node` (LTS) + `npm`                   | `docs/`, `e2e/`, `server/static/`, `examples/`, `packages/pocketci/` each have their own `package.json` and `package-lock.json`. Always run `npm install` from inside the relevant subdir before its npm scripts. |                                                                                                                          |
| `shellcheck`, `shfmt`                  | Lint/format `bin/*.sh`                                                                                                                                                                                            |                                                                                                                          |
| `golangci-lint` (v2 config)            | `.golangci.yml`. Skipped locally when `CI=true` to avoid double work — CI runs it as its own job.                                                                                                                 |                                                                                                                          |
| `playwright` (`@playwright/test` 1.59) | E2E. Browsers cached in CI under `~/.cache/ms-playwright`. Install with `cd e2e && npx playwright install --with-deps` once.                                                                                      |                                                                                                                          |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jtarchie/pocketci](https://github.com/jtarchie/pocketci) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
