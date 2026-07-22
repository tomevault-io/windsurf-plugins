---
trigger: always_on
description: This file is the operating context for coding agents working in this repository.
---

# Skirk Agent Handbook

This file is the operating context for coding agents working in this repository.
Read it before changing code, tests, docs, packaging, or release automation.

## Product Summary

Skirk is a Google Drive backed TCP transport for hostile networks where the
available path is constrained to Google-fronted traffic. The client exposes one
or more local frontends:

- SOCKS5 proxy
- HTTP proxy
- Windows desktop proxy/system-proxy/VPN modes
- Android `VpnService` mode

The exit runs on a server with normal Internet access. Both ends share a Google
Drive mailbox, exchange encrypted mux objects, and the exit opens outbound TCP
connections on behalf of the client.

The main network constraint is intentional: preserve the Google-fronted,
pinned-Google-IP route behavior used by `google_front_pinned`. Do not replace it
with a design that assumes direct raw access from the hostile network.

## Non-Negotiable Security Rules

- Never commit files under `private/`, `.skirk-runs/`, `skirk-kit/`,
  `skirk-config/`, `bin/`, or `dist/`.
- Never commit OAuth refresh tokens, access tokens, Google service account
  material, OAuth client secrets, `.skirk` files, `skirk:` config strings,
  generated `client.json`, generated `exit.json`, keystores, or certificates.
- Treat a one-line `skirk:` profile as a password. It carries enough material
  for a client to use the mailbox.
- Treat generated exit configs as secrets. They contain Google OAuth material.
- Release builds inject the public Skirk OAuth client and Android signing
  material through GitHub repository secrets; do not hard-code those values in
  tracked source.
- Run `scripts/preflight.sh` before release. It intentionally checks for common
  tracked runtime artifacts, personal email residue, and generated credentials.

## Current Production Transport

Mux v4 is the production transport. The implementation is in
`internal/skirk/mux.go`.

Key properties:

- Four Drive lanes carry many logical streams.
- Frames are encrypted and coalesced into Drive objects.
- Priority traffic carries stream opens, resets, first bytes embedded in opens,
  and ordered small-stream follow-on data/FIN while the stream remains under the
  small-stream threshold.
- Normal traffic carries demoted and bulk data with bounded per-stream and
  global queues.
- Client response objects are namespaced by client ID and run ID so multiple
  devices can use a copied profile without consuming each other's responses.
- Upload/download worker windows adapt to Drive health.
- Processed objects are cleaned up by deferred cleanup so foreground traffic is
  not blocked by delete calls.

Do not resurrect old transports or experimental protocols as defaults unless a
new design beats mux v4 on mixed browser plus bulk traffic. Synthetic
single-stream download speed is not enough.

## Performance Model

Drive is an object API, not a stream API. The hot path always includes upload,
Drive visibility delay, prefix discovery, download, and cleanup. The practical
goal is to minimize avoidable objects and keep interactive traffic moving while
bulk traffic is active.

Important lessons from previous work:

- `files.list` prefix polling is the proven low-latency discovery path for this
  design.
- `changes.list` is not prefix-filtered. It can be useful for research, but a
  production design must handle mailbox-wide pollution and extra bookkeeping.
- Known-ID and range-read primitives are fast after an ID is known, but previous
  live transports lost to mux v4 when they added extra control objects or
  metadata waits.
- Bulk-only throughput can be misleading. Promotion requires small request
  latency under active downloads, browser startup behavior, and multi-client
  stability.

## Repository Map

Top-level files:

- `README.md`, `README.fa.md`: user-facing overview.
- `CHANGELOG.md`: release notes.
- `LICENSE`, `DISCLAIMER.md`, `SECURITY.md`, `CONTRIBUTING.md`: project policy.
- `install.sh`: Linux installer used by the public quick-start command.
- `Makefile`: local build, test, preflight, and packaging entry points.
- `.github/workflows/ci.yml`: CI validation.
- `.github/workflows/release.yml`: tag-triggered release build and publish.

Command package:

- `cmd/skirk/main.go`: command dispatch and CLI flags.
- `cmd/skirk/setup.go`: Google kit creation, OAuth login, mailbox setup, and
  optional exit service startup.
- `cmd/skirk/oauth_wizard.go`: personal OAuth setup guidance.
- `cmd/skirk/menu.go`: interactive operator menu.
- `cmd/skirk/service.go`: Linux systemd service lifecycle.
- `cmd/skirk/uninstall.go`: local uninstall, optional kit deletion, OAuth
  revocation, and Drive deletion flow.
- `cmd/skirk/client_ui.go`: optional desktop-style browser dashboard.
- `cmd/skirk/parent_watch_*.go`: platform-specific parent process watching.
- `cmd/skirk/signals_*.go`: platform-specific shutdown signal sets.

Core package:

- `internal/skirk/config.go`: config structs, defaults, inline `skirk:` config
  encoding/decoding, OAuth token source.
- `internal/skirk/drive.go`: Google Drive API operations, listing, upload,
  download, delete, cleanup, quota accounting, and limiter behavior.
- `internal/skirk/httpclient.go`: route-aware HTTP transport and Google-fronted

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ShahabSL/Skirk](https://github.com/ShahabSL/Skirk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
