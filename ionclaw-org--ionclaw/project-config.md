---
trigger: always_on
description: C++20 AI-agent orchestrator. A Poco HTTP server exposes a REST + WebSocket API and an MCP endpoint, drives an agent loop over pluggable LLM providers, and bridges the same agents onto messaging channels. The engine also builds as a shared library for the iOS/tvOS/watchOS xcframework and the Android aar.
---

# IonClaw

C++20 AI-agent orchestrator. A Poco HTTP server exposes a REST + WebSocket API and an MCP endpoint, drives an agent loop over pluggable LLM providers, and bridges the same agents onto messaging channels. The engine also builds as a shared library for the iOS/tvOS/watchOS xcframework and the Android aar.

Dependencies come through CPM: Poco (HTTP/TLS/XML/Zip), nlohmann/json, yaml-cpp, spdlog, OpenSSL, jwt-cpp, hnswlib, and llama.cpp (isolated). User language for chat is Brazilian Portuguese; code and comments are English.

## Build and test

- `make build` — release server (`build/bin/ionclaw-server`).
- `make test` — configure with `-DIONCLAW_BUILD_TESTS=ON` and run the doctest suite (`build/bin/ionclaw-tests`). The suite covers pure helpers, internal-state round-trips, and stress/concurrency cases (many-thread producers/consumers, lock-scope races, persistence under concurrent writers). Add a test file to `tests/`, list it in `tests/CMakeLists.txt`, and keep every fix covered by a regression case. doctest cannot decompose `&&`/`||` inside `CHECK` — assign to a bool first. Guard any potential-infinite-loop regression with a `std::async` + `wait_for` timeout so a regression fails instead of hanging CI.
- `make build-web` — Vue client into `main/resources/web` (gitignored, embedded via CMake). Rebuild it before a cmake build when the web changed, then re-run cmake so the embed picks up the new asset hashes.
- `make build-lib` / `make build-xcframework` / `make build-android` — shared-library targets.

CI builds all six platforms on every push: macOS arm64, macOS x86_64, Linux x86_64, Windows x86_64, iOS xcframework, Android aar. A change is only done when all six are green.

## Code standard (strict)

Code and comments are in English. The code must read as if an experienced product C++ engineer wrote it, never like generated output. Do a change only when it makes sense and is genuinely needed, never to show work.

### Philosophy

- No fallbacks, no backward-compatibility shims, no legacy code, no gambiarras, no dirty code, no dead code.
- Never keep a branch that exists only because the code used to work differently. Write the new, final version and refactor, remove, or rewrite whatever that requires, without worrying about compatibility.
- No generic fallback and no implicit unexpected behavior. Do not add an `else` for an unknown case that would produce surprising behavior. Handle the known cases explicitly and surface anything else as a clear error.
- Every error or failure is reported clearly. A failure the AI or the operator must see is logged and returned, never swallowed.

### Naming and structure

- All functions are class methods. Never a free function in a namespace, and never a function in an anonymous namespace. Generic or shared logic goes on a helper class as a `private static` or public `static` method (`StringHelper::foo`, `ToolHelper::bar`).
- No member `_` suffix. Names carry the meaning, so correct naming replaces most comments.
- Keep headers, implementation, namespaces, types, names, and responsibilities consistent with each other.
- When a method takes on too much responsibility, extract small objective methods. Do not extract just to shrink a function, and avoid artificial abstractions that obscure the main flow.

### Comments

- These rules apply to every language in the repo, not just C++: `//` in C++, Swift, Kotlin, and Dart, and `#` in CMake, Makefile, YAML, and shell all follow the same standard. No `====`/`----` banner rows and no capitalized section headers anywhere, including build files.
- Comments are rare and used only where naming cannot carry the meaning. If it is obvious, decorative, or restates the code, delete it.
- A comment explains intent or context, not the literal code.
- Never add artificial section separators like `helpers`, `validators`, or `public methods`. Never write header comments that describe a method, a member, or a section.
- A `//` or `#` one-line comment is lowercase and one objective sentence, with no semicolon splitting one sentence into clauses.
- If a comment needs more than one line, finish each sentence with punctuation before the next line. Never continue one sentence onto the next line, and keep comments objective and natural rather than verbose or narrative.
- In a complex method, a short one-line comment can mark the intent of each meaningful block.

### Logging

- Every log message carries a `[Component]` prefix in PascalCase, e.g. `[BrowserTool]`, `[VectorStore]`, matching the class or subsystem that emits it.
- The message after the prefix starts with a capital letter and reads as a sentence: `spdlog::warn("[AgentLoop] No provider found for '{}'", name)`. This is the opposite of the lowercase rule for `//` comments, which never carry a prefix.
- Report every failure the operator or the AI must see; never swallow it. Keep the message objective and free of semicolon-split clauses, same as comments.

### Formatting and flow


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ionclaw-org/ionclaw](https://github.com/ionclaw-org/ionclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
