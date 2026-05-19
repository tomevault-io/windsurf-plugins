---
trigger: always_on
description: DuckDuckGo's subscription system provides access to premium features including VPN (Network Protection), Personal Information Removal (PIR), Identity Theft Restoration (ITR), and AI Chat. The system supports multiple purchase platforms and cross-platform activation.
---


# Subscription Architecture & Implementation

## Overview
DuckDuckGo's subscription system provides access to premium features including VPN (Network Protection), Personal Information Removal (PIR), Identity Theft Restoration (ITR), and AI Chat. The system supports multiple purchase platforms and cross-platform activation.

## Core Architecture

### Shared Foundation: BrowserServicesKit
All subscription logic is centralized in `BrowserServicesKit/Sources/Subscription/`:

```swift
// ✅ CORRECT - Use BrowserServicesKit for core subscription logic
import BrowserServicesKit

final class SubscriptionViewModel: ObservableObject {
    private let subscriptionManager: SubscriptionManager
    
    init(subscriptionManager: SubscriptionManager = SubscriptionManager.shared) {
        self.subscriptionManager = subscriptionManager
    }
}

// ❌ INCORRECT - Don't duplicate subscription logic in platform code
final class SubscriptionViewModel: ObservableObject {
    func checkSubscriptionStatus() {
        // Don't reimplement subscription logic
    }
}
```

### Platform-Specific Purchase Methods

#### iOS
- **Purchase Method**: App Store only (StoreKit)
- **Geographic Coverage**: Global
- **Cross-Platform**: Can activate Stripe subscriptions from other platforms

#### macOS App Store Build
- **Purchase Method**: App Store only (StoreKit)
- **Geographic Coverage**: Global
- **Cross-Platform**: Can activate Stripe subscriptions

#### macOS Direct Download Build
- **US Users**: Stripe web purchases
- **Non-US Users**: Redirected to iOS App Store
- **Cross-Platform**: Primary platform for Stripe purchases

### Version Management
ALWAYS use V2 implementations for new code:

```swift
// ✅ CORRECT - Use V2 implementations
let subscriptionManager = SubscriptionManagerV2()
let purchaseManager = StorePurchaseManagerV2()
let purchaseFlow = AppStorePurchaseFlowV2()

// ❌ INCORRECT - Don't use V1 implementations
let subscriptionManager = SubscriptionManager() // Legacy
let purchaseFlow = AppStorePurchaseFlow() // Legacy
```

## Premium Features Implementation

### Feature Entitlements
```swift
// ✅ CORRECT - Check entitlements through SubscriptionManager
final class FeatureViewModel: ObservableObject {
    private let subscriptionManager: SubscriptionManager
    
    var isFeatureEnabled: Bool {
        subscriptionManager.hasEntitlement(for: .networkProtection)
    }
    
    var availableFeatures: [SubscriptionFeature] {
        subscriptionManager.entitlements.compactMap { entitlement in
            switch entitlement {
            case .networkProtection:
                return .vpn
            case .dataBrokerProtection:
                return .personalInformationRemoval
            case .identityTheftRestoration:
                return .identityTheftRestoration
            default:
                return nil
            }
        }
    }
}
```

### VPN Integration
```swift
// ✅ CORRECT - VPN entitlement integration
final class VPNManager: ObservableObject {
    private let subscriptionManager: SubscriptionManager
    
    func enableVPN() async {
        guard subscriptionManager.hasEntitlement(for: .networkProtection) else {
            await showSubscriptionPrompt()
            return
        }
        
        // Enable VPN functionality
        await startVPNConnection()
    }
}
```

### Personal Information Removal (PIR)
```swift
// ✅ CORRECT - PIR implementation with freemium support
final class PIRManager: ObservableObject {
    private let subscriptionManager: SubscriptionManager
    
    var isFreemiumEligible: Bool {
        // Check feature flag and eligibility
        FeatureFlags.shared.isEnabled(.freemiumPIR) &&
        !subscriptionManager.isUserSubscribed &&
        isUSUser
    }
    
    func performScan() async {
        if subscriptionManager.hasEntitlement(for: .dataBrokerProtection) {
            await performFullScan()
        } else if isFreemiumEligible {
            await performLimitedScan()
        } else {
            await showSubscriptionPrompt()
        }
    }
}
```

## Purchase Flow Implementation

### Free Trial Support
```swift
// ✅ CORRECT - Free trial implementation
final class SubscriptionPurchaseViewModel: ObservableObject {
    @Published var isTrialEligible = false
    @Published var trialPeriod: String = ""
    
    func checkTrialEligibility() async {
        guard FeatureFlags.shared.isEnabled(.privacyProFreeTrial) else {
            isTrialEligible = false
            return
        }
        
        // Check server-side eligibility
        let eligible = await subscriptionManager.checkFreshFreeTrialEligibility()
        await MainActor.run {
            isTrialEligible = eligible
            if let product = subscriptionManager.currentProduct,
               let offer = product.introductoryOffer {
                trialPeriod = offer.localizedPeriod
            }
        }
    }
}
```

### Platform-Specific Purchase
```swift
// ✅ CORRECT - Platform-aware purchase flow
final class PurchaseFlowCoordinator {
    private let subscriptionManager: SubscriptionManager
    

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duckduckgo/apple-browsers](https://github.com/duckduckgo/apple-browsers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
