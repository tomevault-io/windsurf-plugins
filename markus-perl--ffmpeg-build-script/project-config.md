---
trigger: always_on
description: Instructions for AI coding agents working in this repository.
---

# AGENTS.md

Instructions for AI coding agents working in this repository.

## What this repo is

A bash script, `build-ffmpeg`, that downloads, builds and statically links FFmpeg and ~70
of its dependencies from source, plus the Dockerfiles and CI that exercise it. There is no
application code, no test suite, and no build system of its own — the script *is* the
project.

The script is not one file: `build-ffmpeg` is a thin entry point that `source`s the
fragments under `src/` in an explicit order. There is **no assembly or codegen step** —
the fragments are the source, sourced at runtime. Users always get the whole tree (release
tarball, `git clone`, or the Dockerfiles' `COPY src`).

## Repository layout

| Path | What it is |
| --- | --- |
| `build-ffmpeg` | Entry point only: resolves `SCRIPT_DIR`, checks `src/` exists, sources the fragments in order. Edit it only to add, remove or reorder a fragment. |
| `src/` | **The script.** Almost every change goes here — see the fragment list below. |
| `web-install.sh`, `web-install-gpl-and-non-free.sh` | One-liner installers. They resolve the **latest release**, download GitHub's auto-generated archive for that tag, extract it and run `build-ffmpeg` from it. They do not fetch anything from `master`. |
| `Dockerfile`, `cuda-ubuntu.dockerfile`, `full-static.dockerfile`, `export.dockerfile` | Container builds, all exercised by CI. |
| `.github/workflows/build.yml` | `lint`, then six full builds: `build-linux`, `build-linux-with-system-libs`, `build-macos`, `build-docker`, `build-cuda-ubuntu-docker`, `build-full-static`, then `release-version-check` on `v*` tags only. |
| `README.md` | End-user documentation. Not contributor docs. |
| `.editorconfig` | shfmt reads its indent keys from here. |
| `.gitattributes` | `export-ignore` entries that keep repo infrastructure out of the release tarball `git archive` builds. Nothing the build needs may be listed there. |
| `packages/`, `workspace/`, `build/` | **Build output. Gitignored. Never read or edit these.** `packages/` holds ~70 extracted upstream source trees; grepping it will bury you in unrelated code. Not to be confused with `src/packages/`, which is script source — the `.gitignore` entries are anchored (`/packages`) precisely so they do not swallow it. |
| `docs/`, `plans/` | Gitignored scratch notes. Not part of the project. |

When searching the repo, restrict the search to the tracked files. `git ls-files` is the
reliable filter; a bare `grep -r .` is not.

## Releases vs master

`master` holds unreleased work; users get releases.

**Releases are drafted and published by hand.** Nothing is uploaded to them and nothing
needs to be: GitHub generates a source archive for every tag, so if the tag exists its
archive exists. `.gitattributes` `export-ignore` applies to those archives, which is what
keeps repo infrastructure out of what users download.

The installers resolve the newest release from the `/releases/latest` redirect, which
points at `/releases/tag/<tag>`, and then fetch `/archive/refs/tags/<tag>.tar.gz`. Do not
replace that with an `api.github.com` lookup: it is rate-limited to 60/hr per IP, breaks CI
runners behind shared NAT, and would add a `jq` dependency.

The `release-version-check` job asserts that a pushed tag matches `SCRIPT_VERSION` *in the
tagged commit*, read from `src/00-header.sh` (`v9.0.3` requires `SCRIPT_VERSION=9.0.3`), so
master carrying the next
release's version is legal while a mismatched tag fails. It does not gate on the builds —
there is no artifact to withhold, so it reports in seconds instead of after an hour.

## Style and tooling

- **Formatting is enforced.** `shfmt` (v3.12.0) must produce no diff. It reads
  `.editorconfig`: 4-space indent, `binary_next_line = false`, `switch_case_indent = false`.
  Run `shfmt -d build-ffmpeg web-install.sh web-install-gpl-and-non-free.sh src/` before
  finishing.
- **ShellCheck is enforced at `--severity=style`**, its strictest level, pinned to v0.11.0.
  Run it as `shellcheck -x --severity=style build-ffmpeg web-install.sh
  web-install-gpl-and-non-free.sh src/*.sh src/packages/*.sh`. **The fragments have to be
  named on the command line.** `-x` follows the `source` lines only to resolve definitions
  for the file being checked; it emits no diagnostics for the sourced files, so `-x` on the
  entry point alone lints the loader and nothing else. Each fragment therefore carries a
  `# shellcheck shell=bash` directive (fragments have no shebang), and the handful of
  variables that are set in one fragment and read in another carry a narrow
  `# shellcheck disable=` with a reason on the same line. The `# shellcheck source=`
  directives in the entry point stay — they make `-x` resolve the loader's own call graph.
  The scripts are currently clean, so anything it reports is a regression you introduced.
  Prefer fixing over silencing; if a `# shellcheck disable=` really is warranted, give it a
  reason comment on the same line.
- **Package consistency is enforced too**, in the same `lint` job, because both failures it
  catches are silent and only surface at download time — up to an hour into a build.
  `./build-ffmpeg --list-packages | grep MISSING` must print nothing: `download()` derives

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [markus-perl/ffmpeg-build-script](https://github.com/markus-perl/ffmpeg-build-script) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
