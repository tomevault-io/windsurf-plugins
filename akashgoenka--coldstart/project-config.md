---
trigger: always_on
description: Lightweight navigation layer for AI agents. Answers one question: which files are relevant to this task? Fast static index over file paths, symbol names, and exports — queried via two operations, `find` (ranked file paths matched against declared names: filenames, path segments, exported symbols, plus a repo-wide grep-recall pass) and `gs` (for a single file: top-level symbols with per-symbol cross-file callers, 1-hop internal imports, and reverse importers; `view: "full"` is the default and ret
---

# coldstart-mcp

Lightweight navigation layer for AI agents. Answers one question: which files are relevant to this task? Fast static index over file paths, symbol names, and exports — queried via two operations, `find` (ranked file paths matched against declared names: filenames, path segments, exported symbols, plus a repo-wide grep-recall pass) and `gs` (for a single file: top-level symbols with per-symbol cross-file callers, 1-hop internal imports, and reverse importers; `view: "full"` is the default and returns all of it in one call). Both are exposed two ways with identical output: as **CLI commands** (`coldstart find` / `coldstart gs`) for shell-capable agents — the primary path — and as **MCP tools** (`find` / `gs`) for no-shell clients like Claude Desktop.

**Architecture — one keeper, thin readers.** The query surfaces are stateless readers over an on-disk cache; a single background **keeper** process keeps that cache fresh. It does NOT serve queries.
- **Keeper** (`coldstart --daemon`, `src/index.ts` runKeeper + `src/index-manager.ts`): watches the repo, patches/rebuilds the index, debounce-saves it to the disk cache. Lazy-spawned by the first reader (`src/keeper.ts` ensureKeeper); logs to `~/.coldstart/daemon/<root>.log`; exits when its lockfile is removed, a foreign pid takes over the lock, or **its own entry binary disappears** (`watchOwnBinary` in `src/daemon-lock.ts` — a 15 s poll, two consecutive misses, so a keeper self-reaps within ~30 s of an `npm uninstall`/moved install, the only shutdown path once the CLI is gone). The lock carries pid/rootDir/version (no port — it serves nothing).
- **CLI readers** (`coldstart find`/`gs`, `src/cli.ts`): load the cache under a load profile, run the same engine, print, exit. They `ensureKeeper` so uncommitted edits stay live. **Readers NEVER build**: cache miss → wait for the keeper's first save (180 s); git-HEAD drift → wait for its reconcile re-save (12 s); in-process build only on the no-keeper fallback.
- **MCP reader** (default invocation, `src/server/mcp.ts` + `src/index.ts` makeCacheReader): a long-lived stdio server that reads the keeper's cache and mtime-reloads when it changes. The keeper is the single freshness authority; the server has no watcher of its own.

**Freshness (no TTL — deleted in v18).** Validity = CACHE_VERSION + git HEAD + reconcile + live watcher. At keeper start, `src/indexer/reconcile.ts` stat-walks every indexed file against its `[mtimeMs,size]` fingerprint (stamped in baseIndexedFile) + git-diffs against the indexed HEAD, then patches exactly the changed set (branch switch = ~3 s patch, was a full rebuild). After every patch, `src/indexer/invariants.ts` lints the index (violation → auto-rebuild); after every save, a rotating 50-file fingerprint audit re-patches watcher-missed drift. `keeper-state.json` (last reconcile/patch/rebuild/save) + `repair.jsonl` (failures) sit beside the cache; `status` renders both. The keeper watches its own lockfile with pid-ownership + a 30 s poll backstop (foreign pid → exit).

**`find` speed:** the repo-wide reference scan resolves a real ripgrep (`src/server/searcher.ts`: COLDSTART_RG → PATH → bundled @vscode/ripgrep → editor-app copies; winner persisted in `~/.coldstart/searcher.json`), falling back to git grep → grep → Node scan.

For Rails repos: the Ruby parser emits synthetic edges for `has_many`/`belongs_to`/`has_one`/`has_and_belongs_to_many` (gated to `app/models/`), parses `config/routes.rb` resources, and adds bidirectional controller↔views edges. Polymorphic associations and gem-backed models stay unresolved (runtime DSL).

**Index pipeline:** walk → parse (Tree-sitter via **web-tree-sitter (WASM)** — the native `node-tree-sitter` engine was dropped in #68; all 15 grammars are inert `.wasm` vendored in `vendor/wasm/`, loaded once by `ensureParsersReady()` in `src/indexer/parser.ts`, no per-grammar peer-dep / node-gyp / install scripts → plain `npm i` for TS/JS/JSX/TSX/Java/Ruby/Python/Go/Rust/C#/PHP/Kotlin/C++/YAML/TOML/XML/Groovy; SFC script blocks extracted from Vue/Svelte/Astro before TS parsing; GraphQL/.env/AngularJS 1.x via regex extractors; Swift/Dart not indexed) → resolve imports → build graph (including cross-file call edge resolution) → save cache. Behaviour is byte-identical to the old native engine (proven with natives pruned) and 1.3–3.6× faster through the walk-heavy extract phase.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AkashGoenka/coldstart](https://github.com/AkashGoenka/coldstart) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
