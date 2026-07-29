---
trigger: always_on
description: > Observed conventions from the codebase. Rules marked `[HYPOTHESIS]` are inferred from patterns but not explicitly stated.
---

# Zafiro.Avalonia — Coding Conventions

> Observed conventions from the codebase. Rules marked `[HYPOTHESIS]` are inferred from patterns but not explicitly stated.

---

## C# Conventions

### Naming

| Element | Convention | Example | Evidence |
|---|---|---|---|
| Private fields | camelCase, **no** leading underscore | `private readonly CompositeDisposable disposable` | 0 underscore-prefixed fields in samples; only 4 in all of src/ (legacy panel code) |
| `[Reactive]` backing fields | `private` lowercase | `[Reactive] private string name;` | `Page1ViewModel.cs`, `MasterDetailsSampleViewModel.cs` |
| Methods returning `Task` | **No** `Async` suffix | `Task OnShowMessage(...)` not `OnShowMessageAsync` | `DialogSampleViewModel.cs`, `WizardViewModel.cs` — sole exception: 1 private `RunAsync` in samples |
| Interfaces | `IHaveX` / `IX` pattern | `IHaveHeader`, `IHaveTitle`, `IHaveFooter` | `Zafiro.UI.Navigation` namespace |
| ViewModels | `FooViewModel` | `HomeViewModel`, `Page1ViewModel` | Universal across samples |
| Views | `FooView` (matching ViewModel) | `HomeView`, `Page1View` | Required by `NamingConventionGeneratedViewLocator` |

### Types and Patterns

| Pattern | Convention | Evidence |
|---|---|---|
| ViewModel base | `ReactiveObject` or `ReactiveValidationObject` | 100% of ViewModels in src/ and samples/ |
| Records for DTOs | `record` for immutable data | `record SampleCard(string Name, string Description, string Icon, string Category, Type ViewModelType)` |
| Nullable reference types | Enabled; prefer `Maybe<T>` over null | `string?` is used for nullable; `Maybe<T>` for semantic absence |
| Command return types | `Result<T>` for fallible; `Unit` for void | `ReactiveCommand<Unit, Result<int>>`, `ReactiveCommand<Unit, Maybe<string>>` |
| DI registration | Constructor injection only | `NavigationSampleViewModel(INavigator navigator)` — primary constructors used |
| `IDisposable` ViewModels | `CompositeDisposable` + `DisposeWith` | `WizardViewModel : IDisposable` |

### Code-Behind Rule

View code-behind (`.axaml.cs`) files contain **only**:

```csharp
public partial class HomeView : UserControl
{
    public HomeView() => InitializeComponent();
}
```

No event handlers. No DI. No logic. All behavior lives in the ViewModel or in Behaviors/Converters.

**Evidence**: Every `.axaml.cs` in `samples/` checked — 100% compliance.

---

## AXAML Conventions

### Namespace Declarations

```xml
<UserControl xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
             xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
             xmlns:local="clr-namespace:MyApp.Views"
             mc:Ignorable="d" d:DesignWidth="800" d:DesignHeight="450"
             x:Class="MyApp.Views.HomeView"
             x:DataType="local:HomeViewModel">
```

- `x:DataType` is declared on the root element for type-safe bindings.
- `x:CompileBindings` is **not** globally enabled — only used in ~7% of AXAML files, on a per-file opt-in basis.

### Binding Patterns

```xml
<!-- Standard property binding (most common) -->
{Binding PropertyName}

<!-- Async observable binding (note the ^) -->
{Binding Navigator.Content^}

<!-- Parent binding with type cast (for DataTemplate contexts) -->
{Binding $parent[UserControl].((vm:HomeViewModel)DataContext).NavigateToSample}

<!-- Self binding -->
{Binding $self.Bounds.Width}

<!-- Two-way binding (explicit) -->
{Binding Text, Mode=TwoWay}
```

**Evidence**: `HomeView.axaml`, `MainView.axaml`, `SlimDataGridView.axaml`.

`[HYPOTHESIS]` The `^` operator on `Navigator.Content^` unwraps `IObservable<T>` — this is standard Avalonia reactive binding syntax. The codebase uses it for `INavigator.Content` which is an observable.

### Style Class Usage (observed in samples)

These utility classes appear in `HomeView.axaml` and other sample views:

| Class | Used On | Likely Purpose |
|---|---|---|
| `Size-XS`, `Size-S`, `Size-M`, `Size-XL` | `TextBlock` | Font size presets |
| `Weight-Bold` | `TextBlock` | Font weight |
| `Text-Muted` | `TextBlock` | Reduced opacity/subdued color |
| `Ghost` | `Button`, `EnhancedButton` | Transparent/minimal button style |
| `Card` | `Border`, `OverlayBorder` | Card styling (defined in `Common.axaml`) |
| `Elevate` | `Border` | Box shadow elevation |
| `Expand` | `EnhancedButton` | Stretch to fill (defined in `Button.axaml`) |
| `CenterContent` | `EnhancedButton` | Center-align content |
| `ShowEmptyContent` | `ListBox`, `CardGrid`, `ItemsControl` | Show empty-state placeholder |
| `H1`, `H3` | `TextBlock` | Heading sizes |

`[HYPOTHESIS]` `Size-*`, `Weight-*`, `Text-Muted`, `Ghost`, `Elevate`, `H1`, `H3` are likely defined in the consuming app's styles or in FluentAvalonia, not in `Zafiro.Avalonia/Styles/`. The library's `Common.axaml` only defines `Card` on `Border`/`OverlayBorder`. `Expand` is defined in `Button.axaml`. Verify where these classes originate before relying on them.

### EnhancedButton Role Classes (confirmed in library)

Defined in `src/Zafiro.Avalonia/Controls/EnhancedButton.axaml`:

- `Primary` — accent-colored
- `Secondary` — subdued

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SuperJMN/Zafiro.Avalonia](https://github.com/SuperJMN/Zafiro.Avalonia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
