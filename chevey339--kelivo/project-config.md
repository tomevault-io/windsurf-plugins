---
trigger: always_on
description: > Kelivo is a cross-platform Flutter LLM chat client (Android / iOS / macOS / Windows / Linux).
---

# AGENTS.md

> Kelivo is a cross-platform Flutter LLM chat client (Android / iOS / macOS / Windows / Linux).
> This file defines hard constraints for AI-assisted development. Predictable, auditable, repeatable.

## 1. Repository Facts

- This is a Flutter app repository. Root `pubspec.yaml` declares `sdk: ^3.8.1` with `flutter.generate: true`.
- Main code lives in `lib/`, tests in `test/`. Local path dependencies exist:
  - `dependencies/mcp_client`
  - `dependencies/tray_manager/packages/tray_manager`
  - `dependencies/flutter_tts`
  - `dependencies/flutter-permission-handler/permission_handler_windows`
- Localization is driven by `l10n.yaml`:
  - `arb-dir: lib/l10n`
  - `template-arb-file: app_en.arb`
  - `output-localization-file: app_localizations.dart`
  - `untranslated-messages-file: desiredFileName.txt`
- There are exactly 4 ARB files that must stay in sync:
  - `lib/l10n/app_en.arb`
  - `lib/l10n/app_zh.arb`
  - `lib/l10n/app_zh_Hans.arb`
  - `lib/l10n/app_zh_Hant.arb`
- The following are generated or build artifacts. Never hand-edit them:
  - `lib/l10n/app_localizations*.dart`
  - `lib/core/models/*.g.dart`
  - All other generated logic must go through commands, not manual edits
  - `.dart_tool/**`
  - `build/**`
- The package name is `Kelivo`. Existing imports use `package:Kelivo/...` everywhere. Do not "normalize" the package name.
- Top-level platform entry is `_selectHome()` in `lib/main.dart`:
  - macOS / Windows / Linux -> `DesktopHomePage`
  - Android / iOS -> `HomePage`
- Desktop is NOT "mobile stretched wider":
  - `lib/desktop/desktop_home_page.dart` is the desktop app shell: nav rail, window title bar, hotkeys, desktop settings, translate/storage tabs, and other desktop-level interactions
  - `lib/desktop/desktop_chat_page.dart` is the desktop chat entry, currently reusing `HomePage`
  - `lib/features/home/pages/home_page.dart` only handles the shared chat page, switching internally by width to `home_mobile_layout.dart` or `home_desktop_layout.dart`
  - Therefore "wide/tablet layout" != "desktop app entry". Do not conflate them.
- Reusable UI primitives live in these locations:
  - `lib/shared/widgets/ios_tactile.dart`: `IosIconButton`, `IosCardPress`
  - `lib/shared/widgets/ios_tile_button.dart`
  - `lib/shared/widgets/ios_switch.dart`
  - `lib/shared/widgets/ios_checkbox.dart`
  - `lib/shared/widgets/ios_form_text_field.dart`
  - `lib/desktop/widgets/desktop_select_dropdown.dart`
  - `lib/shared/dialogs/**`
  - `lib/shared/responsive/**`
- Theme and dynamic color follow the repo as-is:
  - `lib/theme/**` is the single source of truth for theming and tokens
  - Android dynamic color is only enabled per-platform in `main.dart`. Do not extrapolate Android visual or interaction rules to desktop.

## 2. Working Style

- Communicate in Chinese throughout. Stay focused on the current task. No vague suggestions.
- Facts first. All conclusions must be based on current code, config, tests, build scripts, or git state. No guessing.
- Debug-first. Never add silent degradation, swallowed errors, hidden fallback paths, or fake success branches just to "make it run".
- Default to KISS / YAGNI:
  - Use the most direct, most verifiable approach first.
  - Do not pre-plant extra layers, empty abstractions, or config switches for "architectural completeness" or "might need it later".
- SOLID is a tool, not a goal:
  - Only split responsibilities when it genuinely reduces coupling and improves readability.
  - Do not shatter simple logic into a chain of tiny files just for formal layering.
- Minimal closed loop. Make only the minimum change needed for the current task. Do not fix unrelated issues on the side.
- Parallel context gathering by default during exploration:
  - Independent file reads, `rg` searches, `git status`, config checks, and log inspections should be batched in a single parallel round.
  - Do not serialize what can be parallelized.
- For complex tasks, write a brief Mini Control Contract before touching code:
  - `Primary Setpoint`: What exactly must be achieved
  - `Acceptance`: What command, test, or behavior proves it
  - `Guardrails`: What must not break as a side effect
  - `Boundary`: Which files/modules are in scope
  - `Risks`: 1 to 3 key risks

## 3. Mandatory Rules

### 3.1 All User-Visible Text Must Be Localized

- No user-visible text may be hardcoded in Dart UI code. This includes but is not limited to:
  - Page titles
  - Button labels
  - `SnackBar` / `Dialog` / `Tooltip` content
  - `semanticLabel`
  - Notification text
  - Tray menu text
- When adding or modifying user-visible strings, ALL 4 files must be updated simultaneously:
  - `lib/l10n/app_en.arb`
  - `lib/l10n/app_zh.arb`
  - `lib/l10n/app_zh_Hans.arb`
  - `lib/l10n/app_zh_Hant.arb`
- Updating only `app_en.arb` or only `app_zh.arb` and stopping is not acceptable.
- Placeholders, plurals, selects, and `@key` metadata must be consistent across all four ARB files.
- New keys follow the existing camelCase convention with a feature prefix. Do not use context-free short names like `title1` or `labelText`.
- After ARB changes, run:

```bash
flutter gen-l10n
```

- Never hand-edit `lib/l10n/app_localizations.dart` or `lib/l10n/app_localizations_*.dart`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Chevey339/kelivo](https://github.com/Chevey339/kelivo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
