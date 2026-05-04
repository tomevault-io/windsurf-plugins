---
trigger: always_on
description: Focused runtime for coding agents. Exposes an HTTP + SSE API for session
---

# AnyHarness

Focused runtime for coding agents. Exposes an HTTP + SSE API for session
management, agent orchestration, and tool execution.

- Repo: <https://github.com/proliferate-ai/proliferate>

## Build & Dev

Runtime baseline: Rust stable, Node 22+, pnpm, Python 3.12, and `uv` for the
server.

```bash
# Runtime
cargo build
cargo run -- serve

# Core SDK
cd anyharness/sdk
pnpm install
pnpm run generate
pnpm run build

# Server
cd server
uv run pytest -q
```

## Local Full-Stack Profiles

Use `make dev PROFILE=<name>` for full-stack local development, especially when
multiple worktrees need to run at the same time. Do not use the default-port
`make dev-runtime`, `make dev-server`, or `make dev-desktop` shortcuts for
multi-worktree testing.

Useful commands:

```bash
make dev-init PROFILE=<name>
make dev-list
make dev PROFILE=<name>
make dev PROFILE=<name> STRIPE=1
```

Profile state lives under
`~/.proliferate-local/dev/profiles/<name>/`; AnyHarness runtime state lives
under `~/.proliferate-local/runtimes/<name>/`. Read
`docs/reference/dev-profiles.md` before changing profile launch behavior,
ports, generated Tauri config, or dev app identity.

## Read This First

Start with:

1. `docs/README.md`

The docs under `docs/**` are the authoritative standards for their area. If
this file overlaps with an area doc, the area doc wins.

You MUST READ the relevant area doc before touching code in that area.
Always do so at the start of the task, not halfway through implementation.

## Area Read Order

### Frontend (`desktop/src/**`)

1. `docs/frontend/README.md`
2. `docs/frontend/styling.md` when the change touches UI styling, primitives,
   tokens, or theme usage
3. `docs/frontend/chat-composer.md` when the change touches the chat composer,
   the panels above it (todo tracker, approval card, workspace/cloud status),
   workspace review panels/defaults, or the Claude plan card in the transcript

### SDK (`anyharness/sdk/**`, `anyharness/sdk-react/**`)

1. `docs/sdk/README.md`

### Server (`server/**`)

1. `docs/server/README.md`

### AnyHarness Runtime (`anyharness/crates/**`)

1. `docs/anyharness/README.md`
2. `docs/anyharness/contract.md` if the change touches contract schemas
3. `docs/anyharness/binary.md` if the change touches the binary crate
4. the relevant subsystem doc under `docs/anyharness/src/**` when the change
   touches runtime logic

### CI/CD, Release, and Deployment (`.github/workflows/**`, `desktop/infra/**`, `server/infra/**`, updater publishing, desktop updater flow)

1. `docs/ci-cd/README.md`

If the release or deployment change also touches frontend, server, SDK, or
AnyHarness code, read the relevant area doc too.

## Repo-Wide Expectations

- Read the relevant area doc as early as possible before editing code in that
  area.
- Preserve current behavior unless an explicit behavior change is requested.
- Prefer ownership-correct extractions over cosmetic churn.
- Do not leave duplicate old and new code paths behind after a migration.
- Keep the repo buildable and run targeted verification when feasible.
- Follow the area docs before inventing new folders, layers, or patterns.

## Desktop Release Procedure

**NEVER trigger `Release Desktop` via `workflow_dispatch` on `main`.** The
updater manifest version is derived from `${GITHUB_REF_NAME#desktop-v}`. When
triggered on `main`, the manifest gets `version: "main"` instead of valid
semver, and the Tauri updater silently ignores it — users never see the update.

Correct procedure:

1. Bump the version in all three files (must match):
   - `desktop/package.json`
   - `desktop/src-tauri/tauri.conf.json`
   - `desktop/src-tauri/Cargo.toml`
2. Commit and push to `main`.
3. Create and push a tag: `git tag desktop-v<VERSION> && git push origin desktop-v<VERSION>`
4. The workflow triggers automatically on the tag push, or run manually
   **from the tag ref**: `gh workflow run "Release Desktop" --ref desktop-v<VERSION>`



----


# Repeated version of `docs/frontend/README.md`



# Frontend Standards

Status: authoritative for frontend code in this repo.

Scope:

- `desktop/src/**`

Use this doc to decide where new frontend logic goes and what rules it must
follow. Read [styling.md](styling.md) only when the change touches styling,
primitives, tokens, or theme usage.
Read [telemetry.md](telemetry.md) when the change touches analytics, error
capture, Sentry, PostHog, replay masking, or telemetry payloads.
Read [chat-composer.md](chat-composer.md) when the change touches the chat
composer area — the input, the panels that sit above it (todo tracker,
approval card, workspace status, cloud runtime), or the Claude plan card in
the transcript.

Cloud note:

- The cloud guidance below describes the target architecture now, even where
  some current hooks are still transitional.

Component note:

- The component guidance below describes the target architecture now, even
  where some current folders still reflect an older mixed layout.

## 1. File Tree

```text
desktop/src/
  App.tsx
  main.tsx
  assets/
  components/
    ui/
    <domain>/
  config/
  hooks/
    ui/
    <domain>/
  lib/
    domain/
      <domain>/
    infra/
    integrations/
      cloud/
      anyharness/
  pages/
  platform/
    tauri/
  providers/
  stores/
    <domain>/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [proliferate-ai/proliferate](https://github.com/proliferate-ai/proliferate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
