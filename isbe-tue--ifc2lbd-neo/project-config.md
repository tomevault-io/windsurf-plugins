---
trigger: always_on
description: This file governs how AI agents (Claude Code, Codex, Copilot Workspace, etc.) must behave when
---

# AGENTS.md — Architectural rules for AI agents

This file governs how AI agents (Claude Code, Codex, Copilot Workspace, etc.) must behave when
modifying IFC2LBD-Neo. All rules below are **hard constraints**, not suggestions.

---

## 1. Understand the pipeline before touching it

The conversion pipeline has a fixed stage order:

```
Import → Preprocess → Produce → Postprocess → Serialize → Export
```

- **Import**: parse the IFC file into `IfcModel` / `StepFile`, insert into `PipelineContext`.
- **Preprocess**: enrich / validate the model; run sequentially via `spawn_preprocessors()`.
- **Produce**: emit RDF `TaggedBatch`es into a bounded channel; run in parallel via `spawn_producers()`.
- **Postprocess**: inspect / mutate the full batch set; run sequentially via `spawn_postprocessors()`.
- **Serialize**: convert batches to bytes (Turtle / N-Quads); bespoke in each runner.
- **Export**: write bytes to disk / blob / memory; session-based via `ExportSession`.

Never add a new stage without updating `PipelineStage` in `lbd-pipeline/src/lib.rs` AND updating
both runners (`ifc2lbd-cli/src/main.rs` and `ifc2lbd-wasm/src/runner.rs`).

---

## 2. Plugin traits

| Stage | Trait | Dispatch function |
|-------|-------|------------------|
| Preprocess | `PreprocessPlugin` | `spawn_preprocessors()` |
| Produce | `ProducerPlugin` | `spawn_producers()` |
| Postprocess | `PostprocessPlugin` | `spawn_postprocessors()` |
| Serialize | `SerializerPlugin` | **none** — marker trait only |
| Export | `ExportPlugin` | `ExportPlugin::start_session()` |

### SerializerPlugin is a marker trait

`SerializerPlugin` has **no dispatch method**. Never add `fn serialize()` or any similar method
to the trait. Serialisation logic lives bespoke in `main.rs` / `runner.rs`. The trait exists only
for registration and `conflicts_with` resolution.

### ExportPlugin uses the session pattern

`ExportPlugin::start_session(ctx)` returns `Box<dyn ExportSession>`. All output goes through the
session's `open_sink()` / `accept_derived_file()` / `finalize()`. Never bypass the session.

**CLI**: `main.rs` resolves the active export plugin via
`registry.resolve_active_export(&plan.enabled_ids)`, opens a session, wraps it in
`Arc<Mutex<Option<Box<dyn ExportSession>>>>`, and every file-writing thread (LBD serializer,
IfcOWL sidecar, `QuadChunkWriter` for chunked N-Quads, manifest writer) requests its writer
through `session::open_sink()`. After all writer threads join, `session::finalize()` returns the
per-file summaries which are logged.

**WASM**: the streaming runner (`run_to_sink`) emits bytes via a `js_sys::Function` JS callback
which is `!Send`. The `ExportSession` trait requires `Send` for CLI-side mutex sharing, so the
streaming path cannot use the trait directly — `SinkChunkWriter` writes straight to the JS
callback. The runner still honors the activation plan: it checks `settings.has(LOG_EXPORT_ID)`
before emitting per-module log sidecars, so plugin-driven dispatch decisions are respected even
though the delivery mechanism (JS callback) is platform-fixed. The `run_memory` in-memory path
uses `WasmFileExportSession` / `WasmLogExportSession` directly because buffered byte vectors
are `Send`.

---

## 3. `PipelineContext` rules

```rust
ctx.insert(Arc::new(value));     // add a NEW type — panics if T already present
ctx.replace(Arc::new(value));    // update an EXISTING type — safe to call any time
ctx.get::<T>()                   // read a typed value — returns Option<Arc<T>>
```

- **Never call `insert` for a type that is already in context.** Use `replace`.
- **Never downcast `Arc<dyn Any>` manually.** Always use `ctx.get::<T>()`.
- **Sidecar channel** — `ctx.sidecar_tx: Option<Sender<DerivedFile>>`:
  - Always guard: `if let Some(tx) = &ctx.sidecar_tx { ... }`.
  - Ignore send errors: the receiver may have been dropped during shutdown.
  - `DerivedFile.mime_type` is `&'static str` — use a string literal.

---

## 4. Sidecar file lifecycle

Sidecar artefacts (geometry files, thumbnails, etc.) flow like this:

```
ProducerPlugin::produce()
  └─ ctx.sidecar_tx.send(DerivedFile { ... })
        │
        ▼ (orchestrator drains channel after all producers finish)
ExportSession::accept_derived_file(file)
  └─ write to disk / upload / buffer in memory
```

The orchestrator creates the channel before producers run and tears it down after. Producers must
not hold the `sidecar_tx` beyond the `produce()` call.

---

## 5. Registration rules

- Every plugin must be registered in **both** `ifc2lbd-cli/src/pipeline_plugins.rs` AND
  `ifc2lbd-wasm/src/plugins.rs`, UNLESS `wasm_compatible: false` in the manifest (in which case
  omit WASM registration).
- Use `registry.register_preprocess()`, `register_producer()`, `register_postprocess()`,
  `register_serializer()`, or `register_export()` — never push to internal vecs directly.
- Plugin IDs are kebab-case, globally unique, and **immutable once published**. A rename is a
  breaking change (stored in persisted state).

---

## 6. Template crates are the canonical starting point

When creating a new plugin, **always** copy the matching template:

| Plugin type | Template |
|-------------|----------|
| Preprocess | `crates/plugin-template-preprocess/` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ISBE-TUe/IFC2LBD-Neo](https://github.com/ISBE-TUe/IFC2LBD-Neo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
