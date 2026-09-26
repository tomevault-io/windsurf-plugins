---
trigger: always_on
description: All tests run inside Docker against **real KOReader** (headless). No mocks,
---

# Project: acsm.koplugin

## Testing

All tests run inside Docker against **real KOReader** (headless). No mocks,
no host-only tier. Uses the [koplugin-dev](https://github.com/kaikozlov/koplugin-dev)
Docker image (`ghcr.io/kaikozlov/koplugin-dev`) which ships the official
KOReader Linux release with all native FFI libraries (`libcrypto.so.57`,
`libz.so.1`, `libarchive`, `libSDL3`, etc.) so tests exercise the exact same
code paths as the plugin on a real device.

```bash
just setup                 # install git hooks and pull the koplugin-dev image (one-time)
just verify                # read-only fmt + lint + all non-e2e tests (pre-push/CI)
just verify-static         # read-only fmt + lint checks (pre-commit)
just test                  # run all tests (quiet; excludes e2e)
V=1 just test              # same, with full busted --verbose output
just test-e2e              # run e2e tests (hits real Adobe servers)
just test-all              # run everything including e2e
just test-filter Crypto    # run a subset by pattern
just build                 # build a release zip (versioned from _meta.lua)
just shell                 # drop into bash inside the container
just lint                  # run luacheck inside the container
just fmt-check             # check Lua formatting with stylua
just fmt                   # format Lua code with stylua
just check                 # mutating fmt + lint + test pass in one container
```

Shared recipes are vendored at `just/shared.just` (from koplugin-dev). Refresh with
`just sync-shared` when upstream recipes change, then commit the file.
Product packaging stays local: `just build`.

### Spec layout

All specs live under `spec/` and run together via `busted-koreader`:

| Location | What | Notes |
|---|---|---|
| `spec/*_spec.lua` | Module-level tests (epub, naming, fulfillment) | Real KOReader libs, real crypto |
| `spec/integration/*_spec.lua` | Cross-module tests (lifecycle, flows, DOM, crypto round-trips) | Real KOReader libs, real crypto |
| `spec/integration/pdf_e2e_spec.lua` | Full activation → fulfillment → PDF decrypt (Daisy Miller) | Tagged `#e2e`, requires network |
| `spec/integration/sample_library_e2e_spec.lua` | Full library sweep: activation → fulfillment → decrypt for 26 public ACSM samples | Tagged `#e2e`, ~90 s, ~15 MB of downloads |

The only tag in use is `#e2e` — `just test` excludes it because it hits
Adobe's servers and needs network access.

### E2E tests

The e2e suite (`spec/integration/e2e_spec.lua`) hits **real Adobe Content
Server** end-to-end. It exercises the entire pipeline that a user would go
through:

1. Download a real `.acsm` from Adobe's free sample library
2. Anonymous sign-in via `adobe.signIn`
3. Device activation via `adobe.activate`
4. Fulfillment: ACSM → encrypted EPUB download
5. Decryption: remove Adobe DRM, produce a valid EPUB

The test uses Adobe's smallest free sample ("God Is A Salesman" chapter 1,
~100 KB) to minimize download time. It validates the output is a real EPUB
(PK zip header) and that decryption produced >0 entries.

`spec/integration/sample_library_e2e_spec.lua` runs the same pipeline across
26 public ACSM samples (both formats, multiple publishers). Its validated
resource IDs are embedded so tracked tests never depend on the intentionally
untracked `REFERENCE/` directory. It shares one anonymous activation across
all books and asserts each output's format from magic bytes — not catalog
metadata, which is stale for at least one sample (Der Schimmelreiter is listed
as EPUB but ships as PDF).

```bash
just test-e2e   # must have network; uses --network=host
```

No credentials or configuration are needed — anonymous sign-in works for
free samples. If Adobe's servers are down or rate-limiting, the test will
fail with a network or HTTP error.

### How it works

- **Image**: `ghcr.io/kaikozlov/koplugin-dev` — unified dev image
  with KOReader + busted + luacheck + stylua.
- **KOReader**: extracted to `/opt/lib/koreader/`, includes bundled `luajit`.
- **Plugin**: bind-mounted at `/opt/plugin`, auto-symlinked into KOReader's
  plugins dir by `entrypoint.sh` so `PluginLoader:_discover()` finds it.
- **Bootstrap**: `/opt/koplugin-dev/commonrequire.lua` (busted `--helper`)
  sets up headless mode (`einkfb.dummy`, `Input.dummy`), isolated settings
  in `/tmp`, and exposes `load_plugin()`, `fastforward_ui_events()`,
  `disable_plugins()`.
- **Wrapper**: `/usr/local/bin/busted-koreader` invokes KOReader's `luajit`
  with `LUA_PATH` pointing at busted and KOReader modules.

### Driving real KOReader in tests

The Docker environment runs the **real KOReader binary, headless** — it is not a
mocked host tier. Every busted spec can `require()` any real KOReader module
(`FileManager`, `ReaderUI`, `UIManager`, `DocumentRegistry`, `PluginLoader`,
`BookInfo`, …) and drive it directly, exactly as a device would. **Prefer this
over hand-rolled stubs.** When KOReader is how your code gets called, make the
test call it the same way.

The bootstrap (`commonrequire.lua`) exposes three globals for this:

| Helper | What it does |
|---|---|
| `disable_plugins()` | Wipes the PluginLoader cache so only what you explicitly load runs |
| `load_plugin("acsm.koplugin")` | Discovers and loads just our plugin |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kaikozlov/acsm.koplugin](https://github.com/kaikozlov/acsm.koplugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
