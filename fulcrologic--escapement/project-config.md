---
trigger: always_on
description: Guidance for Claude Code when working in the **Escapement** repository.
---

# CLAUDE.md

Guidance for Claude Code when working in the **Escapement** repository.

## Working context

**Read `workingcontext.md` first** (it is gitignored — local, per-session,
never committed). It holds the current goal, known issues, gotchas, and the
focused list of files we're working on this session. If it does not exist,
create it with these sections:

- **Goal** — one paragraph: what we're trying to accomplish this session.
- **Known Issues** — bugs/gaps in the area being worked on, grouped by
  domain. Each bullet: terse fact + `file:line`. When resolved, strike
  through with `~~…~~` and append **FIXED <date>** (keep the original line).
- **Gotchas** — non-obvious things a future session will trip over (silent
  no-ops, hot-reload caveats, sentinel values, wire-format quirks). One
  line per fact, `file:line` when relevant.
- **Relevant Files** — the focused subset this session is touching, grouped
  by domain. Trim aggressively; this is not a codebase index.

You may autonomously append to `workingcontext.md` whenever you discover
anything that will save us time later — surprises, gotchas, non-obvious
wiring, awkward patterns, undocumented conventions, or pointers to files
we'd otherwise have to re-search for. Keep entries terse (one line per
fact, with a file path + line number when relevant). Do not duplicate
what's already there; do not turn it into prose. Only add facts that a
future session would thank you for.

## Project shape

Escapement is a statechart-driven autonomous coding agent. **Everything runs
under Babashka** — the agent process, the CLI, and the test suite (`bb test`).
No JVM is required.

- Runtime: bb (SCI). Source under `src/` and `demos/` must stay bb-compatible.
- Tests: bb via `bb test`, using `fulcrologic/fulcro-spec 3.2.9` (which added
  bb support). Test runner: `bb_test/escapement/test/runner.clj`. Discovers
  `test/**/*_test.clj` automatically.
- Sanity smoke: `bb sanity` (runs `bb_test/sanity.clj`).
- An optional `:test` alias exists in `deps.edn` for IDE/JVM REPL workflows,
  but the project does **not** depend on it.

## Layering: engine core vs. presentation add-ons

Escapement is a runtime/library **core** plus optional presentation add-ons.
The dependency direction is strictly one-way (`add-on → core`, never the
reverse), and that boundary is **enforced by a test**:

- **Engine/library core** — `engine.*`, `runner`, `lib`, `protocols`,
  `invocation.*` (incl. the `HumanRenderer` protocol + dependency-free
  `StdinRenderer`, so charts are interactive headlessly), `llm.*`, `tools.*`,
  `storage.*`, `transcript`, `debug.{controller,control-handle,d2}`, `config`,
  and the CLI (`cli.clj`). The embeddable entry point is `escapement.lib`. Core
  must NOT statically require the web/Pathom/RAD UI or the terminal UI.
- **Web/API add-on** — `escapement.ui.*` + Pathom/EQL/transit + the RAD/CLJS
  bundle. Loaded **lazily** by `--api-server` via `requiring-resolve`; if its
  deps are absent the flag fails with a clear message.

  The **released *library*** is kept clean of the Pathom/EQL/transit tree: the
  Clojars artifact's deps come from the hand-maintained **`pom.xml`**, which omits
  them, so a library consumer is never infected. The **CLI *tool*** however DOES
  need them, and both tool classpaths carry them: `bb.edn`'s `:deps` (the bb
  runtime tool) and — crucially — **`deps.edn` base `:deps`** (the `bbin`-installed
  tool). The latter is required because a `bbin install` builds its classpath ONLY
  from base `deps.edn` via a generated `bb --config <local/root>` launcher that
  ignores `bb.edn` and cannot activate a deps alias; parking these in the old
  `:api` alias left the CLI unable to serve its own `--api-server` web UI. The
  `:api` alias is now an empty no-op retained so existing `-M:api:...` composes
  still resolve. The only namespace that requires Pathom is
  `escapement.ui.resolvers` (via `server.clj`); it is server-side only and is NOT
  in the browser build graph, and no core namespace *statically* requires it
  (the `--api-server`/`--tui=opentui` bridges stay `requiring-resolve`), so the
  architecture-boundary test — which scans requires, not deps — stays green.
- **Terminal-UI add-ons** — there are two, both used by the CLI front-end only; the
  embeddable library (`escapement.lib`) never pulls either:
  1. **JLine TUI** (default) — `escapement.tui` (+ JLine), in-process.
  2. **OpenTUI sidecar** (opt-in via `--tui=opentui`) — an out-of-process Bun + SolidJS UI under
     `tui/opentui/` (outside `src/`, so the boundary scanner never sees it). Its agent-side glue lives in
     the `escapement.ui.*` add-on (`ws_push`, `remote_renderer`, `opentui_sidecar`, the
     `escapement.human/answer` mutation in `resolvers.cljc`) and is reached from `cli.clj` only via
     `requiring-resolve` (like `--api-server`). The agent runs headless and pushes its transcript
     event stream to the sidecar over a WebSocket on the api-server; the sidecar owns the TTY. `bb
     test` covers the agent side (incl. `test/escapement/ui/opentui_push_test.clj`); the TypeScript UI
     is tested under `bun test` (`bb opentui-test`) — a separate test path. See
     [`docs/opentui-ui.md`](docs/opentui-ui.md).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fulcrologic/escapement](https://github.com/fulcrologic/escapement) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
