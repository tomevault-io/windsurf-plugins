---
trigger: always_on
description: Build a Go language server for legacy Vim script and Vim9 script, with grammar
---

# vimls-go agent guide

## Scope and source of truth

Build a Go language server for legacy Vim script and Vim9 script, with grammar
and metadata pinned to **Vim v9.2.1015**. Later syntax is unsupported until the
pin advances. The current contracts are [language support](docs/language-support.md),
[diagnostics](docs/diagnostics.md), [configuration](docs/configuration.md) and
[roadmap](docs/roadmap.md). Read the parts relevant to the task.

Resolve version-sensitive behavior in this order:

1. Pinned Vim source and tests, especially `src/testdir/test_vim9_*.vim`,
   `test_vimscript.vim` and Ex command definitions.
2. Matching runtime help: `vim9.txt`, `vim9class.txt`, `eval.txt`, `usr_41.txt`,
   `repeat.txt` and the affected command/option's help.
3. LSP 3.18 and JSON-RPC 2.0 specifications for protocol behavior.
4. Curated, side-effect-free reproductions with a clean supported Vim executable.

The official checkout `/Users/chemzqm/lib/vim` is read-only for this project.
Inspect its status, preserve local changes and read the exact tag; its HEAD may
be newer. Record Vim version/patch level for oracle evidence. Never generalize
behavior observed outside the pin into an unconditional language rule.

## Language and analysis invariants

- Legacy and Vim9 have independent root parsers, sharing neutral syntax and Ex
  command data. Cross-dialect constructs recover loosely without errors merely
  for mixing dialects; exhaustive `def` in a legacy root and `function` in a
  Vim9 root remain deferred.
- Dialect is contextual: `vim9script`, `def`, `function`, `vim9cmd`, `legacy`
  and `scriptversion` affect parsing. `vim9cmd` and `legacy` apply only to the
  following command, not persistent parser mode.
- Parse ranges, modifiers, abbreviations, bang, separators, comments,
  continuations, heredocs and embedded payloads contextually. Preserve byte
  spans and trivia. A line split or regex-only parser is insufficient.
- Keep incomplete and unknown commands as opaque recoverable syntax. Unknown
  uppercase user commands may receive the documented E492 warning after source
  indexing and runtime help are ready; explicit command definitions and Ex-command
  help tags both establish known names. This is not a parser error.
- Keep dynamic legacy semantics conservative: use `unknown` when proof is
  unavailable. Preserve documented warning policies rather than silently
  converting uncertain runtime behavior into hard errors.
- Do not source or execute user Vim scripts during analysis. Optional clean Vim
  default-runtimepath discovery is allowed as documented; it loads no user
  configuration/plugins and fails silently.

## Package and runtime boundaries

| Package | Owns |
| --- | --- |
| `internal/jsonrpc` | Framing and JSON-RPC request lifecycle |
| `internal/text` | Immutable snapshots and line/position indexes |
| `internal/syntax` | Dialect-aware tokens, AST, parsing and recovery |
| `internal/analysis` | Scopes, symbols, references, types and diagnostics |
| `internal/workspace` | File discovery, imports and indexes |
| `internal/server` | Capability handlers composing these packages |

- Dependencies point from server toward smaller packages. Syntax/analysis must
  not depend on transport or process state. Avoid empty packages and speculative
  layers; use existing packages and types first.
- Use pinned `go.lsp.dev/protocol` wire types. Byte-to-client-position conversion
  belongs in server/text; do not add a parallel `internal/lsp` layer.
- Keep stdout for LSP frames and logs on stderr. Advertise only implemented
  capabilities; preserve required unknown fields and standard JSON-RPC errors.
- Results belong to immutable URI/version/content and consumed index snapshots.
  Validate identity at installation/publication; stale work must not overwrite
  newer results. Test cancellation, shutdown and refresh ordering with barriers.
- Keep workspace and external runtimepath indexing separate. Retain unchanged
  runtime facts during rebuilds; exclude runtime-only symbols at query time.
  Refresh only supported client capabilities whose consumed data changed.
- Use Go for production, generators and repository tools. Vimscript is permitted
  for curated fixtures/oracles; shell for small CI/test orchestration. Pin
  dependencies and add them only for a concrete current benefit, never `@latest`.

## Go development and validation

- For Go symbol/reference/caller discovery, start with available gopls MCP tools
  before broad text searches. Prefer semantic rename. Request diagnostics after
  Go edits; use compiler/tests as final evidence. If MCP has stale overlays,
  use `gopls check <files>` and focused `rg` rather than fixing nonexistent errors.
- Add focused tests beside the owning package; shared fixtures go in `testdata/`.
  Accepted syntax needs a positive fixture; diagnostics/recovery need negative
  or incomplete fixtures. Test shared context mechanisms without duplicating
  every form across dialect/version combinations.
- For text changes, cover ordered edits, bytes/runes/UTF-16, CRLF/BOM, combining
  characters and astral characters. Parser/framing fuzz targets must not panic,
  hang or grow memory without bound; retain discovered crashes in the corpus.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [neoclide/vimls-go](https://github.com/neoclide/vimls-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
