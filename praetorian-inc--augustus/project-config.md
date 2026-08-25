---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Augustus is a Go-based LLM vulnerability scanner that tests large language models against 230+ adversarial attacks. It integrates with 28 LLM providers and produces actionable vulnerability reports.

## Build and Test Commands

```bash
# Build
make build                    # Build binary to bin/augustus
go build ./cmd/augustus       # Alternative direct build

# Test
make test                     # Run all tests with race detection
go test ./pkg/scanner -v      # Run specific package tests
go test ./... -run TestName   # Run single test by name
make test-equiv               # Run equivalence tests (Go vs Python)
make test-cover               # Run tests with coverage report

# Lint & format
make lint                     # Run golangci-lint per .golangci.yml (standard linters + gofumpt/goimports formatters); auto-installs the pinned version, falls back to go vet if unavailable
golangci-lint fmt ./...       # Apply gofumpt + goimports formatting — matches what CI enforces (plain `go fmt` no longer satisfies the lint gate)
```

Linting is configured by `.golangci.yml` (golangci-lint v2): the default `standard` linter set plus a `formatters` block enabling `gofumpt` and `goimports`. CI runs this via the shared `public-workflows/go-ci.yml` reusable workflow on every PR, so formatting drift fails the build — keep the tree `golangci-lint fmt`-clean.

## Architecture

### Core Interfaces (pkg/types/)

All capabilities implement these interfaces:
- **Prober**: Generates attack prompts, returns `[]*attempt.Attempt`
- **Generator**: Wraps LLM APIs, handles `*attempt.Conversation` → `[]attempt.Message`
- **Detector**: Analyzes outputs, returns scores `[0.0, 1.0]` (0=safe, 1=vulnerable)
- **Buff**: Transforms prompts before sending (encoding, translation, paraphrase)

Probes may also implement these **optional** interfaces (all in `pkg/types/prober.go`) for advanced behavior:
- **ProbeMetadata**: `Description`/`Goal`/`GetPrimaryDetector`/`GetPrompts` for introspection
- **ProbeDetectorConfig**: `GetDetectorConfig()` — per-probe detector config overrides
- **ProbeSecondaryDetectors**: `GetSecondaryDetectors()` — run extra detectors alongside the primary; the attempt verdict reflects the **max score across all detectors** (see `attempt.GetEffectiveScores`), so a secondary hit alone marks the attempt vulnerable
- **ProbeTools**: `GetTools()` / `GetToolChoice()` — declare function-calling tool schemas for tool-use/agent probes (sent via the native wire layer in `internal/attackengine/toolcalls.go`)
- **RiskDescriber**: `RiskInfo()` — carry a curated `types.RiskInfo` write-up (description/impact/recommendation/references/taxonomies/CVSS v4.0 vector/verification) next to the probe so consumers retrieve it via type assertion instead of duplicating it. `Verification` is a static, reproduction-oriented write-up (how the probe confirms the finding + how to reproduce it) distinct from `Description`; it carries no live target values and no template tokens, so a consumer can render it and safely append a data-built reproduction command

Generators may also implement these **optional** interfaces (in `pkg/types/generator.go`):
- **UsageReporter**: `AccumulatedTokens() int64` — reports the cumulative tokens consumed across all `Generate` calls. The scanner type-asserts each generator for this interface and records the per-run delta into `Metrics.TokensConsumed` (surfaced as `augustus_tokens_consumed`). Implement it for free by embedding `types.UsageCounter` (a concurrency-safe `atomic.Int64`) and calling `g.AddTokens(...)` wherever the provider returns usage. Generators whose provider doesn't report usage still embed `UsageCounter` and simply never `AddTokens`, contributing 0 (honest partial coverage, never an estimate).
- **VisionCapable**: `SupportsVision() bool` — declares that the generator's wire layer can transmit image attachments (`Message.Images`). Multimodal image probes gate on this to skip generators that can't carry images rather than silently sending a text-only request and mis-reporting the target as safe. Report **structural** capability (the generator emits image content blocks), not per-model support — e.g. an OpenAI-compatible generator returns `true` on its chat path even though a given model may ignore images; for generators with both image-capable and text-only paths (OpenAI/Azure completion models, Bedrock Titan/Llama), return the path-accurate value (e.g. `g.isChat`, or the model family).
- **DocumentCapable**: `SupportsDocuments() bool` — the document-modality parallel of `VisionCapable`: declares that the generator's wire layer can transmit document attachments (`Message.Documents`, e.g. PDFs). Document probes (`internal/probes/pdf/*`) gate on this so a generator that can't carry documents fails the probe rather than silently sending a text-only request and mis-reporting the target as safe. Report **structural** capability — Anthropic returns `true` unconditionally; Bedrock returns `true` only for the Claude builder (Nova/Titan/Llama return `false`, as only the Claude path emits document blocks).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [praetorian-inc/augustus](https://github.com/praetorian-inc/augustus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
