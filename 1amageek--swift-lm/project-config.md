---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working in this repository.

## Global Instructions

These repository instructions are cumulative with `~/.claude/CLAUDE.md` and the
repository-local `CLAUDE.md`. When one of these documents changes, keep the
shared rules aligned here as well.

### Trust and ownership

- Trust is the highest-priority value. Investigate suspicious or ambiguous
  content before acting.
- Assume repository ownership and GitHub operations are tied to
  `https://github.com/1amageek`.

### Commit, documentation, and language rules

- Never include Claude or AI promotion in git commit messages.
- Never add `Co-Authored-By: Claude` or similar signatures.
- When reading Apple documentation on `developer.apple.com/documentation`, use
  `remark`.
- Keep user-facing conversation in Japanese.
- Keep code comments, identifiers, `MARK` sections, and documentation comments
  in English.

### Prompt and coding rules

- Do not include few-shot examples in prompts sent to LLMs. Describe the schema
  and constraints without example answers.
- Follow SOLID, value types first, protocol-oriented design, one primary type
  per file, dependency injection, and explicit typed error handling.
- Do not use `try?`.
- Set timeouts on test commands and run tests in focused scopes.
- Before fixing code, read the related area, understand the impact, and prefer
  root-cause fixes over symptom patches.
- Do not silence compiler or type errors with casts whose only purpose is to
  make the error disappear.
- Do not mix unrelated changes into a bug fix.
- Never introduce silent fallback behavior. Report failures explicitly and let
  callers decide.
- Wrap synchronous AI-model load and inference boundaries in
  `autoreleasepool` when Metal-backed objects would otherwise outlive the
  needed scope.

## Project Goal

`swift-lm` is a Core AI-first declarative model authoring and export package for
macOS and iOS 27+. It provides the graph and runtime foundation for Core
AI-backed language-model applications on Apple platforms.

The core thesis is to describe model families once in Swift, normalize them to
backend-independent IR, and emit a validated Core AI export contract. Apple's
official exporter owns deployment asset generation for supported models. Custom
stateful families use the low-level Core AI Torch converter and Swift runtime
wrapper.

The direct Metal compiler and `SwiftLM` runtime remain as the 0.10 compatibility
path. They are not the default target for new public APIs or model support.

The consumer input contract is a HuggingFace bundle
(`config.json`, `safetensors`, `tokenizer.json`, and related metadata), not a
model-specific Swift type.

## Project Overview

`swift-lm` is a Swift package for declarative model graphs, Core AI export, and
Core AI runtime integration on Apple platforms.

The current repository is not a GGUF/MLX runtime. The active architecture is:

- HuggingFace model bundles as input: `config.json`, `tokenizer.json`, `tokenizer_config.json`, `chat_template.jinja`, `*.safetensors`
- `safetensors` as the source of truth for weights
- STAF as a regenerable GPU execution cache for the 0.10 compatibility path
- a backend-independent IR and model-declaration DSL
- a versioned Core AI export-document format
- Apple's Core AI model exporter for standard Transformer models
- a low-level Core AI program exporter for custom stateful families
- direct Metal compilation and execution for the 0.10 compatibility path

Core AI consumers start with `CoreAIExportDocument` or the `swiftlm-ir` tool,
validate the document with `swiftlm-coreai`, and produce an `.aimodel` through
Apple's exporter. `SwiftLMFoundationModels` adapts Apple's high-level language
bundle API, while `SwiftLMCoreAI` owns low-level asset and stateful execution.
`SwiftLM.ModelBundleLoader` remains the direct Metal compatibility entry point.

## Build & Test

```bash
# Build
swift build

# Run all tests
xcodebuild test -scheme swift-lm-Package -destination 'platform=macOS'

# Run one test target
xcodebuild test -scheme swift-lm-Package -destination 'platform=macOS' -only-testing:ModelsTests
xcodebuild test -scheme swift-lm-Package -destination 'platform=macOS' -only-testing:SwiftLMTests
xcodebuild test -scheme swift-lm-Package -destination 'platform=macOS' -only-testing:MetalCompilerTests
```

Important:

- Prefer `xcodebuild test` over `swift test` for this repository.
- For the Qwen3.5+ multimodal suites, prefer [`scripts/benchmarks/run-qwen35-vision-tests.sh`](/Users/1amageek/Desktop/swift-lm/scripts/benchmarks/run-qwen35-vision-tests.sh) over a single large `xcodebuild test` invocation. It uses `build-for-testing` once and then runs `test-without-building` suite-by-suite to reduce peak memory pressure.
- For generation benchmarks, prefer [`scripts/benchmarks/run-generation-pipeline.sh`](/Users/1amageek/Desktop/swift-lm/scripts/benchmarks/run-generation-pipeline.sh) over running the full benchmark file in one `xcodebuild test` process. It builds once and then runs the split benchmark suites sequentially.
- Generation benchmark suites are intentionally split by cost:
  - `SwiftLMTests/GenerationThroughputBenchmarkTests`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [1amageek/swift-lm](https://github.com/1amageek/swift-lm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
