---
trigger: always_on
description: Fast HDR / HDR10+ / Dolby Vision metadata inspector: one native Rust binary that memory-maps a video
---

# CLAUDE.md — working notes for hdrprobe

Fast HDR / HDR10+ / Dolby Vision metadata inspector: one native Rust binary that memory-maps a video
file, demuxes without decoding, samples RPUs, and prints a sectioned report in less than 1 second.
It also parses metadata **sidecar** files (raw DV RPU, DV CM XML, HDR10+ JSON) into the same
report. This file plus the module-level doc comments are the design reference — read the
relevant section and the code it points at before non-trivial changes.

## Commands

```sh
cargo build --release          # binary at target/release/hdrprobe
cargo test                     # 550 unit tests
cargo clippy --release         # must stay at zero warnings
./target/release/hdrprobe testfiles/integration/ -q   # one-line report per corpus file
```

Bar for any change: **zero `cargo build` warnings, zero `cargo clippy` warnings, all tests
pass, and the corpus (`-q`) output is unchanged** unless the change intends to alter it.

## Branch flow

Every version cycle develops on one long-lived `dev` branch. **Never commit
work-in-progress directly to main**: main receives the cycle as a single `--no-ff` merge at
release time, so pre-release doc/schema edits (README, SCHEMA.md "Ships in" notes) never
appear on main ahead of the version they describe. The project-local `/commit` skill pushes
to `dev` only; `/release` performs the merge, tag, and push to main, and leaves `dev` in
place rather than deleting it. (History before v0.8.0 was committed straight to main;
v0.8.0 through v1.0.0 used a per-cycle `dev/vX.Y.Z` branch, renamed to a persistent `dev`
after v1.0.0 — `dev` and `dev/*` cannot coexist as refs, so the old names are gone rather
than kept alongside.)

## Third-party license attribution

`THIRD-PARTY-LICENSES.md` is **generated — never hand-edit it.** It lists every crate compiled
into the release binary, grouped by license, and is produced by [`cargo about`](https://github.com/EmbarkStudios/cargo-about)
from `about.toml` (the accepted-license allowlist + the target set we publish for) and `about.hbs`
(the Markdown template):

```sh
cargo install --locked --features cli cargo-about     # one-time
cargo about generate about.hbs -o THIRD-PARTY-LICENSES.md
```

The committed file must match what the current dependency tree produces, so after any change to
`Cargo.toml`/`Cargo.lock` — and as a **release gate** — regenerate and fail on drift:

```sh
cargo about generate about.hbs -o THIRD-PARTY-LICENSES.md
git diff --exit-code THIRD-PARTY-LICENSES.md          # nonzero exit => stale; commit the update
```

Generation itself **fails** if a dependency pulls in a license not in `about.toml`'s `accepted`
list — that's the guard against silently bundling an incompatible (e.g. copyleft) license into a
binary, not a nuisance: vet the license and confirm MIT-compatibility before adding it. The project
itself stays MIT (see `LICENSE`); dev-dependencies (not shipped) and the `hdrprobe` crate itself are
excluded by config.

## Release binaries

Pushing a version tag (`v*`) runs `.github/workflows/release.yml`: it enforces the gates above
(clippy/tests under `-Dwarnings`, the license drift check, tag == `Cargo.toml` version), builds and
tests the binary for Windows x86_64, Linux x86_64 + aarch64 each as glibc + fully-static musl
(no libc/loader dependency — aarch64 for minimal userspaces like CoreELEC/LibreELEC boxes across
old vendor kernels through current, x86_64 for appliance NAS OSes like Unraid whose RAM-resident
root makes a drop-in static binary the natural install), macOS arm64 + Intel (Intel is
cross-compiled on the arm64 runner and tested via Rosetta), and FreeBSD x86_64 (no GitHub
runner exists, so a separate job builds, tests, and packages inside a FreeBSD VM on the Linux
runner — keeping the build-and-test-on-target rule), and attaches the archives plus
`SHA256SUMS` to a **draft** GitHub release for manual review. Archives are named by
user-facing platform (`hdrprobe-<version>-linux-x64-static` etc., the matrix `name`
field), never raw target triples — keep that mapping when adding targets. A `workflow_dispatch` run exercises
the gates and builds without creating a release. The corpus `-q` check stays a manual pre-tag step
(`testfiles/` is local-only). The code is deliberately portable outside `shell.rs`/`prefetch.rs`'s
`cfg(windows)` branches — keep new platform-specific code behind `cfg` with a non-Windows path, and
never parse bytes native-endian.

- `main.rs` — clap CLI, per-file dispatch (sidecar files first, then the Blu-ray ISO branch,
  then the video pipeline), exit codes (0 ok / 1 usage / 2 unreadable).
- `container/` — one hand-rolled demuxer per format: `mp4.rs`, `mkv.rs`, `ts.rs`, `annexb.rs`,
  `av1.rs` (which also owns the IVF wrapper's FourCC dispatch: `VP90` → the VP9 IVF demux,
  `VP80` → an honest error, else AV1), `mpegv.rs` (raw MPEG-1/2 video elementary stream, the
  thinnest backend in the tree: a bounded head read fills the General fields and `chunks` stays
  empty, per the metadata-only contract on `TrackDemux::chunks`; under `--full` a count-only
  fused walk — `RawFullStream::Mpegv`, the Ogg shape — counts picture start codes so duration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [matthane/hdrprobe](https://github.com/matthane/hdrprobe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
