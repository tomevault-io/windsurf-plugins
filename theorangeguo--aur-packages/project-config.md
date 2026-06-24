---
trigger: always_on
description: - This repo is an Arch Linux AUR package monorepo.
---

# AGENTS.md

## Purpose
- This repo is an Arch Linux AUR package monorepo.
- Source-of-truth PackageSpec v1 definitions live in `package.toml`, with optional `hooks.sh` and optional `files/` assets.
- `PKGBUILD` and `.SRCINFO` are generated only in temporary workspaces during local runs and CI.
- Prefer small, package-scoped changes over broad cleanup.
- Do not introduce new tooling or languages unless the user asks.

## Read first
- `README.md`
- `docs/CONTRIBUTING.md`
- `docs/INTEGRATION.md`
- `.github/workflows/aur-publish.yml`
- `.github/workflows/package-test.yml`
- `scripts/ci.sh`
- `scripts/aurpkg.py`

## Repository facts and local rules
- No Cursor rules were found in `.cursor/rules/` or `.cursorrules`.
- No Copilot instructions were found in `.github/copilot-instructions.md`.
- CI auto-discovers package directories by locating PackageSpec v1 `package.toml` files.
- Scheduled AUR publishing first runs upstream-only update detection and then dispatches only changed package jobs.
- The current package state baseline comes from the AUR repo, not from this monorepo.
- Keep workflow YAML thin; most behavior belongs in `scripts/`.
- Build steps must run as the non-root `builder` user.
- If you create a git commit for the user, push it to the relevant remote branch in the same workflow unless the user explicitly says not to push.

## Repository workflow
- Main flow: discover packages -> detect upstream changes -> read AUR state -> resolve upstream -> prepare declared artifacts -> render temporary `PKGBUILD` -> refresh checksums -> generate `.SRCINFO` -> build -> publish to AUR.
- Validation flow: discover packages -> resolve upstream -> prepare declared artifacts in safe local mode -> render temporary `PKGBUILD` -> build -> install package in a container -> run smoke checks.
- CI entrypoint: `scripts/ci.sh package-test-*` and `scripts/ci.sh aur-publish-*` keep workflow YAML thin and handle CI bootstrap/argument wiring.
- Framework entrypoint: `python3 scripts/aurpkg.py discover`, `python3 scripts/aurpkg.py detect-updates`, `python3 scripts/aurpkg.py preflight <pkgname-or-path>`, `python3 scripts/aurpkg.py prepare-artifacts <pkgname-or-path> ...`, `python3 scripts/aurpkg.py run-publish <pkgname-or-path> ...`, `python3 scripts/aurpkg.py run-test <pkgname-or-path>`
- `scripts/aurpkg.py` owns package framework behavior; `scripts/ci.sh` owns GitHub Actions orchestration only.
- When touching update logic, inspect `scripts/aurpkg.py` and any package-local `hooks.sh`.

## Framework contract rules
- Treat package definitions as a stable contract. `package.toml` is the current PackageSpec v1 frontend: strict TOML declarative data plus explicit extension points, not a programming language.
- Prefer mechanism over solution: add reusable framework components such as version/origin resolvers, source resolvers, packaging templates, artifact producers, install/service renderers, validation primitives, or publishers instead of package-specific workflow/script branches.
- Prefer composition over integration: packages should combine independent components (`template`, `[version]`, `[origins.*]`, `[inputs.sources.*]`, `[inputs.artifacts.*]`, `[install]`, `[service]`, `[tests]`) rather than opt into monolithic custom flows.
- Keep package behavior package-local. Root manifests, if introduced, may only control discovery scope such as `packages/*/spec.yml`; they must not hold package behavior or become a second source of truth.
- Keep component boundaries sharp: resolvers resolve upstream state only; artifact recipes produce artifacts only; artifact storage locates or publishes reusable artifacts only; sources declare PKGBUILD source entries only; templates render/build packages only; detectors optimize dispatch only; publishers validate and compare against live AUR state before push.
- PackageSpec v1 uses `[version]`, `[origins.*]`, `[inputs.sources.*]`, and `[inputs.artifacts.*]`: `version` decides package identity, `origins` provide external metadata, and `inputs` contains only things that become or produce packaging inputs.
- Keep hooks narrow. Existing `hooks.sh` should only resolve upstream state; future spec hooks should be phase-specific subprocesses with whitelisted outputs, not sourced code that mutates framework internals.
- Do not allow cross-package imports, remote includes, deep inheritance, loops, conditionals, or arbitrary command execution in package specs. Local files must be declared by role (patch, doc, license, service, wrapper, test asset), not blindly included.
- Version package specs with `spec_version`, normalize package definitions into the same internal model, and fail fast on unsupported major schema versions.

## GitHub Actions failure triage
- When investigating failed Actions, first use `gh run list` and `gh run view <run-id> --log-failed` to identify the exact failed package/job before editing.
- Distinguish package-specific failures from transient infrastructure failures. Treat AUR clone failures, GitHub API timeouts, and network download failures as potentially transient unless repeated.
- For GitHub release asset matching failures, inspect upstream release asset names and compare them against `[inputs.sources.*]` selectors in the affected package.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [theorangeguo/aur-packages](https://github.com/theorangeguo/aur-packages) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
