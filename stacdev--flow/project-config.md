---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

**flow_ui** is a chat/assistant UI component library for Flutter — the presentation layer for AI assistant interfaces. It is a plain Flutter package (no codegen) at `packages/flow_ui/`, inside a pub workspace driven by melos. Downstream packages depend on the public API exported from `lib/flow_ui.dart`, so treat it as a compatibility surface.

Two hard constraints shape everything here:

- **One dependency beyond flutter.dev, argued.** `dependencies:` in `pubspec.yaml` holds the Flutter SDK, Flutter's own first-party packages — `material_ui` (Material's home since Flutter 3.47) and its transitive set, all published by flutter.dev — plus `file_selector` (flutter.dev), the plugin behind the composer's attach button. Nothing else, and adding one is a decision rather than a convenience: it must not force configuration on hosts that never touch the feature (this is why `file_selector` and not `image_picker`, which writes a permission, a FileProvider and a Play-services entry into every host's Android manifest), and the PR has to argue it. What the one we have does ask for is documented: the picker needs macOS `com.apple.security.files.user-selected.read-only`. Dev dependencies (`flutter_test`, `flutter_lints`) are fine.
- **The typefaces ship, they are not fetched.** Google Sans and Google Sans Code live in `packages/flow_ui/fonts/` under the SIL Open Font License, declared under `flutter: fonts:` with every cut (sans 400–700, mono 300–800, each upright and italic) and addressed as `package: 'flow_ui'`. Google Sans is a Latin subset — the full cuts are ~1.95 MB each — rebuilt by `packages/flow_ui/tool/subset_fonts.sh`; the scripts it drops fall back to the platform face. Nothing reaches the network for a glyph, so hosts need no INTERNET permission and no macOS network entitlement.
- **Nothing model-facing.** Components render state passed in and report intent out through callbacks. No prompts, schemas, provider/network calls, or any LLM awareness — that belongs to the layers built on top.

The theme, the conversation components (message, thread, streaming text, actions, loading), the composer and its menus, attachments with their preview, suggestions, and the chat surface are implemented; the roadmap below tracks the rest. Message content is modeled as typed parts (`lib/src/models/`) — sealed `FlowMessagePart` subtypes rendered by `FlowMessage`, with `FlowCustomPart` + `FlowCustomPartBuilder` as the extension seam for host-injected content.

## Layout

- Root `pubspec.yaml` is the pub workspace (members under `workspace:`) with the melos scripts; root `analysis_options.yaml` (very_good_analysis) governs `tool/` and the SDK package only.
- `packages/flow_ui/` — the published package: `lib/`, `example/` (the README's chat screen against Gemini), `assets/`, `fonts/` (the bundled typefaces and their OFL texts), `tool/` (`subset_fonts.sh`, not published), its own flutter_lints `analysis_options.yaml` and `.pubignore`.
- `packages/stacflow/` — the StacFlow SDK package (see "SDK package"), with `example/` (the README's chat screen against Gemini; flutter_lints like the flow_ui example).
- `playground/` — the Flow UI Playground: a full Flutter app and workspace member depending on `flow_ui: ^0.4.0`. Use it to demo and manually exercise components (every component has a stage demo, with variant pills and code snippets).
- `docs/` — the Astro site behind flowui.stac.dev. `contracts/` — the SDK wire contract.

## Commands

**Don't write tests for now.** `packages/flow_ui` has no `test/` yet: the component surface is still being reshaped design-first, so tests written now would mostly encode values about to change. Verify a change with `flutter analyze` and by exercising it in the `playground/` app, not by adding a test file. If something seems to genuinely need one, say so and let the user decide.

**Don't write comments unless asked.** No doc comments, file headers or inline explanations in new or edited code; the code and the commit message carry the intent. The one exception is a comment a lint requires (for example `document_ignores` above an `// ignore`), kept to one line. Public API dartdoc is written only when the user asks for it.

From the repo root:

```bash
flutter pub get                    # resolves the whole workspace
dart run melos run analyze         # dart analyze --fatal-infos in every member
dart run melos run format
dart run melos run test            # SDK package tests; flow_ui has none yet
cd packages/flow_ui && flutter analyze && flutter pub publish --dry-run
```

Playground app:

```bash
cd playground
flutter pub get
flutter run -d chrome    # or any device
```

## SDK package


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [StacDev/flow](https://github.com/StacDev/flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
