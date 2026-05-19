---
trigger: always_on
description: Define a concrete pattern for removing `.shared` singletons in the macOS app by replacing them with app-owned instances and dependency injection. The `AIChatPreferences` and `AboutPreferences` refactors in `AppDelegate` are canonical examples.
---


## macOS Singleton Removal Rules

### Purpose

Define a concrete pattern for removing `.shared` singletons in the macOS app by replacing them with app-owned instances and dependency injection. The `AIChatPreferences` and `AboutPreferences` refactors in `AppDelegate` are canonical examples.

### Rules

1. **Do not introduce new singletons**
   - Never add new `static let shared` or similar global singletons.
   - New dependencies must be passed in via initializers or factory methods, not fetched from global state.

2. **Move ownership to the composition root (AppDelegate)**
   - Add a stored property on the macOS composition root (currently `AppDelegate`) for the dependency, for example:
     - `let aiChatPreferences: AIChatPreferences`
   - Construct the instance during app setup using real dependencies:
     - Inject storage (e.g. `DefaultAIChatPreferencesStorage`)
     - Inject configuration objects (e.g. `AIChatMenuVisibilityConfigurable`)
     - Inject window managers via protocols (e.g. `WindowControllersManagerProtocol`)
     - Inject feature flaggers and other services as needed
   - Prefer protocol-typed properties in `AppDelegate` when the dependency has a clear protocol (to keep testing and substitution easy).

3. **Thread the dependency through initializers**
   - For view controllers and models that need the former singleton, add initializer parameters and store them as non-optional properties. Example:
     - `init(..., aiChatPreferences: AIChatPreferences = NSApp.delegateTyped.aiChatPreferences, ...)`
   - Avoid using `NSApp.delegateTyped` or `Application.appDelegate` and prefer to pass the dependency down from a parent object.
     - If a parent object doesn't contain the dependency, it should be updated to have it passed down from its own parent object, observing the exceptions mentioned below.
   - It is explicitly allowed to use `NSApp.delegateTyped`:
     - In the `Tab` initializer (following the existing pattern for other dependencies)
     - In default parameter values for the `MainViewController` initializer (this is the entry point for the dependency chain)
     - In default parameter values for the `TabCollectionViewModel` initializer (following the existing pattern for other dependencies)
     - In default parameter values for the `TabViewModel` initializer (temporary exception, will be refactored later)
     - **Exception**: For `@MainActor` initializers, use optional parameters with `nil` defaults and assign from `NSApp.delegateTyped` in the initializer body.
   - **Important: Main actor isolation**: If an initializer is marked `@MainActor` and you need to default a parameter from `NSApp.delegateTyped`, use an optional parameter with `nil` default instead of accessing `NSApp.delegateTyped` in the default value. Then assign the value inside the initializer body:
     - ❌ `init(savedZoomLevelsCoordinating: SavedZoomLevelsCoordinating = NSApp.delegateTyped.accessibilityPreferences)` (causes main actor isolation warning)
     - ✅ `init(savedZoomLevelsCoordinating: SavedZoomLevelsCoordinating? = nil)` with `self.savedZoomLevelsCoordinating = savedZoomLevelsCoordinating ?? NSApp.delegateTyped.accessibilityPreferences` in the body
   - When creating child objects from a parent that already has the dependency, pass the property down rather than re-reading from `NSApp.delegateTyped`.
   - **For preferences models that need to reach SwiftUI views**, thread through the entire chain:
     - `MainViewController` (with default parameter) → `BrowserTabViewController` → `PreferencesViewController` → `PreferencesSidebarModel` → `PreferencesRootView`
     - Follow the existing pattern used by other preferences (e.g., `searchPreferences`, `tabsPreferences`, `aiChatPreferences`)
     - When adding to `PreferencesSidebarModel`, add the property alongside existing preferences and update both the main `init` and convenience `init`
   - **For dependencies that need to reach UserScripts initialization** (e.g., `DuckPlayerPreferences`), thread through the content blocking infrastructure:
     - `AppDelegate` → `AppContentBlocking` → `UserContentUpdating` → `ScriptSourceProvider` (via `ScriptSourceProviding` protocol) → `UserScripts`
     - Add the dependency to `ScriptSourceProviding` protocol as a property
     - Add it to `ScriptSourceProvider` struct (property and initializer parameter)
     - Add it to `UserContentUpdating` initializer and pass to `ScriptSourceProvider` in `makeValue` closure
     - Add it to `AppContentBlocking` initializers (both convenience and main) and pass to `UserContentUpdating`
     - Pass it from `AppDelegate` to `AppContentBlocking` initialization
     - In `UserScripts`, access via `sourceProvider.duckPlayerPreferences` instead of using a default parameter
     - This follows the same pattern as `WebTrackingProtectionPreferences` and `CookiePopupProtectionPreferences`

4. **Update utility code and extensions carefully**
   - For helpers like `URL` extensions where dependency injection is impractical, read the instance from the composition root instead of a singleton:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duckduckgo/apple-browsers](https://github.com/duckduckgo/apple-browsers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
