---
trigger: always_on
description: Native iOS companion app for herdr (https://herdr.dev): an agent console over SSH, not a terminal app. Read `CONTEXT.md` for vocabulary and `docs/adr/` before challenging architecture decisions — the transport design in particular was reached after eliminating several dead ends.
---

# Heeler

Native iOS companion app for herdr (https://herdr.dev): an agent console over SSH, not a terminal app. Read `CONTEXT.md` for vocabulary and `docs/adr/` before challenging architecture decisions — the transport design in particular was reached after eliminating several dead ends.

## Architecture

- **Stack**: SwiftUI, iOS 18+, iPhone + iPad. SSH via the repository-local `Packages/HeelerSSH` (libssh2 + OpenSSL), terminal rendering via the pinned libghostty-spm `GhosttyTerminal` product. See ADR 0001 (native stack), ADR 0003 (the superseded Dictation-era target raise), and ADR 0004 (terminal engine).
- **Transport**: herdr's JSON API (NDJSON over a remote Unix socket) reached through OpenSSH direct-streamlocal channels onto the socket itself — no socat, and no Host-side prerequisite beyond SSH access and a running herdr. A server that denies stream-local forwarding fails preflight rather than falling back. Interactive terminals request a PTY and exec `herdr agent attach` on it. See ADR 0011, which supersedes ADR 0002.
- The UI layer must depend on a transport abstraction (protocol), never on an SSH library's types directly.
- Sibling deliverables live in this repo: `plugin/` is the herdr plugin that renders Pairing Codes and posts Agent Notifications (Node, zero framework, `npm test`); `relay/` is the stateless Push Relay it posts to (dependency-free Node, `npm test`); `landing/` is the marketing site at `heeler.bybee.dev` (Astro, zero-JS static build on Cloudflare Workers, deployed by `.github/workflows/landing.yml`). iOS CI (`.github/workflows/ci.yml`) runs only on app/simulator paths; plugin, relay, and wire-type codegen run in `.github/workflows/ci-node.yml`; a landing copy edit never starts either. Wire types in `Sources/Heeler/Transport/Generated/` are produced by `scripts/generate-wire-types.py` from the committed schema snapshot `scripts/herdr-schema.json` — regenerate with `--schema scripts/herdr-schema.json` (the flag-less default shells out to a local `herdr` instead of the snapshot) rather than hand-edit; CI fails on drift via `--check`. The vectors in `plugin/test-vectors/` are consumed by both the Node and Swift suites so the two implementations cannot drift; change them in lockstep.

## Load-bearing herdr facts

Rediscovering these is expensive. Each fact retains its evidence version and method (live observation or source review); older observations are not re-tests on a newer release. For 0.9.0 subscription ordering, compatibility evidence, and remaining verification, read [the integration review](docs/research/herdr-0.9.0-compatibility.md).

- The herdr API socket serves **one request per connection** (read one line, write one line, close). Only `events.subscribe` keeps the connection open. Plan channel usage accordingly. (Re-verified live on 0.8.0: a second write on a served connection gets `EPIPE`. `pane.graphics.stream`, previously listed here, does not exist in the 0.8.0 schema — graphics methods are `pane.graphics.set`/`clear`/`info`, all one-shot.)
- Wire format: request `{"id": "<any string>", "method": "...", "params": {...}}` + `\n`; `params` is required, and `{}` satisfies it. Success `{"id", "result"}`, failure `{"id", "error": {"code", "message"}}` (both re-verified live on 0.8.0), subscription event lines `{"event", "data"}` with no id.
- The first message on any new connection path should be `ping` — it returns the server protocol version (0.8.2 answers `{"version":"0.8.2","protocol":20,...}`; 0.8.0, verified live, answered `{"version":"0.8.0","protocol":19,...}`). herdr's API has no stability guarantee; parse leniently (ignore unknown fields). The app enforces a **floor**, not equality: `HeelerSSHTransport.minimumProtocolVersion` refuses older servers, `generatedProtocolVersion` only drives an advisory notice. Equality here made every 0.8.0 Host unusable (#140) — do not restore it.
- The API schema is exported offline via `herdr api schema --json` (JSON Schema 2020-12). Its `$ref` paths are non-standard nested (`#/schemas/request/$defs/X`) — preprocess before feeding codegen tools. The committed 0.9.0 snapshot (protocol 22) declares 102 request methods and 26 event kinds (its event and subscription-event schemas are unchanged from 0.8.2), and every one of those kinds has a typed `EventData` variant; separately, 3 pane-scoped kinds (`pane.output_matched`, `pane.agent_status_changed`, `pane.scroll_changed`) have typed `SubscriptionEventData`. Payloads herdr actually emits are still worth verifying empirically.
- Pane ids are opaque strings. Protocol-22 `scripts/herdr-schema.json` declares `pane_id` as a string with no `pattern`. Observed herdr 0.7.5 captures (`w1:pA`, `w3:pB`, `wV:p1`) and live 0.8.0 `agent.list` samples (`w1:pT`, `w1C:p1`, `wR:pC`, `wV:p1H`) are alphanumeric `w…:p…` identifiers and include uppercase letters. Those captures contain no tmux-style `%N`; that does not prove herdr can never emit it. Treat every pane id as an opaque string; do not parse or validate a grammar.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZingerLittleBee/Heeler](https://github.com/ZingerLittleBee/Heeler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
