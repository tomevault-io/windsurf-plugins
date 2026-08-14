---
trigger: always_on
description: This file is the shared project memory for BrowserPane. Keep it short, code-aligned, and current.
---

# BrowserPane Agent Guide

This file is the shared project memory for BrowserPane. Keep it short, code-aligned, and current.

Project-wide Rust coding standards live in `RUST_STANDARDS.md`.
- Apply them to all Rust crates in this repo.
- Update that file instead of expanding this one with detailed Rust style rules.

Project-wide TypeScript and Node.js coding standards live in `NODEJS_STANDARDS.md`.
- Apply them to `code/web/bpane-client`, `code/web/bpane-admin`,
  `code/web/bpane-admin-unified`, `code/integrations/mcp-bridge`,
  `code/integrations/recording-worker`,
  `code/integrations/workflow-worker`, and future TS/Node packages.
- Update that file instead of expanding this one with detailed TS/Node style rules.

When docs disagree, prefer:
1. The code
2. Runtime manifests and package scripts
3. This file
4. `README.md`

For the frozen owner-scoped session-control contract, use `openapi/bpane-control-v1.yaml`.

## What BrowserPane is

BrowserPane is a browser-native remote browser/desktop stack for a Linux host container.

Current product shape:
- A Linux container runs Xorg dummy + Openbox + Chromium.
- `bpane-host` captures and classifies the surface.
- `bpane-gateway` exposes WebTransport plus legacy and versioned HTTP APIs.
- Phase 0 session resources are persisted in Postgres behind the gateway.
- The browser client renders a tile-first stream with optional ROI H.264 video.
- Shared sessions are collaborative by default; optional exclusive-owner mode can lock later browser clients into read-only viewers.

## Current support matrix

- Host runtime: Linux only. Ubuntu 24.04 container is the primary target.
- Browser runtime: Chromium desktop only. Firefox and Safari are not production targets.
- Shared sessions: supported for small curated groups, not broadcast-scale delivery.
- Exclusive browser-owner mode: optional in `bpane-gateway` via `--exclusive-browser-owner`; default is disabled.
- Viewer cap: configurable in `bpane-gateway` via `--max-viewers`, default `10` for restricted browser viewers.
- MCP automation: supported via `mcp-bridge` and gateway ownership APIs.
- Service-principal registry: owner-scoped external OIDC client metadata is supported through `/api/v1/service-principals`; disabled registered principals cannot be assigned as new automation delegates.
- Browser extensions: owner-approved unpacked extensions are supported for docker-backed sessions and workflow runs; `static_single` does not support session extension sets.
- Project policies can restrict live browser uploads/downloads, session-file
  bindings, and manual recording starts for project-scoped sessions. Session
  resources expose `capabilities.file_transfer=false` when a project disables
  either live browser upload or live browser download transfer.
- Egress traffic logging is proxy-side. BrowserPane should expose sanitized
  session/profile/container correlation metadata, while the configured egress
  proxy or secure web gateway owns outbound URL/status/timing and full traffic
  logs. BrowserPane can ingest sanitized per-session receive/transmit byte
  deltas for project usage and alerting, but must not ingest requested URLs,
  headers, proxy credentials, payload contents, decrypted traffic, or raw CA
  material.
  Egress profiles can be owner-scoped or project-scoped; project-bound profiles
  and project-bound proxy credential bindings are only usable by sessions in the
  same project. Session and egress-profile resources expose sanitized egress
  diagnostics so
  operators can distinguish configuration-only proof, runtime launch metadata,
  and active browser probe evidence without exposing requested URLs, proxy
  credentials, CA material, or decrypted traffic. Active browser probes run only
  against already-ready session runtimes; diagnostics calls must not implicitly
  start stopped sessions.
  Full HTTPS inspection must be explicit through an egress profile
  `traffic_observation.mode=tls_intercept` with proxy, custom CA, and approved
  sensitive-log sink references.
- Camera ingress: disabled by default in compose; requires browser H.264 encode support and a mapped `v4l2loopback` device on the host.
- In exclusive-owner sessions, restricted browser viewers are view-only: no input, clipboard, microphone, camera, upload, download, or resize.

## Architecture map

- `code/apps/bpane-host`
  - Linux host agent. Main orchestration lives in `src/main.rs`.
  - `capture/`: X11 capture and ROI video capture support.
  - `tiles/`: tile classification and Fill/QOI/zstd emission.
  - `audio/`: desktop audio out and microphone ingest.
  - `camera.rs`: H.264 browser camera ingress to virtual camera.
  - `clipboard.rs`, `filetransfer.rs`, `input/`, `resize.rs`: host-side interaction plumbing.
- `code/apps/bpane-gateway`
  - WebTransport gateway and shared-session coordinator.
  - `lifecycle.rs`: shared starting/running/draining state, signal handling, and bounded listener/task drain coordination.
  - `readiness.rs`: concurrent, timeout-bounded readiness checks for the configured session store, runtime manager, credential provider, and artifact stores.
  - `metrics.rs`: gateway-owned OpenMetrics registry, bounded HTTP RED labels,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ITmedes/browserpane](https://github.com/ITmedes/browserpane) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
