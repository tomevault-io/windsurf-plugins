---
trigger: always_on
description: This file contains Downright-specific context, invariants, and evidence gates. Follow any applicable higher-level instructions as well.
---

# Downright Repository Guide

This file contains Downright-specific context, invariants, and evidence gates. Follow any applicable higher-level instructions as well.

## Project Model and Live State

- Downright is a native macOS 14+ Markdown reader/editor built with AppKit and TextKit 2. It is local-first and source-preserving.
- The main boundaries are `MarkdownCore` for parsing and source edits, `MarkdownRender` for TextKit presentation, `DownrightApp` for windows/files/actions, `DownrightQL` and `DownrightThumb` for Finder, Spotlight targets for metadata, and `down` for the CLI.
- This checkout is the native app. `/Volumes/Neural/downright-website` is the Astro site; `/Volumes/Neural/product-downright` is a separate product/media workspace. Do not substitute one for another.
- Source, SwiftPM bundles, Xcode bundles, `/Applications/Downright.app`, Finder extensions, mounted DMGs, and public releases are distinct surfaces. Name the surface in scope before changing or validating it.
- Treat this file as durable policy, not a status page. Never rely on hard-coded SHAs, branch divergence, version/build numbers, test counts, benchmark results, release tags, download totals, or installed-app identity.
- Resolve only the volatile state relevant to the task or a claim from its live authority. A documentation edit does not require querying releases or inspecting the installed app:
  - checkout: `git status --short --branch`, `HEAD`, and refreshed upstream state when current upstream truth is required;
  - local version/build: `Config/version.env`;
  - current tests and budgets: the relevant scripts’ fresh output;
  - public release/update state: the live GitHub Release, DMG/checksum, and Sparkle appcast;
  - running app state: exact bundle path, `Info.plist` version/build, executable identity, and process path.
- If live verification is unavailable or intentionally skipped, label conclusions snapshot-only. If this or a nested instruction file changes during the task, reread the applicable instructions before continuing.
- When a change alters an architectural, validation, privacy, or release contract named here, update this file and the relevant project documentation in the same change. Do not append dated status snapshots.
- Consult the relevant contract before changing behavior: `PRODUCT.md`, `DESIGN.md`, `Docs/ARCHITECTURE.md`, `Docs/QUICKLOOK.md`, `Docs/PERFORMANCE.md`, or `Docs/RELEASE.md`. `project.yml` is the XcodeGen source; `Config/version.env` is the canonical version/build source. Report material conflicts with the requested behavior or applicable contract; pause only work whose correctness or authorization depends on resolving them.

## Product Invariants

- Raw Markdown bytes on disk are authoritative. Rendering, hidden syntax, reflow, attachments, Quick Look, and presentation state must not rewrite the document.
- Source UTF-16 positions are canonical. Route source/display conversions through `DisplayMap`; never reuse a TextKit/display offset as a source offset.
- Preserve selection, undo, revision convergence, and a semantic source/reading anchor for operations expected to preserve position. Explicit navigation may move to its target; verify both the target and post-navigation anchor.
- Keep the one-TextKit-2-surface architecture unless the task explicitly changes that contract. Incremental typing must not cause a whole-document rebuild; length-changing edits must invalidate all shifted ranges and fragments.
- Make source mutations and undo boundaries explicit. Storage, autosave, watcher, and external-write changes need focused coverage for the relevant clean/dirty states and, when applicable, atomic replacement, deletion/rename, own-write suppression, conflict/review, failure, and recovery.
- Never translate an I/O failure into “no change,” recreate a deleted file silently, overwrite source after a rendering failure, or save after the user chose Discard.
- Downright has no app telemetry and does not upload document contents. The one recurring network request a production build makes on its own is a conditional `GET` of the public appcast, carrying no identifier and reading only a version; it stops with the "check automatically" setting, and it may trigger a Sparkle check but never parses the feed or installs anything. Public acquisition means GitHub Release `.dmg` asset requests across stable and versioned DMGs; exclude Sparkle ZIP/delta requests and never describe requests as unique people or completed installs.

## Validation by Surface

- For code changes, use focused tests while iterating, then run `Scripts/check.sh` as the authoritative source gate. For documentation-only changes, inspect content, links, and referenced commands without rebuilding the app. Do not substitute bare `swift test`; zero executed tests or a masked pipeline failure is a failed gate.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ezzy1630/Downright](https://github.com/ezzy1630/Downright) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
