---
trigger: always_on
description: *This guide covers UI testing practices and patterns specifically for the DuckDuckGo macOS browser.*
---


# UI Testing Guidelines & Best Practices

*This guide covers UI testing practices and patterns specifically for the DuckDuckGo macOS browser.*

## Overview

UI Tests verify the end-to-end user experience and interface behavior. They test user workflows, navigation patterns, window management, and complex interactions across the entire application.

## Setting Up UI Tests

### ❗Always Use UITestCase Base Class

```swift
class FeatureUITests: UITestCase {  // ✅ Use UITestCase, not XCTestCase
    private var app: XCUIApplication!
    
    override func setUpWithError() throws {
        continueAfterFailure = false
        app = XCUIApplication.setUp()  // ✅ Use setUp(), never launch() directly
        // app is already launched and configured by setUp()
    }
}
```

**Why UITestCase is Required**:
- Provides proper app lifecycle management
- Handles feature flag configuration
- Sets up test server environment
- Manages window state and cleanup
- Provides debugging utilities

### Feature Flag Configuration

```swift
// Configure feature flags during test setup
override func setUpWithError() throws {
    app = XCUIApplication.setUp(featureFlags: [
        "contextualOnboarding": true,
        "visualUpdates": false,
        "duckPlayer": true
    ])
    // Feature flags are automatically applied via FEATURE_FLAGS environment variable
}

// Alternative: Custom environment
app = XCUIApplication.setUp(environment: [
    "UITEST_MODE_ONBOARDING": "1"
], featureFlags: [
    "newTabPageSections": true
])
```

#### Privacy Subfeature Configuration

```swift
// Configure privacy subfeatures (separate from feature flags)
override func setUpWithError() throws {
    app = XCUIApplication.setUp(privacySubfeatures: [
        "autoconsent-filterlist": true,
        "tracker-allowlist": true
    ])
    // Privacy subfeatures are applied via PRIVACY_SUBFEATURES environment variable
}

// Combined feature flags and privacy subfeatures
app = XCUIApplication.setUp(
    featureFlags: [
        "contextualOnboarding": true
    ],
    privacySubfeatures: [
        "autoconsent-filterlist": true
    ]
)
```

**❗Why Feature Flag and Privacy Subfeature Configuration is Critical**:
- UI tests run against notarized builds - feature flags can't be changed at runtime
- MockFeatureFlagger is NOT available in UI tests (only real DefaultFeatureFlagger)
- Feature flags must be configured via FEATURE_FLAGS environment variable before app launch
- **Privacy subfeatures are controlled by PrivacyConfiguration, not feature flags**
- Privacy subfeatures must be configured via PRIVACY_SUBFEATURES environment variable
- Incorrect feature/subfeature state will cause UI tests to fail when expected UI elements don't appear

**Key Differences**:
- **Feature Flags**: Control app features (e.g., `contextualOnboarding`, `duckPlayer`)
- **Privacy Subfeatures**: Control privacy functionality (e.g., `autoconsent-filterlist`, `tracker-allowlist`)
- Both use separate environment variables and configuration systems

### File Management in UI Tests

The `UITestCase` base class provides built-in file management capabilities for handling downloads, temporary files, and other file operations during testing.

**Important**: UI tests run in a sandboxed environment and cannot directly read or delete files from user directories using standard FileManager calls. For non-temp directories, use the `filesToCleanup` pattern that's handled in the base class `tearDown()`.

#### Automatic File Cleanup

All UI test classes automatically clean up tracked files after test completion:

```swift
class DownloadsUITests: UITestCase {
    func testFileDownload() {
        let downloadsDir = FileManager.default.urls(for: .downloadsDirectory, in: .userDomainMask)[0]
        let fileName = "test-file.json"
        let filePath = downloadsDir.appendingPathComponent(fileName).path
        
        // Track file for automatic cleanup
        trackForCleanup(filePath)
        
        // Perform download test...
        // File will be automatically cleaned up after test completes
    }
}
```

#### Reading Files via Local Server

Use `readFileViaLocalServer()` to read files that may have permission restrictions:

```swift
func testJSONFileContent() throws {
    let filePath = "/Users/admin/Downloads/test-results.json"
    
    // Read file via local test server (bypasses permission issues)
    let jsonData = try readFileViaLocalServer(filePath: filePath)
    let results = try JSONDecoder().decode(TestResults.self, from: jsonData)
    
    // Validate file contents
    XCTAssertFalse(results.items.isEmpty)
}
```

#### File Management Best Practices

```swift
class FileBasedUITests: UITestCase {
    func testCompleteFileWorkflow() throws {
        // 1. Track all files that will be created
        let tempDir = FileManager.default.temporaryDirectory.appendingPathComponent("test-files")
        trackForCleanup(tempDir.path)
        
        let downloadedFile = "/Users/admin/Downloads/results.json"
        trackForCleanup(downloadedFile)
        
        // 2. Perform file operations
        try FileManager.default.createDirectory(at: tempDir, withIntermediateDirectories: true)
        
        // 3. Read files via server if needed

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duckduckgo/apple-browsers](https://github.com/duckduckgo/apple-browsers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
