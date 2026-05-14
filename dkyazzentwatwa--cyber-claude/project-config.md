---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Development Commands

```bash
npm run build          # Build TypeScript to dist/
npm run dev            # Run in development mode (tsx)
npm start              # Run built CLI
npm test               # Run tests with vitest

# CLI usage after build
./dist/cli/index.js scan --quick
./dist/cli/index.js webscan https://example.com
./dist/cli/index.js web3 scan contract.sol
./dist/cli/index.js interactive
```

## Architecture Overview

### Multi-Provider AI System

```
CyberAgent (src/agent/core.ts)
    ↓
AIProvider interface (src/agent/providers/base.ts)
    ├─ ClaudeProvider (claude.ts)
    ├─ OpenAIProvider (openai.ts)
    ├─ GeminiProvider (gemini.ts)
    └─ OllamaProvider (ollama.ts)
```

Provider selection happens in `CyberAgent` constructor based on model's `provider` field from `src/utils/models.ts`. Conversation history is provider-agnostic.

### Agent Modes

Seven modes defined in `src/agent/prompts/system.ts` (all lowercase):
- `base`, `redteam`, `blueteam`, `desktopsecurity`, `webpentest`, `osint`, `smartcontract`

System prompts composed by: `base prompt + mode-specific prompt`

### CLI Structure

```
src/cli/index.ts (entry, registers all commands)
    ↓
src/cli/commands/*.ts (individual command files)
    ↓
src/cli/session.ts (InteractiveSession for REPL mode)
```

Default action: `cyber-claude` with no args starts interactive session.

### Security Tools Architecture

**Desktop/System** (`src/agent/tools/`):
- `DesktopScanner` - Uses `systeminformation` for OS/process/network data
- `HardeningChecker` - Platform-specific security checks
- `SecurityReporter` - Formats findings to terminal/JSON/Markdown

**Web Security** (`src/agent/tools/web/`):
- `WebScanner` - Orchestrates OWASP Top 10 vulnerability detection
- `HeaderAnalyzer` - Security headers (CSP, HSTS, etc.)
- `Authorization` - Ethical framework with domain blocklists

**Network Analysis** (`src/agent/tools/`):
- `PcapAnalyzer` - Parses pcap files, extracts protocols/conversations
- `PcapReporter` - Statistics and export to JSON/Markdown/CSV

**OSINT** (`src/agent/tools/osint/`):
- 10 tools: DNS, WHOIS, subdomains, emails, usernames, breaches, tech detection, Wayback, IP lookup
- All tools require NO API keys
- `OSINTOrchestrator` coordinates tools, `OSINTReporter` formats output

**Web3/Smart Contract** (`src/agent/tools/web3/`):
- `SmartContractScanner` - Orchestrates Solidity vulnerability detection
- `SolidityParser` - Parses contract source into structured format
- `Web3Reporter` - Terminal display and Markdown export with DeFiHackLabs references
- 11 vulnerability detectors in `detectors/`:
  - ReentrancyDetector, AccessControlDetector, IntegerOverflowDetector
  - StateModificationDetector, FlashLoanDetector, OracleManipulationDetector
  - PrecisionLossDetector, ArbitraryCallDetector, StorageCollisionDetector
  - TimestampDependenceDetector, WeakRandomnessDetector
- `exploits/DeFiHackLabsDB.ts` - Real-world exploit references (24 curated exploits)

### Autonomous Agent Mode

`src/agent/core/` contains agentic execution:
- `AgenticCore` - Main orchestrator for autonomous tasks
- `Planner` - AI-powered task decomposition
- `Context` - Execution state and findings
- `Reflection` - Self-assessment and adaptation
- `Validator` - Safety checks

### Professional Analysis

- **IOC Extraction** (`src/utils/ioc.ts`) - IPs, domains, URLs, hashes, CVEs with STIX 2.1 export
- **MITRE ATT&CK** (`src/utils/mitre.ts`) - 20+ technique mappings
- **Evidence Preservation** (`src/utils/evidence.ts`) - Chain of custody, triple hash verification

## Key Implementation Patterns

### ESM Module System

**Critical**: This is ESM-only (`"type": "module"`). All imports MUST include `.js` extension:
```typescript
import { config } from '../utils/config.js';  // Correct
import { config } from '../utils/config';     // Will NOT resolve
```

### Adding a New Vulnerability Detector (Web3)

1. Create detector in `src/agent/tools/web3/detectors/` implementing `VulnDetector` interface
2. Register in `src/agent/tools/web3/detectors/index.ts` DetectorRegistry
3. Add vulnerability type to `Web3VulnType` in `src/agent/tools/web3/types.ts`
4. Optionally add exploit references in `exploits/DeFiHackLabsDB.ts`

### Adding a New CLI Command

1. Create `src/cli/commands/<name>.ts` exporting `create<Name>Command()` function
2. Register in `src/cli/index.ts` with `program.addCommand()`
3. For interactive support, add handling in `InteractiveSession.handleCommand()` in `session.ts`

### Adding a New AI Provider

1. Create provider class in `src/agent/providers/` implementing `AIProvider` interface
2. Add provider type to model definitions in `src/utils/models.ts`
3. Add initialization case in `CyberAgent` constructor

## UI System

`src/utils/ui.ts` provides consistent terminal output:
- `ui.banner()`, `ui.section()`, `ui.box()` - Formatted sections
- `ui.spinner()` - Loading states (returns Ora instance)
- `ui.finding()` - Security findings with severity colors
- `ui.success/error/warning/info()` - Status messages
- `ui.formatAIResponse()` - Markdown to terminal conversion


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dkyazzentwatwa/Cyber-Claude](https://github.com/dkyazzentwatwa/Cyber-Claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
