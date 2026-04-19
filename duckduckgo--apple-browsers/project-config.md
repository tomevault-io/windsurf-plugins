---
trigger: always_on
description: Use enum-based feature flags with protocols for type safety:
---


# Feature Flag Patterns

## Type-Safe Feature Flags
Use enum-based feature flags with protocols for type safety:

```swift
// ✅ CORRECT - Type-safe feature flags
protocol FeatureFlag: RawRepresentable where RawValue == String {
    var defaultValue: Bool { get }
    var description: String { get }
}

enum UIFeatureFlag: String, FeatureFlag {
    case newTabPageRedesign = "new_tab_page_redesign"
    case advancedPrivacySettings = "advanced_privacy_settings"
    case voiceSearch = "voice_search"
    case experimentalUI = "experimental_ui"
    
    var defaultValue: Bool {
        switch self {
        case .newTabPageRedesign: return false
        case .advancedPrivacySettings: return true
        case .voiceSearch: return false
        case .experimentalUI: return false
        }
    }
    
    var description: String {
        switch self {
        case .newTabPageRedesign: 
            return "Enable redesigned new tab page"
        case .advancedPrivacySettings: 
            return "Show advanced privacy settings"
        case .voiceSearch: 
            return "Enable voice search functionality"
        case .experimentalUI:
            return "Enable experimental UI components"
        }
    }
}

enum NetworkFeatureFlag: String, FeatureFlag {
    case networkProtectionV2 = "network_protection_v2"
    case enhancedBlocking = "enhanced_blocking"
    
    var defaultValue: Bool {
        switch self {
        case .networkProtectionV2: return false
        case .enhancedBlocking: return true
        }
    }
    
    var description: String {
        switch self {
        case .networkProtectionV2:
            return "Enable Network Protection V2"
        case .enhancedBlocking:
            return "Enhanced content blocking"
        }
    }
}
```

## FeatureFlagger Protocol Implementation
Extend the existing FeatureFlagger with type-safe methods:

```swift
// ✅ CORRECT - Type-safe FeatureFlagger extension
extension FeatureFlagger {
    func isEnabled<Flag: FeatureFlag>(_ flag: Flag) -> Bool {
        return isFeatureOn(flag.rawValue) ?? flag.defaultValue
    }
    
    func setEnabled<Flag: FeatureFlag>(_ enabled: Bool, for flag: Flag) {
        setFeatureOn(flag.rawValue, enabled: enabled)
    }
}

// Usage in code
final class FeatureViewModel: ObservableObject {
    private let featureFlagger: FeatureFlagger
    
    init(featureFlagger: FeatureFlagger) {
        self.featureFlagger = featureFlagger
    }
    
    func loadContent() {
        if featureFlagger.isEnabled(UIFeatureFlag.newTabPageRedesign) {
            loadNewDesign()
        } else {
            loadLegacyDesign()
        }
    }
}
```

## Feature Flag ViewModifier
Create SwiftUI modifiers for conditional UI:

```swift
// ✅ ADVANCED - SwiftUI feature flag modifier
struct FeatureFlagModifier<Flag: FeatureFlag>: ViewModifier {
    let flag: Flag
    let featureFlagger: FeatureFlagger
    let fallback: () -> AnyView
    
    func body(content: Content) -> some View {
        if featureFlagger.isEnabled(flag) {
            content
        } else {
            fallback()
        }
    }
}

extension View {
    func featureFlag<Flag: FeatureFlag>(
        _ flag: Flag,
        featureFlagger: FeatureFlagger,
        @ViewBuilder fallback: @escaping () -> some View = { EmptyView() }
    ) -> some View {
        modifier(FeatureFlagModifier(
            flag: flag,
            featureFlagger: featureFlagger,
            fallback: { AnyView(fallback()) }
        ))
    }
}

// Usage
struct ContentView: View {
    @Environment(\.dependencies) var dependencies
    
    var body: some View {
        VStack {
            NewFeatureView()
                .featureFlag(UIFeatureFlag.experimentalUI, 
                           featureFlagger: dependencies.featureFlagger) {
                    LegacyFeatureView()
                }
        }
    }
}
```

## Feature Flag Property Wrapper
Create a property wrapper for reactive feature flags:

```swift
// ✅ ADVANCED - Reactive feature flag property wrapper
@propertyWrapper
struct FeatureFlagState<Flag: FeatureFlag>: DynamicProperty {
    @ObservedObject private var flagger: ObservableFeatureFlagger
    private let flag: Flag
    
    var wrappedValue: Bool {
        get { flagger.isEnabled(flag) }
        nonmutating set { flagger.setEnabled(newValue, for: flag) }
    }
    
    var projectedValue: Binding<Bool> {
        Binding(
            get: { wrappedValue },
            set: { wrappedValue = $0 }
        )
    }
    
    init(_ flag: Flag) {
        self.flag = flag
        self._flagger = ObservedObject(wrappedValue: ObservableFeatureFlagger.shared)
    }
}

// Usage in SwiftUI views
struct SettingsView: View {
    @FeatureFlagState(UIFeatureFlag.voiceSearch) var voiceSearchEnabled
    @FeatureFlagState(UIFeatureFlag.experimentalUI) var experimentalUIEnabled
    
    var body: some View {
        Form {
            Toggle("Voice Search", isOn: $voiceSearchEnabled)
            Toggle("Experimental UI", isOn: $experimentalUIEnabled)
        }
    }
}
```

## A/B Testing Integration
Integrate feature flags with A/B testing:

```swift
// ✅ ADVANCED - A/B testing with feature flags
enum ABTestVariant: String, CaseIterable {
    case control = "control"
    case variantA = "variant_a"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/duckduckgo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
