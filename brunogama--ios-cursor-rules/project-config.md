---
trigger: always_on
description: As an AI assistant working on iOS applications, I should follow these guidelines to ensure high-quality, user-friendly, and efficient iOS apps.
---

# iOS Development Best Practices

As an AI assistant working on iOS applications, I should follow these guidelines to ensure high-quality, user-friendly, and efficient iOS apps.

## iOS App Architecture

### Architecture Patterns

- Use appropriate architecture patterns based on app complexity:
  - **MVC**: Simple apps or when following Apple's basic patterns
  - **MVVM**: Medium complexity apps, works well with SwiftUI/Combine
  - **VIPER/Clean**: Complex apps requiring clear separation of concerns
  - **Composable Architecture**: Advanced reactive apps with predictable state management

- Keep view controllers/views lightweight by moving business logic to separate components
- Use coordinator pattern for complex navigation flows
- Consider feature-based modularization for large applications

### Project Organization

- Organize project by feature, not by type
- Use Swift packages to modularize components
- Group related files in logical folders
- Utilize Xcode's project navigator groups to maintain code organization

```
AppProject/
├── Core/
│   ├── Networking/
│   ├── Storage/
│   ├── Authentication/
│   └── Common UI Components/
├── Features/
│   ├── User Profile/
│   │   ├── Models/
│   │   ├── Views/
│   │   └── ViewModels/
│   ├── Shopping Cart/
│   │   ├── Models/
│   │   ├── Views/
│   │   └── ViewModels/
├── Resources/
│   ├── Assets.xcassets/
│   ├── Localization/
│   └── Fonts/
└── Supporting Files/
    ├── AppDelegate.swift
    ├── SceneDelegate.swift
    ├── Info.plist
    └── Configuration/
```

## UIKit vs SwiftUI

### When to Use UIKit
- When targeting iOS 12 or earlier
- For complex custom UI that's difficult to implement in SwiftUI
- When you need precise control over UI performance optimization
- For apps heavily dependent on UIKit-specific features

### When to Use SwiftUI
- For new apps targeting iOS 14+
- For rapid development and prototyping
- When you want to share UI code across Apple platforms
- For list-based interfaces and standard UI components

### Hybrid Approach
- Consider UIHostingController to embed SwiftUI views in UIKit apps
- Use UIViewRepresentable and UIViewControllerRepresentable to embed UIKit in SwiftUI
- Adopt SwiftUI for new features while maintaining existing UIKit code

```swift
// UIKit hosting SwiftUI example
let profileView = ProfileView(user: currentUser)
let hostingController = UIHostingController(rootView: profileView)
navigationController.pushViewController(hostingController, animated: true)

// SwiftUI hosting UIKit example
struct MapViewWrapper: UIViewRepresentable {
    func makeUIView(context: Context) -> MKMapView {
        return MKMapView()
    }
    
    func updateUIView(_ uiView: MKMapView, context: Context) {
        // Update the map view
    }
}
```

## iOS App Lifecycle Management

### Modern App Lifecycle (iOS 13+)
- Use SceneDelegate for apps supporting multiple windows
- Properly handle state transitions in `sceneWillResignActive`, `sceneDidEnterBackground`, etc.
- Save user data during state transitions

### Legacy App Lifecycle
- For iOS 12 and earlier, use AppDelegate for lifecycle events
- Handle all state transitions appropriately: `applicationWillResignActive`, `applicationDidEnterBackground`, etc.

### Background Tasks
- Register background tasks with identifiers in your app delegate
- Keep background execution code efficient to avoid system termination
- Use appropriate background modes in Info.plist
- Consider using BGAppRefreshTask for periodic updates

```swift
// Registering a background task
var backgroundTask: UIBackgroundTaskIdentifier = .invalid

func startBackgroundTask() {
    backgroundTask = UIApplication.shared.beginBackgroundTask { [weak self] in
        self?.endBackgroundTask()
    }
    
    // Perform background work
    
    endBackgroundTask()
}

func endBackgroundTask() {
    if backgroundTask != .invalid {
        UIApplication.shared.endBackgroundTask(backgroundTask)
        backgroundTask = .invalid
    }
}
```

## Handling Device Capabilities and Constraints

### Device Adaptation
- Use Auto Layout for responsive UI across different screen sizes
- Implement size classes to adapt layouts between iPhone and iPad
- Use Dynamic Type to support different text sizes
- Test on multiple device sizes and orientations

### Resource Management
- Optimize images and assets for different screen scales (@1x, @2x, @3x)
- Use SF Symbols where possible instead of custom icons
- Monitor and optimize memory usage, especially on older devices
- Implement appropriate caching strategies for network resources

### Performance
- Use Instruments to profile app performance (CPU, memory, energy usage)
- Ensure smooth scrolling in table/collection views with cell reuse
- Move heavy processing to background queues
- Implement pagination for large data sets

```swift
// Example of dispatching work to background queue
DispatchQueue.global(qos: .userInitiated).async {
    // Perform expensive operation
    let processedData = self.processLargeDataSet()
    
    DispatchQueue.main.async {
        // Update UI with results
        self.updateUI(with: processedData)
    }
}
```

## Accessibility

### General Guidelines

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brunogama/ios-cursor-rules](https://github.com/brunogama/ios-cursor-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
