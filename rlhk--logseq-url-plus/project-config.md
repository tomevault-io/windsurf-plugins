---
trigger: always_on
description: URL+ is a **Logseq plugin written in ClojureScript**, built with shadow-cljs,
---

# Repository Guidelines

URL+ is a **Logseq plugin written in ClojureScript**, built with shadow-cljs,
UI in Rum, styled with Tailwind + daisyUI, tasks run by Babashka.
There is no TypeScript and no JS bundler config. `package.json` exists only to
declare npm deps and the Logseq plugin manifest.

## Prerequisites

Run `./script/bootstrap.sh` first on an unknown machine. It is plain `sh` and
needs nothing, whereas `bb doctor` cannot tell you that `bb` itself is missing.
Thereafter `bb doctor` is the authoritative check.

- **A JDK 21 or newer must be installed**, because shadow-cljs bundles a
  Closure Compiler built for class-file version 65. You do not need to export
  `JAVA_HOME`: the `bb` tasks resolve a suitable JDK themselves and will look
  past an older system default. `bb java` shows which one they picked.
- Node, Yarn, Babashka, clj-kondo. CI pins Node 22 and Java 21.
- `yarn install` before any build, `node_modules/` is not committed.

## Build, Test, and Development Commands

Everything runs through `bb`; `bb tasks` lists them all. The tasks are written
to be driven by a coding agent as well as by a person, so they are
non-interactive, safe to re-run, and report status through exit codes.

| Task | Purpose | Exit code |
| --- | --- | --- |
| `bb doctor` | Check the toolchain; prints what is missing and how to fix it | 1 if anything missing |
| `bb java` | Which JDK the tasks resolved | 0 |
| `bb ci` | Everything CI runs: lint, check-css, test, build | 1 on any failure |
| `bb lint` | clj-kondo over `src` | 1 on any warning |
| `bb check-css` | Every daisyUI class used by the UI still exists | 1 if any are gone |
| `bb test` | Unit + integration suites | 1 on failure |
| `bb build` | Release bundle into `dist/` (`:advanced`) | 1 on failure |
| `bb dev` | Watch CLJS + Tailwind, re-running tests on save, **blocks** | 1 if already running |
| `bb dev agent` | Same watch, **detached**, waits until ready | 0 when ready |
| `bb dev-logs` | Show the background watch log | 0 |
| `bb repl-status` | Is the `:plugin` CLJS runtime attached? | **0 = ready**, 1 = not |
| `bb stop` | Stop the shadow-cljs server (idempotent) | 0 |
| `bb restart` | `stop` then `dev` |, |
| `bb sideload` | Create/refresh the dev plugin copy and register it with a running Logseq | 1 if `dist/` is missing |
| `bb reload` | Reload the side-loaded plugin; prints the slash-command count | 1 if Logseq is unreachable |
| `bb deps` | Check for Node + Clojure dependency updates | **1 = updates available** |
| `bb release` | Runs `bb ci`, then tags and pushes | 1 if CI fails |

**Start the watch with `bb dev agent`.** Plain `bb dev` is the same watch in
the foreground: it never returns, so it will hang your tool call. Adding
`agent` runs it detached, waits until both builds report ready, prints a status
block and exits 0, typically in under ten seconds. Then poll `bb repl-status`.
`bb restart agent` takes the same mode. A person at a terminal wants bare
`bb dev`, where blocking is the point.

You do not need to set `JAVA_HOME`; see Prerequisites.

## REPL-Driven Development

The plugin's CLJS runtime lives **inside a Logseq iframe**. It does not exist
until Logseq is running with the plugin loaded. Keep three states distinct:

1. **server/watch alive**, a shadow-cljs worker exists for the build
2. **build ready**, that worker compiled successfully
3. **runtime attached**, a live JS runtime is connected

```bash
bb repl-status   # runtimes=0 means do NOT attach or evaluate yet
```

If you evaluate CLJS while `runtimes=0`, you are talking to the **JVM Clojure**
REPL instead, and the results will be confusing. This is the single most common
cause of "the REPL is behaving strangely" in this repo.

Workflow:

1. `bb dev agent` (bare `bb dev` is the same watch in the foreground and will block forever)
2. Launch Logseq with `--remote-debugging-port=9223`, then `bb sideload`.
   It registers under a *different* plugin id, so the Marketplace copy can stay
   installed. See `docs/dev-notes.md` for the launch command, an integrated
   terminal breaks it via `ELECTRON_RUN_AS_NODE`.
3. `bb repl-status` → confirm `:plugin runtimes=1`
4. Connect Calva to the shadow-cljs nREPL on port **8702** (`:init-ns core`)
5. Rich `(comment ...)` blocks at the end of `ui.cljs` and `ls.cljs` hold
   scratch expressions for reload and state inspection

Hot reload does **not** reach the plugin, despite `:after-load` being wired.
Run `bb reload` after every change; it reports the slash-command count, and
anything other than 11 means registration is broken.

## Coding Style & Conventions

- Run `bb lint` after every save and fix all warnings before proceeding.
- **Never hand-balance parens.** Use structural editing (Calva/Paredit) or a
  structural tool; do not count brackets manually.
- Every namespace gets a docstring naming its single responsibility.
- Keep pure logic in `util.cljs` / `api.cljs` / `feat/` where it is unit-testable;
  `core.cljs` orchestrates, `ls.cljs` is the only Logseq interop layer, and
  `entry.cljs` is the only namespace that imports `@logseq/libs`.
- `feat/define.cljs` is the model to follow: small, pure, well covered.

## Testing

Two tiers, both run by `bb test`:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rlhk/logseq-url-plus](https://github.com/rlhk/logseq-url-plus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-27 -->
