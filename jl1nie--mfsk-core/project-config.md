---
trigger: always_on
description: Repo-level notes for assistants working in this tree. Anything specific to
---

# mfsk-core — agent notes

Repo-level notes for assistants working in this tree. Anything specific to
a sub-crate lives in its own `CLAUDE.md` or `README.md`; this file is for
cross-crate workflow that's easy to forget between sessions.

## Embedded targets

The active production crates are `embedded-poc/m5stack-s3-app/` (S3
LX7, repositioned as **demo / acoustic-fallback** in the 2026-05-17
pivot — the StickS3 board can't do USB host),
`embedded-poc/m5stack-core2-app/` (Core2 LX6, wav_sim only — no USB
peripheral on classic ESP32), and `embedded-poc/m5stack-cores3-app/`
(S3 LX7, **main UAC controller target** — CoreS3 has AXP2101 +
AW9523B for proper USB-OTG host mode). Phase 0-Core (board bring-up)
and Phase 1-Core (UAC host code) both shipped 2026-05-23; what's
still open is Phase 1-Verify — live IC-705 hardware confirmation,
tracked as issue #163 and, as of this writing, the single blocker for
the rest of the Phase B-Core sequence (see `docs/notes/ROADMAP.md`
Phase B-Core for the live status). `m5stack-s3-app` and
`m5stack-core2-app` each have their own `CLAUDE.md` covering
board-specific bring-up; `m5stack-cores3-app` does not yet — read
`docs/notes/ROADMAP.md` Phase B-Core for its status instead. This
section captures the shared workflow that's easy to forget between
sessions.

**WSPR embedded RX (Phase E, issue #260) is a separate track from the
FT8-controller line above** — it never goes through `decode_block` or
the UAC/controller stack, so it isn't blocked on #163 the way Phase
B-Core is (both share #163 only for live audio capture; everything
else already works against WAV-fed/synthetic baseband). Lives in the
same `m5stack-cores3-app` crate as two separate binaries:
`src/bin/wspr_bench.rs` (timing measurement) and `src/bin/wspr_app.rs`
(the standalone receiver — LCD spot list, WiFi, HTTP config, NTP, and
real UAC audio through `AudioSink`). Note that `wspr_app` *does* now
share `uac.rs` with the FT8 line, so it shares #163 for that path too
— open items in #313. See `docs/reference/EMBEDDED.md`'s "WSPR on
embedded" section and `docs/notes/ROADMAP.md` Phase E for status.

- **Build & flash via `espflash`**, not host cargo. Both crates'
  `.cargo/config.toml` set `runner = "espflash flash --monitor"`, so the
  basic user workflow is:
  ```sh
  cd embedded-poc/m5stack-s3-app   # or m5stack-core2-app
  cargo run --release              # builds + flashes + opens serial monitor
  ```
  In practice, for capturing per-session logs to a file, use
  `embedded-poc/scripts/flash-monitor.sh` (see next section) instead of
  the bare runner.
- The `+esp` Rust toolchain (Xtensa fork, espup-installed) is selected
  by each crate's `rust-toolchain.toml`. Target triple per
  `.cargo/config.toml`: `xtensa-esp32-espidf` for Core2 (LX6),
  `xtensa-esp32s3-espidf` for S3 (LX7).
- `cargo check` from inside each crate validates code changes without
  flashing (~30-50 s with prebuilt esp-idf).
- Logs from the device land in `embedded-poc/<crate>/logs/` — user has
  been capturing per-session sweep output there.
- The user actively flashes both boards during embedded work; do NOT
  assume "host check is enough" when changing `src/main.rs` or anything
  that affects the runtime path. Offer to flash and capture a new log.
- The compute-bench crate `embedded-poc/m5stack-s3/` still exists for
  S3-only decoder timing sweeps. The Core2 bench (`m5stack-core2/`) was
  retired in `#61` Phase 3 — `m5stack-core2-app` covers the same
  wav_sim decode path in a production-app shape.

## Capturing logs from a flashed device (ESP32 / S3)

Use `embedded-poc/scripts/flash-monitor.sh` — **never** roll your own
`espflash flash --monitor` + redirect, and never `cat /dev/ttyACM0`. Two
foot-guns this script avoids:

1. `espflash monitor` defaults to `--before default-reset`, which pulses
   DTR/RTS and on S3 USB-OTG boards drops the chip into DOWNLOAD mode
   (`rst:0x15 USB_UART_CHIP_RESET … waiting for download`). The script
   passes `--before no-reset --after no-reset` so the just-flashed app
   keeps running.
2. Re-flashing the same ELF prints "Segment … has not changed, skipping
   write" and finishes in ~5 s. **That is not a successful flash** — the
   chip still runs the previous binary. Touch a source file or change a
   `log::info!` line to force a real rewrite, and expect ~15-25 s for a
   real factory-partition write.

```sh
source ~/export-esp.sh
cd embedded-poc/m5stack-s3-app   # or m5stack-s3, m5stack-core2-app
cargo build --release --bin <bin>
../scripts/flash-monitor.sh \
    target/<triple>/release/<bin> \
    logs/<bin>_<tag>_$(date +%Y-%m-%d).log \
    90    # capture seconds (optional, default 90; use ≥120 for fresh Core2 flashes — 1.3 MB binary takes ~55 s to write)
```

## Test fixture paths

Never hardcode absolute paths like `/home/ubuntu/...` or `/Users/...`
for test inputs. AI assistants tend to "fix" path failures by
translating to whichever local environment they happen to run in
(commit `119657a` flipped `/home/minoru/` → `/home/ubuntu/`), which
just relocates the bug.

- **In-repo assets**: use the `asset_path!` macro from
  `mfsk-core/tests/common/mod.rs` (integration tests) or
  `concat!(env!("CARGO_MANIFEST_DIR"), "/../embedded-poc/assets/<f>")`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jl1nie/mfsk-core](https://github.com/jl1nie/mfsk-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
