---
trigger: always_on
description: - **Swift 6.0 compatible**: `swift-tools-version: 6.0`. Never use features that require a newer compiler.
---

## RULES

### Compatibility (non-negotiable)
- **Swift 6.0 compatible**: `swift-tools-version: 6.0`. Never use features that require a newer compiler.
- **Cross-platform**: must build and run without crashes/segfaults on both macOS and Linux. CI tests both (`macos-15` + the pinned Linux image from `scripts/toolchain.env`).
- When in doubt, verify with the CI pipeline before merging.

### Pre-Push Verification (non-negotiable)
- **Before pushing to GitHub**: ALWAYS run `./scripts/test-linux.sh` to verify build + tests pass on both macOS and Linux.
- The script runs the complete warning-fatal quality gate natively on macOS, then repeats it in the immutable Swift 6.0.3 Docker image from `scripts/toolchain.env`.
- **Never push code that has not been verified on both platforms.**
- Usage: `./scripts/test-linux.sh` (both), `./scripts/test-linux.sh macos`, `./scripts/test-linux.sh linux`, or `./scripts/test-linux.sh shell`
- Requires Docker Desktop to be running.

### Architecture (non-negotiable)

#### General Principles
- No Singletons
- The package graph contains no C/C++ targets or native decoder dependencies
- Image decoding is limited to static PNG and JPEG, implemented by vendored, namespaced pure Swift sources with documented provenance
- Validate image resource limits before invoking a format decoder
- **Before implementing ANYTHING NEW: Search the codebase** for similar patterns, reusable code, existing solutions
- Consolidate and reuse before adding new functions or types
- "Reinventing the wheel" is a code smell: investigate why it exists first

#### Code Reuse Checklist
1. Does a similar feature exist? Use it or extend it
2. Can I reuse a helper function/extension/modifier? Do it
3. Does a pattern already exist? Follow it exactly
4. Am I duplicating logic? Refactor into a shared utility
5. **Never implement features in isolation**: maximize consistency and minimize maintenance burden

### Workflow
- **NEVER merge PRs autonomously**: stop after creating, let user merge

### SwiftUI API Parity (non-negotiable)
Public APIs MUST match SwiftUI signatures exactly unless terminal constraints require deviation (document why in comments).

| Aspect | Requirement |
|--------|-------------|
| Parameter names | Exact (`isPresented`, not `isVisible`) |
| Parameter order | Exact (title, binding, actions, message) |
| Parameter types | Match closely (ViewBuilder closures, not pre-built values) |
| Trailing closures | `@ViewBuilder () -> T`, not `String` |

**Before implementing:** Look up exact SwiftUI signature first.
**TUI-specific APIs:** OK to add, but keep separate from SwiftUI equivalents.

### View Architecture (non-negotiable)

#### Public API: Every control is a View with a real body

**The Rule:**
- Every **public** control MUST be a `View` with a real `body: some View`
- The `body` MUST return actual Views (not `Never`, not `fatalError()`)
- All modifiers MUST propagate through the entire View hierarchy
- Environment values MUST flow down automatically

**Why this matters:**
```swift
// This MUST work exactly like SwiftUI:
List("Items", selection: $selection) {
    ForEach(items) { item in
        Text(item.name)
    }
}
.foregroundColor(.red)  // MUST affect all Text inside!
.disabled(true)         // MUST disable the entire List!
```

#### Renderable: When and where it is allowed

Terminal UI requires procedural buffer assembly (ANSI codes, Unicode borders,
buffer overlays). `Renderable` is the mechanism for this. It is allowed in
these cases:

| Layer | Example | Renderable? |
|-------|---------|-------------|
| **Leaf nodes** | `Text`, `Spacer`, `Divider` | Yes (terminal primitives) |
| **Private `_*Core` views** | `_ButtonCore`, `_VStackCore` | Yes (procedural ANSI rendering) |
| **Layout primitives** | `_VStackCore`, `_HStackCore` | Yes + `Layoutable` (two-pass layout) |
| **Modifier infrastructure** | `ModifiedView`, `EnvironmentModifier` | Yes (context/buffer pipeline) |
| **Public controls** | `Button`, `VStack`, `List` | **No** (must use `body: some View`) |

**The `_*Core` pattern:**
```swift
// Public View: real body, environment flows through
public struct MyControl<Content: View>: View {
    let content: Content

    public var body: some View {
        _MyControlCore(content: content)
    }
}

// Private Core: Renderable for terminal-specific rendering
private struct _MyControlCore<Content: View>: View, Renderable {
    let content: Content
    var body: Never { fatalError("_MyControlCore renders via Renderable") }

    func renderToBuffer(context: RenderContext) -> FrameBuffer {
        // Read environment from context, render with ANSI codes
    }
}
```

**Preferred: Pure composition (Box.swift is the reference):**
```swift
public struct MyControl<Content: View>: View {
    let content: Content

    public var body: some View {
        content
            .padding()
            .border()
    }
}
```

When possible, prefer composition over `_*Core`. Use `_*Core` + `Renderable`
only when the rendering requires procedural buffer manipulation that cannot
be expressed as View composition.

**WRONG Pattern (public control with Renderable):**
```swift
public struct MyControl: View {
    public var body: Never { fatalError() }  // WRONG!
}


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [phranck/TUIkit](https://github.com/phranck/TUIkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
