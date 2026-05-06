---
trigger: always_on
description: Kivio (formerly KeyLingo through v2.4.4) is a lightweight desktop screen-level AI assistant for macOS and Windows. Its core focus is a small package size and low runtime footprint, providing instant text translation, screenshot translation, and AI-powered visual Q&A through global shortcuts.
---

# Kivio Agent Guidelines

Kivio (formerly KeyLingo through v2.4.4) is a lightweight desktop screen-level AI assistant for macOS and Windows. Its core focus is a small package size and low runtime footprint, providing instant text translation, screenshot translation, and AI-powered visual Q&A through global shortcuts.

## Tech Stack & Architecture

- **Frontend**: React 18 + TypeScript + Vite + TailwindCSS v4 (ESM)
- **Backend**: Rust + Tauri v2
- **Package Manager**: npm (lockfile: `package-lock.json`)
- **Build Targets**: macOS (DMG), Windows (MSI + NSIS)

The app uses a classic Tauri split architecture: a single-page React frontend invokes Rust backend commands via Tauri's `invoke` bridge. The backend handles global shortcuts, window management, system tray, screenshot capture, HTTP API calls, and settings persistence.

## Project Directory Structure

```
src/                          # Frontend React + TypeScript source
  main.tsx                    # React entry point (mounts to #root)
  App.tsx                     # Root component: switches views by URL hash
  Settings.tsx                # Settings page main component
  Lens.tsx                    # Lens (screenshot translation + AI vision Q&A)
  api/tauri.ts                # Tauri bridge: all invoke calls & event listeners centralized
  settings/                   # Settings UI helper modules
    components.tsx            # Reusable form components (Toggle, Select, Input, etc.)
    i18n.ts                   # Internationalization strings & language utilities
    utils.ts                  # Settings page utilities (hotkey formatting, platform detection)
  index.css                   # Global styles (Tailwind imports, scrollbar, transparent window)
  App.css                     # Component-specific styles

src-tauri/
  src/                        # Rust backend source
    main.rs                   # App main entry: state, commands, hotkeys, tray, API calls
    lens.rs                   # Lens window enumeration and screenshot capture
    screenshot.rs             # Screenshot capture utilities and temp file cleanup
    sck.rs                    # ScreenCaptureKit integration (macOS 14+)
    settings.rs               # Settings data structures, serialization, migration
    windows.rs                # Window creation & retrieval helpers
    utils.rs                  # Language detection, timestamps, etc.
  tauri.conf.json             # Tauri app config (windows, bundling, icons)
  Cargo.toml                  # Rust dependencies
  icons/                      # App icon assets

public/                       # Static assets (icons, SVGs)
.github/workflows/release.yml # GitHub Actions automated release workflow
```

## Core Module Details

### Frontend View Routing (Hash-based)

`App.tsx` switches modes via `window.location.hash`, mapping to different windows/views:

- `''` or `'translator'`: Main translator window (392x152, floating input)
- `'settings'`: Settings page (640x520)
- `'lens'`: Lens window (600x72 select mode / 600x420 answering mode, floating)

### Rust Backend State (`AppState`)

Defined in `main.rs`, the global shared state includes:

- `settings: RwLock<Settings>` — App settings (multiple readers, single writer)
- `explain_images: Mutex<HashMap<String, PathBuf>>` — Temporary image map for Lens
- `current_explain_image_id: Mutex<Option<String>>` — Currently active Lens image
- `lens_busy: AtomicBool` — Concurrency guard for Lens operations
- `explain_stream_generation: AtomicU64` — Stream cancellation token
- `key_cooldowns: Mutex<HashMap<(String, usize), Instant>>` — API key failover cooldown tracking
- `active_key_idx: Mutex<HashMap<String, usize>>` — Currently active API key index per provider
- `http: Client` — Shared HTTP client for API calls

### Settings Persistence & Security

- Settings body is stored in Tauri Store as `settings.json`
- **API Keys are stored directly in `settings.json`** (as of v2.4.0); the `keyring` crate is only used for one-time migration from legacy keyring storage
- On load, `sanitize_settings` cleans data and migrates legacy single-provider configs to the multi-provider system
- `settings.rs` contains full defaults, normalization logic (hotkeys, prompts), and keyring migration helpers

### Multi-Provider Routing

The app supports separate OpenAI-compatible providers for each feature:

- **Text Translation**: `translator_provider_id` + `translator_model`
- **Screenshot Translation / OCR**: `screenshot_translation.provider_id` + `model`
- **Lens (AI Vision)**: `lens.provider_id` + `model`

Each `ModelProvider` has `id`, `name`, `base_url`, `api_keys`, `available_models`, and `enabled_models`.

### API Key Failover

- Each provider stores multiple API keys (`api_keys: string[]`)
- Primary key is `api_keys[0]`, backups follow
- Backend `send_with_failover` automatically rotates to next key on 401/402/403/429 or quota/billing/balance errors
- Failed key enters 60-second cooldown before retry
- **Test Connection intentionally only probes the primary key**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZMGID/kivio](https://github.com/ZMGID/kivio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
