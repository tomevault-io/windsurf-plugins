---
trigger: always_on
description: You are working on `dsh-plugins`, a repository for developing plugins for
---

# DSH Plugins Agent Guide

You are working on `dsh-plugins`, a repository for developing plugins for
DeepSeek Harness (DSH).

Your job is to develop, test, document, and maintain DSH plugins in this
repository. These plugins extend DeepSeek Harness through Cordis composition,
profile bundle patches, model-visible tools, runtime services, and optional
Web client modules.

This is plugin development, not DeepSeek Harness host development. The
DeepSeek Harness checkout at:

`/Users/yifanxu/Ephemeral-AI-Lab/deepseek-harness`

is read-only reference and runtime infrastructure. Do not modify it unless the
user explicitly authorizes a host-repository change.

## Repository boundary

All implementation changes belong under:

`/Users/yifanxu/Ephemeral-AI-Lab/dsh-plugins`

Do not edit, format, build, install dependencies, clean, reset, or commit in
`deepseek-harness` by default. Running its existing `pnpm dsh web` command to
launch the Web host is allowed. If host artifacts are missing or stale, stop
and ask before rebuilding the host.

Do not create a nested `deepseek-harness-docs` repository or directory. Keep
plugin documentation under this repository's `docs/` directory.

Before and after a task, inspect both working trees without changing them:

```sh
git -C /Users/yifanxu/Ephemeral-AI-Lab/dsh-plugins status --short
git -C /Users/yifanxu/Ephemeral-AI-Lab/deepseek-harness status --short
```

Preserve pre-existing user changes. Never reset, clean, or revert unrelated
work.

## Read before starting

Read these files in order:

1. [`README.md`](README.md) for the package inventory and installation examples.
2. [`docs/project.md`](docs/project.md) for DSH architecture, Cordis composition,
   Host versus agent planes, and session lifecycle.
3. [`docs/ephemeral_ai_harness_preset.md`](docs/ephemeral_ai_harness_preset.md)
   for the Web profile loading, restart, and validation contract.
4. [`docs/tools.md`](docs/tools.md) when changing model-visible tools or tool
   composition.
5. The target plugin's own `README.md`, `package.json`, `cordis.patch.yml`,
   `src/index.ts`, tests, and `SPEC.md` or other implementation notes.

Use the host repository only as read-only reference when the task requires
host behavior details:

- `deepseek-harness/README.md`
- `deepseek-harness/apps/cli/README.md`
- `deepseek-harness/apps/cli/reference/README.md`
- `deepseek-harness/docs/user/develop/basic/publish.md`
- `deepseek-harness/docs/subsystems/client-modules.md` for client HMR behavior

The architecture guide records a historical DSH baseline. Confirm current
host behavior from the checked-out source before relying on version-sensitive
details.

## Repository map

```text
dsh-plugins/
├── plugins/              Official plugin sources (independently buildable packages)
│   ├── codex-terminal/   Persistent exec_command/write_stdin tools
│   ├── coding-plan/      Codex and Grok subscription providers
│   ├── loop/             Recurring prompts and loop UI
│   ├── mock/             Deterministic model/replay testing
│   ├── preset-builder/   Preset configuration UI
│   ├── sessions/         Session discovery and messaging
│   ├── sidechat/         Web workbench side-chat panel
│   └── work-bench-ui/    Web workbench frame
├── registry/             Marketplace manifests (see docs/market.md)
│   ├── schema/           plugin-manifest.v1.json
│   ├── official/         one manifest per Ephemeral AI Lab plugin
│   ├── dsh/              DeepSeek optional plugin listings
│   ├── community/        third-party submissions (PR-reviewed)
│   ├── submission-template.json
│   └── review-checklist.md
├── scripts/              validate-manifests / verify-packages / build-index
├── dist/                 generated index.json + catalog.json (workflow-owned)
└── docs/                 Architecture, tool references, how-to guides, market spec
```

Each plugin is an independently buildable package. Its `cordis.patch.yml` is
part of the runtime contract. Treat `package.json`, the patch, source, tests,
and generated `lib/` output as one package workflow.

## Marketplace rules

- Registry manifests are discovery/install metadata only; never make them a
  runtime dependency. The manifest spec is `docs/market.md`.
- `dist/` is generated. Never edit it by hand; the `index-publish` workflow
  owns it through an auto-merged PR.
- Every registry change must pass `node scripts/validate-manifests.mjs` and
  `node scripts/verify-packages.mjs` (the latter checks npm and scratch-installs
  GitHub sources; run with `--offline` when network access is unavailable, or
  `--skip-install` to skip only GitHub scratch installs).
- Official plugins install from GitHub (`github:...#main&path:plugins/<name>`)
  because the `dsh-*` npm names are currently unpublished. Their `lib/` build
  output is committed on purpose — git installs fetch sources, so rebuild and
  re-commit `lib/` when releasing a plugin. Plugins without committed `lib/`
  (none today) must not declare a GitHub source.
- When moving or renaming a plugin directory, update the matching manifest's
  `install.rows[].github.subdir` in the same change.

## Local plugin workflow

Use absolute paths when installing a checkout into a DSH profile:

```sh

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ephemeral-AI-Lab/dsh-plugins](https://github.com/Ephemeral-AI-Lab/dsh-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
