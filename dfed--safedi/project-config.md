---
trigger: always_on
description: The core documentation is `Documentation/Manual.md`. Read it before making changes to understand SafeDI's API, macros, configuration options, and mock generation. The manual is the source of truth for user-facing behavior — if you change behavior, update the manual.
---

# SafeDI — Claude Code Guidelines

## Documentation

The core documentation is `Documentation/Manual.md`. Read it before making changes to understand SafeDI's API, macros, configuration options, and mock generation. The manual is the source of truth for user-facing behavior — if you change behavior, update the manual.

## Build & Test

```bash
swift build --traits sourceBuild   # Build all targets
swift test --traits sourceBuild    # Run all tests
./CLI/lint.sh                      # SwiftFormat — must pass before every push
swift test --traits sourceBuild --enable-code-coverage  # Coverage report
```

The `sourceBuild` trait is required for local development to compile SafeDITool from source. Without it, the plugin uses the prebuilt binary from the artifact bundle.

Always lint before pushing. Always run the full test suite after changes — don't rely on filtered runs alone.

## Architecture

SafeDI is a compile-time dependency injection framework for Swift. It uses Swift macros (`@Instantiable`, `@Instantiated`, `@Received`, `@Forwarded`) to declare dependency graphs, then generates initializer code and mock methods via a build tool plugin.

### Key modules

| Module | Role |
|--------|------|
| `SafeDICore` | Models (`TypeDescription`, `Property`, `Instantiable`, `Dependency`), visitors (`FileVisitor`, `InstantiableVisitor`), generators (`ScopeGenerator`, `DependencyTreeGenerator`) |
| `SafeDIMacros` | Swift macro implementations (`@Instantiable`, `@Received`, etc.) |
| `SafeDITool` | CLI entry point with `scan` and `generate` subcommands — scans Swift files for `@Instantiable` types, builds dependency tree, generates output |
| Plugins (`SafeDIGenerator`) | SPM build tool plugin that wires the tool into the build |

### Code generation flow

1. **Plugin** writes CSV of swift files → runs `SafeDITool scan` to build manifest → runs `SafeDITool generate` to produce code
2. **`SafeDITool generate`** parses all files via `FileVisitor` → builds `DependencyTreeGenerator` → generates per-root code + mock code
3. **DependencyTreeGenerator** creates `ScopeGenerator` trees → each generates its code via `generatePropertyCode`
4. **Mock generation** (`generateMockCode`) creates `mock()` static methods with `@autoclosure @escaping` parameters, `T? = nil` subtree parameters, and `MockContext` for disambiguation

### Mock generation flow

Mock generation follows this pipeline:

1. **`generateMockCode`** builds the mock scope map via `createMockTypeDescriptionToScopeMapping` (includes ALL types, not just reachable from roots)
2. For each type with `generateMock: true`, **`createMockRootScopeGenerator`** promotes `@Received` dependencies to root-level children and validates for cycles
3. **`ScopeGenerator.generateMockRootCode`** builds the mock method:
   - Calls `collectMockParameterTree` to walk the dependency tree and build `MockParameterNode` trees
   - Collects flat parameters: `@Forwarded` deps, uncovered `@Instantiated` deps, `@Received` deps not in the tree
   - Generates `SafeDIOverrides` struct (if tree children exist) and the `mock()` method signature and body

The production code path (`generatePropertyCode` with `.dependencyTree`) and mock path (`.mock`) share the same `ScopeGenerator` but diverge at `generatePropertyCode`. Mock fields (`mockInitializer`, `mockReturnType`, `customMockName`) are only accessed in mock code paths — never in production paths.

### Cycle mechanisms

Multiple cycle-related mechanisms coexist — keep them distinct:

| Name | Scope | Set by / when | Role |
|------|-------|---------------|------|
| `isPropertyCycle` on a scope/node | Per-root, syntactic | `Scope.createScopeGenerator` when a property label repeats in the ancestor stack | Terminates infinite descent during a single root's walk; differs between roots for the same underlying cycle. |
| `cycleEdges` (set of `CycleEdge`) | Global | `DependencyTreeGenerator.computeCycleEdges`, alphabetical-DFS feedback arc set | Chooses a single back-edge per cycle consistently across all roots — required because shared `SafeDIMockConfiguration` structs must have one shape. Consumed by struct generation AND override-path navigation in mock bodies. |
| `throwIfInvalidCycle` | Shared classifier | Called from `validatePropertiesAreFulfillable` (production) and `validateMockRootScopeForCycles` (mock) | Rejects constant + partially-lazy + received-lazy cycles. Accepts fully-lazy non-received cycles (those compile as long as pruning breaks the config-struct recursion). |
| `validateMockRootScopeForCycles` | Mock graph | Called after `@Received` promotion in `createMockRootScopeGenerator` | Walks the post-promotion scope to catch cycles that only surface once received deps are promoted to mock-root children. |

Key consequence: if `throwIfInvalidCycle` accepts a cycle, every hop in it is non-constant (Instantiator/ErasedInstantiator-typed). Any code that assumes "a cycle node reached through a constant-typed property" is reasoning about something impossible.

### Mock generation specifics


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dfed/SafeDI](https://github.com/dfed/SafeDI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
