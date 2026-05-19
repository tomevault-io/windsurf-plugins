---
trigger: always_on
description: DuckPlayer uses two primary UserScript components to bridge native iOS functionality with web content:
---


# DuckPlayer UserScript Integration Guide

## Overview

DuckPlayer uses two primary UserScript components to bridge native iOS functionality with web content:
- `DuckPlayerUserScriptYouTube`: Manages communication with YouTube.com pages
- `DuckPlayerUserScriptPlayer`: Handles communication within the DuckPlayer web view

## Architecture Overview

### UserScript Communication Flow

```swift
// Communication flow:
// Web Content -> UserScript -> Native Handler -> ViewModel/Presenter
// Native UI -> Publisher -> UserScript -> Web Content

// ✅ CORRECT - Bidirectional communication pattern
final class DuckPlayerUserScriptYouTube: NSObject, Subfeature {
    // Incoming: Web -> Native
    func handler(forMethodNamed methodName: String) -> Subfeature.Handler? {
        switch methodName {
        case "onCurrentTimeStamp": return onCurrentTimeStamp
        case "onYoutubeError": return onYoutubeError
        default: return nil
        }
    }
    
    // Outgoing: Native -> Web
    private func pushToWebView(method: String, params: [String: String]) {
        broker?.push(method: method, params: params, for: self, into: webView)
    }
}
```

## DuckPlayerUserScriptYouTube Integration

### Component Responsibilities

**Primary Role**: Bridge between YouTube.com pages and native DuckPlayer controls

**Key Responsibilities**:
- Manages media control events (play/pause)
- Handles audio muting state
- Tracks video timestamp updates
- Responds to URL changes
- Manages script readiness state with event queuing
- Provides initial setup configuration

### Event Queuing System

The UserScript implements an event queuing system to handle events before scripts are ready:

```swift
// ✅ CORRECT - Event queuing implementation
private enum QueuedEvent {
    case mediaControl(pause: Bool)
    case muteAudio(mute: Bool)
    case urlChanged(pageType: String)
}

private var otherEventsQueue: [QueuedEvent] = []
private var areScriptsReady = false

private func handleEvent(_ event: QueuedEvent) {
    switch event {
    case .urlChanged:
        // URL changes are always processed immediately
        processEvent(event)
    default:
        if areScriptsReady {
            processEvent(event)
        } else {
            // Queue events until scripts are ready
            otherEventsQueue.append(event)
        }
    }
}

// Process queued events when scripts become ready
func onDuckPlayerScriptsReady(params: Any, original: WKScriptMessage) -> Encodable? {
    areScriptsReady = true
    while !otherEventsQueue.isEmpty {
        let event = otherEventsQueue.removeFirst()
        processEvent(event)
    }
    return nil
}
```

### Publisher Integration Pattern

```swift
// ✅ CORRECT - Reactive publisher pattern
private func setupSubscriptions() {
    duckPlayer?.mediaControlPublisher
        .sink { [weak self] pause in
            self?.handleMediaControl(pause: pause)
        }
        .store(in: &cancellables)
    
    duckPlayer?.muteAudioPublisher
        .sink { [weak self] mute in
            self?.handleMuteAudio(mute: mute)
        }
        .store(in: &cancellables)
    
    duckPlayer?.urlChangedPublisher
        .sink { [weak self] url in
            self?.onUrlChanged(url: url)
        }
        .store(in: &cancellables)
}
```

### Message Origin Security

```swift
// ✅ CORRECT - Strict origin validation
let messageOriginPolicy: MessageOriginPolicy = .only(rules: [
    .exact(hostname: DuckPlayerSettingsDefault.OriginDomains.duckduckgo),
    .exact(hostname: DuckPlayerSettingsDefault.OriginDomains.youtube),
    .exact(hostname: DuckPlayerSettingsDefault.OriginDomains.youtubeMobile),
    .exact(hostname: DuckPlayerSettingsDefault.OriginDomains.youtubeWWW),
    .exact(hostname: DuckPlayerSettingsDefault.OriginDomains.youtubeNoCookie),
    .exact(hostname: DuckPlayerSettingsDefault.OriginDomains.youtubeNoCookieWWW)
])
```

### Page Type Detection

```swift
// ✅ CORRECT - URL-based page type detection
func onUrlChanged(url: URL) {
    areScriptsReady = false
    
    // Determine page type for proper script behavior
    let pageType = DuckPlayerUserScript.getPageType(url: url)
    let shouldClearEvents = pageType != DuckPlayerUserScript.PageType.YOUTUBE
    
    if shouldClearEvents {
        // Clear queued events when navigating away from YouTube
        otherEventsQueue.removeAll()
    }
    
    // Always store the latest URL change event
    handleEvent(.urlChanged(pageType: pageType))
}
```

## DuckPlayerUserScriptPlayer Integration

### Component Responsibilities

**Primary Role**: Handle communication within the DuckPlayer web view

**Key Responsibilities**:
- Provides initial setup configuration
- Updates video timestamps to the view model
- Handles YouTube error states
- Manages locale and page type information

### ViewModel Communication

```swift
// ✅ CORRECT - Direct view model updates
@MainActor
private func onCurrentTimeStamp(params: Any, original: WKScriptMessage) -> Encodable? {
    guard let dict = params as? [String: Any],
          let timeString = dict["timestamp"] as? String,
          let timeInterval = Double(timeString) else {
        return [:] as [String: String]
    }
    
    // Update view model directly

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duckduckgo/apple-browsers](https://github.com/duckduckgo/apple-browsers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
