---
trigger: always_on
description: PuppeteerSharp is a C# library for automating Chromium-based browsers and Firefox. It provides a high-level API to control browsers via the DevTools Protocol (CDP) and WebDriver BiDi protocol.
---

# PuppeteerSharp Codebase Overview

PuppeteerSharp is a C# library for automating Chromium-based browsers and Firefox. It provides a high-level API to control browsers via the DevTools Protocol (CDP) and WebDriver BiDi protocol.

## Goal

The goal of this project is to be a port of the popular Node.js Puppeteer library to .NET.
Everything in PuppeteerSharp is inspired by the original Puppeteer library, but adapted to C# idioms and .NET practices.

## C# Code Style

When editing C# files, always place static members before non-static members (SA1204/SA1202 style rules). Never place a static method after instance methods.

## Git Workflow

Always verify which git branch you are on before making commits or creating PRs. Never commit to main/master. If the task requires a new branch, create it immediately and confirm with `git branch --show-current` before proceeding.

## Upstream Porting Conventions

When porting upstream TypeScript/Puppeteer changes to PuppeteerSharp (.NET), translate API naming conventions: camelCase → PascalCase, Promise<T> → Task<T>, interfaces prefixed with I, events use C# event patterns. Use `ReloadOptions`-style single parameter objects rather than creating redundant overloads.

## External Sources:
These external sources are referenced or used as inspiration in the codebase. Feel free to explore them for deeper understanding.
You are allowed to run git commands to update these repositories locally.

- Original Puppeteer repository: ../../puppeteer/puppeteer. Every time "upstream" is mentioned we are referring to this code.
- Bidi Driver: ../../webdriverbidi-net/webdriverbidi-net

## chromium-bidi Mapper Version

Chrome BiDi support uses the chromium-bidi mapper (downloaded at build time from npm). The version is tracked in `lib/PuppeteerSharp/Bidi/chromium-bidi.version`. When syncing with upstream Puppeteer, check if the chromium-bidi version has changed in `packages/puppeteer-core/package.json` (`"chromium-bidi": "X.Y.Z"`) and update our version file accordingly.

## Upstream code structure

- Code in upstream puppeteer-core/src/api/* are our abstract class. For instance our public abstract class Frame.
- Code in upstream puppeteer-core/src/bidi/* are our Bidi* classes.
- Code in upstream puppeteer-core/src/cdp/* are our Cdp* classes.

## Project Structure

```
lib/
├── PuppeteerSharp/                 # Main library
│   ├── Bidi/                       # WebDriver BiDi protocol implementation
│   ├── Cdp/                        # Chrome DevTools Protocol implementation
│   ├── Transport/                  # Communication layer (WebSocket, etc.)
│   ├── Helpers/                    # Utility functions and extensions
│   ├── Input/                      # Keyboard, Mouse, Touchscreen input handling
│   ├── Media/                      # Screenshot, PDF, viewport options
│   ├── Mobile/                     # Device descriptors and emulation
│   ├── PageAccessibility/          # Accessibility tree and ARIA handling
│   ├── PageCoverage/               # Code coverage tracking
│   ├── QueryHandlers/              # CSS, XPath, Pierce, Text selectors
│   ├── BrowserData/                # Browser version and channel management
│   ├── States/                     # Process state machine
│   ├── Injected/                   # JavaScript to be injected into pages
│   └── *.cs                        # Core classes (Browser, Page, Frame, etc.)
├── PuppeteerSharp.Tests/           # Test categories
├── PuppeteerSharp.Nunit/           # NUnit test framework integration
├── PuppeteerSharp.TestServer/      # Local HTTP server for testing
├── PuppeteerSharp.TestServer/wwwroot/ # Test fixtures and assets
├── demo/                           # Demo application
└── PuppeteerSharp.sln              # Solution file
```

### Protocol Abstraction Pattern

The library uses a base class pattern with protocol-specific implementations:

- **Interface Level**: `IBrowser`, `IPage`, `IFrame`, `IRequest`, `IResponse`
- **Abstract Base Classes**: `Browser`, `Page`, `Frame`, `Request<TResponse>`, `Response<TRequest>`
- **Protocol Implementations**: `CdpBrowser`/`BidiBrowser`, `CdpPage`/`BidiPage`, etc.

## Testing Infrastructure

### Test Infrastructure

#### PuppeteerSharp.Nunit (`PuppeteerSharp.Nunit/`)
- `PuppeteerTestAttribute`: Decorates tests with upstream metadata
- Test expectations system (skip/fail/timeout conditions)
- Platform-specific expectations (Win32, Linux, Darwin)
- Browser-specific expectations (Chrome, Firefox)
- Protocol-specific expectations (CDP, WebDriver BiDi)
- Headless mode variations (headless, headful, headless-shell)
- Local and upstream expectation merging
- Tests should always match the code in upstream. Tests should never be changed to match the code local code.

**IMPORTANT: Test Expectations Files Rules:**
- `TestExpectations.upstream.json`: This file should NEVER be edited unless syncing with the upstream Puppeteer project. It contains expectations that match the upstream Puppeteer test expectations.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hardkoded/puppeteer-sharp](https://github.com/hardkoded/puppeteer-sharp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
