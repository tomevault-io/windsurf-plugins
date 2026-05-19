---
trigger: always_on
description: Pattern for abstracting pixel and wide event instrumentation behind domain-specific protocols.
---

# Instrumentation Facades

Feature code often becomes verbose when sending many pixels, or mixing pixel calls and wide event lifecycle management.

Consider a subscription purchase flow that needs to:
- Fire a daily pixel when purchase starts
- Start a wide event flow
- Update the wide event with timing data
- Fire unique pixels on success
- Complete the wide event with success/failure/cancelled status
- Handle multiple error cases with different failing steps

This leads to instrumentation code scattered throughout the feature, making it hard to:
- Understand the feature's core logic
- Test the feature in isolation
- Modify instrumentation without touching feature code
- Ensure all instrumentation points are covered

We can improve this using the facade pattern, abstracting our instrumentation behind a protocol. This is done by defining a protocol with domain-specific hooks that the feature calls, then implementing the protocol in a dedicated object that handles all instrumentation.

### Benefits

1. **Feature code emits domain events only** - Cleaner, more readable feature logic
2. **Instrumentation logic is centralized** - Easy to audit and modify
3. **Easy to inject mocks for unit testing** - Test feature behavior without pixel dependencies

## File Organization

Instrumentation facades should be placed in the **same module as the feature** they instrument:

| Component | Location |
|-----------|----------|
| Protocol | Feature module (e.g., `Subscription/SubscriptionPurchaseInstrumentation.swift`) |
| Default Implementation | Same module as protocol |
| Mock for Testing | Test target or same module |

For features that span iOS and macOS, place the protocol and implementation in a shared package (e.g., `BrowserServicesKit`).

## Pattern Structure

### Step 1: Define the Protocol

Create a protocol with methods for each instrumentation hook your feature needs. Name methods after domain events, not pixels:

```swift
public protocol SubscriptionPurchaseInstrumentation: AnyObject {
    func purchaseAttemptStarted(selectionID: String, freeTrialEligible: Bool, ...)
    func purchaseCancelled()
    func purchaseFailed(step: FailingStep, error: Error)
    func activationSucceeded()
    // ... other domain events
}
```

### Step 2: Implement the Default Class

Create an implementation that translates domain events to pixels and wide events. The implementation:
- Fires appropriate pixels (standard, daily, or unique)
- Manages wide event lifecycle (start, update, complete)
- Tracks internal state like the current wide event data

```swift
public final class DefaultSubscriptionPurchaseInstrumentation: SubscriptionPurchaseInstrumentation {
    private let wideEvent: WideEventManaging
    private var purchaseWideEventData: SubscriptionPurchaseWideEventData?

    public func purchaseAttemptStarted(...) {
        DailyPixel.fireDailyAndCount(pixel: .subscriptionPurchaseAttempt, ...)
        purchaseWideEventData = SubscriptionPurchaseWideEventData(...)
        wideEvent.startFlow(purchaseWideEventData!)
    }

    public func activationSucceeded() {
        UniquePixel.fire(pixel: .subscriptionActivated)
        wideEvent.completeFlow(purchaseWideEventData!, status: .success, ...)
    }
}
```

### Step 3: Use in Feature Code

Inject the instrumentation protocol and call it at appropriate points in your feature logic:

```swift
final class SubscriptionPurchaseFeature {
    private let instrumentation: SubscriptionPurchaseInstrumentation

    func subscriptionSelected(...) async {
        instrumentation.purchaseAttemptStarted(...)

        switch await performPurchase() {
        case .success:
            instrumentation.activationSucceeded()
        case .failure(let error):
            instrumentation.purchaseFailed(step: .accountPayment, error: error)
        }
    }
}
```

## Dependency Injection

### Constructor Injection (Preferred)

Pass the instrumentation as an init parameter:

```swift
final class SubscriptionPurchaseFeature {
    private let instrumentation: SubscriptionPurchaseInstrumentation

    init(instrumentation: SubscriptionPurchaseInstrumentation = DefaultSubscriptionPurchaseInstrumentation()) {
        self.instrumentation = instrumentation
    }
}
```

### Property Injection

For cases where the instrumentation is set after initialization (e.g., UserScripts):

```swift
final class DebugUserScript {
    weak var instrumentation: TabInstrumentationProtocol?
}

// In the parent object:
private let instrumentation = TabInstrumentation()

func configureUserScripts() {
    userScripts.debugScript.instrumentation = instrumentation
}
```

## Testing with Mocks

Create a mock that records method calls for verification:

```swift
final class MockSubscriptionPurchaseInstrumentation: SubscriptionPurchaseInstrumentation {
    private(set) var purchaseAttemptStartedCalls: [...] = []
    private(set) var activationSucceededCallCount = 0

    func purchaseAttemptStarted(...) {
        purchaseAttemptStartedCalls.append(...)
    }
}
```

Then verify behavior in tests:

```swift
func testPurchaseSuccess() async {
    let mock = MockSubscriptionPurchaseInstrumentation()
    let feature = SubscriptionPurchaseFeature(instrumentation: mock)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duckduckgo/apple-browsers](https://github.com/duckduckgo/apple-browsers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
