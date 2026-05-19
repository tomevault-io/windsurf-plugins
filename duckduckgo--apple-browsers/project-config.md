---
trigger: always_on
description: ALWAYS use WindowsManager for creating and managing browser windows:
---


# macOS Window Management and AppKit Patterns

## WindowsManager for Window Operations
ALWAYS use WindowsManager for creating and managing browser windows:

```swift
// ✅ CORRECT - WindowsManager usage
@MainActor
final class FeatureCoordinator {
    func openNewWindow() {
        let tabCollection = TabCollectionViewModel()
        WindowsManager.openNewWindow(
            with: tabCollection,
            burnerMode: .regular,
            droppingPoint: nil
        )
    }
    
    func openWindowWithURL(_ url: URL) {
        let tabCollection = TabCollectionViewModel()
        let window = WindowsManager.openNewWindow(with: tabCollection)
        window?.tabCollectionViewModel.addTab(with: url)
    }
}

// ❌ INCORRECT - Direct window creation
final class FeatureCoordinator {
    func openNewWindow() {
        let window = NSWindow() // Don't create windows directly
        window.makeKeyAndOrderFront(nil)
    }
}
```

## Window Controller Architecture
Use NSWindowController for complex window management:

```swift
// ✅ CORRECT - NSWindowController pattern
final class FeatureWindowController: NSWindowController {
    private let viewModel: FeatureViewModel
    
    init(viewModel: FeatureViewModel) {
        self.viewModel = viewModel
        super.init(window: nil)
        setupWindow()
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    private func setupWindow() {
        let contentViewController = FeatureViewController(viewModel: viewModel)
        
        window = NSWindow(contentViewController: contentViewController)
        window?.setContentSize(NSSize(width: 800, height: 600))
        window?.minSize = NSSize(width: 400, height: 300)
        window?.center()
        window?.title = "Feature Window"
        
        // Configure window behavior
        window?.isRestorable = true
        window?.identifier = NSUserInterfaceItemIdentifier("FeatureWindow")
    }
    
    override func windowDidLoad() {
        super.windowDidLoad()
        
        // Additional window setup
        window?.delegate = self
        setupToolbar()
    }
}

// MARK: - NSWindowDelegate
extension FeatureWindowController: NSWindowDelegate {
    func windowWillClose(_ notification: Notification) {
        // Clean up resources
        viewModel.cleanup()
    }
    
    func windowDidBecomeMain(_ notification: Notification) {
        // Handle window becoming main
        viewModel.windowDidBecomeActive()
    }
}
```

## Multi-Window State Management
Use TabCollectionViewModel for window-specific state:

```swift
// ✅ CORRECT - Window-specific state management
@MainActor
final class WindowCoordinator {
    private let tabCollectionViewModel: TabCollectionViewModel
    private weak var windowController: NSWindowController?
    
    init(tabCollectionViewModel: TabCollectionViewModel) {
        self.tabCollectionViewModel = tabCollectionViewModel
    }
    
    func currentTab() -> Tab? {
        return tabCollectionViewModel.selectedTab
    }
    
    func addNewTab(with url: URL? = nil) {
        tabCollectionViewModel.addTab(with: url)
    }
    
    func closeCurrentTab() {
        guard let currentTab = tabCollectionViewModel.selectedTab else { return }
        tabCollectionViewModel.removeTab(currentTab)
    }
    
    func closeWindow() {
        windowController?.close()
    }
}
```

## Window State Restoration
Implement proper state restoration:

```swift
// ✅ CORRECT - Window state restoration
extension FeatureWindowController {
    override func restoreState(with coder: NSCoder) {
        super.restoreState(with: coder)
        
        // Restore window-specific state
        if let savedData = coder.decodeObject(forKey: "viewModelState") as? Data {
            viewModel.restoreState(from: savedData)
        }
    }
    
    override func encodeRestorableState(with coder: NSCoder) {
        super.encodeRestorableState(with: coder)
        
        // Save window-specific state
        if let stateData = viewModel.encodeState() {
            coder.encode(stateData, forKey: "viewModelState")
        }
    }
}
```

## NSViewController and SwiftUI Integration
Use NSHostingView for SwiftUI integration:

```swift
// ✅ CORRECT - NSHostingView integration
final class FeatureViewController: NSViewController {
    private let viewModel: FeatureViewModel
    
    init(viewModel: FeatureViewModel) {
        self.viewModel = viewModel
        super.init(nibName: nil, bundle: nil)
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    override func loadView() {
        view = NSHostingView(rootView: FeatureView(viewModel: viewModel))
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Feature"
        preferredContentSize = NSSize(width: 400, height: 300)
    }
    
    override func viewWillAppear() {
        super.viewWillAppear()
        viewModel.viewWillAppear()
    }
    
    override func viewDidAppear() {
        super.viewDidAppear()
        viewModel.viewDidAppear()
    }
}
```

## View Controller Lifecycle
Follow AppKit view controller patterns:

```swift
// ✅ CORRECT - AppKit lifecycle management

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duckduckgo/apple-browsers](https://github.com/duckduckgo/apple-browsers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
