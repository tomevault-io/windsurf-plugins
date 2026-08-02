---
trigger: always_on
description: Zhip is an iOS Zilliqa wallet app built with UIKit + Combine using a custom MVVM architecture centered on reactive data-binding. The app targets **iOS 17.0** (see `project.yml` and `Package.swift`).
---

# Zhip — Codebase Guide for Claude

## Overview

Zhip is an iOS Zilliqa wallet app built with UIKit + Combine using a custom MVVM architecture centered on reactive data-binding. The app targets **iOS 17.0** (see `project.yml` and `Package.swift`).

The reactive layer is Apple's `Combine` framework throughout. There are no RxSwift/RxCocoa/RxDataSources dependencies — all streams are `AnyPublisher`, all subjects are `PassthroughSubject`/`CurrentValueSubject`, and subscription lifetime is managed by `Set<AnyCancellable>`.

The reactive-MVVM scaffolding (`ViewModelType`, `Binder<T>`, `-->`, `Coordinating`/`BaseCoordinator`/`Navigator`, the DI primitives) lives in a **sibling repo** at `../NanoViewController/`, consumed by Zhip via a local-path SPM dep. Library products:

- **NanoViewController** repo (sibling): `NanoViewControllerCore` / `Combine` / `Navigation` / `Controller` / `SceneViews` / `DIPrimitives` — the reactive-MVVM scaffolding (formerly the in-tree `SingleLineController*` modules).
- `Validation` — reactive-validation framework, lives in this repo.
- `Resources` — bundled fonts/html/audio (Bundle.module shim).
- `AppFeature` — the entire Zhip app: Coordinators, Scenes, ViewModels, Views, Models, UseCases, Persistence, DI, Extensions. Consumed by the Zhip iOS-app target as a single SPM product; the iOS-app target itself is just `App/AppDelegate.swift` plus bundle resources.

Zhip's own SPM targets live at `/Sources/<TargetName>/`. The NanoViewController sources live at `../NanoViewController/Sources/<TargetName>/`. Per-feature extraction of `AppFeature` into smaller SPM modules is a possible future step.

### Project generation

The `Zhip.xcodeproj` is **generated** from `/project.yml` via XcodeGen and is gitignored. After cloning / checking out / changing `project.yml`:

```sh
just gen   # alias for `xcodegen generate`
```

`just test` / `just cov` etc. assume `Zhip.xcodeproj` exists, so run `just gen` first.

---

## Architecture: Reactive MVVM with `SceneController<View>`

### Core Concept

Every screen consists of exactly two files:
- **`<Scene>View.swift`** — UIView subclass conforming to `ViewModelled`
- **`<Scene>ViewModel.swift`** — inherits `BaseViewModel`, implements `ViewModelType`

A generic `SceneController<View>` instantiates both, wires them together, and is almost never subclassed. The "scene controller" is the glue, not the business logic.

### Data Flow (Unidirectional)

```
User Action (tap, type, toggle)
        ↓
InputFromView struct  ←  View's `inputFromView` computed property
        ↓
ViewModel.Input(fromView:, fromController:)
        ↓
ViewModel.transform(input:) → Output
        ↓
View.populate(with: output) → [AnyCancellable]
        ↓
UI updated via `-->` operator binding outputs to Binders
```

---

## Key Protocols & Types

### `ViewModelType`
**File**: `../NanoViewController/Sources/NanoViewControllerCore/ViewModelType.swift`

```swift
protocol ViewModelType {
    associatedtype Input: InputType
    associatedtype OutputVM
    func transform(input: Input) -> OutputVM
}
```

All ViewModels implement `transform`. It is the only public method — input in, output out.

### `InputType`
**File**: `../NanoViewController/Sources/NanoViewControllerCore/InputType.swift`

```swift
protocol InputType {
    associatedtype FromView
    associatedtype FromController
    var fromView: FromView { get }
    var fromController: FromController { get }
    init(fromView: FromView, fromController: FromController)
}
```

Every ViewModel's `Input` has two channels:
- `fromView` — user interactions (taps, text, toggles) as `AnyPublisher<T, Never>`
- `fromController` — lifecycle events and navigation subjects from `InputFromController`

### `ViewModelled`
**File**: `../NanoViewController/Sources/NanoViewControllerController/ViewModelled.swift`

```swift
protocol ViewModelled: EmptyInitializable {
    associatedtype ViewModel: ViewModelType
    typealias InputFromView = ViewModel.Input.FromView
    var inputFromView: InputFromView { get }
    func populate(with viewModel: ViewModel.OutputVM) -> [AnyCancellable]
}
```

Views provide:
- `inputFromView` — computed property building the `InputFromView` struct from UIKit publisher extensions
- `populate(with:)` — binds ViewModel output streams back to UI controls, returns `[AnyCancellable]`

### `InputFromController`
**File**: `../NanoViewController/Sources/NanoViewControllerController/InputFromController.swift`

```swift
struct InputFromController {
    let viewDidLoad: AnyPublisher<Void, Never>
    let viewWillAppear: AnyPublisher<Void, Never>
    let viewDidAppear: AnyPublisher<Void, Never>
    let leftBarButtonTrigger: AnyPublisher<Void, Never>
    let rightBarButtonTrigger: AnyPublisher<Void, Never>
    let titleSubject: PassthroughSubject<String, Never>         // ViewModel sends → controller updates nav title
    let leftBarButtonContentSubject: PassthroughSubject<BarButtonContent, Never>
    let rightBarButtonContentSubject: PassthroughSubject<BarButtonContent, Never>
    let toastSubject: PassthroughSubject<Toast, Never>
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sajjon/Zhip](https://github.com/Sajjon/Zhip) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
