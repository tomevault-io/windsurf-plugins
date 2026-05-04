---
trigger: always_on
description: **Never create GitHub releases or draft releases.** The build pipeline creates immutable releases automatically. Creating releases manually (including drafts) interferes with this process.
---

# ClearanceKit — Developer Guide

## Releases

**Never create GitHub releases or draft releases.** The build pipeline creates immutable releases automatically. Creating releases manually (including drafts) interferes with this process.

Editing the **description/notes** of an existing release is fine — use `gh release edit <tag> --notes "..."`. Only the creation of releases is forbidden.

## Architecture

ClearanceKit uses hexagonal architecture. Three distinct layers must stay separate:

**Domain** (`Shared/`) — Pure Swift logic with no I/O, framework, or OS dependencies.
- `FAAPolicy.swift` — file-access policy evaluation
- `GlobalAllowlist.swift` — global allowlist matching
- `ProcessTree.swift` — process ancestry service

**Ports** — Swift protocols that define what the domain needs from the outside world.
- `ProcessTreeProtocol` — ancestry lookup
- `ServiceProtocol` / `ClientProtocol` (`XPCProtocol.swift`) — IPC surface

**Adapters** (`opfilter/`, `clearancekit/`) — Concrete implementations that translate between external systems and domain types.

`opfilter/` is organised into subdirectories by adapter role:
- `EndpointSecurity/` — translates Endpoint Security C events → domain types (`ESInboundAdapter`, `ESProcessRecord`, `MutePath`)
- `XPC/` — IPC boundary with the GUI app (`XPCServer`, `ConnectionValidator`, `EventBroadcaster`, `ProcessEnumerator`, `NSXPCConnection+AuditToken`)
- `Database/` — SQLite persistence (`Database`, `DatabaseMigrations`)
- `Policy/` — policy loading, signing, and state management (`PolicyRepository`, `PolicySigner`, `ManagedPolicyLoader`, `ManagedAllowlistLoader`)
- `Filter/` — filter orchestration and output (`FilterInteractor`, `AuditLogger`, `TTYNotifier`)

**Rule**: domain code never imports `EndpointSecurity`, `AppKit`, `SwiftUI`, `SQLite`, or any other infrastructure framework. Adapters own those imports.

## Build and Test

```bash
# Build and test via xcodebuild
xcodebuild test -scheme clearancekitTests -destination 'platform=macOS'
```

Tests live in `Tests/` and use the Swift Testing framework (`@Suite`, `@Test`, `#expect`).

## Testing Strategy

### New code — TDD
Write the test first. Red → Green → Refactor. Keep the cycle short: one failing assertion at a time.

### Existing untested code — Feathers approach
1. **Characterise before changing.** Write a test that pins the current behaviour exactly, even if that behaviour looks wrong. Do not change any logic yet; only add the test.
2. **Introduce a seam.** Extract an interface or closure parameter so the behaviour under test can be driven without real I/O. Prefer the simplest seam: a `@Sendable` closure injected at the call site beats a full protocol when only one behaviour varies.
3. **Narrow the scope.** Test the smallest unit that isolates the logic you need to change, using a fake collaborator (named `Fake…`, never `Mock…`).
4. **Change under the safety net.** Modify the logic only once a characterisation test is green. Keep commits atomic: characterisation test → logic change → refactoring, each as its own commit.
5. **Do not retrofit tests onto integration boundaries.** Adapter code that calls OS APIs belongs behind a protocol seam so the domain logic can be tested without it.

### What to test
- Domain logic (policy evaluation, path matching, allowlist resolution) must be fully covered.
- Adapters are tested at integration or system level when practical, not unit-level.
- Do not test Swift language mechanics or framework plumbing.

### Test structure
```swift
@Suite("Feature area")
struct FeatureAreaTests {
    @Test("specific observable behaviour")
    func specificObservableBehaviour() async {
        // Arrange
        // Act
        // #expect
    }
}
```

Fake collaborators are private nested structs/classes inside the test file. They implement only the protocol methods exercised by the test.

## Swift Conventions

### Naming
- Names describe intent at the call site, not the implementation.
- No abbreviations. No generic placeholders (`data`, `temp`, `value`, `result`, `info`).
- Prefer nouns for types, verb phrases for methods that perform work.

### Control flow
- Guard clauses first; exit early; reduce nesting.
- No `else` after a `return`, `throw`, or `continue` — extract a method instead.
- Ternaries on one line only; never nested.
- Complex boolean logic lives in a clearly named computed property or method.

### Methods
- Single responsibility. If you need to describe a method with "and", split it.
- Short parameter lists; group related parameters into a dedicated type.
- No side effects in computed properties.

### Collections
- Never return or pass `nil` for a collection; return an empty collection.
- Iterate directly; no `if !collection.isEmpty { for … }` guard before a loop.

### Immutability
- `let` for everything that does not change after initialisation.
- Prefer value types (`struct`, `enum`) for domain models.

### Enums over booleans
- Replace flag parameters and boolean return values with enums that name the cases explicitly.
- Pattern-match on enum cases; never inspect `.rawValue` to drive logic.

### Errors and assertions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [craigjbass/clearancekit](https://github.com/craigjbass/clearancekit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
