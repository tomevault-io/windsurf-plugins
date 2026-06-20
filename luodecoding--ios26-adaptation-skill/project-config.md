---
trigger: always_on
description: iOS 26 adaptation expert. Guides through SDK build adaptation and Liquid Glass design language migration. Handles SceneDelegate architecture, deprecated API replacement, and two-phase adaptation strategy.
---


# iOS 26 Adaptation Expert

## Critical Information

### Deadlines

| Date | Requirement | Phase |
|------|-------------|-------|
| **2026-04-28** | Must build with iOS 26 SDK | Phase 1 |
| **~2026-09** | Liquid Glass mandatory, `UIDesignRequiresCompatibility` removed | Phase 2 |

### Common Misconceptions

- ❌ **Deployment Target change required**: No, keep your current minimum version
- ❌ **Users forced to iOS 26**: No, runtime requirement unchanged
- ❌ **Existing apps removed**: No, only affects new submissions
- ❌ **Grace period exists**: No, April 28 is a hard deadline

---

## Decision Framework

### Step 1: Determine Adaptation Strategy

Based on your next release date:

```
┌─────────────────────────────────────────────┐
│  When is your next app release?              │
└─────────────────────┬───────────────────────┘
                      │
    ┌─────────────────┼─────────────────┐
    │                 │                 │
    ▼                 ▼                 ▼
┌──────────┐   ┌──────────────┐   ┌──────────────┐
│ Before   │   │ 2026-04-28 ~ │   │ After Xcode  │
│ 2026-04- │   │ Xcode 27     │   │ 27 (~2026-09)│
│ 28       │   │              │   │              │
└────┬─────┘   └──────┬───────┘   └──────┬───────┘
     │                │                  │
     ▼                ▼                  ▼
┌──────────┐   ┌──────────────┐   ┌──────────────┐
│ Strategy │   │ Strategy B   │   │ Strategy C   │
│    A     │   │              │   │              │
└──────────┘   └──────────────┘   └──────────────┘
```

### Strategy A: Branch-based Adaptation (Before 2026-04-28 release)

**When to use**: You have a release planned before April 28, 2026

**Approach**:
1. Keep main branch unchanged for current release
2. Create `feature/ios26-adaptation` branch
3. Complete Phase 1 adaptation in branch
4. Merge after April 28 deadline

**Branch Commands**:
```bash
git checkout main
git checkout -b feature/ios26-adaptation
# Complete adaptation work
git checkout main
git merge feature/ios26-adaptation
```

### Strategy B: Phase 1 Required, Phase 2 Evaluated (April 28 ~ Xcode 27)

**When to use**: Release between April 28 and Xcode 27 launch

**Approach**:
1. Must complete Phase 1 (SDK build adaptation)
2. Temporarily disable Liquid Glass
3. Evaluate Phase 2 based on pre-Xcode 27 releases

### Strategy C: Combined Phases (After Xcode 27)

**When to use**: No release until after Xcode 27

**Approach**:
1. Complete both phases in single iteration
2. No temporary disabling needed
3. Full Liquid Glass adaptation upfront

---

## Phase 1: SDK Build Adaptation

### Goal
Build with iOS 26 SDK, maintain existing UI appearance

### Key Tasks

#### 1. Environment Setup

| Tool | Minimum Version |
|------|-----------------|
| Xcode | 26.0+ (recommend 26.3) |
| macOS | Sequoia 15.3+ |
| iOS SDK | 26.0+ |

#### 2. Deprecated API Replacement

| Deprecated API | Replacement | Severity |
|---------------|-------------|----------|
| `keyWindow` | Unified window access interface | Error |
| `delegate.window` | Unified window access interface | Error |
| `UNNotificationPresentationOptionAlert` | `Banner \| List` (iOS 14.0+) | Warning |
| `UNAuthorizationOptionAlert` | Still valid — do NOT replace | — |
| `UIScreen.main` | `UIWindowScene.screen` (iOS 13+) | Warning |

#### 3. SceneDelegate Architecture

**Required Changes**:

1. **Create UIApplication Extension** (Unified Access)
   - `mainWindow()` - Get current window
   - `visibleViewController()` - Get top visible VC
   - `currentNavigationController()` - Get current nav controller

2. **Modify AppDelegate**
   - Add `sharedInstance` class method
   - Separate `setupApplication(launchOptions:)` method
   - Separate `setupSceneUI(window:)` method
   - Add Scene Session configuration

3. **Create/Modify SceneDelegate**
   - Window creation in `willConnectTo`
   - Forward to AppDelegate for business setup
   - Lifecycle event forwarding

4. **Global Code Replacement**
   - Replace all `keyWindow` calls
   - Replace all `delegate.window` calls
   - Update window-based navigation

#### 4. Info.plist Configuration

```xml
<!-- Temporarily disable Liquid Glass -->
<key>UIDesignRequiresCompatibility</key>
<true/>

<!-- SceneDelegate configuration -->
<key>UIApplicationSceneManifest</key>
<dict>
    <key>UIApplicationSupportsMultipleScenes</key>
    <false/>
    <key>UISceneConfigurations</key>
    <dict>
        <key>UIWindowSceneSessionRoleApplication</key>
        <array>
            <dict>
                <key>UISceneConfigurationName</key>
                <string>Default Configuration</string>
                <key>UISceneDelegateClassName</key>
                <string>SceneDelegate</string>
            </dict>
        </array>
    </dict>
</dict>
```

#### 5. Complete Implementation Examples

Select the section matching your project's primary language:

- [Swift Projects](#swift-projects)
- [Objective-C Projects](#objective-c-projects)
- [Mixed (Swift/Objective-C) Projects](#mixed-swiftobjective-c-projects)

Production-ready templates for each are in `templates/swift/`, `templates/objc/`, and `templates/mixed/`.

##### Swift Projects


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [luodeCoding/ios26-adaptation-skill](https://github.com/luodeCoding/ios26-adaptation-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
