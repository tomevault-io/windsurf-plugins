---
trigger: always_on
description: BrickKit is a SwiftUI APP Acceleration Development Kit that provides:
---

# BrickKit AI Skills

## Project Overview

BrickKit is a SwiftUI APP Acceleration Development Kit that provides:
- View extensions via `Brick<Wrapped>` pattern (`view.ss.xxx`)
- Pre-built UI components for rapid development
- Core utility tools

## Directory Structure

```
Sources/Brick/
├── SwiftUI/           # SwiftUI extensions
│   ├── GridIem++     # Grid item extensions (GridItem.Size.adaptive/flexible)
│   ├── String+       # String extensions (localized, toInt, etc.)
│   ├── View+Geometry # View geometry extensions (onGeometryChange)
│   ├── Angle++       # Angle extensions (radians/degrees conversion)
│   ├── Collection+   # Collection extensions (push, pop, popTo helpers)
│   ├── ForEach++     # ForEach extensions
│   ├── Image++       # Image extensions (resize, tint, etc.)
│   ├── Label++       # Label extensions
│   ├── List++        # List extensions
│   ├── Menu++        # Menu extensions
│   ├── NavigationLink++ # NavigationLink extensions
│   ├── Section++     # Section extensions
│   ├── Shape++       # Shape extensions
│   ├── Spacer+       # Spacer extensions
│   ├── Task+         # Task extensions
│   ├── Text++        # Text extensions
│   ├── URL+          # URL extensions (scheme detection)
│   ├── View++        # General View extensions (eraseToAnyView, etc.)
│   ├── View+Background # View background extensions
│   ├── View+Conditionals # Conditional view modifiers
│   ├── View+Frame    # View frame extensions
│   ├── View+Geometry # View geometry extensions (onGeometryChange)
│   ├── View+Haptic   # View haptic feedback extensions
│   ├── View+Mask     # View mask extensions
│   └── View+Shadow   # View shadow extensions
├── Wrapped/          # View extensions via Brick<Wrapped> pattern (`view.ss.xxx`)
│   ├── ViewWrapped.swift    # Core Brick<Wrapped> + shadow/border
│   ├── Background.swift     # Background + hideListBackground/hideTextViewBackground
│   ├── Overlay.swift        # Overlay with alignment
│   ├── Geometry.swift       # onGeometryChange, visualEffect
│   ├── Alert.swift          # Alert presentation
│   ├── Badge.swift          # Badge overlay
│   ├── Border.swift         # Border modifier
│   ├── Corner.swift         # Corner radius modifier
│   ├── Mask.swift           # Mask modifier
│   ├── Hidden.swift         # Hidden view modifier
│   ├── SafeArea.swift       # Safe area insets
│   ├── BottomSafeArea.swift # Bottom safe area handling
│   ├── Alignment.swift      # Alignment helpers
│   ├── Section.swift        # Section helpers
│   ├── Submit.swift         # Submit handling
│   ├── Task.swift           # Task handling
│   ├── OnChange.swift       # onChange wrapper
│   ├── TabbarColor.swift    # TabBar color customization
│   ├── TabbarVisible.swift  # TabBar visibility control
│   ├── NavigationTitle.swift # Navigation title customization
│   ├── NavigationBarColor.swift # NavigationBar color
│   ├── CustomBackButton.swift # Custom back button
│   ├── ListSpace.swift      # List spacing
│   ├── OnTapLocal.swift     # Local tap gesture
│   ├── PushTransition.swift # Push transition animations
│   ├── ShareSheet.swift     # Share sheet presentation
│   ├── Checkmark.swift      # Checkmark view
│   ├── RequestReview.swift  # App Store review request
│   ├── AppStore.swift       # App Store helpers
│   └── ProposedViewSize.swift # Proposed view size
├── Utilities/         # Core utilities
│   ├── Brick.swift            # Core wrapper type (Brick<Wrapped>)
│   ├── BrickLog.swift         # Logging utilities
│   ├── Adapter.swift          # Responsive design adapter (screen scaling)
│   ├── Application.swift      # App info (version, build, name, bundle ID)
│   ├── CurrentLanguage.swift  # Language detection utilities
│   ├── Keyboard.swift         # Keyboard manager (show/hide, height tracking)
│   ├── Color+.swift           # SwiftUI Color extensions
│   ├── UIColor++.swift        # UIKit UIColor extensions
│   ├── UIView++.swift         # UIKit UIView extensions
│   ├── Screen++.swift         # Screen utilities (size, scale, safe area)
│   ├── CGSize++.swift         # CGSize extensions
│   ├── Image+.swift           # Image extensions
│   ├── SFSymbols/             # Type-safe SF Symbols
│   │   ├── SFSymbolName.swift # Protocol for SF Symbol names
│   │   └── SFSymbolsV1-V7.swift # SF Symbol versions (iOS 13-18)
│   └── ViewLifeCycle/         # View lifecycle hooks
│       ├── OnFirstAppear.swift    # onFirstAppear view modifier
│       ├── WillDisappear.swift    # onWillDisappear view modifier
│       └── DidDisappear.swift     # onDidDisappear view modifier
└── Tools/             # UI Components
    ├── Animation/     # Animation components (Animation+, WithAnimation+)
    ├── Blur/          # Blur effects (GlassBlurView, BlurView)
    ├── CarouselView/  # Carousel components (CarouselView, CarouselViewModel)
    ├── Date/          # Date utilities (Date+, DateFormatter, JSONEncoder/Decoder+Date)
    ├── Gestures/      # Gesture handlers (GestureButton, ScrollViewGestureButton, SwipeGesture)
    ├── Keychain/      # Keychain utilities (KeychainService, KeychainWrapper, KeychainItemAccessibility)
    ├── Language/      # Language settings (Language, LanguageSettings, LanguageView, SelectLanguageView)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zjinhu/Brick_SwiftUI](https://github.com/zjinhu/Brick_SwiftUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
