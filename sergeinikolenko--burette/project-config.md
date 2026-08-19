---
trigger: always_on
description: Burette is a macOS desktop app, Finder Quick Look extension, source-built
---

# Agent Dispatch

Burette is a macOS desktop app, Finder Quick Look extension, source-built
iPhone preview app, and hosted public plugin for molecular structure files.
Keep this file as a dispatcher; load the focused doc for the surface you are
changing.

## Project Contracts

- Product name: `Burette`
- Package name: `burette`
- Tauri app identifier: `com.local.BuretteV10`
- Package manager: `bun@1.3.8`
- Stable agent CLIs: `scripts/burette-agent.mjs` and
  `scripts/agent-preview.mjs`
- Packaged agent plugin root: `plugins/burette-agent`
- Quick Look base bundle identifiers:
  `com.local.BuretteV10.Preview` and `com.local.BuretteV10.Thumbnail`
- Development installs must use `BURETTE_DEV_FLAVOR` so local app, extension,
  container, and forced-content-type namespaces do not collide.

## Documentation Graph

- User-facing overview: [README.md](README.md)
- Changelog pointer: [CHANGELOG.md](CHANGELOG.md)
- Documentation map: [docs/README.md](docs/README.md)
- Architecture: [docs/architecture.md](docs/architecture.md)
- Product direction: [docs/product.md](docs/product.md)
- Design direction: [docs/design-system.md](docs/design-system.md)
- Configuration: [docs/configuration.md](docs/configuration.md)
- Modular runtime refactor: [docs/modular-runtime-refactor.md](docs/modular-runtime-refactor.md)
- Renderer support: [docs/renderer-support.md](docs/renderer-support.md)
- Security and permissions: [docs/security-and-permissions.md](docs/security-and-permissions.md)
- Quick Look debugging: [docs/quicklook-debugging.md](docs/quicklook-debugging.md)
- Agent platform: [docs/agent-platform.md](docs/agent-platform.md)
- Repo-local Codex maintenance skills: [.codex/README.md](.codex/README.md)
- Agent tool index: [docs/tools/index.md](docs/tools/index.md)
- Testing surfaces: [docs/tools/testing-surfaces.md](docs/tools/testing-surfaces.md)
- Release process: [docs/releasing.md](docs/releasing.md)

## Directory Context

- Desktop hooks: [apps/desktop/src/hooks/README.md](apps/desktop/src/hooks/README.md)
- Desktop library helpers: [apps/desktop/src/lib/README.md](apps/desktop/src/lib/README.md)
- Desktop Vite runtime: [apps/desktop/vite/README.md](apps/desktop/vite/README.md)
- Quick Look extension: [PreviewExtension/AGENTS.md](PreviewExtension/AGENTS.md)
- iOS mobile app: [ios/BuretteMobile/AGENTS.md](ios/BuretteMobile/AGENTS.md)
- Agent plugin: [plugins/burette-agent/AGENTS.md](plugins/burette-agent/AGENTS.md)
- Repository scripts: [scripts/README.md](scripts/README.md)

## Runtime Boundaries

| Boundary | Primary paths | First doc to read |
| --- | --- | --- |
| Desktop app shell | `apps/desktop/src`, `apps/desktop/src-tauri` | `docs/architecture.md` |
| Browser-dev runtime | `apps/desktop/vite`, `apps/desktop/src/hooks` | `docs/tools/testing-surfaces.md` |
| Finder Quick Look | `PreviewExtension`, `PreviewExtension/Web` | `PreviewExtension/AGENTS.md` |
| iPhone source app | `ios/BuretteMobile` | `ios/BuretteMobile/AGENTS.md` |
| Agent CLI and sessions | `scripts/burette-agent.mjs`, `scripts/agent-preview.mjs` | `docs/agent-platform.md` |
| Packaged MCP plugin | `plugins/burette-agent` | `plugins/burette-agent/AGENTS.md` |
| Hosted public plugin and MCP | `apps/burette-public-plugin` | `docs/agent-platform.md` |
| Native compute layer | `crates/burette-compute-*`, `compute/`, `apps/desktop/src-tauri/src/compute` | `docs/gpu-compute-status.md` |
| Shared workspace packages | `packages/burette`, `packages/ketcher-agent-contract` | `docs/repository-layout.md` |
| Release and repository tooling | `scripts`, `.codex/skills` | `docs/tools/index.md` |

## Common Routing

- For frontend development and JavaScript validation, use Vite+ through `vp`;
  see [docs/vite-plus.md](docs/vite-plus.md) and [scripts/README.md](scripts/README.md).
- Install or refresh lightweight repository tools when the documented workflow
  requires them, for example `vp install` for Vite+ native bindings. Do not
  change application code to work around a missing local toolchain.
- For browser previews, use the built-in Browser plugin. Do not use macOS
  `open`, Chrome, Safari, or another external browser unless the user explicitly
  asks for an external browser.
- Do not open the desktop app as a substitute for a browser preview. Use
  `desktop-app` only for packaged app, native app, Quick Look, or other
  desktop-specific verification.
- For packaged local testing, always use a unique `BURETTE_DEV_FLAVOR` unless
  the task is explicitly release-bundle work.
- For Quick Look work, read [PreviewExtension/AGENTS.md](PreviewExtension/AGENTS.md)
  before building, installing, or forcing previews.
- For iPhone app work, read [ios/BuretteMobile/AGENTS.md](ios/BuretteMobile/AGENTS.md)
  and verify the intended surface: generic build, Simulator, or real device.
- For Apple-platform build/run/test work, invoke the relevant plugin first:
  `@build-ios-apps` for iOS and `@build-macos-apps` for macOS/Quick Look.
- For Apple-platform UI, UX, icon, SF Symbols, SwiftUI/AppKit, or visual polish
  work, invoke `$apple-design`; invoke `@product-design` for product flow,
  prototype, or design-context work before implementation.
- For plugin/MCP/skill work, read
  [plugins/burette-agent/AGENTS.md](plugins/burette-agent/AGENTS.md) and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SergeiNikolenko/Burette](https://github.com/SergeiNikolenko/Burette) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
