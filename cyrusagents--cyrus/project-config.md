---
trigger: always_on
description: This package provides a provider-agnostic wrapper around the Gemini CLI that implements the `IAgentRunner` interface from `cyrus-core`.
---

# gemini-runner Package Guide

This package provides a provider-agnostic wrapper around the Gemini CLI that implements the `IAgentRunner` interface from `cyrus-core`.

## Overview

**GeminiRunner** translates between Gemini CLI's streaming JSON format and the Claude SDK message types, enabling seamless integration of Google's Gemini models into the Cyrus agent framework.

## Key Features

### 1. Result Message Coercion

Unlike Claude's CLI which includes final assistant content in result messages, Gemini's result messages contain only metadata (status, stats, duration). GeminiRunner solves this by:

- **Tracking** the last assistant message emitted during execution
- **Extracting** text content from the tracked message
- **Injecting** actual response content into result messages

**Implementation:**
- `GeminiRunner.lastAssistantMessage` - Private field tracking most recent assistant message
- `GeminiRunner.getLastAssistantMessage()` - Public accessor for external use
- `geminiEventToSDKMessage()` - Accepts optional `lastAssistantMessage` parameter to coerce result content

**Why this matters:**
Without coercion, result messages would always say "Session completed successfully" instead of containing the actual final output. This breaks EdgeWorker's expectation that result messages contain summary content from final subroutines.

### 2. Single-Turn Mode Support

Summary subroutines (like `concise-summary`, `question-answer`) need to run in single-turn mode to prevent unnecessary back-and-forth. GeminiRunner enables this through:

**Auto-Generated Settings:**
- On first spawn, creates `~/.gemini/settings.json` if missing
- Generates `-shortone` aliases for all main Gemini models:
  - `gemini-3-pro-preview-shortone`
  - `gemini-2.5-pro-shortone`
  - `gemini-2.5-flash-shortone`
  - `gemini-2.5-flash-lite-shortone`
- Each alias configured with `maxSessionTurns: 1`
- Enables `previewFeatures: true` for latest Gemini capabilities

**EdgeWorker Integration:**
- When `subroutine.singleTurn === true`, EdgeWorker appends `-shortone` to model name
- Example: `gemini-2.5-flash` → `gemini-2.5-flash-shortone`
- This ensures Gemini CLI enforces single-turn constraint

**Reference:**
- Gemini CLI Configuration: https://github.com/google-gemini/gemini-cli/blob/main/docs/get-started/configuration.md

### 3. Streaming Stdin Support

GeminiRunner supports both string and streaming prompt modes:

**String Mode:**
```typescript
await runner.start("Analyze this codebase");
```

**Streaming Mode:**
```typescript
await runner.startStreaming("Initial task");
runner.addStreamMessage("Additional context");
runner.addStreamMessage("More details");
runner.completeStream(); // Closes stdin to trigger processing
```

**Critical Implementation Details:**
- Initial prompt written to stdin **immediately** after spawn (line 253)
- Stdin remains **open** for `addStreamMessage()` calls
- Stdin closed only in `completeStream()` (line 118)
- Prevents gemini CLI's 500ms timeout from firing prematurely

**How Gemini CLI stdin works:**
1. 500ms timeout starts when process spawns
2. If **no data** arrives within 500ms → assumes no piped input, continues
3. Once **data arrives** → cancels timeout, waits for stdin close (`end` event)
4. Continues reading chunks until stdin closes

**Test Coverage:** `test-scripts/test-stdin-direct.ts` proves multiple stdin writes work correctly.

## Testing

### Comprehensive Integration Test

The package includes one comprehensive end-to-end integration test in `test-scripts/test-gemini-runner.ts` that verifies all GeminiRunner features:

#### test-gemini-runner.ts
**Purpose:** Complete end-to-end verification of all GeminiRunner functionality

**What it tests:**

1. **Settings.json Auto-Generation**
   - `~/.gemini/settings.json` created if missing
   - All 4 `-shortone` model aliases present
   - Each alias has `maxSessionTurns: 1`

2. **Stdin Streaming (Multiple Writes)**
   - Multiple messages written to stdin
   - Process accepts all messages before stdin closes
   - Gemini processes all input correctly

3. **Result Message Coercion**
   - Result message contains actual assistant response
   - NOT generic "Session completed successfully"
   - Content matches last assistant message exactly

4. **Single-Turn Mode (All 4 Models)**
   - Tests all 4 main Gemini models with `-shortone` aliases:
     - `gemini-3-pro-preview-shortone`
     - `gemini-2.5-pro-shortone`
     - `gemini-2.5-flash-shortone`
     - `gemini-2.5-flash-lite-shortone`
   - Each completes in ≤1 turns
   - maxSessionTurns constraint enforced

5. **getLastAssistantMessage() Public API**
   - Returns null before session starts
   - Captures last assistant message after session
   - Content accessible via public method

**Usage:**
```bash
cd packages/gemini-runner
export GEMINI_API_KEY='your-key-here'
pnpm build
bun test-scripts/test-gemini-runner.ts
```

**Expected output:**
```
============================================================
🧪 GeminiRunner End-to-End Integration Tests
============================================================

Prerequisites:
   ✅ GEMINI_API_KEY environment variable set
   ✅ Test directory: /Users/user/.cyrus-test-gemini


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cyrusagents/cyrus](https://github.com/cyrusagents/cyrus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
