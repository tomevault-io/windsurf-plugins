---
trigger: always_on
description: Ruby-native, model-agnostic agent harness where everything is a plugin. Twelve gems in one
---

# Terret

Ruby-native, model-agnostic agent harness where everything is a plugin. Twelve gems in one
repo:

- `gems/hames` is the kernel. Services in a context, typed events, reversible effects,
  dependency-driven boot. It knows nothing about LLMs and is reusable for any
  plugin-composed application.
- `gems/terret-core` is the harness built on it. Session log, tools pipeline, agent loop,
  LLM seam (vocabulary, `AdapterBase` retry policy, `FakeAdapter`), plus the M6
  long-lived-agent services: durable approvals (`ctx[:approvals]`, opt-in per tool), the
  compactor on the sole-provider `ctx[:summarizer]` seam, the titler, and hot-reloadable
  per-agent policy (`AllowList` as a log projection). M8 added the sole-provider
  `ctx[:subagents]` seam (a child agent on a fresh session, run to completion) and the
  loop's tool barrier, which runs a message's `concurrency: :parallel` calls together on
  the reactor while a `:serial` tool is a barrier of one. `Terret::Credentials`
  (`ctx[:credentials]`) lives here too: ENV `<PROVIDER>_API_KEY` first, then an optional
  AES-256-GCM file store, every resolved value fed to the scrubber.
- `gems/terret-openrouter` is the one real adapter (plan §6.5): OpenRouter's
  OpenAI-compatible API behind `ctx.llm`, streaming SSE with tool calling and usage
  accounting. The transport is injectable, so its unit tests need no network and no
  gems; only the default `AsyncTransport` requires `async-http`.
- `gems/terret-store-sqlite` is the durable session store (M3): the append-only log
  one event per row in SQLite (WAL) behind the `ctx[:session_store]` seam. Memory and
  JSONL providers live in terret-core; the store row is explicit in every boot.
- `gems/terret-ws` is the v1 interface (M4): one WebSocket per agent behind `ctx[:ws]`,
  the §9.2 frames, exact replay-then-tail on the session log; wire contract in
  `docs/protocol.md`; only the real endpoint requires `async-websocket`.
- `gems/terret-acp` is the second interface (M8): an Agent Client Protocol server behind
  `ctx[:acp]` so an editor can drive an agent over JSON-RPC on stdio. It consumes
  `session/event` and drives `ctx[:loop]` — the same two seams the socket does, on a
  different transport, with no change to core; mapping in `docs/acp.md`. stdlib-only.
- `gems/terret-mcp` is the MCP client (M5): manceps-backed stdio and streamable-HTTP
  servers mounted as `mcp__<server>__<tool>` sources behind `ctx[:tools]`, per-server
  approval, per-call timeouts, the allow list in terret-core; mapping in `docs/mcp.md`.
- `gems/terret-morph` is a `ctx[:summarizer]` provider (M6): Morph's Compact API on the
  wire proven in the deployed agora integration (bearer key, `compression_ratio: 0.4`,
  nil-on-any-failure), an injectable transport so its unit tests need no network, and a
  `MORPH_LIVE=1` live lane (pending a `MORPH_API_KEY` in this environment).
- `gems/terret-exec` is the execution world (M7, plan §6.6): `ctx[:fs]` with every path
  realpath-contained to a granted workspace dir behind an `fs/authorize` waterfall,
  `ctx[:subprocess]` (spawn and PTY under the one reactor, SIGTERM→SIGKILL cancellation),
  `ctx[:shell]` (one persistent bash per agent, sentinel protocol), `ctx[:terminals]`
  (named long-lived PTYs, capped), and `ctx[:sandbox]` with the `None` provider — every
  argv passes `sandbox.wrap` before it spawns. Stdlib only.
- `gems/terret-tools-std` is the standard roster (M7, plan §6.7): `Read`, `Write`, `Edit`,
  `Glob`, `Grep`, `Bash`, `WebFetch`, `terminal_open`/`input`/`read`/`close` — Claude
  Code's names verbatim, no alias map — registered on those seams with honest
  `mutating`/`approval`/`concurrency` metadata. `Bash`'s approval derives from
  `sandbox.isolated?` at registration; `WebFetch` sits behind a deny-by-default domain
  policy re-checked per redirect hop, plus a host-side loopback/link-local floor. M8 added
  `Task` (delegate to a child agent via `ctx[:subagents]`), `TodoWrite`, and
  `job_start`/`job_collect`/`job_stop` over `ctx[:jobs]`.
- `gems/terret-sandbox-docker` is the container provider (M7): a long-lived container per
  boot, each workspace dir bind-mounted at the same absolute path, argv wrapped into
  `docker exec`, `--network none` by default. One patch row moves the execution world into
  it; docker-gated tests skip clean when the daemon is absent.
- `gems/terret` is the meta-gem and the composition layer (M8, plan §7): bundles ship
  ordered config rows, profiles stack bundles, patches adjust rows by id, and
  `Terret.boot` hands the result to the Hames loader. It ships `terret-base`
  (`config/bundle.yml`), the `headless` profile template, and the `trt` executable
  (`boot`, `dump-config`, `doctor`, `acp`). Its `discover_bundles` mounts a third-party
  bundle off that gem's gemspec `metadata["terret"]`, so a plugin gem becomes composable
  by shipping normally. The contract is `docs/composition.md`; read it and plan §7 before
  changing anything here. `Terret::Meta::VERSION` is the gem's version —
  `Terret::VERSION` belongs to terret-core.

The full roadmap is `docs/terret-implementation-plan.md`; phases are in its §12. What is
here covers M0–M8, the whole harness: kernel, session log with the invariant, tools

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [terret-org/terret](https://github.com/terret-org/terret) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
