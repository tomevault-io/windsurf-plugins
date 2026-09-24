---
trigger: always_on
description: Guidance for AI coding agents (Codex, Claude Code, Cursor, and similar) working
---

# AGENTS.md

Guidance for AI coding agents (Codex, Claude Code, Cursor, and similar) working
in this repository. Humans should start with [README.md](README.md).

> [`agent.md`](agent.md) is the user-facing install and setup guide for agents.
> Project-internal outreach operations live under [`ops/outreach/`](ops/outreach/).

## What this project is

`codex-profiles` is a single-file, dependency-free Bash CLI for named Codex
homes and named ChatGPT windows with separate local state. Every profile
selects a `CODEX_HOME`; named macOS Desktop launches additionally select
Electron user data for the entire ChatGPT window across Chat, Work, and Codex.
The whole program is [`bin/codex-profile`](bin/codex-profile).

It is community-maintained and is **not** an official OpenAI project.

## Repository layout

- `bin/codex-profile` — the entire CLI (Bash). Edit this for behavior changes.
- `agent.md` — user-facing install and profile-setup instructions for AI agents.
- `.agents/skills/` — repo-local Codex outreach workflow skills.
- `ops/outreach/` — project-internal distribution prompt, launch playbook, and
  tracker runbook.
- `scripts/check` — canonical syntax, test, lint, and full-check dispatcher.
- `scripts/release/` — directly tested release-channel programs; workflow YAML
  only maps permissions, inputs, secrets, and ordering.
- `scripts/aur/` — non-pushing AUR preparation and read-only verification.
- `test/` — suites grouped by `cli`, `install`, `packaging`, `release`, `site`,
  and `outreach`; shared infrastructure lives in `test/lib` and fixtures in
  `test/fixtures`.
- `docs/` — GitHub Pages site plus `llms.txt`, `robots.txt`, and `sitemap.xml`.
- `Makefile` — stable delegates including `check`, `test`, `lint`, install, and
  package smoke targets.
- `CHANGELOG.md` — Keep a Changelog format; add entries under `## Unreleased`.

## Setup, build, and test

There is no build step. Run the complete local gate, including syntax, all
behavior suites, and ShellCheck:

```sh
make check
```

For focused iteration, behavior tests and lint remain separate:

```sh
make test
make lint
bash test/cli/profiles-test.sh
node test/release/workflow-contract-test.mjs
```

Install locally from source (copies `bin/codex-profile` to `~/.local/bin`):

```sh
make install
```

## How to run the tool

```sh
codex-profile init work                  # create the work profile's CODEX_HOME
codex-profile login work                 # authenticate that profile once
codex-profile cli work                   # Codex CLI on the work profile
codex-profile cli work exec "run tests"  # one-shot Codex CLI command
codex-profile app default ~/Dev/project  # stock ChatGPT session (macOS)
codex-profile app work ~/Dev/project     # named ChatGPT window with separate local state
codex-profile status                     # read-only Codex-local overview
codex-profile doctor                     # environment diagnostics
```

Profile-to-path mapping: `default -> ~/.codex`; any other name `<x> -> ~/.codex-<x>`.

## Conventions for changes

- Keep the CLI dependency-free: Bash plus standard POSIX/macOS tools only. Do not
  add a runtime users would have to install.
- Run `make check` before proposing changes. If ShellCheck is unavailable, run
  `make test` and state the missing lint result explicitly.
- Put repository automation under `scripts/`, keep workflow YAML declarative,
  and place tests in the directory matching the responsibility they validate.
  Shared test helpers must provide infrastructure rather than product policy.
- Match the existing Bash style in `bin/codex-profile`: `set -euo pipefail`,
  `command_*` functions for subcommands, and the `die`/`note` helpers.
- If you change the command surface, update `usage()` in `bin/codex-profile`,
  the README command reference, the shell completion generators, and
  `docs/llms.txt` so all four stay in sync.
- Bump the version together in `bin/codex-profile`, `package.json`, both
  `package-lock.json` version fields, `docs/index.html`,
  `packaging/aur/PKGBUILD`, and `packaging/aur/.SRCINFO`. CI and the GEO test
  enforce this synchronization.
- Document user-facing changes under `## Unreleased` in `CHANGELOG.md`. Release
  preparation then promotes those entries into a dated `## <version> - <date>`
  section matching the newly tracked version, and the GEO test requires that
  dated heading to exist. Leave already-tagged entries in the section whose tag
  shipped them.
- Keep the scope contract explicit: `cli`/`login`/`env`/`use` are Codex-only;
  `app default` preserves stock ChatGPT Desktop state; named `app` launches use
  matching `CODEX_HOME` and Electron data for the entire ChatGPT window.
- Do not add `--shared-home`, auth.json copying, or store-level session
  symlinks. Current Codex Desktop canonicalizes rollout paths, so a `sessions/`
  link that escapes `CODEX_HOME` breaks fork and side chats. Desktop identity
  follows `CODEX_HOME/auth.json`, not Electron data alone. Downstream wrappers
  may decouple those; this tool must not.
- Desktop code must launch the original signed bundle. Do not reintroduce app
  clones, metadata patching, ad-hoc signing, global quitting, or broad kills.
- Keep `--instance`, `--rebuild`, and `app-instance` as deprecated

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ducksss/codex-profiles](https://github.com/Ducksss/codex-profiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
