---
trigger: always_on
description: **ALWAYS use Makefile commands.** Never call scripts directly via `fvm dart run scripts/...`. The Makefile is the single entry point for all operations.
---

# libsignal - Claude Code Configuration

## Important Rules

**ALWAYS use Makefile commands.** Never call scripts directly via `fvm dart run scripts/...`. The Makefile is the single entry point for all operations.

```bash
# Correct - pass arguments via ARGS variable
make test
make analyze ARGS="--fatal-infos"
make check-new-libsignal-version ARGS="--update"

# Wrong - never do this
fvm dart run scripts/check_updates.dart --update
make test test/keys/  # make interprets test/keys/ as target!
```

## Available Makefile Commands

### Setup
```bash
make setup                        # Full setup (FVM + Rust tools + protoc)
make setup-fvm                    # Install FVM + Flutter only
make setup-rust-tools             # Install Rust tools (cargo-audit, cargo-deny, frb_codegen)
make setup-frb-codegen            # Install the pinned flutter_rust_bridge_codegen
make setup-protoc                 # Install protoc (Protocol Buffers compiler)
make setup-web                    # Install web build tools (wasm-pack)
make setup-android                # Install Android build tools (cargo-ndk)
make setup-repo-protections       # Apply the GitHub repo rulesets (.github/rulesets/)
```

### Code Generation
```bash
make codegen                      # Generate Dart bindings from Rust code
```

**Note:** `make codegen` automatically creates a `.skip_libsignal_hook` marker file to prevent Build Hooks from downloading libraries during codegen. The marker is automatically removed after completion.

### Build
```bash
make build                              # Build for current platform (always release)
make build ARGS="--target <target>"     # Build for specific Rust target
make build-android                      # Build for Android (all ABIs)
make build-android ARGS="--target arm64-v8a"  # Build for specific Android ABI
make verify-android-alignment           # Check the built .so files are 16 KB-aligned
make build-web                          # Build WASM for web
```

`ARGS` reaches `build-android` **after** the `build` word, and that is
deliberate: cargo-ndk's own options (`--target arm64-v8a`) are recognised
anywhere on the line, cargo's are not, so this position takes both kinds and the
other one takes only cargo-ndk's.

⚠ **The 16 KB alignment of the Android libraries comes from `cargo-ndk`, not
from the NDK.** cargo-ndk passes `-Wl,-z,max-page-size=16384`; artefacts built
with r26 and r28 measure `p_align=0x4000` alike. Google Play has required it of
an app's bundled native libraries, for apps targeting Android 15 or later, since
1 November 2025 — so a cargo-ndk that stopped passing the flag would not break
anything here, it would make **consumers'** apps unpublishable. The version is
therefore pinned by hand in both Android jobs (`build-libsignal.yml`
and `test-reusable.yml`, together — a gate on a different tool than the release
is not a gate), and `make verify-android-alignment` measures the result rather
than trusting the pin. Both jobs run it.

### Web

```bash
make test-web                     # Browser suite: headless Chrome, real DOM APIs
make test-web CHROMEDRIVER=/path/to/chromedriver   # pin the driver (see below)
make test-web ARGS="--release"    # release profile, when the suite gets slow
make test-web ARGS="-- --nocapture"                # show console::log output
```

`make test-web` is the **only** check here that EXECUTES web code. `make test`
is the Dart VM and `make build-web` only compiles, so without it every
`cfg(target_arch = "wasm32")` branch in the crate is covered by nothing — and
those branches are exactly the ones nothing else can reach, because a wasm32
body is a *different implementation* of the same function rather than the same
code on another host.

⚠ **Pin chromedriver.** wasm-pack looks for one on `$PATH`, honours
`--chromedriver`, and otherwise DOWNLOADS THE LATEST — an unpinned network
fetch on every run. The `CHROMEDRIVER` **environment variable is not honoured**
— wasm-pack overwrites it with its own — hence the make variable, which becomes
the flag. CI resolves the runner image's preinstalled driver and fails closed if
there is none.

⚠ **That pins the DRIVER and not the BROWSER, and a green run is no evidence
otherwise.** With no `webdriver.json` and no `WASM_BINDGEN_TEST_WEBDRIVER_JSON`,
the test runner sends empty capabilities and ChromeDriver launches whatever
Chrome is installed. To pin it, point that variable at a file carrying
`{"goog:chromeOptions": {"binary": "<browser path>"}}`; the run then prints `Ok`
instead of `Not found` under "Try find webdriver.json". `Ok` alone only proves
the file was READ — check that a deliberately bad path fails, or the pin may be
parsed and dropped.

⚠ **A mismatched driver does not necessarily fail loudly**, so "it went green"
does not mean the driver matched the browser.

⚠ **`WASM_BINDGEN_TEST_TIMEOUT` is raised to 120 s in the Makefile.** The
browser runs every test on one JS thread, so a test that blocks it starves the
callbacks every concurrently-driven test is waiting on. The failure that
produces names whichever test was scheduled LAST rather than the slow one, so it
reads as a hang somewhere unrelated.

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [djx-y-z/libsignal_dart](https://github.com/djx-y-z/libsignal_dart) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
