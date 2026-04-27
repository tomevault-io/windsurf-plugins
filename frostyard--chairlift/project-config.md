---
trigger: always_on
description: ChairLift is a modern GTK4/Libadwaita system management tool originally designed for Snow Linux but made portable to other distributions. It provides a unified interface for:
---

# GitHub Copilot Instructions for ChairLift

## Project Overview

ChairLift is a modern GTK4/Libadwaita system management tool originally designed for Snow Linux but made portable to other distributions. It provides a unified interface for:

- Homebrew package management
- System health monitoring
- Flatpak application management
- NBC (bootc container) system updates
- System updates and maintenance

## Architecture

### Technology Stack

- **UI Framework**: GTK4 + Libadwaita via [puregotk](https://codeberg.org/puregotk/puregotk)
- **Language**: Go
- **Build System**: Make
- **Configuration**: YAML-based

### Key Design Patterns

1. **Async Operations**: All long-running operations (package installation, searches, updates) run in goroutines with `runOnMainThread()` helper for UI updates via `glib.IdleAdd()`

2. **Configuration-Driven UI**: The application reads `config.yml` to dynamically show/hide UI groups and configure application launchers, making it portable across distributions

3. **Toast Notifications**: User feedback is provided via `adw.Toast` notifications accessed through the `ToastAdder` interface

4. **Dry-Run Mode**: All packages (homebrew, flatpak, nbc) support `--dry-run` flag to preview changes without modifying the system

### Project Structure

```
chairlift/
├── cmd/
│   └── chairlift/
│       └── main.go           # Application entry point
├── internal/
│   ├── app/
│   │   └── app.go            # Main application setup
│   ├── config/
│   │   └── config.go         # YAML configuration loading
│   ├── flatpak/
│   │   └── flatpak.go        # Flatpak integration
│   ├── homebrew/
│   │   └── homebrew.go       # Homebrew integration
│   ├── nbc/
│   │   └── nbc.go            # NBC bootc integration
│   ├── views/
│   │   └── userhome.go       # Page content and UI logic
│   └── window/
│       └── window.go         # Main window and navigation
├── data/                     # Desktop files, icons, policies
├── config.yml                # Default configuration
├── CONFIG.md                 # Configuration documentation
└── Makefile                  # Build commands
```

## Development Workflow

### After Any Code Changes

**Always run these commands after making changes:**

```bash
make fmt    # Format code with gofmt
make lint   # Run golangci-lint
make build  # Verify it compiles
```

### Build Commands

- `make build` - Build the application
- `make run` - Build and run with `--dry-run` flag
- `make fmt` - Format all Go code
- `make lint` - Run linter (requires golangci-lint)
- `make test` - Run tests
- `make clean` - Clean build artifacts

## Configuration System

**Critical**: ChairLift uses a YAML configuration file to control UI behavior. Always maintain this pattern when adding new features.

### Config File Locations (searched in order)

1. `/etc/chairlift/config.yml` (system-wide - highest priority)
2. `/usr/share/chairlift/config.yml` (package maintainer defaults)
3. `config.yml` (development/source directory)

### Adding Configurable Features

When adding new preference groups or external app integrations:

1. **Add to config.yml**:

```yaml
page_name:
  group_name:
    enabled: true
    app_id: com.example.App # for external apps
```

2. **Add to internal/config/config.go** (default config):

```go
PageConfig{
    "group_name": GroupConfig{Enabled: true},
}
```

3. **Read in views code**:

```go
if uh.config.IsGroupEnabled("page_name", "group_name") {
    // Build and add the group
}

// For app IDs:
groupCfg := uh.config.GetGroupConfig("page_name", "group_name")
appID := "default.app"
if groupCfg != nil && groupCfg.AppID != "" {
    appID = groupCfg.AppID
}
```

4. **Update documentation**: Add entries to CONFIG.md

## Coding Conventions

### Go Style

- Follow standard Go conventions
- Use meaningful variable names, especially for UI widgets
- Add comments to exported functions
- Keep functions focused and small

### Threading Pattern

Always follow this pattern for async operations:

```go
func (uh *UserHome) onActionClicked(button *gtk.Button) {
    button.SetSensitive(false)
    button.SetLabel("Processing...")

    go func() {
        // Do work in goroutine
        result, err := someOperation()

        runOnMainThread(func() {
            button.SetSensitive(true)
            button.SetLabel("Original Label")

            if err != nil {
                uh.toastAdder.ShowErrorToast(fmt.Sprintf("Failed: %v", err))
                return
            }
            uh.toastAdder.ShowToast("Success!")
        })
    }()
}
```

The `runOnMainThread()` helper uses `glib.IdleAdd()` to safely update UI from goroutines.

### UI Widget Creation

- Use `adw.NewPreferencesGroup()` for logical groupings
- Use `adw.NewActionRow()` for list items
- Use `adw.NewExpanderRow()` for collapsible sections
- Add icons with `gtk.NewImageFromIconName()`
- Store widget references in the struct for later updates

### Adding New Packages (internal modules)

When adding a new package manager integration (like the flatpak package):

1. Create `internal/packagename/packagename.go`
2. Implement `SetDryRun(bool)` and `IsDryRun() bool` for dry-run support

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [frostyard/chairlift](https://github.com/frostyard/chairlift) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
