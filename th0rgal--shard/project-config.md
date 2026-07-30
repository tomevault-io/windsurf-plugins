---
trigger: always_on
description: If you update this file, also update `.codex/CODEX.md` and `.cursor/rules/context.mdx` to keep contexts aligned.
---

# Shard Launcher

## Sync note
If you update this file, also update `.codex/CODEX.md` and `.cursor/rules/context.mdx` to keep contexts aligned.

## Overview
Shard is a minimal, clean, CLI-first Minecraft launcher focused on stability, reproducibility, and low duplication.

| Directory | Tech Stack | Purpose |
|-----------|------------|---------|
| `launcher/` | Rust | Core library + CLI (profiles, store, downloads, launching) |
| `desktop/` | Tauri 2, React, TypeScript, Vite | Desktop application UI |
| `desktop/src-tauri/` | Rust | Tauri backend (bridge to core library) |
| `web/` | Next.js 16, React 19, Nextra | Website + documentation |
| `companion/` | (placeholder) | Future companion mode |

## Philosophy
- **Single source of truth**: profiles are declarative manifests; instances are derived artifacts.
- **Deduplication first**: mods and packs live in a content-addressed store (SHA-256); profiles only reference hashes.
- **Stable + boring**: plain JSON on disk, predictable layout, no magic state.
- **Replaceable parts**: authentication, Minecraft data, and profile management are isolated modules.
- **CLI-first**: everything is designed to be scripted and composed.

## Architecture (core concepts)
- **Profiles** (`profiles/<id>/profile.json`): manifest for version + mod/pack selection + runtime flags.
- **Templates** (`templates/<id>/template.json`): reusable profile configurations (Fabric, Forge, etc.).
- **Stores** (`store/*/sha256/`): content-addressed blobs for mods, resourcepacks, shaderpacks.
- **Library** (global content cache): deduplicated content across all profiles.
- **Instances** (`instances/<id>/`): launchable game dirs (symlinked mods/packs + overrides).
- **Minecraft data** (`minecraft/`): versions, libraries, assets, natives.
- **Accounts** (`accounts.json`): multiple Microsoft accounts with refresh + access tokens.

## Code map (Rust - launcher/)
- `launcher/src/lib.rs`: core library entry point, re-exports all modules.
- `launcher/src/main.rs`: CLI entry point with subcommands.
- `launcher/src/profile.rs`: profile management and serialization.
- `launcher/src/template.rs`: profile templates for quick setup.
- `launcher/src/store.rs`: content-addressed store operations.
- `launcher/src/content_store.rs`: unified content store abstraction.
- `launcher/src/library.rs`: global library/content management, enrichment from profiles, unused content detection.
- `launcher/src/instance.rs`: instance materialization from profiles.
- `launcher/src/minecraft.rs`: version/library/asset downloads, loader version fetching (Fabric, Forge, Quilt, NeoForge).
- `launcher/src/modrinth.rs`: Modrinth API client for mod search/install.
- `launcher/src/curseforge.rs`: CurseForge API client for mod search/install.
- `launcher/src/ops.rs`: higher-level operations (download, install, launch).
- `launcher/src/auth.rs`: Microsoft OAuth device code flow.
- `launcher/src/accounts.rs`: account storage + selection.
- `launcher/src/skin.rs`: Minecraft skin fetching and upload.
- `launcher/src/java.rs`: Java runtime detection and management.
- `launcher/src/config.rs`: global configuration handling.
- `launcher/src/paths.rs`: data path helpers.
- `launcher/src/logs.rs`: logging infrastructure.
- `launcher/src/updates.rs`: update checking functionality.
- `launcher/src/util.rs`: shared helpers.

## Code map (Desktop - desktop/)
- `desktop/src/App.tsx`: main application component, routing, modal management.
- `desktop/src/store/index.ts`: Zustand state management.
- `desktop/src/styles.css`: design tokens, theme variables, component styles.
- `desktop/src/components/Sidebar.tsx`: navigation sidebar with profile selector, drag-and-drop folders.
- `desktop/src/components/ProfileView.tsx`: profile details, content management, launch controls.
- `desktop/src/components/StoreView.tsx`: Modrinth/CurseForge mod browser with search and install.
- `desktop/src/components/LibraryView.tsx`: global content library browser with collapsible search and sticky details panel.
- `desktop/src/components/AccountView.tsx`: account details, skin management, skin URL import, cape preview.
- `desktop/src/components/AccountsView.tsx`: account list and switching.
- `desktop/src/components/LogsView.tsx`: live game log viewer.
- `desktop/src/components/SettingsView.tsx`: application settings with storage cleanup section.
- `desktop/src/components/SkinViewer.tsx`: 3D Minecraft skin and cape renderer.
- `desktop/src/components/SkinThumbnail.tsx`: compact skin preview thumbnail component.
- `desktop/src/components/ContentItemRow.tsx`: compact content item display with platform links.
- `desktop/src/components/PlatformIcon.tsx`: Modrinth/CurseForge/Local platform icons.
- `desktop/src/components/Modal.tsx`: base modal component with close button.
- `desktop/src/components/modals/`: modal dialogs for various actions.
- `desktop/src/components/modals/PurgeStorageModal.tsx`: unused content detection and cleanup.
- `desktop/src/components/modals/ProfileJsonModal.tsx`: profile JSON viewer with syntax highlighting and copy button.

## Code map (Web - web/)
- `web/app/page.tsx`: homepage with launcher preview hero and rotating taglines.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Th0rgal/shard](https://github.com/Th0rgal/shard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
