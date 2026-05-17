---
trigger: always_on
description: **Revision B — 2025-12-22**
---

# Fabric Engineering Specification

**Revision B — 2025-12-22**

> This document supersedes transient chat discussions and supplements
> the public documentation (`README.md`, `ARCHITECTURE.md`, `NODES.md`, etc.) - which you should consult.
> It defines the architectural contracts, design patterns, and coding guidelines
> that all Fabric contributors (human or AI) must follow.
> 
> the public code base exists at https://github.com/Fabric-Project/Fabric and should be consulted
> 
> **Purpose:** This spec exists to guide consistent, high-performance, ergonomic development
> of the Fabric node-based runtime. It is intended for internal engineering and AI-assisted development,
> not public distribution.

## Immediate Next Goals

Major Effort:

We need deterministic, non-recursive evaluation of graphs that contain feedback loops (cycles). The current pull-eval recursion breaks on cycles unless we short-circuit, but short-circuiting without defined semantics breaks outputs. The correct semantics for most Fabric feedback graphs are “temporal feedback” (cycle reads previous-frame values), which requires a stable previous-frame snapshot per output port.

Separate concerns:
- PortType - schema/editor-time contract (connectability, UI, serialization, declared type).
- PortValue - runtime value container (optional in Step 1, useful for cache/inspection later).
- NodePort<T> - remains strongly typed for node authors.
- Feedback - should be an execution/renderer caching details and not baked into ports/nodes for 3rd party devs to reason about.
- Update Classes that vend `FabricImages` to always output a new image per frame using our `GraphRenderers` helper method `newImage(withWidth width:Int, height:Int) -> FabricImage?`

## Engineering Guidelines

### General
- Do not introduce third-party frameworks without asking first.
- Avoid UIKit / AppKit unless requested.
- We use Swift 5.9 for now
- We use SwiftUI
- We target macOS 15 + , iOS 18 +, visionOS 2.0 +
- We priortize clean code, with variable and function names optimized for legibility and self documentation - we can be verbose to avoid ambiguity
- We avoid single, acronym style variable or function names
- We do not violate D.R.Y.
- We keep separation of responsibilities.

### Swift
- Always mark @Observable classes with @MainActor.
- Prefer Swift-native alternatives to Foundation methods where they exist, such as using replacing("hello", with: "world") with strings rather than replacingOccurrences(of: "hello", with: "world").
- Prefer modern Foundation API, for example URL.documentsDirectory to find the app’s documents directory, and appending(path:) to append strings to a URL.
- Never use C-style number formatting such as Text(String(format: "%.2f", abs(myNumber))); always use Text(abs(change), format: .number.precision(.fractionLength(2))) instead.
- Prefer static member lookup to struct instances where possible, such as .circle rather than Circle(), and .borderedProminent rather than BorderedProminentButtonStyle().
- Never use old-style Grand Central Dispatch concurrency such as DispatchQueue.main.async(). If behavior like this is needed, always use modern Swift concurrency.
- Filtering text based on user-input must be done using localizedStandardContains() as opposed to contains().
- Avoid force unwraps and force try unless it is unrecoverable.

### SwiftUI instructions

- Always use foregroundStyle() instead of foregroundColor().
- Always use clipShape(.rect(cornerRadius:)) instead of cornerRadius().
- Always use the Tab API instead of tabItem().
- Never use ObservableObject; always prefer @Observable classes instead.
- Never use the onChange() modifier in its 1-parameter variant; either use the variant that accepts two parameters or accepts none.
- Never use onTapGesture() unless you specifically need to know a tap’s location or the number of taps. All other usages should use Button.
- Never use Task.sleep(nanoseconds:); always use Task.sleep(for:) instead.
- Never use UIScreen.main.bounds to read the size of the available space.
- Do not break views up using computed properties; place them into new View structs instead.
- Do not force specific font sizes; prefer using Dynamic Type instead.
- Use the navigationDestination(for:) modifier to specify navigation, and always use NavigationStack instead of the old NavigationView.
- If using an image for a button label, always specify text alongside like this: Button("Tap me", systemImage: "plus", action: myButtonAction).
- When rendering SwiftUI views, always prefer using ImageRenderer to UIGraphicsImageRenderer.
- Don’t apply the fontWeight() modifier unless there is good reason. If you want to make some text bold, always use bold() instead of fontWeight(.bold).
- Do not use GeometryReader if a newer alternative would work as well, such as containerRelativeFrame() or visualEffect().
- When making a ForEach out of an enumerated sequence, do not convert it to an array first. So, prefer ForEach(x.enumerated(), id: \.element.id) instead of ForEach(Array(x.enumerated()), id: \.element.id).
- When hiding scroll view indicators, use the .scrollIndicators(.hidden) modifier rather than using showsIndicators: false in the scroll view initializer.
- Place view logic into view models or similar, so it can be tested.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Fabric-Project/Fabric](https://github.com/Fabric-Project/Fabric) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
