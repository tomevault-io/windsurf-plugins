---
trigger: always_on
description: validates (`rt/resolve-dyn`, `MAX-DYN-RANGE` 10k), CYCLE-CHECKS over static ∪
---

# SaltRim — working instructions

A simple-but-powerful spreadsheet: **Clojure** engine on **Spindel** (reactive),
**Datastar** UI (hypermedia, SSE), Datahike persistence, live collaboration.
Read `SPEC.md` for the technical architecture. This file = how to work here.

## MCP servers usage

- **Qdrant (preferred memory):** Use the Qdrant MCP server (`mcp-server-qdrant`,
  `qdrant-local`, etc...) for persistent vector memory — it is the **preferred**
  store (semantic recall across sessions). Explicitly use collection name
  `dev-saltrim` (project history: `dev-calcloj` → `dev-clorax` → `dev-saltrim`
  across renames; older collections kept as backups). If the MCP tools error
  (they have — empty errors, often a corrupt fastembed model cache or a stale
  server process), the Qdrant REST API on `localhost:6333` works directly:
  `POST /collections/dev-saltrim/points/query` (embed the query first) or
  `/points/scroll` for reads; embed with the MCP's own fastembed python env and
  `PUT /points` for writes — see the `qdrant-collection-saltrim` memory for the
  exact recipe.
- **Mirror memory both ways.** Qdrant can be unavailable (MCP broken, server
  down, or you're offline), so **also** write anything worth keeping to the
  file-based memory under `~/.claude/projects/.../memory/` — it loads every
  session regardless of Qdrant health. Qdrant is the preferred/searchable copy;
  the file store is the always-available mirror. Keep them in sync.
- **Clojure:** Always use `clojure-mcp` for interactive Clojure development. REPL
  is Clojure's superpower. If nREPL server isn't active, run it using command 
  `clojure -M:nrepl --port 7888` background command. Do not use Claude default 
  code execution and file creation capabilities with Clojure code. Use 
  `clojure-mcp` instead.

## Communication style

The user runs a "caveman" mode plugin — terse, fragments, drop filler. Match it
in chat. **Write code, commits, PRs, and docs normally** (full sentences).
If the user types `/caveman`, invoke the `caveman` Skill.

## How the user works (observed preferences)

- **Decisive, hands-on, opinionated.** They review closely and push back when an
  approach is wrong (e.g. "use another server?", "I don't like heartbeat",
  "collaboration is a need"). Take pushback seriously — they're usually right.
- **Verify before claiming.** They dislike hand-waving. Read real source, run
  spikes, test in the browser, show evidence. Don't assert behavior you haven't
  checked.
- **Prefers clean structure**: no imperative JS in HTML (Datastar attributes +
  a thin `app.cljs`, bridged by custom events — not hidden trigger buttons), no
  stray top-level forms, single source of truth, separate files.
- **Wants extensibility planned now** for near-future features (e.g. style/format
  as reactive properties; the persistence format already leaves room).
- They sometimes edit files between turns (addr.clj, gitignore, etc.). Respect
  those edits; don't revert them.

## Workflow

- **PR workflow**: commit/push/open PRs freely on feature branches — no need
  to ask. Never commit directly to `main`; the user reviews and merges PRs.
  End commit messages with:
  `Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>`
- One coherent change per commit; write a real body explaining *why*.
- **Spike risky unknowns first** — as REPL walkthroughs under `spikes/` (eval the
  forms at a dev REPL; see `spikes/README.md`), not cold-run mains. Don't build
  UI on unproven engine assumptions.
- **Quality gate — BOTH halves, every PR that touches engine or client code.**
  Green tests are half the bar; the other half is that the sheet did not get
  slower. Run them before opening the PR and put the numbers in it:
  1. `clojure -X:test` — must stay green. Add tests for new behavior. The count
     moves every PR, so don't pin it here — run the suite and read the tail.
     (~250 tests / ~1250 assertions as of the load-order PR.)
  2. `clojure -T:build cljs-test` after any `.cljs`/`.cljc` edit — the browser
     half is a separate compile AND a separate runtime, so a green JVM suite
     says nothing about it. Plus `node --check resources/public/app.js`, which
     is the only thing that exercises the `:advanced` bundle the suite doesn't.
  2b. **Never assert `=` on a double that came through `Math/pow`, `exp`, `log`
     or `erf`.** `java.lang.Math` is allowed 1 ulp of error and explicitly
     permits hardware-specific intrinsics, so a borrowed financial function
     answers differently on arm64 and x86_64 — CI caught `npv` and `pmt`
     differing in the 15th digit after the suite had been green locally for
     months. Use `approx/=` (`test/…/approx.clj`, which explains the rule and
     pins the real observed values). `Math/sqrt` and `+ - * /` ARE exact, so
     keep `=` there — loosening a test that cannot fail only hides the next bug.
     Production already knew this: `xlsx/demote-verify!` compares with
     `close-num?` and `roundtrip-test` with a 1e-9 tolerance.
  3. `clojure -M:bench` — compare against the recorded table in `doc/bench.md`.
     **A regression is a FAILING gate, not a footnote.** Find the cause and fix
     it, or state plainly in the PR what feature bought the time and why it is

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aldebogdanov/saltrim](https://github.com/aldebogdanov/saltrim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
