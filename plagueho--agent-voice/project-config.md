---
trigger: always_on
description: Agent Voice is a VS Code extension that turns Azure OpenAI GPT Realtime into a full-duplex voice copilot for GitHub Copilot Chat. The extension orchestrates configuration, authentication, realtime audio streaming, and Copilot prompt delivery through a service-based dependency injection layer. Treat this guide as the single source for automated agents working in this repository.
---

# Agent Voice VS Code Extension Development Guide

## Project Overview

Agent Voice is a VS Code extension that turns Azure OpenAI GPT Realtime into a full-duplex voice copilot for GitHub Copilot Chat. The extension orchestrates configuration, authentication, realtime audio streaming, and Copilot prompt delivery through a service-based dependency injection layer. Treat this guide as the single source for automated agents working in this repository.

## Tech Stack

- **Runtime**: VS Code extension host (Node.js 22+, ES2022)
- **Language**: TypeScript 5 with strict mode and ES module syntax compiled to CommonJS
- **AI**: Azure OpenAI GPT Realtime (`gpt-realtime`) via WebRTC (preferred) with WebSocket fallback
- **Auth**: `@azure/identity` (`DefaultAzureCredential`) with optional ephemeral key issuance
- **Networking**: `openai` SDK (Azure-compatible), `ws`, `axios`
- **Audio**: WebRTC transport, AudioWorklets, PCM pipelines, interruption engine
- **Tooling**: TypeScript compiler, ESLint (flat config), Mocha + `@vscode/test-electron`, Webpack, NYC coverage, VSCE packaging

## Development Environment Setup

### Requirements

- Node.js 22 or later, VS Code 1.105+, Azure CLI with Bicep, Git, PowerShell (for Bicep tasks)

### Setup

```bash
npm install
az version
az bicep version
```

Optional: `npm install -g @vscode/vsce` for packaging.

## Project Structure

```text
agent-voice/
├── src/
│   ├── extension.ts                     # Activation entry point
│   ├── core/                            # ExtensionController, retry utilities, logger
│   ├── config/                          # Configuration manager, sections, validators
│   ├── auth/                            # Credential + ephemeral key services, validators
│   ├── services/                        # Privacy, audio feedback, realtime STT, error handling
│   ├── audio/                           # WebRTC transport, capture, processing chain, worklets
│   ├── conversation/                    # State machine, transcript privacy aggregation
│   ├── copilot/                         # Copilot Chat bridge + prompt flow
│   ├── session/                         # Session manager, timers, interruption engine
│   ├── telemetry/                       # Lifecycle + metrics logging
│   └── ui/                              # Voice control panel, status bar, error presenter
├── test/                                # Unit + integration specs and fixtures
├── docs/                                # Design references and technical indices
├── infra/                               # Azure Bicep templates and scripts
├── media/                               # Webview assets (JS/CSS/worklets)
└── .vscode/                             # Tasks, launch configs
```

## Core Development Principles

- Obey single-responsibility modules and service injection contracts (`ServiceInitializable` lifecycle).
- Follow clean TypeScript practices: strict typing, async/await, no implicit `any`, camelCase variables, PascalCase classes, kebab-case filenames.
- Prefer composition over inheritance; avoid long methods or monolithic classes.
- Keep configuration, authentication, session, and UI initialization in the documented order.
- Log with structured metadata and sanitize user content before persistence or telemetry.

## Development Workflow and Architecture

- `ExtensionController` orchestrates dependency initialization, error orchestration, privacy purges, and UI wiring.
- Boot sequence: `ConfigurationManager` → `CredentialManagerImpl`/`EphemeralKeyServiceImpl` → `SessionManagerImpl` (with timers + recovery) → UI surfaces (`VoiceControlPanel`, `StatusBar`, `ErrorPresenter`).
- Conversation execution flows through `ConversationStateMachine`, `ChatIntegration`, `TranscriptPrivacyAggregator`, `AudioFeedbackServiceImpl`, and `InterruptionEngineImpl`.
- Error handling relies on `ErrorEventBusImpl`, `RecoveryOrchestrator`, retry providers/executors, and typed `Agent VoiceError` envelopes.
- Privacy controls (`PrivacyController`) manage transcript lifecycle, purge commands, and policy enforcement.

## TypeScript & Code Quality

- Read `.github/instructions/typescript-5-es2022.instructions.md` before touching `*.ts` files.
- Use ES module syntax in source; compiled output lives only in `out/` (never edit compiled JS).
- ESLint (flat config) is authoritative; fix lint before running integration tests.
- Prefer interfaces for object shapes, discriminated unions for events, and utility types instead of `any`.

## Azure and External Service Integration

- Default to keyless auth with `DefaultAzureCredential` + `getBearerTokenProvider`. Ephemeral keys remain available for WebRTC startups.
- Use `AzureOpenAI` from the `openai` SDK with deployment + API version set by configuration (`2025-04-01-preview` for REST, `2025-08-28` for realtime).
- All credentials persist in VS Code `SecretStorage` via `CredentialManagerImpl`; never write secrets to disk or logs.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PlagueHO/agent-voice](https://github.com/PlagueHO/agent-voice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
