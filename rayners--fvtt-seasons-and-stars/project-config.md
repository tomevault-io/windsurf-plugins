---
trigger: always_on
description: > `AGENTS.md` is a symlink to this file for tool compatibility.
---

# Seasons & Stars Internal Development Context

> `AGENTS.md` is a symlink to this file for tool compatibility.

## Development Guidelines

- Source code is written in TypeScript and bundled with Rollup; do not commit generated `dist` output.
- All calendar data must be verifiable against publicly available information unless the user has explicitly confirmed the data's validity from an external source. Document verification by adding a `sources` array to calendar definitions when possible.
- Only include reference or book citations in the `sources` array when a user has supplied the exact citation text. Do not invent or summarize bibliography entries—use publicly accessible URLs instead.
- Before committing, run:
  - `npm run lint`
  - `npm run typecheck`
  - `npm run test:run`
  - `npm run build`
- Use `npm run format` or `npm run lint:fix` to resolve formatting issues.
- Write descriptive commit messages and keep pull requests focused.
- Continuous integration runs on Node 20, 22, and 24, executing lint, typecheck, build, tests, and calendar validation.
- PR titles must follow the Conventional Commits style.
- Refer to the GitHub Action config in `.github/workflows/semantic-pull-request.yml` for allowed commit message types and scopes.
- Releases are automated with [release-please](https://github.com/googleapis/release-please); do not manually edit version numbers or `CHANGELOG.md`.

### Simple Calendar Compatibility Bridge Coordination

- **Coordinate early:** When planning any Seasons & Stars API change, notify the maintainers of the [Simple Calendar Compatibility Bridge](https://github.com/rayners/foundryvtt-simple-calendar-compat) and link to the tracking issue/PR in both repositories.
- **What counts as an API change?** Treat the following as externally consumed surfaces that require Bridge validation:
  - Hook events documented in `docs/DEVELOPER-GUIDE.md` or exposed through `Hooks.call`/`Hooks.callAll`.
  - Exported TypeScript definitions, interfaces, or enums under `packages/core/src/types/` and any symbols re-exported from package entry points.
  - Public methods on `CalendarManager`, `CalendarEngine`, `BridgeIntegration`, or other classes referenced in integration guides.
  - JSON contracts passed between the compatibility layer and downstream modules (e.g., note payloads, time advancement messages).
- **Examples that require coordination:**
  - Renaming or removing a hook such as `seasons-and-stars.calendarUpdated` or changing its payload structure.
  - Adding mandatory parameters to `BridgeIntegration` methods or modifying the Simple Calendar translation tables.
  - Changing the schema of exported interfaces like `CalendarDateData` that compatibility bridges deserialize.
- **Process checklist:**
  1. Open or update an issue in the Bridge repository describing the required adjustments and reference the Seasons & Stars PR.
  2. Provide migration notes or a draft PR for the Bridge when feasible so dependent modules can test pre-release builds.
  3. Call out the coordination requirement in the Seasons & Stars changelog or release notes once the work lands.
- **Automation:** We are evaluating a GitHub Action that diffs exported symbols and hook declarations to flag potential API changes automatically. Until it lands, rely on manual review and err on the side of coordinating even “minor” tweaks.

## Module Architecture Deep Dive

### Calendar Engine Core (packages/core/src/core/)

**Primary Components**:

- `CalendarEngine`: Core date arithmetic and calendar interpretation
- `DateFormatter`: Human-readable date string generation with cultural context
- `TimeConverter`: Bidirectional worldTime <-> calendar date conversion
- `CalendarValidator`: JSON schema validation and error reporting
- `CalendarManager`: High-level calendar system coordination
- `CalendarLoader`: Dynamic calendar definition loading and caching
- `TimeAdvancementService`: Centralized time progression management

**Critical Implementation Details**:

- Uses `game.time.worldTime` as single source of truth
- JSON calendar definitions with fallback value handling
- Round-trip conversion testing prevents data loss bugs
- Intercalary day support requires special arithmetic handling
- **Monorepo Architecture**: Packages-based structure with core logic in packages/core/
- Notes system with recurring events, categories, and permissions
- Performance-optimized note management with search capabilities

### Widget System Architecture (packages/core/src/ui/)

**Widget Factory Pattern**:

- `WidgetManager`: Registration and lifecycle management
- `CalendarWidget`: Full calendar grid display
- `CalendarMiniWidget`: Compact date display with SmallTime integration
- `CalendarGridWidget`: Main calendar interface
- `QuickTimeButtons`: Time advancement controls (in core/)

**UI Integration Patterns**:

- Element-specific positioning (targets player list for mini widget)
- Automatic fallback positioning for cross-version compatibility
- Event-driven updates with debouncing for performance
- Dialog-based calendar selection and configuration

### System Integration Layer

**Game System Adapters**:

- `BridgeIntegration`: Modern integration approach (packages/core/src/core/)
- `CompatibilityManager`: Feature detection and graceful degradation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rayners/fvtt-seasons-and-stars](https://github.com/rayners/fvtt-seasons-and-stars) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
