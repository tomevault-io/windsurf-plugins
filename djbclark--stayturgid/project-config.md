---
trigger: always_on
description: > **AI agents (any vendor):** this file is the entry point — the AGENTS.md
---

# stayturgid

> **AI agents (any vendor):** this file is the entry point — the AGENTS.md
> convention that coding agents from multiple vendors check first. Project
> overview for humans: [README.md](README.md). Current dated
> state — fleet health, active blockers, operator-action queue — lives in
> [docs/STATUS.md](docs/STATUS.md); the most load-bearing parts are inlined
> below so you have them even if you stop at this file. See "Where
> documentation goes" further down for what belongs where.

Keeps wireless ADB (port 5555), Shizuku, and SSH alive on unrooted Android phones
across reboots. Generic example fleet hosts (`oneui-device`, `stock-android-device`,
`fireos-device`) live in `ansible/inventory/hosts.yml.example`; live inventory
belongs in a private site overlay (see
[multi-site-topology.md](docs/architecture/multi-site-topology.md) §4).

## Current state (condensed — full/current version: [docs/STATUS.md](docs/STATUS.md))

**This section only lists active blockers and operator-gated items — it is
not the full work menu.** Before selecting new work, check
[docs/options.md](docs/options.md) (strategic/deferred tracks) and the
[open GitHub issues](https://github.com/djbclark/stayturgid/issues) (discrete
bugs/follow-ups) for what's actually available to pick up next.

**Active blockers:**

- K1 native-agent cutover (2026-07-22) is **not fully verified** — AutoJs6
  removal was live-checked 2026-07-25 (the cutover's claim was false; fixed
  fleet-wide now) but the forced `CLOSED_NO_SHELL` soak still hasn't run.
  Tracked in [#43](https://github.com/djbclark/stayturgid/issues/43) and
  [#45](https://github.com/djbclark/stayturgid/issues/45).
- OpenObserve↔Vector auth **fixed 2026-07-25**, pending 24h clean-log
  verification before closing. Tracked in
  [#44](https://github.com/djbclark/stayturgid/issues/44) — see
  [docs/STATUS.md](docs/STATUS.md) for the root cause.

**Operator-action queue (things only a human can do):**

1. Physically check offline fleet devices (Tailscale unreachable).
2. Decide the F1 consent-surface phasing question ([#46](https://github.com/djbclark/stayturgid/issues/46)).
3. Remove (or authorize removal of) a stray `~/stayturgid` file — the real repo is `${OPS_ROOT:-~/ops}/stayturgid`.

If any of this looks stale, trust `docs/STATUS.md` and `git log` over this
section — it's a snapshot, not updated every commit.

## Quick start

```bash
cd ${OPS_ROOT:-~/ops}/site-djbclark && just ops-release-status
cd ${OPS_ROOT:-~/ops}/stayturgid
just health && just firerpa-health
```

## Versioned deploy releases — retired

**Retired 2026-08-23.** Work directly in `${OPS_ROOT:-~/ops}` with ordinary
git: edit, commit to `master`, push. Worktrees and `ops-vMAJOR.MINOR.PATCH`
releases still work but are optional; `~/ops` is live, so commits land with no
release gate in front of them. Rationale: "Where work happens" in
`~/CLAUDE.md`. Commands and full procedure:
`${OPS_ROOT:-~/ops}/site-djbclark/docs/OPS-RELEASES.md`.

## Key commands

The full command table lives in **[docs/commands.md](docs/commands.md)** —
moved there 2026-08-24 because it is reference material and this file is
loaded into context every session.

> **If you cannot read `docs/commands.md`, say so in your reply rather than
> guessing at commands.** It is a normal file in this repository; being unable
> to open it means something is wrong with your checkout or your file access,
> and the operator wants to know.

Day to day: `just health`, `just errors`, `just firerpa-health`, `just test`.

## Environment

- **Orchestration:** `just` (command runner, replaces `make`). The Makefile was migrated to a `justfile` in July 2026. Install: `brew install just`. Run `just --list` to see all targets or `just` for categorized help.
- **Mac shell:** `/bin/bash` (dotfiles: `~/.bash_profile`, `~/.bashrc`)
- **FIRERPA venv:** Python 3.12 at `~/.venv-stayturgid-firerpa` — `source ~/.venv-stayturgid-firerpa/bin/activate`
- **Python tooling:** `uv` (package manager) + `ruff` (linter/formatter) — `brew install uv ruff`
- **JavaScript tooling:** `bun` (package manager) — `brew install oven-sh/bun/bun`; `biome` (linter/formatter) — `brew install biome`
- **Shell tooling:** `shellcheck` (linter) + `shfmt` (formatter) — `brew install shellcheck shfmt`
- **Markdown tooling:** `markdownlint` (linter) + `prettier` (formatter) — `brew install markdownlint-cli prettier`
- **Ansible tooling:** `ansible-lint` (linter) + `yamllint` (linter) — `uv tool install ansible-lint yamllint`
- **INI tooling:** `pyinilint` (linter) — `uv tool install pyinilint`
- **Web tooling:** `html-validate`, `stylelint`, `pa11y`, `puppeteer`, `vnu-jar` — `bun install` (devDependencies); `lychee` (link checker) — `brew install lychee`; `lighthouse` (full-page audit) — `npm install -g lighthouse` (requires Chrome)
- **Other linters:** `dotenv-linter` (.env) — `brew install dotenv-linter`; `caddy` (Caddyfile fmt) — `brew install caddy`
- **Git tooling:** `pre-commit` (hooks) + `typos` (spell check) — `brew install pre-commit typos-cli`; run `pre-commit install`
- **SSH CA:** `~/.ssh/stayturgid_ca` — `just ca-status`
- **OpenCode web:** site-local service (see site overlay / landing); not a public fixed IP

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [djbclark/stayturgid](https://github.com/djbclark/stayturgid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
