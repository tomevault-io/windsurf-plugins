---
trigger: always_on
description: Entry point for coding agents working on KeeForge. This file is intentionally brief; most useful guidance now lives in folder-local `README.md` files next to the code.
---

# AGENTS.md

Entry point for coding agents working on KeeForge. This file is intentionally brief; most useful guidance now lives in folder-local `README.md` files next to the code.

## High level guidance

- Do not preserve backward compatibility unless required by the existing install base. Remove obsolete paths instead of adding compatibility layers, fallbacks, or migrations.
- Choose the simplest implementation that fully meets the current requirements. Avoid speculative abstractions, configuration, and indirection.
- Grow the system in layers. Start from the smallest version that works end to end, and add each new capability on top of a product that already works. Never trade a working product for unfinished complexity.
- Keep components modular and concerns clearly separated.
- Prefer established, well-maintained libraries when they reduce overall complexity or improve reliability. Do not reimplement common functionality without a clear reason.
- Lean on the dependencies already in the project before writing your own implementation or adding packages. Do not assume a library lacks a capability without checking its documentation and types.
- Make architectural decisions for the long term. Do not accept a stopgap that only works for now and is meant to be replaced later.

## Project Snapshot

- Native iOS KeePass manager for KDBX 4.x databases; also reads KDBX 3.1 (read-only)
- Swift 6, SwiftUI, iOS 18+ / macOS 14+, `@Observable`, strict concurrency
- XcodeGen build graph: edit `project.yml`, then regenerate `KeeForge.xcodeproj`
- Main targets: `KeeForge`, `KeeForgeMac`, `KeeForgeAutoFill`, `KeeForgeMacAutoFill`, `KeeForgeTests`, `KeeForgeMacTests`, `KeeForgeUITests`, `KeeForgeMacUITests`. The macOS targets are ON HOLD and must not ship (see CHANGELOG.md's "## macOS App" section).

## Open The Local Doc First

- `KeeForge/README.md` — app-target map and cross-cutting flows
- `KeeForge/App/README.md` — app lifecycle, root navigation, scene handling
- `KeeForge/Models/README.md` — parser, writer, edit-draft, and core data-model guidance
- `KeeForge/Services/README.md` — storage, local save, cloud sync, keychain, bookmarks, device integrations
- `KeeForge/Services/{AppSupport,AutoFill,Cloud,Persistence,Security}/README.md` — per-subfolder service maps and constraints
- `KeeForge/ViewModels/README.md` — list, unlock, save, search, sort, and TOTP state ownership
- `KeeForge/Views/README.md` — screen ownership and UI/testing conventions
- `KeeForge/Extensions/README.md` — shared Swift extension helpers
- `KeeForge/Resources/README.md` — string catalogs, assets, and resource conventions
- `KeeForgeMac/README.md` — macOS target constraints
- `AutoFillExtension/README.md` — extension constraints and shared-source notes
- `KeeForgeTests/README.md` — unit-test map and selection rules
- `KeeForgeUITests/README.md` — XCUITest workflow and flake-avoidance guidance
- `KeeForgeMacUITests/README.md` — macOS XCUITest workflow
- `TestFixtures/README.md` — bundled databases, passwords, and key files
- `Vendor/KeeForgeTwofish/README.md` — vendored Twofish cipher package
- `ci_scripts/README.md` — Xcode Cloud bootstrap and `run_kdbx_compatibility_gate.sh`, the required local release gate
- `scripts/README.md` — local dev tooling
- `.github/AGENTS.md` — CI workflow gating map (named `AGENTS.md` because GitHub renders a `.github/README.md` as the repo front-page README)
- `docs/README.md` — historical archive of past design specs, audits, and notes (may not match current code), except `docs/macos-security-notes.md`, a living doc kept truthful with the code

## Agent Orchestration

If you are a very powerful model like Fable/Opus/GPT 5.6 Sol, feel free to delegate implementation and test to sub-agents with appropriate models.

Repo skills live in `.agents/skills/` (`release`, `spec-creator`, `publish-app-store-version`), symlinked under `.claude/skills/`; `release` defines the release-branch → TestFlight soak → App Store flow (`release/{major}.{minor}` branches, `rc/{version}-b{build}` candidate tags, `v{version}` as a record of the shipped build).

## Repo-Wide Rules

### Coding Styles

- Use `@Observable`, not `ObservableObject` / `@Published`.
- Use `NavigationStack` + `NavigationPath`, not `NavigationView`.
- Keep crypto, parsing, and secret handling off the main thread.
- Treat these `KeeForge/Models/` files as stable core: `KDBXParser.swift`, `KDBX3Parser.swift`, `KDBXWriter.swift`, `KDBXXMLSerializer.swift`, `KDBXCrypto.swift`, `KDBXOuterCipher.swift`, `OpaqueXMLNodes.swift`, `DatabaseDraft.swift`, `EntryEdit.swift`, `Entry.swift`, `Group.swift`, `EncryptedValue.swift`, `TOTPGenerator.swift`. Change them only for real bugs or intentional format/security work, and add focused tests.
- No force unwraps outside tests.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KeeForge/KeeForge](https://github.com/KeeForge/KeeForge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
