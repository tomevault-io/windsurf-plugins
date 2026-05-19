---
trigger: always_on
description: *This guide covers testing practices and patterns for the DuckDuckGo browser on iOS and macOS platforms.*
---


# Testing Guidelines & Best Practices

*This guide covers testing practices and patterns for the DuckDuckGo browser on iOS and macOS platforms.*

## 🚨 MANDATORY: Testing Execution Rules

### NEVER Run Tests Without Permission
**NEVER execute any test commands without EXPLICIT user permission or unles user explicitly asked to in their prompt.**

#### Required Testing Workflow:
1. Write or modify test code as requested
2. if user did not ask to run tests in their prompt, **STOP** before running any test commands:
   - `swift test`
   - `npm test` 
   - `xcodebuild test`
   - `fastlane test`
   - Any other test execution commands
3. **ASK** the user: "Should I run the tests?"
4. **WAIT** for explicit permission (e.g., "yes", "run tests", "test it")
5. Only then execute test commands

**This rule applies to ALL test execution - unit tests, integration tests, UI tests, performance tests, etc.**

---

## Future Improvements

This guide is a living document. Consider these areas for future improvements:

- **Tab Extensions Testing**: Expand patterns for testing complex tab extension interactions and lifecycle management
- **WebKit Integration Testing**: Add comprehensive patterns for testing WKWebView configurations, user scripts, and content blocking integration
- **Privacy Feature Testing**: Develop specialized testing approaches for tracker protection, HTTPS upgrade, and content blocking rule validation
- **Cross-Platform Testing**: Create patterns for testing SharedPackages functionality across iOS and macOS with consistent behavior validation
- **Fire Button Integration Testing**: Add comprehensive testing patterns for data clearing workflows across all browser components
- **Autofill and Credential Testing**: Expand testing approaches for AutofillCredentialProvider, password management, and form filling scenarios
- **Sync Testing**: Develop patterns for testing bookmark sync, conflict resolution, and cross-device data consistency
- **AI Chat Integration Testing**: Add testing patterns for AI chat functionality, context management, and user interaction flows
- **Feature Flag Testing**: Expand MockFeatureFlagger usage patterns and integration testing with real feature configurations

## Unit Tests

### What to Include

Unit tests should focus on testing individual components, functions, or classes in isolation. They should be:

- **Fast**: Run quickly (< 1 second per test)
- **Independent**: Not depend on external systems or other tests
- **Deterministic**: Always produce the same result given the same input
- **Focused**: Test one specific behavior or functionality

### ✅ When to Write Unit Tests

#### Model Logic
Testing business logic, data transformations, and model behavior:

```swift
func testBookmarkFolderCreation() {
    let folder = BookmarkFolder(title: "Test Folder")
    XCTAssertEqual(folder.title, "Test Folder")
    XCTAssertTrue(folder.children.isEmpty)
}
```

#### Algorithms/Parsers
Testing parsing logic, URL manipulation, search algorithms:

```swift
func testURLSchemeDetection() {
    let detector = URLSchemeDetector()
    XCTAssertTrue(detector.isValidURL("https://duckduckgo.com"))
    XCTAssertFalse(detector.isValidURL("invalid-url"))
}
```

#### Utility Functions
Testing helper functions, extensions, formatters:

```swift
func testDateFormatter() {
    let formatter = DateFormatter.shortDate
    let date = Date(timeIntervalSince1970: 1640995200) // 2022-01-01
    XCTAssertEqual(formatter.string(from: date), "1/1/22")
}
```

#### State Management
Testing ViewModels, state transitions, and data flow:

```swift
func testViewModelStateTransition() {
    let viewModel = SearchViewModel()
    viewModel.performSearch("test query")
    XCTAssertEqual(viewModel.state, .loading)
}
```

### ❌ What to Avoid

#### Simple Property Toggles
Testing trivial getters/setters:

```swift
// ❌ DON'T test this
func testIsEnabledToggle() {
    feature.isEnabled = true
    XCTAssertTrue(feature.isEnabled)
}
```

#### Complex UI Interactions
Use Integration or UI tests instead.

#### External Dependencies
File system, network calls, databases.

#### State/Strategy Pattern Switching
These are better suited for integration tests:

```swift
// ❌ DON'T test state switching in unit tests
func testStateSwitching() {
    stateMachine.transition(to: .loading)
    stateMachine.transition(to: .loaded)
    // This is brittle and doesn't test real behavior
}
```

## Mocks and Test Helpers

The DuckDuckGo browser project includes multiple mock categories for testing different components and scenarios:

### Mock Categories

#### Unit Tests Mocks
For testing individual components in isolation:
- **UI mocks**: MockWindow, MockTabViewItemDelegate
- **WebView mocks**: WebViewMock, WKSecurityOriginMock
- **Storage mocks**: FileStoreMock, UserDefaultsMock
- **Feature-specific mocks**: MockBookmarkManager, MockFireproofDomains, MockAIChatPreferencesStorage

#### Integration Tests Mocks
For testing component interactions and workflows:
- Content blocking mocks
- Tab navigation mocks
- Fire integration mocks
- Onboarding flow mocks
- System integration mocks

#### BSK Tests Mocks
For testing BrowserServicesKit functionality:
- **Feature flag mocks**: MockFeatureFlagger

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duckduckgo/apple-browsers](https://github.com/duckduckgo/apple-browsers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
