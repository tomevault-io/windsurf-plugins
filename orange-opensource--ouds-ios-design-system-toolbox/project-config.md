---
trigger: always_on
description: This file provides guidance to GitHub Copilot when working on this repository.
---

# OUDS iOS Design System Toolbox app - GitHub Copilot Instructions

This file provides guidance to GitHub Copilot when working on this repository.
It covers contributor and maintainer guidelines: code formatting, architecture, build process, best practices, ecodesign, accessibility, development requirements, build commands and review guidelines.

## 1. Project Overview

OUDS means Orange Unified Design System and is the new cohesive and unified design system for Orange Group.
It provides a Swift Package and a demo application (this repository) called Design System Tooblox which embeds the Swift Package to expose its public API.
The project is open source under MIT license and hosted on GitHub in Orange-OpenSource organization.
The products support iOS 15, iPadOS 15, macOS 15, visionOS 1, watchOS 11 and tvOS 16.
The products are written in Swift with SwiftUI as UI framework and Swift 6 (format, grammar and concurrency).

## 2. Vocabulary

- *tokenator*: an internal tool which uses Figma specifications exported as JSON to convert them and send through pull requests the Swift code for tokens
- *token*: variable containing a value in most of cases defined by *tokenator*
- *raw token*: a family of tokens which have for value a raw type value like String, Int, or CGFloat
- *semantic token*: a family of tokens which point to raw tokens and bring meanings in their name, used inside components
- *component tokens*: a family of tokens for some components if semantic tokens are not enough, and have for values semantic tokens
- *theme*: a set of tokens, assets like fonts and images, to use in app to style it and change their look and feels
- *tuning*: some small configuration elements for a theme like rounded corners
- *token provider*: an object in a theme gathering tokens (semantics and components)
- *component*: mainly a SwiftUI view with specific features and layouts like buttons, switch, link etc.

## 3. Code formating

The source code is formatted for Swift 6.3. Configuration of formater is in `.swiftformat` and linter in `.swiftlint`.

## 4. Project structure

### 4.1 DesignToolbox

Contains the sources of the Design System Toolbox app for iOS, iPadOS, macOS and visionOS.

### 4.2 DesignToolboxUITests

Contains the sources of UI tests to run on simulators or devices making tests on components and navigating between pages.

### 4.3 DesignToolboxSnapshotTests

Contains the sources of snapshots tests, i.e. unit tests where there are comparisons of the tokens and components looks and feels using screen rendering.

### 4.4 DesignToolboxUnitTests

Contains the sources of some unit tests.

### 4.5 DesignToolbox (Light)

Contains source code of the design toolbox app but only for watchOS and tvOS as a light version with few configurations possibilities and more catalogs of components displayed in one time.

## 5. Architecture details

The Design System Toolbox is quite simple and must be usable in iOS, iPadOS, macOS, visionOS and watchOS.

### 5.1 Pages

Here are the "views" of the app displaying the tokens and components, gathered by components and tokens, and with folder in higher level depending to navigation.

### 5.2 Utils

Here are some utilities, extensions and objects to sued everywhere in the app.

### 5.3 Resources

Here are assets, images, HTML files like legal notices and fonts.

## 6. Architecture guidelines

- SwiftUI is the default UI paradigm - embrace its declarative nature
- Avoid legacy UIKit patterns and unnecessary abstractions
- Focus on simplicity, clarity, and native data flow
- Let SwiftUI handle the complexity - don't fight the framework
- Organize by components, keeps things isolated
- Keep related code together in the same file when appropriate
- Use extensions to organize large files
- Follow Swift naming conventions consistently
- Public enum must be marked `@frozen`
- Class must be marked `final`
- Small functions when possible must be marked `@inlinable`

## 7. Build verification process

**IMPORTANT**: When editing code, you MUST:
1. Format the sources
2. Build the project after making changes
3. Fix any compilation errors before proceeding
4. Run the tests
5. Run the linter and fix any warnings and errors

## 8. Best practices

### 8.1 DO

- Write documentation in Swift DocC format for public API
- Use Swift's type system for safety
- Use public modifier only when needed, prefer internal or private
- **IMPORTANT**: The project supports iOS 26 SDK while maintaining iOS 15 as the minimum deployment target. Use `#available` checks when adopting iOS 15+ APIs.
- **IMPORTANT**: Use `#available` checks when adopting watchOS 11.6+ APIs.
- **IMPORTANT**: Use `#available` checks when adopting visionOS 1.3+ APIs.
- **IMPORTANT**: Use `#available` checks when adopting macOS 15.6+ APIs.
- **IMPORTANT**: Use `#available` checks when adopting tvOS 16.6+ APIs.
- **IMPORTANT**: The project runs for iOS / iPadOS, macOS, visionOS abd watchOS. Use `#if os` checks to compile only code avaialble for specific API
- If a third party dependency is added or updated, update the Software Bill of Material
- If a third party dependency is added or updated, update the 3rd parties list in the Design System Toolbox
- Apply Clean Code, DRY, SOLID and TDD principes

### 8.1 DON'T


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Orange-OpenSource/ouds-ios-design-system-toolbox](https://github.com/Orange-OpenSource/ouds-ios-design-system-toolbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
