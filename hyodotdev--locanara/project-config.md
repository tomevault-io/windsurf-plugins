---
trigger: always_on
description: > **Repository: hyodotdev/locanara**
---

# Locanara - AI Agent Guidelines

> **Repository: hyodotdev/locanara**
>
> This is the open-source SDK for on-device AI.

## Project Overview

Locanara is an on-device AI **framework** for iOS, Android, and Web, inspired by LangChain. It provides composable chains, memory management, guardrails, and a pipeline DSL for building production AI features using platform-native models.

### Core Principles

- **On-Device Only**: All AI processing happens locally. No cloud fallback.
- **Privacy First**: User data never leaves the device.
- **Framework, Not Just API**: Composable chains, memory, guardrails, and pipeline DSL.
- **Unified API**: Same concepts and structure across all platforms.

### Supported Platforms

- **iOS/macOS**: Foundation Models on iOS 26+/macOS 26+; local fallback engines support iOS 17+/macOS 14+
- **Android**: Gemini Nano (ML Kit GenAI Prompt API) on supported Android 14+ devices; ExecuTorch supports API 31+
- **Web**: Chrome Built-in AI. Treat browser/API availability as runtime capability, not a static browser-version guarantee.

### Distribution

| Platform | Installation                                                     |
| -------- | ---------------------------------------------------------------- |
| iOS      | `https://github.com/hyodotdev/locanara` (SPM) or CocoaPods       |
| Android  | Maven Central: `implementation("com.locanara:locanara:VERSION")` |
| Web      | npm: `npm install locanara`                                      |

## Project Structure

```text
locanara/
├── packages/
│   ├── apple/          # Swift SDK (SPM + CocoaPods)
│   │   ├── Sources/
│   │   │   ├── Core/            # LocanaraModel, PromptTemplate, OutputParser, Schema
│   │   │   ├── Composable/      # Chain, Tool, Memory, Guardrail
│   │   │   ├── BuiltIn/         # SummarizeChain, ClassifyChain, etc.
│   │   │   ├── DSL/             # Pipeline, PipelineStep, ModelExtensions
│   │   │   ├── Runtime/         # Agent, Session, ChainExecutor
│   │   │   ├── Platform/        # FoundationLanguageModel
│   │   │   ├── Engine/          # InferenceRouter, LlamaCppEngine
│   │   │   ├── ModelManager/    # ModelManager, ModelDownloader
│   │   │   ├── RAG/             # VectorStore, DocumentChunker
│   │   │   ├── Personalization/ # PersonalizationManager, FeedbackCollector
│   │   │   └── Features/        # Legacy feature executors
│   │   ├── Tests/
│   │   └── Example/    # Example app
│   ├── android/        # Kotlin SDK (Maven Central)
│   │   ├── locanara/
│   │   │   └── src/main/kotlin/com/locanara/
│   │   │       ├── core/            # LocanaraModel, PromptTemplate, OutputParser, Schema
│   │   │       ├── composable/      # Chain, Tool, Memory, Guardrail
│   │   │       ├── builtin/         # SummarizeChain, ClassifyChain, etc.
│   │   │       ├── dsl/             # Pipeline, ModelExtensions
│   │   │       ├── runtime/         # Agent, Session, ChainExecutor
│   │   │       ├── platform/        # PromptApiModel
│   │   │       ├── engine/          # InferenceEngine, ExecuTorchEngine
│   │   │       ├── rag/             # VectorStore, RAGManager
│   │   │       └── personalization/ # PersonalizationManager
│   │   └── example/    # Example app
│   ├── gql/            # GraphQL schema definitions and type generators
│   ├── web/            # Browser SDK for Chrome Built-in AI
│   └── site/           # Website (landing + docs + community)
├── libraries/          # Third-party framework integrations
│   ├── expo-ondevice-ai/         # Expo module
│   ├── react-native-ondevice-ai/ # React Native Nitro module
│   └── flutter_ondevice_ai/      # Flutter plugin
└── .claude/
    ├── commands/       # Slash commands
    └── guides/         # Project guides
```

## Skills and Slash Commands

- `$locanara-workflows` - Natural-language router for repository workflows
- `$locanara-docs` - Implementation-backed documentation authoring
- `$rebase-main` - Safe main update and work-branch rebase
- `$review-pr` - PR feedback, CI, and five-minute monitoring loop
- `$review-self` - Self-review and five-minute stabilization loop
- `/locanara` - Project-wide routing and source-of-truth map
- `/gql` - GraphQL schema and generated-type workflow
- `/apple` - Apple SDK development
- `/android` - Android SDK development
- `/test` - Cross-platform test workflow
- `/docs` - Documentation workflow
- `/audit-code` - Code audit against project rules
- `/verify-all` - Changed-path or full repository verification
- `/resolve-issue` - Evidence-backed GitHub issue workflow
- `/review-pr` - Detailed pull request feedback command
- `/knowledge-compile` - Upstream technology research and impact notes
- `/commit` - Local commit and PR workflow

### AI Assistant Compatibility

`AGENTS.md` is the root policy source shared by the `CLAUDE.md` and `GEMINI.md`
symlinks. Slash-command workflows live in `.claude/commands/`. Canonical
cross-agent skill procedures live in `.codex/skills/`, while matching
`.claude/skills/` files are thin Claude Code discovery adapters.

Install only the globally unique Locanara skills into the local Codex home when
needed:

```bash
./.codex/scripts/install-skills.sh
```

Keep `$review-pr`, `$review-self`, and `$rebase-main` repository-local because
other projects provide project-specific workflows with the same names. When

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hyodotdev/locanara](https://github.com/hyodotdev/locanara) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-01 -->
