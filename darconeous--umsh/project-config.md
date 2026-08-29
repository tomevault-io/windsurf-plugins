---
trigger: always_on
description: Experimental, LoRa-oriented mesh protocol with strong cryptography, strict layer
---

# UMSH

Experimental, LoRa-oriented mesh protocol with strong cryptography, strict layer
separation, and tight bandwidth discipline. Inspired by MeshCore but redesigned:
endpoints are identified by Ed25519 public keys, multicast uses shared symmetric
channel keys, and the MAC layer is timestamp-free (monotonic frame counters for
replay protection) with AES-SIV (RFC 5297) nonce-misuse-resistant encryption. The repo
holds the protocol **spec**, a Rust **reference implementation**, embedded
**firmware** for several LoRa boards, an **iOS app**, and a **Wireshark dissector**.

Spec: `docs/protocol/` (mdBook). Everything here was written with heavy LLM
assistance and is explicitly experimental — expect code smells and WIP APIs.

There is no significant installed user base, so versioning and migration are a matter
of immediate convenience and not mandatory at this time. Carefully weigh the design
costs of migration before considering implementing it.

## Repository layout

- `crates/` — host-side + `no_std` library crates (the reference implementation).
  `umsh-hal` is standalone (no workspace deps); `umsh-bsp-*` are board support over a
  shared `nrf52840` base.
- `umsh/` — umbrella crate re-exporting the workspace; defines the `Platform` trait + Tokio/Embassy adapters. **Library only** — host binaries live in `tools/`, which is what keeps clap/rustyline out of the umbrella's dependency tree.
- `firmware/` — nRF52840 firmware (thumbv7em, UF2/DFU). **One shipping image per board** (t1000e / techo / sensecap-solar / wio-tracker-l1 / xiao-nrf52) — there is no separate repeater build; role is configuration. All five are thin manifests over the shared `firmware/nrf52-tracker/src/main.rs`, differing only by their `board-*` feature. The existing `*-console` builds are per-board bringup harnesses, not products; **new boards do not get one** (xiao-nrf52 ships the device image only).
- `firmware-esp32/` — **separate cargo workspace** for Xtensa boards (Heltec LoRa32 V2 and V3, LILYGO T-Beam Supreme). See `firmware-esp32/CLAUDE.md` for its toolchain.
- `apps/ios/` — SwiftUI app; `packages/UMSHMobileCore` — UniFFI Swift package.
- `tools/` — host binaries and dev tooling (`crates/` is reserved for library crates):
  - `umshctl` — the radio tool (clap + rustyline; capture is a subcommand)
  - `umsh-bridge` — the internet bridge daemon (`docs/protocol/src/internet-bridging.md`); lib+bin split so the integration tests can stand a whole bridge up in one process. TOML config + `tracing`, its own dependency table
  - `regiondb-build` — the region-database compiler. **Python, not Rust**, and the repo's only
    packaged Python (`pyproject.toml` + `uv.lock`, run through `uv`); `scripts/` stays
    stdlib-only because building firmware must need nothing but a Rust toolchain
  - `ulcp-web-debugger`, `uniffi-bindgen`
- `regions/` — source data and manifests for the geographic region database, plus the
  committed test fixture. Three stages with distinct commit policies: `regions-fetch`
  (network → gitignored `vendor/`), `regions-update` (→ **committed** `extracts/`), and
  `regions-build` (committed tree → `dist/`). Every layer's extract is committed — the
  country layer is land buffered ~100 km seaward and clipped to the EEZ ceiling, not coastlines, so it stays small — and a
  clean checkout builds the world offline. See `regions/README.md` and `regions/FORMAT.md`.
- `docs/` — protocol spec (`protocol/`), per-board hardware docs, firmware/feature plans, UX.
- `dissectors/umsh/` — Wireshark Lua dissector. `diag/`, `contrib/systemd/`, `scripts/` (`mkuf2.py` builds the UF2, `flash.py` only flashes one, board table in `firmware_image.py`).

## Build / test

- **Formatting**: enable the checked-in pre-commit hook once per clone with `git config core.hooksPath .githooks`. It rejects commits that aren't rustfmt-clean in either workspace; CI checks the same two. `firmware-esp32/` needs `cargo +stable fmt --all` (its `rust-toolchain.toml` pins the `esp` channel; stable rustfmt gives identical output).

- Host: `cargo build` / `cargo test` / `cargo check` from root — **skips `firmware/*`** by design (`default-members`); host crates only.
- Firmware crates are **excluded from default builds** and must be built from inside their own directory so the per-firmware `.cargo/config.toml` (target triple + linker flags) is picked up. Building with `--manifest-path`/`-p` from root silently drops those flags and yields a broken ELF.
- nRF52840 firmware **only links in `--release`** (dev overflows flash). `cargo check` is fine at any profile.
- No bindgen env vars needed — plain `cargo build --release` works (do NOT set LIBCLANG_PATH/BINDGEN_EXTRA_CLANG_ARGS).
- Region database: `make regions-test` (ruff + pytest + `cargo test -p umsh-regiondb`) and
  `make regions-check` both run offline against the **committed** `regions/tests/fixture/`
  database. `cargo test` never builds one — regenerate it with `make regions-build-fixture`
  after changing the fixture tree or the compiled format, and regenerate
  `regions/tests/conformance.json` with it.

## Flashing

Use the Makefile — don't invoke objcopy/uf2conv/espflash by hand. Per-board targets, DFU
entry, UF2 families, and flash-layout gotchas: see the `flashing` skill.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [darconeous/umsh](https://github.com/darconeous/umsh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
