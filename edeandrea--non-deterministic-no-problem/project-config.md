---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**"Non-Deterministic? No Problem!"** is a demo application (Parasol Insurance) that shows how to
test and continuously evaluate non-deterministic AI systems.

It is a **single Quarkus application** (`org.parasol:parasol-app`, Java 25, Quarkus 3.39.3) with a React/PatternFly
frontend served via Quinoa. There are **no sub-modules** — one `pom.xml` at the root.

The Java source is split into two top-level packages representing two distinct concerns:

| Package | Concern |
|---|---|
| `org.parasol` | The insurance claims business application (claims REST API, AI chat bot, email notification, guardrails) |
| `ai.scoring` | The reusable AI-quality layer (Langfuse integration, session scoring, drift detection) |

Supporting docs:
- `README.md` — build/run instructions, Ollama profiles, Langfuse integration notes
- `langfuse-evaluation.md` — **the** design document: the three-tier evaluation strategy, Langfuse
  platform/language gaps, and the workarounds implemented here. Read this before touching anything
  under `ai.scoring.langfuse`.
- `docs/*.puml` — three PlantUML diagrams: `application-flow.puml` (the business flow —
  chat → tool → email), `continuous-scoring-architecture.puml` (single-container component view of
  the evaluation layer) and `continuous-scoring-sequence.puml` (the tier-2 session-scoring
  sequence).
- `images/arch.png` — the hand-drawn overview of the business flow, framed as "Code I write" vs
  "Is this code?", embedded at the top of README.md's Architecture section with
  `docs/application-flow.png` below it as the detailed complement. It is a **source-less raster**
  (no `.excalidraw`/`.drawio` original) that has already been pixel-edited — a white rectangle
  painted over a now-removed "Input Guardrails" box describing a component that does not exist (all
  seven guardrails here are output-side). Changing it means pixel editing or a full redraw, not
  editing a source file, and this machine has no Pillow, numpy or ImageMagick — the last edit
  needed a hand-rolled Python PNG codec.

## Commands

All Maven commands run from the repository root via the wrapper.

```bash
# Dev mode (OpenAI, default) — requires OPENAI_API_KEY (and COHERE_API_KEY for judge/sentiment)
./mvnw quarkus:dev

# Dev mode against a local Ollama
./mvnw -Pollama quarkus:dev

# Dev mode against Ollama via its OpenAI-compatible endpoint
./mvnw -Pollama-openai quarkus:dev

# Unit tests
./mvnw test

# A single test class
./mvnw test -Dtest=PolitenessOutputGuardrailTests

# Full verify (unit + integration tests)
./mvnw verify

# Exercise the drift-detection tests (otherwise skipped — see Testing below).
# Needs a reachable Langfuse with populated datasets.
./mvnw verify -Dquarkus.test.profile=drift

# Build, skipping tests
./mvnw package -DskipTests

# Run the built app outside dev mode
java -Dquarkus.profile=ollama,prod -jar target/quarkus-app/quarkus-run.jar
```

### Diagrams

```bash
# Re-render every docs/*.puml to a sibling PNG (pinned PlantUML, needs graphviz `dot`)
./docs/render-diagrams.sh
```

### Frontend (from `src/main/webui/`)

```bash
npm test        # Jest
npm run build   # production build into dist/
```

Quinoa builds the frontend as part of the Maven build — you rarely need to run npm directly.

### CI

`.github/workflows/simple-build-test.yml` runs `./mvnw -B clean verify` on Java 25 across the
`ollama` and `ollama-openai` profiles. CI has no real OpenAI/Cohere/Gemini credentials, so **any new
test must pass under the Ollama profiles**.

## Architecture

### Business application — `org.parasol`

**AI services** (Quarkus LangChain4j `@RegisterAiService`):

- `ClaimService` — the chat bot. `@ChatScoped` (session-scoped conversation), exposed as a chat
  route (`@ChatRoute("chat")` / `@DefaultChatRoute`) over the websocket chat-routes endpoint
  `/_chat/routes` provided by `quarkus-langchain4j-chat-scopes-websocket`. Uses RAG over
  `src/main/resources/policies/policy-info.pdf` (Easy RAG, embeddings reused via
  `easy-rag-embeddings.json`) and a `@ToolBox(NotificationService.class)`. Annotated with
  `@OutputGuardrails(DriftDetectionOutputGuardrail.class)`, which is a no-op unless
  `interaction-mode` is `DRIFT_DETECTION`.
- `GenerateEmailService` — generates a `{subject, body}` `Email` record; four output guardrails.
- `PolitenessService` — AI-backed politeness check used by `PolitenessOutputGuardrail`.

**Output guardrails** on `GenerateEmailService` (all extend `GenerateEmailOutputGuardrail`, itself a
`dev.langchain4j.guardrails.JsonExtractorOutputGuardrail<Email>`):
`EmailContainsRequiredInformationOutputGuardrail`, `EmailStartsAppropriatelyOutputGuardrail`,
`EmailEndsAppropriatelyOutputGuardrail`, `PolitenessOutputGuardrail`.

The project has **seven guardrail classes in total and they are all output guardrails** — the four
email ones above plus `DriftDetectionOutputGuardrail`, `SessionSentimentGuardrail` and
`EvaluatorResultOutputGuardrail`. There are **zero input guardrails**.

**Email flow:** the chat bot calls the `NotificationService.updateClaimStatus` tool → updates the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [edeandrea/non-deterministic-no-problem](https://github.com/edeandrea/non-deterministic-no-problem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
