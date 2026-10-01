---
trigger: always_on
description: This file is the repository-wide operating contract for AI coding agents and
---

# Pogo Engineering Guidelines

This file is the repository-wide operating contract for AI coding agents and
human contributors. It applies to every change unless a more specific
`AGENTS.md` exists below the directory being changed. Read the relevant source,
tests, `README.md`, and `DEV.md` before modifying behavior. Prefer the smallest
correct change that preserves Pogo's latency, memory, compatibility, security,
and failure-isolation properties.

Use `MAINTENANCE.md` for operational CI triage, release publication and
verification, benchmark evidence, rendered documentation, GitHub transport, and
handoff procedures. `AGENTS.md` remains authoritative when the playbook or an
older plan conflicts with this contract.

## Project Mission

Pogo is a Django ORM language server. It provides runtime-accurate completion,
hover, signature help, diagnostics, and definitions to LSP 3.16 clients while
keeping editor interactions independent of Python startup and project import
latency.

The system has two deliberately separated execution domains:

1. The Go coordinator owns LSP transport, incremental Tree-Sitter parsing,
   conservative Python/Django inference, immutable schema indexes, and every
   editor-facing hot-path lookup.
2. The embedded Python worker starts the target project's installed Django,
   inspects the initialized app registry in the background, and serializes a
   bounded schema snapshot over authenticated local IPC.

Runtime Django metadata is the source of schema truth. The validated Go graph is
the only source used while serving editor feature requests.

## Core Non-Negotiables

### Hot-Path Isolation

- Python MUST NEVER run synchronously or asynchronously as a consequence of a
  completion, hover, signature-help, definition, diagnostic, or
  keystroke/document-change lookup.
- Hot-path handlers MUST read only current Go document state and the currently
  published in-memory schema generation. They must continue to work when the
  worker has exited or is unavailable.
- Do not add worker RPC, process startup, filesystem scans, Django imports,
  network access, subprocess execution, or unbounded waits to editor request
  handlers.
- Schema extraction is permitted only during initialization and debounced
  background refreshes caused by schema-affecting saves. Publication occurs only
  after complete payload validation and graph construction.
- Preserve tests proving cached handlers make no worker requests, especially
  `TestCachedFeatureHandlersDoNotRequestWorker`-style coverage in
  `internal/lsp`.

### Zero External Python Dependencies

- Production code in `src/daemon` may import only Python's standard library and
  Django from the interpreter selected for the target project.
- Never install or require Pogo-specific PyPI packages in a user's project or
  selected interpreter. Do not introduce `requests`, `pydantic`, `msgpack`,
  `typing_extensions`, or similar convenience dependencies into the worker.
- The fixture environment may install Django for tests. That does not permit a
  production worker dependency beyond Django.
- Keep the worker self-contained and embeddable through `src/daemon/embed.go`.
  A release binary must not depend on repository source files or test fixtures
  at runtime.
- Run `make release-check` after changing worker imports or embedding. Its AST
  inspection is an architectural gate, not an optional lint.

### Fault Tolerance And Last-Valid State

- Syntax errors, import failures, missing settings, invalid Django projects,
  malformed or oversized frames, worker crashes, timeouts, and invalid schemas
  MUST NOT crash or wedge the Go LSP coordinator.
- Never replace a valid graph with `nil`, a partially built graph, an invalid
  snapshot, or a result from a superseded refresh. Failed refreshes retain the
  last valid generation.
- Build and validate a candidate graph completely before publication. Publish a
  graph and generation together in one atomic state replacement.
- User-visible failures must be concise and actionable. Detailed worker and
  project output belongs in the language-server log, never LSP stdout.
- Worker restarts must remain bounded, cancellable, backoff-controlled, and
  leak-free. Shutdown must reap child processes and remove runtime directories,
  sockets, and token files.

### Microsecond Performance Mindset

- Treat document updates, AST extraction, model inference, graph traversal, and
  editor handlers as allocation-sensitive code.
- The engineering target for ordinary warmed editor interactions is p95 below
  `100 us`; use the tracked profiles in `DEV.md` and generated
  `benchmark-results/profile.json` as the current baseline. The automated
  release gate is intentionally broader (`completion p95 < 10 ms`, Go RSS
  `<= 50 MiB`, combined Go and worker RSS `<= 150 MiB`) and is not permission to
  consume the margin.
- The reference Go coordinator footprint is approximately `19 MiB` RSS. Avoid
  persistent per-document, per-model, or per-field duplication without measured
  justification.
- Minimize conversions among `string`, `[]byte`, and UTF-16 positions; avoid
  regexes, reflection, interface boxing, temporary maps, and sorting in hot
  loops unless benchmarks prove the cost acceptable.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amirhasanzadehpy/Pogo](https://github.com/amirhasanzadehpy/Pogo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
