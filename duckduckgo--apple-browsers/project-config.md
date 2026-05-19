---
trigger: always_on
description: DuckPlayer provides video playback within the app, separate from the web view-based player. The architecture separates concerns into distinct components using a presenter pattern, native UI views, and JavaScript integration for seamless video playback experiences.
---


# DuckPlayer Implementation Guide

## Overview

DuckPlayer provides video playback within the app, separate from the web view-based player. The architecture separates concerns into distinct components using a presenter pattern, native UI views, and JavaScript integration for seamless video playback experiences.

## Architecture Components

### Core Architecture Pattern

DuckPlayer follows a presenter-driven architecture with clear separation of concerns:

```swift
// ✅ CORRECT - Presenter coordinates between components
final class DuckPlayerNativeUIPresenter {
    private let navigationHandler: NativeDuckPlayerNavigationHandler
    private let state: DuckPlayerState
    private let pixelFiring: DuckPlayerPixelFiring
    
    func presentPlayer(for videoID: String) {
        // Coordinates pill presentation, player setup, and analytics
        updateState(videoID: videoID)
        configurePillType()
        firePixels()
    }
}

// ❌ INCORRECT - Don't manage all responsibilities in one view
struct DuckPlayerView: View {
    @State private var videoID: String = ""
    @State private var isPresented = false
    // Don't handle navigation, state, and analytics directly in views
}
```

### State Management Pattern

Use `DuckPlayerState` for centralized video state management:

```swift
// ✅ CORRECT - Centralized state management
final class DuckPlayerState {
    var videoID: String?
    var hasBeenShown: Bool = false
    var timestamp: TimeInterval?
    
    func reset() {
        videoID = nil
        hasBeenShown = false
        timestamp = nil
    }
}

// ❌ INCORRECT - Scattered state across components
struct DuckPlayerView: View {
    @State private var videoID: String = ""
    @State private var timestamp: TimeInterval = 0
    // Don't duplicate state management
}
```

## Component Responsibilities

### DuckPlayerNativeUIPresenter

**Role**: Primary coordinator and state manager for the Native UI

**Key Responsibilities**:
- Manages presentation lifecycle of player UI components
- Coordinates between pill types (welcome, entry, re-entry)
- Handles user interactions and navigation events
- Manages constraint updates and visibility state
- Integrates with app navigation and browser features
- Handles orientation changes and UI adaptations
- Manages pixel firing for analytics tracking
- Controls toast notifications and dismiss count tracking

```swift
// ✅ CORRECT - Presenter pattern implementation
final class DuckPlayerNativeUIPresenter {
    private weak var containerView: DuckPlayerContainer?
    private let navigationHandler: NativeDuckPlayerNavigationHandler
    private let state: DuckPlayerState
    private let pixelFiring: DuckPlayerPixelFiring
    
    func presentWelcomePill() {
        // Configure welcome pill for first-time users
        configureContainerForPill(.welcome)
        fireWelcomePillPixel()
    }
    
    func presentEntryPill(for videoID: String) {
        // Configure entry pill for returning users
        state.videoID = videoID
        configureContainerForPill(.entry)
        fireEntryPillPixel()
    }
    
    func presentReEntryPill(for videoID: String) {
        // Configure re-entry pill for previously watched videos
        state.videoID = videoID
        configureContainerForPill(.reEntry)
        fireReEntryPillPixel()
    }
}
```

### NativeDuckPlayerNavigationHandler

**Role**: Manages video playback navigation and browser integration

```swift
// ✅ CORRECT - Navigation handler pattern
final class NativeDuckPlayerNavigationHandler {
    private let webView: WKWebView
    private let presenter: DuckPlayerNativeUIPresenter
    
    func handleYouTubeURL(_ url: URL) -> Bool {
        guard shouldHandleNatively(url) else { return false }
        
        let videoID = extractVideoID(from: url)
        presenter.presentPlayer(for: videoID)
        return true
    }
    
    private func shouldHandleNatively(_ url: URL) -> Bool {
        // Check if URL should be handled by native player
        return isYouTubeURL(url) && isNativeUIEnabled()
    }
}

// ❌ INCORRECT - Don't handle navigation directly in views
struct DuckPlayerView: View {
    func handleURL(_ url: URL) {
        // Don't put navigation logic in views
    }
}
```

## View Architecture

### Pill Management System

DuckPlayer uses a three-tier pill system based on user interaction history:

```swift
// ✅ CORRECT - Pill type management
enum DuckPlayerPillType {
    case welcome    // First-time users (priming modal not yet presented)
    case entry      // Returning users viewing new videos
    case reEntry    // Users returning to previously watched videos
}

final class DuckPlayerContainer: UIView {
    private var currentPillType: DuckPlayerPillType?
    
    func configurePill(_ type: DuckPlayerPillType, for videoID: String) {
        switch type {
        case .welcome:
            presentWelcomePill()
        case .entry:
            presentEntryPill(videoID: videoID)
        case .reEntry:
            presentReEntryPill(videoID: videoID)
        }
    }
}
```

### SwiftUI View Components


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duckduckgo/apple-browsers](https://github.com/duckduckgo/apple-browsers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
