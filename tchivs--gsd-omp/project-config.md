---
trigger: always_on
description: `gsd-omp` is an independently maintained CommonJS host plugin that adapts `@opengsd/gsd-core` to Oh My Pi (OMP). It negotiates GSD Embeddable Orchestration System (EoS) protocol 1, projects GSD agents and skills into an OMP runtime profile, and exposes GSD commands, `gsd_invoke`, status surfaces, and native task/session integrations.
---

# Repository Guidelines

## Project Overview

`gsd-omp` is an independently maintained CommonJS host plugin that adapts `@opengsd/gsd-core` to Oh My Pi (OMP). It negotiates GSD Embeddable Orchestration System (EoS) protocol 1, projects GSD agents and skills into an OMP runtime profile, and exposes GSD commands, `gsd_invoke`, status surfaces, and native task/session integrations.

This is a pure Node.js package: there is no transpilation or bundling step. The `npm pack` tarball is the distribution artifact. The package requires Node.js `>=24.0.0` and GSD Core `>=1.11.0` (dependency range `^1.11.0`).

## Architecture & Data Flow

1. **Install and ownership** — `bin/gsd-omp.cjs` resolves the OMP runtime root (`--root` > `PI_CODING_AGENT_DIR` > `~/.omp/agent`), initializes EoS, checks the GSD Core version, builds projected artifacts, and performs a staged install. `.gsd-omp-manifest.json` records SHA-256 ownership so modified or unmanaged files are protected.
2. **EoS contract** — `src/eos.cjs` loads GSD Core's Host-Integration SDK, negotiates protocol `1` with the `programmatic-cli` profile, and caches the imperative OMP adapter, passive model adapter, host hook bus, and filesystem state adapter.
3. **Projection** — `src/projection.cjs` reads GSD Core `agents/`, `skills/`, and `commands/gsd/`. It rewrites runtime paths and `gsd:<name>` references, injects OMP tool/orchestration guidance, and emits artifacts under the selected runtime root's `agents/` and `skills/` directories. Core files remain the source of truth.
4. **OMP extension load** — The generated `extensions/gsd-omp.ts` wrapper loads `src/extension.cjs` and passes the runtime root. The extension factory registers the GSD command surface, `gsd_invoke`, completions, resource discovery, status/widget/overlay integrations, and lifecycle handlers through capability-checked OMP APIs.
5. **Command execution** — `/gsd ...` and `gsd_invoke` resolve the bundled GSD Core `gsd-tools.cjs`, run it with the project as `cwd`, `GSD_RUNTIME=omp`, and the effective `GSD_AGENTS_DIR`, then return bounded output as OMP messages. Hook subprocesses are bounded and fail open unless a hook explicitly blocks a call.
6. **Native runtime state** — OMP owns session lifecycle, model selection, approvals, native tasks, jobs, isolation, compaction, retries, aborts, and session navigation. GSD Core owns its CLI semantics and `.planning/` artifacts. `gsd-omp` translates between those boundaries and feeds project state, native task state, context signals, and Goal Mode state into `/gsd-status`, the widget, footer, and overlay.
7. **Goal Mode** — `goal_updated` is optional. The extension registers it inside a guarded block, validates and caches event state, and can recover the latest goal from the OMP session journal. It does not call private Goal APIs or mutate OMP's GoalRuntime. An active OMP goal holds a pending GSD continuation and instructs the user to pause/drop the goal before `/gsd-next`; OMP 17 remains functional without the event.
8. **Localization** — Shell CLI/EoS messages use `src/locale.cjs` and POSIX environment precedence. In-session extension messages use the project's `response_language`, intentionally keeping shell locale and project session language separate.

## Key Directories

- `bin/` — executable installer/updater and manifest ownership logic.
- `src/` — EoS adapter, OMP extension, artifact projection, localization, and graphify worker.
- `src/locales/` — English and Simplified Chinese message dictionaries.
- `scripts/` — release metadata synchronization and packed OMP host smoke testing.
- `test/` — Node built-in unit and contract tests with temporary-directory fixtures.
- `docs/` — architecture, configuration, development, testing, and getting-started guides.
- `.github/workflows/` — Node unit CI, OMP 17/latest host smoke, upstream drift reporting, and automated release.

## Development Commands

Install dependencies for an editable checkout, or reproduce CI's lockfile install:

```bash
npm install
npm ci
```

Run the package checks:

```bash
npm run lint                         # node --check for all shipped .cjs modules
npm test                             # node --test across test/
node --test test/extension.test.cjs  # focused suite while iterating
```

`prepack` runs lint and tests. To inspect the actual distribution artifact without lifecycle scripts:

```bash
package_tarball="$(npm pack --silent --ignore-scripts)"
npm install --global "./${package_tarball}"
```

Exercise the installer without touching the normal OMP profile:

```bash
root="$(mktemp -d)"
PI_CODING_AGENT_DIR="$root" node bin/gsd-omp.cjs install --json
PI_CODING_AGENT_DIR="$root" node bin/gsd-omp.cjs doctor --json
PI_CODING_AGENT_DIR="$root" node bin/gsd-omp.cjs descriptor --json
PI_CODING_AGENT_DIR="$root" node bin/gsd-omp.cjs uninstall --json
rm -rf "$root"
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tchivs/gsd-omp](https://github.com/tchivs/gsd-omp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
