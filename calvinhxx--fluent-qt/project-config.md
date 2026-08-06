---
trigger: always_on
description: - Use [README.md](README.md) for the project overview, supported Qt versions, CMake presets, and local IDE expectations.
---

# Project Guidelines

## Start Here

- Use [README.md](README.md) for the project overview, supported Qt versions, CMake presets, and local IDE expectations.
- Use [docs/development/README.md](docs/development/README.md) as the index for reusable workflow rules; link to those docs instead of copying them into new agent files.
- Use [docs/development/gallery-control-images.md](docs/development/gallery-control-images.md) when adding or regenerating Gallery component-card icons under `app/assets/control_images/` (transparent canvas, category color families, qrc registration).
- Use [docs/development/app-sample-optimization.md](docs/development/app-sample-optimization.md) when adding or editing Gallery live examples: Source code snippets must stay semantically aligned with the preview UI.
- Use [docs/development/linux-workflow.md](docs/development/linux-workflow.md) for desktop Linux portability, supported Qt versions, CI baselines, and Linux build/test presets.
- Use [docs/development/release-governance.md](docs/development/release-governance.md) for lightweight branch, Angular-style Conventional Commit, version, tag, changelog, and release checklist rules.
- Use [docs/development/packaging-workflow.md](docs/development/packaging-workflow.md) for macOS DMG, Windows installer, and Linux DEB packaging commands.
- Use [docs/architecture/README.md](docs/architecture/README.md) for architecture contracts, especially [docs/architecture/overlay-behavior.md](docs/architecture/overlay-behavior.md) when touching popup, flyout, dropdown, drawer, or other same-window overlay behavior.

## Build and Test

- The project is C++17 with Qt 5.15+ or Qt 6.2+. The reusable `FluentQt`
  library depends only on Qt Widgets; the Gallery adds spdlog/fmt and tests add
  GTest plus the application logging support through optional vcpkg features.
- Public CMake presets require `VCPKG_ROOT` and intentionally do not hard-code Qt, Ninja, Visual Studio, or Xcode paths. Put machine-specific paths in ignored `CMakeUserPresets.json` files.
- Common macOS arm64 flow:

```bash
cmake --preset vcpkg-osx
cmake --build --preset vcpkg-osx --parallel
ctest --preset vcpkg-osx --output-on-failure
```

- Always enable parallel compilation for project builds. Use
  `cmake --build --preset <preset> --parallel`, including focused target builds,
  unless a diagnosed toolchain or resource issue requires an explicit lower
  job count.
- Build focused test targets with the current host preset, for example
  `cmake --build --preset vcpkg-linux --target test_<name> --parallel` on Linux
  or `cmake --build --preset vcpkg-osx --target test_<name> --parallel` on
  macOS.
- Prefer anchored CTest label filters, for example `ctest --preset vcpkg-linux -L '^test_<name>$' --output-on-failure`; see [docs/development/testing-workflow.md](docs/development/testing-workflow.md).
- VisualCheck tests are interactive by design. Automated CTest runs set `SKIP_VISUAL_TEST=1`; run binaries directly with `--gtest_filter="*VisualCheck*"` for manual review or `VISUAL_SNAPSHOT=1` for snapshots.

## Architecture Map

- [src/design/](src/design/) contains Fluent design tokens for color, spacing, typography, radius, material, elevation, animation, and breakpoints.
- [src/compatibility/](src/compatibility/) contains Qt and platform compatibility helpers. Use `compatibility/QtCompat.h` and `FluentEnterEvent` instead of direct `QEnterEvent` in new `enterEvent` overrides.
- [src/utils/](src/utils/) contains library-side Qt logging categories and debug
  overlay helpers. Reusable code uses `qCDebug`, `qCInfo`, and `qCWarning` with
  a `fluentqt.*` category. Gallery and test diagnostics use the non-exported
  facade in [support/logging/Log.h](support/logging/Log.h); never add a
  spdlog dependency to `FluentQt` itself.
- [src/components/foundation/](src/components/foundation/) contains shared component infrastructure such as `fluent::FluentElement`, `fluent::QMLPlus`, `fluent::AnchorLayout`, and `fluent::overlay` contracts.
- [src/components/layout/](src/components/layout/) contains reusable composition surfaces such as `fluent::layout::Card`, `fluent::layout::Divider`, and `fluent::layout::Expander`.
- [src/components/](src/components/) is grouped by component category; component tests mirror those categories under [tests/components/](tests/components/).

## Component Conventions

- Treat [cmake/FluentQtInstallHeaders.cmake](cmake/FluentQtInstallHeaders.cmake)
  as the installed-header allowlist. Public header changes must update it;
  private implementation headers must stay out of the development package.
- Application examples use the single `<FluentQt/FluentQt.h>` entry header.
- Follow [docs/development/component-api-conventions.md](docs/development/component-api-conventions.md) and [docs/development/component-api-audit.md](docs/development/component-api-audit.md) when adding or changing public component APIs.
- Public non-trivial APIs under `src/` use concise Doxygen comments with English `@brief` plus a `zh_CN:` line. Do not mechanically rewrite untouched comments; see [docs/development/comment-style.md](docs/development/comment-style.md).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [calvinhxx/Fluent-Qt](https://github.com/calvinhxx/Fluent-Qt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
