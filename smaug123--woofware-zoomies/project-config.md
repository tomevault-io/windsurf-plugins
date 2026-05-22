---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Project

This is WoofWare.Zoomies, a reactive immediate-mode TUI framework which will eventually follow the model of Jane Street's Bonsai. It is written in F#, but is currently incomplete.

The intended philosophy of the project is that from a small set of powerful and coherent primitives, it should be possible to build a number of higher-level ergonomic libraries which provide an easy-to-use interface.
When using the higher-level libraries, the coherence of the underlying primitives should permit the user to drop down seamlessly, as low-level as necessary to achieve any particular customisation.
Ideally, the foundation of the framework is very small, with perfect compositional properties, such that an ergonomic system is merely a natural corollary of the primitives.

The framework should avoid doing work unless that work is necessary.
Ultimately, we will use something like Bonsai to deduplicate work.

The framework renders the world afresh each cycle.
It doesn't inherently consider the virtual DOM on one render loop to be related to the DOM of the previous loop, for example.
It stores state internally for efficiency, but as far as the end-user programmer is concerned, they simply provide a virtual DOM on request every tick, and we render it.

The framework should never throw exceptions when user input is bad.
Instead, invalid inputs should be handled gracefully by using sensible defaults or rendering fallback content.
This ensures the application remains stable and provides a better user experience.
(Throwing on *framework* bugs is fine: I prefer loud failures to silent ones.)

# Commands

## Build
- `dotnet build` - Standard .NET build (`dotnet` is on the path thanks to `direnv`)
- `nix build` - Build using Nix flake

## Test
- `dotnet test` - Run all tests
- `dotnet test --filter "TestMethod=SpecificTest"` - Run specific test
- `dotnet test WoofWare.Zoomies.Test` - Run only the test project

## Code Quality
- `dotnet fantomas .` - Format F# code using Fantomas
- `./analyzers/run.sh` - Run F# analyzers. These generally don't fail with an exit code; they'll print their output, though. Treat warnings as errors (this is not actually easy to do with the current analyzer tool, otherwise I'd have made the analyzer script fail on warnings).

Always format with Fantomas before committing, and run the analyzers with `./analyzers/run.sh`.

## Debugging

### Debug Layout Logging

Set the environment variable `WOOFWARE_ZOOMIES_DEBUG_TO_FILE=true` (or `=1`) to enable debug layout logging. When enabled, the framework will write detailed layout information to a temporary file with a path like `/tmp/zoomies-layout-<guid>.txt`. The file path will be printed to stderr when the application starts.

This is useful for diagnosing layout issues by seeing how the framework is arranging components in the virtual DOM.

# Architecture

## Project Structure
- **WoofWare.Zoomies/** - Core TUI framework library
- **WoofWare.Zoomies.App/** - Demonstration/example application
- **WoofWare.Zoomies.Test/** - Unit tests and test infrastructure

## Documentation

Architecture documentation lives in `docs/architecture/`.

## Core Concepts

### Virtual DOM (Vdom)
The framework uses a virtual DOM approach with these key types:
- `Vdom<'bounds>` - Virtual DOM nodes supporting text content, checkboxes, bordered panels, and split layouts
- Direction-based panel splitting (Vertical/Horizontal) with proportional or absolute sizing
- Focus management system for interactive elements

### World State Management
- `WorldFreezer<'appEvent>` - Manages application state changes and event processing
- `WorldProcessor<'appEvent, 'userState>` - Processes world state changes and updates VDOM
- Event-driven architecture with keystroke handling and focus cycling. The render loop summons a readout of all the external changes at the start of each render; the framework consumes input only on demand

### Rendering System
- `RenderState` - Tracks rendering state including previous VDOM for diffing
- Console abstraction layer for terminal operations
- ANSI control sequence and mouse mode support

## Writing tests

* When writing tests that exercise the UI, use the system as a user would. For example, don't use spooky external mutable state to control vdom creation; just let the WoofWare.Zoomies framework give you an appropriate user state, and send keystrokes to manipulate the state.

## Dependency libraries

### WoofWare.Expect snapshot testing

The usual workflow for updating snapshots using the WoofWare.Expect snapshot testing library is:

* Enter bulk-update mode by setting a `[<OneTimeSetUp>]` function (from NUnit.Framework) to `GlobalBuilderConfig.enterBulkUpdateMode ()`
* Run the tests. They will fail in the process of updating snapshots (this is so that you can't accidentally commit a test in update mode).
* Undo bulk-update mode by commenting out the `enterBulkUpdateMode ()`.
* Rerun the tests, if you like, to observe that the snapshots are now working.

# F# Language Gotchas and Hints

## Recursive functions in modules require `let rec ... and` syntax


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Smaug123/WoofWare.Zoomies](https://github.com/Smaug123/WoofWare.Zoomies) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
