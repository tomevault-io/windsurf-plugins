---
trigger: always_on
description: Guidance for AI agents and contributors working in this repo. Read this before making non-trivial changes.
---

# AGENTS.md

Guidance for AI agents and contributors working in this repo. Read this before making non-trivial changes.

## What this is

storagesdk is a multi-provider SDK for object storage. One API across providers (S3, R2, MinIO, Azure Blob, Google Cloud Storage, Vercel Blob, Tigris, filesystem), with **snapshot** and **fork** as core operations alongside upload, download, delete, list, copy, move, and signed URLs.

The user-facing spec lives in [the top-level README](README.md), and the shipped code is the source of truth for the contract. Read both before proposing API or architecture changes.

## Locked design decisions

These are decided. Don't re-litigate without a clear reason.

- **Errors:** operations throw `StorageError`. No Result type. Codes: `NotFound | NotSupported | Conflict | Unauthorized | InvalidArgument | Provider`.
- **Verbs:** `upload`, `download`, `delete`, `head`, `list`, `copy`, `move`, `url`, `uploadUrl`.
- **Two item types.** `StorageItemMeta` (metadata only) is returned by `head` and as items inside `list`. `StorageItem` extends `StorageItemMeta` with `readonly body: Uint8Array` and is returned by `download`. No body accessors / closures.
- **Storage and Adapter are decoupled.** `Adapter` is the contract adapter authors implement (and `defineAdapter` accepts). `Storage` and `ReadOnlyStorage` are the consumer classes. They do NOT `implements Adapter` — they evolve independently. Notably, `Storage.download` / `ReadOnlyStorage.download` are overloaded (`as: 'stream' | 'text' | 'bytes' | 'blob' | 'json'`); `Adapter.download` is a single-signature method that returns `StorageItem`.
- **Interface hierarchy.** `ReadOnlyAdapter` has the four read methods (`download`, `head`, `list`, `url`). `Adapter` extends it with writes plus the `snapshots: AdapterSnapshots` and `forks: AdapterForks` namespaces. `AdapterSnapshots` and `AdapterForks` are exported interfaces so adapter authors can implement them in isolation. No combined `Snapshot` or `Fork` types — namespace methods return info or adapters separately.
- **`AdapterSnapshots` and `AdapterForks` are symmetric.** Both have `create`, `list`, `head`, `delete`, `get`. `head(id)` returns `SnapshotInfo` / `ForkInfo`. `create` returns `SnapshotInfo` / `ForkInfo` too. `AdapterSnapshots.get(id)` returns `ReadOnlyAdapter` (a reader). `AdapterForks.get(name)` returns `Adapter` (full storage).
- **Storage class** wraps `snapshots.get` and `forks.get` returns in `ReadOnlyStorage` and `Storage` respectively so consumers keep the download overloads on snapshot readers and forks. The `snapshots` and `forks` properties on `Storage` use inline types (not named interfaces) — they're consumer-facing shapes, not part of the adapter contract. No `storage.fork()` method — call `storage.forks.create(opts)`.
- **`ReadOnlyStorage` is exported as a type only.** Consumers receive `ReadOnlyStorage` instances from `storage.snapshots.get(id)`; there is no public constructor. `Storage` is the only constructible class.
- **`Raw` generic on `Adapter` and `Storage`.** `Adapter<Raw = unknown>` exposes a typed `raw: Raw` escape hatch. `Storage<Raw>`, `StorageOptions<Raw>`, `AdapterForks<Raw>`, and `defineAdapter<Raw>` all flow it through; `forks.get(name)` returns `Storage<Raw>` so the typed escape hatch survives fork navigation. Adapter authors who want it declare e.g. `Adapter<S3Client>` as the factory return type — `Raw` is otherwise inferred from the impl's `raw` field. Adapters that don't bother get the default `unknown` behavior unchanged.
- **Multipart auto-decide lives in the SDK, not the adapter.** `Storage.upload` resolves `opts.multipart` before calling the adapter: explicit `true`/`false` wins; otherwise size-known bodies multipart only above `opts.multipartThreshold` (default 5 MB), and `ReadableStream`s always multipart (size unknown upfront). Adapters that support multipart (e.g. S3) just check `opts.multipart === true`; adapters that don't (FS, in-memory) ignore it.
- **Snapshot identity is SDK-assigned (`id`); fork identity is user-provided (`name`).** `snapshots.create(opts)` returns a system-generated `id`. `forks.create(opts)` accepts a `name` which is the fork's identifier; you call `forks.get(name)` to address it.
- **Paths are normalized inside `defineAdapter`.** Leading slashes stripped, empty paths throw `StorageError`. Adapter implementations always see clean paths. Authors who construct an `Adapter` literal without `defineAdapter` are responsible themselves.
- **No bucket vocabulary in the public API.** The storage location is the adapter's concern.
- **Snapshot and fork are core operations.** `snapshots` and `forks` are required on every `Adapter` — there is no SDK-level polyfill. Adapters that can't support either throw `StorageError` with code `NotSupported` from each method.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [storagesdk/storagesdk](https://github.com/storagesdk/storagesdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
