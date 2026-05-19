---
trigger: always_on
description: Use the KVO pattern with KeyValueStore for all new persistent settings:
---


# User Defaults Settings Storage and Reading

## ✅ RECOMMENDED - KVO Pattern with KeyValueStore

Use the KVO pattern with KeyValueStore for all new persistent settings:

```swift
// ✅ CORRECT - KVO pattern with KeyValueStore
struct AppearancePreferencesUserDefaultsPersistor: AppearancePreferencesPersistor {

    enum Key: String {
        case newTabPageIsOmnibarVisible = "new-tab-page.omnibar.is-visible"
        case newTabPageIsProtectionsReportVisible = "new-tab-page.protections-report.is-visible"
        case userPreferences = "user.preferences"
        case lastUpdateCheck = "last.update.check"
    }

    private let keyValueStore: KeyValueStoring

    init(keyValueStore: KeyValueStoring) {
        self.keyValueStore = keyValueStore
    }

    var isOmnibarVisible: Bool {
        get { (try? keyValueStore.object(forKey: Key.newTabPageIsOmnibarVisible.rawValue) as? Bool) ?? true }
        set { try? keyValueStore.set(newValue, forKey: Key.newTabPageIsOmnibarVisible.rawValue) }
    }

    var isProtectionsReportVisible: Bool {
        get { (try? keyValueStore.object(forKey: Key.newTabPageIsProtectionsReportVisible.rawValue) as? Bool) ?? false }
        set { try? keyValueStore.set(newValue, forKey: Key.newTabPageIsProtectionsReportVisible.rawValue) }
    }

    var userPreferences: [String: String] {
        get { (try? keyValueStore.object(forKey: Key.userPreferences.rawValue) as? [String: String]) ?? [:] }
        set { try? keyValueStore.set(newValue, forKey: Key.userPreferences.rawValue) }
    }

    var lastUpdateCheck: Date {
        get { (try? keyValueStore.object(forKey: Key.lastUpdateCheck.rawValue) as? Date) ?? Date.distantPast }
        set { try? keyValueStore.set(newValue, forKey: Key.lastUpdateCheck.rawValue) }
    }
}
```

## Key Guidelines for KVO Pattern

1. **Use struct conforming to protocol** - Follow the persistor pattern
2. **Define keys as enum with String raw values** - Use kebab-case for key names
3. **Use KeyValueStoring protocol** - Not direct UserDefaults access
4. **Computed properties with get/set** - Handle storage operations in accessors
5. **Use try? for error handling** - KeyValueStore operations can throw
6. **Provide default values** - Use nil coalescing operator (??) for defaults
7. **Inject KeyValueStore in init** - Enable dependency injection and testing

## Advanced Pattern for Optional Values

```swift
// ✅ CORRECT - Optional values pattern
struct SettingsUserDefaultsPersistor: SettingsPersistor {

    enum Key: String {
        case optionalUserName = "user.name"
        case optionalTheme = "app.theme"
    }

    private let keyValueStore: KeyValueStoring

    init(keyValueStore: KeyValueStoring) {
        self.keyValueStore = keyValueStore
    }

    var optionalUserName: String? {
        get { try? keyValueStore.object(forKey: Key.optionalUserName.rawValue) as? String }
        set { 
            if let value = newValue {
                try? keyValueStore.set(value, forKey: Key.optionalUserName.rawValue)
            } else {
                try? keyValueStore.removeObject(forKey: Key.optionalUserName.rawValue)
            }
        }
    }

    var selectedTheme: Theme? {
        get { 
            guard let rawValue = try? keyValueStore.object(forKey: Key.optionalTheme.rawValue) as? String else { return nil }
            return Theme(rawValue: rawValue)
        }
        set { 
            if let value = newValue {
                try? keyValueStore.set(value.rawValue, forKey: Key.optionalTheme.rawValue)
            } else {
                try? keyValueStore.removeObject(forKey: Key.optionalTheme.rawValue)
            }
        }
    }
}
```

## Platform-Specific Storage

```swift
// ✅ CORRECT - Platform-specific KeyValueStore usage
struct PlatformSettingsUserDefaultsPersistor: PlatformSettingsPersistor {

    enum Key: String {
        case platformSpecificSetting = "platform.specific.setting"
    }

    private let keyValueStore: KeyValueStoring

    init(keyValueStore: KeyValueStoring) {
        self.keyValueStore = keyValueStore
    }

    var platformSpecificSetting: Bool {
        get { 
            #if os(iOS)
            return (try? keyValueStore.object(forKey: Key.platformSpecificSetting.rawValue) as? Bool) ?? false
            #elseif os(macOS)
            return (try? keyValueStore.object(forKey: Key.platformSpecificSetting.rawValue) as? Bool) ?? true
            #endif
        }
        set { 
            try? keyValueStore.set(newValue, forKey: Key.platformSpecificSetting.rawValue)
        }
    }
}
```

## 🚫 DEPRECATED - @UserDefaultsWrapper Pattern

The following pattern is deprecated and should not be used for new code:

```swift
// ❌ DEPRECATED - Do not use @UserDefaultsWrapper for new code
extension AppUserDefaults {
    @UserDefaultsWrapper(key: .newFeatureEnabled, defaultValue: false)
    var newFeatureEnabled: Bool
    
    @UserDefaultsWrapper(key: .lastUpdateCheck, defaultValue: Date.distantPast)
    var lastUpdateCheck: Date
}
```

## Migration from Property Wrappers

When migrating from `@UserDefaultsWrapper` to the KVO pattern:

1. **Create a new persistor struct** - Following the naming convention `*UserDefaultsPersistor`
2. **Define keys enum** - Convert string keys to enum cases

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duckduckgo/apple-browsers](https://github.com/duckduckgo/apple-browsers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
