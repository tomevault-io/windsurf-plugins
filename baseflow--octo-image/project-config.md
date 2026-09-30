---
trigger: always_on
description: Technical reference for AI agents and contributors developing in this repository.
---

# octo_image agent instructions

Technical reference for AI agents and contributors developing in this repository.

Process and conduct live in their own files: contribution workflow in
[CONTRIBUTING.md](CONTRIBUTING.md), the [Contributor Covenant Code of
Conduct](CODE_OF_CONDUCT.md).

## Scope and stack

- **OctoImage** is a Flutter widget maintained by [Baseflow](https://baseflow.com):
  a drop-in replacement for `Image` that adds a placeholder-or-progress phase, an
  error phase, and a post-load transform hook, with cross-fades between phases.
- Single package at the repo root, not a monorepo. There is no nested package
  directory to `cd` into.
- The package has no runtime dependencies beyond the Flutter SDK itself.
- Run Flutter and Dart commands with the same tooling CI uses (`flutter`, `dart`).
  Nothing in this repo requires anything else. If you happen to manage SDK
  versions locally with [fvm](https://fvm.app), prefix commands with `fvm`; that
  is a personal setup choice and is never checked in.

### Prerequisites

- Basic Dart and Flutter knowledge
- A working Flutter SDK installation, stable channel, matching CI, currently
  Flutter **3.47.4** (`FLUTTER_VERSION` in `.github/workflows/build.yaml`)
- For running or building the example on iOS/macOS, access to a Mac is required
- Android example builds require JDK 17

### Reference documentation

This package is a **Dart/Flutter widget library**, not a federated platform
plugin. Prefer official Flutter/Dart docs and this repo's existing code for
hands-on work:

- [Using packages](https://docs.flutter.dev/packages-and-plugins/using-packages)
- [Developing packages & plugins](https://docs.flutter.dev/packages-and-plugins/developing-packages)
- [Effective Dart](https://dart.dev/effective-dart)
- [pub versioning philosophy](https://dart.dev/tools/pub/versioning)

## Architecture overview

`OctoImage` takes any `ImageProvider` and delegates all actual decoding to a
framework `Image`; it owns no networking and no caching of its own. The four
public builder typedefs in `lib/src/image/image.dart` are the extension points:

```
OctoImage(imageBuilder, placeholderBuilder, progressIndicatorBuilder, errorBuilder)
    → ImageHandler (lib/src/image/image_handler.dart, internal)
        maps the builders onto Image's frameBuilder / loadingBuilder / errorBuilder
        and inserts FadeWidget cross-fades between phases
    → Image (framework widget, does the actual decode)
```

`OctoSet` (`lib/src/octo_set.dart`) bundles a placeholder-or-progress builder
with an optional image builder and error builder, consumed by
`OctoImage.fromSet`. `OctoPlaceholder`, `OctoProgressIndicator`, `OctoError` and
`OctoImageTransformer` are namespaces of prebuilt builders that sets are usually
assembled from.

## Authoritative project structure

- Public API: `lib/octo_image.dart` (exports only; the widget itself lives under `lib/src/`)
- Core widget: `lib/src/image/image.dart` (`OctoImage`, the four builder typedefs)
- Adapter/engine, not exported: `lib/src/image/image_handler.dart` (`ImageHandler`)
- Cross-fade, not exported: `lib/src/image/fade_widget.dart` (`FadeWidget`)
- Builder bundle: `lib/src/octo_set.dart` (`OctoSet`)
- Prebuilt placeholders: `lib/src/placeholders.dart` (`OctoPlaceholder`)
- Prebuilt progress indicators: `lib/src/progress_indicators.dart` (`OctoProgressIndicator`)
- Prebuilt error widgets: `lib/src/errors.dart` (`OctoError`)
- Prebuilt transformers: `lib/src/image_transformers.dart` (`OctoImageTransformer`)
- Example app: `example/`
- CI: `.github/workflows/build.yaml`

## Where to make changes

- **Public API / docs for app developers** → `lib/src/image/image.dart` exports
  and `README.md`
- **New prebuilt placeholder / error / progress indicator / transformer** →
  the matching file in the list above, following the existing static-factory
  style
- **Cross-fade behavior** → `lib/src/image/fade_widget.dart`

Two things to know before changing `ImageHandler` or `OctoSet`:

- **Placeholder and progress indicator are mutually exclusive.** Both
  `OctoSet` and `ImageHandler` assert this. Do not add a code path that sets
  both.
- **`ImageHandler` carries mutable state across callbacks.**
  `_wasSynchronouslyLoaded` and `_isLoaded` are written by one callback
  (`_preLoadingBuilder`) and read by another (`_loadingBuilder`). Gapless
  playback lives in `_OctoImageState.didUpdateWidget`, which rebuilds the
  `ImageHandler` on every widget update rather than only when the inputs
  actually changed; that behavior is the subject of an open pull request, so
  check for one before changing it.

Keep changes minimal in scope; match existing naming and testing patterns.

## Development setup

Baseflow's open-source forking workflow:

1. Fork `https://github.com/Baseflow/octo_image` on GitHub.
2. Clone your fork: `git clone git@github.com:<your_name>/octo_image.git`
3. Add upstream (the official repo you fetch from, not your fork):

```bash
git remote add upstream git@github.com:Baseflow/octo_image.git
```

4. Branch from latest `main`:

```bash
git fetch upstream
git checkout upstream/main -b <name_of_your_branch>
```

Expected remotes after setup:

```
origin    git@github.com:<your_name>/octo_image.git   # your fork (push here)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Baseflow/octo_image](https://github.com/Baseflow/octo_image) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
