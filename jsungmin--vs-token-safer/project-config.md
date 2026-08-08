---
trigger: always_on
description: Force code search through an **official language server's index** (clangd for C++, a Roslyn-based C# LSP)
---

# vs-token-safer — Claude rules

Force code search through an **official language server's index** (clangd for C++, a Roslyn-based C# LSP)
instead of Bash grep, and **token-cap** the result to a compact `file:line` list. The
Visual-Studio / IDE-agnostic sibling of `rider-mcp-enforcer`. Local-only. Ships as MCP server + CLI
(`vts`). npm package + plugin name: `vs-token-safer`.

## First, orient (every session)
1. Read this file, then `node eval/run.mjs` — must print `EVAL PASSED` (41/41) before you change anything.
2. Resume context lives in: this file · the wiki (`wiki_query "vs-token-safer"`, pages under
   `.omc/wiki/`) · memory anchor `project-vs-token-safer`. The wiki **Status and TODO** page is the
   live checklist.

## What's true
- **Engine = official, glue = ours.** clangd (LLVM) / Roslyn (MS) do the analysis; we only write the
  LSP↔MCP glue. Never reuse a 3rd-party MCP server over source; never reimplement Roslyn.
- **Local-only, zero transmission.** Same trust model as the other plugins. The token-cap returns
  `file:line` (no bodies) → less raw source reaches the model than grep-and-paste.
- **Async.** `runTool` is async (LSP is async). MCP/CLI adapters must `await` and `disposeClients()`.
- **Naming umbrella.** "token-safer" is deliberately broad — more token-saving features/backends can be
  added under this name beyond C++/C# search.

## Layout
- `server/lsp.js` — generic LSP client (JSON-RPC/stdio). The one new, careful piece. `didOpen` is
  open-or-refresh: first call → `didOpen(v1)`, a re-call on an already-open doc → `didChange` (bumped
  version, current disk text) so a file changed after warm-up isn't answered from a stale buffer; a
  since-deleted file → `didClose`. Position tools re-call `didOpen` before each query, so hover/goto/
  outline/rename always re-read the file. The LSP engine keeps UNOPENED files fresh itself (clangd
  file-watch + background re-index); our warmset caches self-invalidate (include-graph by mtime+size composite
  key + an FNV-1a content hash [`warmset.js fnv1a`, zero-dep — codebase-memory-mcp XXH3 parity; reuses cached
  includes when bytes are unchanged despite mtime/size jitter, catches a real change a mtime-only key would miss],
  query-history by re-record; `_censusCache` is process-lifetime → restart/re-setup to refresh).
  LSP-spec conformance: server→client requests get shape-correct replies (`_serverRequestReply`:
  `workspace/configuration`→array, `workspace/applyEdit`→`{applied:false}`, `window/showDocument`→
  `{success:false}`, void reqs→null, unknown→MethodNotFound -32601); a timed-out request sends
  `$/cancelRequest`; client declares `synchronization` + `workspace.configuration` capabilities.
- `server/scope.js` — INDEXING SCOPE (cold-latency attack): index a SUBTREE not the whole monorepo. Config
  `scope` / `VTS_SCOPE` (comma-list of dirs rel to root); `vts setup --scope "MyGame,Plugins"` persists it.
  `scopeDirs`/`inScope`/`scopedCdb` (writes a FILTERED compile_commands.json of only in-scope TUs to the
  out-of-tree dir → clangd `--compile-commands-dir` points there → it background-indexes far fewer TUs;
  live UE5: `VTS_SCOPE=MyGame` = 3,377 of 26,488 TUs (13%), ~7.8× cut) / `scopeStats`. UNIVERSAL: every
  backend's afterInit warm walk is scope-filtered too (no tsconfig/sln edit). `backends/index.js`
  `effectiveCdbDir(root)` = scoped CDB when scope set, else `resolveCdbDir`; `scopeDirsFor(root)`. clangd
  STATIC PREINDEX: `clangd-indexer` (full LLVM release bundles it; `VTS_CLANGD_INDEXER_CMD`/next-to-clangd/
  PATH) builds a monolithic .idx over the scoped CDB → clangd loads it via `--index-file` (LOCAL file, no
  remote server) for an instant project-wide index; `buildStaticIndex`/`hasClangdIndexer`/`staticIndexPath`,
  absent → warm-pass fallback + advisory to install full LLVM. Ops `vts_scope` (show scope + TU stats +
  top-level dirs) / `vts_preindex` (build ahead: static index if indexer present, else warm pass); CLI `vts
  scope`/`vts preindex`, folded into `vts_admin`. Eval guard 79. Env: `VTS_SCOPE`, `VTS_CLANGD_INDEXER_CMD`,
  `VTS_INDEXER_TIMEOUT_MS` (1800000). PREINDEX GATING: `vts preindex` DEFAULT = fast scoped background warm;
  the clangd-indexer STATIC `--index-file` (parses every in-scope TU, tens of min on a big scope) is OPT-IN via
  `static=true`/`--static` — never auto-run just because the indexer exists (an existing `vts-static.idx` is
  still auto-loaded, cheap). within-scope cert: `completenessCert({scoped})` qualifies a semantic COMPLETE/0 as
  "within the configured indexing scope" (search_symbol/find_references), and `clangdIndexAdvisory` counts
  TUs/shards against the EFFECTIVE (scoped) CDB.
- `server/policy.js` — UNIFIED TOOL-ROUTING POLICY (vts COMPLEMENTS Claude Code, not competes). `shouldSuppressSteer(file)`
  stays SILENT where CC-native is clearly better — generated/build-output paths (`Intermediate|Binaries|Saved|
  DerivedDataCache|node_modules|build|dist|out|obj|.git`, `*.generated.*`/`*.g.cs`/`*.min.js`); wired into the
  edit-steer hook (a whole-decl edit there isn't nagged AND isn't counted against adoption). `VTS_SUPPRESS=0` off.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JSungMin/vs-token-safer](https://github.com/JSungMin/vs-token-safer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
