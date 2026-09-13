---
trigger: always_on
description: This file defines the repository-wide working agreement for coding agents and
---

# AGENTS.md

This file defines the repository-wide working agreement for coding agents and
automation operating in MirrorProxy. It applies to every file below the
repository root unless a more specific `AGENTS.md` is added in a subdirectory.

## Mission

MirrorProxy is a self-hosted source and package proxy composed of:

- a Rust server with an embedded React administration console;
- a cross-platform Rust CLI for configuring native package-manager clients;
- a shared catalog that keeps supported sources and capabilities consistent;
- packaging, installation, smoke-test, Wiki, and release automation.

Prefer narrow, compatible, observable changes. A change is complete only when
the relevant runtime path, tests, documentation, and delivery surface agree.

## Non-negotiable rules

1. Never commit credentials, private keys, cookies, tokens, production data,
   local databases, downloaded GeoIP databases, dependency directories, or
   generated `web/dist` assets.
2. Preserve backward compatibility for routes, configuration, databases,
   installers, and client output unless a breaking change is explicitly
   requested and documented.
3. Do not weaken authentication, authorization, CSRF, SSRF, path validation,
   TLS, secret encryption, quota, rate-limit, or audit controls to make a test
   pass.
4. Do not deploy, publish a release, close an issue, change repository security
   alerts, or mutate production unless the user explicitly authorizes it.
5. Preserve unrelated work in a dirty worktree. Never use destructive Git
   cleanup or broad rewrites to discard changes you did not create.
6. Use locked dependency resolution in validation and release builds. Keep
   dependency upgrades focused and explain any security or compatibility
   impact.
7. Treat a successful compile or HTTP 200 as partial evidence, not proof of an
   end-to-end behavior. Validate the actual protocol, payload, asset, digest,
   or live route relevant to the change.

## Repository map

- `crates/catalog`: shared source definitions, aliases, capabilities, and CLI
  generation metadata.
- `crates/client`: the `mirrorproxy` CLI and package-manager configuration
  writers, previews, rollback, and safety checks.
- `crates/server`: `mirrorproxy-server`, proxy adapters, configuration,
  database, authentication, administration APIs, observability, and embedded
  frontend serving.
- `web/src`: React/TypeScript console and its design system.
- `web/e2e`: Playwright browser tests.
- `web/scripts`: frontend and Docker-context contract checks.
- `scripts`: installers, packaging, GeoIP acquisition, and real smoke tests.
- `docs/wiki`: canonical English and Simplified Chinese Wiki source.
- `docs/releases`: detailed GitHub Release notes.
- `.github/workflows`: CI, CodeQL, Docker, Release, and Wiki automation.
- `config.example.toml`: public configuration reference; keep it aligned with
  validated server defaults and documented behavior.

## Start every task this way

1. Read the user request and classify it as inspect, diagnose, change, release,
   deploy, or monitor. Do not infer permission for a broader action.
2. Inspect `git status --short --branch`, relevant files, and recent history
   before editing.
3. Trace the actual call chain or delivery path. For example, a new source can
   touch catalog metadata, configuration, a server adapter, routing, the Web
   console, CLI output, documentation, and smoke coverage.
4. State any material assumption. Prefer a safe, discoverable default over
   blocking on a minor ambiguity.
5. Make the smallest coherent change and add regression coverage near the
   behavior being changed.

## Development prerequisites

- Rust stable, using the versions resolved by `Cargo.lock`.
- Node.js 24 and npm.
- Chromium installed through Playwright for browser E2E.
- `musl-tools` for the default x86_64 musl release build.
- Network access and ecosystem clients only for smoke tests that explicitly
  exercise public upstreams.

The server embeds `web/dist`. Build the Web console before building a server
artifact whose embedded UI must reflect frontend changes.

```bash
cd web
npm ci
npm run build
cd ..
cargo build --workspace --locked
```

For the canonical release-style local build, use `./build.sh`. It builds the
Web console, fetches verified GeoIP inputs, embeds Git metadata, and produces
server and client binaries for `TARGET` (default:
`x86_64-unknown-linux-musl`).

## Editing guidance

### Rust

- Keep protocol adapters strict: validate paths, reject traversal, do not
  forward client credentials or hop-by-hop headers, and preserve bounded
  streaming/cache behavior.
- Use existing configuration validation and redaction patterns. New secrets
  must never appear in debug output, API responses, logs, or persisted
  plaintext when the master-key mode applies.
- Keep network targets allowlisted. For dynamically discovered endpoints,
  enforce scheme/origin rules, validate resolved addresses, account for DNS
  rebinding, and avoid unsafe redirects.
- Maintain bearer-token automation separately from browser cookie flows; do not
  bypass browser CSRF checks for convenience.
- Add focused unit tests in the owning module. Prefer deterministic local test
  servers over mutable public services for unit tests.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [inbjo/MirrorProxy](https://github.com/inbjo/MirrorProxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
