---
trigger: always_on
description: **ALWAYS respond in Hebrew!** This includes:
---

# AI Agent Guidelines for Otzaria

## CRITICAL: Communication Language
**ALWAYS respond in Hebrew!** This includes:
- All answers and explanations
- Your thinking process
- Error messages and debugging info
- Only code/comments can be in English when appropriate

## Mandatory Workflow
1. **Plan** - Create detailed action plan before execution
2. **Execute** - Step by step until completion
3. **Validate** - Run `flutter analyze` after EVERY change
4. **Fix ALL errors before proceeding to next step**
5. **Never skip validation - errors compound quickly!**

## Bug Fix Workflow (MANDATORY)

**Primary rule: Investigate first, ask only if you truly must.**

Before writing any fix, perform the following steps **on your own** without asking the user:

1. **Understand the symptom** - What did the user report? If critical details are missing that are needed to *execute* the fix (not to analyze) - ask everything **in a single message**.
2. **Investigate git** - Run `git log --oneline -20` and check commits that touched relevant code.
3. **Read the code** - Read the code before proposing any fix. Don't assume, know.
4. **Identify the root cause** - If found, explain to the user what caused the bug before fixing it.

### Decision Tree

```
User reports bug
       │
       ▼
  Investigate first:
  git log + read code
       │
       ▼
 Root cause found?
   ┌───┴───┐
  YES      NO
   │           │
   ▼           ▼
Apply MINIMAL  Ask user ONE message
fix & explain  with ALL missing info
               then investigate again
```

### Fix Philosophy - CRITICAL

**Bug fix ≠ adding code!**

- **FIRST** - try to **remove** or **revert** code that caused the bug
- **SECOND** - try to **change** existing logic minimally
- **LAST RESORT** - add new code, only if truly necessary
- Adding more code to work around a bug = introducing future bugs

### Red Flags - Stop and Ask

If you find yourself about to:
- Add a `try/catch` to silence an error → find out *why* the error occurs first
- Add a null check that "shouldn't be needed" → find out *why* it's null
- Add a workaround flag/boolean → reconsider the root cause
- Write more than ~15 lines to fix a single bug → something is wrong, reassess

## Architecture

### Design Patterns
- **BLoC Pattern** - State management (every feature needs: bloc/event/state)
- **Repository Pattern** - Separates data access from business logic
- **Provider** - For dependency injection across the app

### Feature Structure (MUST follow)
```
lib/feature_name/
├── bloc/
│   ├── feature_bloc.dart      # Business logic
│   ├── feature_event.dart     # User actions/events
│   └── feature_state.dart     # UI states
├── models/
│   └── feature_model.dart     # Data models
├── repository/
│   └── feature_repository.dart # Data layer
└── view/
    ├── feature_screen.dart    # Main screen
    └── widgets/               # Feature-specific widgets
```

### Key Code Locations
```
lib/
├── data/repository/
│   └── books_repository.dart          # Central books management
├── models/
│   ├── books.dart                     # Book model (title, path, etc)
│   └── app_model.dart                 # Main app state
├── widgets/
│   ├── rtl_text_field.dart           # RTL text input (USE THIS!)
│   └── [other shared widgets]
├── core/
│   └── scaffold_messenger.dart        # UiSnack for messages
├── search/
│   ├── bloc/                          # Search state management
│   └── search_repository.dart         # Search engine
├── settings/
│   ├── settings_repository.dart       # App settings
│   └── bloc/
├── bookmarks/repository/              # Bookmarks system
├── history/                           # Reading history
├── personal_notes/                    # User notes feature
├── pdf_book/                          # PDF viewer screens
├── text_book/                         # Text viewer screens
└── utils/
    └── open_book.dart                 # Book opening logic
```


## MANDATORY UI Components

### 1. Icons - ONLY from `fluentui_system_icons`
```dart
import 'package:fluentui_system_icons/fluentui_system_icons.dart';
import 'package:otzaria/widgets/misc/rtl_icon.dart';

// Regular icon (symmetric, no RTL flipping needed):
Icon(FluentIcons.search_24_regular)
Icon(FluentIcons.settings_24_regular)

// RtlIcon — ONLY for icons registered in lib/widgets/misc/rtl_icon.dart:
RtlIcon(FluentIcons.book_24_filled)              // in _flippableIcons
RtlIcon(FluentIcons.arrow_left_24_regular)        // in _fluentMirrorMap — auto-mirrors to arrow_right in RTL
RtlIcon(FluentIcons.chevron_right_24_regular)     // in _fluentMirrorMap
```

**When to use `RtlIcon` vs `Icon`:**

| Icon is registered in `rtl_icon.dart`? | Use |
|---|---|
| Yes (in `_fluentMirrorMap`, `_materialMirrorMap`, or `_flippableIcons`) | `RtlIcon(...)` |
| No | `Icon(...)` — plain, no wrapper |

**Icons currently registered in `lib/widgets/misc/rtl_icon.dart`:**

*`_fluentMirrorMap` (swaps to opposite-direction variant in RTL):*
- `chevron_right/left_24/20/16_regular`
- `arrow_right/left_24_regular`, `arrow_right/left_24_filled`
- `panel_left/right_24_regular`, `panel_left/right_24_filled`
- `text_align_right/left_24_regular`

*`_materialMirrorMap` (Material icons, swaps in RTL):*
- `arrow_forward/back`, `arrow_forward/back_ios`
- `arrow_right/left`, `chevron_right/left`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Otzaria/otzaria](https://github.com/Otzaria/otzaria) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-02 -->
