---
trigger: always_on
description: **Flippio** = Tauri 2 desktop app for iOS/Android DB inspect.
---

# Flippio AI Coding Instructions

## Architecture Overview

**Flippio** = Tauri 2 desktop app for iOS/Android DB inspect.
- **Backend**: Rust, SQLx, `idevice*`, `adb`, async commands
- **Frontend**: React + TypeScript, Chakra UI, Zustand, React Query
- **Bridge**: `tauri-api.ts` map Electron-like API to Tauri `invoke()`

## Critical Development Workflows

### Build & Development
```bash
# Development with hot reload (both frontend and backend)
npm run tauri:dev

# Frontend-only development (requires mock API)
npm run dev:renderer

# Production builds with signing
npm run tauri:build:signed  # Uses env vars from .env
```

### Testing Strategy
- Frontend: Vitest + jsdom + mocks in `test-utils/`
- Backend: Cargo test + fixtures in `src-tauri/tests/fixtures/databases/`
- Integration: use `Makefile`
```bash
make test-all          # Full test suite
make quick-test-platforms  # Fast platform-specific tests
make coverage-html     # Generate coverage with llvm-cov
```

### External Dependencies Management
- Flippio bundle iOS `idevice*` tools + Android SDK tools. CI break without them.
- CI builds use `tauri-ci.conf.json`
- macOS universal deps live in `src-tauri/macos-deps/`
- CI config simplified to avoid cross-platform binary issues

## Project-Specific Patterns

### State Management Architecture
```typescript
// Zustand stores with specific responsibilities
appStore.ts              // Global app state and selections
useCurrentDeviceSelection.ts   // Device picker logic
useCurrentDatabaseSelection.ts // Database file management
useTableData.ts          // AG Grid data management
useRowEditingStore.ts    // Side panel editing state
```

### Tauri Command Integration
Rust commands return `DeviceResponse<T>`:
```rust
pub struct DeviceResponse<T> {
    pub success: bool,
    pub data: Option<T>,
    pub error: Option<String>,
}
```

Frontend `tauri-api.ts` map snake_case Rust commands to camelCase APIs:
```typescript
// Maps 'adb:getDevices' -> 'adb_get_devices'
const COMMAND_MAP = { /* ... */ }
```

### Database Connection Patterns
- Rust use `DbConnectionCache` + SQLx pooling
- Frontend always pass `currentDbPath` for multi-DB support
- Test fixtures from `scripts/generate-test-databases.js`

### Device Detection Logic
```typescript
// Device type detection in tauri-api.ts
if (deviceId.match(/^[A-F0-9-]{36,40}$/i)) deviceType = 'iphone-device'
else if (deviceId.match(/^[A-F0-9-]{8,}$/i)) deviceType = 'simulator'
else deviceType = 'android'
```

### Error Handling Conventions
- Backend: `DeviceResponse<T>`
- Frontend: toast notifications
- Logging: Tauri plugin + Sentry in prod

## Component Testing Patterns

### Provider Wrapping
```typescript
// From test-utils/render.tsx
<QueryClientProvider client={testQueryClient}>
  <Provider>  {/* Chakra UI */}
    {component}
</QueryClientProvider>
```

### Store Mocking
```typescript
beforeEach(() => {
  useAppStore.getState().setDevices([])
  // Reset other store state...
})
```

### AG Grid Testing
```typescript
vi.mock('ag-grid-react', () => ({ AgGridReact: MockAgGridReact }))
```

## Integration Points & Cross-Component Communication

### Device → App → Database Flow
1. `useDevices` fetch all devices
2. Device select triggers `useApplications`
3. App select triggers `useDatabaseFiles`
4. DB select opens pool, enables table browse

### File Upload/Export Flow
- Import: `webUtils.getPathForFile()` -> `save_dropped_file`
- Export: `dialog_save_file` with platform filters
- Push: platform-specific commands, ADB vs libimobiledevice

### Auto-Updater Integration
- Prod builds include updater artifacts: `createUpdaterArtifacts: true`
- GitHub releases + signed updates, `pubkey` in `tauri.conf.json`
- Frontend call `checkForUpdates()`

## Development Environment Setup

### Required Tools
- macOS: Xcode command line tools, iOS Simulator
- Android: Android SDK, `adb` in PATH
- Rust: latest stable + `llvm-cov`
- Node: v20+ + Yarn

### Environment Variables (.env)
```bash
APPLE_SIGNING_IDENTITY="Developer ID Application: ..."
VITE_SENTRY_DSN="https://..."
VITE_POSTHOG_API_KEY="phc_..."
```

### Pre-commit Automation
- `.git/hooks/pre-push` wired for full test suite, now disabled

## Device-Specific Database Extraction Workflows

### Android Database Extraction (ADB)
Priority search:
```rust
// Priority locations: secured → public → fallback
let locations = vec![
    ("/data/data/", true),           // Secured app data (requires run-as)
    ("/sdcard/Android/data/", false), // Public storage
    ("/storage/emulated/0/Android/data/", false), // Legacy public storage
];
```

Access pattern:
1. Preferred: `adb shell run-as <package> find` + `exec-out run-as <package> cat`
2. Fallback: `adb pull`

Admin extraction:
```bash
adb -s device exec-out run-as package cat /path/file.db > local_file.db
```

### iOS Physical Device Extraction (libimobiledevice)
Documents access:
```rust
// Uses afcclient (Apple File Conduit client) for file access
let cmd_args = ["--documents", package_name, "-u", device_id, "ls", "Documents"];
// Pull: afcclient --documents package -u device pull /Documents/file.db local_file.db
```

Flow:
1. `afcclient --documents <app> -u <device> ls Documents`
2. `afcclient --documents <app> -u <device> pull /Documents/<file> <local_path>`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [groot007/flippio](https://github.com/groot007/flippio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
