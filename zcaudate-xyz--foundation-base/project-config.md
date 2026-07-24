---
trigger: always_on
description: - **Fact Structure**: Tests are defined using `fact`. Metadata controls execution.
---

# Learnings

## `code.test` Framework

- **Fact Structure**: Tests are defined using `fact`. Metadata controls execution.
- **Component Injection**: Components like `|rt|` are NOT injected via `:use` or `:let` in the way standard `clojure.test` fixtures might work.
- **Global Components**: `fact:global` defines components that can be accessed in tests.
- **Broken `:use`/`:let`**: The `:use` and `:let` keys in `fact` metadata are stripped out by `fact-thunk` in `src/code/test/compile.clj` and are not functional for binding in the test body.
- **Setup/Teardown**: The correct way to handle per-test setup/teardown is using `:setup` and `:teardown` metadata keys, which can call helper functions.
- **Global State**: `fact:global` can define `:setup` and `:teardown` for the entire namespace, but per-fact isolation requires explicit `:setup`/`:teardown` on the fact itself if not using the global ones.

## `std.scheduler.spawn`

- **Runtime Management**: Spawns are managed within a runtime atom.
- **Testing Spawns**: Tests often require a runtime `|rt|` to be initialized.
- **Clear Operation**: The `clear` function needed to be updated to also remove spawns from the `:running` map, not just `:programs` and `:past`.

## `hara.lang` Tests

- **Macro Expansion**: `defstruct.rs` and other macros in `hara.lang.model.spec-rust` work correctly in isolation, suggesting previous "Syntax error macroexpanding" issues might have been context-dependent or transient.
- **R Integration**: `hara.lang.model.spec-r-test` uses a guard `hara.lang.spec.r/CANARY`. If this guard fails (returns false), the tests in that fact are skipped or fail with "Guard failed".

## Foundation MCP Server

- **Server Location**: The foundation MCP server is defined in `code.ai.server` within `foundation-base`.
- **Transport**: By default, `code.ai.server/create-server` configures the server to use SSE transport on port 3001.
- **Tools**:
    - `echo`, `ping`: Basic connectivity tests.
    - `lang-emit-as`: Transpiles Clojure DSL code to target languages (e.g., JS, Lua).
    - `std-lang-list`, `std-lang-modules`: Introspection for `hara.lang` languages and modules.
    - `clj-eval`: Evaluates arbitrary Clojure code.
    - `code-test`: Runs tests using `code.test.task/run`.
    - `code-doc-init`, `code-doc-deploy`, `code-doc-publish`: Manages documentation sites.
    - `code-manage`: Executes management tasks defined in `code.manage`.
- **Stdio Support**: While `mcp-clj` supports stdio transport (as seen in `mcp-clj.stdio-server.main`), `code.ai.server` currently hardcodes SSE. To use it via stdio, a custom entry point or modification to `create-server` would be required.

# Exported Memories

- **Grammar Retrieval**: To programmatically retrieve the grammar for a `hara.lang` language, use `(:grammar (l/get-book (l/default-library) :<lang-keyword>))`. The grammar is not exposed as a public var.
- **Test Timeouts**: When executing facts with timeouts in `code.test`, `java.util.concurrent.ExecutionException` is caught and unwrapped to re-throw the underlying cause (unless it is a `TimeoutException`), ensuring consistent error reporting between synchronous and asynchronous execution.
- **Context Documentation**: Documentation for `std.lib.context` is located in `plans/std_lib_context/`, including architectural analysis (`analysis.md`), general usage proposals (`proposals.md`), 3D virtual world mappings (`3d_virtual_worlds.md`), the applicative model (`applicative_model.md`), the invoke-as pattern (`invoke_as_patterns.md`), AI agent architecture (`ai_agents.md`), and async evaluation analysis (`async_evaluation.md`).
- **JavaScript Wrappers**: The `src/js` directory contains Clojure files defining standard JavaScript library wrappers (e.g., `js.react`, `js.tamagui`, `js.blessed`) using the `l/script` DSL, rather than raw JavaScript files.
- **DebounceDispatch**: `DebounceDispatch` (`std.dispatch.debounce`) offers three strategies for filtering high-frequency events: `:eager` (executes first, then blocks), `:delay` (waits for a pause, executes last), and `:notify` (executes first, then enforces a cooldown).
- **Console Log Testing**: Tests for `std.log.console` view functions (e.g., `console-status-outcome`) should handle ANSI escape codes in output, often necessitating regex matching or simple type checks (`string?`) instead of exact string equality.
- **Spawn Requirements**: The `std.scheduler.spawn/create-spawn` function requires the program map argument to include a `:create-fn` key.
- **Unchecked Task**: The `code.manage/unchecked` task identifies untested variables by recursively searching (using `flatten`) the test definition's s-expression for the `=>` assertion symbol, ensuring nested checks are detected. It also explicitly excludes facts marked with `^:unchecked` metadata from the report.
- **Dispatch Creation**: The `std.dispatch/create` function requires an `:options` map containing a valid `:pool` configuration (e.g., `{:size 1}`) to pass schema validation.
- **Layout Spacing**: When the indentation level is zero, the `row-spacing` generated by `std.block.layout.common/layout-by` consists of a single newline character without any trailing spaces.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zcaudate-xyz/foundation-base](https://github.com/zcaudate-xyz/foundation-base) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
