---
trigger: always_on
description: > shadcn/ui for React Native. Beautiful. Minimal. Yours.
---

# CLAUDE.md — AniUI

> shadcn/ui for React Native. Beautiful. Minimal. Yours.

## CRITICAL RULES — READ FIRST

1. **NEVER install components as npm dependencies.** Components are SOURCE FILES copied into user's project via CLI. Users OWN the code.
2. **NEVER use StyleSheet.create().** ALL styling is NativeWind className only.
3. **NEVER create barrel files (index.ts re-exports).** Each component is standalone.
4. **NEVER use default exports.** Named exports everywhere.
5. **NEVER add a dependency unless 3+ components need it.** Keep node_modules minimal.
6. **NEVER wrap components in unnecessary container Views.** Every element must earn its place.
7. **NEVER use `any` type.** 100% strict TypeScript.
8. **NEVER build for web.** Mobile-only (iOS + Android). If NativeWind renders on web automatically, fine — but don't optimize or test for it.
9. **ONE file per component.** No splitting into multiple files unless absolutely necessary.
10. **Target: each component under 80 lines of code.** If it's longer, simplify it.

## Project Identity

- **Name:** AniUI
- **npm package:** `aniui` (the CLI tool only)
- **License:** MIT
- **Creator:** Anish
- **Tagline:** "Beautiful React Native components. Copy. Paste. Ship."

## Locked Dependency Versions

DO NOT deviate from these versions. They are tested together.

```json
{
  "peerDependencies": {
    "react": ">=18.2.0",
    "react-native": ">=0.76.0",
    "nativewind": ">=4.2.1",
    "tailwindcss": ">=3.4.17",
    "react-native-reanimated": ">=3.10.0",
    "react-native-safe-area-context": ">=4.10.0"
  },
  "dependencies_for_components": {
    "class-variance-authority": "^0.7.1",
    "clsx": "^2.1.1",
    "tailwind-merge": "^2.6.0"
  }
}
```

**Dual SDK Support:** AniUI supports two generations:
- **Expo SDK 54 (v4):** NativeWind v4 + Tailwind v3 + Reanimated v3 (Old + New Architecture)
- **Expo SDK 55 (v5):** NativeWind v5 + Tailwind v4 + Reanimated v4 (New Architecture only)

Components use `className`/`cn()`/`cva()` which work identically on both. The CLI auto-detects the SDK generation and uses the matching templates.

## Supported Platforms

- Expo SDK 54 (NativeWind v4 + Tailwind v3)
- Expo SDK 55 (NativeWind v5 + Tailwind v4)
- Bare React Native CLI 0.76+
- iOS 15+, Android API 24+
- New Architecture: supported on all SDKs
- Old Architecture: supported on SDK 54 and earlier

## Repository Structure

```
aniui/
├── CLAUDE.md
├── LICENSE                    # MIT — copy from https://opensource.org/licenses/MIT
├── README.md
├── .gitignore
│
├── cli/                       # Published to npm as "aniui"
│   ├── package.json
│   ├── tsconfig.json
│   ├── bin/
│   │   └── index.ts           # #!/usr/bin/env node
│   └── src/
│       ├── index.ts
│       ├── commands/
│       │   ├── init.ts
│       │   ├── add.ts
│       │   └── theme.ts
│       ├── registry.ts        # Component name → file + deps mapping
│       └── utils/
│           ├── detect-project.ts
│           ├── file-ops.ts
│           └── logger.ts
│
├── components/                # Source files — copied by CLI into user's project
│   └── ui/
│       ├── button.tsx
│       ├── text.tsx
│       ├── input.tsx
│       ├── card.tsx
│       ├── badge.tsx
│       ├── separator.tsx
│       ├── avatar.tsx
│       ├── alert.tsx
│       ├── switch.tsx
│       ├── checkbox.tsx
│       ├── textarea.tsx
│       ├── label.tsx
│       ├── spinner.tsx
│       ├── progress.tsx
│       ├── radio-group.tsx
│       ├── list.tsx
│       ├── skeleton.tsx
│       ├── accordion.tsx
│       ├── tabs.tsx
│       ├── dialog.tsx
│       ├── alert-dialog.tsx
│       ├── toast.tsx
│       ├── collapsible.tsx
│       ├── tooltip.tsx
│       ├── popover.tsx
│       ├── bottom-sheet.tsx
│       ├── action-sheet.tsx
│       ├── select.tsx
│       ├── date-picker.tsx
│       ├── slider.tsx
│       ├── toggle.tsx
│       ├── toggle-group.tsx
│       ├── drawer.tsx
│       ├── input-otp.tsx
│       ├── table.tsx
│       ├── search-bar.tsx
│       ├── chip.tsx
│       ├── fab.tsx
│       ├── empty-state.tsx
│       ├── dropdown-menu.tsx
│       ├── image.tsx
│       ├── segmented-control.tsx
│       ├── carousel.tsx
│       ├── rating.tsx
│       ├── stepper.tsx
│       ├── banner.tsx
│       ├── calendar.tsx
│       ├── swipeable-list-item.tsx
│       ├── form.tsx
│       ├── password-input.tsx
│       ├── theme-provider.tsx
│       ├── safe-area.tsx
│       ├── header.tsx
│       ├── tab-bar.tsx
│       ├── status-indicator.tsx
│       ├── labeled-separator.tsx
│       ├── masked-input.tsx
│       ├── phone-input.tsx
│       ├── number-input.tsx
│       ├── combobox.tsx
│       ├── progress-steps.tsx
│       ├── timeline.tsx
│       ├── chat-bubble.tsx
│       ├── stat-card.tsx
│       ├── grid.tsx
│       ├── price.tsx
│       ├── refresh-control.tsx
│       ├── infinite-list.tsx
│       ├── pagination.tsx
│       ├── file-picker.tsx
│       ├── connection-banner.tsx
│       ├── typing-indicator.tsx
│       └── image-gallery.tsx
│
├── lib/                       # Shared utils — also copied to user's project
│   └── utils.ts               # cn() helper — THE ONLY utility file
│
├── templates/                 # Copied during `npx @aniui/cli init`
│   ├── global.css
│   ├── tailwind.config.js
│   └── nativewind-env.d.ts
│
└── examples/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anishlp7/aniui](https://github.com/anishlp7/aniui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
