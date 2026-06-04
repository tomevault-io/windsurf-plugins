---
trigger: always_on
description: ./scripts/dev.sh build              # debug build (xcodegen + xcodebuild)
---

# Factory Floor - Project Instructions

## Development Workflow

### Build, Test, Run
```bash
./scripts/dev.sh build              # debug build (xcodegen + xcodebuild)
./scripts/dev.sh br                 # build and run
./scripts/dev.sh run [dir]          # kill and relaunch (optionally with a directory)
./scripts/dev.sh test               # run XCTest suite
./scripts/dev.sh release            # release build matching CI (hardened runtime)
./scripts/dev.sh release --run      # release build and run
./scripts/dev.sh clean              # clean build artifacts
./scripts/release.sh [version]      # release build: sign, notarize, create DMG
./scripts/build-editor.sh           # rebuild Monaco editor bundle (auto-run by dev.sh)
```

### After code changes
1. If you added/removed files or changed `project.yml`: run `xcodegen generate` first
2. Build and run: `./scripts/dev.sh br`
3. If tmux mode was on: `tmux -L factoryfloor kill-server`
4. If you changed the tmux config: `rm -f ~/Library/Caches/factoryfloor/tmux.conf`

### When to regenerate the Xcode project
Run `xcodegen generate` when:
- Adding or removing Swift source files
- Adding or removing localization files (lproj)
- Changing `project.yml` (build settings, dependencies, targets)

Do NOT edit `FactoryFloor.xcodeproj` directly. It is generated from `project.yml`.

### Developer setup
```bash
uvx prek install                    # install pre-commit hooks
uvx prek run --all-files            # run hooks on all files (optional)
```

### Release build
```bash
./scripts/release.sh [version]   # builds, signs, notarizes, creates DMG
```

## Git Workflow

### Conventional Commits
All commits MUST use [Conventional Commits](https://www.conventionalcommits.org/) format. This is required for release-please to generate changelogs and version bumps.

Format: `type(scope): description`

Types:
- `feat`: new feature (triggers minor version bump)
- `fix`: bug fix (triggers patch version bump)
- `refactor`: code change that neither fixes a bug nor adds a feature
- `perf`: performance improvement
- `docs`: documentation only
- `ci`: CI/CD changes
- `chore`: maintenance, dependencies, config

Examples:
```
feat: add multiple terminal tabs with Cmd+T
fix: resolve terminal freeze when opening second surface
refactor: extract env var injection to WorkstreamEnvironment
docs: update README with keyboard shortcuts
ci: add GitHub Pages deploy workflow
feat(website): add language switcher to footer
fix(browser): auto-focus address bar on new tab
```

Breaking changes: add `!` after the type or include `BREAKING CHANGE:` in the footer.

### Branching
- Work on feature branches, not directly on `main`
- Branch names: `feat/description`, `fix/description`, `refactor/description`
- Open PRs against `main`
- release-please manages version bumps and changelogs via PR

## Architecture

- **SwiftUI sidebar** + **AppKit terminal views** (Metal GPU-rendered via libghostty)
- **XcodeGen** for project generation (`project.yml` -> xcodeproj)
- **Ghostty** as git submodule (pinned to stable tags), xcframework built with `zig build`
- **Bridging header** at `Resources/FactoryFloor-Bridging-Header.h`
- **Single-window** app via `Window` (not `WindowGroup`)
- **`factoryfloor://`** URL scheme for single-instance behavior
- **AppConstants** (`appID`, `appName`, `configDirectory`, `cacheDirectory`)
- **Sparkle** for auto-updates (DMG users), `UpdateChecker` for Homebrew users
- **prek** pre-commit hooks (`prek.toml`)

### Key directories
- `Sources/Models/` - Data models, git operations, tmux, name generator, app constants
- `Sources/Terminal/` - Ghostty integration (TerminalApp singleton, TerminalView NSView)
- `Sources/Views/` - SwiftUI views (sidebar, settings, project overview, workspace, browser, editor)
- `Localization/` - lproj directories with Localizable.strings
- `Resources/` - Entitlements, bridging header, Assets.xcassets, CLI script
- `Resources/MonacoEditor/` - Built Monaco editor bundle (gitignored, built by `scripts/build-editor.sh`)
- `editor/` - Monaco editor Vite project (source for `Resources/MonacoEditor/`). Built with bun.
- `ghostty/` - Git submodule (do not modify, pinned to stable release tag)
- `website/` - Hugo + Tailwind CSS site for factory-floor.com. **Do not use `.AllTranslations`** in Hugo templates; it returns duplicates because localized contentDirs are nested inside the English `content/` dir. Use a hardcoded language code list instead (see `footer.html` or `docs.html` for the pattern).
- `scripts/` - Release and build automation
- `docs/` - Distribution guide and reference docs

### Data flow
- **Projects/workstreams** stored in UserDefaults (`factoryfloor.projects`), accessed via `ProjectStore`. Wrapped in `ProjectList: ObservableObject` for reference-type semantics.
- **Settings** use `@AppStorage` (UserDefaults), keyed as `factoryfloor.*`
- **Terminal surfaces** cached in `TerminalSurfaceCache` (keyed by UUID)
- **Git repo info** cached in `AppEnvironment`, refreshed async every 15s
- **Tool detection** runs at startup in `AppEnvironment.refresh()`
- **Sidebar state** (selection, expanded sections) stored in UserDefaults (`factoryfloor.selection`, `factoryfloor.expandedProjects`)

### Workstream lifecycle

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alltuner/factoryfloor](https://github.com/alltuner/factoryfloor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
