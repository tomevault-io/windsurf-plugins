---
trigger: always_on
description: Rules for implementing Figma designs using the Figma MCP server. Covers component organization, styling conventions, design tokens, asset handling, and the required Figma-to-code workflow for this Flutter/Dart pet health monitoring app.
---


# Pet Circle — Figma Design System Rules

## Project Overview

Pet Circle is a **Flutter/Dart** cross-platform app (iOS, Android, Web, Desktop) for pet health monitoring. It uses a **neumorphic design system** with centralized design tokens aligned to Figma variables. Typography is powered by **Google Fonts (Inter)**.

## Figma MCP Integration Rules

These rules define how to translate Figma inputs into code for this project and must be followed for every Figma-driven change.

### Required Flow (do not skip)

1. Run `get_design_context` first to fetch the structured representation for the exact node(s)
2. If the response is too large or truncated, run `get_metadata` to get the high-level node map, then re-fetch only the required node(s) with `get_design_context`
3. Run `get_screenshot` for a visual reference of the node variant being implemented
4. Only after you have both `get_design_context` and `get_screenshot`, download any assets needed and start implementation
5. Translate the output (usually React + Tailwind) into **Flutter widgets and Dart code** using this project's conventions, tokens, and architecture
6. Validate against Figma for 1:1 look and behavior before marking complete

### Implementation Rules

- IMPORTANT: Treat the Figma MCP output (React + Tailwind) as a **representation of design and behavior**, not as final code
- IMPORTANT: Translate all React components to **Flutter StatelessWidget / StatefulWidget** classes
- IMPORTANT: Replace Tailwind utility classes with the project's design tokens (`AppColors`, `AppSpacing`, `AppRadii`, `AppTextStyles`, `AppShadows`)
- Reuse existing widgets from `lib/widgets/` instead of duplicating functionality
- Use the project's color system, typography scale, and spacing tokens consistently
- Respect existing routing (`app_routes.dart`), state management, and data-fetch patterns
- Strive for 1:1 visual parity with the Figma design
- Validate the final UI against the Figma screenshot for both look and behavior

---

## Component Organization

### Directory Structure

```
lib/
├── widgets/          # Reusable UI components (buttons, cards, inputs, etc.)
├── screens/          # Screen-level components, organized by feature
│   ├── auth/
│   ├── dashboard/
│   ├── measurement/
│   ├── medication/
│   ├── messages/
│   ├── onboarding/
│   ├── pet_detail/
│   ├── settings/
│   └── trends/
├── models/           # Data models (Pet, User, Measurement, Medication, etc.)
├── stores/           # ChangeNotifier stores (pet, measurement, note, medication, etc.)
├── services/         # Business logic (auth_service, user_service)
├── providers/        # State providers
├── theme/            # Design tokens and theme configuration
│   ├── app_theme.dart    # Colors, spacing, typography, shadows, radii
│   └── app_assets.dart   # Asset path constants
├── data/             # Mock/demo data
└── l10n/             # Localization files (English, Hebrew)
```

### Placement Rules

- IMPORTANT: Place new **reusable UI components** in `lib/widgets/`
- IMPORTANT: Place new **screen components** in `lib/screens/<feature>/`
- Place new **data models** in `lib/models/`
- Place new **services** in `lib/services/`
- Place new **asset constants** in `lib/theme/app_assets.dart`

### Existing Widgets (check before creating new ones)

| Widget | File | Purpose |
|--------|------|---------|
| `PrimaryButton` | `lib/widgets/primary_button.dart` | Full-width button (filled/outlined variants) |
| `RoundIconButton` | `lib/widgets/round_icon_button.dart` | Circular icon button |
| `NeumorphicCard` | `lib/widgets/neumorphic_card.dart` | Card with neumorphic shadows |
| `LabeledTextField` | `lib/widgets/labeled_text_field.dart` | Text field with label |
| `BreedSearchField` | `lib/widgets/breed_search_field.dart` | Searchable breed dropdown (148 breeds, live filter) |
| `AppDropdown` | `lib/widgets/app_dropdown.dart` | Label + dropdown selector with chevron |
| `SettingsRow` | `lib/widgets/settings_row.dart` | Settings row (icon + title + trailing) |
| `AppHeader` | `lib/widgets/app_header.dart` | Header with avatar, pet switcher chip, notification bell |
| `BottomNavBar` | `lib/widgets/bottom_nav_bar.dart` | Tab bar: Home, Trends, Measure, Medication |
| `StatusBadge` | `lib/widgets/status_badge.dart` | Colored status badge |
| `TogglePill` | `lib/widgets/toggle_pill.dart` | Pill-shaped toggle |
| `UserAvatar` | `lib/widgets/user_avatar.dart` | User avatar with fallback |
| `DogPhoto` | `lib/widgets/dog_photo.dart` | Pet photo with fallback |
| `AppImage` | `lib/widgets/app_image.dart` | Image with error handling |
| `OnboardingShell` | `lib/widgets/onboarding_shell.dart` | Onboarding layout with Back/Next buttons |

---

## Design Tokens

All tokens are centralized in `lib/theme/app_theme.dart`. IMPORTANT: Never hardcode colors, spacing, or typography values — always use the token classes.

### Colors — `AppColors` (9 Figma primitive tokens)

```dart
AppColors.white       // #FFFFFF — backgrounds, card surfaces
AppColors.offWhite    // #F8F1E7 — warm background, scaffold

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HiLuLiT/pet-circle](https://github.com/HiLuLiT/pet-circle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
