---
trigger: always_on
description: This file provides guidance to GitHub Copilot when working on this repository.
---

# OUDS iOS - GitHub Copilot Instructions

This file provides guidance to GitHub Copilot when working on this repository.
It covers contributor and maintainer guidelines: code formatting, architecture, build process, best practices, ecodesign, accessibility, development requirements, build commands and review guidelines.

## 1. Code formatting

The source code is formatted for Swift 6.3. Configuration of formatter is in `.swiftformat` and linter in `.swiftlint`.

## 2. Architecture details

The Swift Package contains several products or targets exposing API in separated dependencies.

### 2.1 Modules

Here are the modules of the Swift Package, i.e. set of features with external dependencies.

#### Core / Components

Here the the components provided by the Swift Package like buttons, switchs, checkboxes, chips, tags, links, etc.

#### Core / Themes / Orange

Here is the definition of the Orange theme for all Orange products.

#### Core / Themes / Orange Compact

Here is the definition of the Orange theme for all Orange products but with constraints of spaces and sizes.

#### Core / Themes / Sosh

Here is the definition of the Sosh theme for all Sosh products.

#### Core / Themes / Wireframe

Here is the definition of the Wireframe theme for prototyping and mockups.

#### Core / ThemesContract

Here is the definition of the theme contract, the tokens provider and abstract entities.

#### Core / Tokens / ComponentTokens

Here are the definitions of the tokens used for components styling and look and feel.

#### Core / Tokens / SemanticTokens

Here are the definitions of the tokens used by component tokens, pointing to raw tokens.

#### Core / Tokens / RawTokens

Here are the definitions of the tokens pointing to raw values like strings, floats and ints, used by semantic tokens.

#### Foundations

Here are some shared logics and objects with plenty of extensions and utilities used everywhere.

### 2.2 Architecture guidelines

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

## 3. Build verification process ⚠️ CRITICAL

**IMPORTANT**: When editing code, you MUST:
1. Format the sources
2. Build the project after making changes
3. Fix any compilation errors before proceeding
4. Run the tests
5. Run the linter and fix any warnings and errors

## 4. Best practices

### 4.1 DO

- Write documentation in Swift DocC format for public API
- Use Swift's type system for safety
- Use public modifier only when needed, prefer internal or private
- **IMPORTANT**: The project supports iOS 26 SDK while maintaining iOS 15 as the minimum deployment target. Use `#available` checks when adopting iOS 15+ APIs.
- **IMPORTANT**: The project runs for iOS / iPadOS, macOS, visionOS, tvOS and watchOS. Use `#if os` checks to compile only code available for specific API / OS
- If a third party dependency is added or updated, update the Software Bill of Material
- Apply Clean Code, DRY, SOLID and TDD principles

### 4.2 DON'T

- Add abstraction layers without clear benefit
- Use Combine for simple async operations
- Overcomplicate simple features
- Use UIKit except for some specific API related to accessibility if needed

## 5. Ecodesign basics 🟡 RECOMMENDED

### 5.1 Animations

- Use native / system animations if animations must be used

### 5.2 Bad patterns

- Prefer pull to refresh instead of infinite scroll
- Avoid autocompletion if it makes network requests

### 5.3 Cache

- For heavy objects or costly objects to compute (data from networks, date formatters, etc.), use cache like `NSCache`
- For HTTP requests, use also HTTP cache

### 5.4 CPU

- Distribute tasks across different threads to free the CPU up as soon as possible
- Don't use the CPU unnecessarily
- Use app lifecycle to stop background tasks

### 5.5 Downloads

- Avoid automatic download
- Prefer download on Wifi networks

### 5.6 Energy

- Never ignore low energy mode
- If this mode is enabled, disable animations, intensive tasks, display of images and videos, cellular connections, HD / 4K (and above) features, use low colors instead of high (overall on Android with AMOLED screens)
- Avoid forcing the brightness to maximum

### 5.7 Fonts

- Prefer system fonts if possible, but in OUDS context use still the view modifiers and provided typography
- Use WOFF2 otherwise

### 5.8 Network connections

- Prefer wired and Wi-Fi connections to cellular connections
- If using a cellular connection, group requests as much as possible to avoid the device constantly being connected to the cell tower
- Use data caching and Gzip compression
- Avoid periodic polling to prevent rapid battery drain
- Avoid maintaining connections; services like Apple Push Notifications and Firebase Cloud Messaging can help

### 5.9 Notifications


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Orange-OpenSource/ouds-ios](https://github.com/Orange-OpenSource/ouds-ios) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
