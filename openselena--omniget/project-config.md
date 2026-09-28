---
trigger: always_on
description: Desktop download manager built with Tauri 2.0 (Rust backend) + SvelteKit (frontend). Modern design principles (2025-2026): clarity over features, immediate feedback, and universal accessibility. Monorepo: `src-tauri/` is Rust, `src/` is SvelteKit + TypeScript. Run `cargo tauri dev` to start.
---

# OmniGet

Desktop download manager built with Tauri 2.0 (Rust backend) + SvelteKit (frontend). Modern design principles (2025-2026): clarity over features, immediate feedback, and universal accessibility. Monorepo: `src-tauri/` is Rust, `src/` is SvelteKit + TypeScript. Run `cargo tauri dev` to start.

## Commands
```bash
pnpm install          # install frontend deps
pnpm dev              # SvelteKit dev server only
cargo tauri dev       # full app (Rust + frontend)
cargo check           # typecheck Rust without building
pnpm check            # svelte-check + tsc
```

## Tech Stack

- **Backend:** Rust, Tauri 2.x, tokio, reqwest, serde, sqlx (SQLite), chromiumoxide
- **Frontend:** SvelteKit 2, Svelte 5 (runes: `$state`, `$derived`, `$effect`, `$props`), TypeScript strict
- **Styling:** Scoped CSS with CSS custom properties. NO Tailwind. NO CSS-in-JS.
- **Icons:** `@tabler/icons-svelte` — import individually: `import IconDownload from "@tabler/icons-svelte/IconDownload.svelte"`
- **Fonts:** System fonts as default; IBM Plex Mono for code and technical content only
- **i18n:** `sveltekit-i18n` with JSON locale files in `i18n/{lang}/`
- **Bundler:** Vite 5, adapter-static, pnpm as package manager

## Project Layout
src-tauri/src/
commands/       # Tauri IPC commands (invoked from frontend)
platforms/      # Download platform implementations (trait-based plugin system)
traits.rs     # PlatformDownloader trait — all platforms implement this
hotmart/      # Hotmart-specific: auth, api, parser, downloader
core/           # Shared engine: registry, hls_downloader, media_processor, queue
models/         # Data structs: media.rs, download.rs, settings.rs
storage/        # Persistence: config, database (SQLite), cache
src/
routes/         # SvelteKit file-based routing (+page.svelte, +layout.svelte)
components/     # Reusable UI, organized by domain (see Component Organization)
lib/            # Shared logic: stores/, i18n/

## Design System

### Core Principles

**Minimalism with Purpose**: Every element has a function. No decoration.

**Immediate Feedback**: Progress bars show percent + bytes + speed. Errors are explicit ("Missing API key" not "Error").

**Color as Guidance**: Accent for primary actions, error for destructive. Never communicate through color alone (pair with icon + text).

**Accessibility First**: WCAG 2.2 AA baseline. 4.5:1 text contrast, 3:1 focus indicators, all elements keyboard accessible.

### Color Architecture

CSS custom properties exclusively. NEVER hardcode colors. Theme via `[data-theme="dark"]` / `[data-theme="light"]`. Every token defined in both `:root` and `[data-theme="dark"]`.

**Tokens** (see app.css for values): `--primary`, `--secondary`, `--tertiary`, `--accent`, `--success`, `--error`, `--warning`, `--button`, `--button-hover`, `--button-press`, `--button-text`, `--button-stroke`, `--button-elevated`, `--sidebar-bg`, `--sidebar-highlight`, `--content-border`, `--input-border`, `--input-bg`, `--popup-bg`, `--dialog-backdrop`.

**Contrast tokens** for text on colored backgrounds: `--on-primary`, `--on-accent`, `--on-success`, `--on-error`, `--on-button`, `--on-button-elevated`.

**Layout constants**: `--padding: 12px`, `--border-radius: 11px`, `--sidebar-width: 80px`.

### Typography

System fonts for UI, monospace for code only:
- `--font-system: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif`
- `--font-mono: "IBM Plex Mono", "Courier New", monospace`

Headings: `font-weight: 500`, `margin-block: 0`. Scale: h1=24px, h2=20px, h3=16px, h4=14.5px, h5=12px, h6=11px.

Body: buttons=14.5px/500, .label=13px/500, .body=14px/400/1.6, .subtext=12.5px/500, .caption=11.5px/400, code=13px/mono.

### Spacing & Radius

Base unit: `var(--padding)` (12px). Derive: `calc(var(--padding) / 2)` (6px), `calc(var(--padding) * 2)` (24px). Border radius: `var(--border-radius)` (11px).

## Component Patterns

### File Organization
components/
buttons/       # ActionButton, SettingsButton, SettingsToggle, Switcher
dialog/        # DialogContainer, DialogButton, SmallDialog, PickerDialog
hints/         # Contextual hints UI
hotmart/       # Hotmart-specific components
icons/         # Custom SVG icon components (only when tabler doesn't have it)
mascot/        # Loop mascot animations
misc/          # Toggle, Skeleton, Placeholder, SectionHeading, OuterLink
omnibox/       # OmniboxInput, MediaPreview, FormatSelector, QualityPicker, DownloadModeSelector
onboarding/    # First-run onboarding flow
services/      # Platform service components
settings/      # SettingsCategory, SettingsDropdown, SettingsInput
toast/         # Toast notification components

### Button System

Classes: `.button` (default), `.button.elevated`, `.button.active`, `.button.active.color`. Hover via `@media (hover: hover)`. Focus: `outline: var(--focus-ring)` on `:focus-visible` only.

### Custom Controls

- **Toggle**: `<input type="checkbox" role="switch" aria-checked>` with CSS transform animation. RTL-aware via `:dir(rtl)`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenSelena/omniget](https://github.com/OpenSelena/omniget) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-28 -->
