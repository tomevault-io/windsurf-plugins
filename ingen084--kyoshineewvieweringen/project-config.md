---
trigger: always_on
description: **Japanese Priority**: This project is a Japanese disaster prevention application. All user-facing content must be in Japanese:
---

# CLAUDE.md

## Language Support

**Japanese Priority**: This project is a Japanese disaster prevention application. All user-facing content must be in Japanese:
- **UI text and messages**: All interface elements, dialogs, and user messages must be in Japanese
- **Log messages**: Write all log messages in Japanese as a characteristic of disaster prevention applications
- **Comments in code**: Write comments in Japanese to maintain consistency with the application domain
- **Error messages**: Display error messages to users in Japanese
- **Terminology**: Earthquake, tsunami, and weather terminology should follow Japan Meteorological Agency (JMA) standards

**Documentation**: Technical documentation and code structure explanations may be in English for international collaboration, but implementation details should prioritize Japanese.

## Project Overview

**KyoshinEewViewer for ingen** - Japanese disaster prevention application
- C# .NET 9.0 + Avalonia UI for cross-platform support
- Monitors seismic activity from JMA and strong motion networks
- Displays real-time earthquake early warnings and earthquake information

## Build Commands

```bash
# Main project
dotnet build src/KyoshinEewViewer/KyoshinEewViewer.csproj

# Desktop version
dotnet build src/KyoshinEewViewer.Desktop/KyoshinEewViewer.Desktop.csproj
```

## Architecture

### Series Architecture
Plugin-based modular architecture separating monitoring functions:

- **KyoshinMonitor**: Strong motion network monitoring and earthquake early warnings
- **Earthquake**: JMA XML earthquake information processing
- **Tsunami**: Tsunami warning system
- **Typhoon**: Typhoon tracking
- **Lightning**: Lightning detection
- **Radar**: Weather radar
- **Qzss**: Satellite disaster crisis management reporting

Each Series structure (`src/KyoshinEewViewer/Series/[SeriesName]/`):
- View (AXAML/ViewModel)
- Layer (Map rendering)
- Services (Data processing)
- Models (Data structures)
- SettingPages (Settings UI)
- Templates (Script templates)
- Workflow (Workflow definitions)

### Core Technology Stack
- **Avalonia UI**: AXAML, MVVM, cross-platform
- **ReactiveUI**: Reactive programming
- **KyoshinMonitorLib**: Strong motion monitor processing
- **FluentAvalonia**: Modern UI
- **Scriban**: Template engine
- **ManagedBass**: Audio

## Project Structure

### Main Projects
- `KyoshinEewViewer`: Main application (Series, UI, services)
- `KyoshinEewViewer.Desktop`: Desktop version entry point
- `KyoshinEewViewer.Core`: Shared models, themes, utilities
- `KyoshinEewViewer.Map`: Geographic rendering and map projection
- `KyoshinEewViewer.CustomControl`: Custom UI controls

### Parser Libraries
- `KyoshinEewViewer.JmaXmlParser`: JMA XML parsing
- `KyoshinEewViewer.DCReportParser`: QZSS disaster crisis management report parsing
- `KyoshinEewViewer.CsvSourceGenerator`: CSV dictionary code generation

### Configuration
- `common.props`: Shared MSBuild properties (.NET 9.0, Nullable, etc.)

## Development Patterns

### UI Development (Avalonia)
- MVVM: ViewModels inheriting from `ViewModelBase`
- AXAML markup (Avalonia version of XAML)
- Compiled bindings (enabled by default)
- FluentAvalonia component usage
- **Command Binding**: Avalonia recognizes methods directly as Commands, so `ICommand` implementation is unnecessary. However, since Avalonia 12, compiled bindings only resolve methods with no parameters or a single `object` parameter (AVLN2000 otherwise). For methods that receive a `CommandParameter`, take `object?` and cast with a guard clause (`if (parameter is not Foo foo) return;`); for methods also called from C# with typed arguments, keep the typed method and add a parameterless overload for XAML
- **StringFormat Binding**: When binding a numeric or date/time value with `StringFormat` for display only (even on `Run`/`TextBlock`/`Label`), always specify `Mode=OneWay` explicitly. Without it, Avalonia attempts a reverse conversion (string → source type) that raises noisy first-chance `FormatException`/`ArgumentException` when the formatted string contains units (e.g. `"000.1 km/h"`). If the exception persists, fall back to exposing a pre-formatted string property on the ViewModel instead.
- **Markdown Display**: Always render Markdown through `Controls/MarkdownViewer.cs` (`MarkdownViewer`), never the raw `LiveMarkdown.Avalonia` `MarkdownRenderer` directly — it lacks the link-click workaround needed for LiveMarkdown.Avalonia 2.2.0's link-click bug.

#### Conditional Styling Pattern

For applying different styles based on boolean properties, use the `Classes.` syntax instead of creating converters:

**Use Classes Pattern** (Recommended):

```xml
<Button>
    <Button.Styles>
        <Style Selector="ui|FASymbolIcon.muted">
            <Setter Property="Foreground" Value="{DynamicResource EmphasisForegroundColor}" />
        </Style>
    </Button.Styles>
    <ui:FASymbolIcon Classes.muted="{Binding IsMuted}" />
</Button>
```

**Style Reusability**:

- When styles need to be reused across multiple controls or files, define them in shared resource dictionaries or at a higher level in the visual tree
- Prefer extracting reusable styles to `UserControl.Styles` or `Window.Styles` sections

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ingen084/KyoshinEewViewerIngen](https://github.com/ingen084/KyoshinEewViewerIngen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
