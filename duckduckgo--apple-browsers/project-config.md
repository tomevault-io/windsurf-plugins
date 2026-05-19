---
trigger: always_on
description: How to define, name, and fire pixels on iOS and macOS.
---

# Pixels

Pixels are one-off telemetry events sent via HTTP GET with a name and optional parameters. They are used for:

- Basic feature usage events (e.g., button clicks, screen impressions)
- Errors (e.g., network failures, parsing errors)
- Conversion and retention (e.g, subscription purchase and activation)

Pixels have the following requirements:

- Use clear & transparent naming, so it's obvious what the pixel and parameters are for. Pixel names should be self-documenting - avoid cryptic abbreviations or shorthand.
- Only include information that is essential for the pixel
- Do not use values that are overly precise, e.g. if using an integer value in a parameter, bucket it into ranges rather than including the value verbatim
- Never include PII, URLs, or other forms of user-identifiable information in pixel names or parameters

## Types of Pixels

### Standard Pixels

Sent every time the event occurs.

```swift
Pixel.fire(pixel: .subscriptionRestoreAfterPurchaseAttempt)

Pixel.fire(pixel: .autofillLoginsSavePromptDisplayed, withAdditionalParameters: [
    PixelParameters.autofillPromptTrigger: "manual"
])
```

### Daily Pixels

Sent once per day per event. Used to determine the number of users affected by a particular error 

```swift
DailyPixel.fireDailyAndCount(pixel: .subscriptionPurchaseAttempt, pixelNameSuffixes: DailyPixel.Constant.legacyDailyPixelSuffixes)
```

### Unique Pixels

Sent once per install for the lifetime of the install.

```swift
UniquePixel.fire(pixel: .subscriptionActivated)
```

## Pixel Definition Patterns

### iOS Pixels

iOS pixels are defined as cases on `Pixel.Event` in `iOS/Core/PixelEvent.swift`. Each enum case maps to an HTTP pixel name string via a computed `name` property.

#### Adding a New iOS Pixel

1. Add a new enum case to `Pixel.Event` in `iOS/Core/PixelEvent.swift`.
2. Add a corresponding case to the `name` computed property (also in `PixelEvent.swift`) that returns the pixel's string name.
3. Fire the pixel using `Pixel.fire`, `DailyPixel.fireDailyAndCount`, or `UniquePixel.fire`.
4. Add the matching pixel definition in `iOS/PixelDefinitions/pixels/definitions/*.json5` - see the `Pixel Validation` section below for more.

#### Enum Case Definition

```swift
extension Pixel {
    public enum Event {
        case appInstall
        case appLaunch
        case subscriptionPurchaseAttempt
        case subscriptionPurchaseSuccess
        case subscriptionActivated
        // ...
    }
}
```

#### Enum-to-String Mapping

The `Pixel.Event` enum has a computed `name` property that maps each case to its HTTP pixel name string:

```swift
extension Pixel.Event {
    public var name: String {
        switch self {
        case .appInstall: return "m_install"
        case .appLaunch: return "ml"
        case .subscriptionPurchaseAttempt: return "m_subscribe"
        // ...
        }
    }
}
```

#### Naming Convention

iOS pixel names follow these conventions:

- **Prefix**: Always start with `m_` (for "mobile").
- **Separators**: Use underscores (`_`) or hyphens (`-`) between words.
- **Format**: Use `m_feature_action` or `m_feature-sub-feature_action`.
- **Clarity**: Names should be clear and self-documenting. Anyone reading the pixel name should understand what it represents without needing additional context.

**Avoid legacy shorthand naming.** The codebase contains legacy pixels with cryptic names like `ml`, `mp`, `mf`, `m_r`. These are difficult to understand and should not be used as a template for new pixels. New pixels should use descriptive names.

Examples:

| Style | Enum Case | String Name | Notes |
|-------|-----------|-------------|-------|
| Good | `.pullToRefresh` | `"m_pull-to-reload"` | Clear and descriptive |
| Good | `.autofillLoginsSavePromptDisplayed` | `"m_autofill_logins_save_prompt_displayed"` | Self-documenting |
| Legacy | `.appLaunch` | `"ml"` | Avoid this style for new pixels |
| Legacy | `.privacyDashboardOpened` | `"mp"` | Avoid this style for new pixels |

#### Parameterized Pixel Cases

Enum cases can have associated values that are interpolated into the pixel name:

```swift
// Enum definition with associated value
case networkProtectionLatency(quality: String)
case syncLocalTimestampResolutionTriggered(Feature)

// In the name property
case .networkProtectionLatency(let quality):
    return "m_netp_ev_\(quality)_latency"
case .syncLocalTimestampResolutionTriggered(let feature):
    return "m_sync_\(feature.name)_local_timestamp_resolution_triggered"
```

### macOS Pixels (PixelKit)

macOS uses `PixelKitEvent` protocol, typically in feature-specific files. The implementation of the protocol looks like this:

```swift
// SharedPackages/BrowserServicesKit/Sources/PixelKit/PixelKitEvent.swift
public protocol PixelKitEvent {
    var name: String { get }
    var standardParameters: [PixelKitStandardParameter]? { get }
    var parameters: [String: String]? { get }
    var error: NSError? { get }
}
```

An example implementation of this protocol is:

```swift
// macOS/DuckDuckGo/Statistics/SubscriptionPixel.swift
enum SubscriptionPixel: PixelKitEvent {
    case subscriptionActive(AuthVersion)
    case subscriptionPurchaseAttempt
    case subscriptionPurchaseSuccess
    // ...

    var name: String {
        switch self {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duckduckgo/apple-browsers](https://github.com/duckduckgo/apple-browsers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
