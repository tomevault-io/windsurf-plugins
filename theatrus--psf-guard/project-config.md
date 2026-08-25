---
trigger: always_on
description: This is the canonical guide for people and coding agents working in this
---

# PSF Guard contributor guide

This is the canonical guide for people and coding agents working in this
repository. `AGENTS.md` is a symlink to this file so both entry points stay in
sync.

## Writing

- Use plain, direct English.
- Cut words that do not add meaning.
- Prefer active voice.
- Keep technical terms when they improve accuracy.
- Update the nearest focused document instead of adding another root-level
  plan.

## Product

PSF Guard is an astrophotography catalog, grader, and quality-analysis tool. It
can build a catalog from FITS folders or open an existing N.I.N.A. Target
Scheduler database. The same Rust application runs as a CLI, an Axum server,
or a Tauri desktop app. A React frontend supplies the grader and management UI.

Main capabilities include:

- multi-database FITS catalog management;
- fast image review, comparison, grading, and export;
- star detection, PSF fitting, spatial screening, and photometry;
- pixel-derived astrometry, sky overlays, and satellite prediction;
- calibrated mono stacks and RGB, LRGB, or narrowband previews;
- header-first imports, background quality backfill, and safe database sync;
- reversible reject archives and remote image intake.

## Toolchain and common commands

The repository uses the Rust toolchain pinned in `rust-toolchain.toml`, Rust
2024, Node 24 in CI, and Tauri 2.

```bash
# Rust checks used by CI
cargo fmt -- --check
cargo build
cargo clippy --all-targets --all-features -- -D warnings
cargo test

# Frontend checks
cd static
npm ci
npm run lint
npx vitest run
npm run build

# Browser end-to-end tests; requires a built psf-guard binary
npx playwright test

# Run the server or desktop app
cargo run -- server db.sqlite /path/to/images
cargo tauri dev

# Build release forms
cargo build --release
cargo build --release --features tauri
cargo tauri build
```

Use `--registry /tmp/psf-guard-test.json` for ad-hoc server work that must not
touch the user's real database registry. Do not run import, sync, catalog
installation, quality scans, or database-management actions against a live
catalog unless the task grants that access.

## Repository map

| Path | Purpose |
|---|---|
| `src/cli_main.rs`, `src/commands/` | CLI commands and batch workflows |
| `src/db.rs`, `src/db_registry.rs` | Target Scheduler access and multi-database registry |
| `src/server/` | Axum API, jobs, caches, import, sync, previews, and remote intake |
| `src/sequence_analysis.rs` | Sequence scoring and issue classification |
| `src/spatial_analysis.rs`, `src/photometry.rs` | Cloud, obstruction, glow, and transparency evidence |
| `src/astrometry.rs`, `src/acquisition_context.rs` | Solving, target context, and pointing evidence |
| `src/satellites.rs` | Cached satellite prediction and trail alignment |
| `src/calibration.rs` | Calibration catalog, matching, and master provenance |
| `static/src/` | React application |
| `static/e2e/` | Playwright coverage and real FITS fixture manifest |
| `docs/` | Current user and engineering documentation |
| `docs/design/` | Design records that still guide active architecture |
| `packaging/` | RPM and platform packaging |
| `icons/` | Generated desktop icons; see `icons/README.md` |

The server caches `notice.json` for 24 hours for both browser and Tauri modes;
UI reloads read `/api/update-notice` and never fetch the public feeds. Only
Tauri loads the signed updater plugin. Both feeds try `updates.psf-guard.com`
before the GitHub release fallback. See [docs/UPDATES.md](docs/UPDATES.md).

## Core invariants

### Databases and grading

- Treat Target Scheduler compatibility as a public contract. Probe optional
  columns and tolerate supported schema differences.
- Target Scheduler stores right ascension in decimal hours. Convert it to ICRS
  degrees at the astrometry boundary.
- `gradingStatus` values are `0 = Pending`, `1 = Accepted`, and `2 = Rejected`.
  Keep `rejectreason` consistent with the grade.
- Match records across databases by stable GUID. Do not substitute names or
  nearby coordinates for sync identity.
- Open a source database read-only during sync. Refuse the same source and
  destination path. Preview every UI transfer before Apply.
- The Overview merges databases, but Grid, Detail, Comparison, and Sequence
  routes must keep the `db` slug in URL state. The Overview carries the same
  scope so it can mark where the user was and hand it back, so anything that
  scopes to one database must read `useScopedDbId` and ignore the slug there.

Design records: [multi-database](docs/design/multi-database.md),
[data transfer](docs/design/data-transfer.md), and
[reject archive](docs/design/reject-archive.md).

### Image evidence and grading

- Keep catalog predictions separate from pixel evidence. A FITS header, target
  coordinate, or satellite ephemeris does not prove that the pixels match.
- Quality grading must renormalize when evidence is missing. Do not punish an
  image because an optional scan has not run.
- Only fresh pixel-derived solves support astrometry grading. Embedded WCS can
  support display but does not prove the current pixels.
- A lone no-solve is advisory. Automatic rejection needs corroborating
  pointing, cloud, obstruction, or tracking evidence.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [theatrus/psf-guard](https://github.com/theatrus/psf-guard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-24 -->
