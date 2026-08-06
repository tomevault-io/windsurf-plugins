---
trigger: always_on
description: This file defines the working rules for Codex and other AI agents in this project. Follow these rules before changing code.
---

# AGENTS.md

This file defines the working rules for Codex and other AI agents in this project. Follow these rules before changing code.

## Fixed Stack

SkyMusicPlay Lite must use this stack:

- Tauri v2
- React
- TypeScript
- Rust
- Vite
- Git and GitHub
- Windows first

Do not replace this stack unless the human user explicitly changes the project direction.

## UI Refresh Rules

- This project is gradually introducing Tailwind CSS, Radix UI, and lucide-react.
- Do not rewrite the whole UI at once.
- Do not remove `App.css` until the migration is explicitly finished.
- Do not remove `iconfont.css` until every icon usage has been replaced and verified.
- New icons should use `lucide-react`.
- Do not add new font icon dependencies.
- Do not add new handwritten inline SVG icons unless there is no suitable `lucide-react` icon and the user approves it.
- Existing iconfont and handwritten SVG icons should be migrated gradually to `lucide-react`.
- Preserve existing icon wrapper class names during icon migration unless a later cleanup stage explicitly removes them.
- Do not remove `iconfont.css` until a dedicated cleanup stage confirms there are no remaining iconfont usages.
- Lucide icon migration must not re-enable Tailwind startup integration.
- Use Radix UI for interactive primitives such as Dialog, AlertDialog, DropdownMenu, Toast, Tooltip, Slider, Progress, and Popover.
- Do not install the `radix-ui` umbrella package during the UI refresh.
- Add Radix primitives only when a stage actually uses them, using specific packages such as `@radix-ui/react-dialog`, `@radix-ui/react-dropdown-menu`, `@radix-ui/react-tooltip`, `@radix-ui/react-toast`, `@radix-ui/react-popover`, `@radix-ui/react-slider`, or `@radix-ui/react-progress`.
- Do not keep unused UI dependencies in `package.json` because they can increase install, build, and dev startup cost.
- Use Tailwind CSS for new UI styling, but preserve existing CSS during the gradual migration.
- UI polish should prefer `App.css` and existing class-level changes before component rewrites.
- During the early migration, do not use the full `@import "tailwindcss";` in the existing app styles. Use the no-Preflight Tailwind import until the user explicitly approves enabling Preflight.
- During early Alpha UI Refresh stages, Tailwind files and foundation components may exist without enabling the Tailwind Vite plugin in the active app startup path. Only enable the Tailwind Vite plugin when a stage actually migrates visible UI to Tailwind and the startup cost is accepted by the user.
- If enabling a UI tool noticeably slows startup compared with the stable version, prefer deferring active integration until the tool is actually used by visible UI.
- New shared UI building blocks should live in `src/components/ui/`.
- Shared className helpers should live in `src/lib/cn.ts`.
- Do not use `clsx`, `tailwind-merge`, or new styling dependencies unless the user explicitly approves them.
- UI foundation components must not be wired into existing screens until a later stage explicitly asks for it.
- Do not introduce Mantine, shadcn/ui CLI, or Radix Themes unless the user explicitly approves it.
- Controlled text inputs must update their value synchronously.
- Do not wrap controlled text input value updates in `startTransition`.
- Do not manually build input text from keydown events.
- Do not block IME composition events.
- Search input and other text inputs must support Chinese IME correctly.
- Drag-and-drop score import must reuse `handleImportScoreFiles`; do not duplicate parsing, decryption, or import logic in drag handlers, and do not add recursive directory import unless explicitly requested.
- Built-in score parser changes must stay compatible with the index generator and must run a regression test that parses every entry in `public/builtin-scores/index.json` against its source file.
- Built-in lazy loading must parse only the indexed song entry, and the runtime parser must accept the same legacy compatibility cases as the index generator.
- Failed built-in score loads must remain retryable; do not permanently cache transient fetch or parse failures.
- Built-in score files under `public/builtin-scores/scores` must be strict non-empty JSON arrays; do not add runtime recovery for corrupted assets unless explicitly requested.
- The runtime built-in index sanitizer must accept safe uppercase file names emitted by the generator while still rejecting path traversal, nested paths, spaces, hidden files, and unsupported extensions.
- Built-in score tests must cover raw asset JSON validity, generator/runtime index consistency, and indexed song lazy-load parsing.
- Locate-current-score behavior must use `librarySong.id`, not only an array index.
- Trigger library DOM scrolling through explicit locate requests, not on every selection change; import auto-scroll must reuse the same locate request mechanism.
- The library's main scroll container is `.app-layout`, not `window`.
- Search playback context must include all current search result IDs, not only the clicked song; repeat-all must cycle within that captured context.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Whitewind0987/sky-music-play-lite](https://github.com/Whitewind0987/sky-music-play-lite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
