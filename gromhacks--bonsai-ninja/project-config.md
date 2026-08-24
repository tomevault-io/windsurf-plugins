---
trigger: always_on
description: validates that every sidecar describes one current workspace snapshot and
---

# bonsai-ninja

Use `bonsai-ninja` when you need structural code intelligence: map a
repo, find symbols, trace behavior, debug dataflow, or run SAST.

Command truth comes from the binary:

```shell
./target/release/bonsai-ninja --help
./target/release/bonsai-ninja <command> --help
./target/release/bonsai-ninja security --help
```

Prefer `./target/release/bonsai-ninja`; use debug only if release is
missing. For scripts use `--format json --no-color --no-progress`; add
`--all` or `--context uncapped` only for intentional exhaustive
artifacts. For LLM-readable text use `--no-color --no-progress
--context 16k`.
Keep the workspace positional and prefer explicit selector flags
(`--query`, `--symbol`, `--file`, `--from`, `--to`, `--id`) in scripts and
agent calls. Positional selectors remain supported for interactive use, but
the CLI rejects supplying both forms. Output files accept `-o`, `--output`,
and `--output-path`.
Use `--html-output <file>` for a standalone themed human report; it wraps the
selected command's text view and must never enable additional analysis.
For save-time workflows, keep `index <workspace> --watch --no-progress`
running; command and SDK facades refresh saved file changes before they
render.
`index <workspace>` is the syntax/construct warm-up path: it parses source
and builds declaration/import indexes without forcing a whole-workspace
semantic prewarm. Use `index <workspace> --semantic` only when you
intentionally want structural semantic sidecars and
the external workspace-cache `manifest.json` built up front; commands still validate sidecar
headers/payloads before reuse and compute requested exact facts on demand.
Retrieval is candidate lookup only: search and literal-filtered browse can
reuse a fresh sidecar before candidate lookup, and large-workspace inspect can
use a warmed sidecar only before opening a scoped workspace. Rendered facts
still hydrate through canonical APIs, and scoped query workspaces do not
publish partial retrieval sidecars under the full workspace cache.

Analysis sidecars live in a canonical-path-keyed OS cache directory, not in
the inspected repository; `cache stats <workspace>` reports it and
`BONSAI_WORKSPACE_DIR` overrides it. The cache root carries a locked canonical
workspace binding so dependency-manifest freshness cannot be lost when a
sidecar path is outside the source tree. Workspace-local rule overlays remain
under `<workspace>/.bonsai/rules/` and are not analysis caches.

Treat the analyzer as a compiler pipeline. Each language adapter owns its
Tree-sitter grammar, source-syntax recognition, declaration/import lowering,
literal/value node inventories, and `FlowEvent`/capability facts. Shared
analysis consumes that typed IR; do
not add language-id branches, cross-language token inventories, or API-name
guesses to shared crates. Library/package/framework identities and every
security-sensitive value belong in `security-patterns/langs/<lang>`, not in
shared analysis or an adapter; adapters emit generic syntax/capability facts
and rule data assigns their security meaning. Pack-wide package spelling,
review profiles, test-path policy, dependency metadata, and taxonomy live in
`security-patterns/metadata.yml`. The production taint engine is the sparse IDG
fixed-point closure. It has no BFS name search, call-depth ceiling, iteration
limit, or result cap. Paging and diagnostic path limits affect rendering only
and must report truncation explicitly.

`index --semantic` first publishes an immutable content-addressed generation
of per-file compiler objects. Each object is exact adapter-lowered IR plus
diagnostics, validated by path, adapter, frontend ABI, and SHA-256 source
content. Import indexes, direct-call receiver-field initializer linkage, and
compact syntax-target facts (calls, assignment aliases, factory assignments,
inline callbacks, exact assignment/return/call-argument value shapes, typed callables, and
receiver/type evidence) are
integrity-checked compiler headers inside the same generation and must remain
independently decodable from declaration/flow bodies. Broad rule planning
filters raw source anchors, exact import/package headers, and exact syntax
targets in that order before decoding a surviving body. Later phases stream
those objects; they must not reparse source or invent a parallel lowering
path. Rulepack return typing retains its declared imports; exact workspace
values and ordinary functions shadow external `kind: new` models, and mixed
or ambiguous callable identities fail closed. Every derived semantic pipeline
identity includes the compiler-object frontend ABI; a lowering change
invalidates older callgraph/IDG sidecars even when source bytes are unchanged,
and root-only validators reconstruct the same identity as a full workspace
open. Persisted IDG construction lowers transfer facts once, spools typed
stitch records/node maps, and replays them per segment. Independent transfer
segments lower continuously on bounded dedicated workers under exact
source-size memory permits. Completed output retains its permit until a
bounded reorder map publishes canonical ascending `SegmentId` order to the
serial stitcher. Do not reintroduce per-batch barriers, and do not accept an

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gromhacks/bonsai-ninja](https://github.com/gromhacks/bonsai-ninja) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
