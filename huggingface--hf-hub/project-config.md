---
trigger: always_on
description: Async Rust client library for the [Hugging Face Hub API](https://huggingface.co/docs/hub/api). This is the Rust equivalent of the Python [`huggingface_hub`](https://github.com/huggingface/huggingface_hub) library.
---

# hf-hub

Async Rust client library for the [Hugging Face Hub API](https://huggingface.co/docs/hub/api). This is the Rust equivalent of the Python [`huggingface_hub`](https://github.com/huggingface/huggingface_hub) library.

The primary entry point is the `HFClient` struct, which wraps an `Arc<HFClientInner>` for cheap cloning. All methods are async and use `reqwest` as the HTTP client. Paginated endpoints return `impl Stream<Item = Result<T>>` via `futures::stream::try_unfold`. Methods take parameters via [`bon`](https://docs.rs/bon) per-method builders finished with `.send().await` (mirroring reqwest / aws-sdk / octocrab style).

Key capabilities:

- Repository info, listing, creation, deletion, and settings updates
- File upload, download, listing, and deletion
- Commit creation, commit history, diffs between revisions
- Branch and tag management
- User and organization info
- Xet high-performance transfers

## Code Standards

These rules apply to ALL code written or modified in this repo:

### Style

- NO trivial comments — do not add comments that restate what the code does
- Descriptive variable and function names
- No wildcard imports (e.g., `use foo::*`), except `pub use` re-exports in `lib.rs`
- All imports are at the top of the file or top of module
- Latest stable Rust features are allowed

### Error Handling

- Use `Result<T, E>` with explicit error handling — never panic
- Define custom error types using `thiserror` for domain-specific errors
- Provide helpful, actionable error messages

### Performance

- Be mindful of allocations in hot paths
- Prefer structured logging (tracing/log macros with fields, not string formatting)

### Dependencies

- Add all dependencies to `Cargo.toml` (workspace root) or `hf-hub/Cargo.toml` (crate-level)
- Prefer well-maintained crates from crates.io
- Shared dependencies belong in the workspace `[dependencies]` table, not per-crate

### Testing

#### Unit Tests

- Place in the same file using `#[cfg(test)]` modules
- Run: `cargo test -p hf-hub`

#### Integration Tests

- Located in the `integration-tests` workspace crate, under `integration-tests/tests/` (`integration_test.rs`, `blocking_test.rs`, `bucket_sync_test.rs`, `bucket_xet_transfer_test.rs`, `cache_test.rs`, `download_test.rs`, `xet_transfer_test.rs`). Shared helpers live in `integration-tests/src/test_utils.rs`.
- Require a valid `HF_TOKEN` environment variable and internet access
- Tests skip gracefully when `HF_TOKEN` is not set (no failures)
- Run read-only tests: `HF_TOKEN=HF_xxx cargo test -p integration-tests`
- Write operation tests (create/delete repos, upload files) require `HF_TEST_WRITE=1`
- Run all tests including writes: `HF_TOKEN=HF_xxx HF_TEST_WRITE=1 cargo test -p integration-tests`

### Formatting and Linting

- Format: `cargo +nightly fmt`
- Lint: `cargo clippy -p hf-hub --all-features -- -D warnings`
- ALWAYS run both after making changes — do not skip this step

### Minimal Changes

- Verify that every change is minimal and necessary — do not include unrelated modifications

### WebAssembly compatibility

`hf-hub` supports `wasm32-unknown-unknown`; try not to break compatibility.
Filesystem- and tokio-runtime-dependent code is gated behind
`#[cfg(not(target_family = "wasm"))]` — keep new code out of wasm builds or
wasm-safe. Verify with `./scripts/verify_wasm.sh` (CI runs the same check);
browser tests and runtime build flags live under `wasm/`.

### Cargo features

| Feature | Default? | Notes |
|---------|----------|-------|
| `blocking` | off | Synchronous `*Sync` wrappers in `blocking.rs`. Pulls in `tokio/rt`. Native only. |
| `rustls-tls` | off | Force the rustls TLS backend on reqwest's native build. No effect on wasm. |
| `default` | (empty) | No features by default. |

### Method builders (bon)

All public methods on `HFClient`, `HFRepository`, `HFSpace`, and `HFBucket` use `bon`'s
per-method `#[builder(finish_fn = send)]` pattern. Do NOT introduce `*Params` wrapper structs —
parameters live directly on the method. Every method (including parameterless ones like `info`,
`exists`, `pause`, `whoami`) is finished with `.send()` for uniformity.

```rust
use bon::bon;

#[bon]
impl HFRepository {
    #[builder(finish_fn = send)]
    pub async fn my_method(
        &self,
        #[builder(into)] name: String,
        revision: Option<String>,
    ) -> HFResult<MyOutput> { ... }
}
```

Translation rules from `typed-builder` attributes (now removed):

| typed-builder | bon (as method param) |
|---|---|
| `#[builder(setter(into))] x: String` | `#[builder(into)] x: String` |
| `#[builder(default, setter(strip_option))] x: Option<T>` | `x: Option<T>` (implicit) |
| `#[builder(default, setter(into, strip_option))] x: Option<String>` | `#[builder(into)] x: Option<String>` |
| `#[builder(default)] x: bool` | `#[builder(default)] x: bool` |
| `#[builder(default = expr)] x: T` | `#[builder(default = expr)] x: T` |
| required field, no attribute | `x: T` |

bon also generates a `maybe_<field>(opt)` setter alongside each `Option<T>` parameter — use it

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [huggingface/hf-hub](https://github.com/huggingface/hf-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
