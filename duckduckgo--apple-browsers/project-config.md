---
trigger: always_on
description: This document provides a comprehensive overview of how content blocking has been implemented on iOS to protect user privacy and block tracking attempts. Our implementation uses a dual-approach strategy that combines the efficiency of WebKit's native content blocking with the flexibility of JavaScript-based protection.
---


# iOS Tracker Blocking Implementation Guide

## Overview

This document provides a comprehensive overview of how content blocking has been implemented on iOS to protect user privacy and block tracking attempts. Our implementation uses a dual-approach strategy that combines the efficiency of WebKit's native content blocking with the flexibility of JavaScript-based protection.

**Implementation Strategy:**
- **Primary Layer**: Content Blocker Rules (WebKit native, compiled to bytecode)
- **Secondary Layer**: JavaScript injection with Tracker Radar data (gap coverage + surrogates)

## Content Blocker Rules

### Architecture Overview

Content Blocker Rules form the primary layer of our tracker blocking implementation. These rules are converted from our Tracker Radar dataset into Apple's Content Blocker Rules format and compiled by WebKit into efficient bytecode for optimal performance.

**Key Characteristics:**
- ✅ **High Performance**: Rules are compiled to bytecode by WebKit
- ✅ **Low Memory Footprint**: Optimized for mobile devices
- ✅ **Battery Efficient**: Minimal CPU overhead
- ❌ **Limited Flexibility**: Cannot support complex logic or surrogates
- ❌ **No Runtime Modification**: Rules are static once compiled

### Implementation Details

#### ContentBlockerRulesManager

The `ContentBlockerRulesManager` is responsible for converting Tracker Radar data into Apple's Content Blocker format and managing the rule compilation process.

```swift
import WebKit
import BrowserServicesKit

final class ContentBlockerRulesManager {
    private let trackerDataManager: TrackerDataManager
    private let compilationQueue = DispatchQueue(label: "content-blocker-compilation", qos: .utility)
    
    init(trackerDataManager: TrackerDataManager) {
        self.trackerDataManager = trackerDataManager
    }
    
    /// Converts Tracker Radar data to Content Blocker Rules format
    func generateContentBlockerRules() async throws -> [WKContentRuleList] {
        return try await withCheckedThrowingContinuation { continuation in
            compilationQueue.async {
                do {
                    let trackerData = self.trackerDataManager.embeddedTrackerData
                    let rules = self.convertToContentBlockerFormat(trackerData)
                    let ruleList = try self.compileRules(rules)
                    continuation.resume(returning: [ruleList])
                } catch {
                    continuation.resume(throwing: error)
                }
            }
        }
    }
    
    /// Converts TrackerData to Apple's Content Blocker Rules JSON format
    private func convertToContentBlockerFormat(_ trackerData: TrackerData) -> [[String: Any]] {
        var rules: [[String: Any]] = []
        
        for (domain, tracker) in trackerData.trackers {
            for rule in tracker.rules ?? [] {
                let contentBlockerRule: [String: Any] = [
                    "trigger": [
                        "url-filter": rule.rule,
                        "resource-type": rule.resourceTypes,
                        "if-domain": rule.whitelist?.compactMap { "*\($0)" }
                    ].compactMapValues { $0 },
                    "action": [
                        "type": "block"
                    ]
                ]
                rules.append(contentBlockerRule)
            }
        }
        
        return rules
    }
    
    /// Compiles rules using WebKit's WKContentRuleListStore
    private func compileRules(_ rules: [[String: Any]]) throws -> WKContentRuleList {
        let jsonData = try JSONSerialization.data(withJSONObject: rules)
        let jsonString = String(data: jsonData, encoding: .utf8)!
        
        return try await withCheckedThrowingContinuation { continuation in
            WKContentRuleListStore.default().compileContentRuleList(
                forIdentifier: "DuckDuckGoContentBlocker",
                encodedContentRuleList: jsonString
            ) { ruleList, error in
                if let error = error {
                    continuation.resume(throwing: error)
                } else if let ruleList = ruleList {
                    continuation.resume(returning: ruleList)
                } else {
                    continuation.resume(throwing: ContentBlockerError.compilationFailed)
                }
            }
        }
    }
}

enum ContentBlockerError: Error {
    case compilationFailed
    case invalidRuleFormat
    case trackerDataUnavailable
}
```

#### Integration with WKWebView

Content Blocker Rules are applied to WKWebView through the `WKUserContentController`:

```swift
extension BrowserWebView {
    func applyContentBlockerRules() async {
        do {
            let ruleLists = try await contentBlockerManager.generateContentBlockerRules()
            
            await MainActor.run {
                for ruleList in ruleLists {
                    webView.configuration.userContentController.add(ruleList)
                }

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duckduckgo/apple-browsers](https://github.com/duckduckgo/apple-browsers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
