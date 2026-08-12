---
trigger: always_on
description: > **Architecture source of truth is [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md)** — what LinkCode is, the host/agent/abstraction layers, the data-plane vs system-plane split, the package map, the core principles you must follow, the key contracts, and the **open questions you must never answer yourself; ask first**. The runbook (prerequisites, running the apps, tests, E2E, triage) is [`docs/DEVELOPMENT.md`](docs/DEVELOPMENT.md); release/signing/notarization/update-feed is [`docs/RELEASE.md`](
---

# AGENTS.md

> **Architecture source of truth is [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md)** — what LinkCode is, the host/agent/abstraction layers, the data-plane vs system-plane split, the package map, the core principles you must follow, the key contracts, and the **open questions you must never answer yourself; ask first**. The runbook (prerequisites, running the apps, tests, E2E, triage) is [`docs/DEVELOPMENT.md`](docs/DEVELOPMENT.md); release/signing/notarization/update-feed is [`docs/RELEASE.md`](docs/RELEASE.md). This file is the always-loaded discipline + routing layer on top of them.

## Invariants — ranked by blast radius

Each of these breaks the product, a release, or the build with **no loud error**.

1. **Two wire versions, and only one of them is lockstep.** Both live in `packages/foundation/schema/src/wire/message.ts` (read the values there — they are deliberately not repeated here). `WIRE_PROTOCOL_VERSION` is what a build stamps and **every** wire change bumps it. `MIN_COMPATIBLE_WIRE_VERSION` is the oldest a build still accepts, and it moves **only for a breaking change** — a variant or field removed, renamed, or given a new meaning. Get that call wrong in the additive direction and nothing breaks; get it wrong in the breaking direction and peers silently misread each other. An unrecognized `kind` from a newer peer is dropped by itself (logged once per connection) and the connection lives on, so adding a frame no longer forces every peer to upgrade together. A peer *below* the floor is still the hard case: its frames are refused, it never answers a `ping`, and the handshake ends in the 5s timeout — the drop is logged, but only an out-of-band probe can name it (CODE-447).
2. **`foxts/once` prewarms by default.** `once(fn)` runs `fn` immediately at construction and caches the result; call-at-most-once semantics need `once(fn, false)`. The default has already shipped a daemon that ran its shutdown at boot and transports whose close-callback fired at construction. Read any foxts helper's `.d.ts`/source before adopting it — the lodash-alike name lies.
3. **Native deps must be allow-listed.** pnpm blocks install scripts by default; a native dep (e.g. `better-sqlite3`) missing from `allowBuilds:` in `pnpm-workspace.yaml` installs fine but fails at `require()` time with missing bindings.
4. **`check:ci` does not run `pnpm test`.** CI runs vitest as a separate TypeScript-job step, while `check:ci` remains format/lint/typecheck only. Run both commands before every commit; passing either one alone is not the complete JavaScript gate.
5. **A release is a version+tag pair.** Bump `apps/desktop/package.json` `version`, then push a `v*.*.*` tag; CI fails unless `v${version}` equals the tag. Never hand-tag or hand-edit workspace versions ([`docs/RELEASE.md`](docs/RELEASE.md)).
6. **Every daemon-side child process sets `windowsHide: true`.** Node defaults it to `false`, and the daemon runs console-less (Electron `utilityProcess`), so a console-subsystem child spawned without it — agent CLI, git, sidecar, bsdtar — pops a visible console window on packaged Windows only; silent everywhere else, dev included. Applies to every `spawn`/`exec*`/cross-spawn call in `apps/daemon` and `packages/host/{engine,agent-adapter,assets}` (CODE-236 swept all sites 2026-07 — keep new ones consistent). SDK-internal spawns are out of reach: claude's SDK hides its own; opencode servers are spawned by our own `native/opencode/serve.ts` (CODE-76), not the SDK, so the flag applies there too.

## Routing — touching X, read Y first

| You're about to… | Read first |
| --- | --- |
| Run, test, E2E, or debug the app locally | [`docs/DEVELOPMENT.md`](docs/DEVELOPMENT.md) |
| Read, add, or override an environment variable | [`docs/ENVIRONMENT.md`](docs/ENVIRONMENT.md) |
| Touch the renderer (React, UI, icons, i18n, client state) | [`.claude/rules/frontend.md`](.claude/rules/frontend.md) |
| Touch Electron main / preload / CSP / cloud-auth wiring | [`apps/desktop/AGENTS.md`](apps/desktop/AGENTS.md) |
| Integrate or change an agent (claude-code, codex, opencode, pi), approvals, history | [`packages/host/agent-adapter/AGENTS.md`](packages/host/agent-adapter/AGENTS.md) |
| Work on the daemon: ports, `runtime.json`, spawn, PTY sidecar | [`apps/daemon/AGENTS.md`](apps/daemon/AGENTS.md); triage lives in [`docs/DEVELOPMENT.md`](docs/DEVELOPMENT.md) |
| Package, sign, notarize, or publish a release | [`docs/RELEASE.md`](docs/RELEASE.md) + [`apps/desktop/AGENTS.md`](apps/desktop/AGENTS.md) |
| Change the wire protocol, schema, or transport | [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) contracts + `packages/foundation/schema` (Invariant 1) |
| Work on mobile (Expo) | [`apps/mobile/AGENTS.md`](apps/mobile/AGENTS.md) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arcboxlabs/linkcode](https://github.com/arcboxlabs/linkcode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
