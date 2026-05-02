---
trigger: always_on
description: This document describes the specialized tax preparation agent system built on the OpenAI Codex SDK platform.
---

# Tax Agent Documentation

This document describes the specialized tax preparation agent system built on the OpenAI Codex SDK platform.

## Overview

The Tax Preparation Agent provides an intelligent, end-to-end tax filing system that automatically processes tax documents, performs calculations, and generates completed tax returns. The system uses an **agent-driven architecture** where the Codex agent orchestrates tax processing via CLI tools.

### Key Components

- **Tax Agent Server**: Express-based server with agent-driven tax processing
- **taxctl CLI**: Command-line tools the agent uses to parse, calculate, and generate forms
- **AI Tax Assistant**: Conversational interface for tax guidance
- **Tax Engine**: 2024 tax calculation and Form 1040 generation

## Agent-Driven Architecture

The system uses an "agent as orchestrator" pattern where the Codex agent executes shell commands to process taxes:

```
Frontend Upload → POST /api/tax/process-documents
                      ↓
              CodexService.runAgentTurn(prompt)
                      ↓
              Agent executes CLI commands:
                - taxctl list-documents
                - taxctl parse <file>
                - taxctl calculate (JSON stdin)
                - taxctl generate-pdf (JSON stdin or file)
                      ↓
              Agent emits structured JSON result
                      ↓
              Return to frontend
```

### Why Agent-Driven?

- **Flexible**: Agent can reason about complex tax scenarios
- **Extensible**: Adding new document types (1099s, 1098s) just requires updating the prompt
- **Intelligent**: Agent can handle edge cases and ambiguous data
- **Future-proof**: Can incorporate conversational data from chat into tax processing

## taxctl CLI Tool

The `taxctl` CLI provides 4 commands the agent uses for tax processing:

```bash
# List documents in input folder
taxctl list-documents
# Output: { "success": true, "documents": [...], "count": 1 }

# Parse a tax document
taxctl parse w2_2024.pdf
# Output: { "success": true, "documentType": "w2", "data": {...} }

# Calculate taxes (reads JSON from stdin)
echo '{"w2Forms": [...], "taxpayerInfo": {...}, "filingStatus": "single"}' | taxctl calculate
# Output: { "success": true, "form1040": {...}, "summary": {...} }

# Generate PDF (from file or stdin)
taxctl generate-pdf /tmp/content/output/form1040.json
# OR pipe from calculate:
echo '{"form1040": {...}}' | taxctl generate-pdf
# Output: { "success": true, "pdfPath": "/tmp/content/output/....pdf" }
```

### Agent Package File Structure

```
packages/tax-processing/src/
  cli/
    index.ts                    # Commander CLI entry point
    commands/
      list-documents.ts         # List /tmp/content/input/ files
      parse.ts                  # Parse W-2/1099 documents
      calculate.ts              # Calculate taxes (stdin JSON)
      generate-pdf.ts           # Generate PDF from JSON (file or stdin)
  bin/
    taxctl.ts                   # Executable entry point
    run-agent-turn.ts           # Single-turn agent runner
  lib/
    agent-loader.ts             # Prompt loading infrastructure
    tax-agent-library.ts        # Tax-specific agent API
  prompts/
    agent.txt                   # Main agent behavior
    shared/
      w2-parsing.txt            # W-2 parsing instructions
      tax-brackets-2024.txt     # Tax calculation rules (2024)
      form1040-schema.txt       # Form 1040 JSON schema
  services/
    codex.service.ts            # Codex SDK wrapper
    document-parser.ts          # W-2 parsing logic
    tax-engine.ts               # Tax calculations
    pdf-generator.ts            # PDF generation
```

## Core Services

### CodexService (`packages/tax-processing/src/services/codex.service.ts`)

The enhanced `CodexService` provides two modes:

1. **Streaming Chat** (`streamResponse`): For conversational tax guidance via SSE
2. **Agent Turn** (`runAgentTurn`): For single-turn agent processing with CLI tools

```typescript
// Agent-driven processing
const result = await codexService.runAgentTurn(prompt);
// Returns: { success, response, rawResponse, error? }
```

### Tax Agent Prompt Library (`packages/tax-processing/src/lib/tax-agent-library.ts`)

The tax agent prompt system is now modularized using text files for easy customization:

**Main Agent File** (`src/prompts/agent.txt`):
- High-level agent behavior and workflow
- References to shared instruction files
- Critical rules and error handling

**Shared Instruction Files** (`src/prompts/shared/`):
- `w2-parsing.txt`: W-2 parsing instructions and CLI tool usage
- `tax-brackets-2024.txt`: 2024 tax calculation workflow and brackets
- `form1040-schema.txt`: Complete Form 1040 JSON schema

**API**:
```typescript
// Load complete tax agent system prompt
const systemPrompt = getTaxAgentSystemPrompt();

// Build task-specific prompt with context
const taskPrompt = buildTaxAgentPrompt({
  filename: 'w2_2024.pdf',
  filingStatus: 'single',
  taxpayerInfo: { ... }
});

// Use custom agent file for benchmarking
const customPrompt = buildCustomTaxAgentPrompt(
  'examples/agent-conservative.txt',
  { filename: 'w2_2024.pdf', filingStatus: 'single' }
);
```

### Supporting Services


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [runloopai/codex-tax-man](https://github.com/runloopai/codex-tax-man) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
