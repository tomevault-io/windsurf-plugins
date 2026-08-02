---
trigger: always_on
description: iOS-specific conventions for the SwiftUI layer in `iosApp/`.
---

# iOS App — CLAUDE.md

iOS-specific conventions for the SwiftUI layer in `iosApp/`.

## Component + ComponentModel Pattern

Every screen and navigation view follows the Component + ComponentModel pattern:

1. **Protocol** (`*ComponentModelProtocol`) — defines the public interface (properties, actions)
2. **ComponentModel** (`@Observable final class`) — conforms to the protocol, wraps KMP component
3. **Component** (`struct *Component<Model: *Protocol>: View`) — generic SwiftUI view

```swift
// Protocol
protocol MyComponentModelProtocol: AnyObject {
    var title: String { get }
    var alert: AlertModel? { get set }

    func onAction()
}

// ComponentModel
@Observable
final class MyComponentModel: MyComponentModelProtocol { ... }

// View
struct MyComponent<Model: MyComponentModelProtocol>: View {
    @State var model: Model
    ...
}
```

Screen-level protocols use `AnyObject` constraint. Navigation-level protocols do not.

### KMP Deviation Comment

Screen-level ComponentModel protocols include this doc comment:

```swift
/// KMP deviation: Conforms to `AnyObject` instead of FuturedKit's `ComponentModel` protocol
/// because navigation and event handling are managed by KMP Decompose, not a Swift Coordinator.
```

### Debug Mocks

Each screen ComponentModel has a `#if DEBUG` mock for SwiftUI previews:

```swift
#if DEBUG
@Observable
final class MyComponentModelMock: MyComponentModelProtocol {
    var title = "Preview Title"
    var alert: AlertModel?
    func onAction() {}
}
#endif
```

## Bindings to @State Model Properties

When creating a `Binding` to a property on an `@State` model, use the `$` projection syntax — never use verbose `Binding(get:set:)`:

```swift
// Correct
TabView(selection: $model.selectedTab) { ... }
.sheet(item: $model.sheetItem) { ... }
.alert("", isPresented: $model.alert.isPresented) { ... }

// Wrong
TabView(selection: Binding(
    get: { model.selectedTab },
    set: { model.onTabSelected($0) }
)) { ... }
```

To make a property bindable, expose it with a getter/setter in the ComponentModel:

```swift
var selectedTab: NavigationTab {
    get { viewState.selectedTab }
    set { actions.onTabSelected(tab: newValue) }
}
```

## Slot Binding Dismiss Actions

Every Decompose slot binding used with `.sheet(item:)` or `.fullScreenCover(item:)` **must** call a KMP dismiss action in its setter — never leave it empty. Without this, SwiftUI swipe-to-dismiss does not notify KMP, leaving stale slot state.

```swift
// In ComponentModel
var sheetItem: DecomposeSlotItem<SomeSheetChild>? {
    get {
        guard let child = _sheet.child else { return nil }
        return DecomposeSlotItem(id: ObjectIdentifier(child), instance: child.instance)
    }
    set { // swiftlint:disable:this unused_setter_value
        actions.onSheetDismissed()
    }
}
```

## Navigation ComponentModel Structure

Navigation ComponentModels use MARK comments to organize sections:

```swift
@Observable
final class MyNavigationComponentModel: MyNavigationComponentModelProtocol {

    // MARK: Public computed properties
    ...

    // MARK: Private stored properties
    ...

    // MARK: Private @ObservationIgnored properties
    ...

    // MARK: Init / Deinit
    ...

    // MARK: Public functions
    ...
}
```

Views never access `model.actions` directly — all actions are wrapped in model methods or computed property setters.

## View Modifier Ordering

Screen-level views apply modifiers in this order:

```swift
var body: some View {
    content
        .background(...)                    // 1. Appearance
        .navigationTitle(...)               // 2. Navigation setup
        .navigationBarTitleDisplayMode(.inline)
        .toolbar { ... }
        .onChange(of: ...) { ... }          // 3. Event handling
}
```

Not every modifier is present on every screen — only include what the screen needs, but keep the relative order consistent.

For screens with complex content, extract a `contentView` computed property and apply modifiers to it in `body`.

## Localization

All user-facing strings **must** use KMP localization keys from `Localizable` — never hardcode text directly. The KMP module (`import KMP`) exposes `Localizable` with `.localized` for accessing translated strings from `strings.xml`.

```swift
// Correct
Text(Localizable.first_screen_title.localized)
Button(Localizable.first_screen_button.localized, action: model.onNext)

// Wrong — hardcoded string
Text("First Screen")
```

## Icons

All icons **must** come from KMP shared resources (`Images.ic_*`) — never add new icons to `Assets.xcassets`. The KMP module is the single source of truth for icons shared across platforms.

- Use `.templateImage` for monochrome icons that should adopt the current foreground style (menus, toolbars, list rows)
- Use `.image` only for non-icon resources (logos, illustrations) that should render with original colors

```swift
// Correct — KMP icon with template rendering
Images.ic_trash.templateImage

// Wrong — adding icons to xcassets
Image(systemName: "trash")
```

## Struct Property Access Control

When a struct has a custom `init`, all stored properties that are not accessed from external call sites **must** be marked `private`. The custom `init` provides the public API — internal properties should not leak.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [futuredapp/kmp-futured-template](https://github.com/futuredapp/kmp-futured-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
