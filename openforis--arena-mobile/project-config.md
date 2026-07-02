---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Arena Mobile is a React Native mobile application built with Expo for offline data collection on Android and iOS. It's a companion app to Open Foris Arena, enabling field data collection for forest inventories, surveys, and interviews with support for diverse data types (numbers, coordinates, images, videos, text) and customizable validation rules.

**Key Technologies:**
- React Native 0.81.5 with Expo 54
- TypeScript (migrated from JavaScript)
- Redux Toolkit for state management
- React Navigation for routing
- React Native Paper for UI components
- SQLite (expo-sqlite) for local data persistence
- Socket.io for real-time server communication
- @openforis/arena-core for shared business logic

## Development Commands

### Starting the App
```bash
# Start development server
yarn start

# Start with tunnel (for testing on external devices)
yarn start-tunnel

# Platform-specific starts
yarn android
yarn ios
yarn web
```

### Code Quality
```bash
# Run linter with auto-fix
yarn lint

# Type check TypeScript files
yarn test:types
```

**Note:** This project does not have automated unit tests. Testing is done manually on devices.

### Building & Deployment
```bash
# Build for development (internal distribution)
eas build --profile development --platform [android|ios]

# Build preview (internal distribution)
eas build --profile preview --platform [android|ios]

# Build for production
eas build --profile production --platform [android|ios]

# Submit to App Store (iOS only, requires AuthKey_4TQM7KV3QK.p8)
eas submit --platform ios --profile production
```

## Code Architecture

### Module Resolution
The project uses Babel module resolver with `src/` as the root. Import from top-level directories without relative paths:
```javascript
import { SurveyService } from "service"
import { useEffectiveTheme } from "hooks"
import { SurveySelectors } from "state"
```

### State Management (Redux)
Located in `src/state/`, organized by domain:
- **dataEntry**: Current record editing state, selected nodes, validation
- **survey**: Survey definitions, loaded surveys, current survey
- **settings**: App settings (GPS locking, screen orientation, language, etc.)
- **remoteConnection**: Server connection state, authentication
- **deviceInfo**: Battery state, disk storage, device information
- **screenOptions**: UI state for different screens
- **message/confirm/toast**: UI dialog state
- **jobMonitor**: Background job status (imports, exports, sync)

Each domain exports: `Actions`, `Reducer`, `Selectors`, and sometimes `State` utilities.

### Data Layer

**Database (src/db/):**
- `SQLiteClient.ts`: Custom wrapper around expo-sqlite with migration support
- `migrations/`: Sequential database schema migrations (version-controlled via PRAGMA user_version)
- Database is initialized in `AppInitializer` before app renders

**Repositories (src/service/repository/):**
- `recordRepository.ts`: CRUD operations for survey records (~630 lines, complex)
- `surveyRepository.ts`: Survey metadata storage/retrieval
- `surveyFSRepository.ts`: Survey file system operations
- `recordFileRepository.ts`: Record file attachments
- Repositories interact directly with SQLite, not abstracted ORMs

**Services (src/service/):**
- Business logic layer between components and repositories
- `surveyService.ts`: Survey import, export, demo survey loading
- `recordService.ts`: Record lifecycle, validation, updates
- `recordRemoteService.ts`: Server sync for records
- `authService.ts`: Authentication tokens, secure storage
- `preferencesService.ts`: User preferences persistence
- Job services (`*Job.ts`): Background operations (imports, exports, backups)

### Navigation (src/navigation/)
Single `AppStack` component using React Navigation Native Stack. Screen definitions in `src/screens/screens.ts` with keys in `src/screens/screenKeys.ts`.

### Screens (src/screens/)
Major screens:
- **HomeScreen**: Survey selection, remote connection
- **SurveysList**: Local and remote survey management
- **RecordsList**: Record browsing, filtering, sync status
- **RecordEditor**: Complex multi-page form editor (see below)
- **RecordValidationReport**: Validation errors display
- **SettingsScreen**: App configuration
- **AboutScreen**: Version info, licenses

### Record Editor Architecture
`src/screens/RecordEditor/` is the most complex screen:

- **RecordEditor.tsx**: Main container coordinating all subcomponents
- **RecordPageForm**: Renders a single page of form fields
- **NodeComponentSwitch**: Maps node types to specific input components
  - `nodeTypes/`: Individual input components (NodeTextComponent, NodeCodeComponent, NodeCoordinateComponent, NodeImageOrVideoComponent, NodeTaxonComponent, NodeDateComponent, NodeFileComponent, etc.)
- **NodeMultipleEntityComponent**: Table view for repeating entities
- **RecordNodesCarousel**: Horizontal swipe navigation between nodes
- **PagesNavigationTree**: Hierarchical page navigation drawer
- **PageNodesList**: List view of nodes in current page
- **BottomNavigationBar**: Quick navigation between entity pages

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openforis/arena-mobile](https://github.com/openforis/arena-mobile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-02 -->
